# Useful Links — Week 3 Day 2: Subnets — Public vs Private

## Official AWS Documentation

**Amazon VPC — Configure Subnets**
https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html
Complete reference for subnet creation, CIDR allocation, AZ binding, route table association, and NACL attachment. The "Subnet sizing for IPv4" section explains the 5 reserved IPs with the exact calculation for usable hosts per subnet size.

**AWS VPC Subnet Sizing Guide**
https://docs.aws.amazon.com/vpc/latest/userguide/subnet-sizing.html
AWS's own guidance on sizing subnets for IPv4 and IPv6. Covers minimum viable subnet sizes for EKS node groups (`/28`), ECS tasks, and RDS deployments. Includes growth considerations — critical for enterprise VPC design where CIDR changes are expensive.

**Internet Gateways — Official Documentation**
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html
Full IGW documentation: creation, attachment, route table configuration, and the mechanics of how the IGW performs NAT for instances with public IPs. The "Enable internet access" step-by-step is useful for confirming the exact console flow.

**Route Tables — Official Documentation**
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html
Everything about route tables: the local route, custom route tables, main route table behaviour, subnet associations, and route priority. The section on route table associations is directly relevant to today's lab — how new subnets inherit the main route table.

**VPC Security Best Practices — Subnet Isolation Patterns**
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-best-practices.html
Official AWS guidance on subnet security: keeping data resources in isolated subnets, layered security (SG + NACL + routing), enabling VPC Flow Logs for traffic visibility. Directly relevant to the compliance isolation discussion from today's slides.

---

## Operational Tools

**VPC Flow Logs — Subnet Traffic Monitoring**
https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html
VPC Flow Logs capture metadata about traffic accepted and rejected at subnet and ENI level. This is the CloudTrail equivalent for network traffic — essential for security auditing, CBN and PCI-DSS compliance reporting, and debugging connectivity issues. Can be delivered to CloudWatch Logs or S3.

**AWS Reachability Analyzer**
https://docs.aws.amazon.com/vpc/latest/reachabilityanalyzer/what-is-reachability-analyzer.html
A network diagnostics tool that analyses paths between two AWS network resources. Useful for verifying that your private subnets have no route to the internet, and that your public subnets can reach the IGW. Produces a detailed path report useful for troubleshooting and compliance evidence.

---

## Architecture References

**AWS Architecture Blog — VPC Subnet Design Best Practices**
https://aws.amazon.com/blogs/networking-and-content-delivery/vpc-subnet-design-best-practices/
AWS's opinionated guide to subnet sizing, CIDR planning for growth, and the recommended multi-AZ, multi-tier architecture. The numbering convention (public `10.0.1–9.x`, app `10.0.10–19.x`, data `10.0.20–29.x`) is from this blog post.

**AWS Well-Architected Framework — Security Pillar, Infrastructure Protection**
https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/infrastructure-protection.html
Covers network protection at multiple layers: subnet isolation, Security Group design, NACL configuration. The framework explicitly states that data stores should be in isolated subnets with no internet routing — this is the Well-Architected principle behind the 3-tier pattern.

---

## Exam Preparation

**SAA-C03 Exam Guide — Domains 1 and 2**
https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
Domain 2, Task Statement 2.2 covers designing secure network architectures — specifically which resource types belong in which subnet tier, how route tables enforce security isolation, and the difference between Security Group and NACL control layers. Download and annotate this PDF.

**Tutorials Dojo — SAA-C03 VPC and Subnet Practice Questions**
https://tutorialsdojo.com/aws-vpc/
Strong community-maintained cheat sheet and practice question bank. Particularly useful for the subnet sizing calculation questions (reserved IPs, usable hosts per CIDR) and the route-table-determines-publicness scenarios.

---

## Nigerian Regulatory Context

**CBN — Risk-Based Cybersecurity Framework**
https://www.cbn.gov.ng/out/2022/bspd/exposure_draft_-_revised_risk-based_cybersecurity_framework_for_deposit_money_banks_and_payment_service_banks.pdf
Section on network security: network segmentation between customer-facing systems and financial data backends is a mandatory control. The route table configuration for data subnets (local route only) is the AWS implementation of this requirement. The route table printout is your audit evidence.

**PCI-DSS v4.0 — Network Security Controls**
https://www.pcisecuritystandards.org/document_library/
PCI-DSS Requirement 1 mandates installation and maintenance of network security controls. Requirement 1.3 covers restricting inbound and outbound traffic to what is necessary. The isolated data subnet with no internet route satisfies both. Relevant for any Nigerian fintech processing card payments — Flutterwave, Paystack, and similar platforms are subject to PCI-DSS alongside CBN oversight.
