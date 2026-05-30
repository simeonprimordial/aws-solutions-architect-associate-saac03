# IAM Policy Labs — Week 2 Day 2

---

## Lab 1: Read a Managed Policy First

### Steps
1. Go to **IAM → Policies**
2. Search for `AmazonS3ReadOnlyAccess` — click on it
3. Click the **JSON** tab
4. Read the policy carefully. Identify: Effect, Action, Resource
5. Note what the wildcards (`*`) mean in this context

### The Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:Get*",
        "s3:List*",
        "s3:Describe*",
        "s3-object-lambda:Get*",
        "s3-object-lambda:List*"
      ],
      "Resource": "*"
    }
  ]
}
```

### What I Observed
The `*` wildcards in the Action field cover all Get, List, and Describe operations under S3 — not just `s3:GetObject` and `s3:ListBucket`. This is broader than I expected. It also applies to `Resource: *`, meaning it covers every S3 bucket in the account, not just specific ones.

The policy has no Deny statement. It also has no Condition block — access is not restricted by IP, Region, or MFA status.

### What I Learned
- AWS managed policies are written broadly by design — they are intended to be general-purpose. In a real environment, you often need a customer managed policy with tighter resource scoping.
- The action wildcards (`s3:Get*`) are a shorthand that covers any current and future S3 Get-type actions. This is convenient but means the policy auto-expands if AWS adds new Get actions — something to be aware of in security-sensitive environments.
- Reading a managed policy before writing your own is good practice — it shows you what AWS considers the baseline for a given access pattern, and you can decide whether to tighten or adapt it.

---

## Lab 2: Write a Custom Policy from Scratch

### Steps
1. Go to **IAM → Policies → Create policy → JSON tab**
2. Delete the existing placeholder content
3. Type the following policy **from scratch** (the lab guide specifies no copy-pasting — typing forces you to read every character):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowReadOnlyMyBucketOnly",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-app-bucket-*",
        "arn:aws:s3:::my-app-bucket-*/*"
      ]
    },
    {
      "Sid": "DenyDeleteEverywhere",
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "*"
    }
  ]
}
```

4. Click **Next**
5. Name: `S3-AppBucket-ReadOnly-NoDeletion`
6. Description: `Custom policy — allow read on app buckets, deny all delete operations`
7. Click **Create policy**

### What I Observed
The JSON editor in the IAM console has inline validation — it highlights syntax errors immediately. When I initially missed a closing bracket, the editor flagged it before I could proceed. Useful guardrail.

The two-ARN requirement for S3 caught me on the first attempt — I originally wrote only `arn:aws:s3:::my-app-bucket-*` and the simulator later showed Denied for `s3:GetObject`. Adding the `/*` variant for objects fixed it.

The Deny statement uses a single string for Action (`"s3:DeleteObject"`) rather than an array. Both formats are valid JSON — a single string or an array with one item both work in IAM policies.

### What I Learned
- Typing policies by hand forces you to understand every element. Copy-pasting is faster but you retain less. The muscle memory of `"Effect": "Allow"` helps when reading unfamiliar policies quickly under exam conditions.
- The `Sid` field is optional but worth using — it makes debugging much easier. When the Policy Simulator returns Denied, the Sid tells you which statement is responsible.
- Naming conventions matter. `S3-AppBucket-ReadOnly-NoDeletion` tells any IAM admin exactly what the policy does without opening it. Good naming saves time during incident response.

---

## Lab 3: Attach the Policy and Test in the Policy Simulator

### Steps
1. Go to **IAM → Users → analyst-user → Add permissions → Attach policies directly**
2. Search for `S3-AppBucket-ReadOnly-NoDeletion` — attach it
3. Go to **IAM Policy Simulator** at `policysim.aws.amazon.com`
4. Select entity: `analyst-user`
5. Run three test scenarios:

**Test 1:** `s3:GetObject` on `arn:aws:s3:::my-app-bucket-test/file.txt`
→ Expected: **Allowed** (matches the Allow statement, correct bucket prefix)

**Test 2:** `s3:GetObject` on `arn:aws:s3:::other-bucket/file.txt`
→ Expected: **Denied** (not covered by any Allow — default deny applies)

**Test 3:** `s3:DeleteObject` on `arn:aws:s3:::my-app-bucket-test/file.txt`
→ Expected: **Denied** (explicit Deny in Statement 2 — overrides everything)

6. Screenshot all three results

### What I Observed
All three tests returned the expected results. The simulator shows not just Allowed or Denied — it shows the **specific policy statement and source** responsible for the decision. For Test 3, it explicitly named the `DenyDeleteEverywhere` statement as the reason for denial.

For Test 2, the simulator indicated the denial was an implicit deny — no matching Allow — rather than an explicit Deny. This distinction matters: an implicit deny can be resolved by adding an Allow elsewhere. The explicit Deny in Test 3 cannot.

### What I Learned
- The Policy Simulator is the single best tool for verifying policy logic before applying it in a live environment. It is free and safe — nothing is provisioned, no real API calls are made.
- The explicit Deny for delete applies even to `my-app-bucket-*` objects, even though the Allow statement covers those buckets. The Deny wins. If the exam asks whether an Allow for a resource overrides a Deny on `Resource: *`, the answer is no — Deny always wins regardless of Resource scope.
- Testing edge cases matters: "what happens on the wrong bucket" and "what happens with the denied action" are exactly the kind of scenarios the exam presents. Running them in the simulator trains the right mental model.

---

## Lab 4: Publish to GitHub Gist

### Steps
1. Go to [gist.github.com](https://gist.github.com)
2. Create a new Gist
3. Filename: `s3-readonly-nodelete-policy.json`
4. Paste the full policy JSON
5. Description: `Custom IAM policy: S3 read access scoped to named bucket prefix, with explicit deny on delete operations. Written as part of AWS Cloud Accelerator — Week 2 Day 2.`
6. Set visibility to **Public**
7. Click **Create public gist** — copy and save the URL

### What I Observed
GitHub renders JSON Gists with syntax highlighting automatically. The policy JSON looks clean and readable in the rendered view. The Gist URL is shareable directly — no GitHub account required to view it.

### What I Learned
- Publishing work publicly, even a 20-line JSON file, builds a visible track record. A hiring manager looking at your GitHub can see that you wrote and understood a real IAM policy, not just read about one.
- Gists are permanent and indexed — this is a low-effort way to document portfolio work without needing a full repo for every small artifact.

---

## Bonus Lab: Add an IP Address Condition

### The Modified Policy

Added a Condition block to Statement 1 to restrict read access to requests coming from a specific IP address (your own public IP, retrieved from `whatismyip.com`):

```json
{
  "Sid": "AllowReadOnlyMyBucketOnly",
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:ListBucket"
  ],
  "Resource": [
    "arn:aws:s3:::my-app-bucket-*",
    "arn:aws:s3:::my-app-bucket-*/*"
  ],
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": "YOUR_PUBLIC_IP/32"
    }
  }
}
```

The `/32` CIDR notation means exactly one IP address. To allow a range (e.g. an office network), you would use something like `197.210.0.0/24`.

### What I Observed
The Policy Simulator does not always evaluate IP conditions correctly when running from the console — it may show the condition as not applicable. To test IP conditions properly, you need to make actual API calls from the restricted IP and observe the result, or use the simulator's "Context Entries" section to manually set the source IP value.

### What I Learned
- Conditions dramatically increase policy precision. Restricting S3 access to a known office IP range is a real security control used in corporate environments — it means even if credentials are leaked, the attacker cannot use them from an unknown IP.
- The `/32` notation is worth knowing: it means a single host. This is a subnet mask concept from networking that appears regularly in AWS security contexts (Security Groups, NACLs, IAM conditions).
- Always test Condition-based policies in the simulator with the Context Entries section filled in — otherwise the simulator evaluates without the condition context and the result may be misleading.
