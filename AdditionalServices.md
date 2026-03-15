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
