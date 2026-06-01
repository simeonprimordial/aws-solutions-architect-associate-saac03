# Exam Prep — Week 3 Day 2: Subnets — Public vs Private

## SAA-C03 Context

Subnet design appears in Domain 1 (Design Resilient Architectures, ~30%) and Domain 2 (Design Secure Architectures, ~26%). Domain 1 tests multi-AZ subnet placement for high availability — specifically which resources go in which tier and why. Domain 2 tests the security implications of routing decisions — route table vs Security Group as an isolation control, data tier isolation for compliance, and the Default VPC's lack of network segmentation. Subnet questions appear in almost every multi-service scenario on the exam because every service that runs inside a VPC requires a subnet placement decision. Getting the placement wrong in the scenario answer is an automatic fail on that question.

---

## Exam Traps — Deep Explanations

### Trap 1: Route table determines publicness — never naming or Auto-Assign

The exam's favourite subnet trap. A subnet can be named `Public-Subnet-Production-AZ1`, have Auto-Assign Public IP enabled, and have EC2 instances with public IPs — and still be a private subnet if the route table contains only the `local` route. The exam will describe this exact scenario and ask "why can the EC2 instance not reach the internet?" The answer is always: the route table has no `0.0.0.0/0 → IGW` entry. No other explanation is correct. Routing determines access.

### Trap 2: Subnets are single-AZ — VPCs are Regional

These appear as opposite distractors. "A subnet spanning multiple AZs" is always wrong — subnets are created in one AZ and cannot be moved or extended. "A VPC spanning multiple AZs" is always correct — a VPC is a Regional resource that automatically covers all AZs in the Region. The distinction matters for high-availability design: you create separate subnets per AZ and deploy resources across them. The exam will ask about deploying a resilient application — the answer always involves multiple subnets in multiple AZs, not a single subnet trying to span them.

### Trap 3: A /24 subnet has 251 usable IPs — not 256. AWS reserves 5.

The SAA-C03 tests this with specific numbers. The question pattern: "A company needs X EC2 instances in a single subnet. Which subnet size is sufficient?" The trap is choosing the mathematically smallest subnet that fits the total count, forgetting to subtract 5. A company needing 251 instances cannot use a `/24` (251 usable exactly = no margin). A company needing 250 instances CAN use a `/24` (251 usable, one spare). If the question says "needs 252 or more" — the answer is `/23` (507 usable). Always subtract 5 first.

**Reference table for the exam:**

| CIDR | Total | Usable (−5) |
|---|---|---|
| `/28` | 16 | 11 |
| `/27` | 32 | 27 |
| `/26` | 64 | 59 |
| `/25` | 128 | 123 |
| `/24` | 256 | 251 |
| `/23` | 512 | 507 |
| `/22` | 1,024 | 1,019 |

### Trap 4: Default VPC subnets are all public — no exceptions

The exam will describe a scenario where an engineer launches EC2 instances "in the default VPC for testing" and then asks what network-level risk this creates. The answer: all Default VPC subnets are public — they have `0.0.0.0/0 → IGW` in their route table and Auto-Assign Public IP enabled. Every EC2 instance launched there automatically gets a public IP and is reachable from the internet unless a Security Group blocks it. Candidates who assume the Default VPC has any network segmentation or private subnets are wrong.

### Trap 5: Security Group alone cannot satisfy compliance isolation requirements

An exam question will describe a compliance requirement for "data tier to have no internet access, enforceable at the network layer." One distractor option will be: "configure a Security Group on RDS that denies all traffic from `0.0.0.0/0`." This is wrong for two reasons: (1) a Security Group is an operational control — any IAM user with the right permissions can modify it, and it leaves no structural evidence of isolation, and (2) the subnet may still have a route to the internet. The correct answer is the route table: a data subnet with only the `local` VPC route. No `0.0.0.0/0` entry of any kind. This is architectural isolation, provable and tamper-evident.

---

## Architecture Decision Table

| Scenario | Correct Subnet / Routing Decision |
|---|---|
| Application Load Balancer (receives internet traffic) | Public subnet with IGW route |
| EC2 web tier servers (receive traffic from ALB only) | Private-App subnet with NAT GW route |
| EC2 app servers (call external APIs, no inbound from internet) | Private-App subnet with NAT GW route |
| RDS database (no internet access required) | Private-Data subnet with local route ONLY |
| Bastion Host (SSH entry point for engineers) | Public subnet OR Management `/28` subnet with IGW route |
| NAT Gateway (allows private instances to reach internet) | MUST be in a public subnet (needs IGW route itself) |
| Company needs 300 instances in one subnet | `/23` (507 usable) — `/24` only gives 251 |
| Company needs 10 instances in one subnet | `/28` (11 usable) — smallest viable subnet |
| Data tier must prove no internet path to auditor | Route table with local-only — no `0.0.0.0/0` entry |
| New subnet added to VPC — what route table does it use? | Main route table by default — keep main route table private |
| Multi-AZ HA for an application | Separate subnets per AZ per tier — minimum 6 subnets for 3-tier 2-AZ |

---

## Practice Question

**A solutions architect is designing a three-tier application on AWS for a Lagos financial services company. The web tier serves public user traffic, the application tier processes business logic, and the data tier stores customer financial records. The company's compliance team requires that the data tier must have NO path to the internet, even for outbound traffic, and this must be enforceable at the network layer — not just the application layer. Which combination of configurations meets ALL requirements? (Select TWO)**

**A.** Place the data tier in a private subnet. Configure a Security Group on the RDS instances that denies all inbound and outbound traffic from `0.0.0.0/0`.

**B.** Place the data tier in a private subnet with a route table that contains ONLY the local VPC route (`10.0.0.0/16 → local`). Do not add a NAT Gateway route to this route table.

**C.** Place the web tier EC2 instances and the Application Load Balancer in the same public subnet to simplify the architecture.

**D.** Place the application tier in a private subnet with a NAT Gateway route for outbound-only internet access. Ensure no route to the Internet Gateway exists on this subnet.

**E.** Configure a Network ACL on the data tier subnet that denies all traffic with a source or destination of `0.0.0.0/0`. Leave the route table with a NAT Gateway route.

---

**Correct Answers: B and D**

**A — Wrong.** A Security Group is an operational control, not a network-layer architectural control. The compliance requirement is explicitly for network-layer enforcement. Even if the Security Group correctly blocks all traffic, the subnet may still have a route to the internet via a NAT Gateway — and any IAM user with `ec2:AuthorizeSecurityGroupIngress` can change the Security Group at any time. The route table must have no internet entry. A Security Group alone is never sufficient for this type of regulatory requirement.

**B — Correct.** A route table with only the local VPC entry (`10.0.0.0/16 → local`) creates zero internet path in either direction. No IGW route. No NAT Gateway route. The data subnet can communicate with other resources within the VPC (the app tier, via the `local` route) but has no path to the internet. This is provable at the network architecture layer — open the route table, show the auditor the single `local` entry. This is the route table configuration that satisfies CBN, SEC, and PCI-DSS data isolation requirements.

**C — Wrong.** EC2 web tier instances should never be in a public subnet. The Application Load Balancer belongs in the public subnet — it is the internet-facing entry point. The EC2 web tier instances should be in a private subnet, receiving traffic from the ALB via the ALB's Security Group. Placing EC2 instances in the public subnet exposes them directly to internet traffic even if the ALB is in front of them. The architectural principle is: the ALB absorbs internet exposure, the EC2 instances are shielded behind it.

**D — Correct.** Application tier servers must be able to reach external services — payment networks, card processors, SMS gateways, fraud detection APIs. A NAT Gateway route (`0.0.0.0/0 → nat-xxxxxxxx`) enables this outbound access while blocking all inbound internet connections. There is no IGW route on this subnet — instances cannot be reached from the internet directly. Only the ALB's Security Group can route traffic to them. This is the standard private-app subnet configuration.

**E — Wrong.** A NACL can block traffic, but the underlying route still exists. If the NACL is ever misconfigured — a single rule change — the NAT Gateway route immediately becomes active and provides an outbound internet path. The compliance requirement says "enforceable at the network layer" — this means the route must not exist, not that it is blocked by a NACL. Defence in depth means removing the path entirely from the route table, not relying on a NACL to block a path that could be reopened.

---

## Quick-Recall Test

**Q1: A subnet named `Database-Subnet` has a route table with only `10.0.0.0/16 → local`. Is it public or private?**
Private — specifically isolated. No internet path in either direction. The name is irrelevant.

**Q2: How many usable IPs are in a /27 subnet?**
32 total − 5 reserved = 27 usable.

**Q3: What is the minimum number of subnets needed for a standard 3-tier application with High Availability across 2 AZs?**
6 subnets: 2 public, 2 private-app, 2 private-data.

**Q4: A company needs 260 instances in a single subnet. Which CIDR should they use?**
`/23` — 512 total, 507 usable. `/24` only gives 251 usable, which is less than 260.

**Q5: What is the difference between the `0.0.0.0/0 → igw-xxx` route and the `0.0.0.0/0 → nat-xxx` route?**
The IGW route enables two-way internet communication — inbound and outbound. The NAT Gateway route enables outbound-only internet access for private subnet instances — inbound is blocked because the NAT Gateway rejects all unsolicited inbound connections.

**Q6: Where must a NAT Gateway be placed?**
In a public subnet. The NAT Gateway itself needs an IGW route to reach the internet on behalf of private instances. A NAT Gateway in a private subnet cannot function.

**Q7: A developer argues that a Security Group denying all inbound on an RDS instance in a public subnet satisfies the PCI-DSS isolation requirement. Why is this wrong?**
The subnet has an IGW route — it is architecturally public. A Security Group is an operational control that any authorised IAM user can modify. It provides no structural proof of isolation. PCI-DSS and CBN require network-layer evidence — the route table must have no internet entry. Route table = architecture. Security Group = configuration.

**Q8: What happens to a new subnet created in a VPC if you don't explicitly associate it with a route table?**
It is automatically associated with the VPC's main route table. This is why the main route table should always remain private (local route only) — any new subnet defaults to the main table, so keeping it without an IGW route means new resources are private by default and must be deliberately made public.
