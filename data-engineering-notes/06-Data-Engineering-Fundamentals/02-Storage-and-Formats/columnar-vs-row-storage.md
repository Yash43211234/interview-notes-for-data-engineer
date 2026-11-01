# Columnar Storage vs Row Storage

## 1. Introduction

Data can be stored in different physical layouts depending on how applications access and process it.

Two important storage layouts are:

1. **Row-based storage**
2. **Columnar storage**

The main difference is how values are organized on disk.

- Row storage keeps all column values of a record together.
- Columnar storage keeps values from the same column together.

Understanding this difference is important in data engineering because storage layout affects query performance, compression, and analytical workloads.

## 2. What Is Row-Based Storage?

**Row-based storage** organizes data record by record. All values belonging to one row are stored together.

### Example

Consider an employee table:

| employee_id | name | department | salary |
|---:|---|---|---:|
| 101 | Rahul | IT | 50000 |
| 102 | Priya | HR | 60000 |
| 103 | Amit | IT | 55000 |

Conceptually, row storage looks like this:

```text
ROW 1: 101, Rahul, IT, 50000
ROW 2: 102, Priya, HR, 60000
ROW 3: 103, Amit, IT, 55000
```

All values for one employee are stored together.

### How It Works

If an application wants employee 102, the database can retrieve the corresponding row containing:

```text
102, Priya, HR, 60000
```

It is particularly useful when applications frequently insert, update, or retrieve complete records.

### Common Examples

- MySQL with InnoDB.
- PostgreSQL.
- SQL Server rowstore tables.
- Traditional transactional database storage.

### Characteristics

- Stores complete records together.
- Efficient for retrieving individual rows.
- Suitable for transactional workloads.
- Supports frequent inserts and updates efficiently.
- Analytical queries that read only a few columns may require more unnecessary data reads.

## 3. What Is Columnar Storage?

**Columnar storage** organizes data column by column.

Instead of storing all values of one record together, it stores values belonging to the same column together.

Using the same employee table:

```text
employee_id: 101, 102, 103

name: Rahul, Priya, Amit

department: IT, HR, IT

salary: 50000, 60000, 55000
```

Each column is stored separately or in column-oriented data structures within the file.

### How It Works

Suppose a business analyst wants to calculate the average salary.

```sql
SELECT AVG(salary)
FROM employees;
```

The query needs only the `salary` column.

A columnar storage engine can read the required column without reading all the values from `employee_id`, `name`, and `department`.

This can significantly reduce data scanned for analytical queries.

### Common Columnar Formats

- Apache Parquet.
- Apache ORC.

### Characteristics

- Stores values column by column.
- Efficient for analytical queries.
- Supports reading only required columns.
- Often provides better compression for similar values.
- Can reduce disk I/O for queries that access a small number of columns.
- Frequent individual record updates may be less efficient in many columnar file-based systems.

## 4. Visual Comparison

Consider this employee dataset:

| ID | Name | Department | Salary |
|---:|---|---|---:|
| 101 | Rahul | IT | 50000 |
| 102 | Priya | HR | 60000 |
| 103 | Amit | IT | 55000 |

### Row Storage

```text
+----------------------------------+
| 101 | Rahul | IT | 50000         |
+----------------------------------+
| 102 | Priya | HR | 60000         |
+----------------------------------+
| 103 | Amit  | IT | 55000         |
+----------------------------------+
```

### Columnar Storage

```text
+----------+----------+------------+----------+
| ID       | Name     | Department | Salary   |
+----------+----------+------------+----------+
| 101      | Rahul    | IT         | 50000    |
| 102      | Priya    | HR         | 60000    |
| 103      | Amit     | IT         | 55000    |
+----------+----------+------------+----------+

Conceptually stored as:

ID:         101, 102, 103
Name:       Rahul, Priya, Amit
Department: IT, HR, IT
Salary:     50000, 60000, 55000
```

The second representation illustrates the logical column-oriented organization, not the exact binary layout of a Parquet or ORC file.

## 5. Row Storage vs Columnar Storage

| Feature | Row Storage | Columnar Storage |
|---|---|---|
| Organization | Row by row | Column by column |
| Data access | Complete records | Selected columns |
| Best suited for | Transactional workloads | Analytical workloads |
| Query pattern | Frequent point lookups | Aggregations and scans |
| Column selection | May read unnecessary columns | Reads required columns |
| Compression | Often less effective for mixed values | Often effective for similar column values |
| Individual updates | Generally efficient | Can be more expensive in file-based formats |
| Common examples | PostgreSQL, MySQL InnoDB | Parquet, ORC |

**Remember:**

- Row storage = Better suited to retrieving and modifying complete records.
- Columnar storage = Better suited to scanning selected columns and performing analytics.

## 6. Why Is Columnar Storage Faster for Analytics?

Consider a table containing:

- 100 columns.
- 10 million records.

A business analyst runs:

```sql
SELECT
    department,
    AVG(salary)
FROM employees
GROUP BY department;
```

Only two columns are required:

- `department`
- `salary`

### With Row Storage

The database may need to read records containing all 100 columns, even though the query uses only two.

The exact amount read depends on indexes, storage engine, caching, and execution plan.

### With Columnar Storage

The engine can read the two required columns rather than scanning all 100 columns.

This reduces unnecessary data reads.

### Benefits

1. Less data may need to be read from storage.
2. Column-level compression can reduce storage size.
3. Analytical scans can become more efficient.
4. Vectorized processing can improve execution efficiency in supported engines.
5. Column statistics can help skip irrelevant data.

## 7. Columnar Compression

Columnar storage often provides better compression because values in the same column tend to have similar types and patterns.

Consider a department column:

```text
IT, IT, IT, HR, HR, Finance, Finance
```

Because values repeat, compression techniques can represent the data efficiently.

### Common Compression Techniques

| Technique | Simple Explanation |
|---|---|
| Dictionary Encoding | Stores unique values in a dictionary and uses IDs |
| Run-Length Encoding (RLE) | Stores repeated consecutive values as a value and count |
| Bit Packing | Stores small integer values using fewer bits |
| Delta Encoding | Stores differences between consecutive values |
| Compression Codecs | Reduce the size of encoded data, such as Snappy or Zstandard |

### Example: Dictionary Encoding

Original values:

```text
IT, HR, IT, Finance, HR
```

Dictionary:

```text
0 = IT
1 = HR
2 = Finance
```

Encoded values:

```text
0, 1, 0, 2, 1
```

This can reduce storage requirements, particularly when a column contains many repeated values.

**Important:** The actual compression ratio depends on the data, encoding, and compression codec. Columnar storage does not guarantee that every dataset will compress better.

## 8. Row Storage and Columnar Storage in Real Systems

### Row-Oriented Database Systems

Commonly used for transactional applications.

Examples:

- MySQL InnoDB.
- PostgreSQL.
- SQL Server rowstore.

Typical use cases:

- Banking transactions.
- Customer account updates.
- Order creation.
- Payment processing.
- Inventory updates.

### Columnar Storage Systems and Formats

Commonly used for analytical workloads.

Examples:

- Apache Parquet.
- Apache ORC.
- Column-oriented analytical databases such as ClickHouse.
- Amazon Redshift's columnar storage architecture.

Typical use cases:

- Revenue analysis.
- Monthly sales reports.
- Large-scale aggregations.
- Historical data analysis.
- Business intelligence dashboards.

**Important:** A database system can support multiple storage structures. The classification above describes common storage approaches, not every feature of each product.

## 9. Apache Parquet

**Apache Parquet** is an open-source columnar file format designed for efficient storage and analytical processing.

It is widely used in big data platforms.

### Key Features

- Column-oriented storage.
- Column-level compression and encoding.
- Schema information stored in the file.
- Efficient column pruning.
- Support for nested data structures.
- Integration with Spark and other analytical engines.

### Example

Read a Parquet file using PySpark:

```python
df = spark.read.parquet("/data/employees/")

df.printSchema()
df.show()
```

Select only the required columns:

```python
df.select(
    "department",
    "salary"
).show()
```

Spark can use column pruning to avoid reading columns that are not required, when supported by the source and execution plan.

### Writing Parquet

```python
df.write \
    .mode("overwrite") \
    .parquet("/data/output/employees/")
```

Spark generally writes a directory containing one or more part files, rather than a single file.

## 10. Apache ORC

**Apache ORC (Optimized Row Columnar)** is another columnar file format designed for efficient analytical workloads.

### Key Features

- Column-oriented storage.
- Compression and encoding.
- Column statistics.
- Indexes and metadata that can help skip data.
- Support for nested data.
- Integration with big data processing systems.

### Example

Read ORC using PySpark:

```python
df = spark.read.orc("/data/employees/")
df.show()
```

Write ORC:

```python
df.write \
    .mode("overwrite") \
    .orc("/data/output/employees/")
```

## 11. Parquet vs ORC

Both are columnar file formats used in data engineering.

| Feature | Parquet | ORC |
|---|---|---|
| Storage layout | Columnar | Columnar |
| Compression | Supported | Supported |
| Schema information | Stored in file metadata | Stored in file metadata |
| Column pruning | Supported | Supported |
| Nested data | Supported | Supported |
| Ecosystem | Broad cross-platform support | Strong use in Hadoop/Hive ecosystems |
| Common use | Spark, lakehouses, analytics | Hive and big data analytics |

Neither format is universally better.

The choice depends on:
- Processing engine.
- Existing platform.
- Interoperability requirements.
- Workload.
- Performance testing.

## 12. Row Storage vs Columnar Storage in Data Engineering

In a typical data engineering architecture, both storage approaches can be used.

### Example Architecture

```text
       APPLICATION SYSTEMS
                |
                v
       RELATIONAL DATABASE
          (ROW STORAGE)
                |
                v
          DATA INGESTION
                |
                v
         DATA LAKE STORAGE
                |
                v
       PARQUET / ORC FILES
        (COLUMNAR STORAGE)
                |
                v
         SPARK / PYSPARK
                |
                v
        ANALYTICAL TABLES
                |
                v
         BI / REPORTING
```

### Why Use Both?

The application database handles transactional operations such as inserting and updating orders.

The data lake stores historical data in columnar formats such as Parquet for large-scale analytical processing.

This separates operational workloads from analytical workloads.

## 13. Important Concept: OLTP vs OLAP

Row and column storage are commonly associated with two types of workloads.

### OLTP – Online Transaction Processing

OLTP systems handle frequent business transactions.

Examples:
- Creating an order.
- Updating customer information.
- Processing payments.
- Updating inventory.

Typical characteristics:

- Many small transactions.
- Frequent inserts, updates, and deletes.
- Point lookups.
- Strong transactional consistency requirements.

Row-oriented storage is commonly suitable for these workloads.

### OLAP – Online Analytical Processing

OLAP systems analyze large volumes of historical data.

Examples:
- Monthly revenue analysis.
- Customer segmentation.
- Sales trends.
- Product performance reports.

Typical characteristics:

- Large data scans.
- Aggregations.
- Grouping and filtering.
- Reading selected columns.
- Complex analytical queries.

Columnar storage is commonly suitable for these workloads.

### Comparison

| Feature | OLTP | OLAP |
|---|---|---|
| Purpose | Run business transactions | Analyze business data |
| Data access | Individual records | Large datasets |
| Operations | INSERT, UPDATE, DELETE, SELECT | SELECT, GROUP BY, aggregations |
| Query type | Short and frequent | Complex analytical queries |
| Common storage | Row-oriented | Column-oriented |
| Example | Order management | Revenue reporting |

**Important:** OLTP and OLAP describe workload types, while row and column storage describe storage organization. They are related, but they are not interchangeable terms.

## 14. Important Performance Concepts

### 14.1 Column Pruning

Column pruning means reading only the columns required by a query.

Example:

```python
df.select("department", "salary")
```

If the source is Parquet, Spark can often read only these columns.

### 14.2 Predicate Pushdown

Predicate pushdown means applying supported filters as close to the data source as possible.

Example:

```python
df.filter(df.salary > 50000)
```

A compatible source may apply the filter during scanning rather than reading all data first.

Predicate pushdown is not guaranteed for every format, expression, or data source.

### 14.3 Data Skipping

Data skipping uses metadata and statistics to avoid reading files or data blocks that cannot satisfy a query.

For example, if a Parquet row group has a maximum salary of 40000, a query filtering for salaries above 50000 may be able to skip that row group.

Whether data can be skipped depends on available statistics and the filter condition.

### 14.4 Vectorized Reading

Vectorized reading processes batches of values instead of handling each value individually.

This can improve CPU efficiency and analytical performance in supported execution engines.

## 15. Important Interview Considerations

### 15.1 Is Parquet a Database?

No.

Parquet is a file format. It defines how data is organized and stored in files.

Spark, Databricks, and other compatible engines can read and process Parquet files.

### 15.2 Is Delta Lake a Columnar Format?

Delta Lake is a table format and storage framework built on Parquet data files and a transaction log.

It provides features such as ACID transactions, schema enforcement, and version history.

Parquet defines the columnar file layout; Delta Lake adds table-management and transaction capabilities.

### 15.3 Does Columnar Storage Always Perform Better?

No.

Columnar storage is generally better for analytical queries that scan selected columns across many records.

Row storage is generally better for transactional workloads that frequently access or modify complete records.

Performance depends on workload, indexes, data layout, engine, and query design.

### 15.4 Why Is Columnar Storage Good for Compression?

Values in the same column often have similar data types and repeated patterns. This makes techniques such as dictionary encoding and run-length encoding effective for many datasets.

### 15.5 Can We Update a Single Record in Parquet?

Parquet is an immutable file format rather than a transactional database table.

Changing a record generally requires rewriting the affected data or using a table format such as Delta Lake that manages changes through transactions and file replacement.

## 16. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Row storage is always faster | It is commonly better for point lookups and transactional operations |
| Columnar storage is always faster | It is commonly better for analytical scans |
| Parquet is a database | Parquet is a columnar file format |
| Delta Lake and Parquet are the same | Delta Lake manages tables using Parquet files and a transaction log |
| Columnar storage reads every column | It can read only required columns |
| Columnar storage always compresses better | Compression depends on data and encoding |
| OLTP means row storage | OLTP is a workload type commonly associated with row storage |
| OLAP means columnar storage | OLAP is a workload type commonly associated with columnar storage |
| CSV is a columnar format | CSV is a text-based tabular format |
| ORC and Parquet are databases | Both are columnar file formats |

## 17. Interview Questions and Answers

**Q1. What is row-based storage?**

Row-based storage organizes data record by record. All column values belonging to a row are stored together. It is commonly suitable for transactional workloads.

**Q2. What is columnar storage?**

Columnar storage organizes values column by column. It is commonly suitable for analytical queries that scan selected columns across large datasets.

**Q3. What is the main difference between row and column storage?**

Row storage keeps complete records together, while columnar storage keeps values from the same column together.

**Q4. Why is columnar storage preferred for analytics?**

Analytical queries often read only a few columns from large datasets. Columnar storage allows engines to read only the required columns, reducing unnecessary data reads.

**Q5. Why is row storage suitable for OLTP?**

OLTP applications frequently insert, update, or retrieve individual records. Row storage keeps complete records together, making these operations suitable for many transactional workloads.

**Q6. What are Parquet and ORC?**

Parquet and ORC are open-source columnar file formats designed for efficient analytical storage and processing.

**Q7. What is column pruning?**

Column pruning is an optimization that reads only the columns required by a query.

**Q8. What is predicate pushdown?**

Predicate pushdown moves supported filter conditions closer to the data source so that unnecessary data can be avoided during scanning.

**Q9. What is data skipping?**

Data skipping uses file or block statistics to avoid reading data that cannot match a query condition.

**Q10. What is dictionary encoding?**

Dictionary encoding stores unique values in a dictionary and represents repeated values using dictionary IDs.

**Q11. What is the difference between Parquet and Delta Lake?**

Parquet is a columnar file format. Delta Lake is a table format and storage framework that uses Parquet data files and a transaction log to provide features such as ACID transactions and version history.

**Q12. Can Parquet handle updates and deletes directly?**

Parquet itself does not provide transactional row-level update and delete operations. A table-management layer such as Delta Lake can manage updates and deletes by coordinating file changes.

**Q13. Does columnar storage always use less storage?**

No. It often compresses analytical data effectively, but the actual storage size depends on data characteristics, encoding, and compression settings.

**Q14. What is vectorized reading?**

Vectorized reading processes batches of values rather than processing each value individually, which can improve CPU efficiency.

**Q15. Can row and column storage be used in the same architecture?**

Yes. An operational database can use row-oriented storage, while a data lake stores historical data in columnar formats for analytics.

## 18. Quick Revision

| Concept | Remember |
|---|---|
| Row Storage | Stores complete records together |
| Columnar Storage | Stores values column by column |
| OLTP | Transactional workload |
| OLAP | Analytical workload |
| Parquet | Columnar file format |
| ORC | Columnar file format |
| Column Pruning | Reads only required columns |
| Predicate Pushdown | Applies supported filters near the source |
| Data Skipping | Avoids reading irrelevant data |
| Dictionary Encoding | Replaces repeated values with dictionary IDs |
| Run-Length Encoding | Stores repeated values with counts |
| Vectorized Reading | Processes batches of values |
| Delta Lake | Table framework using Parquet and a transaction log |

### One-Minute Interview Answer

Row-based and columnar storage are two ways of organizing data.

In row-based storage, all values belonging to a record are stored together. It is commonly suitable for OLTP workloads, where applications frequently insert, update, and retrieve individual records.

In columnar storage, values from the same column are stored together. It is commonly suitable for OLAP workloads because analytical queries often read only a few columns from large datasets.

For example, MySQL InnoDB commonly uses row-oriented storage, while Parquet and ORC are columnar file formats.

In Spark and Databricks, Parquet is widely used because it supports column pruning, compression, and efficient analytical processing. Delta Lake builds on Parquet by adding transaction and table-management capabilities.

## 19. Self-Assessment

- [ ] Explain row-based storage.
- [ ] Explain columnar storage.
- [ ] Draw both storage layouts.
- [ ] Compare row and column storage.
- [ ] Explain why columnar storage is useful for analytics.
- [ ] Explain why row storage is suitable for transactional workloads.
- [ ] Differentiate OLTP and OLAP.
- [ ] Explain Parquet and ORC.
- [ ] Explain column pruning.
- [ ] Explain predicate pushdown.
- [ ] Explain data skipping.
- [ ] Explain dictionary encoding.
- [ ] Explain vectorized reading.
- [ ] Differentiate Parquet and Delta Lake.
- [ ] Explain how row and column storage can coexist in one architecture.

**Interview readiness:** You should be able to explain both storage layouts, connect them to OLTP and OLAP workloads, and describe why Parquet is commonly used in Spark and modern data lake architectures.