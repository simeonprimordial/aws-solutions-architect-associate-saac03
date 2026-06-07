# Exam Prep — Week 3 Day 5: VPC Peering & Connectivity

## SAA-C03 Context

Connectivity appears across Domain 1 (Design Resilient Architectures, ~30%) and Domain 3 (Design High-Performing Architectures, ~24%). Domain 1 tests multi-VPC and hybrid connectivity for resilience — specifically Transit Gateway HA patterns, Direct Connect failover, and VPN redundancy. Domain 3 tests which connectivity option provides the best performance for a given data transfer pattern — specifically Direct Connect vs VPN for high-bandwidth, low-latency workloads. Domain 2 tests VPC Endpoints for security — keeping traffic off the internet by using private paths to AWS services. The SAA-C03 typically has 3–5 connectivity questions per attempt. The decision framework — knowing which tool to choose for which scenario — answers most of them directly.

---

## Exam Traps — Deep Explanations

### Trap 1: VPC Peering is non-transitive — the most-tested peering concept

The exam will always present a three-VPC scenario and ask how to enable A-to-C connectivity. The distractor answer is always something about updating the peering configuration or making it transitive. This is impossible. VPC Peering is architecturally non-transitive — traffic entering a peered VPC will not be forwarded to another peered VPC. The correct answer is either: (1) create a direct A-C peering connection and add route entries in both A and C, or (2) use Transit Gateway, which is designed for multi-VPC routing. Knowing which answer the question expects depends on the number of VPCs: 2–3 → peering. 5+ → TGW.

### Trap 2: Peering alone routes nothing — update both route tables

Creating a peering connection is a necessary but not sufficient step for connectivity. After creating `pcx-xxxxx` between VPC A and VPC B: VPC A's route table needs `172.16.0.0/16 → pcx-xxxxx`, and VPC B's route table needs `10.0.0.0/16 → pcx-xxxxx`. The exam will describe one VPC being able to reach the other but not vice versa — the answer is always "add the missing route in the reverse direction." One-way routes = one-way traffic. Both route tables, both directions.

### Trap 3: Direct Connect is NOT encrypted by default

This is stated plainly but is counterintuitive enough that it catches candidates off guard. A private dedicated fibre circuit is private — it does not share infrastructure with other organisations. But the data on the wire is not cryptographically encrypted unless you explicitly add encryption. For any scenario mentioning CBN compliance, PCI-DSS, financial data regulations, or "private AND encrypted" — the answer is DX + VPN. DX alone satisfies the private requirement. VPN alone satisfies the encrypted requirement. Only their combination satisfies both simultaneously. An exam option offering "Direct Connect for privacy and compliance" without VPN for encryption is a trap.

### Trap 4: VPN has two tunnels — both must be monitored, no automatic failover

AWS provisions two IPsec tunnels per VPN connection automatically. This looks like automatic redundancy. But there is no automatic failover between tunnels without explicit configuration on the on-premises Customer Gateway device. The exam may describe a scenario where a VPN connection was working, one tunnel failed, and the connection went down — ask what should have been configured. The answer is: monitor both tunnels with CloudWatch, configure the on-premises device for active-passive failover between the two tunnels. The two tunnels do not self-heal.

### Trap 5: Gateway Endpoints for S3 and DynamoDB are free — always use them

For any scenario where a private subnet needs to reach S3 or DynamoDB: the answer is a Gateway Endpoint. Not a NAT Gateway (which costs `$0.045/hr + $0.045/GB` and routes traffic through the internet). Not an Interface Endpoint (which costs `$0.01/hr` per AZ). A Gateway Endpoint is free — `$0/hr`, `$0/GB`. It adds a route table entry routing S3 and DynamoDB traffic to the endpoint over the AWS backbone. The exam will offer NAT Gateway as a distractor for this scenario. NAT Gateway is always wrong for S3/DynamoDB private access when a Gateway Endpoint exists.

---

## Architecture Decision Table — Full Connectivity Framework

| Scenario | Correct Tool | Wrong Answer / Trap |
|---|---|---|
| Connect 2 VPCs, same account | VPC Peering | TGW — overkill and higher cost |
| Connect 5 VPCs, Dev cannot reach Prod | Transit Gateway + route table isolation | VPC Peering — cannot enforce isolation cleanly |
| On-premises branch office to AWS, 50GB/month | Site-to-Site VPN | Direct Connect — weeks to set up, over-engineered |
| On-premises data centre, 500GB/day, <20ms latency | Direct Connect | VPN — 1.25 Gbps cap, 80–200ms internet latency |
| On-premises to AWS: both private path AND encryption required | DX + VPN | DX alone (not encrypted), VPN alone (internet, variable latency) |
| Private subnet EC2 needs to read/write S3 | S3 Gateway Endpoint (FREE) | NAT Gateway (costly, routes through internet) |
| Private subnet EC2 needs to call SSM | Interface Endpoint for SSM | NAT Gateway (unnecessary internet exposure) |
| 10 VPCs, all need to reach a shared services VPC | Transit Gateway | 10 VPC Peering connections — manageable but does not scale |
| Direct Connect fails — maintain connectivity | VPN as automatic failover backup | No backup = complete outage |
| Need to add a new VPC to 10-VPC architecture | TGW: one new attachment | Peering: up to 10 new connections |
| Cross-account VPC connectivity | VPC Peering (2–3 VPCs) or TGW (many) | Neither required for same-account |
| SaaS vendor service needs to be accessible privately | Interface Endpoint (PrivateLink) | Public endpoint over internet |

---

## Practice Question

**A Lagos fintech has five VPCs (Production, Analytics, Staging, Shared Services, and Development) and a Lagos data centre. ALL VPCs must be able to reach the Shared Services VPC. Production and Analytics VPCs must communicate with each other. The Development VPC must NOT be able to reach Production. The data centre sends 300GB/day to the Analytics VPC. Which configurations meet ALL requirements? (Select TWO)**

**A.** Full-mesh VPC Peering between all five VPCs, plus Direct Connect from the Lagos data centre.

**B.** Transit Gateway connecting all five VPCs with separate TGW route tables (Prod/Analytics/Staging/Shared in one table, Dev/Shared in another), plus Direct Connect from the Lagos data centre.

**C.** Transit Gateway connecting all five VPCs with a single shared route table, plus Site-to-Site VPN from the Lagos data centre.

**D.** VPC Peering from each VPC individually to Shared Services VPC, plus Direct Connect from the Lagos data centre.

---

**Correct Answer: B only (the question asks for the combination that meets ALL requirements)**

**A — Wrong.** Full-mesh peering for five VPCs requires 10 peering connections. More importantly, VPC Peering cannot enforce the Dev-cannot-reach-Prod requirement — with full-mesh peering, every VPC can reach every other VPC (within the direct peering relationships). There is no mechanism in VPC Peering to prevent Dev from reaching Prod if they are both peered to a shared VPC. Additionally, 300GB/day at VPN-equivalent data transfer rates would be expensive and slow — Direct Connect is the correct choice for that volume, which A does include, but the peering topology fails the isolation requirement.

**B — Correct.** Transit Gateway with separate TGW route tables enforces the Dev-cannot-reach-Prod requirement at the network level: `prod-rt` includes Prod, Analytics, Staging, and Shared Services — all can communicate. `dev-rt` includes Dev and Shared Services only — Dev can reach Shared Services (needed for DNS/auth) but cannot reach Prod even through the TGW. One TGW attachment per VPC (5 attachments) is simpler than 10 peering connections. Direct Connect is the correct choice for 300GB/day — consistent low latency, dramatically lower per-GB data transfer cost, no 1.25 Gbps bandwidth ceiling.

**C — Wrong.** Transit Gateway is correctly chosen for the multi-VPC topology. But a single shared TGW route table means all five VPCs — including Dev — have routes to all other VPCs including Prod. The isolation requirement is not met. Additionally, Site-to-Site VPN at 300GB/day is expensive (`300 × $0.045 = $13.50/day`, `$405/month` in data charges alone, plus `$36/month` connection fee) and the 1.25 Gbps cap may limit throughput for burst transfers. Direct Connect at this volume is the correct choice — lower per-GB rates and consistent bandwidth.

**D — Wrong.** Five individual peering connections to Shared Services does not provide Prod-Analytics communication (they are not peered to each other, only to Shared Services). This is the non-transitive problem: Prod peers Shared Services, Analytics peers Shared Services, but Prod cannot reach Analytics through Shared Services. Also provides no mechanism to prevent Dev from reaching Prod if they are both peered to the same VPCs. The topology fails both the Prod-Analytics communication requirement and the Dev-isolation requirement.

---

## Quick-Recall Test — Decision Framework Without Notes

**Q1: Two VPCs in different AWS accounts must communicate privately. Which service?**
VPC Peering — direct, private, works cross-account. Update both VPCs' route tables.

**Q2: 10 VPCs need full connectivity, but Dev must not reach Prod. Which service?**
Transit Gateway — hub-and-spoke with separate TGW route tables for isolation. 10 attachments vs 45 peering connections.

**Q3: On-premises office needs to connect to AWS. Setup must happen today. Which service?**
Site-to-Site VPN — provisions in minutes, no physical provisioning, IPsec encrypted by default.

**Q4: 500GB/day from a Lagos data centre to AWS Analytics VPC. Latency requirements: consistent sub-20ms. Which service?**
Direct Connect — dedicated private fibre, consistent low latency, dramatically cheaper per-GB at high volume.

**Q5: Direct Connect is used but the compliance team requires data to be encrypted in transit. What is the solution?**
Add a Site-to-Site VPN over the Direct Connect connection. DX provides the private path; VPN adds IPsec encryption. DX + VPN = private AND encrypted.

**Q6: A private subnet EC2 needs to write logs to S3. There is no NAT Gateway and no internet route. How?**
S3 Gateway Endpoint — free, adds a route table entry routing S3 traffic to the endpoint over the AWS backbone. No internet path required.

**Q7: A private subnet EC2 needs AWS Systems Manager Session Manager access. No internet. Which service?**
VPC Interface Endpoints for `ssm`, `ssmmessages`, and `ec2messages` — powered by PrivateLink, creates ENI with private IP in the subnet.

**Q8: VPC A peers VPC B. VPC B peers VPC C. Can VPC A reach VPC C?**
No. VPC Peering is non-transitive. A has no route to C. A direct A-C peering connection with route entries in both A and C is required, or use Transit Gateway.
