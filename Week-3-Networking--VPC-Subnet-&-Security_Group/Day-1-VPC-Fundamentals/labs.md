# Labs — Week 3 Day 1: VPC Fundamentals

---

## Lab 1: Plan the VPC Architecture Before Touching the Console

### Steps

1. Open Excalidraw (excalidraw.com) on a second screen.
2. Draw a large rectangle labelled `OluTech-Production-VPC | 10.0.0.0/16`.
3. Inside the VPC boundary, draw two columns — one for AZ-a, one for AZ-b.
4. In each column, draw two subnet boxes: one green (public), one blue (private).
5. Label each subnet with its name and CIDR:
   - `Public-Subnet-AZ-A` | `10.0.1.0/24`
   - `Public-Subnet-AZ-B` | `10.0.2.0/24`
   - `Private-Subnet-App-AZ-A` | `10.0.10.0/24`
   - `Private-Subnet-DB-AZ-B` | `10.0.20.0/24`
6. Add an Internet Gateway icon at the top of the diagram connected to both public subnets.
7. Add a NAT Gateway icon inside each public subnet connected to the private subnets.
8. Label route table logic: `Public RT: 0.0.0.0/0 → IGW` and `Private RT: 0.0.0.0/0 → NAT GW`.
9. Export as PNG.

### What I Observed

Drawing the diagram first changed how I saw the console work. When I created the subnets one by one, I already knew where each one sat in the architecture — the diagram was the mental map and the console steps were just confirming what I had already decided. The gap between `10.0.1.0/24` (public) and `10.0.10.0/24` (private) is intentional: it leaves address space for future subnets between `/24` ranges without overlapping.

### What I Learned

- Planning before clicking is a professional habit, not a beginner step — I plan, then I build.
- CIDR ranges don't need to be sequential. Leaving gaps (`10.0.1.0/24`, `10.0.2.0/24` then jumping to `10.0.10.0/24`) is deliberate capacity planning for future subnet additions.
- The diagram is the deliverable that goes into portfolio. Screenshots are evidence. The diagram is the artefact.
- Visualising IGW at the top, NAT Gateway inside the public subnet, and private subnets pointing to NAT made the routing logic obvious in a way that a text description alone doesn't.

---

## Lab 2: Create the VPC

### Steps

1. Open the AWS Management Console. Navigate to **VPC → Your VPCs → Create VPC**.
2. Select **VPC only** (not "VPC and more" — we build manually to understand every layer).
3. Name tag: `OluTech-Production-VPC`.
4. IPv4 CIDR: `10.0.0.0/16`.
5. Tenancy: `Default`.
6. Click **Create VPC**.
7. Confirm the VPC appears in the **Your VPCs** list with the correct CIDR and state `available`.

### What I Observed

The "VPC and more" option would have created all subnets, route tables, and gateways automatically. I deliberately chose "VPC only" to build each component manually and understand what each one does. After creation the VPC state shows `available` immediately — there is no provisioning delay. The console also auto-creates a default route table, a default NACL, and a default security group for every new VPC. I noted these exist but did not use the defaults for the production configuration.

### What I Learned

- AWS auto-creates a main route table, default NACL, and default security group for every new VPC. These are there even if you never configured them.
- The main route table only has the `local` route by default — no internet access. This is correct default behaviour.
- "VPC and more" is a convenience shortcut for demos. In production, building manually means you understand what you own and why each component is there.

---

## Lab 3: Create the Four Subnets

### Steps

1. Navigate to **VPC → Subnets → Create subnet**.
2. Select `OluTech-Production-VPC` from the VPC dropdown.
3. **Subnet 1:**
   - Subnet name: `Public-Subnet-AZ-A`
   - Availability Zone: select first AZ (`af-south-1a` or equivalent)
   - IPv4 CIDR: `10.0.1.0/24`
4. Click **Add new subnet**.
5. **Subnet 2:**
   - Subnet name: `Public-Subnet-AZ-B`
   - Availability Zone: second AZ (`af-south-1b`)
   - IPv4 CIDR: `10.0.2.0/24`
6. Click **Add new subnet**.
7. **Subnet 3:**
   - Subnet name: `Private-Subnet-App-AZ-A`
   - Availability Zone: first AZ (`af-south-1a`)
   - IPv4 CIDR: `10.0.10.0/24`
8. Click **Add new subnet**.
9. **Subnet 4:**
   - Subnet name: `Private-Subnet-DB-AZ-B`
   - Availability Zone: second AZ (`af-south-1b`)
   - IPv4 CIDR: `10.0.20.0/24`
10. Click **Create subnet**.
11. Filter the Subnets list by `OluTech-Production-VPC`. Confirm all four subnets appear with correct names, AZs, and CIDRs.

### What I Observed

All four subnets are created in a single console workflow using the "Add new subnet" flow — I didn't need to navigate away and repeat the process four times. Every subnet shows 251 available IPv4 addresses regardless of whether it is public or private — because at this point all subnets are private by default. None has internet access yet. The console shows the correct AZ assignment for each subnet. One thing I confirmed: all four subnets are currently associated with the main route table (no IGW route). They are all private at this point, including the ones I plan to make public.

### What I Learned

- Subnets are private by default at creation. There is no "make public" toggle — you make a subnet public by configuring its route table.
- 251 available IPs in a `/24` confirms the 5 AWS reserved addresses: `.0`, `.1`, `.2`, `.3`, `.255`.
- The multi-subnet creation flow in a single workflow saves time and prevents context-switching errors.
- All subnets land on the main route table automatically. The next step — route table configuration — is what actually differentiates public from private.

---

## Lab 4: Enable Auto-Assign Public IP on Public Subnets

### Steps

1. In the Subnets list, select `Public-Subnet-AZ-A`.
2. Click **Actions → Edit subnet settings**.
3. Check the box: **Enable auto-assign public IPv4 address**.
4. Click **Save**.
5. Select `Public-Subnet-AZ-B`.
6. Repeat: **Actions → Edit subnet settings → Enable auto-assign public IPv4 address → Save**.
7. Confirm `Private-Subnet-App-AZ-A` and `Private-Subnet-DB-AZ-B` have auto-assign OFF. Do not modify them.

### What I Observed

The setting is a subnet-level toggle in the "subnet settings" panel. It is not in the main Create VPC flow. This is intentional — it is a separate decision from creating the subnet. After enabling it, the subnet detail page shows "Auto-assign public IPv4 address: Yes" for the public subnets and "No" for the private ones. At this point the subnets still have no internet access — this setting only controls whether EC2 instances launched here automatically get a public IP. The route table is still what determines whether that public IP can actually reach the internet.

### What I Learned

- Auto-assign public IP is a convenience feature. It does not make the subnet public. A subnet with auto-assign ON but no IGW route is still private — the public IP assigned to instances in it is unreachable from the internet.
- The correct mental model: auto-assign IP is about the resource having an address; the route table is about whether that address can communicate with the internet. Both are required for a functional public subnet.
- Private subnets should always have auto-assign OFF. Assigning a public IP to an instance in a private subnet wastes an IP and creates confusion. It will never work without the IGW route.

---

## Lab 5: Verify and Diagram

### Steps

1. Navigate to **VPC → Your VPCs**. Confirm `OluTech-Production-VPC` exists with CIDR `10.0.0.0/16` and state `available`.
2. Navigate to **VPC → Subnets**. Filter by `OluTech-Production-VPC`.
3. Confirm all four subnets are present:
   - `Public-Subnet-AZ-A` | `10.0.1.0/24` | `af-south-1a` | auto-assign: Yes
   - `Public-Subnet-AZ-B` | `10.0.2.0/24` | `af-south-1b` | auto-assign: Yes
   - `Private-Subnet-App-AZ-A` | `10.0.10.0/24` | `af-south-1a` | auto-assign: No
   - `Private-Subnet-DB-AZ-B` | `10.0.20.0/24` | `af-south-1b` | auto-assign: No
4. Return to Excalidraw. Update the diagram to include auto-assign labels on the public subnets.
5. Export the final diagram as PNG.
6. Add PNG to `/screenshots/vpc-architecture-diagram.png`.

### What I Observed

The verification step is where I caught that the private subnets were still on the main route table — which is correct at this stage since we haven't created the IGW or NAT Gateway yet. The subnets are built correctly. The routing configuration is the next step in production (not covered in this lab). The Excalidraw export produced a clean PNG that I can add to the GitHub repo.

### What I Learned

- Verification before moving to the next step is a production habit. Finding an error in subnet CIDR at this point costs 30 seconds. Finding it after launching EC2 instances costs hours.
- The main route table having only the `local` route at this point is expected. It confirms the VPC is correctly isolated from the internet until we explicitly create the IGW and route table entries.
- The architecture diagram is a professional portfolio artefact. Future employers and interviewers will see this — labelling every component with IP ranges and AZ assignments demonstrates the depth of understanding, not just that I clicked through a console lab.

---

## Bonus Lab: Default VPC vs Custom VPC Comparison

### Steps

1. Navigate to **VPC → Your VPCs**. Find the default VPC (usually CIDR `172.31.0.0/16`).
2. Click through the default VPC and note: CIDR, number of subnets, route tables, IGW attachment, subnet settings.
3. Compare each attribute to `OluTech-Production-VPC`.
4. Write the 5 differences.

### What I Observed

The default VPC is more permissive than I expected. AWS makes it easy to launch resources quickly by defaulting to a fully public configuration — every subnet in the default VPC has auto-assign public IP enabled and a route to an internet gateway. This is convenient for demos and experimentation but disqualifying for production use.

**Five differences I documented:**

1. **CIDR block:** Default VPC uses `172.31.0.0/16`. My custom VPC uses `10.0.0.0/16`. The `172.31.x.x` range is harder to peer with on-premises networks that often use `172.x.x.x` private ranges.
2. **Subnet type:** All default VPC subnets have auto-assign public IP enabled and route directly to the IGW — they are all public. My custom VPC separates subnets into public and private tiers.
3. **Auto-assign public IP:** Default subnets assign a public IP to every instance automatically. Private subnets in my VPC have this disabled — instances in those subnets are unreachable from the internet even if accidentally misconfigured.
4. **Route table design:** Default VPC has one route table with `0.0.0.0/0 → IGW` used by all subnets. My VPC separates public and private route tables — private subnets cannot accidentally be given internet access by someone adding a single route.
5. **Network isolation:** The default VPC has no private subnets, no NAT Gateway structure, and no separation between app and data tiers. Any RDS database launched into the default VPC would be in a public subnet. This violates CBN and SEC network segmentation requirements for Nigerian financial services — and it would fail a security audit immediately.

### What I Learned

- Never use the default VPC in production. Not because it is technically impossible to secure it, but because the defaults are optimised for experimentation, not security, and the risk of misconfiguration is too high.
- The separation between public and private subnets is not just a preference — it is a structural defence. A developer cannot accidentally expose a database to the internet in a properly designed custom VPC because the route table prevents it by default.
- For Cowrywise, Kuda, or any CBN-regulated platform: the architecture diagram showing private data subnets with isolated route tables is not optional. It is what you show an auditor.
