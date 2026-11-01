

# Data Ingestion Framework Design

## 1. Introduction

A **Data Ingestion Framework** is a reusable system that collects data from different source systems and transfers it into a centralized storage or processing platform.

It provides a consistent approach to extracting, receiving, validating, and storing data.

Instead of building completely separate ingestion logic for every source, an organization can create a reusable framework that handles common ingestion requirements.

### Example

An e-commerce company receives data from:

- Customer database.
- Orders database.
- Payment APIs.
- Product CSV files.
- Application event logs.

A data ingestion framework collects these datasets and moves them into a data lake for further processing.

```text
          Data Sources
         /     |      \
        /      |       \
   Databases  APIs     Files
        \      |      /
         \     |     /
              |
              v
       Ingestion Framework
              |
              v
          Raw Storage
              |
              v
       Processing Pipeline
              |
              v
        Analytics Platform
```

---

## 2. Why Do We Need an Ingestion Framework?

Without a reusable framework, every data source may require independently developed code.

This can lead to:

- Repeated implementation.
- Inconsistent error handling.
- Different logging mechanisms.
- Difficult maintenance.
- Duplicate ingestion logic.
- Inconsistent metadata tracking.
- More effort when adding new sources.

A common ingestion framework standardizes these operations.

### Benefits

| Benefit | Explanation |
|---|---|
| Reusability | Reuse ingestion logic across sources |
| Scalability | Add more sources without redesigning everything |
| Maintainability | Centralize common logic |
| Reliability | Standardize retries and failure handling |
| Monitoring | Track ingestion status consistently |
| Data Quality | Apply common validation rules |
| Security | Centralize credential and access practices |
| Auditability | Maintain ingestion history and metadata |

---

## 3. Requirements of an Ingestion Framework

Before designing the framework, identify its requirements.

### 3.1 Functional Requirements

The framework should support:

1. Multiple data sources.
2. Different data formats.
3. Full and incremental ingestion.
4. Batch and streaming ingestion where required.
5. Schema validation.
6. Data quality checks.
7. Duplicate handling.
8. Retry and failure handling.
9. Metadata tracking.
10. Configurable ingestion.
11. Logging and monitoring.
12. Secure source connectivity.

### 3.2 Non-Functional Requirements

| Requirement | Meaning |
|---|---|
| Scalability | Handle increasing data volume and source count |
| Reliability | Ingest data consistently |
| Performance | Meet ingestion time requirements |
| Availability | Keep ingestion services accessible |
| Security | Protect data and credentials |
| Maintainability | Support easy updates |
| Extensibility | Support new source types |
| Observability | Provide logs, metrics, and alerts |
| Cost Efficiency | Avoid unnecessary compute and storage usage |

---

## 4. High-Level Ingestion Framework Architecture

A typical framework contains the following components:

```text
                  Data Sources
                /      |       \
               /       |        \
          Databases   APIs      Files
               \       |        /
                \      |       /
                       |
                       v
                Configuration
                   Manager
                       |
                       v
                Ingestion Engine
                       |
             +---------+---------+
             |         |         |
             v         v         v
          Extract   Validate   Metadata
                     Schema    Tracking
             |         |         |
             +---------+---------+
                       |
                       v
                Error Handling
                  and Logging
                       |
                       v
                 Raw Storage
                       |
                       v
                Downstream Jobs
```

### Main Components

| Component | Responsibility |
|---|---|
| Source Connector | Connect to databases, APIs, and files |
| Configuration Manager | Store source-specific settings |
| Ingestion Engine | Execute extraction and loading |
| Schema Validator | Validate incoming structure |
| Data Quality Module | Check data correctness |
| Metadata Manager | Track ingestion progress |
| Error Handler | Manage failures and retries |
| Logging Module | Record execution details |
| Storage Connector | Write data to the target |
| Monitoring Module | Track metrics and alerts |

---

## 5. Data Source Connectors

A connector is a component that communicates with a particular source system.

Different sources require different connection and extraction mechanisms.

### Common Source Types

| Source | Example | Extraction Method |
|---|---|---|
| Relational Database | Azure SQL, PostgreSQL | JDBC, SQL queries |
| REST API | Payment API | HTTP requests |
| CSV Files | Sales reports | File reading |
| JSON Files | Application records | JSON parsing |
| Cloud Storage | ADLS, S3 | Storage SDK or connector |
| Kafka | Event streams | Kafka consumer |
| SaaS Applications | CRM systems | API or managed connector |

### Connector Responsibilities

- Establish connections.
- Authenticate securely.
- Extract data.
- Handle pagination where necessary.
- Manage source-specific errors.
- Return data in a supported representation.
- Close resources safely.

### Example: Database Extraction

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

orders_df = (
    spark.read
    .format("jdbc")
    .option("url", jdbc_url)
    .option("dbtable", "dbo.orders")
    .option("user", username)
    .option("password", password)
    .option("driver", "com.microsoft.sqlserver.jdbc.SQLServerDriver")
    .load()
)
```

Credentials should come from a secure secret-management mechanism, not hardcoded values.

---

## 6. Configuration-Driven Ingestion

One of the most important design principles is configuration-driven ingestion.

Instead of hardcoding source details in every pipeline, store source-specific settings in configuration files or metadata tables.

### Example Configuration

```json
{
  "source_name": "orders",
  "source_type": "sql",
  "source_table": "dbo.orders",
  "target_path": "/bronze/orders",
  "load_type": "incremental",
  "watermark_column": "updated_at",
  "primary_key": "order_id",
  "file_format": "delta",
  "enabled": true
}
```

The framework reads this configuration and determines how to execute ingestion.

### Benefits

- Less duplicated code.
- Easier onboarding of new sources.
- Centralized configuration management.
- Consistent ingestion behavior.
- Easier maintenance.

### Configuration vs Code

| Configuration | Code |
|---|---|
| Defines source table | Implements extraction logic |
| Defines target location | Implements writing logic |
| Defines load type | Implements incremental processing |
| Defines key columns | Implements deduplication |
| Defines enabled status | Executes framework behavior |

**Important:** Configuration should describe what to ingest. Reusable code should define how ingestion works.

---

## 7. Batch and Streaming Ingestion

An ingestion framework may support batch ingestion, streaming ingestion, or both.

### Batch Ingestion

Processes accumulated data at scheduled or triggered intervals.

Example:

```text
Source Database
       |
       v
Scheduled Extraction
       |
       v
Raw Storage
       |
       v
Batch Processing
```

Common use cases:
- Daily sales data.
- Monthly financial reports.
- Customer data synchronization.
- Historical database extraction.

### Streaming Ingestion

Continuously receives or processes incoming events.

Example:

```text
Application Events
        |
        v
   Kafka / Event Hubs
        |
        v
 Streaming Consumer
        |
        v
    Data Storage
```

Common use cases:
- Application events.
- Payment notifications.
- Real-time monitoring.
- Clickstream data.

### Comparison

| Feature | Batch | Streaming |
|---|---|---|
| Processing | Accumulated records | Continuously arriving records |
| Latency | Usually higher | Usually lower |
| Scheduling | Commonly scheduled | Continuously running or triggered |
| Complexity | Often simpler | Requires event and state management |
| Use Case | Daily reports | Live event processing |

A framework should use the ingestion mode that matches the business requirement rather than choosing streaming for every workload.

---

## 8. Full Load and Incremental Load

A reusable ingestion framework should support different loading strategies.

### Full Load

Extracts the entire source dataset.

```sql
SELECT *
FROM orders;
```

Useful for:
- Initial ingestion.
- Small datasets.
- Complete source snapshots.
- Controlled historical rebuilds.

### Incremental Load

Extracts records that are new or changed since the previous successful load.

Example:

```sql
SELECT *
FROM orders
WHERE updated_at > :last_successful_watermark;
```

Useful for:
- Large datasets.
- Frequent ingestion.
- Reducing unnecessary source reads.

### CDC-Based Load

Change Data Capture captures source database changes such as:

- INSERT.
- UPDATE.
- DELETE.

CDC is useful when timestamp-based extraction cannot reliably identify all changes.

### Comparison

| Strategy | Extracted Data | Complexity |
|---|---|---|
| Full Load | Entire dataset | Relatively low |
| Incremental Load | New or changed records | Medium |
| CDC | Captured database changes | Depends on source and CDC implementation |

### Important Watermark Rule

The framework should update its successful watermark only after the target write has succeeded.

Consider this sequence:

```text
Read Source
    |
    v
Write Target
    |
    v
Verify Successful Commit
    |
    v
Update Watermark
```

If the target write fails, the watermark should not advance.

Otherwise, records could be skipped during the next run.

**Late-arriving updates:** Timestamp-based extraction may miss records if source timestamps are delayed or unreliable. Use a lookback window, CDC, or reconciliation when required.

**Deletes:** A simple `updated_at` query does not necessarily detect physical deletes. Use CDC, soft deletes, or periodic reconciliation.

---

## 9. Schema Management and Validation

Different source systems may change their schemas over time.

For example, an orders table initially contains:

```text
order_id
customer_id
amount
```

Later, the source adds:

```text
discount_amount
```

The ingestion framework should detect and handle this change according to defined rules.

### Schema Validation Checks

- Required columns exist.
- Column names are valid.
- Data types are compatible.
- Required fields are present.
- Unexpected columns are handled.
- Schema changes are recorded.

### Possible Schema Change Strategies

| Strategy | Behavior |
|---|---|
| Strict Validation | Reject unexpected schema changes |
| Controlled Evolution | Accept approved changes |
| Quarantine | Isolate incompatible records or files |
| Rescued Data | Capture certain unexpected or incompatible values where supported |

### Example: PySpark Schema Validation

```python
expected_columns = {
    "order_id",
    "customer_id",
    "amount",
    "updated_at"
}

actual_columns = set(orders_df.columns)

missing_columns = expected_columns - actual_columns

if missing_columns:
    raise ValueError(
        f"Missing columns: {missing_columns}"
    )
```

Explicit schema validation is useful because successful file reading does not guarantee that every record contains valid business data.

---

## 10. Data Quality Checks

The ingestion framework should validate data before or during loading, depending on the architecture.

### Common Checks

| Check | Example |
|---|---|
| Completeness | Required order ID is not NULL |
| Uniqueness | Order ID follows expected uniqueness rules |
| Validity | Amount is non-negative |
| Consistency | Customer references are valid |
| Format | Date follows the expected format |
| Freshness | Data arrives within the expected time |
| Volume | Record count is within an expected range |

### Example: SQL Validation

```sql
SELECT COUNT(*) AS invalid_records
FROM orders
WHERE order_id IS NULL
   OR customer_id IS NULL
   OR amount < 0;
```

### Invalid Record Handling

The framework may:

1. Reject the complete batch.
2. Quarantine invalid records.
3. Continue with valid records.
4. Generate an alert.

The correct strategy depends on business criticality and data quality requirements.

---

## 11. Metadata Management

Metadata tracking is essential for reliable ingestion.

The framework should maintain information about each ingestion execution.

### Example Metadata Table

| Column | Purpose |
|---|---|
| pipeline_name | Pipeline identifier |
| source_name | Source dataset |
| target_name | Target dataset |
| run_id | Unique execution identifier |
| start_time | Execution start |
| end_time | Execution end |
| status | Success or failure |
| records_read | Extracted records |
| records_written | Successfully written records |
| records_rejected | Rejected records |
| watermark_value | Last successful extraction position |
| error_message | Failure details |

### Example Metadata Record

```text
pipeline_name: orders_ingestion
source_name: dbo.orders
target_name: bronze.orders
run_id: 20261031_001
status: SUCCESS
records_read: 100000
records_written: 99980
records_rejected: 20
```

### Why Is Metadata Important?

- Tracks pipeline execution.
- Supports auditing.
- Helps identify failures.
- Maintains incremental progress.
- Supports operational reporting.
- Helps reconcile source and target records.

**Important:** A run-level metadata record and a source watermark are related but different concepts. The watermark represents extraction progress; run metadata records what happened during an execution.

---

## 12. Error Handling and Retry Mechanism

Failures can occur during source extraction, validation, transformation, or target writing.

A robust framework should classify failures and handle them appropriately.

### Common Failure Scenarios

- Database connection failure.
- API rate limit.
- Authentication failure.
- Network interruption.
- Invalid input file.
- Schema mismatch.
- Storage permission issue.
- Target write failure.

### Retry Strategy

Retry transient failures, not every failure.

Examples:

| Failure | Suggested Handling |
|---|---|
| Temporary Network Failure | Retry with backoff |
| API Rate Limit | Respect retry instructions and wait |
| Authentication Failure | Stop and fix credentials or permissions |
| Schema Mismatch | Quarantine or fail according to policy |
| Invalid Records | Apply data quality policy |
| Storage Failure | Retry if transient; otherwise alert |

### Exponential Backoff

A common retry approach increases the waiting time after each failed attempt.

```python
import time

max_retries = 3
base_delay = 2

for attempt in range(max_retries):
    try:
        run_ingestion()
        break

    except TemporaryConnectionError:
        if attempt == max_retries - 1:
            raise

        delay = base_delay * (2 ** attempt)
        time.sleep(delay)
```

Example delays:

```text
Attempt 1 fails → Wait 2 seconds
Attempt 2 fails → Wait 4 seconds
Attempt 3 fails → Wait 8 seconds
```

Production implementations should also consider maximum delay, jitter, timeouts, and API-specific retry guidance.

### Idempotency

A retry must not create duplicate or inconsistent target data.

Common techniques:
- Stable business keys.
- MERGE operations.
- Deterministic transformations.
- Controlled partition overwrites.
- Duplicate detection.
- Safe progress tracking.

A retry mechanism alone does not guarantee idempotency.

---

## 13. File-Based Ingestion

Many organizations receive data through files.

Common formats include:

- CSV.
- JSON.
- Parquet.
- Avro.
- ORC.

### Example File Ingestion with PySpark

```python
orders_df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("/landing/orders/")
)
```

For production ingestion, prefer an explicitly defined schema when possible rather than relying entirely on schema inference.

### Writing to Bronze

```python
(
    orders_df.write
    .format("delta")
    .mode("append")
    .save("/lakehouse/bronze/orders")
)
```

### Important File Ingestion Considerations

1. Avoid processing incomplete files.
2. Track processed files.
3. Handle duplicate file delivery.
4. Validate schema and file format.
5. Handle corrupt records.
6. Maintain source file metadata.
7. Avoid overwriting historical data unintentionally.

---

## 14. Auto Loader as an Ingestion Component

In Databricks, Auto Loader provides incremental file discovery and ingestion from cloud storage.

It uses the `cloudFiles` source.

### Example

```python
stream_df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option(
        "cloudFiles.schemaLocation",
        "/metadata/orders_schema"
    )
    .load("/landing/orders/")
)
```

### Write to Delta

```python
query = (
    stream_df.writeStream
    .format("delta")
    .option(
        "checkpointLocation",
        "/checkpoints/orders"
    )
    .outputMode("append")
    .trigger(availableNow=True)
    .start("/lakehouse/bronze/orders")
)

query.awaitTermination()
```

### Important Concepts

| Component | Purpose |
|---|---|
| `cloudFiles` | Auto Loader source |
| `schemaLocation` | Stores schema metadata |
| `checkpointLocation` | Tracks streaming progress and state |
| `availableNow` | Processes available data across one or more micro-batches, then stops |
| `awaitTermination()` | Waits for the query to finish |

Auto Loader handles file discovery and ingestion. It does not automatically perform business-key deduplication or CDC merging.

---

## 15. Ingestion Framework vs Orchestration Framework

These two concepts are related but have different responsibilities.

### Ingestion Framework

Focuses on moving data from sources into target storage.

Responsibilities:
- Source connectivity.
- Extraction.
- Loading.
- Schema handling.
- Ingestion metadata.
- Source-specific error handling.

### Orchestration Framework

Coordinates execution of workflows and their dependencies.

Responsibilities:
- Scheduling.
- Task dependencies.
- Workflow execution.
- Retries.
- Notifications.
- Pipeline monitoring.

### Comparison

| Ingestion Framework | Orchestration Framework |
|---|---|
| Moves data | Coordinates activities |
| Connects to sources | Schedules pipelines |
| Extracts and loads records | Manages dependencies |
| Tracks ingestion progress | Tracks workflow execution |
| Example: Reusable Python ingestion framework | Example: Airflow or ADF |

An orchestration tool can invoke an ingestion framework as one of its tasks.

---

## 16. Security and Access Management

An ingestion framework often connects to sensitive databases and APIs.

Security must be built into the framework.

### Best Practices

1. Never hardcode passwords or API keys.
2. Use managed identities or service principals where appropriate.
3. Store secrets in a secrets manager.
4. Apply least-privilege permissions.
5. Encrypt data in transit.
6. Use secure network connectivity.
7. Audit source access.
8. Avoid exposing sensitive values in logs.
9. Separate development and production credentials.
10. Rotate credentials according to policy.

### Azure Examples

| Requirement | Azure Service |
|---|---|
| Identity | Microsoft Entra ID |
| Secret Management | Azure Key Vault |
| Storage | ADLS Gen2 |
| Orchestration | Azure Data Factory |
| Monitoring | Azure Monitor |

Authentication establishes identity; authorization determines what that identity can access.

---

## 17. Monitoring and Observability

Monitoring helps ensure that ingestion jobs are operating correctly.

### Important Metrics

| Metric | Purpose |
|---|---|
| Records Read | Number of extracted records |
| Records Written | Number of successfully loaded records |
| Records Rejected | Number of invalid records |
| Execution Duration | Time taken by ingestion |
| Failure Count | Number of failed runs |
| Data Freshness | Age of the latest available data |
| Source Availability | Whether the source is reachable |
| Data Volume | Amount of ingested data |
| Retry Count | Number of retry attempts |
| Cost | Resource consumption |

### Example Alerts

- Pipeline failed.
- No data received during the expected period.
- Record count dropped unexpectedly.
- Schema changed unexpectedly.
- Data freshness SLA breached.
- Retry limit exceeded.

A pipeline can report success while ingesting incomplete or incorrect data. Therefore, monitor data quality and freshness in addition to execution status.

---

## 18. Practical Design: Reusable Ingestion Framework

### Business Requirement

An organization needs to ingest 100 datasets from different databases and APIs into ADLS Gen2.

The framework should:
- Support full and incremental loading.
- Use configuration-driven ingestion.
- Validate schemas.
- Track metadata.
- Handle failures.
- Support retries.
- Avoid duplicate records.
- Provide monitoring.

### Proposed Architecture

```text
              Source Systems
             /      |       \
            /       |        \
        Azure SQL  REST API  CSV Files
            \       |        /
             \      |       /
                    |
                    v
             Configuration DB
                    |
                    v
             Ingestion Framework
                    |
         +----------+----------+
         |          |          |
         v          v          v
      Extract    Validate   Metadata
         |          |          |
         +----------+----------+
                    |
                    v
              Error Handling
                    |
                    v
                ADLS Gen2
                    |
                    v
               Bronze Layer
                    |
                    v
             Databricks / Spark
                    |
                    v
              Silver / Gold
                    |
                    v
                Analytics

          Azure Data Factory
          Coordinates execution
```

### Execution Flow

1. ADF triggers the ingestion workflow.
2. The framework reads source configuration.
3. It selects the appropriate source connector.
4. It determines full or incremental load.
5. It extracts source data.
6. It validates the incoming schema.
7. It records ingestion metadata.
8. It writes data to Bronze storage.
9. It verifies the target write.
10. It updates the successful watermark.
11. It reports execution status.
12. Downstream processing begins after successful ingestion.

### Why This Design?

- Configuration reduces duplicated code.
- Reusable connectors simplify onboarding.
- Metadata supports audit and recovery.
- Validation improves data quality.
- Idempotent writes support safe retries.
- Orchestration coordinates execution.
- Monitoring improves operational visibility.

---

## 19. Common Interview Mistakes

1. Confusing ingestion with transformation.
2. Confusing ingestion framework with orchestration.
3. Assuming incremental loading automatically handles deletes.
4. Updating watermarks before target writes succeed.
5. Assuming retries prevent duplicate records.
6. Ignoring late-arriving data.
7. Treating schema inference as schema validation.
8. Assuming Auto Loader performs business-level deduplication.
9. Storing credentials directly in configuration files.
10. Monitoring only pipeline success or failure.
11. Ignoring source API rate limits.
12. Failing to track rejected records.
13. Using one generic extraction method for every source type.
14. Ignoring concurrent ingestion runs and overlapping execution.

---

## 20. Interview Questions and Answers

### Q1. What is a data ingestion framework?

A reusable system that extracts data from multiple sources and loads it into a centralized storage or processing platform.

### Q2. Why do we need an ingestion framework?

It standardizes ingestion logic, reduces duplicated code, improves maintainability, and provides common error handling and monitoring.

### Q3. What are the main components?

Source connectors, configuration manager, ingestion engine, schema validator, metadata manager, error handler, storage connector, and monitoring.

### Q4. What is configuration-driven ingestion?

A design where source-specific settings are stored as configuration rather than hardcoded into every pipeline.

### Q5. What is the difference between full and incremental loading?

Full loading extracts the complete dataset. Incremental loading extracts only new or changed records.

### Q6. What is CDC?

Change Data Capture identifies source database inserts, updates, and deletes so downstream systems can process changes.

### Q7. What is a watermark?

A stored progress value, often a timestamp or identifier, that helps identify which source records to extract next.

### Q8. When should the watermark be updated?

Only after the target write has successfully completed and the framework has verified the required commit.

### Q9. How do you handle schema changes?

Use schema validation, controlled schema evolution, and quarantine or failure handling for incompatible changes.

### Q10. How do you prevent duplicate ingestion?

Use stable business keys, deterministic deduplication, file tracking where appropriate, and idempotent target writes.

### Q11. What is idempotency?

An operation is idempotent when repeating it has the same intended final effect under defined conditions.

### Q12. How do you handle API rate limits?

Respect the API's rate-limit instructions, use controlled retries with backoff, and avoid excessive concurrent requests.

### Q13. What is the difference between ingestion and orchestration?

Ingestion extracts and loads data. Orchestration schedules and coordinates the activities that make up a workflow.

### Q14. What is the role of metadata?

It tracks source, target, run status, record counts, timestamps, and extraction progress.

### Q15. What is Auto Loader?

A Databricks capability for incrementally discovering and ingesting files from cloud storage using Structured Streaming.

### Q16. Does Auto Loader automatically remove duplicate business records?

No. File discovery and business-record deduplication are separate responsibilities.

### Q17. How do you monitor an ingestion framework?

Track execution status, duration, record counts, rejected records, freshness, source availability, retry count, and failures.

### Q18. How do you design a reusable ingestion framework for 100 tables?

Use metadata-driven configurations, reusable connectors, common validation and logging modules, load strategies, and centralized orchestration.

### Q19. How do you recover from a failed ingestion?

Identify the failed stage, preserve the correct watermark, fix the cause, and rerun using idempotent writes or controlled recovery logic.

### Q20. How would you build an ingestion framework using Azure?

Use ADF for orchestration, reusable Python or Databricks ingestion components, ADLS Gen2 for raw storage, Key Vault for secrets, and Azure Monitor for operational monitoring.

---

## 21. Quick Revision

| Concept | Key Point |
|---|---|
| Ingestion Framework | Reusable system for collecting and loading data |
| Connector | Connects to a particular source |
| Configuration-Driven | Uses metadata instead of hardcoded source details |
| Full Load | Extracts the entire dataset |
| Incremental Load | Extracts new or changed data |
| CDC | Captures inserts, updates, and deletes |
| Watermark | Tracks incremental extraction progress |
| Schema Validation | Checks incoming data structure |
| Data Quality | Checks record correctness |
| Metadata | Tracks ingestion execution and progress |
| Idempotency | Makes safe reruns possible |
| Retry | Reattempts suitable failures |
| Auto Loader | Incremental cloud-file ingestion |
| Orchestration | Coordinates pipeline execution |
| Monitoring | Tracks pipeline and data health |

### Five Important Interview Points

1. Build reusable connectors instead of duplicating extraction code.
2. Use configuration-driven ingestion to onboard new sources easily.
3. Update watermarks only after successful target writes.
4. Design retries and writes together to prevent duplicate or inconsistent results.
5. Monitor both pipeline execution and data quality.

---

## 22. Self-Assessment

Try answering without looking at the notes.

1. What is a data ingestion framework?
2. Why do organizations build reusable ingestion frameworks?
3. Draw a high-level ingestion architecture.
4. What are the main framework components?
5. What is configuration-driven ingestion?
6. Explain full load, incremental load, and CDC.
7. What is a watermark?
8. Why should a watermark be updated after a successful target write?
9. How do you handle late-arriving updates?
10. How do you handle physical deletes?
11. How do you validate an incoming schema?
12. What is the purpose of metadata tracking?
13. How do you handle transient failures?
14. What is exponential backoff?
15. What is idempotency?
16. How do you prevent duplicate file ingestion?
17. What is the difference between ingestion and orchestration?
18. What is the role of Auto Loader?
19. How do you monitor ingestion quality?
20. Design a reusable ingestion framework for 100 source tables.

**Interview readiness:** You should be able to draw the architecture, explain configuration-driven ingestion, compare full/incremental/CDC approaches, describe watermark management, and explain how you handle schema changes, retries, idempotency, metadata, security, and monitoring.

