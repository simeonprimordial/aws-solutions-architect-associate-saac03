# Challenges & Blockers — Week 3 Day 3: Route Tables, Internet Gateways & NAT

---

## Challenge 1: SSH Source Required /32 — Not Just the IP

**What happened:**
When adding the SSH inbound rule to `SG-WebServers`, I entered my IP address from whatismyip.com as `102.89.45.x` without the CIDR prefix. The console showed a validation error: "Invalid CIDR block." I was not sure what format the source field expected.

**What I tried:**
- Tried entering the IP without any suffix — invalid.
- Tried the "My IP" option in the Source dropdown — the console auto-populated the correct format.
- Checked the format it used: `102.89.45.x/32`.

**Resolution:**
Security Group source fields require CIDR notation, not a bare IP address. A single IP address is expressed as a `/32` — 32 fixed bits means only one possible host address. `102.89.45.x/32` means exactly that one IP and nothing else. The "My IP" button in the AWS console auto-fills the correct `/32` format based on the public IP of the browser session. I used "My IP" and then recorded the value for documentation.

**Lesson learned:**
Always add `/32` for a single IP address in any AWS rule or CIDR field. A `/24` would allow all 251 usable addresses in that subnet — a major overreach for an SSH access rule. The `/32` is the minimal-privilege pattern for individual IP-based access.

---

## Challenge 2: Security Group Chaining Source Not Obvious in the Console

**What happened:**
When creating `SG-WebServers` and setting the inbound port 80 source to `SG-LoadBalancer`, I could not find where to enter a Security Group reference. The Source field had a dropdown showing options like "Anywhere", "Custom", "My IP" — I expected a separate "Security Group" option but it was not visible as a labelled button.

**What I tried:**
- Selected "Custom" in the Source dropdown.
- The text field appeared expecting a CIDR. I typed `SG-Load` and a dropdown appeared showing matching Security Groups by name.
- Selected `SG-LoadBalancer` from the dropdown.

**Resolution:**
The Security Group reference option is hidden inside the "Custom" source selection — it is not a separate menu item. After clicking "Custom", the field accepts both CIDR blocks and Security Group names/IDs. Typing the SG name in the field triggers a filtered dropdown showing matching Security Groups. Once I knew the pattern, chaining the database SG to the web server SG was straightforward.

**Lesson learned:**
The console UX is not intuitive for Security Group chaining. In production and in IaC (Terraform, CloudFormation), SG-to-SG references are expressed using the `source_security_group_id` parameter — unambiguous. The console approach of typing an SG name in what looks like a CIDR field is a common stumbling block for new practitioners.

---

## Challenge 3: NACL Rule Order Produced Silent Failures

**What happened:**
During the bonus NACL lab, I added the inbound rules in the wrong order by accident — the catch-all DENY (Rule 100) ended up before the MySQL ALLOW (Rule 200). After associating the NACL with the private subnet, a simulated MySQL connection test failed with a timeout. There was no error message about the NACL — the traffic just stopped.

**What I tried:**
- Checked Security Group on the RDS instance — rules were correct.
- Checked route table — correct.
- Checked the NACL inbound rules — found the DENY at Rule 100 firing before the ALLOW at Rule 200.
- Changed Rule 100 to the MySQL ALLOW and Rule 200 to the catch-all DENY.
- Traffic resumed immediately after saving.

**Resolution:**
NACL rules are evaluated in ascending number order and stop at the first match. A catch-all DENY at Rule 100 blocked everything — MySQL on port 3306 was a subset of "all traffic" so it matched the DENY before reaching the ALLOW. The correct order: specific ALLOW rules at lower numbers (100, 200...) and the catch-all DENY at the highest number (32767 or `*`).

**Lesson learned:**
NACL rule numbering is not cosmetic — it is the evaluation order. Convention: leave gaps between rules (100, 200, 300) to allow insertion of new rules without renumbering. The catch-all DENY always goes at the highest number. Any time traffic stops after a NACL change, the rule order is the first thing to check.

---

## Challenge 4: Two Different NAT Mechanisms — IGW NAT vs NAT Gateway

**What happened:**
Reading the slides about the Internet Gateway performing NAT (translating EC2 private IPs to public IPs) confused me briefly because I had been thinking of NAT as purely the NAT Gateway's job. The IGW also does NAT? I was not sure if they were the same process or different.

**What I tried:**
- Re-read the IGW description in the slides: "The IGW also performs Network Address Translation (NAT) — converting EC2 private IPs to public IPs for outbound traffic."
- Re-read the NAT Gateway description: "allows resources in private subnets to initiate outbound connections by translating their private IP to the NAT Gateway's Elastic IP."
- Compared the two: both perform source IP translation, but for different purposes and different resources.

**Resolution:**
Two different NAT mechanisms for two different scenarios:

**IGW NAT:** An EC2 in a public subnet with a public IP sends traffic outbound. The IGW translates the source IP from the EC2's private IP (`10.0.1.5`) to its public IP (`52.x.x.x`) before sending to the internet. Response comes back to the public IP, IGW translates it back to the private IP. The EC2 instance itself only sees its private IP — the public IP is maintained by the IGW transparently.

**NAT Gateway NAT:** An EC2 in a private subnet (no public IP) sends traffic outbound. The private-rt routes it to the NAT Gateway. The NAT Gateway translates the source IP from the EC2's private IP (`10.0.11.5`) to the NAT Gateway's Elastic IP (`52.y.y.y`) before sending to the internet. Response comes back to the EIP, NAT GW translates it back to the private IP and forwards to the EC2.

**Lesson learned:**
Both perform NAT but at different layers: IGW NAT handles public-subnet instances with public IPs. NAT Gateway handles private-subnet instances with no public IP. The key difference: the IGW NAT is stateless (1:1 mapping between private and public IP). The NAT Gateway performs many-to-one NAT (many private instances share one Elastic IP) using port mapping to track connections.

---

## Challenge 5: Calculating NAT Gateway Cost — Surprised by the Bill

**What happened:**
The before-next-class homework asked me to calculate the cost of 2 NAT Gateways + 500GB/month data. I initially guessed this would be around $50/month — it turned out to be significantly higher and forced me to re-examine how AWS bills for NAT Gateway.

**What I tried:**
- Looked up the pricing from the slides: `$0.045/hr` per NAT GW, `$0.045/GB` processed.
- Calculated: 2 NAT GWs × `$0.045/hr` × 24 hours × 30 days = `$64.80/month` in hourly charges alone.
- Data processing: 500GB × `$0.045/GB` = `$22.50/month`.
- Total: `$64.80 + $22.50 = $87.30/month`.
- If any of the 500GB crosses AZ boundaries: extra `$0.01/GB` = up to `$5/month` more.

**Resolution:**
The total is approximately `$87–92/month` for 2 NAT Gateways handling 500GB/month. Significantly more than my initial guess of $50. The hourly charge is the dominant cost at low-to-medium data volumes — the NAT Gateway is expensive whether it processes traffic or not.

**Lesson learned:**
NAT Gateway is the most commonly overlooked cost item in AWS VPC architectures. Always calculate: (number of NAT GWs × `$0.045` × 720 hours) + (GB processed × `$0.045`). For cost-sensitive workloads, a NAT Instance running on a `t3.nano` (~`$3.80/month`) is dramatically cheaper — at the cost of managing patching, HA, and monitoring yourself. The SAA-C03 exam tests this cost trade-off explicitly.

---

*Add new challenges here as they come up in future days.*
