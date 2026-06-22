# Security Groups & NACLs

AWS gives every VPC two completely independent mechanisms for controlling traffic. They operate at different layers, behave differently, and are both necessary for a production-grade security setup. The Security Group is the bouncer at the door of each individual resource — it checks every connection before letting it through to a specific EC2 instance, RDS database, or Lambda function. The NACL is security at the gate of an entire neighbourhood — it checks traffic entering and leaving the whole subnet before any individual resource ever sees a packet. Understanding exactly how these two layers interact, and where each one fails on its own, is the core security knowledge for the SAA-C03 exam and for any cloud engineering role.

---

## Security Groups

### What They Are

A Security Group is a virtual stateful firewall applied at the Elastic Network Interface (ENI) level — the network card attached to individual AWS resources. Every EC2 instance, RDS database instance, Lambda with VPC attachment, and load balancer has at least one Security Group attached. One Security Group can be attached to multiple resources. One resource can have up to five Security Groups simultaneously.

### Stateful Behaviour

Security Groups are stateful. This means they track connection state. If an inbound rule allows a connection, the return traffic for that connection is automatically allowed — you do not need a matching outbound rule. The Security Group remembers that the connection was established and permits the response to flow back.

**Example:**
- Inbound rule: Allow TCP 443 from `0.0.0.0/0`.
- A user connects via HTTPS. The request arrives.
- The server responds. The response traffic is automatically allowed outbound.
- No outbound rule for port 443 is needed. The Security Group handles it.

This is the fundamental difference from NACLs. With Security Groups, you only write rules for the traffic you want to allow in. You never write rules for the responses.

### Rule Anatomy

Every Security Group rule has:
- **Type** — a named protocol/port combination (SSH, HTTPS, MySQL/Aurora, Custom TCP, etc.)
- **Protocol** — TCP, UDP, ICMP, or All
- **Port range** — single port or range
- **Source (inbound) or Destination (outbound)** — a CIDR block, a Security Group ID, or a prefix list
- **Action** — always Allow (Security Groups have no Deny)

**Example Security Group rules for a 3-tier application:**

```
SG-LoadBalancer — Inbound
Type     Protocol   Port    Source          Action
HTTPS    TCP        443     0.0.0.0/0       ALLOW
HTTP     TCP        80      0.0.0.0/0       ALLOW

SG-AppServers — Inbound
Type     Protocol   Port    Source              Action
Custom   TCP        8080    sg-loadbalancer     ALLOW   ← SG reference
SSH      TCP        22      sg-bastion          ALLOW   ← SG reference

SG-Database — Inbound
Type          Protocol   Port    Source          Action
PostgreSQL    TCP        5432    sg-appservers   ALLOW   ← SG reference
```

### Security Group Defaults

| Property | Default Value |
|---|---|
| Inbound | Deny ALL — nothing enters until you add allow rules |
| Outbound | Allow ALL — all outbound traffic is permitted by default |
| Rule type | Allow only — no explicit deny exists |
| Evaluation | All rules evaluated simultaneously — most permissive wins |

> ⚠️ **Exam Tip:** Security Groups have NO explicit deny. If you need to block a specific IP address — a known malicious scanner, a former employee's home IP — you cannot do it in a Security Group. You must use a NACL Deny rule on the subnet.

### Security Group Referencing

Instead of using a CIDR block as the source/destination, you can reference another Security Group by its ID. This is the professional production pattern.

**Why SG referencing is superior to CIDR-based rules:**

If `SG-Database` has an inbound rule `TCP 5432 from sg-appservers`, only instances that are members of `sg-appservers` can reach the database — regardless of their IP address. When Auto Scaling adds new EC2 instances to the app tier, they automatically receive `sg-appservers` and immediately have database access. No rule update. No IP tracking. When old instances are terminated, their IP addresses become irrelevant — the membership in `sg-appservers` is what matters, and terminated instances lose membership automatically.

Compare to a CIDR rule: `TCP 5432 from 10.0.11.0/24`. Any resource in that subnet — including monitoring agents, test instances, or a compromised developer VPN endpoint — can query the production database. The CIDR is too broad.

**The Interswitch finding:** Their RDS Security Group had `TCP 5432 from 10.0.0.0/16` (the entire VPC CIDR). Any EC2 in any subnet, any developer laptop connected via VPN, any monitoring server — all could query the production payment database directly. Fix: replace with `TCP 5432 from sg-appservers`. Least privilege, automatically maintained.

### One Security Group on Many Resources

Attaching the same Security Group to multiple EC2 instances means they all share the same rules. When you add or modify a rule, all attached resources get the update immediately — without touching each instance individually. This is the operational scalability of Security Groups: manage rules once, apply to a fleet.

---

## Network ACLs (NACLs)

### What They Are

A Network ACL is a stateless firewall applied at the subnet boundary. Every subnet has exactly one NACL associated with it. The NACL evaluates ALL traffic entering and leaving the subnet — every packet in every connection, independently. One NACL can be associated with multiple subnets. A subnet can only have one NACL.

### Stateless Behaviour — The Critical Difference

NACLs are stateless. They track no connection state. Every packet is evaluated independently. This has one critical practical consequence: you must write explicit rules for BOTH directions of every traffic flow.

**The ephemeral port problem:**

When a client (EC2 app server, user browser) initiates a TCP connection to a server (RDS database, ALB), the client OS picks a random source port in the **ephemeral port range** (`1024–65535`) for the server's responses to return on. The client connects from its ephemeral port to the server's well-known port (e.g. port 5432 for PostgreSQL). The server responds back to the client's ephemeral port.

A NACL that allows inbound TCP 5432 permits the database connection request. But the response — coming from the RDS instance on port 5432 back to the EC2 app server on its random ephemeral port (e.g. port 54321) — hits the NACL outbound rules. If there is no outbound rule allowing TCP `1024–65535`, the response is silently dropped. The EC2 app server never receives the database response. The connection times out.

**The correct NACL configuration for a data subnet:**

```
NACL Inbound — Private-Data Subnet
Rule    Protocol   Port Range    Source              Action
100     TCP        5432          10.0.11.0/24        ALLOW   ← app subnet CIDR
120     TCP        1024-65535    10.0.11.0/24        ALLOW   ← ephemeral responses IN
*       All        All           0.0.0.0/0           DENY    ← implicit catch-all

NACL Outbound — Private-Data Subnet
Rule    Protocol   Port Range    Destination         Action
100     TCP        1024-65535    10.0.11.0/24        ALLOW   ← responses OUT
*       All        All           0.0.0.0/0           DENY    ← implicit catch-all
```

The outbound Rule 100 allowing TCP `1024–65535` to the app subnet is what enables responses to leave the data subnet and reach the requesting EC2 app server.

> ⚠️ **Exam Tip:** Missing the ephemeral port outbound rule in a NACL is the most common NACL misconfiguration and the most common NACL question type on the SAA-C03. The symptom: requests arrive at the instance (inbound NACL + SG both allow), the instance processes and responds, but the client sees a timeout. The fix is always: add outbound TCP `1024–65535` allow rule.

### Rule Anatomy and Evaluation

Every NACL rule has:
- **Rule number** — determines evaluation order. Lower numbers evaluated first.
- **Protocol** — TCP, UDP, ICMP, All
- **Port range** — single port or range
- **Source/Destination** — CIDR only (NACLs cannot reference Security Group IDs)
- **Action** — Allow or Deny

**Evaluation:** Rules are processed in ascending number order. The **first matching rule wins** and processing stops. Unlike Security Groups (which evaluate all rules simultaneously), a NACL stops at the first match.

**Rule numbering convention:** Use increments of 100 (`100`, `200`, `300`...). This leaves gaps to insert new rules between existing ones without renumbering.

**A DENY rule with a lower number than an ALLOW rule for the same traffic will always block that traffic:**

```
NACL Inbound (incorrect order)
Rule 90:  DENY  TCP 443 from 0.0.0.0/0     ← evaluated first
Rule 100: ALLOW TCP 443 from 0.0.0.0/0     ← never reached

Result: ALL HTTPS traffic is blocked.
```

```
NACL Inbound (correct order)
Rule 100: ALLOW TCP 443 from 0.0.0.0/0     ← evaluated first
Rule 200: DENY  TCP 22 from 0.0.0.0/0      ← blocks Telnet globally

Result: HTTPS allowed, Telnet blocked.
```

### Default NACL vs Custom NACL

| Property | Default NACL | Custom NACL |
|---|---|---|
| Created by | Automatically at VPC creation | You, explicitly |
| Initial inbound | Allow ALL | Deny ALL (`*` rule only) |
| Initial outbound | Allow ALL | Deny ALL (`*` rule only) |
| What it does | Nothing — allows everything | Nothing — blocks everything |
| Risk | False sense of security | Immediate traffic disruption if no rules added |

The default NACL is a non-control — it allows all traffic in both directions. It is not a security measure. Creating a custom NACL and associating it with a subnet without adding rules immediately blocks ALL traffic to and from that subnet. This is the most common cause of "everything stopped working after I added a custom NACL."

> ⚠️ **Exam Tip:** When you create a custom NACL, you start from zero. The default NACL allows all. They look identical in the console list until you check the rules. Always verify which NACL is applied to a subnet when troubleshooting sudden traffic failures after a security configuration change.

---

## Security Groups vs NACLs — Full Comparison

| Feature | Security Group | Network ACL (NACL) |
|---|---|---|
| Operates at | EC2 instance / ENI | Subnet boundary |
| State | **STATEFUL** — return traffic automatic | **STATELESS** — both directions explicit |
| Rule types | **Allow only** — no explicit deny | **Allow AND Deny** |
| Rule evaluation | ALL rules simultaneously — most permissive wins | **Number order — first match wins** |
| Default (VPC auto-created) | Default SG: deny all inbound, allow all outbound | Default NACL: allow ALL inbound AND outbound |
| Custom (you create) | New SG: deny all inbound, allow all outbound | New NACL: deny ALL (only `*` rule) |
| Ephemeral ports | Not needed — stateful handles return | **REQUIRED** — explicit outbound `1024–65535` |
| Source/destination | CIDR or **Security Group ID** | CIDR only |
| Scope | One or more specific instances | Entire subnet — all resources |
| Explicit deny | **NO** — use NACL to block specific IPs | **YES** — use for IP blocking at subnet |
| Best use | Primary security layer for all resources | Secondary defence — subnet-wide IP/protocol blocks |

---

## Traffic Flow Through Both Layers

**Full path for a request from internet user to RDS PostgreSQL database:**

```
1. Internet → NACL (public subnet boundary, inbound)
   Checks: Rule 100 ALLOW TCP 443 from 0.0.0.0/0 ✓
   
2. NACL passes → Security Group (ALB instance, inbound)
   Checks: ALLOW TCP 443 from 0.0.0.0/0 ✓ (stateful — response auto-allowed)
   
3. ALB processes → sends to EC2 app server
   Traffic is VPC-internal — local route, no subnet boundary crossing
   NACL on private-app subnet inbound: ALLOW TCP 8080 from public CIDR ✓
   EC2 Security Group inbound: ALLOW TCP 8080 from SG-LoadBalancer ✓
   
4. EC2 app server → RDS PostgreSQL
   NACL on private-data subnet inbound: ALLOW TCP 5432 from app CIDR ✓
   RDS Security Group inbound: ALLOW TCP 5432 from sg-appservers ✓
   
5. RDS responds → back to EC2 app server (stateless NACL)
   NACL outbound on private-data: ALLOW TCP 1024-65535 to app CIDR ✓
   (The response goes to the EC2's ephemeral source port)
   EC2 Security Group outbound: auto-allowed (stateful) ✓
```

Any REJECT at any layer silently drops the packet. The most common failure point: missing the outbound ephemeral port rule at step 5.

---

## Bastion Host Pattern

A Bastion Host is a hardened EC2 instance in a public subnet whose sole purpose is to provide secure SSH access to private subnet instances. It is the controlled entry point for administrative access.

**Configuration:**
- EC2 instance in a **public subnet** with a public or Elastic IP
- Security Group (`SG-Bastion`): inbound SSH (port 22) from specific IP(s) only — office static IP, `/32`
- All private instances: Security Group allows SSH (port 22) from `SG-Bastion` only — not from `0.0.0.0/0`, not from a CIDR

**Connection pattern:**
```
Engineer's laptop → (SSH on port 22) → Bastion Host (public IP)
Bastion Host → (SSH on port 22, private network) → Private-App-Server (private IP only)
```

The private instance has no public IP, no internet route, and accepts SSH only from the Bastion's Security Group. An attacker scanning the internet cannot reach it. An attacker who compromises the bastion still needs valid SSH keys for the private instances.

**The Interswitch finding:** SSH port 22 was open to `0.0.0.0/0` on all EC2 instances. A brute-force scanner made 14,000 attempts in 48 hours. The fix: remove `0.0.0.0/0` SSH. Add SSH from `sg-bastion` only. Attackers cannot reach any internal instance without first compromising the Bastion.

### AWS Systems Manager Session Manager — The Modern Replacement

Session Manager eliminates the Bastion Host entirely. Advantages:

1. **No open port 22.** Port 22 is never opened on any instance — not on the Bastion, not on any private instance. The attack surface is zero for SSH-based attacks.
2. **No key management.** No `.pem` files to distribute, rotate, or protect. Access is controlled by IAM permissions — the same identity system managing all other AWS access.
3. **Full audit trail.** Every session is logged to CloudTrail and optionally to S3 and CloudWatch Logs. You can see exactly which IAM user ran which commands on which instance, with timestamps — evidence that meets PCI-DSS and CBN audit requirements.

The trade-off: Session Manager requires the SSM Agent running on the instance and an IAM instance profile with `AmazonSSMManagedInstanceCore`. It does not work if the instance has no internet access and no VPC Endpoint for SSM — for fully isolated instances, a VPC Interface Endpoint for Systems Manager is required.

---

## Common Exam Traps

**Trap 1 — Security Groups have no explicit deny.** To block a specific IP at the subnet level, use a NACL Deny rule. Security Groups work on implicit deny — anything not allowed is blocked — but you cannot write a specific "deny this IP" rule.

**Trap 2 — NACL rule order is evaluation order.** A DENY at rule 90 blocks traffic even if ALLOW exists at rule 100. The allow is never evaluated. Always put specific ALLOW rules at lower numbers than any broad DENY rules.

**Trap 3 — Custom NACLs start with DENY ALL.** Associating a fresh custom NACL with a subnet immediately blocks all traffic. You must add all allow rules before associating, or accept a brief total outage. The default NACL (which the subnet was using before) allowed everything.

**Trap 4 — Ephemeral ports `1024–65535` are mandatory in NACL rules.** Both inbound (for responses arriving back from external services) and outbound (for responses leaving to clients). The exact range varies by OS — Linux uses `32768–60999`, Windows uses `49152–65535` — but `1024–65535` covers all cases and is the standard exam answer.

**Trap 5 — Always check BOTH layers when troubleshooting.** A packet that clears the NACL can still be blocked by the Security Group. A packet that clears the Security Group can still be blocked by a NACL rule on return. Use VPC Flow Logs: REJECT at subnet level = NACL. REJECT at ENI level = Security Group.
