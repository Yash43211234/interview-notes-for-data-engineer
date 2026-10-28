# SQL Practice — Medium Level

**File:** `07-practice/medium.md`

## 1. How to Practice

These problems focus on SQL patterns commonly tested in Data Engineer and product-company interviews.

For every problem:

1. Understand the expected output.
2. Identify the required tables and columns.
3. Decide whether you need a join, CTE, subquery, aggregation, or window function.
4. Solve it independently before opening the solution.
5. Test edge cases such as duplicate values, NULLs, and customers with no orders.
6. Explain why your query works.

**Recommended time:** 15–25 minutes per problem. If you get stuck, identify the missing concept and try again before checking the solution.

---

## 2. Sample Tables

Use these tables throughout the practice set.

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

### `products`

| product_id | product_name | category | price |
|---:|---|---|---:|
| 1 | Laptop | Electronics | 60000 |
| 2 | Mouse | Electronics | 800 |
| 3 | Chair | Furniture | 5000 |
| 4 | Desk | Furniture | 12000 |
| 5 | Keyboard | Electronics | 1500 |

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

## 3. Window Functions and Ranking

### Problem 1: Rank employees by salary

Rank employees from highest to lowest salary within each department. Employees with equal salaries should receive the same rank, with gaps after ties.

**Concepts:** `RANK()`, `PARTITION BY`

<details>
<summary>Solution</summary>

```sql
SELECT
    employee_name,
    department,
    salary,
    RANK() OVER (
        PARTITION BY department
        ORDER BY salary DESC
    ) AS salary_rank
FROM employees;
```

</details>

### Problem 2: Find the second-highest salary in each department

Return the second-highest **distinct** salary for every department. If a department has fewer than two distinct salaries, return no row for that department.

**Concepts:** `DENSE_RANK()`, CTE

<details>
<summary>Solution</summary>

```sql
WITH ranked_employees AS (
    SELECT
        employee_name,
        department,
        salary,
        DENSE_RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT employee_name, department, salary
FROM ranked_employees
WHERE salary_rank = 2;
```

</details>

`DENSE_RANK()` assigns the same rank to equal salaries without skipping the next rank. This makes it suitable for finding the second distinct salary.

### Problem 3: Find the highest-paid employee in each department

Return every employee tied for the highest salary in their department.

**Concepts:** `RANK()`, CTE

<details>
<summary>Solution</summary>

```sql
WITH ranked_employees AS (
    SELECT
        employee_id,
        employee_name,
        department,
        salary,
        RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT employee_id, employee_name, department, salary
FROM ranked_employees
WHERE salary_rank = 1;
```

</details>

### Problem 4: Calculate a running total of revenue

For each sales date, calculate the cumulative revenue up to that date.

**Concepts:** `SUM() OVER`, running totals

<details>
<summary>Solution</summary>

```sql
SELECT
    sale_date,
    revenue,
    SUM(revenue) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING
                 AND CURRENT ROW
    ) AS running_revenue
FROM daily_sales
ORDER BY sale_date;
```

</details>

The explicit `ROWS` frame makes the calculation proceed row by row. If multiple rows share a date, use a unique ordering column as a tie-breaker when the business requirement calls for a deterministic row-by-row total.

### Problem 5: Compare each day's revenue with the previous available sales day

Display the previous recorded revenue and the difference from the previous recorded row.

**Concepts:** `LAG()`

<details>
<summary>Solution</summary>

```sql
SELECT
    sale_date,
    revenue,
    LAG(revenue) OVER (
        ORDER BY sale_date
    ) AS previous_revenue,
    revenue - LAG(revenue) OVER (
        ORDER BY sale_date
    ) AS revenue_difference
FROM daily_sales
ORDER BY sale_date;
```

</details>

`LAG()` compares rows, not necessarily consecutive calendar dates. For example, January 5 is compared with January 3 because January 4 has no row.

### Problem 6: Find the latest order for each customer

Return the latest order per customer. If multiple orders share the latest date, choose the order with the highest order ID.

**Concepts:** `ROW_NUMBER()`, partitioning

<details>
<summary>Solution</summary>

```sql
WITH ranked_orders AS (
    SELECT
        order_id,
        customer_id,
        amount,
        order_date,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM orders
)
SELECT order_id, customer_id, amount, order_date
FROM ranked_orders
WHERE rn = 1;
```

</details>

### Problem 7: Calculate a three-row moving average

Calculate the average revenue for the current sales row and the two preceding sales rows.

**Concepts:** Window frames, `AVG()`

<details>
<summary>Solution</summary>

```sql
SELECT
    sale_date,
    revenue,
    AVG(revenue) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_average
FROM daily_sales
ORDER BY sale_date;
```

</details>

This is a three-row moving average, not necessarily a three-calendar-day average. Missing dates do not create rows automatically.

---

## 4. Aggregation and Conditional Logic

### Problem 8: Calculate completed and cancelled order totals per customer

Display each customer with separate totals for completed and cancelled orders. Include customers without orders.

**Concepts:** Conditional aggregation, `LEFT JOIN`, `COALESCE`

<details>
<summary>Solution</summary>

```sql
SELECT
    c.customer_id,
    c.customer_name,
    COALESCE(
        SUM(CASE WHEN o.status = 'Completed'
                 THEN o.amount ELSE 0 END),
        0
    ) AS completed_amount,
    COALESCE(
        SUM(CASE WHEN o.status = 'Cancelled'
                 THEN o.amount ELSE 0 END),
        0
    ) AS cancelled_amount
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name;
```

</details>

### Problem 9: Find customers whose completed-order total exceeds the average completed-order total per customer

First calculate each customer's completed-order total, then compare those totals with the average across customers who have at least one completed order.

**Concepts:** CTEs, aggregation, subquery

<details>
<summary>Solution</summary>

```sql
WITH customer_totals AS (
    SELECT
        customer_id,
        SUM(amount) AS total_amount
    FROM orders
    WHERE status = 'Completed'
    GROUP BY customer_id
)
SELECT customer_id, total_amount
FROM customer_totals
WHERE total_amount > (
    SELECT AVG(total_amount)
    FROM customer_totals
);
```

</details>

The average is calculated across customers with at least one completed order, not across individual orders.

### Problem 10: Find departments whose average salary is above the company average

Compare each department's average salary with the average salary of all employees.

**Concepts:** `GROUP BY`, `HAVING`, scalar subquery

<details>
<summary>Solution</summary>

```sql
SELECT
    department,
    AVG(salary) AS department_average
FROM employees
GROUP BY department
HAVING AVG(salary) > (
    SELECT AVG(salary)
    FROM employees
);
```

</details>

### Problem 11: Calculate the percentage of completed orders

Return the total number of orders, the number completed, and the completion percentage.

**Concepts:** Conditional aggregation

<details>
<summary>Solution</summary>

```sql
SELECT
    COUNT(*) AS total_orders,
    SUM(CASE WHEN status = 'Completed'
             THEN 1 ELSE 0 END) AS completed_orders,
    ROUND(
        100.0 * SUM(CASE WHEN status = 'Completed'
                          THEN 1 ELSE 0 END)
        / NULLIF(COUNT(*), 0),
        2
    ) AS completion_percentage
FROM orders;
```

</details>

`NULLIF()` protects against division by zero if the table has no rows.

### Problem 12: Find categories with more than one product and an average price above ₹5,000

**Concepts:** `GROUP BY`, `HAVING`

<details>
<summary>Solution</summary>

```sql
SELECT
    category,
    COUNT(*) AS product_count,
    AVG(price) AS average_price
FROM products
GROUP BY category
HAVING COUNT(*) > 1
   AND AVG(price) > 5000;
```

</details>

---

## 5. Joins and Subqueries

### Problem 13: Find customers who placed orders in both January and March 2024

Return each customer only once.

**Concepts:** Conditional aggregation

<details>
<summary>Solution</summary>

```sql
SELECT customer_id
FROM orders
WHERE order_date >= '2024-01-01'
  AND order_date < '2024-04-01'
GROUP BY customer_id
HAVING SUM(
    CASE WHEN order_date >= '2024-01-01'
           AND order_date < '2024-02-01'
         THEN 1 ELSE 0 END
) > 0
AND SUM(
    CASE WHEN order_date >= '2024-03-01'
           AND order_date < '2024-04-01'
         THEN 1 ELSE 0 END
) > 0;
```

</details>

### Problem 14: Find customers whose every order is completed

Customers with no orders should not be included.

**Concepts:** `GROUP BY`, conditional aggregation

<details>
<summary>Solution</summary>

```sql
SELECT customer_id
FROM orders
GROUP BY customer_id
HAVING COUNT(*) = SUM(
    CASE WHEN status = 'Completed' THEN 1 ELSE 0 END
);
```

</details>

This checks whether every recorded order is completed. Customers with no orders do not appear because the query starts from `orders`.

### Problem 15: Find products priced above their category average

**Concepts:** Correlated subquery

<details>
<summary>Solution</summary>

```sql
SELECT
    p.product_id,
    p.product_name,
    p.category,
    p.price
FROM products p
WHERE p.price > (
    SELECT AVG(p2.price)
    FROM products p2
    WHERE p2.category = p.category
);
```

</details>

The inner query calculates the average price for the current product's category.

### Problem 16: Find employees earning more than their managers

**Concepts:** Self-join

<details>
<summary>Solution</summary>

```sql
SELECT
    e.employee_name,
    e.salary,
    m.employee_name AS manager_name,
    m.salary AS manager_salary
FROM employees e
JOIN employees m
    ON e.manager_id = m.employee_id
WHERE e.salary > m.salary;
```

</details>

### Problem 17: Find customers who have at least one completed order but no cancelled orders

**Concepts:** `EXISTS`, `NOT EXISTS`

<details>
<summary>Solution</summary>

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.status = 'Completed'
)
AND NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
      AND o.status = 'Cancelled'
);
```

</details>

`EXISTS` checks whether a matching row exists. `NOT EXISTS` checks that no matching row exists.

### Problem 18: Find pairs of employees in the same department with equal salaries

Return each pair once, without pairing an employee with themselves.

**Concepts:** Self-join

<details>
<summary>Solution</summary>

```sql
SELECT
    e1.employee_name AS employee_1,
    e2.employee_name AS employee_2,
    e1.department,
    e1.salary
FROM employees e1
JOIN employees e2
    ON e1.department = e2.department
   AND e1.salary = e2.salary
   AND e1.employee_id < e2.employee_id;
```

</details>

Using `<` on employee IDs prevents duplicate pairs and self-pairs.

---

## 6. CTEs and Multi-Step Problems

### Problem 19: Find the top two earners in each department

Return at most two employees per department. Break salary ties using employee ID.

**Concepts:** `ROW_NUMBER()`, CTE

<details>
<summary>Solution</summary>

```sql
WITH ranked_employees AS (
    SELECT
        employee_id,
        employee_name,
        department,
        salary,
        ROW_NUMBER() OVER (
            PARTITION BY department
            ORDER BY salary DESC, employee_id
        ) AS rn
    FROM employees
)
SELECT employee_id, employee_name, department, salary
FROM ranked_employees
WHERE rn <= 2;
```

</details>

If the requirement is to include all employees tied for second place, use a ranking function such as `DENSE_RANK()` instead and clarify how ties should be handled.

### Problem 20: Find customers whose latest order was completed

**Concepts:** `ROW_NUMBER()`, CTE, filtering

<details>
<summary>Solution</summary>

```sql
WITH ranked_orders AS (
    SELECT
        order_id,
        customer_id,
        status,
        order_date,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM orders
)
SELECT customer_id, order_id, status, order_date
FROM ranked_orders
WHERE rn = 1
  AND status = 'Completed';
```

</details>

Rank all orders first and then check the latest order's status. Filtering to completed orders inside the CTE before ranking would answer a different question: customers whose latest *completed* order is selected.

### Problem 21: Find the first order date for every customer

Include only customers who have placed at least one order.

**Concepts:** `MIN`, `GROUP BY`

<details>
<summary>Solution</summary>

```sql
SELECT
    customer_id,
    MIN(order_date) AS first_order_date
FROM orders
GROUP BY customer_id;
```

</details>

### Problem 22: Calculate the difference between each employee's salary and their department average

Display employee name, salary, department average, and difference.

**Concepts:** Window aggregate

<details>
<summary>Solution</summary>

```sql
SELECT
    employee_name,
    department,
    salary,
    AVG(salary) OVER (
        PARTITION BY department
    ) AS department_average,
    salary - AVG(salary) OVER (
        PARTITION BY department
    ) AS difference_from_average
FROM employees;
```

</details>

Unlike `GROUP BY`, a window function returns the aggregate alongside each original employee row.

---

## 7. Date and Sequence Problems

### Problem 23: Find dates with revenue higher than the previous recorded sales date

**Concepts:** `LAG()`, CTE

<details>
<summary>Solution</summary>

```sql
WITH sales_comparison AS (
    SELECT
        sale_date,
        revenue,
        LAG(revenue) OVER (
            ORDER BY sale_date
        ) AS previous_revenue
    FROM daily_sales
)
SELECT sale_date, revenue, previous_revenue
FROM sales_comparison
WHERE revenue > previous_revenue;
```

</details>

### Problem 24: Identify gaps between recorded sales dates

Display each sales date and the number of calendar days since the previous recorded date. Return only gaps greater than one day.

**Concepts:** `LAG()`, date arithmetic

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH sales_gaps AS (
    SELECT
        sale_date,
        LAG(sale_date) OVER (
            ORDER BY sale_date
        ) AS previous_date
    FROM daily_sales
)
SELECT
    previous_date,
    sale_date,
    sale_date - previous_date AS gap_days
FROM sales_gaps
WHERE sale_date - previous_date > 1;
```

</details>

Date subtraction differs across SQL databases. In MySQL, use `DATEDIFF(sale_date, previous_date)` for the day difference.

### Problem 25: Calculate a seven-day revenue total

For each date, calculate revenue over the current date and the preceding six calendar days, including missing dates as zero.

**Concepts:** Date calendar, `LEFT JOIN`, window functions

<details>
<summary>Solution — PostgreSQL</summary>

```sql
WITH calendar AS (
    SELECT generate_series(
        DATE '2024-01-01',
        DATE '2024-01-09',
        INTERVAL '1 day'
    )::date AS sale_date
),
daily_revenue AS (
    SELECT
        c.sale_date,
        COALESCE(s.revenue, 0) AS revenue
    FROM calendar c
    LEFT JOIN daily_sales s
        ON c.sale_date = s.sale_date
)
SELECT
    sale_date,
    revenue,
    SUM(revenue) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS seven_day_revenue
FROM daily_revenue
ORDER BY sale_date;
```

</details>

The calendar ensures every day has a row. Without it, a seven-row window would mean seven recorded rows rather than seven calendar days. This example returns dates from January 1 through January 9; extend the calendar range for a larger reporting period.

---

## 8. Deduplication and Data Quality

### Problem 26: Remove duplicate records logically

Suppose an `events` table has:

- `event_id`
- `user_id`
- `event_type`
- `event_timestamp`

Identify duplicate events defined as the same `user_id`, `event_type`, and `event_timestamp`. Keep the row with the smallest `event_id`.

**Concepts:** `ROW_NUMBER()`, deduplication

<details>
<summary>Solution</summary>

```sql
WITH ranked_events AS (
    SELECT
        event_id,
        user_id,
        event_type,
        event_timestamp,
        ROW_NUMBER() OVER (
            PARTITION BY user_id, event_type, event_timestamp
            ORDER BY event_id
        ) AS rn
    FROM events
)
SELECT event_id, user_id, event_type, event_timestamp
FROM ranked_events
WHERE rn = 1;
```

</details>

This returns one row per defined duplicate group. It does not physically delete any records.

### Problem 27: Find customers with duplicate names

Find names that occur more than once in the `customers` table.

**Concepts:** `GROUP BY`, `HAVING`

<details>
<summary>Solution</summary>

```sql
SELECT customer_name, COUNT(*) AS name_count
FROM customers
GROUP BY customer_name
HAVING COUNT(*) > 1;
```

</details>

In real datasets, consider whether comparisons should ignore case or surrounding spaces.

### Problem 28: Find orders with invalid amounts

Find orders with zero or negative amounts.

**Concepts:** Data validation

<details>
<summary>Solution</summary>

```sql
SELECT *
FROM orders
WHERE amount <= 0;
```

</details>

This assumes `amount` is not NULL. If NULL amounts should also be treated as invalid, include `OR amount IS NULL`.

---

## 9. Try Without Solutions

Attempt these questions independently. Write the SQL before opening any reference.

| No. | Task | Main concept |
|---:|---|---|
| 29 | Find the third-highest distinct salary in each department. | `DENSE_RANK()` |
| 30 | Find customers whose total completed-order amount is at least ₹3,000. | Conditional aggregation |
| 31 | Find employees hired before their managers. | Self-join |
| 32 | Find each customer's percentage contribution to total completed revenue. | Aggregation, window functions |
| 33 | Find the product with the second-highest price in each category. | Ranking |
| 34 | Find customers who placed orders on at least three different dates. | `COUNT(DISTINCT ...)` |
| 35 | Find departments where every employee earns at least ₹60,000. | `GROUP BY`, `HAVING` |
| 36 | Find the longest streak of consecutive sales dates. | Gaps and islands |
| 37 | Find customers who ordered in January but not February. | `EXISTS`, `NOT EXISTS` |
| 38 | Return the first and last order dates per customer. | `MIN`, `MAX` |
| 39 | Calculate month-over-month revenue changes. | Aggregation, `LAG()` |
| 40 | Find employees earning more than the second-highest salary in the company. | Subquery, ranking |

---

## 10. Common Mistakes

### Mistake 1: Filtering a window function in `WHERE`

Incorrect in most SQL databases:

```sql
SELECT
    employee_name,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS rn
FROM employees
WHERE rn = 1;
```

Use a CTE or subquery to calculate the row number first, then filter it in the outer query.

### Mistake 2: Confusing `ROW_NUMBER`, `RANK`, and `DENSE_RANK`

| Function | Tie behavior |
|---|---|
| `ROW_NUMBER()` | Assigns a different number to each row |
| `RANK()` | Equal values share a rank; subsequent ranks can have gaps |
| `DENSE_RANK()` | Equal values share a rank; subsequent ranks have no gaps |

### Mistake 3: Using `COUNT(*)` after a `LEFT JOIN`

`COUNT(*)` counts the preserved customer row even if there is no matching order. Use `COUNT(o.order_id)` to count matching orders.

### Mistake 4: Filtering too early

Filtering completed orders before ranking can change the meaning of a latest-order question. Decide whether the condition applies before or after the ranking operation.

### Mistake 5: Treating missing dates as zero automatically

Window functions operate on rows. Generate a calendar and fill missing dates when a problem requires calendar-day calculations.

### Mistake 6: Ignoring ties

If a problem asks for the top two employees, clarify whether it means exactly two employees or everyone tied within the top two ranks.

---

## 11. Interview Readiness Checklist

- [ ] Rank rows within groups using window functions.
- [ ] Find the Nth-highest distinct value.
- [ ] Calculate running totals and moving averages.
- [ ] Compare current rows with previous rows using `LAG()`.
- [ ] Perform conditional aggregation.
- [ ] Use `EXISTS` and `NOT EXISTS`.
- [ ] Solve correlated subquery problems.
- [ ] Use CTEs to break multi-step problems into stages.
- [ ] Deduplicate data using `ROW_NUMBER()`.
- [ ] Handle missing dates and consecutive-date problems.
- [ ] Explain ties, NULL values, and join behavior.
- [ ] Explain how you would validate query results.

## 12. Recommended Practice Order

1. Problems 1–7: Window functions.
2. Problems 8–12: Aggregation and conditional logic.
3. Problems 13–18: Joins and subqueries.
4. Problems 19–25: CTEs, ranking, and dates.
5. Problems 26–28: Deduplication and data quality.
6. Problems 29–40: Independent interview practice.

**Goal:** Solve the independent problems without hints and explain the logic clearly. Then move to harder patterns such as sessionization, gaps and islands, retention, cohort analysis, and complex multi-table transformations.