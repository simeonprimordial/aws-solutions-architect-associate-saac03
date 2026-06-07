# VPC Peering & Connectivity

Real-world AWS deployments rarely live in a single VPC. Production, development, staging, and different business units each run in separate VPCs for security isolation and cost control. Nigerian banks like Zenith, GTBank, and UBA separate core banking from analytics from internal tools — all in different accounts and VPCs. Eventually those VPCs need to communicate with each other, reach on-premises data centres, or access AWS services privately. AWS gives you five tools to do this. Choosing the wrong one is one of the most expensive architecture mistakes — both in monthly cost and in SAA-C03 exam marks.

---

## VPC Peering

VPC Peering creates a direct, private network link between two VPCs using AWS's private backbone — never the internet. Traffic behaves as if the instances are on the same private network. No NAT Gateway needed. No encryption overhead — the traffic is already private over AWS infrastructure.

**How it works:** A peering connection (`pcx-xxxxx`) is created between VPC A and VPC B. EC2 in VPC A (`10.0.1.5`) can communicate directly with RDS in VPC B (`172.16.1.8`).

**Route tables — both VPCs:** Peering alone does NOT route traffic. You must add a route in VPC A's route table pointing to VPC B's CIDR, AND a route in VPC B's route table pointing to VPC A's CIDR. Both directions must be configured. One-way routes produce one-way connectivity.

```
VPC A route table — add:
Destination: 172.16.0.0/16    Target: pcx-xxxxx (peering connection to VPC B)

VPC B route table — add:
Destination: 10.0.0.0/16      Target: pcx-xxxxx (peering connection to VPC A)
```

**Security Group referencing across peered VPCs:** In the same region, a Security Group in VPC A can reference a Security Group in VPC B as an inbound source. More secure than CIDR blocks because it follows the resource, not the IP address.

**Key rules:**

No CIDR overlap — if VPC A is `10.0.0.0/16` and VPC B is `10.0.0.0/24`, they overlap and cannot peer. Plan IP addressing before creating VPCs.

Non-transitive routing — this is the most-tested VPC Peering concept. If A peers B and B peers C, then A CANNOT reach C through B. Traffic does NOT transit through a peered VPC. Every direct connection requires its own peering relationship with its own route entries.

Peering connection count: `n(n-1)/2`. For 3 VPCs needing full mesh: 3 connections. For 10 VPCs: 45 connections — use Transit Gateway instead.

Cross-account and cross-region: peering works across AWS accounts and across regions. Inter-region peering has higher data transfer costs than intra-region peering.

**When to use:** 2–3 VPCs, simple topology, same or different accounts. Simple, cheap, private, low overhead. Avoid for anything requiring Dev/Prod isolation that peering topology alone cannot enforce.

> ⚠️ **Exam Tip:** The exam will describe a connectivity issue between VPC A and VPC C where A peers B and B peers C. The answer is never "make the peering transitive" — peering is structurally non-transitive. The answer is a direct A-C peering connection with new route entries in both A and C, or Transit Gateway.

---

## AWS Transit Gateway

Transit Gateway is a regional routing hub. Every VPC and on-premises network connects once to the TGW. TGW handles all routing between all attachments. The architecture is hub-and-spoke: each spoke connects to the hub once, and the hub manages all cross-spoke routing.

**Attachments:** VPCs, Site-to-Site VPNs, Direct Connect Gateways, and other Transit Gateways (for inter-region connectivity via TGW peering).

**vs VPC Peering at scale:**

| Number of VPCs | VPC Peering (full mesh) | Transit Gateway |
|---|---|---|
| 3 | 3 connections | 3 attachments |
| 5 | 10 connections | 5 attachments |
| 10 | 45 connections | 10 attachments |
| 20 | 190 connections | 20 attachments |

**TGW Route Table Isolation:** Unlike VPC Peering, Transit Gateway enforces isolation via separate route tables. Dev VPC and Prod VPC can both attach to the same TGW — but if they are in different TGW route tables, they cannot reach each other. Shared Services VPC can be in both route tables, reachable from both Dev and Prod.

**Zenith Bank example:**
- `prod-rt` (TGW route table 1): Prod VPC, Analytics VPC, Staging VPC, Shared Services VPC — all can communicate
- `dev-rt` (TGW route table 2): Dev VPC, Shared Services VPC — Dev can reach Shared Services but CANNOT reach Prod

**Cost:** `$0.05/hr` per attachment + `$0.02/GB` processed. Regional resource — set up per region.

**When to use:** 10+ VPCs, need traffic isolation between VPCs, hub-and-spoke architecture, mix of VPC and on-premises connectivity through a single hub.

> ⚠️ **Exam Tip:** Transit Gateway does NOT automatically allow all attached VPCs to talk to each other — it depends on TGW route tables. A scenario where Dev cannot reach Prod even through TGW is enabled by putting them in different TGW route tables, not different peering connections.

---

## AWS Site-to-Site VPN

Site-to-Site VPN connects an on-premises office or data centre to a VPC using an encrypted IPsec tunnel over the public internet.

**Components:**
- **Customer Gateway (CGW)** — represents the on-premises router or firewall. Configured with the public IP of the on-premises device.
- **Virtual Private Gateway (VGW)** — the AWS-side VPN endpoint, attached to the VPC.
- **Two IPsec tunnels** — AWS automatically creates two redundant tunnels per VPN connection. Both must be monitored. No automatic failover without explicit configuration.

**Key properties:**

| Property | Value |
|---|---|
| Encryption | IPsec — encrypted by default |
| Speed | Up to 1.25 Gbps per connection |
| Latency | Variable — traverses public internet |
| Setup time | Minutes — no physical provisioning |
| Cost | ~`$0.05/hr` per connection + data transfer |
| Redundancy | 2 tunnels auto-provisioned — monitor both |

**Best for:** Branch offices, development environments, organisations not ready for a Direct Connect commitment, backup/failover for Direct Connect.

> ⚠️ **Exam Tip:** VPN creates 2 tunnels automatically. If one fails silently and the second also fails, the result is a complete outage. Both tunnels must be actively monitored with health alarms configured.

---

## AWS Direct Connect

Direct Connect is a dedicated private fibre connection from an on-premises data centre to an AWS Direct Connect location. Traffic never traverses the public internet.

**Key properties:**

| Property | Value |
|---|---|
| Encryption | NOT encrypted by default |
| Speed | 1 Gbps, 10 Gbps, or 100 Gbps dedicated |
| Latency | Consistent, predictable — no internet variability |
| Setup time | Weeks to months — physical provisioning required |
| Cost | Port hour fee (~`$216/month` for 1 Gbps) + reduced data transfer rates |

**Best for:** High-bandwidth workloads (100GB+/day), latency-sensitive applications, compliance environments, reducing data transfer costs at scale.

**Redundancy:** Provision two DX connections in different Direct Connect locations. Use a Site-to-Site VPN as automatic fallback if DX fails.

**DX is NOT encrypted by default — the most important DX fact.** A dedicated private fibre is private (not shared with other organisations) but traffic on it is not cryptographically encrypted. For environments requiring both a private path AND encryption:

**DX + VPN = best of both worlds:**
- Direct Connect provides the dedicated private path with consistent low latency
- Site-to-Site VPN layered over DX provides IPsec encryption
- Result: private, encrypted, consistent, compliant

**Zenith Bank Lagos DC:** 500GB+/day to Analytics VPC. Treasury systems need `<20ms` latency — internet VPN latency of 80–200ms is unacceptable for real-time trading positions. 1 Gbps DX provisioned via AWS partner in Lagos. Data transfer cost reduced 70%. Treasury latency: 15–25ms consistent. VPN configured as automatic failover.

> ⚠️ **Exam Tip:** Direct Connect is NOT encrypted by default. Any scenario requiring both a dedicated private connection AND encryption → DX + VPN combined. DX alone fails the encryption requirement. VPN alone fails the latency and dedicated-path requirements.

---

## VPC Endpoints

VPC Endpoints allow private subnet resources to access AWS services without any traffic leaving the AWS network — no NAT Gateway, no Internet Gateway, no public IP required.

### Gateway Endpoints — Free

For S3 and DynamoDB only. Works by adding a route table entry in the private subnet's route table pointing S3/DynamoDB traffic to the endpoint. Traffic travels over the AWS private backbone.

```
Private subnet route table — with S3 Gateway Endpoint:
Destination               Target
10.0.0.0/16               local
pl-xxxxx (S3 prefix list) vpce-xxxxx (S3 Gateway Endpoint)
0.0.0.0/0                 nat-xxxxx  (still needed for other internet traffic)
```

Cost: `$0/hr`. `$0/GB`. There is no reason not to use this for S3 and DynamoDB in any private subnet.

The Zenith Bank core banking scenario: EC2s in private subnets with NO NAT Gateway route (CBN requires zero internet path from core banking systems) needed to write encrypted audit logs to S3. Without an endpoint there was no path to S3. S3 Gateway Endpoint added to route tables — traffic routes to S3 via AWS backbone, never the internet. Cost: $0. CBN isolation maintained.

### Interface Endpoints — Paid (PrivateLink)

For all other AWS services: SSM, SQS, SNS, SES, ECR, Secrets Manager, KMS, CloudWatch, and 100+ others. Creates an Elastic Network Interface (ENI) with a private IP in the subnet.

Cost: ~`$0.01/hr` per AZ + ~`$0.01/GB` processed.

AWS PrivateLink is the underlying technology. It also allows SaaS vendors to expose their services privately to customer VPCs without traffic crossing the internet.

**Required for Session Manager on isolated instances:** Private instances with no NAT Gateway that need Session Manager access require Interface Endpoints for `ssm`, `ssmmessages`, and `ec2messages`.

> ⚠️ **Exam Tip:** Gateway Endpoints (S3, DynamoDB) are free. Interface Endpoints (everything else) cost per hour per AZ. For private subnet S3 access — Gateway Endpoint is always the answer. Choosing NAT Gateway for S3 access costs money and routes traffic over the internet unnecessarily.

---

## Decision Framework

| Scenario / Keywords | Best Tool | Avoid |
|---|---|---|
| Connect 2–3 VPCs, same or different accounts | VPC Peering | TGW — overkill |
| Connect 10+ VPCs, isolation between VPCs, hub-and-spoke | Transit Gateway | Peering — unmanageable mesh |
| On-premises → AWS, low volume, quick setup | Site-to-Site VPN | DX — weeks lead time |
| On-premises → AWS, high bandwidth, low latency, high volume | Direct Connect | VPN — 1.25 Gbps cap, variable latency |
| Encryption AND dedicated private connection required | DX + VPN | DX alone — not encrypted |
| Private subnet → S3 or DynamoDB without internet | S3/DynamoDB Gateway Endpoint (FREE) | NAT Gateway — costs money |
| Private subnet → SSM, SQS, SNS, etc. without internet | Interface Endpoint (PrivateLink) | NAT Gateway — internet exposure |

**Decision shortcut:** Few VPCs → Peering. Many VPCs → TGW. On-prem low volume → VPN. On-prem high volume → DX. Private path + encryption → DX + VPN. Private subnet → S3 → Gateway Endpoint.

---

## Common Exam Traps

**Trap 1 — VPC Peering is non-transitive.** A peers B and B peers C does NOT give A-C. This is the most-tested peering concept. Each pair of VPCs needing communication requires its own peering connection. At scale, use Transit Gateway.

**Trap 2 — Peering alone routes nothing.** Create the peering connection, then add route entries in BOTH VPCs' route tables. One-way route produces one-way traffic. The exam will describe a one-way connectivity issue — add the missing route in the other VPC.

**Trap 3 — Direct Connect is NOT encrypted by default.** DX = private, not encrypted. VPN = encrypted, not dedicated. DX + VPN = private AND encrypted. For CBN financial compliance requiring both — the answer is DX + VPN.

**Trap 4 — VPN has two tunnels — both must be monitored.** Auto-provisioned two tunnels. No automatic failover without configuration. Both must have health monitoring and alarms.

**Trap 5 — Gateway Endpoints are free.** For S3 and DynamoDB: zero cost. Never use NAT Gateway for S3 access in private subnets. Gateway Endpoint is always the correct answer. No exceptions.
