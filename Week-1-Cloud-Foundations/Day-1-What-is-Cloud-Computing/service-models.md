# Cloud Service Models — Day 1 Notes

---

## IaaS — Infrastructure as a Service

Provides virtualised computing resources over the internet. You manage the OS, runtime, and applications. The provider manages the physical hardware and hypervisor.

**You control:** OS, middleware, runtime, data, applications
**Provider controls:** Servers, storage, networking, virtualisation

**AWS Examples:**
- EC2 (virtual machines)
- S3 (raw object storage)
- VPC (virtual networking)

**When to use:** When you need maximum control over your environment.

---

## PaaS — Platform as a Service

Provides a managed platform for deploying applications. You focus on writing code — the provider manages the underlying infrastructure, OS, and runtime.

**You control:** Applications and data
**Provider controls:** Everything below the application layer

**AWS Examples:**
- Elastic Beanstalk
- RDS (managed databases)
- Lambda (serverless functions)

**When to use:** When you want to deploy fast without managing servers.

---

## SaaS — Software as a Service

Fully managed software applications delivered over the internet. You just use the software — the provider handles everything.

**You control:** Your data and user settings
**Provider controls:** Everything else

**Examples:**
- Gmail
- Dropbox
- Google Drive
- Salesforce

**When to use:** When you need ready-to-use software with no setup.

---

## Quick Comparison

| Model | You Manage | Provider Manages | Control Level |
|---|---|---|---|
| IaaS | OS → App | Hardware, Network | High |
| PaaS | App only | OS → Hardware | Medium |
| SaaS | Nothing | Everything | Low |

> ⚠️ **Exam Tip:** Know which AWS services fall into each category. EC2 = IaaS. Elastic Beanstalk = PaaS. WorkMail = SaaS.
