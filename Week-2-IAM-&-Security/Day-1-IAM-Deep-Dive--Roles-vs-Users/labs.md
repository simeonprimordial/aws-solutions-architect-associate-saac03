# IAM Labs — Week 2 Day 1

---

## Lab 1: Create Three IAM Groups

### Steps
1. Go to **IAM → User groups → Create group**
2. Create Group 1: `Developers`
   - Attach policies: `AmazonEC2FullAccess` + `AWSLambdaFullAccess`
3. Create Group 2: `Analysts`
   - Attach policies: `AmazonS3ReadOnlyAccess` + `AmazonAthenaFullAccess`
4. Create Group 3: `Managers`
   - Attach policy: `ReadOnlyAccess` (grants read-only access to ALL AWS services)
5. Confirm all three groups appear in the IAM User groups console

### What I Observed
Attaching a policy to a group is instantaneous — no propagation delay. The policy list search bar in the console is useful; typing `EC2Full` filters the results immediately. `ReadOnlyAccess` is a large AWS-managed policy that covers hundreds of services — suitable for a manager who needs visibility without the ability to change anything.

### What I Learned
- Policies are attached to **groups**, not individual users. This is the correct IAM pattern.
- AWS-managed policies are pre-built and regularly updated by AWS to include new services — safer than writing your own from scratch for common use cases.
- The principle of least privilege in practice: Analysts get read-only S3 and Athena access, not EC2. Developers get EC2 and Lambda, not S3 write. Each group has exactly what its role requires.

---

## Lab 2: Create Three IAM Users

### Steps
1. Go to **IAM → Users → Create user**
2. Create User: `dev-user`
   - No console access (programmatic only — this user will use CLI or SDK)
   - Add to group: `Developers`
   - Download the access keys CSV — store it securely, this is the only time you can download it
3. Create User: `analyst-user`
   - Enable console access
   - Set a custom password or auto-generated one
   - Add to group: `Analysts`
4. Create User: `manager-user`
   - Enable console access
   - Add to group: `Managers`
5. Confirm all three users appear in the IAM Users list with their respective groups shown

### What I Observed
When you enable console access for a user, AWS generates a sign-in URL specific to your account. The format is: `https://<account-id>.signin.aws.amazon.com/console`. This URL is what you share with team members — not your root account sign-in URL.

The access key CSV for `dev-user` shows two values: an Access Key ID (starts with `AKIA`) and a Secret Access Key. The Secret Access Key is shown **once only** — if you lose it, you must delete the key pair and create a new one.

### What I Learned
- `dev-user` has no console access by design — developers using the CLI or SDK don't need it, and removing it reduces the attack surface.
- Never share the access keys CSV over Slack, email, or any unencrypted channel. Store it in a password manager.
- If an access key is accidentally committed to a public GitHub repo, rotate it immediately. GitHub and AWS both scan for exposed keys — AWS may automatically quarantine the key, but don't rely on that.

---

## Lab 3: Test Permissions by Signing In

### Steps
1. Open a **private/incognito browser window** (keeps your admin session separate)
2. Sign in as `analyst-user` using the account-specific sign-in URL
3. Navigate to **S3** — attempt to list buckets → should succeed (ReadOnly allows `ListBucket`)
4. Attempt to **create a new S3 bucket** → expect Access Denied
5. Screenshot the Access Denied message
6. Sign out. Sign in as `manager-user`
7. Navigate to **EC2** → attempt to launch an instance → expect Access Denied
8. Screenshot the Access Denied message

### What I Observed
Listing S3 buckets worked immediately for `analyst-user`. The moment I clicked **Create bucket**, the console returned an Access Denied error. The error message shows the specific IAM action that was denied — in this case `s3:CreateBucket` — which is useful for debugging in real environments.

For `manager-user`, the EC2 dashboard loaded fine (read access), but clicking **Launch instance** immediately returned Access Denied. `ReadOnlyAccess` allows viewing resources but not `ec2:RunInstances`.

### What I Learned
- Least privilege works exactly as intended. Read-only means read-only — even if the console UI shows buttons, the API calls behind them are blocked.
- The AWS console is just an interface over API calls. Every button click is an API request. IAM policies operate at the API level, not the UI level. This is why disabling a button in a custom dashboard is not a security control — you need IAM policies.
- Using a private browser window is the cleanest way to test multiple users simultaneously without logging out of your admin session.

---

## Lab 4: Create an EC2 IAM Role

### Steps
1. Go to **IAM → Roles → Create role**
2. Select trusted entity type: **AWS service**
3. Select service: **EC2**
4. Attach permission policy: `AmazonS3ReadOnlyAccess`
5. Role name: `EC2-S3-ReadOnly-Role`
6. Review the auto-generated **Trust Policy** — confirm `ec2.amazonaws.com` is the principal
7. Click **Create role**

### What I Observed
When you select EC2 as the trusted entity, AWS automatically generates the Trust Policy for you:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

This means only the EC2 service can assume this role. An IAM user cannot assume it directly — they would need to be added as a principal in the Trust Policy.

### What I Learned
- The Trust Policy is separate from the Permissions Policy — two distinct JSON documents on every Role. The Trust Policy is what makes Roles fundamentally different from Users.
- Attaching this role to an EC2 instance means any application running on that instance can read from S3 without any credentials being stored anywhere on the server.
- This is how production applications should work: the infrastructure handles authentication, the developer writes code, nobody manages keys.

---

## Lab 5: Draw the IAM Architecture Diagram

### Steps
1. Open [Excalidraw](https://excalidraw.com) (free, no account needed)
2. Draw the hierarchy:
   - Root Account (top) → IAM Admin User
   - IAM Admin User → three Groups (Developers, Analysts, Managers)
   - Each Group → assigned Users
   - Each Group → attached Policies
3. Draw the EC2 Role separately:
   - EC2 Service → (trust arrow) → `EC2-S3-ReadOnly-Role` → (permission arrow) → S3
4. Label every element clearly
5. Export as PNG → save to `/screenshots/iam-architecture-diagram.png`

### What I Observed
Visualising the structure on a diagram made the permission inheritance immediately obvious. The moment you draw the arrow from Group → User, it's clear why policies belong on groups and not on individual users: one arrow updates all members.

### What I Learned
- Architecture diagrams are a core deliverable in real cloud engineering work. Getting comfortable with diagramming tools now is a practical skill, not just a learning aid.
- The Role section of the diagram looks different from the User/Group section — it's a separate identity chain (EC2 → STS → Role → S3) rather than a human permission chain. This visual separation reinforces the conceptual difference.

---

## Bonus Lab: IAM Policy Simulator

### Steps
1. Go to [policysim.aws.amazon.com](https://policysim.aws.amazon.com)
2. Select user: `analyst-user`
3. Select service: **S3**
4. Select action: `s3:DeleteObject`
5. Click **Run Simulation**
6. Screenshot the result

### What I Observed
The simulator returned **Denied** for `s3:DeleteObject`. The `AmazonS3ReadOnlyAccess` policy attached to the Analysts group only grants `s3:GetObject`, `s3:ListBucket`, and related read actions. Delete operations are not included.

### What I Learned
- The IAM Policy Simulator is an underused but powerful tool. In real environments, use it before deploying a change to verify permissions work as intended — far better than discovering a misconfiguration in production.
- It shows the **specific policy statement** that caused an allow or deny, which makes debugging IAM issues much faster than trial and error in the console.
