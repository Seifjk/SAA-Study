# Coverage Audit — In-Scope Services vs Chapter Coverage

**Built:** 2026-05-31 · **Source of truth:** `_examguide.txt` (official SAA-C03 in-scope list, lines 501–677) cross-checked against all 14 chapters via `/tmp/coverage.sh`.

**Headline:** Coverage is strong. Of all high-value in-scope services, only **3 hard gaps** + a handful of partials/nuance-gaps. This cross-validates Mock1Analysis (which independently flagged EFS storage classes + Shared Services VPC).

> How to read: a service is "covered" if it appears in any chapter. This audit finds **missing services/features**, not over-depth — trimming (build-it cuts) is the separate `chapter_trimming_method.md` job.

---

## 🔴 HARD GAPS — in-scope, not in any chapter (ADD these)

| # | Missing item | In-scope per guide | Add to | Source |
|---|---|---|---|---|
| G1 | **EFS Storage Classes** (Standard, Standard-IA, One Zone, One Zone-IA) | Storage cost tiering (TS 4.1) | `Storage.md` EFS section — currently only has perf modes (GP/Max I/O) + throughput modes (Elastic/Provisioned/Bursting), NO storage classes | Mock1 Q27 |
| G2 | **Shared Services VPC** (hub VPC on TGW hosting shared DNS/AD/monitoring for spokes) | Network topology (TS 4.4 TGW/peering) | `Networking.md` near Transit Gateway | Mock1 Q8 |
| G3 | **Cost and Usage Report (CUR)** | Named explicitly in TS 4.1–4.4 "cost management tools" | `CostOptimization.md` — has Cost Explorer + Budgets but not CUR (the most granular billing export) | Guide lines 532, 347 |

## 🟡 NUANCE GAPS — service covered, but exam-tested detail missing (from Mock1Analysis, never patched)

| # | Missing nuance | Add to |
|---|---|---|
| N1 | EC2 user data runs as **root** by default | `Compute.md` §7 User Data |
| N2 | ASG won't terminate instance in **Impaired** status (beyond grace period) | `HighAvailability.md` health checks |
| N3 | RDS Multi-AZ **engine upgrades** hit primary+standby simultaneously → downtime (vs failover 60-120s) | `DatabasesRelational.md` Multi-AZ |
| N4 | SG **outbound** rule port = destination's *listening* port | `Networking.md` security groups |
| N5 | Snowball can't target Glacier directly — land in S3 then lifecycle | `Storage.md` Snow Family |
| N6 | RI mix strategy: RIs for steady baseline + Spot/On-Demand for peak | `Compute.md` purchasing or CostOptimization |
| N7 | Aurora Cloning is the dev/test-clone answer; Read Replica is the anti-pattern | `DatabasesRelational.md` Aurora |
| N8 | DataSync over DX uses **private VIF + PrivateLink interface endpoint** for EFS | `Storage.md` DataSync |

## 🟢 PARTIALS — lightly covered, probably fine but verify depth
- **Directory Service** — covered indirectly in `SecurityIdentity.md` (AWS Managed AD / AD Connector mentioned under SAML federation, lines 109/122). No standalone treatment; likely sufficient for SAA.
- **AWS Batch** — only in AdditionalServices. Fine (minor exam presence; mainly "long job > 15min → Batch/Fargate").
- **Outposts, MSK, Lake Formation, QuickSight** — AdditionalServices only. Minor weight, OK as-is.

---

## ✅ WELL COVERED (no action) — confirmed present, often across many chapters
Lambda, Fargate, ECS, EKS, ECR, EC2 ASG, Step Functions, API Gateway, X-Ray (MonitoringAudit only — OK), SQS, SNS, EventBridge, Amazon MQ, AppSync, RDS, Aurora, Aurora Serverless, DynamoDB, ElastiCache, DocumentDB, Neptune, Keyspaces, QLDB, Redshift, RDS Proxy, S3, S3 Glacier, EBS, EFS, FSx, Storage Gateway, AWS Backup, VPC, CloudFront, Route 53, Global Accelerator, Direct Connect, Site-to-Site VPN, Client VPN, PrivateLink, Transit Gateway, ELB/ALB/NLB, Network Firewall, IAM, IAM Identity Center, Cognito, KMS, CloudHSM, ACM, Secrets Manager, GuardDuty, Macie, Inspector, Shield, WAF, Firewall Manager, Security Hub, RAM, Organizations/SCP, Control Tower, Detective, CloudWatch, CloudTrail, Config, CloudFormation, Systems Manager, Trusted Advisor, Compute Optimizer, DataSync, DMS, Snow Family, Transfer Family, Kinesis, Athena, Glue, EMR, Savings Plans, Cost Explorer, Budgets.

## How to regenerate
`zsh /tmp/coverage.sh` (script lists each service + which chapters mention it). Tune regex patterns if false-negatives appear. Verify any GAP by hand-grepping the target chapter before trusting it.
