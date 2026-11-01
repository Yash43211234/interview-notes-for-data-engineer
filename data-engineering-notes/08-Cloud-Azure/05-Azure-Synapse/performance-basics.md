# Azure Synapse Analytics – Performance Basics

## 1. What Is Performance Optimization in Azure Synapse?

Performance optimization means improving query execution time, data loading speed, resource utilization, and overall workload efficiency.

In Azure Synapse Analytics, performance depends on several factors:

- Data distribution across compute nodes.
- Table design and storage format.
- Query structure and execution plan.
- Data movement between distributions.
- Compute resources and workload concurrency.
- Data volume and partitioning strategy.

The goal is not simply to make one query faster. It is to build a system that performs reliably as data volume and user activity increase.

---

## 2. Understand Synapse SQL Pool Architecture

Azure Synapse provides two important SQL execution options:

1. **Dedicated SQL pool:** Uses provisioned compute resources and distributed processing.
2. **Serverless SQL pool:** Queries data directly from files in the data lake without provisioning a dedicated SQL pool.

Their optimization approaches are different.

| Area | Dedicated SQL Pool | Serverless SQL Pool |
|---|---|---|
| Data storage | Managed relational tables | External files in storage |
| Compute | Provisioned SQL compute | On-demand query processing |
| Distribution | Hash, round-robin, replicated | No dedicated table distribution |
| Main optimization | Distribution, columnstore, statistics | File format, file size, partition elimination |
| Cost consideration | Compute is provisioned | Generally based on data processed |

Do not apply dedicated SQL pool distribution concepts to serverless SQL pool queries.

---

## 3. Dedicated SQL Pool Architecture

A dedicated SQL pool uses a Massively Parallel Processing (MPP) architecture.

### Main components

**Control node**
- Receives SQL queries.
- Parses and optimizes queries.
- Creates an execution plan.
- Coordinates distributed execution.

**Compute nodes**
- Execute query operations in parallel.
- Process data stored in their assigned distributions.
- Perform joins, aggregations, and other operations.

**Data distributions**
- A dedicated SQL pool distributes table data across 60 distributions.
- Compute resources process these distributions in parallel.

### Why MPP matters

Suppose a table contains 600 million rows.

Instead of processing the entire table on one machine, Synapse distributes data across multiple distributions so that work can happen in parallel.

However, parallel processing is effective only when data is distributed appropriately and unnecessary data movement is minimized.

---

## 4. Data Distribution Strategies

Data distribution is one of the most important performance decisions in a dedicated SQL pool.

Synapse supports three distribution methods.

### 4.1 Hash Distribution

Rows are assigned to distributions based on a hash of the selected distribution column.

```sql
CREATE TABLE dbo.Sales
(
    SaleID BIGINT,
    CustomerID INT,
    ProductID INT,
    Amount DECIMAL(12, 2),
    SaleDate DATE
)
WITH
(
    DISTRIBUTION = HASH(CustomerID),
    CLUSTERED COLUMNSTORE INDEX
);
```

**Advantages**
- Supports large tables.
- Can improve joins when related tables use compatible distribution keys.
- Helps reduce data movement for suitable joins and aggregations.

**Disadvantages**
- Poor key selection can cause data skew.
- A low-cardinality column may distribute data unevenly.
- Changing the distribution strategy may require recreating or redistributing the table.

### 4.2 Round-Robin Distribution

Rows are distributed across distributions in a round-robin manner.

```sql
CREATE TABLE dbo.StagingSales
(
    SaleID BIGINT,
    CustomerID INT,
    Amount DECIMAL(12, 2)
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN,
    HEAP
);
```

**Advantages**
- Fast and simple data loading.
- Does not require choosing a distribution key.
- Useful for staging tables.

**Disadvantages**
- Related rows are not intentionally colocated.
- Joins may require data movement.

### 4.3 Replicated Distribution

A copy of the table is stored on each compute node.

```sql
CREATE TABLE dbo.DimRegion
(
    RegionID INT,
    RegionName VARCHAR(100)
)
WITH
(
    DISTRIBUTION = REPLICATE,
    HEAP
);
```

**Advantages**
- Small dimension tables can be available locally on each compute node.
- Can reduce data movement during joins.

**Disadvantages**
- Replication adds storage and maintenance overhead.
- Not suitable for large tables.

### Distribution comparison

| Distribution | Best use | Main concern |
|---|---|---|
| Hash | Large fact tables | Data skew |
| Round-robin | Staging and temporary loading tables | Data movement during joins |
| Replicated | Small dimension tables | Replication overhead |

**Interview point:** A common design is to use hash distribution for large fact tables and replicated distribution for small dimensions, but the correct choice depends on actual data size and query patterns.

---

## 5. Data Skew

Data skew occurs when data is distributed unevenly across distributions.

For example, suppose a table is distributed by `CustomerID`.

If one customer has 40% of all records, a large portion of the data may be assigned to one distribution.

This can cause:
- One distribution to process significantly more data.
- Other distributions to finish earlier and remain idle.
- Longer query execution times.
- Uneven memory and resource consumption.

### How to reduce data skew

1. Choose a high-cardinality distribution column.
2. Avoid columns with a small number of distinct values.
3. Check the frequency of values before selecting a key.
4. Consider a different distribution column if skew is severe.
5. Use round-robin for staging when a suitable hash key is unavailable.

**Important:** High cardinality alone does not guarantee a good distribution key. The values should also have a reasonably balanced frequency.

---

## 6. Columnstore Indexes

Dedicated SQL pool supports columnstore storage, which is useful for analytical workloads.

### Row storage vs column storage

**Row storage**
- Stores values belonging to a row together.
- Useful for retrieving or modifying individual rows.
- Often suitable for transactional workloads.

**Column storage**
- Stores values from the same column together.
- Reads only the required columns.
- Supports efficient compression and analytical scans.

### Clustered Columnstore Index

A clustered columnstore index is commonly used for large analytical fact tables.

```sql
CREATE TABLE dbo.FactSales
(
    SaleID BIGINT,
    CustomerID INT,
    ProductID INT,
    SaleDate DATE,
    Amount DECIMAL(12, 2)
)
WITH
(
    DISTRIBUTION = HASH(CustomerID),
    CLUSTERED COLUMNSTORE INDEX
);
```

### Benefits

- High compression.
- Efficient scanning of selected columns.
- Good performance for large analytical queries.
- Supports parallel processing.

### When to consider a heap

A heap may be useful for staging tables where data is loaded before being transformed or moved into analytical tables.

The final choice should depend on loading patterns, table size, and query workload.

---

## 7. Statistics

Statistics help the SQL query optimizer estimate how many rows will be returned by a query.

The optimizer uses these estimates to select an execution plan.

For example, statistics can help determine:
- Which table should be processed first.
- Whether a join strategy is appropriate.
- How much data may be involved in an operation.
- Whether a particular access path is efficient.

### Why statistics matter

Suppose a query joins a large sales table with a customer table.

If the optimizer has inaccurate information about the number of rows in either table, it may choose a poor execution plan.

### Update statistics

```sql
UPDATE STATISTICS dbo.FactSales;
```

You can also update statistics for a particular column:

```sql
UPDATE STATISTICS dbo.FactSales (CustomerID);
```

For relevant workloads, automatic statistics creation and updates can help, but they should not replace monitoring and deliberate statistics maintenance.

**Interview point:** Statistics do not directly make a query faster. They help the optimizer choose a better execution plan.

---

## 8. Data Movement and Shuffle Operations

Data movement is a major performance consideration in dedicated SQL pools.

It happens when data must move between distributions to complete an operation.

For example, two large tables may be distributed using different keys. A join between them may require rows to be redistributed before matching records.

### Common data movement operations

**ShuffleMoveOperation**
- Redistributes rows across distributions.
- Commonly occurs when data needs to be colocated for a join or aggregation.

**BroadcastMoveOperation**
- Copies a set of rows to other distributions.
- Can be useful when one side of a join is relatively small.

**Move operations**
- Are not always bad.
- Their cost depends on the amount of data moved and the operation involved.

### How to reduce data movement

1. Use compatible hash distribution keys for frequently joined large tables.
2. Replicate suitable small dimension tables.
3. Avoid unnecessary joins.
4. Filter data as early as possible when it reduces the amount of data processed.
5. Review execution plans to identify expensive movement operations.

### Example

Suppose:

- `FactSales` contains 500 million rows.
- `DimCustomer` contains 2 million rows.
- Both tables are distributed by `CustomerID`.

A join on `CustomerID` may avoid significant redistribution when the distributions are compatible.

However, the actual execution plan depends on table definitions, statistics, and query details.

---

## 9. Query Optimization

Writing efficient SQL is important even when the underlying platform supports distributed execution.

### 9.1 Select only required columns

Avoid:

```sql
SELECT *
FROM dbo.FactSales;
```

Prefer:

```sql
SELECT
    CustomerID,
    SaleDate,
    Amount
FROM dbo.FactSales;
```

Reading fewer columns can reduce data scanning and processing.

### 9.2 Filter unnecessary rows

```sql
SELECT
    CustomerID,
    SUM(Amount) AS TotalSales
FROM dbo.FactSales
WHERE SaleDate >= '2026-01-01'
GROUP BY CustomerID;
```

Apply selective filters where appropriate to reduce the amount of data processed.

### 9.3 Avoid unnecessary transformations

Repeatedly applying functions to filter columns can make optimization more difficult.

For example:

```sql
-- Avoid when a direct range predicate is possible
WHERE YEAR(SaleDate) = 2026;
```

Prefer:

```sql
WHERE SaleDate >= '2026-01-01'
  AND SaleDate < '2027-01-01';
```

The range predicate is clearer and can be more optimization-friendly.

### 9.4 Avoid unnecessary DISTINCT

`DISTINCT` requires duplicate elimination and can be expensive on large datasets.

Use it only when duplicate removal is actually required.

### 9.5 Filter before joining

When possible, reduce the input data before joining large tables.

```sql
WITH RecentSales AS
(
    SELECT
        CustomerID,
        Amount
    FROM dbo.FactSales
    WHERE SaleDate >= '2026-01-01'
)
SELECT
    c.CustomerID,
    SUM(s.Amount) AS TotalSales
FROM RecentSales s
JOIN dbo.DimCustomer c
    ON s.CustomerID = c.CustomerID
GROUP BY c.CustomerID;
```

The optimizer may rewrite queries, so a CTE does not guarantee materialization or a particular execution order.

---

## 10. Partitioning

Partitioning divides a large table into smaller logical sections based on a partitioning column.

It is commonly used for large tables that are queried or maintained by date.

Example:

A sales table can be partitioned by `SaleDate`, allowing queries to eliminate irrelevant partitions when filters match the partitioning scheme.

### Benefits

- Can support partition elimination.
- Helps manage large tables.
- Can simplify data loading and retention operations.
- Can support partition switching in suitable designs.

### Important considerations

- Choose a partition column based on query and maintenance patterns.
- Avoid creating excessive numbers of partitions.
- Partitioning does not automatically improve every query.
- Distribution and partitioning solve different problems.

**Remember:**

- Distribution determines how rows are spread across the 60 distributions.
- Partitioning divides data into logical sections within the table.

---

## 11. Workload Management

Dedicated SQL pools use workload management to control how queries consume resources.

Workload management helps manage:
- Query concurrency.
- Resource allocation.
- Workload priorities.
- Isolation between different workloads.

### Why it matters

Imagine a dedicated SQL pool running:

- Large ETL transformations.
- Dashboard queries.
- Ad hoc analytical queries.

If all workloads compete for the same resources, critical reporting queries may become slow.

Workload management helps allocate resources according to workload requirements.

### Important concepts

**Workload classification**
- Assigns requests to workload groups based on classification rules.

**Workload groups**
- Define resource allocation and concurrency behavior.

**Workload importance**
- Helps prioritize requests relative to other workloads.

**Resource classes**
- Provide predefined resource allocations for users or sessions.
- They are a legacy workload-management approach compared with workload groups.

The appropriate configuration depends on workload patterns and the dedicated SQL pool's capacity.

---

## 12. Monitoring and Troubleshooting

Performance optimization should be based on measurements rather than assumptions.

### What to monitor

- Query execution duration.
- Data movement operations.
- Distribution-level workload imbalance.
- Resource utilization.
- Query failures and waits.
- Load duration.
- Concurrency and queueing.
- Statistics freshness.

### Useful tools

**Synapse Studio**
- Monitor SQL requests and activity.
- Inspect query execution details.
- Review dedicated SQL pool health.

**Execution plans**
- Help identify data movement, joins, scans, and other expensive operations.

**Dynamic Management Views (DMVs)**
- Provide information about requests, sessions, distributions, and workload activity.

### Example: Inspect running requests

```sql
SELECT
    request_id,
    session_id,
    status,
    submit_time,
    start_time,
    total_elapsed_time,
    command
FROM sys.dm_pdw_exec_requests
ORDER BY submit_time DESC;
```

This DMV is useful for inspecting SQL requests in a dedicated SQL pool.

### Example: Inspect request steps

```sql
SELECT
    request_id,
    step_index,
    operation_type,
    distribution_type,
    location_type,
    status,
    row_count,
    total_elapsed_time
FROM sys.dm_pdw_request_steps
WHERE request_id = 'QID123';
```

Replace `QID123` with an actual request ID.

Request-step information can help identify expensive operations, including data movement.

**Note:** DMV visibility and permissions depend on the user's database permissions.

---

## 13. Serverless SQL Pool Performance

Serverless SQL pool works differently from dedicated SQL pool.

It queries files in external storage, such as Azure Data Lake Storage.

Performance is influenced by:

- File format.
- File size.
- Number of files.
- Partition layout.
- Amount of data scanned.
- Query predicates.
- Storage access and network conditions.

### 13.1 Prefer columnar formats

For analytical workloads, prefer Parquet or other supported columnar formats over CSV when practical.

Parquet allows the engine to read selected columns rather than parsing every field in every row.

### 13.2 Avoid too many small files

A large number of tiny files can increase file discovery and query overhead.

Organize data into reasonably sized files based on workload and platform guidance.

There is no single ideal file size for every workload.

### 13.3 Use partitioned folder structures

Example:

```text
sales/
    year=2025/
        month=01/
        month=02/
    year=2026/
        month=01/
        month=02/
```

A query filtering on year and month may be able to eliminate irrelevant folders when the external table or query is configured to use the partition information correctly.

### 13.4 Read only required data

Avoid selecting unnecessary columns and scanning unnecessary date ranges.

### Example: Query Parquet files

```sql
SELECT
    CustomerID,
    SUM(Amount) AS TotalSales
FROM OPENROWSET
(
    BULK 'https://mystorage.dfs.core.windows.net/sales/year=2026/*.parquet',
    FORMAT = 'PARQUET'
) AS sales
GROUP BY CustomerID;
```

The storage URL and access permissions must be configured correctly.

**Interview point:** Serverless SQL pool performance is strongly influenced by how data is stored in the data lake, because queries read external files rather than managed dedicated-pool tables.

---

## 14. Dedicated vs Serverless: Optimization Checklist

| Optimization area | Dedicated SQL Pool | Serverless SQL Pool |
|---|---|---|
| Distribution strategy | Very important | Not applicable |
| Data skew | Important | Not a dedicated distribution concern |
| Columnstore index | Important for large analytical tables | Not applicable to external files |
| Statistics | Important | Also relevant to supported external-query scenarios |
| Data movement | Major consideration | Not the same MPP table-distribution model |
| File format | Relevant to loading and staging | Very important |
| File size | Relevant to loading and storage | Very important |
| Partition elimination | Table partitioning | External file/folder partitioning |
| Workload management | Dedicated-pool resource control | Different serverless resource model |

---

## 15. Common Performance Problems and Solutions

| Problem | Possible reason | What to investigate |
|---|---|---|
| Slow joins | Data movement or poor distribution | Distribution keys and request steps |
| Uneven execution | Data skew | Distribution-level row counts |
| Slow aggregations | Large scans or poor data layout | Columnstore, filters, distribution |
| Slow queries after loading | Outdated or insufficient statistics | Statistics maintenance |
| Serverless query is slow | Too many small files or excessive scanning | File format, layout, filters |
| Queries are queued | High concurrency or resource limits | Workload management |
| Large data loads are slow | Poor loading approach or resource pressure | Load method, staging, concurrency |
| Query plan is inefficient | Inaccurate estimates or unsuitable design | Statistics and execution plan |

---

## 16. Practical Performance Optimization Scenario

### Scenario

You have a dedicated SQL pool containing:

- `FactSales`: 500 million rows.
- `DimCustomer`: 2 million rows.
- `DimProduct`: 10,000 rows.

A report joins all three tables and calculates total sales by customer and product.

### Step 1: Review table distribution

- Check how `FactSales` is distributed.
- Check the distribution keys of the dimension tables.
- Consider hash distribution for large fact tables.
- Consider replicated distribution for small dimensions.

### Step 2: Review storage design

- Check whether the fact table uses a clustered columnstore index.
- Review staging-table storage choices.
- Check whether table design matches analytical access patterns.

### Step 3: Review statistics

- Check statistics on join and filter columns.
- Update statistics where necessary.
- Confirm that estimates are reasonable.

### Step 4: Review query execution

- Inspect the execution plan.
- Look for expensive shuffle or broadcast operations.
- Identify large scans and skewed processing.

### Step 5: Optimize the SQL

- Select only required columns.
- Filter rows before joining where appropriate.
- Avoid unnecessary `DISTINCT`.
- Aggregate at the required business grain.

### Step 6: Monitor again

Compare execution duration, data movement, resource use, and result correctness before and after changes.

**Important:** Do not assume a design change is an improvement until you measure its effect.

---

## 17. Common Interview Questions and Answers

### Q1. What is MPP in Azure Synapse?

MPP stands for Massively Parallel Processing. It divides work across multiple compute resources so that different portions of a query can execute in parallel.

### Q2. What are the three distribution types in a dedicated SQL pool?

- Hash distribution.
- Round-robin distribution.
- Replicated distribution.

### Q3. When should you use hash distribution?

Use it for large tables when a suitable distribution key can balance data and support frequently used joins or aggregations.

### Q4. What is data skew?

Data skew occurs when rows are distributed unevenly, causing some distributions to process much more data than others.

### Q5. Why use replicated tables?

Replicated tables are useful for relatively small dimension tables because copies are available on compute nodes, potentially reducing data movement during joins.

### Q6. What is data movement?

Data movement transfers or redistributes data between distributions so that a query operation can execute correctly.

### Q7. Why are columnstore indexes useful?

They store data by column, support compression, and make analytical queries that scan selected columns more efficient.

### Q8. What is the role of statistics?

Statistics help the optimizer estimate row counts and select an appropriate query execution plan.

### Q9. Does partitioning always improve performance?

No. Partitioning helps when query filters and maintenance operations align with the partitioning design. Too many or poorly selected partitions can add overhead.

### Q10. How do you troubleshoot a slow dedicated SQL pool query?

Check the execution plan, request steps, data movement, distribution skew, statistics, workload queues, and resource utilization.

### Q11. How do you improve serverless SQL pool performance?

Use suitable columnar formats such as Parquet, avoid excessive small files, organize data into useful partitions, and scan only required columns and rows.

### Q12. What is the difference between distribution and partitioning?

Distribution determines how rows are spread across the dedicated SQL pool's 60 distributions. Partitioning divides a table into logical sections based on a partitioning column.

### Q13. What is workload management?

It controls how dedicated SQL pool resources are allocated across workloads, helping manage concurrency, priorities, and resource consumption.

### Q14. Is a data movement operation always a problem?

No. Data movement may be necessary for a query. The goal is to reduce unnecessary or expensive movement, not eliminate every movement operation.

### Q15. How do you prove that a performance optimization worked?

Compare execution time, resource consumption, data movement, workload behavior, and result correctness before and after the change under comparable conditions.

---

## 18. Common Mistakes to Avoid

1. Choosing a hash distribution key without checking its value frequency.
2. Using round-robin for every table without considering join patterns.
3. Replicating large tables.
4. Assuming columnstore indexes automatically make every query fast.
5. Ignoring statistics.
6. Confusing partitioning with distribution.
7. Assuming every data movement operation is bad.
8. Using `SELECT *` unnecessarily.
9. Creating excessive small files in a data lake.
10. Assuming serverless SQL pool uses dedicated-pool distribution strategies.
11. Making optimization changes without measuring the results.
12. Ignoring concurrency and workload management.

---

## 19. Quick Revision

| Concept | Remember |
|---|---|
| MPP | Parallel processing across compute resources |
| Dedicated SQL pool | Provisioned analytical compute |
| Serverless SQL pool | Queries external data on demand |
| 60 distributions | Dedicated SQL pool data distribution model |
| Hash | Large tables with a suitable distribution key |
| Round-robin | Simple distribution, often useful for staging |
| Replicated | Small tables copied to compute nodes |
| Data skew | Uneven distribution of rows |
| Columnstore | Efficient analytical storage |
| Statistics | Help the optimizer estimate data |
| Data movement | Redistribution or copying of data |
| Partitioning | Divides data into logical sections |
| Workload management | Controls resource use and concurrency |
| Serverless optimization | File format, file layout, and reduced scanning |

---

## 20. Self-Assessment

Try answering these without looking at the notes:

1. Explain the architecture of a dedicated SQL pool.
2. Why does Synapse use 60 distributions?
3. When would you choose hash, round-robin, and replicated distribution?
4. How can a poor distribution key cause data skew?
5. What is the difference between data distribution and partitioning?
6. Why are columnstore indexes useful for analytical workloads?
7. How do statistics influence query execution?
8. What are shuffle and broadcast data movement operations?
9. How would you troubleshoot a slow join between two large tables?
10. How would you optimize a serverless query over Parquet files?
11. How does workload management help with concurrency?
12. How would you measure whether a performance improvement is successful?

**Interview preparation priority:** Focus first on MPP architecture, the three distribution strategies, data skew, columnstore indexes, statistics, and data movement. These concepts form the foundation of dedicated SQL pool performance optimization.