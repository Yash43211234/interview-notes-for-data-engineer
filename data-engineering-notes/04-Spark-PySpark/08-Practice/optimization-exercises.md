# PySpark Optimization Exercises

This document contains practical exercises to improve PySpark performance and prepare for Data Engineer interviews.

The focus is not just writing code that works, but understanding how to make it efficient for large datasets.

## 1. How to Approach Optimization Problems

Whenever you receive a performance-related problem, follow this process:

1. Understand the current implementation.
2. Identify expensive operations.
3. Check whether unnecessary data is being processed.
4. Identify shuffle operations.
5. Inspect joins and partitioning.
6. Check for data skew.
7. Review the execution plan and Spark UI.
8. Apply an optimization.
9. Compare performance before and after.

**Important:** Do not optimize blindly. First identify the actual bottleneck.

---

## 2. Exercise 1: Avoid Reading Unnecessary Columns

<badge color="success">Beginner</badge>

**Problem:**

You have an orders dataset containing 30 columns, but your transformation needs only:

- order_id
- customer_id
- amount

### Inefficient Approach

```python
df = spark.read.parquet("/data/orders")

result = df.select(
    "order_id",
    "customer_id",
    "amount"
)
```

### Optimized Approach

```python
result = (
    spark.read.parquet("/data/orders")
        .select(
            "order_id",
            "customer_id",
            "amount"
        )
)
```

### Why Is This Better?

Spark can apply projection pushdown for supported data sources and read only the required columns.

This reduces unnecessary data reading and processing.

**Concept:** Projection Pushdown.

### Interview Question

What is projection pushdown, and why does it improve performance?

---

## 3. Exercise 2: Apply Filters Early

<badge color="success">Beginner</badge>

**Problem:**

You need to calculate total revenue for successful orders only.

### Inefficient Approach

```python
result = (
    df.groupBy("customer_id")
      .sum("amount")
      .filter("customer_id IS NOT NULL")
)
```

This does not apply the business filter on order status before aggregation.

### Optimized Approach

```python
from pyspark.sql.functions import col, sum

result = (
    df.filter(
        col("status") == "SUCCESS"
    )
    .groupBy("customer_id")
    .agg(
        sum("amount").alias("total_revenue")
    )
)
```

### Why Is This Better?

Filtering early can reduce the number of records processed by subsequent transformations.

Spark may also optimize filter placement through predicate pushdown and other optimizer rules when supported.

**Concept:** Predicate Pushdown and Early Filtering.

### Interview Question

Why should we filter data as early as possible?

---

## 4. Exercise 3: Replace collect() with Distributed Processing

<badge color="success">Beginner</badge>

**Problem:**

You need to calculate the total amount of a large orders dataset.

### Inefficient Approach

```python
rows = df.collect()

total = sum(
    row["amount"]
    for row in rows
)
```

### Optimized Approach

```python
from pyspark.sql.functions import sum

result = df.agg(
    sum("amount").alias("total_amount")
)

result.show()
```

### Why Is This Better?

`collect()` transfers all records to the driver.

For a large dataset, this can cause driver memory problems.

The optimized approach performs aggregation in the distributed Spark execution environment and returns only the aggregate result.

**Concept:** Distributed Processing and Driver Memory Management.

### Interview Question

Why is `collect()` dangerous for large datasets?

---

## 5. Exercise 4: Optimize a Join Using Broadcast

<badge color="warning">Intermediate</badge>

**Problem:**

You have:
- Orders: 100 million records.
- Customers: 50,000 records.

You need to join both datasets using `customer_id`.

### Basic Approach

```python
result = orders.join(
    customers,
    on="customer_id",
    how="inner"
)
```

### Optimized Approach

```python
from pyspark.sql.functions import broadcast

result = orders.join(
    broadcast(customers),
    on="customer_id",
    how="inner"
)
```

### Why Is This Better?

A broadcast join distributes the smaller DataFrame to the worker nodes.

This can avoid shuffling the large DataFrame for the join.

### Important Considerations

- The smaller DataFrame should fit safely in executor memory.
- Check the estimated table size and query plan.
- A broadcast hint does not guarantee that the optimizer will use the hinted strategy in every situation.
- Broadcast joins are suitable for supported join types and conditions.

**Concept:** Broadcast Hash Join.

### Interview Question

When would you use a broadcast join instead of a sort-merge join?

---

## 6. Exercise 5: Avoid Unnecessary Shuffles

<badge color="warning">Intermediate</badge>

**Problem:**

You need to calculate the number of orders for each customer.

### Basic Approach

```python
result = df.groupBy("customer_id").count()
```

This is a valid aggregation, but it generally requires data redistribution because records for the same customer must be grouped together.

### Optimization

If the source is partitioned appropriately, or if the DataFrame is reused and partitioning is beneficial for subsequent operations, consider an appropriate partitioning strategy.

```python
partitioned_df = df.repartition(
    20,
    "customer_id"
)

result = partitioned_df.groupBy(
    "customer_id"
).count()
```

### Why Is This Not Always Faster?

`repartition()` itself introduces a shuffle.

If this is a one-time aggregation, explicitly repartitioning first may add unnecessary work.

The optimization is useful only when the resulting partitioning benefits the wider processing workflow.

**Concept:** Shuffle and Partitioning.

### Interview Question

Does repartitioning always improve performance?

---

## 7. Exercise 6: Handle Data Skew

<badge color="warning">Intermediate</badge>

**Problem:**

You are joining orders and customers.

One customer has 40% of all orders, while most other customers have very few records.

This creates an uneven distribution of data.

### Basic Approach

```python
result = orders.join(
    customers,
    "customer_id",
    "inner"
)
```

### Possible Optimization 1: Broadcast Join

If the customers DataFrame is small enough:

```python
result = orders.join(
    broadcast(customers),
    "customer_id",
    "inner"
)
```

### Possible Optimization 2: Salting

Salting distributes records belonging to a skewed key across multiple partitions.

Conceptual example:

```python
from pyspark.sql.functions import (
    col,
    when,
    floor,
    rand,
    explode,
    array,
    lit
)

salt_count = 5

salted_orders = orders.withColumn(
    "salt",
    when(
        col("customer_id") == 101,
        floor(rand() * salt_count).cast("int")
    ).otherwise(lit(0))
)

salted_customers = customers.withColumn(
    "salt",
    when(
        col("customer_id") == 101,
        explode(array(*[
            lit(i) for i in range(salt_count)
        ]))
    ).otherwise(lit(0))
)

result = salted_orders.join(
    salted_customers,
    ["customer_id", "salt"],
    "inner"
).drop("salt")
```

### Why Does Salting Help?

The heavily represented key is distributed across multiple salt values.

This can reduce the workload concentrated in one partition.

**Important:** This is a simplified example. Production salting must handle key types and join semantics carefully. The salt values must be assigned consistently, and the matching dimension-side key must be replicated across all salt values.

**Concept:** Data Skew and Salting.

### Interview Question

What is data skew, and how does salting help?

---

## 8. Exercise 7: Avoid Repeated Computations

<badge color="warning">Intermediate</badge>

**Problem:**

You use the same transformed DataFrame multiple times.

### Basic Approach

```python
filtered_df = df.filter(
    "amount > 100"
)

filtered_df.groupBy("status").count().show()

filtered_df.groupBy("customer_id").count().show()
```

Spark may recompute the shared transformation when separate actions execute.

### Optimized Approach

```python
filtered_df = (
    df.filter("amount > 100")
      .cache()
)

filtered_df.groupBy("status").count().show()

filtered_df.groupBy("customer_id").count().show()

filtered_df.unpersist()
```

### Why Is This Better?

Caching can avoid recomputing the same expensive DataFrame transformation when the result is reused.

### Important Considerations

- Cache only DataFrames that are reused.
- Caching consumes memory and possibly disk.
- `cache()` is lazy.
- An action is needed to materialize the cached data.
- Unpersist when the cached result is no longer needed.

**Concept:** Caching and Persisting.

### Interview Question

When should you cache a DataFrame?

---

## 9. Exercise 8: Optimize Multiple withColumn Operations

<badge color="warning">Intermediate</badge>

**Problem:**

You need to add several calculated columns.

### Basic Approach

```python
from pyspark.sql.functions import col

df1 = df.withColumn(
    "tax",
    col("amount") * 0.18
)

df2 = df1.withColumn(
    "total_amount",
    col("amount") + col("tax")
)

df3 = df2.withColumn(
    "discount",
    col("amount") * 0.05
)
```

### Alternative Approach

```python
from pyspark.sql.functions import col

result = df.select(
    "*",
    (col("amount") * 0.18).alias("tax"),
    (col("amount") * 1.18).alias("total_amount"),
    (col("amount") * 0.05).alias("discount")
)
```

### Why Consider This?

A single `select()` expresses multiple derived columns together.

It can make the transformation easier to read and may avoid unnecessarily deep projection plans.

**Important:** Spark can collapse many adjacent projections automatically. Multiple `withColumn()` calls are not automatically a performance problem. Inspect the plan before changing working code.

**Concept:** Logical Plan Optimization.

### Interview Question

Can multiple withColumn calls affect performance?

---

## 10. Exercise 9: Choose Between repartition() and coalesce()

<badge color="warning">Intermediate</badge>

**Problem:**

Your DataFrame contains 100 partitions, but you want to write output using fewer partitions.

### Approach 1: repartition()

```python
result = df.repartition(10)
```

- Can increase or decrease partition count.
- Performs a shuffle.
- Redistributes data across partitions.

### Approach 2: coalesce()

```python
result = df.coalesce(10)
```

- Primarily reduces partition count.
- Avoids a full shuffle by default.
- May produce uneven partition sizes.

### Which One Should You Use?

Use `coalesce()` when reducing partitions without requiring full redistribution.

Use `repartition()` when you need redistribution or want to increase partitions.

**Concept:** Partition Management.

### Interview Question

Why is coalesce generally cheaper than repartition when reducing partitions?

---

## 11. Exercise 10: Avoid Unnecessary distinct()

<badge color="warning">Intermediate</badge>

**Problem:**

You need to remove duplicate customer IDs.

### Basic Approach

```python
result = df.select(
    "customer_id"
).distinct()
```

### Alternative

```python
result = df.dropDuplicates(
    ["customer_id"]
)
```

### What Should You Know?

Both approaches perform deduplication and may require a shuffle.

Use `distinct()` when you want unique complete rows.

Use `dropDuplicates(["customer_id"])` when uniqueness is based on a particular key.

**Important:** `dropDuplicates(["customer_id"])` does not guarantee which record is retained. Use a window function when you need to keep the latest record.

**Concept:** Deduplication and Shuffle.

### Interview Question

What is the difference between distinct() and dropDuplicates()?

---

## 12. Exercise 11: Optimize File Reading

<badge color="warning">Intermediate</badge>

**Problem:**

You have a large Parquet dataset, but you need only records from a particular date and a few columns.

### Basic Approach

```python
df = spark.read.parquet(
    "/data/orders"
)

result = df.filter(
    "order_date = '2026-10-01'"
)
```

### Optimized Approach

```python
result = (
    spark.read.parquet("/data/orders")
        .filter("order_date = '2026-10-01'")
        .select(
            "order_id",
            "customer_id",
            "amount"
        )
)
```

### Why Is This Better?

Spark may use:
- Partition pruning.
- Predicate pushdown.
- Projection pushdown.

If the dataset is physically partitioned by `order_date`, partition pruning can avoid reading irrelevant date partitions.

**Important:** Filtering by a date does not automatically guarantee partition pruning. It depends on how the data is stored and whether the filter can be pushed down.

**Concept:** File Reading Optimization.

### Interview Question

What is the difference between partition pruning and predicate pushdown?

---

## 13. Exercise 12: Avoid Python UDFs When Built-in Functions Exist

<badge color="danger">Advanced</badge>

**Problem:**

You need to calculate tax as 18% of the order amount.

### Less Efficient Approach

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import DoubleType

def calculate_tax(amount):
    return amount * 0.18

tax_udf = udf(
    calculate_tax,
    DoubleType()
)

result = df.withColumn(
    "tax",
    tax_udf("amount")
)
```

### Preferred Approach

```python
from pyspark.sql.functions import col

result = df.withColumn(
    "tax",
    col("amount") * 0.18
)
```

### Why Is This Better?

Built-in Spark SQL functions allow Spark to optimize expressions and often avoid Python serialization overhead.

Python UDFs can introduce additional execution overhead and limit some optimizations.

**Important:** UDFs are useful when the required logic cannot be expressed efficiently using built-in functions. Pandas UDFs can improve some Python-heavy workloads, but they are not automatically faster for every use case.

**Concept:** Built-in Functions vs Python UDFs.

### Interview Question

Why are built-in PySpark functions generally preferred over Python UDFs?

---

## 14. Exercise 13: Use Explain to Inspect Execution Plans

<badge color="danger">Advanced</badge>

**Problem:**

You have optimized a join, but you want to verify whether Spark is using a broadcast join.

### Code

```python
result = large_df.join(
    broadcast(small_df),
    "customer_id"
)

result.explain("formatted")
```

### What Should You Look For?

- `BroadcastHashJoin`
- `BroadcastExchange`
- `SortMergeJoin`
- `Exchange`
- `Filter`
- `Project`

### Understanding the Plan

| Plan Element | Meaning |
|---|---|
| BroadcastHashJoin | Broadcast-based join |
| BroadcastExchange | Prepares data for broadcasting |
| SortMergeJoin | Sort-merge join |
| Exchange | Data redistribution or shuffle boundary |
| Filter | Applies filtering |
| Project | Selects or calculates columns |

**Important:** The plan may be adaptive. Runtime behavior can differ from the initial plan when Adaptive Query Execution is enabled.

**Concept:** Physical Execution Plan.

### Interview Question

How do you verify whether a broadcast join is being used?

---

## 15. Exercise 14: Use Adaptive Query Execution

<badge color="danger">Advanced</badge>

**Problem:**

A query has inefficient shuffle partitions and a skewed join.

### Configuration

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

### What Can AQE Do?

Adaptive Query Execution can use runtime statistics to perform supported optimizations, including:

1. Coalescing shuffle partitions.
2. Changing join strategies.
3. Handling certain skewed joins.

### Why Is It Useful?

Spark can adjust parts of the execution plan based on actual runtime statistics instead of relying only on estimates made before execution.

**Important:** AQE does not automatically solve every performance problem. Configuration and behavior depend on Spark version and platform.

### Interview Question

What is AQE, and how does it improve Spark performance?

---

## 16. Exercise 15: Optimize an End-to-End Transformation

<badge color="danger">Advanced</badge>

**Problem:**

You have:
- Orders: 100 million records.
- Customers: 50,000 records.
- Products: 20,000 records.

You need to calculate successful revenue for each customer.

### Basic Implementation

```python
from pyspark.sql.functions import (
    col,
    sum
)

orders = spark.read.parquet(
    "/data/orders"
)

customers = spark.read.parquet(
    "/data/customers"
)

products = spark.read.parquet(
    "/data/products"
)

result = (
    orders
        .join(customers, "customer_id")
        .join(products, "product_id")
        .filter(col("status") == "SUCCESS")
        .groupBy(
            "customer_id",
            "customer_name"
        )
        .agg(
            sum("amount").alias("total_revenue")
        )
)
```

### Optimization Strategy

1. Read only required columns.
2. Apply the status filter early.
3. Broadcast small dimension DataFrames if they fit safely in memory and the join is supported.
4. Avoid unnecessary transformations.
5. Inspect the execution plan.
6. Check for skew and shuffle overhead.
7. Verify the output against the original business logic.

### Optimized Implementation

```python
from pyspark.sql.functions import (
    col,
    sum,
    broadcast
)

orders = (
    spark.read.parquet("/data/orders")
        .filter(col("status") == "SUCCESS")
        .select(
            "customer_id",
            "product_id",
            "amount"
        )
)

customers = (
    spark.read.parquet("/data/customers")
        .select(
            "customer_id",
            "customer_name"
        )
)

products = (
    spark.read.parquet("/data/products")
        .select("product_id")
)

result = (
    orders
        .join(
            broadcast(customers),
            "customer_id"
        )
        .join(
            broadcast(products),
            "product_id"
        )
        .groupBy(
            "customer_id",
            "customer_name"
        )
        .agg(
            sum("amount").alias("total_revenue")
        )
)
```

### Why Is This Better?

- Reads only the required columns.
- Filters unsuccessful orders early.
- Can avoid shuffling the large orders side for suitable broadcast joins.
- Reduces unnecessary data processing.

**Important:** Validate the join cardinality and business logic. If either dimension contains duplicate keys, the join may multiply order records and inflate revenue. Broadcasting also requires sufficient executor memory.

### Interview Question

How would you optimize a PySpark pipeline joining a large fact table with multiple small dimension tables?

---

## 17. Optimization Exercise Summary

| Exercise | Optimization Concept | Difficulty |
|---|---|---|
| 1 | Projection Pushdown | Beginner |
| 2 | Early Filtering | Beginner |
| 3 | Avoid collect() | Beginner |
| 4 | Broadcast Join | Intermediate |
| 5 | Shuffle Reduction | Intermediate |
| 6 | Data Skew and Salting | Intermediate |
| 7 | Cache and Persist | Intermediate |
| 8 | Multiple withColumn Operations | Intermediate |
| 9 | Repartition vs Coalesce | Intermediate |
| 10 | Deduplication | Intermediate |
| 11 | File Reading Optimization | Intermediate |
| 12 | Built-in Functions vs UDFs | Advanced |
| 13 | Explain and Physical Plans | Advanced |
| 14 | Adaptive Query Execution | Advanced |
| 15 | End-to-End Optimization | Advanced |

---

## 18. Interview Questions

**Q1. What are the first things you check when a PySpark job is slow?**

I check the Spark UI, execution plan, shuffle metrics, task duration, data skew, and memory usage. I identify the bottleneck before changing the code.

**Q2. Why should we select only required columns?**

It reduces unnecessary data reading and processing. Columnar formats such as Parquet can benefit from projection pushdown.

**Q3. Why should we filter data early?**

It reduces the number of records processed by subsequent operations and may allow predicate pushdown.

**Q4. When should we use broadcast joins?**

When one side of a join is sufficiently small to be safely broadcast to executors and the join strategy is supported.

**Q5. What is the biggest disadvantage of shuffle?**

Shuffle involves data redistribution, network communication, serialization, and possible disk I/O, which can increase execution time.

**Q6. Does repartition always improve performance?**

No. Repartitioning introduces a shuffle and should be used when the resulting distribution benefits later operations.

**Q7. When should we cache a DataFrame?**

When an expensive DataFrame is reused multiple times and the cost of caching is justified by the saved recomputation.

**Q8. How do you handle data skew?**

I investigate skewed keys and task imbalance. Depending on the situation, I use broadcast joins, salting, AQE, or other appropriate strategies.

**Q9. Why are built-in functions better than Python UDFs in many cases?**

Built-in functions are understood by Spark's optimizer and often avoid the extra Python execution and serialization overhead associated with Python UDFs.

**Q10. How do you verify that an optimization worked?**

I compare execution plans, Spark UI metrics, input/output volume, shuffle activity, and runtime under comparable conditions. I also verify that the output remains correct.

---

## 19. Practical Optimization Checklist

Before deploying a PySpark pipeline, check:

- [ ] Am I reading only required columns?
- [ ] Am I filtering unnecessary records early?
- [ ] Am I avoiding unnecessary `collect()`?
- [ ] Are my joins using suitable strategies?
- [ ] Can a small DataFrame be broadcast safely?
- [ ] Am I creating unnecessary shuffles?
- [ ] Is my data skewed?
- [ ] Am I caching only reused DataFrames?
- [ ] Are my partitions appropriately sized?
- [ ] Am I avoiding unnecessary Python UDFs?
- [ ] Have I inspected the execution plan?
- [ ] Have I checked Spark UI metrics?
- [ ] Have I compared performance before and after?
- [ ] Have I verified output correctness?

---

## 20. How to Practice These Exercises

Do not memorize the optimized code without understanding the reason behind it.

For each exercise:

1. Write the basic implementation yourself.
2. Run it using a small dataset.
3. Inspect the execution plan.
4. Apply the optimization.
5. Inspect the new execution plan.
6. Compare runtime and relevant Spark UI metrics.
7. Explain why the optimization helps.
8. Identify situations where it might not help.

**Final takeaway:** A good Data Engineer does not simply write PySpark code that produces the correct result. They understand data movement, partitioning, joins, memory usage, execution plans, and how to improve performance without changing business logic.