# Databases (NoSQL & Caching)

> **Start here — what is "NoSQL", and why not just always use SQL?**
> A relational (SQL) database is great, but it has limits: a *fixed* table structure, and it gets hard to scale to truly massive size. **NoSQL** databases drop the rigid table model in exchange for **enormous scale, flexible structure, and consistent fast latency**. The trade-off: no complex joins, fewer guarantees. "NoSQL" really means "not the traditional relational model" — there are several shapes (key-value, document, graph, etc.).
>
> **This chapter has three parts, and they solve three different problems:**
> 1. **DynamoDB** — the star. A key-value/document database that scales essentially infinitely with single-digit-millisecond latency. *Learn this one deepest — it's the most tested.*
> 2. **ElastiCache** — not really a database; an in-memory **cache** you put *in front of* a database to make reads lightning fast.
> 3. **Purpose-built databases** — a small zoo of specialized databases (graph, time-series, ledger…). For these, you don't go deep — you just learn **which one matches which shape of data**.

---

### **SECTION 1: DYNAMODB (THE SERVERLESS DATABASE)**

> **What is DynamoDB?** A fully-managed NoSQL database where you store **items** (records) and look them up by a **key** — like a giant, infinitely-scalable dictionary/lookup-table. "Serverless" here means there's no server for you to size or manage *at all* — you just create a table and use it; AWS scales it silently from tiny to trillions of requests. You pick it when you need **massive scale + predictable fast latency** and your access pattern is simple key lookups (not complex SQL joins).

### **1. DynamoDB Overview**

- **The Rule:** Fully managed, serverless NoSQL — key-value and document (JSON), flexible schema, millisecond latency at any scale. Auto-scales to trillions of requests/day; data stored across **3 AZs**.
- **Core Concepts:** Table = collection of items; Item = a record (max **400 KB**); Attribute = a field.
- **Primary Key:** Either a **Partition Key** alone (e.g., `UserID`) or a **Composite Key** = Partition Key + Sort Key (e.g., `UserID` + `Timestamp`).

---

### **2. Read/Write Capacity Modes**

### **A. Provisioned Mode (The Planned Capacity)**

- Pre-allocate capacity (pay even if unused), with optional auto-scaling and limited burst buffer. Exceeding capacity → `ProvisionedThroughputExceededException`.
- **RCU:** 1 RCU = 1 strongly consistent read/sec of 4 KB (or 2 eventually consistent reads). **WCU:** 1 WCU = 1 write/sec of 1 KB.
- **Calc examples:** Read 10 items/sec × 8 KB strongly consistent → 2 RCU each → **20 RCUs**. Write 5 items/sec × 3 KB → 3 WCU each → **15 WCUs**.

### **B. On-Demand Mode (The Serverless)**

- No capacity planning, pay per request, auto-scales instantly. Pricier per request. Use for unpredictable workloads, new apps, spiky traffic.

**Exam Trigger:** "Unknown or variable workload" → On-Demand Mode.

---

### **3. Consistency Models**

- **Eventually Consistent Reads (Default):** May be stale (AZ replication lag); **50% cheaper** (2x RCU efficiency).
- **Strongly Consistent Reads:** Always latest data; higher cost (1 RCU = 4 KB); set `ConsistentRead=true`.

**Exam Trigger:** "Must always read latest data" → Strongly Consistent Reads.

---

### **4. Indexes (Querying Beyond Primary Key)**

> **The problem in plain terms:** DynamoDB is a lookup-table — it's fast *only* when you search by the **key** you designed it around. If your table's key is `UserID`, you can instantly find a user by ID — but you **cannot** efficiently ask "find the user with this *email*", because email isn't the key. Your only option would be to scan the *entire* table. Slow and expensive.
> **The fix:** An **index** — a second way into the same data, organized by a *different* attribute. There are two kinds (GSI and LSI); the table below contrasts them. The one-line difference: a **GSI** lets you search by a completely different key and can be added anytime; an **LSI** only gives a different *sort* order and must be set up when the table is created.

**Problem:** Queries require the Partition Key — indexes let you query by other attributes.

### **A. Global Secondary Index (GSI)**

- **Alternate Primary Key** (different PK and/or Sort Key), spans the **entire table**. Projection = ALL/KEYS_ONLY/INCLUDE.
- Has **its own** RCU/WCU; max 20 per table; reads are **always eventually consistent**; can be created **after** table creation.
- *Example:* Base PK `UserID` + Sort `Timestamp` → add GSI with PK `Email` to query by email.

### **B. Local Secondary Index (LSI)**

- **Same Partition Key** as base table, **different Sort Key**; scoped to a single partition. Shares RCU/WCU; max 5 per table; supports strongly consistent reads; must be created **at table creation**.

**Exam Contrast: GSI vs. LSI**

| **Feature** | **GSI** | **LSI** |
| --- | --- | --- |
| **Partition Key** | Different from base table | **Same as base table** |
| **Sort Key** | Different | Different |
| **Capacity** | **Own RCU/WCU** | Shares with base table |
| **Consistency** | Eventually consistent only | Strongly or Eventually |
| **Creation** | Anytime | **Only at table creation** |
| **Limit** | 20 per table | 5 per table |

**Exam Trigger:** "Query by different attribute" → GSI. "Query with same PK but different sort" → LSI.

---

### **5. DynamoDB Streams**

**The Rule:** Captures a **time-ordered sequence** of item-level changes (INSERT, UPDATE, DELETE), retained **24 hours**, and can trigger **Lambda** for real-time processing.

- **Use Cases:** Welcome emails on insert, cross-table/region replication, audit logs, change analytics.
- **View Types:** `KEYS_ONLY`, `NEW_IMAGE`, `OLD_IMAGE`, `NEW_AND_OLD_IMAGES`.

**Exam Trigger:** "React to changes in DynamoDB" → DynamoDB Streams + Lambda.

---

### **6. DynamoDB Accelerator (DAX)**

**The Rule:** In-memory cache that sits in front of DynamoDB — reduces read latency from **milliseconds to microseconds**. Reads hit DAX first; writes are write-through to DynamoDB.

- Drop-in (minimal code changes); item cache + query cache; TTL default 5 min; multi-node cluster for HA (3+ recommended).

**Exam Trigger:** "Reduce DynamoDB read latency to microseconds" → DAX.

**DAX vs. ElastiCache:**

- **DAX:** Specifically for DynamoDB. Drop-in. Object-level cache.
- **ElastiCache:** General-purpose. Requires application code changes. Aggregation cache.

---

### **7. Global Tables**

**The Rule:** Multi-region, multi-active — all regions can read/write. Writes propagate to other regions typically < 1 sec; conflicts resolved Last-Writer-Wins (timestamp).

- **Requirements:** DynamoDB Streams enabled; On-Demand or Auto Scaling recommended.
- **Use Cases:** Global low-latency apps; multi-region active-active DR.

**Exam Trigger:** "Multi-region, low-latency reads and writes" → Global Tables.

---

### **8. Time to Live (TTL)**

**The Rule:** Auto-delete items after an expiration timestamp. Add a TTL attribute (**Unix epoch seconds**, not ms); DynamoDB deletes expired items within **48 hours**. Deletions consume no WCUs and appear in Streams if enabled.

- **Use Cases:** Session data, temporary logs, time-limited event data.

**Exam Trigger:** "Automatically delete old data" → TTL.

---

### **9. Backups**

### **A. On-Demand Backups**

- Manual full backups, retained until deleted. No performance/capacity impact. Restore creates a new table (same or cross-region).

### **B. Point-in-Time Recovery (PITR)**

- Continuous backups, restore to any second in the last **35 days**. Must be explicitly enabled; restore creates a new table. Protects against accidental deletes/updates.

**Exam Trigger:** "Restore to specific timestamp" → PITR.

---

### **10. Transactions**

**The Rule:** ACID transactions across **multiple items/tables**. `TransactWriteItems` = atomic write (all-or-nothing); `TransactGetItems` = atomic strongly-consistent read.

- **Cost:** 2x RCU/WCU. **Limits:** max 100 items, 4 MB total per call.
- **Use Cases:** Financial transfers (debit + credit), order processing (inventory + order atomically).

**Exam Trigger:** "All-or-nothing operation across multiple items" → Transactions.

---

### **11. DynamoDB PartiQL**

- **What it does:** SQL-compatible query language for DynamoDB. Use familiar `SELECT`, `INSERT`, `UPDATE`, `DELETE` syntax instead of DynamoDB-specific API calls.
- **Supports:** Queries against tables and GSIs. Batch operations with PartiQL.
- **Not a replacement:** Under the hood, still uses the same RCU/WCU capacity. Just a different query interface.
- *Exam Trigger:* "SQL-like queries on DynamoDB" → PartiQL.

---

### **12. DynamoDB Export to S3**

**The Rule:** Export full table data to **S3** with no RCU consumption or performance impact. Requires PITR; exports DynamoDB JSON or Amazon Ion at any point within the 35-day PITR window. Query exports with Athena, EMR, or Redshift Spectrum.

- **Use Cases:** Analytics without impacting production, data lake ingestion, archival/compliance snapshots.

**Exam Trigger:** "Analyze DynamoDB data without impacting performance" or "Export DynamoDB to S3 without consuming RCUs" → DynamoDB Export to S3.

---

### **SECTION 2: ELASTICACHE (IN-MEMORY CACHE)**

> **What problem does a cache solve?** Every time your app asks the database for something, that query takes time and effort. If 10,000 users all request the *same* popular data, the database does the *same* work 10,000 times. Wasteful and slow.
> **The fix — a cache:** A small, blazing-fast store that keeps a copy of frequently-needed data **in memory (RAM)**. The app checks the cache first; if the data's there ("cache hit"), it's returned in *sub-millisecond* time and the database is never touched. ElastiCache is AWS's managed cache. **Important:** a cache sits *in front of* a real database — it usually isn't the permanent home of your data (the exception is MemoryDB, later).

### **1. ElastiCache Overview**

> 🔧 *Like:* managed Redis / Memcached — it *is* those, AWS just runs them for you.

- **The Rule:** Managed in-memory data store, **sub-millisecond latency**. Engines: **Redis** (feature-rich — persistence, replication, backup) and **Memcached** (simple, multi-threaded, no persistence).
- **Use Cases:** Cache query results, session storage, real-time analytics, leaderboards, rate limiting.

---

### **2. Redis vs. Memcached**

| **Feature** | **Redis** | **Memcached** |
| --- | --- | --- |
| **Data Structures** | Strings, Lists, Sets, Sorted Sets, Hashes, Bitmaps | **Strings only** |
| **Persistence** | **Yes** (RDB snapshots, AOF logs) | **No** (In-memory only) |
| **Replication** | **Yes** (Multi-AZ with failover) | **No** |
| **Backups** | **Yes** | **No** |
| **Transactions** | **Yes** | **No** |
| **Pub/Sub** | **Yes** | **No** |
| **Multi-Threaded** | **No** (Single-threaded per core) | **Yes** |
| **Sharding** | **Yes** (Cluster mode) | **Yes** |
| **Use Case** | Complex data, HA, Persistence | Simple caching, Multi-core performance |

**Exam Trigger:**

- "Need backups/persistence/replication" → Redis.
- "Simple cache, Multi-threaded" → Memcached.

---

### **3. Redis Cluster Mode**

### **A. Cluster Mode Disabled (Simple)**

- One primary + up to 5 replicas (async replication). Automatic failover (Multi-AZ). Scales **vertically only** (change node type) — no horizontal scaling.

### **B. Cluster Mode Enabled (Distributed)**

- Data **sharded** across multiple primaries (each shard = 1 primary + up to 5 replicas), up to **500 nodes** total. Scales **horizontally** for higher throughput and large datasets.

**Exam Trigger:** "Scale Redis horizontally" → Cluster Mode Enabled.

---

### **4. Caching Strategies**

### **A. Lazy Loading (Cache-Aside)**

- Cache hit → return; cache miss → fetch from DB, write to cache, return. **Pros:** only requested data cached (efficient memory). **Cons:** miss penalty on first request, possible stale data.

### **B. Write-Through**

- App writes to DB and cache simultaneously. **Pros:** cache always fresh, no stale reads. **Cons:** write penalty, caches data that may never be read.

**Exam Best Practice:** Combine Lazy Loading + Write-Through + TTL.

---

### **5. Redis Auth & Security**

- **Redis AUTH:** Password-based authentication (Set in Parameter Group).
- **In-Transit Encryption:** Enable TLS.
- **At-Rest Encryption:** Enable KMS encryption.
- **Network:** Deploy in **Private Subnet**. Use Security Groups.

**Exam Trigger:** "Secure Redis cluster" → AUTH token + Encryption + Private Subnet.

### **6. ElastiCache Global Datastore (Redis)**

- Cross-region Redis replication, active-passive with read-only secondary clusters serving local reads. < 1 sec cross-region latency. Promote a secondary region to primary for DR.
- *Exam Trigger:* "Cross-region Redis for DR or global reads" → ElastiCache Global Datastore.

---

### **SECTION 3: PURPOSE-BUILT DATABASES**

> **The idea:** RDS and DynamoDB cover most needs — but some data has a *special shape* that a general database handles awkwardly. A network of friends-of-friends, a stream of sensor readings every second, a tamper-proof audit log — each of these has a database **purpose-built** just for it.
> **How to study this section:** Do **not** memorize the internals. The exam gives you ~1 question each, and it always works the same way: it describes a *shape of data* and you match it to the right service. So learn only the **trigger → service** pair for each. The comparison table at the end of this section is the thing to memorize; everything above it is just context.

*AWS offers specialized databases for specific workloads. The exam tests whether you pick the right one.*

### **1. Amazon DocumentDB**

> 🔧 *Like:* MongoDB — MongoDB-compatible, AWS-managed.

- **The Rule:** Fully managed **MongoDB-compatible** document database. Scales up to millions of requests/sec.
- **Architecture:** Storage decoupled from compute, auto-scales up to **64 TB**. Replicates **6 copies across 3 AZs**.
- **Exam Trigger:** "MongoDB migration" or "MongoDB-compatible" → **DocumentDB**.

---

### **2. Amazon Neptune**

> 🔧 *Like:* Neo4j — a graph database (nodes & relationships).

- **The Rule:** Fully managed **graph database**. Supports Property Graph (Gremlin) and RDF (SPARQL).
- **Architecture:** Up to **15 read replicas**, Multi-AZ HA, storage auto-scales up to 64 TB.
- **Exam Trigger:** "Social network", "Fraud detection", "Recommendation engine", "Knowledge graph" → **Neptune**.

---

### **3. Amazon Timestream**

- **The Rule:** Fully managed **time-series database**. 1000x faster and 1/10th cost of relational databases for time-series.
- **Architecture:** Automatic tiering — recent data in memory, historical in magnetic storage. Built-in analytics functions.
- **Exam Trigger:** "IoT sensor data", "DevOps metrics", "Time-series data" → **Timestream**.

---

### **4. Amazon QLDB (Quantum Ledger Database)**

- **The Rule:** Fully managed **immutable ledger**. Cryptographically verifiable transaction log (Uses SHA-256 hash chain).
- **Key Distinction:** Centralized (Owned by one authority). **NOT blockchain** — no decentralization or multi-party consensus.
- **Exam Trigger:** "Immutable ledger", "Financial transactions audit trail", "Cryptographic verification" → **QLDB**.

---

### **5. Amazon Keyspaces**

> 🔧 *Like:* Apache Cassandra — Cassandra-compatible, AWS-managed.

- **The Rule:** Fully managed **Apache Cassandra-compatible** database. Serverless, pay-per-use.
- **Architecture:** Tables replicated **3x across multiple AZs**. On-demand or provisioned capacity (Like DynamoDB).
- **Exam Trigger:** "Cassandra migration" or "Cassandra-compatible" → **Keyspaces**.

---

### **6. Amazon MemoryDB for Redis**

- **The Rule:** Durable, **Redis-compatible** in-memory database. Multi-AZ durability with transaction log.
- **Key Distinction:** Use as a **primary database** (Durable). ElastiCache Redis = **cache layer** (May lose data). MemoryDB = microsecond reads + single-digit millisecond writes with durability.
- **Exam Trigger:** "Durable in-memory database" or "Redis with durability as primary database" → **MemoryDB**.

---

### **Purpose-Built Database Comparison**

| **Service** | **Use Case** | **Exam Trigger** |
| --- | --- | --- |
| **DocumentDB** | Document DB (MongoDB workloads) | "MongoDB migration/compatible" |
| **Neptune** | Graph relationships | "Social network, Fraud detection, Recommendations" |
| **Timestream** | Time-series data | "IoT sensors, DevOps metrics, Time-series" |
| **QLDB** | Immutable ledger | "Audit trail, Immutable, Cryptographic verification" |
| **Keyspaces** | Wide-column (Cassandra workloads) | "Cassandra migration/compatible" |
| **MemoryDB** | Durable in-memory primary DB | "Redis with durability, Durable in-memory" |

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **Unpredictable DynamoDB workload?** → On-Demand Mode.
2. **Query DynamoDB by non-PK attribute?** → GSI.
3. **Must read latest data from DynamoDB?** → Strongly Consistent Reads.
4. **Reduce DynamoDB latency to microseconds?** → DAX.
5. **React to DynamoDB changes?** → DynamoDB Streams + Lambda.
6. **Multi-region DynamoDB with read/write?** → Global Tables.
7. **Auto-delete old DynamoDB items?** → TTL.
8. **Atomic operation across multiple items?** → Transactions.
9. **Analyze DynamoDB data without consuming RCUs?** → Export to S3.
10. **Cache with backups/persistence?** → Redis.
11. **Simple multi-threaded cache?** → Memcached.
12. **Scale Redis horizontally?** → Cluster Mode Enabled.
13. **Session storage for stateless app?** → ElastiCache (Redis or Memcached).
14. **MongoDB migration?** → DocumentDB.
15. **Social network / Fraud detection / Recommendations?** → Neptune.
16. **IoT sensor data / Time-series?** → Timestream.
17. **Immutable ledger / Audit trail?** → QLDB (Not blockchain).
18. **Cassandra migration?** → Keyspaces.
19. **Durable in-memory database (not just cache)?** → MemoryDB for Redis.
20. **SQL-like queries on DynamoDB?** → PartiQL.
21. **Cross-region Redis for DR/global reads?** → ElastiCache Global Datastore.

---

# **REAL EXAM SCENARIOS**

### Scenario 1

**The Situation:** A mobile gaming application uses DynamoDB to store player profiles and game state. The read latency is currently 5-10 milliseconds, but the product team wants to improve user experience by reducing latency to **under 1 millisecond**. Read traffic is 10x higher than write traffic.

**The Options:**

A. Switch to Provisioned Capacity Mode with higher RCUs.

B. Deploy DynamoDB Accelerator (DAX) in front of the table.

C. Use ElastiCache Redis as a cache.

D. Enable DynamoDB Global Tables.

**The Logic:**

- **Trap A — More RCUs:** RCUs control *throughput* (how many reads/sec), not *latency*. Even with unlimited RCUs, DynamoDB's native read latency stays at single-digit milliseconds. Can't reach sub-millisecond.
- **Trap C — ElastiCache Redis:** Would deliver the speed, but it's a *generic* cache — you must write application logic for cache population, invalidation, and miss handling. Significant code change.
- **Trap D — Global Tables:** Global Tables replicate the table to other **regions** for multi-region access/DR. They don't reduce *read latency* within a region. Wrong feature.
- **The Fix — Option B:** **DAX** is an in-memory cache **purpose-built for DynamoDB** — sits in front of the table, returns cached reads in **microseconds**, and is nearly drop-in (point the SDK at the DAX endpoint). Read-heavy workload + microsecond target = DAX.

---

### Scenario 2

**The Situation:** A DynamoDB table stores user data with `UserID` (Partition Key) and `RegistrationDate` (Sort Key). The application now needs to query users by their **email address**. Currently, scanning the entire table is too slow and expensive.

**The Options:**

A. Add a Local Secondary Index (LSI) with Email as Sort Key.

B. Create a Global Secondary Index (GSI) with Email as Partition Key.

C. Switch to RDS to support flexible querying.

D. Use DynamoDB Streams to maintain a separate table indexed by email.

**The Logic:**

- **Trap A — LSI with Email as Sort Key:** An LSI must reuse the **base table's Partition Key** (`UserID`). To query by email alone you'd need `UserID` too — which you don't have. Also, LSIs can only be created **at table creation**, not added later.
- **Trap C — Switch to RDS:** Migrating to a relational engine just to gain a query pattern is over-engineering. DynamoDB already solves this with a GSI.
- **Trap D — Streams to a separate email-indexed table:** This *works* but it's a manual, custom-built secondary index — you'd own the Lambda, the consistency, and the second table. A GSI does exactly this, natively and managed.
- **The Fix — Option B:** A **GSI** with `Email` as its Partition Key gives an alternate key structure for efficient email queries. A GSI **can be added after table creation** and has its own RCU/WCU. The native, intended solution.

---

### Scenario 3

**The Situation:** A web application needs to store user session data for 30 minutes. The application runs on an Auto Scaling Group across multiple AZs. Session data must be shared across all instances. If the cache cluster fails, session data must be recoverable.

**The Options:**

A. Use ElastiCache Memcached.

B. Use ElastiCache Redis with Multi-AZ and AOF persistence.

C. Store sessions in DynamoDB.

D. Store sessions on EC2 instance storage.

**The Logic:**

- **Trap A — ElastiCache Memcached:** Memcached has **no persistence and no replication**. If the cluster fails, every session is lost — directly violating the "must be recoverable" requirement.
- **Trap C — DynamoDB:** Genuinely works as a session store (shared, durable, TTL support). The reason it's not the best answer here: the scenario points at a *cache cluster* and sub-millisecond access — Redis is the more precise fit. On a real exam, if Redis isn't offered, DynamoDB is a correct alternative.
- **Trap D — EC2 instance storage:** Local instance store is **not shared** across the ASG and is **ephemeral**. Other instances can't see the session, and it's lost on stop. Fails both requirements.
- **The Fix — Option B:** **ElastiCache Redis with Multi-AZ + persistence (AOF/RDB)** gives shared, sub-millisecond session access, HA replication, and recoverability if the cluster fails. Set a 30-minute TTL to auto-expire sessions.

---

### Scenario 4

**The Situation:** An e-commerce application stores orders in DynamoDB. When a new order is created, the system must send a confirmation email to the customer and update inventory in a separate table. This must happen **in real-time** without polling.

**The Options:**

A. Run a cron job every minute to scan the Orders table for new items.

B. Enable DynamoDB Streams and trigger a Lambda function on INSERT events.

C. Use SQS to poll DynamoDB.

D. Use CloudWatch Events to monitor the table.

**The Logic:**

- **Trap A — Cron job scanning the table:** Polling = not real-time (up to a minute late) and a full table Scan every minute is slow and expensive. The question explicitly says "without polling."
- **Trap C — SQS polling DynamoDB:** SQS cannot "poll" a database — there's no mechanism for SQS to read DynamoDB. The data flow is backwards; not a real option.
- **Trap D — CloudWatch Events:** CloudWatch Events/EventBridge does not natively emit an event for an individual DynamoDB *item* change. It can't see row-level INSERTs.
- **The Fix — Option B:** **DynamoDB Streams** captures every INSERT/UPDATE/DELETE as an event. A **Lambda triggered on the stream** fires the moment an order is inserted — sends the email and updates inventory. Real-time, event-driven, no polling.

---

### Scenario 5

**The Situation:** A DynamoDB table has **1000 WCUs** provisioned. A Global Secondary Index (GSI) on the table has **50 WCUs** provisioned. During peak hours, write operations fail with `ProvisionedThroughputExceededException`, but the base table's write metrics show only 40% utilization.

**The Options:**

A. Increase the base table's WCU to 2000.

B. Increase the GSI's WCU to match expected write load.

C. Switch the table to On-Demand mode.

D. Remove the GSI.

**The Logic:**

- **Key Fact:** Every write to the base table is **also written to each GSI**. If a GSI runs out of WCUs, DynamoDB **throttles the base table write too** — which is why the base table shows only 40% use yet writes still fail. The GSI (50 WCU) is the real bottleneck.
- **Trap A — Increase base table WCU to 2000:** The base table isn't the constraint (it's at 40%). Doubling its capacity spends money and changes nothing — the GSI still throttles.
- **Trap D — Remove the GSI:** This stops the throttling but **destroys the query capability** the GSI provides. Solving a capacity problem by deleting a feature is wrong.
- **The Fix — Option B (or C):** **Raise the GSI's WCUs** to match the write load — a GSI's capacity must meet or exceed the base table's write rate. Alternatively, **switch the table to On-Demand mode** (Option C), which removes capacity planning entirely and is also a correct answer.
