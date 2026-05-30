# Challenges & How I Solved Them — Week 2 Day 3

This file tracks blockers I encountered during labs and how I resolved them. Documenting this helps me learn from mistakes and may help others hitting the same issues.

---

## Challenge 1: Confusion Between Root Account and Root Container

**What happened:**
When reading the AWS Organizations docs, I kept confusing "Root" (the top-level OU container in Organizations) with "root user" (the unrestricted login tied to the account email address). The docs use both terms frequently and the distinction is not always obvious from context.

**What I tried:**
- Re-read the relevant section — the docs describe the Root container as the "parent of all OUs and accounts"
- Looked for a visual — the hierarchy diagram in the docs made it clearer: Root (container) → OUs → Accounts. The management account sits above and outside this hierarchy.

**Resolution:**
The disambiguation: **Root (capital R)** = the top-level container in AWS Organizations. **root user (lowercase)** = the unrestricted account owner identity. When exam questions say "SCPs attached to the Root", they mean the container. When they say "root user of a member account", they mean the identity.

**Lesson learned:**
AWS documentation reuses common terms with different meanings depending on context. Always clarify which "root" a question refers to — the Organizations container or the IAM root user identity.

---

## Challenge 2: Understanding Why Global Services Bypass Region SCPs

**What happened:**
While writing the `DenyNonApprovedRegions` SCP for the bonus lab, I realised I was uncertain whether blocking `Action: *` with a region condition would also block IAM, CloudFront, and Route 53 — all of which are global services.

**What I tried:**
- Searched the AWS docs for "global services SCP region condition"
- Found the relevant AWS documentation note: global services like IAM, CloudFront, Route 53, AWS Organizations, STS, and AWS Support do not use the `aws:RequestedRegion` condition key. Requests to these services do not carry a region value, so the `StringNotIn` condition evaluates to false and the Deny does not trigger.

**Resolution:**
Global services are effectively exempt from region-based SCPs by their nature. A region SCP with `Action: *` restricts regional services (EC2, RDS, Lambda, S3 in a region) but leaves global services unaffected. This is correct and expected behaviour — you cannot restrict IAM to a specific region because IAM has no regional scope.

**Lesson learned:**
Region-based SCPs are not a universal lock on all AWS actions. They only apply to regional services. Global services (IAM, CloudFront, Route 53, STS, etc.) operate outside the regional scope and cannot be restricted by `aws:RequestedRegion`. The exam tests this distinction.

---

## Challenge 3: Difference Between StringNotIn and StringNotEquals in Conditions

**What happened:**
When writing the bonus SCP, I initially used `StringNotEquals` for the region condition instead of `StringNotIn`. The policy worked for a single region but I was unsure how to specify multiple approved regions.

**What I tried:**
- Tested `StringNotEquals` with a list — found that `StringNotEquals` evaluates each value individually and returns true if the condition key does not match any of the listed values, but this behaves differently than expected when multiple values are listed
- Read the IAM condition operator docs and found that `StringNotIn` is specifically designed for "the value is NOT in this list" checks — cleaner and more readable for this use case

**Resolution:**
Use `StringNotIn` when checking that a value is not present in a list of approved values. Use `StringNotEquals` for a single value comparison. For multiple approved regions, `StringNotIn` is the correct operator.

**Lesson learned:**
The IAM condition operator library has specific operators for list membership checks: `StringIn` (is in list) and `StringNotIn` (is not in list). For single-value comparisons: `StringEquals` and `StringNotEquals`. Choosing the right operator avoids subtle policy logic errors that are hard to debug without the Policy Simulator.

---

## Challenge 4: Break-Glass Role ARN Wildcard Format

**What happened:**
When writing the break-glass exemption in the bonus SCP, I was unsure of the correct ARN format for targeting a role across multiple accounts using a wildcard.

**What I tried:**
- Checked the IAM ARN format documentation
- Found that ARN wildcards (`*`) can be used in specific ARN segments
- The format `arn:aws:iam::*:role/GlobalBreakGlassRole` uses `*` in the account ID position, which matches the same role name in any account in the organisation

**Resolution:**
`arn:aws:iam::*:role/RoleName` is the correct format to reference an IAM role by name across all accounts in an organisation. The `*` in the account ID position is valid in SCP condition values and matches any account.

**Lesson learned:**
Wildcard ARNs in SCP conditions are useful for cross-account role exemptions. The account ID segment of an ARN can be wildcarded in policy conditions (though not in IAM resource statements for all services). This pattern is commonly used in production SCPs to create break-glass exemptions that apply across all member accounts without hardcoding individual account IDs.

---

*Add new challenges here as they come up in future days.*
