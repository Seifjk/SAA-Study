# SAA-C03 Study Guide

**Start Date:** Monday, May 18, 2026
**Pace:** 2 hours/day (two 50-min blocks with a 10-min break)
**Target Exam Date:** Thursday, June 11, 2026

> ## 🔄 SCHEDULE RESET — June 4, 2026
> The original day-by-day plan slipped. **Reality as of today (Jun 4):** 10 of 14 chapters done; starting **CDN/DNS today**. 4 chapters left to learn (CDN/DNS, Security, Monitoring, Additional) and **7 days to the exam**.
> **The new plan compresses learning into Jun 4–7, then runs 3 mocks Jun 8–10.** This is tight but doable because (a) the remaining chapters are mostly Medium/Light, (b) the chapters are already trimmed to exam scope, and (c) Recall.md now covers all 14 chapters for daily review. **Mocks are non-negotiable** — they're the highest-value thing left; don't let learning eat all 7 days.
> See the updated Content Map and the **Compressed Plan (Jun 4–11)** section below. The original Phase-1 day notes are kept at the bottom for reference.

**Already done (10/14):** Networking ✅ · Compute ✅ · High Availability ✅ · Storage ✅ · DB Relational ✅ · DB NoSQL ✅ · Decoupling ✅ · Serverless ✅ · Containers ✅ · Cost/Migration/DR ✅
**Left to learn (4/14):** CDN/DNS · Security & Identity · Monitoring & Audit · Additional Services
**Mocks taken:** Maarek #1 diagnostic (May 24) = **50%**

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
4. **Mock exams** — 1 early diagnostic (May 24, after ~60% of content) + 3 Phase 2 mocks (Jun 8–10) force full recall of all 14 chapters

---

## Content Map (Compressed — current as of Jun 4)

| Date | Plan | File | Load |
|------|------|------|------|
| **Thu Jun 4** | **CDN / DNS** + review one old cheat sheet | [ContentDeliveryDNS](./ContentDeliveryDNS.md) | ✅ Done |
| **Fri Jun 5** | **Security & Identity** (heaviest remaining) | [SecurityIdentity](./SecurityIdentity.md) | Heavy |
| **Sat Jun 6** | **Monitoring & Audit** + **Additional Services** (skim) | [MonitoringAudit](./MonitoringAudit.md) · [AdditionalServices](./AdditionalServices.md) | Medium + Light |
| **Sun Jun 7** | **Full recall sweep** — all 14 chapters via [Recall.md](./Recall.md) | [Recall](./Recall.md) | Review |
| **Mon Jun 8** | **MOCK #2** (Maarek #2) + review every wrong answer | — | Mock |
| **Tue Jun 9** | **MOCK #3** (Davis #1 or Bonso #1) + review | — | Mock |
| **Wed Jun 10** | **MOCK #4** (Bonso) + final weak-list review | — | Mock — **GO/NO-GO** |
| **Thu Jun 11** | **EXAM DAY** — light skim of weak list only | — | Exam |

*All 14 chapters now have a Practice (fill-in) + Answer-Key cheat sheet, and Recall.md covers all 14 — use those for the fast daily review, not full rereads.*

---

## Compressed Plan (Jun 4 – Jun 11)

**The shape stays the same** — Block 1 = review an older cheat sheet (cover answers, self-test) + start new content · Break · Block 2 = finish the chapter, do all 5 scenarios, write the key table from memory. **If you only have time for one thing each learning day, do the scenarios** — they're closest to the real exam.

> **📝 Topic drills (NOT mocks):** End each content day with a short **Tutorials Dojo (Bonso) Section-Based / Topic quiz** on the chapter you just finished — ~10–15 questions, untimed. This is *not* a mock; it tests only what's fresh and builds trap-spotting (most of your Mock-1 misses were test-taking, not knowledge). **Review every wrong answer immediately** — that's the point, not the score. Full timed mocks stay on Jun 8–10, when all content is done.

### 🟢 Thu Jun 4 — CDN / DNS ← TODAY
- **Block 1:** Quick review of an older cheat sheet (Cost or Networking — pick whatever feels rustiest, cover the answers). Then start ContentDeliveryDNS.md: the **7 Route 53 routing policies** table, Alias vs CNAME, Route 53 Resolver.
- **Block 2:** Finish it — CloudFront **OAC**, Signed URLs vs Cookies, **CloudFront Functions vs Lambda@Edge**, Origin Groups, **CloudFront vs Global Accelerator**. Do all 5 scenarios. **Write from memory:** the 7 routing policies; Functions vs Lambda@Edge.
- **Close (~15 min):** TD topic quiz → *Route 53 / CloudFront / Edge*. Review misses.

### 🔴 Fri Jun 5 — Security & Identity (heaviest remaining — give it your sharpest block)
- **Block 1:** Review CDN/DNS cheat sheet (quiz it). Then start SecurityIdentity.md: **IAM evaluation logic** (Explicit Deny > Allow > Default Deny), Permission Boundaries, STS / role assumption, IAM Identity Center.
- **Block 2:** Finish it — **KMS** + Envelope Encryption, **KMS vs CloudHSM**, ACM (CloudFront → us-east-1), **Secrets Manager vs Parameter Store**, **Cognito User Pools vs Identity Pools**, WAF/Shield, GuardDuty/Inspector/Macie, the AD/federation triggers. Do all 5 scenarios. **Write from memory:** IAM eval logic; Cognito pools; KMS vs CloudHSM.
- **Close (~15 min):** TD topic quiz → *Security / IAM* (worth extra here — heaviest domain, 30% of exam). Review misses.

### 🟡 Sat Jun 6 — Monitoring & Audit + Additional Services (two-fer)
- **Block 1:** Review Security cheat sheet. Then MonitoringAudit.md: the **CloudWatch vs CloudTrail vs Config** table (this is the whole chapter's core), CloudWatch Agent, X-Ray, SSM Session Manager. Do its 5 scenarios.
- **Block 2:** AdditionalServices.md — **skim, don't memorize**. These are ~1 question each: Athena, Redshift, Glue, CloudFormation, Beanstalk, OpenSearch, Batch, EMR, ML services, Outposts/Local Zones/Wavelength. Just lock the **trigger word → service** for each (the cheat sheet is enough). **Write from memory:** CloudWatch vs CloudTrail vs Config.
- **Close (~10 min):** TD topic quiz → *Management & Governance* (covers Monitoring + most Additional). Review misses.

### 🔵 Sun Jun 7 — Full Recall Sweep (all 14 chapters)
- **Block 1 + 2:** Work through **all of Recall.md** — cover the answers, say the trigger→answer out loud, mark every miss. This is your last full pass before mocks. *No new questions today — consolidation only.*
- Make a **paper weak-list** of everything that didn't come back instantly. That list is what you drill between mocks.

---

## Phase 2: Mocks (Jun 8 – Jun 10)

**3 mocks in 3 days.** Same structure: Block 1 (65 min) take the mock under exam conditions · Break · Block 2 (50 min) review **every** wrong answer, going back to the chapter for each.

> **Your 3 banks (you own all three):** **Maarek** = straightforward, closest to AWS phrasing · **Neal Davis (DCT)** = hardest, runs ~8% low · **Tutorials Dojo / Bonso** = closest to the real exam, best predictor. *(TD and "Bonso" are the same bank — Jon Bonso wrote it. You used TD's Topic mode for the content-day drills; here you take its full timed exam.)*

| Date | Mock (full, timed) | Target | Notes |
|------|------|--------|-------|
| Mon Jun 8 | **Maarek #2** | jump from 50% | First full-content mock. Compare to the May 24 diagnostic — expect a clear rise. Easiest bank — good confidence check. |
| Tue Jun 9 | **Neal Davis #1** | don't panic if low | Hardest bank (~8% under). A 60% here ≈ ~68% real. Tests depth. |
| Wed Jun 10 | **Tutorials Dojo / Bonso (full exam)** | **GO/NO-GO** | Best real-exam predictor — this is your decision signal. See table below. |

### Wed Jun 10 — GO / NO-GO
| Score (Bonso/Maarek scale) | Read | Action |
|---|---|---|
| 72%+ | Passing range | Go. Light weak-list review only on Jun 11. |
| 65–71% | Borderline | Go, but spend Jun 10 evening + Jun 11 morning hammering the weak list. |
| <65% | High risk | Seriously consider **rescheduling 1 week** — better than a fail + 14-day wait + retake fee. |

> **Reality check on compression:** the old plan had 8 mocks; you'll get 3. That's the cost of the slip — but 3 mocks with disciplined wrong-answer review still calibrates you. The bigger risk now is letting the 4 chapters bleed past Jun 7. **Protect the mock days.**

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
DIAGNOSTIC (May 24) Maarek #1:    _50_% | Weak: _______________ ← BASELINE (~60% content)
Mock #2 (Jun 8)  Maarek #2:       ___% | Weak: _______________
Mock #3 (Jun 9)  Neal Davis #1:   ___% | Weak: _______________  (hardest — expect ~8% low)
Mock #4 (Jun 10) TutorialsDojo:   ___% | Weak: _______________ ← GO/NO-GO (best predictor)
```

**Healthy:** scores trend upward from the 50% diagnostic. Davis runs ~8% below Maarek (normal). Bonso is your best predictor of the real exam.
**Warning:** flat or dropping across the 3 mocks → reread your 2 weakest chapters from the weak list, don't take another mock blind.

---

## If You Slip Again (the plan is already compressed — be ruthless)

- **Behind on a learning day (Jun 4–7):** Don't reread — study **only** from each chapter's cheat sheet + 5 scenarios. That's ~80% of the exam value in ~40% of the time.
- **If Jun 7 arrives and a chapter isn't done:** Skip the full read entirely — learn it from its Recall.md section + cheat sheet only. Do NOT sacrifice a mock day for it.
- **Can't fit 3 mocks:** Protect at least **2** — one to calibrate (Jun 8) and the Jun 10 go/no-go. A mock + wrong-answer review beats another chapter reread at this stage.
- **If Jun 10 go/no-go is <65%:** Rescheduling 1 week is the smart move, not a failure. A fail costs the fee + a 14-day lockout anyway.
- **Golden rule:** never skip the Block 1 cheat-sheet review to cram new content. Fading old material is worse than being a day behind.

---

## What You Need

- [x] All 14 chapter files (in this folder — see [Chapters.md](./Chapters.md)) — all trimmed to SAA scope + practice/answer cheat sheets
- [ ] Stephane Maarek SAA-C03 practice exams (Udemy) — for Jun 8 mock
- [ ] Neal Davis / Tutorials Dojo + Bonso SAA-C03 practice exams (Udemy) — for Jun 9–10 mocks
- [ ] A notebook for writing from memory (paper > app for retention)
- [ ] A timer for the 50-min blocks
- [x] AWS Certification account + booked exam slot for June 11
