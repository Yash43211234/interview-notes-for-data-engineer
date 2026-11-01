

# DAGs, Tasks, and Dependencies in Apache Airflow

## 1. Overview

Apache Airflow organizes workflows using **DAGs, tasks, and dependencies**.

These are the three fundamental concepts you need to understand before building Airflow pipelines.

- **DAG:** Defines the overall workflow.
- **Task:** Represents an individual unit of work.
- **Dependency:** Defines the execution order between tasks.

Example of a daily data engineering pipeline:

```text
Extract Data
     |
     v
Validate Data
     |
     v
Transform Data
     |
     v
Load Warehouse
     |
     v
Send Notification
```

Airflow uses these relationships to determine which tasks can run and when.

---

## 2. What Is a DAG?

DAG stands for **Directed Acyclic Graph**.

It is a representation of a workflow in which tasks are connected through directed dependencies without forming cycles.

### Understanding the term

| Term | Meaning |
|---|---|
| Directed | Dependencies have a defined direction |
| Acyclic | A task cannot eventually depend on itself |
| Graph | Tasks are nodes connected by edges |

### Example

```text
        Extract
           |
           v
        Validate
           |
      -------------
      |           |
      v           v
 Transform A  Transform B
      |           |
      -------------
           |
           v
          Load
```

In this workflow:

1. Extract runs first.
2. Validate runs after extraction.
3. Transform A and Transform B can run in parallel.
4. Load waits for both transformations to finish successfully under the default dependency behavior.

### Why must a DAG be acyclic?

A cycle would create an impossible dependency.

For example:

```text
Task A → Task B → Task C → Task A
```

Task A waits for Task C, Task C waits for Task B, and Task B waits for Task A.

No task can become ready to execute.

Airflow therefore requires DAG dependencies to be acyclic.

---

## 3. What Is a Task?

A task is an individual unit of work within a DAG.

Examples:

- Extract data from a database.
- Run a SQL query.
- Copy files to ADLS Gen2.
- Execute a Databricks notebook.
- Validate record counts.
- Send an email notification.

Each task has a unique `task_id` within its DAG.

### Example

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.empty import EmptyOperator

with DAG(
    dag_id="customer_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:

    extract = EmptyOperator(
        task_id="extract_data"
    )

    transform = EmptyOperator(
        task_id="transform_data"
    )

    load = EmptyOperator(
        task_id="load_data"
    )
```

This DAG contains three tasks:

- `extract_data`
- `transform_data`
- `load_data`

`EmptyOperator` is used to demonstrate task creation. It performs no actual data processing.

### Important points

- A task is defined inside a DAG.
- Each task has a unique ID within that DAG.
- A task can have upstream and downstream dependencies.
- A task instance represents a task's execution in a particular DAG run.

---

## 4. What Is a Task Instance?

A Task Instance is one execution of a task for a particular DAG Run.

Suppose a DAG runs daily:

```text
DAG: daily_orders_pipeline
```

It contains:

```text
extract_orders
transform_orders
load_orders
```

For the October 25 DAG Run, Airflow creates corresponding task instances.

```text
DAG Run: October 25

Task Instances:
    extract_orders
    transform_orders
    load_orders
```

The next day's DAG Run has separate task instances.

### DAG vs DAG Run vs Task vs Task Instance

| Concept | Meaning |
|---|---|
| DAG | Workflow definition |
| DAG Run | One execution of a DAG |
| Task | Individual unit of work in the DAG |
| Task Instance | Execution of a task within a specific DAG Run |

**Interview point:** A task is a definition. A task instance is an execution of that task.

---

## 5. Defining Dependencies

Dependencies define the execution order of tasks.

Airflow provides two commonly used dependency operators:

- `>>` for downstream dependencies.
- `<<` for upstream dependencies.

### 5.1 Using `>>`

```python
extract >> transform >> load
```

This means:

```text
Extract → Transform → Load
```

The transform task depends on extract, and load depends on transform.

### 5.2 Using `<<`

```python
load << transform << extract
```

This defines the same dependency relationship:

```text
Extract → Transform → Load
```

### 5.3 Using `set_downstream()`

```python
extract.set_downstream(transform)
transform.set_downstream(load)
```

This also creates the same dependency chain.

### 5.4 Using `set_upstream()`

```python
load.set_upstream(transform)
transform.set_upstream(extract)
```

This defines the same execution order.

### Comparison

| Method | Meaning |
|---|---|
| `A >> B` | B depends on A |
| `A << B` | A depends on B |
| `A.set_downstream(B)` | B depends on A |
| `A.set_upstream(B)` | A depends on B |

---

## 6. Multiple Dependencies

A task can depend on multiple upstream tasks or have multiple downstream tasks.

### Example 1: One task with multiple downstream tasks

```python
extract >> [transform_a, transform_b]
```

Graph:

```text
          Extract
          /     \
         v       v
   Transform A  Transform B
```

Both transformations can run independently after extraction succeeds.

### Example 2: Multiple upstream tasks

```python
[transform_a, transform_b] >> load
```

Graph:

```text
Transform A    Transform B
      \          /
       \        /
          Load
```

By default, `load` waits for both upstream tasks to succeed.

### Example 3: Complete workflow

```python
extract >> [transform_a, transform_b] >> load
```

Graph:

```text
             Extract
                |
         --------------
         |            |
         v            v
    Transform A  Transform B
         |            |
         --------------
                |
                v
               Load
```

### Important point

Airflow supports parallel task execution when dependencies allow it and sufficient execution capacity is available.

Creating parallel dependencies does not guarantee simultaneous execution. Executor capacity, DAG limits, pools, and other settings also matter.

---

## 7. Creating a Complete DAG

Let's create a simple customer data pipeline.

### Workflow

1. Start pipeline.
2. Extract customer data.
3. Validate extracted data.
4. Transform data.
5. Load data.
6. Finish pipeline.

### Python code

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.empty import EmptyOperator
from airflow.operators.python import PythonOperator


def extract_data():
    print("Extracting customer data")


def validate_data():
    print("Validating customer data")


def transform_data():
    print("Transforming customer data")


def load_data():
    print("Loading customer data")


with DAG(
    dag_id="customer_data_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
    tags=["data-engineering"],
) as dag:

    start = EmptyOperator(
        task_id="start"
    )

    extract = PythonOperator(
        task_id="extract_data",
        python_callable=extract_data,
    )

    validate = PythonOperator(
        task_id="validate_data",
        python_callable=validate_data,
    )

    transform = PythonOperator(
        task_id="transform_data",
        python_callable=transform_data,
    )

    load = PythonOperator(
        task_id="load_data",
        python_callable=load_data,
    )

    finish = EmptyOperator(
        task_id="finish"
    )

    start >> extract >> validate >> transform >> load >> finish
```

### Important observations

- `DAG` defines the workflow.
- `PythonOperator` executes a Python callable.
- `EmptyOperator` represents a task without business logic.
- `python_callable` specifies the Python function to execute.
- `>>` defines dependencies.
- `schedule="@daily"` schedules the workflow daily.
- `catchup=False` prevents automatic creation of the full missed backlog.

The print statements are only demonstrations. A production pipeline would contain real extraction, validation, transformation, and loading logic.

---

## 8. Parallel Task Execution

Airflow supports parallel execution when tasks do not depend on one another.

Consider this workflow:

```text
              Extract
                 |
                 v
             Validate
                 |
          -------------
          |           |
          v           v
      Customer     Orders
     Transform   Transform
          |           |
          -------------
                 |
                 v
             Load Data
```

The customer and orders transformations can run independently.

### Python example

```python
from airflow.operators.empty import EmptyOperator

extract = EmptyOperator(
    task_id="extract"
)

customer_transform = EmptyOperator(
    task_id="customer_transform"
)

orders_transform = EmptyOperator(
    task_id="orders_transform"
)

load = EmptyOperator(
    task_id="load"
)

extract >> [customer_transform, orders_transform]
[customer_transform, orders_transform] >> load
```

### Why use parallel execution?

- Reduces total workflow duration.
- Uses available compute resources efficiently.
- Allows independent data processing operations to run concurrently.

### Important consideration

Parallel tasks should not create uncontrolled load on shared systems.

Use pools and concurrency limits when tasks access the same database, API, or compute platform.

---

## 9. Task Dependencies vs Execution Order

A dependency determines when a task becomes eligible to run. It does not always guarantee that the upstream task must succeed.

This depends on the downstream task's `trigger_rule`.

The default trigger rule is:

```python
trigger_rule="all_success"
```

This means the task normally runs only when all its upstream tasks succeed.

### Example

```python
from airflow.operators.empty import EmptyOperator

extract = EmptyOperator(
    task_id="extract"
)

transform = EmptyOperator(
    task_id="transform"
)

extract >> transform
```

If `extract` fails, `transform` will normally not run.

It will generally be marked `upstream_failed` once its execution is evaluated.

---

## 10. Trigger Rules

Trigger rules control when a task is allowed to run based on the states of its upstream tasks.

Common trigger rules include:

| Trigger Rule | Meaning |
|---|---|
| `all_success` | All upstream tasks succeeded |
| `all_failed` | All upstream tasks failed |
| `all_done` | All upstream tasks reached a completed state |
| `all_skipped` | All upstream tasks were skipped |
| `one_success` | At least one upstream task succeeded |
| `one_failed` | At least one upstream task failed |
| `none_failed` | No upstream task failed or was marked upstream_failed |
| `none_failed_min_one_success` | No upstream task failed, and at least one succeeded |
| `none_skipped` | No upstream task was skipped |

The exact set of available trigger rules can vary by Airflow version.

### Example: Run notification after completion

Suppose a notification should run whether the pipeline succeeds or fails.

```python
from airflow.utils.trigger_rule import TriggerRule
from airflow.operators.empty import EmptyOperator

notify = EmptyOperator(
    task_id="notify",
    trigger_rule=TriggerRule.ALL_DONE,
)
```

`ALL_DONE` allows the notification task to run after all its upstream tasks reach terminal states, including success, failure, or skipped.

### Example: Run cleanup only after failure

```python
cleanup = EmptyOperator(
    task_id="cleanup",
    trigger_rule=TriggerRule.ONE_FAILED,
)
```

This task becomes eligible when at least one upstream task fails.

### Important distinction

Dependencies define task relationships. Trigger rules define how upstream task states affect downstream execution.

---

## 11. Branching in Airflow

Sometimes a pipeline needs to choose different execution paths based on a condition.

Airflow provides branching operators for this purpose.

Example:

```text
          Check Data
               |
         -------------
         |           |
         v           v
    Valid Data   Invalid Data
         |           |
         v           v
    Load Data    Send Alert
```

### Python example

```python
from airflow.operators.python import BranchPythonOperator
from airflow.operators.empty import EmptyOperator


def choose_branch():
    data_is_valid = True

    if data_is_valid:
        return "load_data"

    return "send_alert"


check_data = BranchPythonOperator(
    task_id="check_data",
    python_callable=choose_branch,
)

load_data = EmptyOperator(
    task_id="load_data"
)

send_alert = EmptyOperator(
    task_id="send_alert"
)

check_data >> [load_data, send_alert]
```

The branch function returns the task ID or task IDs to follow.

The unselected branch is marked skipped.

### Important point

If branches later join, choose the join task's trigger rule carefully. The default `all_success` may not be suitable when some upstream tasks are intentionally skipped.

A common option is:

```python
from airflow.utils.trigger_rule import TriggerRule

join = EmptyOperator(
    task_id="join",
    trigger_rule=TriggerRule.NONE_FAILED_MIN_ONE_SUCCESS,
)
```

This allows the join to run when at least one upstream branch succeeds and none fail.

---

## 12. TaskGroups

TaskGroups allow related tasks to be grouped together visually and logically in the Airflow UI.

They do not create a separate execution environment.

Example:

```python
from airflow.utils.task_group import TaskGroup
from airflow.operators.empty import EmptyOperator

with TaskGroup(group_id="customer_processing") as customer_group:

    extract_customer = EmptyOperator(
        task_id="extract_customer"
    )

    transform_customer = EmptyOperator(
        task_id="transform_customer"
    )

    load_customer = EmptyOperator(
        task_id="load_customer"
    )

    extract_customer >> transform_customer >> load_customer
```

The group appears as a logical section in the DAG view.

### Why use TaskGroups?

- Improve DAG readability.
- Organize related tasks.
- Reduce visual complexity.
- Make large workflows easier to understand.

### TaskGroup vs DAG

| TaskGroup | DAG |
|---|---|
| Groups related tasks | Defines the complete workflow |
| Exists inside a DAG | Is the workflow definition |
| Does not create an independent DAG run | Has its own DAG runs |
| Improves organization | Controls scheduling and dependencies |

**Important:** TaskGroups are organizational structures. They do not provide the same isolation as separate DAGs.

---

## 13. Dynamic Task Mapping

Dynamic Task Mapping allows Airflow to create task instances at runtime based on an input collection.

This is useful when the number of items to process is not known when the DAG is initially written.

### Example scenario

Suppose a pipeline needs to process files from multiple source systems:

```text
Process Files
     |
  -----------
  |    |    |
  v    v    v
File1 File2 File3
```

The number of files may change each day.

Instead of manually defining one task per file, dynamic task mapping creates mapped task instances.

### Python example

```python
from airflow.decorators import dag, task
from datetime import datetime


@dag(
    dag_id="dynamic_file_processing",
    start_date=datetime(2026, 1, 1),
    schedule=None,
    catchup=False,
)
def dynamic_file_processing():

    @task
    def get_files():
        return [
            "file1.csv",
            "file2.csv",
            "file3.csv",
        ]

    @task
    def process_file(file_name):
        print(f"Processing {file_name}")

    files = get_files()

    process_file.expand(file_name=files)


dynamic_file_processing()
```

### Important concepts

- `expand()` creates mapped task instances.
- Each mapped task receives one input value.
- The number of mapped instances depends on the runtime input.
- Airflow controls execution according to its task and concurrency limits.

Dynamic task mapping is useful for processing multiple files, tables, partitions, or customer groups.

---

## 14. Dependencies Across TaskGroups

Dependencies can be created between individual tasks and TaskGroups.

Example:

```python
from airflow.utils.task_group import TaskGroup
from airflow.operators.empty import EmptyOperator

start = EmptyOperator(
    task_id="start"
)

with TaskGroup(group_id="extract_group") as extract_group:

    extract_customers = EmptyOperator(
        task_id="customers"
    )

    extract_orders = EmptyOperator(
        task_id="orders"
    )

with TaskGroup(group_id="transform_group") as transform_group:

    transform_customers = EmptyOperator(
        task_id="customers"
    )

    transform_orders = EmptyOperator(
        task_id="orders"
    )

finish = EmptyOperator(
    task_id="finish"
)

start >> extract_group >> transform_group >> finish
```

This defines a dependency between the groups.

The downstream group is scheduled according to the states of its relevant upstream tasks and the applicable trigger rules.

---

## 15. Airflow Task Dependencies in a Data Engineering Project

Consider an e-commerce data pipeline.

### Requirements

- Extract customers and orders.
- Validate both datasets.
- Transform both datasets.
- Load curated tables.
- Run a final reconciliation.
- Send a notification.

### Workflow design

```text
                  Start
                    |
             ---------------
             |             |
             v             v
       Extract Customers  Extract Orders
             |             |
             v             v
       Validate Customers Validate Orders
             |             |
             v             v
       Transform Customers Transform Orders
             |             |
             ---------------
                    |
                    v
               Load Tables
                    |
                    v
              Reconciliation
                    |
                    v
                Notification
```

### Design considerations

1. Extract customer and order data independently when possible.
2. Validate each dataset before transformation.
3. Run independent transformations in parallel.
4. Load only after required transformations succeed.
5. Reconcile the final data.
6. Send notifications based on the final execution state.

### Practical tool integration

Airflow tasks can use operators or hooks to interact with external systems.

For example:

- SQL operators for database queries.
- Databricks operators for notebook or job execution.
- Cloud storage operators for file operations.
- Python operators for custom logic.
- Sensors for waiting on external conditions.

Airflow coordinates the work; the external systems perform their respective processing.

---

## 16. Common Mistakes to Avoid

- Thinking a DAG is the same as a DAG Run.
- Confusing a task definition with a task instance.
- Assuming `A >> B` means both tasks execute simultaneously.
- Assuming independent tasks always run in parallel.
- Forgetting that the default trigger rule is `all_success`.
- Assuming dependencies always require upstream success regardless of trigger rules.
- Using `all_done` when downstream processing should happen only after successful upstream tasks.
- Forgetting that branching marks unselected paths as skipped.
- Using the default `all_success` join after branching without considering skipped tasks.
- Treating TaskGroups as separate DAGs.
- Creating too many tiny tasks unnecessarily.
- Ignoring pools and concurrency limits.
- Assuming dynamic task mapping removes execution limits.

---

## 17. Interview Questions and Answers

### Q1. What is a DAG in Airflow?

A DAG is a Directed Acyclic Graph that defines tasks and their dependencies in a workflow.

### Q2. What is a task?

A task is an individual unit of work within a DAG, such as extracting data or running a transformation.

### Q3. What is a task instance?

A task instance is one execution of a task within a particular DAG Run.

### Q4. What is the difference between a DAG and a DAG Run?

A DAG defines the workflow. A DAG Run represents one execution of that workflow.

### Q5. How do you define dependencies in Airflow?

Use `>>`, `<<`, `set_downstream()`, or `set_upstream()`.

### Q6. What does `extract >> transform >> load` mean?

It means transform depends on extract, and load depends on transform.

### Q7. Can Airflow execute tasks in parallel?

Yes. Tasks without unmet dependencies can run concurrently, subject to executor capacity and concurrency settings.

### Q8. What is a trigger rule?

A trigger rule determines when a task is allowed to run based on the states of its upstream tasks.

### Q9. What is the default trigger rule?

`all_success`. All upstream tasks must succeed before the downstream task can run.

### Q10. What is the purpose of `all_done`?

It allows a task to run after all upstream tasks reach terminal states, regardless of whether they succeeded, failed, or were skipped.

### Q11. What is branching in Airflow?

Branching selects which downstream task or tasks should execute based on a condition. Unselected paths are skipped.

### Q12. What is a TaskGroup?

A TaskGroup organizes related tasks visually and logically within a DAG. It does not create an independent DAG.

### Q13. What is dynamic task mapping?

Dynamic task mapping creates task instances at runtime based on a collection of input values.

### Q14. What happens if an upstream task fails?

Under the default `all_success` trigger rule, its downstream task will not run normally and may become `upstream_failed`.

### Q15. How would you design a pipeline with independent datasets?

I would extract and validate each dataset independently, run their transformations in parallel, and create dependencies before loading the final combined output.

### Q16. What is the difference between a TaskGroup and a separate DAG?

A TaskGroup organizes tasks within one DAG. A separate DAG has its own scheduling and DAG-run lifecycle.

### Q17. How do you prevent too many tasks from accessing the same database simultaneously?

I would use Airflow pools, task concurrency limits, DAG-level limits, and appropriate executor capacity.

### Q18. How would you handle a notification task that must run after success or failure?

I would configure the notification task with `TriggerRule.ALL_DONE` and inspect upstream task states to determine the notification content.

---

## 18. Quick Revision

| Concept | Key Point |
|---|---|
| DAG | Defines the workflow |
| Directed | Dependencies have direction |
| Acyclic | No dependency cycles |
| Task | Individual unit of work |
| Task Instance | Execution of a task in a DAG Run |
| DAG Run | One execution of a DAG |
| `>>` | Defines downstream dependency |
| `<<` | Defines upstream dependency |
| Parallel execution | Independent tasks can run concurrently |
| `all_success` | All upstream tasks must succeed |
| `all_done` | All upstream tasks must finish |
| Branching | Selects execution paths |
| TaskGroup | Organizes related tasks |
| Dynamic Task Mapping | Creates task instances from runtime inputs |
| Pool | Controls concurrent use of shared resources |

### Five points to remember

1. A DAG defines the workflow; a task represents one unit of work.
2. A task instance is one execution of a task within a DAG Run.
3. `>>` and `<<` define dependency relationships.
4. Trigger rules determine how upstream task states affect downstream execution.
5. TaskGroups organize tasks, while dynamic task mapping creates task instances at runtime.

---

## 19. Self-Assessment

Try answering these questions without looking at the notes:

1. Explain DAG, task, DAG Run, and task instance.
2. Why must a DAG be acyclic?
3. What is the difference between `>>` and `<<`?
4. How do you create parallel tasks?
5. What is the default Airflow trigger rule?
6. Explain `all_success`, `all_done`, and `one_failed`.
7. How does branching work in Airflow?
8. What is a TaskGroup, and how is it different from a DAG?
9. What is dynamic task mapping?
10. Design an Airflow DAG for an e-commerce pipeline with parallel customer and order processing.

