# IAM Policies — Exam Prep — Week 2 Day 2

---

## SAA-C03 Context

IAM Policies appear throughout the exam — not just in IAM-specific questions. Policy evaluation logic shows up in EC2 questions (instance roles), S3 questions (bucket policies vs identity policies), and architecture questions (least privilege design). Know this topic from multiple angles.

---

## Policy Evaluation Decision Tree

When AWS evaluates an API request, work through this order:

```
1. Explicit Deny in any policy (SCP, identity, resource)?
   → DENIED. Stop.

2. SCP present? Does it allow the action?
   → If SCP denies or does not allow → DENIED. Stop.

3. Identity-based policy allows the action?
   → If yes, check resource-based policy (if applicable).

4. For cross-account access: both identity-based AND resource-based policy must allow.
   → If either is missing → DENIED.

5. No matching Allow anywhere?
   → DENIED (implicit/default deny).
```

For same-account access (most exam questions): a single identity-based policy Allow is sufficient if no Deny is present anywhere.

---

## The Key Policy Distinctions to Know

| Concept | What to Remember |
|---|---|
| Explicit Deny vs Implicit Deny | Explicit Deny cannot be overridden. Implicit Deny can be resolved by adding an Allow. |
| SCP vs IAM Policy | SCP sets the ceiling. IAM policy works within that ceiling. SCP does not grant — it only restricts. |
| Managed vs Inline | Managed = reusable, versioned, auditable. Inline = tied to one identity, not reusable, avoid. |
| Identity-based vs Resource-based | Identity-based attaches to the user/role. Resource-based attaches to the resource (S3 bucket, etc.). For cross-account: need both. |
| `Resource: *` | Applies the action to ALL resources of that type. Dangerous in Allow; sometimes correct in Deny. |

---

## Practice Question — Week 2 Day 2

**Scenario:** A developer at a Nigerian fintech company has an IAM identity-based policy granting `s3:PutObject` on `Resource: *` (all S3 buckets). The company's AWS Organizations SCP explicitly denies `s3:PutObject` on the production bucket. What happens when the developer tries to upload a file to the production bucket?

**A.** Access is denied. An SCP explicit Deny overrides all IAM identity-based Allow policies.

**B.** Access is granted. IAM identity-based policies are evaluated before SCPs, so the Allow takes effect first.

**C.** Access is granted. Using `Resource: *` in an IAM policy grants unrestricted access that cannot be blocked by other policies.

**D.** Access is denied only if the production bucket also has a resource-based bucket policy that explicitly denies the action.

**Answer: A**

**Why B is wrong:** Evaluation order does not determine who wins — explicit Deny always wins regardless of order. Evaluation order matters for understanding the flow, not for overriding Deny.

**Why C is wrong:** `Resource: *` in an Allow means the permission is not scoped to a specific resource. It does not make the Allow unblockable. An explicit Deny anywhere still overrides it.

**Why D is wrong:** A resource-based bucket policy is not needed for an SCP Deny to take effect. SCPs operate at the account level and override IAM policies without needing a corroborating resource policy.

---

## Scenario Table — Which Policy Type?

| Situation | Correct Approach |
|---|---|
| All developers in the account should never be able to delete production resources | SCP on the AWS Organizations OU containing the production account |
| A single developer needs temporary upload access to one S3 bucket | Customer managed policy with scoped Resource ARN, attached directly or via group |
| An EC2 instance needs to read from S3 | IAM Role with customer managed or AWS managed policy attached to the role |
| You need to allow another AWS account to read objects from your S3 bucket | Resource-based bucket policy on the S3 bucket granting access to the other account's principal |
| You want to prevent any IAM action from running without MFA | Customer managed policy with a Deny on all IAM actions when `aws:MultiFactorAuthPresent` is false |

---

## Policy JSON Recall Test

Read this policy and answer the questions below without looking at your notes:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2List",
      "Effect": "Allow",
      "Action": "ec2:DescribeInstances",
      "Resource": "*"
    },
    {
      "Sid": "DenyStopInProd",
      "Effect": "Deny",
      "Action": "ec2:StopInstances",
      "Resource": "arn:aws:ec2:af-south-1:123456789012:instance/*",
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Environment": "production"
        }
      }
    }
  ]
}
```

Questions:
1. Can a user with this policy list all EC2 instances in the account?
2. Can they stop a development EC2 instance tagged `Environment: development`?
3. Can they stop a production EC2 instance tagged `Environment: production`?
4. Can they launch a new EC2 instance?
5. If another policy grants `ec2:StopInstances` on `Resource: *`, can they stop a production instance?

Answers:
1. Yes — Statement 1 explicitly allows `ec2:DescribeInstances` on all resources.
2. Yes — Statement 2 Deny only applies when the tag is `production`. The development instance is not covered by the Deny, and there is no explicit Allow for StopInstances (only DescribeInstances), so this would be implicitly denied unless another policy grants it.
3. No — Statement 2 explicitly Denies `ec2:StopInstances` on instances tagged `Environment: production`.
4. No — `ec2:RunInstances` is not in any Allow statement. Default deny applies.
5. No — An explicit Deny always overrides any Allow, regardless of Resource scope in the Allow statement.
