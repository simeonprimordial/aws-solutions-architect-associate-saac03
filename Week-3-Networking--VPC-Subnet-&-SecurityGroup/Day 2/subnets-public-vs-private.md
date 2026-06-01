# Subnets — Public vs Private

The subnet placement decision is the most fundamental architectural choice in any AWS deployment. Every EC2 instance, every RDS database, every ECS task, every Lambda with a VPC attachment — all of them must be placed in a specific subnet. That placement determines whether the resource has a path to the internet, what network-layer security controls apply, which Availability Zone it lives in, and whether it is reachable from outside the VPC at all. Getting this wrong means either exposing internal services unnecessarily or failing a regulatory audit. For Nigerian fintechs operating under CBN and SEC oversight — or any platform handling PCI-DSS cardholder data like Flutterwave — subnet architecture is not an implementation detail. It is the compliance evidence.

---

## Public Subnet

A public subnet is any subnet whose route table contains a `0.0.0.0/0` route pointing to an Internet Gateway. That is the complete definition. Nothing else makes a subnet public.

**What resources belong in public subnets:**
- Application Load Balancer — receives internet traffic and forwards it to the private app tier
- Bastion Host — the only SSH entry point to the environment
- NAT Gateway — lives in the public subnet but serves private subnet instances

**What does NOT belong in public subnets:**
- EC2 app servers
- RDS databases
- ElastiCache clusters
- Any resource that handles cardholder data, PII, or financial records

**Route table for a public subnet:**
```
Destination     Target          Description
10.0.0.0/16     local           VPC-internal traffic stays inside the VPC
0.0.0.0/0       igw-xxxxxxxx    All other traffic → Internet Gateway
```

For a resource in a public subnet to actually communicate with the internet, two things are required: the subnet must have the IGW route AND the resource must have a public IP or Elastic IP. The route opens the path. The IP is the address.

> ⚠️ **Exam Tip:** The exam will describe a subnet with "Public" in the name and Auto-Assign Public IP enabled, then ask why instances in it cannot reach the internet. The answer is always: the route table has no `0.0.0.0/0 → IGW` entry. Naming and Auto-Assign are irrelevant to determining publicness.

---

## Private Subnet

A private subnet has NO direct route to an Internet Gateway. Resources are isolated from inbound internet traffic by architecture — not by Security Group rules. A private subnet may have a NAT Gateway route for outbound-only internet access, or it may have no internet route at all (fully isolated).

**Two types of private subnet:**

**Private-App Subnet** — has outbound internet access via NAT Gateway. App servers can download patches, call external APIs (Paystack, Interswitch card networks, SMS providers). No inbound internet path. Security Group allows traffic only from the ALB's Security Group on the specific application port.

```
Destination     Target          Description
10.0.0.0/16     local           VPC-internal traffic stays inside
0.0.0.0/0       nat-xxxxxxxx    Outbound internet → NAT Gateway (outbound only)
```

**Private-Data (Isolated) Subnet** — has NO internet route of any kind. Not even a NAT Gateway route. The route table contains only the `local` VPC entry. RDS, ElastiCache, and any other data-layer resource lives here. This configuration is what CBN, SEC, and PCI-DSS auditors inspect to verify network-level isolation of financial data.

```
Destination     Target          Description
10.0.0.0/16     local           VPC-internal traffic ONLY. No internet path exists.
```

> ⚠️ **Exam Tip:** The distinction between a private-app subnet (with NAT GW route) and an isolated subnet (local only) is directly tested. If the question says "the data tier must have no internet path even for outbound traffic" — the answer is the isolated route table, not a private subnet with a NAT Gateway.

---

## Auto-Assign Public IP

A subnet-level setting. When enabled, every EC2 instance launched in that subnet automatically receives a public IPv4 address. When disabled, instances receive only a private IP.

**Critical distinctions:**
- Auto-assigned public IP ≠ Elastic IP. Auto-assigned IPs are ephemeral — they change every time the instance stops and restarts. Elastic IPs are static and persist.
- Auto-Assign is **ON by default in the Default VPC**. Every EC2 you launch in the default VPC gets a public IP whether you want it or not.
- Auto-Assign is **OFF by default in custom subnets**. You must explicitly enable it on public subnets where instances need direct public IPs.
- **Never enable Auto-Assign on private or data subnets.** An EC2 instance in a private subnet with a public IP is not internet-accessible (because the route table has no IGW route), but it is a misconfiguration that creates confusion and may violate compliance configuration baselines.

| Subnet Type | Auto-Assign Setting | Reason |
|---|---|---|
| Public (ALB, Bastion) | ON | ALB and Bastion need public IPs |
| Private-App | OFF | No internet exposure required |
| Private-Data | OFF | Strictly no internet access |
| Management (/28) | OFF | Bastion gets Elastic IP, not auto-assigned |

---

## Subnet CIDR Blocks

Every subnet's CIDR must be a non-overlapping subset of the VPC CIDR. If the VPC is `10.0.0.0/16`, valid subnet CIDRs include `10.0.1.0/24`, `10.0.2.0/24`, `10.0.10.0/24` — they all fall within the `/16` range and do not overlap each other.

**Sizing reference:**

| CIDR | Total IPs | AWS Reserved | Usable IPs | Use Case |
|---|---|---|---|---|
| `/16` | 65,536 | — | VPC level | VPC CIDR |
| `/22` | 1,024 | 5 | 1,019 | Large app tier (200+ instances) |
| `/23` | 512 | 5 | 507 | Medium tier |
| `/24` | 256 | 5 | 251 | Standard subnet — most common |
| `/26` | 64 | 5 | 59 | Small subnet |
| `/27` | 32 | 5 | 27 | Minimal subnet |
| `/28` | 16 | 5 | 11 | Bastion/management subnet |

**The 5 reserved IPs** — AWS reserves these in every subnet regardless of size:

| Address | Reserved For |
|---|---|
| `x.x.x.0` | Network address |
| `x.x.x.1` | VPC router |
| `x.x.x.2` | AWS DNS server |
| `x.x.x.3` | Future use (reserved by AWS) |
| `x.x.x.255` | Broadcast address |

> ⚠️ **Exam Tip:** This is directly tested. "A company needs 250 usable IPs in a single subnet. They create a /24. Will this work?" — No. `/24` gives 251 usable, which seems like enough, but the question says "needs 250" — it works mathematically but the exam often phrases the requirement as slightly above the usable count. Know the `/23` answer: 507 usable. Always subtract 5 before comparing against the requirement.

---

## Subnet AZ Binding

Every subnet is created in exactly one Availability Zone. This binding cannot be changed after creation. A subnet cannot span AZs.

This is the mechanism behind multi-AZ resilience. To survive an AZ failure:
- Create separate subnets in each AZ for every tier.
- Deploy redundant resources (EC2 instances, RDS replicas, ALB nodes) across the subnets in different AZs.
- The ALB automatically distributes traffic across healthy AZs.

For a production 3-tier architecture across two AZs: minimum 6 subnets.

| Subnet Name | AZ | CIDR | Tier | Route Table |
|---|---|---|---|---|
| `prod-public-az1a` | `af-south-1a` | `10.0.1.0/24` | Public | `public-rt` |
| `prod-public-az1b` | `af-south-1b` | `10.0.2.0/24` | Public | `public-rt` |
| `prod-app-az1a` | `af-south-1a` | `10.0.11.0/24` | Private-App | `private-app-rt` |
| `prod-app-az1b` | `af-south-1b` | `10.0.12.0/24` | Private-App | `private-app-rt` |
| `prod-data-az1a` | `af-south-1a` | `10.0.21.0/24` | Private-Data | `isolated-rt` |
| `prod-data-az1b` | `af-south-1b` | `10.0.22.0/24` | Private-Data | `isolated-rt` |

**CIDR numbering convention:** Public `10.0.1–9.x`, App `10.0.10–19.x`, Data `10.0.20–29.x`, Management `10.0.100+`. This makes any subnet instantly identifiable from its IP range alone — useful in logs, flow logs, and firewall rules.

> ⚠️ **Exam Tip:** VPC = Regional resource (spans all AZs). Subnet = single AZ (cannot span AZs). These two facts appear in opposite directions as exam distractors. "A subnet spanning multiple AZs" is never correct. "A VPC spanning multiple AZs" is always correct.

---

## Default VPC vs Custom VPC — Subnet Comparison

| Feature | Default VPC Subnets | Custom VPC Subnets |
|---|---|---|
| Created by | AWS automatically at account creation | You, explicitly |
| CIDR | Fixed `172.31.0.0/16`, `/20` subnets | You choose — designed for growth |
| Internet route | `0.0.0.0/0 → IGW` by default — ALL public | No IGW route by default — ALL private |
| Auto-Assign Public IP | Enabled by default | Disabled by default |
| Network segmentation | None — all tiers equally exposed | Full control — three-tier isolation |
| Production use | Never | Always |
| Compliance capability | Cannot prove isolation | Route table is auditable evidence |

The default VPC is a convenience tool for learning and experimentation. Every serious AWS deployment uses a custom VPC. Any Nigerian bank, fintech, or regulated entity using the default VPC would fail a CBN IT examination immediately.

---

## Route Table as Compliance Evidence

This is the critical distinction between architectural controls and operational controls — and it matters in regulatory contexts.

**Security Group (operational control):** A Security Group rule denying all inbound and outbound traffic to `0.0.0.0/0` prevents traffic from reaching specific instances. But the subnet still has a route to the internet. Any authorised IAM user with `ec2:AuthorizeSecurityGroupIngress` permissions can open that Security Group at any time. The route remains. The exposure is latent.

**Route table (architectural control):** A route table with only the `local` entry on a data subnet has no internet path at all. There is nothing to misconfigure. There is no route to remove or add — the path simply does not exist. A CBN auditor can open the route table in the AWS console and see no `0.0.0.0/0` entry. That is the evidence. It is tamper-evident and structurally enforced.

The correct answer for "data tier must be isolated from the internet at the network layer, provably" is always: isolated route table with `local` only. A Security Group alone will never satisfy this requirement on the SAA-C03 or in a real compliance audit.

---

## Management Subnet Pattern

A management subnet is a small `/28` subnet (11 usable IPs) used exclusively for Bastion Host EC2 instances. It is a common production pattern for secure SSH access to private resources.

**Configuration:**
- CIDR: `10.0.100.0/28` (11 usable IPs — enough for 2 Bastion Hosts with room for expansion)
- Route table: `public-rt` (needs IGW route so the Bastion has internet reachability for SSH inbound)
- Security Group on Bastion: allows SSH (port 22) ONLY from specific static IPs (company office IP, CISO home IP)
- All sessions logged via AWS Systems Manager Session Manager to S3 for regulatory audit trail
- Auto-Assign Public IP: OFF — Bastion Hosts get Elastic IPs for predictable, stable public addresses
- NACL on management subnet: blocks all traffic except SSH (port 22) inbound — no other resource type can receive traffic in this subnet even if accidentally deployed

**Why /28 and not /24:** Bastion Hosts need 2 IPs at most (one per AZ for HA). A `/28` wastes fewer addresses and signals intent — it is a signal in architecture documentation that this subnet is deliberately constrained to a specific single purpose.

---

## Common Exam Traps

**Trap 1 — The name "Public" means nothing without the route table.** Auto-Assign IP enabled + subnet named "Public" + IGW exists — but if the route table has only the `local` route, the subnet is private. The exam tests this every time.

**Trap 2 — Subnets cannot span AZs.** "A subnet spanning multiple AZs" is always wrong. "A VPC spanning multiple AZs" is always correct.

**Trap 3 — A /24 has 251 usable IPs, not 256.** If the requirement is 250 instances and the candidate selects /24, they are wrong by 1 IP — the reserved 5 bring it below the minimum. Use /23.

**Trap 4 — Default VPC subnets are all public.** The default VPC is not safe by default. Every EC2 launched there gets a public IP. Security depends entirely on the default Security Group — one misconfiguration and resources are internet-exposed.

**Trap 5 — Security Groups cannot satisfy network-layer isolation requirements.** Route table = architecture, provable and tamper-evident. Security Group = operational control, changeable by any authorised IAM principal. For compliance, the route table is the evidence. Always.
