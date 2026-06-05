# SAA-C03 — 30-Minute Pre-Mock Recall

> **Run this once, start to finish, right before each mock (Jun 8 · 9 · 10). It is built to take ~30 min and then STOP.**
> Cover the right column, read the trigger, *say the answer out loud before you uncover it*. If you blank → mark it, move on (don't re-read now). Saying it out loud = retrieval = the thing that actually moves your score; silent reading does not.
>
> **This is a recall drill, not a reference.** It's deliberately incomplete — only the highest-yield ~110 triggers + the traps you actually fell for. For depth on anything you miss, the full reference is **Recall.md** (use it *after* the mock, on wrong answers — not now).
>
> **Time budget:** Tactics 4 min · 🔴 Traps 6 min · 14 chapters 18 min · Numbers skim 2 min. **If you run over 30, stop anyway** — finishing matters more than completeness.

---

## ⏱️ Block 1 — Tactics (4 min) — *read, don't memorize*

**Your Mock-1 data: 72% of misses were test-taking, not knowledge.** So this block is worth more than any fact below.

1. **Circle the qualifier before reading answers** → *MOST cost-effective · LEAST operational overhead · MOST secure · real-time vs near-real-time vs batch · NOT/EXCEPT · minimal code.*
2. **Predict the answer before looking at the options.** Then find it.
3. **Between the final two, say the ONE word that kills the wrong one out loud.** (This single habit attacks most of your misses.)
4. **>60 sec on a question → flag and move.** Come back. Trust your first instinct on review.

**Tie-breakers when truly stuck:** managed > custom · serverless > provisioned (spiky/low-ops) · purpose-built > general · private > public · deny by default.

---

## ⏱️ Block 2 — 🔴 Your Traps (6 min) — *the highest-value 6 minutes*

*These are the exact discriminations you got wrong on Mock-1. Cover the right side; for each, say what disqualifies the tempting one.*

| Tempting wrong answer | Right answer when… | The disqualifying word |
|---|---|---|
| SNS | **SQS** | "buffer / can't lose / **replay**" — SNS doesn't store |
| Cognito **Identity** Pool | **User Pool** | "**authentication** / sign-in" |
| Dedicated **Hosts** | **Dedicated Instances** | "**cheapest** single-tenant" (Hosts = per-socket BYOL only) |
| EFS **Bursting** (throughput) | **Max I/O** (performance) | "big-data throughput" — wrong axis |
| Macie | **S3 Access Points** | "**enforce** access" (Macie only *finds* PII) |
| oldest Launch **Template** | oldest Launch **Configuration** | "ASG default termination order" |
| Read Replica as dev DB | **Aurora Cloning** | "clone for **dev/test**" |
| "rely on Multi-AZ" | **Aurora Blue/Green** | "minimize downtime during **engine upgrade**" |
| Firehose | **Kinesis Data Streams** | "**replay** / retention / multiple consumers" |
| CloudFront | **Global Accelerator** | "TCP/UDP / gaming / static IP / DNS-cache failover" |
| CNAME | **Alias** | "**root/apex** domain" |
| Reserved Concurrency | **Provisioned Concurrency** | "eliminate **cold starts**" |
| Task **Execution** Role | Task **Role** | "the **app** needs S3/DynamoDB" |
| Multi-AZ | **Read Replica** | "standby **readable** / scale reads / **cross-region**" |

**And the 6 facts that had no home before (now drill them):**

| Trigger | Answer |
|---|---|
| EC2 user data — privileges + when | **as root**, **once at first boot** |
| EFS, files rarely accessed, cut cost | **EFS Lifecycle → Standard-IA** (single-AZ OK → One Zone-IA) |
| Ship bulk data to Glacier cheapest | **Snowball → S3 → Lifecycle to Glacier** (can't target Glacier directly) |
| SG outbound port (web→DB) | **= destination's listening port** (1433/3306/5432), not 443; source = the **SG**, not a CIDR |
| Central VPC for shared DNS/AD/logging | **Shared Services VPC** (on TGW) |
| Minimize downtime, DB **engine upgrade** | **Aurora Blue/Green** (Multi-AZ upgrade = downtime) |

---

## ⏱️ Block 3 — Top triggers, 14 chapters (18 min) — *~75 sec per chapter*

### 1. Networking
| Trigger | Answer |
|---|---|
| Block a specific IP | **NACL** (SG allow-only) |
| Private S3/DynamoDB, free | **Gateway Endpoint** (else **Interface**/PrivateLink, $) |
| Connect 2 VPCs / 50 VPCs | **Peering** (not transitive) / **Transit Gateway** (transitive) |
| Private instances → internet out | **NAT Gateway** (public subnet) · IPv6 → **Egress-Only IGW** |
| Most secure private access, no SSH | **SSM Session Manager** |
| Expose your service to other VPCs privately | **NLB + PrivateLink** |
| Encrypt Direct Connect | **VPN over DX** (DX alone = private, NOT encrypted) |
| Hybrid DNS on-prem↔AWS | **Route 53 Resolver** |

### 2. Compute
| Trigger | Answer |
|---|---|
| Lowest latency between instances | **Cluster Placement Group** |
| Cheapest fault-tolerant batch | **Spot** |
| Per-socket / BYOL licensing | **Dedicated Hosts** (cheapest single-tenant = **Instances**) |
| Preserve private IP on failover | **Attach ENI** |
| Resume app w/ RAM intact | **Hibernate** |
| EC2 → S3 no keys | **IAM Instance Profile** |
| Prevent metadata theft (SSRF) | **IMDSv2** |
| "ASG launch config" | **Launch Template** (Config is legacy) |

### 3. High Availability
| Trigger | Answer |
|---|---|
| Route by URL path/host | **ALB** · static IP/TCP/UDP → **NLB** · 3rd-party firewall → **GWLB** |
| Keep CPU at 60% | **Target Tracking** · weekly → **Scheduled** · predicted → **Predictive** |
| Scale on queue depth | **`ApproximateNumberOfMessagesVisible`** + Target Tracking |
| New AMI across ASG, no downtime | **Instance Refresh** |
| Auth users at the LB | **ALB + Cognito User Pool** |
| Slow-booting instance fails health check | **Increase Grace Period** · 8-min boot → **Warm Pool** |
| ASG won't terminate instance | within **grace period** OR **Impaired** status |
| ASG default termination order | most-instances AZ → **oldest Launch Config** → billing hour |

### 4. Storage
| Trigger | Answer |
|---|---|
| SQL on 1 object / many objects | **S3 Select** / **Athena** |
| Replicate cross-region | **CRR** (needs Versioning both sides) |
| Cost over time (you set rules) / AWS decides | **Lifecycle** / **Intelligent-Tiering** |
| WORM / can't delete | **Object Lock Compliance** |
| Temp access to private object | **Pre-signed URL** |
| Process file on upload (near-real-time) | **S3 Event → SQS → Lambda** |
| High IOPS / throughput-HDD block | **io1/io2** / **st1** |
| Windows / Linux / HPC shared FS | **FSx Windows** / **EFS** / **FSx Lustre** |
| EFS cold files cheaper | **Lifecycle → Standard-IA** (axes: performance ≠ throughput ≠ class) |
| Migrate/sync NFS-SMB → AWS | **DataSync** · ongoing hybrid bridge → **Storage Gateway** |

### 5. Databases — Relational
| Trigger | Answer |
|---|---|
| Scale reads / production HA | **Read Replica** (async, readable) / **Multi-AZ** (sync, not readable) |
| Encrypt existing DB | **snapshot → copy-encrypt → restore** (only path) |
| Unpredictable load, auto-scale + HA | **Aurora Serverless v2** |
| Global reads, RTO<1min | **Aurora Global Database** |
| Lambda exhausting connections | **RDS Proxy** |
| Fast dev/test copy | **Aurora Cloning** (NOT a Read Replica) |
| Undo bad data change (Aurora) | **Backtrack** (72h) |
| Zero-downtime upgrade | **Aurora Blue/Green** |

### 6. Databases — NoSQL
| Trigger | Answer |
|---|---|
| Unpredictable DynamoDB load | **On-Demand** |
| Query by non-PK | **GSI** (eventual only) / **LSI** (same PK, at create) |
| Microsecond reads | **DAX** |
| React to changes | **Streams → Lambda** |
| Multi-region active-active | **Global Tables** · auto-delete old → **TTL** |
| Cache: persistence/HA vs simple | **Redis** vs **Memcached** |
| Durable in-memory DB | **MemoryDB** |
| MongoDB / Cassandra / graph / ledger | **DocumentDB** / **Keyspaces** / **Neptune** / **QLDB** |

### 7. Decoupling
| Trigger | Answer |
|---|---|
| Decouple producer/consumer | **SQS** · ordering+no-dupes → **FIFO** |
| Msg > 256 KB | **SQS Extended Client** (S3 + reference) |
| Processed multiple times | **↑ Visibility Timeout** · repeated failures → **DLQ** |
| Same msg to many services | **SNS + SQS Fan-Out** |
| Real-time <1s + replay | **Kinesis Data Streams** |
| Load stream to S3, no code | **Firehose** (~60s) |
| Migrate ActiveMQ/RabbitMQ / new app | **Amazon MQ** / **SQS-SNS** |
| React to AWS events / replay past events | **EventBridge** / **EventBridge Archive & Replay** |

### 8. Serverless
| Trigger | Answer |
|---|---|
| Task > 15 min | **NOT Lambda** (ECS/Batch/Fargate) |
| Persistent shared storage | **EFS** (Lambda in VPC) — /tmp is ephemeral |
| Eliminate cold starts / cap concurrency | **Provisioned** / **Reserved** Concurrency |
| Lambda can't reach S3/DynamoDB | fix **Execution Role** |
| Lambda → RDS private | **attach to VPC** |
| Cheap API proxy / caching+WAF | **HTTP API** / **REST API** |
| Multi-step workflow + retries | **Step Functions** (long → Standard, short-high-vol → Express) |

### 9. Containers
| Trigger | Answer |
|---|---|
| Store images privately | **ECR** |
| No servers / on EC2 (Spot/RI) | **Fargate** / **ECS EC2** |
| Simplest, zero infra | **App Runner** |
| Task-level SGs | **awsvpc** mode |
| App needs S3 vs pull image+logs | **Task Role** vs **Execution Role** |
| Migrate Kubernetes | **EKS** |

### 10. CDN & DNS
| Trigger | Answer |
|---|---|
| Apex domain → ALB | **Alias** (not CNAME) |
| % to new version / fastest region / DR failover | **Weighted** / **Latency** / **Failover** (needs health check) |
| Serve S3 via CloudFront, block direct | **OAC** |
| Premium content access | **Signed URLs/Cookies** |
| Static IP, UDP, fast failover, DNS-cache | **Global Accelerator** |
| Edge URL rewrite vs complex edge logic | **CloudFront Functions** vs **Lambda@Edge** |
| Origin HA | **Origin Groups** |

### 11. Security & Identity
| Trigger | Answer |
|---|---|
| Deny vs Allow | **Deny wins** (Deny → Allow → default Deny) |
| Cap max perms a user can get | **Permission Boundary** |
| Auto-rotate secret / free storage | **Secrets Manager** / **Parameter Store** |
| Sign-in vs temp AWS creds | **User Pool** vs **Identity Pool** |
| SQLi / rate-limit | **WAF** · free DDoS → **Shield Std** |
| Threats from logs / vuln scan / find PII | **GuardDuty** / **Inspector** / **Macie** |
| Cross-account S3 | **Bucket Policy** |
| Free TLS cert (CloudFront → us-east-1) | **ACM** · dedicated HSM/FIPS-L3 → **CloudHSM** |
| SSO multi-account | **IAM Identity Center** |
| Federate corporate AD (SAML) | **AssumeRoleWithSAML** |

### 12. Monitoring & Audit
| Trigger | Answer |
|---|---|
| EC2 memory/disk metrics | **CloudWatch Agent** |
| Who made which API call | **CloudTrail** (Data Events = object-level) |
| Resource config state + compliance | **Config** (+ Rule + Remediation to auto-fix) |
| Alarm on a log pattern | **Metric Filter + Alarm** |
| Debug latency across microservices | **X-Ray** |
| **The 3-way** | CloudWatch = perf/metrics · CloudTrail = API calls · Config = config state |

### 13. Cost / Migration / DR
| Trigger | Answer |
|---|---|
| Analyze spend / alert on budget / granular billing→S3 | **Cost Explorer** / **Budgets** / **CUR** |
| Flexible savings / locked-family / change-family | **Compute SP** / **EC2-Instance SP** / **Convertible RI** |
| Baseline + spiky peaks cheapest | **RI/SP for baseline + Spot/On-Demand for peak** |
| Lift-shift / →RDS / →SaaS / re-architect | **Rehost** / **Replatform** / **Repurchase** / **Refactor** |
| Migrate DB minimal downtime | **DMS** (+SCT if heterogeneous) |
| Offline bulk transfer | **Snow Family** |
| DR cheapest → fastest | **Backup&Restore → Pilot Light → Warm Standby → Active-Active** |
| Centralized cross-account backup / restrict services | **AWS Backup** / **SCPs** |

### 14. Additional Services
| Trigger | Answer |
|---|---|
| SQL on S3 / data warehouse / ETL | **Athena** / **Redshift** / **Glue** |
| Cut Athena cost / csv→parquet | **Parquet+partition** / **Glue** |
| Spark-Hadoop / BI dashboards / full-text search | **EMR** / **QuickSight** / **OpenSearch** |
| Infra as Code / detect drift | **CloudFormation** / **Drift Detection** |
| Deploy web app low-mgmt / long Docker batch | **Beanstalk** / **AWS Batch** |
| Images / OCR / sentiment / chatbot / custom ML | **Rekognition** / **Textract** / **Comprehend** / **Lex** / **SageMaker** |

---

## ⏱️ Block 4 — Numbers, skim (2 min) — *glance, don't drill*

**720** pass · **256 KB** SQS msg · **30s/12h** visibility default/max · **15 min** Lambda max · **60-120s** RDS failover · **<1s** Aurora Global lag · **5/15** RDS/Aurora read replicas · **20/5** GSI/LSI per table · **72h** Backtrack · **1024-65535** ephemeral ports · **us-east-1** CloudFront ACM · **4 (2×2)** max DX resiliency · **FREE** inbound transfer + Gateway endpoints.

---

**Done. Stop here.** Took the mock. → Afterward, for every wrong answer, open **Recall.md** (the full reference) and re-read that section *the same day*. Log the score + weak list in StudyGuide.md.
