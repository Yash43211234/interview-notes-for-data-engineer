

# Scheduling and Retries in Apache Airflow

## 1. Overview

Scheduling and retries are important features of Apache Airflow that help manage when workflows execute and how they recover from failures.

- **Scheduling:** Determines when a DAG should run.
- **Retries:** Allow Airflow to attempt a failed task again.
- **Timeouts:** Prevent tasks from running longer than the allowed duration.
- **Concurrency controls:** Limit how many DAG runs or tasks can execute simultaneously.

Example: A daily orders pipeline should run every day, retry temporary database connection failures, and stop waiting if a task exceeds its execution limit.

---

## 2. How DAG Scheduling Works

Airflow uses a DAG's schedule to determine when DAG Runs should be created.

A schedule can be defined using:

- Cron expressions.
- Preset schedules such as `@daily`.
- Timedelta intervals.
- Timetables.
- Asset-based scheduling in supported Airflow versions.

### Example: Daily schedule

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.empty import EmptyOperator

with DAG(
    dag_id="daily_orders_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:

    process_orders = EmptyOperator(
        task_id="process_orders"
    )
```

Here:

- `schedule="@daily"` defines a daily schedule.
- `start_date` establishes the beginning of the scheduling timeline.
- `catchup=False` prevents automatic creation of the full historical backlog.

**Important:** A schedule determines when a DAG Run becomes eligible. It does not guarantee that the task starts at that exact time.

---

## 3. Common Airflow Schedule Presets

Airflow provides convenient scheduling presets.

| Preset | Meaning |
|---|---|
| `@once` | Run once |
| `@hourly` | Run hourly |
| `@daily` | Run daily |
| `@weekly` | Run weekly |
| `@monthly` | Run monthly |
| `@quarterly` | Run quarterly |
| `@yearly` | Run yearly |
| `None` | No automatic time-based schedule |

### Examples

```python
schedule="@hourly"
```

Runs hourly.

```python
schedule="@daily"
```

Runs daily.

```python
schedule="@weekly"
```

Runs weekly.

```python
schedule=None
```

The DAG does not have a time-based schedule. It can still be triggered manually or through supported external mechanisms.

---

## 4. Cron Expressions

Cron expressions provide more control over scheduling.

A standard five-field cron expression has this structure:

```text
minute hour day-of-month month day-of-week
```

| Field | Allowed Values |
|---|---|
| Minute | 0–59 |
| Hour | 0–23 |
| Day of month | 1–31 |
| Month | 1–12 |
| Day of week | 0–6 or names, depending on cron implementation |

### Common examples

| Cron Expression | Meaning |
|---|---|
| `0 * * * *` | Every hour at minute 0 |
| `0 9 * * *` | Every day at 9:00 AM |
| `30 8 * * *` | Every day at 8:30 AM |
| `0 0 * * *` | Every day at midnight |
| `0 9 * * 1` | Every Monday at 9:00 AM |
| `0 0 1 * *` | First day of every month at midnight |
| `*/15 * * * *` | Every 15 minutes |

### Example

```python
with DAG(
    dag_id="morning_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="30 8 * * *",
    catchup=False,
) as dag:
    ...
```

This schedules the DAG for 8:30 AM according to the DAG's timezone.

**Important:** A cron schedule defines the intended schedule, not the exact task execution time. Scheduler delays, executor capacity, and task dependencies can affect actual execution.

---

## 5. Understanding `start_date`

`start_date` defines the beginning of the DAG's scheduling timeline.

Example:

```python
start_date=datetime(2026, 1, 1)
```

This establishes January 1, 2026, as the starting point for the DAG's schedule.

### Important points

- `start_date` is not necessarily the actual time when a task executes.
- It helps determine the first eligible data interval.
- For scheduled DAGs, the first run generally becomes eligible after its data interval ends.
- Use a fixed start date instead of dynamically calculating `datetime.now()`.

### Why avoid `datetime.now()`?

Incorrect:

```python
start_date=datetime.now()
```

This can cause the effective start date to change whenever the DAG is parsed, creating confusing scheduling behavior.

Better:

```python
start_date=datetime(2026, 1, 1)
```

For production, choose a deliberate, fixed date and an appropriate timezone.

---

## 6. Logical Date and Data Interval

These concepts are especially important in Airflow interviews.

### Logical Date

The logical date identifies the start of a DAG Run's data interval in common time-based scheduling.

It is not necessarily the actual execution time.

### Data Interval

The data interval represents the period of data that a scheduled DAG Run is intended to process.

### Example: Daily schedule

Suppose a DAG runs daily.

```text
Data interval:
October 31, 2026, 00:00
             |
             v
November 1, 2026, 00:00

DAG Run becomes eligible:
After the interval ends
```

The run may execute after midnight on November 1, depending on scheduler and executor availability.

The run's logical date generally identifies the beginning of that interval.

### Why does this matter?

Suppose a daily pipeline processes orders.

Instead of querying data using the current execution time, it should use the intended data interval.

Conceptually:

```sql
SELECT *
FROM orders
WHERE order_date >= '2026-10-31 00:00:00'
  AND order_date <  '2026-11-01 00:00:00';
```

This ensures that the pipeline processes the intended period.

---

## 7. Timezones in Airflow

Timezone configuration determines how schedules are interpreted.

For example, a pipeline may need to run at 9:00 AM India Standard Time.

Use a timezone-aware start date.

```python
import pendulum

start_date = pendulum.datetime(
    2026, 1, 1,
    tz="Asia/Kolkata"
)
```

Then:

```python
with DAG(
    dag_id="india_morning_pipeline",
    start_date=start_date,
    schedule="0 9 * * *",
    catchup=False,
) as dag:
    ...
```

This defines a daily 9:00 AM schedule in the Asia/Kolkata timezone.

### Important points

- Airflow stores and processes many timestamps internally in UTC.
- DAG timezone affects how schedules and data intervals are interpreted.
- Daylight-saving transitions can affect schedules in regions that observe daylight saving time.
- India does not observe daylight saving time.

---

## 8. What Is Catchup?

Catchup determines whether Airflow should create DAG Runs for eligible historical schedule intervals that were not previously created.

Example:

A daily DAG has a start date of January 1, but it was not active for several days.

When enabled, catchup can create the missing eligible DAG Runs.

### `catchup=True`

```python
with DAG(
    dag_id="daily_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=True,
) as dag:
    ...
```

Airflow can create missing scheduled runs from the start date, subject to the DAG's scheduling and run limits.

### `catchup=False`

```python
with DAG(
    dag_id="daily_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:
    ...
```

Airflow does not automatically create the entire missed historical backlog.

### Comparison

| Catchup True | Catchup False |
|---|---|
| Creates eligible missed scheduled runs | Does not create the full missed backlog |
| Useful for historical processing | Useful for regular forward processing |
| Can create many DAG Runs | Avoids automatic backlog creation |

**Important:** Catchup is not the same as backfill. Explicit backfills can be performed even when `catchup=False`.

---

## 9. What Are Retries?

Retries allow Airflow to attempt a failed task again.

They are useful when failures are temporary.

Examples:

- Temporary network problems.
- Database connection interruptions.
- API rate limiting.
- Temporary external service unavailability.

### Example

```python
from datetime import timedelta

from airflow.operators.python import PythonOperator

extract = PythonOperator(
    task_id="extract_data",
    python_callable=extract_data,
    retries=3,
    retry_delay=timedelta(minutes=5),
)
```

Here:

- `retries=3` allows up to three retries after the initial attempt.
- `retry_delay` defines the waiting time between attempts.

### Example execution

```text
Attempt 1 → FAILED
              |
              v
         Wait 5 minutes
              |
              v
Attempt 2 → FAILED
              |
              v
         Wait 5 minutes
              |
              v
Attempt 3 → SUCCESS
```

With three retries, there can be up to four total attempts:

- One initial attempt.
- Three retry attempts.

---

## 10. Retry Configuration

Airflow provides several settings to control retry behavior.

| Parameter | Purpose |
|---|---|
| `retries` | Maximum number of retries |
| `retry_delay` | Delay between retry attempts |
| `retry_exponential_backoff` | Increases retry delay exponentially |
| `max_retry_delay` | Maximum retry delay when exponential backoff is enabled |
| `execution_timeout` | Maximum allowed duration of a task execution |
| `retry_timeout` | Not a standard BaseOperator retry parameter |

### Example

```python
from datetime import timedelta

extract = PythonOperator(
    task_id="extract_data",
    python_callable=extract_data,
    retries=3,
    retry_delay=timedelta(minutes=2),
    retry_exponential_backoff=True,
    max_retry_delay=timedelta(minutes=20),
    execution_timeout=timedelta(minutes=30),
)
```

### Exponential backoff

Instead of retrying at a fixed interval, Airflow can increase the delay between attempts.

Conceptually:

```text
Attempt 1 → Failure
       |
       v
   Wait 2 minutes
       |
       v
Attempt 2 → Failure
       |
       v
   Wait longer
       |
       v
Attempt 3 → Failure
       |
       v
   Wait longer, up to configured maximum
```

The exact delay is calculated by Airflow and can include jitter. It is not always a simple exact doubling of the previous delay.

### Why use exponential backoff?

It helps avoid repeatedly hitting an overloaded external service.

---

## 11. When Should Retries Be Used?

Retries are most useful for temporary failures.

| Failure | Retry Appropriate? |
|---|---|
| Temporary network interruption | Yes |
| Temporary API unavailability | Yes |
| Temporary database connection issue | Yes |
| API throttling | Often, with appropriate backoff |
| Invalid SQL syntax | Usually no |
| Missing Python package | No |
| Incorrect credentials | Usually no |
| Invalid source data | Usually no |
| Permanent permission error | Usually no |

### Important point

Retries do not fix the root cause of a permanent failure.

For example, retrying an incorrect SQL query three times will not make the query valid.

---

## 12. What Is `execution_timeout`?

`execution_timeout` limits how long an individual task execution is allowed to run.

Example:

```python
from datetime import timedelta

extract = PythonOperator(
    task_id="extract_data",
    python_callable=extract_data,
    execution_timeout=timedelta(minutes=30),
)
```

If the task exceeds the configured limit, Airflow raises a timeout exception and the task is marked failed. Retry behavior may then apply if retries are configured.

### Why use it?

- Prevent tasks from running indefinitely.
- Detect slow external operations.
- Protect worker capacity.
- Improve pipeline reliability.

### Important distinction

| Setting | Meaning |
|---|---|
| `execution_timeout` | Limits an individual task execution |
| `retry_delay` | Waiting period between retry attempts |
| `retries` | Maximum number of retries |
| Sensor `timeout` | Limits how long a sensor waits for its condition |

---

## 13. DAG-Level and Task-Level Defaults

You can define common retry settings for a DAG using `default_args`.

### Example

```python
from datetime import datetime, timedelta

from airflow import DAG
from airflow.operators.python import PythonOperator


default_args = {
    "retries": 2,
    "retry_delay": timedelta(minutes=5),
    "execution_timeout": timedelta(minutes=30),
}


with DAG(
    dag_id="pipeline_with_defaults",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
    default_args=default_args,
) as dag:

    extract = PythonOperator(
        task_id="extract_data",
        python_callable=extract_data,
    )

    transform = PythonOperator(
        task_id="transform_data",
        python_callable=transform_data,
        retries=4,
    )

    extract >> transform
```

Here:

- Both tasks inherit the default retry settings unless overridden.
- The transform task overrides the default retry count with four retries.

### Why use default arguments?

- Avoid repeating configuration.
- Maintain consistent retry behavior.
- Make DAG definitions easier to manage.

---

## 14. Concurrency and Parallel Execution

Scheduling determines when DAG Runs are created, while concurrency settings control how much work can execute at the same time.

Important settings include:

| Setting | Purpose |
|---|---|
| `max_active_runs` | Limits active DAG Runs for a DAG |
| `max_active_tasks` | Limits concurrently running task instances across active runs of a DAG |
| `max_active_tis_per_dag` | Limits concurrent instances of a particular task across DAG Runs |
| `max_active_tis_per_dagrun` | Limits concurrent instances of a particular task within a DAG Run |
| Pools | Limit concurrent use of shared resources |
| Executor capacity | Determines available execution capacity |

The exact settings available and their behavior can vary by Airflow version and executor.

### Example

```python
with DAG(
    dag_id="controlled_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
    max_active_runs=2,
    max_active_tasks=5,
) as dag:
    ...
```

This limits the DAG to two active DAG Runs and five concurrently running task instances across those active runs.

### Why is concurrency control important?

Suppose several tasks access the same production database.

Without limits, they may overload the database.

You can use pools and task concurrency limits to control the workload.

---

## 15. Airflow Pools

A pool is a mechanism for limiting concurrent task execution against a shared resource.

Example:

Suppose a production database can safely handle only three simultaneous extraction tasks.

Create a pool:

```text
Pool name: database_pool
Slots: 3
```

Assign tasks to it:

```python
extract = PythonOperator(
    task_id="extract_data",
    python_callable=extract_data,
    pool="database_pool",
    pool_slots=1,
)
```

### How it works

If three tasks are already using all three slots, another task assigned to that pool must wait until capacity becomes available.

### When should pools be used?

- Limit database connections.
- Control API request concurrency.
- Protect external services.
- Limit expensive processing workloads.

**Important:** Pools control tasks assigned to that pool. They do not automatically limit every task in the Airflow environment.

---

## 16. Scheduling vs Retries vs Backfill

These concepts are frequently confused.

| Concept | Purpose |
|---|---|
| Scheduling | Determines when DAG Runs should be created |
| Retry | Attempts a failed task again |
| Catchup | Creates eligible missed scheduled DAG Runs |
| Backfill | Creates DAG Runs for selected historical intervals |
| Rerun | Executes a task or DAG Run again, depending on the operation |

### Example

Suppose a daily orders pipeline failed on October 20.

- **Retry:** Attempts the failed task again in its existing DAG Run.
- **Backfill:** Creates or runs the historical interval for October 20, depending on the existing run and backfill options.
- **Catchup:** Controls automatic creation of eligible missed scheduled intervals.
- **Scheduling:** Creates future DAG Runs according to the configured schedule.

**Interview point:** Retry is about recovering a failed task. Backfill is about processing historical data intervals.

---

## 17. Safe Retries and Idempotency

Retries can cause duplicate records if a task partially completes before failing.

### Example problem

A task inserts 1,000 orders into a database.

The insert succeeds, but the task fails before Airflow records success.

When the task retries, it may insert the same 1,000 orders again.

This creates duplicates.

### Safer approaches

- Use `MERGE` or upsert operations.
- Use deterministic business keys.
- Replace a specific data partition when appropriate.
- Use database transactions where supported.
- Validate target data before marking processing complete.
- Design tasks so repeated execution has the same final effect.

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

Before executing this MERGE, ensure the source has at most one applicable row per target key.

### Important distinction

Airflow retries do not guarantee exactly-once data processing.

Exactly-once effective results depend on the source, processing logic, sink, transaction handling, and idempotency strategy.

---

## 18. Practical Example: Daily Orders Pipeline

### Requirement

Build a daily pipeline that:

1. Runs every day at 9:00 AM.
2. Extracts orders.
3. Retries temporary failures.
4. Stops tasks that exceed 30 minutes.
5. Transforms the data.
6. Loads the target table.

### DAG example

```python
from datetime import datetime, timedelta

import pendulum

from airflow import DAG
from airflow.operators.python import PythonOperator


def extract_orders():
    print("Extracting orders")


def transform_orders():
    print("Transforming orders")


def load_orders():
    print("Loading orders")


default_args = {
    "retries": 3,
    "retry_delay": timedelta(minutes=5),
    "retry_exponential_backoff": True,
    "max_retry_delay": timedelta(minutes=20),
    "execution_timeout": timedelta(minutes=30),
}


with DAG(
    dag_id="daily_orders_pipeline",
    start_date=pendulum.datetime(
        2026, 1, 1,
        tz="Asia/Kolkata"
    ),
    schedule="0 9 * * *",
    catchup=False,
    max_active_runs=1,
    default_args=default_args,
) as dag:

    extract = PythonOperator(
        task_id="extract_orders",
        python_callable=extract_orders,
    )

    transform = PythonOperator(
        task_id="transform_orders",
        python_callable=transform_orders,
    )

    load = PythonOperator(
        task_id="load_orders",
        python_callable=load_orders,
    )

    extract >> transform >> load
```

### Explanation

- `schedule="0 9 * * *"` schedules the DAG daily at 9:00 AM.
- `Asia/Kolkata` sets the scheduling timezone.
- `catchup=False` avoids automatic creation of the full historical backlog.
- `retries=3` allows three retries after the initial attempt.
- `retry_delay` sets the initial retry delay.
- `retry_exponential_backoff=True` enables increasing retry delays.
- `max_retry_delay` caps the retry delay.
- `execution_timeout` limits each task execution.
- `max_active_runs=1` prevents multiple active DAG Runs for this DAG.

The Python functions are placeholders. A production pipeline would include actual extraction, transformation, and loading logic.

---

## 19. Common Mistakes to Avoid

1. Assuming the schedule is the exact task execution time.
2. Confusing logical date with actual execution time.
3. Using `datetime.now()` as a dynamic DAG start date.
4. Forgetting to configure the correct timezone.
5. Confusing retries with backfills.
6. Setting too many retries for permanent failures.
7. Increasing timeouts without investigating slow execution.
8. Forgetting that retries can repeat partial writes.
9. Ignoring idempotency.
10. Assuming `catchup=False` disables explicit backfills.
11. Ignoring DAG and task concurrency limits.
12. Assuming pools control all tasks automatically.
13. Setting a very high `max_active_runs` without checking downstream system capacity.
14. Assuming exponential backoff always doubles the delay exactly.

---

## 20. Interview Questions and Answers

### Q1. What is scheduling in Airflow?

Scheduling determines when Airflow should create DAG Runs based on the DAG's configured schedule.

### Q2. What is the difference between `@daily` and a cron expression?

`@daily` is a convenient preset for a daily schedule. A cron expression provides more precise control over scheduling.

### Q3. What is `start_date`?

It defines the beginning of a DAG's scheduling timeline and helps determine the eligible data intervals.

### Q4. What is a logical date?

It identifies the logical start of a DAG Run's data interval in common time-based scheduling. It is not necessarily the actual execution time.

### Q5. What is a data interval?

It represents the period of data that a scheduled DAG Run is intended to process.

### Q6. What is catchup?

Catchup controls whether Airflow creates eligible missed scheduled DAG Runs from the DAG's scheduling history.

### Q7. What is the difference between catchup and backfill?

Catchup automatically creates eligible missed scheduled runs. Backfill explicitly creates or processes DAG Runs for selected historical intervals.

### Q8. What are retries?

Retries allow Airflow to attempt a failed task again.

### Q9. What does `retries=3` mean?

It allows up to three retry attempts after the initial task attempt, giving up to four total attempts.

### Q10. What is `retry_delay`?

It specifies the waiting time between retry attempts.

### Q11. What is exponential backoff?

It increases the delay between retry attempts to avoid repeatedly overwhelming a temporarily unavailable service.

### Q12. What is `execution_timeout`?

It defines the maximum duration allowed for an individual task execution.

### Q13. What is the difference between `retry_delay` and `execution_timeout`?

`retry_delay` controls how long Airflow waits before another attempt. `execution_timeout` limits how long an individual attempt can run.

### Q14. Why is idempotency important when using retries?

A task may partially write data before failing. Idempotency helps prevent duplicate or inconsistent results when the task runs again.

### Q15. What is `max_active_runs`?

It limits the number of active DAG Runs for a particular DAG.

### Q16. What is the purpose of Airflow pools?

Pools limit the number of concurrent tasks using shared resources, such as databases or external APIs.

### Q17. How would you handle a temporary API failure?

I would configure appropriate retries and exponential backoff, use a reasonable timeout, and ensure that repeated requests are safe.

### Q18. How would you prevent multiple DAG Runs from overloading a database?

I would control DAG-level concurrency and use an Airflow pool for tasks that access the database.

### Q19. Does a successful task mean the data is correct?

No. Task success means the task completed without reporting an execution failure. Data quality and business validation must be checked separately.

### Q20. How would you design a reliable daily data pipeline?

I would define the correct schedule and timezone, use data intervals for processing, configure appropriate retries and timeouts, limit concurrency, make writes idempotent, and monitor both pipeline execution and data quality.

---

## 21. Quick Revision

| Topic | Key Point |
|---|---|
| Schedule | Determines when DAG Runs are created |
| `@daily` | Daily schedule preset |
| Cron | Custom scheduling expression |
| `start_date` | Beginning of scheduling timeline |
| Logical date | Identifies the data interval's logical start |
| Data interval | Period of data to process |
| Timezone | Controls schedule interpretation |
| Catchup | Creates eligible missed scheduled runs |
| Retry | Attempts a failed task again |
| `retries` | Maximum retry attempts |
| `retry_delay` | Delay between attempts |
| Exponential backoff | Increases retry delays |
| `execution_timeout` | Maximum duration of one task attempt |
| `max_active_runs` | Limits active DAG Runs |
| `max_active_tasks` | Limits concurrently running task instances across active DAG Runs |
| Pool | Controls concurrent use of shared resources |
| Idempotency | Makes repeated execution safe |

### Five points to remember

1. Scheduling determines when DAG Runs become eligible.
2. Logical date and data interval are not the same as actual execution time.
3. Retries help recover from temporary task failures.
4. Timeouts and concurrency limits protect pipeline resources.
5. Idempotency is essential for safe retries and reruns.

---

## 22. Self-Assessment

Try answering these questions without looking at the notes:

1. Explain how scheduling works in Airflow.
2. What is the difference between `@daily` and `0 9 * * *`?
3. Explain `start_date`, logical date, and data interval.
4. Why should you avoid `datetime.now()` for a DAG's `start_date`?
5. How do timezones affect DAG scheduling?
6. What is the difference between catchup and backfill?
7. How does retry work in Airflow?
8. What is the difference between `retries` and `retry_delay`?
9. Explain exponential backoff.
10. What is the difference between `execution_timeout` and sensor `timeout`?
11. How do `max_active_runs` and `max_active_tasks` differ?
12. What is an Airflow pool?
13. Why is idempotency important for data pipelines?
14. Design a daily orders pipeline with retries, timeouts, and concurrency controls.

