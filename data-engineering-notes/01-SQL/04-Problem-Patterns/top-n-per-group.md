# Top N per Group in SQL

## 1. What Is Top N per Group?

Top N per Group means selecting the top N records within each group based on a specified ranking or sorting condition.

Examples:
- Find the top 3 highest-paid employees in each department.
- Find the top 5 products by sales in each category.
- Find the 2 most recent orders for every customer.
- Find the highest-scoring student in each class.

**Key idea:** We need to rank records separately within each group, rather than across the entire table.

## 2. Sample Table

Assume a table named `employees`.

| employee_id | employee_name | department | salary |
|---:|---|---|---:|
| 101 | Amit | IT | 90000 |
| 102 | Priya | IT | 85000 |
| 103 | Rahul | IT | 85000 |
| 104 | Neha | IT | 70000 |
| 105 | Ravi | HR | 75000 |
| 106 | Sneha | HR | 65000 |
| 107 | Karan | HR | 60000 |
| 108 | Pooja | Sales | 80000 |
| 109 | Aman | Sales | 72000 |
| 110 | Riya | Sales | 68000 |

## 3. Find the Top 3 Employees by Salary in Each Department

Use `ROW_NUMBER()` with `PARTITION BY department`.

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
SELECT
    employee_id,
    employee_name,
    department,
    salary
FROM ranked_employees
WHERE rn <= 3
ORDER BY department, salary DESC, employee_id;
```

### How it works

1. `PARTITION BY department` divides employees into separate department groups.
2. `ORDER BY salary DESC` ranks higher salaries first.
3. `ROW_NUMBER()` assigns a unique sequential number within each department.
4. `WHERE rn <= 3` returns the first three employees from each group.

The `employee_id` tie-breaker makes the selection deterministic when employees have the same salary.

**Important:** `ROW_NUMBER()` is calculated after the rows are selected for the window operation. Use a CTE or subquery to filter its result in the outer query.

## 4. Difference Between ROW_NUMBER(), RANK(), and DENSE_RANK()

These functions behave differently when values tie.

Suppose three employees have salaries of 90,000, 85,000, and 85,000.

| Salary | ROW_NUMBER | RANK | DENSE_RANK |
|---:|---:|---:|---:|
| 90000 | 1 | 1 | 1 |
| 85000 | 2 | 2 | 2 |
| 85000 | 3 | 2 | 2 |
| 70000 | 4 | 4 | 3 |

### ROW_NUMBER()

Assigns a unique number to every row, even when salaries tie.

Use it when you need **exactly N rows per group**, provided each group contains at least N rows.

### RANK()

Assigns the same rank to tied values and skips subsequent rank numbers.

Use it when tied records should share a rank and the ranking should contain gaps.

### DENSE_RANK()

Assigns the same rank to tied values without skipping subsequent rank numbers.

Use it when you want the top N distinct ranking levels, including all ties at those levels.

## 5. Find the Top 3 Salary Ranks Including Ties

Use `DENSE_RANK()` when you want everyone whose salary falls within the three highest distinct salary levels in each department.

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
SELECT
    employee_id,
    employee_name,
    department,
    salary,
    salary_rank
FROM ranked_employees
WHERE salary_rank <= 3
ORDER BY department, salary DESC, employee_id;
```

**Difference:** This query can return more than three employees per department because ties are included.

Do not add `employee_id` to the `DENSE_RANK()` ordering if employees with equal salaries must share the same rank.

## 6. Find the Highest-Paid Employee in Each Department

Use `ROW_NUMBER()` when you need exactly one employee per department, including a deterministic choice among tied salaries.

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
SELECT
    employee_id,
    employee_name,
    department,
    salary
FROM ranked_employees
WHERE rn = 1;
```

If all employees tied for the highest salary must be returned, use `RANK()` ordered by salary alone and filter with `salary_rank = 1`.

## 7. Find the Top 2 Most Recent Orders per Customer

Assume an `orders` table with `order_id`, `customer_id`, and `order_date`.

```sql
WITH ranked_orders AS (
    SELECT
        order_id,
        customer_id,
        order_date,
        ROW_NUMBER() OVER (
            PARTITION BY customer_id
            ORDER BY order_date DESC, order_id DESC
        ) AS rn
    FROM orders
)
SELECT
    order_id,
    customer_id,
    order_date
FROM ranked_orders
WHERE rn <= 2
ORDER BY customer_id, order_date DESC, order_id DESC;
```

This returns at most two orders per customer. `order_id` is used as a tie-breaker when order dates are identical.

## 8. Find the Top 3 Products by Sales in Each Category

Assume a table named `product_sales` with `product_id`, `category`, and `sales_amount`.

```sql
WITH ranked_products AS (
    SELECT
        product_id,
        category,
        sales_amount,
        ROW_NUMBER() OVER (
            PARTITION BY category
            ORDER BY sales_amount DESC, product_id
        ) AS rn
    FROM product_sales
)
SELECT
    product_id,
    category,
    sales_amount
FROM ranked_products
WHERE rn <= 3;
```

If the table contains multiple transaction rows per product, aggregate sales by product and category first. Otherwise, the query ranks transaction records rather than total product sales.

## 9. Common Interview Questions

**Q1. Why can't we use `WHERE ROW_NUMBER() ... <= 3` directly?**

Window functions are evaluated after the `WHERE` clause in the logical processing order. Calculate the row number in a CTE or subquery, then filter it in the outer query.

**Q2. What does `PARTITION BY` do?**

It divides rows into independent groups for the window function. Ranking starts again from the beginning for every group.

**Q3. Which function should you use for exactly three employees per department?**

Use `ROW_NUMBER()` with a deterministic tie-breaker, then filter with `rn <= 3`. A department with fewer than three employees returns fewer than three rows.

**Q4. Which function should you use for the top three distinct salary levels?**

Use `DENSE_RANK()` ordered by salary descending, then filter with `salary_rank <= 3`.

**Q5. What if a department has tied salaries at the third position?**

- `ROW_NUMBER()`: Selects rows according to the tie-breaking order, potentially excluding tied employees.
- `RANK()`: Includes ties at rank 3, but a rank gap may mean fewer than three distinct salary levels are represented.
- `DENSE_RANK()`: Includes all employees in the top three distinct salary levels.

## 10. Common Mistakes

- Forgetting `PARTITION BY`, which ranks the entire table rather than each group.
- Using `RANK()` when exactly N rows are required.
- Using `ROW_NUMBER()` when all ties at the cutoff must be included.
- Adding a unique column to the ranking order when tied values are supposed to share a rank.
- Filtering a window function in the same query's `WHERE` clause.
- Ranking transaction records instead of aggregated product or customer totals.
- Forgetting deterministic tie-breaking when selecting a fixed number of records.

## 11. Interview Practice

1. Find the top 3 highest-paid employees in every department.
2. Find all employees tied for the second-highest salary in each department.
3. Find the latest order for every customer.
4. Find the top 5 products by revenue within each category.
5. Find the two most recent transactions per account.
6. Find the top 10% of employees by salary within each department.
7. Find the highest-scoring student in each class, returning all ties.
8. Find each customer's first and third orders.

## Quick Revision

- **Top N rows per group:** `ROW_NUMBER()` + `PARTITION BY` + `WHERE rn <= N`.
- **Ranking with gaps for ties:** `RANK()`.
- **Ranking without gaps for ties:** `DENSE_RANK()`.
- **Independent ranking groups:** `PARTITION BY`.
- **Ranking order:** `ORDER BY`.
- **Filtering window results:** Use a CTE or subquery.

**Core interview pattern:** Rank rows within each group, then filter the rank in an outer query.
