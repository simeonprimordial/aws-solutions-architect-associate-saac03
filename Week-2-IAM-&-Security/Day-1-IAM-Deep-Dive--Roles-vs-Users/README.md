# AWS IAM Deep Dive — Week 2 Day 1

## Topic
IAM Deep Dive — Roles vs Users

This repository contains my notes, labs, and screenshots from Day 1 of Week 2 of my AWS Cloud journey. Week 2 focuses on IAM and Security — Domain 1 of the SAA-C03 exam, which accounts for **30% of the total marks**. Getting this week right is non-negotiable.

---

## What I Learned

### Core IAM Concepts
- Root Account — unlimited power, lock it with MFA, never use it daily
- IAM Users — permanent credentials for human operators
- IAM Groups — assign policies at group level, users inherit automatically
- IAM Policies — JSON documents defining Effect, Action, and Resource
- IAM Roles — temporary identities for AWS services; no passwords, no long-term keys
- Least Privilege — grant only the minimum access required, nothing more

### Users vs Roles — The Critical Distinction
- **IAM User** → a human needs console or CLI access → permanent credentials
- **IAM Role** → an AWS service (EC2, Lambda) needs access to resources → temporary STS credentials
- Never hardcode access keys in application code — attach a Role instead

### How IAM Roles Work Under the Hood
- Every Role has two policies: **Trust Policy** (who can assume it) + **Permissions Policy** (what it can do)
- AWS STS issues temporary credentials (access key + secret + session token) when a role is assumed
- Credentials expire automatically (default 1 hour for EC2, max 12 hours)
- EC2 instances retrieve credentials via the instance metadata service at `169.254.169.254`

---

## Hands-On Labs Completed
- Created three IAM Groups (Developers, Analysts, Managers) with appropriate policies
- Created three IAM Users (`dev-user`, `analyst-user`, `manager-user`) and assigned them to groups
- Tested permission boundaries by signing in as each user and attempting restricted actions
- Created an EC2 IAM Role (`EC2-S3-ReadOnly-Role`) with a trust policy for the EC2 service
- Ran the IAM Policy Simulator to test `s3:DeleteObject` for `analyst-user` (Bonus)
- Drew an IAM architecture diagram in Excalidraw

---

## AWS Services & Features Explored
- IAM — Users, Groups, Roles, Policies
- IAM Policy Simulator — `policysim.aws.amazon.com`
- AWS STS (Security Token Service) — issues temporary role credentials
- EC2 — used as the trusted service entity for the role

---

## Screenshots
All screenshots are stored in the `/screenshots` folder:
- `iam-groups-with-policies.png` — IAM console showing all 3 groups with policies attached
- `analyst-access-denied-s3.png` — Access Denied when analyst-user tried to create an S3 bucket
- `manager-access-denied-ec2.png` — Access Denied when manager-user tried to launch an EC2 instance
- `ec2-s3-readonly-role.png` — EC2-S3-ReadOnly-Role created in IAM Roles
- `iam-architecture-diagram.png` — Excalidraw architecture diagram of the full IAM structure
- `policy-simulator-s3-delete.png` — Policy Simulator showing analyst-user cannot perform s3:DeleteObject

---

## Challenges & Blockers
See `/notes/challenges.md` for issues I ran into and how I resolved them.

---

## Key Exam Traps to Remember
- Root account **cannot** be restricted by any IAM policy — not even by AWS Organizations SCPs
- IAM is **global** — users, roles, and policies are not Regional
- Never hardcode access keys into application code — the exam will offer this as a trap option
- Understand the difference between a **Trust Policy** and a **Permissions Policy** on a Role

---

## Goal
Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
