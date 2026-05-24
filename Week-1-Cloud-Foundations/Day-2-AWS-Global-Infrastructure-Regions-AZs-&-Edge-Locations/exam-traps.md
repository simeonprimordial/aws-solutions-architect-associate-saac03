# Exam Traps & Practice Questions — Day 2

---

## The 3 Most Dangerous Exam Traps

### Trap 1 — AZs use private fibre, NOT the public internet
**Wrong thinking:** "AZs must connect over the internet like everything else."
**Correct:** AZs within a Region are connected via **low-latency private fibre networks**. This is what makes Multi-AZ failover fast (milliseconds), not slow like an internet round-trip.

**How it appears on the exam:** A question about Multi-AZ failover speed, or asking what connects AZs — the wrong answers will say "public internet" or "VPN".

---
[text](exam-traps.md) [text](global-infrastructure.md) [text](labs.md) [text](README.md) [text](real-world-scenarios.md) [text](region-cheat-sheet.md) [text](challenges.md)
### Trap 2 — Edge Locations are NOT Availability Zones
**Wrong thinking:** "Edge Locations are just smaller AZs, I can deploy servers there."
**Correct:** Edge Locations handle **caching (CloudFront) and DNS (Route 53) only**. You cannot launch EC2 instances, RDS databases, or any primary compute there.

**How it appears on the exam:** A scenario asking where to deploy a low-latency application "close to users" — the correct answer is a Local Zone or a Region, not an Edge Location.

---

### Trap 3 — Data does NOT automatically leave a Region
**Wrong thinking:** "AWS automatically backs up my data to another region for safety."
**Correct:** Data stays inside the Region you chose **unless you explicitly configure** Cross-Region Replication (CRR) for S3, RDS read replicas, or Route 53 failover.

**How it appears on the exam:** Compliance/data sovereignty questions. The correct answer will always reference an explicit configuration step — not an automatic AWS behaviour.

---

## SAA-C03 Practice Question — Day 2

**Question:**
A Solutions Architect at a Lagos fintech company is designing the AWS infrastructure for a new payment platform. They need to implement best practices related to AWS Global Infrastructure. Which of the following statements is CORRECT?

**A.** Data stored within a specific AWS Region is not replicated to other Regions automatically unless explicitly configured by the architect.

**B.** An AWS Region consists of a single, massive data centre that serves an entire geographic area.

**C.** Availability Zones within a Region are connected to each other over the public internet to ensure maximum global reach.

**D.** Edge Locations are specialised Availability Zones where the architect can deploy primary EC2 database servers for localised compute.

---

**Answer: A**

**Why A is correct:**
Data sovereignty and containment are foundational AWS principles. By default, data stays in the Region where it was created. For a fintech company handling Nigerian payment data, this is the expected behaviour — and it requires deliberate action to replicate across Regions.

**Why B is wrong:**
A Region contains **multiple** data centres spread across 2+ physically separated Availability Zones. A single data centre would be a massive single point of failure.

**Why C is wrong:**
AZs are connected via **private fibre**, not the public internet. Public internet connectivity would introduce latency, security risks, and unreliable failover speeds.

**Why D is wrong:**
Edge Locations are for CloudFront caching and Route 53 DNS only. You cannot deploy EC2 instances or databases there.

---

## Quick Recall Quiz

Test yourself — cover the answers and answer from memory:

| Question | Answer |
|---|---|
| How many Regions does AWS have? (2026) | 39 |
| How many AZs does AWS have? (2026) | 123 |
| How many Edge Locations? | 400+ |
| Minimum AZs per Region? | 2 |
| What connects AZs within a Region? | Private low-latency fibre |
| What service uses Edge Locations for CDN? | Amazon CloudFront |
| What service uses Edge Locations for DNS? | Amazon Route 53 |
| Closest Region to Nigeria? | af-south-1 (Cape Town) |
| Does data auto-replicate across Regions? | No — must be explicitly configured |
| Can you launch EC2 in an Edge Location? | No |
