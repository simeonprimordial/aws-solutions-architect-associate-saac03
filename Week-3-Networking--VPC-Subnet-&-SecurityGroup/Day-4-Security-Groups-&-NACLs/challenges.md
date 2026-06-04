# Challenges & Blockers — Week 3 Day 4: Security Groups & NACLs

---

## Challenge 1: chmod 400 Was Required Before SSH — Got Confusing Error

**What happened:**
After downloading `bastion-key.pem` and running the SSH command immediately, I got:
```
Permissions 0644 for 'bastion-key.pem' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "bastion-key.pem": bad permissions
Permission denied (publickey).
```
I initially thought the key pair was wrong or the EC2 instance had not fully started.

**What I tried:**
- Checked the EC2 instance state — it was `Running` with status checks passed.
- Re-read the error message: "permissions too open."
- Ran `ls -la bastion-key.pem` — output showed `-rw-r--r--` (644 = owner read/write, group read, others read).
- Ran `chmod 400 bastion-key.pem` — changed to `-r--------` (400 = owner read only).
- SSH succeeded immediately.

**Resolution:**
SSH deliberately refuses to use a private key file that is readable by other users or groups. It is a security precaution — if the key file is world-readable, it might have been compromised. `chmod 400` sets the file to owner-read-only, which SSH accepts. On Windows, the equivalent is using the file's security properties to grant access to only your user account.

**Lesson learned:**
`chmod 400 <keyfile>.pem` is the first command to run after downloading any AWS key pair on Mac/Linux. It is not optional. I now have this as a permanent step in my mental checklist before any SSH command. The error message is informative — always read the full error before assuming the problem is on the AWS side.

---

## Challenge 2: Ran ssh Before scp — Could Not Hop to Private Instance

**What happened:**
After successfully SSHing into the Bastion Host, I immediately tried to SSH from the Bastion to the private instance:
```
ssh -i bastion-key.pem ec2-user@10.0.10.x
```
The response was:
```
Warning: Identity file bastion-key.pem not accessible: No such file or directory.
Permission denied (publickey).
```
The key file did not exist on the Bastion Host — it only existed on my local laptop.

**What I tried:**
- Opened a second terminal tab on my local machine (keeping the Bastion session open in the first).
- Ran `scp` from local to Bastion:
  ```
  scp -i bastion-key.pem bastion-key.pem ec2-user@<BASTION-IP>:/home/ec2-user/
  ```
- Went back to the Bastion session, ran `chmod 400 bastion-key.pem`.
- SSH to private instance succeeded.

**Resolution:**
The `.pem` key exists on my local machine only. The Bastion Host is a fresh EC2 instance — it has no files from my local environment. The `scp` command uses the `-i` flag to authenticate to the Bastion (using the key from my local machine) while simultaneously copying the same key file to the Bastion's home directory.

**Lesson learned:**
The two-terminal approach is the cleaner way to do this lab: keep the Bastion session open in one tab, run `scp` from your local machine in a second tab. In production, SSH agent forwarding (`ssh -A`) is the correct approach — it lets the Bastion use the keys stored in your local SSH agent without ever placing the private key file on the Bastion's disk. Agent forwarding is more secure because the key never leaves your local machine.

---

## Challenge 3: Private Instance Security Group Source Was Confusing

**What happened:**
When creating `SG-PrivateInstance` for the private EC2, the lab guide said "Allow SSH from `SG-WebServers` only." I initially set the source to my home IP (`x.x.x.x/32`) by habit — the same source I used for the Bastion. This would mean SSH to the private instance is only allowed from my home IP, not from the Bastion.

**What I tried:**
- Attempted SSH from Bastion to private instance — got `Permission denied (publickey)` even though the key was correct.
- Re-read the Security Group rules: source was my home IP, not the Bastion's Security Group.
- Edited the inbound rule: changed source from `x.x.x.x/32` to Custom → `SG-WebServers`.
- SSH from Bastion succeeded immediately.

**Resolution:**
The SSH connection from the Bastion to the private instance originates from the Bastion Host — its source IP is the Bastion's private IP (`10.0.1.x`), not my home IP. My home IP is the source when I SSH to the Bastion. The Security Group for the private instance must reference `SG-WebServers` (the Bastion's SG) as the allowed source — not my home IP.

**Lesson learned:**
The source IP in a Security Group is always the IP of the machine making the connection — not the ultimate user. When traffic hops through the Bastion, the database/private instance sees the Bastion's IP as the source. This is the fundamental reason Security Group chaining (referencing SG IDs) is more reliable than IP-based rules for multi-hop architectures.

---

## Challenge 4: Custom NACL Blocked All Traffic Including the SSH Hop

**What happened:**
While working on the bonus NACL lab from Day 3 (retained for further testing in this session), I had a custom NACL associated with `Private-Subnet-App-AZ-A`. The custom NACL had no rules beyond the implicit `*` DENY. When I launched `Private-App-Server` into this subnet and tried the SSH hop from the Bastion, the connection timed out — not a `Permission denied`, just a hanging connection that eventually timed out.

**What I tried:**
- Checked the Security Group on `Private-App-Server` — rules were correct (SSH from `SG-WebServers`).
- Checked the route table — correct (main route table, local only).
- Checked VPC Flow Logs — found `REJECT` entries at the subnet level for incoming TCP packets on port 22.
- Identified: the custom NACL was rejecting the inbound SSH traffic before it reached the Security Group.
- Added NACL inbound rule 100: Allow TCP 22 from `10.0.1.0/24` (public subnet CIDR where Bastion lives).
- Added NACL outbound rule 100: Allow TCP `1024–65535` to `10.0.1.0/24` (ephemeral responses back to Bastion).
- SSH hop worked immediately after.

**Resolution:**
A custom NACL starts with DENY ALL — both inbound and outbound. Even though the Security Group allowed SSH from the Bastion, the NACL dropped the packet at the subnet boundary before the Security Group was ever evaluated. The timeout (rather than immediate `Permission denied`) is characteristic of a NACL block — the TCP connection attempt is silently dropped, and the SSH client waits until its timeout period before reporting failure.

**Lesson learned:**
The two-layer diagnostic approach: if traffic is timing out (not getting a `Permission denied` or `Connection refused`), the NACL is the more likely culprit. `Permission denied` usually means the Security Group allowed the connection but authentication failed. A hang-then-timeout usually means the packet is being dropped before it reaches the instance. Check the NACL first when you see timeouts. VPC Flow Logs confirm which layer is rejecting.

---

## Challenge 5: Understanding Why Intermittent Timeouts in the Interswitch Scenario

**What happened:**
The slides described an Interswitch finding where the custom NACL had correct inbound rules for port 5432 but no outbound ephemeral port rule — causing intermittent database connection timeouts. I initially did not understand why the timeouts would be intermittent rather than total. If the outbound NACL blocks all responses, shouldn't every connection fail?

**What I tried:**
- Thought through the TCP connection lifecycle.
- Researched how TCP ephemeral ports work: the client OS picks a random port in `1024–65535` for each new connection.
- Considered: what if the client's randomly chosen ephemeral port happens to be allowed by a NACL rule for a different reason? e.g. if there was a NACL rule allowing outbound TCP on port 5432 for some reason, and the client's ephemeral port happened to be 5432 (extremely rare but possible), that one connection might succeed.

**Resolution:**
The intermittency in practice comes from a more realistic scenario: the custom NACL had some outbound rules that were added for other purposes — perhaps allowing TCP outbound on specific ports for monitoring or logging. Occasionally, a connection from the application server would use an ephemeral port that happened to fall within a range covered by one of those unrelated outbound rules. Those connections succeeded. Connections where the ephemeral port fell outside any allowed outbound rule range failed.

This is also why ephemeral timeouts are so hard to diagnose without VPC Flow Logs: the failure rate depends on which random ephemeral ports the OS picks, making it look like an intermittent application bug rather than a network configuration issue.

**Lesson learned:**
Intermittent network failures in AWS that do not reproduce consistently → always check NACLs and look for partial outbound rules. The correct fix is always to add TCP `1024–65535` outbound to the NACL — covering the full ephemeral range rather than relying on incidental coverage from other rules.

---

*Add new challenges here as they come up in future days.*
