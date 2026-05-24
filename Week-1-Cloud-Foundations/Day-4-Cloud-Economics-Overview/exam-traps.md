# Exam Traps & Practice Questions — Day 4

---

## The 3 Critical Exam Traps

### Trap 1 — Free Tier Has THREE Types, Not One

**The trap:** Exam questions will describe a service usage scenario and ask whether it falls under the Free Tier. Getting this wrong means not knowing *which type* applies.

**The three types:**

| Type | What It Means | Key Examples |
|---|---|---|
| 12-Month Free | Free for 12 months from account creation — then billing starts | EC2 t2.micro (750 hrs/mo), S3 (5 GB), RDS db.t3.micro |
| Always Free | Never expires — free forever regardless of account age | Lambda (1M requests/mo), DynamoDB (25 GB), CloudFront (1 TB/mo), SNS (1M publishes) |
| Free Trials | Short per-service trial period only | Redshift (2 months), SageMaker (2 months), Amazon Lightsail (3 months) |

**How it appears on the exam:** "A company has had an AWS account for 14 months. Which service would still be available under the Free Tier?" → The answer must be an Always Free service (e.g. Lambda) — 12-month services expired at month 13.

**Memorise this pair:** EC2 = 12-month. Lambda = Always Free.

---

### Trap 2 — Spot Instances Can Be Terminated With 2-Minute Notice

**The trap:** Spot Instances offer the largest discount (90%) — exam questions use this to tempt you into recommending Spot for workloads that require reliability.

**The rule:** If the workload cannot tolerate interruption, Spot Instances is **always the wrong answer**. Full stop.

**Scenarios where Spot is WRONG:**
- Primary production database (RDS or MySQL on EC2)
- Payment processing systems (Paystack, Flutterwave)
- Any stateful application that cannot checkpoint state
- Web servers behind a load balancer handling active user sessions (debatable — depends on statefulness)
- Any scenario where the exam describes "critical," "must be available," or "cannot be interrupted"

**Scenarios where Spot is CORRECT:**
- Batch processing / data transformation jobs
- Machine learning model training (can checkpoint)
- Video rendering or transcoding
- Log analysis pipelines
- Anything described as "fault-tolerant," "stateless," or "can be retried"

---

### Trap 3 — Data Transfer Costs Are Not Zero

**The trap:** Many students assume all data movement within AWS is free. It is not.

**The data transfer rules:**
- Inbound data to AWS from the internet → **Free**
- Outbound from AWS to the internet → **Charged** (tiered pricing)
- Same AZ, same service → **Free**
- Between AZs in the same Region → **Small charge** (~$0.01/GB each direction)
- Between Regions → **Larger charge** (varies by Region pair, ~$0.02–$0.09/GB)
- To/from CloudFront → **Special pricing** (usually cheaper than direct outbound)

**How it appears on the exam:** A cost optimisation question where the architect can reduce cross-AZ data transfer by co-locating services in the same AZ. The trade-off: same-AZ placement reduces transfer costs but eliminates AZ fault tolerance. The correct answer will acknowledge this trade-off.

---

## SAA-C03 Practice Question — Day 4

**Question:**
A Solutions Architect at a Lagos fintech company needs to run two types of workloads: (1) a core payment processing API that handles real-time transactions 24/7, and (2) a nightly batch job that processes transaction reports and can be restarted if it fails. The architect wants to minimise costs. Which combination of pricing models is MOST cost-effective?

**A.** On-Demand for both workloads — predictable pricing with no commitment risk.

**B.** Spot Instances for both workloads — maximises the 90% discount across the entire architecture.

**C.** Reserved Instances for the payment processing API, and Spot Instances for the nightly batch job.

**D.** Savings Plans for the payment processing API, and Reserved Instances for the nightly batch job.

---

**Answer: C**

**Why C is correct:**
The payment API runs 24/7 and cannot tolerate interruption — Reserved Instances are ideal: steady-state workload, predictable, and the 1-year commitment makes financial sense. The nightly batch job is explicitly described as restartable if it fails — this is the textbook definition of a Spot-eligible workload, and the 90% discount dramatically reduces the batch processing cost.

**Why A is wrong:**
On-Demand for a 24/7 steady-state payment API is expensive and misses a clear opportunity for Reserved Instance savings. On-Demand is appropriate for unpredictable workloads, not always-on production systems.

**Why B is wrong:**
Spot Instances for a real-time payment API is catastrophically wrong. A 2-minute termination notice mid-transaction would cause payment failures, data corruption, and customer-facing outages. The 90% discount is irrelevant when the business risk is that high.

**Why D is wrong:**
Savings Plans for the API is reasonable, but Reserved Instances for the batch job is wasteful — you're committing 1–3 years to capacity for a workload that only runs a few hours per night. Spot Instances are far more cost-effective for interruptible nightly jobs.

---

## Quick Recall Quiz

Cover the answers and test yourself:

| Question | Answer |
|---|---|
| What is the max discount for Spot Instances? | 90% |
| What is the max discount for Reserved Instances? | 72% |
| What is the max discount for Savings Plans? | 66% |
| How much notice does AWS give before terminating a Spot Instance? | 2 minutes |
| How many types of Free Tier are there? | 3 (12-month, always-free, trial) |
| Which Free Tier type covers Lambda? | Always Free |
| Which Free Tier type covers EC2 t2.micro? | 12-Month Free |
| How many EC2 hours/month in the Free Tier? | 750 hours |
| Is data transfer between AZs in the same Region free? | No — small charge per GB |
| Is inbound data to AWS free? | Yes |
| What does Savings Plans commit to vs Reserved Instances? | Spend level (not instance type) |
| Which pricing model applies across EC2, Lambda, AND Fargate? | Savings Plans |
| Which tool sets a spend alert BEFORE you exceed your budget? | AWS Budgets |
| Which tool shows HISTORICAL spend with 13-month forecasting? | AWS Cost Explorer |
| What's the first cost control step on any new AWS account? | Create a Zero Spend Budget |
