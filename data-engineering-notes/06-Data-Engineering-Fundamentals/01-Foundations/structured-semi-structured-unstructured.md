# Structured vs Semi-Structured vs Unstructured Data

## 1. Introduction

In data engineering, data comes from different sources and in different formats.

Based on how data is organized, it is commonly classified into three categories:

1. **Structured Data**
2. **Semi-Structured Data**
3. **Unstructured Data**

Understanding these categories helps data engineers choose the right storage systems, ingestion methods, processing tools, and data models.

## 2. Structured Data

**Structured data** is data organized according to a predefined schema, usually in rows and columns.

Each column has a defined meaning and data type.

### Example

Consider an employee table:

| employee_id | name | department | salary |
|---:|---|---|---:|
| 101 | Rahul | IT | 50000 |
| 102 | Priya | HR | 60000 |
| 103 | Amit | Finance | 55000 |

Here:
- Each row represents an employee.
- Each column represents an attribute.
- Each column has a defined data type.
- Data follows a predefined table structure.

### Characteristics

- Predefined schema.
- Organized into rows and columns.
- Easy to query using SQL.
- Suitable for filtering, sorting, and aggregation.
- Usually stored in relational databases or structured files.

### Common Formats and Storage Systems

| Format / System | Description |
|---|---|
| MySQL | Relational database |
| PostgreSQL | Relational database |
| SQL Server | Relational database |
| CSV | Tabular text file |
| Relational tables | Rows and columns |

### Example SQL Query

```sql
SELECT
    department,
    AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

Structured data is generally straightforward to query because its schema is known.

## 3. Semi-Structured Data

**Semi-structured data** does not follow a rigid relational table structure, but it contains organizational information such as keys, values, tags, or metadata.

It may have a flexible or evolving schema.

### Example: JSON

```json
{
  "employee_id": 101,
  "name": "Rahul",
  "department": "IT",
  "skills": ["Python", "SQL", "Spark"],
  "address": {
    "city": "Kolkata",
    "country": "India"
  }
}
```

Here:
- `employee_id`, `name`, and `department` are fields.
- `skills` is an array.
- `address` is a nested object.
- Different records may contain different optional fields.

### Another Example: XML

```xml
<employee>
    <employee_id>101</employee_id>
    <name>Rahul</name>
    <department>IT</department>
</employee>
```

### Characteristics

- Flexible or evolving structure.
- Uses keys, values, tags, or metadata.
- Can contain nested objects and arrays.
- Different records may have different fields.
- Often requires parsing before relational analysis.

### Common Formats

| Format | Description |
|---|---|
| JSON | JavaScript Object Notation |
| XML | Extensible Markup Language |
| Avro | Schema-based serialization format |
| Parquet | Columnar storage format |
| ORC | Columnar storage format |

**Important:** Parquet and ORC are structured, schema-based columnar storage formats. They are not automatically semi-structured just because they are sometimes used in data lakes alongside JSON and XML.

### Example: Reading JSON Using PySpark

```python
df = spark.read.json("/data/employees.json")

df.printSchema()
df.show(truncate=False)
```

PySpark can infer a schema from JSON records or use an explicitly defined schema.

## 4. Unstructured Data

**Unstructured data** does not have a consistent predefined tabular schema.

It may contain text, images, audio, video, or other complex content.

### Examples

**Images**
- Employee photographs.
- Product images.
- Medical images.

**Videos**
- CCTV recordings.
- Training videos.
- Customer-uploaded videos.

**Audio**
- Voice recordings.
- Call-center conversations.
- Music recordings.

**Documents**
- PDF reports.
- Word documents.
- Scanned documents.

**Text**
- Emails.
- Social media posts.
- Customer feedback.

### Characteristics

- No fixed relational table structure.
- May contain complex content.
- Often requires specialized processing.
- Can be large in size.
- Frequently stored in object storage or file systems.

### Common Storage Systems

| Storage System | Common Use |
|---|---|
| Amazon S3 | Object storage |
| Azure Data Lake Storage Gen2 | Cloud data storage |
| Google Cloud Storage | Object storage |
| Hadoop HDFS | Distributed file storage |

### Example

Suppose a company stores customer support recordings.

```text
customer-recordings/
    call_001.mp3
    call_002.mp3
    call_003.mp3
```

These audio files are unstructured data.

A data pipeline may use speech-to-text processing to convert the recordings into text and then analyze the resulting content.

## 5. Main Differences

| Feature | Structured | Semi-Structured | Unstructured |
|---|---|---|---|
| Organization | Rows and columns | Keys, values, tags, nested fields | No consistent tabular structure |
| Schema | Predefined | Flexible or evolving | No fixed relational schema |
| Structure | Highly organized | Partially organized | Not consistently organized |
| Querying | SQL | SQL or specialized parsing | Often requires extraction or specialized tools |
| Examples | Tables, CSV | JSON, XML, Avro | Images, audio, video |
| Storage | Relational databases | Files, document databases, data lakes | Object storage, file systems |
| Processing | SQL, Spark | Parsing, SQL, Spark | OCR, NLP, image/audio processing |

## 6. Understanding with a Real-World Example

Consider an e-commerce company.

It collects different types of data from its customers.

### Structured Data

Order information:

| order_id | customer_id | amount |
|---:|---:|---:|
| 1001 | 501 | 1500 |
| 1002 | 502 | 2500 |

This data follows a predefined table structure.

### Semi-Structured Data

Customer activity event:

```json
{
  "customer_id": 501,
  "event": "product_view",
  "product_id": 901,
  "timestamp": "2026-10-31T10:30:00Z",
  "device": {
    "type": "mobile",
    "os": "Android"
  }
}
```

This data has fields and nested objects, but its structure can evolve.

### Unstructured Data

Product images and customer review recordings:

```text
product-images/
    phone.jpg
    laptop.jpg

customer-audio/
    review_001.mp3
    review_002.mp3
```

These files do not have a consistent relational table structure.

## 7. How Data Engineers Process These Data Types

A data engineer may receive all three categories from different source systems.

A common architecture looks like this:

```text
          DATA SOURCES
               |
     +---------+----------+
     |         |          |
     v         v          v
 Structured  Semi-     Unstructured
   Data     Structured     Data
     |         |          |
     v         v          v
 Relational  JSON/XML    Images/Audio
 Database    Files       Documents
     |         |          |
     +---------+----------+
               |
               v
           INGESTION
               |
               v
       DATA LAKE / STORAGE
               |
               v
       PROCESSING ENGINE
        (Spark / PySpark)
               |
               v
       CLEANED DATASETS
               |
               v
       DATA WAREHOUSE /
          LAKEHOUSE
```

The processing method depends on the data type.

| Data Type | Typical Processing |
|---|---|
| Structured | SQL transformations, joins, aggregations |
| Semi-Structured | JSON/XML parsing, schema handling, flattening |
| Unstructured | OCR, NLP, speech-to-text, image processing |

## 8. How PySpark Handles These Data Types

PySpark can process structured and semi-structured data directly through DataFrame APIs. Unstructured files can also be ingested and processed, but their content may require additional libraries or specialized services.

### 8.1 Structured Data: CSV

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

The CSV file contains tabular records.

### 8.2 Semi-Structured Data: JSON

```python
df = spark.read.json("/data/employees.json")

df.printSchema()
df.show(truncate=False)
```

JSON fields can be nested.

You can access nested fields using dot notation:

```python
from pyspark.sql.functions import col

df.select(
    col("name"),
    col("address.city")
).show()
```

### 8.3 Unstructured Data: Images

Spark can read image files as binary records using its image data source when supported by the environment.

```python
images_df = spark.read.format("image").load("/data/images/")
```

The resulting DataFrame contains image metadata and image data.

For tasks such as image classification or object detection, additional machine-learning libraries or services are generally needed.

## 9. Schema-on-Write vs Schema-on-Read

These concepts are closely related to how data is stored and processed.

### 9.1 Schema-on-Write

Schema-on-write means the structure is defined or enforced when data is written into the target system.

The data must conform to the expected schema before or during storage.

**Example:**

A relational database table has a predefined schema:

```sql
CREATE TABLE employees (
    employee_id INT,
    name VARCHAR(100),
    salary DECIMAL(10, 2)
);
```

Incoming data must follow the table's requirements.

**Common association:** Traditional relational databases and structured data systems.

### 9.2 Schema-on-Read

Schema-on-read means the structure is applied or interpreted when data is read or queried.

Data may be stored first, and its fields are interpreted later.

**Example:**

A company stores JSON files in a data lake.

When reading them with Spark, it can infer or apply a schema:

```python
df = spark.read.json("/data/employees/")
```

**Common association:** Data lakes and flexible data-processing systems.

### Comparison

| Feature | Schema-on-Write | Schema-on-Read |
|---|---|---|
| Schema application | During writing | During reading |
| Data storage | Usually conforms to defined schema | Can retain raw or flexible data |
| Flexibility | More controlled | More flexible |
| Validation | Often performed before or during writing | Can be performed during processing |
| Common example | Relational database | JSON files in a data lake |

**Important:** These are architectural approaches, not strict rules tied to one file format or technology. Modern systems can support both approaches.

## 10. Important Interview Concepts

### 10.1 Is JSON Structured or Semi-Structured?

JSON is generally classified as semi-structured because it uses key-value pairs and can contain nested objects and arrays without requiring every record to follow an identical relational table schema.

### 10.2 Is CSV Structured or Semi-Structured?

CSV is generally treated as structured or tabular data because it organizes records into rows and columns.

However, CSV does not inherently enforce data types or a complete schema. Those rules are usually supplied by the application or processing system.

### 10.3 Is Parquet Structured or Semi-Structured?

Parquet is a structured, schema-based columnar file format.

It stores data in a columnar layout and supports efficient analytical queries.

### 10.4 Can Unstructured Data Be Stored in a Data Lake?

Yes.

A data lake can store structured, semi-structured, and unstructured data in their original formats.

### 10.5 Can Unstructured Data Become Structured?

Yes.

For example, OCR can extract text from scanned invoices, and speech-to-text can convert audio recordings into text.

The extracted information can then be organized into structured tables.

## 11. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| JSON is structured data | JSON is generally semi-structured |
| CSV always enforces a schema | CSV is tabular, but does not inherently enforce data types |
| Parquet is semi-structured | Parquet is a schema-based columnar format |
| Unstructured data cannot be processed by Spark | Spark can ingest files, but specialized processing may be needed |
| Data lakes store only unstructured data | Data lakes can store all three categories |
| Semi-structured data has no organization | It has organizational elements such as keys, values, and tags |
| Unstructured data cannot become structured | Extraction and processing can produce structured datasets |
| Schema-on-read means no schema exists | A schema is applied or interpreted when data is read |

## 12. Interview Questions and Answers

**Q1. What are the three main types of data?**

Structured, semi-structured, and unstructured data.

**Q2. What is structured data?**

Structured data follows a predefined schema and is usually organized into rows and columns. Relational database tables are a common example.

**Q3. What is semi-structured data?**

Semi-structured data does not require a rigid relational table structure but contains organizational elements such as keys, values, tags, and nested fields. JSON and XML are common examples.

**Q4. What is unstructured data?**

Unstructured data does not follow a consistent predefined tabular schema. Images, audio, videos, and documents are examples.

**Q5. What is the difference between structured and semi-structured data?**

Structured data follows a predefined table schema. Semi-structured data has a more flexible organization and can contain nested fields or optional attributes.

**Q6. Is JSON structured or semi-structured?**

JSON is generally semi-structured because it uses key-value pairs and supports nested objects and arrays.

**Q7. Is Parquet structured or semi-structured?**

Parquet is a schema-based columnar storage format and is generally classified as structured data.

**Q8. What is schema-on-write?**

Schema-on-write means data is validated against or conforms to a defined schema during the write process.

**Q9. What is schema-on-read?**

Schema-on-read means the structure is interpreted or applied when data is read or queried.

**Q10. Can a data lake store all three data types?**

Yes. A data lake can store structured, semi-structured, and unstructured data.

**Q11. How does PySpark process JSON data?**

PySpark can read JSON using `spark.read.json()`. It can infer or apply a schema and access nested fields using DataFrame operations.

**Q12. How can unstructured data be converted into structured data?**

Specialized techniques such as OCR, NLP, and speech-to-text can extract information from documents, images, and audio. The extracted information can then be organized into tables.

**Q13. What is the difference between a data lake and a relational database in terms of data types?**

A data lake can store many data types and formats, including raw files. A relational database primarily organizes data into tables with defined schemas.

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Structured Data | Predefined schema, rows and columns |
| Semi-Structured Data | Flexible structure, keys and values |
| Unstructured Data | No consistent tabular structure |
| CSV | Tabular data |
| JSON | Semi-structured data |
| XML | Semi-structured data |
| Parquet | Structured, columnar format |
| ORC | Structured, columnar format |
| Images | Unstructured data |
| Audio | Unstructured data |
| Videos | Unstructured data |
| Schema-on-Write | Schema applied during writing |
| Schema-on-Read | Schema applied or interpreted during reading |
| Data Lake | Can store all three categories |

### One-Minute Interview Answer

Data is commonly classified into three types: structured, semi-structured, and unstructured.

Structured data follows a predefined schema and is organized into rows and columns. Relational database tables and CSV files are common examples.

Semi-structured data has a flexible structure and uses elements such as keys, values, and nested fields. JSON and XML are common examples.

Unstructured data does not follow a consistent tabular structure. Images, audio, videos, and documents are examples.

As a data engineer, I may ingest these different data types into a data lake using tools such as Azure Data Factory or Auto Loader. I can process structured and semi-structured data using PySpark, while unstructured data may require additional tools such as OCR, NLP, or speech-to-text.

## 14. Self-Assessment

- [ ] Define structured data.
- [ ] Define semi-structured data.
- [ ] Define unstructured data.
- [ ] Explain the difference between all three.
- [ ] Identify CSV, JSON, XML, and Parquet correctly.
- [ ] Explain how a data lake stores different data types.
- [ ] Explain how PySpark reads CSV and JSON.
- [ ] Explain how nested JSON fields are accessed.
- [ ] Explain how unstructured data can be processed.
- [ ] Explain schema-on-write.
- [ ] Explain schema-on-read.
- [ ] Give a real-world e-commerce example.

**Interview readiness:** You should be able to classify common file formats, explain the three data categories with examples, and describe how a data engineer ingests and processes each type.