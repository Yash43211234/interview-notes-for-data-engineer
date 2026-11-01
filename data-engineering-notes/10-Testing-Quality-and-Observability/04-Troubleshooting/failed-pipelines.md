

# Failed Data Pipelines: Troubleshooting and Recovery

## 1. Introduction

A failed data pipeline is a pipeline that does not complete its intended processing successfully.

Pipeline failures can occur during data ingestion, transformation, validation, or loading.

In production data engineering, identifying the failure is only the first step. Engineers must also find the root cause, recover safely, validate the output, and prevent the issue from happening again.

### Example

Consider a daily orders pipeline:

```text
Source Database
      |
      v
Data Ingestion
      |
      v
Raw Layer
      |
      v
PySpark Transformation
      |
      v
Silver Delta Table
      |
      v
Gold Aggregation
      |
      v
Reporting
```

Suppose the PySpark transformation fails because the source contains an unexpected data type.

As a result:
- The Silver table is not updated.
- The Gold aggregation cannot run.
- The daily sales report may become stale.

The data engineer must identify the failed task, investigate the error, fix the issue, and safely rerun the pipeline.

---

## 2. Common Pipeline Failure Types

### 2.1 Source System Failure

The source system is unavailable or cannot provide data.

Examples:
- Database server is down.
- API is unavailable.
- Network connection fails.
- Source file has not arrived.

### 2.2 Data-Related Failure

The input data does not match the expected structure or business rules.

Examples:
- Required columns are missing.
- Unexpected NULL values.
- Invalid date formats.
- Duplicate primary keys.
- Incorrect data types.

### 2.3 Transformation Failure

The transformation logic fails during processing.

Examples:
- Invalid SQL syntax.
- Incorrect PySpark expressions.
- Division by zero.
- Unsupported data conversion.
- Memory-related failures.

### 2.4 Authentication and Permission Failure

The pipeline cannot access a required resource.

Examples:
- Expired credentials.
- Missing Azure RBAC permissions.
- Incorrect ADLS Gen2 ACLs.
- Invalid service principal configuration.
- Missing Databricks permissions.

### 2.5 Infrastructure Failure

The execution environment cannot complete the workload.

Examples:
- Driver out-of-memory error.
- Executor failures.
- Insufficient disk space.
- Compute resource unavailable.
- Network interruption.

### 2.6 Orchestration Failure

The orchestration system cannot schedule or execute the required tasks.

Examples:
- Airflow scheduler issue.
- ADF integration runtime unavailable.
- Incorrect task dependencies.
- Trigger configuration problem.
- Databricks job configuration error.

---

## 3. Common Causes of Pipeline Failures

| Category | Possible Cause |
|---|---|
| Source | Database unavailable |
| Network | Connection timeout |
| Data | Invalid input records |
| Schema | Unexpected schema change |
| Code | Transformation logic error |
| Authentication | Expired credentials |
| Authorization | Missing permissions |
| Compute | Out-of-memory error |
| Storage | Insufficient storage or write failure |
| Orchestration | Dependency or scheduling failure |

A good troubleshooting process should identify the actual cause rather than repeatedly restarting the pipeline.

---

## 4. Step-by-Step Troubleshooting Process

Follow a structured process whenever a production pipeline fails.

### Step 1: Identify the Failed Pipeline

Check the monitoring platform or orchestration tool.

Identify:
- Pipeline name.
- Run ID.
- Failed task.
- Execution status.
- Failure timestamp.
- Environment.

Example:

```text
Pipeline: orders_pipeline
Run ID: run_20261025_001
Task: transform_orders
Status: FAILED
Environment: PROD
```

### Step 2: Inspect the Error Message

Read the complete error message and relevant stack trace.

Look for:
- Exception type.
- Failed operation.
- Source or target location.
- SQL or transformation error.
- Authentication details.
- Timeout information.

Do not rely only on a short failure summary.

### Step 3: Check Logs

Inspect:
- Pipeline logs.
- Task logs.
- Application logs.
- Driver logs.
- Executor logs.
- Source-system logs.

The logs can help identify where execution stopped and what happened immediately before failure.

### Step 4: Check Dependencies

Determine whether the failed task depends on another task.

Example:

```text
Extract Customers
       |
       v
Extract Orders
       |
       v
Transform Orders
       |
       v
Load Gold Table
```

If customer extraction fails, downstream tasks may not execute.

### Step 5: Check Data and Schema

Validate:
- Input file availability.
- Column names.
- Data types.
- NULL values.
- Duplicate records.
- Unexpected record volume.
- Schema changes.

### Step 6: Check Infrastructure

Inspect:
- CPU and memory usage.
- Driver and executor status.
- Disk usage.
- Network connectivity.
- Compute availability.
- Storage permissions.

### Step 7: Identify the Root Cause

Determine the actual reason for the failure.

For example:

```text
Failure:
Cannot cast STRING to DATE

Root Cause:
Source system introduced a new date format.

Impact:
Orders transformation failed.

Resolution:
Validate and normalize the incoming date format.
```

### Step 8: Fix and Recover

Apply the appropriate fix and rerun the affected pipeline or task.

Before rerunning, check whether earlier tasks already wrote data to the target.

### Step 9: Validate the Output

After recovery, verify:
- Record counts.
- Data quality.
- Duplicate records.
- Source-to-target reconciliation.
- Data freshness.
- Downstream availability.

### Step 10: Document the Incident

Record:
- Root cause.
- Business impact.
- Resolution.
- Recovery time.
- Preventive action.

---

## 5. Common Errors and Their Solutions

| Error | Possible Cause | Troubleshooting |
|---|---|---|
| Connection timeout | Network or source issue | Check connectivity and source availability |
| Permission denied | Missing authorization | Verify identity, RBAC, and ACLs |
| File not found | Incorrect path or missing file | Verify source location and arrival |
| Schema mismatch | Unexpected column or type | Inspect schema and apply controlled evolution |
| Out of memory | Large workload or inefficient processing | Review Spark UI, partitions, and memory usage |
| Duplicate records | Retry or ingestion logic issue | Check idempotency and deduplication |
| Job timeout | Slow processing or blocked dependency | Inspect task duration and bottlenecks |
| Invalid SQL | Query or transformation error | Validate query syntax and column references |
| Executor lost | Infrastructure or resource problem | Inspect executor logs and cluster events |
| Delta concurrent modification | Conflicting writes | Investigate overlapping transactions and retry safely |

These are common causes, not guaranteed explanations. Always confirm the root cause using logs and execution details.

---

## 6. Troubleshooting in Azure Data Factory

Azure Data Factory provides pipeline-level and activity-level execution details.

### What to Check

1. Open the ADF Monitor section.
2. Find the failed pipeline run.
3. Open the pipeline run details.
4. Identify the failed activity.
5. Inspect the activity error message.
6. Check activity input and output details.
7. Verify linked services and datasets.
8. Check Integration Runtime availability.
9. Verify authentication and permissions.
10. Rerun after fixing the issue.

### Common ADF Failure Scenarios

| Problem | What to Check |
|---|---|
| Copy Activity failure | Source, sink, mapping, network |
| Linked Service failure | Connection and authentication |
| File not found | Container, directory, file path |
| Authentication failure | Managed identity or service principal |
| Integration Runtime error | IR status and connectivity |
| Databricks activity failure | Notebook/job error and parameters |
| Timeout | Activity duration and source performance |

### ADF Failure Flow

```text
ADF Pipeline Failure
        |
        v
Open Monitor
        |
        v
Find Failed Activity
        |
        v
Inspect Error Details
        |
        v
Check Source / Sink / IR
        |
        v
Fix the Issue
        |
        v
Rerun and Validate
```

### Example: Handling an Activity Failure

An ADF pipeline contains:

```text
Lookup Watermark
       |
       v
Copy New Orders
       |
       v
Update Watermark
```

Suppose Copy Activity fails.

The watermark update must not execute as if the load succeeded.

The pipeline should update the watermark only after the target write has completed successfully.

This prevents the pipeline from advancing its extraction position before the data is safely loaded.

---

## 7. Troubleshooting in Databricks and PySpark

Databricks pipelines can fail during notebook execution, Spark transformations, job execution, or storage operations.

### What to Check

- Job run details.
- Failed task.
- Notebook output.
- Driver logs.
- Executor logs.
- Spark UI.
- Input schema.
- Storage access.
- Compute configuration.

### Common Spark Errors

**1. Out-of-Memory Error**

Possible causes:
- Too much data collected on the driver.
- Large broadcast table.
- Excessive caching.
- Data skew.
- Insufficient memory.

Possible solutions:
- Avoid unnecessary `collect()`.
- Review join strategy.
- Reduce unnecessary caching.
- Handle skewed keys.
- Review partitioning and resource allocation.

**2. Data Skew**

Data skew occurs when some partitions contain significantly more data than others.

Symptoms:
- A few tasks run much longer.
- Most tasks finish quickly.
- Some tasks spill heavily.
- Stage completion is delayed.

Possible solutions:
- Review partition distribution.
- Use broadcast joins where appropriate.
- Consider salting for suitable skewed joins.
- Reassess partitioning strategy.

**3. Schema Mismatch**

Possible causes:
- Source column type changed.
- Required column is missing.
- Unexpected nested structure.

Possible solutions:
- Inspect the incoming schema.
- Validate required columns.
- Use controlled schema evolution.
- Quarantine invalid records where appropriate.

### Example: Detect Invalid Records

```python
from pyspark.sql import functions as F

valid_orders = df.filter(
    F.col("order_id").isNotNull() &
    F.col("amount").isNotNull() &
    (F.col("amount") >= 0)
)

invalid_orders = df.filter(
    F.col("order_id").isNull() |
    F.col("amount").isNull() |
    (F.col("amount") < 0)
)
```

This separates records that satisfy the example validation rules from those that do not.

The pipeline should define whether invalid records are rejected, quarantined, or cause the batch to fail.

---

## 8. Troubleshooting in Apache Airflow

Airflow provides DAG-level and task-level execution information.

### What to Check

- DAG run status.
- Failed task instance.
- Task logs.
- Upstream dependencies.
- Retry attempts.
- Scheduler status.
- Worker status.
- Connections and variables.

### Common Airflow Task States

| State | Meaning |
|---|---|
| `queued` | Waiting for execution |
| `running` | Currently executing |
| `success` | Completed successfully |
| `failed` | Execution failed |
| `up_for_retry` | Waiting for another attempt |
| `upstream_failed` | An upstream dependency failed |
| `skipped` | Task was skipped |

### Airflow Troubleshooting Steps

1. Open the DAG in the Airflow UI.
2. Identify the failed DAG run.
3. Find the failed task.
4. Open the task logs.
5. Check the exception and traceback.
6. Inspect upstream dependencies.
7. Verify connections and credentials.
8. Fix the issue.
9. Clear or rerun the appropriate task.
10. Validate downstream results.

### Important

Clearing and rerunning a task can repeat its side effects.

Before clearing a production task, confirm that its writes are idempotent or otherwise safe to repeat.

---

## 9. Retry vs Rerun vs Backfill

These concepts are related but serve different purposes.

### Retry

A retry is another automatic or configured attempt after a task fails.

Example:

A database connection fails temporarily. Airflow retries the task after a delay.

### Rerun

A rerun is manually or operationally executing a pipeline or task again.

Example:

An engineer fixes an incorrect file path and reruns the failed pipeline.

### Backfill

A backfill runs a pipeline for historical data intervals that were not previously processed or need reprocessing.

Example:

A daily pipeline failed for three dates. The team processes those missing dates through a backfill.

| Feature | Retry | Rerun | Backfill |
|---|---|---|---|
| Purpose | Recover from a failed attempt | Execute again | Process historical intervals |
| Initiated by | Configured retry logic or platform | Engineer or workflow | Engineer or scheduler |
| Scope | Failed attempt/task | Selected run or task | Historical data intervals |
| Example | Retry after timeout | Restart after code fix | Reprocess last week's data |

A backfill is not simply a retry for a failed task. It is a historical processing operation.

---

## 10. Idempotency and Safe Pipeline Recovery

Idempotency is essential when recovering failed pipelines.

An operation is idempotent when repeating it produces the same intended final effect under defined conditions.

### Example: Duplicate Records Problem

Suppose a pipeline reads 1,000 orders and appends them to a target table.

The first attempt writes the data successfully but fails before reporting success.

When the pipeline is rerun, the same 1,000 records are appended again.

Target result: 2,000 rows, including duplicates.

### Better Approach: Use MERGE

```sql
MERGE INTO target_orders AS target
USING source_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
    UPDATE SET
        target.amount = source.amount,
        target.order_date = source.order_date

WHEN NOT MATCHED THEN
    INSERT (
        order_id,
        amount,
        order_date
    )
    VALUES (
        source.order_id,
        source.amount,
        source.order_date
    );
```

This example performs an upsert based on `order_id`.

**Important:**
- Deduplicate the source before MERGE.
- Ensure the matching key is correct.
- Define how updates and deletes should be handled.
- Ensure repeated execution produces the intended final result.

MERGE alone does not guarantee idempotency if the source is nondeterministic or contains conflicting duplicate keys.

### Other Recovery Techniques

- Use deterministic batch IDs.
- Track processed source files.
- Maintain watermarks carefully.
- Use staging tables.
- Validate target state before retrying.
- Use transactions where supported.

---

## 11. Partial Failure and Recovery

A pipeline can partially succeed.

Example:

```text
Extract Orders       SUCCESS
       |
       v
Load Raw Data        SUCCESS
       |
       v
Transform Orders     FAILED
       |
       v
Load Gold Table      NOT RUN
```

The Raw layer already contains the extracted data.

A recovery strategy should identify the last successfully completed stage and resume from the correct point.

### Recovery Options

**Option 1: Rerun the Entire Pipeline**

Use when the pipeline is safe to repeat and the workload is small.

**Option 2: Rerun Only the Failed Task**

Use when upstream outputs are valid and can be reused.

**Option 3: Reprocess from a Checkpoint**

Use when the pipeline has reliable progress tracking or checkpoints.

**Option 4: Reprocess from Source**

Use when intermediate data is incomplete or corrupted.

The correct option depends on data correctness, dependencies, and idempotency.

---

## 12. Root Cause Analysis (RCA)

Root Cause Analysis is the process of identifying the underlying reason for an incident rather than treating only its visible symptoms.

### Example

**Incident:** Daily orders pipeline failed.

**Observed Error:** Delta write operation failed.

**Investigation:**
- The source schema changed.
- A numeric field was delivered as a string.
- The transformation expected a numeric type.

**Root Cause:** An upstream schema change was not handled.

**Resolution:**
- Validate the incoming schema.
- Add controlled type conversion.
- Quarantine invalid records.
- Rerun the pipeline safely.

**Preventive Action:**
- Introduce schema validation.
- Add data contract checks.
- Alert on unexpected schema changes.

### Five Whys Technique

The Five Whys technique repeatedly asks why an issue occurred to reach its underlying cause.

Example:

1. Why did the report become stale?  
   The Gold table was not updated.

2. Why was the Gold table not updated?  
   The transformation task failed.

3. Why did the transformation fail?  
   An input column had an unexpected type.

4. Why was the type unexpected?  
   The source system changed its schema.

5. Why was the change not detected earlier?  
   No schema validation or change notification existed.

The preventive action is to improve schema-change detection and handling.

---

## 13. Production Incident Handling

A production pipeline failure should be managed as an incident when it affects business operations or agreed service levels.

### Incident Management Process

```text
Failure Detected
       |
       v
Alert Generated
       |
       v
Incident Acknowledged
       |
       v
Severity Classified
       |
       v
Root Cause Investigation
       |
       v
Recovery / Workaround
       |
       v
Data Validation
       |
       v
Business Confirmation
       |
       v
Incident Closure
```

### Important Incident Information

| Field | Description |
|---|---|
| Incident ID | Unique incident reference |
| Pipeline | Affected pipeline |
| Run ID | Failed execution |
| Severity | Business impact |
| Start time | When the issue began |
| Detection time | When the issue was identified |
| Resolution time | When service was restored |
| Root cause | Underlying problem |
| Recovery action | Steps taken to restore service |
| Preventive action | Steps to reduce recurrence |

### Mean Time Metrics

**MTTD — Mean Time to Detect**

Average time taken to detect an incident.

**MTTA — Mean Time to Acknowledge**

Average time taken for a responsible person to acknowledge an incident.

**MTTR — Mean Time to Recover or Repair**

Average time taken to restore service or resolve an incident. The precise definition should be consistent within the organization.

These metrics help teams evaluate incident response effectiveness.

---

## 14. Troubleshooting Best Practices

1. Start with the failed pipeline and task.
2. Inspect complete error messages and logs.
3. Check upstream dependencies.
4. Verify source availability and input data.
5. Validate schema and data types.
6. Check authentication and permissions.
7. Inspect compute and infrastructure metrics.
8. Identify the root cause before rerunning.
9. Confirm whether partial writes occurred.
10. Use idempotent recovery logic.
11. Validate output after recovery.
12. Communicate business impact and recovery status.
13. Document the root cause.
14. Add preventive checks where possible.
15. Avoid repeatedly retrying permanent errors.

### When Should You Retry?

Retries are useful for temporary failures such as:
- Short network interruptions.
- Temporary service unavailability.
- Transient resource contention.
- Rate limiting, when appropriate backoff is used.

Retries are usually not useful for permanent failures such as:
- Invalid SQL syntax.
- Missing required columns.
- Incorrect credentials.
- Invalid business logic.

Fix permanent errors before retrying.

---

## 15. Practical Interview Scenario

**Question: A production PySpark pipeline failed after successfully writing data to the Silver Delta table. What would you do?**

### Answer

I would first identify the failed task and inspect its logs to understand the reason for failure.

Then I would determine whether the Silver write was committed successfully and whether downstream tasks started.

My recovery process would be:

1. Check the pipeline run ID and failed task.
2. Inspect driver and executor logs.
3. Confirm the Silver table's committed state.
4. Identify the root cause.
5. Fix the transformation or infrastructure issue.
6. Check whether rerunning could create duplicates.
7. Use idempotent writes or MERGE logic where appropriate.
8. Rerun the failed task or required downstream stages.
9. Validate record counts, duplicates, and data quality.
10. Confirm the Gold table is updated and downstream reporting is available.

I would also document the incident and add preventive checks if the failure could happen again.

---

## 16. Common Interview Questions and Answers

### Q1. What is a failed data pipeline?

A pipeline that does not complete its intended processing successfully.

### Q2. What are common causes of pipeline failure?

Source unavailability, invalid data, schema changes, transformation errors, permission problems, resource failures, and orchestration issues.

### Q3. How do you troubleshoot a failed pipeline?

Check the failed run, inspect logs, examine dependencies, validate input data, check infrastructure, identify the root cause, and recover safely.

### Q4. What is the difference between retry and rerun?

A retry is another configured attempt after failure. A rerun is an explicit execution of a pipeline or task again.

### Q5. What is a backfill?

Backfill is the processing or reprocessing of historical data intervals.

### Q6. What is idempotency?

Idempotency means repeating an operation produces the same intended final effect under defined conditions.

### Q7. Why is idempotency important?

It prevents retries and reruns from producing unintended duplicate or inconsistent results.

### Q8. What is a partial pipeline failure?

A partial failure occurs when some tasks succeed but a later task fails.

### Q9. How do you recover from a partial failure?

Identify the last successful stage, verify its output, fix the issue, and rerun from the appropriate point.

### Q10. What is root cause analysis?

RCA identifies the underlying reason for an incident and helps prevent recurrence.

### Q11. When should you avoid retrying a task?

Avoid repeated retries for permanent failures such as invalid SQL, missing columns, or incorrect permissions.

### Q12. How do you troubleshoot a Databricks out-of-memory error?

Inspect Spark UI and logs, review data volume and partitioning, avoid unnecessary driver collection and caching, and investigate skew and join strategies.

### Q13. How do you troubleshoot an ADF permission failure?

Verify the identity used by the linked service, required RBAC roles, ADLS Gen2 ACLs, and access scope.

### Q14. What is the difference between `failed` and `upstream_failed` in Airflow?

`failed` means the task itself failed during execution. `upstream_failed` means an upstream dependency failed and prevented the task from running successfully.

### Q15. What is MTTD?

Mean Time to Detect measures the average time taken to detect incidents.

### Q16. What is MTTR?

Mean Time to Recover or Repair measures the average time taken to restore service or resolve an incident, based on the organization's definition.

### Q17. What should you validate after a pipeline rerun?

Record counts, duplicates, data quality, source-to-target reconciliation, data freshness, and downstream availability.

### Q18. How do you prevent recurring pipeline failures?

Use schema validation, data quality checks, monitoring, alerting, automated testing, idempotent processing, and root cause analysis.

---

## 17. Quick Revision

| Concept | Meaning |
|---|---|
| Pipeline Failure | Pipeline does not complete successfully |
| Retry | Another configured execution attempt |
| Rerun | Explicitly executing a pipeline or task again |
| Backfill | Processing historical data intervals |
| Partial Failure | Some tasks succeed while others fail |
| Idempotency | Safe repeated execution under defined conditions |
| RCA | Root Cause Analysis |
| Recovery | Restoring the pipeline to a correct operating state |
| MTTD | Mean Time to Detect |
| MTTA | Mean Time to Acknowledge |
| MTTR | Mean Time to Recover or Repair |

### Remember These Five Points

1. Always inspect logs and identify the failed task before taking action.
2. Find the root cause instead of repeatedly retrying.
3. Check for partial writes before rerunning.
4. Use idempotent processing to support safe recovery.
5. Validate the final output and document the incident.

---

## 18. Self-Assessment

Try answering these questions without looking at the notes.

1. What is a failed data pipeline?
2. What are the common causes of pipeline failure?
3. Explain the steps you follow to troubleshoot a production failure.
4. How do you investigate a source connection failure?
5. How do you troubleshoot a schema mismatch?
6. How do you troubleshoot a Databricks out-of-memory error?
7. How do you investigate an ADF permission failure?
8. How do you debug a failed Airflow task?
9. What is the difference between retry, rerun, and backfill?
10. What is idempotency?
11. Why is idempotency important during recovery?
12. What is a partial pipeline failure?
13. How do you recover from a partial failure?
14. What is Root Cause Analysis?
15. Explain the Five Whys technique.
16. What are MTTD, MTTA, and MTTR?
17. What should you validate after rerunning a pipeline?
18. Design a recovery strategy for a pipeline that fails after writing to the Silver layer.

**Interview readiness:** Be able to explain the troubleshooting sequence, common platform failures, retry vs rerun vs backfill, idempotent recovery, and root cause analysis using a realistic production pipeline scenario.

