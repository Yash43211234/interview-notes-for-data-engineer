# Azure Event Hubs

## 1. What Is Azure Event Hubs?

**Azure Event Hubs** is a fully managed, real-time event ingestion service provided by Microsoft Azure.

It can receive and process large volumes of events from multiple producers and make those events available to downstream consumers.

It is designed for:
- High-throughput event ingestion.
- Real-time data pipelines.
- Telemetry collection.
- IoT data ingestion.
- Application event streaming.
- Log and clickstream collection.
- Streaming analytics.

**Simple explanation:** Azure Event Hubs acts as a large-scale event ingestion layer that collects events from different sources and makes them available to applications and processing engines.

### Example

An e-commerce application generates events whenever a customer:
- Opens a product page.
- Adds an item to a cart.
- Places an order.
- Makes a payment.

Event Hubs can collect these events and make them available to downstream systems such as Azure Databricks, Stream Analytics, and other applications.

---

## 2. Why Do We Need Event Hubs?

Consider an online shopping platform that receives millions of user activities every day.

If every application component directly sends data to every downstream system, the architecture becomes tightly coupled.

Event Hubs provides an intermediary event ingestion layer.

### Without Event Hubs

```text
       Application
            |
     +------+------+ 
     |      |      |
     v      v      v
  Database  API  Analytics
```

Each application component may need separate integration logic.

### With Event Hubs

```text
       Event Producers
              |
              v
          Event Hubs
              |
       +------+------+ 
       |      |      |
       v      v      v
    Databricks  Stream   Other
                Analytics Consumers
```

Benefits:
- Producers and consumers are loosely coupled.
- Multiple consumers can process the same event stream.
- Large volumes of events can be ingested.
- Processing can happen in near real time.
- Downstream systems can scale independently.

---

## 3. Main Components of Event Hubs

The important components are:

1. Event Hub Namespace.
2. Event Hub.
3. Event Producer.
4. Event Consumer.
5. Partition.
6. Consumer Group.
7. Offset and Sequence Number.
8. Capture.

### 3.1 Event Hubs Namespace

A namespace is the top-level management and networking container for one or more Event Hubs.

It provides:
- A unique endpoint.
- Authentication and authorization configuration.
- Network settings.
- Throughput and capacity configuration.
- Management of Event Hubs within the namespace.

Example:

```text
Namespace: ecommerce-events
```

### 3.2 Event Hub

An Event Hub is the actual event stream within a namespace.

Example:

```text
Namespace: ecommerce-events

Event Hubs:
    orders
    payments
    user-activity
```

Each Event Hub receives events from producers and makes them available to consumers.

### 3.3 Event Producer

A producer is an application or service that sends events to Event Hubs.

Examples:
- Web applications.
- Mobile applications.
- IoT devices.
- Backend services.
- Log collection systems.

### 3.4 Event Consumer

A consumer reads events from Event Hubs.

Examples:
- Azure Databricks.
- Azure Stream Analytics.
- Azure Functions.
- Custom Python or Java applications.

### 3.5 Partition

A partition is an ordered sequence of events within an Event Hub.

Partitions allow an event stream to be divided into multiple independent sequences for parallel processing.

Example:

```text
              Event Hub
                  |
       +----------+----------+
       |          |          |
       v          v          v
   Partition 0 Partition 1 Partition 2
       |          |          |
       v          v          v
    Events     Events     Events
```

Each partition maintains its own sequence of events.

**Important:** Ordering is guaranteed within a partition, not across all partitions.

### 3.6 Consumer Group

A consumer group is an independent view of an Event Hub stream.

It allows multiple applications to consume the same event data independently.

Example:

```text
               Event Hub
                   |
          +--------+--------+
          |        |        |
          v        v        v
      Consumer  Consumer  Consumer
       Group A   Group B   Group C
          |        |        |
          v        v        v
      Databricks Analytics  Archive
```

Each consumer group maintains its own reading progress.

**Example:**

- Consumer Group A: Real-time order processing.
- Consumer Group B: Business analytics.
- Consumer Group C: Archival processing.

One consumer group's progress does not normally affect another consumer group's progress.

### 3.7 Offset and Sequence Number

Each event in a partition has an associated position.

**Offset:** Identifies the position of an event within a partition.

**Sequence number:** A monotonically increasing number assigned to events within a partition.

Consumers use offsets and checkpoints to track their reading progress.

### 3.8 Capture

Event Hubs Capture automatically stores incoming event data in Azure Storage or Azure Data Lake Storage.

It is useful for retaining raw event data for batch analytics and long-term processing.

---

## 4. Event Hubs Architecture

```text
              Event Producers
                     |
          +----------+----------+
          |          |          |
          v          v          v
        Web App    IoT       Backend
                  Devices    Services
          |          |          |
          +----------+----------+
                     |
                     v
              Event Hubs
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Partition 0 Partition 1 Partition 2
          |          |          |
          +----------+----------+
                     |
              Consumer Groups
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Databricks  Stream      Other
                  Analytics   Consumers
                     |
                     v
                ADLS Gen2
```

### Execution flow

1. Producers generate events.
2. Events are sent to Event Hubs.
3. Event Hubs distributes events across partitions.
4. Consumers read events from their assigned partitions.
5. Consumers process the events independently.
6. Consumer progress is tracked through offsets or checkpoints.
7. Processed data can be written to databases, data lakes, or analytical systems.

---

## 5. Partitions and Parallel Processing

Partitions are one of the most important Event Hubs concepts.

They allow multiple consumers to process an event stream in parallel.

### Example

Suppose an Event Hub contains four partitions.

```text
Event Hub
    |
    +---- Partition 0
    |
    +---- Partition 1
    |
    +---- Partition 2
    |
    +---- Partition 3
```

A consumer application can process these partitions concurrently.

### Partition key

A producer can specify a partition key when sending an event.

Events with the same partition key are routed to the same partition under the normal partition-key routing mechanism.

Example:

```json
{
  "order_id": "ORD1001",
  "customer_id": "C101",
  "amount": 2500
}
```

If `customer_id` is used as the partition key, events for the same customer can be routed to the same partition.

This can help preserve ordering for that key.

### Choosing a partition key

A good partition key should:
- Distribute event traffic reasonably evenly.
- Align with ordering requirements.
- Avoid concentrating too much traffic on one partition.
- Support the consumer's processing needs.

A poor partition key can create a hot partition.

### Important interview points

- Ordering is maintained within a partition.
- Ordering is not guaranteed across partitions.
- Parallelism is related to partition count.
- Increasing partitions does not automatically improve every workload.
- Partition-key distribution affects performance.

---

## 6. Consumer Groups

Consumer groups allow multiple independent applications to read the same event stream.

### Example

An e-commerce platform sends order events to Event Hubs.

Three applications need the same events:

1. Real-time order analytics.
2. Fraud detection.
3. Historical data ingestion.

Each application can use its own consumer group.

```text
                 Event Hub
                     |
          +----------+----------+
          |          |          |
          v          v          v
      Group A    Group B    Group C
          |          |          |
          v          v          v
      Analytics   Fraud      Data Lake
                  Detection  Ingestion
```

### Why are consumer groups useful?

- Consumers operate independently.
- Each group tracks its own progress.
- Multiple applications can read the same events.
- One consumer group's processing does not require another group to finish first.

**Important:** A consumer group is not the same as a partition.

A partition divides the event stream. A consumer group provides an independent consumption view of that stream.

---

## 7. Event Hubs Capture

Event Hubs Capture automatically writes incoming event data to Azure Blob Storage or ADLS Gen2.

It is useful when organizations want to retain raw event data for later processing.

### Architecture

```text
             Event Producers
                    |
                    v
                Event Hubs
                    |
             +------+------+
             |             |
             v             v
         Consumers      Capture
                           |
                           v
                      ADLS Gen2
                           |
                           v
                      Databricks
                           |
                           v
                       Delta Lake
```

### Benefits

- Automatically stores event data.
- Supports long-term retention in storage.
- Separates event ingestion from analytical processing.
- Enables batch processing of captured files.
- Reduces the need to build a separate archival consumer.

### Important points

- Capture writes data in supported file formats such as Avro or Parquet, depending on configuration.
- Capture uses configured time or size windows.
- Captured files are not necessarily available instantly after every event.
- Capture is useful for archival and batch processing, not a replacement for real-time consumers.

---

## 8. Event Hubs Throughput and Scaling

Event Hubs is designed for high-throughput event ingestion.

Its capacity and scaling behavior depend on the selected tier and configuration.

### 8.1 Throughput Units

In the Standard tier, throughput units provide capacity for ingress and egress.

A throughput unit represents a predefined amount of throughput capacity.

If the workload exceeds available capacity, throttling may occur.

### 8.2 Processing Units

Dedicated capacity options use processing units to provide reserved capacity.

The exact capacity model depends on the selected Event Hubs tier.

### 8.3 Partitions and Throughput

Partitions support parallel event processing.

However:
- More partitions do not automatically provide more namespace throughput.
- Namespace capacity and tier limits still matter.
- Consumer processing capacity must also be sufficient.

### 8.4 Auto-Inflate

In supported tiers, Auto-Inflate can automatically increase throughput units when utilization approaches a configured threshold.

It helps handle unexpected increases in event traffic.

It does not automatically scale every aspect of the consumer application.

---

## 9. Event Delivery and Reliability

Event Hubs is a distributed event ingestion service. Consumers must be designed to handle retries and failures.

### 9.1 At-Least-Once Processing

A consumer may process an event more than once, depending on its checkpointing and recovery behavior.

Applications should therefore handle duplicate processing safely.

### 9.2 Checkpointing

Checkpointing records the consumer's progress through a partition.

If a consumer restarts, it can resume from its last saved checkpoint rather than starting from the beginning.

Checkpointing is commonly managed through a supported checkpoint store, such as Azure Blob Storage, when using the Event Processor client.

### 9.3 Idempotency

Idempotency means that repeated processing produces the same final business effect under defined conditions.

Example:

If an order event is received twice, the processing system should avoid creating duplicate order records.

Possible approaches:
- Use a unique event ID.
- Maintain a processed-event table.
- Use idempotent upserts.
- Enforce unique business keys.
- Track processing status.

### 9.4 Event Retention

Event Hubs retains events for a configured retention period.

Consumers can read events within the available retention window.

Event Hubs is not intended to be a permanent archive.

Use Event Hubs Capture or another archival mechanism when long-term retention is required.

---

## 10. Event Hubs vs Azure Service Bus vs Event Grid

These three services support messaging and event-driven architectures, but they solve different problems.

### 10.1 Azure Event Hubs

Designed for:
- High-throughput event ingestion.
- Telemetry.
- Streaming data.
- Large-scale event streams.
- Multiple independent consumers.

### 10.2 Azure Service Bus

Designed for enterprise messaging.

Common use cases:
- Business messages.
- Reliable command processing.
- Message queues.
- Topics and subscriptions.
- Dead-letter handling.
- Message sessions and ordering scenarios.

### 10.3 Azure Event Grid

Designed for event routing and event-driven integration.

Common use cases:
- Reacting to resource changes.
- Routing notifications.
- Triggering serverless workflows.
- Distributing discrete event notifications.

### Comparison

| Feature | Event Hubs | Service Bus | Event Grid |
|---|---|---|---|
| Main purpose | Event streaming and ingestion | Enterprise messaging | Event routing |
| Data pattern | High-volume event stream | Business messages | Discrete event notifications |
| Ordering | Within a partition | Supported through sessions and other mechanisms | Not a general ordered stream |
| Retention | Configured event retention | Message lifecycle and expiry | Event delivery and retry policies |
| Main use | Telemetry and streaming | Reliable business workflows | Reacting to events |

### Example

- Millions of IoT sensor readings: Event Hubs.
- Process a payment command reliably: Service Bus.
- Trigger a function when a storage blob is created: Event Grid.

**Interview point:** Event Hubs is optimized for event streams, Service Bus for enterprise messaging, and Event Grid for event routing.

---

## 11. Azure Event Hubs in Data Engineering

Event Hubs is commonly used as the ingestion layer in streaming data pipelines.

### Example architecture

```text
            IoT Devices / Applications
                       |
                       v
                  Event Hubs
                       |
                       v
              Azure Databricks
                       |
                 Spark Streaming
                       |
             +---------+---------+
             |                   |
             v                   v
          ADLS Gen2          Delta Lake
             |                   |
             +---------+---------+
                       |
                       v
                    Analytics
```

### Example scenario: Real-time order processing

An e-commerce platform generates order events whenever a customer places an order.

Example event:

```json
{
  "event_id": "EVT1001",
  "order_id": "ORD1001",
  "customer_id": "C101",
  "amount": 2500,
  "event_type": "ORDER_PLACED",
  "event_time": "2026-10-25T10:30:00Z"
}
```

### Processing flow

1. The application generates an order event.
2. The producer sends the event to Event Hubs.
3. Event Hubs stores the event in a partition.
4. Databricks reads the event stream.
5. Spark parses and validates the event.
6. Valid records are written to a Bronze Delta table.
7. Business transformations produce Silver and Gold data.
8. Analytics applications consume the curated data.

### Why use Event Hubs?

- It separates event producers from processing systems.
- It supports high-volume ingestion.
- It allows multiple consumer applications.
- It supports parallel consumption through partitions.
- It integrates with Azure streaming services.

---

## 12. Event Hubs with Azure Databricks

Azure Databricks can consume events from Event Hubs using the supported Spark connector or Kafka-compatible endpoint.

The exact connector and configuration depend on the Databricks Runtime and integration approach.

### General architecture

```text
             Event Hubs
                  |
                  v
          Spark Structured
              Streaming
                  |
                  v
            Parse Events
                  |
                  v
          Validate and Clean
                  |
                  v
           Bronze Delta Table
                  |
                  v
          Silver / Gold Tables
```

### Example: Read events using the Kafka-compatible endpoint

Event Hubs provides a Kafka-compatible endpoint for supported Kafka clients.

The following is an illustrative PySpark pattern:

```python
from pyspark.sql.functions import col

event_hubs_stream = (
    spark.readStream
    .format("kafka")
    .option(
        "kafka.bootstrap.servers",
        "<namespace>.servicebus.windows.net:9093"
    )
    .option("subscribe", "<event-hub-name>")
    .option("kafka.security.protocol", "SASL_SSL")
    .option("kafka.sasl.mechanism", "PLAIN")
    .option(
        "kafka.sasl.jaas.config",
        "org.apache.kafka.common.security.plain.PlainLoginModule required "
        "username=\"$ConnectionString\" "
        "password=\"<event-hubs-connection-string>\";"
    )
    .option("startingOffsets", "latest")
    .load()
)
```

The Kafka source exposes event data in a binary `value` column.

For JSON messages, parse the value before applying business transformations.

```python
from pyspark.sql.functions import col, from_json
from pyspark.sql.types import (
    StructType,
    StructField,
    StringType,
    DoubleType
)

schema = StructType([
    StructField("event_id", StringType()),
    StructField("order_id", StringType()),
    StructField("customer_id", StringType()),
    StructField("amount", DoubleType()),
    StructField("event_type", StringType())
])

parsed_events = (
    event_hubs_stream
    .select(
        from_json(
            col("value").cast("string"),
            schema
        ).alias("event")
    )
    .select("event.*")
)
```

Write the stream to Delta Lake:

```python
query = (
    parsed_events.writeStream
    .format("delta")
    .option(
        "checkpointLocation",
        "/path/to/checkpoints/orders"
    )
    .outputMode("append")
    .start("/path/to/delta/bronze_orders")
)
```

### Important implementation points

- Install or use a compatible Kafka connector in the Databricks environment.
- Configure the Event Hubs endpoint and authentication securely.
- Do not hardcode connection strings in production notebooks.
- Use Databricks secrets or an appropriate identity-based authentication approach.
- Use a unique checkpoint location for each independent streaming query.
- Choose `startingOffsets` based on the required ingestion behavior.
- Design downstream processing to handle duplicates and recovery.

**Important:** The Kafka-compatible endpoint is an integration option. It does not mean Event Hubs is a complete replacement for every Kafka capability.

---

## 13. Event Hubs vs Apache Kafka

Event Hubs provides a Kafka-compatible endpoint, allowing supported Kafka clients and applications to connect without requiring a separate Kafka cluster.

However, the services are not identical.

| Feature | Azure Event Hubs | Apache Kafka |
|---|---|---|
| Service model | Managed Azure service | Open-source distributed event streaming platform |
| Infrastructure management | Azure manages service infrastructure | Depends on deployment and managed offering |
| Kafka protocol | Kafka-compatible endpoint | Native Kafka protocol |
| Scaling | Azure-managed capacity and partitions | Depends on cluster and deployment |
| Integration | Azure ecosystem | Broad ecosystem |
| Operations | Managed service | Self-managed or managed service |

### When to choose Event Hubs

- The organization uses Azure services.
- Managed event ingestion is preferred.
- Kafka-compatible integration is sufficient.
- The workload needs high-throughput streaming.

### When to choose Kafka

- The application depends on Kafka-specific capabilities.
- The organization needs more control over deployment.
- Existing Kafka infrastructure and operational expertise are available.
- Portability or a broader Kafka ecosystem is a priority.

---

## 14. Common Errors and Troubleshooting

| Problem | Possible reason | What to check |
|---|---|---|
| Authentication failed | Invalid credentials or permissions | Connection string, Entra identity, RBAC |
| Consumer cannot connect | Network or endpoint issue | Firewall, DNS, port 9093 for Kafka endpoint |
| Events are not received | Wrong Event Hub or consumer group | Configuration and subscription |
| Consumer is slow | Insufficient processing capacity | Consumer lag and processing time |
| Throttling occurs | Available throughput exceeded | Namespace capacity and tier |
| Duplicate events | Retry or checkpoint behavior | Idempotency and checkpointing |
| Events appear out of order | Events across different partitions | Partition key and ordering requirements |
| Capture files are delayed | Capture time or size window | Capture configuration |
| Hot partition | Uneven partition-key distribution | Producer partition-key design |

### Troubleshooting sequence

1. Verify the namespace and Event Hub names.
2. Check producer authentication.
3. Check consumer group configuration.
4. Verify partition assignment.
5. Inspect consumer lag and throughput.
6. Check checkpoint storage and progress.
7. Review throttling and quota metrics.
8. Verify network connectivity.
9. Inspect application logs and retry behavior.

---

## 15. Common Interview Questions and Answers

### Q1. What is Azure Event Hubs?

Azure Event Hubs is a fully managed event ingestion service designed to receive and distribute high volumes of real-time events.

### Q2. What is an Event Hubs namespace?

A namespace is the top-level resource that contains one or more Event Hubs and provides their endpoint and configuration.

### Q3. What is an Event Hub?

An Event Hub is an event stream within a namespace that receives events from producers and makes them available to consumers.

### Q4. What is a partition?

A partition is an ordered sequence of events within an Event Hub. Partitions support parallel consumption.

### Q5. Is ordering guaranteed across partitions?

No. Ordering is guaranteed within a partition, not across all partitions.

### Q6. What is a consumer group?

A consumer group provides an independent view of an event stream so multiple applications can consume the same events independently.

### Q7. What is a partition key?

A partition key helps route related events to the same partition, supporting ordering for those events and influencing data distribution.

### Q8. What is Event Hubs Capture?

Capture automatically stores incoming events in Azure Blob Storage or ADLS Gen2 for archival and batch processing.

### Q9. What is an offset?

An offset identifies an event's position within a partition and helps consumers track reading progress.

### Q10. What is checkpointing?

Checkpointing saves consumer progress so processing can resume from a known position after a restart.

### Q11. What is the difference between Event Hubs and Service Bus?

Event Hubs is designed for high-throughput event streaming, while Service Bus is designed for reliable enterprise messaging and business workflows.

### Q12. What is the difference between Event Hubs and Event Grid?

Event Hubs handles high-volume event streams, while Event Grid routes discrete events to subscribers and services.

### Q13. Can Databricks read data from Event Hubs?

Yes. Databricks can consume events through supported Spark connectors or the Kafka-compatible endpoint and process them using Structured Streaming.

### Q14. What is consumer lag?

Consumer lag represents how far a consumer is behind the latest available events in the stream.

### Q15. How do you prevent duplicate processing?

Use unique event identifiers, idempotent writes, deduplication, and appropriate checkpointing.

### Q16. Is Event Hubs a permanent storage solution?

No. It retains events for a configured period. Use Capture or another storage service for long-term archival.

---

## 16. Common Mistakes to Avoid

1. Confusing Event Hubs with Azure Service Bus.
2. Confusing Event Hubs with Event Grid.
3. Thinking a namespace and an Event Hub are the same resource.
4. Confusing partitions with consumer groups.
5. Assuming events are ordered across all partitions.
6. Assuming adding partitions automatically increases namespace throughput.
7. Ignoring partition-key distribution.
8. Treating Event Hubs as permanent data storage.
9. Assuming checkpointing alone guarantees exactly-once business processing.
10. Ignoring duplicate-event handling.
11. Assuming all Kafka features are available through the Kafka-compatible endpoint.
12. Assuming Capture provides immediate file availability for every event.

---

## 17. Quick Revision

| Concept | Remember |
|---|---|
| Azure Event Hubs | Managed event ingestion and streaming |
| Namespace | Top-level Event Hubs resource |
| Event Hub | Individual event stream |
| Producer | Sends events |
| Consumer | Reads events |
| Partition | Ordered sequence of events |
| Partition key | Routes related events |
| Consumer group | Independent consumption view |
| Offset | Position within a partition |
| Sequence number | Ordered event identifier within a partition |
| Checkpoint | Saved consumer progress |
| Capture | Stores events in Blob Storage or ADLS |
| RU | Not an Event Hubs capacity unit |
| Throughput unit | Standard-tier capacity unit |
| Event Grid | Event routing |
| Service Bus | Enterprise messaging |
| Kafka endpoint | Kafka-compatible connectivity |

### Remember these five points

1. Event Hubs is designed for high-throughput event ingestion.
2. Partitions provide parallelism and ordering within each partition.
3. Consumer groups allow independent applications to read the same stream.
4. Capture stores incoming events in Azure Storage or ADLS Gen2.
5. Databricks can consume Event Hubs data using supported streaming integrations.

---

## 18. Self-Assessment

Try answering these questions without looking at the notes:

1. What is Azure Event Hubs?
2. Why is it useful in real-time data engineering?
3. Explain Namespace vs Event Hub.
4. What is a partition?
5. What is a partition key?
6. Why is partition-key selection important?
7. What is the difference between a partition and a consumer group?
8. Is event ordering guaranteed across partitions?
9. What are offsets and sequence numbers?
10. What is checkpointing?
11. What is Event Hubs Capture?
12. Explain Event Hubs vs Service Bus vs Event Grid.
13. How can Databricks consume events from Event Hubs?
14. What is consumer lag?
15. How would you handle duplicate events?
16. What happens when Event Hubs throughput capacity is exceeded?
17. Why is Event Hubs not a permanent storage solution?

**Interview preparation priority:** Focus on Event Hubs architecture, partitions, partition keys, consumer groups, offsets, checkpointing, Capture, throughput, and integration with Databricks Structured Streaming.