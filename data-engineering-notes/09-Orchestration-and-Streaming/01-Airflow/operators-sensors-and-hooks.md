

# Operators, Sensors, and Hooks in Apache Airflow

## 1. Overview

Operators, Sensors, and Hooks are important building blocks in Apache Airflow. They help define what a pipeline does, wait for external conditions, and communicate with external systems.

- **Operator:** Defines a unit of work to execute.
- **Sensor:** Waits for a specific condition to become true.
- **Hook:** Provides an interface for connecting to and interacting with an external system.

Example: An order-processing pipeline may need to:

1. Wait for an orders file to arrive in cloud storage.
2. Read the file.
3. Transform the data.
4. Load it into a database.
5. Send a notification.

Airflow uses sensors, operators, and hooks to coordinate these activities.

---

## 2. What Is an Operator?

An operator is a reusable class that defines a unit of work in an Airflow DAG.

When an operator is instantiated inside a DAG, it creates a task.

Examples:

- `PythonOperator`: Executes a Python function.
- `BashOperator`: Executes a shell command.
- `SQLExecuteQueryOperator`: Executes SQL against a supported database.
- `EmptyOperator`: Represents a task without business logic.
- `DatabricksSubmitRunOperator`: Submits a Databricks job run.

### Basic example

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.python import PythonOperator


def extract_data():
    print("Extracting customer data")


with DAG(
    dag_id="operator_example",
    start_date=datetime(2026, 1, 1),
    schedule=None,
    catchup=False,
) as dag:

    extract = PythonOperator(
        task_id="extract_data",
        python_callable=extract_data,
    )
```

Here:

- `PythonOperator` is the operator class.
- `extract` is the task object created from that operator.
- `task_id` identifies the task.
- `python_callable` specifies the function to execute.

**Important:** An operator defines the task's behavior. A task instance represents one execution of that task.

---

## 3. Common Types of Operators

### 3.1 PythonOperator

Executes a Python function.

Useful for:
- Data validation.
- Custom transformations.
- Calling Python libraries.
- Running application logic.

```python
from airflow.operators.python import PythonOperator


def validate_data():
    print("Validating data")


validate = PythonOperator(
    task_id="validate_data",
    python_callable=validate_data,
)
```

### 3.2 BashOperator

Executes a Bash command.

Useful for:
- Running shell scripts.
- Executing command-line tools.
- Running system-level operations.

```python
from airflow.operators.bash import BashOperator

run_script = BashOperator(
    task_id="run_script",
    bash_command="echo 'Starting data pipeline'",
)
```

### 3.3 SQLExecuteQueryOperator

Executes SQL using an Airflow connection and a supported database provider.

Useful for:
- Running SQL transformations.
- Executing stored procedures.
- Creating or updating database tables.

```python
from airflow.providers.common.sql.operators.sql import (
    SQLExecuteQueryOperator
)

run_query = SQLExecuteQueryOperator(
    task_id="run_query",
    conn_id="warehouse_connection",
    sql="SELECT COUNT(*) FROM orders;",
)
```

The connection ID must be configured in Airflow, and the appropriate database provider package must be installed.

### 3.4 EmptyOperator

Represents a task that performs no business operation.

Useful for:
- Starting or ending workflow sections.
- Organizing dependencies.
- Creating logical workflow boundaries.

```python
from airflow.operators.empty import EmptyOperator

start = EmptyOperator(
    task_id="start"
)
```

### 3.5 Databricks Operators

Airflow can submit and monitor Databricks workloads through the Databricks provider.

For example, `DatabricksSubmitRunOperator` submits a run using a Databricks job or task configuration.

Useful for:
- Running PySpark transformations.
- Executing Databricks notebooks.
- Triggering data processing jobs.
- Coordinating Databricks workflows.

The operator requires the appropriate provider package and a configured Databricks connection.

---

## 4. Operator vs Task vs DAG

These terms are related but different.

| Concept | Meaning |
|---|---|
| DAG | Defines the overall workflow |
| Operator | Defines what a task does |
| Task | An instantiated operator within a DAG |
| Task Instance | One execution of a task in a DAG Run |

Example:

```python
extract = PythonOperator(
    task_id="extract_data",
    python_callable=extract_data,
)
```

- `PythonOperator` is the operator.
- `extract` is the task object.
- A particular execution of `extract` for a DAG Run is a task instance.

---

## 5. What Is a Sensor?

A sensor is a specialized operator that waits for a condition to become true.

Unlike a typical operator that performs an operation, a sensor primarily checks whether something has happened.

Common examples:

- Wait for a file to arrive.
- Wait for a database record.
- Wait for another DAG to complete.
- Wait for an external job to finish.
- Wait until a particular time.

### Example: Waiting for a file

Suppose a daily pipeline should start processing only after `orders.csv` arrives.

```text
Wait for orders.csv
        |
        v
   Extract Data
        |
        v
  Transform Data
        |
        v
    Load Data
```

A file sensor can wait until the file becomes available.

---

## 6. Common Types of Sensors

### 6.1 FileSensor

Waits for a file to appear at a specified location.

```python
from airflow.providers.standard.sensors.filesystem import FileSensor

wait_for_file = FileSensor(
    task_id="wait_for_orders_file",
    fs_conn_id="filesystem_default",
    filepath="/data/orders.csv",
    poke_interval=60,
    timeout=3600,
    mode="reschedule",
)
```

Explanation:

- `fs_conn_id`: Filesystem connection.
- `filepath`: File path to check.
- `poke_interval`: Time between checks, in seconds.
- `timeout`: Maximum time to wait, in seconds.
- `mode`: Defines how the sensor waits.

The filesystem connection must be configured, and the path must be accessible from the sensor's execution environment.

### 6.2 TimeSensor

Waits until a specified time of day.

```python
from datetime import time
from airflow.providers.standard.sensors.time import TimeSensor

wait_until_time = TimeSensor(
    task_id="wait_until_9am",
    target_time=time(9, 0),
)
```

The time is interpreted using the applicable DAG and timezone context.

### 6.3 ExternalTaskSensor

Waits for a task or DAG run in another DAG to reach a required state.

Example:

```python
from airflow.sensors.external_task import ExternalTaskSensor

wait_for_upstream = ExternalTaskSensor(
    task_id="wait_for_customer_pipeline",
    external_dag_id="customer_pipeline",
    external_task_id=None,
    allowed_states=["success"],
    failed_states=["failed"],
    poke_interval=60,
    timeout=3600,
    mode="reschedule",
)
```

Here, `external_task_id=None` means it waits for the external DAG Run rather than a particular task.

This is useful when one pipeline depends on another pipeline.

### 6.4 SQL Sensors

A SQL sensor waits until a SQL query returns a result that satisfies its condition.

For example, a pipeline can wait until a source table contains records for the required business date.

The exact SQL sensor class and behavior depend on the installed provider and Airflow version.

### 6.5 HttpSensor

Waits for an HTTP endpoint to meet a configured response condition.

Useful for:
- Checking whether an API is available.
- Waiting for an external service.
- Checking whether a job-status endpoint reports completion.

It requires the appropriate HTTP provider and connection configuration.

---

## 7. Sensor Modes: Poke vs Reschedule vs Deferrable

Sensors can wait in different ways.

### 7.1 Poke Mode

In poke mode, the sensor checks the condition repeatedly while keeping its worker slot.

```python
mode="poke"
```

Example:

```text
Check condition
      |
      v
Condition false?
      |
      v
Wait for interval
      |
      v
Check again
```

Advantages:
- Simple behavior.
- Suitable for short waits.

Disadvantage:
- Holds a worker slot while waiting.

### 7.2 Reschedule Mode

In reschedule mode, the sensor releases its worker slot between checks.

```python
mode="reschedule"
```

Example:

```text
Check condition
      |
      v
Condition false?
      |
      v
Release worker slot
      |
      v
Wait for next check
      |
      v
Check again
```

Advantages:
- Uses worker capacity more efficiently for longer waits.

Disadvantage:
- The task must be scheduled again for each check.

### 7.3 Deferrable Sensors

Deferrable sensors use asynchronous waiting.

When the condition is not met, the task defers and hands control to the Triggerer. The worker slot is released while waiting.

Example:

```python
from airflow.providers.http.sensors.http import HttpSensor

wait_for_api = HttpSensor(
    task_id="wait_for_api",
    http_conn_id="api_connection",
    endpoint="health",
    deferrable=True,
)
```

Deferrable support depends on the particular sensor, provider, and Airflow version.

### Comparison

| Feature | Poke | Reschedule | Deferrable |
|---|---|---|---|
| Worker slot while waiting | Held | Released between checks | Released |
| Waiting mechanism | Repeated checks | Repeated scheduled checks | Triggerer-based asynchronous waiting |
| Suitable for | Short waits | Longer polling waits | Long asynchronous waits |
| Requires Triggerer | No | No | Yes |

**Interview point:** Reschedule and deferrable modes both release worker capacity, but they use different waiting mechanisms.

---

## 8. What Is a Hook?

A Hook is an interface that allows Airflow code to connect to and interact with an external system.

Examples:

- PostgreSQL Hook.
- MySQL Hook.
- HTTP Hook.
- Databricks Hook.
- Cloud storage Hooks.

Hooks typically handle connection configuration and provide methods for interacting with the external service.

### Why use Hooks?

Without a Hook, developers may need to write connection and authentication logic manually.

Hooks help centralize this logic and integrate it with Airflow Connections and provider functionality.

### Example: PostgreSQL Hook

```python
from airflow.providers.postgres.hooks.postgres import PostgresHook


def get_orders():
    postgres_hook = PostgresHook(
        postgres_conn_id="postgres_connection"
    )

    records = postgres_hook.get_records(
        "SELECT order_id, amount FROM orders;"
    )

    print(records)
```

This example uses a PostgreSQL connection configured in Airflow.

The PostgreSQL provider package must be installed.

### What does the Hook do?

1. Reads the configured connection.
2. Establishes a connection to PostgreSQL.
3. Executes the query through its database interface.
4. Returns the result to the calling code.

---

## 9. Common Types of Hooks

| Hook | Purpose |
|---|---|
| `PostgresHook` | Interacts with PostgreSQL |
| `MySqlHook` | Interacts with MySQL |
| `MsSqlHook` | Interacts with Microsoft SQL Server |
| `HttpHook` | Makes HTTP requests |
| `DatabricksHook` | Interacts with Databricks |
| Cloud storage Hooks | Interact with supported cloud storage services |

Hook names and import paths depend on the provider package and its version.

### Example: HTTP Hook

```python
from airflow.providers.http.hooks.http import HttpHook


def call_api():
    http_hook = HttpHook(
        method="GET",
        http_conn_id="api_connection",
    )

    response = http_hook.run(
        endpoint="customers"
    )

    print(response.status_code)
```

The HTTP connection stores the base URL and any required connection configuration.

Do not hardcode API credentials in the DAG.

---

## 10. Operator vs Sensor vs Hook

This is one of the most important interview topics.

| Feature | Operator | Sensor | Hook |
|---|---|---|---|
| Main purpose | Performs a unit of work | Waits for a condition | Connects to an external system |
| Creates a task | Yes | Yes, because it is a specialized operator | No, not by itself |
| Typical behavior | Executes an operation | Checks a condition | Provides connection and interaction methods |
| Example | PythonOperator | FileSensor | PostgresHook |
| Common use | Transform data | Wait for a file | Execute database operations through a connection |

### Simple example

Suppose you need to load a file into PostgreSQL.

- `FileSensor`: Waits for the file.
- `PythonOperator`: Runs custom loading logic.
- `PostgresHook`: Connects to PostgreSQL and executes database operations.

```text
FileSensor
    |
    v
PythonOperator
    |
    v
PostgresHook
    |
    v
PostgreSQL
```

**Remember:** A Hook is generally used inside task code. An Operator defines the task, while a Sensor defines a task that waits.

---

## 11. Using a Hook Inside a PythonOperator

A common data engineering pattern is to use a PythonOperator with a Hook.

Example:

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.postgres.hooks.postgres import PostgresHook


def extract_orders():
    hook = PostgresHook(
        postgres_conn_id="postgres_connection"
    )

    records = hook.get_records(
        "SELECT order_id, amount FROM orders;"
    )

    print(f"Extracted {len(records)} records")


with DAG(
    dag_id="postgres_extraction_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule=None,
    catchup=False,
) as dag:

    extract = PythonOperator(
        task_id="extract_orders",
        python_callable=extract_orders,
    )
```

### Execution flow

1. Airflow schedules the `extract_orders` task.
2. PythonOperator executes the Python function.
3. The function creates a PostgresHook.
4. The Hook uses the configured Airflow connection.
5. The SQL query executes.
6. The result is returned to the Python function.

### Important point

Use an appropriate provider operator when it already provides the required functionality. Use a Hook inside custom Python code when you need more control over the interaction.

---

## 12. Connections and Hooks

Airflow Connections store configuration needed to communicate with external systems.

A connection may contain:

- Connection ID.
- Connection type.
- Host.
- Port.
- Username.
- Password.
- Database.
- Extra configuration.

### Example

A PostgreSQL connection might be configured as:

| Field | Example |
|---|---|
| Connection ID | `postgres_connection` |
| Connection Type | PostgreSQL |
| Host | `database.example.com` |
| Port | `5432` |
| Database | `sales` |
| Username | Configured user |
| Password | Stored securely |

A Hook uses the connection ID to retrieve the configuration.

```python
hook = PostgresHook(
    postgres_conn_id="postgres_connection"
)
```

### Important security practices

- Avoid hardcoding passwords in DAG files.
- Use Airflow Connections.
- Use a secrets backend where appropriate.
- Apply least-privilege permissions.
- Avoid logging credentials or tokens.
- Use managed identities or service principals where supported and appropriate.

A configured connection does not automatically guarantee access. The identity must also have the required permissions and network connectivity.

---

## 13. Building an End-to-End Pipeline

### Scenario

A daily orders pipeline must wait for a CSV file, validate it, load records into PostgreSQL, and complete.

### Workflow

```text
Wait for Orders File
         |
         v
   Validate File
         |
         v
    Load Orders
         |
         v
       Finish
```

### Example DAG

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.empty import EmptyOperator
from airflow.operators.python import PythonOperator
from airflow.providers.standard.sensors.filesystem import FileSensor
from airflow.providers.postgres.hooks.postgres import PostgresHook


def validate_file():
    print("Validating orders file")


def load_orders():
    hook = PostgresHook(
        postgres_conn_id="postgres_connection"
    )

    records = hook.get_records(
        "SELECT COUNT(*) FROM orders;"
    )

    print(f"Current target count: {records[0][0]}")


with DAG(
    dag_id="orders_file_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:

    wait_for_file = FileSensor(
        task_id="wait_for_orders_file",
        fs_conn_id="filesystem_default",
        filepath="/data/orders.csv",
        poke_interval=60,
        timeout=3600,
        mode="reschedule",
    )

    validate = PythonOperator(
        task_id="validate_file",
        python_callable=validate_file,
    )

    load = PythonOperator(
        task_id="load_orders",
        python_callable=load_orders,
    )

    finish = EmptyOperator(
        task_id="finish"
    )

    wait_for_file >> validate >> load >> finish
```

This is a learning example. The validation function currently logs a message, and the loading function reads a target count rather than loading the CSV. A production implementation must add actual file validation and ingestion logic.

---

## 14. Common Mistakes to Avoid

1. Confusing an Operator with a Hook.
2. Thinking a Hook creates an Airflow task by itself.
3. Using a sensor for work that should be performed by a normal operator.
4. Using poke mode for long waits without considering worker capacity.
5. Assuming every sensor supports deferrable execution.
6. Forgetting that deferrable tasks require a functioning Triggerer.
7. Hardcoding connection credentials in Python code.
8. Assuming an Airflow Connection automatically grants external-system permissions.
9. Forgetting to install the required provider package.
10. Using an outdated operator or Hook import path.
11. Ignoring sensor timeout and polling configuration.
12. Using custom Python code when an appropriate provider operator already handles the task.

---

## 15. Interview Questions and Answers

### Q1. What is an Operator in Airflow?

An Operator is a reusable class that defines a unit of work. When instantiated inside a DAG, it creates a task.

### Q2. What is a Sensor?

A Sensor is a specialized operator that waits for a condition, such as a file arriving or another DAG completing.

### Q3. What is a Hook?

A Hook provides an interface for connecting to and interacting with external systems such as databases, APIs, and cloud services.

### Q4. What is the difference between an Operator and a Hook?

An Operator defines a task. A Hook provides connection and interaction functionality that can be used inside task code.

### Q5. What is the difference between an Operator and a Sensor?

An Operator performs a unit of work. A Sensor is a specialized operator that waits for a particular condition.

### Q6. What is PythonOperator used for?

It executes a Python function and is useful for custom logic, validation, and data processing.

### Q7. What is BashOperator used for?

It executes shell commands or scripts.

### Q8. What is FileSensor?

FileSensor waits until a specified file becomes available at a configured location.

### Q9. What is ExternalTaskSensor?

It waits for a task or DAG Run in another DAG to reach a specified state.

### Q10. What is the difference between poke and reschedule modes?

Poke mode holds a worker slot while waiting. Reschedule mode releases the worker slot between checks.

### Q11. What is a deferrable operator or sensor?

It can defer its execution and hand waiting to the Triggerer, releasing the worker slot while waiting.

### Q12. What is the role of the Triggerer?

The Triggerer runs asynchronous triggers for deferrable tasks and reports when their conditions are met.

### Q13. How do Hooks use Airflow Connections?

A Hook uses a connection ID to retrieve connection configuration and establish communication with the external system.

### Q14. Can we use a Hook without PythonOperator?

Yes. Hooks can be used inside other task implementations, including custom operators. They do not have to be used specifically with PythonOperator.

### Q15. When would you use a Hook instead of an Operator?

I would use a Hook when I need custom Python logic to interact with an external system. If a suitable provider Operator already performs the required task, I would generally prefer that Operator.

### Q16. How would you wait for a file before processing it?

I would use a FileSensor with the correct connection and file path, configure an appropriate polling interval and timeout, and then set the extraction task as its downstream dependency.

### Q17. How do you securely connect Airflow to a database?

I would configure an Airflow Connection or secrets backend, use an appropriate authentication method, and ensure the execution identity has the required permissions.

### Q18. What is a provider package?

A provider package adds integrations for external systems, including operators, sensors, hooks, connections, and related functionality.

---

## 16. Quick Revision

| Concept | Remember |
|---|---|
| Operator | Defines a unit of work |
| Sensor | Waits for a condition |
| Hook | Connects to an external system |
| PythonOperator | Executes Python functions |
| BashOperator | Executes shell commands |
| SQLExecuteQueryOperator | Executes SQL |
| EmptyOperator | Performs no business operation |
| FileSensor | Waits for a file |
| TimeSensor | Waits until a specified time |
| ExternalTaskSensor | Waits for another DAG or task |
| Poke mode | Holds worker slot while waiting |
| Reschedule mode | Releases worker slot between checks |
| Deferrable | Uses Triggerer for asynchronous waiting |
| Airflow Connection | Stores external connection configuration |
| Provider | Supplies integrations for external systems |

### Five points to remember

1. Operators define tasks.
2. Sensors are specialized operators that wait for conditions.
3. Hooks provide interfaces to external systems.
4. Sensors can use poke, reschedule, or deferrable execution where supported.
5. Hooks commonly use Airflow Connections to establish external connections.

---

## 17. Self-Assessment

Try answering these questions without looking at the notes:

1. Explain Operator, Sensor, and Hook.
2. What is the difference between a task and an operator?
3. What is PythonOperator used for?
4. When would you use BashOperator?
5. What is the purpose of FileSensor?
6. Explain ExternalTaskSensor with a practical example.
7. What is the difference between poke and reschedule modes?
8. How does a deferrable sensor work?
9. What is the role of the Triggerer?
10. How does a Hook use an Airflow Connection?
11. Can a Hook create an Airflow task by itself?
12. How would you build a pipeline that waits for a file and then loads it into PostgreSQL?

