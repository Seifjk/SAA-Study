# Serverless

### **SECTION 1: LAMBDA (THE COMPUTE ENGINE)**

*Run code without provisioning servers.*

### **1. Lambda Overview**

- **The Rule:** Event-driven compute. No servers to manage. Pay only for compute time.
- **Execution Model:** Code runs in response to triggers (events).
- **Scaling:** Automatically scales from 0 to thousands of concurrent executions.
- **Pricing:** Pay per request and compute duration (100 ms increments).
    - **Free Tier:** 1 million requests/month + 400,000 GB-seconds.

**Supported Languages:**

- Native: Node.js, Python, Java, C#, Go, Ruby, PowerShell
- Custom: Any language via Custom Runtime or Container Image (up to 10 GB).

---

### **2. Lambda Limits (MEMORIZE THESE)**

| **Limit** | **Value** | **Exam Trap** |
| --- | --- | --- |
| **Execution Timeout** | **15 minutes max** | "Long-running batch job" → NOT Lambda |
| **Memory** | 128 MB - 10,240 MB (10 GB) | More memory = More CPU |
| **Deployment Package** | 50 MB (zipped), 250 MB (unzipped) | Large dependencies → Use Layers or Container |
| **Concurrent Executions** | 1,000 per region (soft limit) | Throttling → Request increase or Reserved Concurrency |
| **Environment Variables** | 4 KB total | Large config → Use Parameter Store |
| **/tmp Storage** | 512 MB - 10,240 MB | Ephemeral only (Not persistent) |
| **Invocation Payload** | 6 MB (sync), **1 MB** (async) | Large payload → Use S3 |

**Exam Triggers:**

- "Task takes 20 minutes" → NOT Lambda (Use ECS/Batch).
- "Need persistent storage" → NOT /tmp (Use EFS/S3).
- "Deployment package 300 MB" → Use Container Image or Layers.

---

### **3. Invocation Types**

### **A. Synchronous (The Wait)**

- **The Rule:** Caller waits for response. Lambda returns result directly.
- **Use Cases:** API Gateway, ALB, Cognito, Step Functions.
- **Error Handling:** Caller receives error. Caller must retry.
- **Timeout:** Caller waits up to 15 minutes.

### **B. Asynchronous (The Fire and Forget)**

- **The Rule:** Caller gets immediate acknowledgment. Lambda processes in background.
- **Use Cases:** S3 events, SNS, EventBridge, SES.
- **Retries:** Lambda retries **2 times** automatically on failure (Total 3 attempts).
- **Dead Letter Queue (DLQ):** Send failed events to **SQS** or **SNS** after retries exhausted.
- **Destinations:** Send results to SQS, SNS, Lambda, EventBridge (Success or Failure paths).

**Exam Trigger:** "Lambda invoked by S3" → Asynchronous.

### **C. Event Source Mapping (The Poller)**

- **The Rule:** Lambda **polls** the source and pulls records.
- **Use Cases:** **Kinesis Data Streams**, **DynamoDB Streams**, **SQS**.
- **Batch Processing:** Lambda processes batches (configurable batch size).
- **Concurrency:** Kinesis/DynamoDB → 1 Lambda per shard. SQS → Scales up to 1,000 concurrent.
- **Error Handling:**
    - **Kinesis/DynamoDB:** Failed batch blocks shard until success or TTL expires.
    - **SQS:** Failed messages return to queue (After retries → DLQ).

**Exam Trigger:** "Lambda reads from Kinesis" → Event Source Mapping.

---

### **4. Execution Model**

**Lambda Lifecycle:**

1. **Cold Start:** First invocation. AWS provisions execution environment (Slow, 100-1000 ms).
2. **Warm Execution:** Subsequent invocations reuse environment (Fast, < 10 ms).
3. **Freeze:** After idle time (minutes), environment freezes but stays warm.
4. **Shutdown:** After extended idle, environment destroyed.

**Optimization Tips:**

- **Minimize Cold Starts:**
    - Use **Provisioned Concurrency** (Pre-warmed environments).
    - Keep functions small (Fast initialization).
    - Avoid heavy SDKs in function code.
- **Reuse Connections:** Initialize DB connections outside handler (Reused across invocations).

---

### **5. Lambda Layers**

**The Rule:** Package libraries and dependencies separately from function code.

**How It Works:**

- Create Layer (e.g., common libraries, SDKs).
- Attach Layer to multiple functions.
- Reduces deployment package size.

**Benefits:**

- Share code across functions.
- Faster deployments.
- Separate dependencies from business logic.

**Limits:**

- **5 Layers per function**.
- **250 MB total** (unzipped) including all layers + function code.

**Exam Trigger:** "Share common libraries across Lambda functions" → Layers.

---

### **6. Lambda Versions & Aliases**

### **A. Versions**

- **The Rule:** Immutable snapshot of function code + configuration.
- **$LATEST:** The working version (Mutable).
- **Numbered Versions:** Published versions (v1, v2, v3) - Immutable.

### **B. Aliases**

- **The Rule:** Pointer to a specific version (Mutable).
- **Use Cases:**
    - **Blue/Green Deployment:** Alias points to v1. Test v2. Switch alias to v2.
    - **Weighted Routing:** Route 90% traffic to v1, 10% to v2 (Canary testing).
- **Example:** Alias `PROD` → v5. Alias `DEV` → $LATEST.

**Exam Trigger:** "Gradually shift traffic to new Lambda version" → Aliases with Weighted Routing.

---

### **7. Environment Variables & Configuration**

- **Environment Variables:** Key-value pairs available to function (Max 4 KB total).
- **Encryption:** Encrypted at rest via KMS (Automatic). Can encrypt in transit too (Helpers).
- **Parameter Store / Secrets Manager:** For large/sensitive config (Fetch at runtime).

**Exam Trigger:** "Store DB password in Lambda" → Secrets Manager (Not hardcoded).

---

### **8. Lambda with VPC**

**The Rule:** By default, Lambda runs in AWS-managed VPC (Can access internet and AWS services, but NOT your VPC resources).

**To Access VPC Resources (RDS, ElastiCache):**

- Attach Lambda to **your VPC** (Specify subnets + Security Groups).
- Lambda gets **ENI** in your subnet.

**Consequences:**

- **Internet Access:** Lost by default. Need **NAT Gateway** in public subnet to access internet.
- **Cold Start:** Slightly slower (ENI provisioning).

**VPC Endpoint:**

- To access AWS services (S3, DynamoDB) without NAT Gateway → Use **VPC Endpoints**.

**Exam Trigger:** "Lambda needs to access RDS in private subnet" → Attach Lambda to VPC.

---

### **9. Reserved Concurrency & Provisioned Concurrency**

### **A. Reserved Concurrency**

- **The Rule:** Guarantee a number of concurrent executions for a function.
- **Effect:** Reserves capacity (Other functions cannot use it).
- **Cost:** Free.
- **Use Case:** Prevent one function from consuming all account concurrency.

### **B. Provisioned Concurrency**

- **The Rule:** Pre-warmed execution environments (Eliminates cold starts).
- **Cost:** Pay for provisioned capacity (Even if unused).
- **Use Case:** Latency-sensitive applications (APIs requiring < 10 ms).

**Exam Trigger:** "Eliminate cold starts" → Provisioned Concurrency.

---

### **SECTION 2: API GATEWAY (THE FRONTEND)**

*Managed API service for Lambda and backend services.*

### **1. API Gateway Overview**

- **The Rule:** Create, publish, monitor, and secure APIs at any scale.
- **Backend Integrations:** Lambda, HTTP endpoints, AWS services (DynamoDB, S3, Step Functions).
- **API Types:**
    - **REST API:** Full-featured. Supports caching, request validation, API keys.
    - **HTTP API:** Simpler, cheaper (70% less), faster. Best for proxying to Lambda/HTTP.
    - **WebSocket API:** Two-way communication (Chat, Real-time).

**Exam Trigger:** "Build RESTful API for Lambda" → API Gateway.

---

### **2. Deployment Stages**

**The Rule:** APIs deployed to **stages** (e.g., `dev`, `test`, `prod`).

**Features:**

- Each stage has unique URL (e.g., `https://api.example.com/prod`).
- Stage Variables: Environment-specific config (e.g., Lambda alias, backend URL).
- Canary Deployments: Route % of traffic to new stage version.

**Exam Trigger:** "Separate dev and prod APIs" → Stages.

---

### **3. Throttling & Quotas**

**Throttling Limits:**

- **Account-Level:** 10,000 requests per second (Soft limit).
- **Burst:** 5,000 concurrent requests.
- **Method-Level:** Set custom throttle per API method.

**Usage Plans & API Keys:**

- **Usage Plan:** Define throttle and quota per customer/tier (e.g., Free tier: 1,000 req/day, Paid: 100,000).
- **API Key:** Identify client + Associate with Usage Plan.

**Exam Trigger:** "Limit API requests per customer" → Usage Plans + API Keys.

---

### **4. Caching**

**The Rule:** Cache responses at API Gateway to reduce backend load.

**How It Works:**

- Enable cache per stage (0.5 GB - 237 GB).
- TTL: 0-3600 seconds (Default: 300).
- Cache Key: Based on query strings, headers (Configurable).
- Invalidation: Client can send `Cache-Control: max-age=0` header (If authorized).

**Cost:** Hourly charge for cache size.

**Exam Trigger:** "Reduce Lambda invocations for repeated requests" → Enable Caching.

---

### **5. CORS (Cross-Origin Resource Sharing)**

**The Rule:** Browser blocks API calls from different domain unless CORS enabled.

**Scenario:** Frontend at `www.example.com` calls API at `api.example.com` → CORS required.

**Fix:** Enable CORS in API Gateway (Returns `Access-Control-Allow-Origin: *` header).

**Exam Trigger:** "Browser blocks API call" → Enable CORS.

---

### **6. Authorizers**

### **A. Lambda Authorizer (Custom)**

- **The Rule:** Use Lambda function to validate token (e.g., OAuth, SAML).
- **Flow:** Client sends token → API Gateway calls Lambda → Lambda returns IAM policy (Allow/Deny).
- **Caching:** Auth result cached (TTL 0-3600 sec).

### **B. Cognito User Pools**

- **The Rule:** Use Cognito for user authentication.
- **Flow:** User logs in via Cognito → Gets JWT token → Sends token to API Gateway → API Gateway validates token.

**Exam Trigger:** "Custom authentication logic" → Lambda Authorizer. "User sign-up/sign-in" → Cognito.

---

### **7. Integration Types**

- **Lambda Proxy:** Request passed directly to Lambda (Simplest).
- **Lambda Non-Proxy:** Transform request/response (Mapping templates).
- **HTTP Proxy:** Forward to HTTP endpoint.
- **AWS Service:** Directly invoke AWS service (e.g., DynamoDB PutItem).
- **Mock:** Return static response (No backend).

**Exam Trigger:** "Transform API request before Lambda" → Lambda Non-Proxy with Mapping Template.

---

### **SECTION 3: STEP FUNCTIONS (THE ORCHESTRATOR)**

*Coordinate multiple AWS services into workflows.*

### **1. Step Functions Overview**

- **The Rule:** Visual workflow for serverless applications. Chain Lambda, ECS, SNS, DynamoDB, etc.
- **State Machine:** JSON definition (Amazon States Language - ASL).
- **Use Cases:** Order processing, ETL pipelines, Multi-step approvals.

**Benefits:**

- Automatic retry and error handling.
- Built-in state management.
- Visual monitoring.

---

### **2. Workflow Types**

### **A. Standard Workflows**

- **Duration:** Up to **1 year**.
- **Execution Rate:** 2,000 per second.
- **Pricing:** Pay per state transition.
- **Use Case:** Long-running workflows (Data processing, batch jobs).

### **B. Express Workflows**

- **Duration:** Up to **5 minutes**.
- **Execution Rate:** 100,000 per second.
- **Pricing:** Pay per execution (Cheaper for high-volume, short workflows).
- **Types:**
    - **Synchronous:** Wait for response (API Gateway integration).
    - **Asynchronous:** Fire and forget (EventBridge).
- **Use Case:** High-volume event processing (IoT, Streaming).

**Exam Trigger:** "Long-running multi-step workflow" → Standard. "High-volume short tasks" → Express.

---

### **3. State Types**

- **Task:** Do work (Call Lambda, ECS, DynamoDB, etc.).
- **Choice:** Conditional branch (if/else).
- **Parallel:** Execute branches concurrently.
- **Wait:** Delay for fixed time or until timestamp.
- **Succeed/Fail:** End execution.
- **Pass:** Pass input to output (Testing).
- **Map:** Iterate over array (For-each).

**Exam Trigger:** "Run Lambda functions in parallel" → Parallel State. "Conditional workflow" → Choice State.

---

### **4. Error Handling**

**Retry:**

- Automatically retry failed tasks.
- Configurable: Max attempts, backoff rate, interval.

**Catch:**

- Catch specific errors → Route to fallback state.

**Example:**

```json
"Retry": [{"ErrorEquals": ["States.Timeout"], "MaxAttempts": 3}],
"Catch": [{"ErrorEquals": ["States.ALL"], "Next": "FailureState"}]
```

**Exam Trigger:** "Automatically retry Lambda on timeout" → Step Functions Retry.

---

### **5. Integration with Services**

**Optimized Integrations (No Lambda wrapper needed):**

- DynamoDB (PutItem, GetItem, UpdateItem)
- SNS (Publish)
- SQS (SendMessage)
- ECS/Fargate (RunTask)
- Glue (StartJobRun)
- SageMaker (Training, Batch Transform)

**Exam Trigger:** "Chain multiple AWS services without Lambda glue code" → Step Functions.

---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **Timeout > 15 minutes?** → NOT Lambda (Use ECS, Batch, Fargate).
2. **Persistent storage in Lambda?** → NOT /tmp (Use EFS or S3).
3. **Share libraries across Lambda?** → Layers.
4. **Eliminate Lambda cold starts?** → Provisioned Concurrency.
5. **Gradually shift traffic to new Lambda version?** → Aliases with Weighted Routing.
6. **Lambda needs to access RDS?** → Attach to VPC + NAT Gateway for internet.
7. **Limit API requests per customer?** → Usage Plans + API Keys.
8. **Reduce Lambda invocations for repeated API calls?** → API Gateway Caching.
9. **Browser blocks API call?** → Enable CORS.
10. **Coordinate multi-step serverless workflow?** → Step Functions.
11. **Long-running workflow (hours)?** → Step Functions Standard.
12. **High-volume short workflow (seconds)?** → Step Functions Express.

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Long Video Processing" (Lambda Limit)**

**The Situation:** A video processing application needs to transcode uploaded videos. The transcoding process takes **25 minutes** per video. The architect proposes using Lambda triggered by S3 uploads.

**The Options:**

A. Use Lambda with 15-minute timeout and process in batches.

B. Use Lambda to trigger an ECS Fargate task to process the video.

C. Increase Lambda timeout to 30 minutes.

D. Use Step Functions to chain multiple Lambda functions.

**The Logic:**

- **Trap:** Lambda has a hard limit of **15 minutes**. Cannot be increased (Option C is impossible).
- **Trap:** Batching (Option A) still requires 25 minutes per video (Exceeds limit).
- **Trap:** Chaining Lambdas (Option D) doesn't solve the problem (Single video needs 25 minutes continuous).
- **The Fix:** **Option B**. Lambda (triggered by S3) starts an **ECS Fargate task** (No time limit). Fargate processes the video. Pattern: Lambda as orchestrator, ECS/Fargate for long tasks.

---

### **Scenario 2: The "Cold Start Killer" (Provisioned Concurrency)**

**The Situation:** A customer-facing API uses Lambda behind API Gateway. Users report that the **first request** after periods of inactivity takes 2 seconds (cold start), but subsequent requests are < 100 ms. The business requires **all requests** to complete in < 200 ms.

**The Options:**

A. Increase Lambda memory to 10 GB.

B. Use Reserved Concurrency.

C. Use Provisioned Concurrency to keep 10 execution environments warm.

D. Switch to EC2 instances.

**The Logic:**

- **Trap:** More memory (Option A) speeds up execution but doesn't eliminate cold start (Still need to provision environment).
- **Trap:** Reserved Concurrency (Option B) reserves capacity but doesn't pre-warm environments.
- **The Fix:** **Option C**. **Provisioned Concurrency** keeps execution environments **pre-initialized and warm**. Eliminates cold starts. First request is fast. Costs more (Pay for provisioned capacity), but meets latency SLA.

---

### **Scenario 3: The "API Rate Limit" (Usage Plans)**

**The Situation:** A SaaS company offers a public API with three tiers: **Free** (100 requests/day), **Basic** (10,000 requests/day), and **Premium** (unlimited). They need to enforce these limits and identify which customer made each request.

**The Options:**

A. Use Lambda to count requests in DynamoDB and reject if exceeded.

B. Use API Gateway Usage Plans with API Keys for each tier.

C. Use WAF to block excessive requests.

D. Use CloudWatch alarms to alert on high usage.

**The Logic:**

- **Trap:** Lambda + DynamoDB (Option A) is custom code (Reinventing the wheel).
- **Trap:** CloudWatch (Option D) is monitoring, not enforcement.
- **The Fix:** **Option B**. **Usage Plans** define throttle (requests/sec) and quota (requests/day) per tier. **API Keys** identify customers and associate them with a plan. API Gateway enforces limits automatically. Return `429 Too Many Requests` when exceeded.

---

### **Scenario 4: The "Shared Dependencies" (Lambda Layers)**

**The Situation:** A company has 50 Lambda functions that all use the same data validation library (200 MB). Deploying updates requires uploading 200 MB to each function (slow). Developers want a way to share this library across all functions.

**The Options:**

A. Embed the library in each function's deployment package.

B. Create a Lambda Layer containing the library and attach it to all functions.

C. Store the library in S3 and download it in /tmp at runtime.

D. Use EFS to mount shared storage.

**The Logic:**

- **Trap:** Embedding (Option A) is current state (Slow, duplicated storage).
- **Trap:** S3 download (Option C) adds latency and complexity.
- **The Fix:** **Option B**. **Lambda Layers** package shared code (libraries, dependencies) separately. Attach Layer to all 50 functions. Update library once → All functions get update. Reduces deployment time and storage.

---

### **Scenario 5: The "Order Orchestration" (Step Functions)**

**The Situation:** An e-commerce platform's order processing involves: (1) Validate payment (Lambda), (2) Update inventory (DynamoDB), (3) Send confirmation email (SNS), (4) Wait 30 minutes, (5) Check if shipped (Lambda). If any step fails, send alert and stop. Need visual monitoring and automatic retries.

**The Options:**

A. Chain Lambda functions with SNS between each step.

B. Use SQS with Lambda polling each queue.

C. Use Step Functions Standard Workflow to coordinate all steps.

D. Write custom orchestration logic in a single Lambda function.

**The Logic:**

- **Trap:** SNS chaining (Option A) lacks visual monitoring and complex error handling.
- **Trap:** SQS (Option B) requires polling logic (Complex).
- **Trap:** Single Lambda (Option D) hits 15-minute timeout (30-minute wait required).
- **The Fix:** **Option C**. **Step Functions** orchestrates all steps: Task (Lambda), Task (DynamoDB), Task (SNS), **Wait** (30 min), Task (Lambda). Built-in retry/error handling. Visual workflow. Can run for hours. Exactly designed for this.
