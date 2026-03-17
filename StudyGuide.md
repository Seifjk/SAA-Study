# SAA-C03 Study Guide

**Start Date:** Thursday, March 26, 2026
**Pace:** 2 hours/day (two 50-min blocks with 10-min break)
**Rest Days:** Every Wednesday (1 per week — memory consolidation)
**Exam Date:** Book for April 23, 2026
**Total Duration:** 29 calendar days (25 study days + 4 rest days)

---

## How This Plan Works

**The science behind this structure:**
- Sustained attention peaks at ~50 min then drops sharply. Two 50-min blocks keeps you in the high-retention zone the entire session.
- The 10-min break between blocks is critical — quiet rest after learning boosts memory consolidation by 15-40%.
- One rest day per week is backed by research — memory solidifies during rest, not during study. Studying 7 days straight actually reduces retention vs 6 on / 1 off.
- 2 hours/day is the evidence-based sweet spot for working professionals: long enough to go deep, short enough to sustain for 4 weeks without burnout.

**Learning days (Phase 1):**
```
Block 1 (50 min)  →  Break (10 min)  →  Block 2 (50 min)
  [Review old]         [Walk/snack]        [New content + scenarios]
```

**Mock exam days (Phase 2):**
```
Block 1 (60 min)  →  Break (10 min)  →  Block 2 (50 min)
  [Take mock exam]     [Walk/stretch]      [Review wrong answers]
```

**4 weapons against forgetting:**
1. **Spaced review** — Block 1 of every day reviews older material
2. **10-min rest breaks** — brain consolidates what you just reviewed before new input
3. **Scenarios** — each chapter ends with 5 exam-style scenarios that anchor concepts
4. **Mock exams** — starting Day 17, every mock forces full recall of all 14 chapters

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
| — | **REST DAY** | — | — |
| 7 | Serverless | Serverless.md | Heavy |
| 8 | Containers | Containers.md | Medium |
| 9 | Cost/Migration/DR | CostOptimization.md | Heavy |
| 10 | Decoupling | Decoupling.md | Heavy |
| 11 | Content Delivery & DNS | ContentDeliveryDNS.md | Medium |
| 12 | Security & Identity | SecurityIdentity.md | Heavy |
| — | **REST DAY** | — | — |
| 13 | Monitoring & Audit + Additional Services | MonitoringAudit.md + AdditionalServices.md | Paired |
| 14 | Full review + buffer | All cheat sheets | Review |

---

## Phase 1: Learn (Days 1-14)

---

### Day 1 — Thu Mar 26: Networking

**Block 1 (50 min):** First day — no review. Start reading Networking.md. Focus on: SG vs NACL table, VPC Endpoints (Gateway = S3/DynamoDB free, Interface = everything else paid), NAT GW must be in public subnet, Transit Gateway = hub-and-spoke.
**Break (10 min):** Walk away from screen. Let it settle.
**Block 2 (50 min):** Finish Networking.md. Do all 5 networking scenarios — cover the answer, explain why the other 3 are wrong. **Write from memory:** SG vs NACL table. VPC Endpoint types. Cheat sheet triggers.

---

### Day 2 — Fri Mar 27: Storage

**Block 1 (50 min):** Close Networking.md. From memory write: "Block specific IP → ?" "Access S3 privately free → ?" "Connect 50 VPCs → ?" "Instance has public IP but no internet → ?" Check against cheat sheet. Redo any you missed. Then start reading Storage.md.
**Break (10 min)**
**Block 2 (50 min):** Finish Storage.md — focus on: EBS type triggers (gp3 default, io1/io2 for IOPS, st1 for throughput), S3 storage classes waterfall, EFS vs FSx decision, S3 Express One Zone, Multi-Attach = io1/io2 only. Do all 5 storage scenarios. **Write from memory:** S3 storage classes in order. EBS type selection triggers.

---

### Day 3 — Sat Mar 28: Compute

**Block 1 (50 min):** Networking + Storage cheat sheets. Cover answers, test every trigger line. Mark any you miss — these go on your weak list. Start reading Compute.md.
**Break (10 min)**
**Block 2 (50 min):** Finish Compute.md — focus on: purchasing options table (On-Demand/RI/Spot/Dedicated Host vs Instance), Placement Groups (Cluster/Spread/Partition), ENI vs ENA vs EFA, Hibernate limits (60 days, 150GB RAM, encrypted EBS). Do all 5 compute scenarios. **Write from memory:** Purchasing options table. Placement Groups table.

---

### Day 4 — Sun Mar 29: High Availability

**Block 1 (50 min):** Storage + Compute cheat sheets. Redo Networking Scenario 6 (NACL vs SG) from memory. Start reading HighAvailability.md.
**Break (10 min)**
**Block 2 (50 min):** Finish HighAvailability.md — focus on: ALB vs NLB vs GWLB comparison table, NLB now supports Security Groups, all 5 scaling policy types (Target Tracking/Step/Simple/Scheduled/Predictive), Connection Draining, Warm Pool. Do all 5 HA scenarios. **Write from memory:** ALB vs NLB vs GWLB table. 5 scaling policies with triggers.

---

### Day 5 — Mon Mar 30: Databases Relational

**Block 1 (50 min):** Networking + Storage cheat sheets (Day 1-2, now +3/+4 days old — key reinforcement window). Rapid fire triggers only. Start reading DatabasesRelational.md.
**Break (10 min)**
**Block 2 (50 min):** Finish DatabasesRelational.md — focus on: Multi-AZ vs Read Replicas table (WILL be on exam), Aurora Serverless v2, Aurora Global Database (<1 sec replication, promote <1 min), RDS Proxy for Lambda, Backtrack (in-place rewind, seconds, 72h max). Do all 5 relational DB scenarios. **Write from memory:** Multi-AZ vs Read Replicas table. Aurora decision tree.

---

### Day 6 — Tue Mar 31: Databases NoSQL

**Block 1 (50 min):** HA + Compute cheat sheets. Test: "Static IP → ?" "Millions req/sec → ?" "GENEVE protocol → ?" "Per-socket licensing → ?" "Cheapest fault-tolerant batch → ?" Start reading DatabasesNoSQL.md.
**Break (10 min)**
**Block 2 (50 min):** Finish DatabasesNoSQL.md — focus on: Provisioned vs On-Demand, GSI vs LSI table, DynamoDB Streams + Lambda, DAX (microseconds) vs ElastiCache (general cache), Redis vs Memcached table, Global Tables (multi-region active-active). Do all 5 NoSQL scenarios. **Write from memory:** GSI vs LSI table. Redis vs Memcached table. RCU calc: 10 items/sec, 8KB, strongly consistent = 20 RCU.

---

### ☁ Wed Apr 1: REST DAY

No studying. Walk, exercise, do anything else. Your brain is consolidating 6 days of material right now. This is not wasted time — it's when long-term memories form.

---

### Day 7 — Thu Apr 2: Serverless

**Block 1 (50 min):** DB Relational + DB NoSQL cheat sheets (hardest chapters — quiz every line). Redo "GSI Throttling" scenario from memory. Start reading Serverless.md.
**Break (10 min)**
**Block 2 (50 min):** Finish Serverless.md — focus on: Lambda limits table (MEMORIZE: 15 min timeout, 10GB mem, 1MB async payload, 10GB /tmp, 1000 concurrent), invocation types, Provisioned Concurrency = kill cold starts, API Gateway throttling/caching, Step Functions Standard (1 year) vs Express (5 min). Do all 5 serverless scenarios. **Write from memory:** Lambda limits table. Step Functions Standard vs Express.

---

### Day 8 — Fri Apr 3: Containers

**Block 1 (50 min):** Networking + Storage cheat sheets (Day 1-2, now +8 days — critical 1-week+ reinforcement). If you can't recall 80%+ instantly, flag for extra review on Day 14. Start reading Containers.md.
**Break (10 min)**
**Block 2 (50 min):** Finish Containers.md — focus on: ECS EC2 vs Fargate table, Task Role vs Execution Role, awsvpc mode, EFS for persistent shared storage, EKS = Kubernetes migration, Capacity Provider + ASG. Do all 5 container scenarios. **Write from memory:** ECS EC2 vs Fargate table. Task Role vs Execution Role difference.

---

### Day 9 — Sat Apr 4: Cost/Migration/DR

**Block 1 (50 min):** Compute + HA cheat sheets (rapid fire). Test: "Per-socket licensing → ?" "Cheapest fault-tolerant batch → ?" "Maintain CPU at 60% → ?" "Static IP → ?" Start reading CostOptimization.md.
**Break (10 min)**
**Block 2 (50 min):** Finish CostOptimization.md — focus on: Cost Explorer vs Budgets vs Compute Optimizer, DMS (+SCT for heterogeneous), DataSync vs Snow Family, DR patterns table (RPO/RTO — WILL be on exam), AWS Organizations SCPs, AWS Backup. Quiz yourself on all DR pattern triggers + migration service triggers. **Write from memory:** DR patterns table (4 strategies, RPO/RTO/Cost). "Lift-and-shift servers → ?" "Centralized backup → ?" "Cheapest DR → ?"

---

### Day 10 — Sun Apr 5: Decoupling

**Block 1 (50 min):** Serverless + Containers cheat sheets. Test: "Timeout > 15 min → ?" "Eliminate cold starts → ?" "Run containers without managing servers → ?" "Container needs S3 access → ?" Start reading Decoupling.md.
**Break (10 min)**
**Block 2 (50 min):** Finish Decoupling.md — focus on: SQS Standard vs FIFO table, Long Polling (reduce API calls), Visibility Timeout, DLQ, SNS + SQS Fan-Out pattern, Kinesis Data Streams vs Data Firehose table, Amazon MQ = RabbitMQ migration only. Do all 5 decoupling scenarios. **Write from memory:** SQS Standard vs FIFO table. Kinesis Data Streams vs Data Firehose table.

---

### Day 11 — Mon Apr 6: Content Delivery & DNS

**Block 1 (50 min):** DB Relational + DB NoSQL cheat sheets (now 6+ days old — reinforce before they fade). Redo "Lambda Connection Storm" scenario. Start reading ContentDeliveryDNS.md.
**Break (10 min)**
**Block 2 (50 min):** Finish ContentDeliveryDNS.md — focus on: All 7 Route 53 routing policies table (WILL be on exam), Alias vs CNAME (root domain → Alias only), CloudFront OAC (not OAI), Signed URLs (single file) vs Signed Cookies (multiple files), CloudFront vs Global Accelerator table. Do all 5 CDN/DNS scenarios. **Write from memory:** All 7 routing policies with triggers. CloudFront vs Global Accelerator table.

---

### Day 12 — Tue Apr 7: Security & Identity

**Block 1 (50 min):** Decoupling + Cost/Migration cheat sheets. Test: "Strict ordering → ?" "Fan-out → ?" "Migrate RabbitMQ → ?" "Cheapest DR → ?" "Lift-and-shift servers → ?" "Centralized backup → ?" Start reading SecurityIdentity.md.
**Break (10 min)**
**Block 2 (50 min):** Finish SecurityIdentity.md — focus on: IAM evaluation logic (Deny > Allow > Default Deny), Permission Boundaries, KMS key types + Envelope Encryption, Secrets Manager vs Parameter Store table, Cognito User Pools vs Identity Pools table, WAF rate limiting, GuardDuty vs Inspector vs Macie table. Do all 5 security scenarios. **Write from memory:** IAM eval logic. Cognito User Pools vs Identity Pools. GuardDuty vs Inspector vs Macie.

---

### ☁ Wed Apr 8: REST DAY

Second rest day. 12 chapters absorbed. Brain needs to consolidate. Go outside.

---

### Day 13 — Thu Apr 9: Monitoring & Audit + Additional Services

**Block 1 (50 min):** Networking + Storage + HA cheat sheets (12+ days old — if you can't recall these, they need serious reinforcement on Day 14). Start reading MonitoringAudit.md.
**Break (10 min)**
**Block 2 (50 min):** Finish MonitoringAudit.md — focus on: CloudWatch vs CloudTrail vs Config table (WILL be on exam), CloudWatch Agent for memory/disk (not default metrics), EventBridge = CloudWatch Events 2.0, SSM Session Manager (no SSH/port 22), SSM Patch Manager, Trusted Advisor 5 pillars. Then skim AdditionalServices.md (15 min) — Athena, Redshift, Glue, CloudFormation, Elastic Beanstalk, OpenSearch, AppSync, MSK. Do all 5 monitoring scenarios. **Write from memory:** CloudWatch vs CloudTrail vs Config. "Query S3 with SQL → ?" "Data warehouse → ?" "ETL → ?" "IaC → ?"

---

### Day 14 — Fri Apr 10: Full Review Day + Buffer

**Block 1 (50 min):** ALL 14 chapter cheat sheets — rapid fire. Cover the answer, test yourself. Mark anything you can't recall instantly.
**Break (10 min)**
**Block 2 (50 min):** Continue cheat sheets. Write your **weak list** on paper — every trigger/answer you missed. You'll use this for the rest of the plan.

This day also serves as a **buffer** — if you fell behind by 1 day during Phase 1, use today to catch up on the last chapter instead, and do the full review as Block 1 on Day 15.

**Also today:** Buy mock exams if you haven't:
- Stephane Maarek SAA-C03 practice exams (Udemy)
- Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy)

**Book your exam today for April 23.**

---

## Phase 2: Mock Exams + Reinforcement (Days 15-25)

**Mock exam days = 2 hours:**
```
Block 1 (60 min): Take mock exam under real conditions
Break (10 min): Walk, stretch, decompress
Block 2 (50 min): Review EVERY wrong answer — go back to chapter for each topic
```

**Review days = 2 hours:**
```
Block 1 (50 min): Reread weak chapter sections + redo scenarios
Break (10 min)
Block 2 (50 min): Speed-run cheat sheets + drill weak list
```

**Pattern: Mock → Review → Mock → Review.** Never more than 2 consecutive mock days. Review days fix gaps before the next mock exposes them again.

---

### Day 15 — Sat Apr 11: Mock #1 (Maarek) [2 hours]
**Block 1 (60 min):** Stephane Maarek Exam #1 under real conditions.
**Break (10 min)**
**Block 2 (50 min):** Review every wrong answer + every flagged answer. Go back to the chapter for each wrong topic.
**Expected score:** 55-70%. This is your baseline. Don't panic.
**After:** Update your weak list.

---

### Day 16 — Sun Apr 12: Review Day [2 hours]
**Block 1 (50 min):** Focus ONLY on topics you got wrong on Mock #1. Reread the relevant chapter sections. Redo the scenarios for those chapters.
**Break (10 min)**
**Block 2 (50 min):** Rewrite the cheat sheet items from memory. Speed-run Chapters 1-7 cheat sheets (these are now 10-16 days old).

---

### Day 17 — Mon Apr 13: Mock #2 (Davis) [2 hours]
**Block 1 (60 min):** Neal Davis / Tutorials Dojo Exam #1.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Expected score:** 50-65% (Davis is intentionally harder — this is normal).
**After:** Notice patterns — same topics wrong in Mock #1 AND #2 = blind spots. Highlight on weak list.

---

### Day 18 — Tue Apr 14: Review Day [2 hours]
**Block 1 (50 min):** Blind spot topics from Mock #1 + #2 — reread those full chapter sections.
**Break (10 min)**
**Block 2 (50 min):** Speed-run Chapters 8-14 cheat sheets. Redo your 5 weakest scenarios from any chapter.

---

### ☁ Wed Apr 15: REST DAY

Third rest day. You've done 2 mocks. Let it consolidate. Trust the process.

---

### Day 19 — Thu Apr 16: Mock #3 (Maarek) [2 hours]
**Block 1 (60 min):** Stephane Maarek Exam #2.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Target score:** 65-78%. You should see improvement from Mock #1.
**After:** If same topics are STILL wrong, it's a comprehension gap not memory. Reread that chapter more carefully tonight.

---

### Day 20 — Fri Apr 17: Mock #4 (Davis) [2 hours]
**Block 1 (60 min):** Neal Davis Exam #2.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Target score:** 62-74%.
**After:** If DR patterns, cost tools, or migration questions keep tripping you, reread CostOptimization.md tonight.

---

### Day 21 — Sat Apr 18: Review Day [2 hours]
**Block 1 (50 min):** All 14 cheat sheets — rapid fire trigger/answer pairs.
**Break (10 min)**
**Block 2 (50 min):** Your weak list — drill until you can recall every item. Focus on any topic wrong in 3+ mocks.

---

### Day 22 — Sun Apr 19: Mock #5 (Maarek) — CHECKPOINT [2 hours]
**Block 1 (60 min):** Stephane Maarek Exam #3.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Target score:** 72-82%.

**Decision point:**

| Score | Action |
|-------|--------|
| 78%+ | Keep April 23. You're ready. |
| 72-77% | Keep April 23. Focus remaining days on weak list only. |
| 65-71% | Reschedule to April 30. Take 2 more mocks + review. |
| <65% | Reschedule to May 7. Reread your 3 weakest chapters fully. |

---

### Day 23 — Mon Apr 20: Mock #6 (Maarek) [2 hours]
**Block 1 (60 min):** Stephane Maarek Exam #4.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Target score:** 78-85%.

---

### Day 24 — Tue Apr 21: Mock #7 (Maarek) — GO/NO-GO [2 hours]
**Block 1 (60 min):** Stephane Maarek Exam #5.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Target score:** 80%+.

If 80%+: You're passing. Cruise to exam day.
If 75-79%: You'll likely pass. Review weak list, don't cram.
If <75%: Push exam back 1 week. Take remaining Davis exams.

---

### ☁ Wed Apr 22: REST DAY

Final rest day. Do NOT study. Your brain needs this before the exam tomorrow. Walk, relax, sleep early.

---

### Day 25 — Thu Apr 23: EXAM DAY

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

Every learning day, Block 1 reviews older material before Block 2 introduces new content. The 10-min break between them lets your brain consolidate the review before new input.

```
Day  1: Learn Networking                    (no review — first day)
Day  2: Learn Storage                       Review → Networking (+1 day)
Day  3: Learn Compute                       Review → Networking, Storage (+1/+2 days)
Day  4: Learn HA                            Review → Storage, Compute + Networking scenario (+2/+3 days)
Day  5: Learn DB Relational                 Review → Networking, Storage (+3/+4 days)
Day  6: Learn DB NoSQL                      Review → HA, Compute (+2/+3 days)
       ☁ REST DAY                           Brain consolidates Days 1-6
Day  7: Learn Serverless                    Review → DB Relational, DB NoSQL (+1/+2 days)
Day  8: Learn Containers                    Review → Networking, Storage (+8 days — critical)
Day  9: Learn Cost/Migration/DR             Review → Compute, HA (+5/+6 days)
Day 10: Learn Decoupling                    Review → Serverless, Containers (+3/+2 days)
Day 11: Learn CDN & DNS                     Review → DB Relational, DB NoSQL (+5/+6 days)
Day 12: Learn Security                      Review → Decoupling, Cost/Migration (+2/+3 days)
       ☁ REST DAY                           Brain consolidates Days 7-12
Day 13: Learn Monitoring + Additional       Review → Networking, Storage, HA (+12 days)
Day 14: Full review                         Review → ALL 14 cheat sheets
Day 15+: Mock exams                         Every mock reviews EVERYTHING at once
       ☁ REST DAY after every 6 study days
```

**Result:** By exam day, Day 1 content has been reviewed 10+ times across:
- 5 dedicated Block 1 reviews
- 1 full review day
- 7 mock exams that test all topics
- 4 rest days for memory consolidation

---

## When to Book the Exam

**Book it on Day 14 (April 10) for April 23.**

- Having a locked date creates healthy pressure
- You can reschedule up to 24 hours before (AWS policy)
- Day 22 checkpoint tells you if you need to push back

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
WEEK 1 — LEARN (Mar 26 - Apr 1)                    [2h/day, 50+10+50]
  Thu 26: Networking
  Fri 27: Storage
  Sat 28: Compute
  Sun 29: High Availability
  Mon 30: Databases Relational
  Tue 31: Databases NoSQL
  Wed  1: ☁ REST DAY

WEEK 2 — LEARN + BUFFER (Apr 2 - Apr 8)            [2h/day, 50+10+50]
  Thu  2: Serverless
  Fri  3: Containers
  Sat  4: Cost/Migration/DR
  Sun  5: Decoupling
  Mon  6: Content Delivery & DNS
  Tue  7: Security & Identity
  Wed  8: ☁ REST DAY

WEEK 3 — FINAL LEARN + MOCKS (Apr 9 - Apr 15)      [2h/day]
  Thu  9: Monitoring & Audit + Additional Services
  Fri 10: Full review + buffer (book exam today)
  Sat 11: Mock #1 (Maarek)
  Sun 12: Review day
  Mon 13: Mock #2 (Davis)
  Tue 14: Review day
  Wed 15: ☁ REST DAY

WEEK 4 — MOCKS + FINAL (Apr 16 - Apr 23)
  Thu 16: Mock #3 (Maarek)
  Fri 17: Mock #4 (Davis)
  Sat 18: Review day
  Sun 19: Mock #5 (Maarek) ← CHECKPOINT
  Mon 20: Mock #6 (Maarek)
  Tue 21: Mock #7 (Maarek) ← GO/NO-GO
  Wed 22: ☁ REST DAY (pre-exam)
  Thu 23: EXAM DAY
```

---

## If You Miss a Day

**Missed 1 day during Phase 1:** Use the next rest day to catch up. If the rest day is too far away, do a 2.5h session the next day (three 50-min blocks with two 10-min breaks).

**Missed 1 day during Phase 2:** Skip the review day, not the mock. The mock is more valuable.

**Missed 2+ days:** Shift everything forward. Reschedule exam if needed.

**Golden rule:** Never skip Block 1 review to cram new content. Old material fading is worse than being 1 day behind. And never skip rest days — they're not wasted time, they're when your brain builds long-term memory.

---

## What You Need Before Day 1

- [ ] All 14 chapter files (already in this folder)
- [ ] Stephane Maarek SAA-C03 practice exams (Udemy) — buy by Day 12
- [ ] Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy) — buy by Day 12
- [ ] A notebook for writing from memory (paper > app for retention)
- [ ] A timer (set 50-min blocks — phone timer works)
- [ ] AWS Certification account (aws.amazon.com/certification)
