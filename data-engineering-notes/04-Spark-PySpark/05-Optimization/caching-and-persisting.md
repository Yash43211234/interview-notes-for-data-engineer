# Caching and Persisting in Apache Spark

## 1. What Is Caching?

Caching is a technique used to store a DataFrame's computed data so that Spark can reuse it in subsequent operations instead of recomputing it from the original source.

Spark uses lazy evaluation. Transformations are not executed immediately.

When an action triggers execution, Spark calculates the required data. If the DataFrame is cached, Spark can retain its computed partitions for reuse.

**Example:**

```python
df = spark.read.parquet("/data/sales")

cached_df = df.cache()

cached_df.count()
```

The `count()` action triggers execution and materializes the cached DataFrame.

If you use the same cached DataFrame again, Spark can reuse its cached partitions when available.

## 2. Why Do We Need Caching?

Suppose you have a DataFrame that requires expensive transformations.

```python
df = spark.read.parquet("/data/sales")

transformed_df = df.filter(
    "amount > 1000"
).groupBy(
    "region"
).sum("amount")
```

Now suppose you perform multiple actions:

```python
transformed_df.show()

transformed_df.count()

transformed_df.write.parquet(
    "/output/sales"
)
```

Without caching, Spark may need to recompute parts of the transformation lineage for separate actions.

If the transformed DataFrame is expensive to calculate and reused multiple times, caching can reduce repeated computation.

```python
transformed_df = transformed_df.cache()

transformed_df.count()

transformed_df.show()

transformed_df.write.parquet(
    "/output/sales"
)
```

**Main benefit:** Avoids unnecessary recomputation when the same intermediate DataFrame is reused.

## 3. What Is Persisting?

`persist()` is used to store a DataFrame's computed partitions using a selected storage level.

It gives more control over how Spark stores the cached data.

Example:

```python
from pyspark import StorageLevel

df.persist(
    StorageLevel.MEMORY_AND_DISK
)
```

The storage level determines where and how Spark stores the data.

Examples:
- Memory.
- Disk.
- Memory and disk.
- Serialized or deserialized representation, depending on the storage level.

## 4. Cache vs Persist

In PySpark, `cache()` is a convenient method that uses a default storage level.

`persist()` allows you to specify a storage level.

| Feature | cache() | persist() |
|---|---|---|
| Purpose | Reuse computed data | Reuse computed data |
| Storage level | Uses default level | Allows explicit storage level |
| Customization | Limited | More control |
| Use case | Simple caching | When storage behavior needs control |

Example:

```python
df.cache()
```

```python
from pyspark import StorageLevel

df.persist(
    StorageLevel.MEMORY_AND_DISK
)
```

**Important:** Both methods are lazy. Calling them marks the DataFrame for persistence; an action is generally required to materialize the cached data.

## 5. Common Storage Levels

Spark provides different storage levels.

| Storage level | Meaning |
|---|---|
| MEMORY_ONLY | Store deserialized data in memory |
| MEMORY_AND_DISK | Store data in memory and spill partitions that do not fit to disk |
| MEMORY_ONLY_SER | Store serialized data in memory |
| MEMORY_AND_DISK_SER | Store serialized data in memory and disk |
| DISK_ONLY | Store data on disk |

The availability and exact behavior of storage-level options can depend on the Spark API and version.

### A. MEMORY_ONLY

```python
from pyspark import StorageLevel

df.persist(
    StorageLevel.MEMORY_ONLY
)
```

- Stores cached partitions in memory.
- Data is generally stored in deserialized form for DataFrames.
- Partitions that cannot fit in memory may not remain cached.
- Missing partitions can be recomputed when needed.

### B. MEMORY_AND_DISK

```python
df.persist(
    StorageLevel.MEMORY_AND_DISK
)
```

- Stores data in memory when possible.
- Stores partitions that do not fit in memory on disk.
- Reduces the need to recompute partitions that could not fit in memory.

### C. DISK_ONLY

```python
df.persist(
    StorageLevel.DISK_ONLY
)
```

- Stores cached data on disk.
- Useful when memory is limited but reuse is still beneficial.
- Disk access is generally slower than memory access.

## 6. What Happens Internally When We Cache?

Consider:

```python
df = spark.read.parquet("/data/sales")

cached_df = df.filter(
    "amount > 1000"
).cache()

cached_df.count()
```

Execution flow:

```text
Read Source Data
       |
Filter Records
       |
Cache Requested
       |
count() Action
       |
Execute Transformation
       |
Store Computed Partitions
       |
Reuse Cached Data
```

Important points:

1. Spark marks the DataFrame for persistence.
2. The first action triggers computation.
3. Computed partitions are stored according to the selected storage level.
4. Subsequent operations can reuse available cached partitions.
5. If a partition is missing or evicted, Spark may recompute it from lineage.

**Important:** Caching does not mean the entire DataFrame is guaranteed to remain in memory.

## 7. Cache Materialization

Caching is lazy.

Example:

```python
df.cache()
```

This does not immediately execute the DataFrame.

To materialize the cache:

```python
df.count()
```

Other actions can also materialize the cached DataFrame.

For example:

```python
df.show()
```

However, an action may only evaluate the data it needs. Do not assume every action necessarily materializes every partition of the complete DataFrame.

A full scan such as `count()` is commonly used when the intention is to materialize all partitions.

## 8. When Should You Use Cache or Persist?

Use caching when:

1. The same DataFrame is reused multiple times.
2. Transformations are expensive.
3. An intermediate DataFrame is used in multiple downstream operations.
4. Recomputing data would be more expensive than storing it.
5. Iterative algorithms repeatedly access the same dataset.

Example:

```python
filtered_df = spark.read.parquet(
    "/data/transactions"
).filter(
    "status = 'SUCCESS'"
).cache()

filtered_df.count()

# Reuse the same filtered DataFrame
filtered_df.groupBy("region").count().show()

filtered_df.groupBy("payment_type").sum(
    "amount"
).show()
```

## 9. When Should You Avoid Caching?

Avoid caching when:

- The DataFrame is used only once.
- The transformation is inexpensive.
- The dataset is too large for available storage resources.
- Caching creates unnecessary memory pressure.
- The DataFrame is already efficiently reused through another mechanism.

**Interview point:** Caching everything is not an optimization. It can reduce performance by consuming memory needed for other Spark operations.

## 10. How to Remove Cached Data?

Use `unpersist()` when cached data is no longer needed.

```python
df.unpersist()
```

By default, `unpersist()` is non-blocking.

If you want Spark to wait for the removal operation:

```python
df.unpersist(blocking=True)
```

For example:

```python
filtered_df = df.filter(
    "amount > 1000"
).cache()

filtered_df.count()

# Perform required operations

filtered_df.unpersist()
```

This helps release storage resources for other operations.

## 11. Cache and Spark Memory Management

Spark uses memory for different purposes, including:

- Execution memory: Used by operations such as joins, sorting, and aggregations.
- Storage memory: Used for cached or persisted data.

Spark's unified memory management allows execution and storage memory to share a common region.

Cached data may be evicted when execution requires memory, subject to Spark's memory-management rules.

Important points:

- Cached data is not permanently protected.
- Memory pressure can cause cached partitions to be evicted.
- Disk-backed storage levels can retain partitions on disk.
- Missing cached partitions can be recomputed using lineage.

This is why choosing the correct storage level matters.

## 12. Cache vs Checkpoint

Caching and checkpointing are different concepts.

| Feature | Cache / Persist | Checkpoint |
|---|---|---|
| Main purpose | Reuse computed data | Save data and truncate lineage |
| Storage | Memory and/or disk | Checkpoint storage |
| Lineage | Preserved | Truncated after successful checkpoint |
| Main benefit | Avoid recomputation | Limit long lineage and improve fault recovery |
| Typical use | Repeated DataFrame operations | Long-running iterative computations |

Example of caching:

```python
df.cache()
```

Example of checkpointing:

```python
spark.sparkContext.setCheckpointDir(
    "/tmp/spark-checkpoints"
)

checkpointed_df = df.checkpoint()
```

**Important:** Checkpointing is useful when lineage becomes very long or when a reliable recovery boundary is needed. It is not simply another name for caching.

## 13. How to Check Storage Information?

### A. Check Storage Level

```python
print(df.storageLevel)
```

This shows the configured storage level.

### B. Use Spark UI

Open the Storage tab in Spark UI.

You can inspect:
- Cached DataFrames.
- Storage level.
- Memory used.
- Disk used.
- Number of cached partitions.
- Total partitions.

This helps verify whether the DataFrame was actually cached and how much storage it uses.

## 14. Common Mistakes

1. Assuming `cache()` executes the DataFrame immediately.
2. Assuming caching guarantees all data remains in memory.
3. Caching a DataFrame that is used only once.
4. Forgetting to call `unpersist()` when the cached data is no longer needed.
5. Confusing cache with checkpoint.
6. Assuming cache always improves performance.
7. Ignoring memory pressure and eviction.
8. Assuming a DataFrame action always materializes every cached partition.
9. Using a storage level without understanding its memory and disk implications.

## 15. Interview Questions

**Q1. What is caching in Spark?**

Caching stores computed DataFrame partitions so they can be reused without repeating the same transformations whenever possible.

**Q2. What is the difference between cache and persist?**

`cache()` uses the default storage level, while `persist()` allows an explicit storage level.

**Q3. Is caching lazy or eager?**

Caching is lazy. An action is generally required to materialize the cached data.

**Q4. How do you materialize a cached DataFrame?**

Use an action such as `count()` to trigger computation.

**Q5. What happens if cached data does not fit in memory?**

The behavior depends on the storage level. With `MEMORY_ONLY`, partitions that cannot remain cached may be recomputed later. With `MEMORY_AND_DISK`, partitions that do not fit in memory can be stored on disk.

**Q6. How do you remove cached data?**

Use `unpersist()`.

**Q7. When should you cache a DataFrame?**

When an expensive DataFrame is reused multiple times and storing it is more efficient than recomputing it.

**Q8. What is the difference between cache and checkpoint?**

Cache is mainly for reusing computed data while retaining lineage. Checkpoint saves data to checkpoint storage and truncates lineage after successful completion.

**Q9. Can cached data be evicted?**

Yes. Spark can evict cached partitions under memory pressure. Missing partitions can be recomputed from lineage.

**Q10. How do you verify whether a DataFrame is cached?**

Check its storage level and inspect the Storage tab in Spark UI.

## 16. Practice Questions

1. You perform five different aggregations on the same filtered DataFrame. Would you cache it? Why?
2. You cache a DataFrame but do not run an action. Has the data been materialized?
3. Your cached DataFrame does not fit in memory. What happens with `MEMORY_ONLY` and `MEMORY_AND_DISK`?
4. Your Spark application has memory pressure after caching many DataFrames. What would you do?
5. How is checkpointing different from caching?
6. Why should you unpersist a DataFrame?
7. How would you verify cache usage in Spark UI?
8. Can Spark recompute an evicted cached partition?
9. Why is caching a DataFrame used only once often unnecessary?

## 17. Quick Revision

- **Cache:** Stores computed data for reuse using the default storage level.
- **Persist:** Stores computed data using a selected storage level.
- **Lazy Evaluation:** Cache is generally materialized when an action runs.
- **MEMORY_ONLY:** Keeps cached partitions in memory when possible.
- **MEMORY_AND_DISK:** Uses memory and stores partitions that do not fit on disk.
- **DISK_ONLY:** Stores cached partitions on disk.
- **unpersist():** Removes persisted data.
- **Checkpoint:** Saves data and truncates lineage after successful completion.
- **Spark UI Storage Tab:** Shows cached data and storage usage.
- **Main benefit:** Reduces repeated computation.
- **Main risk:** Unnecessary memory and storage consumption.

## 18. Self-Check

- [ ] Explain caching and persisting.
- [ ] Differentiate `cache()` and `persist()`.
- [ ] Explain why caching is lazy.
- [ ] Explain cache materialization.
- [ ] Understand common storage levels.
- [ ] Explain when to cache and when to avoid it.
- [ ] Explain how to remove cached data.
- [ ] Differentiate cache and checkpoint.
- [ ] Explain memory pressure and cache eviction.
- [ ] Verify cached data using Spark UI.

**Interview goal:** Be able to explain when caching improves Spark performance, how Spark stores cached partitions, and why caching everything can reduce performance.