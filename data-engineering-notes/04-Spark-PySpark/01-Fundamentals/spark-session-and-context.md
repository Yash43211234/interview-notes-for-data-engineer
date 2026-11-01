# SparkSession and SparkContext — Interview Notes

## 1. Introduction

When working with PySpark, two important components are:

- **SparkSession:** The main entry point for working with Spark DataFrames, SQL, and other Spark functionality.
- **SparkContext:** The connection between the Spark application and the underlying cluster execution environment.

Both are important, but they have different responsibilities.

**Simple way to remember:**

- SparkSession → Work with data.
- SparkContext → Connect to and coordinate Spark execution with the cluster.

---

## 2. What Is SparkSession?

`SparkSession` is the primary entry point for using Spark SQL and DataFrame APIs.

It allows us to:
- Read data from different sources.
- Create DataFrames.
- Execute SQL queries.
- Access Spark configuration.
- Manage catalogs and tables.
- Access the underlying SparkContext.

### Create a SparkSession

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("DataEngineeringProject")
    .getOrCreate()
)
```

### Explanation

| Method | Purpose |
|---|---|
| `builder` | Starts the SparkSession builder |
| `appName()` | Sets the application name |
| `getOrCreate()` | Returns an existing session or creates one |

### Example: Read a CSV file

```python
df = spark.read.csv(
    "/data/customers.csv",
    header=True,
    inferSchema=True
)

df.show()
```

Here:
- `spark` is the SparkSession object.
- `read.csv()` reads the CSV data.
- `show()` displays the DataFrame records.

### Example: Execute SQL

```python
df.createOrReplaceTempView("customers")

spark.sql("""
    SELECT *
    FROM customers
    WHERE age > 25
""").show()
```

SparkSession provides the SQL interface for executing this query.

---

## 3. What Is SparkContext?

`SparkContext` is the lower-level entry point to Spark's core execution functionality.

It represents the connection between the Spark application and the cluster.

It is responsible for functionality such as:
- Communicating with the cluster manager.
- Requesting and coordinating execution resources.
- Supporting RDD operations.
- Providing access to Spark application configuration.
- Coordinating distributed execution through Spark's core infrastructure.

### Access SparkContext

```python
sc = spark.sparkContext
```

You do not normally need to create a separate SparkContext when you already have a SparkSession.

### Example: Get application information

```python
print(sc.appName)
print(sc.master)
print(sc.applicationId)
```

These properties help identify the Spark application and its execution environment.

### Example: Create an RDD

```python
numbers = [10, 20, 30, 40, 50]

rdd = sc.parallelize(numbers)

print(rdd.collect())
```

Output:

```text
[10, 20, 30, 40, 50]
```

Here:
- `parallelize()` creates an RDD from a local collection.
- `collect()` returns the RDD elements to the driver.

**Important:** `collect()` should be used carefully with large datasets because it brings all results to the driver.

---

## 4. Relationship Between SparkSession and SparkContext

SparkSession provides a higher-level interface and internally uses SparkContext for core Spark execution.

You can access SparkContext through SparkSession:

```python
spark = (
    SparkSession.builder
    .appName("MySparkApp")
    .getOrCreate()
)

sc = spark.sparkContext
```

Conceptually:

```text
          PySpark Application
                  |
                  v
             SparkSession
                  |
          -----------------
          |               |
       DataFrame         SQL
          |
          v
      SparkContext
          |
          v
      Spark Core
          |
          v
      Cluster Manager
          |
          v
       Executors
```

This is a simplified conceptual relationship. SparkSession also provides access to other Spark SQL facilities, including the catalog and session configuration.

---

## 5. SparkSession vs SparkContext

This comparison is important for interviews.

| Feature | SparkSession | SparkContext |
|---|---|---|
| Level | Higher-level API | Lower-level API |
| Main purpose | DataFrame and SQL operations | Core Spark execution |
| Introduced | Spark 2.0 | Earlier Spark versions |
| DataFrame support | Main entry point | Not its primary purpose |
| SQL support | Yes | No direct SQL interface |
| RDD support | Can access through `spark.sparkContext` | Direct RDD functionality |
| Access | `spark` | `spark.sparkContext` |

### Example

Using SparkSession:

```python
df = spark.read.parquet("/data/orders")

df.filter("amount > 1000").show()
```

Using SparkContext:

```python
rdd = sc.parallelize([10, 20, 30, 40])

result = rdd.map(lambda x: x * 2)

print(result.collect())
```

**Interview takeaway:** Modern PySpark development primarily uses SparkSession. SparkContext remains important for understanding and accessing Spark's core execution functionality.

---

## 6. Why Was SparkSession Introduced?

Before Spark 2.0, developers commonly worked with separate entry points, including:

- `SQLContext`
- `HiveContext`
- `SparkContext`

This could make application setup more complicated.

Spark 2.0 introduced SparkSession as a unified entry point for Spark SQL and DataFrame functionality.

Instead of separately creating SQL-related contexts, developers can use SparkSession for these operations.

Example:

```python
spark = (
    SparkSession.builder
    .appName("UnifiedSparkApplication")
    .enableHiveSupport()
    .getOrCreate()
)
```

This provides a unified interface for working with DataFrames, SQL, and Hive-enabled functionality when the environment is configured appropriately.

---

## 7. Important SparkSession Methods

| Method | Purpose |
|---|---|
| `SparkSession.builder` | Creates a session builder |
| `appName()` | Sets application name |
| `master()` | Sets the master URL |
| `config()` | Sets Spark configuration |
| `getOrCreate()` | Gets an existing session or creates one |
| `read` | Reads external data |
| `sql()` | Executes SQL queries |
| `table()` | Loads a table as a DataFrame |
| `createDataFrame()` | Creates a DataFrame |
| `stop()` | Stops the SparkSession and its associated SparkContext |

### Example: Configure SparkSession

```python
spark = (
    SparkSession.builder
    .appName("CustomerAnalysis")
    .master("local[*]")
    .config("spark.sql.shuffle.partitions", "8")
    .getOrCreate()
)
```

Explanation:

- `appName()` sets the application name.
- `master("local[*]")` runs Spark locally using available processor cores.
- `config()` sets a Spark configuration property.
- `getOrCreate()` obtains or creates the session.

**Note:** In managed platforms such as Databricks, the SparkSession is often already available. You generally do not need to create another one manually.

---

## 8. Important SparkContext Methods and Properties

| Method / Property | Purpose |
|---|---|
| `spark.sparkContext` | Accesses the SparkContext |
| `parallelize()` | Creates an RDD from a local collection |
| `textFile()` | Reads a text file as an RDD |
| `appName` | Returns application name |
| `master` | Returns master URL |
| `applicationId` | Returns application ID |
| `defaultParallelism` | Returns the default level of parallelism |
| `stop()` | Stops the SparkContext |

### Example: Read a text file using SparkContext

```python
sc = spark.sparkContext

rdd = sc.textFile("/data/logs.txt")

print(rdd.take(5))
```

This reads the text file as an RDD and returns up to five records to the driver.

---

## 9. Can We Create Multiple SparkSessions?

Yes. Spark supports multiple SparkSession instances within the same Spark application.

For example:

```python
spark1 = (
    SparkSession.builder
    .appName("MainApplication")
    .getOrCreate()
)

spark2 = spark1.newSession()
```

`newSession()` creates a separate session with isolated SQL configurations, temporary views, and registered functions.

However, both sessions share the same underlying SparkContext and execution resources.

### Why use multiple sessions?

- Separate SQL configurations.
- Separate temporary views.
- Independent session-level settings.
- Different SQL workflows within one Spark application.

**Important distinction:**

| Component | Multiple instances |
|---|---|
| SparkSession | Multiple sessions are supported |
| SparkContext | Normally one active SparkContext per JVM |

A separate SparkSession does not mean a separate Spark application or separate cluster allocation.

---

## 10. SparkSession Configuration vs SparkContext Configuration

Configuration can be set at different levels.

### SparkSession-level SQL configuration

```python
spark.conf.set(
    "spark.sql.shuffle.partitions",
    "20"
)
```

Read the configuration:

```python
spark.conf.get("spark.sql.shuffle.partitions")
```

### SparkContext configuration

```python
sc = spark.sparkContext

print(sc.getConf().get("spark.app.name"))
```

SparkContext exposes the application-level Spark configuration through its `SparkConf`.

**Interview point:** Session-level SQL settings can be isolated between sessions, whereas the underlying SparkContext and application resources are shared.

---

## 11. Common Mistakes

1. **Confusing SparkSession with SparkContext:** SparkSession is the higher-level entry point; SparkContext provides access to core execution functionality.
2. **Creating SparkContext unnecessarily:** When using SparkSession, access the existing context through `spark.sparkContext`.
3. **Assuming multiple sessions mean multiple applications:** Multiple SparkSessions can share one SparkContext.
4. **Using `collect()` on huge datasets:** It can cause driver memory problems.
5. **Creating a new session for every operation:** Usually unnecessary and adds complexity.
6. **Calling `stop()` too early:** It stops the Spark application context and prevents further operations using it.
7. **Assuming `getOrCreate()` always creates a new session:** It can return an existing session.

---

## 12. Common Interview Questions

**Q1. What is SparkSession?**

SparkSession is the unified entry point for working with Spark DataFrames, SQL, catalogs, and related Spark SQL functionality.

**Q2. What is SparkContext?**

SparkContext is the lower-level interface that connects the Spark application to the cluster execution environment and supports core Spark functionality such as RDD operations.

**Q3. What is the relationship between SparkSession and SparkContext?**

SparkSession provides access to SparkContext through `spark.sparkContext`. SparkSession uses the underlying Spark infrastructure to execute DataFrame and SQL operations.

**Q4. Why was SparkSession introduced?**

It was introduced in Spark 2.0 to provide a unified entry point for Spark SQL and DataFrame functionality, replacing the need to separately manage older SQL context entry points.

**Q5. How do you access SparkContext from SparkSession?**

```python
sc = spark.sparkContext
```

**Q6. Can we create multiple SparkSessions?**

Yes. Multiple SparkSessions can exist within one Spark application. They can have separate SQL configurations and temporary views while sharing the underlying SparkContext.

**Q7. Can we create multiple SparkContexts in one application?**

Normally, only one active SparkContext is supported per JVM. Multiple independent Spark applications generally have their own contexts.

**Q8. What is the difference between `getOrCreate()` and `newSession()`?**

- `getOrCreate()` returns an existing SparkSession or creates one if needed.
- `newSession()` creates another SparkSession with separate session-level SQL state while sharing the existing SparkContext.

**Q9. What happens when we call `spark.stop()`?**

It stops the SparkSession and its associated SparkContext, ending the application's Spark execution environment.

**Q10. Which should you use in modern PySpark development?**

Use SparkSession for most DataFrame and SQL operations. Access SparkContext when you specifically need core Spark functionality, such as RDD operations or application-level information.

---

## 13. Quick Revision

| Question | Answer |
|---|---|
| Main entry point for PySpark DataFrames? | SparkSession |
| Entry point for core Spark functionality? | SparkContext |
| Access SparkContext? | `spark.sparkContext` |
| Create or reuse a session? | `getOrCreate()` |
| Create another session? | `newSession()` |
| Create an RDD from a collection? | `sc.parallelize()` |
| Execute SQL? | `spark.sql()` |
| Read a CSV? | `spark.read.csv()` |
| Stop the Spark application? | `spark.stop()` |

### Most important code

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("DataEngineering")
    .getOrCreate()
)

sc = spark.sparkContext

print(sc.appName)
print(sc.applicationId)
```

---

## 14. Self-Check

- [ ] I can explain SparkSession.
- [ ] I can explain SparkContext.
- [ ] I understand their relationship.
- [ ] I can create a SparkSession.
- [ ] I can access SparkContext.
- [ ] I understand `getOrCreate()` and `newSession()`.
- [ ] I know why multiple sessions can share one context.
- [ ] I can explain why SparkSession is preferred for modern DataFrame development.

**Interview takeaway:** SparkSession is the unified entry point for modern PySpark development, while SparkContext provides access to the underlying core Spark execution environment. Understand their relationship rather than treating them as two independent Spark applications.