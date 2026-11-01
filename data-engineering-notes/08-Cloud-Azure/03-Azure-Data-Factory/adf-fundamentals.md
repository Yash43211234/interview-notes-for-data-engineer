# Azure Data Factory Fundamentals

## 1. What Is Azure Data Factory?

**Azure Data Factory (ADF)** is a cloud-based data integration and orchestration service provided by Microsoft Azure.

It helps build, schedule, and manage data pipelines that move and transform data between different systems.

ADF is commonly used for:

- Extracting data from different sources.
- Loading data into data lakes and databases.
- Orchestrating data transformation workflows.
- Scheduling data pipelines.
- Monitoring pipeline execution.
- Managing dependencies between pipeline activities.

### Example

Suppose a company stores customer data in SQL Server and wants to load it into Azure Data Lake Storage Gen2.

ADF can coordinate the process:

```text
SQL Server
    |
    v
Azure Data Factory
    |
    v
Copy Activity
    |
    v
ADLS Gen2
    |
    v
Databricks Transformation
    |
    v
Processed Data
```

ADF manages the workflow, while the connected services perform the actual data storage and processing.

---

## 2. Why Do We Need Azure Data Factory?

In a data engineering project, data may come from multiple sources:

- SQL Server.
- Azure SQL Database.
- REST APIs.
- CSV and JSON files.
- On-premises databases.
- Cloud storage.
- Other cloud platforms.

Managing all these operations manually is difficult.

ADF helps automate and coordinate these processes.

### Main Benefits

1. **Data Integration:** Connects different data sources and destinations.
2. **Automation:** Runs pipelines automatically.
3. **Scheduling:** Executes pipelines at configured times.
4. **Orchestration:** Controls the order of activities.
5. **Monitoring:** Tracks pipeline and activity execution.
6. **Scalability:** Supports integration workloads across different environments.
7. **Connectivity:** Supports cloud and on-premises data sources through appropriate connectors and Integration Runtimes.

---

## 3. Core Components of Azure Data Factory

The main ADF components are:

1. Pipeline.
2. Activity.
3. Linked Service.
4. Dataset.
5. Integration Runtime.
6. Trigger.
7. Parameters and Variables.

### 3.1 Pipeline

A pipeline is a logical group of activities that performs a data integration workflow.

Example:

```text
Pipeline: CustomerDataPipeline

    |
    v
Copy Customer Data
    |
    v
Validate Data
    |
    v
Run Databricks Notebook
    |
    v
Load Curated Data
```

A pipeline can contain one activity or multiple activities.

### 3.2 Activity

An activity is an individual task inside a pipeline.

Common activities:

| Activity | Purpose |
|---|---|
| Copy | Moves data |
| Lookup | Retrieves data or configuration |
| Get Metadata | Retrieves file or dataset metadata |
| ForEach | Iterates over a collection |
| If Condition | Executes a conditional branch |
| Execute Pipeline | Runs another pipeline |
| Stored Procedure | Executes a database procedure |
| Databricks Notebook | Runs a Databricks notebook |
| Delete | Deletes files or folders |
| Web | Calls a web endpoint |

### 3.3 Linked Service

A Linked Service defines the connection information required to connect to an external service or data store.

Examples:

- Azure SQL Database.
- ADLS Gen2.
- Azure Blob Storage.
- SQL Server.
- Azure Databricks.

Think of a Linked Service as the connection configuration.

Example:

```text
Linked Service: AzureSQL_Connection
    |
    v
Connects to Azure SQL Database
```

A Linked Service typically contains connection details and authentication configuration.

It does not describe the structure of an individual file or table.

### 3.4 Dataset

A Dataset represents the data structure or location that an activity uses.

Examples:

- A CSV file in ADLS Gen2.
- A table in Azure SQL Database.
- A JSON file in Blob Storage.

Example:

```text
Linked Service: ADLS_Connection
             |
             v
Dataset: CustomerCSV
             |
             v
Container: raw
Folder: customers
File: customers.csv
```

The Linked Service defines the connection, while the Dataset identifies the relevant data.

**Important:** A dataset is a logical representation of data. It does not necessarily represent the entire physical data store.

### 3.5 Integration Runtime

Integration Runtime (IR) is the compute infrastructure used by ADF to provide data integration capabilities.

It handles activities such as:

- Data movement.
- Dispatching transformation activities.
- Connecting to data sources.
- Executing supported integration operations.

There are three main Integration Runtime types.

#### Azure Integration Runtime

Used for cloud-based data integration.

Common use cases:

- Copying data between cloud services.
- Executing supported data flows.
- Connecting to supported cloud data stores.

#### Self-hosted Integration Runtime

Used when ADF needs connectivity to data sources inside a private network or on-premises environment.

Example:

```text
On-Premises SQL Server
         |
         v
Self-hosted Integration Runtime
         |
         v
Azure Data Factory
         |
         v
ADLS Gen2
```

The self-hosted IR is installed on a machine that can reach the private data source.

#### Azure-SSIS Integration Runtime

Used to run existing SQL Server Integration Services (SSIS) packages in Azure Data Factory.

It is useful when an organization is migrating SSIS workloads to Azure.

### Integration Runtime Comparison

| IR Type | Main Purpose |
|---|---|
| Azure IR | Cloud data integration |
| Self-hosted IR | Private network and on-premises connectivity |
| Azure-SSIS IR | Execute SSIS packages in Azure |

**Interview point:** Integration Runtime is not the same as a dataset or linked service. It provides the execution and connectivity environment for supported activities.

### 3.6 Trigger

A trigger starts a pipeline based on a configured condition.

Common trigger types:

- Schedule trigger.
- Tumbling window trigger.
- Event-based trigger.

Example:

```text
Schedule Trigger
       |
       | Every day at 2 AM
       v
CustomerDataPipeline
```

A trigger starts a pipeline; it does not define the activities inside it.

### 3.7 Parameters and Variables

Parameters and variables help make pipelines dynamic.

**Parameters:**
- Pass values into a pipeline or activity.
- Useful for reusable pipelines.
- Generally treated as read-only during a pipeline run.

**Variables:**
- Store values during pipeline execution.
- Can be changed using supported variable activities.
- Useful for temporary workflow state.

Example:

```text
Pipeline Parameter: tableName = customers
```

Dynamic expression:

```text
@pipeline().parameters.tableName
```

---

## 4. Linked Service vs Dataset vs Integration Runtime

This is one of the most important ADF interview topics.

| Component | Meaning | Example |
|---|---|---|
| Linked Service | Defines connection information | ADLS connection |
| Dataset | Identifies data structure or location | `customers.csv` |
| Integration Runtime | Provides execution and connectivity infrastructure | Azure IR |

### Simple Analogy

Think about reading a file from a storage account.

- **Linked Service:** How to connect to the storage account.
- **Dataset:** Which file to read.
- **Integration Runtime:** Which integration environment handles the operation.

### Example

```text
Azure Data Factory
       |
       v
Linked Service
(ADLS Connection)
       |
       v
Dataset
(raw/customers.csv)
       |
       v
Copy Activity
       |
       v
Integration Runtime
       |
       v
Destination
```

The exact runtime used depends on the activity, connector, and configuration.

---

## 5. How Azure Data Factory Works

A typical ADF workflow follows these steps:

1. Create an Azure Data Factory instance.
2. Configure Linked Services.
3. Create Datasets where required.
4. Build a Pipeline.
5. Add Activities.
6. Configure activity dependencies.
7. Select or configure the required Integration Runtime.
8. Configure Parameters and Variables.
9. Publish the pipeline.
10. Run the pipeline manually or through a Trigger.
11. Monitor execution and troubleshoot failures.

### Example Workflow

Suppose you want to copy customer data from Azure SQL Database to ADLS Gen2.

```text
Azure SQL Database
        |
        v
ADF Pipeline
        |
        v
Copy Activity
        |
        v
ADLS Gen2
        |
        v
customers.csv
```

ADF coordinates the operation using the configured connections, datasets, and Integration Runtime.

---

## 6. Data Movement vs Data Transformation vs Orchestration

These three concepts are important in data engineering.

### Data Movement

Moving data from one location to another.

Example:

```text
Azure SQL Database → ADLS Gen2
```

ADF commonly uses Copy Activity for this.

### Data Transformation

Changing data into the required structure or format.

Examples:

- Removing duplicates.
- Handling NULL values.
- Joining datasets.
- Aggregating records.
- Applying business rules.

Transformation can be performed using:

- Mapping Data Flows.
- Azure Databricks.
- Azure SQL.
- Other supported processing services.

### Orchestration

Managing the order and execution of different tasks.

Example:

```text
Copy Data
    |
    v
Run Databricks Notebook
    |
    v
Validate Data
    |
    v
Load Warehouse
```

ADF is particularly useful for orchestrating these steps.

**Remember:**

- Data movement = Move data.
- Data transformation = Process data.
- Orchestration = Coordinate tasks.

---

## 7. ADF Architecture in a Data Engineering Project

Consider a typical Azure data engineering architecture.

```text
                 Source Systems
                       |
          +------------+------------+
          |            |            |
       SQL DB        REST API      CSV Files
          |            |            |
          +------------+------------+
                       |
                       v
              Azure Data Factory
                       |
                 Copy Activities
                       |
                       v
                  ADLS Gen2
                   Raw Layer
                       |
                       v
                Azure Databricks
                 PySpark / SQL
                       |
                       v
                 ADLS Gen2
               Processed Layer
                       |
                       v
                 ADLS Gen2
                 Curated Layer
                       |
                       v
              Synapse / Power BI
```

### Responsibilities

| Service | Responsibility |
|---|---|
| Azure Data Factory | Ingestion orchestration and workflow management |
| ADLS Gen2 | Data lake storage |
| Azure Databricks | Distributed data processing |
| PySpark | Data transformation |
| Azure Synapse | Analytics and SQL workloads |
| Power BI | Reporting and visualization |

The architecture can vary depending on the project.

---

## 8. ADF Studio: Main Interface

Azure Data Factory Studio is the web interface used to create and manage ADF resources.

Important sections include:

### Author

Used to create and edit:

- Pipelines.
- Activities.
- Datasets.
- Linked Services.
- Data Flows.
- Triggers.

### Monitor

Used to monitor:

- Pipeline runs.
- Activity runs.
- Trigger runs.
- Integration Runtime status.
- Execution duration.
- Errors and failure details.

### Manage

Used to manage:

- Linked Services.
- Integration Runtimes.
- Triggers.
- Parameters and global settings where applicable.
- Source control and publishing configuration.

The exact interface may change as Azure updates the product.

---

## 9. Debug, Trigger Now, and Publish

These are common ADF development operations.

### Debug

Runs a pipeline for testing without requiring the normal published trigger workflow.

Useful for:

- Testing activity configurations.
- Checking expressions.
- Validating dependencies.
- Finding errors during development.

### Trigger Now

Manually starts a pipeline run using the published pipeline version.

### Publish

Saves and publishes changes from the authoring environment to the live Data Factory service.

In Git-integrated mode, changes are typically developed in a collaboration branch and published through the configured publishing process.

**Important:** Debug and Trigger Now are not interchangeable. Debug is for development and testing; Trigger Now starts a run using the published version.

---

## 10. ADF Pipeline Run vs Activity Run

These terms are frequently used in monitoring.

### Pipeline Run

Represents one execution of a pipeline.

### Activity Run

Represents the execution of an individual activity inside a pipeline run.

Example:

```text
Pipeline Run: CustomerPipeline
       |
       +---- Copy Activity Run
       |
       +---- Lookup Activity Run
       |
       +---- Databricks Activity Run
```

One pipeline run can contain multiple activity runs.

A pipeline can also invoke child pipelines, creating additional pipeline runs.

---

## 11. Parameters and Dynamic Content

ADF supports dynamic expressions to make pipelines reusable.

### Pipeline Parameter

Example parameter:

```text
tableName = customers
```

Access it using:

```text
@pipeline().parameters.tableName
```

### Current ForEach Item

```text
@item().table_name
```

### Activity Output

Access the output of an activity:

```text
@activity('LookupConfig').output.value
```

### Pipeline Run ID

```text
@pipeline().RunId
```

### Current UTC Time

```text
@utcNow()
```

### Example: Dynamic File Path

```text
@concat(
    'raw/',
    pipeline().parameters.tableName,
    '/'
)
```

If `tableName` is `customers`, the resulting path is:

```text
raw/customers/
```

Dynamic expressions reduce hardcoding and make pipelines reusable.

---

## 12. ADF Security Basics

ADF connects to external systems, so authentication and authorization are important.

Common authentication approaches include:

- Managed Identity.
- Service Principal.
- Service-specific credentials.
- Key Vault-backed secrets.
- Other connector-supported authentication methods.

### Managed Identity

Managed Identity allows an Azure resource to authenticate to supported services without storing credentials directly in pipeline definitions.

Example:

```text
Azure Data Factory
       |
       v
Managed Identity
       |
       v
Microsoft Entra ID
       |
       v
ADLS Gen2
```

The identity must still receive the required permissions on the target resource.

### Azure Key Vault

Key Vault can securely store secrets such as passwords, keys, and connection credentials.

ADF can use a Key Vault-backed Linked Service to retrieve supported secrets.

### Security Best Practices

- Prefer Managed Identity where supported.
- Use least-privilege permissions.
- Store secrets in Key Vault.
- Avoid hardcoding credentials.
- Restrict network access where appropriate.
- Monitor access and pipeline activity.
- Avoid exposing sensitive values in logs.

---

## 13. ADF Monitoring and Troubleshooting

Monitoring helps identify pipeline failures and performance problems.

### What Should You Monitor?

- Pipeline status.
- Activity status.
- Execution duration.
- Data read and written.
- Error messages.
- Retry attempts.
- Trigger execution.
- Integration Runtime availability.

### Common Failure Scenarios

| Problem | Possible Cause |
|---|---|
| Authentication failure | Invalid credentials or identity permissions |
| File not found | Incorrect path or missing file |
| Connection failure | Network or Integration Runtime issue |
| Copy failure | Source/sink configuration or schema issue |
| Databricks activity failure | Notebook or compute error |
| Timeout | Activity took longer than configured timeout |

### Troubleshooting Approach

1. Open the Monitor section.
2. Find the failed pipeline run.
3. Open the activity run.
4. Read the detailed error message.
5. Identify whether the problem is related to authentication, connectivity, configuration, or processing.
6. Fix the root cause.
7. Rerun safely, considering whether the operation is idempotent.

---

## 14. ADF vs SSIS

Azure Data Factory and SQL Server Integration Services are both used for data integration, but they differ in architecture and deployment.

| Feature | Azure Data Factory | SSIS |
|---|---|---|
| Platform | Azure cloud service | Microsoft integration platform |
| Primary use | Cloud and hybrid data integration | Data integration and ETL |
| Orchestration | Pipelines and activities | Packages and control flow |
| Execution | Integration Runtime and connected services | SSIS runtime |
| Cloud integration | Broad Azure and cloud connector support | Depends on package and deployment setup |
| Existing workload migration | Can orchestrate migration and cloud workflows | Existing packages can run using Azure-SSIS IR |

ADF also supports running existing SSIS packages through Azure-SSIS Integration Runtime.

---

## 15. ADF vs Azure Databricks

ADF and Databricks are complementary services.

| Feature | Azure Data Factory | Azure Databricks |
|---|---|---|
| Main role | Data integration and orchestration | Data processing and analytics |
| Data movement | Copy Activity | Can read and write data |
| Transformation | Mapping Data Flows and external services | Spark, PySpark, SQL |
| Workflow | Pipelines and triggers | Jobs and workflows |
| Distributed processing | Not its primary role | Core capability |
| Typical use | Coordinate pipeline execution | Transform large datasets |

### Example

```text
ADF
 |
 +---- Copy source data to ADLS
 |
 +---- Run Databricks Notebook
 |
 +---- Validate output
 |
 +---- Start next pipeline
```

ADF coordinates the workflow. Databricks performs the Spark-based transformation.

---

## 16. Practical Interview Scenario

**Question:** How would you design a daily pipeline to load customer data from Azure SQL Database into ADLS Gen2 and transform it using Databricks?

### Answer

I would create an Azure Data Factory pipeline with the following steps:

1. Configure a Linked Service for Azure SQL Database.
2. Configure a Linked Service for ADLS Gen2.
3. Create the required datasets.
4. Add a Copy Activity to ingest customer data into the raw layer.
5. Configure the Copy Activity to use the required Integration Runtime.
6. Add a Databricks Notebook Activity after successful ingestion.
7. Pass the raw and destination paths as notebook parameters.
8. Perform transformations using PySpark.
9. Write the transformed data to the processed or curated layer.
10. Configure dependencies, retries, logging, and failure handling.
11. Add a schedule trigger for daily execution.
12. Monitor pipeline and activity runs.

### Architecture

```text
Azure SQL Database
        |
        v
Azure Data Factory
        |
        v
Copy Activity
        |
        v
ADLS Gen2 - Raw
        |
        v
Databricks Notebook
        |
        v
PySpark Transformation
        |
        v
ADLS Gen2 - Processed
        |
        v
Curated Data
```

For production, I would also consider incremental loading, idempotency, data quality checks, and recovery from failed runs.

---

## 17. Common Mistakes to Avoid

- Confusing a pipeline with an activity.
- Confusing Linked Services with Datasets.
- Treating Integration Runtime as a data source.
- Assuming ADF performs every transformation itself.
- Assuming Managed Identity automatically has access to storage.
- Confusing Debug with Trigger Now.
- Forgetting to publish changes before running the published pipeline.
- Hardcoding file paths and table names.
- Storing passwords directly in pipeline definitions.
- Ignoring activity dependencies.
- Ignoring retry and timeout configuration.
- Assuming a successful pipeline run means the data is correct.
- Ignoring incremental loading and duplicate handling.
- Treating ADF and Databricks as competing services rather than complementary tools.

---

## 18. Common Interview Questions

### Q1. What is Azure Data Factory?

Azure Data Factory is a cloud-based data integration and orchestration service used to build, schedule, and manage data pipelines.

### Q2. What are the main components of ADF?

Pipeline, Activity, Linked Service, Dataset, Integration Runtime, Trigger, Parameters, and Variables.

### Q3. What is a pipeline?

A pipeline is a logical workflow containing activities that perform a data integration task.

### Q4. What is a Linked Service?

A Linked Service defines the connection information and authentication configuration for an external service or data store.

### Q5. What is a Dataset?

A Dataset represents the structure or location of the data used by an activity.

### Q6. What is Integration Runtime?

Integration Runtime provides the execution and connectivity infrastructure for supported ADF activities.

### Q7. What are the three types of Integration Runtime?

Azure IR, Self-hosted IR, and Azure-SSIS IR.

### Q8. When would you use Self-hosted Integration Runtime?

When ADF needs to connect to data sources in a private network or on-premises environment that are not directly reachable through the available cloud connectivity.

### Q9. What is the difference between a trigger and an activity?

A trigger starts a pipeline. An activity performs a task inside the pipeline.

### Q10. What is the difference between parameters and variables?

Parameters pass values into a pipeline or activity and are generally read-only during a run. Variables store values that can be changed during execution.

### Q11. What is the difference between Debug and Trigger Now?

Debug is used to test a pipeline during development. Trigger Now starts a run using the published pipeline version.

### Q12. What is the difference between a pipeline run and an activity run?

A pipeline run represents one execution of a pipeline. An activity run represents the execution of an individual activity within it.

### Q13. How does ADF work with Databricks?

ADF orchestrates the Databricks notebook execution, passes parameters, and manages dependencies. Databricks performs the data processing.

### Q14. How do you secure credentials in ADF?

Use Managed Identity where supported, Azure Key Vault for secrets, and least-privilege permissions.

### Q15. How do you monitor ADF pipelines?

Use the Monitor section to inspect pipeline runs, activity runs, durations, status, and error details.

### Q16. Can ADF connect to on-premises databases?

Yes. Self-hosted Integration Runtime can provide connectivity to supported on-premises data sources.

### Q17. Is ADF an ETL tool or an orchestration tool?

ADF supports data integration and transformation workflows, but it is especially important as an orchestration service. It can move data and coordinate transformations performed by other services.

### Q18. Does ADF guarantee that a pipeline produces correct data?

No. A successful execution only indicates that the configured operations completed successfully. Data quality checks and business validation are still required.

---

## 19. Quick Revision

| Component | One-Line Meaning |
|---|---|
| Azure Data Factory | Cloud data integration and orchestration service |
| Pipeline | Group of activities |
| Activity | Individual task |
| Linked Service | Connection configuration |
| Dataset | Data structure or location |
| Integration Runtime | Execution and connectivity infrastructure |
| Azure IR | Cloud integration |
| Self-hosted IR | Private network/on-premises connectivity |
| Azure-SSIS IR | Runs SSIS packages |
| Trigger | Starts a pipeline |
| Parameter | Input value |
| Variable | Runtime value that can change |
| Debug | Development testing |
| Trigger Now | Manually starts published pipeline |
| Publish | Publishes authoring changes |
| Monitor | Tracks pipeline and activity execution |

---

## 20. Self-Assessment

Try answering these questions without looking at the notes:

1. What is Azure Data Factory?
2. Why do data engineers use ADF?
3. Explain the main ADF components.
4. What is the difference between a Pipeline and an Activity?
5. What is the difference between a Linked Service and a Dataset?
6. What is Integration Runtime?
7. Explain the three Integration Runtime types.
8. When would you use Self-hosted Integration Runtime?
9. What is the difference between data movement and data transformation?
10. What is the role of ADF in a Databricks pipeline?
11. What is the difference between a Trigger and an Activity?
12. Explain Parameters and Variables.
13. What is the difference between Debug and Trigger Now?
14. How do you monitor a pipeline?
15. How do you secure credentials in ADF?
16. How would you design a daily SQL-to-ADLS ingestion pipeline?
17. How would you troubleshoot a failed Copy Activity?
18. Why are data quality checks necessary even when a pipeline succeeds?

**Interview readiness:** You should be able to explain ADF architecture, its core components, Integration Runtime types, the difference between orchestration and processing, and how ADF works with ADLS Gen2 and Databricks.