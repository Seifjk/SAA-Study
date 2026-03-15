# Monitoring & Audit

### **SECTION 1: CLOUDWATCH (THE MONITORING SERVICE)**

*Metrics, Logs, Alarms, and Events.*

### **1. CloudWatch Overview**

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

- **The Rule:** Application-level metrics (PutMetricData API). Examples: Queue depth, User logins, Business metrics.
- **Resolution:**
    - Standard: 1 minute.
    - High Resolution: 1 second (Higher cost).
- **Dimensions:** Key-value pairs to filter metrics (e.g., `InstanceId=i-1234`, `Environment=Prod`).

**EC2 Memory/Disk Utilization:**

- **NOT** included in default metrics (Only hypervisor-level metrics like CPU).
- **Solution:** Install **CloudWatch Agent** on EC2 → Sends custom metrics (Memory, Disk).

**Exam Trigger:** "Monitor EC2 memory usage" → CloudWatch Agent (Custom Metrics).

---

### **3. CloudWatch Logs**

**The Rule:** Store, monitor, and analyze log files.

**Sources:**

- EC2 (via CloudWatch Agent)
- Lambda (Automatic)
- ECS/EKS
- VPC Flow Logs
- CloudTrail
- Route 53 Query Logs

**Log Structure:**

- **Log Group:** Container for log streams (e.g., `/aws/lambda/my-function`).
- **Log Stream:** Sequence of log events from a single source (e.g., Instance ID).

**Retention:**

- Default: Never expire.
- Configurable: 1 day to 10 years (Or never).

**Log Insights:**

- **The Rule:** Query logs using SQL-like syntax.
- **Use Case:** Analyze logs (e.g., "Find all 5xx errors in last hour").

**Exam Trigger:** "Query application logs for errors" → CloudWatch Logs Insights.

---

### **4. CloudWatch Alarms**

**The Rule:** Trigger actions based on metric thresholds.

**Actions:**

- **SNS:** Send notification (Email, SMS, Lambda trigger).
- **Auto Scaling:** Scale EC2 instances.
- **EC2 Action:** Stop, Terminate, Reboot, Recover instance.

**Alarm States:**

- **OK:** Metric within threshold.
- **ALARM:** Metric breached threshold.
- **INSUFFICIENT_DATA:** Not enough data.

**Configuration:**

- **Metric:** Which metric to monitor (e.g., CPU).
- **Threshold:** Value to trigger (e.g., > 80%).
- **Evaluation Period:** How many data points to evaluate (e.g., 3 out of 5 periods).
- **Datapoints to Alarm:** How many breaches trigger alarm.

**Composite Alarms:**

- **The Rule:** Combine multiple alarms with AND/OR logic.
- **Use Case:** Reduce alarm noise (Alert only if CPU **AND** Network high).

**Exam Trigger:** "Alert when CPU > 80% for 5 minutes" → CloudWatch Alarm.

---

### **5. CloudWatch Events / EventBridge**

**The Rule:** Event-driven automation. React to AWS events or schedule tasks.

**EventBridge = CloudWatch Events 2.0** (More features, supports 3rd-party SaaS integrations).

**Event Sources:**

- **AWS Services:** EC2 state change, S3 object created, Auto Scaling action.
- **Custom Applications:** PutEvents API.
- **SaaS Partners:** Datadog, Zendesk, PagerDuty.

**Targets:**

- Lambda
- SNS/SQS
- Step Functions
- ECS Task
- CodePipeline
- Kinesis

**Scheduled Events (Cron/Rate):**

- **The Rule:** Run tasks on schedule (e.g., "Every day at 9 AM" or "Every 5 minutes").
- **Use Case:** Backup automation, Periodic Lambda invocation, Start/Stop EC2 instances.

**Exam Trigger:** "Run Lambda every hour" → EventBridge Scheduled Rule.

---

### **6. CloudWatch Dashboards**

**The Rule:** Visual representation of metrics (Graphs, numbers).

**Features:**

- Cross-region (View metrics from multiple regions).
- Cross-account (Aggregate metrics from multiple accounts).
- Automatic refresh.

**Use Case:** Operations dashboard, Real-time monitoring.

---

### **7. CloudWatch Agent (Deep Dive)**

**The Rule:** Collect **custom metrics** and **logs** from EC2/On-Prem servers.

**What It Collects:**

- System-level metrics (Memory, Disk, Swap).
- Logs (Application logs, System logs).

**Installation:**

1. Install CloudWatch Agent on instance.
2. Configure `config.json` (Which metrics/logs to collect).
3. Attach IAM Role to instance (Permissions to send metrics/logs).

**Unified Agent vs. Logs Agent:**

- **Unified Agent:** Collects metrics + logs (Recommended).
- **Logs Agent:** Only logs (Legacy).

**Exam Trigger:** "Monitor EC2 disk usage" → CloudWatch Agent.

---

### **SECTION 2: CLOUDTRAIL (THE AUDIT LOG)**

*Track API calls and user activity.*

### **1. CloudTrail Overview**

- **The Rule:** Records **API calls** made in your AWS account (Who did what, when, from where).
- **Scope:** Regional by default, Can enable for all regions.
- **Audit Use Cases:** Security analysis, Compliance, Troubleshooting.

**What It Logs:**

- **Management Events:** Control plane actions (CreateInstance, DeleteBucket, AttachUserPolicy).
- **Data Events:** Data plane actions (S3 GetObject, Lambda Invoke, DynamoDB PutItem) - High volume, optional.
- **Insights Events:** Detect unusual API activity (ML-based anomaly detection).

---

### **2. CloudTrail Structure**

**Trail:**

- **The Rule:** Configuration that defines which events to log and where to store them.
- **Storage:** S3 bucket (Logs stored as JSON).
- **Optional:** Send to CloudWatch Logs for real-time analysis.

**Event Retention:**

- **CloudTrail Console:** 90 days (Free).
- **S3 Bucket:** Forever (Until you delete). Use S3 Lifecycle policies to archive to Glacier.

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

**The Rule:** Validate that log files haven't been tampered with (Digest files).

**How It Works:**

- CloudTrail generates **digest files** (Hash of log files).
- Use digest to verify log integrity.

**Exam Trigger:** "Ensure audit logs haven't been modified" → Enable Log File Validation.

---

### **SECTION 3: AWS CONFIG (COMPLIANCE & CONFIGURATION TRACKING)**

*Track resource configuration changes.*

### **1. AWS Config Overview**

- **The Rule:** Record **configuration changes** over time (Not API calls like CloudTrail, but resource state).
- **Use Cases:** Compliance auditing, Resource inventory, Configuration history.

**What It Tracks:**

- Resource configurations (e.g., "Is S3 bucket public?", "Is encryption enabled?").
- Configuration changes (e.g., "Security Group rule added at 2:30 PM").
- Resource relationships (e.g., "This EC2 instance uses this Security Group").

---

### **2. Config Rules**

**The Rule:** Evaluate resource compliance against rules.

**Types:**

- **AWS Managed Rules:** Pre-built (e.g., `s3-bucket-public-read-prohibited`, `encrypted-volumes`).
- **Custom Rules:** Write Lambda function to evaluate compliance.

**Triggers:**

- **Configuration Change:** Evaluate when resource changes.
- **Periodic:** Evaluate on schedule (e.g., Every 24 hours).

**Remediation:**

- **Manual:** View non-compliant resources and fix manually.
- **Automatic:** Trigger **SSM Automation Document** or **Lambda** to auto-remediate.

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

**The Rule:** Store configuration data and secrets (Centralized).

**Types:**

- **Standard:** Free. Max 10,000 parameters. Max 4 KB per parameter.
- **Advanced:** Paid. Max 100,000 parameters. Max 8 KB per parameter. Parameter policies (TTL, expiration notifications).

**Hierarchy:**

- Organize parameters in paths (e.g., `/app/prod/db-password`).

**Integration:** Lambda, ECS, CloudFormation, CodeBuild.

**Parameter Store vs. Secrets Manager:**

| **Feature** | **Parameter Store** | **Secrets Manager** |
| --- | --- | --- |
| **Auto-Rotation** | No (Manual via Lambda) | **Yes** (Built-in for RDS) |
| **Cost** | Free (Standard), Paid (Advanced) | $0.40/secret/month |
| **Use Case** | App config, Free secrets | DB passwords, Auto-rotation |

**Exam Trigger:** "Store app config for free" → Parameter Store. "Auto-rotate DB password" → Secrets Manager.

---

### **3. SSM Session Manager**

**The Rule:** Connect to EC2 instances **without SSH/RDP** (Browser-based shell or CLI).

**Benefits:**

- No open SSH ports (Security Group doesn't need port 22).
- No Bastion Host needed.
- Session logs sent to S3 or CloudWatch Logs (Audit trail).
- IAM-based access control.

**Requirements:**

- EC2 must have **SSM Agent** installed (Pre-installed on Amazon Linux 2, Ubuntu, Windows).
- EC2 must have **IAM Role** with SSM permissions.

**Exam Trigger:** "Connect to EC2 without SSH key or open port 22" → Session Manager.

---

### **4. SSM Run Command**

**The Rule:** Execute commands on EC2/on-prem servers **remotely** (No SSH).

**Use Cases:**

- Install software on 100 instances simultaneously.
- Restart services.
- Collect system inventory.

**Exam Trigger:** "Run script on 50 EC2 instances" → SSM Run Command.

---

### **5. SSM Patch Manager**

**The Rule:** Automate OS patching for EC2 and on-prem servers.

**How It Works:**

- Define **Patch Baseline** (Which patches to install).
- Create **Maintenance Window** (When to patch).
- Patch Manager installs patches during window.

**Exam Trigger:** "Automate OS patching" → SSM Patch Manager.

---

### **SECTION 5: TRUSTED ADVISOR**

*Recommendations for cost, performance, security, fault tolerance.*

### **1. Trusted Advisor Overview**

- **The Rule:** Automated checks and recommendations across **5 pillars**.

**Pillars:**

1. **Cost Optimization:** Idle resources, Unused Reserved Instances, Low-utilization EC2.
2. **Performance:** High-utilization instances, EBS throughput optimization.
3. **Security:** Open Security Group ports, IAM password policy, MFA on root account.
4. **Fault Tolerance:** Multi-AZ RDS, ELB without instances, S3 bucket versioning.
5. **Service Limits:** Approaching service limits (VPC, EIP, EC2 instances).

**Check Results:**

- **Green:** No problem detected.
- **Yellow:** Investigation recommended.
- **Red:** Action recommended.

**Access Levels:**

- **Basic/Developer Support:** 7 core checks (Free).
- **Business/Enterprise Support:** **All checks** + API access + CloudWatch integration.

**Exam Trigger:** "Identify underutilized EC2 instances" → Trusted Advisor (Cost Optimization).

---

### **SECTION 6: PERSONAL HEALTH DASHBOARD**

*Service health events affecting your resources.*

### **1. Personal Health Dashboard**

- **The Rule:** Alerts about AWS events impacting **your** resources (Not just general AWS status).

**Examples:**

- "Scheduled maintenance on your EC2 instance in us-east-1."
- "Performance degradation on your RDS instance."
- "Security vulnerability in AMI you're using."

**Integration:** EventBridge (Automate responses, e.g., Failover, Send SNS alert).

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

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Memory Alert" (CloudWatch Agent)**

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

### **Scenario 2: The "Compliance Audit" (AWS Config)**

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

### **Scenario 3: The "SSH-Less Access" (Session Manager)**

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

### **Scenario 4: The "Deleted Resource Mystery" (CloudTrail)**

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

### **Scenario 5: The "Auto-Remediation" (Config Rule + Remediation)**

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
