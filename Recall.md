# SAA-C03 — Recall & Mock Cockpit (full reference)

> ⚠️ **For your ≤30-min pre-mock recall, use [Recall30.md](./Recall30.md), not this file.** This one is the **full reference** (~280 triggers) — too long to recite daily. Use it *after* a mock, to read up on the sections you got wrong. Don't try to recall the whole thing.

> **Your one-stop sheet to attack the mocks (Jun 8–10) and the exam (Jun 11).**
> Two ways to use it:
> 1. **Daily recall** — read the trigger, say the answer out loud, then click **"reveal"** to check yourself. (Answers in the chapter tables §1–§14 are hidden in collapsible dropdowns so you test first, then see the answer.) Whole 14-chapter sweep in 20–30 min. Mark misses, re-read that section.
> 2. **Pre-mock warm-up (10 min)** — read **§0 Exam Tactics**, the **§0b Killer-Distractor table**, and the **§0c Numbers** sheet right before you start a timed mock. These are where most of your Mock-1 points leaked (72% of misses were test-taking, not knowledge).
>
> **Sections:** §0 Tactics · §0b Killer distractors · §0c Numbers cold · §0d Confusable pairs · §0e Cross-service "which wins" · then the 14 chapters: Networking · Compute · HA · Storage · DB Relational · DB NoSQL · Decoupling · Serverless · Containers · CDN/DNS · Security · Monitoring · Cost/Migration/DR · Additional.
>
> 🔴 = a fact you actually got wrong on Maarek #1 (May 24). Drill these hardest.

---

## 0. Exam Tactics — read this first, every mock

**The 4 things that cost you points on Mock-1 (all test-taking, not knowledge):**

1. **Name the one disqualifying word out loud.** When two answers look right, force yourself to say the *single word* that kills the wrong one (e.g. "*authentication* → User Pool, not Identity Pool"). 10-sec habit, biggest payoff. See §0b.
2. **Mode ≠ tier ≠ class.** Many services have *multiple independent axes* (EFS: performance mode vs throughput mode vs storage class). If the question names one axis, don't answer from a different axis. See §0d.
3. **HA ≠ scale ≠ DR.** Multi-AZ survives failure; Read Replica scales reads; Aurora Serverless auto-scales capacity; Aurora Cloning makes a dev copy. Don't conflate. See §5.
4. **Read the qualifier in the stem.** Circle the constraint before reading answers:

| Qualifier in question | What it's really asking |
|---|---|
| **MOST cost-effective / cheapest** | Managed + serverless + right storage tier usually wins; eliminate anything always-on if spiky |
| **LEAST operational overhead / managed** | Pick the *more* managed service (Fargate > EC2, Aurora Serverless > self-managed, SQS > MQ for new apps) |
| **MOST secure** | No open ports, IAM over keys, private over public, deny by default |
| **Highly available / fault-tolerant** | Multi-AZ minimum; "across regions" → cross-region replica / Global |
| **Real-time vs near-real-time vs batch** | <1s = Kinesis · ~60s = Firehose · minutes/hours = Batch/EMR |
| **NOT / EXCEPT** | The odd one out — read every option, the 3 "good" ones are the distractors |
| **Minimal code / no code changes** | Managed feature over custom (ALB+Cognito over app login, RDS Proxy over connection pooling code) |

**Process per question:** (1) read stem, circle the qualifier · (2) predict the answer *before* looking · (3) eliminate 2 obviously wrong · (4) between the last 2, name the disqualifying word · (5) flag-and-move if >60s. Trust pass-1 instinct on review.

**Default tie-breakers when genuinely unsure:** managed > custom · simpler > complex · purpose-built > general · AWS-native > self-hosted · serverless > provisioned (for spiky/low-ops).

---

## 0b. Killer Distractors — the "one word that kills it" table

*These are the exact trap pairs that cost you on Mock-1 (🔴) plus the highest-frequency look-alikes. Memorize the disqualifier.*

| You're tempted by… | …but pick this instead when the stem says | Disqualifying word |
|---|---|---|
| 🔴 SNS | **SQS** | "buffer / lost data / **replay** / can't lose messages" — SNS doesn't store |
| 🔴 Cognito **Identity** Pool | **User Pool** | "**authentication** / sign-in / user directory" (Identity Pool = AWS creds for already-authed users) |
| 🔴 Dedicated **Hosts** | **Dedicated Instances** | "cheapest single-tenant" (Hosts cost more — only for **per-socket BYOL licensing** visibility) |
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
| Strongly Consistent read | (not available) | the read is **on a GSI** (GSIs are eventually consistent only) |

---

## 0c. Numbers You Must Know Cold

*Exams love exact numbers as distractor bait. These come up most.*

| Number | What it is |
|---|---|
| **720 / 1000 (~72%)** | Passing score. 65 questions, 130 min, 50 scored + 15 unscored |
| **256 KB** | Max SQS message size (bigger → **Extended Client** → S3) |
| **30 sec / 12 h** | SQS visibility timeout default / max |
| **20 sec** | SQS long-polling `WaitTimeSeconds` (cuts empty receives) |
| **0–15 min** | SQS Delay Queue / Message Timer range |
| **14 days** | SQS max message retention (default 4 days) |
| **15 min** | Lambda max timeout (longer → ECS/Batch/Fargate) |
| **10 GB** | Lambda container image max · `/tmp` configurable 512 MB–10 GB (ephemeral) |
| **/tmp = 512 MB–10 GB** | Lambda ephemeral scratch; **persistent shared** → EFS or S3 |
| **5 / 15** | Max RDS read replicas / Aurora read replicas |
| **5** | Aurora reader **regions** (Global DB) · also LSI per table |
| **20** | GSI per DynamoDB table |
| **72 h** | Aurora Backtrack window |
| **<1 sec** | Aurora Global DB cross-region replication lag |
| **60–120 sec** | RDS Multi-AZ failover time |
| **35 days** | RDS automated backup max retention (default 7) |
| **1 day–365 days** | Kinesis Data Streams retention |
| **~60 sec** | Firehose buffer/delivery latency |
| **2 MB/s** | Kinesis Enhanced Fan-Out throughput **per consumer** |
| **1 MB/s in, 2 MB/s out** | Kinesis throughput **per shard** |
| **1024–65535** | Ephemeral port range (NACL outbound replies) |
| **4 (2×2)** | Max DX resiliency: 2 connections × 2 locations |
| **us-east-1** | Where ACM cert for **CloudFront** must live |
| **5** | Targets per EventBridge rule |
| **7 / AZ** | Spread Placement Group instance cap |
| **256,000 IOPS** | io2 Block Express max |
| **0** | Cost of **inbound** data transfer to AWS, and of **Gateway** VPC endpoints |

---

## 0d. Confusable Pairs — "don't swap these"

| Pair | The distinction |
|---|---|
| **SG vs NACL** | SG = stateful, allow-only, ENI level · NACL = stateless, allow+deny, subnet level |
| **Multi-AZ vs Read Replica** | survive failure (sync, not readable) · scale reads (async, readable, cross-region) |
| **Gateway vs Interface Endpoint** | S3/DynamoDB, free, route-table · everything else, PrivateLink, hourly $ + ENI |
| **User Pool vs Identity Pool** | sign-in / user directory · hand out temp AWS creds to authed users |
| **Task Role vs Execution Role** | what your app can do · what ECS needs to start the task (ECR pull, logs) |
| **Reserved vs Provisioned Concurrency** | cap a function's share · pre-warm to kill cold starts |
| **Grace Period vs Warm Pool vs Cooldown** | wait before health-checking new instance · pre-initialized instances ready to flip · wait between scaling actions |
| **CloudFront vs Global Accelerator** | cache HTTP content at edge · TCP/UDP anycast IP, fast failover, no caching |
| **CloudFront Functions vs Lambda@Edge** | sub-ms, viewer-only, header/URL rewrite · heavier, all 4 events, network/body access, ≤30s |
| **CloudWatch vs CloudTrail vs Config** | performance/metrics/logs · who made which API call · resource config state + compliance |
| **Cluster vs Spread vs Partition PG** | low latency (1 AZ) · max isolation (≤7/AZ) · HDFS/Kafka racks |
| **Standard vs FIFO SQS** | high TPS, best-effort order, at-least-once · ordered, exactly-once, 300 msg/s |
| **Standard vs Convertible RI** | sell/modify-size, locked family · change family, can't sell |
| **Compute vs EC2 Instance Savings Plan** | any compute/region/family · locked to instance family, deeper discount |
| **Inspector vs GuardDuty vs Macie** | scan EC2/ECR vulns · detect threats from logs · find PII in S3 |
| **Shield Std vs Advanced vs WAF** | free L3/4 DDoS · paid + DDoS cost protection + DRT · L7 rules (SQLi/rate-limit) |
| **Secrets Manager vs Parameter Store** | auto-rotates secrets ($) · free config/secret storage (no auto-rotate) |
| **KMS vs CloudHSM** | AWS-managed multi-tenant keys · your dedicated single-tenant HSM, FIPS 140-2 L3 |
| **DataSync vs Storage Gateway** | move/sync data (migration) · ongoing hybrid bridge |
| **Snowball vs Snowmobile** | ~80 TB device · 100 PB truck |
| **Rehost/Replatform/Repurchase/Refactor** | lift-shift (MGN) · tweak (→RDS) · buy SaaS · re-architect cloud-native |

---

## 0e. Cross-Service "Which One Wins" — span-the-chapters triggers

*When the answer requires choosing across categories.*

| Scenario | Winner | Why not the others |
|---|---|---|
| Speed up **uploads** of large files to S3 from far away | **S3 Transfer Acceleration** | CloudFront = downloads/cache; GA = TCP/UDP apps |
| Speed up **dynamic API / global app** failover, non-HTTP | **Global Accelerator** | CloudFront caches static; Route 53 is DNS (clients cache it) |
| Cache **static** content globally | **CloudFront** | GA doesn't cache |
| **Replay** a stream for 7 days / multiple consumers | **Kinesis Data Streams** | SQS deletes on consume; Firehose no replay |
| Load streaming data to S3/Redshift, **no code** | **Firehose** | KDS needs consumer code |
| **New** decoupling between microservices | **SQS / SNS** | Amazon MQ only for migrating existing JMS/AMQP |
| **Migrate existing** ActiveMQ/RabbitMQ | **Amazon MQ** | SQS/SNS aren't protocol-compatible |
| React to **AWS service events** (EC2 state, S3 PUT) | **EventBridge** | SNS is for your own pub/sub fan-out |
| **Fan-out** one message to many queues | **SNS → SQS** | EventBridge if you need filtering/AWS events |
| Coordinate **multi-step** workflow w/ retries | **Step Functions** | not SQS chaining |
| Job **> 15 min** in a container | **ECS/Fargate/Batch** | Lambda caps at 15 min |
| **Private** S3/DynamoDB access, free | **Gateway Endpoint** | Interface endpoint costs hourly |
| **Private** access to your own service across VPCs | **NLB + PrivateLink** | peering exposes whole CIDR |
| Encrypt traffic **over Direct Connect** | **VPN over DX** | DX alone is private but **not encrypted** |
| **SQL on S3**: one object vs whole bucket | **S3 Select** (one) · **Athena** (many) | — |
| Eliminate **DB connection storms** from Lambda | **RDS Proxy** | not bigger instance |
| **Microsecond** reads on DynamoDB | **DAX** | ElastiCache is for RDS/general cache |
| **In-memory DB** that's durable (not just cache) | **MemoryDB** | ElastiCache Redis is a cache |

---

## 1. Networking

| Trigger | Answer |
|---|---|
| Block a specific IP | <details><summary>reveal</summary>**NACL** (SG is allow-only)</details> |
| Private S3 access, free | <details><summary>reveal</summary>**Gateway Endpoint**</details> |
| Private Kinesis/SQS/CloudWatch access | <details><summary>reveal</summary>**Interface Endpoint** (PrivateLink, hourly $)</details> |
| Connect 2 VPCs | <details><summary>reveal</summary>**VPC Peering** (no CIDR overlap, not transitive)</details> |
| Connect 50 VPCs | <details><summary>reveal</summary>**Transit Gateway** (hub-and-spoke, transitive)</details> |
| 🔴 Central VPC for shared DNS/AD/monitoring used by many VPCs | <details><summary>reveal</summary>**Shared Services VPC** (on TGW hub; *not* legacy Transit VPC)</details> |
| Private instances → outbound internet | <details><summary>reveal</summary>**NAT Gateway** in public subnet</details> |
| Bastion Host | <details><summary>reveal</summary>Public-subnet EC2 for **inbound SSH jump**; SG = your IP only</details> |
| Most secure private-instance access (no SSH) | <details><summary>reveal</summary>**SSM Session Manager** (no open ports, IAM, logged)</details> |
| Expose your service to other VPCs privately | <details><summary>reveal</summary>**NLB + PrivateLink (Endpoint Service)**</details> |
| Encrypt Direct Connect | <details><summary>reveal</summary>**Site-to-Site VPN over DX** (DX is private, NOT encrypted)</details> |
| Max DX resiliency | <details><summary>reveal</summary>**2 connections × 2 locations = 4 total**</details> |
| IPv6 outbound only | <details><summary>reveal</summary>**Egress-Only Internet Gateway**</details> |
| Deep packet inspection at VPC level | <details><summary>reveal</summary>**AWS Network Firewall** (Flow Logs = metadata only)</details> |
| Increase VPN bandwidth | <details><summary>reveal</summary>**ECMP via Transit Gateway**</details> |
| Hybrid DNS (on-prem ↔ AWS) | <details><summary>reveal</summary>**Route 53 Resolver** (Inbound = on-prem→AWS, Outbound = AWS→on-prem)</details> |
| Share subnets across accounts | <details><summary>reveal</summary>**RAM** (Resource Access Manager)</details> |
| Remote user VPN to AWS | <details><summary>reveal</summary>**Client VPN** (S2S = network↔network, Client = user↔AWS)</details> |
| Custom NACL setup | <details><summary>reveal</summary>Deny-all by default; remember **ephemeral ports 1024-65535** outbound</details> |
| 🔴 SG outbound rule port (web→DB) | <details><summary>reveal</summary>**= the destination's listening port** (1433/3306/5432), NOT 443 — and reference the **source SG**, not a CIDR</details> |

**3-tier SG chain:** Web SG ← `0.0.0.0/0:443` · App SG ← **Web-SG** on app port · DB SG ← **App-SG** on DB port. (SGs are stateful → never open the ephemeral reply port.)

---

## 2. Compute

| Trigger | Answer |
|---|---|
| Lowest latency between instances | <details><summary>reveal</summary>**Cluster Placement Group**</details> |
| Maximum fault isolation | <details><summary>reveal</summary>**Spread PG** (≤7/AZ) or **Partition PG** for HDFS/Kafka</details> |
| Cheapest for fault-tolerant batch | <details><summary>reveal</summary>**Spot** (~70-90% off)</details> |
| T2/T3 slow during business hours | <details><summary>reveal</summary>Credits exhausted → **T2/T3 Unlimited** or move to **M5**</details> |
| Per-socket / BYOL licensing | <details><summary>reveal</summary>**Dedicated Hosts**</details> |
| Cheapest single-tenant (no licensing need) | <details><summary>reveal</summary>**Dedicated Instances** (cheaper than Hosts)</details> |
| HPC sub-millisecond network | <details><summary>reveal</summary>**EFA** (Elastic Fabric Adapter — *network*, not storage)</details> |
| Preserve private IP on failover | <details><summary>reveal</summary>**Attach ENI** to new instance</details> |
| Resume app with RAM intact | <details><summary>reveal</summary>**Hibernate**</details> |
| Auto-install software at launch | <details><summary>reveal</summary>**User Data** script</details> |
| 🔴 User data privileges / when it runs | <details><summary>reveal</summary>Runs **as root**, **once, at first boot** only</details> |
| EC2 → S3 without hardcoded keys | <details><summary>reveal</summary>**IAM Instance Profile**</details> |
| Guarantee capacity in an AZ | <details><summary>reveal</summary>**Capacity Reservation** (not a discount)</details> |
| Static public IP across stop/start | <details><summary>reveal</summary>**Elastic IP**</details> |
| Prevent SSRF / metadata theft | <details><summary>reveal</summary>**IMDSv2** (token-based)</details> |
| SSH without key pairs | <details><summary>reveal</summary>**EC2 Instance Connect**</details> |
| ASG "launch config"? | <details><summary>reveal</summary>**Launch Template** (Launch Configuration is legacy)</details> |

**Launch Template vs User Data vs Lifecycle Hook — the one that trips you:**

| Trigger | Answer |
|---|---|
| Define **what** the instance looks like (AMI, type, SG, IAM role, contains user data) | <details><summary>reveal</summary>**Launch Template**</details> |
| Run a script the **first time** the instance boots (install Apache, register service) | <details><summary>reveal</summary>**User Data** (a field *inside* the Launch Template)</details> |
| **Pause** the instance during scale-out/in to do work (drain conns, copy logs, custom warm-up) | <details><summary>reveal</summary>**Lifecycle Hook**</details> |
| "Copy logs before termination" | <details><summary>reveal</summary>**Lifecycle Hook** (terminating)</details> |
| "Pre-warm cache before serving traffic" | <details><summary>reveal</summary>**Lifecycle Hook** (pending — user data already finished)</details> |
| "Install software on boot" | <details><summary>reveal</summary>**User Data**</details> |
| "Versioned instance config" / "standardize across ASG" | <details><summary>reveal</summary>**Launch Template**</details> |

*Mental model: **Launch Template** = the recipe · **User Data** = a step inside the recipe that runs at first boot · **Lifecycle Hook** = ASG hits pause so you can act before the instance joins or leaves.*

---

## 3. High Availability

| Trigger | Answer |
|---|---|
| Route based on URL path / host | <details><summary>reveal</summary>**ALB**</details> |
| Static IP, extreme TPS, TCP/UDP | <details><summary>reveal</summary>**NLB**</details> |
| 3rd-party firewall traffic inspection (GENEVE) | <details><summary>reveal</summary>**GWLB**</details> |
| Invoke Lambda from HTTP | <details><summary>reveal</summary>**ALB**</details> |
| Stateful app on multiple instances | <details><summary>reveal</summary>**ALB + Stickiness**</details> |
| Maintain CPU at 60% | <details><summary>reveal</summary>**Target Tracking Scaling**</details> |
| Scale every Monday 9 AM | <details><summary>reveal</summary>**Scheduled Scaling**</details> |
| Scale before a predicted spike | <details><summary>reveal</summary>**Predictive Scaling**</details> |
| Let in-flight requests finish before terminate | <details><summary>reveal</summary>**Connection Draining** (Deregistration Delay)</details> |
| Scale workers based on queue depth | <details><summary>reveal</summary>**SQS `ApproximateNumberOfMessagesVisible` + Target Tracking**</details> |
| App behind ALB needs original client IP | <details><summary>reveal</summary>**`X-Forwarded-For`** header</details> |
| Roll out new AMI across ASG without downtime | <details><summary>reveal</summary>**Instance Refresh** (set min healthy %)</details> |
| NLB cross-zone — free? | <details><summary>reveal</summary>**No** (NLB inter-AZ is **paid**; ALB cross-zone is free)</details> |
| Authenticate users at the LB | <details><summary>reveal</summary>**ALB + OIDC/Cognito User Pool**</details> |
| 🔴 New instance fails health checks at startup (slow boot) | <details><summary>reveal</summary>**Increase Health Check Grace Period**</details> |
| 🔴 Why ASG won't terminate an unhealthy-looking instance | <details><summary>reveal</summary>Within **grace period** OR in **Impaired** status</details> |
| App takes 8 min to boot, scales too slow | <details><summary>reveal</summary>**Warm Pool**</details> |
| 🔴 ASG default termination order | <details><summary>reveal</summary>AZ with most instances → **oldest Launch Configuration** → closest billing hour</details> |
| 🔴 "4 instances must always be up, survive 1 AZ loss, cheapest" | <details><summary>reveal</summary>**3 AZs × 2 each** (=6) — losing one AZ still leaves 4</details> |

*Don't swap: **Grace Period** = wait before health-checking new instance · **Warm Pool** = pre-initialized instances ready to flip in seconds · **Cooldown** = wait between scaling actions.*

---

## 4. Storage — S3

| Trigger | Answer |
|---|---|
| SQL on a single S3 object | <details><summary>reveal</summary>**S3 Select** (across many → **Athena**)</details> |
| Replicate across regions | <details><summary>reveal</summary>**CRR** (requires Versioning on both sides)</details> |
| Protect against accidental delete | <details><summary>reveal</summary>**Versioning** (+ **MFA Delete**)</details> |
| Reduce cost over time, you set rules | <details><summary>reveal</summary>**Lifecycle Policy**</details> |
| Auto-archive cold objects, no retrieval fee, AWS decides | <details><summary>reveal</summary>**Intelligent-Tiering**</details> |
| Replicate **existing** objects | <details><summary>reveal</summary>**S3 Batch Replication** (normal replication = new only)</details> |
| WORM / regulatory / cannot delete | <details><summary>reveal</summary>**Object Lock — Compliance Mode** (or Glacier Vault Lock)</details> |
| Object Lock modes | <details><summary>reveal</summary>**Governance** = override w/ permission · **Compliance** = nobody, not even root</details> |
| Temporary access to private object | <details><summary>reveal</summary>**Pre-signed URL**</details> |
| Process file on upload | <details><summary>reveal</summary>**S3 Event Notification → Lambda** (multi-target → **EventBridge**)</details> |
| Near-real-time per-upload processing, decoupled | <details><summary>reveal</summary>**S3 Event → SQS → Lambda**</details> |
| Static site + HTTPS + custom domain | <details><summary>reveal</summary>**S3 + CloudFront (OAC) + Route 53 + ACM**</details> |
| Multiple apps, different policies on one bucket | <details><summary>reveal</summary>**S3 Access Points**</details> |
| Requester pays egress | <details><summary>reveal</summary>**S3 Requester Pays**</details> |
| Redact PII on retrieval | <details><summary>reveal</summary>**S3 Object Lambda**</details> |
| Bulk tag/copy/process existing objects | <details><summary>reveal</summary>**S3 Batch Operations**</details> |
| 503 errors with SSE-KMS | <details><summary>reveal</summary>**KMS quota exceeded**</details> |
| Speed up **uploads** from far-away users | <details><summary>reveal</summary>**S3 Transfer Acceleration** (NOT CloudFront)</details> |

**Classes hot → cold:** Standard → Intelligent-Tiering → Standard-IA → One Zone-IA → Glacier Instant → Glacier Flexible → Glacier Deep Archive.

## 4b. Storage — Block & File

| Trigger | Answer |
|---|---|
| High IOPS block storage | <details><summary>reveal</summary>**EBS io1/io2** (256K IOPS → **io2 Block Express**)</details> |
| High throughput / streaming HDD | <details><summary>reveal</summary>**st1**</details> |
| Instance Store vs EBS | <details><summary>reveal</summary>Instance Store = **ephemeral, fastest** · EBS = persistent network</details> |
| EBS shared by 2+ EC2 in same AZ | <details><summary>reveal</summary>**EBS Multi-Attach** (io1/io2, cluster-aware app)</details> |
| Windows shared drive | <details><summary>reveal</summary>**FSx for Windows**</details> |
| Linux shared drive (NFS) | <details><summary>reveal</summary>**EFS**</details> |
| HPC / supercomputer FS | <details><summary>reveal</summary>**FSx for Lustre**</details> |
| 🔴 EFS, files rarely accessed, cut cost | <details><summary>reveal</summary>**EFS Lifecycle → Standard-IA** (single-AZ OK → **One Zone-IA**, cheapest)</details> |
| EFS axes (don't swap) | <details><summary>reveal</summary>**Performance** mode (GP / Max I/O) ≠ **Throughput** mode (Elastic/Provisioned/Bursting) ≠ **Storage class** (Standard/IA/One Zone)</details> |
| Schedule EBS backups | <details><summary>reveal</summary>**Data Lifecycle Manager (DLM)**</details> |
| No first-read latency on EBS snapshot | <details><summary>reveal</summary>**Fast Snapshot Restore (FSR)**</details> |
| Migrate NFS / sync on-prem → AWS | <details><summary>reveal</summary>**DataSync** (NFS/SMB/HDFS → S3/EFS/FSx)</details> |
| Existing SFTP/FTP workflow to S3 | <details><summary>reveal</summary>**Transfer Family**</details> |
| 🔴 Ship bulk data to Glacier cheapest | <details><summary>reveal</summary>**Snowball → S3 → Lifecycle to Glacier** (can't target Glacier directly)</details> |

---

## 5. Databases — Relational

| Trigger | Answer |
|---|---|
| Scale read workload | <details><summary>reveal</summary>**Read Replicas** (async, up to 5 RDS / 15 Aurora)</details> |
| Production HA / auto-failover | <details><summary>reveal</summary>**Multi-AZ** (sync, standby NOT readable)</details> |
| Standby NOT accessible | <details><summary>reveal</summary>**Multi-AZ**</details> |
| Standby IS accessible | <details><summary>reveal</summary>**Read Replica**</details> |
| 🔴 Minimize downtime during DB **engine upgrade** | <details><summary>reveal</summary>**Aurora Blue/Green** — Multi-AZ does NOT help (upgrades hit primary+standby together = downtime)</details> |
| Encrypt existing unencrypted DB | <details><summary>reveal</summary>**Snapshot → Copy w/ encryption → Restore** (only path)</details> |
| Eliminate DB password | <details><summary>reveal</summary>**IAM Database Authentication**</details> |
| Auto-scale DB storage | <details><summary>reveal</summary>**Storage Auto Scaling**</details> |
| 5× faster MySQL | <details><summary>reveal</summary>**Aurora**</details> |
| Variable / unpredictable load, auto-scale capacity + HA | <details><summary>reveal</summary>**Aurora Serverless v2**</details> |
| Global reads, RTO < 1 min DR | <details><summary>reveal</summary>**Aurora Global Database** (1 writer, up to 5 reader regions, <1s lag)</details> |
| Lambda exhausting DB connections | <details><summary>reveal</summary>**RDS Proxy**</details> |
| Quickly undo a bad data change (Aurora) | <details><summary>reveal</summary>**Backtrack** (up to 72h, in-place)</details> |
| 🔴 Fast dev/test copy of prod DB | <details><summary>reveal</summary>**Aurora Cloning** (copy-on-write) — NOT a Read Replica (replicas scale reads, aren't dev envs)</details> |
| Need OS-level DB access | <details><summary>reveal</summary>**RDS Custom** (Oracle / SQL Server only)</details> |
| Zero-downtime DB upgrade | <details><summary>reveal</summary>**Aurora Blue/Green Deployment**</details> |
| Performance bottleneck analysis | <details><summary>reveal</summary>**RDS Performance Insights**</details> |
| Alert on failover/maintenance | <details><summary>reveal</summary>**RDS Event Notifications → SNS**</details> |

**Multi-AZ vs Read Replica:**

| Feature | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | **HA / failover** | **Scale reads** |
| Replication | **Sync** | **Async** |
| Standby readable | No | **Yes** |
| Failover | **Auto (DNS)** | Manual promotion |
| Data loss | Zero | Possible (lag) |
| Cross-region | **No** | **Yes** |

**The HA-vs-scale-vs-clone cheat (memorize as a set):** Multi-AZ = *survive failure* · Read Replica = *scale reads* · Aurora Serverless v2 = *auto-scale capacity* · Aurora Cloning = *fast dev copy* · Snapshot→copy-encrypt→restore = *encrypt existing*.

---

## 6. Databases — NoSQL

| Trigger | Answer |
|---|---|
| Unpredictable DynamoDB workload | <details><summary>reveal</summary>**On-Demand** mode</details> |
| Query DynamoDB by non-PK attribute | <details><summary>reveal</summary>**GSI** (or **LSI** if same PK, different sort)</details> |
| Must read the very latest write | <details><summary>reveal</summary>**Strongly Consistent Reads** (2× cost, not on GSI)</details> |
| DynamoDB latency to microseconds | <details><summary>reveal</summary>**DAX**</details> |
| React to DynamoDB changes | <details><summary>reveal</summary>**DynamoDB Streams → Lambda**</details> |
| Multi-region active-active DynamoDB | <details><summary>reveal</summary>**Global Tables**</details> |
| Auto-delete old items | <details><summary>reveal</summary>**TTL**</details> |
| Atomic across multiple items | <details><summary>reveal</summary>**Transactions** (ACID)</details> |
| Analyze DynamoDB w/o consuming RCUs | <details><summary>reveal</summary>**Export to S3**</details> |
| SQL-like queries on DynamoDB | <details><summary>reveal</summary>**PartiQL**</details> |
| Cache with persistence/backups/HA | <details><summary>reveal</summary>**Redis**</details> |
| Simple multi-threaded cache | <details><summary>reveal</summary>**Memcached**</details> |
| Scale Redis horizontally | <details><summary>reveal</summary>**Cluster Mode Enabled**</details> |
| Cross-region Redis for DR | <details><summary>reveal</summary>**ElastiCache Global Datastore**</details> |
| Durable in-memory DB (not just cache) | <details><summary>reveal</summary>**MemoryDB for Redis**</details> |
| MongoDB migration | <details><summary>reveal</summary>**DocumentDB**</details> |
| Social network / fraud / recommendations | <details><summary>reveal</summary>**Neptune** (graph)</details> |
| IoT / time-series | <details><summary>reveal</summary>**Timestream**</details> |
| Immutable ledger / audit trail | <details><summary>reveal</summary>**QLDB** (not blockchain)</details> |
| Cassandra migration | <details><summary>reveal</summary>**Keyspaces**</details> |

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
| Decouple producers from consumers | <details><summary>reveal</summary>**SQS**</details> |
| Strict ordering, no duplicates | <details><summary>reveal</summary>**SQS FIFO** (Message Group ID = the thing you order *per*, e.g. Account ID)</details> |
| Highest throughput, duplicates OK | <details><summary>reveal</summary>**SQS Standard** (the default — best-effort order, at-least-once)</details> |
| Reduce SQS API calls / cost | <details><summary>reveal</summary>**Long Polling** (`WaitTimeSeconds = 20`)</details> |
| Message processed multiple times | <details><summary>reveal</summary>**Increase Visibility Timeout** (default 30s, max 12h)</details> |
| Handle messages failing repeatedly | <details><summary>reveal</summary>**Dead Letter Queue** (after `MaxReceiveCount` failures)</details> |
| 🔴 **SQS message larger than 256 KB** | <details><summary>reveal</summary>**SQS Extended Client Library** (payload → S3, reference → SQS)</details> |
| Delay ALL messages 0-15 min | <details><summary>reveal</summary>**Delay Queue**</details> |
| Delay specific messages | <details><summary>reveal</summary>**Message Timer** (per-message, overrides delay queue)</details> |
| Cross-account SQS access (S3 → SQS in another account) | <details><summary>reveal</summary>**SQS Access Policy** (resource-based)</details> |
| **SNS vs SQS** | <details><summary>reveal</summary>SNS = **push, pub/sub, many subscribers** · SQS = **pull, queue, one consumer group**</details> |
| Send same message to multiple services | <details><summary>reveal</summary>**SNS + SQS Fan-Out** (1 topic → N queues, each service gets its own buffer)</details> |
| Subscriber receives only certain message types | <details><summary>reveal</summary>**SNS Message Filtering** (JSON filter policy on attributes)</details> |
| Pub/sub with strict ordering | <details><summary>reveal</summary>**SNS FIFO Topic + SQS FIFO Queues** (FIFO topics can ONLY subscribe FIFO queues)</details> |
| Real-time (<1s) + replay last N days | <details><summary>reveal</summary>**Kinesis Data Streams** (retention 1d-365d)</details> |
| Load streaming data to S3/Redshift/OpenSearch, no code | <details><summary>reveal</summary>**Amazon Data Firehose** (~60s latency, auto-scales, optional Lambda transform)</details> |
| Scale Kinesis consumers | <details><summary>reveal</summary>**Add shards (split) + KCL workers** (max 1 worker per shard — extras sit idle)</details> |
| Dedicated 2 MB/s per consumer on Kinesis | <details><summary>reveal</summary>**Enhanced Fan-Out** (HTTP/2 push, no shared throughput)</details> |
| Kinesis vs SQS — "replay last 7 days" | <details><summary>reveal</summary>**Kinesis** (SQS deletes on consume; can't replay)</details> |
| Migrate on-prem RabbitMQ / ActiveMQ | <details><summary>reveal</summary>**Amazon MQ**</details> |
| New cloud-native app | <details><summary>reveal</summary>**SQS / SNS** (NOT Amazon MQ)</details> |
| React to AWS service events (EC2 state, S3 PUT, CloudTrail API call) | <details><summary>reveal</summary>**EventBridge**</details> |
| Schedule cron / rate-based task | <details><summary>reveal</summary>**EventBridge Scheduler**</details> |
| Event-driven with filtering, many targets, cross-account | <details><summary>reveal</summary>**EventBridge Rules** (up to 5 targets per rule)</details> |
| 🔴 **Replay past events for debugging** | <details><summary>reveal</summary>**EventBridge Archive & Replay** (NOT Firehose — Firehose is one-way load to S3, no replay)</details> |

**The "queue is never the destination" model:** Producer → SQS → Consumer fleet (Lambda / EC2 ASG / ECS-Fargate / on-prem) → final data store (DB / S3 / API). The queue is a **shock absorber** between spiky producers and steady consumers — scale consumers on `ApproximateNumberOfMessagesVisible`.

**🔴 = the 2 you missed on Day 6 (May 24). Drill these first.**

---

## 8. Serverless — Lambda / API Gateway / Step Functions

| Trigger | Answer |
|---|---|
| Task > 15 minutes | <details><summary>reveal</summary>**NOT Lambda** (ECS / Batch / Fargate)</details> |
| Lambda persistent storage | <details><summary>reveal</summary>**NOT /tmp** (EFS or S3) — /tmp is ephemeral</details> |
| Persistent **shared** storage across invocations | <details><summary>reveal</summary>**Mount EFS** (Lambda must be in VPC)</details> |
| Share libraries across many functions | <details><summary>reveal</summary>**Layers**</details> |
| Eliminate cold starts | <details><summary>reveal</summary>**Provisioned Concurrency** (NOT Reserved)</details> |
| Stop one function eating all concurrency | <details><summary>reveal</summary>**Reserved Concurrency**</details> |
| Lambda → RDS in private subnet | <details><summary>reveal</summary>**Attach Lambda to VPC** (gets ENI; + NAT for internet)</details> |
| Lambda can't access S3/DynamoDB | <details><summary>reveal</summary>Fix the **Execution Role** IAM permissions</details> |
| Route async results (success + failure) | <details><summary>reveal</summary>**Destinations** (DLQ = failures only)</details> |
| Lambda invoked by S3/SNS/EventBridge | <details><summary>reveal</summary>**Async** (retries 2×, then DLQ)</details> |
| Lambda reads from Kinesis/DynamoDB Stream/SQS | <details><summary>reveal</summary>**Event Source Mapping** (Lambda polls)</details> |
| Huge deps / existing container as Lambda | <details><summary>reveal</summary>**Container image** from ECR (≤10 GB)</details> |
| Cheapest / simplest API proxy | <details><summary>reveal</summary>**HTTP API** (~70% cheaper than REST)</details> |
| API needs caching / validation / WAF | <details><summary>reveal</summary>**REST API**</details> |
| Limit/meter API requests per customer | <details><summary>reveal</summary>**Usage Plans + API Keys**</details> |
| Browser blocks cross-origin API call | <details><summary>reveal</summary>Enable **CORS**</details> |
| User sign-in at the API | <details><summary>reveal</summary>**Cognito User Pools** · custom token logic → **Lambda Authorizer**</details> |
| Coordinate multi-step workflow (retries + visual) | <details><summary>reveal</summary>**Step Functions**</details> |
| Long-running workflow (hours/days) | <details><summary>reveal</summary>**Standard** · high-volume short → **Express** (≤5 min)</details> |

---

## 9. Containers — ECS / EKS / Fargate / ECR

| Trigger | Answer |
|---|---|
| Store Docker images privately | <details><summary>reveal</summary>**ECR** (+ Image Scanning for vulnerabilities)</details> |
| Run containers without managing servers | <details><summary>reveal</summary>**Fargate**</details> |
| Run containers on EC2 (use RIs/Spot) | <details><summary>reveal</summary>**ECS EC2 Launch Type**</details> |
| Zero infrastructure management, simplest | <details><summary>reveal</summary>**App Runner** (simpler than Fargate)</details> |
| Persistent shared storage for containers | <details><summary>reveal</summary>**EFS**</details> |
| Task-level Security Groups | <details><summary>reveal</summary>**awsvpc** network mode</details> |
| Ensure N containers always running | <details><summary>reveal</summary>**ECS Service** (Desired Count = N)</details> |
| Scale EC2 hosts when tasks are pending | <details><summary>reveal</summary>**Capacity Provider + ASG**</details> |
| Migrate Kubernetes to AWS | <details><summary>reveal</summary>**EKS**</details> |
| Simple AWS-native orchestration | <details><summary>reveal</summary>**ECS**</details> |
| Container needs S3/DynamoDB access | <details><summary>reveal</summary>**Task Role** (NOT Execution Role)</details> |
| ECS agent needs to pull image from ECR | <details><summary>reveal</summary>**Execution Role** (NOT Task Role)</details> |

*Task Role vs Execution Role: **Task Role** = what your app code can do (S3, DynamoDB) · **Execution Role** = what ECS needs to start the task (pull ECR image, write logs).*

---

## 10. Content Delivery & DNS — Route 53 / CloudFront / Global Accelerator

| Trigger | Answer |
|---|---|
| Map root/apex domain to ALB | <details><summary>reveal</summary>**Alias Record** (NOT CNAME)</details> |
| Route % of traffic to new version | <details><summary>reveal</summary>**Weighted Routing**</details> |
| Route to fastest region | <details><summary>reveal</summary>**Latency-Based Routing**</details> |
| Automatic failover to DR site | <details><summary>reveal</summary>**Failover Routing** (requires Health Check)</details> |
| Different content by country | <details><summary>reveal</summary>**Geolocation Routing**</details> |
| Serve S3 via CloudFront, block direct access | <details><summary>reveal</summary>**Origin Access Control (OAC)**</details> |
| Restrict access to premium content | <details><summary>reveal</summary>**Signed URLs / Signed Cookies**</details> |
| Block access from specific countries | <details><summary>reveal</summary>**CloudFront Geo Restriction**</details> |
| Cache static content globally | <details><summary>reveal</summary>**CloudFront**</details> |
| Static IP, UDP, gaming, fast failover | <details><summary>reveal</summary>**Global Accelerator**</details> |
| Blue/green when clients DNS-cache aggressively | <details><summary>reveal</summary>**Global Accelerator** (static anycast IP, no DNS cache problem)</details> |
| Replicate S3 to a region (not edge cache) | <details><summary>reveal</summary>**S3 CRR** (NOT CloudFront)</details> |
| Manually clear CloudFront cache | <details><summary>reveal</summary>**Invalidation** (costs $; prefer versioned filenames)</details> |
| Simple URL rewrite / header tweak at edge | <details><summary>reveal</summary>**CloudFront Functions** (sub-ms, viewer only)</details> |
| Complex edge logic (network/body access) | <details><summary>reveal</summary>**Lambda@Edge** (all 4 events, up to 30s)</details> |
| CloudFront origin failover / HA | <details><summary>reveal</summary>**Origin Groups** (primary + secondary)</details> |
| Internal DNS within a VPC | <details><summary>reveal</summary>**Route 53 Private Hosted Zone** (associate with VPC)</details> |

---

## 11. Security & Identity — IAM / KMS / Cognito / WAF

| Trigger | Answer |
|---|---|
| EC2 needs S3 access without keys | <details><summary>reveal</summary>**IAM Role** (Instance Profile)</details> |
| Explicit Deny vs Allow | <details><summary>reveal</summary>**Deny always wins** (order: explicit Deny → Allow → default Deny)</details> |
| Limit max permissions a user/role can get | <details><summary>reveal</summary>**Permission Boundary** (on the principal — caps escalation; not an SCP, not a group)</details> |
| Restrict an action to one region / by IP | <details><summary>reveal</summary>IAM **condition** (`aws:RequestedRegion`, `aws:SourceIp`)</details> |
| Auto-rotate RDS password | <details><summary>reveal</summary>**Secrets Manager**</details> |
| Free secret storage | <details><summary>reveal</summary>**Parameter Store (Standard)**</details> |
| User sign-up / sign-in | <details><summary>reveal</summary>**Cognito User Pools**</details> |
| Mobile app uploads to S3 without backend | <details><summary>reveal</summary>**Cognito Identity Pools** (temp AWS creds)</details> |
| S3 IAM policy structure | <details><summary>reveal</summary>`ListBucket` on **bucket ARN** + `GetObject` on **bucket/\*** (two resources)</details> |
| Block SQL injection / rate-limit API | <details><summary>reveal</summary>**WAF**</details> |
| Free DDoS protection | <details><summary>reveal</summary>**Shield Standard**</details> |
| Detect compromised instances / threats | <details><summary>reveal</summary>**GuardDuty**</details> |
| Scan EC2/ECR for vulnerabilities | <details><summary>reveal</summary>**Inspector**</details> |
| Discover PII in S3 | <details><summary>reveal</summary>**Macie**</details> |
| Cross-account S3 access | <details><summary>reveal</summary>**Bucket Policy** (resource-based)</details> |
| Encrypt large files efficiently | <details><summary>reveal</summary>**Envelope Encryption (KMS)**</details> |
| Free SSL/TLS for ALB / CloudFront | <details><summary>reveal</summary>**ACM** (CloudFront → must be **us-east-1**)</details> |
| SSL cert on EC2 | <details><summary>reveal</summary>**NOT ACM** — CloudHSM or import manually</details> |
| FIPS 140-2 Level 3 / dedicated HSM | <details><summary>reveal</summary>**CloudHSM**</details> |
| SSO across multiple AWS accounts | <details><summary>reveal</summary>**IAM Identity Center**</details> |
| Centralized security findings | <details><summary>reveal</summary>**Security Hub**</details> |
| Manage WAF rules across all accounts | <details><summary>reveal</summary>**Firewall Manager**</details> |
| Check config compliance (e.g. cert is ACM-issued not imported) | <details><summary>reveal</summary>**AWS Config** managed rule</details> |
| Federate corporate AD (SAML) | <details><summary>reveal</summary>**AssumeRoleWithSAML**</details> |
| Social login / web identity | <details><summary>reveal</summary>**AssumeRoleWithWebIdentity** (Cognito Identity Pools)</details> |
| Use existing on-prem AD with AWS | <details><summary>reveal</summary>**AD Connector** (proxy) or trust with **Managed Microsoft AD**</details> |
| Full managed AD for Windows/SQL | <details><summary>reveal</summary>**AWS Managed Microsoft AD**</details> |

---

## 12. Monitoring & Audit — CloudWatch / CloudTrail / Config

| Trigger | Answer |
|---|---|
| Monitor EC2 memory / disk (not default) | <details><summary>reveal</summary>**CloudWatch Agent** (custom metrics)</details> |
| Alert when metric crosses threshold | <details><summary>reveal</summary>**CloudWatch Alarm**</details> |
| Run Lambda on a schedule | <details><summary>reveal</summary>**EventBridge Scheduled Rule**</details> |
| Who did what / API call audit | <details><summary>reveal</summary>**CloudTrail** (enable Data Events for object-level)</details> |
| Was a resource configured a certain way? | <details><summary>reveal</summary>**AWS Config** (config history)</details> |
| Enforce + auto-fix resource compliance | <details><summary>reveal</summary>**Config Rule + Remediation**</details> |
| Query logs for patterns | <details><summary>reveal</summary>**CloudWatch Logs Insights**</details> |
| Alarm on a log pattern | <details><summary>reveal</summary>**Metric Filter + Alarm**</details> |
| Stream logs to OpenSearch real-time | <details><summary>reveal</summary>**Logs Subscription Filter**</details> |
| Debug latency across microservices | <details><summary>reveal</summary>**X-Ray** (Service Map)</details> |
| Identify underutilized resources | <details><summary>reveal</summary>**Trusted Advisor**</details> |
| **CloudWatch vs CloudTrail vs Config** | <details><summary>reveal</summary>CloudWatch = **performance/metrics/logs** · CloudTrail = **who made API calls** · Config = **resource config state + compliance**</details> |

---

## 13. Cost / Migration / DR

| Trigger | Answer |
|---|---|
| Analyze spending / forecast | <details><summary>reveal</summary>**Cost Explorer**</details> |
| Alert when budget exceeded | <details><summary>reveal</summary>**AWS Budgets**</details> |
| Most granular line-item billing → S3 | <details><summary>reveal</summary>**Cost and Usage Report (CUR)**</details> |
| Right-size EC2 | <details><summary>reveal</summary>**Compute Optimizer**</details> |
| Flexible savings across regions/compute | <details><summary>reveal</summary>**Compute Savings Plans**</details> |
| Highest discount, locked to family | <details><summary>reveal</summary>**EC2 Instance Savings Plans / Standard RI**</details> |
| Change instance family mid-term | <details><summary>reveal</summary>**Convertible RI**</details> |
| Sell unused RI | <details><summary>reveal</summary>**Standard RI only** (Convertible can't)</details> |
| 🔴 Predictable baseline + spiky peaks, cheapest | <details><summary>reveal</summary>**RIs/Savings Plans for baseline + Spot/On-Demand for peak** (not 100% of either)</details> |
| Inbound data transfer to AWS | <details><summary>reveal</summary>**FREE**</details> |
| Cut S3/DynamoDB cost from VPC | <details><summary>reveal</summary>**Gateway VPC Endpoint** (free)</details> |
| Lift-and-shift to AWS, minimal change | <details><summary>reveal</summary>**Rehost** (MGN)</details> |
| Move DB to managed service | <details><summary>reveal</summary>**Replatform** (e.g. → RDS)</details> |
| Replace with SaaS | <details><summary>reveal</summary>**Repurchase**</details> |
| Re-architect cloud-native | <details><summary>reveal</summary>**Refactor**</details> |
| Migrate DB minimal downtime | <details><summary>reveal</summary>**DMS** (+ SCT if heterogeneous)</details> |
| SFTP/FTP files to S3 | <details><summary>reveal</summary>**Transfer Family**</details> |
| Online NFS/SMB → S3 sync | <details><summary>reveal</summary>**DataSync**</details> |
| Offline bulk transfer (TB-PB) | <details><summary>reveal</summary>**Snow Family** (Snowball / Snowmobile / Snowcone)</details> |
| Cheapest DR | <details><summary>reveal</summary>**Backup & Restore**</details> |
| DR with Route 53 failover + always-on small ASG | <details><summary>reveal</summary>**Warm Standby**</details> |
| Near-zero RTO DR | <details><summary>reveal</summary>**Multi-Site Active-Active**</details> |
| Centralized cross-account backup | <details><summary>reveal</summary>**AWS Backup**</details> |
| Restrict services across accounts | <details><summary>reveal</summary>**SCPs (Organizations)**</details> |
| Track migration progress | <details><summary>reveal</summary>**Migration Hub**</details> |
| Break down costs by team/project | <details><summary>reveal</summary>**Cost Allocation Tags**</details> |

*DR order (cheapest/slowest → priciest/fastest):* **Backup & Restore → Pilot Light → Warm Standby → Multi-Site Active-Active.**

---

## 14. Additional Services — Analytics / IaC / ML

| Trigger | Answer |
|---|---|
| Query S3 data with SQL | <details><summary>reveal</summary>**Athena**</details> |
| Data warehouse / OLAP | <details><summary>reveal</summary>**Redshift** (query S3 → **Redshift Spectrum**)</details> |
| ETL / transform data | <details><summary>reveal</summary>**Glue** (auto-schema → **Crawlers**; metadata → **Data Catalog**)</details> |
| Reduce Athena cost | <details><summary>reveal</summary>**Parquet/ORC columnar + partitioning**</details> |
| Convert .csv → .parquet | <details><summary>reveal</summary>**Glue** (ETL job)</details> |
| Big data / Spark / Hadoop | <details><summary>reveal</summary>**EMR**</details> |
| Dashboards / BI | <details><summary>reveal</summary>**QuickSight**</details> |
| Data lake governance / column security | <details><summary>reveal</summary>**Lake Formation**</details> |
| Full-text search / log analytics | <details><summary>reveal</summary>**OpenSearch**</details> |
| Infrastructure as Code | <details><summary>reveal</summary>**CloudFormation** (multi-account/region → **StackSets**)</details> |
| Detect manual infra changes | <details><summary>reveal</summary>**CloudFormation Drift Detection**</details> |
| Deploy web app, minimal management | <details><summary>reveal</summary>**Elastic Beanstalk**</details> |
| GraphQL / real-time data sync | <details><summary>reveal</summary>**AppSync**</details> |
| Migrate Kafka to AWS | <details><summary>reveal</summary>**MSK** (new streaming app → **Kinesis**)</details> |
| Long-running batch jobs in Docker | <details><summary>reveal</summary>**AWS Batch** (NOT Lambda)</details> |
| Analyze images / detect faces | <details><summary>reveal</summary>**Rekognition**</details> |
| Extract text from documents (OCR) | <details><summary>reveal</summary>**Textract**</details> |
| Sentiment / NLP | <details><summary>reveal</summary>**Comprehend**</details> |
| Build chatbot | <details><summary>reveal</summary>**Lex**</details> |
| Custom ML model | <details><summary>reveal</summary>**SageMaker**</details> |
| AWS services on-premises | <details><summary>reveal</summary>**Outposts**</details> |
| Low latency to a specific city | <details><summary>reveal</summary>**Local Zones**</details> |
| 5G edge computing | <details><summary>reveal</summary>**Wavelength**</details> |

---

## 🎯 Final pre-mock checklist (run §0 → §0e in 10 min)

- [ ] Read **§0 Tactics** — circle the qualifier, predict before reading, name the disqualifier.
- [ ] Skim **§0b Killer distractors** — the 🔴 rows are your proven weak spots.
- [ ] Glance at **§0c Numbers** — 720 pass, 256 KB, 15 min, 60-120s, <1s, us-east-1.
- [ ] After the mock: log score in StudyGuide.md tracker + add every miss to your paper weak-list.
- [ ] Re-read the chapter section for each miss **same day** (wrong-answer review is the highest-value hour).
