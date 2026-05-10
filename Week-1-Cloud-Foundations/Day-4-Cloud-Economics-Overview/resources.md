# Week 1 · Day 4 — Further Resources

---

## Official AWS Documentation

### 💰 AWS Cloud Pricing Overview
**URL:** [aws.amazon.com/pricing](https://aws.amazon.com/pricing)  
The master reference for pricing across all AWS services. Covers every service's billing model, pricing tiers, and regional differences. Bookmark this — you will return to it constantly.

---

### 🎁 AWS Free Tier Dashboard
**URL:** [aws.amazon.com/free](https://aws.amazon.com/free)  
Track your 12-month free, always-free, and trial limits in real time. Essential reading before running any lab to avoid unexpected charges. Know the three tier types cold before your exam.

---

### 📈 AWS Savings Plans
**URL:** [aws.amazon.com/savingsplans](https://aws.amazon.com/savingsplans)  
Detailed breakdown of the flexible discount model covering EC2, Lambda, and Fargate under one commitment. Explains the difference between Compute Savings Plans vs EC2 Instance Savings Plans — both are SAA-C03 exam territory.

---

### 📊 AWS Cost Explorer
**URL:** [aws.amazon.com/aws-cost-management/aws-cost-explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer)  
Visualise and forecast your AWS spend over 13 months. Free to activate and use. Learn the Group By filters — they map directly to exam questions about cost allocation and tagging strategies.

---

### 🔔 AWS Budgets
**URL:** [aws.amazon.com/aws-cost-management/aws-budgets](https://aws.amazon.com/aws-cost-management/aws-budgets)  
Set spend alerts before you exceed your budget. Supports cost budgets, usage budgets, and reservation budgets. Set up your $0 Zero Spend Budget on Day 1 of every new account.

---

### 🧮 AWS Pricing Calculator
**URL:** [calculator.aws](https://calculator.aws)  
Estimate your monthly AWS bill before committing to any architecture. Build shareable estimates and export them. Used directly in the Day 4 lab — get comfortable navigating it.

---

## Exam-Specific Reading

### EC2 On-Demand Pricing
**URL:** [aws.amazon.com/ec2/pricing/on-demand](https://aws.amazon.com/ec2/pricing/on-demand)  
Real per-hour and per-second rates by instance type and Region. Used in Step 3 of the lab. Useful for understanding how Region selection affects cost.

### EC2 Reserved Instance Pricing
**URL:** [aws.amazon.com/ec2/pricing/reserved-instances/pricing](https://aws.amazon.com/ec2/pricing/reserved-instances/pricing)  
Compare No Upfront, Partial Upfront, and All Upfront payment options. All Upfront gives the maximum discount; No Upfront gives flexibility. Know the trade-offs.

### EC2 Spot Instance Advisor
**URL:** [aws.amazon.com/ec2/spot/instance-advisor](https://aws.amazon.com/ec2/spot/instance-advisor)  
Shows historical Spot pricing and interruption frequency by instance type and Region. Used in the lab to record the Spot discount for t3.micro in us-east-1.

---

## Quick Reference

| Concept | One-Line Reminder |
|---|---|
| Pay-As-You-Go | No upfront, no termination fees, pay per second/request |
| Reserved Instances | 1–3 yr instance commitment → up to 72% off |
| Spot Instances | Up to 90% off, but 2-min termination notice — never for DBs |
| Savings Plans | Commit to $ spend level → up to 66% off EC2 + Lambda + Fargate |
| Free Tier (12-month) | EC2 t2.micro 750 hrs/month, S3 5 GB |
| Free Tier (Always free) | Lambda 1M requests/month |
| Free Tier (Trial) | e.g. Redshift 2-month trial |
| Cost Explorer | Historical + 13-month forecast — free to use |
| AWS Budgets | Proactive alerts — set $0 alert on account Day 1 |
| Pricing Calculator | Estimate costs before building |
