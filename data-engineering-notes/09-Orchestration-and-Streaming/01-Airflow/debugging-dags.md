

# Debugging DAGs in Apache Airflow

## 1. Overview

Debugging a DAG means identifying, understanding, and fixing problems that prevent an Airflow workflow from executing correctly.

In a data engineering pipeline, failures can happen because of:

- Incorrect DAG definitions.
- Python import errors.
- Invalid task dependencies.
- Missing packages or connections.
- Authentication and permission issues.
- Database or API connectivity problems.
- Incorrect SQL or transformation logic.
- Resource limitations and timeouts.
- Incorrect scheduling or configuration.

A good debugging approach is to identify **where the failure occurred, why it occurred, and whether rerunning the task is safe**.

---

## 2. Understand the Airflow Execution Flow

Before debugging, understand how Airflow processes a DAG.

```text
DAG Python File
      |
      v
DAG Parsing
      |
      v
DAG Processor
      |
      v
Scheduler
      |
      v
Task Instance Created
      |
      v
Executor
      |
      v
Worker / Execution Environment
      |
      v
Task Execution
      |
      v
Task Logs and Final State
```

A failure can occur at different stages.

| Stage | Possible Problem |
|---|---|
| DAG parsing | Syntax or import errors |
| DAG scheduling | Incorrect schedule or start date |
| Dependency evaluation | Upstream task not ready |
| Task execution | Python, SQL, or application error |
| External connection | Network or authentication failure |
| Resource allocation | Insufficient capacity or timeout |
| Data writing | Schema mismatch or duplicate records |

**Important:** Not every DAG problem is a task execution problem. A DAG may fail to load before any task instance is created.

---

## 3. Understand Common Airflow Task States

Task states help identify what happened during execution.

| State | Meaning |
|---|---|
| `none` | No task instance state has been assigned yet |
| `scheduled` | Task is eligible and waiting for scheduling |
| `queued` | Task has been sent for execution |
| `running` | Task is currently executing |
| `success` | Task completed successfully |
| `failed` | Task execution failed |
| `upstream_failed` | Required upstream task failed |
| `skipped` | Task was intentionally skipped or not selected |
| `up_for_retry` | Task is waiting for another attempt |
| `up_for_reschedule` | A rescheduling sensor is waiting to check again |
| `deferred` | A deferrable task has handed control to the Triggerer |
| `removed` | Task was removed from the DAG definition after the relevant run was created |

Some states and their availability can vary by Airflow version and execution configuration.

### Example

```text
Extract Data       SUCCESS
       |
       v
Transform Data    FAILED
       |
       v
Load Data         UPSTREAM_FAILED
```

In this example:

- Extraction completed successfully.
- Transformation failed.
- Loading did not run because its upstream dependency failed.

**Interview point:** `upstream_failed` does not necessarily mean the task itself executed and failed. It means an upstream dependency prevented it from running successfully.

---

## 4. Debugging DAG Import Errors

A DAG must be parsed successfully before Airflow can schedule its tasks.

Common import errors include:

- Python syntax errors.
- Missing Python packages.
- Incorrect module imports.
- Incorrect operator names.
- Invalid DAG configuration.
- Code executed at import time that raises an exception.

### Example: Incorrect import

```python
from airflow.operators.python_operator import PythonOperator
```

This is an older import path and may not work in newer Airflow versions.

Use the current import path:

```python
from airflow.operators.python import PythonOperator
```

### Example: Syntax error

Incorrect:

```python
def extract_data()
    print("Extracting data")
```

Correct:

```python
def extract_data():
    print("Extracting data")
```

### How to troubleshoot

1. Check the DAG's import errors in the Airflow UI.
2. Review DAG processor or scheduler logs, depending on the deployment.
3. Check Python syntax.
4. Verify installed packages.
5. Verify Airflow provider packages.
6. Test the DAG file in the same Python environment used by Airflow.

### CLI checks

```bash
airflow dags list
```

Lists DAGs known to Airflow.

```bash
airflow dags list-import-errors
```

Lists DAG import errors.

```bash
python -m py_compile dags/customer_pipeline.py
```

Checks Python syntax without executing the DAG.

**Important:** A successful syntax check does not guarantee that imports or Airflow-specific DAG parsing will succeed.

---

## 5. Debugging Failed Tasks

When a task fails, follow a consistent process.

### Step 1: Identify the failed task

Open the DAG in the Airflow UI and select the relevant DAG Run.

### Step 2: Open task logs

Inspect the task instance logs.

Look for:

- Exception type.
- Error message.
- Python traceback.
- SQL error.
- Connection failure.
- Timeout.
- Missing configuration.

### Step 3: Find the actual root cause

Do not stop at the last error message.

For example:

```text
Task failed
    |
    v
Connection error
    |
    v
Authentication failed
    |
    v
Incorrect credentials or insufficient permissions
```

The root cause may be an invalid credential rather than an Airflow problem.

### Step 4: Fix the problem

Depending on the error:

- Correct the Python code.
- Update the SQL query.
- Fix the connection configuration.
- Grant the required permissions.
- Install the missing package.
- Increase an appropriate timeout.
- Fix the source or target data.

### Step 5: Rerun safely

Before rerunning, check whether the task partially wrote data.

A retry must not create duplicate or inconsistent records.

---

## 6. Using the Airflow UI for Debugging

The Airflow UI provides several useful views.

### 6.1 DAGs View

Use this to:

- Check whether the DAG is loaded.
- Check whether the DAG is paused.
- View the latest DAG Run.
- Identify scheduling problems.

### 6.2 Grid View

Grid View helps inspect task states across DAG Runs.

It is useful for identifying:

- Repeated failures.
- Tasks that consistently take longer.
- Tasks stuck in a particular state.
- Failures affecting multiple scheduled runs.

### 6.3 Graph View

Graph View shows task dependencies.

Use it to identify:

- Failed upstream tasks.
- Unexpected dependencies.
- Tasks marked `upstream_failed`.
- Branches marked skipped.
- Incorrect workflow structure.

### 6.4 Task Instance Details

Open an individual task instance to inspect:

- Task state.
- Start and end times.
- Duration.
- Try number.
- Host or execution details, when available.
- Logs.
- Rendered template fields.

### 6.5 Rendered Templates

For tasks using Jinja templates, inspect the rendered SQL or command.

Example:

```sql
SELECT *
FROM orders
WHERE order_date = '{{ ds }}';
```

The rendered template shows the actual value used for that task instance.

This helps identify incorrect dates, missing parameters, and unexpected template output.

---

## 7. Debugging with Airflow CLI

The Airflow CLI is useful when the UI is unavailable or when you need to inspect a particular DAG Run.

### List DAGs

```bash
airflow dags list
```

### Check import errors

```bash
airflow dags list-import-errors
```

### Show DAG information

```bash
airflow dags show customer_pipeline
```

### List DAG Runs

```bash
airflow dags list-runs -d customer_pipeline
```

### Test a task

```bash
airflow tasks test customer_pipeline extract_data 2026-10-25
```

This runs the specified task for the supplied logical date in a test context.

**Important:** `airflow tasks test` is useful for debugging a task, but it is not equivalent to a normal scheduled task execution. It does not reproduce the complete scheduler and executor workflow, and it may not reflect normal dependency evaluation.

### Check task states

```bash
airflow tasks states-for-dag-run customer_pipeline scheduled__2026-10-25T00:00:00+00:00
```

The exact CLI command options and accepted run identifiers can vary by Airflow version. Check the installed version's CLI help.

---

## 8. Debugging PythonOperator Tasks

A PythonOperator executes a Python callable.

Example:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from datetime import datetime


def extract_data():
    print("Extracting data")

    data = [100, 200, 300]

    return data


with DAG(
    dag_id="python_debugging_example",
    start_date=datetime(2026, 1, 1),
    schedule=None,
    catchup=False,
) as dag:

    extract = PythonOperator(
        task_id="extract_data",
        python_callable=extract_data,
    )
```

### Common Python errors

| Error | Possible Cause |
|---|---|
| `NameError` | Variable or function is not defined |
| `TypeError` | Incorrect data type or function argument |
| `ValueError` | Invalid value supplied |
| `KeyError` | Dictionary key does not exist |
| `FileNotFoundError` | File path does not exist in the execution environment |
| `ModuleNotFoundError` | Required package or module is unavailable |
| `PermissionError` | Insufficient file or resource permissions |
| `TimeoutError` | Operation exceeded its allowed time |

### Example: KeyError

Incorrect:

```python
def process_data():
    customer = {
        "customer_id": 101,
        "name": "Rahul",
    }

    print(customer["email"])
```

The dictionary does not contain the `email` key.

Correct:

```python
def process_data():
    customer = {
        "customer_id": 101,
        "name": "Rahul",
    }

    email = customer.get("email")

    print(email)
```

Using `.get()` avoids a `KeyError` when the key is missing, but the application should still validate whether the missing value is acceptable.

### Debugging best practices

- Log meaningful messages.
- Validate input data.
- Handle expected exceptions.
- Avoid catching every exception and silently continuing.
- Do not print passwords, tokens, or sensitive information.
- Keep task functions focused on a clear responsibility.

---

## 9. Logging and Error Messages

Logs are one of the most important tools for diagnosing task failures.

Use Python's logging module rather than relying only on `print()` statements.

### Example

```python
import logging

logger = logging.getLogger(__name__)


def extract_data():
    logger.info("Starting data extraction")

    try:
        logger.info("Connecting to source database")

        # Extraction logic goes here

        logger.info("Data extraction completed")

    except Exception:
        logger.exception("Data extraction failed")
        raise
```

### Common logging levels

| Level | Purpose |
|---|---|
| DEBUG | Detailed diagnostic information |
| INFO | Normal execution progress |
| WARNING | Unexpected condition that does not necessarily stop execution |
| ERROR | An operation failed |
| CRITICAL | Serious failure affecting the application |

### Why use `logger.exception()`?

`logger.exception()` records an error message and includes the exception traceback when called inside an exception handler.

The `raise` statement is important because it allows Airflow to recognize the task as failed.

### Bad practice

```python
try:
    # Data processing
    pass
except Exception:
    print("Something went wrong")
```

This hides the original exception and may allow the task to appear successful if the exception is swallowed.

### Better practice

```python
try:
    # Data processing
    pass
except Exception:
    logger.exception("Data processing failed")
    raise
```

---

## 10. Debugging Connection and Authentication Errors

Data engineering pipelines frequently interact with external systems.

Examples:

- Azure SQL Database.
- ADLS Gen2.
- Databricks.
- REST APIs.
- PostgreSQL.
- Snowflake.

### Common problems

| Error | Possible Cause |
|---|---|
| Connection refused | Service unavailable or port blocked |
| Connection timeout | Network, firewall, or routing issue |
| Authentication failed | Invalid credentials or token |
| Authorization failed | Identity lacks required permissions |
| DNS resolution failure | Incorrect hostname or DNS configuration |
| SSL/TLS error | Certificate or TLS configuration issue |

### Debugging checklist

1. Verify the hostname and port.
2. Check whether the target service is available.
3. Check network connectivity from the actual Airflow execution environment.
4. Verify credentials or managed identity configuration.
5. Verify the required permissions.
6. Check firewall and private endpoint configuration.
7. Inspect the connection configuration in Airflow.

### Important distinction

Authentication and authorization are different.

- Authentication confirms who the caller is.
- Authorization determines what the caller is allowed to do.

A valid managed identity does not automatically have permission to read or write data.

### Airflow Connections

Airflow Connections store connection configuration for external systems.

They can contain:

- Host.
- Port.
- Username.
- Password.
- Database.
- Extra configuration.

Use Airflow Connections and an appropriate secrets backend instead of hardcoding credentials in DAG files.

---

## 11. Debugging Scheduling Problems

Sometimes tasks do not fail; they simply do not start.

Possible causes include:

- DAG is paused.
- Incorrect schedule.
- Incorrect `start_date`.
- Incorrect timezone.
- `catchup=False` and no new scheduled interval is due.
- Upstream dependencies are incomplete.
- DAG or task concurrency limits.
- Executor or worker capacity is unavailable.
- A task is waiting for a pool slot.

### Example

```python
with DAG(
    dag_id="daily_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:
    ...
```

Check the following:

- Is the DAG unpaused?
- Is the schedule correct?
- Has the relevant data interval become eligible?
- Are upstream tasks complete?
- Is the scheduler healthy?
- Are executor slots available?
- Is the task waiting for a pool slot?

### Logical date vs actual execution time

A common source of confusion is the logical date.

For a scheduled DAG Run, the logical date generally identifies the start of its data interval. It is not necessarily the time when the task actually executes.

For example, a daily run may process data for the previous day even though it executes the following day.

Always inspect the data interval and rendered template values before assuming the task used the wrong date.

---

## 12. Debugging Sensors and Deferrable Tasks

Sensors wait for an external condition before allowing downstream tasks to proceed.

Examples:

- Wait for a file in ADLS Gen2.
- Wait for a database record.
- Wait for another DAG.
- Wait for an external job to complete.

### Common sensor problems

- Expected file does not arrive.
- Incorrect file path.
- Wrong storage permissions.
- Sensor timeout.
- Incorrect polling interval.
- External system is unavailable.

### Sensor modes

| Mode | Behavior |
|---|---|
| `poke` | Sensor keeps its worker slot while checking |
| `reschedule` | Sensor releases its worker slot between checks |
| Deferrable | Task defers execution and uses the Triggerer for asynchronous waiting |

### Example: File sensor

```python
from airflow.providers.standard.sensors.filesystem import FileSensor

wait_for_file = FileSensor(
    task_id="wait_for_file",
    fs_conn_id="my_filesystem",
    filepath="/data/orders.csv",
    poke_interval=60,
    timeout=3600,
    mode="reschedule",
)
```

This example requires the appropriate provider package and a correctly configured filesystem connection. Sensor import paths and provider availability can vary by Airflow installation.

### Debugging checklist

1. Verify the expected file path.
2. Check the connection ID.
3. Confirm that the file exists from the sensor's execution environment.
4. Verify permissions.
5. Review sensor logs.
6. Check the timeout and polling configuration.
7. Confirm whether the sensor is waiting, rescheduled, or deferred.

---

## 13. Retries and Timeouts

Retries allow Airflow to attempt a task again after a failure.

Timeouts prevent tasks from running indefinitely.

### Example

```python
from datetime import timedelta
from airflow.operators.python import PythonOperator

extract = PythonOperator(
    task_id="extract_data",
    python_callable=extract_data,
    retries=3,
    retry_delay=timedelta(minutes=5),
    execution_timeout=timedelta(minutes=30),
)
```

### Important parameters

| Parameter | Purpose |
|---|---|
| `retries` | Number of retry attempts after the initial attempt |
| `retry_delay` | Waiting time between attempts |
| `execution_timeout` | Maximum allowed execution time for the task |

### Example behavior

If a task has:

```python
retries=3
```

It can have one initial attempt plus up to three retry attempts.

### When should retries be used?

Good candidates:

- Temporary network failures.
- Temporary API unavailability.
- Transient database connection errors.
- Temporary service throttling.

Retries do not fix:

- Invalid SQL syntax.
- Incorrect credentials.
- Missing Python packages.
- Invalid input data.
- Permanent permission failures.

**Important:** A retry may repeat side effects. Ensure the task is idempotent or has a safe recovery strategy before enabling retries.

---

## 14. Debugging Data Pipeline Failures

A task can succeed technically while producing incorrect data.

For example, a SQL query may execute successfully but return zero records unexpectedly.

Therefore, debugging must include both task health and data health.

### Example data validation

```python
def validate_orders():
    order_count = 1000
    expected_minimum = 100

    if order_count < expected_minimum:
        raise ValueError(
            "Order count is below the expected minimum"
        )
```

This is a simple illustration. In production, retrieve the count from the actual source or target and compare it with an appropriate business rule.

### Common data problems

| Problem | Possible Cause |
|---|---|
| Zero records | Incorrect date filter or empty source |
| Duplicate records | Repeated ingestion or missing deduplication |
| Missing records | Incorrect watermark or late-arriving data |
| Incorrect totals | Join duplication or aggregation error |
| Schema mismatch | Source schema changed |
| Null values | Missing or invalid source data |
| Stale data | Pipeline did not run or failed before publishing |

### Data engineering debugging checklist

- Compare source and target record counts.
- Check primary or business-key uniqueness.
- Validate null values in required columns.
- Compare important business totals.
- Check the date range and data interval.
- Verify incremental watermark values.
- Inspect rejected or malformed records.
- Confirm that retries did not create duplicates.

---

## 15. Idempotency and Safe Reruns

Idempotency is critical when debugging and rerunning failed data pipelines.

An operation is idempotent when repeating it produces the same final effect under the defined conditions.

### Problem: Duplicate records

Suppose a task inserts 1,000 records into a target table and then fails before Airflow records the task as successful.

When retried, it may insert the same 1,000 records again.

The target could now contain duplicates.

### Safer approaches

- Use `MERGE` or upsert logic.
- Use deterministic business keys.
- Replace a specific data partition when appropriate.
- Use transaction boundaries where supported.
- Track completed processing intervals.
- Validate target data before marking the operation complete.

### Example: Delta Lake MERGE

```sql
MERGE INTO target_orders AS target
USING source_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
    UPDATE SET
        target.amount = source.amount,
        target.status = source.status

WHEN NOT MATCHED THEN
    INSERT (
        order_id,
        amount,
        status
    )
    VALUES (
        source.order_id,
        source.amount,
        source.status
    );
```

Before using this pattern, ensure the source contains at most one applicable row per target key. Multiple source matches can make a Delta MERGE ambiguous.

**Interview point:** Airflow retries provide another execution attempt. They do not automatically guarantee exactly-once data processing.

---

## 16. Debugging with External Logs and Monitoring

In production, Airflow's task logs may not be sufficient.

Use centralized monitoring to correlate failures across systems.

Examples:

- Airflow logs.
- Worker logs.
- Scheduler logs.
- Triggerer logs.
- Azure Monitor.
- Log Analytics.
- Databricks job logs.
- Database logs.
- API logs.

### Example troubleshooting flow

```text
Airflow Task Failed
        |
        v
Inspect Task Logs
        |
        v
Identify External System
        |
        v
Inspect External Logs
        |
        v
Find Root Cause
        |
        v
Fix Configuration / Code / Data
        |
        v
Rerun Safely
        |
        v
Validate Target Data
```

### What should be monitored?

**Pipeline health**
- DAG Run status.
- Task failure rate.
- Task duration.
- Retry count.
- Queue delay.

**Data health**
- Record counts.
- Data freshness.
- Null and duplicate counts.
- Schema changes.
- Source-to-target reconciliation.

A successful DAG Run does not automatically prove that the business data is correct.

---

## 17. Production Debugging Scenario

### Scenario

An Airflow pipeline loads daily orders into a Delta Lake table.

The workflow is:

```text
Extract Orders
      |
      v
Validate Orders
      |
      v
Transform Orders
      |
      v
Load Delta Table
      |
      v
Send Notification
```

One day, the `load_delta_table` task fails.

### Step 1: Identify the failed task

Open the relevant DAG Run in the Airflow UI and locate `load_delta_table`.

### Step 2: Inspect logs

Suppose the log shows:

```text
PermissionDenied:
The identity does not have write permission
on the target storage location.
```

### Step 3: Identify the root cause

The Airflow task is attempting to write to ADLS Gen2, but its configured identity does not have sufficient permissions.

### Step 4: Verify access

Check:

- Which identity the task uses.
- Whether authentication succeeds.
- Whether the identity has the required data-plane RBAC role or ACL permissions.
- Whether network access is configured correctly.

### Step 5: Fix the issue

Grant the required least-privilege access to the correct identity and target location.

### Step 6: Rerun safely

Before rerunning, check whether the failed operation partially committed data.

Use an idempotent write or MERGE strategy to avoid duplicates.

### Step 7: Validate

Compare source and target counts, key uniqueness, and important business totals.

### Interview answer

"I would first identify the failed task and inspect its logs to find the actual exception. Then I would determine whether the issue is related to code, connectivity, authentication, permissions, or data. After fixing the root cause, I would check whether the task partially wrote data and ensure that rerunning it is safe. Finally, I would validate the target data and confirm that the pipeline completed successfully."

---

## 18. Common Mistakes to Avoid

1. Looking only at the DAG Run status instead of inspecting the failed task.
2. Treating `upstream_failed` as an actual execution failure of that task.
3. Assuming every task failure is caused by Airflow.
4. Ignoring import errors.
5. Swallowing Python exceptions without re-raising them.
6. Increasing retries without identifying the failure type.
7. Increasing timeouts without understanding why execution is slow.
8. Rerunning data-writing tasks without checking for partial writes.
9. Hardcoding credentials in DAG files.
10. Confusing authentication failures with authorization failures.
11. Ignoring logical dates and data intervals.
12. Assuming task success guarantees data correctness.
13. Deleting checkpoints or intermediate data without understanding the recovery impact.
14. Debugging only from Airflow logs when the root cause belongs to an external service.

---

## 19. Interview Questions and Answers

### Q1. How do you debug a failed Airflow DAG?

I identify the failed task, inspect its logs, determine the root cause, fix the issue, and rerun the task safely. I also validate the output data.

### Q2. What is the difference between `failed` and `upstream_failed`?

`failed` means the task itself failed during execution. `upstream_failed` means an upstream dependency failed and prevented the task from running under its trigger rule.

### Q3. How do you debug DAG import errors?

I check the import errors in the Airflow UI or CLI, inspect DAG processor or scheduler logs, and verify Python syntax, imports, packages, and DAG configuration.

### Q4. How do you debug a task that is stuck in queued state?

I check scheduler health, executor configuration, worker availability, queue configuration, pool slots, and task/DAG concurrency limits.

### Q5. How do you debug a task that never starts?

I check whether the DAG is paused, whether the schedule and data interval are correct, whether upstream tasks are complete, and whether scheduler or executor capacity is available.

### Q6. What is the purpose of `airflow tasks test`?

It runs a specific task for a given logical date in a test context. It is useful for debugging task code but does not reproduce the complete normal scheduler and executor workflow.

### Q7. How do you handle temporary failures?

I use retries with an appropriate delay and timeout. I also ensure that rerunning the task does not create duplicate or inconsistent data.

### Q8. What is the difference between retry and backfill?

A retry attempts a failed task again within its DAG Run. A backfill creates DAG Runs for historical data intervals that need processing.

### Q9. How do you debug a PythonOperator failure?

I inspect the Python traceback, validate input values and data types, check package availability, and add meaningful logging around important operations.

### Q10. How do you debug a database connection failure?

I check the connection configuration, hostname, port, network access, credentials, authentication method, and database permissions.

### Q11. Why is idempotency important in Airflow?

Airflow may retry or rerun tasks. Idempotency helps ensure repeated execution does not create duplicate or inconsistent target data.

### Q12. How do you debug a sensor that keeps waiting?

I verify the expected condition, file path or external resource, connection, permissions, polling interval, and timeout.

### Q13. How do you debug incorrect output when the DAG succeeds?

I validate source and target record counts, data types, null values, duplicates, business totals, date filters, and transformation logic.

### Q14. What is the difference between task logs and scheduler logs?

Task logs show details of an individual task execution. Scheduler logs help diagnose DAG scheduling, task scheduling, and scheduler-level problems.

### Q15. How would you prevent sensitive information from appearing in logs?

I avoid logging credentials, tokens, and connection secrets. I use Airflow Connections or a secrets backend and log only non-sensitive diagnostic information.

### Q16. What would you do if a task fails after writing some data?

I inspect the target to determine what was committed. Then I use a safe recovery strategy, such as an idempotent MERGE or controlled partition replacement, before rerunning the task.

---

## 20. Quick Revision

| Topic | Key Point |
|---|---|
| DAG import error | DAG cannot be parsed correctly |
| Task failure | Task execution raised an error |
| `upstream_failed` | Upstream failure prevented execution |
| Grid View | Inspect task states across DAG Runs |
| Graph View | Inspect dependencies and execution paths |
| Task logs | Find the task-level error and traceback |
| Scheduler logs | Diagnose scheduling problems |
| `airflow dags list-import-errors` | Check DAG import errors |
| `airflow tasks test` | Test a specific task in a test context |
| Retry | Attempts task execution again |
| Timeout | Limits task execution duration |
| Sensor | Waits for an external condition |
| Authentication | Verifies identity |
| Authorization | Determines permissions |
| Idempotency | Makes safe repeated execution possible |
| Data validation | Confirms output correctness |

### Five-step debugging approach

1. **Locate:** Identify the DAG Run and failed task.
2. **Inspect:** Read task logs and relevant system logs.
3. **Diagnose:** Find the root cause.
4. **Fix:** Correct the code, configuration, access, or data issue.
5. **Recover:** Rerun safely and validate the output.

---

## 21. Self-Assessment

Try answering these questions without looking at the notes:

1. What are the common reasons a DAG fails to load?
2. What is the difference between `failed` and `upstream_failed`?
3. How do you debug a task that remains queued?
4. How do you use Airflow UI and CLI for debugging?
5. What is the purpose of `airflow tasks test`?
6. Why should exceptions be re-raised after logging?
7. When should retries be used?
8. How do you debug a sensor that cannot find a file?
9. What is the difference between authentication and authorization?
10. Why is idempotency important when rerunning failed tasks?
11. How would you debug a successful DAG that produces incorrect data?
12. Explain how you would troubleshoot a failed Airflow-to-ADLS Gen2 pipeline.

