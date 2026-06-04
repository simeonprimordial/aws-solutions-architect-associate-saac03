# Route Tables, Internet Gateways & NAT

Days 1 and 2 established the structure — VPC boundary, subnets, and the principle that a subnet's routing determines its public or private status. Today answers the mechanism behind that principle: how does a packet actually know where to go? The answer is route tables. Every packet entering, leaving, or travelling within the VPC is evaluated against a route table. That evaluation decides whether the packet goes to the internet, to another subnet in the VPC, or nowhere. Route tables are the traffic control system of the entire cloud network — and they are also the compliance evidence. A route table is inspectable, tamper-evident, and auditable in a way that a Security Group rule is not.

---

## Route Table Anatomy

A route table is a set of rules. Each rule — a route entry — has two components:

- **Destination** — the CIDR block this rule matches against. Examples: `10.0.0.0/16`, `10.0.1.0/24`, `0.0.0.0/0`.
- **Target** — where matching traffic is sent. Examples: `local`, `igw-xxxxxxxx`, `nat-xxxxxxxx`, `eni-xxxxxxxx`.

When a packet arrives, AWS evaluates ALL route entries and selects the one with the **most specific match** — the longest prefix (the smallest IP range that still contains the destination IP). This is called longest prefix match.

**Priority resolution example:**

A packet heading to `10.0.1.5` is evaluated against this route table:

```
Destination     Target          Result
10.0.1.0/24     local           WINS — most specific (/24 beats /16 beats /0)
10.0.0.0/16     local           Loses — less specific
0.0.0.0/0       igw-xxxxxxxx    Loses — least specific (catch-all)
```

The `/24` match wins. The packet stays inside the VPC and never reaches the IGW. This is why VPC-internal traffic between subnets never exits the VPC even when an IGW route exists — the `local` route is always more specific than `0.0.0.0/0`.

> ⚠️ **Exam Tip:** The most specific route always wins. `10.0.1.0/24 > 10.0.0.0/16 > 0.0.0.0/0`. A packet to any address within the VPC CIDR will always match the `local` route before the `0.0.0.0/0` catch-all. The IGW never handles VPC-internal routing.

---

## The Local Route — Permanent and Immutable

Every route table automatically contains one route that was created at VPC creation and cannot be deleted or modified:

```
Destination     Target
10.0.0.0/16     local
```

This route means: all traffic destined for any IP address within the VPC CIDR (`10.0.0.0/16`) stays inside the VPC. It is handled internally by the VPC's routing fabric. It never exits.

**What this means in practice:**
- Traffic from `10.0.1.5` (public subnet EC2) to `10.0.21.8` (private subnet RDS) takes the `local` route — never the IGW.
- You cannot add a route that overrides the `local` route for VPC-internal addresses.
- Even if you add `0.0.0.0/0 → IGW`, any destination within `10.0.0.0/16` will still match the more specific `local` route first.

This is a deliberate, permanent constraint. AWS guarantees that VPC-internal routing never accidentally escapes to the internet.

> ⚠️ **Exam Tip:** Traffic between a public subnet and a private subnet never goes through the Internet Gateway. It always takes the local route. Candidates who think the IGW handles intra-VPC routing lose marks consistently.

---

## Internet Gateway (IGW)

The Internet Gateway is the AWS-managed, horizontally-scaled, redundant component that enables **bidirectional** communication between your VPC and the public internet. One IGW per VPC. No bandwidth limit. No availability concern — AWS manages redundancy.

**Three conditions required for internet connectivity — ALL must be true:**

1. The IGW is created and **attached to the VPC**.
2. The subnet's route table has a `0.0.0.0/0 → igw-xxxxxxxx` entry.
3. The EC2 instance has a **public IP or Elastic IP** address.

Miss any one of these three and internet connectivity fails. Exam connectivity troubleshooting questions almost always have exactly one of the three missing — identify which one.

**IGW NAT behaviour:** When an EC2 instance with a public IP sends traffic outbound, the IGW translates the instance's private IP to its public IP (source NAT). When traffic returns, the IGW translates the public IP back to the private IP. This is transparent NAT — the instance itself only sees its private IP. It does not know its public IP.

**Creating and attaching an IGW:**
```
Step 1: VPC → Internet Gateways → Create internet gateway
Step 2: Actions → Attach to VPC → select the VPC
Step 3: In the public subnet's route table: Edit routes → Add route
         Destination: 0.0.0.0/0
         Target: igw-xxxxxxxx
```

> ⚠️ **Exam Tip:** Creating an IGW does nothing until you attach it. Attaching it does nothing until a route table points to it. The route table entry is what makes a subnet public — not the existence of the IGW.

---

## NAT Gateway

The NAT Gateway sits in a public subnet and allows private subnet instances to initiate **outbound** internet connections — downloading patches, calling external APIs like `api.paystack.co` or Interswitch endpoints — while blocking all unsolicited inbound connections.

**Mechanics:**
- Lives in a **public subnet** (needs the IGW route to reach the internet).
- Requires an **Elastic IP** address.
- Private subnet's route table: `0.0.0.0/0 → nat-xxxxxxxx`.
- Translates the private instance's IP to the NAT Gateway's Elastic IP for outbound. Returns response traffic to the originating instance. Blocks all unsolicited inbound.

**Cost model — not free:**

| Charge Type | Rate |
|---|---|
| Per hour (whether used or not) | `$0.045/hr` ≈ `$32.85/month` |
| Per GB data processed | `$0.045/GB` |
| Cross-AZ data transfer | Extra `$0.01/GB` |

One NAT Gateway, 1 month, 100GB/day: approximately `$166/month`. For HA across two AZs: approximately `$332/month`. NAT Gateway cost is a significant line item in any production AWS bill.

**Multi-AZ High Availability:**

NAT Gateways are AZ-scoped — they live in one AZ. If that AZ has a connectivity issue, the NAT Gateway fails and any private subnet routing through it loses outbound internet access. For HA:

```
AZ-1a private subnets → private-rt-az1a → 0.0.0.0/0 → nat-az1a (in public-az1a)
AZ-1b private subnets → private-rt-az1b → 0.0.0.0/0 → nat-az1b (in public-az1b)
```

Each AZ is independently resilient. If `az1a` fails, `az1b` private subnets continue to function via their own NAT Gateway. This is the production pattern — and the reason cross-AZ NAT routing is an anti-pattern: it creates a cross-AZ dependency and charges extra data transfer fees.

**Protocol limitation:**

NAT Gateway supports TCP, UDP, and ICMP only. It does NOT support:
- GRE (used by PPTP VPN tunnels)
- IPsec ESP/AH in transport mode

If a workload requires GRE or raw IPsec through the NAT path, a NAT Instance (an EC2 with IP forwarding enabled) is required instead — not another NAT Gateway.

> ⚠️ **Exam Tip:** NAT Gateway vs NAT Instance comparison appears frequently. NAT Gateway: managed, highly available, scalable, supports only TCP/UDP/ICMP, higher cost. NAT Instance: EC2-based, you manage patches and HA, supports all protocols (IP forwarding on), lower cost at very low traffic. For cost-optimisation of low-traffic private subnet workloads, the exam often favours NAT Instance.

---

## Egress-Only Internet Gateway

An Egress-Only Internet Gateway is for **IPv6 outbound-only traffic** exclusively. It is not relevant for IPv4 networking.

Why it exists: IPv6 addresses are globally unique — there is no NAT in IPv6. A regular IGW with an IPv6 route would allow both inbound and outbound IPv6 traffic, directly exposing private IPv6 instances to the internet. An Egress-Only IGW allows private IPv6 instances to initiate outbound connections while blocking all unsolicited inbound IPv6.

It is stateful — return traffic for outbound connections is automatically allowed.

**What it is NOT:**
- Not a replacement for NAT Gateway (which handles IPv4).
- Not appropriate for data tier isolation (which requires no internet route of any kind).
- Not a way to add outbound-only access to RDS — the data tier should have no internet route whatsoever.

> ⚠️ **Exam Tip:** An exam question describing a VPC with IPv6 and private resources that need outbound internet access → Egress-Only IGW. An exam question describing a data tier that needs no internet connectivity → isolated route table with local-only. These are completely different scenarios. Egress-Only IGW is specifically for IPv6.

---

## Main Route Table — The Default Danger

When you create a VPC, AWS automatically creates a Main Route Table containing only the `local` route. Every subnet that is NOT explicitly associated with a custom route table automatically uses the Main Route Table.

**The anti-pattern that caused the OPay incident:**

Someone added `0.0.0.0/0 → IGW` to the Main Route Table "temporarily" during debugging. Six months later, four subnets including internal payment processing microservices — which had never been explicitly associated with any route table — were publicly routable via the internet. A security audit found the internal payment API reachable from the public internet.

**Best practice:**
- Keep the Main Route Table as `local` only — never add an IGW or NAT route to it.
- Create named custom route tables for every traffic pattern: `public-rt`, `private-rt`, `isolated-rt`.
- Explicitly associate every subnet with a named custom route table.
- A subnet that falls through to Main gets the `local`-only table — a safe default rather than a security incident.

**Route table naming and tagging convention:**

```
public-rt     → 0.0.0.0/0 → IGW        → associated with: public subnets
private-rt    → 0.0.0.0/0 → NAT GW     → associated with: private-app subnets
isolated-rt   → local only              → associated with: private-data subnets
main-rt       → local only              → default for unassociated subnets (safety net)
```

Tag every route table with: `Name`, `Environment`, `Tier`, `Owner`, `LastReviewed`. This is the operational metadata that makes route tables identifiable during incidents and auditable during compliance reviews.

---

## Security Group Chaining

Security Group chaining is the professional pattern for inter-tier traffic control in a 3-tier architecture. Instead of opening each tier to `0.0.0.0/0` or to a CIDR range, you reference a Security Group as the traffic source. Only resources that are members of the referenced Security Group can send traffic.

**Chain structure for a 3-tier application:**

```
Internet → SG-LoadBalancer → SG-WebServers → SG-Database
```

**SG-LoadBalancer (internet-facing ALB):**

| Rule | Protocol | Port | Source |
|---|---|---|---|
| Inbound | TCP | 443 (HTTPS) | `0.0.0.0/0` |
| Inbound | TCP | 80 (HTTP) | `0.0.0.0/0` |
| Outbound | All | All | Default allow all |

**SG-WebServers (private-app EC2 instances):**

| Rule | Protocol | Port | Source |
|---|---|---|---|
| Inbound | TCP | 80 | `SG-LoadBalancer` ← SG reference |
| Inbound | TCP | 22 (SSH) | `x.x.x.x/32` ← your IP only |
| Outbound | All | All | Default allow all |

**SG-Database (private-data RDS):**

| Rule | Protocol | Port | Source |
|---|---|---|---|
| Inbound | TCP | 3306 (MySQL) | `SG-WebServers` ← SG reference |
| Outbound | All | All | Default allow all |

**Why SG chaining is superior to CIDR-based rules:**

If `SG-WebServers` references `SG-LoadBalancer` as the source, only the specific ALB instances that are members of `SG-LoadBalancer` can reach the web servers — not any IP in the subnet CIDR. If the ALB is scaled out or replaced, the new instances automatically inherit `SG-LoadBalancer` membership and gain access. No rule update required. If the ALB subnet CIDR was the source instead, any resource launched in that subnet could potentially reach the web servers, whether intended or not.

Security Group chaining creates a trust chain based on resource identity, not IP address. This is the pattern every production AWS environment uses.

> ⚠️ **Exam Tip:** An exam question asking about the most secure way to allow an ALB to communicate with EC2 instances → reference the ALB's Security Group as the inbound source on the EC2 Security Group. Not a CIDR. Not `0.0.0.0/0`. The SG reference.

---

## End-to-End Traffic Flow Trace

This is the route table question that appears in disguise in every AWS interview and many SAA-C03 scenarios.

**Scenario: User makes HTTPS request to a 3-tier fintech application.**

**Step 1: User → ALB (inbound public)**
```
Source:  User IP (internet)
Dest:    ALB public IP (in public subnet 10.0.1.0/24)
Path:    Internet → IGW → ALB
Route:   0.0.0.0/0 → igw-xxxxxxxx  (public-rt, public subnet)
SG:      SG-LoadBalancer: inbound TCP 443 from 0.0.0.0/0 ✓
```

**Step 2: ALB → EC2 App Server (VPC-internal)**
```
Source:  ALB (10.0.1.x)
Dest:    EC2 (10.0.11.x, private-app subnet)
Path:    VPC-internal via local route — no internet involved
Route:   10.0.0.0/16 → local  (most specific match for 10.0.11.x)
SG:      SG-WebServers: inbound TCP 80 from SG-LoadBalancer ✓
```

**Step 3: EC2 App Server → RDS (VPC-internal)**
```
Source:  EC2 (10.0.11.x)
Dest:    RDS (10.0.21.x, isolated data subnet)
Path:    VPC-internal via local route — isolated-rt has no other routes
Route:   10.0.0.0/16 → local  (only route in isolated-rt)
SG:      SG-Database: inbound TCP 3306 from SG-WebServers ✓
```

**Step 4: EC2 App Server → External API (outbound via NAT)**
```
Source:  EC2 (10.0.11.x)
Dest:    api.paystack.co (external internet)
Path:    EC2 → private-rt → NAT Gateway (in public subnet) → IGW → internet
Route:   0.0.0.0/0 → nat-xxxxxxxx  (private-rt, private-app subnet)
```

The critical observation: steps 2 and 3 never touch the internet. They use the `local` route throughout — the most specific match for any `10.0.x.x` destination. The IGW is only involved in steps 1 (inbound from internet to ALB) and 4 (outbound from EC2 through NAT).

---

## Common Exam Traps

**Trap 1 — NAT Gateway is not free.** `$0.045/hr + $0.045/GB`. One NAT Gateway running for a month without processing a single packet costs `$32.85`. An exam cost-optimisation question for low-traffic private subnets should point to a NAT Instance, not a NAT Gateway.

**Trap 2 — All three conditions required for internet connectivity.** IGW attached + route table entry + public/Elastic IP. Any one missing breaks connectivity. Debug these three in order.

**Trap 3 — NAT Gateway does not support GRE or IPsec.** TCP, UDP, ICMP only. A failing VPN tunnel through a NAT Gateway requires a NAT Instance with IP forwarding, not another NAT Gateway.

**Trap 4 — Unassociated subnets use the Main Route Table.** A forgotten subnet association is not safe if the Main Route Table has an IGW route. Keep Main as local-only.

**Trap 5 — The local route always beats 0.0.0.0/0 for VPC-internal traffic.** Traffic between subnets within the same VPC never goes through the IGW. The `10.0.0.0/16 → local` route is always more specific than `0.0.0.0/0 → IGW` for any destination within the VPC CIDR.
