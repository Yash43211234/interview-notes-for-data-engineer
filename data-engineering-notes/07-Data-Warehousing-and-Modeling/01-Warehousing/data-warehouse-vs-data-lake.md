# Data Warehouse vs Data Lake

## 1. Introduction

A **Data Warehouse** and a **Data Lake** are two common approaches for storing and managing data in data engineering.

Both store data for analysis, but they differ in how data is stored, processed, organized, and consumed.

- **Data Warehouse:** Stores structured, cleaned, and organized data for reporting and business analytics.
- **Data Lake:** Stores large amounts of raw and processed data in different formats, including structured, semi-structured, and unstructured data.

### Simple Example

Suppose an e-commerce company collects:

- Customer information.
- Orders and payments.
- Website clickstream events.
- Product images.
- Application logs.

A data lake can store all these different types of data.

A data warehouse can store structured, business-ready information such as:

- Total sales by month.
- Revenue by product.
- Customer purchase history.
- Regional sales performance.

**Key idea:** A data lake provides flexible storage for different types of data, while a data warehouse provides organized data for analytical queries and reporting.

---

## 2. What Is a Data Warehouse?

A **Data Warehouse (DWH)** is a centralized data storage system designed for analytical queries, reporting, and business intelligence.

Data is typically collected from multiple sources, cleaned, transformed, and organized before being loaded into the warehouse.

### Characteristics

- Primarily stores structured, analytical data.
- Uses defined schemas and data models.
- Supports complex SQL queries.
- Optimized for reporting and analytics.
- Maintains historical data.
- Often uses dimensional modeling.
- Supports business intelligence tools.

### Example Architecture

```text
Operational Databases
       |
       v
   ETL / ELT
       |
       v
 Data Warehouse
       |
       v
 Data Marts / Views
       |
       v
 BI Dashboards
```

### Example

An e-commerce company may have these warehouse tables:

**Customers**

| customer_id | customer_name | city |
|---|---|---|
| C01 | Rahul | Kolkata |
| C02 | Priya | Delhi |

**Orders**

| order_id | customer_id | amount |
|---|---|---:|
| 101 | C01 | 500 |
| 102 | C02 | 700 |

Business analysts can query these tables to calculate revenue, customer activity, and sales trends.

### Common Data Warehouse Technologies

- <Entity value="Snowflake" category="software" disambig="Cloud data warehouse platform"/>
- <Entity value="Amazon Redshift" category="software" disambig="Cloud data warehouse service"/>
- <Entity value="Google BigQuery" category="software" disambig="Cloud data warehouse and analytics platform"/>
- <Entity value="Azure Synapse Analytics" category="software" disambig="Microsoft analytics platform"/>
- <Entity value="Teradata" category="software" disambig="Enterprise data warehouse platform"/>

---

## 3. What Is a Data Lake?

A **Data Lake** is a centralized storage system that can store large amounts of data in its original or processed form.

It supports different data types and formats, including:

- Structured data: CSV, relational tables.
- Semi-structured data: JSON, XML, logs.
- Unstructured data: Images, audio, video, documents.

Data lakes commonly use cloud object storage or distributed file systems.

### Characteristics

- Stores raw and processed data.
- Supports multiple data formats.
- Uses flexible schema-on-read approaches.
- Scales to large data volumes.
- Supports batch and streaming ingestion.
- Can support analytics, machine learning, and data processing.
- Separates storage from compute in many cloud architectures.

### Example Architecture

```text
Databases
CSV / JSON Files
Application Logs
Images / Videos
Streaming Events
       |
       v
 Data Ingestion
       |
       v
    Data Lake
       |
       v
 Processing Engines
       |
       v
 Cleaned / Transformed Data
       |
       v
 Analytics / ML / Reporting
```

### Example

An e-commerce company stores:

| Data | Format |
|---|---|
| Customer records | CSV |
| Order events | JSON |
| Application logs | Text / JSON |
| Product images | JPEG / PNG |
| Clickstream events | JSON / Avro |
| Processed datasets | Parquet |

The data lake can preserve the original data and provide processed datasets for downstream use.

### Common Data Lake Technologies

- <Entity value="Amazon S3" category="software" disambig="Cloud object storage service"/>
- <Entity value="Azure Data Lake Storage Gen2" category="software" disambig="Microsoft cloud data lake storage"/>
- <Entity value="Google Cloud Storage" category="software" disambig="Google cloud object storage"/>
- <Entity value="Apache Hadoop HDFS" category="software" disambig="Distributed file system"/>
- <Entity value="Apache Spark" category="software" disambig="Distributed data processing engine"/>

---

## 4. Data Warehouse vs Data Lake

| Feature | Data Warehouse | Data Lake |
|---|---|---|
| Main purpose | Reporting and analytics | Flexible data storage and processing |
| Data types | Primarily structured | Structured, semi-structured, unstructured |
| Data condition | Usually cleaned and modeled | Raw and processed |
| Schema approach | Usually schema-on-write | Often schema-on-read |
| Data processing | ETL or ELT | Often ELT and distributed processing |
| Storage | Warehouse-managed storage | Commonly object storage or distributed storage |
| Flexibility | More controlled structure | More flexible formats |
| Querying | SQL and analytical queries | SQL, Spark, and other processing tools |
| Users | Analysts, BI teams, business users | Data engineers, data scientists, analysts |
| Typical workloads | BI, dashboards, reporting | Data processing, ML, exploration, analytics |

### Important Differences

**1. Schema-on-Write vs Schema-on-Read**

- Warehouse: Data is generally modeled and validated before being stored in its analytical structure.
- Lake: Raw data can be stored first, with structure applied when it is read or processed.

These are common design patterns, not absolute rules. Modern platforms can support both approaches.

**2. Data Storage**

- Warehouse: Data is managed in analytical tables and storage structures.
- Lake: Data is commonly stored as files in object storage.

**3. Data Processing**

- Warehouse: Often uses SQL-based transformations and analytical engines.
- Lake: Commonly uses Spark or other distributed processing engines, along with SQL.

**4. Data Flexibility**

- Warehouse: More structured and governed for defined analytical use cases.
- Lake: More flexible for storing different formats and exploring new datasets.

---

## 5. Schema-on-Write vs Schema-on-Read

This is one of the most important differences to understand.

### Schema-on-Write

Schema is defined and applied when data is written into the target structure.

Commonly associated with data warehouses.

Example:

```text
Source Data
    |
    v
Validate Schema
    |
    v
Transform Data
    |
    v
Write into Warehouse
```

The target expects defined columns and data types.

Example:

```sql
CREATE TABLE orders (
    order_id INT,
    customer_id INT,
    amount DECIMAL(12,2),
    order_date DATE
);
```

Incoming data must be compatible with the target schema and applicable constraints.

### Schema-on-Read

Data is stored first, and its structure is interpreted when it is read or processed.

Commonly associated with data lakes.

Example:

```text
Raw JSON / CSV / Logs
        |
        v
    Data Lake
        |
        v
Apply Schema During Read
        |
        v
   Transformation
```

PySpark example:

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    StringType,
    DoubleType
)

schema = StructType([
    StructField("order_id", StringType(), True),
    StructField("customer_id", StringType(), True),
    StructField("amount", DoubleType(), True)
])

df = spark.read.schema(schema).json(
    "/data/orders/"
)
```

The schema is applied when the data is read.

### Comparison

| Schema-on-Write | Schema-on-Read |
|---|---|
| Schema applied during writing | Schema interpreted during reading |
| Common in warehouses | Common in data lakes |
| More controlled structure | More flexible ingestion |
| Useful for consistent reporting | Useful for raw-data exploration |

---

## 6. Data Lake vs Data Warehouse: Practical Example

Consider an e-commerce company.

It collects data from:
- Online orders.
- Customer databases.
- Website activity.
- Payment systems.
- Product images.
- Application logs.

### Data Lake

The company stores raw source data in a lake.

```text
/data-lake/
    bronze/
        orders/
        customers/
        payments/
        clickstream/
        application-logs/
        product-images/
```

This provides flexible storage for different data types.

### Data Warehouse

The company transforms relevant data into analytical tables.

```text
data_warehouse/
    dim_customer
    dim_product
    dim_date
    fact_orders
    fact_payments
```

Business analysts can use these tables to calculate:
- Monthly revenue.
- Product performance.
- Customer lifetime value.
- Regional sales.
- Payment trends.

### Simple Flow

```text
Multiple Source Systems
          |
          v
       Data Lake
     (Raw Data)
          |
          v
 Data Transformation
          |
          v
  Curated / Cleaned Data
          |
          v
    Data Warehouse
          |
          v
    BI Dashboards
```

This is one common architecture. A data lake does not always need a separate warehouse because modern lakehouse platforms can provide analytical table capabilities directly on lake storage.

---

## 7. What Is a Data Lakehouse?

A **Data Lakehouse** combines flexible data lake storage with capabilities commonly associated with data warehouses.

It aims to support:
- Large-scale and flexible data storage.
- Structured analytical tables.
- ACID transactions.
- Schema enforcement and evolution.
- SQL analytics.
- Batch and streaming workloads.
- Data governance.

### Traditional Architecture

```text
Data Lake
    |
    v
Processing
    |
    v
Separate Data Warehouse
    |
    v
BI Reporting
```

### Lakehouse Architecture

```text
Sources
   |
   v
Data Lake Storage
   |
   v
Delta / Iceberg Tables
   |
   v
SQL and Distributed Processing
   |
   v
Analytics / BI / ML
```

### Common Lakehouse Technologies

- <Entity value="Delta Lake" category="software" disambig="Open-source lakehouse table format"/>
- <Entity value="Apache Iceberg" category="software" disambig="Open table format for analytics"/>
- <Entity value="Apache Hudi" category="software" disambig="Open data lake table format"/>
- <Entity value="Databricks" category="software" disambig="Data and AI platform"/>

### Role of Delta Lake

Delta Lake stores data in Parquet files and maintains a transaction log.

It adds capabilities such as:
- ACID transactions.
- Schema enforcement.
- Schema evolution.
- Time Travel.
- MERGE, UPDATE, and DELETE operations.

This makes a data lake more suitable for reliable analytical table workloads.

**Remember:** Delta Lake is a table format and storage framework built on data lake storage. It is not itself a data warehouse.

---

## 8. Data Lake vs Data Lakehouse vs Data Warehouse

| Feature | Data Lake | Data Warehouse | Data Lakehouse |
|---|---|---|---|
| Storage | Flexible file/object storage | Analytical storage | Lake storage with table capabilities |
| Data types | Multiple types | Primarily structured | Multiple types |
| Schema | Often applied during reading | Generally defined for target tables | Supports defined table schemas and flexibility |
| ACID transactions | Not inherent to ordinary files | Commonly supported | Supported by table formats such as Delta, Iceberg, and Hudi |
| Data modeling | Varies | Usually strongly modeled | Supports analytical data models |
| BI analytics | Possible with suitable engines | Core use case | Supported |
| Machine learning | Common use case | Possible | Common use case |
| Data governance | Requires suitable governance tools | Usually built into platform | Requires suitable governance capabilities |

No architecture is automatically best for every workload. The right choice depends on cost, performance, governance, workload, and existing platform capabilities.

---

## 9. When Should You Use Each?

### Use a Data Warehouse When:

- Business reporting is the primary requirement.
- Data is mostly structured.
- Consistent business definitions are important.
- SQL analytics and BI workloads dominate.
- Users need curated datasets.

Example: Monthly revenue reporting for business management.

### Use a Data Lake When:

- Data comes from many different sources.
- You need to preserve raw source data.
- Data includes JSON, logs, images, or other formats.
- Large-scale processing is required.
- Data science and machine learning workloads are important.

Example: Storing website clickstream events and application logs for future analysis.

### Use a Data Lakehouse When:

- You want flexible lake storage with reliable analytical tables.
- Multiple workloads need access to shared data.
- You need transactions, schema management, and SQL analytics.
- You want to reduce unnecessary data movement between separate systems.

Example: A Databricks platform using Delta Lake for Bronze, Silver, and Gold tables.

---

## 10. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| A data lake stores only raw data | It can store raw and processed data |
| A warehouse stores only numbers | It stores structured analytical data of many kinds |
| Data lakes cannot support SQL | SQL engines can query supported lake data |
| Schema-on-read means no schema exists | A schema is interpreted when data is read |
| Every warehouse uses only ETL | Warehouses can also use ELT |
| A data lake is automatically a lakehouse | Lakehouse capabilities require additional table and governance features |
| Parquet is a data lake | Parquet is a columnar file format often used in data lakes |
| Delta Lake is a data warehouse | Delta Lake adds table capabilities to lake storage |
| A lakehouse always replaces a warehouse | Architecture choices depend on workload and platform |

---

## 11. Interview Questions and Answers

### Q1. What is a data warehouse?

A data warehouse is a centralized analytical system that stores structured, organized data for reporting, business intelligence, and analytical queries.

### Q2. What is a data lake?

A data lake is a storage system that holds large amounts of data in different formats, including structured, semi-structured, and unstructured data.

### Q3. What is the main difference between a data lake and a data warehouse?

A data lake provides flexible storage for different types of raw and processed data. A data warehouse stores organized analytical data optimized for reporting and business queries.

### Q4. What is schema-on-write?

Schema-on-write means the target schema is defined and applied when data is written. It is commonly associated with data warehouse workloads.

### Q5. What is schema-on-read?

Schema-on-read means data can be stored first and its structure interpreted when it is read or processed. It is commonly associated with data lake workloads.

### Q6. Can a data lake store structured data?

Yes. It can store structured data such as CSV, Parquet, and relational extracts, along with semi-structured and unstructured data.

### Q7. What is a data lakehouse?

A data lakehouse combines flexible data lake storage with table-management, transaction, governance, and analytical capabilities commonly associated with warehouses.

### Q8. What is the role of Delta Lake?

Delta Lake adds features such as ACID transactions, schema enforcement, schema evolution, and Time Travel to data stored in Parquet files on lake storage.

### Q9. Can a data lake be used for reporting?

Yes. SQL engines and analytical tools can query supported data lake formats. A lakehouse can provide additional reliability and management features.

### Q10. Why do companies use both a data lake and a data warehouse?

Some companies use a lake for flexible storage and large-scale processing, then publish curated datasets to a warehouse for business reporting. Other companies use a lakehouse to support both workloads on shared storage.

### Q11. Which is better: a data lake or a data warehouse?

Neither is universally better. A data lake is useful for flexible storage and varied data workloads, while a warehouse is useful for structured analytical reporting. The choice depends on business requirements.

### Q12. What is the difference between Parquet and Delta Lake?

Parquet is a columnar file format. Delta Lake is a table format and storage framework that uses Parquet data files with a transaction log to provide additional table capabilities.

---

## 12. Quick Revision

| Concept | Remember |
|---|---|
| Data Warehouse | Organized analytical data |
| Data Lake | Flexible storage for different data types |
| Schema-on-Write | Apply schema during writing |
| Schema-on-Read | Interpret schema during reading |
| Data Lakehouse | Lake storage with reliable table and analytics capabilities |
| Parquet | Columnar file format |
| Delta Lake | Table framework using Parquet and transaction log |
| ETL | Extract, Transform, Load |
| ELT | Extract, Load, Transform |
| BI | Business Intelligence |

### One-Minute Interview Answer

A data warehouse and a data lake are two common data storage architectures.

A data warehouse stores structured and organized data for analytical queries, reporting, and business intelligence. A data lake stores raw and processed data in different formats, including structured, semi-structured, and unstructured data.

A warehouse commonly follows a schema-on-write approach, while a lake commonly uses schema-on-read. For example, an e-commerce company can store orders, clickstream events, application logs, and product images in a data lake. It can then transform the relevant data into analytical tables for revenue and customer reporting.

A modern alternative is a data lakehouse, which combines flexible lake storage with capabilities such as ACID transactions, schema management, and SQL analytics using technologies such as Delta Lake.

---

## 13. Self-Assessment

- [ ] Define a data warehouse.
- [ ] Define a data lake.
- [ ] Explain their main differences.
- [ ] Understand schema-on-write.
- [ ] Understand schema-on-read.
- [ ] Identify common data warehouse technologies.
- [ ] Identify common data lake storage technologies.
- [ ] Explain the role of Parquet.
- [ ] Explain the role of Delta Lake.
- [ ] Define a data lakehouse.
- [ ] Compare lake, warehouse, and lakehouse.
- [ ] Explain when to use each architecture.
- [ ] Describe an e-commerce data architecture.

**Interview readiness:** Be able to design a simple architecture where an e-commerce company ingests orders, customer records, clickstream events, and application logs into a data lake, transforms them into curated analytical tables, and makes business metrics available through dashboards.