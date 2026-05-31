# MFA & Security Best Practices — Day 5 Notes

---

## Why Identity Is Your Largest Attack Surface

In the cloud, your perimeter is not a firewall — it is an identity. There are no physical walls between the internet and your AWS infrastructure. Every service, every bucket, every database is accessible over the internet in principle. The only thing standing between an attacker and your resources is credentials: a username, a password, and ideally a second factor.

When those credentials are compromised — through phishing, credential stuffing, a leaked access key on GitHub, or a brute-force attack — everything that identity has access to is immediately available to the attacker. No alarm goes off. No door rattles. They walk straight in using legitimate credentials.

According to the Verizon Data Breach Investigations Report, over 80% of hacking-related breaches involve compromised credentials. MFA is the single most effective control against this class of attack — it reduces account compromise risk by over 99% when properly enforced. Not 50%. Not 80%. Over 99%.

The CBN Cybersecurity Framework requires MFA for all privileged system access. NDPC data breach notification obligations make credential security a legal risk, not just a technical one.

---

## MFA Types — Not All Equal

### Virtual MFA Device (TOTP App)
An authenticator app (Google Authenticator, Authy, Microsoft Authenticator) that generates a Time-Based One-Time Password (TOTP) every 30 seconds.

How TOTP works: At setup, AWS and your app share a secret key. Every 30 seconds, both independently generate the same 6-digit code using that key plus the current Unix timestamp. If the codes match, authentication succeeds. The code is valid for 30 seconds then changes permanently.

- **Strength:** Good. Resistant to password spraying and brute force.
- **Weakness:** Vulnerable to real-time phishing. An attacker who creates a convincing fake AWS login page can intercept the TOTP code live as you enter it and relay it to the real AWS before it expires. This is a known attack pattern.
- **Cost:** Free. No hardware needed.
- **Use for:** Standard IAM users across the engineering team.

### Hardware MFA Token (Physical TOTP Device)
A dedicated physical device (Gemalto token, RSA SecurID) that generates TOTP codes using the same algorithm as virtual MFA — but on separate hardware.

- **Advantage over virtual:** Cannot be compromised by smartphone malware. If someone steals a developer's phone and the authenticator app is unlocked, they have both factors (the password from phishing and the TOTP from the phone). A hardware token is a completely separate physical object.
- **Cannot be cloned:** Unlike a virtual MFA secret which exists on a phone that can be backed up or screenshotted, a hardware token's secret is embedded in the device and cannot be extracted.
- **Cost:** Requires physical device purchase and distribution logistics.
- **Use for:** Administrators, finance teams, root account holders, any user whose compromise would have severe business impact. Required by many financial regulators for privileged access.

> ⚠️ **Exam Tip:** The exam asks which MFA type is more secure than virtual MFA. The answer is hardware tokens — they cannot be cloned or copied from a screenshot.

### FIDO2 / WebAuthn Security Key (YubiKey, Google Titan)
The strongest MFA option available. Uses public-key cryptography instead of shared secrets.

How FIDO2 works: The security key generates a unique key pair per website. The private key **never leaves the device** — ever. When you authenticate, the website sends a challenge, your key signs it with the private key, and the signature is verified with the public key stored on the server.

**Phishing-resistant by design:** The key pair is bound to the specific domain it was registered with. If you are tricked into visiting `aws-login.evil.com` instead of `console.aws.amazon.com`, the FIDO2 key refuses to sign the authentication request — it cryptographically verifies the site origin and the domains do not match. Unlike TOTP, where a live attacker can relay your code to the real site in 30 seconds, FIDO2 makes this attack mathematically impossible.

- **Cost:** Physical device required (YubiKey ~$25–50, Google Titan ~$30).
- **Use for:** Root account, security team, executives, anyone who cannot be phished.

### MFA Comparison

| Type | How It Works | Phishing Resistant | Cost | Best For |
|---|---|---|---|---|
| Virtual MFA (TOTP app) | Shared secret + timestamp | No — code can be intercepted | Free | Standard users |
| Hardware token (TOTP) | Shared secret on dedicated device | No — but harder to steal | Device cost | Admins, finance, privileged users |
| FIDO2 / WebAuthn | Public-key cryptography, domain-bound | Yes — cryptographic origin check | Device cost | Root, security team, executives |

---

## The 8 IAM Security Best Practices

### 1. Lock Down the Root Account Immediately
The root account has unrestricted access to everything — it cannot be limited by SCPs or IAM policies. Enable MFA before anything else. Store the root password in a secrets vault with dual-person access. Never use root for daily operations — create a separate IAM admin user.

### 2. Never Create Root Access Keys
Root access keys cannot be scoped to specific permissions, rotated to a specific role, or restricted by any policy. If they leak, your entire account is compromised with no recovery path. If root access keys exist on your account — delete them today.

> ⚠️ **Exam Trap:** Root access keys and root MFA are two separate controls. Having MFA enabled does not protect root access keys — they work at the API level without MFA. The correct answer always includes BOTH: delete root access keys AND enable MFA.

### 3. Enable MFA for All Human IAM Users
Every human user — not just admins — should have MFA enforced. The enforcement mechanism is an IAM Deny policy with the `aws:MultiFactorAuthPresent` condition key. This blocks all actions for sessions authenticated without MFA, while still allowing the user to set up their own MFA device.

### 4. Apply Principle of Least Privilege to Every Identity
Grant the minimum permissions required — nothing more. Start from zero and add permissions as needed. Use IAM Access Advisor to identify permissions granted but not used in the last 90 days and remove them. Least privilege is not a one-time setup — it is an iterative process of observe, reduce, repeat.

### 5. Use IAM Roles Instead of Long-Term Access Keys
Access keys are static credentials that live until explicitly deleted or rotated. They get accidentally committed to GitHub, stored in `.env` files, embedded in Docker images. IAM roles issue temporary STS credentials that expire automatically. For every EC2 instance, Lambda function, and ECS task — attach a role, never hardcode keys.

### 6. Rotate Access Keys Every 90 Days — Audit with Credential Report
If access keys must exist (e.g. a CI/CD pipeline before migration to IAM Roles Anywhere), rotate them at least every 90 days. The IAM Credential Report shows the age of every key in the account and when it was last used. Keys unused for 90+ days should be disabled and reviewed.

### 7. Use Permission Boundaries to Delegate Safely
When a team needs to create their own IAM roles (e.g. a dev team creating Lambda execution roles), Permission Boundaries prevent privilege escalation. A developer cannot create a role with more permissions than their own boundary allows. The boundary is a hard ceiling attached to the user — any role they create is constrained to that ceiling.

### 8. Use IAM Access Analyzer to Detect Unintended External Access
Access Analyzer continuously monitors resource-based policies (S3 buckets, IAM roles, KMS keys, Lambda functions, SQS queues, Secrets Manager secrets) for resources accessible from outside your account or organisation. If a bucket policy accidentally allows public access or cross-account access to an unexpected principal, Access Analyzer generates a finding within minutes.

---

## Enforcing MFA with an IAM Policy

Creating an MFA device on a user's account is not enough. The MFA device is optional — the user can still log in without it unless a policy blocks them. The enforcement mechanism is an explicit Deny policy on sessions where `aws:MultiFactorAuthPresent` is false.

### The MFA Enforcement Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyWithoutMFA",
      "Effect": "Deny",
      "NotAction": [
        "iam:CreateVirtualMFADevice",
        "iam:EnableMFADevice",
        "iam:GetUser",
        "iam:ListMFADevices",
        "iam:ListVirtualMFADevices",
        "iam:ResyncMFADevice",
        "sts:GetSessionToken"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

**How to read this policy:**
- `Effect: Deny` — this is a blocking statement
- `NotAction` — Deny applies to everything **except** the listed actions. This is the inverse of `Action`. The listed exceptions are MFA setup actions — they must be permitted so a user without MFA can still enrol their device.
- `Condition: BoolIfExists aws:MultiFactorAuthPresent = false` — the Deny only fires when MFA was NOT used at login. If the user authenticated with MFA, the condition is false and the Deny does not apply.

The result: a user who logs in without MFA is immediately locked out of everything except the MFA setup actions. They can enrol their device and re-authenticate with MFA — at which point normal permissions are restored.

**Important:** `BoolIfExists` rather than `Bool` is the correct operator here. `Bool` fails when the condition key is absent entirely (e.g. for CLI calls using access keys, which never have MFA). `BoolIfExists` handles the absence correctly — if the key exists and is false, deny. If it does not exist, the condition evaluates to true (no match) and the Deny does not trigger.

---

## Three IAM Auditing Tools

### IAM Access Analyzer
**Use when:** You need to detect resources in your account accessible from outside your AWS account or organisation.

Monitors: S3 bucket policies, IAM roles, KMS key policies, Lambda function policies, SQS queue policies, Secrets Manager secret policies.

When it finds an external access path, it generates a finding showing: which resource, which external principal, and which actions are permitted. You resolve findings either by archiving (confirming the access is intentional) or fixing the policy.

**SAA-C03 one-liner:** Access Analyzer = detect **external** resource access.

### IAM Credential Report
**Use when:** You need a compliance-grade snapshot of every user's credential status.

Downloaded as a CSV from IAM console. Contains:
- `mfa_active` — true/false — your primary MFA compliance check
- `password_last_used` — identify inactive accounts
- `access_key_1_last_rotated` / `access_key_1_last_used_date` — identify stale keys
- `password_last_changed` — for password age policies

Run monthly. Automate via: `aws iam generate-credential-report` then `aws iam get-credential-report`.

> ⚠️ **Exam Trap:** The Credential Report is a **detective** control only. It observes and reports. It does not enforce MFA. It does not disable stale keys. It does not rotate passwords. Separate enforcement controls are needed for all of those.

**SAA-C03 one-liner:** Credential Report = per-account compliance **snapshot**.

### IAM Access Advisor
**Use when:** You need to identify unused permissions on a specific user or role.

Available in the IAM console under any user or role's Access Advisor tab. Shows the last time each AWS service was accessed. Any service not accessed in 90+ days is a candidate for permission removal.

The iterative least-privilege process: grant what seems necessary → observe for 30–90 days → remove what was never used → repeat at the next review.

**SAA-C03 one-liner:** Access Advisor = per-user **service usage history**.

### Tool Decision Table

| Question | Tool |
|---|---|
| Which of our S3 buckets are accessible from the internet? | IAM Access Analyzer |
| Which IAM users don't have MFA enabled? | IAM Credential Report |
| Which IAM users have access keys older than 90 days? | IAM Credential Report |
| Which services has this developer never actually used? | IAM Access Advisor |
| Does this Lambda function's resource policy allow cross-account access? | IAM Access Analyzer |
| Which users haven't logged in for 6 months? | IAM Credential Report (`password_last_used`) |
| What permissions can we safely remove from this role? | IAM Access Advisor |

---

## Permission Boundaries — Delegation Without Escalation

A Permission Boundary is an IAM managed policy attached to a user or role that defines the **maximum permissions** that identity can ever have, regardless of what the identity policy grants.

### The Intersection Rule
Effective permissions = Identity Policy AND Permission Boundary. Both must allow an action for it to be permitted.

Example:
- Identity policy grants: `s3:*`, `ec2:*`, `iam:*`
- Permission Boundary allows: `s3:*`, `ec2:*` only
- Effective permissions: `s3:*` and `ec2:*` — IAM actions are blocked despite the identity policy granting them

> ⚠️ **Exam Trap:** Permission Boundaries do not grant permissions. If the Boundary allows `s3:*` but the identity policy has no S3 permissions, the user cannot access S3. The boundary defines the ceiling — the identity policy defines the actual grant. Both must overlap.

### The Delegation Use Case
A dev team needs to create Lambda execution roles. Without boundaries, nothing stops them from creating a role with `AdministratorAccess` and assuming it themselves — privilege escalation.

With a Permission Boundary attached to their IAM users: any role they create is constrained. The developer can create `iam:CreateRole`, but if the boundary requirement is attached as a condition, every role they create automatically inherits the boundary and cannot exceed it.

---

## Layered Identity Security Model

Defence in depth — each layer assumes the one above it can be breached:

```
Layer 1: Root Account Hardening
  MFA (FIDO2 recommended) + delete root access keys + vault password
  Protects against: total account takeover

Layer 2: MFA Enforcement (All Users)
  DenyWithoutMFA policy on all user groups
  SCP blocking MFA device deletion at org level
  Protects against: stolen password attacks

Layer 3: Least Privilege + Permission Boundaries
  Access Advisor quarterly reviews
  Credential Report monthly audits
  Permission Boundaries on delegated role creation
  SCPs at OU level
  Protects against: privilege escalation, blast radius expansion

Layer 4: Continuous Monitoring & Detection
  IAM Access Analyzer (external access findings)
  CloudTrail (all API activity)
  CloudWatch Alarms (root login, IAM changes, MFA disable)
  GuardDuty (anomalous credential usage)
  Protects against: undetected persistence, lateral movement, exfiltration
```

Each layer is independently valuable. Together they create a posture that is resilient against the most common cloud attack vectors.

---

## Common Exam Traps

- **Root MFA and root access keys are separate.** Having MFA enabled does not protect access keys — they are a different credential type not covered by MFA at the API level. Correct answer always includes both: delete keys AND enable MFA.
- **Permission Boundaries do not grant permissions.** Boundary defines the ceiling; identity policy defines the grant. Boundary alone grants nothing.
- **Access Analyzer ≠ Access Advisor.** Analyzer = external access detection. Advisor = service usage history. These are tested interchangeably in exam distractors.
- **Credential Report is detective, not preventive.** It shows you what is wrong. It does not fix anything. Enforcement requires separate controls (Deny policies, Lambda automation).
- **FIDO2 is phishing-resistant; TOTP is not.** The strongest MFA option is always FIDO2/WebAuthn for the exam. TOTP codes can be intercepted in real-time phishing attacks.
