# Databricks Job Orchestration

## 1. What Is Job Orchestration?

Job orchestration is the process of coordinating and managing multiple tasks so that a data pipeline executes in the correct order.

In Azure Databricks, Jobs and Workflows are used to automate data processing tasks.

A workflow can:
- Execute multiple tasks.
- Define task dependencies.
- Run tasks sequentially or in parallel.
- Schedule executions.
- Pass parameters between tasks.
- Retry failed tasks.
- Send notifications.
- Monitor execution status.

**Example:** A daily data pipeline reads files from ADLS Gen2, cleans the data, creates aggregations, and updates Gold tables. Instead of running each notebook manually, a Databricks workflow automates the complete process.

## 2. Job vs Task vs Workflow

These terms are related but have different meanings.

| Term | Meaning |
|---|---|
| Job | Configured unit of automated execution |
| Task | Individual unit of work within a job |
| Workflow | Orchestration of tasks and their dependencies |

A job can contain one or multiple tasks.

Example:

```text
Databricks Job
      |
      v
   Workflow
      |
      +----------------+
      |                |
      v                v
 Ingest Data       Load Reference Data
      |                |
      v                |
 Clean Data             |
      |                |
      +-------+--------+
              |
              v
       Create Gold Tables
              |
              v
       Data Quality Checks
```

In this example:
- Ingestion and reference-data loading can run in parallel.
- Cleaning depends on ingestion.
- Gold table creation waits for both branches.
- Data quality checks run after Gold table creation.

## 3. Why Is Job Orchestration Needed?

Without orchestration, engineers may need to execute notebooks manually and manage task order themselves.

Orchestration provides:

1. **Automation:** Runs pipelines without manual execution.
2. **Dependency management:** Ensures tasks run in the correct order.
3. **Parallel execution:** Runs independent tasks simultaneously.
4. **Failure handling:** Supports retries and failure notifications.
5. **Scheduling:** Runs workloads at defined times or intervals.
6. **Monitoring:** Tracks job and task execution.
7. **Parameterization:** Supports reusable workflows.

## 4. Databricks Jobs and Workflows Architecture

A typical workflow looks like this:

```text
             Trigger / Schedule
                    |
                    v
             Databricks Job
                    |
                    v
              Task Scheduler
                    |
          +---------+---------+
          |         |         |
          v         v         v
       Task 1    Task 2    Task 3
          |         |         |
          +---------+---------+
                    |
                    v
              Task 4
                    |
                    v
             Final Output
                    |
                    v
            Monitoring / Alerts
```

The workflow scheduler evaluates task dependencies and starts tasks when their required conditions are satisfied.

## 5. Creating a Multi-Task Workflow

Consider an e-commerce pipeline with these tasks:

| Task | Responsibility |
|---|---|
| `ingest_orders` | Read raw order data |
| `clean_orders` | Validate and clean records |
| `create_customer_sales` | Generate customer-level sales |
| `validate_output` | Validate the final output |

### Dependency structure

```text
ingest_orders
      |
      v
clean_orders
      |
      v
create_customer_sales
      |
      v
validate_output
```

Each task can be implemented as a notebook, Python script, SQL task, or another supported task type.

### Example task responsibilities

**Task 1: Ingest orders**

```python
orders = spark.read.parquet(
    "abfss://bronze@stdataeng001.dfs.core.windows.net/orders/"
)

print(f"Rows read: {orders.count()}")
```

**Task 2: Clean orders**

```python
from pyspark.sql.functions import col

clean_orders = (
    orders
    .filter(col("order_id").isNotNull())
    .filter(col("customer_id").isNotNull())
    .dropDuplicates(["order_id"])
)
```

**Task 3: Create customer sales**

```python
from pyspark.sql.functions import sum

customer_sales = (
    clean_orders
    .groupBy("customer_id")
    .agg(sum("amount").alias("total_sales"))
)
```

**Task 4: Validate output**

```python
if customer_sales.filter(
    col("customer_id").isNull()
).limit(1).count() > 0:
    raise ValueError("Customer ID validation failed")

print("Validation successful")
```

These snippets illustrate the processing logic. In a real multi-task workflow, tasks must exchange data through storage, tables, or explicitly configured task values rather than assuming Python variables are shared across separate notebook tasks.

## 6. Task Dependencies

Task dependencies define the order in which workflow tasks execute.

A task can depend on one or more upstream tasks.

### Sequential execution

```text
Task A
  |
  v
Task B
  |
  v
Task C
```

Task B waits for Task A, and Task C waits for Task B.

### Parallel execution

```text
       Task A
       /    \
      v      v
  Task B   Task C
      \      /
       v    v
       Task D
```

Task B and Task C can run concurrently after Task A completes. Task D waits for both dependencies.

### When should tasks run in parallel?

Use parallel execution when:
- Tasks are independent.
- They do not depend on each other's output.
- Enough compute capacity is available.
- Running them together does not create unsafe writes to shared resources.

Avoid parallel execution when tasks update the same target in conflicting ways or depend on shared mutable state.

## 7. Dependency Conditions

Databricks workflows support dependency conditions that determine whether a downstream task can run.

Common conditions include:

| Condition | Meaning |
|---|---|
| All succeeded | Upstream dependencies succeeded |
| At least one succeeded | At least one dependency succeeded |
| None failed | No dependency failed |
| All done | Dependencies finished, regardless of outcome |

The available condition choices and behavior depend on the workflow configuration and task type.

### Example

Suppose a workflow has two independent validation tasks:

```text
       Data Validation
              |
              v
       Quality Report
```

If the report must run even when validation fails, configure the dependency so that it runs after the upstream task completes rather than only after success.

The report should record the validation failure instead of incorrectly marking the data as valid.

## 8. Job Scheduling and Triggers

Databricks jobs can be started in different ways.

Common execution methods include:

- Manual execution.
- Time-based schedules.
- File-arrival triggers where supported.
- Continuous execution for supported workloads.
- API-based execution.
- Execution initiated by another workflow or orchestration system.

### Scheduled jobs

A scheduled job runs at configured times.

Example:

```text
Daily Orders Pipeline
       |
       v
Every day at 02:00 AM
       |
       v
Ingest -> Transform -> Validate
```

Scheduling configuration includes the time zone and recurrence.

**Important:** A schedule defines when execution is requested. It does not guarantee that a previous run has completed or that no overlapping run will occur. Configure concurrency behavior according to the pipeline's requirements.

## 9. Parameters in Databricks Workflows

Parameters make workflows reusable.

Instead of hardcoding storage paths or processing dates, pass values at runtime.

Example parameters:

```text
environment = production
process_date = 2026-10-25
source_path = abfss://bronze@stdataeng001.dfs.core.windows.net/orders/
```

### Notebook parameters using widgets

```python
dbutils.widgets.text("process_date", "")
dbutils.widgets.text("environment", "dev")

process_date = dbutils.widgets.get("process_date")
environment = dbutils.widgets.get("environment")

print(process_date)
print(environment)
```

A job can provide values to notebook task parameters.

### Using parameters in processing

```python
from pyspark.sql.functions import col

dbutils.widgets.text("process_date", "")

process_date = dbutils.widgets.get("process_date")

filtered_df = orders.filter(
    col("order_date") == process_date
)
```

The date format and data type should be validated before processing.

### Task values

Databricks task values allow one task to publish small values that downstream tasks can access.

Example: An ingestion task publishes the number of records processed.

In the upstream task:

```python
dbutils.jobs.taskValues.set(
    key="records_processed",
    value=12500
)
```

In a downstream task:

```python
records_processed = dbutils.jobs.taskValues.get(
    taskKey="ingest_orders",
    key="records_processed",
    default=0
)

print(records_processed)
```

Task values are useful for passing small metadata values, not large DataFrames or datasets.

For large data, write the output to ADLS Gen2 or a governed table and let downstream tasks read it.

## 10. Passing Data Between Tasks

Tasks in a workflow usually run in separate execution contexts.

Do not assume that Python variables or Spark DataFrames created in one notebook are automatically available in another task.

Common ways to exchange information include:

| Method | Suitable for |
|---|---|
| ADLS Gen2 files | Large datasets and intermediate files |
| Delta tables | Structured, reusable data |
| Task values | Small metadata values |
| Job parameters | Runtime configuration |
| Shared external systems | Data or state managed outside the workflow |

### Example: Sharing data through Delta

Task 1 writes cleaned data:

```python
clean_path = (
    "abfss://silver@stdataeng001.dfs.core.windows.net/orders/"
)

clean_orders.write.format("delta").mode("overwrite").save(clean_path)
```

Task 2 reads it:

```python
clean_orders = (
    spark.read
    .format("delta")
    .load(clean_path)
)
```

For production pipelines, use incremental or transactional write patterns instead of overwriting the full target when appropriate.

## 11. Retries and Failure Handling

A workflow should handle temporary failures safely.

Examples:
- Temporary network problems.
- Storage throttling.
- Compute startup failures.
- Service interruptions.
- Temporary source-system unavailability.

### Retry configuration

A task can be configured with:
- Maximum retry attempts.
- Retry interval.
- Task timeout.
- Notifications on failure.

Example:

```text
Task: ingest_orders

Retry attempts: 3
Retry interval: 2 minutes
Timeout: 30 minutes
```

These values are illustrative, not universal recommendations.

### Important: Idempotency

A task should be designed so that retrying it does not create incorrect duplicate results.

For example, blindly appending the same batch after a failed run can duplicate records.

Safer approaches include:
- MERGE using a stable business key.
- Processing each batch with a unique identifier.
- Overwriting a well-defined partition when safe.
- Recording successfully processed batches.
- Using transactional table writes.

**Interview point:** Retries improve resilience, but retries alone do not guarantee correct data. The pipeline must also be idempotent or have a safe recovery strategy.

## 12. Job Monitoring

Databricks provides job and task run information.

Common monitoring details include:
- Job run status.
- Task run status.
- Start and end time.
- Duration.
- Error messages.
- Execution logs.
- Retry attempts.
- Task dependencies.

### Common statuses

| Status | Meaning |
|---|---|
| Pending | Waiting to start |
| Running | Currently executing |
| Succeeded | Completed successfully |
| Failed | Execution failed |
| Skipped | Not executed due to workflow conditions |
| Cancelled | Execution was cancelled |

Exact status labels may vary by interface and product version.

### Monitoring approach

1. Open the Databricks workspace.
2. Navigate to Jobs and Workflows.
3. Select the required job.
4. Open the relevant run.
5. Review the task graph.
6. Select the failed task.
7. Inspect its error details and logs.
8. Correct the problem and rerun safely.

## 13. Notifications and Alerts

Notifications help teams respond to failures.

Common notification events include:
- Job start.
- Job success.
- Job failure.
- Long-running execution or duration threshold, where supported.

Notifications can be sent through supported email or system integrations.

A useful production setup sends alerts when:
- A critical ingestion task fails.
- A Gold table is not refreshed on time.
- Data quality validation fails.
- A job exceeds its expected duration.

Avoid sending unnecessary notifications for every minor event. Focus on actionable alerts.

## 14. Databricks Workflows vs Azure Data Factory

Both Databricks Workflows and Azure Data Factory can orchestrate data pipelines, but their strengths differ.

| Feature | Databricks Workflows | Azure Data Factory |
|---|---|---|
| Primary focus | Databricks workload orchestration | Data integration and orchestration |
| Notebook execution | Native Databricks tasks | Can invoke Databricks notebooks |
| Spark processing | Executes on Databricks compute | Usually delegates Spark work to a compute service |
| Data movement | Possible through supported tasks and code | Strong built-in Copy Activity |
| Task dependencies | Supported | Supported |
| Scheduling | Supported | Supported |
| External system integration | Depends on supported tasks and integrations | Broad connector ecosystem |

### When to use Databricks Workflows

Use it when:
- Most processing happens inside Databricks.
- Tasks are notebooks, Python, SQL, or other Databricks workloads.
- You want to manage task dependencies near the processing code.

### When to use Azure Data Factory

Use it when:
- You need to ingest data from many external systems.
- You need built-in connectors and Copy Activity.
- You want to orchestrate services across Azure and external platforms.
- Databricks is one processing step in a broader integration pipeline.

### Can both be used together?

Yes.

Example:

```text
             Azure Data Factory
                     |
                     v
              Ingest Source Data
                     |
                     v
               ADLS Gen2
                     |
                     v
            Databricks Workflow
                     |
          +----------+----------+
          |                     |
          v                     v
      Clean Data           Validate Data
          |                     |
          +----------+----------+
                     |
                     v
                Gold Tables
```

Choose one primary orchestration layer for a given workflow where possible. Avoid creating unnecessary nested orchestration that makes monitoring and failure recovery difficult.

## 15. Best Practices

1. Break large pipelines into manageable tasks.
2. Use meaningful task names.
3. Define dependencies explicitly.
4. Run independent tasks in parallel when safe.
5. Pass runtime values through parameters.
6. Use task values only for small metadata.
7. Store large intermediate datasets in ADLS Gen2 or Delta tables.
8. Make tasks idempotent.
9. Configure sensible retry and timeout values.
10. Monitor job duration and failure patterns.
11. Use alerts for critical failures.
12. Separate development and production workflows.
13. Use Git and controlled deployment processes.
14. Avoid hardcoding secrets and environment-specific paths.
15. Document dependencies and recovery procedures.

## 16. Practical Interview Scenario

**Question:** You need to build a daily e-commerce pipeline with four steps: ingest orders, clean orders, create customer sales, and validate the final output. How would you orchestrate it?

### Step 1: Ingestion

Read source files and write them to the Bronze layer.

### Step 2: Cleaning

Read Bronze data, remove invalid records, apply data types, and deduplicate records according to business rules. Write the result to Silver.

### Step 3: Aggregation

Read Silver data and calculate customer-level sales. Write the result to Gold.

### Step 4: Validation

Check:
- Required columns.
- Null values in key columns.
- Duplicate business keys.
- Expected row counts.
- Reconciliation totals.

### Workflow

```text
             Daily Schedule
                    |
                    v
              Ingest Orders
                    |
                    v
              Clean Orders
                    |
                    v
          Create Customer Sales
                    |
                    v
             Validate Output
                    |
             +------+------+
             |             |
             v             v
          Success        Failure
             |             |
             v             v
       Complete Job    Alert Team
```

### Production considerations

- Configure retry for temporary failures.
- Use idempotent writes to support safe reruns.
- Store intermediate data in Delta tables.
- Pass the processing date as a job parameter.
- Monitor task-level execution.
- Alert the team when validation fails.
- Provide a controlled backfill mechanism for missed dates.

## 17. Common Interview Questions and Answers

### Q1. What is job orchestration?

Job orchestration coordinates multiple tasks, manages their dependencies, and controls their execution.

### Q2. What is the difference between a job and a task?

A job is the configured automated execution unit. A task is an individual operation within that job.

### Q3. Can Databricks execute multiple tasks in parallel?

Yes. Independent tasks can execute in parallel when their dependencies are satisfied and sufficient compute capacity is available.

### Q4. How do you define task dependencies?

Dependencies are configured in the workflow task graph. A downstream task starts when its upstream dependency conditions are met.

### Q5. How do you pass parameters to a notebook task?

Use job or task parameters and retrieve them in the notebook through supported parameter mechanisms, such as Databricks widgets.

### Q6. Can one notebook task directly share a DataFrame with another task?

Normally, no. Tasks run in separate execution contexts. Store the DataFrame in a table or file, or use task values for small metadata.

### Q7. What are task values?

Task values allow a task to publish small values that downstream tasks can retrieve.

### Q8. How do you handle task failures?

Use retries, timeouts, notifications, logging, and safe rerun strategies. The processing logic should also be idempotent where possible.

### Q9. Why is idempotency important in orchestration?

A failed task may be retried or rerun. Idempotency prevents repeated execution from creating unintended duplicate or inconsistent results.

### Q10. What is the difference between sequential and parallel execution?

Sequential execution runs dependent tasks in order. Parallel execution runs independent tasks at the same time.

### Q11. What is the difference between Databricks Workflows and ADF?

Databricks Workflows is focused on orchestrating Databricks workloads. ADF provides broader data integration and orchestration capabilities across many sources and services.

### Q12. Can ADF trigger a Databricks workflow?

Yes. ADF can invoke supported Databricks job or notebook execution mechanisms as part of a larger pipeline.

### Q13. How would you monitor a failed workflow?

Open the job run, inspect the task graph, identify the failed task, review its logs and error message, correct the cause, and rerun safely.

### Q14. How do you prevent duplicate data during retries?

Use idempotent writes, stable business keys, Delta MERGE, batch identifiers, or safe partition replacement according to the pipeline design.

### Q15. How would you design a production workflow?

I would divide the pipeline into ingestion, transformation, aggregation, and validation tasks. I would define dependencies, use parameters, enable safe retries, monitor execution, and implement idempotent writes and failure alerts.

## 18. Quick Revision

| Concept | Key point |
|---|---|
| Job | Configured automated execution |
| Task | Individual unit of work |
| Workflow | Orchestrates tasks |
| Dependency | Controls task execution order |
| Parallel execution | Runs independent tasks together |
| Schedule | Starts jobs at configured times |
| Parameters | Pass runtime configuration |
| Task values | Pass small values between tasks |
| Retry | Reattempts failed execution |
| Timeout | Limits execution duration |
| Idempotency | Makes reruns safe under defined conditions |
| Monitoring | Tracks job and task execution |
| ADF | Broad data integration and orchestration |
| Databricks Workflows | Orchestration of Databricks workloads |

## 19. Self-Assessment

Try answering these without looking at the notes:

1. What is job orchestration?
2. Differentiate job, task, and workflow.
3. How do task dependencies work?
4. When should tasks run in parallel?
5. How do you schedule a Databricks job?
6. How do you pass parameters to a notebook?
7. What are task values?
8. How can two notebook tasks share a large DataFrame?
9. How do retries and idempotency work together?
10. How do you monitor a failed workflow?
11. Compare Databricks Workflows and Azure Data Factory.
12. Design a production workflow for a daily e-commerce pipeline.

**Interview readiness target:** You should be able to design a multi-task workflow, explain dependencies and parallel execution, pass parameters between tasks, and describe how retries, idempotency, and monitoring make a production pipeline reliable.