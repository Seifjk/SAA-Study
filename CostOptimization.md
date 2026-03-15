# Cost Optimization & Migration

*Domain 4 of SAA-C03 = 20% of exam. This is critical.*

### **SECTION 1: COST MANAGEMENT TOOLS**

### **1. AWS Cost Explorer**

- **The Rule:** Visualize, understand, and manage AWS costs/usage over time.
- **Features:**
    - View spending by service, account, tag, region.
    - **Forecast** future costs based on historical data.
    - **Right-sizing recommendations** for EC2.
    - **Savings Plans recommendations.**
- *Exam Trigger:* "Analyze spending patterns", "Forecast next month's cost" → Cost Explorer.

### **2. AWS Budgets**

- **The Rule:** Set custom **cost or usage budgets** and get alerts when thresholds are exceeded.
- **Types:**
    - **Cost Budget:** Alert when spending exceeds $X.
    - **Usage Budget:** Alert when usage exceeds X hours.
    - **Reservation Budget:** Track RI/Savings Plan utilization.
- **Actions:** Trigger SNS, apply IAM policy, or stop instances when budget exceeded.
- *Exam Trigger:* "Alert when monthly spend exceeds $10,000" → AWS Budgets.

### **3. AWS Compute Optimizer**

- **The Rule:** ML-based recommendations to **right-size** EC2, Lambda, EBS, and ECS on Fargate.
- **Output:** Over-provisioned, Under-provisioned, or Optimized.
- *Exam Trigger:* "Identify over-provisioned EC2 instances" → Compute Optimizer.

### **4. S3 Storage Lens**

- **The Rule:** Organization-wide visibility into S3 storage usage and activity metrics.
- **Use Case:** Identify buckets with no lifecycle policies, find unused storage.
- *Exam Trigger:* "Analyze S3 storage usage across organization" → S3 Storage Lens.

---

### **SECTION 2: SAVINGS STRATEGIES (Cheat Sheet)**

| **Strategy** | **Savings** | **Exam Trigger** |
| --- | --- | --- |
| **Reserved Instances (1/3yr)** | 40-75% | "Steady-state, Predictable" |
| **Savings Plans** | ~Same as RI | "Flexible across families/regions" |
| **Spot Instances** | Up to 90% | "Fault-tolerant batch jobs" |
| **S3 Lifecycle Policies** | Variable | "Move old data to cheaper tier" |
| **S3 Intelligent-Tiering** | Auto | "Unknown access patterns" |
| **Right-sizing (Compute Optimizer)** | 10-50% | "Over-provisioned instances" |
| **EBS gp3 over gp2** | ~20% | "Independent IOPS scaling" |
| **NAT Gateway vs NAT Instance** | NAT Instance cheaper | "Budget-constrained private subnet internet" |

---

### **SECTION 3: MIGRATION SERVICES**

### **1. AWS DMS (Database Migration Service)**

- **The Rule:** Migrate databases to AWS with **minimal downtime**.
- **Types:**
    - **Homogeneous:** Same engine (e.g., Oracle → Oracle on RDS). Direct migration.
    - **Heterogeneous:** Different engine (e.g., Oracle → PostgreSQL). Requires **SCT (Schema Conversion Tool)** first.
- **CDC (Change Data Capture):** Replicate ongoing changes during migration (Near-zero downtime).
- *Exam Trigger:* "Migrate on-prem database to RDS with minimal downtime" → DMS.

### **2. AWS Application Migration Service (MGN)**

- **The Rule:** **Lift-and-shift** servers to AWS (Formerly Server Migration Service / CloudEndure).
- **How:** Install agent on source server → Continuous replication to AWS → Cutover when ready.
- *Exam Trigger:* "Migrate on-prem servers to EC2 with minimal changes" → MGN.

### **3. AWS DataSync**

- **The Rule:** **Online data transfer** between on-prem storage and AWS (S3, EFS, FSx).
- **Speed:** Up to 10 Gbps. Uses agent on-prem.
- **Features:** Automatic encryption, data integrity validation, scheduling.
- **Comparison:**
    - DataSync = **Online** migration, automated, fast.
    - Snow Family = **Offline** migration (ship physical device).
- *Exam Trigger:* "Transfer large dataset from on-prem NFS to S3 over network" → DataSync.

### **4. AWS Transfer Family**

- **The Rule:** Managed **SFTP/FTPS/FTP** server for transferring files to/from S3 or EFS.
- *Exam Trigger:* "Partners upload files via SFTP to S3" → Transfer Family.

---

### **SECTION 4: DISASTER RECOVERY PATTERNS**

*These 4 strategies are heavily tested. Memorize the RPO/RTO.*

| **Strategy** | **RPO** | **RTO** | **Cost** | **Exam Trigger** |
| --- | --- | --- | --- | --- |
| **Backup & Restore** | Hours | Hours-Days | Lowest | "Cheapest DR", "Non-critical" |
| **Pilot Light** | Minutes | 10s of minutes | Low | "Core DB always running" |
| **Warm Standby** | Seconds | Minutes | Medium | "Scaled-down copy running" |
| **Multi-Site Active-Active** | Near-zero | Near-zero | Highest | "Zero downtime", "Both regions active" |

### **1. Backup & Restore**
- Backup data to S3/Glacier. Restore when disaster hits.
- Services: AWS Backup, S3 CRR, RDS Snapshots.

### **2. Pilot Light**
- Core infrastructure (DB) always running in DR region. Other services launched when needed.
- Example: RDS Multi-AZ in DR region, EC2/ASG launched from AMIs on failover.

### **3. Warm Standby**
- Scaled-down but fully functional copy of production in DR region.
- On failover: Scale up instances to full capacity.

### **4. Multi-Site Active-Active**
- Full production in both regions. Route 53 health checks route traffic.
- Most expensive but near-zero RPO/RTO.

---

### **SECTION 5: AWS ORGANIZATIONS & GOVERNANCE**

### **1. AWS Organizations**

- **The Rule:** Manage **multiple AWS accounts** centrally.
- **Consolidated Billing:** Single bill, volume discounts, RI sharing across accounts.
- **SCPs (Service Control Policies):** Restrict what services/actions accounts can use.
    - SCPs **don't grant** permissions. They set **guardrails** (maximum allowed permissions).
- *Exam Trigger:* "Prevent any account from launching in unapproved region" → SCP.

### **2. AWS Control Tower**

- **The Rule:** Automated setup and governance of **multi-account environment**.
- **Guardrails:** Pre-configured governance rules (Preventive = SCP, Detective = Config Rules).
- *Exam Trigger:* "Set up best-practice multi-account environment" → Control Tower.

### **3. AWS Backup**

- **The Rule:** Centralized backup service across AWS services (EC2, EBS, RDS, DynamoDB, EFS, FSx, S3).
- **Backup Plans:** Define schedule, retention, cross-region copy.
- **Vault Lock:** WORM (Write Once Read Many) for compliance.
- *Exam Trigger:* "Centralized backup across multiple services" → AWS Backup.

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **Analyze spending patterns?** → Cost Explorer.
2. **Alert when budget exceeded?** → AWS Budgets.
3. **Right-size EC2 instances?** → Compute Optimizer.
4. **Migrate database with minimal downtime?** → DMS (+ SCT if heterogeneous).
5. **Lift-and-shift servers to EC2?** → Application Migration Service (MGN).
6. **Transfer files via SFTP to S3?** → Transfer Family.
7. **Online data transfer (NFS to S3)?** → DataSync.
8. **Offline data transfer (100 TB)?** → Snow Family.
9. **Cheapest DR strategy?** → Backup & Restore.
10. **Near-zero RTO DR?** → Multi-Site Active-Active.
11. **Restrict services across accounts?** → SCPs (Organizations).
12. **Centralized backup?** → AWS Backup.
