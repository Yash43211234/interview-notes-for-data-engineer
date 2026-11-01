# Whole-Stage Code Generation in Apache Spark

## 1. What Is Whole-Stage Code Generation?

**Whole-Stage Code Generation** is a Spark SQL optimization technique that combines multiple supported physical operators into a single generated Java code pipeline.

Instead of processing each operator through separate interpreted execution paths, Spark generates code that can execute multiple operations together.

**Simple interview definition:**

Whole-stage code generation is a Spark SQL optimization that combines multiple physical operators into generated Java code to reduce execution overhead and improve CPU efficiency.

### Why Is It Needed?

Consider a DataFrame operation:

```python
result = (
    df.filter(col("salary") > 50000)
      .select("employee_id", "salary")
)
```

Spark needs to:
1. Read employee data.
2. Filter rows.
3. Select required columns.

Without code generation, supported operations may involve separate operator-processing overhead.

With whole-stage code generation, Spark can combine supported operations into one generated execution pipeline.

Benefits:
- Reduces interpretation overhead.
- Reduces intermediate object creation in supported paths.
- Improves CPU efficiency.
- Can improve query execution performance.

---

## 2. How Does It Work?

Spark SQL generates Java code for supported physical operators.

Consider this simplified pipeline:

```text
       FileScan
           |
           v
         Filter
           |
           v
         Project
           |
           v
         Result
```

With whole-stage code generation, Spark can combine the scan, filter, and projection operations into a generated code pipeline when those operators support code generation.

### Step-by-Step Process

1. Spark creates a logical plan.
2. Catalyst optimizes the logical plan.
3. Spark generates a physical plan.
4. Spark identifies compatible operators that support whole-stage code generation.
5. Spark generates Java code for the supported pipeline.
6. The generated code runs as part of task execution.

**Important:** Not every physical operator supports whole-stage code generation. Some operations can break the pipeline.

---

## 3. Without vs With Whole-Stage Code Generation

### Without Whole-Stage Code Generation

Conceptually, operators may process data through separate execution paths.

```text
FileScan
   |
   v
Filter Operator
   |
   v
Project Operator
   |
   v
Result
```

Each operator may introduce additional processing overhead.

### With Whole-Stage Code Generation

Spark combines compatible operators into generated code.

```text
       WholeStageCodegen
               |
               v
       Generated Java Code
               |
       ---------------------
       |        |          |
     Scan     Filter     Project
       |        |          |
       ---------------------
               |
               v
             Result
```

The generated code processes the operations together.

**Remember:** This is a conceptual comparison. Whole-stage code generation does not mean Spark combines every operator in the entire query into one pipeline.

---

## 4. Example in PySpark

Suppose we have an employee DataFrame.

```python
from pyspark.sql.functions import col

result = (
    df.filter(col("salary") > 50000)
      .select("employee_id", "salary")
)

result.show()
```

Possible operations:

- Read employee data.
- Filter employees with salary above 50,000.
- Select employee ID and salary.

Spark may combine supported operators into a whole-stage code-generation pipeline.

The exact plan depends on:
- Spark version.
- Data source.
- Physical operators.
- Configuration.
- Query structure.

---

## 5. How to Check Whole-Stage Code Generation

Use the `explain()` method.

```python
result.explain(True)
```

Or:

```python
result.explain("formatted")
```

In the physical plan, look for:

```text
*(1) Filter
```

```text
*(1) Project
```

The `*` marker commonly indicates operators participating in a whole-stage code-generation stage.

Operators with the same stage number may belong to the same code-generation pipeline.

Example:

```text
*(1) Project
+- *(1) Filter
   +- FileScan parquet
```

This suggests that the filter and projection are part of the same code-generation stage.

**Important:** The exact plan formatting can differ between Spark versions and execution environments.

---

## 6. Configuration

Spark provides a configuration property to enable or disable whole-stage code generation.

### Enable

```python
spark.conf.set(
    "spark.sql.codegen.wholeStage",
    "true"
)
```

### Disable

```python
spark.conf.set(
    "spark.sql.codegen.wholeStage",
    "false"
)
```

### Check Current Configuration

```python
spark.conf.get("spark.sql.codegen.wholeStage")
```

In standard Spark SQL configurations, whole-stage code generation is enabled by default.

For reliable comparisons, use separate Spark sessions or applications when changing this setting, and inspect the resulting physical plans.

---

## 7. Whole-Stage Code Generation vs Tungsten

These concepts are closely related.

| Whole-Stage Code Generation | Project Tungsten |
|---|---|
| A specific execution optimization. | A broader Spark performance initiative. |
| Combines supported operators into generated code. | Focuses on CPU and memory efficiency. |
| Reduces interpretation overhead. | Includes efficient memory and execution techniques. |
| Improves execution efficiency. | Improves overall execution efficiency. |

**Remember:** Whole-stage code generation is one of the important techniques associated with Spark SQL's Tungsten execution improvements.

---

## 8. Whole-Stage Code Generation vs Catalyst Optimizer

| Catalyst Optimizer | Whole-Stage Code Generation |
|---|---|
| Optimizes logical query plans. | Generates code for compatible physical operators. |
| Applies rules such as predicate pushdown and column pruning. | Combines supported operators into execution pipelines. |
| Works during query optimization and planning. | Works during physical execution preparation. |
| Decides an optimized execution plan with the physical planner. | Helps execute supported physical operators efficiently. |

### Simple Example

Query:

```python
df.filter(col("salary") > 50000).select("name")
```

- Catalyst may optimize the filter and projection.
- Spark's physical planner selects physical operators.
- Whole-stage code generation may combine compatible operators into generated code.

---

## 9. When Can Whole-Stage Code Generation Be Limited?

Not every operator can participate in a whole-stage code-generation pipeline.

It can be limited by:
- Operators that do not support code generation.
- Certain Python UDF execution paths.
- Some specialized physical operators.
- Query structures that introduce pipeline boundaries.

For example, Python UDF execution generally requires crossing the JVM-Python boundary, so it cannot simply be fused into the same JVM-generated pipeline as ordinary Spark SQL expressions.

**Interview point:** Whole-stage code generation improves supported execution paths; it does not guarantee that every query will use generated code.

---

## 10. Practical Example: Filter and Aggregation

Consider:

```python
from pyspark.sql.functions import col, sum

result = (
    df.filter(col("salary") > 30000)
      .groupBy("department")
      .agg(sum("salary").alias("total_salary"))
)

result.explain("formatted")
```

A possible physical execution structure:

```text
       HashAggregate
             |
             v
          Exchange
             |
             v
       HashAggregate
             |
             v
           Filter
             |
             v
          FileScan
```

Depending on the physical plan, compatible operators may be grouped into whole-stage code-generation pipelines.

The `Exchange` operation usually introduces a shuffle boundary, so the entire query is not necessarily executed as one continuous generated pipeline.

### Why Is This Useful?

For large-scale Data Engineering workloads, operations such as filtering, projections, and aggregations are common.

Code generation can reduce CPU overhead in supported operations and improve execution efficiency.

---

## 11. Common Interview Questions

### Q1. What is whole-stage code generation?

It is a Spark SQL optimization that combines multiple supported physical operators into generated Java code to reduce execution overhead.

### Q2. Why does Spark use code generation?

To reduce interpretation overhead, improve CPU efficiency, and reduce some intermediate object-processing overhead.

### Q3. What is the relationship between Tungsten and whole-stage code generation?

Whole-stage code generation is an important execution optimization associated with Spark SQL's Tungsten improvements.

### Q4. How can you identify whole-stage code generation in a physical plan?

Use:

```python
df.explain(True)
```

Look for operators with markers such as `*(1)` in the physical plan.

### Q5. Does every Spark operator support whole-stage code generation?

No. Only compatible operators that support code generation can participate in the pipeline.

### Q6. Can Python UDFs participate in the same generated JVM pipeline?

Generally, Python UDF execution requires JVM-Python communication and cannot be fused into the same ordinary JVM-generated pipeline.

### Q7. What is the difference between Catalyst and whole-stage code generation?

Catalyst optimizes query plans, while whole-stage code generation generates code for compatible physical operators.

### Q8. Does whole-stage code generation eliminate all intermediate data?

No. It can reduce intermediate processing overhead, but Spark may still need intermediate data for operations such as shuffles and aggregations.

### Q9. How can you disable whole-stage code generation?

Use:

```python
spark.conf.set(
    "spark.sql.codegen.wholeStage",
    "false"
)
```

### Q10. Does whole-stage code generation always improve performance?

Not necessarily. Its impact depends on the query, operators, data, and execution environment.

---

## 12. Common Mistakes

1. **Thinking whole-stage code generation optimizes the logical plan:** It works with compatible physical operators.

2. **Thinking it combines the entire query:** It combines supported operators into individual pipelines, not necessarily the complete query.

3. **Thinking every operator supports code generation:** Some operators introduce execution boundaries.

4. **Confusing it with Catalyst:** Catalyst optimizes query plans; whole-stage code generation generates execution code.

5. **Thinking it removes all shuffles:** Shuffle operations may still be required for joins and aggregations.

6. **Thinking it eliminates garbage collection:** It can reduce some object-related overhead but does not eliminate JVM garbage collection.

---

## 13. Quick Revision

| Concept | Remember |
|---|---|
| Whole-Stage Code Generation | Combines supported physical operators into generated code. |
| Main Language | Generated Java code. |
| Main Goal | Reduce execution overhead and improve CPU efficiency. |
| Related Initiative | Project Tungsten. |
| Catalyst | Optimizes query plans. |
| Physical Plan | Describes how Spark executes operations. |
| `explain(True)` | Inspect the physical plan. |
| `*(1)` | Common marker for a code-generation stage. |
| Configuration | `spark.sql.codegen.wholeStage` |
| Python UDF | Generally introduces a JVM-Python execution boundary. |

### One-Minute Interview Answer

Whole-stage code generation is a Spark SQL optimization that combines multiple supported physical operators into a single generated Java code pipeline. For example, Spark may combine a scan, filter, and projection into one pipeline instead of processing them through separate interpreted execution paths. This reduces execution overhead and improves CPU efficiency. It is associated with Spark's Tungsten execution improvements. We can inspect it using `explain(True)` and look for markers such as `*(1)` in the physical plan. However, not every operator supports whole-stage code generation.

---

## 14. Practice Questions

1. What is whole-stage code generation?
2. Why does Spark generate Java code?
3. How is it related to Project Tungsten?
4. What is the difference between Catalyst and whole-stage code generation?
5. How can you identify code-generation stages in a physical plan?
6. Why can a shuffle introduce a pipeline boundary?
7. Can Python UDFs participate in ordinary JVM whole-stage code generation?
8. How can you disable whole-stage code generation?
9. Does whole-stage code generation eliminate all intermediate data?
10. Does it always improve performance?

## 15. Self-Assessment

- [ ] I can explain whole-stage code generation in simple language.
- [ ] I understand how Spark combines physical operators.
- [ ] I can explain its relationship with Tungsten.
- [ ] I can distinguish Catalyst from code generation.
- [ ] I can identify code-generation markers in a physical plan.
- [ ] I understand why some operations introduce pipeline boundaries.
- [ ] I can explain its limitations.

**Interview readiness:** You should be able to explain how Spark combines scan, filter, and projection operations into generated code and identify the relevant stages in a physical execution plan.