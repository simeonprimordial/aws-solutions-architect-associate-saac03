# The Shared Responsibility Model — Day 3 Notes

---

## Overview — Why This Matters

The Shared Responsibility Model is one of the **most heavily tested topics on the SAA-C03 exam** because customer misconfiguration — not AWS failure — is the #1 cause of cloud security breaches.

**The 2019 Capital One breach** is the defining real-world example:
- 100 million customer records were exposed
- AWS's physical infrastructure performed perfectly — the data centres, hardware, and network were fully secure
- The breach happened because Capital One misconfigured their Web Application Firewall (WAF), allowing an attacker to exploit a Server-Side Request Forgery (SSRF) vulnerability and access sensitive data stored in S3
- AWS built a secure house. Capital One accidentally left the digital front door open.

**The key mindset shift for Nigerian organisations:**
Many businesses fear moving to the cloud because they think they're losing control of their data. The reality is the opposite — AWS locks the physical doors, but **the customer holds the only digital keys** to their data. The model gives you *more* documented control, not less.

---

## The Two Sides of the Model

### Security OF the Cloud — AWS's Responsibility

AWS is responsible for protecting the infrastructure that runs all AWS services. This includes:

- **Physical facilities** — Data centre buildings, perimeter security, CCTV, access control, biometrics
- **Server hardware** — Physical servers, networking equipment, storage hardware
- **Virtualisation layer** — The hypervisor that isolates one customer's EC2 instance from another's
- **Global network** — AWS's private backbone network connecting Regions, AZs, and Edge Locations
- **Environmental controls** — Power, cooling, fire suppression in data centres

> If a physical disk fails in an AWS data centre, AWS replaces it immediately. You never touch or worry about hardware.

---

### Security IN the Cloud — Customer's Responsibility

The customer is responsible for everything they deploy on top of AWS's infrastructure. This includes:

- **Data** — Classification, encryption at rest and in transit, backup
- **Identity and Access Management** — IAM users, roles, groups, MFA, password policies
- **Operating system** — Patching, hardening, securing the OS on EC2 instances
- **Application security** — Code vulnerabilities, dependency patches, secure coding practices
- **Network controls** — Security Groups, NACLs, VPC configurations, firewall rules
- **Bucket policies** — Making sure S3 buckets are not accidentally public
- **Encryption configuration** — Enabling encryption on S3, RDS, EBS — AWS provides the tools, but you must use them

> AWS provides the tools (KMS, Security Groups, IAM). Using them correctly is your job.

---

## The Managed Services Shift

The responsibility line is **not fixed** — it moves depending on how managed the service is.

```
More Customer Responsibility ←————————————→ More AWS Responsibility

  EC2           RDS          Elastic         Lambda
(IaaS)      (Managed DB)   Beanstalk     (Serverless)
  │               │           │               │
  You manage    AWS manages  AWS manages    AWS manages
  OS upward     DB engine    runtime &      everything
                & OS         scaling        except code
```

### EC2 (IaaS) — Maximum customer responsibility
You manage: OS installation, OS patches, runtime, middleware, application, data, firewall rules
AWS manages: Physical hardware, hypervisor, host OS

### RDS (Managed Database) — Shifted responsibility
You manage: Database user permissions, data stored, network access rules, encryption config
AWS manages: DB engine patches, underlying OS, hardware, automated backups of the engine

### Lambda (Serverless) — Minimum customer responsibility
You manage: Function code, data passed through the function, IAM execution roles
AWS manages: Runtime environment, OS, hardware, auto-scaling, availability

---

## Shared Controls

Some responsibilities are **shared** — both AWS and the customer handle their own layer:

| Control | AWS's Layer | Customer's Layer |
|---|---|---|
| Patch Management | Patches the host OS and hypervisor | Patches the guest OS and applications |
| Configuration Management | Configures AWS infrastructure devices | Configures their own OS, applications, databases |
| Security Awareness Training | Trains AWS employees | Trains their own employees and developers |

> ⚠️ **Exam Trap:** When a question mentions patch management, configuration management, or security training — these are Shared Controls. Both parties own a layer of this.

---

## Inherited Controls

Customers automatically **inherit** AWS's physical and environmental security controls. This is one of the core value propositions of cloud vs. on-premises:

- You do not need to audit AWS's data centres yourself
- You inherit the compliance certifications AWS has earned: ISO 27001, SOC 2, PCI DSS, HIPAA eligibility
- AWS publishes compliance reports via **AWS Artifact** — customers can download these for their own audits

**Practical implication for Nigerian businesses:** A Lagos startup automatically inherits the same physical security standards that took AWS years and billions to build. This is something no Nigerian SME could achieve on-premises at any budget.

---

## The Model as a Mental Contract

Think of it this way:

```
AWS signs a contract that says:
  "We guarantee the physical building is secure.
   We guarantee the hardware is secure.
   We guarantee the virtualisation layer is secure.
   We guarantee our global network is secure."

You sign a contract that says:
  "We will manage our own data appropriately.
   We will configure our own access controls.
   We will patch our own operating systems.
   We will set our own encryption policies."

Neither party can blame the other for failing their own signed obligations.
```
