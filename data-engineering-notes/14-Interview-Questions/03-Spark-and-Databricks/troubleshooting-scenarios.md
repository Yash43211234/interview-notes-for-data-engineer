# Spark and Databricks Troubleshooting Scenarios – Interview Questions and Answers

## 1. How to Approach a Troubleshooting Question

In a Data Engineer interview, troubleshooting questions test whether you can identify the root cause of a problem rather than simply suggest configuration changes.

Use this approach:

1. **Understand the symptom:** What failed, became slow, or produced incorrect data?
2. **Identify the affected component:** Driver, executor, Spark job, storage, network, or source system?
3. **Collect evidence:** Spark UI, logs, execution plans, metrics, and data samples.
4. **Find the root cause:** Avoid assuming the first visible error is the actual cause.
5. **Apply a targeted fix:** Change only what addresses the identified problem.
6. **Validate the result:** Check correctness, performance, and resource usage.
7. **Prevent recurrence:** Add monitoring, validation, alerts, or safer retry logic.

**Interview tip:** Explain the investigation before giving the solution. Saying "I will increase executor memory" without identifying the memory bottleneck is not a strong troubleshooting answer.

---

## 2. Spark Job Performance Scenarios

### Scenario 1: A Spark Job That Previously Took 20 Minutes Now Takes 2 Hours. What Would You Do?

**Possible causes:**
- Input data volume increased.
- Data skew developed.
- More small files appeared.
- A join strategy changed.
- Shuffle volume increased.
- Cluster resources decreased.
- Source storage became slower.
- A code or schema change affected execution.

**Investigation:**

1. Compare current and previous input sizes.
2. Check Spark UI job and stage duration.
3. Compare task durations within slow stages.
4. Inspect shuffle read/write and spill metrics.
5. Review executor failures and garbage collection.
6. Compare physical execution plans.
7. Check recent code, configuration, and data changes.

**Possible solutions:**
- Filter unnecessary data early.
- Select only required columns.
- Improve join strategy.
- Handle skewed keys.
- Compact small files.
- Tune partitions based on measurements.
- Restore appropriate compute resources if they were reduced.

**Interview answer:**

"I would compare the current run with a previous successful run. I would inspect the Spark UI to identify the slow stage, then check input size, shuffle, skew, spill, and executor metrics. After identifying the bottleneck, I would apply a targeted optimization and compare the results."

### Scenario 2: One Spark Stage Takes Much Longer Than Other Stages. Why?

**Possible causes:**
- Large shuffle.
- Data skew.
- Expensive aggregation.
- Sorting.
- Slow external storage.
- Executor resource pressure.

**Investigation:**
- Open the slow stage in Spark UI.
- Compare task duration.
- Check task input size.
- Check shuffle read/write.
- Check spill and executor metrics.

**Solution:**
Optimize the specific operation. For skew, consider AQE, salting, or separate processing for heavy keys. For large shuffle, reduce unnecessary data movement or improve the join and aggregation strategy.

### Scenario 3: Most Tasks Finish Quickly, but a Few Tasks Run for a Very Long Time. What Is the Problem?

<badge color="warning">Likely issue: Data skew</badge>

**Explanation:**

Data may be unevenly distributed across partitions. A few tasks receive much more data than others.

**Solution:**
- Inspect partition-level input sizes.
- Identify heavily repeated keys.
- Use AQE skew handling where supported.
- Apply key salting when appropriate.
- Process unusually heavy keys separately.

Do not assume every slow task is caused by skew. Slow machines, external I/O, and garbage collection can also cause uneven task duration.

### Scenario 4: A Spark Job Has High Shuffle Read and Shuffle Write. How Would You Optimize It?

**Possible causes:**
- Large joins.
- `groupBy()`.
- `distinct()`.
- `orderBy()`.
- Unnecessary repartitioning.
- Wide transformations.

**Solutions:**
- Filter rows before shuffle.
- Select fewer columns.
- Avoid unnecessary wide transformations.
- Broadcast a genuinely small relation when appropriate.
- Aggregate at the correct grain.
- Review partitioning.
- Inspect the physical plan.

Remember: shuffle is sometimes required. The goal is to reduce unnecessary shuffle, not eliminate every shuffle.

### Scenario 5: Spark Job Has High Disk Spill. What Would You Check?

**Possible causes:**
- Large partitions.
- Data skew.
- Memory-intensive sorting or aggregation.
- Large joins.
- Executor memory pressure.

**Investigation:**
- Check memory and disk spill in Spark UI.
- Compare task input sizes.
- Check executor memory and garbage collection.
- Inspect join and aggregation plans.

**Solutions:**
- Reduce data volume.
- Address skew.
- Improve partition distribution.
- Use an appropriate join strategy.
- Avoid unnecessary caching.
- Tune memory only when evidence supports it.

---

## 3. Memory and Executor Failure Scenarios

### Scenario 6: Spark Application Fails With Executor Out-of-Memory Error. What Would You Do?

**Possible causes:**
- A large partition.
- Data skew.
- Oversized broadcast data.
- Memory-intensive joins or aggregations.
- Excessive caching.
- Insufficient executor memory.

**Investigation:**
1. Identify the failed executor and task.
2. Inspect executor logs.
3. Check memory usage and spill.
4. Examine partition sizes.
5. Inspect joins and aggregations.
6. Check whether a broadcast relation is too large.

**Solutions:**
- Reduce data processed.
- Handle skew.
- Increase partition count if partitions are too large and this improves distribution.
- Avoid unnecessary caching.
- Change join strategy.
- Increase executor memory only if the workload genuinely requires it and the cluster has capacity.

### Scenario 7: Spark Driver Runs Out of Memory. What Could Be the Reason?

**Common causes:**
- Calling `collect()` on a large DataFrame.
- Collecting large results through `toPandas()`.
- Building large local Python objects.
- Excessive query-plan or metadata operations.
- Too many small files.
- Driver-side result handling.

**Example of risky code:**

```python
# Risky for large datasets
all_rows = df.collect()
```

**Better approach:**

```python
# Keep processing distributed
result = (
    df.groupBy("department")
      .count()
)

result.show()
```

**Solution:** Keep large-scale processing distributed. Use `collect()` and `toPandas()` only when the resulting data is small enough for driver memory.

### Scenario 8: An Executor Keeps Restarting. How Would You Investigate?

**Possible causes:**
- Out-of-memory errors.
- Container memory limit exceeded.
- Node or infrastructure failure.
- Network problems.
- Executor process crash.
- Resource contention.

**Investigation:**
- Read executor and cluster-manager logs.
- Check exit codes and failure reasons.
- Inspect memory and CPU usage.
- Check whether failures occur on one node or across the cluster.
- Look for repeated failures on the same task or data partition.

**Solution:** Fix the actual cause, such as memory pressure, skew, node instability, or resource allocation. Repeatedly retrying a failing executor is not a complete solution.

### Scenario 9: Spark Application Fails With `java.lang.OutOfMemoryError: Java heap space`. What Does It Mean?

It means a JVM process could not allocate enough heap memory for an operation.

The process may be the driver or an executor.

**What to check:**
- Which process failed.
- Executor or driver logs.
- Heap usage.
- Large partitions.
- Cache usage.
- Join and aggregation behavior.
- Whether the application collects excessive data.

**Solution:** Reduce memory demand or adjust resource allocation based on evidence. Do not increase driver memory when the error is actually occurring in an executor.

---

## 4. Join and Data Skew Scenarios

### Scenario 10: Joining Two Large DataFrames Takes Too Long. What Would You Do?

**Possible causes:**
- Both datasets are large.
- Join causes a large shuffle.
- Join keys are skewed.
- Input data contains unnecessary columns.
- Data types or join conditions are inconsistent.
- Duplicate keys increase output rows.

**Investigation:**
1. Inspect the physical join strategy.
2. Check both input sizes.
3. Check join-key distribution.
4. Check shuffle metrics.
5. Check duplicate keys and expected join grain.

**Solutions:**
- Filter and select columns before joining.
- Use a suitable partitioning strategy.
- Address skew.
- Consider broadcasting only if one side is genuinely small.
- Aggregate data to the correct grain before joining when appropriate.

### Scenario 11: A Small Table Is Being Joined With a Large Table. Which Join Strategy Would You Consider?

A broadcast join may be suitable if the smaller table can safely fit in executor memory and the join supports the strategy.

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    on="customer_id",
    how="inner"
)
```

**Why?**

The small relation is sent to executors, which can avoid shuffling the large side for supported join strategies.

**Caution:** A table with few rows is not necessarily small in bytes. Check its actual size and memory requirements.

### Scenario 12: A Join Produces More Rows Than Expected. What Could Be Wrong?

**Likely cause:** Duplicate join keys or an incorrect join condition.

Example:

- Customer table has one row per customer.
- Transaction table has many rows per customer.
- Joining them produces multiple rows per customer, which may be expected.

However, if both tables contain duplicate customer IDs, the join can multiply records.

**Investigation:**
- Confirm each table's grain.
- Count duplicate keys.
- Validate join conditions.
- Compare expected and actual cardinality.

**Solution:**
- Deduplicate where business rules require it.
- Aggregate to compatible grain.
- Correct the join condition.
- Validate row counts and business totals.

### Scenario 13: A Broadcast Join Causes Executor Out-of-Memory Errors. What Would You Do?

**Possible cause:** The broadcast relation is too large for available executor memory.

**Solution:**
- Remove the broadcast hint.
- Check actual relation size and statistics.
- Allow Spark to choose another join strategy.
- Consider reducing the broadcast dataset first.
- Review executor memory and concurrent workload.

Do not force broadcast merely to avoid shuffle.

### Scenario 14: One Customer ID Has Millions of Records. How Would You Handle It?

This is a possible skewed-key scenario.

**Options:**
- Use AQE skew-join handling where applicable.
- Salt the heavy key.
- Process the heavy key separately.
- Reconsider whether the join or aggregation can be performed at a different grain.

**Important:** Key salting requires careful handling of the other side of the join. Incorrect salting can cause missing or duplicated results.

---

## 5. Partition and File Scenarios

### Scenario 15: A Spark Job Creates Thousands of Small Files. What Is the Problem?

**Problem:** Small files increase metadata and file listing overhead and can create many tiny Spark tasks.

**Possible causes:**
- Too many output partitions.
- Frequent small incremental writes.
- Over-partitioning by high-cardinality columns.
- Small input batches.

**Solutions:**
- Use suitable output partition counts.
- Avoid unnecessary high-cardinality storage partitioning.
- Compact files.
- Use optimized writes or auto compaction where supported.
- Review the write pattern.

For Delta tables on supported Databricks environments:

```sql
OPTIMIZE customers;
```

### Scenario 16: A Spark Job Creates Only One or Two Output Files. Is That Always Good?

No.

Too few output files can reduce parallelism during later reads and create large individual files.

The correct file layout depends on:
- Dataset size.
- Query patterns.
- Available parallelism.
- Storage format.
- Downstream workloads.

Avoid assuming that one file is always optimal.

### Scenario 17: Partition Pruning Is Not Working. What Would You Check?

**Possible causes:**
- The table is not partitioned by the filtered column.
- The filter expression does not match the partition column appropriately.
- A function or cast prevents effective pruning.
- The query reads a different table or location.
- The physical plan does not show expected pruning.

**Investigation:**

```python
df.filter(
    col("year") == 2026
).explain("formatted")
```

Check the scan details and partition filters.

**Solution:** Align the query with the actual partition column and data types. Avoid unnecessary transformations on partition columns.

### Scenario 18: A Table Has Too Many Storage Partitions. What Would You Do?

**Problem:** Excessive partitioning can create many small directories and files.

**Solution:**
- Review partition cardinality.
- Analyze common filter patterns.
- Remove unnecessary high-cardinality partition columns.
- Consider date-based partitions for suitable workloads.
- Evaluate clustering or data-skipping features where supported.

---

## 6. DataFrame and PySpark Coding Scenarios

### Scenario 19: A DataFrame Transformation Is Taking Too Long. How Would You Investigate?

**Investigation:**
- Check whether a Python UDF is being used.
- Inspect the physical plan.
- Look for shuffles.
- Check whether unnecessary columns are carried forward.
- Check whether the same transformation is repeated.
- Check source filter and projection pushdown.

**Solutions:**
- Prefer built-in Spark SQL functions.
- Reduce data early.
- Avoid unnecessary transformations.
- Cache expensive results only when reused.
- Optimize joins and aggregations.

### Scenario 20: A Python UDF Is Slowing Down a Spark Job. What Would You Do?

Python UDFs can introduce serialization and Python execution overhead.

**Example of a built-in alternative:**

```python
from pyspark.sql.functions import upper

df2 = df.withColumn(
    "customer_name",
    upper("customer_name")
)
```

Prefer built-in functions when possible because Spark can understand and optimize their expressions.

If custom Python logic is necessary, consider whether a Pandas UDF is suitable and test its memory and performance characteristics.

### Scenario 21: A DataFrame Transformation Returns Incorrect Results. What Would You Check?

**Investigation:**
- Input schema.
- Null values.
- Data types.
- Join keys.
- Duplicate records.
- Filter conditions.
- Aggregation grain.
- Time-zone and date handling.
- Unexpected implicit casts.

**Solution:** Build a small test dataset with known expected output. Validate each transformation independently before running the complete pipeline.

### Scenario 22: `union()` Produces Incorrect Column Values. Why?

`union()` combines columns by position, not by column name.

Example:

```python
# Column order may differ
result = df1.union(df2)
```

If the schemas have different column order, values may be assigned to the wrong logical columns.

**Solution:**

```python
result = df1.unionByName(df2)
```

For supported use cases, missing columns can be handled with:

```python
result = df1.unionByName(
    df2,
    allowMissingColumns=True
)
```

Validate data types and missing-column behavior.

### Scenario 23: `dropDuplicates()` Keeps an Unexpected Record. Why?

`dropDuplicates()` does not guarantee which record is retained when multiple records share the specified key.

**Solution:** Use a window function with deterministic ordering when you need to retain a particular record.

```python
from pyspark.sql import Window
from pyspark.sql.functions import col, row_number

window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy(
        col("updated_at").desc(),
        col("source_sequence").desc()
    )
)

latest_df = (
    df.withColumn(
        "row_num",
        row_number().over(window_spec)
    )
    .filter(col("row_num") == 1)
    .drop("row_num")
)
```

The tie-breaker must be meaningful and deterministic.

---

## 7. Delta Lake Troubleshooting Scenarios

### Scenario 24: A Delta Write Fails Because of a Schema Mismatch. What Would You Do?

**Possible causes:**
- Incoming column type differs from the target.
- New columns were introduced.
- Column names do not match.
- Unsupported schema changes were attempted.

**Investigation:**
- Compare source and target schemas.
- Check the error message.
- Inspect nullability and data types.
- Identify whether the change is expected.

**Solutions:**
- Cast or clean incoming values.
- Correct the source schema.
- Use controlled schema evolution when appropriate.
- Avoid automatically accepting unexpected schema changes.

### Scenario 25: A MERGE Operation Fails With Multiple Source Matches. What Does It Mean?

Multiple source records may match the same target row.

This can make the intended update ambiguous.

**Solution:**
1. Check duplicate source business keys.
2. Define the correct source record selection rule.
3. Deduplicate the source.
4. Retry MERGE with deterministic input.

Do not remove duplicates arbitrarily without understanding business rules.

### Scenario 26: A Delta Table Has Poor Query Performance. What Would You Check?

**Possible causes:**
- Too many small files.
- Poor partitioning.
- Ineffective data skipping.
- Large shuffle.
- Inefficient join.
- Missing or unsuitable statistics.
- Excessive data scanned.

**Solutions:**
- Inspect the query plan.
- Check scan metrics.
- Compact small files with OPTIMIZE when useful.
- Review partitioning and clustering.
- Consider ZORDER or liquid clustering where supported.
- Reduce unnecessary columns and rows.

### Scenario 27: Time Travel Query Fails for an Older Version. Why?

**Possible causes:**
- Required transaction-log history was cleaned up.
- Required data files were removed by VACUUM.
- Retention settings do not preserve the requested version.

**Solution:**
- Check `DESCRIBE HISTORY`.
- Review log and data retention settings.
- Check whether the required files still exist.
- Restore from an independent backup if the required history is no longer available.

Time Travel is not a guaranteed permanent backup.

### Scenario 28: A VACUUM Operation Removes Files Needed by an Old Version. What Happened?

VACUUM removed unreferenced data files that were eligible under the configured retention rules.

If an older table version depended on those files, that version may no longer be readable.

**Prevention:**
- Set appropriate retention.
- Understand recovery and audit requirements.
- Avoid disabling safety checks without a clear reason.
- Use backups or replication for long-term recovery needs.

### Scenario 29: Two Jobs Update the Same Delta Table at the Same Time. What Could Happen?

Delta Lake uses optimistic concurrency control.

If the operations conflict, one transaction may fail during commit.

**Solution:**
- Identify the conflicting operations.
- Re-read the latest table state.
- Recompute if necessary.
- Retry safely.
- Consider coordinating writes if the workload frequently conflicts.

### Scenario 30: A Delta Table Contains Unexpected Duplicate Records After a Retry. Why?

Possible causes:
- The pipeline is not idempotent.
- The source batch was processed more than once.
- The write logic uses append without deduplication.
- The business key is not correctly defined.
- Retry logic advanced or reused source progress incorrectly.

**Solution:**
- Define the target grain and business key.
- Use deterministic deduplication.
- Use MERGE or another suitable idempotent write pattern.
- Track source batch IDs or CDC positions.
- Reconcile the target after retries.

---

## 8. Structured Streaming and Pipeline Recovery

### Scenario 31: A Structured Streaming Query Stops Unexpectedly. What Would You Check?

**Possible causes:**
- Source connectivity issue.
- Invalid data or schema issue.
- Executor failure.
- Sink failure.
- Checkpoint problem.
- Resource exhaustion.
- Unhandled application exception.

**Investigation:**
1. Check query status and exception details.
2. Inspect driver and executor logs.
3. Check source and sink availability.
4. Inspect checkpoint accessibility.
5. Check schema and malformed records.
6. Review resource metrics.

**Solution:** Fix the root cause and restart using the correct checkpoint when the query and checkpoint remain compatible.

### Scenario 32: A Streaming Query Restarts and Processes Data Again. Is That Always a Bug?

Not necessarily.

Structured Streaming uses checkpointed progress to recover from failures. Depending on source, sink, and write behavior, records or batches may be retried.

The important question is whether the final committed results remain correct.

**Solution:**
- Use supported fault-tolerant source and sink patterns.
- Make external writes idempotent.
- Preserve checkpoint state.
- Validate recovery behavior.
- Do not claim exactly-once behavior for arbitrary sinks.

### Scenario 33: A Streaming Query Stops Finding New Files. What Would You Check?

For file-based ingestion, check:
- Whether new files are arriving.
- Source directory and permissions.
- File naming and supported formats.
- Auto Loader configuration, if used.
- Checkpoint state.
- Schema evolution behavior.
- Notification or directory-listing setup.

For Databricks Auto Loader, remember that `cloudFiles.schemaLocation` and `checkpointLocation` serve different purposes.

### Scenario 34: A Streaming Query Has Increasing Processing Delay. What Could Be the Cause?

**Possible causes:**
- Incoming data rate exceeds processing capacity.
- Slow sink writes.
- Expensive transformations.
- Large stateful operations.
- Data skew.
- Insufficient compute resources.

**Investigation:**
- Compare input and processed rows per batch.
- Check batch duration.
- Check processing and input rates.
- Inspect state-store metrics.
- Check sink latency and executor utilization.

**Solution:** Optimize transformations, reduce unnecessary data, improve sink performance, handle skew, or increase resources when measurements support it.

### Scenario 35: Late-Arriving Events Are Missing From a Streaming Result. Why?

Possible causes:
- Watermark threshold is too small for actual lateness.
- Events arrive after state has been cleaned up.
- Incorrect event-time column.
- Source data is malformed.
- Stateful operation semantics drop data older than the allowed watermark.

**Solution:**
- Understand actual event lateness.
- Choose a suitable watermark delay.
- Validate event-time values.
- Consider how state retention affects results.
- Design a correction or reconciliation process if late data must be recovered.

A watermark is based on observed event-time progress, not simply the current wall-clock time.

---

## 9. Azure and Databricks Operational Scenarios

### Scenario 36: A Databricks Job Fails While Reading From ADLS Gen2. What Would You Check?

**Possible causes:**
- Incorrect storage path.
- Missing permissions.
- Authentication failure.
- Network restrictions.
- Storage account firewall rules.
- Expired or invalid credentials.

**Investigation:**
1. Verify the storage account and filesystem path.
2. Check whether the compute can reach the storage endpoint.
3. Verify the authentication method.
4. Check managed identity or service principal permissions.
5. Check Azure RBAC and ADLS ACLs.
6. Inspect the error message and logs.

**Important distinction:**

Authentication establishes identity. Authorization determines whether that identity can access the requested data.

A managed identity does not automatically have access to ADLS Gen2.

### Scenario 37: A Job Works in Development but Fails in Production. What Would You Investigate?

**Possible causes:**
- Different permissions.
- Different runtime versions.
- Different cluster configuration.
- Missing libraries.
- Different environment variables.
- Different storage paths.
- Different source data.
- Network or firewall restrictions.

**Solution:**
- Compare environments.
- Validate dependencies.
- Check identity and access.
- Compare runtime and configuration.
- Test with representative production data.
- Use controlled deployment and configuration management.

### Scenario 38: A Databricks Job Is Successful but the Output Table Is Empty. What Would You Check?

**Possible causes:**
- Input source is empty.
- Filters removed all rows.
- Incorrect date or timestamp range.
- Wrong storage location.
- Unexpected join behavior.
- Output mode or write logic.
- Incorrect partition path.

**Investigation:**
- Check source row count.
- Check row counts after each transformation.
- Validate filters and join keys.
- Inspect output path and table location.
- Check whether the write operation completed.

**Solution:** Add stage-level row-count checks and data-quality validations.

### Scenario 39: A Databricks Job Takes Much Longer After a Runtime Upgrade. What Would You Do?

**Investigation:**
- Compare runtime versions.
- Compare Spark configurations.
- Compare physical plans.
- Check library compatibility.
- Inspect Photon or AQE behavior where relevant.
- Compare input data and cluster configuration.
- Review release notes for relevant changes.

**Solution:** Reproduce the issue in a controlled environment, identify the changed behavior, and tune or correct the affected operation.

---

## 10. Data Quality and Pipeline Troubleshooting

### Scenario 40: Source and Target Row Counts Do Not Match. What Would You Check?

**Possible causes:**
- Duplicate source records.
- Filter conditions.
- Failed records.
- Incremental load logic.
- Join multiplication.
- Late-arriving data.
- Deletes or updates.
- Incorrect source checkpoint.

**Investigation:**
1. Compare source and target counts at the same grain.
2. Check duplicate business keys.
3. Check rejected and quarantined records.
4. Validate filters and joins.
5. Compare source and target key sets.
6. Check incremental load boundaries.

**Solution:** Reconcile using business keys and appropriate business rules, not only total row counts.

### Scenario 41: The Pipeline Runs Successfully but Data Is Incorrect. What Would You Do?

A successful job status does not guarantee correct business results.

**Investigation:**
- Validate source schema.
- Check transformation logic.
- Check null handling.
- Verify join cardinality.
- Verify aggregation grain.
- Check time zones and date boundaries.
- Compare sample records with expected output.

**Solution:** Add data-quality tests and reconciliation checks before publishing the output.

### Scenario 42: Incremental Loads Are Missing Updated Records. Why?

**Possible causes:**
- Timestamp watermark advanced too early.
- Late updates arrived behind the saved watermark.
- Timestamp precision is insufficient.
- Source records were updated without changing the tracked timestamp.
- Deletes are not captured.

**Solutions:**
- Use CDC where available.
- Use an overlap or lookback window.
- Use a reliable source sequence or change position.
- Advance progress only after the target write succeeds.
- Make reruns idempotent.
- Reconcile periodically.

### Scenario 43: A Pipeline Processes the Same Data Multiple Times. How Would You Fix It?

**Possible causes:**
- Retry behavior.
- Incorrect checkpoint handling.
- Append-only writes without deduplication.
- Repeated source file discovery.
- Incorrect batch identifiers.

**Solutions:**
- Track source batch or event IDs.
- Use idempotent target writes.
- Deduplicate using business keys.
- Use reliable checkpoints.
- Validate the source-to-target processing boundary.

---

## 11. Troubleshooting Commands and Code

### 11.1 Inspect a Physical Execution Plan

```python
df.explain("formatted")
```

Use it to inspect:
- Scan operations.
- Filters.
- Exchanges.
- Join strategies.
- Aggregations.
- Adaptive execution details where available.

### 11.2 Inspect Spark Configuration

```python
spark.conf.get(
    "spark.sql.shuffle.partitions"
)

spark.conf.get(
    "spark.sql.adaptive.enabled"
)
```

Some settings are not available through every configuration interface or may be controlled by the platform.

### 11.3 Check DataFrame Row Counts

```python
print("Source:", source_df.count())
print("Filtered:", filtered_df.count())
print("Target:", target_df.count())
```

For large datasets, repeated `count()` actions can be expensive. Use these checks selectively or use pipeline metrics already collected during processing.

### 11.4 Check Duplicate Business Keys

```python
from pyspark.sql.functions import count

duplicates = (
    df.groupBy("customer_id")
      .agg(count("*").alias("record_count"))
      .filter("record_count > 1")
)

duplicates.show()
```

### 11.5 Check Null Values

```python
from pyspark.sql.functions import col, sum

df.select(
    sum(
        col("customer_id").isNull().cast("int")
    ).alias("null_customer_ids")
).show()
```

### 11.6 Check Partition Count

```python
print(df.rdd.getNumPartitions())
```

This returns the number of Spark partitions represented by the DataFrame's underlying RDD. It does not directly report the number of storage directories or output files.

### 11.7 Inspect Streaming Query Status

```python
for query in spark.streams.active:
    print("Name:", query.name)
    print("Active:", query.isActive)
    print("Status:", query.status)
    print("Progress:", query.lastProgress)
```

---

## 12. Common Troubleshooting Mistakes

1. Increasing memory before identifying the cause.
2. Increasing partitions without checking data distribution.
3. Broadcasting a relation that is too large.
4. Treating every slow task as data skew.
5. Assuming a successful job means correct output.
6. Deleting streaming checkpoints to solve every restart problem.
7. Running VACUUM without checking retention requirements.
8. Using `collect()` to inspect large datasets.
9. Ignoring join cardinality and duplicate keys.
10. Retrying a non-idempotent pipeline without checking its previous output.
11. Checking only row counts instead of reconciling business keys and values.
12. Changing several configurations simultaneously, making it difficult to identify which change helped.

---

## 13. Interview Answer Framework

When an interviewer gives you a troubleshooting problem, organize your response like this:

**1. Symptom:** What is happening?

**2. Possible causes:** What could explain it?

**3. Evidence:** Which logs, metrics, plans, or data checks would confirm the cause?

**4. Root cause:** What did the evidence reveal?

**5. Fix:** What targeted change would you make?

**6. Validation:** How would you prove the issue is resolved?

**7. Prevention:** What monitoring or validation would prevent recurrence?

### Example: Spark Job Is Running Slowly

- **Symptom:** Job duration increased from 20 minutes to 2 hours.
- **Possible causes:** Skew, shuffle, small files, or resource pressure.
- **Evidence:** Spark UI, task duration, shuffle metrics, and physical plan.
- **Root cause:** One join key produces a heavily skewed partition.
- **Fix:** Apply an appropriate skew-handling strategy.
- **Validation:** Compare stage duration and task distribution.
- **Prevention:** Monitor key distribution and stage-level performance.

---

## 14. Quick Revision Table

| Problem | First Investigation | Possible Solution |
|---|---|---|
| Slow Spark job | Spark UI and physical plan | Optimize identified bottleneck |
| One slow stage | Stage and task metrics | Fix expensive operation |
| Few slow tasks | Partition distribution | Handle skew |
| High shuffle | Join and aggregation plan | Reduce unnecessary shuffle |
| High disk spill | Memory and task metrics | Reduce pressure and improve distribution |
| Executor OOM | Executor logs | Fix skew, memory pressure, or oversized operations |
| Driver OOM | Driver logs and code | Avoid large `collect()` |
| Slow large join | Join plan and key distribution | Suitable join strategy |
| Unexpected extra rows | Join cardinality | Correct grain and duplicates |
| Small files | File count and size | Compact files |
| Too few partitions | Parallelism and task sizes | Repartition appropriately |
| Schema mismatch | Source and target schemas | Correct schema or controlled evolution |
| MERGE failure | Duplicate source keys | Deterministic deduplication |
| Time Travel failure | Log and file retention | Check history and backups |
| Streaming restart issue | Checkpoint and sink behavior | Safe recovery and idempotency |
| Missing late events | Watermark and event time | Adjust late-data strategy |
| Empty output | Row counts and filters | Correct transformation logic |
| Missing incremental updates | Watermark and CDC logic | Lookback, CDC, reconciliation |
| ADLS access failure | Identity, RBAC, ACL, network | Correct access and connectivity |

---

## 15. Self-Assessment

Before a Data Engineer interview, make sure you can:

- [ ] Investigate a slow Spark job using Spark UI.
- [ ] Identify a slow stage and slow tasks.
- [ ] Explain data skew and shuffle problems.
- [ ] Troubleshoot executor and driver memory errors.
- [ ] Optimize large joins.
- [ ] Diagnose unexpected join output.
- [ ] Troubleshoot small files and partitioning.
- [ ] Investigate schema mismatch and MERGE failures.
- [ ] Explain VACUUM and Time Travel risks.
- [ ] Troubleshoot Structured Streaming failures.
- [ ] Explain checkpoint recovery and idempotency.
- [ ] Investigate missing late-arriving records.
- [ ] Troubleshoot ADLS Gen2 access failures.
- [ ] Diagnose source-to-target count mismatches.
- [ ] Validate data correctness, not just job success.
- [ ] Explain your troubleshooting process clearly.

**Preparation priority:** Practice explaining the investigation process for slow jobs, data skew, shuffle, OOM errors, large joins, MERGE failures, streaming recovery, and incremental-load issues. These scenarios connect Spark fundamentals with real Data Engineering work.