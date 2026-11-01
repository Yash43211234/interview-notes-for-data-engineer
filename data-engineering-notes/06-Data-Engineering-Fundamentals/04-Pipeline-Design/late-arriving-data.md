# Late-Arriving Data in Data Engineering

## 1. Introduction

**Late-arriving data is data that becomes available to a data pipeline later than expected or after the pipeline has already processed the corresponding time period.**

It is common in batch and streaming systems because data may be delayed by:
- Network interruptions.
- Source application delays.
- Offline devices.
- Delayed database transactions.
- File delivery problems.
- Different time zones.
- Upstream processing failures.

### Example

Suppose a sales pipeline processes transactions every day.

A transaction occurs on October 24, but due to a network issue, it reaches the data platform on October 26.

The pipeline has already processed October 24.

The transaction is called late-arriving data because it arrived after the expected processing time.

**Key idea:** Event occurrence time and data arrival time are not always the same.

---

## 2. Event Time vs Processing Time vs Ingestion Time

These three timestamps are important when handling late-arriving data.

### 2.1 Event Time

The actual time when an event occurred in the source system.

Example:

A customer places an order at:

```text
2026-10-24 10:30:00
```

This is the event time.

### 2.2 Ingestion Time

The time when the data enters the data platform or ingestion system.

Example:

The order reaches cloud storage at:

```text
2026-10-26 08:00:00
```

This is the ingestion time.

### 2.3 Processing Time

The time when the processing engine processes the record.

Example:

Spark processes the order at:

```text
2026-10-26 08:15:00
```

This is the processing time.

### Comparison

| Timestamp | Meaning | Example |
|---|---|---|
| Event time | When the event actually happened | Oct 24, 10:30 AM |
| Ingestion time | When data entered the platform | Oct 26, 8:00 AM |
| Processing time | When the pipeline processed it | Oct 26, 8:15 AM |

### Important Point

A record can have an old event timestamp but a recent ingestion and processing timestamp.

For accurate business reporting, event time is often more meaningful than processing time.

---

## 3. Why Does Late-Arriving Data Occur?

### 3.1 Network Delays

Devices or applications may temporarily lose network connectivity.

Events are uploaded after connectivity is restored.

### 3.2 Source System Delays

The source system may generate or export records later than expected.

### 3.3 Offline Devices

Mobile applications, IoT devices, and remote systems may store events locally before sending them.

### 3.4 Upstream Pipeline Failure

An upstream job may fail and deliver its output after recovery.

### 3.5 Delayed Transactions

A database transaction may be committed or exposed to downstream systems later than expected.

### 3.6 File Delivery Delays

A source file may arrive hours or days after its expected delivery time.

---

## 4. Why Is Late-Arriving Data a Problem?

Late-arriving data can make previously generated results incomplete or incorrect.

Common problems include:

- Incorrect daily sales totals.
- Missing customer activity.
- Inaccurate financial reports.
- Incorrect time-window aggregations.
- Missing records in incremental pipelines.
- Inconsistent downstream tables.
- Incorrect historical analytics.

### Example: Daily Sales Report

Suppose the sales pipeline processes October 24 and reports:

| Date | Sales |
|---|---:|
| Oct 24 | ₹50,000 |

On October 26, a late transaction worth ₹5,000 arrives for October 24.

The correct total should now be ₹55,000.

The pipeline needs to update the historical result instead of treating the late transaction as October 26 sales.

---

## 5. Handling Late-Arriving Data in Batch Pipelines

Batch pipelines commonly use timestamps, overlapping extraction windows, historical reprocessing, and reconciliation.

### 5.1 Lookback Window

A lookback window re-reads a recent historical period during each incremental execution.

This helps capture records that arrived late or were updated after the previous extraction.

### Example

Suppose the last successful watermark is October 24.

Instead of reading only records after October 24, the pipeline reads from October 22.

```text
Previous Watermark: October 24
Lookback Period: 2 Days

Next Extraction:
October 22 onward
```

### PySpark Example

```python
from pyspark.sql import functions as F

last_watermark = "2026-10-24 00:00:00"

lookback_start = F.expr(
    "timestamp('2026-10-24 00:00:00') - INTERVAL 2 DAYS"
)

incremental_df = source_df.filter(
    F.col("updated_at") >= lookback_start
)
```

The target must safely handle repeated records, for example through deduplication and MERGE.

**Limitation:** A two-day lookback cannot capture a record that arrives ten days late unless another mechanism processes it.

### 5.2 Historical Reprocessing

If late data arrives outside the lookback period, the pipeline can reprocess the affected historical date or partition.

Example:

A transaction for October 10 arrives on October 26.

The pipeline can reprocess October 10 data and update the corresponding target results.

### 5.3 Source-to-Target Reconciliation

Periodically compare source and target data to identify missing or inconsistent records.

Possible checks:
- Record counts.
- Business-key differences.
- Total sales amounts.
- Missing dates.
- Duplicate records.

Reconciliation helps detect records that normal incremental extraction missed.

---

## 6. Handling Late-Arriving Data in Spark Structured Streaming

Streaming systems often receive events out of order.

For example:

```text
Event A: 10:00 AM
Event B: 10:05 AM
Event C: 10:02 AM
```

Event C arrives after Event B even though its event time is earlier.

This is called out-of-order event arrival.

Spark Structured Streaming supports event-time processing and watermarks to manage late data in stateful operations.

### 6.1 What Is a Streaming Watermark?

A streaming watermark is a mechanism that tracks event-time progress and helps Spark decide how long to maintain state for certain stateful operations.

It can also determine when certain late events are too old to be considered by a stateful operation.

Example:

```python
from pyspark.sql import functions as F

watermarked_df = events.withWatermark(
    "event_time",
    "10 minutes"
)
```

This declares a 10-minute event-time lateness threshold for supported stateful operations.

### 6.2 Watermark with Window Aggregation

```python
from pyspark.sql import functions as F

aggregated_df = (
    events
    .withWatermark("event_time", "10 minutes")
    .groupBy(
        F.window("event_time", "5 minutes")
    )
    .agg(
        F.count("*").alias("total_events")
    )
)
```

This groups events into five-minute event-time windows.

The watermark helps manage state and determine when window results can be finalized in supported output modes.

### 6.3 How Does It Work?

Suppose the configured watermark delay is 10 minutes.

As the maximum observed event time advances, Spark calculates a watermark based on event-time progress and the configured delay.

Events arriving sufficiently late may be dropped from stateful processing.

The watermark is not simply a wall-clock timer.

### Important Points

- Watermarks are based on observed event-time progress.
- They help control state retention for stateful operations.
- They support handling of late and out-of-order events.
- Events later than the effective watermark may be dropped by applicable stateful operations.
- The configured delay is not a guarantee that every event within that period will be accepted.
- Exact behavior depends on the operation, output mode, and streaming query configuration.

---

## 7. Batch Watermark vs Streaming Watermark

The word *watermark* has two different meanings in data engineering.

| Feature | Batch Incremental Watermark | Streaming Event-Time Watermark |
|---|---|---|
| Purpose | Track extraction progress | Manage event-time progress and state |
| Common use | Batch incremental pipelines | Structured Streaming |
| Typical value | Last processed timestamp or ID | Event-time progress boundary |
| Storage | Often a control table | Maintained by the streaming engine |
| Main concern | Avoid missing or repeating source changes | Handle late events and state cleanup |

**Interview point:** A batch watermark tracks source extraction progress. A streaming watermark helps manage event-time processing and state in streaming queries.

---

## 8. Late-Arriving Dimensions in Data Warehousing

Late-arriving data also occurs in dimensional data warehouses.

A fact record may arrive before its corresponding dimension record.

### Example

An order fact arrives with:

```text
customer_id = C101
```

But customer C101 is not yet available in the customer dimension table.

This creates a referential-integrity problem.

### Common Solutions

**1. Inferred Dimension Member**

Create a temporary dimension record using the available business key.

Later, update it when the complete dimension data arrives.

**2. Unknown or Placeholder Member**

Associate the fact with an unknown dimension member until the actual dimension record becomes available.

**3. Delayed Fact Processing**

Hold the fact temporarily until the required dimension record becomes available.

### Example

| customer_key | customer_id | customer_name |
|---:|---|---|
| 0 | UNKNOWN | Unknown Customer |
| 101 | C101 | Rahul |

A fact record can temporarily reference the unknown member.

When the actual customer record arrives, the warehouse can resolve the relationship according to its dimension-handling design.

### Important Point

Late-arriving facts and late-arriving dimensions are related but different:

- Late-arriving fact: A business event arrives late.
- Late-arriving dimension: Descriptive information about a business entity arrives late.

---

## 9. Handling Late Data in a Medallion Architecture

Late-arriving data can affect multiple layers of a data platform.

### Bronze Layer

Store incoming data with useful metadata such as:
- Event timestamp.
- Ingestion timestamp.
- Source file name.
- Source identifier.

Keep sufficient raw history to support reprocessing, subject to retention and governance policies.

### Silver Layer

Apply:
- Schema validation.
- Deduplication.
- Data-quality checks.
- Late-record handling.
- Business-key matching.

For current-state tables, apply updates using an appropriate upsert strategy.

### Gold Layer

Recalculate affected aggregates or update the relevant business results.

For example, a late October 24 order may require updating the October 24 sales total.

### Architecture

```text
Late-Arriving Source Data
          |
          v
Bronze Layer
(Raw Data + Ingestion Metadata)
          |
          v
Silver Layer
(Validation + Deduplication + Updates)
          |
          v
Gold Layer
(Recalculate Affected Aggregates)
          |
          v
Reports and Analytics
```

A late record does not always require rebuilding every downstream table. Reprocess the affected data and dependencies.

---

## 10. Best Practices for Handling Late-Arriving Data

### 1. Store Multiple Timestamps

Where useful, maintain:
- Event time.
- Ingestion time.
- Processing time.

This helps identify where delays occur.

### 2. Use Appropriate Lookback Windows

Choose the lookback period based on actual source delays and business requirements.

### 3. Use Idempotent Writes

Repeated processing should not create duplicate logical records.

### 4. Deduplicate Records

Use stable business keys and deterministic ordering rules.

### 5. Use CDC Where Appropriate

CDC can capture inserts, updates, and deletes when the source supports it.

### 6. Configure Streaming Watermarks Carefully

Choose a lateness threshold based on observed event delays, state-management needs, and business requirements.

### 7. Support Historical Reprocessing

Provide a way to reprocess affected dates or partitions.

### 8. Monitor Data Freshness

Track how long data takes to arrive and how much data arrives late.

### 9. Reconcile Important Data

Compare source and target results periodically.

### 10. Understand Business Impact

For financial or operational reports, determine whether historical results should be updated when late data arrives.

---

## 11. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Event time and processing time are identical | They represent different moments |
| Late data always belongs to the arrival date | It may belong to an earlier event date |
| A lookback window captures every late record | It only covers its configured historical range |
| A streaming watermark is a batch extraction watermark | They serve different purposes |
| Watermark delay guarantees acceptance of all events within that period | Actual behavior depends on event-time progress and stateful operation semantics |
| Watermarks are based on wall-clock time | They are based on observed event-time progress |
| Late data always requires a full pipeline rebuild | Often only affected partitions and downstream results need updating |
| Append mode is always safe for late data | Repeated records can create duplicates |
| Auto Loader automatically fixes late business records | It discovers files; business-level corrections need additional logic |
| Late-arriving facts and dimensions are identical | They are different warehouse scenarios |

---

## 12. Interview Questions and Answers

### Q1. What is late-arriving data?

Late-arriving data is data that becomes available later than expected or after the pipeline has already processed its corresponding time period.

### Q2. What is the difference between event time and processing time?

Event time is when the event actually occurred. Processing time is when the processing engine handled the event.

### Q3. What is ingestion time?

Ingestion time is when the record enters the data platform or ingestion system.

### Q4. Why does late-arriving data occur?

It can occur due to network delays, offline devices, source-system delays, upstream failures, delayed transactions, or late file delivery.

### Q5. How do you handle late-arriving data in batch pipelines?

I can use lookback windows, historical reprocessing, CDC, or source-to-target reconciliation. I also ensure that target writes are idempotent.

### Q6. What is a lookback window?

A lookback window re-reads a recent historical period during incremental extraction to capture delayed or late-arriving changes.

### Q7. What is a watermark in Spark Structured Streaming?

It is an event-time mechanism that tracks progress and helps manage state retention and late data in supported stateful operations.

### Q8. Does a streaming watermark guarantee that all events within the configured delay are accepted?

No. The watermark is based on observed event-time progress, and exact late-data behavior depends on the operation and query configuration.

### Q9. What happens when an event arrives after the streaming watermark?

For applicable stateful operations, sufficiently late events may be dropped because the required state may already have been cleaned up.

### Q10. What is the difference between a batch watermark and a streaming watermark?

A batch watermark tracks extraction progress. A streaming watermark tracks event-time progress and helps manage state in streaming operations.

### Q11. What is a late-arriving dimension?

It occurs when a fact record arrives before the corresponding dimension record is available in the data warehouse.

### Q12. How do you handle late-arriving dimensions?

I can use an inferred dimension member, an unknown placeholder member, or temporarily delay fact processing until the dimension record becomes available.

### Q13. How do you handle late data in a medallion architecture?

I preserve the incoming record in Bronze, validate and deduplicate it in Silver, and update affected Gold aggregates or business tables.

### Q14. How do you prevent duplicates when reprocessing late data?

I use stable business keys, deterministic deduplication, and idempotent writes such as suitable Delta MERGE operations.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Late-arriving data | Data arrives later than expected |
| Event time | When the event happened |
| Ingestion time | When data entered the platform |
| Processing time | When data was processed |
| Out-of-order data | Events arrive in a different order than their event timestamps |
| Lookback window | Re-read a recent historical period |
| Historical reprocessing | Process affected old data again |
| Batch watermark | Tracks extraction progress |
| Streaming watermark | Manages event-time progress and state |
| Late-arriving fact | Business event arrives late |
| Late-arriving dimension | Dimension information arrives late |
| Reconciliation | Compare source and target data |

### One-Minute Interview Answer

Late-arriving data is data that reaches the pipeline later than expected or after its corresponding time period has already been processed.

I distinguish event time, ingestion time, and processing time because they represent different stages of the data lifecycle.

In batch pipelines, I handle late records using lookback windows, CDC, historical reprocessing, and reconciliation. In Spark Structured Streaming, I use event-time watermarks to manage late data and state for supported stateful operations.

In a medallion architecture, I preserve the incoming data in Bronze, validate and deduplicate it in Silver, and update affected Gold aggregates. I also use idempotent writes to prevent duplicates during reprocessing.

---

## 14. Self-Assessment

- [ ] Define late-arriving data.
- [ ] Differentiate event time, ingestion time, and processing time.
- [ ] Explain common causes of late data.
- [ ] Describe how late data affects business reports.
- [ ] Explain lookback windows.
- [ ] Write a PySpark lookback-window filter.
- [ ] Explain historical reprocessing.
- [ ] Explain streaming event-time watermarks.
- [ ] Differentiate batch and streaming watermarks.
- [ ] Explain late-arriving facts and dimensions.
- [ ] Describe late-data handling in Bronze, Silver, and Gold.
- [ ] Explain how idempotency prevents duplicates during reprocessing.

**Interview readiness:** Be able to explain this scenario: *Your pipeline has already processed October 24, but an October 24 transaction arrives on October 26. How would you capture it, update the target, and ensure that the historical sales report is corrected without creating duplicates?*