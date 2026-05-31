# AWS Account Security Hardening Runbook
### Version 1.0 — Built during AWS Cloud Accelerator, Week 2

---

## Section 1: Account Setup Checklist
*Run on every new AWS account before any workloads are deployed.*

### Root Account
- [ ] Enable MFA on root immediately (FIDO2 security key preferred; virtual MFA as minimum)
- [ ] Delete all root access keys — go to IAM → Security Credentials → delete both access keys if present
- [ ] Store root password in a company secrets vault with dual-person access requirement
- [ ] Set root email to a shared security team inbox, not a personal email
- [ ] Enable billing alerts: set a Zero Spend Budget alert (AWS Budgets) and a Cost Anomaly Detection alert
- [ ] Never use root for daily operations — create a separate IAM admin user immediately

### IAM Admin User
- [ ] Create an IAM admin user: `[company]-iam-admin` — do NOT name it "admin" (too predictable)
- [ ] Attach `AdministratorAccess` policy to this user only
- [ ] Enable MFA on this admin user (virtual MFA minimum; hardware MFA for regulated environments)
- [ ] Download access keys only if CLI access is needed — store securely, never commit to code
- [ ] Use this admin user for all account configuration — never root

### Account-Wide Security Baseline
- [ ] Enable CloudTrail: multi-region trail, deliver to S3, enable log file integrity validation, enable CloudWatch Logs
- [ ] Enable IAM Access Analyzer (account or organisation scope)
- [ ] Enable AWS Security Hub: AWS Foundational Security Best Practices standard
- [ ] Enable GuardDuty (threat detection — Week 3 topic, enable now)
- [ ] Enable AWS Config (resource configuration history)
- [ ] Set S3 Block Public Access at the account level (S3 → Block Public Access → enable all)
- [ ] Enable IAM password policy: minimum 12 characters, require uppercase, lowercase, numbers, symbols

---

## Section 2: IAM Best Practices
*The 8 rules — apply to every IAM configuration decision.*

1. **Root account: MFA + no access keys + never use daily.** This is non-negotiable. No exceptions.

2. **No root access keys.** Root access keys cannot be scoped to specific permissions or restricted by IAM policies. Delete them. If they were created, treat the account as potentially compromised and audit all activity since key creation.

3. **MFA for all human users.** Attach the `RequireMFA` policy to every user group. No console-enabled user should be able to operate without MFA. Verify via the IAM Credential Report monthly.

4. **Least privilege always.** Never assign `AdministratorAccess` to a service account or regular user. Start from zero and add only what is needed. Use IAM Access Advisor quarterly to identify and remove unused permissions.

5. **Roles over access keys for services.** EC2 instances, Lambda functions, ECS tasks, and CI/CD pipelines should use IAM roles and STS temporary credentials. No access keys in application code, environment variables, or Docker images.

6. **Rotate access keys every 90 days.** If access keys must exist: rotate at 90 days. Disable at 180 days if never used. Delete at 365 days. Automate this with a Lambda function triggered on a schedule.

7. **Permission Boundaries on delegated role creation.** Any team that can create IAM roles must have a Permission Boundary that prevents privilege escalation. The boundary should exclude `iam:*` at minimum for non-IAM-admin teams.

8. **Access Analyzer on continuously.** Review findings weekly. Archive intentional external access. Treat unexpected findings as incidents until proven otherwise.

---

## Section 3: What to Do If Credentials Are Compromised

### Immediate Response (First 30 Minutes)

**Step 1 — Contain**
```
If IAM user access keys are compromised:
  → IAM → Users → [compromised user] → Security Credentials
  → Deactivate both access keys immediately
  → Do NOT delete yet — preserve for forensics

If IAM user console password is compromised:
  → IAM → Users → [compromised user] → Security Credentials
  → Disable console access (change the sign-in mode to Programmatic access only)
  → Force a logout of all active sessions (does not exist as one button — see Step 2)

If root credentials are compromised:
  → Log in as root immediately from a trusted device
  → Change root password
  → Review MFA devices — remove any unrecognised devices
  → Treat as a critical incident — escalate to senior team and legal
```

**Step 2 — Revoke Active Sessions**
There is no single "logout all sessions" button in IAM. To revoke an IAM role's active sessions:
- Attach an inline Deny policy with `Effect: Deny, Action: *, Resource: *` — this immediately blocks all requests from existing sessions
- Remove the policy after rotating credentials to restore access for legitimate users

**Step 3 — Audit with CloudTrail**
Query CloudTrail for all actions by the compromised principal in the 72 hours before and after the suspected compromise:
- Look for: `iam:CreateUser`, `iam:CreateAccessKey`, `iam:AttachUserPolicy` (attacker creating persistence)
- Look for: `cloudtrail:StopLogging` (attempt to destroy evidence — should be blocked by root SCP)
- Look for: `s3:GetObject` in high volume from unexpected IPs (data exfiltration — requires Data Events enabled)
- Look for: `ec2:RunInstances` in unusual regions (cryptomining)

**Step 4 — Validate Log Integrity**
```bash
aws cloudtrail validate-logs \
  --trail-arn arn:aws:cloudtrail:[region]:[account]:trail/[trail-name] \
  --start-time [72-hours-before-incident]
```
If validation passes, logs are forensically admissible. If it fails, escalate immediately — log tampering is a separate critical incident.

**Step 5 — Document and Report**
- Export the CloudTrail events for the incident window to a time-sorted table
- Document: what was accessed, by which principal, from which IP, at what time, with what result
- If customer data (PII) was accessed: NDPC breach notification obligation may apply (72-hour window)
- If in a regulated sector: CBN incident disclosure timeline applies (typically 24–48 hours)

**Step 6 — Rotate All Related Credentials**
- Rotate all access keys for the compromised user and any roles they could have assumed
- Rotate any secrets that may have been accessed (Secrets Manager, Parameter Store)
- If attacker created new IAM users or access keys during the incident: delete all of them
- Enable or strengthen MFA on the affected account

---

## Section 4: Monthly Security Review Checklist

### Week 1 of Each Month — Credential Audit
- [ ] Download IAM Credential Report
- [ ] Filter `mfa_active = false` → notify affected users and schedule remediation
- [ ] Filter `access_key_1_last_used_date` > 90 days → disable those keys pending review
- [ ] Filter `password_last_used` > 90 days → suspend inactive accounts pending review
- [ ] Filter `access_key_1_last_rotated` > 90 days → schedule rotation
- [ ] Archive the report to S3 with a date prefix for compliance history

### Week 2 of Each Month — Permissions Review
- [ ] Open IAM Access Advisor on the top 5 most-privileged roles/users
- [ ] Identify any services not accessed in the past 90 days
- [ ] Submit proposed permission removals to team lead for approval
- [ ] Apply approved removals

### Week 3 of Each Month — External Access Review
- [ ] Open IAM Access Analyzer findings
- [ ] Review any new unarchived findings since the last review
- [ ] For each finding: confirm it is intentional → archive; or fix the policy
- [ ] Check for any S3 buckets with public access warnings

### Week 4 of Each Month — Security Hub Score Review
- [ ] Open Security Hub → Summary dashboard
- [ ] Review current compliance score vs previous month
- [ ] Prioritise and assign top 3 FAILED controls for remediation in the coming month
- [ ] Document any controls marked as exceptions with business justification

---

## Additional Recommendations — From AWS Trusted Advisor

*(Added from Bonus Lab — Trusted Advisor security checks)*

1. **S3 Bucket Permissions** — Regularly audit S3 buckets for broad public access. Trusted Advisor flags any bucket with public read or write permissions. Enable S3 Block Public Access at the account level as a baseline control.

2. **Security Groups — Unrestricted Access** — Trusted Advisor flags security groups with `0.0.0.0/0` inbound rules on sensitive ports (SSH/22, RDP/3389). In production: restrict inbound SSH to a known jump host IP range, never open to the internet.

3. **MFA on Root** — Trusted Advisor checks this at account level. If this check shows as WARNING, stop everything and fix it before proceeding with any other work.
