# MERGE, UPSERT, and DELETE in Delta Lake

## 1. Overview

Delta Lake supports modifying existing table data using operations such as:

- **INSERT:** Add new records.
- **UPDATE:** Modify existing records.
- **DELETE:** Remove records.
- **MERGE:** Insert, update, or delete records based on a matching condition.

These operations are especially useful in data engineering pipelines where source data changes over time.

## 2. What Is UPSERT?

UPSERT means **UPDATE + INSERT**.

It combines two operations:

- If a matching record already exists, update it.
- If no matching record exists, insert it.

### Example

Suppose the target table contains:

| customer_id | name | city |
|---|---|---|
| 1 | Rahul | Delhi |
| 2 | Amit | Mumbai |

New source data:

| customer_id | name | city |
|---|---|---|
| 2 | Amit | Pune |
| 3 | Priya | Kolkata |

After UPSERT:

| customer_id | name | city |
|---|---|---|
| 1 | Rahul | Delhi |
| 2 | Amit | Pune |
| 3 | Priya | Kolkata |

- Customer 2 is updated.
- Customer 3 is inserted.
- Customer 1 remains unchanged.

**Important:** UPSERT is a logical operation, not a separate SQL command in standard Delta Lake usage. It is commonly implemented using `MERGE INTO`.

## 3. MERGE INTO

`MERGE INTO` is used to synchronize a target Delta table with a source dataset.

It compares source and target records using a matching condition.

### Basic syntax

```sql
MERGE INTO target_table AS target
USING source_table AS source
ON target.id = source.id

WHEN MATCHED THEN
    UPDATE SET
        target.name = source.name

WHEN NOT MATCHED THEN
    INSERT (id, name)
    VALUES (source.id, source.name);
```

### Understanding the clauses

| Clause | Meaning |
|---|---|
| `MERGE INTO` | Specifies the target table |
| `USING` | Specifies the source data |
| `ON` | Defines the matching condition |
| `WHEN MATCHED` | Executes when a source row matches a target row |
| `WHEN NOT MATCHED` | Executes when a source row has no target match |
| `UPDATE SET` | Updates existing records |
| `INSERT` | Inserts new records |

## 4. MERGE Example: Update and Insert

Assume the target table is `customers` and the source table is `new_customers`.

```sql
MERGE INTO customers AS target
USING new_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
    UPDATE SET
        target.name = source.name,
        target.city = source.city

WHEN NOT MATCHED THEN
    INSERT (
        customer_id,
        name,
        city
    )
    VALUES (
        source.customer_id,
        source.name,
        source.city
    );
```

**What happens?**

1. Delta Lake compares records using `customer_id`.
2. Matching customer IDs are updated.
3. New customer IDs are inserted.
4. Existing target records without a source match remain unchanged.

This is a common implementation of an UPSERT operation.

## 5. MERGE with Conditional Updates

Sometimes we should update records only when their values have actually changed.

```sql
MERGE INTO customers AS target
USING new_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED
AND (
    NOT (target.name <=> source.name)
    OR NOT (target.city <=> source.city)
)
THEN UPDATE SET
    target.name = source.name,
    target.city = source.city

WHEN NOT MATCHED THEN
    INSERT (
        customer_id,
        name,
        city
    )
    VALUES (
        source.customer_id,
        source.name,
        source.city
    );
```

Here, `<=>` is Spark SQL's null-safe equality operator.

- Returns true when both values are equal.
- Returns true when both values are NULL.
- Returns false when only one value is NULL.

This avoids unnecessary updates when source and target values are unchanged.

## 6. DELETE Operation

`DELETE` removes matching records from a Delta table.

### Syntax

```sql
DELETE FROM customers
WHERE customer_id = 10;
```

This deletes the record where `customer_id = 10`.

### Delete using a condition

```sql
DELETE FROM customers
WHERE city = 'Delhi';
```

This deletes all records whose city is Delhi.

### Delete all records

```sql
DELETE FROM customers
WHERE 1 = 1;
```

This removes all rows from the table but does not drop the table itself.

**Important:** Delta Lake records the logical removal in its transaction log. The underlying data files are not necessarily physically deleted immediately. Old files may remain until they become eligible for `VACUUM`.

## 7. MERGE with DELETE

`MERGE` can also delete existing target records.

Example: Remove customers marked as inactive in the source.

```sql
MERGE INTO customers AS target
USING customer_updates AS source
ON target.customer_id = source.customer_id

WHEN MATCHED
AND source.status = 'INACTIVE'
THEN DELETE

WHEN MATCHED THEN
    UPDATE SET
        target.name = source.name,
        target.city = source.city

WHEN NOT MATCHED
AND source.status = 'ACTIVE'
THEN INSERT (
    customer_id,
    name,
    city
)
VALUES (
    source.customer_id,
    source.name,
    source.city
);
```

**Execution logic:**

- Matching inactive customers are deleted.
- Other matching customers are updated.
- New active customers are inserted.
- New inactive customers are not inserted.

The order of applicable `WHEN MATCHED` clauses matters. If multiple clauses are present, only the first applicable clause is executed for a given matched row.

## 8. WHEN NOT MATCHED BY SOURCE

This clause handles target records that do not have a matching source record.

For example, delete target records that are absent from the source:

```sql
MERGE INTO customers AS target
USING source_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
    UPDATE SET
        target.name = source.name

WHEN NOT MATCHED BY SOURCE THEN
    DELETE;
```

**Important:** This can delete many target records. Use it only when the source represents the complete and authoritative dataset for the relevant target scope.

You can also use a condition:

```sql
WHEN NOT MATCHED BY SOURCE
AND target.status = 'INACTIVE'
THEN DELETE
```

This clause is supported by modern Delta Lake implementations, but availability can depend on the runtime and version.

## 9. MERGE in PySpark

Delta Lake provides the `DeltaTable` API for performing MERGE operations.

### Import

```python
from delta.tables import DeltaTable
```

### Load target table

```python
target = DeltaTable.forName(
    spark,
    "customers"
)
```

### Perform UPSERT

```python
(
    target.alias("target")
    .merge(
        source_df.alias("source"),
        "target.customer_id = source.customer_id"
    )
    .whenMatchedUpdate(
        set={
            "name": "source.name",
            "city": "source.city"
        }
    )
    .whenNotMatchedInsert(
        values={
            "customer_id": "source.customer_id",
            "name": "source.name",
            "city": "source.city"
        }
    )
    .execute()
)
```

### MERGE with DELETE

```python
(
    target.alias("target")
    .merge(
        source_df.alias("source"),
        "target.customer_id = source.customer_id"
    )
    .whenMatchedDelete(
        condition="source.status = 'INACTIVE'"
    )
    .whenMatchedUpdate(
        condition="source.status = 'ACTIVE'",
        set={
            "name": "source.name",
            "city": "source.city"
        }
    )
    .whenNotMatchedInsert(
        condition="source.status = 'ACTIVE'",
        values={
            "customer_id": "source.customer_id",
            "name": "source.name",
            "city": "source.city"
        }
    )
    .execute()
)
```

The PySpark API supports multiple conditional clauses. Make sure the conditions and clause ordering reflect the intended business rules.

## 10. Important MERGE Considerations

### A. Duplicate records in the source

Suppose the source contains multiple rows with the same `customer_id`.

```text
customer_id | name
------------|------
101         | Rahul
101         | Rohan
```

Both rows may match the same target record.

This can cause an ambiguous MERGE operation because Delta Lake cannot determine which source row should update the target.

**Solution:** Deduplicate the source before MERGE.

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy(F.col("updated_at").desc())
)

source_deduplicated = (
    source_df
    .withColumn(
        "row_num",
        F.row_number().over(window_spec)
    )
    .filter(F.col("row_num") == 1)
    .drop("row_num")
)
```

Use a deterministic tie-breaker if multiple rows have the same `updated_at`.

### B. NULL values in matching conditions

Normal equality does not match NULL values.

```sql
ON target.customer_id = source.customer_id
```

If both IDs are NULL, the condition does not evaluate to true.

If NULL-safe matching is required, use:

```sql
ON target.customer_id <=> source.customer_id
```

However, business keys should generally be non-NULL and unique.

### C. Concurrent MERGE operations

Delta Lake uses optimistic concurrency control.

If concurrent operations conflict, one operation may fail and need to be retried.

Do not assume every concurrent MERGE will succeed automatically.

### D. MERGE is not automatically idempotent

An operation is idempotent when running it repeatedly produces the same final result.

A carefully designed MERGE based on stable keys and deterministic source data can be idempotent.

However, operations involving counters, generated values, changing source data, or non-deterministic expressions may not be idempotent.

Design retry logic carefully.

### E. Performance considerations

MERGE can become expensive when large source and target datasets need to be compared.

Possible optimization techniques:

- Filter the source to relevant records.
- Deduplicate the source before MERGE.
- Use appropriate target partitioning or clustering.
- Keep file sizes manageable.
- Use data skipping effectively.
- Avoid unnecessary updates.
- Monitor query execution and file scanning.

Do not partition a table by a high-cardinality column just to improve MERGE performance.

## 11. MERGE vs UPDATE vs DELETE vs INSERT

| Operation | Purpose |
|---|---|
| INSERT | Add new records |
| UPDATE | Modify existing records |
| DELETE | Remove matching records |
| MERGE | Insert, update, and/or delete based on matching conditions |
| UPSERT | Update existing records and insert new records |

**Interview point:** MERGE is useful when source data contains both new and existing records and we need to synchronize them with a target table.

## 12. Real-World Data Engineering Use Cases

### Use case 1: Customer data synchronization

A source system sends customer changes every day.

- Existing customers: Update their details.
- New customers: Insert them.
- Inactive customers: Delete or mark them inactive according to business requirements.

### Use case 2: Incremental ETL pipeline

Instead of reloading the entire target table every day:

1. Read newly arrived or changed source records.
2. Apply data quality checks.
3. Deduplicate using business keys and update timestamps.
4. MERGE into the target Delta table.
5. Validate the results.

### Use case 3: CDC pipeline

Change Data Capture (CDC) provides information about source database changes.

Typical CDC operations include:

- INSERT
- UPDATE
- DELETE

A Delta MERGE operation can apply these changes to a target table.

The pipeline must correctly handle event ordering, duplicate events, and delete events.

## 13. Common Interview Questions

**Q1. What is MERGE in Delta Lake?**

MERGE is an operation that compares source and target data using a matching condition. It can update existing records, insert new records, and delete records based on specified conditions.

**Q2. What is UPSERT?**

UPSERT means updating an existing record or inserting a new record when no match exists. In Delta Lake, it is commonly implemented using MERGE.

**Q3. What is the difference between MERGE and UPDATE?**

UPDATE modifies existing matching records. MERGE can update matching records and insert or delete records based on additional conditions.

**Q4. What happens when multiple source records match one target record?**

The MERGE may fail because the update is ambiguous. Deduplicate the source data before executing MERGE.

**Q5. Does DELETE immediately remove data files?**

No. DELETE logically removes records through a new Delta transaction. The old data files may remain until they are eligible for physical removal through VACUUM.

**Q6. Can MERGE perform only INSERT operations?**

Yes. You can use `WHEN NOT MATCHED THEN INSERT` without defining an update clause.

**Q7. Can MERGE perform only UPDATE operations?**

Yes. You can define only a `WHEN MATCHED THEN UPDATE` clause.

**Q8. What is WHEN NOT MATCHED BY SOURCE?**

It handles target records that have no matching source record. It can be used to update or delete those target records when supported by the runtime.

**Q9. Why should we deduplicate source data before MERGE?**

Duplicate source keys can cause multiple source rows to match the same target row, resulting in ambiguous updates or incorrect results.

**Q10. Is MERGE automatically idempotent?**

No. It can be designed to be idempotent, but this depends on stable keys, deterministic source data, and safe update logic.

**Q11. How is MERGE used in CDC pipelines?**

CDC events are processed according to their operation type. MERGE can insert new records, update existing records, and delete records based on the incoming changes.

**Q12. What happens if a MERGE operation fails?**

A failed transaction does not become a partially committed table update. Delta Lake's transaction mechanism ensures readers see committed table versions. A retry may still be necessary, particularly for concurrency conflicts.

## 14. Quick Revision

| Concept | Remember |
|---|---|
| UPSERT | UPDATE + INSERT |
| MERGE | Synchronizes source and target |
| WHEN MATCHED | Source record matches target |
| WHEN NOT MATCHED | Source record has no target match |
| WHEN NOT MATCHED BY SOURCE | Target record has no source match |
| DELETE | Logically removes matching records |
| Duplicate source keys | Can cause ambiguous MERGE |
| NULL matching | Use `<=>` for NULL-safe equality |
| Concurrency | Optimistic concurrency control |
| Idempotency | Must be designed, not assumed |
| VACUUM | Physically removes eligible old files |

## 15. Self-Assessment

Before moving to the next topic, make sure you can answer:

- [ ] Explain UPSERT with an example.
- [ ] Write a MERGE SQL statement.
- [ ] Explain WHEN MATCHED and WHEN NOT MATCHED.
- [ ] Perform conditional UPDATE and DELETE.
- [ ] Explain WHEN NOT MATCHED BY SOURCE.
- [ ] Perform MERGE using PySpark.
- [ ] Explain why source deduplication is important.
- [ ] Explain how DELETE relates to VACUUM.
- [ ] Explain MERGE in an incremental ETL or CDC pipeline.
- [ ] Explain concurrency conflicts and idempotency.

**Interview takeaway:** MERGE is one of the most important Delta Lake operations for incremental data processing. Be comfortable writing it in SQL and PySpark, handling duplicate source records, and explaining how it supports UPSERT and CDC pipelines.