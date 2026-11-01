

# Window Functions in PySpark

## 1. What Are Window Functions?

Window functions perform calculations across a group of related rows without reducing the number of rows in the DataFrame.

They allow us to perform calculations such as:

- Ranking records
- Finding the latest or earliest record
- Calculating running totals
- Comparing current and previous rows
- Calculating moving averages
- Finding the highest or lowest value within a group

**Interview definition:**

A window function performs calculations across a specified group of rows related to the current row while retaining individual rows in the output.

### Example

Consider an employee DataFrame:

| employee_id | name | department | salary |
|---:|---|---|---:|
| 101 | Rahul | IT | 50000 |
| 102 | Amit | IT | 60000 |
| 103 | Priya | HR | 45000 |
| 104 | Neha | HR | 55000 |
| 105 | Karan | IT | 70000 |

Suppose we want to rank employees based on salary within each department.

A normal `groupBy()` aggregation returns one row per department, whereas a window function keeps every employee row and adds a ranking column.

---

## 2. Creating a Window Specification

PySpark provides the `Window` class to define window specifications.

Import:

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import (
    row_number,
    rank,
    dense_rank,
    lag,
    lead,
    sum,
    avg,
    col
)
```

### Basic syntax

```python
window_spec = (
    Window
    .partitionBy("department")
    .orderBy(col("salary").desc())
)
```

A window specification defines which rows belong together and how they should be ordered.

It has three important concepts:

| Method | Purpose |
|---|---|
| `partitionBy()` | Divides rows into groups |
| `orderBy()` | Defines the order of rows |
| `rowsBetween()` | Defines a row-based window frame |

### A. partitionBy()

Groups rows based on one or more columns.

```python
Window.partitionBy("department")
```

For example:
- IT employees form one partition.
- HR employees form another partition.

### B. orderBy()

Defines the order of rows within each partition.

```python
Window.partitionBy("department").orderBy(
    col("salary").desc()
)
```

This orders employees by salary from highest to lowest within each department.

### C. rowsBetween()

Defines which rows are included in the calculation relative to the current row.

Example:

```python
Window.orderBy("employee_id").rowsBetween(
    Window.unboundedPreceding,
    Window.currentRow
)
```

This includes all rows from the beginning of the ordered window up to the current row.

---

## 3. Ranking Functions

Ranking functions assign a position to each row based on a specified ordering.

PySpark provides three commonly used ranking functions:

1. `row_number()`
2. `rank()`
3. `dense_rank()`

### A. row_number()

Assigns a unique sequential number to each row within a window partition.

```python
window_spec = (
    Window
    .partitionBy("department")
    .orderBy(col("salary").desc())
)

df = df.withColumn(
    "row_num",
    row_number().over(window_spec)
)
```

Example output:

| employee_id | name | department | salary | row_num |
|---:|---|---|---:|---:|
| 105 | Karan | IT | 70000 | 1 |
| 102 | Amit | IT | 60000 | 2 |
| 101 | Rahul | IT | 50000 | 3 |
| 104 | Neha | HR | 55000 | 1 |
| 103 | Priya | HR | 45000 | 2 |

The numbering starts again for each department.

**Use case:** Select the latest record for each customer when ordering by transaction date descending.

### B. rank()

Assigns the same rank to tied values and leaves gaps after ties.

Example:

| name | salary | rank |
|---|---:|---:|
| Rahul | 70000 | 1 |
| Amit | 70000 | 1 |
| Priya | 60000 | 3 |
| Neha | 50000 | 4 |

Code:

```python
df.withColumn(
    "salary_rank",
    rank().over(window_spec)
)
```

### C. dense_rank()

Assigns the same rank to tied values but does not leave gaps.

Example:

| name | salary | dense_rank |
|---|---:|---:|
| Rahul | 70000 | 1 |
| Amit | 70000 | 1 |
| Priya | 60000 | 2 |
| Neha | 50000 | 3 |

Code:

```python
df.withColumn(
    "salary_dense_rank",
    dense_rank().over(window_spec)
)
```

### Difference between ranking functions

| Function | Handles ties | Leaves gaps |
|---|---|---|
| `row_number()` | Gives each row a different number | No |
| `rank()` | Same rank for ties | Yes |
| `dense_rank()` | Same rank for ties | No |

**Interview tip:** If asked to find the second-highest distinct salary, `dense_rank()` is often the appropriate choice.

---

## 4. Finding the Top N Records per Group

This is one of the most common PySpark interview problems.

### Requirement

Find the top two highest-paid employees in each department.

```python
window_spec = (
    Window
    .partitionBy("department")
    .orderBy(col("salary").desc())
)

result_df = (
    df
    .withColumn(
        "rank",
        row_number().over(window_spec)
    )
    .filter(col("rank") <= 2)
)
```

Expected output:

| name | department | salary | rank |
|---|---|---:|---:|
| Karan | IT | 70000 | 1 |
| Amit | IT | 60000 | 2 |
| Neha | HR | 55000 | 1 |
| Priya | HR | 45000 | 2 |

### Important distinction

- Use `row_number()` when you need exactly N rows per group, assuming each group has enough rows.
- Use `rank()` or `dense_rank()` when ties should receive the same ranking and may result in more than N rows.

---

## 5. lag() – Accessing Previous Rows

`lag()` retrieves a value from a previous row within the specified window.

### Example

Suppose we have monthly sales:

| month | sales |
|---|---:|
| January | 1000 |
| February | 1500 |
| March | 1200 |
| April | 1800 |

We want to compare each month's sales with the previous month.

```python
window_spec = Window.orderBy("month_number")

result_df = df.withColumn(
    "previous_month_sales",
    lag("sales", 1).over(window_spec)
)
```

Output:

| month | sales | previous_month_sales |
|---|---:|---:|
| January | 1000 | NULL |
| February | 1500 | 1000 |
| March | 1200 | 1500 |
| April | 1800 | 1200 |

The first row has NULL because no previous row exists.

### Syntax

```python
lag(column, offset, default)
```

- `column`: Column whose value is required.
- `offset`: Number of rows to look backward.
- `default`: Optional value returned when the requested previous row does not exist.

Example:

```python
lag("sales", 1, 0)
```

Returns `0` when no previous row exists.

**Important:** Always use a meaningful ordering column. In the example, `month_number` ensures chronological ordering. Ordering by a month name alphabetically would not produce the correct sequence.

---

## 6. lead() – Accessing Next Rows

`lead()` retrieves a value from a future row within the specified window.

Example:

```python
window_spec = Window.orderBy("month_number")

result_df = df.withColumn(
    "next_month_sales",
    lead("sales", 1).over(window_spec)
)
```

Output:

| month | sales | next_month_sales |
|---|---:|---:|
| January | 1000 | 1500 |
| February | 1500 | 1200 |
| March | 1200 | 1800 |
| April | 1800 | NULL |

### Difference between lag() and lead()

| Function | Purpose |
|---|---|
| `lag()` | Accesses previous rows |
| `lead()` | Accesses next rows |

**Common use cases:**
- Comparing current and previous transactions.
- Identifying changes in customer behavior.
- Calculating month-over-month growth.
- Detecting gaps between dates.

---

## 7. Running Total Using Window Functions

A running total calculates the cumulative sum of values up to the current row.

Example:

| transaction_id | amount |
|---:|---:|
| 1 | 100 |
| 2 | 200 |
| 3 | 150 |
| 4 | 300 |

Code:

```python
window_spec = (
    Window
    .orderBy("transaction_id")
    .rowsBetween(
        Window.unboundedPreceding,
        Window.currentRow
    )
)

result_df = df.withColumn(
    "running_total",
    sum("amount").over(window_spec)
)
```

Output:

| transaction_id | amount | running_total |
|---:|---:|---:|
| 1 | 100 | 100 |
| 2 | 200 | 300 |
| 3 | 150 | 450 |
| 4 | 300 | 750 |

### Understanding rowsBetween()

```python
.rowsBetween(
    Window.unboundedPreceding,
    Window.currentRow
)
```

- `unboundedPreceding`: Start from the first row in the partition.
- `currentRow`: End at the current row.

### Running total per customer

```python
window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy("transaction_date")
    .rowsBetween(
        Window.unboundedPreceding,
        Window.currentRow
    )
)

result_df = df.withColumn(
    "running_total",
    sum("amount").over(window_spec)
)
```

This calculates a separate running total for each customer.

---

## 8. Moving Average

A moving average calculates the average over a fixed number of nearby rows.

Example: Calculate a three-row moving average.

```python
window_spec = (
    Window
    .orderBy("transaction_id")
    .rowsBetween(-2, 0)
)

result_df = df.withColumn(
    "moving_average",
    avg("amount").over(window_spec)
)
```

Here:

- `-2`: Two rows before the current row.
- `0`: The current row.

The window contains up to three rows, including the current row.

Example:

| transaction_id | amount | moving_average |
|---:|---:|---:|
| 1 | 100 | 100 |
| 2 | 200 | 150 |
| 3 | 150 | 150 |
| 4 | 300 | 216.67 |

The average for transaction 4 is:

\[
\frac{200+150+300}{3}=216.67
\]

**Important:** At the beginning of a dataset, fewer than three rows may be available, so the moving average uses the available rows.

---

## 9. Window Functions Without partitionBy()

You do not always need `partitionBy()`.

Example:

```python
window_spec = Window.orderBy(
    col("salary").desc()
)

df.withColumn(
    "salary_rank",
    rank().over(window_spec)
)
```

This ranks employees across the entire DataFrame rather than separately by department.

### Difference

```python
Window.partitionBy("department").orderBy("salary")
```

Creates separate windows for each department.

```python
Window.orderBy("salary")
```

Uses one window across all rows.

**Important:** A window without partitioning may require data to be brought together for ordering. This can become expensive for large datasets.

---

## 10. Window Functions vs groupBy()

Both can perform calculations on groups, but they produce different output structures.

### Using groupBy()

```python
df.groupBy("department").agg(
    avg("salary").alias("average_salary")
)
```

Output:

| department | average_salary |
|---|---:|
| IT | 60000 |
| HR | 50000 |

`groupBy()` returns one row per department.

### Using a window function

```python
window_spec = Window.partitionBy("department")

result_df = df.withColumn(
    "average_salary",
    avg("salary").over(window_spec)
)
```

Output:

| name | department | salary | average_salary |
|---|---|---:|---:|
| Rahul | IT | 50000 | 60000 |
| Amit | IT | 60000 | 60000 |
| Karan | IT | 70000 | 60000 |
| Priya | HR | 45000 | 50000 |
| Neha | HR | 55000 | 50000 |

The window function retains all employee rows.

### Key differences

| groupBy() | Window Functions |
|---|---|
| Groups rows and aggregates them | Calculates across related rows |
| Usually reduces the number of rows | Retains individual rows |
| Useful for department-level summaries | Useful for ranking and row-level comparisons |
| Uses aggregation functions | Uses ranking, analytical, and aggregation functions |

---

## 11. Common Real-World ETL Use Cases

Window functions are frequently used in production data pipelines.

### A. Find the latest record for each customer

```python
window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy(col("updated_at").desc())
)

latest_df = (
    df
    .withColumn(
        "row_num",
        row_number().over(window_spec)
    )
    .filter(col("row_num") == 1)
    .drop("row_num")
)
```

### B. Identify changes between consecutive records

```python
window_spec = (
    Window
    .partitionBy("customer_id")
    .orderBy("transaction_date")
)

result_df = df.withColumn(
    "previous_amount",
    lag("amount").over(window_spec)
)
```

### C. Calculate salary ranking by department

```python
window_spec = (
    Window
    .partitionBy("department")
    .orderBy(col("salary").desc())
)

result_df = df.withColumn(
    "salary_rank",
    dense_rank().over(window_spec)
)
```

### D. Calculate cumulative sales by region

```python
window_spec = (
    Window
    .partitionBy("region")
    .orderBy("sales_date")
    .rowsBetween(
        Window.unboundedPreceding,
        Window.currentRow
    )
)

result_df = df.withColumn(
    "cumulative_sales",
    sum("sales").over(window_spec)
)
```

---

## 12. Performance Considerations

1. **Use partitionBy() when appropriate.**

   It divides calculations into logical groups. However, partitioning by a highly skewed column can create performance problems.

2. **Choose meaningful ordering columns.**

   Ranking and lag/lead calculations require a reliable ordering.

3. **Avoid unnecessary window calculations.**

   Window operations may involve data shuffling and sorting.

4. **Reuse window specifications.**

   If multiple calculations use the same partitioning and ordering, define the window specification once.

5. **Handle duplicate ordering values.**

   When deterministic row ordering is important, add a unique tie-breaker column.

   ```python
   Window.partitionBy("customer_id").orderBy(
       col("updated_at").desc(),
       col("transaction_id").desc()
   )
   ```

6. **Understand the difference between rowsBetween() and rangeBetween().**

   `rowsBetween()` uses physical row positions, whereas `rangeBetween()` uses the ordering expression's value range.

7. **Watch out for skewed partitions.**

   If one partition contains significantly more records than others, that partition may take much longer to process.

---

## 13. Common Mistakes

1. **Forgetting `.over(window_spec)`**

   Window functions need a window specification to define how they operate.

2. **Using rank() when exactly one row is required**

   Ties can produce multiple rows with the same rank. Use `row_number()` with a deterministic ordering when exactly one record is required.

3. **Not using partitionBy() when group-level calculations are required**

   Without it, the calculation may apply across the entire DataFrame.

4. **Using incorrect ordering for lag() and lead()**

   Always order by a meaningful date, timestamp, or sequence column.

5. **Assuming row_number() produces a deterministic result with ties**

   Add a tie-breaker column when multiple rows have the same ordering value.

6. **Confusing rowsBetween() with partitionBy()**

   `partitionBy()` defines groups. `rowsBetween()` defines the frame of rows used for a calculation.

7. **Confusing window functions with groupBy()**

   `groupBy()` reduces rows through aggregation. Window functions retain individual rows.

8. **Ignoring NULL values**

   Aggregation and analytical functions have different NULL-handling behavior. Understand how the selected function handles NULL values.

---

## 14. Common Interview Questions

**Q1. What is a window function in PySpark?**

A window function performs calculations across related rows without reducing the number of rows in the output.

**Q2. What is the purpose of Window.partitionBy()?**

It divides the DataFrame into separate groups for window calculations.

**Q3. What is the purpose of Window.orderBy()?**

It defines the order of rows within a window.

**Q4. What is the difference between row_number(), rank(), and dense_rank()?**

- `row_number()` assigns a unique sequential number.
- `rank()` gives tied rows the same rank and leaves gaps.
- `dense_rank()` gives tied rows the same rank without gaps.

**Q5. What is the difference between lag() and lead()?**

`lag()` accesses a previous row, while `lead()` accesses a future row.

**Q6. How do you find the latest record for each customer?**

Use `row_number()` with `partitionBy("customer_id")` and order by the timestamp in descending order. Filter for row number 1.

**Q7. How do you calculate a running total?**

Use `sum()` over a window ordered by the required sequence column, with a frame from `unboundedPreceding` to `currentRow`.

**Q8. What is the difference between groupBy() and window functions?**

`groupBy()` typically reduces the dataset to aggregated rows. Window functions calculate values while retaining the original rows.

**Q9. What is rowsBetween() used for?**

It defines the row-based frame used for a window calculation.

**Q10. What does Window.unboundedPreceding mean?**

It represents the first row in the current window partition.

**Q11. Can we use window functions without partitionBy()?**

Yes. The calculation applies across the entire DataFrame window.

**Q12. How would you find the top three employees in each department?**

Use `row_number()` or a ranking function over a window partitioned by department and ordered by salary descending, then filter the result.

**Q13. What is the difference between rowsBetween() and rangeBetween()?**

`rowsBetween()` defines a frame based on physical row positions. `rangeBetween()` defines a frame based on values of the ordering expression.

**Q14. Why can window functions be expensive?**

They may require shuffling and sorting data, especially when partitions are large or the ordering involves many records.

---

## 15. Practice Questions

Use an employee DataFrame with these columns:

`employee_id`, `name`, `department`, `salary`, `joining_date`.

Try solving these problems:

1. Assign a unique row number to every employee.
2. Rank employees by salary within each department.
3. Find the second-highest distinct salary in each department.
4. Find the top three employees in each department.
5. Find the latest employee record for each department.
6. Use `lag()` to find the previous employee's salary within each department.
7. Use `lead()` to find the next employee's salary.
8. Calculate the running total of salaries within each department.
9. Calculate the average salary for each department without reducing the number of rows.
10. Find employees whose salary is greater than the previous employee's salary within their department.
11. Calculate a three-row moving average of sales.
12. Remove duplicate customer records while retaining the latest updated record.

---

## 16. Quick Revision

| Function / Method | Purpose |
|---|---|
| `Window.partitionBy()` | Divides rows into groups |
| `Window.orderBy()` | Defines row ordering |
| `Window.rowsBetween()` | Defines a row-based frame |
| `row_number()` | Unique sequential numbering |
| `rank()` | Ranking with gaps |
| `dense_rank()` | Ranking without gaps |
| `lag()` | Previous row value |
| `lead()` | Next row value |
| `sum().over()` | Running or windowed sum |
| `avg().over()` | Windowed average |
| `Window.unboundedPreceding` | Beginning of the partition |
| `Window.currentRow` | Current row |

### Important code to remember

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import (
    row_number,
    rank,
    dense_rank,
    lag,
    lead,
    sum,
    avg,
    col
)

# Define window
window_spec = (
    Window
    .partitionBy("department")
    .orderBy(col("salary").desc())
)

# Row number
df.withColumn(
    "row_num",
    row_number().over(window_spec)
)

# Rank
df.withColumn(
    "rank",
    rank().over(window_spec)
)

# Dense rank
df.withColumn(
    "dense_rank",
    dense_rank().over(window_spec)
)

# Previous row
df.withColumn(
    "previous_salary",
    lag("salary").over(window_spec)
)

# Next row
df.withColumn(
    "next_salary",
    lead("salary").over(window_spec)
)

# Running total
running_window = (
    Window
    .partitionBy("department")
    .orderBy("employee_id")
    .rowsBetween(
        Window.unboundedPreceding,
        Window.currentRow
    )
)

df.withColumn(
    "running_total",
    sum("salary").over(running_window)
)
```

---

## 17. Self-Assessment

Before moving to the next topic, make sure you can:

- [ ] Explain what window functions are.
- [ ] Define a window specification.
- [ ] Understand partitionBy(), orderBy(), and rowsBetween().
- [ ] Explain row_number(), rank(), and dense_rank().
- [ ] Find top N records within each group.
- [ ] Use lag() and lead().
- [ ] Calculate running totals.
- [ ] Calculate moving averages.
- [ ] Explain window functions versus groupBy().
- [ ] Find the latest record for each customer.
- [ ] Identify performance considerations and common mistakes.

**Interview readiness:** You should be able to solve top-N-per-group, latest-record, ranking, running-total, and previous-versus-current-row problems using PySpark window functions.

