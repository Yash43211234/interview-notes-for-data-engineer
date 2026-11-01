# Large-Scale Reporting Platform — System Design

## 1. What Is a Reporting Platform?

A **reporting platform** collects data from multiple sources, transforms it into reliable business metrics, and makes those metrics available through dashboards, scheduled reports, and analytical queries.

Examples:
- Sales and revenue dashboards
- Customer activity reports
- Financial and operational reports
- Daily business performance summaries
- Executive dashboards showing KPIs

A large-scale reporting platform must support large data volumes, many concurrent users, reliable calculations, fast queries, and fresh data.

**Main objective:** Deliver accurate, consistent, and timely reports without overloading source systems or making users wait too long.

---

## 2. Requirements

### Functional requirements

The platform should support:

1. Ingesting data from databases, files, APIs, and event streams.
2. Cleaning and transforming source data.
3. Calculating business metrics and KPIs.
4. Supporting interactive dashboards and analytical queries.
5. Generating scheduled reports.
6. Filtering reports by date, region, product, customer, or other dimensions.
7. Providing historical comparisons and trend analysis.
8. Controlling access to sensitive reports and data.
9. Supporting corrections, backfills, and historical recalculations.

### Non-functional requirements

| Requirement | Meaning |
|---|---|
| Performance | Reports return within an acceptable time |
| Scalability | The system handles growing data and user traffic |
| Freshness | Reports reflect data within the required time window |
| Reliability | Pipelines and reports recover from failures |
| Accuracy | Metrics follow approved business definitions |
| Availability | Users can access reports when needed |
| Security | Users only access authorized information |
| Cost efficiency | Storage and compute costs remain controlled |

### Clarify these requirements during an interview

Before selecting technologies, ask:

- How much data arrives each day?
- How frequently must reports update?
- How many concurrent users are expected?
- What is the acceptable dashboard response time?
- Are reports primarily historical, near-real-time, or both?
- Do users need ad hoc queries or only predefined dashboards?
- What are the data retention and compliance requirements?

---

## 3. High-Level Architecture

```text
Source Systems
  ├── OLTP Databases
  ├── APIs and Files
  └── Event Streams
          |
          v
   Ingestion Layer
   ├── Batch ingestion
   ├── Incremental loads / CDC
   └── Streaming ingestion
          |
          v
   Raw Data Storage
          |
          v
   Data Processing
   ├── Validation
   ├── Cleaning and deduplication
   ├── Business transformations
   └── Data reconciliation
          |
          v
   Curated Data / Lakehouse
   ├── Detailed facts and dimensions
   └── Aggregated reporting tables
          |
          v
   Serving Layer
   ├── SQL Warehouse
   ├── Semantic / Metrics Layer
   └── Optional Cache
          |
          v
   BI and Reporting
   ├── Interactive dashboards
   ├── Scheduled reports
   └── Analytical queries
```

Cross-cutting capabilities include orchestration, monitoring, access control, data lineage, data quality, and cost management.

**Important distinction:** The processing engine transforms data. The orchestration system schedules and coordinates those transformations. The BI layer presents results to users.

---

## 4. Data Ingestion

The ingestion strategy depends on the source, data volume, and freshness requirement.

### Batch ingestion

Data is loaded at scheduled intervals, such as hourly or daily.

Suitable for:
- Daily financial reports
- Historical sales summaries
- Overnight data warehouse loads
- Large periodic file deliveries

Advantages:
- Straightforward recovery and reconciliation
- Efficient for large scheduled workloads
- Often easier to operate than continuous processing

Limitation: Reports may remain stale until the next successful load.

### Incremental ingestion

Instead of reloading all historical data, ingest only new or changed records.

Common approaches:
- Increasing IDs
- Reliable update timestamps
- Change Data Capture (CDC)
- Source-provided change files

For a timestamp-based load, maintain a checkpoint or watermark representing the last successfully processed source position.

A safe sequence is:

1. Read new or changed records.
2. Validate and transform them.
3. Write them to the target using retry-safe logic.
4. Confirm the target write succeeded.
5. Advance the ingestion checkpoint.

If the target write succeeds but checkpoint advancement fails, the same records may be read again. Idempotent writes or deduplication prevent duplicate business results.

Timestamp extraction can miss late updates or deletes. CDC, overlapping extraction windows, and periodic reconciliation can help address these cases.

### Streaming ingestion

Use streaming when the business needs data freshness measured in seconds or minutes.

Suitable for:
- Live order monitoring
- Operational dashboards
- Fraud or anomaly monitoring
- Near-real-time customer activity

Streaming adds complexity around late events, duplicate delivery, ordering, state management, and recovery.

**Interview point:** Do not use streaming simply because the platform is large. Choose it when the business freshness requirement justifies the additional operational complexity.

---

## 5. Storage and Data Organization

A lakehouse or data warehouse can provide the foundation for the reporting platform.

### Medallion-style organization

| Layer | Purpose | Example |
|---|---|---|
| Bronze | Preserve ingested source data | Raw order records |
| Silver | Clean, validate, and standardize | Deduplicated orders |
| Gold | Serve business-ready data | Daily revenue by region |

Bronze supports investigation and reprocessing. Silver provides consistent detailed records. Gold provides reporting-friendly models and aggregates.

These are logical responsibilities; they do not necessarily require three separate storage accounts or physical systems.

### Storage format

Columnar formats such as Parquet are commonly used for analytical data because they support column pruning and compression.

Delta Lake adds transaction-log-based table management and features such as ACID transactions, schema enforcement, and version history on supported platforms.

Do not confuse a file format with a table framework:

- **Parquet:** Columnar file format.
- **Delta Lake:** Table framework that uses Parquet data files and a transaction log.

### Partitioning

Partitioning organizes data to help queries skip irrelevant data.

For reporting workloads, useful partition columns may include a date or a low-to-moderate-cardinality business attribute when query patterns justify it.

Avoid partitioning on highly unique columns such as transaction IDs. Excessive partitioning can create many small files and increase metadata overhead.

Partition design should be based on actual query patterns and measured workload performance.

---

## 6. Data Modeling for Reporting

A reporting platform should model data around business processes and reporting questions.

### Fact and dimension tables

A **fact table** stores business events or measurements.

Examples:
- Orders
- Payments
- Sales
- Website visits

A **dimension table** provides descriptive context.

Examples:
- Customer
- Product
- Region
- Date

Example model:

```text
fact_sales
----------
sales_id
order_id
customer_key
product_key
date_key
quantity
gross_amount
discount_amount
net_amount

dim_customer
------------
customer_key
customer_id
customer_name
region
segment

dim_product
-----------
product_key
product_id
product_name
category

dim_date
--------
date_key
calendar_date
month
quarter
year
```

### Define the grain first

The **grain** describes exactly what one row represents.

For example:

> One row in `fact_sales` represents one product line within one order.

The grain determines which measurements can be safely aggregated.

If the fact table contains one row per order line, summing `quantity` and `net_amount` may be valid under the agreed business definitions. But summing an order-level total repeated on every line will overstate revenue.

### Slowly changing dimensions

If customer attributes change over time, decide whether reports should show the current attribute or the historical attribute.

- **SCD Type 1:** Overwrite the previous value.
- **SCD Type 2:** Create a new version to preserve history.

For historical reporting, facts may need to reference the correct Type 2 dimension version at the time of the event.

### Separate facts with different grains

Orders, payments, refunds, and order lines can have different grains. Joining them directly can multiply rows and inflate totals.

A safer approach is to aggregate each dataset to a compatible grain before combining the results.

---

## 7. Business Metrics and Semantic Layer

Different teams may calculate the same KPI differently unless definitions are centralized.

A **semantic or metrics layer** defines reusable business measures, dimensions, relationships, and calculation rules.

Example metric definitions:

| Metric | Example definition |
|---|---|
| Order count | Number of distinct qualifying orders |
| Gross sales | Sum of eligible sales amounts before specified deductions |
| Net sales | Sales after the agreed discounts, returns, and adjustments |
| Average order value | Defined order revenue divided by qualifying orders |
| Active customers | Distinct customers meeting a defined activity rule |

These definitions are illustrative. The business must specify inclusion rules, time windows, currency conversion, cancellations, refunds, and other relevant conditions.

### Example SQL: Daily revenue

Assume `fact_order_lines` contains one row per order line and `net_amount` is already defined according to the approved revenue rules.

```sql
SELECT
    order_date,
    SUM(net_amount) AS daily_net_revenue
FROM fact_order_lines
WHERE order_status = 'COMPLETED'
GROUP BY order_date
ORDER BY order_date;
```

This query is correct only if the source's status and `net_amount` definitions match the business KPI. Refunds or adjustments stored separately must also be included if required by that definition.

### Example SQL: Monthly revenue by region

```sql
SELECT
    d.year,
    d.month,
    c.region,
    SUM(f.net_amount) AS net_revenue
FROM fact_sales f
JOIN dim_date d
    ON f.date_key = d.date_key
JOIN dim_customer c
    ON f.customer_key = c.customer_key
GROUP BY
    d.year,
    d.month,
    c.region
ORDER BY
    d.year,
    d.month,
    c.region;
```

This assumes the dimension joins preserve the intended fact grain and the customer dimension has the correct historical version for each fact.

### Metric governance

For every important KPI, document:
- Business meaning
- Formula
- Inclusion and exclusion rules
- Grain
- Time zone and reporting calendar
- Currency handling
- Data owner
- Refresh frequency
- Validation and reconciliation rules

---

## 8. Aggregation Strategy

Large reports can be expensive if every dashboard request scans billions of detailed records.

Pre-aggregation reduces the amount of data read at query time.

### Detailed data

```text
fact_sales
- Individual transaction or order-line records
```

### Aggregate data

```text
agg_daily_sales
- sales_date
- region
- product_category
- order_count
- units_sold
- net_revenue
```

A dashboard that displays monthly revenue by region can query a daily or monthly aggregate rather than scanning all transaction-level records, provided the aggregate supports the required filters and metric definitions.

### When to use aggregates

Use pre-aggregated tables when:
- The same report is queried frequently.
- The source detail is very large.
- The KPI has stable definitions.
- Low-latency responses are important.

Keep detailed data available for drill-down, investigation, and new analytical questions.

### Aggregate refresh strategies

**Full refresh:** Recalculate the entire aggregate. Simple but potentially expensive.

**Incremental refresh:** Recalculate only affected dates, partitions, or keys. More efficient, but requires reliable change tracking and correction logic.

For example, a late-arriving order from yesterday may require recalculating yesterday's aggregate even though the current processing date is today.

A robust pipeline identifies affected business periods and refreshes them safely.

---

## 9. Query Performance and Concurrency

Reporting performance depends on data layout, query design, compute resources, and concurrent workload.

### Common optimization techniques

1. **Column pruning:** Read only required columns.
2. **Partition pruning:** Skip irrelevant partitions.
3. **Predicate pushdown:** Push supported filters closer to the data source.
4. **Pre-aggregation:** Reduce rows before dashboard queries.
5. **Join optimization:** Use appropriate join strategies and statistics.
6. **File compaction:** Reduce excessive small files.
7. **Caching:** Reuse results or data when freshness and invalidation are managed.
8. **Workload isolation:** Separate interactive queries from heavy transformations when needed.
9. **Concurrency controls:** Limit or queue expensive queries if required.
10. **Capacity planning:** Size compute based on measured workload and service targets.

These techniques are not guaranteed to help every query. Validate improvements using query plans, execution metrics, and representative load tests.

### Interactive and batch workloads

Heavy data transformations can compete with interactive dashboards for compute, memory, and I/O.

Possible design:
- Transformation compute for ingestion and data preparation.
- SQL-serving compute for dashboards and ad hoc analytics.
- Separate workloads or pools for scheduled report generation.

The degree of separation depends on the platform and workload size.

### Caching considerations

Caching can improve repeated-query latency, but cached results may become stale.

Define:
- Cache lifetime
- Refresh or invalidation behavior
- Whether users can tolerate stale data
- Whether results vary by user permissions

Caching must not bypass row-level security or expose one user's results to another.

---

## 10. Freshness and Scheduling

Freshness is a business requirement, not just a technical metric.

A daily report may be acceptable with a 24-hour refresh cycle, while an operations dashboard may require updates every few minutes.

### End-to-end freshness

Track the time between the source event or source update and the moment the result becomes available to report users.

A useful freshness measure is:

```text
Data freshness lag = Current time - Latest qualifying source event time
```

This metric must account for the source's event-time semantics and expected delays. A recent processing timestamp does not prove that the underlying source data is complete.

### Scheduling example

```text
02:00  Ingest source data
02:30  Validate and standardize
03:00  Update curated tables
03:30  Refresh reporting aggregates
04:00  Run reconciliation checks
04:15  Publish report-ready status
04:30  Send scheduled reports
```

These are illustrative times, not a universal schedule. The workflow should be dependency-driven rather than relying only on fixed delays.

A report should be marked ready only after its required upstream data and validation checks have succeeded.

---

## 11. Reliability and Failure Recovery

A large reporting platform must handle failures without publishing incorrect or incomplete results.

### Common failure scenarios

| Failure | Possible response |
|---|---|
| Source system unavailable | Retry with backoff; alert if freshness is at risk |
| Ingestion fails | Resume from a safe checkpoint or replay source data |
| Transformation fails | Retry after correcting the issue; avoid partial publication |
| Duplicate source records | Apply deterministic deduplication rules |
| Late-arriving data | Reprocess affected windows or partitions |
| Incorrect KPI calculation | Fix the logic, backfill affected periods, reconcile |
| Serving layer unavailable | Recover or redirect to a supported fallback |
| Report delivery fails | Retry delivery without unnecessarily recalculating the report |

### Idempotency

An operation is idempotent when repeating it has the same final effect under its defined conditions.

For reporting pipelines, idempotency can be implemented through:
- Deterministic merge keys
- Deduplication rules
- Partition replacement
- Transactional writes
- Stable batch identifiers
- Safe retry logic

Appending the same batch repeatedly is not idempotent unless duplicates are prevented by another mechanism.

### Atomic publication

Do not expose partially refreshed reporting data as if it were complete.

Possible approaches include:
- Transactional table writes
- Staging data before publication
- Replacing affected partitions atomically where supported
- Publishing a validated snapshot or version
- Switching a reporting view to the new completed dataset

The implementation depends on the storage and serving technology.

### Reconciliation

Compare source and target data using suitable checks:
- Record counts
- Amount totals
- Distinct business keys
- Missing records
- Duplicate records
- Period-level totals

Counts alone are not enough. Two datasets can have the same number of records but contain different business data.

---

## 12. Data Quality and Observability

A report can be technically available and still be wrong. Monitoring must cover both pipeline health and data correctness.

### Pipeline monitoring

- Job success and failure
- Duration and resource utilization
- Retry counts
- Queue or backlog size
- Source ingestion lag
- Serving query latency
- Report generation and delivery status

### Data monitoring

- Freshness
- Completeness
- Uniqueness
- Validity
- Referential integrity
- Unexpected schema changes
- Reconciliation differences
- Changes in distributions or business metrics

### Example quality checks

```sql
-- Find duplicate order identifiers
SELECT
    order_id,
    COUNT(*) AS row_count
FROM fact_orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

Use this check only if the table grain requires one row per order. It would be incorrect for a table containing multiple order lines per order.

```sql
-- Identify records with missing required values
SELECT *
FROM fact_orders
WHERE order_id IS NULL
   OR order_date IS NULL;
```

A production pipeline should define what happens when checks fail:
- Stop publication
- Quarantine invalid records
- Allow publication with a warning
- Escalate to the data owner

The response depends on the severity of the issue and the business requirement.

---

## 13. Security and Governance

Reporting systems often expose commercially sensitive and personally identifiable information.

### Core controls

- Centralized identity management
- Role-based access control
- Least-privilege permissions
- Row-level and column-level security where required
- Encryption in transit and at rest
- Secret management
- Data classification and retention policies
- Audit logging
- Controlled access to exports and scheduled reports

### Example

A regional manager may be allowed to view sales for their own region but not other regions.

Enforce this restriction in a trusted serving or semantic layer. Hiding a region filter in the dashboard UI is not a security boundary.

### Governance practices

- Assign owners to critical datasets and KPIs.
- Maintain definitions in a data catalog or metrics layer.
- Track lineage from source to report.
- Record schema and metric changes.
- Use controlled development, testing, and production environments.

---

## 14. Capacity Estimation

Capacity estimates help determine whether the proposed design is realistic.

Suppose a business has:
- 100 million new records per day
- Average stored record size of 500 bytes
- 10,000 daily report users
- A peak of 500 concurrent users

### Raw data volume

```text
Daily raw volume
= 100,000,000 × 500 bytes
= 50,000,000,000 bytes
≈ 50 GB per day (decimal units)
```

Approximate raw storage for 30 days:

```text
50 GB × 30 = 1.5 TB
```

This excludes indexes, metadata, replicas, retained historical versions, derived tables, and other overhead. Compression may reduce physical storage, depending on the data and format.

### Average ingestion rate

Assuming records arrive evenly over 24 hours:

```text
100,000,000 / 86,400
≈ 1,157 records per second
```

Real ingestion rates are rarely perfectly uniform. Peak traffic, backfills, and retries can create much higher bursts.

### Reporting query load

Daily users alone do not determine query throughput. Estimate:
- Queries per user per day
- Whether users run queries simultaneously
- Query duration
- Dashboard refresh frequency
- Scheduled report volume
- Query complexity and data scanned

Then test with a representative workload. Do not infer the required number of compute nodes from user count alone.

---

## 15. Azure-Based Implementation

One possible implementation uses Azure services.

| Layer | Possible technology | Responsibility |
|---|---|---|
| Source systems | Azure SQL, operational databases, APIs | Generate business data |
| Batch ingestion | Azure Data Factory | Schedule and move data |
| Streaming ingestion | Azure Event Hubs | Ingest event streams |
| Data lake | Azure Data Lake Storage Gen2 | Store raw and curated data |
| Processing | Azure Databricks with PySpark | Clean and transform data |
| Table layer | Delta Lake | Manage reliable analytical tables |
| Serving | Databricks SQL or Azure SQL, depending on workload | Serve reporting queries |
| BI | Power BI | Build dashboards and reports |
| Orchestration | Azure Data Factory or Databricks Workflows | Coordinate pipeline tasks |
| Governance | Microsoft Purview and platform access controls | Catalog, lineage, and governance |
| Monitoring | Azure Monitor and platform-native monitoring | Track failures and performance |

This is an example architecture, not a requirement to use every service. The serving technology should be chosen based on concurrency, query patterns, integration needs, cost, and operational constraints.

### End-to-end flow

1. ADF ingests source database records into ADLS Gen2.
2. Raw data is retained in the Bronze layer.
3. Databricks validates, standardizes, and deduplicates the data.
4. Silver tables store clean, detailed business records.
5. Gold tables store dimensional models and reporting aggregates.
6. A serving SQL engine executes dashboard queries.
7. Power BI presents the metrics to users.
8. Monitoring tracks pipeline status, data freshness, quality, and query latency.

### Example PySpark transformation

```python
from pyspark.sql import functions as F

clean_orders = (
    raw_orders
    .filter(F.col("order_id").isNotNull())
    .withColumn("order_date", F.to_date("order_timestamp"))
    .withColumn("amount", F.col("amount").cast("decimal(18,2)"))
    .filter(F.col("amount").isNotNull())
)
```

This is only a basic cleaning example. Production logic should also define timestamp parsing and time zones, invalid-value handling, duplicate resolution, currency rules, and business-level validation.

### Example daily aggregate

Assume `clean_orders` has already been deduplicated according to a defined business rule, and each row represents one order.

```python
daily_sales = (
    clean_orders
    .groupBy("order_date", "region")
    .agg(
        F.countDistinct("order_id").alias("order_count"),
        F.sum("amount").alias("total_amount")
    )
)
```

If the input contains order-line records, `countDistinct` may still calculate order count, but the meaning of `amount` must be checked to avoid summing repeated order-level totals.

---

## 16. Important Trade-Offs

### Freshness vs cost

Frequent processing reduces data lag but increases compute usage and operational complexity.

Choose the refresh interval according to business needs.

### Pre-aggregation vs flexibility

Aggregates improve common dashboard queries but may not support every ad hoc dimension or calculation.

Keep detailed curated data for drill-down and new questions.

### Shared vs isolated compute

Shared compute can improve utilization and reduce cost. Isolated serving compute can protect dashboard performance from heavy transformations.

Choose based on workload contention and service-level requirements.

### Centralized metrics vs team autonomy

A shared metrics layer improves consistency. Teams may still need flexibility for exploratory calculations.

Separate official business KPIs from clearly labeled exploratory metrics.

### Full refresh vs incremental refresh

Full refresh is simpler and may be appropriate for smaller datasets. Incremental refresh saves compute at scale but requires reliable change detection, correction handling, and reconciliation.

---

## 17. Common Design Mistakes

1. Running every dashboard directly against operational databases.
2. Reloading all historical data for every small change.
3. Using streaming when daily batch processing meets the requirement.
4. Building dashboards on raw, unvalidated data.
5. Allowing each dashboard to define revenue differently.
6. Joining fact tables with different grains without aggregation.
7. Summing repeated order totals or counting order lines as orders.
8. Treating ingestion success as proof of data correctness.
9. Advancing source checkpoints before target writes succeed.
10. Retrying append operations without deduplication or idempotency.
11. Publishing data before validation and reconciliation finish.
12. Ignoring late-arriving data and historical corrections.
13. Using highly unique columns as physical partition keys.
14. Adding caches without considering freshness and authorization.
15. Sizing compute from user counts without load testing.
16. Monitoring job status but not data freshness or KPI accuracy.
17. Treating dashboard filters as security controls.
18. Failing to preserve lineage, metric definitions, and ownership.

---

## 18. Interview Walkthrough

If asked, “Design a large-scale reporting platform,” structure your answer as follows.

### Step 1: Clarify the requirements

Ask about data volume, refresh interval, concurrent users, response-time targets, report types, historical retention, and security.

### Step 2: Identify the sources

List operational databases, APIs, files, and event streams. Choose batch, incremental, CDC, or streaming ingestion based on the source and freshness needs.

### Step 3: Design the data flow

Describe ingestion, raw storage, transformation, curated tables, aggregate tables, serving, and BI.

### Step 4: Design the data model

Define fact-table grains, dimensions, KPI formulas, historical attribute handling, and relationships between datasets.

### Step 5: Optimize the reporting workload

Use appropriate columnar storage, partition pruning, query optimization, aggregates, and suitable serving compute.

### Step 6: Ensure correctness

Cover deduplication, idempotency, data quality checks, reconciliation, late-arriving data, and safe publication.

### Step 7: Ensure reliability

Explain retries, checkpointing, recovery, reprocessing, backfills, alerting, and how the platform avoids exposing partial results.

### Step 8: Secure the platform

Discuss least privilege, row/column security, secrets, audit logs, governance, and lineage.

### Step 9: Estimate capacity

Calculate raw data volume, ingestion rate, retention storage, peak workload, concurrency, and expected query load. Validate assumptions through testing.

### Step 10: Explain trade-offs

Justify choices based on business requirements rather than naming technologies without a reason.

---

## 19. Interview Questions and Answers

### Q1. What is a reporting platform?

A system that ingests and prepares data, calculates business metrics, and serves reports and dashboards to users.

### Q2. Why not query the operational database directly?

Analytical queries may compete with transactional workloads, increase source latency, and require expensive joins or scans. A separate analytical layer reduces this contention.

### Q3. What is the difference between a data warehouse and a reporting layer?

A warehouse stores and organizes analytical data. A reporting layer exposes business-ready models, metrics, and aggregates optimized for reporting.

### Q4. Why are aggregate tables useful?

They reduce the amount of data scanned and processed for frequently requested metrics, improving response time and potentially reducing cost.

### Q5. How do you ensure reports are accurate?

Use governed metric definitions, quality checks, deduplication, correct table grain, reconciliation, and controlled publication.

### Q6. How do you handle late-arriving data?

Identify affected business dates or keys, update detailed records safely, recalculate affected aggregates, and track whether the report is current.

### Q7. How do you prevent duplicate data during retries?

Use deterministic keys, idempotent writes, deduplication, transactional operations, or safe partition replacement.

### Q8. What happens if an aggregate refresh fails halfway?

Keep the previous valid report available where possible, prevent incomplete results from being published, retry or rebuild the affected data, validate it, and then publish the completed version.

### Q9. How would you improve a slow dashboard?

Inspect query plans and execution metrics, identify large scans and expensive joins, verify partition pruning and statistics, consider pre-aggregation or caching, and test the serving capacity under realistic concurrency.

### Q10. How do you support both detailed and summary reports?

Store detailed curated data for drill-down and historical analysis, then build aggregates for common reporting queries.

### Q11. How do you ensure consistency across dashboards?

Centralize important metric definitions in a governed semantic or metrics layer, document business rules, and validate implementations against agreed definitions.

### Q12. How would you monitor the platform?

Monitor job health, ingestion lag, data freshness, data quality, reconciliation, query latency, concurrency, report delivery, and cost.

### Q13. How would you support millions of records per day?

Use incremental ingestion, scalable distributed processing where appropriate, columnar storage, efficient data layouts, incremental aggregates, and measured serving capacity.

### Q14. Does a successful pipeline mean the report is correct?

No. The pipeline may succeed while loading incomplete, duplicated, stale, or semantically incorrect data. Data quality and reconciliation are separate requirements.

### Q15. How would you choose between batch and streaming?

Use batch when the required freshness can tolerate scheduled updates. Use streaming when the business needs continuous or low-latency updates and the additional complexity is justified.

---

## 20. Quick Revision

- **Reporting platform:** Converts source data into trusted, usable reports.
- **Ingestion:** Batch, incremental, CDC, or streaming according to freshness needs.
- **Bronze:** Raw source data.
- **Silver:** Cleaned and standardized records.
- **Gold:** Business-ready models and aggregates.
- **Fact table:** Stores measurements at a defined grain.
- **Dimension table:** Provides descriptive context.
- **Semantic layer:** Defines consistent business metrics.
- **Aggregation:** Reduces query work for common reports.
- **Idempotency:** Makes retries safe under defined conditions.
- **Reconciliation:** Checks that source and target results agree according to business rules.
- **Freshness:** Measures how current the report data is.
- **Observability:** Monitors pipeline health, data quality, and serving performance.
- **Security:** Enforces authorization in trusted data-serving components.
- **Scalability:** Supports growing data, queries, and users.
- **Atomic publication:** Prevents incomplete refreshes from appearing as valid reports.

## 21. Self-Assessment

Before moving on, make sure you can answer these without reading the note:

- [ ] Draw the end-to-end reporting architecture.
- [ ] Explain when to use batch, incremental ingestion, CDC, and streaming.
- [ ] Define fact-table grain and explain why it prevents incorrect totals.
- [ ] Explain why semantic layers and aggregate tables are useful.
- [ ] Describe how to handle late data and historical corrections.
- [ ] Explain idempotency and safe retries.
- [ ] Describe how to prevent partial data from being published.
- [ ] Explain how to investigate slow dashboard queries.
- [ ] Estimate daily storage volume and average ingestion rate.
- [ ] Describe security, lineage, monitoring, and reconciliation.
- [ ] Walk through an Azure-based implementation.
- [ ] Explain the trade-offs in your proposed design.

**Final interview principle:** Start with business requirements, design for correct metrics, and then optimize freshness, latency, scale, reliability, security, and cost. A reporting platform is successful when users can trust its results—not merely when the pipeline runs.