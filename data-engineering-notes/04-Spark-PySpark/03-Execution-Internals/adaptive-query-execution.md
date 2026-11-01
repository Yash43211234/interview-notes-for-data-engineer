

# Adaptive Query Execution (AQE) in Apache Spark

## 1. What is Adaptive Query Execution?

Adaptive Query Execution (AQE) is a Spark SQL optimization feature that improves query performance by adjusting the execution plan at runtime using actual data statistics.

Normally, Spark creates an execution plan before running a query. However, the actual data distribution may be different from the estimates used during planning.

AQE allows Spark to modify parts of the execution plan while the query is running.

**Interview definition:**

Adaptive Query Execution is a Spark SQL optimization feature that uses runtime statistics to re-optimize query execution plans and improve performance.

### Why is AQE needed?

Consider a join between two large DataFrames.

Before execution, Spark may not know the actual size of intermediate data.

It might choose a sort-merge join, even if one side becomes small enough for a broadcast join after filtering.

AQE can examine runtime statistics and change the join strategy when appropriate.

**Main benefit:** Spark can make better optimization decisions using actual runtime information rather than relying only on estimates.

---

## 2. How AQE Works

Without AQE:

1. Spark analyzes the query.
2. Spark creates an initial execution plan.
3. Spark executes the query using that plan.

With AQE:

1. Spark analyzes the query.
2. Spark creates an initial execution plan.
3. Spark executes parts of the query.
4. Spark collects runtime statistics at execution boundaries.
5. Spark evaluates whether the remaining execution plan can be improved.
6. Spark modifies the plan when a suitable optimization is available.
7. Execution continues using the updated plan.

### Simple flow

```text
SQL / DataFrame Query
         |
         v
  Initial Query Plan
         |
         v
    Execute Stages
         |
         v
 Collect Runtime Statistics
         |
         v
 Evaluate Execution Plan
         |
         v
  Re-optimize if Possible
         |
         v
 Continue Query Execution
         |
         v
      Final Result
```

**Important:** AQE does not continuously rewrite every operation. It uses runtime information at appropriate execution boundaries to optimize the remaining query.

---

## 3. Three Major AQE Optimizations

AQE is commonly associated with three major optimizations:

1. Coalescing post-shuffle partitions.
2. Switching join strategies.
3. Optimizing skewed joins.

Let's understand each one.

---

## 4. Coalescing Post-Shuffle Partitions

### Problem: Too Many Small Partitions

During a shuffle operation, Spark redistributes data across partitions.

For example, suppose Spark creates 200 shuffle partitions.

After filtering or aggregation, the actual amount of data may be very small.

Some partitions may contain only a few records.

This creates unnecessary task scheduling and execution overhead.

### How AQE solves it

AQE examines the actual sizes of shuffle partitions and can combine smaller partitions into fewer, larger partitions.

This process is called **post-shuffle partition coalescing**.

### Example

Suppose the initial shuffle creates 200 partitions.

After execution, Spark finds that many partitions contain very little data.

AQE may combine them into fewer partitions.

<text weight="semibold" size="sm">Conceptual illustration</text>

<box border radius="lg" padding={3} gap={3}>
  <box gap={1}>
    **Before AQE**
    <row gap={1} wrap="wrap">
      {#each Array.from({length:12},(_,i)=>i) as i}<box background="rgba(59,130,246,0.16)" radius="xs" flex="1" minWidth="18px" height="25px" justify="center" align="center"><text size="2xs" color="default">{i+1}</text></box>{/each}
    </row>
    <caption>Many small partitions (illustrative)</caption>
  </box>
  <box align="center">
    <icon name="arrow-down" color="secondary" size="lg"/>
    <badge color="success">AQE coalescing</badge>
  </box>
  <box gap={1}>
    **After AQE**
    <row gap={2}>
      {#each Array.from({length:4},(_,i)=>i) as i}<box background="rgba(22,163,74,0.16)" radius="xs" flex="1" height="30px" justify="center" align="center"><text size="xs" color="default">P{i+1}</text></box>{/each}
    </row>
    <caption>Fewer, larger partitions (illustrative)</caption>
  </box>
</box>

The actual number of partitions depends on runtime statistics and configuration.

### Configuration

```python
spark.conf.set(
    "spark.sql.adaptive.coalescePartitions.enabled",
    "true"
)
```

### Related configuration

```python
spark.conf.set(
    "spark.sql.adaptive.advisoryPartitionSizeInBytes",
    "64MB"
)
```

This sets an advisory target size for coalesced shuffle partitions. It is not a strict guarantee that every partition will have exactly that size.

### When is it useful?

- Queries with many small shuffle partitions.
- Aggregations that significantly reduce data.
- Joins that produce small intermediate datasets.
- Workloads with unnecessary small tasks.

---

## 5. Dynamic Join Strategy Switching

AQE can change the join strategy at runtime when actual data statistics make another strategy more efficient.

### Example: Sort-Merge Join vs Broadcast Hash Join

Suppose we have two DataFrames:

**Orders DataFrame**

| order_id | customer_id | amount |
|---:|---:|---:|
| 101 | 1 | 500 |
| 102 | 2 | 700 |
| 103 | 1 | 300 |

**Customers DataFrame**

| customer_id | customer_name |
|---:|---|
| 1 | Rahul |
| 2 | Amit |

We want to join orders with customers.

```python
result_df = orders_df.join(
    customers_df,
    "customer_id",
    "inner"
)
```

### What can happen?

Initially, Spark may choose a sort-merge join because the estimated size of the customer DataFrame is not small enough for broadcasting.

After filtering or executing earlier stages, Spark may discover that the actual customer-side data is small.

AQE can then change the remaining join strategy to a broadcast hash join, when the conditions are suitable.

### Why is broadcast join useful?

A broadcast join sends the smaller DataFrame to the worker nodes so that the larger DataFrame can be joined locally without the usual shuffle required for a sort-merge join.

### Configuration

```python
spark.conf.set(
    "spark.sql.adaptive.autoBroadcastJoinThreshold",
    "10MB"
)
```

This controls the size threshold used by AQE for considering adaptive broadcast joins.

**Important:** This setting is separate from the regular static broadcast threshold:

```python
spark.conf.get(
    "spark.sql.autoBroadcastJoinThreshold"
)
```

AQE does not guarantee that every join will become a broadcast join. The join type, available statistics, query structure, and configuration all matter.

---

## 6. Skew Join Optimization

### What is Data Skew?

Data skew occurs when data is distributed unevenly across partitions.

Example:

Suppose an orders DataFrame contains 10 million records.

| customer_id | Records |
|---|---:|
| 101 | 100 |
| 102 | 200 |
| 103 | 9,500,000 |
| 104 | 200 |
| Others | 499,500 |

Customer 103 has a disproportionately large number of records.

When Spark performs a join or aggregation, the partition containing customer 103 may become much larger than the other partitions.

This creates a performance problem.

### Problems caused by data skew

- Some tasks process significantly more data.
- A few tasks take much longer than others.
- Other executors may remain idle while waiting.
- The overall job takes longer to complete.

### How AQE solves it

AQE can detect skewed shuffle partitions during supported join operations.

It can split a large skewed partition into smaller pieces and replicate the corresponding partition from the other side of the join when needed.

This allows the join work to be distributed across multiple tasks.

<text weight="semibold" size="sm">Conceptual illustration</text>
<box border radius="lg" padding={3} gap={2}>
  **Before skew optimization**
  <row gap={2} align="center">
    <box background="rgba(59,130,246,0.16)" radius="sm" padding={2} flex="1">
      <title size="lg" tabularNums>100</title>
      <caption>Records</caption>
    </box>
    <box background="rgba(59,130,246,0.16)" radius="sm" padding={2} flex="1">
      <title size="lg" tabularNums>200</title>
      <caption>Records</caption>
    </box>
    <box background="rgba(239,68,68,0.15)" radius="sm" padding={2} flex="1">
      <title size="lg" color="danger" tabularNums>9.5M</title>
      <caption>Skewed partition</caption>
    </box>
  </row>
  <divider color="subtle"/>
  <box align="center" gap={1}>
    <icon name="arrow-down" color="secondary" size="lg"/>
    <badge color="success">AQE skew optimization</badge>
  </box>
  **After splitting the skewed partition**
  <row gap={1}>
    {#each [2.4,2.4,2.4,2.3] as v,i}
      <box background="rgba(22,163,74,0.15)" radius="sm" flex="1" padding={2} align="center">
        <title size="md" tabularNums>{v}M</title>
        <caption>Split {i+1}</caption>
      </box>
    {/each}
  </row>
  <caption>Illustrative split only. Actual split sizes depend on the data and AQE settings.</caption>
</box>

### Configuration

```python
spark.conf.set(
    "spark.sql.adaptive.skewJoin.enabled",
    "true"
)
```

Related configuration:

```python
spark.conf.set(
    "spark.sql.adaptive.skewJoin.skewedPartitionFactor",
    "5"
)
```

```python
spark.conf.set(
    "spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes",
    "256MB"
)
```

A partition is considered skewed when it exceeds the applicable skew-size threshold and is sufficiently larger than the median partition size, based on these settings.

### When is it useful?

- Large joins with uneven key distributions.
- Customer or product IDs with highly uneven record counts.
- Fact-to-fact joins with skewed keys.
- Workloads where a few tasks take much longer than the rest.

**Important:** AQE skew optimization does not automatically solve every type of data skew. It is particularly useful for supported shuffle join operations.

---

## 7. Enabling and Checking AQE

In Spark 3.2 and later, AQE is enabled by default in standard Spark SQL configurations, although distributions and platforms may override defaults.

You can explicitly enable it:

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

Check the setting:

```python
spark.conf.get(
    "spark.sql.adaptive.enabled"
)
```

Check other related settings:

```python
spark.conf.get(
    "spark.sql.adaptive.coalescePartitions.enabled"
)

spark.conf.get(
    "spark.sql.adaptive.skewJoin.enabled"
)
```

### Important AQE configurations

| Configuration | Purpose |
|---|---|
| `spark.sql.adaptive.enabled` | Enables AQE |
| `spark.sql.adaptive.coalescePartitions.enabled` | Enables post-shuffle partition coalescing |
| `spark.sql.adaptive.autoBroadcastJoinThreshold` | Threshold for adaptive broadcast join consideration |
| `spark.sql.adaptive.skewJoin.enabled` | Enables skew join optimization |
| `spark.sql.adaptive.advisoryPartitionSizeInBytes` | Advisory target size for shuffle partitions |
| `spark.sql.adaptive.skewJoin.skewedPartitionFactor` | Relative skew detection factor |
| `spark.sql.adaptive.skewJoin.skewedPartitionThresholdInBytes` | Absolute skew detection threshold |

---

## 8. How to Verify AQE in Spark UI

You can inspect the execution plan and Spark UI to understand whether AQE changed the query plan.

### Using explain()

```python
result_df.explain("formatted")
```

This displays the query's physical plan.

Look for indicators such as:

- `AdaptiveSparkPlan`
- `isFinalPlan`
- `AQEShuffleRead`
- `BroadcastHashJoin`
- `SortMergeJoin`

The exact operators depend on the query and Spark version.

### Using Spark UI

1. Open the Spark application in Spark UI.
2. Navigate to the SQL/DataFrame section.
3. Select the query.
4. Inspect the initial and final physical plans, when available.
5. Compare shuffle sizes, number of tasks, and task execution times.

**Interview point:** Do not assume AQE improved a query just because it is enabled. Verify the final plan and execution metrics.

---

## 9. AQE vs Static Query Optimization

| Static Query Optimization | Adaptive Query Execution |
|---|---|
| Uses information available before execution | Uses runtime statistics |
| Creates an initial execution plan | Can modify the remaining plan |
| Decisions are based on estimates and available metadata | Decisions can use actual shuffle statistics |
| Cannot react to later runtime discoveries in the same way | Can adapt to runtime discoveries |

**Remember:** AQE builds on Spark's existing query optimization. It does not replace the Catalyst optimizer.

---

## 10. AQE vs Catalyst Optimizer

These two concepts are related but different.

### Catalyst Optimizer

Catalyst is Spark SQL's query optimization framework.

It performs optimizations such as:

- Predicate pushdown.
- Projection pruning.
- Constant folding.
- Logical plan optimization.
- Physical plan selection.

### Adaptive Query Execution

AQE uses runtime statistics to improve the execution plan while a query is running.

| Catalyst Optimizer | AQE |
|---|---|
| Optimizes query plans | Adapts plans at runtime |
| Uses logical and available planning statistics | Uses runtime execution statistics |
| Produces an initial optimized plan | Can re-optimize the remaining plan |

**Interview answer:** Catalyst optimizes the query plan before execution, while AQE can further improve the physical execution plan using runtime statistics.

---

## 11. Real-World Data Engineering Example

Suppose a data engineering pipeline processes:

- 100 million order records.
- 2 million customer records.
- A customer lookup join.
- Aggregation by region.

A simplified pipeline:

```python
from pyspark.sql.functions import (
    col,
    sum
)

spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)

result_df = (
    orders_df
    .join(
        customers_df,
        "customer_id",
        "inner"
    )
    .groupBy("region")
    .agg(
        sum("amount").alias("total_sales")
    )
)

result_df.show()
```

AQE may help by:

1. Choosing a more suitable join strategy using runtime statistics.
2. Coalescing small shuffle partitions after intermediate operations.
3. Handling skewed shuffle partitions during supported joins.

The actual optimization depends on data sizes, query structure, and configuration.

---

## 12. Common Mistakes

1. **Assuming AQE replaces Catalyst**

   AQE complements the existing Spark SQL optimization process.

2. **Assuming AQE is always faster**

   AQE can improve performance, but the result depends on the query and data.

3. **Confusing AQE with dynamic allocation**

   AQE changes query execution plans. Dynamic allocation adjusts the number of executors.

4. **Assuming AQE automatically fixes all skew**

   It supports skew optimization for certain shuffle joins, not every skew-related workload.

5. **Assuming coalescing always increases performance**

   Excessive coalescing can reduce parallelism if partitions become too large.

6. **Confusing initial and final execution plans**

   With AQE, the final physical plan may differ from the initial plan.

7. **Changing every AQE configuration without measurement**

   Start with defaults, inspect execution metrics, and tune only when there is a demonstrated problem.

---

## 13. Common Interview Questions

**Q1. What is Adaptive Query Execution in Spark?**

AQE is a runtime optimization feature that uses actual execution statistics to modify query plans and improve performance.

**Q2. Why is AQE required?**

Because estimated data sizes may differ from actual runtime data sizes. AQE helps Spark make better decisions using runtime information.

**Q3. What are the three major AQE optimizations?**

1. Post-shuffle partition coalescing.
2. Adaptive join strategy switching.
3. Skew join optimization.

**Q4. What is post-shuffle partition coalescing?**

It combines small shuffle partitions into fewer, larger partitions to reduce unnecessary task overhead.

**Q5. How does AQE optimize joins?**

It can change join strategies, such as switching from a sort-merge join to a broadcast hash join when runtime statistics make broadcasting suitable.

**Q6. What is data skew?**

Data skew occurs when some partitions contain significantly more data than others, causing uneven task execution times.

**Q7. How does AQE handle skewed joins?**

It detects sufficiently large skewed shuffle partitions and can split them into smaller pieces to distribute join work.

**Q8. Is AQE enabled by default?**

In standard Spark SQL configurations from Spark 3.2 onward, AQE is enabled by default. Platform-specific configurations may differ.

**Q9. How do you enable AQE?**

```python
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)
```

**Q10. How do you check whether AQE is working?**

Inspect the final physical execution plan and Spark UI SQL metrics.

**Q11. What is the difference between AQE and Catalyst?**

Catalyst performs query optimization using logical and planning information. AQE uses runtime statistics to adapt the execution plan.

**Q12. Does AQE eliminate the need for repartitioning?**

No. AQE optimizes certain shuffle operations, but explicit repartitioning may still be useful when the workload requires a particular distribution.

---

## 14. Practice Questions

1. Explain why Spark may need to modify an execution plan at runtime.
2. Explain the difference between static and adaptive query optimization.
3. What happens when Spark creates too many small shuffle partitions?
4. How does AQE reduce unnecessary shuffle task overhead?
5. Explain how AQE can change a sort-merge join into a broadcast hash join.
6. What is data skew, and how does it affect Spark performance?
7. Explain how AQE handles skewed joins.
8. How can you verify AQE behavior using Spark UI?
9. What is the difference between AQE and dynamic allocation?
10. Describe a real-world ETL scenario where AQE could improve performance.

---

## 15. Quick Revision

| Topic | Remember |
|---|---|
| AQE | Runtime query optimization |
| Runtime statistics | Actual execution information |
| Coalescing | Combines small shuffle partitions |
| Join switching | Changes join strategy when suitable |
| Skew optimization | Splits large skewed shuffle partitions |
| Catalyst | Initial query optimization framework |
| Spark UI | Helps inspect final execution plans |
| `spark.sql.adaptive.enabled` | Main AQE setting |

### Important code to remember

```python
# Enable AQE
spark.conf.set(
    "spark.sql.adaptive.enabled",
    "true"
)

# Enable partition coalescing
spark.conf.set(
    "spark.sql.adaptive.coalescePartitions.enabled",
    "true"
)

# Enable skew join optimization
spark.conf.set(
    "spark.sql.adaptive.skewJoin.enabled",
    "true"
)

# Check AQE
spark.conf.get(
    "spark.sql.adaptive.enabled"
)

# Inspect execution plan
result_df.explain("formatted")
```

---

## 16. Self-Assessment

Before moving to the next topic, make sure you can:

- [ ] Define Adaptive Query Execution.
- [ ] Explain why AQE is needed.
- [ ] Explain how AQE uses runtime statistics.
- [ ] Understand post-shuffle partition coalescing.
- [ ] Explain adaptive join strategy switching.
- [ ] Understand skew join optimization.
- [ ] Identify important AQE configurations.
- [ ] Verify AQE using Spark UI and execution plans.
- [ ] Explain AQE versus Catalyst Optimizer.
- [ ] Describe a real-world ETL use case.

**Interview readiness:** You should be able to explain the three major AQE optimizations, describe how runtime statistics influence execution decisions, and give a practical example of how AQE can improve a Spark job.

