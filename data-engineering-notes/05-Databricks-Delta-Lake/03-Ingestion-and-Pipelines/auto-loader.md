# Databricks Auto Loader

## 1. What Is Auto Loader?

**Auto Loader** is a Databricks feature that incrementally processes new files arriving in cloud storage.

It is commonly used to build scalable data ingestion pipelines that continuously or periodically ingest files from locations such as:

- Azure Data Lake Storage (ADLS)
- Amazon S3
- Google Cloud Storage

Instead of repeatedly scanning and processing all files in a storage directory, Auto Loader tracks which files have already been discovered and processes newly arriving files.

**Example:**

Suppose an e-commerce application delivers new order files to ADLS every hour.

- 10:00 AM: `orders_01.csv` arrives.
- 11:00 AM: `orders_02.csv` arrives.
- 12:00 PM: `orders_03.csv` arrives.

Auto Loader incrementally discovers and processes these files without requiring the pipeline to reprocess the entire directory.

## 2. Why Do We Need Auto Loader?

Consider a storage directory containing millions of files.

A traditional file ingestion process may repeatedly list or scan the directory to identify new files. This can become expensive and inefficient as the number of files increases.

Auto Loader helps by providing:

1. **Incremental ingestion:** Processes newly discovered files.
2. **Scalability:** Supports large-scale file ingestion.
3. **Schema inference:** Can infer the structure of incoming data.
4. **Schema evolution:** Can detect certain schema changes and handle them according to configuration.
5. **Checkpointing:** Tracks ingestion progress so a query can recover after failure.
6. **Flexible execution:** Supports scheduled micro-batch and continuous-style ingestion patterns through Structured Streaming.

## 3. How Auto Loader Works

Auto Loader uses the `cloudFiles` source format in Structured Streaming.

The basic flow is:

1. Files arrive in cloud storage.
2. Auto Loader discovers new files.
3. It tracks discovered-file information using its metadata mechanism.
4. The streaming query reads the file contents.
5. Data is transformed if required.
6. Data is written to a destination, commonly a Delta table.
7. Checkpoint information helps the query recover and continue processing.

### Basic Architecture

```text
Source Files
     |
     v
ADLS / S3 / GCS
     |
     v
Databricks Auto Loader
     |
     |-- File Discovery
     |-- Schema Inference
     |-- Schema Evolution
     |-- Checkpointing
     |
     v
PySpark Transformations
     |
     v
Delta Lake Table
     |
     v
Bronze Layer
```

## 4. Basic Auto Loader Syntax

```python
df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "csv")
    .option("cloudFiles.schemaLocation", schema_path)
    .option("header", "true")
    .load(source_path)
)
```

Important components:

| Component | Purpose |
|---|---|
| `readStream` | Creates a streaming DataFrame |
| `format("cloudFiles")` | Selects Auto Loader |
| `cloudFiles.format` | Specifies the incoming file format |
| `cloudFiles.schemaLocation` | Stores schema information |
| `header` | Treats the first CSV row as column names |
| `load()` | Specifies the source directory |

Notice that `cloudFiles` is the Auto Loader source, while `csv` is the actual file format.

## 5. Complete Auto Loader Example

Suppose incoming customer CSV files arrive in ADLS.

### Step 1: Define Paths

```python
source_path = "/Volumes/main/raw/customer_files/"

schema_path = "/Volumes/main/checkpoints/customer_schema/"

checkpoint_path = "/Volumes/main/checkpoints/customer_stream/"

target_path = "/Volumes/main/bronze/customers/"
```

These are illustrative Unity Catalog volume paths. Use storage locations and permissions configured in your workspace.

### Step 2: Read Files Using Auto Loader

```python
df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "csv")
    .option("cloudFiles.schemaLocation", schema_path)
    .option("header", "true")
    .option("cloudFiles.inferColumnTypes", "true")
    .load(source_path)
)
```

### Step 3: Write Data to Delta Lake

```python
query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .outputMode("append")
    .start(target_path)
)
```

### Step 4: Wait for the Query

```python
query.awaitTermination()
```

`awaitTermination()` waits for the streaming query to terminate. It does not start the query; `.start()` does that.

**Important:** In production, the streaming query is generally managed through a Databricks Job rather than manually waiting in an interactive notebook.

## 6. Schema Inference and Schema Evolution

Incoming files may contain new columns or changes to existing columns.

Auto Loader provides schema inference and schema evolution capabilities to handle supported changes.

### Schema Inference

Auto Loader can inspect incoming data and infer its schema.

Example:

Initial file:

| customer_id | name | city |
|---|---|---|
| 101 | Rahul | Kolkata |
| 102 | Amit | Delhi |

Inferred schema:

```text
customer_id
name
city
```

### Schema Evolution

Suppose a new file arrives with an additional column:

| customer_id | name | city | country |
|---|---|---|---|
| 103 | Priya | Mumbai | India |

Auto Loader can detect the new `country` column and evolve the schema according to the configured evolution mode.

Example:

```python
df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "csv")
    .option("cloudFiles.schemaLocation", schema_path)
    .option("cloudFiles.schemaEvolutionMode", "addNewColumns")
    .option("header", "true")
    .load(source_path)
)
```

With `addNewColumns`, a newly detected column can update the stored schema and cause the stream to stop with a schema evolution error. The query can then be restarted using the updated schema.

**Important:** Schema evolution does not mean every type change or malformed value is automatically accepted. Behavior depends on the evolution mode, file format, and runtime configuration.

### Common Schema Evolution Modes

| Mode | Behavior |
|---|---|
| `addNewColumns` | Adds newly detected columns; the stream may stop and require restart |
| `addNewColumnsWithTypeWidening` | Supports certain type-widening changes where supported |
| `rescue` | Captures unexpected or incompatible data in a rescued data column |
| `failOnNewColumns` | Fails when new columns are detected |
| `none` | Does not automatically evolve the schema |

Availability and exact behavior can depend on the Databricks Runtime version and source configuration.

## 7. Schema Location vs Checkpoint Location

This is an important interview topic.

Both locations store metadata, but they serve different purposes.

| Schema Location | Checkpoint Location |
|---|---|
| Stores inferred schema information and schema history | Stores streaming query progress and recovery information |
| Supports schema inference and evolution | Supports recovery after failure |
| Configured using `cloudFiles.schemaLocation` | Configured using `checkpointLocation` |
| Used by Auto Loader | Used by Structured Streaming |

Example:

```python
.option("cloudFiles.schemaLocation", schema_path)
```

```python
.option("checkpointLocation", checkpoint_path)
```

**Remember:** Schema location and checkpoint location are not interchangeable.

For production pipelines, use persistent, durable storage locations. Do not delete or reuse checkpoint locations casually because doing so can affect file-processing guarantees and cause previously processed data to be considered again.

## 8. File Discovery: Directory Listing vs File Notification

Auto Loader supports different file-discovery approaches.

### A. Directory Listing

Auto Loader discovers files by listing the source directory and identifying new files.

- Useful for straightforward ingestion.
- Does not require configuring cloud notification infrastructure.
- Listing large directories can become expensive.

### B. File Notification Mode

Auto Loader can use cloud event notifications to discover newly arriving files.

For example:

- Azure Event Grid and queue-based notification infrastructure.
- AWS SQS and related S3 event notifications.
- Google Cloud notification infrastructure.

Advantages:

- More efficient discovery for large-scale ingestion.
- Avoids repeatedly listing large directories in suitable configurations.
- Useful when files arrive continuously.

The exact setup and supported notification mechanisms depend on the cloud provider and Databricks environment.

**Interview point:** Directory listing and file notification are discovery mechanisms. They do not change the basic purpose of Auto Loader: incrementally ingesting new files.

## 9. Handling Unexpected Data with the Rescued Data Column

Sometimes incoming files contain unexpected columns or values that do not match the expected schema.

Auto Loader can capture certain unexpected data in a rescued data column instead of simply discarding it.

Example:

```python
df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", schema_path)
    .option("rescuedDataColumn", "_rescued_data")
    .load(source_path)
)
```

The `_rescued_data` column can contain data that could not be mapped correctly to the expected schema.

This helps with:

- Identifying schema mismatches.
- Investigating unexpected fields.
- Avoiding silent loss of certain unexpected data.

**Important:** A rescued data column is not a replacement for data-quality validation. You should still monitor and investigate rescued records.

## 10. Auto Loader vs COPY INTO

Both Auto Loader and `COPY INTO` can incrementally load files into Delta tables, but their usage patterns differ.

| Feature | Auto Loader | COPY INTO |
|---|---|---|
| Main purpose | Scalable incremental file ingestion | Incremental file loading |
| Interface | Structured Streaming source | SQL command |
| Processing | Supports streaming and micro-batch execution | Batch-oriented command |
| File tracking | Uses streaming checkpoint and file-discovery metadata | Tracks files loaded by the command |
| Schema evolution | Supports configurable schema evolution | Has different schema-handling capabilities |
| Best suited for | Continuously arriving files and scalable pipelines | Simpler scheduled or batch file ingestion |

Example `COPY INTO`:

```sql
COPY INTO bronze.customers
FROM '/mnt/raw/customers/'
FILEFORMAT = CSV
FORMAT_OPTIONS ('header' = 'true')
COPY_OPTIONS ('mergeSchema' = 'true');
```

**Interview point:** Choose Auto Loader when you need scalable, incremental ingestion of continuously arriving files. `COPY INTO` is useful for simpler batch-oriented incremental loading.

Exact options and behavior depend on the Databricks environment and supported file formats.

## 11. Auto Loader vs Standard Spark File Streaming

| Feature | Standard File Streaming | Auto Loader |
|---|---|---|
| Source | `spark.readStream.format("json")` | `spark.readStream.format("cloudFiles")` |
| File discovery | Uses standard file-streaming discovery | Provides Auto Loader discovery mechanisms |
| Schema handling | Typically requires explicit schema for reliable streaming ingestion | Supports schema inference and evolution |
| Scalability | Depends on the file source and discovery configuration | Designed for scalable cloud-file ingestion |
| Checkpointing | Uses Structured Streaming checkpointing | Uses Structured Streaming checkpointing |
| Best use | Simpler streaming file sources | Large-scale cloud-file ingestion |

## 12. Important Production Considerations

When implementing Auto Loader in a real project, consider the following.

### 1. Use Persistent Checkpoint Storage

Store checkpoints in durable cloud storage.

Avoid temporary locations that may disappear when compute is restarted or deleted.

### 2. Use a Separate Schema Location

Maintain a persistent schema location for each ingestion stream.

### 3. Avoid Sharing Checkpoints

Different streaming queries should not share the same checkpoint location.

### 4. Handle Schema Changes

Define a schema evolution strategy instead of allowing unexpected source changes to go unnoticed.

### 5. Monitor Failed Files and Queries

Monitor streaming query status, ingestion progress, rescued records, and malformed data.

### 6. Make Downstream Processing Safe to Retry

A streaming query may restart after failure. Design downstream operations to handle retries safely.

### 7. Choose the Right Trigger

For continuously arriving files, use an appropriate processing trigger. For scheduled ingestion, consider `AvailableNow` where supported.

Example:

```python
query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .trigger(availableNow=True)
    .toTable("bronze.customers")
)
```

`AvailableNow` processes data available when the run starts, potentially in multiple micro-batches, and then stops.

## 13. Auto Loader in a Medallion Architecture

Auto Loader is commonly used to ingest source files into the Bronze layer.

```text
Source Systems
      |
      v
Cloud Storage
      |
      v
Auto Loader
      |
      v
Bronze Delta Table
      |
      v
Silver Delta Table
      |
      |-- Cleaning
      |-- Deduplication
      |-- Validation
      |-- Standardization
      |
      v
Gold Delta Table
      |
      v
Reporting / Analytics
```

### Bronze Layer

- Ingest raw source data.
- Preserve source information where appropriate.
- Add ingestion metadata if required.
- Store data in Delta format.

### Silver Layer

- Clean invalid records.
- Handle duplicates.
- Apply business rules.
- Standardize data types and formats.

### Gold Layer

- Create business-level aggregations.
- Build reporting tables.
- Prepare data for analytics and dashboards.

**Interview point:** Auto Loader handles file ingestion. It does not automatically implement the entire Bronze, Silver, and Gold transformation process.

## 14. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Using `format("csv")` instead of `format("cloudFiles")` for Auto Loader | Use `cloudFiles` as the source format and specify CSV using `cloudFiles.format` |
| Treating schema location as checkpoint location | They serve different purposes |
| Deleting checkpoints casually | Can affect recovery and cause reprocessing |
| Assuming schema evolution handles every schema change | Only supported changes are handled according to configuration |
| Assuming Auto Loader automatically cleans data | Data-quality transformations are still required |
| Assuming file notification is mandatory | Directory listing is also supported |
| Assuming Auto Loader processes every file exactly once under all circumstances | End-to-end guarantees depend on checkpointing, source, sink, and downstream operations |
| Sharing one checkpoint across multiple queries | Use a separate checkpoint for each streaming query |

## 15. Interview Questions and Answers

**Q1. What is Databricks Auto Loader?**

Auto Loader is a Databricks feature that incrementally discovers and ingests new files from cloud storage using Structured Streaming.

**Q2. Which format is used to configure Auto Loader?**

```python
.format("cloudFiles")
```

**Q3. What is the purpose of `cloudFiles.format`?**

It specifies the actual incoming file format, such as CSV, JSON, or Parquet.

**Q4. What is the difference between schema location and checkpoint location?**

Schema location stores schema information, while checkpoint location stores streaming progress and recovery information.

**Q5. What happens when a new column arrives?**

The behavior depends on the configured schema evolution mode. With `addNewColumns`, Auto Loader can update the schema and stop the stream so it can be restarted with the new schema.

**Q6. What are the two main file-discovery approaches?**

Directory listing and file notification.

**Q7. What is a rescued data column?**

It captures certain unexpected or incompatible source data that cannot be mapped to the expected schema.

**Q8. Can Auto Loader process files continuously?**

Yes. It supports Structured Streaming execution, including micro-batch processing. It can also be used for scheduled incremental ingestion.

**Q9. What happens if a streaming job fails?**

When restarted with its original checkpoint, Structured Streaming can recover its progress and continue processing according to the source and sink guarantees.

**Q10. What is the difference between Auto Loader and `COPY INTO`?**

Auto Loader is a Structured Streaming-based ingestion mechanism designed for scalable incremental file ingestion. `COPY INTO` is a SQL-based, batch-oriented incremental loading command.

**Q11. Does Auto Loader automatically remove duplicate records?**

No. File discovery and record-level deduplication are different concerns. Apply deduplication using appropriate business keys and logic.

**Q12. Where is Auto Loader commonly used in a data engineering pipeline?**

It is commonly used to ingest files from cloud storage into Bronze Delta tables.

## 16. Quick Revision

| Topic | Remember |
|---|---|
| Auto Loader | Incremental cloud-file ingestion |
| Source format | `cloudFiles` |
| Actual file format | `cloudFiles.format` |
| Schema location | Stores schema information |
| Checkpoint location | Stores streaming progress |
| Schema evolution | Handles supported schema changes |
| File discovery | Directory listing or file notification |
| Rescued data column | Captures certain unexpected data |
| Destination | Commonly a Delta table |
| Common architecture | Cloud Storage → Auto Loader → Bronze Delta |

### Important Code to Remember

```python
df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", schema_path)
    .load(source_path)
)

query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .outputMode("append")
    .start(target_path)
)
```

## 17. Self-Assessment

Before moving to the next topic, make sure you can answer these questions without looking at the notes:

- [ ] What is Auto Loader, and why do we use it?
- [ ] Explain the complete Auto Loader ingestion flow.
- [ ] Why do we use `cloudFiles`?
- [ ] Explain schema location vs checkpoint location.
- [ ] Explain schema inference and schema evolution.
- [ ] What are directory listing and file notification?
- [ ] What is a rescued data column?
- [ ] Compare Auto Loader with `COPY INTO`.
- [ ] How would you ingest files into a Bronze Delta table?
- [ ] What would you check if an Auto Loader pipeline failed?

**Interview readiness:** You should be able to explain Auto Loader in 2 minutes, write the basic ingestion code, and describe how schema evolution and checkpoint recovery work.