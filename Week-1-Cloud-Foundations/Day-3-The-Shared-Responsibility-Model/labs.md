# AWS Shared Responsibility Model Labs — Day 3

---

## Lab 1: Read & Define in Your Own Words

**Objective:** Read the official AWS Shared Responsibility Model page and write definitions without copying.

**Source:** https://aws.amazon.com/compliance/shared-responsibility-model/

### My Definitions (In My Own Words)

**Security OF the Cloud:**
AWS is responsible for protecting everything that makes up the cloud infrastructure itself — the physical buildings, the servers inside them, the networking equipment connecting those servers, and the virtualisation software that lets multiple customers share the same hardware safely. If something goes wrong at the physical or infrastructure level, that's AWS's problem to fix.

**Security IN the Cloud:**
The customer is responsible for everything they build and deploy on top of that infrastructure. This includes the operating system running on their virtual machines, the applications they write and deploy, the rules governing who can access their data, whether their data is encrypted, and whether their storage buckets are properly locked down. If a customer misconfigures their firewall or leaves a database open to the internet, AWS will not intervene.

### What I Observed
The official AWS page uses a clear visual diagram showing a line dividing AWS and customer responsibilities. The line shifts left or right depending on the service — IaaS leaves more to the customer, SaaS leaves almost nothing to the customer.

### What I Learned
AWS has intentionally designed the model to give customers complete control over their data and access — meaning a customer can never say "AWS gave someone access to my data." All access control is in the customer's hands. This is a feature, not a limitation.

---

## Lab 2: Categorise 12 Real-World Items

**Objective:** Correctly assign each item to AWS, Customer, or Shared responsibility.

### Results Table

| # | Item | Responsibility | Reasoning |
|---|---|---|---|
| 1 | Physical server hardware | **AWS** | AWS owns and maintains all physical hardware in its data centres |
| 2 | EC2 operating system patches | **Customer** | You chose to run an OS on EC2 — patching it is your job |
| 3 | S3 bucket access policy | **Customer** | Bucket policies are written and applied by the customer |
| 4 | Data centre cooling & power | **AWS** | Physical environmental controls are entirely AWS's domain |
| 5 | IAM user passwords & MFA | **Customer** | IAM is always customer-managed — AWS never controls who has access to your account |
| 6 | Network firewall between AZs | **AWS** | AWS manages the physical network infrastructure including inter-AZ routing |
| 7 | Encrypting data stored in S3 | **Customer** | AWS provides encryption tools (SSE-S3, SSE-KMS) — enabling them is the customer's choice |
| 8 | Who can access your AWS account | **Customer** | Root account, IAM users, MFA — all customer-controlled |
| 9 | SSL certificate on your web app | **Customer** | Application-level TLS/HTTPS is the customer's configuration |
| 10 | Hypervisor (virtualisation layer) | **AWS** | AWS manages the hypervisor — customers never interact with it directly |
| 11 | Database backups on RDS (managed) | **AWS** | RDS automated backups are managed by AWS as part of the managed service |
| 12 | Database backups — MySQL on EC2 | **Customer** | You installed MySQL yourself on EC2 — you own all backup processes |

### Shared Controls

| Control | AWS Layer | Customer Layer |
|---|---|---|
| Patch Management | Patches host OS and hypervisor | Patches guest OS and application dependencies |
| Configuration Management | Configures AWS infrastructure devices | Configures their OS, databases, and applications |
| Security Awareness Training | Trains AWS employees | Trains their own developers and staff |

### What I Observed
Items 11 and 12 were the most instructive. The task is identical (database backups) but the responsibility flips based entirely on the service type. RDS is managed — AWS handles the engine. EC2-hosted MySQL is self-managed — you handle everything.

### What I Learned
The critical exam principle: **service type determines responsibility boundary, not task type.** The same action (backups, patching, monitoring) can be AWS's responsibility in one service and the customer's in another. Always ask "how managed is this service?" before assigning responsibility.

---

## Lab 3: Build the Responsibility Diagram

**Objective:** Create a visual 3-column table categorising all items under AWS Responsibility | Shared Controls | Customer Responsibility.

### Completed Diagram

```
┌─────────────────────────────┬──────────────────────────┬─────────────────────────────┐
│      AWS RESPONSIBILITY     │     SHARED CONTROLS      │   CUSTOMER RESPONSIBILITY   │
│    (Security OF the Cloud)  │   (Both Parties Own a    │   (Security IN the Cloud)   │
│                             │     Layer of This)       │                             │
├─────────────────────────────┼──────────────────────────┼─────────────────────────────┤
│ Physical server hardware    │ Patch Management         │ EC2 OS patches              │
│ Data centre cooling & power │ Configuration Management │ S3 bucket access policy     │
│ Network firewall between AZs│ Security Awareness       │ IAM user passwords & MFA    │
│ Hypervisor / virt. layer    │ Training                 │ Encrypting data in S3       │
│ RDS automated backups       │                          │ Who can access your account │
│ Physical facility security  │                          │ SSL cert on your web app    │
│ Global network backbone     │                          │ MySQL on EC2 backups        │
└─────────────────────────────┴──────────────────────────┴─────────────────────────────┘
```

**Title:** AWS Shared Responsibility Model — My Summary
**Date:** Week 1, Day 3

### What I Observed
Laying it out visually makes the pattern obvious: AWS's column is all physical and infrastructure-level. The customer's column is all logical, digital, and configuration-level. The shared column is all process-level.

### What I Learned
The visual confirms a useful rule of thumb: if you can touch it physically → AWS. If you configure it in the console → Customer. If it's an ongoing process involving both layers → Shared.

---

## Lab 4: 3-Sentence Plain-English Explanation

**Target audience:** A Lagos restaurant owner considering moving their POS system and customer data to the cloud.

**My explanation:**

"When you move your business to AWS, Amazon takes full responsibility for the physical security of the computers your data is stored on — they lock the building, monitor the facility 24/7, and replace any failing hardware without you ever knowing. However, you are still fully responsible for who in your business can access that data, making sure your records are encrypted, and setting the rules for what is private versus what is visible to others. This split is actually better than keeping servers on-site — because Amazon's physical security exceeds anything a restaurant could build, while you retain complete digital control over your own customer data."

### What I Observed
Writing for a non-technical audience required removing every piece of AWS jargon. "IAM" became "who in your business can access that data." "Encryption at rest" became "making sure your records are encrypted." The meaning survives; the confusion is removed.

### What I Learned
Explaining a technical concept in plain language forces you to understand it more deeply than any amount of reading. If you can't explain the Shared Responsibility Model to a restaurant owner in 3 sentences, you don't fully understand it yet.

---

## Bonus Challenge: Capital One Breach Research

See the full 5-bullet analysis in `/notes/real-world-scenarios.md`.

**Short summary:**
The 2019 Capital One breach exposed 100 million records. AWS's infrastructure was never compromised. A misconfigured Web Application Firewall (customer responsibility) allowed an attacker to obtain IAM credentials via SSRF. Overly permissive IAM roles (customer responsibility) allowed those credentials to access S3 buckets. Every point of failure was on the customer's side of the responsibility model.
