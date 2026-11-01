# CSV vs JSON vs Parquet vs Avro

## 1. Introduction

In data engineering, data is stored and transferred using different file formats.

Four commonly used formats are:

1. **CSV** – Comma-Separated Values
2. **JSON** – JavaScript Object Notation
3. **Parquet** – Columnar storage format
4. **Avro** – Row-oriented, schema-based serialization format

Each format has different characteristics, storage layouts, and use cases.

Choosing the right format affects:
- Storage size.
- Read and write performance.
- Data compression.
- Schema management.
- Data exchange between systems.
- Analytical query performance.

## 2. CSV (Comma-Separated Values)

**CSV** is a text-based format that stores tabular data using rows and delimiters.

Each line generally represents one record, and fields are separated by commas or another configured delimiter.

### Example

```csv
employee_id,name,department,salary
101,Rahul,IT,50000
102,Priya,HR,60000
103,Amit,Finance,55000
```

### Characteristics

- Human-readable.
- Simple tabular structure.
- Easy to create and exchange.
- No built-in schema or data type enforcement.
- Does not inherently support nested objects.
- Usually larger than compressed binary formats.
- Can be processed by almost every data platform.

### Advantages

- Simple and widely supported.
- Easy to inspect using a text editor.
- Useful for exchanging small datasets.
- Compatible with spreadsheets and databases.

### Disadvantages

- No built-in schema enforcement.
- Data types must be interpreted separately.
- Nested structures are not naturally supported.
- Repeated column names and values increase storage.
- Large files can be expensive to scan.

### Reading CSV Using PySpark

```python
df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("/data/employees.csv")
)

df.show()
df.printSchema()
```

### Writing CSV Using PySpark

```python
df.write \
    .mode("overwrite") \
    .option("header", True) \
    .csv("/data/output/employees/")
```

**Important:** Spark generally writes a directory containing one or more part files, not necessarily a single CSV file.

### Common Use Cases

- Data exchange.
- Small datasets.
- Simple reports.
- Input files from business applications.
- Exporting data for spreadsheets.

## 3. JSON (JavaScript Object Notation)

**JSON** is a text-based format that stores data using key-value pairs.

It supports nested objects and arrays, making it useful for representing flexible and hierarchical data.

### Example

```json
{
  "employee_id": 101,
  "name": "Rahul",
  "department": "IT",
  "salary": 50000,
  "skills": ["Python", "SQL", "Spark"],
  "address": {
    "city": "Kolkata",
    "country": "India"
  }
}
```

### Characteristics

- Human-readable text format.
- Supports key-value pairs.
- Supports nested objects.
- Supports arrays.
- Flexible structure.
- Commonly used in REST APIs.
- Can contain optional or evolving fields.

### Advantages

- Supports complex and nested data.
- Easy to exchange between applications.
- Widely supported by programming languages.
- Useful for API communication.
- Flexible when data structures evolve.

### Disadvantages

- Repeated field names increase file size.
- Parsing can require additional processing.
- Text representation may use more storage than binary formats.
- Schema is not inherently enforced by the JSON format.

### Reading JSON Using PySpark

```python
df = spark.read.json("/data/employees.json")

df.show(truncate=False)
df.printSchema()
```

### Accessing Nested Fields

```python
from pyspark.sql.functions import col

df.select(
    col("employee_id"),
    col("name"),
    col("address.city")
).show()
```

### Writing JSON Using PySpark

```python
df.write \
    .mode("overwrite") \
    .json("/data/output/employees/")
```

### Common Use Cases

- REST API responses.
- Application logs.
- Event data.
- Nested records.
- Data exchange between services.

## 4. Parquet

**Apache Parquet** is an open-source, columnar storage file format designed for efficient analytical processing.

Unlike CSV and JSON, Parquet stores data in a binary, column-oriented representation.

### Example Dataset

| employee_id | name | department | salary |
|---:|---|---|---:|
| 101 | Rahul | IT | 50000 |
| 102 | Priya | HR | 60000 |
| 103 | Amit | Finance | 55000 |

Conceptually, Parquet organizes the values by column:

```text
employee_id: 101, 102, 103

name: Rahul, Priya, Amit

department: IT, HR, Finance

salary: 50000, 60000, 55000
```

This is a simplified representation, not the exact binary structure of a Parquet file.

### Characteristics

- Columnar storage.
- Binary file format.
- Stores schema information.
- Supports compression and encoding.
- Supports nested data structures.
- Supports column pruning.
- Suitable for analytical workloads.

### Advantages

- Efficient analytical queries.
- Reads only required columns when supported.
- Often provides good compression.
- Supports schema information and nested data.
- Widely supported by Spark and other data platforms.

### Disadvantages

- Not human-readable in a normal text editor.
- Less suitable for frequent individual record updates.
- Requires compatible processing tools.
- Not ideal for simple manual editing.

### Reading Parquet Using PySpark

```python
df = spark.read.parquet("/data/employees.parquet")

df.show()
df.printSchema()
```

### Writing Parquet Using PySpark

```python
df.write \
    .mode("overwrite") \
    .parquet("/data/output/employees/")
```

### Reading Only Required Columns

```python
df = (
    spark.read
    .parquet("/data/employees/")
    .select("department", "salary")
)
```

Spark can use column pruning to avoid reading unnecessary columns.

### Common Use Cases

- Data lakes.
- Data warehouses and lakehouses.
- Databricks.
- Apache Spark analytics.
- Large-scale batch processing.
- Historical analytical datasets.

## 5. Avro

**Apache Avro** is a schema-based data serialization format commonly used for data exchange and event-driven systems.

It stores records in a compact binary representation and uses a schema to describe the structure and data types.

### Example Schema

```json
{
  "type": "record",
  "name": "Employee",
  "fields": [
    {
      "name": "employee_id",
      "type": "int"
    },
    {
      "name": "name",
      "type": "string"
    },
    {
      "name": "department",
      "type": "string"
    },
    {
      "name": "salary",
      "type": "double"
    }
  ]
}
```

This schema defines the structure of an employee record.

An Avro data file stores records according to this schema.

### Characteristics

- Schema-based serialization.
- Binary data representation.
- Row-oriented record storage.
- Supports schema evolution.
- Supports compact data representation.
- Commonly used in event streaming and data exchange.

### Advantages

- Compact binary representation.
- Supports schema evolution.
- Useful for exchanging records between systems.
- Suitable for event-driven architectures.
- Supports compatibility rules when schemas evolve.

### Disadvantages

- Not directly human-readable.
- Requires schema-aware tools.
- Schema compatibility must be managed.
- Less optimized than columnar formats for queries that scan only a few columns across large datasets.

### Reading Avro Using PySpark

Avro support may require the appropriate Spark Avro module or connector, depending on the Spark environment.

```python
df = spark.read.format("avro").load("/data/employees.avro")

df.show()
df.printSchema()
```

### Writing Avro Using PySpark

```python
df.write \
    .format("avro") \
    .mode("overwrite") \
    .save("/data/output/employees_avro/")
```

### Common Use Cases

- Apache Kafka event serialization.
- Data exchange between services.
- Event-driven architectures.
- Streaming pipelines.
- Schema-managed data communication.

## 6. Main Differences: CSV vs JSON vs Parquet vs Avro

| Feature | CSV | JSON | Parquet | Avro |
|---|---|---|---|---|
| Format | Text | Text | Binary | Binary |
| Storage layout | Tabular rows | Nested records | Columnar | Row-oriented records |
| Schema | Not built in | Not enforced by format | Stored in file metadata | Defined by schema |
| Human-readable | Yes | Yes | No | No |
| Nested data | Not naturally supported | Supported | Supported | Supported |
| Compression | External compression | External compression | Built-in codec support | Supports compression |
| Schema evolution | External handling | Flexible structure | Supported through compatible readers/writers | Explicit schema evolution support |
| Best suited for | Simple data exchange | APIs and nested data | Analytical queries | Serialization and event exchange |

## 7. Row-Based vs Columnar Formats

This distinction is especially important for interviews.

### Row-Oriented Formats

- CSV stores records line by line.
- Avro stores records in a row-oriented binary representation.

They are commonly useful when records are exchanged or processed as complete units.

### Columnar Formats

- Parquet stores data by column.
- ORC also uses columnar storage.

They are commonly useful for analytical queries that scan selected columns across large datasets.

**Remember:**

- Avro = Row-oriented binary serialization.
- Parquet = Column-oriented analytical storage.

CSV and JSON are text-based formats; their physical organization should not be treated as identical to a database's row-store implementation.

## 8. When Should We Use Each Format?

| Requirement | Suitable Format |
|---|---|
| Simple tabular data exchange | CSV |
| REST API communication | JSON |
| Nested application data | JSON |
| Large-scale analytical processing | Parquet |
| Data lake analytics | Parquet |
| Event serialization | Avro |
| Kafka record serialization | Avro or other supported formats |
| Schema-managed record exchange | Avro |
| Easy manual inspection | CSV or JSON |

### Practical Understanding

**Use CSV when:**
- The dataset is simple and tabular.
- Human readability matters.
- Data needs to be exchanged with spreadsheets.

**Use JSON when:**
- Data contains nested structures.
- Applications communicate through APIs.
- Records have flexible attributes.

**Use Parquet when:**
- Processing large datasets.
- Running analytical queries.
- Reducing storage and I/O through columnar storage.

**Use Avro when:**
- Exchanging serialized records.
- Building event-driven pipelines.
- Managing schemas and compatibility between producers and consumers.

No format is universally best. The choice depends on how data is produced, stored, transferred, and queried.

## 9. Schema Evolution

**Schema evolution** means changing a dataset's schema over time while maintaining compatibility with existing data or applications.

For example, the original employee schema is:

```json
{
  "employee_id": 101,
  "name": "Rahul"
}
```

Later, the company adds a department field:

```json
{
  "employee_id": 101,
  "name": "Rahul",
  "department": "IT"
}
```

The system must handle the new field correctly.

### Schema Evolution by Format

| Format | Schema Handling |
|---|---|
| CSV | Schema is usually maintained externally |
| JSON | Flexible structure; readers must handle missing or additional fields |
| Parquet | Stores schema metadata; compatibility depends on readers and writers |
| Avro | Explicit schema evolution and compatibility rules |

### Avro Schema Evolution

Avro supports compatibility rules such as:

- Backward compatibility.
- Forward compatibility.
- Full compatibility.

These rules describe whether producers and consumers using different schema versions can work together.

For example, adding a field with a suitable default value can allow older records to be read using a newer schema.

**Important:** Schema evolution is not automatic in every situation. Compatibility depends on the change, schema rules, and processing system.

## 10. Compression Support

Compression reduces the storage size of data.

Different formats support compression in different ways.

| Format | Compression Approach |
|---|---|
| CSV | Often compressed externally, such as Gzip |
| JSON | Often compressed externally, such as Gzip |
| Parquet | Supports codecs such as Snappy, Gzip, and ZSTD |
| Avro | Supports codecs such as Deflate, Snappy, and Zstandard, depending on implementation |

### Example: Parquet with Snappy

```python
df.write \
    .option("compression", "snappy") \
    .parquet("/data/output/")
```

### Example: CSV with Gzip

```python
df.write \
    .option("compression", "gzip") \
    .option("header", True) \
    .csv("/data/output/")
```

**Important:** Compression support and available codecs depend on the format implementation and runtime.

## 11. File Format Selection in a Real Data Engineering Pipeline

Consider an e-commerce company that collects customer orders from different systems.

A possible architecture:

```text
         SOURCE SYSTEMS
                |
        +-------+-------+
        |       |       |
        v       v       v
       CSV    JSON    KAFKA
        |       |       |
        |       |       v
        |       |      AVRO
        |       |       |
        +-------+-------+
                |
                v
          DATA INGESTION
                |
                v
          RAW DATA STORAGE
                |
                v
         DATABRICKS / SPARK
                |
                v
         TRANSFORMATION
                |
                v
          PARQUET / DELTA
                |
                v
          ANALYTICAL TABLES
                |
                v
            POWER BI
```

### Example Workflow

1. A business application exports customer information as CSV.
2. A REST API provides product information as JSON.
3. Kafka receives order events serialized using Avro.
4. Data is ingested into cloud storage.
5. Databricks processes the incoming records using PySpark.
6. Cleaned analytical data is stored as Delta tables using Parquet data files.
7. Power BI uses the processed datasets for reporting.

This is a common pattern, although actual architectures vary.

## 12. Important Concept: Avro vs Parquet

This is a frequently asked data engineering interview question.

| Feature | Avro | Parquet |
|---|---|---|
| Storage layout | Row-oriented | Column-oriented |
| Main purpose | Serialization and data exchange | Analytical storage |
| Schema | Explicit schema | Schema stored in file metadata |
| Best use | Streaming and record exchange | Analytics and large-scale scans |
| Reading records | Efficient for record-oriented access | Optimized for column-oriented access |
| Common integration | Kafka and event systems | Spark and data lakes |

### Simple Example

Suppose an application generates millions of order events.

Avro can serialize each event into a compact record for transmission through Kafka.

Later, a data engineering pipeline can process those events and write the results into Parquet or Delta tables for analytical queries.

**Interview answer:**

Avro is generally preferred for schema-based record serialization and event exchange, while Parquet is generally preferred for columnar analytical storage.

## 13. Important Concept: CSV vs JSON

| Feature | CSV | JSON |
|---|---|---|
| Structure | Tabular | Key-value and nested |
| Data representation | Delimited values | Objects and arrays |
| Nested data | Not naturally supported | Supported |
| Human-readable | Yes | Yes |
| Common use | Reports and simple datasets | APIs and application data |

### Example

CSV:

```csv
employee_id,name,department
101,Rahul,IT
102,Priya,HR
```

JSON:

```json
[
  {
    "employee_id": 101,
    "name": "Rahul",
    "department": "IT"
  },
  {
    "employee_id": 102,
    "name": "Priya",
    "department": "HR"
  }
]
```

CSV is simpler for tabular data, while JSON is more flexible for nested and hierarchical data.

## 14. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Parquet is a database | Parquet is a columnar file format |
| Avro is a columnar format | Avro is a row-oriented serialization format |
| JSON has a fixed schema | JSON does not inherently enforce a schema |
| CSV stores data types | CSV does not inherently enforce data types |
| Parquet is human-readable | Parquet is a binary format |
| Avro and Parquet serve exactly the same purpose | Avro is commonly used for serialization; Parquet for analytics |
| JSON is always larger than CSV | Size depends on data structure and compression |
| Parquet cannot store nested data | Parquet supports nested structures |
| Avro cannot evolve schemas | Avro explicitly supports schema evolution |
| Every format supports every codec | Codec support depends on implementation and runtime |
| Delta Lake and Parquet are identical | Delta Lake adds transaction and table-management features over Parquet data files |

## 15. Interview Questions and Answers

**Q1. What are CSV, JSON, Parquet, and Avro?**

They are data formats used to store or exchange data. CSV and JSON are text-based formats, while Parquet and Avro are binary formats with different storage layouts and purposes.

**Q2. What is the difference between CSV and JSON?**

CSV stores tabular data using delimiters. JSON stores key-value pairs and supports nested objects and arrays.

**Q3. What is Parquet?**

Parquet is an open-source columnar file format designed for efficient analytical processing. It supports compression, encoding, and column pruning.

**Q4. What is Avro?**

Avro is a schema-based binary serialization format that stores records in a row-oriented representation. It is commonly used for data exchange and event streaming.

**Q5. What is the difference between Avro and Parquet?**

Avro is generally used for serializing and exchanging complete records, while Parquet is designed for analytical queries that read selected columns.

**Q6. Why is Parquet preferred in Spark?**

Parquet supports column pruning, compression, and efficient analytical reads, making it suitable for large-scale Spark workloads.

**Q7. Why is Avro used with Kafka?**

Avro provides compact binary serialization and schema-based record exchange. It can help producers and consumers communicate using managed schemas.

**Q8. Does CSV support schema evolution?**

CSV does not have a built-in schema management mechanism. Schema changes must be handled by the systems that produce and consume the files.

**Q9. Does JSON support schema evolution?**

JSON has a flexible structure, so producers can add or omit fields. However, consumers still need to handle those changes correctly.

**Q10. What is schema evolution?**

Schema evolution is the process of changing a dataset's structure over time while maintaining compatibility with existing data and applications.

**Q11. What is the difference between row-based and columnar formats?**

Row-based formats organize complete records together, while columnar formats organize values by column. Avro is row-oriented, and Parquet is column-oriented.

**Q12. Which format would you choose for large-scale analytics?**

I would generally choose Parquet because it supports columnar storage, compression, and efficient analytical queries.

**Q13. Which format would you choose for API data?**

JSON is commonly used because it is human-readable and supports nested structures.

**Q14. Can Spark read all four formats?**

Spark supports reading and writing CSV, JSON, Parquet, and Avro. Avro support may require the appropriate module or connector in the runtime.

**Q15. What is the difference between Parquet and Delta Lake?**

Parquet is a columnar file format. Delta Lake is a table format and storage framework that uses Parquet data files and a transaction log to provide features such as ACID transactions, schema enforcement, and version history.

**Q16. Which format is best for streaming pipelines?**

Avro is a common choice for schema-based event serialization, but JSON, Protobuf, and other formats are also used. The choice depends on the producer, consumer, schema management, and compatibility requirements.

## 16. Quick Revision

| Format | Type | Storage Layout | Main Use |
|---|---|---|---|
| CSV | Text | Delimited records | Simple tabular exchange |
| JSON | Text | Objects and arrays | APIs and nested data |
| Parquet | Binary | Columnar | Analytics and data lakes |
| Avro | Binary | Row-oriented records | Serialization and event exchange |

### One-Minute Interview Answer

CSV, JSON, Parquet, and Avro are common data formats used in data engineering.

CSV is a simple text-based format used for tabular data exchange. JSON is a text-based format that supports key-value pairs, nested objects, and arrays, making it common in APIs.

Parquet is a binary columnar format designed for analytical workloads. It supports compression and column pruning, so it is widely used with Spark and data lakes.

Avro is a binary, schema-based, row-oriented serialization format commonly used for event streaming and data exchange, including Kafka pipelines.

In a real data engineering architecture, I might ingest CSV and JSON from source systems, receive Avro-serialized events from Kafka, and store processed analytical datasets in Parquet or Delta tables.

## 17. Self-Assessment

- [ ] Define CSV.
- [ ] Define JSON.
- [ ] Define Parquet.
- [ ] Define Avro.
- [ ] Differentiate all four formats.
- [ ] Explain row-oriented vs columnar storage.
- [ ] Explain why Parquet is useful for analytics.
- [ ] Explain why Avro is useful for event serialization.
- [ ] Explain schema evolution.
- [ ] Explain compression support.
- [ ] Read CSV using PySpark.
- [ ] Read JSON using PySpark.
- [ ] Read Parquet using PySpark.
- [ ] Read Avro using PySpark.
- [ ] Explain Parquet vs Delta Lake.
- [ ] Choose an appropriate format for a real pipeline.

**Interview readiness:** You should be able to explain the purpose, structure, advantages, and limitations of all four formats, and justify why you would choose Avro for record serialization or Parquet for analytical storage.