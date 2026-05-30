# IAM Exam Prep — Week 2 Day 1

---

## SAA-C03 Context

IAM falls under **Domain 1: Design Secure Architectures** — the largest single domain at **30% of the exam**. IAM questions appear across all domains in different forms: security questions test IAM directly, compute questions test how EC2/Lambda use roles, data questions test S3 and RDS access patterns. There is no part of the exam where IAM knowledge is irrelevant.

---

## Most-Tested IAM Concepts

### 1. User vs Role — Know This Cold

| Scenario | Correct Choice |
|---|---|
| A developer needs daily console and CLI access | IAM User |
| An EC2 instance needs to write to S3 | IAM Role attached to EC2 |
| A Lambda function needs to query DynamoDB | IAM Role attached to Lambda |
| A mobile app needs temporary AWS access | IAM Role via Cognito Identity Pools |
| Another AWS account needs cross-account access | IAM Role with cross-account Trust Policy |
| An on-premises server needs to call AWS APIs | IAM Role via AWS STS AssumeRoleWithWebIdentity or IAM User (last resort) |

### 2. Never Hardcode Access Keys
The exam will present hardcoded credentials as an option. It is always wrong. The correct answer is always to use an IAM Role so that STS handles credential issuance and rotation automatically.

### 3. Least Privilege Is Always the Right Default
When the exam asks what permissions to grant, the answer is always the minimum required. Assigning `AdministratorAccess` to solve a permissions problem is never the correct production answer.

### 4. Explicit Deny Always Wins
If any policy attached to a user or role contains an explicit Deny for an action, that deny overrides all Allow statements — regardless of how many policies grant access.

### 5. IAM Is Global
IAM users, groups, roles, and policies exist globally across all Regions in your account. However, the resources they access (EC2 instances, RDS databases, S3 buckets in specific regions) may be Regional. Don't confuse IAM scope with resource scope.

---

## Practice Question — Week 2 Day 1

**Scenario:** A developer at a Lagos fintech company has deployed a Python application on an EC2 instance. The application needs to read files from S3 and write logs to CloudWatch. The developer created an IAM User, generated access keys, and pasted them into the application code on the server. A senior engineer flags this as a security violation. What is the correct solution?

**A.** Encrypt the access keys using AWS KMS before storing them in the application code.

**B.** Store the IAM User access keys in a `.env` file on the EC2 instance with restricted file permissions.

**C.** Remove the access keys. Create an IAM Role with S3 read and CloudWatch write permissions, then attach the role to the EC2 instance. The application retrieves temporary credentials automatically.

**D.** Create a new IAM User for the EC2 instance, assign it to a group with the required policies, and rotate the access keys every 30 days using a Lambda function.

**Answer: C**

**Why A is wrong:** Encrypting hardcoded credentials is still storing long-term credentials on the instance. KMS protects data at rest, not credential management.

**Why B is wrong:** A `.env` file with restricted permissions is still long-term credentials on disk. If the instance is compromised, the keys are compromised.

**Why D is wrong:** Rotating keys on a schedule reduces the window of exposure but does not eliminate the fundamental problem. IAM Users still have long-term credentials. D adds complexity without solving the root issue.

**Why C is correct:** Roles use STS-issued temporary credentials that expire automatically. No credentials are stored anywhere on the instance. If the instance is compromised, the stolen credentials expire within hours and cannot be renewed without re-assuming the role.

---

## Key Policies to Know by Name

| Policy Name | What It Does |
|---|---|
| `AdministratorAccess` | Full access to all AWS services and resources |
| `ReadOnlyAccess` | Read-only access to all AWS services |
| `AmazonEC2FullAccess` | Full EC2 management |
| `AmazonS3ReadOnlyAccess` | Read-only S3 access (list + get, no write/delete) |
| `AmazonS3FullAccess` | Full S3 access including delete |
| `AWSLambdaFullAccess` | Full Lambda management |
| `AmazonAthenaFullAccess` | Full Athena query access |
| `IAMFullAccess` | Full IAM management — grant with extreme caution |

---

## Two-Sentence Recall Test

Before you close this file, answer these without looking:

1. What is the difference between an IAM User and an IAM Role in one sentence?
2. What are the two policies attached to every IAM Role, and what does each one control?
3. What AWS service issues temporary credentials when a role is assumed?
4. Why can the root account not be restricted by any IAM policy?
5. Where does an EC2 instance automatically retrieve role credentials from at runtime?

If you can answer all five without hesitation, you're ready for the IAM section of the exam.
