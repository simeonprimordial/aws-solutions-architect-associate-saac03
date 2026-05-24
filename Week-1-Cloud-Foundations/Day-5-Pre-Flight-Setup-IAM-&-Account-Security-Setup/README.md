# AWS Cloud Foundations — Week 1 Day 5

## Topic
IAM & Account Security Setup

This repository contains my notes, labs, screenshots, and portfolio artifacts from Day 5 of my AWS Cloud journey. Today's focus was locking down the AWS account properly — creating an IAM admin user, assigning groups and policies, enabling MFA everywhere, and ensuring the root account is never used for daily work.

---

## What I Learned

### The Core Principle: Never Use Root
The root account (the email used to create the AWS account) has unlimited, unrestricted access to every AWS service and billing function. It cannot be locked down with IAM policies — it bypasses all of them. This is exactly why it must be secured with MFA and then locked away. All daily work happens through IAM users.

### IAM Identities

| Identity | What It Is | Use Case |
|---|---|---|
| Root User | The account creation email — unrestricted power | Only for initial account setup and billing changes |
| IAM User | Individual identity with unique credentials | Humans and applications that need console or API access |
| IAM Group | Collection of users sharing the same permissions | Attach one policy to a group instead of 10 individual users |
| IAM Role | Temporary permissions assumed by a service or user | EC2 accessing S3, Lambda calling DynamoDB, cross-account access |

### IAM Policy Structure

Every IAM policy is a JSON document with three core fields:
- **Effect** — `Allow` or `Deny`
- **Action** — What AWS API calls are permitted (e.g. `s3:GetObject`, `ec2:StartInstances`)
- **Resource** — Which specific AWS resources the action applies to (ARN or `*` for all)

### Least Privilege Principle
Grant only the minimum permissions needed to perform the task. Never use `*` on all resources by default. Start with no permissions and add only what is required.

### MFA — Where It Must Be Enabled
- Root account — non-negotiable, Day 1
- All IAM users with console access
- Any IAM user with admin permissions

---

## Hands-On Labs Completed
- IAM dashboard explored — Security Recommendations checklist reviewed
- IAM Administrators group created with AdministratorAccess policy
- IAM admin user (`admin-yourname`) created with console access
- MFA enabled on IAM admin user
- Signed in as IAM user — confirmed root is no longer needed for daily work
- Root account security verified — no access keys, MFA enabled
- Bonus: Read-only IAM user created — confirmed S3 bucket creation is denied (Access Denied)

---

## AWS Services Covered
- **IAM** — Users, Groups, Roles, Policies, Security Recommendations
- **CloudWatch** — Billing Alarms (us-east-1 only)
- **AWS Budgets** — Cost alerts (more feature-rich than CloudWatch alarms)
- **AWS Cost Explorer** — Spend visualisation and forecasting

---

## Screenshots
All screenshots stored in `/screenshots`:
- `iam-administrators-group.png` — IAM dashboard showing Administrators group
- `iam-user-mfa-assigned.png` — IAM user listed with MFA assigned
- `iam-user-console-signin.png` — Signed in as IAM user (username shown top-right)
- `iam-security-recommendations-green.png` — Security Recommendations all resolved
- `bonus-access-denied-s3.png` — ReadOnly user denied creating S3 bucket

---

## Challenges & Blockers
See `/notes/challenges.md`

---

## Goal
Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
