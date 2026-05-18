# Monitoring & Audit

### **SECTION 1: CLOUDWATCH (THE MONITORING SERVICE)**

*Metrics, Logs, Alarms, and Events.*

### **1. CloudWatch Overview**

> 🔧 *Like:* Prometheus + Grafana — metrics, logs, dashboards, and alerting.

- **The Rule:** Monitor AWS resources and applications via **metrics, logs, alarms, and dashboards**.
- **Scope:** Regional service.

---

### **2. CloudWatch Metrics**

**The Rule:** Time-series data points (CPU, Network, Disk, etc.).

**Built-In Metrics (AWS Services):**

- **EC2:** CPUUtilization, NetworkIn/Out, StatusCheckFailed (Every 5 minutes by default, 1 minute with Detailed Monitoring).
- **EBS:** VolumeReadOps, VolumeWriteOps, BurstBalance.
- **ELB:** RequestCount, TargetResponseTime, UnHealthyHostCount.
- **RDS:** DatabaseConnections, ReadLatency, FreeStorageSpace.

**Custom Metrics:**

- Application-level metrics via the PutMetricData API (queue depth, user logins, business metrics). Resolution: Standard 1 min, High Resolution 1 sec (higher cost). Dimensions are key-value filters (e.g., `InstanceId=i-1234`).
- **EC2 Memory/Disk:** NOT in default metrics (only hypervisor-level like CPU) — install the **CloudWatch Agent** to send memory/disk metrics.

**Exam Trigger:** "Monitor EC2 memory usage" → CloudWatch Agent (Custom Metrics).

---

### **3. CloudWatch Logs**

**The Rule:** Store, monitor, and analyze log files.

- **Sources:** EC2 (via CloudWatch Agent), Lambda (automatic), ECS/EKS, VPC Flow Logs, CloudTrail, Route 53 Query Logs.
- **Structure:** Log Group = container for log streams (e.g., `/aws/lambda/my-function`); Log Stream = log events from a single source.
- **Retention:** Default never expire; configurable 1 day – 10 years.
- **Log Insights:** Query logs with SQL-like syntax (e.g., "find all 5xx errors in last hour").

**Exam Trigger:** "Query application logs for errors" → CloudWatch Logs Insights.

---

### **4. CloudWatch Alarms**

**The Rule:** Trigger actions based on metric thresholds.

- **Actions:** SNS (notify via email/SMS/Lambda), Auto Scaling (scale EC2), EC2 Action (stop/terminate/reboot/recover).
- **States:** OK, ALARM, INSUFFICIENT_DATA.
- **Config:** metric, threshold, evaluation period, datapoints to alarm.
- **Composite Alarms:** Combine multiple alarms with AND/OR logic to reduce noise (e.g., alert only if CPU AND Network high).

**Exam Trigger:** "Alert when CPU > 80% for 5 minutes" → CloudWatch Alarm.

---

### **5. CloudWatch Events / EventBridge**

**The Rule:** Event-driven automation — react to AWS events or schedule tasks. **EventBridge = CloudWatch Events 2.0** (more features, 3rd-party SaaS integrations).

- **Sources:** AWS services (EC2 state change, S3 object created), custom apps (PutEvents API), SaaS partners (Datadog, Zendesk, PagerDuty).
- **Targets:** Lambda, SNS/SQS, Step Functions, ECS Task, CodePipeline, Kinesis.
- **Scheduled Events (cron/rate):** Run tasks on a schedule for backup automation, periodic Lambda, start/stop EC2.

**Exam Trigger:** "Run Lambda every hour" → EventBridge Scheduled Rule.

---

### **6. CloudWatch Dashboards**

**The Rule:** Visual metric display (graphs, numbers) — cross-region, cross-account, auto-refresh. For operations dashboards and real-time monitoring.

---

### **7. CloudWatch Agent (Deep Dive)**

**The Rule:** Collects **custom metrics** (memory, disk, swap) and **logs** (app/system logs) from EC2/on-prem servers. Install the agent, configure `config.json` for which metrics/logs to collect, and attach an IAM Role with send permissions.

- **Unified Agent** (metrics + logs, recommended) vs **Logs Agent** (logs only, legacy).

**Exam Trigger:** "Monitor EC2 disk usage" → CloudWatch Agent.

---

### **8. CloudWatch Logs Subscription Filters**

**The Rule:** Stream log data in **real-time** from CloudWatch Logs to a destination. Define a **filter pattern** on a Log Group; matching events stream to the destination.

- **Destinations:** Lambda (process/transform), Kinesis Data Streams (custom pipeline), Kinesis Data Firehose (to S3/Redshift/OpenSearch), OpenSearch (log analytics).
- Up to **2 subscription filters per Log Group** (use Kinesis Data Streams to fan out beyond that).

**Exam Trigger:** "Real-time log processing" or "Stream logs to OpenSearch" --> CloudWatch Logs Subscription Filter.

---

### **9. CloudWatch Metric Filters**

**The Rule:** Extract metric data from log events using **filter patterns** — turn log patterns into custom metrics, then alarm on them. Define a filter pattern on a Log Group (e.g., `"404"`, `"ERROR"`); CloudWatch creates a custom metric incremented on each match; create an alarm on it.

- **Use Cases:** Count 404s / failed logins / app error patterns → alarm above a threshold.

**Exam Trigger:** "Create alarm based on log pattern" or "Alert when error count in logs exceeds threshold" --> CloudWatch Metric Filter + Alarm.

---

### **10. CloudWatch Contributor Insights**

**The Rule:** Identify the **top-N contributors** to a metric using log data (e.g., top 10 IP addresses hitting your API, top accounts generating errors).

- **Real-time analysis** of log data using rules you define.
- Works with CloudWatch Logs and VPC Flow Logs.

**Exam Trigger:** "Find top contributors to high API traffic" or "Identify top IPs causing errors" --> CloudWatch Contributor Insights.

---

### **11. CloudWatch Synthetics**

**The Rule:** Create **Canary scripts** that monitor your APIs and websites on a schedule.

- Test endpoint **availability and latency** before real users notice issues.
- Scripts written in Node.js or Python (runs on Lambda under the hood).
- Can monitor URLs, REST APIs, and multi-step workflows.

**Exam Trigger:** "Proactively monitor API availability" or "Test website latency on a schedule" --> CloudWatch Synthetics Canaries.

---

### **SECTION 1.5: AWS X-RAY**

*Distributed tracing for microservices.*

### **1. X-Ray Overview**

- **The Rule:** Analyze and debug distributed apps — trace requests across **microservices** (Lambda, API Gateway, ECS, EC2, Elastic Beanstalk). The **Service Map** visualizes architecture, connections, latency, errors, and fault rates.
- **Capabilities:** End-to-end request tracing; segments per service + subsegments for external calls (DB, HTTP); annotations (indexed key-value pairs for filtering); groups (filter by annotation/error type).
- **Integration:** Lambda and API Gateway have built-in integration; install the **X-Ray Daemon** on EC2/ECS.

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

**The Rule:** Centralized store for configuration data and secrets. Organize parameters in paths (e.g., `/app/prod/db-password`). Integrates with Lambda, ECS, CloudFormation, CodeBuild.

- **Standard:** Free, max 10,000 parameters, 4 KB each. **Advanced:** Paid, max 100,000 parameters, 8 KB each, parameter policies (TTL, expiration notifications).

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

### **4. SSM Run Command**

**The Rule:** Execute commands on EC2/on-prem servers **remotely** (no SSH) — install software on many instances at once, restart services, collect inventory.

**Exam Trigger:** "Run script on 50 EC2 instances" → SSM Run Command.

---

### **5. SSM Patch Manager**

**The Rule:** Automate OS patching for EC2/on-prem servers — define a **Patch Baseline** (which patches), set a **Maintenance Window** (when), and Patch Manager applies them.

**Exam Trigger:** "Automate OS patching" → SSM Patch Manager.

---

### **SECTION 5: TRUSTED ADVISOR**

*Recommendations for cost, performance, security, fault tolerance.*

### **1. Trusted Advisor Overview**

- **The Rule:** Automated checks and recommendations across **5 pillars:** Cost Optimization (idle resources, unused RIs), Performance (high-utilization instances, EBS throughput), Security (open SG ports, IAM password policy, root MFA), Fault Tolerance (Multi-AZ RDS, empty ELBs, S3 versioning), Service Limits (approaching VPC/EIP/EC2 limits).
- **Results:** Green (OK), Yellow (investigate), Red (action recommended).
- **Access:** Basic/Developer Support → 7 core checks (free); Business/Enterprise → all checks + API + CloudWatch.

**Exam Trigger:** "Identify underutilized EC2 instances" → Trusted Advisor (Cost Optimization).

---

### **SECTION 6: PERSONAL HEALTH DASHBOARD**

*Service health events affecting your resources.*

### **1. Personal Health Dashboard**

- **The Rule:** Alerts about AWS events impacting **your** resources (not just general AWS status) — e.g., scheduled EC2 maintenance, RDS performance degradation, AMI security vulnerabilities. Integrates with EventBridge to automate responses (failover, SNS alert).

**Exam Trigger:** "Automated notification when AWS event affects your resources" → Personal Health Dashboard + EventBridge.

---

### **Exam Summary Cheat Sheet (Memorize This)**

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

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Memory Alert"**

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

### **Scenario 2: The "Compliance Audit"**

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

### **Scenario 3: The "SSH-Less Access"**

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

### **Scenario 4: The "Deleted Resource Mystery"**

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

### **Scenario 5: The "Auto-Remediation"**

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