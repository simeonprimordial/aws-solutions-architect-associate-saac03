# VPC Fundamentals

Every EC2 instance, RDS database, Lambda function with a private endpoint, and ECS task you deploy on AWS lives inside a network. That network is the Amazon Virtual Private Cloud. Understanding VPC is not optional for the SAA-C03 — it appears in Domain 1 (Resilient Architectures), Domain 2 (Secure Architectures), and as the invisible backdrop to almost every multi-service scenario question. In Nigerian fintech and banking contexts, VPC architecture is also a regulatory requirement: CBN and SEC both mandate network segmentation between customer-facing systems and financial processing backends. Getting this right is the difference between a compliant architecture and a non-compliant one.

---

## VPC (Virtual Private Cloud)

A VPC is your own logically isolated, software-defined network inside an AWS Region. It behaves like a traditional on-premises data centre network — you control the IP address space, subnets, routing rules, and firewall behaviour — but there is no physical hardware to rack, cable, or maintain. Everything is software configuration.

**Key mechanics:**
- A VPC is a **Regional resource**. It automatically spans all Availability Zones in the Region where it is created.
- Nothing enters or leaves your VPC unless you explicitly allow it through routing and security rules.
- Each AWS account gets a default VPC per Region. Never use the default VPC for production workloads.
- You define the IP address space when creating the VPC using a CIDR block. You cannot easily change it after creation.

> ⚠️ **Exam Tip:** A VPC spans all AZs in a Region. Subnets are zonal — each subnet exists in exactly one AZ. "A subnet that spans multiple AZs" is never a correct answer.

---

## CIDR Blocks

CIDR stands for Classless Inter-Domain Routing. It is the notation used to define your VPC's IP address range. Written as an IP address followed by a prefix length: `10.0.0.0/16`.

**How the prefix works:**
- The `/N` number means N bits of the address are fixed (the network portion).
- The remaining `32 - N` bits are variable (the host addresses).
- `/16` → `32 - 16 = 16` variable bits → `2^16 = 65,536` total addresses.
- `/24` → `32 - 24 = 8` variable bits → `2^8 = 256` total addresses.
- `/28` → `32 - 28 = 4` variable bits → `2^4 = 16` total addresses.

**Smaller prefix = larger range:** `/8 > /16 > /24 > /28`.

**AWS reserves 5 IPs per subnet** — these are not available for your resources:

| Reserved Address | Purpose |
|---|---|
| `x.x.x.0` | Network address |
| `x.x.x.1` | VPC router |
| `x.x.x.2` | AWS DNS |
| `x.x.x.3` | Future use |
| `x.x.x.255` | Broadcast address |

A `/24` subnet (256 addresses) gives you **251 usable IPs** after the 5 reserved addresses are removed.

**Standard CIDR choices:**

| CIDR | Total IPs | Usable per Subnet | Use Case |
|---|---|---|---|
| `/16` | 65,536 | VPC level | Production VPC CIDR |
| `/24` | 256 | 251 | Standard subnet |
| `/28` | 16 | 11 | Minimal subnet (bastion hosts, endpoints) |

> ⚠️ **Exam Tip:** SAA-C03 questions ask how many hosts fit in a given subnet. Always subtract 5 from the total for AWS reserved addresses. `/26` = 64 total, 59 usable. `/27` = 32 total, 27 usable.

---

## Subnets

A subnet is a subdivision of your VPC's IP address space, tied to **exactly one Availability Zone**. You deploy resources — EC2 instances, RDS databases, Lambda ENIs — into subnets. The subnet's AZ determines where those resources physically reside.

**Public vs Private — the only thing that matters is the route table:**

A subnet is public if its route table has a `0.0.0.0/0` route pointing to an Internet Gateway. Nothing else determines publicness. Not the subnet's name. Not whether auto-assign public IP is enabled. Only the route table.

```
Public subnet route table example:
Destination     Target
10.0.0.0/16     local           ← all internal VPC traffic stays inside
0.0.0.0/0       igw-0abc1234    ← all other traffic → Internet Gateway
```

```
Private subnet route table example:
Destination     Target
10.0.0.0/16     local           ← all internal VPC traffic stays inside
0.0.0.0/0       nat-0def5678    ← all other traffic → NAT Gateway (outbound only)
```

```
Isolated subnet route table example:
Destination     Target
10.0.0.0/16     local           ← only VPC-internal traffic. No internet route at all.
```

**Auto-assign public IP** is a subnet setting that automatically assigns a public IPv4 address to instances launched into the subnet. Enable this on public subnets. Leave it off for private subnets. But remember: auto-assign public IP alone does not make a subnet public. Without the IGW route, the public IP is useless for internet connectivity.

> ⚠️ **Exam Tip:** The most important VPC concept on the SAA-C03. A subnet named "Public-Subnet" with auto-assign public IP enabled but no IGW route is a private subnet. Routing determines access — always.

---

## Route Tables

A route table is the set of rules that determines where network traffic from a subnet is directed. Every subnet must be associated with exactly one route table. Multiple subnets can share the same route table.

**How route evaluation works:**
- AWS evaluates the most specific (longest prefix) route first.
- `10.0.0.0/16 local` always exists — VPC-internal traffic never leaves the VPC regardless of other routes.
- The `0.0.0.0/0` route (the default route) catches all traffic not matched by a more specific entry.

**Production VPC uses at least two route tables:**
1. Public route table — associated with public subnets, contains `0.0.0.0/0 → IGW`.
2. Private route table — associated with private subnets, contains `0.0.0.0/0 → NAT Gateway`.

The main route table is the default that new subnets are associated with when created. Best practice: keep the main route table as private (no IGW route) and explicitly associate public subnets with a custom public route table.

---

## Internet Gateway (IGW)

The Internet Gateway is an AWS-managed, horizontally scaled, redundant gateway that enables two-way communication between your VPC and the public internet. It serves both inbound traffic from the internet and outbound traffic from your instances to the internet.

**Mechanics:**
- One IGW per VPC.
- Create the IGW, then attach it to your VPC. These are two separate steps.
- Then create a route in your public subnet's route table: `0.0.0.0/0 → igw-xxxxxxxx`.
- The IGW also performs Network Address Translation (NAT) for instances with public IP addresses — translating between the instance's private IP and its public IP.

**An IGW is what makes a subnet public.** No IGW route = private subnet, regardless of any other configuration.

> ⚠️ **Exam Tip:** If a question describes instances in a subnet that cannot be reached from the internet, check first whether an IGW exists and is attached, then whether the route table has a `0.0.0.0/0 → IGW` entry. Missing either one = no internet connectivity.

---

## NAT Gateway

The NAT Gateway allows EC2 instances in **private subnets** to initiate **outbound** internet connections — downloading software patches, calling external APIs like Paystack webhooks — while blocking all unsolicited inbound connections from the internet.

**Mechanics:**
- The NAT Gateway itself lives in a **public subnet** and requires an **Elastic IP address**.
- Private subnet instances route their internet traffic to the NAT Gateway via the private subnet's route table: `0.0.0.0/0 → nat-xxxxxxxx`.
- The NAT Gateway translates the private instance's private IP to its Elastic IP for outbound connections. Response traffic is allowed back through. Unsolicited inbound is blocked.
- AWS-managed: highly available within an AZ. For multi-AZ resilience, create one NAT Gateway per AZ and have each AZ's private subnets route to the NAT Gateway in the same AZ.

**NAT Gateway vs NAT Instance:**

| Feature | NAT Gateway | NAT Instance |
|---|---|---|
| Management | AWS-managed | Self-managed EC2 |
| Availability | Highly available in AZ | Single point of failure |
| Bandwidth | Up to 100 Gbps | Instance type dependent |
| Cost | Per hour + per GB | Instance cost |
| SAA-C03 answer | Always preferred | Legacy — almost never the right answer |

> ⚠️ **Exam Tip:** NAT Gateway is for OUTBOUND internet from private subnet instances. It does NOT enable inbound internet access. Any scenario where users need to reach your application requires a public subnet + IGW + Load Balancer or public IP. A NAT Gateway alone will never solve an inbound access requirement.

---

## Security Groups vs Network ACLs

This is the most-tested distinction in the entire VPC topic. Know it cold.

| Feature | Security Group | Network ACL (NACL) |
|---|---|---|
| Level | Instance-level (applied to EC2 ENI) | Subnet-level (applied to entire subnet) |
| State | **STATEFUL** — return traffic auto-allowed | **STATELESS** — must explicitly allow both directions |
| Rules | Allow rules only — no explicit deny | Allow AND Deny rules |
| Evaluation | ALL rules evaluated — most permissive wins | Rules evaluated in number order — first match wins |
| Default inbound | DENY all (nothing allowed until you add rules) | ALLOW all (default NACL allows everything) |
| Default outbound | ALLOW all | ALLOW all |
| Use case | Primary instance-level security | Secondary defence — blocking IPs/ports at subnet boundary |

**The stateful vs stateless difference in practice:**

Security Group: You add an inbound rule allowing TCP port 443 from `0.0.0.0/0`. A user connects on HTTPS. The response traffic flows back automatically — you do not need an outbound rule for it. The Security Group tracks the connection state.

NACL: You add an inbound rule allowing TCP port 443. The request comes in. The server tries to send a response on an **ephemeral port** (range `1024–65535` — the client picks a random high port for the reply). If your NACL does not have an outbound allow rule for ports `1024–65535`, the response is blocked and the connection fails silently. This trips up candidates who treat NACLs like Security Groups.

**NACL Rule Evaluation:**

Rules are evaluated in ascending number order. The first matching rule wins and processing stops. Always end with a rule number 32767 or `*` that explicitly denies everything not matched above.

```
Example NACL inbound rules for a public subnet:
Rule #   Type        Protocol   Port Range   Source          Action
100      HTTPS       TCP        443          0.0.0.0/0       ALLOW
110      HTTP        TCP        80           0.0.0.0/0       ALLOW
32767    All Traffic All        All          0.0.0.0/0       DENY
```

```
Corresponding NACL outbound rules (required for stateless return traffic):
Rule #   Type        Protocol   Port Range   Destination     Action
100      Custom TCP  TCP        1024-65535   0.0.0.0/0       ALLOW   ← ephemeral ports
32767    All Traffic All        All          0.0.0.0/0       DENY
```

> ⚠️ **Exam Tip:** Any NACL question involving blocked traffic — check the outbound ephemeral port rule first. Missing `1024–65535` outbound is the most common NACL misconfiguration and a specific SAA-C03 exam requirement.

---

## Additional Networking Concepts

### VPC Peering

A networking connection between two VPCs that enables routing between them using private IP addresses. Works across accounts and regions.

**Non-transitive by design:** If VPC A peers VPC B, and VPC B peers VPC C, instances in VPC A cannot reach VPC C through VPC B. Each peering relationship is a direct, private link between exactly two VPCs. To connect three VPCs fully to each other: three peering connections (A↔B, B↔C, A↔C). At scale, use AWS Transit Gateway instead.

### Elastic IP (EIP)

A static, public IPv4 address allocated to your account. Unlike auto-assigned public IPs (which change when an instance stops and restarts), an Elastic IP persists. Attach it to an EC2 instance or a NAT Gateway. You pay for Elastic IPs that are allocated but not attached to a running resource — release unused EIPs.

### VPC Endpoints

Allow private communication between your VPC and AWS services without traffic leaving the AWS network and without requiring an IGW, NAT Gateway, or VPN.

| Type | Services | Cost |
|---|---|---|
| Gateway Endpoint | S3, DynamoDB | Free |
| Interface Endpoint | Most other AWS services | Per hour + per GB |

> ⚠️ **Exam Tip:** VPC Endpoints connect to AWS services only. They do not provide general internet access. A question asking how to allow private subnet instances to reach S3 without internet traffic → VPC Gateway Endpoint. A question asking how to download OS patches → NAT Gateway. These are completely different requirements.

---

## Production Architecture — The 3-Tier VPC Pattern

This is the standard architecture for any production AWS workload. The SAA-C03 will test variations of this pattern repeatedly.

```
Internet / Users
      │
      ▼
Internet Gateway (IGW)
      │
      ▼
┌─────────────────────────────────────────────────────────┐
│  VPC — 10.0.0.0/16 (af-south-1 for Nigeria workloads)  │
│                                                         │
│  ┌─────────────────┐    ┌─────────────────┐            │
│  │  AZ-1a          │    │  AZ-1b          │            │
│  │                 │    │                 │            │
│  │ Public Subnet   │    │ Public Subnet   │            │
│  │ 10.0.1.0/24     │    │ 10.0.2.0/24     │            │
│  │ ALB · Bastion   │    │ ALB · Bastion   │            │
│  │ NAT Gateway     │    │ NAT Gateway     │            │
│  │                 │    │                 │            │
│  │ Private App     │    │ Private App     │            │
│  │ 10.0.10.0/24    │    │ 10.0.10.0/24    │            │
│  │ EC2 App Servers │    │ EC2 App Servers │            │
│  │ ECS Tasks       │    │ ECS Tasks       │            │
│  │                 │    │                 │            │
│  │ Private Data    │    │ Private Data    │            │
│  │ 10.0.20.0/24    │    │ 10.0.20.0/24    │            │
│  │ RDS · ElastiCch │    │ RDS · ElastiCch │            │
│  │ No internet rt  │    │ No internet rt  │            │
│  └─────────────────┘    └─────────────────┘            │
└─────────────────────────────────────────────────────────┘
```

**Why each decision:**
- ALB in public subnet: receives internet traffic via IGW route. Terminates TLS. Forwards only to app tier.
- EC2 in private app subnet: no public IPs. Only the ALB's Security Group can send traffic to them. NAT Gateway allows outbound (patches, API calls to Paystack, Interswitch).
- RDS in private data subnet with NO internet route: no `0.0.0.0/0` entry at all — not even a NAT Gateway route. App servers reach RDS only on port `5432` via Security Group. This is the architecture CBN and SEC require for financial data isolation.

---

## Common Exam Traps

**Trap 1 — Naming does not determine publicness.** You can name a subnet "Public-Subnet-AZ-A" and enable auto-assign public IP, but if the route table has no `0.0.0.0/0 → IGW` entry, it is a private subnet. Routing determines access. Always.

**Trap 2 — Security Groups are stateful, NACLs are stateless.** These behave completely differently. Missing the outbound ephemeral port rule (`1024–65535`) in a NACL is the most common cause of silent traffic failures and the most common NACL question type.

**Trap 3 — NAT Gateway is outbound only.** "Add a NAT Gateway so users can access the application" is always wrong. NAT Gateway is for private subnet instances initiating outbound connections. Inbound access requires a public subnet, IGW, and a load balancer or public IP.

**Trap 4 — VPC Peering is non-transitive.** A→B and B→C does not give A→C. Every direct connection requires its own peering relationship or Transit Gateway.

**Trap 5 — VPC Endpoints are for AWS services, not the internet.** A VPC Endpoint for S3 keeps S3 API calls off the internet. It cannot replace a NAT Gateway for general outbound internet access to package repositories or external APIs.
