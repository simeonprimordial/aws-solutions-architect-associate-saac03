# Real-World Scenarios — Day 2

---

## Scenario: Adaeze's Nollywood Streaming Platform

**Business:** A video streaming platform serving Nollywood content to Nigerian and diaspora audiences.

**Architecture Decision:** Deploy in af-south-1 (Cape Town) across two Availability Zones.

### Setup
- **AZ-1 (af-south-1a):** Primary application servers, database master
- **AZ-2 (af-south-1b):** Standby servers, database replica
- **CloudFront:** Distributes video files from Edge Locations in Lagos, Johannesburg, and Nairobi

### What Happens When AZ-1 Goes Down

```
Normal state:
  Users → Load Balancer → AZ-1 (primary) ✅
                        → AZ-2 (standby) 

After AZ-1 failure (e.g. electrical transformer fault):
  Users → Load Balancer → AZ-1 ❌ (DOWN)
                        → AZ-2 ✅ (now handling all traffic)
```

**Result:** Zero downtime for users. The load balancer detects the failure in seconds and routes all traffic to AZ-2. Users in Kano loading a video see no interruption because the video file is already cached at the Lagos Edge Location — the origin server failure in AZ-1 doesn't affect content already in CloudFront's cache.

### Key Insight
The combination of Multi-AZ deployment + CloudFront provides two independent layers of resilience:
1. **Compute resilience** — Multi-AZ protects against data centre failures
2. **Content resilience** — CloudFront serves cached content even if the origin is temporarily unreachable

---

## Scenario: Lagos Fintech — Payment Data Compliance

**Business:** A Nigerian payment processing company handling cardholder data and PII under NDPC.

**The Problem:** The company needs to prove to regulators that customer data stays within Africa.

**AWS Solution:**
- Deploy everything in af-south-1 (Cape Town)
- **Do NOT configure** S3 Cross-Region Replication to any non-African region
- Use AWS Config to monitor and alert on any unintended cross-region data movement
- Document the Region selection in a data processing impact assessment

**Why this works:**
AWS guarantees that data placed in af-south-1 does not leave af-south-1 without explicit configuration. This default containment is the foundational premise of data sovereignty on AWS.

**The exam version of this scenario:**
Any question mentioning "data residency," "compliance," "data sovereignty," or "data must stay in [country/region]" → The correct answer will involve choosing the right Region and confirming that cross-region replication is NOT enabled.

---

## Discussion — Businesses You Know

**Prompt from class:** Think about a Nigerian business you know. How would this AWS infrastructure change how they operate?

**My answer:**

**Business: A Nigerian e-commerce platform (e.g. Jumia-style marketplace)**

**Current problem:** During flash sales or peak periods (Black Friday, 12.12), the site slows to a crawl or goes down entirely — losing sales at the highest-demand moments.

**AWS solution:**
- Deploy in af-south-1 across 3 AZs with Auto Scaling on EC2 — handles demand spikes automatically
- CloudFront Edge Location in Lagos caches product images and static pages — the most common bottleneck during traffic spikes
- Multi-AZ RDS for the product catalogue database — no single data centre failure takes down the whole site

**Business impact:** A platform that stays up during peak sales, serves pages faster to Nigerian users, and maintains uptime SLAs — directly translating to revenue protected and customer trust maintained.
