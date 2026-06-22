# Challenges & Blockers — Week 3 Day 5: VPC Peering & Connectivity

---

## Challenge 1: TGW Route Table Isolation — Took Time to Understand Fully

**What happened:**
The Zenith Bank scenario described Dev VPC and Prod VPC both attaching to the same Transit Gateway, with Dev unable to reach Prod. I initially assumed that attaching to the same TGW meant all attached VPCs could communicate — like all subnets in the same VPC being able to route to each other via the local route. The concept of TGW having its own route tables that independently control what can reach what took a second reading to fully understand.

**What I tried:**
- Drew the TGW hub on paper with all five VPCs attached as spokes.
- Wrote out `prod-rt` and `dev-rt` as two separate TGW route tables.
- Traced a packet from Dev VPC to Prod VPC: Dev sends packet → arrives at TGW → TGW checks `dev-rt` route table → `dev-rt` has no route to Prod VPC's CIDR → packet dropped at TGW → Prod VPC never receives it.
- Traced a packet from Dev VPC to Shared Services VPC: Dev sends packet → arrives at TGW → TGW checks `dev-rt` → `dev-rt` has a route to Shared Services CIDR → packet forwarded to Shared Services → connection succeeds.

**Resolution:**
TGW route tables are like separate routing policies applied per attachment. A VPC attachment is associated with one TGW route table — that table determines which other attachments it can reach. A different attachment can be in a completely different TGW route table with no route to the first VPC. This is fundamentally different from VPC Peering, where the ability to communicate is determined by whether a peering connection exists at all. TGW adds a policy layer on top of connectivity.

**Lesson learned:**
TGW route table isolation is the key reason to choose TGW over VPC Peering for multi-account environments where security boundaries between environments (Dev vs Prod) must be enforced. VPC Peering cannot enforce this kind of policy isolation — you would need to accept that any peered VPC can always reach any other peered VPC (within the direct peering relationships). TGW gives you the routing topology AND the policy control.

---

## Challenge 2: Direct Connect Not Encrypted — Counterintuitive

**What happened:**
When I read that Direct Connect "is NOT encrypted by default," I did not initially believe it. I assumed that a dedicated private fibre connection — one that never touches the public internet — would inherently be more secure and therefore encrypted. The concept that "private" and "encrypted" are two different properties took a moment to separate.

**What I tried:**
- Thought through the distinction: private means no other organisation's traffic shares the link. Encrypted means the data itself is ciphertextually unreadable without a key.
- A dedicated circuit can carry plaintext traffic that no external organisation can intercept — but an insider at the Direct Connect facility or an AWS employee with access to the physical infrastructure could theoretically read unencrypted traffic.
- For regulated environments (CBN, PCI-DSS), both properties are required: private (so traffic does not traverse the shared internet) AND encrypted (so data is ciphertextually protected at all points).

**Resolution:**
Direct Connect provides network-level privacy — the traffic is not routed through shared internet infrastructure. But it does not provide application-level or transport-level encryption. The data on the wire is plaintext unless you explicitly add encryption. Adding a Site-to-Site VPN over the DX connection provides IPsec encryption on top of the private path. This combination is the correct answer for any scenario that requires both properties simultaneously.

**Lesson learned:**
"Private" and "encrypted" are distinct security properties that both need explicit implementation in AWS. The exam tests this distinction in every DX scenario. Any question mentioning a regulatory requirement for both private connectivity AND data encryption in transit → DX + VPN. DX alone fails the encryption requirement even though it feels more secure than VPN over the internet.

---

## Challenge 3: Draw.io AWS Icon Set — Initial Setup Time

**What happened:**
The lab guide mentioned downloading the AWS icon set before starting the diagram. I spent about 10 minutes on setup: finding the correct icon library in draw.io, enabling it, and understanding which icon to use for each service. EC2, RDS, and ALB were straightforward. The NAT Gateway and Internet Gateway icons were harder to locate immediately.

**What I tried:**
- Used draw.io's shape search: typed "NAT" in the search bar → found the NAT Gateway icon in the AWS VPC category.
- Typed "Internet Gateway" → found the IGW icon.
- Used the AWS architecture page `https://aws.amazon.com/architecture/icons/` as a reference for the canonical icon names.

**Resolution:**
The draw.io built-in AWS library organises icons by category (VPC, Compute, Database, etc.). The search function finds icons by name — knowing the exact service name speeds up the search significantly. The AWS architecture icons page shows all icon names and their categories — bookmarked this for future diagram work.

**Lesson learned:**
Investing 10 minutes in setting up the diagramming tool correctly at the start saves time on every subsequent diagram. Having the correct icon for each service matters — using a generic server icon for EC2 or a box for RDS makes diagrams look amateurish compared to official AWS icons. The icon set is free and using it is a professional standard.

---

## Challenge 4: Showing Route Table Entries on the Diagram

**What happened:**
The lab guide said to "label the arrow with the route table entry." I was not sure whether to put the route table entry on the arrow itself, inside the subnet box, or as a separate annotation. The diagram started looking cluttered when I put text on every arrow.

**What I tried:**
- Put full route table entries (`Destination: 0.0.0.0/0, Target: igw-xxxxxxxx`) on the arrows — too verbose, cluttered the diagram.
- Abbreviated to just `0.0.0.0/0 → IGW` — cleaner but still added noise when applied to every arrow.
- Moved route table annotations inside each subnet box as small secondary labels — cleaner, allows arrows to remain annotation-free.

**Resolution:**
The best approach for this diagram: primary arrows show the traffic flow direction without annotation. Each subnet box contains a small route table summary: `RT: 0.0.0.0/0 → IGW` (public), `RT: 0.0.0.0/0 → NAT` (private-app), `RT: local only` (private-data). Security Group chain arrows carry the port and SG reference as a short label: `TCP 80 | SG ref`. This keeps the diagram readable while still communicating the routing and security model.

**Lesson learned:**
Diagram readability requires deliberate choices about information density. Every label should earn its place — it should add information that a reader cannot infer from the topology alone. Route table summaries in subnet boxes are more readable than annotations on every arrow. The goal is a diagram that someone can read in 60 seconds and understand the entire architecture.

---

## Challenge 5: VPN Two Tunnels — Initially Did Not Understand the Risk

**What happened:**
The slides mentioned "AWS provisions 2 tunnels automatically — both must be monitored." I initially thought: if there are 2 tunnels, that means there is automatic redundancy and monitoring them is just good practice. I did not understand why monitoring was described as a specific risk rather than standard operational hygiene.

**What I tried:**
- Read the trap description again: "If one fails with no failover configuration → outage."
- Researched: AWS creates two tunnels but they are active-passive by default on the AWS side, and the on-premises device must be configured to use both. Without configuration, the on-premises device might only route traffic through one tunnel.
- If the active tunnel fails: traffic stops. The second tunnel is physically there but is not carrying traffic unless the on-premises device detects the failure and switches.
- If both tunnels fail simultaneously: complete outage.

**Resolution:**
The risk is that "two tunnels exist" does not mean "automatic failover works." The on-premises Customer Gateway device must be configured to detect tunnel health and fail over to the second tunnel. AWS does not provide automatic failover between VPN tunnels. The recommended approach: monitor both tunnels with CloudWatch alarms, configure the on-premises device for active-passive failover, and use Direct Connect as a higher-reliability backup for critical connectivity.

**Lesson learned:**
In AWS, redundancy features often require explicit configuration to actually provide redundancy — they are not turnkey. Two VPN tunnels exist but manual failover configuration is required. Two DX connections provide redundancy only if both are active and configured with BGP failover. High availability on AWS is designed, not assumed.

---

*Add new challenges here as they come up in future days.*
