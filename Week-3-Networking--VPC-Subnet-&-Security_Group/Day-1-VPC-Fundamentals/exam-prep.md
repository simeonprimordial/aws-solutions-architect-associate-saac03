# Exam Prep — Week 3 Day 1: VPC Fundamentals

## SAA-C03 Context

VPC Fundamentals appears across multiple domains of the SAA-C03 exam. Domain 1 (Design Resilient Architectures, ~30%) tests multi-AZ VPC design, subnet placement for high availability, and the correct use of NAT Gateways for private subnet outbound access. Domain 2 (Design Secure Architectures, ~26%) tests Security Group vs NACL behaviour, subnet isolation for sensitive workloads, and the security implications of routing decisions. Domain 3 (Design High-Performing Architectures) tests VPC Endpoints for reducing latency to AWS services. VPC knowledge is the foundation every other AWS architect topic builds on — understanding it at this depth makes every subsequent domain easier.

---

## Exam Traps — Deep Explanations

### Trap 1: A subnet is public because of its route table — not its name or auto-assign setting

The most important VPC concept on the exam. AWS will describe a subnet with a public-sounding name and auto-assign public IP enabled, then ask why instances in it cannot reach the internet. The answer is always: no Internet Gateway route in the route table. You can name a subnet `Public-Production-AZ-A`, enable auto-assign public IPv4, put an EC2 instance in it with a public IP — and if the route table has only the `local` route, the instance is unreachable from the internet and cannot reach it either. Publicness = `0.0.0.0/0 → IGW` in the route table. Nothing else.

### Trap 2: Security Groups are STATEFUL — NACLs are STATELESS

These behave completely differently and the exam tests this distinction in every domain. A Security Group tracks connection state — add an inbound allow rule and the return traffic flows automatically. You do not need a corresponding outbound rule. NACLs track nothing — they evaluate every packet independently. Allow HTTP inbound on port 80 in a NACL and the response traffic (going back to the client on an ephemeral port `1024–65535`) is blocked unless you add an explicit outbound allow rule covering that port range. This trips candidates consistently. The symptom: a NACL-protected subnet where requests arrive but responses never return. The fix: outbound rule allowing TCP `1024–65535` to `0.0.0.0/0`.

Additionally: Security Groups only have allow rules. You cannot write a deny rule in a Security Group. NACLs have both allow and deny rules — you can use a NACL to block a specific IP address at the subnet level. This is the correct answer for questions asking how to block a known malicious IP from reaching any resource in a subnet.

### Trap 3: NAT Gateway is outbound only — never inbound

Read the question carefully. "Instances in private subnets need to download patches" → NAT Gateway is correct. "Internet users need to access the application in private subnets" → NAT Gateway is wrong. The correct answer for inbound: public subnet + IGW + Application Load Balancer in the public subnet forwarding traffic to instances in the private subnet. NAT Gateway blocks all unsolicited inbound. Any distractor option that uses a NAT Gateway to solve an inbound access problem is always wrong.

### Trap 4: VPC Peering is non-transitive

A→B peers and B→C peers does not mean A→C works. VPC A's route table has a route to B's CIDR via the A↔B peering connection. VPC A has no route to C. AWS does not propagate routes through peering chains. To give A access to C: create a direct A↔C peering connection AND add explicit route entries in both A and C's route tables for each other's CIDRs. Three fully-connected VPCs need three peering connections. Ten fully-connected VPCs need 45 peering connections. The exam answer for many-VPC connectivity is always AWS Transit Gateway.

### Trap 5: VPC Endpoints connect to AWS services, not to the internet

Gateway VPC Endpoints cover S3 and DynamoDB only — and they are free. Interface VPC Endpoints (powered by AWS PrivateLink) cover most other AWS services and cost per hour. Neither type provides general outbound internet access. A question asking how to let private subnet instances download OS updates from `apt` or `yum` repositories → NAT Gateway. A question asking how to let private subnet instances access S3 without going over the internet → VPC Gateway Endpoint. These are completely different requirements and the services are not interchangeable.

---

## Architecture Decision Table

| Scenario | Correct Solution |
|---|---|
| Internet users must reach EC2 app servers | Public subnet + IGW + ALB in public subnet forwarding to private EC2 |
| Private EC2 instances must download patches | NAT Gateway in public subnet, private route table `0.0.0.0/0 → NAT GW` |
| Private EC2 must access S3 without internet | VPC Gateway Endpoint for S3 (free) |
| Private EC2 must access SQS/SNS without internet | VPC Interface Endpoint (PrivateLink, per hour cost) |
| Block a malicious IP from reaching all resources in a subnet | NACL Deny rule for that IP on the subnet |
| Block a malicious IP from reaching one specific EC2 | Security Group inbound rule removal (SGs have no deny — remove the allow) |
| Connect two VPCs in the same region for private communication | VPC Peering |
| Connect 10+ VPCs with full mesh routing | AWS Transit Gateway |
| SSH access to private EC2 instances | Bastion Host in public subnet OR AWS Systems Manager Session Manager (no bastion needed) |
| Database in private subnet — prevent ALL internet traffic | Isolated subnet route table with NO `0.0.0.0/0` entry — not even a NAT GW route |
| Multi-AZ NAT Gateway for high availability | One NAT Gateway per AZ, each AZ's private subnets route to the NAT GW in the same AZ |

---

## Practice Question

**A solutions architect at a Lagos fintech is designing AWS infrastructure for a payment processing application. The EC2 instances running the payment API must be able to download software patches from the internet, but must NOT be directly reachable from the internet. Users must access the application through a load balancer only. Which combination of VPC components correctly meets these requirements? (Select TWO)**

**A.** Place the EC2 instances in a public subnet with a Security Group that blocks all inbound traffic from `0.0.0.0/0`.

**B.** Place the EC2 instances in a private subnet and configure a NAT Gateway in a public subnet. Add a route in the private subnet's route table pointing `0.0.0.0/0` to the NAT Gateway.

**C.** Place the Application Load Balancer in a public subnet with an Internet Gateway route. Configure the ALB to forward traffic to the EC2 instances in the private subnet.

**D.** Place the EC2 instances in an isolated subnet with no route table entries and use a VPC Endpoint to allow them to download patches.

**E.** Enable a VPN Gateway on the VPC and route all outbound traffic from EC2 instances through the corporate VPN to access the internet.

---

**Correct Answers: B and C**

**A — Wrong.** Public subnet instances have a route to the Internet Gateway. Even if the Security Group blocks all inbound, the instances are internet-reachable at the network layer — a Security Group misconfiguration by any team member would immediately expose them. The requirement is architectural isolation, not just Security Group rules. Architectural controls beat configuration controls. Private subnet + no IGW route is the correct answer.

**B — Correct.** This is the standard pattern for private instances that need outbound internet access. Private subnet = no IGW route = no direct inbound from internet. NAT Gateway in public subnet = instances can initiate outbound connections to download patches. The NAT Gateway blocks all unsolicited inbound by design. Users cannot reach the EC2 directly through the NAT Gateway.

**C — Correct.** ALB in a public subnet with an IGW route receives HTTPS traffic from users. It forwards to the EC2 target group in the private subnet. Users reach the application through the ALB — they never have a direct connection to the EC2 instance. This is the standard 3-tier architecture pattern. Both B and C together complete the architecture: B provides patch download capability, C provides the user-facing access path.

**D — Wrong.** VPC Endpoints connect to AWS services (S3, DynamoDB, SQS, etc.) — not to the public internet. Software patches from package managers (`apt`, `yum`, `dnf`) require general internet access to reach package repository servers. A VPC Endpoint cannot provide this. A NAT Gateway is required for general outbound internet access from a private subnet.

**E — Wrong.** A VPN Gateway connects your VPC to an on-premises corporate network. It does not provide general internet access for patch downloads. Routing patch traffic through a corporate VPN would require the corporate network to have internet egress and be willing to route that traffic — it is architecturally incorrect for this requirement, adds unnecessary latency and complexity, and introduces a corporate network dependency.

---

## Quick-Recall Test

**Q1: What is the ONLY thing that determines whether a subnet is public?**
The route table. Specifically, whether the route table associated with the subnet has a `0.0.0.0/0` route pointing to an Internet Gateway.

**Q2: How many usable IP addresses are in a `/26` subnet?**
64 total − 5 reserved = 59 usable.

**Q3: A NACL allows inbound TCP port 80 but HTTP responses are not reaching clients. What is wrong?**
Missing outbound rule allowing TCP ports `1024–65535` for ephemeral return traffic. NACLs are stateless — both directions must be explicitly allowed.

**Q4: A NAT Gateway is in `af-south-1a`. The private subnet in `af-south-1b` routes its internet traffic through it. The NAT Gateway becomes unavailable. What happens?**
Private instances in `af-south-1b` lose internet access. For multi-AZ resilience, create one NAT Gateway per AZ and have each AZ's private subnets point to the NAT Gateway in the same AZ.

**Q5: VPC A peers VPC B. VPC B peers VPC C. Can instances in VPC A communicate with instances in VPC C?**
No. VPC Peering is non-transitive. A has no route to C. A direct A↔C peering connection with explicit route entries in both A and C is required.

**Q6: What is the difference between a Gateway VPC Endpoint and an Interface VPC Endpoint?**
Gateway Endpoints: S3 and DynamoDB only, free, uses route table entries. Interface Endpoints: most other AWS services, powered by PrivateLink, billed per hour and per GB, creates an ENI in your subnet.

**Q7: A Security Group allows inbound TCP 443. Do you need a separate outbound rule for the HTTPS response?**
No. Security Groups are stateful. The return traffic for an allowed inbound connection is automatically permitted without an explicit outbound rule.

**Q8: What is the minimum number of peering connections needed to fully connect 4 VPCs (every VPC can reach every other VPC)?**
6 peering connections. Formula: `n(n-1)/2` where n = number of VPCs. `4 × 3 / 2 = 6`. At this scale, AWS Transit Gateway is the better architectural choice.
