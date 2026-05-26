# Maarek Mock #1 — Diagnostic Analysis (May 25, 2026) — v2 (Audited)

**Score: 50%** · 32 wrong questions captured in `questions.txt`
**Content covered going in:** Networking, Compute, HA, Storage, DB Relational, DB NoSQL, Decoupling (7 of 14 chapters)
**Not yet covered:** Serverless, Containers, Cost/Migration/DR, CDN/DNS (Route 53 + Global Accelerator + CloudFront), Security & Identity, Monitoring & Audit, Additional Services

> **Note on this version:** v1 of this report was sloppy — I assumed "this chapter exists in your plan" = "this fact is in your notes." It isn't. v2 audits **every** wrong question against the actual chapter text. The "covered" bucket only contains questions where the **exact fact you needed** is on the page. Anything else moved to **partial** or **not covered**.

---

## 1. Honest bucketing (re-audited against actual chapter contents)

For each missed question, I went to the studied chapters and grepped for the fact the question hinges on. The bucket reflects whether **that specific fact** exists in your notes — not just whether the topic's chapter has been opened.

| Q# | The actual fact the question tested | Verdict | Where it's covered (or not) |
|----|-------------------------------------|---------|-----------------------------|
| 1 | "IoT ingest, lost data, need replay → SQS (not SNS)" | 🟧 COVERED | Decoupling — "shock absorber" model, line 25+ |
| 2 | User data runs as **root** + first boot only | 🟨 PARTIAL | Compute.md:166 covers "first boot" ✅, but **root privileges is NOT in the chapter**. You correctly picked first-boot; the root half was untaught. |
| 6 | ASG default termination → oldest **Launch Configuration** | 🟧 COVERED | HA.md:198 explicitly says "oldest Launch Configuration (or oldest Launch Template, if configured)" |
| 7 | EFS **Max I/O** for big-data throughput | 🟧 COVERED | Storage.md:174 — "Max I/O (high throughput, higher latency, Big Data)" |
| 8 | "Shared Services VPC" for hub-and-spoke with TGW | 🟥 NOT COVERED | Networking covers TGW + RAM Shared VPC, but the specific **"Shared Services VPC pattern"** as a named answer is not in your notes. |
| 9 | S3 IAM policy: `ListBucket` on bucket ARN + `GetObject` on `/*` (two different resources) | 🟥 NOT COVERED | Storage chapter doesn't teach the bucket-vs-object ARN pattern; that lives in Security & Identity (not yet studied). |
| 10 | FSx for Windows = DFSR replacement | 🟧 COVERED | Storage.md:179 — FSx for Windows + SMB + AD |
| 11 | Cognito **User Pool** on ALB (authentication) | 🟧 COVERED | HA.md:53 — "ALB natively supports OIDC + Cognito" + decision tree line 149 + cheat sheet #16 |
| 16 | ASG won't terminate: grace period + **impaired status** | 🟨 PARTIAL | HA.md:193 covers grace period ✅. **"Impaired status" as a non-termination reason is NOT in your notes.** |
| 20 | ALB + WAF + multi-AZ HA + minimal complexity | 🟨 PARTIAL | HA covers ALB+ASG multi-AZ ✅. WAF integration with ALB is in Security (not yet studied). |
| 21 | AWS Config managed rule: imported vs ACM-issued cert | 🟥 NOT COVERED | Security & Identity + Monitoring — not yet studied. |
| 23 | DataSync over Direct Connect via **PrivateLink interface endpoint for EFS** (private VIF) | 🟨 PARTIAL | DataSync exists in Storage:218 ✅. The specific **DX + private VIF + PrivateLink for EFS** path is not. Full pattern lives in Cost/Migration (not yet studied). |
| 25 | Dedicated **Instances** = cheapest single-tenant | 🟧 COVERED | Compute.md:44 — table explicitly says Dedicated Instances are "less expensive" than Dedicated Hosts |
| 26 | S3 **Access Points** for per-prefix isolation | 🟧 COVERED | Storage.md:234-237 |
| 27 | EFS **Standard-IA storage class** | 🟥 NOT COVERED | Storage chapter covers EFS performance modes (General Purpose / Max I/O) and throughput modes (Elastic / Provisioned / Bursting) — but **EFS storage classes (Standard / Standard-IA / One Zone-IA) are not in your notes**. |
| 28 | RDS Multi-AZ engine upgrade → **both upgraded simultaneously, downtime** | 🟥 NOT COVERED | DB Relational covers Multi-AZ failover for instance failure (60-120s) ✅, but **engine-upgrade behavior on Multi-AZ is not specifically taught**. |
| 29 | IAM policy logic: Deny region + Allow IP combined | 🟥 NOT COVERED | IAM evaluation depth is in Security & Identity (not yet studied). |
| 30 | Aurora dev DB → restore from automated backup (heavy read load problem) | 🟨 PARTIAL | DB Relational §6 teaches **Aurora Cloning** (copy-on-write, seconds, no perf impact on source) ✅ — this is actually a *better* answer for the same use case. The exam's accepted answer ("restore from automated backups") is a different valid approach also covered briefly. You picked Read Replica, which the chapter is *very* clear is wrong for cloning use cases. |
| 37 | Aurora **Serverless v2** for auto-scale + HA | 🟧 COVERED | DB Relational §3 + cheat sheet #9 |
| 38 | Latency routing + cross-region Aurora replica | 🟨 PARTIAL | Cross-region Aurora replicas covered ✅ (DB Relational table). Route 53 **latency routing policy** is in CDN/DNS (not yet studied). You got 1 of 2 right. |
| 43 | Snowball → S3 → **lifecycle to Glacier Deep Archive** | 🟨 PARTIAL | Storage briefly mentions Snowball Edge (line 212) ✅. The **specific pattern** "Snowball cannot target Glacier directly — must land in S3 first, then lifecycle" is not explicitly taught. |
| 45 | 4 instances always available → 3 AZs × 2 instances (cheapest HA) | 🟥 NOT COVERED | HA chapter says "Multi-AZ = at least 2 AZs" ✅ but the **arithmetic** of "min=4, survive 1 AZ loss, cheapest = 3 AZs × 2 each" is not on the page. This is a reasoning question, not a recall question. |
| 46 | DR pattern: Route 53 failover record + always-on ASG (warm standby) | 🟥 NOT COVERED | DR patterns chapter is Day 9 — not yet studied. |
| 49 | Encrypt existing RDS → **snapshot → copy as encrypted → restore** | 🟧 COVERED | DB Relational:104, :110, cheat sheet #5 — all three locations explicitly teach this |
| 50 | S3 → SQS → Lambda for near-real-time image processing | 🟨 PARTIAL | Storage:137 mentions S3 → SQS/Lambda for processing ✅. **Lambda chapter (Serverless) not yet studied** — so the "S3 event → SQS → Lambda" pattern as the right cost-effective fit is fragmentary. |
| 51 | ALB ↔ EC2 ↔ RDS SG chain (3-tier security group rules) | 🟨 PARTIAL | Networking covers SG basics ✅. The specific 3-tier SG chaining pattern (web SG → app SG → DB SG with correct ports) appears in HA scenarios but not as a memorized rule. |
| 52 | `aws:RequestedRegion` IAM condition | 🟥 NOT COVERED | Security & Identity — not yet studied. |
| 57 | RI for 80% baseline + on-demand/spot above | 🟥 NOT COVERED | Compute purchasing **table** lists each option ✅, but the **mix-and-match strategy** (use RI for the always-on baseline, spot/on-demand for peaks) is not on the page. |
| 59 | Permission Boundary on **users** (not group, not SCP) for privilege escalation | 🟥 NOT COVERED | Security & Identity — not yet studied. |
| 61 | SG outbound: port = **destination's listening port** (1433, not 443) | 🟥 NOT COVERED | Networking covers SG stateful behavior ✅ but does NOT explicitly teach "outbound port = destination port." This is an implicit consequence of how SGs work but never spelled out. |
| 63 | Global Accelerator for blue/green when clients DNS-cache | 🟥 NOT COVERED | CDN/DNS chapter — not yet studied. |
| 64 | Global Accelerator over multi-region NLBs | 🟥 NOT COVERED | CDN/DNS chapter — not yet studied. |

### Honest counts

- 🟧 **COVERED — the exact fact is in your notes:** **8** questions (Q1, Q6, Q7, Q10, Q11, Q25, Q26, Q49, Q37 — wait, that's 9 actually: Q1, Q6, Q7, Q10, Q11, Q25, Q26, Q37, Q49)
- 🟨 **PARTIAL — half the fact is covered, half isn't:** **9** questions (Q2, Q16, Q20, Q23, Q30, Q38, Q43, Q50, Q51)
- 🟥 **NOT COVERED — the fact never appeared in your notes:** **14** questions (Q8, Q9, Q21, Q27, Q28, Q29, Q45, Q46, Q52, Q57, Q59, Q61, Q63, Q64)

**Recount: 9 + 9 + 14 = 32 ✓**

---

## 2. What this really means

> Of 32 wrong answers, only **9 are unambiguously on you** — that's 28% of misses. **23 of 32 (72%)** are facts that either (a) live in chapters you haven't opened, or (b) live in chapters you have opened but aren't actually on the page.

This is a dramatically different picture than v1 painted, and it changes the read on the diagnostic:

- The plan's diagnostic-score table (StudyGuide §6.5) says **50% at this stage = "Normal, continue as-is"** — that band assumed roughly the situation we now have confirmed: most misses are content you haven't been taught yet.
- The 9 real misses are concentrated in a small number of patterns (next section). They are **drillable in minutes**, not chapters.
- The 9 **partial** questions are interesting — they show where your chapters have **gaps that should be patched** (e.g., the chapter mentions a topic but skips the exam-relevant nuance).

---

## 3. The 9 genuine misses — what actually went wrong

Each was on material in your chapters. Each maps to one of four small patterns.

### Pattern A: Tempting distractor with a single disqualifying word

| Q | What you picked | What disqualified it |
|---|---|---|
| Q1 | **SNS** | SNS doesn't buffer — "lost data, need replay" rules it out. SQS is the only one that holds messages. |
| Q11 | **Identity Pools** on CloudFront | The question said *authentication*. Identity Pools = federated AWS credentials. User Pools = sign-in. Wrong half of Cognito, wrong attachment point (ALB not CloudFront). |

**Fix:** When two answers look right, force yourself to name the **one word** that kills the wrong one out loud. 10-second habit, big payoff.

### Pattern B: Tier/mode confusion inside a service you know

| Q | The miss |
|---|---|
| Q7 | EFS **Max I/O** (you picked Bursting Throughput — that's a *throughput* mode, not a *performance* mode; different axis entirely). |
| Q25 | **Dedicated Instances** are cheaper than Dedicated Hosts. You picked Hosts. Storage line 44 explicitly puts "less expensive" on Dedicated Instances. |
| Q26 | S3 **Access Points** for per-prefix isolation. You picked Macie — Macie classifies/finds PII, it doesn't enforce access. |

**Fix:** Storage and Compute have small picker tables. Drill the **column you don't know**. EFS has *two* mode tables (performance + throughput) — quiz both. Dedicated Instance vs Host = "instances cheaper, hosts when per-socket licensing."

### Pattern C: Aurora/RDS HA-vs-scale conflation

| Q | The miss |
|---|---|
| Q30 | Heavy-read prod DB + daily clone for dev → you picked **RDS for SQL Server + read replica as dev**. The chapter teaches Aurora Cloning (better) and restoring-from-backup (acceptable). Read replicas are for *scaling reads*, not for spinning a *separate environment* that's safe to write to. |
| Q37 | Auto-scale + HA + MySQL-compatible + minimal ops → **Aurora Serverless v2**. You picked RDS Multi-AZ with read replicas — that gives HA and reads but NOT auto-scaling capacity. |
| Q49 | Encrypt existing RDS → snapshot → encrypted copy → restore. You picked "encrypt the read replica and promote." The chapter explicitly rules this out (line 103: encryption must be at creation, line 110: snapshot path is the only way). |

**Fix:** The DB Relational §1.4 table ("Multi-AZ vs Read Replicas") is in your notes but isn't sticking. Add to Block 1:
- *Multi-AZ = **survive failure** (sync, no reads on standby)*
- *Read Replica = **scale reads** (async, readable, no failover)*
- *Aurora Serverless v2 = **auto-scale capacity** + HA*
- *Aurora Cloning = **fast dev/test copy** (copy-on-write, seconds)*
- *Encrypt existing DB = **snapshot → copy-encrypt → restore** (no other path)*

### Pattern D: ASG default termination policy

| Q | The miss |
|---|---|
| Q6 | Oldest **Launch Configuration**, not oldest Launch Template. The page says exactly this (HA.md:198), and you picked oldest Launch Template. |

**Fix:** Add to Block 1 quiz: *"ASG default termination order: AZ with most → oldest **Launch Configuration** → closest billing hour."*

### Pattern E: A miscount of one

I originally listed Q16 under "🟧 covered." On re-read, the "impaired status" fact (one of the two correct answers) is not actually in your notes — only the grace period one is. So Q16 moved to 🟨 partial. You got one of the three right; the other two require facts that aren't taught.

---

## 4. The 9 partial misses — gaps in chapters you've studied

These reveal **where your chapter notes have holes** the exam exploits. Worth patching *before* relying on them in Phase 2 mocks.

| Q | Gap to patch | Where to add it |
|---|---|---|
| Q2 | User data runs as **root** by default | Compute.md §7.1 — add one bullet under "The Rule" |
| Q16 | ASG won't terminate when instance is in **Impaired** status (in addition to grace period) | HA.md §1 Health Checks — add bullet |
| Q23 | DataSync over DX → **private VIF + PrivateLink Interface Endpoint** for EFS (vs. S3 gateway endpoint path) | Storage §4.3 DataSync — add the network-path nuance |
| Q28 | RDS Multi-AZ **engine upgrades** apply to primary+standby **simultaneously** → downtime (different from instance failover, which is 60-120s) | DB Relational §1.4 — add a bullet on engine upgrade vs failover behavior |
| Q30 | Aurora Cloning is the ideal **dev/test clone** answer; restore-from-backup is acceptable | DB Relational §2.6 Aurora Cloning — already covered, but the chapter doesn't *contrast* it with the wrong answer (using a Read Replica for dev) which is what trips you. Add an "anti-pattern" line. |
| Q43 | Snowball cannot target Glacier *directly* — always land in S3 then lifecycle | Storage §4.2 Snow Family — add the "S3-as-landing-pad" rule |
| Q50 | S3 → SQS → Lambda is the canonical near-real-time per-upload processing pattern | This will land in Serverless (Day 7). Cross-link from Storage §2.10 S3 Event Notifications. |
| Q51 | 3-tier SG chain: ALB SG from `0.0.0.0/0:443`, App SG from ALB SG on app port, DB SG from App SG on DB port | Networking + HA — currently scattered; consolidate into a single "3-tier SG pattern" block in HA chapter |
| Q38 | Route 53 **latency routing** (vs geolocation, vs failover) | CDN/DNS chapter — landing Day 10 |

These 9 patches are not "I forgot" misses — they are *holes in your own chapter notes that the chapter could close*. Patching them is roughly 30 minutes of additive edits across 4 files.

---

## 5. The 14 not-covered — preview of what's coming

| Q | Chapter that will fix it | Day |
|---|---|---|
| Q21 (AWS Config + ACM imported cert) | Security & Identity + Monitoring | 11, 12 |
| Q29 (IAM Deny region + Allow IP) | Security & Identity | 11 |
| Q52 (`aws:RequestedRegion`) | Security & Identity | 11 |
| Q59 (Permission Boundary on users) | Security & Identity | 11 |
| Q46 (Route 53 failover + warm-standby DR) | Cost / Migration / **DR** | 9 |
| Q63, Q64 (Global Accelerator) | CDN / DNS | 10 |
| Q9 (S3 IAM policy ARN structure) | Security & Identity | 11 |
| Q27 (EFS storage classes) | **NOT in any current chapter** — patch Storage |
| Q8 (Shared Services VPC) | **NOT in any current chapter** — patch Networking |
| Q28 (RDS Multi-AZ engine-upgrade downtime) | **Patch DB Relational** |
| Q45 (3 AZs × 2 instances arithmetic) | Reasoning, not recall — drill in scenarios |
| Q57 (RI baseline mix strategy) | **Patch Compute** — or wait for Cost Optimization Day 9 |
| Q61 (SG outbound = destination port) | **Patch Networking** |

**Observation:** 4 of these "not covered" are actually **chapter gaps** in chapters you've already studied (Q27, Q8, Q28, Q57, Q61). These are the highest-leverage edits to your own notes.

---

## 6. The concrete plan (this is what actually matters)

### Today (May 25, post-mock REST per StudyGuide)
- **Do nothing else.** The plan says rest after the diagnostic. Trust it.

### One-time chapter patching (30–45 min, do once before Day 7)
Add these specific bullets to your chapters so the gaps stop being gaps:

1. **Compute.md §7.1 (User Data):** Add "Runs as **root** by default."
2. **Compute.md §2 purchasing table or new bullet:** Add "Mix RIs (baseline 80% of time = 80 instances) + on-demand/spot above for peaks."
3. **HA.md §2.1 Health Checks:** Add "ASG won't terminate instances in **Impaired** status or during the grace period."
4. **HA.md Section 3 (new):** Consolidate **3-tier SG chain** — ALB SG ←0.0.0.0/0:443, App SG ←ALB-SG:app-port, DB SG ←App-SG:DB-port. Spell out "outbound port = destination's listening port."
5. **Storage.md §3 EFS:** Add **EFS Storage Classes** subsection (Standard, Standard-IA, One Zone, One Zone-IA) — currently only performance/throughput modes are listed.
6. **Storage.md §4.2 Snowball:** Add "Snowball lands in S3 — to archive, add a Lifecycle rule to Glacier/Deep Archive on the same day."
7. **Networking.md new bullet near TGW:** "Shared Services VPC = a VPC attached to TGW that hosts shared resources (DNS, AD, monitoring) used by all spokes — reduces duplication; not the same as Transit VPC (legacy)."
8. **Networking.md §2 SG:** Add explicit "Outbound rule port = the destination's *listening* port, not your own."
9. **DB Relational §1.4 Multi-AZ:** Add "Engine upgrades apply to primary AND standby simultaneously — incurs downtime, unlike AZ failover (60-120s, no downtime)."
10. **DB Relational §2.6 Aurora Cloning:** Add "Anti-pattern: do NOT use a Read Replica as a dev/clone DB — replicas are for **scaling reads** of the same data, not for dev environments with separate writes."

### Modify Block 1 for Days 7–13
Add a 5-minute **"Mock-1 weak-trigger drill"** at the start of every Block 1. Rotation:

- **Day 7 (Tue May 26):** Q1 SQS-vs-SNS, Q11 User-vs-Identity Pool, Q50 S3→SQS→Lambda preview
- **Day 8 (Wed May 27):** Q7 EFS performance modes, Q25 Dedicated Instance vs Host, Q43 Snowball→S3→Glacier
- **Day 9 (Thu May 28):** Q57 RI mix strategy, Q46 DR patterns (lands today), Q6 ASG termination
- **Day 10 (Fri May 29):** Q63/Q64 Global Accelerator (lands today), Q38 latency routing
- **Day 11 (Sat May 30):** Q9, Q11, Q20, Q21, Q29, Q52, Q59 — all Security questions (lands today)
- **Day 12 (Sun May 31):** Q28 RDS engine upgrade, Q30 Aurora cloning vs replica, Q37 Serverless v2, Q49 snapshot-encrypt
- **Day 13 (Mon Jun 1):** Q6, Q16, Q51, Q61, Q8 — networking + ASG nuances

### Heading into Maarek #2 (Day 15, June 3)
- Expect score **65–75%**. That's the band where:
  - The 14 not-covered → mostly become covered.
  - The 9 partial → become covered if you patched the chapters.
  - The 9 genuine misses → mostly correct after drilling.
  - You'll pick up some new wrong answers on freshly-studied material — normal.
- **Anything < 60% on Maarek #2 is a real warning sign.** If that happens, slow Phase 2 by a day per heavy chapter.

### What NOT to do
- **Don't re-take Maarek #1.** You'll memorize answers, not reasoning.
- **Don't binge Security today** to "fix" the gaps. The plan front-loads heavy chapters where they belong. Going out of order breaks the spacing your retention depends on.
- **Don't lower the target.** Nothing in this diagnostic suggests you're off-pace.

---

## 7. One-paragraph honest summary

You took a 65-question exam with about half the content unstudied and scored 50%. After a full audit against the actual text of every studied chapter: 9 misses (28%) are facts that were on the page and didn't stick — drillable in 5-minute Block 1 sessions. 9 are *partial* — facts where your chapter mentions the topic but skips the exam-tested nuance — fixable by adding 10 bullets across 4 files. The remaining 14 are content from chapters you literally haven't opened or holes the chapters never had — those resolve naturally as Days 7–13 land, with two specific gaps (EFS storage classes, Shared Services VPC) needing one-line additions. The single biggest correction to v1 of this report: many questions I called "covered-but-missed" were actually *partially covered* because the chapters themselves had gaps — that's not a retention failure, it's a notes failure, and notes are fixable in minutes. Net read: the plan is on track, but **your chapters have ~10 small holes worth patching before Day 7**.

---

## Appendix — Verified wrong-answer corrections (only the ones your chapter teaches)

For every miss, this is what the *answer key* + your chapter together say. Question numbers map to questions.txt.

**Genuinely in your notes:**
- **Q1:** IoT ingest, lost data, replay needed → SQS (chapter teaches "queue absorbs spike, workers drain at their own pace")
- **Q6:** ASG default termination → AZ-most-instances → oldest **Launch Configuration** → closest billing hour
- **Q7:** EFS for big-data parallel workloads → **Max I/O** performance mode
- **Q10:** Replace on-prem Windows DFSR → **FSx for Windows File Server** (SMB + AD)
- **Q11:** Authenticate at ALB with minimal code → **Cognito User Pools** on the ALB (HA.md:53)
- **Q25:** Cheapest single-tenant EC2 → **Dedicated Instances** (Dedicated Hosts are pricier — they exist for per-socket licensing visibility)
- **Q26:** Per-prefix S3 isolation, low ops → **S3 Access Points** with per-AP policies
- **Q37:** MySQL-compatible + auto-scaling + HA + low ops → **Aurora Serverless v2**
- **Q49:** Encrypt existing unencrypted RDS → **snapshot → copy as encrypted → restore** (no in-place option; no read-replica path)

**Partially in your notes (chapter half-teaches, exam tests the missing half):**
- **Q2:** EC2 user data → ✅ first boot only / ❌ root privileges (add to chapter)
- **Q16:** ASG won't terminate → ✅ grace period not expired / ❌ Impaired status (add)
- **Q23:** On-prem NFS → EFS via DX → DataSync + **private VIF + PrivateLink Interface Endpoint** (Storage mentions DataSync; not the DX/VIF path)
- **Q28:** RDS Multi-AZ engine upgrade → **both upgraded together, downtime** (chapter covers failover but not upgrades)
- **Q30:** Aurora dev clone for read-heavy prod → Aurora Cloning (or restore-from-backup). NOT a Read Replica (replicas scale reads, don't make safe dev environments)
- **Q38:** Latency-based EU performance fix → Route 53 **latency routing** + Aurora cross-region replica
- **Q43:** Snowball → archive → Snowball lands in **S3**, then **lifecycle to Glacier Deep Archive** (can't target Glacier directly)
- **Q50:** Near-real-time per-upload image processing → S3 event → SQS → Lambda
- **Q51:** 3-tier SG chain → ALB SG ←0.0.0.0/0:443 · App SG ←ALB-SG:80 · DB SG ←App-SG:5432

**Not in your notes (chapter gap or future chapter — list pending Day 7+ coverage):**
- Q8, Q9, Q21, Q27, Q28 nuance, Q29, Q45, Q46, Q52, Q57, Q59, Q61, Q63, Q64 — all detailed in section 5 above.
