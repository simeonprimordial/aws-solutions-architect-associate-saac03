# Challenges & How I Solved Them — Day 4

---

## Challenge 1: Understanding the Difference Between Savings Plans and Reserved Instances

**What happened:**
Both Savings Plans and Reserved Instances offer discounts for commitment — I initially thought they were the same thing with different names.

**The actual difference:**

| Dimension | Reserved Instances | Savings Plans |
|---|---|---|
| What you commit to | Specific instance type + Region | A dollar spend level per hour |
| Flexibility | Low — locked to instance type | High — applies across EC2 families, Lambda, Fargate |
| Max discount | 72% | 66% |
| Instance family change | Not allowed (Standard RI) | Allowed automatically |

**The mental model that helped:**
- Reserved Instances = "I promise to use this exact t3.medium in us-east-1 for 1 year."
- Savings Plans = "I promise to spend at least $5/hr on compute for 1 year, use whatever you want."

**When each makes sense:**
- Use Reserved Instances when your instance type and Region are completely fixed (e.g. a specific database server that will never change).
- Use Savings Plans when you want flexibility to change instance types as your architecture evolves — common for growing startups.

**Lesson learned:**
Savings Plans is the more modern, flexible option. AWS now recommends Savings Plans over Reserved Instances for most use cases. The exam will test whether you know which applies when.

---

## Challenge 2: Spot Instance Interruption Probability

**What happened:**
The lab asked me to evaluate whether Spot Instances were appropriate for the Paystack Black Friday surge. My first instinct was "yes — 90% discount, huge savings." I had to think through the interruption risk more carefully.

**The key insight I missed initially:**
Spot Instance interruption risk isn't constant. It's highest precisely when you need capacity most — during global peak demand events. Black Friday is a global peak demand event for AWS compute. Every retailer, every payment processor, every logistics company worldwide is scaling simultaneously. AWS's spare capacity shrinks dramatically. Spot Instance interruption probability spikes.

**Resolution:**
The correct analysis isn't "Spot is cheaper, therefore use Spot." It's "what is the cost of an interruption?" For batch jobs: low cost (just restart). For payment processing on the highest-transaction-day of the year: catastrophic. The $12.60 On-Demand premium is not a cost — it's insurance.

**Lesson learned:**
Always evaluate Spot Instances in context of both the financial saving AND the business risk of interruption. The exam rewards this nuanced thinking.

---

## Challenge 3: Free Tier Expiry Awareness

**What happened:**
While exploring the Free Tier dashboard, I noticed services I'd been using since account creation (Day 1) are on the 12-month clock. I had assumed Free Tier was "just free" without tracking the expiry.

**The specific risk:**
- EC2 t2.micro: Free for 750 hours/month for 12 months. At Month 13, billing starts automatically.
- RDS db.t3.micro: Free for 750 hours/month for 12 months. Same expiry.
- S3: 5 GB free for 12 months — if I'm storing screenshots and PDFs, this could exceed 5 GB.

**Resolution:**
- Set a calendar reminder at Month 10 to review which resources are running and evaluate whether to switch to Reserved Instances, resize, or shut down before billing kicks in
- Enabled AWS Budgets Zero Spend Budget — will get an email the moment any charge appears
- Checked current Free Tier usage in the AWS Free Tier Dashboard — accessible via the Billing console

**Lesson learned:**
Free Tier is not a permanent safety net. It's a runway for learning. Treat it as a 12-month trial, not a permanent state, and plan the transition to paid tiers deliberately.
