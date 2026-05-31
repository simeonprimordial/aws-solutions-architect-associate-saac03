# CloudTrail & Audit Logging — Day 4 Notes

---

## Why Audit Logging Is Non-Negotiable

Think of CloudTrail as CCTV for your AWS account. Every action is recorded — who did it, what they did, when, from which IP, and what the result was. You may not watch the footage every day, but when something goes wrong, you need it to exist.

Without CloudTrail, you cannot answer the questions that matter most during an incident: Who deleted that production database? Which IAM user modified the security group rule that opened port 22 to the world? What IP address accessed customer PII at 3am? These questions have forensic, regulatory, and legal weight. CloudTrail is what makes them answerable.

The Central Bank of Nigeria (CBN) Cybersecurity Framework requires financial institutions to maintain audit trails of all system activity. ISO 27001 explicitly requires evidence of access and activity logging. CloudTrail is AWS's implementation of both requirements.

---

## Core Concepts

### AWS CloudTrail
A service that continuously records all API calls and account activity across your AWS infrastructure — via the console, CLI, SDK, or AWS services acting on your behalf — and stores them as compressed JSON log files.

CloudTrail is regional in scope but can be configured to capture all regions from a single Trail. It is not a streaming service — logs are delivered to S3 in batches, typically within 15 minutes.

### Trail
A named configuration that defines:
- Where to deliver logs (S3 bucket — required; CloudWatch Logs — optional)
- Which regions to capture (single region or all regions)
- Which event types to include (Management, Data, Insights)

Without a Trail, you only have Event History — the last 90 days of management events in the console. A Trail is what makes logs persistent and alertable.

### Event History
A free, always-on view of the last 90 days of management events available directly in the CloudTrail console. No configuration needed.

Limitations: 90-day maximum retention. Management events only — no data events. Cannot trigger alerts. Cannot be exported automatically. Not a replacement for a Trail.

> ⚠️ **Exam Tip:** The most common CloudTrail trap on the exam is confusing Event History with a Trail. If a question mentions retaining logs for more than 90 days, detecting data-plane operations, or triggering automated alerts, the answer requires a Trail — not Event History alone.

### CloudTrail Event
A single JSON record of one API call or AWS action. Every event contains:

| Field | What It Records | Example |
|---|---|---|
| `userIdentity` | WHO made the call — principal ARN, type, account | `arn:aws:iam::123456789:user/developer1` |
| `eventTime` | WHEN — UTC timestamp | `2024-09-15T14:32:01Z` |
| `eventName` | WHAT action was performed | `DeleteBucket`, `CreateUser` |
| `eventSource` | WHICH service received the call | `s3.amazonaws.com`, `iam.amazonaws.com` |
| `awsRegion` | WHICH region the call was made in | `eu-west-1`, `af-south-1` |
| `sourceIPAddress` | WHERE from — the originating IP address | `197.210.52.8` |
| `requestParameters` | WHAT inputs were sent with the call | `{"bucketName": "my-bucket"}` |
| `responseElements` | What AWS returned — or `null` for read operations | object or null |
| `errorCode` / `errorMessage` | If the call was denied or failed | `AccessDenied` |

### S3 Bucket (Log Destination)
CloudTrail delivers log files to an S3 bucket. The bucket must have a bucket policy granting CloudTrail permission to write (`s3:PutObject`). AWS creates this policy automatically when you create a Trail via the console.

Log files are:
- Compressed (`.gz` format) — extract with 7-Zip on Windows, `gunzip` on Linux/Mac
- Organised by path: `BucketName/AWSLogs/AccountID/CloudTrail/Region/YYYY/MM/DD/`
- Delivered within approximately 15 minutes of the event
- Named with a timestamp and a random suffix to prevent collisions

### CloudTrail Lake
A managed event data store that lets you run SQL queries directly against CloudTrail events — without extracting files from S3. Supports multi-account, multi-region queries. Retains data for up to 7 years. Better for ad-hoc investigation and long-term analytics. Costs more than raw S3 storage.

---

## Three Event Types — What Gets Captured

### Management Events (Control Plane)
Operations that create, modify, or delete AWS resources and configurations.

- **Enabled by default** in all Trails
- Examples: `CreateBucket`, `DeleteBucket`, `AttachRolePolicy`, `CreateUser`, `RunInstances`, `TerminateInstances`, `AuthorizeSecurityGroupIngress`, `ConsoleLogin`
- This is what governance and compliance solutions depend on
- Volume: moderate — typically hundreds to thousands per day per account

### Data Events (Data Plane)
Operations on the data inside resources — reading and writing objects.

- **NOT enabled by default** — must be explicitly turned on per Trail and per service
- Examples:
  - S3: `GetObject`, `PutObject`, `DeleteObject`
  - Lambda: `InvokeFunction`
  - DynamoDB: `GetItem`, `PutItem`, `DeleteItem`
  - Cognito: `InitiateAuth`
- Volume: potentially millions per day for active S3 buckets — generates significant cost
- Required for: forensic investigation of data access, PII access audits, detecting exfiltration

> ⚠️ **Exam Tip:** If a question asks how to detect which IAM principal downloaded specific S3 objects, the answer is Data Events must be enabled on the Trail. Management Events alone will not capture `s3:GetObject`.

### CloudTrail Insights (Anomaly Detection)
An optional, additional-cost add-on that uses machine learning to detect unusual API activity patterns.

- Triggers on: unusual spike in write API call volume, burst of error rates, activity deviating from the baseline
- Delivers Insights events to the same S3 bucket as normal events
- Requires standard Management Events as a prerequisite
- It is **not** a standard event capture mechanism — it is a detection layer on top

> ⚠️ **Exam Tip:** Insights ≠ logging. The exam will offer "enable CloudTrail Insights" as an answer to questions about capturing root login events or detecting data access. Wrong — Insights detects API volume anomalies, it does not capture specific individual events.

---

## Key Trail Configurations

### Multi-Region Trail
A single Trail configured to capture events from all AWS regions simultaneously.

Without this: a single-region Trail only captures API calls made in that specific region. An attacker who creates IAM users in `us-west-2` while your Trail is in `eu-west-1` goes completely unlogged.

Always create multi-region Trails. There is no cost penalty — you pay for log storage (S3) and optionally for data events, not for the number of regions captured.

### Log File Integrity Validation
A one-checkbox option during Trail creation that enables SHA-256 hashing of every delivered log file.

How it works: CloudTrail generates a daily digest file containing the hash of every log file delivered that day. The digest file is cryptographically signed by AWS using a private key. You can verify the entire chain with:

```bash
aws cloudtrail validate-logs \
  --trail-arn arn:aws:cloudtrail:eu-west-1:123456789:trail/my-trail \
  --start-time 2024-09-01T00:00:00Z
```

If any log file was modified, deleted, or forged after delivery, validation fails. This makes CloudTrail logs admissible as forensic evidence — they have a cryptographic chain of custody.

Always enable this. It is a single checkbox and costs nothing.

### Organisation Trail
A Trail created from the management account that automatically captures events from every member account in the AWS Organisation. No per-account configuration needed. Logs are delivered to a central S3 bucket in the management account.

This is the correct architecture for multi-account logging — not individual Trails in each account.

### CloudWatch Logs Integration
Optional delivery of CloudTrail events to a CloudWatch Logs log group, in addition to S3. This enables near-real-time monitoring and alerting.

The alert pipeline:
```
CloudTrail → CloudWatch Logs Log Group
    → Metric Filter (matches specific event pattern)
    → CloudWatch Alarm (threshold breached)
    → SNS Topic
    → Email / PagerDuty / Slack
```

Example Metric Filter pattern for root account login:
```
{ $.userIdentity.type = "Root" && $.eventName = "ConsoleLogin" }
```

This is the standard architecture for real-time security alerting on CloudTrail events. Memorise the pipeline — the exam tests it regularly.

---

## Reading a CloudTrail Event — Annotated Example

This is a real-format CloudTrail event for a `DeleteBucket` action:

```json
{
  "eventVersion": "1.08",
  "userIdentity": {
    "type": "IAMUser",
    "principalId": "AIDAIOSFODNN7EXAMPLE",
    "arn": "arn:aws:iam::123456789012:user/developer1",
    "accountId": "123456789012",
    "userName": "developer1"
  },
  "eventTime": "2024-09-15T14:32:01Z",
  "eventSource": "s3.amazonaws.com",
  "eventName": "DeleteBucket",
  "awsRegion": "eu-west-1",
  "sourceIPAddress": "197.210.52.8",
  "userAgent": "AWS Console",
  "requestParameters": {
    "bucketName": "fintech-customer-data"
  },
  "responseElements": null,
  "requestID": "F2A8E7C1D9B3EXAMPLE",
  "eventID": "3a4b5c6d-7e8f-example",
  "readOnly": false,
  "eventType": "AwsApiCall",
  "managementEvent": true,
  "recipientAccountId": "123456789012"
}
```

Reading this event:
- **WHO:** IAM user `developer1` in account `123456789012`
- **WHAT:** Called `DeleteBucket` on S3
- **WHEN:** 15 September 2024 at 14:32 UTC
- **WHERE from:** IP `197.210.52.8` (Lagos-based IP range — expected. If this was a foreign IP, immediate investigation required)
- **WHICH resource:** Bucket `fintech-customer-data`
- **Outcome:** `responseElements: null` — for destructive actions, null means the action succeeded

If `responseElements` contained an error code, the deletion was denied. This is how you distinguish a successful breach from a blocked attempt.

---

## CloudTrail vs CloudWatch — The Critical Distinction

| Question | Service |
|---|---|
| Who deleted that S3 bucket? | CloudTrail |
| Which IAM user changed the security group? | CloudTrail |
| What IP logged in before the breach? | CloudTrail |
| Is EC2 CPU above 80%? | CloudWatch |
| How many Lambda errors in the last hour? | CloudWatch |
| Is RDS running out of storage? | CloudWatch |
| Alert when root user logs in | CloudTrail + CloudWatch Logs Metric Filter |
| Alert when EC2 latency exceeds 200ms | CloudWatch Alarm |

CloudTrail = **audit trail** (who, what, when, where). CloudWatch = **operational metrics** (how is it performing right now).

> ⚠️ **Exam Trap:** Questions about "monitoring IAM activity" or "detecting unauthorised API calls" → CloudTrail. Questions about "CPU utilisation", "disk I/O", or "request latency" → CloudWatch. The exam will mix these up.

---

## The Global Service Events Trap

IAM, STS, and CloudFront are global services. Their API calls are not associated with a specific region. By default, these events are only logged in `us-east-1`.

If your Trail is in a single region (e.g. `eu-west-1`), you will miss all IAM API calls unless you explicitly enable **"Include global service events"** on that Trail.

Multi-region Trails handle this automatically — one of several reasons to always prefer multi-region Trails.

---

## Common Exam Traps

- **Event History ≠ Trail.** Anything requiring more than 90 days retention, data event logging, or automated alerting requires a Trail. Event History is read-only and temporary.
- **Data Events are off by default.** Every candidate knows CloudTrail logs API calls. The trap is assuming S3 object reads and writes are included. They are not — you must explicitly enable Data Events.
- **~15 minute delivery delay.** CloudTrail is not real-time. For near-real-time alerting, integrate with CloudWatch Logs. If an exam option says "immediately detect using CloudTrail", it is likely wrong.
- **CloudTrail ≠ CloudWatch.** Separate tools, separate questions. Audit vs metrics. Do not confuse them.
- **Global service events in us-east-1 only.** A single-region Trail not in us-east-1 misses IAM and STS events. Always use multi-region Trails.
