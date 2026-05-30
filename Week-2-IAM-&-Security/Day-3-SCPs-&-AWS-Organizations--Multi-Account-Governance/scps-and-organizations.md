# SCPs & AWS Organizations — Day 3 Notes

---

## Why Multi-Account Architecture Matters

When a company is small, one AWS account is fine. When it grows — multiple teams, production and dev environments, regulatory requirements, separate billing — a single account becomes unmanageable. IAM policies can only go so far: you cannot easily prevent a developer in a shared account from accidentally touching production resources, and you cannot isolate the blast radius of a security incident.

AWS Organizations solves this by making multi-account management a first-class feature. Instead of treating each account as an island, Organizations connects them into a hierarchy with centralised billing, centralised governance, and centralised policy enforcement. SCPs are the enforcement arm of that governance.

The Nigerian context is directly relevant here. The Central Bank of Nigeria (CBN) requires financial institutions to maintain segregated environments, immutable audit logs, and data residency controls. AWS Organizations + SCPs is how compliant multi-account architectures implement these requirements in practice.

Think of it this way: AWS Organizations is the holding company. Each subsidiary (business unit) is a separate AWS account. SCPs are the group-wide compliance rules the holding company imposes — no subsidiary can override them, no matter what their internal policies say.

---

## AWS Organizations — Core Components

### Management Account
The AWS account that creates the organisation. Also called the payer account because it owns consolidated billing for all member accounts.

- Full control over the organisation: can create OUs, invite or remove accounts, attach or detach SCPs
- **Never restricted by any SCP** — not even an SCP attached to the Root. This is by design and cannot be changed
- Because of this exemption, the Management Account should be used **only** for billing and organisational management — never for running workloads

### Root
The top-level container created automatically when you enable Organizations. It sits above all OUs and accounts.

- SCPs attached to the Root apply to every OU and every account in the entire organisation
- Root-level SCPs are the right place for organisation-wide non-negotiable controls: prevent leaving the org, protect audit logging, enforce MFA requirements

### Organisational Unit (OU)
A logical grouping container for AWS accounts. You build a hierarchy by nesting OUs inside each other and placing accounts at any level.

- SCPs attached to an OU automatically apply to every account inside it and every nested OU beneath it
- Accounts inherit the SCPs of their parent OU, the parent's parent, and so on all the way to the Root
- Inheritance is **additive and cumulative** — an account at the bottom of a three-level hierarchy is restricted by all SCPs from all levels above it

### Member Accounts
Individual AWS accounts that belong to the organisation. Each one operates independently (its own IAM, resources, billing data) but is subject to the SCPs of every OU and Root above it.

- Can be invited to join or created fresh from within Organizations
- The root user of a member account **is** restricted by SCPs, unlike IAM policies which cannot restrict account root users

---

## Service Control Policies (SCPs)

### What SCPs Are
JSON policy documents — same syntax as IAM policies — attached to the Root, an OU, or an individual account. They define the **maximum permissions ceiling** for every principal in the target.

Key properties:
- SCPs **never grant permissions**. They only restrict. Even an SCP that says `"Effect": "Allow"` on an action does not grant that action — it merely declares that IAM is permitted to grant it. The actual Allow must still come from an IAM policy.
- The **effective permission** for any action is the intersection: what the SCP allows AND what the IAM policy allows. Remove either and access is denied.
- One explicit Deny in an SCP overrides every Allow in every IAM policy in the account — including `AdministratorAccess`

### The FullAWSAccess Default
When you first enable SCPs in Organizations, AWS automatically attaches a policy called `FullAWSAccess` to every root, OU, and account. This policy allows all actions on all services — it is completely permissive.

This means enabling Organizations does not automatically restrict anything. You start from a position of full access and add restrictions on top. This is the Deny List strategy.

### SCP Inheritance — How It Flows

```
Root (Management Account — never affected)
│   Root SCP: DenyDisableCloudTrail + DenyLeaveOrganization
│   (applies to ALL OUs and accounts below)
│
├── Production OU
│   │   OU SCP: DenySpotInstances + DenyUntaggedResources
│   │
│   ├── prod-backend account
│   │   (inherits Root SCPs + Production OU SCPs)
│   └── prod-database account
│       (inherits Root SCPs + Production OU SCPs)
│
├── Development OU
│   │   OU SCP: DenyReservedInstances + DenyBillingChanges
│   │
│   ├── dev-team account
│   │   (inherits Root SCPs + Development OU SCPs)
│   └── staging account
│       (inherits Root SCPs + Development OU SCPs)
│
└── Security OU
    │   OU SCP: AllowReadOnlyAndCloudTrailOnly
    │
    ├── audit-log account
    │   (inherits Root SCPs + Security OU SCPs)
    └── security-tools account
        (inherits Root SCPs + Security OU SCPs)
```

An account at any level is restricted by the **combined effect of all SCPs from every layer above it**. There is no way for an account to opt out of an SCP from a parent OU.

---

## The Two SCP Strategies

### Deny List Strategy (Recommended Default)
Keep `FullAWSAccess` attached at every level. Add explicit Deny statements for anything you want to forbid. Everything not explicitly denied remains allowed.

**Pros:** Easy to implement and maintain. Only the forbidden actions need documentation. Onboarding a new AWS service does not require updating the SCP.

**Cons:** Relies on knowing what to deny. A gap in the deny list is a gap in governance.

**When to use:** Most organisations starting with SCPs. Any environment where the set of forbidden actions is clearer than the set of allowed ones.

### Allow List Strategy (High-Security)
Remove `FullAWSAccess`. Replace with an explicit Allow for only approved services (e.g. `ec2:*`, `s3:*`, `rds:*`). Anything not listed is implicitly denied.

**Pros:** Maximum control. Nothing can be used unless explicitly approved. Satisfies the most stringent compliance requirements.

**Cons:** High operational overhead. Every new AWS service must be added to the Allow list before anyone in the organisation can use it. A missed addition causes a confusing deny with no obvious cause.

**When to use:** Financial services (CBN compliance, PCI-DSS), healthcare (HIPAA), defence. Environments where the allowed surface area is well-defined and rarely changes.

---

## Reading an SCP — Worked Examples

### Example 1: CloudTrail Protection

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyDisableCloudTrail",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail",
        "cloudtrail:UpdateTrail"
      ],
      "Resource": "*"
    }
  ]
}
```

What this does: Prevents anyone in the target accounts from stopping, deleting, or modifying CloudTrail trails. This includes IAM admins and the member account root user — neither can override an SCP Deny. Attach this at the Root level and it is non-negotiable across the entire organisation.

Why this matters: CloudTrail is the audit log of everything that happens in an AWS account. If an attacker or a rogue insider disables it, you lose the ability to detect, investigate, and prove what happened. This SCP makes audit logging immutable.

### Example 2: Deny Leaving the Organisation

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyLeaveOrganization",
      "Effect": "Deny",
      "Action": "organizations:LeaveOrganization",
      "Resource": "*"
    }
  ]
}
```

What this does: Prevents any member account from removing itself from the organisation. Without this SCP, the root user of any member account could leave the organisation — instantly escaping all SCPs, consolidated billing, and governance controls.

This is one of the first SCPs any security-conscious organisation should attach at the Root level.

### Example 3: Data Residency — Deny Non-Approved Regions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonApprovedRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotIn": {
          "aws:RequestedRegion": [
            "eu-west-1",
            "af-south-1"
          ]
        }
      }
    }
  ]
}
```

What this does: Blocks all AWS API calls unless the requested region is `eu-west-1` (Ireland) or `af-south-1` (Cape Town). A developer who accidentally selects `us-east-1` gets an immediate Access Denied — the API call is rejected before any resource is created. Customer data never leaves CBN-approved regions.

The `StringNotIn` condition key is the right operator here — it reads as "if the requested region is NOT in this list, Deny". Any region outside the approved list triggers the Deny. The two approved regions are exempt.

---

## SCP vs IAM Policy — Comparison

| Feature | SCP | IAM Policy |
|---|---|---|
| What it controls | Maximum permissions for an account/OU | What a specific user/group/role can do |
| Who it applies to | Every principal in the account, including root user of member accounts | Specific IAM users, groups, or roles |
| Can it grant permissions | No — only restricts | Yes — grants and/or denies |
| Where it is applied | Root, OU, or individual account (via Organizations) | User, group, role, or resource |
| Does it affect the Management Account | Never | Yes (IAM policies in the management account apply normally) |
| Does it affect the root user | Yes (member account root only) | No (IAM policies cannot restrict root) |
| What overrides it | Nothing — SCP Deny is absolute for member accounts | An explicit Deny in another policy overrides an Allow |
| JSON syntax | Same as IAM policy | Same as SCP |
| Service that manages it | AWS Organizations | AWS IAM |

> ⚠️ **Exam Tip:** The biggest distinction to drill: SCPs restrict even the root user of member accounts, but IAM policies cannot. This makes SCPs the stronger tool for multi-account governance.

---

## Common Exam Traps

- **Management Account exemption.** The Management Account is never subject to SCPs. If a question asks what an SCP attached to the Root restricts, the answer excludes the Management Account.
- **SCPs do not grant permissions.** If an answer option says "attach an SCP that allows X to grant access", that is wrong. SCPs define the ceiling; IAM grants access within that ceiling.
- **SCPs do restrict member account root users.** This is the key difference from IAM policies. On the exam, if you see a question about restricting account root users across an organisation, the answer is always an SCP, never an IAM policy.
- **FullAWSAccess is the default.** Enabling Organizations does not restrict anything by default. The starting state is fully permissive.
- **An SCP must be attached to take effect.** A well-written SCP sitting unattached in the policy library does nothing. The exam will describe this scenario and ask why the SCP has no effect.
- **Inheritance is cumulative.** An account deep in a nested OU hierarchy inherits SCPs from every level above it. The effective permission at any account is the intersection of all inherited SCPs plus its own attached SCPs.
