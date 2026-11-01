

# Metadata and Data Catalog Design

## 1. Introduction

Modern data platforms manage hundreds or thousands of datasets across databases, data lakes, warehouses, and analytical systems.

Without proper metadata management, it becomes difficult to understand:

- What data is available?
- Where is the data stored?
- What does each column represent?
- Who owns a dataset?
- Where did the data come from?
- Which pipelines produce it?
- Who can access it?
- How reliable and fresh is it?

Two important concepts help solve these problems:

**Metadata Management:** The process of collecting, storing, maintaining, and managing information about data.

**Data Catalog:** A centralized inventory of data assets that helps users discover, understand, govern, and access available data.

---

## 2. What Is Metadata?

Metadata means **data about data**.

It describes the structure, meaning, origin, ownership, usage, and management of a dataset.

### Example

Suppose an organization has an `orders` table.

| order_id | customer_id | amount | order_date |
|---|---|---:|---|
| 101 | 1 | 500 | 2026-10-29 |
| 102 | 2 | 800 | 2026-10-30 |

The actual records are business data.

Information describing the table is metadata.

| Metadata Attribute | Example |
|---|---|
| Table Name | orders |
| Database | sales |
| Schema | production |
| Column | order_id |
| Data Type | INTEGER |
| Description | Unique order identifier |
| Owner | Sales Data Team |
| Source | E-commerce Database |
| Storage Format | Delta |
| Location | ADLS Gen2 |
| Created At | 2026-10-01 |
| Refresh Frequency | Daily |

Metadata helps users understand the dataset without examining every record.

---

## 3. Types of Metadata

Metadata is commonly divided into three major categories.

### 3.1 Technical Metadata

Describes the technical structure and storage of data.

Examples:

- Database name.
- Schema name.
- Table name.
- Column names.
- Data types.
- File format.
- Storage location.
- Partition columns.
- Table size.
- Schema version.

Example:

```text
Table: orders
Database: sales
Format: Delta
Columns:
    order_id: BIGINT
    customer_id: BIGINT
    amount: DECIMAL(12,2)
    order_date: DATE
```

**Purpose:** Helps data engineers understand how data is organized and stored.

### 3.2 Business Metadata

Describes the business meaning of data.

Examples:

- Business definitions.
- Data owner.
- Business glossary.
- KPI definitions.
- Data classification.
- Usage instructions.
- Business rules.

Example:

```text
Column: amount
Business Meaning: Total order value before applicable refunds
Owner: Sales Analytics Team
Classification: Internal
```

**Purpose:** Helps business users understand what the data actually represents.

### 3.3 Operational Metadata

Describes how data is created, processed, and maintained.

Examples:

- Pipeline execution status.
- Last successful run.
- Processing duration.
- Record counts.
- Data freshness.
- Failure information.
- Ingestion timestamps.
- Job identifiers.

Example:

```text
Pipeline: orders_daily_pipeline
Status: SUCCESS
Start Time: 01:00 AM
End Time: 01:18 AM
Records Read: 100000
Records Written: 99800
Records Rejected: 200
```

**Purpose:** Helps engineers monitor pipeline health and troubleshoot problems.

### Comparison

| Metadata Type | Describes | Main Users |
|---|---|---|
| Technical | Structure and storage | Data Engineers |
| Business | Meaning and ownership | Business Analysts, Data Owners |
| Operational | Processing and execution | Data Engineers, Operations Teams |

Some organizations also maintain security, governance, and usage metadata as separate categories.

---

## 4. What Is a Data Catalog?

A data catalog is a centralized inventory of data assets and their associated metadata.

It helps users discover and understand datasets across an organization.

### Example

Imagine a company has these datasets:

```text
Production
    |
    +-- Sales
    |     |
    |     +-- orders
    |     +-- customers
    |     +-- payments
    |
    +-- Marketing
    |     |
    |     +-- campaigns
    |     +-- customer_activity
    |
    +-- Finance
          |
          +-- invoices
          +-- transactions
```

Without a catalog, users may not know:
- Which table contains customer information.
- Whether a dataset is reliable.
- Who owns a table.
- Which table should be used for reporting.
- Whether the data contains sensitive information.

A data catalog provides searchable information about these assets.

### Main Capabilities

1. Data discovery.
2. Metadata management.
3. Search and classification.
4. Ownership information.
5. Data lineage.
6. Access and governance integration.
7. Business glossary.
8. Data quality information.
9. Dataset documentation.

---

## 5. Metadata Management vs Data Catalog

These concepts are related but not identical.

| Metadata Management | Data Catalog |
|---|---|
| Broader process of managing metadata | Platform or inventory for discovering data assets |
| Collects and maintains metadata | Organizes and exposes metadata |
| Includes technical and operational metadata | Helps users search and understand assets |
| Can involve several systems | Often provides a unified interface |
| Focuses on metadata lifecycle | Focuses on discovery, understanding, and governance |

**Remember:** Metadata is information about data. Metadata management is the process of managing it. A data catalog is a system that makes data assets and their metadata discoverable.

---

## 6. High-Level Data Catalog Architecture

A data catalog usually collects metadata from multiple systems.

```text
              Data Sources
             /      |       \
            /       |        \
       Databases  Data Lake  Warehouse
            \       |        /
             \      |       /
                    |
                    v
             Metadata Connectors
                    |
                    v
              Metadata Scanner
                    |
                    v
              Metadata Repository
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Search   Classification Lineage
          |         |         |
          +---------+---------+
                    |
                    v
                Data Catalog
                    |
             +------+------+
             |             |
             v             v
        Data Engineers  Business Users
```

### Architecture Components

| Component | Responsibility |
|---|---|
| Data Sources | Contain actual datasets |
| Metadata Connectors | Connect to databases, lakes, and warehouses |
| Metadata Scanner | Extracts metadata |
| Metadata Repository | Stores collected metadata |
| Search Engine | Enables dataset discovery |
| Classification Engine | Identifies and labels sensitive data |
| Lineage Engine | Tracks data dependencies |
| Catalog Interface | Provides access to metadata |

The catalog stores information about datasets; it does not necessarily store the actual business records.

---

## 7. Metadata Collection

Metadata can be collected automatically or manually.

### 7.1 Automatic Metadata Collection

A scanner connects to a source system and extracts metadata.

For example, it can identify:
- Table names.
- Column names.
- Data types.
- Schema information.
- File formats.
- Table locations.
- Available technical statistics.

Example:

```sql
SELECT
    TABLE_SCHEMA,
    TABLE_NAME,
    COLUMN_NAME,
    DATA_TYPE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'orders';
```

This query retrieves column-level metadata from a database that supports the relevant `INFORMATION_SCHEMA` views.

### 7.2 Manual Metadata Collection

Users provide information that cannot always be inferred automatically.

Examples:
- Business definitions.
- Data ownership.
- Dataset purpose.
- Business rules.
- Usage instructions.
- Approved KPI definitions.

### Best Practice

Combine automatic collection with manual business documentation.

Automatic scanning can identify a column named `amount`, but it may not understand whether that amount represents gross sales, net sales, or refunded value.

---

## 8. Data Discovery and Search

Data discovery helps users find the correct dataset.

### Example

A business analyst wants to calculate monthly revenue.

They search the catalog for:

```text
monthly sales
revenue
orders
transactions
```

The catalog may show:

| Dataset | Description | Owner |
|---|---|---|
| gold.daily_sales | Daily sales summary | Sales Analytics |
| silver.orders | Cleaned order records | Data Engineering |
| silver.payments | Payment transactions | Finance Data Team |

The user can inspect descriptions, ownership, quality information, and lineage before choosing a dataset.

### Useful Search Features

- Table and column search.
- Keyword search.
- Business glossary search.
- Owner-based filtering.
- Domain-based filtering.
- Classification filters.
- Certified or recommended datasets.

A catalog improves discovery, but users must still verify whether a dataset matches their business requirements.

---

## 9. Data Lineage

Data lineage describes where data comes from and how it moves through different systems.

It helps answer:

- Where did this table originate?
- Which pipeline created it?
- Which transformations were applied?
- Which reports depend on it?
- What will be affected if the source changes?

### Example

```text
Azure SQL Database
        |
        v
ADF Ingestion Pipeline
        |
        v
ADLS Bronze Orders
        |
        v
Databricks PySpark Job
        |
        v
ADLS Silver Orders
        |
        v
Gold Daily Sales
        |
        v
Power BI Dashboard
```

### Types of Lineage

**1. Source-to-Target Lineage**

Shows the relationship between source datasets and target datasets.

**2. Column-Level Lineage**

Shows how individual target columns are derived from source columns.

Example:

```text
source_orders.amount
          |
          v
silver_orders.amount
          |
          v
gold_sales.total_revenue
```

**3. Pipeline-Level Lineage**

Shows which jobs or pipelines create and update datasets.

**4. Report-Level Lineage**

Shows which datasets are used by reports and dashboards.

### Why Is Lineage Important?

- Impact analysis.
- Debugging.
- Auditing.
- Regulatory compliance.
- Understanding transformations.
- Identifying upstream dependencies.

**Important:** Lineage is only as complete as the metadata and execution information collected by the platform. Dynamic SQL, external systems, and unsupported tools can create lineage gaps.

---

## 10. Data Ownership and Stewardship

A catalog should identify who is responsible for each dataset.

### Data Owner

A data owner is accountable for the dataset from a business or organizational perspective.

Responsibilities may include:
- Defining business meaning.
- Approving access.
- Setting usage rules.
- Confirming data requirements.
- Resolving ownership issues.

### Data Steward

A data steward helps maintain data quality, definitions, documentation, and governance.

Responsibilities may include:
- Maintaining business glossary terms.
- Reviewing data quality.
- Managing metadata.
- Supporting classification.
- Coordinating data-related issues.

### Data Custodian

A data custodian manages technical storage and protection of data.

Responsibilities may include:
- Storage administration.
- Backup and recovery.
- Technical access controls.
- Infrastructure operations.

Exact responsibilities vary between organizations.

### Comparison

| Role | Main Responsibility |
|---|---|
| Data Owner | Business accountability |
| Data Steward | Data quality and documentation |
| Data Custodian | Technical storage and protection |
| Data Engineer | Data pipelines and platform implementation |

---

## 11. Data Classification and Governance

Not all data should be accessible to every employee.

A catalog can store classification labels to help organizations apply appropriate governance policies.

### Example Classification

| Classification | Example |
|---|---|
| Public | Public product information |
| Internal | Internal operational reports |
| Confidential | Business financial information |
| Restricted | Sensitive personal or regulated information |

### Personally Identifiable Information (PII)

PII refers to information that can identify an individual, directly or indirectly, depending on the applicable definition and context.

Examples:
- Full name.
- Email address.
- Phone number.
- Government identification number.
- Address.

### Governance Practices

1. Classify sensitive data.
2. Assign data owners.
3. Apply least-privilege access.
4. Maintain access audit logs.
5. Define retention policies.
6. Mask or tokenize sensitive values when required.
7. Document data usage restrictions.
8. Review permissions regularly.

**Important:** A classification label in a catalog does not automatically secure the underlying data. Enforcement must be configured in the storage, query engine, and governance systems.

---

## 12. Data Quality Metadata

A catalog can display information about dataset quality.

For example:

| Quality Metric | Value |
|---|---:|
| Total Records | 100,000 |
| Valid Records | 99,800 |
| Invalid Records | 200 |
| Duplicate Records | 0 |
| Null Order IDs | 0 |
| Last Successful Update | 01:18 AM |

### Common Quality Dimensions

- Completeness.
- Uniqueness.
- Validity.
- Accuracy.
- Consistency.
- Freshness.
- Referential integrity.

### Example: Data Quality Metadata

Suppose the `orders` table must have a non-null `order_id`.

```sql
SELECT
    COUNT(*) AS total_records,
    SUM(
        CASE
            WHEN order_id IS NULL THEN 1
            ELSE 0
        END
    ) AS null_order_ids
FROM orders;
```

The quality results can be stored as operational metadata and displayed in the catalog.

**Important:** A quality score is meaningful only when the underlying rules, measurement period, and calculation method are documented.

---

## 13. Business Glossary

A business glossary provides standardized definitions of important business terms.

Different teams may use the same term differently.

For example, the term "revenue" may mean:
- Gross order value.
- Net sales after discounts.
- Recognized revenue after accounting rules.

A business glossary helps standardize these definitions.

### Example

| Business Term | Definition |
|---|---|
| Revenue | Business-approved revenue calculation |
| Active Customer | Customer meeting defined activity criteria |
| Order Value | Value of an order under agreed rules |
| Churn Rate | Percentage of customers lost during a defined period |
| Conversion Rate | Defined ratio of successful conversions to eligible visits |

### Why Is It Important?

- Reduces ambiguity.
- Standardizes KPIs.
- Improves communication.
- Helps analysts select the correct data.
- Supports consistent reporting.

A glossary should include calculation rules and scope wherever a term has multiple possible interpretations.

---

## 14. Metadata Repository Design

A metadata repository stores the information collected by the catalog.

A simplified relational design could contain the following tables.

### 14.1 Dataset Table

| Column | Description |
|---|---|
| dataset_id | Unique dataset identifier |
| dataset_name | Dataset name |
| database_name | Database name |
| schema_name | Schema name |
| storage_location | Dataset location |
| format | File or table format |
| owner_id | Responsible owner |
| created_at | Creation timestamp |

### 14.2 Column Metadata Table

| Column | Description |
|---|---|
| column_id | Unique column identifier |
| dataset_id | Related dataset |
| column_name | Column name |
| data_type | Data type |
| description | Business or technical description |
| nullable | Whether NULL is allowed |
| classification | Sensitivity label |

### 14.3 Pipeline Metadata Table

| Column | Description |
|---|---|
| pipeline_id | Pipeline identifier |
| pipeline_name | Pipeline name |
| source_dataset_id | Source dataset |
| target_dataset_id | Target dataset |
| schedule | Execution frequency |
| owner_id | Pipeline owner |

### 14.4 Dataset Run Metadata Table

| Column | Description |
|---|---|
| run_id | Execution identifier |
| dataset_id | Dataset identifier |
| status | Run status |
| records_read | Records extracted |
| records_written | Records loaded |
| start_time | Start timestamp |
| end_time | End timestamp |

### 14.5 Lineage Table

| Column | Description |
|---|---|
| lineage_id | Unique lineage identifier |
| source_dataset_id | Upstream dataset |
| target_dataset_id | Downstream dataset |
| pipeline_id | Transformation pipeline |
| transformation_description | Transformation details |

### Simplified Relationship

```text
       Dataset
          |
    +-----+------+
    |     |      |
    v     v      v
 Columns Pipeline Runs
           |
           v
        Lineage
```

This is an illustrative schema. Production catalogs may use graph databases, search indexes, relational repositories, or combinations of these.

---

## 15. Data Catalog Tools

Several tools provide metadata discovery, cataloging, governance, and lineage capabilities.

### Common Tools

| Tool | Main Purpose |
|---|---|
| Databricks Unity Catalog | Governance and management of Databricks data assets |
| Microsoft Purview | Data governance, cataloging, and discovery |
| AWS Glue Data Catalog | Centralized metadata repository for AWS analytics |
| Apache Atlas | Metadata management and governance |
| OpenMetadata | Open-source metadata management |
| DataHub | Metadata platform and data discovery |
| Google Cloud Dataplex | Data management and governance capabilities |

### Unity Catalog

Unity Catalog is Databricks' centralized governance solution.

It supports:
- Data discovery.
- Access control.
- Metadata management.
- Data lineage.
- Auditing.
- Management of tables, views, and other assets.

Common namespace:

```text
catalog.schema.table
```

Example:

```sql
SELECT *
FROM production.sales.orders;
```

### Microsoft Purview

Microsoft Purview provides capabilities for:
- Data cataloging.
- Data discovery.
- Classification.
- Governance.
- Data lineage, subject to supported sources and integrations.

### AWS Glue Data Catalog

AWS Glue Data Catalog stores metadata about datasets and tables used by AWS analytics services.

It is commonly used with services such as:
- AWS Glue.
- Amazon Athena.
- Amazon EMR.
- Amazon Redshift Spectrum.

**Interview point:** A metadata catalog is not necessarily a storage engine. It describes and organizes data assets rather than replacing the systems that store the actual data.

---

## 16. Metadata Management in an Azure Data Platform

Consider an Azure-based data platform using:

- Azure Data Factory.
- ADLS Gen2.
- Azure Databricks.
- Delta Lake.
- Power BI.

### Architecture

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
             Databricks / Delta
                    |
                    v
             Bronze / Silver / Gold
                    |
                    v
              Metadata Catalog
                    |
          +---------+---------+
          |                   |
          v                   v
     Data Discovery       Data Lineage
          |                   |
          +---------+---------+
                    |
                    v
               Power BI
```

A governance solution such as Unity Catalog or Microsoft Purview can help manage and discover metadata, depending on the scope and integration requirements.

### Example Metadata Flow

1. ADF ingests source data.
2. Databricks creates Delta tables.
3. The catalog captures table and column metadata.
4. Business descriptions and owners are added.
5. Lineage information is collected from supported integrations.
6. Access policies are configured.
7. Users search for approved datasets.
8. BI tools consume curated data.

---

## 17. Metadata Management Challenges

### 1. Metadata Inconsistency

Different systems may use different names and definitions.

**Solution:** Establish naming standards and a common business glossary.

### 2. Outdated Metadata

A table may change while its documentation remains unchanged.

**Solution:** Schedule metadata scans and establish ownership for documentation updates.

### 3. Incomplete Lineage

Some transformations may not be captured.

**Solution:** Integrate supported pipeline, query, and processing metadata sources.

### 4. Sensitive Data Exposure

Catalog descriptions or sample values may expose sensitive information.

**Solution:** Apply access controls and avoid storing sensitive values unnecessarily.

### 5. Large Metadata Volume

Thousands of tables and columns create discovery and management challenges.

**Solution:** Use search, classification, ownership, and domain-based organization.

### 6. Multiple Catalogs

Different platforms may maintain separate metadata repositories.

**Solution:** Define an enterprise metadata strategy and integrate catalogs where practical.

---

## 18. Common Interview Mistakes

1. Confusing metadata with actual business data.
2. Treating metadata management and data catalog as identical.
3. Assuming a catalog stores all actual dataset records.
4. Confusing data lineage with data profiling.
5. Assuming automatic scanners understand business meaning.
6. Assuming a classification label automatically enforces security.
7. Ignoring metadata freshness.
8. Treating a data catalog as a replacement for a database.
9. Ignoring ownership and stewardship.
10. Assuming lineage is always complete across every system.

---

## 19. Interview Questions and Answers

### Q1. What is metadata?

Metadata is information that describes data, such as its structure, meaning, source, ownership, location, and processing history.

### Q2. What are the three major types of metadata?

Technical metadata, business metadata, and operational metadata.

### Q3. What is technical metadata?

It describes the technical structure of data, including tables, columns, data types, formats, and storage locations.

### Q4. What is business metadata?

It describes the business meaning of datasets, including definitions, ownership, KPIs, and business rules.

### Q5. What is operational metadata?

It describes pipeline and data operations, such as execution status, timestamps, record counts, and processing duration.

### Q6. What is a data catalog?

A data catalog is a searchable inventory of data assets and their metadata.

### Q7. What is the difference between metadata management and a data catalog?

Metadata management is the broader process of managing metadata. A data catalog organizes and exposes metadata to help users discover and understand datasets.

### Q8. What is data lineage?

Data lineage tracks where data originates, how it is transformed, and where it is consumed.

### Q9. Why is data lineage important?

It supports impact analysis, debugging, auditing, and understanding data dependencies.

### Q10. What is a business glossary?

A business glossary contains standardized definitions of business terms and metrics.

### Q11. What is data stewardship?

Data stewardship involves maintaining data quality, documentation, definitions, and governance.

### Q12. What is the difference between a data owner and a data steward?

A data owner is accountable for the dataset. A data steward helps maintain its quality, definitions, and documentation.

### Q13. How is metadata collected?

Metadata can be collected automatically through connectors and scanners or manually through business documentation and ownership processes.

### Q14. What is data classification?

Data classification assigns labels to datasets based on their sensitivity or business importance.

### Q15. What is the purpose of a metadata repository?

It stores metadata about datasets, columns, pipelines, ownership, and lineage.

### Q16. What is Unity Catalog?

Unity Catalog is Databricks' centralized governance solution for data discovery, access control, metadata, lineage, and auditing.

### Q17. What is Microsoft Purview?

Microsoft Purview is Microsoft's data governance solution, providing cataloging, discovery, classification, and related governance capabilities.

### Q18. Does a data catalog store actual business data?

Usually, it stores metadata and references to data assets rather than replacing the underlying storage system.

### Q19. How do you keep metadata up to date?

Use scheduled scans, event-driven updates where supported, pipeline metadata publishing, and clear ownership responsibilities.

### Q20. How would you design a data catalog for a large organization?

Connect source systems, collect technical metadata, maintain business definitions and ownership, store metadata centrally, capture lineage, apply governance, and provide searchable discovery.

---

## 20. Quick Revision

| Concept | Key Point |
|---|---|
| Metadata | Data about data |
| Technical Metadata | Structure, schema, format, location |
| Business Metadata | Meaning, ownership, KPI definitions |
| Operational Metadata | Pipeline runs, counts, freshness |
| Metadata Management | Process of managing metadata |
| Data Catalog | Searchable inventory of data assets |
| Metadata Repository | Stores metadata |
| Data Discovery | Finding relevant datasets |
| Data Lineage | Tracking data origin and movement |
| Data Owner | Business accountability |
| Data Steward | Quality and documentation |
| Data Custodian | Technical storage and protection |
| Business Glossary | Standard business definitions |
| Data Classification | Sensitivity labeling |
| Unity Catalog | Databricks governance solution |
| Microsoft Purview | Microsoft data governance solution |

### Five Important Interview Points

1. Metadata describes data; it is not the actual business data.
2. Technical, business, and operational metadata serve different purposes.
3. A data catalog helps users discover and understand datasets.
4. Data lineage explains how data moves from sources to reports.
5. Cataloging, classification, and access enforcement are related but separate responsibilities.

---

## 21. Self-Assessment

Try answering without looking at the notes.

1. What is metadata?
2. Explain technical, business, and operational metadata.
3. What is a data catalog?
4. Metadata management vs data catalog: what is the difference?
5. Draw a high-level data catalog architecture.
6. How is metadata collected?
7. What is data discovery?
8. What is data lineage?
9. Explain source-to-target and column-level lineage.
10. What is a business glossary?
11. Explain data owner vs data steward vs data custodian.
12. What is data classification?
13. What is the purpose of a metadata repository?
14. How do you track data quality metadata?
15. What is Unity Catalog?
16. What is Microsoft Purview?
17. How would you keep metadata updated?
18. What are common metadata management challenges?
19. How would you design a catalog for 1,000 datasets?
20. How does metadata management help data engineers?

**Interview readiness:** You should be able to explain the three metadata types, draw a catalog architecture, describe metadata collection and lineage, distinguish ownership roles, and explain how cataloging supports discovery and governance in an Azure or Databricks environment.

