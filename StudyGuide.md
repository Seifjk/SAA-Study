# SAA-C03 Study Guide

**Start Date:** Tuesday, March 25, 2026
**Pace:** 3 hours/day (three 50-min blocks with 10-min breaks)
**Rest Days:** 1 (mid-learning consolidation)
**Exam Date:** Book for April 11, 2026
**Total Duration:** 18 calendar days (17 study days + 1 rest day)

---

## How This Plan Works

**The science behind this structure:**
- Sustained attention peaks at ~50 min then drops sharply. Three 50-min blocks keeps you in the high-retention zone the entire session.
- The 10-min breaks between blocks are critical — quiet rest after learning boosts memory consolidation by 15-40%.
- 3 hours/day is aggressive but sustainable for a focused sprint. The third block gives you room to pair lighter chapters or go deeper on heavy ones.

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
4. **Mock exams** — starting Day 11, 7 mocks force full recall of all 14 chapters (455 practice questions)

---

## Content Map

| Day | Chapter(s) | File(s) | Load |
|-----|-----------|---------|------|
| 1 | Networking | [Networking](./Networking.md) | Medium |
| 2 | Storage | [Storage](./Storage.md) | Medium |
| 3 | Compute + High Availability | [Compute](./Compute.md) + [HighAvailability](./HighAvailability.md) | Paired |
| 4 | Databases Relational | [DatabasesRelational](./DatabasesRelational.md) | Heavy |
| 5 | Databases NoSQL + Decoupling | [DatabasesNoSQL](./DatabasesNoSQL.md) + [Decoupling](./Decoupling.md) | Paired |
| — | **REST DAY** | — | — |
| 6 | Serverless | [Serverless](./Serverless.md) | Heavy |
| 7 | Cost/Migration/DR + CDN/DNS | [CostOptimization](./CostOptimization.md) + [ContentDeliveryDNS](./ContentDeliveryDNS.md) | Paired |
| 8 | Security & Identity | [SecurityIdentity](./SecurityIdentity.md) | Heavy |
| 9 | Monitoring/Audit + Containers + Additional Services | [MonitoringAudit](./MonitoringAudit.md) + [Containers](./Containers.md) + [AdditionalServices](./AdditionalServices.md) | Triple |
| 10 | Full review + buffer | All cheat sheets | Review |

---

## Phase 1: Learn (Days 1-10)

---

### Day 1 — Tue Mar 25: Networking

**Block 1 (50 min):** First day — no review. Start reading Networking.md. Focus on: SG vs NACL table, VPC Endpoints (Gateway = S3/DynamoDB free, Interface = everything else paid), NAT GW must be in public subnet, Transit Gateway = hub-and-spoke.
**Break (10 min):** Walk away from screen. Let it settle.
**Block 2 (50 min):** Continue Networking.md. Focus on: VPC Peering (no transitive), PrivateLink, Site-to-Site VPN vs Direct Connect table, Route Table propagation, Bastion Host vs Session Manager.
**Break (10 min)**
**Block 3 (50 min):** Do all 5 networking scenarios — cover the answer, explain why the other 3 are wrong. **Write from memory:** SG vs NACL table. VPC Endpoint types. Direct Connect vs VPN. Cheat sheet triggers.

---

### Day 2 — Wed Mar 26: Storage

**Block 1 (50 min):** Close Networking.md. From memory write: "Block specific IP → ?" "Access S3 privately free → ?" "Connect 50 VPCs → ?" "Instance has public IP but no internet → ?" Check against cheat sheet. Redo any you missed. Then start reading Storage.md.
**Break (10 min)**
**Block 2 (50 min):** Continue Storage.md — focus on: EBS type triggers (gp3 default, io1/io2 for IOPS, st1 for throughput), S3 storage classes waterfall, S3 Express One Zone, Multi-Attach = io1/io2 only, S3 Object Lock, S3 Replication (CRR vs SRR).
**Break (10 min)**
**Block 3 (50 min):** Finish Storage.md — EFS vs FSx decision, Transfer Family, Snow Family. Do all 5 storage scenarios. **Write from memory:** S3 storage classes in order. EBS type selection triggers. EFS vs FSx vs S3 decision tree.

---

### Day 3 — Thu Mar 27: Compute + High Availability

**Paired day — both are medium-weight and deeply connected (compute scales via HA).**

**Block 1 (50 min):** Networking + Storage cheat sheets. Cover answers, test every trigger line. Mark any you miss — these go on your weak list. Start reading Compute.md.
**Break (10 min)**
**Block 2 (50 min):** Finish Compute.md — focus on: purchasing options table (On-Demand/RI/Spot/Dedicated Host vs Instance), Placement Groups (Cluster/Spread/Partition), ENI vs ENA vs EFA, Hibernate limits. Do all 5 compute scenarios. Then start HighAvailability.md — focus on: ALB vs NLB vs GWLB comparison table, NLB now supports Security Groups.
**Break (10 min)**
**Block 3 (50 min):** Finish HighAvailability.md — all 5 scaling policy types (Target Tracking/Step/Simple/Scheduled/Predictive), Connection Draining, Warm Pool, Cross-Zone Load Balancing, Sticky Sessions. Do all 5 HA scenarios. **Write from memory:** Purchasing options table. ALB vs NLB vs GWLB table. 5 scaling policies with triggers.

---

### Day 4 — Fri Mar 28: Databases Relational

**Block 1 (50 min):** Networking + Storage cheat sheets (rapid fire triggers — now 3+ days old). Compute + HA triggers: "Per-socket licensing → ?" "Static IP load balancer → ?" "GENEVE protocol → ?" "Maintain CPU at 60% → ?" Start reading DatabasesRelational.md.
**Break (10 min)**
**Block 2 (50 min):** Continue DatabasesRelational.md — focus on: Multi-AZ vs Read Replicas table (WILL be on exam), Aurora Serverless v2, Aurora Global Database (<1 sec replication, promote <1 min), RDS Proxy for Lambda, Backtrack (in-place rewind, seconds, 72h max).
**Break (10 min)**
**Block 3 (50 min):** Finish DatabasesRelational.md. Do all 5 relational DB scenarios. **Write from memory:** Multi-AZ vs Read Replicas table. Aurora decision tree. RDS engine list. "Read-heavy workload → ?" "Global DB <1 sec → ?" "Lambda + RDS → ?"

---

### Day 5 — Sat Mar 29: Databases NoSQL + Decoupling

**Paired day — NoSQL is heavy, Decoupling is medium. Both deal with high-throughput distributed patterns.**

**Block 1 (50 min):** HA + Compute cheat sheets. DB Relational triggers: "Multi-AZ failover → ?" "Read replicas cross-region → ?" "Serverless variable DB → ?" "Lambda connection storm → ?" Start reading DatabasesNoSQL.md.
**Break (10 min)**
**Block 2 (50 min):** Finish DatabasesNoSQL.md — focus on: Provisioned vs On-Demand, GSI vs LSI table, DynamoDB Streams + Lambda, DAX (microseconds) vs ElastiCache, Redis vs Memcached table, Global Tables. Do all 5 NoSQL scenarios. Then start Decoupling.md — focus on: SQS Standard vs FIFO table, Long Polling, Visibility Timeout, DLQ.
**Break (10 min)**
**Block 3 (50 min):** Finish Decoupling.md — SNS + SQS Fan-Out pattern, Kinesis Data Streams vs Data Firehose table, Amazon MQ = RabbitMQ migration only. Do all 5 decoupling scenarios. **Write from memory:** GSI vs LSI table. Redis vs Memcached. SQS Standard vs FIFO. Kinesis Streams vs Firehose. RCU calc: 10 items/sec, 8KB, strongly consistent = 20 RCU.

---

### ☁ Sun Mar 30: REST DAY

No studying. Walk, exercise, do anything else. Your brain is consolidating 5 days of heavy material right now. This is not wasted time — it's when long-term memories form.

---

### Day 6 — Mon Mar 31: Serverless

**Block 1 (50 min):** DB Relational + DB NoSQL + Decoupling cheat sheets (hardest chapters — quiz every line). Redo "GSI Throttling" scenario from memory. Redo "Fan-Out" scenario from memory.
**Break (10 min)**
**Block 2 (50 min):** Read Serverless.md — focus on: Lambda limits table (MEMORIZE: 15 min timeout, 10GB mem, 1MB async payload, 10GB /tmp, 1000 concurrent), invocation types, Provisioned Concurrency = kill cold starts, API Gateway throttling/caching.
**Break (10 min)**
**Block 3 (50 min):** Finish Serverless.md — Step Functions Standard (1 year) vs Express (5 min), SQS + Lambda, S3 + Lambda, EventBridge integration. Do all 5 serverless scenarios. **Write from memory:** Lambda limits table. Step Functions Standard vs Express. API Gateway cache/throttle.

---

### Day 7 — Tue Apr 1: Cost/Migration/DR + CDN/DNS

**Paired day — Cost/Migration is heavy on memorization, CDN/DNS is pattern-based. Clean boundary.**

**Block 1 (50 min):** Networking + Storage cheat sheets (Day 1-2, now 7+ days old — critical reinforcement window). If you can't recall 80%+ instantly, flag for extra review on Day 10. Start reading CostOptimization.md.
**Break (10 min)**
**Block 2 (50 min):** Finish CostOptimization.md — Cost Explorer vs Budgets vs Compute Optimizer, DMS (+SCT for heterogeneous), DataSync vs Snow Family, DR patterns table (RPO/RTO — WILL be on exam), AWS Organizations SCPs, AWS Backup. Do all 5 cost scenarios. Then start ContentDeliveryDNS.md — focus on: All 7 Route 53 routing policies table (WILL be on exam), Alias vs CNAME (root domain → Alias only).
**Break (10 min)**
**Block 3 (50 min):** Finish ContentDeliveryDNS.md — CloudFront OAC (not OAI), Signed URLs (single file) vs Signed Cookies (multiple files), CloudFront vs Global Accelerator table. Do all 5 CDN/DNS scenarios. **Write from memory:** DR patterns table (4 strategies, RPO/RTO/Cost). All 7 routing policies. "Lift-and-shift servers → ?" "Cheapest DR → ?" "Root domain → ?"

---

### Day 8 — Wed Apr 2: Security & Identity

**Block 1 (50 min):** Serverless + Decoupling cheat sheets. Cost/Migration + CDN/DNS triggers: "Cheapest DR → ?" "Fan-out → ?" "Strict ordering → ?" "Migrate RabbitMQ → ?" "Monitor spending → ?" Start reading SecurityIdentity.md.
**Break (10 min)**
**Block 2 (50 min):** Continue SecurityIdentity.md — focus on: IAM evaluation logic (Deny > Allow > Default Deny), Permission Boundaries, KMS key types + Envelope Encryption, Secrets Manager vs Parameter Store table, Cognito User Pools vs Identity Pools table.
**Break (10 min)**
**Block 3 (50 min):** Finish SecurityIdentity.md — WAF rate limiting, GuardDuty vs Inspector vs Macie table, Shield Standard vs Advanced, ACM, CloudHSM. Do all 5 security scenarios. **Write from memory:** IAM eval logic. Cognito User Pools vs Identity Pools. GuardDuty vs Inspector vs Macie. Secrets Manager vs Parameter Store.

---

### Day 9 — Thu Apr 3: Monitoring/Audit + Containers + Additional Services

**Triple day — all three are lighter. Monitoring is medium, Containers is medium, Additional is a skim.**

**Block 1 (50 min):** Networking + Storage + HA cheat sheets (9+ days old — if you can't recall these, flag for heavy review on Day 10). Then read MonitoringAudit.md — focus on: CloudWatch vs CloudTrail vs Config table (WILL be on exam), CloudWatch Agent for memory/disk (not default metrics), EventBridge = CloudWatch Events 2.0.
**Break (10 min)**
**Block 2 (50 min):** Finish MonitoringAudit.md — SSM Session Manager (no SSH/port 22), SSM Patch Manager, Trusted Advisor 5 pillars. Do top 3 monitoring scenarios. Then read Containers.md — focus on: ECS EC2 vs Fargate table, Task Role vs Execution Role, awsvpc mode, EFS for persistent shared storage, EKS = Kubernetes migration, Capacity Provider + ASG.
**Break (10 min)**
**Block 3 (50 min):** Finish Containers scenarios. Then skim AdditionalServices.md (20 min) — Athena (SQL on S3), Redshift (data warehouse), Glue (ETL), CloudFormation (IaC), Elastic Beanstalk (PaaS), OpenSearch (search/log analytics), AppSync (GraphQL), MSK (Kafka migration). **Write from memory:** CloudWatch vs CloudTrail vs Config. ECS EC2 vs Fargate. "Query S3 with SQL → ?" "Data warehouse → ?" "ETL → ?" "Monitor memory → ?" "Who deleted resource → ?"

---

### Day 10 — Fri Apr 4: Full Review Day + Buffer

**Block 1 (50 min):** ALL 14 chapter cheat sheets — rapid fire. Cover the answer, test yourself. Mark anything you can't recall instantly.
**Break (10 min)**
**Block 2 (50 min):** Continue cheat sheets. Write your **weak list** on paper — every trigger/answer you missed.
**Break (10 min)**
**Block 3 (50 min):** Redo the 5 hardest scenarios across all chapters (whichever ones tripped you up). Reread weak chapter sections.

This day also serves as a **buffer** — if you fell behind by 1 day during Phase 1, use today to catch up on the last chapter instead, and do the full review as Block 1 on Day 11.

**Also today:** Buy mock exams if you haven't:
- Stephane Maarek SAA-C03 practice exams (Udemy) — you need 4 exams from this set
- Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy) — you need 3 exams from this set

---

## Phase 2: Mock Exams + Reinforcement (Days 11-17)

**Every day is a mock exam day = 3 hours:**
```
Block 1 (65 min): Take mock exam under real conditions
Break (10 min): Walk, stretch, decompress
Block 2 (50 min): Review EVERY wrong answer — go back to chapter for each topic
Break (10 min)
Block 3 (45 min): Reread weak chapter sections + drill weak list
```

**7 mocks in 7 days. No dedicated review days needed** — Block 2+3 of each mock day gives you 95 minutes of targeted review, which is more effective than generic cheat sheet review because it's focused on exactly what you got wrong.

**Pattern: Mock every day.** Alternating Maarek (easier, closer to real exam) and Davis (harder, builds resilience).

---

### Day 11 — Sat Apr 5: Mock #1 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #1 under real conditions.
**Break (10 min)**
**Block 2 (50 min):** Review every wrong answer + every flagged answer. Go back to the chapter for each wrong topic.
**Break (10 min)**
**Block 3 (45 min):** Reread the weakest 2-3 chapter sections. Start your weak list.
**Expected score:** 55-70%. This is your baseline. Don't panic.

---

### Day 12 — Sun Apr 6: Mock #2 (Davis) [3 hours]
**Block 1 (65 min):** Neal Davis / Tutorials Dojo Exam #1.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers. Notice patterns — same topics wrong in Mock #1 AND #2 = blind spots.
**Break (10 min)**
**Block 3 (45 min):** Highlight blind spots on weak list. Reread those chapter sections. Redo the scenarios for blind spot chapters.
**Expected score:** 50-65% (Davis is intentionally harder — this is normal).

---

### Day 13 — Mon Apr 7: Mock #3 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #2.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Speed-run all cheat sheets. Focus extra time on blind spot chapters from Mock #1+2.
**Target score:** 65-75%. You should see improvement from Mock #1.

---

### Day 14 — Tue Apr 8: Mock #4 (Davis) — CHECKPOINT [3 hours]
**Block 1 (65 min):** Neal Davis / Tutorials Dojo Exam #2.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Deep-dive into persistent weak topics. If DR patterns, cost tools, or migration questions keep tripping you, reread CostOptimization.md fully.
**Target score:** 60-72% (Davis scale — equivalent to 70-78% on real exam).

**Decision point:**

| Davis Score | Real Exam Equivalent | Action |
|-------------|---------------------|--------|
| 68%+ | ~75%+ | Keep April 11. You're on track. |
| 60-67% | ~68-74% | Keep April 11. Remaining days focus exclusively on weak list. |
| 52-59% | ~60-67% | Reschedule to April 18. Take 3 more mocks + review. |
| <52% | ~<60% | Reschedule to April 25. Reread your 3 weakest chapters fully. |

---

### Day 15 — Wed Apr 9: Mock #5 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #3.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Drill weak list. Redo your 5 weakest scenarios from any chapter.
**Target score:** 75-85%.

---

### Day 16 — Thu Apr 10: Mock #6 (Maarek) [3 hours]
**Block 1 (65 min):** Stephane Maarek Exam #4.
**Break (10 min)**
**Block 2 (50 min):** Review all wrong answers.
**Break (10 min)**
**Block 3 (45 min):** Speed-run all cheat sheets. Write from memory your top 10 weakest triggers.
**Target score:** 78-88%.

---

### Day 17 — Fri Apr 11 morning: Mock #7 (Davis) — GO/NO-GO [2 hours]
**Block 1 (65 min):** Neal Davis Exam #3. Real conditions. Time it.
**Break (10 min)**
**Block 2 (45 min):** Quick review of wrong answers. Skim weak list one final time. Done.

**This is your morning session — exam is in the afternoon.**

If 68%+ on Davis: You're passing. Relax until exam time.
If 60-67% on Davis: You'll likely pass. Skim your 3 weakest cheat sheets, then stop.
If <60% on Davis: Push exam back 1 week. You have 3 more Maarek exams to use.

---

### Fri Apr 11 afternoon: EXAM DAY

**After Mock #7 review — rest protocol:**
- Lunch. Walk. Decompress.
- Do NOT re-study. You just did 7 mocks. You're ready.

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
Day  5: Learn DB NoSQL + Decoupling                Review → HA, Compute (+2/+3 days)
       ☁ REST DAY                                  Brain consolidates Days 1-5
Day  6: Learn Serverless                           Review → DB Relational, DB NoSQL, Decoupling (+1/+2 days)
Day  7: Learn Cost/DR + CDN/DNS                    Review → Networking, Storage (+7 days — critical)
Day  8: Learn Security                             Review → Serverless, Decoupling (+2/+3 days)
Day  9: Learn Monitoring + Containers + Additional Review → Networking, Storage, HA (+9 days — critical)
Day 10: Full review                                Review → ALL 14 cheat sheets
Day 11+: Mock exams                                Every mock reviews EVERYTHING at once
```

**Result:** By exam day, Day 1 content has been reviewed 10+ times across:
- 4 dedicated Block 1 reviews
- 1 full review day
- 7 mock exams that test all topics (455 practice questions)
- 1 rest day for memory consolidation

---

## When to Book the Exam

**Book it NOW for Friday April 11, afternoon slot.**

- Exam is already booked — no backing out, only forward
- You can reschedule up to 24 hours before (AWS policy)
- Day 14 checkpoint tells you if you need to push back

---

## Mock Exam Score Tracker

```
Mock #1 (Day 11) Maarek #1:  ___% | Weak: _______________
Mock #2 (Day 12) Davis  #1:  ___% | Weak: _______________
Mock #3 (Day 13) Maarek #2:  ___% | Weak: _______________
Mock #4 (Day 14) Davis  #2:  ___% | Weak: _______________ ← CHECKPOINT
Mock #5 (Day 15) Maarek #3:  ___% | Weak: _______________
Mock #6 (Day 16) Maarek #4:  ___% | Weak: _______________
Mock #7 (Day 17) Davis  #3:  ___% | Weak: _______________ ← GO/NO-GO (exam day morning)
```

**Healthy:** Each Maarek score 5-10% higher than the previous. Davis scores ~8% lower than Maarek is normal.
**Warning:** Plateau or drop for 2 consecutive → reread your 2 weakest chapters entirely.

---

## Calendar View

```
WEEK 1 — LEARN (Mar 25 - Mar 30)                    [3h/day, 50+10+50+10+50]
  Tue 25: Networking
  Wed 26: Storage
  Thu 27: Compute + High Availability
  Fri 28: Databases Relational
  Sat 29: Databases NoSQL + Decoupling
  Sun 30: ☁ REST DAY

WEEK 2 — LEARN + REVIEW (Mar 31 - Apr 4)            [3h/day, 50+10+50+10+50]
  Mon 31: Serverless
  Tue  1: Cost/Migration/DR + CDN/DNS
  Wed  2: Security & Identity
  Thu  3: Monitoring + Containers + Additional
  Fri  4: Full review + buffer (book exam today)

WEEK 3 — MOCKS (Apr 5 - Apr 11)                     [3h/day, 65+10+50+10+45]
  Sat  5: Mock #1 (Maarek)
  Sun  6: Mock #2 (Davis)
  Mon  7: Mock #3 (Maarek)
  Tue  8: Mock #4 (Davis) ← CHECKPOINT
  Wed  9: Mock #5 (Maarek)
  Thu 10: Mock #6 (Maarek)
  Fri 11: Mock #7 (Davis, morning) → ☁ REST → EXAM (afternoon)
```

---

## If You Miss a Day

**Missed 1 day during Phase 1:** Use the rest day (Mar 30) to catch up. If already past the rest day, the third block gives you more flexibility — extend to a 4th block (3.5h) the next day.

**Missed 1 day during Phase 2:** You have 7 mocks scheduled. Dropping to 6 is fine — skip the last Maarek (Day 16), keep the Davis go/no-go on exam morning.

**Missed 2+ days:** Shift everything forward. Reschedule exam if needed.

**Golden rule:** Never skip Block 1 review to cram new content. Old material fading is worse than being 1 day behind. And never skip rest days — they're when your brain builds long-term memory.

---

## What You Need Before Day 1

- [ ] All 14 chapter files (already in this folder)
- [ ] Stephane Maarek SAA-C03 practice exams (Udemy) — buy by Day 9
- [ ] Neal Davis / Tutorials Dojo SAA-C03 practice exams (Udemy) — buy by Day 9
- [ ] A notebook for writing from memory (paper > app for retention)
- [ ] A timer (set 50-min blocks — phone timer works)
- [ ] AWS Certification account (aws.amazon.com/certification)
