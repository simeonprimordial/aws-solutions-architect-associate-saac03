# Week 1 - Day 3: Lab - Draw & Explain the Shared Responsibility Model

**Objective**: Create a clear visual diagram and explanation of the Shared Responsibility Model.

---

## ✅ Prerequisites
- Microsoft Word or Excel (recommended)
- Access to [AWS Shared Responsibility Model page](https://aws.amazon.com/compliance/shared-responsibility-model/)

---

## Lab Steps

### Step 1: Understand the Model
- Read the official AWS Shared Responsibility Model page.
- Write definitions in your own words for:
  - Security **OF** the Cloud
  - Security **IN** the Cloud

### Step 2: Categorize Responsibilities
Create a table and categorize the following:

**AWS Responsibility**:
- Physical server hardware
- Data centre cooling & power
- Network firewall between AZs
- Hypervisor (virtualization layer)
- RDS database backups (managed)

**Customer Responsibility**:
- EC2 OS patches
- S3 bucket access policy
- IAM user passwords & MFA
- Encrypting data in S3
- SSL certificate on web app
- MySQL backups on EC2

**Shared Controls**:
- Patch Management
- Configuration Management
- Security Awareness Training

### Step 3: Build Visual Diagram
- Create a 3-column table: **AWS** | **Shared** | **Customer**
- Add title: "AWS Shared Responsibility Model"

---

## 📸 Portfolio Deliverables
- Completed diagram (PDF/screenshot)
- Categorization table
- 3-sentence plain-English explanation

