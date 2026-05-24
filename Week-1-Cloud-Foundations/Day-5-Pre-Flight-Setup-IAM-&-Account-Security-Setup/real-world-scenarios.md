# Real-World Scenarios — Day 5

---

## Scenario 1: David's $47 Forgotten EC2 Instance

**What happened:**
David, a bootcamp student, completed a lab exercise involving an EC2 instance. He finished the lab, closed his laptop, and didn't terminate the instance. Without a billing alert configured, he had no visibility into the running cost.

When his monthly AWS invoice arrived, he discovered a $47 charge — roughly the cost of running a t3.small instance 24/7 for a month.

**Why the alert would have saved him:**
With a $5 CloudWatch Billing Alarm or AWS Zero Spend Budget configured before starting any labs:
- The alarm would have triggered within 2–3 days of the instance running
- The email notification would have prompted him to check the console
- He would have terminated the instance before the bill exceeded $5–10
- Total spend would have been under $2 instead of $47

**The habit this creates:**
Before every lab:
1. Confirm your billing alert is active
2. After every lab: terminate (not just stop) resources you don't need
3. Check the EC2 console for any running instances before logging off
4. Check Billing Dashboard weekly

**The practical difference between Stop and Terminate:**
- **Stop** — The instance is paused. Compute billing stops. But the EBS storage volume continues to be charged. The instance data is preserved.
- **Terminate** — The instance and its default storage are deleted. All billing for that resource stops. Data is lost unless backed up.

For lab exercises with no persistent data: always **Terminate**, not Stop.

---

## Scenario 2: Nigerian Fintech — IAM for a 5-Person Engineering Team

**Business:** A Lagos fintech startup with 5 team members:
- 1 Cloud Architect (full admin access needed)
- 2 Backend Developers (can deploy to EC2 and Lambda, read S3)
- 1 Data Analyst (read-only access to S3 and Athena)
- 1 Finance Manager (billing and cost explorer only)

**Wrong approach (what most startups do):**
Share the root account credentials with everyone. "It's faster and everyone can do everything."

**What actually happens:**
- Any team member's compromised laptop = full account takeover
- No audit trail — CloudTrail logs show all actions as "root" with no way to know who did what
- A developer accidentally deletes a production database — no way to identify who, no way to scope the blast
- Finance manager can now accidentally (or intentionally) terminate production instances

**Correct IAM architecture:**

```
AWS Account
│
├── IAM Group: Administrators
│   └── Policy: AdministratorAccess
│   └── Members: cloud-architect-chidi
│
├── IAM Group: Developers
│   └── Policy: Custom developer policy (EC2, Lambda deploy, S3 read)
│   └── Members: dev-amaka, dev-tunde
│
├── IAM Group: DataAnalysts
│   └── Policy: Custom analyst policy (S3 read, Athena query)
│   └── Members: analyst-funmi
│
└── IAM Group: Finance
    └── Policy: Billing + CostExplorer read access
    └── Members: finance-bola
```

**What this achieves:**
- Compromise of any developer credential → attacker has developer-level access only, not admin
- CloudTrail logs show `dev-amaka` deleted resource X at 14:32 — full accountability
- Finance manager logs show billing queries only — can never touch infrastructure
- Adding a new developer: create user, add to Developers group — permissions inherited automatically
- Developer leaves: delete or disable their user — permissions revoked instantly

---

## Scenario 3: EC2 Application Accessing S3 — Role vs Access Keys

**Business context:** A Nigerian e-commerce application runs on EC2 and needs to upload product images to S3.

**Wrong approach (dangerous):**
```
# Hardcoded in the application config file:
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=xxxxx...
```

**Why this is dangerous:**
- If this config file is accidentally committed to GitHub — attacker has API access to your account
- If the EC2 instance is compromised — attacker extracts the keys from the config
- Keys never expire — a leaked key can be used indefinitely
- GitHub scanners (both legitimate security tools and malicious actors) actively scan for AWS key patterns

**Correct approach — IAM Role:**
1. Create an IAM Role: `ec2-product-image-upload-role`
2. Attach a policy: Allow `s3:PutObject` on `arn:aws:s3:::product-images-bucket/*` only
3. Attach the role to the EC2 instance at launch
4. The application uses the AWS SDK — it automatically picks up the temporary role credentials from the EC2 metadata endpoint
5. No keys in code. No keys in config files. No keys anywhere.

**What AWS does automatically:**
- Issues temporary credentials to the EC2 instance via the metadata service
- Rotates these credentials automatically every few hours
- If credentials are somehow exposed, they expire quickly — the blast radius is contained

**Exam phrase to memorise:** "EC2 needs to access another AWS service → attach an IAM Role to the instance."

---

## Bonus Challenge: Read-Only User Access Denied Test

**Lab scenario:** Create a second IAM user with ReadOnlyAccess. Sign in as that user and attempt to create an S3 bucket.

**What happens:**
The console displays an `Access Denied` error. The AWS error message:
```
User: arn:aws:iam::123456789012:user/readonly-test is not authorized
to perform: s3:CreateBucket on resource: arn:aws:s3:::test-bucket
```

**What this proves:**
IAM policies are not theoretical — they are enforced in real-time by AWS at the API level. The `ReadOnlyAccess` managed policy explicitly allows only `Get*`, `List*`, and `Describe*` actions. `s3:CreateBucket` is a write action — it is implicitly denied because it is not in the Allow list.

**The key insight:**
IAM policies use **default deny** — everything is denied unless explicitly allowed. There is no need to write a Deny statement for every action you want to block. You only write Allow statements. Anything not explicitly allowed is automatically denied.

---

## Discussion — Nigerian Business Application

**Prompt:** Think about a Nigerian business. How would IAM change how they operate?

**My answer: A Nigerian hospital group with 3 locations in Lagos**

**Current situation (no IAM):**
The IT team shares one set of admin credentials. Doctors, nurses, and billing staff all request IT to perform any cloud operation on their behalf. Slow. No audit trail. One shared credential = massive blast radius if compromised.

**With proper IAM:**
- Doctors → ReadOnly access to the patient records S3 bucket in their location only
- Billing staff → Access to billing databases and RDS queries only — cannot view patient records
- Radiologists → Access to medical imaging S3 bucket only
- IT team → Admin access via IAM admin user with MFA

**NDPC compliance impact:**
Nigeria's Data Protection Commission requires that patient data access be logged and restricted. IAM + CloudTrail provides the full audit trail: who accessed patient record X, at what time, from which IP address. Without IAM, this audit trail is impossible — "the IT team did it" is not a compliance-ready answer.
