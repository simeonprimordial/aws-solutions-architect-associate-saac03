# Cloud Concepts — Day 1 Notes

---

## What Is Cloud Computing?

Cloud computing is the delivery of computing services — servers, storage, databases, networking, and software — over the internet.

Instead of buying and maintaining physical servers, businesses rent resources on-demand from providers like AWS.

### Benefits
- Lower upfront costs
- Scalability on demand
- High availability and fault tolerance
- Faster deployment cycles
- Pay-as-you-go pricing model

---

## Five Essential Characteristics

### 1. On-Demand Self-Service
Provision resources instantly without human interaction from the provider. You spin up an EC2 instance yourself through the console or CLI — no waiting for a ticket to be approved.

### 2. Broad Network Access
Access cloud services from any device over the internet — laptop, phone, tablet.

### 3. Resource Pooling
Multiple customers share the same physical infrastructure securely using **multi-tenancy**. AWS manages the isolation between customers.

### 4. Rapid Elasticity
Scale resources up or down automatically based on demand. A website can handle a Black Friday spike and scale back down afterward, paying only for what was used.

### 5. Measured Service
Pay only for the resources consumed. Usage is monitored, controlled, and billed transparently.

> ⚠️ **Exam Tip:** There are exactly FIVE essential characteristics — not four, not six. Memorise them.

---

## Deployment Models

| Model | Description | Example |
|---|---|---|
| Public Cloud | Shared infrastructure, managed by provider | AWS, Azure, GCP |
| Private Cloud | Dedicated environment for one organisation | On-prem VMware |
| Hybrid Cloud | Mix of on-premises and public cloud | AWS + data center |
| Multi-Cloud | Multiple cloud providers used together | AWS + GCP |

---

## Common Exam Traps

- Cloud computing has **FIVE** essential characteristics — not four.
- On-Demand pricing has **no upfront commitment**.
- Reserved Instances and Savings Plans **require a commitment** (1 or 3 years).
- Understand the **shared responsibility model** — AWS secures the cloud, you secure what's in the cloud.
