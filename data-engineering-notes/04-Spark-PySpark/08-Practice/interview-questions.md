# PySpark DataFrame Interview Questions

This document covers commonly asked PySpark DataFrame interview questions, from fundamentals to practical data engineering scenarios.

## 1. PySpark Fundamentals

### Q1. What is PySpark?

PySpark is the Python API for Apache Spark. It allows us to process large datasets in a distributed environment using Python.

It supports:
- DataFrame operations.
- SQL queries.
- Data processing and transformations.
- Batch processing.
- Structured Streaming.
- Machine learning through Spark MLlib.

### Q2. What is a DataFrame in PySpark?

A DataFrame is a distributed collection of data organized into named columns.

It is similar to a table in a relational database, but it is distributed across multiple partitions and can be processed across a Spark cluster.

### Q3. What is the difference between RDD and DataFrame?

| RDD | DataFrame |
|---|---|
| Low-level distributed collection | Structured data with named columns |
| Does not have a tabular schema by default | Has a schema |
| Less optimization for structured queries | Uses Spark SQL optimizations |
| More control over low-level operations | Easier to use for structured processing |
| Requires more manual coding | Supports SQL-like operations |

**Interview point:** DataFrames are generally preferred for structured data processing because Spark can optimize their execution using the Catalyst Optimizer and other execution optimizations.

### Q4. What is SparkSession?

SparkSession is the main entry point for working with Spark DataFrames and Spark SQL.

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
        .appName("DataEngineering")
        .getOrCreate()
)
```

### Q5. What is the difference between `createDataFrame()` and `read`?

- `createDataFrame()` creates a DataFrame from existing Python data, RDDs, or supported data structures.
- `spark.read` loads data from external sources such as CSV, JSON, Parquet, and databases.

### Q6. What is a schema?

A schema defines the structure of a DataFrame, including column names, data types, and nullability metadata.

We can inspect it using:

```python
df.printSchema()
```

### Q7. What is the difference between StructType and StructField?

- `StructType` represents the complete schema.
- `StructField` represents an individual column definition.

### Q8. What is the difference between `show()`, `collect()`, and `take()`?

| Method | Purpose |
|---|---|
| `show()` | Displays a limited number of rows |
| `collect()` | Returns all rows to the driver |
| `take(n)` | Returns up to n rows to the driver |

**Important:** Avoid using `collect()` on large datasets because it can cause driver out-of-memory errors.

---

## 2. Transformations and Actions

### Q9. What is a transformation in PySpark?

A transformation creates a new DataFrame or RDD from an existing one.

Examples:

```python
df.select("name")

df.filter("salary > 50000")

df.groupBy("department").count()
```

### Q10. What is an action?

An action triggers execution and returns a result or writes data.

Examples:

```python
df.show()

df.count()

df.collect()

df.write.parquet("/output")
```

### Q11. What is lazy evaluation?

Lazy evaluation means Spark does not immediately execute most transformations.

Instead, Spark builds a logical execution plan and executes it when an action or another execution-triggering operation requires results.

Example:

```python
df1 = df.filter("amount > 100")

df2 = df1.select("order_id", "amount")

df2.show()
```

The transformations are evaluated when `show()` requests the result.

### Q12. What is the difference between narrow and wide transformations?

| Narrow Transformation | Wide Transformation |
|---|---|
| Usually does not require data movement between partitions | Requires data exchange between partitions |
| Usually no shuffle | Usually involves shuffle |
| Examples: `filter()`, `select()` | Examples: `groupBy()`, `distinct()` |

**Important:** Some operations, such as joins, may use different physical strategies. Do not assume every join always requires a shuffle.

### Q13. What is the difference between `filter()` and `where()`?

Both methods perform row filtering and are equivalent in PySpark DataFrames.

```python
df.filter("amount > 100")

df.where("amount > 100")
```

### Q14. What is the difference between `select()` and `withColumn()`?

- `select()` chooses or creates the columns included in the output.
- `withColumn()` adds a new column or replaces an existing column.

```python
from pyspark.sql.functions import col

df.select("order_id", "amount")

df.withColumn(
    "tax",
    col("amount") * 0.18
)
```

### Q15. Are PySpark DataFrames mutable?

No. DataFrames are immutable.

Operations such as `filter()` and `withColumn()` return new DataFrames rather than modifying the original DataFrame.

---

## 3. Filtering and Null Handling

### Q16. How do you filter records based on multiple conditions?

Use `&` for AND, `|` for OR, and `~` for NOT.

```python
from pyspark.sql.functions import col

result = df.filter(
    (col("amount") > 100) &
    (col("status") == "SUCCESS")
)
```

Use parentheses around individual conditions.

### Q17. How do you filter NULL values?

```python
df.filter(col("customer_id").isNull())
```

To filter non-NULL values:

```python
df.filter(col("customer_id").isNotNull())
```

### Q18. What is the difference between NULL and empty string?

- NULL represents a missing or unknown value.
- An empty string `""` is a valid string containing zero characters.

They are not equivalent.

### Q19. How do you replace NULL values?

Using `fillna()`:

```python
df.fillna({
    "amount": 0,
    "customer_name": "Unknown"
})
```

Or using `coalesce()`:

```python
from pyspark.sql.functions import coalesce, col, lit

df.withColumn(
    "amount",
    coalesce(col("amount"), lit(0))
)
```

### Q20. What is the difference between `dropna()` and `fillna()`?

- `dropna()` removes rows containing NULL values according to the selected conditions.
- `fillna()` replaces NULL values with specified values.

---

## 4. Aggregations and Grouping

### Q21. What is `groupBy()`?

`groupBy()` groups records based on one or more columns so that aggregate calculations can be performed.

```python
from pyspark.sql.functions import sum

result = (
    df.groupBy("customer_id")
      .agg(sum("amount").alias("total_amount"))
)
```

### Q22. What is the difference between `groupBy()` and `agg()`?

- `groupBy()` defines grouping columns.
- `agg()` applies aggregate expressions.

Example:

```python
df.groupBy("department").agg(
    sum("salary").alias("total_salary")
)
```

### Q23. How do you calculate multiple aggregations?

```python
from pyspark.sql.functions import (
    sum,
    avg,
    count,
    max,
    min
)

result = df.groupBy("department").agg(
    sum("salary").alias("total_salary"),
    avg("salary").alias("average_salary"),
    count("*").alias("employee_count"),
    max("salary").alias("maximum_salary"),
    min("salary").alias("minimum_salary")
)
```

### Q24. What is the difference between `count()` and `countDistinct()`?

- `count()` counts records or non-NULL values in a column, depending on the expression.
- `countDistinct()` counts distinct non-NULL values.

```python
from pyspark.sql.functions import count, countDistinct

df.agg(
    count("*").alias("total_rows"),
    countDistinct("customer_id").alias("unique_customers")
)
```

### Q25. How do you filter aggregated results?

Use `filter()` or `where()` after `groupBy()` and aggregation.

```python
result = (
    df.groupBy("department")
      .agg(sum("salary").alias("total_salary"))
      .filter(col("total_salary") > 100000)
)
```

---

## 5. Joins

### Q26. What is a join in PySpark?

A join combines two DataFrames based on a common column or join condition.

### Q27. What are the different types of joins?

| Join Type | Description |
|---|---|
| Inner | Matching records from both DataFrames |
| Left | All left records and matching right records |
| Right | All right records and matching left records |
| Full Outer | All records from both DataFrames |
| Left Semi | Matching left-side records only |
| Left Anti | Non-matching left-side records only |
| Cross | Cartesian product |

### Q28. How do you perform an inner join?

```python
result = orders.join(
    customers,
    on="customer_id",
    how="inner"
)
```

### Q29. What is the difference between left semi and left anti joins?

**Left Semi:**
Returns records from the left DataFrame that have a matching record in the right DataFrame.

**Left Anti:**
Returns records from the left DataFrame that do not have a matching record in the right DataFrame.

```python
matched = orders.join(
    customers,
    "customer_id",
    "left_semi"
)

unmatched = orders.join(
    customers,
    "customer_id",
    "left_anti"
)
```

### Q30. How do you avoid duplicate column names after joining?

Select the required columns explicitly or use a shared join key.

```python
result = orders.join(
    customers,
    on="customer_id",
    how="inner"
).select(
    "order_id",
    "customer_id",
    "amount",
    "customer_name"
)
```

### Q31. What is a broadcast join?

A broadcast join sends a small DataFrame to the worker nodes so that it can be joined with a larger DataFrame without shuffling the large side for that join.

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    "customer_id"
)
```

**Important:** Broadcasting is useful when the smaller dataset can safely fit in executor memory. A broadcast hint does not guarantee that every query will use a broadcast join.

---

## 6. Union and Deduplication

### Q32. What is the difference between `union()` and `unionByName()`?

| union() | unionByName() |
|---|---|
| Matches columns by position | Matches columns by name |
| Requires compatible column structures | Can handle different column order |
| Does not remove duplicates | Does not remove duplicates |

### Q33. Does `union()` remove duplicate records?

No. `union()` behaves like SQL `UNION ALL` and retains duplicate rows.

Use `distinct()` or `dropDuplicates()` when duplicate removal is required.

### Q34. How do you remove duplicate records?

```python
df.dropDuplicates()
```

To remove duplicates based on a particular key:

```python
df.dropDuplicates(["order_id"])
```

### Q35. How do you keep the latest record for each customer?

Use a window function with a deterministic ordering.

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number, col

window_spec = (
    Window.partitionBy("customer_id")
          .orderBy(
              col("updated_at").desc(),
              col("record_id").desc()
          )
)

result = (
    df.withColumn(
        "rn",
        row_number().over(window_spec)
    )
    .filter(col("rn") == 1)
    .drop("rn")
)
```

The second ordering column provides a tie-breaker when timestamps are equal.

---

## 7. Window Functions

### Q36. What is a window function?

A window function performs calculations across a group of related rows without collapsing them into one row per group.

### Q37. What are common PySpark window functions?

- `row_number()`
- `rank()`
- `dense_rank()`
- `lag()`
- `lead()`
- `sum()`
- `avg()`

### Q38. What is the difference between row_number, rank, and dense_rank?

| Function | Behavior |
|---|---|
| row_number | Assigns a unique sequential number |
| rank | Gives equal rank to ties and leaves gaps |
| dense_rank | Gives equal rank to ties without gaps |

Example scores: 100, 100, 90.

| Score | row_number* | rank | dense_rank |
|---:|---:|---:|---:|
| 100 | 1 or 2 | 1 | 1 |
| 100 | 2 or 1 | 1 | 1 |
| 90 | 3 | 3 | 2 |

*The order between tied rows is not deterministic unless an additional tie-breaker is specified.*

### Q39. What is the difference between lag() and lead()?

- `lag()` accesses a previous row.
- `lead()` accesses a following row.

```python
from pyspark.sql.functions import lag, lead

result = df.withColumn(
    "previous_amount",
    lag("amount").over(window_spec)
).withColumn(
    "next_amount",
    lead("amount").over(window_spec)
)
```

### Q40. How do you calculate a running total?

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import sum

window_spec = (
    Window.partitionBy("customer_id")
          .orderBy("order_date", "order_id")
          .rowsBetween(
              Window.unboundedPreceding,
              Window.currentRow
          )
)

result = df.withColumn(
    "running_total",
    sum("amount").over(window_spec)
)
```

---

## 8. Reading and Writing Data

### Q41. How do you read a CSV file?

```python
df = (
    spark.read
        .format("csv")
        .option("header", True)
        .option("inferSchema", True)
        .load("/data/orders.csv")
)
```

For production pipelines, an explicit schema is often preferable.

### Q42. How do you read a Parquet file?

```python
df = spark.read.parquet("/data/orders")
```

### Q43. What is the difference between CSV and Parquet?

| CSV | Parquet |
|---|---|
| Text-based format | Columnar binary format |
| Does not inherently store a complete typed schema | Stores schema metadata |
| Often requires more storage | Usually offers better compression |
| Common for data exchange | Common for analytical workloads |

### Q44. How do you write a DataFrame?

```python
df.write \
    .mode("overwrite") \
    .parquet("/output/orders")
```

### Q45. What are common write modes?

| Mode | Behavior |
|---|---|
| append | Adds output to existing data |
| overwrite | Replaces existing output |
| ignore | Does nothing if output already exists |
| errorifexists | Fails if output already exists |

### Q46. Why does Spark create multiple output files?

Spark processes data in partitions. Distributed writes generally create output files from the partitions writing the data.

The number of files depends on the number of non-empty writer partitions and the write configuration.

---

## 9. Performance and Optimization

### Q47. What is a shuffle?

A shuffle is the process of redistributing data across partitions or executors.

It commonly occurs during operations such as:
- `groupBy()`
- `distinct()`
- Some joins.
- `repartition()`.

Shuffles involve network and disk I/O and can be expensive.

### Q48. What is partitioning in PySpark?

Partitioning divides data into smaller logical pieces so Spark can process them in parallel.

A DataFrame consists of partitions distributed across the cluster.

### Q49. What is the difference between repartition and coalesce?

| repartition() | coalesce() |
|---|---|
| Can increase or decrease partitions | Primarily reduces partitions |
| Usually performs a shuffle | Avoids a full shuffle by default |
| Can redistribute data more evenly | May produce uneven partition sizes |

```python
df.repartition(10)

df.coalesce(5)
```

**Important:** `repartition()` by a column uses hash partitioning, but it does not automatically solve data skew.

### Q50. What is caching?

Caching stores computed data so that it can be reused without recomputing the entire lineage.

```python
df.cache()
```

Caching is lazy. The DataFrame is materialized when an action evaluates the required data.

### Q51. What is the difference between cache and persist?

- `cache()` uses the default caching storage level for the relevant API and Spark version.
- `persist()` allows an explicit storage level to be selected.

```python
from pyspark import StorageLevel

df.persist(StorageLevel.MEMORY_AND_DISK)
```

### Q52. What is data skew?

Data skew occurs when some partitions contain significantly more data than others.

It can cause:
- Uneven task execution time.
- Long-running tasks.
- Memory pressure.
- Slow joins and aggregations.

Common solutions:
- Broadcast joins where appropriate.
- Salting skewed keys.
- Adaptive Query Execution.
- Repartitioning with a suitable strategy.
- Pre-aggregation where valid.

### Q53. What is the Catalyst Optimizer?

The Catalyst Optimizer is Spark SQL's query optimization framework.

It analyzes and optimizes logical query plans before physical execution.

### Q54. What is Adaptive Query Execution (AQE)?

AQE optimizes query execution using runtime statistics.

It can perform supported optimizations such as:
- Coalescing shuffle partitions.
- Changing join strategies.
- Handling certain skewed joins.

### Q55. What is predicate pushdown?

Predicate pushdown means pushing supported filter conditions closer to the data source so that less data needs to be read.

### Q56. What is projection pushdown?

Projection pushdown means reading only the required columns instead of loading every column.

It is particularly useful with columnar formats such as Parquet.

### Q57. How do you investigate a slow PySpark job?

I would:
1. Check the Spark UI.
2. Identify slow stages and tasks.
3. Inspect shuffle read and write.
4. Check data skew.
5. Review the physical execution plan.
6. Check memory and spill metrics.
7. Review joins and partitioning.
8. Optimize unnecessary transformations.

---

## 10. Data Quality and Schema Validation

### Q58. How do you validate a DataFrame schema?

Compare expected column names and data types with the actual DataFrame schema.

```python
expected_columns = [
    "order_id",
    "customer_id",
    "amount"
]

missing = set(expected_columns) - set(df.columns)
```

### Q59. How do you identify invalid records?

Use validation conditions and separate valid and invalid records.

```python
valid_condition = (
    col("order_id").isNotNull() &
    col("amount").isNotNull() &
    (col("amount") > 0)
)

valid_df = df.filter(valid_condition)

invalid_df = df.filter(~valid_condition)
```

### Q60. What is the difference between schema validation and data quality validation?

- Schema validation checks the structure and data types.
- Data quality validation checks whether actual values satisfy defined rules.

For example, `amount` may have the correct DoubleType but still contain a negative value.

### Q61. How do you handle malformed records?

Depending on the source and format, configure appropriate parsing options, capture corrupt records, and separate invalid data for investigation.

For CSV and JSON, Spark provides parsing modes such as:
- `PERMISSIVE`
- `DROPMALFORMED`
- `FAILFAST`

The exact behavior and corrupt-record handling depend on the source format and configuration.

---

## 11. Practical Scenario-Based Questions

### Q62. You have a large orders DataFrame. How would you find the top three orders for each customer?

Use `row_number()` with a window partitioned by customer.

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number, col

window_spec = (
    Window.partitionBy("customer_id")
          .orderBy(
              col("amount").desc(),
              col("order_id").asc()
          )
)

result = (
    df.withColumn(
        "rn",
        row_number().over(window_spec)
    )
    .filter(col("rn") <= 3)
    .drop("rn")
)
```

### Q63. How would you find customers who have no orders?

Use a left anti join.

```python
result = customers.join(
    orders.select("customer_id").distinct(),
    on="customer_id",
    how="left_anti"
)
```

### Q64. How would you find duplicate order IDs?

```python
from pyspark.sql.functions import count

result = (
    df.groupBy("order_id")
      .agg(count("*").alias("record_count"))
      .filter(col("record_count") > 1)
)
```

### Q65. How would you find the second-highest salary in each department?

Use `dense_rank()`.

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import dense_rank, col

window_spec = (
    Window.partitionBy("department")
          .orderBy(col("salary").desc())
)

result = (
    df.withColumn(
        "salary_rank",
        dense_rank().over(window_spec)
    )
    .filter(col("salary_rank") == 2)
)
```

### Q66. How would you calculate the total revenue by month?

```python
from pyspark.sql.functions import (
    sum,
    date_format
)

result = (
    df.withColumn(
        "month",
        date_format("order_date", "yyyy-MM")
    )
    .groupBy("month")
    .agg(
        sum("amount").alias("total_revenue")
    )
)
```

### Q67. How would you process a very large dataset efficiently?

I would:
1. Read only required columns.
2. Apply filters early.
3. Use efficient formats such as Parquet or Delta.
4. Avoid unnecessary shuffles.
5. Optimize joins.
6. Handle skewed data.
7. Cache only reused DataFrames.
8. Inspect the Spark UI and execution plan.

### Q68. How would you handle a new column arriving in a source dataset?

I would first identify whether the column is expected and whether downstream systems need it.

Then I would:
- Validate the new schema.
- Decide whether to accept or reject the change.
- Update transformations if necessary.
- Apply supported schema evolution where appropriate.
- Test downstream compatibility.

---

## 12. Important Coding Patterns to Remember

| Requirement | PySpark Approach |
|---|---|
| Filter records | `filter()` / `where()` |
| Select columns | `select()` |
| Add a column | `withColumn()` |
| Rename a column | `withColumnRenamed()` |
| Remove a column | `drop()` |
| Remove NULL rows | `dropna()` |
| Replace NULL values | `fillna()` |
| Group records | `groupBy()` |
| Aggregate values | `agg()` |
| Join DataFrames | `join()` |
| Remove duplicates | `dropDuplicates()` |
| Combine DataFrames | `union()` |
| Combine by column name | `unionByName()` |
| Rank records | `rank()` / `dense_rank()` |
| Number records | `row_number()` |
| Previous row | `lag()` |
| Next row | `lead()` |
| Sort records | `orderBy()` |
| Change partitions | `repartition()` / `coalesce()` |
| Persist results | `cache()` / `persist()` |
| Inspect schema | `printSchema()` |
| Inspect execution plan | `explain()` |

---

## 13. Interview Preparation: What to Prioritize

For Data Engineer interviews, focus on being able to write code and explain why you selected a particular approach.

| Priority | Topic | What You Should Be Able to Do |
|---|---|---|
| High | DataFrame fundamentals | Create, select, filter, transform |
| High | Aggregations | Group and calculate metrics |
| High | Joins | Explain and implement join types |
| High | Window functions | Ranking, latest records, running totals |
| High | NULL handling | Identify and handle missing values |
| High | Deduplication | Remove duplicates and retain latest records |
| High | Performance | Explain shuffle, skew, partitions, broadcast |
| Medium | Schema validation | Define and compare schemas |
| Medium | Reading and writing | Work with CSV, Parquet, Delta |
| Medium | Data quality | Validate and quarantine invalid records |
| Medium | Unit testing | Test transformations with Pytest |
| Medium | Streaming | Understand streaming DataFrames and micro-batches |

### How to Answer Practical Questions

Use this simple structure:

1. Understand the requirement.
2. Identify the correct PySpark operation.
3. Write the code.
4. Explain why the approach works.
5. Mention edge cases and performance considerations.

---

## 14. Quick Revision

- DataFrame → Distributed structured dataset.
- SparkSession → Entry point for DataFrame operations.
- Transformation → Creates a new DataFrame or RDD.
- Action → Triggers execution or returns results.
- Lazy evaluation → Defers execution until results are needed.
- Narrow transformation → No shuffle in the normal execution model.
- Wide transformation → Requires data redistribution.
- `filter()` → Filters rows.
- `withColumn()` → Adds or replaces a column.
- `groupBy()` → Groups records.
- `agg()` → Performs aggregations.
- `join()` → Combines DataFrames.
- `left_semi` → Matching left records.
- `left_anti` → Non-matching left records.
- `union()` → Combines by position and retains duplicates.
- `unionByName()` → Combines by column name.
- `row_number()` → Unique sequential ranking.
- `rank()` → Ranking with gaps.
- `dense_rank()` → Ranking without gaps.
- `lag()` → Previous row.
- `lead()` → Next row.
- `repartition()` → Redistributes partitions using a shuffle.
- `coalesce()` → Reduces partitions without a full shuffle by default.
- `cache()` → Reuses computed data.
- Broadcast join → Shares a small dataset with workers.
- Data skew → Uneven distribution of data.
- Catalyst → Query optimization framework.
- AQE → Runtime query optimization.

---

## 15. Self-Assessment Checklist

### Fundamentals
- [ ] Can I explain DataFrame vs RDD?
- [ ] Can I explain SparkSession?
- [ ] Can I explain transformations and actions?
- [ ] Can I explain lazy evaluation?

### DataFrame Operations
- [ ] Can I use select, filter, and withColumn?
- [ ] Can I handle NULL values?
- [ ] Can I perform aggregations?
- [ ] Can I implement different joins?
- [ ] Can I remove duplicates?
- [ ] Can I use union and unionByName?

### Advanced Operations
- [ ] Can I write window functions?
- [ ] Can I calculate running totals?
- [ ] Can I find latest records?
- [ ] Can I solve ranking problems?

### Performance
- [ ] Can I explain shuffle?
- [ ] Can I explain repartition vs coalesce?
- [ ] Can I explain caching?
- [ ] Can I explain data skew?
- [ ] Can I explain broadcast joins?
- [ ] Can I use Spark UI to debug slow jobs?

### Data Engineering
- [ ] Can I validate schemas?
- [ ] Can I implement data quality checks?
- [ ] Can I read and write different file formats?
- [ ] Can I write unit tests?
- [ ] Can I explain basic Structured Streaming?

**Final takeaway:** For PySpark interviews, knowing the syntax is not enough. You should be able to solve DataFrame problems, explain execution behavior, handle edge cases, and understand how your code performs on large datasets.