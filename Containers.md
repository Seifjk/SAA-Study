# Containers

### **SECTION 1: ECR (ELASTIC CONTAINER REGISTRY)**

*Docker Hub but AWS-managed.*

### **1. ECR Overview**

- **The Rule:** Fully managed Docker container registry.
- **Storage:** Stored in S3 (AWS-managed).
- **Integration:** Native integration with ECS, EKS, Lambda.
- **Security:** IAM-based access control. Images encrypted at rest (KMS). Scan for vulnerabilities.

**ECR Public vs. Private:**

- **Private:** Accessible only with IAM permissions (Default).
- **Public:** Publicly accessible (Like Docker Hub).

**Exam Trigger:** "Store Docker images privately in AWS" → ECR.

---

### **SECTION 2: ECS (ELASTIC CONTAINER SERVICE)**

*AWS-native container orchestration (Like Kubernetes, but simpler).*

### **1. ECS Overview**

- **The Rule:** Run Docker containers on AWS.
- **Cluster:** Logical grouping of tasks/services.
- **Task Definition:** Blueprint (Which image, CPU, Memory, Networking, IAM Role).
- **Task:** Running instance of Task Definition (Ephemeral).
- **Service:** Ensures N tasks always running (Long-lived). Integrates with ELB.

---

### **2. Launch Types (How Containers Run)**

### **A. EC2 Launch Type (The Managed Fleet)**

**The Rule:** You manage EC2 instances. ECS manages container placement.

**How It Works:**

- Create ECS Cluster.
- Launch EC2 instances with **ECS Agent** (AMI: Amazon ECS-Optimized AMI).
- ECS schedules tasks on instances.

**Responsibilities:**

- **You Manage:** EC2 instances (Patching, Scaling, Instance Types).
- **AWS Manages:** Container orchestration (Placement, Scheduling).

**Pricing:** Pay for EC2 instances (No extra ECS charge).

**Use Cases:**

- Need control over instances.
- Cost optimization with Reserved/Spot Instances.
- Steady-state workloads.

**Exam Trigger:** "Run containers on EC2 with Reserved Instances" → ECS EC2 Launch Type.

---

### **B. Fargate Launch Type (The Serverless)**

**The Rule:** AWS manages infrastructure. You only define task (CPU/Memory). No EC2 to manage.

**How It Works:**

- Create Task Definition (Specify CPU, Memory).
- Run Task → AWS provisions infrastructure automatically.

**Responsibilities:**

- **You Manage:** Task Definition (Image, CPU, Memory).
- **AWS Manages:** Infrastructure (Servers, Scaling, Patching).

**Pricing:** Pay per vCPU/GB per second (More expensive per task, but no idle instance cost).

**Use Cases:**

- Don't want to manage instances.
- Variable workloads (Scale to zero).
- Microservices.

**Exam Trigger:** "Run containers without managing servers" → Fargate.

---

### **ECS EC2 vs. Fargate Comparison**

| **Feature** | **ECS EC2 Launch Type** | **Fargate** |
| --- | --- | --- |
| **Infrastructure** | You manage EC2 instances | AWS manages (Serverless) |
| **Pricing** | Pay for EC2 (Running instances) | Pay per task (vCPU/Memory) |
| **Control** | Full control (Instance type, AMI, Storage) | Limited (Only task config) |
| **Scaling** | Manual/ASG for EC2 + ECS Service scaling | Auto-scales tasks |
| **Cost Optimization** | Reserved/Spot Instances | No idle cost (Pay per task) |
| **Use Case** | Predictable workload, Cost control | Unpredictable, No ops |

---

### **3. ECS Components Deep Dive**

### **A. Task Definition (The Blueprint)**

**Key Parameters:**

- **Image:** ECR or Docker Hub URL.
- **CPU/Memory:** Task-level or container-level.
- **Port Mappings:** Host port → Container port.
- **Environment Variables:** Pass config (Can use Secrets Manager/Parameter Store).
- **IAM Role:** **Task Role** (Permissions for application) vs. **Execution Role** (Permissions for ECS agent to pull image, write logs).
- **Logging:** Send logs to CloudWatch Logs.
- **Volumes:** Attach EFS or bind mounts.

**Task Role vs. Execution Role:**

- **Task Role:** Assigned to the **application** (e.g., Lambda accesses S3 → Task Role allows S3).
- **Execution Role:** Used by **ECS agent** (Pull image from ECR, Write logs to CloudWatch).

---

### **B. Service (The Long-Running)**

**The Rule:** Ensures desired number of tasks always running.

**Features:**

- **Desired Count:** Number of tasks to run.
- **Load Balancer Integration:** ALB/NLB distributes traffic across tasks.
- **Auto Scaling:** Scale tasks based on CloudWatch metrics (CPU, Memory, ALB Request Count).
- **Deployment Types:**
    - **Rolling Update:** Replace old tasks gradually (Default).
    - **Blue/Green:** Deploy new version, test, then switch traffic (Requires CodeDeploy).
- **Service Discovery:** Use AWS Cloud Map (DNS-based discovery for microservices).

**Exam Trigger:** "Ensure 10 containers always running" → ECS Service.

---

### **4. ECS Scaling**

### **A. Service Auto Scaling (Task-Level)**

- **Target Tracking:** Maintain target metric (e.g., CPU = 70%).
- **Step Scaling:** Add/remove tasks based on CloudWatch alarms.
- **Scheduled Scaling:** Scale at specific times.

**Metrics:**

- ECS Service CPU/Memory
- ALB Request Count Per Target
- Custom CloudWatch metrics

### **B. Cluster Auto Scaling (EC2 Launch Type Only)**

- **The Rule:** Automatically add/remove EC2 instances when tasks can't fit.
- **Mechanism:** Uses **Capacity Provider** (Links ECS Service to ASG).
- **Note:** Fargate doesn't need this (Infinite capacity).

**Exam Trigger:** "Scale EC2 instances when tasks pending" → Capacity Provider + ASG.

---

### **5. ECS Networking**

### **A. awsvpc Network Mode (Recommended)**

- **The Rule:** Each task gets its own **ENI** (Elastic Network Interface) and **private IP**.
- **Security Groups:** Attach SG directly to task (Fine-grained security).
- **Required for Fargate.**

### **B. bridge Network Mode (Legacy)**

- Uses Docker bridge network.
- Multiple containers share instance's network.
- Less secure (No task-level SG).

**Exam Trigger:** "Task-level Security Groups" → awsvpc mode.

---

### **6. ECS Storage**

### **A. EFS (Elastic File System)**

- **The Rule:** Persistent shared storage for tasks.
- **Use Case:** Multiple tasks need access to same files (Shared content, Config files).
- **Supports:** Both EC2 and Fargate.

**Exam Trigger:** "Persistent shared storage for containers" → EFS.

### **B. Docker Volumes (EC2 Launch Type Only)**

- **Bind Mounts:** Mount host directory into container.
- **Ephemeral:** Data lost when task stops.

### **C. Fargate Ephemeral Storage**

- Fargate tasks get 20 GB ephemeral storage (Non-persistent).

---

### **SECTION 3: EKS (ELASTIC KUBERNETES SERVICE)**

*Managed Kubernetes on AWS.*

### **1. EKS Overview**

- **The Rule:** Fully managed Kubernetes control plane.
- **Kubernetes:** Open-source container orchestration (Industry standard).
- **Why EKS:** Run Kubernetes without managing control plane (Master nodes).

**When to Use EKS:**

- Already using Kubernetes on-prem (Migration).
- Need Kubernetes features (Helm, Operators, Custom controllers).
- Multi-cloud (Kubernetes portable).

**When to Use ECS:**

- AWS-native (Simpler, cheaper).
- No Kubernetes expertise required.

---

### **2. EKS Worker Nodes**

**Options:**

- **Managed Node Groups:** AWS manages EC2 instances (Patching, Updates).
- **Self-Managed Nodes:** You provision and manage EC2 instances.
- **Fargate:** Serverless (No EC2 to manage).

---

### **3. EKS Networking**

- **VPC CNI Plugin:** Each pod gets IP from VPC (Like awsvpc in ECS).
- **Security Groups for Pods:** Attach SG directly to pods.
- **Load Balancer Controller:** Provision ALB/NLB from Kubernetes Ingress/Service.

---

### **4. EKS Storage**

- **EBS:** Persistent volumes (Single-AZ, Block storage).
- **EFS:** Persistent volumes (Multi-AZ, Shared).
- **FSx for Lustre:** High-performance workloads.

---

### **ECS vs. EKS Comparison**

| **Feature** | **ECS** | **EKS** |
| --- | --- | --- |
| **Orchestrator** | AWS proprietary | Kubernetes (Open-source) |
| **Complexity** | Simpler | More complex |
| **Portability** | AWS-only | Multi-cloud |
| **Cost** | Free (Pay for resources) | $0.10/hour per cluster + resources |
| **Use Case** | AWS-native, Simpler workloads | Kubernetes migration, Advanced features |

**Exam Trigger:** "Migrate on-prem Kubernetes to AWS" → EKS. "Simple container orchestration" → ECS.

---

### **SECTION 4: ECR IMAGE SCANNING & LIFECYCLE**

### **1. Image Scanning**

- **The Rule:** Scan images for vulnerabilities (CVEs).
- **Types:**
    - **Basic Scanning:** Free. Scans on push (Uses Clair).
    - **Enhanced Scanning (Inspector):** Continuous scanning. More detailed findings.

**Exam Trigger:** "Scan container images for vulnerabilities" → ECR Image Scanning.

---

### **2. Lifecycle Policies**

- **The Rule:** Automatically delete old/unused images.
- **Rules:** Based on image age or count.
- **Example:** "Keep only last 10 images" or "Delete images older than 30 days."

**Exam Trigger:** "Automatically clean up old container images" → ECR Lifecycle Policies.

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **Store Docker images privately?** → ECR.
2. **Run containers without managing servers?** → Fargate.
3. **Run containers on EC2 with Reserved Instances?** → ECS EC2 Launch Type.
4. **Persistent shared storage for containers?** → EFS.
5. **Task-level Security Groups?** → awsvpc network mode.
6. **Ensure 10 containers always running?** → ECS Service (Desired Count = 10).
7. **Scale EC2 instances when tasks pending?** → Capacity Provider + ASG.
8. **Migrate Kubernetes to AWS?** → EKS.
9. **Simple container orchestration (AWS-native)?** → ECS.
10. **Scan images for vulnerabilities?** → ECR Image Scanning.
11. **Container needs S3 access?** → Task Role (Not Execution Role).
12. **ECS agent needs to pull image from ECR?** → Execution Role.

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "No Ops Containers" (Fargate)**

**The Situation:** A startup wants to run microservices in Docker containers. They have a small team with **no DevOps experience** and don't want to manage servers. The workload is **variable** (spikes during business hours, near-zero at night).

**The Options:**

A. Use ECS with EC2 Launch Type and Auto Scaling Groups.

B. Use ECS with Fargate Launch Type.

C. Use EKS with Managed Node Groups.

D. Run Docker directly on EC2 with Docker Compose.

**The Logic:**

- **Trap:** EC2 Launch Type (Option A) requires managing instances (Patching, Scaling, AMIs).
- **Trap:** EKS (Option C) is complex (Requires Kubernetes knowledge).
- **The Fix:** **Option B**. **Fargate** is serverless (No servers to manage). Auto-scales tasks. Pay per task (No idle instance cost). Perfect for variable workloads and small teams.

---

### **Scenario 2: The "Shared Content" (EFS)**

**The Situation:** A media processing application runs 50 ECS tasks (Fargate) that all need **read/write access** to the same set of video files. The files must persist even if tasks restart. Tasks may run concurrently and need **shared access**.

**The Options:**

A. Use S3 and mount with s3fs.

B. Use EBS volumes attached to each task.

C. Use EFS and mount it to all tasks.

D. Use Fargate ephemeral storage.

**The Logic:**

- **Trap:** S3 (Option A) is object storage (Not true filesystem, s3fs has limitations).
- **Trap:** EBS (Option B) cannot be shared across tasks (Single-AZ, single-attach unless Multi-Attach io2).
- **Trap:** Ephemeral storage (Option D) is not persistent (Lost when task stops).
- **The Fix:** **Option C**. **EFS** is a shared, persistent filesystem. Multi-AZ, POSIX-compliant. All 50 tasks can mount the same EFS filesystem concurrently. Supported by both EC2 and Fargate.

---

### **Scenario 3: The "Cost Optimization" (EC2 Launch Type + Spot)**

**The Situation:** A batch processing workload runs in ECS 24/7. The workload is **fault-tolerant** (tasks can restart). The company wants to reduce costs by **70%** while maintaining throughput.

**The Options:**

A. Use Fargate Spot.

B. Use ECS EC2 Launch Type with Spot Instances in the Auto Scaling Group.

C. Use ECS EC2 Launch Type with Reserved Instances.

D. Use Lambda instead.

**The Logic:**

- **Trap:** Fargate Spot (Option A) saves ~70% over Fargate On-Demand, but Fargate is more expensive per vCPU than EC2.
- **Trap:** Lambda (Option D) has 15-minute limit (Batch processing may exceed).
- **The Fix:** **Option B**. **ECS EC2 Launch Type with Spot Instances** provides the deepest savings (Up to 90% off On-Demand EC2). Use Capacity Provider to mix Spot + On-Demand for reliability. Since workload is fault-tolerant, Spot interruptions are acceptable.

---

### **Scenario 4: The "S3 Access" (Task Role)**

**The Situation:** An ECS task (Fargate) needs to read files from an S3 bucket. The developer hardcoded AWS credentials in the container image. Security team flags this as a violation.

**The Options:**

A. Store credentials in environment variables in Task Definition.

B. Create an IAM **Task Role** with S3 read permissions and attach to Task Definition.

C. Create an IAM **Execution Role** with S3 read permissions.

D. Use Secrets Manager to store credentials and fetch at runtime.

**The Logic:**

- **Trap:** Environment variables (Option A) still expose credentials (Visible in console/logs).
- **Trap:** Execution Role (Option C) is for ECS agent (Pull image, Write logs), not for application.
- **Trap:** Secrets Manager (Option D) is for database passwords, not IAM permissions.
- **The Fix:** **Option B**. **Task Role** grants permissions to the **application running inside the container**. No credentials in code/config. AWS automatically provides temporary credentials via metadata endpoint. Task Role = Application permissions. Execution Role = ECS agent permissions.

---

### **Scenario 5: The "Kubernetes Migration" (EKS)**

**The Situation:** A company runs a large application on **Kubernetes** in their on-premise datacenter using Helm charts, custom operators, and StatefulSets. They want to migrate to AWS with **minimal changes** to their Kubernetes manifests.

**The Options:**

A. Rewrite the application to use ECS Task Definitions and Services.

B. Use EKS and redeploy existing Kubernetes manifests.

C. Use Fargate and manually convert Kubernetes YAML to Task Definitions.

D. Use Lambda and refactor to serverless.

**The Logic:**

- **Trap:** Rewriting to ECS (Option A) requires significant effort (Different orchestration model).
- **Trap:** Lambda (Option D) is a complete architecture change (Not compatible with StatefulSets, Operators).
- **The Fix:** **Option B**. **EKS** is managed Kubernetes. Existing Helm charts, Operators, and Kubernetes manifests work with minimal changes. Just point `kubectl` to EKS cluster. AWS manages control plane. Kubernetes portability in action.
