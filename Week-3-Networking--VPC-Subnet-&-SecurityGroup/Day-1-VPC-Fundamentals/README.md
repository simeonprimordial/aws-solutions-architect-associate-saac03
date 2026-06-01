# VPC Fundamentals — Week 3 Day 1

## Topic

Building the network foundation every AWS workload runs on: Virtual Private Cloud, subnets, routing, gateways, and security layers.

This day covers the Amazon Virtual Private Cloud from first principles. I built a custom VPC from scratch — `OluTech-Production-VPC` — with four subnets across two Availability Zones, separated into public and private tiers. The session goes well beyond the console steps: CIDR math, route table logic, the stateful/stateless distinction between Security Groups and NACLs, and the production 3-tier architecture pattern that appears in almost every SAA-C03 scenario question.

VPC is where AWS architecture starts. Every service I touch from this point forward — EC2, RDS, ECS, Lambda with private endpoints — sits inside a VPC. Getting this mental model right now means every future day builds on a solid foundation. The slides framed it well: the difference between a junior cloud engineer and a mid-level one is largely VPC.

## What I Learned

- **VPC** — Your own logically isolated, software-defined network inside an AWS Region. One VPC spans all Availability Zones in a Region automatically. Nothing enters or leaves unless you explicitly allow it.
- **CIDR Blocks** — The IP address notation that defines your network range. A `/16` gives 65,536 addresses. A `/24` gives 256 (251 usable after AWS reserves 5). A smaller prefix number means a larger range: `/8 > /16 > /24 > /28`.
- **AWS Reserved IPs** — AWS reserves 5 IPs per subnet: `.0` (network address), `.1` (VPC router), `.2` (DNS), `.3` (future use), `.255` (broadcast). This is why a `/24` subnet gives 251 usable hosts, not 256.
- **Subnets** — Each subnet is tied to exactly ONE Availability Zone. A subnet is public because its route table has a `0.0.0.0/0` route to an Internet Gateway. Not because of its name. Not because auto-assign public IP is enabled. The route table is the only thing that matters.
- **Route Tables** — The set of rules directing where traffic from a subnet goes. Every subnet is associated with exactly one route table. Public subnets point `0.0.0.0/0` to the IGW. Private subnets point `0.0.0.0/0` to the NAT Gateway. Isolated subnets have no `0.0.0.0/0` entry at all.
- **Internet Gateway (IGW)** — AWS-managed, horizontally scaled, redundant. Attaching an IGW to a VPC and routing a subnet's traffic to it is what makes that subnet public. Without an IGW route, the subnet is private regardless of any other setting.
- **NAT Gateway** — Lives in a public subnet with an Elastic IP. Private subnet instances route outbound internet traffic through it to download patches, reach external APIs. The NAT Gateway blocks all unsolicited inbound traffic by design — that is the entire security point of it.
- **Security Groups vs NACLs** — The most-tested VPC distinction on SAA-C03. Security Groups are stateful and instance-level: add an inbound allow rule and the return traffic is automatically permitted. NACLs are stateless and subnet-level: you must explicitly allow both directions, including outbound ephemeral ports `1024–65535` for response traffic.
- **VPC Peering** — A direct private connection between two VPCs enabling private-IP routing. Non-transitive: if VPC A peers B and B peers C, A cannot reach C through B. Three-VPC full connectivity needs three peering connections or AWS Transit Gateway.
- **Production 3-Tier Pattern** — ALB in public subnet (receives internet traffic via IGW) → EC2 app servers in private app subnet (no public IPs, reachable only via ALB Security Group) → RDS in private data subnet (no internet route of any kind, reachable only on port 5432 from app subnet Security Group).

## Hands-On Labs Completed

- Planned the VPC on Excalidraw before touching the console — CIDR ranges, AZ placement, public/private tiers labelled
- Created `OluTech-Production-VPC` with CIDR `10.0.0.0/16`
- Created four subnets: `Public-Subnet-AZ-A` (`10.0.1.0/24`), `Public-Subnet-AZ-B` (`10.0.2.0/24`), `Private-Subnet-App-AZ-A` (`10.0.10.0/24`), `Private-Subnet-DB-AZ-B` (`10.0.20.0/24`)
- Enabled auto-assign public IPv4 on both public subnets; left private subnets with auto-assign off
- Verified all four subnets in the console filtered by VPC
- Completed and exported Excalidraw architecture diagram with CIDR ranges, AZ labels, and VPC boundary box
- **Bonus:** Compared `OluTech-Production-VPC` to the AWS default VPC — documented 5 differences and explained why the default VPC must never be used in production

## AWS Services & Features Used

- Amazon VPC
- VPC Subnets
- Route Tables
- Internet Gateway
- NAT Gateway
- Elastic IP (EIP)
- Security Groups
- Network ACLs (NACLs)
- VPC Peering
- VPC Endpoints (Gateway and Interface types)

## Screenshots

- `/screenshots/olutech-vpc-created.png` — VPC console showing `OluTech-Production-VPC` with CIDR `10.0.0.0/16`
- `/screenshots/four-subnets-list.png` — Subnets list filtered by VPC showing all four subnets with correct CIDRs
- `/screenshots/public-subnet-auto-assign.png` — Subnet settings showing auto-assign public IPv4 enabled on `Public-Subnet-AZ-A`
- `/screenshots/vpc-architecture-diagram.png` — Completed Excalidraw diagram with VPC boundary, four subnets, AZ labels, CIDR ranges on every component

## Challenges & Blockers

See `notes/challenges.md` for full details.

- Confused about why a subnet named "Public" wasn't actually public until I attached the IGW route
- Missed the 5 reserved IPs per subnet on first read — affects CIDR sizing decisions
- Struggled with the NACL ephemeral port range requirement — counterintuitive coming from Security Groups
- Default VPC comparison revealed settings I didn't expect (default VPC is a /16 too — the surprises are in the routing and subnet defaults)

## Key Exam Traps to Remember

- **A subnet is public because of its route table — never because of its name or auto-assign setting.** If there is no `0.0.0.0/0 → IGW` route, the subnet is private. Full stop.
- **Security Groups are STATEFUL. NACLs are STATELESS.** Adding an inbound allow in a Security Group automatically allows the response. NACLs require explicit outbound rules for ephemeral ports `1024–65535` or response traffic is dropped.
- **NAT Gateway enables OUTBOUND internet for private subnet instances. It does NOT allow inbound connections.** Any exam option saying "add a NAT Gateway so internet users can reach the application" is wrong.
- **VPC Peering is NON-TRANSITIVE.** A→B and B→C does not give A→C. Full mesh needs three connections or Transit Gateway.
- **VPC is Regional. Subnets are Zonal.** A VPC spans all AZs in a Region. Each subnet exists in exactly one AZ. No subnet ever spans multiple AZs — that option is always wrong on the exam.
- **VPC Endpoints connect to AWS services — not to the public internet.** A VPC Endpoint for S3 keeps S3 traffic off the internet. It cannot replace a NAT Gateway for general outbound internet access.

## Goal

Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
