# SAA-C03 Study Guide

**Start Date:** Thursday, March 26, 2026
**Pace:** 1.5 hours/day (2 hours on mock exam days)
**Exam Date:** Book for April 21, 2026
**Total Duration:** 27 days

---

## How This Plan Works

**Learning days (Days 1-12):** 90 min sessions split into:
- **Block A (20 min):** Review older material. Close the file, write what you remember, then check. This is what stops you from forgetting Day 1 by Day 12.
- **Block B (70 min):** New content + scenarios at end of chapter.

**Mock exam days (Days 14-23):** 2 hour sessions:
- Take exam under real conditions (~80 min)
- Review every wrong answer thoroughly (~40 min) — this is where most of the learning happens, don't rush it.

**3 weapons against forgetting:**
1. **Spaced review** — built into every day as Block A
2. **Scenarios** — each chapter ends with 5 exam-style scenarios that anchor the concepts
3. **Mock exams** — starting Day 15, every mock forces full recall of all 14 chapters at once

---

## Content Map

| Day | Chapter(s) | File(s) | Load |
|-----|-----------|---------|------|
| 1 | Networking | Networking.md | Medium |
| 2 | Storage | Storage.md | Medium |
| 3 | Compute | Compute.md | Medium |
| 4 | High Availability | HighAvailability.md | Medium |
| 5 | Databases Relational | DatabasesRelational.md | Heavy |
| 6 | Databases NoSQL | DatabasesNoSQL.md | Heavy |
| 7 | Serverless | Serverless.md | Heavy |
| 8 | Containers | Containers.md | Medium |
| 9 | Cost/Migration/DR | CostOptimization.md | Heavy |
| 10 | Decoupling | Decoupling.md | Heavy |
| 11 | Content Delivery & DNS | ContentDeliveryDNS.md | Medium |
| 12 | Security & Identity | SecurityIdentity.md | Heavy |
| 13 | Monitoring & Audit + Additional Services | MonitoringAudit.md + AdditionalServices.md | Paired |
| 14 | Full review + buffer | All cheat sheets | Review |

---

## Phase 1: Learn (Days 1-14)

---

### Day 1 — Thu Mar 26: Networking

**Block A:** None (first day).
**Block B (90 min):**
- Read Networking.md — focus on: SG vs NACL table, VPC Endpoints (Gateway = S3/DynamoDB free, Interface = everything else paid), NAT GW must be in public subnet, Transit Gateway = hub-and-spoke.
- Do all 5 networking scenarios. Cover the answer, explain why the other 3 are wrong.
- **Write from memory:** SG vs NACL table. VPC Endpoint types. Cheat sheet triggers.

---

### Day 2 — Fri Mar 27: Storage

**Block A (20 min):** Close Networking.md. From memory write: "Block specific IP → ?" "Access S3 privately free → ?" "Connect 50 VPCs → ?" "Instance has public IP but no internet → ?" Check against cheat sheet.
**Block B (70 min):**
- Read Storage.md — focus on: EBS type triggers (gp3 default, io1/io2 for IOPS, st1 for throughput), S3 storage classes waterfall, EFS vs FSx decision, S3 Express One Zone, Multi-Attach = io1/io2 only.
- Do all 5 storage scenarios.
- **Write from memory:** S3 storage classes in order with triggers. EBS type selection triggers.

---

### Day 3 — Sat Mar 28: Compute

**Block A (20 min):** Networking + Storage cheat sheets. Cover answers, test every trigger line. Mark any you miss.
**Block B (70 min):**
- Read Compute.md — focus on: purchasing options table (On-Demand/RI/Spot/Dedicated Host vs Instance), Placement Groups (Cluster/Spread/Partition), ENI vs ENA vs EFA, Hibernate limits (60 days, 150GB RAM, encrypted EBS).
- Do all 5 compute scenarios.
- **Write from memory:** Purchasing options table. Placement Groups table.

---

### Day 4 — Sun Mar 29: High Availability

**Block A (20 min):** Storage + Compute cheat sheets. Redo Networking Scenario 6 (NACL vs SG) from memory — can you nail it without looking?
**Block B (70 min):**
- Read HighAvailability.md — focus on: ALB vs NLB vs GWLB comparison table, NLB now supports Security Groups, all 5 scaling policy types (Target Tracking/Step/Simple/Scheduled/Predictive), Connection Draining, Warm Pool.
- Do all 5 HA scenarios.
- **Write from memory:** ALB vs NLB vs GWLB table. 5 scaling policies with triggers.

---

### Day 5 — Mon Mar 30: Databases Relational

**Block A (20 min):** Networking + Storage cheat sheets (Day 1-2, now +3/+4 days old — key reinforcement window). Rapid fire triggers only.
**Block B (70 min):**
- Read DatabasesRelational.md — focus on: Multi-AZ vs Read Replicas table (WILL be on exam), Aurora Serverless v2, Aurora Global Database (<1 sec replication, promote <1 min), RDS Proxy for Lambda, Backtrack (in-place rewind, seconds, 72h max).
- Do all 5 relational DB scenarios.
- **Write from memory:** Multi-AZ vs Read Replicas table. Aurora decision tree.

---

### Day 6 — Tue Mar 31: Databases NoSQL

**Block A (20 min):** HA + Compute cheat sheets. Test: "Static IP → ?" "Millions req/sec → ?" "GENEVE protocol → ?" "Per-socket licensing → ?" "Cheapest fault-tolerant batch → ?"
**Block B (70 min):**
- Read DatabasesNoSQL.md — focus on: Provisioned vs On-Demand, GSI vs LSI table, DynamoDB Streams + Lambda, DAX (microseconds) vs ElastiCache (general cache), Redis vs Memcached table, Global Tables (multi-region active-active).
- Do all 5 NoSQL scenarios.
- **Write from memory:** GSI vs LSI table. Redis vs Memcached table. RCU calc: 10 items/sec, 8KB, strongly consistent = 20 RCU.

---

### Day 7 — Wed Apr 1: Serverless

**Block A (20 min):** DB Relational + DB NoSQL cheat sheets (hardest chapters — quiz every line). Redo "GSI Throttling" scenario from memory.
**Block B (70 min):**
- Read Serverless.md — focus on: Lambda limits table (MEMORIZE: 15 min timeout, 10GB mem, 1MB async payload, 10GB /tmp, 1000 concurrent), invocation types, Provisioned Concurrency = kill cold starts, API Gateway throttling/caching, Step Functions Standard (1 year) vs Express (5 min).
- Do all 5 serverless scenarios.
- **Write from memory:** Lambda limits table. Step Functions Standard vs Express.

---

### Day 8 — Thu Apr 2: Containers

**Block A (20 min):** Networking + Storage cheat sheets (Day 1-2, now +7 days — critical 1-week reinforcement). If you can't recall 80%+ instantly, flag for extra review on Day 14.
**Block B (70 min):**
- Read Containers.md — focus on: ECS EC2 vs Fargate table, Task Role vs Execution Role, awsvpc mode, EFS for persistent shared storage, EKS = Kubernetes migration, Capacity Provider + ASG.
- Do all 5 container scenarios.
- **Write from memory:** ECS EC2 vs Fargate table. Task Role vs Execution Role difference.

---

### Day 9 — Fri Apr 3: Cost/Migration/DR

**Block A (20 min):** Compute + HA cheat sheets (rapid fire). Test: "Per-socket licensing → ?" "Cheapest fault-tolerant batch → ?" "Maintain CPU at 60% → ?" "Static IP → ?"
**Block B (70 min):**
- Read CostOptimization.md — focus on: Cost Explorer vs Budgets vs Compute Optimizer, DMS (+SCT for heterogeneous), DataSync vs Snow Family, DR patterns table (RPO/RTO — WILL be on exam), AWS Organizations SCPs, AWS Backup.
- Quiz yourself on all DR pattern triggers + migration service triggers.
- **Write from memory:** DR patterns table (4 strategies, RPO/RTO/Cost). "Lift-and-shift servers → ?" "Centralized backup → ?" "Cheapest DR → ?"

---

### Day 10 — Sat Apr 4: Decoupling

**Block A (20 min):** Serverless + Containers cheat sheets. Test: "Timeout > 15 min → ?" "Eliminate cold starts → ?" "Run containers without managing servers → ?" "Container needs S3 access → ?"
**Block B (70 min):**
- Read Decoupling.md — focus on: SQS Standard vs FIFO table, Long Polling (reduce API calls), Visibility Timeout, DLQ, SNS + SQS Fan-Out pattern, Kinesis Data Streams vs Data Firehose table, Amazon MQ = RabbitMQ migration only.
- Do all 5 decoupling scenarios.
- **Write from memory:** SQS Standard vs FIFO table. Kinesis Data Streams vs Data Firehose table.

---

### Day 11 — Sun Apr 5: Content Delivery & DNS

**Block A (20 min):** DB Relational + DB NoSQL cheat sheets (now 6 days old — reinforce before they fade). Redo "Lambda Connection Storm" scenario.
**Block B (70 min):**
- Read ContentDeliveryDNS.md — focus on: All 7 Route 53 routing policies table (WILL be on exam), Alias vs CNAME (root domain → Alias only), CloudFront OAC (not OAI), Signed URLs (single file) vs Signed Cookies (multiple files), CloudFront vs Global Accelerator table.
- Do all 5 CDN/DNS scenarios.
- **Write from memory:** All 7 routing policies with triggers. CloudFront vs Global Accelerator table.

---

### Day 12 — Mon Apr 6: Security & Identity

**Block A (20 min):** Decoupling + Cost/Migration cheat sheets. Test: "Strict ordering → ?" "Fan-out → ?" "Migrate RabbitMQ → ?" "Cheapest DR → ?" "Lift-and-shift servers → ?" "Centralized backup → ?"
**Block B (70 min):**
- Read SecurityIdentity.md — focus on: IAM evaluation logic (Deny > Allow > Default Deny), Permission Boundaries, KMS key types + Envelope Encryption, Secrets Manager vs Parameter Store table, Cognito User Pools vs Identity Pools table, WAF rate limiting, GuardDuty vs Inspector vs Macie table.
- Do all 5 security scenarios.
- **Write from memory:** IAM eval logic. Cognito User Pools vs Identity Pools. GuardDuty vs Inspector vs Macie.

---

### Day 13 — Tue Apr 7: Monitoring & Audit + Additional Services

**Block A (20 min):** Networking + Storage + HA cheat sheets (12+ days old — if you can't recall these, they need serious reinforcement on Day 14).
**Block B (70 min):**
- Read MonitoringAudit.md (45 min) — focus on: CloudWatch vs CloudTrail vs Config table (WILL be on exam), CloudWatch Agent for memory/disk (not default metrics), EventBridge = CloudWatch Events 2.0, SSM Session Manager (no SSH/port 22), SSM Patch Manager, Trusted Advisor 5 pillars.
- Read AdditionalServices.md (15 min) — quick reference for 1-2 question services: Athena (SQL on S3), Redshift (data warehouse), Glue (ETL), CloudFormation (IaC), Elastic Beanstalk (PaaS), OpenSearch (search/log analytics), AppSync (GraphQL), MSK (Kafka migration).
- Do all 5 monitoring scenarios. Skim additional services cheat sheet.
- **Write from memory:** CloudWatch vs CloudTrail vs Config vs VPC Flow Logs. "Query S3 with SQL → ?" "Data warehouse → ?" "ETL → ?" "IaC → ?"

---

### Day 14 — Wed Apr 8: Full Review Day + Buffer

**Full 90 min:** Go through ALL 14 chapter cheat sheets:
- Read each trigger/answer pair
- Cover the answer, test yourself
- Mark any you can't recall instantly → this becomes your **weak list**

Write your weak list on paper. You'll use it for the rest of the plan.

This day also serves as a **buffer** — if you fell behind by 1 day during Phase 1, use today to catch up on the last chapter instead, and do the full review as Block A on Day 15.

**Also today:** Buy mock exams if you haven't:
- Stephane Maarek SAA-C03 practice exams (Udemy)
- Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy)

**Book your exam today for April 21.**

---

## Phase 2: Mock Exams + Reinforcement (Days 15-26)

**Mock exam days = 2 hours.** The extra 30 min over normal sessions is specifically for proper wrong-answer review. Rushing the review defeats the purpose of mock exams.

**Pattern: Mock → Review → Mock → Review.** Never more than 2 consecutive mock days. Review days prevent burnout and let you fix gaps before the next mock exposes them again.

---

### Day 15 — Thu Apr 9: Mock #1 (Maarek) [2 hours]
**Take:** Stephane Maarek Exam #1 (~80 min)
**Review:** Every wrong answer + every flagged answer (~40 min). Go back to the chapter for each wrong topic.
**Expected score:** 55-70%. This is your baseline. Don't panic.
**After:** Update your weak list.

---

### Day 16 — Fri Apr 10: Review Day [90 min]
**Focus ONLY on topics you got wrong on Mock #1:**
- Reread the relevant chapter sections
- Redo the scenarios for those chapters
- Rewrite the cheat sheet items from memory
- Speed-run Chapters 1-7 cheat sheets (these are now 10-14 days old)

---

### Day 17 — Sat Apr 11: Mock #2 (Davis) [2 hours]
**Take:** Neal Davis / Tutorials Dojo Exam #1 (~80 min)
**Review:** All wrong answers (~40 min)
**Expected score:** 50-65% (Davis is intentionally harder — this is normal).
**After:** Notice patterns — same topics wrong in Mock #1 AND #2 = blind spots. Highlight these on your weak list.

---

### Day 18 — Sun Apr 12: Review Day [90 min]
- Blind spot topics from Mock #1 + #2 — reread those full chapter sections (40 min)
- Speed-run Chapters 8-14 cheat sheets (30 min)
- Redo your 5 weakest scenarios from any chapter (20 min)

---

### Day 19 — Mon Apr 13: Mock #3 (Maarek) [2 hours]
**Take:** Stephane Maarek Exam #2 (~80 min)
**Review:** All wrong answers (~40 min)
**Target score:** 65-78%. You should see improvement from Mock #1.
**After:** If same topics are STILL wrong, it's a comprehension gap not memory. Reread that chapter more carefully tonight.

---

### Day 20 — Tue Apr 14: Mock #4 (Davis) [2 hours]
**Take:** Neal Davis Exam #2 (~80 min)
**Review:** All wrong answers (~40 min)
**Target score:** 62-74%.
**After:** If DR patterns, cost tools, or migration questions keep tripping you, reread CostOptimization.md tonight.

---

### Day 21 — Wed Apr 15: Review Day [90 min]
- All 14 cheat sheets — rapid fire trigger/answer pairs (40 min)
- Your weak list — drill until you can recall every item (30 min)
- Focus on any topic wrong in 3+ mocks — these are your critical gaps (20 min)

---

### Day 22 — Thu Apr 16: Mock #5 (Maarek) — CHECKPOINT [2 hours]
**Take:** Stephane Maarek Exam #3 (~80 min)
**Review:** All wrong answers (~40 min)
**Target score:** 72-82%.

**Decision point:**

| Score | Action |
|-------|--------|
| 78%+ | Keep April 21. You're ready. |
| 72-77% | Keep April 21. Focus remaining days on weak list only. |
| 65-71% | Reschedule to April 28. Take 2 more mocks + review. |
| <65% | Reschedule to May 5. Reread your 3 weakest chapters fully. |

---

### Day 23 — Fri Apr 17: Mock #6 (Maarek) [2 hours]
**Take:** Stephane Maarek Exam #4 (~80 min)
**Review:** All wrong answers (~40 min)
**Target score:** 78-85%.

---

### Day 24 — Sat Apr 18: Mock #7 (Maarek) — GO/NO-GO [2 hours]
**Take:** Stephane Maarek Exam #5 (~80 min)
**Review:** All wrong answers (~40 min)
**Target score:** 80%+.

If 80%+: You're passing. Cruise to exam day.
If 75-79%: You'll likely pass. Review weak list, don't cram.
If <75%: Push exam back 1 week. Take remaining Davis exams.

---

### Day 25 — Sun Apr 19: Final Review [90 min]
- All 14 cheat sheets one last time (40 min)
- Your weak list (20 min)
- Redo your 5 hardest scenarios (20 min)
- DR patterns + cost tools + migration services (10 min)

---

### Day 26 — Mon Apr 20: Pre-Exam [45 min max]
- Skim all cheat sheets (don't study deeply, just skim)
- Optionally take Maarek Exam #6 as warm-up (don't obsess over score)
- Stop by early afternoon

Relax. Walk. Sleep well.

---

### Day 27 — Tue Apr 21: EXAM DAY

**Morning:**
- Light breakfast
- Skim your weak list one time (15 min max)
- Stop. You're ready.

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

Every learning day has a Block A review. Here's the rotation:

```
Day  1: Learn Networking                    (no review — first day)
Day  2: Learn Storage                       Review → Networking (+1 day)
Day  3: Learn Compute                       Review → Networking, Storage (+1/+2 days)
Day  4: Learn HA                            Review → Storage, Compute + Networking scenario (+2/+3 days)
Day  5: Learn DB Relational                 Review → Networking, Storage (+3/+4 days)
Day  6: Learn DB NoSQL                      Review → HA, Compute (+2/+3 days)
Day  7: Learn Serverless                    Review → DB Relational, DB NoSQL (+1/+2 days)
Day  8: Learn Containers                    Review → Networking, Storage (+7 days — critical)
Day  9: Learn Cost/Migration/DR             Review → Compute, HA (+5/+6 days)
Day 10: Learn Decoupling                    Review → Serverless, Containers (+3/+2 days)
Day 11: Learn CDN & DNS                     Review → DB Relational, DB NoSQL (+5/+6 days)
Day 12: Learn Security                      Review → Decoupling, Cost/Migration (+2/+3 days)
Day 13: Learn Monitoring + Additional       Review → Networking, Storage, HA (+12 days)
Day 14: Full review                         Review → ALL 14 cheat sheets
Day 15+: Mock exams                         Every mock reviews EVERYTHING at once
```

**Result:** By exam day, Day 1 content has been reviewed 10+ times across:
- 5 dedicated Block A reviews
- 1 full review day
- 7 mock exams that test all topics

---

## When to Book the Exam

**Book it on Day 14 (April 8) for April 21.**

- Having a locked date creates healthy pressure
- You can reschedule up to 24 hours before (AWS policy)
- Day 21 checkpoint tells you if you need to push back

---

## Mock Exam Score Tracker

```
Mock #1 (Day 15) Maarek #1:  ___% | Weak: _______________
Mock #2 (Day 17) Davis  #1:  ___% | Weak: _______________
Mock #3 (Day 19) Maarek #2:  ___% | Weak: _______________
Mock #4 (Day 20) Davis  #2:  ___% | Weak: _______________
Mock #5 (Day 22) Maarek #3:  ___% | Weak: _______________ ← CHECKPOINT
Mock #6 (Day 23) Maarek #4:  ___% | Weak: _______________
Mock #7 (Day 24) Maarek #5:  ___% | Weak: _______________ ← GO/NO-GO
```

**Healthy:** Each Maarek score 5-10% higher than the previous.
**Warning:** Plateau or drop for 2 consecutive → reread your 2 weakest chapters entirely.

---

## Calendar View

```
WEEK 1 — LEARN (Mar 26 - Apr 1)                    [90 min/day]
  Thu 26: Networking
  Fri 27: Storage
  Sat 28: Compute
  Sun 29: High Availability
  Mon 30: Databases Relational
  Tue 31: Databases NoSQL
  Wed  1: Serverless

WEEK 2 — LEARN + BUFFER (Apr 2 - Apr 8)            [90 min/day]
  Thu  2: Containers
  Fri  3: Cost/Migration/DR
  Sat  4: Decoupling
  Sun  5: Content Delivery & DNS
  Mon  6: Security & Identity
  Tue  7: Monitoring & Audit + Additional Services
  Wed  8: Full review + buffer (book exam today)

WEEK 3 — MOCKS + REVIEW (Apr 9 - Apr 15)           [alternating 90 min / 2h]
  Thu  9: Mock #1 (Maarek)
  Fri 10: Review day
  Sat 11: Mock #2 (Davis)
  Sun 12: Review day
  Mon 13: Mock #3 (Maarek)
  Tue 14: Mock #4 (Davis)
  Wed 15: Review day

WEEK 4 — FINAL (Apr 16 - Apr 21)
  Thu 16: Mock #5 (Maarek) ← CHECKPOINT
  Fri 17: Mock #6 (Maarek)
  Sat 18: Mock #7 (Maarek) ← GO/NO-GO
  Sun 19: Final review
  Mon 20: Pre-exam (light)
  Tue 21: EXAM DAY
```

---

## If You Miss a Day

**Missed 1 day during Phase 1:** Do 2 hours the next day to catch up. Don't skip Block A.

**Missed 1 day during Phase 2:** Skip the review day, not the mock. The mock is more valuable.

**Missed 2+ days:** Shift everything forward. Reschedule exam if needed.

**Golden rule:** Never skip Block A review to cram new content. Old material fading is worse than being 1 day behind.

---

## What You Need Before Day 1

- [ ] All 14 chapter files (already in this folder)
- [ ] Stephane Maarek SAA-C03 practice exams (Udemy) — buy by Day 12
- [ ] Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy) — buy by Day 12
- [ ] A notebook for writing from memory (paper > app for retention)
- [ ] AWS Certification account (aws.amazon.com/certification)
