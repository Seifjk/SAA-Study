# Containers

> **📌 Scope note:** AWS tests these with *"determine WHEN to use"* — you pick the service/launch type, you don't author task definitions or configure scaling policies. If it's not here, the exam doesn't ask it.

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

**The Rule:** You manage EC2 instances (running the ECS Agent, on the ECS-Optimized AMI); ECS handles container placement/scheduling. You manage EC2 (patching, scaling, instance types); pay for EC2 only (no extra ECS charge).

- **Use Cases:** Need instance control, cost optimization with Reserved/Spot Instances, steady-state workloads.

**Exam Trigger:** "Run containers on EC2 with Reserved Instances" → ECS EC2 Launch Type.

---

### **B. Fargate Launch Type (The Serverless)**

**The Rule:** AWS manages all infrastructure (servers, scaling, patching); you only define the Task Definition (image, CPU, memory). Pay per vCPU/GB-second — pricier per task but no idle instance cost.

- **Use Cases:** Don't want to manage instances, variable workloads (scale to zero), microservices.

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

### **3. ECS IAM Roles (The One Distinction That's Tested)**

**Task Role vs. Execution Role** — the classic trap:

- **Task Role:** Permissions for the **application** running in the container (e.g., container needs to read S3 → Task Role allows S3).
- **Execution Role:** Permissions for the **ECS agent** (pull image from ECR, write logs to CloudWatch).

---

### **B. Service (The Long-Running)**

**The Rule:** Ensures a desired number of tasks always run.

- **Desired Count:** number of tasks. **Load Balancer:** ALB/NLB distributes traffic. **Auto Scaling:** scale tasks on CloudWatch metrics (CPU, Memory, ALB Request Count). **Service Discovery** via AWS Cloud Map.

**Exam Trigger:** "Ensure 10 containers always running" → ECS Service.

---

### **4. ECS Scaling**

### **A. Service Auto Scaling (Task-Level)**

- Scale the **number of tasks** up/down on a CloudWatch metric (ECS CPU/Memory or ALB Request Count Per Target). This is the task-count knob.

### **B. Cluster Auto Scaling (EC2 Launch Type Only)**

- Automatically adds/removes EC2 instances when tasks can't fit, via a **Capacity Provider** (links ECS Service to ASG). Fargate doesn't need this.

**Exam Trigger:** "Scale EC2 instances when tasks pending" → Capacity Provider + ASG.

---

### **5. ECS Networking**

- **awsvpc Mode (recommended, required for Fargate):** Each task gets its own **ENI** and **private IP**; attach a Security Group directly to the task for fine-grained security.
- **bridge Mode (legacy):** Docker bridge network — containers share the instance's network, no task-level SG, less secure.

**Exam Trigger:** "Task-level Security Groups" → awsvpc mode.

---

### **6. ECS Storage**

- **EFS:** Persistent shared storage for tasks (multiple tasks accessing the same files). Supports both EC2 and Fargate.
- **Docker Volumes (EC2 only):** Bind mounts a host directory; ephemeral, lost when the task stops.
- **Fargate Ephemeral Storage:** 20 GB per task, non-persistent.

**Exam Trigger:** "Persistent shared storage for containers" → EFS.

---

### **SECTION 3: EKS (ELASTIC KUBERNETES SERVICE)**

*Managed Kubernetes on AWS.*

### **1. EKS Overview**

- **The Rule:** Fully managed Kubernetes control plane — it literally **is** Kubernetes (AWS runs the master nodes). Pick it when "Kubernetes" is in the question.
- **Use EKS** for on-prem Kubernetes migration, Kubernetes-specific features (Helm, Operators, custom controllers), or multi-cloud portability. **Use ECS** for AWS-native, simpler, cheaper workloads with no Kubernetes expertise.

---

### **2. EKS Worker Nodes**

- **Managed Node Groups:** AWS manages the EC2 instances (patching, updates). **Self-Managed Nodes:** you provision/manage EC2. **Fargate:** serverless, no EC2.

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

### **SECTION 4: AWS APP RUNNER**

*The simplest way to run a containerized web app on AWS.*

### **1. App Runner Overview**

- **The Rule:** Point to an ECR image or source repo → App Runner builds, deploys, scales, and load-balances automatically. No Task Definitions, Services, or Clusters.
- Built-in auto scaling (up on traffic, down to zero); auto-redeploy on new image/commit; HTTPS endpoint out of the box; private VPC access via VPC Connector. Pay per vCPU/memory while active + small pause fee when scaled to zero.
- **vs Fargate:** App Runner = zero orchestration config (best for simple web apps/APIs); Fargate still needs Task Definitions, Services, LB setup (more control, more config).

**Exam Trigger:** "Deploy container with zero infrastructure management" / "Simplest way to run a web app from a container image" → App Runner (simpler than Fargate).

---

### **SECTION 5: ECR IMAGE SCANNING & LIFECYCLE**

### **1. Image Scanning**

- **The Rule:** Scan images for vulnerabilities (CVEs). **Basic Scanning** (free, on push, uses Clair) vs **Enhanced Scanning** (Inspector, continuous, more detailed findings).

**Exam Trigger:** "Scan container images for vulnerabilities" → ECR Image Scanning.

---

### **2. Lifecycle Policies**

- Automatically delete old/unused images by age or count (e.g., "keep only last 10 images", "delete images older than 30 days").

**Exam Trigger:** "Automatically clean up old container images" → ECR Lifecycle Policies.

---

### **Exam Summary Cheat Sheet - Practice (Fill In the Blank)**

1. **Store Docker images privately?** → ECR private.
2. **Run containers without managing servers?** → Fargate
3. **Run containers on EC2 with Reserved Instances?** → ECS EC2 Launch Type
4. **Persistent shared storage for containers?** → EFS/EBS
5. **Task-level Security Groups?** → awsvpc network mode
6. **Ensure 10 containers always running?** → ECS
7. **Scale EC2 instances when tasks pending?** → Capacity Provider + ASG
8. **Migrate Kubernetes to AWS?** → EKS
9. **Simple container orchestration (AWS-native)?** → ECS/Fargate
10. **Scan images for vulnerabilities?** → Basic Scan or Enhanced Scan
11. **Container needs S3 access?** → Task Role
12. **ECS agent needs to pull image from ECR?** →  Execution Role
13. **Deploy container with zero infrastructure management?** → AppRunner

### **Exam Summary Cheat Sheet - Answer Key**

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
13. **Deploy container with zero infrastructure management?** → App Runner (simpler than Fargate).

---

# **REAL EXAM SCENARIOS**

### Scenario 1

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

### Scenario 2

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

### Scenario 3

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

### Scenario 4

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

### Scenario 5

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
