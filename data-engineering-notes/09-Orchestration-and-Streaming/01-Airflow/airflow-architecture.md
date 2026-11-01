

# Apache Airflow Architecture

## 1. What Is Apache Airflow?

Apache Airflow is an open-source platform used to **author, schedule, orchestrate, and monitor workflows**.

It allows data engineers to define workflows as Python code and manage their execution through a central orchestration system.

For example, a daily data engineering workflow may involve:

1. Extract data from a source database.
2. Copy raw data into cloud storage.
3. Run a Databricks transformation.
4. Validate the transformed data.
5. Load the final data into a warehouse.
6. Send a notification when the workflow finishes.

Airflow coordinates these tasks and manages their dependencies.

### Simple definition

Apache Airflow is a workflow orchestration platform that schedules and monitors tasks defined as Directed Acyclic Graphs (DAGs).

---

## 2. Why Do We Need Airflow?

Consider a data pipeline with multiple dependent operations.

```text
Extract Data
     |
     v
Clean Data
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

Without an orchestrator, developers may need to manage execution order, schedules, retries, failures, and monitoring manually.

Airflow provides a centralized way to manage these operations.

### Key benefits

- **Scheduling:** Run workflows at defined times.
- **Dependency management:** Execute tasks in the correct order.
- **Retries:** Retry failed tasks according to configuration.
- **Monitoring:** Track workflow and task execution.
- **Logging:** Inspect task execution logs.
- **Alerting:** Notify teams about failures or important events.
- **Extensibility:** Integrate with databases, cloud platforms, Spark, and other systems.
- **Python-based workflows:** Define workflow logic using Python.

---

## 3. What Is a DAG?

DAG stands for **Directed Acyclic Graph**.

It represents a workflow as tasks connected through dependencies.

- **Directed:** Connections have a direction.
- **Acyclic:** A task cannot eventually depend on itself through a cycle.
- **Graph:** Tasks are represented as nodes, and dependencies as edges.

Example:

```text
        Extract Data
             |
             v
        Validate Data
             |
       -------------
       |           |
       v           v
 Transform A   Transform B
       |           |
       -------------
             |
             v
       Load Warehouse
```

Here:

- Extract runs first.
- Validation runs after extraction.
- Transform A and Transform B can run in parallel.
- Loading starts after both transformations finish successfully, assuming the default success dependency behavior.

### DAG vs Pipeline

| DAG | Pipeline |
|---|---|
| A graph defining tasks and dependencies | A sequence or collection of data-processing operations |
| Airflow uses DAGs to represent workflows | A general data engineering concept |
| Defines execution relationships | Describes the end-to-end data movement or processing |

An Airflow DAG is one way to implement a data pipeline.

---

## 4. High-Level Airflow Architecture

Airflow consists of several major components.

<AsyncImage query="Apache Airflow architecture diagram scheduler webserver metadata database executor workers dag processor" aspectRatio="16:9" width="100%" fit="contain"/>

The main components are:

1. DAG files
2. DAG Processor
3. Scheduler
4. Executor
5. Workers
6. Webserver or Airflow UI
7. Metadata Database
8. Triggerer

Their responsibilities are different, but they work together to execute workflows.

---

## 5. DAG Files

DAG files contain Python code that defines workflows.

They specify:

- DAG name.
- Tasks.
- Task dependencies.
- Scheduling information.
- Retry configuration.
- Execution parameters.

Example:

```python
from datetime import datetime

from airflow import DAG
from airflow.operators.empty import EmptyOperator

with DAG(
    dag_id="daily_data_pipeline",
    start_date=datetime(2026, 1, 1),
    schedule="@daily",
    catchup=False,
) as dag:

    start = EmptyOperator(task_id="start")

    extract = EmptyOperator(task_id="extract")

    transform = EmptyOperator(task_id="transform")

    load = EmptyOperator(task_id="load")

    start >> extract >> transform >> load
```

This example defines a simple DAG with four tasks.

`EmptyOperator` is used here to demonstrate dependencies. It does not perform actual data extraction or transformation.

### Important points

- DAGs are written in Python.
- The DAG file defines the workflow, not the actual execution history.
- Tasks define individual units of work.
- Dependencies determine execution order.
- The scheduler uses DAG information to decide when task instances can run.

---

## 6. DAG Processor

The DAG Processor is responsible for processing DAG files and making their definitions available to Airflow.

It:

- Reads Python DAG files.
- Parses DAG definitions.
- Identifies tasks and dependencies.
- Processes DAG metadata.
- Makes updated DAG information available to the scheduling system.

### Why is DAG processing important?

The scheduler needs to know:

- Which DAGs exist.
- Which tasks belong to each DAG.
- How tasks depend on each other.
- When a DAG should be scheduled.

DAG processing provides this information.

### Important distinction

DAG processing is not the same as executing task logic.

The DAG Processor reads and processes workflow definitions. Workers execute the actual tasks in worker-based execution configurations.

---

## 7. Scheduler

The Scheduler is responsible for deciding when tasks should run.

It is one of the most important Airflow components.

### Responsibilities

- Identifies DAG runs that need to be scheduled.
- Checks task dependencies.
- Determines which task instances are ready.
- Considers available execution capacity.
- Sends eligible tasks to the executor.
- Updates scheduling information in the metadata database.

### Example

Suppose a DAG contains:

```text
Extract
   |
   v
Transform
   |
   v
Load
```

The scheduler ensures:

1. Extract becomes eligible at the appropriate time.
2. Transform waits for Extract to meet its dependency condition.
3. Load waits for Transform to meet its dependency condition.

### Important point

The scheduler decides which tasks are ready to execute. It does not necessarily execute the task code itself.

The executor is responsible for arranging task execution through the configured execution model.

---

## 8. Executor

The Executor determines how Airflow runs task instances.

It acts as the interface between the scheduler and the execution environment.

Depending on the executor, tasks may run in the scheduler's environment, in separate processes, or on distributed workers.

### Common executor types

| Executor | Execution model | Typical use |
|---|---|---|
| SequentialExecutor | Executes tasks sequentially | Simple testing or limited environments |
| LocalExecutor | Runs tasks as local processes | Small to medium single-machine deployments |
| CeleryExecutor | Distributes tasks to Celery workers | Distributed worker-based deployments |
| KubernetesExecutor | Creates Kubernetes pods for task instances | Containerized, dynamically allocated execution |

### 8.1 SequentialExecutor

Runs tasks sequentially rather than concurrently.

It is suitable for simple environments and testing, not high-throughput production workloads.

### 8.2 LocalExecutor

Runs tasks as local processes on the machine hosting the executor.

It can execute multiple tasks concurrently, subject to configured capacity and system resources.

### 8.3 CeleryExecutor

Uses Celery to distribute tasks across worker machines.

```text
              Scheduler
                  |
                  v
              Executor
                  |
                  v
              Message Broker
                  |
          -------------------
          |        |        |
          v        v        v
       Worker 1 Worker 2 Worker 3
```

The message broker distributes task messages to workers.

A Celery deployment commonly uses Redis or RabbitMQ as its broker.

### 8.4 KubernetesExecutor

Runs each task instance in its own Kubernetes pod.

```text
              Scheduler
                  |
                  v
              Executor
                  |
                  v
             Kubernetes
                  |
          -------------------
          |        |        |
          v        v        v
        Pod 1    Pod 2    Pod 3
```

Advantages:

- Task-level isolation.
- Dynamic resource allocation.
- Container-based execution.
- Independent environments for tasks.

### Executor comparison

| Feature | Sequential | Local | Celery | Kubernetes |
|---|---|---|---|---|
| Sequential execution | Yes | No | No | No |
| Parallel execution | No | Yes | Yes | Yes |
| Distributed workers | No | No | Yes | Kubernetes pods |
| External broker | No | No | Yes | No Celery broker required |
| Container isolation | Not inherent | Not inherent | Optional | Yes |
| Operational complexity | Low | Low to medium | Higher | Higher |

**Important:** Executor availability and support can depend on the Airflow version and deployment. Some older executor options have changed or been deprecated in newer releases. Understand the execution model rather than memorizing only the names.

---

## 9. Workers

Workers execute task code in worker-based Airflow deployments.

They receive tasks through the configured execution mechanism and run the corresponding operator or task logic.

### Responsibilities

- Execute Python functions or operators.
- Connect to external systems.
- Run data extraction or transformation commands.
- Generate task logs.
- Report task execution results.

### Example

A worker may execute a task that:

- Runs a SQL query.
- Submits a Spark job.
- Starts a Databricks notebook.
- Copies data to ADLS Gen2.
- Calls an external API.

### Important distinction

Airflow workers execute the orchestration tasks. They do not necessarily perform all heavy data processing themselves.

For example, an Airflow task can submit a Databricks job and monitor its result. Databricks performs the distributed Spark processing.

---

## 10. Metadata Database

The Metadata Database stores information about Airflow's operational state.

It is essential for tracking workflow execution.

### Information stored

- DAG and task execution metadata.
- DAG runs.
- Task instances.
- Task states.
- Scheduling information.
- Connections and variables.
- Pools and other configuration metadata.
- Other operational information, depending on the Airflow version and configuration.

Common database choices include:

- PostgreSQL
- MySQL

SQLite is useful for simple development and testing, but it is not appropriate for a production deployment.

### Example

Suppose a task runs successfully.

Airflow stores its execution status in the metadata database.

```text
Task: extract_data

State: Success
Start Time: 09:00
End Time:   09:03
Duration:   3 minutes
```

The UI can retrieve this information and display the task status.

### Important point

The metadata database is not intended to store the actual business datasets processed by the pipeline.

It stores Airflow's metadata, not the company's analytical data.

---

## 11. Webserver and Airflow UI

The Webserver provides the Airflow user interface.

It allows users to interact with and monitor workflows.

### Main responsibilities

- Display available DAGs.
- Show DAG execution history.
- Display task statuses.
- Show task logs.
- Support manual DAG triggering.
- Provide access to configuration and operational views.

### Common task states

| State | Meaning |
|---|---|
| `none` | Task instance has not entered a defined execution state |
| `scheduled` | Task is eligible and waiting for scheduling |
| `queued` | Task has been sent to the executor and is waiting for execution |
| `running` | Task is currently executing |
| `success` | Task completed successfully |
| `failed` | Task execution failed |
| `up_for_retry` | Task failed and is waiting for a retry |
| `up_for_reschedule` | A rescheduling sensor is waiting for another execution attempt |
| `skipped` | Task was intentionally skipped |
| `upstream_failed` | An upstream task failed, preventing normal execution |
| `deferred` | Task has deferred its execution and is waiting for an event or condition |

The exact states and transitions depend on Airflow version and task configuration.

### Example

```text
extract_data       SUCCESS
transform_data     SUCCESS
validate_data      FAILED
load_warehouse     UPSTREAM_FAILED
```

This makes it easier to identify where a workflow failed.

---

## 12. Triggerer

The Triggerer is a component used by Airflow's deferrable operators and triggers.

It allows tasks to wait for external events without continuously occupying a worker process.

### Why is it useful?

Consider a task waiting for a file to arrive in cloud storage.

A traditional sensor may occupy a worker while waiting.

A deferrable sensor can defer its execution and release the worker slot. The Triggerer monitors the asynchronous condition.

When the event occurs, the task is resumed.

### Simplified flow

```text
          Worker
             |
             v
       Deferrable Task
             |
             v
       Task Defers
             |
             v
          Triggerer
             |
       Wait for Event
             |
             v
       Event Occurs
             |
             v
       Task Resumes
```

### Benefits

- Reduces unnecessary worker usage.
- Supports efficient waiting for external events.
- Improves resource utilization for long-running waits.

**Important:** The Triggerer does not replace the Scheduler or Executor. It supports asynchronous waiting for deferrable tasks.

---

## 13. How Airflow Components Work Together

Consider a daily pipeline that extracts orders, transforms them, and loads a warehouse.

### Architecture flow

```text
                 DAG Files
                     |
                     v
                DAG Processor
                     |
                     v
                 Scheduler
                     |
                     v
                  Executor
                     |
              ----------------
              |              |
              v              v
           Worker 1       Worker 2
              |              |
              v              v
          Extract Data   Transform Data
              |              |
              ---------------
                     |
                     v
               Load Warehouse

          Metadata Database
             <--->
       Scheduler / UI / Components

          Webserver / Airflow UI
                     |
                     v
              User Monitoring
```

### Step-by-step execution

1. A developer writes a DAG in Python.
2. The DAG Processor parses the DAG file.
3. DAG information becomes available to the scheduling system.
4. The Scheduler identifies a DAG run that needs to execute.
5. The Scheduler checks task dependencies and scheduling constraints.
6. Eligible task instances are sent to the Executor.
7. The Executor arranges their execution according to the configured executor.
8. Workers execute tasks in worker-based deployments.
9. Task states and execution metadata are recorded in the metadata database.
10. The Webserver displays workflow status and logs.
11. If a task uses deferrable execution, the Triggerer handles its asynchronous waiting period.

---

## 14. Airflow Deployment Architectures

Airflow can run in different deployment configurations.

### 14.1 Standalone or Simple Development Setup

A simple setup is useful for local learning and experimentation.

```text
       Airflow Installation
                |
        -----------------
        |               |
        v               v
     Scheduler       Web UI
        |
        v
    Task Execution
```

The exact component arrangement depends on the Airflow version and deployment method.

### 14.2 Single-Machine Deployment

Several Airflow components run on one machine.

Example:

```text
        Single Machine
              |
      -------------------
      |        |        |
      v        v        v
  Scheduler  Web UI   Workers
      |
      v
 Metadata Database
```

This can be suitable for smaller workloads, but it has limited resource isolation and scalability.

### 14.3 Distributed Deployment

Components can run on separate machines or containers.

Example using CeleryExecutor:

```text
             Webserver
                 |
                 v
             Scheduler
                 |
                 v
             Metadata DB
                 |
                 v
            Message Broker
                 |
        ---------------------
        |         |         |
        v         v         v
     Worker 1  Worker 2  Worker 3
```

Benefits:

- Distributed task execution.
- Independent worker scaling.
- Better separation of responsibilities.
- Support for larger workloads.

### 14.4 Kubernetes Deployment

Airflow components run as Kubernetes workloads, while task execution can use KubernetesExecutor or another supported executor.

This provides container orchestration and flexible resource allocation.

---

## 15. Airflow vs Data Processing Engines

Airflow is an orchestrator, not a distributed data processing engine.

| Tool | Main responsibility |
|---|---|
| Apache Airflow | Workflow orchestration |
| Apache Spark | Distributed data processing |
| Databricks | Managed data processing and analytics platform |
| Azure Data Factory | Data integration and workflow orchestration |
| Kafka | Event streaming platform |
| Flink | Stream processing |

### Example: Airflow + Databricks

```text
             Apache Airflow
                    |
                    v
           Submit Databricks Job
                    |
                    v
             Databricks
                    |
                    v
              Spark Cluster
                    |
                    v
          Transform Large Dataset
                    |
                    v
              ADLS Gen2
```

Airflow coordinates the job. Databricks executes the Spark workload.

### Interview point

Airflow does not replace Spark. It orchestrates when and how Spark jobs are triggered and how their dependencies are managed.

---

## 16. Airflow Scheduling and Execution Concepts

Some important concepts are closely connected to the architecture.

### DAG Run

A DAG Run represents one execution of a DAG.

For example, a daily DAG may have one scheduled DAG Run for each logical daily interval.

### Task Instance

A Task Instance is one execution of a particular task within a specific DAG Run.

For example:

```text
DAG: daily_orders_pipeline
DAG Run: 2026-10-25

Task Instances:
    extract_orders
    transform_orders
    load_warehouse
```

Each task instance has its own state and execution details.

### Logical Date

The logical date identifies the data interval associated with a DAG Run. It is not necessarily the actual time when the DAG starts executing.

### Data Interval

The data interval represents the period of data that a scheduled DAG Run is intended to process.

For a daily workflow, a run may process data for the previous calendar day, depending on the schedule and timetable.

### Catchup

Catchup determines whether Airflow creates DAG Runs for past scheduled intervals that have not yet been run.

For example, if a daily DAG was paused for several days, catchup can cause the missed intervals to be scheduled when it is enabled.

### Important distinction

A DAG's schedule, logical date, data interval, and actual execution time are related but not interchangeable.

---

## 17. Scaling and Performance

Airflow performance depends on several components.

### Scheduler performance

Important factors:

- Number of DAGs.
- DAG parsing time.
- Number of task instances.
- Scheduling configuration.
- Metadata database performance.

### Worker performance

Important factors:

- CPU and memory.
- Number of concurrent tasks.
- Task execution duration.
- External system response time.
- Network connectivity.

### Metadata database performance

Important factors:

- Database capacity.
- Connection management.
- Metadata volume.
- Database maintenance.
- Query performance.

### Scaling approaches

| Component | Scaling approach |
|---|---|
| Scheduler | Configure scheduler capacity and, where supported, multiple schedulers |
| Workers | Add worker capacity or replicas |
| Kubernetes task execution | Scale task pods based on workload |
| Metadata database | Increase database capacity and optimize operations |
| DAG processing | Improve DAG parsing and processor capacity |

### Best practices

- Keep DAG files lightweight.
- Avoid expensive operations at DAG import time.
- Avoid unnecessary task dependencies.
- Use appropriate executor capacity.
- Keep task execution logic efficient.
- Use deferrable operators for supported long waits.
- Monitor metadata database performance.
- Avoid creating excessive numbers of tiny tasks.

---

## 18. Common Architecture Mistakes

- Thinking the Scheduler directly executes every task.
- Treating the Executor and Worker as the same component.
- Assuming the Metadata Database stores business datasets.
- Confusing DAG parsing with task execution.
- Assuming Airflow itself performs distributed Spark processing.
- Treating the Triggerer as a replacement for the Scheduler.
- Assuming all executor types use workers in the same way.
- Assuming the Webserver executes task code.
- Running heavy database queries or API calls at DAG import time.
- Assuming a successful DAG parse means the task will execute successfully.
- Assuming a task's actual execution date is always its logical date.
- Using SQLite as a production metadata database.

---

## 19. Interview Questions and Answers

### Q1. What is Apache Airflow?

Apache Airflow is an open-source workflow orchestration platform used to define, schedule, execute, and monitor workflows represented as DAGs.

### Q2. What is a DAG?

A DAG is a Directed Acyclic Graph. It represents tasks and their dependencies without allowing cycles.

### Q3. What are the main components of Airflow architecture?

The main components include:

- DAG Processor
- Scheduler
- Executor
- Workers
- Metadata Database
- Webserver
- Triggerer

### Q4. What is the role of the Scheduler?

The Scheduler identifies eligible DAG runs and task instances, evaluates dependencies, and submits ready tasks for execution through the Executor.

### Q5. What is the role of the Executor?

The Executor determines how task instances are executed according to the configured execution model.

### Q6. What is the difference between Scheduler and Executor?

The Scheduler determines which tasks are ready to run. The Executor handles how those task instances are dispatched and executed.

### Q7. What is the role of a Worker?

A Worker executes task code in worker-based execution configurations.

### Q8. What is the Metadata Database?

It stores Airflow operational metadata, including DAG runs, task instances, task states, and other configuration information.

### Q9. Does the Metadata Database store actual business data?

No. It stores Airflow's operational metadata. Business data is normally stored in systems such as ADLS Gen2, databases, or data warehouses.

### Q10. What is the role of the Webserver?

It provides the user interface for viewing DAGs, execution history, task status, and logs, and for interacting with workflows.

### Q11. What is the Triggerer?

The Triggerer handles asynchronous triggers for deferrable tasks, allowing tasks to wait for events without continuously occupying worker capacity.

### Q12. What is the difference between LocalExecutor and CeleryExecutor?

LocalExecutor runs tasks as local processes on the execution machine. CeleryExecutor distributes task execution across workers using a message broker.

### Q13. What is KubernetesExecutor?

KubernetesExecutor launches task instances in Kubernetes pods, providing isolated container-based execution.

### Q14. Does Airflow process large datasets?

Airflow is not designed to be the distributed data processing engine. It orchestrates external processing tools such as Spark and Databricks.

### Q15. How does Airflow work with Databricks?

Airflow can submit Databricks jobs, monitor their execution, and trigger downstream tasks after successful completion.

### Q16. What happens when a task fails?

Depending on its configuration, Airflow can retry the task, mark it as failed, and prevent downstream tasks from running when their dependency conditions are not met.

### Q17. What is the difference between a DAG Run and a Task Instance?

A DAG Run represents one execution of an entire DAG. A Task Instance represents one task's execution within that DAG Run.

### Q18. How does Airflow support parallel execution?

Tasks without unmet dependencies can run concurrently, subject to executor capacity, DAG limits, pools, and other concurrency settings.

### Q19. What is the difference between a traditional sensor and a deferrable sensor?

A traditional sensor may occupy a worker while waiting. A deferrable sensor can defer and use the Triggerer to monitor the condition, releasing the worker during the wait.

### Q20. How would you scale Airflow for a large data engineering workload?

I would choose an appropriate distributed execution model, scale workers, tune scheduler capacity, optimize DAG parsing, monitor the metadata database, and use external processing engines for heavy transformations.

---

## 20. Quick Revision

| Component | Main responsibility |
|---|---|
| DAG | Defines workflow and dependencies |
| DAG Processor | Parses and processes DAG files |
| Scheduler | Determines which task instances are ready |
| Executor | Arranges task execution |
| Worker | Executes task code in worker-based deployments |
| Metadata Database | Stores Airflow operational metadata |
| Webserver | Provides Airflow UI |
| Triggerer | Handles asynchronous triggers for deferrable tasks |
| DAG Run | One execution of a DAG |
| Task Instance | One task execution within a DAG Run |
| Logical Date | Identifies the DAG Run's logical time |
| Data Interval | Period of data associated with a scheduled run |

### Five points to remember

1. Airflow is a workflow orchestrator, not a data processing engine.
2. The Scheduler decides which tasks are ready to run.
3. The Executor determines how task execution is arranged.
4. The Metadata Database stores workflow execution information.
5. The Triggerer supports asynchronous waiting for deferrable tasks.

---

## 21. Self-Assessment

Try answering these questions without looking at the notes:

1. Explain Apache Airflow architecture.
2. What is a DAG, and why must it be acyclic?
3. Explain the responsibilities of the DAG Processor and Scheduler.
4. What is the difference between Scheduler, Executor, and Worker?
5. What information does the Metadata Database store?
6. Compare SequentialExecutor, LocalExecutor, CeleryExecutor, and KubernetesExecutor.
7. What is the purpose of the Triggerer?
8. Explain how Airflow schedules and executes a task.
9. How does Airflow orchestrate a Databricks job?
10. How would you design and scale Airflow for a daily data pipeline?

