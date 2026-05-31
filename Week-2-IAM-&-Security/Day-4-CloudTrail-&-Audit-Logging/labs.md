# CloudTrail Labs — Week 2 Day 4

---

## Lab 1: Enable CloudTrail

### Steps
1. Search for **CloudTrail** in the AWS console — click on it
2. Click **Create trail**
3. Trail name: `my-account-audit-trail`
4. Storage location: **Create new S3 bucket**
   - Bucket name: `cloudtrail-logs-[yourname]-[date]` (must be globally unique)
5. Log file SSE-KMS encryption: leave unchecked (free tier — KMS adds cost)
6. CloudWatch Logs: enable and create a new log group (`/aws/cloudtrail/my-account-audit-trail`)
7. Ensure **Apply trail to all regions** is selected — this is non-negotiable
8. Events: leave default — Management events only, Read and Write
9. Log file validation: **enable** — single checkbox, no cost, forensic-grade tamper detection
10. Click **Create trail**

### What I Observed
The console creates a specific S3 bucket policy automatically when you choose a new bucket — it grants `cloudtrail.amazonaws.com` permission to call `s3:PutObject` to the specific path prefix for your account and trail name. Without this policy, CloudTrail cannot write to the bucket and logs would be silently dropped.

The folder structure AWS creates in the bucket is: `AWSLogs/[AccountID]/CloudTrail/[Region]/[YYYY]/[MM]/[DD]/`. With multi-region enabled, there is one subfolder per active region.

After creating the trail, the console shows it as **Active** with a green indicator. The first log files do not arrive immediately — there is nothing to log yet.

### What I Learned
- Always enable multi-region from day one. You cannot retroactively capture events from regions your Trail was not monitoring. Past blind spots stay blind.
- Log file integrity validation is always worth enabling. It costs nothing and the cryptographic chain of custody it provides could be the difference between admissible and inadmissible evidence in a regulatory investigation.
- CloudWatch Logs integration is optional but important. Without it, you only have delayed S3 log files. With it, you can build real-time alerting for critical events like root logins.
- The S3 bucket policy created automatically by CloudTrail is the minimum required. If you ever point a Trail at an existing bucket, you must add the policy manually.

---

## Lab 2: Generate Audit Events

### Steps
Performed the following actions in the AWS console — each generates a CloudTrail management event:

1. **IAM → Users → Create user:** `test-cloudtrail-user` (console access, no group)
2. **S3 → Create bucket:** `test-bucket-cloudtrail-[yourname]` (default settings, `af-south-1`)
3. **EC2 → Launch instance:** completed the wizard through to the Review page — then clicked **Cancel** (the event `RunInstances` is only logged on launch, not on viewing the wizard)
4. **IAM → Users → Delete:** `test-cloudtrail-user`
5. **S3 → Delete:** `test-bucket-cloudtrail-[yourname]` (emptied first, then deleted)

Then waited 10–15 minutes for CloudTrail to deliver log files to S3.

### What I Observed
The 15-minute delivery window is real. Nothing appeared in S3 immediately. After about 12 minutes, the first `.json.gz` files appeared in the `YYYY/MM/DD` folder for today's date.

Cancelling the EC2 instance wizard does not generate a CloudTrail event — no API call was made to AWS until I clicked Launch. This reinforced the point that CloudTrail records API calls, not console interactions. Clicking buttons does nothing until the API is called.

### What I Learned
- CloudTrail is API-level, not UI-level. Browsing menus, previewing config pages, and filling in form fields before submitting — none of this creates a log entry. The log is created the moment the API call is executed.
- Deleting an S3 bucket requires emptying it first. This generates both `DeleteObjects` events (for each object deleted) and `DeleteBucket` for the final deletion — three or more log entries from one action.
- The 15-minute delivery window is important for incident response planning. If you need near-real-time detection, you need CloudWatch Logs integration — S3-only delivery has a lag you cannot eliminate.

---

## Lab 3: Find and Read the Log Files

### Steps
1. Navigate to **S3 → your CloudTrail bucket**
2. Browse to: `AWSLogs → [Account ID] → CloudTrail → [Region] → [Year] → [Month] → [Day]`
3. Find a `.json.gz` file — click it → **Download**
4. Extract the file:
   - Windows: right-click → 7-Zip → Extract here
   - Mac: double-click
   - Linux: `gunzip filename.json.gz`
5. Open the JSON in VS Code or any text editor
6. Search for `"eventName": "CreateUser"` — locate the event
7. Search for `"eventName": "DeleteUser"` — compare the two entries

### The CreateUser Event

```json
{
  "eventVersion": "1.08",
  "userIdentity": {
    "type": "IAMUser",
    "arn": "arn:aws:iam::123456789012:user/my-admin-user",
    "userName": "my-admin-user"
  },
  "eventTime": "2024-09-15T10:14:23Z",
  "eventSource": "iam.amazonaws.com",
  "eventName": "CreateUser",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "197.210.52.8",
  "requestParameters": {
    "userName": "test-cloudtrail-user"
  },
  "responseElements": {
    "user": {
      "userName": "test-cloudtrail-user",
      "userId": "AIDAIOSFODNN7EXAMPLE",
      "arn": "arn:aws:iam::123456789012:user/test-cloudtrail-user",
      "createDate": "Sep 15, 2024 10:14:23 AM"
    }
  }
}
```

### The DeleteUser Event

```json
{
  "userIdentity": {
    "type": "IAMUser",
    "arn": "arn:aws:iam::123456789012:user/my-admin-user",
    "userName": "my-admin-user"
  },
  "eventTime": "2024-09-15T10:28:47Z",
  "eventSource": "iam.amazonaws.com",
  "eventName": "DeleteUser",
  "awsRegion": "us-east-1",
  "sourceIPAddress": "197.210.52.8",
  "requestParameters": {
    "userName": "test-cloudtrail-user"
  },
  "responseElements": null
}
```

### What I Observed
The IAM events (`CreateUser`, `DeleteUser`) appeared in the `us-east-1` region folder, not `af-south-1` — even though I performed them from the Lagos console. This is the global service events behaviour: IAM is global and all its API calls are logged in `us-east-1`.

The `CreateUser` event has a `responseElements` object containing the new user's ARN and userId. The `DeleteUser` event has `responseElements: null` — which is expected, deletion does not return a resource object. This is how you tell successful from failed deletions in logs: failed calls include an `errorCode` field.

The `S3` events appeared in the `af-south-1` region folder because S3 is regional.

### What I Learned
- The five fields every CloudTrail event contains: `userIdentity` (who), `eventName` (what), `eventTime` (when), `sourceIPAddress` (where from), `requestParameters` (what inputs). These five fields answer every forensic question.
- `responseElements: null` does not mean failure — it means the action succeeded but produced no return object (common for destructive operations). Failure shows up as `errorCode` and `errorMessage`.
- Global service events in `us-east-1` is not just a theoretical concept — it is immediately visible in the actual S3 folder structure from a real lab. The IAM events are in a different folder from the S3 events.

---

## Lab 4: Document Incident Response Process

### What 5 Fields Does Every CloudTrail Event Contain?

1. `userIdentity` — WHO made the call (IAM user, role, service, root)
2. `eventName` — WHAT action was called (`DeleteBucket`, `CreateUser`, `RunInstances`)
3. `eventTime` — WHEN the action occurred (UTC timestamp)
4. `sourceIPAddress` — WHERE the call originated (IP address)
5. `requestParameters` — WHAT inputs were passed (resource names, IDs, configuration)

### How to Use CloudTrail for Incident Response

**Step 1 — Establish the timeline.**
Filter CloudTrail events by the timeframe of the suspected incident. Start broad (±24 hours around the suspected event) and narrow based on what you find. Use CloudTrail Lake or Athena on S3 if the volume is large — do not manually read individual `.gz` files for a multi-day incident.

**Step 2 — Identify the principal.**
Find the `userIdentity.arn` of whoever triggered the suspicious action. Check whether it is a known IAM user, an assumed role, or an external service. If it is an assumed role, trace back to the session that assumed it — the `sessionContext` field contains the source principal.

**Step 3 — Scope the blast radius.**
Once you have the principal ARN, search all events by that principal within the incident window. Look for: IAM changes (creating new users, creating access keys, attaching policies), data access events (if Data Events are enabled), and any `cloudtrail:StopLogging` attempts. The absence of a `StopLogging` success event means the attacker could not disable logging — your SCP from Day 3 would have blocked that.

---

## Bonus Lab: ConsoleLogin Investigation in Event History

### Steps
1. Go to **CloudTrail → Event History** in the console sidebar
2. Click the **Filter** dropdown → select **Event name**
3. Type `ConsoleLogin` and press Enter
4. Scan the results — find your own most recent login
5. Click on the event to expand it
6. Note the `sourceIPAddress` field — confirm it matches your known IP

### What I Observed
My own `ConsoleLogin` events appeared with my Lagos IP address in the `sourceIPAddress` field. The events showed `userIdentity.type: "IAMUser"` and the full ARN of my admin user.

One thing I noticed: `ConsoleLogin` events appear in `us-east-1` in the Event History view regardless of which region you are actually logged into the console in. Console logins are global authentication events, not regional ones.

### What I Learned
- If you opened Event History and saw a `ConsoleLogin` from an IP you do not recognise — especially a foreign country — that is a credential compromise signal requiring immediate action: rotate credentials, revoke sessions, and investigate every subsequent event from that principal.
- `ConsoleLogin` events are a management event (no extra cost) that appear in Event History immediately — they are among the fastest CloudTrail events to appear in the console, often within 2–3 minutes.
- This is exactly the Metric Filter target for the root login alert from the concept notes: `{ $.userIdentity.type = "Root" && $.eventName = "ConsoleLogin" }`. Filter on this, wire to SNS, get an immediate alert any time root is used.
