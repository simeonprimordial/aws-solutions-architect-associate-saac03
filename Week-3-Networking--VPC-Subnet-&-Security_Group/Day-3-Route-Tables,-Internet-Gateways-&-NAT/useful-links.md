# Useful Links — Week 3 Day 3: Route Tables, Internet Gateways & NAT

## Official AWS Documentation

**VPC Route Tables — Official Documentation**
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html
Complete reference covering route entry configuration, Main Route Table behaviour, subnet associations, route priority resolution (longest prefix match), and route propagation for VPN/Direct Connect. The "Route priority" section is essential exam reading — it explains exactly how conflicting routes are resolved.

**Internet Gateways**
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html
Full IGW documentation: creation, attachment, the NAT translation behaviour for public IPs, the three conditions required for internet connectivity, and the difference between an auto-assigned public IP and an Elastic IP.

**NAT Gateways — Configuration & Pricing**
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html
NAT Gateway setup, Elastic IP requirements, multi-AZ HA deployment pattern, bandwidth limits (up to 100 Gbps), protocol support (TCP/UDP/ICMP only — no GRE, no IPsec), and the full pricing model. Read the "NAT gateway use cases" section for the exam scenarios on when to use NAT Gateway vs NAT Instance.

**NAT Instances — When to Use Instead of NAT Gateway**
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html
Configuration for NAT Instance EC2 (source/destination check disabled, IP forwarding enabled), performance characteristics, and the comparison table against NAT Gateway. The key exam scenario: cost optimisation for low-traffic workloads and protocol support for GRE/IPsec.

**Egress-Only Internet Gateway — IPv6 Outbound**
https://docs.aws.amazon.com/vpc/latest/userguide/egress-only-internet-gateway.html
Why IPv6 needs a separate gateway mechanism (all IPv6 addresses are globally unique — no NAT), how Egress-Only IGW provides outbound-only IPv6 for private resources, and the SAA-C03 context where this appears.

**Security Groups for VPCs**
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html
Full Security Group reference including: stateful behaviour, the default allow-all outbound rule, Security Group chaining (referencing SGs as inbound sources), and the 5-SG per ENI limit. The "Security group rules" section covers the SG-as-source pattern.

---

## Operational Tools

**AWS Reachability Analyzer**
https://docs.aws.amazon.com/vpc/latest/reachabilityanalyzer/what-is-reachability-analyzer.html
Analyses the network path between two AWS resources and produces a detailed hop-by-hop report. Useful for debugging connectivity failures (which of the three conditions is missing) and for compliance evidence (showing there is no path from the internet to RDS). Free to run; each analysis costs per execution.

**VPC Flow Logs**
https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html
Flow Logs capture metadata for every packet: accepted, rejected, source IP, destination IP, port, protocol. Delivered to CloudWatch Logs or S3. Essential for debugging silent NACL drops (where NACL evaluation order is blocking traffic without any error message at the application layer).

**AWS Config — Route Table Monitoring Rule**
https://docs.aws.amazon.com/config/latest/developerguide/vpc-default-security-group-closed.html
AWS Config can monitor route tables and alert when unexpected `0.0.0.0/0` entries appear on isolated subnets. This is the automated compliance control from the OPay scenario — stronger than a NACL because it detects route table changes in real time rather than blocking traffic after the fact.

---

## Cost References

**AWS VPC Pricing — NAT Gateway**
https://aws.amazon.com/vpc/pricing/
Current per-region pricing for NAT Gateways, including the per-hour and per-GB rates. Bookmark this and use it to calculate costs for architecture proposals. The pricing varies by region — `af-south-1` (Cape Town) and `eu-west-1` (Ireland) have slightly different rates.

**AWS Cost Calculator**
https://calculator.aws/pricing/2/home
Build a monthly cost estimate including NAT Gateway costs. Input: number of NAT Gateways, hours per month, GB of data processed. Useful for architecture design reviews and for the SAA-C03 cost-optimisation question type.

---

## Exam Preparation

**SAA-C03 Exam Guide — Domains 1 and 2**
https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
Domain 1, Task Statement 1.4 covers designing cost-optimised architectures — NAT Gateway vs NAT Instance is a direct test here. Domain 2, Task Statement 2.2 covers secure network topology design — route table isolation for compliance, Security Group chaining, and the three-tier pattern.

**Tutorials Dojo — Route Tables and NAT Practice Questions**
https://tutorialsdojo.com/aws-vpc/
Strong question bank for route priority scenarios (which route wins), NAT Gateway HA patterns, and the connectivity troubleshooting flow (three conditions). Useful for drilling the pattern recognition needed to answer quickly under exam time pressure.

---

## Nigerian Regulatory Context

**CBN IT Governance Framework — Network Segmentation Requirements**
https://www.cbn.gov.ng/out/2022/bspd/exposure_draft_-_revised_risk-based_cybersecurity_framework_for_deposit_money_banks_and_payment_service_banks.pdf
Requires documented evidence of network segmentation — specifically that financial data backends have no direct or indirect internet connectivity. Route table isolation (isolated-rt with local-only) is the AWS implementation. AWS Config monitoring of route tables is the automated compliance evidence. The OPay scenario from today's slides is a direct illustration of what failing this audit looks like.

**Terraform AWS Provider — Route Table Resources**
https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/route_table
IaC reference for managing route tables with mandatory tagging (`Name`, `Environment`, `Tier`, `Owner`). The OPay remediation used Terraform to manage all route tables and enforce tagging standards. Using IaC means route table configurations are version-controlled, reviewable, and auditable — a significant improvement over manual console changes.
