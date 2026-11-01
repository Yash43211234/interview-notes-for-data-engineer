# Sort Merge Join (SMJ) in Apache Spark

## 1. What Is a Sort Merge Join?

Sort Merge Join is a join strategy used by Apache Spark to combine two datasets based on matching join keys.

It generally works in three main steps:

1. **Shuffle:** Redistribute both DataFrames based on the join key.
2. **Sort:** Sort records within each partition by the join key.
3. **Merge:** Compare the sorted records and combine matching keys.

It is commonly used when joining large DataFrames, especially when neither side is small enough for a broadcast join.

**Key idea:** Spark brings matching keys into the same partition, sorts them, and merges the records.

## 2. Example

Suppose we have two DataFrames.

**Customers**

| customer_id | name |
|---|---|
| 3 | Priya |
| 1 | Rahul |
| 2 | Amit |

**Orders**

| order_id | customer_id | amount |
|---|---|---:|
| 103 | 3 | 500 |
| 101 | 1 | 800 |
| 102 | 2 | 300 |

Join:

```python
result = customers.join(
    orders,
    on="customer_id",
    how="inner"
)
```

Spark may choose Sort Merge Join for this operation.

## 3. How Does Sort Merge Join Work?

### Step 1: Shuffle Both DataFrames

Spark redistributes both DataFrames using the join key.

Records with the same join key are directed to the same target partition.

For example:

```text
Customers                Orders

Partition 1              Partition 1
customer_id = 1          customer_id = 1
customer_id = 3          customer_id = 2

Partition 2              Partition 2
customer_id = 2          customer_id = 3
```

This is a simplified illustration of the redistribution process.

### Step 2: Sort Within Each Partition

After the shuffle, Spark sorts the records within each partition using the join key.

Before sorting:

```text
Customers:
3, 1, 2

Orders:
3, 1, 2
```

After sorting:

```text
Customers:
1, 2, 3

Orders:
1, 2, 3
```

Sorting makes it easier to compare records in order.

### Step 3: Merge Matching Records

Spark compares the sorted keys from both sides.

Conceptually:

```text
Customers          Orders
    1  ------------   1
    2  ------------   2
    3  ------------   3
```

When keys match, Spark combines the corresponding records.

The merge process can move forward through the sorted inputs instead of repeatedly searching the entire datasets.

## 4. Visual Execution Flow

```text
       Customers DataFrame       Orders DataFrame
                |                       |
                v                       v
         Shuffle by key          Shuffle by key
                |                       |
                v                       v
         Sort within             Sort within
          partitions              partitions
                |                       |
                +-----------+-----------+
                            |
                            v
                       Sort Merge
                          Join
                            |
                            v
                       Final Result
```

## 5. Why Does Sort Merge Join Require Shuffle?

Suppose customer ID 101 is in partition 1 of the Customers DataFrame but in partition 5 of the Orders DataFrame.

Spark cannot directly perform a local join unless matching records are brought together.

Therefore, Spark generally redistributes both sides using the join key.

This ensures matching keys are available in the same target partition.

**Important:** A shuffle is generally required when compatible partitioning is not already available. Spark can sometimes reuse existing partitioning and avoid an additional shuffle.

## 6. Why Does Sort Merge Join Require Sorting?

After redistribution, Spark sorts records within each partition.

Why?

Because sorted data allows Spark to compare keys sequentially and efficiently identify matching records.

Example:

```text
Left side:   1, 2, 3, 5, 7
Right side:  1, 3, 4, 5, 8
```

Conceptually, Spark compares the current keys:

- 1 matches 1.
- 2 is smaller than 3, so move forward on the left.
- 3 matches 3.
- 5 matches 5.
- Remaining unmatched keys are handled according to the join type.

For an inner join, only matching records are returned.

## 7. How Does Sort Merge Join Handle Different Join Types?

Sort Merge Join can support several join types when the join conditions and other planning requirements are suitable.

| Join type | Result |
|---|---|
| Inner Join | Only matching records |
| Left Outer Join | All left records and matching right records |
| Right Outer Join | All right records and matching left records |
| Full Outer Join | All records from both sides |
| Left Semi Join | Left records with a match on the right |
| Left Anti Join | Left records without a match on the right |

The physical strategy depends on the join condition and supported execution rules.

## 8. Sort Merge Join vs Broadcast Hash Join

| Feature | Sort Merge Join | Broadcast Hash Join |
|---|---|---|
| Best suited for | Large-to-large joins | Large-to-small joins |
| Shuffle | Generally both sides | Usually avoids shuffling the large side |
| Sorting | Required | Not required for the hash join |
| Hash table | No | Yes |
| Network transfer | Shuffle data | Broadcast small side |
| Memory considerations | Sorting and execution memory | Broadcast data and hash-table memory |

**Interview takeaway:** When one side is small enough to broadcast safely, Broadcast Hash Join may be more efficient. For large datasets, Sort Merge Join is often a suitable strategy.

## 9. Sort Merge Join vs Shuffle Hash Join

| Feature | Sort Merge Join | Shuffle Hash Join |
|---|---|---|
| Shuffle | Generally both sides | Generally both sides |
| Sorting | Yes | Not required for the hash join |
| Hash table | No | Yes |
| Main operation | Sort and merge | Build and probe hash table |
| Memory consideration | Sorting and execution memory | Hash-table memory |
| Common use | Large equi-joins | When build-side partitions are sufficiently small |

**Remember:** Both strategies can involve shuffle, but their local matching mechanisms are different.

## 10. How to Identify Sort Merge Join?

Use:

```python
result.explain("formatted")
```

Look for:

```text
SortMergeJoin
```

A simplified physical plan may look like:

```text
SortMergeJoin
:- Sort
:  +- Exchange
:     +- Scan Customers
+- Sort
   +- Exchange
      +- Scan Orders
```

Explanation:

- `SortMergeJoin`: Performs the join.
- `Sort`: Sorts records within partitions.
- `Exchange`: Redistributes data, commonly through shuffle.
- `Scan`: Reads the input data.

You can also use:

```python
result.explain(True)
```

This provides extended plan information.

## 11. Performance Considerations

### A. Shuffle Cost

Both DataFrames may need to be shuffled.

This increases:
- Network I/O.
- Disk I/O.
- Serialization and deserialization overhead.

### B. Sorting Cost

Sorting requires processing time and execution memory.

Large partitions can increase sorting overhead and may cause disk spill.

### C. Data Skew

If one join key occurs much more frequently than others, one partition may become significantly larger.

This can create slow tasks.

Possible solutions:
- Use AQE skew-join optimization where supported.
- Use salting for suitable hot-key joins.
- Broadcast a suitable small DataFrame.
- Investigate key distribution.

### D. Partition Count

An unsuitable partition count can affect performance.

- Too few partitions can create large tasks.
- Too many partitions can create scheduling and small-task overhead.

Choose partitioning based on data size and workload rather than increasing the partition count blindly.

## 12. How AQE Can Optimize Sort Merge Join

Adaptive Query Execution can modify the physical plan using runtime statistics.

For example, Spark may initially choose Sort Merge Join based on estimated data sizes.

After runtime statistics become available, AQE may determine that one side is sufficiently small and convert the join to Broadcast Hash Join.

AQE can also split eligible skewed shuffle partitions for supported joins.

Important configuration:

```python
spark.conf.get(
    "spark.sql.adaptive.enabled"
)
```

AQE is enabled by default in standard Spark SQL configurations from Spark 3.2 onward, but platform settings may differ.

**Important:** AQE does not guarantee that every Sort Merge Join will be converted or that every skew problem will be solved.

## 13. Common Mistakes

1. Assuming every join uses Sort Merge Join.
2. Assuming every Sort Merge Join always needs a new shuffle.
3. Confusing Sort Merge Join with Shuffle Hash Join.
4. Thinking sorting happens across the entire cluster as one global sort.
5. Assuming Sort Merge Join uses a hash table.
6. Ignoring data skew.
7. Assuming AQE always converts Sort Merge Join to Broadcast Hash Join.
8. Assuming Sort Merge Join is always slower than Broadcast Hash Join.

## 14. Interview Questions

**Q1. What is Sort Merge Join in Spark?**

It is a join strategy where Spark generally shuffles both datasets by the join key, sorts records within partitions, and merges matching keys.

**Q2. What are the three main steps of Sort Merge Join?**

Shuffle, sort, and merge.

**Q3. Why does Spark shuffle both DataFrames?**

To ensure records with matching join keys are available in the same target partition.

**Q4. Why is sorting required?**

Sorting allows Spark to compare keys sequentially and identify matches efficiently.

**Q5. Does Sort Merge Join use a hash table?**

No. It uses sorted records and a merge process rather than building a hash table.

**Q6. When is Sort Merge Join suitable?**

It is commonly suitable for large equi-joins where broadcasting one side is not appropriate.

**Q7. What is the difference between Sort Merge Join and Broadcast Hash Join?**

Sort Merge Join generally shuffles and sorts both sides. Broadcast Hash Join broadcasts a small side and uses a hash table to join it with local partitions of the other side.

**Q8. What is the difference between Sort Merge Join and Shuffle Hash Join?**

Both generally shuffle data. Sort Merge Join sorts and merges records, whereas Shuffle Hash Join builds a hash table within each partition and probes it.

**Q9. How can you identify Sort Merge Join in Spark?**

Use `explain("formatted")` and look for the `SortMergeJoin` operator.

**Q10. How does AQE help Sort Merge Join?**

AQE can convert eligible joins to broadcast joins and optimize supported skewed shuffle partitions using runtime statistics.

## 15. Practice Questions

1. Two large DataFrames are joined using `customer_id`. Why might Spark choose Sort Merge Join?
2. Your physical plan contains `SortMergeJoin`, two `Exchange` operators, and two `Sort` operators. Explain the execution.
3. Why does Spark sort records after shuffle?
4. Can Spark perform a Sort Merge Join without a new shuffle? Explain.
5. How does data skew affect Sort Merge Join?
6. When would you prefer Broadcast Hash Join?
7. How can AQE change a Sort Merge Join?
8. Why might sorting cause disk spill?
9. How would you investigate a slow Sort Merge Join in Spark UI?
10. Explain the difference between Sort Merge Join and Shuffle Hash Join.

## 16. Quick Revision

- **Sort Merge Join:** A join strategy based on sorting and merging matching keys.
- **Step 1:** Shuffle both sides by the join key when needed.
- **Step 2:** Sort records within partitions.
- **Step 3:** Merge matching records.
- **Shuffle:** Brings matching keys into the same partition.
- **Sort:** Arranges records by join key.
- **Merge:** Compares sorted keys and produces matching records.
- **Physical-plan operator:** `SortMergeJoin`.
- **Common use:** Large equi-joins.
- **Main performance concerns:** Shuffle, sorting, memory pressure, and data skew.
- **AQE:** Can adapt eligible join strategies using runtime statistics.

## 17. Self-Check

- [ ] Explain Sort Merge Join in simple language.
- [ ] Explain shuffle, sort, and merge.
- [ ] Explain why matching keys must reach the same partition.
- [ ] Explain why sorting is required.
- [ ] Differentiate Sort Merge Join and Broadcast Hash Join.
- [ ] Differentiate Sort Merge Join and Shuffle Hash Join.
- [ ] Identify Sort Merge Join in a physical plan.
- [ ] Explain how data skew affects Sort Merge Join.
- [ ] Explain how AQE can optimize a join.

**Interview goal:** Be able to explain the complete Sort Merge Join execution flow and justify why Spark may choose it for large datasets.