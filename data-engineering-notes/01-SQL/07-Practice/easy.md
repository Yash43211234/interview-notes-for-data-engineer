# SQL Practice — Easy Level

**File:** `07-practice/easy.md`

## 1. How to Practice

For each problem:

1. Read the question and identify the required output.
2. Identify the table and columns you need.
3. Decide whether you need `WHERE`, `GROUP BY`, `HAVING`, `JOIN`, or a window function.
4. Write the query without looking at the solution.
5. Test your query against the sample data.
6. Explain your approach in simple words, as you would in a Data Engineer interview.

**Rule:** Try each problem for 10–15 minutes before checking the solution.

---

## 2. Sample Tables

Use these tables for the practice questions.

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

---

## 3. Level 1 — Basic SELECT and Filtering

### Problem 1: Display all employees

Display every column from the `employees` table.

**Concepts:** `SELECT`, `FROM`

<details>
<summary>Solution</summary>

```sql
SELECT *
FROM employees;
```

</details>

### Problem 2: Select specific columns

Display each employee's name, department, and salary.

**Concepts:** Column selection

<details>
<summary>Solution</summary>

```sql
SELECT employee_name, department, salary
FROM employees;
```

</details>

### Problem 3: Find high-salary employees

Find employees whose salary is greater than ₹70,000.

**Concepts:** `WHERE`, comparison operators

<details>
<summary>Solution</summary>

```sql
SELECT employee_name, salary
FROM employees
WHERE salary > 70000;
```

</details>

### Problem 4: Filter by department

Find all employees working in the IT department.

**Concepts:** `WHERE`

<details>
<summary>Solution</summary>

```sql
SELECT *
FROM employees
WHERE department = 'IT';
```

</details>

### Problem 5: Use multiple conditions

Find IT employees whose salary is at least ₹70,000.

**Concepts:** `AND`

<details>
<summary>Solution</summary>

```sql
SELECT employee_name, salary
FROM employees
WHERE department = 'IT'
  AND salary >= 70000;
```

</details>

### Problem 6: Use IN

Find employees working in either IT or Finance.

**Concepts:** `IN`

<details>
<summary>Solution</summary>

```sql
SELECT employee_name, department
FROM employees
WHERE department IN ('IT', 'Finance');
```

</details>

### Problem 7: Find a salary range

Find employees earning between ₹60,000 and ₹80,000, inclusive.

**Concepts:** `BETWEEN`

<details>
<summary>Solution</summary>

```sql
SELECT employee_name, salary
FROM employees
WHERE salary BETWEEN 60000 AND 80000;
```

</details>

### Problem 8: Sort employees by salary

Display employees from highest salary to lowest salary.

**Concepts:** `ORDER BY`, descending order

<details>
<summary>Solution</summary>

```sql
SELECT employee_name, salary
FROM employees
ORDER BY salary DESC;
```

</details>

### Problem 9: Find distinct departments

Display each department only once.

**Concepts:** `DISTINCT`

<details>
<summary>Solution</summary>

```sql
SELECT DISTINCT department
FROM employees;
```

</details>

### Problem 10: Find the top three salaries

Display the three employees with the highest salaries.

**Concepts:** `ORDER BY`, `LIMIT`

<details>
<summary>Solution</summary>

```sql
SELECT employee_name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;
```

</details>

**Note:** `LIMIT` is supported by MySQL and PostgreSQL. SQL Server commonly uses `TOP` or `OFFSET/FETCH`.

---

## 4. Level 2 — Aggregation and Grouping

### Problem 11: Count employees

Find the total number of employees.

**Concepts:** `COUNT`

<details>
<summary>Solution</summary>

```sql
SELECT COUNT(*) AS total_employees
FROM employees;
```

</details>

### Problem 12: Calculate average salary

Find the average salary of all employees.

**Concepts:** `AVG`

<details>
<summary>Solution</summary>

```sql
SELECT AVG(salary) AS average_salary
FROM employees;
```

</details>

### Problem 13: Find department-wise employee counts

Display each department and the number of employees in it.

**Concepts:** `GROUP BY`, `COUNT`

<details>
<summary>Solution</summary>

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department;
```

</details>

### Problem 14: Find department-wise average salaries

Calculate the average salary for every department.

**Concepts:** `GROUP BY`, `AVG`

<details>
<summary>Solution</summary>

```sql
SELECT department, AVG(salary) AS average_salary
FROM employees
GROUP BY department;
```

</details>

### Problem 15: Find departments with at least three employees

Display departments that have three or more employees.

**Concepts:** `GROUP BY`, `HAVING`

<details>
<summary>Solution</summary>

```sql
SELECT department, COUNT(*) AS employee_count
FROM employees
GROUP BY department
HAVING COUNT(*) >= 3;
```

</details>

### Problem 16: Find the highest salary per department

Display the highest salary in each department.

**Concepts:** `MAX`, `GROUP BY`

<details>
<summary>Solution</summary>

```sql
SELECT department, MAX(salary) AS highest_salary
FROM employees
GROUP BY department;
```

</details>

### Problem 17: Calculate total order amount

Find the total amount across all orders.

**Concepts:** `SUM`

<details>
<summary>Solution</summary>

```sql
SELECT SUM(amount) AS total_order_amount
FROM orders;
```

</details>

### Problem 18: Count orders by status

Display the number of orders for each status.

**Concepts:** `GROUP BY`, `COUNT`

<details>
<summary>Solution</summary>

```sql
SELECT status, COUNT(*) AS order_count
FROM orders
GROUP BY status;
```

</details>

### Problem 19: Find customers with completed orders worth more than ₹2,000

Find orders that are completed and have an amount greater than ₹2,000.

**Concepts:** `WHERE`, multiple conditions

<details>
<summary>Solution</summary>

```sql
SELECT order_id, customer_id, amount
FROM orders
WHERE status = 'Completed'
  AND amount > 2000;
```

</details>

### Problem 20: Find customers whose total order amount exceeds ₹3,000

Calculate the total amount of orders for each customer and return customers whose total exceeds ₹3,000.

**Concepts:** `GROUP BY`, `SUM`, `HAVING`

<details>
<summary>Solution</summary>

```sql
SELECT customer_id, SUM(amount) AS total_amount
FROM orders
GROUP BY customer_id
HAVING SUM(amount) > 3000;
```

</details>

**Interview distinction:** `WHERE` filters rows before grouping; `HAVING` filters groups after aggregation.

---

## 5. Level 3 — JOINs

### Problem 21: Display orders with customer names

Show each order's ID, amount, and customer name.

**Concepts:** `INNER JOIN`

<details>
<summary>Solution</summary>

```sql
SELECT o.order_id, c.customer_name, o.amount
FROM orders o
INNER JOIN customers c
    ON o.customer_id = c.customer_id;
```

</details>

### Problem 22: Display all customers and their orders

Include customers who have no orders.

**Concepts:** `LEFT JOIN`

<details>
<summary>Solution</summary>

```sql
SELECT c.customer_id,
       c.customer_name,
       o.order_id,
       o.amount
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id;
```

</details>

### Problem 23: Find customers with no orders

Display customers who have never placed an order.

**Concepts:** `LEFT JOIN`, `IS NULL`

<details>
<summary>Solution</summary>

```sql
SELECT c.customer_id, c.customer_name
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
WHERE o.order_id IS NULL;
```

</details>

### Problem 24: Display employees and their managers

Show each employee's name and their manager's name.

**Concepts:** Self-join

<details>
<summary>Solution</summary>

```sql
SELECT e.employee_name AS employee,
       m.employee_name AS manager
FROM employees e
LEFT JOIN employees m
    ON e.manager_id = m.employee_id;
```

</details>

### Problem 25: Find total completed order amount per customer

Display each customer and the total value of their completed orders. Include customers with zero completed orders.

**Concepts:** `LEFT JOIN`, conditional filtering, aggregation

<details>
<summary>Solution</summary>

```sql
SELECT c.customer_id,
       c.customer_name,
       COALESCE(SUM(o.amount), 0) AS completed_amount
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
   AND o.status = 'Completed'
GROUP BY c.customer_id, c.customer_name;
```

</details>

**Important:** Putting `o.status = 'Completed'` in the `WHERE` clause would remove customers without matching completed orders, changing the result of the `LEFT JOIN`.

---

## 6. Level 4 — CASE, Dates, and Subqueries

### Problem 26: Classify employees by salary

Create a salary category:
- `High` for salary at least ₹80,000
- `Medium` for salary at least ₹60,000 but below ₹80,000
- `Low` otherwise

**Concepts:** `CASE`

<details>
<summary>Solution</summary>

```sql
SELECT employee_name,
       salary,
       CASE
           WHEN salary >= 80000 THEN 'High'
           WHEN salary >= 60000 THEN 'Medium'
           ELSE 'Low'
       END AS salary_category
FROM employees;
```

</details>

### Problem 27: Find orders placed in February 2024

Display all orders placed during February 2024.

**Concepts:** Date filtering

<details>
<summary>Solution</summary>

```sql
SELECT *
FROM orders
WHERE order_date >= '2024-02-01'
  AND order_date < '2024-03-01';
```

</details>

Using a half-open date range also works when the column contains timestamps, including times on the final day of February.

### Problem 28: Find employees earning above the company average

Display employees whose salaries are greater than the average salary of all employees.

**Concepts:** Scalar subquery

<details>
<summary>Solution</summary>

```sql
SELECT employee_name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

</details>

### Problem 29: Find the second-highest distinct salary

Return the second-highest unique salary.

**Concepts:** Subquery, `DISTINCT`, `MAX`

<details>
<summary>Solution</summary>

```sql
SELECT MAX(salary) AS second_highest_salary
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

</details>

This returns the second-highest distinct salary even when multiple employees share the highest salary. If there is no second distinct salary, the result is `NULL`.

### Problem 30: Find products above the average price

Display products whose prices are higher than the average product price.

**Concepts:** Subquery, `AVG`

<details>
<summary>Solution</summary>

```sql
SELECT product_name, price
FROM products
WHERE price > (
    SELECT AVG(price)
    FROM products
);
```

</details>

---

## 7. Level 5 — Easy Interview Patterns

### Problem 31: Count completed orders per customer

Display each customer ID and the number of completed orders. Include customers with zero completed orders.

**Concepts:** `LEFT JOIN`, conditional aggregation

<details>
<summary>Solution</summary>

```sql
SELECT c.customer_id,
       c.customer_name,
       COUNT(o.order_id) AS completed_orders
FROM customers c
LEFT JOIN orders o
    ON c.customer_id = o.customer_id
   AND o.status = 'Completed'
GROUP BY c.customer_id, c.customer_name;
```

</details>

Use `COUNT(o.order_id)` rather than `COUNT(*)` because a left join produces a row with NULL order columns for a customer without matching orders.

### Problem 32: Find the total revenue by month

Calculate monthly order amounts for 2024.

**Concepts:** Date functions, aggregation

<details>
<summary>Solution — MySQL</summary>

```sql
SELECT DATE_FORMAT(order_date, '%Y-%m') AS order_month,
       SUM(amount) AS total_amount
FROM orders
WHERE order_date >= '2024-01-01'
  AND order_date < '2025-01-01'
GROUP BY DATE_FORMAT(order_date, '%Y-%m')
ORDER BY order_month;
```

</details>

The date-formatting function varies by database. PostgreSQL, for example, uses `TO_CHAR(order_date, 'YYYY-MM')`.

### Problem 33: Find duplicate employee salaries

Find salary values shared by more than one employee.

**Concepts:** `GROUP BY`, `HAVING`

<details>
<summary>Solution</summary>

```sql
SELECT salary, COUNT(*) AS employee_count
FROM employees
GROUP BY salary
HAVING COUNT(*) > 1;
```

</details>

### Problem 34: Find employees hired after 2022

Display employees hired on or after January 1, 2023.

**Concepts:** Date comparison

<details>
<summary>Solution</summary>

```sql
SELECT employee_name, hire_date
FROM employees
WHERE hire_date >= '2023-01-01';
```

</details>

### Problem 35: Find each department's percentage of total employees

Display the employee count and percentage for each department.

**Concepts:** Aggregation, subquery

<details>
<summary>Solution</summary>

```sql
SELECT department,
       COUNT(*) AS employee_count,
       ROUND(
           100.0 * COUNT(*) /
           (SELECT COUNT(*) FROM employees),
           2
       ) AS employee_percentage
FROM employees
GROUP BY department;
```

</details>

The `100.0` helps avoid integer division in databases that perform integer arithmetic for integer operands.

---

## 8. Practice Without Looking at Solutions

Try these independently before opening the earlier solutions or searching online.

| No. | Task | Main concept |
|---:|---|---|
| 36 | Find the minimum salary in each department. | `MIN`, `GROUP BY` |
| 37 | Find employees whose names start with `A`. | `LIKE` |
| 38 | Find customers who placed at least two orders. | `GROUP BY`, `HAVING` |
| 39 | Find the most expensive product in each category. | Aggregation |
| 40 | Find orders that are not completed. | `WHERE`, `<>` |
| 41 | Display customers and their total number of orders. | `LEFT JOIN`, `COUNT` |
| 42 | Find the average order amount for completed orders. | `AVG`, `WHERE` |
| 43 | Find employees who earn the same salary as another employee. | Self-join or aggregation |
| 44 | Find the highest order amount for each customer. | `MAX`, `GROUP BY` |
| 45 | Display the five most expensive products. | `ORDER BY`, `LIMIT` |

---

## 9. Self-Assessment Checklist

Mark a topic only after you can solve at least one question without looking at the solution.

- [ ] `SELECT`, `WHERE`, `ORDER BY`, `DISTINCT`
- [ ] Comparison operators, `IN`, `BETWEEN`, `LIKE`
- [ ] `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`
- [ ] `GROUP BY` and `HAVING`
- [ ] `INNER JOIN` and `LEFT JOIN`
- [ ] Self-joins
- [ ] `CASE` and `COALESCE`
- [ ] Date filtering
- [ ] Scalar subqueries
- [ ] Duplicate detection
- [ ] Second-highest distinct salary
- [ ] Conditional aggregation

## 10. Interview Practice Rules

For each question, practise explaining:

1. **Approach:** What data do you need?
2. **Logic:** Why did you choose this clause or join?
3. **Edge cases:** What happens if values are NULL, duplicated, or missing?
4. **Complexity:** If asked, how might the query scale with larger tables?
5. **Validation:** How would you check that the output is correct?

For product-company Data Engineer interviews, focus first on writing correct SQL, understanding joins and aggregation, handling duplicates and NULLs, and explaining your logic clearly. Once these easy questions feel comfortable, progress to medium-level problems involving window functions, CTEs, ranking, gaps and consecutive dates, and deduplication.