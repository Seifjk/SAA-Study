# Databases (NoSQL & Caching)

### **SECTION 1: DYNAMODB (THE SERVERLESS DATABASE)**

*AWS-managed NoSQL key-value and document database.*

### **1. DynamoDB Overview**

- **The Rule:** Fully managed, serverless NoSQL. Millisecond latency at any scale.
- **Data Model:** Key-Value and Document (JSON).
- **Schema:** Flexible (No fixed schema).
- **Scale:** Automatically scales to handle trillions of requests per day.
- **Replication:** Data stored across **3 AZs** automatically.

**Core Concepts:**

- **Table:** Collection of items (like rows).
- **Item:** Single record (like a row). Max size: **400 KB**.
- **Attribute:** Field in an item (like a column).
- **Primary Key:** Uniquely identifies each item. Two types:
    - **Partition Key (PK):** Single attribute (e.g., `UserID`).
    - **Composite Key (PK + Sort Key):** Two attributes (e.g., `UserID` + `Timestamp`).

---

### **2. Read/Write Capacity Modes**

### **A. Provisioned Mode (The Planned Capacity)**

- **The Rule:** Pre-allocate read/write capacity.
- **Units:**
    - **RCU (Read Capacity Unit):** 1 RCU = 1 strongly consistent read per second of 4 KB (or 2 eventually consistent reads).
    - **WCU (Write Capacity Unit):** 1 WCU = 1 write per second of 1 KB.
- **Auto Scaling:** Can enable auto-scaling to adjust based on load.
- **Cost:** Pay for provisioned capacity (even if unused).
- **Burst Capacity:** Short bursts above provisioned allowed (Limited buffer).
- **Throttling:** If you exceed capacity → `ProvisionedThroughputExceededException`.

**Calculation Examples:**

- **Example 1:** Read 10 items per second, each 8 KB, strongly consistent.
    - 8 KB / 4 KB = 2 RCUs per item.
    - 10 items * 2 = **20 RCUs**.
- **Example 2:** Write 5 items per second, each 3 KB.
    - 3 KB rounds up to 3 WCUs per item.
    - 5 items * 3 = **15 WCUs**.

### **B. On-Demand Mode (The Serverless)**

- **The Rule:** No capacity planning. Pay per request.
- **Scaling:** Auto-scales instantly to handle any load.
- **Cost:** More expensive per request, but no upfront capacity planning.
- **Use Case:** Unpredictable workloads, New applications, Spiky traffic.

**Exam Trigger:** "Unknown or variable workload" → On-Demand Mode.

---

### **3. Consistency Models**

- **Eventually Consistent Reads (Default):**
    - Data may be stale (Replication lag across AZs).
    - **50% cheaper** than strongly consistent reads (2x RCU efficiency).
- **Strongly Consistent Reads:**
    - Always returns latest data.
    - Higher cost (1 RCU = 4 KB).
    - Use `ConsistentRead=true` in API call.

**Exam Trigger:** "Must always read latest data" → Strongly Consistent Reads.

---

### **4. Indexes (Querying Beyond Primary Key)**

**Problem:** DynamoDB queries require Partition Key. What if you want to query by another attribute?

**Solution:** Indexes.

### **A. Global Secondary Index (GSI)**

- **The Rule:** Create an **alternate Primary Key** (Different PK and/or Sort Key).
- **Scope:** Spans **entire table**.
- **Projection:** Choose which attributes to copy to index (ALL, KEYS_ONLY, INCLUDE).
- **Capacity:** GSI has **its own** RCU/WCU (Separate from base table).
- **Limit:** 20 GSIs per table.
- **Eventual Consistency:** GSI reads are **always eventually consistent** (Cannot be strongly consistent).
- **Creation:** Can create GSI **after** table creation.

**Example:** Base table PK = `UserID`, Sort Key = `Timestamp`. Create GSI with PK = `Email` to query by email.

### **B. Local Secondary Index (LSI)**

- **The Rule:** **Same Partition Key** as base table, but **different Sort Key**.
- **Scope:** Only within the **same partition**.
- **Capacity:** Shares RCU/WCU with base table.
- **Limit:** 5 LSIs per table.
- **Consistency:** Can use **strongly consistent** or eventually consistent reads.
- **Creation:** Must create LSI **at table creation** (Cannot add later).

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

**The Rule:** Captures **time-ordered sequence** of item-level changes (INSERT, UPDATE, DELETE).

**How It Works:**

- Change happens in table → Event written to Stream (Retained for **24 hours**).
- Stream can trigger **Lambda** for real-time processing.

**Use Cases:**

- Send welcome email when new user inserted.
- Replicate data to another table/region.
- Audit log of changes.
- Analytics on change patterns.

**Stream View Types:**

- `KEYS_ONLY`: Only PK and Sort Key.
- `NEW_IMAGE`: Entire item after change.
- `OLD_IMAGE`: Entire item before change.
- `NEW_AND_OLD_IMAGES`: Both before and after.

**Exam Trigger:** "React to changes in DynamoDB" → DynamoDB Streams + Lambda.

---

### **6. DynamoDB Accelerator (DAX)**

**The Rule:** In-memory cache for DynamoDB. **Microsecond latency**.

**How It Works:**

- DAX sits in front of DynamoDB.
- Read requests hit DAX first (If cached → Return immediately, else → Fetch from DynamoDB + Cache).
- Write requests go through DAX to DynamoDB (Write-through cache).

**Features:**

- **Latency:** Reduces read latency from **milliseconds** to **microseconds**.
- **Compatibility:** Drop-in replacement (Minimal code changes).
- **Caching:** Item cache + Query cache.
- **TTL:** Default 5 minutes (Configurable).
- **Cluster:** Multi-node for HA (3+ nodes recommended).

**Exam Trigger:** "Reduce DynamoDB read latency to microseconds" → DAX.

**DAX vs. ElastiCache:**

- **DAX:** Specifically for DynamoDB. Drop-in. Object-level cache.
- **ElastiCache:** General-purpose. Requires application code changes. Aggregation cache.

---

### **7. Global Tables**

**The Rule:** Multi-region, multi-active (All regions can read/write).

**How It Works:**

- Data replicated across selected regions.
- Writes in any region automatically propagate to others (Typically < 1 second).
- Conflict resolution: Last Writer Wins (Based on timestamp).

**Requirements:**

- **DynamoDB Streams must be enabled**.
- **On-Demand or Auto Scaling** recommended for capacity.

**Use Cases:**

- Global applications (Low-latency reads/writes for users worldwide).
- Disaster Recovery (Multi-region active-active).

**Exam Trigger:** "Multi-region, low-latency reads and writes" → Global Tables.

---

### **8. Time to Live (TTL)**

**The Rule:** Automatically delete items after expiration timestamp.

**How It Works:**

- Add a TTL attribute (Unix epoch timestamp) to items.
- DynamoDB deletes expired items within **48 hours** (Background process).
- Deletions don't consume WCUs.

**Use Cases:**

- Session data (Auto-expire after 24 hours).
- Temporary logs.
- Event data (Keep only last 30 days).

**Exam Trigger:** "Automatically delete old data" → TTL.

---

### **9. Backups**

### **A. On-Demand Backups**

- **The Rule:** Manual backups. Retained until deleted.
- **Process:** Full backup. Does not affect performance or consume RCU/WCU.
- **Restore:** Creates new table (Same region or cross-region).

### **B. Point-in-Time Recovery (PITR)**

- **The Rule:** Continuous backups. Restore to any second in last **35 days**.
- **Enable:** Must be explicitly enabled.
- **Restore:** Creates new table.
- **Use Case:** Protect against accidental deletes/updates.

**Exam Trigger:** "Restore to specific timestamp" → PITR.

---

### **10. Transactions**

**The Rule:** ACID transactions across **multiple items/tables**.

**APIs:**

- `TransactWriteItems`: Atomic write (All succeed or all fail).
- `TransactGetItems`: Atomic read (Strongly consistent).

**Cost:** Consumes **2x** RCU/WCU.

**Use Cases:**

- Financial transactions (Debit one account, Credit another).
- Order processing (Update inventory + Create order atomically).

**Exam Trigger:** "All-or-nothing operation across multiple items" → Transactions.

---

### **SECTION 2: ELASTICACHE (IN-MEMORY CACHE)**

*Managed Redis and Memcached.*

### **1. ElastiCache Overview**

- **The Rule:** Managed in-memory data store. **Sub-millisecond latency**.
- **Engines:**
    - **Redis:** Feature-rich. Persistence, Replication, Backup.
    - **Memcached:** Simple. Multi-threaded. No persistence.

**Use Cases:**

- Cache database query results.
- Session storage.
- Real-time analytics.
- Leaderboards, Rate limiting.

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

### **A. Cluster Mode Disabled (The Simple Setup)**

- **The Rule:** One primary node + Up to 5 replicas.
- **Replication:** Async replication to replicas.
- **Failover:** Automatic promotion of replica to primary (Multi-AZ).
- **Scaling:** **Vertical only** (Change node type). Cannot scale horizontally.

### **B. Cluster Mode Enabled (The Distributed)**

- **The Rule:** Data **sharded** across multiple primary nodes (Each shard = 1 primary + up to 5 replicas).
- **Shards:** Up to **500 nodes** total (90 shards * 5 replicas + 90 primaries).
- **Scaling:** **Horizontal** (Add/remove shards).
- **Performance:** Higher throughput.
- **Use Case:** Large datasets, High throughput.

**Exam Trigger:** "Scale Redis horizontally" → Cluster Mode Enabled.

---

### **4. Caching Strategies**

### **A. Lazy Loading (Cache-Aside)**

- **Flow:**
    1. App requests data from cache.
    2. If **Cache Hit** → Return data.
    3. If **Cache Miss** → Fetch from DB → Write to cache → Return data.
- **Pros:** Only requested data is cached (Efficient memory use).
- **Cons:** First request is slow (Miss penalty). Stale data possible.

### **B. Write-Through**

- **Flow:**
    1. App writes data to DB.
    2. App writes data to cache (Simultaneously).
- **Pros:** Data always fresh in cache. No stale reads.
- **Cons:** Every write hits cache (Even if data never read). Write penalty.

**Exam Best Practice:** Combine Lazy Loading + Write-Through + TTL.

---

### **5. Redis Auth & Security**

- **Redis AUTH:** Password-based authentication (Set in Parameter Group).
- **In-Transit Encryption:** Enable TLS.
- **At-Rest Encryption:** Enable KMS encryption.
- **Network:** Deploy in **Private Subnet**. Use Security Groups.

**Exam Trigger:** "Secure Redis cluster" → AUTH token + Encryption + Private Subnet.

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
9. **Cache with backups/persistence?** → Redis.
10. **Simple multi-threaded cache?** → Memcached.
11. **Scale Redis horizontally?** → Cluster Mode Enabled.
12. **Session storage for stateless app?** → ElastiCache (Redis or Memcached).

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Microsecond Latency" (DAX)**

**The Situation:** A mobile gaming application uses DynamoDB to store player profiles and game state. The read latency is currently 5-10 milliseconds, but the product team wants to improve user experience by reducing latency to **under 1 millisecond**. Read traffic is 10x higher than write traffic.

**The Options:**

A. Switch to Provisioned Capacity Mode with higher RCUs.

B. Deploy DynamoDB Accelerator (DAX) in front of the table.

C. Use ElastiCache Redis as a cache.

D. Enable DynamoDB Global Tables.

**The Logic:**

- **Trap:** Higher RCUs (Option A) don't reduce latency below DynamoDB's native millisecond level.
- **Trap:** ElastiCache (Option C) works but requires significant application code changes for cache management.
- **The Fix:** **Option B**. **DAX** is a drop-in in-memory cache specifically for DynamoDB. Reduces latency to **microseconds** with minimal code changes (Just change endpoint). Supports both item cache and query cache.

---

### **Scenario 2: The "Email Query" (GSI)**

**The Situation:** A DynamoDB table stores user data with `UserID` (Partition Key) and `RegistrationDate` (Sort Key). The application now needs to query users by their **email address**. Currently, scanning the entire table is too slow and expensive.

**The Options:**

A. Add a Local Secondary Index (LSI) with Email as Sort Key.

B. Create a Global Secondary Index (GSI) with Email as Partition Key.

C. Switch to RDS to support flexible querying.

D. Use DynamoDB Streams to maintain a separate table indexed by email.

**The Logic:**

- **Trap:** LSI (Option A) requires the **same Partition Key** as base table (UserID). Can't query by email alone.
- **Trap:** RDS (Option C) is over-engineering for this use case.
- **The Fix:** **Option B**. **GSI** creates an alternate key structure. Create GSI with `Email` as Partition Key. Now can efficiently query by email. GSI has its own RCU/WCU. Can be created after table creation.

---

### **Scenario 3: The "Session Store" (Redis)**

**The Situation:** A web application needs to store user session data for 30 minutes. The application runs on an Auto Scaling Group across multiple AZs. Session data must be shared across all instances. If the cache cluster fails, session data must be recoverable.

**The Options:**

A. Use ElastiCache Memcached.

B. Use ElastiCache Redis with Multi-AZ and AOF persistence.

C. Store sessions in DynamoDB.

D. Store sessions on EC2 instance storage.

**The Logic:**

- **Trap:** Memcached (Option A) has **no persistence**. If cluster fails, all sessions lost.
- **Trap:** Instance storage (Option D) is not shared across instances.
- **Trap:** DynamoDB (Option C) works but has higher latency (milliseconds vs. sub-millisecond).
- **The Fix:** **Option B**. **Redis** supports **persistence** (AOF/RDB), **Multi-AZ replication** (HA), and **sub-millisecond latency**. Perfect for session storage with durability requirements. Set TTL to 30 minutes to auto-expire sessions.

---

### **Scenario 4: The "Real-Time Notifications" (DynamoDB Streams)**

**The Situation:** An e-commerce application stores orders in DynamoDB. When a new order is created, the system must send a confirmation email to the customer and update inventory in a separate table. This must happen **in real-time** without polling.

**The Options:**

A. Run a cron job every minute to scan the Orders table for new items.

B. Enable DynamoDB Streams and trigger a Lambda function on INSERT events.

C. Use SQS to poll DynamoDB.

D. Use CloudWatch Events to monitor the table.

**The Logic:**

- **Trap:** Cron job (Option A) is not real-time (1-minute delay) and inefficient (Scans entire table).
- **Trap:** CloudWatch Events (Option D) doesn't natively monitor DynamoDB item changes.
- **The Fix:** **Option B**. **DynamoDB Streams** captures every INSERT, UPDATE, DELETE event. Configure Lambda to trigger on Stream events. Lambda processes new order (Send email, Update inventory). Real-time, event-driven, efficient.

---

### **Scenario 5: The "Throttling Hell" (GSI Capacity)**

**The Situation:** A DynamoDB table has **1000 WCUs** provisioned. A Global Secondary Index (GSI) on the table has **50 WCUs** provisioned. During peak hours, write operations fail with `ProvisionedThroughputExceededException`, but the base table's write metrics show only 40% utilization.

**The Options:**

A. Increase the base table's WCU to 2000.

B. Increase the GSI's WCU to match expected write load.

C. Switch the table to On-Demand mode.

D. Remove the GSI.

**The Logic:**

- **Trap:** Increasing base table WCU (Option A) doesn't help. The **GSI is the bottleneck**.
- **Key Fact:** When you write to base table, DynamoDB also writes to all GSIs. If GSI lacks capacity, the **base table write is throttled** too.
- **The Fix:** **Option B** or **Option C**. Increase GSI's WCU to handle write load, or switch entire table to On-Demand mode (Eliminates capacity planning). GSI capacity must match or exceed write load to avoid throttling the base table.
