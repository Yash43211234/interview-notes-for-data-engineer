# Common SQL Patterns for Data Engineer Interviews

## 1. Why Learn SQL Patterns?

Many SQL interview questions look different but use the same underlying logic. Instead of memorizing every solution, learn to recognize the pattern required by the problem.

For Data Engineer interviews, these patterns are especially useful for:
- Finding top records within groups.
- Removing duplicate records.
- Comparing current and previous records.
- Finding missing or unmatched data.
- Calculating cumulative values.
- Identifying consecutive dates or events.
- Finding customers who meet multiple conditions.

**The goal:** Read a problem, identify its pattern, and build the query step by step.

## 2. Pattern: Find the Highest or Lowest Value

**Use when:** You need the highest salary, largest order, latest date, or minimum price.

### Example: Find the highest salary

```sql
SELECT MAX(salary) AS highest_salary
FROM employees;
```

### Example: Return employees earning the highest salary

```sql
SELECT employee_id, employee_name, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
);
```

The second query returns every employee tied for the highest salary.

**Remember:** `MAX()` returns a value. To retrieve the entire row associated with that value, use a subquery, join, or ranking function.

## 3. Pattern: Find the Nth-Highest Distinct Value

**Use when:** You need the second-highest salary, third-highest order amount, or another distinct rank.

### Example: Find the second-highest distinct salary

```sql
WITH ranked_salaries AS (
    SELECT
        employee_id,
        employee_name,
        salary,
        DENSE_RANK() OVER (
            ORDER BY salary DESC
        ) AS salary_rank
    FROM employees
)
SELECT employee_id, employee_name, salary
FROM ranked_salaries
WHERE salary_rank = 2;
```

### Why `DENSE_RANK()`?

Suppose salaries are:

`100000, 90000, 90000, 80000`

Their dense ranks are:

`1, 2, 2, 3`

The second-highest distinct salary is 90000.

**Important:** If the problem asks for the second row rather than the second distinct value, the appropriate approach may be `ROW_NUMBER()` instead.

## 4. Pattern: Find Top N Records per Group

**Use when:** You need the top three products per category or the highest-paid employees in each department.

### Example: Top two salary levels per department

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
WHERE salary_rank <= 2;
```

`PARTITION BY department` restarts the ranking for each department.

Choose the ranking function based on the requirement:

- `ROW_NUMBER()` — at most N rows per group, with deterministic ordering.
- `RANK()` — ranking positions with gaps after ties.
- `DENSE_RANK()` — top N distinct value levels without gaps.

## 5. Pattern: Remove Duplicates and Keep One Record

**Use when:** A data pipeline receives repeated records and you need to retain one record per business key.

Suppose `customer_updates` contains:

| customer_id | customer_name | updated_at |
|---|---|---|
| C101 | Amit | 2026-01-01 10:00 |
| C101 | Amit Kumar | 2026-01-03 12:00 |
| C102 | Priya | 2026-01-02 09:00 |

### Keep the latest record per customer

```sql
WITH ranked_updates AS (
    SELECT
        customer_id,
        customer_name,
        updated_at,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY updated_at DESC, customer_name
        ) AS row_num
    FROM customer_updates
)
SELECT customer_id, customer_name, updated_at
FROM ranked_updates
WHERE row_num = 1;
```

This keeps one latest record per customer.

**Production note:** If timestamps can tie, use a stable unique column, such as `update_id`, as the tie-breaker. The example uses `customer_name` only to make the ordering explicit; a unique update identifier is preferable when available.

### Important distinction

- `DISTINCT` removes duplicate combinations of selected column values.
- `ROW_NUMBER()` lets you choose which record to keep according to a business rule.

## 6. Pattern: Find Records That Do Not Have a Match

**Use when:** You need customers without orders, products without sales, or source records missing from a target table.

### Example: Customers without orders

Assume `customers.customer_id` is the customer key and `orders.customer_id` identifies the customer who placed each order.

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

`NOT EXISTS` returns customers for whom no matching order exists.

This is often a good pattern for finding unmatched records in data validation.

## 7. Pattern: Find Records That Have a Match

**Use when:** You need customers who have placed at least one order.

```sql
SELECT
    c.customer_id,
    c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
);
```

`EXISTS` checks whether at least one matching row exists.

It does not multiply customer rows when a customer has multiple orders.

## 8. Pattern: Find Groups That Meet a Condition

**Use when:** You need customers with more than five orders or departments with an average salary above a threshold.

### Example: Customers with at least three orders

```sql
SELECT
    customer_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) >= 3;
```

Use:
- `WHERE` to filter individual rows before grouping.
- `HAVING` to filter groups after aggregation.

### Example: Customers who spent more than 10000

```sql
SELECT
    customer_id,
    SUM(amount) AS total_spent
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 10000;
```

This calculates total spending per customer and returns only customers whose total exceeds 10000.

## 9. Pattern: Conditional Aggregation

**Use when:** You need multiple metrics in one query, such as successful and failed transactions per customer.

Assume `transactions` contains `customer_id`, `status`, and `amount`.

```sql
SELECT
    customer_id,
    COUNT(*) AS total_transactions,
    SUM(CASE
        WHEN status = 'SUCCESS' THEN 1
        ELSE 0
    END) AS successful_transactions,
    SUM(CASE
        WHEN status = 'FAILED' THEN 1
        ELSE 0
    END) AS failed_transactions,
    SUM(CASE
        WHEN status = 'SUCCESS' THEN amount
        ELSE 0
    END) AS successful_amount
FROM transactions
GROUP BY customer_id;
```

This produces several metrics for each customer in one pass through the logical query.

**Important:** Decide how `NULL` values should be treated. For example, `COUNT(*)` counts all rows, while `COUNT(amount)` counts only rows where `amount` is not `NULL`.

## 10. Pattern: Calculate Running Totals

**Use when:** You need cumulative revenue, account balance, or quantities over time.

```sql
SELECT
    sale_date,
    sales,
    SUM(sales) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM daily_sales
ORDER BY sale_date;
```

The calculation starts at the first row and adds each row's sales to the previous cumulative total.

For separate totals per store, add:

```sql
PARTITION BY store_id
```

before `ORDER BY`.

Use a stable tie-breaker when multiple records have the same date and their sequence matters.

## 11. Pattern: Compare Current and Previous Rows

**Use when:** You need to identify changes in sales, prices, balances, or event values.

```sql
SELECT
    sale_date,
    sales,
    LAG(sales) OVER (
        ORDER BY sale_date
    ) AS previous_sales,
    sales - LAG(sales) OVER (
        ORDER BY sale_date
    ) AS sales_difference
FROM daily_sales
ORDER BY sale_date;
```

The first row has no previous row, so the previous value is `NULL` by default.

### Find rows where sales increased

```sql
WITH sales_comparison AS (
    SELECT
        sale_date,
        sales,
        LAG(sales) OVER (
            ORDER BY sale_date
        ) AS previous_sales
    FROM daily_sales
)
SELECT sale_date, sales, previous_sales
FROM sales_comparison
WHERE sales > previous_sales;
```

This compares each row with the preceding row in the specified ordering.

**Important:** `LAG()` moves between rows, not necessarily consecutive calendar dates. If a date is missing, the previous row may be several days earlier.

## 12. Pattern: Find Consecutive Dates

**Use when:** You need to identify consecutive login days, sales days, or activity periods.

A common approach uses `LAG()` to compare each date with the preceding date.

Assume `daily_activity` has one row per user per activity date.

```sql
WITH previous_dates AS (
    SELECT
        user_id,
        activity_date,
        LAG(activity_date) OVER (
            PARTITION BY user_id
            ORDER BY activity_date
        ) AS previous_date
    FROM daily_activity
)
SELECT user_id, activity_date, previous_date
FROM previous_dates
WHERE activity_date = previous_date + INTERVAL '1 day';
```

This syntax is PostgreSQL-style. Date arithmetic differs across databases.

The query finds dates that immediately follow the previous recorded activity date for the same user.

**Important:** If a user has multiple activity rows on the same date, deduplicate to one row per user per date first. Otherwise, repeated dates can affect the comparison.

## 13. Pattern: Find Missing Values or Data Quality Issues

**Use when:** You need to identify records with missing required fields.

```sql
SELECT *
FROM customers
WHERE customer_id IS NULL
   OR customer_name IS NULL;
```

To find duplicate business keys:

```sql
SELECT
    customer_id,
    COUNT(*) AS record_count
FROM customers
GROUP BY customer_id
HAVING COUNT(*) > 1;
```

These are useful validation queries before or after an ETL/ELT pipeline loads data.

**Important:** Duplicate keys do not always mean duplicate records. Confirm the intended business key and whether multiple records are valid.

## 14. Pattern: Find Values Above the Group Average

**Use when:** You need employees earning more than their department's average salary.

```sql
WITH employee_averages AS (
    SELECT
        employee_id,
        employee_name,
        department,
        salary,
        AVG(salary) OVER (
            PARTITION BY department
        ) AS department_avg_salary
    FROM employees
)
SELECT
    employee_id,
    employee_name,
    department,
    salary,
    department_avg_salary
FROM employee_averages
WHERE salary > department_avg_salary;
```

The window function calculates the department average without removing individual employee rows.

An alternative is to calculate the average in a grouped subquery and join it back to the employees table.

## 15. Pattern: Use a Self Join to Compare Related Rows

**Use when:** You need to compare rows in the same table, such as employees and their managers.

```sql
SELECT
    e.employee_name AS employee,
    m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

The table is referenced twice using different aliases:
- `e` represents the employee.
- `m` represents the manager.

A `LEFT JOIN` preserves employees even when they do not have a manager.

Use a self join when the relationship between two rows is explicit. Use a recursive CTE when you need to traverse an unknown number of hierarchy levels.

## 16. Pattern: Find the First or Last Record per Group

**Use when:** You need the first transaction, latest order, or earliest event for each customer.

### Latest record per customer

```sql
WITH ranked_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS row_num
    FROM orders
)
SELECT order_id, customer_id, order_date
FROM ranked_orders
WHERE row_num = 1;
```

This selects one latest order per customer, assuming `order_id` provides a suitable tie-breaker.

To find the earliest order, change the ordering to ascending.

**Important:** If you need every record tied for the latest date, use `RANK()` over `order_date DESC` and filter for rank 1 rather than using `ROW_NUMBER()`.

## 17. Pattern: Calculate Percentage Change

**Use when:** You need to compare revenue, price, or volume with the previous period.

```sql
WITH sales_comparison AS (
    SELECT
        sale_date,
        sales,
        LAG(sales) OVER (
            ORDER BY sale_date
        ) AS previous_sales
    FROM daily_sales
)
SELECT
    sale_date,
    sales,
    previous_sales,
    100.0 * (sales - previous_sales)
        / NULLIF(previous_sales, 0) AS percentage_change
FROM sales_comparison;
```

Formula:

\[
\text{Percentage Change} =
\frac{\text{Current Value}-\text{Previous Value}}
{\text{Previous Value}}\times100
\]

`NULLIF(previous_sales, 0)` avoids division by zero by returning `NULL` when the previous value is zero.

## 18. Pattern: Find Customers Who Meet Multiple Conditions

**Use when:** You need customers who purchased multiple categories or completed multiple kinds of activities.

### Example: Customers with both successful and failed transactions

```sql
SELECT customer_id
FROM transactions
GROUP BY customer_id
HAVING SUM(CASE
    WHEN status = 'SUCCESS' THEN 1
    ELSE 0
END) > 0
AND SUM(CASE
    WHEN status = 'FAILED' THEN 1
    ELSE 0
END) > 0;
```

The query groups transactions by customer and checks that each customer has at least one transaction of each status.

Another approach is to use `EXISTS` twice when the conditions need to be evaluated against separate subsets of data.

## 19. How to Identify the Correct Pattern

When you read a SQL problem, ask these questions in order:

1. **Do I need one row per group or the original rows?**  
   Consider `GROUP BY` or a window function.

2. **Am I ranking records?**  
   Consider `ROW_NUMBER()`, `RANK()`, or `DENSE_RANK()`.

3. **Am I comparing the current row with another row?**  
   Consider `LAG()`, `LEAD()`, or a self join.

4. **Am I looking for missing or matching records?**  
   Consider `NOT EXISTS`, `EXISTS`, or an outer join.

5. **Am I calculating a cumulative value?**  
   Consider a windowed `SUM()` or `AVG()`.

6. **Am I processing hierarchical data?**  
   Consider a recursive CTE.

7. **Am I calculating multiple metrics under different conditions?**  
   Consider conditional aggregation with `CASE`.

## 20. Common Interview Mistakes

- Using `DISTINCT` when the requirement is to keep the latest record for each key.
- Using `ROW_NUMBER()` when all tied highest values must be returned.
- Forgetting `PARTITION BY` in a top-N-per-group problem.
- Filtering aggregated results with `WHERE` instead of `HAVING`.
- Assuming `LAG()` compares consecutive calendar dates.
- Ignoring `NULL` values in joins and aggregate calculations.
- Using a non-unique ordering column when a deterministic row order is required.
- Using `NOT IN` without considering whether the subquery can return `NULL`; `NOT EXISTS` is often a safer alternative.
- Returning duplicate rows because a join matches multiple rows on the other side.
- Solving the problem before clarifying whether the requirement concerns rows, groups, distinct values, or time periods.

## 21. Practice Questions

Try solving these without looking at the examples.

1. Find all employees who earn the highest salary.
2. Find the second-highest distinct salary.
3. Find the top three products by sales within each category.
4. Keep the latest record for each customer.
5. Find customers who have never placed an order.
6. Find customers with at least five orders.
7. Calculate successful and failed transaction counts per customer.
8. Calculate a running total of daily sales for each store.
9. Find dates when sales increased compared with the previous recorded date.
10. Find employees who earn more than their department's average.
11. Find duplicate customer IDs.
12. Find users with activity on consecutive dates.
13. Calculate the percentage change in sales.
14. Find employees and their managers using a self join.
15. Explain when you would use a recursive CTE rather than a self join.

## 22. Quick Revision Table

| Problem type | Common SQL pattern |
|---|---|
| Highest or lowest value | `MAX()`, `MIN()`, or ordered ranking |
| Nth-highest distinct value | `DENSE_RANK()` |
| Top N per group | Ranking function with `PARTITION BY` |
| Keep the latest record | `ROW_NUMBER()` with descending timestamp |
| Find unmatched records | `NOT EXISTS` or an outer join |
| Find records with matches | `EXISTS` |
| Filter groups | `GROUP BY` and `HAVING` |
| Multiple conditional metrics | `SUM(CASE WHEN ...)` |
| Running total | `SUM() OVER (...)` |
| Previous-row comparison | `LAG()` |
| Next-row comparison | `LEAD()` |
| Consecutive dates | `LAG()` plus date arithmetic |
| Data quality checks | `IS NULL`, `GROUP BY`, `HAVING` |
| Above-group average | Windowed `AVG()` or grouped subquery |
| Compare related rows | Self join |
| Traverse hierarchies | Recursive CTE |
| Percentage change | `LAG()` plus arithmetic and `NULLIF()` |

### Self-check

- [ ] Can you recognize the correct pattern from the problem statement?
- [ ] Can you explain why the chosen function fits the requirement?
- [ ] Can you handle ties and duplicate records correctly?
- [ ] Can you account for `NULL` values?
- [ ] Can you explain your query's result, not just write the syntax?
- [ ] Can you adapt the query to your target database's date and window-function syntax?

**Final interview tip:** Do not memorize these queries word for word. Learn the problem pattern, understand why each clause is needed, and practice writing the query from scratch. That is what makes SQL problem-solving transferable across product-company interviews.