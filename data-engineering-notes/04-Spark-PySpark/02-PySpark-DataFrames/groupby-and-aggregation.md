# GroupBy and Aggregation in PySpark

## 1. Overview

`groupBy()` is used to group rows based on one or more columns.

Aggregation functions calculate summary values for each group, such as:

- Total sales
- Average salary
- Number of employees
- Minimum and maximum values

In data engineering, `groupBy()` is commonly used for reporting, data analysis, and data transformation.

Import the required functions:

```python
from pyspark.sql import functions as F
```

---

## 2. Sample Data

Consider an orders DataFrame:

```python
data = [
    (1, "Alice", "Electronics", 5000),
    (2, "Bob", "Clothing", 2000),
    (3, "Alice", "Electronics", 3000),
    (4, "David", "Clothing", 1500),
    (5, "Bob", "Electronics", 4000)
]

columns = ["order_id", "customer", "category", "amount"]

df = spark.createDataFrame(data, columns)
```

Sample records:

| order_id | customer | category | amount |
|---:|---|---|---:|
| 1 | Alice | Electronics | 5000 |
| 2 | Bob | Clothing | 2000 |
| 3 | Alice | Electronics | 3000 |
| 4 | David | Clothing | 1500 |
| 5 | Bob | Electronics | 4000 |

---

## 3. Basic groupBy()

Syntax:

```python
df.groupBy("column_name")
```

Example: Group orders by customer.

```python
df.groupBy("customer").count().show()
```

Expected output:

| customer | count |
|---|---:|
| Alice | 2 |
| Bob | 2 |
| David | 1 |

**Important:** `groupBy()` alone does not calculate a final result. You generally apply an aggregation function after grouping.

---

## 4. Common Aggregation Functions

### 4.1 count()

Counts the number of rows in each group.

```python
df.groupBy("customer").count()
```

### 4.2 sum()

Calculates the total value.

```python
df.groupBy("customer").agg(
    F.sum("amount").alias("total_amount")
)
```

Expected output:

| customer | total_amount |
|---|---:|
| Alice | 8000 |
| Bob | 6000 |
| David | 1500 |

### 4.3 avg() / mean()

Calculates the average value.

```python
df.groupBy("customer").agg(
    F.avg("amount").alias("average_amount")
)
```

### 4.4 min()

Finds the minimum value.

```python
df.groupBy("customer").agg(
    F.min("amount").alias("minimum_amount")
)
```

### 4.5 max()

Finds the maximum value.

```python
df.groupBy("customer").agg(
    F.max("amount").alias("maximum_amount")
)
```

### 4.6 Other Useful Aggregation Functions

```python
F.countDistinct("customer")
F.collect_list("category")
F.collect_set("category")
F.first("amount")
F.last("amount")
```

- `countDistinct()` counts unique non-null values.
- `collect_list()` collects values into an array, including duplicates.
- `collect_set()` collects distinct values into an array.
- `first()` and `last()` return a value from the group; without explicit ordering, do not assume they represent the earliest or latest record.

---

## 5. Multiple Aggregations Using agg()

You can calculate multiple aggregations in one `groupBy()` operation.

Example: Calculate total, average, minimum, maximum, and order count for each customer.

```python
df.groupBy("customer").agg(
    F.count("*").alias("total_orders"),
    F.sum("amount").alias("total_amount"),
    F.avg("amount").alias("average_amount"),
    F.min("amount").alias("minimum_amount"),
    F.max("amount").alias("maximum_amount")
).show()
```

Expected output:

| customer | total_orders | total_amount | average_amount | minimum_amount | maximum_amount |
|---|---:|---:|---:|---:|---:|
| Alice | 2 | 8000 | 4000 | 3000 | 5000 |
| Bob | 2 | 6000 | 3000 | 2000 | 4000 |
| David | 1 | 1500 | 1500 | 1500 | 1500 |

**Interview point:** `agg()` allows multiple aggregation expressions to be applied after grouping.

---

## 6. GroupBy Multiple Columns

You can group data using more than one column.

Example: Calculate total sales for each customer and category.

```python
df.groupBy("customer", "category").agg(
    F.sum("amount").alias("total_sales")
).show()
```

Expected output:

| customer | category | total_sales |
|---|---|---:|
| Alice | Electronics | 8000 |
| Bob | Clothing | 2000 |
| Bob | Electronics | 4000 |
| David | Clothing | 1500 |

**Important:** A separate group is created for every unique combination of grouping-column values.

---

## 7. GroupBy with countDistinct()

Use `countDistinct()` when you need to count unique values.

Example: Count unique customers in each category.

```python
df.groupBy("category").agg(
    F.countDistinct("customer").alias("unique_customers")
).show()
```

Expected output:

| category | unique_customers |
|---|---:|
| Electronics | 2 |
| Clothing | 2 |

Difference:

```python
F.count("customer")
F.countDistinct("customer")
F.count("*")
```

- `count("customer")` counts non-null customer values.
- `countDistinct("customer")` counts unique non-null customer values.
- `count("*")` counts all rows, including rows containing null values in other columns.

---

## 8. GroupBy with Conditional Aggregation

Conditional aggregation calculates summary values based on a condition.

Example: Calculate total electronics sales for each customer.

```python
df.groupBy("customer").agg(
    F.sum(
        F.when(
            F.col("category") == "Electronics",
            F.col("amount")
        ).otherwise(0)
    ).alias("electronics_sales")
)
```

Another example: Count electronics orders for each customer.

```python
df.groupBy("customer").agg(
    F.sum(
        F.when(
            F.col("category") == "Electronics",
            1
        ).otherwise(0)
    ).alias("electronics_orders")
)
```

**Important:** `when()` creates a conditional expression. `sum()` aggregates the resulting values within each group.

---

## 9. Difference Between groupBy() and agg()

| groupBy() | agg() |
|---|---|
| Creates groups based on columns | Calculates aggregation expressions |
| Defines grouping columns | Defines summary calculations |
| Usually followed by an aggregation | Used after grouping or directly on a DataFrame |

Example:

```python
df.groupBy("customer").agg(
    F.sum("amount").alias("total_amount")
)
```

Here:

- `groupBy("customer")` creates customer groups.
- `agg()` calculates the total amount for each group.

You can also aggregate the entire DataFrame without grouping:

```python
df.agg(
    F.sum("amount").alias("total_sales"),
    F.avg("amount").alias("average_sales")
)
```

This returns one summary row for the entire DataFrame.

---

## 10. Filtering Before and After Aggregation

### 10.1 Filter Before Grouping

Use `filter()` or `where()` to remove rows before aggregation.

Example: Calculate total sales by customer for orders above 2000.

```python
df.filter(
    F.col("amount") > 2000
).groupBy("customer").agg(
    F.sum("amount").alias("total_sales")
)
```

### 10.2 Filter After Grouping

Use `filter()` on the aggregated result to filter groups based on calculated values.

Example: Find customers whose total sales exceed 5000.

```python
result = df.groupBy("customer").agg(
    F.sum("amount").alias("total_sales")
)

result.filter(
    F.col("total_sales") > 5000
).show()
```

**SQL comparison:**

| PySpark | SQL |
|---|---|
| `filter()` before grouping | `WHERE` |
| `filter()` after aggregation | `HAVING` |

---

## 11. Handling Null Values in Aggregation

Consider a DataFrame containing null amounts.

```python
data = [
    ("Alice", 5000),
    ("Alice", None),
    ("Bob", 2000),
    ("Bob", None)
]

df = spark.createDataFrame(data, ["customer", "amount"])
```

Example:

```python
df.groupBy("customer").agg(
    F.count("*").alias("row_count"),
    F.count("amount").alias("non_null_count"),
    F.sum("amount").alias("total_amount"),
    F.avg("amount").alias("average_amount")
).show()
```

Expected behavior:

- `count("*")` counts all rows.
- `count("amount")` ignores null amounts.
- `sum("amount")` ignores null amounts.
- `avg("amount")` ignores null amounts.

If all values in a group are null, `sum()` and `avg()` return null.

**Important:** Do not assume that all aggregation functions handle null values in the same way.

---

## 12. Practical Data Engineering Example

Suppose you receive daily order data and need to generate customer-level sales summaries.

```python
customer_summary = (
    df
    .groupBy("customer")
    .agg(
        F.count("*").alias("total_orders"),
        F.sum("amount").alias("total_sales"),
        F.avg("amount").alias("average_order_value"),
        F.max("amount").alias("highest_order"),
        F.min("amount").alias("lowest_order")
    )
)
```

This produces a summary DataFrame that can be written into a reporting table or used in downstream transformations.

---

## 13. Performance Considerations

`groupBy()` is an expensive operation in distributed processing because it commonly requires a shuffle.

A shuffle moves data between partitions so that records belonging to the same group can be processed together.

Important points:

1. **Shuffle:** Grouping data often requires data movement across executors.
2. **High-cardinality columns:** Grouping by columns with many unique values can create many groups.
3. **Data skew:** If a few grouping keys contain a very large percentage of records, some tasks may take much longer than others.
4. **Filter early:** Apply appropriate filters before grouping to reduce the amount of data processed.
5. **Avoid unnecessary columns:** Select only the columns required for the aggregation.
6. **Use built-in functions:** Prefer Spark SQL functions over Python UDFs for common aggregations.

Example:

```python
result = (
    df
    .select("customer", "amount")
    .filter(F.col("amount").isNotNull())
    .groupBy("customer")
    .agg(F.sum("amount").alias("total_sales"))
)
```

---

## 14. Common Interview Questions

**Q1. What is `groupBy()` in PySpark?**

It groups rows based on one or more columns so that aggregation functions can calculate summary values for each group.

**Q2. What is the difference between `count()` and `countDistinct()`?**

- `count()` counts values or rows depending on the expression.
- `countDistinct()` counts unique non-null values.

**Q3. Can we perform multiple aggregations in one groupBy operation?**

Yes. Use `agg()` with multiple aggregation expressions.

**Q4. Can we use groupBy without an aggregation function?**

You can create a grouped DataFrame using `groupBy()`, but to obtain a result, you generally apply an aggregation such as `count()` or `agg()`.

**Q5. What happens to null values during aggregation?**

Most standard numeric aggregations, such as `sum()`, `avg()`, `min()`, and `max()`, ignore null values. `count("*")` counts all rows, while `count(column)` counts only non-null values.

**Q6. What is the difference between filtering before and after aggregation?**

Filtering before aggregation removes individual rows. Filtering after aggregation removes groups based on calculated summary values.

**Q7. Why is groupBy expensive in Spark?**

Because it commonly requires a shuffle to bring records with the same grouping keys together across partitions.

**Q8. What is data skew in groupBy?**

Data skew occurs when some grouping keys have significantly more records than others, causing uneven workload across partitions.

**Q9. Can we group by multiple columns?**

Yes.

```python
df.groupBy("customer", "category").sum("amount")
```

**Q10. What is the difference between `collect_list()` and `collect_set()`?**

- `collect_list()` preserves duplicate values.
- `collect_set()` returns distinct values.

Neither function guarantees a particular element order after a distributed aggregation.

---

## 15. Practice Questions

Using the sample orders DataFrame:

1. Count the total number of orders for each customer.
2. Calculate total sales for each customer.
3. Find the average order amount for each category.
4. Find the maximum order amount for each customer.
5. Calculate total sales by customer and category.
6. Count unique customers in each category.
7. Find customers whose total sales exceed 5000.
8. Count electronics orders for each customer.
9. Calculate total and average sales in one aggregation.
10. Find the number of orders for each category where the amount is greater than 2000.

---

## 16. Quick Revision

| Function | Purpose |
|---|---|
| `groupBy()` | Groups rows |
| `agg()` | Performs multiple aggregations |
| `count()` | Counts rows or non-null values |
| `sum()` | Calculates total |
| `avg()` / `mean()` | Calculates average |
| `min()` | Finds minimum |
| `max()` | Finds maximum |
| `countDistinct()` | Counts unique values |
| `collect_list()` | Collects values including duplicates |
| `collect_set()` | Collects distinct values |
| `filter()` | Filters rows or aggregated results |
| `when()` | Creates conditional expressions |

**Remember these three concepts:**

1. `groupBy()` defines how data is grouped.
2. `agg()` defines what calculations are performed.
3. `filter()` can be applied before or after aggregation.

---

## 17. Self-Assessment

- [ ] Explain how `groupBy()` works.
- [ ] Use `count()`, `sum()`, `avg()`, `min()`, and `max()`.
- [ ] Perform multiple aggregations using `agg()`.
- [ ] Group by multiple columns.
- [ ] Use `countDistinct()`.
- [ ] Perform conditional aggregation.
- [ ] Filter data before and after aggregation.
- [ ] Explain null handling in aggregation.
- [ ] Explain why `groupBy()` causes a shuffle.
- [ ] Explain data skew and its impact.

**Interview readiness:** You should be able to write a customer-level or category-level aggregation from scratch and explain why Spark may need a shuffle to execute it.