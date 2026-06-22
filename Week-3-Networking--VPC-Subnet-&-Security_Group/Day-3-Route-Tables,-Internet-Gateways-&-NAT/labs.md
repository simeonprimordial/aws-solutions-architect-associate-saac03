# Labs — Week 3 Day 3: Route Tables, Internet Gateways & NAT

---

## Lab 1: Understand Security Groups vs NACLs Before Building

### Steps

1. Before touching the console, write down the 5 key differences between Security Groups and NACLs from memory.
2. Compare your list against the reference table:

| Feature | Security Group | Network ACL (NACL) |
|---|---|---|
| Level | Instance (applied to ENI) | Subnet (applied to entire subnet) |
| State | Stateful — return traffic auto-allowed | Stateless — both directions explicit |
| Rules | Allow only — no deny | Allow AND Deny |
| Evaluation | All rules evaluated, most permissive wins | Number order, first match wins |
| Default inbound | Deny all | Allow all |

3. Identify: in a 3-tier architecture, which layer does most of the work? Security Groups. NACLs are a secondary layer for subnet-wide blocking (specific IPs, insecure protocols like Telnet/FTP).

### What I Observed

Writing out the comparison from memory before consulting notes revealed exactly which distinction I was fuzzy on — the evaluation model. I knew Security Groups were stateful and NACLs were stateless, but I initially wrote "Security Groups allow explicit deny" which is wrong. Security Groups only have allow rules. Deny is implicit — anything not explicitly allowed is denied. NACLs are where explicit deny rules live.

### What I Learned

- Security Groups do the heavy lifting for instance-level access control. NACLs are a subnet-wide backstop.
- Security Groups have no explicit deny — not having an allow rule is effectively a deny. This means you cannot write a "block this specific IP but allow everything else" rule in a Security Group.
- NACLs are where you block specific IP addresses at the subnet level. The use case is: a known malicious IP is scanning your infrastructure — add a NACL DENY rule to block it at the subnet before it reaches any instance's Security Group.

---

## Lab 2: Create the Load Balancer Security Group

### Steps

1. Navigate to **EC2 → Security Groups → Create security group**.
2. Security group name: `SG-LoadBalancer`.
3. Description: `Allow HTTPS and HTTP from internet`.
4. VPC: `OluTech-Production-VPC`.
5. **Inbound rules — Add rule:**
   - Type: HTTPS | Protocol: TCP | Port: 443 | Source: `0.0.0.0/0`
6. **Add second inbound rule:**
   - Type: HTTP | Protocol: TCP | Port: 80 | Source: `0.0.0.0/0`
7. Outbound rules: keep the default (`All traffic | 0.0.0.0/0`).
8. Click **Create security group**.
9. Confirm both inbound rules appear in the Inbound rules tab.

### What I Observed

The console separates inbound and outbound rule sets clearly. The default outbound rule (`All traffic → 0.0.0.0/0`) is pre-populated — Security Groups allow all outbound by default. I left this unchanged for the ALB: it needs to be able to forward traffic to the web tier on any port the application uses.

The `0.0.0.0/0` source on both inbound rules means any IP on the internet can reach the ALB on ports 80 and 443. This is correct and intentional — the ALB is the public-facing entry point. No instance behind it has a public IP. The ALB absorbs internet exposure so the application tier does not have to.

### What I Learned

- The ALB Security Group is the only one that should have `0.0.0.0/0` as an inbound source. Every other tier in the chain uses a Security Group reference as the source — never a CIDR or `0.0.0.0/0`.
- HTTP on port 80 is included alongside HTTPS on 443 to allow the ALB to respond to unencrypted requests and redirect them to HTTPS. Without port 80 open, a user who types `http://` in the browser gets no response at all.
- Security Group names are regional and unique within a VPC — use descriptive names that make the SG's purpose obvious in any context: `SG-LoadBalancer` is immediately clear in an audit or incident review.

---

## Lab 3: Create the Web Server Security Group (Security Group Chaining)

### Steps

1. Navigate to **EC2 → Security Groups → Create security group**.
2. Security group name: `SG-WebServers`.
3. Description: `Allow traffic from Load Balancer only`.
4. VPC: `OluTech-Production-VPC`.
5. **Inbound rules — Add rule:**
   - Type: HTTP | Protocol: TCP | Port: 80 | Source: **Custom** → type `SG-LoadBalancer` and select it from the dropdown.
   - This is the security group chain — port 80 is allowed ONLY from instances in `SG-LoadBalancer`.
6. **Add second inbound rule:**
   - Type: SSH | Protocol: TCP | Port: 22 | Source: **My IP** or enter `x.x.x.x/32` from whatismyip.com.
7. Outbound rules: keep default.
8. Click **Create security group**.

### What I Observed

The source selection for the first rule is where the chaining happens. When I clicked the Source field and started typing the SG name, the console showed a dropdown with matching Security Group IDs and names. Selecting `SG-LoadBalancer` here means the rule is: "allow TCP port 80 from any instance that is a member of `SG-LoadBalancer`." Not "from any IP in the ALB's subnet." Not "from `0.0.0.0/0`." Only from instances carrying that specific Security Group.

The SSH rule on port 22 sourced to `/32` (my exact IP) is the minimal-access pattern for Bastion-style SSH: one specific IP, one specific port, nothing else. In production, this would be the office static IP or a VPN endpoint IP — not a personal home IP.

### What I Learned

- Security Group chaining is the professional standard. CIDR-based rules are for public-facing SGs only.
- When `SG-WebServers` references `SG-LoadBalancer`, new ALB instances automatically get access when they join `SG-LoadBalancer` — no rule update needed. If the ALB subnet CIDR were the source instead, any resource in that subnet could reach web servers regardless of intent.
- The `/32` CIDR means exactly one IP address. Any other prefix (e.g. `/24`) would allow an entire subnet to SSH. Always use `/32` for individual IP-based SSH access restrictions.

---

## Lab 4: Create the Database Security Group

### Steps

1. Navigate to **EC2 → Security Groups → Create security group**.
2. Security group name: `SG-Database`.
3. Description: `Allow MySQL from Web Servers only. No internet access. No SSH.`
4. VPC: `OluTech-Production-VPC`.
5. **Inbound rules — Add rule:**
   - Type: MySQL/Aurora | Protocol: TCP | Port: 3306 | Source: **Custom** → select `SG-WebServers`.
6. No SSH rule. No internet rule. No other inbound rules.
7. Outbound rules: keep default.
8. Click **Create security group**.

### What I Observed

`SG-Database` has one inbound rule and that is correct. Only the EC2 web servers can reach the database, only on the MySQL port, and only because they carry `SG-WebServers`. The database has no SSH rule because no engineer should ever SSH directly to a database instance — access to the database is via the application or via a database admin tool routed through the Bastion Host. Having no SSH rule is not an oversight; it is the intended design.

The outbound default (`All traffic → 0.0.0.0/0`) on `SG-Database` looks permissive but is irrelevant — the database subnet's isolated route table has no `0.0.0.0/0` entry. Even if the Security Group allows outbound to the internet, there is no route to take. The defence is layered: route table blocks the path at the network layer, Security Group controls access at the instance layer.

### What I Learned

- The database Security Group having a permissive outbound rule is fine because the route table enforces isolation at the network layer. Defence in depth means both layers are present, but the route table is the structural control.
- No SSH to RDS is correct and intentional. RDS is a managed service — there is no instance OS to SSH into for MySQL/PostgreSQL RDS anyway. For Aurora or self-managed EC2 databases, access goes through the Bastion Host.
- The three-SG chain in summary: ALB accepts internet → Web tier accepts from ALB SG → DB accepts from Web SG. Each hop narrows the source to the exact preceding tier.

---

## Lab 5: Document the Security Group Architecture

### Steps

1. Create the documentation table in notes:

| SG Name | Inbound Rules | Outbound Rules | Connected To |
|---|---|---|---|
| `SG-LoadBalancer` | TCP 443 from `0.0.0.0/0`; TCP 80 from `0.0.0.0/0` | All traffic | Internet |
| `SG-WebServers` | TCP 80 from `SG-LoadBalancer`; TCP 22 from `x.x.x.x/32` | All traffic | `SG-LoadBalancer` |
| `SG-Database` | TCP 3306 from `SG-WebServers` | All traffic | `SG-WebServers` |

2. Draw the chaining diagram in Excalidraw:
   - Arrow: `Internet → SG-LoadBalancer` labelled `TCP 443 / TCP 80`
   - Arrow: `SG-LoadBalancer → SG-WebServers` labelled `TCP 80 (SG ref)`
   - Arrow: `SG-WebServers → SG-Database` labelled `TCP 3306 (SG ref)`
3. Add a red X on any arrow that should NOT exist (e.g. `Internet → SG-Database` — blocked).
4. Export PNG.

### What I Observed

Drawing the chain diagram made the security model immediately visible. Each arrow is a deliberate trust relationship. The absence of an arrow from the internet to `SG-WebServers` or `SG-Database` is as important as the arrows that do exist. In an architecture review, this diagram is the answer to "how is your database protected from the internet?" — you can point to the diagram and show there is no path from the internet to `SG-Database` at either the routing layer or the Security Group layer.

### What I Learned

- Architecture diagrams with explicit Security Group labels are portfolio artefacts. The labels on the arrows (`TCP 3306 from SG-WebServers`) show you understand the mechanism, not just the topology.
- The documentation table is also useful for IAM and compliance reviews — it answers "who can access the database" with a specific, auditable answer.

---

## Bonus Lab: Network ACL for Private Subnet with Explicit Protocol Blocks

### Steps

1. Navigate to **VPC → Network ACLs → Create network ACL**.
2. Name: `NACL-Private-Subnet`.
3. VPC: `OluTech-Production-VPC`.
4. **Inbound rules — Edit inbound rules:**
   - Rule 100: Type: Custom TCP | Port: 3306 | Source: `10.0.0.0/16` | Action: **ALLOW**
   - Rule 200: Type: Custom TCP | Port: 23 (Telnet) | Source: `0.0.0.0/0` | Action: **DENY**
   - Rule 300: Type: Custom TCP | Port: 21 (FTP) | Source: `0.0.0.0/0` | Action: **DENY**
   - Rule 32767: Type: All Traffic | Source: `0.0.0.0/0` | Action: **DENY** (implicit catch-all)
5. Test evaluation order: move the DENY rule for Telnet to Rule 50 (above the ALLOW). Observe: a MySQL connection from within the VPC should still work because port 3306 is different from port 23. The DENY on port 23 does not affect port 3306.
6. Test: temporarily move the ALLOW (port 3306) to Rule 300 and the DENY (all) to Rule 100. Confirm: MySQL traffic is now blocked because the DENY at Rule 100 fires before the ALLOW at Rule 300 for all traffic.
7. Restore to the correct order before completing.
8. Associate the NACL with `Private-Subnet-App-AZ-A`.

### What I Observed

The evaluation order test was the most instructive part. When I moved the catch-all DENY to Rule 100 and the MySQL ALLOW to Rule 300, MySQL traffic stopped immediately — the `DENY ALL` at Rule 100 matched first and the ALLOW at Rule 300 was never reached. This is the behaviour that trips people up: unlike Security Groups (which evaluate all rules and pick most permissive), NACLs stop at the first matching rule. One misplaced rule number can silently block legitimate traffic.

The Telnet and FTP deny rules at higher numbers after the MySQL allow demonstrate that specific protocol blocks can coexist with application-level allows without interfering. Rule 100 allows MySQL from the VPC CIDR. Rule 200 blocks Telnet. Rule 300 blocks FTP. Each rule targets a different port — no conflict.

### What I Learned

- NACL rule numbering is everything. The same allow and deny rules produce completely different outcomes if the numbers are swapped.
- NACL explicit deny is the tool for blocking specific protocols at the subnet level — Telnet (port 23), FTP (port 21), RDP from the internet (port 3389), known malicious IPs. These blocks belong in NACLs, not Security Groups (which cannot deny).
- Always test NACL rule order changes before deploying — a wrong number can block all traffic to a subnet silently. Use VPC Reachability Analyzer or VPC Flow Logs to verify traffic is flowing as expected after any NACL change.
