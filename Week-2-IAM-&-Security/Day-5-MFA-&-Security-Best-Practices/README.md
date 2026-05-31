# MFA & Security Best Practices — Week 2 Day 5

## Topic
MFA & Security Best Practices — Hardening Your AWS Identity Layer

This repository contains my notes, labs, and screenshots from Day 5 of Week 2 — the final day of the IAM and Security week. Days 1–4 built the technical foundations: identities, policies, SCPs, and audit logging. Today pulls everything together into a security posture: how to harden an AWS account end to end, enforce MFA at scale, and build the runbook that makes it repeatable.

This is also the capstone of Week 2. The Security Runbook produced today is a direct portfolio artifact — a document that demonstrates hands-on security engineering knowledge to any technical hiring manager or CBN auditor.

---

## What I Learned

### Why Identity Is the Attack Surface
In the cloud, there is no physical perimeter. The only thing between an attacker and your AWS resources is a set of credentials. Over 80% of hacking-related breaches involve compromised credentials. MFA is the single most effective control — reducing account compromise risk by over 99% when properly enforced.

### Three MFA Types and When to Use Each
- **Virtual MFA (TOTP app)** — Google Authenticator, Authy; free; 30-second codes; good for standard users; vulnerable to real-time phishing
- **Hardware MFA token** — physical TOTP device (e.g. Gemalto); cannot be cloned from a screenshot; required for admins and finance teams by many financial regulators
- **FIDO2 / WebAuthn security key** — YubiKey, Google Titan; strongest option; phishing-resistant by cryptographic design; FIDO2 verifies the site origin and refuses fake sites

### The 8 IAM Security Best Practices
1. Lock down the root account immediately — MFA + no access keys + vault the password
2. Never create root access keys — they cannot be scoped or restricted by any policy
3. Enable MFA for all human IAM users — enforce via `aws:MultiFactorAuthPresent` Deny policy
4. Apply Principle of Least Privilege to every identity — start from zero, use Access Advisor to trim
5. Use IAM roles instead of long-term access keys — STS temporary credentials, no hardcoded keys
6. Rotate access keys every 90 days — audit with IAM Credential Report
7. Use Permission Boundaries to delegate safely — delegation without escalation
8. Use IAM Access Analyzer to detect unintended external access — continuous resource policy monitoring

### Three IAM Auditing Tools — Keep Them Separate
- **IAM Access Analyzer** — detects resources accessible from OUTSIDE your account (S3, roles, KMS, Lambda, SQS, Secrets Manager)
- **IAM Credential Report** — CSV snapshot of every user's MFA status, key age, last used, password age
- **IAM Access Advisor** — per-user service usage history; identifies unused permissions for removal

### The Layered Security Model
Layer 1 → Root hardening (total account takeover protection)
Layer 2 → MFA enforcement (stolen credential protection)
Layer 3 → Least privilege + Permission Boundaries (blast radius control)
Layer 4 → Continuous monitoring (CloudTrail + Access Analyzer + GuardDuty)

---

## Hands-On Labs Completed
- Enabled AWS Security Hub and ran the AWS Foundational Security Best Practices assessment
- Created and attached the `RequireMFA` enforcement policy to the Analysts group
- Downloaded and analysed the IAM Credential Report
- Ran IAM Access Advisor on own admin user — identified unused services
- Enabled IAM Access Analyzer — reviewed external access findings
- Built the full AWS Security Hardening Runbook (see `/runbook/`)
- Verified IAM Security recommendations dashboard — all green
- Ran AWS Trusted Advisor security checks (Bonus)

---

## AWS Services Used
- AWS Security Hub — Foundational Security Best Practices standard assessment
- IAM — MFA enforcement policy, Credential Report, Access Advisor
- IAM Access Analyzer — external resource access findings
- AWS Trusted Advisor — additional security recommendations (Bonus)

---

## Screenshots
All screenshots are stored in the `/screenshots` folder:
- `security-hub-findings-dashboard.png` — Security Hub initial assessment with PASSED/FAILED controls
- `require-mfa-policy-json.png` — RequireMFA policy JSON in IAM policy editor
- `iam-security-recommendations-green.png` — IAM Security recommendations dashboard showing all green
- `security-runbook-cover-page.png` — First page of the Security Hardening Runbook
- `trusted-advisor-security-findings.png` — AWS Trusted Advisor security category findings (Bonus)

---

## Policy Files
See `/policy-files/` for the MFA enforcement policy JSON.

## Runbook
See `/runbook/security-hardening-runbook.md` — the complete AWS Account Security Runbook built during this lab.

---

## Challenges & Blockers
See `/notes/challenges.md` for issues I ran into and how I resolved them.

---

## Key Exam Traps to Remember
- MFA on root is **not automatic** — and root access keys are **separate from MFA**; both must be addressed
- Permission Boundaries **do not grant permissions** — intersection rule applies; identity policy must also Allow
- **Access Analyzer ≠ Access Advisor** — Analyzer finds external access; Advisor shows usage history
- The IAM Credential Report is a **detective** control — it detects but does not enforce anything
- FIDO2 is **phishing-resistant**; TOTP (virtual MFA) is not — TOTP codes can be intercepted in real time

---

## Goal
Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
