# Challenges & How I Solved Them — Week 2 Day 5

This file tracks blockers I encountered during labs and how I resolved them. Documenting this helps me learn from mistakes and may help others hitting the same issues.

---

## Challenge 1: MFA Policy Locked Out analyst-user From MFA Setup

**What happened:**
When first testing the `RequireMFA` policy, I wrote an earlier version using `Action: *` instead of `NotAction`. This Denied everything including the MFA setup actions. When I tried to log in as `analyst-user` without MFA, the policy blocked even the MFA enrolment pages — the user had no way to fix the situation themselves.

**What I tried:**
- Attempted to open IAM → Security Credentials as `analyst-user` → got AccessDenied
- Signed back into admin account and reviewed the policy
- Found the issue: `Action: *` denied everything including `iam:EnableMFADevice`

**Resolution:**
Rewrote the policy using `NotAction` instead of `Action`. This inverts the logic: deny everything EXCEPT the listed MFA setup actions. The NotAction exceptions give the locked-out user exactly the permissions needed to resolve their own lockout.

**Lesson learned:**
MFA enforcement policies must use `NotAction` with the MFA setup exceptions — not `Action: *`. Using `Action: *` creates a permanent lockout for new users and requires admin intervention every time. `NotAction` is the correct pattern for this specific use case.

---

## Challenge 2: BoolIfExists vs Bool — CLI Calls Not Covered Correctly

**What happened:**
When first writing the policy, I used `Bool` instead of `BoolIfExists` for the `aws:MultiFactorAuthPresent` condition. When testing with the AWS CLI using access keys, all calls were denied — even from my admin user whose keys I knew were legitimate.

**What I tried:**
- Checked the IAM docs for `aws:MultiFactorAuthPresent`
- Found the note: when using CLI access keys (not a console session), the condition key `aws:MultiFactorAuthPresent` is not present in the request context at all
- `Bool` requires the key to be present — if absent, the condition evaluates in an unexpected way
- `BoolIfExists` handles absence correctly: if the key doesn't exist, the condition does not match and the Deny does not fire

**Resolution:**
Changed `Bool` to `BoolIfExists` in the condition block. CLI calls using access keys are now handled correctly — the Deny does not fire because the condition key is absent (not false).

**Lesson learned:**
For conditions where the key may sometimes be absent (IAM context keys vs Global context keys), always use the `IfExists` variant: `BoolIfExists`, `StringEqualsIfExists`, etc. The standard versions require the key to be present — the IfExists versions gracefully handle absence. This is particularly important for `aws:MultiFactorAuthPresent` which is absent on programmatic/CLI sessions.

---

## Challenge 3: Security Hub Showing 200+ Failed Controls on a Fresh Account

**What happened:**
After enabling Security Hub with the AWS Foundational Security Best Practices standard, it returned over 200 FAILED controls. This looked alarming — is the account fundamentally broken?

**What I tried:**
- Read the individual finding descriptions carefully
- Found that many failures were for services not yet configured: GuardDuty not enabled, AWS Config not enabled, VPC Flow Logs not enabled, etc.
- These services were not set up yet because the curriculum hasn't reached them

**Resolution:**
The large number of failures is normal and expected for a new account that hasn't been fully configured. Security Hub checks against an ambitious complete security baseline — it is not saying your account is compromised, it is showing you what remains to be configured. The relevant controls for Week 2 (IAM, CloudTrail, root MFA) were all either PASSED or were on today's remediation list.

**Lesson learned:**
Security Hub is a forward-looking compliance tool — it shows you the target state, not necessarily your current state. A new account will fail many checks. The correct approach is to work through them systematically by topic area, not to panic at the overall number. By the end of the course, most of these checks should be green.

---

## Challenge 4: IAM Access Analyzer Flagging the CloudTrail S3 Bucket Policy

**What happened:**
After enabling IAM Access Analyzer, it immediately flagged the CloudTrail log bucket from yesterday's lab as having an "external access" finding. I thought I had misconfigured the bucket.

**What I tried:**
- Opened the finding details — it showed `cloudtrail.amazonaws.com` as the external principal with `s3:PutObject` permission
- This is the bucket policy that CloudTrail requires to deliver logs — we set it up deliberately in Day 4
- The access path is intentional — CloudTrail writing logs to the bucket is correct behaviour

**Resolution:**
Archived the finding. When you archive a finding, Access Analyzer marks it as "intentional" and stops flagging it in future scans. New unarchived findings indicate new unexpected access — archived ones are documented as known-good.

**Lesson learned:**
Not every Access Analyzer finding is a security problem. Many will be legitimate cross-service or cross-account access patterns. The correct workflow is: review each finding, understand it, confirm it is intentional, then archive it. The archive is your record of reviewed-and-approved external access. Any new finding that appears after a quiet period warrants investigation.

---

## Challenge 5: Trusted Advisor Showing Some Checks as "Unavailable"

**What happened:**
Several Trusted Advisor security checks showed as "Unavailable" with a message about needing a Business or Enterprise support plan.

**What I tried:**
- Read the check descriptions — these were the extended checks beyond the free tier
- The free tier of Trusted Advisor includes only 7 security checks (the most critical ones: MFA on root, IAM use, security groups, S3 permissions, etc.)
- The full check library (50+ security checks) requires Business/Enterprise support plans

**Resolution:**
Used the available free checks, documented the top 3 applicable findings in the Security Runbook, and noted that the full suite is available on Business/Enterprise support plans.

**Lesson learned:**
AWS Trusted Advisor has two tiers: free (basic checks for all accounts) and paid (full suite for Business/Enterprise support). For personal learning accounts, the free tier covers the most important security controls. For production accounts where you need the full check suite, it is included in Business support ($100+/month minimum). Security Hub covers more checks at lower cost than upgrading support tiers purely for Trusted Advisor.

---

*Add new challenges here as they come up in future days.*
