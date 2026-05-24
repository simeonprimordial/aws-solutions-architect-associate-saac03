# AWS Global Infrastructure Labs — Day 2

---

## Lab 1: AWS Global Infrastructure Map Exploration

**Objective:** Use the interactive AWS map to document Regions and identify the closest to Nigeria.

### Steps
1. Visit [https://infrastructure.aws](https://infrastructure.aws) — the interactive AWS global map
2. Identify all AWS Regions shown in **Africa**
3. Click on **US East (N. Virginia)** — note the number of Availability Zones
4. Click on **af-south-1 (Cape Town)** — note its AZ count and available services
5. Find the nearest **Edge Location** to Nigeria — check for Lagos or West Africa

### Findings

**Africa Regions:**
| Region | Code | AZs | Notes |
|---|---|---|---|
| Cape Town | af-south-1 | 3 | Closest to Nigeria |

**Edge Locations near Nigeria:**
- Lagos, Nigeria — closest PoP for CloudFront/Route 53

**AZ count comparison:**
- us-east-1 (N. Virginia): 6 AZs — largest Region globally
- af-south-1 (Cape Town): 3 AZs

### What I Observed
The interactive map makes it visually clear how unevenly distributed AWS infrastructure is globally. North America and Europe are densely covered; Africa has a single Region in Cape Town with more regions listed as "coming soon."

### What I Learned
- The density of Regions in a geography directly reflects cloud maturity and demand in that market.
- For Nigerian workloads, there is currently no closer option than af-south-1. This makes Edge Locations (Lagos PoP) even more important for latency-sensitive applications.

---

## Lab 2: Console Region Switching & AZ Verification

**Objective:** Use the AWS Console to verify Region and AZ configurations firsthand.

### Steps
1. Sign in to the AWS Console
2. Click the **Region selector** (top-right corner) — count total Regions listed
3. Switch to **af-south-1 (Cape Town)** — note any services showing as unavailable
4. Switch to **us-east-1 (N. Virginia)** — the Region with the broadest service coverage
5. Open **EC2 Dashboard** → note the Availability Zones listed for us-east-1

### Findings

**Total Regions in console dropdown:** ~33 (some Regions require opt-in before they appear)

**af-south-1 observations:**
- Core services available: EC2, S3, RDS, VPC, IAM, Lambda, CloudFront
- Some services show as unavailable vs us-east-1

**us-east-1 EC2 AZs:**
- us-east-1a, us-east-1b, us-east-1c, us-east-1d, us-east-1e, us-east-1f (6 AZs)

### What I Observed
Switching Regions in the console changes the entire service view — resources created in us-east-1 are completely invisible when you switch to af-south-1. This is by design; Regions are fully isolated.

### What I Learned
- **Region switching is a common real-world mistake.** Many beginners create an EC2 instance in us-east-1, switch Regions, and think the instance "disappeared." It's still there — just invisible from the wrong Region.
- Always confirm your Region before creating resources. The Region name is displayed in the top-right corner of every console page.

---

## Lab 3: Build Your AWS Region Cheat Sheet

**Objective:** Create a permanent reference document covering key Regions, AZ counts, and Nigerian market context.

### Steps
1. Open a text editor or Notion
2. Create a table: Region Name | Region Code | AZ Count | Key Services | Notes
3. Fill in at least 6 rows (see table below)
4. Add a "Key Facts" section with the 3 exam traps
5. Add a "Nigeria Context" section with latency data and NDPC notes
6. Save as a permanent reference document — to be updated every week

### Completed Cheat Sheet

**AWS Regions Reference**

| Region Name | Code | AZs | Key Services | Notes |
|---|---|---|---|---|
| US East (N. Virginia) | us-east-1 | 6 | All services | Most services available here first |
| US West (Oregon) | us-west-2 | 4 | All services | Common for US West Coast workloads |
| EU (Ireland) | eu-west-1 | 3 | All services | GDPR-compliant EU data residency |
| Africa (Cape Town) | af-south-1 | 3 | Core services | Closest to Nigeria — requires opt-in |
| Asia Pacific (Singapore) | ap-southeast-1 | 3 | All services | Hub for Southeast Asia |
| South America (São Paulo) | sa-east-1 | 3 | Most services | Closest Region for Brazil |

**Key Facts to Remember:**
1. AZs connect via **private fibre** — NOT the public internet
2. Edge Locations are for **caching and DNS only** — no EC2 or RDS
3. Data **does not automatically** leave a Region — cross-region replication must be configured

**Nigeria Context:**
- Nearest Region: af-south-1 (Cape Town) — ~85ms from Lagos
- vs eu-west-1 (Ireland): ~140ms from Lagos
- vs us-east-1 (Virginia): ~185ms from Lagos
- Edge Location in Lagos means CloudFront content loads in <20ms locally
- NDPC compliance: use af-south-1 to keep Nigerian personal data on the African continent

### What I Observed
Building the table manually forced me to actually look up each Region rather than passively reading. The latency numbers make the af-south-1 choice concrete — it's not just "closest on a map," it's nearly half the latency of a European Region.

### What I Learned
- A personal reference document you build yourself is more useful than any pre-made cheat sheet because you remember creating each entry.
- The Region opt-in requirement for af-south-1 is something many Nigerians building on AWS miss — you have to explicitly enable it in Account Settings before resources can be deployed there.

---

## Lab 4: Publish to GitHub Gist

**Objective:** Publish your Region cheat sheet as a public portfolio artifact.

### Steps
1. Go to [https://gist.github.com](https://gist.github.com)
2. Paste your cheat sheet content in Markdown format
3. Title it: `AWS Regions Cheat Sheet — Week 1`
4. Set visibility to **Public**
5. Click **Create public gist**
6. Copy the URL — this is your first shareable cloud portfolio artifact

### Gist Description Used
"AWS Regions reference sheet built during Week 1 of the AWS Cloud Accelerator course. Includes all regions, AZ counts, and Nigerian market latency notes. Part of my journey to SAA-C03 certification."

### What I Observed
Publishing publicly adds a layer of accountability — it's no longer just a local file. The Gist URL is permanent and shareable in a LinkedIn post, CV, or GitHub profile.

### What I Learned
- GitHub Gists are underused as portfolio tools. A well-formatted, clearly titled Gist signals organisation and documentation habits to any engineer reviewing your work.
- The act of formatting notes for public consumption forces you to be more precise and complete than private notes.

---

## Bonus: Latency Test Results

Using [cloudpingtest.com](https://cloudpingtest.com) from Lagos:

| Region | Code | Latency from Lagos |
|---|---|---|
| Africa (Cape Town) | af-south-1 | ~85ms |
| EU (Ireland) | eu-west-1 | ~140ms |
| EU (Frankfurt) | eu-central-1 | ~155ms |
| US East (N. Virginia) | us-east-1 | ~185ms |
| US West (Oregon) | us-west-2 | ~220ms |
| Asia Pacific (Singapore) | ap-southeast-1 | ~210ms |

**Conclusion:** af-south-1 is the clear winner for Nigerian users. For global audiences, a multi-region deployment with CloudFront in front reduces effective latency regardless of origin region.
