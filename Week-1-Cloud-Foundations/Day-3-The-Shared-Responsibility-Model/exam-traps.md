# Exam Traps & Practice Questions — Day 3

---

## The 3 Critical Exam Traps

### Trap 1 — AWS is ALWAYS responsible for physical infrastructure; customers are ALWAYS responsible for data and IAM

**The trap:** Some questions try to blur the line at the hardware/infrastructure level — suggesting the customer might share physical security responsibility, or that AWS manages IAM.

**The rule:**
- AWS is **always** responsible for: physical hardware, facilities, global network, virtualisation/hypervisor
- Customer is **always** responsible for: data classification, IAM access controls, who has access to the AWS account

There is no scenario where a customer shares responsibility for the physical data centre. There is no scenario where AWS manages your IAM users.

---

### Trap 2 — RDS vs. MySQL on EC2 (same task, different responsibility owner)

This is the most frequently tested nuance on the SAA-C03 and is almost guaranteed to appear.

**The scenario:**
- Both setups involve a MySQL database
- The question asks who is responsible for database backups, OS patches, or database engine updates

| Task | RDS (Managed) | MySQL on EC2 (Self-managed) |
|---|---|---|
| DB engine patches | **AWS** | **Customer** |
| OS patches | **AWS** | **Customer** |
| Automated DB backups | **AWS** | **Customer** |
| DB user permissions | Customer | Customer |
| Data stored | Customer | Customer |
| Network access rules | Customer | Customer |
| Encryption config | Customer | Customer |

**The key mental model:**
- RDS = AWS took over the engine and OS. You manage the data and access.
- MySQL on EC2 = You installed the DB yourself. You manage everything from the OS upward.

> ⚠️ **Exam trigger phrase:** "A company is running a MySQL database on an EC2 instance..." → Customer manages OS patches, backups, everything. "A company is using Amazon RDS for MySQL..." → AWS manages the engine and OS. Customer manages data and access.

---

### Trap 3 — The Grey Area: Shared Controls

**The trap:** A question presents patch management, configuration management, or security awareness training and asks who is responsible.

**The answer:** These are **Shared Controls** — both AWS and the customer have responsibility at different layers.

When in doubt on an exam question that mentions these three topics, the correct answer acknowledges that both parties have a role — not that one party owns it entirely.

---

## SAA-C03 Practice Question — Day 3

**Question:**
A Solutions Architect at a Lagos fintech company is designing the AWS infrastructure for a new payment platform. The company's CISO asks: "If we store customer payment data on AWS, does that mean AWS is responsible for protecting it?" Which response correctly describes the Shared Responsibility Model?

**A.** AWS is fully responsible for all data stored on its infrastructure, including encryption and access controls, because it controls the underlying hardware.

**B.** The customer is solely responsible for all aspects of security once they deploy on AWS, including the physical facilities and hardware.

**C.** AWS secures the physical infrastructure and virtualisation layer. The customer is responsible for encrypting the payment data, configuring IAM access controls, and ensuring S3 bucket policies are correctly set.

**D.** Responsibility for data protection is shared equally — AWS and the customer each handle exactly 50% of all security tasks.

---

**Answer: C**

**Why C is correct:**
This is a precise, accurate description of the model. AWS handles physical and hypervisor security (Security OF the Cloud). The customer handles data encryption, IAM, and access policies (Security IN the Cloud). No ambiguity, no overclaiming.

**Why A is wrong:**
AWS does NOT take responsibility for data encryption or access controls. These are explicitly customer responsibilities. This is exactly the misconception that leads to breaches like Capital One.

**Why B is wrong:**
The customer is never responsible for physical facilities and hardware. Those are always AWS's domain. The customer responsibility starts at the operating system and application layer.

**Why D is wrong:**
The split is not 50/50 and varies by service. The model defines specific ownership boundaries — it is not an equal division of all tasks.

---

## Quick Recall Quiz

Cover the answers and test yourself:

| Question | Answer |
|---|---|
| Who secures the physical data centre? | AWS |
| Who is responsible for S3 bucket policies? | Customer |
| Who patches EC2 operating systems? | Customer |
| Who patches RDS database engine? | AWS |
| Who manages IAM user passwords and MFA? | Customer |
| Who secures the hypervisor? | AWS |
| Who manages data classification? | Customer |
| Who is responsible for MySQL backups on EC2? | Customer |
| Who is responsible for RDS automated backups? | AWS |
| What are the three Shared Controls? | Patch management, configuration management, security awareness training |
| What does a customer inherit from AWS? | Physical and environmental controls (no data centre audit needed) |
| Which service has MORE customer responsibility — EC2 or Lambda? | EC2 |

---

## The 12-Item Categorisation (Lab Answer Guide)

| # | Item | Responsibility |
|---|---|---|
| 1 | Physical server hardware | **AWS** |
| 2 | EC2 operating system patches | **Customer** |
| 3 | S3 bucket access policy | **Customer** |
| 4 | Data centre cooling & power | **AWS** |
| 5 | IAM user passwords & MFA | **Customer** |
| 6 | Network firewall between AZs | **AWS** |
| 7 | Encrypting data stored in S3 | **Customer** |
| 8 | Who can access your AWS account | **Customer** |
| 9 | SSL certificate on your web app | **Customer** |
| 10 | Hypervisor (virtualisation layer) | **AWS** |
| 11 | Database backups on RDS (managed) | **AWS** |
| 12 | Database backups — MySQL on EC2 | **Customer** |
| — | Patch management | **Shared** |
| — | Configuration management | **Shared** |
| — | Security awareness training | **Shared** |

> Items 11 and 12 are the most important pair. Same task (DB backups), different service type, different responsibility owner. This exact pairing appears on the SAA-C03 exam.
