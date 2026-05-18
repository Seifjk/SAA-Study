# Serverless

### **SECTION 1: LAMBDA (THE COMPUTE ENGINE)**

*Run code without provisioning servers.*

### **1. Lambda Overview**

> 🔧 *Like:* Google Cloud Functions / Azure Functions — Functions-as-a-Service, run code without servers.

- **The Rule:** Event-driven compute — no servers, code runs on triggers, auto-scales from 0 to thousands of concurrent executions.
- **Pricing:** Pay per request + compute duration (100 ms increments). Free Tier: 1M requests/month + 400,000 GB-seconds.
- **Languages:** Native Node.js, Python, Java, C#, Go, Ruby, PowerShell; any language via Custom Runtime or Container Image (up to 10 GB).

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

- Caller waits for the response (up to 15 min) and receives errors directly — caller must retry. Use cases: API Gateway, ALB, Cognito, Step Functions.

### **B. Asynchronous (Fire and Forget)**

- Caller gets immediate ack; Lambda processes in background. Use cases: S3 events, SNS, EventBridge, SES.
- Retries **2 times** on failure (3 total attempts). Exhausted retries → DLQ (SQS or SNS). **Destinations** route results to SQS/SNS/Lambda/EventBridge on success or failure paths.

**Exam Trigger:** "Lambda invoked by S3" → Asynchronous.

### **C. Event Source Mapping (The Poller)**

- Lambda **polls** the source and pulls record batches (configurable size). Use cases: Kinesis Data Streams, DynamoDB Streams, SQS.
- **Concurrency:** Kinesis/DynamoDB → 1 Lambda per shard; SQS → scales up to 1,000 concurrent.
- **Errors:** Kinesis/DynamoDB → failed batch blocks the shard until success or TTL expires; SQS → failed messages return to queue, then DLQ after retries.

**Exam Trigger:** "Lambda reads from Kinesis" → Event Source Mapping.

---

### **4. Lambda Execution Role & Destinations**

**Execution Role (IAM):**

- Every function assumes an **IAM Execution Role**; trust policy must allow `lambda.amazonaws.com`. Permissions must cover CloudWatch Logs (always) plus any services the function accesses.
- *Exam Trigger:* "Lambda can't write to S3 / needs DynamoDB access" → Check/add permissions on the Execution Role.

**Lambda Destinations (Preferred over DLQ for async):**

- Route async invocation results to SQS/SNS/Lambda/EventBridge on **separate success and failure paths**.
- **Advantage over DLQ:** Captures both success and failure (DLQ = failures only) and includes execution context (request/response).
- *Exam Trigger:* "Route async Lambda results to different services" → Lambda Destinations.

---

### **5. Execution Model**

**Lambda Lifecycle:** Cold Start (first invocation, AWS provisions environment, 100-1000 ms) → Warm Execution (reuses environment, < 10 ms) → Freeze (after idle, stays warm) → Shutdown (after extended idle).

**Optimization:** Minimize cold starts with Provisioned Concurrency, small functions, and lean SDKs. Initialize DB connections **outside the handler** so they're reused across invocations.

---

### **5. Lambda Layers**

**The Rule:** Package libraries/dependencies separately from function code, then attach to multiple functions — shares code, speeds deployments, shrinks deployment packages.

- **Limits:** 5 layers per function; 250 MB total unzipped (layers + function code).

**Exam Trigger:** "Share common libraries across Lambda functions" → Layers.

---

### **6. Lambda Versions & Aliases**

### **A. Versions**

- Immutable snapshot of code + configuration. `$LATEST` = the mutable working version; numbered versions (v1, v2…) are immutable.

### **B. Aliases**

- Mutable pointer to a specific version (e.g., `PROD` → v5). Used for Blue/Green deployment (switch alias to new version) and Weighted Routing (e.g., 90% v1, 10% v2 canary).

**Exam Trigger:** "Gradually shift traffic to new Lambda version" → Aliases with Weighted Routing.

---

### **7. Environment Variables & Configuration**

- **Environment Variables:** Key-value pairs available to function (Max 4 KB total).
- **Encryption:** Encrypted at rest via KMS (Automatic). Can encrypt in transit too (Helpers).
- **Parameter Store / Secrets Manager:** For large/sensitive config (Fetch at runtime).

**Exam Trigger:** "Store DB password in Lambda" → Secrets Manager (Not hardcoded).

---

### **8. Lambda with VPC**

**The Rule:** By default Lambda runs in an AWS-managed VPC (internet + AWS service access, but NOT your VPC resources). To reach VPC resources (RDS, ElastiCache), attach Lambda to **your VPC** (subnets + Security Groups) — it gets an **ENI** in your subnet.

- **Consequences:** Loses internet access by default — need a **NAT Gateway** for internet; slightly slower cold start (ENI provisioning). Use **VPC Endpoints** to reach AWS services (S3, DynamoDB) without a NAT Gateway.

**Exam Trigger:** "Lambda needs to access RDS in private subnet" → Attach Lambda to VPC.

### **Lambda + EFS (Persistent Shared Storage)**

- Lambda can mount an **EFS file system** (via EFS Access Points) for persistent storage shared across invocations. Requires Lambda in the **same VPC** as EFS.
- **vs. /tmp:** /tmp is ephemeral and per-invocation; EFS is persistent and shared. **vs. S3:** EFS is a POSIX file system (read/write, locking); S3 is object storage.

**Exam Trigger:** "Lambda needs persistent shared storage" or "share large files across invocations" → EFS mount.

---

### **Lambda Container Image Support**

- Lambda can run **container images up to 10 GB** from **ECR**; the container must implement the **Lambda Runtime API**. For large dependencies, existing container workflows, or custom runtimes.
- **NOT ECS/Fargate:** Still runs as Lambda (event-driven, 15-min timeout, pay-per-invocation) — just packaged as a container.

**Exam Trigger:** "Deploy existing container to Lambda" or "Lambda with large dependencies" → Container image from ECR.

---

### **SNS + Lambda Fan-Out Pattern**

- SNS Topic triggers **multiple Lambda functions in parallel**, each subscription invoked independently.
- **vs. SNS + SQS Fan-Out:** Use SQS for persistence/retry; use Lambda directly for immediate parallel processing.

**Exam Trigger:** "Process same event with multiple Lambda functions" → SNS fan-out to Lambda.

---

### **9. Reserved Concurrency & Provisioned Concurrency**

- **Reserved Concurrency:** Guarantees concurrent executions for a function (reserves capacity from other functions). Free. Prevents one function from consuming all account concurrency.
- **Provisioned Concurrency:** Pre-warmed execution environments that eliminate cold starts. Pay for provisioned capacity even if unused. For latency-sensitive APIs.

**Exam Trigger:** "Eliminate cold starts" → Provisioned Concurrency.

---

### **SECTION 2: API GATEWAY (THE FRONTEND)**

*Managed API service for Lambda and backend services.*

### **1. API Gateway Overview**

> 🔧 *Like:* Kong / Nginx as an API gateway — managed front door for APIs (routing, auth, throttling).

- **The Rule:** Create, publish, monitor, and secure APIs at any scale.
- **Backend Integrations:** Lambda, HTTP endpoints, AWS services (DynamoDB, S3, Step Functions).
- **API Types:**
    - **REST API:** Full-featured. Supports caching, request validation, API keys, WAF integration, Lambda authorizers (Cognito + custom).
    - **HTTP API:** Simpler, cheaper (70% less), faster. Best for proxying to Lambda/HTTP. Supports JWT authorizers, OIDC. No caching, no request validation.
    - **WebSocket API:** Two-way communication (Chat, Real-time).

| **Feature** | **REST API** | **HTTP API** |
| --- | --- | --- |
| **Cost** | Higher | **~70% cheaper** |
| **Caching** | Yes | No |
| **Request Validation** | Yes | No |
| **WAF Integration** | Yes | No |
| **API Keys / Usage Plans** | Yes | No |
| **Authorizers** | Lambda, Cognito, IAM | JWT, OIDC, IAM |
| **Use Case** | Full API management | Simple, low-cost proxy |

- *Exam Trigger:* "Build RESTful API for Lambda" → API Gateway. "Cheapest API proxy" → HTTP API. "API needs caching/validation/WAF" → REST API.

---

### **2. Deployment Stages**

**The Rule:** APIs deployed to **stages** (`dev`, `test`, `prod`), each with a unique URL. Stage Variables hold environment-specific config (Lambda alias, backend URL); canary deployments route a % of traffic to a new stage version.

**Exam Trigger:** "Separate dev and prod APIs" → Stages.

---

### **3. Throttling & Quotas**

- **Throttling:** Account-level 10,000 req/sec (soft), burst 5,000 concurrent; custom throttle per method.
- **Usage Plans & API Keys:** Usage Plan defines throttle + quota per customer/tier; API Key identifies the client and ties it to a plan.

**Exam Trigger:** "Limit API requests per customer" → Usage Plans + API Keys.

---

### **4. Caching**

**The Rule:** Cache responses at API Gateway to reduce backend load. Enabled per stage (0.5–237 GB), TTL 0–3600 sec (default 300), cache key from query strings/headers. Authorized clients invalidate via `Cache-Control: max-age=0`. Hourly charge for cache size.

**Exam Trigger:** "Reduce Lambda invocations for repeated requests" → Enable Caching.

---

### **5. CORS (Cross-Origin Resource Sharing)**

**The Rule:** Browsers block API calls from a different domain unless CORS is enabled (e.g., frontend `www.example.com` calling API `api.example.com`). Fix: enable CORS in API Gateway (returns `Access-Control-Allow-Origin` header).

**Exam Trigger:** "Browser blocks API call" → Enable CORS.

---

### **6. Authorizers**

- **Lambda Authorizer (Custom):** A Lambda validates the token (OAuth, SAML) and returns an Allow/Deny IAM policy. Auth result cached (TTL 0–3600 sec).
- **Cognito User Pools:** User logs in via Cognito → gets JWT token → API Gateway validates it.

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

> 🔧 *Like:* Airflow / Temporal — workflow orchestration, chains steps with state and retries.

- **The Rule:** Visual workflow chaining Lambda, ECS, SNS, DynamoDB, etc. State machine defined in JSON (Amazon States Language).
- **Use Cases:** Order processing, ETL pipelines, multi-step approvals. **Benefits:** automatic retry/error handling, built-in state management, visual monitoring.

---

### **2. Workflow Types**

- **Standard Workflows:** Up to **1 year**, 2,000 executions/sec, pay per state transition. For long-running workflows (data processing, batch jobs).
- **Express Workflows:** Up to **5 minutes**, 100,000 executions/sec, pay per execution (cheaper for high-volume short workflows). Synchronous (API Gateway) or Asynchronous (EventBridge). For high-volume event processing (IoT, streaming).

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

- **Retry:** Automatically retry failed tasks — configurable max attempts, backoff rate, interval.
- **Catch:** Catch specific errors and route to a fallback state.

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

### **6. Data Flow Control**

- **InputPath:** Filter which part of the input JSON to pass to the task (select a subset).
- **ResultPath:** Where to put the task's result within the original input (merge result into state).
- **OutputPath:** Filter the final output before passing to the next state.
- **Key Insight:** These three together control what data flows between states — critical for complex workflows.

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
13. **Lambda needs persistent shared storage?** → Mount EFS (Lambda must be in VPC).
14. **Lambda with large dependencies or existing container?** → Container image from ECR (up to 10 GB).
15. **Process same event with multiple Lambda functions?** → SNS fan-out to Lambda.
16. **Lambda can't access S3/DynamoDB?** → Check Execution Role IAM permissions.
17. **Simple cheap API proxy?** → HTTP API (~70% cheaper than REST API).
18. **API needs caching, validation, WAF?** → REST API.
19. **Route async Lambda results (success + failure)?** → Lambda Destinations.

---

# **REAL EXAM SCENARIOS**

### Scenario 1

**The Situation:** A video processing application needs to transcode uploaded videos. The transcoding process takes **25 minutes** per video. The architect proposes using Lambda triggered by S3 uploads.

**The Options:**

A. Use Lambda with 15-minute timeout and process in batches.

B. Use Lambda to trigger an ECS Fargate task to process the video.

C. Increase Lambda timeout to 30 minutes.

D. Use Step Functions to chain multiple Lambda functions.

**The Logic:**

- **Trap A — Lambda + process in batches:** A single video transcode is a 25-minute *continuous* job — it can't be sliced into sub-15-minute pieces. Batching changes nothing.
- **Trap C — Increase Lambda timeout to 30 minutes:** **Impossible.** Lambda's timeout is a hard ceiling of **15 minutes** — it cannot be raised. This option describes something that doesn't exist.
- **Trap D — Chain Lambdas via Step Functions:** Chaining helps when *separate steps* each fit in 15 min. One video needs 25 minutes of *uninterrupted* processing — no individual Lambda in the chain can do it.
- **The Fix — Option B:** S3 upload triggers a **Lambda that launches an ECS Fargate task** — Fargate has **no execution time limit** and handles the 25-minute transcode. Classic pattern: **Lambda as the lightweight trigger/orchestrator, Fargate for long-running compute.**

---

### Scenario 2

**The Situation:** A customer-facing API uses Lambda behind API Gateway. Users report that the **first request** after periods of inactivity takes 2 seconds (cold start), but subsequent requests are < 100 ms. The business requires **all requests** to complete in < 200 ms.

**The Options:**

A. Increase Lambda memory to 10 GB.

B. Use Reserved Concurrency.

C. Use Provisioned Concurrency to keep 10 execution environments warm.

D. Switch to EC2 instances.

**The Logic:**

- **Trap A — Increase memory to 10 GB:** More memory gives more CPU, so *execution* is faster — but the **cold start** (downloading code, spinning up the runtime, initializing) still happens. The 2-second first-request delay remains.
- **Trap B — Reserved Concurrency:** A common mix-up. Reserved Concurrency *caps/guarantees* how many concurrent executions a function can have — it does **not** pre-warm anything. Cold starts still occur. (Reserved = a capacity limit; Provisioned = pre-warmed environments.)
- **Trap D — Switch to EC2:** Always-on EC2 has no cold starts, but it throws away the entire serverless model — you now manage instances, patching, scaling. Massive over-correction for a latency tweak.
- **The Fix — Option C:** **Provisioned Concurrency** keeps a set number of execution environments **pre-initialized and warm**, so even the first request after idle is fast (< 200 ms). You pay for the provisioned capacity, but it's the only option that actually eliminates cold starts.

---

### Scenario 3

**The Situation:** A SaaS company offers a public API with three tiers: **Free** (100 requests/day), **Basic** (10,000 requests/day), and **Premium** (unlimited). They need to enforce these limits and identify which customer made each request.

**The Options:**

A. Use Lambda to count requests in DynamoDB and reject if exceeded.

B. Use API Gateway Usage Plans with API Keys for each tier.

C. Use WAF to block excessive requests.

D. Use CloudWatch alarms to alert on high usage.

**The Logic:**

- **Trap A — Lambda counts requests in DynamoDB:** This *works* but you're hand-building rate limiting that API Gateway already provides natively — custom counters, atomic increments, race conditions, per-tier logic. Reinventing the wheel.
- **Trap C — WAF to block excessive requests:** WAF rate-based rules block by **IP** at a single threshold. They can't enforce *per-customer* tiered daily quotas (100 / 10,000 / unlimited) or identify *which customer* made a call. Wrong granularity.
- **Trap D — CloudWatch alarms on high usage:** CloudWatch only *observes and alerts*. It cannot *enforce* a limit or reject a request. Monitoring ≠ enforcement.
- **The Fix — Option B:** **API Gateway Usage Plans** define a **throttle** (req/sec) and **quota** (req/day) per tier; **API Keys** identify each customer and bind them to a plan. API Gateway enforces it automatically and returns `429 Too Many Requests` when exceeded — built-in, no custom code.

---

### Scenario 4

**The Situation:** A company has 50 Lambda functions that all use the same data validation library (200 MB). Deploying updates requires uploading 200 MB to each function (slow). Developers want a way to share this library across all functions.

**The Options:**

A. Embed the library in each function's deployment package.

B. Create a Lambda Layer containing the library and attach it to all functions.

C. Store the library in S3 and download it in /tmp at runtime.

D. Use EFS to mount shared storage.

**The Logic:**

- **Trap A — Embed the library in each package:** This is the *current* painful state — 200 MB duplicated into 50 deployment packages, re-uploaded on every update. The problem itself, not a fix.
- **Trap C — Store in S3, download to /tmp at runtime:** Adds **cold-start latency** (downloading 200 MB on init) and custom bootstrap code. Fragile and slow compared to a native mechanism.
- **Trap D — Mount shared storage via EFS:** EFS *can* hold shared code, but it's meant for large/persistent shared *data*, adds VPC + mount configuration, and is overkill for distributing a library. Layers are the purpose-built answer.
- **The Fix — Option B:** A **Lambda Layer** packages the shared library once and attaches to all 50 functions. Update the layer version once → every function picks it up. Deployment packages shrink, upload time drops — exactly what Layers exist for.

---

### Scenario 5

**The Situation:** An e-commerce platform's order processing involves: (1) Validate payment (Lambda), (2) Update inventory (DynamoDB), (3) Send confirmation email (SNS), (4) Wait 30 minutes, (5) Check if shipped (Lambda). If any step fails, send alert and stop. Need visual monitoring and automatic retries.

**The Options:**

A. Chain Lambda functions with SNS between each step.

B. Use SQS with Lambda polling each queue.

C. Use Step Functions Standard Workflow to coordinate all steps.

D. Write custom orchestration logic in a single Lambda function.

**The Logic:**

- **Trap A — Chain Lambdas with SNS between steps:** You'd hand-build the wiring, the retries, and the failure handling — and there's **no visual monitoring** of where an order is in the flow. The requirement explicitly asks for visual monitoring and automatic retries.
- **Trap B — SQS with Lambda polling each queue:** Possible, but you're manually constructing an orchestration out of queues and poll logic — complex, and still no visual workflow or built-in step-level retry/error semantics.
- **Trap D — Single Lambda doing all orchestration:** Step 4 is a **30-minute wait** — a single Lambda invocation caps at **15 minutes**, so it physically cannot hold the workflow open. Impossible.
- **The Fix — Option C:** **Step Functions (Standard Workflow)** models the whole flow — Task → Task → Task → **Wait (30 min)** → Task — with **built-in retry/catch**, a **visual** execution graph, and the ability to run for up to a year. Purpose-built for multi-step orchestration with waits and error handling.
