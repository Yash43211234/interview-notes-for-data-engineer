# Azure Data Factory: Monitoring and Error Handling

## 1. Introduction

Monitoring and error handling are important parts of building reliable data pipelines in Azure Data Factory (ADF).

A pipeline may fail because of:

- Source or destination connectivity problems.
- Authentication or permission issues.
- Incorrect SQL queries.
- Schema mismatches.
- Network interruptions.
- Data quality problems.
- Timeout or resource limitations.

Monitoring helps us understand what happened during execution. Error handling helps us respond to failures and recover safely.

### Key Objectives

1. Track pipeline and activity execution.
2. Identify failed activities.
3. Understand error messages.
4. Retry operations when appropriate.
5. Execute alternative workflows after failures.
6. Log errors and notify responsible teams.
7. Prevent duplicate or inconsistent data during recovery.

---

## 2. Monitoring in Azure Data Factory

ADF provides monitoring capabilities through Azure Data Factory Studio and Azure Monitor.

### What Can We Monitor?

- Pipeline runs.
- Activity runs.
- Trigger runs.
- Integration Runtime status.
- Copy Activity performance.
- Execution duration.
- Failed and successful activities.
- Error messages and failure details.

### Monitoring Flow

```text
ADF Pipeline Execution
          |
          v
     Activity Runs
          |
          v
   Execution Status
          |
          v
   ADF Monitor View
          |
          v
  Investigate Failures
          |
          v
  Fix and Rerun Pipeline
```

---

## 3. Pipeline Run vs Activity Run

This is an important interview topic.

### Pipeline Run

A pipeline run represents one execution of a pipeline.

It has its own:

- Pipeline run ID.
- Start time.
- End time.
- Execution status.
- Parameters.

Example:

```text
Pipeline: PL_Load_Customers
Run ID: 12345
Status: Failed
Duration: 5 minutes
```

### Activity Run

An activity run represents the execution of an individual activity inside a pipeline.

Example:

```text
Pipeline: PL_Load_Customers

Activity 1: Lookup_Watermark
Status: Succeeded

Activity 2: Copy_Customers
Status: Failed

Activity 3: Update_Watermark
Status: Skipped
```

### Difference

| Pipeline Run | Activity Run |
|---|---|
| Represents the entire pipeline execution | Represents one activity execution |
| Contains overall execution status | Contains activity-specific status |
| Has a pipeline run ID | Has an activity run ID |
| Can contain multiple activity runs | Belongs to a pipeline run |

**Interview point:** One pipeline run can contain multiple activity runs.

---

## 4. Monitoring Pipeline Runs

### Steps

1. Open Azure Data Factory Studio.
2. Select **Monitor**.
3. Open Pipeline Runs.
4. Select the required time range.
5. Find the pipeline execution.
6. Check its status.
7. Open the activity details to investigate further.

### Common Pipeline Statuses

| Status | Meaning |
|---|---|
| Queued | Waiting to execute |
| In Progress | Currently executing |
| Succeeded | Completed successfully |
| Failed | Execution failed |
| Cancelled | Execution was cancelled |

The exact status values displayed can vary by monitoring view and activity type.

### What Should We Check?

- Pipeline name.
- Run ID.
- Start and end time.
- Duration.
- Parameters.
- Error details.
- Activity execution status.

---

## 5. Monitoring Activity Runs

Activity monitoring helps identify exactly which operation failed.

### Example

Suppose a pipeline has three activities:

```text
Lookup Activity
      |
      v
Copy Activity
      |
      v
Stored Procedure Activity
```

Execution result:

| Activity | Status |
|---|---|
| Lookup | Succeeded |
| Copy | Failed |
| Stored Procedure | Skipped |

The Copy Activity is the failure point.

### Useful Activity Details

For a failed activity, inspect:

- Error code.
- Error message.
- Activity type.
- Input and output details.
- Start and end time.
- Duration.
- Retry information, where available.

**Important:** Activity input and output can contain sensitive information. Access to monitoring details should follow organizational security policies.

---

## 6. Copy Activity Monitoring

Copy Activity provides additional performance and data movement details.

Useful metrics include:

- Data read.
- Data written.
- Rows read.
- Rows copied.
- Copy duration.
- Throughput.
- Source and sink details.
- Performance bottlenecks.

### Example

Suppose you copy 1 million records from Azure SQL Database to ADLS Gen2.

```text
Source: Azure SQL Database
Sink: ADLS Gen2

Rows Read: 1,000,000
Rows Written: 1,000,000
Status: Succeeded
```

If the pipeline takes too long, investigate:

1. Source database performance.
2. Destination storage performance.
3. Network bandwidth and latency.
4. Integration Runtime capacity.
5. Parallel copy configuration.
6. Connector limitations.

### Important

A successful Copy Activity does not automatically prove that the business data is correct. Row counts, data quality, and reconciliation checks may still be required.

---

## 7. Common ADF Errors

| Error Category | Possible Cause |
|---|---|
| Authentication failure | Incorrect credentials or missing permissions |
| Connectivity failure | Firewall, DNS, network, or private endpoint issue |
| Source error | Database unavailable or query failure |
| Sink error | Destination unavailable or write permission missing |
| Schema mismatch | Source and destination structures differ |
| Timeout | Operation exceeded the configured timeout |
| Throttling | Source or destination limited requests |
| Integration Runtime error | Runtime unavailable or incorrectly configured |
| Data conversion error | Invalid or incompatible data types |

### Example: Authentication Failure

Possible causes:

- Incorrect username or password.
- Managed identity lacks required permissions.
- Incorrect service principal configuration.
- Expired or invalid credentials.

### Example: Connectivity Failure

Possible causes:

- Incorrect server or endpoint.
- Firewall blocking access.
- DNS resolution failure.
- Private endpoint configuration problem.
- Self-hosted IR cannot reach the source.

### Example: Schema Mismatch

Suppose the source contains:

```text
customer_id
customer_name
age
```

But the destination expects:

```text
customer_id
customer_name
```

Depending on the mapping and connector configuration, the extra source column may cause a mapping or schema problem.

Validate source and sink schemas and configure explicit mappings where appropriate.

---

## 8. Error Handling in ADF

ADF allows activities to follow different execution paths depending on the outcome of previous activities.

Common dependency conditions:

- Succeeded.
- Failed.
- Skipped.
- Completed.

### Example: Success and Failure Paths

```text
             Copy Activity
               /       \
              /         \
       Succeeded       Failed
           |              |
           v              v
    Success Activity  Error Handler
```

### Succeeded

The next activity executes when the previous activity succeeds.

### Failed

The next activity executes when the previous activity fails.

### Skipped

The next activity executes when the previous activity is skipped.

### Completed

The next activity executes after the previous activity finishes, regardless of whether it succeeded or failed.

**Important:** A Completed dependency does not mean the previous activity succeeded. Always inspect the actual activity status when deciding what to do next.

---

## 9. Retry Configuration

Retry is useful for temporary failures.

Examples:

- Temporary network interruption.
- Service throttling.
- Temporary database unavailability.
- Transient service errors.

ADF activities that support retry configuration allow you to specify:

- Retry count.
- Retry interval.

### Example

```text
Activity: Copy_Customers

Retry Count: 3
Retry Interval: 30 seconds
```

If the activity fails because of a transient problem, ADF can attempt it again according to the configured settings.

### When Should We Use Retry?

Use retry for failures that may resolve without changing the pipeline configuration or source data.

Examples:

- Temporary connectivity problems.
- Transient service errors.
- Temporary throttling.

### When Should We Avoid Blind Retry?

Do not repeatedly retry failures such as:

- Invalid SQL syntax.
- Missing columns.
- Incorrect credentials.
- Missing permissions.
- Invalid data conversion.
- Incorrect configuration.

These usually require a correction before execution can succeed.

---

## 10. Timeout Configuration

Timeout defines how long an activity is allowed to execute before it is treated as timed out.

### Example

```text
Activity: Copy_Large_Table

Timeout: 1 hour
```

If the activity does not complete within the configured limit, it can fail due to timeout.

### Retry vs Timeout

| Retry | Timeout |
|---|---|
| Controls how many retry attempts are allowed | Controls the maximum allowed execution duration |
| Helps with transient failures | Prevents an activity from running indefinitely |
| Can cause another execution attempt | Can cause an activity to fail when the limit is reached |

**Important:** Timeout and retry are separate settings. A retry does not automatically fix a timeout caused by insufficient capacity or a consistently slow source.

---

## 11. Using If Condition for Error Handling

If Condition Activity allows a pipeline to execute different branches based on a Boolean expression.

### Example

Suppose a Lookup Activity returns the number of records to process.

If records exist, run Copy Activity. Otherwise, log that no data was found.

```text
       Lookup Activity
              |
              v
        If Condition
          /      \
        True     False
         |         |
         v         v
       Copy      Log No
      Activity    Data
```

Example expression:

```text
@greater(activity('Lookup_Count').output.firstRow.row_count, 0)
```

This assumes the Lookup activity has **First row only** enabled and returns a `row_count` field.

If First row only is disabled, the output structure is different and the expression must be adjusted.

---

## 12. Logging Errors

For production pipelines, error information should be captured in a central location.

A common approach is to use a database logging table.

### Example Error Log Table

```sql
CREATE TABLE dbo.PipelineErrorLog (
    error_id BIGINT IDENTITY(1,1) PRIMARY KEY,
    pipeline_name VARCHAR(200),
    pipeline_run_id VARCHAR(100),
    activity_name VARCHAR(200),
    error_code VARCHAR(100),
    error_message VARCHAR(MAX),
    failure_time DATETIME2
);
```

### Example Logging Workflow

```text
             Main Pipeline
                  |
                  v
             Copy Activity
              /       \
             /         \
       Succeeded      Failed
           |             |
           v             v
       Continue       Error Path
                         |
                         v
                  Capture Error Details
                         |
                         v
                  Execute Logging Step
                         |
                         v
                   Error Log Table
```

### Useful ADF Expressions

Pipeline name:

```text
@pipeline().Pipeline
```

Pipeline run ID:

```text
@pipeline().RunId
```

Activity error code:

```text
@activity('Copy_Customers').error.errorCode
```

Activity error message:

```text
@activity('Copy_Customers').error.message
```

These activity error references are useful in a failure path that depends on the failed activity. Confirm the activity name and output structure when building the expression.

### Best Practice

Capture enough information to investigate a failure:

- Pipeline name.
- Pipeline run ID.
- Activity name.
- Error code.
- Error message.
- Failure timestamp.
- Source and destination identifiers, where safe.

Avoid logging passwords, access tokens, connection strings, or sensitive personal data.

---

## 13. Notifications and Alerts

A production pipeline should notify the responsible team when an important workflow fails.

ADF can be integrated with Azure Monitor and other notification mechanisms.

A common approach is:

1. Configure diagnostic settings for supported ADF logs and metrics.
2. Send the selected data to a Log Analytics workspace or another supported destination.
3. Create Azure Monitor alert rules.
4. Configure an action group.
5. Send notifications through supported channels.

For pipeline-specific notifications, ADF can also call an Azure Function, Logic App, or other supported endpoint through an appropriate activity.

### Example

```text
       ADF Pipeline
             |
             v
        Copy Activity
             |
             v
           Failed
             |
             v
       Error Handling Path
             |
             v
       Azure Function /
        Logic App
             |
             v
      Email or Notification
```

### Important

An alert rule and an ADF failure path are different mechanisms:

- ADF failure path handles execution inside the pipeline.
- Azure Monitor alerting detects configured conditions and sends notifications.

---

## 14. Rerunning Failed Pipelines

ADF provides options to rerun pipeline executions.

Common approaches include:

- Rerun a failed pipeline run.
- Rerun from a failed activity where supported.
- Start a new pipeline run with corrected parameters.
- Build a recovery pipeline for specific failure scenarios.

### Example

Suppose the pipeline contains:

```text
Lookup Watermark
       |
       v
Copy Data
       |
       v
Validate Data
       |
       v
Update Watermark
```

If Copy Data fails, investigate the failure and rerun the appropriate workflow.

If the target write succeeded but the pipeline failed before updating the watermark, a rerun may process the same records again.

Therefore, target writes should be designed to handle replay safely.

### Important Recovery Principles

1. Identify the actual failure point.
2. Understand whether earlier activities changed data.
3. Fix the root cause.
4. Confirm whether rerunning can create duplicates.
5. Use idempotent writes or deduplication where appropriate.
6. Update watermarks only after the target operation succeeds and required validation passes.
7. Reconcile source and target when necessary.

---

## 15. Idempotency and Error Recovery

**Idempotency** means that repeating an operation has the same final effect as performing it once, under the defined conditions.

It is essential for reliable pipeline retries and recovery.

### Example: Non-idempotent Load

Suppose every pipeline run appends the same source records to a target table.

```text
First Run:  1,000 records
Retry:     1,000 duplicate records
```

The retry creates duplicates.

### Example: Idempotent Load

Use a stable business key and an appropriate upsert strategy.

For example, in a Delta table:

```sql
MERGE INTO target_customers AS target
USING source_customers AS source
ON target.customer_id = source.customer_id

WHEN MATCHED THEN
  UPDATE SET
    target.customer_name = source.customer_name,
    target.city = source.city

WHEN NOT MATCHED THEN
  INSERT (
    customer_id,
    customer_name,
    city
  )
  VALUES (
    source.customer_id,
    source.customer_name,
    source.city
  );
```

This example assumes the source has at most one relevant row per `customer_id`. Deduplicate the source before MERGE if multiple source rows can match the same target row.

### Why Idempotency Matters

- Safe retries.
- Recovery after partial failure.
- Duplicate prevention.
- Reliable incremental loading.
- Consistent target data.

**Interview point:** Retry handles another attempt. Idempotency makes repeated attempts safe.

---

## 16. Monitoring with Azure Monitor and Log Analytics

Azure Data Factory can send diagnostic information to supported monitoring destinations.

### Azure Monitor

Azure Monitor provides monitoring and alerting capabilities across Azure resources.

### Log Analytics

Log Analytics allows you to query collected logs using Kusto Query Language (KQL).

### Example Architecture

```text
       Azure Data Factory
                |
                v
       Diagnostic Settings
                |
                v
        Log Analytics
                |
                v
        KQL Queries
                |
                v
        Alert Rules
                |
                v
       Action Group
                |
                v
         Notifications
```

### Example KQL Query

The exact table and column names depend on the diagnostic mode and destination configuration.

For resource-specific ADF diagnostic tables, a query may look like:

```kusto
ADFActivityRun
| where TimeGenerated > ago(24h)
| where Status == "Failed"
| project TimeGenerated,
          PipelineName,
          ActivityName,
          Status,
          ErrorCode,
          ErrorMessage
| order by TimeGenerated desc
```

For resource-specific pipeline run logs:

```kusto
ADFPipelineRun
| where TimeGenerated > ago(24h)
| where Status == "Failed"
| project TimeGenerated,
          PipelineName,
          RunId,
          Status
| order by TimeGenerated desc
```

These are illustrative queries. Verify the actual table names and available columns in the Log Analytics workspace because diagnostic destination and schema configurations can differ.

---

## 17. Production Monitoring Best Practices

### Execution Monitoring

- Monitor pipeline and activity success rates.
- Track execution duration.
- Identify repeated failures.
- Monitor long-running activities.
- Review Integration Runtime availability.

### Data Monitoring

- Validate source and target row counts.
- Check duplicate records.
- Validate required columns.
- Monitor data freshness.
- Reconcile important business totals.

### Error Handling

- Configure retries for transient failures.
- Set appropriate timeouts.
- Use failure dependencies for error paths.
- Log meaningful error details.
- Send alerts for critical failures.
- Make retryable operations idempotent.

### Operational Practices

- Use meaningful pipeline and activity names.
- Keep logs structured and searchable.
- Avoid exposing secrets in logs.
- Define ownership for failed pipelines.
- Maintain recovery procedures.
- Test failure scenarios before production deployment.

---

## 18. Common Interview Questions

### Q1. How do you monitor an Azure Data Factory pipeline?

I use the Monitor section in ADF Studio to inspect pipeline runs, activity runs, execution status, duration, and error details. For centralized monitoring and alerts, I use Azure Monitor and diagnostic settings.

### Q2. What is the difference between a Pipeline Run and an Activity Run?

A Pipeline Run represents one execution of the entire pipeline. An Activity Run represents the execution of an individual activity inside that pipeline.

### Q3. How do you identify which activity failed?

I open the failed pipeline run in the Monitor view and inspect its activity runs to identify the failed activity and review its error details.

### Q4. What are the common causes of ADF pipeline failures?

Authentication issues, network problems, incorrect SQL queries, schema mismatches, missing permissions, source or sink unavailability, throttling, and timeouts.

### Q5. How do you handle temporary failures?

I configure appropriate retry settings for activities that can fail temporarily because of network interruptions, throttling, or transient service errors.

### Q6. What is the difference between retry and timeout?

Retry controls how many additional attempts are allowed after failure. Timeout limits how long an activity can execute.

### Q7. How do you handle activity failures in ADF?

I use activity dependency conditions such as Failed or Completed to route execution to an error-handling path. I can capture error details, log them, and send notifications.

### Q8. How do you log pipeline errors?

I capture pipeline name, run ID, activity name, error code, error message, and failure time, then write these details to a logging destination such as Azure SQL Database.

### Q9. What is idempotency in data pipelines?

Idempotency means that repeating an operation produces the same final effect under defined conditions. It helps prevent duplicate or inconsistent data during retries.

### Q10. Why is idempotency important when rerunning a failed pipeline?

A pipeline may partially complete before failing. If the workflow is not idempotent, rerunning it can insert duplicate records or repeat side effects.

### Q11. What is the difference between ADF Monitor and Azure Monitor?

ADF Monitor provides pipeline-specific execution details. Azure Monitor provides broader Azure resource monitoring, diagnostic collection, alerting, and integration with Log Analytics.

### Q12. How can you send an alert when a pipeline fails?

I can configure ADF failure handling to invoke a notification workflow, or configure diagnostic settings and Azure Monitor alert rules with an action group.

### Q13. What would you do if a Copy Activity fails after writing some data?

I would first inspect the error and determine whether the sink write was partial or committed. Then I would verify target state, fix the root cause, and rerun using an idempotent or otherwise safe recovery strategy.

### Q14. Does a successful pipeline mean the data is correct?

No. A successful execution confirms that configured activities completed successfully. Business-level validation, row-count reconciliation, and data quality checks may still be required.

### Q15. How do you troubleshoot a slow pipeline?

I inspect activity duration and Copy Activity performance details, then investigate source performance, sink performance, network latency, Integration Runtime capacity, parallelism, and throttling.

---

## 19. Quick Revision

| Topic | Key Point |
|---|---|
| Pipeline Run | One execution of a pipeline |
| Activity Run | One execution of an activity |
| Monitor | Inspect pipeline and activity execution |
| Copy Monitoring | Rows, bytes, duration, throughput |
| Retry | Attempts another execution |
| Timeout | Limits execution duration |
| Failed dependency | Runs a branch after failure |
| Completed dependency | Runs after completion, regardless of outcome |
| Error Logging | Captures failure details |
| Azure Monitor | Central monitoring and alerting |
| Log Analytics | Query collected logs using KQL |
| Action Group | Sends configured alerts |
| Idempotency | Makes repeated operations safe |
| Recovery | Fix the cause and rerun safely |

### Most Important Interview Takeaways

1. Know how to inspect pipeline and activity runs.
2. Understand retry, timeout, and dependency conditions.
3. Explain how to log errors and notify teams.
4. Understand partial failures and safe reruns.
5. Explain why idempotency matters.
6. Distinguish ADF Monitor from Azure Monitor.

---

## 20. Self-Assessment

Try answering these questions without looking at the notes:

1. What is monitoring in Azure Data Factory?
2. What is the difference between a pipeline run and an activity run?
3. How do you identify a failed activity?
4. What details do you inspect when Copy Activity fails?
5. What are common reasons for pipeline failure?
6. What is the difference between retry and timeout?
7. How do activity dependency conditions help with error handling?
8. What is the difference between Failed and Completed dependencies?
9. How would you log pipeline errors?
10. How would you notify a team about a pipeline failure?
11. What is idempotency?
12. Why is idempotency important for pipeline recovery?
13. How would you safely rerun a failed incremental load?
14. What is the difference between ADF Monitor and Azure Monitor?
15. How would you investigate a slow Copy Activity?

**Interview readiness:** You should be able to explain how you monitor an ADF pipeline, identify and handle activity failures, configure retries, log errors, and safely recover from partial execution.