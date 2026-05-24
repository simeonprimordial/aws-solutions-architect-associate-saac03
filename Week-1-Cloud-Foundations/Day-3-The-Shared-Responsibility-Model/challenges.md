# Challenges & How I Solved Them — Day 3

---

## Challenge 1: Confusing RDS Backups vs. EC2 MySQL Backups

**What happened:**
During the 12-item categorisation lab, I initially marked "Database backups on RDS (managed)" as Customer responsibility — because I was thinking of backups as always being my job.

**Why I was wrong:**
With RDS, AWS manages the underlying database engine, OS, and automated backup functionality as part of the managed service. You can configure the backup window and retention period, but the actual execution of automated backups is handled by AWS.

The confusion: there's a difference between **configuring** backups (customer) and **executing** automated backups at the engine level (AWS for RDS).

**Resolution:**
The clearest mental model: if you installed the database yourself (MySQL on EC2), you manage everything. If you asked AWS to run the database for you (RDS), AWS handles the engine-level operations — including automated backups.

**Lesson learned:**
The RDS vs EC2 MySQL distinction is tested directly on the SAA-C03. Memorise it as a pair: same task (backups), opposite responsibility owner depending on service type.

---

## Challenge 2: Understanding Where "Shared Controls" Ends

**What happened:**
Patch management is listed as a Shared Control, but it felt contradictory — I also noted that "EC2 OS patches are the customer's responsibility." How can patching be both shared and the customer's?

**How I resolved it:**
The key is understanding *which layer* is being patched:
- **Host OS / hypervisor patching** → AWS's responsibility (happens beneath your EC2 instance)
- **Guest OS patching** → Customer's responsibility (the Ubuntu or Windows OS inside your EC2 instance)
- **Patch Management as a process** → Shared, because both parties run a patch management programme for their respective layers

So "Shared Control" doesn't mean you split one task 50/50. It means both parties independently manage patching for the layers they own. On the exam, if the question asks who patches the *guest OS on EC2*, the answer is Customer. If it asks who owns *patch management as a practice*, the answer acknowledges both.

**Lesson learned:**
Shared Controls are about process ownership at separate layers, not joint responsibility for a single task.

---

## Challenge 3: Applying the Model to a Non-Technical Explanation

**What happened:**
Step 4 of the lab asked me to explain the Shared Responsibility Model to a Lagos restaurant owner in 3 sentences. My first draft used terms like "IAM," "S3," and "bucket policies" — completely useless to the target audience.

**How I fixed it:**
I rewrote it removing all AWS-specific jargon and anchored every concept to something physical that anyone understands:
- "Physical security" instead of "data centre perimeter controls"
- "Who in your business can access that data" instead of "IAM access management"
- "Your records are encrypted" instead of "SSE-KMS at rest"

**Lesson learned:**
The ability to explain a technical concept to a non-technical audience is as important as knowing the concept itself — especially in client-facing cloud architecture work. If a Nigerian business owner doesn't understand the Shared Responsibility Model, they will misconfigure their own systems and blame AWS when it goes wrong.
