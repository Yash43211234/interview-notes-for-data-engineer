# Shuffle Hash Join in Spark

## 1. What Is a Shuffle Hash Join?

A Shuffle Hash Join is a join strategy in Apache Spark where both DataFrames are redistributed (shuffled) based on the join key, and Spark builds a hash table from one side of each partition to find matching records.

It is commonly used for equi-joins when the build side is sufficiently small within each partition.

**Key idea:**
- Shuffle both DataFrames using the join key.
- Build a hash table from one side of each partition.
- Probe that hash table using records from the other side.
- Return matching records.

Unlike a Broadcast Hash Join, it does not broadcast one complete DataFrame to all executors.

## 2. How Does Shuffle Hash Join Work?

Suppose we have two DataFrames.

**Customers**

| customer_id | name |
|---|---|
| 1 | Rahul |
| 2 | Amit |
| 3 | Priya |

**Orders**

| order_id | customer_id | amount |
|---|---|---|
| 101 | 1 | 500 |
| 102 | 2 | 800 |
| 103 | 1 | 300 |

Join:

```python
result = customers.join(
    orders,
    on="customer_id",
    how="inner"
)
```

Conceptually, Spark performs these steps:

1. **Shuffle:** Redistribute both DataFrames using `customer_id`.
2. **Partitioning:** Matching join keys are directed to the same target partition.
3. **Build:** Build a hash table using the selected build-side DataFrame within each partition.
4. **Probe:** Read records from the other side and look for matching keys in the hash table.
5. **Output:** Return the matching records.

Each partition performs its local hash join independently.

## 3. Understanding Build Side and Probe Side

A hash join has two important sides.

**Build side**
- Spark creates a hash table using this side.
- Its records are stored in a hash-based structure.
- It should be sufficiently small to fit in memory for each partition.

**Probe side**
- Spark reads records from this side.
- It uses each record's join key to look for matches in the hash table.

Example:

```text
Build side: Customers
Probe side: Orders

Hash table:
customer_id → customer details

Probe:
Order customer_id = 1
        ↓
Find customer_id = 1
        ↓
Return matching customer and order
```

The build side is selected by Spark's planner based on factors such as estimated sizes, join type, and applicable constraints.

## 4. Why Is It Called Shuffle Hash Join?

The name describes two operations:

**Shuffle:** Both sides are redistributed by the join key so matching records reach the same partition.

**Hash Join:** Spark builds a hash table from one side and uses it to find matches from the other side.

Remember:

- Shuffle Hash Join = shuffle + partition-level hash table.
- Broadcast Hash Join = broadcast a small side + hash table on executors.

## 5. Shuffle Hash Join vs Broadcast Hash Join

| Feature | Shuffle Hash Join | Broadcast Hash Join |
|---|---|---|
| Data movement | Shuffles both sides by join key | Broadcasts one side |
| Hash table | Built within each partition | Built from broadcast data on executors |
| Small-side requirement | Build side should fit in partition-level memory | Broadcast side should be small enough for cluster memory |
| Large-side shuffle | Usually required | Usually avoided for the join |
| Network overhead | Can be high due to shuffle | Broadcast transfer required |
| Typical use | Suitable partition-level build side | One sufficiently small DataFrame |

**Interview takeaway:** Both use hash tables, but their data distribution and memory requirements are different.

## 6. Shuffle Hash Join vs Sort Merge Join

| Feature | Shuffle Hash Join | Sort Merge Join |
|---|---|---|
| Shuffle | Generally both sides | Generally both sides |
| Sorting | Not required for the hash join itself | Required |
| Hash table | Yes | No |
| Main operation | Build and probe hash table | Sort and merge matching keys |
| Memory | Hash table memory required | Sorting and execution memory required |
| Common use | When a suitable build side is small enough per partition | Large equi-joins |

A Shuffle Hash Join can avoid sorting, but its hash table can consume significant memory.

A Sort Merge Join may be a better choice when hash tables would be too large.

## 7. When Does Spark Choose Shuffle Hash Join?

Spark's optimizer considers:
- Join type.
- Join condition.
- Estimated size of each side.
- Available statistics.
- Build-side eligibility.
- Join hints and configuration.
- AQE runtime statistics, when enabled.

A relevant configuration is:

```python
spark.conf.get(
    "spark.sql.join.preferSortMergeJoin"
)
```

The standard Spark SQL default is `true`.

When this preference is enabled, Spark generally favors Sort Merge Join over Shuffle Hash Join when both are applicable.

However, this setting does not mean Shuffle Hash Join is impossible. Other planning conditions, hints, and adaptive decisions can influence the selected strategy.

## 8. How to Encourage Shuffle Hash Join

Spark supports a join hint:

```python
result = large_df1.hint(
    "shuffle_hash"
).join(
    large_df2,
    on="customer_id",
    how="inner"
)
```

You can also apply the hint to a particular side:

```python
result = large_df1.join(
    large_df2.hint("shuffle_hash"),
    on="customer_id",
    how="inner"
)
```

The hint influences Spark's strategy selection. It is not an unconditional guarantee that the requested strategy will be used.

Always verify the actual physical plan.

## 9. How to Identify Shuffle Hash Join?

Use:

```python
result.explain("formatted")
```

Look for:

```text
ShuffledHashJoin
```

A simplified physical plan may contain:

```text
ShuffledHashJoin
:- Exchange
:  +- ...
+- Exchange
   +- ...
```

Interpretation:

- `ShuffledHashJoin`: Spark is using a shuffle hash join.
- `Exchange`: Data redistribution, commonly associated with shuffle.
- The child plans show how each input is prepared.

You can also inspect the Spark UI:
- Shuffle Read.
- Shuffle Write.
- Task duration.
- Memory spill.
- Disk spill.
- Uneven partition sizes.

## 10. Performance Problems

### A. Hash Table Does Not Fit in Memory

If the build-side partition is too large, creating its hash table can cause memory pressure.

Possible symptoms:
- Memory pressure.
- Execution spill.
- Garbage collection overhead.
- Slow tasks or task failures.

**Possible solutions:**
- Reduce input data using filters.
- Select only necessary columns.
- Consider Sort Merge Join.
- Consider Broadcast Hash Join if one complete side is safely small enough.

### B. Data Skew

Suppose most records have `customer_id = 1`.

After shuffling by `customer_id`, many records with that key can be concentrated in one partition.

That partition may take much longer than the others.

**Possible solutions:**
- Investigate key distribution.
- Use AQE skew-join optimization where supported.
- Consider salting for suitable hot-key joins.
- Consider a different join strategy.

### C. Excessive Shuffle

Shuffling both sides can cause significant network and disk activity.

**Possible solutions:**
- Filter before joining.
- Select only required columns.
- Avoid unnecessary repartitioning.
- Use a broadcast join when one side is sufficiently small.
- Review whether the join is necessary.

## 11. Important Difference: Shuffle Hash Join Does Not Mean No Sorting Anywhere

Shuffle Hash Join does not require sorting both sides as part of the hash join itself.

However, other operators in the complete query plan may still require sorting.

Do not assume that the entire query has no sort operations simply because the join strategy is Shuffle Hash Join.

## 12. Common Mistakes

1. Confusing Shuffle Hash Join with Broadcast Hash Join.
2. Assuming the entire DataFrame is broadcast in a Shuffle Hash Join.
3. Assuming Shuffle Hash Join avoids shuffling.
4. Assuming it never uses sorting anywhere in the query.
5. Ignoring build-side memory requirements.
6. Assuming a `shuffle_hash` hint always guarantees the strategy.
7. Thinking Shuffle Hash Join is always faster than Sort Merge Join.
8. Ignoring data skew and uneven partition sizes.

## 13. Interview Questions

**Q1. What is a Shuffle Hash Join?**

It is a Spark join strategy where both DataFrames are shuffled by the join key, and a hash table is built from one side within each partition to find matching records.

**Q2. Why is it called Shuffle Hash Join?**

Because it combines data redistribution through shuffle with hash-table-based matching.

**Q3. What is the build side?**

The build side is the input Spark uses to construct the hash table within each partition.

**Q4. What is the probe side?**

The probe side is the input whose records are checked against the hash table to find matching keys.

**Q5. Does Shuffle Hash Join require a shuffle?**

Generally, yes. Both inputs are redistributed by the join key so matching records reach the same partition.

**Q6. What is the difference between Shuffle Hash Join and Broadcast Hash Join?**

Shuffle Hash Join redistributes both inputs by the join key and builds partition-level hash tables. Broadcast Hash Join broadcasts one side to executors and joins it with local partitions of the other side.

**Q7. What is the difference between Shuffle Hash Join and Sort Merge Join?**

Both generally shuffle data. Shuffle Hash Join builds a hash table and probes it, while Sort Merge Join sorts both sides and merges matching keys.

**Q8. Why might Spark avoid Shuffle Hash Join?**

The build-side partitions may be too large for available memory, or the planner may prefer Sort Merge Join based on its configuration and cost considerations.

**Q9. How do you identify Shuffle Hash Join?**

Use `explain("formatted")` and look for `ShuffledHashJoin`.

**Q10. How can you encourage Spark to use Shuffle Hash Join?**

Use the `shuffle_hash` join hint and verify the physical plan.

## 14. Practice Questions

1. Both DataFrames are large, but one side has relatively small partitions. Could Shuffle Hash Join be suitable? Explain.
2. Your physical plan shows `ShuffledHashJoin` and two `Exchange` operators. What is happening?
3. A Shuffle Hash Join causes memory spill. What would you investigate?
4. How does data skew affect Shuffle Hash Join?
5. Why might Spark choose Sort Merge Join instead of Shuffle Hash Join?
6. If one DataFrame is only 5 MB, which join strategy would you consider first?
7. What is the difference between the build side and probe side?

## 15. Quick Revision

- **Shuffle Hash Join:** Shuffles both sides and performs hash-based matching.
- **Shuffle:** Redistributes records based on join keys.
- **Build side:** Input used to construct the hash table.
- **Probe side:** Input used to search the hash table.
- **Hash table:** Stores records in a structure that supports key-based lookup.
- **Sort Merge Join:** Shuffles, sorts, and merges matching keys.
- **Broadcast Hash Join:** Broadcasts a small side and performs hash-based matching.
- **Physical-plan operator:** `ShuffledHashJoin`.
- **Join hint:** `shuffle_hash`.
- **Main risk:** Hash-table memory pressure and data skew.

## 16. Self-Check

- [ ] Explain Shuffle Hash Join in simple language.
- [ ] Explain the build side and probe side.
- [ ] Explain why both inputs are shuffled.
- [ ] Differentiate Shuffle Hash Join and Broadcast Hash Join.
- [ ] Differentiate Shuffle Hash Join and Sort Merge Join.
- [ ] Identify `ShuffledHashJoin` in a physical plan.
- [ ] Explain memory pressure and data skew.
- [ ] Explain how to use the `shuffle_hash` hint.

**Interview goal:** Be able to explain how Spark redistributes data, builds a hash table within partitions, and uses it to match records.