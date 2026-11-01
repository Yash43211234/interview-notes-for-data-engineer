# Schema Validation in PySpark

## 1. What Is Schema Validation?

Schema validation is the process of checking whether incoming data follows the expected structure and data types.

A schema defines:
- Column names.
- Data types.
- Nullability.
- Expected structure of the dataset.

### Example

Suppose an orders dataset should contain:

| Column | Expected Type |
|---|---|
| order_id | Integer |
| customer_id | Integer |
| amount | Double |
| order_date | Date |
| status | String |

If the incoming data contains `amount` as a string instead of a numeric value, it may cause problems during calculations.

Schema validation helps identify these issues before data moves further through the pipeline.

**Key idea:** Schema validation ensures that incoming data has the expected structure before processing.

---

## 2. Why Is Schema Validation Important?

In real-world data pipelines, data can come from different sources:

- CSV files.
- JSON files.
- Databases.
- APIs.
- Kafka.
- Data lakes.

The structure of incoming data may change unexpectedly.

Common problems include:

1. Missing columns.
2. Unexpected columns.
3. Incorrect data types.
4. Unexpected NULL values.
5. Schema changes in source systems.
6. Inconsistent column names.

### Benefits

- Prevents unexpected transformation failures.
- Maintains consistent data structure.
- Detects source data changes.
- Improves data quality.
- Makes downstream processing more reliable.

---

## 3. Understanding PySpark Schema

PySpark provides `StructType` and `StructField` to define DataFrame schemas.

### StructType

`StructType` represents the complete schema of a DataFrame.

### StructField

`StructField` represents an individual column in the schema.

### Example

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    StringType,
    DoubleType,
    DateType
)

schema = StructType([
    StructField("order_id", IntegerType(), False),
    StructField("customer_id", IntegerType(), False),
    StructField("amount", DoubleType(), True),
    StructField("status", StringType(), True),
    StructField("order_date", DateType(), True)
])
```

### Understanding StructField

```python
StructField(
    "order_id",
    IntegerType(),
    False
)
```

It contains:

- `"order_id"`: Column name.
- `IntegerType()`: Expected data type.
- `False`: Column is declared non-nullable in the schema.

**Important:** The nullable setting is schema metadata. It does not automatically clean incoming data or guarantee that every source value is valid.

---

## 4. Common PySpark Data Types

| PySpark Type | Description |
|---|---|
| StringType | Text |
| IntegerType | 32-bit integer |
| LongType | 64-bit integer |
| FloatType | 32-bit floating-point number |
| DoubleType | 64-bit floating-point number |
| BooleanType | True or False |
| DateType | Date without time |
| TimestampType | Timestamp |
| DecimalType | Fixed-precision decimal |
| ArrayType | Array of values |
| MapType | Key-value structure |
| StructType | Nested structure |

Example:

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    StringType,
    IntegerType,
    ArrayType
)

schema = StructType([
    StructField("customer_id", IntegerType(), True),
    StructField("customer_name", StringType(), True),
    StructField(
        "phone_numbers",
        ArrayType(StringType()),
        True
    )
])
```

---

## 5. Defining Schema While Reading Data

Instead of relying on schema inference, we can define an explicit schema.

### Example: Reading CSV

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    DoubleType
)

schema = StructType([
    StructField("order_id", IntegerType(), True),
    StructField("customer_id", IntegerType(), True),
    StructField("amount", DoubleType(), True)
])

df = (
    spark.read
        .format("csv")
        .option("header", True)
        .schema(schema)
        .load("/data/orders.csv")
)
```

### Why Use an Explicit Schema?

1. Avoids unnecessary schema inference.
2. Makes expected data types clear.
3. Provides predictable DataFrame structure.
4. Helps detect incompatible or malformed input values.

**Important:** Defining a schema does not guarantee that every source value is valid. For example, an invalid numeric value may be handled according to the CSV parser's mode and options.

---

## 6. Inspecting an Existing Schema

PySpark provides useful methods to inspect a DataFrame's schema.

### 1. printSchema()

```python
df.printSchema()
```

Example output:

```text
root
 |-- order_id: integer (nullable = true)
 |-- customer_id: integer (nullable = true)
 |-- amount: double (nullable = true)
```

### 2. schema

```python
df.schema
```

Returns the DataFrame's schema as a `StructType` object.

### 3. dtypes

```python
df.dtypes
```

Returns column names and their data type names.

Example:

```python
[
    ("order_id", "int"),
    ("customer_id", "int"),
    ("amount", "double")
]
```

### 4. columns

```python
df.columns
```

Returns the list of column names.

---

## 7. Validating Column Names

Sometimes incoming data contains missing or unexpected columns.

We can compare the actual columns with the expected columns.

### Example

```python
expected_columns = [
    "order_id",
    "customer_id",
    "amount"
]

actual_columns = df.columns

missing_columns = set(expected_columns) - set(actual_columns)

unexpected_columns = set(actual_columns) - set(expected_columns)

print("Missing:", missing_columns)
print("Unexpected:", unexpected_columns)
```

### Example Result

Suppose the incoming DataFrame contains:

```python
["order_id", "customer_id", "status"]
```

Then:

```text
Missing: {'amount'}
Unexpected: {'status'}
```

### Validate Automatically

```python
if missing_columns:
    raise ValueError(
        f"Missing columns: {missing_columns}"
    )

if unexpected_columns:
    print(
        f"Unexpected columns: {unexpected_columns}"
    )
```

**Interview point:** Set comparison is a simple way to identify missing and unexpected columns. If column order is important, compare the column lists directly rather than using sets.

---

## 8. Validating Data Types

We can compare actual data types with expected data types.

### Example

```python
from pyspark.sql.types import (
    IntegerType,
    DoubleType
)

expected_schema = {
    "order_id": IntegerType(),
    "amount": DoubleType()
}

actual_schema = {
    field.name: field.dataType
    for field in df.schema.fields
}

for column, expected_type in expected_schema.items():

    actual_type = actual_schema.get(column)

    if actual_type != expected_type:
        print(
            f"Type mismatch in {column}: "
            f"expected {expected_type}, "
            f"received {actual_type}"
        )
```

### Example

Expected:

```text
amount → DoubleType
```

Actual:

```text
amount → StringType
```

This indicates a data type mismatch.

**Important:** Spark types should be compared using their actual type objects, not by relying on display strings alone.

---

## 9. Type Casting and Validation

Sometimes source data contains values in the wrong format.

We may need to convert them into the expected type.

Example:

```python
from pyspark.sql.functions import col

df = df.withColumn(
    "amount",
    col("amount").cast("double")
)
```

### Check Invalid Cast Results

For example, a string such as `"ABC"` cannot be converted into a valid double.

```python
invalid_df = df.filter(
    col("amount").isNull()
)
```

However, this check also identifies original NULL values.

A better approach is to preserve the original value and identify failed conversions separately.

```python
from pyspark.sql.functions import (
    col,
    expr
)

df = df.withColumn(
    "amount_converted",
    expr("try_cast(amount AS DOUBLE)")
)

invalid_df = df.filter(
    col("amount").isNotNull() &
    col("amount_converted").isNull()
)
```

**Remember:**
- `cast()` attempts type conversion.
- `try_cast()` returns NULL for values that cannot be converted, where supported.
- A successful cast does not necessarily mean the value satisfies business rules.

---

## 10. Handling Schema Mismatches

Suppose two datasets have different structures.

### Dataset 1

| order_id | amount |
|---|---:|
| 1 | 500 |
| 2 | 300 |

### Dataset 2

| order_id | amount | status |
|---|---:|---|
| 3 | 200 | SUCCESS |
| 4 | 400 | CANCELLED |

Using `union()` directly requires compatible column counts and positions.

For matching column names, use `unionByName()`.

```python
result = df1.unionByName(
    df2,
    allowMissingColumns=True
)
```

### Result

| order_id | amount | status |
|---|---:|---|
| 1 | 500 | NULL |
| 2 | 300 | NULL |
| 3 | 200 | SUCCESS |
| 4 | 400 | CANCELLED |

`allowMissingColumns=True` fills missing columns with NULL.

**Important:** `unionByName()` handles missing columns, but incompatible data types may still require explicit casting or other schema normalization.

---

## 11. Schema Evolution

### What Is Schema Evolution?

Schema evolution means handling changes in the structure of data over time.

For example, an existing orders dataset contains:

```text
order_id
customer_id
amount
```

A new version introduces:

```text
order_id
customer_id
amount
discount
```

The new `discount` column represents a schema change.

### Common Schema Changes

1. Adding a new column.
2. Removing a column.
3. Changing a data type.
4. Renaming a column.
5. Changing nested structures.

### How to Handle Schema Changes

- Compare incoming and expected schemas.
- Decide whether the change is allowed.
- Add missing columns with appropriate defaults or NULL values.
- Cast compatible data types.
- Reject or quarantine incompatible records.
- Update downstream transformations when necessary.

### Schema Evolution in Delta Lake

Delta Lake supports schema evolution for supported write operations.

Example:

```python
(
    new_df.write
        .format("delta")
        .mode("append")
        .option("mergeSchema", "true")
        .save("/data/delta/orders")
)
```

`mergeSchema` allows supported schema additions to be merged into the target table schema.

**Important:** Schema evolution should be controlled. Automatically accepting every schema change can introduce unexpected columns or break downstream logic. Type changes and column renames have additional compatibility considerations.

---

## 12. Schema Validation in Streaming

Schema validation is especially important in streaming pipelines because data arrives continuously.

For file-based streaming sources, an explicit schema is generally required.

### Example

```python
stream_df = (
    spark.readStream
        .format("json")
        .schema(schema)
        .load("/input/orders")
)
```

### Validate Required Columns

```python
required_columns = [
    "order_id",
    "customer_id",
    "amount"
]

missing_columns = (
    set(required_columns) -
    set(stream_df.columns)
)

if missing_columns:
    raise ValueError(
        f"Missing columns: {missing_columns}"
    )
```

### Important Considerations

- Define the expected schema before starting the query.
- Validate source data types and required fields.
- Handle malformed records appropriately.
- Monitor schema changes at the source.
- Avoid making incompatible changes to a running query without understanding recovery implications.

**Interview point:** Streaming schema validation helps prevent unexpected source changes from silently affecting downstream processing.

---

## 13. Schema Validation in Medallion Architecture

Schema validation can be performed at different stages.

| Layer | Validation Responsibility |
|---|---|
| Bronze | Check incoming structure and preserve raw data |
| Silver | Enforce expected columns, types, and standardized formats |
| Gold | Ensure business-ready datasets follow reporting requirements |

### Typical Flow

```text
Source Data
     |
     v
Bronze Layer
(Raw Data + Schema Inspection)
     |
     v
Silver Layer
(Schema Validation + Cleansing)
     |
     v
Gold Layer
(Consistent Business Structure)
```

Example:

- Bronze: Identify unexpected source columns.
- Silver: Convert amount to a numeric type and validate required fields.
- Gold: Ensure reporting tables contain the expected business columns and types.

---

## 14. Common Mistakes

1. Relying on schema inference for every production pipeline.
2. Assuming an explicit schema automatically validates every record.
3. Confusing NULL values with failed type conversions.
4. Ignoring missing columns.
5. Ignoring unexpected columns.
6. Assuming `union()` matches columns by name.
7. Using `unionByName()` without considering incompatible data types.
8. Automatically accepting every schema change.
9. Treating `nullable=False` as a complete runtime data quality check.
10. Changing a streaming schema without considering checkpoint compatibility.

---

## 15. Interview Questions

**Q1. What is schema validation in PySpark?**

Schema validation checks whether incoming data follows the expected structure, including column names, data types, and required fields.

**Q2. What is the difference between StructType and StructField?**

`StructType` represents the complete DataFrame schema, while `StructField` represents one individual column.

**Q3. Why do we use an explicit schema?**

It makes the expected structure clear, avoids unnecessary inference, and provides predictable data types.

**Q4. How do you inspect a DataFrame schema?**

Using `printSchema()`, `schema`, `dtypes`, and `columns`.

**Q5. How do you identify missing columns?**

Compare the expected column names with the actual DataFrame column names using set operations.

**Q6. How do you validate data types?**

Compare the expected PySpark data types with the actual types in `df.schema.fields`.

**Q7. What happens when we cast an invalid value?**

A normal cast may fail depending on the Spark configuration and execution mode. `try_cast()` returns NULL for invalid conversions where supported.

**Q8. What is schema evolution?**

Schema evolution is the process of handling changes in data structure, such as adding new columns or changing supported schema elements.

**Q9. What is the difference between union and unionByName?**

`union()` combines DataFrames by column position, while `unionByName()` combines them by column name.

**Q10. What does allowMissingColumns=True do?**

It allows `unionByName()` to handle missing columns by filling their values with NULL.

**Q11. What is mergeSchema in Delta Lake?**

It is a supported Delta Lake write option that allows compatible schema additions to be merged into the target table schema.

**Q12. How do you validate schema in Structured Streaming?**

Define the expected schema, inspect the streaming DataFrame structure, validate required fields and data types, and handle malformed or incompatible input.

**Q13. Does nullable=False guarantee that a column never contains NULL?**

No. It is schema metadata and is not a substitute for explicit data validation.

---

## 16. Quick Revision

| Concept | Remember |
|---|---|
| Schema | Structure of a DataFrame |
| StructType | Complete schema |
| StructField | Individual column definition |
| printSchema() | Displays schema |
| dtypes | Displays column data types |
| columns | Returns column names |
| Explicit Schema | Predefined expected structure |
| Schema Validation | Checks actual vs expected schema |
| Type Casting | Converts data types |
| Schema Evolution | Handles schema changes |
| union() | Combines by position |
| unionByName() | Combines by column name |
| allowMissingColumns | Fills missing columns with NULL |
| mergeSchema | Supports Delta schema additions |

### One-Line Revision

- StructType → Complete schema.
- StructField → One column.
- Explicit schema → Predefined structure.
- Schema validation → Check expected structure.
- Type casting → Convert data types.
- Schema evolution → Handle structure changes.
- `union()` → Match by position.
- `unionByName()` → Match by name.
- `mergeSchema` → Merge supported Delta schema additions.

---

## 17. Self-Check

- [ ] Can I explain schema validation?
- [ ] Can I explain StructType and StructField?
- [ ] Can I define an explicit schema?
- [ ] Can I inspect an existing schema?
- [ ] Can I identify missing and unexpected columns?
- [ ] Can I validate data types?
- [ ] Can I handle invalid type conversions?
- [ ] Can I explain schema evolution?
- [ ] Can I explain union vs unionByName?
- [ ] Can I explain mergeSchema?
- [ ] Can I validate schemas in streaming pipelines?
- [ ] Can I explain schema validation in Bronze, Silver, and Gold?

**Final takeaway:** Schema validation ensures that incoming data follows the expected structure. In a reliable data pipeline, validate column names, data types, and required fields before allowing data to move into downstream transformations.