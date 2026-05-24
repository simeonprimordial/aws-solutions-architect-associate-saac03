# Cloud Economics — Day 4 Notes

---

## Overview — The Fundamental Shift

### Traditional IT: Capital Expenditure (CapEx)
Before cloud, every company had to buy servers upfront. The problem: you had to buy for **peak demand**, even if that peak lasted 3 days a year. The rest of the time, expensive hardware sat idle, depreciating.

**Example:** A Nigerian e-commerce platform buying servers to handle Christmas traffic:
- Purchases $50,000 of servers in October
- Those servers run at 10% capacity from January to November
- For 3 weeks in December, they hit peak load
- In January, you still own and pay for $50,000 of hardware that's mostly idle again

**Additional CapEx burdens:**
- Data centre space / colocation fees
- Power and cooling costs
- Hardware maintenance contracts
- Dedicated sysadmin salary
- Hardware refresh every 3–5 years

---

### Cloud: Operating Expenditure (OpEx)
Cloud converts upfront capital costs into predictable operational expenses. You pay for what you consume, when you consume it.

**Benefits of OpEx model:**
- No idle hardware — stop an EC2 instance and billing stops immediately
- Scale up during peak, scale down after — pay only for the peak duration
- Cash flow improvement — no large capital outlay before revenue
- No hardware refresh cycles — AWS handles all infrastructure upgrades
- Faster time to market — launch in hours, not the weeks needed to procure and configure physical servers

**The African business impact:**
A Lagos startup can launch production-ready infrastructure on AWS with the equivalent of a few thousand naira in credits. Paystack, Flutterwave, and Piggyvest all built on cloud economics — scaling spend only as customers grew.

---

## The Five AWS Pricing Models

### 1. On-Demand
**What it is:** Pay per second (minimum 60 seconds) or per hour of actual usage. No upfront commitment. No termination fee.

**How billing works:**
- EC2: billed per second (Linux/Unix) or per hour (Windows)
- S3: billed per GB stored per month + per-request fees
- Lambda: billed per invocation + per millisecond of execution time
- Stopping an EC2 instance stops compute billing — but EBS storage billing continues

**When to use:** Dev/test environments, unpredictable or spiky workloads, short-term projects, new applications where usage patterns are unknown.

**Exam note:** On-Demand is always the baseline. It's the most expensive per-hour rate. All other models are measured as discounts from On-Demand.

---

### 2. Reserved Instances (RIs)
**What it is:** A 1-year or 3-year commitment to a specific EC2 instance type in a specific Region, in exchange for up to 72% discount vs On-Demand pricing.

**Three payment options:**
- **All Upfront** — Pay full amount at start. Maximum discount.
- **Partial Upfront** — Pay some upfront, rest monthly. Mid-tier discount.
- **No Upfront** — Pay monthly only. Lower discount but no initial cash outlay.

**Convertible vs Standard RIs:**
- **Standard RI** — Cannot change instance family or Region. Higher discount (up to 72%).
- **Convertible RI** — Can change instance type, OS, or tenancy during the term. Lower discount (up to 66%).

**When to use:** Steady-state, predictable workloads — production databases, always-on application servers, workloads running 24/7 for a year or more.

**When NOT to use:** Dev/test, workloads that might change instance type, anything uncertain.

---

### 3. Spot Instances
**What it is:** Purchase unused AWS EC2 capacity at up to 90% discount. AWS can reclaim Spot Instances with only **2 minutes notice** when it needs the capacity back.

**The critical constraint:** If AWS reclaims your Spot Instance, it terminates with 2 minutes warning. Your application must be able to tolerate interruption — save state frequently, checkpoint progress.

**When to use:**
- Batch processing jobs (render farms, data analysis, machine learning training)
- Fault-tolerant distributed computing
- Background processing that can be retried
- Test and development workloads on a budget

**NEVER use for:**
- Primary production databases
- Stateful applications that cannot tolerate interruption
- Payment processing systems
- Any workload where losing 2 minutes of work is unacceptable

> ⚠️ **Critical exam rule:** Spot Instances can be interrupted with 2-minute notice. If the exam scenario involves a database, payment system, or stateful application — Spot Instances is always the wrong answer.

---

### 4. Savings Plans
**What it is:** A flexible commitment model where you agree to spend a minimum dollar amount per hour (e.g. $10/hr) for 1 or 3 years. In return, you receive discounts of up to 66% on covered services.

**Key difference from Reserved Instances:**
- Reserved Instances lock you to a specific instance type and Region
- Savings Plans commit to a **spend level** — the discount applies across EC2 instance types, Lambda, and Fargate automatically

**Two types:**
- **Compute Savings Plans** — Most flexible. Applies to EC2 (any family, size, Region, OS), Lambda, and Fargate. Up to 66% discount.
- **EC2 Instance Savings Plans** — Locked to instance family in a Region but offers slightly higher discount (up to 72%).

**When to use:** When you have predictable overall compute spend but want flexibility to change instance types as your architecture evolves.

---

### 5. AWS Free Tier
**What it is:** AWS provides free usage of certain services to allow learning, testing, and new service exploration.

**Three distinct types — all tested on SAA-C03:**

| Type | Duration | Example Services |
|---|---|---|
| **12-Month Free** | 12 months from account creation | EC2 t2.micro (750 hrs/mo), S3 (5 GB), RDS db.t3.micro |
| **Always Free** | Never expires | Lambda (1M requests/mo), DynamoDB (25 GB), CloudFront (1 TB/mo) |
| **Free Trials** | Short-term per-service trials | Redshift (2 months), SageMaker (2 months) |

> ⚠️ **Exam trap:** The exam will ask which type of Free Tier applies to a specific service. Memorise: EC2 = 12-month. Lambda = Always Free. Trial services = short-term only.

---

## How AWS Bills You — The Mechanics

### Billing Metrics by Service
| Service | Billing Unit | Key Details |
|---|---|---|
| EC2 | Per second (min 60s) | Stops when instance stopped; EBS continues |
| S3 | Per GB/month + per request | Inbound transfer free; outbound charged |
| Lambda | Per invocation + per ms | First 1M requests always free |
| RDS | Per hour + storage + I/O | Multi-AZ doubles compute cost |
| Data Transfer | Per GB out | Within-Region AZ transfer charged (small); cross-Region charged (larger) |

### The Data Transfer Trap
- **Inbound data to AWS:** Free
- **Outbound from AWS to internet:** Charged (tiered, lower rate at higher volume)
- **Between AZs in the same Region:** Small charge per GB
- **Between Regions:** Larger charge per GB
- **Within the same AZ:** Free (same service to same service)

> ⚠️ **Exam trap:** Data transfer between AZs within the same Region is NOT free. It is a small but real cost that must be accounted for in architecture estimates. Cross-Region transfer is significantly more expensive.

---

## AWS Cost Control Tools

### AWS Budgets
Set spend alerts **before** you exceed your budget — not after.
- Zero Spend Budget: alerts at $0.01 — the moment any charge appears
- Cost budget: alert at a dollar threshold (e.g. $50/month)
- Usage budget: alert based on service usage (e.g. EC2 hours)
- Set this up on **Day 1** of any new account

### AWS Cost Explorer
- Visualise historical AWS spend across services, accounts, and tags
- 13-month look-back window + forecasting
- Group by: Service, Region, Usage Type, Linked Account, Tag
- Free to use

### AWS Pricing Calculator
- Estimate monthly costs **before** you build
- Build multi-service estimates and share via public link
- URL: https://calculator.aws

### AWS Trusted Advisor
- Analyses your account for cost optimisation opportunities
- Flags idle EC2 instances, underutilised Reserved Instances, low S3 usage
- Basic checks free; full access requires Business or Enterprise Support plan
