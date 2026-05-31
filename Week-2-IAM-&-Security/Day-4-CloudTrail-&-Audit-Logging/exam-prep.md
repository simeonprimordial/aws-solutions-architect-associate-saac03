# CloudTrail & Audit Logging — Exam Prep — Week 2 Day 4

---

## SAA-C03 Context

CloudTrail appears in Domain 2 (Design Secure Architectures) for security monitoring and audit questions, and in Domain 4 (Cost-Optimised Architectures) for log retention cost trade-offs (CloudTrail Lake vs raw S3 vs Glacier). Questions test whether you know the correct combination of services for a complete audit solution — not just CloudTrail alone.

---

## The Five Exam Traps — This Topic

**Trap 1 — Event History expires at 90 days.**
Event History is free and automatic. A Trail + S3 is what makes logs persistent. Any question mentioning retention beyond 3 months, alerting, or data events needs a Trail. "Enable CloudTrail Event History" as an answer option for long-term retention is always wrong.

**Trap 2 — Data Events are OFF by default.**
Management events (create, delete, configure resources) are always on. Data events (read/write objects — S3 GetObject, Lambda Invoke) are not. If a question asks how to detect who downloaded specific S3 files, the answer requires enabling S3 Data Events on the Trail.

**Trap 3 — CloudTrail has a ~15 minute delivery lag.**
Not real-time. If an answer option says "use CloudTrail to immediately alert on" something, it is likely wrong unless CloudWatch Logs integration is also included in the solution. The alert pipeline — CloudTrail → CloudWatch Logs → Metric Filter → Alarm → SNS — is near-real-time on the CloudWatch side.

**Trap 4 — CloudTrail ≠ CloudWatch.**
CloudTrail = audit trail (who, what, when, where). CloudWatch = operational metrics (CPU, latency, error rates, disk). Security and governance questions → CloudTrail. Performance and operational questions → CloudWatch. Mixing them up is the #1 distractor for this topic pair.

**Trap 5 — Global service events only in us-east-1.**
IAM, STS, CloudFront: their events are logged in `us-east-1` only. A single-region Trail in any other region misses all IAM API calls. Multi-region Trails handle this automatically. If a question describes a Trail that is missing IAM activity, the reason is always: single-region Trail without global service event inclusion, or Trail in a region other than `us-east-1`.

---

## CloudTrail Architecture Decision Table

| Requirement | Configuration |
|---|---|
| Capture all regions from one Trail | Multi-region Trail enabled |
| Capture all 12 accounts centrally | Organisation Trail from management account |
| Retain logs for 3 years | S3 lifecycle policy (move to Glacier after 90 days) |
| Detect root login in real time | CloudTrail + CloudWatch Logs + Metric Filter + Alarm + SNS |
| Detect who downloaded specific S3 files | Enable S3 Data Events on the Trail |
| Prove log files were not tampered with | Enable log file integrity validation (SHA-256) |
| Run SQL queries on 2 years of logs | CloudTrail Lake |
| Detect unusual API call volume spikes | CloudTrail Insights |
| Detect IAM API calls made from eu-west-1 Trail | Enable "Include global service events" OR use multi-region Trail |
| Auto-remediate when an IAM key is used from unknown IP | CloudTrail → EventBridge → Lambda |

---

## Practice Question — Week 2 Day 4

**Scenario:** A security engineer at a Lagos fintech needs to ensure all AWS API activity across 12 accounts in an AWS Organisation is captured, stored for 3 years, and triggers an automated alert whenever the root user logs in to any account. Which TWO configurations meet all requirements?

**A.** Enable CloudTrail Event History in each account. Store event history exports in S3 manually.

**B.** Create an Organisation Trail in the management account with multi-region enabled. Deliver logs to a central S3 bucket with a 3-year lifecycle policy.

**C.** Enable CloudTrail Insights in each member account to detect root login anomalies.

**D.** Create a CloudWatch Logs metric filter for the organisation trail that matches root account ConsoleLogin events, and configure a CloudWatch Alarm to publish to SNS.

**Answers: B and D**

**Why A is wrong:** Event History retains only 90 days (not 3 years), requires per-account setup, and cannot trigger automated alerts. Manual exports are not a sustainable solution.

**Why C is wrong:** CloudTrail Insights detects unusual API volumes using ML — it is not a standard event capture mechanism and does not specifically detect or alert on root logins. It would not satisfy any of the three requirements.

**Why B is correct:** An Organisation Trail automatically covers all 12 accounts from a single configuration in the management account. Multi-region ensures no region is missed. S3 lifecycle policy handles 3-year retention at the lowest cost.

**Why D is correct:** The standard root login alert pattern. CloudTrail captures the `ConsoleLogin` event. CloudWatch Logs receives it via the Trail's log group integration. The metric filter `{ $.userIdentity.type = "Root" && $.eventName = "ConsoleLogin" }` matches. The Alarm triggers. SNS sends the notification.

---

## Forensic Investigation Framework — CloudTrail

When you receive an incident report and need to investigate using CloudTrail, follow this order:

```
1. SCOPE the timeframe
   When was the suspicious activity? Start ±24h, narrow from there.

2. IDENTIFY the principal
   Find the userIdentity.arn. Is it a known IAM user? An assumed role?
   If assumed role: check sessionContext for the source principal.

3. MAP the blast radius
   Search all events by that principal in the timeframe.
   Key events to look for:
   - iam:CreateUser, iam:CreateAccessKey (attacker creating persistence)
   - iam:AttachUserPolicy, iam:PutUserPolicy (escalating privileges)
   - cloudtrail:StopLogging (attempt to destroy evidence — check if blocked by SCP)
   - s3:GetObject (data exfiltration — only visible if Data Events enabled)
   - ec2:RunInstances (cryptomining or lateral movement)

4. VALIDATE log integrity
   Run: aws cloudtrail validate-logs
   Confirm no log files were modified after delivery.
   If validation passes: logs are admissible as forensic evidence.

5. DOCUMENT and REPORT
   Export the relevant events to a structured timeline.
   Include: eventTime, userIdentity.arn, eventName, sourceIPAddress, requestParameters.
   This is what you submit to the CBN in the 48-hour incident disclosure window.
```

---

## Quick-Recall Test

Answer these without looking at notes:

1. What is the maximum retention period of CloudTrail Event History?
2. Are S3 `GetObject` events captured by a default Trail?
3. What is the approximate delivery lag for CloudTrail logs to S3?
4. A Trail is configured in `eu-west-1`. A developer creates an IAM user. In which S3 folder does the event appear?
5. You need to detect root user logins in real time and alert the security team. What is the full pipeline?
6. What CLI command validates that CloudTrail log files have not been tampered with?
7. What makes CloudTrail Insights different from standard Management Events?

Answers:
1. 90 days.
2. No — S3 data events (`GetObject`, `PutObject`) are not enabled by default. Must be explicitly turned on.
3. Approximately 15 minutes.
4. In the `us-east-1` folder — IAM is a global service, its events always appear in `us-east-1`.
5. CloudTrail → CloudWatch Logs Log Group → Metric Filter (`userIdentity.type = Root AND eventName = ConsoleLogin`) → CloudWatch Alarm → SNS Topic → Email/notification.
6. `aws cloudtrail validate-logs --trail-arn [arn] --start-time [timestamp]`
7. Insights uses ML to detect unusual API volume patterns. It is an anomaly detection layer — not a record of individual events. Management Events capture individual API calls. Insights runs on top of Management Events and cannot exist without them.
