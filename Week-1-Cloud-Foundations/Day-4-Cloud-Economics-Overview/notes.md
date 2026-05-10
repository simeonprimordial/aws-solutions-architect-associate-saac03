# Week 1 · Day 4 — Cloud Economics Overview

---

## 1. Why This Matters

| Traditional IT (CapEx) | Cloud (OpEx) |
|---|---|
| Buy servers upfront for peak demand | Pay only for what you use |
| Hardware sits idle most of the time | No idle hardware costs |
| Massive capital locked in infrastructure | Cash flow improves dramatically |

**The African Business Advantage:** A Lagos startup can launch production-ready on AWS with ₦50,000 in credits and scale spend only as customers grow.

> 💡 FinTechs like Paystack, Flutterwave, and PiggyVest launched fast by converting CapEx to OpEx — no upfront servers, no waiting months to go live.

---

## 2. Key Concepts

### Pay-As-You-Go
- No upfront cost, no termination fees
- Billed per second, hour, or request of actual usage
- Stop an EC2 instance → billing stops immediately

### Reserved Instances
- 1 or 3-year commitment to a specific instance type
- Up to **72% discount** vs On-Demand pricing
- Best for steady-state, predictable workloads
- ⚠️ Less flexible — instance type is locked in

### Spot Instances
- Use spare AWS capacity at up to **90% discount**
- Can be **interrupted with only 2-minute notice**
- Best for fault-tolerant, interruptible jobs (batch processing, data pipelines)
- ⚠️ NEVER use for primary databases or stateful applications

### Savings Plans
- Commit to a **spend level** (not a specific instance type)
- Up to **66% discount** across EC2, Lambda, and Fargate
- More flexible than Reserved Instances
- 1 or 3-year commitment

### AWS Free Tier
Three distinct types — knowing which is which is an exam question:

| Type | Example | Limit |
|---|---|---|
| 12-month free | EC2 t2.micro / t3.micro | 750 hours/month |
| Always free | Lambda | 1M requests + 400K GB-seconds/month |
| Always free | S3 | 5 GB + 20K GET + 2K PUT/month |
| Short-term trial | Amazon Redshift | 2-month trial |

---

## 3. AWS Pricing Models — Side-by-Side

| Model | Commitment | Max Discount | Best For | Exam Watch |
|---|---|---|---|---|
| On-Demand | None | 0% | Dev/test, unpredictable workloads | Baseline pricing |
| Savings Plans | 1–3 yr (spend level) | 66% | EC2 + Lambda + Fargate | Commit to $ spend, not instance type |
| Reserved Instances | 1–3 yr (instance type) | 72% | Steady, predictable workloads | Instance type locked — less flexible |
| Spot Instances | None | 90% | Fault-tolerant, interruptible jobs | 2-min termination notice — NEVER for DBs |
| AWS Free Tier | N/A | 100%* | Learning & new service trials | 3 types: 12-month, always-free, trial |

**Spectrum:** On-Demand → Savings Plans → Reserved Instances → Spot Instances  
*(More flexible / lower savings → Higher savings / more commitment)*

---

## 4. How AWS Bills You

- **EC2:** Per second (minimum 60 seconds), based on instance type and Region
- **S3:** Per GB stored per month + per-request fees
- **Lambda:** Per invocation + per millisecond of execution time

---

## 5. Cost Management Tools

| Tool | Purpose |
|---|---|
| **AWS Cost Explorer** | Historical spend analysis with 13-month forecasting |
| **AWS Budgets** | Set alerts for actual or forecasted spend before going over |
| **AWS Pricing Calculator** | Estimate costs before you build (calculator.aws) |

> 💡 **Best practice:** Set a $0 budget alert in AWS Budgets on Day 1 of your account.

---

## 6. Real-World Nigerian Scenario — Paystack on AWS

| | Physical Servers | AWS |
|---|---|---|
| **Year 1 Cost** | ~$73,000+ | ~$300/month to start |
| **Scaling** | Fixed capacity, idle 90% of the time | Auto-scales for Black Friday surges |
| **Ops burden** | Dedicated sysadmin + data centre fees | AWS manages infra; team focuses on product |
| **Payment at 3am** | Same fixed cost | Near-zero cost (pay per transaction) |

**Savings in Year 1 alone: ~$69,400**

---

## 7. Common Mistakes & Exam Traps

❌ **Free Tier has THREE types** — knowing which type applies to each service is tested directly.

❌ **Spot Instances = 2-minute warning** — never use for stateful apps, primary databases, or jobs that cannot tolerate interruption.

❌ **Data transfer costs money** — within the same Region between AZs incurs small charges; between Regions incurs larger charges. Always account for this in cost estimates.

---

## 8. Key Takeaways

- ✅ Cloud converts CapEx (upfront) to OpEx (pay-as-you-go)
- ✅ Four main pricing models: On-Demand, Reserved, Spot, Savings Plans
- ✅ AWS Free Tier has three distinct types — memorise which is which
- ✅ Spot = cheapest but interruptible; Reserved = best for predictable load
- ✅ Set up AWS Budgets and Cost Explorer on Day 1

---