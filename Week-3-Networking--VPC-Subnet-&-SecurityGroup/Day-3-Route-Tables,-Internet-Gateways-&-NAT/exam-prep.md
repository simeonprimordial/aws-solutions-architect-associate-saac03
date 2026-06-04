# Exam Prep — Week 3 Day 3: Route Tables, Internet Gateways & NAT

## SAA-C03 Context

Route table design is tested across Domain 1 (Resilient Architectures, ~30%) and Domain 2 (Secure Architectures, ~26%) — often in the same multi-part scenario question. Domain 1 tests NAT Gateway HA patterns (one per AZ), the cost implications of cross-AZ routing, and the three conditions required for internet connectivity. Domain 2 tests route table as a compliance control — specifically whether a data tier is truly isolated at the network layer. The exam will also present the route priority rules (longest prefix match, local route immutability, Main Route Table fallback) as scenario distractors where one wrong route table decision invalidates the entire architecture. Route tables appear in every multi-service networking scenario from this point forward.

---

## Exam Traps — Deep Explanations

### Trap 1: NAT Gateway is not free — and HA requires one per AZ

The exam presents NAT Gateway as the default answer for private subnet outbound internet access, which is correct. But cost-optimisation and HA scenarios add nuance. The billing is `$0.045/hour` regardless of traffic plus `$0.045/GB` processed. One NAT Gateway running idle for a month costs `$32.85`. For HA across two AZs: two NAT Gateways = `$65.70/month` before any data charges. Cross-AZ NAT traffic (private subnet in AZ-1b routing through a NAT GW in AZ-1a) incurs an additional `$0.01/GB` data transfer charge plus the latency of crossing AZ boundaries. The correct HA pattern: one NAT Gateway per AZ, each AZ's private subnets route to their own NAT GW. Exam questions on cost optimisation for low-traffic workloads point to NAT Instance as the cheaper alternative.

### Trap 2: Three conditions for internet connectivity — all required simultaneously

The exam will describe a scenario where EC2 instances cannot reach the internet despite the engineer believing the VPC is correctly configured. The answer is always one of three missing conditions: (1) IGW exists but is not attached to the VPC, (2) IGW is attached but the subnet's route table has no `0.0.0.0/0 → igw-xxx` entry, or (3) IGW is attached and route exists but the EC2 has no public or Elastic IP. All three must be true simultaneously. When connectivity debugging is the question, trace these three in order before looking at Security Groups.

### Trap 3: NAT Gateway protocol limitation — GRE and IPsec fail silently

This appears in advanced scenario questions. A team migrates a workload that uses VPN tunnels (PPTP/L2TP use GRE) or raw IPsec to a private subnet with NAT Gateway. The VPN tunnels fail. The NAT Gateway is not logging errors — it simply drops the non-TCP/UDP/ICMP traffic silently. The fix is not adding more NAT Gateways. The fix is a NAT Instance (an EC2 with source/destination check disabled and IP forwarding enabled) which handles all IP protocols. The exam distractors will suggest adding a second NAT Gateway, adding NACLs, or changing Security Groups — all wrong. Protocol support is the issue.

### Trap 4: Unassociated subnets fall to Main Route Table — not to "no routing"

The exam presents a scenario where a newly created subnet has unexpected internet access. The cause: the Main Route Table has an IGW route, and the new subnet was never explicitly associated with a custom table. This is the OPay finding: someone added `0.0.0.0/0 → IGW` to the Main Route Table temporarily. Months later, unassociated subnets are publicly routable. The fix: remove the IGW route from the Main Route Table and explicitly associate every subnet with a named custom table. Best practice: keep Main as local-only so that any forgotten association is a safe default rather than a security incident.

### Trap 5: The local route always wins for VPC-internal traffic

Traffic between any two subnets within the same VPC always takes the `local` route — `10.0.0.0/16 → local` is more specific than `0.0.0.0/0 → igw-xxx` for any destination within `10.0.0.0/16`. The IGW never handles intra-VPC routing. This is tested in scenarios where candidates think traffic from the public subnet to the private subnet goes through the IGW. It does not. It takes the `local` route directly. Similarly, traffic from a private subnet EC2 to an RDS in the isolated subnet never touches the NAT Gateway — `10.0.21.x` is within `10.0.0.0/16` and matches the `local` route before `0.0.0.0/0`.

---

## Architecture Decision Table

| Scenario | Correct Solution |
|---|---|
| Private subnet EC2 needs to download patches | NAT Gateway in public subnet + `0.0.0.0/0 → nat-xxx` in private route table |
| Private subnet needs outbound HA across 2 AZs | One NAT Gateway per AZ, separate per-AZ private route tables |
| Private subnet needs outbound, cost is critical, low traffic | NAT Instance (EC2) — ~`$3.80/month` vs NAT GW `$32.85/month` |
| Private subnet workload uses GRE/IPsec protocols | NAT Instance with IP forwarding enabled — NAT Gateway does not support GRE/IPsec |
| Private IPv6 resources need outbound-only internet access | Egress-Only Internet Gateway — not NAT Gateway (which is IPv4 only) |
| EC2 can't reach internet despite IGW present | Check all three: IGW attached + route table entry + public/Elastic IP |
| New subnet accidentally has internet access | Check Main Route Table — likely has an IGW route that should be removed |
| Data tier must have zero internet path (compliance) | Isolated route table with `local` only — no `0.0.0.0/0` entry of any kind |
| ALB must only accept traffic from a specific EC2 | Reference the EC2's SG as source on the ALB SG — not a CIDR |
| EC2 must only accept traffic from the ALB | Reference the ALB's SG (`SG-LoadBalancer`) as source on the EC2 SG |
| Block Telnet (port 23) from all instances in a subnet | NACL DENY rule for port 23 — Security Groups cannot deny |
| One AZ outage should not break other AZ's NAT routing | Separate NAT GW per AZ + per-AZ private route tables |

---

## Practice Question

**A solutions architect is reviewing the VPC configuration for a Lagos-based fintech. The VPC has three subnets: a public subnet hosting an Application Load Balancer, a private subnet hosting EC2 application servers, and a private subnet hosting an RDS database. The EC2 application servers must be able to download security patches from the internet. The RDS database must have NO internet connectivity in any direction. Currently, all three subnets are associated with the Main Route Table which has the entry: `0.0.0.0/0 → igw-xxxxxxxx`. Which set of actions will correctly implement the required routing? (Select THREE)**

**A.** Create a NAT Gateway in the public subnet. Create a custom route table with routes `10.0.0.0/16 → local` and `0.0.0.0/0 → NAT Gateway`. Associate this route table with the private EC2 subnet.

**B.** Create a custom route table with only the local VPC route (`10.0.0.0/16 → local`). Associate this route table with the RDS subnet.

**C.** Remove the `0.0.0.0/0 → igw-xxxxxxxx` route from the Main Route Table. Create a custom public route table with `10.0.0.0/16 → local` and `0.0.0.0/0 → igw-xxxxxxxx`. Associate it with the ALB subnet.

**D.** Add a Security Group rule to the RDS instance denying all inbound traffic from `0.0.0.0/0`. Keep the existing Main Route Table association on the RDS subnet.

**E.** Create an Egress-Only Internet Gateway and associate it with the RDS subnet to allow outbound-only access for database updates.

---

**Correct Answers: A, B, and C**

**A — Correct.** The EC2 app servers need outbound internet access for patch downloads without being directly reachable from the internet. NAT Gateway in the public subnet provides exactly this: it enables outbound connections while blocking all unsolicited inbound. The private route table with `0.0.0.0/0 → nat-xxx` routes outbound traffic to the NAT Gateway. The NAT Gateway in the public subnet routes it to the internet via the IGW route in `public-rt`. EC2 instances in this subnet have no public IPs and are not directly internet-reachable.

**B — Correct.** The RDS database must have no internet connectivity in any direction — including outbound. An isolated route table with only the `local` VPC route entry achieves this. No `0.0.0.0/0` IGW route. No `0.0.0.0/0` NAT Gateway route. The database subnet can only route traffic to other resources within the VPC via the `local` route. This satisfies the compliance requirement at the network architecture layer — not just the application layer.

**C — Correct.** The problem in the current state is that the Main Route Table has an IGW route, which means all three subnets currently have internet access. To fix this: remove the IGW route from the Main Route Table (restoring it to local-only), then create a dedicated `public-rt` with the IGW route and associate it only with the ALB subnet. This implements correct routing separation. The Main Route Table with local-only also ensures any future new subnet is safe by default.

**D — Wrong.** A Security Group is an application-layer operational control. The RDS subnet still has the `0.0.0.0/0 → IGW` route via the Main Route Table. Even with the Security Group denying all inbound, the RDS subnet has an architectural internet path. Any IAM user with `ec2:AuthorizeSecurityGroupIngress` can open the Security Group at any time. The compliance requirement is network-layer enforcement — route table isolation. A Security Group alone never satisfies this.

**E — Wrong.** An Egress-Only Internet Gateway handles IPv6 outbound-only traffic. This question involves IPv4. Furthermore, the requirement explicitly states no internet connectivity in ANY direction — including outbound. Even if an Egress-Only IGW were applicable to IPv4 (it is not), it would still create an internet path, violating the requirement. The RDS subnet must have no `0.0.0.0/0` entry of any type.

---

## Quick-Recall Test

**Q1: A packet destined for `10.0.11.5` is evaluated against a route table containing `10.0.11.0/24 → local`, `10.0.0.0/16 → local`, and `0.0.0.0/0 → igw-xxx`. Which route wins?**
`10.0.11.0/24 → local` — the most specific (longest prefix) match. `/24` beats `/16` beats `/0`.

**Q2: Can you delete the local VPC route from a route table?**
No. The `10.0.0.0/16 → local` route is permanent and immutable. AWS does not allow it to be deleted or modified.

**Q3: A NAT Gateway costs `$0.045/hour` and processes 200GB of data per month. What is the monthly cost?**
Hourly: `$0.045 × 720 = $32.40`. Data: `200 × $0.045 = $9.00`. Total: `$41.40/month`.

**Q4: A startup wants to save money on NAT Gateway. What is the alternative?**
NAT Instance — an EC2 instance with source/destination check disabled and IP forwarding enabled. Costs the EC2 hourly rate (e.g. `t3.nano` ≈ `$3.80/month`). Trade-off: you manage patching, HA, and monitoring. For low-traffic workloads the cost savings are significant.

**Q5: A private subnet workload uses PPTP VPN tunnels (GRE protocol). Traffic is failing through the NAT Gateway. What is the fix?**
Replace the NAT Gateway with a NAT Instance (EC2 with IP forwarding enabled). NAT Gateway supports only TCP, UDP, and ICMP. GRE is not supported and fails silently. The fix is a NAT Instance, not a NAT Gateway configuration change.

**Q6: A new subnet was created but never explicitly associated with a route table. What route table does it use?**
The Main Route Table for the VPC. This is the default for any subnet without an explicit custom association. Keep the Main Route Table local-only to ensure this default is safe.

**Q7: What is the most secure way to allow an ALB to communicate with EC2 instances in the web tier?**
Set the inbound rule on the EC2's Security Group to allow the relevant port (e.g. TCP 80) with the source being the ALB's Security Group ID — not `0.0.0.0/0`, not a CIDR. This is Security Group chaining.

**Q8: Traffic between a public subnet EC2 (`10.0.1.5`) and a private subnet RDS (`10.0.21.8`) — does it go through the IGW?**
No. Both `10.0.1.5` and `10.0.21.8` fall within the VPC CIDR `10.0.0.0/16`. The `local` route (`10.0.0.0/16 → local`) is more specific than `0.0.0.0/0 → igw-xxx` and wins for this traffic. All intra-VPC traffic takes the local path — the IGW is never involved.
