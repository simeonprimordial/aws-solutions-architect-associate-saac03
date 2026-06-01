# Challenges & Blockers — Week 3 Day 1: VPC Fundamentals

---

## Challenge 1: Subnet Named "Public" Was Not Actually Public

**What happened:**
After creating `Public-Subnet-AZ-A` and enabling auto-assign public IPv4, I assumed the subnet was now public and ready for internet-facing resources. I navigated away before attaching an Internet Gateway or configuring a route table. When I came back to review the routing, I realised the subnet had no IGW route — it was behaving as a private subnet despite its name and auto-assign setting.

**What I tried:**
- Re-read the subnet settings panel looking for a "make public" toggle — there isn't one.
- Checked whether the auto-assign setting applied the IGW route — it does not.
- Read the route table associated with the subnet and found only the `local` route.

**Resolution:**
The fix was understanding that publicness is exclusively determined by the route table. The subnet name is a label. Auto-assign public IP controls whether instances get an address, not whether that address can communicate with the internet. Making the subnet actually public requires: (1) create an IGW, (2) attach it to the VPC, (3) add a route `0.0.0.0/0 → igw-xxxxxxxx` in a custom route table, (4) associate that route table with the subnet. All four steps are required.

**Lesson learned:**
The route table is the single source of truth for subnet access. Before assuming any subnet is public, check its route table. This is also the most-tested conceptual distinction on the SAA-C03 — I will not get this wrong on the exam.

---

## Challenge 2: AWS Reserved IPs — Miscount on Usable Addresses

**What happened:**
When planning the subnet CIDRs, I initially calculated a `/24` as giving 256 usable IP addresses. The console showed 251 available IPs after creating the subnet, which didn't match. I went back to check whether I had made a configuration error.

**What I tried:**
- Recalculated the host count: `2^8 = 256`. That is the total, not the usable count.
- Searched the VPC documentation for the discrepancy.
- Found the AWS reserved addresses section: `.0`, `.1`, `.2`, `.3`, `.255` — five addresses reserved per subnet.

**Resolution:**
`256 - 5 = 251` usable IPs in a `/24`. This is not a bug or misconfiguration. AWS reserves five addresses in every subnet for the network address, VPC router, DNS resolver, future use, and broadcast. I updated my CIDR planning notes and added this to the exam prep section.

**Lesson learned:**
The SAA-C03 will test this. A question asks: "a company needs 29 usable IPs in a subnet — which CIDR block should they use?" The answer is `/27` (32 total, 27 usable) not `/28` (16 total, 11 usable). Always subtract 5 before comparing against the requirement. I added this calculation pattern to my flashcards.

---

## Challenge 3: NACL Ephemeral Ports — Why Response Traffic Was Being Dropped

**What happened:**
During the conceptual walkthrough of NACL rules, the slides showed that a NACL allowing inbound HTTP on port 80 would still drop response traffic unless an outbound rule explicitly allowed ephemeral ports `1024–65535`. This didn't make sense to me initially. I understood the stateless concept in theory but couldn't see why the port range was `1024–65535` specifically rather than port 80.

**What I tried:**
- Re-read the NACL stateless explanation: both directions must be explicitly allowed.
- Looked up how TCP connections work: the client picks a random source port in the ephemeral range (`1024–65535`) for its connection. The server responds TO that port, not to port 80.
- Traced through a worked example: user on port `52431` (ephemeral) → server on port `80` (HTTP). Server responds from port `80` back to `52431`. The NACL outbound rule must allow TCP `1024–65535` to permit the server's response to reach the client.

**Resolution:**
Port 80 is only the destination port on the inbound request. The response goes from the server back to the client's ephemeral port. Since NACLs are stateless, they don't remember the inbound connection and cannot infer that the outbound traffic on port `52431` is a response. The outbound rule must cover the entire ephemeral range because the actual client port is unpredictable.

**Lesson learned:**
NACLs require two rules for every traffic flow: one inbound (the request) and one outbound covering the ephemeral port range (the response). Security Groups handle this automatically because they are stateful. This is the core operational difference — not just an exam concept.

---

## Challenge 4: Default VPC Comparison — Surprised by Default Route Table

**What happened:**
During the bonus challenge, I expected the default VPC to be more restricted than my custom VPC. Instead, every default subnet had a route to the Internet Gateway, auto-assign public IP enabled, and the default NACL allowing all traffic in both directions. Any EC2 instance launched into the default VPC without explicit Security Group rules would be directly reachable from the internet.

**What I tried:**
- Checked all default subnets — all showed `Auto-assign public IPv4: Yes`.
- Checked the main route table — `0.0.0.0/0 → igw-xxxxxxxx` was already there.
- Checked the default NACL — `ALLOW ALL` inbound and outbound.
- Checked the default Security Group — `DENY ALL` inbound (the only protection in the default configuration).

**Resolution:**
This is by design. AWS makes the default VPC work immediately for experimentation — you can launch an EC2 and SSH into it within minutes. The security model relies entirely on the default Security Group blocking inbound. But if a developer modifies the default Security Group or launches into a custom SG that allows inbound, the resource is directly internet-exposed with no subnet-level backstop. The custom VPC I built today has defence-in-depth: private subnets with no IGW route, NACLs, Security Groups, and route table isolation. The default VPC has one layer. That is why the default VPC must never be used in production.

**Lesson learned:**
The default VPC is a learning environment, not a production environment. Any CBN-regulated Nigerian fintech launching resources into the default VPC would fail a network security audit immediately — RDS in a public subnet with a direct internet route is not acceptable for financial data.

---

## Challenge 5: VPC Peering Non-Transitivity — Conceptual Block

**What happened:**
The non-transitive peering concept tripped me up initially. I kept thinking: if VPC A is connected to VPC B, and VPC B is connected to VPC C, why can't A route through B to reach C? The connection exists. B knows about both networks.

**What I tried:**
- Drew the three-VPC scenario on paper: A—B—C.
- Looked at what route entries exist in each VPC's route table after peering A↔B and B↔C.
- Found the answer: VPC B's route table knows how to reach A and knows how to reach C. But VPC A's route table only has a peering route to B. A has no route entry for C's CIDR. AWS does not automatically propagate routes through peers.

**Resolution:**
Peering is a direct connection between two VPCs. It does not create a transit path. The routes only exist in the directly peered VPCs — there is no automatic route propagation through a peering chain. To give A access to C, you need a direct A↔C peering connection with explicit route entries in both A and C's route tables. At scale (10+ VPCs), this becomes a full-mesh peering problem that AWS Transit Gateway solves by acting as a central hub.

**Lesson learned:**
When I see a three-VPC scenario on the SAA-C03, the answer is never "add a peering connection between A and B." The answer is a direct A↔C peering OR Transit Gateway. I wrote this rule on a card: non-transitive means every pair needs its own direct connection.

---

*Add new challenges here as they come up in future days.*
