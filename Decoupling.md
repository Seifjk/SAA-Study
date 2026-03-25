# Decoupling (Queues & Messaging)

### **SECTION 1: SQS (SIMPLE QUEUE SERVICE)**

*Managed message queue for decoupling applications.*

### **1. SQS Overview**

- **The Rule:** Fully managed message queue. **Pull-based** (Consumers poll for messages).
- **Use Case:** Decouple producers from consumers (Async processing).
- **Scale:** Unlimited throughput, unlimited messages in queue.
- **Retention:** 1 minute - 14 days (Default: 4 days).
- **Message Size:** Max **256 KB**. For larger payloads → **SQS Extended Client Library** (stores payload in S3, sends S3 reference in SQS message).

**Pattern:** Producer → SQS Queue → Consumer (Poll)

---

### **2. Queue Types**

### **A. Standard Queue (The Default)**

**Characteristics:**

- **Throughput:** Unlimited (Nearly unlimited transactions per second).
- **Ordering:** **Best-effort ordering** (Not guaranteed).
- **Delivery:** **At-least-once delivery** (Message may be delivered multiple times).
- **Use Case:** High throughput, Order doesn't matter, Consumer handles duplicates.

**Exam Trigger:** "Highest throughput, Duplicates acceptable" → Standard Queue.

---

### **B. FIFO Queue (The Ordered)**

**Characteristics:**

- **Throughput:** **300 messages/sec** (3,000 with batching).
- **Ordering:** **Strict ordering** (Messages processed in exact order sent).
- **Delivery:** **Exactly-once processing** (Deduplication within 5-minute window).
- **Naming:** Queue name must end with `.fifo` (e.g., `orders.fifo`).
- **Message Group ID:** Group messages (Order preserved **within** group, parallel processing **across** groups).
- **Deduplication:** Based on Message Deduplication ID or Content-based hash.

**Use Case:** Order matters (e.g., Bank transactions, Trading orders).

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

### **A. Short Polling (The Immediate Return)**

- **The Rule:** Consumer polls queue. If no messages → Returns immediately (Empty response).
- **Problem:** Many empty responses → Wasted API calls → Higher cost.
- **Default:** Short polling.

### **B. Long Polling (The Wait)**

- **The Rule:** Consumer polls queue. If no messages → **Waits up to 20 seconds** for messages to arrive.
- **Benefit:** Reduces empty responses → Lower cost, Lower latency.
- **Configuration:** Set `ReceiveMessageWaitTimeSeconds` > 0 (Max 20 seconds).

**Exam Trigger:** "Reduce SQS API calls and cost" → Enable Long Polling.

---

### **4. Visibility Timeout**

**The Rule:** When consumer receives message, message becomes **invisible** to other consumers for a duration (Visibility Timeout).

**How It Works:**

1. Consumer receives message → Message hidden from queue.
2. Consumer processes message.
3. Consumer deletes message → Message removed permanently.
4. If consumer fails to delete before timeout → Message becomes **visible again** (Another consumer can process).

**Default:** 30 seconds. Max: 12 hours.

**ChangeMessageVisibility API:** Consumer can extend timeout if processing takes longer.

**Exam Trap:** "Message processed multiple times" → Visibility Timeout too short. Increase timeout or consumer processing too slow.

---

### **5. Dead Letter Queue (DLQ)**

**The Rule:** After message fails to process **N times** (MaxReceiveCount), send it to a **Dead Letter Queue** for inspection.

**How It Works:**

- Set `MaxReceiveCount` (e.g., 3 attempts).
- If message received 3 times but not deleted → Move to DLQ.
- DLQ is just another SQS queue (Standard or FIFO).

**Use Case:** Debug failed messages, Prevent poison messages from blocking queue.

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

- **Encryption:**
    - **In-Transit:** HTTPS (Enabled by default).
    - **At-Rest:** KMS encryption (Optional).
- **Access Control:**
    - **IAM Policies:** Control who can send/receive.
    - **SQS Access Policies (Resource-based):** Cross-account access, Allow S3/SNS to send messages.

**Exam Example:** "S3 bucket sends event to SQS queue in different account" → SQS Access Policy.

---

### **SECTION 2: SNS (SIMPLE NOTIFICATION SERVICE)**

*Managed pub/sub messaging.*

### **1. SNS Overview**

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

**The Rule:** SNS Topic → Multiple SQS Queues (Each service gets its own queue).

**How It Works:**

1. Publisher sends message to SNS Topic.
2. SNS pushes message to **all subscribed SQS queues**.
3. Each service polls its own SQS queue independently.

**Benefits:**

- Decoupled (Services don't know about each other).
- Reliable (SQS persists messages).
- Scalable (Each service scales independently).

**Exam Trigger:** "Send same message to multiple services" → SNS + SQS Fan-Out.

---

### **4. SNS Message Filtering**

**The Rule:** Subscribers receive only messages matching their **filter policy** (JSON).

**How It Works:**

- Publisher sends message with **attributes** (e.g., `event_type: "order_placed"`).
- Subscriber defines filter policy (e.g., Only receive `event_type: "order_placed"`).
- SNS delivers only matching messages.

**Use Case:** Reduce noise (Subscriber only gets relevant messages).

**Exam Trigger:** "Subscriber receives only specific message types" → Message Filtering.

---

### **5. SNS FIFO Topics**

**The Rule:** Like SQS FIFO. Strict ordering + Deduplication.

**Requirements:**

- SNS FIFO Topic can **only** subscribe to **SQS FIFO Queues** (Not Standard).
- Throughput: 300 msg/sec (3,000 with batching).

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

**The Rule:** Real-time data ingestion. **Producers** write records, **Consumers** read records.

**Core Concepts:**

- **Shard:** Unit of capacity (1 MB/sec write, 2 MB/sec read per shard).
- **Record:** Data (Max 1 MB) + Partition Key.
- **Partition Key:** Determines which shard (Hash-based distribution).
- **Retention:** 24 hours (Default) - 365 days (Extended).

**Scaling:**

- Add/remove shards to increase/decrease capacity.
- **Shard Splitting:** Increase capacity (1 shard → 2 shards).
- **Shard Merging:** Decrease capacity (2 shards → 1 shard).

**Consumers:**

- **Standard Consumers (Pull):** Each shard supports **5 reads/sec** (2 MB/sec total). Shared across all consumers.
- **Enhanced Fan-Out Consumers (Push):** Each consumer gets **2 MB/sec per shard** (Dedicated throughput). Uses HTTP/2.

**Use Cases:**

- Real-time analytics (Clickstream).
- Log aggregation.
- IoT telemetry.

**Exam Trigger:** "Real-time data processing, Sub-second latency" → Kinesis Data Streams.

### **Kinesis Client Library (KCL)**

- **The Rule:** Library that runs on **consumers** to process Kinesis Data Streams records.
- **Key Limit:** **One KCL worker per shard** (max). 6 shards → max 6 KCL workers processing in parallel.
- **Adding more workers than shards = idle workers** (No benefit, wasted resources).
- **Scaling Pattern:** To increase consumer throughput → **add shards** (shard splitting) + add KCL workers.
- **Checkpoint:** KCL tracks which records have been processed (Uses DynamoDB table for checkpointing).

**Exam Trigger:** "Scale Kinesis consumers" → Add shards + KCL workers. "More consumers than shards" → Idle workers, add shards first.

---

### **3. Amazon Data Firehose (The Loader)**

*Formerly Kinesis Data Firehose (renamed Feb 2024).*

**The Rule:** Load streaming data to **data stores** (S3, Redshift, OpenSearch, Splunk, HTTP endpoints). **Near real-time** (60 seconds latency minimum).

**How It Works:**

1. Producers write to Firehose.
2. Firehose buffers data (60 sec or 1 MB minimum).
3. Firehose writes to destination (S3, Redshift, etc.).

**Features:**

- **Fully Managed:** No shards to manage. Auto-scales.
- **Transformation:** Can invoke Lambda to transform data before writing.
- **Compression:** GZIP, Snappy, ZIP (Reduces storage cost).
- **Backup:** Can send raw data to S3 as backup (In addition to main destination).

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

- **The Rule:** Managed **RabbitMQ** and **ActiveMQ**.
- **Why Exist:** Lift-and-shift on-prem message brokers to AWS (Compatibility).
- **Protocols:** AMQP, MQTT, OpenWire, STOMP, WebSocket.

**When to Use:**

- Migrating on-prem apps using RabbitMQ/ActiveMQ.
- Need traditional message broker features (Topic/Queue, Request/Reply patterns).

**When NOT to Use:**

- New cloud-native apps → Use SQS/SNS (Simpler, cheaper, more scalable).

**Exam Trigger:** "Migrate RabbitMQ from on-prem to AWS" → Amazon MQ. "New application" → SQS/SNS.

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
14. **Scale Kinesis consumers?** → Add shards + KCL workers (max 1 worker per shard).

---

# **REAL EXAM SCENARIOS**

### **Scenario 1: The "Order Processing" (SQS FIFO)**

**The Situation:** An e-commerce platform processes orders. Each order has multiple steps: Validate Payment → Update Inventory → Ship Order. Steps must execute **in exact order** for each order. The system currently uses SQS Standard Queue, but orders are occasionally processed out of order, causing inventory errors.

**The Options:**

A. Continue using SQS Standard and add sequence numbers in application logic.

B. Switch to SQS FIFO Queue with Message Group ID = Order ID.

C. Use Kinesis Data Streams.

D. Use SNS Topic with Lambda.

**The Logic:**

- **Trap:** Application logic (Option A) is complex and error-prone.
- **Trap:** Kinesis (Option C) is for streaming, not task queues.
- **The Fix:** **Option B**. **SQS FIFO** guarantees strict ordering. Use **Message Group ID = Order ID** to ensure all messages for the same order are processed in order. Different orders can process in parallel (Different Message Groups).

---

### **Scenario 2: The "Fan-Out Notification" (SNS + SQS)**

**The Situation:** When a user uploads a photo, the system must: (1) Send confirmation email, (2) Generate thumbnail (Lambda), (3) Store metadata in DynamoDB, (4) Audit log in S3. Currently, the upload service calls each system directly (Tight coupling). If one system is down, upload fails.

**The Options:**

A. Use Lambda to call all systems sequentially.

B. Use SQS Queue and have consumers poll for different message types.

C. Use SNS Topic with 4 subscribers: SES (Email), Lambda (Thumbnail), Lambda (DynamoDB), Lambda (S3).

D. Use SNS Topic with 4 SQS Queues (One per service) in a Fan-Out pattern.

**The Logic:**

- **Trap:** Lambda (Option A) doesn't decouple (Still tight coupling).
- **Trap:** Single SQS (Option B) requires complex routing logic in consumers.
- **Trap:** SNS direct to Lambda (Option C) works but less resilient (No message persistence if Lambda fails).
- **The Fix:** **Option D**. **SNS + SQS Fan-Out**. Upload service sends to SNS Topic → SNS pushes to 4 SQS Queues → Each service polls its queue independently. Fully decoupled. Resilient (SQS persists messages). Each service can scale independently.

---

### **Scenario 3: The "Cost Reduction" (Long Polling)**

**The Situation:** A Lambda function polls an SQS queue every second using short polling. CloudWatch shows **86,400 API calls per day** (1 call/sec * 86,400 sec), but only **500 messages** are processed per day. Most API calls return empty (No messages).

**The Options:**

A. Increase polling frequency to every 5 seconds.

B. Enable SQS Long Polling with WaitTimeSeconds = 20.

C. Switch to SNS with Lambda subscription.

D. Use Kinesis Data Streams.

**The Logic:**

- **Trap:** Increasing interval (Option A) reduces calls but increases latency.
- **Trap:** SNS (Option C) is push-based but requires event source (Not suitable for queue pattern).
- **The Fix:** **Option B**. **Long Polling** makes Lambda wait up to 20 seconds for messages. Reduces API calls by ~95% (1 call every 20 sec vs. 1 call every sec). Lower cost, lower latency (Messages returned immediately when available).

---

### **Scenario 4: The "Real-Time Dashboard" (Kinesis Data Streams)**

**The Situation:** A gaming company collects player telemetry (Position, Score, Actions) from 100,000 concurrent players. Data must be processed **in real-time** (< 1 second latency) to update live leaderboards. The analytics team also needs to **replay data** from the last 7 days for model training.

**The Options:**

A. Use SQS Standard Queue with Lambda consumers.

B. Use Kinesis Data Streams with 7-day retention and Lambda consumers.

C. Use Kinesis Firehose to write to S3, then query with Athena.

D. Write directly to DynamoDB.

**The Logic:**

- **Trap:** SQS (Option A) deletes messages after consumption (Cannot replay). Max retention 14 days, but replay not supported.
- **Trap:** Firehose (Option C) has 60-second minimum latency (Not real-time).
- **Trap:** DynamoDB (Option D) doesn't support replay or streaming processing.
- **The Fix:** **Option B**. **Kinesis Data Streams** provides real-time processing (< 1 sec). Set retention to 7 days. Lambda consumes in real-time for leaderboards. Analytics team can replay data from any point in the 7-day window.

---

### **Scenario 5: The "ETL Pipeline" (Amazon Data Firehose)**

**The Situation:** IoT devices send telemetry data (JSON) to AWS. The data must be **compressed**, **transformed** (Convert Celsius to Fahrenheit), and loaded into **S3** for analytics with Athena. The company wants a **fully managed solution** with no servers or custom consumers.

**The Options:**

A. Use Kinesis Data Streams with Lambda consumer to transform and write to S3.

B. Use Amazon Data Firehose with Lambda transformation and S3 destination.

C. Use SQS with Lambda to process and write to S3.

D. Use SNS to trigger Lambda to write to S3.

**The Logic:**

- **Trap:** Kinesis Data Streams + Lambda (Option A) requires managing shards and consumer code.
- **Trap:** SQS (Option C) is for queueing, not streaming ETL.
- **The Fix:** **Option B**. **Amazon Data Firehose** is fully managed (No shards, auto-scales). Configure Lambda for transformation (Convert temp). Enable compression (GZIP). Set destination to S3. Zero infrastructure management. Built for this exact use case.
