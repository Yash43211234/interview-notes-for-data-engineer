# Full vs Incremental Loads in Data Engineering

## 1. Introduction

Data pipelines need to move data from source systems to target systems.

There are two common loading strategies:

1. **Full Load:** Load the entire dataset from the source into the target.
2. **Incremental Load:** Load only new or changed data since the previous successful load.

The choice depends on data volume, update frequency, source capabilities, processing time, and business requirements.

---

## 2. Full Load

A full load reads and loads the complete dataset from the source during each execution.

It does not depend on identifying which records changed since the previous run.

### Example

Suppose an `employees` source table contains 100,000 records.

Every day, the pipeline reads all 100,000 records and loads them into the target table.

```text
Source Table
100,000 Records
       |
       v
Read All Records
       |
       v
Transform Data
       |
       v
Load Complete Dataset
       |
       v
Target Table
100,000 Records
```

### PySpark Example

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()

# Read complete source dataset
df = (
    spark.read
    .format("parquet")
    .load("/data/source/employees")
)

# Transform data
clean_df = df.dropDuplicates(["employee_id"])

# Replace target data
(
    clean_df.write
    .format("delta")
    .mode("overwrite")
    .saveAsTable("silver.employees")
)
```

This example performs a complete target replacement.

Use overwrite only when the incoming dataset is intended to represent the complete target contents and the write semantics are appropriate.

### Advantages

- Simple to implement.
- No need for watermark tracking.
- Can capture historical corrections and changes.
- Useful for small datasets.
- Useful for rebuilding a target table.

### Disadvantages

- Reads the entire dataset repeatedly.
- Consumes more compute and storage I/O.
- Can increase execution time.
- Becomes expensive for large datasets.
- Full replacement may affect downstream consumers if not managed carefully.

### When to Use Full Load

- Small reference tables.
- Initial data loading.
- Complete table refreshes.
- Historical rebuilds.
- Sources without reliable change-tracking mechanisms.

---

## 3. Incremental Load

An incremental load processes only new or changed records since the previous successful load.

Instead of reading the entire source table, the pipeline identifies the records that need to be processed.

### Example

Suppose an employee table contains 100,000 records.

On the next day:
- 500 new employees are inserted.
- 200 existing employees are updated.

The incremental pipeline processes only the 700 changed records, assuming the source provides a reliable way to identify them.

```text
Source Table
100,000 Records
       |
       v
Identify New or Changed Records
       |
       v
700 Changed Records
       |
       v
Transform Data
       |
       v
Insert / Update Target
       |
       v
Target Table
```

### PySpark Example: Timestamp-Based Incremental Load

Assume the source contains an `updated_at` column.

```python
from pyspark.sql import functions as F

last_watermark = "2026-10-24 00:00:00"

source_df = spark.read.format("parquet").load(
    "/data/source/employees"
)

incremental_df = source_df.filter(
    F.col("updated_at") > F.to_timestamp(
        F.lit(last_watermark)
    )
)
```

The pipeline now selects records whose update timestamp is later than the stored watermark.

The watermark must represent successfully processed progress, not simply the latest timestamp observed in the source.

### Advantages

- Processes fewer records.
- Reduces repeated source reads.
- Usually improves processing time for large datasets with relatively few changes.
- Reduces unnecessary compute and I/O.
- Supports frequent data refreshes.

### Disadvantages

- More complex to implement.
- Requires reliable change tracking.
- Needs progress or watermark management.
- Late-arriving updates can be missed.
- Physical deletes need separate handling.
- Requires retry-safe target writes.

### When to Use Incremental Load

- Large transactional tables.
- Frequently updated business data.
- Daily or hourly pipelines.
- Sources that support timestamps, change tracking, CDC, or reliable file discovery.

---

## 4. Full Load vs Incremental Load

| Feature | Full Load | Incremental Load |
|---|---|---|
| Data processed | Complete dataset | New or changed data |
| Implementation | Simpler | More complex |
| Processing time | Usually higher for large datasets | Usually lower when changes are small |
| Compute usage | Often higher | Often lower |
| Change detection | Not required | Required |
| Watermark tracking | Usually not required | Commonly required |
| Historical corrections | Included if present in source | Need to be detected or reprocessed |
| Delete handling | Reflected by complete replacement | Requires delete detection |
| Best suited for | Small tables and full refreshes | Large, frequently changing datasets |

**Interview takeaway:** Full load is simpler, while incremental load is generally more efficient for large datasets when changes can be identified reliably.

---

## 5. Types of Incremental Loading

Incremental loading can use different mechanisms depending on the source system.

### 5.1 Timestamp-Based Loading

Uses a timestamp column such as:
- `created_at`
- `updated_at`
- `modified_date`

Example:

```sql
SELECT *
FROM employees
WHERE updated_at > '2026-10-24 00:00:00';
```

The pipeline extracts records modified after the previous watermark.

**Limitation:** If timestamps are missing, unreliable, or updated incorrectly, records may be missed.

### 5.2 ID-Based Loading

Uses an increasing identifier to identify newly inserted records.

Example:

```sql
SELECT *
FROM orders
WHERE order_id > 50000;
```

This works for new records when IDs increase reliably.

**Limitation:** It does not detect updates to existing records with older IDs.

### 5.3 Change Data Capture (CDC)

CDC captures database changes such as:
- INSERT
- UPDATE
- DELETE

It can use database logs or other source-supported mechanisms.

Example:

```text
Database
   |
   v
CDC Mechanism
   |
   v
Change Events
   |
   v
Data Pipeline
   |
   v
Target Table
```

CDC is useful when the pipeline needs to capture inserts, updates, and deletes without repeatedly scanning the entire source table.

The exact capabilities depend on the database and CDC implementation.

### 5.4 File-Based Incremental Loading

Processes newly arrived files instead of reading all historical files again.

Example:
- Day 1: `orders_2026_10_24.csv`
- Day 2: `orders_2026_10_25.csv`
- Day 3: `orders_2026_10_26.csv`

Databricks Auto Loader can discover and process newly arriving files using Structured Streaming.

**Important:** File discovery does not automatically perform business-level deduplication or identify updates to existing business records.

---

## 6. Watermark in Incremental Loading

A watermark is a stored progress value that indicates how far the pipeline has successfully processed source data.

It is commonly used in batch incremental pipelines.

### Example

Suppose the last successful watermark is:

```text
2026-10-24 00:00:00
```

The next execution selects records where:

```sql
WHERE updated_at > '2026-10-24 00:00:00'
```

After successful processing and validation, the pipeline advances the watermark.

### Workflow

```text
Read Last Watermark
        |
        v
Extract Changed Records
        |
        v
Transform Data
        |
        v
Write to Target
        |
        v
Validate Output
        |
        v
Update Watermark
```

### Important Rules

1. Read the previous successful watermark.
2. Extract records after that watermark.
3. Transform and validate the data.
4. Write the target successfully.
5. Advance the watermark only after the required target operation succeeds.
6. Make the target write retry-safe.

### What Happens If the Pipeline Fails?

Suppose the target write succeeds, but updating the watermark fails.

The next execution may read the same records again.

If the target uses an unsafe append operation, duplicates can occur.

Use idempotent writes or transaction-aware progress handling to make retries safe.

### Watermark vs Streaming Watermark

These are different concepts:

| Batch Incremental Watermark | Streaming Event-Time Watermark |
|---|---|
| Tracks extraction progress | Tracks event-time progress |
| Commonly stored in a control table | Maintained by Structured Streaming |
| Helps select new or changed source records | Helps manage late data and state cleanup |
| Used in batch incremental pipelines | Used in stateful streaming operations |

Do not confuse these two meanings of watermark.

---

## 7. Handling Inserts, Updates, and Deletes

Incremental pipelines must handle different types of source changes.

Suppose the source contains an employee record:

| employee_id | name | salary |
|---|---|---:|
| 101 | Rahul | 50000 |

Later, the employee's salary changes.

| employee_id | name | salary |
|---|---|---:|
| 101 | Rahul | 60000 |

A simple incremental append may insert another row instead of updating the existing record.

For a current-state target table, an upsert strategy is more appropriate.

### Using Delta MERGE

```sql
MERGE INTO silver.employees AS target
USING staging.employees AS source
ON target.employee_id = source.employee_id

WHEN MATCHED THEN
    UPDATE SET
        target.name = source.name,
        target.salary = source.salary

WHEN NOT MATCHED THEN
    INSERT (
        employee_id,
        name,
        salary
    )
    VALUES (
        source.employee_id,
        source.name,
        source.salary
    );
```

This handles inserts and updates for the matched business key.

**Important:** Deduplicate the source by business key before MERGE. Multiple source rows matching the same target row can cause ambiguous or failed operations.

### Handling Deletes

A normal timestamp-based query may not detect physical deletion because the deleted record is no longer available in the source table.

Common approaches include:

- CDC delete events.
- Soft-delete flags.
- Source audit tables.
- Periodic source-to-target reconciliation.

For example, a CDC event may identify that `employee_id = 101` was deleted.

The target can then apply the corresponding delete operation.

---

## 8. Late-Arriving Data and Overlapping Loads

Timestamp-based incremental loading can miss records if they arrive late or their timestamps are delayed.

### Example

The pipeline successfully processes data through October 24.

On October 26, a record arrives with an `updated_at` timestamp of October 23.

If the next extraction uses only:

```sql
WHERE updated_at > '2026-10-24'
```

the late record will be missed.

### Solution: Lookback Window

Read a small overlapping period before the previous watermark.

For example, read the previous two days again.

```python
from pyspark.sql import functions as F

last_watermark = "2026-10-24 00:00:00"

lookback_start = F.expr(
    "timestamp('2026-10-24 00:00:00') - INTERVAL 2 DAYS"
)

incremental_df = source_df.filter(
    F.col("updated_at") >= lookback_start
)
```

The target must handle repeated records safely, for example through deduplication and MERGE.

Other approaches include:
- CDC.
- Source-side change tracking.
- Periodic reconciliation.
- Reprocessing affected historical ranges.

A lookback window reduces the risk of missing late changes but does not guarantee detection of arbitrarily late records.

---

## 9. Initial Load and Subsequent Incremental Loads

Many production pipelines use a combination of both strategies.

### Step 1: Initial Full Load

Load the complete historical dataset.

```text
Source
  |
  v
Full Extraction
  |
  v
Initial Target Table
```

### Step 2: Subsequent Incremental Loads

After the initial load, process only newly inserted or changed records.

```text
Source Changes
      |
      v
Incremental Extraction
      |
      v
Transform
      |
      v
MERGE into Target
```

### Example

| Execution | Source Records | Records Processed |
|---|---:|---:|
| Initial load | 1,000,000 | 1,000,000 |
| Day 1 | 1,010,000 | 10,000 |
| Day 2 | 1,018,000 | 8,000 |
| Day 3 | 1,025,000 | 7,000 |

The initial load processes everything. Later executions process only the identified changes.

---

## 10. Choosing the Right Loading Strategy

Before selecting a loading strategy, consider:

### 1. Data Volume

Small tables may be suitable for full loads.

Large tables often benefit from incremental processing.

### 2. Change Frequency

If only a small percentage of records changes, incremental loading may be more efficient.

### 3. Source Capabilities

Check whether the source supports:
- Reliable modification timestamps.
- Increasing identifiers.
- CDC.
- Change tracking.
- File-arrival detection.

### 4. Delete Requirements

Determine how physical deletes will be captured.

### 5. Data Freshness

Consider whether the pipeline runs:
- Daily.
- Hourly.
- Every few minutes.
- Continuously.

### 6. Recovery Requirements

Ensure the pipeline can safely retry failed batches and reprocess historical data.

### Decision Table

| Situation | Suitable Strategy |
|---|---|
| Small reference table | Full load |
| Initial historical ingestion | Full load |
| Large table with reliable update timestamps | Incremental load |
| Database with CDC support | CDC-based incremental load |
| New files arriving continuously | File-based ingestion |
| Historical transformation correction | Reprocessing or backfill |
| Source does not provide reliable change tracking | Full load or reconciliation-based approach |

---

## 11. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Incremental load means only new records | It can include inserts, updates, and deletes |
| Increasing IDs detect all changes | They generally detect new records, not updates to older IDs |
| Timestamp loading always captures every change | Late or unreliable timestamps can cause missed records |
| Updating the watermark before writing the target is safe | A failed target write can cause data loss |
| Append is always suitable for incremental loading | Updates may create duplicates |
| MERGE automatically resolves duplicate source records | Source deduplication is still important |
| Incremental loading automatically detects deletes | Delete capture needs CDC, soft deletes, or reconciliation |
| Full load is always inefficient | It can be appropriate for small datasets and full refreshes |
| Auto Loader performs business-level deduplication | It primarily handles file discovery and ingestion |
| Batch watermark and streaming watermark are identical | They solve different problems |

---

## 12. Interview Questions and Answers

### Q1. What is a full load?

A full load reads and loads the entire source dataset into the target, usually replacing or rebuilding the target contents depending on the write strategy.

### Q2. What is an incremental load?

An incremental load processes only new or changed records since the previous successful execution.

### Q3. What is the main difference between full and incremental loading?

Full loading processes the complete dataset, while incremental loading processes only identified changes.

### Q4. Why is incremental loading preferred for large datasets?

It reduces repeated source reads, processing time, and compute usage when only a small portion of the dataset changes.

### Q5. What is a watermark?

A watermark in batch incremental loading is a stored progress value used to identify which source records should be extracted next.

### Q6. What happens if a pipeline fails after writing data but before updating its watermark?

The same records may be extracted again. The target write should be idempotent or transaction-aware to avoid duplicates.

### Q7. How do you handle updates in incremental loading?

Use a reliable change-detection mechanism and an upsert operation, such as Delta MERGE, when maintaining a current-state target table.

### Q8. How do you handle deletes?

Use CDC delete events, soft-delete flags, audit tables, or periodic reconciliation.

### Q9. What is the difference between timestamp-based loading and CDC?

Timestamp-based loading identifies records using modification timestamps. CDC captures source changes through a change-tracking mechanism, which can include inserts, updates, and deletes.

### Q10. What is a lookback window?

A lookback window re-reads a recent historical period to capture late-arriving or delayed updates. The target must safely handle repeated records.

### Q11. Can incremental loading miss records?

Yes. It can miss records due to unreliable timestamps, late-arriving data, physical deletes, incorrect watermark updates, or source-side change-tracking limitations.

### Q12. What is the difference between initial load and incremental load?

Initial load usually loads the complete historical dataset. Subsequent incremental loads process only new or changed records.

### Q13. Would you use full or incremental loading for a small dimension table?

A full load may be simpler and practical when the table is small and a complete refresh is inexpensive.

### Q14. How would you design a reliable incremental pipeline?

I would use a reliable change-detection mechanism, maintain a successful watermark, validate incoming data, deduplicate by business key, write using an idempotent strategy, and update progress only after successful target processing.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Full load | Process the complete dataset |
| Incremental load | Process new or changed data |
| Initial load | Usually a full historical load |
| Timestamp-based load | Uses modification timestamps |
| ID-based load | Uses increasing identifiers |
| CDC | Captures inserts, updates, and deletes |
| Watermark | Tracks batch extraction progress |
| Lookback window | Re-reads a recent period |
| MERGE | Supports insert and update logic |
| Soft delete | Marks a record as deleted |
| Reconciliation | Compares source and target data |
| Idempotency | Makes retries safe |

### One-Minute Interview Answer

Full load means processing the complete source dataset during each execution, whereas incremental load processes only new or changed records since the previous successful run.

For small tables and initial ingestion, full load is often simpler. For large transactional tables, incremental loading is generally more efficient because it reduces repeated reads and processing.

Incremental loading can use timestamps, increasing IDs, CDC, or file-arrival detection. I would maintain a watermark, validate incoming records, deduplicate by business key, and use an idempotent write strategy such as Delta MERGE when appropriate. I would also handle late-arriving records and deletes separately because a simple timestamp filter may not capture every change.

---

## 14. Self-Assessment

- [ ] Explain full load with an example.
- [ ] Explain incremental load with an example.
- [ ] Compare full and incremental loading.
- [ ] Explain timestamp-based loading.
- [ ] Explain ID-based loading.
- [ ] Explain CDC.
- [ ] Describe how a watermark works.
- [ ] Explain why watermark updates must follow successful target processing.
- [ ] Handle inserts, updates, and deletes.
- [ ] Explain late-arriving data and lookback windows.
- [ ] Explain initial full load followed by incremental loads.
- [ ] Design a retry-safe incremental pipeline.

**Interview readiness:** Be able to explain how you would load a table containing 100 million records initially and then process only the changes every day without missing updates or creating duplicates.