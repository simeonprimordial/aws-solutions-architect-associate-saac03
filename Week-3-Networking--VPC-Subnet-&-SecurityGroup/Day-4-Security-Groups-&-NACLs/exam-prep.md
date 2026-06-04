# Exam Prep — Week 3 Day 4: Security Groups & NACLs

## SAA-C03 Context

Security Groups and NACLs are tested in Domain 2 (Design Secure Architectures, ~26%) and appear in scenario questions across every other domain. Domain 2, Task Statement 2.2 explicitly covers designing secure network architectures — Security Group referencing, NACL rule design, and the stateful/stateless distinction are all named in the task statement. The exam typically has 2–4 questions specifically about Security Group and NACL behaviour. More than that, SG and NACL configuration appear as secondary considerations in almost every multi-service architecture question — the wrong Security Group choice makes an otherwise correct answer wrong. Knowing this topic cold is a multiplier across the entire exam.

---

## Exam Traps — Deep Explanations

### Trap 1: Security Groups have no explicit deny — NACLs do

The exam presents a scenario: "An attacker is continuously scanning port 22 from IP `203.x.x.x`. How do you block it?" The distractor answer: "Add a Deny rule to the Security Group blocking `203.x.x.x/32`." This is impossible. Security Groups have no Deny rule type. The correct answer: "Add a NACL Deny rule for TCP port 22 from `203.x.x.x/32` at rule number lower than the existing Allow rule." This also reinforces that NACL Deny blocks specific IPs — the exact use case NACLs are designed for at the subnet level.

### Trap 2: NACL rule evaluation order — lower number wins, first match stops

The exam presents NACL rules out of order and asks what happens. The critical skill is reading the rule numbers, not the rule content. A DENY rule at number 90 beats an ALLOW rule at number 100 for the same traffic — the DENY fires first and evaluation stops. This is the exact opposite of Security Groups (which evaluate all rules simultaneously and take the most permissive). Exam questions present a NACL configuration and ask why traffic is unexpectedly blocked or unexpectedly allowed — always trace the rules in numerical order from lowest to highest.

### Trap 3: Custom NACL starts DENY ALL — Default NACL allows ALL

Two facts that look like one but are opposites. The Default NACL (auto-created with the VPC) allows all inbound AND all outbound — it is effectively transparent and provides no security. A Custom NACL (created by you) starts with DENY ALL — only the implicit `*` DENY rule exists. Creating a custom NACL and associating it with a subnet without adding explicit Allow rules immediately breaks ALL traffic to that subnet. This is the most common cause of "everything stopped working after we added a custom NACL." The exam presents a scenario where a custom NACL was added and traffic fails — the answer is: add the missing Allow rules including ephemeral ports.

### Trap 4: Ephemeral ports `1024–65535` in NACL outbound rules

This is the single most tested NACL behaviour. The full sequence: client (EC2 app server) connects to server (RDS on port 5432) → request goes INBOUND to the data subnet → NACL inbound rule allows TCP 5432 → reaches RDS Security Group → RDS processes query → RDS sends response → response goes OUTBOUND from the data subnet → the response's destination port is the app server's random ephemeral port (e.g. 54312) → if NACL outbound only allows TCP 5432, the response on port 54312 is dropped → app server sees a connection timeout.

The symptom (timeout, not connection refused) and the solution (add outbound TCP `1024–65535`) are both exam-tested. The specific OS ephemeral port ranges (Linux: `32768–60999`, Windows: `49152–65535`) may appear — the safe exam answer is always `1024–65535` which covers all platforms.

### Trap 5: Check BOTH layers — use VPC Flow Logs to identify which

The exam presents a scenario where traffic is failing and asks which component is blocking it. The Security Group rules look correct. The answer requires identifying that the NACL on the subnet has a missing or incorrect rule. Without knowing to check both layers — and knowing which symptom corresponds to which layer — this question type is impossible to answer correctly.

The diagnostic key: REJECT at the subnet/ENI for the inbound packet = NACL blocking before the SG. REJECT at the ENI level after the packet entered = SG blocking. VPC Flow Logs show ACCEPT/REJECT at both levels. The exam may describe Flow Log output and ask what it indicates — REJECT on inbound at subnet level = NACL. REJECT on inbound at ENI level = SG.

---

## Architecture Decision Table

| Scenario | Correct Solution |
|---|---|
| Block a specific attacker IP from reaching a subnet | NACL Deny rule with the attacker IP and a lower rule number than any allow |
| Allow only the ALB to reach EC2 web servers | Security Group on EC2: inbound from `SG-LoadBalancer` (SG reference) |
| Allow only app servers to reach RDS | Security Group on RDS: inbound TCP 5432 from `sg-appservers` (SG reference) |
| SSH access to private instances only via Bastion | SG on private instances: inbound TCP 22 from `sg-bastion` only |
| No SSH keys, no open port 22, full audit trail for admin access | AWS Systems Manager Session Manager |
| Custom NACL applied, all traffic to subnet stopped | Add inbound and outbound ALLOW rules including TCP `1024–65535` |
| Database response timeouts even though SG and NACL inbound are correct | Missing NACL outbound TCP `1024–65535` allow rule (ephemeral ports) |
| NACL rule 90 DENY and rule 100 ALLOW for same traffic — what happens? | Traffic is DENIED — rule 90 fires first. ALLOW at 100 never reached. |
| Block Telnet (port 23) from reaching any instance in a subnet | NACL DENY rule for TCP 23 — SGs cannot deny |
| New Auto Scaling EC2 instances should automatically get DB access | Security Group referencing: DB SG allows `sg-appservers` — new ASG instances inherit SG automatically |
| Troubleshoot: request arrives but no response, connection times out | Check NACL outbound ephemeral port rule — likely missing `1024–65535` |

---

## Practice Question

**A solutions architect at a Lagos fintech has set up a VPC with a public subnet and a private subnet. The private subnet has a custom NACL applied. An EC2 application server in the public subnet needs to connect to an RDS database in the private subnet on port 5432. The security team has added the following NACL rules:**

```
Inbound Rule 100:  Allow TCP 5432 from 10.0.1.0/24 (public subnet CIDR)
Outbound Rule 100: Allow TCP 5432 to 10.0.1.0/24
```

**Developers report the EC2 server cannot connect to the database. The RDS Security Group correctly allows port 5432 from the EC2 instance's Security Group. Which change will fix the connectivity issue?**

**A.** Add an inbound NACL rule allowing TCP port 5432 from `0.0.0.0/0`, to ensure all sources can reach the database.

**B.** Add an outbound NACL rule allowing TCP ports `1024–65535` to `10.0.1.0/24` (the ephemeral port range).

**C.** Remove the custom NACL from the private subnet and rely on the default NACL instead.

**D.** Add an inbound Security Group rule on the RDS instance allowing TCP 5432 from `0.0.0.0/0`.

**E.** Change the RDS Security Group to allow inbound traffic from the public subnet CIDR `10.0.1.0/24` instead of the EC2 Security Group ID.

---

**Correct Answer: B**

**A — Wrong.** The inbound rule for port 5432 from the public subnet CIDR already exists (Inbound Rule 100). The connection request is reaching RDS — the query executes. The problem is the response being dropped. Opening inbound to `0.0.0.0/0` would be a serious security violation and does not address the root cause. If anything, this makes the configuration less secure.

**B — Correct.** This is the ephemeral port problem. The NACL is stateless. When RDS responds to the EC2 connection on port 5432, the response is sent back to the EC2 application server's random ephemeral source port — a random port in `1024–65535` that the EC2's OS picked when initiating the connection. The current Outbound Rule 100 only allows TCP port 5432 outbound, not the ephemeral range. The response packet hits the outbound NACL with a destination port of (e.g.) 54312, matches no allow rule, hits the implicit `*` DENY, and is dropped. The EC2 app server never receives the response. Adding outbound TCP `1024–65535` to `10.0.1.0/24` allows all response packets to leave the data subnet and reach the EC2 application server.

**C — Wrong.** The default NACL allows all traffic, so removing the custom NACL would restore connectivity. But this removes all subnet-level security controls — the custom NACL was presumably added for a reason. The correct fix preserves security while enabling connectivity. Removing the NACL entirely is not an appropriate production response to a misconfiguration.

**D — Wrong.** The RDS Security Group already correctly allows port 5432 from the EC2 instance's Security Group — the question states this explicitly. The problem is not the Security Group. Opening `0.0.0.0/0` on the Security Group would be a security violation. The problem is the NACL's missing outbound ephemeral port rule.

**E — Wrong.** The RDS Security Group referencing the EC2 Security Group ID is the correct secure pattern — more secure than a CIDR because it allows only instances carrying that specific SG, not all IPs in the subnet CIDR. Changing this to a CIDR would decrease security. The Security Group is working correctly. The problem is the NACL.

---

## Quick-Recall Test

**Q1: Can you add an explicit Deny rule to a Security Group?**
No. Security Groups support Allow rules only. Deny is implicit — anything not explicitly allowed is denied. To explicitly block a specific IP, use a NACL Deny rule.

**Q2: A custom NACL is created and associated with a subnet. No rules have been added. What happens to traffic?**
All traffic to and from the subnet is blocked. A custom NACL starts with only the implicit `*` DENY rule — all inbound and outbound is denied until you add explicit Allow rules.

**Q3: NACL Rule 90: DENY TCP 443. Rule 100: ALLOW TCP 443. What happens to HTTPS traffic?**
It is denied. Rule 90 fires first (lower number evaluated first) and stops evaluation. Rule 100 is never reached.

**Q4: Security Group allows inbound TCP 5432. NACL allows inbound TCP 5432. The connection times out. What is the most likely cause?**
Missing NACL outbound rule for TCP `1024–65535`. The request arrives and is processed, but the response is dropped at the NACL outbound because ephemeral port range is not allowed.

**Q5: What is the default inbound behaviour of a newly created (custom) Security Group?**
Deny all inbound. Nothing is allowed until you add explicit Allow rules. Default outbound is allow all.

**Q6: What is the default inbound behaviour of the Default NACL (auto-created with VPC)?**
Allow all inbound. The Default NACL allows all traffic in both directions — it provides no security filtering.

**Q7: An Auto Scaling group adds a new EC2 instance. The RDS Security Group allows TCP 5432 from `sg-appservers`. Will the new instance have database access?**
Yes — automatically. When the new instance is launched as part of the Auto Scaling group, it inherits `sg-appservers`. The RDS Security Group references `sg-appservers` as the source. No rule change required.

**Q8: What are the three advantages of AWS Systems Manager Session Manager over a Bastion Host?**
(1) No open port 22 on any instance — eliminates SSH-based attack surface. (2) No key pair management — access controlled by IAM permissions. (3) Full audit trail — every session logged to CloudTrail and optionally S3/CloudWatch Logs.
