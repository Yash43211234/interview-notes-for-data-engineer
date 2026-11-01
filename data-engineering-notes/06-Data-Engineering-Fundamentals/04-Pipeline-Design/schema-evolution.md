# Schema Evolution in Data Engineering

## 1. Introduction

**Schema evolution is the process of handling changes in the structure of data over time without unnecessarily breaking existing data pipelines or applications.**

A schema defines the structure of a dataset, including:
- Column names.
- Data types.
- Nullability.
- Nested fields.
- Field relationships.

As business requirements change, source systems may introduce new columns, remove existing columns, or change data types.

A reliable data pipeline should handle these changes in a controlled way.

### Example

Initially, an orders table contains:

| order_id | customer_id | amount |
|---|---|---:|
| 101 | C01 | 500 |
| 102 | C02 | 700 |

Later, the source system adds a new column called `payment_method`.

| order_id | customer_id | amount | payment_method |
|---|---|---:|---|
| 101 | C01 | 500 | UPI |
| 102 | C02 | 700 | Card |

The schema has evolved because a new column was introduced.

**Key idea:** Schema evolution allows data structures to change while maintaining compatibility and data quality.

---

## 2. Why Is Schema Evolution Needed?

In real-world data systems, schemas are not always fixed.

Schema changes can happen because of:
- New business requirements.
- New source-system features.
- Application updates.
- New data attributes.
- Changes in business logic.
- Integration with additional data sources.

### Example

An e-commerce application initially stores:

```text
order_id
customer_id
amount
```

Later, the business wants to track:

```text
payment_method
delivery_status
discount_amount
```

The source schema changes, and downstream pipelines must handle the additional fields.

### Benefits

- Supports changing business requirements.
- Reduces unnecessary pipeline failures.
- Allows new attributes to be introduced.
- Helps maintain compatibility between systems.
- Supports evolving data platforms.

---

## 3. Common Types of Schema Changes

### 3.1 Adding a Column

A new column is introduced.

Before:

```text
employee_id
employee_name
salary
```

After:

```text
employee_id
employee_name
salary
department
```

This is generally an additive schema change.

Existing records may have NULL values for the new column when the value is not available.

### 3.2 Removing a Column

An existing column is removed.

Before:

```text
employee_id
employee_name
salary
department
```

After:

```text
employee_id
employee_name
salary
```

This can break downstream applications that depend on `department`.

Before removing a column, identify its dependencies.

### 3.3 Renaming a Column

Example:

```text
customer_name
```

changes to:

```text
full_name
```

Applications referencing the old column may fail.

Renaming should be coordinated with downstream consumers.

### 3.4 Changing a Data Type

Example:

```text
salary: INTEGER
```

changes to:

```text
salary: DECIMAL(12,2)
```

This may affect calculations, storage, validation, and application logic.

Some type changes are compatible, while others can cause data loss or query failures.

### 3.5 Adding a Nested Field

A nested JSON structure may gain a new field.

Before:

```json
{
  "customer": {
    "id": "C101",
    "name": "Rahul"
  }
}
```

After:

```json
{
  "customer": {
    "id": "C101",
    "name": "Rahul",
    "city": "Kolkata"
  }
}
```

The pipeline must handle the additional nested field.

---

## 4. Backward and Forward Compatibility

Compatibility describes whether different versions of a schema can work with existing producers and consumers.

### 4.1 Backward Compatibility

A new schema can read data written using an older schema.

Example:

A new application understands the original `employee_id` and `employee_name` fields, even when older records do not contain the newly added `department` field.

### 4.2 Forward Compatibility

An older consumer can handle data written using a newer schema.

Example:

A consumer reads `employee_id` and `employee_name` while ignoring an additional `department` field.

This requires the consumer and serialization format to support handling unknown fields appropriately.

### 4.3 Full Compatibility

A schema change is fully compatible when both backward and forward compatibility requirements are satisfied under the relevant format and system rules.

### Comparison

| Compatibility | Meaning |
|---|---|
| Backward | New reader can read old data |
| Forward | Old reader can handle new data |
| Full | Both directions are supported |

**Important:** Compatibility depends on the serialization format, schema rules, and reader/writer behavior. Adding a column is not automatically compatible in every system.

---

## 5. Schema Evolution in PySpark

PySpark supports schema handling through explicit schemas, schema inference, and DataFrame transformations.

### 5.1 Explicit Schema

An explicit schema defines the expected structure of the incoming data.

```python
from pyspark.sql.types import (
    StructType,
    StructField,
    IntegerType,
    StringType,
    DoubleType
)

schema = StructType([
    StructField("order_id", IntegerType(), True),
    StructField("customer_id", StringType(), True),
    StructField("amount", DoubleType(), True)
])

df = (
    spark.read
    .schema(schema)
    .option("header", True)
    .csv("/data/orders.csv")
)
```

**Benefits:**
- Consistent data types.
- Predictable processing.
- Less dependence on schema inference.
- Better control over expected columns.

An explicit schema does not guarantee that every incoming value is valid. Data validation is still required.

### 5.2 Adding a Column

A new column can be added using `withColumn()`.

```python
from pyspark.sql import functions as F

updated_df = df.withColumn(
    "payment_method",
    F.lit(None).cast("string")
)
```

This creates a new DataFrame with an additional nullable column.

### 5.3 Selecting Expected Columns

If the incoming source contains extra columns, select the fields required by the target.

```python
expected_df = updated_df.select(
    "order_id",
    "customer_id",
    "amount",
    "payment_method"
)
```

This is useful when the pipeline needs a controlled output schema.

### Important Point

PySpark DataFrames are immutable. Operations such as `withColumn()` and `select()` return new DataFrames rather than modifying the original DataFrame.

---

## 6. Schema Evolution in Delta Lake

Delta Lake supports schema enforcement and controlled schema evolution.

### 6.1 Schema Enforcement

Schema enforcement checks whether incoming data is compatible with the existing Delta table schema during writes.

It helps prevent unintended schema changes and incompatible writes.

Example:

```python
(
    df.write
    .format("delta")
    .mode("append")
    .saveAsTable("silver.orders")
)
```

If the incoming schema is incompatible with the target schema, the write may fail.

### 6.2 Schema Evolution Using mergeSchema

When adding compatible new columns is intended, schema evolution can be enabled.

```python
(
    updated_df.write
    .format("delta")
    .option("mergeSchema", "true")
    .mode("append")
    .saveAsTable("silver.orders")
)
```

The `mergeSchema` option allows supported schema changes, such as adding compatible columns, to be incorporated into the target schema.

### 6.3 Session-Level Configuration

A session-level configuration can enable schema auto-merge for supported operations:

```python
spark.conf.set(
    "spark.databricks.delta.schema.autoMerge.enabled",
    "true"
)
```

Use this carefully because it can allow schema changes across multiple operations in the session.

Prefer operation-level configuration when a change is needed only for a specific write.

### 6.4 Schema Enforcement vs Schema Evolution

| Feature | Schema Enforcement | Schema Evolution |
|---|---|---|
| Purpose | Protect the existing schema | Allow supported schema changes |
| Behavior | Reject incompatible writes | Incorporate approved changes |
| Main benefit | Data consistency | Flexibility |
| Example | Prevent unexpected columns | Add a new column |

**Interview point:** Schema enforcement protects the target structure, while schema evolution allows controlled changes to that structure.

---

## 7. Schema Evolution in Databricks Auto Loader

Auto Loader supports schema inference and evolution when ingesting files from cloud storage.

It is commonly used when incoming files may contain new columns over time.

### Example

```python
from pyspark.sql import functions as F

source_df = (
    spark.readStream
    .format("cloudFiles")
    .option("cloudFiles.format", "json")
    .option(
        "cloudFiles.schemaLocation",
        "/checkpoints/orders/schema"
    )
    .option(
        "cloudFiles.schemaEvolutionMode",
        "addNewColumns"
    )
    .load("/data/incoming/orders")
)
```

### Important Options

| Option | Purpose |
|---|---|
| `cloudFiles.format` | Specifies the input file format |
| `cloudFiles.schemaLocation` | Stores schema-tracking information |
| `cloudFiles.schemaEvolutionMode` | Defines how schema changes are handled |
| `addNewColumns` | Allows supported new columns to be added |
| `rescue` | Keeps unexpected or incompatible data in the rescued data column when configured |
| `failOnNewColumns` | Stops processing when new columns are detected |

### Schema Evolution Modes

Common modes include:

**1. `addNewColumns`**

- Adds supported new columns.
- Updates schema metadata.
- The stream stops when a new column is detected.
- The stream must be restarted to continue with the updated schema.

**2. `rescue`**

- Does not automatically evolve the schema for new columns.
- Unexpected or incompatible data can be captured in the rescued data column when configured.

**3. `failOnNewColumns`**

- Fails when new columns are detected.
- Useful when strict schema control is required.

**4. `none`**

- Does not evolve the schema automatically.
- Unexpected fields may be ignored unless rescued-data handling is configured.

Exact behavior and supported modes depend on the Databricks Runtime and Auto Loader configuration.

### Important Point

Auto Loader schema evolution and Delta Lake schema evolution are different:

- Auto Loader handles changes in incoming file schemas.
- Delta Lake handles supported changes to the target table schema.

A pipeline may need both.

---

## 8. Schema Evolution in a Medallion Architecture

Schema changes should be handled carefully across Bronze, Silver, and Gold layers.

### Bronze Layer

Store incoming data with as much source information as required.

Common practices:
- Preserve raw data.
- Track source metadata.
- Capture schema changes.
- Avoid unnecessary loss of source fields.

### Silver Layer

Apply controlled schema handling.

Common practices:
- Validate expected columns.
- Cast data types.
- Handle missing fields.
- Rename columns when required.
- Capture invalid records.
- Apply business rules.

### Gold Layer

Maintain stable schemas for reporting and analytics.

Common practices:
- Expose only required business columns.
- Coordinate breaking changes with consumers.
- Update downstream models and dashboards.
- Preserve consistent business definitions.

### Example

```text
Source Files
     |
     v
Bronze
(Raw Schema + New Fields)
     |
     v
Silver
(Validation + Standardization)
     |
     v
Gold
(Stable Business Schema)
     |
     v
Reports and Analytics
```

A new source column does not necessarily need to appear in every downstream table.

---

## 9. Schema Drift vs Schema Evolution

These terms are related but different.

### Schema Drift

Schema drift occurs when the incoming data structure changes unexpectedly compared with the expected schema.

Examples:
- A new column appears.
- A column disappears.
- A data type changes.
- A field name changes.

### Schema Evolution

Schema evolution is the controlled process of handling and adapting to schema changes.

### Comparison

| Schema Drift | Schema Evolution |
|---|---|
| Unexpected or detected schema change | Controlled handling of schema changes |
| Can cause pipeline failures | Helps maintain pipeline compatibility |
| Describes a change or condition | Describes a management process |
| Example: New source column appears unexpectedly | Pipeline safely incorporates the approved new column |

**Remember:** Schema drift is a change that needs attention. Schema evolution is how the system manages that change.

---

## 10. Challenges and Best Practices

### Common Challenges

1. Downstream jobs may depend on old column names.
2. Data type changes may cause conversion errors.
3. Missing columns may produce NULL values.
4. New fields may introduce unexpected data.
5. Schema changes may affect reporting logic.
6. Automatic evolution may introduce unwanted columns.
7. Different source files may contain different schema versions.

### Best Practices

**1. Prefer Explicit Schemas**

Use defined schemas for important production datasets instead of relying entirely on inference.

**2. Validate Incoming Data**

Check required columns, data types, nullability, and business rules.

**3. Control Schema Changes**

Review new fields before exposing them to downstream consumers.

**4. Handle Missing Columns**

Use explicit defaults or nullable values where appropriate.

**5. Avoid Unplanned Type Changes**

Validate compatibility before changing data types.

**6. Maintain Data Lineage**

Understand which pipelines, tables, and reports depend on affected columns.

**7. Separate Raw and Business Schemas**

Preserve source information in Bronze and expose controlled schemas in Silver and Gold.

**8. Test Schema Changes**

Test old and new data versions before deploying changes to production.

**9. Monitor Schema Changes**

Log new columns, removed fields, type changes, and rejected records.

**10. Coordinate Breaking Changes**

Communicate changes to downstream teams before renaming or removing fields.

---

## 11. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Schema evolution means accepting every change | Changes should be controlled and validated |
| `mergeSchema` fixes every schema mismatch | It supports specific compatible evolution scenarios |
| Schema inference guarantees correct data types | Inference can produce incorrect or inconsistent types |
| Explicit schema validates every record | Values still need data-quality validation |
| Adding a column is always harmless | Downstream consumers and compatibility rules matter |
| Auto Loader and Delta schema evolution are identical | They handle different stages of the pipeline |
| Missing columns always cause failure | Behavior depends on the read/write operation and schema configuration |
| Renaming a column is always backward compatible | Existing consumers may depend on the old name |
| Schema drift and schema evolution mean the same thing | Drift describes a change; evolution describes how it is handled |
| New Bronze columns must appear in Gold | Each layer should expose only the fields required by its purpose |

---

## 12. Interview Questions and Answers

### Q1. What is schema evolution?

Schema evolution is the process of handling changes in data structure over time while maintaining compatibility and data quality.

### Q2. Why is schema evolution important?

Source systems and business requirements change. Schema evolution allows pipelines to adapt to new columns, nested fields, and supported type changes without unnecessary failures.

### Q3. What are common types of schema changes?

Adding, removing, or renaming columns, changing data types, and modifying nested structures.

### Q4. What is schema enforcement?

Schema enforcement checks incoming data against the existing target schema and rejects incompatible writes.

### Q5. What is the difference between schema enforcement and schema evolution?

Schema enforcement protects the existing structure. Schema evolution allows supported and controlled changes to that structure.

### Q6. How do you enable schema evolution in Delta Lake?

For supported writes, I can use the `mergeSchema` option or an appropriate session-level auto-merge configuration.

### Q7. What is Auto Loader schema evolution?

Auto Loader schema evolution allows supported changes in incoming file schemas to be detected and handled according to the configured evolution mode.

### Q8. What is the difference between `schemaLocation` and `checkpointLocation`?

`schemaLocation` stores Auto Loader schema metadata. `checkpointLocation` stores Structured Streaming query progress and state.

### Q9. What is schema drift?

Schema drift is an unexpected or detected change in the incoming data structure, such as a new column or changed data type.

### Q10. How do you handle schema drift?

I compare incoming data with the expected schema, validate changes, capture unexpected fields when appropriate, and apply approved changes in a controlled manner.

### Q11. What happens if a new column appears in Auto Loader with `addNewColumns` mode?

Auto Loader updates its schema metadata and stops the stream. The stream must be restarted to continue using the updated schema.

### Q12. How do you handle schema changes in a medallion architecture?

I preserve source information in Bronze, validate and standardize the schema in Silver, and maintain controlled business-facing schemas in Gold.

### Q13. How would you handle a column data type change?

I would check whether the change is compatible, assess downstream dependencies, validate existing and incoming values, and deploy the change in a controlled manner.

### Q14. Should every new source column be added to the Gold layer?

No. Gold should contain fields required by business reporting and analytics. A new source column should be included only when there is a business requirement.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Schema | Structure of a dataset |
| Schema evolution | Controlled handling of schema changes |
| Schema drift | Unexpected or detected structure change |
| Schema enforcement | Protects the target schema |
| `mergeSchema` | Allows supported Delta schema evolution |
| Auto Loader | Supports incremental file ingestion and schema evolution |
| `schemaLocation` | Stores Auto Loader schema metadata |
| `checkpointLocation` | Stores streaming progress and state |
| Backward compatibility | New reader reads old data |
| Forward compatibility | Old reader handles new data |
| Bronze | Preserve incoming source information |
| Silver | Validate and standardize schema |
| Gold | Maintain business-facing schema |

### One-Minute Interview Answer

Schema evolution is the process of handling changes in data structure over time.

For example, an orders source table may initially contain `order_id`, `customer_id`, and `amount`. Later, the source may introduce a new column called `payment_method`.

In PySpark, I can define explicit schemas and validate incoming data. In Delta Lake, I can enable supported schema changes using `mergeSchema`. In Databricks Auto Loader, I can configure schema evolution modes to handle new fields in incoming files.

In a medallion architecture, I preserve source information in Bronze, validate and standardize data in Silver, and maintain stable business-facing schemas in Gold. I also check downstream dependencies before making breaking changes.

---

## 14. Self-Assessment

- [ ] Define schema evolution.
- [ ] Explain why schemas change.
- [ ] Identify common schema changes.
- [ ] Differentiate backward and forward compatibility.
- [ ] Explain schema enforcement.
- [ ] Explain Delta Lake `mergeSchema`.
- [ ] Explain Auto Loader schema evolution.
- [ ] Differentiate `schemaLocation` and `checkpointLocation`.
- [ ] Explain schema drift.
- [ ] Describe schema handling in Bronze, Silver, and Gold.
- [ ] Explain how to handle a new column.
- [ ] Explain how to handle a data type change.
- [ ] Identify downstream risks of renaming or removing columns.

**Interview readiness:** Be able to explain this scenario: *Your source system adds a new column to an orders dataset. How would you detect the change, handle it in Auto Loader and Delta Lake, and prevent downstream reporting pipelines from breaking?*