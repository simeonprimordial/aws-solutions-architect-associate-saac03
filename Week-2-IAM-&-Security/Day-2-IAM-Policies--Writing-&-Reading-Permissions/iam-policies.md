# IAM Policies — Writing & Reading Permissions — Day 2 Notes

---

## Why Policies Matter

IAM Users, Groups, and Roles are just identities — empty containers with no access by default. Policies are what actually grant or deny access. Every single action in AWS — creating an EC2 instance, reading a file from S3, calling a Lambda function — is checked against IAM policies before it executes. No policy? Access denied. Wrong policy? Access denied. This check is automatic, instant, and happens at the AWS API level before any resource is touched.

Understanding how to read, write, and reason about policies is a core skill. You do not need to memorise every possible policy document, but you do need to:
- Read a policy JSON and know exactly what it allows and blocks
- Understand the difference between managed and inline policies
- Know how AWS evaluates multiple overlapping policies
- Recognise the correct policy for a given scenario on the exam

---

## The Policy JSON Structure

Every IAM policy is a JSON document. The structure is always the same:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "OptionalStatementId",
      "Effect": "Allow",
      "Action": ["service:ActionName"],
      "Resource": ["arn:aws:service:::resource"],
      "Condition": {}
    }
  ]
}
```

### `Version`
Always `"2012-10-17"`. This is the current policy language version. Do not change it.

### `Statement`
An array of one or more permission blocks. Each block is evaluated independently.

### `Sid` (Statement ID)
Optional. A label you choose to identify the statement — useful for documentation and debugging. Has no effect on evaluation.

### `Effect`
**Allow** or **Deny**. There is no middle ground. If Effect is missing, the statement is invalid.

### `Action`
The specific AWS API call being controlled. Format is always `service:ActionName`.

Common examples:
- `s3:GetObject` — read a file from S3
- `s3:PutObject` — upload a file to S3
- `s3:DeleteObject` — delete a file from S3
- `s3:ListBucket` — list objects in a bucket
- `ec2:RunInstances` — launch an EC2 instance
- `ec2:DescribeInstances` — list EC2 instances (read-only)
- `iam:CreateUser` — create a new IAM user
- `s3:*` — all S3 actions (wildcard)
- `*` — all actions on all services (never use this except for admin)

### `Resource`
The ARN (Amazon Resource Name) of the specific resource the action applies to. Format varies by service.

S3 ARN examples:
- `arn:aws:s3:::my-bucket` — the bucket itself
- `arn:aws:s3:::my-bucket/*` — all objects inside the bucket
- `arn:aws:s3:::my-app-bucket-*` — all buckets whose name starts with `my-app-bucket-`
- `*` — all resources

> ⚠️ **S3 Gotcha:** For most S3 actions you need TWO resource ARNs in the same statement — one for the bucket and one for the objects. Missing either one breaks the policy silently — the simulator will show Denied and you will spend time debugging.

```json
"Resource": [
  "arn:aws:s3:::my-bucket",
  "arn:aws:s3:::my-bucket/*"
]
```

### `Condition`
Optional. Adds constraints that must be met for the Effect to apply. Examples:

Restrict to a specific IP address:
```json
"Condition": {
  "IpAddress": {
    "aws:SourceIp": "197.210.0.0/16"
  }
}
```

Require MFA to be active:
```json
"Condition": {
  "Bool": {
    "aws:MultiFactorAuthPresent": "true"
  }
}
```

Restrict to a specific AWS Region:
```json
"Condition": {
  "StringEquals": {
    "aws:RequestedRegion": "af-south-1"
  }
}
```

Conditions make policies far more granular. For the exam, know that they exist and understand the IP and MFA examples above.

---

## Types of IAM Policies

### AWS Managed Policies
Pre-built by AWS. Maintained and updated by AWS as new services and actions are added.

- Ready to attach immediately — no JSON writing required
- Cannot be edited — you get the policy as AWS defines it
- Examples: `AmazonS3ReadOnlyAccess`, `AmazonEC2FullAccess`, `ReadOnlyAccess`, `AdministratorAccess`

**When to use:** For common, well-defined access patterns where the AWS-provided scope is acceptable.

### Customer Managed Policies
Created and managed by you. Stored in your account and reusable across any number of users, groups, or roles.

- Full control over the JSON
- Versioned — AWS keeps the last five versions; you can roll back
- Can be audited and tracked in CloudTrail

**When to use:** When AWS managed policies are too broad or too narrow for your specific use case. This is the correct pattern for production environments.

### Inline Policies
JSON embedded directly into a single IAM user, group, or role. Not stored separately.

- Not reusable — deleting the identity deletes the policy with it
- Not versionable
- Harder to audit across the account
- If you attach the same inline policy to 10 users and need to change it, you must edit all 10 individually

**When to use:** Almost never. The only legitimate use case is when you explicitly want a policy that dies with the identity it is attached to. For everything else, use a customer managed policy.

> ⚠️ **Exam Tip:** If a question asks about best practices for policy management, inline policies are always the wrong answer.

---

## Policy Evaluation Logic

When you make an API call in AWS, the following happens in order:

```
1. Is there an explicit Deny anywhere in ANY applicable policy?
   → YES: ACCESS DENIED. Stop. No further evaluation.

2. Is there an explicit Allow in at least one applicable policy?
   → YES: ACCESS GRANTED.

3. Neither?
   → ACCESS DENIED (implicit deny / default deny).
```

"Applicable policies" includes all of the following evaluated together:
- SCPs from AWS Organizations (set the ceiling — if SCP denies it, nothing below can allow it)
- Identity-based policies on the user, group, or role
- Resource-based policies on the target resource (e.g. S3 bucket policy)
- Permission boundaries (advanced — not covered today)

### The Three Policy Layers

| Layer | Who Sets It | What It Controls |
|---|---|---|
| Service Control Policies (SCPs) | AWS Organizations admin | Maximum permissions across the entire account |
| Identity-Based Policies | IAM admin | What a specific user/group/role can do |
| Resource-Based Policies | Resource owner | Who can access a specific resource (cross-account use case) |

A key distinction: SCPs are **guardrails**, not grants. They do not grant permissions — they only limit what is possible. Even if you have an IAM policy that allows `s3:DeleteBucket`, if an SCP denies it, the action is blocked. Full stop.

---

## Reading a Real Policy — Worked Example

This is the custom policy written in today's lab:

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

Reading this statement by statement:

**Statement 1 — AllowReadOnlyMyBucketOnly:**
- `Effect: Allow` — this is a grant
- `Action: s3:GetObject + s3:ListBucket` — read files and list bucket contents
- `Resource: my-app-bucket-*` — only buckets whose name starts with `my-app-bucket-`. Any other bucket is not covered by this Allow

What this user **cannot** do with this statement alone:
- Cannot upload files (`s3:PutObject` is not listed)
- Cannot delete files (`s3:DeleteObject` is not listed here, and is explicitly denied below)
- Cannot access any bucket not named `my-app-bucket-*`
- Cannot access EC2, IAM, RDS, or any other service

**Statement 2 — DenyDeleteEverywhere:**
- `Effect: Deny` — this is a block
- `Action: s3:DeleteObject` — delete any S3 object
- `Resource: *` — on ALL buckets and objects

Even if another policy (from a group, or a future policy) grants `s3:DeleteObject`, this explicit Deny overrides it. The user can never delete an S3 object as long as this policy is attached to them.

This is the power of combining an explicit Deny with `Resource: *` — it creates an absolute restriction that cannot be bypassed by other attached policies.

---

## Common Exam Traps

- **Explicit Deny always wins.** An SCP that denies `s3:DeleteBucket` blocks the action for everyone in the account — including the account's own IAM admin users. Only the root account is exempt from SCPs.
- **`Resource: *` is dangerous.** Using `*` for resource in an Allow statement means the permission applies to every resource in that service. Always scope to the minimum ARN.
- **Inline policies are not best practice.** They exist but should be avoided. The exam will test whether you know when managed policies are more appropriate.
- **S3 needs two ARNs.** `arn:aws:s3:::bucket-name` covers the bucket. `arn:aws:s3:::bucket-name/*` covers the objects. You need both for most S3 operations.
- **Default deny is not the same as explicit deny.** Default deny (no matching Allow) can be overridden by adding an Allow elsewhere. Explicit deny (`Effect: Deny`) cannot be overridden by anything except removing the Deny statement itself.
