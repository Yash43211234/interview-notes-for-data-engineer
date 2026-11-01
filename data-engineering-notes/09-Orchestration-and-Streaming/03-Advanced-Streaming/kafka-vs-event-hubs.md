

# Apache Kafka vs Azure Event Hubs

## 1. Introduction

Apache Kafka and Azure Event Hubs are distributed event-streaming platforms used to ingest, store, and distribute large volumes of events.

Both support:
- Real-time data ingestion.
- Event-driven architectures.
- Distributed event processing.
- Multiple consumers.
- Partition-based scalability.
- Integration with stream-processing engines.

However, they differ in architecture, infrastructure management, ecosystem, and operational responsibilities.

**The key difference:**

- **Apache Kafka** is an open-source distributed event-streaming platform that can be self-managed or used through managed services.
- **Azure Event Hubs** is a fully managed Azure event-streaming service with Kafka-compatible endpoints.

---

## 2. What Is Apache Kafka?

Apache Kafka is a distributed event-streaming platform used to publish, store, and consume streams of records.

It uses brokers, topics, partitions, and consumer groups to distribute events across applications.

### Architecture

```text
Producer Applications
        |
        v
    Kafka Cluster
   ┌────┼────┐
   v    v    v
Broker Broker Broker
   |     |     |
   └─────┼─────┘
         |
      Topics
         |
         v
 Consumer Groups
         |
         v
 Data Processing
```

### Main components

| Component | Purpose |
|---|---|
| Producer | Publishes records |
| Broker | Stores and serves records |
| Topic | Logical category of records |
| Partition | Ordered sequence of records |
| Consumer | Reads records |
| Consumer Group | Coordinates consumers |
| Offset | Identifies a record's position within a partition |
| Replication | Provides data redundancy |

### Example

An e-commerce application publishes order events to a Kafka topic named `orders`.

```text
Order Application
       |
       v
 Kafka Producer
       |
       v
 Topic: orders
       |
       v
 Kafka Consumers
       |
       v
 Spark / Flink / Database
```

---

## 3. What Is Azure Event Hubs?

Azure Event Hubs is a fully managed event-streaming service provided by Microsoft Azure.

It is designed to ingest and process large volumes of events from distributed applications and devices.

### Architecture

```text
Applications / Devices
          |
          v
    Event Producers
          |
          v
    Azure Event Hubs
          |
    ┌─────┼─────┐
    v     v     v
Partition Partition Partition
    |     |     |
    └─────┼─────┘
          |
     Consumer Groups
          |
          v
  Stream Processing
          |
     ┌────┴────┐
     v         v
 Databricks  Azure Storage
```

### Main components

| Component | Purpose |
|---|---|
| Namespace | Administrative and networking boundary for Event Hubs |
| Event Hub | Named event stream |
| Partition | Ordered sequence of events |
| Producer | Sends events |
| Consumer | Reads events |
| Consumer Group | Independent logical view of an event stream |
| Offset | Identifies a position in a partition |
| Checkpoint | Stores consumer progress |

### Example

An IoT platform receives temperature readings from thousands of devices.

Event Hubs ingests the readings and makes them available to multiple consumer applications.

```text
IoT Devices
     |
     v
Event Hubs
     |
     ├── Consumer Group 1 → Databricks
     |
     ├── Consumer Group 2 → Azure Functions
     |
     └── Consumer Group 3 → Data Archival
```

---

## 4. Kafka vs Event Hubs: Architecture

Both platforms use partitions to distribute events and support parallel consumption.

However, their infrastructure management is different.

### Kafka

A Kafka deployment consists of brokers that store and serve records.

Depending on the deployment, the team may manage:
- Broker infrastructure.
- Storage.
- Replication.
- Cluster upgrades.
- Monitoring.
- Capacity planning.
- Security configuration.

Kafka can also be used through managed cloud services.

### Event Hubs

Microsoft manages the underlying infrastructure.

The user mainly configures:
- Namespace.
- Event Hub.
- Partition count.
- Consumer groups.
- Retention.
- Throughput capacity.
- Authentication and networking.

### Comparison

| Feature | Kafka | Azure Event Hubs |
|---|---|---|
| Infrastructure | Self-managed or managed | Fully managed Azure service |
| Broker management | Required for self-managed deployments | Managed by Azure |
| Storage management | Depends on deployment | Managed by Azure |
| Scaling | Deployment-dependent | Managed service scaling options |
| Maintenance | Depends on hosting model | Mostly handled by Azure |
| Deployment | On-premises, cloud, or managed service | Azure |
| Operational control | Greater control in self-managed deployments | Less infrastructure control |

**Interview point:** Kafka is not necessarily self-managed. Managed Kafka services also exist.

---

## 5. Topics vs Event Hubs

Kafka uses topics to organize records.

Event Hubs uses Event Hubs, which serve a similar role as named event streams.

| Kafka | Event Hubs |
|---|---|
| Topic | Event Hub |
| Kafka Cluster | Event Hubs Namespace |
| Partition | Partition |
| Consumer Group | Consumer Group |
| Offset | Offset |
| Producer | Event Producer |
| Consumer | Event Consumer |

### Example

Kafka:

```text
Kafka Cluster
     |
     v
Topic: orders
     |
     ├── Partition 0
     ├── Partition 1
     └── Partition 2
```

Event Hubs:

```text
Event Hubs Namespace
        |
        v
Event Hub: orders
        |
        ├── Partition 0
        ├── Partition 1
        └── Partition 2
```

The concepts are similar, but their configuration, APIs, and operational behavior are not identical.

---

## 6. Partitions and Parallel Processing

Both Kafka and Event Hubs use partitions to distribute events.

A partition is an ordered sequence of events.

### Example

Suppose an order stream has three partitions.

```text
             Orders
                |
       ┌────────┼────────┐
       v        v        v
   Partition 0 Partition 1 Partition 2
       |        |        |
       v        v        v
   Consumer  Consumer  Consumer
```

### Why partitions are important

- Enable parallel consumption.
- Distribute storage and processing.
- Support scalability.
- Preserve ordering within a partition.

### Important points

1. Ordering is guaranteed within a partition, not across all partitions.
2. More partitions can allow more parallel consumers.
3. A consumer group generally has at most one active consumer assigned to a given partition at a time.
4. A hot partition can create a processing bottleneck.
5. Partition count should be planned according to throughput and consumer parallelism requirements.

### Key-based partitioning

If events have a customer ID, the system can use that key to group related events into the same partition.

Example:

```text
Customer C01 → Partition 0
Customer C02 → Partition 1
Customer C01 → Partition 0
```

This helps preserve the order of events for a key, provided the partitioning strategy and partition count remain stable.

---

## 7. Consumer Groups

Consumer groups allow multiple applications or processing instances to consume the same event stream independently.

### Example

An e-commerce company has three applications:

- Databricks for analytics.
- Fraud detection for suspicious orders.
- Archival for long-term storage.

Each can use a separate consumer group.

```text
              Orders Stream
                    |
          ┌─────────┼─────────┐
          v         v         v
      Group 1   Group 2   Group 3
          |         |         |
          v         v         v
      Databricks  Fraud    Archival
```

### Kafka consumer groups

Kafka manages partition assignment within a consumer group and supports committed offsets.

### Event Hubs consumer groups

Event Hubs provides independent views of the event stream.

Consumer applications generally maintain their own progress using an Event Processor or checkpointing mechanism.

### Important difference

Kafka has built-in consumer-group coordination and offset management.

Event Hubs consumer applications commonly use the Event Hubs client libraries and an external checkpoint store, such as Azure Blob Storage, when using the Event Processor pattern.

---

## 8. Offset and Checkpoint Management

Both platforms allow consumers to track their reading position.

### Kafka Offset

An offset identifies a record's position within a partition.

Example:

```text
Partition 0
    |
    ├── Offset 0
    ├── Offset 1
    ├── Offset 2
    └── Offset 3
```

Kafka consumers can commit offsets to Kafka's internal `__consumer_offsets` topic.

### Event Hubs Checkpoint

Event Hubs consumers can store their processing progress as checkpoints.

A checkpoint generally records the last successfully processed position for a partition.

The Event Hubs processor pattern commonly uses Azure Blob Storage as the checkpoint store.

### Comparison

| Kafka | Event Hubs |
|---|---|
| Consumer offsets | Consumer checkpoints |
| Built-in offset storage in Kafka | Commonly uses external checkpoint storage |
| `__consumer_offsets` | Often Azure Blob Storage |
| Managed by Kafka consumer APIs | Managed by Event Hubs consumer-processing libraries or application logic |

**Important:** Event Hubs also exposes offsets and sequence numbers. A checkpoint is a saved consumer position, not a replacement name for an offset.

---

## 9. Retention and Event Replay

Both platforms retain events for a configured period, allowing consumers to read previously published data.

### Kafka

Kafka retention can be configured based on:
- Time.
- Log size.
- Topic-level policies.

Kafka also supports log compaction for topics where the latest value for each key is important.

### Event Hubs

Event Hubs retains events according to the selected tier and configuration.

Consumers can replay retained events by starting from an earlier available position.

Event Hubs Capture can also archive incoming events to Azure Blob Storage or ADLS Gen2.

### Comparison

| Feature | Kafka | Event Hubs |
|---|---|---|
| Retention | Configurable by time or size | Tier and configuration dependent |
| Replay | Supported within retention | Supported within retention |
| Log compaction | Supported | Not a direct equivalent to Kafka log compaction |
| Long-term archival | External storage or tiered storage options | Event Hubs Capture to Azure Storage |

### Important point

Retention and archival are different.

Retention controls how long the event-streaming platform keeps events available. Archival stores data separately for longer-term access.

---

## 10. Kafka Compatibility in Azure Event Hubs

One of the important features of Azure Event Hubs is its Kafka-compatible endpoint.

It allows many Kafka clients and applications to communicate with Event Hubs using the Kafka protocol.

### Architecture

```text
Kafka Producer / Consumer
          |
          v
   Kafka Protocol
          |
          v
   Azure Event Hubs
          |
          v
      Partitions
```

### Why is this useful?

A company may already have Kafka-based applications but want to use Azure-managed infrastructure.

Instead of completely rewriting the application, it can often configure the Kafka client to connect to Event Hubs.

### Typical configuration differences

| Kafka | Event Hubs Kafka endpoint |
|---|---|
| Bootstrap server | Event Hubs namespace endpoint |
| Topic | Event Hub name |
| Kafka authentication | Event Hubs-supported authentication |
| Kafka broker cluster | Azure-managed Event Hubs |
| Kafka-specific configuration | Some settings need Event Hubs-specific values |

### Example configuration

Illustrative Kafka producer properties:

```properties
bootstrap.servers=<namespace>.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="<EVENT_HUBS_CONNECTION_STRING>";
```

This is an illustrative connection pattern. Production applications should use secure secret management and the authentication options supported by the chosen client and Event Hubs configuration.

### Important limitations

Kafka compatibility does not mean Event Hubs is a complete replacement for every Kafka feature.

Some Kafka APIs, administrative operations, configurations, and ecosystem features may not be supported or may behave differently.

Always check the current Event Hubs Kafka compatibility documentation before migrating a workload.

---

## 11. Throughput and Scaling

Both systems are designed to handle large event volumes.

However, their scaling models differ.

### Kafka

Scaling depends on:
- Broker resources.
- Number of brokers.
- Partition count.
- Replication factor.
- Storage capacity.
- Network capacity.
- Producer and consumer throughput.

For self-managed Kafka, teams are responsible for infrastructure capacity and cluster operations.

### Event Hubs

Event Hubs offers managed capacity options.

Common capacity models include:

- Basic.
- Standard.
- Premium.
- Dedicated.

Availability of capabilities and limits depends on the selected tier.

Important scaling concepts:

| Concept | Description |
|---|---|
| Throughput Units | Standard-tier capacity units |
| Processing Units | Premium-tier capacity units |
| Capacity Units | Dedicated-tier capacity units |
| Auto-inflate | Automatically increases Standard-tier throughput units up to a configured maximum |

### Important point

Event Hubs scaling is not simply the same as adding Kafka brokers.

Partition count, tier limits, throughput quotas, and consumer capacity all affect performance.

---

## 12. Security and Authentication

Both platforms support authentication and authorization, but their security models differ.

### Kafka

Common security mechanisms include:
- TLS encryption.
- SASL authentication.
- ACL-based authorization.
- Integration with external identity systems.

The exact setup depends on the Kafka distribution and deployment.

### Azure Event Hubs

Common security mechanisms include:
- Microsoft Entra ID authentication.
- Shared Access Signature (SAS) authentication.
- Azure RBAC.
- Private endpoints.
- Network access controls.
- TLS encryption.

### Comparison

| Feature | Kafka | Event Hubs |
|---|---|---|
| Encryption | TLS | TLS |
| Authentication | SASL and deployment-specific integrations | Entra ID and SAS |
| Authorization | Kafka ACLs and configured authorizers | Azure RBAC and SAS policies |
| Private networking | Deployment-dependent | Azure networking and Private Link options |
| Identity integration | Depends on deployment | Native Azure identity integration |

### Important distinction

Authentication verifies identity. Authorization determines what that identity can do.

A managed identity or service principal does not automatically receive permission to send or receive events. Appropriate Event Hubs data roles must be assigned.

---

## 13. Integration with Data Engineering Tools

Both platforms integrate with distributed processing engines and data platforms.

### Kafka integrations

- Apache Spark.
- Apache Flink.
- Apache Storm.
- Kafka Connect.
- Databases.
- Custom applications.

### Event Hubs integrations

- Azure Databricks.
- Azure Stream Analytics.
- Azure Functions.
- Azure Data Explorer.
- Azure Storage.
- Apache Spark.
- Apache Flink through supported connectors and Kafka-compatible integrations.

### Example: Event Hubs to Databricks

```text
Applications / IoT Devices
          |
          v
     Event Hubs
          |
          v
    Databricks Spark
          |
          v
   Data Validation
          |
          v
   Bronze Delta Table
          |
          v
   Silver Delta Table
          |
          v
   Gold Delta Table
```

### Example: Kafka to Databricks

```text
Applications
     |
     v
Kafka Topics
     |
     v
Databricks Spark
     |
     v
Transformations
     |
     v
Delta Lake
```

Both architectures can support streaming data pipelines.

The connector configuration, authentication, offset handling, and operational requirements differ.

---

## 14. Event Hubs Capture vs Kafka Connect

These are related integration capabilities, but they are not identical.

### Event Hubs Capture

Event Hubs Capture automatically archives streaming data to Azure Blob Storage or ADLS Gen2.

It is useful for:
- Long-term storage.
- Data lake ingestion.
- Batch analytics.
- Historical replay from archived files.

### Kafka Connect

Kafka Connect is a framework for moving data between Kafka and external systems using source and sink connectors.

It supports a wider integration ecosystem through connector plugins.

### Comparison

| Event Hubs Capture | Kafka Connect |
|---|---|
| Managed Event Hubs archival feature | General integration framework |
| Captures Event Hubs data to Azure Storage | Moves data between Kafka and external systems |
| Azure-focused destinations | Broad connector ecosystem |
| Minimal infrastructure management | Deployment and connector management depend on hosting model |

**Important:** Event Hubs Capture is not a complete replacement for Kafka Connect.

---

## 15. Performance and Reliability Considerations

Both systems need careful configuration to achieve reliable, high-throughput processing.

### Kafka considerations

- Broker capacity.
- Partition distribution.
- Replication factor.
- Producer batching.
- Consumer lag.
- Network throughput.
- Disk performance.
- Retention configuration.

### Event Hubs considerations

- Throughput capacity.
- Partition count.
- Consumer lag.
- Namespace limits.
- Tier selection.
- Auto-inflate configuration.
- Capture configuration.
- Authentication and networking.

### Common performance problems

| Problem | Possible cause | Approach |
|---|---|---|
| Low throughput | Insufficient capacity | Review partitions and capacity |
| High consumer lag | Consumers cannot keep up | Scale consumers where possible |
| Hot partition | Uneven key distribution | Review partition keys |
| Slow processing | Downstream bottleneck | Optimize consumer application |
| Throttling | Throughput limits reached | Review tier and capacity |
| Data loss concerns | Incorrect acknowledgment or recovery design | Review producer, consumer, and sink guarantees |

---

## 16. Kafka vs Event Hubs: Complete Comparison

| Feature | Apache Kafka | Azure Event Hubs |
|---|---|---|
| Type | Distributed event-streaming platform | Managed event-streaming service |
| Hosting | Self-managed or managed | Azure-managed |
| Main component | Broker | Namespace and Event Hub |
| Data organization | Topics and partitions | Event Hubs and partitions |
| Consumer groups | Supported | Supported |
| Ordering | Within a partition | Within a partition |
| Offsets | Built-in Kafka offset management | Offsets exposed; checkpoints commonly managed by consumers |
| Retention | Configurable | Tier and configuration dependent |
| Replay | Supported | Supported |
| Log compaction | Supported | No direct equivalent |
| Kafka protocol | Native | Kafka-compatible endpoint |
| Infrastructure management | Deployment-dependent | Managed by Azure |
| Security | SASL, TLS, ACLs, integrations | Entra ID, SAS, RBAC, TLS |
| Scaling | Brokers, partitions, and resources | Managed capacity and partition configuration |
| Capture/archival | Connectors and storage integrations | Built-in Capture to Azure Storage |
| Best fit | Flexible Kafka ecosystem and deployment choices | Azure-native managed event ingestion |

---

## 17. When Should You Choose Kafka?

Choose Kafka when:

1. You need the broader Kafka ecosystem.
2. You need Kafka Connect and its connector ecosystem.
3. You require control over broker configuration and deployment.
4. You want to deploy across different cloud providers or on-premises.
5. You need Kafka-specific capabilities such as log compaction.
6. You already have Kafka infrastructure and expertise.

Kafka can also be used through managed services when you want to reduce operational responsibilities.

---

## 18. When Should You Choose Azure Event Hubs?

Choose Event Hubs when:

1. Your architecture is primarily on Azure.
2. You want managed event ingestion.
3. You want to reduce broker infrastructure management.
4. You need integration with Azure Databricks, Functions, or Stream Analytics.
5. You want built-in archival through Event Hubs Capture.
6. You want to use Kafka-compatible clients without operating a Kafka cluster.

### Simple decision

- Need broad Kafka ecosystem and deployment flexibility? Consider Kafka.
- Need managed Azure-native event ingestion? Consider Event Hubs.
- Already have Kafka applications and want to move to Azure? Evaluate Event Hubs Kafka compatibility before migration.

---

## 19. Practical Data Engineering Scenario

Consider a company collecting events from thousands of IoT devices.

The company wants to process sensor readings, detect abnormal temperatures, and store historical data.

### Option A: Kafka architecture

```text
IoT Devices
     |
     v
Kafka Producers
     |
     v
Kafka Cluster
     |
     ├── Flink
     |
     ├── Spark
     |
     └── Kafka Connect
             |
             v
        Data Storage
```

The company manages or selects a managed Kafka deployment and configures the surrounding infrastructure.

### Option B: Event Hubs architecture

```text
IoT Devices
     |
     v
Azure Event Hubs
     |
     ├── Azure Databricks
     |
     ├── Azure Functions
     |
     └── Event Hubs Capture
             |
             v
          ADLS Gen2
```

Azure manages the Event Hubs infrastructure, and the company focuses more on event processing and downstream data pipelines.

### Interview discussion

If the company already uses Azure Databricks, ADLS Gen2, and Azure identity services, Event Hubs may be a natural fit.

If it depends heavily on Kafka-specific ecosystem features, Kafka may be more appropriate.

The decision should be based on functional requirements, cost, latency, operational skills, and migration complexity.

---

## 20. Common Interview Questions and Answers

### Q1. What is Apache Kafka?

Apache Kafka is a distributed event-streaming platform used to publish, store, and consume streams of records.

### Q2. What is Azure Event Hubs?

Azure Event Hubs is a fully managed Azure service for ingesting and streaming large volumes of events.

### Q3. What is the main difference between Kafka and Event Hubs?

Kafka is an event-streaming platform that can be self-managed or managed. Event Hubs is a fully managed Azure event-streaming service.

### Q4. Is Event Hubs a replacement for Kafka?

Not in every situation. It supports Kafka-compatible clients, but it does not implement every Kafka feature or API.

### Q5. What is the Kafka-compatible endpoint in Event Hubs?

It allows supported Kafka clients to connect to Event Hubs using the Kafka protocol.

### Q6. What is the difference between a Kafka topic and an Event Hub?

Both represent named event streams, but they belong to different platforms and have different configuration and operational behavior.

### Q7. What is a partition?

A partition is an ordered sequence of events used for parallelism and scalability.

### Q8. What is a consumer group?

A consumer group provides an independent logical view of an event stream and allows multiple consumers to process partitions.

### Q9. What is the difference between Kafka offset and Event Hubs checkpoint?

A Kafka offset identifies a record position in a partition and can be committed to Kafka. An Event Hubs checkpoint stores consumer progress, commonly in Azure Blob Storage when using the Event Processor pattern.

### Q10. Does Event Hubs support event replay?

Yes. Consumers can read events again from an earlier available position within the configured retention period.

### Q11. What is Event Hubs Capture?

It automatically archives events from Event Hubs to Azure Blob Storage or ADLS Gen2.

### Q12. What is the difference between Event Hubs Capture and Kafka Connect?

Capture is a managed archival feature for Azure Storage. Kafka Connect is a general-purpose framework for integrating Kafka with external systems.

### Q13. How does Event Hubs scale?

It uses managed capacity options such as Throughput Units, Processing Units, and Capacity Units, depending on the tier.

### Q14. What are Throughput Units?

Throughput Units are capacity units used by the Standard tier to control available ingress and egress throughput.

### Q15. Does Event Hubs support consumer groups?

Yes. Multiple consumer groups can independently consume the same Event Hub.

### Q16. Can Databricks read from Event Hubs?

Yes. Databricks can consume Event Hubs data using supported connectors or Kafka-compatible integration, depending on the environment and configuration.

### Q17. Can Kafka and Event Hubs guarantee ordering across all partitions?

No. Ordering is guaranteed within a partition, not across the entire topic or Event Hub.

### Q18. Which is better for an Azure-based data engineering pipeline?

Event Hubs is often a natural choice for Azure-native managed ingestion. Kafka may be preferable when Kafka-specific features or a broader Kafka ecosystem are required.

### Q19. What is the difference between retention and archival?

Retention controls how long the streaming platform keeps events available. Archival stores data separately for longer-term use.

### Q20. Does using the Kafka-compatible endpoint mean every Kafka application works without changes?

No. Many Kafka clients work with configuration changes, but unsupported APIs, authentication differences, and platform-specific behavior may require application changes.

---

## 21. Common Mistakes to Avoid

1. **Confusing Event Hubs with Kafka:** They are separate platforms.
2. **Assuming Event Hubs is a complete Kafka clone:** Kafka compatibility has limitations.
3. **Confusing Namespace with Event Hub:** Namespace is the administrative boundary; Event Hub is the named stream.
4. **Confusing offsets with checkpoints:** They represent related but different concepts.
5. **Assuming ordering across partitions:** Ordering is partition-specific.
6. **Assuming Event Hubs Capture is equivalent to Kafka Connect:** Their purposes and capabilities differ.
7. **Assuming Kafka always requires self-management:** Managed Kafka services exist.
8. **Assuming Event Hubs has unlimited throughput:** Capacity and tier limits apply.
9. **Confusing retention with archival:** They serve different purposes.
10. **Assuming Azure RBAC automatically grants all access:** The appropriate Event Hubs data role must be assigned.
11. **Assuming a separate consumer group creates a separate copy of stored events:** It provides an independent consumption position, not a separate physical copy.
12. **Assuming a Kafka-compatible endpoint supports every Kafka feature:** Always verify compatibility before migration.

---

## 22. Quick Revision

| Concept | Kafka | Event Hubs |
|---|---|---|
| Platform | Event-streaming platform | Managed Azure event-streaming service |
| Main stream | Topic | Event Hub |
| Infrastructure | Self-managed or managed | Azure-managed |
| Partition | Supported | Supported |
| Consumer group | Supported | Supported |
| Position tracking | Offset | Offset and consumer checkpoint |
| Retention | Configurable | Tier/configuration dependent |
| Replay | Supported | Supported |
| Kafka protocol | Native | Compatible endpoint |
| Archival | External tools/connectors | Event Hubs Capture |
| Security | SASL, TLS, ACLs | Entra ID, SAS, RBAC, TLS |
| Scaling | Brokers and partitions | Managed capacity and partitions |

### Remember these five points

1. Kafka is a distributed event-streaming platform.
2. Event Hubs is a fully managed Azure event-streaming service.
3. Both use partitions and consumer groups.
4. Event Hubs supports Kafka-compatible clients, but not every Kafka feature.
5. Choose based on ecosystem, operational control, cloud environment, cost, and requirements.

---

## 23. Self-Assessment

Try answering these questions without looking at the notes.

1. What is Apache Kafka?
2. What is Azure Event Hubs?
3. Explain the architectural difference between them.
4. What is the difference between a Kafka topic and an Event Hub?
5. Explain partitions and consumer groups.
6. What is the difference between Kafka offsets and Event Hubs checkpoints?
7. How does Kafka compatibility work in Event Hubs?
8. Does Event Hubs support all Kafka APIs?
9. What is Event Hubs Capture?
10. Compare Kafka Connect and Event Hubs Capture.
11. Explain the scaling models of Kafka and Event Hubs.
12. Compare their security mechanisms.
13. How can Databricks consume events from Event Hubs?
14. When would you choose Kafka over Event Hubs?
15. When would you choose Event Hubs over Kafka?
16. Design an Azure-based real-time IoT pipeline using Event Hubs, Databricks, and ADLS Gen2.

**Interview readiness:** You should be able to explain the architectural differences, partitioning, consumer groups, offset management, Kafka compatibility, and when to choose each platform for a data engineering pipeline.

