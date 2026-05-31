# MFA & Security Best Practices Labs — Week 2 Day 5

---

## Lab 1: Run the AWS Security Hub Check

### Steps
1. Search for **Security Hub** in the console — click on it
2. Click **Enable Security Hub** (free for the first 30 days, then $0.001 per check/account/month)
3. On the Standards page, enable: **AWS Foundational Security Best Practices v1.0.0**
4. Wait 2–3 minutes for the initial assessment to run
5. Review the findings — note which controls show FAILED vs PASSED
6. Screenshot the summary dashboard

### What I Observed
Security Hub ran an automated assessment against the AWS Foundational Security Best Practices standard — a set of controls mapped directly to the IAM best practices from today's lesson. The initial findings within the first few minutes of a fresh account included:

FAILED controls (expected for a new learning account):
- `[IAM.1]` IAM policies should not allow full administrative privileges — flagged the `AdministratorAccess` policy being attached to a non-admin group
- `[IAM.4]` IAM root user access key should not exist — passed (we deleted root keys in Week 1)
- `[IAM.5]` MFA should be enabled for all IAM users with console access — flagged `analyst-user` with no MFA
- `[CloudTrail.1]` CloudTrail should be enabled and configured with at least one multi-region trail — PASSED (from yesterday's lab)
- `[S3.1]` S3 Block Public Access setting should be enabled — flagged the test bucket from Day 4

The score showed as approximately 62% compliant initially. This is normal for a learning account — production accounts with full hardening should aim for 90%+.

### What I Learned
- Security Hub gives you a single-pane visibility of your security posture against a defined standard. This is the kind of dashboard a CISO or CBN auditor asks to see. Having a score above 90% is a concrete, measurable security target.
- Each finding links directly to the remediation steps. This makes it actionable — not just a list of problems but a guided fix list.
- The findings map exactly to what was taught this week. `[IAM.5]` is the MFA enforcement from today. `[CloudTrail.1]` is yesterday's lab. The week's content is the implementation guide for these controls.
- Security Hub is not a configuration tool — it is detection only. Findings tell you what is wrong; you fix them yourself.

---

## Lab 2: Write the MFA Enforcement Policy

### Steps
1. Go to **IAM → Policies → Create policy → JSON tab**
2. Name: `RequireMFA`
3. Enter the following JSON:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyWithoutMFA",
      "Effect": "Deny",
      "NotAction": [
        "iam:CreateVirtualMFADevice",
        "iam:EnableMFADevice",
        "iam:GetUser",
        "iam:ListMFADevices",
        "iam:ListVirtualMFADevices",
        "iam:ResyncMFADevice",
        "sts:GetSessionToken"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

4. Add description: `Denies all actions except MFA setup if the session was authenticated without MFA. Attach to all user groups to enforce MFA enrollment before access.`
5. Click **Create policy**
6. Go to **IAM → User groups → Analysts**
7. Attach the `RequireMFA` policy to the Analysts group

### What I Observed
The `NotAction` field is the key mechanism here — it is the inverse of `Action`. Rather than listing everything to deny (which would require listing every single AWS API call), `NotAction` lists the small set of exceptions and denies everything else. Much cleaner and more maintainable.

The `BoolIfExists` operator handles the edge case where the condition key is absent. For CLI calls using access keys, `aws:MultiFactorAuthPresent` is not set at all. `BoolIfExists` treats the absence as "condition not triggered" — the Deny only fires when the key is present and explicitly false (i.e. a console login session without MFA).

The six MFA-related actions in the NotAction list are specifically those needed to set up an MFA device. Without them in the exception list, a new user locked out by this policy could not even enrol MFA — they would be permanently locked out and require admin intervention.

### Testing the Policy
Signed in as `analyst-user` without MFA in an incognito window:
- Attempted to list S3 buckets → **AccessDenied**
- Attempted to open IAM → Users → my own user → **accessible** (`iam:GetUser` is in NotAction)
- Navigated to Security Credentials → Assign MFA Device → **accessible** (MFA setup actions in NotAction)
- After enrolling MFA and re-logging in with the TOTP code → S3 listing worked normally

The policy worked exactly as designed: blocks everything except MFA setup, forces enrolment, then unlocks on re-authentication with MFA.

### What I Learned
- `NotAction` is a powerful but dangerous pattern. Used correctly here — it is exactly the right tool for a policy that must allow a small set of exceptions and deny everything else. In general use, `NotAction` should be approached carefully because a new AWS service launched after the policy was written is automatically included in the Deny.
- Testing policies in an incognito window remains the best workflow. You can verify the locked-out state and the unlock behaviour without interrupting your admin session.
- The user experience of MFA enforcement is important. If you block too aggressively — including the MFA setup actions — you create a permanent lockout situation that requires admin intervention for every new user. The NotAction exceptions are not optional.

---

## Lab 3: Download and Analyse the IAM Credential Report

### Steps
1. Go to **IAM → Credential Report**
2. Click **Download Report**
3. Open the CSV in Excel or Google Sheets
4. Create filters on: `mfa_active`, `access_key_1_last_used_date`, `password_last_used`
5. Filter `mfa_active = FALSE` — these are your non-compliant users
6. Filter `password_last_used > 90 days ago` — inactive accounts
7. Filter `access_key_1_last_used_date > 90 days ago` — stale access keys

### What I Observed
The report showed all IAM users in the account with these key columns populated:
- `user` — IAM username
- `mfa_active` — `true` for my admin user (MFA enabled), `false` for `analyst-user` before today's lab
- `password_enabled` — `true` for console-enabled users, `N/A` for programmatic-only users (`dev-user`)
- `access_key_1_active` — `true` if key exists and is active
- `access_key_1_last_used_date` — the last time the key was actually called
- `password_last_used` — the last console login timestamp

`dev-user` had `mfa_active: false` and `password_enabled: N/A`. This is expected — programmatic-only users cannot enrol MFA in the traditional sense (no console access). However, the access keys themselves should still be rotated regularly.

### What I Learned
- The Credential Report is best run monthly and archived to S3 for a rolling 12-month compliance history. This gives you a time-series view of your security posture — useful for CBN audits that ask for historical evidence.
- `mfa_active: false` on a console-enabled user is a remediation target. On a programmatic-only user it may be acceptable, but the access key age and usage should still be reviewed.
- The report does not track roles — only users. For service accounts using roles, the tracking mechanism is IAM Access Advisor on the role.

---

## Lab 4: Run IAM Access Advisor

### Steps
1. Go to **IAM → Users → select your admin user**
2. Click the **Access Advisor** tab
3. Review the services list — sorted by last accessed date
4. Note any services showing "Not accessed" or last accessed date > 90 days ago
5. Repeat for each IAM user and the roles created this week

### What I Observed
On my admin user, the Access Advisor showed:
- IAM: accessed today (from today's labs)
- CloudTrail: accessed yesterday
- S3: accessed across multiple days
- EC2: last accessed 3 days ago (from the IAM lab)
- Athena: **Not accessed** — `analyst-user` has `AmazonAthenaFullAccess` from Day 1 but Athena has never been used

The Athena finding is a real least-privilege opportunity. `analyst-user` was given Athena access when the group was created because the lab required it — but Athena was never actually used. In a production environment, after 30–90 days of observation, this permission should be removed.

### What I Learned
- Access Advisor turns the principle of least privilege from an aspiration into an operational practice. You do not need to know upfront what permissions someone needs — you can grant what seems right, observe for 90 days, and remove what was never touched.
- The 90-day window is the standard benchmark. AWS's own security tooling (Security Hub, Access Analyzer) uses 90 days as the threshold for "unused" permissions.
- Access Advisor works on roles too — this is important for service roles. An EC2 instance role that has `AmazonS3FullAccess` but only ever calls `s3:GetObject` should have its policy narrowed after observation.

---

## Lab 5: Enable IAM Access Analyzer

### Steps
1. Go to **IAM → Access Analyzer**
2. Click **Create Analyzer**
3. Analyzer type: **Account** (or Organisation if working in a multi-account structure)
4. Give it a name: `account-access-analyzer`
5. Click **Create Analyzer**
6. Wait a few minutes — initial findings appear automatically
7. Review any findings shown

### What I Observed
Access Analyzer ran within 2 minutes and returned one finding: the S3 bucket created in the CloudTrail lab (`cloudtrail-logs-[name]-[date]`) had a bucket policy granting `cloudtrail.amazonaws.com` permission to write logs — flagged as "cross-service access" from a principal outside the account.

This is an expected and intentional finding — CloudTrail requires write access to deliver logs. I archived the finding after confirming it was intentional access.

### What I Learned
- Not every Access Analyzer finding is a problem. Legitimate cross-account and cross-service access patterns (CloudTrail writing to S3, cross-account deployment pipelines, partner integrations) will all generate findings. The correct workflow is: review each finding, confirm it is intentional, and archive it. New unarchived findings indicate new unexpected access paths.
- The fact that CloudTrail's bucket policy generates a finding shows how sensitive Access Analyzer is. This is good — it catches things that would be invisible otherwise.
- For production accounts, enable the analyser at the Organisation level (not just account level). This lets you detect cross-account access paths from member accounts to shared resources.

---

## Bonus Lab: AWS Trusted Advisor Security Checks

### Steps
1. Search for **Trusted Advisor** in the console
2. Navigate to the **Security** category
3. Review the security recommendations

### What I Observed
Trusted Advisor's free security checks flagged:
- **MFA on Root Account** — PASS (enabled in Week 1)
- **IAM Use** — flagged that S3 buckets were being accessed directly rather than via IAM roles in some cases
- **Security Groups — Unrestricted Access** — no flagged issues (no security groups with `0.0.0.0/0` inbound on sensitive ports yet)
- **Amazon S3 Bucket Permissions** — flagged the test bucket from Day 4 as having broad permissions

Added the top 3 findings to the Security Runbook as additional recommendations.

### What I Learned
- Trusted Advisor is a separate tool from Security Hub but they overlap. Security Hub checks against formal compliance standards (CIS Benchmark, FSBP). Trusted Advisor gives broader operational recommendations across cost, performance, and security.
- For production accounts, Trusted Advisor Business or Enterprise tier (included with those support plans) gives access to a much wider set of checks. The free tier covers only the most critical security controls.
