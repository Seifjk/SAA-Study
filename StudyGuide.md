# SAA-C03 Study Guide

**Start Date:** Monday, May 18, 2026
**Pace:** 2 hours/day (two 50-min blocks with a 10-min break)
**Rest Days:** 2 (mid-learning + pre-mocks consolidation)
**Target Exam Date:** Thursday, June 11, 2026
**Total Duration:** 25 calendar days — 13 learning days + 1 review day + 8 mock days + 2 rest days
**Already done:** Networking ✅

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
4. **Mock exams** — 8 mocks force full recall of all 15 chapters

---

## Content Map

| Day | Date | Chapter | File | Load |
|-----|------|---------|------|------|
| — | — | Networking | [Networking](./Networking.md) | ✅ done |
| 1 | Mon May 18 | Storage | [Storage](./Storage.md) | Medium |
| 2 | Tue May 19 | Compute | [Compute](./Compute.md) | Medium |
| 3 | Wed May 20 | High Availability | [HighAvailability](./HighAvailability.md) | Medium |
| 4 | Thu May 21 | Databases Relational | [DatabasesRelational](./DatabasesRelational.md) | Heavy |
| 5 | Fri May 22 | Databases NoSQL | [DatabasesNoSQL](./DatabasesNoSQL.md) | Heavy |
| 6 | Sat May 23 | Decoupling | [Decoupling](./Decoupling.md) | Heavy |
| — | Sun May 24 | **REST DAY** | — | — |
| 7 | Mon May 25 | Serverless | [Serverless](./Serverless.md) | Heavy |
| 8 | Tue May 26 | Containers | [Containers](./Containers.md) | Medium |
| 9 | Wed May 27 | Cost / Migration / DR | [CostOptimization](./CostOptimization.md) | Heavy |
| 10 | Thu May 28 | CDN / DNS | [ContentDeliveryDNS](./ContentDeliveryDNS.md) | Medium |
| 11 | Fri May 29 | Security & Identity | [SecurityIdentity](./SecurityIdentity.md) | Heavy |
| 12 | Sat May 30 | Monitoring & Audit | [MonitoringAudit](./MonitoringAudit.md) | Medium |
| 13 | Sun May 31 | Additional Services + buffer | [AdditionalServices](./AdditionalServices.md) | Light |
| — | Mon Jun 1 | **REST DAY** | — | — |
| 14 | Tue Jun 2 | Full review — all cheat sheets | All | Review |

---

## Phase 1: Learn (Days 1-14)

**Every learning day follows the same shape:** Block 1 = review the previous chapter's cheat sheet (cover the answers, test yourself) + begin the new chapter. Break. Block 2 = finish the new chapter, do all 5 scenarios, write key tables from memory.

### Day 1 — Mon May 18: Storage
- **Block 1:** Review Networking cheat sheet — quiz every trigger. Then start Storage.md: EBS type triggers (gp3 default, io1/io2 for high IOPS, st1 for throughput), Instance Store = ephemeral.
- **Block 2:** Finish Storage.md — S3 storage classes waterfall (Standard → IA → One Zone-IA → Glacier IR → Glacier Flexible → Deep Archive), Intelligent-Tiering, Versioning, Lifecycle, Replication (CRR/SRR), Object Lock (Governance vs Compliance), EFS vs FSx, DataSync vs Storage Gateway. Do all 5 scenarios. **Write from memory:** S3 classes in order, EBS type selection triggers.

### Day 2 — Tue May 19: Compute
- **Block 1:** Review Storage cheat sheet. Then start Compute.md: purchasing options table (On-Demand / RI / Spot / Dedicated Host vs Instance), instance families (T/M/C/R/I/G).
- **Block 2:** Finish Compute.md — Placement Groups (Cluster/Spread/Partition), ENI vs ENA vs EFA, Hibernate, Capacity Reservations, IMDSv2, Launch Templates. Do all 5 scenarios. **Write from memory:** purchasing table, placement group use cases.

### Day 3 — Wed May 20: High Availability
- **Block 1:** Review Compute cheat sheet. Then start HighAvailability.md: ALB vs NLB vs GWLB comparison table, cross-zone defaults.
- **Block 2:** Finish HighAvailability.md — 5 scaling policies (Target Tracking/Step/Simple/Scheduled/Predictive), SQS-based scaling, Grace Period vs Cooldown, Connection Draining, Warm Pool, Instance Refresh. Do all 5 scenarios. **Write from memory:** ALB vs NLB vs GWLB table, 5 scaling policies.

### Day 4 — Thu May 21: Databases Relational
- **Block 1:** Review HA cheat sheet. Then start DatabasesRelational.md: RDS overview, storage, Read Replicas.
- **Block 2:** Finish DatabasesRelational.md — Multi-AZ vs Read Replicas table (RDS = 5 replicas, Aurora = 15), Aurora Serverless v2, Aurora Global Database, RDS Proxy, Backtrack, Blue/Green. Do all 5 scenarios. **Write from memory:** Multi-AZ vs Read Replicas table.

### Day 5 — Fri May 22: Databases NoSQL
- **Block 1:** Review DB Relational cheat sheet. Then start DatabasesNoSQL.md: DynamoDB modes, consistency.
- **Block 2:** Finish DatabasesNoSQL.md — GSI vs LSI, Streams, DAX, Global Tables, TTL, Redis vs Memcached, purpose-built DBs (DocumentDB/Neptune/Timestream/QLDB/Keyspaces/MemoryDB). Do all 5 scenarios. **Write from memory:** GSI vs LSI, Redis vs Memcached, purpose-built DB table.

### Day 6 — Sat May 23: Decoupling
- **Block 1:** Review DB NoSQL cheat sheet. Then start Decoupling.md: SQS Standard vs FIFO, polling, visibility timeout, DLQ.
- **Block 2:** Finish Decoupling.md — SNS + SQS fan-out, Kinesis Streams vs Firehose, EventBridge. Do all 5 scenarios. **Write from memory:** SQS Standard vs FIFO, Kinesis Streams vs Firehose.

### ☁ Sun May 24: REST DAY
No studying. Your brain consolidates 6 days of material. This is when long-term memories form.

### Day 7 — Mon May 25: Serverless
- **Block 1:** Review Decoupling cheat sheet. Then start Serverless.md: Lambda limits (MEMORIZE: 15 min timeout, 10GB mem, 10GB /tmp), invocation types.
- **Block 2:** Finish Serverless.md — Execution Role, Provisioned Concurrency, Layers, Versions/Aliases, API Gateway (REST vs HTTP API), Step Functions (Standard vs Express). Do all 5 scenarios. **Write from memory:** Lambda limits, Step Functions Standard vs Express.

### Day 8 — Tue May 26: Containers
- **Block 1:** Review Serverless cheat sheet. Then start Containers.md: ECR, ECS overview, launch types.
- **Block 2:** Finish Containers.md — ECS EC2 vs Fargate, Task Role vs Execution Role, awsvpc mode, Capacity Providers (Fargate Spot), EKS, App Runner. Do all 5 scenarios. **Write from memory:** ECS EC2 vs Fargate, Task Role vs Execution Role.

### Day 9 — Wed May 27: Cost / Migration / DR
- **Block 1:** Review Networking + Storage cheat sheets (oldest material — critical reinforcement). Then start CostOptimization.md: Cost Explorer vs Budgets vs Compute Optimizer, Savings Plans, RI types.
- **Block 2:** Finish CostOptimization.md — data transfer cost rules, 6 R's of migration, DMS, Snow Family, DataSync, DR patterns table (RPO/RTO — WILL be on exam), Organizations SCPs, Control Tower, AWS Backup. Do all 5 scenarios. **Write from memory:** DR patterns table, 6 R's, data transfer rules.

### Day 10 — Thu May 28: CDN / DNS
- **Block 1:** Review Cost cheat sheet. Then start ContentDeliveryDNS.md: 7 Route 53 routing policies table, Resolver, Alias vs CNAME.
- **Block 2:** Finish ContentDeliveryDNS.md — CloudFront OAC, Signed URLs vs Cookies, CloudFront Functions vs Lambda@Edge, Origin Groups, CloudFront vs Global Accelerator. Do all 5 scenarios. **Write from memory:** 7 routing policies, CloudFront Functions vs Lambda@Edge.

### Day 11 — Fri May 29: Security & Identity
- **Block 1:** Review Networking + Storage + HA cheat sheets. Then start SecurityIdentity.md: IAM evaluation logic (Deny > Allow > Default Deny), Permission Boundaries, STS, IAM Identity Center.
- **Block 2:** Finish SecurityIdentity.md — KMS key types + Envelope Encryption, KMS vs CloudHSM, ACM, Secrets Manager vs Parameter Store, Cognito User Pools vs Identity Pools, WAF, GuardDuty/Inspector/Macie. Do all 5 scenarios. **Write from memory:** IAM eval logic, KMS vs CloudHSM, Cognito pools.

### Day 12 — Sat May 30: Monitoring & Audit
- **Block 1:** Review Security cheat sheet. Then start MonitoringAudit.md: CloudWatch vs CloudTrail vs Config table, CloudWatch Agent.
- **Block 2:** Finish MonitoringAudit.md — Contributor Insights, Synthetics, X-Ray, SSM Session Manager, Patch Manager, Trusted Advisor. Do all 5 scenarios. **Write from memory:** CloudWatch vs CloudTrail vs Config table.

### Day 13 — Sun May 31: Additional Services + Buffer
- **Block 1:** Review Monitoring cheat sheet. Then read AdditionalServices.md — Athena, Redshift, Glue, CloudFormation, Elastic Beanstalk, OpenSearch, Batch, EMR, ML/AI services, Outposts/Local Zones/Wavelength. These are 1-2 questions each — skim, don't memorize deeply.
- **Block 2:** Do all 5 Additional scenarios. **Buffer:** if you fell behind by a day during Phase 1, use this block to catch up instead.

### ☁ Mon Jun 1: REST DAY
No studying. You've finished all 15 chapters. Let your brain consolidate before the mock gauntlet.

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
| 15 | Wed Jun 3 | Maarek #1 | Baseline. Expect 55-70%. Don't panic. |
| 16 | Thu Jun 4 | Davis #1 | Davis runs ~8% lower — normal. |
| 17 | Fri Jun 5 | Bonso #1 | Closest to real exam — most reliable signal. |
| 18 | Sat Jun 6 | Davis #2 | **CHECKPOINT** — see decision table below. |
| 19 | Sun Jun 7 | Maarek #2 | Target 75-82%. |
| 20 | Mon Jun 8 | Bonso #2 | Compare with Bonso #1 — expect clear improvement. |
| 21 | Tue Jun 9 | Davis #3 | **GO/NO-GO**. Target 68-75% (Davis scale). |
| 22 | Wed Jun 10 | Bonso #3 | Final tune-up. Target 75-85%. |
| — | Thu Jun 11 | **EXAM DAY** | Light skim of weak list only. No new study. |

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
Mock #1 (Day 15) Maarek #1:  ___% | Weak: _______________
Mock #2 (Day 16) Davis  #1:  ___% | Weak: _______________
Mock #3 (Day 17) Bonso  #1:  ___% | Weak: _______________
Mock #4 (Day 18) Davis  #2:  ___% | Weak: _______________ ← CHECKPOINT
Mock #5 (Day 19) Maarek #2:  ___% | Weak: _______________
Mock #6 (Day 20) Bonso  #2:  ___% | Weak: _______________
Mock #7 (Day 21) Davis  #3:  ___% | Weak: _______________ ← GO/NO-GO
Mock #8 (Day 22) Bonso  #3:  ___% | Weak: _______________
```

**Healthy:** scores trend upward. Davis ~8% below Maarek is normal. Use Bonso scores as your best predictor.
**Warning:** plateau or drop for 2 consecutive mocks → reread your 2 weakest chapters entirely.

---

## If You Miss a Day

- **Missed 1 day in Phase 1:** Use the Day 13 buffer block, or the rest days (May 24 / Jun 1), to catch up.
- **Missed 1 day in Phase 2:** Drop to 7 mocks — skip Maarek #2 (Day 19), keep both go/no-go signals.
- **Missed 2+ days:** Shift everything forward and reschedule the exam.
- **Golden rule:** never skip the Block 1 review to cram new content. Fading old material is worse than being a day behind.

---

## What You Need

- [x] All 15 chapter files (in this folder — see [Chapters.md](./Chapters.md))
- [ ] Stephane Maarek SAA-C03 practice exams (Udemy) — buy by Day 12
- [ ] Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy) — buy by Day 12
- [ ] A notebook for writing from memory (paper > app for retention)
- [ ] A timer for the 50-min blocks
- [ ] AWS Certification account + booked exam slot for June 11
