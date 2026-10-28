# EXPLAIN and SQL Query Plans

## 1. What Is a Query Execution Plan?

A **query execution plan** describes how a database intends to retrieve data and execute a SQL query.

The plan can show operations such as:

- Table scans and index scans.
- Index lookups.
- Joins.
- Sorting.
- Aggregation.
- Filtering.
- Data movement and other database-specific operations.

As a Data Engineer, you use execution plans to understand why a query is slow and identify opportunities to optimize it.

## 2. What Is EXPLAIN?

`EXPLAIN` is a SQL command supported by many database systems that displays the execution plan for a query.

Example:

```sql
EXPLAIN
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

The database returns a plan describing how it expects to execute the query.

The exact output depends on the database engine.

**Important:** In many databases, plain `EXPLAIN` shows an estimated plan rather than executing the query and reporting actual runtime measurements.

## 3. EXPLAIN vs. EXPLAIN ANALYZE

| Feature | EXPLAIN | EXPLAIN ANALYZE |
|---|---|---|
| Main purpose | Inspect the planned execution strategy | Inspect the plan and actual execution measurements |
| Executes the query? | Usually does not execute the query itself | Yes, in systems that support this form |
| Estimated costs | Usually available | Usually available |
| Actual row counts and timing | Generally unavailable | Available in supported implementations |
| Main use | Understand the proposed plan | Compare estimates with actual behavior |

Syntax varies by database. PostgreSQL supports both `EXPLAIN` and `EXPLAIN ANALYZE`. SQL Server commonly uses graphical or XML execution plans and actual execution statistics.

**Warning:** `EXPLAIN ANALYZE` executes the statement in PostgreSQL. For statements that modify data, this can cause real changes unless the operation is appropriately controlled. Do not run it on production write queries without understanding the consequences.

## 4. Example Tables

Assume the following table:

`employees`

| employee_id | employee_name | department_id | salary |
|---:|---|---:|---:|
| 101 | Amit | 10 | 90000 |
| 102 | Priya | 10 | 85000 |
| 103 | Rahul | 20 | 70000 |
| 104 | Neha | 20 | 65000 |
| 105 | Ravi | 10 | 75000 |

Consider this query:

```sql
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

The database must locate employees whose `department_id` equals `10`.

Depending on the table size, indexes, statistics, and optimizer decisions, the database may use a table scan or an index-based access path.

## 5. Common Operations in a Query Plan

### A. Table Scan

A table scan reads rows from a table, potentially examining every row to find matches.

```sql
SELECT *
FROM employees
WHERE department_id = 10;
```

If there is no suitable index, a scan may be the chosen plan.

A table scan is not automatically bad. It can be efficient for small tables or queries that need a large proportion of the rows.

### B. Index Scan

An index scan uses an index to locate relevant entries and may then retrieve the corresponding table rows.

For example, an index on `department_id` may help this query:

```sql
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

Whether an index scan is beneficial depends on selectivity, table size, the columns needed, and the database's cost estimates.

### C. Index Seek or Index Lookup

Some database systems distinguish an index seek or lookup from an index scan.

- **Index seek:** Navigates to a specific key or range in an index.
- **Index scan:** Reads a range or a larger portion of an index.

Terminology differs across database engines. An index seek is not guaranteed to be faster than every scan, and a seek can still retrieve many rows.

### D. Filter

A filter operation checks a condition and retains matching rows.

Example:

```sql
WHERE salary > 80000
```

Check how many rows reach the filter and how many remain after it. A large number of discarded rows can indicate that the query reads more data than necessary, although this is not always avoidable.

### E. Sort

A sort operation arranges rows according to the requested ordering.

```sql
SELECT employee_id, salary
FROM employees
ORDER BY salary DESC;
```

Sorting a large result may consume substantial CPU and memory and may spill to disk.

An appropriate index can sometimes help avoid a separate sort.

### F. Aggregate

Aggregation calculates results such as `COUNT`, `SUM`, and `AVG`.

```sql
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id;
```

The plan may use hash aggregation, sort-based aggregation, or another database-specific strategy.

### G. Join

A query joining two tables may use different join algorithms.

```sql
SELECT
    e.employee_name,
    d.department_name
FROM employees e
JOIN departments d
    ON e.department_id = d.department_id;
```

The optimizer may choose a nested-loop join, hash join, merge join, or another available strategy.

The best choice depends on table sizes, indexes, data distribution, join conditions, and estimated row counts.

## 6. Understanding Estimated Cost and Rows

An execution plan may display estimates such as:

- Estimated number of rows.
- Estimated operation cost.
- Estimated number of rows per operation.
- Estimated startup and total cost.

For example, a plan might estimate that a filter returns 100 rows, while the actual execution returns 50,000.

This difference can affect later decisions, such as which join algorithm to use.

**Important:** Cost values are generally optimizer-specific estimates, not actual elapsed time in seconds. Do not compare raw cost numbers directly across different database systems.

## 7. Estimated Rows vs. Actual Rows

Comparing estimated and actual row counts is one of the most useful ways to diagnose a problematic plan.

Suppose a query plan reports:

| Metric | Value |
|---|---:|
| Estimated rows | 100 |
| Actual rows | 50000 |

The optimizer significantly underestimated the number of rows.

Possible reasons include:

- Outdated statistics.
- Skewed data distribution.
- Correlated columns.
- Unusual parameter values.
- Complex predicates that are difficult to estimate.

Potential remedies include updating statistics, reviewing indexes, examining data distribution, and using database-specific features for improved estimates.

Do not assume that statistics are the cause until the plan and data support that conclusion.

## 8. How to Read a Query Plan

Follow this practical process.

### Step 1: Identify the expensive operations

Look for large scans, expensive joins, sorts, aggregations, disk spills, or repeated operations.

Do not assume that the operation with the largest-looking cost is necessarily the real bottleneck; understand the plan's cost model and actual measurements.

### Step 2: Check the row counts

Compare the number of rows entering and leaving each operation.

Ask:

- Is the query processing far more rows than expected?
- Is a join multiplying rows unexpectedly?
- Is a filter removing most rows only after substantial processing?

### Step 3: Check index usage

Determine whether an appropriate index exists and whether the optimizer uses it.

An index may not help when the query reads much of the table, the index is not selective, or retrieving table rows individually would be expensive.

### Step 4: Inspect joins

Check:

- The join algorithm.
- The estimated and actual input sizes.
- The number of output rows.
- Whether the join keys and data types are appropriate.
- Whether the join is creating unintended duplicates.

### Step 5: Inspect sorts and aggregations

Look for expensive ordering, grouping, memory usage, and spills to disk.

Check whether unnecessary columns or rows are being processed.

### Step 6: Make one targeted change

Examples include adding an appropriate index, rewriting a non-sargable filter, reducing unnecessary data, or correcting a join condition.

### Step 7: Compare the plans

Measure the original and modified queries under comparable conditions. Validate that both return the intended result.

## 9. Example: Comparing Plans Before and After an Index

Query:

```sql
EXPLAIN
SELECT employee_id, employee_name
FROM employees
WHERE department_id = 10;
```

If the table is large and there is no useful index, the database might choose a sequential or table scan.

A candidate index could be:

```sql
CREATE INDEX idx_employees_department
ON employees (department_id);
```

Then inspect the plan again.

The optimizer may choose an index-based plan if it estimates that the index is beneficial.

**Important:** The index does not guarantee a faster query. If most employees belong to department 10, scanning the table may still be more efficient.

For a read-heavy workload, a composite or covering index may sometimes be useful, but its design should be based on actual queries and write costs.

## 10. Example: Diagnosing an Expensive Sort

Query:

```sql
SELECT employee_id, salary
FROM employees
ORDER BY salary DESC;
```

An execution plan might include a sort operation.

Possible investigation steps:

1. Check how many rows must be sorted.
2. Determine whether the application really needs all rows.
3. Check whether an index can satisfy the ordering.
4. Check memory usage and disk spills where available.
5. If only a small top-ranked result is needed, evaluate a top-N query.

Example:

```sql
SELECT employee_id, salary
FROM employees
ORDER BY salary DESC, employee_id
LIMIT 10;
```

`LIMIT` syntax varies by database. Also, a row limit does not guarantee that the database can avoid scanning or sorting a large input; inspect the actual plan.

## 11. Query Plans in Data Engineering

Execution plans are useful in both transactional databases and analytical systems.

| Workload | What to inspect |
|---|---|
| Relational database query | Scans, indexes, joins, sorts, estimates, and actual rows |
| Data warehouse query | Data scanned, partition pruning, joins, aggregations, and spills |
| Spark SQL / Databricks | Logical and physical plans, scans, exchanges, shuffles, and join strategies |
| Incremental ETL | Filter pushdown, source reads, merge operations, and unnecessary repeated processing |

In Spark SQL, `EXPLAIN` can display logical and physical plans. Depending on the format, the plan may reveal operations such as `Exchange`, which often indicates data redistribution or a shuffle.

Example in PySpark:

```python
df = spark.sql("""
    SELECT department_id, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department_id
""")

df.explain("formatted")
```

The exact plan depends on the Spark version, table format, configuration, and available statistics.

A shuffle is not always a problem; joins and aggregations often require data movement. The goal is to identify unnecessary or disproportionately expensive work.

## 12. EXPLAIN vs. EXPLAIN ANALYZE in PostgreSQL

Example:

```sql
EXPLAIN
SELECT *
FROM employees
WHERE department_id = 10;
```

This shows the planned operations and estimates.

To collect actual execution measurements:

```sql
EXPLAIN ANALYZE
SELECT *
FROM employees
WHERE department_id = 10;
```

In PostgreSQL, `EXPLAIN ANALYZE` executes the query and reports actual row counts and timing alongside plan information.

For more detailed diagnostics, PostgreSQL also supports options such as `BUFFERS`, which can report buffer activity when used appropriately.

Remember that actual timing can be influenced by caching, concurrency, and system load. A single execution is not always enough to establish that one plan is consistently faster.

## 13. Common Interview Questions

**Q1. What is an execution plan?**

It describes how the database plans to execute a query, including scans, joins, sorting, and aggregation.

**Q2. What is the difference between EXPLAIN and EXPLAIN ANALYZE?**

In PostgreSQL, `EXPLAIN` reports the planned strategy, while `EXPLAIN ANALYZE` executes the query and includes actual measurements.

**Q3. Is a table scan always bad?**

No. A scan can be the best option for a small table or when a large proportion of its rows is required.

**Q4. What does a large difference between estimated and actual rows indicate?**

It may indicate inaccurate cardinality estimates caused by statistics, data distribution, correlated columns, or other estimation difficulties.

**Q5. Does an index always improve query performance?**

No. Indexes add storage and write overhead, and the optimizer may prefer another access path.

**Q6. What does a sort operation indicate?**

The database must order rows at that point in the plan. The sort may be inexpensive or expensive depending on the number of rows, memory, and execution strategy.

**Q7. What is a hash join?**

A join algorithm that commonly builds a hash table from one input and probes it with rows from the other input. It is often useful for equi-joins when the inputs and memory requirements make it appropriate.

**Q8. How do you investigate a slow query?**

Inspect its plan, compare estimated and actual rows, check scans and joins, examine sorts and resource usage, make a targeted change, and measure again.

**Q9. What is the difference between a logical and a physical plan?**

A logical plan describes the query operations at a logical level. A physical plan specifies execution strategies for those operations.

**Q10. What does EXPLAIN show in Spark SQL?**

It can display logical and physical plans, helping identify scans, exchanges, joins, aggregations, and other execution operations.

## 14. Common Mistakes

- Assuming the most expensive-looking operator is always the actual bottleneck.
- Treating optimizer cost units as elapsed seconds.
- Assuming every table scan is inefficient.
- Assuming an index must always be used.
- Ignoring actual row counts when available.
- Assuming a query plan stays identical across data sizes or database configurations.
- Running `EXPLAIN ANALYZE` on production write statements without understanding its effects.
- Optimizing the plan without verifying the query's correctness.
- Assuming a Spark shuffle always indicates a mistake.
- Comparing runtimes without considering caching, concurrency, and workload differences.

## 15. Interview Practice

1. Explain the difference between `EXPLAIN` and `EXPLAIN ANALYZE`.
2. A query has a table scan and runs slowly. How would you investigate it?
3. The optimizer estimates 100 rows but processes 50,000. What could cause this?
4. How can you determine whether an index is useful for a query?
5. What do nested-loop, hash, and merge joins mean?
6. How would you diagnose an expensive sort or aggregation?
7. What is the difference between a logical plan and a physical plan?
8. How would you investigate an expensive `Exchange` operation in Spark?
9. Why should you measure a query before and after optimization?
10. Why is an index not always the best solution for a slow query?

## Quick Revision

- **EXPLAIN:** Inspect the planned query strategy.
- **EXPLAIN ANALYZE:** Execute the query and collect actual measurements in supported systems.
- **Table scan:** Reads table rows; not inherently inefficient.
- **Index scan / seek:** Uses an index when the access path is beneficial.
- **Estimated rows:** The optimizer's prediction.
- **Actual rows:** The rows processed or returned during execution.
- **Join operators:** Show how tables are combined.
- **Sort and aggregate operators:** Reveal ordering and grouping work.
- **Physical plan:** Shows the execution strategies selected by the engine.
- **Spark Exchange:** Often signals data redistribution.
- **Best practice:** Inspect, measure, optimize, and validate.

**Core interview takeaway:** An execution plan helps you identify where a query spends its work. Use estimates and actual measurements where available, optimize the real bottleneck, and verify both performance and correctness.
