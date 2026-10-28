# SQL LEAD() and LAG() Functions

## 1. What Are LEAD() and LAG()?

`LEAD()` and `LAG()` are SQL window functions that let you access values from another row without joining the table to itself.

- **`LAG()`** retrieves a value from a previous row.
- **`LEAD()`** retrieves a value from a subsequent row.

They are useful for comparing consecutive records, analyzing changes over time, identifying gaps, and building data quality checks.

Common data engineering use cases:
- Comparing today's sales with yesterday's sales.
- Calculating changes in customer balances.
- Finding the previous or next event for a user.
- Detecting gaps between timestamps.
- Comparing current and previous pipeline metrics.

## 2. Basic Syntax

### LAG()

```sql
LAG(expression, offset, default_value)
OVER (
    PARTITION BY group_column
    ORDER BY sort_column
)
```

### LEAD()

```sql
LEAD(expression, offset, default_value)
OVER (
    PARTITION BY group_column
    ORDER BY sort_column
)
```

| Argument | Meaning |
|---|---|
| `expression` | The column or expression whose value you want |
| `offset` | Number of rows backward for `LAG()` or forward for `LEAD()`; defaults to 1 |
| `default_value` | Value returned when the target row is outside the partition; defaults to `NULL` |
| `PARTITION BY` | Separates rows into independent groups |
| `ORDER BY` | Defines which rows are previous and next |

**Remember:** `ORDER BY` determines row sequence, not the final display order. Add an outer `ORDER BY` if the output must appear in a particular order.

---

## 3. Sample Data

We will use a `daily_sales` table.

| sale_date | store_id | sales |
|---|---|---:|
| 2026-01-01 | 1 | 1000 |
| 2026-01-02 | 1 | 1200 |
| 2026-01-03 | 1 | 900 |
| 2026-01-04 | 1 | 1500 |
| 2026-01-01 | 2 | 800 |
| 2026-01-02 | 2 | 950 |
| 2026-01-03 | 2 | 1100 |

Assume each store has at most one record per date.

## 4. LAG(): Access the Previous Row

### Example: Compare sales with the previous day's sales

```sql
SELECT
    sale_date,
    sales,
    LAG(sales) OVER (
        ORDER BY sale_date
    ) AS previous_sales
FROM daily_sales
WHERE store_id = 1
ORDER BY sale_date;
```

Expected result:

| sale_date | sales | previous_sales |
|---|---:|---:|
| 2026-01-01 | 1000 | NULL |
| 2026-01-02 | 1200 | 1000 |
| 2026-01-03 | 900 | 1200 |
| 2026-01-04 | 1500 | 900 |

**How it works:**

1. The rows are ordered by `sale_date`.
2. `LAG(sales)` retrieves the sales value from the preceding row.
3. The first row has no preceding row, so its result is `NULL`.

Notice that the query filters to store 1 before applying the window function. The window therefore operates only on that store's rows.

---

## 5. LEAD(): Access the Next Row

### Example: Compare sales with the next day's sales

```sql
SELECT
    sale_date,
    sales,
    LEAD(sales) OVER (
        ORDER BY sale_date
    ) AS next_sales
FROM daily_sales
WHERE store_id = 1
ORDER BY sale_date;
```

Expected result:

| sale_date | sales | next_sales |
|---|---:|---:|
| 2026-01-01 | 1000 | 1200 |
| 2026-01-02 | 1200 | 900 |
| 2026-01-03 | 900 | 1500 |
| 2026-01-04 | 1500 | NULL |

The last row has no following row, so `LEAD()` returns `NULL`.

---

## 6. LAG() with PARTITION BY

`PARTITION BY` separates the data into groups. The previous or next row is then calculated within each group rather than across the entire table.

### Example: Previous sales for each store

```sql
SELECT
    store_id,
    sale_date,
    sales,
    LAG(sales) OVER (
        PARTITION BY store_id
        ORDER BY sale_date
    ) AS previous_sales
FROM daily_sales
ORDER BY store_id, sale_date;
```

The first record for each store has `NULL` for `previous_sales`.

**Why is this important?**

Without `PARTITION BY store_id`, the first row for store 2 could refer to the final row for store 1. Partitioning prevents values from crossing store boundaries.

---

## 7. Specify the Offset

The offset determines how many rows backward or forward to look.

### Example: Compare sales with two rows earlier

```sql
SELECT
    sale_date,
    sales,
    LAG(sales, 2) OVER (
        PARTITION BY store_id
        ORDER BY sale_date
    ) AS sales_two_rows_ago
FROM daily_sales
WHERE store_id = 1
ORDER BY sale_date;
```

Expected result:

| sale_date | sales | sales_two_rows_ago |
|---|---:|---:|
| 2026-01-01 | 1000 | NULL |
| 2026-01-02 | 1200 | NULL |
| 2026-01-03 | 900 | 1000 |
| 2026-01-04 | 1500 | 1200 |

`LAG(sales, 2)` retrieves the value two rows earlier.

Similarly, `LEAD(sales, 2)` retrieves the value two rows later.

**Important:** An offset counts rows, not calendar days. If dates are missing, two rows earlier might be several days earlier.

---

## 8. Specify a Default Value

You can provide a default value for cases where the target row does not exist.

```sql
SELECT
    sale_date,
    sales,
    LAG(sales, 1, 0) OVER (
        PARTITION BY store_id
        ORDER BY sale_date
    ) AS previous_sales
FROM daily_sales
WHERE store_id = 1
ORDER BY sale_date;
```

The first row returns `0` instead of `NULL`.

**Important distinction:** The default is used when the requested row is outside the partition. It does not automatically replace a `NULL` value stored in an existing previous row.

---

## 9. Calculate the Difference Between Consecutive Rows

A common interview question is to calculate the change between the current and previous values.

### Example: Calculate the change in sales

```sql
SELECT
    sale_date,
    sales,
    LAG(sales) OVER (
        PARTITION BY store_id
        ORDER BY sale_date
    ) AS previous_sales,
    sales - LAG(sales) OVER (
        PARTITION BY store_id
        ORDER BY sale_date
    ) AS sales_change
FROM daily_sales
WHERE store_id = 1
ORDER BY sale_date;
```

Expected result:

| sale_date | sales | previous_sales | sales_change |
|---|---:|---:|---:|
| 2026-01-01 | 1000 | NULL | NULL |
| 2026-01-02 | 1200 | 1000 | 200 |
| 2026-01-03 | 900 | 1200 | -300 |
| 2026-01-04 | 1500 | 900 | 600 |

A positive value indicates an increase; a negative value indicates a decrease.

### Calculate percentage change

```sql
SELECT
    sale_date,
    sales,
    LAG(sales) OVER (
        PARTITION BY store_id
        ORDER BY sale_date
    ) AS previous_sales,
    100.0 * (
        sales - LAG(sales) OVER (
            PARTITION BY store_id
            ORDER BY sale_date
        )
    ) / NULLIF(
        LAG(sales) OVER (
            PARTITION BY store_id
            ORDER BY sale_date
        ),
        0
    ) AS percentage_change
FROM daily_sales
WHERE store_id = 1
ORDER BY sale_date;
```

`NULLIF(previous_sales, 0)` prevents division by zero.

For January 2, the percentage change is:

\[
\frac{1200-1000}{1000}\times100=20\%
\]

The first row has no previous value, so the percentage change is `NULL`.

---

## 10. Find Rows Where Values Increased

You can compare a value with the result of `LAG()` to identify increases.

### Example: Find days when sales increased

```sql
WITH sales_comparison AS (
    SELECT
        store_id,
        sale_date,
        sales,
        LAG(sales) OVER (
            PARTITION BY store_id
            ORDER BY sale_date
        ) AS previous_sales
    FROM daily_sales
)
SELECT
    store_id,
    sale_date,
    sales,
    previous_sales
FROM sales_comparison
WHERE sales > previous_sales
ORDER BY store_id, sale_date;
```

This returns rows where sales exceed the previous row's sales within the same store.

A CTE is useful because many databases do not allow a window function to be referenced directly in the same query block's `WHERE` clause.

---

## 11. Find Gaps Between Events

`LAG()` is useful for checking the time between consecutive events.

Assume an `events` table has `user_id` and `event_time`.

```sql
SELECT
    user_id,
    event_time,
    LAG(event_time) OVER (
        PARTITION BY user_id
        ORDER BY event_time
    ) AS previous_event_time
FROM events;
```

You can then calculate the elapsed time between events using your database's timestamp-difference function.

**Use cases:**
- Detect long gaps in user activity.
- Identify delayed event processing.
- Analyze intervals between transactions.
- Check whether expected records are missing.

Remember that a gap between consecutive rows is not necessarily a missing record. You need a business rule defining the expected frequency.

---

## 12. LAG() and LEAD() vs. Self Joins

| Feature | LAG()/LEAD() | Self join |
|---|---|---|
| Purpose | Access preceding or following rows in an ordered window | Match rows from a table to other rows in the same table |
| Row relationship | Defined by ordering and offset | Defined by a join condition |
| Common use | Sequential comparisons and event analysis | Matching related records or more complex relationships |
| Readability | Often simpler for previous/next row problems | Can be more flexible for complex matching |

**Interview takeaway:** If the task is to compare the current row with the previous or next row in a defined sequence, start by considering `LAG()` or `LEAD()`.

---

## 13. Common Mistakes

1. **Forgetting `ORDER BY` inside `OVER()`.** The previous and next rows need a defined sequence.
2. **Forgetting `PARTITION BY`.** Rows from different customers, stores, or accounts may be compared accidentally.
3. **Assuming the offset represents days.** It counts rows, not calendar days.
4. **Ignoring duplicate sort values.** Add a unique tie-breaker, such as an ID, when necessary to make the order deterministic.
5. **Confusing the default with `NULL` replacement.** The default applies when the requested row does not exist, not when an existing row's value is `NULL`.
6. **Filtering rows too early.** `WHERE` filtering occurs before window functions in the same query block, so it can change which rows are available for comparison.
7. **Dividing by zero.** Use `NULLIF()` or another suitable safeguard for percentage calculations.
8. **Assuming `LAG()` returns the previous calendar day.** It returns the previous row according to the specified ordering.

---

## 14. Interview Questions

### Q1. What is the difference between LEAD() and LAG()?

`LAG()` accesses a previous row; `LEAD()` accesses a subsequent row.

### Q2. Why do we use PARTITION BY?

To calculate previous and next values independently within each group.

### Q3. What happens to the first row when using LAG()?

It returns `NULL` by default because there is no preceding row in the partition, unless a default value is supplied.

### Q4. Does LAG(sales, 7) return sales from seven days ago?

Not necessarily. It returns the value seven rows earlier in the ordered partition.

### Q5. Can LEAD() and LAG() be used with aggregate functions?

They are window functions and can be combined with aggregate expressions in supported query structures. The appropriate query layers depend on the calculation.

### Q6. How do you find day-over-day sales changes?

Use `LAG()` ordered by date and subtract the previous value from the current value.

### Q7. How do you avoid comparing records from different customers?

Use `PARTITION BY customer_id` along with the required ordering.

### Q8. Can you use LAG() directly in WHERE?

Generally, no. Calculate it in a CTE or derived table, then filter in the outer query.

### Q9. How do you calculate percentage change safely?

Subtract the previous value from the current value, divide by the previous value using `NULLIF(previous_value, 0)`, and multiply by 100.

### Q10. What is the difference between a window function and a self join?

A window function works across rows in a defined window without combining the table with itself. A self join explicitly matches rows using join conditions.

---

## 15. Practice Questions

Use the `daily_sales` table from Section 3.

### Beginner

- [ ] Display each store's previous day's sales.
- [ ] Display each store's next day's sales.
- [ ] Retrieve the sales value two rows earlier.
- [ ] Replace the missing previous-row result with zero.

### Intermediate

- [ ] Calculate the difference between current and previous sales.
- [ ] Calculate percentage change from the previous row.
- [ ] Find dates when sales increased compared with the previous row.
- [ ] Find the previous event timestamp for each user.

### Advanced

- [ ] Find the first date on which each store's sales exceeded 1,000.
- [ ] Identify consecutive records separated by a gap greater than one day.
- [ ] Explain how duplicate timestamps can affect results and how to make ordering deterministic.
- [ ] Explain how filtering rows before applying `LAG()` can change the result.

---

## 16. Quick Revision

| Concept | Remember |
|---|---|
| `LAG()` | Previous row |
| `LEAD()` | Next row |
| `offset` | Number of rows to move |
| `default_value` | Value used when the target row is outside the partition |
| `PARTITION BY` | Separates groups |
| `ORDER BY` | Defines row sequence |
| Difference | Current value minus previous value |
| Percentage change | Difference divided by previous value, multiplied by 100 |
| Missing dates | Row offsets are not calendar offsets |
| Filtering | Use a CTE or derived table when filtering window results |

## 17. Self-Check

Before considering this topic complete, make sure you can:

- [ ] Explain `LEAD()` and `LAG()` in simple language.
- [ ] Write both functions with `PARTITION BY` and `ORDER BY`.
- [ ] Use offsets and default values.
- [ ] Calculate consecutive-row differences and percentage changes.
- [ ] Find increases and gaps between events.
- [ ] Explain why deterministic ordering matters.
- [ ] Distinguish row offsets from calendar-day differences.
- [ ] Solve the practice questions without copying the examples.

**Final takeaway:** Use `LAG()` for previous-row comparisons and `LEAD()` for next-row comparisons. Always define the correct partition and ordering for the business problem.