# Week 4 - Day 1: EC2 Introduction & Launch

---

## 📌 Overview

**Amazon EC2 (Elastic Compute Cloud)** is AWS’s core virtual server service. It allows you to launch virtual machines in the cloud in under 90 seconds, eliminating the need to buy and manage physical hardware.

EC2 is the foundation for most AWS workloads — web servers, APIs, batch jobs, databases (on EC2), and many other services run on it.


---

## 🔑 Core Concepts

| Concept              | Description |
|----------------------|-----------|
| **EC2 Instance**     | Virtual server (CPU + RAM + Network) running in a VPC subnet |
| **AMI**              | Template (OS + software). Sources: AWS-provided, Marketplace, Custom |
| **Instance Type**    | Defines CPU, RAM, storage & networking (e.g., `t3.micro`) |
| **Key Pair**         | SSH authentication (.pem file). Private key must be kept secure |
| **Security Group**   | Virtual firewall for your instance |
| **EBS Volume**       | Persistent block storage (survives stop/start) |
| **Elastic IP**       | Static public IP (normal public IP changes on stop/start) |
| **User Data**        | Script that runs automatically on first boot |

### Instance Families (Summary)
- **General Purpose (t, m)**: Balanced — web servers, dev environments
- **Compute Optimized (c)**: High CPU — batch, gaming, ML inference
- **Memory Optimized (r, x)**: High RAM — databases, caching
- **Storage Optimized (i, d)**: High disk I/O
- **Accelerated (p, g)**: GPU workloads

---

## 💰 EC2 Pricing Models

| Pricing Model       | Discount     | Best For                              | Risk |
|---------------------|--------------|---------------------------------------|------|
| **On-Demand**       | 0%           | Dev/test, short-term, unpredictable   | Most expensive |
| **Reserved Instances** | Up to 72%   | Steady 24/7 workloads                 | Commitment |
| **Spot Instances**  | Up to 90%    | Fault-tolerant, interruptible jobs    | Can be terminated |
| **Dedicated Hosts** | None         | Compliance / licensed software        | Most expensive |

---

## ⚙️ How It Works — Launch Steps

1. Choose **AMI** (OS template)
2. Choose **Instance Type**
3. Configure **Network** (VPC + Subnet) and **Security Group**
4. Configure **Storage** (EBS) + **User Data** script
5. Select **Key Pair** → Launch

---

## 🛠️ Important Operational Differences

- **Stop** → Instance stops (save compute cost), public IP changes, EBS persists
- **Terminate** → Instance and EBS (by default) are deleted permanently
- **Reboot** → Everything stays the same
- **Instance Store** → Very fast but data is **lost** on stop/terminate

---

## 🌍 Real-World Nigerian Scenario

A fintech payment platform uses:
- `t3.medium` + Custom AMI for application servers (Auto Scaling)
- 1-year Reserved Instances for baseline load
- Spot Instances for overnight batch processing
- Spread Placement Group for high availability

---

## Key Take Away

- Public IPs change on **Stop/Start** → Use Elastic IP for static address
- EBS persists on Stop, but is deleted on Terminate (unless disabled)
- Instance Store data is lost on Stop/Terminate
- User Data runs **only once** at first launch

---