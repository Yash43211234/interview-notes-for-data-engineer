# Predicate Pushdown and Projection Pushdown in PySpark

## 1. What Are Pushdown Optimizations?

Pushdown optimizations reduce the amount of data Spark needs to read and process by moving supported operations closer to the data source.

Two important optimizations are:

1. **Predicate Pushdown:** Push filters closer to the data source.
2. **Projection Pushdown:** Read only the required columns from the data source.

Both can reduce I/O, memory usage, and processing time.

**Key idea:** Read less data and process fewer records whenever possible.

---

## 2. What Is Predicate Pushdown?

Predicate pushdown is an optimization in which Spark attempts to apply filter conditions at the data source instead of reading all records and filtering them afterward.

A predicate is simply a filtering condition.

Examples:
- `age > 25`
- `country = 'India'`
- `order_date >= '2026-01-01'`

### Without predicate pushdown

Imagine a dataset containing 10 million records.

You need only records where `country = 'India'`.

Without filter pushdown, the source may need to return all 10 million records, and Spark filters them afterward.

### With predicate pushdown

If the data source supports the filter, it can filter records before returning data to Spark.

For example, the source may return only the matching records.

This reduces the amount of data transferred and processed.

### PySpark example

```python
from pyspark.sql.functions import col

df = spark.read.parquet("/data/customers")

filtered_df = df.filter(
    col("country") == "India"
)
```

Spark can push this filter into the Parquet scan when supported.

### SQL example

```sql
SELECT *
FROM customers
WHERE country = 'India';
```

The optimizer may push the filter closer to the data source.

---

## 3. What Is Projection Pushdown?

Projection pushdown is an optimization in which Spark reads only the columns required by the query instead of reading every column.

A projection means selecting specific columns.

### Example

Suppose a customers table has 10 columns:

- customer_id
- customer_name
- age
- country
- email
- phone
- address
- city
- state
- created_date

You only need `customer_id`, `customer_name`, and `country`.

### Without projection pushdown

The source may read all 10 columns before Spark selects the required columns.

### With projection pushdown

For a supported columnar source such as Parquet, Spark can read only the required columns.

### PySpark example

```python
df = spark.read.parquet("/data/customers")

selected_df = df.select(
    "customer_id",
    "customer_name",
    "country"
)
```

This can reduce disk I/O and memory usage.

### SQL example

```sql
SELECT
    customer_id,
    customer_name,
    country
FROM customers;
```

Spark can push the required column selection into the source scan when supported.

---

## 4. Predicate Pushdown vs Projection Pushdown

| Feature | Predicate Pushdown | Projection Pushdown |
|---|---|---|
| Purpose | Reduce rows read | Reduce columns read |
| Operation | Filter | Select |
| Common PySpark method | `filter()` / `where()` | `select()` |
| Main benefit | Less data transferred and processed | Less data read and stored |
| Example | `country = 'India'` | Select `customer_id`, `country` |

**Easy way to remember:**

- Predicate pushdown → Fewer rows.
- Projection pushdown → Fewer columns.

---

## 5. How Do Both Work Together?

Suppose you have a large orders dataset with 20 columns and 50 million records.

Your requirement is:

- Only orders from 2026.
- Only `order_id`, `customer_id`, and `amount`.

### PySpark example

```python
from pyspark.sql.functions import col

df = spark.read.parquet("/data/orders")

result = (
    df.filter(col("order_date") >= "2026-01-01")
      .select(
          "order_id",
          "customer_id",
          "amount"
      )
)
```

Spark can apply:

1. Predicate pushdown to filter records using `order_date`.
2. Projection pushdown to read only the required columns, including the column needed for filtering.

The source scan may therefore read fewer columns and return fewer records.

**Important:** The actual optimization depends on the data source, file format, query expressions, and connector capabilities.

---

## 6. How Does Spark Implement Pushdown?

Spark SQL uses the Catalyst optimizer to optimize query plans.

During optimization, Spark can:

- Push supported filters toward the data source.
- Push required column selection toward the data source.
- Simplify expressions.
- Remove unnecessary operations.

For supported data sources, Spark's physical plan can show filters and column selections being applied at the scan.

### Example

```python
df = spark.read.parquet("/data/orders")

result = (
    df.filter(col("country") == "India")
      .select("order_id", "amount")
)

result.explain("formatted")
```

Look for a scan node such as:

```text
FileScan parquet
```

Depending on Spark version and the source, you may see details such as:

```text
PartitionFilters: [...]
PushedFilters: [...]
ReadSchema: struct<order_id:bigint,amount:double>
```

Interpretation:

| Plan detail | Meaning |
|---|---|
| `PartitionFilters` | Filters applied to storage partition columns |
| `PushedFilters` | Filters passed to the data source |
| `ReadSchema` | Columns requested from the source |

**Important:** These fields can vary by source and Spark version. A pushed filter does not guarantee that every part of the filtering operation is executed by the source.

---

## 7. Predicate Pushdown vs Partition Pruning

These concepts are related but different.

### Predicate pushdown

Spark passes a supported filter condition to the data source so that fewer records need to be returned or processed.

Example:

```python
df.filter(col("country") == "India")
```

### Partition pruning

Spark skips entire storage partitions that cannot match the filter condition.

Suppose data is stored like this:

```text
orders/
    country=India/
    country=USA/
    country=UK/
```

Query:

```python
df = spark.read.parquet("/data/orders")

result = df.filter(
    col("country") == "India"
)
```

If the source recognizes the filter on the partition column, Spark can read only the India partition.

### Difference

| Predicate Pushdown | Partition Pruning |
|---|---|
| Pushes filters toward the data source | Skips irrelevant storage partitions |
| Can work on non-partition columns when supported | Requires a filter on suitable partition columns |
| Reduces records read or processed | Avoids reading entire storage partitions |

**Interview point:** Partition pruning avoids scanning irrelevant partitions, while predicate pushdown applies supported filters closer to the data source.

---

## 8. Predicate Pushdown and Projection Pushdown by File Format

Pushdown support depends on the data source and connector.

| Format / Source | Predicate Pushdown | Projection Pushdown |
|---|---|---|
| Parquet | Supported for eligible filters | Supported |
| ORC | Supported for eligible filters | Supported |
| CSV | Generally limited compared with columnar formats | Generally cannot skip individual columns during file reading in the same way |
| JSON | Generally limited compared with columnar formats | Generally cannot skip individual fields during file reading in the same way |
| JDBC | Depends on the database and connector | Depends on the database and connector |

Columnar formats such as Parquet and ORC are particularly effective for projection pushdown because they store data in a column-oriented layout.

**Important:** Do not assume that every filter or expression can be pushed down. Check the physical plan and source documentation.

---

## 9. Why Are Pushdown Optimizations Important?

### Benefits

1. Reduces unnecessary data reading.
2. Reduces I/O.
3. Reduces data transfer between source and Spark.
4. Reduces memory pressure.
5. Can reduce processing time.
6. Improves performance for large datasets.

### Example

Suppose a Parquet dataset has:

- 50 million records.
- 20 columns.

Your query requires:

- 2 million matching records.
- Only 3 columns.

With effective predicate and projection pushdown, Spark can avoid reading unnecessary data and columns.

The actual performance improvement depends on file layout, compression, statistics, and the selectivity of the filter.

---

## 10. Common Mistakes

1. Assuming every filter is pushed down.
2. Assuming predicate pushdown and partition pruning are the same.
3. Assuming projection pushdown means Spark never reads columns used in filter conditions.
4. Expecting CSV to provide the same column-level reading benefits as Parquet.
5. Assuming `filter()` always reduces physical data read.
6. Ignoring the execution plan.
7. Selecting every column when only a few are needed.
8. Using unsupported expressions and assuming they will be pushed down.

---

## 11. Interview Questions

**Q1. What is predicate pushdown?**

Predicate pushdown is an optimization where Spark pushes supported filter conditions closer to the data source to reduce the amount of data returned and processed.

**Q2. What is projection pushdown?**

Projection pushdown means reading only the required columns instead of reading all columns from the source.

**Q3. What is the difference between predicate and projection pushdown?**

Predicate pushdown reduces rows, while projection pushdown reduces columns.

**Q4. Which file formats support projection pushdown effectively?**

Columnar formats such as Parquet and ORC support efficient column-level reading.

**Q5. Is predicate pushdown the same as partition pruning?**

No. Predicate pushdown passes filters to the data source, while partition pruning skips entire storage partitions that cannot match the filter.

**Q6. How can you check whether predicate pushdown is happening?**

Use:

```python
df.explain("formatted")
```

Inspect the scan details for pushed filters and partition filters.

**Q7. How can you check projection pushdown?**

Inspect the scan's `ReadSchema` in the physical plan to see which columns are requested from the source.

**Q8. Does Spark always push down filters?**

No. It depends on the source, connector, supported expressions, and query plan.

**Q9. Why is Parquet useful for projection pushdown?**

Parquet stores data in a columnar format, allowing Spark to read selected columns without reading every column in the dataset.

**Q10. Can predicate and projection pushdown work together?**

Yes. Spark can push supported filters and required column selections toward the source in the same query.

---

## 12. Quick Revision

| Concept | Remember |
|---|---|
| Predicate | Filtering condition |
| Predicate pushdown | Push filters closer to the source |
| Projection | Selecting columns |
| Projection pushdown | Read only required columns |
| Catalyst | Spark SQL query optimizer |
| Partition pruning | Skip irrelevant storage partitions |
| `PushedFilters` | Filters passed to the source |
| `PartitionFilters` | Filters on storage partition columns |
| `ReadSchema` | Columns requested from the source |
| Parquet | Efficient columnar format |

### One-line revision

- Predicate pushdown = Read fewer rows.
- Projection pushdown = Read fewer columns.
- Partition pruning = Skip irrelevant storage partitions.

---

## 13. Self-Check

- [ ] Can I explain predicate pushdown?
- [ ] Can I explain projection pushdown?
- [ ] Can I differentiate both?
- [ ] Can I explain how Catalyst helps?
- [ ] Can I differentiate pushdown from partition pruning?
- [ ] Can I identify pushed filters in an execution plan?
- [ ] Can I explain why Parquet is effective for projection pushdown?
- [ ] Can I explain why pushdown is not guaranteed for every source or expression?

**Final takeaway:** Predicate pushdown reduces unnecessary rows, projection pushdown reduces unnecessary columns, and partition pruning skips irrelevant storage partitions. Together, these optimizations help Spark read and process less data.