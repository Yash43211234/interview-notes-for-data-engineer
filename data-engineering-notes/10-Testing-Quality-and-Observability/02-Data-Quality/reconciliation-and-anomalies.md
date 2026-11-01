

# Data Reconciliation and Anomaly Detection

## 1. Introduction

Data reconciliation and anomaly detection are important practices in data engineering. They help verify that data is transferred correctly between systems and identify unusual changes in datasets.

- **Data Reconciliation:** Compares data between two or more systems to confirm that records and values are consistent.
- **Anomaly Detection:** Identifies unusual patterns, values, or changes that differ from expected behavior.

### Example

Suppose an e-commerce company moves order data from a source database into a Delta Lake table.

```text
Source Database
      |
      v
Data Ingestion
      |
      v
PySpark Transformation
      |
      v
Delta Lake
```

After processing, we need to verify:

- Did all expected records reach Delta Lake?
- Does the total order amount match?
- Are any records missing?
- Are duplicate records present?
- Did today's order volume suddenly decrease?
- Is the average order value unusually high?

Reconciliation checks correctness between systems. Anomaly detection identifies unusual behavior that may require investigation.

---

## 2. What Is Data Reconciliation?

Data reconciliation is the process of comparing data from different stages or systems to verify that the data has been transferred and transformed correctly.

It is commonly used during:

- ETL and ELT processing.
- Database migrations.
- Data warehouse loading.
- Batch pipeline execution.
- Financial data processing.
- Source-to-target validation.

### Example

Suppose the source system contains:

| order_id | amount |
|---:|---:|
| 101 | 500 |
| 102 | 300 |
| 103 | 200 |

The target table contains:

| order_id | amount |
|---:|---:|
| 101 | 500 |
| 102 | 300 |
| 103 | 200 |

The source and target have:

- Source count: 3
- Target count: 3
- Source total: 1,000
- Target total: 1,000

The basic reconciliation checks pass.

However, matching counts and totals alone do not prove that every record is identical. We should also compare keys and relevant values.

---

## 3. Types of Data Reconciliation

### 3.1 Record Count Reconciliation

Compares the number of records between source and target.

```sql
SELECT COUNT(*) AS source_count
FROM source_orders;

SELECT COUNT(*) AS target_count
FROM target_orders;
```

Expected condition:

\[
SourceCount = TargetCount
\]

If the source has 10,000 records and the target has 9,800, there may be missing records or intentional filtering.

**Important:** Record counts should be compared after accounting for expected filters, deduplication, and business transformations.

### 3.2 Aggregate Reconciliation

Compares aggregate values such as:

- Total sales.
- Total quantity.
- Total revenue.
- Total transaction amount.

SQL:

```sql
SELECT
    COUNT(*) AS total_records,
    SUM(amount) AS total_amount
FROM source_orders;
```

Run the same query on the target.

Expected conditions:

\[
SourceTotalAmount = TargetTotalAmount
\]

Aggregate comparisons are useful, but different records can sometimes produce the same total.

### 3.3 Key-Level Reconciliation

Compares primary keys or business keys to identify missing records.

SQL:

```sql
SELECT order_id
FROM source_orders

EXCEPT

SELECT order_id
FROM target_orders;
```

This identifies keys present in the source but absent from the target.

For the reverse comparison:

```sql
SELECT order_id
FROM target_orders

EXCEPT

SELECT order_id
FROM source_orders;
```

This identifies keys present in the target but absent from the source.

**Note:** `EXCEPT` generally returns distinct rows. If duplicate multiplicity matters, use grouped counts or another comparison method that accounts for duplicates.

### 3.4 Column-Level Reconciliation

Compares important column values for matching records.

```sql
SELECT
    s.order_id,
    s.amount AS source_amount,
    t.amount AS target_amount
FROM source_orders s
JOIN target_orders t
    ON s.order_id = t.order_id
WHERE
    s.amount <> t.amount
    OR (s.amount IS NULL AND t.amount IS NOT NULL)
    OR (s.amount IS NOT NULL AND t.amount IS NULL);
```

This identifies records where the amount differs, including NULL mismatches.

### 3.5 Date-Wise Reconciliation

Compares records and totals by date.

```sql
SELECT
    order_date,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_amount
FROM source_orders
GROUP BY order_date
ORDER BY order_date;
```

Run the equivalent query on the target.

Date-wise reconciliation helps identify:

- Missing daily data.
- Partial loads.
- Unexpected changes in daily totals.
- Incorrect date transformations.

### 3.6 Hash-Based Reconciliation

A hash can summarize selected column values for comparison.

For example, a database may calculate a row hash from `order_id`, `customer_id`, and `amount`.

Conceptual SQL:

```sql
SELECT
    order_id,
    HASH(order_id, customer_id, amount) AS row_hash
FROM source_orders;
```

Compare the corresponding hashes in the target.

Hash functions and syntax vary by database.

**Important:** Hashes are useful for efficient comparison, but they are not a mathematical guarantee of equality because hash collisions are possible. Define consistent data types, NULL handling, column order, and normalization before comparing hashes.

---

## 4. Reconciliation Using PySpark

Suppose we have source and target DataFrames.

```python
source_data = [
    (101, 500.0),
    (102, 300.0),
    (103, 200.0)
]

target_data = [
    (101, 500.0),
    (102, 300.0),
    (103, 200.0)
]

source_df = spark.createDataFrame(
    source_data,
    ["order_id", "amount"]
)

target_df = spark.createDataFrame(
    target_data,
    ["order_id", "amount"]
)
```

### 4.1 Compare Record Counts

```python
source_count = source_df.count()
target_count = target_df.count()

assert source_count == target_count
```

### 4.2 Compare Aggregate Values

```python
from pyspark.sql.functions import sum, count

source_summary = source_df.agg(
    count("*").alias("record_count"),
    sum("amount").alias("total_amount")
)

target_summary = target_df.agg(
    count("*").alias("record_count"),
    sum("amount").alias("total_amount")
)
```

Compare the summary results after collecting the small aggregate rows.

For financial values, use an appropriate decimal type rather than floating-point values when exact decimal precision is required.

### 4.3 Find Missing Records

```python
source_only = source_df.join(
    target_df.select("order_id"),
    on="order_id",
    how="left_anti"
)

target_only = target_df.join(
    source_df.select("order_id"),
    on="order_id",
    how="left_anti"
)
```

- `source_only`: Records present in source but missing in target.
- `target_only`: Records present in target but missing in source.

### 4.4 Find Value Mismatches

```python
from pyspark.sql.functions import col

mismatches = (
    source_df.alias("s")
    .join(
        target_df.alias("t"),
        on="order_id",
        how="inner"
    )
    .filter(
        ~col("s.amount").eqNullSafe(
            col("t.amount")
        )
    )
)
```

`eqNullSafe()` handles NULL comparisons explicitly.

This example assumes `order_id` uniquely identifies a record in both datasets. Duplicate keys should be investigated before relying on a simple key-based join.

---

## 5. What Is Anomaly Detection?

Anomaly detection is the process of identifying observations or patterns that differ significantly from expected behavior.

An anomaly does not always mean that data is incorrect. It indicates something unusual that may require investigation.

### Example

Suppose an e-commerce platform normally receives between 8,000 and 12,000 orders per day.

| Date | Orders |
|---|---:|
| Monday | 9,500 |
| Tuesday | 10,200 |
| Wednesday | 9,800 |
| Thursday | 10,100 |
| Friday | 1,200 |

Friday's order count is significantly lower than normal.

Possible reasons:

- Source system failure.
- Incomplete ingestion.
- Missing files.
- Incorrect date filter.
- Actual reduction in business activity.

Anomaly detection identifies the unusual change. Further investigation determines its cause.

---

## 6. Types of Data Anomalies

### 6.1 Point Anomaly

A single data point is significantly different from expected values.

Example:

Most order amounts are between ₹100 and ₹5,000, but one order amount is ₹5,00,000.

### 6.2 Contextual Anomaly

A value is unusual only in a particular context, such as a specific date, location, or season.

Example:

- 10,000 orders on a normal weekday may be expected.
- 10,000 orders during a low-traffic period may be unusually high.

### 6.3 Collective Anomaly

A group of records collectively shows unusual behavior, even if individual records appear normal.

Example:

A payment system normally receives transactions throughout the day, but all transactions suddenly arrive within a few minutes.

### 6.4 Data Pipeline Anomaly

An unexpected change occurs in pipeline behavior or data characteristics.

Examples:

- Sudden drop in record count.
- Unexpected increase in NULL values.
- Large increase in duplicate records.
- Missing daily partition.
- Sudden change in average transaction amount.
- Unexpected schema change.

---

## 7. Common Anomaly Detection Techniques

### 7.1 Threshold-Based Detection

Uses predefined limits to identify unusual values.

Example:

Alert if daily order count falls below 5,000.

```python
daily_orders = 1200

if daily_orders < 5000:
    print("Anomaly detected")
```

Advantages:
- Easy to understand.
- Easy to implement.
- Suitable for known business limits.

Limitations:
- Requires appropriate thresholds.
- May not adapt to changing business patterns.
- Can generate false alerts.

### 7.2 Percentage Change Detection

Compares current values with previous values.

Formula:

\[
PercentageChange =
\frac{Current - Previous}{Previous}
\times 100
\]

Example:

Previous day orders = 10,000

Current day orders = 8,000

\[
\frac{8000-10000}{10000}\times100=-20\%
\]

The order count decreased by 20%.

Python:

```python
previous = 10000
current = 8000

if previous != 0:
    change = (
        (current - previous) / previous
    ) * 100

    print(f"Change: {change:.2f}%")
```

A percentage-change alert should consider normal daily and seasonal variations.

### 7.3 Z-Score Method

The Z-score measures how far a value is from the mean in standard deviation units.

Formula:

\[
Z = \frac{x-\mu}{\sigma}
\]

Where:

- \(x\) = Current value.
- \(\mu\) = Mean.
- \(\sigma\) = Standard deviation.

Example:

Suppose daily orders have:

- Mean = 10,000
- Standard deviation = 1,000
- Current orders = 7,000

\[
Z = \frac{7000-10000}{1000}=-3
\]

The current value is three standard deviations below the mean.

A common rule is to flag observations with:

\[
|Z| > 3
\]

This is a heuristic, not a universal rule. It works best when the distribution and comparison period make the mean and standard deviation meaningful.

Python:

```python
import numpy as np

values = [9500, 10200, 9800, 10100, 9700]

mean = np.mean(values)
std = np.std(values)

current = 7000

if std > 0:
    z_score = (current - mean) / std

    if abs(z_score) > 3:
        print("Anomaly detected")
```

For production monitoring, calculate statistics from an appropriate historical baseline and avoid including the current anomalous value in its own baseline.

### 7.4 Interquartile Range (IQR)

IQR is a statistical method used to identify unusually low or high values.

Formula:

\[
IQR = Q3 - Q1
\]

Where:

- \(Q1\) = First quartile.
- \(Q3\) = Third quartile.

Lower bound:

\[
Q1 - 1.5 \times IQR
\]

Upper bound:

\[
Q3 + 1.5 \times IQR
\]

Values outside these bounds are commonly flagged as potential outliers.

Python:

```python
import numpy as np

values = [100, 120, 130, 140, 150, 160, 1000]

q1 = np.percentile(values, 25)
q3 = np.percentile(values, 75)

iqr = q3 - q1

lower_bound = q1 - 1.5 * iqr
upper_bound = q3 + 1.5 * iqr

anomalies = [
    value
    for value in values
    if value < lower_bound or value > upper_bound
]

print(anomalies)
```

IQR is useful for identifying potential outliers in numerical data. It does not automatically determine whether an unusual value is wrong.

### 7.5 Moving Average

A moving average compares current values with a rolling historical average.

Example:

Calculate the average order count over the previous seven days and compare today's count with that baseline.

SQL:

```sql
SELECT
    order_date,
    order_count,
    AVG(order_count) OVER (
        ORDER BY order_date
        ROWS BETWEEN 7 PRECEDING AND 1 PRECEDING
    ) AS previous_7_day_avg
FROM daily_order_summary;
```

This assumes one row per calendar day. If dates are missing, first create a complete date series or calendar table so that seven rows represent seven days.

### 7.6 Machine Learning-Based Detection

For complex datasets, machine learning techniques can identify unusual patterns.

Examples:

- Isolation Forest.
- Local Outlier Factor.
- One-Class SVM.
- Clustering-based detection.
- Time-series forecasting and residual analysis.

These methods may be useful when simple thresholds or statistical rules are insufficient.

However, they require suitable features, training data, validation, and monitoring. A complex model is not automatically better than a simple business rule.

---

## 8. Reconciliation vs Anomaly Detection

| Feature | Data Reconciliation | Anomaly Detection |
|---|---|---|
| Purpose | Verify consistency between datasets | Identify unusual behavior |
| Main focus | Source-to-target correctness | Unexpected patterns or values |
| Comparison | Usually two or more datasets | Current data against expected behavior |
| Example | Source count vs target count | Sudden 80% drop in daily orders |
| Techniques | Counts, sums, joins, hashes | Thresholds, Z-score, IQR, ML |
| Result | Matching or mismatching data | Normal or potentially unusual behavior |
| Typical use | ETL validation and migration | Monitoring and early warning |

**Simple difference:**

- Reconciliation asks: Did the data move correctly?
- Anomaly detection asks: Does the data look unusual?

Both can be used together.

---

## 9. Reconciliation and Anomaly Detection in a Data Pipeline

Consider a daily order processing pipeline.

```text
       Source Database
              |
              v
       Data Ingestion
              |
              v
         Bronze Layer
              |
              v
       Data Validation
              |
              v
        Silver Layer
              |
              v
       Reconciliation
              |
              v
         Gold Layer
              |
              v
       Anomaly Monitoring
              |
              v
       Reports and Alerts
```

### Step 1: Ingest Data

Read source data into the Bronze layer.

### Step 2: Validate Data

Check:
- Required columns.
- NULL values.
- Duplicate keys.
- Invalid values.

### Step 3: Transform Data

Apply business transformations and write the Silver layer.

### Step 4: Reconcile Data

Compare:
- Source and target record counts.
- Aggregate amounts.
- Missing keys.
- Important column values.

### Step 5: Monitor Anomalies

Check:
- Daily order volume.
- Revenue trends.
- NULL percentage.
- Duplicate percentage.
- Data freshness.

### Step 6: Take Action

Depending on severity:
- Stop the pipeline.
- Quarantine invalid data.
- Send an alert.
- Investigate the source.
- Reprocess affected data.

---

## 10. Practical Example: Source-to-Target Reconciliation

Suppose an ETL pipeline transfers customer orders from a source database into a Delta table.

Source:

| order_id | amount |
|---:|---:|
| 101 | 500 |
| 102 | 300 |
| 103 | 200 |
| 104 | 400 |

Target:

| order_id | amount |
|---:|---:|
| 101 | 500 |
| 102 | 300 |
| 103 | 250 |

### Reconciliation Results

| Check | Source | Target | Result |
|---|---:|---:|---|
| Record count | 4 | 3 | Mismatch |
| Total amount | 1,400 | 1,050 | Mismatch |
| Missing order IDs | — | 104 | Mismatch |
| Amount mismatch | Order 103: 200 | Order 103: 250 | Mismatch |

The reconciliation identifies:
- One missing order.
- One amount mismatch.
- A difference in record counts.
- A difference in total amount.

The pipeline should investigate these differences before treating the target as fully reconciled.

---

## 11. Handling Reconciliation Failures

When reconciliation fails, the pipeline should follow a defined process.

### 11.1 Identify the Failure

Determine which check failed:
- Count mismatch.
- Missing records.
- Duplicate records.
- Aggregate mismatch.
- Column-level mismatch.

### 11.2 Investigate the Cause

Possible reasons:
- Incomplete source extraction.
- Incorrect filtering.
- Transformation logic defect.
- Duplicate ingestion.
- Late-arriving records.
- Incorrect join condition.
- Failed write operation.

### 11.3 Decide the Response

| Situation | Possible Action |
|---|---|
| Critical missing records | Stop or hold downstream processing |
| Small approved variation | Record and continue |
| Duplicate records | Investigate and deduplicate according to business rules |
| Invalid records | Quarantine |
| Partial load | Retry or reprocess safely |
| Unexpected anomaly | Alert and investigate |

### 11.4 Record Reconciliation Results

Maintain an audit table with fields such as:

| Column | Description |
|---|---|
| pipeline_name | Name of the pipeline |
| run_id | Unique pipeline execution identifier |
| source_count | Source record count |
| target_count | Target record count |
| source_total | Source aggregate |
| target_total | Target aggregate |
| missing_count | Missing records |
| mismatch_count | Records with value differences |
| status | Passed or failed |
| checked_at | Validation timestamp |

This makes reconciliation results traceable across pipeline runs.

---

## 12. Best Practices

1. Define reconciliation rules before pipeline development.
2. Compare data at compatible grains.
3. Validate record counts and important business aggregates.
4. Compare keys and column values when required.
5. Handle NULL values explicitly.
6. Check duplicate multiplicity where relevant.
7. Use decimal types for financial comparisons.
8. Account for late-arriving and intentionally filtered records.
9. Maintain historical reconciliation results.
10. Define acceptable tolerances for approximate comparisons.
11. Establish clear alert thresholds.
12. Separate genuine anomalies from expected business seasonality.
13. Avoid treating every statistical outlier as bad data.
14. Make pipeline recovery and reprocessing safe.
15. Monitor both data correctness and pipeline freshness.

---

## 13. Common Interview Questions and Answers

### Q1. What is data reconciliation?

Data reconciliation is the process of comparing source and target datasets to verify that data has been transferred or transformed correctly.

### Q2. Why is reconciliation important in ETL pipelines?

It helps identify missing records, duplicate records, incorrect values, and incomplete data loads.

### Q3. What are common reconciliation techniques?

Record count comparison, aggregate comparison, key-level comparison, column-level comparison, date-wise comparison, and hash-based comparison.

### Q4. Is matching record count enough to prove that data is correct?

No. Two datasets can have the same number of records but different keys or values. We should also compare important keys and business values.

### Q5. What is anomaly detection?

Anomaly detection identifies unusual values or patterns that differ from expected behavior.

### Q6. What is a point anomaly?

A single observation that differs significantly from the expected range.

### Q7. What is a contextual anomaly?

A value that is unusual in a particular context, such as a specific date, location, or season.

### Q8. What is a collective anomaly?

A group of records that collectively displays unusual behavior.

### Q9. What is the Z-score?

The Z-score measures how many standard deviations a value is away from the mean.

### Q10. What is IQR?

IQR is the difference between the third quartile and the first quartile. It is commonly used to identify potential outliers.

### Q11. What is the difference between reconciliation and anomaly detection?

Reconciliation verifies consistency between datasets, while anomaly detection identifies unusual patterns in data.

### Q12. How do you find missing records in PySpark?

Use a left anti join between the source and target DataFrames on the appropriate business key.

### Q13. How do you handle reconciliation failures?

Identify the failed check, investigate the cause, record the results, and decide whether to retry, reprocess, quarantine, alert, or stop the pipeline.

### Q14. Can an anomaly be valid data?

Yes. A sudden increase in sales may be caused by a genuine promotion. An anomaly requires investigation; it is not automatically an error.

### Q15. How would you monitor daily order volume?

Compare daily counts with historical baselines, use appropriate thresholds or statistical methods, and alert when unexpected changes occur.

---

## 14. Quick Revision

| Concept | Meaning |
|---|---|
| Data Reconciliation | Compares data between systems |
| Record Count Check | Compares total records |
| Aggregate Check | Compares sums and other metrics |
| Key-Level Check | Finds missing or extra keys |
| Column-Level Check | Finds mismatched values |
| Hash-Based Check | Compares row fingerprints |
| Anomaly Detection | Identifies unusual patterns |
| Point Anomaly | One unusual observation |
| Contextual Anomaly | Unusual in a specific context |
| Collective Anomaly | A group with unusual behavior |
| Z-Score | Distance from mean in standard deviations |
| IQR | Quartile-based outlier detection |
| Moving Average | Rolling historical average |
| Quarantine | Separates records requiring investigation |

### Remember These Five Points

1. Reconciliation verifies whether source and target data match according to defined rules.
2. Count and aggregate checks alone are not sufficient for complete validation.
3. Anomaly detection identifies unusual patterns, not necessarily incorrect data.
4. Z-score and IQR are common statistical detection techniques.
5. Reconciliation results and anomalies should be logged, monitored, and investigated.

---

## 15. Self-Assessment

Try answering these questions without looking at the notes.

1. What is data reconciliation?
2. What is anomaly detection?
3. Explain the difference between reconciliation and anomaly detection.
4. How do you compare source and target record counts?
5. How do you identify missing records using SQL?
6. How do you find missing records using PySpark?
7. Why are aggregate checks alone insufficient?
8. What is hash-based reconciliation?
9. What is a point anomaly?
10. Explain contextual and collective anomalies.
11. What is a Z-score?
12. What is IQR?
13. How does a moving average help detect anomalies?
14. How would you investigate a sudden drop in daily order volume?
15. Design a source-to-target reconciliation process for an ADLS-to-Delta pipeline.

**Interview readiness:** Be able to explain source-to-target reconciliation, write SQL and PySpark comparisons, describe Z-score and IQR, and design a monitoring strategy for a production data pipeline.

