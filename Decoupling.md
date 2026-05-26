# Decoupling (Queues & Messaging)

> 📊 **Visual reference:** [DecouplingDiagrams.md](./DecouplingDiagrams.md) — 7 Mermaid diagrams: SQS flow + visibility timeout, SNS fan-out (direct vs +SQS), Kinesis shards/KCL, Standard vs Enhanced Fan-Out, Kinesis vs Firehose, EventBridge architecture, DLQ pattern.

### **SECTION 1: SQS (SIMPLE QUEUE SERVICE)**

*Managed message queue for decoupling applications.*

### **1. SQS Overview**

> 🔧 *Like:* RabbitMQ / a managed message queue — producers drop messages, consumers poll them.

- **The Rule:** Fully managed message queue. **Pull-based** (Consumers poll for messages).
- **Use Case:** Decouple producers from consumers (Async processing).
- **Scale:** Unlimited throughput, unlimited messages in queue.
- **Retention:** 1 minute - 14 days (Default: 4 days).
- **Message Size:** Max **256 KB**. For larger payloads → **SQS Extended Client Library** (stores payload in S3, sends S3 reference in SQS message).

**Pattern:** Producer → SQS Queue → Consumer (Poll)

---

### ⭐ **Mental Model: A queue is never the destination**

SQS is a **shock absorber** between a producer and a consumer fleet — it is *always* in the middle of a chain, never the end.

**The universal pattern:**

```
Producer → SQS Queue → Consumer fleet → Final data store
                                        (DB / S3 / API / another queue)
```

**Why the queue exists at all:**
- Producers push at **unpredictable rates** (Friday 6 PM surge could be 10× Tuesday morning).
- Consumers process at a **steady rate** based on how many workers you have.
- Without SQS, a producer spike overwhelms the workers or forces the producer to retry/fail.
- **With SQS**, the queue absorbs the spike, workers drain it at their own pace, and you can **auto-scale the worker fleet on queue depth** (`ApproximateNumberOfMessagesVisible` — the HA chapter trigger).

**The consumer in an SAA question is almost always one of these four:**
1. **Lambda** (event-source mapping polls SQS for you — most common modern answer)
2. **Auto Scaling Group of EC2** (classic worker pattern, scales on queue depth)
3. **ECS / Fargate tasks** (containerized workers)
4. **On-prem app** polling via the SDK (hybrid use cases)

**Full chains for the real-world examples in this chapter:**

| Use case | Full pipeline |
|---|---|
| **Uber driver pings** (SQS Standard) | Driver phones → SQS Standard → Lambda / EC2 ASG → DynamoDB `{driver_id → lat,lng,updated_at}` → Rider app reads DynamoDB |
| **Bank transactions** (SQS FIFO, Group ID = Account ID) | Banking app → SQS FIFO → Lambda worker → RDS Aurora (debits/credits) |
| **Video transcoder** (Visibility Timeout) | Upload service → SQS → EC2 ASG running FFmpeg → S3 (output MP4) + DynamoDB (job status) |
| **Shopify orders** (DLQ) | Checkout → SQS → Lambda processor → on success: orders DB + payment API; on 3 failures: DLQ for triage |
| **OrderPlaced fan-out** (SNS+SQS) | Checkout → SNS topic → 4 SQS queues → 4 Lambda fleets → 4 destinations (SES email, DynamoDB inventory, Redshift analytics, S3 audit) |

**On the exam:** when you see SQS in a question, **mentally draw the full chain** — what feeds it, what drains it, where the data ends up. That's how you spot traps:
- "Producer is overwhelming the DB" → insert SQS between them.
- "Messages reappear after 30s" → Visibility Timeout vs consumer runtime.
- "One downstream is down and breaks the whole upload" → SNS+SQS fan-out so each consumer has its own buffer.

---

### **2. Queue Types**

### **A. Standard Queue (The Default)**

- Unlimited throughput, **best-effort ordering**, **at-least-once delivery** (duplicates possible). Use when order doesn't matter and the consumer handles duplicates.

**Exam Trigger:** "Highest throughput, Duplicates acceptable" → Standard Queue.

*Real-world example:* Uber drops 100K "driver location updated" pings/sec into an SQS Standard queue — a duplicate ping is harmless, and order doesn't matter (only the latest position does).

---

### **B. FIFO Queue (The Ordered)**

- **300 msg/sec** (3,000 with batching), **strict ordering**, **exactly-once processing** (dedup within 5-min window). Queue name must end with `.fifo`.
- **Message Group ID:** Order preserved within a group, parallel processing across groups. Dedup via Message Deduplication ID or content-based hash.
- Use when order matters (bank transactions, trading orders).

**Exam Trigger:** "Strict ordering, No duplicates" → FIFO Queue.

*Real-world example:* A bank uses SQS FIFO with Message Group ID = Account ID — for one account, "deposit $100" MUST process before "withdraw $80", and the deposit must never be applied twice.

---

### **SQS Standard vs. FIFO**

| **Feature** | **Standard** | **FIFO** |
| --- | --- | --- |
| **Throughput** | Unlimited | 300 msg/sec (3,000 with batching) |
| **Ordering** | Best-effort | **Strict** |
| **Delivery** | At-least-once (Duplicates possible) | **Exactly-once** |
| **Use Case** | High throughput, Order doesn't matter | Order matters, No duplicates |

---

### **3. Polling Methods**

- **Short Polling (default):** Returns immediately even if empty → many empty responses → wasted API calls and higher cost.
- **Long Polling:** Waits up to **20 seconds** for messages. Reduces empty responses → lower cost and latency. Set `ReceiveMessageWaitTimeSeconds` > 0 (max 20).

**Exam Trigger:** "Reduce SQS API calls and cost" → Enable Long Polling.

*Real-world example:* A startup's Lambda was polling every 1 sec → 86,400 calls/day for ~500 real messages. Switching to Long Polling (`WaitTimeSeconds = 20`) cut API calls ~95% and lowered the AWS bill.

---

### **4. Visibility Timeout**

**The Rule:** When a consumer receives a message, it becomes **invisible** to other consumers for the Visibility Timeout. The consumer must delete it before the timeout expires, or the message becomes **visible again** for another consumer.

- **Default:** 30 sec. **Max:** 12 hours. `ChangeMessageVisibility` API extends the timeout if processing runs long.

**Exam Trap:** "Message processed multiple times" → Visibility Timeout too short — increase it.

*Real-world example:* A video transcoder takes 8 min per job, but the queue's Visibility Timeout is 30 sec — at the 30-sec mark the message reappears and a second worker starts the same job. Bump the timeout to 10 min.

---

### **5. Dead Letter Queue (DLQ)**

**The Rule:** After a message fails to process `MaxReceiveCount` times (e.g., 3), it's moved to a **Dead Letter Queue** (just another SQS queue, Standard or FIFO) for inspection.

- Use to debug failed messages and prevent poison messages from blocking the queue.

**Exam Trigger:** "Handle messages that repeatedly fail processing" → DLQ.

*Real-world example:* Shopify's order processor keeps crashing on an order with a malformed JSON field — after 3 failed attempts the message is routed to a DLQ so an engineer can inspect it without blocking the live queue.

---

### **6. Delay Queue & Message Timers**

### **A. Delay Queue**

- **The Rule:** All messages delayed before becoming visible (0-15 minutes).
- **Use Case:** Delay all messages (e.g., Order confirmation email after 5 minutes).

### **B. Message Timer (Per-Message Delay)**

- **The Rule:** Delay individual messages (Overrides Delay Queue setting).
- **Use Case:** Different delays per message (e.g., Retry after 1 min, 5 min, 15 min).

**Exam Trigger:** "Delay processing of specific messages" → Message Timer.

*Real-world example:* A retry pipeline requeues failed jobs with escalating per-message delays — 1 min on the first retry, 5 min on the second, 15 min on the third — using Message Timer rather than a fixed Delay Queue.

---

### **7. SQS Security**

- **Encryption:** In-transit via HTTPS (default); at-rest via KMS (optional).
- **Access Control:** IAM policies control who can send/receive; SQS Access Policies (resource-based) enable cross-account access and let S3/SNS send messages.

**Exam Example:** "S3 bucket sends event to SQS queue in different account" → SQS Access Policy.

*Real-world example:* An S3 bucket in the data account drops "new file uploaded" events into an SQS queue owned by the security account's incident-response team — the queue's resource-based Access Policy grants the bucket permission to send.

---

### **SECTION 2: SNS (SIMPLE NOTIFICATION SERVICE)**

*Managed pub/sub messaging.*

### **1. SNS Overview**

> 🔧 *Like:* a pub/sub topic (Google Pub/Sub, Kafka topic) — one publisher, many subscribers.

- **The Rule:** **Push-based** messaging. Publisher sends message to **Topic**, Topic pushes to **Subscribers**.
- **Pattern:** Publisher → SNS Topic → Multiple Subscribers (Fan-out).
- **Subscribers:** Lambda, SQS, HTTP/HTTPS, Email, SMS, Mobile Push.

**Use Case:** Send same message to multiple systems (e.g., Order placed → Email customer + Update inventory + Log to S3).

---

### **2. SNS vs. SQS**

| **Feature** | **SNS** | **SQS** |
| --- | --- | --- |
| **Model** | **Push** (Pub/Sub) | **Pull** (Queue) |
| **Consumers** | **Multiple** (Fan-out) | **Single** (One consumer group) |
| **Persistence** | Not stored (Push and forget) | Stored until consumed/deleted |
| **Use Case** | Notifications, Fan-out | Decoupling, Async processing |

---

### **3. SNS + SQS Fan-Out Pattern**

**The Rule:** SNS Topic → multiple SQS Queues (one per service). Publisher sends to the topic, SNS pushes to all subscribed queues, each service polls its own queue independently.

- **Benefits:** Decoupled, reliable (SQS persists messages), independently scalable.

**Exam Trigger:** "Send same message to multiple services" → SNS + SQS Fan-Out.

*Real-world example:* An e-commerce site publishes one "OrderPlaced" event to an SNS topic; 4 SQS queues subscribe — email service, inventory service, analytics pipeline, audit log — each scales independently and survives the others being down.

---

### **4. SNS Message Filtering**

**The Rule:** Subscribers receive only messages matching their **filter policy** (JSON). Publisher attaches message attributes; SNS delivers only matching messages so subscribers get relevant traffic only.

**Exam Trigger:** "Subscriber receives only specific message types" → Message Filtering.

*Real-world example:* Slack publishes every "user-activity" event to one SNS topic — the mobile-push subscriber filters for `event_type = login_failed`, while the marketing subscriber filters for `event_type = purchase`, so each only gets what it cares about.

---

### **5. SNS FIFO Topics**

**The Rule:** Like SQS FIFO — strict ordering + deduplication. Can **only** subscribe **SQS FIFO Queues** (not Standard). Throughput 300 msg/sec (3,000 with batching).

**Exam Trigger:** "Pub/Sub with strict ordering" → SNS FIFO Topic + SQS FIFO Queues.

*Real-world example:* A stock-trading platform fans out trade events to risk, ledger, and notification services using an SNS FIFO topic → SQS FIFO queues — every downstream system must see "buy 100" before "sell 50", in that exact order.

---

### **SECTION 3: KINESIS (REAL-TIME STREAMING)**

*Managed streaming data service.*

### **1. Kinesis Services Overview**

AWS has **4 Kinesis services**:

- **Kinesis Data Streams:** Real-time data ingestion (Like Kafka).
- **Amazon Data Firehose** (formerly Kinesis Data Firehose): Load streaming data to S3/Redshift/OpenSearch (No code).
- **Amazon Managed Service for Apache Flink** (formerly Kinesis Data Analytics): Process streaming data with Apache Flink.
- **Kinesis Video Streams:** Stream video from devices.

**Exam Focus:** Data Streams and Firehose.

---

### **2. Kinesis Data Streams (The Real-Time)**

**The Rule:** Real-time data ingestion — producers write records, consumers read them.

- **Core Concepts:** Shard = capacity unit (1 MB/sec write, 2 MB/sec read); Record = data (max 1 MB) + Partition Key (hash-based shard routing); retention 24 hours (default) – 365 days.
- **Scaling:** Add/remove shards via shard splitting (increase) or merging (decrease).
- **Consumers:** *Standard (pull)* — 5 reads/sec (2 MB/sec) per shard, shared across consumers. *Enhanced Fan-Out (push, HTTP/2)* — dedicated 2 MB/sec per shard per consumer.
- **Use Cases:** Real-time clickstream analytics, log aggregation, IoT telemetry.

**Exam Trigger:** "Real-time data processing, Sub-second latency" → Kinesis Data Streams.

*Real-world example:* A mobile game ingests position/score/action telemetry from 100K concurrent players via Kinesis Data Streams to update live leaderboards in under 1 sec.

### **Kinesis Client Library (KCL)**

- Library that runs on **consumers** to process stream records. **Max one KCL worker per shard** — 6 shards → max 6 parallel workers; extra workers sit idle.
- To scale consumer throughput → **add shards** (splitting) + add workers. KCL checkpoints processed records in a DynamoDB table.

**Exam Trigger:** "Scale Kinesis consumers" → Add shards + KCL workers. "More consumers than shards" → Idle workers, add shards first.

*Real-world example:* Netflix's clickstream pipeline runs 10 KCL workers but only 6 shards — 4 workers sit idle. Splitting shards to 12 lets all 10 workers process in parallel.

---

### **3. Amazon Data Firehose (The Loader)**

*Formerly Kinesis Data Firehose (renamed Feb 2024).*

**The Rule:** Load streaming data to data stores (S3, Redshift, OpenSearch, Splunk, HTTP endpoints). **Near real-time** (60-sec minimum latency). Buffers until buffer time (60 sec min) OR buffer size (1–128 MB) is reached, then writes to the destination.

- **Fully managed** (no shards, auto-scales). Can invoke Lambda for transformation, compress (GZIP/Snappy/ZIP), and back up raw data to S3.

**Data Firehose vs. Data Streams:**

| **Feature** | **Data Streams** | **Data Firehose** |
| --- | --- | --- |
| **Latency** | Real-time (< 1 sec) | Near real-time (~60 sec) |
| **Capacity** | Manual (Provision shards) | Auto-scales |
| **Data Retention** | 24 hours - 365 days | None (Immediate delivery) |
| **Consumers** | Custom (Write code) | Pre-built (S3, Redshift, etc.) |
| **Use Case** | Real-time processing (Custom logic) | ETL to data lake (No code) |

**Exam Trigger:** "Load streaming data to S3 without code" → Amazon Data Firehose.

*Real-world example:* A fleet of IoT temperature sensors streams JSON into Data Firehose, which calls a Lambda to convert C → F, GZIP-compresses the batch, and lands it in S3 — the analytics team queries it with Athena. Zero servers, zero shards.

---

### **4. Kinesis vs. SQS**

| **Feature** | **Kinesis Data Streams** | **SQS** |
| --- | --- | --- |
| **Model** | Streaming (Real-time) | Queueing (Async processing) |
| **Ordering** | Per shard (Partition Key) | FIFO Queue (Strict) or Standard (Best-effort) |
| **Consumers** | Multiple (Parallel reads) | Single (One consumer deletes) |
| **Retention** | 24 hours - 365 days | 1 min - 14 days |
| **Throughput** | Limited by shards | Unlimited |
| **Use Case** | Real-time analytics, Replay data | Decoupling, Work queues |

**Exam Trigger:** "Replay data from last 7 days" → Kinesis Data Streams (SQS max 14 days, but SQS deletes after consumption).

*Real-world example:* The same gaming company replays the last 7 days of player telemetry from Kinesis to retrain its ML matchmaking model — SQS couldn't do this because it deletes messages on consumption.

---

### **SECTION 4: AMAZON MQ**

*Managed message broker (RabbitMQ, ActiveMQ).*

### **1. Amazon MQ Overview**

> 🔧 *Like:* managed RabbitMQ / ActiveMQ — it *is* those, for lift-and-shift of existing apps.

- **The Rule:** Managed **RabbitMQ** and **ActiveMQ** — for lift-and-shift of on-prem message brokers (compatibility). Protocols: AMQP, MQTT, OpenWire, STOMP, WebSocket.
- **Use** when migrating on-prem RabbitMQ/ActiveMQ apps or needing traditional broker features. **Don't use** for new cloud-native apps → SQS/SNS (simpler, cheaper, more scalable).

**Exam Trigger:** "Migrate RabbitMQ from on-prem to AWS" → Amazon MQ. "New application" → SQS/SNS.

*Real-world example:* A bank's 10-year-old Java app speaks AMQP + STOMP to a RabbitMQ broker and can't be rewritten — they lift it onto Amazon MQ unchanged. A greenfield serverless app at the same bank uses SQS/SNS instead.

---

### **SECTION 5: AMAZON EVENTBRIDGE**

*The "Central Event Router" for AWS — heavily tested on SAA-C03.*

### **1. EventBridge Overview**

- **The Rule:** Serverless event bus that routes events from sources to targets based on rules. The modern replacement for CloudWatch Events.
- **Event Buses:**
    - **Default Bus:** Receives events from AWS services automatically (EC2 state change, S3 events, CloudTrail API calls).
    - **Custom Bus:** Your application sends custom events (e.g., "OrderPlaced", "PaymentFailed").
    - **Partner Bus:** SaaS partners send events (Datadog, Zendesk, Auth0).

### **2. Rules and Targets**

- **Rules:** Match incoming events using event patterns (JSON) and route them to targets.
- **Targets:** Lambda, SQS, SNS, Step Functions, Kinesis, API Gateway, ECS task, and 18+ more AWS services.
- **Multiple Targets:** A single rule can route to **up to 5 targets**.
- **Event Transformation:** Can transform the event before sending to target (extract fields, add constants).

### **3. Key Features**

- **Scheduling:** Cron/rate expressions to trigger events on a schedule (replaces CloudWatch Events scheduled rules).
- **Archive & Replay:** Archive events to replay later (testing, debugging, reprocessing).
- **Cross-Account:** Send events between AWS accounts. **Schema Registry:** Discover/store event schemas, auto-generate code bindings.

### **4. EventBridge vs SNS**

| **Feature** | **EventBridge** | **SNS** |
| --- | --- | --- |
| **Filtering** | Advanced JSON pattern matching | Simple attribute-based filtering |
| **Targets** | 18+ AWS services | SQS, Lambda, HTTP, Email, SMS |
| **Scheduling** | Built-in cron/rate expressions | Not supported |
| **Archive/Replay** | Yes | No |
| **Use Case** | Event-driven architectures, AWS service events | Simple pub/sub fan-out |

- *Exam Trigger:* "React to AWS service events" → EventBridge. "Schedule tasks / cron jobs" → EventBridge Scheduler. "Event-driven architecture with filtering" → EventBridge Rules. "Cross-account event routing" → EventBridge.

*Real-world examples:*
- **React to AWS events:** Airbnb's SecOps team uses an EventBridge rule on the default bus — whenever any EC2 instance in prod enters `stopped` state, a Lambda pages the on-call.
- **Schedule cron:** A finance team uses EventBridge Scheduler to fire a Lambda every weekday at 6 AM that generates the daily sales report.
- **Filtering + many targets:** A SaaS app's custom "OrderPlaced" event fans out via EventBridge Rules to Step Functions (fulfillment), Lambda (loyalty points), and an SQS queue in the finance account — all from one rule.
- **Cross-account:** A central audit account ingests "IAM PolicyChanged" events from 50 sub-accounts via cross-account EventBridge buses.

---

### **Exam Summary Cheat Sheet (Practice — Fill In Yourself)**


1. **Decouple producers/consumers?** → SQS Queue
2. **Strict ordering, No duplicates?** → FIFO Queue
3. **Highest throughput, Duplicates OK?** → SQS + upgrade or whatever
4. **Reduce SQS API calls?** → Long polls
5. **Message processed multiple times?** → Visibility timeout
6. **Handle repeatedly failing messages?** → DLQ
7. **Send same message to multiple services?** → SNS+SQS FANOUT
8. **Subscriber receives only specific message types?** → SNS filtering
9. **Real-time streaming, Replay data?** → Kinesis Data Streams
10. **Load streaming data to S3 without code?** → Firehose
11. **Migrate RabbitMQ to AWS?** → Managed AWS MQ
12. **New cloud-native app?** → use SNS/SQS
13. **SQS message larger than 256 KB?** → 
14. **Scale Kinesis consumers?** → Scale shards with shard splitting
15. **React to AWS service events (EC2 state, S3, API calls)?** → Eventbridge
16. **Schedule tasks / cron in AWS?** → Eventbridge
17. **Event-driven with advanced filtering + many targets?** → Eventbridge
18. **Replay past events for debugging?** → Firehose?


---

### **Exam Summary Cheat Sheet (Memorize This)**

1. **Decouple producers/consumers?** → SQS.
2. **Strict ordering, No duplicates?** → SQS FIFO.
3. **Highest throughput, Duplicates OK?** → SQS Standard.
4. **Reduce SQS API calls?** → Enable Long Polling.
5. **Message processed multiple times?** → Increase Visibility Timeout.
6. **Handle repeatedly failing messages?** → Dead Letter Queue.
7. **Send same message to multiple services?** → SNS + SQS Fan-Out.
8. **Subscriber receives only specific message types?** → SNS Message Filtering.
9. **Real-time streaming, Replay data?** → Kinesis Data Streams.
10. **Load streaming data to S3 without code?** → Amazon Data Firehose.
11. **Migrate RabbitMQ to AWS?** → Amazon MQ.
12. **New cloud-native app?** → SQS/SNS (NOT Amazon MQ).
13. **SQS message larger than 256 KB?** → SQS Extended Client Library (stores payload in S3).
14. **Scale Kinesis consumers?** → Add shards + KCL workers (max 1 worker per shard). Or use Enhanced Fan-Out for dedicated 2 MB/sec per consumer.
15. **React to AWS service events (EC2 state, S3, API calls)?** → EventBridge.
16. **Schedule tasks / cron in AWS?** → EventBridge Scheduler.
17. **Event-driven with advanced filtering + many targets?** → EventBridge Rules.
18. **Replay past events for debugging?** → EventBridge Archive & Replay.

---

# **REAL EXAM SCENARIOS**

### Scenario 1

**The Situation:** An e-commerce platform processes orders. Each order has multiple steps: Validate Payment → Update Inventory → Ship Order. Steps must execute **in exact order** for each order. The system currently uses SQS Standard Queue, but orders are occasionally processed out of order, causing inventory errors.

**The Options:**

A. Continue using SQS Standard and add sequence numbers in application logic.

B. Use Kinesis Data Streams.

C. Switch to SQS FIFO Queue with Message Group ID = Order ID.

D. Use SNS Topic with Lambda.

**The Logic:**

- **Trap A — SQS Standard + app-side sequencing:** SQS Standard is **best-effort ordering** by design — no fix on the queue side. Re-sorting in application code means buffering, sequence tracking, and handling duplicates: complex and error-prone. The right move is the right queue type.
- **Trap B — Kinesis Data Streams:** Kinesis *does* preserve order within a shard, but it's a high-throughput **streaming** service for analytics/telemetry, not a task queue for a 3-step order workflow. Wrong tool category.
- **Trap D — SNS Topic with Lambda:** SNS is fan-out pub/sub with **no ordering guarantee** (standard topics). It would make the out-of-order problem just as bad.
- **The Fix — Option C:** **SQS FIFO** guarantees strict in-order delivery. Set **Message Group ID = Order ID** so all messages for one order process sequentially, while *different* orders (different group IDs) still process in parallel — ordering without losing throughput.

---

### Scenario 2

**The Situation:** When a user uploads a photo, the system must: (1) Send confirmation email, (2) Generate thumbnail (Lambda), (3) Store metadata in DynamoDB, (4) Audit log in S3. Currently, the upload service calls each system directly (Tight coupling). If one system is down, upload fails.

**The Options:**

A. Use SNS Topic with 4 SQS Queues (One per service) in a Fan-Out pattern.

B. Use Lambda to call all systems sequentially.

C. Use SQS Queue and have consumers poll for different message types.

D. Use SNS Topic with 4 subscribers: SES (Email), Lambda (Thumbnail), Lambda (DynamoDB), Lambda (S3).

**The Logic:**

- **Trap B — Lambda calls all systems sequentially:** This is the *current* tight-coupled design with extra steps. If one downstream is down, the chain still fails. No decoupling at all.
- **Trap C — Single SQS, consumers filter by message type:** One queue forces every consumer to read and discard messages meant for others, plus custom routing logic. Messy and inefficient — fan-out is the clean pattern.
- **Trap D — SNS direct to 4 subscribers (SES + 3 Lambdas):** Works and is decoupled, but SNS delivery is **fire-and-forget** — if a Lambda is throttled or erroring, that message can be lost (no durable buffer). Less resilient.
- **The Fix — Option A:** **SNS + SQS Fan-Out** — upload service publishes once to an SNS topic, which pushes to **4 SQS queues** (one per service). Each service polls its own queue, scales independently, and SQS **persists messages** so a down service just catches up later. Fully decoupled and resilient.

---

### Scenario 3

**The Situation:** A Lambda function polls an SQS queue every second using short polling. CloudWatch shows **86,400 API calls per day** (1 call/sec * 86,400 sec), but only **500 messages** are processed per day. Most API calls return empty (No messages).

**The Options:**

A. Increase polling frequency to every 5 seconds.

B. Switch to SNS with Lambda subscription.

C. Use Kinesis Data Streams.

D. Enable SQS Long Polling with WaitTimeSeconds = 20.

**The Logic:**

- **Trap A — Poll every 5 seconds:** Cuts API calls 5× but still mostly returns empty, *and* it adds up to 5 seconds of latency before a message is picked up. Trades one problem for another — and Long Polling beats it on both axes.
- **Trap B — Switch to SNS + Lambda:** SNS is push-based pub/sub — re-architecting the queue into a topic is a much bigger change, and SNS lacks the durable buffering/retry of SQS. Overkill for what is just an inefficient-polling problem.
- **Trap C — Kinesis Data Streams:** A streaming service for high-volume real-time data — 500 messages/day is the opposite of that. Wrong tool, and a needless rewrite.
- **The Fix — Option D:** Enable **SQS Long Polling** (`WaitTimeSeconds = 20`). Each poll waits up to 20s for a message instead of returning empty immediately — ~95% fewer API calls, *lower* cost, and *lower* latency (a message is returned the instant it arrives, not on the next tick).

---

### Scenario 4

**The Situation:** A gaming company collects player telemetry (Position, Score, Actions) from 100,000 concurrent players. Data must be processed **in real-time** (< 1 second latency) to update live leaderboards. The analytics team also needs to **replay data** from the last 7 days for model training.

**The Options:**

A. Use Kinesis Firehose to write to S3, then query with Athena.

B. Use Kinesis Data Streams with 7-day retention and Lambda consumers.

C. Use SQS Standard Queue with Lambda consumers.

D. Write directly to DynamoDB.

**The Logic:**

- **Trap A — Firehose → S3 → Athena:** Firehose buffers before delivery — minimum ~60-second latency. That breaks the "< 1 second" real-time leaderboard requirement. (It's the right answer for *batch* ETL, not live dashboards.)
- **Trap C — SQS Standard + Lambda:** SQS **deletes a message once it's consumed** — there's no way to replay the last 7 days for model training. It also isn't built for ordered, high-rate stream processing. Fails the replay requirement.
- **Trap D — Write directly to DynamoDB:** DynamoDB is a database, not a stream processor — no native real-time fan-out to consumers and no time-window replay. Wrong category.
- **The Fix — Option B:** **Kinesis Data Streams** handles real-time ingest (< 1s) for the live leaderboard via Lambda consumers, **and** its configurable **retention (set to 7 days)** lets the analytics team **replay** any point in that window for training. One service satisfies both needs.

---

### Scenario 5

**The Situation:** IoT devices send telemetry data (JSON) to AWS. The data must be **compressed**, **transformed** (Convert Celsius to Fahrenheit), and loaded into **S3** for analytics with Athena. The company wants a **fully managed solution** with no servers or custom consumers.

**The Options:**

A. Use Amazon Data Firehose with Lambda transformation and S3 destination.

B. Use Kinesis Data Streams with Lambda consumer to transform and write to S3.

C. Use SQS with Lambda to process and write to S3.

D. Use SNS to trigger Lambda to write to S3.

**The Logic:**

- **Trap B — Kinesis Data Streams + Lambda consumer:** Works, but you manage **shards** (capacity planning) and write/operate the consumer code. The requirement explicitly says *fully managed, no servers, no custom consumers* — this fails that bar.
- **Trap C — SQS + Lambda:** SQS is a message queue, not a streaming-ETL pipeline. You'd still hand-build the transform, compression, and S3-batching logic. Wrong tool and not "no custom consumers."
- **Trap D — SNS → Lambda → S3:** SNS is pub/sub notification, not a buffered delivery pipeline to S3. You'd build the batching/compression/transform yourself. Not the managed ETL solution asked for.
- **The Fix — Option A:** **Amazon Data Firehose** is fully managed — no shards, auto-scaling. It has **built-in Lambda transformation** (Celsius→Fahrenheit), **built-in compression** (GZIP), and a native **S3 destination** for Athena queries. Zero infrastructure — built for exactly this.
