# Checkpoints and Schema Location in Databricks

## 1. Introduction

When building data ingestion pipelines using Databricks Auto Loader and Structured Streaming, two important concepts are:

1. **Checkpoint Location:** Stores streaming progress and recovery information.
2. **Schema Location:** Stores schema information used by Auto Loader for schema inference and evolution.

Both are important for building reliable ingestion pipelines, but they serve different purposes.

**Simple way to remember:**

- Checkpoint location answers: *Where did my streaming query stop, and how can it recover?*
- Schema location answers: *What schema has Auto Loader discovered for my incoming files?*

## 2. What Is Checkpointing?

Checkpointing is the process of saving streaming query progress and relevant state information to durable storage.

If a streaming job fails or the cluster restarts, Structured Streaming can use the checkpoint to recover and continue processing.

Without proper checkpointing, a streaming application may lose its progress information and could reprocess previously processed data.

### Example

Suppose an Auto Loader pipeline processes files from cloud storage.

| File | Status |
|---|---|
| orders_01.json | Processed |
| orders_02.json | Processed |
| orders_03.json | Processed |
| orders_04.json | Not yet processed |

The streaming job fails after processing the first three files.

When restarted with the same valid checkpoint, the query can recover its progress and continue with the remaining files, subject to the source and sink guarantees.

### Checkpoint Flow

```text
Incoming Files
      |
      v
Structured Streaming
      |
      v
Process Micro-Batches
      |
      v
Write Output
      |
      v
Update Checkpoint
      |
      v
Next Micro-Batch
```

The checkpoint contains information required for recovery, including source progress and, where applicable, state information for stateful operations.

## 3. What Does a Checkpoint Store?

A Structured Streaming checkpoint directory can contain several types of metadata.

| Component | Purpose |
|---|---|
| Offset log | Records source progress or offsets |
| Commit log | Records completed micro-batch commits |
| State store | Stores state for stateful operations |
| Metadata | Stores information about the streaming query |
| Source information | Supports recovery of source progress |

The exact files and directory structure depend on the Spark version, source, sink, and query operations.

### Important Files

You may encounter files and directories such as:

```text
checkpoint/
    metadata
    offsets/
    commits/
    sources/
    state/
```

These are internal streaming metadata. You should not manually modify or delete them while a query depends on that checkpoint.

**Important:** Not every streaming query has a `state` directory. Stateful operations such as aggregations, deduplication, and stream-stream joins may require state storage.

## 4. How to Configure Checkpoint Location

In PySpark Structured Streaming, configure checkpoint storage using:

```python
.option("checkpointLocation", checkpoint_path)
```

### Example

```python
checkpoint_path = "/Volumes/main/checkpoints/orders/"

query = (
    df.writeStream
    .format("delta")
    .outputMode("append")
    .option("checkpointLocation", checkpoint_path)
    .start("/Volumes/main/bronze/orders/")
)
```

Here:

- `df` is a streaming DataFrame.
- `checkpointLocation` stores streaming progress.
- `format("delta")` specifies Delta Lake as the sink.
- `start()` starts the streaming query.

Use a durable storage location accessible to the query.

## 5. What Happens When a Streaming Job Fails?

Consider this scenario:

1. A streaming job starts.
2. It processes several micro-batches.
3. The cluster unexpectedly terminates.
4. The job is restarted using the same checkpoint location.
5. Structured Streaming reads the saved progress and recovers the query.

```text
Streaming Job Starts
        |
        v
Processes Micro-Batches
        |
        v
Saves Checkpoint Information
        |
        v
Job Fails
        |
        v
Job Restarts
        |
        v
Reads Existing Checkpoint
        |
        v
Recovers Streaming Progress
        |
        v
Continues Processing
```

**Interview point:** Checkpointing supports fault tolerance and recovery. It does not automatically guarantee exactly-once results for every possible external sink.

## 6. What Happens If We Delete the Checkpoint?

Deleting a checkpoint removes the saved progress and state information associated with that query.

When the query starts again with a new or empty checkpoint location, Spark treats it as a new streaming query.

Possible consequences include:

- Previously processed files may be processed again.
- Source offsets may be read from the beginning or from a newly configured starting position.
- Stateful aggregations may lose their previous state.
- Duplicate records may appear in the destination.
- The query may produce different results from the original run.

**Example:**

Suppose a pipeline has already processed 1 million records.

If its checkpoint is deleted and the pipeline starts again from the beginning, those records may be ingested again.

The exact behavior depends on the source, starting-position configuration, and sink.

### Production Rule

Do not delete a checkpoint simply to fix a failed streaming query.

First investigate the failure. If a new checkpoint is genuinely required, plan how to handle reprocessing, duplicates, source offsets, and state recovery.

## 7. What Is Schema Location?

Schema location is a storage path used by Auto Loader to save inferred schema information and schema history.

Auto Loader uses this information when discovering and processing incoming files.

It is configured using:

```python
.option("cloudFiles.schemaLocation", schema_path)
```

### Example

Suppose incoming CSV files initially contain:

| customer_id | name | city |
|---|---|---|
| 101 | Rahul | Kolkata |
| 102 | Amit | Delhi |

Auto Loader discovers the columns and stores the schema information in the schema location.

Later, a new file arrives with an additional column:

| customer_id | name | city | country |
|---|---|---|---|
| 103 | Priya | Mumbai | India |

Depending on the configured schema evolution mode, Auto Loader can detect and handle this new column.

Schema location helps Auto Loader maintain the schema information across ingestion runs.

## 8. How to Configure Schema Location

### Example Using Auto Loader

```python
source_path = "/Volumes/main/raw/customer_files/"

schema_path = "/Volumes/main/checkpoints/customer_schema/"

checkpoint_path = "/Volumes/main/checkpoints/customer_stream/"

df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "csv")
    .option("cloudFiles.schemaLocation", schema_path)
    .option("header", "true")
    .load(source_path)
)
```

Here:

- `cloudFiles` selects Auto Loader.
- `cloudFiles.format` specifies CSV as the file format.
- `cloudFiles.schemaLocation` stores schema information.
- `header` indicates that the first CSV row contains column names.

**Important:** Schema location is an Auto Loader configuration. It is not a general requirement for every Structured Streaming source.

## 9. Schema Evolution and Schema Location

Schema evolution allows Auto Loader to handle supported changes in incoming file schemas.

For example, a new column may be added to incoming data.

### Example

Initial schema:

```text
customer_id
customer_name
city
```

New incoming schema:

```text
customer_id
customer_name
city
country
```

Configure schema evolution:

```python
df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", schema_path)
    .option("cloudFiles.schemaEvolutionMode", "addNewColumns")
    .load(source_path)
)
```

With `addNewColumns`, a newly detected column can update the stored schema and cause the streaming query to stop with a schema evolution error. Restarting the query allows it to continue using the updated schema.

### Common Schema Evolution Modes

| Mode | Behavior |
|---|---|
| `addNewColumns` | Adds newly detected columns; the stream may stop and require restart |
| `addNewColumnsWithTypeWidening` | Supports certain type-widening changes where available |
| `rescue` | Captures unexpected or incompatible data in a rescued data column |
| `failOnNewColumns` | Fails when new columns are detected |
| `none` | Does not automatically evolve the schema |

Exact behavior and availability depend on the Databricks Runtime version and configuration.

## 10. Schema Location vs Checkpoint Location

This is one of the most important Auto Loader interview questions.

| Feature | Checkpoint Location | Schema Location |
|---|---|---|
| Main purpose | Streaming recovery | Schema management |
| Stores | Query progress and state information | Inferred schema and schema history |
| Used by | Structured Streaming | Auto Loader |
| Configuration | `checkpointLocation` | `cloudFiles.schemaLocation` |
| Required for | Reliable recovery of streaming queries | Auto Loader schema tracking |
| Shared between queries? | No, use a separate checkpoint for each query | Use an appropriate schema location for each ingestion source/schema |

### Simple Example

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
    .start(target_path)
)
```

In this example:

- `schema_path` stores schema information.
- `checkpoint_path` stores streaming progress and recovery information.

**Never confuse these two paths.**

## 11. Can We Use the Same Location for Both?

No. Keep schema location and checkpoint location separate.

They serve different purposes and contain different metadata.

Recommended structure:

```text
Databricks Storage
|
|-- checkpoints/
|   |
|   |-- orders_stream/
|   |   |-- offsets/
|   |   |-- commits/
|   |
|   |-- customers_stream/
|       |-- offsets/
|       |-- commits/
|
|-- schemas/
    |
    |-- orders/
    |
    |-- customers/
```

This structure keeps query recovery information separate from Auto Loader schema information.

For production pipelines, use dedicated, persistent paths and avoid reusing them for unrelated queries.

## 12. Can Multiple Queries Share a Checkpoint?

No. Each independent streaming query should have its own checkpoint location.

For example:

```python
orders_checkpoint = "/Volumes/main/checkpoints/orders/"

customers_checkpoint = "/Volumes/main/checkpoints/customers/"
```

Why?

- Each query has its own progress.
- Each query may have different source offsets.
- Each query may maintain different state.
- Sharing a checkpoint can cause recovery failures and inconsistent behavior.

Use separate checkpoint locations even if both queries write to the same Delta table, unless they are deliberately designed as a single managed query.

## 13. Checkpointing and Exactly-Once Processing

Checkpointing is important for reliable streaming, but it is not the same as exactly-once processing.

### What Checkpointing Provides

- Saves query progress.
- Supports recovery after failure.
- Helps avoid unnecessary reprocessing when the source and sink support the required recovery behavior.
- Preserves state for supported stateful operations.

### What Exactly-Once Means

Exactly-once semantics means the final committed results are correct without unintended duplicate or missing effects within the guarantees of the complete pipeline.

It does not mean Spark never executes the same computation more than once.

For example:

- A micro-batch may be retried after a failure.
- An external API call may happen again if it is not idempotent.
- A custom sink may need its own transaction or deduplication mechanism.

Delta Lake supports transactional writes for supported Structured Streaming integrations, helping provide exactly-once results when used correctly.

**Remember:** Checkpointing supports recovery; end-to-end exactly-once behavior depends on the source, processing engine, and sink.

## 14. Production Best Practices

### Checkpoint Best Practices

1. Use persistent cloud storage.
2. Assign a unique checkpoint path to each independent streaming query.
3. Do not manually edit checkpoint files.
4. Do not delete checkpoints without a recovery plan.
5. Monitor query progress and failures.
6. Ensure the query can safely handle retries.
7. Keep checkpoint paths stable across normal job restarts.

### Schema Location Best Practices

1. Use a persistent location.
2. Keep schema information separate from checkpoint data.
3. Use a suitable schema evolution mode.
4. Monitor new columns and unexpected type changes.
5. Review rescued data.
6. Avoid reusing a schema location for unrelated ingestion sources.

### Example Production Configuration

```python
source_path = "/Volumes/main/raw/orders/"

schema_path = "/Volumes/main/schemas/orders/"

checkpoint_path = "/Volumes/main/checkpoints/orders/"

target_table = "main.bronze.orders"

df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option("cloudFiles.schemaLocation", schema_path)
    .option("cloudFiles.schemaEvolutionMode", "addNewColumns")
    .load(source_path)
)

query = (
    df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path)
    .outputMode("append")
    .trigger(availableNow=True)
    .toTable(target_table)
)
```

This example:

- Reads incoming JSON files using Auto Loader.
- Maintains schema information separately.
- Tracks streaming progress through a checkpoint.
- Writes data into a Bronze Delta table.
- Processes available files and stops using `AvailableNow`.

For a production implementation, also configure permissions, data-quality checks, monitoring, and safe restart procedures.

## 15. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Treating schema location and checkpoint location as the same | They store different information |
| Deleting checkpoints to fix every failure | Investigate the failure first |
| Sharing one checkpoint between independent queries | Use a separate checkpoint for each query |
| Storing checkpoints in temporary local storage | Use persistent, durable storage |
| Assuming schema location tracks processed files | Checkpointing and Auto Loader file-discovery metadata handle ingestion progress |
| Assuming checkpointing guarantees exactly-once for every sink | End-to-end guarantees depend on source and sink |
| Assuming every streaming query needs schema location | Schema location is specifically used by Auto Loader for schema tracking |
| Assuming schema evolution accepts every change | Only supported changes are handled according to the configured mode |
| Manually editing checkpoint files | Let Structured Streaming manage its checkpoint |

## 16. Interview Questions and Answers

**Q1. What is checkpointing in Spark Structured Streaming?**

Checkpointing stores streaming query progress and relevant state information so the query can recover after failure.

**Q2. Why do we need a checkpoint location?**

It allows the streaming query to recover its progress and continue processing after an interruption.

**Q3. What does a checkpoint store?**

It can store source progress, micro-batch commit information, query metadata, and state information for stateful operations.

**Q4. What happens if we delete the checkpoint?**

The query loses its saved progress and state. Starting with a new checkpoint creates a new query execution history and may cause data to be reprocessed.

**Q5. What is schema location in Auto Loader?**

It is the location where Auto Loader stores inferred schema information and schema history.

**Q6. What is the difference between schema location and checkpoint location?**

Schema location manages schema information, whereas checkpoint location manages streaming progress and recovery.

**Q7. Can multiple streaming queries share one checkpoint?**

No. Independent queries should use separate checkpoint locations.

**Q8. Is schema location required for every Structured Streaming query?**

No. It is an Auto Loader configuration used for schema tracking.

**Q9. Does checkpointing guarantee exactly-once processing?**

Not by itself. End-to-end guarantees depend on the source and sink. Delta Lake provides transactional support for supported streaming integrations.

**Q10. What happens when a new column arrives in Auto Loader?**

The behavior depends on the schema evolution mode. With `addNewColumns`, Auto Loader can update its schema and stop the query so it can be restarted.

**Q11. Why should checkpoint storage be persistent?**

Because the query needs its saved progress and state information after cluster restarts or failures.

**Q12. Can we manually modify checkpoint files?**

No. Checkpoint files are internal metadata managed by Structured Streaming.

## 17. Quick Revision

| Topic | Remember |
|---|---|
| Checkpointing | Saves streaming progress and state |
| Checkpoint location | Recovery metadata storage |
| Schema location | Auto Loader schema metadata storage |
| `checkpointLocation` | Structured Streaming option |
| `cloudFiles.schemaLocation` | Auto Loader option |
| Offset log | Tracks source progress |
| Commit log | Tracks completed micro-batches |
| State store | Stores state for stateful operations |
| New checkpoint | Starts a new query history |
| New schema | Handled according to schema evolution mode |
| Multiple queries | Separate checkpoint locations |
| Exactly-once | Depends on source, processing, and sink guarantees |

### One-Minute Interview Answer

Checkpoint location and schema location are two important configurations in Databricks Auto Loader pipelines.

Checkpoint location stores streaming progress and state information. It helps Structured Streaming recover after a failure and continue processing.

Schema location stores inferred schema information and schema history. Auto Loader uses it to manage incoming data schemas and supported schema changes.

Both should use persistent storage and should be maintained separately. Each independent streaming query should have its own checkpoint location.

## 18. Self-Assessment

- [ ] Explain checkpointing in Structured Streaming.
- [ ] What information is stored in a checkpoint?
- [ ] What happens when a streaming job fails?
- [ ] What happens if the checkpoint is deleted?
- [ ] Explain schema location in Auto Loader.
- [ ] Differentiate checkpoint location and schema location.
- [ ] Why should every independent query have a separate checkpoint?
- [ ] Explain how schema evolution uses schema location.
- [ ] Does checkpointing alone guarantee exactly-once processing?
- [ ] Write a complete Auto Loader pipeline with both locations configured.

**Interview readiness:** You should be able to explain checkpointing and schema location in 2 minutes, write the configuration correctly, and describe what happens when a streaming job fails or restarts.