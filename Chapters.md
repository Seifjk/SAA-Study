# SAA-C03 Chapter Index

15 content chapters covering the four SAA-C03 exam domains. Each file = core content + Exam Cheat Sheet + 5 real exam scenarios.

**Status legend:** ✅ studied · 🔲 not yet

---

## The Infrastructure Core

| # | Chapter | File | Status | Exam weight |
|---|---------|------|--------|-------------|
| 1 | Networking (VPC) | [Networking.md](./Networking.md) | ✅ | High — VPC/SG/NACL heavily tested |
| 2 | Storage (EBS, S3, EFS, FSx) | [Storage.md](./Storage.md) | 🔲 | High — S3 classes + EBS types |
| 3 | Compute (EC2) | [Compute.md](./Compute.md) | 🔲 | High — purchasing, placement groups |
| 4 | High Availability (ELB + Auto Scaling) | [HighAvailability.md](./HighAvailability.md) | 🔲 | **Very high — ~20% of exam** |

## The Data Layer

| # | Chapter | File | Status | Exam weight |
|---|---------|------|--------|-------------|
| 5 | Databases — Relational (RDS, Aurora) | [DatabasesRelational.md](./DatabasesRelational.md) | 🔲 | High — Multi-AZ vs Read Replicas |
| 6 | Databases — NoSQL & Caching (DynamoDB, ElastiCache) | [DatabasesNoSQL.md](./DatabasesNoSQL.md) | 🔲 | High — DynamoDB + purpose-built DBs |

## The Modern Stack

| # | Chapter | File | Status | Exam weight |
|---|---------|------|--------|-------------|
| 7 | Serverless (Lambda, API Gateway, Step Functions) | [Serverless.md](./Serverless.md) | 🔲 | High |
| 8 | Containers (ECS, EKS, Fargate, ECR) | [Containers.md](./Containers.md) | 🔲 | Medium |
| 9 | Decoupling (SQS, SNS, Kinesis, EventBridge) | [Decoupling.md](./Decoupling.md) | 🔲 | High |

## The Global & Security Layer

| # | Chapter | File | Status | Exam weight |
|---|---------|------|--------|-------------|
| 10 | Content Delivery & DNS (Route 53, CloudFront, GA) | [ContentDeliveryDNS.md](./ContentDeliveryDNS.md) | 🔲 | Medium-High |
| 11 | Security & Identity (IAM, KMS, Cognito, WAF) | [SecurityIdentity.md](./SecurityIdentity.md) | 🔲 | **Very high — Domain 1 = 30%** |
| 12 | Monitoring & Audit (CloudWatch, CloudTrail, Config) | [MonitoringAudit.md](./MonitoringAudit.md) | 🔲 | Medium |

## Cost, Migration & Governance — Domain 4 = 20%

| # | Chapter | File | Status | Exam weight |
|---|---------|------|--------|-------------|
| 13 | Cost Optimization & Migration & DR | [CostOptimization.md](./CostOptimization.md) | 🔲 | High — DR patterns + cost tools |

## Additional Services — 1-2 questions each

| # | Chapter | File | Status | Exam weight |
|---|---------|------|--------|-------------|
| 14 | Additional Services (Athena, Redshift, Glue, etc.) | [AdditionalServices.md](./AdditionalServices.md) | 🔲 | Low per topic, adds up |

---

## Exam domain mapping

| Domain | Weight | Covered by chapters |
|--------|--------|---------------------|
| 1 — Design Secure Architectures | 30% | 11, 1 |
| 2 — Design Resilient Architectures | 26% | 4, 5, 13 (DR) |
| 3 — Design High-Performing Architectures | 24% | 3, 2, 5, 6, 9, 10 |
| 4 — Design Cost-Optimized Architectures | 20% | 13 |

**The day-by-day plan with dates lives in [StudyGuide.md](./StudyGuide.md).** Update the Status column here as you finish each chapter.
