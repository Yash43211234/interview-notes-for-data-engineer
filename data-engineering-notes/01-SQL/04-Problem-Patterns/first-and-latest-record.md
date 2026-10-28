# SQL: Finding the First and Latest Record — Interview Notes

## 1. What Does This Problem Mean?

In SQL interviews, you may need to find:

- The first order placed by each customer.
- The latest transaction for each account.
- The first and most recent login for each user.
- The earliest and latest event for each device.
- The first and latest status of a record.

The correct solution depends on what defines *first* or *latest*, such as a timestamp, date, sequence number, or business priority.

**Most important rule:** Use `ORDER BY` to define which record is first or latest. Do not assume the physical order of rows.

---

## 2. Sample Data

Assume we have an `orders` table:

| order_id | customer_id | order_date | amount |
|---:|---:|---|---:|
| 101 | 1 | 2026-01-05 | 500 |
| 102 | 1 | 2026-01-10 | 700 |
| 103 | 1 | 2026-01-15 | 300 |
| 104 | 2 | 2026-01-03 | 900 |
| 105 | 2 | 2026-01-12 | 600 |
| 106 | 3 | 2026-01-08 | 400 |

We will use this table for the examples below.

## 3. Find the First Record Overall

To find the earliest order in the entire table:

```sql
SELECT *
FROM orders
ORDER BY order_date ASC, order_id ASC
LIMIT 1;
```

Expected result:

| order_id | customer_id | order_date | amount |
|---:|---:|---|---:|
| 104 | 2 | 2026-01-03 | 900 |

### Explanation

- `ASC` sorts from earliest to latest.
- `LIMIT 1` returns only one row.
- `order_id` provides a tie-breaker if multiple orders have the same date.

**Database note:** SQL Server typically uses `TOP (1)` or `OFFSET ... FETCH`; other databases support different row-limiting syntax.

---

## 4. Find the Latest Record Overall

To find the most recent order:

```sql
SELECT *
FROM orders
ORDER BY order_date DESC, order_id DESC
LIMIT 1;
```

Expected result:

| order_id | customer_id | order_date | amount |
|---:|---:|---|---:|
| 103 | 1 | 2026-01-15 | 300 |

### Remember

- `ASC` → earliest first.
- `DESC` → latest first.

This query finds the latest record **overall**, not the latest record for each customer.

---

## 5. Find the First Record for Each Customer

This is a common interview question.

**Question:** Find each customer's first order, including the complete order record.

```sql
WITH ranked_orders AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date ASC, order_id ASC
        ) AS rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM ranked_orders
WHERE rn = 1
ORDER BY customer_id;
```

Expected result:

| order_id | customer_id | order_date | amount |
|---:|---:|---|---:|
| 101 | 1 | 2026-01-05 | 500 |
| 104 | 2 | 2026-01-03 | 900 |
| 106 | 3 | 2026-01-08 | 400 |

### How it works

1. `PARTITION BY customer_id` creates a separate group for each customer.
2. `ORDER BY order_date ASC` places the earliest order first.
3. `ROW_NUMBER()` assigns a sequence number within each customer group.
4. `WHERE rn = 1` retains the first order from each group.

**Interview pattern:** To find the first record per entity, use `ROW_NUMBER()` with ascending order.

---

## 6. Find the Latest Record for Each Customer

**Question:** Find each customer's most recent order, including the amount.

```sql
WITH ranked_orders AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM ranked_orders
WHERE rn = 1
ORDER BY customer_id;
```

Expected result:

| order_id | customer_id | order_date | amount |
|---:|---:|---|---:|
| 103 | 1 | 2026-01-15 | 300 |
| 105 | 2 | 2026-01-12 | 600 |
| 106 | 3 | 2026-01-08 | 400 |

### Key difference from the first-record query

Only the sorting direction changes:

```sql
-- First record
ORDER BY order_date ASC, order_id ASC

-- Latest record
ORDER BY order_date DESC, order_id DESC
```

The partition remains the same.

---

## 7. Find Both the First and Latest Record for Each Customer

Sometimes the interviewer asks for both records in one result.

### Method 1: Use `ROW_NUMBER()` and conditional aggregation

```sql
WITH ranked_orders AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date ASC, order_id ASC
        ) AS first_rn,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS latest_rn
    FROM orders
)
SELECT
    customer_id,
    MAX(CASE WHEN first_rn = 1 THEN order_id END)
        AS first_order_id,
    MAX(CASE WHEN first_rn = 1 THEN order_date END)
        AS first_order_date,
    MAX(CASE WHEN latest_rn = 1 THEN order_id END)
        AS latest_order_id,
    MAX(CASE WHEN latest_rn = 1 THEN order_date END)
        AS latest_order_date
FROM ranked_orders
GROUP BY customer_id
ORDER BY customer_id;
```

Expected result:

| customer_id | first_order_id | first_order_date | latest_order_id | latest_order_date |
|---:|---:|---|---:|---|
| 1 | 101 | 2026-01-05 | 103 | 2026-01-15 |
| 2 | 104 | 2026-01-03 | 105 | 2026-01-12 |
| 3 | 106 | 2026-01-08 | 106 | 2026-01-08 |

### Why use `MAX()`?

The `CASE` expression returns a value only for the row matching the required rank. `MAX()` then selects that value within the customer group.

This works for the selected order IDs and dates. If you need additional columns, include corresponding conditional expressions for them.

---

## 8. Use `FIRST_VALUE()` and `LAST_VALUE()`

Window value functions can also retrieve first and latest values.

### Get the first order date for each customer

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    FIRST_VALUE(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date ASC, order_id ASC
    ) AS first_order_date
FROM orders;
```

This returns every order row, alongside the customer's first order date.

### Get the latest order date for each customer

With `LAST_VALUE()`, explicitly define the window frame:

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    LAST_VALUE(order_date) OVER (
        PARTITION BY customer_id
        ORDER BY order_date ASC, order_id ASC
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND UNBOUNDED FOLLOWING
    ) AS latest_order_date
FROM orders;
```

### Why is the window frame important?

`LAST_VALUE()` often returns the last value in the **current window frame**, which may end at the current row by default. Without the correct frame, it might not return the latest value in the entire customer partition.

**Interview advice:** For selecting the complete first or latest record, `ROW_NUMBER()` is usually easier to explain and safer to apply.

---

## 9. Use `MIN()` and `MAX()` — When Are They Enough?

If the question asks only for the earliest and latest dates per customer:

```sql
SELECT
    customer_id,
    MIN(order_date) AS first_order_date,
    MAX(order_date) AS latest_order_date
FROM orders
GROUP BY customer_id;
```

This is simple and efficient for date-only results.

However, it does **not** identify the complete row containing the earliest or latest date.

For example, this query is not a reliable way to retrieve the amount of the latest order:

```sql
SELECT
    customer_id,
    MAX(order_date) AS latest_order_date,
    amount
FROM orders
GROUP BY customer_id, amount;
```

Grouping by `amount` changes the grouping level, so it may return multiple rows per customer.

**Rule of thumb:**

- Need only the earliest/latest value? Use `MIN()` or `MAX()`.
- Need the complete record? Use `ROW_NUMBER()` or another suitable row-selection technique.

---

## 10. Handle Multiple Records With the Same Timestamp

Suppose a customer has two orders on the same date.

| order_id | customer_id | order_date | amount |
|---:|---:|---|---:|
| 201 | 4 | 2026-02-01 | 300 |
| 202 | 4 | 2026-02-01 | 450 |

If you use:

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER BY order_date ASC
)
```

SQL does not have a complete ordering rule for these two rows. Which row receives `rn = 1` may vary.

Add a tie-breaker:

```sql
ROW_NUMBER() OVER (
    PARTITION BY customer_id
    ORDER BY order_date ASC, order_id ASC
)
```

Now order `201` is first according to the defined rule.

**Important:** A tie-breaker should follow the business requirement. The greatest ID is not automatically the latest event unless the ID's meaning supports that conclusion.

---

## 11. Find the First Record After a Specific Date

**Question:** Find each customer's first order on or after January 10, 2026.

```sql
WITH ranked_orders AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date ASC, order_id ASC
        ) AS rn
    FROM orders
    WHERE order_date >= '2026-01-10'
)
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM ranked_orders
WHERE rn = 1;
```

The filter runs before the window function ranks the remaining rows. Therefore, the query finds the first order **within the filtered period**, not the customer's first order ever.

---

## 12. Common Mistakes

1. **Forgetting `PARTITION BY`:** You get the first or latest record overall instead of one per customer.
2. **Using the wrong sort direction:** `ASC` gives the earliest first; `DESC` gives the latest first.
3. **Using `MIN()` or `MAX()` to retrieve an entire row:** These functions return values, not the complete associated record.
4. **Ignoring ties:** Add a suitable secondary ordering column.
5. **Using `LAST_VALUE()` without checking the window frame:** It may return the last value of the current frame, not the full partition.
6. **Filtering at the wrong stage:** Filtering before ranking changes which records qualify as first or latest.
7. **Confusing first inserted with first occurred:** An ingestion timestamp and an event timestamp can represent different meanings.

---

## 13. Common Interview Questions

**Q1. How do you find the latest order for every customer?**

Use `ROW_NUMBER()` partitioned by `customer_id`, ordered by `order_date DESC`, and retain `rn = 1`.

**Q2. What is the difference between `MIN(order_date)` and `ROW_NUMBER()`?**

`MIN()` returns the earliest date. `ROW_NUMBER()` lets you select a particular row according to a defined ordering rule.

**Q3. How do you find the first and latest login for each user?**

Rank login records in ascending and descending timestamp order, then select rank one for each direction.

**Q4. What if two records have the same timestamp?**

Add a stable tie-breaker, such as a unique event ID, based on the business rule.

**Q5. What is the difference between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()`?**

- `ROW_NUMBER()` assigns a unique number to each row.
- `RANK()` assigns equal ranks to ties and leaves gaps after ties.
- `DENSE_RANK()` assigns equal ranks to ties without gaps.

For exactly one first or latest record per group, `ROW_NUMBER()` with deterministic ordering is generally appropriate.

**Q6. Can `FIRST_VALUE()` return the first value for every row?**

Yes. It returns the first value in the ordered window frame, so each row can display the same first value for its partition.

**Q7. How would you find the first event for each device in a large event table?**

Partition by the device key, order by event timestamp and a suitable tie-breaker, and use `ROW_NUMBER()`. In a distributed system such as Spark, consider data volume, partitioning, and the cost of the window operation.

---

## 14. Practice Questions

Try solving these without looking at the examples.

1. Find the earliest order for each customer.
2. Find the latest order for each customer, including the amount.
3. Find each employee's first and most recent login.
4. Find the earliest transaction for each account after February 1, 2026.
5. Find the first and latest event timestamps for each device using aggregation.
6. Explain why `MAX(event_timestamp)` alone cannot reliably return the complete latest event record.
7. Modify a latest-record query to handle duplicate timestamps deterministically.
8. Explain when you would choose `MIN()`/`MAX()` instead of `ROW_NUMBER()`.

---

## 15. Quick Revision

| Requirement | Best starting point |
|---|---|
| First record overall | `ORDER BY ... ASC` with a row limit |
| Latest record overall | `ORDER BY ... DESC` with a row limit |
| First record per customer | `ROW_NUMBER()` + ascending order |
| Latest record per customer | `ROW_NUMBER()` + descending order |
| First and latest dates only | `MIN()` and `MAX()` |
| First and latest values on every row | `FIRST_VALUE()` / `LAST_VALUE()` |
| Exactly one row when timestamps tie | `ROW_NUMBER()` plus a tie-breaker |

### Core pattern to remember

```sql
WITH ranked AS (
    SELECT
        *,
        ROW_NUMBER() OVER (
            PARTITION BY entity_id
            ORDER BY event_timestamp DESC, unique_id DESC
        ) AS rn
    FROM source_table
)
SELECT *
FROM ranked
WHERE rn = 1;
```

Use `DESC` for the latest record and `ASC` for the first record. Replace the sample columns with the actual business key, timestamp, and tie-breaker.

---

## 16. Self-Check

- [ ] I can find the first record overall.
- [ ] I can find the latest record overall.
- [ ] I can find the first and latest records per entity.
- [ ] I know when `MIN()` and `MAX()` are sufficient.
- [ ] I understand `ROW_NUMBER()` and deterministic ordering.
- [ ] I can explain the window-frame issue with `LAST_VALUE()`.
- [ ] I can solve these problems for orders, logins, transactions, and events.

**Interview takeaway:** First identify the entity, then define what “first” or “latest” means, order the records correctly, and handle ties explicitly.