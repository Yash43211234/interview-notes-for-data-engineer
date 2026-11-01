

# Backfills and Catchup in Apache Airflow

## 1. Overview

In data engineering, pipelines sometimes need to process historical data again.

For example:

- A daily pipeline was paused for five days.
- A bug caused incorrect results for the previous month.
- A new business rule requires recalculating historical records.
- Some scheduled pipeline runs failed and need to be processed again.

Apache Airflow provides two important concepts for handling historical workflow execution:

- **Catchup:** Automatically creates scheduled DAG runs for past data intervals that have not been run.
- **Backfill:** Creates DAG runs for a specified historical date or data interval range.

Both help process historical data, but they serve different purposes.

---

## 2. What Is Catchup?

Catchup is an Airflow DAG scheduling feature that determines whether the scheduler should create DAG runs for past scheduled intervals that have not yet been processed.

Consider a daily DAG:

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.empty import EmptyOperator

with DAG(
    dag_id="daily_orders",
    start_date=datetime(2026, 10, 1),
    schedule="@daily",
    catchup=True,
) as dag:

    process_orders = EmptyOperator(
        task_id="process_orders"
    )
```

Suppose:

- The DAG starts on October 1.
- The DAG is enabled on October 6.
- The schedule is daily.
- Catchup is enabled.

Airflow can create DAG runs for the missed scheduled intervals.

Conceptually:

```text
October 1  -> DAG Run
October 2  -> DAG Run
October 3  -> DAG Run
October 4  -> DAG Run
October 5  -> DAG Run
October 6  -> Current scheduled interval
```

The exact run creation depends on the DAG's timetable, start date, current time, and existing DAG runs.

### Important point

Catchup does not mean rerunning every task that failed.

It means creating missing scheduled DAG runs for past data intervals.

---

## 3. What Is `catchup`?

The `catchup` parameter controls whether Airflow creates DAG runs for missed historical intervals.

### Catchup enabled

```python
with DAG(
    dag_id="orders_pipeline",
    start_date=datetime(2026, 10, 1),
    schedule="@daily",
    catchup=True,
) as dag:
    ...
```

Airflow can create runs for eligible past intervals that have not already been created.

### Catchup disabled

```python
with DAG(
    dag_id="orders_pipeline",
    start_date=datetime(2026, 10, 1),
    schedule="@daily",
    catchup=False,
) as dag:
    ...
```

Airflow does not create the entire backlog of missed scheduled intervals. It schedules the latest eligible interval according to the timetable and current time.

### Comparison

| Feature | `catchup=True` | `catchup=False` |
|---|---|---|
| Historical intervals | Creates missing eligible runs | Does not create the full missed backlog |
| Typical use | Historical processing | Regular forward processing |
| Risk | Large number of runs | Historical intervals may remain unprocessed |
| Common scenario | Backlog recovery | Daily production pipeline |

**Interview point:** Catchup is a scheduling behavior. It is not a command to rerun failed tasks or automatically repair historical data.

---

## 4. What Is Backfill?

Backfill is the process of creating DAG runs for a specified historical date or data interval range.

For example, suppose a company needs to recalculate order data for October 1 through October 10.

Instead of waiting for the regular schedule, you can create historical DAG runs for that range.

```text
Historical Date Range
October 1 - October 10
          |
          v
     Airflow Backfill
          |
   ---------------------
   |   |   |   |       |
   v   v   v   v       v
  Oct 1 ...       ... Oct 10
   |                 |
   v                 v
Process Historical Data
```

### Common reasons for backfill

- Correcting historical data.
- Reprocessing failed periods.
- Applying new business logic.
- Rebuilding warehouse tables.
- Recovering data after a source-system outage.
- Loading historical data into a new platform.

### Example

A transformation bug caused incorrect revenue calculations for October 1–5.

After fixing the code, the data engineering team backfills those intervals to regenerate correct results.

---

## 5. Catchup vs Backfill

These concepts are related but not identical.

| Feature | Catchup | Backfill |
|---|---|---|
| Purpose | Create missing scheduled runs | Process a specified historical range |
| Initiation | DAG scheduling behavior | Explicit backfill operation |
| Date range | Determined by schedule and eligible missing intervals | Selected by the user |
| Main use | Handle missed scheduled intervals | Reprocess historical data |
| Control | `catchup` setting | Backfill command or API |
| Can create historical DAG runs? | Yes | Yes |

### Simple way to remember

- **Catchup:** Airflow notices missing scheduled intervals and creates eligible DAG runs.
- **Backfill:** You explicitly request historical DAG runs for a particular range.

---

## 6. Understanding `start_date`, Schedule, and Logical Date

To understand catchup and backfill, you must understand how Airflow identifies scheduled runs.

### `start_date`

The `start_date` defines the beginning of the DAG's scheduling timeline.

It does not necessarily mean that the first DAG run starts executing at that exact timestamp.

Example:

```python
start_date=datetime(2026, 10, 1)
schedule="@daily"
```

For a daily schedule, the first scheduled run generally becomes eligible after its associated daily data interval has completed.

### Logical Date

The logical date identifies the scheduled time associated with a DAG run.

It is not necessarily the actual time when the task starts executing.

### Data Interval

The data interval represents the period of data that a scheduled DAG run is intended to process.

For a daily DAG, a run might process:

```text
Data Interval:
October 1, 00:00
       to
October 2, 00:00
```

The run is generally scheduled after the interval ends.

### Example

Suppose:

```python
start_date=datetime(2026, 10, 1)
schedule="@daily"
```

A daily DAG run may have:

| Logical date | Data interval |
|---|---|
| October 1 | Oct 1 to Oct 2 |
| October 2 | Oct 2 to Oct 3 |
| October 3 | Oct 3 to Oct 4 |

The actual execution can happen later depending on scheduler availability and workload.

**Important:** The exact boundaries depend on the DAG's timetable and timezone configuration. Do not assume logical date is the task's actual execution timestamp.

---

## 7. How Catchup Works Internally

Consider a daily DAG with:

```python
with DAG(
    dag_id="daily_sales_pipeline",
    start_date=datetime(2026, 10, 1),
    schedule="@daily",
    catchup=True,
) as dag:
    ...
```

Suppose the DAG is enabled on October 6 and no historical runs exist.

### Execution flow

1. The DAG Processor processes the DAG definition.
2. The Scheduler identifies the DAG's timetable and start date.
3. The Scheduler determines which scheduled intervals are eligible.
4. It checks existing DAG runs.
5. It creates missing eligible DAG runs when catchup is enabled.
6. Eligible task instances are scheduled.
7. The Executor arranges task execution.
8. Task states are stored in the metadata database.

### Visual flow

```text
       DAG Definition
             |
             v
       Scheduler
             |
             v
      Check Schedule
             |
             v
   Identify Missing Intervals
             |
             v
       catchup=True?
          /     \
        Yes      No
         |        |
         v        v
   Create Past   Schedule Latest
   Eligible      Eligible Run
   DAG Runs
         |
         v
   Schedule Tasks
```

Catchup is subject to Airflow's scheduling limits, existing DAG runs, timetable behavior, and concurrency settings.

---

## 8. Performing a Backfill

The exact backfill interface depends on the Airflow version.

In modern Airflow versions, the CLI supports the `airflow backfill create` command.

Example:

```bash
airflow backfill create \
    --dag-id daily_orders \
    --from-date 2026-10-01 \
    --to-date 2026-10-05
```

This requests backfill runs for the selected date range.

The date range is interpreted according to the DAG's timetable and Airflow's backfill semantics.

Check the installed Airflow version's CLI help before using additional options:

```bash
airflow backfill create --help
```

### Important

Backfill is not simply a command to execute a task five times.

It creates historical DAG runs, each associated with its own logical date and data interval.

---

## 9. Backfill Configuration and Concurrency

Backfilling many historical intervals can create a large workload.

Suppose you need to process one year of daily data.

That may involve hundreds of DAG runs.

If every run is allowed to execute concurrently, it can overload:

- Source databases.
- ADLS Gen2.
- Databricks clusters.
- Data warehouses.
- API services.

### Control concurrency

Airflow provides several controls, including:

- DAG-level `max_active_runs`.
- Task-level `max_active_tis_per_dag`.
- Pools.
- Executor capacity.
- Backfill-specific options supported by the installed version.

Example:

```python
with DAG(
    dag_id="historical_orders",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
    max_active_runs=2,
) as dag:
    ...
```

This limits the number of active runs of this DAG according to the configured DAG limit.

### Pools

Pools limit the number of tasks that can use a constrained resource simultaneously.

Example:

```python
from airflow.operators.empty import EmptyOperator

process_orders = EmptyOperator(
    task_id="process_orders",
    pool="database_pool",
)
```

The pool must be created and configured in Airflow.

If a pool has five slots, no more than five pool slots can be occupied simultaneously.

A task may consume more than one slot if configured to do so.

### Best practices

- Limit concurrent historical runs.
- Protect production databases.
- Use pools for shared external resources.
- Monitor worker and executor capacity.
- Start with a smaller date range.
- Increase concurrency only after validating system capacity.

---

## 10. Backfill and Idempotency

Idempotency is especially important when rerunning historical data.

An operation is idempotent when repeating it under the defined conditions produces the same intended final result.

### Example: Unsafe append

Suppose a daily pipeline loads orders into a warehouse.

```python
df.write.mode("append").saveAsTable("orders")
```

If the same historical interval is backfilled twice, duplicate records may be inserted unless the pipeline has additional duplicate protection.

### Safer approaches

Depending on the target system and business requirements, use:

- Partition overwrite.
- `MERGE` operations.
- Unique business keys.
- Deduplication.
- Transactional writes.
- Explicit replacement of the affected date range.

Example using a Delta table:

```sql
MERGE INTO target_orders AS target
USING source_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN UPDATE SET
    target.amount = source.amount,
    target.order_date = source.order_date

WHEN NOT MATCHED THEN INSERT (
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

This example assumes `order_id` uniquely identifies an order and the source contains at most one matching row per key.

### Important point

Airflow can rerun a task or DAG run, but it cannot automatically guarantee that the external data operation is idempotent.

The task implementation and destination system must support safe reruns.

---

## 11. Backfill and Data Quality

Historical reprocessing should include data quality validation.

For example, after backfilling sales data, validate:

- Record counts.
- Duplicate business keys.
- NULL values in required columns.
- Revenue totals.
- Date-range completeness.
- Reconciliation with source data.

Example SQL:

```sql
SELECT
    order_date,
    COUNT(*) AS total_orders,
    SUM(amount) AS total_revenue
FROM orders
WHERE order_date BETWEEN '2026-10-01' AND '2026-10-05'
GROUP BY order_date
ORDER BY order_date;
```

Compare the results with source-system totals or trusted historical results.

### Recommended workflow

```text
Identify Incorrect Data
          |
          v
Fix Transformation Logic
          |
          v
Select Historical Range
          |
          v
Run Backfill
          |
          v
Validate Historical Results
          |
          v
Reconcile with Source
          |
          v
Confirm Data Correctness
```

---

## 12. Handling Failed DAG Runs

Catchup and backfill are not the only ways to recover failed pipelines.

Different recovery methods solve different problems.

| Situation | Appropriate action |
|---|---|
| Scheduled interval was never created | Catchup or explicit backfill |
| DAG run exists but a task failed | Clear and rerun the failed task or use an appropriate retry |
| Historical data needs recalculation | Backfill |
| Entire historical range is missing | Backfill or catchup, depending on the situation |
| Source data was corrected | Reprocess affected intervals |
| Task failed because of temporary network issue | Retry after checking the failure |

### Clearing a task

Airflow allows users to clear task instances through the UI or supported CLI/API workflows.

Clearing a task instance makes it eligible for execution again, subject to dependencies and configuration.

**Important:** Clearing a task is different from creating a new historical DAG run.

- Clearing resets an existing task instance for another attempt.
- Backfill creates DAG runs for requested historical intervals.

---

## 13. Backfill and Catchup in a Data Engineering Scenario

### Scenario

A company processes daily customer transactions.

The pipeline:

1. Extracts transactions from a source database.
2. Stores raw data in ADLS Gen2.
3. Transforms data using Databricks.
4. Loads curated data into a warehouse.
5. Validates daily totals.

Suppose the pipeline failed to run for October 1–5 because the source database was unavailable.

### Option A: Catchup

If the missing scheduled DAG runs have not been created and catchup is enabled, Airflow can create eligible runs for those intervals.

### Option B: Backfill

If the team explicitly wants to reprocess October 1–5, it can request a backfill for that date range.

### Recommended approach

1. Confirm the source data is available for the missing dates.
2. Check whether historical DAG runs already exist.
3. Identify whether the problem is missing runs or failed tasks.
4. Confirm the transformation logic is correct.
5. Verify the destination supports safe reruns.
6. Select the required historical intervals.
7. Control backfill concurrency.
8. Execute the backfill.
9. Validate the resulting data.
10. Monitor the normal scheduled pipeline afterward.

---

## 14. Common Mistakes to Avoid

- Thinking catchup reruns failed tasks automatically.
- Thinking backfill and catchup are identical.
- Assuming `start_date` is the actual task execution time.
- Confusing logical date with wall-clock execution time.
- Running a large backfill without concurrency controls.
- Using append-only writes without duplicate protection.
- Assuming Airflow guarantees idempotency.
- Forgetting to validate historical results.
- Creating new backfill runs when clearing an existing failed task would be sufficient.
- Assuming `catchup=False` means historical DAG runs can never be created manually.
- Assuming every Airflow version has identical backfill CLI options.
- Ignoring timezone and timetable behavior when selecting date ranges.

---

## 15. Interview Questions and Answers

### Q1. What is catchup in Airflow?

Catchup is a scheduling feature that allows Airflow to create missing DAG runs for eligible past scheduled intervals.

### Q2. What is backfill?

Backfill is the process of creating DAG runs for a user-selected historical date or data interval range.

### Q3. What is the difference between catchup and backfill?

Catchup is controlled by the DAG's scheduling configuration and handles missing scheduled intervals. Backfill is an explicit request to process a specified historical range.

### Q4. What does `catchup=False` mean?

It prevents Airflow from creating the full backlog of missed scheduled intervals. Airflow schedules the latest eligible interval according to the timetable.

### Q5. Does catchup rerun failed tasks?

No. Catchup creates missing scheduled DAG runs. Failed tasks are handled through retries, clearing, or other recovery operations.

### Q6. When would you use backfill?

I would use backfill when historical data needs to be reprocessed, such as after fixing transformation logic or recovering data for a specific date range.

### Q7. What is the difference between clearing a task and backfilling a DAG?

Clearing makes an existing task instance eligible to run again. Backfilling creates DAG runs for specified historical intervals.

### Q8. Why is idempotency important during backfill?

Historical intervals may be processed more than once. Idempotent writes help prevent duplicate or inconsistent results.

### Q9. How can you control backfill concurrency?

Use DAG-level limits, task-level limits, pools, executor capacity, and supported backfill options.

### Q10. What is a logical date?

It is the logical timestamp associated with a DAG run and helps identify the scheduled interval being processed. It is not necessarily the actual execution time.

### Q11. What is a data interval?

It is the time range of data that a scheduled DAG run is intended to process.

### Q12. What problems can occur during a large backfill?

It can overload source databases, storage systems, compute clusters, or warehouses. It can also create duplicate data if writes are not safe to repeat.

### Q13. How would you backfill data after fixing a transformation bug?

I would identify the affected intervals, confirm source availability, verify idempotent target writes, execute a controlled backfill, and validate the corrected historical results.

### Q14. Does `catchup=False` prevent manual backfills?

No. It controls automatic catchup behavior. Historical runs can still be explicitly requested through supported backfill mechanisms.

### Q15. How do you decide between catchup and backfill?

I use catchup when the scheduler should create missing scheduled intervals. I use backfill when I need explicit control over a historical range or need to reprocess data.

---

## 16. Quick Revision

| Concept | Key Point |
|---|---|
| Catchup | Creates missing eligible historical scheduled DAG runs |
| `catchup=True` | Allows automatic catchup |
| `catchup=False` | Prevents creation of the full missed backlog |
| Backfill | Explicit historical DAG-run creation |
| `start_date` | Beginning of the scheduling timeline |
| Logical Date | Logical timestamp associated with a DAG run |
| Data Interval | Period of data processed by a DAG run |
| DAG Run | One execution of a DAG |
| Task Instance | One task execution within a DAG run |
| Idempotency | Safe repeated execution under defined conditions |
| Pool | Limits concurrent use of a shared resource |
| Clearing | Makes an existing task instance eligible to run again |

### Five points to remember

1. Catchup handles missing scheduled intervals.
2. Backfill processes a specifically requested historical range.
3. `catchup=False` does not disable explicit backfills.
4. Idempotency is essential for safe historical reprocessing.
5. Control concurrency and validate data during large backfills.

---

## 17. Self-Assessment

Try answering these questions without looking at the notes:

1. Explain catchup in Apache Airflow.
2. What is the difference between catchup and backfill?
3. What happens when `catchup=True`?
4. What happens when `catchup=False`?
5. Explain `start_date`, logical date, and data interval.
6. How would you reprocess five days of historical sales data?
7. What is the difference between clearing a task and backfilling a DAG?
8. Why is idempotency important during historical processing?
9. How would you prevent a large backfill from overloading Databricks or a source database?
10. Design a safe backfill process for a production data warehouse.

