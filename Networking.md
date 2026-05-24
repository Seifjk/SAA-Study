# Networking

> 📊 **Visual reference:** [VPCDiagrams.md](./VPCDiagrams.md) — 14 Mermaid diagrams covering VPC anatomy, NAT flow, SG vs NACL, endpoints, PrivateLink, peering, TGW, hybrid connectivity, and a full 3-tier reference architecture.

### **SECTION 1: THE ANATOMY (CIDR, SUBNETS, ROUTES)**

*The Skeleton of your network.*

### **1. VPC & CIDR (The Block)**

- **The Rule:** A VPC is a logical datacenter in **ONE Region**.
- **CIDR:** The IP range (e.g., `10.0.0.0/16`).
    - Min size: `/28` (16 IPs).
    - Max size: `/16` (65,536 IPs).
    - **Secondary CIDRs:** Up to **5 CIDR blocks** per VPC (1 primary + 4 secondary) for expansion.
- **Reserved IPs:** AWS reserves **5 IPs** in *every* subnet. (You cannot use them).
    - `.0` (Network), `.1` (Router), `.2` (DNS), `.3` (Future), `.255` (Broadcast).
    - *Exam Trap:* "You create a /28 subnet. How many usable IPs?" -> Answer: 16 - 5 = **11**.

### **2. Subnets (The Zones)**

- **The Rule:** A Subnet is locked to **ONE Availability Zone (AZ)**.
- **Public Subnet:** Has a Route Table entry pointing to an **Internet Gateway (IGW)**.
- **Private Subnet:** Does **NOT** have a route to the IGW.
- **Exam Trap:** "An instance has a Public IP but cannot reach the internet." -> Check if the Subnet's **Route Table** points to the IGW. (Just having an IP isn't enough; you need the route).

### **3. Route Tables**

- **Local Route:** Every Route Table has a `Local` rule by default. It allows all subnets *inside* the VPC to talk to each other. You cannot delete this.
- **Custom Routes:** You add these to send traffic to IGW, NAT, or Peering connections.
- **Longest Prefix Match:** The most specific route wins. (e.g., `10.0.1.0/24` wins over `10.0.0.0/16`).

---

### **SECTION 2: SECURITY (THE FIREWALLS)**

*The most common exam topic. Memorize the difference.*

| **Feature** | **Security Group (SG)** | **Network ACL (NACL)** |
| --- | --- | --- |
| **Level** | **ENI Level** (attached to network interfaces) | **Subnet Level** |
| **State** | **Stateful** (Return traffic allowed auto) | **Stateless** (Must allow Inbound AND Outbound) |
| **Rules** | **Allow Only** (Cannot DENY specific IPs) | **Allow AND Deny** |
| **Order** | All rules evaluated (Order doesn't matter) | **Number order** (Lowest number wins) |
| **Default** | Deny All Inbound / Allow All Outbound | **Default NACL:** Allow All. **Custom NACL:** Deny All (must add rules) |
| **Exam Trigger** | "Timeout" | "Permission Denied" (usually) |
| **Use Case** | Standard application security | Blocking a specific malicious IP |
- **Ephemeral Ports:** NACLs are stateless — return traffic needs explicit rules. You must allow **ephemeral ports (1024-65535)** for outbound responses. SGs handle this automatically (stateful).
- **Exam Trap:** "You blocked an IP in the Security Group but they can still access." -> You *can't* block in SG. You must use **NACL**.
- **Exam Trap:** "Custom NACL created, instances can't communicate" → Custom NACLs deny all by default. You must add allow rules.

### **AWS Network Firewall**

- **Purpose:** Managed firewall for **VPC-level traffic inspection** — goes far beyond SGs/NACLs. Deep packet inspection, IDS/IPS, protocol filtering, domain-based filtering. Deployed in a dedicated **firewall subnet** with route tables directing traffic through it.
- **vs SGs/NACLs:** Those do simple IP/port rules and can't inspect packet contents; Network Firewall does full L3-L7 inspection with Suricata-compatible stateful rules.
- *Exam Trigger:* "Inspect traffic for malicious payloads", "IDS/IPS", "Filter outbound traffic by domain name", "Deep packet inspection at VPC level".

---

### **SECTION 3: CONNECTIVITY (GETTING OUT)**

*How instances talk to the world.*

### **1. Internet Gateway (IGW)**

- **The Rule:** One per VPC.
- **Purpose:** Allows **Public** Subnets to talk to the Internet.
- **Architecture:** Horizontally scaled, highly available (you don't manage it).

### **2. NAT (Network Address Translation)**

- **Purpose:** Allows **Private** instances to talk to the Internet (e.g., download updates) but **blocks** the Internet from initiating connections to them.
- **Option A: NAT Gateway (The Standard)**
    - **Managed:** AWS handles scaling. Auto-scales up to **45 Gbps** bandwidth.
    - **Location:** Must be in a **Public Subnet**.
    - **Cost:** Hourly charge + Data processing fee.
    - **Availability:** **Zonal**. (If AZ goes down, NAT GW goes down). *Architecture:* Create one NAT GW per AZ for HA.
- **Option B: NAT Instance (The Legacy)**
    - **Self-Managed:** It's just an EC2.
    - **Configuration:** Must disable **Source/Destination Check** on the instance.
    - **Performance:** Limited by EC2 size.
    - *Exam Trigger:* "Cheap/Legacy NAT solution" or "Disable Source/Destination Check".
    - **Not a Bastion Host.** NAT Instance = outbound internet for private instances. Bastion Host = inbound SSH jump box for admins. Different purpose.

### **3. Egress-Only Internet Gateway (IPv6)**

- **Purpose:** The IPv6 equivalent of a NAT Gateway. Allows **outbound** IPv6 traffic from private instances but **blocks** inbound connections from the internet.
- **Why it exists:** IPv6 addresses are all public (no "private range" like `10.x.x.x`). You need this to prevent the internet from initiating connections to your IPv6 instances.
- **Rule:** Only for **IPv6**. For IPv4 outbound, use NAT Gateway.
- *Exam Trigger:* "IPv6 instances need outbound internet access but must not be reachable from the internet."

### **4. VPC Endpoints (PrivateLink)**

- **Purpose:** Access AWS Services (S3, DynamoDB, CloudWatch) **privately** (without going over the public internet).
- **Type A: Gateway Endpoint**
    - **Services:** **S3** and **DynamoDB** ONLY.
    - **Mechanism:** Modifies the **Route Table**.
    - **Cost:** **Free**.
- **Type B: Interface Endpoint**
    - **Services:** Everything else (CloudWatch, SNS, SQS, etc.).
    - **Mechanism:** Creates an **ENI** (IP address) in your subnet. Uses DNS.
    - **Cost:** **$$$** (Hourly + Data).

### **5. AWS PrivateLink (Exposing Your Own Services)**

- **Purpose:** Expose a service **you built** to other VPCs (even other accounts) **privately**, without VPC Peering, IGW, NAT, or public IPs.
- **Architecture:** Provider VPC puts the service behind an **NLB** and creates a **VPC Endpoint Service**; Consumer VPC creates an **Interface Endpoint** (ENI) to connect.
- **vs Interface Endpoints:** Those connect to **AWS services**; PrivateLink Endpoint Services connect to **your own services** via NLB. Works across accounts and AWS Marketplace.
- *Exam Trigger:* "Expose service to hundreds of customer VPCs without peering", "NLB + PrivateLink", "Service provider/consumer model".

---

### **SECTION 4: CONNECTING NETWORKS (TOPOLOGY)**

### **1. VPC Peering**

- **Rule:** Connects two VPCs directly via private IP.
- **Transitive?** **NO.** (If A↔B and B↔C, A cannot talk to C).
- **CIDR Overlap:** **Not Allowed.** (VPCs cannot have matching IP ranges).
- **Region:** Can be same region or cross-region.

### **2. Transit Gateway (TGW)**

- **Rule:** The "Hub and Spoke" router.
- **Transitive?** **YES.** (Connects thousands of VPCs and On-Prem VPNs).
- **Architecture:** Solves the messy "Peering Mesh" problem.
- **TGW Peering:** Connect Transit Gateways across regions for cross-region connectivity.
- *Exam Trigger:* "Simplify network management", "Connect 100 VPCs", "Star Topology", "Cross-region hub-and-spoke".

### **3. Site-to-Site VPN**

- **Components:**
    - **VGW (Virtual Private Gateway):** On the AWS side (attached to VPC).
    - **CGW (Customer Gateway):** On the User side (Physical Router IP).
- **Speed:** Runs over Public Internet (encrypted). Max 1.25 Gbps per tunnel.
- **ECMP (Equal-Cost Multi-Path):** Spreads traffic across multiple VPN tunnels for higher throughput.
    - **Only works with Transit Gateway + VPN.** Does NOT work with standard VPN on a VGW (Virtual Private Gateway).
    - *Exam Trap:* "Increase VPN bandwidth" → ECMP via TGW, not VGW.

### **4. VPN CloudHub**

- **Purpose:** Connect **multiple branch offices** to each other through AWS via hub-and-spoke. Each branch has its own Site-to-Site VPN to the **same VGW**, which routes branch-to-branch traffic — no direct branch links needed. Low-cost, uses VPN over public internet.
- *Exam Trigger:* "Connect multiple branch offices to each other through AWS", "Hub-and-spoke VPN for remote offices" → VPN CloudHub.

### **5. Direct Connect (DX)**

- **Rule:** Dedicated physical fiber. **No Public Internet.**
- **Speed:** 1 Gbps or 10 Gbps (or 100 Gbps).
- **Reliability:** High.
- **Time:** Takes weeks/months to setup.
- **Backup:** Often use VPN as a backup for Direct Connect.

### **6. DX + VPN (Encryption over Direct Connect)**

- **The Problem:** Direct Connect is private but **NOT encrypted** — data travels in cleartext over the fiber.
- **The Fix:** Run a **Site-to-Site VPN over the Direct Connect** link, adding IPSec encryption. DX provides the reliable pipe, VPN provides encryption.
- *Exam Trigger:* "Encrypted connection with consistent latency", "Encrypt Direct Connect traffic", "Compliance requires encryption in transit over dedicated link".
- *Exam Trap:* "Direct Connect is private, so it's encrypted." → **Wrong.** Private ≠ Encrypted.

### **7. Direct Connect Resiliency Patterns**

- **High Resiliency:** **2 DX connections at 2 different DX locations.** (One connection per location). Survives a single location failure.
- **Maximum Resiliency:** **2 DX connections per location at 2 different DX locations** (4 connections total). Survives device failure AND location failure.
- **Cost vs. Resilience tradeoff:** Maximum resiliency costs 2x but is required for critical workloads.
- *Exam Trigger:* "Mission-critical workload needs highest DX resiliency" → Maximum resiliency (2 connections x 2 locations).

### **8. Client VPN**

- **Purpose:** Managed VPN for **end-user remote access** to AWS or on-prem resources.
- **How it works:** Users install OpenVPN client → connect to Client VPN endpoint → access VPC resources.
- **Key distinction:** Client VPN = **user-to-AWS** (remote workers). Site-to-Site VPN = **network-to-network** (office to AWS).
- *Exam Trigger:* "Remote employees need VPN access to private resources" → Client VPN.

### **9. Route 53 Resolver (Hybrid DNS)**

- **Purpose:** Hybrid DNS resolution between on-premises and AWS. **Inbound Endpoint:** on-prem DNS forwards queries to AWS (resolves private hosted zones). **Outbound Endpoint:** AWS forwards queries to on-prem DNS (resolves on-prem domains). **Resolver Rules** define which domains forward where (e.g., `corp.internal` → on-prem DNS IPs).
- *Exam Trigger:* "Hybrid DNS resolution", "On-prem resolves AWS private hosted zones", "AWS resolves on-prem domains" → Route 53 Resolver.

### **10. VPC Sharing with RAM**

- **Purpose:** Share subnets across AWS accounts within an Organization via **Resource Access Manager (RAM)** — owner shares a subnet, participant accounts launch resources into it. Reduces VPC sprawl, centralizes network management, no peering/TGW needed for same-VPC communication.
- *Exam Trigger:* "Multiple accounts need resources in the same VPC" → VPC Sharing via RAM.

---

### **SECTION 5: MONITORING (THE EYES)**

### **VPC Flow Logs**

- **Purpose:** Captures metadata about IP traffic (Source IP, Dest IP, Port, Action: ALLOW/DENY).
- **Limitations:** Does **NOT** capture packet content (payload). Just the address label.
- **Storage:** Sends data to **S3** or **CloudWatch Logs**.
- *Exam Trigger:* "Debug why traffic is blocked", "Audit network traffic".

---

### **SECTION 6: BASTION HOST vs. SESSION MANAGER**

| **Feature** | **Bastion Host** | **Session Manager (SSM)** |
| --- | --- | --- |
| **What** | EC2 jump box in Public Subnet | AWS Systems Manager feature |
| **SSH / Port 22** | Required (open inbound port 22) | **Not needed** (no open ports) |
| **Infrastructure** | You manage/patch the EC2 instance | No infrastructure to manage |
| **Access Control** | Security Group (IP-based) | **IAM policies** (identity-based) |
| **Logging** | Manual setup | Built-in: logs to **CloudWatch Logs / S3** |
| **Cost** | EC2 instance cost | Free (EC2 needs SSM Agent + IAM role) |

- *Exam Trigger:* "Access private instances without opening SSH" → Session Manager. "Most secure way to access private instances" → Session Manager. "Audit all shell commands to private instances" → Session Manager (CloudWatch/S3 logging).
- *Exam Trap:* "Bastion Host is the most secure way to access private instances." → **Wrong.** Session Manager eliminates SSH, open ports, and key management entirely.

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **Block a specific IP?** → NACL (Security Groups can't deny).
2. **Access S3 Privately (Free)?** → Gateway Endpoint.
3. **Access Kinesis/SQS/CloudWatch Privately?** → Interface Endpoint.
4. **Connect 50 VPCs together?** → Transit Gateway.
5. **Connect 2 VPCs?** → VPC Peering (Check for no CIDR overlap).
6. **Private instances need internet (outbound only)?** → NAT Gateway (in Public Subnet).
7. **Instance has Public IP but no internet?** → Check Route Table for IGW.
8. **Bastion Host?** → Public Subnet EC2 for SSH jump access. SG: Allow SSH from *your IP only*.
9. **NAT Instance?** → Legacy/cheap NAT. Disable Source/Destination Check. Not the same as Bastion.
10. **VPC Peering not working?** → Check Route Tables on *both* sides and Security Groups.
11. **Expose your service to other VPCs privately?** → NLB + PrivateLink (Endpoint Service).
12. **Encrypt Direct Connect?** → Run Site-to-Site VPN over the DX connection.
13. **Highest DX resiliency?** → Maximum resiliency: 2 connections x 2 locations (4 total).
14. **IPv6 outbound only (no inbound)?** → Egress-Only Internet Gateway.
15. **Deep packet inspection / IDS/IPS at VPC level?** → AWS Network Firewall.
16. **Increase VPN bandwidth?** → ECMP via Transit Gateway (not VGW).
17. **Hybrid DNS resolution (on-prem ↔ AWS)?** → Route 53 Resolver (Inbound/Outbound Endpoints).
18. **Share subnets across AWS accounts?** → RAM (Resource Access Manager).
19. **Remote user VPN access to AWS?** → Client VPN.
20. **Custom NACL blocks everything?** → Custom NACLs deny all by default. Add allow rules + ephemeral ports (1024-65535).
17. **Connect multiple branch offices to each other through AWS?** → VPN CloudHub (hub-and-spoke via VGW).
18. **Access private instances without SSH / most secure access?** → Session Manager (no ports, IAM-based, logged).

This covers the network logic. If you can draw the packet flow from a private instance -> NAT GW -> IGW -> Internet, you pass.



# **REAL EXAM SCENARIOS**

### **Scenario 1**

**The Situation:** Your security team has detected a Denial of Service attack coming from a specific IP address `203.0.113.5`. You need to **immediately block** all traffic from this IP.

**The Options:**

A. Add a Deny rule to the Security Group attached to the web servers.

B. Remove the Internet Gateway from the VPC.

C. Use AWS WAF to inspect the traffic.

D. Add a Deny rule to the Network ACL (NACL) associated with the public subnet.

**The Logic:**

- **Trap A — Security Group:** SGs are **allow-only** — there is no such thing as a Deny rule in an SG. You literally cannot do this.
- **Trap B — Remove the IGW:** This blocks the *entire* VPC from the internet, taking down every instance for every user. Catastrophic overreach to stop one IP.
- **Trap C — WAF:** WAF *can* block an IP, but it only protects Layer 7 resources (CloudFront, ALB, API Gateway). For a raw L3/L4 DoS hitting the subnet, and for an *immediate* block, NACL is the direct tool. WAF is also the slower thing to stand up if not already deployed.
- **The Fix — Option D:** A **NACL** operates at the subnet level and supports explicit **Deny** rules. Add a Deny for `203.0.113.5` — it's the only option that surgically blocks one IP immediately.

### **Scenario 2**

**The Situation:** You have a fleet of EC2 instances in a **Private Subnet**. They need to download security patches from the internet, but they must **not** be accessible from the public internet.

**The Options:**

A. Attach an Internet Gateway to the Private Subnet.

B. Create a NAT Gateway in the Public Subnet and update the Private Subnet's route table.

C. Create a NAT Gateway in the Private Subnet and update the route table.

D. Use a VPC Endpoint for S3.

**The Logic:**

- **Trap A — IGW on the private subnet:** Attaching an IGW route makes the subnet *public* — instances would become reachable from the internet, violating the explicit "must not be accessible" requirement.
- **Trap C — NAT Gateway in the private subnet:** A NAT GW needs its *own* route to an IGW to function. Placed in a private subnet it has no internet path — it doesn't work.
- **Trap D — VPC Endpoint for S3:** A Gateway Endpoint only reaches **S3** (and DynamoDB). Security patches come from OS/vendor repos across the general internet — an S3 endpoint can't fetch them.
- **The Fix — Option B:** Put the **NAT Gateway in a public subnet**, then point the private subnet's route table at it. Flow: Private instance → NAT GW (public subnet) → IGW → Internet. Outbound works; inbound stays blocked.

### **Scenario 3**

**The Situation:** An application in a private subnet needs to transfer sensitive data to an S3 bucket. Corporate policy states that this data **must not traverse the public internet**, even if encrypted.

**The Options:**

A. Use a VPC Gateway Endpoint for S3.

B. Use a NAT Gateway.

C. Use a VPC Peering connection to S3.

D. Enable S3 Transfer Acceleration.

**The Logic:**

- **Trap B — NAT Gateway:** A NAT GW routes traffic out through the **IGW to the public internet**. The destination is the public S3 endpoint — the data traverses the internet (masked, but still public). Violates the policy.
- **Trap C — VPC Peering to S3:** You cannot peer with S3. VPC Peering connects two **VPCs**; S3 is not a VPC. Not a real option.
- **Trap D — Transfer Acceleration:** This *speeds up* uploads by routing through edge locations over the **public internet** — it's a performance feature and makes the policy violation worse, not better.
- **The Fix — Option A:** A **Gateway Endpoint for S3** adds a route so traffic to S3 stays entirely on the **AWS private network** — it never touches the public internet. It's also free.

### **Scenario 4**

**The Situation:** Your company has acquired 3 smaller startups. You now have **50 different VPCs** across different accounts that all need to communicate with each other and with an on-premise data center. Managing individual peering connections is becoming impossible.

**The Options:**

A. Create a full mesh of VPC Peering connections (A-B, A-C, B-C, etc.).

B. Use AWS Direct Connect Gateway.

C. Use a Shared VPC.

D. Implement an AWS Transit Gateway.

**The Logic:**

- **Trap A — Full mesh of VPC Peering:** 50 VPCs would need N×(N−1)/2 = **1,225 peering connections**, and peering is non-transitive so it can't simplify. Exactly the "impossible to manage" problem stated.
- **Trap B — Direct Connect Gateway:** DX Gateway connects **on-premises to VPCs** over a dedicated line. It doesn't solve VPC-to-VPC connectivity between the 50 VPCs — only part of the requirement.
- **Trap C — Shared VPC:** RAM-shared VPCs let multiple accounts deploy *into one VPC*. It doesn't interconnect 50 *existing separate* VPCs — you'd have to re-architect everything.
- **The Fix — Option D:** A **Transit Gateway** is a central hub-and-spoke router. Attach all 50 VPCs (across accounts) and the on-prem VPN to one TGW — it's transitive, so everything can reach everything with a single managed component.

### **Scenario 5: The "Broken Internet"**

**The Situation:** You launched an EC2 instance in a Public Subnet with a Public IP address. However, you cannot SSH into it from your home laptop. The Security Group allows Port 22 from `0.0.0.0/0`. What is the most likely cause?

**The Options:**

A. The Network ACL is blocking outbound traffic on ephemeral ports.

B. The Subnet's Route Table is missing a route to `0.0.0.0/0` targeting the Internet Gateway.

C. The Internet Gateway is missing from the VPC.

D. The Instance does not have an IAM Role attached.

**The Logic:**

- **Trap A — NACL blocking outbound ephemeral ports:** This *would* break the SSH reply (NACLs are stateless). But it's the less common misconfiguration, and the question asks for the **most likely** cause — the route table is the classic culprit. Plausible distractor, not the best answer.
- **Trap C — Missing IGW:** Possible, but if the subnet is already set up as "Public" an IGW usually exists. The route entry is the far more common error.
- **Trap D — No IAM Role:** IAM Roles grant the instance permissions to call **AWS APIs**. They have nothing to do with inbound SSH connectivity. Irrelevant.
- **The Fix — Option B:** A public IP + an IGW are not enough — the **subnet's route table** must have a `0.0.0.0/0 → igw-xxxx` entry. Without that route, packets have no path out. This is the textbook "public IP but no connectivity" cause.

---

