# Coverage Audit — In-Scope Services vs Chapter Coverage

**Built:** 2026-05-31 · **Last re-run:** 2026-06-12 (post-trim, verified by hand) · **Source of truth:** `_examguide.txt` (official SAA-C03 in-scope list, lines 501–677) cross-checked against all 14 chapters via `_coverage_audit.sh`.

**Headline (2026-06-12 re-run):** Coverage is now **complete** — **0 hard gaps remain**. All 3 original hard gaps are resolved (G2/G3 patched into chapters; G1 was a script false-negative — Storage.md lines 177–179 already had it). All 8 nuance gaps confirmed present in `Recall.md`. The one residual item (N8 DataSync-over-DX) was patched into `Recall.md` §4b on 2026-06-12.

> How to read: a service is "covered" if it appears in any chapter. This audit finds **missing services/features**, not over-depth — trimming (build-it cuts) is the separate `chapter_trimming_method.md` job.

---

## ✅ HARD GAPS — ALL RESOLVED (was 3, now 0)

| # | Item | Status (2026-06-12) | Where it lives now |
|---|---|---|---|
| G1 | **EFS Storage Classes** (Standard, Standard-IA, One Zone, One Zone-IA) | ✅ Was a **false negative** — script regex couldn't span the table formatting | `Storage.md` lines 177–179 (all 4 classes + Lifecycle + exam trigger). Regex fixed in `_coverage_audit.sh`. |
| G2 | **Shared Services VPC** (hub VPC on TGW hosting shared DNS/AD/monitoring) | ✅ Patched | `Networking.md` (near TGW) + `Recall.md` §1 line 169 🔴 |
| G3 | **Cost and Usage Report (CUR)** | ✅ Patched | `CostOptimization.md` + `Recall.md` §13 line 542 |

## ✅ NUANCE GAPS — ALL CONFIRMED PRESENT IN Recall.md (was 8 open, now 0)

| # | Nuance | Status | Where |
|---|---|---|---|
| N1 | EC2 user data runs as **root**, once at first boot | ✅ | `Recall.md` §2 line 203 |
| N2 | ASG won't terminate instance in **Impaired** status (beyond grace period) | ✅ | `Recall.md` §3 line 245 |
| N3 | RDS Multi-AZ **engine upgrades** hit primary+standby → downtime (use Blue/Green) | ✅ | `Recall.md` §5 lines 51, 307 |
| N4 | SG **outbound** rule port = destination's *listening* port (ref source SG) | ✅ | `Recall.md` §1 line 183 🔴 |
| N5 | Snowball can't target Glacier directly — land in S3 then lifecycle | ✅ | `Recall.md` §4b line 295 |
| N6 | RI mix: RIs for steady baseline + Spot/On-Demand for peak | ✅ | `Recall.md` §13 line 548 |
| N7 | Aurora Cloning is the dev/test-clone answer; Read Replica is the anti-pattern | ✅ | `Recall.md` §5 lines 50, 316 |
| N8 | DataSync over DX uses **private VIF + PrivateLink interface endpoint** | ✅ **Patched 2026-06-12** | `Recall.md` §4b (new line after DataSync) |

## 🟢 PARTIALS — lightly covered, probably fine but verify depth
- **Directory Service** — covered indirectly in `SecurityIdentity.md` (AWS Managed AD / AD Connector mentioned under SAML federation, lines 109/122). No standalone treatment; likely sufficient for SAA.
- **AWS Batch** — only in AdditionalServices. Fine (minor exam presence; mainly "long job > 15min → Batch/Fargate").
- **Outposts, MSK, Lake Formation, QuickSight** — AdditionalServices only. Minor weight, OK as-is.

---

## ✅ WELL COVERED (no action) — confirmed present, often across many chapters
Lambda, Fargate, ECS, EKS, ECR, EC2 ASG, Step Functions, API Gateway, X-Ray (MonitoringAudit only — OK), SQS, SNS, EventBridge, Amazon MQ, AppSync, RDS, Aurora, Aurora Serverless, DynamoDB, ElastiCache, DocumentDB, Neptune, Keyspaces, QLDB, Redshift, RDS Proxy, S3, S3 Glacier, EBS, EFS, FSx, Storage Gateway, AWS Backup, VPC, CloudFront, Route 53, Global Accelerator, Direct Connect, Site-to-Site VPN, Client VPN, PrivateLink, Transit Gateway, ELB/ALB/NLB, Network Firewall, IAM, IAM Identity Center, Cognito, KMS, CloudHSM, ACM, Secrets Manager, GuardDuty, Macie, Inspector, Shield, WAF, Firewall Manager, Security Hub, RAM, Organizations/SCP, Control Tower, Detective, CloudWatch, CloudTrail, Config, CloudFormation, Systems Manager, Trusted Advisor, Compute Optimizer, DataSync, DMS, Snow Family, Transfer Family, Kinesis, Athena, Glue, EMR, Savings Plans, Cost Explorer, Budgets.

## How to regenerate
`zsh /tmp/coverage.sh` (script lists each service + which chapters mention it). Tune regex patterns if false-negatives appear. Verify any GAP by hand-grepping the target chapter before trusting it.
