# Schema Enforcement and Schema Evolution in Delta Lake

## 1. Overview

Schema enforcement and schema evolution help maintain data consistency when writing data into Delta tables.

They solve two different problems:

- **Schema Enforcement:** Prevents incompatible data from being written into a table.
- **Schema Evolution:** Allows the table schema to change when incoming data introduces new columns or supported schema changes.

These concepts are important in production ETL pipelines because source data can change over time.

## 2. What Is a Schema?

A schema defines the structure of a table or DataFrame.

It specifies:
- Column names
- Data types
- Nullability metadata
- Column order

Example:

| Column | Data Type |
|---|---|
| customer_id | INT |
| customer_name | STRING |
| city | STRING |
| age | INT |

A Delta table stores its schema in its transaction log.

You can inspect a table's schema using:

```sql
DESCRIBE TABLE customers;
```

In PySpark:

```python
df.printSchema()
```

## 3. Schema Enforcement

Schema enforcement is also called **schema validation**.

Delta Lake checks incoming data against the target table's schema during a write operation.

If the incoming data has an incompatible schema, Delta Lake rejects the write instead of silently accepting an incompatible structure.

### Example

Existing Delta table:

```text
customer_id INT
name        STRING
age         INT
```

Incoming DataFrame:

```text
customer_id INT
name        STRING
age         STRING
```

The incoming `age` column is STRING, but the target table expects INT.

This is a schema mismatch.

Depending on the write operation and supported type conversions, the write may fail.

### Why is schema enforcement important?

Without proper schema validation, unexpected source changes can introduce data-quality problems.

Benefits:

- Maintains consistent table structure.
- Prevents incompatible schema changes.
- Reduces unexpected data-type problems.
- Protects downstream ETL transformations.
- Makes data pipelines more predictable.

## 4. Schema Enforcement Example

Suppose we already have a Delta table named `customers`.

```python
df = spark.createDataFrame(
    [
        (1, "Rahul", 25),
        (2, "Amit", 30)
    ],
    ["customer_id", "name", "age"]
)

df.write.format("delta").mode("overwrite").saveAsTable(
    "customers"
)
```

Now suppose the incoming DataFrame contains an additional column:

```python
new_df = spark.createDataFrame(
    [
        (3, "Priya", 28, "Kolkata")
    ],
    ["customer_id", "name", "age", "city"]
)
```

Attempting to append it without schema evolution:

```python
new_df.write.format("delta").mode("append").saveAsTable(
    "customers"
)
```

This normally fails because the incoming DataFrame contains a column that is not part of the existing table schema.

The exact compatibility checks depend on the write path and supported type conversions.

## 5. What Is Schema Evolution?

Schema evolution allows a Delta table's schema to change to accommodate incoming data.

For example, an existing table contains:

```text
customer_id INT
name        STRING
```

A new source dataset contains:

```text
customer_id INT
name        STRING
city        STRING
```

With schema evolution enabled, Delta Lake can add the new `city` column to the target table.

The resulting schema becomes:

```text
customer_id INT
name        STRING
city        STRING
```

Existing records receive NULL for the newly added column unless another value is provided.

### Why is schema evolution useful?

Real-world source systems change over time.

Examples:
- A source application introduces a new column.
- A business starts collecting additional customer information.
- An upstream team changes the data structure.
- New fields are added to incoming JSON or other semi-structured data.

Schema evolution helps pipelines handle supported changes without manually recreating the target table.

## 6. Enable Schema Evolution in PySpark

One common approach is using the `mergeSchema` option.

```python
new_df.write \
    .format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \
    .saveAsTable("customers")
```

This allows supported incoming schema additions to be merged into the existing Delta table schema.

### SQL example

In supported Databricks environments, schema evolution can be enabled for a MERGE operation using:

```sql
MERGE WITH SCHEMA EVOLUTION INTO customers AS target
USING new_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
    UPDATE SET *

WHEN NOT MATCHED THEN
    INSERT *;
```

The exact syntax and availability depend on the Databricks Runtime or Delta Lake version.

Another approach in supported environments is to use the `spark.databricks.delta.schema.autoMerge.enabled` configuration.

```python
spark.conf.set(
    "spark.databricks.delta.schema.autoMerge.enabled",
    "true"
)
```

This is a session-level configuration commonly used in Databricks environments. Prefer operation-level schema evolution when available because it makes the intended schema change more explicit.

## 7. Schema Enforcement vs Schema Evolution

| Feature | Schema Enforcement | Schema Evolution |
|---|---|---|
| Purpose | Validate incoming schema | Allow supported schema changes |
| Behavior | Rejects incompatible writes | Updates table schema |
| New columns | Not automatically accepted | Can be added |
| Data consistency | Protects existing structure | Adapts structure |
| Main use | Prevent unexpected changes | Handle expected changes |

**Interview point:**

Schema enforcement protects the existing table structure, whereas schema evolution allows controlled changes to that structure.

They are complementary features, not opposites.

## 8. Schema Evolution in MERGE

Schema evolution is particularly useful when performing incremental data processing using MERGE.

### Existing target table

```text
customer_id INT
name        STRING
```

### Incoming source data

```text
customer_id INT
name        STRING
city        STRING
```

We want to:
- Update existing customers.
- Insert new customers.
- Add the new `city` column to the target schema.

In a supported Databricks Runtime, use:

```sql
MERGE WITH SCHEMA EVOLUTION INTO customers AS target
USING new_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
    UPDATE SET *

WHEN NOT MATCHED THEN
    INSERT *;
```

After a successful operation, the target table schema can include `city`.

**Important:** Schema evolution does not mean every incoming change is automatically accepted. The change must be supported by the runtime and the operation. Also, wildcard assignments such as `UPDATE SET *` and `INSERT *` have column-matching behavior and should be used carefully.

## 9. Adding a Column Manually

You can also change a Delta table's schema explicitly.

```sql
ALTER TABLE customers
ADD COLUMNS (
    city STRING
);
```

This adds a new column without requiring schema evolution during a data write.

Existing records return NULL for this column until values are written.

In production pipelines, explicit schema changes are often preferable when a change needs review and controlled deployment.

## 10. Schema Evolution vs ALTER TABLE

| Schema Evolution | ALTER TABLE |
|---|---|
| Schema changes during supported write operations | Schema changes through an explicit SQL command |
| Useful for incoming source changes | Useful for planned schema updates |
| Can automatically add supported columns | Allows controlled table modification |
| Depends on write options and runtime | Depends on supported DDL operations |

Both approaches can change the schema, but they serve different workflow requirements.

## 11. Schema Evolution Best Practices

1. **Validate incoming data:** Check column names, data types, and required fields before writing.
2. **Avoid automatic acceptance of every change:** Unexpected columns may indicate an upstream issue.
3. **Use explicit schema definitions:** Avoid relying entirely on schema inference in production pipelines.
4. **Review data-type changes:** Changing INT to STRING or other types may require explicit migration logic.
5. **Use operation-level evolution where available:** Keep schema changes visible in the write operation.
6. **Test downstream dependencies:** New or changed columns can affect SQL queries, reports, and transformations.
7. **Maintain backward compatibility:** Avoid removing or renaming columns without coordinating downstream consumers.
8. **Monitor schema changes:** Record and review changes introduced by source systems.

## 12. Real-World Data Engineering Example

Suppose an e-commerce application initially sends:

```text
order_id
customer_id
amount
```

Later, the application introduces:

```text
order_id
customer_id
amount
discount
```

A production pipeline can follow this process:

1. Read incoming order data.
2. Validate existing column names and data types.
3. Identify the newly introduced `discount` column.
4. Confirm that the change is expected.
5. Enable schema evolution or apply an approved ALTER TABLE.
6. Write the data into the Delta table.
7. Test downstream transformations and reports.

This allows the pipeline to adapt while maintaining control over data structure changes.

## 13. Common Interview Questions

**Q1. What is schema enforcement in Delta Lake?**

Schema enforcement validates incoming data against the existing Delta table schema and rejects incompatible writes.

**Q2. What is schema evolution?**

Schema evolution allows supported schema changes, such as adding new columns, during a write operation.

**Q3. What is the difference between schema enforcement and schema evolution?**

Enforcement protects the existing schema from incompatible changes. Evolution allows supported changes to the schema.

**Q4. How do you enable schema evolution in PySpark?**

A common approach is:

```python
df.write \
    .format("delta") \
    .mode("append") \
    .option("mergeSchema", "true") \
    .saveAsTable("customers")
```

**Q5. What happens to existing records when a new column is added?**

Existing records return NULL for the new column unless values are populated through a subsequent update or write.

**Q6. Does schema evolution automatically accept every data-type change?**

No. Schema evolution supports specific changes depending on the operation, runtime, and configuration. Some changes require explicit casting or table migration.

**Q7. Can schema evolution be used with MERGE?**

Yes. Supported Delta Lake and Databricks versions provide schema evolution for MERGE operations.

**Q8. What is the difference between mergeSchema and autoMerge?**

`mergeSchema` is commonly specified as a write option for a particular operation. `spark.databricks.delta.schema.autoMerge.enabled` is a session-level configuration that enables automatic schema evolution for supported operations.

**Q9. What is the purpose of ALTER TABLE ADD COLUMNS?**

It explicitly adds columns to an existing table schema without depending on an incoming write to introduce them.

**Q10. Why should schema evolution be controlled in production?**

Uncontrolled schema changes can break downstream transformations, dashboards, and applications. Expected changes should be validated and communicated.

## 14. Quick Revision

| Concept | Remember |
|---|---|
| Schema | Defines table structure |
| Schema enforcement | Validates incoming schema |
| Schema evolution | Allows supported schema changes |
| `mergeSchema` | Common write option for schema evolution |
| `autoMerge` | Session-level automatic evolution configuration |
| `ALTER TABLE ADD COLUMNS` | Explicitly adds columns |
| New column | Existing records generally return NULL |
| MERGE evolution | Supported in compatible versions |
| Production practice | Validate and control schema changes |

## 15. Self-Assessment

- [ ] Explain schema enforcement.
- [ ] Explain schema evolution.
- [ ] Differentiate enforcement and evolution.
- [ ] Write PySpark code using `mergeSchema`.
- [ ] Explain how schema evolution works with MERGE.
- [ ] Explain how to add a column using ALTER TABLE.
- [ ] Describe what happens to existing records after adding a column.
- [ ] Explain why uncontrolled schema changes are risky.
- [ ] Give a real-world example of schema evolution in an ETL pipeline.

**Interview takeaway:** Schema enforcement protects Delta tables from incompatible incoming data, while schema evolution allows controlled structural changes. In production, validate source changes before enabling evolution so that flexibility does not compromise data quality.