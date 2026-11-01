# Catalyst Optimizer in Apache Spark

## 1. What Is the Catalyst Optimizer?

The **Catalyst Optimizer** is Spark SQL's query optimization framework.

It analyzes a query, creates an optimized logical execution plan, and helps Spark execute the query efficiently.

It works with:
- Spark SQL queries
- PySpark DataFrame operations
- Dataset operations

**Simple definition for interviews:**

The Catalyst Optimizer is Spark SQL's optimization engine. It transforms a logical query plan into a more efficient logical plan before Spark generates the physical execution plan.

For example:

```python
df.filter(col("age") > 25).select("name")
```

Spark does not immediately execute every operation exactly as written. Catalyst analyzes the operations and applies optimization rules before execution.

---

## 2. Where Does Catalyst Fit in Spark Execution?

A simplified Spark SQL execution flow:

```text
SQL / PySpark DataFrame Code
             |
             v
     Unresolved Logical Plan
             |
             v
       Analyzed Plan
             |
             v
   Optimized Logical Plan
             |
             v
      Physical Plan
             |
             v
    Code Generation
             |
             v
         Execution
```

### Stage 1: Unresolved Logical Plan

Spark creates an initial logical representation of the query.

At this stage, Spark may not yet know:
- Whether referenced columns exist.
- Which table a column belongs to.
- Whether functions and data types are valid.

### Stage 2: Analyzed Logical Plan

Spark's analyzer resolves the query using available metadata.

It checks:
- Column names.
- Table names.
- Data types.
- Function references.

For example, if a query references a column that does not exist, analysis can fail before execution.

### Stage 3: Optimized Logical Plan

Catalyst applies optimization rules to improve the query.

Examples:
- Predicate pushdown.
- Column pruning.
- Constant folding.
- Combining filters.
- Simplifying expressions.

### Stage 4: Physical Plan

Spark selects physical operators to execute the optimized query.

For example:
- Broadcast hash join.
- Sort merge join.
- Hash aggregation.
- Scan operations.

### Stage 5: Code Generation and Execution

Spark may generate optimized JVM bytecode for supported operations using whole-stage code generation.

The selected physical plan is then executed across the cluster.

**Important:** Catalyst handles query planning and optimization. Actual distributed execution is performed by Spark's execution engine.

---

## 3. Important Catalyst Optimization Techniques

### 3.1 Predicate Pushdown

Predicate pushdown means applying filters as close as possible to the data source.

Example:

```python
df.filter(col("country") == "India") \
  .select("name", "salary")
```

Instead of reading all rows and filtering them later, Spark may push the filter toward the data source.

Benefits:
- Reads fewer rows when supported by the source.
- Reduces data processing.
- Can reduce I/O and execution time.

**Interview point:** Predicate pushdown depends on the data source and the operations involved. Not every filter can be pushed down.

### 3.2 Column Pruning

Column pruning means reading or processing only the required columns.

Example:

```python
df.select("employee_id", "salary")
```

Suppose the source contains:

```text
employee_id
name
department
salary
address
joining_date
```

If the query only needs `employee_id` and `salary`, Spark may avoid reading unnecessary columns when the source supports column-level access.

Benefits:
- Reduces data scanned.
- Reduces memory usage.
- Improves query performance.

### 3.3 Constant Folding

Constant folding means evaluating expressions containing constants during query optimization.

Example:

```sql
SELECT *
FROM employees
WHERE salary > 1000 + 500;
```

Spark can simplify:

```sql
salary > 1000 + 500
```

to:

```sql
salary > 1500
```

This avoids repeatedly calculating the same constant expression during execution.

### 3.4 Predicate Simplification

Catalyst can simplify filter conditions.

Example:

```sql
WHERE age > 25 AND age > 30
```

Can be simplified to:

```sql
WHERE age > 30
```

The simplified condition is logically equivalent and easier to evaluate.

### 3.5 Combining Filters

Consider:

```python
df.filter(col("age") > 25) \
  .filter(col("salary") > 50000)
```

Catalyst can combine these filters into one condition:

```python
df.filter(
    (col("age") > 25) &
    (col("salary") > 50000)
)
```

This is an example of logical plan optimization.

### 3.6 Join Optimization

Catalyst can select or support different physical join strategies based on query information, statistics, and configuration.

Common join strategies:

| Join strategy | Basic idea |
|---|---|
| Broadcast Hash Join | Broadcast a small table to worker nodes |
| Sort Merge Join | Sort both sides and merge matching keys |
| Shuffled Hash Join | Shuffle data by key and build hash tables |
| Broadcast Nested Loop Join | Compare records using a nested-loop approach |

The chosen strategy depends on factors such as table sizes, join conditions, statistics, and configuration.

**Important:** Catalyst's logical optimizer and Spark's physical planning strategies work together to select an execution approach.

---

## 4. Catalyst Optimizer vs AQE

Both improve Spark query performance, but they work at different stages.

| Catalyst Optimizer | Adaptive Query Execution (AQE) |
|---|---|
| Optimizes the query plan before execution. | Adapts the physical plan during execution. |
| Uses logical rules and available statistics. | Uses runtime statistics collected during execution. |
| Can simplify filters and prune columns. | Can coalesce shuffle partitions. |
| Helps select physical execution strategies. | Can change certain strategies based on runtime information. |

### Simple example

**Catalyst Optimizer:**

Optimizes a query before execution by pushing down filters and removing unnecessary columns.

**AQE:**

After observing actual runtime data, AQE may change a sort merge join into a broadcast hash join when the runtime conditions allow it.

**Remember:**

Catalyst = Query optimization and planning.

AQE = Runtime adaptive optimization.

---

## 5. How to Check the Optimized Plan

In PySpark, use `explain()`.

Example:

```python
from pyspark.sql.functions import col

df = spark.read.parquet("/data/employees")

result = (
    df.filter(col("salary") > 50000)
      .select("employee_id", "salary")
)

result.explain(True)
```

`explain(True)` displays multiple plan stages, including:
- Parsed logical plan.
- Analyzed logical plan.
- Optimized logical plan.
- Physical plan.

For a more detailed formatted view:

```python
result.explain("formatted")
```

Look for:
- `Filter`
- `Project`
- `FileScan`
- `PushedFilters`
- `ReadSchema`
- Join operators

The exact output depends on the Spark version, source format, and query.

---

## 6. Practical Example: Employee Data

Suppose an employee dataset contains:

| employee_id | name | department | salary |
|---|---|---|---:|
| 101 | Amit | IT | 60000 |
| 102 | Rahul | HR | 45000 |
| 103 | Priya | IT | 75000 |
| 104 | Neha | Finance | 55000 |

Query:

```python
from pyspark.sql.functions import col

result = (
    df.filter(
        (col("department") == "IT") &
        (col("salary") > 50000)
    )
    .select("employee_id", "name", "salary")
)

result.show()
```

Output:

```text
+-----------+-----+------+
|employee_id| name|salary|
+-----------+-----+------+
|        101| Amit| 60000|
|        103|Priya| 75000|
+-----------+-----+------+
```

Potential optimizations:

1. **Predicate pushdown:** Push eligible filters toward the data source.
2. **Column pruning:** Read only the required columns where supported.
3. **Filter optimization:** Simplify and combine filter conditions.
4. **Physical planning:** Choose suitable operators for execution.

The exact optimizations depend on the source and execution plan.

---

## 7. Important Interview Questions

### Q1. What is the Catalyst Optimizer?

Catalyst is Spark SQL's query optimization framework. It analyzes SQL and DataFrame queries, optimizes logical plans, and helps generate efficient physical execution plans.

### Q2. What are the main stages of query planning?

1. Unresolved logical plan.
2. Analyzed logical plan.
3. Optimized logical plan.
4. Physical plan.
5. Code generation and execution, where applicable.

### Q3. What is predicate pushdown?

Predicate pushdown moves eligible filter operations closer to the data source so that unnecessary rows can be avoided during reading.

### Q4. What is column pruning?

Column pruning removes unnecessary columns from the query plan so Spark processes only the required columns.

### Q5. What is constant folding?

Constant folding evaluates constant expressions during optimization.

Example:

```sql
100 + 200
```

becomes:

```sql
300
```

### Q6. Does Catalyst work with PySpark DataFrames?

Yes. PySpark DataFrame operations are translated into Spark SQL logical plans, which Catalyst can analyze and optimize.

### Q7. What is the difference between a logical plan and a physical plan?

- Logical plan describes what operations need to happen.
- Physical plan describes how Spark will execute those operations.

### Q8. What is the difference between Catalyst and AQE?

Catalyst optimizes query plans using optimization rules and available information. AQE uses runtime statistics to adapt parts of the physical plan during execution.

### Q9. How can you inspect the optimized logical plan?

Use:

```python
df.explain(True)
```

Look for the optimized logical plan in the output.

### Q10. Does Catalyst guarantee that every query will run faster?

No. Catalyst applies optimization rules, but the actual performance depends on data size, data layout, partitioning, statistics, cluster resources, and the physical execution plan.

---

## 8. Common Mistakes

1. **Thinking Catalyst executes the query:** Catalyst optimizes and plans queries; Spark's execution engine performs the work.

2. **Confusing logical and physical plans:** A logical plan describes the required operations; a physical plan describes their execution.

3. **Assuming all filters are pushed to the source:** Pushdown depends on source capabilities and expression support.

4. **Assuming Catalyst and AQE are the same:** Catalyst performs query optimization; AQE adapts execution using runtime statistics.

5. **Assuming DataFrame code executes immediately:** Most DataFrame transformations are lazy. Execution is triggered by an action.

6. **Assuming `explain()` runs the entire query:** `explain()` displays the plan; it does not normally execute the complete query to produce its result.

---

## 9. Quick Revision

| Concept | Remember |
|---|---|
| Catalyst Optimizer | Spark SQL query optimization framework |
| Unresolved Plan | Initial plan before references are resolved |
| Analyzed Plan | Columns, tables, and functions are resolved |
| Optimized Logical Plan | Optimization rules are applied |
| Physical Plan | Execution strategy is selected |
| Predicate Pushdown | Move eligible filters closer to the source |
| Column Pruning | Avoid unnecessary columns |
| Constant Folding | Evaluate constant expressions early |
| Join Optimization | Choose suitable physical join strategies |
| `explain(True)` | Inspect logical and physical plans |
| AQE | Adapt execution using runtime statistics |

### One-Minute Interview Answer

The Catalyst Optimizer is the query optimization framework used by Spark SQL. When we write SQL queries or PySpark DataFrame operations, Spark creates a logical plan. The analyzer resolves columns, tables, and data types. Catalyst then applies optimization techniques such as predicate pushdown, column pruning, constant folding, and filter simplification. After that, Spark generates a physical execution plan and executes it across the cluster. We can inspect the plans using `explain(True)`. AQE complements Catalyst by adapting the physical plan using runtime statistics.

---

## 10. Practice Questions

Try answering these without looking at the notes.

1. Explain the complete Catalyst optimization flow.
2. What is the difference between predicate pushdown and column pruning?
3. How does Catalyst optimize multiple filter conditions?
4. What is constant folding? Give an example.
5. Why does Spark need both logical and physical plans?
6. How does Catalyst help optimize joins?
7. How do you inspect an optimized logical plan?
8. Can every filter be pushed down to a Parquet or CSV source?
9. Explain Catalyst Optimizer vs AQE.
10. Does calling `explain(True)` execute the complete query?

## 11. Self-Assessment

- [ ] I can explain Catalyst in simple language.
- [ ] I understand the query planning stages.
- [ ] I can explain predicate pushdown and column pruning.
- [ ] I understand constant folding and filter simplification.
- [ ] I can distinguish logical and physical plans.
- [ ] I can inspect a plan using `explain(True)`.
- [ ] I can explain Catalyst vs AQE without confusion.

**Interview readiness:** You should be able to explain the Catalyst execution flow and at least four optimization techniques without referring to the notes.