

# Batch Data Platform Design

## 1. Introduction

A batch data platform is a system that collects, stores, processes, and delivers data in groups at scheduled intervals or when a defined amount of data becomes available.

Instead of processing every record immediately, the platform accumulates data and processes it together.

### Example

An e-commerce company generates millions of order records every day.

It needs to:
- Collect order data from databases and files.
- Store raw data.
- Clean and validate records.
- Calculate daily sales.
- Generate customer and product analytics.
- Deliver processed data to reporting systems.

A batch data platform performs these operations through scheduled data pipelines.

### High-Level Architecture

```text
          Data Sources
         /     |      \
        /      |       \
   Databases  APIs     Files
        \      |       /
         \     |      /
              |
              v
       Data Ingestion
              |
              v
         Raw Storage
              |
              v
      Batch Processing
       (Spark / SQL)
              |
              v
     Curated Data Storage
              |
              v
      Data Warehouse
              |
       +------+------+
       |             |
       v             v
    BI Reports    Analytics
```

---

## 2. What Is Batch Processing?

Batch processing is a data processing approach in which accumulated records are processed together.

### Example

Suppose an application generates 1 million transactions every day.

Instead of processing each transaction immediately for analytics, the system processes the accumulated data every night.

```text
Transactions Generated During the Day
                 |
                 v
           Raw Data Storage
                 |
                 v
          Scheduled Batch Job
                 |
                 v
         Data Transformation
                 |
                 v
          Daily Sales Report
```

### Characteristics

- Processes accumulated data.
- Runs at scheduled or triggered intervals.
- Handles large datasets.
- Supports historical processing.
- Often provides higher latency than real-time streaming.
- Commonly used for reporting and analytics.

### Common Use Cases

- Daily sales reports.
- Monthly financial summaries.
- Customer analytics.
- Data warehouse loading.
- Historical data processing.
- Machine learning feature preparation.
- Regulatory reporting.

---

## 3. Batch Data Platform Requirements

Before designing the platform, identify its functional and non-functional requirements.

### 3.1 Functional Requirements

The platform should support:

1. Data ingestion from multiple sources.
2. Raw data storage.
3. Data validation.
4. Data transformation.
5. Incremental and full loads.
6. Data deduplication.
7. Historical data processing.
8. Data warehouse loading.
9. Pipeline scheduling.
10. Reporting and analytics.

### 3.2 Non-Functional Requirements

| Requirement | Meaning |
|---|---|
| Scalability | Handle increasing data volumes |
| Reliability | Produce correct results consistently |
| Availability | Keep required platform services accessible |
| Performance | Complete processing within expected time |
| Data Quality | Maintain data correctness and validity |
| Security | Protect data and credentials |
| Recoverability | Support retries and recovery |
| Maintainability | Make pipelines easy to modify |
| Cost Efficiency | Control infrastructure and processing costs |

### Example Business Requirements

Suppose the business specifies:

- Process 500 GB of data every day.
- Complete daily processing before 6 AM.
- Support 2 years of historical data.
- Prevent duplicate order records.
- Provide validated sales reports.
- Recover failed pipelines without reprocessing everything unnecessarily.

These requirements guide the architecture.

---

## 4. Data Sources

A batch platform usually collects data from multiple source systems.

### Common Data Sources

| Source | Example Data |
|---|---|
| Relational Database | Orders, customers, payments |
| REST API | Customer activity, external services |
| CSV Files | Sales reports |
| JSON Files | Application events |
| Parquet Files | Analytical datasets |
| ERP Systems | Inventory and finance |
| CRM Systems | Customer information |
| Logs | Application activity |

### Example: E-Commerce Data

```text
Customers Database
Orders Database
Payments Database
Product API
Application CSV Files
          |
          v
     Ingestion Layer
```

Different sources may have different:
- Schemas.
- Data formats.
- Update frequencies.
- Authentication mechanisms.
- Data volumes.

The ingestion layer must handle these differences.

---

## 5. Data Ingestion Layer

The ingestion layer extracts data from source systems and moves it into the data platform.

### Common Ingestion Methods

**1. Full Load**

Extracts the complete source dataset.

Useful when:
- Initial historical loading is required.
- The source dataset is small.
- Complete replacement is acceptable.

**2. Incremental Load**

Extracts only new or changed records.

Useful when:
- Source datasets are large.
- Only a small percentage changes daily.
- Repeated full extraction is expensive.

**3. Change Data Capture (CDC)**

Captures database changes such as inserts, updates, and deletes.

Useful when:
- Changes must be tracked accurately.
- Source databases support CDC or transaction-log-based capture.
- Incremental processing needs delete information.

### Example: Incremental Load

Suppose the source contains:

| order_id | amount | updated_at |
|---|---:|---|
| 101 | 500 | 2026-10-30 10:00 |
| 102 | 800 | 2026-10-30 11:00 |
| 103 | 600 | 2026-10-31 09:00 |

The pipeline stores a previous successful watermark:

```text
2026-10-30 12:00:00
```

It extracts records with a later update timestamp.

```sql
SELECT *
FROM orders
WHERE updated_at > '2026-10-30 12:00:00';
```

In production, use a safe upper watermark, appropriate timestamp precision, and a strategy for late-arriving updates. A timestamp filter alone does not capture physical deletes.

### Azure Ingestion Services

- Azure Data Factory.
- Azure Functions.
- Azure Event Hubs for event-based ingestion.
- Azure Storage for landing files.

---

## 6. Storage Layer

The storage layer stores raw, processed, and curated datasets.

A common architecture is the data lake or lakehouse.

### Storage Zones

```text
             Data Lake
                 |
       +---------+---------+
       |         |         |
       v         v         v
     Bronze    Silver    Gold
       |         |         |
       v         v         v
     Raw       Cleaned   Business
     Data      Data      Data
```

### Bronze Layer

Stores raw or minimally processed source data.

Characteristics:
- Preserves source information.
- Supports historical reprocessing.
- May contain duplicates or invalid records.
- Keeps ingestion metadata where appropriate.

### Silver Layer

Stores cleaned and validated data.

Typical operations:
- Data type conversion.
- Null handling.
- Deduplication.
- Schema validation.
- Standardization.
- Data enrichment.

### Gold Layer

Stores business-ready datasets.

Examples:
- Daily sales summaries.
- Customer lifetime value.
- Product performance.
- Revenue by region.
- Monthly financial reports.

### Storage Technologies

| Technology | Purpose |
|---|---|
| Azure Data Lake Storage Gen2 | Distributed data lake storage |
| Amazon S3 | Object storage |
| Google Cloud Storage | Object storage |
| Delta Lake | Transactional table layer over data files |
| Apache Parquet | Columnar file format |
| Apache Iceberg | Open table format |

**Important:** Bronze, Silver, and Gold are logical data layers. They do not necessarily require separate storage accounts or physical storage systems.

---

## 7. Data Processing Layer

The processing layer transforms raw data into usable datasets.

### Common Processing Technologies

- Apache Spark.
- PySpark.
- Databricks.
- SQL engines.
- Apache Flink for suitable bounded or unbounded processing workloads.
- Cloud-native data processing services.

### Typical Processing Operations

1. Filtering.
2. Joining.
3. Aggregation.
4. Deduplication.
5. Data type conversion.
6. Data validation.
7. Business calculations.
8. Data enrichment.
9. Partitioning.
10. Data modeling.

### Example: PySpark Transformation

Suppose the Bronze orders dataset contains:

| order_id | customer_id | amount | status |
|---|---|---:|---|
| 101 | 1 | 500 | completed |
| 102 | 2 | 800 | cancelled |
| 103 | 1 | 700 | completed |

We want to calculate completed sales by customer.

```python
from pyspark.sql import functions as F

orders = spark.read.format("delta").load(
    "/data/bronze/orders"
)

completed_orders = orders.filter(
    F.col("status") == "completed"
)

customer_sales = (
    completed_orders
    .groupBy("customer_id")
    .agg(
        F.sum("amount").alias("total_sales"),
        F.countDistinct("order_id").alias("order_count")
    )
)

customer_sales.show()
```

Expected result:

| customer_id | total_sales | order_count |
|---|---:|---:|
| 1 | 1200 | 2 |

This is a simplified example. Production pipelines should also validate source keys, amounts, schema, and duplicate handling.

---

## 8. Orchestration Layer

Orchestration coordinates the execution of different pipeline activities.

It controls:
- Execution order.
- Dependencies.
- Scheduling.
- Retries.
- Failure handling.
- Notifications.
- Monitoring.

### Example

A daily sales pipeline may contain:

```text
       Start
         |
         v
    Extract Orders
         |
         v
     Extract Customers
         |
         v
    Validate Data
         |
         v
    Transform Data
         |
         v
    Load Gold Tables
         |
         v
     Notify Users
```

Some independent extraction activities can run in parallel.

### Common Orchestration Tools

| Tool | Purpose |
|---|---|
| Apache Airflow | Workflow orchestration |
| Azure Data Factory | Cloud data integration and orchestration |
| Databricks Workflows | Databricks job orchestration |
| Apache Oozie | Hadoop workflow scheduling |
| Prefect | Workflow orchestration |
| Dagster | Data orchestration |

### Orchestration vs Processing

| Orchestration | Processing |
|---|---|
| Coordinates activities | Executes transformations |
| Manages dependencies | Processes records |
| Schedules jobs | Performs computation |
| Handles workflow retries | Handles data-processing logic |
| Example: Airflow | Example: Spark |

An orchestrator may launch a Spark job, monitor its status, and trigger the next activity after completion.

---

## 9. Batch Pipeline Execution Flow

A complete batch data pipeline follows a defined sequence.

### Step 1: Source Extraction

Extract data from databases, APIs, or files.

### Step 2: Raw Data Ingestion

Store source data in the Bronze layer.

### Step 3: Data Validation

Check:
- Schema.
- Null values.
- Duplicate records.
- Invalid data types.
- Required fields.

### Step 4: Data Transformation

Apply business logic, joins, filters, and aggregations.

### Step 5: Curated Data Storage

Write cleaned and transformed data to Silver and Gold layers.

### Step 6: Data Quality Validation

Check business rules and reconcile source and target data.

### Step 7: Publish Data

Make validated datasets available to BI tools and downstream applications.

### Step 8: Monitoring

Track:
- Pipeline status.
- Execution duration.
- Record counts.
- Data freshness.
- Failure rates.
- Resource utilization.

---

## 10. Incremental Processing and Full Loads

A production batch platform should support both full and incremental processing.

### Full Load

Reads and processes the complete dataset.

Example:

```sql
SELECT *
FROM orders;
```

Advantages:
- Simple logic.
- Useful for initial ingestion.
- Rebuilds the target from source data.

Disadvantages:
- Expensive for large datasets.
- Repeatedly processes unchanged records.
- Can increase execution time.

### Incremental Load

Processes only new or changed records.

Example:

```sql
SELECT *
FROM orders
WHERE updated_at > :last_successful_watermark;
```

Advantages:
- Reduces processing volume.
- Improves execution efficiency.
- Suitable for large datasets.

Disadvantages:
- Requires watermark management.
- Must handle late updates.
- Requires safe recovery logic.
- Physical deletes need CDC or another deletion-detection method.

### Comparison

| Feature | Full Load | Incremental Load |
|---|---|---|
| Data processed | Entire dataset | New or changed data |
| Execution cost | Often higher | Often lower |
| Complexity | Relatively simple | More complex |
| Historical rebuild | Suitable | Requires reprocessing strategy |
| Watermark | Not required | Commonly required |
| Common use | Initial load | Daily production loads |

---

## 11. Data Quality and Validation

Data quality is essential because a successfully completed pipeline can still produce incorrect data.

### Common Data Quality Checks

| Check | Purpose |
|---|---|
| Completeness | Required values are present |
| Uniqueness | Keys are not duplicated |
| Validity | Values follow expected rules |
| Consistency | Related datasets agree |
| Accuracy | Values reflect trusted reality |
| Referential Integrity | Relationships between entities are valid |
| Freshness | Data is updated within the expected time |

### Example: Duplicate Check

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

### Example: Null Check

```sql
SELECT COUNT(*) AS invalid_records
FROM orders
WHERE order_id IS NULL
   OR customer_id IS NULL;
```

### Handling Invalid Data

A platform can:
- Reject the entire batch.
- Quarantine invalid records.
- Continue processing valid records.
- Send alerts for investigation.

The correct choice depends on business requirements. Invalid records should never be silently discarded.

---

## 12. Reliability and Failure Recovery

Batch pipelines must be designed to recover safely from failures.

### Common Failure Scenarios

- Source database unavailable.
- Network interruption.
- Storage permission failure.
- Spark job failure.
- Invalid input schema.
- Data quality failure.
- Insufficient compute resources.
- Target database unavailable.

### Recovery Techniques

**1. Retry**

Retry transient failures such as temporary network interruptions.

**2. Idempotent Processing**

Repeated execution should produce the same intended final result under defined conditions.

**3. Checkpointing**

Preserve progress for supported processing workloads.

**4. Backfill**

Reprocess historical data for missing or corrected periods.

**5. Transactional Writes**

Use transactions where supported to prevent partially visible writes.

### Example: Idempotent Delta MERGE

```python
from delta.tables import DeltaTable

target = DeltaTable.forPath(
    spark,
    "/data/silver/orders"
)

(
    target.alias("target")
    .merge(
        source_df.alias("source"),
        "target.order_id = source.order_id"
    )
    .whenMatchedUpdateAll()
    .whenNotMatchedInsertAll()
    .execute()
)
```

The source should contain at most one applicable row per target key for an unambiguous MERGE.

Also, deterministic transformations and correct handling of deletes are necessary for reliable reruns.

**Important:** A successful MERGE does not automatically make an entire multi-stage pipeline exactly-once. Idempotency must be considered across all relevant stages.

---

## 13. Partitioning and Performance Optimization

Large datasets should be organized and processed efficiently.

### Partitioning

Partitioning divides data into smaller logical groups.

Example:

```text
orders/
    order_date=2026-10-29/
    order_date=2026-10-30/
    order_date=2026-10-31/
```

A query filtering by date may read only relevant partitions when partition pruning is supported and the query uses the partition column appropriately.

### Example: Write Partitioned Data

```python
(
    customer_sales
    .write
    .format("delta")
    .mode("overwrite")
    .partitionBy("sales_date")
    .save("/data/gold/customer_sales")
)
```

### Performance Optimization Techniques

1. Use Parquet or Delta for analytical workloads.
2. Avoid unnecessary columns.
3. Filter data early where practical.
4. Optimize joins.
5. Handle data skew.
6. Avoid excessive shuffles.
7. Avoid too many small files.
8. Use appropriate partitioning.
9. Cache reused intermediate datasets when beneficial.
10. Monitor Spark UI and query execution plans.

**Important:** High-cardinality partition columns can create excessive directories and small files. Partitioning should be based on actual query patterns and data distribution.

---

## 14. Storage Format Selection

The choice of file format affects storage size, processing performance, and interoperability.

| Format | Type | Common Use |
|---|---|---|
| CSV | Text, tabular | Simple data exchange |
| JSON | Semi-structured text | APIs and nested records |
| Avro | Row-oriented binary | Serialization and data exchange |
| Parquet | Columnar | Analytical processing |
| ORC | Columnar | Analytical workloads |
| Delta Lake | Transactional table framework using Parquet files | Reliable lakehouse tables |

### Why Parquet Is Commonly Used

- Columnar storage.
- Compression support.
- Efficient analytical scans.
- Schema information.
- Column projection and predicate pushdown where supported.

### Why Delta Lake Is Useful

- ACID transaction support.
- Schema enforcement.
- Schema evolution capabilities.
- MERGE, UPDATE, and DELETE.
- Time travel.
- Integration with Spark.

Delta Lake is a table framework, not simply another file format equivalent to CSV or Parquet.

---

## 15. Batch Platform Monitoring

Monitoring helps identify pipeline failures and performance issues.

### Important Metrics

| Metric | Purpose |
|---|---|
| Pipeline Status | Success or failure |
| Execution Duration | Time required to complete |
| Records Ingested | Input volume |
| Records Processed | Processing volume |
| Records Rejected | Invalid records |
| Data Freshness | Age of available data |
| CPU Utilization | Compute usage |
| Memory Usage | Memory consumption |
| Shuffle Size | Data exchanged between executors |
| Failure Rate | Frequency of failed executions |
| Cost per Run | Operational cost |

### Example

Suppose the daily pipeline normally completes in 45 minutes.

Today it takes 3 hours.

Possible causes:
- Increased source data volume.
- Data skew.
- Expensive joins.
- Small files.
- Resource shortage.
- Slow source or target system.

Monitoring should help identify the actual bottleneck rather than simply increasing compute resources.

---

## 16. Security and Governance

A batch data platform may contain sensitive customer, financial, and business information.

Security should be applied across ingestion, storage, processing, and consumption.

### Important Security Practices

1. Use managed identities or service principals where appropriate.
2. Apply least-privilege access.
3. Encrypt data in transit and at rest.
4. Store credentials in a secrets manager.
5. Restrict network access.
6. Mask or tokenize sensitive data when required.
7. Maintain audit logs.
8. Track data lineage.
9. Apply retention and deletion policies.
10. Separate development, testing, and production environments.

### Azure Examples

| Requirement | Azure Service |
|---|---|
| Data Storage | ADLS Gen2 |
| Orchestration | Azure Data Factory |
| Processing | Azure Databricks |
| Secrets | Azure Key Vault |
| Identity | Microsoft Entra ID |
| Monitoring | Azure Monitor |
| Analytics | Azure Synapse / Power BI |

A managed identity provides authentication, but it still needs appropriate permissions on the target resource.

---

## 17. Example: Azure Batch Data Platform

Suppose we are designing a daily e-commerce data platform using Azure services.

### Architecture

```text
         Source Systems
        /      |       \
       /       |        \
   Azure SQL  REST API  CSV Files
       \       |        /
        \      |       /
               |
               v
       Azure Data Factory
               |
               v
        ADLS Gen2 Bronze
               |
               v
       Azure Databricks
          (PySpark)
               |
               v
        ADLS Gen2 Silver
               |
               v
       Azure Databricks
          (PySpark)
               |
               v
         Gold Delta Tables
               |
               v
       Synapse / SQL / BI
               |
               v
           Power BI
```

### Responsibility of Each Service

| Service | Responsibility |
|---|---|
| Azure SQL | Operational source database |
| REST API | External data source |
| Azure Data Factory | Ingestion and orchestration |
| ADLS Gen2 | Data lake storage |
| Azure Databricks | Distributed processing |
| PySpark | Transformation logic |
| Delta Lake | Transactional data tables |
| Synapse | SQL-based analytics and serving |
| Power BI | Business reporting |
| Key Vault | Secret management |
| Azure Monitor | Monitoring and alerts |

### Daily Execution

1. ADF starts the scheduled pipeline.
2. It extracts new and changed source records.
3. Raw data is stored in Bronze.
4. Databricks reads Bronze datasets.
5. PySpark performs cleaning and validation.
6. Cleaned data is stored in Silver.
7. Business aggregations generate Gold tables.
8. Data quality checks validate the output.
9. Curated datasets become available for analytics.
10. Monitoring records execution status and freshness.

### Why This Architecture?

- ADF handles orchestration and data movement.
- ADLS Gen2 provides scalable storage.
- Databricks handles distributed transformation.
- Delta Lake supports reliable table operations.
- Power BI provides reporting.
- Azure Monitor supports operational visibility.

The architecture separates orchestration, storage, processing, and consumption responsibilities.

---

## 18. Common Design Mistakes

1. Using full loads for very large datasets without justification.
2. Not maintaining ingestion metadata.
3. Updating watermarks before successful target writes.
4. Ignoring late-arriving records.
5. Assuming retries automatically prevent duplicates.
6. Using high-cardinality partition columns.
7. Ignoring small-file problems.
8. Mixing orchestration logic with transformation logic unnecessarily.
9. Publishing data before validation completes.
10. Ignoring data retention and storage costs.
11. Hardcoding credentials in notebooks.
12. Not monitoring data freshness.
13. Assuming successful job execution guarantees correct data.
14. Deleting streaming or processing checkpoints without understanding recovery implications.

---

## 19. Interview Questions and Answers

### Q1. What is a batch data platform?

A batch data platform collects, stores, transforms, and delivers accumulated data at scheduled or triggered intervals.

### Q2. What are the main components?

Data sources, ingestion, storage, processing, orchestration, data quality, monitoring, and serving.

### Q3. What is batch processing?

Processing accumulated data together rather than processing every event immediately.

### Q4. What is the difference between batch and streaming?

Batch processes bounded or accumulated data. Streaming processes continuously arriving data, often with lower latency requirements.

### Q5. What is the medallion architecture?

A data organization approach with Bronze raw data, Silver cleaned data, and Gold business-ready data.

### Q6. What is the difference between Bronze, Silver, and Gold?

Bronze preserves source data, Silver contains cleaned and validated data, and Gold contains curated datasets for business consumption.

### Q7. What is incremental loading?

Loading only new or changed data rather than repeatedly extracting the entire dataset.

### Q8. What is a watermark?

A stored progress value, commonly a timestamp or identifier, used to identify which source records should be extracted next in an incremental batch process.

### Q9. What is CDC?

Change Data Capture identifies source database changes such as inserts, updates, and deletes.

### Q10. What is orchestration?

Orchestration coordinates the scheduling, dependencies, execution, and monitoring of pipeline activities.

### Q11. What is the difference between ADF and Databricks?

ADF is primarily used for data integration and orchestration. Databricks provides distributed data processing and analytics capabilities.

### Q12. Why is Delta Lake useful?

It adds transactional table capabilities, schema management, MERGE operations, and time travel to data lake storage.

### Q13. How do you handle duplicate records?

Identify the business key, define a deterministic deduplication rule, and use appropriate SQL or PySpark operations before writing the target.

### Q14. How do you make a batch pipeline idempotent?

Use deterministic processing, stable keys, safe overwrite or MERGE patterns, and carefully managed progress tracking.

### Q15. How do you recover a failed batch pipeline?

Identify the failed stage, inspect partial outputs, correct the root cause, and rerun safely using idempotent operations or controlled backfills.

### Q16. How do you optimize a large Spark batch job?

Analyze Spark UI metrics, reduce unnecessary shuffles, optimize joins, handle skew, select appropriate partitioning, and use efficient file formats.

### Q17. How do you monitor data quality?

Track completeness, uniqueness, validity, consistency, freshness, and source-to-target reconciliation.

### Q18. How do you secure a batch data platform?

Use least-privilege access, managed identities, encryption, secret management, network restrictions, and auditing.

### Q19. What is the difference between a data lake and a data warehouse?

A data lake stores diverse data in flexible formats. A data warehouse organizes data for structured querying and analytics.

### Q20. How would you design a daily sales reporting platform?

I would ingest source data incrementally, store raw data, clean and validate it using distributed processing, build curated sales datasets, and publish them to an analytics layer with orchestration and monitoring.

---

## 20. Quick Revision

| Component | Responsibility |
|---|---|
| Data Sources | Generate business data |
| Ingestion | Extract and move data |
| Bronze | Store raw data |
| Silver | Clean and validate data |
| Gold | Store business-ready data |
| Processing | Transform and aggregate records |
| Orchestration | Schedule and coordinate activities |
| Data Quality | Validate correctness |
| Monitoring | Track pipeline health |
| Serving | Make data available to consumers |

### Important Concepts

- Batch processing handles accumulated data.
- Full load extracts the complete dataset.
- Incremental load extracts new or changed data.
- CDC captures source database changes.
- Watermarks track incremental extraction progress.
- Delta Lake supports transactional data lake tables.
- Idempotency enables safe reruns.
- Backfills reprocess historical data.
- Partitioning can improve query performance.
- Monitoring must cover both pipeline health and data health.

### Remember These Five Points

1. Separate ingestion, storage, processing, and orchestration.
2. Use incremental processing when repeated full extraction is unnecessary.
3. Keep raw data available for recovery and reprocessing.
4. Make pipeline writes safe for retries and reruns.
5. Validate data before publishing business-ready datasets.

---

## 21. Self-Assessment

Try answering these questions without looking at the notes.

1. What is a batch data platform?
2. Explain batch processing with an example.
3. What are the functional requirements of a batch platform?
4. What are the non-functional requirements?
5. What are the major components of a batch architecture?
6. What is the responsibility of the ingestion layer?
7. Explain full load vs incremental load.
8. What is CDC?
9. What is a watermark?
10. Explain Bronze, Silver, and Gold.
11. What is the role of Spark in a batch platform?
12. What is the role of ADF or Airflow?
13. Explain orchestration vs processing.
14. How do you handle duplicate records?
15. How do you make a pipeline idempotent?
16. How do you recover from pipeline failure?
17. How do you optimize a large batch job?
18. What is the role of Delta Lake?
19. How do you monitor pipeline health and data quality?
20. Design an Azure-based batch data platform for daily e-commerce reporting.

**Interview readiness:** You should be able to draw the architecture, explain each layer, describe full and incremental ingestion, explain the role of Spark and orchestration, and discuss reliability, data quality, performance, security, and recovery.

