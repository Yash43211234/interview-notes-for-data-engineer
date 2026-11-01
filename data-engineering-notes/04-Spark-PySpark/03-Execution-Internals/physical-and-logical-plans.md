# Logical and Physical Plans in Apache Spark

## 1. What Are Execution Plans?

When we write SQL queries or PySpark DataFrame operations, Spark does not immediately execute each instruction as written.

Instead, Spark creates an execution plan that describes:
- **What operations** need to be performed.
- **How those operations** will be executed.

There are two important types of plans:

1. Logical Plan
2. Physical Plan

**Simple interview definition:**

A logical plan describes what operations Spark needs to perform, while a physical plan describes how Spark will execute those operations across the cluster.

---

## 2. Logical Plan

A logical plan represents the operations required to produce the result.

It focuses on the query's meaning rather than the exact execution strategy.

For example:

```python
result = (
    df.filter(col("salary") > 50000)
      .select("employee_id", "name")
)
```

The logical plan represents these operations:

```text
Read Employee Data
        |
        v
Filter salary > 50000
        |
        v
Select employee_id, name
        |
        v
      Result
```

It describes:
- Read the employee data.
- Filter employees whose salary is greater than 50,000.
- Select the required columns.

It does not yet specify the complete physical execution strategy.

### Types of Logical Plans

Spark SQL query planning commonly involves these stages:

**1. Unresolved Logical Plan**

Spark creates an initial representation of the query.

At this stage, references such as column names and table names may not yet be resolved.

**2. Analyzed Logical Plan**

Spark's analyzer resolves references using metadata.

It checks:
- Whether columns exist.
- Whether tables exist.
- Whether data types are compatible.
- Whether functions are valid.

**3. Optimized Logical Plan**

Catalyst applies optimization rules to improve the logical plan.

Examples:
- Predicate pushdown.
- Column pruning.
- Constant folding.
- Filter simplification.
- Combining compatible operations.

---

## 3. Physical Plan

A physical plan describes how Spark will execute the operations represented by the optimized logical plan.

It selects physical operators and execution strategies.

For example, a logical join operation could be executed using different physical strategies:

- Broadcast Hash Join.
- Sort Merge Join.
- Shuffled Hash Join.
- Broadcast Nested Loop Join.

The physical planner selects a suitable strategy based on factors such as:
- Data statistics.
- Join conditions.
- Available configuration.
- Supported execution strategies.

### Simple Example

Suppose we have two DataFrames:

```python
employees.join(departments, "department_id")
```

The logical plan says:

```text
Join employees and departments
using department_id
```

The physical plan may choose:

```text
SortMergeJoin
   /       \
Sort       Sort
 |           |
Exchange   Exchange
 |           |
Employees  Departments
```

This is an illustrative execution plan. The actual plan may use a different join strategy.

---

## 4. Logical Plan vs Physical Plan

| Logical Plan | Physical Plan |
|---|---|
| Describes what needs to happen. | Describes how operations will execute. |
| Represents query operations. | Represents physical execution operators. |
| Does not specify the final execution strategy. | Specifies the selected execution strategy. |
| Optimized by Catalyst rules. | Selected by Spark's physical planning process. |
| Example: Join two DataFrames. | Example: Broadcast Hash Join or Sort Merge Join. |

**Easy way to remember:**

Logical Plan = What to do.

Physical Plan = How to do it.

---

## 5. Complete Spark SQL Execution Flow

Understand this flow carefully because it is a common interview topic.

```text
       SQL / PySpark Code
                |
                v
      Unresolved Logical Plan
                |
                v
       Analyzed Logical Plan
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
       Distributed Execution
                |
                v
             Result
```

### Step-by-Step Explanation

**Step 1: Write a query**

```python
result = df.filter(col("salary") > 50000)
```

**Step 2: Create an unresolved logical plan**

Spark creates an initial representation of the filter operation.

**Step 3: Analyze the plan**

Spark resolves the `salary` column and checks whether the expression is valid.

**Step 4: Optimize the logical plan**

Catalyst applies relevant optimization rules.

**Step 5: Generate the physical plan**

Spark selects physical operators to perform the required operations.

**Step 6: Execute the query**

When an action triggers execution, Spark runs the selected plan across the cluster.

For supported operations, Spark may use whole-stage code generation to improve execution efficiency.

**Important:** This is a simplified flow. Spark can perform additional planning and adaptive optimization steps, depending on the query and configuration.

---

## 6. How to Inspect Logical and Physical Plans

Spark provides the `explain()` method.

### Example

```python
from pyspark.sql.functions import col

df = spark.read.parquet("/data/employees")

result = (
    df.filter(col("salary") > 50000)
      .select("employee_id", "name")
)
```

### 1. Display All Main Plan Stages

```python
result.explain(True)
```

This displays:
- Parsed logical plan.
- Analyzed logical plan.
- Optimized logical plan.
- Physical plan.

### 2. Display the Physical Plan

```python
result.explain()
```

By default, this displays the physical plan.

### 3. Display a Formatted Plan

```python
result.explain("formatted")
```

This provides a more structured view of the physical plan.

### 4. Display Extended Plan

```python
result.explain("extended")
```

This displays the extended plan information, including logical and physical plans.

**Interview tip:** Use `explain(True)` when you want to inspect the complete query-planning process.

---

## 7. Important Physical Operators

When examining a Spark physical plan, you may see these operators.

| Operator | Meaning |
|---|---|
| FileScan | Reads data from a file-based source. |
| Filter | Applies a filter condition. |
| Project | Selects or calculates columns. |
| Exchange | Redistributes data, often through a shuffle. |
| Sort | Sorts data. |
| HashAggregate | Performs hash-based aggregation. |
| SortMergeJoin | Joins sorted datasets using matching keys. |
| BroadcastHashJoin | Uses a broadcast hash join strategy. |
| BroadcastExchange | Prepares data for broadcasting. |

### Important: Exchange

`Exchange` is especially important in Spark execution plans.

It often represents a shuffle or data redistribution between partitions.

Example:

```python
df.groupBy("department").count()
```

Spark may need to redistribute rows by department so that rows with the same department can be aggregated together.

This redistribution can involve an `Exchange` operation.

**Interview point:** Shuffle can be expensive because it may involve network transfer, disk I/O, and serialization.

---

## 8. Practical Example: Filter and Aggregation

Suppose we want to calculate the total salary for each department.

```python
from pyspark.sql.functions import col, sum

result = (
    df.filter(col("salary") > 30000)
      .groupBy("department")
      .agg(sum("salary").alias("total_salary"))
)

result.explain(True)
```

### Logical Plan Concept

```text
Read Employee Data
        |
        v
Filter salary > 30000
        |
        v
Group By department
        |
        v
Sum salary
        |
        v
      Result
```

### Possible Physical Plan Concept

```text
       Final HashAggregate
                |
                v
             Exchange
                |
                v
      Partial HashAggregate
                |
                v
              Filter
                |
                v
             FileScan
```

This is a simplified conceptual representation. The actual operators and their order depend on Spark's optimization and physical planning.

### What Happens?

1. Spark reads the employee data.
2. It filters rows based on salary.
3. It performs partial aggregation where possible.
4. It redistributes intermediate data by department.
5. It performs final aggregation.
6. It produces the result.

**Why partial aggregation?**

Spark can calculate intermediate aggregation results locally before shuffling data. This can reduce the amount of data transferred across the cluster.

---

## 9. Logical Plan, Physical Plan, and Catalyst Optimizer

These concepts are related but different.

| Concept | Responsibility |
|---|---|
| Logical Plan | Represents the required query operations. |
| Catalyst Optimizer | Analyzes and optimizes the logical plan. |
| Physical Plan | Represents the selected execution strategy. |
| Spark Execution Engine | Executes the selected plan across the cluster. |

### Example

Query:

```python
df.filter(col("salary") > 50000)
```

- Logical Plan: Apply a salary filter.
- Catalyst: Applies relevant optimization rules.
- Physical Plan: Selects physical operators, such as a filter over a data scan.
- Execution Engine: Executes the plan when an action triggers computation.

---

## 10. Logical and Physical Plans vs AQE

Adaptive Query Execution (AQE) can modify parts of the physical plan during execution using runtime statistics.

Example:

1. Spark initially plans a join.
2. It executes part of the query and collects runtime statistics.
3. It discovers that one side of the join is small enough to broadcast.
4. AQE may change the join strategy to Broadcast Hash Join.

Therefore:

- Logical Plan describes the required operations.
- Physical Plan describes the execution strategy.
- AQE can adapt the physical plan during execution.

---

## 11. Common Interview Questions

### Q1. What is a logical plan in Spark?

A logical plan represents the operations required to produce the query result without specifying the final physical execution strategy.

### Q2. What is a physical plan?

A physical plan describes the actual execution strategy selected by Spark, including physical operators such as scans, filters, exchanges, and joins.

### Q3. What is the difference between logical and physical plans?

A logical plan describes what needs to be done, while a physical plan describes how Spark will execute those operations.

### Q4. What are the stages of a logical plan?

1. Unresolved logical plan.
2. Analyzed logical plan.
3. Optimized logical plan.

### Q5. What is the role of Catalyst?

Catalyst analyzes and optimizes Spark SQL logical plans and helps Spark generate efficient physical execution plans.

### Q6. How can you see the execution plan in PySpark?

Use:

```python
df.explain(True)
```

### Q7. What does Exchange mean in a physical plan?

Exchange generally represents data redistribution between partitions. It often indicates a shuffle operation.

### Q8. Why can shuffle be expensive?

Shuffle may require network communication, disk I/O, serialization, and data redistribution across executors.

### Q9. Can one logical plan have multiple physical execution strategies?

Yes. For example, a logical join can potentially be implemented using Broadcast Hash Join, Sort Merge Join, or another supported join strategy.

### Q10. What is the difference between Catalyst and AQE?

Catalyst optimizes the query plan before execution using optimization rules and available information. AQE adapts parts of the physical plan during execution using runtime statistics.

### Q11. Does `explain()` execute the complete query?

No. It displays the execution plan. It does not normally execute the complete query to produce its result.

### Q12. What is whole-stage code generation?

It is a Spark SQL optimization that combines supported operators into generated code to reduce execution overhead.

---

## 12. Common Mistakes

1. **Confusing logical and physical plans:** Logical describes what; physical describes how.

2. **Thinking the logical plan executes the query:** It represents the operations, not the actual distributed execution.

3. **Thinking every Exchange is necessarily bad:** Exchange often indicates a shuffle, but some data redistribution is necessary for operations such as joins and aggregations.

4. **Assuming the physical plan never changes:** AQE can adapt parts of the plan during execution.

5. **Thinking `explain(True)` executes the query:** It displays plan information.

6. **Confusing Catalyst with the execution engine:** Catalyst optimizes and plans queries; Spark's execution engine performs the work.

---

## 13. Quick Revision

| Term | One-line explanation |
|---|---|
| Logical Plan | What operations need to happen. |
| Unresolved Plan | Initial plan with unresolved references. |
| Analyzed Plan | References and data types are resolved. |
| Optimized Plan | Catalyst applies optimization rules. |
| Physical Plan | How Spark will execute the query. |
| Exchange | Data redistribution, often involving shuffle. |
| FileScan | Reads data from a file source. |
| Project | Selects or calculates columns. |
| Filter | Applies conditions. |
| HashAggregate | Performs hash-based aggregation. |
| `explain(True)` | Displays the main logical and physical plan stages. |
| AQE | Adapts the physical plan using runtime statistics. |

### One-Minute Interview Answer

In Spark, when we write SQL or PySpark DataFrame operations, Spark first creates a logical plan that describes what operations are required. The analyzer resolves columns, tables, and data types. Then Catalyst optimizes the logical plan using techniques such as predicate pushdown and column pruning. After that, Spark generates a physical plan that describes how operations will execute using operators such as FileScan, Filter, Exchange, and different join strategies. When an action triggers execution, Spark runs the plan across the cluster. We can inspect these plans using `explain(True)`.

---

## 14. Practice Questions

Try answering without looking at the notes.

1. Explain the complete query-planning flow in Spark.
2. What is the difference between an unresolved and analyzed logical plan?
3. What does Catalyst do to a logical plan?
4. How is a physical plan generated?
5. Can one logical join have different physical implementations?
6. What does Exchange mean?
7. Why does Spark perform partial aggregation?
8. How do you inspect the optimized logical plan?
9. What is the relationship between AQE and the physical plan?
10. What is whole-stage code generation?

## 15. Self-Assessment

- [ ] I can explain logical and physical plans.
- [ ] I understand the three main logical-plan stages.
- [ ] I can explain Catalyst's role.
- [ ] I can recognize common physical operators.
- [ ] I understand Exchange and shuffle.
- [ ] I can use `explain(True)` confidently.
- [ ] I can explain how AQE changes the physical plan.

**Interview readiness:** You should be able to explain the complete Spark query-planning flow and interpret a basic physical plan without looking at the notes.