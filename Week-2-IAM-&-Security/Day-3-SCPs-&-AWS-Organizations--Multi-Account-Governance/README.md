# SCPs & AWS Organizations — Week 2 Day 3

## Topic
SCPs & AWS Organizations — Multi-Account Governance

This repository contains my notes, labs, and screenshots from Day 3 of Week 2. Days 1 and 2 covered IAM inside a single account. Today zooms out to the multi-account level — how large organisations structure AWS accounts and enforce governance across all of them simultaneously. This is where IAM knowledge from the previous two days connects to real enterprise architecture.

---

## What I Learned

### Why Multi-Account Architecture Exists
A single AWS account has hard limits — on IAM policies, resource quotas, and blast radius. Large organisations separate workloads into multiple accounts so that a breach, misconfiguration, or runaway cost in one account cannot affect the others. AWS Organizations is the service that manages all of these accounts under one roof.

### AWS Organizations — Core Structure
- **Management Account** — creates and controls the organisation; never restricted by SCPs
- **Organisational Units (OUs)** — logical folders that group accounts (e.g. Production OU, Dev OU, Security OU)
- **Member Accounts** — individual AWS accounts placed inside OUs; inherit all SCPs attached to their parent OU
- **Root** — the top-level container; SCPs attached here apply to every account in the entire organisation

### Service Control Policies (SCPs)
- JSON policy documents that set the **maximum permissions ceiling** for an account or OU
- SCPs **never grant permissions** — they only restrict. Actual access still requires an IAM Allow
- Effective permission = IAM Policy **AND** SCP — both must allow for access to succeed
- One explicit Deny in an SCP overrides every IAM Allow in the account, including `AdministratorAccess`
- SCPs **do** restrict the root user of member accounts (unlike IAM policies, which cannot)

### The Two SCP Strategies
- **Deny List** (recommended default) — keep `FullAWSAccess` attached, add explicit Deny statements on top for what is forbidden
- **Allow List** (high-security) — remove `FullAWSAccess`, replace with explicit Allows for approved services only; anything not listed is implicitly denied

### SCPs Written Today
- `DenyDisableCloudTrail` — prevents anyone from stopping, deleting, or modifying audit logs
- `DenyLeaveOrganization` — prevents any member account from removing itself from the organisation
- `DenyNonApprovedRegions` — data residency control; blocks all resource creation outside `af-south-1` and `eu-west-1` (Bonus)

---

## Hands-On Labs Completed
- Read the AWS Organizations documentation (overview + SCP section)
- Designed the OluPay Ltd multi-account structure in Excalidraw (Root → 3 OUs → 6 accounts)
- Wrote `DenyDisableCloudTrail` SCP JSON from scratch with explanation comments
- Wrote `DenyLeaveOrganization` SCP JSON
- Built the SCP vs IAM Policy comparison table
- Wrote the `DenyNonApprovedRegions` SCP (Bonus)

---

## AWS Services & Concepts Covered
- AWS Organizations — account management, OU hierarchy, consolidated billing
- Service Control Policies (SCPs) — Deny List strategy, Allow List strategy, inheritance model
- AWS CloudTrail — the audit logging service that SCPs protect
- AWS Control Tower — mentioned as the managed layer on top of Organizations

---

## Screenshots
All screenshots are stored in the `/screenshots` folder:
- `olupay-organizations-diagram.png` — Excalidraw diagram of OluPay Ltd multi-account structure with OUs, accounts, and SCP labels
- `cloudtrail-protection-scp-json.png` — CloudTrail protection SCP in the text editor
- `scp-vs-iam-comparison-table.png` — SCP vs IAM Policy comparison table

---

## Policy Files
See `/policy-files/` for all SCP JSON documents written during this lab.

---

## Challenges & Blockers
See `/notes/challenges.md` for issues I ran into and how I resolved them.

---

## Key Exam Traps to Remember
- The Management Account is **never** restricted by SCPs — not even by a Root-level SCP
- SCPs **do not grant permissions** — attaching an SCP that "allows" something does not give access; IAM still needs to grant it
- SCPs **do** restrict the root user of member accounts — more powerful than IAM in that respect
- `FullAWSAccess` is the default SCP — enabling Organizations does **not** automatically restrict anything
- Creating an SCP does nothing — it must be **attached** to a root, OU, or account to take effect

---

## Goal
Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
