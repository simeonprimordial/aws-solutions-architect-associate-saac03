# AWS Global Infrastructure — Day 2 Notes

---

## Overview — Why This Matters

AWS operates one of the largest and most reliable cloud infrastructures in the world — 39 Regions and 123 Availability Zones as of 2026, with continuous expansion.

Understanding this infrastructure is critical for:
- Designing systems that are **resilient** (survive failures)
- Achieving **low latency** for your target users
- Maintaining **data sovereignty** compliance (especially for Nigerian fintech and healthcare)

For Nigerian businesses, the nearest AWS Region is **af-south-1 (Cape Town)**. As Africa's digital infrastructure matures, understanding latency, data residency, and disaster recovery will become increasingly important.

---

## Key Concepts

### AWS Region
A geographic area containing **2 or more Availability Zones**, physically isolated from every other Region.

- Each Region is completely independent
- Data does **not** automatically leave a Region — cross-region replication must be explicitly configured
- You select a Region when creating most AWS resources
- Region code format: `continent-direction-number` (e.g. `af-south-1`, `us-east-1`, `eu-west-2`)

**Africa Region:** `af-south-1` — Cape Town, South Africa

---

### Availability Zone (AZ)
One or more discrete data centres with **independent power, cooling, and networking** within a Region.

- Each Region has at least 2 AZs (most have 3–6)
- AZs are physically separated by tens of miles — far enough that a single flood, earthquake, or power failure won't affect more than one
- AZs within a Region are connected via **ultra-low latency private fibre** (NOT the public internet)
- AZ code format: Region code + letter (e.g. `af-south-1a`, `af-south-1b`, `us-east-1a`)

**Why it matters:** Deploying your application across 2+ AZs means if one data centre goes down, traffic automatically routes to the other. This is called **Multi-AZ architecture**.

---

### Edge Location (Point of Presence / PoP)
A smaller AWS facility distributed globally — **400+ locations** — used to cache content close to end users.

**Used by:**
- **Amazon CloudFront** (CDN) — caches static content (images, videos, CSS) near users
- **Amazon Route 53** (DNS) — resolves DNS queries from the nearest PoP

**Example:** A user in Abuja requests a video hosted in us-east-1. Without CloudFront, the request travels to Virginia and back — high latency. With CloudFront, the video is cached at the Lagos Edge Location and served in milliseconds.

> ⚠️ **Critical:** Edge Locations are NOT Availability Zones. You **cannot** launch EC2 instances or RDS databases in Edge Locations. They handle caching and DNS only.

---

### Local Zone
An extension of an AWS Region that places **compute, storage, and database** services closer to specific large metropolitan areas not covered by a full Region.

- Useful for workloads requiring single-digit millisecond latency to a specific city
- Example: AWS Local Zone in Los Angeles connected to the us-west-2 (Oregon) Region

---

### Wavelength Zone
AWS infrastructure **embedded directly inside telecom provider networks** — designed for ultra-low latency 5G mobile applications.

- Traffic stays inside the telecom network — never touches the public internet on the way to AWS
- Target latency: single-digit milliseconds
- Use case: real-time gaming, autonomous vehicles, live video processing over 5G

---

## How It All Connects

```
AWS Global Infrastructure
│
├── Region (af-south-1 — Cape Town)
│   ├── Availability Zone A (af-south-1a)  ←──┐
│   │   └── Data centres                       │ Private fibre
│   └── Availability Zone B (af-south-1b)  ←──┘ (low latency)
│
└── Edge Locations (Lagos, Johannesburg, Nairobi...)
    └── CloudFront cache + Route 53 DNS
```

---

## Infrastructure Comparison Table

| Component | Count (2026) | Purpose | Can Deploy EC2? |
|---|---|---|---|
| Region | 39 | Geographic boundary, data residency | Yes |
| Availability Zone | 123 | Fault isolation within a region | Yes |
| Edge Location | 400+ | Content caching, DNS | No |
| Local Zone | ~30 | Low latency for specific cities | Yes |
| Wavelength Zone | ~20 | Ultra-low latency 5G apps | Yes |

---

## Data Sovereignty — Critical for Nigeria

Under the **Nigeria Data Protection Commission (NDPC)** framework, organisations processing Nigerian personal data must understand where that data is stored and processed.

Key AWS behaviour:
- Data stored in af-south-1 stays in af-south-1 **unless you configure replication**
- Cross-Region replication (e.g. af-south-1 → eu-west-1) must be explicitly enabled
- This default containment is what makes AWS usable for regulated Nigerian sectors (fintech, healthcare, government)

> ⚠️ **Exam Trap:** Data does NOT automatically leave a Region. Cross-region replication must be explicitly configured. Always.
