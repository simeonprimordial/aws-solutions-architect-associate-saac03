# Challenges & How I Solved Them — Day 2

---

## Challenge 1: Some Services Unavailable in af-south-1

**What happened:**
When I switched the console to af-south-1 (Cape Town), several AWS services showed as unavailable or greyed out compared to us-east-1.

**Why this happens:**
Not all AWS services are available in all Regions. Newer or less common Regions like af-south-1 have a smaller service catalogue than flagship Regions like us-east-1 (N. Virginia), which has the broadest service availability of any Region.

**Resolution:**
Checked the [AWS Regional Services List](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) to confirm which services are available in af-south-1 before designing any architecture for Nigerian-targeted workloads.

**Lesson learned:**
Always verify service availability in your target Region before committing to an architecture. For Nigerian use cases, af-south-1 covers the core services (EC2, S3, RDS, VPC, IAM) but may lack niche managed services.

---

## Challenge 2: Confusing Edge Locations with AZs

**What happened:**
During the infrastructure map exploration, I initially confused Edge Locations with Availability Zones since both represent physical AWS infrastructure.

**How I clarified it:**
The key distinction is **what you can run there**:
- **AZ** → full compute, storage, databases, networking — your workloads live here
- **Edge Location** → cache only (CloudFront static content) and DNS (Route 53) — no compute workloads

**Lesson learned:**
Think of Edge Locations as CDN caches, not data centres. The moment a question mentions deploying servers or databases "close to users," the answer is a Local Zone or a nearby Region — not an Edge Location.

---

## Challenge 3: Understanding Which Region to Use for Nigerian Users

**What happened:**
I wasn't sure whether to use af-south-1 (Cape Town) or a European region for Nigerian users, since both are geographically separated.

**How I resolved it:**
- Ran a latency test using [cloudpingtest.com](https://cloudpingtest.com) from my location
- af-south-1 latency from Lagos: ~85ms
- eu-west-1 (Ireland) latency from Lagos: ~140ms
- us-east-1 (Virginia) latency from Lagos: ~185ms

**Resolution:**
af-south-1 is the clear choice for latency. Additionally, using af-south-1 keeps data on the African continent — critical for NDPC compliance in regulated sectors.

**Lesson learned:**
Latency testing with real numbers is more convincing than theory. These measurements are now in my Region cheat sheet as real-world evidence.
