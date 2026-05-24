# Cloud Economics Labs — Day 4

---

## Lab 1: Activate Your Cost Safety Net — AWS Budgets

**Objective:** Set up a Zero Spend Budget that alerts at the first dollar of charge.

### Steps
1. In the AWS Console search bar, type **Budgets** → click **AWS Budgets**
2. Click **Create budget** → **Use a template** → **Zero spend budget**
3. Enter email address for notifications
4. Click **Create budget**

### Budget Configuration Documented

| Field | Value |
|---|---|
| Budget name | `zero-spend-alert` |
| Alert threshold | $0.01 (triggers at first charge) |
| Notification email | [your email] |
| Budget type | Cost budget |

### What I Observed
The Zero Spend Budget template requires zero configuration beyond the email address. It pre-sets the threshold at $0.01 — meaning any charge, however small, triggers an alert immediately.

### What I Learned
- AWS Budgets alerts are proactive — they fire before or as soon as charges appear, not after the billing cycle closes. This is the critical difference between Budgets (real-time alerting) and Cost Explorer (historical analysis).
- If a cost alert arrives during this course, it means a resource was left running. The correct response: stop the resource immediately, then check the billing console to identify what was charged and for how long.
- Budget alerts can take up to 24 hours to activate after creation — they do not alert retroactively.

---

## Lab 2: Explore the Free Tier Dashboard

**Objective:** Document the Free Tier limits for EC2, S3, and Lambda.

### Steps
1. In the search bar type **Free Tier** → open AWS Free Tier page
2. Locate EC2, S3, and Lambda in the service list
3. Document the Free Tier type and monthly limits

### Free Tier Reference Table

| Service | Free Tier Type | Monthly Limit | Expires? |
|---|---|---|---|
| EC2 (t2.micro / t3.micro) | 12-Month Free | 750 hours/month | Yes — Month 13 |
| S3 | 12-Month Free | 5 GB storage · 20K GET · 2K PUT | Yes — Month 13 |
| Lambda | Always Free | 1M requests + 400K GB-seconds compute | Never |
| DynamoDB | Always Free | 25 GB storage + 25 WCU + 25 RCU | Never |
| CloudFront | Always Free | 1 TB data transfer + 10M requests/month | Never |
| RDS (db.t3.micro) | 12-Month Free | 750 hours/month + 20 GB storage | Yes — Month 13 |

### What I Observed
The Free Tier page in the console shows both your *limit* and your *current usage* for each service with a progress bar. It's the fastest way to check if you're approaching a limit that would trigger billing.

### What I Learned
- The distinction between 12-month and Always Free is an SAA-C03 exam question. Lambda's Always Free tier means you can build serverless architectures that remain free indefinitely at low usage — critical for startups in a bootstrapped phase.
- 750 EC2 hours/month is exactly enough for one t2.micro running 24/7 (730 hours). Running two t2.micros simultaneously halves that budget to ~375 hours each — hitting the limit at about Day 16 of the month.

---

## Lab 3: Compare All Four Pricing Models

**Objective:** Find real t3.micro pricing data across all pricing models from official AWS pricing pages.

### Sources Used
- On-Demand: https://aws.amazon.com/ec2/pricing/on-demand
- Reserved: https://aws.amazon.com/ec2/pricing/reserved-instances/pricing
- Spot: https://aws.amazon.com/ec2/spot/instance-advisor

### Completed Pricing Table (t3.micro, us-east-1, Linux)

| Pricing Model | Price / hr | Monthly (730 hrs) | Commitment | Best For |
|---|---|---|---|---|
| On-Demand | $0.0104 | $7.59 | None | Unpredictable/dev workloads |
| Reserved (1-yr, No Upfront) | $0.0066 | $4.82 | 1 year | Steady 24/7 production |
| Reserved (1-yr, All Upfront) | ~$0.0059 | $4.31 | 1 year + lump sum | Maximum savings on known workload |
| Spot (t3.micro, us-east-1) | ~$0.0031 | ~$2.26 (variable) | None (interruptible) | Batch, fault-tolerant jobs |
| Free Tier (t2.micro) | $0.00 | $0.00 (≤750 hrs) | 12 months | Learning and labs |

### Key Numbers
- **On-Demand vs Reserved (1-yr, No Upfront):** 37% savings — $2.77/month saved
- **On-Demand vs Spot:** ~70% savings — but with interruption risk
- **3-year Reserved projection:** Lock in savings for 36 months = ~$100 saved vs On-Demand for one t3.micro alone

### What I Observed
The absolute dollar differences for a single t3.micro look small, but the principles scale. A company running 500 EC2 instances: that 37% Reserved discount is not $2.77/month — it's thousands of dollars per month in savings.

### What I Learned
- Reserved Instance savings compound with scale. The business case for Reserved Instances is strongest for large, predictable fleets.
- Spot pricing fluctuates based on supply and demand. The Spot Instance Advisor also shows interruption frequency — t3.micro in us-east-1 has very low interruption frequency, making it a relatively safe Spot candidate for non-critical batch work.

---

## Lab 4: Build a Startup Cost Estimate — AWS Pricing Calculator

**Objective:** Build a cost estimate for a Lagos fintech startup MVP, then compare On-Demand vs Reserved pricing.

**URL:** https://calculator.aws

### Configuration Built

| Service | Spec | Monthly Config |
|---|---|---|
| EC2 | 1 × t3.micro, On-Demand, Linux, us-east-1 | 730 hrs/month |
| S3 | Standard storage | 10 GB, 50K GET, 5K PUT/month |
| RDS | db.t3.micro, MySQL, Single-AZ, us-east-1 | 20 GB General Purpose SSD |

### Results

| Configuration | Monthly Estimate | Annual Estimate |
|---|---|---|
| EC2 (On-Demand) + S3 + RDS | ~$38.50 | ~$462 |
| EC2 (Reserved 1-yr, No Upfront) + S3 + RDS | ~$26.20 | ~$314 |
| **Annual saving with Reserved EC2** | **~$148** | |
| **3-year saving with Reserved EC2** | **~$444** | |

### Architect's Notes on the Estimate
- The RDS db.t3.micro dominates the monthly bill (approximately $26 of the ~$38.50 total on On-Demand) — the database is the biggest cost driver for this configuration
- S3 at 10 GB is negligible — ~$0.23/month for the storage above the 5 GB Free Tier limit
- At MVP stage: run On-Demand and use Free Tier. Commit nothing until traffic is proven.
- At 6 months: switch EC2 and RDS to 1-year Reserved Instances — the workload is now predictable

### What I Observed
The Pricing Calculator's "Save and share" generates a public URL that can be linked in portfolio posts. This is a real deliverable architects produce for clients — an estimate document with a shareable link.

### What I Learned
- For this startup configuration, the database (RDS) is a bigger cost driver than compute (EC2). In many real-world architectures, the database is the dominant cost. Right-sizing the database and enabling Multi-AZ only when needed are the highest-leverage cost optimisation decisions.
- Single-AZ RDS vs Multi-AZ roughly doubles the compute cost. The trade-off: Single-AZ is appropriate for dev/test; Multi-AZ is required for production.

---

## Lab 5: Activate AWS Cost Explorer

**Objective:** Enable Cost Explorer so historical spend data is available as the account accumulates usage.

### Steps
1. In the search bar type **Cost Explorer** → open it
2. If prompted, click **Enable Cost Explorer** (free to activate and use)
3. Explore Date Range and Group By filters

### Group By Options Most Useful for a Startup

| Group By | Why It's Useful |
|---|---|
| **Service** | See which AWS service is driving the most spend — helps identify cost optimisation targets |
| **Usage Type** | Break down costs within a service (e.g. EC2 data transfer vs compute vs storage) |

### What I Observed
On a new account, the Cost Explorer graph is empty — this is expected. It only shows data after usage has occurred. The 13-month look-back window means historical data accumulates automatically as you use the account.

### What I Learned
- Cost Explorer is reactive (shows what happened). AWS Budgets is proactive (alerts before it happens). A complete cost management strategy needs both.
- Enabling Cost Explorer as early as possible means the historical data will be available for analysis by the time the account has meaningful usage. If you wait 6 months to enable it, you miss 6 months of trend data.

---

## Bonus Challenge: Paystack Black Friday Surge

See full analysis in `/notes/real-world-scenarios.md`.

### Summary Findings

| Configuration | 24-hr Surge Cost (18 instances) |
|---|---|
| 18 × t3.medium On-Demand | ~$18.00 |
| 18 × t3.medium Spot | ~$5.40 |
| **Cost difference** | **~$12.60** |

**Recommendation:** On-Demand for the surge. The $12.60 saving is not worth the Spot interruption risk on the highest-transaction-day of the year. Full reasoning in `/notes/real-world-scenarios.md`.
