# Real-World Scenarios — Day 4

---

## Scenario 1: Paystack — Before and After Cloud

### Before: Physical Servers (CapEx Model)

| Cost Item | Amount |
|---|---|
| Production-grade server purchase | $50,000 (one-time) |
| Data centre colocation fees | $15,000/year |
| Dedicated sysadmin salary | $8,000/year |
| **Year One Total** | **$73,000+** |

**Hidden costs of physical servers:**
- Fixed capacity — idle 90% of the time, but fully paid for 100% of the time
- 6–12 weeks to procure and configure hardware (time to market impact)
- If servers fail during peak, no rapid scaling option
- Hardware refresh every 3–5 years = another $50,000 cycle

### After: AWS (OpEx Model)

| Scenario | Monthly Cost |
|---|---|
| Startup launch (minimal traffic) | ~$300/month |
| Growth phase (moderate traffic) | ~$2,000/month |
| Peak events (Black Friday, payday) | Auto-scales, pay only for the surge hours |

**Year One AWS saving vs physical:** ~$69,400
**Additional benefits:**
- Launch in days, not months
- Auto-scales for Black Friday payment surges — no manual server provisioning
- AWS manages underlying infrastructure — Paystack team focuses 100% on product
- Pay per transaction processed — almost zero cost at 3am, appropriate cost at 12pm payday

---

## Scenario 2: Lagos Fintech Startup — Pricing Calculator Estimate

**Scenario:** You are the cloud architect for a new Lagos fintech startup. Build a 3-month AWS cost estimate for their MVP infrastructure.

### Configuration

| Service | Configuration | Pricing Model |
|---|---|---|
| EC2 | 1 × t3.micro, Linux, us-east-1, 730 hrs/month | On-Demand |
| S3 | 10 GB Standard storage, 50K GET, 5K PUT requests/month | Standard |
| RDS | db.t3.micro, MySQL, Single-AZ, 20 GB General Purpose SSD | On-Demand |

### Monthly Estimates (from AWS Pricing Calculator)

| Configuration | Monthly Estimate | 3-Month Total |
|---|---|---|
| EC2 (On-Demand) + S3 + RDS | ~$38.50 | ~$115.50 |
| EC2 (Reserved 1-yr, No Upfront) + S3 + RDS | ~$26.20 | ~$78.60 |
| **Savings with Reserved (3-yr projection)** | **~$12.30/mo** | **~$443/3 years** |

### Key Observations
- The total cost is surprisingly low for a production MVP — this is the cloud advantage
- The Free Tier covers the EC2 t2.micro entirely for the first 12 months — so actual Year 1 EC2 cost could be $0
- Switching to Reserved Instances saves ~32% on the EC2 line item alone
- S3 costs are minimal at this scale — Free Tier covers the first 5 GB; 10 GB adds ~$0.23/month
- RDS Single-AZ is cheaper — switching to Multi-AZ would roughly double the compute cost but add fault tolerance

### Architect's Recommendation for the Startup
At MVP stage: Use On-Demand + Free Tier. Commit zero capital, validate the product first.
At 6 months with proven traffic: Switch EC2 and RDS to 1-year Reserved Instances. The savings compound over time and the workload is now predictable.

---

## Scenario 3: Paystack Black Friday Surge (Bonus Challenge)

### The Problem
Paystack expects 10× normal traffic on Black Friday (1 day only).
- Normal load: 2 × t3.medium EC2 instances (running 24/7 year-round)
- Surge load: 10× = needs 20 × t3.medium for 24 hours on Black Friday

### Pricing Model Comparison for the Surge

**Option A: On-Demand for surge (18 additional instances)**
- t3.medium On-Demand rate (us-east-1): ~$0.0416/hr
- 18 instances × $0.0416/hr × 24 hours = **~$18.00 for the Black Friday surge**

**Option B: Spot Instances for surge (18 additional instances)**
- t3.medium Spot rate (us-east-1, approx): ~$0.0125/hr (70% discount)
- 18 instances × $0.0125/hr × 24 hours = **~$5.40 for the Black Friday surge**
- Spot savings vs On-Demand: **~$12.60 for a single surge day**

### My Recommendation (Written as Required by Lab)

For Paystack's Black Friday payment processing surge, I would recommend **On-Demand instances for the surge — not Spot Instances.**

The cost difference between On-Demand and Spot for the entire Black Friday surge is approximately $12.60. That is a negligible saving for a company processing millions of naira in transactions over 24 hours. Spot Instances can be terminated by AWS with only 2 minutes notice if AWS needs the capacity back. On Black Friday — the highest-demand day of the year for global cloud resources — Spot Instance interruption risk is at its peak, because every company in the world is trying to scale simultaneously. If Paystack's surge instances are interrupted mid-transaction, the consequences are payment failures, chargebacks, merchant complaints, and regulatory scrutiny from the CBN. No responsible architect would risk $12.60 in savings against the reputational and financial cost of a payment outage on the most important trading day of the year.

**The correct architecture:**
- 2 × t3.medium Reserved (1-yr) for the always-on baseline — maximum savings on predictable load
- 18 × t3.medium On-Demand for the 24-hour Black Friday surge — reliable, interruptible-free, and trivially cheap for a 24-hour window
- Auto Scaling Group manages the surge automatically — no manual instance launches on the day

---

## Discussion — Nigerian Business Application

**Prompt from class:** Think about a Nigerian business you know. How would cloud economics change how they operate?

**My answer:**

**Business: A Nigerian logistics/last-mile delivery startup**

**Current problem:** The startup needs tracking infrastructure (server, database, map API) to show real-time delivery locations to customers. The CapEx cost of buying a server and getting a data centre connection before they have a single customer is a barrier to even starting.

**Cloud economics impact:**
- Launch an MVP with EC2 t3.micro + RDS db.t3.micro + S3 for under $40/month On-Demand — or $0 on Free Tier for Year 1
- Scale servers only when delivery volume grows — no wasted capacity in early months
- Use Spot Instances for nightly route optimisation batch jobs (running delivery route algorithms overnight) — saves 90% on what would otherwise be their biggest compute bill
- Reserve capacity (Reserved Instances) only after 6 months when traffic patterns are proven

**Business impact:** A startup that previously couldn't afford to build because of hardware procurement costs can now launch, validate, and scale — only spending money when they're serving customers. This is why Nigerian tech companies can compete with global players without raising $10M Series A just to buy servers.
