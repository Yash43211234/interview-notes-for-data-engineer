# Apache Spark Fundamentals – Interview Questions and Answers

## 1. Introduction to Apache Spark

### 1.1 What is Apache Spark?

Apache Spark is an open-source distributed data processing engine used to process large datasets across multiple machines.

It supports:
- Batch processing.
- Stream processing.
- SQL queries.
- DataFrame operations.
- Machine learning workloads.
- Graph processing through supported libraries.

Spark distributes computation across a cluster instead of processing the entire dataset on one machine.

### 1.2 Why is Apache Spark Used?

Traditional processing on a single machine can become slow or run out of memory when datasets grow.

Spark helps by:
- Distributing data processing across multiple workers.
- Processing data in parallel.
- Optimizing query execution.
- Supporting multiple data sources and file formats.
- Reusing intermediate data through caching.
- Supporting batch and streaming workloads.

### 1.3 What are the Main Components of Spark?

| Component | Purpose |
|---|---|
| Spark Core | Execution engine, task scheduling, memory management and fault recovery |
| Spark SQL | SQL queries, DataFrames and structured data processing |
| Structured Streaming | Stream processing using the Spark SQL engine |
| MLlib | Distributed machine learning functionality |
| GraphX | Graph processing API, primarily for Scala |

### 1.4 What is the Difference Between Hadoop and Spark?

| Hadoop MapReduce | Apache Spark |
|---|---|
| Disk-oriented execution model | Can reuse data in memory and on disk |
| MapReduce programming model | DataFrame, SQL, RDD and other APIs |
| Typically writes intermediate results to disk | Can pipeline operations and cache intermediate data |
| Separate ecosystem components for different workloads | Integrated processing engine |
| Suitable for large batch workloads | Supports batch and streaming workloads |

Spark does not automatically replace every Hadoop component. It can use Hadoop-compatible storage and resource-management infrastructure.

### 1.5 Is Spark a Database?

No. Spark is a distributed data processing engine, not a traditional database.

It can read and write data from:
- Data lakes.
- Data warehouses.
- Relational databases.
- Cloud object storage.
- Message brokers.

It does not generally act as the primary transactional database for an application.

---

## 2. Spark Architecture

### 2.1 Explain Spark Architecture.

Spark uses a driver-worker architecture.

The main components are:

1. **Driver:** Coordinates application execution.
2. **Cluster Manager:** Allocates resources to Spark applications.
3. **Worker Nodes:** Machines that provide resources for computation.
4. **Executors:** Processes launched for an application on worker nodes.
5. **Tasks:** Units of work executed by executors.

<text>Application execution flow:</text>

```text
             Spark Application
                     |
                     v
               Driver Program
                     |
          Requests Cluster Resources
                     |
                     v
              Cluster Manager
                     |
          Allocates Worker Resources
                     |
          +----------+----------+
          |                     |
          v                     v
      Worker Node 1         Worker Node 2
          |                     |
       Executor              Executor
          |                     |
       Tasks                 Tasks
          |                     |
          +----------+----------+
                     |
                Results / Data
```

### 2.2 What is a Driver?

The driver is the process responsible for coordinating a Spark application.

Its responsibilities include:
- Creating the Spark session and context.
- Building logical execution plans.
- Coordinating job execution.
- Requesting resources.
- Scheduling tasks.
- Collecting task status and results.

The driver does not normally process every distributed record itself.

### 2.3 What is an Executor?

An executor is a process launched for a Spark application on a worker node.

Responsibilities:
- Execute tasks.
- Store cached or persisted data.
- Perform shuffle operations.
- Report task status to the driver.

Executors are generally dedicated to an application for its lifetime in common cluster deployments.

### 2.4 What is a Worker Node?

A worker node is a machine in the cluster that provides resources such as CPU and memory.

It can host one or more executors, depending on the cluster configuration.

### 2.5 What is a Cluster Manager?

A cluster manager allocates compute resources to Spark applications.

Common options:
- Standalone Cluster Manager.
- Hadoop YARN.
- Kubernetes.
- Managed cloud-platform resource management.

The cluster manager allocates resources; Spark's driver handles application-level scheduling.

### 2.6 What is the Difference Between Driver and Executor?

| Driver | Executor |
|---|---|
| Coordinates application execution | Executes tasks |
| Builds and schedules work | Processes assigned partitions |
| Tracks job and task progress | Stores cached data and shuffle data |
| Usually one per application | Usually multiple per application |
| Can become a bottleneck if overloaded | Provides distributed compute capacity |

### 2.7 What is SparkSession?

`SparkSession` is the main entry point for working with Spark SQL and DataFrames.

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("SparkInterview")
    .getOrCreate()
)

df = spark.read.csv(
    "data/customers.csv",
    header=True,
    inferSchema=True
)

df.show()
```

It provides access to:
- DataFrame operations.
- SQL execution.
- Configuration.
- Catalog operations.
- Reading and writing structured data.

### 2.8 What is SparkContext?

`SparkContext` is the lower-level entry point to Spark Core functionality.

It connects a Spark application to the execution environment and supports RDD operations.

In modern PySpark applications, use `SparkSession` as the main entry point. Its underlying context is available through:

```python
sc = spark.sparkContext
```

---

## 3. RDD, DataFrame, and Dataset

### 3.1 What is an RDD?

RDD stands for Resilient Distributed Dataset.

It is an immutable, distributed collection of objects that can be processed in parallel.

Important characteristics:
- Distributed across partitions.
- Immutable.
- Supports transformations and actions.
- Can recover lost partitions using lineage.
- Supports low-level operations.

Example:

```python
numbers = spark.sparkContext.parallelize(
    [1, 2, 3, 4, 5]
)

result = numbers.map(lambda x: x * 2)

print(result.collect())
# [2, 4, 6, 8, 10]
```

### 3.2 What is a DataFrame?

A DataFrame is a distributed collection of data organized into named columns.

It provides a structured API and is optimized through Spark SQL's query optimizer and execution engine.

Example:

```python
df = spark.createDataFrame(
    [
        (1, "Amit", 50000),
        (2, "Rahul", 60000),
        (3, "Neha", 70000)
    ],
    ["id", "name", "salary"]
)

df.show()
```

### 3.3 What is a Dataset?

A Dataset is a strongly typed distributed collection available in Scala and Java Spark APIs.

PySpark does not expose the typed Dataset API in the same way. PySpark DataFrames are the standard structured API.

### 3.4 RDD vs DataFrame vs Dataset

| Feature | RDD | DataFrame | Dataset |
|---|---|---|---|
| Abstraction | Low-level | Structured | Structured and typed |
| Schema | No built-in tabular schema | Yes | Yes |
| Type safety | Language-level object types | Column-based API | Compile-time typed API in Scala/Java |
| Optimization | Less automatic SQL optimization | Catalyst and execution optimizations | Catalyst and execution optimizations |
| PySpark support | Yes | Yes | No typed Dataset API |
| Typical use | Low-level custom processing | Most structured data workloads | Typed Scala/Java workloads |

### 3.5 Why are DataFrames Generally Preferred Over RDDs?

DataFrames are generally preferred for structured data because:
- Spark understands their schema.
- Catalyst can optimize query plans.
- Tungsten-related execution improvements can be applied.
- Column pruning and predicate pushdown may reduce data processed.
- They provide a declarative API.

RDDs remain useful when low-level control or custom object processing is required.

---

## 4. Transformations and Actions

### 4.1 What is a Transformation?

A transformation creates a new RDD or DataFrame from an existing one.

Transformations are generally lazy: Spark records the operation rather than immediately executing the full computation.

Examples:
- `select()`
- `filter()`
- `withColumn()`
- `groupBy()`
- `join()`
- `map()` for RDDs.

```python
filtered_df = df.filter(df.salary > 50000)
```

The transformation defines work but does not normally execute the complete job yet.

### 4.2 What is an Action?

An action requests a result or causes Spark to execute the required computation.

Examples:
- `show()`
- `count()`
- `collect()`
- `take()`
- `write()`
- `first()`.

```python
filtered_df.show()
```

This triggers execution of the required plan.

### 4.3 Transformation vs Action

| Transformation | Action |
|---|---|
| Defines a new computation | Triggers computation |
| Generally lazy | Initiates execution |
| Returns a new RDD/DataFrame | Returns a result or writes output |
| Example: `filter()` | Example: `count()` |

### 4.4 What is Lazy Evaluation?

Lazy evaluation means Spark delays execution of transformations until an action requires their results.

Example:

```python
df1 = df.filter(df.salary > 50000)

df2 = df1.select("name", "salary")

df2.show()
```

Spark can optimize the combined plan before executing it.

Benefits:
- Reduces unnecessary computation.
- Enables query optimization.
- Allows operation pipelining.
- Helps optimize data access.

### 4.5 Does Every Action Create a Separate Job?

Not every action necessarily creates a new job in a simplistic one-to-one way.

An action normally submits work for execution. A single action can involve multiple stages, and some APIs or operations may trigger additional jobs internally.

For interview explanations, say that actions initiate Spark execution rather than assuming every action always corresponds to exactly one job.

---

## 5. Jobs, Stages, and Tasks

### 5.1 What is a Spark Job?

A job is a unit of execution triggered by an action.

A job can contain one or more stages.

### 5.2 What is a Stage?

A stage is a group of tasks that can execute without crossing a shuffle dependency.

Spark generally divides work into stages around shuffle boundaries.

### 5.3 What is a Task?

A task is the smallest unit of execution scheduled by Spark.

For a stage, Spark commonly creates one task per partition being processed by that stage.

### 5.4 Job vs Stage vs Task

| Concept | Meaning |
|---|---|
| Job | Work initiated by an action |
| Stage | Group of tasks separated by shuffle dependencies |
| Task | Work performed on one partition |

### 5.5 Example of Job, Stage, and Task

```python
result = (
    df.filter(df.salary > 50000)
      .groupBy("department")
      .count()
)

result.show()
```

Conceptual execution:

1. `filter()` defines a transformation.
2. `groupBy()` requires records with the same grouping key to be brought together.
3. This usually introduces a shuffle.
4. Spark creates stages around the shuffle dependency.
5. Each stage executes tasks over partitions.
6. `show()` triggers the computation.

The exact physical plan and number of stages depend on Spark's optimization and execution decisions.

---

## 6. Narrow and Wide Transformations

### 6.1 What is a Narrow Transformation?

A narrow transformation allows each output partition to depend on a limited number of input partitions, without requiring a general data redistribution across the cluster.

Examples:
- `filter()`
- `select()`
- `withColumn()`
- Some `map()` operations.

Narrow transformations can often be pipelined within a stage.

### 6.2 What is a Wide Transformation?

A wide transformation requires data from multiple input partitions to be redistributed or combined across partitions.

Examples:
- `groupBy()`
- `distinct()`
- `repartition()`
- Many joins.
- `orderBy()`.

Wide transformations commonly introduce a shuffle.

### 6.3 Narrow vs Wide Transformation

| Narrow | Wide |
|---|---|
| No general cross-partition redistribution | Requires data redistribution |
| Often avoids shuffle | Commonly causes shuffle |
| Can often be pipelined | Usually creates a stage boundary |
| Usually less expensive | Can be expensive |
| Example: `filter()` | Example: `groupBy()` |

### 6.4 Why is Shuffle Expensive?

Shuffle moves data across partitions and often across executors or machines.

It can involve:
- Serialization.
- Network transfer.
- Disk I/O.
- Sorting or hashing.
- Memory pressure.
- Additional task scheduling.

### 6.5 How Can You Reduce Shuffle?

- Filter unnecessary rows early.
- Select only required columns.
- Avoid unnecessary `distinct()`.
- Use appropriate join strategies.
- Broadcast genuinely small tables when suitable.
- Avoid repeated repartitioning.
- Use suitable partitioning for the workload.
- Inspect the execution plan and Spark UI.

---

## 7. Partitions

### 7.1 What is a Partition?

A partition is a logical division of a distributed dataset.

Spark processes partitions in parallel by scheduling tasks.

### 7.2 Why are Partitions Important?

Partitions affect:
- Parallelism.
- Task count.
- Data distribution.
- Shuffle behavior.
- Memory usage.
- Processing performance.

### 7.3 What is `repartition()`?

`repartition()` changes the number of partitions and can repartition data using specified columns.

It generally performs a shuffle.

```python
df_repartitioned = df.repartition(8)
```

Partition by a column:

```python
df_repartitioned = df.repartition(8, "department")
```

This uses hash partitioning by the supplied expression. It does not guarantee balanced data if key frequencies are skewed.

### 7.4 What is `coalesce()`?

`coalesce()` reduces the number of partitions, generally avoiding a full shuffle.

```python
df_coalesced = df.coalesce(4)
```

It is useful when reducing partitions, such as before writing a smaller output.

### 7.5 Repartition vs Coalesce

| Repartition | Coalesce |
|---|---|
| Can increase or decrease partitions | Normally reduces partitions |
| Generally causes shuffle | Usually avoids full shuffle |
| Can redistribute data | May preserve uneven partition sizes |
| Useful for balancing or increasing parallelism | Useful for reducing partition count |

Neither operation guarantees an exact number of output files in every write scenario.

### 7.6 What is Data Skew?

Data skew occurs when some partitions contain significantly more data or work than others.

Example:

A dataset contains a `country` column where one country represents 80% of all records.

A hash partitioning strategy may send a large share of records for that key to one partition.

Consequences:
- A few slow tasks.
- Uneven executor workload.
- Long-running stages.
- Memory pressure.
- Possible task failures.

### 7.7 How Can You Handle Data Skew?

- Broadcast a small table when appropriate.
- Use key salting for skewed keys.
- Repartition using a better distribution strategy.
- Process heavy keys separately.
- Enable Adaptive Query Execution where supported.
- Inspect task-level metrics to confirm skew.

---

## 8. Spark SQL and Catalyst Optimizer

### 8.1 What is Spark SQL?

Spark SQL is the Spark module used for structured data processing through SQL and DataFrame APIs.

It supports:
- SQL queries.
- DataFrames.
- Structured data sources.
- Query optimization.
- Integration with catalogs and tables.

### 8.2 What is Catalyst Optimizer?

Catalyst is Spark SQL's extensible query optimizer.

It transforms logical query plans into optimized plans before execution.

Common optimization examples:
- Predicate pushdown.
- Projection pruning.
- Constant folding.
- Boolean simplification.
- Join optimization.
- Logical plan optimization.

### 8.3 What is Predicate Pushdown?

Predicate pushdown means applying filters as close to the data source as possible, when supported.

Example:

```python
df.filter(df.country == "India")
```

If the source supports filter pushdown, Spark may reduce the amount of data read.

Predicate pushdown is not guaranteed for every source or expression.

### 8.4 What is Projection Pruning?

Projection pruning means reading only the required columns instead of loading every column.

```python
df.select("customer_id", "amount")
```

This can reduce I/O and memory usage, especially with columnar formats such as Parquet.

### 8.5 What is Constant Folding?

Constant folding evaluates expressions involving constants during optimization.

Example:

```sql
SELECT 10 * 5 AS result;
```

The expression can be evaluated during planning rather than repeatedly during execution.

### 8.6 What is `explain()`?

`explain()` displays the execution plan for a DataFrame query.

```python
df.filter(df.salary > 50000).explain(
    mode="formatted"
)
```

Useful for checking:
- Logical plan.
- Physical plan.
- Scan operations.
- Filter placement.
- Join strategy.
- Exchange and shuffle operations.

Available explain modes depend on Spark version.

---

## 9. Spark Execution and Query Optimization

### 9.1 What is Tungsten?

Tungsten refers to Spark's execution-efficiency initiatives, including improvements in memory management and CPU efficiency.

Modern Spark SQL uses optimized execution techniques such as:
- Binary internal data representations.
- Whole-stage code generation for supported operators.
- Efficient memory and CPU usage.

Do not assume every Spark operation uses code generation.

### 9.2 What is Whole-Stage Code Generation?

Whole-stage code generation combines supported physical operators into generated code to reduce execution overhead.

It can improve CPU efficiency by reducing interpretation and virtual function-call overhead.

It does not apply to every operator or workload.

### 9.3 What is a Broadcast Join?

A broadcast join sends a sufficiently small relation to executors so that a large relation can be joined locally on each executor.

Example:

```python
from pyspark.sql.functions import broadcast

result = large_df.join(
    broadcast(small_df),
    on="customer_id",
    how="inner"
)
```

Benefits:
- Can avoid shuffling the large side for supported join strategies.
- Can improve join performance when one side is genuinely small.

Risks:
- Broadcasting a large table can cause executor memory pressure.
- It is not suitable for every join type or condition.
- A hint influences planning but does not guarantee every desired physical strategy.

### 9.4 What is Adaptive Query Execution (AQE)?

AQE allows Spark SQL to revise parts of the physical execution plan using runtime statistics.

Common capabilities include:
- Coalescing small shuffle partitions.
- Changing some sort-merge joins to broadcast joins.
- Handling certain skewed join partitions.

AQE is enabled by default in standard Spark SQL configurations from Spark 3.2, but platform distributions and application settings can override defaults.

### 9.5 How Do You Optimize a Slow Spark Job?

A practical approach:

1. Inspect the Spark UI.
2. Identify slow stages and tasks.
3. Check for shuffle-heavy operations.
4. Check for data skew.
5. Review the physical execution plan.
6. Reduce unnecessary columns and rows.
7. Optimize joins.
8. Review partition count and file sizes.
9. Check memory and spill metrics.
10. Test changes with representative data.

Do not blindly increase executor memory or the number of partitions. First identify the bottleneck.

---

## 10. Fault Tolerance and Lineage

### 10.1 What is Fault Tolerance in Spark?

Fault tolerance is Spark's ability to recover from certain failures and continue computation where possible.

Spark uses:
- RDD lineage.
- Task retries.
- Data recomputation.
- Shuffle data recovery mechanisms.
- Checkpointing for certain workloads.

Recovery behavior depends on the operation, storage, cluster configuration, and failure type.

### 10.2 What is Lineage?

Lineage is the record of transformations used to create an RDD or DataFrame.

Spark can use lineage to recompute lost partitions when necessary.

### 10.3 What is Checkpointing?

Checkpointing saves data to reliable storage and can truncate long lineage chains.

It is useful for:
- Long-running iterative processing.
- Stateful streaming recovery.
- Reducing the cost of recomputing long dependency chains.

RDD checkpointing marks the RDD for checkpointing; an action triggers the materialization. DataFrame checkpointing also depends on the selected eager or lazy behavior.

### 10.4 Cache vs Checkpoint

| Cache/Persist | Checkpoint |
|---|---|
| Keeps computed data for reuse | Saves data to checkpoint storage |
| Primarily improves repeated access | Helps recovery and lineage truncation |
| Can be lost with executor failure | Can provide a more reliable recovery point |
| Does not normally truncate lineage | Truncates lineage after materialization |

Checkpointing is not a substitute for designing reliable source and sink operations.

---

## 11. PySpark Fundamentals

### 11.1 What is PySpark?

PySpark is the Python API for Apache Spark.

It allows developers to use Python to perform distributed data processing.

Common operations include:
- Reading files.
- Filtering records.
- Transforming columns.
- Joining datasets.
- Aggregating data.
- Writing results.

### 11.2 How Do You Create a DataFrame?

```python
data = [
    (1, "Amit", 50000),
    (2, "Rahul", 60000),
    (3, "Neha", 70000)
]

columns = ["id", "name", "salary"]

df = spark.createDataFrame(data, columns)

df.show()
```

### 11.3 How Do You Filter Data?

```python
from pyspark.sql.functions import col

filtered_df = df.filter(
    col("salary") > 50000
)
```

### 11.4 How Do You Add a New Column?

```python
from pyspark.sql.functions import col

df2 = df.withColumn(
    "annual_salary",
    col("salary") * 12
)
```

DataFrames are immutable: `withColumn()` returns a new DataFrame rather than modifying the original.

### 11.5 How Do You Group and Aggregate Data?

```python
from pyspark.sql.functions import avg, count

result = (
    df.groupBy("department")
      .agg(
          count("*").alias("employee_count"),
          avg("salary").alias("average_salary")
      )
)
```

### 11.6 How Do You Join Two DataFrames?

```python
result = employees.join(
    departments,
    on="department_id",
    how="inner"
)
```

Common join types:
- Inner.
- Left.
- Right.
- Full outer.
- Left semi.
- Left anti.

### 11.7 What is the Difference Between `filter()` and `where()`?

In PySpark DataFrame API, `filter()` and `where()` are equivalent ways to filter rows.

```python
df.filter(col("salary") > 50000)

df.where(col("salary") > 50000)
```

### 11.8 What is the Difference Between `select()` and `withColumn()`?

- `select()` chooses or creates a set of output columns.
- `withColumn()` adds or replaces one named column.

Both return new DataFrames.

### 11.9 What is the Difference Between `collect()` and `show()`?

- `collect()` returns all rows to the driver as a Python list.
- `show()` displays a limited number of rows by default.

Avoid `collect()` on large datasets because it can exhaust driver memory.

---

## 12. Spark File Formats

### 12.1 What File Formats Does Spark Support?

Common formats:
- CSV.
- JSON.
- Parquet.
- ORC.
- Avro, when the appropriate support is available.
- Delta Lake tables through Delta Lake integration.

### 12.2 What is Parquet?

Parquet is a columnar file format designed for efficient analytical processing.

Advantages:
- Column pruning.
- Compression.
- Encoding.
- Schema information.
- Efficient analytical reads.

### 12.3 CSV vs JSON vs Parquet

| Feature | CSV | JSON | Parquet |
|---|---|---|---|
| Structure | Delimited text | Semi-structured | Columnar |
| Schema | Not inherently enforced | Flexible | Schema stored in file metadata |
| Nested data | Limited | Natural support | Supported |
| Analytics performance | Often less efficient | Depends on structure and reader | Often efficient |
| Compression | Supported externally or by codec | Supported externally or by codec | Commonly compressed |

### 12.4 How Do You Read and Write Parquet?

```python
df = spark.read.parquet(
    "data/input/"
)

df.write.mode("overwrite").parquet(
    "data/output/"
)
```

Spark generally writes output as a directory containing part files and metadata, not as one guaranteed single file.

### 12.5 What is Partition Pruning?

Partition pruning means skipping storage partitions that cannot match a filter condition.

For example, if data is partitioned by year and a query filters for one year, Spark may read only the relevant partition directories.

Partition pruning is different from predicate pushdown.

---

## 13. Common Spark Performance Problems

### 13.1 Too Many Small Files

Small files increase:
- File listing overhead.
- Metadata operations.
- Task scheduling overhead.
- Query planning work.

Possible solutions:
- Compact files.
- Use suitable output partition counts.
- Avoid excessive partitioning.
- Use optimized write features when available.

### 13.2 Too Few Partitions

Too few partitions may limit parallelism and leave available cluster resources unused.

### 13.3 Too Many Partitions

Too many tiny partitions can increase:
- Task scheduling overhead.
- Metadata overhead.
- Small output file count.

### 13.4 Data Skew

One or a few partitions process much more data than others.

Use Spark UI task metrics to identify unusually long-running tasks and inspect the distribution of keys.

### 13.5 Excessive `collect()`

`collect()` transfers all rows to the driver and can cause driver out-of-memory errors.

Use distributed transformations and aggregations. For inspection, prefer `show()` or `take()` when appropriate.

### 13.6 Repeated Computation

If the same expensive DataFrame is reused multiple times, caching or persisting may reduce repeated computation.

```python
df_filtered = df.filter(
    col("status") == "ACTIVE"
).cache()

df_filtered.count()

# Reuse df_filtered for multiple operations

df_filtered.unpersist()
```

Cache only when reuse justifies the memory and storage cost. Materialize it through an action when you want to populate the cache before reuse.

---

## 14. Important Spark Interview Questions

### Q1. What happens when you call `show()`?

Spark evaluates the required query plan and executes the work needed to retrieve the requested rows for display. It does not necessarily collect the entire DataFrame.

### Q2. Why is Spark called a distributed processing engine?

Because it divides work across partitions and executes tasks in parallel on multiple executors across a cluster.

### Q3. Does Spark always process data in memory?

No. Spark can use memory and disk. Shuffle operations and memory pressure can cause disk I/O.

### Q4. What happens when an executor fails?

Spark may retry failed tasks and recompute lost data using lineage or recover shuffle data, depending on the failure and configuration.

### Q5. Can a DataFrame be modified?

No. DataFrames are immutable. Transformations return new DataFrames.

### Q6. What is the difference between `cache()` and `persist()`?

`cache()` is a convenient default persistence method. `persist()` allows an explicit storage level. Exact default storage levels depend on Spark version and API.

### Q7. What is the difference between `repartition()` and `coalesce()`?

`repartition()` generally shuffles and redistributes data. `coalesce()` generally reduces partitions without a full shuffle.

### Q8. What is a shuffle?

A shuffle redistributes data across partitions, commonly involving network transfer and intermediate storage.

### Q9. Why is `groupBy()` expensive?

It commonly requires records with the same key to be brought together, which introduces a shuffle.

### Q10. What is the difference between `map()` and `flatMap()`?

- `map()` produces one output element for each input element.
- `flatMap()` can produce zero, one, or multiple output elements for each input element.

### Q11. What is the difference between `union()` and `unionByName()`?

- `union()` combines columns by position and retains duplicate rows.
- `unionByName()` combines columns by name.
- `allowMissingColumns=True` can fill missing columns with NULL.

### Q12. What is the difference between `dropDuplicates()` and `distinct()`?

- `distinct()` removes duplicate rows across all selected columns.
- `dropDuplicates()` removes duplicate rows, optionally considering a subset of columns.

When using a subset, do not assume which row will be retained. Use a window function with deterministic ordering when a specific record must be selected.

### Q13. What is the difference between Spark and PySpark?

Spark is the distributed processing engine. PySpark is its Python API.

### Q14. What is the difference between Spark SQL and PySpark DataFrame API?

Spark SQL uses SQL syntax, while PySpark DataFrame API uses Python methods and expressions. Both can use Spark SQL's optimizer and execution engine.

### Q15. What is the Spark UI?

Spark UI is a monitoring interface that helps inspect:
- Jobs.
- Stages.
- Tasks.
- Executors.
- Storage.
- SQL queries.
- Environment configuration.

It is useful for identifying slow stages, skew, shuffle, spills, and executor failures.

---

## 15. Practical Spark Interview Scenario

**Question:** You have a 500 GB customer transaction dataset stored in Parquet. You need to filter transactions, join customer information, and calculate total transaction amount by customer. How would you approach it?

### Step 1: Read Only Required Columns

```python
transactions = (
    spark.read.parquet("data/transactions/")
    .select(
        "customer_id",
        "transaction_amount",
        "transaction_date"
    )
)

customers = (
    spark.read.parquet("data/customers/")
    .select(
        "customer_id",
        "customer_name"
    )
)
```

### Step 2: Filter Unnecessary Records

```python
from pyspark.sql.functions import col

filtered_transactions = transactions.filter(
    col("transaction_amount") > 0
)
```

### Step 3: Choose an Appropriate Join Strategy

If the customer table is genuinely small enough to broadcast safely:

```python
from pyspark.sql.functions import broadcast

joined = filtered_transactions.join(
    broadcast(customers),
    on="customer_id",
    how="inner"
)
```

Otherwise, allow Spark to choose a suitable join strategy or tune it based on statistics and observed performance.

### Step 4: Aggregate Data

```python
from pyspark.sql.functions import sum

result = (
    joined.groupBy(
        "customer_id",
        "customer_name"
    )
    .agg(
        sum("transaction_amount").alias("total_amount")
    )
)
```

### Step 5: Inspect and Optimize

Check:
- Whether only required columns are read.
- Whether filters are pushed down.
- Whether the join causes a large shuffle.
- Whether customer keys are skewed.
- Whether the number of partitions is appropriate.
- Whether the output contains duplicate customer records due to the source grain.

**Interview explanation:** I would begin with source inspection and data-size analysis, reduce unnecessary columns and rows, choose a suitable join strategy, aggregate at the required grain, and use the Spark UI and execution plan to validate performance.

---

## 16. Quick Revision

| Concept | Key Point |
|---|---|
| Apache Spark | Distributed data processing engine |
| Driver | Coordinates application |
| Executor | Executes tasks |
| Worker | Provides compute resources |
| Cluster Manager | Allocates cluster resources |
| SparkSession | Main structured API entry point |
| RDD | Low-level distributed collection |
| DataFrame | Distributed data with named columns |
| Transformation | Defines new computation |
| Action | Triggers execution |
| Lazy Evaluation | Delays execution until required |
| Job | Work initiated by an action |
| Stage | Tasks separated by shuffle dependencies |
| Task | Work on a partition |
| Narrow Transformation | Limited input-partition dependency |
| Wide Transformation | Requires redistribution |
| Shuffle | Data movement across partitions |
| Partition | Unit of distributed data processing |
| Repartition | Redistributes partitions |
| Coalesce | Usually reduces partitions without full shuffle |
| Data Skew | Uneven distribution of work |
| Catalyst | Spark SQL query optimizer |
| Predicate Pushdown | Apply filters near the data source |
| Projection Pruning | Read only required columns |
| Broadcast Join | Send small relation to executors |
| AQE | Adjusts plans using runtime statistics |
| Lineage | Transformation dependency information |
| Checkpoint | Saves data and truncates lineage |
| Parquet | Columnar analytical file format |
| Spark UI | Application monitoring and debugging |

---

## 17. Self-Assessment

Before a Data Engineer interview, make sure you can:

- [ ] Explain Spark architecture.
- [ ] Differentiate driver, executor, worker, and cluster manager.
- [ ] Explain RDD, DataFrame, and Dataset.
- [ ] Differentiate transformations and actions.
- [ ] Explain lazy evaluation.
- [ ] Explain jobs, stages, and tasks.
- [ ] Differentiate narrow and wide transformations.
- [ ] Explain partitions, repartition, and coalesce.
- [ ] Explain data skew and its solutions.
- [ ] Explain shuffle and why it is expensive.
- [ ] Explain Catalyst and `explain()`.
- [ ] Describe broadcast joins and AQE.
- [ ] Explain lineage, caching, and checkpointing.
- [ ] Write basic PySpark DataFrame transformations.
- [ ] Read and write Parquet.
- [ ] Use Spark UI to investigate a slow job.

**Preparation priority:** First master Spark architecture, lazy evaluation, jobs/stages/tasks, transformations/actions, partitions, shuffle, and PySpark DataFrame operations. Then focus on query optimization, joins, AQE, data skew, and fault tolerance.