# Week 4 - Day 1: Lab - Launch, Configure & Connect to Your First EC2 Instance

**Objective**: Launch a real web server on EC2 and access it from the internet.

---

## ✅ Prerequisites
- Custom VPC + Subnets from Week 3
- Security Group knowledge
- SSH client installed

---

## Lab Steps

### Step 1: Launch the EC2 Instance
1. Go to **EC2 → Launch Instance**
2. Name: `Web-Server-Lab`
3. AMI: **Amazon Linux 2** (Free Tier)
4. Instance Type: `t2.micro` or `t3.micro`
5. Key Pair: Create new (`web-server-key.pem`)
6. Network: Your custom VPC → Public Subnet → Auto-assign Public IP: Enable
7. Security Group: Allow **HTTP (80)** from anywhere + **SSH (22)** from your IP
8. User Data: Paste the Apache installation script
9. Launch

### Step 2: Monitor & Connect
- Wait until instance is **Running**
- SSH into it: `chmod 400 web-server-key.pem` then `ssh -i web-server-key.pem ec2-user@<Public-IP>`
- Verify Apache is running

### Step 3: Verify Web Server
- Open browser → `http://<Your-EC2-Public-IP>`
- You should see your custom "Hello from EC2!" page

### Step 4: Explore Instance Metadata
- Inside the instance, run:
  ```bash
  curl http://169.254.169.254/latest/meta-data/
  curl http://169.254.169.254/latest/meta-data/public-ipv4