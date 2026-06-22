# Useful Links — Week 3 Day 5: VPC Peering & Connectivity

## Official AWS Documentation

**VPC Peering User Guide**
https://docs.aws.amazon.com/vpc/latest/peering/
Complete reference for creating peering connections, configuring route tables in both VPCs, Security Group referencing across peered VPCs, CIDR overlap restrictions, and cross-account/cross-region peering. The "Routing for VPC peering" section is essential — it covers exactly which route entries must be added and why one-way routes produce one-way traffic.

**Transit Gateway User Guide**
https://docs.aws.amazon.com/vpc/latest/tgw/
Full TGW documentation: creating attachments (VPC, VPN, DX, TGW peering), TGW route tables and associations, route table isolation for security boundaries, bandwidth limits, and the multi-region connectivity pattern. The "How Transit Gateway works" section explains attachment routing clearly.

**AWS Site-to-Site VPN**
https://docs.aws.amazon.com/vpn/latest/s2svpn/
Complete S2S VPN reference: Customer Gateway and Virtual Private Gateway creation, the two-tunnel architecture, tunnel monitoring with CloudWatch, and failover configuration. The "Monitoring your Site-to-Site VPN connection" section covers the CloudWatch metrics needed to detect tunnel failures.

**AWS Direct Connect**
https://docs.aws.amazon.com/directconnect/latest/UserGuide/
Full DX documentation: connection types (dedicated vs hosted), port speeds, Direct Connect Gateways for multi-VPC connectivity, encryption options (DX + MACsec, or DX + VPN), and the DX + VPN combination for compliance workloads. The "Encryption in transit" section explicitly addresses the not-encrypted-by-default limitation.

**VPC Endpoints — AWS PrivateLink**
https://docs.aws.amazon.com/vpc/latest/privatelink/
Complete reference for Gateway Endpoints (S3, DynamoDB — free), Interface Endpoints (PrivateLink — paid), endpoint policies, DNS configuration for Interface Endpoints, and the PrivateLink mechanism for SaaS vendor private service delivery.

---

## Architecture & Decision References

**AWS Well-Architected Framework — Network Connectivity**
https://docs.aws.amazon.com/wellarchitected/latest/framework/
The Reliability pillar covers multi-VPC connectivity resilience patterns — specifically TGW HA and DX failover with VPN. The Security pillar covers VPC Endpoint usage for keeping traffic off the internet.

**AWS Networking — Decision Guide**
https://aws.amazon.com/vpc/
The official VPC page includes connectivity decision guides and links to each connectivity service. Useful as a quick reference for the connectivity decision matrix.

**AWS Reference Architecture — Hybrid Connectivity**
https://aws.amazon.com/solutions/implementations/
Search for "hybrid connectivity" — AWS publishes reference architectures showing TGW + DX + VPN patterns for enterprise connectivity. The financial services reference architecture shows the pattern most applicable to Nigerian banks and fintechs.

---

## Cost References

**Transit Gateway Pricing**
https://aws.amazon.com/transit-gateway/pricing/
`$0.05/hr` per attachment + `$0.02/GB` processed. Calculate per-region costs for multi-VPC deployments. For a 5-VPC deployment in `eu-west-1`: `5 × $0.05 × 720 = $180/month` in attachment fees before data processing charges.

**AWS Site-to-Site VPN Pricing**
https://aws.amazon.com/vpn/pricing/
`$0.05/hr` per VPN connection = `$36/month`. Plus data transfer charges. For low-volume on-premises connectivity, VPN is highly cost-effective.

**AWS Direct Connect Pricing**
https://aws.amazon.com/directconnect/pricing/
Port hour fees by connection speed (1 Gbps: `$216/month`). Data transfer out from AWS via DX is significantly cheaper than standard data transfer rates — the breakeven point where DX saves money over VPN/internet is roughly 10–20TB/month depending on region.

**VPC Gateway Endpoint Pricing**
https://aws.amazon.com/privatelink/pricing/
S3 and DynamoDB Gateway Endpoints: `$0`. No per-hour, no per-GB cost. Free. End of discussion.

---

## Diagramming Tools

**draw.io (app.diagrams.net)**
https://app.diagrams.net/
Free, browser-based, no signup. Built-in AWS icon library. Exports to PNG, PDF, SVG, XML. The standard tool for AWS architecture diagrams in portfolios. AWS icons are available from the Shape panel — search "AWS".

**Official AWS Architecture Icons**
https://aws.amazon.com/architecture/icons/
The canonical AWS icon set for all services and resources. Download the full icon pack for use in draw.io, Visio, PowerPoint, or any diagram tool. Using official icons in portfolio diagrams signals professional familiarity with AWS.

**Lucidchart — AWS Integration**
https://www.lucidchart.com/pages/solutions/aws-diagram-tool
Commercial alternative to draw.io with native AWS shape libraries and team collaboration. Has a free tier sufficient for portfolio diagrams.

---

## Nigerian Context

**CBN Risk-Based Cybersecurity Framework — Connectivity Requirements**
https://www.cbn.gov.ng/out/2022/bspd/exposure_draft_-_revised_risk-based_cybersecurity_framework_for_deposit_money_banks_and_payment_service_banks.pdf
Sections on network architecture and data transmission security. CBN requires encrypted channels for data transmission between financial systems — the DX + VPN combination satisfies both the private path requirement and the encryption requirement. Transit Gateway route table isolation satisfies the network segmentation requirement for separating production from development environments.

**AWS Direct Connect Locations — Africa**
https://aws.amazon.com/directconnect/locations/
AWS Direct Connect locations include partners in South Africa (Johannesburg, Cape Town) and growing presence in West Africa. Nigerian banks currently use DX partners in Lagos, typically through regional AWS partners. The latency from Lagos to `af-south-1` (Cape Town) or `eu-west-1` (Ireland) via DX is significantly lower than VPN over the Nigerian internet infrastructure.
