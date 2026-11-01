

# Schema and Data Types in PySpark

## 1. What is a Schema?

A schema defines the structure of a DataFrame.

It specifies:
- Column names
- Data types
- Whether a column can contain `NULL` values (`nullable`)

Example:

| Column | Data Type | Nullable |
|---|---|---|
| customer_id | Integer | No |
| name | String | Yes |
| age | Integer | Yes |
| salary | Double | Yes |

A schema helps Spark understand how data should be interpreted and processed.

**Interview definition:**

A schema is the structure of a DataFrame that defines column names, data types, and nullability.

---

## 2. Why is Schema Important?

Using a well-defined schema helps to:

1. Avoid incorrect data type inference.
2. Detect unexpected or invalid data.
3. Improve data consistency.
4. Prevent unwanted type conversions.
5. Make ETL pipelines more reliable.
6. Handle large datasets more efficiently by avoiding unnecessary schema inference.

**Practical example:**

Suppose a CSV file contains:

```csv
customer_id,name,age,salary
101,Rahul,25,45000.50
102,Amit,30,55000.00
```

Without an explicit schema, Spark may need to inspect the data to infer column types.

With an explicit schema, we tell Spark exactly how to read each column.

---

## 3. Common PySpark Data Types

PySpark provides data types through `pyspark.sql.types`.

| Data Type | Purpose | Example |
|---|---|---|
| `StringType()` | Text | `"Rahul"` |
| `IntegerType()` | Whole numbers | `25` |
| `LongType()` | Large whole numbers | `9876543210` |
| `FloatType()` | Floating-point numbers | `25.5` |
| `DoubleType()` | Double-precision numbers | `45000.75` |
| `DecimalType(p, s)` | Exact decimal values | `DecimalType(10, 2)` |
| `BooleanType()` | True or False | `True` |
| `DateType()` | Date without time | `2026-10-30` |
| `TimestampType()` | Date and time | `2026-10-30 10:30:00` |
| `ArrayType()` | Collection of values | `[10, 20, 30]` |
| `MapType()` | Key-value pairs | `{"city": "Kolkata"}` |
| `StructType()` | Nested structure | Customer details |
| `BinaryType()` | Binary data | Byte values |

### Important differences

**IntegerType vs LongType**
- `IntegerType`: 32-bit signed integer.
- `LongType`: 64-bit signed integer.
- Use `LongType` when values may exceed the IntegerType range.

**FloatType vs DoubleType**
- `FloatType`: 32-bit floating-point number.
- `DoubleType`: 64-bit floating-point number.
- `DoubleType` provides greater precision.

**DoubleType vs DecimalType**
- `DoubleType` is suitable for approximate numerical calculations.
- `DecimalType` is suitable for exact decimal calculations, such as financial amounts.

For financial data, prefer `DecimalType` when exact precision is required.

---

## 4. Creating an Explicit Schema

Use `StructType` and `StructField` to define a schema.

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    StringType,
    DoubleType
)

schema = StructType([
    StructField("customer_id", IntegerType(), False),
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True),
    StructField("salary", DoubleType(), True)
])
```

### Understanding the syntax

```python
StructField("column_name", DataType(), nullable)
```

- `column_name`: Name of the column.
- `DataType()`: Data type of the column.
- `nullable`: Whether the column can contain NULL values.

Example:

```python
StructField("customer_id", IntegerType(), False)
```

This defines `customer_id` as an integer column that is marked as non-nullable.

**Important:** Declaring a field as non-nullable does not automatically guarantee that every incoming data source will reject or clean invalid NULL values. Validate the actual data and source behavior.

---

## 5. Reading Data with an Explicit Schema

```python
df = (
    spark.read
    .option("header", True)
    .schema(schema)
    .csv("/data/customers.csv")
)
```

Here:
- `header=True`: Uses the first row as column names.
- `.schema(schema)`: Applies the predefined schema.
- `.csv()`: Reads the CSV file.

Check the schema:

```python
df.printSchema()
```

Possible output:

```text
root
 |-- customer_id: integer (nullable = false)
 |-- name: string (nullable = true)
 |-- age: integer (nullable = true)
 |-- salary: double (nullable = true)
```

Check the data:

```python
df.show()
```

---

## 6. Inferring Schema Automatically

Spark can automatically infer data types from the source data.

```python
df = (
    spark.read
    .option("header", True)
    .option("inferSchema", True)
    .csv("/data/customers.csv")
)
```

Here, Spark examines the data and attempts to identify suitable column types.

### Explicit Schema vs Schema Inference

| Explicit Schema | Schema Inference |
|---|---|
| Defined by the developer | Identified by Spark |
| More predictable | Depends on source data |
| Avoids inference work | Requires examining data |
| Useful for production pipelines | Useful for exploration |
| Gives control over data types | May infer unexpected types |

**Interview point:** For production ETL pipelines, an explicit schema is generally preferred because it makes data types predictable and helps detect unexpected input.

---

## 7. Checking DataFrame Schema and Data Types

### Print complete schema

```python
df.printSchema()
```

### Get schema as a StructType object

```python
df.schema
```

### Get column names

```python
df.columns
```

### Get data types

```python
df.dtypes
```

Example output:

```python
[
    ("customer_id", "int"),
    ("name", "string"),
    ("age", "int"),
    ("salary", "double")
]
```

### Get a specific column's data type

```python
df.schema["salary"].dataType
```

### Check whether a column exists

```python
"salary" in df.columns
```

---

## 8. Changing Data Types Using cast()

Sometimes source data contains incorrect data types.

Example: An age column is stored as a string instead of an integer.

Use `cast()` to convert it.

```python
from pyspark.sql.functions import col
from pyspark.sql.types import IntegerType

df = df.withColumn(
    "age",
    col("age").cast(IntegerType())
)
```

You can also use:

```python
df = df.withColumn(
    "age",
    col("age").cast("int")
)
```

### Casting multiple columns

```python
df = (
    df
    .withColumn("age", col("age").cast("int"))
    .withColumn("salary", col("salary").cast("double"))
)
```

### What happens when casting fails?

For example, converting `"ABC"` into an integer cannot produce a valid integer.

Depending on the Spark version and ANSI mode, an invalid cast may return `NULL` or raise an exception.

For safer handling, validate and clean the source values before conversion.

---

## 9. Nested Data Types

PySpark supports complex data types such as arrays, maps, and structs.

### A. ArrayType

Stores multiple values in a single column.

```python
from pyspark.sql.types import ArrayType, StringType

schema = StructType([
    StructField("customer_id", IntegerType(), False),
    StructField("skills", ArrayType(StringType()), True)
])
```

Example data:

```text
101, ["Python", "SQL", "Spark"]
```

Access an array element:

```python
from pyspark.sql.functions import col

df.select(col("skills")[0]).show()
```

Array indexing starts at zero.

### B. MapType

Stores key-value pairs.

```python
from pyspark.sql.types import MapType, StringType

schema = StructType([
    StructField(
        "customer_id",
        IntegerType(),
        False
    ),
    StructField(
        "details",
        MapType(StringType(), StringType()),
        True
    )
])
```

Example:

```text
{
    "city": "Kolkata",
    "country": "India"
}
```

Access a map value:

```python
df.select(col("details")["city"]).show()
```

### C. StructType

Stores nested fields inside a column.

```python
from pyspark.sql.types import StructType, StructField, StringType

address_schema = StructType([
    StructField("city", StringType(), True),
    StructField("state", StringType(), True)
])

schema = StructType([
    StructField("customer_id", IntegerType(), False),
    StructField("address", address_schema, True)
])
```

Access a nested field:

```python
df.select("address.city").show()
```

**Remember:**
- `ArrayType`: Multiple values.
- `MapType`: Key-value pairs.
- `StructType`: Multiple named fields in a nested structure.

---

## 10. Schema Evolution in Delta Lake

Schema evolution means allowing a table's schema to change when new data arrives.

Example:

Existing Delta table:

| customer_id | name |
|---|---|
| 101 | Rahul |
| 102 | Amit |

New incoming data:

| customer_id | name | city |
|---|---|---|
| 103 | Priya | Kolkata |

The incoming data contains a new column called `city`.

Delta Lake can support adding this column through schema evolution.

Example:

```python
(
    new_df.write
    .format("delta")
    .mode("append")
    .option("mergeSchema", "true")
    .save("/data/customers_delta")
)
```

The `mergeSchema` option allows compatible new columns to be added to the existing table schema during the write.

**Important:**
- Schema evolution should be controlled.
- Unexpected columns should not automatically be accepted without validation.
- Schema evolution does not mean every incompatible data type change is automatically supported.

---

## 11. Schema Validation in ETL Pipelines

In real-world data engineering, incoming data should be checked before processing.

Example:

```python
required_columns = [
    "customer_id",
    "name",
    "age",
    "salary"
]

missing_columns = [
    column
    for column in required_columns
    if column not in df.columns
]

if missing_columns:
    raise ValueError(
        f"Missing columns: {missing_columns}"
    )
```

This checks whether all required columns exist.

### Validate data types

```python
expected_types = {
    "customer_id": "int",
    "name": "string",
    "age": "int",
    "salary": "double"
}

actual_types = dict(df.dtypes)

for column, expected_type in expected_types.items():
    if actual_types.get(column) != expected_type:
        print(f"Type mismatch: {column}")
```

In a production pipeline, you can reject invalid data, send it to a quarantine location, or apply a controlled conversion depending on business requirements.

---

## 12. Common Mistakes

1. **Depending entirely on schema inference**

   Inferred types may not match the expected business data types.

2. **Using DoubleType for every numerical column**

   Financial values may require DecimalType for exact precision.

3. **Ignoring NULL values**

   A column's data type does not guarantee that its values are valid.

4. **Casting without validation**

   Invalid values may become NULL or cause exceptions.

5. **Confusing StructType and StructField**

   `StructType` defines the complete schema, while `StructField` defines one column.

6. **Assuming schema evolution fixes all schema mismatches**

   Adding compatible columns is different from handling incompatible type changes.

7. **Assuming nullable=False cleans incoming data**

   Nullability is schema metadata, not a data-cleaning operation.

---

## 13. Common Interview Questions

**Q1. What is a schema in PySpark?**

A schema defines the structure of a DataFrame, including column names, data types, and nullability.

**Q2. What is the difference between StructType and StructField?**

`StructType` represents the complete DataFrame schema. `StructField` represents an individual column.

**Q3. How do you define an explicit schema?**

Using `StructType` and `StructField` from `pyspark.sql.types`.

**Q4. What is schema inference?**

Schema inference is Spark's process of automatically identifying column data types from source data.

**Q5. Why use an explicit schema in production?**

It provides predictable data types, reduces inference overhead, and makes the pipeline easier to validate.

**Q6. How do you change a column's data type?**

Using `cast()`.

```python
df.withColumn("age", col("age").cast("int"))
```

**Q7. What is the difference between IntegerType and LongType?**

IntegerType uses 32-bit integers, while LongType uses 64-bit integers.

**Q8. What is the difference between ArrayType and MapType?**

ArrayType stores a collection of values. MapType stores key-value pairs.

**Q9. What is schema evolution in Delta Lake?**

It allows supported schema changes, such as adding new columns, when writing data to an existing Delta table.

**Q10. How do you check a DataFrame's schema?**

Using:

```python
df.printSchema()
```

**Q11. What happens when you cast an invalid string to an integer?**

It may return NULL or raise an exception depending on Spark's ANSI mode and version.

**Q12. What is the difference between nullable=True and nullable=False?**

They indicate whether a field is allowed to contain NULL values according to its schema definition.

---

## 14. Practice Questions

Try these without looking at the answers.

1. Create a schema for an employee DataFrame with employee_id, name, department, salary, and joining_date.
2. Read a CSV file using an explicit schema.
3. Print the schema and data types of a DataFrame.
4. Convert a salary column from string to double.
5. Create a DataFrame schema containing an ArrayType column.
6. Create a nested StructType schema for employee address details.
7. Check whether a required column exists.
8. Write new columns into an existing Delta table using schema evolution.
9. Explain why explicit schemas are preferred in production ETL pipelines.
10. Explain the difference between DoubleType and DecimalType.

---

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Schema | Structure of a DataFrame |
| StructType | Complete schema |
| StructField | Individual column definition |
| IntegerType | 32-bit integer |
| LongType | 64-bit integer |
| DoubleType | Approximate floating-point value |
| DecimalType | Exact decimal value |
| ArrayType | Collection of values |
| MapType | Key-value pairs |
| cast() | Changes a column's data type |
| printSchema() | Displays schema |
| inferSchema | Automatically identifies data types |
| mergeSchema | Supports compatible schema evolution in Delta writes |

### Important code to remember

```python
# Define schema
schema = StructType([
    StructField("id", IntegerType(), False),
    StructField("name", StringType(), True)
])

# Read with explicit schema
df = (
    spark.read
    .option("header", True)
    .schema(schema)
    .csv("/data/file.csv")
)

# Print schema
df.printSchema()

# Check data types
df.dtypes

# Change data type
df = df.withColumn(
    "id",
    col("id").cast("long")
)
```

---

## 16. Self-Assessment

Before moving to the next topic, make sure you can:

- [ ] Explain what a DataFrame schema is.
- [ ] Define a schema using StructType and StructField.
- [ ] Identify common PySpark data types.
- [ ] Read CSV data using an explicit schema.
- [ ] Explain schema inference versus explicit schema.
- [ ] Change column types using cast().
- [ ] Work with ArrayType, MapType, and StructType.
- [ ] Explain schema evolution in Delta Lake.
- [ ] Validate incoming DataFrame columns and data types.

**Interview readiness:** You should be able to define a schema, read a file with it, validate the incoming structure, and explain how schema changes are handled in a production ETL pipeline.

