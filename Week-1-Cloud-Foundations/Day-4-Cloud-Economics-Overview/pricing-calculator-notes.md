# AWS Pricing Calculator Notes — Day 4

**Calculator URL:** https://calculator.aws

---

## Lab Exercise: Lagos Fintech Startup — 3-Month Cost Estimate

**Scenario:** You are the cloud architect for a Lagos fintech startup. Build a cost estimate for their initial infrastructure.

### Configuration Used

| Service | Configuration | Details |
|---|---|---|
| EC2 | 1 × t3.micro, Linux | On-Demand, us-east-1, 730 hrs/month |
| S3 | Standard storage | 10 GB/month, 50,000 GET, 5,000 PUT |
| RDS | db.t3.micro, MySQL | Single-AZ, 20 GB General Purpose SSD, us-east-1 |

---

## Results

### On-Demand Estimate

| Line Item | Monthly Cost (USD) |
|---|---|
| EC2 t3.micro (On-Demand, 730 hrs) | ~$7.59 |
| S3 (10 GB storage + requests) | ~$0.28 |
| RDS db.t3.micro Single-AZ (MySQL) | ~$13.10 |
| **Monthly Total** | **~$20.97** |
| **3-Month Total** | **~$62.91** |

> Note: All three services fall within AWS Free Tier for the first 12 months (EC2 750 hrs, S3 5 GB free, RDS 750 hrs). Real costs above apply after Free Tier expires.

---

### Reserved Instance Estimate (EC2 switched to 1-Year, No Upfront)

| Line Item | Monthly Cost (USD) |
|---|---|
| EC2 t3.micro (Reserved 1-yr, No Upfront) | ~$4.38 |
| S3 (same as above) | ~$0.28 |
| RDS db.t3.micro Single-AZ (same) | ~$13.10 |
| **Monthly Total** | **~$17.76** |
| **3-Month Total** | **~$53.28** |

---

### Savings Comparison

| Configuration | Monthly | 12-Month | 36-Month |
|---|---|---|---|
| EC2 On-Demand + S3 + RDS | ~$20.97 | ~$251.64 | ~$754.92 |
| EC2 Reserved (1-yr) + S3 + RDS | ~$17.76 | ~$213.12 | ~$639.36 |
| **Savings with Reserved** | **~$3.21/mo** | **~$38.52** | **~$115.56** |

> For this small workload, the absolute saving is modest. The Reserved Instance model's value scales significantly with larger instance types and multiple instances running 24/7.

---

## Pricing Model Comparison — EC2 t3.micro (us-east-1)

| Pricing Model | Price/hr | Monthly (730 hrs) | Commitment |
|---|---|---|---|
| On-Demand | ~$0.0104 | ~$7.59 | None |
| Reserved (1-yr, No Upfront) | ~$0.006 | ~$4.38 | 1 year |
| Reserved (1-yr, All Upfront) | ~$0.005 | ~$3.65 (amortised) | 1 year + full upfront |
| Spot (varies) | ~$0.003–0.005 | ~$2–4 (variable) | None (interruptible) |
| Free Tier (t2.micro) | $0.00 | $0.00 (up to 750 hrs) | 12-month account only |

---

## Bonus: Paystack Black Friday Surge Model

**Scenario:** Paystack expects 10× normal traffic for 1 day. Normal load = 2 × t3.medium instances.

### Setup
- **Baseline:** 2 × t3.medium Reserved (1-yr) for year-round traffic
- **Surge:** 18 additional t3.medium instances for 24 hours only (Black Friday)

### Cost Comparison for 24-Hour Surge (18 × t3.medium)

| Model | Price/hr (t3.medium) | 24-hr Cost (18 instances) |
|---|---|---|
| On-Demand | ~$0.0416 | ~$17.97 |
| Spot | ~$0.013 (variable) | ~$5.62 |
| **Saving with Spot** | | **~$12.35** |

### Recommendation

**Use On-Demand for the Black Friday surge. Do NOT use Spot.**

Paystack processes live payment transactions. A Spot interruption mid-transaction during Black Friday — the single highest-revenue day of the year — could mean:
- Failed payment transactions reaching customers
- Incomplete database writes requiring reconciliation
- Regulatory and reputational consequences
- Customer support volume that costs more than $12.35 to resolve

The $12.35 saving from Spot vs On-Demand is not a rational trade for this risk profile. On-Demand instances guarantee no AWS-initiated interruption. For the surge capacity on a payment processor's peak day, the correct answer is On-Demand — not the cheapest option.

**Architecture recommendation for next year:** Pre-warm Reserved capacity for the expected peak level, use On-Demand for overflow above that. Plan 3 months ahead so the Reserved Instances are active before Black Friday.
