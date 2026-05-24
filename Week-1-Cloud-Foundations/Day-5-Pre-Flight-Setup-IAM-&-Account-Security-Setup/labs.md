# IAM & Account Security Labs — Day 5

**Objective:** Create an IAM admin user with MFA, create an IAM group with AdministratorAccess, add the user to the group, and verify root account security.

**Time:** 55 minutes
**Billable resources created:** Zero — IAM users, groups, and MFA setup are always free.

---

## Lab 1: Navigate to IAM

### Steps
1. Sign in to the AWS Console using the root account (email + password + MFA)
2. Search for **IAM** in the top search bar → click IAM
3. Review the **Security Recommendations** section on the IAM dashboard

### What I Observed
The IAM Security Recommendations panel listed actionable security items with red warnings. On a freshly set up account with Day 1 MFA already configured, the main open item was the absence of an IAM admin user — the panel flags when root is the only active identity.

The panel also shows the **IAM Security Score** — a percentage reflecting how many best practices are currently implemented. Starting from a partly configured account, this was already above 50% thanks to root MFA from Day 1.

### What I Learned
The Security Recommendations panel is a live security checklist — not just a one-time setup guide. It should be reviewed regularly, especially after adding new users or changing permissions. AWS updates it when new recommendations are added.

---

## Lab 2: Create the IAM Administrators Group

### Steps
1. IAM left sidebar → **User groups** → **Create group**
2. Group name: `Administrators`
3. In "Attach permissions policies" → search for `AdministratorAccess` → check the box
4. Click **Create group**

### What I Observed
`AdministratorAccess` is an AWS Managed Policy — pre-built by AWS and always up to date. Its JSON grants `"Action": "*"` on `"Resource": "*"` — full access to all AWS services. This is intentionally broad for an admin group; the least-privilege boundary is enforced at the group membership level (only cloud architects belong here, not developers).

### What I Learned
Creating the group before creating the user is the correct sequence. This way, when the user is created in the next step, they can be added to the group in a single step — permissions are assigned via group membership from the start, not retrofitted afterward.

IAM groups cannot contain other groups — only users. This is a frequently confused point: groups are flat collections, not nested hierarchies.

---

## Lab 3: Create the IAM Admin User

### Steps
1. IAM left sidebar → **Users** → **Create user**
2. Username: `admin-[yourname]` (e.g. `admin-daniel`)
3. Check: **Provide user access to the AWS Management Console**
4. Select: **I want to create an IAM user**
5. Set a custom password → optionally check: **Users must create a new password at next sign-in**
6. Next screen → **Add user to group** → select `Administrators`
7. Complete creation → **Download the CSV with credentials** → save securely

### IMPORTANT: Downloaded CSV
The CSV contains the IAM username, console sign-in URL, and password. This is the only time AWS shows this information. Store it in a secure password manager — not in a text file on the desktop, not in a WhatsApp message, not in email.

### What I Observed
The creation flow shows a clear summary before completion: the user's name, group membership, and the policy that will be inherited. The confirmation screen shows a green banner confirming successful creation and a link to download credentials.

The IAM sign-in URL shown on the IAM dashboard follows the format:
`https://<12-digit-account-id>.signin.aws.amazon.com/console`

### What I Learned
The username convention matters in teams. `admin-daniel` is better than `daniel` or `user1` because:
- The `admin-` prefix immediately signals the permission level in any audit log
- CloudTrail entries show the IAM username — meaningful names make incident investigation faster
- Future team members can understand the naming convention without documentation

---

## Lab 4: Enable MFA on the IAM Admin User

### Steps
1. Click on the new IAM user → **Security credentials** tab
2. Under **Multi-factor authentication (MFA)** → click **Assign MFA device**
3. Choose **Authenticator app** → scan QR code with Google Authenticator or Authy
4. Enter two consecutive 6-digit codes → click **Add MFA**

### What I Observed
The MFA setup flow is identical to the root MFA setup from Day 1 — same QR code process, same two-code verification. The Security credentials tab now shows the MFA device listed as active with the device type and registration date.

### What I Learned
An IAM admin user without MFA is almost as dangerous as using root directly. If the IAM admin password is compromised (phishing, weak password, reused password), an attacker gets AdministratorAccess to the entire account. MFA makes a compromised password useless without also having the physical phone.

Best practice for production accounts: require MFA as a **policy condition** — deny all actions unless MFA is present in the authentication session. This can be enforced via an IAM policy condition key: `"Condition": {"Bool": {"aws:MultiFactorAuthPresent": "true"}}`.

---

## Lab 5: Sign In as the IAM User

### Steps
1. Find the Account ID on the IAM dashboard (top-right or IAM dashboard)
2. Sign out from the root account
3. Sign in at: `https://<account-id>.signin.aws.amazon.com/console`
4. Enter IAM username and password → enter MFA code when prompted
5. Verify: the account name displayed top-right should show the IAM username, not root

### What I Observed
After signing in as the IAM admin user, the console looks identical to the root view — because `AdministratorAccess` grants full access to all services. The difference is visible only in the top-right corner: instead of showing the root email address, it shows `admin-daniel` (the IAM username).

This is the correct state. All future lab work is done from this IAM user, not root.

### What I Learned
The IAM sign-in URL includes the account ID as a prefix. This is how AWS distinguishes between root login (no account ID in URL, uses root email) and IAM login (account ID in URL, uses IAM username). Bookmarking the IAM sign-in URL as the default starting point prevents accidentally logging in as root.

---

## Lab 6: Verify Root Account Security

### Steps
1. Sign back in as root one final time
2. IAM → **Security recommendations** → verify:
   - Root MFA is enabled ✅
   - Root has no access keys ✅
3. If root access keys exist: **Manage** → **Delete** — root must never have access keys
4. Security Recommendations panel should show all green checkmarks

### What I Observed
After completing the IAM setup, the Security Recommendations panel showed significantly improved status. The most important checks:
- Root MFA enabled: ✅ (configured on Day 1)
- Root access keys: ✅ None (never created)
- IAM users with admin access exist: ✅
- MFA on admin user: ✅

### What I Learned
This green checklist represents the minimum viable security baseline for any AWS account. In a real production environment, the checklist would be more extensive — but for a learning account, these four checks cover the highest-risk exposures.

After this step, root is locked away. All future console access is through `admin-[yourname]`.

---

## Bonus Challenge: ReadOnly User — Access Denied Test

### Setup
1. Create a second IAM user: `readonly-test`
2. Add to a new group `ReadOnly` with `ReadOnlyAccess` managed policy
3. Enable console access for this user

### Test
1. Sign in as `readonly-test`
2. Navigate to S3 → attempt to create a new bucket
3. Document the result

### Result
```
Error: Access Denied

User: arn:aws:iam::123456789012:user/readonly-test is not authorized
to perform: s3:CreateBucket because no identity-based policy allows
the s3:CreateBucket action
```

### What This Proves
The `ReadOnlyAccess` managed policy allows only `Get*`, `List*`, and `Describe*` actions across AWS services. `s3:CreateBucket` is a write operation — it is not in the Allow list, so it is **implicitly denied** by the IAM default-deny behaviour.

This is the fundamental IAM principle in action: **everything is denied unless explicitly allowed**. There is no need to write an explicit `"Effect": "Deny"` for every action you want to block — the absence of an Allow is sufficient to deny it.

### Practical application
This test models a real security control. In production:
- Auditors get `ReadOnlyAccess` — they can see everything but cannot change anything
- The Access Denied error is logged in CloudTrail — if an auditor attempts to create resources, it is captured as a security event
- Screenshot of the Access Denied message is a portfolio-quality proof of IAM enforcement
