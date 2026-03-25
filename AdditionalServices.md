# Additional Services (1-2 Question Topics)

*Quick reference for services that appear infrequently on the SAA-C03 exam but are still fair game.*

---

### **SECTION 1: AMAZON ATHENA**

*Serverless SQL query engine for S3 data.*

**Key Facts:**

- **The Rule:** Run SQL queries directly on data stored in S3. No servers to provision.
- **Pricing:** Pay per query (~$5 per TB scanned). Use columnar formats to reduce cost.
- **Supported Formats:** CSV, JSON, Parquet, ORC, Avro.
- **Integration:** Uses **AWS Glue Data Catalog** as the metadata store (table definitions, schema).
- **Federated Queries:** Query data beyond S3 -- RDS, DynamoDB, Redshift, on-prem databases via Lambda-based connectors.
- **Performance Tip:** Convert data to **Parquet or ORC** (Columnar) to reduce scan cost by up to 90%.

**Exam Triggers:**

- "Query S3 data with SQL" --> Athena.
- "Serverless analytics on data lake" --> Athena.
- "Reduce Athena query cost" --> Use Parquet/ORC (Columnar format), partition data.

**Common Traps:**

- Athena is for **ad-hoc queries**, not a data warehouse (that is Redshift).
- Athena does not store data -- it queries data **in place** in S3.

---

### **SECTION 2: AMAZON REDSHIFT**

*Managed data warehouse for OLAP workloads.*

**Key Facts:**

- **The Rule:** Columnar storage, massively parallel processing (MPP). Optimized for **OLAP** (Online Analytical Processing), NOT OLTP.
- **Architecture:** Leader node (query planning) + Compute nodes (execution). Up to 128 compute nodes.
- **Redshift Spectrum:** Query data in S3 directly without loading it into Redshift (like Athena but from within Redshift).
- **Redshift Serverless:** No cluster to manage. Auto-scales. Pay for compute used.
- **Snapshots:** Automated and manual. Can copy snapshots to another region for DR.
- **Enhanced VPC Routing:** Forces all COPY/UNLOAD traffic through VPC (for security/compliance).

**Exam Triggers:**

- "Data warehouse" or "OLAP" --> Redshift.
- "Complex analytical queries across petabytes" --> Redshift.
- "Query S3 from Redshift without loading" --> Redshift Spectrum.

**Common Traps:**

- Redshift is NOT for OLTP (transactional) workloads -- that is RDS or Aurora.
- Redshift is NOT serverless by default -- you must choose Redshift Serverless explicitly.
- Do not confuse Redshift Spectrum (query from Redshift cluster) with Athena (standalone query engine).

---

### **SECTION 3: AWS GLUE**

*Serverless ETL service and central metadata catalog.*

**Key Facts:**

- **The Rule:** Fully managed ETL (Extract, Transform, Load). No servers to provision.
- **Glue Data Catalog:** Central metadata repository. Stores table definitions, schema, location. Used by Athena, Redshift Spectrum, and EMR.
- **Glue Crawlers:** Automatically scan data sources (S3, RDS, DynamoDB) and infer schema. Populate the Data Catalog.
- **Glue Jobs:** Run ETL scripts (Python or Scala) on Apache Spark under the hood.
- **Glue Job Bookmarks:** Track previously processed data to prevent reprocessing on subsequent runs.
- **Glue Studio:** Visual ETL job authoring interface.

**Exam Triggers:**

- "ETL" or "Transform data between formats" --> Glue.
- "Discover schema automatically" --> Glue Crawlers.
- "Central metadata catalog" --> Glue Data Catalog.
- "Convert CSV to Parquet" --> Glue ETL Job.

**Common Traps:**

- Glue is for **batch ETL**, not real-time streaming (that is Kinesis or Firehose).
- Glue Data Catalog is a **metadata store**, not a data store -- data stays in S3/RDS/etc.

---

### **SECTION 4: AMAZON EVENTBRIDGE**

*Serverless event bus for event-driven architectures.*

**Key Facts:**

- **The Rule:** Successor to CloudWatch Events. Receives events and routes them to targets based on rules.
- **Event Bus Types:**
    - **Default Event Bus:** Receives all AWS service events automatically (EC2 state change, S3 events, etc.).
    - **Custom Event Bus:** For your application events.
    - **Partner Event Bus:** Receive events from SaaS partners (Zendesk, Datadog, Auth0).
- **Rules:** Match events using event patterns (content-based filtering) or run on a schedule (cron/rate).
- **Targets:** Lambda, SQS, SNS, Step Functions, Kinesis, ECS tasks, API Gateway, and more.
- **Schema Registry:** Discover and store event schemas. Generate code bindings.
- **Archive & Replay:** Archive events and replay them later (useful for debugging).

**Exam Triggers:**

- "React to AWS service events" or "Event-driven architecture" --> EventBridge.
- "Schedule Lambda function" --> EventBridge rule with schedule expression.
- "Receive events from SaaS providers" --> EventBridge Partner Event Bus.
- "Decouple microservices with events" --> EventBridge.

**Common Traps:**

- EventBridge is NOT a message queue (that is SQS). Events are routed, not queued.
- CloudWatch Events and EventBridge use the same underlying API -- EventBridge is the newer, preferred service.

---

### **SECTION 5: AWS CLOUDFORMATION**

*Infrastructure as Code (IaC) service.*

**Key Facts:**

- **The Rule:** Define AWS resources in templates (JSON or YAML). Deploy templates as **Stacks**.
- **Stack:** A collection of AWS resources managed as a single unit. Create/update/delete together.
- **StackSets:** Deploy stacks across **multiple accounts and regions** simultaneously.
- **Drift Detection:** Detect if resources have been changed outside CloudFormation (manual changes).
- **Rollback:** On failure, CloudFormation automatically rolls back to the previous working state (default behavior).
- **Change Sets:** Preview what changes CloudFormation will make before executing an update.
- **Nested Stacks:** Reusable templates referenced from a parent stack (modular IaC).
- **Cost:** CloudFormation itself is free. You pay only for the resources it creates.

**Exam Triggers:**

- "Infrastructure as Code" or "Automate resource provisioning" --> CloudFormation.
- "Deploy same infrastructure across multiple accounts/regions" --> StackSets.
- "Detect manual changes to resources" --> Drift Detection.
- "Preview infrastructure changes before applying" --> Change Sets.

**Common Traps:**

- CloudFormation is AWS-specific. Terraform is multi-cloud (but not an AWS service).
- StackSets require **AWS Organizations** or self-managed permissions for cross-account deployment.

---

### **SECTION 6: AWS ELASTIC BEANSTALK**

*Platform as a Service (PaaS) for web applications.*

**Key Facts:**

- **The Rule:** Deploy application code. AWS handles provisioning (EC2, ALB, ASG, RDS). You retain full control of underlying resources.
- **Supported Platforms:** Java, .NET, Node.js, Python, Go, Ruby, PHP, Docker.
- **Components:** Application --> Environment (Web Server or Worker). Environments contain EC2 instances, load balancers, auto scaling groups.
- **Deployment Strategies:** All at once, Rolling, Rolling with additional batch, Immutable, Blue/Green (via DNS swap).
- **Cost:** Elastic Beanstalk itself is free. You pay only for the underlying resources (EC2, ALB, RDS, etc.).
- **Not Serverless:** Uses EC2 instances under the hood. For serverless, use Lambda.
- **.ebextensions:** Configuration files (YAML/JSON) in `.ebextensions/` folder to customize environment.

**Exam Triggers:**

- "Deploy web app with minimal infrastructure management" --> Elastic Beanstalk.
- "PaaS on AWS" --> Elastic Beanstalk.
- "Developer wants to upload code, not manage servers" --> Elastic Beanstalk.

**Common Traps:**

- Elastic Beanstalk is NOT serverless -- it provisions EC2 instances.
- Elastic Beanstalk vs. CloudFormation: Beanstalk is higher-level (focused on web apps). CloudFormation is lower-level (any AWS resource).
- You CAN still SSH into the EC2 instances and customize everything -- full control is maintained.

---

### **SECTION 7: AMAZON OPENSEARCH**

*Search and analytics engine (formerly Amazon ElasticSearch Service).*

**Key Facts:**

- **The Rule:** Managed OpenSearch (fork of Elasticsearch). Full-text search, log analytics, application monitoring.
- **OpenSearch Dashboards:** Built-in visualization tool (formerly Kibana). Create dashboards from indexed data.
- **Common Data Sources:** Kinesis Data Firehose, CloudWatch Logs, IoT, DynamoDB Streams, custom applications.
- **Deployment:** Runs on managed instances (not serverless by default). OpenSearch Serverless is also available.
- **Multi-AZ:** Supports up to 3 AZs for high availability.
- **Patterns:**
    - DynamoDB Table --> DynamoDB Stream --> Lambda --> OpenSearch (searchable copy of DynamoDB data).
    - CloudWatch Logs --> Subscription Filter --> Lambda/Firehose --> OpenSearch (log analytics).

**Exam Triggers:**

- "Full-text search" or "Search functionality" --> OpenSearch.
- "Log analytics with dashboards" --> OpenSearch + OpenSearch Dashboards.
- "Make DynamoDB data searchable" --> DynamoDB Streams + Lambda + OpenSearch.

**Common Traps:**

- OpenSearch is NOT a primary database -- it is a search/analytics engine. Data should be replicated from the primary source.
- Do not confuse with CloudWatch Logs Insights (simple log queries) vs. OpenSearch (full analytics platform with dashboards).

---

### **SECTION 8: AWS SES (SIMPLE EMAIL SERVICE)**

*Managed email sending and receiving service.*

**Key Facts:**

- **The Rule:** Send and receive emails at scale. Transactional, marketing, and bulk emails.
- **Use Cases:** Order confirmations, password resets, marketing campaigns, notifications.
- **Sending:** Via SMTP interface or AWS SDK/API.
- **Receiving:** Process incoming emails with rules (store in S3, trigger Lambda, send to SNS).
- **Reputation Dashboard:** Monitor bounce rates, complaints, delivery rates.

**Exam Triggers:**

- "Send transactional emails" or "Send bulk emails" --> SES.
- "Email service" --> SES.

**Common Traps:**

- **SES vs. SNS:** SES is for **email** (rich HTML, attachments). SNS is for **notifications** (pub/sub to multiple protocols including email, but plain text only). "Send formatted emails" --> SES. "Send notifications to multiple subscribers" --> SNS.

---

### **SECTION 9: AWS APPSYNC**

*Managed GraphQL API service.*

**Key Facts:**

- **The Rule:** Build GraphQL APIs with real-time data sync and offline capabilities.
- **Data Sources:** DynamoDB, Lambda, Aurora, OpenSearch, HTTP endpoints, RDS.
- **Real-Time:** WebSocket-based subscriptions. Clients receive updates instantly when data changes.
- **Offline:** Built-in conflict resolution for mobile/web apps that work offline and sync when reconnected.
- **Caching:** Optional API-level caching to reduce backend load.

**Exam Triggers:**

- "GraphQL" --> AppSync.
- "Real-time data sync for mobile/web apps" --> AppSync.
- "Offline-capable mobile app with data sync" --> AppSync.

**Common Traps:**

- AppSync is for **GraphQL**, not REST. For REST APIs, use API Gateway.
- Do not confuse AppSync (GraphQL API) with Cognito Sync (user data sync -- deprecated in favor of AppSync).

---

### **SECTION 10: AMAZON MSK (MANAGED STREAMING FOR APACHE KAFKA)**

*Managed Apache Kafka service.*

**Key Facts:**

- **The Rule:** Fully managed Apache Kafka. Run Kafka on AWS without managing brokers, ZooKeeper, or infrastructure.
- **MSK Serverless:** No capacity planning needed. Auto-scales.
- **MSK Connect:** Managed Kafka Connect workers to stream data to/from Kafka (S3, databases, etc.).
- **Storage:** Data stored on EBS volumes. Configurable retention.
- **Multi-AZ:** Deploy across 2 or 3 AZs for high availability.

**Exam Triggers:**

- "Migrate existing Kafka workload to AWS" --> MSK (NOT Kinesis).
- "Managed Kafka" --> MSK.
- "Apache Kafka compatible" --> MSK.

**Common Traps:**

- **MSK vs. Kinesis:** MSK is for teams already using Kafka (API-compatible, easy migration). Kinesis is AWS-native streaming (simpler, tighter AWS integration). "New streaming application on AWS" --> Kinesis. "Migrate existing Kafka" --> MSK.
- MSK is NOT serverless by default -- you must choose MSK Serverless explicitly.

---

### **SECTION 11: AWS BATCH**

*Managed batch computing service.*

**Key Facts:**

- **The Rule:** Run batch computing jobs at any scale. AWS handles provisioning, scaling, and managing compute resources (EC2 or Fargate).
- **Jobs:** Packaged as **Docker containers**. Submit jobs to a job queue, Batch schedules them on optimal compute.
- **Compute Environments:** Managed (AWS provisions EC2/Fargate) or Unmanaged (you manage instances).
- **Auto-Provisioning:** Automatically selects optimal instance types based on job requirements (CPU, memory, GPU).
- **No Time Limit:** Jobs can run for hours or days (unlike Lambda's 15-minute max).

**Exam Triggers:**

- "Large-scale batch jobs" or "Long-running batch processing" --> AWS Batch.
- "Batch processing with Docker containers" --> AWS Batch.
- "Processing job that takes hours" --> AWS Batch (NOT Lambda -- 15 min max).

**Common Traps:**

- AWS Batch vs. Lambda: Batch is for **long-running, resource-heavy jobs**. Lambda is for **short, event-driven functions** (15 min max, limited memory).
- AWS Batch is NOT real-time -- it is for batch (asynchronous) workloads.

---

### **SECTION 12: AMAZON EMR**

*Managed big data framework.*

**Key Facts:**

- **The Rule:** Managed clusters running **Apache Hadoop, Spark, Hive, Presto, HBase, Flink**. Process massive datasets.
- **Architecture:** Master node + Core nodes (store data on HDFS) + Task nodes (compute only, optional).
- **Storage:** HDFS on cluster, or use S3 (EMRFS) for persistent storage that outlives the cluster.
- **Use Cases:** Log analysis, ETL at scale, machine learning training, genomics, financial analysis.

**Exam Triggers:**

- "Big data processing" or "Apache Spark" or "Hadoop cluster" --> EMR.
- "Process petabytes of data with Spark/Hive" --> EMR.
- "Managed Hadoop" --> EMR.

**Common Traps:**

- EMR vs. Glue: EMR gives you **full control** over the big data framework. Glue is **serverless ETL** (simpler, less control).
- EMR clusters can be transient (spin up, process, terminate) to save cost.

---

### **SECTION 13: AMAZON QUICKSIGHT**

*Serverless business intelligence and visualization.*

**Key Facts:**

- **The Rule:** Serverless BI service. Create interactive dashboards and visualizations. Pay per session.
- **Data Sources:** S3, Athena, Redshift, RDS, Aurora, OpenSearch, on-prem databases, Excel/CSV files.
- **SPICE Engine:** In-memory calculation engine for fast dashboard performance. Import data into SPICE for sub-second query response.
- **Sharing:** Publish dashboards to users/groups. Embed dashboards in applications.

**Exam Triggers:**

- "Create dashboards" or "Business intelligence" --> QuickSight.
- "Visualize data from S3/Redshift/Athena" --> QuickSight.
- "Serverless BI" --> QuickSight.

**Common Traps:**

- QuickSight is for **visualization/dashboards**, not data processing. Use Athena/Redshift for queries, QuickSight to display results.

---

### **SECTION 14: AWS LAKE FORMATION**

*Data lake governance and management.*

**Key Facts:**

- **The Rule:** Build, secure, and manage **data lakes** on S3. Simplifies data ingestion, cataloging, and access control.
- **Under the Hood:** Uses Glue crawlers, Glue ETL, and Glue Data Catalog.
- **Fine-Grained Access Control:** Column-level and row-level security on data lake tables. Central permission model replaces complex S3/IAM policies.
- **Data Ingestion:** Import data from RDS, S3, on-prem databases. Automatically converts to optimized formats.
- **Cross-Account Access:** Share data lake tables across AWS accounts securely.

**Exam Triggers:**

- "Data lake governance" or "Fine-grained access to data lake" --> Lake Formation.
- "Column-level security on S3 data" --> Lake Formation.
- "Build a data lake on S3 with centralized permissions" --> Lake Formation.

**Common Traps:**

- Lake Formation is a **governance layer** on top of Glue and S3 -- it does not replace them.
- Lake Formation vs. Glue: Glue is the ETL engine. Lake Formation adds **security and governance** on top.

---

### **SECTION 15: ML/AI SERVICES (QUICK REFERENCE)**

*Managed AI/ML services -- know what each one does.*

| **Service** | **Purpose** | **Exam Trigger** |
| --- | --- | --- |
| **Rekognition** | Image and video analysis (faces, objects, text, content moderation) | "Analyze images" or "Detect faces in video" |
| **Textract** | Extract text and data from documents (OCR) | "Extract text from scanned documents" |
| **Comprehend** | Natural Language Processing -- sentiment analysis, entity extraction | "Sentiment analysis" or "NLP" |
| **Transcribe** | Speech-to-text | "Convert audio to text" |
| **Polly** | Text-to-speech | "Convert text to audio" |
| **Lex** | Build chatbots (same technology that powers Alexa) | "Chatbot" or "Conversational interface" |
| **Translate** | Language translation | "Translate text between languages" |
| **SageMaker** | Build, train, and deploy custom ML models | "Custom ML model" or "Train ML model" |
| **Kendra** | Intelligent enterprise search powered by ML | "Intelligent search across documents" |

**Exam Rule:** You do NOT need to know how to use these services in depth. Just know **which service matches which use case**.

---

### **SECTION 16: EDGE & HYBRID INFRASTRUCTURE**

*AWS infrastructure beyond standard regions.*

### **AWS Outposts**

- **The Rule:** AWS-managed infrastructure installed **in your on-premises data center**. Same AWS hardware, APIs, and services -- but physically in your facility.
- **Use Cases:** Data residency requirements, ultra-low latency to on-prem systems, local data processing.
- **Exam Trigger:** "Must keep data on-premises but want AWS services" --> Outposts.

### **AWS Local Zones**

- **The Rule:** Extend an AWS Region to a specific metro area. Run latency-sensitive workloads closer to end users.
- **Use Cases:** Real-time gaming, media content creation, video streaming, AR/VR.
- **Exam Trigger:** "Single-digit millisecond latency to a specific city" --> Local Zones.

### **AWS Wavelength**

- **The Rule:** Embed AWS compute and storage at **5G network edges** (inside telecom provider data centers). Ultra-low latency for mobile devices.
- **Use Cases:** Connected vehicles, interactive live video, AR/VR, real-time gaming on mobile.
- **Exam Trigger:** "5G edge computing" or "Ultra-low latency for mobile/5G devices" --> Wavelength.

**Quick Differentiation:**

| **Need** | **Service** |
| --- | --- |
| AWS services on-premises | Outposts |
| Low latency to a specific city | Local Zones |
| Low latency for 5G mobile devices | Wavelength |

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "SQL on S3" (Athena vs. Redshift Spectrum)**

**The Situation:** A company stores 5 TB of web analytics data in S3 as compressed JSON files. Business analysts need to run ad-hoc SQL queries against this data to generate weekly reports. The company wants the simplest, most cost-effective solution with no infrastructure to manage.

**The Options:**

A. Load the data into a Redshift cluster and run queries from Redshift.

B. Use Amazon Athena to query the data directly in S3.

C. Use Redshift Spectrum to query the data in S3 from a Redshift cluster.

D. Set up an EMR cluster with Hive to query the data.

**The Logic:**

- **Trap:** Redshift cluster (Option A) requires provisioning and ongoing management/cost -- overkill for weekly ad-hoc reports.
- **Trap:** Redshift Spectrum (Option C) requires an existing Redshift cluster -- adds unnecessary cost and complexity.
- **Trap:** EMR with Hive (Option D) requires cluster management -- not "simplest."
- **The Fix:** **Option B**. **Athena** is serverless, no infrastructure to manage. Pay per query ($5/TB scanned). Queries S3 data in place. For even lower cost, convert JSON to **Parquet** (columnar) to reduce scan volume.

---

### **Scenario 2: The "Batch Processing" (AWS Batch vs. Lambda)**

**The Situation:** A genomics research company needs to process 10,000 DNA sequence files. Each file takes 2-4 hours to process and requires 16 GB of memory. The processing is packaged as a Docker container. The workload runs once per week.

**The Options:**

A. Use AWS Lambda functions triggered by S3 uploads for each file.

B. Use AWS Batch with a managed compute environment to process all files.

C. Use AWS Step Functions to orchestrate Lambda functions for each file.

D. Use Amazon ECS with a fixed cluster of EC2 instances running 24/7.

**The Logic:**

- **Trap:** Lambda (Option A) has a **15-minute timeout** and **10 GB max memory** -- 2-4 hour jobs with 16 GB memory cannot run on Lambda.
- **Trap:** Step Functions + Lambda (Option C) still hits the same Lambda limits.
- **Trap:** ECS 24/7 cluster (Option D) wastes money when the workload only runs weekly.
- **The Fix:** **Option B**. **AWS Batch** handles Docker containers with no time limit. Auto-provisions optimal EC2 instances (16 GB+ memory). Processes 10,000 jobs in parallel across dynamically scaled compute. Terminates resources when done -- pay only for what you use.

---

### **Scenario 3: The "Data Lake" (Glue + Lake Formation + Athena)**

**The Situation:** A healthcare company has data scattered across S3 (CSV files), RDS (patient records), and DynamoDB (IoT sensor data). They want to build a centralized data lake on S3 with the ability to query across all sources. Different departments need different levels of access -- some can see all columns, others only non-PII columns.

**The Options:**

A. Copy all data into Redshift and manage access with Redshift user permissions.

B. Use AWS Glue to catalog and transform data into S3, then use Athena for queries with S3 bucket policies for access control.

C. Use AWS Lake Formation to build the data lake with Glue ETL, and enforce column-level access control. Query with Athena.

D. Use EMR to process and consolidate all data into a single S3 bucket.

**The Logic:**

- **Trap:** Redshift (Option A) is a data warehouse, not a data lake. Expensive for storing raw data.
- **Trap:** Glue + Athena + S3 bucket policies (Option B) works for ETL and querying, but S3 bucket policies cannot enforce **column-level** access control.
- **Trap:** EMR (Option D) consolidates data but provides no governance or fine-grained access control.
- **The Fix:** **Option C**. **Lake Formation** provides centralized governance with **column-level and row-level security**. Glue crawlers discover schema, Glue ETL transforms data into Parquet on S3. Athena queries the data. Lake Formation manages who can access which columns -- ideal for PII restrictions.

---

### **Scenario 4: The "Legacy Migration" (Elastic Beanstalk vs. ECS)**

**The Situation:** A company is migrating a monolithic Java web application from on-premises to AWS. The team has limited AWS experience and wants to deploy quickly with minimal infrastructure management. They want automated scaling, load balancing, and deployment management. The application is NOT containerized.

**The Options:**

A. Deploy on EC2 instances manually with an ALB and Auto Scaling Group.

B. Containerize the app and deploy on Amazon ECS with Fargate.

C. Deploy using AWS Elastic Beanstalk with a Java platform.

D. Refactor the app into Lambda functions.

**The Logic:**

- **Trap:** Manual EC2 + ALB + ASG (Option A) works but requires significant infrastructure management -- not "minimal management."
- **Trap:** ECS/Fargate (Option B) requires containerizing the application first -- adds migration effort.
- **Trap:** Lambda (Option D) requires breaking the monolith into functions -- major refactor.
- **The Fix:** **Option C**. **Elastic Beanstalk** supports Java natively. Upload the WAR/JAR file, Beanstalk provisions EC2, ALB, ASG automatically. Built-in deployment strategies (rolling, immutable). Team retains full control if needed. Fastest path for a non-containerized Java app with minimal AWS experience.

---

### **Scenario 5: The "Event-Driven Architecture" (EventBridge vs. SNS)**

**The Situation:** An e-commerce platform needs to react to order events (order placed, shipped, delivered, returned). Different microservices need different events: the inventory service needs "order placed," the notification service needs "shipped" and "delivered," and the analytics service needs all events. Events also need to integrate with a third-party SaaS returns-processing system.

**The Options:**

A. Use SNS with one topic per event type and subscribe each service to the relevant topics.

B. Use Amazon EventBridge with content-based filtering rules to route events to the correct targets.

C. Use SQS queues with one queue per service, and have the order service publish to all queues.

D. Use Kinesis Data Streams with one stream for all events and filter in each consumer.

**The Logic:**

- **Trap:** SNS (Option A) can do basic fan-out, but content-based filtering is limited. Also, SNS does NOT natively integrate with SaaS partners.
- **Trap:** SQS (Option C) requires the producer to know all consumers -- tight coupling.
- **Trap:** Kinesis (Option D) is for high-throughput streaming -- overkill for event routing. Each consumer must filter events itself.
- **The Fix:** **Option B**. **EventBridge** excels at content-based filtering -- route "order placed" to inventory, "shipped"/"delivered" to notifications, and all events to analytics, all from a single event bus. Partner Event Bus supports SaaS integration for the returns system. Schema Registry helps teams discover event formats. Archive & Replay enables debugging.

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **"Query S3 data with SQL"?** --> Athena.
2. **"Data warehouse / OLAP"?** --> Redshift.
3. **"Query S3 from Redshift"?** --> Redshift Spectrum.
4. **"ETL / Transform data"?** --> Glue.
5. **"Discover schema automatically"?** --> Glue Crawlers.
6. **"Central metadata catalog"?** --> Glue Data Catalog.
7. **"React to AWS events / Event-driven"?** --> EventBridge.
8. **"Schedule Lambda"?** --> EventBridge rule with schedule.
9. **"Infrastructure as Code"?** --> CloudFormation.
10. **"Deploy across multiple accounts/regions"?** --> CloudFormation StackSets.
11. **"Deploy web app, minimal management"?** --> Elastic Beanstalk.
12. **"Full-text search / Log analytics with dashboards"?** --> OpenSearch.
13. **"Send transactional emails"?** --> SES (NOT SNS).
14. **"GraphQL / Real-time data sync"?** --> AppSync.
15. **"Migrate Kafka to AWS"?** --> MSK (NOT Kinesis).
16. **"New streaming app on AWS"?** --> Kinesis (NOT MSK).
17. **"Reduce Athena cost"?** --> Columnar format (Parquet/ORC) + partitioning.
18. **"Detect manual infrastructure changes"?** --> CloudFormation Drift Detection.
19. **"Long-running batch jobs with Docker"?** --> AWS Batch (NOT Lambda).
20. **"Big data / Apache Spark / Hadoop"?** --> EMR.
21. **"Create dashboards / BI"?** --> QuickSight.
22. **"Data lake governance / column-level security"?** --> Lake Formation.
23. **"Analyze images / Detect faces"?** --> Rekognition.
24. **"Extract text from documents (OCR)"?** --> Textract.
25. **"Sentiment analysis / NLP"?** --> Comprehend.
26. **"Build chatbot"?** --> Lex.
27. **"Custom ML model"?** --> SageMaker.
28. **"AWS services on-premises"?** --> Outposts.
29. **"Low latency to specific city"?** --> Local Zones.
30. **"5G edge computing"?** --> Wavelength.

---

### **Quick Differentiation Table**

| **Need** | **Service** | **NOT This** |
| --- | --- | --- |
| Query S3 with SQL | Athena | Redshift |
| Data warehouse (OLAP) | Redshift | RDS (OLTP) |
| ETL / Transform data | Glue | Lambda (simple transforms OK) |
| Event routing | EventBridge | SNS (pub/sub) |
| Infrastructure as Code | CloudFormation | Elastic Beanstalk (PaaS) |
| Deploy web app (PaaS) | Elastic Beanstalk | CloudFormation (IaC) |
| Full-text search | OpenSearch | DynamoDB (key-value) |
| Send emails | SES | SNS (notifications) |
| GraphQL API | AppSync | API Gateway (REST) |
| Managed Kafka | MSK | Kinesis (AWS-native) |
| Long-running batch jobs | AWS Batch | Lambda (15 min max) |
| Big data (Spark/Hadoop) | EMR | Glue (serverless ETL) |
| BI dashboards | QuickSight | OpenSearch (log analytics) |
| Data lake governance | Lake Formation | Glue (ETL only) |
| AWS on-premises | Outposts | Local Zones (metro area) |
| Low latency to city | Local Zones | Wavelength (5G mobile) |
| 5G edge computing | Wavelength | Local Zones (metro area) |
