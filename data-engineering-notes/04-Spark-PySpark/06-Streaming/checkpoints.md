# Checkpointing in Apache Spark

## 1. What Is Checkpointing?

Checkpointing is a technique in Spark used to save the state of a computation to reliable storage.

It helps Spark truncate a long execution lineage and recover from failures without recomputing the entire chain of transformations.

**Key idea:** Checkpointing saves data to reliable storage and creates a new starting point for future computation.

It is especially useful when:
- A DataFrame or RDD has a very long lineage.
- Recomputing previous transformations is expensive.
- A long-running computation needs a reliable recovery point.
- Iterative algorithms repeatedly build up execution dependencies.

---

## 2. What Is Lineage in Spark?

Lineage is the record of transformations used to produce a DataFrame or RDD.

Spark uses lineage to understand how data was created and how to recompute it if necessary.

### Example

```python
df1 = spark.read.parquet("/data/orders")

df2 = df1.filter("amount > 1000")

df3 = df2.groupBy("customer_id").count()

df4 = df3.filter("count > 5")
```

Conceptually, the lineage is:

```text
Read Data
   |
   v
Filter
   |
   v
GroupBy
   |
   v
Filter
   |
   v
Final DataFrame
```

If an intermediate partition is lost, Spark can use lineage to recompute the required data from its dependencies.

### Problem with long lineage

Imagine a pipeline with hundreds of transformations.

Recomputing lost data may become expensive because Spark needs to execute many earlier operations again.

Checkpointing helps by saving an intermediate result and truncating the previous lineage.

---

## 3. Why Do We Need Checkpointing?

Checkpointing is useful for:

1. Truncating long lineage.
2. Reducing repeated recomputation.
3. Improving recovery from failures.
4. Supporting long-running computations.
5. Providing a reliable recovery point.

### Example scenario

Suppose a pipeline performs:

```text
Read
  |
  v
Filter
  |
  v
Join
  |
  v
Aggregation
  |
  v
Transformation
  |
  v
Another Join
  |
  v
Final Result
```

If this pipeline becomes very large, Spark may need to follow a long dependency chain during recomputation.

You can checkpoint an intermediate DataFrame:

```text
Read
  |
  v
Filter
  |
  v
Join
  |
  v
Aggregation
  |
  v
CHECKPOINT
  |
  v
New Lineage
  |
  v
Transformation
  |
  v
Final Result
```

After checkpointing, Spark can use the saved result rather than depending on the entire earlier transformation chain.

---

## 4. Types of Checkpointing

Spark provides two important checkpointing approaches:

1. Reliable Checkpointing.
2. Local Checkpointing.

### A. Reliable Checkpointing

Reliable checkpointing saves the computed data to a reliable, fault-tolerant storage location.

Examples include:
- HDFS.
- Supported cloud storage.
- Other reliable distributed storage systems.

It truncates the previous lineage and provides a recovery point independent of the original computation chain.

### PySpark example

First, configure the checkpoint directory:

```python
spark.sparkContext.setCheckpointDir(
    "/tmp/spark-checkpoints"
)
```

Create a DataFrame:

```python
df = spark.read.parquet("/data/orders")

filtered_df = df.filter("amount > 1000")
```

Apply checkpointing:

```python
checkpointed_df = filtered_df.checkpoint(
    eager=True
)
```

Here:

- `setCheckpointDir()` specifies where checkpoint files are stored.
- `checkpoint()` creates a checkpoint.
- `eager=True` computes and saves the checkpoint immediately.

**Important:** The checkpoint directory must be accessible to the Spark application. In a distributed environment, use suitable shared storage rather than assuming a local path is accessible to every executor.

### B. Local Checkpointing

Local checkpointing stores the computed data using the caching subsystem, typically on executor-local storage.

Example:

```python
local_df = filtered_df.localCheckpoint(
    eager=True
)
```

It can truncate lineage with less overhead than reliable checkpointing.

However, it is less reliable because the checkpointed data is not stored in fault-tolerant shared storage. If the required executor-local data is lost, Spark may not be able to recover it.

**Remember:** Local checkpointing trades reliability for potentially lower overhead.

---

## 5. Reliable Checkpoint vs Local Checkpoint

| Feature | Reliable Checkpoint | Local Checkpoint |
|---|---|---|
| Storage | Reliable checkpoint storage | Executor-local storage through caching |
| Fault tolerance | Higher | Lower |
| Lineage truncation | Yes | Yes |
| Recovery after executor loss | Can recover from reliable checkpoint data | May fail if local checkpoint data is lost |
| Storage overhead | Depends on checkpoint storage | Uses caching resources |
| Typical use | Reliable recovery and long lineage | Lower-overhead lineage truncation when data-loss risk is acceptable |

### Interview answer

Reliable checkpointing saves data to reliable storage and provides a more fault-tolerant recovery point. Local checkpointing uses the caching subsystem and executor-local storage, so it is generally faster or lower overhead but less reliable if executor data is lost.

---

## 6. What Is Eager and Lazy Checkpointing?

PySpark DataFrame checkpointing supports an `eager` parameter.

### Eager checkpointing

```python
checkpointed_df = df.checkpoint(
    eager=True
)
```

Spark computes and saves the checkpoint immediately.

### Non-eager checkpointing

```python
checkpointed_df = df.checkpoint(
    eager=False
)
```

The checkpoint is not immediately materialized by the method call. It is computed when an action requires the result.

**Important:** Checkpointing is not a normal transformation that can simply be ignored. It introduces a materialization requirement, either immediately or when execution is triggered.

---

## 7. Checkpointing vs Caching

These concepts are often confused.

### Caching

Caching stores computed data so it can be reused without repeating the same computation.

```python
cached_df = df.cache()

cached_df.count()
```

Caching:
- Helps when data is reused.
- Uses memory and/or disk depending on the storage level.
- Does not normally truncate lineage.
- Can lose cached partitions and recompute them using lineage.

### Checkpointing

Checkpointing saves data and truncates the previous lineage.

```python
checkpointed_df = df.checkpoint(
    eager=True
)
```

Checkpointing:
- Helps manage long lineage.
- Creates a new recovery point.
- Requires writing checkpoint data.
- Can avoid recomputing the entire previous transformation chain.

### Comparison

| Feature | Cache | Checkpoint |
|---|---|---|
| Main purpose | Reuse computed data | Truncate lineage |
| Stores computed data | Yes | Yes |
| Truncates lineage | No | Yes |
| Recovery | Can recompute using lineage | Depends on checkpoint type |
| Storage | Memory and/or disk | Reliable storage or local caching subsystem |
| Typical use | Repeated DataFrame reuse | Long lineage and recovery |

**Interview point:** Cache improves reuse; checkpointing truncates lineage.

---

## 8. Checkpointing vs Persisting

`persist()` allows you to choose a storage level for retaining computed data.

Example:

```python
from pyspark import StorageLevel

persisted_df = df.persist(
    StorageLevel.MEMORY_AND_DISK
)
```

Checkpointing and persisting solve different problems.

| Feature | persist() | checkpoint() |
|---|---|---|
| Main purpose | Retain data for reuse | Truncate lineage |
| Storage control | Storage level can be selected | Checkpoint storage is managed by Spark |
| Lineage truncation | No | Yes |
| Typical use | Repeated computation | Long transformation chains |

---

## 9. Checkpointing in RDDs vs DataFrames

Spark supports checkpointing for both RDDs and DataFrames, but their APIs differ.

### RDD checkpointing

```python
rdd = spark.sparkContext.parallelize(
    [1, 2, 3, 4, 5]
)

spark.sparkContext.setCheckpointDir(
    "/tmp/spark-checkpoints"
)

rdd.checkpoint()

rdd.count()
```

The RDD checkpoint operation marks the RDD for checkpointing. An action triggers its computation and checkpointing.

### DataFrame checkpointing

```python
df = spark.read.parquet("/data/orders")

checkpointed_df = df.checkpoint(
    eager=True
)
```

For DataFrames, `checkpoint()` returns a new DataFrame backed by the checkpointed result.

**Important:** Do not confuse the RDD and DataFrame APIs. Their execution and checkpoint materialization details differ.

---

## 10. When Should You Use Checkpointing?

Use checkpointing when:

1. A transformation chain becomes very long.
2. Recomputing earlier transformations is expensive.
3. An iterative algorithm repeatedly extends lineage.
4. You need a reliable intermediate recovery point.
5. Long-running processing creates complex dependencies.

### When should you avoid it?

Avoid unnecessary checkpointing when:
- The pipeline is simple.
- The DataFrame is used only once.
- The lineage is already manageable.
- The cost of writing checkpoint data outweighs the benefit.

Checkpointing introduces storage I/O, so using it everywhere can reduce performance.

---

## 11. Common Mistakes

1. Assuming checkpointing and caching are the same.
2. Assuming caching truncates lineage.
3. Assuming local checkpointing is as fault-tolerant as reliable checkpointing.
4. Forgetting to configure a checkpoint directory for reliable checkpointing.
5. Checkpointing every intermediate DataFrame unnecessarily.
6. Assuming `checkpoint(eager=False)` immediately saves the data.
7. Using executor-local paths as if they were shared distributed storage.
8. Ignoring the storage and I/O cost of checkpointing.
9. Confusing RDD checkpointing with DataFrame checkpointing.

---

## 12. Interview Questions

**Q1. What is checkpointing in Spark?**

Checkpointing saves computed data to storage and truncates the previous lineage, helping reduce recomputation and improve recovery.

**Q2. What is lineage?**

Lineage is the sequence of transformations and dependencies used to produce a DataFrame or RDD.

**Q3. Why do we need checkpointing?**

It helps manage long lineage, reduce recomputation, and provide recovery points for long-running computations.

**Q4. What are the two types of checkpointing?**

Reliable checkpointing and local checkpointing.

**Q5. What is the difference between reliable and local checkpointing?**

Reliable checkpointing uses fault-tolerant storage, while local checkpointing uses executor-local storage through the caching subsystem and is less reliable if executor data is lost.

**Q6. Does checkpointing truncate lineage?**

Yes. Both reliable and local checkpointing truncate the previous lineage.

**Q7. Does caching truncate lineage?**

No. Caching stores computed data for reuse but normally preserves lineage.

**Q8. What is eager checkpointing?**

Eager checkpointing computes and saves the checkpoint immediately.

**Q9. When would you use checkpointing instead of caching?**

When the main requirement is to truncate a long lineage or establish a recovery point rather than simply reuse computed data.

**Q10. Does checkpointing improve performance automatically?**

No. It adds storage I/O. It can improve overall performance when the cost of repeated recomputation is greater than the checkpointing overhead.

**Q11. Where does reliable checkpointing store data?**

In a configured reliable storage location, such as HDFS or supported cloud storage.

**Q12. What happens if local checkpoint data is lost?**

Spark may not be able to recompute the lost data because the previous lineage has been truncated.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Lineage | Transformation dependency chain |
| Checkpointing | Saves data and truncates lineage |
| Reliable checkpoint | Stores data in reliable storage |
| Local checkpoint | Uses executor-local storage through caching |
| Eager checkpoint | Computes checkpoint immediately |
| Lazy checkpoint | Computes when an action requires it |
| Cache | Reuses computed data |
| Persist | Retains data using a selected storage level |
| Main benefit | Reduces recomputation and manages long lineage |

### One-line revision

- Cache → Reuse computed data.
- Persist → Retain data with a selected storage level.
- Checkpoint → Save data and truncate lineage.
- Reliable checkpoint → Better fault tolerance.
- Local checkpoint → Lower overhead, but less reliable.

---

## 14. Self-Check

- [ ] Can I explain checkpointing?
- [ ] Can I explain Spark lineage?
- [ ] Can I explain why long lineage can be expensive?
- [ ] Can I differentiate reliable and local checkpointing?
- [ ] Can I explain eager and non-eager checkpointing?
- [ ] Can I differentiate cache, persist, and checkpoint?
- [ ] Can I configure a checkpoint directory?
- [ ] Can I explain when checkpointing is useful?
- [ ] Can I explain the limitations of local checkpointing?

**Final takeaway:** Checkpointing saves an intermediate result and truncates the previous lineage. Use reliable checkpointing when fault tolerance matters, and use local checkpointing only when its lower reliability is acceptable.