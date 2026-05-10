# Week 1 - Day 3: The Shared Responsibility Model

---

## 📌 Overview

The **Shared Responsibility Model** defines who is responsible for security in the cloud. It is one of the most important and heavily tested topics in the SAA-C03 exam.

Customer misconfiguration is the **#1 cause** of cloud breaches (e.g., 2019 Capital One breach).

> **Key Message**:  
> AWS secures the **cloud** (physical infrastructure).  
> You secure **your data and applications** in the cloud.

---

## 🔑 Key Concepts

| Concept                    | Definition |
|---------------------------|----------|
| **Security OF the Cloud** | AWS responsibility — physical facilities, hardware, virtualization, global network. |
| **Security IN the Cloud** | Customer responsibility — OS patches, application security, data classification, IAM, encryption. |
| **Managed Services Shift** | The more managed the service (e.g., Lambda, RDS), the more responsibility AWS takes. |
| **Shared Controls**       | Responsibilities shared by both (e.g., patch management, configuration management). |
| **Inherited Controls**    | Customers automatically inherit AWS’s physical and environmental controls. |

---

## ⚙️ How It Works

1. **AWS** is responsible for the physical infrastructure and virtualization layer.
2. **Customer** is responsible for everything they deploy and configure on top.
3. Responsibility **shifts** based on the service model:
   - **IaaS (EC2)** → Customer manages more
   - **PaaS/SaaS (RDS, Lambda)** → AWS manages more

---

## 🛡️ Architecture & Visual Model

- **AWS Responsibility** (Security OF the Cloud)
- **Shared Responsibility Zone**
- **Customer Responsibility** (Security IN the Cloud)

**Best Practice**: Always understand the responsibility boundary for every AWS service you use.

---

## 🌍 Real-World Nigerian Scenario

A healthcare startup in Lagos stores patient records in Amazon S3.  
- **AWS** ensures the physical hard drives are in secure, monitored data centers.  
- **Startup** must encrypt the data, set proper IAM bucket policies, enable logging, and avoid making the bucket public.

---

## ⚠️ Common Mistakes & Exam Traps

- AWS is **always** responsible for physical infrastructure.
- For **RDS**: AWS patches the OS and database engine, but the customer manages data, access permissions, and encryption.
- **Shared Controls** (patch management, configuration) often appear in exam questions.

---

## 💡 Key Takeaways

- Understand the difference between **Security OF the Cloud** and **Security IN the Cloud**.
- Responsibility changes depending on the service (EC2 vs RDS vs Lambda).
- Always configure your resources securely — AWS will not fix customer misconfigurations.

**Next**: Continue to Day 4