# Labs — Week 3 Day 4: Security Groups & NACLs

---

## Lab 1: Launch the Bastion Host in the Public Subnet

### Steps

1. Navigate to **EC2 → Instances → Launch Instance**.
2. Name: `Bastion-Host`.
3. AMI: **Amazon Linux 2** (Free Tier eligible).
4. Instance type: `t2.micro` (Free Tier).
5. Key pair: Click **Create new key pair**.
   - Key pair name: `bastion-key`
   - Key pair type: RSA
   - Private key file format: `.pem`
   - Click **Create key pair** — browser downloads `bastion-key.pem` immediately. Save it somewhere you will not lose it. There is no way to download it again.
6. **Network settings → Edit:**
   - VPC: `OluTech-Production-VPC`
   - Subnet: `Public-Subnet-AZ-A`
   - Auto-assign public IP: **Enable**
7. Firewall (security groups): Select existing security group → `SG-WebServers` (has SSH port 22 from your IP).
8. Click **Launch Instance**.
9. Wait for state to show `Running` and status checks to pass. Note the Public IPv4 address.

### What I Observed

The key pair download is a one-shot operation — AWS stores only the public key and there is no mechanism to recover the private key after this window. I saved `bastion-key.pem` to a dedicated project folder immediately. The instance reached `Running` state in about 90 seconds. The public IP was assigned immediately on launch because `Public-Subnet-AZ-A` has auto-assign enabled and the instance is in a public subnet with an IGW route.

The EC2 console shows the Public IPv4 address in the instance summary. The Private IPv4 address is also shown — it comes from the `10.0.1.x` range (the public subnet CIDR), confirming the placement is correct.

### What I Learned

- Key pairs are created once. The private key (`.pem`) is downloaded immediately and never stored by AWS. Lose it and you lose SSH access to that instance permanently — unless you have Session Manager configured or can stop the instance and replace the key via a workaround.
- `t2.micro` in a Free Tier account costs nothing for the first 750 instance-hours per month. But Free Tier or not, always terminate instances after a lab to prevent unexpected charges.
- The Bastion Host is the only instance in the entire VPC that needs a public IP. Every other resource sits behind it.

---

## Lab 2: Launch the Private Instance

### Steps

1. Navigate to **EC2 → Launch Instance**.
2. Name: `Private-App-Server`.
3. AMI: Amazon Linux 2.
4. Instance type: `t2.micro`.
5. Key pair: **Select existing** → `bastion-key` (same key — required for the SSH hop from Bastion).
6. **Network settings → Edit:**
   - VPC: `OluTech-Production-VPC`
   - Subnet: `Private-Subnet-App-AZ-A`
   - Auto-assign public IP: **Disable**
7. Firewall: **Create security group**.
   - Name: `SG-PrivateInstance`
   - Inbound rule: SSH | TCP | 22 | Source: Custom → select `SG-WebServers` (the Bastion's SG)
   - No other inbound rules. No internet access.
8. Click **Launch Instance**.
9. Note: the instance has no Public IPv4 address in the console — only a Private IPv4 (`10.0.10.x`). This confirms private subnet placement.

### What I Observed

The console explicitly shows no public IP — the field is empty or shows a dash. This is correct. `Private-Subnet-App-AZ-A` has auto-assign disabled and no IGW route, so no public IP is allocated. The private IP is in the `10.0.10.x` range as expected.

The Security Group for this instance (`SG-PrivateInstance`) references `SG-WebServers` as the SSH source — not my laptop's IP, not `0.0.0.0/0`. This is Security Group chaining applied to SSH: only the Bastion Host (which carries `SG-WebServers`) can initiate an SSH connection to this instance. Even if someone has the `bastion-key.pem` file, they cannot SSH to `Private-App-Server` directly from the internet — the Security Group rejects any connection not originating from an instance in `SG-WebServers`.

### What I Learned

- Using the same key pair (`bastion-key`) on both instances is intentional for this lab. In production, private instances would have their own separate key pairs — you would use SSH agent forwarding rather than copying a key to the Bastion.
- Auto-assign public IP disabled means the instance never gets a public IP regardless of any other setting. No public IP = no direct internet access path, even if someone misconfigures the route table later.
- The Security Group chaining for SSH (`SG-PrivateInstance` allows port 22 from `SG-WebServers`) is the professional pattern. It means the Bastion is not just a recommended access path — it is the enforced, only possible path.

---

## Lab 3: SSH into the Bastion Host

### Steps

**On Mac/Linux:**
1. Open Terminal.
2. Set correct permissions on the key file:
   ```
   chmod 400 ~/path/to/bastion-key.pem
   ```
3. SSH to the Bastion using its Public IPv4 address:
   ```
   ssh -i ~/path/to/bastion-key.pem ec2-user@<BASTION-PUBLIC-IP>
   ```
4. Accept the host key fingerprint prompt (type `yes`).
5. You should see the Amazon Linux welcome banner and the prompt:
   ```
   [ec2-user@ip-10-0-1-x ~]$
   ```

**On Windows:**
1. Use Windows Terminal with OpenSSH, or PuTTY with the `.pem` converted to `.ppk` using PuTTYgen.
2. Command is the same in Windows Terminal: `ssh -i bastion-key.pem ec2-user@<BASTION-PUBLIC-IP>`.
6. Take a screenshot of the terminal prompt.

### What I Observed

Without `chmod 400` first, SSH refuses to run and prints:
```
Permissions 0644 for 'bastion-key.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
```
This is a security feature of SSH — if the private key file is readable by others, SSH refuses to use it. `chmod 400` sets the permissions to owner-read-only.

After setting permissions and running the SSH command, the connection was immediate. The `ip-10-0-1-x` in the prompt confirmed I was inside the Bastion Host (`10.0.1.x` is the public subnet CIDR). The Bastion has no application workload — just a shell prompt. Its only purpose is to be the jump point.

### What I Learned

- `chmod 400` is required before any SSH command with a `.pem` file on Mac/Linux. Running SSH before this step always fails.
- The `ec2-user` username is standard for Amazon Linux 2. Other AMIs use different usernames: `ubuntu` for Ubuntu, `admin` for Debian, `centos` for CentOS.
- The host key fingerprint prompt on first connection is normal — type `yes` to add the host to `known_hosts`. If you see this prompt on a subsequent connection to the same IP, something has changed and it may indicate a security event.

---

## Lab 4: SSH Hop from Bastion to Private Instance

### Steps

1. From your local terminal, copy `bastion-key.pem` to the Bastion Host:
   ```
   scp -i ~/path/to/bastion-key.pem ~/path/to/bastion-key.pem ec2-user@<BASTION-PUBLIC-IP>:/home/ec2-user/
   ```
2. In the terminal session on the Bastion Host, set correct permissions:
   ```
   chmod 400 bastion-key.pem
   ```
3. Get the Private IPv4 address of `Private-App-Server` from the EC2 console (e.g. `10.0.10.x`).
4. From the Bastion terminal, SSH to the private instance using its private IP:
   ```
   ssh -i bastion-key.pem ec2-user@<PRIVATE-APP-SERVER-PRIVATE-IP>
   ```
5. You are now inside a private subnet instance — no internet access, no public IP.
6. Confirm with:
   ```
   hostname
   whoami
   ```
7. Take a screenshot showing the private IP prompt and command output.

### What I Observed

The `scp` command looks almost identical to `ssh` but copies a file to the remote host. After copying, I needed `chmod 400` again on the Bastion for the copied key — permissions are not preserved across `scp` by default.

The SSH command from the Bastion used the private IP (`10.0.10.x`) rather than a public IP. This is VPC-internal routing — the connection goes from the Bastion (`10.0.1.x` in the public subnet) to the private instance (`10.0.10.x` in the private subnet) entirely via the `local` route. No internet involved, no NAT Gateway, no IGW. Just private VPC-internal routing.

The prompt changed from `[ec2-user@ip-10-0-1-x ~]$` to `[ec2-user@ip-10-0-10-x ~]$` — confirming I had successfully hopped to the private subnet. The `hostname` command returned `ip-10-0-10-x.eu-west-1.compute.internal` — the internal DNS name, not a public name. `whoami` returned `ec2-user`.

To prove isolation: from the private instance, I tried `curl https://google.com`. It timed out. This confirmed the private subnet has no outbound internet access (the `Private-Subnet-App-AZ-A` is on the main route table with only the `local` route — no NAT Gateway).

### What I Learned

- The SSH agent forwarding approach (`ssh -A`) is more secure in production than copying the private key to the Bastion. It allows the Bastion to use the key without it ever being stored on the Bastion's disk. The lab uses the simpler `scp` method for clarity.
- The two-hop prompt change is the proof. When the IP in the prompt changes from the public subnet range to the private subnet range, you know the hop worked.
- Testing `curl` from the private instance and seeing it fail confirmed the isolation was real — not just theoretical. This is the kind of verification that belongs in a deployment runbook.

---

## Lab 5: Clean Up — Terminate Both Instances

### Steps

1. Navigate to **EC2 → Instances**.
2. Select both `Bastion-Host` and `Private-App-Server`.
3. Click **Instance State → Terminate Instance**.
4. Confirm termination.
5. Verify both instances show state `Terminated` within a few minutes.
6. Screenshot the terminated state for portfolio evidence.

### What I Observed

Termination is immediate and irreversible. Terminated instances remain visible in the console for a few hours in `terminated` state before disappearing entirely. The Elastic IP was not allocated for this lab (the Bastion used an auto-assigned IP) so there is no EIP to release. The Security Groups created in this lab are not automatically deleted when instances terminate — they stay associated with the VPC.

### What I Learned

- Termination vs stopping: stopping an instance pauses billing for compute but leaves EBS volumes running (billed separately). Termination deletes the instance and its EBS root volume by default. For a lab with no data worth preserving, terminate always.
- Terminated instances do not generate charges. Running or stopped instances with attached EBS volumes do. Clean up after every lab.
- The `SG-PrivateInstance` Security Group created in this lab persists after termination. Future instances in the same VPC can reuse it.

---

## Bonus Lab: AWS Systems Manager Session Manager Research

### Steps

1. Read the AWS Systems Manager Session Manager documentation.
2. Understand how it provides shell access to EC2 instances without SSH.
3. Write 3 advantages over the Bastion Host pattern.

### What I Observed

Session Manager uses the SSM Agent (pre-installed on Amazon Linux 2) and an IAM instance profile to establish a secure channel from the AWS control plane to the instance. The engineer opens a session from the AWS console browser, the AWS CLI (`aws ssm start-session --target i-xxxxxxxx`), or the AWS Management Console — no SSH client needed.

The session runs inside the browser or CLI entirely over HTTPS/port 443, not SSH/port 22. The instance never needs port 22 opened. There is no Bastion Host to manage, patch, or pay for.

**Three advantages over Bastion Host:**

1. **No open port 22 on any instance.** Port 22 is never opened — not on the Bastion, not on any private instance. The entire SSH-based attack surface is eliminated. The Interswitch brute-force attack (14,000 attempts in 48 hours on SSH) would have had zero targets with Session Manager.

2. **No key pair management.** No `.pem` files to create, distribute, lose, or rotate. Access is governed entirely by IAM permissions (`ssm:StartSession` on the target instance). Revoking someone's access is an IAM policy change — not hunting down and rotating SSH keys across every instance they ever accessed.

3. **Automatic, complete audit trail.** Every session is logged to AWS CloudTrail (who started which session, when, from which IP). Session content can optionally be logged to S3 or CloudWatch Logs — a complete record of every command run. This exceeds the audit capability of any SSH-based approach and directly satisfies PCI-DSS and CBN audit trail requirements.

### What I Learned

- Session Manager is the AWS-recommended approach for EC2 shell access from 2020 onward. The Bastion Host pattern is still widely used and important to understand (it appears in interview questions and the SAA-C03 exam), but in any greenfield deployment, Session Manager is the correct choice.
- The trade-off: Session Manager requires the SSM Agent on the instance and an IAM instance profile with `AmazonSSMManagedInstanceCore`. For fully isolated instances (no internet, no NAT), a VPC Interface Endpoint for Systems Manager is also required so the agent can reach the SSM service.
- For the portfolio: document both patterns. The ability to explain why you would choose Session Manager over a Bastion Host demonstrates a senior-level security mindset.
