# Databases (Relational)

### **SECTION 1: RDS (RELATIONAL DATABASE SERVICE)**

*Managed SQL databases.*

### **1. RDS Overview**

- **The Rule:** AWS-managed relational database. AWS handles OS patching, backups, failover.
- **Supported Engines:**
    - PostgreSQL
    - MySQL
    - MariaDB
    - Oracle
    - Microsoft SQL Server
    - **Aurora** (AWS proprietary, covered separately)

**What AWS Manages:**

- OS patching
- Automated backups
- Point-in-time recovery
- Monitoring dashboards
- Multi-AZ failover (if enabled)

**What You Manage:**

- Query optimization
- Schema design
- Index management
- Application-level tuning

---

### **2. Storage (The Disk)**

**Storage Types:**

- **General Purpose SSD (gp2/gp3):** Default. Good for most workloads.
- **Provisioned IOPS (io1):** Consistent high IOPS for production DBs.
- **Magnetic:** Legacy. Not recommended.

**Storage Auto Scaling:**

- **The Rule:** RDS automatically increases storage when running low.
- **Trigger:** When free storage < 10% and condition lasts 5 minutes.
- **Max Storage Threshold:** You set max limit (up to 64 TB for most engines).
- *Exam Trigger:* "Unpredictable growth" → Enable Storage Auto Scaling.

---

### **3. Read Replicas (Scaling Reads)**

**The Rule:** Asynchronous replication. Offload **READ** traffic from primary DB.

**Key Facts:**

- **Replication:** **Asynchronous** (Eventual consistency. Slight lag possible).
- **Count:** Up to **5 Read Replicas** per primary (Standard RDS). Aurora supports up to **15**.
- **Location:** Same AZ, Cross-AZ, or **Cross-Region**.
- **Use Case:** Reporting, Analytics, Read-heavy workloads.
- **Promotion:** Can promote Read Replica to **standalone DB** (Breaks replication).
- **Cost:**
    - **Same Region:** No data transfer charge for replication.
    - **Cross-Region:** Data transfer charged.

**Application Changes:**

- **The Rule:** Application must explicitly connect to Read Replica endpoint.
- *Exam Trap:* Read Replicas do **not** automatically route traffic. You must update connection string.

**Supported Engines:** All RDS engines + Aurora.

**Read Replica Replication Lag:**

- Monitor `ReplicaLag` CloudWatch metric.
- High lag = Read Replica falling behind.

---

### **4. Multi-AZ (High Availability & Disaster Recovery)**

**The Rule:** **Synchronous** replication to **standby** instance in different AZ.

**Key Facts:**

- **Replication:** **Synchronous** (Zero data loss during failover).
- **Standby:** **NOT accessible**. Cannot read/write to it. Only for failover.
- **Failover:** Automatic (DNS flips to standby in 60-120 seconds).
- **Trigger Failover:**
    - Primary AZ failure
    - Primary instance failure
    - Manual reboot with failover
    - OS patching
- **Use Case:** Production workloads requiring **high availability**.
- **Cost:** ~2x (Running two instances).

**Exam Contrast: Multi-AZ vs. Read Replicas**

| **Feature** | **Multi-AZ** | **Read Replicas** |
| --- | --- | --- |
| **Purpose** | High Availability (Failover) | Scalability (Read performance) |
| **Replication** | **Synchronous** | **Asynchronous** |
| **Standby Accessible?** | **NO** (Standby not readable) | **YES** (Can query replicas) |
| **Failover** | Automatic (DNS switch) | Manual promotion |
| **Data Loss** | Zero (Sync replication) | Possible (Async lag) |
| **Cross-Region** | **NO** | **YES** |
| **Use Case** | DR, Production HA | Analytics, Reporting, Global reads |

**Multi-AZ Deployment (Two Options):**

1. **Multi-AZ with One Standby (Default):** One standby in different AZ.
2. **Multi-AZ with Two Readable Standbys (New):** Two standby replicas that **can serve reads**. Available for MySQL/PostgreSQL only.

---

### **5. Backups (Recovery)**

### **A. Automated Backups**

- **The Rule:** Daily full snapshot of DB + Transaction logs (Point-in-Time Recovery).
- **Retention:** 0-35 days (Default: 7 days). Set to 0 = Disabled.
- **Recovery:** Restore to **any second** within retention period.
- **Window:** Runs during backup window (AWS chooses or you specify).
- **Storage:** Stored in S3 (AWS-managed). No charge up to DB size. Extra charges beyond that.
- **Impact:** I/O may freeze for a few seconds during snapshot (Multi-AZ mitigates this).

### **B. Manual Snapshots**

- **The Rule:** User-triggered. Kept **forever** until manually deleted.
- **Use Case:** Before major changes, Long-term archival.
- **Cost:** Storage charges apply.

**Restore Process:**

- Creates a **new DB instance** (New endpoint). Does not overwrite existing DB.
- *Exam Trap:* "Restore DB" → Creates new instance, update app connection string.

---

### **6. Encryption**

**At-Rest Encryption:**

- **The Rule:** Encrypts data on disk using **AWS KMS**.
- **When:** Must enable at DB **creation**. Cannot enable later.
- **Workaround to Encrypt Existing DB:**
    1. Create snapshot of unencrypted DB.
    2. Copy snapshot → Enable encryption.
    3. Restore from encrypted snapshot.
- **Encryption Scope:** Includes backups, snapshots, Read Replicas.

**In-Transit Encryption:**

- **The Rule:** Use SSL/TLS to encrypt connection between app and DB.
- **Enforcement (PostgreSQL):** Set `rds.force_ssl=1` in Parameter Group.
- **Enforcement (MySQL):** Grant `REQUIRE SSL` to user accounts.

**Exam Trap:** "How to encrypt unencrypted DB?" → Snapshot → Copy with encryption → Restore.

---

### **7. Security**

- **Network:** Deploy in **Private Subnet**. Use Security Groups (Allow port 3306/5432 only from app SG).
- **IAM Authentication:**
    - **The Rule:** Use IAM roles to authenticate to DB (No passwords).
    - **Supported:** MySQL, PostgreSQL, Aurora.
    - **Mechanism:** Generate 15-minute auth token using IAM.
    - *Exam Trigger:* "Eliminate hardcoded DB passwords" → IAM DB Authentication.

---

### **8. RDS Custom**

**The Rule:** RDS Custom gives you **OS-level and database-level access** while AWS still manages backups, patching, and HA. Available for **Oracle** and **Microsoft SQL Server** only.

**Key Facts:**

- **OS Access:** Full SSH/RDP access to the underlying EC2 instance.
- **Database Access:** Can install custom patches, configure OS settings, install agents.
- **Automation Mode:** Pause AWS automation when making custom changes (prevents conflicts). Resume after.
- **Supported Engines:** Oracle, SQL Server only.

**How It Differs from Standard RDS:**

| **Feature** | **Standard RDS** | **RDS Custom** |
| --- | --- | --- |
| **OS Access** | **NO** | **YES** (SSH/RDP) |
| **Custom Patches** | **NO** (AWS manages) | **YES** |
| **Automation** | Always on | Can pause/resume |
| **Engines** | All 6 engines | Oracle, SQL Server only |

**Exam Trigger:** "Need root/admin OS access but still want managed database" → RDS Custom.

---

### **SECTION 2: AURORA (THE CLOUD-NATIVE DB)**

*AWS-built MySQL/PostgreSQL-compatible DB.*

### **1. Aurora Overview**

- **The Rule:** AWS proprietary DB. **5x faster than MySQL**, **3x faster than PostgreSQL** (AWS claims).
- **Compatible APIs:** MySQL 5.6/5.7/8.0, PostgreSQL 11/12/13/14/15.
- **Storage:** Auto-scales from **10 GB to 128 TB** in 10 GB increments.
- **Cost:** 20% more than RDS, but better performance/scaling.

**Why Aurora is Different:**

- **Storage:** Data stored across **3 AZs** with **6 copies** (2 per AZ).
- **Self-Healing:** Corrupted blocks auto-repaired using peer-to-peer replication.
- **Resilience:** Can lose 2 copies for writes, 3 copies for reads without impact.

---

### **2. Aurora High Availability**

**Architecture:**

- **One Writer Instance:** Handles all writes.
- **Up to 15 Read Replicas:** Handle reads.
- **Shared Storage:** All instances use the **same storage layer** (not replicated like RDS).
- **Failover:** If writer fails, Aurora promotes a Read Replica to writer (Typically < 30 seconds).

**Endpoints:**

- **Writer Endpoint:** Always points to current writer (Automatically updated during failover).
- **Reader Endpoint:** Load-balances across all Read Replicas.
- **Custom Endpoint:** Route specific queries to specific replica sets (e.g., analytics queries to larger instances).

---

### **3. Aurora Serverless**

**The Rule:** Auto-scales capacity based on load. Pay per second of use.

**How It Works:**

- **Aurora Capacity Units (ACUs):** 2 GB RAM each. Aurora auto-adjusts ACUs based on CPU/connections/memory.
- **Min/Max ACU:** You set boundaries (e.g., 2-16 ACUs).
- **Pause:** Can auto-pause after inactivity (Pay $0 for compute, only storage).
- **Resume:** Resumes in seconds when connection request arrives.

**Use Cases:**

- Infrequent/unpredictable workloads
- Development/Test environments
- New applications with unknown load

**Key Points (Aurora Serverless v2 - only current version):**

- Scales in **seconds**.
- Supports **Read Replicas**.
- Suitable for production workloads.
- *(Aurora Serverless v1 was deprecated March 2025. Only v2 exists now.)*

**Exam Trigger:** "Variable load, pay only for what you use" → Aurora Serverless v2.

---

### **4. Aurora Global Database**

**The Rule:** One **Primary Region** (Read/Write) + Up to **5 Secondary Regions** (Read-Only).

**Replication:**

- **Latency:** Typically < 1 second to secondary regions.
- **Mechanism:** Physical replication at storage layer (Not logical).

**Use Cases:**

- Disaster Recovery (Promote secondary region in < 1 minute).
- Global read performance (Serve users from local region).

**Failover:**

- Promote secondary region to primary. Takes < 1 minute (RTO < 1 min).

**Exam Trigger:** "Global low-latency reads, RTO < 1 minute" → Aurora Global Database.

---

### **5. Backtrack**

**The Rule:** Rewind DB to previous point in time **without restoring from backup**.

**How It Works:**

- Enabled at cluster creation.
- Can backtrack up to 72 hours.
- Takes **seconds** (vs. minutes/hours for snapshot restore).
- Does **not** create new DB instance (In-place rewind).

**Use Cases:**

- Undo bad write query
- Recover from accidental DELETE

**Exam Trigger:** "Quickly undo bad data changes without restore" → Aurora Backtrack.

---

### **6. Aurora Cloning**

**The Rule:** Create a **copy-on-write clone** of an Aurora DB cluster. Fast, storage-efficient.

**How It Works:**

- Uses **copy-on-write protocol** — clone initially shares the same data pages as the source.
- New data pages are only allocated when data is **modified** in the clone.
- Creation takes **seconds** regardless of database size (vs. hours for snapshot+restore).

**Key Facts:**

- Clone is a fully independent Aurora cluster (own endpoint, own configuration).
- No performance impact on the source/production database.
- Much faster and more storage-efficient than snapshot+restore.
- Useful for creating multiple clones from the same source.

**Use Cases:**

- Create test/staging copy of production DB.
- Run experiments or analytics without affecting production.
- Debug production issues with real data.

**Exam Trigger:** "Create test copy of production database quickly without affecting production" → Aurora Cloning.

---

### **SECTION 3: RDS PROXY**

**The Rule:** Managed connection pooling for RDS/Aurora.

**Problem It Solves:**

- Applications open/close many DB connections → DB exhausts connection limit.
- Lambda functions spike connections (1000 concurrent = 1000 DB connections).

**How It Works:**

- RDS Proxy maintains a **pool of connections** to DB.
- Applications connect to Proxy (Proxy reuses connections).
- Reduces DB connection overhead by ~50%.

**Features:**

- **Failover:** Reduces failover time by 66% (Proxy handles connection gracefully).
- **IAM Authentication:** Enforce IAM auth. Store credentials in Secrets Manager.
- **VPC:** Must deploy in **same VPC** as DB.

**Exam Trigger:** "Lambda causing too many DB connections" → RDS Proxy.

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **Scale read workload?** → Read Replicas (Async).
2. **High availability for production?** → Multi-AZ (Sync, auto-failover).
3. **Standby is not accessible** → Multi-AZ.
4. **Standby is accessible** → Read Replicas.
5. **Encrypt existing unencrypted DB?** → Snapshot → Copy with encryption → Restore.
6. **Eliminate DB password?** → IAM Database Authentication.
7. **Auto-scale storage?** → Enable Storage Auto Scaling.
8. **5x faster MySQL?** → Aurora.
9. **Variable/unpredictable load?** → Aurora Serverless.
10. **Global reads, DR with RTO < 1 min?** → Aurora Global Database.
11. **Lambda exhausting DB connections?** → RDS Proxy.
12. **Quickly undo bad data change?** → Aurora Backtrack.
13. **Create test copy of production DB fast?** → Aurora Cloning (copy-on-write, seconds).
14. **Need OS-level access on managed DB?** → RDS Custom (Oracle/SQL Server only).

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Read-Heavy Analytics" (Read Replicas)**

**The Situation:** A production MySQL database serves a web application with thousands of users. The data team wants to run long-running analytical queries but these queries slow down the primary database, causing performance issues for users.

**The Options:**

A. Enable Multi-AZ deployment.

B. Create a Read Replica and point analytics queries to the replica endpoint.

C. Increase the instance size of the primary DB.

D. Migrate to Aurora Serverless.

**The Logic:**

- **Trap:** Multi-AZ (Option A) provides HA but the standby is **not accessible** for queries.
- **Trap:** Increasing instance size (Option C) helps but is expensive and doesn't isolate workloads.
- **The Fix:** **Option B**. **Read Replicas** offload read traffic from the primary. Point analytics queries to the Read Replica endpoint. Asynchronous replication ensures primary performance is unaffected.

---

### **Scenario 2: The "Production Failover" (Multi-AZ)**

**The Situation:** A financial application requires a MySQL database with **zero data loss** during failures. The database must automatically failover to a standby instance if the primary AZ becomes unavailable. RTO (Recovery Time Objective) must be under 2 minutes.

**The Options:**

A. Use Read Replicas in multiple AZs and manually promote one.

B. Enable Multi-AZ deployment.

C. Take hourly snapshots and restore when needed.

D. Use Aurora Serverless.

**The Logic:**

- **Trap:** Read Replicas (Option A) require **manual** promotion and use **async** replication (possible data loss).
- **Trap:** Snapshots (Option C) take minutes/hours to restore (Exceeds RTO).
- **The Fix:** **Option B**. **Multi-AZ** provides **synchronous** replication (zero data loss) and **automatic failover** via DNS switch (60-120 seconds). Meets both requirements.

---

### **Scenario 3: The "Lambda Connection Storm" (RDS Proxy)**

**The Situation:** A serverless application uses Lambda functions to query a PostgreSQL RDS database. During traffic spikes, Lambda scales to 2000 concurrent executions. The database starts rejecting connections with "too many connections" errors.

**The Options:**

A. Increase the DB instance size to support more connections.

B. Use RDS Proxy to pool database connections.

C. Switch to DynamoDB.

D. Enable Multi-AZ.

**The Logic:**

- **Trap:** Increasing instance size (Option A) increases connection limit but doesn't solve the fundamental problem (Each Lambda = 1 connection).
- **Trap:** Multi-AZ (Option D) doesn't increase connection capacity.
- **The Fix:** **Option B**. **RDS Proxy** pools connections. 2000 Lambdas share a pool of (e.g.) 100 connections to the DB. Proxy multiplexes connections efficiently. Also improves failover time by 66%.

---

### **Scenario 4: The "Global Reads" (Aurora Global Database)**

**The Situation:** An e-commerce company has customers in North America (us-east-1) and Europe (eu-west-1). The application requires **low-latency reads** in both regions. Writes only happen in North America. In case of regional failure, they need to promote the European region to primary in **under 1 minute**.

**The Options:**

A. Use RDS Read Replicas in eu-west-1.

B. Use Aurora Global Database with primary in us-east-1 and secondary in eu-west-1.

C. Run two separate RDS instances and replicate with DMS.

D. Use S3 Cross-Region Replication.

**The Logic:**

- **Trap:** RDS Read Replicas (Option A) work for reads but promoting a cross-region replica takes 5-10 minutes (Exceeds RTO).
- **Trap:** DMS (Option C) adds complexity and doesn't meet RTO requirement.
- **The Fix:** **Option B**. **Aurora Global Database** replicates to eu-west-1 with < 1 second latency. European users read from local secondary. If us-east-1 fails, promote eu-west-1 to primary in < 1 minute. Meets both read latency and RTO requirements.

---

### **Scenario 5: The "Accidental DELETE" (Aurora Backtrack)**

**The Situation:** A developer accidentally runs `DELETE FROM orders WHERE status='pending'` without a WHERE clause, deleting **all orders** from the production Aurora database. The mistake is discovered 30 minutes later. The business needs the data back **immediately** with minimal downtime.

**The Options:**

A. Restore from the latest automated backup (taken 6 hours ago).

B. Restore from a manual snapshot (taken yesterday).

C. Use Aurora Backtrack to rewind the database to 30 minutes ago.

D. Contact AWS Support.

**The Logic:**

- **Trap:** Automated backup (Option A) restores to 6 hours ago (Loses 5.5 hours of data).
- **Trap:** Manual snapshot (Option B) loses even more data (24 hours).
- **The Fix:** **Option C**. **Aurora Backtrack** rewinds the DB to the exact point before the DELETE (30 minutes ago) in **seconds**. No data loss. No new DB instance. Application reconnects and continues. Requires Backtrack to be enabled at cluster creation (retention up to 72 hours).
