# Week 1 - Day 2: AWS Global Infrastructure

---

## 📌 Overview

AWS operates one of the largest and most reliable cloud infrastructures in the world — **39 Regions** and **123 Availability Zones** (as of 2026).

Understanding AWS Global Infrastructure is essential for building **resilient**, **performant**, and **compliant** cloud architectures.

> **Nigeria Context**: The nearest AWS Region is `af-south-1` (Cape Town, South Africa). Critical for latency, data sovereignty (NDPC), and disaster recovery.

---

## 🔑 Key Concepts

| Concept                | Definition |
|------------------------|----------|
| **AWS Region**         | A geographic area containing 2+ Availability Zones, physically isolated from other Regions. |
| **Availability Zone (AZ)** | One or more discrete data centers with independent power, cooling, and networking **within a Region**. |
| **Edge Location**      | Smaller facilities (400+ globally) for caching content (CloudFront) and DNS (Route 53). |
| **Local Zone**         | Extension of a Region that places compute and storage closer to specific metropolitan areas. |
| **Wavelength Zone**    | AWS infrastructure embedded in telecom networks for ultra-low latency 5G applications. |

---

## ⚙️ How It Works

1. Choose a **Region** → Defines where your data resides.
2. Each Region has multiple **AZs** connected by **low-latency private fiber** (not public internet).
3. **Edge Locations** cache content close to users for fast delivery.

**Key Rule**: You control **Region** and **AZ** selection. AWS manages the physical data center allocation inside an AZ.

---

## 🛡️ Architecture Summary

- **Regions** → Compliance, disaster recovery, data residency
- **Availability Zones** → High availability & fault tolerance (Multi-AZ)
- **Edge Locations** → Performance & low latency

---

## 🌍 Real-World Nigerian Scenario

Adaeze’s Nollywood streaming platform runs in `af-south-1` across two AZs (`af-south-1a` & `af-south-1b`).  
- Power failure in one AZ → Load balancer automatically fails over to the other AZ (Zero downtime).  
- CloudFront serves videos from Edge Locations near users (e.g., Lagos/Kano).

---

## ⚠️ Common Exam Traps

- AZs are connected via **private fiber**, **not** the public internet.
- You **cannot** launch EC2/RDS in Edge Locations.
- Data does **not** automatically replicate across Regions.

---

**Key Takeaway**: Design with **Multi-AZ** for availability and **Regions** for compliance.