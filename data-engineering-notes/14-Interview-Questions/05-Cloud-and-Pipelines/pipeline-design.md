# Data Pipeline Design – Interview Questions and Concepts

## 1. What Is a Data Pipeline?

A data pipeline is a sequence of processes that moves data from one or more sources to a destination, applying required transformations and validations along the way.

A typical pipeline performs:

1. Data extraction.
2. Data ingestion.
3. Data storage.
4. Data validation.
5. Data transformation.
6. Data loading.
7. Monitoring and error handling.

Example:

```text
Source Systems
      |
      v
Data Ingestion
      |
      v
Raw Storage
      |
      v
Data Validation
      |
      v
Data Transformation
      |
      v
Curated Storage
      |
      v
Reporting / Analytics
```

The objective is to deliver accurate, reliable, secure, and usable data.

## 2. What Are the Main Components of a Data Pipeline?

| Component | Responsibility |
|---|---|
| Source | Generates the data |
| Ingestion | Extracts or receives data |
| Storage | Stores raw and processed data |
| Processing | Cleans and transforms data |
| Orchestration | Coordinates pipeline execution |
| Data quality | Checks data correctness |
| Serving | Makes data available to consumers |
| Monitoring | Tracks pipeline and data health |

### Source Systems

Sources can include:
- Relational databases.
- APIs.
- CSV and JSON files.
- Application logs.
- Message brokers.
- SaaS applications.
- On-premises systems.

### Ingestion Layer

The ingestion layer brings data from sources into the data platform.

Common approaches:
- Full load.
- Incremental load.
- Change Data Capture (CDC).
- Batch ingestion.
- Streaming ingestion.

### Storage Layer

Stores incoming and processed data.

Common technologies:
- Azure Data Lake Storage Gen2.
- Amazon S3.
- Google Cloud Storage.
- Delta Lake.
- Cloud data warehouses.

### Processing Layer

Applies transformations such as:
- Filtering.
- Deduplication.
- Data type conversion.
- Joining datasets.
- Aggregation.
- Business-rule implementation.
- Schema validation.

### Orchestration Layer

Controls:
- Execution order.
- Scheduling.
- Dependencies.
- Retries.
- Failure handling.
- Notifications.

Examples: Azure Data Factory, Airflow, Databricks Workflows.

---

## 3. Batch vs Streaming Pipelines

### What Is Batch Processing?

Batch processing collects data over a period and processes it together.

Example:

A pipeline runs every night at 1 AM and processes the previous day's transactions.

```text
Source Data
     |
     v
Collect Data
     |
     v
Scheduled Batch
     |
     v
Transform
     |
     v
Target Table
```

Characteristics:
- Processes bounded datasets.
- Runs on a schedule or when triggered.
- Often suitable for historical reporting.
- Can be simpler to operate than continuous streaming.

### What Is Streaming Processing?

Streaming processing handles continuously arriving events, often with low latency.

Example:

An e-commerce platform processes purchase events as customers place orders.

```text
Applications
     |
     v
Event Hubs / Kafka
     |
     v
Streaming Processor
     |
     v
Storage / Analytics
```

Characteristics:
- Handles continuously arriving data.
- Often uses event-time and processing-time concepts.
- Requires careful handling of late events and duplicates.
- Maintains state for operations such as windowed aggregation.

### Batch vs Streaming

| Batch | Streaming |
|---|---|
| Processes bounded data | Processes continuously arriving data |
| Scheduled or triggered | Continuously running or periodically triggered |
| Usually higher latency | Can provide lower latency |
| Suitable for historical processing | Suitable for real-time or near-real-time processing |
| Easier to replay a bounded input | Requires event retention, offsets, checkpoints, and state recovery |

### What Is Micro-Batch Processing?

Micro-batch processing divides incoming data into small batches and processes them repeatedly.

Spark Structured Streaming commonly uses micro-batch execution.

It can provide low-latency processing without processing every event individually.

**Interview point:** Streaming does not automatically mean event-by-event processing. A streaming system may use micro-batches.

---

## 4. ETL vs ELT

### What Is ETL?

ETL stands for:

- Extract.
- Transform.
- Load.

Data is transformed before it reaches the target system.

```text
Source
   |
   v
Extract
   |
   v
Transform
   |
   v
Load
   |
   v
Target
```

### What Is ELT?

ELT stands for:

- Extract.
- Load.
- Transform.

Data is first loaded into the target storage or platform, then transformed there.

```text
Source
   |
   v
Extract
   |
   v
Load Raw Data
   |
   v
Transform in Target
   |
   v
Curated Data
```

### ETL vs ELT

| ETL | ELT |
|---|---|
| Transformation happens before loading into the target | Transformation happens after loading |
| Often uses a separate processing engine | Uses target platform compute |
| Can reduce the amount of data loaded into the target | Can preserve raw data for later transformations |
| Common in traditional integration systems | Common in modern lakehouse and warehouse platforms |

**Important:** ETL and ELT describe transformation and loading order. They do not determine whether a pipeline is batch or streaming.

---

## 5. Full Load vs Incremental Load

### What Is Full Load?

Full load extracts and loads the complete dataset.

Example:

```sql
SELECT *
FROM customers;
```

Advantages:
- Simple implementation.
- Useful for initial loading.
- Can rebuild a target from source data.

Disadvantages:
- More data transferred.
- Higher processing cost.
- Longer execution time for large datasets.

### What Is Incremental Load?

Incremental loading processes only new or changed records since a previous successful run.

Example:

```sql
SELECT *
FROM customers
WHERE modified_at > :last_watermark;
```

Advantages:
- Less data processing.
- Lower transfer cost.
- Faster execution for many workloads.

Disadvantages:
- Requires reliable change detection.
- Late updates can be missed.
- Deletes need separate handling.
- Requires checkpoint or watermark management.

### Full Load vs Incremental Load

| Full Load | Incremental Load |
|---|---|
| Processes all records | Processes new or changed records |
| Simple to understand | Requires change tracking |
| More expensive for large datasets | Usually more efficient |
| Useful for initial load and small tables | Useful for large, frequently updated tables |

### What Is a Watermark?

A watermark is a stored source position used to identify the next set of records to process.

For timestamp-based ingestion, it may contain the latest successfully processed modification timestamp.

Example:

| Table | Last Watermark |
|---|---|
| Customers | 2026-10-30 10:00:00 |
| Orders | 2026-10-30 11:00:00 |

A typical incremental pipeline:

```text
Read Previous Watermark
          |
          v
Determine Current Upper Bound
          |
          v
Extract Changed Records
          |
          v
Transform and Validate
          |
          v
Write to Target
          |
          v
Verify Successful Commit
          |
          v
Update Watermark
```

**Important:** Advance the watermark only after the target write has succeeded and the required validation has passed.

### What Is the Problem with Timestamp-Based Incremental Loading?

Suppose a pipeline loads records with:

```sql
WHERE modified_at > :last_watermark
```

Potential problems:
- A record is updated late.
- Two records have the same timestamp.
- Source and target clocks differ.
- A record is deleted.
- The source timestamp is not reliably maintained.

Possible solutions:
- Use CDC where available.
- Use a composite watermark such as timestamp plus unique ID.
- Read an overlapping time window.
- Perform periodic reconciliation.
- Capture deletes using CDC or soft-delete flags.

---

## 6. Change Data Capture (CDC)

### What Is CDC?

Change Data Capture identifies changes made to source data.

It can capture:
- INSERT.
- UPDATE.
- DELETE.

Instead of repeatedly extracting an entire table, the pipeline processes only captured changes.

### Common CDC Approaches

**1. Timestamp-Based CDC**

Uses a column such as `updated_at` to detect changed rows.

Limitation: Physical deletes and unreliable timestamps are not handled automatically.

**2. Log-Based CDC**

Reads database transaction logs to identify changes.

Advantages:
- Can capture inserts, updates, and deletes.
- Often avoids repeated full-table scans.
- Can provide detailed change information.

**3. Trigger-Based CDC**

Uses database triggers to record changes.

It may add workload to the source database and requires careful implementation.

### CDC Event Structure

A CDC record may contain:

```json
{
  "customer_id": 101,
  "operation": "UPDATE",
  "name": "Rahul",
  "updated_at": "2026-10-31T10:30:00"
}
```

Actual CDC schemas depend on the source connector.

### How Do You Apply CDC to a Target Table?

A common approach is:

1. Read CDC events.
2. Identify the business key.
3. Deduplicate events using a reliable source sequence or version.
4. Apply inserts and updates using MERGE.
5. Apply deletes using the source delete indicator.
6. Record the processed source position.

Example:

```sql
MERGE INTO target_customers AS target
USING source_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED AND source.operation = 'DELETE'
THEN DELETE

WHEN MATCHED
THEN UPDATE SET
    target.name = source.name,
    target.updated_at = source.updated_at

WHEN NOT MATCHED AND source.operation <> 'DELETE'
THEN INSERT (
    customer_id,
    name,
    updated_at
)
VALUES (
    source.customer_id,
    source.name,
    source.updated_at
);
```

This is illustrative SQL. Exact syntax and CDC handling vary by platform.

**Important:** If multiple changes for the same key occur in one batch, select the correct latest event deterministically before applying the MERGE.

---

## 7. Idempotency and Exactly-Once Processing

### What Is Idempotency?

An operation is idempotent when repeating it produces the same intended final result as executing it once, under the defined conditions.

Example:

If a pipeline fails after writing data but before recording success, the pipeline may retry.

Without idempotent logic, the same records may be inserted twice.

With an appropriate MERGE or deterministic overwrite strategy, the retry can produce the intended target state.

### How Do You Make a Pipeline Idempotent?

- Define business keys.
- Deduplicate incoming records.
- Use deterministic transformations.
- Use MERGE or controlled partition replacement.
- Maintain checkpoints and watermarks correctly.
- Make external side effects retry-safe.
- Validate target state after failures.

### What Is Exactly-Once Processing?

Exactly-once processing means the system produces the intended result once despite retries or failures, within a clearly defined source-to-sink guarantee.

A checkpoint alone does not guarantee exactly-once results for every external sink.

A pipeline may use:
- Source offset tracking.
- Transactional writes.
- Idempotent sink operations.
- Stable event identifiers.
- Atomic commits.

**Interview point:** Explain exactly-once as an end-to-end property of the source, processing engine, and sink. Do not claim that every retry or checkpoint automatically guarantees it.

### At-Least-Once vs At-Most-Once vs Exactly-Once

| Delivery Semantics | Meaning |
|---|---|
| At-most-once | An event is processed zero or one time; loss may occur |
| At-least-once | An event is processed one or more times; duplicates may occur |
| Exactly-once | The defined processing result is reflected once, under the system's guarantees |

---

## 8. Data Pipeline Orchestration

### What Is Orchestration?

Orchestration coordinates the execution of pipeline tasks.

It manages:
- Task dependencies.
- Execution order.
- Scheduling.
- Retries.
- Failure handling.
- Notifications.
- Parameters.

Example:

```text
Task 1: Extract Customers
          |
          v
Task 2: Validate Customers
          |
          v
Task 3: Transform Customers
          |
          v
Task 4: Load Gold Table
          |
          v
Task 5: Refresh Report
```

### What Is the Difference Between Orchestration and Processing?

| Orchestration | Processing |
|---|---|
| Coordinates tasks | Executes data transformations |
| Manages dependencies | Filters, joins, aggregates, and cleans data |
| Controls schedules and retries | Uses compute resources to process data |
| Example: ADF, Airflow | Example: Spark, Databricks |

ADF can orchestrate a Databricks notebook, while Databricks executes the transformation.

### What Is a DAG?

DAG stands for Directed Acyclic Graph.

It represents tasks and their dependencies.

- Directed: Tasks have execution relationships.
- Acyclic: A dependency path cannot form a cycle.
- Graph: Tasks are represented as nodes and dependencies as edges.

Example:

```text
       Extract
       /     \
      v       v
 Customers  Orders
      \       /
       v     v
       Join
         |
         v
      Aggregate
```

The customer and order tasks can run independently if their dependencies allow it.

### What Is Parallel Execution?

Parallel execution runs independent tasks at the same time.

Benefits:
- Reduces total execution time.
- Improves resource utilization.

Risks:
- Resource contention.
- Source-system overload.
- Concurrent writes to the same target.
- Increased operational complexity.

Parallelism should be based on dependency and resource limits.

---

## 9. Data Quality in Pipelines

### What Is Data Quality?

Data quality measures whether data is suitable for its intended use.

Important dimensions:

| Dimension | Meaning |
|---|---|
| Completeness | Required data is present |
| Accuracy | Values correctly represent reality |
| Validity | Values follow defined rules |
| Uniqueness | Duplicate business records are controlled |
| Consistency | Data agrees across systems |
| Timeliness | Data arrives within the required period |
| Integrity | Relationships and constraints are maintained |

### What Data Quality Checks Would You Add?

Example checks:

**1. Null Check**

```sql
SELECT COUNT(*)
FROM customers
WHERE customer_id IS NULL;
```

**2. Duplicate Check**

```sql
SELECT customer_id, COUNT(*)
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

**3. Validity Check**

```sql
SELECT COUNT(*)
FROM orders
WHERE amount < 0;
```

Whether negative amounts are invalid depends on the business rules; refunds may legitimately be represented as negative values.

**4. Referential Integrity Check**

```sql
SELECT o.customer_id
FROM orders o
LEFT JOIN customers c
    ON o.customer_id = c.customer_id
WHERE c.customer_id IS NULL;
```

**5. Freshness Check**

Verify that the latest available business event or source update is within the expected freshness threshold.

### What Is Schema Evolution?

Schema evolution is the process of handling changes in a dataset's structure over time.

Examples:
- Adding a column.
- Renaming a column.
- Changing a data type.
- Removing a column.

A safe pipeline should:
- Detect schema changes.
- Validate compatibility.
- Apply approved changes.
- Preserve unexpected data where supported.
- Alert owners about breaking changes.

Do not automatically accept every schema change. A new field or changed type may alter business meaning.

---

## 10. Data Storage and File Formats

### What Are Common Data File Formats?

| Format | Type | Typical Use |
|---|---|---|
| CSV | Text, row-oriented | Simple tabular exchange |
| JSON | Semi-structured text | APIs and nested data |
| XML | Semi-structured text | Structured document exchange |
| Avro | Row-oriented binary | Data exchange and streaming |
| Parquet | Columnar binary | Analytics |
| ORC | Columnar binary | Analytical workloads |

### Why Is Parquet Commonly Used in Data Engineering?

Parquet is a columnar storage format.

Benefits:
- Efficient column reads.
- Compression.
- Encoding.
- Schema information.
- Better analytical scan performance for many workloads.

Example:

A query needs only `customer_id` and `amount` from a wide dataset. A columnar format can read the required columns rather than scanning every column.

### What Is Partitioning?

Partitioning divides data into logical or physical groups based on selected column values.

Example:

```text
orders/
    year=2026/
        month=10/
            part-0001.parquet
            part-0002.parquet
```

Benefits:
- Can reduce data scanned.
- Helps organize large datasets.
- Can improve query performance.

Risks:
- Too many partitions.
- Small files.
- Skewed partition sizes.
- High-cardinality partition columns.

Partition columns should be selected based on query patterns and data distribution.

### What Is the Difference Between Partitioning and Bucketing?

**Partitioning:** Divides data based on column values, often producing directory-level organization.

**Bucketing:** Distributes records into a fixed number of buckets based on a hash of selected columns.

Bucketing can help some join and aggregation workloads, but support and optimization behavior depend on the processing engine.

---

## 11. Error Handling and Recovery

### What Are Common Reasons for Pipeline Failure?

- Source connectivity failure.
- Authentication or authorization errors.
- Network problems.
- Schema mismatch.
- Invalid records.
- Insufficient compute resources.
- Out-of-memory errors.
- Storage failures.
- Incorrect transformation logic.
- Concurrent writes.
- Source-system downtime.

### How Should You Handle Bad Records?

A common approach is to separate valid and invalid records.

```text
Incoming Data
      |
      v
Validation
    /   \
   v     v
Valid   Invalid
   |       |
   v       v
Silver   Quarantine
```

A quarantine area should retain useful diagnostic information such as:
- Original record or reference.
- Validation error.
- Source identifier.
- Ingestion timestamp.
- Pipeline run ID.

Do not silently discard bad records unless the business explicitly permits it.

### What Is Retry Logic?

Retry logic repeats a failed operation when the failure may be temporary.

Examples:
- Temporary network errors.
- Rate limits.
- Service unavailability.

Use exponential backoff where appropriate.

Do not repeatedly retry permanent errors such as invalid schema or insufficient permissions without addressing the root cause.

### What Is Dead-Letter Storage?

A dead-letter location stores messages or records that cannot be processed successfully after the configured handling attempts.

It helps:
- Prevent bad messages from blocking processing.
- Preserve failed records.
- Support investigation.
- Allow controlled reprocessing.

### What Is Checkpointing?

Checkpointing stores progress and, for stateful processing, recovery information.

In streaming systems, checkpoint data may include:
- Source progress.
- State information.
- Commit metadata.

Checkpointing supports recovery, but it does not replace idempotent sink logic or business-level reconciliation.

### What Is Reprocessing?

Reprocessing means processing previously ingested data again.

It is useful when:
- Transformation logic changes.
- A pipeline has a bug.
- Data quality issues are corrected.
- Historical records need rebuilding.

A reliable design preserves raw data and supports controlled backfills.

---

## 12. Pipeline Monitoring and Observability

### What Is Pipeline Observability?

Observability is the ability to understand pipeline behavior using logs, metrics, traces, and data-quality signals.

Monitor three areas:

**1. Operational Health**
- Job success and failure.
- Execution duration.
- Retry count.
- Resource utilization.
- Error rate.

**2. Data Health**
- Input and output row counts.
- Duplicate count.
- Null count.
- Schema changes.
- Rejected records.

**3. Business Health**
- Revenue totals.
- Order counts.
- Customer activity.
- Business KPI consistency.
- Reporting freshness.

### What Is Data Freshness?

Data freshness measures how recently data has been updated or made available for use.

Example:

A daily sales report is expected to be updated by 7 AM.

If the latest complete data is from two days earlier, the freshness requirement has not been met.

### What Is Data Lineage?

Data lineage tracks where data originated and how it moved through systems.

Example:

```text
Azure SQL
    |
    v
ADF Copy Activity
    |
    v
ADLS Bronze
    |
    v
Databricks Transformation
    |
    v
Delta Silver
    |
    v
Gold Sales Table
    |
    v
Power BI Report
```

Lineage helps with:
- Impact analysis.
- Debugging.
- Auditing.
- Governance.
- Understanding dependencies.

---

## 13. End-to-End Azure Data Pipeline Design

### Scenario: Design an E-Commerce Data Pipeline

**Requirement:** Build a data platform that processes customer, product, order, and payment data.

The system should support:
- Daily reporting.
- Incremental data ingestion.
- Data validation.
- Historical analysis.
- Secure storage.
- Pipeline monitoring.

### Proposed Architecture

```text
         Source Systems
        /      |       \
       v       v        v
   Azure SQL  REST API  Event Hubs
       \       |        /
        \      |       /
         v     v      v
       Azure Data Factory
        /              \
       v                v
  Batch Ingestion   Event Ingestion
       |                |
       v                v
             ADLS Gen2
             Bronze Layer
                  |
                  v
           Azure Databricks
           Cleaning and Validation
                  |
                  v
             Silver Layer
                  |
                  v
          Business Transformations
                  |
                  v
               Gold Layer
                  |
                  v
          Power BI / Analytics
```

### Layer Responsibilities

**Bronze Layer**
- Stores source data in its raw or minimally processed form.
- Preserves source information.
- Records ingestion metadata.
- Supports reprocessing.

**Silver Layer**
- Cleans and standardizes data.
- Applies schema validation.
- Handles duplicates.
- Applies CDC changes.
- Resolves data-quality issues.

**Gold Layer**
- Stores business-ready datasets.
- Contains facts and dimensions.
- Supports reporting and analytics.
- Implements agreed business metrics.

### Example Data Flow

1. ADF extracts customer and product data from Azure SQL.
2. ADF copies the data into ADLS Bronze.
3. Databricks reads Bronze datasets.
4. PySpark validates schemas and cleans records.
5. Duplicate records are removed using business keys and deterministic ordering.
6. CDC changes are applied to Silver Delta tables.
7. Gold fact and dimension tables are created.
8. Power BI consumes Gold datasets.
9. Pipeline and data-quality metrics are monitored.

### Security Design

- Use managed identities where supported.
- Store required secrets in Azure Key Vault.
- Apply least-privilege RBAC.
- Configure ADLS ACLs where needed.
- Use private networking when required.
- Audit access to sensitive data.

### Reliability Design

- Maintain pipeline run IDs.
- Use retry policies for transient failures.
- Use idempotent target writes.
- Store watermarks or source offsets.
- Preserve raw data.
- Support backfills and reprocessing.
- Reconcile source and target data.

---

## 14. Common Pipeline Design Interview Scenarios

### Q1. How Would You Design a Pipeline for a 1 TB Daily Dataset?

Approach:

1. Understand source format and update frequency.
2. Check whether the entire 1 TB must be processed daily.
3. Identify incremental or CDC capabilities.
4. Estimate throughput and processing windows.
5. Select scalable distributed storage.
6. Use distributed processing such as Spark where appropriate.
7. Partition data based on access patterns.
8. Avoid excessive small files.
9. Monitor throughput, failures, and data quality.
10. Test performance using realistic data volumes.

Do not choose cluster size based only on dataset size. Consider file format, compression, transformation complexity, skew, concurrency, and SLA.

### Q2. How Would You Handle Late-Arriving Data?

1. Distinguish event time from ingestion time.
2. Preserve original event timestamps.
3. Define an acceptable late-arrival window.
4. Use lookback or CDC where appropriate.
5. Reprocess affected partitions or records.
6. Update aggregates and downstream tables.
7. Monitor late-arrival rates.

For streaming stateful operations, event-time watermarks help bound state and control how long late events are accepted.

### Q3. How Would You Prevent Duplicate Records?

1. Identify the business key or event ID.
2. Determine the source's duplicate behavior.
3. Define deterministic ordering.
4. Deduplicate incoming records.
5. Use MERGE or another idempotent write strategy.
6. Reconcile target records.

`dropDuplicates()` alone may not guarantee which duplicate record is retained. Use a deterministic rule when the latest or highest-priority record must win.

### Q4. How Would You Handle Schema Changes?

1. Detect schema differences.
2. Identify additive and breaking changes.
3. Validate the change with the data owner.
4. Apply approved schema evolution.
5. Route incompatible records appropriately.
6. Update downstream consumers.
7. Record and monitor the schema version.

### Q5. How Would You Process Data from Multiple Sources?

1. Understand each source's schema and update behavior.
2. Ingest each source independently.
3. Preserve source metadata.
4. Standardize data types and formats.
5. Validate each source.
6. Join using correct business keys.
7. Handle missing and conflicting records.
8. Publish curated datasets.

### Q6. How Would You Handle a Pipeline That Takes Longer Every Day?

Investigate:
- Increasing source volume.
- Growing target-table size.
- Poor partition pruning.
- Data skew.
- Join strategy.
- Shuffle and spill.
- Small files.
- Inefficient incremental logic.
- Resource contention.

Compare historical execution metrics to identify the bottleneck before changing compute or code.

### Q7. What Would You Do if the Source and Target Row Counts Do Not Match?

1. Confirm that the source and target use the same processing window.
2. Check source extraction filters.
3. Check duplicate handling.
4. Check rejected records.
5. Check deletes and updates.
6. Check join fanout or filtering.
7. Compare business keys.
8. Compare important values and aggregates.
9. Reconcile and correct missing or extra records.

Row counts alone are not sufficient to prove data correctness.

### Q8. How Would You Design a Pipeline for Multiple Tables?

Use a metadata-driven framework.

Store configuration such as:
- Source connection reference.
- Source table.
- Target path.
- Load type.
- Watermark column.
- Primary key.
- Schema version.
- Data-quality rules.

Use a reusable orchestration pipeline to process each configured table.

Keep table-specific logic separate when business rules differ.

---

## 15. Important Design Trade-Offs

| Decision | Trade-Off |
|---|---|
| Batch vs streaming | Simplicity and cost vs lower latency |
| Full vs incremental load | Simplicity vs processing efficiency |
| ETL vs ELT | Pre-load control vs target-platform flexibility |
| Parquet vs CSV | Analytical efficiency vs simple interchange |
| Managed vs self-hosted runtime | Operational simplicity vs network control |
| Centralized vs distributed orchestration | Simplicity vs team and workload independence |
| Strict validation vs quarantine | Strong quality enforcement vs pipeline availability |
| Partitioning vs no partitioning | Data skipping vs small-file and metadata overhead |
| Immediate retry vs delayed retry | Recovery speed vs pressure on an unhealthy service |

A good design is based on business requirements, data volume, latency, cost, security, and operational capabilities—not on choosing the most complex architecture.

---

## 16. Common Pipeline Design Mistakes

1. Designing before understanding business requirements.
2. Ignoring source data volume and update frequency.
3. Using full loads for every large table without justification.
4. Advancing watermarks before successful target writes.
5. Assuming timestamp-based loading captures deletes.
6. Ignoring duplicate and late-arriving records.
7. Using non-idempotent append operations with automatic retries.
8. Failing to preserve raw source data.
9. Treating row-count equality as complete reconciliation.
10. Ignoring schema evolution.
11. Creating too many small files.
12. Choosing partition columns without considering query patterns.
13. Running every independent task sequentially.
14. Running too many tasks in parallel without resource limits.
15. Retrying permanent failures indefinitely.
16. Logging sensitive information.
17. Monitoring job status but not data quality.
18. Failing to design backfill and recovery procedures.

---

## 17. Interview Questions and Short Answers

**Q1. What is a data pipeline?**

A sequence of processes that ingests, stores, transforms, validates, and delivers data.

**Q2. What is the difference between ETL and ELT?**

ETL transforms before loading into the target. ELT loads first and transforms inside the target platform.

**Q3. What is incremental loading?**

Processing only new or changed records since a previous successful source position.

**Q4. What is CDC?**

Change Data Capture identifies inserts, updates, and deletes in source data.

**Q5. What is a watermark?**

A stored source position used to determine which records should be processed next.

**Q6. What is idempotency?**

Repeating an operation produces the same intended final result under defined conditions.

**Q7. What is orchestration?**

Coordinating pipeline execution, dependencies, schedules, retries, and failures.

**Q8. What is a DAG?**

A Directed Acyclic Graph representing tasks and their dependencies.

**Q9. What is data lineage?**

Tracking where data comes from and how it moves through systems.

**Q10. What is schema evolution?**

Handling structural changes in datasets over time.

**Q11. What is a dead-letter queue or location?**

A place to retain messages or records that cannot be processed successfully.

**Q12. What is checkpointing?**

Saving processing progress and recovery information.

**Q13. Does checkpointing guarantee exactly-once processing?**

No. End-to-end guarantees also depend on source behavior, processing semantics, and sink writes.

**Q14. What is data freshness?**

How recently data has been updated and made available to consumers.

**Q15. What is a backfill?**

Processing historical data again to populate or correct a target.

**Q16. What is a metadata-driven pipeline?**

A reusable pipeline that reads configuration metadata to determine which sources and targets to process and how.

**Q17. Why use Bronze, Silver, and Gold layers?**

They separate raw ingestion, cleaned standardized data, and business-ready datasets.

**Q18. What is the difference between orchestration and transformation?**

Orchestration controls when and how tasks run; transformation changes the data itself.

**Q19. Why is idempotency important?**

It makes retries and recovery safer by preventing repeated execution from creating unintended results.

**Q20. What should be monitored in a production pipeline?**

Execution health, data quality, data freshness, throughput, resource utilization, and business metrics.

---

## 18. Quick Revision

| Concept | Key Point |
|---|---|
| Data Pipeline | End-to-end data movement and processing |
| Batch | Processes bounded datasets |
| Streaming | Processes continuously arriving data |
| Micro-batch | Processes streaming input in small batches |
| ETL | Extract, Transform, Load |
| ELT | Extract, Load, Transform |
| Full Load | Processes complete dataset |
| Incremental Load | Processes new or changed data |
| CDC | Captures source changes |
| Watermark | Tracks processed source position |
| Idempotency | Safe repeated execution |
| Exactly-Once | Defined result reflected once under system guarantees |
| Orchestration | Coordinates tasks |
| DAG | Directed Acyclic Graph |
| Data Quality | Measures data suitability |
| Schema Evolution | Handles structural changes |
| Checkpoint | Stores progress and recovery information |
| Dead Letter | Retains failed records |
| Backfill | Reprocesses historical data |
| Data Lineage | Tracks data movement and origin |
| Data Freshness | Measures data recency |
| Bronze | Raw data |
| Silver | Cleaned and standardized data |
| Gold | Business-ready data |

---

## 19. Self-Assessment Checklist

### Pipeline Fundamentals
- [ ] Explain the components of a data pipeline.
- [ ] Differentiate batch and streaming.
- [ ] Explain ETL and ELT.
- [ ] Differentiate full and incremental loads.
- [ ] Explain CDC and watermarking.

### Reliability
- [ ] Explain idempotency.
- [ ] Explain delivery semantics.
- [ ] Explain checkpointing.
- [ ] Explain retry and recovery.
- [ ] Design a backfill process.

### Orchestration
- [ ] Explain DAGs and task dependencies.
- [ ] Explain parallel execution.
- [ ] Differentiate orchestration and processing.
- [ ] Design a metadata-driven pipeline.

### Data Quality
- [ ] Explain data-quality dimensions.
- [ ] Write null and duplicate checks.
- [ ] Explain schema evolution.
- [ ] Explain late-arriving data.
- [ ] Explain source-to-target reconciliation.

### System Design
- [ ] Design an end-to-end Azure data pipeline.
- [ ] Design a 1 TB daily ingestion pipeline.
- [ ] Explain Bronze, Silver, and Gold.
- [ ] Explain monitoring and observability.
- [ ] Discuss cost, reliability, latency, and security trade-offs.

**Preparation priority:** Be able to explain one complete pipeline from source to reporting, including incremental ingestion, CDC, data validation, idempotent writes, failure recovery, monitoring, and security. Practice explaining the reason behind each design decision rather than memorizing definitions.