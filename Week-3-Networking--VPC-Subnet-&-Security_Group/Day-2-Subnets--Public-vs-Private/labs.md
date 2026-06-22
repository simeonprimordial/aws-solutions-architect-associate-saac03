# Labs — Week 3 Day 2: Subnets — Public vs Private

---

## Lab 1: Create and Attach an Internet Gateway

### Steps

1. Navigate to **VPC → Internet gateways → Create internet gateway**.
2. Name tag: `OluTech-IGW`.
3. Click **Create internet gateway**.
4. After creation, click **Actions → Attach to VPC**.
5. Select `OluTech-Production-VPC` from the dropdown.
6. Click **Attach internet gateway**.
7. Confirm status shows `Attached` with the correct VPC ID.

### What I Observed

The Internet Gateway is created detached — it exists as an AWS resource but has no VPC association until you explicitly attach it. The attach step is separate from the create step, which I nearly missed. After attaching, the status changes from `Detached` to `Attached` and the VPC ID populates in the console. One IGW per VPC is enforced — attempting to attach a second IGW to the same VPC returns an error immediately.

I also noticed that creating the IGW alone does nothing for routing. The VPC has an IGW attached, but no subnet has internet access yet — because no route table points to it. The attachment is infrastructure. The routing is the next step.

### What I Learned

- IGW creation and IGW attachment are two distinct steps. It is possible to have an IGW that is created but not attached — a common misconfiguration.
- One IGW per VPC is a hard limit. You cannot attach multiple IGWs.
- Attaching the IGW does not automatically update any route table. The `0.0.0.0/0 → IGW` route must be added manually to the relevant route table.

---

## Lab 2: Create the Public Route Table

### Steps

1. Navigate to **VPC → Route tables → Create route table**.
2. Name: `Public-Route-Table`.
3. VPC: `OluTech-Production-VPC`.
4. Click **Create route table**.
5. Select `Public-Route-Table` → **Routes tab → Edit routes**.
6. The `10.0.0.0/16 → local` route is already present. Do NOT delete it — this is the route that keeps VPC-internal traffic inside the VPC.
7. Click **Add route**.
   - Destination: `0.0.0.0/0`
   - Target: Internet Gateway → select `OluTech-IGW`
8. Click **Save changes**.
9. Confirm the Routes tab shows two entries: `10.0.0.0/16 → local` and `0.0.0.0/0 → igw-xxxxxxxx`.

### What I Observed

When editing routes, the `local` route is pre-populated and greyed out — AWS does not allow you to modify or delete it. This is a hard constraint: VPC-internal routing is always preserved. The `local` route ensures that traffic between subnets within the VPC never leaves the VPC boundary.

Adding the `0.0.0.0/0 → IGW` route was a single line in the console. But the impact is significant — this one route entry is what makes any subnet associated with this route table a public subnet. I took a screenshot immediately after saving to capture this as the before/after evidence.

### What I Learned

- The `local` route (`10.0.0.0/16 → local`) is immutable. AWS always preserves it. VPC-internal traffic can never accidentally be routed externally.
- The entire difference between a public subnet and a private subnet is this one route: `0.0.0.0/0 → igw-xxxxxxxx`. Without this route, the IGW is attached to the VPC but functionally irrelevant to that subnet.
- Creating a named, separate route table for public subnets (instead of using the main route table) is production best practice. The main route table should stay private — any new subnet not explicitly associated with a custom route table falls back to the main route table, so keeping it without an IGW route means new subnets are private by default.

---

## Lab 3: Associate Public Subnets with the Public Route Table

### Steps

1. Select `Public-Route-Table` → **Subnet associations tab → Edit subnet associations**.
2. Check both:
   - `Public-Subnet-AZ-A`
   - `Public-Subnet-AZ-B`
3. Click **Save associations**.
4. Confirm the Explicit subnet associations panel shows both subnets linked to `Public-Route-Table`.

### What I Observed

Before this step, both public subnets were associated with the main route table (local only). After saving the associations, they moved to `Public-Route-Table` — and are now genuinely public. The change was immediate, no provisioning delay. I confirmed by checking the individual subnet detail pages — each now shows `Public-Route-Table` under Route table.

The key realisation here: naming a subnet `Public-Subnet-AZ-A` on Day 1 meant nothing until this step. The subnet became public at exactly the moment I saved the route table association. Before that, it was a private subnet with a misleading name.

### What I Learned

- Route table association is the moment a subnet becomes public. Not when it is named. Not when auto-assign IP is enabled. This step.
- Multiple subnets can share the same route table. Both public subnets point to the same `Public-Route-Table`. This is correct and efficient — one route table for all public subnets means one place to update if the IGW ever changes.
- The main route table now only has the private subnets associated with it. Any new subnet created in this VPC will default to the main route table (private), which is the safe default behaviour I want.

---

## Lab 4: Verify Private Subnets Have No Internet Route

### Steps

1. Navigate to **VPC → Route tables**.
2. Select the main route table for `OluTech-Production-VPC` (the one not named `Public-Route-Table`).
3. Click the **Routes tab**. Confirm it shows only: `10.0.0.0/16 → local`. No `0.0.0.0/0` entry.
4. Click **Subnet associations**. Confirm `Private-Subnet-App-AZ-A` and `Private-Subnet-DB-AZ-B` are associated here.
5. Verify neither private subnet has a route to the internet in any form.

### What I Observed

The main route table has exactly one route: `10.0.0.0/16 → local`. Both private subnets are associated with it. This confirms that traffic from those subnets can only go to other resources within the VPC — there is no path to the internet in either direction. No IGW route, no NAT Gateway route.

This is the configuration I would show a CBN auditor for the data subnet: open the route table, point to the single `local` entry, and confirm no `0.0.0.0/0` exists. That is the architectural evidence of isolation.

### What I Learned

- The verification step is not bureaucratic. It is a check against the most common misconfiguration in production VPCs: accidentally associating a private subnet with the public route table, which exposes it to the internet.
- A route table with only the `local` route is the correct and complete configuration for a data tier subnet in a regulated environment. Nothing else is needed. Nothing else should be there.
- VPC Flow Logs on private subnets would capture any unexpected outbound attempts — even though there is no route, they are useful for detecting misconfiguration over time.

---

## Lab 5: Draw the Traffic Flow Diagram

### Steps

1. Open Excalidraw.
2. Draw the VPC boundary box labelled `OluTech-Production-VPC | 10.0.0.0/16`.
3. At the top, outside the VPC, draw the **Internet** and an **Internet Gateway** icon labelled `OluTech-IGW`.
4. Inside the VPC, draw:
   - **Public Subnet** (`10.0.1.0/24`) containing an ALB and a Bastion Host
   - **Private-App Subnet** (`10.0.10.0/24`) containing an EC2 App Server
   - **Private-Data Subnet** (`10.0.20.0/24`) containing an RDS instance
5. Draw arrows for traffic flow:
   - Internet → IGW (arrow labelled: `0.0.0.0/0 → igw-xxxxxxxx`)
   - IGW → ALB in Public Subnet (arrow labelled: `Route: Public-Route-Table`)
   - ALB → EC2 App Server in Private Subnet (arrow labelled: `local route / Security Group`)
   - EC2 App Server → RDS in Private-Data Subnet (arrow labelled: `local route only | port 5432`)
6. Draw a red X or blocked arrow showing: Private Subnets → Internet (blocked — no route exists).
7. Add a note: `Main Route Table: 10.0.0.0/16 → local ONLY. No 0.0.0.0/0 entry.`
8. Export as PNG.

### What I Observed

Drawing the traffic flow with explicit route labels made the routing logic click in a way that just reading about it doesn't. When I labelled the arrow from the internet to the ALB with the route entry `0.0.0.0/0 → igw-xxxxxxxx`, I could see exactly why removing that route entry would break that arrow — and why the private subnets, which have no such entry, have no corresponding arrow to the internet.

The blocked arrow showing private subnets cannot reach the internet is the most important element of the diagram. It is the visual proof of isolation.

### What I Learned

- Diagramming with route table annotations is how you explain architecture to non-technical stakeholders, auditors, and interviewers. It shows that you understand the mechanism, not just the names.
- The traffic flow diagram is a portfolio artefact. Any interview or assessment that asks about VPC architecture — this diagram is the answer.
- Routing logic flows from left to right in the diagram: internet to public tier, public tier to private tier, private tier to data tier. Each transition is controlled by a specific route table entry or Security Group rule. Understanding which layer controls each transition is the SAA-C03 architecture question pattern.

---

## Bonus Lab: NAT Gateway — Add Outbound Internet to Private Subnets

### Steps

1. Navigate to **VPC → NAT gateways → Create NAT gateway**.
2. Name: `OluTech-NAT-GW`.
3. Subnet: `Public-Subnet-AZ-A` (NAT Gateway must live in a public subnet).
4. Connectivity type: Public.
5. Elastic IP: Click **Allocate Elastic IP** to create a new EIP. Select it.
6. Click **Create NAT gateway**. Wait ~2 minutes for state to show `Available`.
7. Navigate to **VPC → Route tables**.
8. Select the main route table (used by private subnets).
9. **Routes tab → Edit routes → Add route**:
   - Destination: `0.0.0.0/0`
   - Target: NAT Gateway → select `OluTech-NAT-GW`
10. Save changes.
11. Confirm the private route table now shows: `10.0.0.0/16 → local` AND `0.0.0.0/0 → nat-xxxxxxxx`.
12. **IMPORTANT — DELETE after 15 minutes to avoid cost:** NAT Gateway costs ~$0.045/hour. Navigate to **NAT gateways → Actions → Delete NAT gateway**. Also release the Elastic IP: **Elastic IPs → Actions → Release Elastic IP address**.

### What I Observed

The NAT Gateway required the Elastic IP allocation before it could be created — the EIP allocation is a separate step that is easy to miss on first attempt. After creation, provisioning took about 90 seconds to reach `Available` state.

The difference from the IGW route is clear from the route table: the IGW route (`0.0.0.0/0 → igw-xxxxxxxx`) enables two-way internet communication for resources in the subnet. The NAT Gateway route (`0.0.0.0/0 → nat-xxxxxxxx`) enables OUTBOUND only — the NAT Gateway translates the private instance's IP to its Elastic IP for outbound connections and blocks all unsolicited inbound. A private subnet instance with the NAT GW route can download patches and call external APIs. Nothing from the internet can initiate a connection to it.

I deleted the NAT Gateway and released the EIP within 15 minutes. The private route table went back to `local` only after I removed the route entry.

### What I Learned

- NAT Gateway requires an Elastic IP. No EIP = cannot create NAT GW. This is a prerequisite, not an optional step.
- NAT Gateway must be in a PUBLIC subnet — it needs the IGW route to reach the internet on behalf of private instances. A NAT GW in a private subnet does nothing.
- The NAT GW route is `0.0.0.0/0 → nat-xxxxxxxx`. The IGW route is `0.0.0.0/0 → igw-xxxxxxxx`. Both use the same destination but completely different targets with completely different behaviour.
- Cost management: NAT Gateways bill per hour regardless of usage. Delete them in lab environments after testing. This is a real money trap for AWS learners who forget running resources.
