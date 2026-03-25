# Networking

### **SECTION 1: THE ANATOMY (CIDR, SUBNETS, ROUTES)**

*The Skeleton of your network.*

### **1. VPC & CIDR (The Block)**

- **The Rule:** A VPC is a logical datacenter in **ONE Region**.
- **CIDR:** The IP range (e.g., `10.0.0.0/16`).
    - Min size: `/28` (16 IPs).
    - Max size: `/16` (65,536 IPs).
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
| **Level** | **Instance Level** (Network Card/ENI) | **Subnet Level** |
| **State** | **Stateful** (Return traffic allowed auto) | **Stateless** (Must allow Inbound AND Outbound) |
| **Rules** | **Allow Only** (Cannot DENY specific IPs) | **Allow AND Deny** |
| **Order** | All rules evaluated (Order doesn't matter) | **Number order** (Lowest number wins) |
| **Default** | Deny All Inbound / Allow All Outbound | Allow All Inbound / Allow All Outbound |
| **Exam Trigger** | "Timeout" | "Permission Denied" (usually) |
| **Use Case** | Standard application security | Blocking a specific malicious IP |
- **Exam Trap:** "You blocked an IP in the Security Group but they can still access." -> You *can't* block in SG. You must use **NACL**.

### **AWS Network Firewall**

- **Purpose:** Managed firewall service for **VPC-level traffic inspection**. Goes far beyond what SGs and NACLs can do.
- **Capabilities:** Deep packet inspection, intrusion detection/prevention (IDS/IPS), protocol filtering, domain-based filtering (block traffic to specific domains).
- **Placement:** Deployed in a dedicated **firewall subnet**. Route tables direct traffic through it.
- **Key Distinction:**
    - **SGs/NACLs:** Simple IP/port rules. Cannot inspect packet contents.
    - **Network Firewall:** Full L3-L7 inspection, Suricata-compatible rules, stateful filtering.
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
    - **Managed:** AWS handles scaling.
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

- **Purpose:** Expose a service **you built** in your VPC to other VPCs (even in other accounts) **privately**, without VPC Peering, IGW, NAT, or public IPs.
- **Architecture:**
    - **Provider VPC:** Put your service behind a **Network Load Balancer (NLB)**, then create a **VPC Endpoint Service**.
    - **Consumer VPC:** Creates an **Interface Endpoint** (ENI) that connects to your service.
- **Key Distinction:** Interface Endpoints (above) connect to **AWS services**. PrivateLink Endpoint Services connect to **your own services** via NLB.
- **Scalability:** Works across accounts and even with AWS Marketplace services.
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
- *Exam Trigger:* "Simplify network management", "Connect 100 VPCs", "Star Topology".

### **3. Site-to-Site VPN**

- **Components:**
    - **VGW (Virtual Private Gateway):** On the AWS side (attached to VPC).
    - **CGW (Customer Gateway):** On the User side (Physical Router IP).
- **Speed:** Runs over Public Internet (encrypted). Max 1.25 Gbps per tunnel.
- **ECMP (Equal-Cost Multi-Path):** Spreads traffic across multiple VPN tunnels for higher throughput.
    - **Only works with Transit Gateway + VPN.** Does NOT work with standard VPN on a VGW (Virtual Private Gateway).
    - *Exam Trap:* "Increase VPN bandwidth" → ECMP via TGW, not VGW.

### **4. VPN CloudHub**

- **Purpose:** Connect **multiple branch offices** to each other through AWS using a hub-and-spoke model.
- **How It Works:** Each branch has its own Site-to-Site VPN connection to the **same VGW**. The VGW routes traffic between branches.
- **Cost:** Low-cost. Uses existing VPN connections over the public internet.
- **Key Point:** Branch-to-branch traffic flows through the VGW (hub). No direct branch-to-branch link needed.
- *Exam Trigger:* "Connect multiple branch offices to each other through AWS", "Hub-and-spoke VPN for remote offices" → VPN CloudHub.

### **5. Direct Connect (DX)**

- **Rule:** Dedicated physical fiber. **No Public Internet.**
- **Speed:** 1 Gbps or 10 Gbps (or 100 Gbps).
- **Reliability:** High.
- **Time:** Takes weeks/months to setup.
- **Backup:** Often use VPN as a backup for Direct Connect.

### **6. DX + VPN (Encryption over Direct Connect)**

- **The Problem:** Direct Connect is a private connection, but it is **NOT encrypted** by default. Data travels in cleartext over the dedicated fiber.
- **The Fix:** Run a **Site-to-Site VPN connection over the Direct Connect** link. This adds IPSec encryption on top of the dedicated connection.
- **Architecture:** DX provides the reliable pipe, VPN provides the encryption. Best of both worlds.
- *Exam Trigger:* "Encrypted connection with consistent latency", "Encrypt Direct Connect traffic", "Compliance requires encryption in transit over dedicated link".
- *Exam Trap:* "Direct Connect is private, so it's encrypted." → **Wrong.** Private ≠ Encrypted.

### **7. Direct Connect Resiliency Patterns**

- **High Resiliency:** **2 DX connections at 2 different DX locations.** (One connection per location). Survives a single location failure.
- **Maximum Resiliency:** **2 DX connections per location at 2 different DX locations** (4 connections total). Survives device failure AND location failure.
- **Cost vs. Resilience tradeoff:** Maximum resiliency costs 2x but is required for critical workloads.
- *Exam Trigger:* "Mission-critical workload needs highest DX resiliency" → Maximum resiliency (2 connections x 2 locations).

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
17. **Connect multiple branch offices to each other through AWS?** → VPN CloudHub (hub-and-spoke via VGW).
18. **Access private instances without SSH / most secure access?** → Session Manager (no ports, IAM-based, logged).

This covers the network logic. If you can draw the packet flow from a private instance -> NAT GW -> IGW -> Internet, you pass.



### **3. SOLVE: VPC Scenarios (The "Plumbing" Tests)**

### **Scenario 1: The "Bad Actor" (NACL vs. SG)**

**The Situation:** Your security team has detected a Denial of Service attack coming from a specific IP address `203.0.113.5`. You need to **immediately block** all traffic from this IP.

**The Options:**

A. Add a Deny rule to the Security Group attached to the web servers.

B. Add a Deny rule to the Network ACL (NACL) associated with the public subnet.

C. Remove the Internet Gateway from the VPC.

D. Use AWS WAF to inspect the traffic.

**The Logic:**

- **Trap:** Security Groups (Option A) **cannot** have DENY rules. They are "Allow Only."
- **The Fix:** **Option B**. **NACLs** are the only place at the subnet level where you can explicitly DENY a specific IP.

### **Scenario 2: The "Private Update" (NAT Gateway)**

**The Situation:** You have a fleet of EC2 instances in a **Private Subnet**. They need to download security patches from the internet, but they must **not** be accessible from the public internet.

**The Options:**

A. Attach an Internet Gateway to the Private Subnet.

B. Create a NAT Gateway in the Private Subnet and update the route table.

C. Create a NAT Gateway in the Public Subnet and update the Private Subnet's route table.

D. Use a VPC Endpoint for S3.

**The Logic:**

- **Trap:** Putting the NAT Gateway in the Private Subnet (Option B). A NAT Gateway needs its *own* route to the internet to work. It must live in the **Public Subnet**.
- **The Fix:** **Option C**. Private Instance -> NAT GW (in Public) -> IGW -> Internet.

### **Scenario 3: The "S3 Security" (Gateway Endpoint)**

**The Situation:** An application in a private subnet needs to transfer sensitive data to an S3 bucket. Corporate policy states that this data **must not traverse the public internet**, even if encrypted.

**The Options:**

A. Use a NAT Gateway.

B. Use a VPC Gateway Endpoint for S3.

C. Use a VPC Peering connection to S3.

D. Enable S3 Transfer Acceleration.

**The Logic:**

- **Trap:** NAT Gateway (Option A) sends traffic over the internet (even though it's masked).
- **The Fix:** **Option B**. **Gateway Endpoints** keep traffic entirely within the AWS internal network. It never touches the public internet.

### **Scenario 4: The "Star Topology" (Transit Gateway)**

**The Situation:** Your company has acquired 3 smaller startups. You now have **50 different VPCs** across different accounts that all need to communicate with each other and with an on-premise data center. Managing individual peering connections is becoming impossible.

**The Options:**

A. Create a full mesh of VPC Peering connections (A-B, A-C, B-C, etc.).

B. Use AWS Direct Connect Gateway.

C. Implement an AWS Transit Gateway.

D. Use a Shared VPC.

**The Logic:**

- **Trap:** VPC Peering (Option A) becomes a management nightmare (N*(N-1)/2 connections).
- **The Fix:** **Option C**. **Transit Gateway** acts as a central hub/router. You connect all 50 VPCs to the TGW once.

### **Scenario 5: The "Broken Internet" (Route Tables)**

**The Situation:** You launched an EC2 instance in a Public Subnet with a Public IP address. However, you cannot SSH into it from your home laptop. The Security Group allows Port 22 from `0.0.0.0/0`. What is the most likely cause?

**The Options:**

A. The Network ACL is blocking outbound traffic on ephemeral ports.

B. The Instance does not have an IAM Role attached.

C. The Internet Gateway is missing from the VPC.

D. The Subnet's Route Table is missing a route to `0.0.0.0/0` targeting the Internet Gateway.

**The Logic:**

- **Trap:** Missing IGW (Option C) is unlikely if the subnet is already designated "Public" (usually implies IGW exists), but the *Route* is the common configuration error.
- **The Fix:** **Option D**. Having an IGW isn't enough; the **Route Table** must explicitly say "Send internet traffic (`0.0.0.0/0`) to `igw-xxxx`".

---

### **4. SYNTHESIZE: How to Use This**

**Clear Answer:**

These 5 scenarios map directly to the core networking rules above.

- **Scenario 1 (Bad Actor)** proves why you memorize "Security Groups = Allow Only" → use NACL to deny.
- **Scenario 2 (Private Update)** proves the NAT Gateway placement rule → Public Subnet only.
- **Scenario 3 (S3 Security)** proves Gateway Endpoints keep traffic off the public internet.
- **Scenario 4 (Star Topology)** proves Transit Gateway solves the peering mesh problem.
- **Scenario 5 (Broken Internet)** proves that a Public IP alone isn't enough → Route Table must point to IGW.

**Next Step:** Read one scenario, cover the answer, and explain *why* the other three are wrong. That is how you pass.