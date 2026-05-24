# Challenges & How I Solved Them — Day 5

---

## Challenge 1: Understanding IAM Roles vs IAM Users

**What happened:**
During the lab, I initially tried to think of roles as "another type of user." The mental model broke when the lab guide said "EC2 instances use roles — not user credentials." An EC2 instance is not a person — so how does it authenticate?

**How I resolved it:**
The key distinction: IAM Users have permanent, long-lived credentials (a password or access keys that don't expire unless you rotate them). IAM Roles issue **temporary credentials** — AWS automatically generates them, attaches them to the service (EC2, Lambda, etc.), and rotates them every few hours.

The mental model that clicked: think of an IAM Role as a **vending machine badge**. A contractor coming to fix your office vending machine gets a temporary visitor badge for the day. They can access the vending area only, for that day only. When they leave, the badge is revoked. They don't get a permanent employee keycard. IAM Roles work the same way — temporary, scoped, automatically revoked.

**Lesson learned:**
Roles are for services and temporary access. Users are for humans and long-term programmatic access. When the question is "how does an EC2 instance access S3 securely?" — the answer is always a Role.

---

## Challenge 2: The IAM Policy JSON Was Confusing at First

**What happened:**
The policy JSON structure looked intimidating. Multiple nested objects, ARN syntax I didn't recognise, and unclear what `"Action": "s3:*"` actually meant vs `"Action": "s3:GetObject"`.

**How I resolved it:**
Breaking it into three questions for every policy statement:
1. **Effect** — Am I allowing or denying this? (`Allow` or `Deny`)
2. **Action** — What specific API calls? (`service:ActionName` — e.g. `s3:GetObject` reads a file, `s3:PutObject` uploads a file, `s3:*` does everything in S3)
3. **Resource** — To which specific thing? (An ARN like `arn:aws:s3:::my-bucket/*` means all objects inside `my-bucket`)

Once I mapped those three questions to the JSON keys, it became readable:
```json
{
  "Effect": "Allow",        ← Yes, this is allowed
  "Action": "s3:GetObject", ← Specifically: downloading a file from S3
  "Resource": "arn:aws:s3:::reports-bucket/*"  ← Only from the reports bucket
}
```

**Lesson learned:**
IAM policies are logic gates written in JSON. They're not complicated — they're just unfamiliar syntax. Every policy statement answers three questions: allow or deny? what action? on which resource?

---

## Challenge 3: Forgetting to Switch to us-east-1 for CloudWatch Billing Alarms

**What happened:**
I initially tried to create a CloudWatch Billing Alarm while my console was set to af-south-1. The billing metric (Estimated Charges) was not available in the metric selection dropdown — I couldn't find it anywhere.

**How I resolved it:**
The exam trap from today's slides: billing data is only in us-east-1. Switching the Region selector to us-east-1 immediately made the Estimated Charges metric available under CloudWatch → Alarms → Billing.

**Lesson learned:**
Any time you're working with billing-related monitoring, switch to us-east-1 first. This is a habit worth building permanently. The rule is so counterintuitive (why would billing be in only one Region?) that AWS tests it directly on the SAA-C03.

The reason: AWS processes billing data centrally and surfaces it in the us-east-1 Region. It's an AWS architecture decision, not a rule you need to understand deeply — just memorise it.

---

## Challenge 4: Stop vs Terminate Confusion

**What happened:**
After completing the lab, I stopped an EC2 instance thinking I was done. I then realised from the billing notes that stopping an instance doesn't stop all charges — the EBS volume continues billing.

**How I resolved it:**
The distinction:
- **Stop** = power off. Compute billing pauses. Storage billing continues. Data preserved.
- **Terminate** = delete. All billing stops. Data gone.

For lab exercises with no persistent data to keep — always Terminate. For production instances you intend to restart — Stop is appropriate.

**Lesson learned:**
"Stopped" is not the same as "free." After every lab, check the EC2 console for any instances in a "stopped" state that should have been terminated. A stopped instance with a 100 GB EBS volume costs ~$10/month in storage charges — even while "off."
