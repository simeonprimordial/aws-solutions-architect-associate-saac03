# SCPs & AWS Organizations — Exam Prep — Week 2 Day 3

---

## SAA-C03 Context

SCPs and AWS Organizations appear in Domain 1: Design Secure Architectures. Questions typically present a multi-account governance scenario and ask which solution correctly enforces a restriction centrally. The key skill is knowing when SCPs are the right answer, when IAM policies are the right answer, and — critically — what each one cannot do.

---

## The Five Exam Traps for This Topic

These five appear regularly. Know them cold.

**Trap 1 — Management Account exemption.**
No SCP ever restricts the Management Account. If a question describes an SCP attached to the Root and asks what it affects, the Management Account is always excluded from the answer. This is unconditional.

**Trap 2 — SCPs do not grant permissions.**
An SCP with `"Effect": "Allow"` does not grant access. It only declares that IAM is permitted to grant that access. A principal still needs an IAM Allow to actually perform the action. The exam will offer "attach an SCP to grant access" as an answer option — it is always wrong.

**Trap 3 — SCPs restrict root users of member accounts.**
IAM policies cannot restrict root users. SCPs can. If a question asks how to prevent the root user of a member account from performing an action, the answer is always an SCP, never an IAM policy.

**Trap 4 — FullAWSAccess is the default.**
Enabling SCPs in Organizations does not restrict anything automatically. The FullAWSAccess SCP is attached by default, keeping everything permissive. Restrictions must be deliberately added.

**Trap 5 — An unattached SCP does nothing.**
A correctly written SCP in the policy library that has never been attached to a root, OU, or account has zero effect. If a question describes this scenario and asks why the SCP is not working, the answer is: it was never attached.

---

## Practice Question — Week 2 Day 3

**Scenario:** A company has multiple AWS accounts managed through AWS Organizations. The security team wants to prevent all developers in the Development OU from creating EC2 instances in any AWS region except `eu-west-1` and `af-south-1`. The solution must be centralised and must not require changes to IAM policies in individual accounts. Which solution meets these requirements?

**A.** Create an IAM permission boundary in each developer account that includes a Deny for `ec2:RunInstances` in non-approved regions, attached to all developer IAM users.

**B.** Attach an SCP to the Development OU with a Deny for `ec2:RunInstances` using the `aws:RequestedRegion` condition key, excluding `eu-west-1` and `af-south-1`.

**C.** Attach an SCP to the Management Account that denies `ec2:RunInstances` in all non-approved regions.

**D.** Create an AWS Config rule in each development account that automatically terminates EC2 instances launched in non-approved regions via an SSM automation remediation.

**Answer: B**

**Why A is wrong:** Permission boundaries are per-account and must be manually applied to individual users. Not centralised. Also, permission boundaries cannot be deployed centrally across accounts from Organizations.

**Why C is wrong:** The Management Account is never affected by SCPs. Attaching an SCP to the Management Account restricts only principals in the Management Account — it does not propagate to OUs or member accounts.

**Why D is wrong:** AWS Config rules are detective controls — they detect and remediate after the fact. EC2 instances are created before the Config rule fires and the SSM remediation terminates them. This is not a preventive control. It also requires per-account setup.

**Why B is correct:** Attaching an SCP to the Development OU is centralised, automatically inherited by all accounts in the OU, prevents the action before it happens, and cannot be overridden by any IAM policy within the target accounts.

---

## Scenario Decision Table — When to Use What

| Requirement | Correct Tool |
|---|---|
| Prevent an entire OU from using a specific service | SCP on the OU |
| Prevent a single IAM user from performing an action | IAM Deny policy on the user |
| Restrict the root user of a member account | SCP (IAM policies cannot do this) |
| Enforce data residency across all accounts in an organisation | SCP with `aws:RequestedRegion` condition at Root level |
| Grant a developer access to EC2 | IAM policy (SCPs cannot grant — only restrict) |
| Prevent an account from leaving the organisation | SCP with `organizations:LeaveOrganization` Deny at Root |
| Protect CloudTrail from being disabled across all accounts | SCP with CloudTrail Deny actions at Root level |
| Prevent dev accounts from purchasing Reserved Instances | SCP on Development OU |
| Require MFA for all API calls in member accounts | SCP with `aws:MultiFactorAuthPresent` false condition Deny |

---

## Quick-Recall Test

Close your notes and answer these without looking:

1. Can an SCP restrict the Management Account?
2. Can an SCP restrict the root user of a member account?
3. What does `FullAWSAccess` do and when is it applied?
4. What is the effective permission formula for a principal in a member account?
5. You write a perfect SCP and save it. A developer in the Dev OU still performs the forbidden action. What is the most likely reason?
6. A developer has `AdministratorAccess` IAM policy. An SCP on their OU blocks `s3:DeleteBucket`. Can they delete an S3 bucket?
7. What is the difference between the Deny List and Allow List SCP strategies, and when would you use each?

Answers:
1. No — never, by design.
2. Yes — unlike IAM policies, SCPs restrict member account root users.
3. Allows all actions on all services. Applied automatically to every root, OU, and account when SCPs are first enabled.
4. Effective permission = IAM Policy Allow AND SCP Allow. Both must permit the action.
5. The SCP was never attached to the OU or relevant account.
6. No — the SCP Deny overrides the IAM Allow regardless of how broad the IAM policy is.
7. Deny List: keep FullAWSAccess, add Denies for forbidden actions. Use when starting out or when forbidden actions are well-defined. Allow List: remove FullAWSAccess, add Allows only for approved services. Use in high-security regulated environments (financial, healthcare, defence).
