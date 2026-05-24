# Real-World Scenarios — Day 3

---

## Scenario: Lagos Healthcare Startup — Patient Records on S3

**Business:** A healthcare startup storing patient medical records on Amazon S3.

### What AWS is Responsible For
- The physical hard drives storing the data are in a locked, access-controlled, CCTV-monitored facility
- AWS manages the hardware durability — 99.999999999% (11 nines) data durability by design
- Physical security: no unauthorised person can walk into the data centre and retrieve a drive

### What the Customer is Responsible For
The startup must:
1. **Encrypt the data** — Enable S3 server-side encryption (SSE-S3 or SSE-KMS) so the records are encrypted at rest
2. **Restrict access via IAM** — Configure bucket policies and IAM roles so only authorised doctors can read patient records
3. **Enable S3 Access Logging** — Log all read and write operations for HIPAA/NDPC audit requirements
4. **Block public access** — Ensure the S3 bucket is never accidentally made public; use S3 Block Public Access settings
5. **Manage encryption in transit** — Enforce HTTPS-only access using a bucket policy condition: `aws:SecureTransport: true`

### What Would Happen Without These Controls
If the startup neglected to set a bucket policy and left public access enabled, anyone with the S3 URL could download patient records. AWS would not prevent this — it is the customer's configured permission that allows it. AWS built the secure room; the startup left the digital door unlocked.

---

## Scenario: Nigerian Restaurant POS System — Plain Language Explanation

**Target audience:** A Lagos restaurant owner considering moving their POS system and customer data to the cloud.

**3-sentence explanation (no jargon):**

"When you move your business to AWS, Amazon takes full responsibility for the physical security of the computers your data is stored on — they lock the building, monitor the facility 24/7, and replace any failing hardware without you ever knowing. However, you are still fully responsible for who in your business can access that data, making sure your records are encrypted, and setting the rules for what is private versus what is visible to the public. This split is actually better than keeping servers on-site — because Amazon's physical security exceeds anything a restaurant could build, while you retain complete digital control over your own customer data."

---

## Bonus Challenge: The 2019 Capital One Data Breach

**Research source:** https://www.capitalone.com/digital/facts2019/

### 5-Bullet Summary

**1. What happened:**
In July 2019, a former AWS employee exploited a misconfigured Web Application Firewall (WAF) at Capital One to execute a Server-Side Request Forgery (SSRF) attack, gaining access to Capital One's AWS environment and exfiltrating data from S3 buckets. Approximately 100 million US and 6 million Canadian customers had their personal information exposed.

**2. Was this an AWS failure or customer misconfiguration?**
This was **entirely a customer misconfiguration**. AWS's physical infrastructure, hypervisor, and network were never compromised. The attacker never touched AWS hardware or exploited any AWS software vulnerability.

**3. What specific responsibility did Capital One fail to fulfil?**
Capital One failed to correctly configure their Web Application Firewall — a customer-managed resource. The misconfigured WAF allowed the SSRF attack to reach the EC2 metadata endpoint (`169.254.169.254`), from which the attacker obtained temporary IAM credentials. Capital One should have restricted outbound calls from their WAF and applied least-privilege IAM policies so stolen credentials could not access S3 buckets.

**4. The exact responsibilities that were neglected:**
- **Firewall configuration** — Customer responsibility. The WAF rule set allowed unexpected outbound requests.
- **IAM least privilege** — Customer responsibility. The IAM role attached to the server had broader S3 permissions than it needed.
- **S3 bucket access controls** — Customer responsibility. The bucket should have restricted access to only the specific services that required it.

**5. The lesson for every cloud architect:**
AWS passed its half of the contract perfectly. Capital One failed theirs. The breach could have been prevented at multiple points — all on the customer side. This is why the SAA-C03 exam tests the Shared Responsibility Model so heavily: misunderstanding it in production costs companies hundreds of millions in fines, legal fees, and reputational damage. Understanding it correctly is the foundation of every secure architecture.

---

## Discussion — Nigerian Business Application

**Prompt from class:** Think about a Nigerian business you know. How would understanding the Shared Responsibility Model change how they operate?

**My answer:**

**Business: A Nigerian digital bank (e.g. a neobank handling customer KYC data)**

**Current risk without understanding the model:**
A dev team deploys customer ID documents to an S3 bucket for KYC verification. They assume "it's on AWS, AWS will keep it secure." They don't set a bucket policy, don't enable encryption, and leave public access unblocked during testing — and forget to re-enable it.

**How the model changes their operation:**
Understanding that S3 encryption, bucket policies, and public access settings are 100% their responsibility means:
- Encryption at rest is enabled on bucket creation — non-negotiable
- Block Public Access is turned on at the account level — no individual bucket can accidentally go public
- IAM roles follow least privilege — only the KYC verification service can read ID documents; no other service can
- CloudTrail logs every S3 access — full audit trail for CBN/NDPC compliance

**Business impact:**
Customer trust, regulatory compliance, and avoidance of the reputational and financial damage that comes with a breach. A Nigerian neobank that gets breached due to a misconfigured S3 bucket loses its CBN licence — not AWS's fault, but the bank's problem entirely.
