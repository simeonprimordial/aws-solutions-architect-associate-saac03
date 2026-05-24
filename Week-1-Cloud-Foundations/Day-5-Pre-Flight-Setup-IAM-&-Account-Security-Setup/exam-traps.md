# Exam Traps & Practice Questions — Day 5

---

## The 4 Critical Exam Traps

### Trap 1 — Billing Data Lives ONLY in us-east-1

**The trap:** CloudWatch Billing Alarms need to be created in a specific Region. Students create them in their closest Region (e.g. af-south-1) and wonder why no data appears.

**The rule:** AWS consolidates all billing and cost data in **us-east-1 (N. Virginia)** only. If you create a CloudWatch Billing Alarm in any other Region, the billing metric won't be available — the alarm will never trigger because it has no data to evaluate.

**Action:** Always switch to us-east-1 before creating billing-related CloudWatch alarms.

**Note:** AWS Budgets is not affected by this — it works globally regardless of which Region your console is set to.

---

### Trap 2 — Root Account Should NEVER Have Access Keys

**The trap:** Exam questions will describe a scenario where someone creates root access keys for programmatic access. This is always the wrong answer.

**The rule:**
- Root access keys grant full, unrestricted programmatic access to your entire account
- If root access keys are compromised, an attacker has god-mode API access — and can even lock you out
- If root access keys exist in your account: delete them immediately
- For programmatic access, create IAM users or use IAM roles — never root keys

**How it appears on the exam:** "A developer needs programmatic access to AWS. Which credentials should they use?" → IAM user access keys, never root access keys.

---

### Trap 3 — Policies Attached to Groups vs Directly to Users

**The trap:** Exam questions sometimes describe attaching policies directly to each individual user. This is technically possible but is always the wrong architectural choice.

**The rule:**
- Attaching policies to IAM groups is the correct and scalable approach
- Directly attaching policies to users creates management chaos at scale — changing permissions means updating every individual user
- AWS's own best practices explicitly recommend groups over direct user policies

**How it appears on the exam:** Any question about managing permissions for a team of developers → the answer involves creating a group, attaching the policy to the group, and adding users to the group. Direct-to-user policy attachment is always wrong in a multi-user scenario.

---

### Trap 4 — IAM Roles for EC2, Not Access Keys Embedded in Code

**The trap:** Developers sometimes hardcode AWS access keys into application code or EC2 instance configuration files to allow the app to access other AWS services (e.g. S3). This is a critical security anti-pattern.

**The rule:**
- EC2 instances authenticate to other AWS services via **IAM Roles** — not access keys
- IAM Roles issue temporary credentials automatically, rotated by AWS
- Hardcoded access keys in code can be accidentally committed to GitHub, leaked in logs, or exposed in error messages
- AWS Config and Security Hub can detect hardcoded credentials — and attackers scan GitHub for them too

**How it appears on the exam:** "An EC2 application needs to read from S3. What is the MOST secure way to provide access?" → Attach an IAM Role to the EC2 instance with the required S3 permissions. "Store access keys in a config file on the instance" is always wrong.

---

## SAA-C03 Practice Question — Day 5

**Question:**
A solutions architect at a Lagos startup is setting up a new AWS account for a fintech application. A junior developer suggests using the root account credentials to run daily deployments because it has full access to everything. Which response correctly identifies the risk AND provides the right solution?

**A.** The root account is appropriate for daily tasks as long as MFA is enabled — MFA makes it secure enough for regular use.

**B.** The root account should never be used for daily tasks. Instead, create an IAM admin user, assign it AdministratorAccess via an IAM group, enable MFA on that user, and use it for all daily work. The root account should have MFA enabled, no access keys, and be used only for account-level tasks that explicitly require root.

**C.** Create individual IAM users for each developer with AdministratorAccess attached directly to each user — this is more granular than groups.

**D.** The root account is only a risk if access keys are created. As long as you only use the console, daily root usage is acceptable.

---

**Answer: B**

**Why B is correct:**
This is a precise, complete answer. It identifies the root account risk, provides the correct mitigation (IAM admin user with MFA), specifies the right architecture (group-based policy attachment), and defines the exact scope of legitimate root usage. This is the AWS-documented best practice, word for word.

**Why A is wrong:**
MFA protects against password theft but does not make root appropriate for daily tasks. The root account bypasses all IAM policies and has capabilities that no daily task requires (closing the account, changing root email, modifying billing details). Using root daily means any session compromise is catastrophic — no IAM boundary contains the damage.

**Why C is wrong:**
Attaching AdministratorAccess directly to individual users is architecturally incorrect. Use a group. More importantly, every developer having AdministratorAccess violates least privilege — developers should have developer-level permissions, not admin-level.

**Why D is wrong:**
Daily root console usage is dangerous regardless of access keys. The session can be hijacked, the password can be phished, and the scope of what root can do from the console (closing the account, billing changes, IAM lockout of all other users) makes any root compromise catastrophic.

---

## Quick Recall Quiz

Cover the answers and test yourself:

| Question | Answer |
|---|---|
| What does IAM stand for? | Identity and Access Management |
| What are the four IAM identity types? | Root user, IAM Users, IAM Groups, IAM Roles |
| Should you use root for daily tasks? | Never — use an IAM admin user |
| What are the three required fields in an IAM policy statement? | Effect, Action, Resource |
| What does Effect: Deny do vs Effect: Allow? | Explicit Deny always overrides Allow |
| What is the Least Privilege Principle? | Grant only the minimum permissions needed — never use * by default |
| Which identity type uses temporary credentials? | IAM Roles |
| How should an EC2 app access S3 securely? | Attach an IAM Role to the EC2 instance — never embed access keys |
| Should root have access keys? | Never — delete them if they exist |
| Where must CloudWatch Billing Alarms be created? | us-east-1 (N. Virginia) only |
| Which is more feature-rich: AWS Budgets or CloudWatch Billing Alarms? | AWS Budgets |
| Are IAM users charged? | No — always free |
| Which AWS services are global (not regional)? | IAM, Billing, Route 53, CloudFront |
| What is an IAM Group? | A collection of users sharing the same permissions via an attached policy |
| What is a Managed Policy? | A reusable JSON policy document — either AWS-created or customer-created |
