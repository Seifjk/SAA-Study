# SAA-C03 Study Guide

**Start Date:** Wednesday, March 26, 2026
**Pace:** 3 hours/day (three 50-min blocks with 10-min breaks)
**Rest Days:** 2 (mid-learning + pre-mocks consolidation)
**Exam Date:** Thursday, April 17, 2026 (early afternoon)
**Total Duration:** 23 calendar days (21 study days + 2 rest days)

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
| 1 | Networking | [Networking](./Networking.md) | Medium |
| 2 | Storage | [Storage](./Storage.md) | Medium |
| 3 | Compute + High Availability | [Compute](./Compute.md) + [HighAvailability](./HighAvailability.md) | Paired |
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

## Phase 1: Learn (Days 1-12)

---

### Day 1 — Thu Mar 26: Networking

**Block 1 (50 min):** First day — no review. Start reading Networking.md. Focus on: SG vs NACL table, VPC Endpoints (Gateway = S3/DynamoDB free, Interface = everything else paid), NAT GW must be in public subnet, Transit Gateway = hub-and-spoke.
**Break (10 min):** Walk away from screen. Let it settle.
**Block 2 (50 min):** Continue Networking.md. Focus on: VPC Peering (no transitive), PrivateLink (NLB + Endpoint Service), Site-to-Site VPN vs Direct Connect table, DX + VPN for encryption, Bastion Host vs Session Manager.
**Break (10 min)**
**Block 3 (50 min):** Do all 5 networking scenarios — cover the answer, explain why the other 3 are wrong. **Write from memory:** SG vs NACL table. VPC Endpoint types. Direct Connect vs VPN. Cheat sheet triggers.

---

### Day 2 — Fri Mar 27: Storage

**Block 1 (50 min):** Close Networking.md. From memory write: "Block specific IP → ?" "Access S3 privately free → ?" "Connect 50 VPCs → ?" "Instance has public IP but no internet → ?" "DX but need encryption → ?" Check against cheat sheet. Redo any you missed. Then start reading Storage.md.
**Break (10 min)**
**Block 2 (50 min):** Continue Storage.md — focus on: EBS type triggers (gp3 default, io1/io2 for IOPS, st1 for throughput), S3 storage classes waterfall, S3 Versioning, S3 Lifecycle Policies, S3 Object Lock (Governance vs Compliance mode), Multi-Attach = io1/io2 only.
**Break (10 min)**
**Block 3 (50 min):** Finish Storage.md — S3 Replication (CRR/SRR), EFS vs FSx decision, DataSync vs Storage Gateway, Pre-signed URLs, EBS Snapshots + DLM. Do all 5 storage scenarios. **Write from memory:** S3 storage classes in order. EBS type selection triggers. S3 Lifecycle transition rules. Object Lock modes.

---

### Day 3 — Sat Mar 28: Compute + High Availability

**Paired day — both are medium-weight and deeply connected (compute scales via HA).**

**Block 1 (50 min):** Networking + Storage cheat sheets. Cover answers, test every trigger line. Mark any you miss — these go on your weak list. Start reading Compute.md.
**Break (10 min)**
**Block 2 (50 min):** Finish Compute.md — focus on: purchasing options table (On-Demand/RI/Spot/Dedicated Host vs Instance), Placement Groups (Cluster/Spread/Partition), ENI vs ENA vs EFA, Hibernate limits, Capacity Reservations. Do all 5 compute scenarios. Then start HighAvailability.md — focus on: ALB vs NLB vs GWLB comparison table, NLB supports Security Groups but cross-zone costs extra.
**Break (10 min)**
**Block 3 (50 min):** Finish HighAvailability.md — all 5 scaling policy types (Target Tracking/Step/Simple/Scheduled/Predictive), SQS-based ASG scaling, Connection Draining, Warm Pool, Instance Refresh, X-Forwarded-For. Do all 5 HA scenarios. **Write from memory:** Purchasing options table. ALB vs NLB vs GWLB table. 5 scaling policies with triggers.

---

### Day 4 — Sun Mar 29: Databases Relational

**Block 1 (50 min):** Networking + Storage cheat sheets (rapid fire triggers — now 3+ days old). Compute + HA triggers: "Per-socket licensing → ?" "Static IP load balancer → ?" "GENEVE protocol → ?" "Maintain CPU at 60% → ?" "Scale workers based on queue depth → ?" Start reading DatabasesRelational.md.
**Break (10 min)**
**Block 2 (50 min):** Continue DatabasesRelational.md — focus on: Multi-AZ vs Read Replicas table (WILL be on exam — RDS = 5 replicas, Aurora = 15), Aurora Serverless v2, Aurora Global Database (<1 sec replication, promote <1 min), RDS Proxy for Lambda, Backtrack (in-place rewind, seconds, 72h max), Aurora Cloning, RDS Custom.
**Break (10 min)**
**Block 3 (50 min):** Finish DatabasesRelational.md. Do all 5 relational DB scenarios. **Write from memory:** Multi-AZ vs Read Replicas table. Aurora decision tree. "Read-heavy workload → ?" "Global DB <1 sec → ?" "Lambda + RDS → ?" "Fast test copy of prod DB → ?" "Need OS access on managed DB → ?"

---

### Day 5 — Mon Mar 30: Databases NoSQL

**Block 1 (50 min):** HA + Compute cheat sheets. DB Relational triggers: "Multi-AZ failover → ?" "Read replicas cross-region → ?" "Serverless variable DB → ?" "Lambda connection storm → ?" Start reading DatabasesNoSQL.md.
**Break (10 min)**
**Block 2 (50 min):** Continue DatabasesNoSQL.md — focus on: Provisioned vs On-Demand, GSI vs LSI table, DynamoDB Streams + Lambda, DAX (microseconds) vs ElastiCache, Redis vs Memcached table, Global Tables, DynamoDB Export to S3.
**Break (10 min)**
**Block 3 (50 min):** Finish DatabasesNoSQL.md — Purpose-Built Databases section is critical: DocumentDB (MongoDB), Neptune (graph), Timestream (time-series), QLDB (ledger), Keyspaces (Cassandra), MemoryDB (durable Redis). Do all 5 NoSQL scenarios. **Write from memory:** GSI vs LSI table. Redis vs Memcached. Purpose-built DB table (service → use case → trigger). RCU calc: 10 items/sec, 8KB, strongly consistent = 20 RCU.

---

### Day 6 — Tue Mar 31: Decoupling

**Block 1 (50 min):** DB Relational + DB NoSQL cheat sheets (hardest chapters — quiz every line). Purpose-built DB triggers: "MongoDB → ?" "Graph database → ?" "Time-series → ?" "Immutable ledger → ?" "Cassandra → ?" "Durable Redis → ?" Start reading Decoupling.md.
**Break (10 min)**
**Block 2 (50 min):** Continue Decoupling.md — focus on: SQS Standard vs FIFO table, Long Polling, Visibility Timeout, DLQ, Extended Client Library (messages > 256 KB → S3), SNS + SQS Fan-Out pattern.
**Break (10 min)**
**Block 3 (50 min):** Finish Decoupling.md — Kinesis Data Streams vs Data Firehose table, KCL (1 worker per shard), Amazon MQ = RabbitMQ/ActiveMQ migration only. Do all 5 decoupling scenarios. **Write from memory:** SQS Standard vs FIFO. Kinesis Streams vs Firehose. "Strict ordering → ?" "Fan-out → ?" "Message > 256 KB → ?" "Migrate RabbitMQ → ?"

---

### ☁ Wed Apr 1: REST DAY

No studying. Walk, exercise, do anything else. Your brain is consolidating 6 days of heavy material right now. This is not wasted time — it's when long-term memories form.

---

### Day 7 — Thu Apr 2: Serverless

**Block 1 (50 min):** DB Relational + DB NoSQL + Decoupling cheat sheets (quiz every line — these are now 3-6 days old). Redo "GSI Throttling" and "Fan-Out" scenarios from memory.
**Break (10 min)**
**Block 2 (50 min):** Read Serverless.md — focus on: Lambda limits table (MEMORIZE: 15 min timeout, 10GB mem, 1MB async payload, 10GB /tmp, 1000 concurrent), invocation types, Provisioned Concurrency = kill cold starts, Lambda + EFS for persistent storage, Lambda container images from ECR, API Gateway throttling/caching.
**Break (10 min)**
**Block 3 (50 min):** Finish Serverless.md — Step Functions Standard (1 year) vs Express (5 min), SNS + Lambda fan-out pattern, SQS + Lambda, S3 + Lambda. Do all 5 serverless scenarios. **Write from memory:** Lambda limits table. Step Functions Standard vs Express. API Gateway cache/throttle.

---

### Day 8 — Fri Apr 3: Cost/Migration/DR

**Block 1 (50 min):** Networking + Storage cheat sheets (Day 1-2, now 7+ days old — critical reinforcement window). If you can't recall 80%+ instantly, flag for extra review on Day 12. Start reading CostOptimization.md.
**Break (10 min)**
**Block 2 (50 min):** Continue CostOptimization.md — focus on: Cost Explorer vs Budgets vs Compute Optimizer vs Trusted Advisor, Savings Plans (Compute vs EC2 Instance), RI types (Standard vs Convertible), Data Transfer cost rules (inbound free, cross-AZ costs, Gateway endpoints free), 6 R's of Migration.
**Break (10 min)**
**Block 3 (50 min):** Finish CostOptimization.md — DMS (+SCT for heterogeneous), Snow Family (Snowcone/Snowball/Snowmobile), DataSync, Migration Hub, DR patterns table (RPO/RTO — WILL be on exam), AWS Organizations SCPs, AWS Backup. Do all 5 cost scenarios. **Write from memory:** DR patterns table (4 strategies, RPO/RTO/Cost). 6 R's table. Data transfer rules. "Lift-and-shift servers → ?" "Cheapest DR → ?" ">10 TB slow internet → ?"

---

### Day 9 — Sat Apr 4: CDN/DNS

**Block 1 (50 min):** Serverless + Decoupling cheat sheets. Cost triggers: "Cheapest DR → ?" "Move to AWS minimal changes → ?" "Inbound data cost → ?" "Cross-AZ data cost → ?" Start reading ContentDeliveryDNS.md.
**Break (10 min)**
**Block 2 (50 min):** Continue ContentDeliveryDNS.md — focus on: All 7 Route 53 routing policies table (WILL be on exam), Alias vs CNAME (root domain → Alias only), public vs private hosted zones, CloudFront OAC (not OAI), Signed URLs vs Signed Cookies.
**Break (10 min)**
**Block 3 (50 min):** Finish ContentDeliveryDNS.md — CloudFront Functions vs Lambda@Edge table (heavily tested), Origin Groups (failover), CloudFront vs Global Accelerator table. Do all 5 CDN/DNS scenarios. **Write from memory:** All 7 routing policies. CloudFront Functions vs Lambda@Edge. "Root domain → ?" "Simple edge header manipulation → ?" "Static IP + global performance → ?"

---

### Day 10 — Sun Apr 5: Security & Identity

**Block 1 (50 min):** Networking + Storage + HA cheat sheets (10+ days old — critical). Cost/Migration + CDN/DNS triggers from yesterday. Start reading SecurityIdentity.md.
**Break (10 min)**
**Block 2 (50 min):** Continue SecurityIdentity.md — focus on: IAM evaluation logic (Deny > Allow > Default Deny), Permission Boundaries, STS (AssumeRole, AssumeRoleWithWebIdentity, AssumeRoleWithSAML), IAM Identity Center (SSO), KMS key types + Envelope Encryption, KMS vs CloudHSM table, ACM (free SSL, us-east-1 for CloudFront).
**Break (10 min)**
**Block 3 (50 min):** Finish SecurityIdentity.md — Secrets Manager vs Parameter Store, Cognito User Pools vs Identity Pools, WAF, GuardDuty vs Inspector vs Macie, Security Hub, Firewall Manager, Network Firewall. Do all 5 security scenarios. **Write from memory:** IAM eval logic. KMS vs CloudHSM. Cognito pools. GuardDuty vs Inspector vs Macie. "Free SSL for ALB → ?" "FIPS 140-2 Level 3 → ?" "SSO across accounts → ?" "Centralized security findings → ?"

---

### Day 11 — Mon Apr 6: Monitoring/Audit + Containers + Additional Services

**Triple day — all three are lighter. Monitoring is medium, Containers is medium, Additional is a skim.**

**Block 1 (50 min):** DB Relational + DB NoSQL cheat sheets (now 7+ days old — reinforce). Purpose-built DB triggers rapid fire. Then read MonitoringAudit.md — focus on: CloudWatch vs CloudTrail vs Config table (WILL be on exam), CloudWatch Agent for memory/disk, X-Ray for distributed tracing, Metric Filters + Subscription Filters.
**Break (10 min)**
**Block 2 (50 min):** Finish MonitoringAudit.md — SSM Session Manager, Patch Manager, Trusted Advisor 5 pillars. Do top 3 monitoring scenarios. Then read Containers.md — focus on: ECS EC2 vs Fargate table, Task Role vs Execution Role, awsvpc mode, EFS for persistent storage, EKS = Kubernetes migration, App Runner = simplest container deployment.
**Break (10 min)**
**Block 3 (50 min):** Finish Containers scenarios. Then read AdditionalServices.md — Athena (SQL on S3), Redshift (data warehouse), Glue (ETL), Batch (long-running jobs), EMR (Hadoop/Spark), QuickSight (BI), Lake Formation (data lake governance), ML/AI services table, Outposts/Local Zones/Wavelength. Do all 5 additional scenarios. **Write from memory:** CloudWatch vs CloudTrail vs Config. ECS EC2 vs Fargate. "Query S3 with SQL → ?" "Data warehouse → ?" "ETL → ?" "Long batch jobs → ?" "Graph database → ?" "Deploy container zero config → ?"

---

### Day 12 — Tue Apr 7: Full Review Day + Buffer

**Block 1 (50 min):** ALL 14 chapter cheat sheets — rapid fire. Cover the answer, test yourself. Mark anything you can't recall instantly.
**Break (10 min)**
**Block 2 (50 min):** Continue cheat sheets. Write your **weak list** on paper — every trigger/answer you missed.
**Break (10 min)**
**Block 3 (50 min):** Redo the 5 hardest scenarios across all chapters (whichever ones tripped you up). Reread weak chapter sections.

This day also serves as a **buffer** — if you fell behind by 1 day during Phase 1, use today to catch up on the last chapter instead, and do the full review as Block 1 on Day 13.

**Also today:** Buy mock exams if you haven't:
- Stephane Maarek SAA-C03 practice exams (Udemy) — you need 5 exams from this set
- Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy) — you need 4 exams from this set

---

### ☁ Wed Apr 8: REST DAY

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

**Pattern: Alternating Maarek (closer to real exam) and Davis (harder, builds resilience).**

---

### Day 13 — Thu Apr 9: Mock #1 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #1 under real conditions.
**Break (10 min)**
**Block 2 (50 min):** Review every wrong answer + every flagged answer. Go back to the chapter for each wrong topic.
**Break (10 min)**
**Block 3 (45 min):** Reread the weakest 2-3 chapter sections. Start your weak list.
**Expected score:** 55-70%. This is your baseline. Don't panic.

---

### Day 14 — Fri Apr 10: Mock #2 (Davis) [3 hours]
**Block 1 (65 min):** Neal Davis / Tutorials Dojo Exam #1.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers. Notice patterns — same topics wrong in Mock #1 AND #2 = blind spots.
**Break (10 min)**
**Block 3 (45 min):** Highlight blind spots on weak list. Reread those chapter sections. Redo the scenarios for blind spot chapters.
**Expected score:** 50-65% (Davis is intentionally harder — this is normal).

---

### Day 15 — Sat Apr 11: Mock #3 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #2.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Speed-run all cheat sheets. Focus extra time on blind spot chapters from Mock #1+2.
**Target score:** 65-75%. You should see improvement from Mock #1.

---

### Day 16 — Sun Apr 12: Mock #4 (Davis) — CHECKPOINT [3 hours]
**Block 1 (65 min):** Neal Davis / Tutorials Dojo Exam #2.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Deep-dive into persistent weak topics. If DR patterns, cost tools, or migration questions keep tripping you, reread CostOptimization.md fully.
**Target score:** 60-72% (Davis scale — equivalent to 70-78% on real exam).

**Decision point:**

| Davis Score | Real Exam Equivalent | Action |
|-------------|---------------------|--------|
| 68%+ | ~75%+ | Keep April 17. You're on track. |
| 60-67% | ~68-74% | Keep April 17. Remaining days focus exclusively on weak list. |
| 52-59% | ~60-67% | Consider rescheduling to April 24. |
| <52% | ~<60% | Reschedule to April 24. Reread your 3 weakest chapters fully. |

---

### Day 17 — Mon Apr 13: Mock #5 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #3.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Drill weak list. Redo your 5 weakest scenarios from any chapter.
**Target score:** 75-82%.

---

### Day 18 — Tue Apr 14: Mock #6 (Davis) [3 hours]
**Block 1 (65 min):** Neal Davis / Tutorials Dojo Exam #3.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Reread any chapter section that still trips you. Update weak list.
**Target score:** 65-75% (Davis scale).

---

### Day 19 — Wed Apr 15: Mock #7 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #4.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Speed-run all cheat sheets. Write from memory your top 10 weakest triggers.
**Target score:** 78-88%.

---

### Day 20 — Thu Apr 16: Mock #8 (Davis) — GO/NO-GO [3 hours]
**Block 1 (65 min):** Neal Davis / Tutorials Dojo Exam #4.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Final weak list drill. Skim the 3 chapters you've scored lowest on across all mocks.
**Target score:** 68-78% (Davis scale — equivalent to 78-85% real exam).

If 68%+ on Davis: You're passing. Cruise to exam day.
If 60-67% on Davis: You'll likely pass. Light review only tomorrow.
If <60% on Davis: Push exam back 1 week.

---

### Day 21 — Fri Apr 17 morning: Mock #9 (Maarek) — FINAL [2 hours]
**Block 1 (65 min):** Stephane Maarek Exam #5. Real conditions. Time it.
**Break (10 min)**
**Block 2 (45 min):** Quick review of wrong answers. Skim weak list one final time. Done.

**This is your morning session — exam is early afternoon.**

**Target score:** 82-90%. If you're here, you're ready.

---

### Thu Apr 17 early afternoon: EXAM DAY

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
Day  1: Learn Networking                           (no review — first day)
Day  2: Learn Storage                              Review → Networking (+1 day)
Day  3: Learn Compute + HA                         Review → Networking, Storage (+1/+2 days)
Day  4: Learn DB Relational                        Review → Networking, Storage, Compute, HA (+1-3 days)
Day  5: Learn DB NoSQL                             Review → HA, Compute (+2/+3 days)
Day  6: Learn Decoupling                           Review → DB Relational, DB NoSQL (+1/+2 days)
       ☁ REST DAY                                  Brain consolidates Days 1-6
Day  7: Learn Serverless                           Review → DB NoSQL, Decoupling (+1/+2 days)
Day  8: Learn Cost/Migration/DR                    Review → Networking, Storage (+7 days — critical)
Day  9: Learn CDN/DNS                              Review → Serverless, Decoupling (+2/+3 days)
Day 10: Learn Security                             Review → Networking, Storage, HA (+10 days — critical)
Day 11: Learn Monitoring + Containers + Additional Review → Cost, CDN/DNS (+2/+3 days)
Day 12: Full review                                Review → ALL 14 cheat sheets
       ☁ REST DAY                                  Brain consolidates Days 1-12
Day 13+: Mock exams                                Every mock reviews EVERYTHING at once
```

**Result:** By exam day, Day 1 content has been reviewed 12+ times across:
- 5 dedicated Block 1 reviews
- 1 full review day
- 9 mock exams that test all topics (585 practice questions)
- 2 rest days for memory consolidation

---

## When to Book the Exam

**Book it NOW for Thursday April 17, early afternoon slot.**

- Exam is booked — no backing out, only forward
- You can reschedule up to 24 hours before (AWS policy)
- Day 16 checkpoint tells you if you need to push back

---

## Mock Exam Score Tracker

```
Mock #1 (Day 13) Maarek #1:  ___% | Weak: _______________
Mock #2 (Day 14) Davis  #1:  ___% | Weak: _______________
Mock #3 (Day 15) Maarek #2:  ___% | Weak: _______________
Mock #4 (Day 16) Davis  #2:  ___% | Weak: _______________ ← CHECKPOINT
Mock #5 (Day 17) Maarek #3:  ___% | Weak: _______________
Mock #6 (Day 18) Davis  #3:  ___% | Weak: _______________
Mock #7 (Day 19) Maarek #4:  ___% | Weak: _______________
Mock #8 (Day 20) Davis  #4:  ___% | Weak: _______________ ← GO/NO-GO
Mock #9 (Day 21) Maarek #5:  ___% | Weak: _______________ ← FINAL (exam day morning)
```

**Healthy:** Each Maarek score 5-10% higher than the previous. Davis scores ~8% lower than Maarek is normal.
**Warning:** Plateau or drop for 2 consecutive → reread your 2 weakest chapters entirely.

---

## Calendar View

```
WEEK 1 — LEARN (Mar 26 - Apr 1)                    [3h/day, 50+10+50+10+50]
  Thu 26: Networking
  Fri 27: Storage
  Sat 28: Compute + High Availability
  Sun 29: Databases Relational
  Mon 30: Databases NoSQL
  Tue 31: Decoupling
  Wed  1: ☁ REST DAY

WEEK 2 — LEARN + REVIEW (Apr 2 - Apr 8)            [3h/day, 50+10+50+10+50]
  Thu  2: Serverless
  Fri  3: Cost/Migration/DR
  Sat  4: CDN/DNS
  Sun  5: Security & Identity
  Mon  6: Monitoring + Containers + Additional
  Tue  7: Full review + buffer
  Wed  8: ☁ REST DAY

WEEK 3 — MOCKS (Apr 9 - Apr 16)                    [3h/day, 65+10+50+10+45]
  Thu  9: Mock #1 (Maarek)
  Fri 10: Mock #2 (Davis)
  Sat 11: Mock #3 (Maarek)
  Sun 12: Mock #4 (Davis) ← CHECKPOINT
  Mon 13: Mock #5 (Maarek)
  Tue 14: Mock #6 (Davis)
  Wed 15: Mock #7 (Maarek)
  Thu 16: Mock #8 (Davis) ← GO/NO-GO

WEEK 4 — FINAL (Apr 17)
  Fri 17: Mock #9 (Maarek, morning) → ☁ REST → EXAM (early afternoon)
```

---

## If You Miss a Day

**Missed 1 day during Phase 1:** Use the first rest day (Apr 1) to catch up. If past the first rest day, use the second rest day (Apr 8). If both are used, extend to a 4th block (3.5h) the next day.

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
