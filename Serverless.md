# Serverless

> **Scope note (SAA-C03):** AWS's exam guide tests these services with the verb *"determine WHEN to use."* You pick the service/mode for a scenario — you do **not** configure internals. Anything build-it level (ASL JSON, mapping templates, deploy pipelines) has been deliberately cut. If it's not here, the exam doesn't ask it.

---

### **SECTION 1: LAMBDA**

*Run code without provisioning servers. Event-driven, auto-scales from 0.*

### **1. Lambda Overview**

- **What it is:** Event-driven compute. No servers, code runs on triggers, auto-scales to thousands of concurrent executions.
- **Pricing:** Pay per request + compute duration (memory × time). No charge when idle.

---

### **2. Lambda Limits (MEMORIZE — these drive the traps)**

| **Limit** | **Value** | **Exam Trap** |
| --- | --- | --- |
| **Execution Timeout** | **15 minutes max** | "Long-running job" → NOT Lambda (ECS/Batch/Fargate) |
| **Memory** | 128 MB – 10,240 MB | More memory = more CPU |
| **Deployment Package** | 50 MB zipped / 250 MB unzipped | Large deps → Layers or Container image |
| **Concurrent Executions** | 1,000 per region (soft) | Throttling → raise limit or Reserved Concurrency |
| **/tmp Storage** | 512 MB – 10,240 MB | **Ephemeral** — not persistent |
| **Invocation Payload** | **6 MB** sync / **256 KB** async | Large payload → use S3 |

**Triggers:** "Task takes 20 min" → not Lambda · "Needs persistent storage" → not /tmp (EFS/S3) · "300 MB package" → Container image or Layers.

---

### **3. Invocation Types** *(know which trigger maps to which type)*

- **Synchronous (caller waits):** API Gateway, ALB, Cognito, Step Functions. Caller gets errors directly and must retry.
- **Asynchronous (fire & forget):** S3 events, SNS, EventBridge, SES. Lambda retries **2×** (3 total), then → DLQ (SQS/SNS).
- **Event Source Mapping (Lambda polls the source):** Kinesis Data Streams, DynamoDB Streams, SQS. Lambda pulls batches.
    - Concurrency: Kinesis/DynamoDB = **1 Lambda per shard**; SQS scales to 1,000 concurrent.

**Triggers:** "Lambda invoked by S3" → async · "Lambda reads from Kinesis/SQS" → Event Source Mapping.

---

### **4. Execution Role & Destinations**

- **Execution Role (IAM):** Every function has one. Must grant access to whatever the function touches (always includes CloudWatch Logs).
    - *Trigger:* "Lambda can't write to S3 / access DynamoDB" → fix the **Execution Role** permissions.
- **Destinations (async results):** Route success **and** failure to SQS/SNS/Lambda/EventBridge. Captures both outcomes (DLQ = failures only).
    - *Trigger:* "Route async Lambda results by success/failure" → **Destinations**.

---

### **5. Cold Starts & Concurrency**

- **Cold start:** First invocation provisions the environment (slower); subsequent invocations reuse it (warm, fast).
- **Reserved Concurrency:** *Caps/guarantees* concurrency for a function. Free. Does **NOT** pre-warm. Prevents one function eating the account limit.
- **Provisioned Concurrency:** Pre-warmed environments that **eliminate cold starts**. Costs money. For latency-sensitive APIs.

**Trigger:** "Eliminate cold starts / consistent low latency" → **Provisioned Concurrency** (NOT Reserved).

---

### **6. Layers**

- Package shared libraries/dependencies separately, attach to many functions. Shrinks deployment packages, shares code.

**Trigger:** "Share common libraries across many Lambda functions" → **Layers**.

---

### **7. Versions & Aliases** *(recognition only)*

- **Version:** Immutable snapshot of code + config (`$LATEST` is the editable copy; published versions are frozen).
- **Alias:** A named pointer to a version (e.g., `PROD` → v5). Lets you repoint without callers changing config; supports gradual traffic shift between two versions.

**Trigger:** "Shift traffic gradually / instant rollback to previous Lambda version" → **Alias**.

---

### **8. Secrets & Config**

- Store DB passwords / API keys in **Secrets Manager** or **SSM Parameter Store** — fetched at runtime, never hardcoded. Env vars are encrypted at rest with KMS.

**Trigger:** "Store DB password for Lambda securely" → **Secrets Manager**.

---

### **9. Lambda with VPC**

- By default Lambda runs in an AWS-managed network (has internet, no access to your VPC). To reach **VPC resources (RDS, ElastiCache)** → attach Lambda to your VPC (subnets + SG); it gets an **ENI** in your subnet.
- **Side effect:** loses default internet access → need a **NAT Gateway** for internet, or **VPC Endpoints** to reach S3/DynamoDB privately.

**Trigger:** "Lambda needs to access RDS in a private subnet" → attach Lambda to the VPC.

---

### **10. Lambda + EFS / Containers**

- **EFS mount:** Persistent, **shared** storage across invocations (vs. /tmp = ephemeral/per-invocation). Lambda must be in the **same VPC** as EFS.
    - *Trigger:* "Lambda needs persistent shared storage / share large files across invocations" → mount **EFS**.
- **Container image:** Lambda can run a container image up to **10 GB** from ECR. Still Lambda (15-min cap, event-driven) — **not** ECS/Fargate.
    - *Trigger:* "Deploy existing container to Lambda / very large dependencies" → container image from ECR.

---

### **11. SNS + Lambda Fan-Out**

- SNS topic triggers **multiple Lambda functions in parallel**. Use SNS+SQS instead when you need persistence/retry buffering.

**Trigger:** "Process the same event with multiple Lambda functions" → **SNS fan-out**.

---

### **SECTION 2: API GATEWAY**

*Managed front door for APIs — exposes Lambda, HTTP endpoints, and AWS services.*

### **1. REST API vs HTTP API** *(the core distinction the exam draws)*

| **Feature** | **REST API** | **HTTP API** |
| --- | --- | --- |
| **Cost** | Higher | **~70% cheaper** |
| **Caching** | Yes | No |
| **Request Validation / API Keys / Usage Plans** | Yes | No |
| **WAF Integration** | Yes | No |
| **Authorizers** | Lambda, Cognito, IAM | JWT, OIDC, IAM |
| **Use for** | Full API management | Simple, low-cost proxy |

*(WebSocket API also exists — for two-way real-time, e.g. chat.)*

**Triggers:** "RESTful API for Lambda" → API Gateway · "Cheapest/simplest API proxy" → **HTTP API** · "API needs caching / validation / WAF" → **REST API**.

---

### **2. Stages, Throttling, Caching**

- **Stages:** Deploy to `dev`/`test`/`prod`, each its own URL. → "Separate dev and prod APIs."
- **Usage Plans + API Keys:** Per-customer throttle (req/sec) + quota (req/day); API Key identifies the client. → "Limit/meter requests **per customer/tier**."
- **Caching:** Cache responses to cut backend load (default TTL 300s). → "Reduce Lambda invocations for repeated identical requests."

---

### **3. Authorizers & CORS**

- **Cognito User Pools:** User signs in → gets JWT → API Gateway validates it. → "User sign-in / managed auth."
- **Lambda Authorizer:** A Lambda validates a token and returns allow/deny. → "Custom authentication logic" (OAuth/SAML/third-party).
- **CORS:** Enable it when a browser on one domain calls an API on another. → "Browser blocks cross-origin API call."

---

### **SECTION 3: STEP FUNCTIONS**

*Serverless orchestration — coordinate multiple steps into one workflow.*

> Exam scope: **determine WHEN to use it.** Internals (state types, ASL JSON, InputPath/ResultPath) are **not** tested.

- **What it is:** Coordinates steps (Lambda, ECS, SNS, DynamoDB…) into a workflow with **built-in retries, error handling, and visual monitoring**.
- **WHEN to pick it:** multi-step workflow · needs retry/error handling · needs to **wait** between steps · needs visual tracking of where execution is. (Also: avoids hand-built "glue" orchestration.)

**Standard vs Express** — the only sub-distinction tested:

- **Standard:** long-running, up to **1 year**. Order fulfillment, ETL, human-approval steps.
- **Express:** high-volume, short, up to **5 minutes**. Streaming / high-frequency event processing.

**Triggers:** "Coordinate/orchestrate multi-step workflow with retries + visual monitoring" → Step Functions · "Long-running (hours/days)" → **Standard** · "High-volume short tasks" → **Express**.

---

### **Exam Summary Cheat Sheet - Practice (Fill In the Blank)**

1. **Timeout > 15 minutes?** → Fargate ECS batch
2. **Persistent storage in Lambda?** → not /tmp, EFS or S3
3. **Persistent *shared* storage across invocations?** → EFS
4. **Share libraries across many functions?** → Layers
5. **Eliminate cold starts?** → Provisioned Concurrency
6. **Stop one function from eating all concurrency?** → Reserved
7. **Lambda → RDS in private subnet?** → VPC and Interface Endpoint
8. **Lambda can't access S3/DynamoDB?** → IAM Role needed
9. **Route async Lambda results (success + failure)?** → Destinations
10. **Process same event with multiple functions?** → SNS+Lambda Fanout
11. **Lambda with huge deps / existing container?** → Lamda with ECR?
12. **Cheapest/simplest API proxy?** → HTTP API
13. **API needs caching / validation / WAF?** → RESTful API
14. **Limit/meter API requests per customer?** Usage Plans + API key
15. **Reduce Lambda calls for repeated requests?** → API Gateway Cachine
16. **Browser blocks cross-origin API call?** → enable CORS
17. **User sign-in at the API?** → Cognito/IAM or custom bullshit or Lambda Authorizer.
18. **Coordinate a multi-step workflow (retries + visual)?** → Step Functions
19. **Long-running workflow (hours)?** → Standard.

### **Exam Summary Cheat Sheet - Answer Key**

1. **Timeout > 15 minutes?** → NOT Lambda (ECS, Batch, Fargate).
2. **Persistent storage in Lambda?** → NOT /tmp (EFS or S3).
3. **Persistent *shared* storage across invocations?** → mount **EFS** (Lambda in VPC).
4. **Share libraries across many functions?** → **Layers**.
5. **Eliminate cold starts?** → **Provisioned** Concurrency (NOT Reserved).
6. **Stop one function from eating all concurrency?** → **Reserved** Concurrency.
7. **Lambda → RDS in private subnet?** → attach Lambda to **VPC** (+ NAT for internet).
8. **Lambda can't access S3/DynamoDB?** → fix **Execution Role** IAM.
9. **Route async Lambda results (success + failure)?** → **Destinations**.
10. **Process same event with multiple functions?** → **SNS fan-out**.
11. **Lambda with huge deps / existing container?** → **Container image** from ECR (≤10 GB).
12. **Cheapest/simplest API proxy?** → **HTTP API** (~70% cheaper).
13. **API needs caching / validation / WAF?** → **REST API**.
14. **Limit/meter API requests per customer?** → **Usage Plans + API Keys**.
15. **Reduce Lambda calls for repeated requests?** → API Gateway **Caching**.
16. **Browser blocks cross-origin API call?** → enable **CORS**.
17. **User sign-in at the API?** → **Cognito User Pools**. Custom token logic → **Lambda Authorizer**.
18. **Coordinate a multi-step workflow (retries + visual)?** → **Step Functions**.
19. **Long-running workflow (hours)?** → **Standard**. High-volume short → **Express**.

---

# **REAL EXAM SCENARIOS**

### Scenario 1

**The Situation:** A video processing application needs to transcode uploaded videos. The transcoding process takes **25 minutes** per video. The architect proposes using Lambda triggered by S3 uploads.

**The Options:**

A. Use Lambda with 15-minute timeout and process in batches.

B. Increase Lambda timeout to 30 minutes.

C. Use Lambda to trigger an ECS Fargate task to process the video.

D. Use Step Functions to chain multiple Lambda functions.

**The Logic:**

- **Trap A — Lambda + process in batches:** A single video transcode is a 25-minute *continuous* job — it can't be sliced into sub-15-minute pieces. Batching changes nothing.
- **Trap B — Increase Lambda timeout to 30 minutes:** **Impossible.** Lambda's timeout is a hard ceiling of **15 minutes** — it cannot be raised. This option describes something that doesn't exist.
- **Trap D — Chain Lambdas via Step Functions:** Chaining helps when *separate steps* each fit in 15 min. One video needs 25 minutes of *uninterrupted* processing — no individual Lambda in the chain can do it.
- **The Fix — Option C:** S3 upload triggers a **Lambda that launches an ECS Fargate task** — Fargate has **no execution time limit** and handles the 25-minute transcode. Classic pattern: **Lambda as the lightweight trigger, Fargate for long-running compute.**

---

### Scenario 2

**The Situation:** A customer-facing API uses Lambda behind API Gateway. Users report that the **first request** after periods of inactivity takes 2 seconds (cold start), but subsequent requests are < 100 ms. The business requires **all requests** to complete in < 200 ms.

**The Options:**

A. Increase Lambda memory to 10 GB.

B. Use Provisioned Concurrency to keep 10 execution environments warm.

C. Use Reserved Concurrency.

D. Switch to EC2 instances.

**The Logic:**

- **Trap A — Increase memory to 10 GB:** More memory gives more CPU, so *execution* is faster — but the **cold start** (downloading code, spinning up the runtime, initializing) still happens. The 2-second first-request delay remains.
- **Trap C — Reserved Concurrency:** A common mix-up. Reserved Concurrency *caps/guarantees* how many concurrent executions a function can have — it does **not** pre-warm anything. Cold starts still occur. (Reserved = a capacity limit; Provisioned = pre-warmed environments.)
- **Trap D — Switch to EC2:** Always-on EC2 has no cold starts, but it throws away the entire serverless model — you now manage instances, patching, scaling. Massive over-correction for a latency tweak.
- **The Fix — Option B:** **Provisioned Concurrency** keeps a set number of execution environments **pre-initialized and warm**, so even the first request after idle is fast (< 200 ms). You pay for the provisioned capacity, but it's the only option that actually eliminates cold starts.

---

### Scenario 3

**The Situation:** A SaaS company offers a public API with three tiers: **Free** (100 requests/day), **Basic** (10,000 requests/day), and **Premium** (unlimited). They need to enforce these limits and identify which customer made each request.

**The Options:**

A. Use Lambda to count requests in DynamoDB and reject if exceeded.

B. Use WAF to block excessive requests.

C. Use CloudWatch alarms to alert on high usage.

D. Use API Gateway Usage Plans with API Keys for each tier.

**The Logic:**

- **Trap A — Lambda counts requests in DynamoDB:** This *works* but you're hand-building rate limiting that API Gateway already provides natively — custom counters, atomic increments, race conditions, per-tier logic. Reinventing the wheel.
- **Trap B — WAF to block excessive requests:** WAF rate-based rules block by **IP** at a single threshold. They can't enforce *per-customer* tiered daily quotas (100 / 10,000 / unlimited) or identify *which customer* made a call. Wrong granularity.
- **Trap C — CloudWatch alarms on high usage:** CloudWatch only *observes and alerts*. It cannot *enforce* a limit or reject a request. Monitoring ≠ enforcement.
- **The Fix — Option D:** **API Gateway Usage Plans** define a **throttle** (req/sec) and **quota** (req/day) per tier; **API Keys** identify each customer and bind them to a plan. API Gateway enforces it automatically and returns `429 Too Many Requests` when exceeded — built-in, no custom code.

---

### Scenario 4

**The Situation:** A company has 50 Lambda functions that all use the same data validation library (200 MB). Deploying updates requires uploading 200 MB to each function (slow). Developers want a way to share this library across all functions.

**The Options:**

A. Embed the library in each function's deployment package.

B. Store the library in S3 and download it in /tmp at runtime.

C. Create a Lambda Layer containing the library and attach it to all functions.

D. Use EFS to mount shared storage.

**The Logic:**

- **Trap A — Embed the library in each package:** This is the *current* painful state — 200 MB duplicated into 50 deployment packages, re-uploaded on every update. The problem itself, not a fix.
- **Trap B — Store in S3, download to /tmp at runtime:** Adds **cold-start latency** (downloading 200 MB on init) and custom bootstrap code. Fragile and slow compared to a native mechanism.
- **Trap D — Mount shared storage via EFS:** EFS *can* hold shared code, but it's meant for large/persistent shared *data*, adds VPC + mount configuration, and is overkill for distributing a library. Layers are the purpose-built answer.
- **The Fix — Option C:** A **Lambda Layer** packages the shared library once and attaches to all 50 functions. Update the layer version once → every function picks it up. Deployment packages shrink, upload time drops — exactly what Layers exist for.

---

### Scenario 5

**The Situation:** An e-commerce platform's order processing involves: (1) Validate payment (Lambda), (2) Update inventory (DynamoDB), (3) Send confirmation email (SNS), (4) Wait 30 minutes, (5) Check if shipped (Lambda). If any step fails, send alert and stop. Need visual monitoring and automatic retries.

**The Options:**

A. Use Step Functions Standard Workflow to coordinate all steps.

B. Chain Lambda functions with SNS between each step.

C. Use SQS with Lambda polling each queue.

D. Write custom orchestration logic in a single Lambda function.

**The Logic:**

- **Trap B — Chain Lambdas with SNS between steps:** You'd hand-build the wiring, the retries, and the failure handling — and there's **no visual monitoring** of where an order is in the flow. The requirement explicitly asks for visual monitoring and automatic retries.
- **Trap C — SQS with Lambda polling each queue:** Possible, but you're manually constructing an orchestration out of queues and poll logic — complex, and still no visual workflow or built-in step-level retry/error semantics.
- **Trap D — Single Lambda doing all orchestration:** Step 4 is a **30-minute wait** — a single Lambda invocation caps at **15 minutes**, so it physically cannot hold the workflow open. Impossible.
- **The Fix — Option A:** **Step Functions (Standard Workflow)** models the whole flow — Task → Task → Task → **Wait (30 min)** → Task — with **built-in retry/catch**, a **visual** execution graph, and the ability to run for up to a year. Purpose-built for multi-step orchestration with waits and error handling.
