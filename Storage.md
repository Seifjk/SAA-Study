# Solution Architect

# Storage

### **SECTION 1: BLOCK STORAGE (EBS & INSTANCE STORE)**

*The "Hard Drive" attached to your EC2.*

### **1. Instance Store (Ephemeral Storage)**

- **The Hardware:** Physically attached SSD/HDD on the host server.
- **The Rule:** **Ephemeral = Temporary.**
    - If you **STOP** the instance → Data is **LOST** (Disk is wiped).
    - If you **REBOOT** the instance → Data **SURVIVES**.
    - If hardware fails → Data is **LOST**.
- **Use Case:** Cache, buffers, temporary content, distributed data stores (Kafka, Cassandra) where the app handles replication.
- **Performance:** Highest possible IOPS (millions) because it’s local, not network-attached.
- **Exam Trap:** "An instance has high I/O needs and data can be lost. Which storage?" → **Instance Store**.

### **2. EBS (Elastic Block Store)**

- **The Hardware:** Network-attached drive. (Latency is slightly higher than Instance Store).
- **The Rule:** **Persistent.** Data survives an instance stop.
- **Scope:** Locked to **ONE Availability Zone (AZ)**.
    - *To move to another AZ:* Snapshot → Copy Snapshot to Region/AZ → Create Volume from Snapshot.

**EBS Volume Types (The IOPS/Throughput Game)**

| **Type** | **Name** | **Use Case** | **Performance / IOPS** | **Exam "Trigger"** |
| --- | --- | --- | --- | --- |
| **General Purpose** | **gp2** | Boot volumes, Dev/Test | **IOPS Linked to Size.** 3 IOPS per GB. (Max 16,000 IOPS). | "Balance price/performance" |
| **General Purpose** | **gp3** | **Default Choice.** | **Decoupled Performance.** You get 3,000 IOPS baseline free. You pay to increase IOPS without increasing size. | "Independent scaling of storage and IOPS" |
| **Provisioned IOPS** | **io1 / io2** | Mission Critical DBs | **High IOPS.** Up to 64,000 IOPS (both io1 and io2). | "Sub-millisecond latency", "Sustained IOPS" |
| **Block Express** | **io2 Block Express** | The "SAN" Killer | Up to **256,000 IOPS**. Sub-millisecond latency. Highest EBS performance. | "Mission critical", "SAP HANA", "256K IOPS" |
| **Throughput Opt HDD** | **st1** | Big Data, Logs, Kafka | Optimized for **Throughput (MB/s)**, not IOPS. Max 500 MB/s. | "Streaming", "Log processing", "Sequential I/O" |
| **Cold HDD** | **sc1** | Archive | Cheapest EBS. Infrequent access. | "Lowest cost block storage" |

**EBS Features:**

- **Multi-Attach:** Only available on **io1 / io2**. Allows *one* volume to attach to *multiple* EC2 instances in the *same* AZ. (Cluster-aware apps only).
- **Encryption:** Enabled by default or at creation.
    - *How to encrypt an unencrypted volume?* Snapshot (Unencrypted) → Copy Snapshot (Check "Encrypt" box) → Create Volume from Encrypted Snapshot.
- **Delete on Termination:**
    - **Root Volume:** Default = **Delete**. (OS disk vanishes when EC2 terminates).
    - **Data Volume:** Default = **Keep**. (Extra disk stays).
    - *Exam Trap:* "Preserve the root volume data" → Change attribute `DeleteOnTermination` to `False`.

**EBS Snapshots:**

- **Incremental:** Only changed blocks since the last snapshot are saved. First snapshot is full copy.
- **Stored in S3** internally (you cannot see them in your S3 console).
- **Cross-Region Copy:** Snapshot → Copy to another Region → Create Volume. This is how you migrate EBS across regions.
- **Data Lifecycle Manager (DLM):** Automate snapshot creation, retention, and deletion on a schedule. *Exam Trigger:* "Automate EBS backups" → DLM.
- **Fast Snapshot Restore (FSR):** Pre-warms the snapshot so volumes created from it have **no latency penalty** on first read. Costs money. *Exam Trigger:* "Eliminate latency on first access from snapshot" → FSR.
- *Exam Trap:* Snapshots are **not real-time backups**. For crash-consistent snapshots, stop I/O or detach the volume first.

---

### **SECTION 2: OBJECT STORAGE (S3)**

*The "Unlimited Bucket" for files.*

**1. Consistency Model**

- **Strong Consistency:** If you write a file and immediately read it, you get the *new* version. (No more "eventual consistency" delays).

**2. Storage Classes (Memorize the Waterfall)**

| **Class** | **Availability** | **Minimum Duration** | **Retrieval Fee?** | **Exam Trigger** |
| --- | --- | --- | --- | --- |
| **Standard** | 99.99% | None | No | "Frequently accessed", "Instant" |
| **Standard-IA** | 99.9% | 30 Days | Yes | "Disaster Recovery", "Once a month access" |
| **Intelligent-Tiering** | 99.9% | None | No | "Unknown/Changing access patterns" |
| **One Zone-IA** | **Risk of Data Loss** | 30 Days | Yes | "Recreatable data", "Secondary backup", "Cheapest instant access" |
| **Glacier Instant** | 99.9% | 90 Days | Yes | "Archive but need millisecond access" |
| **Glacier Flexible** | 99.9% | 90 Days | No (Standard) | "Bulk retrieval", "Wait 3-5 hours" |
| **Glacier Deep Archive** | 99.9% | 180 Days | No | "Compliance", "Retain for 7 years", "Wait 12 hours" |
| **Express One Zone** | 99.95% (Single AZ) | None | No | "Single-digit ms latency", "Frequently accessed in one AZ" |

**3. S3 Performance Optimization**

- **Multipart Upload:** Required for files > 5GB. Recommended for > 100MB. Uploads parts in parallel.
- **S3 Transfer Acceleration:** Uses AWS Edge Locations to speed up uploads over the public internet. (Cost money).
- **Byte-Range Fetches:** Download specific parts of a file. (Speeds up getting just the header of a video).

**4. Security & Encryption**

- **Bucket Policies:** JSON. **Global**. "Allow Public Access" is blocked by default at the account level.
- **Encryption Types:**
    - **SSE-S3:** AWS manages keys. (Default).
    - **SSE-KMS:** AWS manages keys, but you control permissions/rotation. **Impacts KMS Quotas** (Exam trap: "Uploads failing due to ThrottlingException" -> Check KMS limits).
    - **SSE-C:** Client (You) sends the key with every request. AWS does not store the key.

**5. S3 Versioning**

- **What it does:** Keeps every version of every object (including deletions).
- **Enable vs Suspend:** Once enabled, you can only **suspend** versioning (not disable). Existing versions are preserved when suspended.
- **Delete Behavior:**
    - **Simple DELETE** on a versioned bucket → places a **Delete Marker** (soft delete). The object appears gone but all versions still exist. You can recover by removing the delete marker.
    - **Permanent DELETE** requires specifying the exact **Version ID**.
- **MFA Delete:** Optional. Requires MFA to permanently delete versions or change versioning state. Can only be enabled by the **root account** via CLI (not console).
- **Required for Replication:** You cannot enable CRR or SRR without versioning on both source and destination buckets.
- *Exam Trap:* "Protect against accidental deletion" → Enable Versioning (+ optionally MFA Delete).

**6. S3 Lifecycle Policies**

- **Transition Rules:** Automatically move objects between storage classes over time.
    - Example: Standard → Standard-IA (after 30 days) → Glacier Flexible (after 90 days) → Deep Archive (after 180 days).
    - *Constraint:* Cannot transition from Standard-IA to One Zone-IA. The "waterfall" only goes down.
- **Expiration Rules:** Automatically delete objects or old versions after a set period.
    - Can also clean up **incomplete multipart uploads** (important cost-saving exam trap).
- **Minimum Storage Duration Charges:**
    - Standard-IA / One Zone-IA: **30 days** (delete early = still charged for 30 days).
    - Glacier Instant / Glacier Flexible: **90 days**.
    - Glacier Deep Archive: **180 days**.
- *Exam Trigger:* "Reduce storage costs over time" or "Archive old data automatically" → Lifecycle Policy.

**7. S3 Replication**

- **CRR (Cross-Region Replication):** Replicate objects to a bucket in a **different region**. Use cases: compliance, lower-latency access, DR.
- **SRR (Same-Region Replication):** Replicate within the **same region**. Use cases: log aggregation, live replication between prod/test.
- **Requirements:**
    - **Versioning must be enabled** on both source and destination buckets.
    - Proper **IAM role** for S3 to replicate on your behalf.
- **Key Rules:**
    - **Existing objects** are NOT replicated when you enable replication. Use **S3 Batch Replication** to copy existing objects.
    - **No chaining:** If Bucket A replicates to B, and B replicates to C, objects from A do **not** propagate to C.
    - **Delete markers** are NOT replicated by default (optional setting to enable).
    - Permanent deletes (by version ID) are **never** replicated (prevents malicious deletes).
- *Exam Trap:* "Replicate existing objects" → S3 Batch Replication (not just enabling replication rules).

**8. S3 Object Lock & Glacier Vault Lock**

- **S3 Object Lock:** WORM (Write Once, Read Many) compliance at the object level.
    - **Requires Versioning** to be enabled.
    - **Governance Mode:** Users with special IAM permissions (`s3:BypassGovernanceRetention`) can override/delete. Good for internal controls.
    - **Compliance Mode:** **Nobody** can delete or overwrite — not even the root account. The retention period cannot be shortened. Use for regulatory compliance.
    - **Retention Period:** Set a fixed time window (days/years) during which the object is protected.
    - **Legal Hold:** Indefinite protection until explicitly removed. Independent of retention period.
- **Glacier Vault Lock:** WORM compliance at the **vault level** using a Vault Lock Policy.
    - Once the policy is locked, it can **never** be changed or deleted.
    - *Exam Trigger:* "Regulatory compliance", "SEC Rule 17a-4", "Data cannot be deleted for 7 years" → Glacier Vault Lock or S3 Object Lock (Compliance Mode).

**9. S3 Pre-signed URLs**

- **What it does:** Generate a temporary URL that grants time-limited access to a **private** S3 object.
- The URL inherits the permissions of the IAM user/role that generated it.
- **Expiration:** Configurable (default 1 hour, max 7 days with IAM user credentials).
- **Use Cases:** Allow a user to download a private file without making the bucket public. Allow a user to upload to a specific key.
- *Exam Trigger:* "Temporary access to a private object", "Share a file without making it public", "Allow upload without AWS credentials" → Pre-signed URL.

**10. S3 Event Notifications**

- **Triggers:** S3 can emit events on `s3:ObjectCreated:*`, `s3:ObjectRemoved:*`, `s3:ObjectRestore:*`, etc.
- **Destinations:**
    - **SQS** — queue for async processing.
    - **SNS** — fan-out to multiple subscribers.
    - **Lambda** — serverless processing (e.g., thumbnail generation on upload).
    - **Amazon EventBridge** — **all** S3 events can be sent to EventBridge for advanced filtering, multiple destinations, and archive/replay.
- **EventBridge Advantage:** Supports 18+ AWS services as targets, advanced rules, and works with ALL event types. EventBridge is the modern, more flexible option.
- *Exam Trigger:* "Process files on upload" → S3 Event Notification to Lambda. "Fan out to multiple services" → EventBridge.

**11. S3 Static Website Hosting**

- S3 can host a static website (HTML, CSS, JS) directly from a bucket.
- **Endpoint format:** `http://<bucket-name>.s3-website-<region>.amazonaws.com`
- **Must enable public access** (Bucket Policy allowing `s3:GetObject` to `*`) OR use CloudFront with **OAC (Origin Access Control)** to keep the bucket private.
- *Exam Combo:* Static website on S3 + CloudFront (CDN) + OAC (restrict direct S3 access) + Route 53 (custom domain) + ACM (HTTPS). This is a very common architecture question.
- *Exam Trap:* S3 website endpoint does NOT support HTTPS natively. You need CloudFront in front for HTTPS.

---

### **SECTION 3: NETWORK FILE STORAGE (EFS vs FSx)**

*The "Shared Drive" for multiple servers.*

**1. EFS (Elastic File System)**

- **Protocol:** NFS v4.
- **OS:** **Linux Only**. (POSIX).
- **Scale:** Petabytes. Grow/Shrink automatically.
- **Scope:** **Multi-AZ** (Data is stored across multiple AZs).
- **Performance Modes:**
    - *General Purpose:* Default. Low latency. Web servers.
    - *Max I/O:* High throughput, higher latency. Big Data.
- **Throughput Modes:**
    - *Elastic:* **Default (recommended).** Auto-scales throughput up/down. Best for unpredictable workloads.
    - *Provisioned:* Pay for fixed speed (e.g., 500 MB/s) regardless of size.
    - *Bursting:* Throughput tied to size. (Legacy default, still available).

**2. FSx (File System for X)**

- **FSx for Windows:**
    - Protocol: **SMB**.
    - Integration: **Active Directory**.
    - Features: DFS (Distributed File System), Deduplication.
    - *Trigger:* "Migrate Windows File Server", "SharePoint", "SMB share".
- **FSx for Lustre:**
    - Protocol: Parallel Distributed.
    - Use Case: **HPC (High Performance Computing)**, Machine Learning, Video Rendering.
    - *Superpower:* Can "lazy load" from S3. (It reads data from S3 as a file system).
- **FSx for NetApp ONTAP:**
    - Exam Trigger: "Migrate NetApp", "Multi-protocol (NFS/SMB/iSCSI)", "Deduplication/Compression needed".

---

### **SECTION 4: HYBRID & MIGRATION**

**1. Storage Gateway (The Bridge)**

Connects On-Premise to Cloud.

- **File Gateway (S3 File Gateway):**
    - Interface: NFS/SMB.
    - Backend: S3.
    - Behavior: Local **Cache** of most recent files.
- **Volume Gateway:**
    - Interface: iSCSI (Block storage).
    - *Stored Mode:* All data on-prem, Async backup to EBS. (Low latency, full dataset local).
    - *Cached Mode:* Only hot data on-prem, Full data in S3/EBS. (Save local disk space).
- **Tape Gateway:** Virtual Tape Library (VTL). Replaces physical tapes with Glacier.

**2. Snow Family (The Truck)**

- **Snowcone:** Portable, rugged, 8TB. (Mailing a backpack).
- **Snowball Edge:**
    - *Storage Optimized:* 80TB. Data migration.
    - *Compute Optimized:* Run EC2/Lambda on the truck (Edge computing).
- **Snowmobile:** 100PB. (An actual 45-foot shipping container truck).
- *Rule:* Use when network transfer would take > 1 week.

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **High IOPS Block Storage?** → EBS io1/io2. Need 256K IOPS? → io2 Block Express.
2. **High Throughput/Streaming?** → EBS st1.
3. **Instance Store vs EBS?** → Instance Store = Ephemeral/Fast. EBS = Persistent/Network.
4. **Windows Shared Drive?** → FSx for Windows.
5. **Linux Shared Drive?** → EFS.
6. **HPC/Supercomputer?** → FSx for Lustre.
7. **S3 Encryption causing 503 errors?** → KMS Quota exceeded.
8. **Need to query data in S3 using SQL?** → S3 Select (or Athena).
9. **Move data to another region?** → S3 Cross-Region Replication (CRR) — Requires Versioning enabled.
10. **Protect against accidental S3 deletion?** → Enable Versioning (+ MFA Delete for extra protection).
11. **Reduce S3 costs over time automatically?** → Lifecycle Policy (transition + expiration rules).
12. **Replicate existing S3 objects?** → S3 Batch Replication (new replication rules only apply to new objects).
13. **WORM / Regulatory compliance / Cannot delete?** → S3 Object Lock (Compliance Mode) or Glacier Vault Lock.
14. **Temporary access to private S3 object?** → Pre-signed URL.
15. **Process files on S3 upload?** → S3 Event Notification → Lambda. Multiple targets? → EventBridge.
16. **Static website + HTTPS + custom domain?** → S3 + CloudFront (OAC) + Route 53 + ACM.
17. **Automate EBS backup schedule?** → Data Lifecycle Manager (DLM).
18. **No latency on first read from EBS snapshot?** → Fast Snapshot Restore (FSR).
19. **Delete markers not replicating?** → Delete markers are NOT replicated by default. Enable optional setting.
20. **S3 Object Lock modes?** → Governance = override with permissions. Compliance = nobody can delete, not even root.


# Scenarios

---

### **Scenario 1: The "Cluster" Problem (EBS Multi-Attach)**

**The Situation:** A legacy application runs on a cluster of 3 EC2 instances in a **single Availability Zone**. The application requires high-performance **block storage** that can be accessed by all three instances simultaneously to ensure data consistency.

**The Options:**

A. Use an S3 Bucket and mount it using S3FS.

B. Use an EBS gp3 volume and attach it to all three instances.

C. Use an EBS io2 volume with Multi-Attach enabled.

D. Use EFS with General Purpose performance mode.

**The Logic:**

- **Trap:** EFS (Option D) is "shared storage," but the question specified **block storage**, not file storage.
- **Trap:** gp3 (Option B) cannot be attached to multiple instances.
- **The Fix:** **Option C**. **EBS Multi-Attach** is only available on **io1/io2** volumes and allows multiple EC2s in the **same AZ** to share a block volume.

### **Scenario 2: The "Ephemeral" Trap (Instance Store)**

**The Situation:** You are designing a distributed cache using Redis on EC2. The application handles data replication at the software layer. You need the **highest possible disk I/O performance** and low latency. Data persistence on the disk is **not critical** if the instance stops.

**The Options:**

A. EBS io2 Block Express.

B. EBS gp3.

C. EC2 Instance Store.

D. EFS Max I/O mode.

**The Logic:**

- **Trap:** io2 Block Express (Option A) is fast, but it is network-attached.
- **The Fix:** **Option C**. **Instance Store** is physically attached to the server. It beats network storage every time on speed. The "data persistence not critical" line is your green light to use ephemeral storage.

### **Scenario 3: The "Windows Migration" (FSx)**

**The Situation:** A company is migrating 50 TB of data from an on-premise Windows File Server to AWS. The application uses **SMB protocol** and requires integration with **Active Directory** for permissions management.

**The Options:**

A. Amazon S3 with a Gateway Endpoint.

B. Amazon EFS with Windows ACLs.

C. Amazon FSx for Windows File Server.

D. Amazon EBS Multi-Attach.

**The Logic:**

- **Trap:** EFS (Option B) is Linux only (NFS). It does not speak SMB.
- **The Fix:** **Option C**. Keywords "Windows", "SMB", and "Active Directory" always point to **FSx for Windows**.

### **Scenario 4: The "Compliance Archive" (S3 Glacier)**

**The Situation:** A hospital must retain patient records for 7 years for regulatory compliance. These records are almost never accessed, but if an audit occurs, the data must be retrievable within **12 hours**. The solution must be the **lowest possible cost**.

**The Options:**

A. S3 Standard-IA.

B. S3 Glacier Flexible Retrieval.

C. S3 Glacier Deep Archive.

D. S3 Intelligent-Tiering.

**The Logic:**

- **Trap:** Glacier Flexible (Option B) works, but it's not the *lowest* cost.
- **The Fix:** **Option C**. **Deep Archive** is the cheapest storage class. The "12 hours" retrieval window fits perfectly with Deep Archive's standard retrieval time.

### **Scenario 5: The "KMS Bottleneck" (S3 Performance)**

**The Situation:** You are uploading millions of small files to an S3 bucket encrypted with **SSE-KMS**. Your application is receiving HTTP 503 "Slow Down" errors, but S3 metrics show you are well below the bucket throughput limits.

**The Options:**

A. Enable S3 Transfer Acceleration.

B. Increase the request quota for the KMS key.

C. Switch to S3 Multipart Upload.

D. Use DynamoDB instead.

**The Logic:**

- **Trap:** Multipart Upload (Option C) helps with *large* files, not *millions of small* files.
- **The Fix:** **Option B**. Every upload generates a call to KMS to generate a key. You are hitting the **KMS API limit**, not the S3 limit.