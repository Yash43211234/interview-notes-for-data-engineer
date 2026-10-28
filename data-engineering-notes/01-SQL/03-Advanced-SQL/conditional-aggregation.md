# SQL Conditional Aggregation

## 1. What Is Conditional Aggregation?

**Conditional aggregation** means calculating aggregate values such as `SUM()`, `COUNT()`, or `AVG()` only for rows that satisfy a specific condition.

It is commonly used in data engineering and analytics to:
- Calculate department-wise metrics.
- Count successful and failed transactions.
- Calculate revenue for different categories.
- Build reports with multiple metrics in one query.
- Summarize data without writing separate queries for each condition.

The most common approach combines aggregate functions with `CASE WHEN`.

### Basic syntax

```sql
SELECT
    group_column,
    SUM(
        CASE
            WHEN condition THEN value
            ELSE 0
        END
    ) AS result
FROM table_name
GROUP BY group_column;
```

**Remember:** `CASE` decides which values contribute to the aggregate; the aggregate function calculates the final result.

---

## 2. Sample Data

We will use an `orders` table for the examples.

| order_id | customer_id | status | amount |
|---:|---:|---|---:|
| 101 | 1 | Completed | 1000 |
| 102 | 2 | Failed | 500 |
| 103 | 1 | Completed | 1500 |
| 104 | 3 | Pending | 800 |
| 105 | 2 | Failed | 700 |
| 106 | 3 | Completed | 1200 |

---

## 3. Conditional `SUM()`

Use conditional `SUM()` when you want to add values only for rows matching a condition.

### Example: Calculate completed-order revenue

```sql
SELECT
    SUM(
        CASE
            WHEN status = 'Completed' THEN amount
            ELSE 0
        END
    ) AS completed_revenue
FROM orders;
```

Expected result:

| completed_revenue |
|---:|
| 3700 |

**How it works:**

- Completed orders contribute 1000, 1500, and 1200.
- Other orders contribute 0.
- `SUM()` adds the resulting values.

### Example: Calculate revenue by status

```sql
SELECT
    status,
    SUM(amount) AS total_amount
FROM orders
GROUP BY status;
```

This query groups all orders by status. However, if you want multiple status-specific metrics as separate columns, conditional aggregation is more useful.

---

## 4. Conditional `COUNT()`

There are two common approaches for counting rows that satisfy a condition.

### Method 1: `COUNT(CASE WHEN ...)`

```sql
SELECT
    COUNT(
        CASE
            WHEN status = 'Completed' THEN 1
        END
    ) AS completed_orders
FROM orders;
```

Expected result: `3`

`COUNT(expression)` counts non-`NULL` values. Rows that do not satisfy the condition produce `NULL`, so they are not counted.

### Method 2: `SUM(CASE WHEN ...)`

```sql
SELECT
    SUM(
        CASE
            WHEN status = 'Failed' THEN 1
            ELSE 0
        END
    ) AS failed_orders
FROM orders;
```

Expected result: `2`

Each failed order contributes 1; every other order contributes 0.

### Important difference

```sql
COUNT(CASE WHEN condition THEN 1 END)
```

and

```sql
SUM(CASE WHEN condition THEN 1 ELSE 0 END)
```

both count matching rows when the conditions and data types are appropriate.

**Common mistake:** Avoid writing `COUNT(CASE WHEN condition THEN 1 ELSE 0 END)` to count matching rows. Both 1 and 0 are non-`NULL`, so it counts every row.

---

## 5. Conditional Aggregation with `GROUP BY`

Conditional aggregation becomes especially useful when you need metrics for each group.

### Example: Count completed and failed orders for each customer

```sql
SELECT
    customer_id,
    SUM(
        CASE
            WHEN status = 'Completed' THEN 1
            ELSE 0
        END
    ) AS completed_orders,
    SUM(
        CASE
            WHEN status = 'Failed' THEN 1
            ELSE 0
        END
    ) AS failed_orders
FROM orders
GROUP BY customer_id
ORDER BY customer_id;
```

Expected result:

| customer_id | completed_orders | failed_orders |
|---:|---:|---:|
| 1 | 2 | 0 |
| 2 | 0 | 2 |
| 3 | 1 | 0 |

**Why is this useful?**

Instead of running a separate query for each status, you can calculate several metrics in one grouped query.

---

## 6. Conditional `AVG()`

You can calculate an average using only rows that satisfy a condition.

### Example: Calculate the average amount of completed orders

```sql
SELECT
    AVG(
        CASE
            WHEN status = 'Completed' THEN amount
        END
    ) AS avg_completed_amount
FROM orders;
```

Expected result: `1233.33`

The `CASE` expression returns the amount for completed orders and `NULL` for other orders. Since `AVG()` ignores `NULL`, only completed orders contribute to the average.

**Important:** Do not use `ELSE 0` unless you intentionally want nonmatching rows to contribute zeros to the average.

Compare:

```sql
AVG(CASE WHEN status = 'Completed' THEN amount END)
```

This calculates the average of completed orders only.

```sql
AVG(CASE WHEN status = 'Completed' THEN amount ELSE 0 END)
```

This includes zero values for noncompleted orders, changing the result.

---

## 7. Conditional `MIN()` and `MAX()`

Conditional expressions can also be used to find minimum and maximum values for a category.

### Example: Find the highest completed-order amount

```sql
SELECT
    MAX(
        CASE
            WHEN status = 'Completed' THEN amount
        END
    ) AS highest_completed_amount
FROM orders;
```

Expected result: `1500`

### Example: Find the minimum failed-order amount

```sql
SELECT
    MIN(
        CASE
            WHEN status = 'Failed' THEN amount
        END
    ) AS lowest_failed_amount
FROM orders;
```

Expected result: `500`

Nonmatching rows return `NULL` and do not affect `MIN()` or `MAX()`.

---

## 8. Multiple Conditions in One Query

Conditional aggregation allows you to calculate several business metrics in a single query.

### Example: Build an order summary

```sql
SELECT
    COUNT(*) AS total_orders,

    SUM(
        CASE WHEN status = 'Completed'
             THEN 1 ELSE 0 END
    ) AS completed_orders,

    SUM(
        CASE WHEN status = 'Failed'
             THEN 1 ELSE 0 END
    ) AS failed_orders,

    SUM(
        CASE WHEN status = 'Pending'
             THEN 1 ELSE 0 END
    ) AS pending_orders,

    SUM(
        CASE WHEN status = 'Completed'
             THEN amount ELSE 0 END
    ) AS completed_revenue
FROM orders;
```

Expected result:

| total_orders | completed_orders | failed_orders | pending_orders | completed_revenue |
|---:|---:|---:|---:|---:|
| 6 | 3 | 2 | 1 | 3700 |

This pattern is common in reporting pipelines and data warehouse transformations.

---

## 9. Conditional Aggregation vs. `WHERE`

These approaches are related, but they serve different purposes.

### Using `WHERE`

```sql
SELECT SUM(amount) AS completed_revenue
FROM orders
WHERE status = 'Completed';
```

This filters the input rows before aggregation. Only completed orders remain.

### Using conditional aggregation

```sql
SELECT
    SUM(
        CASE
            WHEN status = 'Completed' THEN amount
            ELSE 0
        END
    ) AS completed_revenue
FROM orders;
```

This keeps all input rows in the query but makes only completed orders contribute to the sum.

### When should you use each?

| Requirement | Better fit |
|---|---|
| Calculate one metric for one category | `WHERE` is often simpler |
| Calculate several category-specific metrics together | Conditional aggregation |
| Keep groups even when they have no matching rows | Conditional aggregation may be useful |
| Filter entire groups after aggregation | `HAVING` |

**Interview point:** A `WHERE` condition removes rows from the input. A `CASE` expression inside an aggregate changes what each row contributes to that aggregate.

---

## 10. Conditional Aggregation vs. `FILTER`

Some databases support the aggregate `FILTER` clause, which applies a condition directly to an aggregate.

Example in databases that support it:

```sql
SELECT
    COUNT(*) FILTER (
        WHERE status = 'Completed'
    ) AS completed_orders,
    SUM(amount) FILTER (
        WHERE status = 'Failed'
    ) AS failed_amount
FROM orders;
```

The equivalent approach using `CASE` is:

```sql
SELECT
    COUNT(
        CASE WHEN status = 'Completed' THEN 1 END
    ) AS completed_orders,
    SUM(
        CASE WHEN status = 'Failed' THEN amount END
    ) AS failed_amount
FROM orders;
```

**Remember:** `FILTER` support varies by database. `CASE WHEN` is a widely portable approach, though SQL dialect details can still differ.

---

## 11. Common Mistakes

1. **Using `ELSE 0` with `AVG()` unintentionally.** It can lower the average by including nonmatching rows as zeros.
2. **Counting both 1 and 0.** `COUNT()` counts all non-`NULL` values.
3. **Forgetting `GROUP BY`.** If you need one result per customer or department, group by that column.
4. **Mixing conditions.** Check that each `CASE WHEN` uses the correct status or category.
5. **Ignoring `NULL` values.** Most aggregate functions ignore `NULL`, but `COUNT(*)` counts rows regardless of column values.
6. **Using conditional aggregation when `WHERE` is simpler.** Prefer the simplest query that satisfies the requirement.
7. **Assuming all databases support `FILTER`.** Use `CASE WHEN` when portability matters.

---

## 12. Interview Questions

### Q1. What is conditional aggregation?

It applies conditions inside aggregate expressions so that only matching rows contribute to a calculation.

### Q2. How do you count rows matching a condition?

Use `COUNT(CASE WHEN condition THEN 1 END)` or `SUM(CASE WHEN condition THEN 1 ELSE 0 END)`.

### Q3. What is the difference between `COUNT()` and `SUM()` for conditional counting?

`COUNT()` counts non-`NULL` expressions, while `SUM()` adds numeric values. Both can count matching rows when written correctly.

### Q4. Why might `AVG(CASE WHEN ... THEN amount ELSE 0 END)` be incorrect?

Because nonmatching rows contribute zeros to the average instead of being excluded.

### Q5. Can you calculate multiple metrics in one query?

Yes. Use multiple conditional aggregate expressions in the same `SELECT` statement.

### Q6. What is the difference between `WHERE` and conditional aggregation?

`WHERE` filters input rows for the query. Conditional aggregation lets different aggregate expressions use different conditions over the same input rows.

### Q7. Where is conditional aggregation used in data engineering?

It is commonly used for status summaries, quality checks, financial metrics, reconciliation reports, and warehouse transformations.

---

## 13. Practice Questions

Use the `orders` table from Section 2.

### Beginner

- [ ] Count completed orders.
- [ ] Calculate total revenue from failed orders.
- [ ] Find the average amount of completed orders.
- [ ] Find the highest pending-order amount.

### Intermediate

- [ ] Count completed, failed, and pending orders for each customer.
- [ ] Calculate total completed revenue for each customer.
- [ ] Show total orders and completed orders in the same query.
- [ ] Calculate completed and failed revenue as separate columns.

### Advanced

- [ ] Calculate each customer's completed-order percentage.
- [ ] Find customers with at least two failed orders using `HAVING`.
- [ ] Build a report with total order count, completed revenue, failed revenue, and average completed-order amount for each customer.
- [ ] Explain how `NULL` amounts affect `SUM()`, `COUNT()`, and `AVG()`.

For every query, verify your output against the sample data.

---

## 14. Quick Revision

| Concept | Remember |
|---|---|
| Conditional aggregation | Aggregate values based on conditions |
| `SUM(CASE WHEN...)` | Conditional totals or counts |
| `COUNT(CASE WHEN...)` | Counts matching non-`NULL` results |
| `AVG(CASE WHEN...)` | Average of matching values when nonmatches return `NULL` |
| Conditional `MIN()` / `MAX()` | Minimum or maximum for a condition |
| `GROUP BY` | Calculates metrics for each group |
| `WHERE` | Filters input rows |
| `HAVING` | Filters groups after aggregation |
| Multiple metrics | Use multiple aggregate expressions in one query |

## 15. Self-Check

Before considering this topic complete, make sure you can:

- [ ] Explain conditional aggregation in simple language.
- [ ] Write conditional `SUM()` and `COUNT()` queries.
- [ ] Calculate conditional averages without accidentally including zeros.
- [ ] Calculate multiple metrics in a single query.
- [ ] Combine conditional aggregation with `GROUP BY` and `HAVING`.
- [ ] Explain when to use `WHERE` instead.
- [ ] Check the output manually using sample data.

**Final takeaway:** Use `CASE WHEN` inside aggregate functions when you need several condition-specific metrics from the same set of rows.