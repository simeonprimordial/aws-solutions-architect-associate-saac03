# SCPs & AWS Organizations Labs — Week 2 Day 3

---

## Lab 1: Read the AWS Organizations Documentation

### Steps
1. Go to https://docs.aws.amazon.com/organizations/latest/userguide/orgs_introduction.html
2. Read the Overview section carefully — note Management Account, Member Accounts, Organisational Units
3. Go to https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html
4. Read the SCP section — focus on the inheritance model and the note that SCPs do not grant permissions

### What I Observed
The docs distinguish clearly between the management account and member accounts throughout. Every description of SCP scope specifies "member accounts" — this is the tell that the management account is exempt.

The SCP page has a section titled "SCP effects on permissions" that explains the intersection model with a Venn diagram. The overlapping region between what the IAM policy allows and what the SCP allows is the effective permission. If either circle excludes an action, the action is denied. This mental model is clearer than reading the JSON evaluation rules alone.

The docs also mention a category of AWS services that are **globally scoped and exempt from region-based SCPs** — IAM, CloudFront, Route 53, and STS, for example. These services operate globally and do not respect the `aws:RequestedRegion` condition key. This is an important caveat for any SCP using region restrictions.

### What I Learned
- Reading the official docs before writing code is the correct order. The SCP examples library (`orgs_manage_policies_scps_examples.html`) is worth bookmarking — it contains production-ready SCP templates for common scenarios.
- The docs make clear that `FullAWSAccess` is the default starting SCP. Many teams enable Organizations and assume they are protected — they are not, until they write and attach additional SCPs.
- Global services being exempt from region SCPs is a caveat that the exam tests. A region-restriction SCP blocks EC2 in us-east-1 but does not block IAM or CloudFront, which are always global.

---

## Lab 2: Design the OluPay Ltd Multi-Account Structure

### Steps
1. Open [Excalidraw](https://excalidraw.com) — free, browser-based, no account needed
2. Design the following structure for OluPay Ltd, a fictional Nigerian fintech:

```
Root (Management Account — billing and org admin only)
│
├── OU: Production
│   ├── Account: prod-backend
│   ├── Account: prod-frontend
│   └── Account: prod-database
│
├── OU: Development
│   ├── Account: dev-team
│   └── Account: staging
│
└── OU: Security
    ├── Account: audit-log
    └── Account: security-tools
```

3. Add SCP labels at each OU showing what the attached policies restrict:
   - **Root level:** `DenyDisableCloudTrail`, `DenyLeaveOrganization`
   - **Production OU:** `DenySpotInstances`, `DenyUntaggedResources`
   - **Development OU:** `DenyReservedInstances`, `DenyBillingChanges`
   - **Security OU:** `AllowReadOnlyAndCloudTrailOnly`
4. Export as PNG and save to `/screenshots/olupay-organizations-diagram.png`

### What I Observed
Drawing this diagram made the inheritance model immediately obvious in a way that reading about it did not. The Root SCPs are a horizontal bar that every OU and account sits under — visually, there is no escaping them. The Production OU SCPs narrow the available permissions further for prod accounts only, without touching Dev or Security accounts.

The Security OU is intentionally the most locked-down. Audit and security tool accounts should have minimal permissions to prevent anyone — even security engineers — from doing things in those accounts that are not audit-related. An IAM admin with `AdministratorAccess` in the audit account cannot exceed the Security OU SCP ceiling.

### What I Learned
- Architecture diagrams are a core deliverable for this kind of topic. A well-drawn org structure tells the story of how an organisation governs itself at a glance — no JSON required.
- The OU design reflects business logic, not just technical grouping. Production and Development OUs have different SCP profiles because the risk tolerance is different: dev can experiment freely within limits, prod cannot.
- The Management Account box sitting outside and above the SCP guardrails in the diagram is not just visual convention — it accurately reflects the technical reality. Never run workloads in the Management Account.

---

## Lab 3: Write the CloudTrail Protection SCP

### Steps
1. Open a text editor (VS Code, Notepad — any plaintext editor)
2. Write the following SCP from scratch, line by line:

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

3. Below the JSON, write a plain-English explanation:

> **Why this SCP exists:** CloudTrail records every API call made in an AWS account — who did what, when, from where. It is the primary audit log for security investigations, compliance audits (ISO 27001, SOC 2, CBN requirements), and incident response. If someone disables CloudTrail — whether accidentally or maliciously — you lose the ability to know what happened before the gap. This SCP makes that impossible for anyone in the member accounts, including IAM administrators and the account root user.

4. Write the `DenyLeaveOrganization` SCP:

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

5. Save both JSON files to `/policy-files/`

### What I Observed
`cloudtrail:UpdateTrail` is a less obvious action to include alongside Stop and Delete — but it is equally important. `UpdateTrail` can be used to redirect CloudTrail logs to an attacker-controlled S3 bucket, effectively creating a gap in the legitimate log stream while making it appear that logging is still active. Including it in the Deny closes that attack vector.

`DenyLeaveOrganization` is a single-action policy and looks almost too simple. But its importance is disproportionate to its size. An account that leaves the organisation immediately loses all SCP restrictions. In the seconds between leaving and the security team noticing, a compromised account could do significant damage with no guardrails. Attach this at the Root level and it becomes non-negotiable.

### What I Learned
- SCPs that protect audit infrastructure (CloudTrail, AWS Config, Security Hub) should always be Root-level — no OU should be exempt from them.
- The `cloudtrail:UpdateTrail` inclusion is the kind of detail that separates a well-thought-out SCP from a copy-pasted one. In security, the obvious threats are covered by default; the non-obvious ones are where misconfigurations happen.
- Writing the plain-English explanation alongside the JSON is not just documentation — it forces you to think about why each action is included, not just what it does. This thinking is exactly what the SAA-C03 exam tests.

---

## Lab 4: SCP vs IAM Policy Comparison Table

Built in `notes/scps-and-organizations.md` (see the comparison table section). Key distinctions noted:

- SCPs restrict member account root users. IAM policies cannot restrict root users.
- SCPs never grant permissions. IAM policies grant and/or deny.
- SCPs apply account-wide via Organizations. IAM policies apply to specific identities.

### What I Learned
- The comparison exercise forces you to articulate the difference precisely rather than having a vague sense of it. When you can fill in every cell of the table without looking at notes, you are ready to answer exam questions on this topic.
- The "what overrides it" row is important: nothing overrides an SCP Deny for member accounts (except the Management Account exemption). This absolute quality of SCPs is what makes them the right tool for organisation-wide guardrails.

---

## Bonus Lab: Data Residency SCP (af-south-1 only)

### The SCP

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonAfricaRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotIn": {
          "aws:RequestedRegion": [
            "af-south-1"
          ]
        },
        "StringNotEquals": {
          "aws:PrincipalARN": "arn:aws:iam::*:role/GlobalBreakGlassRole"
        }
      }
    }
  ]
}
```

### What I Added Beyond the Lab Guide
Added a second Condition clause: `StringNotEquals` on `aws:PrincipalARN` for a hypothetical `GlobalBreakGlassRole`. A break-glass role is an emergency access mechanism — an IAM role that is granted unrestricted access and used only in crisis situations (e.g. a region-wide outage that forces operations into an unapproved region). Without a break-glass exemption in the SCP, the region restriction would block disaster recovery operations.

This is not in the lab guide, but it is how production SCPs are written in real environments. A hard region restriction with no emergency override is itself a risk.

### What I Observed
The `StringNotIn` condition is the correct operator for this pattern — it reads as "if the requested region is NOT in this list, apply the Deny". Only `af-south-1` is in the approved list, so every other region triggers the Deny.

### What I Learned
- Data residency SCPs are a real compliance requirement for South African financial institutions under POPIA (Protection of Personal Information Act) and SARB guidance. The `af-south-1` region was specifically designed to help local institutions meet these requirements.
- Every restrictive SCP in a production environment should be reviewed for emergency override scenarios. A security control that cannot be bypassed during a legitimate emergency is a control that will cause its own incidents.
- The `aws:PrincipalARN` condition key can target specific roles and exclude them from an SCP restriction — this is the correct mechanism for break-glass exemptions.
