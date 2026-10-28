# SQL Practice — Hard Level

**File:** `07-practice/hard.md`

## 1. How to Approach Hard SQL Problems

Hard SQL interview questions often combine multiple concepts. The challenge is usually understanding the required result and breaking the problem into smaller steps.

For every problem:

1. Identify the output grain: one row per customer, day, session, product, or group.
2. Identify the filtering conditions and edge cases.
3. Break the logic into stages using CTEs.
4. Use window functions for ranking, comparisons, and sequences.
5. Check duplicate rows, NULL values, missing dates, and ties.
6. Validate the result with small examples before optimizing.

**Recommended time:** 25–40 minutes per problem.

---

## 2. Sample Tables

Use these tables for the problems below. Some questions introduce additional tables explicitly.

### `customers`

| customer_id | customer_name | city |
|---:|---|---|
| 101 | Aman | Delhi |
| 102 | Riya | Mumbai |
| 103 | Sohan | Delhi |
| 104 | Meera | Bengaluru |
| 105 | Kabir | Mumbai |

### `orders`

| order_id | customer_id | amount | order_date | status |
|---:|---:|---:|---|---|
| 1001 | 101 | 1200 | 2024-01-05 | Completed |
| 1002 | 102 | 2500 | 2024-01-10 | Completed |
| 1003 | 101 | 800 | 2024-02-15 | Cancelled |
| 1004 | 103 | 3000 | 2024-02-20 | Completed |
| 1005 | 102 | 1500 | 2024-03-01 | Pending |
| 1006 | 104 | 4000 | 2024-03-12 | Completed |
| 1007 | 101 | 2000 | 2024-03-15 | Completed |

### `employees`

| employee_id | employee_name | department | salary | manager_id | hire_date |
|---:|---|---|---:|---:|---|
| 1 | Amit | IT | 70000 | 4 | 2023-01-15 |
| 2 | Priya | HR | 50000 | 5 | 2022-06-10 |
| 3 | Rahul | IT | 85000 | 4 | 2021-03-20 |
| 4 | Neha | IT | 100000 | NULL | 2020-08-12 |
| 5 | Karan | HR | 60000 | NULL | 2019-11-05 |
| 6 | Sneha | Finance | 65000 | 5 | 2024-02-01 |
| 7 | Arjun | Finance | 75000 | 5 | 2022-09-18 |
| 8 | Pooja | IT | 70000 | 4 | 2023-07-25 |

### `events`

| event_id | user_id | event_type | event_timestamp |
|---:|---:|---|---|
| 1 | 101 | login | 2024-01-01 09:00:00 |
| 2 | 101 | click | 2024-01-01 09:05:00 |
| 3 | 101 | logout | 2024-01-01 09:20:00 |
| 4 | 101 | login | 2024-01-01 10:00:00 |
| 5 | 102 | login | 2024-01-01 09:10:00 |
| 6 | 102 | click | 2024-01-01 09:25:00 |
| 7 | 102 | login | 2024-01-02 11:00:00 |
| 8 | 103 | login | 2024-01-01 12:00:00 |

### `daily_sales`

| sale_date | revenue |
|---|---:|
| 2024-01-01 | 1000 |
| 2024-01-02 | 1500 |
| 2024-01-03 | 1200 |
| 2024-01-05 | 2000 |
| 2024-01-06 | 1800 |
| 2024-01-07 | 2200 |
| 2024-01-09 | 1700 |

---

## 3. Advanced Ranking and Window Functions

### Problem 1: Find the top three customers by completed revenue

Return the top three customers based on their total completed-order amount. Include ties at the third rank.

**Concepts:** Aggregation, `RANK()`, CTEs

<details>
<summary>Solution</summary>

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS total_revenue
    FROM orders
    WHERE status = 'Completed'
    GROUP BY customer_id
),
ranked_customers AS (
    SELECT
        customer_id,
        total_revenue,
        RANK() OVER (
            ORDER BY total_revenue DESC
        ) AS revenue_rank
    FROM customer_revenue
)
SELECT customer_id, total_revenue, revenue_rank
FROM ranked_customers
WHERE revenue_rank <= 3
ORDER BY revenue_rank, customer_id;
```

</details>

**Important:** `RANK() <= 3` includes ties at the third rank. It does not necessarily return exactly three rows.

### Problem 2: Find employees earning more than their department's second-highest distinct salary

Return employees whose salary is strictly greater than the second-highest distinct salary in their department.

**Concepts:** `DENSE_RANK()`, CTE

<details>
<summary>Solution</summary>

```sql
WITH ranked_employees AS (
    SELECT
        employee_id,
        employee_name,
        department,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT employee_id, employee_name, department, salary
FROM ranked_employees
WHERE salary_rank < 2;
```

</details>

The highest salary has dense rank 1. This query returns those highest-paid employees, whose salaries are greater than the second-highest distinct salary. Departments with fewer than two distinct salaries will still return their highest-paid employee; add a department-level distinct-salary count if such departments must be excluded.

### Problem 3: Calculate each customer's percentage contribution to completed revenue

Display each customer, their completed revenue, and their percentage of total completed revenue. Include only customers with completed orders.

**Concepts:** Aggregation, window functions

<details>
<summary>Solution</summary>

```sql
WITH customer_revenue AS (
    SELECT
        customer_id,
        SUM(amount) AS total_revenue
    FROM orders
    WHERE status = 'Completed'
    GROUP BY customer_id
)
SELECT
    customer_id,
    total_revenue,
    ROUND(
        100.0 * total_revenue
        / NULLIF(SUM(total_revenue) OVER (), 0),
        2
    ) AS revenue_percentage
FROM customer_revenue
ORDER BY total_revenue DESC;
```

</details>

### Problem 4: Find consecutive salary increases in employee records

Suppose an additional table `salary_history` contains:

`employee_id`, `effective_date`, `salary`.

Find employees whose salary increased in at least three consecutive recorded changes.

**Concepts:** `LAG()`, grouping sequences

<details>
<summary>Solution</summary>

```sql
WITH salary_changes AS (
    SELECT
        employee_id,
        effective_date,
        salary,
        LAG(salary) OVER (
            PARTITION BY employee_id
            ORDER BY effective_date
        ) AS previous_salary
    FROM salary_history
),
increase_flags AS (
    SELECT
        employee_id,
        effective_date,
        CASE
            WHEN salary > previous_salary THEN 1
            ELSE 0
        END AS increased
    FROM salary_changes
),
streaks AS (
    SELECT
        employee_id,
        effective_date,
        increased,
        SUM(
            CASE WHEN increased = 0 THEN 1 ELSE 0 END
        ) OVER (
            PARTITION BY employee_id
            ORDER BY effective_date
            ROWS UNBOUNDED PRECEDING
        ) AS streak_group
    FROM increase_flags
)
SELECT employee_id
FROM streaks
WHERE increased = 1
GROUP BY employee_id, streak_group
HAVING COUNT(*) >= 3;
```

</details>

This counts three consecutive increases across recorded changes. Ensure `effective_date` uniquely orders changes per employee; if multiple changes can share a date, add a stable tie-breaker.

---

## 4. Gaps, Islands, and Consecutive Dates

### Problem 5: Find the longest streak of consecutive sales dates

Return the longest run of calendar dates present in `daily_sales`, along with its start and end dates.

**Concepts:** Gaps and islands, `ROW_NUMBER()`

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH numbered_dates AS (
    SELECT
        sale_date,
        sale_date
          - ROW_NUMBER() OVER (
                ORDER BY sale_date
            )::int AS island_key
    FROM daily_sales
),
streaks AS (
    SELECT
        MIN(sale_date) AS start_date,
        MAX(sale_date) AS end_date,
        COUNT(*) AS streak_length
    FROM numbered_dates
    GROUP BY island_key
),
ranked_streaks AS (
    SELECT
        start_date,
        end_date,
        streak_length,
        ROW_NUMBER() OVER (
            ORDER BY streak_length DESC, start_date
        ) AS rn
    FROM streaks
)
SELECT start_date, end_date, streak_length
FROM ranked_streaks
WHERE rn = 1;
```

</details>

The key idea is that subtracting the row number from consecutive dates produces the same grouping key for each uninterrupted run.

### Problem 6: Find users who logged in on three consecutive calendar days

Suppose an additional `user_activity` table contains `user_id` and `activity_date`. There may be multiple activity rows per user per day.

Find users who have at least one login on three consecutive dates. Assume `event_type` is also available and must equal `'login'`.

**Concepts:** Deduplication, `LAG()`, date arithmetic

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH login_days AS (
    SELECT DISTINCT
        user_id,
        CAST(event_timestamp AS date) AS login_date
    FROM events
    WHERE event_type = 'login'
),
numbered_logins AS (
    SELECT
        user_id,
        login_date,
        LAG(login_date, 1) OVER (
            PARTITION BY user_id
            ORDER BY login_date
        ) AS previous_date,
        LAG(login_date, 2) OVER (
            PARTITION BY user_id
            ORDER BY login_date
        ) AS two_dates_before
    FROM login_days
)
SELECT DISTINCT user_id
FROM numbered_logins
WHERE login_date = previous_date + 1
  AND previous_date = two_dates_before + 1;
```

</details>

The `DISTINCT` step prevents multiple logins on one day from being mistaken for consecutive days.

### Problem 7: Identify missing dates in a sales calendar

Find dates between January 1 and January 9, 2024, that do not exist in `daily_sales`.

**Concepts:** Calendar generation, anti-join

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH calendar AS (
    SELECT generate_series(
        DATE '2024-01-01',
        DATE '2024-01-09',
        INTERVAL '1 day'
    )::date AS sale_date
)
SELECT c.sale_date AS missing_date
FROM calendar c
LEFT JOIN daily_sales s
    ON c.sale_date = s.sale_date
WHERE s.sale_date IS NULL
ORDER BY c.sale_date;
```

</details>

This assumes `daily_sales` has at most one row per date. If it contains duplicate dates, first aggregate to one row per date.

---

## 5. Sessionization and Event Data

### Problem 8: Assign session IDs to user events

A new session starts when the gap between consecutive events for the same user is greater than 30 minutes.

Return each event with its session number. Use `event_id` as a tie-breaker when timestamps are equal.

**Concepts:** `LAG()`, cumulative sum

<details>
<summary>Solution</summary>

```sql
WITH previous_events AS (
    SELECT
        event_id,
        user_id,
        event_type,
        event_timestamp,
        LAG(event_timestamp) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp, event_id
        ) AS previous_timestamp
    FROM events
),
session_flags AS (
    SELECT
        *,
        CASE
            WHEN previous_timestamp IS NULL THEN 1
            WHEN event_timestamp >
                 previous_timestamp + INTERVAL '30 minutes'
                THEN 1
            ELSE 0
        END AS new_session
    FROM previous_events
)
SELECT
    event_id,
    user_id,
    event_type,
    event_timestamp,
    SUM(new_session) OVER (
        PARTITION BY user_id
        ORDER BY event_timestamp, event_id
        ROWS UNBOUNDED PRECEDING
    ) AS session_id
FROM session_flags
ORDER BY user_id, event_timestamp, event_id;
```

</details>

This syntax uses PostgreSQL interval arithmetic. Other databases may use different date-add functions. The comparison uses `>` so an event exactly 30 minutes later stays in the same session; use `>=` if the rule says 30 minutes or more starts a new session.

### Problem 9: Calculate session duration and event count

Using the same 30-minute session rule, return one row per user session with its first event time, last event time, event count, and duration in minutes.

**Concepts:** Sessionization, aggregation

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH previous_events AS (
    SELECT
        *,
        LAG(event_timestamp) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp, event_id
        ) AS previous_timestamp
    FROM events
),
session_flags AS (
    SELECT
        *,
        CASE
            WHEN previous_timestamp IS NULL THEN 1
            WHEN event_timestamp >
                 previous_timestamp + INTERVAL '30 minutes'
                THEN 1
            ELSE 0
        END AS new_session
    FROM previous_events
),
sessioned_events AS (
    SELECT
        *,
        SUM(new_session) OVER (
            PARTITION BY user_id
            ORDER BY event_timestamp, event_id
            ROWS UNBOUNDED PRECEDING
        ) AS session_id
    FROM session_flags
)
SELECT
    user_id,
    session_id,
    MIN(event_timestamp) AS session_start,
    MAX(event_timestamp) AS session_end,
    COUNT(*) AS event_count,
    EXTRACT(
        EPOCH FROM MAX(event_timestamp) - MIN(event_timestamp)
    ) / 60.0 AS duration_minutes
FROM sessioned_events
GROUP BY user_id, session_id
ORDER BY user_id, session_id;
```

</details>

The duration is the difference between the first and last event timestamps. A session containing one event has a duration of zero.

### Problem 10: Find users who returned within seven days of their first login

Return each user whose next login occurred within seven calendar days of their first login, including the seventh day.

**Concepts:** `MIN()`, `LEAD()`, date filtering

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH login_days AS (
    SELECT DISTINCT
        user_id,
        CAST(event_timestamp AS date) AS login_date
    FROM events
    WHERE event_type = 'login'
),
first_login AS (
    SELECT
        user_id,
        MIN(login_date) AS first_login_date
    FROM login_days
    GROUP BY user_id
),
next_login AS (
    SELECT
        user_id,
        login_date,
        LEAD(login_date) OVER (
            PARTITION BY user_id
            ORDER BY login_date
        ) AS next_login_date
    FROM login_days
)
SELECT DISTINCT f.user_id
FROM first_login f
JOIN next_login n
    ON f.user_id = n.user_id
   AND n.login_date = f.first_login_date
WHERE n.next_login_date <= f.first_login_date + 7;
```

</details>

This uses calendar dates rather than exact timestamps. If the business requirement is precisely 168 hours, compare timestamps instead.

---

## 6. Retention and Cohort Analysis

### Problem 11: Calculate monthly customer retention

Suppose an additional `customer_activity` table contains `customer_id` and `activity_date`.

Define a customer's cohort as the month of their first activity. Calculate how many customers from each cohort were active in each subsequent month, and the retention percentage.

**Concepts:** Cohorts, distinct counts, month arithmetic

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH first_activity AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', MIN(activity_date))::date AS cohort_month
    FROM customer_activity
    GROUP BY customer_id
),
monthly_activity AS (
    SELECT DISTINCT
        customer_id,
        DATE_TRUNC('month', activity_date)::date AS activity_month
    FROM customer_activity
),
cohort_activity AS (
    SELECT
        f.cohort_month,
        a.activity_month,
        (
            EXTRACT(YEAR FROM AGE(a.activity_month, f.cohort_month)) * 12
            + EXTRACT(MONTH FROM AGE(a.activity_month, f.cohort_month))
        )::int AS month_number,
        COUNT(DISTINCT f.customer_id) AS active_customers
    FROM first_activity f
    JOIN monthly_activity a
        ON f.customer_id = a.customer_id
    GROUP BY f.cohort_month, a.activity_month
),
cohort_sizes AS (
    SELECT
        cohort_month,
        COUNT(*) AS cohort_size
    FROM first_activity
    GROUP BY cohort_month
)
SELECT
    a.cohort_month,
    a.activity_month,
    a.month_number,
    a.active_customers,
    s.cohort_size,
    ROUND(
        100.0 * a.active_customers / NULLIF(s.cohort_size, 0),
        2
    ) AS retention_percentage
FROM cohort_activity a
JOIN cohort_sizes s
    ON a.cohort_month = s.cohort_month
ORDER BY a.cohort_month, a.activity_month;
```

</details>

This reports observed cohort-month combinations. Months with zero active customers will not appear unless you generate a calendar of cohort-month combinations and fill missing counts with zero.

### Problem 12: Calculate day-one retention

A customer is retained on day one if they return on the calendar day immediately after their first activity date.

**Concepts:** First activity, `EXISTS`, date arithmetic

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH first_activity AS (
    SELECT
        customer_id,
        MIN(activity_date::date) AS first_date
    FROM customer_activity
    GROUP BY customer_id
)
SELECT
    COUNT(*) AS cohort_size,
    SUM(
        CASE
            WHEN EXISTS (
                SELECT 1
                FROM customer_activity a
                WHERE a.customer_id = f.customer_id
                  AND a.activity_date::date = f.first_date + 1
            )
            THEN 1 ELSE 0
        END
    ) AS retained_customers,
    ROUND(
        100.0 * SUM(
            CASE
                WHEN EXISTS (
                    SELECT 1
                    FROM customer_activity a
                    WHERE a.customer_id = f.customer_id
                      AND a.activity_date::date = f.first_date + 1
                )
                THEN 1 ELSE 0
            END
        ) / NULLIF(COUNT(*), 0),
        2
    ) AS day_one_retention_percentage
FROM first_activity f;
```

</details>

---

## 7. Complex Aggregation and Business Logic

### Problem 13: Find customers whose monthly spending increased for three consecutive months

Suppose `orders` contains completed orders over multiple months. First aggregate completed revenue per customer per month. Then find customers whose revenue increased in three consecutive month-to-month comparisons.

**Concepts:** Monthly aggregation, `LAG()`, sequence detection

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH monthly_revenue AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', order_date)::date AS order_month,
        SUM(amount) AS revenue
    FROM orders
    WHERE status = 'Completed'
    GROUP BY customer_id, DATE_TRUNC('month', order_date)
),
monthly_changes AS (
    SELECT
        customer_id,
        order_month,
        revenue,
        LAG(order_month) OVER (
            PARTITION BY customer_id
            ORDER BY order_month
        ) AS previous_month,
        LAG(revenue) OVER (
            PARTITION BY customer_id
            ORDER BY order_month
        ) AS previous_revenue
    FROM monthly_revenue
),
increase_flags AS (
    SELECT
        *,
        CASE
            WHEN previous_month = order_month - INTERVAL '1 month'
             AND revenue > previous_revenue
            THEN 1 ELSE 0
        END AS increased
    FROM monthly_changes
),
streaks AS (
    SELECT
        *,
        SUM(
            CASE WHEN increased = 0 THEN 1 ELSE 0 END
        ) OVER (
            PARTITION BY customer_id
            ORDER BY order_month
            ROWS UNBOUNDED PRECEDING
        ) AS streak_group
    FROM increase_flags
)
SELECT customer_id
FROM streaks
WHERE increased = 1
GROUP BY customer_id, streak_group
HAVING COUNT(*) >= 3;
```

</details>

Three consecutive increases require four consecutive monthly revenue observations. The query does not treat a missing month as an increase.

### Problem 14: Find the median salary in each department

Return the median salary for every department.

**Concepts:** Ordered-set aggregates

<details>
<summary>Solution — PostgreSQL</summary>

```sql
SELECT
    department,
    PERCENTILE_CONT(0.5)
        WITHIN GROUP (ORDER BY salary) AS median_salary
FROM employees
GROUP BY department;
```

</details>

`PERCENTILE_CONT` can interpolate between two middle values. Some databases use different syntax or require a window-function approach.

### Problem 15: Find the products that account for the first 80% of cumulative revenue

Suppose a table `product_sales` contains `product_id` and `revenue`. Sort products by revenue descending and find products whose cumulative revenue share is at most 80%, plus the first product that crosses the 80% threshold.

**Concepts:** Running totals, cumulative percentages

<details>
<summary>Solution</summary>

```sql
WITH product_totals AS (
    SELECT
        product_id,
        SUM(revenue) AS total_revenue
    FROM product_sales
    GROUP BY product_id
),
cumulative_revenue AS (
    SELECT
        product_id,
        total_revenue,
        SUM(total_revenue) OVER (
            ORDER BY total_revenue DESC, product_id
            ROWS UNBOUNDED PRECEDING
        ) AS running_revenue,
        SUM(total_revenue) OVER () AS overall_revenue
    FROM product_totals
),
shares AS (
    SELECT
        *,
        running_revenue / NULLIF(overall_revenue, 0) AS cumulative_share,
        LAG(
            running_revenue / NULLIF(overall_revenue, 0),
            1,
            0
        ) OVER (
            ORDER BY total_revenue DESC, product_id
        ) AS previous_share
    FROM cumulative_revenue
)
SELECT product_id, total_revenue, cumulative_share
FROM shares
WHERE previous_share < 0.80
ORDER BY total_revenue DESC, product_id;
```

</details>

This includes each product whose preceding cumulative share was below 80%, so the first product that crosses the threshold is included. It assumes nonnegative revenue and a positive overall total.

---

## 8. Advanced Data Quality Problems

### Problem 16: Detect duplicate records and keep the latest record

Suppose `customer_updates` contains:

- `record_id`
- `customer_id`
- `email`
- `updated_at`

Keep only the latest record for each customer. If timestamps tie, keep the largest `record_id`.

**Concepts:** Deduplication, `ROW_NUMBER()`

<details>
<summary>Solution</summary>

```sql
WITH ranked_updates AS (
    SELECT
        record_id,
        customer_id,
        email,
        updated_at,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY updated_at DESC, record_id DESC
        ) AS rn
    FROM customer_updates
)
SELECT record_id, customer_id, email, updated_at
FROM ranked_updates
WHERE rn = 1;
```

</details>

### Problem 17: Find records with missing or invalid customer IDs

Suppose `staging_orders` contains `order_id`, `customer_id`, and `amount`. Find rows with a missing customer ID, a nonpositive amount, or both.

**Concepts:** NULL handling, validation

<details>
<summary>Solution</summary>

```sql
SELECT *
FROM staging_orders
WHERE customer_id IS NULL
   OR amount IS NULL
   OR amount <= 0;
```

</details>

### Problem 18: Compare source and target order totals

Suppose `source_orders` and `target_orders` contain `order_id` and `amount`.

Find order IDs whose amounts differ between source and target, including IDs that exist on only one side.

**Concepts:** `FULL OUTER JOIN`, NULL-safe comparison

<details>
<summary>Solution — PostgreSQL</summary>

```sql
SELECT
    COALESCE(s.order_id, t.order_id) AS order_id,
    s.amount AS source_amount,
    t.amount AS target_amount
FROM source_orders s
FULL OUTER JOIN target_orders t
    ON s.order_id = t.order_id
WHERE s.order_id IS NULL
   OR t.order_id IS NULL
   OR s.amount IS DISTINCT FROM t.amount;
```

</details>

`IS DISTINCT FROM` compares values while treating NULL as a comparable value. For databases that do not support it, use an explicit NULL-safe comparison.

---

## 9. Independent Hard Practice

Try these without looking at the earlier solutions.

| No. | Problem | Main concepts |
|---:|---|---|
| 19 | Find the longest consecutive login streak for each user. | Gaps and islands |
| 20 | Find the first order after a gap of at least 90 days for each customer. | `LAG()`, date arithmetic |
| 21 | Calculate weekly active users and week-over-week growth. | Date truncation, aggregation, `LAG()` |
| 22 | Find customers who purchased in every month of a specified quarter. | Conditional aggregation |
| 23 | Find employees whose salary exceeds their manager's and the department average. | Self-join, window aggregate |
| 24 | Find the top three products per category by monthly revenue. | Aggregation, `DENSE_RANK()` |
| 25 | Calculate a rolling 30-calendar-day revenue total with missing dates filled as zero. | Calendar table, window functions |
| 26 | Identify sessions where a user performed a purchase event without a preceding login in the same session. | Sessionization, event ordering |
| 27 | Find customers whose completed revenue is greater than every customer's average completed revenue. | Multi-stage aggregation |
| 28 | Find source records missing from the target and target records missing from the source. | Anti-joins, reconciliation |
| 29 | Find changes in customer attributes between consecutive snapshots. | `LAG()`, change detection |
| 30 | Calculate customer retention by cohort month and activity month, including zero-activity periods. | Cohorts, calendar generation |

---

## 10. Common Mistakes in Hard SQL

### 1. Filtering before calculating a window function

If you filter rows before applying `LAG()` or ranking, the comparison or ranking may change. First determine whether the filter should apply before or after the window calculation.

### 2. Ignoring the output grain

If the required output is one row per customer-month, aggregate to that level before joining additional tables. Joining raw orders to raw events can multiply rows and inflate totals.

### 3. Confusing rows with calendar days

A seven-row window is not necessarily seven days. Generate a calendar when the business requirement refers to consecutive dates or calendar periods.

### 4. Failing to handle ties

Choose the appropriate ranking function and include a deterministic tie-breaker when the requirement calls for exactly one row.

### 5. Ignoring NULL values

Remember that comparisons such as `amount <> 0` do not match NULL amounts. Use `IS NULL`, `COALESCE`, or a NULL-safe comparison where appropriate.

### 6. Using the wrong date boundary

For timestamps, prefer half-open ranges when filtering a full period:

```sql
WHERE event_timestamp >= '2024-01-01'
  AND event_timestamp <  '2024-02-01'
```

This includes all timestamps in January without relying on the final day's precision.

### 7. Writing one giant query

Use CTEs to make complex logic easier to understand, validate, and debug. CTEs improve readability but do not automatically guarantee better performance.

---

## 11. Performance and Validation Checklist

For large datasets, ask yourself:

- Can filters be applied before expensive joins or aggregations without changing the result?
- Are join keys indexed or otherwise efficiently partitioned?
- Could a join multiply records?
- Does the query scan the same large table repeatedly?
- Is the ordering required by a window function expensive?
- Can the execution plan identify a full scan, shuffle, or expensive sort?
- Have I checked row counts and aggregate totals before and after each transformation?
- Are duplicate keys or NULL values handled according to the business rule?

For Spark SQL and Databricks, also consider partition pruning, shuffle volume, skewed keys, data-file sizes, and whether the execution plan shows unnecessary exchanges.

---

## 12. Recommended Study Order

1. **Ranking and windows:** Problems 1–4.
2. **Gaps and consecutive dates:** Problems 5–7.
3. **Event processing and sessions:** Problems 8–10.
4. **Retention and cohorts:** Problems 11–12.
5. **Business analytics:** Problems 13–15.
6. **Data quality and reconciliation:** Problems 16–18.
7. **Independent interview practice:** Problems 19–30.

**Readiness goal:** You should be able to break a complex question into logical stages, write the SQL without copying a solution, explain edge cases, and validate the result. For Data Engineer interviews, prioritize sessionization, deduplication, incremental-data logic, window functions, cohort analysis, and source-to-target reconciliation.