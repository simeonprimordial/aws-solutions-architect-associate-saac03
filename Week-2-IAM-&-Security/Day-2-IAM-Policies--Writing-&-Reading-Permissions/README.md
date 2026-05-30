# IAM Policies — Writing & Reading Permissions — Week 2 Day 2

## Topic
IAM Policies — Writing & Reading Permissions

This repository contains my notes, labs, and screenshots from Day 2 of Week 2. Yesterday was about IAM identities (Users, Groups, Roles). Today goes one level deeper: how permissions are actually defined, read, and evaluated. You cannot work in AWS without being able to read a policy JSON and understand what it permits or blocks.

---

## What I Learned

### The Default-Deny Model
AWS denies every action by default. If no policy explicitly allows an action, it is blocked — automatically, instantly, at the API level. Access must be intentionally granted. This is the foundation of AWS security.

### Policy Structure — The Four Elements
- `Effect` — Allow or Deny
- `Action` — the specific AWS API call (e.g. `s3:GetObject`, `ec2:RunInstances`)
- `Resource` — the ARN of the target resource (`*` means all)
- `Condition` — optional restrictions (e.g. require MFA, restrict by IP address or Region)

### Types of Policies
- **AWS Managed Policies** — pre-built by AWS, regularly maintained
- **Customer Managed Policies** — custom policies you create and control, reusable across identities
- **Inline Policies** — embedded directly in a single user, group, or role; not reusable; avoid these

### Policy Evaluation — The Three Layers
AWS evaluates permissions by checking all policies in this order:
1. **Service Control Policies (SCPs)** — set by AWS Organizations; define maximum permissions for an account
2. **Identity-Based Policies** — attached to users, groups, roles (managed + inline)
3. **Resource-Based Policies** — attached to the resource itself (e.g. S3 bucket policies)

One explicit Deny at any layer overrides all Allow statements everywhere. No exceptions.

### The Custom Policy I Wrote
`S3-AppBucket-ReadOnly-NoDeletion` — two statements:
- **Allow:** `s3:GetObject` and `s3:ListBucket` scoped to buckets starting with `my-app-bucket-`
- **Deny:** `s3:DeleteObject` on all resources (`*`) — explicit deny, cannot be overridden

---

## Hands-On Labs Completed
- Read and analysed the `AmazonS3ReadOnlyAccess` managed policy JSON
- Wrote a custom IAM policy from scratch (typed, not copy-pasted)
- Attached the custom policy to `analyst-user`
- Tested three scenarios in the IAM Policy Simulator: allowed read, denied wrong bucket, denied delete
- Published the policy JSON to a GitHub Gist
- Added an IP address Condition block to the policy (Bonus)

---

## AWS Services & Features Used
- IAM — Policies, Policy Editor (JSON), Policy Simulator
- AWS Organizations — SCPs (conceptual review, not hands-on today)
- GitHub Gist — published policy as a portfolio artifact

---

## Screenshots
All screenshots are stored in the `/screenshots` folder:
- `managed-policy-json-s3readonly.png` — AmazonS3ReadOnlyAccess JSON in the IAM console
- `custom-policy-json-editor.png` — custom policy JSON in the IAM policy editor before saving
- `simulator-allowed-correct-bucket.png` — Policy Simulator: s3:GetObject on correct bucket → Allowed
- `simulator-denied-wrong-bucket.png` — Policy Simulator: s3:GetObject on wrong bucket → Denied
- `simulator-denied-delete.png` — Policy Simulator: s3:DeleteObject → Denied (explicit deny)
- `github-gist-policy-published.png` — Policy JSON published on GitHub Gist

---

## Policy Files
See `/policy-files/` for the full JSON policies written during this lab.

---

## Challenges & Blockers
See `/notes/challenges.md` for issues I ran into and how I resolved them.

---

## Key Exam Traps to Remember
- Explicit Deny **always** wins — no Allow anywhere can override it, including `Resource: *`
- SCPs set the **maximum** permissions for an account — even if an IAM policy allows an action, an SCP can block it
- `Resource: *` means ALL resources of that type — always scope to the minimum required ARN
- Two ARNs are needed for S3: one for the bucket itself and one for objects inside it (`/*`)
- Inline policies are **not reusable** and are not considered best practice

---

## Goal
Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
