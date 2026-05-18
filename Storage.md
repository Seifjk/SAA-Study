# Storage

### **SECTION 1: BLOCK STORAGE (EBS & INSTANCE STORE)**

*The "Hard Drive" attached to your EC2.*

### **1. Instance Store (Ephemeral Storage)**

- **The Rule:** Physically-attached SSD/HDD on the host. **Ephemeral** — survives **reboot**, lost on **stop** or hardware failure.
- **Performance:** Highest possible IOPS (millions) — local, not network-attached.
- **Use Case:** Cache, buffers, temporary content, distributed data stores (Kafka, Cassandra) where the app handles replication.
- **Exam Trap:** "High I/O needs and data can be lost" → **Instance Store**.

### **2. EBS (Elastic Block Store)**

- **The Rule:** Network-attached, **persistent** drive (data survives a stop). Slightly higher latency than Instance Store.
- **Scope:** Locked to **ONE AZ**. To move: Snapshot → Copy Snapshot to Region/AZ → Create Volume.

**EBS Volume Types (The IOPS/Throughput Game)**

| **Type** | **Name** | **Use Case** | **Performance / IOPS** | **Exam "Trigger"** |
| --- | --- | --- | --- | --- |
| **General Purpose** | **gp2** | Boot volumes, Dev/Test | **IOPS Linked to Size.** 3 IOPS per GB (min 100, max 16,000 IOPS). | "Balance price/performance" |
| **General Purpose** | **gp3** | **Default Choice.** | **Decoupled Performance.** You get 3,000 IOPS baseline free. You pay to increase IOPS without increasing size. | "Independent scaling of storage and IOPS" |
| **Provisioned IOPS** | **io1 / io2** | Mission Critical DBs | **High IOPS.** io1: up to 32,000 IOPS. io2: up to 64,000 IOPS. | "Sub-millisecond latency", "Sustained IOPS" |
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

- **Strong Consistency:** Write then immediately read → you get the new version. No eventual-consistency delay.

**2. Storage Classes (Memorize the Waterfall)**

| **Class** | **Availability** | **Minimum Duration** | **Retrieval Fee?** | **Exam Trigger** |
| --- | --- | --- | --- | --- |
| **Standard** | 99.99% | None | No | "Frequently accessed", "Instant" |
| **Standard-IA** | 99.9% | 30 Days | Yes | "Disaster Recovery", "Once a month access" |
| **Intelligent-Tiering** | 99.9% | None | No | "Unknown/Changing access patterns" (has 6 tiers incl. Archive Access + Deep Archive Access) |
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

- Keeps every version of every object (including deletions). Once enabled, can only be **suspended**, not disabled — existing versions are preserved.
- **Delete Behavior:** Simple DELETE → places a **Delete Marker** (soft delete; recover by removing it). Permanent DELETE requires the exact **Version ID**.
- **MFA Delete:** Optional. Requires MFA to permanently delete versions or change versioning state. Enabled only by **root account** via CLI.
- **Required for Replication** (CRR/SRR) on both source and destination buckets.
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

- **CRR (Cross-Region):** to a **different region** — compliance, low-latency access, DR. **SRR (Same-Region):** within one region — log aggregation, prod/test replication.
- **Requirements:** Versioning enabled on both buckets + an IAM role for S3.
- **Key Rules:** Existing objects are NOT replicated (use **S3 Batch Replication**). No chaining (A→B→C doesn't propagate A to C). Delete markers not replicated by default (optional). Permanent deletes by version ID are never replicated.
- *Exam Trap:* "Replicate existing objects" → S3 Batch Replication.

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

- Temporary URL granting time-limited access to a **private** object; inherits the generating IAM identity's permissions.
- **Expiration:** Default 1 hour, max 7 days (IAM user credentials). Works for download or upload to a specific key.
- *Exam Trigger:* "Temporary access to a private object", "Share a file without making it public", "Allow upload without AWS credentials" → Pre-signed URL.

**10. S3 Event Notifications**

- **Triggers:** `s3:ObjectCreated:*`, `s3:ObjectRemoved:*`, `s3:ObjectRestore:*`, etc.
- **Destinations:** SQS (async queue), SNS (fan-out), Lambda (serverless processing), EventBridge.
- **EventBridge Advantage:** All event types, advanced filtering, archive/replay, 18+ service targets — the modern, flexible option.
- *Exam Trigger:* "Process files on upload" → S3 Event Notification to Lambda. "Fan out to multiple services" → EventBridge.

**11. S3 Select / Glacier Select**

- SQL expressions to retrieve only specific columns/rows from objects (CSV, JSON, Parquet) — S3 filters server-side, cutting data transfer/cost. Glacier Select = same for archived objects.
- **Not Athena:** S3 Select = single object; Athena = many objects via Glue Catalog.
- *Exam Trigger:* "Query specific columns from CSV in S3" / "Reduce data transfer from S3" → S3 Select.

**12. S3 Object Lambda**

- Intercepts S3 GET requests and transforms data on-the-fly via Lambda before returning it (redact PII, convert XML→JSON, resize images, filter rows). Clients call an Object Lambda Access Point instead of the bucket.
- *Exam Trigger:* "Transform S3 objects on retrieval without storing multiple copies" → S3 Object Lambda.

**13. S3 Batch Operations**

- Bulk operations on billions of objects in one request: copy, replace tags, modify ACLs, restore from Glacier, invoke Lambda per object, Batch Replication. Driven by an inventory list (S3 Inventory or CSV).
- *Exam Trigger:* "Bulk tag/copy/process existing S3 objects" → S3 Batch Operations.

**14. S3 Static Website Hosting**

- Hosts static sites (HTML/CSS/JS). Endpoint: `http://<bucket>.s3-website-<region>.amazonaws.com`
- Requires public access (Bucket Policy `s3:GetObject` to `*`) OR CloudFront with **OAC** to keep the bucket private.
- *Exam Combo:* S3 + CloudFront (CDN) + OAC + Route 53 (domain) + ACM (HTTPS) — very common architecture question.
- *Exam Trap:* S3 website endpoint does NOT support HTTPS natively — need CloudFront in front.

---

### **SECTION 3: NETWORK FILE STORAGE (EFS vs FSx)**

*The "Shared Drive" for multiple servers.*

**1. EFS (Elastic File System)**

- **NFS v4**, **Linux only** (POSIX). Scales to petabytes automatically. **Multi-AZ**.
- **Performance Modes:** *General Purpose* (default, low latency, web servers); *Max I/O* (high throughput, higher latency, Big Data).
- **Throughput Modes:** *Elastic* (default/recommended, auto-scales); *Provisioned* (fixed speed); *Bursting* (tied to size, legacy).

**2. FSx (File System for X)**

- **FSx for Windows:** SMB protocol, Active Directory integration, DFS + Deduplication. *Trigger:* "Migrate Windows File Server", "SharePoint", "SMB share".
- **FSx for Lustre:** Parallel distributed file system for **HPC**, ML, video rendering. Can "lazy load" from S3 as a file system.
- **FSx for NetApp ONTAP:** *Trigger:* "Migrate NetApp", "Multi-protocol (NFS/SMB/iSCSI)", "Deduplication/Compression".

---

### **SECTION 4: HYBRID & MIGRATION**

**1. Storage Gateway (The Bridge)** — connects on-premise to cloud.

- **File Gateway (S3):** NFS/SMB interface, S3 backend, local cache of recent files.
- **Volume Gateway:** iSCSI block storage. *Stored Mode* = all data on-prem, async backup to EBS. *Cached Mode* = hot data on-prem, full data in S3/EBS.
- **Tape Gateway:** Virtual Tape Library (VTL) replacing physical tapes with Glacier.

**2. Snow Family (The Truck)**

- **Snowcone:** Portable, rugged, 8TB. **Snowball Edge:** *Storage Optimized* (80TB, migration); *Compute Optimized* (run EC2/Lambda for edge computing). **Snowmobile:** 100PB shipping container truck.
- *Rule:* Use when network transfer would take > 1 week.

**3. AWS DataSync**

- Agent-based service to migrate/sync data between on-prem and AWS (or between AWS services). Supports NFS, SMB, HDFS → S3, EFS, FSx. Auto encryption, integrity validation, scheduling.
- **vs Storage Gateway:** DataSync = migration/sync (move data). Storage Gateway = ongoing hybrid access.
- *Exam Trigger:* "Migrate NFS share to EFS", "One-time large data migration to S3", "Ongoing on-prem → AWS sync" → **DataSync**.

**4. AWS Transfer Family**

- Managed **SFTP/FTPS/FTP** service for transferring files to/from S3 and EFS.
- *Exam Trigger:* "Existing SFTP workflow needs to transfer files to S3", "FTP server for S3" → **Transfer Family**.

**5. S3 Access Points**

- Named network endpoints, each with its own **DNS name** and **IAM policy**, attached to a shared bucket — simplifies access for multiple apps hitting one bucket.
- *Exam Trigger:* "Simplify S3 access for multiple applications", "Dedicated S3 endpoint per application" → **S3 Access Points**.

**6. S3 Requester Pays**

- The **requester** (not bucket owner) pays for transfer and request costs; owner still pays storage. Requester must be an authenticated AWS user.
- *Exam Trigger:* "Share large dataset without paying for transfer" → **Requester Pays**.

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
21. **Migrate NFS share to EFS or sync data on-prem → AWS?** → DataSync (agent-based, supports NFS/SMB/HDFS → S3/EFS/FSx).
22. **Existing SFTP/FTP workflow to S3?** → Transfer Family.
23. **Multiple apps need separate S3 access policies on one bucket?** → S3 Access Points (dedicated DNS + policy per app).
24. **Share large S3 dataset without paying transfer costs?** → S3 Requester Pays.
25. **Transform S3 objects on retrieval (redact PII, convert format)?** → S3 Object Lambda.
26. **Bulk operations on existing S3 objects (tag, copy, process)?** → S3 Batch Operations.
27. **Auto-archive infrequently accessed objects with no retrieval fees?** → S3 Intelligent-Tiering (configure Archive Access + Deep Archive tiers).


# **REAL EXAM SCENARIOS**

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