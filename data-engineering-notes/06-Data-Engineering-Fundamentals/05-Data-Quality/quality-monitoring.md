# Data Quality Monitoring in Data Engineering

## 1. Introduction

**Data quality monitoring is the continuous process of checking, measuring, and tracking data to ensure it remains accurate, complete, consistent, valid, and reliable.**

Data quality checks identify problems at a particular point in a pipeline. Data quality monitoring tracks those checks over time and helps teams detect recurring or newly introduced issues.

For example, an e-commerce pipeline processes order data every hour. The pipeline may run successfully, but the incoming data could still contain:

- Missing customer IDs.
- Duplicate order IDs.
- Negative order amounts.
- Unexpected order statuses.
- Delayed or missing source files.
- Sudden drops in record counts.

Data quality monitoring helps detect these issues before they affect downstream reports and analytics.

**Key idea:** Data quality checks validate data; data quality monitoring continuously tracks the results and alerts teams when quality expectations are not met.

---

## 2. Why Is Data Quality Monitoring Important?

A pipeline can complete successfully while producing incorrect data.

Example:

```text
Source Application
        |
        v
Data Ingestion
        |
        v
Transformation
        |
        v
Delta Table
        |
        v
Dashboard
```

Suppose the ingestion and transformation jobs succeed, but 30% of customer IDs are missing.

The job status is successful, but the data is not reliable.

Monitoring helps identify:

1. Missing or incomplete records.
2. Duplicate records.
3. Invalid values.
4. Unexpected schema changes.
5. Delayed data delivery.
6. Sudden changes in record volume.
7. Failed validation rules.
8. Problems affecting downstream datasets.

### Benefits

- Detect data problems early.
- Improve trust in reports.
- Reduce manual investigation.
- Identify recurring data issues.
- Track quality trends.
- Improve pipeline reliability.
- Support data governance and accountability.

---

## 3. Important Data Quality Dimensions to Monitor

Data quality monitoring tracks multiple dimensions.

### 3.1 Completeness

Checks whether required values are populated.

Example:
- Customer ID must not be NULL.
- Order ID must be present.
- Required source files must arrive.

Metric:

\[
\text{Completeness \%} =
\frac{\text{Populated required values}}
{\text{Total expected values}}
\times 100
\]

### 3.2 Uniqueness

Checks whether records that should be unique contain duplicates.

Example:
- `order_id` must be unique.
- A customer record should not be duplicated for the same business key.

Metric:

\[
\text{Uniqueness \%} =
\frac{\text{Distinct non-NULL keys}}
{\text{Total non-NULL keys}}
\times 100
\]

### 3.3 Validity

Checks whether values follow expected formats, types, ranges, and business rules.

Example:
- Amount must not be negative.
- Status must belong to an approved list.
- Order date must be valid.

### 3.4 Consistency

Checks whether related data agrees across fields or systems.

Example:
- A delivered order should have a delivery date.
- The customer ID in an order should exist in the customer dataset.
- Source and target totals should reconcile.

### 3.5 Accuracy

Checks whether data correctly represents the real-world information it is intended to describe.

Example:
- The recorded order amount should match the source transaction.
- Customer details should match the authoritative source.

Accuracy often requires comparison with a trusted source or external reference; it cannot always be established from the dataset alone.

### 3.6 Timeliness and Freshness

Checks whether data is available when expected.

Example:
- Hourly order data should arrive within the agreed time.
- Daily sales data should be ready before the reporting deadline.

### Quick Comparison

| Dimension | Monitoring Question |
|---|---|
| Completeness | Are required values present? |
| Uniqueness | Are duplicate keys present? |
| Validity | Do values follow defined rules? |
| Consistency | Do related values agree? |
| Accuracy | Does the data reflect the correct information? |
| Freshness | Is the data available on time? |

---

## 4. Important Data Quality Monitoring Metrics

A data engineering team should track measurable indicators rather than relying only on manual inspection.

| Metric | What It Measures | Example |
|---|---|---|
| Record count | Number of incoming records | 10,000 orders |
| NULL percentage | Missing values | 2% missing customer IDs |
| Duplicate count | Duplicate business keys | 150 duplicate orders |
| Invalid record count | Records failing validation | 80 invalid amounts |
| Schema changes | Unexpected structural changes | New or missing column |
| Data freshness | Age of the latest available data | Latest record is 45 minutes old |
| Rejected records | Records moved to quarantine or rejected | 120 records |
| Source-target count difference | Difference between source and target counts | 100 records missing |
| Job success rate | Successful pipeline executions | 98% successful runs |

### Example Monitoring Dashboard

| Metric | Current Value | Expected Value | Status |
|---|---:|---:|---|
| Total records | 10,000 | 9,000–11,000 | Pass |
| Missing customer IDs | 2% | Below 1% | Fail |
| Duplicate order IDs | 0.5% | Below 1% | Pass |
| Invalid amounts | 0.2% | Below 0.5% | Pass |
| Data freshness | 20 min | Below 30 min | Pass |

The thresholds are illustrative. Real thresholds should be agreed upon based on business requirements and historical behavior.

---

## 5. Threshold-Based Monitoring

A threshold defines the acceptable limit for a data quality metric.

Example:

```text
Metric: Missing customer IDs
Expected: Below 1%

If missing percentage < 1%:
    PASS

If missing percentage >= 1%:
    ALERT
```

Thresholds can be defined in different ways.

### 5.1 Fixed Threshold

Uses a predefined value.

Example:

- Duplicate records must be zero.
- Missing customer IDs must be below 1%.
- Data freshness must be below 30 minutes.

### 5.2 Range-Based Threshold

Defines an acceptable range.

Example:

```text
Expected daily order count: 9,000 to 11,000
```

If the count falls outside this range, the system raises an alert.

### 5.3 Historical or Anomaly-Based Threshold

Compares current data with historical behavior.

Example:

- Today's order volume is 60% lower than the usual weekday volume.
- NULL percentage suddenly increases from 1% to 15%.

Anomaly-based monitoring can detect unusual patterns even when a fixed threshold is not violated.

However, a sudden change is not automatically an error. It should be investigated in its business context.

---

## 6. Implementing Data Quality Monitoring in PySpark

Consider an orders DataFrame with these columns:

```text
order_id
customer_id
amount
status
order_date
```

We want to monitor:

- Total records.
- Missing customer IDs.
- Duplicate order IDs.
- Invalid amounts.
- Invalid order statuses.

### 6.1 Calculate Basic Metrics

```python
from pyspark.sql import functions as F

metrics_df = df.agg(
    F.count("*").alias("total_records"),

    F.sum(
        F.when(
            F.col("customer_id").isNull(), 1
        ).otherwise(0)
    ).alias("missing_customer_ids"),

    F.sum(
        F.when(
            F.col("amount") < 0, 1
        ).otherwise(0)
    ).alias("invalid_amounts"),

    F.sum(
        F.when(
            ~F.col("status").isin(
                "Pending",
                "Shipped",
                "Delivered",
                "Cancelled"
            ) | F.col("status").isNull(),
            1
        ).otherwise(0)
    ).alias("invalid_statuses")
)

metrics_df.show()
```

### 6.2 Calculate Duplicate Keys

```python
duplicate_count = (
    df.groupBy("order_id")
    .count()
    .filter(F.col("count") > 1)
    .count()
)
```

This counts the number of distinct duplicated order IDs, not the total number of duplicate rows.

### 6.3 Calculate Missing-Value Percentage

```python
missing_metrics = df.agg(
    (
        F.sum(
            F.when(
                F.col("customer_id").isNull(), 1
            ).otherwise(0)
        ) / F.count("*") * 100
    ).alias("missing_customer_percentage")
)

missing_metrics.show()
```

For an empty dataset, percentage calculations need explicit handling to avoid division by zero.

### 6.4 Apply a Threshold

```python
metrics = metrics_df.first()

total_records = metrics["total_records"]
missing_customer_ids = metrics["missing_customer_ids"]

if total_records == 0:
    print("No records available for validation")

else:
    missing_percentage = (
        missing_customer_ids / total_records
    ) * 100

    if missing_percentage >= 1:
        print("ALERT: Missing customer IDs exceed threshold")
    else:
        print("PASS: Customer ID completeness is acceptable")
```

This is a simple demonstration. In production, monitoring results should be stored in a persistent location and alerts should be integrated with the team's monitoring system.

---

## 7. Data Quality Monitoring in a Medallion Architecture

Data quality monitoring should cover all important stages of a data pipeline.

### Bronze Layer

Monitor:
- Source file arrival.
- Ingestion record count.
- File format and schema.
- Ingestion failures.
- Missing or delayed data.
- Unexpected source changes.

### Silver Layer

Monitor:
- Required-field completeness.
- Duplicate business keys.
- Invalid data types and values.
- Business-rule violations.
- Standardization failures.
- Quarantined records.

### Gold Layer

Monitor:
- Business metric consistency.
- Aggregate correctness.
- Source-to-target reconciliation.
- Reporting freshness.
- Referential consistency.
- Unexpected changes in business totals.

### Example

```text
Source System
      |
      v
Bronze Monitoring
(File Arrival, Schema, Record Count)
      |
      v
Silver Monitoring
(NULL, Duplicates, Validity, Business Rules)
      |
      v
Gold Monitoring
(Aggregates, Reconciliation, Freshness)
      |
      v
Reports and Dashboards
```

The exact checks depend on what each layer represents and how its consumers use the data.

---

## 8. Monitoring Data Freshness

Data freshness measures how recently a dataset has been updated.

Example:

- Expected delivery: Every hour.
- Latest available record: 10:00 AM.
- Current monitoring time: 10:45 AM.
- Freshness delay: 45 minutes.

If the acceptable delay is 30 minutes, the dataset violates its freshness expectation.

### PySpark Example

```python
from pyspark.sql import functions as F

latest_df = df.agg(
    F.max("order_date").alias("latest_order_date")
)

latest_df.show()
```

This finds the latest event timestamp in the data. It does not by itself establish when the source file arrived or when the pipeline completed.

For accurate monitoring, distinguish:

- **Event time:** When the business event occurred.
- **Ingestion time:** When the data entered the platform.
- **Processing time:** When the pipeline processed the data.

A pipeline can ingest data successfully while receiving old business events. Therefore, freshness rules should use the timestamp that matches the business requirement.

---

## 9. Data Quality Monitoring vs Pipeline Monitoring

These concepts are related but different.

### Pipeline Monitoring

Checks whether the pipeline is operating correctly.

Examples:
- Job success or failure.
- Execution duration.
- Task failures.
- Cluster availability.
- Retry count.
- Resource utilization.

### Data Quality Monitoring

Checks whether the data produced by the pipeline meets expected standards.

Examples:
- Missing values.
- Duplicate records.
- Invalid amounts.
- Schema violations.
- Source-target mismatches.
- Data freshness.

### Comparison

| Pipeline Monitoring | Data Quality Monitoring |
|---|---|
| Monitors pipeline execution | Monitors data condition |
| Checks job status | Checks data correctness |
| Tracks execution time | Tracks data freshness |
| Detects task failures | Detects invalid records |
| Monitors infrastructure | Monitors data quality metrics |

**Important:** A successful pipeline execution does not guarantee good data quality.

Both types of monitoring are necessary in production systems.

---

## 10. Alerts and Incident Handling

Monitoring becomes useful when teams can respond to problems.

A typical monitoring workflow is:

```text
Data Pipeline
      |
      v
Quality Checks
      |
      v
Metric Calculation
      |
      v
Threshold Evaluation
      |
      +-------------------+
      |                   |
      v                   v
Checks Pass          Threshold Violated
      |                   |
      v                   v
Continue Pipeline     Create Alert
                          |
                          v
                    Investigate Issue
                          |
                          v
                    Correct / Recover
                          |
                          v
                    Revalidate Data
```

### Common Alert Examples

| Condition | Possible Alert |
|---|---|
| Required column missing | Schema validation failed |
| Duplicate key detected | Duplicate record threshold exceeded |
| NULL percentage too high | Completeness threshold violated |
| Invalid amounts detected | Business validation failed |
| Data not received on time | Freshness SLA breached |
| Record count suddenly drops | Volume anomaly detected |

### Severity Levels

**Critical**
- Pipeline cannot continue safely.
- Required business data is missing.
- Data corruption may affect important reports.

**Warning**
- Quality is degrading but the pipeline may continue under an approved policy.
- A non-critical threshold has been crossed.

**Informational**
- A metric changed but remains within acceptable limits.
- A routine monitoring event occurred.

Severity should be based on business impact, not just the number of failed records.

---

## 11. Monitoring Tools and Technologies

Data quality monitoring can be implemented using different tools.

| Tool / Technology | Common Use |
|---|---|
| PySpark | Custom validation and metric calculation |
| SQL | Data quality queries and reconciliation |
| Databricks Workflows | Job execution and orchestration monitoring |
| Databricks Lakehouse Monitoring | Supported data and model monitoring capabilities |
| Great Expectations | Data validation and expectation-based checks |
| AWS Glue Data Quality | Data quality rules in AWS Glue |
| Soda | Data quality checks and monitoring |
| Prometheus | Metrics collection and alerting infrastructure |
| Grafana | Dashboards and metric visualization |

Tool capabilities, integrations, and licensing vary by platform and version.

A data engineering team does not need every tool. The choice depends on the existing architecture, monitoring requirements, and operational setup.

---

## 12. Data Quality Monitoring Best Practices

1. **Define quality expectations:** Decide which fields and business rules matter.
2. **Monitor important metrics:** Track completeness, uniqueness, validity, consistency, and freshness.
3. **Set meaningful thresholds:** Avoid arbitrary limits.
4. **Monitor trends:** Compare current results with historical behavior.
5. **Validate at multiple stages:** Apply appropriate checks across Bronze, Silver, and Gold.
6. **Store monitoring results:** Maintain historical metrics for analysis.
7. **Configure alerts:** Notify the responsible team when critical rules fail.
8. **Track rejected records:** Record why data failed validation.
9. **Assign ownership:** Make it clear who investigates each issue.
10. **Revalidate after recovery:** Confirm that corrected data meets the contract.
11. **Avoid unnecessary checks:** Focus on rules that protect real business requirements.
12. **Monitor the monitoring process:** Ensure checks themselves are running and producing results.

---

## 13. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Successful job means good data | Pipeline execution and data quality are separate |
| Monitoring only NULL values is enough | Monitor multiple quality dimensions |
| Every threshold violation should stop the pipeline | Action depends on severity and business policy |
| Freshness is always based on event time | Choose event, ingestion, or processing time based on the requirement |
| Duplicate count and duplicate row count are identical | They measure different things |
| A sudden volume change always means an error | Investigate historical and business context |
| Monitoring once is sufficient | Data quality should be tracked continuously |
| Alerts alone solve data problems | Teams also need investigation and recovery processes |
| More validation rules always mean better monitoring | Focus on useful, actionable rules |

---

## 14. Interview Questions and Answers

### Q1. What is data quality monitoring?

It is the continuous process of measuring and tracking data quality to detect missing values, duplicates, invalid records, schema changes, and freshness problems.

### Q2. What is the difference between data quality checks and monitoring?

A quality check validates data against a rule at a particular point. Monitoring tracks the results over time and alerts teams when expectations are not met.

### Q3. What metrics would you monitor in a data pipeline?

I would monitor record counts, NULL percentages, duplicate keys, invalid records, schema changes, source-target count differences, data freshness, and rejected-record counts.

### Q4. Can a pipeline succeed while producing bad data?

Yes. A job may complete successfully while processing incomplete, duplicated, or invalid records. That is why pipeline monitoring and data quality monitoring are separate requirements.

### Q5. How do you monitor duplicates in PySpark?

I group records by the business key, count them, and identify keys with a count greater than one. I also track the number of affected keys or duplicate rows, depending on the monitoring requirement.

### Q6. What is data freshness?

Data freshness measures how recently data has been updated or made available compared with its expected delivery time.

### Q7. What is threshold-based monitoring?

It compares a quality metric with a predefined acceptable limit. If the metric crosses that limit, the monitoring system raises an alert or takes an agreed action.

### Q8. What is anomaly-based monitoring?

It identifies unusual changes compared with historical patterns, such as a sudden drop in daily order volume or an unexpected increase in NULL values.

### Q9. How do you monitor data quality in a medallion architecture?

I monitor ingestion and schema issues in Bronze, completeness and validity in Silver, and business metrics, reconciliation, and freshness in Gold.

### Q10. What would you do if the NULL percentage suddenly increased?

I would identify the affected column and source, compare the current result with historical metrics, inspect recent source or schema changes, isolate affected records, notify the owner, and apply the agreed recovery process.

### Q11. What is the difference between pipeline monitoring and data quality monitoring?

Pipeline monitoring checks execution health, such as job failures and execution time. Data quality monitoring checks the condition of the resulting data, such as missing values, duplicates, and invalid records.

### Q12. How do you monitor data freshness?

I compare the latest relevant timestamp or successful delivery time with the expected freshness threshold. I distinguish event time, ingestion time, and processing time to avoid misleading results.

---

## 15. Quick Revision

| Topic | Key Point |
|---|---|
| Data quality monitoring | Continuous measurement of data quality |
| Completeness | Missing-value monitoring |
| Uniqueness | Duplicate-key monitoring |
| Validity | Format, range, and business-rule monitoring |
| Consistency | Related data agrees |
| Accuracy | Data represents correct information |
| Freshness | Data is available on time |
| Threshold | Acceptable metric limit |
| Anomaly detection | Detects unusual patterns |
| Alerting | Notifies teams about issues |
| Pipeline monitoring | Tracks execution health |
| Data quality monitoring | Tracks data condition |

### One-Minute Interview Answer

Data quality monitoring is the continuous process of tracking whether data meets expected quality standards.

In a production data pipeline, I would monitor completeness, uniqueness, validity, consistency, and freshness. For example, I would track missing customer IDs, duplicate order IDs, invalid amounts, record counts, and the latest available data timestamp.

Using PySpark and SQL, I can calculate these metrics and compare them against predefined thresholds. If a threshold is violated, the system can raise an alert, quarantine invalid records, or stop the pipeline depending on the business impact.

I would also maintain historical monitoring results so the team can identify recurring issues and unusual changes over time.

---

## 16. Self-Assessment

- [ ] Define data quality monitoring.
- [ ] Explain why it is important.
- [ ] Identify the main data quality dimensions.
- [ ] Calculate completeness and uniqueness metrics.
- [ ] Monitor invalid records and schema changes.
- [ ] Explain threshold-based monitoring.
- [ ] Explain anomaly-based monitoring.
- [ ] Calculate monitoring metrics using PySpark.
- [ ] Monitor data freshness correctly.
- [ ] Differentiate pipeline monitoring and data quality monitoring.
- [ ] Explain monitoring across Bronze, Silver, and Gold.
- [ ] Describe alerts and incident handling.
- [ ] Explain how to investigate a sudden increase in NULL values.

**Interview readiness:** Be able to explain how you would monitor an hourly orders pipeline, detect missing records or duplicate IDs, track freshness, and alert the team before unreliable data reaches downstream dashboards.