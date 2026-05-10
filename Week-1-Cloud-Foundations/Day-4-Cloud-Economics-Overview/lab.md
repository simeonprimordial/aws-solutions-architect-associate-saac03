# Week 1 · Day 4 · Lab — Build Your AWS Cost Control Centre
---

## Step 1 — Activate Your Cost Safety Net (AWS Budgets)

> The very first thing you do on a new AWS account is set a $0 budget alert.

1. In the AWS Console search bar, type **Budgets** → click **AWS Budgets**
2. Click **Create budget** → **Use a template** → **Zero spend budget**
3. Enter your email address for notifications → click **Create budget**
4. Document in your tracker:

| Field | Your Value |
|---|---|
| Budget name | |
| Alert threshold | $0.01 |
| Email set | |


---

## Step 2 — Explore the Free Tier Dashboard

1. In the search bar type **Free Tier** → open the AWS Free Tier page
2. Locate EC2, S3, and Lambda in the service list
3. Fill in this table and add it to your services tracker:

| Service | Free Tier Type | Monthly Limit |
|---|---|---|
| EC2 (t2.micro / t3.micro) | 12-month free | 750 hours / month |
| S3 | 12-month free | 5 GB storage · 20K GET · 2K PUT |
| Lambda | Always free | 1M requests / month |

---

## Step 3 — Compare All Four Pricing Models in the EC2 Console

> You will look at real pricing data — without launching any instance.

1. Search **EC2** → left sidebar → **Instances** → **Launch Instances** (stop at the first screen)
2. Note the instance type dropdown — select **t3.micro** as your On-Demand baseline
3. Visit these pages and record the data below:
   - On-Demand: `aws.amazon.com/ec2/pricing/on-demand`
   - Reserved: `aws.amazon.com/ec2/pricing/reserved-instances/pricing`
   - Spot: `aws.amazon.com/ec2/spot/instance-advisor` (search t3.micro in us-east-1)

**Pricing Comparison Table (fill in):**

| Pricing Model | Price / hr (t3.micro) | Commitment | Best For |
|---|---|---|---|
| On-Demand | $________ | None | Unpredictable load |
| Reserved (1-yr, No Upfront) | $________ | 1 year | Steady workloads |
| Spot (t3.micro, us-east-1) | ~$________ (varies) | None | Fault-tolerant batch |
| Free Tier (t2.micro) | $0.00 (up to 750 hrs) | 12 months | Learning & labs |

---

## Step 4 — Build a Startup Cost Estimate (AWS Pricing Calculator)

**Scenario:** You are the cloud architect for a Lagos fintech startup. Build their 3-month AWS cost estimate.

1. Go to `calculator.aws` → click **Create estimate**
2. **Add EC2:** 1 × t3.micro, On-Demand, Linux, us-east-1, 730 hrs/month
3. **Add S3:** Standard storage, 10 GB/month, 50,000 GET requests, 5,000 PUT requests
4. **Add RDS:** db.t3.micro, MySQL, Single-AZ, 20 GB General Purpose SSD
5. Review the monthly total → click **Save and share** → copy public link
6. Now change EC2 to **Reserved Instance (1-year, No Upfront)** → record the new total

---

## Step 5 — Tour AWS Cost Explorer

1. Search **Cost Explorer** → open it
2. If prompted, click **Enable Cost Explorer** (free to activate and use)
3. Explore the **Date Range** and **Group By** filters
4. On a new account the graph will be empty

---

## Screenshot Checklist

- [ ] AWS Budgets dashboard showing your $0 Zero Spend Budget
- [ ] Pricing Calculator estimate showing On-Demand vs Reserved monthly totals
- [ ] Completed pricing model comparison table (Step 3)
- [ ] Cost Explorer activated (even if graph is empty)

---

## 🔥 Bonus Challenge — Paystack Black Friday Surge

**Scenario:** Paystack expects 10× normal traffic on Black Friday (1 day only).  
Normal load = 2 × t3.medium EC2 instances.

**Task 1 — Model the surge in Pricing Calculator:**
- 2 × t3.medium (Reserved, 1-yr) for baseline
- 18 × t3.medium (On-Demand) for the 24-hour Black Friday surge

**Task 2 — Replace surge with Spot Instances:**
- Swap the 18 On-Demand surge instances for Spot Instances
- Estimate and record the cost difference

**Task 3 — Written recommendation:**  
Write one paragraph explaining which model you would recommend for the surge, and why Spot may not be appropriate for Paystack's payment processing core.


---

## Portfolio Post Template

```
Just set up my AWS cost controls:
✅ $0 budget alert
✅ Free Tier tracker
✅ Pricing Calculator estimate
✅ On-Demand vs Reserved comparison


