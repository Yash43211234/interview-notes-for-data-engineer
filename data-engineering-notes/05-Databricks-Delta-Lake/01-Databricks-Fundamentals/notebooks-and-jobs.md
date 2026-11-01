# Databricks Notebooks and Jobs

## 1. What Is a Databricks Notebook?

A Databricks notebook is an interactive environment where developers and Data Engineers write, execute, test, and document code.

Notebooks support multiple languages, including:

- Python
- SQL
- Scala
- R

They are commonly used for data exploration, ETL development, debugging, testing, and data analysis.

### Example

A Data Engineer can use a notebook to:

1. Read raw data from ADLS Gen2.
2. Clean and transform data using PySpark.
3. Store processed data in Delta tables.
4. Validate the output.
5. Run SQL queries for analysis.

**Interview point:** A notebook is primarily a development and execution interface. A Databricks Job is used to orchestrate and automate workloads.

---

## 2. Notebook Structure: Cells

A notebook is divided into cells.

Each cell can contain code or documentation.

### Example: Python Cell

```python
print("Hello Databricks")
```

### Example: SQL Cell

```sql
SELECT *
FROM customers
LIMIT 10;
```

### Example: Markdown Cell

```markdown
# Customer Data Transformation

This notebook cleans customer data.
```

### Cell Execution

You can execute:
- A single cell.
- Multiple selected cells.
- The entire notebook.

**Important:** Notebook cells execute in a shared notebook context, subject to the language and execution environment. Variables and temporary objects can be available to later cells.

---

## 3. Notebook Magic Commands

Databricks provides magic commands to control notebook cell behavior.

Common commands include:

| Magic Command | Purpose |
|---|---|
| `%python` | Execute Python code |
| `%sql` | Execute SQL |
| `%scala` | Execute Scala |
| `%r` | Execute R |
| `%md` | Render Markdown |
| `%fs` | Access supported filesystem operations |
| `%run` | Execute another notebook |
| `%sh` | Run shell commands in the supported environment |

### Example: SQL Cell

```sql
%sql

SELECT customer_id, customer_name
FROM customers;
```

### Example: Markdown Cell

```markdown
%md

# Data Cleaning

This section handles missing values.
```

### Example: Filesystem Command

```python
%fs ls /FileStore/
```

**Important:** `%fs` is a notebook magic command. `dbutils.fs` is a Python-accessible utility interface.

---

## 4. Using dbutils

`dbutils` provides utilities for interacting with the Databricks environment.

Common use cases include:

- Accessing files.
- Managing widgets.
- Working with secrets.
- Exiting notebooks.
- Running other notebooks.

### A. File Operations

```python
dbutils.fs.ls("/FileStore/")
```

Copy a file:

```python
dbutils.fs.cp(
    "dbfs:/FileStore/input.csv",
    "dbfs:/FileStore/archive/input.csv"
)
```

Remove a file:

```python
dbutils.fs.rm(
    "dbfs:/FileStore/input.csv"
)
```

### B. Notebook Execution

```python
dbutils.notebook.run(
    "/Shared/child_notebook",
    300
)
```

The second argument represents the timeout in seconds.

### C. Exit a Notebook

```python
dbutils.notebook.exit("SUCCESS")
```

This returns a value to the calling notebook when using notebook orchestration.

**Important:** Available utilities and filesystem paths depend on the Databricks environment and workspace configuration. Do not assume that DBFS root or FileStore is enabled for every workspace.

---

## 5. What Is a Databricks Job?

A Databricks Job is used to automate and orchestrate workloads.

A job can execute tasks such as:

- Notebooks.
- Python scripts.
- SQL tasks.
- JAR tasks.
- Pipeline tasks.
- Other supported task types.

### Example: Daily ETL Pipeline

Suppose a company receives customer data every day.

A job can automate the following workflow:

1. Read raw customer data.
2. Validate incoming records.
3. Clean and transform the data.
4. Write the output to a Delta table.
5. Run quality checks.
6. Send notifications if the job fails.

This avoids manually executing notebooks every day.

---

## 6. Notebook vs Job

| Feature | Notebook | Job |
|---|---|---|
| Main purpose | Develop and execute code | Automate workloads |
| Execution | Interactive or programmatic | Scheduled or triggered |
| Code organization | Cells | Tasks and dependencies |
| Parameters | Widgets and task parameters | Job and task parameters |
| Monitoring | Notebook execution output | Job runs and task status |
| Typical usage | Development and debugging | Production workflows |

**Remember:** A notebook contains code. A job schedules and orchestrates execution of tasks, which may include notebooks.

A notebook can be executed manually without creating a job.

---

## 7. Creating a Databricks Job

A typical job creation workflow:

1. Open the Databricks workspace.
2. Navigate to Jobs and Pipelines or the relevant Jobs interface.
3. Create a new job.
4. Add a task.
5. Select the task type, such as Notebook.
6. Select the notebook path.
7. Configure compute.
8. Set parameters if required.
9. Configure dependencies and schedule.
10. Save and run the job.

The exact interface and available options may differ by Databricks workspace version.

### Example Job Configuration

```text
Job Name: daily_customer_etl

Task Name: customer_transformation

Task Type: Notebook

Notebook Path: /Shared/customer_etl

Compute: Job Compute

Schedule: Daily at 2:00 AM

Timeout: 60 minutes
```

---

## 8. What Is a Multi-Task Job?

A multi-task job contains multiple tasks connected through dependencies.

For example:

```text
        Raw Data Ingestion
                 |
                 v
        Data Validation
                 |
                 v
        Data Transformation
                 |
                 v
          Data Quality
                 |
                 v
          Final Output
```

### Example Workflow

| Task | Responsibility |
|---|---|
| Task 1 | Read raw customer data |
| Task 2 | Validate the input |
| Task 3 | Clean and transform records |
| Task 4 | Write the Silver Delta table |
| Task 5 | Validate final output |

The next task can depend on the successful completion of a previous task.

Independent tasks may run in parallel when their dependencies allow it.

**Interview point:** Multi-task jobs help organize complex ETL workflows and define execution dependencies.

---

## 9. Job Scheduling

Databricks Jobs can be executed in different ways depending on the workload and configuration.

Common execution methods include:

### A. Manual Execution

A user starts the job manually.

Useful for:
- Testing.
- Debugging.
- One-time execution.

### B. Scheduled Execution

A job runs according to a configured schedule.

Example:

```text
Daily at 2:00 AM
```

Useful for:
- Daily ETL pipelines.
- Hourly processing.
- Periodic reporting.

### C. Trigger-Based Execution

Jobs can also be started by supported triggers or external orchestration systems.

Examples include:
- File arrival triggers where supported.
- External API or workflow triggers.
- Orchestration from another platform.

**Important:** Trigger availability depends on the workspace, job configuration, and platform capabilities.

---

## 10. Job Parameters and Notebook Widgets

Parameters allow the same notebook to process different inputs without changing its code.

### A. Create a Widget

```python
dbutils.widgets.text(
    "process_date",
    "2026-10-01",
    "Process Date"
)
```

### B. Read the Parameter

```python
process_date = dbutils.widgets.get(
    "process_date"
)

print(process_date)
```

### C. Use the Parameter

```python
from pyspark.sql.functions import col, lit

filtered_df = df.filter(
    col("order_date") == lit(process_date).cast("date")
)
```

### Why Are Parameters Useful?

- Reuse notebooks.
- Process different dates.
- Avoid hardcoded values.
- Support scheduled workflows.
- Simplify development and testing.

**Important:** Notebook widgets and job task parameters are related but not identical. A job can pass parameter values to a notebook task, which the notebook can retrieve through its parameter interface.

---

## 11. Passing Parameters Between Notebooks

A parent notebook can execute a child notebook and pass parameter values.

### Parent Notebook

```python
result = dbutils.notebook.run(
    "/Shared/child_notebook",
    300,
    {
        "process_date": "2026-10-01"
    }
)

print(result)
```

### Child Notebook

```python
dbutils.widgets.text(
    "process_date",
    ""
)

process_date = dbutils.widgets.get(
    "process_date"
)

print(process_date)
```

### Return a Value

In the child notebook:

```python
dbutils.notebook.exit("SUCCESS")
```

In the parent notebook:

```python
print(result)
```

**Important:** `dbutils.notebook.run()` is a notebook orchestration mechanism. For production workflows with multiple dependent tasks, Databricks multi-task Jobs often provide better visibility, monitoring, and dependency management.

---

## 12. Notebook Reusability

Avoid writing the same transformation logic in multiple notebooks.

Instead, create reusable functions.

### Example

```python
from pyspark.sql.functions import col

def clean_customer_data(df):
    return (
        df.filter(
            col("customer_id").isNotNull()
        )
        .dropDuplicates(["customer_id"])
    )
```

Use the function:

```python
cleaned_df = clean_customer_data(
    customer_df
)
```

You can organize reusable logic into Python modules and import them where supported.

### Benefits

- Reduces code duplication.
- Improves maintainability.
- Makes testing easier.
- Encourages consistent business logic.

---

## 13. Job Monitoring

After a job runs, you should monitor its execution.

Important information includes:

| Metric or Status | Meaning |
|---|---|
| Run status | Success, failed, canceled, or other execution state |
| Start time | When execution began |
| Duration | Total execution time |
| Task status | Result of each individual task |
| Error message | Reason for failure |
| Run output | Logs and returned task information |
| Retry count | Number of execution attempts |

### Example

```text
Job: daily_customer_etl

Task 1: ingestion        SUCCESS
Task 2: validation       SUCCESS
Task 3: transformation   SUCCESS
Task 4: data_quality     FAILED
```

In this example, the final data quality task failed.

A Data Engineer should investigate the error before considering the pipeline successful.

---

## 14. Retries, Timeouts, and Notifications

Production jobs need failure-handling mechanisms.

### A. Retries

A task can be configured to retry after failure.

Useful for temporary issues such as:
- Network interruptions.
- Temporary service unavailability.
- Transient infrastructure failures.

**Important:** Retries do not fix permanent data or code errors. Ensure that rerunning a task is safe, especially when it writes data.

### B. Timeout

A timeout limits how long a task or job is allowed to run.

Example:

```text
Task Timeout: 60 minutes
```

### C. Notifications

Notifications can alert the team about:
- Successful execution.
- Failed execution.
- Long-running jobs.
- Other configured job events.

**Interview point:** A production pipeline should have monitoring, failure handling, and appropriate notifications.

---

## 15. Job Compute and Cost Management

Jobs can use configured compute resources depending on the job type and workspace capabilities.

For automated ETL workloads, consider job-oriented compute.

Important practices:

1. Choose suitable compute resources.
2. Avoid unnecessarily large clusters.
3. Use autoscaling when appropriate.
4. Configure timeouts.
5. Avoid unnecessary repeated runs.
6. Monitor job duration and resource consumption.
7. Use efficient Spark transformations.

**Remember:** A faster job is not automatically cheaper in every billing model. Compare actual resource usage and charges.

---

## 16. Production ETL Design Example

Consider an e-commerce company with three layers:

- Bronze: Raw ingested data.
- Silver: Cleaned and validated data.
- Gold: Business-ready aggregated data.

### Job Structure

```text
                 Daily ETL Job
                       |
                       v
              Bronze Ingestion
                       |
                       v
               Silver Cleaning
                       |
                       v
               Data Validation
                       |
                       v
              Gold Aggregation
                       |
                       v
                Final Reporting
```

### Example Task Responsibilities

| Task | Input | Output |
|---|---|---|
| Bronze ingestion | Source files | Bronze Delta table |
| Silver cleaning | Bronze table | Silver Delta table |
| Validation | Silver table | Quality result |
| Gold aggregation | Silver table | Gold Delta table |
| Reporting | Gold table | BI-ready data |

### Important Design Considerations

- Define task dependencies.
- Pass processing dates as parameters.
- Validate input schemas.
- Handle late-arriving data.
- Make writes safe to retry.
- Monitor each task.
- Maintain appropriate permissions.
- Avoid hardcoding credentials.

---

## 17. Common Mistakes

| Mistake | Better Practice |
|---|---|
| Manually running production notebooks every day | Use scheduled jobs |
| Hardcoding processing dates | Use parameters |
| Repeating the same code in multiple notebooks | Create reusable functions or modules |
| Ignoring job failures | Monitor task status and logs |
| Retrying every failure blindly | Identify transient versus permanent failures |
| Using interactive compute for every automated workflow | Evaluate job-oriented compute |
| Ignoring task dependencies | Define a clear multi-task workflow |
| Hardcoding credentials | Use supported secret-management mechanisms |
| Running large notebooks without structure | Divide workflows into maintainable tasks |
| Assuming notebook success means pipeline correctness | Add data quality checks |

---

## 18. Interview Questions

**Q1. What is a Databricks notebook?**

A Databricks notebook is an interactive development environment for writing and executing Python, SQL, Scala, and other supported languages.

**Q2. What is a Databricks Job?**

A Databricks Job automates and orchestrates workloads such as notebooks, Python scripts, and SQL tasks.

**Q3. What is the difference between a notebook and a job?**

A notebook is where code is developed and executed. A job manages automated execution, scheduling, and orchestration of tasks.

**Q4. What is a multi-task job?**

It is a job containing multiple tasks connected through dependencies. Tasks can execute sequentially or in parallel depending on the dependency configuration.

**Q5. How do you schedule a notebook to run every day?**

Create a job, add a notebook task, configure its compute and parameters, and set a daily schedule.

**Q6. What are notebook widgets?**

Widgets are notebook input controls used to receive parameter values such as dates, file paths, or environment names.

**Q7. How do you pass parameters to another notebook?**

Use `dbutils.notebook.run()` with a parameter dictionary, or pass parameters through a configured notebook task in a multi-task job.

**Q8. What is dbutils.notebook.exit()?**

It terminates the current notebook execution and returns a value to the calling notebook when using notebook orchestration.

**Q9. How do you handle job failures?**

Inspect task logs and errors, identify the root cause, configure appropriate retries for transient failures, and use notifications and monitoring.

**Q10. Why should ETL workflows use multiple tasks?**

They provide better separation of responsibilities, dependency management, monitoring, and failure isolation.

**Q11. What is the difference between a scheduled job and a manually triggered job?**

A scheduled job executes according to a configured schedule, while a manually triggered job is started by a user or authorized process.

**Q12. How do you make a notebook reusable?**

Use parameters, reusable functions, and shared Python modules instead of hardcoding values and duplicating logic.

**Q13. How would you design a daily Bronze-to-Silver-to-Gold pipeline?**

I would create separate tasks for ingestion, cleaning, validation, and aggregation. I would define dependencies, pass processing parameters, configure suitable compute, and monitor each task.

**Q14. Why are retries important in production jobs?**

Retries help recover from temporary failures without requiring manual intervention, provided that the operation is safe to execute again.

**Q15. What is the difference between dbutils.notebook.run() and a multi-task job?**

`dbutils.notebook.run()` allows one notebook to invoke another. Multi-task Jobs provide a structured orchestration mechanism with task dependencies, run monitoring, and job-level configuration.

---

## 19. Quick Revision

| Concept | Remember |
|---|---|
| Notebook | Interactive code development and execution |
| Cell | Individual notebook execution unit |
| `%python` | Python cell |
| `%sql` | SQL cell |
| `%md` | Markdown cell |
| `%run` | Execute another notebook |
| `dbutils.fs` | Filesystem utilities |
| `dbutils.widgets` | Notebook parameters |
| `dbutils.notebook.run()` | Invoke another notebook |
| `dbutils.notebook.exit()` | Return a value and exit |
| Job | Automated workload execution |
| Multi-task job | Multiple dependent or independent tasks |
| Schedule | Run workload at configured times |
| Retry | Attempt execution again after failure |
| Timeout | Maximum allowed execution time |
| Monitoring | Track job and task execution |

---

## 20. Self-Assessment

Before moving to the next topic, make sure you can explain:

- [ ] What a Databricks notebook is.
- [ ] How notebook cells work.
- [ ] Common notebook magic commands.
- [ ] What `dbutils` is used for.
- [ ] The difference between notebooks and jobs.
- [ ] How to create a Databricks Job.
- [ ] How multi-task Jobs work.
- [ ] How task dependencies work.
- [ ] How to schedule a daily ETL pipeline.
- [ ] How to pass parameters using widgets.
- [ ] How to execute another notebook.
- [ ] How to monitor job execution.
- [ ] How retries and timeouts work.
- [ ] How to design a production ETL workflow.

**Final takeaway:** A Data Engineer should know how to develop transformations in notebooks and automate them through Databricks Jobs. In production, the important skills are parameterization, task orchestration, dependency management, monitoring, failure handling, and cost control.