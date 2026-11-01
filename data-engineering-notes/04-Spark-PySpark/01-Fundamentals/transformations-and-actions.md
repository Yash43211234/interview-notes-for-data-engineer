# Spark Transformations and Actions — Interview Notes

## 1. Introduction

In Apache Spark, operations are divided into two major categories:

1. **Transformations:** Create a new DataFrame or RDD from an existing one.
2. **Actions:** Trigger execution and produce a result or write data to an external destination.

The most important concept is **lazy evaluation**.

Spark generally does not execute transformations immediately. It builds an execution plan and waits until an action requires the result.

**Simple way to remember:**

- Transformation → Defines what to do.
- Action → Triggers execution.

---

## 2. What Are Transformations?

A transformation is an operation that creates a new DataFrame or RDD from an existing dataset.

Transformations are generally lazy.

This means Spark records the operations and builds a logical execution plan instead of immediately processing the complete dataset.

### Example

```python
filtered_df = orders_df.filter(
    orders_df.amount > 1000
)
```

Here:
- `orders_df` is the original DataFrame.
- `filter()` selects records.
- `filtered_df` represents the transformed DataFrame.

The transformation does not normally trigger the complete computation immediately.

### Common Transformations

| Transformation | Purpose |
|---|---|
| `select()` | Select specific columns |
| `filter()` | Filter records |
| `where()` | Filter records |
| `withColumn()` | Add or modify a column |
| `drop()` | Remove columns |
| `distinct()` | Remove duplicate rows |
| `dropDuplicates()` | Remove duplicates based on selected columns |
| `groupBy()` | Group records |
| `orderBy()` | Sort records |
| `join()` | Join DataFrames |
| `union()` | Combine DataFrames |
| `repartition()` | Redistribute data into partitions |
| `coalesce()` | Reduce the number of partitions |

### Example: Multiple Transformations

```python
result_df = (
    orders_df
    .filter("amount > 1000")
    .select("customer_id", "amount")
    .withColumnRenamed("amount", "order_amount")
)
```

Spark builds a plan representing these operations.

Execution is triggered when an action is called.

---

## 3. What Are Actions?

An action is an operation that triggers execution of the computation needed to produce a result.

Actions can:
- Return results to the driver.
- Display data.
- Calculate a result.
- Write data to external storage.

### Common DataFrame Actions

| Action | Purpose |
|---|---|
| `show()` | Displays records |
| `count()` | Returns the number of rows |
| `collect()` | Returns all rows to the driver |
| `first()` | Returns the first row |
| `head()` | Returns the first row or requested number of rows |
| `take(n)` | Returns up to `n` rows |
| `write.save()` | Writes data to storage |
| `write.parquet()` | Writes data in Parquet format |
| `write.csv()` | Writes data in CSV format |

### Example

```python
orders_df.show()
```

`show()` triggers execution to produce the requested output.

Another example:

```python
total_orders = orders_df.count()

print(total_orders)
```

`count()` triggers the computation and returns the row count.

---

## 4. Difference Between Transformations and Actions

This is a very common Spark interview question.

| Feature | Transformation | Action |
|---|---|---|
| Purpose | Creates a new DataFrame or RDD | Produces a result or writes data |
| Execution | Generally lazy | Triggers execution |
| Returns | New DataFrame or RDD | Result, output, or completion information |
| Execution plan | Adds operations to the plan | Triggers the required computation |
| Examples | `filter()`, `select()`, `join()` | `count()`, `show()`, `collect()` |

### Simple Example

```python
df1 = orders_df.filter("amount > 1000")

df2 = df1.select("customer_id", "amount")

df2.show()
```

Execution:

```text
orders_df
    |
    v
filter(amount > 1000)
    |
    v
select(customer_id, amount)
    |
    v
show()
    |
    v
Spark executes the required computation
```

The first two operations are transformations.

The final operation is an action.

---

## 5. What Is Lazy Evaluation?

Lazy evaluation means Spark delays executing transformations until an action requires their results.

### Example

```python
df1 = orders_df.filter("amount > 1000")

df2 = df1.select("customer_id", "amount")

df3 = df2.withColumnRenamed(
    "amount",
    "order_amount"
)
```

At this point, Spark has defined the operations but has not necessarily processed all the underlying data.

Now execute:

```python
df3.show()
```

The action triggers the required computation.

### Why Does Spark Use Lazy Evaluation?

1. **Optimization:** Spark can optimize multiple operations together.
2. **Reduced unnecessary work:** Operations that are never needed may not be executed.
3. **Better execution planning:** Spark can choose an efficient physical execution strategy.
4. **Pipeline optimization:** Compatible operations can sometimes be combined into fewer execution steps.

**Interview answer:**

Spark uses lazy evaluation to delay execution until an action is called. This allows Spark to optimize the complete query plan instead of executing every transformation independently.

---

## 6. Types of Transformations

There are two important transformation categories:

- Narrow transformations.
- Wide transformations.

### A. Narrow Transformations

A narrow transformation occurs when each output partition depends on data from a limited number of input partitions, typically one.

Examples:

- `filter()`
- `select()`
- `map()`
- `flatMap()`

Example:

```python
filtered_df = orders_df.filter(
    orders_df.amount > 1000
)
```

Each output partition can be calculated using its corresponding input partition.

There is generally no shuffle required.

Conceptually:

```text
Input Partitions       Output Partitions

Partition 1  --------> Partition 1
Partition 2  --------> Partition 2
Partition 3  --------> Partition 3
```

### B. Wide Transformations

A wide transformation occurs when an output partition depends on data from multiple input partitions.

These operations generally require a shuffle.

Examples:

- `groupBy()`
- `distinct()`
- `repartition()`
- Many joins.
- Some window operations.

Example:

```python
result_df = orders_df.groupBy(
    "customer_id"
).sum("amount")
```

Spark may need to redistribute records so that records belonging to the same customer can be aggregated together.

Conceptually:

```text
Input Partitions
       |
       v
   Shuffle
       |
       v
Redistribute Data
       |
       v
Output Partitions
```

### Narrow vs Wide

| Feature | Narrow | Wide |
|---|---|---|
| Data dependency | Limited input partitions | Multiple input partitions |
| Shuffle | Generally not required | Generally required |
| Performance | Usually less expensive | Can be more expensive |
| Examples | `filter()`, `select()` | `groupBy()`, `distinct()` |

**Important:** Whether an operation causes a shuffle can depend on the existing partitioning and execution plan. For example, some joins can avoid a shuffle when the data is already appropriately partitioned or when Spark uses a broadcast join.

---

## 7. Common DataFrame Transformations

### A. `select()`

Select specific columns.

```python
df.select(
    "customer_id",
    "amount"
)
```

### B. `filter()`

Filter records based on a condition.

```python
df.filter(
    df.amount > 1000
)
```

### C. `withColumn()`

Add a new column or replace an existing column.

```python
from pyspark.sql.functions import col

df.withColumn(
    "tax",
    col("amount") * 0.18
)
```

### D. `drop()`

Remove a column.

```python
df.drop("tax")
```

### E. `dropDuplicates()`

Remove duplicate records.

```python
df.dropDuplicates(["customer_id"])
```

This keeps one row for each distinct `customer_id`, but does not provide a deterministic rule for which row is retained.

If you need to retain the latest record, use a window function with `row_number()` and an explicit ordering rule.

### F. `groupBy()`

Group records for aggregation.

```python
from pyspark.sql.functions import sum

df.groupBy("customer_id").agg(
    sum("amount").alias("total_amount")
)
```

### G. `join()`

Combine two DataFrames.

```python
result_df = customers_df.join(
    orders_df,
    on="customer_id",
    how="inner"
)
```

### H. `orderBy()`

Sort records.

```python
df.orderBy(
    col("amount").desc()
)
```

---

## 8. Common DataFrame Actions

### A. `show()`

Displays records.

```python
df.show(10)
```

### B. `count()`

Returns the number of rows.

```python
total_rows = df.count()
```

### C. `collect()`

Returns all DataFrame rows to the driver as a list of Row objects.

```python
rows = df.collect()
```

**Warning:** Avoid using `collect()` on large datasets because all returned records must fit in driver memory.

### D. `take()`

Returns up to the specified number of rows.

```python
rows = df.take(5)
```

### E. `first()`

Returns the first row.

```python
row = df.first()
```

Without an explicit sort, the first row is not guaranteed to be the earliest or latest business record.

### F. `write.parquet()`

Writes a DataFrame to Parquet storage.

```python
df.write.mode("overwrite").parquet(
    "/output/orders"
)
```

### G. `write.csv()`

Writes a DataFrame to CSV.

```python
df.write.mode("overwrite").option(
    "header", True
).csv("/output/orders_csv")
```

**Important:** A DataFrame write operation is an action because it triggers the computation needed to produce and save the output.

---

## 9. Do Multiple Actions Create Multiple Jobs?

Yes, separate actions generally trigger separate executions.

Example:

```python
df = orders_df.filter("amount > 1000")

df.count()

df.show()
```

Here:
- `filter()` defines a transformation.
- `count()` triggers an execution.
- `show()` triggers another execution.

Spark may execute the required computation again for the second action.

### How Can We Avoid Repeated Computation?

Use caching or persistence when the same expensive intermediate DataFrame will be reused.

```python
filtered_df = orders_df.filter(
    "amount > 1000"
).cache()

filtered_df.count()

filtered_df.show()
```

The first action computes the DataFrame and populates the cache. The second action can reuse cached partitions when available.

**Important:** Caching is lazy. Calling `cache()` marks the DataFrame for caching; it does not immediately compute and store every partition.

Caching also consumes resources, so use it when repeated reuse justifies the memory or storage cost.

---

## 10. Transformations and Actions in RDDs

Spark also provides transformations and actions for RDDs.

### RDD Transformations

| Operation | Purpose |
|---|---|
| `map()` | Applies a function to each element |
| `filter()` | Filters elements |
| `flatMap()` | Produces zero or more output elements per input |
| `distinct()` | Removes duplicate elements |
| `union()` | Combines two RDDs |
| `reduceByKey()` | Aggregates values by key |

Example:

```python
rdd = sc.parallelize([1, 2, 3, 4, 5])

squared_rdd = rdd.map(
    lambda x: x * x
)
```

### RDD Actions

| Operation | Purpose |
|---|---|
| `collect()` | Returns all elements |
| `count()` | Counts elements |
| `first()` | Returns the first element |
| `take(n)` | Returns up to n elements |
| `reduce()` | Aggregates elements |
| `saveAsTextFile()` | Saves elements to text files |

Example:

```python
result = squared_rdd.collect()

print(result)
```

Output:

```text
[1, 4, 9, 16, 25]
```

---

## 11. Important Interview Questions

**Q1. What is the difference between transformations and actions?**

Transformations create new DataFrames or RDDs and are generally lazy. Actions trigger execution and produce results or write data.

**Q2. Why are transformations lazy in Spark?**

Lazy evaluation allows Spark to build and optimize the execution plan before processing the data.

**Q3. What happens when you call `filter()`?**

Spark creates a transformed DataFrame and adds the filter operation to the execution plan. The actual computation is generally triggered by an action.

**Q4. Is `show()` a transformation or an action?**

It is an action because it triggers execution to display the requested records.

**Q5. Is `groupBy()` a narrow or wide transformation?**

It is generally a wide transformation because grouping records may require a shuffle.

**Q6. Is `filter()` a narrow or wide transformation?**

It is generally a narrow transformation because each output partition depends on its corresponding input partition.

**Q7. Does `cache()` trigger execution?**

No. It marks the DataFrame for caching. Execution and cache population happen when an action requires the data.

**Q8. What happens if you call two actions on the same DataFrame?**

Each action can trigger a separate execution. Spark may recompute the transformations unless the relevant data is cached or otherwise reused.

**Q9. Is `repartition()` a transformation?**

Yes. It is a transformation that changes the DataFrame's partitioning and generally causes a shuffle.

**Q10. Is `coalesce()` a transformation?**

Yes. It returns a DataFrame with a reduced number of partitions. It can often reduce partitions without a full shuffle, depending on how it is used.

**Q11. Is writing a DataFrame to storage an action?**

Yes. A write operation triggers the required computation and writes the result to the destination.

**Q12. Does every transformation create a new job?**

No. Transformations generally build the execution plan. Actions trigger execution.

---

## 12. Common Mistakes

1. **Thinking transformations execute immediately:** Most DataFrame transformations are lazy.
2. **Treating `show()` as a transformation:** It is an action.
3. **Assuming every transformation causes a shuffle:** Narrow transformations generally do not.
4. **Using `collect()` on large data:** It can cause driver out-of-memory errors.
5. **Thinking `cache()` immediately stores data:** Cache population requires an action.
6. **Assuming `groupBy()` is free:** It generally requires data redistribution and can be expensive.
7. **Assuming two actions share computed results automatically:** Without caching or reuse, Spark may recompute the data.
8. **Confusing `coalesce()` and `repartition()`:** They have different partitioning and shuffle behavior.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Transformation | Creates a new DataFrame or RDD |
| Action | Triggers execution |
| Lazy evaluation | Delays transformations until required |
| Narrow transformation | Limited input partition dependency |
| Wide transformation | Multiple input partition dependency |
| Shuffle | Redistributes data across partitions |
| `filter()` | Transformation; generally narrow |
| `select()` | Transformation; generally narrow |
| `groupBy()` | Transformation; generally wide |
| `repartition()` | Transformation; generally wide |
| `show()` | Action |
| `count()` | Action |
| `collect()` | Action |
| `write.parquet()` | Action |
| `cache()` | Marks data for caching; lazy |

### Complete flow

```text
Create DataFrame
       |
       v
Apply Transformations
       |
       v
Build Execution Plan
       |
       v
Trigger Action
       |
       v
Spark Optimizes Plan
       |
       v
Create Jobs, Stages and Tasks
       |
       v
Execute on Executors
       |
       v
Return or Write Results
```

---

## 14. Self-Check

- [ ] I can explain transformations and actions.
- [ ] I understand lazy evaluation.
- [ ] I can identify common DataFrame transformations.
- [ ] I can identify common actions.
- [ ] I understand narrow and wide transformations.
- [ ] I can explain why shuffle happens.
- [ ] I understand why `collect()` can be dangerous.
- [ ] I know how caching helps when multiple actions reuse data.
- [ ] I can explain these concepts with a PySpark example.

**Interview takeaway:** Transformations define the computation, and actions trigger it. Lazy evaluation allows Spark to optimize the plan, while narrow and wide dependencies determine how data is processed and whether shuffle is required.