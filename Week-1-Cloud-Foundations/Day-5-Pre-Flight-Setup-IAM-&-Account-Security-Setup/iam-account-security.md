# IAM & Account Security — Day 5 Notes

---

## Overview — Why IAM Is the Most Important Service

IAM (Identity and Access Management) is the foundation of every AWS security architecture. Before you write a single line of application code or launch a single resource, you need to answer: **who can do what, to which resources, under what conditions?**

IAM answers all four parts of that question.

**Why it is heavily tested on SAA-C03:**
- IAM misconfiguration is the leading cause of AWS account compromises
- The Capital One breach (Day 3) was ultimately an IAM failure — overly permissive roles
- Every AWS service integrates with IAM — you cannot build securely on AWS without understanding it

---

## The Root Account — Lock It Away

The root account is the email address used to create the AWS account. It has:
- Unlimited access to every AWS service
- Access to billing and account closure
- The ability to cancel the account entirely
- **Cannot be restricted by any IAM policy** — it bypasses them all

### Root Account Rules — Non-Negotiable
1. Enable MFA on root immediately after account creation (Day 1 — already done)
2. Never use root for daily tasks — not even creating resources
3. Never create access keys for root — if they exist, delete them now
4. Only use root for: initial account setup, billing changes that require root, closing the account

### What Should Have Zero Root Activity After Setup
- Creating EC2 instances → use IAM admin user
- Creating S3 buckets → use IAM admin user
- Creating IAM users → use IAM admin user
- Running labs → use IAM admin user

> **The mental model:** Root is the master key to your building. You use it once to get in, then you hang it on a hook, enable the alarm, and use your personal key card for everything else.

---

## IAM Identities

### IAM Users
Individual accounts for people or applications. Each has unique credentials (username + password for console, or access key + secret for API/CLI).

**Best practices:**
- Create one IAM user per person — never share credentials
- Use a naming convention: `admin-firstname`, `dev-firstname`, `readonly-firstname`
- Assign permissions through groups, not directly to users
- Enable MFA for any user with console access

**Free Tier note:** IAM users are always free. No cost for creating any number of users, groups, or policies.

---

### IAM Groups
Collections of IAM users. Attach a policy to the group — every user in the group inherits those permissions automatically.

**Why groups over direct user policies:**
- Adding a new developer gets all "Developers" permissions automatically
- Removing someone from a group revokes all group permissions instantly
- Policy updates apply to all group members at once — one change, not 20

**Common group patterns:**

| Group Name | Policy Attached | Who Belongs |
|---|---|---|
| Administrators | AdministratorAccess | Cloud architects, account owners |
| Developers | Custom developer policy | Engineers who deploy applications |
| ReadOnly | ReadOnlyAccess | Auditors, junior staff, monitoring tools |
| Billing | Billing + Cost Explorer access | Finance team |

---

### IAM Roles
Temporary permissions that can be **assumed** by AWS services, applications, or users — not permanently assigned like user credentials.

**Key distinction from IAM users:**
- IAM Users have permanent, long-lived credentials (username/password, access keys)
- IAM Roles have temporary, time-limited credentials issued automatically

**When roles are used:**

| Scenario | Role Needed |
|---|---|
| EC2 instance needs to read from S3 | Attach an IAM role with S3 read permissions to the EC2 instance |
| Lambda function needs to write to DynamoDB | Attach a role with DynamoDB write permissions to the Lambda function |
| Developer in Account A needs to access Account B | Cross-account role with trust policy |
| Employee needs temporary elevated access | Assume a role for the duration of the task |

> **Critical exam concept:** EC2 instances should NEVER have access keys embedded in their code or configuration. Always use IAM Roles — the temporary credentials are automatically rotated by AWS and never stored in plaintext.

---

### IAM Policies
JSON documents that define what actions are allowed or denied on which AWS resources.

**Policy JSON structure:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

**The three required fields in every statement:**
- `Effect` — `Allow` or `Deny`. Explicit Deny always overrides Allow.
- `Action` — The AWS API call being controlled. Format: `service:APIcall` (e.g. `ec2:StartInstances`, `s3:GetObject`). Use `*` for all actions in a service.
- `Resource` — The specific ARN (Amazon Resource Name) the statement applies to. Use `*` for all resources.

**Policy types:**

| Type | Created By | Reusable? | Example |
|---|---|---|---|
| AWS Managed Policy | AWS | Yes — attached to many identities | `AmazonS3ReadOnlyAccess`, `AdministratorAccess` |
| Customer Managed Policy | You | Yes — you create, you reuse | Custom developer policy for your org |
| Inline Policy | Embedded in one identity | No — lives only on that user/role | One-off permissions for a specific resource |

---

## The Least Privilege Principle

Grant only the minimum permissions needed to perform a task. This is the most important security principle in AWS.

**In practice:**
- Start from zero permissions — add only what is required
- Never use `*` (all resources) unless the task genuinely requires access to all resources
- Regularly review and remove permissions that are no longer needed
- Use AWS IAM Access Analyzer to identify overly permissive policies

**Example of violation vs compliance:**

```
VIOLATION — too permissive:
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
This grants every action on every resource — equivalent to root.

COMPLIANT — least privilege:
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:ListBucket"],
  "Resource": "arn:aws:s3:::customer-uploads-bucket/*"
}
This grants only S3 read access on one specific bucket.
```

---

## MFA — Multi-Factor Authentication

MFA adds a second verification step beyond username and password. Even if a password is compromised, the attacker cannot sign in without the physical MFA device.

**Where MFA must be enabled:**
1. Root account — non-negotiable
2. All IAM users with AWS Console access
3. Any IAM user with admin or elevated permissions

**MFA device types:**
- **Virtual MFA** (Google Authenticator, Authy) — most common for individuals
- **Hardware TOTP** (dedicated hardware token) — used in high-security enterprise environments
- **FIDO Security Keys** (YubiKey) — phishing-resistant hardware key
- **SMS MFA** — available but not recommended (SIM-swap attacks)

---

## Billing Alerts — Console-Level Safety

### CloudWatch Billing Alarms
A specific CloudWatch alarm that triggers when estimated monthly charges exceed a threshold.

**Critical requirement:** Billing data is **only available in us-east-1 (N. Virginia)**. Creating a CloudWatch billing alarm in any other Region will show no data. Always switch to us-east-1 first.

**Setup path:**
Console → us-east-1 Region → CloudWatch → Alarms → Billing → Create Alarm → Estimated Charges > $5 → Create SNS topic → Add email → Confirm subscription

### AWS Budgets vs CloudWatch Billing Alarms

| Feature | AWS Budgets | CloudWatch Billing Alarms |
|---|---|---|
| Alert types | Cost, usage, RI utilisation, RI coverage | Cost only |
| Alert thresholds | Multiple (50%, 80%, 100%) | One per alarm |
| Forecasted alerts | Yes — alert before you overspend | No |
| Free tier | 2 budgets free/month | Standard CloudWatch charges apply |
| Best for | Complete cost control | Simple threshold alert |

**Recommendation:** Use AWS Budgets — more feature-rich, more flexible, and the first 2 budgets per month are free.

### Free Tier Usage Alert
A separate, built-in alert under AWS Billing Settings that warns when you approach Free Tier limits. Enable this independently of your manual billing alarm — it covers service-level Free Tier thresholds automatically.

---

## The IAM Security Checklist

AWS displays this as "Security Recommendations" on the IAM dashboard. All items should show green before you do any real lab work:

- [ ] Enable MFA on root account
- [ ] Delete or deactivate root account access keys
- [ ] Create individual IAM users (do not use root)
- [ ] Assign permissions via groups
- [ ] Apply least privilege to all policies
- [ ] Enable MFA on all IAM users with console access
- [ ] Review IAM Access Advisor regularly — remove unused permissions

---

## AWS Management Console — Navigation Notes

### Global vs Regional Services
Most services are regional — resources created in us-east-1 are invisible from af-south-1.

**Always-global services (not affected by Region selector):**
- IAM — users, groups, roles, policies are global
- Billing and Cost Management — global
- Route 53 — global DNS
- CloudFront — global CDN

**Regional services (check your Region before creating):**
- EC2, RDS, S3 (buckets are globally named but regionally stored), Lambda, VPC

> The Region selector in the top-right corner is the single most common source of "where did my resource go?" confusion for beginners.
