# CloudTrail & Audit Logging — Week 2 Day 4

## Topic
CloudTrail & Audit Logging — Your AWS CCTV System

This repository contains my notes, labs, and screenshots from Day 4 of Week 2. Days 1–3 covered how to control who can do what in AWS. Today is about recording everything that actually happens — CloudTrail is the audit log that makes AWS fully accountable. It is the difference between knowing something happened and being able to prove it.

---

## What I Learned

### What CloudTrail Records
Every API call in an AWS account — console, CLI, SDK, or service-to-service — captured as a JSON record answering four questions:
- **WHO** — the IAM principal (user, role, service) that made the call
- **WHAT** — the specific API action (`DeleteBucket`, `CreateUser`, `RunInstances`)
- **WHEN** — ISO 8601 UTC timestamp
- **WHERE** — source IP address and AWS Region

### Event History vs Trails
- **Event History** — free, always on, last 90 days of management events only, no alerting, no persistence beyond 90 days
- **Trail** — explicit configuration you create; delivers compressed JSON logs to S3; persists indefinitely; enables alerting via CloudWatch Logs; required for compliance

### Three Event Types
- **Management Events** — control plane operations (create/delete/modify resources); enabled by default; what governance and compliance rely on
- **Data Events** — data plane operations (`s3:GetObject`, `Lambda:InvokeFunction`, `DynamoDB:GetItem`); **NOT enabled by default**; high volume; costs extra
- **CloudTrail Insights** — ML-based anomaly detection on top of management events; optional add-on; not a standard event capture mechanism

### Critical Configurations for a Proper Trail
- Multi-region enabled — captures API calls in every AWS region from one Trail
- Log file integrity validation — SHA-256 hashing makes logs forensic-grade tamper-evident
- CloudWatch Logs delivery — enables near-real-time metric filters and alarms
- S3 lifecycle policy — controls long-term retention and cost

### The Alert Pipeline
`CloudTrail → CloudWatch Logs → Metric Filter → CloudWatch Alarm → SNS Topic → Email / PagerDuty / Slack`

---

## Hands-On Labs Completed
- Created a multi-region Trail (`my-account-audit-trail`) with S3 log delivery
- Generated audit events: created IAM user, created S3 bucket, attempted EC2 launch, deleted both resources
- Navigated the S3 folder structure and downloaded a `.json.gz` log file
- Read and analysed the raw JSON — identified `CreateUser` and `DeleteUser` event records
- Documented a 3-step incident response process using CloudTrail as evidence
- Investigated `ConsoleLogin` events in Event History and verified source IP (Bonus)

---

## AWS Services Used
- AWS CloudTrail — Trail creation, event types, log delivery
- Amazon S3 — log storage destination
- CloudWatch Logs — log streaming for alerting (configured)
- AWS CloudTrail Event History — used for the bonus ConsoleLogin investigation

---

## Screenshots
All screenshots are stored in the `/screenshots` folder:
- `trail-created-active.png` — CloudTrail trail created with green active status
- `s3-cloudtrail-folder-structure.png` — S3 bucket showing `AWSLogs/AccountID/CloudTrail/Region/YYYY/MM/DD/` path
- `raw-json-log-open.png` — `.json.gz` file extracted and open in VS Code
- `createuser-event-annotated.png` — `CreateUser` event with `eventName`, `userIdentity`, `sourceIPAddress`, `requestParameters` labelled
- `deleteuser-event-annotated.png` — `DeleteUser` event for comparison
- `consolelLogin-event-history.png` — Event History filtered by `ConsoleLogin` showing own sign-in with source IP (Bonus)

---

## Log Samples
See `/log-samples/` for annotated CloudTrail event JSON examples from today's lab.

---

## Challenges & Blockers
See `/notes/challenges.md` for issues I ran into and how I resolved them.

---

## Key Exam Traps to Remember
- Event History expires after **90 days** and cannot trigger alerts — it is not a substitute for a Trail
- Data Events are **OFF by default** — not everything is captured automatically
- CloudTrail delivers logs with a **~15 minute delay** — it is not real-time
- **CloudTrail ≠ CloudWatch**: CloudTrail = who did what (audit); CloudWatch = how is it performing (metrics)
- Global service events (IAM, STS, CloudFront) are only captured in **us-east-1** by default — single-region Trails miss them; multi-region Trails handle it automatically

---

## Goal
Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
