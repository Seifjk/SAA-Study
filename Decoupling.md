# Decoupling (Queues & Messaging)

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

### **2. Queue Types**

### **A. Standard Queue (The Default)**

- Unlimited throughput, **best-effort ordering**, **at-least-once delivery** (duplicates possible). Use when order doesn't matter and the consumer handles duplicates.

**Exam Trigger:** "Highest throughput, Duplicates acceptable" → Standard Queue.

---

### **B. FIFO Queue (The Ordered)**

- **300 msg/sec** (3,000 with batching), **strict ordering**, **exactly-once processing** (dedup within 5-min window). Queue name must end with `.fifo`.
- **Message Group ID:** Order preserved within a group, parallel processing across groups. Dedup via Message Deduplication ID or content-based hash.
- Use when order matters (bank transactions, trading orders).

**Exam Trigger:** "Strict ordering, No duplicates" → FIFO Queue.

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

---

### **4. Visibility Timeout**

**The Rule:** When a consumer receives a message, it becomes **invisible** to other consumers for the Visibility Timeout. The consumer must delete it before the timeout expires, or the message becomes **visible again** for another consumer.

- **Default:** 30 sec. **Max:** 12 hours. `ChangeMessageVisibility` API extends the timeout if processing runs long.

**Exam Trap:** "Message processed multiple times" → Visibility Timeout too short — increase it.

---

### **5. Dead Letter Queue (DLQ)**

**The Rule:** After a message fails to process `MaxReceiveCount` times (e.g., 3), it's moved to a **Dead Letter Queue** (just another SQS queue, Standard or FIFO) for inspection.

- Use to debug failed messages and prevent poison messages from blocking the queue.

**Exam Trigger:** "Handle messages that repeatedly fail processing" → DLQ.

---

### **6. Delay Queue & Message Timers**

### **A. Delay Queue**

- **The Rule:** All messages delayed before becoming visible (0-15 minutes).
- **Use Case:** Delay all messages (e.g., Order confirmation email after 5 minutes).

### **B. Message Timer (Per-Message Delay)**

- **The Rule:** Delay individual messages (Overrides Delay Queue setting).
- **Use Case:** Different delays per message (e.g., Retry after 1 min, 5 min, 15 min).

**Exam Trigger:** "Delay processing of specific messages" → Message Timer.

---

### **7. SQS Security**

- **Encryption:** In-transit via HTTPS (default); at-rest via KMS (optional).
- **Access Control:** IAM policies control who can send/receive; SQS Access Policies (resource-based) enable cross-account access and let S3/SNS send messages.

**Exam Example:** "S3 bucket sends event to SQS queue in different account" → SQS Access Policy.

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

---

### **4. SNS Message Filtering**

**The Rule:** Subscribers receive only messages matching their **filter policy** (JSON). Publisher attaches message attributes; SNS delivers only matching messages so subscribers get relevant traffic only.

**Exam Trigger:** "Subscriber receives only specific message types" → Message Filtering.

---

### **5. SNS FIFO Topics**

**The Rule:** Like SQS FIFO — strict ordering + deduplication. Can **only** subscribe **SQS FIFO Queues** (not Standard). Throughput 300 msg/sec (3,000 with batching).

**Exam Trigger:** "Pub/Sub with strict ordering" → SNS FIFO Topic + SQS FIFO Queues.

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

### **Kinesis Client Library (KCL)**

- Library that runs on **consumers** to process stream records. **Max one KCL worker per shard** — 6 shards → max 6 parallel workers; extra workers sit idle.
- To scale consumer throughput → **add shards** (splitting) + add workers. KCL checkpoints processed records in a DynamoDB table.

**Exam Trigger:** "Scale Kinesis consumers" → Add shards + KCL workers. "More consumers than shards" → Idle workers, add shards first.

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

---

### **SECTION 4: AMAZON MQ**

*Managed message broker (RabbitMQ, ActiveMQ).*

### **1. Amazon MQ Overview**

> 🔧 *Like:* managed RabbitMQ / ActiveMQ — it *is* those, for lift-and-shift of existing apps.

- **The Rule:** Managed **RabbitMQ** and **ActiveMQ** — for lift-and-shift of on-prem message brokers (compatibility). Protocols: AMQP, MQTT, OpenWire, STOMP, WebSocket.
- **Use** when migrating on-prem RabbitMQ/ActiveMQ apps or needing traditional broker features. **Don't use** for new cloud-native apps → SQS/SNS (simpler, cheaper, more scalable).

**Exam Trigger:** "Migrate RabbitMQ from on-prem to AWS" → Amazon MQ. "New application" → SQS/SNS.

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

---

### **Exam Summary Cheat Sheet (Practice — Fill In Yourself)**

1. **Decouple producers/consumers?** → ________
2. **Strict ordering, No duplicates?** → ________
3. **Highest throughput, Duplicates OK?** → ________
4. **Reduce SQS API calls?** → ________
5. **Message processed multiple times?** → ________
6. **Handle repeatedly failing messages?** → ________
7. **Send same message to multiple services?** → ________
8. **Subscriber receives only specific message types?** → ________
9. **Real-time streaming, Replay data?** → ________
10. **Load streaming data to S3 without code?** → ________
11. **Migrate RabbitMQ to AWS?** → ________
12. **New cloud-native app?** → ________
13. **SQS message larger than 256 KB?** → ________
14. **Scale Kinesis consumers?** → ________
15. **React to AWS service events (EC2 state, S3, API calls)?** → ________
16. **Schedule tasks / cron in AWS?** → ________
17. **Event-driven with advanced filtering + many targets?** → ________
18. **Replay past events for debugging?** → ________

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
