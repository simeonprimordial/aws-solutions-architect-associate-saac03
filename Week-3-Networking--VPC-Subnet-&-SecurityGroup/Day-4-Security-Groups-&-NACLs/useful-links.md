# Useful Links — Week 3 Day 4: Security Groups & NACLs

## Official AWS Documentation

**Security Groups for VPCs**
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html
Complete reference: rule anatomy, SG referencing (using SG IDs as source/destination), stateful behaviour, default inbound/outbound, maximum rules per group (60 inbound, 60 outbound by default), and the 5-SG-per-ENI limit. The "Security group rules" section covers all rule components.

**Network ACLs — Official Documentation**
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html
Complete NACL reference: rule numbering, evaluation order, the difference between default and custom NACLs, ephemeral port ranges by OS (Linux: `32768–60999`, Windows: `49152–65535`, NAT Gateway: `1024–65535`), and worked examples showing stateless rule requirements for common protocols.

**VPC Flow Logs — Troubleshooting SG and NACL Issues**
https://docs.aws.amazon.com/vpc/latest/userguide/flow-logs.html
VPC Flow Logs record ACCEPT and REJECT decisions at ENI and subnet levels. REJECT at the subnet boundary = NACL blocked. REJECT at the ENI = Security Group blocked. The log format section shows the fields: `srcaddr`, `dstaddr`, `srcport`, `dstport`, `protocol`, and `action` (ACCEPT/REJECT). Essential for diagnosing silent traffic drops.

**AWS Systems Manager Session Manager**
https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html
Full Session Manager documentation: setup requirements (SSM Agent + IAM instance profile with `AmazonSSMManagedInstanceCore`), browser-based and CLI access, session logging to S3/CloudWatch, and the VPC Endpoint requirement for fully isolated instances. The "Setting up Session Manager" section covers the exact IAM and networking prerequisites.

---

## Security Tooling

**AWS Security Hub — EC2 Security Controls**
https://docs.aws.amazon.com/securityhub/latest/userguide/ec2-controls.html
Security Hub EC2 controls include automated checks for overly permissive Security Groups — specifically flagging SGs that allow unrestricted access (`0.0.0.0/0`) on high-risk ports: SSH (22), RDP (3389), MySQL (3306), PostgreSQL (5432). Enabling this gives automatic detection of the Interswitch-finding class of misconfiguration.

**AWS Config — Security Group Rule Monitoring**
https://docs.aws.amazon.com/config/latest/developerguide/vpc-sg-open-only-to-authorized-ports.html
AWS Config rule `vpc-sg-open-only-to-authorized-ports` flags Security Groups that allow unrestricted inbound access on specified ports. Combine with automatic remediation via Lambda to automatically restrict any SG that opens port 22 or 3306 to `0.0.0.0/0`.

**VPC Reachability Analyzer**
https://docs.aws.amazon.com/vpc/latest/reachabilityanalyzer/what-is-reachability-analyzer.html
Produces a hop-by-hop path analysis between two VPC resources — including which Security Group rule or NACL rule is allowing or blocking traffic at each step. The best tool for pre-deployment verification that a Security Group chain (ALB → EC2 → RDS) is configured correctly before instances are launched.

---

## Exam Preparation

**SAA-C03 Exam Guide — Domain 2 Secure Network Architectures**
https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
Domain 2, Task Statement 2.2 explicitly lists Security Group design, NACL design, and the stateful/stateless distinction as exam content. Download and annotate. The ephemeral port requirement for NACLs and Security Group referencing are both named.

**Tutorials Dojo — Security Groups and NACLs Practice Questions**
https://tutorialsdojo.com/aws-vpc/
Strong practice question bank for the stateful/stateless distinction, NACL rule order scenarios, ephemeral port questions, and Security Group referencing. Run through these before the exam — the pattern recognition from repeated practice is the fastest way to answer SG/NACL questions under time pressure.

**AWS re:Post — Security Group Best Practices**
https://repost.aws/knowledge-center/security-group-vpc
Community Q&A covering the most common Security Group mistakes: `0.0.0.0/0` on database ports, not using SG referencing, and confusing Security Groups with NACLs. The most-viewed questions match the Interswitch audit findings from today's slides exactly.

---

## Nigerian Regulatory Context

**CBN Risk-Based Cybersecurity Framework — Access Control Requirements**
https://www.cbn.gov.ng/out/2022/bspd/exposure_draft_-_revised_risk-based_cybersecurity_framework_for_deposit_money_banks_and_payment_service_banks.pdf
Section 3.4 covers access control requirements for networks and systems. The requirement for least-privilege access to financial data systems — "access restricted to only authorised systems and users" — maps directly to the Security Group referencing pattern: `SG-Database` allows only `sg-appservers`, not the entire VPC CIDR.

**PCI-DSS v4.0 — Requirement 1: Network Security Controls**
https://www.pcisecuritystandards.org/document_library/
Requirement 1.3 mandates restricting inbound and outbound network traffic to only what is necessary. Requirement 1.3.2 specifically addresses prohibiting direct public access to components in the cardholder data environment. The combination of private subnets (route table isolation) + Security Group referencing (application-layer access control) + NACLs (subnet-level protocol blocking) is the AWS implementation of PCI-DSS Requirement 1 for Nigerian fintechs processing card payments.

**NDPC — Nigeria Data Protection Act, 2023 — Security Standards**
https://ndpc.gov.ng/
Section on technical and organisational measures for data protection. Access controls on database systems holding personal data — a Security Group allowing only the application server Security Group as the database source, with no `0.0.0.0/0` access — is a demonstrable technical measure for NDPA compliance.
