# Lakehouse Architecture

## 1. What Is a Data Lakehouse?

A **Data Lakehouse** is a data architecture that combines the flexible storage capabilities of a data lake with the data management and analytical capabilities commonly associated with a data warehouse.

It allows organizations to store different types of data in a shared storage layer while supporting reliable analytical tables, SQL queries, data processing, and machine learning workloads.

### Key Capabilities

- Stores structured, semi-structured, and unstructured data.
- Supports raw and processed datasets.
- Provides ACID transactions through supported table formats.
- Supports schema enforcement and schema evolution.
- Enables batch and streaming data processing.
- Supports SQL analytics and BI workloads.
- Provides data governance and access control through suitable platforms.
- Supports machine learning and advanced analytics.

**Simple definition for interviews:**

A data lakehouse combines the flexibility and scalability of a data lake with the reliability, transaction management, and analytical capabilities of a data warehouse.

---

## 2. Why Do We Need a Lakehouse?

Traditionally, organizations often maintained separate data lakes and data warehouses.

### Traditional Architecture

```text
              Data Sources
                   |
          +--------+--------+
          |                 |
          v                 v
      Data Lake        Data Warehouse
          |                 |
          v                 v
    Raw Data / ML      BI / Reporting
```

This architecture can create challenges:

1. **Data duplication:** Data may need to be copied from the lake into the warehouse.
2. **Higher storage costs:** Multiple copies of datasets may be maintained.
3. **Data movement:** Moving large datasets between systems adds complexity.
4. **Different governance:** Access controls and metadata may be managed separately.
5. **Operational complexity:** Multiple platforms and pipelines need maintenance.

A lakehouse aims to reduce these challenges by supporting multiple workloads over a shared data foundation.

### Lakehouse Architecture

```text
              Data Sources
                   |
                   v
             Data Ingestion
                   |
                   v
            Cloud Object Storage
                   |
                   v
          Lakehouse Table Format
          (Delta / Iceberg / Hudi)
                   |
          +--------+--------+
          |                 |
          v                 v
    SQL Analytics      Data Processing
          |                 |
          v                 v
     BI / Reports       ML / Analytics
```

The same underlying datasets can serve different workloads, subject to the capabilities of the platform and table format.

---

## 3. Main Components of Lakehouse Architecture

A lakehouse generally contains the following components.

### 3.1 Data Sources

Data comes from multiple systems.

Examples:

- Relational databases.
- Application databases.
- CSV and JSON files.
- APIs.
- Application logs.
- Kafka or other event streams.
- IoT devices.

Example:

An e-commerce company receives customer records, orders, payment transactions, and website events.

### 3.2 Data Ingestion Layer

The ingestion layer brings data from source systems into the lakehouse.

Common ingestion methods:

- Batch ingestion.
- Incremental ingestion.
- Change Data Capture (CDC).
- Streaming ingestion.
- File-based ingestion.

Common technologies:

- Azure Data Factory.
- Databricks Auto Loader.
- Apache Kafka.
- Azure Event Hubs.
- Apache Spark.

### 3.3 Storage Layer

The storage layer holds raw and processed data.

Common storage services:

- Azure Data Lake Storage Gen2.
- Amazon S3.
- Google Cloud Storage.

These services provide scalable object storage.

Data can be stored in formats such as:

- CSV.
- JSON.
- Parquet.
- Avro.

For analytical tables, lakehouse architectures commonly use table formats such as Delta Lake, Apache Iceberg, or Apache Hudi.

### 3.4 Table Format Layer

A table format manages how data files behave as a logical table.

Examples:

- Delta Lake.
- Apache Iceberg.
- Apache Hudi.

These formats add capabilities such as:

- Transaction management.
- Table metadata.
- Schema management.
- Data file tracking.
- Updates and deletes.
- Historical table versions, subject to format and engine support.

**Important:** Parquet is a file format. Delta Lake, Iceberg, and Hudi are table formats or table-management frameworks built around data files and metadata.

### 3.5 Processing Layer

The processing layer transforms raw data into useful datasets.

Common operations:

- Cleaning invalid records.
- Removing duplicates.
- Handling NULL values.
- Joining datasets.
- Aggregating data.
- Applying business rules.
- Creating analytical tables.

Common technologies:

- Apache Spark.
- PySpark.
- Databricks.
- SQL engines.

### 3.6 Serving and Consumption Layer

The serving layer makes processed data available to users and applications.

Consumers include:

- Business analysts.
- Data analysts.
- Data scientists.
- BI tools.
- Machine learning applications.
- Downstream data pipelines.

Examples of consumption tools include Power BI, Tableau, and SQL analytics engines.

### 3.7 Governance and Metadata Layer

Governance helps manage and control data across the lakehouse.

Important capabilities:

- Access control.
- Data discovery.
- Data lineage.
- Auditing.
- Metadata management.
- Data ownership.
- Data quality monitoring.

Example: Unity Catalog in Databricks provides centralized governance capabilities for supported data and assets.

---

## 4. Medallion Architecture in a Lakehouse

A common way to organize lakehouse data is the **Medallion Architecture**.

It divides data into three logical layers:

1. Bronze.
2. Silver.
3. Gold.

### Architecture

```text
           Source Systems
                 |
                 v
          +--------------+
          | Bronze Layer |
          |  Raw Data    |
          +--------------+
                 |
                 v
          +--------------+
          | Silver Layer |
          | Cleaned Data |
          +--------------+
                 |
                 v
          +--------------+
          |  Gold Layer  |
          | Business Data|
          +--------------+
                 |
          +------+------+
          |             |
          v             v
       BI Reports     ML Models
```

### Bronze Layer

Stores raw or minimally processed data from source systems.

Characteristics:

- Preserves source information.
- Often retains ingestion metadata.
- Supports reprocessing.
- May contain duplicates or invalid records.
- Commonly stores data in Delta or another supported table format.

Example:

```text
bronze_orders
bronze_customers
bronze_payments
```

### Silver Layer

Stores cleaned, validated, and standardized data.

Typical operations:

- Remove or handle duplicates.
- Validate schemas.
- Handle NULL values.
- Standardize data types.
- Apply business rules.
- Join related datasets.

Example:

```text
silver_orders
silver_customers
silver_payments
```

### Gold Layer

Stores business-ready datasets for reporting and analytics.

Typical operations:

- Aggregation.
- Dimensional modeling.
- Business calculations.
- Fact and dimension table creation.
- Reporting-specific transformations.

Example:

```text
gold_daily_sales
gold_customer_summary
gold_product_performance
```

### Important Points

- Bronze, Silver, and Gold are logical data layers.
- They do not necessarily require separate physical storage systems.
- All three layers can use Delta tables.
- Invalid records can be sent to quarantine or rejected-data locations.
- Not every dataset must pass through every layer.
- Gold data can be consumed by BI tools, applications, or machine learning workloads.

---

## 5. How Data Flows Through a Lakehouse

Consider an e-commerce company processing daily orders.

### Step 1: Ingestion

The company receives order data from an operational database.

Example:

```text
order_id | customer_id | amount | order_date
101      | C01         | 500    | 2026-10-01
102      | C02         | 700    | 2026-10-01
```

Data is ingested into the Bronze layer.

### Step 2: Bronze Storage

The source data is stored with minimal transformation.

```text
bronze_orders
```

The company may also preserve source file names, ingestion timestamps, and batch identifiers.

### Step 3: Silver Transformation

The data engineer:

- Validates the schema.
- Checks required columns.
- Handles invalid amounts.
- Removes duplicate business records.
- Standardizes date and numeric types.

The cleaned data is stored in:

```text
silver_orders
```

### Step 4: Gold Transformation

The company calculates daily revenue.

```sql
SELECT
    order_date,
    SUM(amount) AS total_revenue,
    COUNT(*) AS total_orders
FROM silver_orders
GROUP BY order_date;
```

The result is stored in:

```text
gold_daily_sales
```

### Step 5: Consumption

Business users query the Gold table through SQL or connect it to a BI tool.

```sql
SELECT *
FROM gold_daily_sales
ORDER BY order_date;
```

This provides business-ready information without requiring analysts to repeatedly process raw source data.

---

## 6. Role of Delta Lake in Lakehouse Architecture

Delta Lake is one of the commonly used technologies for implementing a lakehouse.

It uses Parquet data files and a transaction log to provide reliable table operations.

### Important Features

| Feature | Purpose |
|---|---|
| ACID transactions | Provides reliable transactional table operations |
| Schema enforcement | Checks incoming data against the table schema |
| Schema evolution | Supports compatible schema changes |
| Time Travel | Allows access to retained historical table versions |
| MERGE | Supports insert and update patterns |
| UPDATE | Modifies matching records |
| DELETE | Deletes matching records logically |
| OPTIMIZE | Reorganizes data files to improve layout |
| Change Data Feed | Exposes supported row-level changes |

### Simple Example

Writing a Delta table using PySpark:

```python
df.write \
    .format("delta") \
    .mode("append") \
    .save("/lakehouse/silver/orders")
```

Reading the Delta table:

```python
orders_df = spark.read \
    .format("delta") \
    .load("/lakehouse/silver/orders")
```

Creating a managed Delta table using SQL:

```sql
CREATE TABLE silver_orders
USING DELTA
AS
SELECT *
FROM bronze_orders;
```

The exact storage location and management behavior depend on the table type and catalog configuration.

**Remember:** Delta Lake provides table reliability and management capabilities. It does not replace the underlying cloud storage service.

---

## 7. Lakehouse vs Data Lake vs Data Warehouse

| Feature | Data Lake | Data Warehouse | Data Lakehouse |
|---|---|---|---|
| Main purpose | Flexible data storage | Analytical reporting | Combined data and analytics platform |
| Data types | Multiple types | Primarily structured | Multiple types |
| Storage | Object or distributed storage | Warehouse-managed storage | Commonly object storage |
| Schema | Often schema-on-read | Generally schema-on-write | Supports flexible ingestion and managed table schemas |
| ACID transactions | Not inherent to ordinary files | Commonly supported | Supported by suitable table formats |
| SQL analytics | Supported through query engines | Core capability | Supported through SQL engines |
| Machine learning | Common use case | Possible | Common use case |
| Data governance | Requires suitable tools | Platform-dependent | Requires suitable governance capabilities |
| Data duplication | Depends on architecture | May receive copied datasets | Can reduce movement through shared storage |

### Key Difference

- **Data Lake:** Focuses on flexible storage.
- **Data Warehouse:** Focuses on structured analytical workloads.
- **Data Lakehouse:** Combines flexible lake storage with reliable analytical table capabilities.

A lakehouse does not automatically provide every feature of every warehouse. Capabilities depend on the storage format, processing engine, SQL engine, and governance platform.

---

## 8. Lakehouse vs Traditional Data Warehouse Architecture

### Traditional Architecture

```text
Source Systems
      |
      v
Data Ingestion
      |
      v
   Data Lake
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

In this design, data may be copied from the lake into a separate warehouse.

### Lakehouse Architecture

```text
Source Systems
      |
      v
Data Ingestion
      |
      v
Cloud Object Storage
      |
      v
Delta / Iceberg / Hudi Tables
      |
      +----------------+
      |                |
      v                v
 SQL Analytics    Spark Processing
      |                |
      v                v
 BI / Reports       ML Workloads
```

The lakehouse aims to serve multiple workloads using shared data storage and table metadata.

---

## 9. Advantages of Lakehouse Architecture

### 1. Flexible Data Storage

Can store structured, semi-structured, and unstructured data.

### 2. Reduced Data Movement

Multiple workloads can access shared datasets, potentially reducing unnecessary copies.

### 3. Scalability

Cloud object storage can scale to large data volumes.

### 4. Reliable Table Operations

Supported table formats provide transactions and consistent table snapshots.

### 5. Multiple Workloads

Supports analytics, reporting, data engineering, and machine learning.

### 6. Schema Management

Supports schema enforcement and controlled schema evolution.

### 7. Governance

Can provide centralized access control, lineage, and auditing through suitable governance tools.

---

## 10. Challenges of Lakehouse Architecture

A lakehouse also introduces design and operational challenges.

| Challenge | Explanation |
|---|---|
| Performance tuning | File layout, partitioning, indexing-like features, and query plans need attention |
| Small files | Frequent writes can create many small files |
| Data quality | Raw and evolving data require validation |
| Governance | Permissions and ownership must be managed carefully |
| Schema evolution | Incompatible changes can break downstream consumers |
| Cost management | Storage, compute, and data processing all have costs |
| Concurrency | Conflicting writes may need retry and conflict handling |
| Platform dependency | Features differ across engines and table formats |

A lakehouse is not automatically fast, cheap, or well-governed. These properties depend on implementation and operations.

---

## 11. Real-World Example: Azure Databricks Lakehouse

Consider an organization building an analytics platform using Azure.

### Technology Stack

| Layer | Technology | Responsibility |
|---|---|---|
| Data Sources | SQL databases, APIs, files | Generate source data |
| Ingestion | Azure Data Factory, Auto Loader | Bring data into the platform |
| Storage | ADLS Gen2 | Store data files |
| Processing | Azure Databricks, PySpark | Transform data |
| Table Format | Delta Lake | Manage analytical tables |
| Governance | Unity Catalog | Manage access and metadata |
| Consumption | Databricks SQL, Power BI | Analytics and reporting |

### Architecture

```text
SQL Database / APIs / Files
             |
             v
     Azure Data Factory
       / Auto Loader
             |
             v
       ADLS Gen2
             |
             v
      Bronze Delta Tables
             |
             v
      Databricks PySpark
             |
             v
      Silver Delta Tables
             |
             v
      Databricks PySpark
             |
             v
       Gold Delta Tables
             |
             v
     Databricks SQL / Power BI
```

### Data Engineer's Responsibilities

A data engineer may be responsible for:

1. Building ingestion pipelines.
2. Loading source data into Bronze.
3. Applying transformations using PySpark and SQL.
4. Validating and cleaning Silver data.
5. Creating Gold analytical tables.
6. Implementing incremental loads and CDC.
7. Handling failures and retries.
8. Optimizing queries and file layouts.
9. Managing schemas and data quality.
10. Applying governance and monitoring pipeline health.

---

## 12. Common Interview Mistakes

| Incorrect Statement | Correct Understanding |
|---|---|
| A lakehouse is simply a data lake | It adds table-management and analytical capabilities |
| Delta Lake is the storage service | It manages tables over underlying storage |
| Parquet provides ACID transactions | Parquet is a file format; table formats add transaction management |
| Bronze, Silver, Gold must be separate storage systems | They are logical layers |
| Every lakehouse must use Delta Lake | Iceberg and Hudi are also options |
| A lakehouse eliminates all data duplication | It can reduce unnecessary movement, but duplication may still exist |
| A lakehouse automatically guarantees data quality | Data quality rules and monitoring must be implemented |
| All lakehouse engines support identical features | Capabilities differ by engine, format, and version |

---

## 13. Interview Questions and Answers

### Q1. What is a data lakehouse?

A data lakehouse combines flexible data lake storage with capabilities such as ACID transactions, schema management, and analytical querying.

### Q2. Why was the lakehouse architecture introduced?

Traditional architectures often maintain separate lakes and warehouses, which can lead to data duplication, movement, and operational complexity. A lakehouse aims to support multiple workloads over shared storage.

### Q3. What are the main components of a lakehouse?

The main components are data sources, ingestion, storage, table formats, processing, serving, and governance.

### Q4. What is the role of object storage?

Object storage holds the underlying data files. Examples include ADLS Gen2, Amazon S3, and Google Cloud Storage.

### Q5. What is the role of Delta Lake?

Delta Lake manages analytical tables using Parquet data files and a transaction log. It supports ACID transactions, schema management, Time Travel, and table modification operations.

### Q6. How does the Medallion Architecture fit into a lakehouse?

It organizes data into Bronze, Silver, and Gold layers. Bronze contains raw data, Silver contains cleaned and standardized data, and Gold contains business-ready datasets.

### Q7. Can a lakehouse support streaming?

Yes. It can ingest and process streaming data using technologies such as Spark Structured Streaming, Kafka, and cloud event services. Support depends on the selected source, sink, and platform.

### Q8. What is the difference between a lakehouse and a data warehouse?

A data warehouse focuses on structured analytical workloads. A lakehouse uses flexible lake storage with additional table-management and analytics capabilities to support multiple workloads.

### Q9. What is the role of Unity Catalog?

Unity Catalog provides centralized governance capabilities, including access control, data discovery, lineage, and auditing for supported assets in Databricks.

### Q10. Does a lakehouse replace a data warehouse?

Not necessarily. Some organizations use a lakehouse for most analytical workloads, while others continue using a separate warehouse for specific performance, operational, or business requirements.

### Q11. What is the difference between a file format and a table format?

A file format defines how data is encoded and stored in files. A table format manages metadata, snapshots, transactions, and the collection of files that make up a logical table.

Example:
- Parquet: File format.
- Delta Lake: Table format and storage framework.
- Apache Iceberg: Table format.
- Apache Hudi: Table format.

### Q12. How would you design a basic lakehouse for an e-commerce company?

I would ingest data from operational databases, APIs, and event streams into cloud object storage. I would organize the data into Bronze, Silver, and Gold layers, use a supported table format such as Delta Lake, transform the data using PySpark or SQL, apply quality checks, and expose Gold tables to BI tools. I would also implement incremental ingestion, monitoring, retries, and governance.

---

## 14. Quick Revision

| Term | Meaning |
|---|---|
| Lakehouse | Data lake storage with reliable analytical table capabilities |
| Object Storage | Stores underlying data files |
| Table Format | Manages logical tables, metadata, and transactions |
| Delta Lake | Table framework using Parquet and a transaction log |
| Iceberg | Open table format with snapshot and metadata management |
| Hudi | Open table format supporting upserts and incremental data processing |
| Bronze | Raw or minimally processed data |
| Silver | Cleaned and standardized data |
| Gold | Business-ready data |
| Ingestion | Bringing source data into the platform |
| Processing | Transforming data |
| Governance | Managing access, metadata, and data controls |
| Serving | Making data available to consumers |

### One-Minute Interview Answer

A data lakehouse is an architecture that combines the flexibility and scalability of a data lake with the reliability and analytical capabilities of a data warehouse.

It commonly uses cloud object storage for underlying data and table formats such as Delta Lake, Iceberg, or Hudi to provide features like ACID transactions, schema management, and historical table versions.

A common implementation follows the Medallion Architecture. Data is ingested into Bronze, cleaned and standardized in Silver, and transformed into business-ready datasets in Gold. Processing engines such as Apache Spark handle transformations, while SQL engines and BI tools consume the curated data.

The main advantage is that multiple workloads, including data engineering, analytics, and machine learning, can work with a shared data foundation.

---

## 15. Self-Assessment

- [ ] Define data lakehouse architecture.
- [ ] Explain why lakehouse architecture is useful.
- [ ] Identify its main components.
- [ ] Explain the role of cloud object storage.
- [ ] Explain the difference between a file format and a table format.
- [ ] Describe the role of Delta Lake.
- [ ] Explain Bronze, Silver, and Gold.
- [ ] Describe the flow of data through a lakehouse.
- [ ] Compare a lakehouse with a data lake and data warehouse.
- [ ] Explain the Azure Databricks lakehouse stack.
- [ ] Identify lakehouse challenges.
- [ ] Explain how a lakehouse supports batch and streaming workloads.
- [ ] Design a basic e-commerce lakehouse architecture.

**Interview readiness:** Be able to draw a lakehouse architecture from source systems to ingestion, cloud storage, Bronze/Silver/Gold tables, processing, governance, and BI consumption. Explain the responsibility of each component without confusing storage, processing, and table formats.