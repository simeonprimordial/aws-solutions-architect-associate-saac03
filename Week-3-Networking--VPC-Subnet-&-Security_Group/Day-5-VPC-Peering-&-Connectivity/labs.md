# Labs — Week 3 Day 5: VPC Peering & Connectivity

---

## Lab 1: Set Up Your Diagramming Tool

### Steps

1. Go to **app.diagrams.net** (draw.io — free, browser-based, no signup required).
2. In draw.io: **Extras → Edit Diagram** or use the search panel to find AWS shape libraries.
3. Enable the AWS icon set: **View → Shapes → Search "AWS"** or go to **Edit → XML** and use the built-in AWS icon libraries.
4. Alternatively, download the official AWS architecture icons at `https://aws.amazon.com/architecture/icons/` — the icon pack includes the `Resource Icons` and `Category Icons` folders used in official AWS diagrams.
5. Create a new blank diagram. Set the canvas to landscape orientation: **File → Page Setup → Landscape**.

### What I Observed

draw.io has a built-in AWS icon library that activates from the Shape panel on the left — search for "AWS" and a full library appears with EC2, RDS, VPC, IGW, NAT Gateway, and all other service icons used in professional AWS diagrams. The official AWS icon download is a large ZIP file with vector assets in multiple formats — the draw.io built-in library is sufficient for this diagram and faster to access.

### What I Learned

- Professional AWS diagrams use consistent icons — EC2 looks like the orange compute box, RDS looks like the cylinder, ALB looks like the load balancer symbol. Using the correct icons signals AWS familiarity to any reviewer.
- draw.io exports to PNG, PDF, SVG, and XML. Export at high resolution (300 DPI or 2x scale) for portfolio quality.

---

## Lab 2: Layer 1 — VPC and Availability Zone Structure

### Steps

1. Draw a large rectangle with a dashed border — the **VPC boundary**. Label it: `OluTech-Production-VPC (10.0.0.0/16)`.
2. Inside the VPC, draw two vertical columns with lighter borders. Label each: **Availability Zone A** and **Availability Zone B**.
3. In each AZ column, draw two horizontal boxes:
   - Top box: **Public Subnet** — green background. Label with CIDR: `10.0.1.0/24` (AZ-A) and `10.0.2.0/24` (AZ-B).
   - Bottom box: **Private Subnet** — blue background. Label with CIDR: `10.0.10.0/24` (AZ-A) and `10.0.20.0/24` (AZ-B).
4. Add small route table annotations inside each subnet box:
   - Public Subnet: `RT: 0.0.0.0/0 → IGW`
   - Private Subnet: `RT: 10.0.0.0/16 → local`

### What I Observed

The colour coding — green for public, blue for private — is the AWS convention used in official architecture diagrams. Using it immediately signals to any reader which tier is internet-facing. The route table annotations inside each subnet box are the detail that differentiates this diagram from a simple topology sketch: they show that I understand the mechanism, not just the names.

### What I Learned

- Subnet boxes should be sized proportionally — the public subnets are smaller (fewer resources) than the private subnets in a production design. The diagram should reflect the real resource distribution.
- Adding CIDR labels to every subnet is essential. A diagram without CIDRs is an illustration. A diagram with CIDRs is architecture documentation.

---

## Lab 3: Layer 2 — Internet Connectivity Components

### Steps

1. Outside the VPC box at the top, draw a **cloud icon** labelled `Internet`.
2. Below the internet icon, draw the **Internet Gateway** icon. Label: `OluTech-IGW`.
3. Draw an arrow from Internet → IGW, and from IGW → both Public Subnets. Label the arrow: `0.0.0.0/0 → IGW`.
4. Inside **Public Subnet AZ-A**, place a **NAT Gateway** icon. Label it: `OluTech-NAT-GW` with a small `EIP` label showing it has an Elastic IP.
5. Draw an arrow from NAT Gateway → Private Subnet AZ-A. Label: `0.0.0.0/0 → NAT GW (outbound only)`.
6. Add a note: NAT Gateway is AZ-scoped. Private Subnet AZ-B would need its own NAT GW for HA.

### What I Observed

Showing the NAT Gateway in one AZ only — and adding the HA note — demonstrates understanding beyond the basic diagram. Most portfolio diagrams show a single NAT Gateway. Noting that production would require one per AZ for resilience shows architectural maturity.

The arrow labels are critical. Writing `0.0.0.0/0 → IGW` on the arrow from the public subnet to the internet is the specific mechanism. Anyone reading the diagram can trace the packet path just from the arrow labels.

### What I Learned

- The Internet Gateway sits outside the VPC boundary in diagrams. It is a VPC-level resource but it bridges the VPC to the internet — placing it at the edge of the VPC boundary visually communicates its role correctly.
- One-directional arrows on NAT Gateway output (`outbound only`) are important. The NAT Gateway does not accept inbound connections — the arrow should be directional, not bidirectional.

---

## Lab 4: Layer 3 — Compute, Database, and Security Groups

### Steps

1. In both **Public Subnet** boxes, place an **ALB (Application Load Balancer)** icon. Draw a bracket spanning both public subnets to show the ALB is multi-AZ. Label: `ALB | SG-LoadBalancer | TCP 443/80 from 0.0.0.0/0`.
2. In both **Private Subnet AZ-A and AZ-B** (upper tier), place **EC2** icons. Label: `App Server | SG-WebServers | TCP 80 from SG-LoadBalancer`.
3. In both **Private Subnet AZ-A and AZ-B** (lower tier — add a second private subnet row), place **RDS** icons. Label: `MySQL RDS | SG-Database | TCP 3306 from SG-WebServers`.
4. Draw **dashed arrows** showing the Security Group chain:
   - ALB → App Server: dashed arrow labelled `TCP 80 | SG ref`
   - App Server → RDS: dashed arrow labelled `TCP 3306 | SG ref`
5. Use distinct colours for the dashed arrows: orange for SG-LoadBalancer, teal for SG-WebServers, gold for SG-Database.

### What I Observed

The Security Group chain arrows are the most technically dense part of the diagram. Drawing them as dashed (to distinguish from network topology arrows) and labelling with both the port and the reference type (`SG ref`) shows the chaining pattern explicitly. This is the piece that demonstrates Day 3 and Day 4 knowledge in a single visual element.

Adding the subnet row for the data tier required expanding the diagram. In production the data tier subnets would have their own CIDRs — `10.0.20.0/24` (AZ-A) and `10.0.21.0/24` (AZ-B) — and would be on the `isolated-rt` route table with local-only routing. I added this as a third row in each AZ column.

### What I Learned

- Three-tier architecture has three subnet rows per AZ: public (ALB), private-app (EC2), private-data (RDS). Adding the data tier subnets makes the diagram accurate to what a production deployment would look like.
- The Security Group chain is the differentiator between this diagram and a generic 3-tier diagram from a textbook. The SG arrows and labels show that the security model is documented, not just the topology.

---

## Lab 5: Polish, Export, and GitHub Commit

### Steps

1. **Add a title** at the top of the diagram: `OluTech Solutions — 3-Tier AWS Architecture`
2. **Add the date** in the bottom corner: `Week 3 — [date]`
3. **Add a legend** in the bottom left:
   - Green box = Public Subnet
   - Blue box = Private Subnet (App)
   - Orange box = Private Subnet (Data)
   - Solid arrow = Network traffic flow
   - Dashed arrow = Security Group chain
4. **Add your name** in the diagram footer.
5. **Export:**
   - File → Export As → PNG → Scale 200% → Export
   - File → Export As → PDF → Export
6. Open the GitHub repo `simeonprimordial/aws-solutions-architect-associate-saac03`.
7. Navigate to the `Week-3-Day5/` folder. Upload the PNG file.
8. Commit message: `Add Week 3 VPC architecture diagram`.
9. Take a screenshot of the GitHub commit.

### What I Observed

The 200% PNG export produced a high-resolution file suitable for LinkedIn and for the GitHub README preview. At 100%, the subnet CIDR labels were too small to read in the LinkedIn preview — scaling up makes the diagram legible without zooming.

The GitHub commit is as important as the diagram itself for the portfolio. It shows: a working Git workflow, organised repository structure, and a clear commit message. Recruiters and hiring managers who visit the repo see the diagram immediately in the file listing.

### What I Learned

- Professional diagrams always have a title, date, author, and legend. Without a legend, colour-coding is meaningless to anyone who did not create the diagram.
- The GitHub commit message should be descriptive enough to find the diagram in a commit history: `Add Week 3 VPC architecture diagram` is clear. `Update` or `Changes` is not.
- Exporting both PNG (for embedding) and PDF (for full-quality printing and sharing) is professional practice.

---

## Bonus Lab: Add VPC Peering to the Diagram

### Steps

1. Outside the existing VPC boundary, draw a second smaller VPC box. Label: `OluTech-Dev-VPC (10.1.0.0/16)`.
2. Draw a dashed line connecting `OluTech-Production-VPC` to `OluTech-Dev-VPC`. Label it: `VPC Peering (pcx-xxxxx)`.
3. Add route table annotations to both VPCs:
   - Production VPC route table: `10.1.0.0/16 → pcx-xxxxx`
   - Dev VPC route table: `10.0.0.0/16 → pcx-xxxxx`
4. Add a red warning label next to the peering connection: `Non-Transitive: if a 3rd VPC peers Dev, Production cannot reach it through Dev`.
5. Export updated PNG and re-commit to GitHub.

### What I Observed

Adding the second VPC and the peering connection visually demonstrated why CIDR planning matters — the Dev VPC is `10.1.0.0/16`, clearly distinct from the Production `10.0.0.0/16`. If I had used `10.0.0.0/16` for both, they would overlap and the peering would be impossible. The diagram itself enforced the lesson from Day 1.

The non-transitive warning label is what elevates the bonus diagram from a topology illustration to architecture documentation. Any engineer reading the diagram understands the limitation without needing to consult external documentation.

### What I Learned

- CIDR planning is a prerequisite for multi-VPC architectures. The inability to change a VPC CIDR after creation means peering conflicts discovered later are expensive to fix — requiring VPC replacement or complex CIDR expansion.
- The non-transitive note in the diagram is the kind of proactive documentation that separates junior and senior-level architecture work. Documenting known limitations in the diagram itself prevents the limitation from becoming a production surprise later.
