# Azure Databricks Architecture and Workspaces

## 1. What Is Azure Databricks?

Azure Databricks is a cloud-based data analytics platform built on Apache Spark.

It provides an environment for:
- Data engineering and ETL/ELT.
- Batch and streaming data processing.
- Data analytics and machine learning.
- Delta Lake table management.
- Collaborative development using notebooks.
- Scheduling and monitoring data workflows.

Azure Databricks combines Apache Spark's distributed processing capabilities with Azure infrastructure and integrations.

**Simple example:** A company stores millions of order records in ADLS Gen2. Databricks reads the data, distributes processing across multiple machines, transforms it using PySpark, and writes the results back as Delta tables.

## 2. High-Level Azure Databricks Architecture

Azure Databricks architecture can be understood through three major areas:

1. Workspace and control plane.
2. Compute and data processing.
3. Data storage and external services.

```text
                  Users
                    |
                    v
          Azure Databricks Workspace
                    |
             Control Plane
          (UI, APIs, Management)
                    |
                    v
             Compute Resources
          (Driver + Executors)
                    |
                    v
             Apache Spark
                    |
                    v
          Data Processing Layer
                    |
                    v
             ADLS Gen2 / Delta
                    |
                    v
          Analytics and Reporting
```

This is a simplified logical view. The actual architecture depends on the workspace's networking, compute type, security configuration, and governance setup.

## 3. Control Plane and Data Plane

One of the most important Azure Databricks architecture concepts is the separation between the control plane and the compute/data plane.

### 3.1 Control Plane

The control plane is managed by Databricks.

It provides workspace management and coordination capabilities.

Common responsibilities include:
- Workspace user interface.
- Notebook and job management.
- API and workspace operations.
- Cluster or compute management.
- Job scheduling and orchestration metadata.
- Configuration and workspace-level services.

The control plane coordinates operations but is not where the main Spark data processing workload runs.

### 3.2 Data Plane

The data plane is where compute resources execute workloads.

Depending on the deployment and compute type, resources may run in the customer's Azure environment or in a Databricks-managed serverless environment.

Common responsibilities include:
- Running Spark jobs.
- Executing PySpark transformations.
- Processing data in parallel.
- Reading and writing data.
- Executing SQL and other supported workloads.

### Control Plane vs Data Plane

| Feature | Control Plane | Data Plane |
|---|---|---|
| Main purpose | Management and coordination | Workload execution |
| Responsibility | Workspace and job management | Data processing |
| Spark transformations | Coordinates workload execution | Executes workload |
| Compute resources | Manages or coordinates compute | Runs compute |
| Data processing | Not the primary execution layer | Main execution layer |

**Interview point:** The control plane manages and coordinates the Databricks environment, while the data plane executes data processing workloads.

## 4. Compute Architecture

Azure Databricks uses compute resources to execute workloads.

For Spark-based workloads, the important components are:

- Driver.
- Executors.
- Cluster manager.
- Worker nodes.

### 4.1 Driver

The driver is the main process responsible for coordinating a Spark application.

Responsibilities:
- Creates the Spark session and Spark context.
- Builds execution plans.
- Coordinates jobs, stages, and tasks.
- Schedules tasks for execution.
- Tracks application progress.
- Collects results when requested.

Example:

```python
df = spark.read.parquet("/mnt/data/orders")

result = df.groupBy("customer_id").count()

display(result)
```

The driver coordinates the execution of this DataFrame workload.

### 4.2 Executors

Executors are processes that run on worker nodes.

Responsibilities:
- Execute assigned tasks.
- Process partitions of data.
- Store cached or persisted data.
- Send task results and status information to the driver.

A Spark application can have multiple executors working in parallel.

### 4.3 Worker Nodes

Worker nodes provide compute resources where executor processes run.

They contribute:
- CPU.
- Memory.
- Storage resources for local processing.
- Parallel task execution capacity.

### 4.4 Cluster Manager

The cluster manager is responsible for allocating compute resources.

In a Spark deployment, resource allocation and lifecycle management depend on the platform. Databricks manages much of this infrastructure for users.

**Important:** Do not assume every Databricks compute type has exactly the same infrastructure or cluster-management behavior.

## 5. How a Spark Job Runs in Databricks

Consider this PySpark operation:

```python
df = spark.read.parquet("/mnt/data/orders")

result = (
    df.filter(df.amount > 1000)
      .groupBy("customer_id")
      .sum("amount")
)

result.show()
```

Execution flow:

1. The user submits code through a notebook or job.
2. The driver receives and processes the application instructions.
3. Spark builds a logical execution plan.
4. The Catalyst Optimizer optimizes the plan.
5. Spark creates a physical execution plan.
6. The scheduler divides the work into stages and tasks.
7. Executors process partitions in parallel.
8. The driver coordinates execution and receives task results.
9. The final result is returned or written to storage.

The exact execution plan depends on the query, data, statistics, and runtime configuration.

## 6. What Is an Azure Databricks Workspace?

A workspace is a collaborative environment where users develop, run, and manage Databricks workloads.

It provides access to:
- Notebooks.
- Jobs and workflows.
- Compute resources.
- SQL editors and dashboards.
- Repos and Git integration.
- Data and catalog exploration.
- Workspace permissions and administration.

A workspace is not the same as a compute cluster.

- Workspace: Development and management environment.
- Compute: Resources used to execute workloads.

A workspace can contain or access multiple compute resources, depending on permissions and configuration.

## 7. Workspace Components

### 7.1 Notebooks

Notebooks are interactive development environments.

They support languages such as:
- Python.
- SQL.
- Scala.
- R.

Example:

```python
from pyspark.sql.functions import col

df = spark.read.parquet("/mnt/data/orders")

filtered_df = df.filter(col("amount") > 1000)

display(filtered_df)
```

Notebooks are useful for:
- Data exploration.
- Transformation development.
- Testing.
- Debugging.
- Interactive analysis.

### 7.2 Jobs and Workflows

Jobs and Workflows are used to automate and orchestrate tasks.

A workflow can contain multiple tasks with dependencies.

Example:

```text
Ingest Data
     |
     v
Clean Data
     |
     v
Transform Data
     |
     v
Validate Data
     |
     v
Load Gold Tables
```

Tasks can run notebooks, Python scripts, SQL, and other supported task types.

### 7.3 Compute

Compute resources provide the environment for executing notebooks, jobs, and queries.

Common compute categories include:
- All-purpose compute.
- Jobs compute.
- SQL warehouses.
- Serverless compute options.

Availability and behavior depend on the workspace, cloud, region, and Databricks configuration.

### 7.4 Repos and Git Integration

Databricks supports Git integration for source-code version control.

Developers can:
- Connect a Git repository.
- Work with branches.
- Commit changes.
- Pull updates.
- Collaborate through version control.

Git integration helps manage code changes across development environments.

### 7.5 Catalog Explorer

Catalog Explorer provides an interface for discovering and managing governed data assets.

With Unity Catalog, users can explore:
- Catalogs.
- Schemas.
- Tables.
- Views.
- Volumes.
- External locations.
- Permissions.

## 8. Types of Databricks Compute

Compute is selected based on the workload.

### 8.1 All-Purpose Compute

All-purpose compute is commonly used for interactive development.

Use cases:
- Notebook development.
- Data exploration.
- Testing PySpark transformations.
- Collaborative development.

### 8.2 Jobs Compute

Jobs compute is designed for automated workloads.

Use cases:
- Scheduled ETL pipelines.
- Batch processing.
- Production data transformations.
- Workflow execution.

### 8.3 SQL Warehouses

SQL warehouses provide compute for SQL workloads.

Use cases:
- SQL queries.
- BI dashboards.
- Data analysis.
- Reporting tools.

### Comparison

| Compute type | Typical purpose |
|---|---|
| All-purpose | Interactive development |
| Jobs compute | Automated data processing |
| SQL warehouse | SQL analytics and BI |

The exact compute options and their lifecycle behavior can vary with the Databricks product and workspace configuration.

## 9. Classic Compute vs Serverless Compute

Databricks provides different ways to provision compute.

### Classic Compute

Classic compute uses compute resources provisioned in the customer's cloud environment, subject to the selected deployment and configuration.

Characteristics:
- Customer cloud networking configuration is relevant.
- Compute resources are provisioned for workloads.
- The customer has more control over certain infrastructure settings.

### Serverless Compute

Serverless compute allows Databricks to manage more of the underlying compute infrastructure.

Characteristics:
- Faster and simpler compute provisioning for supported workloads.
- Less infrastructure management for users.
- Databricks manages the serverless compute environment.
- Availability and supported features depend on workload and region.

### Comparison

| Feature | Classic compute | Serverless compute |
|---|---|---|
| Infrastructure management | More customer configuration | More managed by Databricks |
| Provisioning | Depends on configured resources | Managed provisioning |
| Networking | Customer cloud networking is relevant | Uses serverless networking configuration |
| Flexibility | More infrastructure control | Simplified operations |
| Availability | Depends on configuration | Depends on supported regions and workloads |

Do not assume serverless is available for every workload or that it removes all networking and governance requirements.

## 10. Workspace Organization

A workspace can organize development assets using folders and permissions.

Example:

```text
Workspace
│
├── Shared
│   ├── Common Utilities
│   └── Shared Notebooks
│
├── Users
│   ├── Developer A
│   │   ├── Development
│   │   └── Testing
│   │
│   └── Developer B
│       ├── Development
│       └── Testing
│
└── Production
    ├── Ingestion
    ├── Transformation
    └── Validation
```

This is an illustrative organization. Actual workspace folder structures depend on the team's development and deployment practices.

### Recommended organization

- Keep development and production code separate.
- Use Git for version control.
- Avoid storing credentials in notebooks.
- Use reusable functions and shared libraries.
- Apply appropriate folder and workspace permissions.
- Deploy production workflows through controlled processes.

## 11. Workspace vs Cluster vs Notebook vs Job

These terms are frequently confused.

| Component | Meaning |
|---|---|
| Workspace | Collaborative Databricks environment |
| Notebook | Interactive code and documentation |
| Compute | Resources that execute workloads |
| Cluster | A type of Spark compute resource |
| Job | Automated execution of one or more tasks |
| Workflow | Orchestration of tasks and dependencies |

Example:

A developer opens a notebook in a workspace, attaches it to compute, tests a transformation, and then schedules it as part of a production workflow.

## 12. Databricks and Azure Services Integration

Azure Databricks integrates with several Azure services.

| Azure service | Purpose |
|---|---|
| ADLS Gen2 | Data lake storage |
| Azure Data Factory | Data ingestion and orchestration |
| Microsoft Entra ID | Identity and authentication |
| Azure Key Vault | Secret management |
| Azure Monitor | Monitoring and diagnostics |
| Azure Event Hubs | Event ingestion and streaming |
| Azure Synapse Analytics | Analytics and data integration |

### Example architecture

```text
          Source Systems
                 |
                 v
        Azure Data Factory
                 |
                 v
            ADLS Gen2
                 |
                 v
        Azure Databricks
          (PySpark / SQL)
                 |
                 v
            Delta Lake
                 |
                 v
        Unity Catalog
                 |
                 v
         SQL / BI Tools
```

In this architecture:
- ADF orchestrates ingestion.
- ADLS Gen2 stores data.
- Databricks processes data.
- Delta Lake manages reliable tables.
- Unity Catalog governs data access.
- BI tools consume curated datasets.

## 13. Security and Networking

Databricks security involves multiple layers.

### Identity

Microsoft Entra ID can provide identity and authentication capabilities.

### Workspace access

Workspace permissions control who can access workspace objects and perform workspace operations.

### Compute permissions

Compute access controls determine who can use or manage particular compute resources.

### Data permissions

Unity Catalog provides centralized permissions for governed data objects.

### Network security

Depending on deployment, security can involve:
- Virtual networks.
- Network security groups.
- Private endpoints.
- Firewall rules.
- Private DNS.
- Secure connectivity to storage and other services.

**Important:** Workspace access does not automatically mean access to every data asset. Workspace permissions, compute permissions, network access, and data permissions are separate concerns.

## 14. High Availability and Reliability Considerations

A production Databricks architecture should consider:

- Job retries and failure handling.
- Idempotent data writes.
- Checkpointing for supported streaming workloads.
- Monitoring and alerting.
- Data quality validation.
- Recovery and backfill strategies.
- Appropriate storage redundancy.
- Secure access and credential management.

A multi-node compute configuration can improve parallel processing and provide resilience against some worker failures. It does not by itself guarantee end-to-end pipeline availability.

## 15. Cost Considerations

Databricks cost depends on the compute and services used.

Important factors include:
- Databricks Units (DBUs).
- Underlying Azure infrastructure costs.
- Compute type and size.
- Runtime and workload duration.
- Autoscaling and auto-termination configuration.
- SQL warehouse usage.
- Storage and data transfer.
- Network architecture.

**DBU is not the same as the underlying Azure VM cost.** Actual billing depends on the selected product, cloud, region, and contract.

### Cost optimization practices

- Use job-specific compute for automated workloads where appropriate.
- Enable auto-termination for supported interactive compute.
- Right-size compute resources.
- Avoid unnecessarily long-running clusters.
- Optimize Spark transformations and data layouts.
- Use appropriate file sizes and table organization.
- Monitor compute usage and job duration.
- Review both Databricks and Azure infrastructure charges.

## 16. Common Architecture Mistakes

1. Confusing workspace with compute.
2. Assuming the control plane executes Spark transformations.
3. Assuming all compute types have identical infrastructure.
4. Treating ADLS Gen2 and Delta Lake as the same technology.
5. Assuming workspace access automatically grants data access.
6. Storing credentials directly in notebooks.
7. Using interactive compute for every production workload without considering cost and lifecycle.
8. Ignoring network and private endpoint configuration.
9. Assuming autoscaling guarantees better performance or lower cost.
10. Assuming a multi-node cluster alone guarantees pipeline reliability.

## 17. Common Interview Questions and Answers

### Q1. What is Azure Databricks?

Azure Databricks is a cloud-based analytics platform built on Apache Spark. It supports distributed data processing, ETL, streaming, SQL analytics, and machine learning.

### Q2. Explain the architecture of Azure Databricks.

The architecture can be understood through the control plane and data plane. The control plane manages workspace operations and coordinates workloads, while compute resources in the data plane execute Spark processing. Data is commonly stored in services such as ADLS Gen2.

### Q3. What is the difference between the control plane and data plane?

The control plane handles workspace management and coordination. The data plane runs the actual compute workloads and processes data.

### Q4. What is a Databricks workspace?

A workspace is a collaborative environment containing notebooks, jobs, compute access, repositories, and other development and management tools.

### Q5. What is the difference between a workspace and a cluster?

A workspace is the environment used to develop and manage workloads. A cluster is a compute resource used to execute Spark workloads.

### Q6. What is the role of the driver?

The driver coordinates the Spark application. It builds execution plans, schedules tasks, tracks execution, and coordinates executors.

### Q7. What is the role of executors?

Executors execute tasks assigned by the driver, process data partitions, and may store cached data.

### Q8. What is the difference between all-purpose and jobs compute?

All-purpose compute is generally used for interactive development, while jobs compute is designed for automated workloads.

### Q9. What is a SQL warehouse?

A SQL warehouse is compute designed to execute SQL queries and support analytics and BI workloads.

### Q10. What is serverless compute?

Serverless compute is a managed compute option where Databricks handles more of the underlying infrastructure provisioning and management for supported workloads.

### Q11. How does Databricks integrate with ADLS Gen2?

Databricks uses the ABFS connector to read and write data in ADLS Gen2. Authentication can use managed identities, service principals, or Unity Catalog storage credentials, depending on the architecture.

### Q12. What is Unity Catalog?

Unity Catalog is Databricks' centralized governance solution for managing data access, discovery, permissions, and lineage across supported workspaces.

### Q13. How would you design a production data engineering architecture?

I would use ADF or another ingestion mechanism to load source data into ADLS Gen2. Databricks would process the data through Bronze, Silver, and Gold layers using Delta Lake. Unity Catalog would manage governance, and workflows and monitoring would support reliable execution.

### Q14. Does the control plane store all processed business data?

No. The control plane primarily manages and coordinates the Databricks environment. Business data is commonly stored in external cloud storage such as ADLS Gen2, although platform-managed storage and metadata also exist.

### Q15. Does a Databricks workspace automatically provide access to ADLS Gen2?

No. The identity and compute environment must have the necessary network connectivity and storage permissions. Unity Catalog can provide governed access through storage credentials and external locations.

## 18. Quick Revision

| Topic | Key point |
|---|---|
| Azure Databricks | Cloud analytics platform built on Spark |
| Workspace | Collaborative development and management environment |
| Control plane | Workspace management and coordination |
| Data plane | Compute and workload execution |
| Driver | Coordinates Spark application |
| Executor | Executes tasks |
| Worker node | Provides compute resources |
| Notebook | Interactive development interface |
| Job | Automated task execution |
| Workflow | Orchestrates multiple tasks |
| All-purpose compute | Interactive development |
| Jobs compute | Automated workloads |
| SQL warehouse | SQL analytics |
| Serverless | Databricks-managed compute for supported workloads |
| ADLS Gen2 | Cloud data storage |
| Delta Lake | Reliable table framework |
| Unity Catalog | Centralized data governance |

## 19. Self-Assessment

Try answering these questions without looking at the notes:

1. Explain Azure Databricks architecture.
2. Differentiate control plane and data plane.
3. What is a workspace?
4. Differentiate workspace, notebook, compute, and job.
5. Explain the roles of driver and executors.
6. How does a Spark job execute in Databricks?
7. Compare all-purpose compute, jobs compute, and SQL warehouses.
8. What is the difference between classic and serverless compute?
9. How do ADLS Gen2 and Unity Catalog fit into the architecture?
10. What security layers should be considered in a production workspace?
11. What factors affect Databricks cost?
12. Design a reliable Bronze-to-Silver-to-Gold architecture.

**Interview readiness target:** You should be able to explain how a Databricks workspace coordinates development and execution, how Spark compute processes data, and how ADLS Gen2 and Unity Catalog fit into a secure production architecture.