# SAA-C03 Study Guide

**Start Date:** Monday, March 30, 2026 (45-min kickoff session)
**Full Days Start:** Tuesday, March 31, 2026
**Pace:** 3 hours/day (three 50-min blocks with 10-min breaks)
**Rest Days:** 2 (mid-learning + pre-mocks consolidation)
**Exam Date:** Wednesday, April 22, 2026 (early afternoon)
**Total Duration:** 24 calendar days (Day 0 kickoff + 21 study days + 2 rest days)

---

## How This Plan Works

**The science behind this structure:**
- Sustained attention peaks at ~50 min then drops sharply. Three 50-min blocks keeps you in the high-retention zone the entire session.
- The 10-min breaks between blocks are critical — quiet rest after learning boosts memory consolidation by 15-40%.
- 3 hours/day is aggressive but sustainable for a focused sprint. The third block gives you room to go deeper on heavy chapters.

**Learning days (Phase 1):**
```
Block 1 (50 min)  →  Break (10 min)  →  Block 2 (50 min)  →  Break (10 min)  →  Block 3 (50 min)
  [Review old]         [Walk/snack]        [New content]        [Walk/stretch]      [Scenarios + write from memory]
```

**Mock exam days (Phase 2):**
```
Block 1 (65 min)  →  Break (10 min)  →  Block 2 (50 min)  →  Break (10 min)  →  Block 3 (45 min)
  [Take mock exam]     [Walk/stretch]      [Review wrong]       [Walk/stretch]      [Reread weak chapters + drill]
```

**4 weapons against forgetting:**
1. **Spaced review** — Block 1 of every day reviews older material
2. **10-min rest breaks** — brain consolidates what you just reviewed before new input
3. **Scenarios** — each chapter ends with 5 exam-style scenarios that anchor concepts
4. **Mock exams** — 9 mocks force full recall of all 14 chapters (585 practice questions)

---

## Content Map

| Day | Chapter(s) | File(s) | Load |
|-----|-----------|---------|------|
| 0 | Networking (overview) | [Networking](./Networking.md) | **45 min** |
| 1 | Networking (deep) + Storage start | [Networking](./Networking.md) + [Storage](./Storage.md) | Paired |
| 2 | Storage (finish) + Compute | [Storage](./Storage.md) + [Compute](./Compute.md) | Paired |
| 3 | High Availability | [HighAvailability](./HighAvailability.md) | Medium |
| 4 | Databases Relational | [DatabasesRelational](./DatabasesRelational.md) | Heavy |
| 5 | Databases NoSQL | [DatabasesNoSQL](./DatabasesNoSQL.md) | Heavy |
| 6 | Decoupling | [Decoupling](./Decoupling.md) | Heavy |
| — | **REST DAY** | — | — |
| 7 | Serverless | [Serverless](./Serverless.md) | Heavy |
| 8 | Cost/Migration/DR | [CostOptimization](./CostOptimization.md) | Heavy |
| 9 | CDN/DNS | [ContentDeliveryDNS](./ContentDeliveryDNS.md) | Medium |
| 10 | Security & Identity | [SecurityIdentity](./SecurityIdentity.md) | Heavy |
| 11 | Monitoring/Audit + Containers + Additional Services | [MonitoringAudit](./MonitoringAudit.md) + [Containers](./Containers.md) + [AdditionalServices](./AdditionalServices.md) | Triple |
| 12 | Full review + buffer | All cheat sheets | Review |
| — | **REST DAY** | — | — |

---

## Phase 1: Learn (Days 0-12)

---

### Day 0 — Mon Mar 30: Networking Overview (45 min)

**Your kickoff session — ease in, build momentum.**

**Single Block (45 min):** Read Networking.md — focus ONLY on the highest-yield concepts: SG vs NACL table (default NACL allows all, custom NACLs deny all), VPC Endpoints (Gateway = S3/DynamoDB free, Interface = everything else paid), NAT GW in public subnet, Transit Gateway = hub-and-spoke. Read the cheat sheet at the end. Don't try to memorize everything — just get the mental map. **Write 3 things from memory** before you stop: "Block specific IP → ?" "Access S3 privately free → ?" "Connect many VPCs → ?"

---

### Day 1 — Tue Mar 31: Networking Deep + Storage Start

**Block 1 (50 min):** Reread Networking.md from where you left off. Focus on: VPC Peering (no transitive), PrivateLink (NLB + Endpoint Service), Site-to-Site VPN vs Direct Connect table, DX + VPN for encryption, Route 53 Resolver (hybrid DNS), RAM for subnet sharing, Client VPN vs Site-to-Site.
**Break (10 min):** Walk away from screen. Let it settle.
**Block 2 (50 min):** Do all 5 networking scenarios — cover the answer, explain why the other 3 are wrong. **Write from memory:** SG vs NACL table. VPC Endpoint types. Direct Connect vs VPN. Then start reading Storage.md — focus on: EBS type triggers (gp3 default, io1 32K IOPS, io2 64K IOPS, st1 for throughput), Instance Store = ephemeral.
**Break (10 min)**
**Block 3 (50 min):** Continue Storage.md — S3 storage classes waterfall (Standard → IA → One Zone-IA → Glacier IR → Glacier Flexible → Deep Archive), S3 Intelligent-Tiering (6 tiers including archive), S3 Versioning, Lifecycle Policies. **Write from memory:** S3 storage classes in order. EBS type selection triggers.

---

### Day 2 — Wed Apr 1: Storage Finish + Compute

**Block 1 (50 min):** Networking cheat sheet rapid fire — test every trigger. Mark misses. Then finish Storage.md — S3 Replication (CRR/SRR), S3 Object Lock (Governance vs Compliance), S3 Object Lambda, S3 Select, S3 Batch Operations, EFS vs FSx, DataSync vs Storage Gateway, Pre-signed URLs.
**Break (10 min)**
**Block 2 (50 min):** Do all 5 storage scenarios. **Write from memory:** S3 Lifecycle transition rules. Object Lock modes. Then start Compute.md — focus on: purchasing options table (On-Demand/RI/Spot/Dedicated Host vs Instance), Placement Groups (Cluster/Spread/Partition), ENI vs ENA vs EFA.
**Break (10 min)**
**Block 3 (50 min):** Finish Compute.md — Hibernate limits, Capacity Reservations, IMDSv2 (token-based metadata, resists SSRF), EC2 Instance Connect, Launch Templates (not Launch Configs). Do all 5 compute scenarios. **Write from memory:** Purchasing options table. Placement Groups use cases. "Secure metadata → ?" "SSH without keys → ?"

---

### Day 3 — Thu Apr 2: High Availability

**Block 1 (50 min):** Networking + Storage cheat sheets (2-3 days old). Compute triggers: "Per-socket licensing → ?" "90% savings batch job → ?" "Secure instance metadata → ?" Start reading HighAvailability.md — focus on: ALB vs NLB vs GWLB comparison table, cross-zone defaults (ALB always on free, NLB off by default), ALB + OIDC/Cognito authentication.
**Break (10 min)**
**Block 2 (50 min):** Continue HighAvailability.md — all 5 scaling policy types (Target Tracking/Step/Simple/Scheduled/Predictive), SQS-based ASG scaling, Health Check Grace Period vs Cooldown, Connection Draining, Warm Pool, Instance Refresh.
**Break (10 min)**
**Block 3 (50 min):** Finish HighAvailability.md — X-Forwarded-For, GWLB = GENEVE protocol (no SSL termination). Do all 5 HA scenarios. **Write from memory:** ALB vs NLB vs GWLB table. 5 scaling policies with triggers. "Authenticate users at LB → ?" "New instance failing health check → ?"

---

### Day 4 — Fri Apr 3: Databases Relational

**Block 1 (50 min):** Networking + Storage cheat sheets (3+ days old — critical). Compute + HA triggers: "Static IP load balancer → ?" "GENEVE protocol → ?" "Maintain CPU at 60% → ?" "Scale workers based on queue depth → ?" Start reading DatabasesRelational.md.
**Break (10 min)**
**Block 2 (50 min):** Continue DatabasesRelational.md — focus on: Multi-AZ vs Read Replicas table (WILL be on exam — RDS = 5 replicas, Aurora = 15), Aurora Serverless v2 (0.5-128 ACUs), Aurora Global Database (<1 sec replication), RDS Proxy for Lambda, Backtrack (72h max), Aurora Blue/Green Deployments, RDS Performance Insights, RDS Custom.
**Break (10 min)**
**Block 3 (50 min):** Finish DatabasesRelational.md. Do all 5 relational DB scenarios. **Write from memory:** Multi-AZ vs Read Replicas table. Aurora decision tree. "Zero-downtime DB upgrade → ?" "Lambda + RDS → ?" "DB performance bottleneck → ?"

---

### Day 5 — Sat Apr 4: Databases NoSQL

**Block 1 (50 min):** HA + Compute cheat sheets. DB Relational triggers: "Multi-AZ failover → ?" "Read replicas cross-region → ?" "Serverless variable DB → ?" "Lambda connection storm → ?" Start reading DatabasesNoSQL.md.
**Break (10 min)**
**Block 2 (50 min):** Continue DatabasesNoSQL.md — focus on: Provisioned vs On-Demand, GSI vs LSI table (GSI = always eventually consistent), DynamoDB Streams + Lambda, DAX (microseconds) vs ElastiCache, Redis vs Memcached table, Global Tables, DynamoDB TTL (no WCU cost), PartiQL, Transactions (100 items max).
**Break (10 min)**
**Block 3 (50 min):** Finish DatabasesNoSQL.md — Purpose-Built Databases section is critical: DocumentDB (MongoDB), Neptune (graph), Timestream (time-series), QLDB (ledger), Keyspaces (Cassandra), MemoryDB (durable Redis). ElastiCache Global Datastore for cross-region Redis. Do all 5 NoSQL scenarios. **Write from memory:** GSI vs LSI table. Redis vs Memcached. Purpose-built DB table. RCU calc: 10 items/sec, 8KB, strongly consistent = 20 RCU.

---

### Day 6 — Sun Apr 5: Decoupling

**Block 1 (50 min):** DB Relational + DB NoSQL cheat sheets (hardest chapters — quiz every line). Purpose-built DB triggers: "MongoDB → ?" "Graph database → ?" "Time-series → ?" "Immutable ledger → ?" "Cassandra → ?" "Durable Redis → ?" Start reading Decoupling.md.
**Break (10 min)**
**Block 2 (50 min):** Continue Decoupling.md — focus on: SQS Standard vs FIFO table, Long Polling, Visibility Timeout, DLQ, Extended Client Library (messages > 256 KB → S3), SNS + SQS Fan-Out pattern, **EventBridge** (event buses, rules, targets, scheduling, cross-account events — HEAVILY TESTED).
**Break (10 min)**
**Block 3 (50 min):** Finish Decoupling.md — Kinesis Data Streams vs Data Firehose table, KCL (1 worker per shard), enhanced fan-out (dedicated 2 MB/sec per consumer), Amazon MQ = RabbitMQ/ActiveMQ migration only. Do all 5 decoupling scenarios. **Write from memory:** SQS Standard vs FIFO. Kinesis Streams vs Firehose. EventBridge triggers. "Event-driven architecture → ?" "Schedule tasks → ?" "Fan-out → ?"

---

### ☁ Mon Apr 6: REST DAY

No studying. Walk, exercise, do anything else. Your brain is consolidating 6 days of heavy material right now. This is not wasted time — it's when long-term memories form.

---

### Day 7 — Tue Apr 7: Serverless

**Block 1 (50 min):** DB Relational + DB NoSQL + Decoupling cheat sheets (quiz every line — these are now 3-6 days old). Redo "GSI Throttling" and "Fan-Out" scenarios from memory.
**Break (10 min)**
**Block 2 (50 min):** Read Serverless.md — focus on: Lambda limits table (MEMORIZE: 15 min timeout, 10GB mem, 1MB async payload, 10GB /tmp, 1000 concurrent), Lambda Execution Role (IAM role Lambda assumes), invocation types, Provisioned Concurrency = kill cold starts, Lambda Destinations (success/failure routing), REST API vs HTTP API comparison (REST = full features, HTTP = cheaper/simpler).
**Break (10 min)**
**Block 3 (50 min):** Finish Serverless.md — Step Functions Standard (1 year) vs Express (5 min), InputPath/OutputPath/ResultPath for data flow, SNS + Lambda fan-out, SQS + Lambda. Do all 5 serverless scenarios. **Write from memory:** Lambda limits table. Step Functions Standard vs Express. "Lambda IAM → ?" "Simple cheap API → ?" "Async error routing → ?"

---

### Day 8 — Wed Apr 8: Cost/Migration/DR

**Block 1 (50 min):** Networking + Storage cheat sheets (Day 0-2, now 7+ days old — critical reinforcement). If you can't recall 80%+ instantly, flag for extra review on Day 12. Start reading CostOptimization.md.
**Break (10 min)**
**Block 2 (50 min):** Continue CostOptimization.md — focus on: Cost Explorer vs Budgets vs Compute Optimizer vs Trusted Advisor, Savings Plans (Compute covers EC2+Fargate+Lambda vs EC2 Instance), RI types (Standard vs Convertible), Data Transfer cost rules (inbound free, cross-AZ costs, Gateway endpoints free), 6 R's of Migration.
**Break (10 min)**
**Block 3 (50 min):** Finish CostOptimization.md — DMS (+SCT for heterogeneous), Snow Family, DataSync, Migration Hub, Application Discovery Service, DR patterns table (RPO/RTO — WILL be on exam), Organizations SCPs, Control Tower guardrails (Preventive = SCPs, Detective = Config Rules), Fault Injection Simulator, AWS Backup. Do all 5 cost scenarios. **Write from memory:** DR patterns table. 6 R's table. Data transfer rules. "Chaos engineering → ?" "Discover on-prem servers → ?" "Cheapest DR → ?"

---

### Day 9 — Thu Apr 9: CDN/DNS

**Block 1 (50 min):** Serverless + Decoupling cheat sheets. Cost triggers: "Cheapest DR → ?" "Move to AWS minimal changes → ?" "Inbound data cost → ?" "Cross-AZ data cost → ?" Start reading ContentDeliveryDNS.md.
**Break (10 min)**
**Block 2 (50 min):** Continue ContentDeliveryDNS.md — focus on: All 7 Route 53 routing policies table (WILL be on exam), Route 53 Resolver (hybrid DNS), Alias vs CNAME (root domain → Alias only), public vs private hosted zones, CloudFront OAC (not OAI), Signed URLs vs Signed Cookies, cache behaviors (path patterns → different origins).
**Break (10 min)**
**Block 3 (50 min):** Finish ContentDeliveryDNS.md — CloudFront Functions vs Lambda@Edge table (heavily tested), Origin Groups (failover on specific HTTP codes), CloudFront real-time logs (→ Kinesis Firehose), CloudFront vs Global Accelerator table. Do all 5 CDN/DNS scenarios. **Write from memory:** All 7 routing policies. CloudFront Functions vs Lambda@Edge. "Root domain → ?" "Hybrid DNS → ?" "Static IP + global performance → ?"

---

### Day 10 — Fri Apr 10: Security & Identity

**Block 1 (50 min):** Networking + Storage + HA cheat sheets (10+ days old — critical). Cost/Migration + CDN/DNS triggers from yesterday. Start reading SecurityIdentity.md.
**Break (10 min)**
**Block 2 (50 min):** Continue SecurityIdentity.md — focus on: IAM evaluation logic (Deny > Allow > Default Deny), Permission Boundaries (intersection with identity policy), STS, IAM Identity Center (SSO), RAM (share resources across accounts), IAM Access Analyzer (find unintended public access), KMS key types + Envelope Encryption + key policy evaluation (need BOTH key policy + IAM policy), KMS vs CloudHSM table, ACM (free public SSL, paid private CA, us-east-1 for CloudFront).
**Break (10 min)**
**Block 3 (50 min):** Finish SecurityIdentity.md — AWS Directory Service (Managed AD vs AD Connector vs Simple AD), VPC Endpoint Policies, Secrets Manager vs Parameter Store, Cognito User Pools vs Identity Pools, WAF, GuardDuty vs Inspector vs Macie, Security Hub. Do all 5 security scenarios. **Write from memory:** IAM eval logic. KMS vs CloudHSM. "Share resources cross-account → ?" "Find public access → ?" "On-prem AD integration → ?" "Internal HTTPS certs → ?" "Control endpoint access → ?"

---

### Day 11 — Sat Apr 11: Monitoring/Audit + Containers + Additional Services

**Triple day — all three are lighter. Monitoring is medium, Containers is medium, Additional is a skim.**

**Block 1 (50 min):** DB Relational + DB NoSQL cheat sheets (now 7+ days old — reinforce). Purpose-built DB triggers rapid fire. Then read MonitoringAudit.md — focus on: CloudWatch vs CloudTrail vs Config table (WILL be on exam), CloudWatch Agent for memory/disk, CloudWatch Contributor Insights (top-N contributors), CloudWatch Synthetics (canary scripts), X-Ray for distributed tracing.
**Break (10 min)**
**Block 2 (50 min):** Finish MonitoringAudit.md — SSM Session Manager, Patch Manager, Trusted Advisor 5 pillars. Do top 3 monitoring scenarios. Then read Containers.md — focus on: ECS EC2 vs Fargate table, Task Role vs Execution Role, awsvpc mode, ECS Capacity Providers (Fargate Spot = ~70% savings), Cloud Map for service discovery, EKS = Kubernetes migration, App Runner = simplest container deployment.
**Break (10 min)**
**Block 3 (50 min):** Finish Containers scenarios. Then read AdditionalServices.md — Athena (SQL on S3), Redshift (data warehouse), Glue (ETL), Batch (long-running jobs), EMR (Hadoop/Spark), QuickSight (BI), Lake Formation, ML/AI services (Kendra = search, Personalize = recommendations), Amplify (web/mobile apps), Managed Grafana/Prometheus. Do all 5 additional scenarios. **Write from memory:** CloudWatch vs CloudTrail vs Config. ECS EC2 vs Fargate. "Query S3 with SQL → ?" "Intelligent search → ?" "Product recommendations → ?" "Cheap Fargate → ?"

---

### Day 12 — Sun Apr 12: Full Review Day + Buffer

**Block 1 (50 min):** ALL 14 chapter cheat sheets — rapid fire. Cover the answer, test yourself. Mark anything you can't recall instantly.
**Break (10 min)**
**Block 2 (50 min):** Continue cheat sheets. Write your **weak list** on paper — every trigger/answer you missed.
**Break (10 min)**
**Block 3 (50 min):** Redo the 5 hardest scenarios across all chapters (whichever ones tripped you up). Reread weak chapter sections.

This day also serves as a **buffer** — if you fell behind by 1 day during Phase 1, use today to catch up on the last chapter instead, and do the full review as Block 1 on Day 13.

**Mock exams ready:**
- Stephane Maarek SAA-C03 (6 exams — using 3, 3 backup)
- Neal Davis SAA-C03 (6 exams — using 3, 3 backup)
- Jon Bonso / Tutorials Dojo SAA-C03 (6 exams — using 3, 3 backup)

---

### ☁ Mon Apr 13: REST DAY

No studying. You just finished all 14 chapters. Let your brain consolidate everything before the mock exam gauntlet begins. Walk, exercise, sleep well.

---

## Phase 2: Mock Exams + Reinforcement (Days 13-21)

**Every day is a mock exam day = 3 hours:**
```
Block 1 (65 min): Take mock exam under real conditions
Break (10 min): Walk, stretch, decompress
Block 2 (50 min): Review EVERY wrong answer — go back to chapter for each topic
Break (10 min)
Block 3 (45 min): Reread weak chapter sections + drill weak list
```

**9 mocks in 9 days. 585 practice questions.** Block 2+3 of each mock day gives you 95 minutes of targeted review focused on exactly what you got wrong — more effective than generic review.

**Pattern: Rotating Maarek (straightforward), Davis (hardest), and Bonso (closest to real exam) for maximum variety.**

---

### Day 13 — Tue Apr 14: Mock #1 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #1 under real conditions.
**Break (10 min)**
**Block 2 (50 min):** Review every wrong answer + every flagged answer. Go back to the chapter for each wrong topic.
**Break (10 min)**
**Block 3 (45 min):** Reread the weakest 2-3 chapter sections. Start your weak list.
**Expected score:** 55-70%. This is your baseline. Don't panic.

---

### Day 14 — Wed Apr 15: Mock #2 (Davis) [3 hours]
**Block 1 (65 min):** Neal Davis Exam #1.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers. Notice patterns — same topics wrong in Mock #1 AND #2 = blind spots.
**Break (10 min)**
**Block 3 (45 min):** Highlight blind spots on weak list. Reread those chapter sections. Redo the scenarios for blind spot chapters.
**Expected score:** 50-65% (Davis is intentionally harder — this is normal).

---

### Day 15 — Thu Apr 16: Mock #3 (Bonso) [3 hours]
**Block 1 (65 min):** Jon Bonso / Tutorials Dojo Exam #1.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Speed-run all cheat sheets. Focus extra time on blind spot chapters from Mock #1+2.
**Target score:** 60-72%. Bonso is closest to the real exam — treat this score as your most reliable indicator.

---

### Day 16 — Fri Apr 17: Mock #4 (Davis) — CHECKPOINT [3 hours]
**Block 1 (65 min):** Neal Davis Exam #2.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Deep-dive into persistent weak topics. If DR patterns, cost tools, or migration questions keep tripping you, reread CostOptimization.md fully.
**Target score:** 60-72% (Davis scale — equivalent to 70-78% on real exam).

**Decision point:**

| Davis Score | Real Exam Equivalent | Action |
|-------------|---------------------|--------|
| 68%+ | ~75%+ | Keep April 22. You're on track. |
| 60-67% | ~68-74% | Keep April 22. Remaining days focus exclusively on weak list. |
| 52-59% | ~60-67% | Consider rescheduling to April 29. |
| <52% | ~<60% | Reschedule to April 29. Reread your 3 weakest chapters fully. |

---

### Day 17 — Sat Apr 18: Mock #5 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #2.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Drill weak list. Redo your 5 weakest scenarios from any chapter.
**Target score:** 75-82%.

---

### Day 18 — Sun Apr 19: Mock #6 (Bonso) [3 hours]
**Block 1 (65 min):** Jon Bonso Exam #2.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Reread any chapter section that still trips you. Update weak list.
**Target score:** 70-78%. Compare with Bonso #1 — you should see clear improvement.

---

### Day 19 — Mon Apr 20: Mock #7 (Davis) [3 hours]
**Block 1 (65 min):** Neal Davis Exam #3.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Speed-run all cheat sheets. Write from memory your top 10 weakest triggers.
**Target score:** 68-75% (Davis scale).

---

### Day 20 — Tue Apr 21: Mock #8 (Bonso) — GO/NO-GO [3 hours]
**Block 1 (65 min):** Jon Bonso Exam #3.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Final weak list drill. Skim the 3 chapters you've scored lowest on across all mocks.
**Target score:** 75-85% (Bonso is closest to real exam — this is your best predictor).

If 75%+ on Bonso: You're passing. Cruise to exam day.
If 68-74% on Bonso: You'll likely pass. Light review only tomorrow.
If <68% on Bonso: Push exam back 1 week.

---

### Day 21 — Wed Apr 22 morning: Mock #9 (Maarek) — FINAL [2 hours]
**Block 1 (65 min):** Stephane Maarek Exam #3. Real conditions. Time it.
**Break (10 min)**
**Block 2 (45 min):** Quick review of wrong answers. Skim weak list one final time. Done.

**This is your morning session — exam is early afternoon.**

**Target score:** 82-90%. If you're here, you're ready.

---

### Wed Apr 22 early afternoon: EXAM DAY

**After Mock #9 review — rest protocol:**
- Lunch. Walk. Decompress.
- Do NOT re-study. You just did 9 mocks. You're ready.

**Exam strategy (65 questions, 130 min):**
- **Pass 1 (45 min):** Answer confident questions, flag everything else
- **Pass 2 (60 min):** Work flagged questions — eliminate 2 wrong answers first
- **Pass 3 (15 min):** Review flagged, trust first instinct
- Watch for: "NOT", "EXCEPT", "MOST cost-effective", "LEAST operational overhead"
- When stuck: managed AWS service > custom code. Simpler > complex.
- Answer everything — no penalty for guessing

**Passing score:** 720/1000 (~72%)

---

## The Review Schedule (How You Don't Forget)

Every learning day, Block 1 reviews older material before Blocks 2-3 introduce new content. The 10-min breaks let your brain consolidate before new input.

```
Day  0: Networking overview (45 min)                (kickoff — get the mental map)
Day  1: Networking deep + Storage start             Review → Networking Day 0 (+1 day)
Day  2: Storage finish + Compute                    Review → Networking (+2 days)
Day  3: High Availability                           Review → Networking, Storage (+2/+3 days)
Day  4: DB Relational                               Review → Networking, Storage, Compute, HA (+1-4 days)
Day  5: DB NoSQL                                    Review → HA, Compute (+2/+3 days)
Day  6: Decoupling                                  Review → DB Relational, DB NoSQL (+1/+2 days)
       ☁ REST DAY                                   Brain consolidates Days 0-6
Day  7: Serverless                                  Review → DB NoSQL, Decoupling (+1/+2 days)
Day  8: Cost/Migration/DR                           Review → Networking, Storage (+7 days — critical)
Day  9: CDN/DNS                                     Review → Serverless, Decoupling (+2/+3 days)
Day 10: Security                                    Review → Networking, Storage, HA (+10 days — critical)
Day 11: Monitoring + Containers + Additional        Review → Cost, CDN/DNS (+2/+3 days)
Day 12: Full review                                 Review → ALL 14 cheat sheets
       ☁ REST DAY                                   Brain consolidates Days 0-12
Day 13+: Mock exams                                 Every mock reviews EVERYTHING at once
```

**Result:** By exam day, Day 0-1 content has been reviewed 12+ times across:
- Day 0 overview + Day 1 deep dive
- 5 dedicated Block 1 reviews
- 1 full review day
- 9 mock exams that test all topics (585 practice questions)
- 2 rest days for memory consolidation

---

## When to Book the Exam

**Book it NOW for Wednesday April 22, early afternoon slot.**

- Exam is booked — no backing out, only forward
- You can reschedule up to 24 hours before (AWS policy)
- Day 16 checkpoint tells you if you need to push back

---

## Mock Exam Score Tracker

```
Mock #1 (Day 13) Maarek #1:  ___% | Weak: _______________
Mock #2 (Day 14) Davis  #1:  ___% | Weak: _______________
Mock #3 (Day 15) Bonso  #1:  ___% | Weak: _______________
Mock #4 (Day 16) Davis  #2:  ___% | Weak: _______________ ← CHECKPOINT
Mock #5 (Day 17) Maarek #2:  ___% | Weak: _______________
Mock #6 (Day 18) Bonso  #2:  ___% | Weak: _______________
Mock #7 (Day 19) Davis  #3:  ___% | Weak: _______________
Mock #8 (Day 20) Bonso  #3:  ___% | Weak: _______________ ← GO/NO-GO
Mock #9 (Day 21) Maarek #3:  ___% | Weak: _______________ ← FINAL (exam day morning)
```

**Healthy:** Scores trend upward across all authors. Davis ~8% lower than Maarek is normal. Bonso is closest to the real exam — use those scores as your best predictor.
**You have 9 backup exams** (3 Maarek + 2 Davis + 4 Bonso) if you need to reschedule or want extra practice.
**Warning:** Plateau or drop for 2 consecutive → reread your 2 weakest chapters entirely.

---

## Calendar View

```
WEEK 1 — KICKOFF + LEARN (Mar 30 - Apr 5)
  Mon 30: Networking overview (45 min — kickoff)
  Tue 31: Networking deep + Storage start            [3h, 50+10+50+10+50]
  Wed  1: Storage finish + Compute                   [3h]
  Thu  2: High Availability                          [3h]
  Fri  3: Databases Relational                       [3h]
  Sat  4: Databases NoSQL                            [3h]
  Sun  5: Decoupling                                 [3h]

WEEK 2 — LEARN + REVIEW (Apr 6 - Apr 12)
  Mon  6: ☁ REST DAY
  Tue  7: Serverless                                 [3h]
  Wed  8: Cost/Migration/DR                          [3h]
  Thu  9: CDN/DNS                                    [3h]
  Fri 10: Security & Identity                        [3h]
  Sat 11: Monitoring + Containers + Additional       [3h]
  Sun 12: Full review + buffer                       [3h]

WEEK 3 — MOCKS (Apr 13 - Apr 21)
  Mon 13: ☁ REST DAY
  Tue 14: Mock #1 (Maarek)                           [3h, 65+10+50+10+45]
  Wed 15: Mock #2 (Davis)
  Thu 16: Mock #3 (Bonso)
  Fri 17: Mock #4 (Davis) ← CHECKPOINT
  Sat 18: Mock #5 (Maarek)
  Sun 19: Mock #6 (Bonso)
  Mon 20: Mock #7 (Davis)
  Tue 21: Mock #8 (Bonso) ← GO/NO-GO

WEEK 4 — FINAL (Apr 22)
  Wed 22: Mock #9 (Maarek, morning) → ☁ REST → EXAM (early afternoon)
```

---

## If You Miss a Day

**Missed 1 day during Phase 1:** Use the first rest day (Apr 6) to catch up. If past the first rest day, use the second rest day (Apr 13). If both are used, extend to a 4th block (3.5h) the next day.

**Missed 1 day during Phase 2:** You have 9 mocks scheduled. Dropping to 8 is fine — skip the last Maarek before the go/no-go (Day 19), keep the Davis go/no-go and exam-morning mock.

**Missed 2+ days:** Shift everything forward. Reschedule exam if needed.

**Golden rule:** Never skip Block 1 review to cram new content. Old material fading is worse than being 1 day behind. And never skip rest days — they're when your brain builds long-term memory.

---

## What You Need Before Day 1

- [ ] All 14 chapter files (already in this folder)
- [ ] Stephane Maarek SAA-C03 practice exams (Udemy) — buy by Day 11
- [ ] Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy) — buy by Day 11
- [ ] A notebook for writing from memory (paper > app for retention)
- [ ] A timer (set 50-min blocks — phone timer works)
- [ ] AWS Certification account (aws.amazon.com/certification)
