# MFA & Security Best Practices — Exam Prep — Week 2 Day 5

---

## SAA-C03 Context

This is also a Week 2 capstone — everything from Days 1–5 converges here. Domain 2 (Design Secure Architectures) is 30% of the exam. The topics from this week — IAM, SCPs, CloudTrail, MFA, Permission Boundaries, Access Analyzer — are all explicitly in the task statements for Domain 2. The exam tests the combination, not just individual pieces.

---

## The Five Exam Traps — This Topic

**Trap 1 — Root MFA and root access keys are two separate controls.**
MFA on root protects console sign-in. Root access keys are a completely separate credential type — they authenticate API calls without going through the MFA flow. Having root MFA enabled does not protect against leaked root access keys. Both controls must be applied: MFA enabled AND access keys deleted.

**Trap 2 — Permission Boundaries do not grant permissions.**
The exam frequently offers "attach a Permission Boundary that allows X" as a way to grant access. It is always wrong. A Permission Boundary defines the maximum ceiling — it does not grant anything. The identity policy must also explicitly Allow the action. Effective permission = intersection of boundary AND identity policy.

**Trap 3 — Access Analyzer ≠ Access Advisor — know both.**
Access Analyzer detects resources accessible from outside your account (external access). Access Advisor shows which AWS services a user has actually accessed (usage history for least privilege). These are tested interchangeably in distractors. If the question is about unused permissions → Access Advisor. If it is about public S3 bucket detection → Access Analyzer.

**Trap 4 — The Credential Report is detective, not preventive.**
The report shows the current state. It does not enforce anything. To enforce MFA: IAM Deny policy. To auto-disable stale keys: Lambda function. To remediate inactive accounts: automation or manual process. The report is the observation tool.

**Trap 5 — FIDO2 is phishing-resistant; TOTP is not.**
The strongest MFA option on the exam is always FIDO2/WebAuthn. TOTP codes (6-digit time-based codes from an app) can be intercepted in a real-time phishing attack — an attacker can relay the code to AWS before it expires. FIDO2 keys cryptographically verify the origin domain and cannot authenticate to a fake site.

---

## Practice Question — Week 2 Day 5

**Scenario:** A security engineer at a Lagos fintech is setting up AWS for a team of 20 developers. The requirements: (1) all developers must use MFA, (2) a junior developer can create IAM roles for Lambda functions but cannot create roles with more permissions than their own, and (3) the security team can identify which developers have not used their S3 permissions in 90 days. Which THREE configurations meet all requirements?

**A.** Attach an IAM policy with `Effect:Deny`, `Condition: aws:MultiFactorAuthPresent=false`, `NotAction: [iam:CreateVirtualMFADevice, iam:EnableMFADevice]` to the Developers IAM group.

**B.** Attach an IAM Permission Boundary to the junior developer's IAM user that limits maximum permissions to what the junior developer currently has. Require the boundary to be attached to any role they create.

**C.** Enable MFA on the root account and require all developers to use root credentials with MFA for all operations.

**D.** Use IAM Access Advisor on each developer's user or role to review last-accessed service dates and identify S3 permissions not used in 90 days.

**E.** Run the IAM Credential Report monthly, filter for `mfa_active=false`, and use IAM Access Analyzer to flag S3 buckets accessible from outside the account.

**Answers: A, B, D**

**Why A is correct:** Standard MFA enforcement pattern. The Deny policy with `NotAction` exceptions enforces MFA across the whole group. New users can still enrol MFA because the setup actions are excluded.

**Why B is correct:** Permission Boundary implements delegation without escalation. The junior developer can create Lambda execution roles, but none can exceed the boundary ceiling.

**Why C is wrong:** Sharing root credentials violates every IAM best practice. Root must never be used for daily operations, and credentials must never be shared.

**Why D is correct:** Access Advisor shows per-user service usage history. Filtering for S3 last access > 90 days identifies unused S3 permissions for removal — the correct tool for this requirement.

**Why E is wrong:** The Credential Report detects users without MFA but does not enforce it. Access Analyzer detects external access to S3 buckets — not whether developers have unused internal S3 permissions. E addresses different requirements than what was asked.

---

## Week 2 Full IAM & Security Summary

Everything from this week in one table:

| Topic | Key Concept | Exam Signal |
|---|---|---|
| IAM Users vs Roles | Users = permanent credentials; Roles = temporary STS credentials | EC2/Lambda needing AWS access → always a Role |
| IAM Policies | Explicit Deny overrides all Allow | Any question with "unless denied" → Deny wins |
| SCPs | Ceiling for accounts, not grants; Management Account exempt | Cross-account governance → SCP |
| CloudTrail | Records WHO/WHAT/WHEN/WHERE for every API call | Audit trail, forensics, compliance → CloudTrail |
| Data Events | Off by default; required for S3 object-level tracking | "Who downloaded this S3 file" → Data Events |
| MFA Types | FIDO2 > Hardware token > Virtual TOTP | Strongest MFA → FIDO2; phishing-resistant → FIDO2 |
| MFA Enforcement | IAM Deny + `aws:MultiFactorAuthPresent=false` | "Require MFA" → DenyWithoutMFA policy |
| Permission Boundaries | Ceiling for delegated role creation; no grants | "Prevent privilege escalation" → Permission Boundary |
| Access Analyzer | External resource access detection | "Public S3 bucket" finding → Access Analyzer |
| Credential Report | Detective CSV: MFA status, key age, last used | "Which users lack MFA" → Credential Report |
| Access Advisor | Per-user service usage history | "Remove unused permissions" → Access Advisor |

---

## Quick-Recall Test — Week 2 Capstone

Answer these without looking at notes:

1. What is the difference between an IAM User and an IAM Role in one sentence?
2. What are the four elements of an IAM policy JSON statement?
3. Can an SCP grant permissions? Can it restrict the root user of a member account?
4. What three CloudTrail configurations must be in place before an incident for an investigation to succeed?
5. What is the delivery delay for CloudTrail logs to S3?
6. What is the condition key used in the MFA enforcement policy?
7. Why must `BoolIfExists` be used instead of `Bool` for the MFA condition?
8. What is the difference between IAM Access Analyzer and IAM Access Advisor?
9. A Permission Boundary allows `s3:*`. The identity policy has no S3 permissions. Can the user access S3?
10. Which MFA type is phishing-resistant and why?

Answers:
1. Users have permanent credentials (password + access keys); Roles have no permanent credentials and issue temporary STS credentials when assumed.
2. Effect (Allow/Deny), Action (API call), Resource (ARN or *), Condition (optional restrictions).
3. No, SCPs cannot grant permissions. Yes, SCPs do restrict the root user of member accounts (unlike IAM policies which cannot).
4. S3 Data Events enabled (for object-level access), multi-region Trail (for all-region coverage), log file integrity validation (for forensic admissibility).
5. Approximately 15 minutes.
6. `aws:MultiFactorAuthPresent` set to `"false"` using `BoolIfExists`.
7. For CLI calls using access keys, `aws:MultiFactorAuthPresent` is absent from the request context. `Bool` fails when the key is absent. `BoolIfExists` treats absence as "condition not triggered" — the Deny only fires when the key is present and explicitly false.
8. Access Analyzer detects resources accessible from outside your AWS account (external access detection). Access Advisor shows which services a user has actually accessed — used to identify unused permissions for removal.
9. No. Permission Boundaries do not grant permissions. Effective permission = intersection of boundary AND identity policy. Both must allow S3 — if the identity policy has no S3 Allow, the user cannot access S3.
10. FIDO2/WebAuthn. It uses public-key cryptography with a key pair bound to the specific website domain. A fake phishing site has a different domain — the key refuses to authenticate to it. TOTP codes can be intercepted and replayed in 30 seconds on a fake site.
