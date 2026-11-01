

# Data Lakehouse Design

## 1. Introduction

A **Data Lakehouse** is a data architecture that combines the flexibility and scalability of a data lake with important data management and analytical capabilities traditionally associated with a data warehouse.

It allows organizations to store large amounts of data in a data lake while supporting reliable tables, structured analytics, governance, and SQL-based querying.

### Why Do We Need a Lakehouse?

Traditionally, organizations used two separate systems:

- **Data Lake:** Stores large volumes of raw, structured, semi-structured, and unstructured data.
- **Data Warehouse:** Stores organized, cleaned, and modeled data for reporting and analytics.

Maintaining both systems can create:
- Duplicate data storage.
- Additional data movement.
- Higher operational costs.
- Complex data pipelines.
- Multiple governance mechanisms.

A lakehouse attempts to reduce this complexity by bringing data lake storage and warehouse-like capabilities together.

---

## 2. Data Lake vs Data Warehouse vs Lakehouse

<AsyncImage query="data lake data warehouse data lakehouse architecture comparison diagram" aspectRatio="16:9" width="100%" />

| Feature | Data Lake | Data Warehouse | Data Lakehouse |
|---|---|---|---|
| Storage | Object storage | Warehouse-managed storage | Data lake storage |
| Data Types | Structured, semi-structured, unstructured | Primarily structured | Structured, semi-structured, unstructured |
| Schema | Often schema-on-read | Typically schema-on-write | Supports managed table schemas |
| Transactions | Depends on storage/table layer | Usually supported | Supported by table format |
| Scalability | High | Depends on platform | High |
| SQL Analytics | Depends on query engine | Strong support | Strong support |
| Data Governance | Requires additional tools or services | Usually integrated | Supported through catalog and governance layers |
| Typical Use | Raw data and exploration | BI and reporting | Engineering, analytics, and BI |

### Data Lake

A data lake stores data in its original or minimally processed form.

Example:

```text
data-lake/
    customers/
        customers.csv
    orders/
        orders.json
    transactions/
        transactions.parquet
    application-logs/
        logs.json
```

Advantages:
- Flexible storage.
- Supports different data formats.
- Suitable for large-scale data.
- Useful for machine learning and data engineering.

Limitations:
- Requires additional table management for reliable updates.
- Poorly managed lakes can become difficult to query and govern.
- Raw files alone do not provide warehouse-style transactions.

### Data Warehouse

A data warehouse organizes data into structured tables designed for analytical queries.

Example:

```text
sales_warehouse/
    fact_sales
    dim_customer
    dim_product
    dim_date
```

Advantages:
- Structured analytical data.
- Strong SQL support.
- Optimized reporting.
- Managed data models.

Limitations:
- Can require additional ingestion and transformation before loading.
- Storage and compute costs depend on the platform.
- Less flexible for some raw-data and unstructured-data workloads.

### Data Lakehouse

A lakehouse combines data lake storage with a transactional table layer and supporting management capabilities.

Example:

```text
                 Data Lakehouse
                       |
              Object Storage
                       |
              +--------+--------+
              |                 |
          Data Files       Transaction Log
          (Parquet)        (Delta Log)
              |                 |
              +--------+--------+
                       |
                Table Management
                       |
               SQL / Spark Engine
                       |
                Analytics / BI
```

The exact architecture depends on the table format and platform.

---

## 3. Core Architecture of a Lakehouse

A lakehouse typically contains the following components:

```text
               Data Sources
             /      |       \
            /       |        \
        Databases   APIs     Files
            \       |        /
             \      |       /
                    |
                    v
              Data Ingestion
                    |
                    v
             Object Storage
                    |
                    v
             Table Format
          (Delta / Iceberg / Hudi)
                    |
                    v
            Processing Engine
              (Spark / SQL)
                    |
                    v
             Data Organization
             Bronze / Silver / Gold
                    |
                    v
             Catalog and Governance
                    |
                    v
              Data Consumption
               /     |      \
              /      |       \
             BI    Analytics   ML
```

### 3.1 Storage Layer

Stores the actual data files.

Common technologies:
- Azure Data Lake Storage Gen2.
- Amazon S3.
- Google Cloud Storage.
- HDFS.

The storage layer provides scalable and durable storage.

### 3.2 Table Format Layer

A table format manages how data files behave as logical tables.

Examples:
- Delta Lake.
- Apache Iceberg.
- Apache Hudi.

It provides capabilities such as:
- Transaction management.
- Schema management.
- Metadata tracking.
- Updates and deletes.
- Table versioning, depending on the format.

### 3.3 Processing Layer

Executes transformations and analytical queries.

Examples:
- Apache Spark.
- PySpark.
- Databricks SQL.
- Trino.
- Other compatible SQL engines.

### 3.4 Catalog and Governance Layer

Manages:
- Table definitions.
- Permissions.
- Metadata.
- Data discovery.
- Lineage.
- Auditing.

Examples include Unity Catalog and other compatible catalog services.

### 3.5 Consumption Layer

Provides data to:
- Power BI.
- Tableau.
- SQL users.
- Data scientists.
- Machine learning applications.
- Downstream services.

---

## 4. Medallion Architecture

A common way to organize lakehouse data is the Medallion Architecture.

It divides data into three logical layers:

<AsyncImage query="Databricks medallion architecture bronze silver gold data lakehouse diagram" aspectRatio="16:9" width="100%" />

### Bronze Layer – Raw Data

Stores data as received from source systems, usually with minimal transformation.

Example:

| order_id | customer_id | amount | status |
|---|---|---:|---|
| 101 | 1 | 500 | completed |
| 102 | 2 | 800 | cancelled |
| 103 | 1 | 700 | completed |

Responsibilities:
- Preserve source data.
- Store ingestion metadata.
- Support historical reprocessing.
- Retain source-level information.

### Silver Layer – Cleaned Data

Contains validated, standardized, and deduplicated data.

Typical operations:
- Null handling.
- Type conversion.
- Deduplication.
- Schema validation.
- Data standardization.
- Data enrichment.

Example:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | 1 | 500 |
| 103 | 1 | 700 |

### Gold Layer – Business-Ready Data

Contains curated datasets designed for business reporting and analytics.

Example:

| customer_id | total_sales | order_count |
|---|---:|---:|
| 1 | 1200 | 2 |

Responsibilities:
- Business aggregations.
- KPI calculations.
- Analytical data models.
- Reporting datasets.

### Comparison

| Layer | Purpose | Data Condition |
|---|---|---|
| Bronze | Raw ingestion | Original or minimally processed |
| Silver | Data cleaning | Validated and standardized |
| Gold | Business consumption | Curated and aggregated |

**Important:** Bronze, Silver, and Gold are logical data organization layers. They do not necessarily require separate storage accounts or physical clusters.

---

## 5. Why Does a Lakehouse Need a Table Format?

Object storage can store files, but individual Parquet files do not automatically provide all the capabilities expected from a database table.

For example, consider an orders dataset containing hundreds of Parquet files.

A user updates an order.

Without a transactional table-management layer, the system must coordinate file changes and metadata carefully. Concurrent readers and writers can otherwise encounter inconsistent views.

A table format provides a structured way to manage those files.

### Important Capabilities

| Capability | Purpose |
|---|---|
| ACID Transactions | Manage reliable table changes |
| Schema Enforcement | Validate incoming table schemas |
| Schema Evolution | Support controlled schema changes |
| Time Travel | Access retained historical table versions |
| MERGE | Apply inserts and updates |
| DELETE | Remove records logically |
| Metadata Management | Track files and table state |
| Concurrency Control | Coordinate conflicting operations |

Not every table format or platform implements every feature in exactly the same way.

---

## 6. Delta Lake

**Delta Lake** is an open-source storage framework that adds transactional table capabilities to data lakes.

It commonly uses:
- Parquet files for data storage.
- A transaction log for table metadata and changes.
- Spark and compatible engines for reading and writing.

### Delta Lake Architecture

```text
             Delta Table
                  |
        +---------+---------+
        |                   |
        v                   v
    Parquet Files       _delta_log/
        |                   |
        |             Transaction Log
        |                   |
        +---------+---------+
                  |
                  v
            Table Snapshot
                  |
                  v
           Spark / SQL Query
```

### Transaction Log

The `_delta_log` directory stores information about table transactions.

It tracks:
- Added files.
- Removed files.
- Table metadata.
- Protocol information.
- Transaction information.

A successful commit creates a new table version.

The transaction log allows readers to identify which data files belong to a particular table snapshot.

### Example: Create a Delta Table

```python
orders_df.write \
    .format("delta") \
    .mode("overwrite") \
    .save("/lakehouse/bronze/orders")
```

### Read a Delta Table

```python
orders_df = (
    spark.read
    .format("delta")
    .load("/lakehouse/bronze/orders")
)
```

### Update a Delta Table

```sql
UPDATE delta.`/lakehouse/silver/orders`
SET status = 'completed'
WHERE order_id = 101;
```

### MERGE Example

```sql
MERGE INTO delta.`/lakehouse/silver/orders` AS target
USING source_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
    UPDATE SET *

WHEN NOT MATCHED THEN
    INSERT *;
```

The source should be deduplicated so that multiple source records do not ambiguously match the same target row.

---

## 7. ACID Transactions in a Lakehouse

ACID stands for:

- Atomicity.
- Consistency.
- Isolation.
- Durability.

Delta Lake provides ACID transaction capabilities for supported table operations.

### Atomicity

A transaction is committed as a unit. Readers do not see a partially committed table snapshot.

### Consistency

A transaction moves the table from one valid state to another, subject to the constraints and rules enforced by the system.

### Isolation

Concurrent operations are coordinated to provide supported isolation behavior.

Conflicting transactions may fail and require retry.

### Durability

Once a transaction is successfully committed, its committed state is preserved according to the storage system's durability guarantees.

**Interview point:** Parquet is a file format. Delta Lake adds a transaction and table-management layer on top of data files.

---

## 8. Delta Lake vs Apache Iceberg vs Apache Hudi

These are table formats or table-management frameworks used to manage analytical tables on data lakes.

| Feature | Delta Lake | Apache Iceberg | Apache Hudi |
|---|---|---|---|
| Main Purpose | Transactional lakehouse tables | Open analytical table format | Data lake table management |
| Storage | Commonly Parquet | Commonly Parquet, also supports other formats | Commonly Parquet, also supports other formats |
| Transactions | Supported | Supported | Supported |
| Schema Evolution | Supported | Supported | Supported |
| Time Travel | Supported | Supported | Supported |
| Updates and Deletes | Supported | Supported | Supported |
| Common Strength | Spark and Databricks ecosystem | Engine interoperability and table evolution | Incremental data processing and upserts |

### When to Use Them

- **Delta Lake:** A strong choice for Spark and Databricks-centered lakehouse architectures.
- **Iceberg:** Useful when open table interoperability across multiple query engines is important.
- **Hudi:** Useful for workloads involving frequent incremental ingestion, upserts, and change-oriented processing.

The best choice depends on workload requirements, engine compatibility, operational maturity, and platform support. Feature support and implementation details vary by version.

---

## 9. Batch Processing in a Lakehouse

Batch processing is commonly used to transform data between lakehouse layers.

### Example Flow

```text
Source Database
       |
       v
   Bronze Layer
       |
       v
   Silver Layer
       |
       v
    Gold Layer
       |
       v
    BI Reports
```

### Example: Bronze to Silver

```python
from pyspark.sql import functions as F

bronze_orders = (
    spark.read
    .format("delta")
    .load("/lakehouse/bronze/orders")
)

silver_orders = (
    bronze_orders
    .filter(F.col("order_id").isNotNull())
    .filter(F.col("customer_id").isNotNull())
    .dropDuplicates(["order_id"])
    .withColumn(
        "amount",
        F.col("amount").cast("decimal(12,2)")
    )
)

(
    silver_orders.write
    .format("delta")
    .mode("overwrite")
    .save("/lakehouse/silver/orders")
)
```

This is a simplified example. Production pipelines should define deterministic deduplication rules, validate conversion failures, and avoid overwriting historical data unintentionally.

### Example: Silver to Gold

```python
silver_orders = (
    spark.read
    .format("delta")
    .load("/lakehouse/silver/orders")
)

gold_sales = (
    silver_orders
    .filter(F.col("status") == "completed")
    .groupBy("customer_id")
    .agg(
        F.sum("amount").alias("total_sales"),
        F.countDistinct("order_id").alias("order_count")
    )
)

(
    gold_sales.write
    .format("delta")
    .mode("overwrite")
    .save("/lakehouse/gold/customer_sales")
)
```

---

## 10. Streaming in a Lakehouse

A lakehouse can support both batch and streaming workloads.

For example:
- Batch jobs process daily historical records.
- Streaming jobs process newly arriving events.
- Both can write to managed lakehouse tables.

### Example Architecture

```text
            Data Sources
             /       \
            /         \
       Batch Data   Live Events
            |           |
            v           v
       Batch Jobs   Streaming Jobs
            |           |
            v           v
          Bronze Delta Tables
                  |
                  v
             Silver Tables
                  |
                  v
              Gold Tables
                  |
                  v
             BI / Analytics
```

### Advantages

- Shared storage and table management.
- Supports historical and near-real-time analytics.
- Reduces the need for separate storage systems.
- Allows downstream consumers to query curated data.

### Important Considerations

- Streaming checkpoints must be managed carefully.
- Late-arriving events require an event-time strategy.
- Duplicate handling depends on source, processing, and sink design.
- Batch and streaming writers must be designed for safe concurrent access.
- A shared table does not automatically guarantee exactly-once end-to-end processing.

---

## 11. Catalog and Governance

A lakehouse needs metadata and governance to make data discoverable, secure, and manageable.

### What Is a Data Catalog?

A data catalog stores metadata about datasets and tables.

It can contain:
- Table names.
- Column names and data types.
- Table locations.
- Ownership.
- Descriptions.
- Permissions.
- Lineage information.

### Unity Catalog

Unity Catalog is Databricks' centralized governance solution.

It supports:
- Centralized access control.
- Data discovery.
- Audit capabilities.
- Lineage.
- Management of tables, views, and other data assets.

A common namespace structure is:

```text
catalog.schema.table
```

Example:

```sql
SELECT *
FROM production.sales.orders;
```

Where:
- `production` is the catalog.
- `sales` is the schema.
- `orders` is the table.

### Governance Best Practices

1. Apply least-privilege access.
2. Use managed identities where appropriate.
3. Avoid hardcoding credentials.
4. Classify sensitive data.
5. Maintain data lineage.
6. Monitor access and changes.
7. Define retention policies.
8. Separate development and production access.

---

## 12. Performance Optimization

Lakehouse performance depends on storage organization, file layout, query execution, and compute resources.

### 1. Partitioning

Organize data by commonly filtered columns when the data volume and query patterns justify it.

```python
(
    orders_df.write
    .format("delta")
    .partitionBy("order_date")
    .save("/lakehouse/silver/orders")
)
```

Avoid high-cardinality partition columns that create too many small directories or files.

### 2. File Compaction

Small files can increase metadata and file-opening overhead.

Delta Lake supports:

```sql
OPTIMIZE silver.orders;
```

This compacts eligible small files into larger files.

### 3. Data Skipping

Delta Lake can use file-level statistics to skip files that cannot satisfy query filters.

Its effectiveness depends on available statistics and the data layout.

### 4. Z-Ordering

Z-Ordering reorganizes data to improve data skipping for selected columns.

```sql
OPTIMIZE silver.orders
ZORDER BY (customer_id);
```

Z-Ordering is not the same as partitioning. Its availability and behavior depend on the platform and runtime.

### 5. Query Optimization

- Select only required columns.
- Filter unnecessary records early.
- Optimize joins.
- Handle data skew.
- Avoid unnecessary shuffles.
- Use appropriate compute resources.
- Inspect execution plans and runtime metrics.

### 6. Caching

Cache data only when repeated access makes it beneficial. Caching unnecessary datasets can waste memory.

---

## 13. Reliability and Data Recovery

A production lakehouse must support reliable writes, recovery, and historical processing.

### Important Capabilities

**Time Travel**

Allows users to query retained historical table versions.

```sql
SELECT *
FROM silver.orders VERSION AS OF 10;
```

**Restore**

Restores a table to a previous state by creating a new table version.

```sql
RESTORE TABLE silver.orders TO VERSION AS OF 10;
```

**VACUUM**

Removes eligible unreferenced data files after the retention period.

```sql
VACUUM silver.orders;
```

**Important cautions:**

- Time Travel requires the relevant transaction-log history and data files to remain available.
- VACUUM can make older versions unavailable by physically deleting files.
- The common default deleted-file retention period is seven days, but settings can vary.
- Do not disable retention safety checks casually.
- Restore does not erase the intervening transaction history.

### Idempotent Processing

A lakehouse pipeline should support safe retries.

Common techniques:
- MERGE using stable business keys.
- Deterministic transformations.
- Controlled partition overwrites.
- Deduplication.
- Progress tracking.
- Reconciliation checks.

A checkpoint alone does not guarantee exactly-once results for arbitrary external sinks.

---

## 14. Lakehouse Design Considerations

Before designing a lakehouse, evaluate the following.

| Area | Design Questions |
|---|---|
| Data Sources | Which databases, APIs, and files are involved? |
| Data Volume | How much data arrives daily? |
| Data Velocity | Is batch or streaming required? |
| Storage | Which object storage platform will be used? |
| Table Format | Delta, Iceberg, or Hudi? |
| Processing | Spark, SQL, or another engine? |
| Data Organization | What Bronze, Silver, and Gold layers are needed? |
| Governance | How will access and metadata be managed? |
| Performance | What are the query and SLA requirements? |
| Reliability | How will failures and reruns be handled? |
| Cost | How will storage and compute costs be controlled? |
| Consumption | Which BI tools and applications need the data? |

### Common Design Mistakes

1. Assuming a data lake automatically provides ACID transactions.
2. Treating Parquet and Delta Lake as equivalent technologies.
3. Creating too many partitions.
4. Ignoring small-file problems.
5. Running VACUUM without understanding retention requirements.
6. Assuming time travel is permanent.
7. Using MERGE without deduplicating the source.
8. Ignoring concurrent writes.
9. Assuming a catalog automatically grants data access.
10. Publishing Gold tables without data quality checks.
11. Assuming a lakehouse removes every need for a traditional data warehouse.
12. Ignoring query-engine compatibility when selecting a table format.

---

## 15. Practical Scenario: Design an E-Commerce Lakehouse

### Business Requirement

An e-commerce company needs a centralized platform for:
- Customer information.
- Orders.
- Payments.
- Products.
- Daily sales analytics.
- Customer behavior analysis.

The platform should support historical reporting and future streaming requirements.

### Proposed Architecture

```text
                 Source Systems
               /       |        \
              /        |         \
        Customer DB  Orders DB  Payment API
              \        |         /
               \       |        /
                       |
                       v
                Data Ingestion
                       |
                       v
                 ADLS Gen2
                       |
                       v
                Bronze Delta
                       |
                       v
                Databricks
                  PySpark
                       |
                       v
                 Silver Delta
                       |
                       v
                Business Logic
                       |
                       v
                  Gold Delta
                       |
             +---------+---------+
             |                   |
             v                   v
          Power BI           SQL Analytics
```

### Step-by-Step Design

**Step 1: Ingestion**

Use Azure Data Factory to extract data from databases, APIs, and files.

**Step 2: Raw Storage**

Store source data in ADLS Gen2 using the Bronze layer.

**Step 3: Processing**

Use Databricks and PySpark to clean and validate data.

**Step 4: Silver Layer**

Create standardized customer, order, payment, and product tables.

**Step 5: Gold Layer**

Create analytical tables such as:
- Daily sales.
- Revenue by product.
- Customer purchase history.
- Payment success rate.

**Step 6: Governance**

Use Unity Catalog or an appropriate catalog and governance solution to manage metadata and access.

**Step 7: Consumption**

Connect Power BI and SQL analytics tools to curated datasets.

**Step 8: Monitoring**

Track pipeline failures, record counts, freshness, quality checks, and execution costs.

### Why Choose a Lakehouse?

- Supports raw and curated data.
- Provides scalable object storage.
- Supports transactional tables through Delta Lake.
- Allows batch and streaming workloads.
- Supports SQL analytics and BI.
- Reduces unnecessary data duplication between separate storage systems.

---

## 16. Interview Questions and Answers

### Q1. What is a data lakehouse?

A data lakehouse combines scalable data lake storage with table management, transactions, governance, and analytical capabilities.

### Q2. Why was the lakehouse architecture introduced?

It addresses limitations of traditional data lakes while reducing the need to maintain separate lake and warehouse systems for many workloads.

### Q3. What is the difference between a data lake and a lakehouse?

A traditional data lake primarily stores files. A lakehouse adds a table-management layer that supports reliable transactions, schema management, and analytical workloads.

### Q4. What is the difference between a data warehouse and a lakehouse?

A warehouse is designed primarily for structured analytical workloads. A lakehouse combines data lake storage with warehouse-like table and analytics capabilities.

### Q5. What is Delta Lake?

Delta Lake is a table framework that adds transactional capabilities to data lakes using data files and a transaction log.

### Q6. What is the purpose of the `_delta_log` directory?

It stores transaction history and metadata needed to reconstruct the active table snapshot.

### Q7. What is the Medallion Architecture?

It organizes data into Bronze, Silver, and Gold layers for raw ingestion, cleaned data, and business-ready data.

### Q8. Why is Parquet used in a lakehouse?

Parquet is a columnar file format that supports efficient analytical reads and compression.

### Q9. Is Delta Lake a file format?

No. Delta Lake is a table-management framework that commonly stores data in Parquet files and uses a transaction log.

### Q10. What are ACID transactions?

They are Atomicity, Consistency, Isolation, and Durability properties that help ensure reliable data operations.

### Q11. What is the difference between Delta Lake and Apache Iceberg?

Both are open table technologies. Their metadata structures, interoperability, ecosystem support, and feature implementations differ.

### Q12. What is Time Travel?

Time Travel allows users to query a previous table version when the required log and data files are still retained.

### Q13. What is VACUUM?

VACUUM removes eligible unreferenced data files after the configured retention period.

### Q14. What is the difference between OPTIMIZE and ZORDER?

OPTIMIZE compacts files. ZORDER reorganizes data to improve data skipping for selected columns.

### Q15. Can a lakehouse support streaming?

Yes. It can support batch and streaming ingestion and processing, depending on the engines and table formats used.

### Q16. What is Unity Catalog?

Unity Catalog is Databricks' centralized governance solution for managing data access, metadata, discovery, and lineage.

### Q17. How do you prevent duplicate records?

Use business keys, deterministic deduplication, and suitable MERGE or other idempotent write strategies.

### Q18. How do you optimize lakehouse performance?

Use suitable file formats, optimize joins, reduce unnecessary shuffles, compact small files, choose appropriate partitioning, and analyze query plans.

### Q19. Does a lakehouse eliminate the need for a data warehouse?

Not necessarily. Some organizations still use dedicated warehouses for specific workloads, performance requirements, or existing business systems.

### Q20. How would you design a lakehouse on Azure?

Use ADLS Gen2 for storage, ADF for orchestration and ingestion, Databricks with Delta Lake for processing and table management, a catalog for governance, and Power BI for reporting.

---

## 17. Quick Revision

| Concept | Key Point |
|---|---|
| Data Lake | Flexible storage for large volumes of diverse data |
| Data Warehouse | Structured analytical storage |
| Data Lakehouse | Data lake storage with reliable table and analytics capabilities |
| Delta Lake | Transactional table framework |
| Apache Iceberg | Open analytical table format |
| Apache Hudi | Table management for incremental and upsert-oriented workloads |
| Parquet | Columnar data file format |
| `_delta_log` | Delta transaction log |
| Bronze | Raw data |
| Silver | Cleaned and validated data |
| Gold | Business-ready data |
| ACID | Reliable transaction properties |
| Time Travel | Query retained historical versions |
| VACUUM | Remove eligible unreferenced files |
| OPTIMIZE | Compact small files |
| ZORDER | Improve data skipping through data organization |
| Unity Catalog | Centralized Databricks governance |

### Five Important Interview Points

1. A lakehouse combines data lake flexibility with warehouse-like capabilities.
2. Parquet stores data; Delta Lake adds transactional table management.
3. The transaction log tracks committed table changes and snapshots.
4. Medallion architecture organizes raw, cleaned, and curated data.
5. A production lakehouse needs governance, monitoring, data quality, and recovery mechanisms.

---

## 18. Self-Assessment

Try answering these questions without looking at the notes.

1. What is a data lakehouse?
2. Why do organizations use lakehouse architecture?
3. Explain data lake vs data warehouse vs lakehouse.
4. Draw the architecture of a lakehouse.
5. What is the role of a table format?
6. Explain Delta Lake architecture.
7. What is stored inside `_delta_log`?
8. Why is Parquet commonly used?
9. Explain Bronze, Silver, and Gold.
10. What are ACID transactions?
11. Compare Delta Lake, Iceberg, and Hudi.
12. How does a lakehouse support batch and streaming?
13. What is the role of Unity Catalog?
14. Explain OPTIMIZE vs ZORDER.
15. What is Time Travel?
16. What does VACUUM do?
17. How do you make a lakehouse pipeline idempotent?
18. How would you optimize a large Delta table?
19. How would you handle concurrent writes?
20. Design an Azure lakehouse for e-commerce analytics.

**Interview readiness:** You should be able to explain the lakehouse architecture, distinguish a file format from a table format, describe Delta Lake's transaction log, explain the Medallion Architecture, and design a practical Azure-based lakehouse.

