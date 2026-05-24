# SAA-C03 — Daily Recall Sheet (Done Chapters)

> **How to use:** Fill in the quiz tables below. Write your answer in the right column without peeking. Once a section is done, scroll to the **ANSWER KEY** (second half of doc) to check. Target: full sheet in ≤15 min.

Covers: **Networking · Compute · High Availability · Storage · Databases Relational · Databases NoSQL**

---

# 📝 PART 1 — QUIZ (Fill These In)

## QUIZ — 1. Networking

| # | Trigger | Your Answer |
|---|---|---|
| 1 | Block a specific IP? | NACL |
| 2 | Access S3 privately, free? | Gateway Endpoint |
| 3 | Access Kinesis/SQS/CloudWatch privately? | Interface Endpoint |
| 4 | Connect 50 VPCs? | VPC Peering|
| 5 | Connect 2 VPCs? | Site-to-Site |
| 6 | Private instances need outbound internet? | Bastion |
| 7 | Bastion Host? | Its an EC2 instance other EC2 connects to to reach interenet|
| 8 | Most secure private-instance access (no SSH)? | Instance Connect |
| 9 | Expose your service to other VPCs privately? | |
| 10 | Encrypt Direct Connect? | DX|
| 11 | Max DX resiliency? | 2 sites with 2 thingies|
| 12 | IPv6 outbound only? | Egress stuff |
| 13 | Deep packet inspection at VPC level? | Log Flow|
| 14 | Increase VPN bandwidth? | |
| 15 | Hybrid DNS (on-prem ↔ AWS)? | |
| 16 | Share subnets across accounts? | |
| 17 | Remote user VPN to AWS? | |
| 18 | Custom NACL blocks everything? | |

---

## QUIZ — 2. Compute

| # | Trigger | Your Answer |
|---|---|---|
| 1 | Lowest latency between instances? | cluster |
| 2 | Maximum fault isolation? | spread multi-az|
| 3 | Cheapest for fault-tolerant batch? | Spot fleet , Spot instance |
| 4 | T2/T3 slow during business hours? | use T2 Unlimited instead |
| 5 | Per-socket / BYOL licensing? | Dedicated Host|
| 6 | HPC sub-millisecond network? | io3/io4 |
| 7 | Preserve private IP on failover? | use ENIs |
| 8 | Resume app with RAM intact? | Hibernate|
| 9 | Auto-install software at launch? | Lifecycle Hook |
| 10 | EC2 → S3 without hardcoded keys? | |
| 11 | Guarantee capacity in an AZ? | Capacity Reservation|
| 12 | Static public IP that survives stop/start? | Elastic IP |
| 13 | Prevent SSRF / metadata theft? | IMDSv2 use|
| 14 | SSH without key pairs? | Instance Connect|
| 15 | ASG launch config? |  |

---

## QUIZ — 3. High Availability

| # | Trigger | Your Answer |
|---|---|---|
| 1 | Route based on URL path? | ALB |
| 2 | Static IP / extreme TPS / TCP-UDP? | NLB |
| 3 | Inspect traffic with 3rd-party firewall (GENEVE)? |GWLB |
| 4 | Invoke Lambda from HTTP? | ALB |
| 5 | Stateful app on multiple instances? | Sticky sessions?|
| 6 | Maintain CPU at 60%? | Target Scaling|
| 7 | Scale every Monday at 9 AM? | Scheduled |
| 8 | Proactively scale before a predicted spike? | Predictive |
| 9 | Run a script before instance is in-service? | Lifecycle Hook|
| 10 | Let in-flight requests finish before terminate? | Drain connection|
| 11 | Scale workers based on queue depth? | Target+SQSblabla activate|
| 12 | App behind ALB needs original client IP? |X-Forwarded-For Header activate |
| 13 | Roll out new AMI across ASG without downtime? | Launch template?|
| 14 | NLB cross-zone — free? | yes, only traffic cross-region is not free|
| 15 | Authenticate users at the LB? | Cognito+Cloudfront+Route |
| 16 | New instance failing health checks at startup? | Increase cooldown |
| 17 | App takes 8 min to boot, scales too slow? | increase grace period|

---

## QUIZ — 4. Storage

| # | Trigger | Your Answer |
|---|---|---|
| 1 | High IOPS block storage? | instance store |
| 2 | High throughput / streaming HDD? | St1|
| 3 | Instance Store vs EBS? | Ephemeral bulshit|
| 4 | EBS shared by 2+ EC2 in same AZ? | Multi-attach|
| 5 | Windows shared drive? | FSx for Windows|
| 6 | Linux shared drive (NFS)? | EFS |
| 7 | HPC / supercomputer FS? | FSx for Lustre |
| 8 | S3 503 errors with SSE-KMS? | KMS Quota|
| 9 | SQL queries on a single S3 object? | S3 Select|
| 10 | Replicate S3 across regions? | CRR/SRR activate |
| 11 | Protect against accidental delete? | Enable Versioning +MFA delete|
| 12 | Reduce S3 cost over time automatically? | Lifecycle |
| 13 | Replicate *existing* objects? | bulk Operation|
| 14 | WORM / regulatory / cannot delete? | Vault Lock  |
| 15 | Object Lock modes? | Governance , Compliance is harder|
| 16 | Temporary access to private S3 object? |Pre-signed URL |
| 17 | Process file on upload? | Lifecycle? |
| 18 | Static site + HTTPS + custom domain? | well do that, with S3 static host|
| 19 | Schedule EBS backups? | activate Backup?|
| 20 | No first-read latency on EBS snapshot? | warm snapshot|
| 21 | Migrate NFS / sync on-prem → AWS? | DataSync |
| 22 | Existing SFTP/FTP workflow to S3? | Family Transfer|
| 23 | Multiple apps, different S3 policies on one bucket? | |
| 24 | Share large S3 dataset, requester pays egress? | S3 Requester pay |
| 25 | Redact PII on retrieval? | |
| 26 | Bulk tag/copy/process existing objects? | Bulk Operation|
| 27 | Auto-archive cold objects, no retrieval fee? | Intelligent Tiering|

---

## QUIZ — 5. Databases Relational

| # | Trigger | Your Answer |
|---|---|---|
| 1 | Scale read workload? | Read Replicas|
| 2 | Production HA / auto-failover? |Multi-AZ|
| 3 | Standby NOT accessible? |Multi-AZ |
| 4 | Standby IS accessible? | Read-replicas|
| 5 | Encrypt existing unencrypted DB? |create snapshot, encrypt it, create new encrypted DB from it |
| 6 | Eliminate DB password? | IAM Connect |
| 7 | Auto-scale DB storage? |On-demand Mode|
| 8 | 5× faster MySQL? | Aurora|
| 9 | Variable / unpredictable load? | Aurora Serverless? |
| 10 | Global reads, RTO < 1 min DR? | Aurora Global DB|
| 11 | Lambda exhausting DB connections? | RDS Proxy |
| 12 | Quickly undo a bad data change (Aurora)? | Aurora Backtrack|
| 13 | Fast test copy of prod DB? | Aurora Cloning|
| 14 | Need OS-level DB access? | RDS Custom |
| 15 | Zero-downtime DB upgrade? | Aurora Upgrade?|
| 16 | DB performance bottleneck analysis? | RDS Insights|
| 17 | Alert on failover/maintenance? | Eventbridge |

**Multi-AZ vs Read Replica — fill in:**

| Feature | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | DR| Scale Reads |
| Replication | Sync | Async|
| Standby readable |No | Yes|
| Failover | Yes| No |
| Data loss | No| Yes|
| Cross-region | Yes| Yes |

---

## QUIZ — 6. Databases NoSQL

| # | Trigger | Your Answer |
|---|---|---|
| 1 | Unpredictable DynamoDB workload? | use On-Demand|
| 2 | Query DynamoDB by non-PK attribute? | GSI/LSI |
| 3 | Must read the very latest write? | use LSI|
| 4 | DynamoDB latency to microseconds? |use DAX |
| 5 | React to DynamoDB changes? |Stream |
| 6 | Multi-region active-active DynamoDB? |Global DS |
| 7 | Auto-delete old items? | TTL|
| 8 | Atomic across multiple items? | Transactions stuff|
| 9 | Analyze DynamoDB data w/o consuming RCUs? |Streams |
| 10 | Cache with persistence/backups? | Redis|
| 11 | Simple multi-threaded cache? | Memcache|
| 12 | Scale Redis horizontally? |In-Memory DB |
| 13 | Session storage for stateless app? | Memcache|
| 14 | MongoDB migration? | DocumentDB|
| 15 | Social network / fraud / recommendations? |Neptune |
| 16 | IoT / time-series? | AWS timeseries DB |
| 17 | Immutable ledger / audit trail? | QWL DB|
| 18 | Cassandra migration? | Keyspaces |
| 19 | Durable in-memory DB (not just cache)? | Managed In-Memory DB from AWS |
| 20 | SQL-like queries on DynamoDB? |DynamoDB PratiQL|
| 21 | Cross-region Redis for DR? | |

**GSI vs LSI — fill in:**

| Feature | GSI | LSI |
|---|---|---|
| Partition Key | | |
| Sort Key | | |
| Capacity | Has its own RCU/WCU| shares provisioned RCU/WCU|
| Consistency | | |
| When created |Whenever you want | Created at Table Creation|
| Limit per table |20 | Unlimited |

**Redis vs Memcached — write a one-line summary for each:**

- Redis: Cache DB if you want performance, persistance, feature rich with minimal app configuration 
- Memcached: Simple multi-threaded ephemeral Cache DB

---

## ⚡ Drill mode — say the answer in 2 seconds (no peeking)

1. Block one IP fast → ?
2. Private S3 access free → ?
3. Cheap fault-tolerant batch → ?
4. Static IP load balancer → ?
5. Path-based routing → ?
6. Same-AZ multi-writer block storage → ?
7. Windows file share / Linux / HPC → ? / ? / ?
8. WORM compliance → ?
9. Survive AZ failure (SQL DB) → ?
10. Scale SQL reads → ?
11. Sub-ms DynamoDB → ?
12. Query DynamoDB by another attribute → ?
13. Persistent cache / simple multi-threaded → ? / ?
14. Graph / time-series / ledger → ? / ? / ?
15. Replicate existing S3 objects → ?

---
---

# ✅ PART 2 — ANSWER KEY (Don't Scroll Until You're Done)

⬇️ ⬇️ ⬇️

## 1. NETWORKING — Answers

| # | Trigger | Answer |
|---|---|---|
| 1 | Block a specific IP? | **NACL** (SGs are allow-only) |
| 2 | Access S3 privately, free? | **Gateway Endpoint** |
| 3 | Access Kinesis/SQS/CloudWatch privately? | **Interface Endpoint** (PrivateLink, hourly fee) |
| 4 | Connect 50 VPCs? | **Transit Gateway** |
| 5 | Connect 2 VPCs? | **VPC Peering** (no CIDR overlap, not transitive) |
| 6 | Private instances need outbound internet? | **NAT Gateway** in public subnet |
| 7 | Bastion Host? | Public-subnet EC2 for SSH jump; SG allows SSH **from your IP only** |
| 8 | Most secure private-instance access (no SSH)? | **SSM Session Manager** |
| 9 | Expose your service to other VPCs privately? | **NLB + PrivateLink (Endpoint Service)** |
| 10 | Encrypt Direct Connect? | **Site-to-Site VPN over DX** |
| 11 | Max DX resiliency? | **2 connections × 2 locations (4 total)** |
| 12 | IPv6 outbound only? | **Egress-Only Internet Gateway** |
| 13 | Deep packet inspection at VPC level? | **AWS Network Firewall** |
| 14 | Increase VPN bandwidth? | **ECMP via Transit Gateway** |
| 15 | Hybrid DNS (on-prem ↔ AWS)? | **Route 53 Resolver** (Inbound/Outbound endpoints) |
| 16 | Share subnets across accounts? | **RAM (Resource Access Manager)** |
| 17 | Remote user VPN to AWS? | **Client VPN** |
| 18 | Custom NACL blocks everything? | Custom NACLs deny-all by default → add allow rules + **ephemeral ports 1024-65535** |

**Picture this:** Private EC2 → NAT GW → IGW → Internet. If you can draw it, you pass the networking questions.

---

## 2. COMPUTE — Answers

| # | Trigger | Answer |
|---|---|---|
| 1 | Lowest latency between instances? | **Cluster Placement Group** |
| 2 | Maximum fault isolation? | **Spread PG** (max 7/AZ) — or **Partition PG** for big distributed apps (HDFS/Kafka) |
| 3 | Cheapest for fault-tolerant batch? | **Spot** (~70-90% off) |
| 4 | T2/T3 slow during business hours? | CPU credits exhausted → **T2/T3 Unlimited** or switch to **M5** |
| 5 | Per-socket / BYOL licensing? | **Dedicated Hosts** (visibility into sockets/cores) |
| 6 | HPC sub-millisecond network? | **EFA** (Elastic Fabric Adapter) |
| 7 | Preserve private IP on failover? | **Attach ENI** to new instance |
| 8 | Resume app with RAM intact? | **Hibernate** |
| 9 | Auto-install software at launch? | **User Data** script |
| 10 | EC2 → S3 without hardcoded keys? | **IAM Instance Profile** |
| 11 | Guarantee capacity in an AZ? | **Capacity Reservation** (not a discount — combine with SP/RI) |
| 12 | Static public IP that survives stop/start? | **Elastic IP** (charged when unassociated) |
| 13 | Prevent SSRF / metadata theft? | **IMDSv2** (token-based) |
| 14 | SSH without key pairs? | **EC2 Instance Connect** |
| 15 | ASG launch config? | **Launch Template** (Launch Configuration is legacy) |

**Purchasing options:** On-Demand (full price) · Reserved (1y/3y, 40-72% off) · Savings Plans (flexible) · Spot (cheapest, can be reclaimed) · Dedicated Host (compliance/licensing).

---

## 3. HIGH AVAILABILITY — Answers

| # | Trigger | Answer |
|---|---|---|
| 1 | Route based on URL path? | **ALB** |
| 2 | Static IP / extreme TPS / TCP-UDP? | **NLB** |
| 3 | Inspect traffic with 3rd-party firewall (GENEVE)? | **GWLB** |
| 4 | Invoke Lambda from HTTP? | **ALB** |
| 5 | Stateful app on multiple instances? | **ALB + Stickiness** |
| 6 | Maintain CPU at 60%? | **Target Tracking Scaling** |
| 7 | Scale every Monday at 9 AM? | **Scheduled Scaling** |
| 8 | Proactively scale before a predicted spike? | **Predictive Scaling** |
| 9 | Run a script before instance is in-service? | **Lifecycle Hook** |
| 10 | Let in-flight requests finish before terminate? | **Connection Draining** (Deregistration Delay) |
| 11 | Scale workers based on queue depth? | **SQS `ApproximateNumberOfMessagesVisible` + ASG Target Tracking** |
| 12 | App behind ALB needs original client IP? | **`X-Forwarded-For` header** |
| 13 | Roll out new AMI across ASG without downtime? | **Instance Refresh** (min healthy %) |
| 14 | NLB cross-zone — free? | **No** (NLB charges inter-AZ; ALB cross-zone is free) |
| 15 | Authenticate users at the LB? | **ALB + OIDC/Cognito** |
| 16 | New instance failing health checks at startup? | **Increase Health Check Grace Period** |
| 17 | App takes 8 min to boot, scales too slow? | **Warm Pool** |

---

## 4. STORAGE — Answers

| # | Trigger | Answer |
|---|---|---|
| 1 | High IOPS block storage? | **io1/io2** (256K IOPS → **io2 Block Express**) |
| 2 | High throughput / streaming HDD? | **st1** |
| 3 | Instance Store vs EBS? | Instance Store = **ephemeral, fastest** · EBS = persistent network |
| 4 | EBS shared by 2+ EC2 in same AZ? | **EBS Multi-Attach** (io1/io2 only, cluster-aware app) |
| 5 | Windows shared drive? | **FSx for Windows** |
| 6 | Linux shared drive (NFS)? | **EFS** |
| 7 | HPC / supercomputer FS? | **FSx for Lustre** |
| 8 | S3 503 errors with SSE-KMS? | **KMS quota exceeded** |
| 9 | SQL queries on a single S3 object? | **S3 Select** · across many → **Athena** |
| 10 | Replicate S3 across regions? | **CRR** (requires **Versioning** on both sides) |
| 11 | Protect against accidental delete? | **Versioning** (+ **MFA Delete** for extra) |
| 12 | Reduce S3 cost over time automatically? | **Lifecycle Policy** (you set rules + can delete) — vs **Intelligent-Tiering** (AWS decides on access patterns, no delete) |
| 13 | Replicate *existing* objects? | **S3 Batch Replication** (replication rules only catch *new* objects) |
| 14 | WORM / regulatory / cannot delete? | **S3 Object Lock — Compliance Mode** (or Glacier Vault Lock) |
| 15 | Object Lock modes? | **Governance** = override with permission · **Compliance** = nobody, not even root |
| 16 | Temporary access to private S3 object? | **Pre-signed URL** |
| 17 | Process file on upload? | **S3 Event Notification → Lambda** · multiple targets → **EventBridge** |
| 18 | Static site + HTTPS + custom domain? | **S3 + CloudFront (OAC) + Route 53 + ACM** |
| 19 | Schedule EBS backups? | **Data Lifecycle Manager (DLM)** |
| 20 | No first-read latency on EBS snapshot? | **Fast Snapshot Restore (FSR)** |
| 21 | Migrate NFS / sync on-prem → AWS? | **DataSync** (NFS/SMB/HDFS → S3/EFS/FSx) |
| 22 | Existing SFTP/FTP workflow to S3? | **Transfer Family** |
| 23 | Multiple apps, different S3 policies on one bucket? | **S3 Access Points** |
| 24 | Share large S3 dataset, requester pays egress? | **S3 Requester Pays** |
| 25 | Redact PII on retrieval? | **S3 Object Lambda** |
| 26 | Bulk tag/copy/process existing objects? | **S3 Batch Operations** |
| 27 | Auto-archive cold objects, no retrieval fee? | **S3 Intelligent-Tiering** (Archive Access + Deep Archive tiers) |

**S3 classes ladder (hot → cold):** Standard → Intelligent-Tiering → Standard-IA → One Zone-IA → Glacier Instant → Glacier Flexible → Glacier Deep Archive.

---

## 5. DATABASES RELATIONAL — Answers

| # | Trigger | Answer |
|---|---|---|
| 1 | Scale read workload? | **Read Replicas** (async, up to 5 RDS / 15 Aurora) |
| 2 | Production HA / auto-failover? | **Multi-AZ** (sync, standby NOT readable) |
| 3 | Standby NOT accessible? | **Multi-AZ** |
| 4 | Standby IS accessible? | **Read Replica** |
| 5 | Encrypt existing unencrypted DB? | **Snapshot → Copy w/ encryption → Restore** |
| 6 | Eliminate DB password? | **IAM Database Authentication** |
| 7 | Auto-scale DB storage? | **Storage Auto Scaling** |
| 8 | 5× faster MySQL? | **Aurora** |
| 9 | Variable / unpredictable load? | **Aurora Serverless v2** |
| 10 | Global reads, RTO < 1 min DR? | **Aurora Global Database** (writer in 1 region, readers in up to 5, <1s lag) |
| 11 | Lambda exhausting DB connections? | **RDS Proxy** |
| 12 | Quickly undo a bad data change (Aurora)? | **Backtrack** (up to 72h, in-place) |
| 13 | Fast test copy of prod DB? | **Aurora Cloning** (copy-on-write, seconds) |
| 14 | Need OS-level DB access? | **RDS Custom** (Oracle / SQL Server only) |
| 15 | Zero-downtime DB upgrade? | **Aurora Blue/Green Deployment** |
| 16 | DB performance bottleneck analysis? | **RDS Performance Insights** |
| 17 | Alert on failover/maintenance? | **RDS Event Notifications → SNS** |

**Multi-AZ vs Read Replica — the one table to know:**

| Feature | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | **HA / Failover** | **Scale reads** |
| Replication | **Synchronous** | **Asynchronous** |
| Standby readable | **No** | **Yes** |
| Failover | Auto (DNS) | Manual promotion |
| Data loss | Zero | Possible (lag) |
| Cross-region | No | **Yes** |

---

## 6. DATABASES NoSQL — Answers

| # | Trigger | Answer |
|---|---|---|
| 1 | Unpredictable DynamoDB workload? | **On-Demand mode** |
| 2 | Query DynamoDB by non-PK attribute? | **GSI** (or LSI if same PK, different sort) |
| 3 | Must read the very latest write? | **Strongly Consistent Reads** (2× cost, no GSI) |
| 4 | DynamoDB latency to microseconds? | **DAX** |
| 5 | React to DynamoDB changes? | **DynamoDB Streams → Lambda** |
| 6 | Multi-region active-active DynamoDB? | **Global Tables** |
| 7 | Auto-delete old items? | **TTL** |
| 8 | Atomic across multiple items? | **Transactions** (ACID) |
| 9 | Analyze DynamoDB data w/o consuming RCUs? | **Export to S3** |
| 10 | Cache with persistence/backups? | **Redis** |
| 11 | Simple multi-threaded cache? | **Memcached** |
| 12 | Scale Redis horizontally? | **Cluster Mode Enabled** |
| 13 | Session storage for stateless app? | **ElastiCache** (Redis or Memcached) |
| 14 | MongoDB migration? | **DocumentDB** |
| 15 | Social network / fraud / recommendations? | **Neptune** (graph) |
| 16 | IoT / time-series? | **Timestream** |
| 17 | Immutable ledger / audit trail? | **QLDB** (not blockchain) |
| 18 | Cassandra migration? | **Keyspaces** |
| 19 | Durable in-memory DB (not just cache)? | **MemoryDB for Redis** |
| 20 | SQL-like queries on DynamoDB? | **PartiQL** |
| 21 | Cross-region Redis for DR? | **ElastiCache Global Datastore** |

**GSI vs LSI:**

| Feature | GSI | LSI |
|---|---|---|
| Partition Key | **Different** from base | **Same** as base |
| Sort Key | Different | Different |
| Capacity | **Own** RCU/WCU | Shares with base |
| Consistency | Eventual only | Strong or Eventual |
| When created | **Anytime** | **Only at table creation** |
| Limit per table | 20 | 5 |

**Redis vs Memcached (one-liner):** Redis = **complex data types + HA + persistence**. Memcached = **simple strings, multi-threaded, no persistence, no replication**.

---

## ⚡ Drill mode — Answers

1. Block one IP fast → **NACL**
2. Private S3 access free → **Gateway Endpoint**
3. Cheap fault-tolerant batch → **Spot**
4. Static IP load balancer → **NLB**
5. Path-based routing → **ALB**
6. Same-AZ multi-writer block storage → **EBS Multi-Attach (io1/io2)**
7. Windows file share → **FSx for Windows** · Linux → **EFS** · HPC → **FSx for Lustre**
8. WORM compliance → **Object Lock Compliance Mode**
9. Survive AZ failure (SQL DB) → **Multi-AZ**
10. Scale SQL reads → **Read Replica**
11. Sub-ms DynamoDB → **DAX**
12. Query DynamoDB by another attribute → **GSI**
13. Persistent cache → **Redis** · simple multi-threaded → **Memcached**
14. Graph data → **Neptune** · time-series → **Timestream** · ledger → **QLDB**
15. Replicate existing S3 objects → **S3 Batch Replication**

---

## Self-test rules

- **Green (got it ≤2s):** skip next time, revisit weekly.
- **Yellow (got it but slow):** revisit in 2 days.
- **Red (missed/wrong):** open the chapter section *today* and re-read just that block.

Target: full sheet under 15 min, ≤3 reds by exam week.

---
---

# 🩹 PART 3 — MISTAKES REPORT (Round 1)

> Your wrong/shaky answers, the right answer, and the *why*. Burn these in — every one of these is a textbook exam question.

## NETWORKING — needs the most work ⚠️

| # | Question | You said | Correct | Why it matters |
|---|---|---|---|---|
| 4 | Connect 50 VPCs? | VPC Peering | **Transit Gateway** | Peering is non-transitive and explodes into a 1,225-link mesh at 50 VPCs. TGW is the **hub-and-spoke** — transitive, scales to thousands. |
| 5 | Connect 2 VPCs? | Site-to-Site | **VPC Peering** | S2S VPN = on-prem ↔ AWS. **Peering = VPC ↔ VPC.** Check no CIDR overlap, not transitive. |
| 6 | Private instances need outbound internet? | Bastion | **NAT Gateway** (in public subnet) | Bastion = inbound SSH jump for admins. NAT GW = **outbound** internet for private instances. Different direction, different purpose. |
| 7 | Bastion Host? | "EC2 other EC2 uses to reach internet" | **Public-subnet EC2 for inbound SSH jump**; SG allows SSH from *your IP only* | You're describing a NAT Instance. Bastion = inbound admin SSH, NOT outbound internet. |
| 8 | Most secure private-instance access (no SSH)? | Instance Connect | **SSM Session Manager** | Instance Connect still uses **port 22**. Session Manager = **no open ports**, IAM-based, logged to CloudWatch/S3. The "most secure" trigger always = SSM. |
| 10 | Encrypt Direct Connect? | DX | **Site-to-Site VPN over DX** | DX itself is *private but NOT encrypted*. To encrypt, run S2S VPN on top of it. Trap: "private ≠ encrypted." |
| 11 | Max DX resiliency? | "2 sites with 2 thingies" ✅ish | **2 connections × 2 locations = 4 total** | Max resiliency = 2 DX per location at 2 locations. Just memorize "4 total". |
| 12 | IPv6 outbound only? | "Egress stuff" ✅ish | **Egress-Only Internet Gateway** | IPv6-only. IPv4 outbound = NAT GW. |
| 13 | Deep packet inspection at VPC level? | Flow Logs | **AWS Network Firewall** | Flow Logs = metadata only (no payload). **Network Firewall** = deep packet inspection / IDS/IPS / domain filtering. |
| 14 | Increase VPN bandwidth? | — | **ECMP via Transit Gateway** | Equal-Cost Multi-Path spreads traffic across multiple tunnels. Works with **TGW + VPN only**, NOT plain VGW. |
| 15 | Hybrid DNS (on-prem ↔ AWS)? | — | **Route 53 Resolver** | Inbound endpoint = on-prem queries AWS. Outbound endpoint = AWS queries on-prem. |
| 16 | Share subnets across accounts? | — | **RAM** (Resource Access Manager) | Shares subnets across accounts in an Org. |
| 17 | Remote user VPN to AWS? | — | **Client VPN** | End-user remote access (OpenVPN). S2S VPN = network-to-network. |
| 18 | Custom NACL blocks everything? | — | Custom NACLs deny-all by default → add allow rules + **ephemeral ports 1024-65535** | NACL is stateless so return traffic needs explicit outbound on 1024-65535. |

**Networking priority:** re-read `NetworkingSpeedRefresher.md` sections **3, 4, 5, 7** today. The TGW vs Peering vs S2S vs Client VPN distinction is worth ~5 exam questions.

---

## COMPUTE

| # | Question | You said | Correct | Why |
|---|---|---|---|---|
| 2 | Maximum fault isolation? | "spread multi-az" ✅ partial | **Spread PG** (max 7/AZ) or **Partition PG** for HDFS/Kafka | Spread = max 7 instances per AZ. Partition = large distributed apps. |
| 6 | HPC sub-millisecond network? | io3/io4 | **EFA** (Elastic Fabric Adapter) | io1/io2 = block storage (EBS). **EFA = network adapter** for HPC, sub-ms latency. Wrong category. |
| 9 | Auto-install software at launch? | Lifecycle Hook | **User Data** script | User Data = bootstrap script at first launch. Lifecycle Hook = ASG pause for actions before in-service. |
| 10 | EC2 → S3 without hardcoded keys? | — | **IAM Instance Profile** | The container that attaches an IAM role to EC2. No hardcoded keys = always Instance Profile. |
| 13 | Prevent SSRF / metadata theft? | "IMDSv2 use" ✅ | IMDSv2 (token-based) | Correct — IMDSv1 was vulnerable to SSRF, v2 requires session tokens. |
| 15 | ASG launch config? | — | **Launch Template** | Launch Configuration is **legacy/deprecated** — always pick Launch Template. |

---

## HIGH AVAILABILITY

| # | Question | You said | Correct | Why |
|---|---|---|---|---|
| 13 | Roll out new AMI across ASG without downtime? | "Launch template?" | **Instance Refresh** (set minimum healthy %) | Launch Template is just *where* you define the new AMI. **Instance Refresh** is the *mechanism* that rolls it out across the ASG. |
| 14 | NLB cross-zone — free? | "yes" — WRONG | **No, NLB charges inter-AZ cross-zone traffic** | ALB cross-zone = **free**. NLB cross-zone = **paid** (inter-AZ data transfer). Common trap. |
| 15 | Authenticate users at the LB? | "Cognito+CloudFront+Route" | **ALB + OIDC/Cognito** | Authentication at the LB = ALB built-in feature. CloudFront/Route aren't doing auth. |
| 16 | New instance failing health checks at startup? | Increase cooldown | **Increase Health Check Grace Period** | Grace Period = how long ASG waits before health-checking new instances (boot time). Cooldown = wait between scaling actions. |
| 17 | App takes 8 min to boot, scales too slow? | Increase grace period | **Warm Pool** | Grace Period delays health checks. **Warm Pool** = pre-initialized instances ready to flip to in-service in seconds → solves slow boot. |

⚠️ #16 and #17 — you swapped them. Memorize: **slow startup health checks = Grace Period · slow boot time = Warm Pool.**

---

## STORAGE

| # | Question | You said | Correct | Why |
|---|---|---|---|---|
| 1 | High IOPS block storage? | Instance Store | **EBS io1/io2** (256K IOPS → io2 Block Express) | Instance Store is fast but **ephemeral**. The exam answer for "high IOPS block storage" = io1/io2. |
| 13 | Replicate *existing* objects? | Bulk Operation | **S3 Batch Replication** | Batch *Operations* = tag/copy/process. Batch **Replication** = replicate *existing* objects (regular replication only catches new ones). |
| 14 | WORM / regulatory / cannot delete? | Vault Lock ✅ish | **S3 Object Lock — Compliance Mode** (or Glacier Vault Lock) | Both valid. Object Lock = S3 native. Vault Lock = Glacier. |
| 17 | Process file on upload? | Lifecycle? | **S3 Event Notification → Lambda** | Lifecycle = age-based transitions/deletion. Event Notification = trigger on upload. |
| 19 | Schedule EBS backups? | "activate Backup?" | **Data Lifecycle Manager (DLM)** | DLM = native EBS snapshot scheduler. AWS Backup also works but DLM is the EBS-specific answer. |
| 20 | No first-read latency on EBS snapshot? | "warm snapshot" | **Fast Snapshot Restore (FSR)** | Term to memorize: **FSR**. Eliminates the first-block "lazy load" latency. |
| 23 | Multiple apps, different S3 policies on one bucket? | — | **S3 Access Points** | Dedicated DNS name + policy per app on one bucket. |
| 25 | Redact PII on retrieval? | — | **S3 Object Lambda** | Transform/redact objects on retrieval (PII redaction, format conversion). |

---

## DATABASES — RELATIONAL

| # | Question | You said | Correct | Why |
|---|---|---|---|---|
| 7 | Auto-scale DB storage? | On-Demand Mode | **Storage Auto Scaling** | On-Demand is a DynamoDB capacity mode. For RDS storage growth = **Storage Auto Scaling** feature. |
| 15 | Zero-downtime DB upgrade? | "Aurora Upgrade?" | **Aurora Blue/Green Deployment** | Specific feature name. Spins up a green env, syncs, swap with one click. |
| 17 | Alert on failover/maintenance? | EventBridge | **RDS Event Notifications → SNS** | Native RDS feature, publishes to SNS. (EventBridge works too but RDS Event Notifications is the textbook answer.) |

**Multi-AZ vs RR table — you got wrong:**
- **Failover:** Multi-AZ = **automatic (DNS)** · Read Replica = **manual promotion**. You had it backwards.
- **Data loss:** Multi-AZ = **zero** (sync) · Read Replica = **possible** (async lag). You had it right but list it the other way for clarity.
- **Cross-region:** Multi-AZ = **NO** · Read Replica = **YES**. You said "Yes / Yes" — Multi-AZ is single-region only.

---

## DATABASES — NoSQL

| # | Question | You said | Correct | Why |
|---|---|---|---|---|
| 3 | Must read the very latest write? | LSI | **Strongly Consistent Reads** | LSI is for *querying* by a different sort key. The "must read latest" trigger = **Strongly Consistent Reads** (2× cost, no GSI). |
| 6 | Multi-region active-active DynamoDB? | Global DS | **Global Tables** | DynamoDB = **Global Tables** (multi-region active-active). "Global Datastore" = ElastiCache Redis. |
| 9 | Analyze DynamoDB data w/o consuming RCUs? | Streams | **Export to S3** | Streams = react to changes (24h retention). **Export to S3** = analyze full table data without consuming RCUs. |
| 12 | Scale Redis horizontally? | "In-Memory DB" | **Cluster Mode Enabled** | Redis cluster mode shards data across nodes for horizontal scale. |
| 13 | Session storage for stateless app? | Memcached | **ElastiCache (Redis OR Memcached)** | Both work for session storage — Redis if you need persistence/HA, Memcached for simple/fast. |
| 16 | IoT / time-series? | "AWS timeseries DB" ✅ish | **Timestream** | Just memorize the name. |
| 17 | Immutable ledger / audit trail? | "QWL DB" — typo | **QLDB** (Quantum Ledger DB) | Q-L-D-B. Immutable, cryptographically verified. *Not* a blockchain. |
| 19 | Durable in-memory DB (not just cache)? | "Managed In-Memory DB" ✅ish | **MemoryDB for Redis** | Specific name. Durable Redis (not just cache). |
| 21 | Cross-region Redis for DR? | — | **ElastiCache Global Datastore** | Cross-region Redis replication for DR. |

**GSI vs LSI table — fill the gaps:**

| Feature | GSI | LSI |
|---|---|---|
| Partition Key | **Different** from base table | **Same** as base table |
| Sort Key | Different | Different |
| Consistency | **Eventual only** | **Strong or Eventual** |
| Limit per table | 20 | **5** (not unlimited!) |

---

## 🎯 Your Top 10 to drill TODAY

1. **TGW vs Peering vs S2S VPN vs Client VPN** — 4 different connectivity tools, you confused them all.
2. **Bastion vs NAT vs Session Manager** — direction + purpose matrix.
3. **DX is NOT encrypted** → need S2S VPN over DX.
4. **Network Firewall** vs Flow Logs (inspect vs log metadata).
5. **EFA** = HPC network adapter (NOT a storage type).
6. **User Data** vs Lifecycle Hook (boot script vs ASG pause).
7. **Grace Period** (slow health check) vs **Warm Pool** (slow boot) vs **Cooldown** (between scaling actions).
8. **Instance Refresh** for rolling AMI updates.
9. **NLB cross-zone is PAID**, ALB cross-zone is free.
10. **DynamoDB: Strongly Consistent Reads ≠ LSI.** LSI = different sort key.

Run the Networking quiz again after reading the Speed Refresher. Skip what you got right; only redo the reds.
