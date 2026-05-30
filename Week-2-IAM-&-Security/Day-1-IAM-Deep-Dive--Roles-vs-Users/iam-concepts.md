# IAM Deep Dive — Day 1 Notes

---

## Why IAM Matters

IAM (Identity and Access Management) is the security backbone of every AWS account. It controls **who can do what** across all AWS services and resources. It is a global service — users, groups, roles, and policies you create are available across all Regions automatically with no extra configuration needed.

IAM sits at the centre of Domain 1 (Design Secure Architectures), which is **30% of the SAA-C03 exam**. No other single topic carries as much weight. Master IAM now.

---

## Core IAM Identities

### Root Account
The identity created automatically when you first sign up for AWS using your email address.

- Has **unrestricted access** to everything in the account — billing, IAM, all services
- Cannot be restricted by any IAM policy, not even AWS Organizations SCPs
- **What to do:** Enable MFA immediately. Create a separate IAM admin user for daily work. Never use root again.

> ⚠️ **Exam Tip:** If a question asks what has unlimited, unrestricted access to an AWS account — the answer is always the root account.

---

### IAM User
An identity representing a **person or application** that needs access to AWS resources.

- Has **no permissions by default** — you must explicitly assign them
- Carries **permanent credentials**: a password (for console access) and/or access keys (for CLI/API access)
- One person = one IAM User — never share credentials between people

**When to use:** A human needs to log into the AWS Console or run CLI commands regularly.

---

### IAM Group
A collection of IAM Users that share the same permissions.

- Policies are attached to the **group**, not individual users
- Users **inherit all permissions** from every group they belong to
- A user can belong to multiple groups
- One policy change at the group level instantly updates permissions for all members

**Best practice:** Always assign permissions via groups, not directly to individual users. Scaling a team becomes painless — add a user to a group and they're set up in seconds.

---

### IAM Policy
A JSON document that defines **what actions are allowed or denied** on which resources.

Each policy statement contains:
- `Effect` — Allow or Deny
- `Action` — the specific API call(s) (e.g. `s3:GetObject`, `ec2:RunInstances`)
- `Resource` — the specific AWS resource(s) the rule applies to (ARN or `*` for all)

**Types of policies:**
- **AWS Managed Policies** — pre-built by AWS (e.g. `AmazonS3ReadOnlyAccess`, `ReadOnlyAccess`)
- **Customer Managed Policies** — custom policies you create and control
- **Inline Policies** — attached directly to a single user, group, or role (not reusable — avoid these)

> ⚠️ **Exam Tip:** An explicit **Deny** always overrides an explicit **Allow** in IAM — regardless of how many policies grant access.

---

### Least Privilege
The principle of granting only the **minimum permissions** required for a task — nothing more, nothing less.

In practice: start with no permissions and add only what the user or service actually needs. Do not assign `AdministratorAccess` by default to save time. That shortcut is a common security mistake in real environments and a guaranteed wrong answer on the exam.

---

## IAM Roles — The Other Identity

A Role is fundamentally different from a User. Understanding this distinction is the most-tested concept in IAM.

### What is a Role?
A **temporary identity** that can be assumed by an AWS service, an application, or a user — to perform actions they are authorised for.

- Has **no password**
- Has **no long-term access keys**
- Issues **temporary credentials** via STS (Security Token Service) that expire automatically

### Trust Policy
Every Role has a Trust Policy. This answers the question: **who is allowed to assume this role?**

Example trust policy for an EC2 role:
```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "ec2.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}
```

### Permissions Policy
Also attached to every Role. This answers: **what can this role do once assumed?**

Together, Trust Policy + Permissions Policy = a complete Role definition.

### How Temporary Credentials Work
When an EC2 instance assumes a role, AWS STS issues three things:
- An access key ID
- A secret access key
- A session token

These expire after a set duration (default 1 hour, max 12 hours for EC2). The application on the instance retrieves them automatically from the **instance metadata service** at `169.254.169.254` — no developer ever touches the keys directly. This is why roles are far more secure than long-term access keys stored in code.

---

## Users vs Roles — The Core Distinction

| Feature | IAM User | IAM Role |
|---|---|---|
| Credentials | Permanent (password + access keys) | Temporary (STS-issued, auto-expire) |
| Used for | Human operators | AWS services, applications |
| Has a password | Yes | No |
| Has long-term access keys | Yes (optional) | No |
| Example use case | Developer logging into console | EC2 instance reading from S3 |

> ⚠️ **Exam Trap:** The exam will offer "create an IAM User with access keys for the EC2 instance" as a plausible-sounding answer. It is always wrong. The correct answer is always: **attach an IAM Role to the service**.

---

## The IAM Permission Flow

```
Root Account (locked with MFA)
    └── IAM Admin User (your daily driver)
         ├── IAM Groups
         │    ├── Developers → AmazonEC2FullAccess + AWSLambdaFullAccess
         │    ├── Analysts → AmazonS3ReadOnlyAccess + AmazonAthenaFullAccess
         │    └── Managers → ReadOnlyAccess (all services)
         │
         └── IAM Roles
              └── EC2-S3-ReadOnly-Role
                   ├── Trust Policy → ec2.amazonaws.com can assume this role
                   └── Permissions Policy → AmazonS3ReadOnlyAccess
```

---

## Common Exam Traps

- **Root cannot be restricted.** No IAM policy, SCP, or permission boundary can limit the root account. Lock it with MFA and walk away.
- **IAM is global.** Users, groups, roles, and policies are not Region-specific. The resources they access (EC2, RDS) may be Regional, but IAM itself is not.
- **Never hardcode access keys.** Not in application code, not in environment variables on an EC2 instance, not in user data scripts. Always attach an IAM Role instead.
- **Inline policies are not best practice.** They cannot be reused, versioned, or audited easily. Always use managed or customer-managed policies.
- **Groups cannot contain other groups.** IAM Groups are flat — you cannot nest a group inside another group.
