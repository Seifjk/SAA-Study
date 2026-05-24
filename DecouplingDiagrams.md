# Decoupling — Visual Reference (Mermaid Diagrams)

> Visual companion to [Decoupling.md](./Decoupling.md). Render in VS Code (Markdown Preview Mermaid Support plugin) or directly on GitHub.

## Table of Contents

1. [SQS — Basic Flow + Visibility Timeout](#1-sqs--basic-flow--visibility-timeout)
2. [SNS Fan-Out vs SNS + SQS Fan-Out](#2-sns-fan-out-vs-sns--sqs-fan-out)
3. [Kinesis Data Streams — Shards + KCL Workers](#3-kinesis-data-streams--shards--kcl-workers)
4. [Kinesis Standard vs Enhanced Fan-Out](#4-kinesis-standard-vs-enhanced-fan-out)
5. [Kinesis Data Streams vs Firehose](#5-kinesis-data-streams-vs-firehose)
6. [EventBridge — Buses, Rules, Targets](#6-eventbridge--buses-rules-targets)
7. [DLQ — Poison Message Pattern](#7-dlq--poison-message-pattern)
8. [The 5 Mental Rules to Take to the Exam](#8-the-5-mental-rules-to-take-to-the-exam)

---

## 1. SQS — Basic flow + Visibility Timeout

```mermaid
graph LR
    Producer["📤 Producer<br/>(Sends message)"]
    Queue[("📥 SQS Queue<br/>📌 Retention: 1 min - 14 days<br/>📌 Default: 4 days<br/>📌 Max msg size: 256 KB")]
    Consumer["📥 Consumer<br/>(Polls for messages)"]

    Producer -->|"1. SendMessage"| Queue
    Queue -->|"2. ReceiveMessage<br/>(message becomes invisible)"| Consumer
    Consumer -->|"3a. DeleteMessage ✅<br/>(processed OK)"| Queue
    Consumer -. "3b. Timeout expires ❌<br/>(message reappears<br/>for another consumer)" .-> Queue

    style Producer fill:#d1ecf1
    style Queue fill:#fff3cd
    style Consumer fill:#d4f1c5
```

**Visibility Timeout:** when a consumer receives a message, it becomes **invisible** to other consumers for the timeout (default **30 sec**, max **12 hours**). The consumer MUST delete before the timeout, or the message returns to the queue.

**Exam trap:** *"Message processed multiple times"* → **increase Visibility Timeout** (consumer is too slow). Use `ChangeMessageVisibility` API to extend mid-processing.

**Polling:**
- **Short polling** (default): returns immediately, even empty → wasted API calls.
- **Long polling** (`WaitTimeSeconds = 1-20`): waits up to 20s → fewer empty responses, lower cost. **Exam trigger:** *"reduce SQS API calls"* → enable Long Polling.

---

## 2. SNS Fan-Out vs SNS + SQS Fan-Out

### A. Direct SNS Fan-Out (less resilient)

```mermaid
graph LR
    Publisher["📤 Publisher"]
    Topic(("📢 SNS Topic"))
    Lambda1["λ Lambda"]
    Email["📧 Email/SMS"]
    HTTP["🌐 HTTP endpoint"]

    Publisher --> Topic
    Topic -->|"push"| Lambda1
    Topic -->|"push"| Email
    Topic -->|"push"| HTTP

    style Publisher fill:#d1ecf1
    style Topic fill:#ffe0b2
```

**Problem:** SNS is **push-and-forget**. If a subscriber is throttled, down, or erroring, the message can be **lost** (only limited retries, no durable buffer).

---

### B. SNS + SQS Fan-Out (resilient — the textbook pattern)

```mermaid
graph LR
    Publisher["📤 Publisher"]
    Topic(("📢 SNS Topic"))
    Q1[("📥 SQS Queue<br/>(Service A)")]
    Q2[("📥 SQS Queue<br/>(Service B)")]
    Q3[("📥 SQS Queue<br/>(Service C)")]
    A["Service A"]
    B["Service B"]
    C["Service C"]

    Publisher --> Topic
    Topic -->|"push"| Q1
    Topic -->|"push"| Q2
    Topic -->|"push"| Q3
    Q1 -->|"poll"| A
    Q2 -->|"poll"| B
    Q3 -->|"poll"| C

    style Publisher fill:#d1ecf1
    style Topic fill:#ffe0b2
    style Q1 fill:#fff3cd
    style Q2 fill:#fff3cd
    style Q3 fill:#fff3cd
```

**Why it's better:** each SQS queue **persists** messages until consumed. If Service B goes down, its queue **buffers** the messages until it comes back.

**Exam trigger:** *"Send same message to multiple services, must be reliable / each service scales independently"* → **SNS + SQS Fan-Out**.

---

## 3. Kinesis Data Streams — shards + KCL workers

```mermaid
graph LR
    P1["📤 Producer 1<br/>(partition key A)"]
    P2["📤 Producer 2<br/>(partition key B)"]
    P3["📤 Producer 3<br/>(partition key C)"]

    subgraph Stream["Kinesis Data Stream"]
        S1[("Shard 1<br/>1 MB/s in<br/>2 MB/s out")]
        S2[("Shard 2")]
        S3[("Shard 3")]
    end

    W1["🔧 KCL Worker 1"]
    W2["🔧 KCL Worker 2"]
    W3["🔧 KCL Worker 3"]
    W4["⚠️ Worker 4<br/>(IDLE — no shard)"]

    P1 -->|"hash(key)"| S1
    P2 -->|"hash(key)"| S2
    P3 -->|"hash(key)"| S3

    S1 --> W1
    S2 --> W2
    S3 --> W3

    style Stream fill:#e1f5fe
    style W4 fill:#fcdada
```

**The 1-worker-per-shard rule:** KCL gives **one** active worker per shard. 3 shards = max 3 active workers. **Worker 4 sits idle.**

**Exam trigger:** *"Scale Kinesis consumers"* → **add shards (splitting)** AND add workers. *"More consumers than shards = idle workers"* → add shards first.

**Concepts:**
- **Shard** = capacity unit (1 MB/s in, 2 MB/s out).
- **Partition Key** = determines which shard a record lands on (consistent hash).
- **Retention** = 24h (default) to **365 days**.
- KCL checkpoints in a **DynamoDB** table.

---

## 4. Kinesis Standard vs Enhanced Fan-Out

```mermaid
graph TB
    subgraph Standard["STANDARD Consumers (pull, shared)"]
        S1[("Shard")]
        C1["Consumer A"]
        C2["Consumer B"]
        C3["Consumer C"]
        S1 -->|"📌 shared 2 MB/s total<br/>📌 5 reads/sec total<br/>📌 ~200ms latency"| C1
        S1 --> C2
        S1 --> C3
    end

    subgraph Enhanced["ENHANCED FAN-OUT (push, HTTP/2, dedicated)"]
        S2[("Shard")]
        E1["Consumer A"]
        E2["Consumer B"]
        E3["Consumer C"]
        S2 -->|"📌 2 MB/s dedicated<br/>📌 ~70ms latency"| E1
        S2 -->|"📌 2 MB/s dedicated"| E2
        S2 -->|"📌 2 MB/s dedicated"| E3
    end

    style Standard fill:#fff3cd
    style Enhanced fill:#c8e6c9
```

**Exam trigger:** *"Multiple consumers, each needs full throughput, low latency"* → **Enhanced Fan-Out**. *"Cost matters, few consumers"* → Standard.

---

## 5. Kinesis Data Streams vs Firehose

```mermaid
graph TB
    subgraph KDS["KINESIS DATA STREAMS — real-time"]
        P1["Producer"]
        Stream[("Stream<br/>📌 < 1 sec latency<br/>📌 You provision shards<br/>📌 24h - 365d retention<br/>📌 Custom consumers")]
        Consumer1["Custom Consumer<br/>(Lambda, KCL app, EFO)"]
        P1 --> Stream --> Consumer1
    end

    subgraph KDF["AMAZON DATA FIREHOSE — near-real-time"]
        P2["Producer"]
        FH[("Firehose<br/>📌 ~60 sec latency<br/>📌 Auto-scales (no shards)<br/>📌 No retention<br/>📌 Built-in transform/compress")]
        S3[("S3")]
        RS[("Redshift")]
        OS[("OpenSearch")]
        Splunk["Splunk / HTTP"]
        P2 --> FH
        FH --> S3
        FH --> RS
        FH --> OS
        FH --> Splunk
    end

    style KDS fill:#e1f5fe
    style KDF fill:#c8e6c9
```

**Buffer rule for Firehose:** flushes when buffer time (**60s min**) OR buffer size (**1-128 MB**) is reached — that's why it's NEVER sub-second.

**Exam triggers:**
- *"Real-time, custom processing, replay last 7 days"* → **Data Streams**
- *"Load streaming data to S3 without code"* → **Firehose**
- *"Transform + compress + load to S3 with zero servers"* → **Firehose + Lambda transform**

---

## 6. EventBridge — buses, rules, targets

```mermaid
graph LR
    subgraph Sources["EVENT SOURCES"]
        AWS["🔵 AWS Services<br/>(EC2, S3, CloudTrail)"]
        Custom["🟡 Custom App Events<br/>(OrderPlaced, etc.)"]
        Partner["🟣 SaaS Partners<br/>(Datadog, Zendesk)"]
        Schedule["⏰ Schedule<br/>(cron/rate)"]
    end

    subgraph Buses["EVENT BUSES"]
        Default(["Default Bus"])
        CustomBus(["Custom Bus"])
        PartnerBus(["Partner Bus"])
    end

    subgraph RulesBox["RULES (JSON pattern match)"]
        R1["Rule 1"]
        R2["Rule 2"]
    end

    subgraph Targets["TARGETS (up to 5 per rule)"]
        T1["λ Lambda"]
        T2["📥 SQS"]
        T3["📢 SNS"]
        T4["🔀 Step Functions"]
        T5["🚀 ECS Task"]
    end

    AWS --> Default
    Custom --> CustomBus
    Partner --> PartnerBus
    Schedule --> Default

    Default --> R1
    CustomBus --> R2

    R1 --> T1
    R1 --> T2
    R2 --> T3
    R2 --> T4
    R2 --> T5

    style Default fill:#e1f5fe
    style CustomBus fill:#fff9c4
    style PartnerBus fill:#f3e5f5
```

**Key facts:**
- 1 rule → **up to 5 targets**.
- **Default Bus** = AWS service events (free-tier).
- **Custom Bus** = your app's events.
- **Partner Bus** = SaaS partners (Datadog, Auth0, Zendesk).
- **Archive & Replay** = replay past events for testing/debugging.
- **Scheduler** = cron/rate (replaces CloudWatch Events scheduled rules).

**Exam triggers:**
- *"React to AWS service events (EC2 state change, S3 upload, CloudTrail API call)"* → **EventBridge**
- *"Schedule a cron job in AWS"* → **EventBridge Scheduler**
- *"Replay past events for debugging"* → **EventBridge Archive & Replay**
- *"Cross-account event routing"* → **EventBridge**

**EventBridge vs SNS:** EventBridge has **advanced JSON filtering, scheduling, archive/replay, 18+ AWS targets**. SNS is simpler pub/sub (Lambda/SQS/HTTP/email/SMS only, no scheduling, no archive).

---

## 7. DLQ — Poison Message Pattern

```mermaid
graph LR
    Producer["📤 Producer"]
    MainQ[("📥 Main SQS Queue<br/>MaxReceiveCount = 3")]
    Consumer["📥 Consumer"]
    DLQ[("☠️ Dead Letter Queue<br/>(for inspection)")]
    Engineer["👤 Engineer<br/>debugs"]

    Producer --> MainQ
    MainQ -->|"attempt 1, 2, 3<br/>(consumer fails each time)"| Consumer
    Consumer -. "fail / no delete" .-> MainQ
    MainQ -->|"After 3 failed receives,<br/>SQS moves message → DLQ"| DLQ
    DLQ --> Engineer

    style MainQ fill:#fff3cd
    style DLQ fill:#fcdada
    style Engineer fill:#d4f1c5
```

**Why it matters:** without a DLQ, a single **poison message** keeps re-appearing and **blocks the queue** — every consumer keeps timing out on it instead of processing real messages.

**Rules:**
- DLQ is just **another SQS queue** (Standard DLQ for Standard source, FIFO DLQ for FIFO source).
- `MaxReceiveCount` is set in the **redrive policy** on the main queue (typical: 3-5).
- DLQ messages have their own retention (often 14 days for max debug window).

**Exam trigger:** *"Handle messages that repeatedly fail processing / prevent poison messages"* → **Dead Letter Queue**.

---

## 8. The 5 mental rules to take to the exam

1. **SQS = pull (queue), SNS = push (pub/sub), EventBridge = router with rules + scheduling.**
2. **SNS alone = fire-and-forget. SNS + SQS = durable fan-out** (each service has its own buffer).
3. **Kinesis = streaming (real-time, replay), SQS = work queue (deleted on consume, no replay).**
4. **1 KCL worker per shard.** More workers than shards = idle. Scale = **add shards + add workers**.
5. **Firehose ≠ real-time** (60s min buffer). If question says "sub-second" → **Data Streams**. If question says "load to S3 / no code" → **Firehose**.

---

## Common trigger → service cheat strip

| Trigger phrase | Answer |
|---|---|
| Decouple producer/consumer | **SQS** |
| Strict ordering, no duplicates | **SQS FIFO** |
| Highest throughput, duplicates OK | **SQS Standard** |
| Reduce SQS API calls | **Long Polling** |
| Message processed multiple times | **Increase Visibility Timeout** |
| Repeatedly failing messages | **DLQ** |
| Send same message to multiple services | **SNS + SQS Fan-Out** |
| Subscriber filters by message type | **SNS Message Filtering** |
| Sub-second real-time + replay 7 days | **Kinesis Data Streams** |
| Load streaming data to S3, no code | **Amazon Data Firehose** |
| Migrate RabbitMQ from on-prem | **Amazon MQ** |
| Schedule cron jobs in AWS | **EventBridge Scheduler** |
| React to AWS service events | **EventBridge** |
| Replay past events for debugging | **EventBridge Archive & Replay** |
| SQS message > 256 KB | **SQS Extended Client Library** (S3 ref) |
