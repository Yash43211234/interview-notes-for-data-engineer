# Batch Processing vs Stream Processing

## 1. What Is Data Processing?

Data processing is the process of collecting, transforming, and storing data so that it can be used for analysis, reporting, or business operations.

In data engineering, data generally arrives in two ways:

1. All together in a collection.
2. Continuously over time.

These lead to two major processing approaches:

- Batch Processing
- Stream Processing

**Simple way to remember:**

- Batch = Process data in groups.
- Streaming = Process data continuously as events arrive.

## 2. What Is Batch Processing?

Batch processing means collecting data over a period of time and processing it together as a batch.

Instead of processing every record immediately, the system waits until a scheduled time or until a particular amount of data is available.

### Example

An e-commerce company receives thousands of orders throughout the day.

Instead of processing every order individually, it collects the records and processes them at midnight.

```text
Orders throughout the day
          |
          v
     Collect Data
          |
          v
     Daily Batch
          |
          v
   Data Processing
          |
          v
    Data Warehouse
          |
          v
   Daily Sales Report
```

### Characteristics

- Processes data in groups.
- Usually runs on a schedule or when a batch is ready.
- Suitable for large-volume historical processing.
- Results are available after the batch completes.
- Often used for ETL and ELT pipelines.

### Common Examples

- Daily sales reports.
- Monthly financial reports.
- Payroll processing.
- Daily customer-data integration.
- Historical data warehouse loading.

## 3. What Is Stream Processing?

Stream processing means processing data continuously as new events arrive.

Instead of waiting for a complete batch, the system processes incoming records with low latency.

### Example

An online payment system receives transactions continuously.

When a customer makes a payment, the system processes the event and updates the transaction status.

```text
Transaction Events
        |
        v
   Event Stream
        |
        v
 Stream Processing
        |
        v
  Processed Results
        |
        v
 Dashboard / Alerts
```

### Characteristics

- Processes continuously arriving data.
- Designed for low-latency results.
- Handles events as they arrive.
- Often uses event brokers or streaming platforms.
- Requires attention to event ordering, duplicates, failures, and late-arriving events.

### Common Examples

- Fraud detection.
- Real-time payment monitoring.
- IoT sensor monitoring.
- Live website activity tracking.
- Real-time recommendation systems.

## 4. Batch vs Stream Processing

| Feature | Batch Processing | Stream Processing |
|---|---|---|
| Processing | Groups of records | Continuously arriving events |
| Data arrival | Collected before processing | Processed as events arrive |
| Latency | Usually higher | Usually lower |
| Execution | Scheduled or triggered | Continuous or micro-batched |
| Data volume | Often large accumulated datasets | Continuous event flow |
| Complexity | Often simpler to operate | Requires more operational considerations |
| Historical analysis | Very suitable | Possible, depending on design |
| Real-time alerts | Not ideal for low-latency alerts | Suitable |
| Example | Daily sales report | Fraud detection |

**Important:** Batch and streaming are not necessarily separate technologies. Some frameworks, including Apache Spark, support both approaches.

## 5. Understanding Processing Latency

Latency is the time between data becoming available and the processing result becoming available.

### Batch Latency

Suppose an organization processes sales data once every 24 hours.

```text
10 AM  -> Order received
02 PM  -> Order received
06 PM  -> Order received
12 AM  -> Batch processing starts
01 AM  -> Report generated
```

The result is not immediately available when the order arrives.

### Streaming Latency

Suppose a payment event arrives at 10:00:00 AM and the system processes it at 10:00:02 AM.

The processing latency is approximately two seconds.

### Key Point

- Batch processing accepts higher latency when scheduled results are sufficient.
- Stream processing aims for lower latency when timely results matter.

There is no universal latency threshold separating batch and streaming. Requirements depend on the business use case and architecture.

## 6. Micro-Batch Processing

Micro-batch processing divides incoming data into small batches and processes each batch at regular intervals.

It sits between traditional large batch processing and record-at-a-time processing.

### Example

Suppose events arrive continuously, but the system processes them every five seconds.

```text
Incoming Events
       |
       v
+--------------+
| Batch 1      |  0–5 seconds
+--------------+
       |
       v
+--------------+
| Batch 2      |  5–10 seconds
+--------------+
       |
       v
+--------------+
| Batch 3      | 10–15 seconds
+--------------+
       |
       v
 Process Results
```

### Advantages

- Lower latency than large scheduled batches.
- Efficient processing of groups of events.
- Supports stateful operations.
- Useful for scalable distributed processing.

### Disadvantage

Results are not necessarily produced immediately for each individual event.

**Interview point:** Micro-batch processing is a common streaming execution model in Apache Spark Structured Streaming.

## 7. Real-Time vs Near-Real-Time Processing

These terms describe how quickly results need to be available.

### Real-Time Processing

The system processes events within a strict or very short time requirement.

Example:

- Payment authorization.
- Time-sensitive fraud detection.
- Critical industrial alerts.

### Near-Real-Time Processing

The system processes events with a small delay that is acceptable for the business.

Example:

- Dashboard updates every few seconds.
- Website activity analytics.
- Periodic inventory updates.

| Type | Meaning |
|---|---|
| Real-time | Meets a defined, often strict response-time requirement |
| Near-real-time | Produces results with a small acceptable delay |
| Batch | Processes accumulated data periodically |

**Important:** Real-time does not automatically mean zero latency. The required latency must be defined by the business.

## 8. Batch Processing Architecture

A typical batch data pipeline looks like this:

```text
       SOURCE SYSTEMS
              |
              v
        Data Extraction
              |
              v
        Raw Data Storage
              |
              v
       Batch Processing
              |
              v
       Data Transformation
              |
              v
        Data Warehouse
              |
              v
        BI / Reporting
```

### Example Technologies

| Layer | Technologies |
|---|---|
| Data Sources | MySQL, PostgreSQL, APIs |
| Storage | ADLS, Amazon S3, HDFS |
| Processing | Apache Spark, PySpark |
| Orchestration | Apache Airflow, Azure Data Factory |
| Warehouse | Snowflake, BigQuery, Synapse |
| Reporting | Power BI, Tableau |

## 9. Streaming Architecture

A typical streaming pipeline looks like this:

```text
       EVENT PRODUCERS
              |
              v
        Event Broker
              |
              v
      Stream Processing
              |
              v
       Processed Output
              |
       +------+------+
       |             |
       v             v
   Data Lake      Dashboard
       |
       v
  Data Warehouse
```

### Example Technologies

| Layer | Technologies |
|---|---|
| Event Producers | Applications, IoT devices |
| Event Broker | Apache Kafka, Azure Event Hubs |
| Processing | Spark Structured Streaming, Flink |
| Storage | ADLS, Amazon S3 |
| Analytics | Power BI, Databricks SQL |

**Important:** Kafka and Event Hubs are event-ingestion or messaging platforms. Spark Structured Streaming and Flink are processing engines. They perform different roles.

## 10. Apache Spark: Batch and Streaming

Apache Spark supports both batch processing and stream processing.

### 10.1 Batch Processing with PySpark

Example:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("BatchProcessing") \
    .getOrCreate()

df = spark.read \
    .format("csv") \
    .option("header", True) \
    .option("inferSchema", True) \
    .load("/data/orders/")

result = df.groupBy("customer_id") \
    .sum("amount")

result.write \
    .mode("overwrite") \
    .parquet("/output/customer_sales/")
```

This example:

1. Reads accumulated CSV files.
2. Groups orders by customer.
3. Calculates total sales.
4. Writes the results to storage.

### 10.2 Streaming Processing with PySpark

Example:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("StreamingProcessing") \
    .getOrCreate()

stream_df = spark.readStream \
    .format("json") \
    .schema(schema) \
    .load("/data/events/")

result = stream_df.groupBy("customer_id") \
    .count()

query = result.writeStream \
    .format("console") \
    .outputMode("complete") \
    .option("checkpointLocation", "/checkpoints/customer_count") \
    .start()

query.awaitTermination()
```

This example:

1. Continuously discovers new JSON files.
2. Reads incoming events.
3. Groups events by customer.
4. Updates the aggregation.
5. Uses a checkpoint location to support recovery.

**Important:** The streaming example assumes `schema` has already been defined. The console sink is useful for learning and testing, not as a production storage destination.

## 11. Batch and Streaming in Databricks

Databricks supports batch and streaming pipelines using Apache Spark and Delta Lake.

### Batch Pipeline

```text
Source Files
     |
     v
PySpark Batch Job
     |
     v
Bronze Table
     |
     v
Silver Table
     |
     v
Gold Table
```

### Streaming Pipeline

```text
Kafka / Event Hubs
        |
        v
Structured Streaming
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

### Example Use Cases

| Layer | Batch Example | Streaming Example |
|---|---|---|
| Bronze | Daily file ingestion | Continuous event ingestion |
| Silver | Daily cleaning and validation | Continuous event transformation |
| Gold | Daily business aggregation | Frequently updated business metrics |

Both approaches can be part of the same medallion architecture.

## 12. How to Choose Between Batch and Streaming?

The choice depends on business requirements.

Ask these questions:

1. How quickly must the result be available?
2. Does the source provide accumulated files or continuous events?
3. Is real-time action required?
4. What is the acceptable processing cost?
5. How complex is the pipeline?
6. Can the business accept periodic updates?

### Choose Batch When

- Daily or hourly results are sufficient.
- Historical data needs large-scale processing.
- The business does not need immediate updates.
- Simpler scheduled pipelines are preferred.

### Choose Streaming When

- Events need to be processed continuously.
- Low latency is important.
- Immediate alerts or actions are required.
- Data arrives continuously from applications or devices.

### Choose a Hybrid Approach When

Some requirements need both historical and current data.

Example:

An e-commerce company may:

- Process historical orders through a daily batch job.
- Process new orders through a streaming pipeline.
- Combine both datasets for business reporting.

## 13. Lambda and Kappa Architectures

These are two common architectural approaches for building data-processing systems.

### 13.1 Lambda Architecture

Lambda Architecture uses separate batch and streaming paths.

```text
            SOURCE DATA
                 |
         +-------+-------+
         |               |
         v               v
    Batch Layer     Speed Layer
         |               |
         v               v
    Batch Views    Real-Time Views
         |               |
         +-------+-------+
                 |
                 v
          Serving Layer
```

### Characteristics

- Batch layer processes complete historical data.
- Speed layer processes recent incoming events.
- Serving layer combines results from both paths.

**Main challenge:** Maintaining two processing paths can increase code duplication and operational complexity.

### 13.2 Kappa Architecture

Kappa Architecture focuses on a streaming-oriented processing pipeline.

```text
          SOURCE EVENTS
                 |
                 v
            Event Log
                 |
                 v
        Stream Processing
                 |
                 v
          Serving Layer
```

Historical data can be reprocessed by replaying retained events through the streaming pipeline, when the source and architecture support it.

### Characteristics

- Uses one primary processing path.
- Can simplify logic shared between live and replayed data.
- Depends on event-log retention and replay capabilities.
- Not every batch workload is naturally suited to this approach.

### Lambda vs Kappa

| Feature | Lambda | Kappa |
|---|---|---|
| Processing paths | Separate batch and streaming paths | One primary streaming path |
| Historical processing | Batch layer | Replay retained events |
| Real-time processing | Speed layer | Streaming pipeline |
| Complexity | Often higher | Can be simpler |
| Main challenge | Maintaining two paths | Managing replay and event retention |

## 14. Common Challenges in Streaming

Streaming systems introduce additional engineering considerations.

### 14.1 Late-Arriving Data

Events may arrive later than expected.

Example:

An event occurred at 10:00 AM but reached the processing system at 10:10 AM.

The pipeline must decide how to handle it.

### 14.2 Duplicate Events

The same event may be delivered more than once.

The pipeline may need deduplication or idempotent processing.

### 14.3 Event Ordering

Events may arrive in a different order from when they occurred.

Event timestamps and sequence information may be required.

### 14.4 State Management

Operations such as aggregations and deduplication may require the engine to maintain state across events.

### 14.5 Fault Tolerance

The pipeline must recover from failures without producing incorrect results.

Checkpoints, replayable sources, and suitable sink semantics help achieve reliable recovery.

## 15. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Batch means small data | Batch can process very large datasets |
| Streaming always processes one record at a time | Some streaming engines use micro-batches |
| Streaming means zero latency | Streaming targets low latency, not zero latency |
| Kafka is a stream-processing engine | Kafka is primarily an event-streaming platform |
| Spark supports only batch processing | Spark supports batch and Structured Streaming |
| Batch cannot process historical data | Batch is commonly used for historical processing |
| Streaming is always better | The correct choice depends on business requirements |
| Micro-batch and traditional batch are identical | Micro-batch processes small, frequent batches of incoming events |
| Checkpointing guarantees exactly-once for every sink | End-to-end guarantees depend on source, processing, and sink behavior |
| Lambda and Kappa are processing engines | They are architectural patterns |

## 16. Interview Questions and Answers

**Q1. What is batch processing?**

Batch processing collects data over a period of time and processes it together, usually on a schedule or when a batch is ready.

**Q2. What is stream processing?**

Stream processing continuously processes incoming events as they arrive.

**Q3. What is the main difference between batch and streaming?**

Batch processes accumulated groups of records, while streaming processes continuously arriving events, usually with lower latency.

**Q4. What is processing latency?**

It is the time between data becoming available and the processing result being produced.

**Q5. What is micro-batch processing?**

It divides incoming events into small batches and processes them at regular intervals.

**Q6. Does Apache Spark support streaming?**

Yes. Spark Structured Streaming supports scalable stream processing, including micro-batch execution and supported continuous-processing capabilities.

**Q7. What is the difference between real-time and near-real-time processing?**

Real-time processing targets a defined response-time requirement, often strict. Near-real-time processing allows a small acceptable delay.

**Q8. What is Lambda Architecture?**

It uses separate batch and streaming processing paths to support historical and low-latency results.

**Q9. What is Kappa Architecture?**

It uses a primary streaming-oriented processing pipeline and can replay retained events for historical reprocessing.

**Q10. What is the role of Kafka in streaming architecture?**

Kafka stores and distributes event streams so producers and consumers can exchange data reliably.

**Q11. What are common streaming challenges?**

Late events, duplicate events, out-of-order data, state management, recovery, and end-to-end delivery guarantees.

**Q12. When would you choose batch processing?**

When the business needs periodic results, large-scale historical processing, or scheduled data warehouse updates.

**Q13. When would you choose stream processing?**

When continuously arriving events must be processed with low latency, such as fraud detection or live monitoring.

**Q14. Can batch and streaming work together?**

Yes. Many organizations use batch for historical data and streaming for new events, then combine the results.

**Q15. What is the difference between event time and processing time?**

Event time is when an event actually occurred. Processing time is when the system processes that event.

## 17. Quick Revision

| Concept | Remember |
|---|---|
| Batch Processing | Processes data in groups |
| Stream Processing | Processes continuously arriving events |
| Latency | Delay before results are available |
| Micro-Batch | Processes small batches frequently |
| Real-Time | Meets a defined response-time requirement |
| Near-Real-Time | Allows a small processing delay |
| Kafka | Event streaming platform |
| Spark Structured Streaming | Stream-processing engine |
| Lambda Architecture | Separate batch and speed layers |
| Kappa Architecture | One primary streaming pipeline |
| Event Time | When event occurred |
| Processing Time | When event is processed |
| Late Data | Event arrives after expected time |
| Checkpoint | Supports recovery and progress tracking |

### One-Minute Interview Answer

Batch processing collects data over a period of time and processes it together, usually on a schedule. It is commonly used for daily reports, historical data processing, and data warehouse loading.

Stream processing continuously processes incoming events with low latency. It is commonly used for fraud detection, payment monitoring, and IoT applications.

Apache Spark supports both approaches. For batch processing, we can use PySpark DataFrames with scheduled jobs. For streaming, we can use Spark Structured Streaming with sources such as Kafka or files.

The choice depends on business latency requirements. If daily results are sufficient, batch processing is usually simpler. If the business needs timely action on incoming events, streaming is more suitable.

## 18. Self-Assessment

- [ ] Define batch processing.
- [ ] Define stream processing.
- [ ] Explain the difference between batch and streaming.
- [ ] Explain processing latency.
- [ ] Explain micro-batch processing.
- [ ] Differentiate real-time and near-real-time.
- [ ] Draw a batch architecture.
- [ ] Draw a streaming architecture.
- [ ] Explain Spark batch processing.
- [ ] Explain Spark Structured Streaming.
- [ ] Explain batch and streaming in Databricks.
- [ ] Explain Lambda Architecture.
- [ ] Explain Kappa Architecture.
- [ ] Explain late-arriving and duplicate events.
- [ ] Explain event time and processing time.
- [ ] Identify when to choose batch, streaming, or hybrid processing.

**Interview readiness:** You should be able to explain both processing models, compare their latency and architecture, write a basic PySpark batch and streaming example, and justify which approach fits a business requirement.