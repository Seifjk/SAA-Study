# SAA-C03 — Daily Recall

> Cover the right column with your hand. Read the trigger, say the answer, slide your hand down. Whole sheet in 20-30 min. Mark anything you missed and re-read that chapter's section tomorrow.
>
> Covers: **Networking · Compute · HA · Storage · DB Relational · DB NoSQL**. Update as new chapters land.

---

## 1. Networking

| Trigger | Answer |
|---|---|
| Block a specific IP | **NACL** (SG is allow-only) |
| Private S3 access, free | **Gateway Endpoint** |
| Private Kinesis/SQS/CloudWatch access | **Interface Endpoint** (PrivateLink, hourly $) |
| Connect 2 VPCs | **VPC Peering** (no CIDR overlap, not transitive) |
| Connect 50 VPCs | **Transit Gateway** (hub-and-spoke, transitive) |
| Private instances → outbound internet | **NAT Gateway** in public subnet |
| Bastion Host | Public-subnet EC2 for **inbound SSH jump**; SG = your IP only |
| Most secure private-instance access (no SSH) | **SSM Session Manager** (no open ports, IAM, logged) |
| Expose your service to other VPCs privately | **NLB + PrivateLink (Endpoint Service)** |
| Encrypt Direct Connect | **Site-to-Site VPN over DX** (DX is private, NOT encrypted) |
| Max DX resiliency | **2 connections × 2 locations = 4 total** |
| IPv6 outbound only | **Egress-Only Internet Gateway** |
| Deep packet inspection at VPC level | **AWS Network Firewall** (Flow Logs = metadata only) |
| Increase VPN bandwidth | **ECMP via Transit Gateway** |
| Hybrid DNS (on-prem ↔ AWS) | **Route 53 Resolver** (Inbound = on-prem→AWS, Outbound = AWS→on-prem) |
| Share subnets across accounts | **RAM** (Resource Access Manager) |
| Remote user VPN to AWS | **Client VPN** (S2S = network↔network, Client = user↔AWS) |
| Custom NACL setup | Deny-all by default; remember **ephemeral ports 1024-65535** outbound |

---

## 2. Compute

| Trigger | Answer |
|---|---|
| Lowest latency between instances | **Cluster Placement Group** |
| Maximum fault isolation | **Spread PG** (≤7/AZ) or **Partition PG** for HDFS/Kafka |
| Cheapest for fault-tolerant batch | **Spot** (~70-90% off) |
| T2/T3 slow during business hours | Credits exhausted → **T2/T3 Unlimited** or move to **M5** |
| Per-socket / BYOL licensing | **Dedicated Hosts** |
| HPC sub-millisecond network | **EFA** (Elastic Fabric Adapter — *network*, not storage) |
| Preserve private IP on failover | **Attach ENI** to new instance |
| Resume app with RAM intact | **Hibernate** |
| Auto-install software at launch | **User Data** script |
| EC2 → S3 without hardcoded keys | **IAM Instance Profile** |
| Guarantee capacity in an AZ | **Capacity Reservation** (not a discount) |
| Static public IP across stop/start | **Elastic IP** |
| Prevent SSRF / metadata theft | **IMDSv2** (token-based) |
| SSH without key pairs | **EC2 Instance Connect** |
| ASG "launch config"? | **Launch Template** (Launch Configuration is legacy) |

**Launch Template vs User Data vs Lifecycle Hook — the one that trips you:**

| Trigger | Answer |
|---|---|
| Define **what** the instance looks like (AMI, type, SG, IAM role, contains user data) | **Launch Template** |
| Run a script the **first time** the instance boots (install Apache, register service) | **User Data** (a field *inside* the Launch Template) |
| **Pause** the instance during scale-out/in to do work (drain conns, copy logs, custom warm-up) | **Lifecycle Hook** |
| "Copy logs before termination" | **Lifecycle Hook** (terminating) |
| "Pre-warm cache before serving traffic" | **Lifecycle Hook** (pending — user data already finished) |
| "Install software on boot" | **User Data** |
| "Versioned instance config" / "standardize across ASG" | **Launch Template** |

*Mental model: **Launch Template** = the recipe · **User Data** = a step inside the recipe that runs at first boot · **Lifecycle Hook** = ASG hits pause so you can act before the instance joins or leaves.*

---

## 3. High Availability

| Trigger | Answer |
|---|---|
| Route based on URL path / host | **ALB** |
| Static IP, extreme TPS, TCP/UDP | **NLB** |
| 3rd-party firewall traffic inspection (GENEVE) | **GWLB** |
| Invoke Lambda from HTTP | **ALB** |
| Stateful app on multiple instances | **ALB + Stickiness** |
| Maintain CPU at 60% | **Target Tracking Scaling** |
| Scale every Monday 9 AM | **Scheduled Scaling** |
| Scale before a predicted spike | **Predictive Scaling** |
| Let in-flight requests finish before terminate | **Connection Draining** (Deregistration Delay) |
| Scale workers based on queue depth | **SQS `ApproximateNumberOfMessagesVisible` + Target Tracking** |
| App behind ALB needs original client IP | **`X-Forwarded-For`** header |
| Roll out new AMI across ASG without downtime | **Instance Refresh** (set min healthy %) |
| NLB cross-zone — free? | **No** (NLB inter-AZ is **paid**; ALB cross-zone is free) |
| Authenticate users at the LB | **ALB + OIDC/Cognito** |
| New instance fails health checks at startup (slow health check) | **Increase Health Check Grace Period** |
| App takes 8 min to boot, scales too slow | **Warm Pool** |

*Don't swap: **Grace Period** = wait before health-checking new instance · **Warm Pool** = pre-initialized instances ready to flip in seconds · **Cooldown** = wait between scaling actions.*

---

## 4. Storage — S3

| Trigger | Answer |
|---|---|
| SQL on a single S3 object | **S3 Select** (across many → **Athena**) |
| Replicate across regions | **CRR** (requires Versioning on both sides) |
| Protect against accidental delete | **Versioning** (+ **MFA Delete**) |
| Reduce cost over time, you set rules | **Lifecycle Policy** |
| Auto-archive cold objects, no retrieval fee, AWS decides | **Intelligent-Tiering** |
| Replicate **existing** objects | **S3 Batch Replication** (normal replication = new only) |
| WORM / regulatory / cannot delete | **Object Lock — Compliance Mode** (or Glacier Vault Lock) |
| Object Lock modes | **Governance** = override w/ permission · **Compliance** = nobody, not even root |
| Temporary access to private object | **Pre-signed URL** |
| Process file on upload | **S3 Event Notification → Lambda** (multi-target → **EventBridge**) |
| Static site + HTTPS + custom domain | **S3 + CloudFront (OAC) + Route 53 + ACM** |
| Multiple apps, different policies on one bucket | **S3 Access Points** |
| Requester pays egress | **S3 Requester Pays** |
| Redact PII on retrieval | **S3 Object Lambda** |
| Bulk tag/copy/process existing objects | **S3 Batch Operations** |
| 503 errors with SSE-KMS | **KMS quota exceeded** |

**Classes hot → cold:** Standard → Intelligent-Tiering → Standard-IA → One Zone-IA → Glacier Instant → Glacier Flexible → Glacier Deep Archive.

## 4b. Storage — Block & File

| Trigger | Answer |
|---|---|
| High IOPS block storage | **EBS io1/io2** (256K IOPS → **io2 Block Express**) |
| High throughput / streaming HDD | **st1** |
| Instance Store vs EBS | Instance Store = **ephemeral, fastest** · EBS = persistent network |
| EBS shared by 2+ EC2 in same AZ | **EBS Multi-Attach** (io1/io2, cluster-aware app) |
| Windows shared drive | **FSx for Windows** |
| Linux shared drive (NFS) | **EFS** |
| HPC / supercomputer FS | **FSx for Lustre** |
| Schedule EBS backups | **Data Lifecycle Manager (DLM)** |
| No first-read latency on EBS snapshot | **Fast Snapshot Restore (FSR)** |
| Migrate NFS / sync on-prem → AWS | **DataSync** (NFS/SMB/HDFS → S3/EFS/FSx) |
| Existing SFTP/FTP workflow to S3 | **Transfer Family** |

---

## 5. Databases — Relational

| Trigger | Answer |
|---|---|
| Scale read workload | **Read Replicas** (async, up to 5 RDS / 15 Aurora) |
| Production HA / auto-failover | **Multi-AZ** (sync, standby NOT readable) |
| Standby NOT accessible | **Multi-AZ** |
| Standby IS accessible | **Read Replica** |
| Encrypt existing unencrypted DB | **Snapshot → Copy w/ encryption → Restore** |
| Eliminate DB password | **IAM Database Authentication** |
| Auto-scale DB storage | **Storage Auto Scaling** |
| 5× faster MySQL | **Aurora** |
| Variable / unpredictable load | **Aurora Serverless v2** |
| Global reads, RTO < 1 min DR | **Aurora Global Database** (1 writer, up to 5 reader regions, <1s lag) |
| Lambda exhausting DB connections | **RDS Proxy** |
| Quickly undo a bad data change (Aurora) | **Backtrack** (up to 72h, in-place) |
| Fast test copy of prod DB | **Aurora Cloning** (copy-on-write) |
| Need OS-level DB access | **RDS Custom** (Oracle / SQL Server only) |
| Zero-downtime DB upgrade | **Aurora Blue/Green Deployment** |
| Performance bottleneck analysis | **RDS Performance Insights** |
| Alert on failover/maintenance | **RDS Event Notifications → SNS** |

**Multi-AZ vs Read Replica:**

| Feature | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | **HA / failover** | **Scale reads** |
| Replication | **Sync** | **Async** |
| Standby readable | No | **Yes** |
| Failover | **Auto (DNS)** | Manual promotion |
| Data loss | Zero | Possible (lag) |
| Cross-region | **No** | **Yes** |

---

## 6. Databases — NoSQL

| Trigger | Answer |
|---|---|
| Unpredictable DynamoDB workload | **On-Demand** mode |
| Query DynamoDB by non-PK attribute | **GSI** (or **LSI** if same PK, different sort) |
| Must read the very latest write | **Strongly Consistent Reads** (2× cost, not on GSI) |
| DynamoDB latency to microseconds | **DAX** |
| React to DynamoDB changes | **DynamoDB Streams → Lambda** |
| Multi-region active-active DynamoDB | **Global Tables** |
| Auto-delete old items | **TTL** |
| Atomic across multiple items | **Transactions** (ACID) |
| Analyze DynamoDB w/o consuming RCUs | **Export to S3** |
| SQL-like queries on DynamoDB | **PartiQL** |
| Cache with persistence/backups/HA | **Redis** |
| Simple multi-threaded cache | **Memcached** |
| Scale Redis horizontally | **Cluster Mode Enabled** |
| Cross-region Redis for DR | **ElastiCache Global Datastore** |
| Durable in-memory DB (not just cache) | **MemoryDB for Redis** |
| MongoDB migration | **DocumentDB** |
| Social network / fraud / recommendations | **Neptune** (graph) |
| IoT / time-series | **Timestream** |
| Immutable ledger / audit trail | **QLDB** (not blockchain) |
| Cassandra migration | **Keyspaces** |

**GSI vs LSI:**

| Feature | GSI | LSI |
|---|---|---|
| Partition Key | **Different** from base | **Same** as base |
| Sort Key | Different | Different |
| Capacity | **Own** RCU/WCU | Shares with base |
| Consistency | **Eventual only** | **Strong or Eventual** |
| When created | **Anytime** | **Only at table creation** |
| Limit per table | 20 | **5** |

**Redis vs Memcached:** Redis = complex types + HA + persistence. Memcached = simple strings, multi-threaded, no persistence, no replication.

---

## 7. Decoupling

| Trigger | Answer |
|---|---|
| Decouple producers from consumers | **SQS** |
| Strict ordering, no duplicates | **SQS FIFO** (Message Group ID = the thing you order *per*, e.g. Account ID) |
| Highest throughput, duplicates OK | **SQS Standard** (the default — best-effort order, at-least-once) |
| Reduce SQS API calls / cost | **Long Polling** (`WaitTimeSeconds = 20`) |
| Message processed multiple times | **Increase Visibility Timeout** (default 30s, max 12h) |
| Handle messages failing repeatedly | **Dead Letter Queue** (after `MaxReceiveCount` failures) |
| 🔴 **SQS message larger than 256 KB** | **SQS Extended Client Library** (payload → S3, reference → SQS) |
| Delay ALL messages 0-15 min | **Delay Queue** |
| Delay specific messages | **Message Timer** (per-message, overrides delay queue) |
| Cross-account SQS access (S3 → SQS in another account) | **SQS Access Policy** (resource-based) |
| **SNS vs SQS** | SNS = **push, pub/sub, many subscribers** · SQS = **pull, queue, one consumer group** |
| Send same message to multiple services | **SNS + SQS Fan-Out** (1 topic → N queues, each service gets its own buffer) |
| Subscriber receives only certain message types | **SNS Message Filtering** (JSON filter policy on attributes) |
| Pub/sub with strict ordering | **SNS FIFO Topic + SQS FIFO Queues** (FIFO topics can ONLY subscribe FIFO queues) |
| Real-time (<1s) + replay last N days | **Kinesis Data Streams** (retention 1d-365d) |
| Load streaming data to S3/Redshift/OpenSearch, no code | **Amazon Data Firehose** (~60s latency, auto-scales, optional Lambda transform) |
| Scale Kinesis consumers | **Add shards (split) + KCL workers** (max 1 worker per shard — extras sit idle) |
| Dedicated 2 MB/s per consumer on Kinesis | **Enhanced Fan-Out** (HTTP/2 push, no shared throughput) |
| Kinesis vs SQS — "replay last 7 days" | **Kinesis** (SQS deletes on consume; can't replay) |
| Migrate on-prem RabbitMQ / ActiveMQ | **Amazon MQ** |
| New cloud-native app | **SQS / SNS** (NOT Amazon MQ) |
| React to AWS service events (EC2 state, S3 PUT, CloudTrail API call) | **EventBridge** |
| Schedule cron / rate-based task | **EventBridge Scheduler** |
| Event-driven with filtering, many targets, cross-account | **EventBridge Rules** (up to 5 targets per rule) |
| 🔴 **Replay past events for debugging** | **EventBridge Archive & Replay** (NOT Firehose — Firehose is one-way load to S3, no replay) |

**The "queue is never the destination" model:** Producer → SQS → Consumer fleet (Lambda / EC2 ASG / ECS-Fargate / on-prem) → final data store (DB / S3 / API). The queue is a **shock absorber** between spiky producers and steady consumers — scale consumers on `ApproximateNumberOfMessagesVisible`.

**🔴 = the 2 you missed on Day 6 (May 24). Drill these first.**
