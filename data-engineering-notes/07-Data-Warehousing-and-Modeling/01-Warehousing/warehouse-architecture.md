# Data Warehouse Architecture

## 1. What Is Data Warehouse Architecture?

**Data Warehouse Architecture** describes how data moves from different source systems into a centralized analytical environment, where it is integrated, transformed, stored, and made available for reporting and analysis.

It defines the components, layers, and processes involved in building and maintaining a data warehouse.

### Main Objectives

- Collect data from multiple sources.
- Integrate data into a consistent structure.
- Clean and transform source data.
- Store historical and analytical data.
- Support complex SQL queries.
- Provide reliable data for BI and reporting.
- Maintain data quality, security, and governance.

### Simple Architecture

```text
       Data Sources
            |
            v
      Data Ingestion
            |
            v
   Staging / Raw Storage
            |
            v
   ETL / ELT Processing
            |
            v
     Data Warehouse
            |
            v
      Data Marts
            |
            v
     BI / Reporting
```

**Key idea:** A data warehouse architecture transforms data from operational sources into reliable, business-ready information.

---

## 2. Main Components of Data Warehouse Architecture

A typical data warehouse contains the following components.

### 2.1 Data Sources

Data sources are the systems from which data is collected.

Examples:

- Relational databases.
- CRM systems.
- ERP systems.
- APIs.
- CSV and Excel files.
- Application logs.
- External business systems.

Example:

An e-commerce company may collect data from:

| Source | Data |
|---|---|
| Customer database | Customer details |
| Order database | Orders and order amounts |
| Payment system | Payment transactions |
| Inventory system | Product stock |
| CRM | Customer interactions |
| Website | Clickstream events |

Data engineers bring these different datasets into a common analytical environment.

### 2.2 Data Ingestion Layer

The ingestion layer extracts or receives data from source systems.

Common ingestion approaches:

- Full data extraction.
- Incremental extraction.
- Change Data Capture (CDC).
- Batch ingestion.
- Streaming ingestion.
- File-based ingestion.

Common technologies:

- Azure Data Factory.
- Apache Kafka.
- Azure Event Hubs.
- Databricks Auto Loader.
- Apache Spark.

The ingestion method depends on data volume, latency requirements, source capabilities, and business needs.

### 2.3 Staging Area

A **Staging Area** is an intermediate storage location where incoming data is temporarily stored before it is loaded into the warehouse.

It helps separate source extraction from warehouse transformation and loading.

Typical activities:

- Store extracted source data.
- Validate basic structure.
- Handle initial data type conversions.
- Identify invalid records.
- Prepare data for transformation.

Example:

```text
Source Database
       |
       v
   Staging Area
       |
       v
 Data Validation
       |
       v
 Data Transformation
       |
       v
 Data Warehouse
```

**Important:** A staging area may be temporary or persistent. Some modern ELT architectures use cloud object storage or raw tables instead of a traditional database staging area.

### 2.4 ETL / ELT Processing Layer

This layer transforms source data into a consistent structure.

**ETL: Extract, Transform, Load**

```text
Extract
   |
   v
Transform
   |
   v
Load
```

Data is transformed before loading into the target warehouse.

**ELT: Extract, Load, Transform**

```text
Extract
   |
   v
Load
   |
   v
Transform
```

Raw or lightly processed data is loaded first, and transformations are performed inside the target platform.

Common transformations include:

- Data cleaning.
- Deduplication.
- NULL handling.
- Data type conversion.
- Joining datasets.
- Aggregation.
- Business rule implementation.
- Fact and dimension creation.

### 2.5 Data Warehouse Storage Layer

This is the central analytical storage system.

It contains integrated, structured, and organized data for analytical workloads.

Typical characteristics:

- Stores historical data.
- Supports analytical queries.
- Integrates multiple source systems.
- Maintains consistent business definitions.
- Supports fact and dimension tables.
- Optimized for reporting and analysis.

Example:

```text
Data Warehouse
      |
      +-- dim_customer
      |
      +-- dim_product
      |
      +-- dim_date
      |
      +-- fact_orders
      |
      +-- fact_payments
```

### 2.6 Data Mart Layer

A **Data Mart** is a subject-oriented subset of data designed for a particular department or business function.

Examples:

- Sales Data Mart.
- Finance Data Mart.
- Marketing Data Mart.
- Customer Data Mart.

Example:

```text
             Data Warehouse
                    |
          +---------+---------+
          |         |         |
          v         v         v
        Sales     Finance   Marketing
        Mart       Mart       Mart
```

A data mart provides users with focused datasets relevant to their work.

### 2.7 Presentation and BI Layer

The presentation layer makes analytical data available to business users.

Common consumers:

- Business analysts.
- Data analysts.
- Managers.
- Executives.
- Data scientists.

Common tools:

- Power BI.
- Tableau.
- Looker.
- SQL query interfaces.

Users can create dashboards, reports, and analytical queries using curated warehouse data.

### 2.8 Metadata and Governance Layer

Metadata describes the data and helps manage the warehouse.

Examples:

- Table and column definitions.
- Data types.
- Source information.
- Data lineage.
- Ownership.
- Refresh schedules.
- Access permissions.

Governance capabilities include:

- Data access control.
- Data security.
- Auditing.
- Data quality.
- Compliance.
- Data lineage.

These capabilities support the warehouse across its different layers.

---

## 3. Three-Tier Data Warehouse Architecture

The three-tier architecture is a common conceptual model for understanding data warehouse systems.

It divides the architecture into three layers:

1. Bottom Tier.
2. Middle Tier.
3. Top Tier.

### Architecture Diagram

```text
          TOP TIER
   +----------------------+
   | BI / Reporting Tools |
   | Dashboards / Queries |
   +----------------------+
              |
              v
         MIDDLE TIER
   +----------------------+
   | OLAP / Query Engine  |
   | Analytical Processing|
   +----------------------+
              |
              v
         BOTTOM TIER
   +----------------------+
   | Data Warehouse       |
   | Staging / ETL / Data |
   | Integration          |
   +----------------------+
              |
              v
        Source Systems
```

### 3.1 Bottom Tier

The bottom tier contains the data warehouse database and the processes that load data into it.

Responsibilities:

- Extract data from sources.
- Integrate data.
- Clean and transform data.
- Load warehouse tables.
- Maintain historical records.

### 3.2 Middle Tier

The middle tier contains the OLAP or analytical processing layer.

It provides analytical access to warehouse data.

Responsibilities:

- Execute analytical queries.
- Perform aggregations.
- Support multidimensional analysis.
- Provide data access to BI tools.

Depending on the architecture, OLAP processing may be implemented through relational, multidimensional, or other analytical engines.

### 3.3 Top Tier

The top tier contains front-end tools used by business users.

Examples:

- BI dashboards.
- Reporting applications.
- Data visualization tools.
- Analytical query interfaces.

Users interact with this tier to understand business performance.

---

## 4. Types of Data Warehouse Architecture

Data warehouse architecture is also commonly explained through three approaches:

1. Single-Tier Architecture.
2. Two-Tier Architecture.
3. Three-Tier Architecture.

### 4.1 Single-Tier Architecture

A single-tier architecture aims to minimize separation between operational data and analytical access.

In a simplified design, data is accessed through a single integrated layer.

```text
+---------------------------+
| Operational and Analytical|
| Data / Processing         |
+---------------------------+
```

**Characteristics:**

- Minimal architectural separation.
- Can introduce competition between operational and analytical workloads.
- Less common as a large enterprise warehouse design.

### 4.2 Two-Tier Architecture

A two-tier architecture separates the data warehouse from the client or presentation layer.

```text
+-------------------------+
| Client / BI Applications|
+-------------------------+
             |
             v
+-------------------------+
| Data Warehouse / OLAP   |
+-------------------------+
             |
             v
       Data Sources
```

**Characteristics:**

- Separates client applications from warehouse storage and processing.
- Suitable for relatively simple architectures.
- Can face scalability and workload-management limitations depending on implementation.

### 4.3 Three-Tier Architecture

A three-tier architecture separates data storage, analytical processing, and presentation.

```text
+---------------------------+
| Top Tier                  |
| BI / Reports / Dashboards |
+---------------------------+
              |
              v
+---------------------------+
| Middle Tier               |
| OLAP / Query Processing   |
+---------------------------+
              |
              v
+---------------------------+
| Bottom Tier               |
| Warehouse / ETL / Storage |
+---------------------------+
              |
              v
        Data Sources
```

**Characteristics:**

- Clear separation of responsibilities.
- Supports analytical processing.
- Easier to organize large enterprise systems.
- Common conceptual model for warehouse design.

**Interview tip:** Understand the difference between the three-tier conceptual model and the three physical layers of a modern cloud data platform. They describe related but different architectural views.

---

## 5. Enterprise Data Warehouse (EDW)

An **Enterprise Data Warehouse (EDW)** is a centralized warehouse that integrates data across an entire organization.

It provides a consistent view of business information across departments.

### Example

A large company may have separate systems for:

- Sales.
- Finance.
- Human Resources.
- Marketing.
- Customer Support.
- Inventory.

An EDW integrates data from these systems into a common analytical environment.

```text
Sales System --------+
Finance System ------+
HR System -----------+----> Enterprise Data Warehouse
Marketing System ----+
Inventory System ----+
                              |
                 +------------+------------+
                 |            |            |
                 v            v            v
              Sales         Finance      HR
              Reports       Reports      Reports
```

### Characteristics

- Organization-wide data integration.
- Shared business definitions.
- Historical data storage.
- Centralized governance.
- Supports cross-department analytics.

---

## 6. Independent vs Dependent Data Marts

Data marts can be designed in different ways.

### Independent Data Mart

An independent data mart is created directly from operational sources or other source systems without depending on a central enterprise warehouse.

```text
Sources
   |
   +--------> Sales Data Mart
   |
   +--------> Finance Data Mart
   |
   +--------> Marketing Data Mart
```

**Advantages:**

- Can be implemented quickly for a specific department.
- Provides focused data access.

**Challenges:**

- Different departments may define metrics differently.
- Data duplication may increase.
- Integration across departments becomes harder.

### Dependent Data Mart

A dependent data mart receives data from a central data warehouse.

```text
           Source Systems
                 |
                 v
       Enterprise Data Warehouse
                 |
        +--------+--------+
        |        |        |
        v        v        v
      Sales   Finance  Marketing
      Mart     Mart      Mart
```

**Advantages:**

- Consistent business definitions.
- Better enterprise-wide integration.
- Centralized data governance.
- Less duplication of transformation logic.

---

## 7. Data Warehouse Schema Architecture

A data warehouse commonly uses dimensional modeling to organize analytical data.

Two important schema designs are:

### 7.1 Star Schema

A star schema contains a central fact table connected directly to dimension tables.

```text
              dim_customer
                    |
                    |
dim_product --- fact_sales --- dim_date
                    |
                    |
                dim_store
```

**Fact Table:**
- Contains measurable business events.
- Stores numeric measures and dimension keys.

**Dimension Tables:**
- Store descriptive attributes.
- Provide context for analyzing facts.

Example:

`fact_sales`

| sales_id | customer_key | product_key | date_key | amount |
|---|---|---|---|---:|
| 1 | 101 | 201 | 20261001 | 500 |
| 2 | 102 | 202 | 20261002 | 700 |

`dim_customer`

| customer_key | customer_name | city |
|---|---|---|
| 101 | Rahul | Kolkata |
| 102 | Priya | Delhi |

### 7.2 Snowflake Schema

A snowflake schema normalizes dimension tables into additional related tables.

```text
              dim_city
                 |
           dim_customer
                 |
dim_product -- fact_sales -- dim_date
                 |
              dim_store
```

For example, customer and city information may be separated into different tables.

### Star vs Snowflake

| Feature | Star Schema | Snowflake Schema |
|---|---|---|
| Dimension structure | Usually denormalized | More normalized |
| Number of joins | Usually fewer | Usually more |
| Query simplicity | Generally simpler | Can be more complex |
| Data redundancy | May be higher | Often lower |
| Design | Simple | More complex |
| Common use | BI and reporting | When normalized dimensions are useful |

The best choice depends on query patterns, platform capabilities, and data model requirements.

---

## 8. Modern Cloud Data Warehouse Architecture

Modern data warehouses commonly separate storage and compute.

This allows storage capacity and query-processing resources to scale independently in many platforms.

### Example Architecture

```text
         Data Sources
               |
               v
        Data Ingestion
               |
               v
       Cloud Storage Layer
               |
               v
       Warehouse Storage
               |
               v
       Compute / SQL Engine
               |
               v
        BI / Reporting
```

Some platforms manage storage and compute as a unified service, while others expose separate resources.

### Common Cloud Data Warehouse Technologies

| Platform | Service |
|---|---|
| AWS | Amazon Redshift |
| Azure | Azure Synapse Analytics |
| Google Cloud | BigQuery |
| Snowflake | Snowflake Cloud Data Platform |

### Benefits

- Elastic scalability.
- Managed infrastructure.
- High availability features.
- Integration with cloud services.
- Support for large analytical workloads.
- Reduced infrastructure administration in managed services.

---

## 9. Data Warehouse Architecture: End-to-End Example

Consider an e-commerce company building a warehouse.

### Source Systems

The company has:

- MySQL customer database.
- PostgreSQL order database.
- Payment API.
- CSV product files.

### Step 1: Data Ingestion

Data is extracted using batch pipelines, incremental loads, or CDC.

### Step 2: Staging

Incoming data is stored temporarily or in a raw landing area.

Example:

```text
staging_customers
staging_orders
staging_payments
staging_products
```

### Step 3: Data Transformation

Data engineers:

- Validate source schemas.
- Standardize data types.
- Handle missing values.
- Deduplicate records.
- Join related datasets.
- Apply business rules.

### Step 4: Warehouse Modeling

Create fact and dimension tables.

```text
dim_customer
dim_product
dim_date
fact_orders
fact_payments
```

### Step 5: Data Marts

Create subject-specific datasets.

```text
sales_data_mart
finance_data_mart
customer_data_mart
```

### Step 6: Reporting

Business users access the curated data through BI tools.

### Complete Flow

```text
MySQL / PostgreSQL / APIs / CSV
                |
                v
          Data Ingestion
                |
                v
         Staging / Raw Layer
                |
                v
         ETL / ELT Processing
                |
                v
         Enterprise Warehouse
                |
                v
            Data Marts
                |
                v
          BI / Dashboards
```

---

## 10. Role of a Data Engineer in Warehouse Architecture

A data engineer builds and maintains the pipelines and data structures that make warehouse data reliable and usable.

### Typical Responsibilities

| Responsibility | Description |
|---|---|
| Source integration | Connect databases, APIs, and files |
| Data ingestion | Build batch and incremental pipelines |
| ETL / ELT | Transform source data |
| Data modeling | Design fact and dimension tables |
| Data quality | Validate and reconcile data |
| Performance tuning | Optimize queries and data layouts |
| Orchestration | Schedule and coordinate workflows |
| Monitoring | Track pipeline failures and data freshness |
| Governance | Apply access controls and lineage practices |

### Example Technology Stack

```text
Source Systems
      |
      v
Azure Data Factory
      |
      v
ADLS Gen2
      |
      v
Azure Databricks
(PySpark / SQL)
      |
      v
Data Warehouse / Delta Tables
      |
      v
Power BI
```

This is one possible implementation. A data warehouse may also ingest data directly from sources without a separate data lake.

---

## 11. Common Interview Mistakes

| Incorrect Understanding | Correct Understanding |
|---|---|
| Staging area and warehouse are the same | Staging is an intermediate area; warehouse stores integrated analytical data |
| A data mart is always independent | It can be dependent on an enterprise warehouse |
| Three-tier architecture means Bronze, Silver, Gold | These are different architectural concepts |
| Every warehouse requires a staging database | Staging can use files, object storage, or other intermediate layers |
| OLAP and ETL are the same | OLAP is analytical processing; ETL is a data integration process |
| A data warehouse stores only current data | It commonly maintains historical data |
| Data marts always contain separate copies | They can be implemented through views, tables, or other logical/physical structures |
| A cloud warehouse always separates storage and compute physically | Implementation differs across platforms |

---

## 12. Interview Questions and Answers

### Q1. What is data warehouse architecture?

It describes how data is collected from source systems, ingested, transformed, stored, governed, and made available for analytical reporting.

### Q2. What are the main components of a data warehouse?

The main components include data sources, ingestion, staging, ETL/ELT processing, warehouse storage, data marts, presentation tools, and governance.

### Q3. What is a staging area?

A staging area is an intermediate location where extracted data is stored and prepared before loading into the warehouse.

### Q4. What is a data mart?

A data mart is a subject-specific subset of data designed for a department or business function, such as sales or finance.

### Q5. What is an enterprise data warehouse?

An EDW is a centralized warehouse that integrates data from multiple departments and provides a consistent organization-wide view of business information.

### Q6. Explain three-tier data warehouse architecture.

The bottom tier contains warehouse storage and data integration processes. The middle tier provides OLAP or analytical processing. The top tier contains BI, reporting, and visualization tools.

### Q7. What is the difference between an independent and dependent data mart?

An independent data mart is built directly from source systems, while a dependent data mart receives data from a central enterprise warehouse.

### Q8. What is the difference between star and snowflake schemas?

A star schema generally has denormalized dimensions connected directly to a fact table. A snowflake schema normalizes dimensions into additional related tables.

### Q9. Why is a staging area useful?

It separates source extraction from warehouse loading and transformation. It can support validation, error handling, recovery, and repeatable processing.

### Q10. What is the role of metadata in a data warehouse?

Metadata describes the data, including schemas, data types, sources, lineage, ownership, and refresh information. It helps users and engineers understand and manage the warehouse.

### Q11. What is the difference between ETL and ELT in warehouse architecture?

ETL transforms data before loading it into the target warehouse. ELT loads data first and performs transformations inside the target platform.

### Q12. How would you design a data warehouse for an e-commerce company?

I would identify the source systems, build ingestion pipelines, land data in a staging or raw layer, validate and transform it, create fact and dimension tables, and expose curated datasets through data marts or BI tools. I would also implement incremental loading, data quality checks, orchestration, monitoring, and governance.

### Q13. Is a data warehouse always built using three tiers?

No. Three-tier architecture is a common conceptual model, but actual warehouse implementations can use different physical and logical architectures.

---

## 13. Quick Revision

| Component | Purpose |
|---|---|
| Data Sources | Generate source data |
| Ingestion | Bring data into the platform |
| Staging Area | Intermediate data storage |
| ETL / ELT | Transform and integrate data |
| Data Warehouse | Central analytical storage |
| Data Mart | Department-specific analytical data |
| OLAP Layer | Analytical query processing |
| Presentation Layer | BI and reporting |
| Metadata | Describes data and its structure |
| Governance | Controls access, quality, and lineage |
| EDW | Organization-wide integrated warehouse |

### One-Minute Interview Answer

Data warehouse architecture describes how data moves from multiple source systems into a centralized analytical environment.

A typical architecture includes data sources, an ingestion layer, a staging area, ETL or ELT processing, warehouse storage, data marts, and BI tools.

The ingestion layer collects data from databases, APIs, and files. The staging area temporarily holds incoming data. ETL or ELT processes clean and transform the data before it is organized into analytical tables, commonly using fact and dimension models.

The warehouse can provide data to subject-specific data marts and BI tools such as Power BI. Governance, metadata, data quality, and monitoring operate across the architecture.

A common conceptual design is the three-tier architecture: bottom tier for warehouse storage and data integration, middle tier for OLAP processing, and top tier for reporting and analysis.

---

## 14. Self-Assessment

- [ ] Define data warehouse architecture.
- [ ] Identify the main architecture components.
- [ ] Explain the role of data sources.
- [ ] Explain data ingestion.
- [ ] Define a staging area.
- [ ] Compare ETL and ELT.
- [ ] Explain the warehouse storage layer.
- [ ] Define a data mart.
- [ ] Explain an enterprise data warehouse.
- [ ] Describe three-tier architecture.
- [ ] Compare independent and dependent data marts.
- [ ] Understand star and snowflake schemas.
- [ ] Explain modern cloud warehouse architecture.
- [ ] Describe the data engineer's responsibilities.
- [ ] Draw an end-to-end e-commerce warehouse architecture.

**Interview readiness:** Be able to draw a complete warehouse architecture from source systems to BI reporting. Explain the responsibility of each layer and distinguish staging, warehouse storage, data marts, and the presentation layer.