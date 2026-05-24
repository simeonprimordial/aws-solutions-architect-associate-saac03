# AWS Cloud Foundations — Week 1 Day 3

## Topic
The Shared Responsibility Model

This repository contains my notes, labs, screenshots, and portfolio artifacts from Day 3 of my AWS Cloud journey. Today's focus was understanding exactly where AWS's security responsibility ends and mine begins — and why getting this wrong causes real-world breaches.

---

## What I Learned

### The Core Principle
The Shared Responsibility Model is a contractual division of security ownership between AWS and the customer.

- **Security OF the Cloud** → AWS's responsibility: physical facilities, hardware, virtualisation layer, global network
- **Security IN the Cloud** → Customer's responsibility: OS patches, application security, data encryption, IAM and access controls

### Key Concepts
- **Managed Services Shift** — The more managed the service, the more AWS takes over. Lambda removes far more customer responsibility than EC2.
- **Shared Controls** — Patch management, configuration management, and security awareness training are shared between both parties.
- **Inherited Controls** — Customers automatically inherit AWS's physical and environmental controls. No need to audit the data centre yourself.

### The Responsibility Shifts by Service Type

| Service | Customer Manages | AWS Manages |
|---|---|---|
| EC2 (IaaS) | OS, patching, app, data, firewall | Hardware, hypervisor, physical facility |
| RDS (Managed DB) | Data, DB user permissions, network rules, encryption config | DB engine patches, OS, underlying hardware |
| Lambda (Serverless) | Function code, data, IAM permissions | Runtime, OS, hardware, scaling |
| S3 (Object Storage) | Bucket policies, encryption, access controls | Physical storage, durability, hardware |

---

## Hands-On Labs Completed
- Read and summarised AWS Shared Responsibility Model page in own words
- Categorised 12 real-world items as AWS vs Customer responsibility (with answers verified)
- Built a visual responsibility diagram
- Wrote a 3-sentence plain-English explanation for a non-technical Nigerian business owner
- Researched the 2019 Capital One breach (bonus challenge)

---

## AWS Services Referenced
- EC2 — IaaS example (customer manages OS and above)
- RDS — Managed DB (AWS patches engine; customer manages data and access)
- S3 — Object storage (customer manages bucket policies and encryption)
- Lambda — Serverless (AWS manages everything below function code)
- IAM — Identity and access management (always customer responsibility)

---

## Screenshots
All screenshots stored in `/screenshots`:
- `responsibility-diagram.png` — Completed 3-column responsibility table
- `12-item-categorised-list.png` — All 12 items correctly categorised
- `3-sentence-explanation.png` — Plain-English explanation document

---

## Portfolio Artifact
LinkedIn post is in `/portfolio-post/linkedin-post.md`

---

## Challenges & Blockers
See `/notes/challenges.md`

---

## Goal
Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
