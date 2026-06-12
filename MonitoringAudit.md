# Monitoring & Audit

> **📌 Scope note:** AWS tests these with *"determine WHEN to use"* — the whole skill here is **which tool answers which question** (audit vs config-state vs metrics vs network vs tracing). You don't operate the agent or write patch baselines. If it's not here, the exam doesn't ask it.

### **SECTION 1: CLOUDWATCH (THE MONITORING SERVICE)**

*Metrics, Logs, Alarms, and Events.*

### **1. CloudWatch Overview**

- **The Rule:** Monitor AWS resources and applications via **metrics, logs, alarms, and dashboards**. Regional service.

---

### **2. CloudWatch Metrics**

**The Rule:** Time-series data points (CPU, Network, etc.). AWS services publish built-in metrics automatically; you can push **Custom Metrics** for app/business data.

- **The one tested gotcha:** EC2 default metrics are hypervisor-level (CPU, network, disk-ops) and **do NOT include memory or in-guest disk usage** — install the **CloudWatch Agent** to get those.
- **Basic vs Detailed Monitoring:** EC2 sends metrics every **5 min** by default (Basic). Enable **Detailed Monitoring** (paid) for **1-min** granularity — needed when ASG must scale faster than 5-min reaction time.

**Exam Trigger:** "Monitor EC2 memory/disk usage" → CloudWatch Agent (Custom Metrics). "Need EC2 metrics every 1 minute" → enable Detailed Monitoring.

---

### **3. CloudWatch Logs**

**The Rule:** Store, monitor, and analyze log files (from EC2 via the Agent, Lambda automatically, VPC Flow Logs, CloudTrail, etc.). Retention is configurable (default never expire).

- **Logs Insights:** Query logs with a SQL-like syntax (e.g., "find all 5xx errors in the last hour").

**Exam Trigger:** "Query application logs for errors" → CloudWatch Logs Insights.

---

### **4. CloudWatch Alarms**

**The Rule:** Trigger actions based on metric thresholds.

- **Actions:** SNS (notify via email/SMS/Lambda), Auto Scaling (scale EC2), EC2 Action (stop/terminate/reboot/recover).
- **States:** OK, ALARM, INSUFFICIENT_DATA.
- **Config:** metric, threshold, evaluation period, datapoints to alarm.
- **Composite Alarms:** Combine multiple alarms with AND/OR logic to reduce noise (e.g., alert only if CPU AND Network high).
- **EC2 Recover (tested):** An alarm on the **System Status Check** (host/hardware failure, not your OS) with the **recover** action auto-moves the instance to new hardware — **same Instance ID, Private IP, and Elastic IP** preserved. (Instance Status Check = your OS/config problem → recover won't help; reboot or fix the instance.)

**Exam Trigger:** "Alert when CPU > 80% for 5 minutes" → CloudWatch Alarm. "Auto-recover EC2 on underlying hardware failure" → CloudWatch Alarm on System Status Check → Recover action.

---

### **5. CloudWatch Events / EventBridge**

**The Rule:** Event-driven automation — react to AWS events (e.g. EC2 state change, S3 object created) or run tasks on a schedule. **EventBridge = CloudWatch Events 2.0** (adds 3rd-party SaaS sources + a schema registry). Targets include Lambda, SNS/SQS, Step Functions, ECS tasks.

- **Scheduled Events (cron/rate):** run tasks on a schedule — periodic Lambda, scheduled start/stop of EC2, backup automation.

**Exam Trigger:** "Run Lambda every hour" → EventBridge Scheduled Rule. "React to an AWS service event" → EventBridge Rule.

---

### **6. CloudWatch Dashboards**

**The Rule:** Visual metric display (graphs, numbers) — cross-region, cross-account, auto-refresh. For operations dashboards and real-time monitoring.

---

### **7. CloudWatch Log Features (Recognize the Names → Pick the Tool)**

These appear on the exam as one-line "which feature" triggers. You don't configure them — you recognize the name:

- **Subscription Filter:** stream matching log events in **real-time** to Lambda / Kinesis / Firehose / OpenSearch. **Trigger:** "real-time log processing" / "stream logs to OpenSearch."
- **Metric Filter:** turn a log pattern (e.g. `"ERROR"`, `"404"`) into a custom metric, then **alarm on it**. **Trigger:** "create an alarm based on a log pattern."
- **Contributor Insights:** find the **top-N contributors** to a metric (e.g. top IPs hitting your API). **Trigger:** "identify top IPs causing errors."
- **Synthetics (Canaries):** scheduled scripts that **proactively** test endpoint availability/latency before users notice. **Trigger:** "proactively monitor API availability."

---

### **SECTION 1.5: AWS X-RAY**

*Distributed tracing for microservices.*

### **1. X-Ray Overview**

- **The Rule:** Analyze and debug distributed apps — trace requests end-to-end across **microservices** (Lambda, API Gateway, ECS, EC2). The **Service Map** visualizes architecture, connections, latency, errors, and fault rates. This is the SAA answer whenever a question is about *finding latency/bottlenecks across services*.

**Exam Triggers:**

- "Debug latency in distributed application" --> X-Ray.
- "Trace requests across microservices" --> X-Ray.
- "Visualize application architecture and dependencies" --> X-Ray Service Map.
- "Identify performance bottleneck in microservices" --> X-Ray.

---

### **SECTION 2: CLOUDTRAIL (THE AUDIT LOG)**

*Track API calls and user activity.*

### **1. CloudTrail Overview**

- **The Rule:** Records **API calls** in your account (who did what, when, from where). Regional by default, can enable all regions. For security analysis, compliance, troubleshooting.
- **What it logs:** Management Events (control plane — CreateInstance, DeleteBucket); Data Events (data plane — S3 GetObject, Lambda Invoke; high volume, optional); Insights Events (ML-based anomaly detection of unusual API activity).

---

### **2. CloudTrail Structure**

- **Trail:** Configuration defining which events to log and where to store them — logs stored as JSON in an S3 bucket; optionally sent to CloudWatch Logs for real-time analysis.
- **Retention:** CloudTrail console 90 days (free); S3 bucket forever (use Lifecycle policies to archive to Glacier).

**Exam Trigger:** "Audit who deleted an S3 object" → Enable CloudTrail Data Events for S3.

---

### **3. CloudTrail vs. CloudWatch Logs vs. VPC Flow Logs**

| **Service** | **Purpose** | **What It Logs** | **Exam Trigger** |
| --- | --- | --- | --- |
| **CloudTrail** | **Audit (Who did what)** | API calls (CreateInstance, DeleteBucket) | "Who deleted the S3 object?" |
| **CloudWatch Logs** | **Application logs** | Custom logs (App errors, Lambda logs) | "Debug application errors" |
| **VPC Flow Logs** | **Network traffic** | IP traffic (Source IP, Dest IP, Port, Action) | "Why is traffic blocked?" |

---

### **4. CloudTrail Log File Integrity**

**The Rule:** Validate that log files haven't been tampered with — CloudTrail generates **digest files** (hashes of log files) used to verify integrity.

**Exam Trigger:** "Ensure audit logs haven't been modified" → Enable Log File Validation.

---

### **SECTION 3: AWS CONFIG (COMPLIANCE & CONFIGURATION TRACKING)**

*Track resource configuration changes.*

### **1. AWS Config Overview**

- **The Rule:** Records **configuration changes** over time (resource state, not API calls). For compliance auditing, resource inventory, configuration history.
- **Tracks:** Resource configurations ("Is S3 bucket public?"), configuration changes ("SG rule added at 2:30 PM"), and resource relationships ("This EC2 uses this SG").

---

### **2. Config Rules**

**The Rule:** Evaluate resource compliance against rules.

- **Types:** AWS Managed Rules (pre-built, e.g. `s3-bucket-public-read-prohibited`, `encrypted-volumes`) or Custom Rules (Lambda-based).
- **Triggers:** on configuration change, or periodic (e.g., every 24 hours).
- **Remediation:** manual fix, or automatic via **SSM Automation Document** or **Lambda**.

**Exam Trigger:** "Ensure all S3 buckets are private, Auto-remediate if not" → Config Rule + Remediation Action.

---

### **3. Config vs. CloudTrail**

| **Feature** | **CloudTrail** | **AWS Config** |
| --- | --- | --- |
| **Purpose** | Audit API calls (Who did what) | Track configuration changes (Resource state) |
| **Question It Answers** | "Who deleted this instance?" | "Was this bucket public yesterday?" |
| **Focus** | **Actions** | **State** |
| **Use Case** | Security audit, Troubleshooting | Compliance, Configuration history |

**Exam Pattern:** Use **both** together. CloudTrail = Who made the change. Config = What was changed.

---

### **SECTION 4: SYSTEMS MANAGER (SSM)**

*Operational management for AWS resources.*

### **1. Systems Manager Overview**

- **The Rule:** Manage EC2 and on-prem servers at scale (No SSH needed).
- **Components:** Parameter Store, Session Manager, Run Command, Patch Manager, Automation.

---

### **2. SSM Parameter Store**

**The Rule:** Centralized store for configuration data and secrets, organized in paths (e.g. `/app/prod/db-password`). Free Standard tier; the decision that's tested is Parameter Store **vs Secrets Manager** (below).

**Parameter Store vs. Secrets Manager:**

| **Feature** | **Parameter Store** | **Secrets Manager** |
| --- | --- | --- |
| **Auto-Rotation** | No (Manual via Lambda) | **Yes** (Built-in for RDS) |
| **Cost** | Free (Standard), Paid (Advanced) | $0.40/secret/month |
| **Use Case** | App config, Free secrets | DB passwords, Auto-rotation |

**Exam Trigger:** "Store app config for free" → Parameter Store. "Auto-rotate DB password" → Secrets Manager.

---

### **3. SSM Session Manager**

**The Rule:** Connect to EC2 instances **without SSH/RDP** (browser shell or CLI). No open port 22, no Bastion Host; session logs to S3/CloudWatch Logs for audit; IAM-based access control.

- **Requirements:** EC2 needs the **SSM Agent** (pre-installed on Amazon Linux 2, Ubuntu, Windows) and an **IAM Role** with SSM permissions.

**Exam Trigger:** "Connect to EC2 without SSH key or open port 22" → Session Manager.

---

### **4. SSM Run Command & Patch Manager**

- **Run Command:** execute a command/script across many EC2/on-prem servers at once, no SSH. **Trigger:** "run a script on 50 EC2 instances" → Run Command.
- **Patch Manager:** automate OS patching across your fleet on a schedule. **Trigger:** "automate OS patching" → Patch Manager.

---

### **SECTION 5: TRUSTED ADVISOR**

*Recommendations for cost, performance, security, fault tolerance.*

### **1. Trusted Advisor Overview**

- **The Rule:** Automated checks and recommendations across **5 pillars:** Cost Optimization (idle resources, unused RIs), Performance (high-utilization instances, EBS throughput), Security (open SG ports, IAM password policy, root MFA), Fault Tolerance (Multi-AZ RDS, empty ELBs, S3 versioning), Service Limits (approaching VPC/EIP/EC2 limits).
- **Results:** Green (OK), Yellow (investigate), Red (action recommended).
- **Access:** Basic/Developer Support → 7 core checks (free); Business/Enterprise → all checks + API + CloudWatch.

**Exam Trigger:** "Identify underutilized EC2 instances" → Trusted Advisor (Cost Optimization).

---

### **SECTION 5.5: COMPUTE OPTIMIZER**

*ML-based right-sizing recommendations.*

- **The Rule:** Analyzes historical **utilization metrics** and recommends the **optimal resource size/type** for EC2 instances, EC2 Auto Scaling groups, EBS volumes, and Lambda functions. Tells you "downsize this m5.xlarge → m5.large" or "this Lambda needs more memory."
- **Trusted Advisor vs Compute Optimizer (the tested trap):** Trusted Advisor = **broad** checks across 5 pillars, flags *idle/underutilized* resources. Compute Optimizer = **deep, ML-driven right-sizing** — the specific better instance type/size. When the question asks for a *specific sizing recommendation based on utilization history*, it's Compute Optimizer.

**Exam Trigger:** "Right-size EC2/EBS/Lambda based on utilization history" → Compute Optimizer. "Specific recommendation for a better instance type" → Compute Optimizer.

---

### **SECTION 6: PERSONAL HEALTH DASHBOARD**

*Service health events affecting your resources.*

### **1. Personal Health Dashboard**

- **The Rule:** Alerts about AWS events impacting **your** resources (not just general AWS status) — e.g., scheduled EC2 maintenance, RDS performance degradation, AMI security vulnerabilities. Integrates with EventBridge to automate responses (failover, SNS alert).

**Exam Trigger:** "Automated notification when AWS event affects your resources" → Personal Health Dashboard + EventBridge.

---

### **Exam Summary Cheat Sheet — Practice (Fill In Yourself)**

1. **Monitor EC2 memory usage?** →
2. **Alert when CPU > 80%?** →
3. **Run Lambda every hour?** →
4. **Who deleted S3 object?** →
5. **Was S3 bucket public yesterday?** →
6. **Ensure all S3 buckets are private, Auto-remediate?** →
7. **Store app config for free?** →
8. **Auto-rotate DB password?** →
9. **Connect to EC2 without SSH?** →
10. **Run script on 50 instances?** →
11. **Automate OS patching?** →
12. **Identify underutilized EC2?** →
13. **Query logs for errors?** →
14. **Audit API calls?** →
15. **Track configuration changes?** →
16. **Debug latency in distributed app?** →
17. **Trace requests across microservices?** →
18. **Stream logs to OpenSearch in real-time?** →
19. **Create alarm based on log pattern?** →
20. **Top contributors to metrics?** →
21. **Monitor API availability?** →
22. **Right-size EC2/Lambda from utilization history?** →
23. **Auto-recover EC2 on hardware failure?** →

---

### **Exam Summary Cheat Sheet — Answer Key**

1. **Monitor EC2 memory usage?** → CloudWatch Agent (Custom Metrics).
2. **Alert when CPU > 80%?** → CloudWatch Alarm.
3. **Run Lambda every hour?** → EventBridge Scheduled Rule.
4. **Who deleted S3 object?** → CloudTrail (Enable Data Events).
5. **Was S3 bucket public yesterday?** → AWS Config.
6. **Ensure all S3 buckets are private, Auto-remediate?** → Config Rule + Remediation.
7. **Store app config for free?** → SSM Parameter Store.
8. **Auto-rotate DB password?** → Secrets Manager.
9. **Connect to EC2 without SSH?** → SSM Session Manager.
10. **Run script on 50 instances?** → SSM Run Command.
11. **Automate OS patching?** → SSM Patch Manager.
12. **Identify underutilized EC2?** → Trusted Advisor.
13. **Query logs for errors?** → CloudWatch Logs Insights.
14. **Audit API calls?** → CloudTrail.
15. **Track configuration changes?** → AWS Config.
16. **Debug latency in distributed app?** → X-Ray.
17. **Trace requests across microservices?** → X-Ray Service Map.
18. **Stream logs to OpenSearch in real-time?** → CloudWatch Logs Subscription Filter.
19. **Create alarm based on log pattern?** → CloudWatch Metric Filter + Alarm.
20. **Top contributors to metrics?** → CloudWatch Contributor Insights.
21. **Monitor API availability?** → CloudWatch Synthetics Canaries.
22. **Right-size EC2/Lambda from utilization history?** → Compute Optimizer.
23. **Auto-recover EC2 on hardware failure?** → CloudWatch Alarm on System Status Check → Recover action.

---

# **REAL EXAM SCENARIOS**

### **Scenario 1**

**The Situation:** An EC2 instance runs a Java application with a memory leak. The instance becomes unresponsive when memory usage exceeds 90%. CloudWatch Alarms are configured for CPU, but the CPU stays low even when the application fails.

**The Options:**

A. Monitor CPU Utilization CloudWatch metric and create an alarm.

B. Install CloudWatch Agent to send memory metrics, then create an alarm for memory > 90%.

C. Use CloudTrail to monitor memory usage.

D. Use VPC Flow Logs to detect the issue.

**The Logic:**

- **Trap:** CPU metric (Option A) already exists but doesn't help (CPU is low, memory is the problem).
- **Trap:** CloudTrail (Option C) logs API calls, not system metrics.
- **Trap:** VPC Flow Logs (Option D) track network traffic, not memory.
- **The Fix:** **Option B**. EC2 default metrics **do not include memory or disk**. Install **CloudWatch Agent** to send custom memory metrics. Create CloudWatch Alarm: "Alert when Memory > 90% for 5 minutes." Agent runs on instance, sends metrics every minute.

---

### **Scenario 2**

**The Situation:** A company must prove to auditors that **all S3 buckets have been private** for the last 6 months. They need historical evidence showing no buckets were ever public during this period.

**The Options:**

A. Check current S3 bucket policies manually.

B. Use CloudTrail to search for `PutBucketPolicy` API calls.

C. Use AWS Config to view configuration timeline for all S3 buckets.

D. Use Trusted Advisor to check for public buckets.

**The Logic:**

- **Trap:** Manual check (Option A) shows **current** state, not historical.
- **Trap:** CloudTrail (Option B) shows **who made changes** but not the actual configuration state over time.
- **Trap:** Trusted Advisor (Option D) shows current recommendations, not historical compliance.
- **The Fix:** **Option C**. **AWS Config** tracks resource configuration over time. Config Timeline shows: "On March 5, Bucket X was made public. On March 6, it was made private again." Config Rules evaluate compliance continuously. Historical evidence for auditors.

---

### **Scenario 3**

**The Situation:** A security policy requires **no open SSH ports** (port 22) on EC2 instances. Developers need to troubleshoot instances by running commands remotely. The current solution uses a Bastion Host in a public subnet with port 22 open.

**The Options:**

A. Continue using Bastion Host but restrict source IPs.

B. Use SSM Session Manager to connect to instances without SSH.

C. Use VPC Peering to access instances.

D. Use EC2 Instance Connect.

**The Logic:**

- **Trap:** Bastion Host (Option A) still requires port 22 open (Violates policy).
- **Trap:** VPC Peering (Option C) doesn't solve remote access (Still need SSH or other method).
- **Trap:** EC2 Instance Connect (Option D) still uses port 22 (Simplified SSH, but port open).
- **The Fix:** **Option B**. **SSM Session Manager** connects via AWS Systems Manager Agent (No port 22 needed). Security Group doesn't need inbound rules. Session logs sent to CloudWatch/S3 (Audit trail). IAM controls access. Zero open ports.

---

### **Scenario 4**

**The Situation:** A production RDS instance was deleted at 3:47 AM. No one on the team admits to deleting it. The team needs to identify **who** deleted the instance and **from which IP address**.

**The Options:**

A. Use AWS Config to view RDS configuration history.

B. Use CloudWatch Logs to search for deletion events.

C. Use CloudTrail to search for `DeleteDBInstance` API call.

D. Use Trusted Advisor to identify the issue.

**The Logic:**

- **Trap:** AWS Config (Option A) shows **what changed** (Instance was deleted), but not **who** did it.
- **Trap:** CloudWatch (Option B) logs application logs, not AWS API calls.
- **Trap:** Trusted Advisor (Option D) provides recommendations, not audit logs.
- **The Fix:** **Option C**. **CloudTrail** logs all API calls. Search CloudTrail for `DeleteDBInstance` event at ~3:47 AM. Event shows:
    - **User:** IAM user or role.
    - **Source IP:** Where request came from.
    - **User Agent:** SDK/CLI version.
    - **Session context:** If assumed role, who assumed it.
    Exactly designed for this.

---

### **Scenario 5**

**The Situation:** Security policy requires **all EBS volumes to be encrypted**. Developers occasionally launch unencrypted volumes by mistake. The security team manually detects and fixes these violations weekly. They want **automatic detection and remediation**.

**The Options:**

A. Use CloudWatch Alarm to alert when unencrypted volume detected.

B. Use CloudTrail to detect `CreateVolume` API calls and alert.

C. Use AWS Config Rule `encrypted-volumes` with automatic remediation to delete unencrypted volumes.

D. Use Trusted Advisor to identify unencrypted volumes.

**The Logic:**

- **Trap:** CloudWatch Alarms (Option A) monitor metrics, not resource configurations.
- **Trap:** CloudTrail (Option B) shows API calls but doesn't evaluate compliance or remediate.
- **Trap:** Trusted Advisor (Option D) identifies issues but doesn't auto-remediate.
- **The Fix:** **Option C**. **AWS Config Rule** `encrypted-volumes` evaluates all EBS volumes. If unencrypted volume detected → Mark as **non-compliant**. Configure **Automatic Remediation** (SSM Automation Document or Lambda) to delete/detach unencrypted volume or create snapshot and replace with encrypted version. Real-time compliance enforcement.
