# AWS Cloud Foundations — Week 1 Day 4

## Topic
Cloud Economics Overview

This repository contains my notes, labs, screenshots, and portfolio artifacts from Day 4 of my AWS Cloud journey. Today's focus was understanding how AWS pricing works, how to choose the right pricing model for any workload, and how to set up cost controls before spending a single naira.

---

## What I Learned

### The Core Shift: CapEx → OpEx
Traditional IT required massive upfront capital expenditure (CapEx) — buying servers sized for peak demand that sat idle 90% of the time. AWS converts this to operational expenditure (OpEx) — pay only for what you use, per second, per request. Cash flow improves dramatically, and a Lagos startup can launch production infrastructure for less than the cost of one physical server.

### AWS Pricing Models

| Model | Commitment | Max Discount | Best For | Exam Watch |
|---|---|---|---|---|
| On-Demand | None | 0% | Dev/test, unpredictable workloads | Baseline pricing |
| Savings Plans | 1–3 yr (spend level) | 66% | EC2 + Lambda + Fargate flexibility | Commit to spend, not instance type |
| Reserved Instances | 1–3 yr (instance type) | 72% | Steady-state, predictable workloads | Instance type locked |
| Spot Instances | None | 90% | Fault-tolerant, interruptible batch jobs | 2-min termination — NEVER for databases |
| Free Tier | N/A | 100% | Learning, labs, new service trials | 3 types: 12-month, always-free, trial |

### AWS Free Tier — Three Types

| Type | Example | Limit |
|---|---|---|
| 12-Month Free | EC2 t2.micro/t3.micro | 750 hours/month for first 12 months |
| Always Free | Lambda | 1M requests + 400K GB-seconds/month, forever |
| Short-Term Trial | Amazon Redshift | 2-month trial for new services |

### Cost Management Tools
- **AWS Budgets** — Set spend alerts before going over (set zero alert on Day 1)
- **AWS Cost Explorer** — Historical spend analysis with 13-month forecasting
- **AWS Pricing Calculator** — Estimate costs before you build at calculator.aws

---

## Hands-On Labs Completed
- AWS Zero Spend Budget configured with email alert
- Free Tier dashboard explored — EC2, S3, Lambda limits documented
- All 4 pricing models compared using real EC2 pricing pages
- AWS Pricing Calculator — Lagos fintech startup cost estimate built (On-Demand vs Reserved)
- AWS Cost Explorer activated
- Bonus: Paystack Black Friday surge modelled — On-Demand vs Spot analysis

---

## AWS Services Explored
- **AWS Budgets** — Cost alerting
- **AWS Cost Explorer** — Spend visualisation and forecasting
- **AWS Pricing Calculator** — Pre-build cost estimation
- **EC2** — Pricing model comparison across all 4 models
- **S3** — Storage and request pricing
- **RDS** — Managed database pricing

---

## Screenshots
All screenshots stored in `/screenshots`:
- `budgets-zero-spend.png` — AWS Budgets dashboard showing Zero Spend Budget
- `pricing-calculator-estimate.png` — On-Demand vs Reserved monthly totals
- `pricing-model-comparison-table.png` — Completed pricing model table from Step 3
- `cost-explorer-activated.png` — Cost Explorer activated

---

## Challenges & Blockers
See `/notes/challenges.md`

---

## Goal
Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
