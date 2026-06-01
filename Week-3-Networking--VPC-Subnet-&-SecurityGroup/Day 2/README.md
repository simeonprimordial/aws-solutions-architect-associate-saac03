# Subnets — Public vs Private — Week 3 Day 2

## Topic

The routing decisions that define whether a subnet is truly public, private, or completely isolated — and why this distinction is the most consequential architectural choice in any AWS deployment.

Day 1 built the VPC and the four subnets. Today I made them functional by attaching an Internet Gateway, creating and associating route tables, and building the routing logic that actually separates public from private. The day goes deeper into subnet design: the three-tier layout, multi-AZ CIDR planning, why a Security Group is never sufficient proof of isolation for a compliance auditor, and the specific subnet sizing traps that appear on the SAA-C03. The Flutterwave-style PCI-DSS scenario made the regulatory implications concrete — the route table is the audit evidence, not the Security Group.

## What I Learned

- **Public Subnet** — A subnet whose route table contains `0.0.0.0/0 → IGW`. Resources in it can communicate with the internet if they also have a public or Elastic IP. Hosts: ALB, Bastion Host, NAT Gateway only. Nothing else belongs in a public subnet in production.
- **Private Subnet** — A subnet with NO direct IGW route. May have a NAT Gateway route for outbound-only internet access. Hosts: EC2 app servers, ECS tasks, Lambda ENIs. Isolated from inbound internet traffic by architecture, not just Security Group rules.
- **Isolated Subnet** — A private subnet with only the `local` VPC route — no `0.0.0.0/0` entry at all, not even a NAT Gateway route. Hosts: RDS, ElastiCache. Zero internet path in either direction. This is the route table configuration that satisfies CBN, SEC, and PCI-DSS compliance requirements for financial data isolation.
- **Auto-Assign Public IP** — Subnet-level setting that assigns a public IPv4 to every EC2 launched there. Must be ON for public subnets (when instances need direct public IPs). Must be OFF for all private and data subnets. An auto-assigned IP is NOT an Elastic IP — it changes every stop/start cycle.
- **Subnet AZ Binding** — Every subnet is bound to exactly one AZ at creation. This cannot be changed. A subnet cannot span AZs. High availability requires separate subnets per AZ with resources deployed across them.
- **Default VPC Subnets** — All public. Auto-Assign IP enabled. Route to IGW. Any EC2 launched in the default VPC gets a public IP immediately. No production workload should ever run in the default VPC.
- **Custom Subnet Defaults** — New custom subnets are private by default. Auto-Assign IP disabled. No IGW route. Nothing is exposed unless you deliberately configure it.
- **Multi-AZ CIDR Numbering Convention** — Public tier: `10.0.1.x–10.0.9.x`. App tier: `10.0.10.x–10.0.19.x`. Data tier: `10.0.20.x–10.0.29.x`. Management: `10.0.100.x+`. This makes subnets instantly identifiable by their IP range alone.
- **Route Table as Compliance Evidence** — A route table with only the `local` route on a data subnet is tamper-evident, auditable proof that no internet path exists. A Security Group that blocks all traffic is an operational control, not architectural evidence — it can be changed by anyone with IAM access and leaves no structural proof.
- **Management Subnet** — A small `/28` subnet (11 usable IPs) for Bastion Hosts only. SSH access locked to static IPs. Sessions logged via AWS Systems Manager Session Manager to S3.

## Hands-On Labs Completed

- Created and attached Internet Gateway `OluTech-IGW` to `OluTech-Production-VPC`
- Created `Public-Route-Table` with routes: `10.0.0.0/16 → local` and `0.0.0.0/0 → OluTech-IGW`
- Associated `Public-Subnet-AZ-A` and `Public-Subnet-AZ-B` with `Public-Route-Table`
- Verified private subnets remain on the main route table with only the `local` route — no internet path
- Drew and exported Excalidraw traffic flow diagram showing packet paths through the VPC, labelled with route table entries
- **Bonus:** Created a NAT Gateway in `Public-Subnet-AZ-A`, added `0.0.0.0/0 → NAT GW` to the private route table, explained the difference from the IGW route, deleted NAT Gateway after 15 minutes to stay free tier

## AWS Services & Features Used

- Amazon VPC
- Internet Gateway (IGW)
- Route Tables
- Subnet Route Table Associations
- NAT Gateway
- Elastic IP (EIP)
- VPC Flow Logs (covered conceptually)
- AWS Systems Manager Session Manager (Bastion logging)

## Screenshots

- `/screenshots/olutech-igw-attached.png` — Internet Gateway console showing `OluTech-IGW` with status `Attached` to `OluTech-Production-VPC`
- `/screenshots/public-route-table-routes.png` — `Public-Route-Table` Routes tab showing `10.0.0.0/16 → local` and `0.0.0.0/0 → igw-xxxxxxxx`
- `/screenshots/public-subnets-associated.png` — Subnet associations tab showing both public subnets linked to `Public-Route-Table`
- `/screenshots/private-subnets-local-only.png` — Main route table showing only the `local` route, with private subnets associated — confirming no internet path
- `/screenshots/traffic-flow-diagram.png` — Excalidraw diagram showing full packet flow: Internet → IGW → Public Subnet → ALB → Private-App → RDS, with route table entries labelling each arrow

## Challenges & Blockers

See `notes/challenges.md` for full details.

- Attached the IGW to the wrong VPC on first attempt — had to detach and reattach
- Accidentally deleted the `local` route from the public route table while editing — had to recreate the route table from scratch
- The NAT Gateway bonus lab took longer than expected because the Elastic IP allocation step isn't obvious — it is a prerequisite before the NAT GW can be created
- Confusion between the main route table and the custom route table — understanding which subnets land on which by default took a second read of the console

## Key Exam Traps to Remember

- **Route table is the only thing that determines public vs private.** Name, Auto-Assign IP setting, Security Groups — all irrelevant. `0.0.0.0/0 → IGW` in the route table is the one and only definition of a public subnet.
- **A /24 subnet has 251 usable IPs — never 256.** AWS reserves 5. A customer needing 250 EC2 instances cannot use a `/24` — they need a `/23` (507 usable).
- **Security Groups cannot replace route table isolation for compliance.** An auditor checking CBN or PCI-DSS requirements will inspect the route table. A Security Group blocking all traffic is an operational control that any authorised IAM user can modify. A route table with no internet entry is structural proof.
- **Default VPC subnets are ALL public.** Auto-Assign IP is enabled, IGW route exists. Never use for production. Any EC2 launched there gets a public IP by default.
- **Subnets are single-AZ only.** "A subnet spanning multiple AZs" is never a valid answer. Create separate subnets per AZ for high availability.

## Goal

Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
