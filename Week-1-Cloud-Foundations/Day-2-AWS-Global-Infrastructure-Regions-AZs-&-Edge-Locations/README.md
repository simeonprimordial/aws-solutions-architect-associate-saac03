# AWS Cloud Foundations — Week 1 Day 2

## Topic
AWS Global Infrastructure

This repository contains my notes, labs, screenshots, and portfolio artifacts from Day 2 of my AWS Cloud journey. Today's focus was understanding how AWS organises its physical infrastructure globally — and why that matters for building resilient systems.

---

## What I Learned

### AWS Global Infrastructure Components
- **AWS Region** — A geographic area containing 2+ Availability Zones, physically isolated from other Regions
- **Availability Zone (AZ)** — One or more discrete data centres with independent power, cooling, and networking within a Region
- **Edge Location** — Smaller AWS facility for caching content close to end users (CloudFront CDN, Route 53)
- **Local Zone** — Extension of an AWS Region placing compute/storage closer to specific metro areas
- **Wavelength Zone** — AWS infrastructure embedded in telecom networks for ultra-low latency 5G apps

### Key Infrastructure Facts (as of 2026)
- 39 geographic Regions globally
- 123 Availability Zones
- 400+ Edge Locations (Points of Presence)

### Why It Matters for Nigeria
- Closest AWS Region: **af-south-1 (Cape Town)**
- Edge Locations exist in Lagos — reducing latency for Nigerian users via CloudFront
- Data sovereignty (NDPC compliance) requires understanding cross-region replication rules

---

## Hands-On Labs Completed
- AWS Global Infrastructure Map Exploration
- Console Region Switching & AZ Verification
- AWS Region Cheat Sheet Built and Published (GitHub Gist)
- Latency Test: Lagos → af-south-1 vs Lagos → us-east-1

---

## AWS Services Explored
- CloudFront — Content Delivery Network using Edge Locations
- Route 53 — DNS service that uses Edge Locations
- EC2 — Verified AZ counts across regions in the console

---

## Screenshots
All screenshots stored in `/screenshots`:
- `infrastructure-map-africa.png` — AWS global map highlighting af-south-1
- `console-region-selector.png` — All available regions in the console dropdown
- `ec2-az-us-east-1.png` — Availability Zones listed for us-east-1
- `region-cheat-sheet.png` — Completed cheat sheet table
- `github-gist-published.png` — Published GitHub Gist URL

---

## Portfolio Artifact
My published AWS Regions cheat sheet (GitHub Gist) is linked in:
`/portfolio-post/linkedin-post.md`

---

## Challenges & Blockers
See `/notes/challenges.md` for issues encountered and how I solved them.

---

## Goal
Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
