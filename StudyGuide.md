# SAA-C03 Study Guide

**Start Date:** Monday, May 18, 2026
**Pace:** 2 hours/day (two 50-min blocks with a 10-min break)
**Rest Days:** 2 (mid-learning + pre-mocks consolidation)
**Target Exam Date:** Thursday, June 11, 2026
**Total Duration:** 25 calendar days — 13 learning days + 1 recall day + 1 review day + 8 mock days + 1 off day + 1 post-diagnostic rest
**Already done:** Networking ✅ · Compute ✅ · High Availability ✅ · Storage ✅ · DB Relational ✅ · DB NoSQL ✅ · Recall sweep ✅ · Decoupling ✅

---

## How This Plan Works

**The science behind this structure:**
- Sustained attention peaks at ~50 min then drops sharply. Two 50-min blocks keeps you in the high-retention zone the whole session.
- The 10-min break between blocks is critical — quiet rest after learning boosts memory consolidation by 15-40%.
- 2 hours/day is sustainable long-term. The trade-off vs a 3h sprint: more calendar days. That's fine — spacing helps retention.

**Learning days (Phase 1):**
```
Block 1 (50 min)  →  Break (10 min)  →  Block 2 (50 min)
  [Review old +        [Walk / snack]      [New content +
   start new content]                      scenarios + write from memory]
```

**Mock exam days (Phase 2):**
```
Block 1 (65 min)  →  Break (10 min)  →  Block 2 (50 min)
  [Take mock exam]      [Walk / stretch]    [Review every wrong answer +
                                             reread weak chapter sections]
```

**4 weapons against forgetting:**
1. **Spaced review** — every learning day opens by reviewing an older chapter's cheat sheet
2. **10-min rest break** — brain consolidates before new input
3. **Scenarios** — each chapter ends with 5 exam-style scenarios that anchor the concepts
4. **Mock exams** — 1 early diagnostic (May 25, after ~60% of content) + 8 Phase 2 mocks force full recall of all 15 chapters

---

## Content Map

| Day | Date | Chapter | File | Load |
|-----|------|---------|------|------|
| — | — | Networking | [Networking](./Networking.md) | ✅ done |
| 1 | Mon May 18 | Compute | [Compute](./Compute.md) | ✅ done |
| 2 | Tue May 19 | High Availability **+ Storage** | [HighAvailability](./HighAvailability.md) · [Storage](./Storage.md) | ✅ done |
| — | Wed May 20 | **OFF DAY** (no study) | — | — |
| 3 | Thu May 21 | Databases Relational | [DatabasesRelational](./DatabasesRelational.md) | ✅ done |
| 4 | Fri May 22 | Databases NoSQL | [DatabasesNoSQL](./DatabasesNoSQL.md) | ✅ done |
| 5 | Sat May 23 | **RECALL DAY** — full quiz across Networking → NoSQL ([Recall.md](./Recall.md)) | [Recall](./Recall.md) | ✅ done |
| 6 | Sun May 24 | Decoupling | [Decoupling](./Decoupling.md) | ✅ done |
| 6.5 | Mon May 25 | **DIAGNOSTIC MOCK** (Maarek #1) | — | Mock — **tomorrow** |
| 7 | Tue May 26 | Serverless | [Serverless](./Serverless.md) | ✅ done |
| 8 | Wed May 27 | Containers | [Containers](./Containers.md) | ✅ done |
| 9 | Thu May 28 | Cost / Migration / DR | [CostOptimization](./CostOptimization.md) | Heavy |
| 10 | Fri May 29 | CDN / DNS | [ContentDeliveryDNS](./ContentDeliveryDNS.md) | Medium |
| 11 | Sat May 30 | Security & Identity | [SecurityIdentity](./SecurityIdentity.md) | Heavy |
| 12 | Sun May 31 | Monitoring & Audit | [MonitoringAudit](./MonitoringAudit.md) | Medium |
| 13 | Mon Jun 1 | Additional Services + buffer | [AdditionalServices](./AdditionalServices.md) | Light |
| 14 | Tue Jun 2 | Full review — all cheat sheets | All | Review |

---

## Phase 1: Learn (Days 1-14)

**Every learning day follows the same shape:** Block 1 = review the previous chapter's cheat sheet (cover the answers, test yourself) + begin the new chapter. Break. Block 2 = finish the new chapter, do all 5 scenarios, write key tables from memory.

### Day 1 — Mon May 18: Compute ✅
- **Block 1:** Review Networking cheat sheet — quiz every trigger. Then start Compute.md: purchasing options table (On-Demand / RI / Spot / Dedicated Host vs Instance), instance families (T/M/C/R/I/G).
- **Block 2:** Finish Compute.md — Placement Groups (Cluster/Spread/Partition), ENI vs ENA vs EFA, Hibernate, Capacity Reservations, IMDSv2, Launch Templates. Do all 5 scenarios. **Write from memory:** purchasing table, placement group use cases.

### Day 2 — Tue May 19: High Availability + Storage ✅
- **Block 1:** Review Compute cheat sheet. Then HighAvailability.md: ALB vs NLB vs GWLB comparison table, cross-zone defaults, 5 scaling policies, Grace Period vs Cooldown, Connection Draining, Warm Pool, Instance Refresh.
- **Block 2:** Storage.md — EBS type triggers (gp3 default, io1/io2 high IOPS, st1 throughput), S3 storage classes waterfall, Intelligent-Tiering vs Lifecycle, Versioning, Replication (CRR/SRR), Object Lock, EFS vs FSx, Storage Gateway (File/Volume/Tape). Both chapters' scenarios.
- *Note: Storage was pulled forward from Day 5 — you're a day ahead. Sat May 23 is now a buffer.*

### ☁ Wed May 20: OFF DAY
No study. Plan resumes Thursday.

### Day 3 — Thu May 21: Databases Relational ✅
*The problem this chapter solves:* you need a **traditional SQL database** (tables, joins, transactions) but don't want to patch servers, handle failover, or babysit backups. RDS/Aurora are AWS running the database engine *for* you. The exam tests **how you make it survive failure (Multi-AZ) and scale reads (Read Replicas)** — that distinction is the heart of the chapter.
- **Block 1:** Review HA cheat sheet (quiz the triggers — cover the answer key). Then start DatabasesRelational.md: what RDS is, the 6 engines, storage/auto-scaling, then **Read Replicas** (scale reads, async, up to 5 RDS / 15 Aurora).
- **Block 2:** Finish DatabasesRelational.md — **Multi-AZ vs Read Replicas** (the key table: Multi-AZ = *survive failure*, Read Replica = *scale reads*), Aurora architecture, Aurora Serverless v2, Aurora Global Database, RDS Proxy, Backtrack, Blue/Green deployments. Do all 5 scenarios.
- **Write from memory:** Multi-AZ vs Read Replicas table; the 6 RDS engines.

### Day 4 — Fri May 22: Databases NoSQL ✅
*The problem this chapter solves:* sometimes a SQL database is the wrong tool — you need **massive scale, flexible schema, single-digit-ms latency**, or a *specific shape* of data (graph, time-series, ledger). This chapter is **DynamoDB** (the star — learn it deepest) plus a tour of **purpose-built databases** — for those, just learn *which one fits which data shape*.
- **Block 1:** Review DB Relational cheat sheet (quiz it). Then start DatabasesNoSQL.md: what NoSQL is and when to pick it over RDS; DynamoDB **capacity modes** (On-Demand vs Provisioned), **consistency** (eventual vs strong).
- **Block 2:** Finish DatabasesNoSQL.md — **GSI vs LSI** (key table), DynamoDB Streams, **DAX** (microsecond cache), Global Tables, TTL; then ElastiCache **Redis vs Memcached**; then the purpose-built tour — DocumentDB (Mongo), Neptune (graph), Timestream (time-series), QLDB (ledger), Keyspaces (Cassandra), MemoryDB (Redis-durable). Do all 5 scenarios.
- **Write from memory:** GSI vs LSI; Redis vs Memcached; purpose-built DB → data-shape table.
- *Heavy day — the purpose-built DBs are ~1 question each; learn the trigger word for each, don't go deep.*

### Day 5 — Sat May 23: RECALL DAY ✅
- Full recall sweep across everything covered so far (Networking → Compute → HA → Storage → DB Relational → DB NoSQL) using [Recall.md](./Recall.md). Cover the answers, quiz yourself, mark what didn't come back instantly.
- This replaces what was originally a buffer day — and it earns the right to skip the diagnostic mock today.

### Day 6 — Sun May 24: Decoupling ✅
- Covered: SQS Standard vs FIFO, Long Polling, Visibility Timeout, DLQ, SNS + SQS Fan-Out, SNS Filtering, Kinesis Data Streams vs Firehose, KCL/shards, Amazon MQ, EventBridge (Rules / Scheduler / Archive & Replay).
- Cheat-sheet score: **15/18**. Reds to re-drill: **SQS Extended Client Library** (>256 KB) and **EventBridge Archive & Replay** (replay past events). Both tagged 🔴 in [Recall.md](./Recall.md) §7.

### Day 6.5 — Mon May 25: DIAGNOSTIC MOCK ← **TOMORROW** (single 65-min block, then rest)

**Why this exists:** You've now covered Networking, Compute, HA, Storage, both Databases, and Decoupling — roughly **60% of exam content** and the heaviest-tested domains. This is the earliest point a mock score actually *means* something. It calibrates whether the plan is on track *before* you've sunk two more weeks in.

- **Single Block (65 min):** Take **one mock exam** (Maarek #1 — save Bonso/Davis for the real Phase 2). Take it under exam conditions. Don't study after — let the rest of the day do its job.
- **Do NOT review answers today.** Just record the raw score and which chapters the wrong answers clustered in.

**Calibration — read your score honestly:**

| Diagnostic Score | Read | Action |
|---|---|---|
| 65%+ | Ahead of pace — material is landing | Continue plan as-is. You can afford to go slightly faster if you want. |
| 50–64% | Normal for this stage (only ~60% covered) | Continue plan as-is. Note weak chapters for extra Block 1 review. |
| 40–49% | Behind, but recoverable | Continue, but add 15 min to each Block 1 review. Re-read weakest chapter fully on Day 13 buffer. |
| <40% | The plan's pace isn't working for you | **Stop and reassess** — either slow down (more days per heavy chapter) or push the exam date. Better to know now than at Day 18. |

**Important:** This is a *diagnostic*, not a verdict. A low score here with 40% of content unlearned is expected — the table accounts for that. The point is the *trend*: this score is your true baseline to measure the Phase 2 mocks against.

### ☁ Mon May 25 (rest of day): REST
After the diagnostic, no more studying. Your brain consolidates the material. This is when long-term memories form.

### Day 7 — Tue May 26: Serverless
- **Block 1:** Review Decoupling cheat sheet. Then start Serverless.md: Lambda limits (MEMORIZE: 15 min timeout, 10GB mem, 10GB /tmp), invocation types.
- **Block 2:** Finish Serverless.md — Execution Role, Provisioned Concurrency, Layers, Versions/Aliases, API Gateway (REST vs HTTP API), Step Functions (Standard vs Express). Do all 5 scenarios. **Write from memory:** Lambda limits, Step Functions Standard vs Express.

### Day 8 — Wed May 27: Containers
- **Block 1:** Review Serverless cheat sheet. Then start Containers.md: ECR, ECS overview, launch types.
- **Block 2:** Finish Containers.md — ECS EC2 vs Fargate, Task Role vs Execution Role, awsvpc mode, Capacity Providers (Fargate Spot), EKS, App Runner. Do all 5 scenarios. **Write from memory:** ECS EC2 vs Fargate, Task Role vs Execution Role.

### Day 9 — Thu May 28: Cost / Migration / DR
- **Block 1:** Review Networking + Storage cheat sheets (oldest material — critical reinforcement). Then start CostOptimization.md: Cost Explorer vs Budgets vs Compute Optimizer, Savings Plans, RI types.
- **Block 2:** Finish CostOptimization.md — data transfer cost rules, 6 R's of migration, DMS, Snow Family, DataSync, DR patterns table (RPO/RTO — WILL be on exam), Organizations SCPs, Control Tower, AWS Backup. Do all 5 scenarios. **Write from memory:** DR patterns table, 6 R's, data transfer rules.

### Day 10 — Fri May 29: CDN / DNS
- **Block 1:** Review Cost cheat sheet. Then start ContentDeliveryDNS.md: 7 Route 53 routing policies table, Resolver, Alias vs CNAME.
- **Block 2:** Finish ContentDeliveryDNS.md — CloudFront OAC, Signed URLs vs Cookies, CloudFront Functions vs Lambda@Edge, Origin Groups, CloudFront vs Global Accelerator. Do all 5 scenarios. **Write from memory:** 7 routing policies, CloudFront Functions vs Lambda@Edge.

### Day 11 — Sat May 30: Security & Identity
- **Block 1:** Review Networking + Storage + HA cheat sheets. Then start SecurityIdentity.md: IAM evaluation logic (Deny > Allow > Default Deny), Permission Boundaries, STS, IAM Identity Center.
- **Block 2:** Finish SecurityIdentity.md — KMS key types + Envelope Encryption, KMS vs CloudHSM, ACM, Secrets Manager vs Parameter Store, Cognito User Pools vs Identity Pools, WAF, GuardDuty/Inspector/Macie. Do all 5 scenarios. **Write from memory:** IAM eval logic, KMS vs CloudHSM, Cognito pools.

### Day 12 — Sun May 31: Monitoring & Audit
- **Block 1:** Review Security cheat sheet. Then start MonitoringAudit.md: CloudWatch vs CloudTrail vs Config table, CloudWatch Agent.
- **Block 2:** Finish MonitoringAudit.md — Contributor Insights, Synthetics, X-Ray, SSM Session Manager, Patch Manager, Trusted Advisor. Do all 5 scenarios. **Write from memory:** CloudWatch vs CloudTrail vs Config table.

### Day 13 — Mon Jun 1: Additional Services + Buffer
- **Block 1:** Review Monitoring cheat sheet. Then read AdditionalServices.md — Athena, Redshift, Glue, CloudFormation, Elastic Beanstalk, OpenSearch, Batch, EMR, ML/AI services, Outposts/Local Zones/Wavelength. These are 1-2 questions each — skim, don't memorize deeply.
- **Block 2:** Do all 5 Additional scenarios. **Buffer:** if you fell behind by a day during Phase 1, use this block to catch up instead.

### Day 14 — Tue Jun 2: Full Review Day
- **Block 1:** ALL 15 chapter cheat sheets — rapid fire. Cover the answer, test yourself. Mark anything you can't recall instantly.
- **Block 2:** Write your **weak list** on paper — every trigger/answer you missed. Redo the 5 hardest scenarios across all chapters.

---

## Phase 2: Mock Exams + Reinforcement (Days 15-22)

**Every day = 2 hours:**
```
Block 1 (65 min): Take mock exam under real conditions
Break (10 min): Walk, decompress
Block 2 (50 min): Review EVERY wrong answer — go back to the chapter for each topic
```

**8 mocks in 8 days.** The shorter Block 2 (vs a 3h plan) means tighter review — focus only on wrong answers, not generic rereading.

**Rotation: Maarek (straightforward), Davis (hardest), Bonso (closest to real exam).**

| Day | Date | Mock | Notes |
|-----|------|------|-------|
| 15 | Wed Jun 3 | Maarek #2 | First *full-content* mock. Compare against the May 25 diagnostic — expect a clear jump. |
| 16 | Thu Jun 4 | Davis #1 | Davis runs ~8% lower — normal. |
| 17 | Fri Jun 5 | Bonso #1 | Closest to real exam — most reliable signal. |
| 18 | Sat Jun 6 | Davis #2 | **CHECKPOINT** — see decision table below. |
| 19 | Sun Jun 7 | Maarek #3 | Target 75-82%. |
| 20 | Mon Jun 8 | Bonso #2 | Compare with Bonso #1 — expect clear improvement. |
| 21 | Tue Jun 9 | Davis #3 | **GO/NO-GO**. Target 68-75% (Davis scale). |
| 22 | Wed Jun 10 | Bonso #3 | Final tune-up. Target 75-85%. |
| — | Thu Jun 11 | **EXAM DAY** | Light skim of weak list only. No new study. |

*(Maarek #1 was used for the May 25 diagnostic, so Phase 2 starts from Maarek #2. Total mocks taken: 1 diagnostic + 8 Phase 2 = 9.)*

### Day 18 Checkpoint (after Davis #2)

| Davis Score | Real Exam Equivalent | Action |
|-------------|---------------------|--------|
| 68%+ | ~75%+ | On track. Keep June 11. |
| 60-67% | ~68-74% | Keep June 11. Focus remaining days only on weak list. |
| 52-59% | ~60-67% | Consider pushing exam back 1 week. |
| <52% | ~<60% | Reschedule. Reread your 3 weakest chapters fully. |

### Day 21 Go/No-Go (after Davis #3)
- **68%+ on Davis:** You're passing. Light review only.
- **<60% on Davis:** Push the exam back 1 week.

---

## Exam Day — Thu June 11

**Exam strategy (65 questions, 130 min):**
- **Pass 1 (45 min):** Answer confident questions, flag everything else.
- **Pass 2 (60 min):** Work flagged questions — eliminate 2 wrong answers first.
- **Pass 3 (25 min):** Review flagged, trust your first instinct.
- Watch for: "NOT", "EXCEPT", "MOST cost-effective", "LEAST operational overhead".
- When stuck: managed AWS service > custom code. Simpler > complex.
- Answer everything — no penalty for guessing.

**Passing score:** 720/1000 (~72%).

---

## Mock Exam Score Tracker

```
DIAGNOSTIC (May 25) Maarek #1:  _50__% | Weak: _______________ ← BASELINE (~60% content)
Mock #1 (Day 15) Maarek #2:  ___% | Weak: _______________
Mock #2 (Day 16) Davis  #1:  ___% | Weak: _______________
Mock #3 (Day 17) Bonso  #1:  ___% | Weak: _______________
Mock #4 (Day 18) Davis  #2:  ___% | Weak: _______________ ← CHECKPOINT
Mock #5 (Day 19) Maarek #3:  ___% | Weak: _______________
Mock #6 (Day 20) Bonso  #2:  ___% | Weak: _______________
Mock #7 (Day 21) Davis  #3:  ___% | Weak: _______________ ← GO/NO-GO
Mock #8 (Day 22) Bonso  #3:  ___% | Weak: _______________
```

**Healthy:** scores trend upward from the diagnostic. Davis ~8% below Maarek is normal. Use Bonso scores as your best predictor.
**Warning:** plateau or drop for 2 consecutive mocks → reread your 2 weakest chapters entirely.

---

## If You Miss a Day

- **Missed 1 day in Phase 1:** Use the Day 13 buffer block, or the post-diagnostic rest (May 25), to catch up.
- **Missed 1 day in Phase 2:** Drop to 7 mocks — skip Maarek #2 (Day 19), keep both go/no-go signals.
- **Missed 2+ days:** Shift everything forward and reschedule the exam.
- **Golden rule:** never skip the Block 1 review to cram new content. Fading old material is worse than being a day behind.

---

## What You Need

- [x] All 15 chapter files (in this folder — see [Chapters.md](./Chapters.md))
- [ ] Stephane Maarek SAA-C03 practice exams (Udemy) — **buy before Mon May 25** (needed for the diagnostic mock)
- [ ] Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy) — buy by Day 12
- [ ] A notebook for writing from memory (paper > app for retention)
- [ ] A timer for the 50-min blocks
- [ ] AWS Certification account + booked exam slot for June 11
