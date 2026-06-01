# Challenges & Blockers — Week 3 Day 2: Subnets — Public vs Private

---

## Challenge 1: Attached the Internet Gateway to the Wrong VPC

**What happened:**
When attaching `OluTech-IGW` to a VPC, the dropdown showed multiple VPCs — including the default VPC (`172.31.0.0/16`) and `OluTech-Production-VPC` (`10.0.0.0/16`). I selected the first entry in the list without checking the CIDR, which happened to be the default VPC. The attachment succeeded with no warning. I only noticed the error when I checked the IGW's VPC association and saw `172.31.0.0/16` instead of `10.0.0.0/16`.

**What I tried:**
- Looked for a way to reassign the IGW without detaching — there isn't one.
- Went to **Actions → Detach from VPC** on the incorrectly attached IGW.
- Re-attached to `OluTech-Production-VPC` and confirmed the correct VPC ID.

**Resolution:**
An IGW can only be attached to one VPC at a time. I detached it from the default VPC and reattached to the production VPC. The whole process took about 2 minutes. No data loss, no resource impact — the IGW had not been used yet.

**Lesson learned:**
Always verify the VPC ID (not just the name) before confirming any attachment. VPC IDs start with `vpc-` followed by a hex string — confirm the CIDR matches what you expect. In production, this error could mean internet traffic accidentally routing through the wrong VPC.

---

## Challenge 2: Accidentally Deleted the Local Route

**What happened:**
While editing routes on the Public Route Table, I misread the interface and clicked the delete icon next to the `10.0.0.0/16 → local` route instead of the edit icon. The console accepted the deletion attempt (though it ultimately prevents you from saving a route table with no local route). I panicked briefly thinking I had broken the VPC routing entirely.

**What I tried:**
- Tried to save the route table without the `local` route — the console threw a validation error: "A local route cannot be deleted."
- Refreshed the page — the `local` route reappeared because the save had not gone through.
- Proceeded to add the `0.0.0.0/0 → IGW` route correctly.

**Resolution:**
AWS prevents deletion of the `local` route at the API level — the console will not save a route table that removes it. My panic was unnecessary. The validation error was a safety net, not a critical failure.

**Lesson learned:**
The `local` route is immutable and protected by AWS. You cannot delete it through the console or API. This is a hard constraint by design — VPC-internal routing is always preserved. I now know to read the interface carefully before clicking delete on any route table entry.

---

## Challenge 3: NAT Gateway Elastic IP Step Was Not Obvious

**What happened:**
During the bonus lab, I clicked through the Create NAT gateway form and reached the Elastic IP field. I assumed I could proceed without one or that AWS would auto-create an EIP — I clicked Create and got an error: "An Elastic IP address must be associated with a NAT Gateway." There was no auto-create option visible and I had not allocated an EIP yet.

**What I tried:**
- Looked for an EIP allocation option within the NAT Gateway creation form — it exists but is easy to miss: **Allocate Elastic IP** is a small link inside the EIP field, not a prominent button.
- Navigated to **VPC → Elastic IPs → Allocate Elastic IP address** separately, then returned to NAT Gateway creation.
- Eventually found the **Allocate Elastic IP** link directly on the creation form and used it inline.

**Resolution:**
The `Allocate Elastic IP` button on the NAT Gateway creation form allocates a new EIP in one click and auto-selects it for the NAT Gateway. I should have used this from the start instead of trying to pre-create the EIP separately. Once I used the inline allocation, the NAT Gateway was created without further issues.

**Lesson learned:**
NAT Gateway requires an Elastic IP — this is not optional. The exam will test this: a scenario where a NAT Gateway is created but instances still cannot reach the internet should trigger a check of whether an EIP was allocated and associated. Also: remember to release the EIP after deleting the NAT Gateway — allocated but unattached EIPs cost money.

---

## Challenge 4: Confusion Between Main Route Table and Custom Route Table

**What happened:**
After creating `Public-Route-Table`, I was looking at the route tables list and could not immediately identify which one was the main route table for the VPC. Both showed `OluTech-Production-VPC` as the VPC. The main route table had no name tag and looked identical in the list to `Public-Route-Table` except for not having the `0.0.0.0/0` route.

**What I tried:**
- Clicked into each route table and checked the Routes tab to distinguish them — the main route table had only the `local` route.
- Found the "Main" column in the route tables list view — it shows "Yes" for the main route table. I had scrolled past this column.

**Resolution:**
The route tables list has a "Main" column that explicitly identifies which route table is the main (default) route table for the VPC. Once I found it, the distinction was clear. I added a name tag to the main route table — `Private-Route-Table` — to make the distinction obvious in future. Named objects in the AWS console are significantly easier to manage than unnamed ones with auto-generated IDs.

**Lesson learned:**
Always name your AWS resources at creation time. An unnamed route table in a VPC with multiple route tables is a support ticket waiting to happen. The naming convention from the slides — `public-rt`, `private-app-rt`, `isolated-rt` — is not just aesthetic. It is operational hygiene that prevents exactly this kind of confusion.

---

## Challenge 5: Misunderstanding the Traffic Flow — Private to Private Routing

**What happened:**
While drawing the traffic flow diagram, I initially drew an arrow from the private-app subnet to the private-data subnet and labelled it "via local route." Then I second-guessed myself: does traffic between two private subnets actually use the `local` route entry? Or does it need an additional route? I wasn't sure whether the `local` route covered all VPC-internal traffic or only the specific subnet.

**What I tried:**
- Re-read the route table mechanics in the slides: "The `local` route covers the entire VPC CIDR — `10.0.0.0/16 → local` means any destination within `10.0.0.0/16` stays inside the VPC."
- Confirmed: `10.0.10.0/24` (app subnet) and `10.0.20.0/24` (data subnet) are both within `10.0.0.0/16`, so the `local` route covers traffic between them.
- No additional route entry is needed for cross-subnet VPC-internal traffic.

**Resolution:**
The `local` route covers the entire VPC CIDR block. Any destination IP within `10.0.0.0/16` is handled by the `local` route — including traffic between any two subnets within the VPC. The Security Group on the RDS instance is what controls whether the app server can actually establish a connection on port `5432`. The route allows the path. The Security Group controls access.

**Lesson learned:**
The `local` route is universal for VPC-internal traffic. There is no need to add specific subnet-to-subnet routes for resources within the same VPC. The Security Group is the access control layer for intra-VPC connections — not the route table.

---

*Add new challenges here as they come up in future days.*
