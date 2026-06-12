# SAA-C03 — Morning Recall (the one file)

> **This is the only file you need before a mock.** Skim it top-to-bottom every morning, then sit the mock. Everything is here — tactics, your traps, numbers, every discrimination, all 14 chapters. Nothing to look up elsewhere.
>
> **How to use it:** answers are in the right column — cover them with your eye, say the answer out loud, glance to check. Out-loud retrieval is what moves your score; silent reading doesn't.
>
> **Order is deliberate:** §A–§E come first because **72% of your Mock-1 misses were test-taking, not knowledge.** The technique and the traps are worth more than any single fact below. 🔴 = a fact you actually got wrong on Maarek #1.
>
> **Map:** §A Tactics · §B 🔴 Traps · §C Numbers · §D Confusable pairs · §E Cross-service "which wins" · then **1–14** the chapters: Networking · Compute · HA · Storage · DB-Relational · DB-NoSQL · Decoupling · Serverless · Containers · CDN/DNS · Security · Monitoring · Cost/Migration/DR · Additional.

---

## §A. Tactics — read first, every morning

**The 5-step process for every question:**
1. **Circle the qualifier** in the stem *before* reading answers (see table below).
2. **Predict the answer** before looking at the options. Then find your prediction.
3. **Eliminate the 2 obviously wrong.**
4. **Between the final 2, say the ONE word out loud that kills the wrong one** (see §B). This single habit attacks most of your misses.
5. **>60 sec → flag and move.** Come back later. Trust your pass-1 instinct on review.

**Circle the qualifier — what it's really asking:**

| Qualifier in the stem | What it means |
|---|---|
| **MOST cost-effective / cheapest** | Managed + serverless + right storage tier usually wins; kill anything always-on if load is spiky |
| **LEAST operational overhead / managed** | Pick the *more* managed option (Fargate > EC2, Aurora Serverless > self-managed, SQS > MQ for new apps) |
| **MOST secure** | No open ports, IAM over keys, private over public, deny by default |
| **Highly available / fault-tolerant** | Multi-AZ minimum; "across regions" → cross-region replica / Global |
| **Real-time vs near-real-time vs batch** | <1s = Kinesis · ~60s = Firehose · minutes/hours = Batch/EMR |
| **NOT / EXCEPT** | The odd one out — read every option; the 3 "good" ones are the distractors |
| **Minimal / no code changes** | Managed feature over custom code (ALB+Cognito over app login, RDS Proxy over pooling code) |

**Tie-breakers when genuinely stuck:** managed > custom · simpler > complex · purpose-built > general · AWS-native > self-hosted · serverless > provisioned (for spiky/low-ops) · private > public · deny by default.

---

## §B. 🔴 Your Traps — the "one word that kills it" table

*The exact discriminations you got wrong on Mock-1 (🔴) plus the highest-frequency look-alikes. Cover the right side; for each, say what disqualifies the tempting answer.*

| You're tempted by… | …pick this instead when the stem says | Disqualifying word |
|---|---|---|
| 🔴 SNS | **SQS** | "buffer / lost data / **replay** / can't lose messages" — SNS doesn't store |
| 🔴 Cognito **Identity** Pool | **User Pool** | "**authentication** / sign-in / user directory" (Identity Pool = AWS creds for already-authed users) |
| 🔴 Dedicated **Hosts** | **Dedicated Instances** | "**cheapest** single-tenant" (Hosts cost more — only for per-socket **BYOL licensing** visibility) |
| 🔴 EFS **Bursting** (throughput mode) | **Max I/O** (performance mode) | "big-data parallel throughput" — wrong axis |
| 🔴 Macie | **S3 Access Points** | "**enforce** per-prefix access" (Macie only *finds* PII, doesn't enforce) |
| 🔴 oldest Launch **Template** | oldest Launch **Configuration** | "ASG default termination order" |
| 🔴 Read Replica as dev DB | **Aurora Cloning** | "clone for **dev/test**" (replicas scale reads, aren't safe write envs) |
| 🔴 "rely on Multi-AZ" for upgrade | **Aurora Blue/Green** | "minimize downtime during **engine upgrade**" (Multi-AZ upgrade = downtime) |
| Firehose | **Kinesis Data Streams** | "**replay** / retention / multiple consumers" (Firehose is one-way load, no replay) |
| CloudFront | **S3 CRR** | "**replicate** to a region" (CloudFront caches at edge, doesn't replicate buckets) |
| CloudFront | **Global Accelerator** | "non-HTTP / TCP/UDP / gaming / static anycast IP / fast regional failover" |
| CNAME | **Alias record** | "**root/apex** domain → AWS resource" (CNAME can't sit at the zone apex) |
| NAT Gateway | **Egress-Only IGW** | the traffic is **IPv6** |
| Security Group | **NACL** | "**block / deny** a specific IP" (SG is allow-only) |
| Reserved Concurrency | **Provisioned Concurrency** | "eliminate **cold starts**" (Reserved just caps/limits) |
| Task **Execution** Role | Task **Role** | "the **app/container** needs S3/DynamoDB" (Execution = pull image + logs) |
| Multi-AZ | **Read Replica** | "standby must be **readable**" / "scale reads" / "**cross-region**" |
| Amazon MQ | **SQS/SNS** | "**new** cloud-native app" (MQ only for lift-and-shift of existing JMS/AMQP/MQTT) |
| Inspector | **GuardDuty** | "detect **threats / compromised** instances from logs" (Inspector = vuln *scanning*) |
| Trusted Advisor | **Compute Optimizer** | "**specific** right-size recommendation from utilization history" (TA = broad checks) |
| Strongly Consistent read | (not available) | the read is **on a GSI** (GSIs are eventually consistent only) |

---

## §C. Numbers You Must Know Cold

*Exams use exact numbers as distractor bait. These come up most.*

| Number | What it is |
|---|---|
| **720 / 1000 (~72%)** | Passing score. 65 questions, 130 min, 50 scored + 15 unscored |
| **256 KB** | Max SQS message size (bigger → **Extended Client** → S3) |
| **30 sec / 12 h** | SQS visibility timeout default / max |
| **20 sec** | SQS long-polling `WaitTimeSeconds` (cuts empty receives) |
| **0–15 min** | SQS Delay Queue / Message Timer range |
| **4 days / 14 days** | SQS message retention default / max |
| **15 min** | Lambda max timeout (longer → ECS/Batch/Fargate) |
| **10 GB** | Lambda container image max · `/tmp` configurable **512 MB–10 GB** (ephemeral) |
| **5 / 15** | Max RDS read replicas / Aurora read replicas |
| **5** | Aurora reader **regions** (Global DB) · also LSI per table |
| **20** | GSI per DynamoDB table |
| **72 h** | Aurora Backtrack window |
| **<1 sec** | Aurora Global DB cross-region replication lag |
| **60–120 sec** | RDS Multi-AZ failover time |
| **7 days / 35 days** | RDS automated backup retention default / max |
| **1 day–365 days** | Kinesis Data Streams retention |
| **~60 sec** | Firehose buffer/delivery latency |
| **1 MB/s in, 2 MB/s out** | Kinesis throughput **per shard** |
| **2 MB/s** | Kinesis Enhanced Fan-Out throughput **per consumer** |
| **5 min / 1 min** | EC2 Basic monitoring interval / Detailed monitoring (paid) |
| **1024–65535** | Ephemeral port range (NACL outbound replies) |
| **4 (2×2)** | Max DX resiliency: 2 connections × 2 locations |
| **us-east-1** | Where the ACM cert for **CloudFront** must live |
| **5** | Targets per EventBridge rule |
| **7 / AZ** | Spread Placement Group instance cap |
| **256,000 IOPS** | io2 Block Express max |
| **0 / FREE** | **Inbound** data transfer to AWS, and **Gateway** VPC endpoints |

---

## §D. Confusable Pairs — don't swap these

| Pair | The distinction |
|---|---|
| **SG vs NACL** | SG = stateful, allow-only, ENI level · NACL = stateless, allow+deny, subnet level |
| **Multi-AZ vs Read Replica** | survive failure (sync, NOT readable) · scale reads (async, readable, cross-region) |
| **Gateway vs Interface Endpoint** | S3/DynamoDB, free, route-table · everything else, PrivateLink, hourly $ + ENI |
| **User Pool vs Identity Pool** | sign-in / user directory · hand out temp AWS creds to authed users |
| **Task Role vs Execution Role** | what your app can do · what ECS needs to start the task (ECR pull, logs) |
| **Reserved vs Provisioned Concurrency** | cap a function's share · pre-warm to kill cold starts |
| **Grace Period vs Warm Pool vs Cooldown** | wait before health-checking new instance · pre-initialized instances ready to flip · wait between scaling actions |
| **CloudFront vs Global Accelerator** | cache HTTP content at edge · TCP/UDP anycast IP, fast failover, no caching |
| **CloudFront Functions vs Lambda@Edge** | sub-ms, viewer-only, header/URL rewrite · heavier, all 4 events, network/body access, ≤30s |
| **CloudWatch vs CloudTrail vs Config** | performance/metrics/logs · who made which API call · resource config state + compliance |
| **Trusted Advisor vs Compute Optimizer** | broad 5-pillar checks (flags idle) · ML right-sizing (the specific better instance type) |
| **Cluster vs Spread vs Partition PG** | low latency (1 AZ) · max isolation (≤7/AZ) · HDFS/Kafka racks |
| **Standard vs FIFO SQS** | high TPS, best-effort order, at-least-once · ordered, exactly-once, 300 msg/s |
| **Standard vs Convertible RI** | sell/modify-size, locked family · change family, can't sell |
| **Compute vs EC2-Instance Savings Plan** | any compute/region/family · locked to instance family, deeper discount |
| **Inspector vs GuardDuty vs Macie** | scan EC2/ECR vulns · detect threats from logs · find PII in S3 |
| **Shield Std vs Advanced vs WAF** | free L3/4 DDoS · paid + DDoS cost protection + DRT · L7 rules (SQLi/rate-limit) |
| **Secrets Manager vs Parameter Store** | auto-rotates secrets ($) · free config/secret storage (no auto-rotate) |
| **KMS vs CloudHSM** | AWS-managed multi-tenant keys · your dedicated single-tenant HSM, FIPS 140-2 L3 |
| **DataSync vs Storage Gateway** | move/sync data (migration) · ongoing hybrid bridge |
| **Snowball vs Snowmobile** | ~80 TB device · 100 PB truck |
| **Rehost/Replatform/Repurchase/Refactor** | lift-shift (MGN) · tweak (→RDS) · buy SaaS · re-architect cloud-native |

---

## §E. Cross-Service "Which One Wins" — span-the-chapters

*When the answer requires choosing across categories.*

| Scenario | Winner | Why not the others |
|---|---|---|
| Speed up **uploads** of large files to S3 from far away | **S3 Transfer Acceleration** | CloudFront = downloads/cache; GA = TCP/UDP apps |
| Speed up **dynamic API / global app** failover, non-HTTP | **Global Accelerator** | CloudFront caches static; Route 53 is DNS (clients cache it) |
| Cache **static** content globally | **CloudFront** | GA doesn't cache |
| **Replay** a stream for N days / multiple consumers | **Kinesis Data Streams** | SQS deletes on consume; Firehose no replay |
| Load streaming data to S3/Redshift, **no code** | **Firehose** | KDS needs consumer code |
| **New** decoupling between microservices | **SQS / SNS** | Amazon MQ only for migrating existing JMS/AMQP |
| **Migrate existing** ActiveMQ/RabbitMQ | **Amazon MQ** | SQS/SNS aren't protocol-compatible |
| React to **AWS service events** (EC2 state, S3 PUT) | **EventBridge** | SNS is for your own pub/sub fan-out |
| **Fan-out** one message to many queues | **SNS → SQS** | EventBridge if you need filtering/AWS events |
| Coordinate **multi-step** workflow w/ retries | **Step Functions** | not SQS chaining |
| Job **> 15 min** in a container | **ECS / Fargate / Batch** | Lambda caps at 15 min |
| **Private** S3/DynamoDB access, free | **Gateway Endpoint** | Interface endpoint costs hourly |
| **Private** access to your own service across VPCs | **NLB + PrivateLink** | peering exposes the whole CIDR |
| Encrypt traffic **over Direct Connect** | **VPN over DX** | DX alone is private but **not encrypted** |
| **SQL on S3**: one object vs whole bucket | **S3 Select** (one) · **Athena** (many) | — |
| Eliminate **DB connection storms** from Lambda | **RDS Proxy** | not a bigger instance |
| **Microsecond** reads on DynamoDB | **DAX** | ElastiCache is for RDS/general cache |
| **In-memory DB** that's durable (not just cache) | **MemoryDB** | ElastiCache Redis is a cache |

---

## 1. Networking

| Trigger | Answer |
|---|---|
| Block a specific IP | **NACL** (SG is allow-only) |
| Private S3/DynamoDB access, free | **Gateway Endpoint** |
| Private Kinesis/SQS/CloudWatch access | **Interface Endpoint** (PrivateLink, hourly $) |
| Connect 2 VPCs | **VPC Peering** (no CIDR overlap, not transitive) |
| Connect 50 VPCs | **Transit Gateway** (hub-and-spoke, transitive) |
| 🔴 Central VPC for shared DNS/AD/monitoring used by many VPCs | **Shared Services VPC** (on TGW hub; *not* legacy Transit VPC) |
| Private instances → outbound internet | **NAT Gateway** in public subnet |
| IPv6 outbound only | **Egress-Only Internet Gateway** |
| Bastion Host | Public-subnet EC2 for **inbound SSH jump**; SG = your IP only |
| Most secure private-instance access (no SSH) | **SSM Session Manager** (no open ports, IAM, logged) |
| Expose your service to other VPCs privately | **NLB + PrivateLink (Endpoint Service)** |
| Encrypt Direct Connect | **Site-to-Site VPN over DX** (DX is private, NOT encrypted) |
| Max DX resiliency | **2 connections × 2 locations = 4 total** |
| Increase VPN bandwidth | **ECMP via Transit Gateway** |
| Deep packet inspection at VPC level | **AWS Network Firewall** (Flow Logs = metadata only) |
| Hybrid DNS (on-prem ↔ AWS) | **Route 53 Resolver** (Inbound = on-prem→AWS · Outbound = AWS→on-prem) |
| Share subnets across accounts | **RAM** (Resource Access Manager) |
| Remote user VPN to AWS | **Client VPN** (S2S = network↔network · Client = user↔AWS) |
| Custom NACL setup | Deny-all by default; remember **ephemeral ports 1024–65535** outbound |
| 🔴 SG outbound rule port (web→DB) | **= the destination's listening port** (1433/3306/5432), NOT 443 — and reference the **source SG**, not a CIDR |

**3-tier SG chain:** Web SG ← `0.0.0.0/0:443` · App SG ← **Web-SG** on app port · DB SG ← **App-SG** on DB port. (SGs are stateful → never open the ephemeral reply port.)

---

## 2. Compute

| Trigger | Answer |
|---|---|
| Lowest latency between instances | **Cluster Placement Group** |
| Maximum fault isolation | **Spread PG** (≤7/AZ) or **Partition PG** for HDFS/Kafka |
| Cheapest for fault-tolerant batch | **Spot** (~70–90% off) |
| T2/T3 slow during business hours | Credits exhausted → **T2/T3 Unlimited** or move to **M5** |
| Per-socket / BYOL licensing | **Dedicated Hosts** |
| Cheapest single-tenant (no licensing need) | **Dedicated Instances** (cheaper than Hosts) |
| HPC sub-millisecond network | **EFA** (Elastic Fabric Adapter — *network*, not storage) |
| Preserve private IP on failover | **Attach ENI** to new instance |
| Resume app with RAM intact | **Hibernate** |
| Auto-install software at launch | **User Data** script |
| 🔴 User data privileges / when it runs | Runs **as root**, **once, at first boot** only |
| EC2 → S3 without hardcoded keys | **IAM Instance Profile** |
| Guarantee capacity in an AZ | **Capacity Reservation** (not a discount) |
| Static public IP across stop/start | **Elastic IP** |
| Prevent SSRF / metadata theft | **IMDSv2** (token-based) |
| SSH without key pairs | **EC2 Instance Connect** |
| ASG "launch config"? | **Launch Template** (Launch Configuration is legacy) |

**Launch Template vs User Data vs Lifecycle Hook:**

| Trigger | Answer |
|---|---|
| Define **what** the instance looks like (AMI, type, SG, IAM role, holds user data) | **Launch Template** |
| Run a script the **first time** the instance boots (install Apache, register service) | **User Data** (a field *inside* the Launch Template) |
| **Pause** the instance during scale-out/in to do work (drain conns, copy logs, warm-up) | **Lifecycle Hook** |
| "Copy logs before termination" | **Lifecycle Hook** (terminating) |
| "Pre-warm cache before serving traffic" | **Lifecycle Hook** (pending — user data already finished) |

*Mental model: **Launch Template** = the recipe · **User Data** = a step inside it that runs at first boot · **Lifecycle Hook** = ASG hits pause so you can act before the instance joins or leaves.*

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
| Authenticate users at the LB | **ALB + OIDC / Cognito User Pool** |
| 🔴 New instance fails health checks at startup (slow boot) | **Increase Health Check Grace Period** |
| App takes 8 min to boot, scales too slow | **Warm Pool** |
| 🔴 Why ASG won't terminate an unhealthy-looking instance | Within **grace period** OR in **Impaired** status |
| 🔴 ASG default termination order | AZ with most instances → **oldest Launch Configuration** → closest billing hour |
| 🔴 "4 instances must always be up, survive 1 AZ loss, cheapest" | **3 AZs × 2 each** (=6) — losing one AZ still leaves 4 |

*Don't swap: **Grace Period** = wait before health-checking new instance · **Warm Pool** = pre-initialized instances ready in seconds · **Cooldown** = wait between scaling actions.*

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
| Near-real-time per-upload processing, decoupled | **S3 Event → SQS → Lambda** |
| Static site + HTTPS + custom domain | **S3 + CloudFront (OAC) + Route 53 + ACM** |
| Multiple apps, different policies on one bucket | **S3 Access Points** |
| Requester pays egress | **S3 Requester Pays** |
| Redact PII on retrieval | **S3 Object Lambda** |
| Bulk tag/copy/process existing objects | **S3 Batch Operations** |
| 503 errors with SSE-KMS | **KMS quota exceeded** |
| Speed up **uploads** from far-away users | **S3 Transfer Acceleration** (NOT CloudFront) |

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
| 🔴 EFS, files rarely accessed, cut cost | **EFS Lifecycle → Standard-IA** (single-AZ OK → **One Zone-IA**, cheapest) |
| EFS axes (don't swap) | **Performance** mode (GP / Max I/O) ≠ **Throughput** mode (Elastic/Provisioned/Bursting) ≠ **Storage class** (Standard/IA/One Zone) |
| Schedule EBS backups | **Data Lifecycle Manager (DLM)** |
| No first-read latency on EBS snapshot | **Fast Snapshot Restore (FSR)** |
| Migrate NFS / sync on-prem → AWS | **DataSync** (NFS/SMB/HDFS → S3/EFS/FSx) |
| DataSync **over Direct Connect** privately | **Private VIF + PrivateLink interface endpoint** (keeps transfer off the public internet) |
| Existing SFTP/FTP workflow to S3 | **Transfer Family** |
| 🔴 Ship bulk data to Glacier cheapest | **Snowball → S3 → Lifecycle to Glacier** (can't target Glacier directly) |

---

## 5. Databases — Relational

| Trigger | Answer |
|---|---|
| Scale read workload | **Read Replicas** (async, up to 5 RDS / 15 Aurora) |
| Production HA / auto-failover | **Multi-AZ** (sync, standby NOT readable) |
| Standby NOT accessible | **Multi-AZ** |
| Standby IS accessible | **Read Replica** |
| 🔴 Minimize downtime during DB **engine upgrade** | **Aurora Blue/Green** — Multi-AZ does NOT help (upgrades hit primary+standby together = downtime) |
| Encrypt existing unencrypted DB | **Snapshot → Copy w/ encryption → Restore** (only path) |
| Eliminate DB password | **IAM Database Authentication** |
| Auto-scale DB storage | **Storage Auto Scaling** |
| 5× faster MySQL | **Aurora** |
| Variable / unpredictable load, auto-scale capacity + HA | **Aurora Serverless v2** |
| Global reads, RTO < 1 min DR | **Aurora Global Database** (1 writer, up to 5 reader regions, <1s lag) |
| Lambda exhausting DB connections | **RDS Proxy** |
| Quickly undo a bad data change (Aurora) | **Backtrack** (up to 72h, in-place) |
| 🔴 Fast dev/test copy of prod DB | **Aurora Cloning** (copy-on-write) — NOT a Read Replica |
| Need OS-level DB access | **RDS Custom** (Oracle / SQL Server only) |
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

**HA-vs-scale-vs-clone (memorize as a set):** Multi-AZ = *survive failure* · Read Replica = *scale reads* · Aurora Serverless v2 = *auto-scale capacity* · Aurora Cloning = *fast dev copy* · Snapshot→copy-encrypt→restore = *encrypt existing*.

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
| Highest throughput, duplicates OK | **SQS Standard** (best-effort order, at-least-once) |
| Reduce SQS API calls / cost | **Long Polling** (`WaitTimeSeconds = 20`) |
| Message processed multiple times | **Increase Visibility Timeout** (default 30s, max 12h) |
| Handle messages failing repeatedly | **Dead Letter Queue** (after `MaxReceiveCount` failures) |
| 🔴 **SQS message larger than 256 KB** | **SQS Extended Client Library** (payload → S3, reference → SQS) |
| Delay ALL messages 0–15 min | **Delay Queue** |
| Delay specific messages | **Message Timer** (per-message, overrides delay queue) |
| Cross-account SQS access (S3 → SQS in another account) | **SQS Access Policy** (resource-based) |
| **SNS vs SQS** | SNS = **push, pub/sub, many subscribers** · SQS = **pull, queue, one consumer group** |
| Send same message to multiple services | **SNS + SQS Fan-Out** (1 topic → N queues, each gets its own buffer) |
| Subscriber receives only certain message types | **SNS Message Filtering** (JSON filter policy on attributes) |
| Pub/sub with strict ordering | **SNS FIFO Topic + SQS FIFO Queues** (FIFO topics can ONLY subscribe FIFO queues) |
| Real-time (<1s) + replay last N days | **Kinesis Data Streams** (retention 1d–365d) |
| Load streaming data to S3/Redshift/OpenSearch, no code | **Amazon Data Firehose** (~60s latency, auto-scales, optional Lambda transform) |
| Scale Kinesis consumers | **Add shards (split) + KCL workers** (max 1 worker per shard) |
| Dedicated 2 MB/s per consumer on Kinesis | **Enhanced Fan-Out** (HTTP/2 push) |
| Migrate on-prem RabbitMQ / ActiveMQ | **Amazon MQ** |
| New cloud-native app | **SQS / SNS** (NOT Amazon MQ) |
| React to AWS service events (EC2 state, S3 PUT, CloudTrail API call) | **EventBridge** |
| Schedule cron / rate-based task | **EventBridge Scheduler** |
| Event-driven with filtering, many targets, cross-account | **EventBridge Rules** (up to 5 targets per rule) |
| 🔴 **Replay past events for debugging** | **EventBridge Archive & Replay** (NOT Firehose — Firehose is one-way, no replay) |

**The "queue is never the destination" model:** Producer → SQS → Consumer fleet (Lambda / EC2 ASG / ECS-Fargate) → final data store (DB / S3 / API). The queue is a **shock absorber** between spiky producers and steady consumers — scale consumers on `ApproximateNumberOfMessagesVisible`.

---

## 8. Serverless — Lambda / API Gateway / Step Functions

| Trigger | Answer |
|---|---|
| Task > 15 minutes | **NOT Lambda** (ECS / Batch / Fargate) |
| Lambda persistent storage | **NOT /tmp** (EFS or S3) — /tmp is ephemeral |
| Persistent **shared** storage across invocations | **Mount EFS** (Lambda must be in VPC) |
| Share libraries across many functions | **Layers** |
| Eliminate cold starts | **Provisioned Concurrency** (NOT Reserved) |
| Stop one function eating all concurrency | **Reserved Concurrency** |
| Lambda → RDS in private subnet | **Attach Lambda to VPC** (gets ENI; + NAT for internet) |
| Lambda can't access S3/DynamoDB | Fix the **Execution Role** IAM permissions |
| Route async results (success + failure) | **Destinations** (DLQ = failures only) |
| Lambda invoked by S3/SNS/EventBridge | **Async** (retries 2×, then DLQ) |
| Lambda reads from Kinesis/DynamoDB Stream/SQS | **Event Source Mapping** (Lambda polls) |
| Huge deps / existing container as Lambda | **Container image** from ECR (≤10 GB) |
| Cheapest / simplest API proxy | **HTTP API** (~70% cheaper than REST) |
| API needs caching / validation / WAF | **REST API** |
| Limit/meter API requests per customer | **Usage Plans + API Keys** |
| Browser blocks cross-origin API call | Enable **CORS** |
| User sign-in at the API | **Cognito User Pools** · custom token logic → **Lambda Authorizer** |
| Coordinate multi-step workflow (retries + visual) | **Step Functions** |
| Long-running workflow (hours/days) | **Standard** · high-volume short → **Express** (≤5 min) |

---

## 9. Containers — ECS / EKS / Fargate / ECR

| Trigger | Answer |
|---|---|
| Store Docker images privately | **ECR** (+ Image Scanning for vulnerabilities) |
| Run containers without managing servers | **Fargate** |
| Run containers on EC2 (use RIs/Spot) | **ECS EC2 Launch Type** |
| Zero infrastructure management, simplest | **App Runner** (simpler than Fargate) |
| Persistent shared storage for containers | **EFS** |
| Task-level Security Groups | **awsvpc** network mode |
| Ensure N containers always running | **ECS Service** (Desired Count = N) |
| Scale EC2 hosts when tasks are pending | **Capacity Provider + ASG** |
| Migrate Kubernetes to AWS | **EKS** |
| Simple AWS-native orchestration | **ECS** |
| Container needs S3/DynamoDB access | **Task Role** (NOT Execution Role) |
| ECS agent needs to pull image from ECR | **Execution Role** (NOT Task Role) |

*Task Role = what your app code can do (S3, DynamoDB) · Execution Role = what ECS needs to start the task (pull ECR image, write logs).*

---

## 10. Content Delivery & DNS — Route 53 / CloudFront / Global Accelerator

| Trigger | Answer |
|---|---|
| Map root/apex domain to ALB | **Alias Record** (NOT CNAME) |
| Route % of traffic to new version | **Weighted Routing** |
| Route to fastest region | **Latency-Based Routing** |
| Automatic failover to DR site | **Failover Routing** (requires Health Check) |
| Different content by country | **Geolocation Routing** |
| Serve S3 via CloudFront, block direct access | **Origin Access Control (OAC)** |
| Restrict access to premium content | **Signed URLs / Signed Cookies** |
| Block access from specific countries | **CloudFront Geo Restriction** |
| Cache static content globally | **CloudFront** |
| Static IP, UDP, gaming, fast failover | **Global Accelerator** |
| Blue/green when clients DNS-cache aggressively | **Global Accelerator** (static anycast IP, no DNS-cache problem) |
| Replicate S3 to a region (not edge cache) | **S3 CRR** (NOT CloudFront) |
| Manually clear CloudFront cache | **Invalidation** (costs $; prefer versioned filenames) |
| Simple URL rewrite / header tweak at edge | **CloudFront Functions** (sub-ms, viewer only) |
| Complex edge logic (network/body access) | **Lambda@Edge** (all 4 events, up to 30s) |
| CloudFront origin failover / HA | **Origin Groups** (primary + secondary) |
| Internal DNS within a VPC | **Route 53 Private Hosted Zone** (associate with VPC) |

---

## 11. Security & Identity — IAM / KMS / Cognito / WAF

| Trigger | Answer |
|---|---|
| EC2 needs S3 access without keys | **IAM Role** (Instance Profile) |
| Explicit Deny vs Allow | **Deny always wins** (explicit Deny → Allow → default Deny) |
| Limit max permissions a user/role can get | **Permission Boundary** (caps escalation; not an SCP, not a group) |
| Restrict an action to one region / by IP | IAM **condition** (`aws:RequestedRegion`, `aws:SourceIp`) |
| Auto-rotate RDS password | **Secrets Manager** |
| Free secret storage | **Parameter Store (Standard)** |
| User sign-up / sign-in | **Cognito User Pools** |
| Mobile app uploads to S3 without backend | **Cognito Identity Pools** (temp AWS creds) |
| S3 IAM policy structure | `ListBucket` on **bucket ARN** + `GetObject` on **bucket/\*** (two resources) |
| Block SQL injection / rate-limit API | **WAF** |
| Free DDoS protection | **Shield Standard** |
| Detect compromised instances / threats | **GuardDuty** |
| Scan EC2/ECR for vulnerabilities | **Inspector** |
| Discover PII in S3 | **Macie** |
| Cross-account S3 access | **Bucket Policy** (resource-based) |
| Encrypt large files efficiently | **Envelope Encryption (KMS)** |
| Free SSL/TLS for ALB / CloudFront | **ACM** (CloudFront → must be **us-east-1**) |
| SSL cert on EC2 | **NOT ACM** — CloudHSM or import manually |
| FIPS 140-2 Level 3 / dedicated HSM | **CloudHSM** |
| SSO across multiple AWS accounts | **IAM Identity Center** |
| Centralized security findings | **Security Hub** |
| Manage WAF rules across all accounts | **Firewall Manager** |
| Federate corporate AD (SAML) | **AssumeRoleWithSAML** |
| Social login / web identity | **AssumeRoleWithWebIdentity** (Cognito Identity Pools) |
| Use existing on-prem AD with AWS | **AD Connector** (proxy) or trust with **Managed Microsoft AD** |
| Full managed AD for Windows/SQL | **AWS Managed Microsoft AD** |

---

## 12. Monitoring & Audit — CloudWatch / CloudTrail / Config

| Trigger | Answer |
|---|---|
| Monitor EC2 memory / disk (not default) | **CloudWatch Agent** (custom metrics) |
| Need EC2 metrics every 1 minute | **Detailed Monitoring** (Basic = 5 min) |
| Alert when metric crosses threshold | **CloudWatch Alarm** |
| 🔴 Auto-recover EC2 on hardware failure | **Alarm on System Status Check → Recover** (keeps same ID/Private IP/EIP) |
| Run Lambda on a schedule | **EventBridge Scheduled Rule** |
| Who did what / API call audit | **CloudTrail** (enable Data Events for object-level) |
| Was a resource configured a certain way (history)? | **AWS Config** |
| Enforce + auto-fix resource compliance | **Config Rule + Remediation** |
| Query logs for patterns | **CloudWatch Logs Insights** |
| Alarm on a log pattern | **Metric Filter + Alarm** |
| Stream logs to OpenSearch real-time | **Logs Subscription Filter** |
| Top-N contributors to a metric (e.g. top IPs) | **Contributor Insights** |
| Proactively test API/endpoint availability | **CloudWatch Synthetics (Canaries)** |
| Debug latency across microservices | **X-Ray** (Service Map) |
| Identify underutilized resources (broad) | **Trusted Advisor** |
| Specific right-size recommendation (utilization history) | **Compute Optimizer** |
| **CloudWatch vs CloudTrail vs Config** | CloudWatch = **performance/metrics/logs** · CloudTrail = **who made API calls** · Config = **resource config state + compliance** |

---

## 13. Cost / Migration / DR

| Trigger | Answer |
|---|---|
| Analyze spending / forecast | **Cost Explorer** |
| Alert when budget exceeded | **AWS Budgets** |
| Most granular line-item billing → S3 | **Cost and Usage Report (CUR)** |
| Right-size EC2 | **Compute Optimizer** |
| Flexible savings across regions/compute | **Compute Savings Plans** |
| Highest discount, locked to family | **EC2 Instance Savings Plans / Standard RI** |
| Change instance family mid-term | **Convertible RI** |
| Sell unused RI | **Standard RI only** (Convertible can't) |
| 🔴 Predictable baseline + spiky peaks, cheapest | **RIs/Savings Plans for baseline + Spot/On-Demand for peak** (not 100% of either) |
| Inbound data transfer to AWS | **FREE** |
| Cut S3/DynamoDB cost from VPC | **Gateway VPC Endpoint** (free) |
| Lift-and-shift to AWS, minimal change | **Rehost** (MGN) |
| Move DB to managed service | **Replatform** (e.g. → RDS) |
| Replace with SaaS | **Repurchase** |
| Re-architect cloud-native | **Refactor** |
| Migrate DB minimal downtime | **DMS** (+ SCT if heterogeneous) |
| SFTP/FTP files to S3 | **Transfer Family** |
| Online NFS/SMB → S3 sync | **DataSync** |
| Offline bulk transfer (TB–PB) | **Snow Family** (Snowball / Snowmobile / Snowcone) |
| Cheapest DR | **Backup & Restore** |
| DR with Route 53 failover + always-on small ASG | **Warm Standby** |
| Near-zero RTO DR | **Multi-Site Active-Active** |
| Centralized cross-account backup | **AWS Backup** |
| Restrict services across accounts | **SCPs (Organizations)** |
| Track migration progress | **Migration Hub** |
| Break down costs by team/project | **Cost Allocation Tags** |

*DR order (cheapest/slowest → priciest/fastest):* **Backup & Restore → Pilot Light → Warm Standby → Multi-Site Active-Active.**

---

## 14. Additional Services — Analytics / IaC / ML

| Trigger | Answer |
|---|---|
| Query S3 data with SQL | **Athena** |
| Data warehouse / OLAP | **Redshift** (query S3 → **Redshift Spectrum**) |
| ETL / transform data | **Glue** (auto-schema → **Crawlers**; metadata → **Data Catalog**) |
| Reduce Athena cost | **Parquet/ORC columnar + partitioning** |
| Convert .csv → .parquet | **Glue** (ETL job) |
| Big data / Spark / Hadoop | **EMR** |
| Dashboards / BI | **QuickSight** |
| Data lake governance / column security | **Lake Formation** |
| Full-text search / log analytics | **OpenSearch** |
| Infrastructure as Code | **CloudFormation** (multi-account/region → **StackSets**) |
| Detect manual infra changes | **CloudFormation Drift Detection** |
| Deploy web app, minimal management | **Elastic Beanstalk** |
| GraphQL / real-time data sync | **AppSync** |
| Migrate Kafka to AWS | **MSK** (new streaming app → **Kinesis**) |
| Long-running batch jobs in Docker | **AWS Batch** (NOT Lambda) |
| Analyze images / detect faces | **Rekognition** |
| Extract text from documents (OCR) | **Textract** |
| Sentiment / NLP | **Comprehend** |
| Build chatbot | **Lex** |
| Custom ML model | **SageMaker** |
| AWS services on-premises | **Outposts** |
| Low latency to a specific city | **Local Zones** |
| 5G edge computing | **Wavelength** |

---

## ✅ Before you click "Start"

- [ ] You read **§A Tactics** — circle the qualifier, predict before reading, name the disqualifier.
- [ ] You drilled **§B 🔴 Traps** — these are your proven weak spots.
- [ ] You glanced at **§C Numbers** — 720 pass · 256 KB · 15 min · 60–120s · <1s · us-east-1.
- [ ] **After the mock:** log the score in StudyGuide.md and add every miss to your weak-list. Re-read that section here **the same day** — wrong-answer review is the highest-value hour.
