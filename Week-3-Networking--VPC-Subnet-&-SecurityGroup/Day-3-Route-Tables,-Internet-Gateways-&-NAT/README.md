# Route Tables, Internet Gateways & NAT — Week 3 Day 3

## Topic

How network traffic actually knows where to go inside a VPC — route table mechanics, priority resolution, gateway types, NAT cost model, and Security Group chaining for a 3-tier application.

Days 1 and 2 built the outer structure: VPC, subnets, and the concept that publicness is determined by routing. Today answers the question those days left open: how exactly does routing work? The slides covered the four route table rules that the SAA-C03 tests hardest — longest prefix match, the immutable local route, the Main Route Table danger, and subnet fallback behaviour. The lab then applied all of it practically by creating Security Groups with chaining — the professional way to lock down inter-tier traffic. The OPay scenario from the slides made the real-world cost of misconfiguration concrete: a single IGW route left on the Main Route Table for six months exposed an internal payment API to the internet on a platform processing transactions for 35 million users.

## What I Learned

- **Route Table Anatomy** — Each route has a Destination CIDR and a Target. When a packet arrives, AWS checks ALL routes and selects the most specific match (longest prefix). `10.0.1.0/24` beats `10.0.0.0/16` beats `0.0.0.0/0` for a packet going to `10.0.1.5`.
- **Local Route — Immutable** — Every route table has `10.0.0.0/16 → local` pre-populated at VPC creation. Cannot be deleted, modified, or overridden. All traffic destined for any address within the VPC CIDR always stays inside the VPC — it never goes through the IGW even if `0.0.0.0/0 → IGW` is also present.
- **Main Route Table Danger** — Any subnet not explicitly associated with a custom route table inherits the Main Route Table. Never add an IGW route to the Main Route Table. Keep it local-only. A forgotten association falls back to the Main table — if Main is local-only, that is safe. If Main has an IGW route, that is a security incident.
- **Internet Gateway** — Enables bidirectional internet for public subnets. Must be created, attached to the VPC, and referenced in a route table entry — all three steps are required. Also performs NAT between EC2 private IPs and their public IPs for outbound traffic.
- **NAT Gateway** — Outbound-only internet for private subnet instances. Costs `$0.045/hour + $0.045/GB processed`. NOT free. For HA, deploy one NAT Gateway per AZ with separate per-AZ private route tables. Cross-AZ NAT traffic incurs an extra `$0.01/GB` data transfer charge.
- **NAT Gateway Protocol Limitation** — Supports TCP, UDP, and ICMP only. Does NOT support GRE or raw IPsec (ESP/AH). A VPN tunnel (PPTP/L2TP using GRE) will fail through a NAT Gateway. The fix is a NAT Instance with IP forwarding, not another NAT Gateway.
- **Egress-Only Internet Gateway** — IPv6 outbound-only. Since IPv6 addresses are globally unique, a regular IGW would allow both inbound and outbound IPv6. An Egress-Only IGW allows outbound IPv6 from private instances while blocking unsolicited inbound. Not relevant for IPv4. Not a substitute for an isolated data tier.
- **Security Group Chaining** — Instead of opening a Security Group to `0.0.0.0/0`, reference another Security Group as the source. `SG-WebServers` allows port 80 from `SG-LoadBalancer` only. `SG-Database` allows port `3306` from `SG-WebServers` only. This creates a chain where each tier only accepts traffic from the immediately preceding tier.
- **Three Conditions for Internet Connectivity** — IGW attached to VPC AND subnet route table has `0.0.0.0/0 → igw-xxx` AND the EC2 instance has a public or Elastic IP. All three must be true simultaneously. Any one missing breaks connectivity.
- **Route Table as Compliance Evidence** — The route table is the tamper-evident, auditable proof of network segmentation. Not the Security Group. Not the NACL. An AWS Config Rule monitoring route tables for unexpected `0.0.0.0/0` entries on data subnets is a stronger detective control than either.

## Hands-On Labs Completed

- Created `SG-LoadBalancer` with inbound HTTPS (port 443) and HTTP (port 80) from `0.0.0.0/0`
- Created `SG-WebServers` with inbound port 80 from `SG-LoadBalancer` (chained) and SSH port 22 from own IP (`x.x.x.x/32`)
- Created `SG-Database` with inbound MySQL port 3306 from `SG-WebServers` only — no internet access, no SSH
- Documented the Security Group chain in a table and drew the chaining diagram: `Internet → SG-LoadBalancer → SG-WebServers → SG-Database`
- **Bonus:** Created a custom NACL for the private subnet with explicit DENY rules for Telnet (port 23) and FTP (port 21), an ALLOW rule for MySQL (port 3306), and observed how NACL rule evaluation order changes the outcome

## AWS Services & Features Used

- Security Groups
- Network ACLs (NACLs)
- Route Tables
- Internet Gateway
- NAT Gateway
- Egress-Only Internet Gateway (conceptual)
- AWS Config (OPay scenario — compliance monitoring)
- Terraform (OPay scenario — IaC for route table tagging)

## Screenshots

- `/screenshots/sg-loadbalancer-rules.png` — `SG-LoadBalancer` showing HTTP and HTTPS inbound from `0.0.0.0/0`
- `/screenshots/sg-webservers-chained.png` — `SG-WebServers` showing port 80 inbound source as `SG-LoadBalancer` (security group reference)
- `/screenshots/sg-database-chained.png` — `SG-Database` showing port 3306 inbound source as `SG-WebServers` only
- `/screenshots/sg-chain-diagram.png` — Architecture diagram of the chaining flow with port labels on each arrow

## Challenges & Blockers

See `notes/challenges.md` for full details.

- Setting the SSH source to `x.x.x.x/32` correctly — had to look up my IP from whatismyip.com and add the `/32` suffix manually
- Security Group chaining source dropdown was not obvious — had to know to select "Custom" then type the SG name to reference it
- NACL bonus challenge: rule evaluation order bit me when I put the ALLOW rule at a higher number than the DENY — traffic was blocked before the allow could fire
- Understanding that the IGW performs NAT for public IPs (private → public translation) while the NAT Gateway performs NAT for private subnets (private → EIP translation) — two different NAT mechanisms for different use cases

## Key Exam Traps to Remember

- **NAT Gateway is NOT free.** `$0.045/hour + $0.045/GB`. One NAT Gateway running all month with 100GB/day costs approximately `$166/month`. For HA across two AZs, double that.
- **Three conditions for internet connectivity** — IGW attached + route table entry + public/Elastic IP. Any one missing and traffic does not flow. Exam connectivity debugging always traces these three.
- **Unassociated subnets use the Main Route Table.** Keep Main as local-only. A forgotten association is safe if Main has no IGW route. It is a security incident if Main has an IGW route.
- **NAT Gateway supports TCP, UDP, ICMP only.** GRE and raw IPsec fail through NAT Gateway. The solution is a NAT Instance with IP forwarding, not replacing the NAT Gateway.
- **The local route always wins for VPC-internal traffic.** A packet from `10.0.1.5` to `10.0.21.8` (RDS) always takes the `local` route — the IGW route `0.0.0.0/0` is never matched because `10.0.0.0/16` is more specific.

## Goal

Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
