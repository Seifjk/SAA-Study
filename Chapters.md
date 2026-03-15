# Untitled

### **THE INFRASTRUCTURE CORE (Must Master)**

- [x]  **1. Networking (VPC)** (*Covered*)
    - *Gaps to fill:* VPN vs. Direct Connect deep dive, Transit Gateway architectures.
- [x]  **2. Storage** (*Covered*)
- [ ]  **3. Compute (EC2)**
    - *Scope:* Instance Types (Spot vs. Reserved), Placement Groups (Cluster vs. Spread), ENI vs. ENA vs. EFA, AMIs.
- [ ]  **4. High Availability (The "Elastic" Part)**
    - *Scope:* **Load Balancers** (ALB vs. NLB vs. GWLB) and **Auto Scaling Groups** (Scaling policies). *This is 20% of the exam.*

### **THE DATA LAYER (The Confusing Part)**

- [ ]  **5. Databases (Relational)**
    - *Scope:* **RDS** vs. **Aurora** (Serverless v2, Global Database).
    - *Key:* Read Replicas vs. Multi-AZ.
- [ ]  **6. Databases (NoSQL & Caching)**
    - *Scope:* **DynamoDB** (DAX, Streams, GSI/LSI) and **ElastiCache** (Redis vs. Memcached).

### **THE MODERN STACK (Your Strong Suit)**

- [ ]  **7. Serverless**
    - *Scope:* **Lambda** (Limits, Layers, Versions), **API Gateway** (Throttling, Stages), **Step Functions**.
- [ ]  **8. Containers**
    - *Scope:* **ECS** (Fargate vs. EC2 mode), **EKS** (High level), **ECR**. *Since you know K8s, we just need to map your knowledge to AWS terms.*
- [ ]  **9. Decoupling (Queues)**
    - *Scope:* **SQS** (Standard vs. FIFO), **SNS** (Fan-out), **Kinesis** (Data Streams vs. Firehose).

### **THE GLOBAL & SECURITY LAYER**

- [ ]  **10. Content Delivery & DNS**
    - *Scope:* **Route 53** (Routing Policies like Latency, Geolocation), **CloudFront** (OAC, Signed URLs), **Global Accelerator**.
- [ ]  **11. Security & Identity**
    - *Scope:* **IAM** (Deep dive), **KMS** (Key types), **Cognito** (User Pools vs. Identity Pools), **WAF & Shield**.
- [ ]  **12. Monitoring & Audit**
    - *Scope:* **CloudWatch** (Logs, Metrics, Events) vs. **CloudTrail** (Who did what) vs. **Config** (Compliance).

### **COST, MIGRATION & GOVERNANCE (Domain 4 = 20% of exam)**

- [ ]  **13. Cost Optimization & Migration**
    - *Scope:* **Cost Explorer**, **Budgets**, **Compute Optimizer**, **DMS**, **DataSync**, **DR Patterns** (Backup/Restore, Pilot Light, Warm Standby, Multi-Site), **AWS Organizations** (SCPs), **AWS Backup**.

---