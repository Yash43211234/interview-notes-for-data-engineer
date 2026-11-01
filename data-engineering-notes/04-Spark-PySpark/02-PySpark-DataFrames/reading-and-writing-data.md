# Reading and Writing Data in PySpark

## 1. Overview

PySpark provides DataFrame reader and writer APIs to load data from different sources and save transformed data to storage systems.

In data engineering, reading and writing data are essential parts of an ETL/ELT pipeline.

Typical workflow:

1. Read raw data from a source.
2. Inspect the schema and records.
3. Clean and transform the data.
4. Write the processed data to a destination.

Common data sources and formats include:

- CSV
- JSON
- Parquet
- ORC
- Delta Lake
- JDBC databases

Import the required functions:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("DataPipeline").getOrCreate()
```

---

## 2. Reading Data Using spark.read

The `spark.read` API is used to load data into a DataFrame.

Basic syntax:

```python
df = spark.read.format("format_name").load("file_path")
```

You can also use format-specific methods such as:

```python
df = spark.read.csv("file_path")
df = spark.read.json("file_path")
df = spark.read.parquet("file_path")
```

---

## 3. Reading CSV Files

CSV stands for Comma-Separated Values.

Example:

```python
df = spark.read.csv(
    "/data/customers.csv",
    header=True,
    inferSchema=True
)
```

Important options:

| Option | Purpose |
|---|---|
| `header` | Uses the first row as column names |
| `inferSchema` | Infers column data types |
| `sep` | Specifies the delimiter |
| `encoding` | Specifies file encoding |
| `nullValue` | Defines the string interpreted as null |
| `mode` | Controls handling of malformed records |
| `quote` | Specifies the quote character |
| `escape` | Specifies the escape character |

Example with options:

```python
df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .option("sep", ",")
    .option("nullValue", "NULL")
    .csv("/data/customers.csv")
)
```

### Reading a CSV Without Inferring Schema

```python
df = spark.read.option(
    "header", True
).csv("/data/customers.csv")
```

By default, CSV columns are read as strings when schema inference is disabled.

### Using an Explicit Schema

For production pipelines, defining a schema is often preferable to relying on inference.

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    StringType,
    DoubleType
)

schema = StructType([
    StructField("customer_id", IntegerType(), True),
    StructField("customer_name", StringType(), True),
    StructField("amount", DoubleType(), True)
])

df = spark.read.csv(
    "/data/customers.csv",
    header=True,
    schema=schema
)
```

**Interview point:** Schema inference is convenient for exploration, while an explicit schema provides better control and consistency in production pipelines.

---

## 4. Reading JSON Files

JSON is commonly used in APIs and semi-structured data pipelines.

Example:

```python
df = spark.read.json("/data/customers.json")
```

You can also use:

```python
df = (
    spark.read
    .option("multiLine", True)
    .json("/data/customers.json")
)
```

Important options:

| Option | Purpose |
|---|---|
| `multiLine` | Reads a JSON document that spans multiple lines |
| `mode` | Controls handling of malformed JSON records |
| `allowComments` | Allows comments in JSON input |
| `allowSingleQuotes` | Allows single quotes in JSON input |

Example JSON:

```json
{"id": 1, "name": "Alice", "city": "Kolkata"}
{"id": 2, "name": "Bob", "city": "Delhi"}
```

Spark can read these JSON records into a DataFrame.

**Important:** JSON supports nested structures such as arrays and objects. The resulting DataFrame can contain nested `StructType` and `ArrayType` columns.

---

## 5. Reading Parquet Files

Parquet is a columnar storage format commonly used in data engineering.

```python
df = spark.read.parquet("/data/customers.parquet")
```

Advantages of Parquet:

1. Columnar storage.
2. Efficient compression.
3. Supports complex data types.
4. Stores schema information.
5. Efficient for analytical queries that read only selected columns.

Example:

```python
df = spark.read.parquet("/data/orders/")
```

Read only required columns:

```python
df = spark.read.parquet(
    "/data/orders/"
).select(
    "order_id",
    "customer_id",
    "amount"
)
```

Spark can use column pruning to avoid reading unnecessary columns when the query plan and data source support it.

---

## 6. Reading Multiple Files

You can read multiple files using a directory path or a list of paths.

### Read All Compatible Files from a Directory

```python
df = spark.read.csv(
    "/data/orders/",
    header=True,
    inferSchema=True
)
```

### Read Specific Files

```python
df = spark.read.csv([
    "/data/orders/january.csv",
    "/data/orders/february.csv"
], header=True, inferSchema=True)
```

### Read Files Using a Wildcard

```python
df = spark.read.csv(
    "/data/orders/*.csv",
    header=True,
    inferSchema=True
)
```

**Important:** Files being combined should have compatible structures and schemas.

---

## 7. Handling Malformed Records

When reading CSV or JSON files, some records may not match the expected schema.

Spark provides different parsing modes.

| Mode | Behavior |
|---|---|
| `PERMISSIVE` | Keeps processing and handles malformed records according to the format and configuration |
| `DROPMALFORMED` | Drops malformed records |
| `FAILFAST` | Stops processing when malformed records are encountered |

Example:

```python
df = (
    spark.read
    .option("mode", "PERMISSIVE")
    .option("columnNameOfCorruptRecord", "_corrupt_record")
    .option("header", True)
    .csv("/data/customers.csv")
)
```

To fail when malformed records are encountered:

```python
df = (
    spark.read
    .option("mode", "FAILFAST")
    .option("header", True)
    .csv("/data/customers.csv")
)
```

**Important:** Malformed-record handling depends on the file format and schema. A value that cannot be parsed into a target type may be handled differently depending on the parsing mode and options.

---

## 8. Writing Data Using df.write

The `df.write` API is used to save a DataFrame.

Basic syntax:

```python
df.write.format("format_name").save("output_path")
```

Format-specific methods:

```python
df.write.csv("output_path")
df.write.json("output_path")
df.write.parquet("output_path")
```

---

## 9. Writing CSV Files

```python
df.write.csv(
    "/output/customers/",
    header=True
)
```

Write with options:

```python
(
    df.write
    .option("header", True)
    .option("sep", ",")
    .mode("overwrite")
    .csv("/output/customers/")
)
```

**Important:** Spark normally writes a directory containing one or more part files, rather than a single CSV file.

---

## 10. Writing JSON Files

```python
df.write.mode("overwrite").json(
    "/output/customers_json/"
)
```

Spark writes the DataFrame as JSON records, generally producing multiple part files.

---

## 11. Writing Parquet Files

```python
df.write.mode("overwrite").parquet(
    "/output/customers_parquet/"
)
```

Parquet is often a good choice for analytical workloads because of its columnar storage and compression.

Example using the format API:

```python
(
    df.write
    .format("parquet")
    .mode("overwrite")
    .save("/output/customers/")
)
```

---

## 12. Save Modes

Save modes control what happens when the output destination already exists.

| Mode | Behavior |
|---|---|
| `append` | Adds new data to existing output |
| `overwrite` | Replaces existing output |
| `ignore` | Does nothing if the destination exists |
| `error` / `errorifexists` | Raises an error if the destination exists |

Examples:

### Append

```python
df.write.mode("append").parquet(
    "/output/orders/"
)
```

### Overwrite

```python
df.write.mode("overwrite").parquet(
    "/output/orders/"
)
```

### Ignore

```python
df.write.mode("ignore").parquet(
    "/output/orders/"
)
```

### Error If Exists

```python
df.write.mode("errorifexists").parquet(
    "/output/orders/"
)
```

**Interview point:** `append` adds data, while `overwrite` replaces the existing output. Always understand the destination and write behavior before running a production pipeline.

---

## 13. Partitioning While Writing

Partitioning organizes output data into directories based on column values.

Example:

```python
df.write.partitionBy("year", "month").parquet(
    "/output/orders/"
)
```

The directory structure may look like:

```text
orders/
    year=2025/
        month=10/
        month=11/
    year=2026/
        month=1/
        month=2/
```

### Why Use Partitioning?

1. Makes data easier to organize.
2. Can reduce the amount of data read by queries.
3. Supports partition pruning when queries filter on partition columns.
4. Helps manage large datasets.

Example query:

```python
df = spark.read.parquet("/output/orders/")

result = df.filter(
    (F.col("year") == 2026) &
    (F.col("month") == 1)
)
```

When partition pruning is supported, Spark can avoid reading irrelevant partitions.

### Important Partitioning Considerations

- Avoid partitioning by columns with extremely high cardinality.
- Too many small partitions can create many small files.
- Choose partition columns based on common query filters.
- Partitioning does not automatically guarantee balanced data.

---

## 14. Controlling the Number of Output Files

Spark writes data using partitions. Each output partition can produce a part file.

### Using repartition()

`repartition()` increases or decreases the number of partitions through a shuffle.

```python
df_repartitioned = df.repartition(4)

df_repartitioned.write.parquet(
    "/output/orders/"
)
```

### Using coalesce()

`coalesce()` can reduce the number of partitions without a full shuffle in the usual case.

```python
df_coalesced = df.coalesce(2)

df_coalesced.write.parquet(
    "/output/orders/"
)
```

**Difference:**

| repartition() | coalesce() |
|---|---|
| Performs a shuffle | Usually avoids a full shuffle when reducing partitions |
| Can increase or decrease partitions | Primarily used to reduce partitions |
| Can redistribute data | May result in uneven partition sizes |

**Important:** Do not use `coalesce(1)` automatically for large datasets. It can create a bottleneck by forcing the output through one partition.

---

## 15. Reading and Writing Delta Tables

Delta Lake adds features such as ACID transactions, schema enforcement, and version history to data stored in Delta format.

### Read Delta Data

```python
df = spark.read.format("delta").load(
    "/data/delta/customers/"
)
```

### Write Delta Data

```python
df.write.format("delta").mode("overwrite").save(
    "/output/delta/customers/"
)
```

### Append to Delta Data

```python
df.write.format("delta").mode("append").save(
    "/output/delta/customers/"
)
```

### Save as a Managed or External Table

```python
df.write.format("delta").saveAsTable(
    "sales.customers"
)
```

The table's storage behavior depends on the catalog and table configuration.

**Interview point:** Delta Lake is useful when pipelines require reliable updates, schema enforcement, and transactional data operations.

---

## 16. Reading and Writing Data from JDBC Sources

PySpark can read from and write to relational databases through JDBC.

### Read from a Database

```python
df = (
    spark.read
    .format("jdbc")
    .option("url", "jdbc:postgresql://host:5432/database")
    .option("dbtable", "public.customers")
    .option("user", "username")
    .option("password", "password")
    .option("driver", "org.postgresql.Driver")
    .load()
)
```

### Write to a Database

```python
(
    df.write
    .format("jdbc")
    .option("url", "jdbc:postgresql://host:5432/database")
    .option("dbtable", "public.customers")
    .option("user", "username")
    .option("password", "password")
    .option("driver", "org.postgresql.Driver")
    .mode("append")
    .save()
)
```

**Important:**

- JDBC requires the appropriate database driver.
- Do not hardcode production credentials in notebooks or source code.
- Use secure credential management.
- For large reads and writes, JDBC partitioning options can help parallelize work.
- Avoid overwhelming the source database with too many concurrent connections.

---

## 17. Schema and Schema Evolution

### Inspect Schema

```python
df.printSchema()
```

### Read with an Explicit Schema

```python
df = spark.read.schema(schema).csv(
    "/data/customers.csv",
    header=True
)
```

### Schema Evolution in Delta Lake

Schema evolution allows compatible schema changes, such as adding new columns, when supported and configured.

Example:

```python
(
    new_df.write
    .format("delta")
    .mode("append")
    .option("mergeSchema", "true")
    .save("/output/delta/customers/")
)
```

**Important:** Schema evolution is not automatically enabled for every write. Understand the target table's schema and the behavior of the configured Delta Lake version.

---

## 18. Practical ETL Pipeline Example

Suppose raw customer data arrives as CSV files. You need to read, clean, and write it as Parquet.

```python
from pyspark.sql import functions as F

# Step 1: Read raw CSV data
df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("/raw/customers/")
)

# Step 2: Clean data
df_clean = (
    df
    .withColumn("name", F.trim("name"))
    .withColumn("email", F.lower(F.trim("email")))
    .filter(F.col("customer_id").isNotNull())
)

# Step 3: Write processed data
(
    df_clean.write
    .format("parquet")
    .mode("overwrite")
    .save("/processed/customers/")
)
```

Pipeline flow:

```text
Raw CSV
   |
   v
Read Data
   |
   v
Inspect Schema
   |
   v
Clean and Transform
   |
   v
Write Parquet
   |
   v
Processed Data
```

This is a basic example of a batch ETL pipeline.

---

## 19. Common Interview Questions

**Q1. How do you read a CSV file in PySpark?**

Use `spark.read.csv()` or `spark.read.format("csv").load()`.

**Q2. What is the difference between CSV and Parquet?**

- CSV is a text-based row-oriented format.
- Parquet is a columnar format designed for efficient analytical workloads.

**Q3. What is the difference between `inferSchema` and an explicit schema?**

- `inferSchema` asks Spark to determine column types from the input.
- An explicit schema defines the expected column names and data types.

**Q4. What are the different write modes in PySpark?**

`append`, `overwrite`, `ignore`, and `errorifexists`.

**Q5. What happens when you write a DataFrame as CSV?**

Spark generally creates an output directory containing one or more part files.

**Q6. What is partitioning in Spark?**

Partitioning organizes output data into directories based on selected column values. It can improve query performance through partition pruning.

**Q7. What is the difference between repartition() and coalesce()?**

- `repartition()` performs a shuffle and can increase or decrease partitions.
- `coalesce()` is mainly used to reduce partitions and usually avoids a full shuffle.

**Q8. Why is Parquet preferred for analytical workloads?**

Because it uses columnar storage, supports compression, and allows efficient reading of selected columns.

**Q9. What is schema evolution?**

It is the ability to accommodate compatible schema changes, such as adding columns, while processing data.

**Q10. How do you read multiple CSV files?**

Read a directory containing the files, pass a list of file paths, or use a wildcard path.

**Q11. How do you handle malformed records?**

Use appropriate parsing modes such as `PERMISSIVE`, `DROPMALFORMED`, or `FAILFAST`, depending on the requirements.

**Q12. What is the difference between save() and saveAsTable()?**

- `save()` writes data to a storage path.
- `saveAsTable()` saves data as a table registered in the catalog.

---

## 20. Common Mistakes

1. Forgetting `header=True` when the CSV file contains column names.
2. Relying on `inferSchema=True` in production without considering schema consistency.
3. Using `overwrite` when existing data should be preserved.
4. Assuming Spark writes one CSV or Parquet file by default.
5. Creating too many small output files.
6. Using `coalesce(1)` for large datasets.
7. Partitioning by columns with very high cardinality.
8. Ignoring malformed records during ingestion.
9. Hardcoding database credentials.
10. Assuming schema evolution is enabled automatically.

---

## 21. Practice Questions

1. Read a CSV file with headers and inferred schema.
2. Read the same file using an explicit schema.
3. Read a JSON file containing multiple records.
4. Read Parquet data and select only three columns.
5. Read all CSV files from a directory.
6. Write a DataFrame as CSV with headers.
7. Write a DataFrame as Parquet using overwrite mode.
8. Append new data to an existing destination.
9. Write data partitioned by year and month.
10. Reduce the number of output partitions before writing.
11. Explain why `coalesce(1)` can be inefficient.
12. Read a Delta table and append new records.
13. Explain how JDBC reads can be parallelized.
14. Build a simple CSV-to-Parquet ETL pipeline.

---

## 22. Quick Revision

| Concept / API | Purpose |
|---|---|
| `spark.read` | Reads data |
| `spark.read.csv()` | Reads CSV |
| `spark.read.json()` | Reads JSON |
| `spark.read.parquet()` | Reads Parquet |
| `df.write` | Writes data |
| `df.write.csv()` | Writes CSV |
| `df.write.json()` | Writes JSON |
| `df.write.parquet()` | Writes Parquet |
| `schema()` | Defines an explicit schema |
| `printSchema()` | Displays schema |
| `append` | Adds data |
| `overwrite` | Replaces existing output |
| `ignore` | Skips writing if destination exists |
| `errorifexists` | Raises an error if destination exists |
| `partitionBy()` | Organizes output into partition directories |
| `repartition()` | Redistributes data through a shuffle |
| `coalesce()` | Reduces partitions, usually without a full shuffle |
| `saveAsTable()` | Saves data as a catalog table |
| `format("delta")` | Reads or writes Delta data |

---

## 23. Self-Assessment

- [ ] Read CSV, JSON, and Parquet files.
- [ ] Define an explicit schema.
- [ ] Explain schema inference.
- [ ] Read multiple files.
- [ ] Handle malformed records.
- [ ] Write data in different formats.
- [ ] Explain all four save modes.
- [ ] Write partitioned data.
- [ ] Explain repartition and coalesce.
- [ ] Read and write Delta data.
- [ ] Explain basic JDBC operations.
- [ ] Build a CSV-to-Parquet ETL pipeline.

**Interview readiness:** You should be able to explain how data moves from a source into Spark, how schema and malformed records are handled, and how transformed data is safely written to a destination.