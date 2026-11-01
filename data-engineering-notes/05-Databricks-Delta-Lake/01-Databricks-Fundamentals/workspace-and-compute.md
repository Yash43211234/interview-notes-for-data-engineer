# Databricks Workspace and Compute

## 1. What Is a Databricks Workspace?

A Databricks workspace is an environment where data engineers, data analysts, and data scientists collaborate to build and manage data solutions.

It provides access to resources such as:

- Notebooks
- Data and files
- Compute resources
- Jobs and workflows
- SQL warehouses
- Git folders
- Catalog and data objects
- Dashboards and queries

**Example:**

A data engineering team can use one Databricks workspace to develop notebooks, execute ETL pipelines, manage tables, and monitor scheduled Jobs.

**Remember:** A workspace is the working environment. Compute provides the processing resources used to execute workloads.

## 2. Workspace Structure

A Databricks workspace organizes project resources.

A simplified structure:

```text
Databricks Workspace
        |
        +-- Users
        |
        +-- Shared
        |
        +-- Git Folders
        |
        +-- Notebooks
        |
        +-- Jobs and Workflows
        |
        +-- SQL
        |
        +-- Compute
        |
        +-- Catalog Explorer
```

The exact workspace layout and available features depend on the Databricks edition, workspace configuration, and permissions.

## 3. What Is Compute in Databricks?

Compute refers to the processing resources used to execute workloads.

When we run a PySpark notebook, Spark needs resources to process data.

These resources include:

- CPU
- Memory
- Executors
- Driver
- Worker nodes

Compute is responsible for executing operations such as:

- Reading data
- Transforming DataFrames
- Performing joins
- Aggregating records
- Writing data to Delta tables

**Example:**

```python
df = spark.read.format("delta").load("/mnt/sales")

result = df.groupBy("region").sum("amount")

display(result)
```

The Spark compute resources execute the transformations and return the result.

## 4. Driver and Worker Nodes

A Spark cluster commonly consists of a driver and worker nodes.

### Driver

The driver is responsible for coordinating the Spark application.

Main responsibilities:

- Creates the Spark session and context.
- Builds execution plans.
- Coordinates job execution.
- Schedules tasks.
- Tracks application progress.
- Collects results returned by tasks.

### Worker Nodes

Worker nodes provide resources for executing tasks.

They run executors that process data partitions.

Main responsibilities:

- Execute tasks.
- Process data partitions.
- Perform transformations.
- Store cached data.
- Exchange data during shuffle operations.

### Architecture

```text
             Spark Application
                     |
                     v
                  Driver
                     |
          Schedules and Coordinates
                     |
          +----------+----------+
          |                     |
          v                     v
       Worker 1              Worker 2
          |                     |
       Executor              Executor
          |                     |
       Tasks                 Tasks
          |                     |
          v                     v
      Data Partitions       Data Partitions
```

**Remember:**

- Driver = Coordinates execution.
- Executors = Execute tasks and process data.
- Workers = Machines that host executors.

## 5. What Is a Cluster?

A cluster is a group of computing resources working together to execute distributed workloads.

In Spark, a cluster generally includes:

- One driver node.
- One or more worker nodes.
- Executors running on worker nodes.

The number and size of nodes depend on the compute configuration.

### Example

Suppose we process a large orders dataset.

Instead of processing all records on one machine, Spark divides the data into partitions and processes them across multiple executors.

This enables distributed processing.

## 6. Types of Databricks Compute

Databricks provides different compute options for different workloads.

Common categories include:

1. All-purpose compute
2. Job compute
3. SQL warehouses
4. Serverless compute

Availability and exact behavior depend on the workspace, cloud provider, and Databricks configuration.

### 6.1 All-Purpose Compute

All-purpose compute is mainly used for interactive development.

Use cases:

- Running notebooks manually.
- Testing PySpark code.
- Exploring datasets.
- Debugging transformations.
- Developing ETL logic.

Example:

A data engineer opens a notebook, attaches it to all-purpose compute, and tests a join operation.

**Remember:** All-purpose compute is commonly used during development and exploration.

### 6.2 Job Compute

Job compute is intended for automated workloads.

Use cases:

- Scheduled ETL pipelines.
- Daily data ingestion.
- Data transformation.
- Data quality validation.
- Production batch processing.

Example:

```text
Daily Orders Pipeline
        |
        v
   Job Compute
        |
        v
 Ingestion Task
        |
        v
Transformation Task
        |
        v
 Validation Task
```

Job compute can be configured for automated execution and managed according to the Job's lifecycle.

### 6.3 SQL Warehouse

A SQL warehouse provides compute resources for SQL workloads.

Common use cases:

- Running SQL queries.
- Data analysis.
- Dashboards.
- BI tools.
- SQL-based reporting.

Example:

```sql
SELECT
    region,
    SUM(amount) AS total_sales
FROM sales
GROUP BY region;
```

The SQL warehouse executes the query.

**Interview point:** A SQL warehouse is designed for SQL analytics, while Spark compute is commonly used for distributed data engineering workloads. Their capabilities can overlap depending on the environment.

### 6.4 Serverless Compute

Serverless compute allows users to run supported workloads without manually managing the underlying compute infrastructure.

Databricks manages the infrastructure and resource provisioning for supported serverless workloads.

Benefits:

- Less infrastructure management.
- Simplified compute setup.
- Faster access to supported workloads.
- Automatic resource management.

**Important:** Serverless availability, supported workloads, networking, and pricing depend on the workspace and cloud configuration.

## 7. All-Purpose Compute vs. Job Compute vs. SQL Warehouse

| Feature | All-Purpose | Job Compute | SQL Warehouse |
|---|---|---|---|
| Main purpose | Interactive development | Automated workloads | SQL analytics |
| Common users | Developers and data engineers | Data engineering teams | Analysts and BI users |
| Typical workload | Notebook execution | Scheduled ETL | SQL queries and dashboards |
| Execution | Usually interactive | Job-triggered | Query-triggered |
| Cost management | Monitor usage and termination | Configure suitable job resources | Configure warehouse sizing and auto-stop |

**Quick understanding:**

- All-purpose compute → Development.
- Job compute → Automated pipelines.
- SQL warehouse → SQL analytics.

## 8. Cluster Configuration

When creating compute, we configure resources according to workload requirements.

Important configuration options include:

| Configuration | Meaning |
|---|---|
| Runtime version | Databricks Runtime and supported Spark environment |
| Node type | Machine type with specific CPU and memory |
| Driver type | Resources allocated to the driver |
| Worker type | Resources allocated to workers |
| Minimum workers | Minimum worker count for autoscaling |
| Maximum workers | Maximum worker count for autoscaling |
| Auto-termination | Stops idle compute after a configured period |
| Access mode | Controls how users and workloads access compute |

The exact settings available depend on the compute type and workspace permissions.

## 9. Autoscaling

Autoscaling automatically adjusts the number of worker nodes according to workload demand and configured limits.

Example:

```text
Minimum Workers = 2
Maximum Workers = 8

Low Workload:
2 Workers

High Workload:
6 Workers

Workload Decreases:
Workers can scale down
```

### Benefits

- Adjusts resources based on workload demand.
- Can reduce unnecessary resource usage.
- Helps handle changing processing requirements.

**Important:** Autoscaling does not guarantee better performance or lower cost. Scaling decisions depend on workload characteristics, configuration, and resource availability.

## 10. Auto-Termination

Auto-termination stops compute after it has been idle for a configured period.

Example:

```text
Auto-Termination = 20 minutes

Last Activity:
10:00 AM

No Activity:
10:00 AM - 10:20 AM

Compute Stops:
Around 10:20 AM
```

### Why is it useful?

- Prevents compute from running unnecessarily.
- Helps control costs.
- Useful for interactive development environments.

**Remember:** Auto-termination is different from autoscaling.

- Autoscaling → Changes the number of workers.
- Auto-termination → Stops idle compute.

## 11. Databricks Runtime

Databricks Runtime is the execution environment used by Databricks compute.

It includes Spark and additional components, optimizations, and integrations provided for the selected runtime.

Runtime versions may include:

- Apache Spark
- Python and supporting libraries
- Delta Lake components
- Databricks-specific optimizations
- Other supported libraries

### Why does Runtime Version Matter?

Different runtime versions can have different:

- Spark versions.
- Python versions.
- Library compatibility.
- Features and optimizations.
- Bug fixes and security updates.

**Example:**

If a project depends on a particular library version or Spark feature, the runtime must support it.

**Interview point:** Always check runtime and library compatibility before upgrading production compute.

## 12. Photon

Photon is Databricks' native vectorized query engine designed to accelerate supported workloads.

It can improve performance for supported SQL and DataFrame operations.

Benefits may include:

- Faster query execution.
- Improved CPU utilization.
- Better performance for supported operations.

**Important:** Photon does not accelerate every operation. Actual performance depends on the workload, supported operators, data format, and configuration.

## 13. Compute Selection: Practical Scenarios

| Scenario | Suitable Compute |
|---|---|
| Developing a PySpark notebook | All-purpose compute |
| Testing a transformation manually | All-purpose compute |
| Running a scheduled daily ETL pipeline | Job compute |
| Executing SQL analytics | SQL warehouse |
| Building dashboards | SQL warehouse |
| Running a supported workload without managing infrastructure | Serverless compute |

The best choice depends on workload requirements, workspace capabilities, and cost.

## 14. Compute Performance Considerations

Choosing compute is only one part of performance optimization.

Important factors include:

1. Dataset size.
2. Number of partitions.
3. Shuffle operations.
4. Join strategy.
5. Data skew.
6. Executor memory.
7. CPU resources.
8. File sizes.
9. Storage format.
10. Runtime and engine optimizations.

### Example

If a job processes a very large dataset and performs expensive joins, increasing compute resources alone may not solve the problem.

We should also investigate:

- Shuffle volume.
- Data skew.
- Join strategy.
- Partitioning.
- Query execution plan.

**Remember:** Good Spark code and suitable compute configuration work together.

## 15. Databricks Cost Management

Compute resources contribute to Databricks and underlying cloud infrastructure costs.

Important cost-management practices:

1. Select compute based on workload size.
2. Enable auto-termination where appropriate.
3. Avoid unnecessary compute startup and idle time.
4. Monitor job execution duration.
5. Optimize expensive transformations.
6. Avoid repeatedly processing unchanged data.
7. Review SQL warehouse sizing and auto-stop settings.
8. Monitor usage and billing information.

### What Is a DBU?

DBU stands for Databricks Unit.

It is a unit of processing capability used for Databricks usage and billing.

**Important:** DBU charges are not necessarily the complete cost of running a workload. Underlying cloud infrastructure charges may also apply. Actual pricing depends on the cloud provider, region, product, and contract.

## 16. Common Interview Questions

### Q1. What is a Databricks workspace?

A workspace is an environment where users manage notebooks, compute, Jobs, SQL resources, and other Databricks project resources.

### Q2. What is compute in Databricks?

Compute provides processing resources such as CPU and memory to execute workloads.

### Q3. What is the difference between a driver and an executor?

The driver coordinates Spark application execution, while executors process data partitions and execute tasks.

### Q4. What is a cluster?

A cluster is a group of computing resources used to execute distributed workloads.

### Q5. What is the difference between all-purpose and job compute?

All-purpose compute is commonly used for interactive development, while job compute is intended for automated workloads.

### Q6. What is a SQL warehouse?

A SQL warehouse provides compute resources for SQL queries, analytics, and dashboards.

### Q7. What is serverless compute?

Serverless compute allows supported workloads to run without users manually managing the underlying infrastructure.

### Q8. What is autoscaling?

Autoscaling automatically adjusts the number of worker nodes based on workload demand and configured limits.

### Q9. What is auto-termination?

Auto-termination stops idle compute after a configured period to avoid unnecessary resource usage.

### Q10. What is Databricks Runtime?

It is the execution environment containing Spark and other components, libraries, and Databricks-specific capabilities.

### Q11. What is Photon?

Photon is Databricks' native vectorized query engine that accelerates supported workloads.

### Q12. What is a DBU?

A DBU is a unit used to measure Databricks usage for billing.

### Q13. Does increasing cluster size always improve performance?

No. Performance also depends on data partitioning, joins, shuffles, skew, query plans, and resource utilization.

### Q14. Which compute would you choose for a daily ETL pipeline?

Job compute is a common choice because it is designed for automated workload execution. The exact choice depends on workload and workspace capabilities.

## 17. Quick Revision

| Concept | Remember |
|---|---|
| Workspace | Databricks working environment |
| Compute | Resources used to execute workloads |
| Driver | Coordinates Spark execution |
| Executor | Executes tasks and processes data |
| Worker | Machine hosting executors |
| Cluster | Group of compute resources |
| All-purpose | Interactive development |
| Job compute | Automated workloads |
| SQL warehouse | SQL analytics |
| Serverless | Managed infrastructure for supported workloads |
| Autoscaling | Adjusts worker count |
| Auto-termination | Stops idle compute |
| Runtime | Spark and supporting execution environment |
| Photon | Accelerates supported workloads |
| DBU | Databricks usage and billing unit |

## 18. Self-Assessment

Before moving to the next topic, make sure you can explain:

- [ ] What is a Databricks workspace?
- [ ] What is compute?
- [ ] What is the difference between driver and executor?
- [ ] What is a cluster?
- [ ] What are the different types of Databricks compute?
- [ ] What is the difference between all-purpose and job compute?
- [ ] What is a SQL warehouse?
- [ ] What is serverless compute?
- [ ] What is autoscaling?
- [ ] What is auto-termination?
- [ ] What is Databricks Runtime?
- [ ] What is Photon?
- [ ] What is a DBU?
- [ ] How would you choose compute for a production ETL pipeline?

**Interview readiness:** You should be able to explain how Databricks workspace resources, Spark architecture, compute types, and configuration choices work together to execute and optimize a production data engineering workload.