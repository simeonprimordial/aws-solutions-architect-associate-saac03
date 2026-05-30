# Challenges & How I Solved Them — Week 2 Day 2

This file tracks blockers I encountered during labs and how I resolved them. Documenting this helps me learn from mistakes and may help others hitting the same issues.

---

## Challenge 1: S3 ListBucket Returning Denied Despite Correct Allow

**What happened:**
After writing the custom policy and running the simulator, `s3:ListBucket` returned Denied even though it was explicitly listed in the Allow Action array.

**What I tried:**
- Double-checked the Action spelling — `s3:ListBucket` was correct
- Re-read the Resource block — found the issue: I only had `arn:aws:s3:::my-app-bucket-*/*` (the objects ARN) but was missing `arn:aws:s3:::my-app-bucket-*` (the bucket-level ARN)

**Resolution:**
`s3:ListBucket` is a **bucket-level** action, not an object-level action. It requires the bucket ARN without the trailing `/*`. Adding `arn:aws:s3:::my-app-bucket-*` as a second resource entry resolved the issue.

**Lesson learned:**
S3 actions split across two levels — bucket actions (ListBucket, GetBucketLocation) need the bucket ARN; object actions (GetObject, PutObject, DeleteObject) need the `/*` objects ARN. Always include both in policies that mix bucket and object actions.

---

## Challenge 2: Policy Simulator Showed "No Policies" for analyst-user

**What happened:**
After creating the custom policy and navigating to the simulator, selecting `analyst-user` showed no applicable policies.

**What I tried:**
- Refreshed the Policy Simulator page — no change
- Went back to IAM → Users → analyst-user → Permissions tab — confirmed the custom policy was listed as attached

**Resolution:**
The simulator page had loaded before I attached the policy. A hard browser refresh (Ctrl+Shift+R / Cmd+Shift+R) reloaded the simulator with the updated policy list.

**Lesson learned:**
The IAM Policy Simulator caches identity data on load. After attaching or modifying a policy, do a hard refresh before running simulations to ensure you are testing the current state.

---

## Challenge 3: JSON Syntax Error Blocking Policy Creation

**What happened:**
When typing the custom policy JSON, the console showed a red error banner and would not let me proceed to the next step.

**What I tried:**
- Read the error message — it said "Invalid JSON: Unexpected token at position 147"
- Counted brackets manually — found a missing comma between the two Statement objects in the array

**Resolution:**
Each Statement object in the array must be separated by a comma. The last Statement object should have no trailing comma. Fixed the syntax and the editor accepted it immediately.

**Lesson learned:**
JSON is unforgiving — a single missing comma or bracket breaks the entire document. When writing policies manually:
- Use a linter or paste into [jsonlint.com](https://jsonlint.com) before submitting
- The IAM console JSON editor shows the position of the error — use it to locate the exact line
- Common JSON mistakes: missing comma between objects, trailing comma after the last item, missing closing bracket

---

## Challenge 4: Bonus Lab — Policy Simulator Not Evaluating IP Condition

**What happened:**
After adding the IP Condition block to the Allow statement, the simulator showed Allowed regardless of what IP I was testing from. Expected it to show Denied from unlisted IPs.

**What I tried:**
- Searched AWS docs for "policy simulator condition evaluation" — found that the simulator requires manual context entries for conditions to be evaluated
- Located the **Context Entries** section in the simulator (bottom of the left panel)
- Added key `aws:SourceIp` with the value of my public IP

**Resolution:**
Setting the `aws:SourceIp` context entry in the simulator caused the condition to be evaluated. Testing with my own IP showed Allowed; changing the IP to a different value showed Denied.

**Lesson learned:**
The IAM Policy Simulator does not automatically inject real context values (IP address, MFA status, current time, etc.) — you must provide them manually in the Context Entries section. If you run a condition-based policy simulation without the relevant context entry, the condition is treated as not applicable and the simulation result is unreliable.

---

*Add new challenges here as they come up in future days.*
