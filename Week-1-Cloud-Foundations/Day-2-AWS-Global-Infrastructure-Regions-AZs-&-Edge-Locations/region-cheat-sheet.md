# AWS Regions Cheat Sheet — Week 1

Built during Week 1 of the AWS Cloud Accelerator course.
Part of my journey to AWS SAA-C03 certification. | #TechAfrica

---

## AWS Regions Reference

| Region Name | Code | AZs | Key Services | Notes |
|---|---|---|---|---|
| US East (N. Virginia) | us-east-1 | 6 | All services | Most services launch here first. Default region for many tutorials. |
| US West (Oregon) | us-west-2 | 4 | All services | Common for US West Coast and global workloads. |
| EU (Ireland) | eu-west-1 | 3 | All services | Popular GDPR-compliant EU region. ~140ms from Lagos. |
| Africa (Cape Town) | af-south-1 | 3 | Core services | Closest Region to Nigeria. Requires opt-in. ~85ms from Lagos. |
| Asia Pacific (Singapore) | ap-southeast-1 | 3 | All services | Hub for Southeast Asian workloads. |
| South America (São Paulo) | sa-east-1 | 3 | Most services | Closest Region for Brazilian users. |

**Global count (2026):** 39 Regions · 123 Availability Zones · 400+ Edge Locations

---

## Key Facts to Remember

1. **AZs use private fibre — NOT the public internet.**
   AZs within a Region are connected via ultra-low latency private fibre. This is what makes Multi-AZ failover happen in milliseconds, not seconds.

2. **Edge Locations are for caching and DNS only — NOT compute.**
   You cannot launch EC2 instances or RDS databases in Edge Locations. They serve CloudFront (CDN) and Route 53 (DNS) only.

3. **Data does NOT automatically leave a Region.**
   Cross-region replication must be explicitly configured. By default, data in af-south-1 stays in af-south-1. Critical for NDPC compliance.

---

## Nigeria Context

### Latency from Lagos (tested via cloudpingtest.com)

| Region | Code | Latency |
|---|---|---|
| Africa (Cape Town) | af-south-1 | ~85ms ✅ Best |
| EU (Ireland) | eu-west-1 | ~140ms |
| EU (Frankfurt) | eu-central-1 | ~155ms |
| US East (Virginia) | us-east-1 | ~185ms |
| US West (Oregon) | us-west-2 | ~220ms |

### Edge Location
AWS has an Edge Location in **Lagos, Nigeria**.
This means Amazon CloudFront can cache content locally — reducing effective latency for Nigerian users to <20ms for cached assets regardless of origin region.

### Data Sovereignty
- Nigeria Data Protection Commission (NDPC) requires organisations to protect Nigerian personal data.
- Deploying in af-south-1 keeps data on the African continent by default.
- Always confirm that Cross-Region Replication is NOT configured unless you have a specific DR requirement with legal sign-off.

### af-south-1 Opt-In
Unlike most Regions, af-south-1 requires **explicit opt-in** in AWS Account Settings before you can deploy resources. Enable it under: Account → Regions → Enable af-south-1.

---

*Last updated: Week 1, Day 2 | To be expanded weekly through the course.*
