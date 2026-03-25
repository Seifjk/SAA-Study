# Compute (EC2)

### **SECTION 1: INSTANCE TYPES (THE FLEET)**

*The Hardware Options.*

### **1. Instance Types (The Naming Convention)**

- **Pattern:** `m5.2xlarge`
    - `m` = Instance Family (General Purpose)
    - `5` = Generation (Higher = Newer)
    - `2xlarge` = Size (nano, micro, small, medium, large, xlarge, 2xlarge, etc.)

**Instance Families (Memorize the Use Cases)**

| **Family** | **Name** | **Use Case** | **Exam Trigger** |
| --- | --- | --- | --- |
| **T2/T3/T4g** | Burstable | Dev/Test, Low Traffic Web Servers | "Variable CPU", "Burst Credits", "Cost-effective" |
| **M5/M6** | General Purpose | Balanced workloads, App Servers | "Standard workload", "Balance compute/memory" |
| **C5/C6** | Compute Optimized | Batch processing, HPC, Gaming Servers | "High CPU", "Compute-intensive" |
| **R5/R6** | Memory Optimized | In-Memory DBs (Redis, SAP HANA) | "High memory", "Database", "Cache" |
| **X1/X2** | Extreme Memory | SAP HANA, Big Data | "TB of RAM", "Mission critical in-memory" |
| **I3/I4i** | Storage Optimized | NoSQL DBs, Data Warehousing, OLTP | "High IOPS", "Low latency storage" |
| **G4/G5** | GPU | Machine Learning Training, Graphics | "GPU", "ML Training", "3D rendering" |
| **P3/P4** | GPU Compute | Deep Learning, Scientific Computing | "Deep Learning", "Parallel processing" |
| **Inf1** | Inferentia | ML Inference (Cost-efficient) | "ML Inference", "Deploy trained models" |

**Deep Dive: T2/T3 "Burstable" (CPU Credits)**

- **Concept:** You earn CPU credits when idle. Spend them during burst.
- **Problem:** If credits run out, CPU is throttled.
- **Solution A:** Switch to T2/T3 **Unlimited** (Auto-pay for extra CPU).
- **Solution B:** Switch to M5/M6 (Steady performance).
- *Exam Trap:* "Web server works fine at night, slow during day." → T2 exhausted credits.

---

### **SECTION 2: PURCHASING OPTIONS (THE PRICING)**

*How You Pay.*

| **Model** | **Commitment** | **Savings** | **Exam Trigger** |
| --- | --- | --- | --- |
| **On-Demand** | None. Pay per second. | 0% | "Unpredictable workload", "Short-term", "Development" |
| **Reserved (RI)** | 1 Year or 3 Year contract | 40-75% | "Steady-state", "Database", "Predictable load" |
| **Convertible RI** | 1/3 Year. Can change instance family. | 30-66% | "Flexibility to change type" |
| **Savings Plan** | 1/3 Year. Commit to $/hour usage. | Similar to RI | "Flexibility across families/regions" |
| **Spot Instances** | Use spare capacity at market price. Set max price (default = On-Demand). Interrupted with 2-min notice if spot price exceeds max. | Up to 90% | "Batch jobs", "Fault-tolerant", "Big Data" |
| **Dedicated Hosts** | Physical server for you. | Expensive | "Per-socket licensing" (Oracle, Windows Server), "Compliance" |
| **Dedicated Instances** | Instances on hardware dedicated to you (but AWS controls placement). | Less expensive | "Regulatory isolation", "No per-socket needs" |

**Deep Dive: Spot Instances**

- **How It Works:** AWS has spare capacity at a fluctuating market price. You set a **max price** (default = On-Demand price). If spot price exceeds your max price, your instance is interrupted (2-min warning). There is no bidding.
- **Spot Fleet:** Launch a mix of Spot + On-Demand to maintain capacity.
- **Spot Block:** ~~Fully deprecated (removed Dec 2022). Not a valid exam answer.~~
- **Interruption Handling:** Use **Spot Instance Interruption Notices** (2-minute warning) to gracefully save state.
- *Exam Trap:* "Cheapest for non-critical batch jobs" → Spot.

### **3. EC2 Capacity Reservations**

- **The Rule:** Reserves capacity for EC2 instances in a **specific AZ**. Guarantees you can launch instances when needed.
- **NOT a billing discount** — you pay On-Demand price whether you use the capacity or not.
- **Combine with Savings Plans or RIs** for both capacity guarantee AND billing discount.
- **No commitment required** — create/cancel anytime.
- *Exam Trigger:* "Guarantee capacity in a specific AZ" → Capacity Reservation.

---

### **SECTION 3: PLACEMENT GROUPS (THE TOPOLOGY)**

*Where Instances Live Relative to Each Other.*

| **Type** | **Behavior** | **Use Case** | **Exam Trigger** |
| --- | --- | --- | --- |
| **Cluster** | All instances packed in **same rack** (same AZ). | Low latency (10 Gbps network), HPC, Tightly coupled apps | "Lowest latency", "High throughput", "Big Data" |
| **Spread** | Each instance on **different hardware** (max 7 per AZ). | Critical apps where failure isolation is required | "Maximize availability", "Isolated failures" |
| **Partition** | Divides instances into **partitions** (racks). Each partition on different hardware. | Hadoop, Cassandra, Kafka (Distributed systems) | "Large distributed workloads", "Partition-aware" |

**Rules:**

- **Cluster:** Cannot span multiple AZs. Risk = If rack fails, all instances fail.
- **Spread:** Can span AZs. Limit = 7 instances per AZ per group.
- **Partition:** Can span AZs. Each partition is isolated (1 partition = 1 rack).

---

### **SECTION 4: NETWORKING (ELASTIC NETWORK INTERFACES)**

*The Network Cards.*

### **1. ENI (Elastic Network Interface)**

- **The Rule:** The standard virtual network card.
- **Features:**
    - Can attach/detach from instances.
    - Can have **multiple ENIs** on one instance (e.g., separate public/private traffic).
    - Use Case: Failover (Move ENI from failed instance to standby).
- *Exam Trap:* "How to preserve a private IP when moving workloads?" → Attach ENI to new instance.

### **2. ENA (Elastic Network Adapter)**

- **The Rule:** Enhanced Networking. **Up to 100 Gbps** bandwidth.
- **Mechanism:** SR-IOV (Hardware acceleration).
- **Exam Trigger:** "High throughput", "Low latency networking", "HPC".

### **3. EFA (Elastic Fabric Adapter)**

- **The Rule:** For **HPC** and **ML** workloads requiring **OS-Bypass** (Bypasses kernel for ultra-low latency).
- **Protocols:** Supports MPI (Message Passing Interface).
- *Exam Trigger:* "Tightly coupled HPC", "Sub-millisecond latency between nodes", "ML distributed training".

### **4. Elastic IP**

- **The Rule:** Static public IPv4 address that you own until you release it.
- **Limit:** 5 per region (can request increase).
- **Charged** when **not** associated with a running instance (AWS discourages wasting IPv4 addresses).
- **Survives stop/start** — unlike the default public IP which changes on stop/start.
- *Exam Trigger:* "Static public IP that survives stop/start" → Elastic IP.

---

### **SECTION 5: STORAGE ATTACHED (AMIS & SNAPSHOTS)**

### **1. AMI (Amazon Machine Image)**

- **The Rule:** The "Template" for an EC2 instance. Contains OS + Software.
- **Scope:** **Regional**. (You must copy to another region to use there).
- **Types:**
    - **AWS-Provided:** Amazon Linux, Ubuntu, Windows, etc.
    - **Marketplace:** Pre-packaged (e.g., NGINX, WordPress).
    - **Custom:** Your own. Built from existing EC2.
- **Create AMI Process:**
    - Stop instance (Recommended for consistency) → Create Image → AMI stored (with snapshot of root EBS).

**Deep Dive: Sharing AMIs**

- Can share AMIs across AWS accounts.
- Cannot share AMIs with **encrypted snapshots** unless you share the KMS key too.

### **2. Instance Metadata**

- **The Rule:** Access instance info from *inside* the instance.
- **URL:** `http://169.254.169.254/latest/meta-data/`
- **Data Includes:** Instance ID, Public IP, IAM Role credentials, User Data.
- *Exam Trap:* "How does an instance retrieve its IAM role credentials?" → Metadata service.

---

### **SECTION 6: EC2 LIFECYCLE (START, STOP, HIBERNATE)**

| **Action** | **EBS Root** | **Instance Store** | **RAM** | **Billing** |
| --- | --- | --- | --- | --- |
| **Stop** | Preserved | **Lost** | Lost | Stop paying for instance (Still pay for EBS) |
| **Terminate** | Deleted (unless DeleteOnTermination=false) | Lost | Lost | Stop paying everything |
| **Reboot** | Preserved | Preserved | Preserved | Continue paying |
| **Hibernate** | Preserved (RAM dumped to EBS) | Lost | **Saved to Disk** | Stop paying for instance |

**Hibernate Deep Dive:**

- **The Rule:** Saves RAM to root EBS, then stops instance. On start, RAM is restored.
- **Use Case:** Long-running processes, Pre-warmed caches.
- **Limits:** Max 60 days hibernate, Max 150 GB RAM, Root volume must be encrypted EBS.
- *Exam Trigger:* "Resume application with all in-memory state intact" → Hibernate.

---

### **SECTION 7: USER DATA & INSTANCE PROFILES**

### **1. User Data**

- **The Rule:** Bash script that runs **ONCE** at instance **FIRST BOOT**.
- **Use Case:** Install software, Configure settings, Pull code from S3.
- **Access:** Available via Metadata service.
- *Exam Trap:* "Automatically configure instance at launch" → User Data.

### **2. IAM Instance Profile**

- **The Rule:** Attaches an **IAM Role** to an EC2 instance.
- **Credentials:** Auto-rotated by AWS. Retrieved via Metadata service.
- *Exam Trap:* "EC2 needs to access S3 without storing credentials" → Attach IAM Role via Instance Profile.

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **Lowest latency between instances?** → Cluster Placement Group.
2. **Maximize fault isolation?** → Spread Placement Group.
3. **Cheapest option for fault-tolerant batch jobs?** → Spot Instances.
4. **T2 instance slow during business hours?** → CPU Credits exhausted. Use Unlimited or switch to M5.
5. **Per-socket software licensing?** → Dedicated Hosts.
6. **HPC with sub-millisecond latency?** → EFA (Elastic Fabric Adapter).
7. **Preserve private IP during failover?** → Attach ENI to new instance.
8. **Resume app with RAM intact?** → Hibernate.
9. **Auto-install software at launch?** → User Data script.
10. **EC2 needs S3 access without hardcoded keys?** → IAM Instance Profile.
11. **Guarantee capacity in a specific AZ?** → Capacity Reservation (not a discount — combine with Savings Plans/RIs).
12. **Static public IP that survives stop/start?** → Elastic IP (charged when unassociated).

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Burst Exhaustion" (T2 Credits)**

**The Situation:** Your development team is running a web application on a `t2.medium` instance. During business hours (9 AM - 5 PM), users report that the application becomes extremely slow. At night and on weekends, the application performs normally. CloudWatch shows CPU at 100% during business hours but low at night.

**The Options:**

A. Increase the EBS volume size.

B. Switch to a `t2.medium` with Unlimited mode enabled.

C. Add more Security Group rules to allow traffic.

D. Enable Auto Scaling.

**The Logic:**

- **Trap:** EBS (Option A) is not the bottleneck. The symptom is CPU throttling.
- **The Fix:** **Option B**. T2/T3 instances earn CPU credits when idle. During business hours, the app exhausts credits and gets throttled. **T2 Unlimited** allows paying for extra CPU beyond credits. Alternative: Switch to M5 for steady performance.

---

### **Scenario 2: The "Batch Job" (Spot Instances)**

**The Situation:** A company needs to process large video rendering jobs overnight. The jobs are **fault-tolerant** (can restart from checkpoints) and not time-sensitive. The current On-Demand fleet costs $5,000/month. The CFO wants to reduce costs by at least 70%.

**The Options:**

A. Purchase 3-year Reserved Instances.

B. Use Spot Instances with Spot Fleet.

C. Switch to Lambda for processing.

D. Use Dedicated Hosts.

**The Logic:**

- **Trap:** Reserved Instances (Option A) save 40-75%, but don't hit 90% savings.
- **Trap:** Lambda (Option C) has a 15-minute execution limit. Video rendering takes hours.
- **The Fix:** **Option B**. **Spot Instances** can save up to 90%. Since the workload is fault-tolerant and can handle interruptions, Spot is perfect. Use **Spot Fleet** to automatically request capacity across multiple instance types.

---

### **Scenario 3: The "HPC Cluster" (Placement Groups)**

**The Situation:** A financial modeling application requires **10 Gbps network bandwidth** between nodes for tightly-coupled parallel processing. The workload cannot tolerate network latency spikes. All instances must be in the **same Availability Zone** for the application to function correctly.

**The Options:**

A. Use a Spread Placement Group.

B. Use a Cluster Placement Group.

C. Use a Partition Placement Group.

D. Place instances in different AZs with VPC Peering.

**The Logic:**

- **Trap:** Spread (Option A) maximizes isolation but doesn't optimize for network performance.
- **The Fix:** **Option B**. **Cluster Placement Group** packs instances on the same rack in the same AZ. This provides **low-latency, high-throughput 10 Gbps networking** required for HPC workloads.

---

### **Scenario 4: The "License Server" (Dedicated Hosts)**

**The Situation:** Your company uses Oracle Database with per-socket licensing. Oracle requires you to track exactly how many physical CPU sockets are in use for licensing compliance. The database workload is steady and runs 24/7.

**The Options:**

A. Use On-Demand Instances.

B. Use Reserved Instances.

C. Use Dedicated Hosts.

D. Use Spot Instances.

**The Logic:**

- **Trap:** Reserved Instances (Option B) save money but don't give you **physical socket visibility** required for Oracle licensing.
- **The Fix:** **Option C**. **Dedicated Hosts** give you an entire physical server. You can see socket count, core count, and manage per-socket/per-core licensing. You can also use **License Manager** to track compliance.

---

### **Scenario 5: The "Failover IP" (ENI)**

**The Situation:** You have a critical application running on an EC2 instance with a **static private IP** that is hardcoded in many downstream systems. If the primary instance fails, you need to launch a standby instance and have it immediately take over the **same private IP** to avoid reconfiguring downstream systems.

**The Options:**

A. Use an Elastic IP and reassign it to the new instance.

B. Detach the ENI from the failed instance and attach it to the standby instance.

C. Use Route 53 to update DNS records.

D. Use an Application Load Balancer.

**The Logic:**

- **Trap:** Elastic IP (Option A) is for **public** IPs, not private IPs.
- **Trap:** Route 53 (Option C) requires DNS propagation time (minutes).
- **The Fix:** **Option B**. **ENI (Elastic Network Interface)** can be detached from one instance and attached to another. The **private IP, MAC address, and Security Groups** move with the ENI, providing instant failover without reconfiguration.
