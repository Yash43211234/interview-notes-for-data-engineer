# Medallion Architecture in Databricks

## 1. What Is Medallion Architecture?

**Medallion Architecture** is a data design pattern used in data engineering to organize data into multiple layers based on its level of processing and quality.

In Databricks, it commonly consists of three layers:

1. **Bronze Layer:** Stores raw ingested data.
2. **Silver Layer:** Stores cleaned, validated, and transformed data.
3. **Gold Layer:** Stores business-ready and aggregated data.

The main purpose is to improve data quality, maintainability, scalability, and data usability.

**Simple way to remember:**

- Bronze = Raw data.
- Silver = Clean data.
- Gold = Business-ready data.

## 2. Why Do We Need Medallion Architecture?

Suppose an e-commerce company receives data from multiple systems:

- Customer information.
- Orders.
- Payments.
- Product details.
- Website activity.

If all data is stored in one table without clear processing stages, it becomes difficult to:

- Identify data-quality problems.
- Track transformations.
- Reprocess historical data.
- Maintain business logic.
- Support different reporting requirements.

Medallion Architecture separates data into layers so each stage has a clear responsibility.

### Main Benefits

1. **Data quality:** Data becomes cleaner at each stage.
2. **Organization:** Separates raw, transformed, and business-level data.
3. **Traceability:** Helps track data from source to final output.
4. **Reusability:** Silver data can support multiple Gold datasets.
5. **Maintainability:** Makes pipeline logic easier to manage.
6. **Scalability:** Supports large-scale data processing.
7. **Debugging:** Helps identify which processing layer introduced an issue.

## 3. Medallion Architecture Flow

```text id="u1x6vr"
             SOURCE SYSTEMS
                    |
                    v
             CLOUD STORAGE
                    |
                    v
          +-------------------+
          |   BRONZE LAYER    |
          |                   |
          | Raw Ingestion     |
          | Original Data     |
          +-------------------+
                    |
                    v
          +-------------------+
          |   SILVER LAYER    |
          |                   |
          | Cleaning          |
          | Validation        |
          | Deduplication     |
          | Standardization   |
          +-------------------+
                    |
                    v
          +-------------------+
          |    GOLD LAYER     |
          |                   |
          | Aggregations      |
          | Business Logic    |
          | Reporting Tables  |
          +-------------------+
                    |
                    v
          BI / DASHBOARDS / ML
```

Each layer can be implemented using Delta tables in Databricks.

## 4. Bronze Layer

The Bronze Layer is the first layer of the Medallion Architecture.

It stores data ingested from source systems with minimal transformation.

### Main Responsibilities

- Ingest raw data.
- Preserve source information.
- Maintain ingestion metadata.
- Support historical reprocessing.
- Capture source data for downstream processing.

### Example

Suppose an e-commerce application sends order data.

Source data:

| order_id | customer_id | amount | order_date |
|---|---|---:|---|
| 101 | 501 | 500 | 2026-10-30 |
| 102 | 502 | 700 | 2026-10-30 |
| 103 | 503 | NULL | 2026-10-30 |

The Bronze layer stores the incoming records without applying business-level cleaning.

### PySpark Example

```python
bronze_df = (
    spark.read
    .format("json")
    .load(source_path)
)

bronze_df.write \
    .format("delta") \
    .mode("append") \
    .save(bronze_path)
```

### Common Bronze Metadata Columns

| Column | Purpose |
|---|---|
| `source_file` | Identifies the original file |
| `ingestion_time` | Records when data was ingested |
| `batch_id` | Identifies the ingestion batch |
| `source_system` | Identifies the originating system |

Example:

```python
from pyspark.sql import functions as F

bronze_df = (
    spark.read
    .format("json")
    .load(source_path)
    .withColumn(
        "ingestion_time",
        F.current_timestamp()
    )
    .withColumn(
        "source_file",
        F.input_file_name()
    )
)
```

**Important:** Bronze does not always mean completely untouched data. Basic technical metadata, format conversion, and ingestion-related handling may be applied. The goal is to preserve the source information and avoid unnecessary business transformations.

## 5. Silver Layer

The Silver Layer contains cleaned, validated, deduplicated, and standardized data.

It converts raw data into a more reliable and consistent format.

### Main Responsibilities

- Remove duplicate records.
- Handle NULL values.
- Validate records.
- Standardize data types.
- Correct data formats.
- Apply basic business rules.
- Handle invalid records according to data-quality requirements.

### Example

Bronze data:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | 501 | 500 |
| 102 | 502 | 700 |
| 103 | 503 | NULL |
| 101 | 501 | 500 |

Problems:

- Order 101 is duplicated.
- Order 103 has a missing amount.

Silver processing should handle these issues according to the business rules.

### PySpark Example

```python
from pyspark.sql import functions as F

silver_df = (
    bronze_df
    .dropDuplicates(["order_id"])
    .filter(F.col("amount").isNotNull())
    .filter(F.col("amount") >= 0)
)
```

This example:

- Removes duplicate order IDs.
- Removes records with missing amounts.
- Removes negative amounts.

In production, define how invalid records should be handled. They may be rejected, quarantined, or retained with a quality-status column rather than simply discarded.

### Standardizing Data Types

```python
silver_df = (
    bronze_df
    .withColumn(
        "order_id",
        F.col("order_id").cast("long")
    )
    .withColumn(
        "amount",
        F.col("amount").cast("decimal(12,2)")
    )
)
```

Explicitly validate failed casts because invalid values may become NULL or cause errors depending on Spark configuration and casting behavior.

**Important:** Silver is generally the trusted, reusable layer for downstream data processing.

## 6. Gold Layer

The Gold Layer contains business-ready data designed for analytics, reporting, and decision-making.

It applies business logic, aggregations, and dimensional modeling where appropriate.

### Main Responsibilities

- Create business-level aggregations.
- Calculate KPIs.
- Prepare reporting tables.
- Build fact and dimension tables.
- Support BI tools and dashboards.
- Provide data for business analysis.

### Example

Suppose the business wants daily sales reports.

Silver data:

| order_id | customer_id | amount | order_date |
|---|---|---:|---|
| 101 | 501 | 500 | 2026-10-30 |
| 102 | 502 | 700 | 2026-10-30 |
| 103 | 503 | 300 | 2026-10-31 |

Gold output:

| order_date | total_orders | total_sales |
|---|---:|---:|
| 2026-10-30 | 2 | 1200 |
| 2026-10-31 | 1 | 300 |

### PySpark Example

```python
from pyspark.sql import functions as F

gold_df = (
    silver_df
    .groupBy("order_date")
    .agg(
        F.countDistinct("order_id").alias("total_orders"),
        F.sum("amount").alias("total_sales")
    )
)
```

Write the result:

```python
gold_df.write \
    .format("delta") \
    .mode("overwrite") \
    .save(gold_path)
```

This overwrite example assumes the complete Gold result is being rebuilt. For incremental Gold processing, use an appropriate incremental aggregation or MERGE strategy instead.

**Important:** Gold tables are often designed around specific reporting and analytical requirements rather than simply copying Silver tables.

## 7. Bronze vs Silver vs Gold

This is one of the most important Medallion Architecture interview questions.

| Feature | Bronze | Silver | Gold |
|---|---|---|---|
| Data type | Raw source data | Cleaned and validated data | Business-ready data |
| Transformation | Minimal | Moderate | Business-oriented |
| Data quality | May contain issues | Improved and validated | Designed for consumption |
| Main purpose | Preserve source data | Create trusted data | Support analytics |
| Typical operations | Ingestion | Cleaning, deduplication, joins | Aggregation, KPIs, modeling |
| Users | Data engineers | Data engineers and analysts | Analysts, BI teams, business users |
| Example | Raw orders | Valid orders | Daily sales summary |

## 8. End-to-End Medallion Pipeline Example

Consider an e-commerce company that receives customer and order data from source systems.

### Step 1: Source Data

```text
Customer Database
Order Database
Payment System
       |
       v
   Cloud Storage
```

Data may arrive as CSV, JSON, or Parquet files.

### Step 2: Bronze Ingestion

Use Auto Loader to ingest incoming files into a Bronze Delta table.

```python
bronze_df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", schema_path)
    .load(source_path)
)

query = (
    bronze_df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .trigger(availableNow=True)
    .toTable("main.bronze.orders")
)
```

### Step 3: Silver Transformation

Read Bronze data and apply data-quality rules.

```python
bronze_orders = spark.read.table(
    "main.bronze.orders"
)

silver_orders = (
    bronze_orders
    .dropDuplicates(["order_id"])
    .filter(F.col("amount").isNotNull())
    .filter(F.col("amount") >= 0)
)

silver_orders.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable("main.silver.orders")
```

This is a simplified full-refresh example. In production, consider incremental processing, deterministic deduplication, and handling invalid records.

### Step 4: Gold Aggregation

Create a daily sales summary.

```python
silver_orders = spark.read.table(
    "main.silver.orders"
)

gold_sales = (
    silver_orders
    .groupBy("order_date")
    .agg(
        F.countDistinct("order_id").alias("total_orders"),
        F.sum("amount").alias("total_sales")
    )
)

gold_sales.write \
    .format("delta") \
    .mode("overwrite") \
    .saveAsTable("main.gold.daily_sales")
```

### Step 5: Reporting

```text
main.gold.daily_sales
          |
          v
Power BI / Tableau
          |
          v
Business Dashboards
```

Business users can now analyze daily sales without querying raw source files.

## 9. Medallion Architecture with Delta Lake

Databricks commonly uses Delta Lake tables for Bronze, Silver, and Gold layers.

Delta Lake provides:

- ACID transactions.
- Schema enforcement.
- Schema evolution.
- Time Travel.
- MERGE operations.
- Batch and streaming support.

Example table structure:

```text
main
 |
 |-- bronze
 |    |-- customers
 |    |-- orders
 |    |-- payments
 |
 |-- silver
 |    |-- customers
 |    |-- orders
 |    |-- payments
 |
 |-- gold
      |-- daily_sales
      |-- customer_summary
      |-- product_performance
```

This is one possible logical organization using Unity Catalog catalogs and schemas.

**Important:** Bronze, Silver, and Gold are logical data layers. They do not have to be separate physical storage systems. Their organization depends on the organization's architecture and governance requirements.

## 10. Medallion Architecture and Incremental Processing

In production, data may arrive every few minutes or every day.

Processing the entire dataset at every run may be expensive.

A better approach is to process new or changed data incrementally.

### Example Flow

```text
New Source Files
       |
       v
Auto Loader
       |
       v
Bronze Delta
       |
       v
Incremental Silver Processing
       |
       v
Silver Delta
       |
       v
Gold Aggregations
```

Common techniques:

- Auto Loader for incremental file ingestion.
- Structured Streaming for continuously arriving data.
- Checkpointing for streaming recovery.
- MERGE for inserts and updates.
- CDC for source changes and deletes.
- Watermarks for managing processing progress.

**Remember:** Medallion Architecture describes how data is organized and refined. It does not require every layer to use streaming or incremental processing.

## 11. Data Quality Across Medallion Layers

Data quality improves as data moves through the layers.

| Layer | Data Quality Responsibility |
|---|---|
| Bronze | Preserve source data and capture ingestion metadata |
| Silver | Validate, clean, standardize, and deduplicate |
| Gold | Apply business rules and ensure analytical correctness |

### Example: Order Data

**Bronze:**

| order_id | amount |
|---|---:|
| 101 | 500 |
| 102 | NULL |
| 103 | -100 |

**Silver:**

| order_id | amount |
|---|---:|
| 101 | 500 |

Invalid records may be stored in a separate quarantine table for investigation.

**Gold:**

| total_orders | total_sales |
|---:|---:|
| 1 | 500 |

This ensures that reporting is based on validated data.

## 12. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Bronze means completely untouched data | Technical metadata and ingestion handling may be added |
| Silver is only a copy of Bronze | Silver applies cleaning and validation |
| Gold is just another raw table | Gold is designed for business consumption |
| Every layer must use streaming | Batch and streaming are both supported |
| Medallion requires three separate storage systems | Layers can be logical organizations within the same platform |
| All invalid records should be deleted | Quarantine or retain them when business requirements demand it |
| Gold tables should always be overwritten | Incremental updates may be more suitable |
| Auto Loader automatically builds all three layers | Auto Loader handles file ingestion; transformations must be implemented separately |
| Medallion automatically guarantees data quality | Data-quality rules and monitoring must be designed |

## 13. Interview Questions and Answers

**Q1. What is Medallion Architecture?**

It is a data architecture pattern that organizes data into Bronze, Silver, and Gold layers based on its level of processing and quality.

**Q2. What is the Bronze Layer?**

The Bronze Layer stores raw ingested data from source systems with minimal transformation.

**Q3. What is the Silver Layer?**

The Silver Layer contains cleaned, validated, standardized, and deduplicated data.

**Q4. What is the Gold Layer?**

The Gold Layer contains business-ready data, including aggregations, KPIs, and reporting datasets.

**Q5. Why do we need three layers?**

They separate raw ingestion, data refinement, and business-level consumption, making pipelines easier to manage and maintain.

**Q6. Can we use Delta Lake in all three layers?**

Yes. Delta Lake tables are commonly used across Bronze, Silver, and Gold.

**Q7. Where would you use Auto Loader?**

Typically in the Bronze ingestion process to incrementally discover and ingest files from cloud storage.

**Q8. Where does deduplication happen?**

It commonly happens in Silver, although ingestion-level duplicate handling may also be required.

**Q9. Can Bronze contain duplicate records?**

Yes. Bronze may preserve source records, including duplicates, depending on the ingestion design.

**Q10. What is the difference between Silver and Gold?**

Silver contains trusted, cleaned data. Gold contains data organized around business requirements, such as KPIs and reporting.

**Q11. Can Medallion Architecture support streaming?**

Yes. Structured Streaming and Auto Loader can be used to incrementally process data through the layers.

**Q12. Does Medallion Architecture require separate physical storage?**

No. Bronze, Silver, and Gold are logical layers. They can be implemented using separate schemas, tables, or storage locations.

**Q13. What happens if incorrect data enters Bronze?**

The data can be investigated, cleaned, quarantined, or rejected during downstream processing. Bronze also provides a source for reprocessing when required.

**Q14. Is Medallion Architecture the same as a data warehouse?**

No. Medallion is a layered data architecture pattern. A data warehouse is a system or platform for storing and serving analytical data. Medallion layers can be implemented on a lakehouse platform.

## 14. Quick Revision

| Layer | Main Purpose | Common Operations |
|---|---|---|
| Bronze | Raw data ingestion | Auto Loader, source ingestion, metadata |
| Silver | Data cleaning and validation | Deduplication, NULL handling, standardization |
| Gold | Business-ready data | Aggregations, KPIs, dimensional modeling |

### Important Concepts

| Concept | Remember |
|---|---|
| Medallion Architecture | Bronze → Silver → Gold |
| Bronze | Raw data |
| Silver | Clean and trusted data |
| Gold | Business-ready data |
| Auto Loader | Commonly used for Bronze ingestion |
| Delta Lake | Supports reliable tables across layers |
| MERGE | Useful for incremental updates |
| CDC | Captures source inserts, updates, and deletes |
| Data quality | Improves through validation and transformation |
| Reprocessing | Bronze can help support downstream rebuilds |

### One-Minute Interview Answer

Medallion Architecture is a layered data architecture commonly used in Databricks.

It has three main layers: Bronze, Silver, and Gold.

Bronze stores raw data from source systems. Silver cleans, validates, standardizes, and deduplicates the data. Gold contains business-ready datasets such as daily sales summaries, customer analytics, and reporting tables.

In Databricks, these layers are commonly implemented using Delta Lake. Auto Loader can ingest files into Bronze, PySpark can transform data into Silver, and SQL or PySpark can create Gold aggregations.

The main benefit is separating raw ingestion, data quality processing, and business-level analytics.

## 15. Self-Assessment

- [ ] Define Medallion Architecture.
- [ ] Explain the purpose of Bronze, Silver, and Gold.
- [ ] Differentiate the three layers.
- [ ] Explain why Bronze data is preserved.
- [ ] Describe common Silver-layer transformations.
- [ ] Explain how Gold supports business analytics.
- [ ] Explain how Delta Lake is used across layers.
- [ ] Explain where Auto Loader fits.
- [ ] Design an end-to-end Medallion pipeline.
- [ ] Explain incremental processing across the layers.
- [ ] Explain data-quality responsibilities at each layer.
- [ ] Explain how you would handle invalid records.

**Interview readiness:** You should be able to explain Medallion Architecture in 2 minutes and design a complete Bronze-to-Silver-to-Gold pipeline using Auto Loader, PySpark, and Delta Lake.