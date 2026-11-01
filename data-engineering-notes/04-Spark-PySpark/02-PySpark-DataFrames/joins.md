# Joins in PySpark

## 1. Overview

Joins are used to combine data from two or more DataFrames based on a related column or condition.

In data engineering, joins are commonly used to:

- Combine customer and order data.
- Enrich datasets with reference data.
- Identify matching and non-matching records.
- Build analytical and reporting tables.
- Combine data from different sources.

Import the required functions:

```python
from pyspark.sql import functions as F
```

---

## 2. Sample DataFrames

Consider two DataFrames: `customers` and `orders`.

### Customers DataFrame

```python
customers_data = [
    (1, "Alice", "Kolkata"),
    (2, "Bob", "Delhi"),
    (3, "Charlie", "Mumbai"),
    (4, "David", "Pune")
]

customers = spark.createDataFrame(
    customers_data,
    ["customer_id", "customer_name", "city"]
)
```

| customer_id | customer_name | city |
|---:|---|---|
| 1 | Alice | Kolkata |
| 2 | Bob | Delhi |
| 3 | Charlie | Mumbai |
| 4 | David | Pune |

### Orders DataFrame

```python
orders_data = [
    (101, 1, 5000),
    (102, 2, 3000),
    (103, 1, 2000),
    (104, 5, 4000)
]

orders = spark.createDataFrame(
    orders_data,
    ["order_id", "customer_id", "amount"]
)
```

| order_id | customer_id | amount |
|---:|---:|---:|
| 101 | 1 | 5000 |
| 102 | 2 | 3000 |
| 103 | 1 | 2000 |
| 104 | 5 | 4000 |

Notice:

- Customer IDs 1 and 2 have matching orders.
- Customer ID 3 and 4 have no orders.
- Customer ID 5 has an order but does not exist in the customers DataFrame.

---

## 3. Basic Join Syntax

```python
df1.join(
    df2,
    condition,
    how="join_type"
)
```

Example:

```python
result = customers.join(
    orders,
    on="customer_id",
    how="inner"
)
```

Parameters:

- `df2`: DataFrame to join.
- `on`: Common column or join condition.
- `how`: Type of join.

---

## 4. Types of Joins

### 4.1 Inner Join

Returns only records that have matching keys in both DataFrames.

```python
result = customers.join(
    orders,
    on="customer_id",
    how="inner"
)
```

Expected matching customer IDs: 1 and 2.

| customer_id | customer_name | city | order_id | amount |
|---:|---|---|---:|---:|
| 1 | Alice | Kolkata | 101 | 5000 |
| 1 | Alice | Kolkata | 103 | 2000 |
| 2 | Bob | Delhi | 102 | 3000 |

**Use case:** Get customer details for customers who have placed orders.

### 4.2 Left Join

Returns all records from the left DataFrame and matching records from the right DataFrame.

If there is no match, right-side columns contain null values.

```python
result = customers.join(
    orders,
    on="customer_id",
    how="left"
)
```

Expected output:

| customer_id | customer_name | city | order_id | amount |
|---:|---|---|---:|---:|
| 1 | Alice | Kolkata | 101 | 5000 |
| 1 | Alice | Kolkata | 103 | 2000 |
| 2 | Bob | Delhi | 102 | 3000 |
| 3 | Charlie | Mumbai | null | null |
| 4 | David | Pune | null | null |

**Use case:** Find all customers, including customers who have never placed an order.

### 4.3 Right Join

Returns all records from the right DataFrame and matching records from the left DataFrame.

```python
result = customers.join(
    orders,
    on="customer_id",
    how="right"
)
```

Expected output:

| customer_id | customer_name | city | order_id | amount |
|---:|---|---|---:|---:|
| 1 | Alice | Kolkata | 101 | 5000 |
| 2 | Bob | Delhi | 102 | 3000 |
| 1 | Alice | Kolkata | 103 | 2000 |
| 5 | null | null | 104 | 4000 |

**Use case:** Keep every order, even if the corresponding customer record is missing.

### 4.4 Full Outer Join

Returns all records from both DataFrames.

- Matching records are combined.
- Non-matching records from either side are retained.
- Missing columns are filled with null values.

```python
result = customers.join(
    orders,
    on="customer_id",
    how="full_outer"
)
```

Expected output:

| customer_id | customer_name | city | order_id | amount |
|---:|---|---|---:|---:|
| 1 | Alice | Kolkata | 101 | 5000 |
| 1 | Alice | Kolkata | 103 | 2000 |
| 2 | Bob | Delhi | 102 | 3000 |
| 3 | Charlie | Mumbai | null | null |
| 4 | David | Pune | null | null |
| 5 | null | null | 104 | 4000 |

**Use case:** Compare two datasets and identify matching and unmatched records.

### 4.5 Left Semi Join

Returns only rows from the left DataFrame that have a match in the right DataFrame.

It returns columns only from the left DataFrame.

```python
result = customers.join(
    orders,
    on="customer_id",
    how="left_semi"
)
```

Expected output:

| customer_id | customer_name | city |
|---:|---|---|
| 1 | Alice | Kolkata |
| 2 | Bob | Delhi |

**Use case:** Find customers who have placed at least one order.

### 4.6 Left Anti Join

Returns rows from the left DataFrame that do not have a match in the right DataFrame.

It returns columns only from the left DataFrame.

```python
result = customers.join(
    orders,
    on="customer_id",
    how="left_anti"
)
```

Expected output:

| customer_id | customer_name | city |
|---:|---|---|
| 3 | Charlie | Mumbai |
| 4 | David | Pune |

**Use case:** Find customers who have never placed an order.

---

## 5. Join Types Quick Comparison

| Join type | Returned records |
|---|---|
| Inner | Matching records from both sides |
| Left | All left records and matching right records |
| Right | All right records and matching left records |
| Full Outer | All records from both sides |
| Left Semi | Matching records from the left only |
| Left Anti | Non-matching records from the left only |

---

## 6. Joining on Different Column Names

Sometimes the join columns have different names.

Example:

- Customers DataFrame has `customer_id`.
- Orders DataFrame has `cust_id`.

Use an explicit join condition:

```python
result = customers.join(
    orders,
    customers.customer_id == orders.cust_id,
    "inner"
)
```

**Important:** When joining with an explicit condition, both join-key columns may appear in the output.

Select the required columns to avoid confusion:

```python
result = customers.join(
    orders,
    customers.customer_id == orders.cust_id,
    "inner"
).select(
    customers.customer_id,
    customers.customer_name,
    orders.order_id,
    orders.amount
)
```

---

## 7. Joining Multiple DataFrames

You can join more than two DataFrames by chaining joins.

Example:

```python
customer_orders = customers.join(
    orders,
    on="customer_id",
    how="inner"
)

final_df = customer_orders.join(
    products,
    on="product_id",
    how="left"
)
```

This combines customer, order, and product information.

**Important:** Make sure each join condition uses the correct key and that the expected relationship between records is understood.

---

## 8. Self Join

A self join joins a DataFrame with itself.

It is useful when records in the same dataset are related.

Example: Employee and manager information.

```python
employee_data = [
    (1, "Alice", None),
    (2, "Bob", 1),
    (3, "Charlie", 1),
    (4, "David", 2)
]

employees = spark.createDataFrame(
    employee_data,
    ["employee_id", "employee_name", "manager_id"]
)
```

Join the employee DataFrame with itself:

```python
emp = employees.alias("emp")
mgr = employees.alias("mgr")

result = emp.join(
    mgr,
    F.col("emp.manager_id") == F.col("mgr.employee_id"),
    "left"
).select(
    F.col("emp.employee_name").alias("employee"),
    F.col("mgr.employee_name").alias("manager")
)
```

Expected output:

| employee | manager |
|---|---|
| Alice | null |
| Bob | Alice |
| Charlie | Alice |
| David | Bob |

**Use case:** Employee-manager relationships, hierarchical data, and comparing records within the same dataset.

---

## 9. Cross Join

A cross join returns every possible combination of rows from two DataFrames.

```python
result = customers.crossJoin(orders)
```

If the first DataFrame contains 4 rows and the second contains 4 rows:

\[
4 \times 4 = 16 \text{ rows}
\]

**Use case:** Generate all possible combinations, such as product and region combinations.

**Warning:** Cross joins can produce very large datasets. Use them only when all combinations are actually required.

---

## 10. Handling Duplicate Column Names

When both DataFrames contain columns with the same name, the output can contain ambiguous column references.

Example:

```python
result = customers.join(
    orders,
    customers.customer_id == orders.customer_id,
    "inner"
)
```

Both DataFrames contain `customer_id`.

A safer approach is to use aliases and explicitly select columns:

```python
c = customers.alias("c")
o = orders.alias("o")

result = c.join(
    o,
    F.col("c.customer_id") == F.col("o.customer_id"),
    "inner"
).select(
    F.col("c.customer_id"),
    F.col("c.customer_name"),
    F.col("o.order_id"),
    F.col("o.amount")
)
```

When both DataFrames have the same join-column name, using:

```python
customers.join(orders, on="customer_id", how="inner")
```

is convenient because Spark produces one shared join-key column in the result.

---

## 11. Join Conditions with Multiple Columns

Sometimes one column is not enough to identify matching records.

Example: Join using both `customer_id` and `order_date`.

```python
result = df1.join(
    df2,
    (df1.customer_id == df2.customer_id) &
    (df1.order_date == df2.order_date),
    "inner"
)
```

Use `&` for AND and `|` for OR.

**Important:** Put each condition inside parentheses when combining conditions.

---

## 12. Join Performance in Spark

Joins can be expensive because Spark may need to shuffle data across partitions.

Important concepts:

### 12.1 Shuffle Join

A shuffle join moves data between partitions so that matching keys can be processed together.

It can involve significant network and disk I/O.

### 12.2 Broadcast Join

A broadcast join sends a small DataFrame to the executors so that it can be joined with a larger DataFrame without shuffling the large DataFrame for that join.

Example:

```python
result = large_df.join(
    F.broadcast(small_df),
    on="customer_id",
    how="inner"
)
```

**Use case:** Joining a large transaction DataFrame with a small lookup or reference DataFrame.

**Important:** Broadcasting is suitable only when the smaller DataFrame can fit safely in executor memory. Do not broadcast a large DataFrame blindly.

### 12.3 Data Skew

Data skew occurs when some join keys have significantly more records than others.

This can cause uneven workload across partitions.

For example, if one customer ID appears in millions of records while most customer IDs appear only a few times, some join tasks may take much longer.

### 12.4 Practical Optimization Tips

1. Filter unnecessary records before joining.
2. Select only the required columns.
3. Use broadcast joins when one side is genuinely small.
4. Check for duplicate keys and unexpected many-to-many relationships.
5. Investigate data skew when a few tasks take much longer than others.
6. Avoid unnecessary cross joins.
7. Use `explain()` to inspect the physical execution plan.

```python
result.explain()
```

---

## 13. Common Join Problems

### Problem 1: Duplicate Rows After a Join

A join can produce multiple rows for one key when either DataFrame contains duplicate keys.

Example:

If customer ID 1 appears twice in the customers DataFrame and three times in the orders DataFrame, joining on that key can produce:

\[
2 \times 3 = 6 \text{ rows}
\]

This is a many-to-many match.

**Solution:** Check key uniqueness and confirm the expected relationship before joining.

### Problem 2: Null Join Keys

In ordinary equality joins, null does not match null.

If both DataFrames contain null join keys, those rows do not match using:

```python
F.col("left.id") == F.col("right.id")
```

If null-safe equality is required, PySpark provides:

```python
F.col("left.id").eqNullSafe(F.col("right.id"))
```

Use this only when null values are intended to match each other.

### Problem 3: Filtering After a Left Join

Consider:

```python
result = customers.join(
    orders,
    on="customer_id",
    how="left"
).filter(
    F.col("amount") > 1000
)
```

Customers without matching orders have null amounts. The filter removes those rows.

As a result, the query no longer retains every customer.

If you want to retain every customer but only match orders above 1000, put the condition in the join condition:

```python
result = customers.join(
    orders,
    (customers.customer_id == orders.customer_id) &
    (orders.amount > 1000),
    "left"
)
```

---

## 14. Common Interview Questions

**Q1. What is a join in PySpark?**

A join combines two DataFrames using matching columns or conditions.

**Q2. What is the difference between inner and left join?**

- Inner join returns only matching records.
- Left join returns every record from the left DataFrame and matching records from the right.

**Q3. What is the difference between left semi and left anti join?**

- Left semi returns matching rows from the left DataFrame.
- Left anti returns non-matching rows from the left DataFrame.

**Q4. What is a broadcast join?**

It distributes a small DataFrame to executors so that it can be joined with a larger DataFrame without shuffling the large DataFrame for that join.

**Q5. Why can a join produce duplicate rows?**

Because the join key may appear multiple times in either DataFrame. Each matching combination can produce a separate output row.

**Q6. What is a self join?**

A self join joins a DataFrame with itself, often to represent relationships such as employee-manager connections.

**Q7. What is a cross join?**

A cross join produces every possible combination of rows from two DataFrames.

**Q8. What happens when join keys contain null values?**

Ordinary equality joins do not match null values. Use null-safe equality when matching nulls is specifically required.

**Q9. What is data skew in joins?**

It occurs when some join keys contain significantly more records than others, resulting in uneven workload across partitions.

**Q10. How do you optimize joins in PySpark?**

Filter early, select required columns, consider broadcast joins for small DataFrames, investigate skew, and inspect the execution plan.

---

## 15. Practice Questions

Using the `customers` and `orders` DataFrames:

1. Find customers who have placed orders using an inner join.
2. Find all customers, including those without orders, using a left join.
3. Find orders whose customer records are missing using a right or anti join with the appropriate table order.
4. Find customers who have never placed an order using a left anti join.
5. Find customers who have placed at least one order using a left semi join.
6. Join two DataFrames that have different join-key column names.
7. Join three DataFrames: customers, orders, and products.
8. Perform a self join to find employee-manager relationships.
9. Join using two columns as the matching condition.
10. Identify why a join is producing duplicate rows.
11. Apply a broadcast join between a large transaction table and a small lookup table.
12. Explain how a left join can accidentally behave like an inner join after filtering.

---

## 16. Quick Revision

| Join type | Main purpose |
|---|---|
| Inner | Keep matching records |
| Left | Keep all left records |
| Right | Keep all right records |
| Full Outer | Keep all records from both sides |
| Left Semi | Keep matching left records only |
| Left Anti | Keep non-matching left records only |
| Self Join | Join a DataFrame with itself |
| Cross Join | Generate all combinations |

**Important performance concepts:**

| Concept | Meaning |
|---|---|
| Shuffle Join | Moves data between partitions |
| Broadcast Join | Sends a small DataFrame to executors |
| Data Skew | Uneven distribution of join keys |
| `explain()` | Displays the execution plan |

---

## 17. Self-Assessment

- [ ] Explain all major join types.
- [ ] Write inner, left, right, and full outer joins.
- [ ] Explain left semi and left anti joins.
- [ ] Join DataFrames with different column names.
- [ ] Join using multiple conditions.
- [ ] Perform a self join.
- [ ] Explain why duplicate rows appear after joins.
- [ ] Explain how null join keys behave.
- [ ] Explain broadcast joins and shuffle joins.
- [ ] Identify data skew and basic join optimization techniques.

**Interview readiness:** You should be able to select the correct join type for a business requirement, write the PySpark code, predict which records will remain, and explain the performance impact.