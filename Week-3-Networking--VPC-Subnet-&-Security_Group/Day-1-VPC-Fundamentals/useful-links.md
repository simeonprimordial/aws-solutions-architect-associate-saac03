# Useful Links — Week 3 Day 1: VPC Fundamentals

## Official AWS Documentation

**Amazon VPC User Guide**
https://docs.aws.amazon.com/vpc/latest/userguide/
The authoritative reference for every VPC feature. Read the "Getting Started with Amazon VPC" and "Security" chapters before anything else. The section on route tables is essential for understanding public vs private subnets.

**VPC Security — Security Groups and NACLs**
https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Security.html
The official AWS comparison of Security Groups and NACLs with worked examples showing stateful vs stateless behaviour. The section on ephemeral ports (1024–65535) for NACL outbound rules is a specific SAA-C03 exam requirement — read this before the exam.

**NAT Gateway**
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html
Full NAT Gateway documentation including pricing, high availability patterns (one per AZ), bandwidth limits, and the difference between public and private NAT Gateways.

**VPC Endpoints**
https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints.html
Gateway Endpoints (S3, DynamoDB — free) vs Interface Endpoints (PrivateLink — billed). The use cases and limitations are tested on SAA-C03.

**VPC Peering**
https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html
Includes the non-transitive routing explanation, peering limitations, and when to use Transit Gateway instead.

---

## Tools

**CIDR Calculator — subnet-calculator.com**
https://www.subnet-calculator.com/
Enter any CIDR block and get the total IP count, usable host count, network address, and broadcast address. Essential for practising SAA-C03 CIDR math questions before the exam.

**cidr.xyz — Visual CIDR Visualiser**
https://cidr.xyz/
Interactive CIDR notation tool with a visual representation of the IP block. Useful for understanding how prefix length translates to address ranges.

---

## Best Practices & Architecture

**AWS VPC Subnet Design Best Practices — Networking Blog**
https://aws.amazon.com/blogs/networking-and-content-delivery/vpc-subnet-design-best-practices/
AWS's own opinionated guide to VPC subnet sizing, IP address planning for growth, and recommended patterns for multi-AZ, multi-tier architectures.

**AWS Well-Architected Framework — Security Pillar: Infrastructure Protection**
https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/infrastructure-protection.html
Covers the VPC design principles that the Well-Architected Framework recommends for production workloads — specifically multi-tier subnet isolation, Security Group design, and NACL usage.

---

## Exam Preparation

**SAA-C03 Exam Guide — AWS**
https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
Domain 1 (Resilient Architectures) and Domain 2 (Secure Architectures) both require deep VPC knowledge. Task Statements 1.1 (design for availability), 2.2 (design secure network topologies), and 2.3 (design appropriate access controls) are all VPC-heavy.

**Tutorials Dojo — SAA-C03 Practice Questions (VPC)**
https://tutorialsdojo.com/aws-vpc/
Community-maintained cheat sheet and practice questions for VPC concepts. Strong on the Security Group vs NACL distinction and subnet routing scenarios.

---

## Nigerian Regulatory Context

**CBN Risk-Based Cybersecurity Framework — Network Security Requirements**
https://www.cbn.gov.ng/out/2022/bspd/exposure_draft_-_revised_risk-based_cybersecurity_framework_for_deposit_money_banks_and_payment_service_banks.pdf
CBN's cybersecurity framework for banks and payment service banks. Section on network security mandates segmentation between production systems, internal networks, and internet-facing components — exactly what the 3-tier VPC architecture implements.

**NDPC — Nigeria Data Protection Commission**
https://ndpc.gov.ng/
The NDPC enforces data protection requirements for Nigerian businesses handling personal data. Network isolation of databases (private data subnets with no internet route) is a practical implementation of the data minimisation and security principles in the Nigeria Data Protection Act.
