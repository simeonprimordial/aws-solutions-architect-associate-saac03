# Challenges & How I Solved Them — Week 2 Day 1

This file tracks blockers I encountered during labs and how I resolved them. Documenting this helps me learn from mistakes and may help others hitting the same issues.

---

## Challenge 1: Policy Attachment Search Returning Too Many Results

**What happened:**
When attaching policies to the Developers group, searching for "EC2" returned over 40 results. It was unclear which specific policy to select.

**What I tried:**
- Searched `AmazonEC2FullAccess` as the exact policy name — this filtered it down to one result
- Used the Type column to confirm it was an AWS managed policy, not a customer-managed one

**Resolution:**
Always search the full exact policy name when you know it. Searching a short keyword like "EC2" is too broad in IAM — the policy list is large.

**Lesson learned:**
Know the exact names of common AWS managed policies before going into the console. The SAA-C03 exam also tests whether you can identify the right policy for a scenario, so learning these names is double value.

---

## Challenge 2: Couldn't Find the Account Sign-In URL for Test Users

**What happened:**
After creating `analyst-user` and `manager-user`, I wasn't sure where to find the sign-in URL to test logging in as them.

**What I tried:**
- Checked the IAM Dashboard — the sign-in URL is displayed at the top: `https://<account-id>.signin.aws.amazon.com/console`
- Also found it by clicking on the user → Security credentials tab

**Resolution:**
The IAM Dashboard homepage shows the account-specific sign-in URL. Bookmark it.

**Lesson learned:**
The sign-in URL for IAM users is different from the root account sign-in. Root uses `https://console.aws.amazon.com/`. IAM users use the account-specific URL. Keep both bookmarked separately.

---

## Challenge 3: Trust Policy Not Immediately Obvious in Role Creation

**What happened:**
During EC2 Role creation, I wasn't sure where the Trust Policy was — I only saw the permissions step and thought I'd missed something.

**What I tried:**
- Clicked through all the tabs on the Role creation wizard
- Found that selecting "AWS service → EC2" in Step 1 automatically generates the Trust Policy — it's shown in the final review step

**Resolution:**
The Trust Policy is auto-generated based on your Step 1 selection. Click through to the review step to verify the JSON before creating the role.

**Lesson learned:**
AWS generates the Trust Policy automatically when you select a trusted entity type. But always review the JSON in the final step — in more advanced scenarios (cross-account access, federated identity), you will need to edit it manually.

---

## Challenge 4: Policy Simulator Loaded But Showed No Actions

**What happened:**
After selecting `analyst-user` in the Policy Simulator, the action list for S3 appeared empty.

**What I tried:**
- Refreshed the page — the list loaded properly on the second attempt
- Confirmed I was signed into the correct admin account (not as analyst-user)

**Resolution:**
Browser refresh resolved the issue. The Policy Simulator requires admin-level access to load policy data — confirm you're signed in as an admin or the root account when using it.

**Lesson learned:**
The IAM Policy Simulator needs to be run from an account with sufficient IAM read permissions. Don't try to run it while signed in as a restricted user — it won't work.

---

*Add new challenges here as they come up in future days.*
