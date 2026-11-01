# Data Lineage in Databricks

## 1. What Is Data Lineage?

**Data Lineage** is the process of tracking where data comes from, how it is transformed, where it moves, and where it is used.

It provides visibility into the complete journey of data from source systems to final destinations.

In Databricks, data lineage helps data engineers and analysts understand relationships between tables, columns, notebooks, jobs, and other data assets.

**Simple way to remember:**

Data Lineage = Where data comes from + How it changes + Where it goes.

## 2. Why Do We Need Data Lineage?

Suppose an organization has a data pipeline:

```text
Source Database
      |
      v
Bronze Table
      |
      v
Silver Table
      |
      v
Gold Table
      |
      v
Power BI Dashboard
```

A business user notices that the total sales in a dashboard are incorrect.

Without lineage, engineers may need to manually investigate:

- Which source table provides the data?
- Which transformations were applied?
- Which intermediate tables were involved?
- Which job created the final table?
- Which reports depend on the affected table?

Data lineage helps answer these questions.

### Main Benefits

1. **Traceability:** Understand where data originated.
2. **Impact analysis:** Identify downstream dependencies before changing a table.
3. **Troubleshooting:** Find where incorrect data entered a pipeline.
4. **Auditing:** Support data governance and compliance activities.
5. **Data discovery:** Understand relationships between data assets.
6. **Change management:** Evaluate the impact of schema or pipeline changes.
7. **Trust:** Help users understand how analytical data was produced.

## 3. Types of Data Lineage

Data lineage can be understood at different levels.

### 3.1 Source-to-Target Lineage

Tracks data from its original source to its final destination.

Example:

```text
MySQL Database
      |
      v
ADLS Gen2
      |
      v
Bronze Delta Table
      |
      v
Silver Delta Table
      |
      v
Gold Delta Table
```

This helps identify the origin and destination of data.

### 3.2 Table-Level Lineage

Tracks relationships between tables.

Example:

```text
bronze.orders
      |
      v
silver.orders
      |
      v
gold.daily_sales
```

It answers questions such as:

- Which tables were used to create `gold.daily_sales`?
- Which downstream tables depend on `silver.orders`?

### 3.3 Column-Level Lineage

Tracks how individual columns move between tables and how they are transformed.

Example:

```text
bronze.orders.amount
          |
          v
silver.orders.amount
          |
          v
SUM(amount)
          |
          v
gold.daily_sales.total_sales
```

Here, the Gold column `total_sales` is derived from the Silver column `amount`.

Column-level lineage is especially useful when investigating incorrect calculations or changing a source column.

## 4. Data Lineage in Medallion Architecture

Medallion Architecture commonly contains three layers:

- Bronze: Raw data.
- Silver: Cleaned and validated data.
- Gold: Business-ready data.

Lineage helps track how data moves between these layers.

### Example

Suppose an e-commerce company receives customer and order data.

```text
       SOURCE SYSTEMS
              |
              v
        BRONZE LAYER
        bronze.orders
              |
              v
        SILVER LAYER
        silver.orders
              |
              v
         GOLD LAYER
        gold.daily_sales
              |
              v
       POWER BI REPORT
```

### Transformation Example

Bronze:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | 501 | 500 |
| 102 | 502 | 700 |
| 103 | 503 | NULL |

Silver transformation:

- Remove duplicate orders.
- Validate amount.
- Handle NULL values.

Gold transformation:

- Group by order date.
- Calculate total sales.
- Prepare reporting data.

Lineage connects these transformations and their data dependencies.

## 5. Unity Catalog and Data Lineage

**Unity Catalog** is Databricks' centralized governance solution for managing data and AI assets.

It provides capabilities such as:

- Access control.
- Data discovery.
- Auditing.
- Data lineage.
- Centralized governance.
- Management of supported data assets.

Unity Catalog can capture lineage for supported Databricks operations.

### Example

Suppose we have:

```text
Catalog: main

Schemas:
    bronze
    silver
    gold

Tables:
    bronze.orders
    silver.orders
    gold.daily_sales
```

When supported queries and transformations run, Unity Catalog can capture relationships between the source and target data assets.

Users with appropriate permissions can explore these relationships through the Databricks interface.

**Important:** Lineage capture depends on supported operations, assets, and platform capabilities. Do not assume every external system or transformation is automatically tracked.

## 6. How to View Data Lineage in Databricks

In a Unity Catalog-enabled workspace, lineage can generally be explored through the Catalog Explorer.

### Basic Steps

1. Open the Databricks workspace.
2. Navigate to Catalog Explorer.
3. Select the required catalog and schema.
4. Open the table.
5. Find the Lineage tab or lineage view.
6. Explore upstream and downstream dependencies.

### Upstream Lineage

Upstream lineage shows where the data comes from.

Example:

```text
bronze.orders
      |
      v
silver.orders
```

If we select `silver.orders`, its upstream dependency may include `bronze.orders`.

### Downstream Lineage

Downstream lineage shows where the data is used.

Example:

```text
silver.orders
      |
      v
gold.daily_sales
      |
      v
BI Dashboard
```

If we select `silver.orders`, its downstream dependencies may include `gold.daily_sales` and supported downstream assets.

**Remember:**

- Upstream = Data sources and dependencies.
- Downstream = Consumers and dependent assets.

## 7. Table-Level Lineage vs Column-Level Lineage

| Feature | Table-Level Lineage | Column-Level Lineage |
|---|---|---|
| Tracks | Table relationships | Individual column relationships |
| Detail | Table level | Column level |
| Example | `silver.orders` → `gold.sales` | `amount` → `total_sales` |
| Main purpose | Understand pipeline dependencies | Understand column transformations |
| Useful for | Pipeline impact analysis | Schema and calculation impact analysis |

Column-level lineage provides more detailed information than table-level lineage.

## 8. Impact Analysis

Impact analysis means identifying what may be affected when a data asset changes.

Suppose a developer wants to remove the `amount` column from `silver.orders`.

Before making the change, the developer should check downstream dependencies.

```text
silver.orders.amount
          |
          +------------------+
          |                  |
          v                  v
gold.daily_sales       gold.customer_sales
          |                  |
          v                  v
Power BI Report A      Power BI Report B
```

Removing `amount` could affect:

- Gold sales aggregations.
- Customer-level reporting.
- BI dashboards.
- Other dependent transformations.

Lineage helps identify these dependencies before making changes.

**Interview point:** Data lineage reduces the risk of breaking downstream pipelines and reports.

## 9. Data Lineage vs Data Catalog vs Data Dictionary

These are related but different concepts.

| Concept | Meaning | Example |
|---|---|---|
| Data Lineage | Tracks where data comes from and where it goes | `bronze.orders` → `silver.orders` |
| Data Catalog | Helps discover and manage data assets | List of available tables |
| Data Dictionary | Describes data elements and their meaning | `amount`: Order transaction value |

### Simple Explanation

- Data Lineage = Data journey.
- Data Catalog = Data discovery and organization.
- Data Dictionary = Data definitions and descriptions.

Unity Catalog provides centralized governance and data discovery capabilities, while lineage is one of its capabilities.

## 10. Data Lineage vs Data Provenance

These terms are related and sometimes used interchangeably, but they can emphasize different things.

| Data Lineage | Data Provenance |
|---|---|
| Focuses on the movement and transformation of data | Focuses on the origin and history of specific data |
| Shows dependencies between assets | Helps establish where particular data came from |
| Often used for pipeline impact analysis | Often used for traceability and evidence |

For interviews, explain the distinction in the context of the organization's terminology because definitions can vary.

## 11. Practical Example Using Databricks

Suppose we create a Silver table from Bronze data.

```python
bronze_df = spark.read.table(
    "main.bronze.orders"
)

silver_df = (
    bronze_df
    .dropDuplicates(["order_id"])
    .filter("amount IS NOT NULL")
)

silver_df.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable("main.silver.orders")
```

Next, create a Gold table:

```python
from pyspark.sql import functions as F

silver_df = spark.read.table(
    "main.silver.orders"
)

gold_df = (
    silver_df
    .groupBy("order_date")
    .agg(
        F.sum("amount").alias("total_sales")
    )
)

gold_df.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable("main.gold.daily_sales")
```

Conceptual lineage:

```text
main.bronze.orders
          |
          | Deduplication
          | NULL filtering
          v
main.silver.orders
          |
          | Group by order_date
          | SUM(amount)
          v
main.gold.daily_sales
```

When these operations are performed through supported Databricks and Unity Catalog workflows, lineage information can be captured and explored.

**Important:** This code demonstrates the data dependencies. It does not manually create lineage metadata. Automatic capture depends on the workspace configuration and supported operations.

## 12. Limitations and Important Considerations

1. **External systems:** Data dependencies outside Databricks may require additional integrations or manual documentation.
2. **Unsupported operations:** Not every transformation or data access pattern is guaranteed to produce lineage.
3. **Permissions:** Users may need appropriate permissions to view lineage information.
4. **Dynamic SQL:** Complex dynamically generated queries can make dependency tracking more challenging.
5. **Custom code:** Some external processing may not be fully visible in native lineage.
6. **Historical information:** Lineage visibility and retention depend on platform capabilities and configuration.
7. **Column-level detail:** Availability can depend on supported operations and data sources.

**Important:** Lineage is not a replacement for pipeline monitoring, data-quality checks, or documentation.

## 13. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Lineage only tracks source tables | It can track upstream and downstream dependencies |
| Lineage and data catalog are the same | Lineage tracks data relationships; catalog supports discovery and governance |
| Lineage automatically tracks every external transformation | Capture depends on supported operations and integrations |
| Upstream means downstream consumers | Upstream means sources and dependencies |
| Column lineage and table lineage are identical | Column lineage provides more granular detail |
| Lineage guarantees data quality | It provides traceability, not automatic quality validation |
| Lineage replaces pipeline monitoring | Monitoring and lineage serve different purposes |

## 14. Interview Questions and Answers

**Q1. What is data lineage?**

Data lineage tracks the origin, movement, transformation, and destination of data across a data pipeline.

**Q2. Why is data lineage important?**

It helps with troubleshooting, auditing, impact analysis, traceability, and understanding data dependencies.

**Q3. What is Unity Catalog?**

Unity Catalog is Databricks' centralized governance solution for managing and controlling access to data and AI assets. It also provides data lineage capabilities.

**Q4. What is upstream lineage?**

Upstream lineage shows the source tables and dependencies used to produce a data asset.

**Q5. What is downstream lineage?**

Downstream lineage shows the tables, reports, and other supported assets that depend on a data asset.

**Q6. What is column-level lineage?**

It tracks how individual columns are derived and transformed between data assets.

**Q7. What is the difference between table-level and column-level lineage?**

Table-level lineage shows relationships between tables. Column-level lineage shows relationships between individual columns.

**Q8. How does lineage help during schema changes?**

It helps identify downstream tables and reports that may be affected before changing or removing a column.

**Q9. How can you view lineage in Databricks?**

In a Unity Catalog-enabled workspace, use Catalog Explorer to open a supported data asset and explore its lineage view, subject to permissions.

**Q10. Is lineage the same as data quality?**

No. Lineage tracks data dependencies and transformations. Data-quality checks validate whether data meets defined requirements.

**Q11. Can Unity Catalog track lineage across all external systems?**

No. Native lineage capture has supported-operation and integration boundaries. External dependencies may need additional tracking.

**Q12. How would you investigate incorrect data in a Gold table?**

I would trace the Gold table's upstream dependencies, inspect the Silver and Bronze data, review transformations and job execution, and identify where the incorrect value entered the pipeline.

**Q13. What is impact analysis?**

Impact analysis identifies downstream assets that may be affected by a change to a table, column, or pipeline.

**Q14. What is the difference between lineage and provenance?**

Lineage generally focuses on data movement and transformation, while provenance often emphasizes origin and history. Their exact meanings can vary by organization.

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Data Lineage | Tracks the data journey |
| Upstream | Sources and dependencies |
| Downstream | Consumers and dependent assets |
| Table-level lineage | Relationships between tables |
| Column-level lineage | Relationships between columns |
| Unity Catalog | Centralized governance in Databricks |
| Impact analysis | Identifies effects of changes |
| Data catalog | Data discovery and organization |
| Data dictionary | Definitions of data elements |
| Data provenance | Origin and history of data |

### One-Minute Interview Answer

Data lineage is the process of tracking where data comes from, how it is transformed, and where it is consumed.

In Databricks, Unity Catalog provides lineage capabilities for supported data operations. It helps track relationships between Bronze, Silver, and Gold tables and can provide table-level and column-level visibility.

For example, if a Gold sales table contains incorrect values, I can use upstream lineage to identify its source Silver table and then trace it back to Bronze. I can also use downstream lineage to understand which reports may be affected by a schema change.

It is useful for troubleshooting, auditing, impact analysis, and data governance.

## 16. Self-Assessment

- [ ] Define data lineage.
- [ ] Explain why lineage is important.
- [ ] Explain source-to-target lineage.
- [ ] Differentiate upstream and downstream lineage.
- [ ] Differentiate table-level and column-level lineage.
- [ ] Explain Unity Catalog's role in lineage.
- [ ] Explain how to view lineage in Databricks.
- [ ] Explain impact analysis.
- [ ] Differentiate lineage, catalog, and dictionary.
- [ ] Explain lineage limitations.
- [ ] Describe how lineage helps troubleshoot incorrect Gold data.
- [ ] Explain how lineage supports schema-change planning.

**Interview readiness:** You should be able to explain data lineage in two minutes, describe how Unity Catalog supports it, and demonstrate how you would use upstream and downstream lineage to investigate a production data issue.