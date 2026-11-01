# Azure Data Factory: Linked Services, Datasets, and Pipelines

## 1. Overview

Azure Data Factory (ADF) uses three important building blocks to create data integration workflows:

1. **Linked Service** – Defines how to connect to a data source, destination, or compute service.
2. **Dataset** – Identifies the data structure or location that an activity reads from or writes to.
3. **Pipeline** – Groups and coordinates activities to implement a data workflow.

These components work together to build ETL and ELT pipelines.

### Simple Example

Suppose you need to copy customer data from Azure SQL Database to ADLS Gen2.

```text
             Azure Data Factory
                     |
                     v
                  Pipeline
                     |
                     v
                Copy Activity
                  /       \
                 v         v
          Source Dataset  Sink Dataset
                 |         |
                 v         v
          SQL Linked      ADLS Linked
           Service        Service
                 |         |
                 v         v
          Azure SQL       ADLS Gen2
```

The pipeline coordinates the work, datasets identify the data, and linked services define the connections.

---

## 2. Linked Service

### What Is a Linked Service?

A **Linked Service** defines the connection information required by Azure Data Factory to connect to an external resource.

It tells ADF:

- Which service or data store to connect to.
- Where the service is located.
- Which authentication method to use.
- Which Integration Runtime or connectivity configuration to use, where applicable.

Think of a Linked Service as a connection configuration.

### Examples

- Azure SQL Database Linked Service.
- Azure Data Lake Storage Gen2 Linked Service.
- Azure Blob Storage Linked Service.
- SQL Server Linked Service.
- Azure Databricks Linked Service.
- REST API Linked Service.

### Example: Azure SQL Database Linked Service

A SQL Database connection may contain:

```text
Server: myserver.database.windows.net
Database: SalesDB
Authentication: Managed Identity
```

These details allow ADF to connect to the database, provided the identity has the required permissions and network access is configured.

### Example: ADLS Gen2 Linked Service

```text
Storage Account: mystorageaccount
Authentication: Managed Identity
```

The linked service defines the storage connection. The identity must separately have appropriate permissions to access the data.

### Important Characteristics

- Stores connection configuration.
- Can be reused by multiple datasets and activities.
- Supports different authentication methods depending on the connector.
- Can use Azure Key Vault for supported secret references.
- Is not the actual data or file.

---

## 3. Dataset

### What Is a Dataset?

A **Dataset** represents the data structure or location that an ADF activity uses.

It identifies what data should be read or written.

A dataset can describe:

- A database table.
- A database query target.
- A file in ADLS Gen2.
- A CSV file in Blob Storage.
- A JSON file.
- A folder or file path.

Think of a dataset as a reference to the data you want to work with.

### Example: SQL Dataset

Suppose a SQL database contains a table:

```sql
CREATE TABLE dbo.Customers (
    customer_id INT,
    customer_name VARCHAR(100),
    city VARCHAR(50)
);
```

A dataset can identify:

```text
Linked Service: AzureSqlDB_LinkedService
Schema: dbo
Table: Customers
```

The dataset identifies the table, while the linked service defines how ADF connects to the database.

### Example: CSV Dataset

Suppose the file is stored at:

```text
container: raw
folder: customers
file: customers.csv
```

The dataset can contain:

```text
Linked Service: ADLS_LinkedService
Container: raw
Directory: customers
File: customers.csv
Format: DelimitedText
```

### Dataset Is Not the Data Itself

A dataset does not contain a copy of the actual data.

It describes where the data is located and, depending on the connector and format, how its structure should be interpreted.

---

## 4. Pipeline

### What Is a Pipeline?

A **Pipeline** is a logical group of activities that performs a data integration workflow.

It defines the sequence and coordination of operations.

A pipeline can contain:

- Copy Activity.
- Lookup Activity.
- Get Metadata Activity.
- ForEach Activity.
- If Condition Activity.
- Execute Pipeline Activity.
- Stored Procedure Activity.
- Databricks Notebook Activity.
- Other supported activities.

### Example: Simple ETL Pipeline

Suppose you need to copy customer data from SQL Database to ADLS Gen2.

```text
       ADF Pipeline
            |
            v
       Copy Activity
            |
            v
     Read Customer Data
            |
            v
      Write to ADLS
```

### Example: Multi-step Pipeline

```text
          Start
            |
            v
       Lookup Activity
            |
            v
       Copy Activity
            |
            v
    Databricks Notebook
            |
            v
      Stored Procedure
            |
            v
           End
```

Each activity performs a specific task, and dependencies determine when the next activity executes.

### Important Characteristics

- A pipeline groups activities.
- Activities perform individual operations.
- Dependencies control execution order.
- Parameters make pipelines reusable.
- Variables can store and update values during execution.
- Pipelines can be started manually or through triggers.

---

## 5. Difference Between Linked Service, Dataset, and Pipeline

This is one of the most important ADF interview topics.

| Component | Main Purpose | Example |
|---|---|---|
| Linked Service | Defines connection information | SQL Database connection |
| Dataset | Identifies data structure or location | `dbo.Customers` |
| Pipeline | Coordinates the workflow | SQL-to-ADLS pipeline |

### Easy Analogy

Imagine you want to travel to a specific place.

- **Linked Service:** The connection details that let you reach the location.
- **Dataset:** The specific place or item you want to access.
- **Pipeline:** The overall plan that coordinates the journey.

### Remember

- Linked Service → How to connect?
- Dataset → Which data?
- Pipeline → What workflow should run?

---

## 6. How They Work Together

Let's understand a practical example.

### Requirement

Copy customer records from Azure SQL Database into ADLS Gen2 as a CSV file.

### Step 1: Create the Source Linked Service

Create a Linked Service for Azure SQL Database.

Example:

```text
Name: AzureSqlDB_LinkedService
Server: myserver.database.windows.net
Database: SalesDB
Authentication: Managed Identity
```

### Step 2: Create the Sink Linked Service

Create a Linked Service for ADLS Gen2.

```text
Name: ADLS_LinkedService
Storage Account: mystorageaccount
Authentication: Managed Identity
```

### Step 3: Create the Source Dataset

```text
Name: Customers_SQL_Dataset
Linked Service: AzureSqlDB_LinkedService
Schema: dbo
Table: Customers
```

### Step 4: Create the Sink Dataset

```text
Name: Customers_CSV_Dataset
Linked Service: ADLS_LinkedService
Container: raw
Folder: customers
File: customers.csv
Format: DelimitedText
```

### Step 5: Create the Pipeline

Create a pipeline named:

```text
Copy_Customers_To_ADLS
```

Add a Copy Activity and configure:

```text
Source: Customers_SQL_Dataset
Sink: Customers_CSV_Dataset
```

### Step 6: Execute the Pipeline

When the pipeline runs:

1. ADF uses the source linked service to connect to SQL Database.
2. The source dataset identifies the `dbo.Customers` table.
3. Copy Activity reads the records.
4. ADF uses the sink linked service to connect to ADLS Gen2.
5. The sink dataset identifies the output location and file format.
6. Copy Activity writes the data to the destination.

### Final Architecture

```text
                 ADF Pipeline
                       |
                       v
                 Copy Activity
                   /       \
                  v         v
          Source Dataset  Sink Dataset
                  |         |
                  v         v
          SQL Linked       ADLS Linked
           Service          Service
                  |         |
                  v         v
          Azure SQL       ADLS Gen2
```

---

## 7. Parameters and Dynamic Datasets

A common requirement is to process multiple tables or files using one pipeline.

Instead of creating separate datasets for every table or file, you can use parameters.

### What Are Dataset Parameters?

Dataset parameters allow values such as table names, file names, folder paths, or schema names to be supplied at runtime.

### Example

Create a parameterized SQL dataset.

Dataset name:

```text
Generic_SQL_Dataset
```

Parameters:

```text
schemaName
tableName
```

Use the parameters in the dataset's table configuration:

```text
Schema: @dataset().schemaName
Table: @dataset().tableName
```

The exact configuration depends on the connector and dataset type.

### Passing Parameters from a Pipeline

Suppose the pipeline passes:

```text
schemaName = dbo
tableName = Customers
```

The dataset resolves to:

```text
dbo.Customers
```

For another run, the pipeline can pass:

```text
schemaName = dbo
tableName = Orders
```

The same dataset can now refer to:

```text
dbo.Orders
```

### Benefits

- Reusability.
- Less duplicated configuration.
- Easier maintenance.
- Supports metadata-driven pipelines.

**Important:** Dataset parameters are not the same as pipeline parameters. Pipeline parameters receive values at pipeline execution, and those values can be passed into dataset parameters.

---

## 8. Pipeline Parameters and Variables

Parameters and variables are both used to work with dynamic values, but they have different purposes.

### Pipeline Parameters

Parameters are inputs supplied to a pipeline.

They are read-only during that pipeline run.

Example:

```text
Parameter: sourceTable
Value: Customers
```

Expression:

```text
@pipeline().parameters.sourceTable
```

### Pipeline Variables

Variables are values stored inside a pipeline and can be updated during execution.

Example:

```text
Variable: rowCount
Type: Integer
Initial Value: 0
```

Set Variable Activity:

```text
@add(variables('rowCount'), 100)
```

### Difference

| Feature | Parameter | Variable |
|---|---|---|
| Purpose | Input to pipeline | Temporary pipeline state |
| Can be changed during run? | No | Yes |
| Scope | Pipeline or activity configuration | Pipeline |
| Common use | Table name, folder path, date | Counter, intermediate value |

**Interview point:** Parameters make pipelines configurable and reusable. Variables hold values that may change during execution.

---

## 9. Pipeline Triggers

A pipeline does not have to be started manually every time.

ADF triggers can start pipeline runs based on configured conditions.

Common trigger types:

1. Schedule Trigger.
2. Tumbling Window Trigger.
3. Storage Event Trigger.

### Schedule Trigger

Runs a pipeline according to a time-based schedule.

Example:

```text
Run every day at 02:00 AM
```

### Tumbling Window Trigger

Runs pipelines for fixed, continuous time windows.

Useful for time-based data processing and dependency management.

### Storage Event Trigger

Starts a pipeline when a supported storage event occurs, such as a file being created.

### Important Difference

A trigger starts a pipeline. Activity dependencies determine the execution order inside the pipeline.

---

## 10. Dataset vs Linked Service in Different Scenarios

### Scenario 1: Multiple Tables in One Database

Suppose you have:

```text
Customers
Orders
Products
Payments
```

You can use:

- One SQL Linked Service for the database connection.
- Multiple datasets for individual tables.

### Scenario 2: Multiple Files in One Storage Account

Suppose you have:

```text
customers.csv
orders.csv
products.csv
```

You can use:

- One ADLS Linked Service.
- Multiple datasets, or a parameterized dataset for reusable file paths.

### Scenario 3: Multiple Databases

If databases have different connection configurations, you may need separate linked services.

A linked service can sometimes be parameterized when the connector supports it and the connection requirements are compatible.

---

## 11. Common Mistakes

1. **Confusing Linked Service with Dataset:** A linked service defines connection details; a dataset identifies the data.
2. **Thinking Dataset Stores Data:** A dataset is metadata/configuration, not a physical copy of the data.
3. **Thinking Pipeline Is an Activity:** A pipeline contains and coordinates activities.
4. **Confusing Parameters and Variables:** Parameters are read-only during a run; variables can be updated.
5. **Creating Duplicate Linked Services:** Reuse connection configurations when appropriate.
6. **Hardcoding Every Table Name:** Consider parameterized datasets for reusable pipelines.
7. **Ignoring Authentication:** A linked service can be configured correctly but still fail if its identity lacks permissions.
8. **Ignoring Network Connectivity:** Correct credentials do not solve private network or firewall problems.
9. **Confusing Triggers with Dependencies:** Triggers start pipelines; dependencies control activity execution order.
10. **Assuming Dataset Defines the Full Workflow:** The pipeline and its activities define the workflow.

---

## 12. Best Practices

- Give linked services, datasets, and pipelines meaningful names.
- Reuse linked services when connection settings are shared.
- Use parameterized datasets for repeated table or file processing.
- Keep pipeline parameters clear and meaningful.
- Use variables only when values need to change during execution.
- Store secrets in Azure Key Vault where supported.
- Prefer managed identities where appropriate.
- Follow least-privilege access.
- Separate development, testing, and production configurations.
- Use pipeline dependencies to control execution order.
- Monitor pipeline and activity runs.
- Use reusable child pipelines for common workflow steps.
- Validate source and sink configurations before publishing.

### Example Naming Convention

| Component | Example Name |
|---|---|
| SQL Linked Service | `LS_AzureSql_Sales` |
| ADLS Linked Service | `LS_ADLS_Raw` |
| SQL Dataset | `DS_SQL_Customers` |
| CSV Dataset | `DS_ADLS_Customers_CSV` |
| Pipeline | `PL_Copy_Customers_To_ADLS` |

These names are examples; consistency matters more than any particular prefix.

---

## 13. Common Interview Questions

### Q1. What is a Linked Service in ADF?

A Linked Service defines the connection information required by ADF to connect to a data store or compute service.

### Q2. What is a Dataset in ADF?

A Dataset represents the structure or location of data that an activity reads from or writes to.

### Q3. What is a Pipeline?

A Pipeline is a logical group of activities that coordinates a data integration workflow.

### Q4. What is the difference between Linked Service and Dataset?

A Linked Service defines how to connect to a resource, while a Dataset identifies the data to access within that resource.

### Q5. Can multiple datasets use the same Linked Service?

Yes. Multiple datasets can reuse the same Linked Service when they use the same connection configuration.

### Q6. Can one pipeline contain multiple activities?

Yes. A pipeline can contain multiple activities connected through dependencies.

### Q7. What is the difference between a pipeline and an activity?

A pipeline coordinates the workflow, while an activity performs an individual operation within that workflow.

### Q8. What are dataset parameters?

Dataset parameters allow values such as table names, file names, and folder paths to be supplied dynamically at runtime.

### Q9. What is the difference between pipeline parameters and variables?

Pipeline parameters are read-only inputs for a run. Variables are internal values that can be updated during execution.

### Q10. Can we use one dataset for multiple tables?

Yes, if the dataset is parameterized and the connector supports the required dynamic configuration.

### Q11. What is the difference between a trigger and an activity dependency?

A trigger starts a pipeline. An activity dependency determines when an activity can execute based on the outcome of another activity.

### Q12. Does a Dataset contain actual data?

No. It contains metadata and configuration that identify the data structure or location.

### Q13. How do Linked Services work with Integration Runtime?

A Linked Service defines the connection configuration. Integration Runtime provides the execution and connectivity infrastructure used by supported activities.

### Q14. How would you design a reusable pipeline for multiple tables?

I would create a parameterized pipeline and dataset. I would pass table names dynamically, use a ForEach activity when processing a list of tables, and reuse the same connection configuration where appropriate.

### Q15. Can a pipeline call another pipeline?

Yes. The Execute Pipeline Activity allows one pipeline to execute another pipeline.

---

## 14. Quick Revision

| Component | Remember |
|---|---|
| Linked Service | How to connect |
| Dataset | Which data to access |
| Pipeline | What workflow to execute |
| Activity | Individual operation |
| Integration Runtime | Connectivity and execution infrastructure |
| Pipeline Parameter | Read-only input |
| Pipeline Variable | Mutable pipeline value |
| Dataset Parameter | Dynamic dataset configuration |
| Trigger | Starts pipeline execution |
| Dependency | Controls activity execution order |

### One-Line Summary

**Linked Service defines the connection, Dataset identifies the data, and Pipeline coordinates the activities that process the data.**

---

## 15. Self-Assessment

Try answering these questions without looking at the notes:

1. What is a Linked Service?
2. What is a Dataset?
3. What is a Pipeline?
4. Explain how these three components work together.
5. Can multiple datasets share one Linked Service?
6. Does a Dataset contain actual data?
7. What is the difference between a Pipeline and an Activity?
8. What are Dataset Parameters?
9. How can one dataset be reused for multiple tables?
10. What is the difference between pipeline parameters and variables?
11. What is the purpose of a trigger?
12. How are triggers different from activity dependencies?
13. How does Integration Runtime relate to Linked Services?
14. How would you build a reusable pipeline for multiple source tables?
15. How would you troubleshoot a pipeline that fails to connect to its source?

**Interview readiness:** You should be able to explain the three core components clearly, describe how they work together in a Copy Activity, and design a reusable parameterized pipeline for multiple tables or files.