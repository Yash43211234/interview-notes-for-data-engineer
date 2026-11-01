

# Pipeline Monitoring in Data Engineering

## 1. Introduction

Pipeline monitoring is the process of continuously tracking the execution, performance, reliability, and data health of data pipelines.

It helps data engineers understand whether pipelines are running correctly and delivering data within the expected time.

A production data pipeline should not only produce the correct output. It should also:
- Complete within the expected time.
- Process the expected volume of data.
- Handle failures safely.
- Deliver fresh and reliable data.
- Provide enough information for troubleshooting.

### Example

Suppose an organization runs a daily customer-orders pipeline.

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
      |
      v
Data Warehouse
      |
      v
Business Reports
```

The pipeline is expected to finish by 7:00 AM.

Pipeline monitoring helps answer:

- Did the pipeline run successfully?
- How long did it take?
- How many records were processed?
- Were any records rejected?
- Is the output data available by 7:00 AM?
- Did any task fail or retry?

---

## 2. What Is Pipeline Monitoring?

Pipeline monitoring is the practice of observing pipeline activities and collecting information about their execution.

It typically tracks:

| Monitoring Area | What It Checks |
|---|---|
| Execution status | Success, failure, running, or cancelled |
| Duration | How long the pipeline takes |
| Data volume | Number of records processed |
| Data quality | NULLs, duplicates, invalid records |
| Freshness | Whether data is updated on time |
| Performance | CPU, memory, shuffle, and processing speed |
| Reliability | Failures, retries, and recovery |
| Dependencies | Whether upstream tasks completed |

### Why Is Pipeline Monitoring Important?

1. Detects failures early.
2. Prevents downstream pipelines from consuming incomplete data.
3. Identifies performance degradation.
4. Helps maintain data freshness.
5. Supports incident investigation.
6. Improves production reliability.
7. Provides visibility into pipeline health.
8. Helps meet business delivery expectations.

---

## 3. Pipeline Monitoring vs Observability

These concepts are related but not identical.

| Pipeline Monitoring | Observability |
|---|---|
| Tracks pipeline health and execution | Provides deeper understanding of system behavior |
| Focuses on predefined checks | Helps investigate known and unknown issues |
| Uses statuses, durations, and alerts | Uses logs, metrics, traces, and context |
| Example: Alert when a pipeline fails | Example: Investigate why a pipeline became slow |

Monitoring is an important part of observability.

A monitoring system can tell you that a pipeline failed. Observability information helps you investigate why it failed.

---

## 4. What Should Be Monitored?

### 4.1 Pipeline Execution Status

Track the state of every pipeline run and task.

Common statuses include:

- Queued.
- Running.
- Succeeded.
- Failed.
- Cancelled.
- Skipped.

Exact status names depend on the orchestration or processing platform.

Example:

| Pipeline | Status |
|---|---|
| Customer Ingestion | Succeeded |
| Orders Transformation | Succeeded |
| Payment Processing | Failed |
| Sales Aggregation | Skipped |

If payment processing fails, downstream sales aggregation may be skipped or prevented from running.

### 4.2 Pipeline Duration

Pipeline duration is the time taken to complete a pipeline run.

\[
\text{Duration} = \text{End Time} - \text{Start Time}
\]

Example:

- Start time: 6:00 AM
- End time: 6:35 AM

Duration = 35 minutes.

If the normal duration is 20 minutes, the pipeline may need investigation.

Monitor:
- Total pipeline duration.
- Individual task duration.
- Average execution duration.
- Maximum execution duration.
- Duration trends over time.

### 4.3 Data Volume

Data volume monitoring tracks how many records are ingested, transformed, and loaded.

Example:

| Stage | Records |
|---|---:|
| Source | 100,000 |
| Raw | 100,000 |
| Validated | 98,000 |
| Rejected | 2,000 |
| Curated | 98,000 |

This helps identify unexpected data loss or volume changes.

However, differences between stages are not automatically errors. Filtering, deduplication, and business rules may intentionally change record counts.

### 4.4 Data Freshness

Data freshness measures how recently data was updated.

Example:

Expected delivery: 7:00 AM.

Actual delivery: 8:00 AM.

The data is one hour late relative to the expected delivery time.

A freshness check can be based on:
- Latest source event timestamp.
- Latest successful ingestion timestamp.
- Latest target-table update timestamp.
- Expected business delivery deadline.

Choose the timestamp that reflects the actual business requirement.

### 4.5 Data Quality

Monitor whether the data meets defined quality rules.

Common checks:

| Check | Example |
|---|---|
| Completeness | Required columns are not NULL |
| Uniqueness | Order IDs are unique |
| Validity | Order amount is non-negative |
| Consistency | Customer ID exists in customer dimension |
| Accuracy | Values match a trusted source |
| Schema | Required columns and data types are present |

### 4.6 Resource Utilization

Monitor infrastructure and processing resources.

Examples:
- CPU utilization.
- Memory usage.
- Disk utilization.
- Network throughput.
- Executor failures.
- Shuffle read and write.
- Memory and disk spill.

For Spark pipelines, these metrics help identify resource pressure and inefficient transformations.

---

## 5. Important Pipeline Monitoring Metrics

| Metric | Meaning |
|---|---|
| Success Rate | Percentage of successful pipeline runs |
| Failure Rate | Percentage of failed pipeline runs |
| Pipeline Duration | Total execution time |
| Throughput | Records processed per unit of time |
| Data Freshness | How current the data is |
| Record Count | Number of processed records |
| Retry Count | Number of retry attempts |
| Data Quality Failure Count | Number of failed quality checks |
| Resource Utilization | CPU, memory, disk, and network usage |
| Pipeline Availability | Whether the pipeline is available when required |

### 5.1 Success Rate

\[
\text{Success Rate} =
\frac{\text{Successful Runs}}{\text{Total Runs}}
\times 100
\]

Example:

- Total runs = 100
- Successful runs = 95

\[
\text{Success Rate} = 95\%
\]

### 5.2 Failure Rate

\[
\text{Failure Rate} =
\frac{\text{Failed Runs}}{\text{Total Runs}}
\times 100
\]

Example:

- Total runs = 100
- Failed runs = 5

Failure rate = 5%.

Define whether retries count as separate attempts or whether the metric counts final pipeline-run outcomes. Otherwise, the metric can be misleading.

### 5.3 Throughput

Throughput measures how much data is processed per unit of time.

\[
\text{Throughput} =
\frac{\text{Records Processed}}{\text{Processing Time}}
\]

Example:

- Records processed = 600,000
- Processing time = 10 minutes = 600 seconds

\[
\text{Throughput} = 1,000 \text{ records/second}
\]

### 5.4 Retry Count

Retry count measures how many times a failed task or operation is attempted again.

A high retry count may indicate:
- Temporary network problems.
- Source-system instability.
- Database connection issues.
- Resource pressure.
- Unhandled application errors.

Retries should be monitored because repeated retries can increase execution time and cost.

---

## 6. Pipeline Monitoring Architecture

A typical monitoring architecture looks like this:

```text
        Data Sources
             |
             v
       Data Pipeline
             |
             v
       Pipeline Execution
             |
             v
    Logs + Metrics + Events
             |
             v
      Monitoring Platform
             |
       /             \
      v               v
  Dashboard        Alert Rules
      |               |
      v               v
  Engineers       Notifications
                      |
                      v
                Incident Handling
```

### Main Components

**1. Data Pipeline**

Executes ingestion, transformation, and loading activities.

**2. Telemetry Collection**

Collects logs, metrics, events, and execution details.

**3. Monitoring Platform**

Stores and analyzes the collected information.

**4. Dashboard**

Displays pipeline health and performance.

**5. Alert Rules**

Detect abnormal conditions and trigger notifications.

**6. Incident Management**

Tracks investigation, ownership, recovery, and resolution.

---

## 7. Monitoring in Azure Data Factory

Azure Data Factory provides pipeline and activity-level monitoring.

### What Can You Monitor?

- Pipeline run status.
- Activity run status.
- Execution duration.
- Activity input and output.
- Error messages.
- Retry attempts.
- Trigger execution.
- Integration Runtime activity.

### ADF Monitoring Flow

```text
ADF Pipeline
     |
     v
Activity Execution
     |
     v
Pipeline Run Details
     |
     v
ADF Monitor
     |
     v
Failure / Duration Analysis
     |
     v
Alert or Investigation
```

### ADF Monitoring Interface

In Azure Data Factory Studio, the Monitor section helps inspect:

- Pipeline runs.
- Trigger runs.
- Activity runs.
- Integration Runtime status.
- Execution details.

### Diagnostic Settings

ADF diagnostic settings can route supported resource logs and metrics to destinations such as:

- Log Analytics workspace.
- Storage account.
- Event Hubs.

This enables centralized monitoring beyond the ADF Studio interface.

### Example KQL Query

The following is an illustrative query for ADF pipeline-run logs in Log Analytics:

```kusto
ADFPipelineRun
| where TimeGenerated > ago(24h)
| summarize RunCount = count()
    by Status
```

The table must be available in the workspace, and diagnostic settings must be configured to collect the relevant logs.

---

## 8. Monitoring in Azure Databricks

Databricks provides job-level and Spark-level execution information.

### Job-Level Monitoring

Monitor:
- Job run status.
- Task status.
- Start and end time.
- Execution duration.
- Retry attempts.
- Job failures.

### Spark-Level Monitoring

The Spark UI provides information about:

- Jobs.
- Stages.
- Tasks.
- Executors.
- SQL queries.
- Shuffle operations.
- Memory usage.
- Failed tasks.
- Execution time.

### Example: Spark Performance Investigation

Suppose a Spark job normally completes in 15 minutes but now takes 50 minutes.

Check:

1. Did input data volume increase?
2. Did shuffle read/write increase?
3. Are some tasks significantly slower than others?
4. Are executors failing?
5. Is data skew affecting a join?
6. Is the job spilling data to disk?
7. Did the source or target become slower?

### Databricks Monitoring Flow

```text
Databricks Job
      |
      v
Task Execution
      |
      v
Job Run Information
      |
      v
Spark UI + Logs + Metrics
      |
      v
Performance Investigation
      |
      v
Optimization / Recovery
```

---

## 9. Monitoring in Apache Airflow

Airflow provides monitoring for DAG runs and individual task instances.

### What Can You Monitor?

- DAG run status.
- Task status.
- Task duration.
- Retry count.
- Scheduling delays.
- Task dependencies.
- Execution logs.
- Scheduler and worker health.

### Airflow Monitoring Interface

The Airflow UI provides:

- Grid View.
- Graph View.
- DAG run details.
- Task instance details.
- Task logs.
- Duration and retry information.

### Airflow Task States

| State | Meaning |
|---|---|
| `scheduled` | Ready for scheduling |
| `queued` | Sent to the executor for execution |
| `running` | Currently executing |
| `success` | Completed successfully |
| `failed` | Execution failed |
| `up_for_retry` | Waiting for another attempt |
| `upstream_failed` | An upstream dependency failed |
| `skipped` | Intentionally skipped |

The exact set of states and their behavior can vary by Airflow version.

### Airflow Monitoring Example

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime

def process_orders():
    print("Processing orders")

with DAG(
    dag_id="orders_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False
) as dag:

    process_task = PythonOperator(
        task_id="process_orders",
        python_callable=process_orders
    )
```

The task execution status and logs can be inspected in the Airflow UI.

---

## 10. Pipeline Monitoring vs Data Quality Monitoring

These concepts overlap but have different purposes.

| Pipeline Monitoring | Data Quality Monitoring |
|---|---|
| Tracks pipeline execution | Tracks correctness and reliability of data |
| Checks task status | Checks data rules |
| Measures duration | Checks duplicates and NULLs |
| Monitors retries | Checks valid values |
| Detects job failures | Detects invalid or incomplete data |

### Example

A pipeline may complete successfully but produce incorrect data.

For example:

- Pipeline status: Succeeded.
- Expected records: 100,000.
- Actual records: 70,000.
- Duplicate order IDs: 5,000.

The pipeline executed successfully from a technical perspective, but the output may violate business expectations.

Therefore, production monitoring must include both execution health and data health.

---

## 11. Alerting and Incident Management

Monitoring becomes more useful when important conditions generate alerts.

### Example Alert Conditions

| Condition | Possible Alert |
|---|---|
| Pipeline fails | Pipeline failure alert |
| Duration exceeds threshold | Performance alert |
| Data freshness exceeds SLA | Freshness alert |
| Record count drops unexpectedly | Volume anomaly alert |
| Data quality check fails | Quality alert |
| Executor failures increase | Infrastructure alert |

### Incident Handling Process

```text
Issue Detected
      |
      v
Alert Generated
      |
      v
Engineer Notified
      |
      v
Issue Acknowledged
      |
      v
Investigation
      |
      v
Root Cause Identified
      |
      v
Recovery and Validation
      |
      v
Incident Closed
```

An alert should include:
- Pipeline name.
- Run ID.
- Environment.
- Failure time.
- Error details.
- Severity.
- Business impact.
- Owner or support team.

---

## 12. SLA, SLO, and SLI

These concepts help define and measure expected service performance.

### 12.1 SLA — Service Level Agreement

An SLA is an agreement that defines the service expectations between a provider and a customer or business stakeholder.

Example:

The daily sales dataset must be available by 7:00 AM on 99% of business days.

### 12.2 SLO — Service Level Objective

An SLO is a measurable target for service performance.

Example:

The daily orders pipeline should complete before 6:45 AM on at least 99% of scheduled runs over a defined measurement period.

### 12.3 SLI — Service Level Indicator

An SLI is the actual measurement used to evaluate service performance.

Example:

Percentage of scheduled pipeline runs completed before 6:45 AM.

### SLA vs SLO vs SLI

| Concept | Meaning | Example |
|---|---|---|
| SLA | Agreement | Data available by 7:00 AM |
| SLO | Target | 99% on-time delivery |
| SLI | Measurement | Actual on-time delivery rate |

**Remember:**

- SLA = Agreement.
- SLO = Objective.
- SLI = Indicator or measurement.

---

## 13. Common Pipeline Monitoring Problems

### 13.1 Pipeline Failure

Possible causes:
- Invalid input data.
- Source unavailable.
- Permission issue.
- Code error.
- Resource failure.

### 13.2 Pipeline Running Slowly

Possible causes:
- Increased data volume.
- Data skew.
- Inefficient joins.
- Excessive shuffling.
- Insufficient resources.
- Slow source or target systems.

### 13.3 Missing Data

Possible causes:
- Source extraction failure.
- Incorrect filters.
- Incremental watermark issue.
- Late-arriving data.
- Failed writes.

### 13.4 Duplicate Data

Possible causes:
- Pipeline rerun without idempotency.
- Duplicate source records.
- Incorrect MERGE condition.
- Missing deduplication logic.

### 13.5 Stale Data

Possible causes:
- Pipeline scheduling failure.
- Upstream dependency delay.
- Failed ingestion.
- Incorrect freshness calculation.

---

## 14. Pipeline Monitoring Best Practices

1. Monitor every critical production pipeline.
2. Track both pipeline-level and task-level execution.
3. Record start time, end time, and duration.
4. Monitor input and output record counts.
5. Track data freshness.
6. Add data quality checks.
7. Configure meaningful alert thresholds.
8. Keep logs structured and searchable.
9. Use consistent run identifiers.
10. Track retries and repeated failures.
11. Define ownership for each pipeline.
12. Maintain dashboards for critical workflows.
13. Avoid excessive or duplicate alerts.
14. Protect sensitive information in logs.
15. Review performance trends regularly.
16. Test recovery and rerun procedures.
17. Ensure reruns are idempotent.
18. Document common failure scenarios and runbooks.

---

## 15. Practical Interview Scenario

**Question: A daily sales pipeline is expected to complete by 7:00 AM, but it has not completed by 7:30 AM. How would you investigate it?**

### Answer

I would follow a structured monitoring and troubleshooting process.

**Step 1: Check Pipeline Status**

Inspect the orchestration platform to determine whether the pipeline is running, failed, queued, or blocked by dependencies.

**Step 2: Check Execution Duration**

Compare the current execution time with historical runs to identify unusual delays.

**Step 3: Inspect Task-Level Details**

Identify which activity or transformation is taking longer than expected.

**Step 4: Check Logs**

Inspect error messages, retries, source connectivity, permissions, and application logs.

**Step 5: Check Metrics**

Review record counts, throughput, CPU, memory, shuffle, and executor failures.

**Step 6: Investigate Data Health**

Check whether source data is available and whether the expected records are being processed.

**Step 7: Send Alerts and Escalate**

Notify the responsible team if the pipeline misses its expected delivery time. Escalate based on severity and the defined support policy.

**Step 8: Recover and Validate**

Resolve the root cause, rerun safely if necessary, and validate the final data before making it available to reporting systems.

---

## 16. Common Interview Questions and Answers

### Q1. What is pipeline monitoring?

Pipeline monitoring is the process of tracking pipeline execution, performance, reliability, and data health.

### Q2. Why is pipeline monitoring important?

It helps detect failures, identify performance problems, maintain data freshness, and ensure reliable data delivery.

### Q3. What are the important pipeline monitoring metrics?

Execution status, duration, success rate, failure rate, throughput, record count, data freshness, retries, and resource utilization.

### Q4. What is the difference between monitoring and observability?

Monitoring checks predefined conditions, while observability helps investigate system behavior using telemetry such as logs, metrics, and traces.

### Q5. What is data freshness?

It measures how recently data was updated relative to the expected delivery time.

### Q6. Can a pipeline succeed but still produce incorrect data?

Yes. Technical execution success does not guarantee data correctness. Data quality checks are also required.

### Q7. How do you monitor ADF pipelines?

Use ADF Monitor for pipeline and activity runs, and diagnostic settings with Log Analytics and Azure Monitor for centralized monitoring and alerts.

### Q8. How do you monitor Databricks jobs?

Use job run details, task status, Spark UI, driver and executor logs, and supported monitoring integrations.

### Q9. How do you monitor Airflow DAGs?

Use the Airflow UI, DAG and task states, execution logs, task duration, retries, and scheduler/worker monitoring.

### Q10. What is throughput?

Throughput measures the amount of data processed per unit of time.

### Q11. What is the difference between success rate and failure rate?

Success rate measures successful runs as a percentage of total runs. Failure rate measures failed runs as a percentage of total runs.

### Q12. What is an SLA?

An SLA is an agreement defining expected service levels.

### Q13. What is an SLO?

An SLO is a measurable target for service performance.

### Q14. What is an SLI?

An SLI is the measurement used to evaluate service performance against an objective.

### Q15. How do you prevent downstream pipelines from using incomplete data?

Use task dependencies, completion checks, data quality validation, and controlled publishing of the final dataset.

### Q16. Why is idempotency important for pipeline monitoring and recovery?

It allows a pipeline to be rerun safely without creating unintended duplicate or inconsistent results, assuming the operation is designed correctly.

---

## 17. Quick Revision

| Concept | Meaning |
|---|---|
| Pipeline Monitoring | Tracking execution and data health |
| Execution Status | Current or final run state |
| Duration | Time taken to complete |
| Data Volume | Number of records processed |
| Data Freshness | How current the data is |
| Throughput | Records processed per unit of time |
| Success Rate | Percentage of successful runs |
| Failure Rate | Percentage of failed runs |
| Retry Count | Number of retry attempts |
| SLA | Service Level Agreement |
| SLO | Service Level Objective |
| SLI | Service Level Indicator |
| ADF Monitor | ADF pipeline and activity monitoring |
| Spark UI | Spark execution diagnostics |
| Airflow UI | DAG and task monitoring |

### Remember These Five Points

1. Pipeline monitoring tracks execution, performance, and data health.
2. Monitor both pipeline status and the quality of its output.
3. Logs help investigate failures; metrics help identify trends.
4. Azure Monitor, ADF Monitor, Spark UI, and Airflow UI provide platform-specific monitoring.
5. Production monitoring should include alerting, recovery, validation, and ownership.

---

## 18. Self-Assessment

Try answering these questions without looking at the notes.

1. What is pipeline monitoring?
2. Why is monitoring important in data engineering?
3. What are the main areas of pipeline monitoring?
4. What is pipeline duration?
5. How do you calculate throughput?
6. How do you calculate success and failure rates?
7. What is data freshness?
8. What is the difference between pipeline monitoring and data quality monitoring?
9. How do you monitor an ADF pipeline?
10. What information does Spark UI provide?
11. How do you monitor an Airflow DAG?
12. What are SLA, SLO, and SLI?
13. Can a successful pipeline produce incorrect data?
14. How would you investigate a slow pipeline?
15. How would you handle a pipeline that misses its delivery deadline?
16. Why is idempotency important during recovery?
17. How would you prevent downstream jobs from using incomplete data?
18. Design a monitoring strategy for a daily production data pipeline.

**Interview readiness:** Be able to explain pipeline monitoring, important metrics, platform-specific monitoring, SLA/SLO/SLI, and how you would troubleshoot a production pipeline that is delayed or failing.

