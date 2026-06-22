# Security Groups & NACLs — Week 3 Day 4

## Topic

The two-layer firewall system inside every VPC — Security Groups at the instance level and Network ACLs at the subnet level — and the Bastion Host pattern for secure SSH access to private resources.

This is the day where the full security model for a VPC comes together. The slides go deep on everything the previous three days touched on lightly: the stateful vs stateless distinction with its real-world consequence (ephemeral ports), the evaluation mechanics that make NACL rule order critical, Security Group referencing as the professional standard, and the Interswitch audit scenario showing what three common misconfigurations look like in a regulated Nigerian payment environment. The lab then makes it concrete by launching a Bastion Host in the public subnet, SSHing into it, and hopping to a private subnet instance that has no public IP — exactly the pattern any production team uses to administer private resources without exposing them to the internet.

## What I Learned

- **Security Group — Stateful, Instance-Level, Allow Only** — Security Groups are attached to ENIs (Elastic Network Interfaces) of individual EC2 instances, RDS databases, and Lambda functions. They are stateful: once an inbound connection is established, the return traffic is automatically allowed — no outbound rule needed. They support allow rules only. You cannot write an explicit deny in a Security Group.
- **Security Group Defaults** — Default inbound: deny all (nothing enters until you add allow rules). Default outbound: allow all. A newly created Security Group blocks all inbound and allows all outbound.
- **Security Group Referencing** — Instead of using an IP CIDR as the source, reference another Security Group ID. `SG-Database` allows port 5432 from `sg-app-servers`. Any EC2 instance carrying `sg-app-servers` gets access — including new instances added by Auto Scaling. IPs change; Security Group membership is stable.
- **NACL — Stateless, Subnet-Level, Allow AND Deny** — NACLs sit at the subnet boundary and evaluate every packet independently — they track no connection state. Rules are evaluated in ascending number order and the first matching rule wins. They support both allow and deny rules.
- **NACL Default vs Custom** — Default NACL (auto-created with VPC): allows all inbound AND all outbound. Custom NACL (you create): denies all inbound AND all outbound — only the implicit `*` deny rule exists. You must add every allow rule yourself.
- **Ephemeral Ports — The NACL Trap** — When a client connects to a server, the client OS picks a random source port in the range `1024–65535` for the server's response to return on. NACLs are stateless — they do not remember the connection. If your NACL outbound rules do not explicitly allow TCP `1024–65535`, response traffic is silently dropped at the subnet boundary. The request arrives, the Security Group allows it, the instance processes it — and the response disappears. The client sees a timeout. Security Groups never have this problem because they are stateful.
- **Traffic Path Through Both Layers** — NACL inbound → Security Group inbound → Instance processes → Security Group outbound (automatic) → NACL outbound (explicit). ALL layers must allow. A single reject at any layer silently drops the traffic.
- **Bastion Host Pattern** — A Bastion Host is a hardened EC2 instance in a public subnet with SSH (port 22) open from specific IPs only. Engineers SSH into the Bastion, then SSH from the Bastion to private subnet instances. The private instances have no public IP and are unreachable from the internet directly — they accept SSH only from the Bastion's Security Group.
- **AWS Systems Manager Session Manager** — The modern replacement for Bastion Hosts. No open SSH port (port 22 never opened on any instance). No key management. No Bastion EC2 to maintain. Access is browser or CLI-based, fully logged to CloudTrail, and works through IAM permissions rather than network-level access.
- **VPC Flow Logs for Troubleshooting** — REJECT at the subnet level = NACL is blocking. REJECT at the ENI level = Security Group is blocking. Flow Logs show ACCEPT/REJECT decisions per packet — they are the diagnostic tool for any connectivity issue where the Security Group looks correct but traffic is still failing.

## Hands-On Labs Completed

- Launched `Bastion-Host` EC2 (Amazon Linux 2, t2.micro) in `Public-Subnet-AZ-A` with public IP enabled, using `SG-WebServers` (SSH from own IP on port 22)
- Launched `Private-App-Server` EC2 (same AMI and type) in `Private-Subnet-App-AZ-A` with public IP disabled and a new Security Group allowing SSH only from `SG-WebServers`
- SSH'd into the Bastion Host: `ssh -i bastion-key.pem ec2-user@<BASTION-PUBLIC-IP>`
- Copied `.pem` key to Bastion via `scp`, then SSH'd from Bastion to Private-App-Server using its private IP
- Confirmed Private-App-Server hostname and private IP from inside the session
- Terminated both instances after completing the lab to avoid charges
- **Bonus:** Researched AWS Systems Manager Session Manager and documented 3 advantages over the Bastion Host pattern

## AWS Services & Features Used

- EC2 (Amazon Linux 2, t2.micro)
- Security Groups
- Network ACLs (NACLs)
- Key Pairs (RSA .pem)
- VPC Bastion Host Pattern
- AWS Systems Manager Session Manager (bonus research)
- VPC Flow Logs (conceptual — troubleshooting)

## Screenshots

- `/screenshots/both-instances-running.png` — EC2 console showing `Bastion-Host` with public IP and `Private-App-Server` with private IP only
- `/screenshots/bastion-ssh-success.png` — Terminal prompt showing successful SSH connection to `Bastion-Host` (`ec2-user@ip-10-0-1-x`)
- `/screenshots/private-hop-success.png` — Terminal prompt showing SSH hop from Bastion to `Private-App-Server` private IP, with `hostname` and `whoami` output confirming different instance
- `/screenshots/instances-terminated.png` — EC2 console showing both instances in `terminated` state

## Challenges & Blockers

See `notes/challenges.md` for full details.

- The `.pem` key needed `chmod 400` before SSH would work — permissions too open causes an immediate SSH error
- SCP to copy the key to the Bastion added an unexpected step I had not planned for — the lab guide calls it out but I still ran `ssh` before `scp` first
- Creating the Security Group for `Private-App-Server` to accept SSH from `SG-WebServers` (the Bastion SG) rather than my IP took a moment — the source is the Bastion's Security Group ID, not my home IP
- Custom NACL on private subnet initially blocked all traffic including the SSH hop — forgot to add the ephemeral port outbound rule

## Key Exam Traps to Remember

- **Security Groups have NO explicit deny.** To block a specific IP address, use a NACL deny rule. Security Groups only allow.
- **NACL rules are evaluated in number order — lower number wins.** A DENY at rule 90 blocks traffic even if ALLOW exists at rule 100. The allow is never reached.
- **Custom NACLs start with DENY ALL.** The default NACL allows everything. A custom NACL starts from zero. Creating one without adding rules immediately breaks all traffic to the subnet.
- **Ephemeral ports `1024–65535` must be in NACL outbound rules.** Missing this causes silent response timeouts — requests arrive, responses are dropped. The most common NACL misconfiguration in production.
- **Check both SG AND NACL when troubleshooting.** Either can reject. Flow Logs show REJECT at subnet level (NACL) vs ENI level (SG).

## Goal

Passing the **AWS Certified Solutions Architect Associate (SAA-C03)**.
