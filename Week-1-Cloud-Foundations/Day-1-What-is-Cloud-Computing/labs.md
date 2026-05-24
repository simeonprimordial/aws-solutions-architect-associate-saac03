# AWS Cloud Foundations Labs — Day 1

---

## Lab 1: AWS Free Tier Account Setup

### Steps
1. Go to [aws.amazon.com](https://aws.amazon.com)
2. Click **Create an AWS Account**
3. Enter your email address and choose an account name
4. Add billing information (credit/debit card required — no charge for Free Tier usage)
5. Verify your phone number via SMS or voice call
6. Select the **Free Support Plan**
7. Sign in to the AWS Management Console

### What I Observed
The account creation process requires a valid payment method even for Free Tier. AWS places a temporary $1 authorisation hold to verify the card — this is reversed within a few days.

### What I Learned
- AWS accounts have a **root user** (the email you sign up with) that has unrestricted access. You should never use root for day-to-day tasks.
- Free Tier has three types: Always Free, 12-Month Free, and Trials. Know the difference.

---

## Lab 2: MFA Setup on Root Account

### Steps
1. Click on your account name (top right) → **Security Credentials**
2. Scroll to **Multi-Factor Authentication (MFA)**
3. Click **Assign MFA Device**
4. Choose **Authenticator App**
5. Scan the QR code with Google Authenticator or Authy
6. Enter two consecutive 6-digit codes to verify
7. Click **Add MFA** to complete setup

### What I Observed
The console shows a green confirmation banner once MFA is successfully assigned. The root account now requires both a password and a one-time code to log in.

### What I Learned
- MFA on the root account is a critical security best practice — AWS flags it as a recommendation in the Security Hub dashboard.
- If you lose access to your MFA device, account recovery through AWS Support is a long process. Back up your MFA seed code securely.

---

## Lab 3: AWS Console Exploration

### Services Explored

| Service | Category | What It Does |
|---|---|---|
| EC2 | Compute | Virtual machines in the cloud |
| S3 | Storage | Object/file storage at scale |
| IAM | Security | Manage users, roles, and permissions |
| RDS | Database | Managed relational databases |
| Lambda | Serverless | Run code without provisioning servers |

### Tasks Completed
- Switched between AWS Regions (us-east-1, af-south-1)
- Used the **Services** search bar to navigate quickly
- Explored the Console Home dashboard and pinned frequently used services

### What I Observed
The AWS Console has hundreds of services. The search bar is the fastest way to navigate. Regions matter — resources created in one region are not visible in another.

### What I Learned
- **Region selection is critical.** Always confirm you're in the right region before creating resources. A common mistake is launching an EC2 instance in the wrong region and wondering where it went.
- **af-south-1 (Cape Town)** is the closest AWS region to Nigeria. Useful for latency-sensitive applications targeting African users.

---

## Lab 4: Billing Budget Alert Setup

### Steps
1. Click account name → **Billing and Cost Management**
2. Navigate to **Budgets** in the left sidebar
3. Click **Create Budget**
4. Select **Zero Spend Budget** template
5. Enter a budget name (e.g. `day1-zero-spend-alert`)
6. Add your email address for notifications
7. Click **Create Budget** and confirm

### What I Observed
The Zero Spend Budget template automatically sets the threshold at $0.01 — meaning you get an alert the moment any charge appears on the account.

### What I Learned
- Setting a billing alert on Day 1 is non-negotiable. Free Tier limits can be exceeded accidentally (e.g. leaving an EC2 instance running).
- AWS Budgets can alert on cost, usage, or reservation coverage. For beginners, a Zero Spend Budget is the safest option.
- Budget alert emails are **not instant** — there can be a delay of several hours between a charge occurring and the notification arriving.
