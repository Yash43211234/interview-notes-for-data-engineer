# Data Pipeline Lifecycle

## 1. What Is a Data Pipeline?

A **data pipeline** is a sequence of steps that moves data from one or more sources to a destination while applying the required processing and transformations.

The purpose is to make data available in a reliable, organized, and usable form.

### Simple Example

An e-commerce company stores order information in a MySQL database.

The company wants to analyze:
- Total sales.
- Number of orders.
- Customer purchasing behavior.
- Monthly revenue.

A data pipeline collects order data, cleans it, transforms it, and loads it into a data warehouse.

```text
Source Database
      |
      v
Data Extraction
      |
      v
Raw Data Storage
      |
      v
Data Transformation
      |
      v
Data Warehouse
      |
      v
Business Reports
```

**Simple way to remember:**

Data Pipeline = Extract + Process + Store + Deliver Data.

## 2. Why Do We Need Data Pipelines?

Organizations generate data from multiple systems:

- Databases.
- Applications.
- APIs.
- Log files.
- IoT devices.
- Cloud storage.
- Transaction systems.

This data is often distributed, inconsistent, or stored in different formats.

A data pipeline brings the data together and prepares it for downstream use.

### Main Benefits

1. Automates data movement.
2. Reduces manual processing.
3. Improves data consistency.
4. Supports batch and streaming workloads.
5. Makes data available for analytics.
6. Supports data quality checks.
7. Improves monitoring and reliability.

## 3. Data Pipeline Lifecycle

A data pipeline generally passes through several stages.

```text
       1. DATA SOURCES
              |
              v
       2. DATA INGESTION
              |
              v
       3. RAW DATA STORAGE
              |
              v
       4. DATA VALIDATION
              |
              v
       5. DATA TRANSFORMATION
              |
              v
       6. DATA LOADING
              |
              v
       7. DATA SERVING
              |
              v
       8. MONITORING
```

These stages describe a common pipeline design. In real projects, some stages may be combined, repeated, or executed in a different order.

## 4. Stage 1: Data Sources

The first stage identifies where the data comes from.

Common data sources include:

| Source | Example |
|---|---|
| Relational Database | MySQL, PostgreSQL, SQL Server |
| NoSQL Database | MongoDB, Cassandra |
| APIs | REST APIs |
| Files | CSV, JSON, Parquet |
| Event Platforms | Apache Kafka |
| Cloud Storage | ADLS, Amazon S3 |
| Applications | E-commerce, CRM, ERP |

### Example

An e-commerce pipeline may collect data from:

- Orders database.
- Customer database.
- Payment API.
- Website activity logs.

**Important:** Before building a pipeline, understand the source schema, data format, update frequency, volume, and extraction method.

## 5. Stage 2: Data Ingestion

Data ingestion means collecting data from source systems and bringing it into the data platform.

There are two common ingestion approaches.

### 5.1 Batch Ingestion

Data is collected periodically and loaded together.

Example:

```text
Source Database
      |
      v
Extract Orders
      |
      v
Daily Batch
      |
      v
Cloud Storage
```

Common use cases:
- Daily database extraction.
- Historical file ingestion.
- Scheduled API extraction.

### 5.2 Streaming Ingestion

Data is ingested continuously as new events arrive.

Example:

```text
Application Events
        |
        v
      Kafka
        |
        v
  Data Ingestion
        |
        v
    Data Lake
```

Common use cases:
- Payment events.
- IoT sensor data.
- Website activity.

### Batch vs Streaming Ingestion

| Feature | Batch | Streaming |
|---|---|---|
| Data arrival | Periodically collected | Continuously arriving |
| Latency | Usually higher | Usually lower |
| Execution | Scheduled or triggered | Continuous or frequent |
| Example | Daily CSV files | Kafka events |

**Interview point:** Ingestion is about bringing data into the platform. Transformation is about changing the data into the required structure or format.

## 6. Stage 3: Raw Data Storage

After ingestion, data is commonly stored in a raw or landing layer.

The purpose is to preserve the incoming data before applying major transformations.

### Example

Suppose the source provides:

`orders.csv`

```text
order_id,customer_id,amount
101,501,500
102,502,700
103,503,900
```

The pipeline stores the received data in a raw storage location.

```text
Data Lake
    |
    v
Bronze / Raw
    |
    +-- orders/
    |      |
    |      +-- orders_20261025.csv
    |
    +-- customers/
           |
           +-- customers_20261025.csv
```

### Why Is Raw Storage Important?

- Preserves source data.
- Supports reprocessing.
- Helps investigate pipeline failures.
- Provides an audit-friendly ingestion layer.
- Separates source ingestion from downstream transformations.

**Important:** Raw does not always mean completely untouched. Some ingestion metadata, such as ingestion timestamp or source filename, may be added.

## 7. Stage 4: Data Validation and Quality Checks

Data validation checks whether incoming data meets the expected requirements.

Common checks include:

1. Null-value checks.
2. Duplicate checks.
3. Data type validation.
4. Schema validation.
5. Range validation.
6. Referential integrity checks.
7. Business-rule validation.

### Example

Incoming data:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | 501 | 500 |
| 102 | NULL | 700 |
| 103 | 503 | -100 |
| 101 | 501 | 500 |

Possible quality issues:

- Row 2: Missing customer ID.
- Row 3: Negative amount, if prohibited by the business rule.
- Row 4: Duplicate order.

### Example PySpark Validation

```python
from pyspark.sql.functions import col

valid_df = df.filter(
    col("order_id").isNotNull() &
    col("customer_id").isNotNull() &
    (col("amount") >= 0)
)
```

Invalid records can be:
- Rejected.
- Quarantined.
- Logged for investigation.
- Corrected when a reliable correction rule exists.

**Important:** Do not silently remove bad records without defining how they will be tracked and handled.

## 8. Stage 5: Data Transformation

Data transformation converts raw data into a clean, consistent, and useful structure.

Common transformations include:

- Filtering unwanted records.
- Removing duplicates.
- Handling NULL values.
- Changing data types.
- Joining datasets.
- Aggregating data.
- Standardizing column names.
- Applying business rules.

### Example

Raw orders:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | 501 | 500 |
| 102 | 502 | 700 |
| 103 | 503 | 900 |

Suppose the business wants total sales per customer.

Transformation:

```python
from pyspark.sql.functions import sum

customer_sales = df.groupBy("customer_id") \
    .agg(sum("amount").alias("total_sales"))
```

Result:

| customer_id | total_sales |
|---|---:|
| 501 | 500 |
| 502 | 700 |
| 503 | 900 |

### ETL vs ELT

**ETL: Extract, Transform, Load**

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
```

Data is transformed before loading into the target system.

**ELT: Extract, Load, Transform**

```text
Source
  |
  v
Extract
  |
  v
Load
  |
  v
Transform
```

Data is loaded first, and transformations happen inside the target platform.

| Feature | ETL | ELT |
|---|---|---|
| Transformation | Before loading | After loading |
| Raw data storage | Not necessarily required | Commonly retained |
| Processing location | ETL processing environment | Target platform |
| Common use | Traditional integration | Modern cloud data platforms |

**Interview point:** ETL and ELT describe the order of operations. Both are used in modern data engineering.

## 9. Stage 6: Data Loading

Data loading means writing processed data into the target storage or destination.

Common destinations include:

- Data warehouses.
- Data lakes.
- Lakehouse tables.
- Relational databases.
- Analytical databases.

### Example

After transformation, customer sales data is loaded into a Gold table.

```text
Transformed Data
       |
       v
    Gold Table
       |
       v
  BI / Analytics
```

### Common Loading Strategies

**1. Full Load**

Loads the complete dataset.

Useful when:
- Dataset is small.
- Full refresh is acceptable.
- Historical data must be rebuilt.

**2. Incremental Load**

Loads only new or changed records.

Useful when:
- Dataset is large.
- Only recent changes need processing.
- Repeated full loads are expensive.

**3. Upsert / MERGE**

Updates existing records and inserts new records.

Useful when:
- Source data changes.
- CDC is used.
- Target tables need to maintain the latest record state.

**Important:** A reliable incremental pipeline must handle retries, duplicate records, late-arriving updates, and failures between writing data and updating its progress marker.

## 10. Stage 7: Data Serving and Consumption

After data is loaded, downstream users and applications consume it.

Common consumers include:

- Business analysts.
- Data analysts.
- Data scientists.
- Machine learning applications.
- BI dashboards.
- Reporting systems.

### Example

```text
Gold Table
    |
    +------> Power BI Dashboard
    |
    +------> SQL Analytics
    |
    +------> Machine Learning
    |
    +------> Business Reports
```

The serving layer should provide data in a structure suitable for its consumers.

For example, a Gold table may contain monthly sales by product and region rather than raw individual transactions.

## 11. Stage 8: Monitoring and Maintenance

A data pipeline must be monitored after deployment.

A pipeline that runs successfully once is not necessarily reliable.

Monitoring helps detect failures, performance issues, and data-quality problems.

### Important Monitoring Metrics

| Metric | Purpose |
|---|---|
| Pipeline Status | Checks success or failure |
| Execution Time | Tracks processing duration |
| Records Read | Number of incoming records |
| Records Written | Number of output records |
| Failed Records | Tracks invalid or rejected data |
| Data Freshness | Checks how recently data was updated |
| Processing Latency | Measures processing delay |
| Cost | Tracks infrastructure and processing expense |

### Common Problems

- Source connection failure.
- Missing input files.
- Schema changes.
- Duplicate records.
- Data-quality failures.
- Slow Spark jobs.
- Insufficient memory.
- Permission errors.
- Target storage failure.

### Monitoring Tools

- Databricks Jobs.
- Azure Data Factory monitoring.
- Apache Airflow.
- Spark UI.
- Cloud monitoring services.
- Logging and alerting platforms.

**Important:** Monitoring should cover both pipeline execution and the quality/freshness of the resulting data.

## 12. Orchestration

Orchestration means coordinating and scheduling pipeline tasks.

A data pipeline may contain multiple dependent jobs.

Example:

```text
       Ingest Orders
             |
             v
       Validate Orders
             |
             v
      Transform Orders
             |
             v
       Load Gold Table
             |
             v
       Refresh Report
```

The orchestration system controls:

- Task execution order.
- Scheduling.
- Dependencies.
- Retries.
- Failure handling.
- Notifications.
- Pipeline status.

### Common Orchestration Tools

| Tool | Purpose |
|---|---|
| Apache Airflow | Workflow orchestration |
| Azure Data Factory | Data integration and orchestration |
| Databricks Workflows | Job and task orchestration |
| Prefect | Workflow orchestration |
| Dagster | Data orchestration |

**Important:** Orchestration coordinates tasks; it is not necessarily the engine performing the data transformations.

## 13. Reliability: Retries, Idempotency, and Checkpointing

A production data pipeline must handle failures safely.

### 13.1 Retries

Retries automatically attempt a failed task again.

Example:

```text
Pipeline Task
      |
      v
   Failure
      |
      v
    Retry
      |
      +----> Success
      |
      +----> Failure after retry limit
```

Retries are useful for temporary failures such as network interruptions.

However, blindly retrying a non-idempotent write can create duplicate data.

### 13.2 Idempotency

An operation is idempotent when repeating the same operation produces the same final result as executing it once, under the defined conditions.

Example:

A pipeline processes an order with ID 101.

If the pipeline runs again, it should update or recognize the existing order instead of inserting another duplicate.

Common techniques:
- MERGE using business keys.
- Deduplication.
- Unique constraints where supported.
- Transaction identifiers.
- Idempotent API or sink operations.

### 13.3 Checkpointing

Checkpointing stores progress and, for stateful processing, relevant recovery information.

In streaming pipelines, checkpoints help a query recover after failure.

**Important:** Checkpointing, idempotency, and retries solve different problems. A checkpoint alone does not make arbitrary external writes exactly-once.

## 14. Data Pipeline Design Using Medallion Architecture

The medallion architecture organizes data into three logical layers.

### Bronze Layer

Stores ingested raw or lightly processed data.

Purpose:
- Preserve source data.
- Support reprocessing.
- Track ingestion.

### Silver Layer

Stores cleaned, validated, and standardized data.

Purpose:
- Remove or handle duplicates.
- Validate schemas.
- Apply data-quality rules.
- Standardize data.

### Gold Layer

Stores business-ready data.

Purpose:
- Aggregations.
- Reporting.
- Analytics.
- Business metrics.

### Example

```text
             SOURCE
                |
                v
          BRONZE LAYER
           Raw Orders
                |
                v
          SILVER LAYER
       Cleaned Orders
                |
                v
           GOLD LAYER
      Customer Sales Summary
                |
                v
        Power BI / Analytics
```

All three layers can use Delta tables in Databricks. They are logical data layers, not necessarily separate physical storage systems.

## 15. Complete Real-World Example: E-Commerce Pipeline

Suppose an e-commerce company wants to build a daily sales analytics pipeline.

### Step 1: Sources

- Orders database.
- Customer database.
- Product database.

### Step 2: Ingestion

Use Azure Data Factory or another ingestion mechanism to extract source data.

### Step 3: Bronze

Store incoming data in ADLS Gen2 or Bronze Delta tables.

### Step 4: Validation

Check:
- Order ID is not NULL.
- Customer ID is valid.
- Amount is within acceptable business rules.
- Duplicate records are handled.

### Step 5: Silver

Use PySpark to:
- Clean data.
- Standardize data types.
- Deduplicate orders.
- Join customer and product information.

### Step 6: Gold

Create business-level tables:

- Daily sales.
- Monthly revenue.
- Customer-wise sales.
- Product-wise sales.

### Step 7: Consumption

Connect Power BI to the Gold tables.

### Step 8: Monitoring

Monitor:
- Job success.
- Record counts.
- Failed records.
- Data freshness.
- Processing time.

### Architecture

```text
 Orders DB    Customer DB    Product DB
     |             |              |
     +-------------+--------------+
                   |
                   v
              INGESTION
                   |
                   v
             ADLS / BRONZE
                   |
                   v
           VALIDATION + CLEANING
                   |
                   v
              SILVER DELTA
                   |
                   v
             PYSPARK / SQL
                   |
                   v
               GOLD DELTA
                   |
                   v
                POWER BI
                   |
                   v
          MONITORING + ALERTS
```

## 16. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| A data pipeline only moves data | It can also validate, transform, store, and serve data |
| Ingestion and transformation are identical | Ingestion collects data; transformation changes it |
| ETL and ELT are identical | Their transformation and loading order differs |
| Orchestration performs every transformation | It coordinates tasks; processing engines perform the work |
| Bronze data must always be completely raw | Ingestion metadata or light processing may be added |
| Incremental loading means loading only new rows | It may also need to capture updates and deletes |
| Retries guarantee safe execution | Retries need idempotent or otherwise safe operations |
| Checkpointing guarantees exactly-once for every sink | End-to-end guarantees depend on source and sink behavior |
| A successful job guarantees good data | Data-quality and freshness checks are also required |
| Monitoring only means checking job status | It should include data quality, freshness, performance, and cost |

## 17. Interview Questions and Answers

**Q1. What is a data pipeline?**

A data pipeline is a sequence of processes that extracts data from sources, processes it, and delivers it to a destination.

**Q2. What are the main stages of a data pipeline?**

Source identification, ingestion, raw storage, validation, transformation, loading, serving, monitoring, and maintenance.

**Q3. What is data ingestion?**

Data ingestion is the process of collecting data from source systems and bringing it into a data platform.

**Q4. What is the difference between ingestion and transformation?**

Ingestion moves data into the platform. Transformation changes its structure, format, or values according to business requirements.

**Q5. What is ETL?**

ETL means Extract, Transform, Load. Data is transformed before loading into the target system.

**Q6. What is ELT?**

ELT means Extract, Load, Transform. Data is loaded first and transformed inside the target platform.

**Q7. What is incremental loading?**

Incremental loading processes only new or changed records instead of reloading the entire dataset.

**Q8. What is a full load?**

A full load processes the complete dataset, usually by replacing or rebuilding the target data.

**Q9. What is orchestration?**

Orchestration coordinates pipeline tasks, dependencies, schedules, retries, and failure handling.

**Q10. What is idempotency?**

Idempotency means repeating an operation produces the same final result as executing it once, under the defined conditions.

**Q11. Why is monitoring important?**

It helps identify pipeline failures, data-quality issues, slow execution, stale data, and unexpected costs.

**Q12. What is the purpose of a Bronze layer?**

It stores incoming raw or lightly processed data and supports reprocessing and traceability.

**Q13. What is the purpose of a Silver layer?**

It contains cleaned, validated, deduplicated, and standardized data.

**Q14. What is the purpose of a Gold layer?**

It contains business-ready data for analytics, reporting, and downstream applications.

**Q15. How do you handle pipeline failures?**

I would investigate logs and failed tasks, identify the root cause, correct the issue, and rerun the pipeline safely using retries, checkpoints, and idempotent write logic where appropriate.

**Q16. How do you make a data pipeline reliable?**

Use data-quality checks, safe retries, idempotent writes, checkpoints where appropriate, monitoring, alerting, and proper failure handling.

**Q17. What happens if the source schema changes?**

The pipeline should detect the change, validate whether it is compatible, and apply a controlled schema-evolution or rejection strategy.

**Q18. What is the difference between a data pipeline and a workflow?**

A data pipeline focuses on moving and processing data. A workflow is a broader sequence of coordinated tasks, which may include data pipelines.

## 18. Quick Revision

| Concept | Remember |
|---|---|
| Data Pipeline | Moves and processes data |
| Source | Origin of data |
| Ingestion | Brings data into the platform |
| Raw Storage | Preserves incoming data |
| Validation | Checks data correctness |
| Transformation | Cleans and changes data |
| Loading | Writes data to the destination |
| Serving | Makes data available to consumers |
| Monitoring | Tracks health and performance |
| Orchestration | Coordinates pipeline tasks |
| ETL | Transform before loading |
| ELT | Load before transforming |
| Full Load | Processes complete dataset |
| Incremental Load | Processes new or changed data |
| Idempotency | Safe repeated execution |
| Checkpointing | Supports recovery and progress tracking |
| Bronze | Raw or lightly processed |
| Silver | Cleaned and standardized |
| Gold | Business-ready |

### One-Minute Interview Answer

A data pipeline is a sequence of steps used to collect data from different sources, process it, and deliver it to a destination.

A typical pipeline starts with source systems such as databases, APIs, and files. Data is ingested into a raw storage layer, validated, transformed using tools such as PySpark or SQL, and loaded into a data warehouse or lakehouse.

In a Databricks environment, I can organize the pipeline using the Bronze, Silver, and Gold layers. Bronze stores incoming data, Silver contains cleaned and validated data, and Gold contains business-ready data for reporting.

For production reliability, I would also implement orchestration, idempotent writes, retries, data-quality checks, monitoring, and alerting.

## 19. Self-Assessment

- [ ] Define a data pipeline.
- [ ] Explain why data pipelines are required.
- [ ] Draw the complete pipeline lifecycle.
- [ ] Explain common data sources.
- [ ] Differentiate batch and streaming ingestion.
- [ ] Explain raw data storage.
- [ ] Explain data-quality checks.
- [ ] Explain data transformation.
- [ ] Differentiate ETL and ELT.
- [ ] Explain full and incremental loading.
- [ ] Explain data serving.
- [ ] Explain monitoring metrics.
- [ ] Explain orchestration.
- [ ] Explain retries and idempotency.
- [ ] Explain checkpointing.
- [ ] Explain Bronze, Silver, and Gold.
- [ ] Design an end-to-end e-commerce pipeline.
- [ ] Explain how to handle pipeline failures.

**Interview readiness:** You should be able to draw an end-to-end data pipeline, explain the responsibility of each stage, design a practical ETL/ELT workflow, and discuss how you would make a production pipeline reliable and maintainable.