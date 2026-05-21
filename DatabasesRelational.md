# Databases (Relational)

> **Start here — what is a "relational" database?**
> It's the classic, traditional kind of database: data lives in **tables** (rows and columns), tables can be **joined** together, and you query it with **SQL**. Think MySQL, PostgreSQL, Oracle. If a question mentions "SQL", "tables", "joins", "transactions", or "ACID" → it's relational, and this chapter is the answer space.
>
> **What this chapter is really about:** Running a database yourself means patching the OS, configuring backups, and handling failure at 3 AM. AWS's relational services (**RDS** and **Aurora**) do all of that *for* you — you just use the database. The exam doesn't test SQL itself; it tests **two operational questions**:
> 1. *How do I make the database survive a failure?* → **Multi-AZ**
> 2. *How do I handle lots of read traffic?* → **Read Replicas**
>
> Almost every relational question is a variation of those two. Keep them straight and most of the chapter falls into place.

---

### **SECTION 1: RDS (RELATIONAL DATABASE SERVICE)**

> **What is RDS?** It's AWS taking a standard database engine (MySQL, PostgreSQL, etc.) and running it on a server *for you* — handling the boring, risky operational work. You still design your tables and write your queries; AWS handles patching, backups, and failover. It's the "managed database" — *managed* meaning **AWS does the maintenance**.

### **1. RDS Overview**

- **The Rule:** AWS-managed relational database. **Engines:** PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, **Aurora** (proprietary, covered separately).
- **AWS manages:** OS patching, automated backups, point-in-time recovery, monitoring, Multi-AZ failover.
- **You manage:** Query optimization, schema design, indexes, application-level tuning.

---

### **2. Storage (The Disk)**

- **Types:** General Purpose SSD (gp2/gp3, default); Provisioned IOPS (io1, consistent high IOPS for prod); Magnetic (legacy).
- **Storage Auto Scaling:** RDS auto-increases storage when free < 10% for 5 min. You set a max threshold (up to 64 TB most engines).
- *Exam Trigger:* "Unpredictable growth" → Enable Storage Auto Scaling.

---

### **3. Read Replicas (Scaling Reads)**

> **The problem:** Your one database server is getting hammered — too many people *reading* data (reports, searches, dashboards) and it's slowing down. You can't just keep buying a bigger server forever.
> **The fix:** Make extra **read-only copies** of the database. Send all the read traffic to the copies; the original keeps handling writes. That's a Read Replica. Key word: **scaling** — it's about *handling more load*, not about surviving failure.

**The Rule:** Asynchronous replication to offload **READ** traffic from the primary (eventual consistency, slight lag possible).

**Key Facts:**

- **Count:** Up to **15** per primary for RDS MySQL/MariaDB/PostgreSQL; **5** for Oracle/SQL Server; Aurora up to **15**. *(Older material says 5 for all RDS — if the exam offers 5, it's accepting the legacy number.)*
- **Location:** Same AZ, Cross-AZ, or **Cross-Region** (cross-region replication incurs data transfer charges).
- **Use Case:** Reporting, analytics, read-heavy workloads. Can promote a replica to a **standalone DB** (breaks replication).
- *Exam Trap:* Replicas do NOT auto-route traffic — the app must connect to the replica endpoint explicitly.
- **Lag:** Monitor the `ReplicaLag` CloudWatch metric.

---

### **4. Multi-AZ (High Availability & Disaster Recovery)**

> **The problem:** Your database runs in one data center (Availability Zone). If that AZ has a power/hardware failure, your whole app is down — and any data not yet saved elsewhere is gone.
> **The fix:** AWS keeps a **standby copy** in a *different* AZ, kept perfectly in sync. If the primary dies, AWS automatically flips to the standby in ~1–2 minutes. Key word: **availability** — it's about *surviving failure*, not handling more load. (Note the contrast with Read Replicas above: Replicas = scale, Multi-AZ = survive. The exam loves to test exactly this difference — see the table below.)

**The Rule:** **Synchronous** replication to a **standby** in a different AZ — zero data loss during failover.

**Key Facts:**

- **Standby:** NOT accessible (no reads/writes) — failover only.
- **Failover:** Automatic, DNS flips to standby in 60-120 sec. Triggered by primary AZ/instance failure, reboot-with-failover, or OS patching.
- **Cost:** ~2x (two instances). Use case: production HA.

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

- Daily full snapshot + transaction logs (Point-in-Time Recovery to any second within retention).
- **Retention:** 0-35 days (default 7; 0 = disabled). Stored in S3, no charge up to DB size.
- I/O may freeze a few seconds during snapshot (Multi-AZ mitigates this).

### **B. Manual Snapshots**

- User-triggered, kept **forever** until manually deleted. Use before major changes or for long-term archival. Storage charges apply.

**Restore Process:** Creates a **new DB instance** (new endpoint), does not overwrite. *Exam Trap:* "Restore DB" → update app connection string to the new endpoint.

---

### **6. Encryption**

**At-Rest Encryption:**

- KMS-based, must be enabled at DB **creation** (cannot enable later). Scope includes backups, snapshots, Read Replicas.
- **Encrypt an existing DB:** Snapshot → Copy snapshot with encryption enabled → Restore from encrypted snapshot.

**In-Transit Encryption:**

- SSL/TLS between app and DB. Enforce: PostgreSQL `rds.force_ssl=1` (Parameter Group); MySQL `REQUIRE SSL` on user accounts.

**Exam Trap:** "How to encrypt unencrypted DB?" → Snapshot → Copy with encryption → Restore.

---

### **7. Security**

- **Network:** Deploy in a **Private Subnet**; Security Group allows port 3306/5432 only from the app SG.
- **IAM Authentication:** Authenticate via IAM (no passwords) using a 15-minute auth token. Supports MySQL, PostgreSQL, Aurora. *Exam Trigger:* "Eliminate hardcoded DB passwords" → IAM DB Authentication.

---

### **8. RDS Custom**

**The Rule:** RDS Custom gives **OS-level and database-level access** while AWS still manages backups, patching, and HA. **Oracle** and **SQL Server** only.

**Key Facts:** Full SSH/RDP to the underlying EC2; install custom patches/agents and tweak OS settings. Pause AWS automation while making custom changes (prevents conflicts), then resume.

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

> **What is Aurora, and why does it exist separately from RDS?** RDS just runs the *standard* MySQL/PostgreSQL software on a server. Aurora is AWS's own **rewritten-from-scratch** version that *speaks the same language* (your MySQL/PostgreSQL app works unchanged) but with a smarter storage engine underneath — faster, and self-healing across 3 AZs automatically.
> **The mental model:** RDS = AWS *runs* a normal database for you. Aurora = AWS *built a better database* that's compatible with the normal one. It costs ~20% more; you pick it when you need its extra power (speed, the 15 replicas, Global Database, Serverless, Backtrack). Everything below is an Aurora-only superpower — each solves a specific problem.

### **1. Aurora Overview**

- **The Rule:** AWS proprietary DB. **5x faster than MySQL**, **3x faster than PostgreSQL** (AWS claims). Cost ~20% more than RDS.
- **Compatible APIs:** MySQL 5.7/8.0, PostgreSQL 13–17. Storage auto-scales **10 GB to 128 TB**. *(Exact version numbers are not tested — just know "your existing MySQL/PostgreSQL app works unchanged.")*
- **Why it's different:** Data stored across **3 AZs with 6 copies** (2 per AZ). Self-healing repairs corrupt blocks via peer-to-peer replication. Tolerates loss of 2 copies for writes, 3 for reads.

---

### **2. Aurora High Availability**

**Architecture:** One writer + up to 15 Read Replicas, all sharing the **same storage layer** (not replicated like RDS). If the writer fails, a replica is promoted to writer (typically < 30 sec).

**Endpoints:**

- **Writer Endpoint:** Always points to the current writer (auto-updated on failover).
- **Reader Endpoint:** Load-balances across all replicas.
- **Custom Endpoint:** Route specific queries to specific replica sets (e.g., analytics to larger instances).

---

### **3. Aurora Serverless**

**The Rule:** Auto-scales capacity based on load. Pay per second of use.

- **Aurora Capacity Units (ACUs):** 2 GB RAM each, auto-adjusted on CPU/connections/memory; you set min/max boundaries. Can auto-pause after inactivity ($0 compute, storage only) and resume in seconds.
- **Use Cases:** Infrequent/unpredictable workloads, dev/test, new apps with unknown load.
- **Serverless v2 (only current version):** Scales 0.5–128 ACUs in 0.5 increments, in seconds; supports Read Replicas; production-suitable. *(v1 deprecated March 2025.)*

**Exam Trigger:** "Variable load, pay only for what you use" → Aurora Serverless v2.

---

### **4. Aurora Global Database**

**The Rule:** One **Primary Region** (R/W) + up to **5 Secondary Regions** (read-only). Physical storage-layer replication, typically < 1 sec latency.

- **Use Cases:** DR (promote a secondary region to primary in < 1 min, RTO < 1 min) and global low-latency reads from a local region.

**Exam Trigger:** "Global low-latency reads, RTO < 1 minute" → Aurora Global Database.

---

### **5. Backtrack**

**The Rule:** Rewind DB to a previous point in time **without restoring from backup** — in-place, takes seconds (no new instance).

- Enabled at cluster creation; backtracks up to 72 hours. Use to undo a bad write query or accidental DELETE.

**Exam Trigger:** "Quickly undo bad data changes without restore" → Aurora Backtrack.

---

### **6. Aurora Cloning**

**The Rule:** Create a **copy-on-write clone** of an Aurora cluster — fast and storage-efficient.

- Clone initially shares the source's data pages; new pages allocated only when data is modified. Creation takes **seconds** regardless of DB size (vs. hours for snapshot+restore).
- Clone is a fully independent cluster (own endpoint/config), with no performance impact on the source.
- **Use Cases:** Test/staging copies, experiments/analytics, debugging production issues with real data.

**Exam Trigger:** "Create test copy of production database quickly without affecting production" → Aurora Cloning.

---

### **SECTION 3: RDS PROXY**

**The Rule:** Managed connection pooling for RDS/Aurora — apps connect to the Proxy, which maintains and reuses a pool of DB connections (~50% less overhead).

- **Problem solved:** Apps/Lambda spike connections (1000 concurrent = 1000 DB connections) and exhaust the DB limit.
- **Features:** Cuts failover time ~66%; supports IAM auth with credentials in Secrets Manager; must deploy in the **same VPC** as the DB.

**Exam Trigger:** "Lambda causing too many DB connections" → RDS Proxy.

---

### **SECTION 4: RDS OPERATIONS & MONITORING**

### **1. Aurora Blue/Green Deployments**

- **Purpose:** Zero-downtime database updates (engine upgrades, schema changes, parameter changes).
- **How it works:** Create a staging environment (Green) alongside production (Blue). Test changes in Green. Switchover with minimal downtime (~1 minute). Rollback if issues found.
- **Key Point:** Green environment stays in sync via replication. Switchover updates the DB endpoint — no application changes needed.
- *Exam Trigger:* "Zero-downtime database upgrade", "Test DB changes before production switchover" → Blue/Green Deployment.

### **2. RDS Performance Insights**

- **Purpose:** Real-time database performance monitoring. Visual dashboard showing active sessions, wait events, and SQL queries.
- **Identifies bottlenecks:** Is the issue CPU, I/O, lock contention, or a specific query?
- **Available for:** All RDS engines and Aurora.
- *Exam Trigger:* "Database slow, need to identify bottleneck" → RDS Performance Insights.

### **3. RDS Event Notifications**

- **Purpose:** Get alerts on DB events via **SNS** (email, SMS, Lambda, SQS).
- **Events include:** Failover, backup, restore, maintenance, configuration change, deletion.
- **How it works:** Create an Event Subscription → select event categories → choose SNS topic as destination.
- *Exam Trigger:* "Alert when RDS fails over" or "Notify on DB maintenance" → RDS Event Subscription + SNS.

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
15. **Zero-downtime DB upgrade?** → Aurora Blue/Green Deployment.
16. **DB performance bottleneck analysis?** → RDS Performance Insights.
17. **Alert on DB failover/maintenance?** → RDS Event Notifications → SNS.

---

# **REAL EXAM SCENARIOS**

### Scenario 1

**The Situation:** A production MySQL database serves a web application with thousands of users. The data team wants to run long-running analytical queries but these queries slow down the primary database, causing performance issues for users.

**The Options:**

A. Enable Multi-AZ deployment.

B. Create a Read Replica and point analytics queries to the replica endpoint.

C. Increase the instance size of the primary DB.

D. Migrate to Aurora Serverless.

**The Logic:**

- **Trap A — Multi-AZ:** The Multi-AZ standby is a *failover* target only — it serves **no read traffic**. You cannot point analytics queries at it, so it does nothing for this problem.
- **Trap C — Bigger primary instance:** Adds headroom but the analytical queries still run **on the same database**, competing with user traffic. It doesn't *isolate* the two workloads, and it's costly.
- **Trap D — Migrate to Aurora Serverless:** A full engine migration to solve a read-contention problem is heavy-handed. Aurora Serverless auto-scales capacity but analytics would *still* hit the writer unless you also add replicas — it doesn't address the actual issue.
- **The Fix — Option B:** A **Read Replica** gives analytics its own endpoint. Heavy queries run on the replica; the primary is untouched. Async replication keeps the replica current without slowing the writer. Workload isolation = the textbook Read Replica use case.

---

### Scenario 2

**The Situation:** A financial application requires a MySQL database with **zero data loss** during failures. The database must automatically failover to a standby instance if the primary AZ becomes unavailable. RTO (Recovery Time Objective) must be under 2 minutes.

**The Options:**

A. Use Read Replicas in multiple AZs and manually promote one.

B. Enable Multi-AZ deployment.

C. Take hourly snapshots and restore when needed.

D. Use Aurora Serverless.

**The Logic:**

- **Trap A — Read Replicas + manual promotion:** Two failures against the requirements: replication is **asynchronous** (so in-flight writes can be lost — violates "zero data loss"), and promotion is **manual** (violates "automatic failover").
- **Trap C — Hourly snapshots + restore:** Restoring a snapshot creates a new instance — minutes to hours. Blows the under-2-minute RTO, and you'd lose up to an hour of data.
- **Trap D — Aurora Serverless:** Auto-scales capacity, but the question is about *failover behavior and data durability*, not scaling. It doesn't specifically deliver synchronous zero-loss failover the way the Multi-AZ answer does, and switching engines isn't asked for.
- **The Fix — Option B:** **Multi-AZ** uses **synchronous** replication to the standby (zero data loss) and fails over **automatically** via a DNS change in ~60–120 seconds — satisfying both the zero-data-loss and sub-2-minute RTO requirements.

---

### Scenario 3

**The Situation:** A serverless application uses Lambda functions to query a PostgreSQL RDS database. During traffic spikes, Lambda scales to 2000 concurrent executions. The database starts rejecting connections with "too many connections" errors.

**The Options:**

A. Increase the DB instance size to support more connections.

B. Use RDS Proxy to pool database connections.

C. Switch to DynamoDB.

D. Enable Multi-AZ.

**The Logic:**

- **Trap A — Bigger DB instance:** A larger instance raises `max_connections`, but 2,000 concurrent Lambdas can still outgrow it — you're chasing the symptom. It delays the wall, doesn't remove it, and costs more.
- **Trap C — Switch to DynamoDB:** DynamoDB has no connection model, so it "solves" the error — but rewriting a relational PostgreSQL app onto a NoSQL data model is an enormous change the question never asked for. Wrong scope.
- **Trap D — Enable Multi-AZ:** Multi-AZ is for failover/HA. It does **not** add connection capacity — the standby serves no traffic. Irrelevant to a connection-exhaustion problem.
- **The Fix — Option B:** **RDS Proxy** sits between Lambda and the DB and **pools/multiplexes** connections — 2,000 Lambdas share a small pool (e.g. 100) to the database. It's the purpose-built fix for the Lambda "connection storm," and it also speeds up failover.

---

### Scenario 4

**The Situation:** An e-commerce company has customers in North America (us-east-1) and Europe (eu-west-1). The application requires **low-latency reads** in both regions. Writes only happen in North America. In case of regional failure, they need to promote the European region to primary in **under 1 minute**.

**The Options:**

A. Use RDS Read Replicas in eu-west-1.

B. Use Aurora Global Database with primary in us-east-1 and secondary in eu-west-1.

C. Run two separate RDS instances and replicate with DMS.

D. Use S3 Cross-Region Replication.

**The Logic:**

- **Trap A — RDS cross-region Read Replicas:** Delivers the low-latency local reads, but promoting a cross-region RDS replica to standalone primary takes ~5–10 minutes — it **misses the under-1-minute RTO**.
- **Trap C — Two RDS instances replicated with DMS:** DMS is a *migration* tool, not a resilient HA replication layer. It adds operational complexity and offers no fast, clean regional-failover promotion. Doesn't meet the RTO.
- **Trap D — S3 Cross-Region Replication:** S3 replicates **objects**, not a relational database. It cannot serve transactional SQL reads at all — wrong service category.
- **The Fix — Option B:** **Aurora Global Database** replicates to eu-west-1 with **sub-second** lag (fast local reads) and supports **cross-region failover promotion in under a minute** — satisfying both the latency and RTO requirements.

---

### Scenario 5

**The Situation:** A developer accidentally runs `DELETE FROM orders WHERE status='pending'` without a WHERE clause, deleting **all orders** from the production Aurora database. The mistake is discovered 30 minutes later. The business needs the data back **immediately** with minimal downtime.

**The Options:**

A. Restore from the latest automated backup (taken 6 hours ago).

B. Restore from a manual snapshot (taken yesterday).

C. Use Aurora Backtrack to rewind the database to 30 minutes ago.

D. Contact AWS Support.

**The Logic:**

- **Trap A — Restore latest automated backup:** Restores to 6 hours ago — loses ~5.5 hours of legitimate orders, and restoring spins up a *new* instance (downtime to repoint the app). Recovers the data, badly.
- **Trap B — Restore manual snapshot from yesterday:** Even worse — loses ~24 hours of data. Same new-instance downtime.
- **Trap D — Contact AWS Support:** Support cannot recover data you deleted yourself — AWS has no magic undo. Wastes the time the business doesn't have.
- **The Fix — Option C:** **Aurora Backtrack** rewinds the *existing* cluster to a chosen point (30 min ago, just before the DELETE) in **seconds** — no new instance, no data loss, app simply reconnects. Caveat: Backtrack must have been enabled on the cluster (retention up to 72h); the scenario assumes it was.
