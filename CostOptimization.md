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

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Cheapest DR" (Backup & Restore vs Pilot Light)**

**The Situation:** A company runs a non-critical internal application on EC2 with an RDS MySQL database. They need a DR strategy for their secondary region. The business accepts up to **24 hours of downtime** and up to **4 hours of data loss**. They want the **lowest cost** DR option.

**The Options:**

A. Deploy a Warm Standby with a scaled-down copy of production in the DR region.

B. Use Multi-Site Active-Active with Route 53 health checks.

C. Back up RDS snapshots and AMIs to the DR region using AWS Backup. Restore when disaster hits.

D. Keep a Pilot Light with the RDS database always running in the DR region.

**The Logic:**

- **Trap:** Warm Standby (Option A) and Multi-Site (Option B) are far more expensive than needed for 24-hour RTO.
- **Trap:** Pilot Light (Option D) keeps the DB running 24/7 in the DR region — cheaper than Warm Standby but more expensive than needed when 24-hour downtime is acceptable.
- **The Fix:** **Option C**. **Backup & Restore** is the cheapest DR strategy. RPO = hours (last backup), RTO = hours (restore from snapshot + launch from AMI). Meets the 24-hour downtime and 4-hour data loss requirements at the lowest cost.

---

### **Scenario 2: The "Database Migration" (DMS + SCT)**

**The Situation:** A company needs to migrate their on-premises **Oracle** database to **Amazon Aurora PostgreSQL**. They require **near-zero downtime** during migration and the application must continue serving traffic throughout the migration.

**The Options:**

A. Use AWS DMS with Change Data Capture (CDC) for continuous replication.

B. Export the Oracle database to a dump file, upload to S3, and import into Aurora.

C. Use AWS DMS with the Schema Conversion Tool (SCT) first, then DMS with CDC.

D. Use AWS DataSync to transfer the database files to Aurora.

**The Logic:**

- **Trap:** DMS alone (Option A) handles data replication but Oracle → PostgreSQL is a **heterogeneous** migration — the schemas are incompatible. DMS needs SCT first to convert the schema.
- **Trap:** Database dump (Option B) requires downtime during export/import (violates near-zero downtime requirement).
- **Trap:** DataSync (Option D) is for file/object transfer, not database migration.
- **The Fix:** **Option C**. **SCT converts the Oracle schema** to PostgreSQL format first. Then **DMS with CDC** replicates existing data and captures ongoing changes, enabling near-zero downtime cutover.

---

### **Scenario 3: The "Multi-Account Governance" (SCPs)**

**The Situation:** A large enterprise uses AWS Organizations with 50 accounts. The security team requires that **no account can launch resources outside of us-east-1 and eu-west-1**. Some developers have administrator access in their accounts.

**The Options:**

A. Create IAM policies in each account to deny non-approved regions.

B. Apply a Service Control Policy (SCP) at the Organization root to deny all regions except us-east-1 and eu-west-1.

C. Use AWS Config rules to detect and auto-remediate resources in non-approved regions.

D. Use AWS Budgets to alert when spending occurs in non-approved regions.

**The Logic:**

- **Trap:** IAM policies (Option A) would need to be applied to every account and every user — not scalable with 50 accounts, and admins could remove them.
- **Trap:** Config rules (Option C) detect after the fact — resources are created first, then remediated. Not a preventive control.
- **Trap:** Budgets (Option D) only alert, they don't prevent launches.
- **The Fix:** **Option B**. **SCPs are guardrails** that restrict the maximum permissions for all accounts in an Organization. Even administrators cannot override SCPs. One policy at the root = 50 accounts governed. SCPs don't grant permissions — they restrict them.

---

### **Scenario 4: The "Lift and Shift" (MGN vs DMS)**

**The Situation:** A company wants to migrate **30 on-premises servers** (mix of Windows and Linux, running custom applications) to AWS EC2 with **minimal changes** to the applications. They want to replicate the servers continuously and perform a cutover during a maintenance window.

**The Options:**

A. Use AWS DMS to migrate the servers to EC2.

B. Manually create AMIs and launch EC2 instances from them.

C. Use AWS Application Migration Service (MGN) to replicate and cutover.

D. Use AWS DataSync to transfer the server data to EBS volumes.

**The Logic:**

- **Trap:** DMS (Option A) is for **database** migration, not server migration.
- **Trap:** Manual AMIs (Option B) require significant effort for 30 servers and don't support continuous replication.
- **Trap:** DataSync (Option D) transfers files to S3/EFS/FSx, not to EBS or EC2.
- **The Fix:** **Option C**. **Application Migration Service (MGN)** is purpose-built for lift-and-shift. Install the agent on each source server → continuous replication to AWS → cutover during maintenance window. Supports both Windows and Linux with minimal application changes.

---

### **Scenario 5: The "Cost Visibility" (Cost Explorer vs Budgets vs Compute Optimizer)**

**The Situation:** A company's AWS bill increased 40% last month. The CTO needs to: (1) understand **which services** drove the increase, (2) set up **automatic alerts** if spending exceeds $50,000 next month, and (3) identify **over-provisioned EC2 instances** to right-size.

**The Options:**

A. Use AWS Budgets for all three requirements.

B. Use Cost Explorer to analyze spending, Budgets for alerts, and Compute Optimizer for right-sizing.

C. Use CloudWatch metrics to track spending and set alarms.

D. Use AWS Trusted Advisor for all three requirements.

**The Logic:**

- **Trap:** Budgets (Option A) handles alerts but doesn't provide detailed spending analysis or right-sizing recommendations.
- **Trap:** CloudWatch (Option C) has billing metrics but not the detailed service breakdown or right-sizing capability.
- **Trap:** Trusted Advisor (Option D) provides some cost recommendations but not the detailed spending analysis or budget alerting.
- **The Fix:** **Option B**. Three tools, three jobs: **Cost Explorer** = analyze spending by service/region/tag (answers "what caused the 40% increase"). **Budgets** = alert when next month exceeds $50K. **Compute Optimizer** = ML-based recommendations to right-size over-provisioned EC2. Each tool serves a specific purpose.
