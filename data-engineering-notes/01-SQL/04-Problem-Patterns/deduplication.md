# SQL Deduplication — Interview Notes

## 1. What Is Deduplication?

**Deduplication** means identifying and removing duplicate records while keeping the correct record according to a defined business rule.

Duplicates can occur because of:
- Repeated file ingestion.
- Duplicate source-system records.
- Retry mechanisms in data pipelines.
- Multiple records for the same business key.
- Incorrect joins or data integration.

**Data Engineering example:** A daily pipeline loads the same order twice. Deduplication ensures that only the intended order record remains.

> Interview rule: Before removing duplicates, define what makes a record a duplicate and which record should be retained.

---

## 2. Identify Duplicates Using `GROUP BY`

Suppose we have this table:

### `orders`

| order_id | customer_id | amount | updated_at |
|---:|---:|---:|---|
| 101 | 1 | 500 | 2026-01-01 10:00 |
| 102 | 2 | 700 | 2026-01-01 11:00 |
| 101 | 1 | 500 | 2026-01-01 10:00 |
| 103 | 3 | 900 | 2026-01-02 09:00 |
| 102 | 2 | 750 | 2026-01-03 12:00 |

Find duplicate `order_id` values:

```sql
SELECT
    order_id,
    COUNT(*) AS record_count
FROM orders
GROUP BY order_id
HAVING COUNT(*) > 1;
```

Expected result:

| order_id | record_count |
|---:|---:|
| 101 | 2 |
| 102 | 2 |

**Important:** This query identifies duplicate keys. It does not remove duplicate records.

Also, `order_id` is treated as the business key here. If the business definition of a duplicate uses multiple columns, group by those columns instead.

---

## 3. Remove Duplicates Using `ROW_NUMBER()`

This is one of the most important SQL interview patterns.

Suppose we want to keep the **latest record for each `order_id`**.

```sql
WITH ranked_orders AS (
    SELECT
        order_id,
        customer_id,
        amount,
        updated_at,
        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY updated_at DESC
        ) AS rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    amount,
    updated_at
FROM ranked_orders
WHERE rn = 1;
```

Expected result:

| order_id | customer_id | amount | updated_at |
|---:|---:|---:|---|
| 101 | 1 | 500 | 2026-01-01 10:00 |
| 102 | 2 | 750 | 2026-01-03 12:00 |
| 103 | 3 | 900 | 2026-01-02 09:00 |

### How it works

1. `PARTITION BY order_id` groups records belonging to the same order.
2. `ORDER BY updated_at DESC` places the newest record first.
3. `ROW_NUMBER()` assigns a unique number within each group.
4. `WHERE rn = 1` keeps only the first record from each group.

**Interview point:** `ROW_NUMBER()` is usually the right choice when you need exactly one retained row per business key.

### Handle tied timestamps

If two records have the same `updated_at`, the chosen record may be nondeterministic.

Use a stable tie-breaker, such as a unique ingestion ID:

```sql
ROW_NUMBER() OVER (
    PARTITION BY order_id
    ORDER BY updated_at DESC, ingestion_id DESC
) AS rn
```

Choose a tie-breaker that follows the business rule.

---

## 4. Remove Exact Duplicate Rows

Sometimes every relevant column is identical, and you want one copy of each distinct combination.

```sql
SELECT DISTINCT
    order_id,
    customer_id,
    amount,
    updated_at
FROM orders;
```

This removes duplicate combinations of the selected columns from the **query result**.

However, it does not delete duplicate rows from the underlying table.

### Important difference

- `DISTINCT`: Removes repeated combinations from the output.
- `ROW_NUMBER()`: Lets you select one row based on a ranking rule.
- `DELETE`: Physically removes records from a table, subject to database support and the deletion logic.

Use `DISTINCT` only when the selected columns represent the complete definition of an exact duplicate.

---

## 5. Find and Delete Duplicate Records

The exact deletion syntax depends on the database and its table structure. A safe approach is to identify rows to delete before running a `DELETE`.

Assume the table has a unique `record_id` column. We want to retain the latest record for each `order_id`.

First, preview the records that would be removed:

```sql
WITH ranked_orders AS (
    SELECT
        record_id,
        order_id,
        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY updated_at DESC, record_id DESC
        ) AS rn
    FROM orders
)
SELECT *
FROM ranked_orders
WHERE rn > 1;
```

After verifying the results and taking appropriate precautions, use a database-supported delete pattern. For example, in PostgreSQL:

```sql
WITH ranked_orders AS (
    SELECT
        record_id,
        ROW_NUMBER() OVER (
            PARTITION BY order_id
            ORDER BY updated_at DESC, record_id DESC
        ) AS rn
    FROM orders
)
DELETE FROM orders AS o
USING ranked_orders AS r
WHERE o.record_id = r.record_id
  AND r.rn > 1;
```

This retains one row per `order_id`, choosing the row with the latest `updated_at`, and then the greatest `record_id` when timestamps tie.

**Safety:** Run the preview query first. Verify the business key, tie-breaker, and intended records. Use a transaction or backup where appropriate.

---

## 6. Deduplicate by Multiple Columns

Suppose a customer can have multiple transactions, but a transaction should be unique for the combination of `customer_id` and `transaction_date`.

```sql
WITH ranked_transactions AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id, transaction_date
            ORDER BY created_at DESC, transaction_id DESC
        ) AS rn
    FROM transactions
)
SELECT *
FROM ranked_transactions
WHERE rn = 1;
```

Here, duplicates are defined by the combination of:

- `customer_id`
- `transaction_date`

The latest record is retained, with `transaction_id` as the tie-breaker.

**Remember:** `PARTITION BY` should contain the columns that define a duplicate according to the business requirement.

---

## 7. Deduplication in Data Engineering Pipelines

In a data pipeline, deduplication often happens after ingestion and before data is written to a curated or reporting layer.

A common workflow is:

1. **Ingest:** Load source records into a raw or Bronze layer.
2. **Identify the business key:** Determine which columns identify the same entity or event.
3. **Rank records:** Use `ROW_NUMBER()` to order records by update time or source priority.
4. **Retain valid records:** Keep the intended record, often `rn = 1`.
5. **Write the result:** Store the deduplicated data in the next layer.
6. **Validate:** Check row counts, duplicate keys, null keys, and rejected records.

### Example: PySpark DataFrame

```python
from pyspark.sql import Window
from pyspark.sql.functions import row_number, col

window_spec = (
    Window
    .partitionBy("order_id")
    .orderBy(
        col("updated_at").desc(),
        col("ingestion_id").desc()
    )
)

deduplicated_df = (
    orders_df
    .withColumn("rn", row_number().over(window_spec))
    .filter(col("rn") == 1)
    .drop("rn")
)
```

This retains one record per `order_id`, prioritizing the latest `updated_at` and then the greatest `ingestion_id`.

If your source can contain null timestamps or null business keys, define how those cases should be handled rather than relying on defaults.

### Deduplication versus idempotency

These concepts are related but different:

- **Deduplication:** Identifies repeated records and retains the intended record.
- **Idempotency:** Ensures that rerunning the same operation does not incorrectly change the final result or introduce additional effects.

A pipeline may deduplicate records and still fail to be idempotent if its writes append duplicates on every rerun.

---

## 8. Common Mistakes

### Mistake 1: Using `DISTINCT` without understanding the business key

Two rows can have the same `order_id` but different amounts. `DISTINCT` will keep both because the full selected combinations differ.

### Mistake 2: Using `ROW_NUMBER()` without deterministic ordering

If several records tie on the ordering columns, the retained row may vary. Add a stable tie-breaker when the business rule allows it.

### Mistake 3: Using `RANK()` instead of `ROW_NUMBER()`

`RANK()` can assign the same rank to tied rows. Filtering on `rank = 1` may retain multiple rows per key.

### Mistake 4: Deleting before checking

A wrong `PARTITION BY` can remove valid business records. Preview the rows marked for deletion first.

### Mistake 5: Assuming a primary key exists

Raw files and staging tables may not have a unique row identifier. If you need to delete individual duplicate rows, first establish a reliable way to identify each physical record.

### Mistake 6: Ignoring late-arriving updates

A record with an older event timestamp might arrive later than a newer record. Decide whether the ranking should use event time, update time, ingestion time, or source-system priority.

---

## 9. Common Interview Questions

**Q1. How do you find duplicate records in SQL?**

Use `GROUP BY` with `HAVING COUNT(*) > 1` on the columns that define a duplicate.

**Q2. How do you remove duplicates while keeping the latest record?**

Use `ROW_NUMBER()` partitioned by the business key and ordered by the latest timestamp first. Keep `rn = 1`.

**Q3. What is the difference between `DISTINCT` and `ROW_NUMBER()`?**

`DISTINCT` removes repeated selected-column combinations from the result. `ROW_NUMBER()` ranks rows and allows a specific record to be retained according to an ordering rule.

**Q4. Why do we use `PARTITION BY` during deduplication?**

It creates separate groups for each business key, so ranking starts again within each group.

**Q5. How would you deduplicate millions of records?**

Define the business key and retention rule, process data in a scalable engine such as Spark when appropriate, reduce unnecessary columns, use suitable partitioning, and validate the output. Avoid collecting the entire dataset to the driver.

**Q6. How do you prevent duplicates when a pipeline reruns?**

Use an idempotent write strategy, stable business keys, and an appropriate upsert or merge mechanism where supported. Deduplication may be one part of the solution.

**Q7. What happens if the ordering timestamps are equal?**

Without a tie-breaker, the selected record may be nondeterministic. Add a stable secondary ordering column if available.

**Q8. Can `GROUP BY` delete duplicate records?**

No. It aggregates records into groups. It can identify duplicate keys, but deletion requires a separate operation.

---

## 10. Practice Questions

Try solving these without looking at the examples above.

1. Find all `customer_id` values appearing more than once in a customer table.
2. Keep the latest customer record for each `customer_id`.
3. Remove exact duplicate combinations of `product_id`, `store_id`, and `sale_date` from a query result.
4. Find duplicate transactions based on `transaction_id`, retaining the record with the greatest `updated_at`.
5. Identify duplicate business keys where timestamps tie, and propose a deterministic tie-breaker.
6. Explain how you would prevent duplicate rows when a daily file is accidentally ingested twice.
7. Explain how deduplication and idempotency differ in a Spark or Delta Lake pipeline.

---

## 11. Quick Revision

| Requirement | Technique |
|---|---|
| Find duplicate keys | `GROUP BY` + `HAVING COUNT(*) > 1` |
| Remove exact duplicates from query output | `SELECT DISTINCT` |
| Keep the latest row per key | `ROW_NUMBER()` + `rn = 1` |
| Preview rows to remove | `ROW_NUMBER()` + `rn > 1` |
| Deduplicate using multiple columns | `PARTITION BY col1, col2, ...` |
| Make the retained row deterministic | Add a suitable tie-breaker |
| Prevent duplicates on reruns | Idempotent pipeline and appropriate write strategy |

### The most important pattern

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY business_key
            ORDER BY updated_at DESC, unique_id DESC
        ) AS rn
    FROM source_table
)
SELECT *
FROM ranked
WHERE rn = 1;
```

Replace `business_key`, `updated_at`, and `unique_id` with the appropriate columns in your dataset.

---

## 12. Self-Check

Before considering this topic complete, make sure you can:

- [ ] Find duplicate keys using `GROUP BY`.
- [ ] Explain when `DISTINCT` is sufficient.
- [ ] Deduplicate using `ROW_NUMBER()`.
- [ ] Keep the latest record for each business key.
- [ ] Handle ties with a deterministic ordering rule.
- [ ] Explain deduplication versus idempotency.
- [ ] Describe how you would apply deduplication in a data pipeline.

**Interview takeaway:** The key is not simply removing repeated rows. It is defining the business key, choosing which record to retain, and ensuring the result remains correct when data arrives late or a pipeline reruns.