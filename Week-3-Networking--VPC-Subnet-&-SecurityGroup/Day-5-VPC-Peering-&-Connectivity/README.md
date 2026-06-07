# VPC Peering & Connectivity — Week 3 Day 5

## Topic

All five AWS connectivity tools — VPC Peering, Site-to-Site VPN, Direct Connect, Transit Gateway, and VPC Endpoints — when to use each one, how they compare, and the decision framework for answering the SAA-C03 question "which connectivity option is MOST appropriate?"

This is the final day of Week 3 and the cap on the entire Networking & VPC module. The previous four days built the internal architecture of a single VPC — subnets, routing, gateways, security layers. Today answers what happens when that VPC needs to connect outward: to another VPC, to an on-premises data centre, or to AWS services privately. The Zenith Bank scenario tied every tool together in a realistic multi-account, multi-site architecture covering Transit Gateway for VPC isolation, Direct Connect for the high-volume Lagos data centre, Site-to-Site VPN for the lighter Abuja connection, and S3 Gateway Endpoints for private subnet access to S3 with no internet path. The lab then produced the Week 3 portfolio centrepiece: a complete 3-tier architecture diagram covering every component from Days 1–5.

## What I Learned

- **VPC Peering** — Direct private connection between exactly two VPCs over the AWS backbone. Non-transitive: A↔B and B↔C does NOT give A↔C. Peering alone does not route traffic — both VPCs need route table entries pointing to each other's CIDR. No CIDR overlap allowed. Works cross-account and cross-region.
- **Transit Gateway** — Regional hub-and-spoke routing hub. Every VPC and on-premises network connects once to the TGW. TGW handles all routing between attachments. Separate TGW route tables enforce isolation — Dev VPC cannot reach Prod even through TGW. 10 VPCs: 45 peering connections vs 10 TGW attachments. Cost: `$0.05/hr` per attachment + `$0.02/GB` processed.
- **Site-to-Site VPN** — Encrypted IPsec tunnel between on-premises Customer Gateway (CGW) and AWS Virtual Private Gateway (VGW). Encrypted by default. Up to 1.25 Gbps. AWS creates two tunnels automatically — both must be monitored. Setup in minutes. Cost: ~`$0.05/hr` per connection. Best for branch offices, dev environments, backup for Direct Connect.
- **Direct Connect** — Dedicated private fibre from on-premises data centre to an AWS Direct Connect location. NOT encrypted by default. 1 Gbps, 10 Gbps, or 100 Gbps. Consistent, predictable low latency — no internet variability. Setup: weeks to months. Best for high-bandwidth, low-latency, compliance-sensitive workloads. Add VPN over DX for encryption + private path combined.
- **VPC Endpoints — Gateway** — Free. S3 and DynamoDB only. Adds a route table entry routing S3/DynamoDB traffic to the endpoint over the AWS backbone. No NAT Gateway needed. No internet required. Zero cost — no reason not to use for private subnet S3/DynamoDB access.
- **VPC Endpoints — Interface (PrivateLink)** — Paid (~`$0.01/hr` per AZ + `$0.01/GB`). All other AWS services — SSM, SQS, SNS, ECR, Secrets Manager, and 100+ more. Creates an ENI with a private IP in the subnet. Powered by AWS PrivateLink — the same mechanism SaaS vendors use to expose services privately to customer VPCs.
- **Decision Framework** — Few VPCs → Peering. Many VPCs → Transit Gateway. On-prem low volume → Site-to-Site VPN. On-prem high volume → Direct Connect. Need both private path AND encryption → DX + VPN. Private subnet → S3 → Gateway Endpoint (free). Private subnet → other AWS services → Interface Endpoint.
- **DX + VPN Combination** — Direct Connect is private but not encrypted. Site-to-Site VPN is encrypted but travels over the internet. Combining them — DX for the private path, VPN layered on top for IPsec encryption — satisfies CBN requirements for both a dedicated private connection AND encryption simultaneously.
- **TGW Route Table Isolation** — Unlike VPC Peering, Transit Gateway can enforce isolation at scale: Prod VPC and Dev VPC both attach to the same TGW but use separate route tables, preventing Dev from reaching Prod routing paths even through the hub.
- **VPN Has Two Tunnels — Both Must Be Monitored** — AWS provisions two IPsec tunnels automatically per VPN connection. There is no automatic failover between them without configuration. Both must be actively monitored.

## Hands-On Labs Completed

- Created the complete Week 3 portfolio architecture diagram in draw.io with AWS icons (PNG + PDF export)
- Diagram covers: VPC boundary, 2 AZs, 4 subnets (2 public, 2 private), IGW, NAT Gateway, ALB spanning both AZs, EC2 App Servers, RDS MySQL, Security Group chain (`SG-LoadBalancer → SG-WebServers → SG-Database`) with route table labels on arrows, title, legend, and date
- Committed diagram to GitHub portfolio repo: `simeonprimordial/aws-solutions-architect-associate-saac03`
- **Bonus:** Added a second Development VPC to the diagram with a VPC Peering connection to Production, labelled with the non-transitive routing limitation

## AWS Services & Features Used

- VPC Peering
- AWS Transit Gateway
- AWS Site-to-Site VPN (Virtual Private Gateway + Customer Gateway)
- AWS Direct Connect
- VPC Gateway Endpoints (S3, DynamoDB)
- VPC Interface Endpoints (PrivateLink)
- AWS PrivateLink

## Screenshots

- `/screenshots/week3-architecture-diagram.png` — Complete 3-tier architecture diagram: OluTech-Production-VPC, 2 AZs, subnets, IGW, NAT GW, ALB, EC2 App Servers, RDS, Security Group chain, route table labels
- `/screenshots/github-diagram-commit.png` — GitHub repo showing diagram committed to `simeonprimordial/aws-solutions-architect-associate-saac03`

## Challenges & Blockers

See `notes/challenges.md` for full details.

- Understanding TGW route table isolation — how Dev VPC can connect to TGW but still not reach Prod
- The DX encryption limitation was counterintuitive — a dedicated private fibre is not encrypted by default
- Draw.io AWS icon set installation added ~10 minutes to lab setup
- The bonus peering diagram required adding route table labels for both VPCs to correctly show that peering alone does not route traffic

## Key Exam Traps to Remember

- **VPC Peering is NON-TRANSITIVE.** A↔B and B↔C does NOT give A↔C. For 3+ VPCs needing full connectivity: direct peering per pair, or Transit Gateway.
- **Peering alone routes nothing.** Update route tables in BOTH VPCs. One-way route = one-way traffic.
- **Direct Connect is NOT encrypted by default.** DX = private, not encrypted. Add a VPN over DX for both private path AND encryption.
- **VPN has two tunnels — monitor both.** AWS creates two tunnels. No automatic failover. Both must be monitored.
- **S3/DynamoDB Gateway Endpoints are free.** Never use NAT Gateway for private subnet S3 or DynamoDB access when a free Gateway Endpoint exists.

## Goal

Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
