# AWS Pricing Calculator Notes — Day 1

---

## Exercise

**Scenario:** Estimate the monthly cost of running one `t2.micro` EC2 instance in the Cape Town Region (`af-south-1`) for 24/7 usage (730 hours/month).

**Calculator URL:** https://calculator.aws/

---

## Configuration Used

| Parameter | Value |
|---|---|
| Region | Africa (Cape Town) — af-south-1 |
| Instance type | t2.micro |
| Operating System | Linux |
| Usage | 730 hours/month (24/7) |
| Storage | 8 GB gp2 EBS (default) |
| Data transfer | Minimal (outbound < 1 GB) |

---

## Estimate Result

| Line Item | Monthly Cost (USD) |
|---|---|
| EC2 t2.micro (730 hrs) | ~$12.41 |
| EBS gp2 storage (8 GB) | ~$0.88 |
| Data transfer (minimal) | ~$0.00 |
| **Total Estimate** | **~$13.29 / month** |

> Note: The t2.micro is **Free Tier eligible** for the first 12 months (750 hrs/month). After Free Tier expires, the above cost applies.

---

## Key Learnings

AWS pricing varies by:
- **Region** — af-south-1 is slightly more expensive than us-east-1 due to lower infrastructure density in Africa.
- **Instance type** — CPU and memory specs directly affect cost. t2.micro (1 vCPU, 1 GB RAM) is the smallest general-purpose option.
- **Storage** — EBS volumes are billed separately from EC2 compute. An instance stopped but not terminated still incurs storage charges.
- **Data transfer** — Inbound data to AWS is free. Outbound data charges apply above 100 GB/month.
- **Pricing model** — On-Demand (no commitment) vs Reserved Instances (1–3 year commitment, up to 72% cheaper) vs Spot Instances (spare capacity, up to 90% cheaper but can be interrupted).

---

## Pricing Model Comparison (t2.micro, us-east-1, 1 year)

| Model | Monthly Cost | Commitment |
|---|---|---|
| On-Demand | ~$8.47 | None |
| 1-Year Reserved (No Upfront) | ~$5.11 | 1 year |
| 1-Year Reserved (All Upfront) | ~$4.38 | 1 year + full payment |
| Spot Instance | ~$2.50 (variable) | None (interruptible) |

> ⚠️ **Exam Tip:** Spot Instances are cheapest but can be terminated by AWS with 2 minutes notice. Never use Spot for critical workloads.
