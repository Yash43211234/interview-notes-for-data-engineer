# Azure Data Factory: Activities and Dependencies

## 1. What Are Activities in Azure Data Factory?

An **activity** is an individual task performed inside an Azure Data Factory pipeline.

A pipeline contains one or more activities that work together to move, transform, or manage data.

Common activities include:

- Copy Activity — copies data from a source to a destination.
- Lookup Activity — retrieves a dataset or configuration value.
- Get Metadata Activity — retrieves metadata about a file or dataset.
- ForEach Activity — iterates over a collection of items.
- If Condition Activity — executes a branch based on a condition.
- Execute Pipeline Activity — runs another pipeline.
- Stored Procedure Activity — executes a database stored procedure.
- Databricks Notebook Activity — runs a Databricks notebook.
- Delete Activity — deletes files or folders.
- Web Activity — calls a REST endpoint.

### Example

Suppose you need to copy multiple CSV files from ADLS Gen2 into a data warehouse.

A pipeline might look like this:

```text
Start
  |
  v
Get Metadata
  |
  v
ForEach
  |
  v
Copy Activity
  |
  v
Stored Procedure
  |
  v
End
```

Each box represents an activity.

---

## 2. Main Categories of Activities

ADF activities can be understood by their purpose.

| Category | Purpose | Examples |
|---|---|---|
| Data movement | Move data between systems | Copy |
| Data transformation | Process or transform data | Mapping Data Flow, Databricks Notebook |
| Control flow | Control pipeline execution | If Condition, ForEach, Until |
| Data inspection | Retrieve information | Lookup, Get Metadata |
| External execution | Run external operations | Web, Stored Procedure, Azure Function |

An activity's category does not necessarily determine where its processing happens. For example, a Databricks Notebook Activity orchestrates a notebook run, while Databricks performs the actual computation.

---

## 3. Copy Activity

Copy Activity is used to move data from a source to a destination.

### Example

```text
Source: SQL Server
       |
       v
   Copy Activity
       |
       v
Destination: ADLS Gen2
```

It supports many types of data stores through ADF connectors.

### Important Components

- **Source:** Where the data comes from.
- **Sink:** Where the data is written.
- **Linked Service:** Defines connection information.
- **Dataset:** Describes the data structure or location.
- **Integration Runtime:** Provides the compute and connectivity environment for the activity.

### Example Scenario

A company wants to copy customer data from Azure SQL Database to ADLS Gen2.

ADF uses:

1. An Azure SQL linked service.
2. A source dataset.
3. An ADLS Gen2 linked service.
4. A sink dataset.
5. A Copy Activity.

### Interview Point

Copy Activity moves data. It does not automatically perform complex business transformations.

For complex transformations, use Mapping Data Flows, Databricks, SQL, or another suitable processing service.

---

## 4. Lookup Activity

Lookup Activity retrieves data or a configuration value from a supported source.

It is commonly used to read:

- Configuration tables.
- File lists.
- Watermark values.
- Processing parameters.
- Small reference datasets.

### Example

Suppose a configuration table contains:

| table_name | source_schema |
|---|---|
| customers | dbo |
| orders | sales |
| products | dbo |

Lookup Activity can retrieve these records.

The pipeline can then use the result in a ForEach activity to process each table.

### Example Expression

To access the first row returned by Lookup:

```text
@activity('LookupConfig').output.firstRow.table_name
```

To access the array returned by a Lookup configured for multiple rows:

```text
@activity('LookupConfig').output.value
```

### Important Settings

- **First row only:** When enabled, Lookup returns a single row through `firstRow`.
- **First row only disabled:** Lookup returns an array through `value`.

Lookup is intended for relatively small result sets, not for extracting large datasets.

---

## 5. Get Metadata Activity

Get Metadata Activity retrieves metadata about a file, folder, or supported dataset.

It does not read the complete contents of a file.

Common metadata fields include:

- `exists`
- `size`
- `lastModified`
- `itemName`
- `itemType`
- `childItems`
- `contentMD5`

The fields available depend on the connector and dataset.

### Example

Suppose a pipeline needs to check whether an input file exists before copying it.

```text
Get Metadata
     |
     v
Check exists
     |
     v
If Condition
   /     \
 Yes      No
  |        |
 Copy    Skip/Log
```

Expression:

```text
@activity('GetFileMetadata').output.exists
```

### Example: Get Folder Contents

For a supported folder dataset, select the `childItems` field.

The output can contain entries such as:

```json
[
  {
    "name": "customers.csv",
    "type": "File"
  },
  {
    "name": "orders.csv",
    "type": "File"
  }
]
```

This result can be passed to a ForEach activity.

**Important:** Get Metadata returns information about items. It is not a replacement for Copy Activity or a data-reading operation.

---

## 6. ForEach Activity

ForEach Activity executes a set of activities for each item in a collection.

It is useful when the same processing logic must run for multiple tables, files, or configurations.

### Example

Suppose Lookup returns three tables:

```text
customers
orders
products
```

ForEach processes each item using the same activity structure.

```text
Lookup Configuration
        |
        v
      ForEach
        |
        +---- customers
        |
        +---- orders
        |
        +---- products
```

### Items Expression

```text
@activity('LookupConfig').output.value
```

Inside the ForEach activity, access the current item using:

```text
@item().table_name
```

### Sequential Execution

When sequential execution is enabled, items are processed one after another.

```text
Item 1 → Item 2 → Item 3
```

### Parallel Execution

When sequential execution is disabled, ADF can process multiple iterations concurrently, subject to the configured batch count and available resources.

```text
Item 1 ─┐
Item 2 ─┼── Parallel Processing
Item 3 ─┘
```

### Important Settings

- `isSequential`: Controls sequential versus parallel iteration.
- `batchCount`: Controls the maximum number of concurrent iterations when parallel execution is used.

The default parallel batch count is 20, and the supported maximum is 50.

### When to Use

Use ForEach when:

- Processing multiple files.
- Loading multiple database tables.
- Running the same notebook for multiple datasets.
- Applying configuration-driven ingestion.

**Important:** Parallelism should be controlled carefully. Too many concurrent activities can overload the source, sink, or integration resources.

---

## 7. If Condition Activity

If Condition Activity executes one of two branches based on a Boolean expression.

It has two branches:

- True activities.
- False activities.

### Example

A pipeline checks whether a file exists.

```text
Get Metadata
      |
      v
 If Condition
    /     \
 True     False
  |         |
 Copy      Log
```

Expression:

```text
@activity('GetFileMetadata').output.exists
```

If the expression evaluates to `true`, the True branch executes. Otherwise, the False branch executes.

### Another Example

Check whether a Lookup returned any rows:

```text
@greater(length(activity('LookupConfig').output.value), 0)
```

This expression checks whether the returned array contains at least one item.

---

## 8. Execute Pipeline Activity

Execute Pipeline Activity runs another pipeline from the current pipeline.

It supports reusable pipeline design.

### Example

A parent pipeline can call separate child pipelines:

```text
Parent Pipeline
       |
       +---- Execute Pipeline: Customer Load
       |
       +---- Execute Pipeline: Orders Load
       |
       +---- Execute Pipeline: Product Load
```

### Why Use It?

- Reuse common processing logic.
- Separate ingestion and transformation.
- Organize large workflows.
- Make pipelines easier to maintain.

### Wait on Completion

The `waitOnCompletion` setting controls whether the calling pipeline waits for the invoked pipeline to finish.

- `true`: Parent waits for the child pipeline to complete.
- `false`: Parent does not wait for child completion.

If the parent must use the child's completed output or depend on its result, configure the execution and dependencies accordingly.

---

## 9. Stored Procedure Activity

Stored Procedure Activity executes a stored procedure in a supported database.

### Example

A pipeline copies data into a staging table.

After the copy completes, it executes a stored procedure to update the final table.

```text
Copy Activity
      |
      v
Stored Procedure Activity
      |
      v
Target Table Updated
```

Example SQL:

```sql
CREATE PROCEDURE dbo.LoadCustomers
AS
BEGIN
    INSERT INTO dbo.CustomerTarget
        (CustomerID, CustomerName)
    SELECT
        CustomerID,
        CustomerName
    FROM dbo.CustomerStaging;
END;
```

The Stored Procedure Activity can invoke this procedure.

**Important:** The procedure executes in the database. ADF orchestrates the call; the database performs the SQL work.

---

## 10. Databricks Notebook Activity

Databricks Notebook Activity runs a notebook in an Azure Databricks workspace.

It is commonly used for PySpark transformations.

### Example Architecture

```text
ADF Pipeline
     |
     v
Copy Activity
     |
     v
ADLS Gen2 - Raw
     |
     v
Databricks Notebook Activity
     |
     v
PySpark Transformation
     |
     v
ADLS Gen2 - Processed
```

### Common Configuration

- Azure Databricks linked service.
- Notebook path.
- Base parameters.
- Compute or cluster configuration, depending on the integration setup.

### Example Notebook Parameter

ADF can pass a parameter such as:

```text
source_path = abfss://raw@storageaccount.dfs.core.windows.net/orders/
```

The notebook can read the parameter using a Databricks widget:

```python
dbutils.widgets.text("source_path", "")

source_path = dbutils.widgets.get("source_path")

df = spark.read.format("parquet").load(source_path)
```

### Interview Point

ADF is the orchestration service. Databricks performs the Spark-based data processing.

---

## 11. What Are Activity Dependencies?

An activity dependency defines when one activity is allowed to execute based on the outcome of another activity.

Dependencies control the execution flow of a pipeline.

ADF supports four main dependency conditions:

1. Succeeded.
2. Failed.
3. Skipped.
4. Completed.

### 11.1 Succeeded

The next activity executes only if the previous activity succeeds.

Example:

```text
Copy Activity
      |
   Succeeded
      |
      v
Databricks Notebook
```

Use this when the next operation requires successful completion of the previous one.

### 11.2 Failed

The next activity executes only if the previous activity fails.

Example:

```text
Copy Activity
      |
     Failed
      |
      v
Send Failure Notification
```

Use this for failure handling and alerting.

### 11.3 Skipped

The next activity executes if the previous activity is skipped.

An activity can be skipped when its execution path is not selected or when its dependencies result in it being skipped.

### 11.4 Completed

The next activity executes when the previous activity finishes, regardless of whether it succeeded or failed.

Use this when cleanup or logging must happen after an activity finishes.

**Important:** Completed does not mean succeeded. It means the activity reached a terminal execution state.

---

## 12. Dependency Conditions Example

Consider a pipeline with three activities:

```text
Copy Data
    |
    | Succeeded
    v
Transform Data
    |
    | Succeeded
    v
Load Target
```

This means:

1. Copy Data runs first.
2. Transform Data runs only after Copy Data succeeds.
3. Load Target runs only after Transform Data succeeds.

If Copy Data fails, the dependent Transform Data activity will not run through the success path.

### Failure Handling

```text
Copy Data
    |
    +---- Succeeded ----> Transform Data
    |
    +---- Failed -------> Send Alert
```

The failure branch can send an email, call a webhook, or write an error record.

---

## 13. Multiple Dependencies

An activity can depend on more than one preceding activity.

### Example

Suppose a reporting pipeline requires both customer and order data to be loaded before generating a report.

```text
Load Customers ----+
                   |
                   v
               Generate Report
                   ^
                   |
Load Orders -------+
```

The Generate Report activity can have two success dependencies.

It runs only when both required dependencies are satisfied.

### Important

When multiple dependencies are configured, understand the dependency conditions and logical evaluation behavior.

Do not assume that one successful upstream activity is sufficient when the activity has multiple required success dependencies.

---

## 14. Activity Dependencies vs Pipeline Triggers

These are different concepts.

| Activity Dependencies | Pipeline Triggers |
|---|---|
| Control execution order inside a pipeline | Start a pipeline |
| Define conditions between activities | Define when or how a pipeline starts |
| Succeeded, Failed, Skipped, Completed | Schedule, tumbling window, event-based, manual execution |

### Example

A schedule trigger starts a pipeline every day at 2 AM.

Inside the pipeline:

```text
Copy → Transform → Load
```

Activity dependencies control the order of these activities.

---

## 15. Activity Dependencies vs Control Flow Activities

Dependencies and control flow activities both influence pipeline execution, but they serve different purposes.

| Feature | Dependencies | Control Flow Activities |
|---|---|---|
| Main purpose | Define execution conditions between activities | Implement workflow logic |
| Example | Run B after A succeeds | ForEach, If Condition, Until |
| Typical use | Sequential and conditional execution | Iteration, branching, looping |

Example:

- Dependency: Run Databricks Notebook after Copy Activity succeeds.
- If Condition: Check whether the file exists.
- ForEach: Process every file in a list.

They are often used together.

---

## 16. Practical Scenario: Configuration-Driven Data Ingestion

Suppose a company needs to load multiple SQL tables into ADLS Gen2.

The configuration table contains:

| table_name | source_schema |
|---|---|
| customers | dbo |
| orders | sales |
| products | dbo |

### Pipeline Design

```text
Start
  |
  v
Lookup Configuration
  |
  v
ForEach Table
  |
  v
Copy Activity
  |
  +---- Succeeded ----> Log Success
  |
  +---- Failed -------> Log Failure
```

### Step-by-Step Execution

1. Lookup retrieves table configuration.
2. ForEach iterates over the returned rows.
3. Copy Activity reads the current table.
4. Data is written to the corresponding ADLS Gen2 location.
5. Success or failure handling records the outcome.

### Expressions

Lookup output:

```text
@activity('LookupConfig').output.value
```

Current table name:

```text
@item().table_name
```

Current schema:

```text
@item().source_schema
```

Example destination path:

```text
@concat('raw/', item().table_name, '/')
```

This approach avoids creating a separate hardcoded pipeline for every table.

---

## 17. Practical Scenario: File Validation Before Processing

Suppose a daily pipeline receives an orders file.

It should process the file only if it exists.

### Pipeline

```text
Get Metadata
      |
      v
If Condition
   /       \
 True      False
  |          |
 Copy       Log
  |
  v
Databricks Notebook
  |
  v
Load Curated Data
```

### Condition

```text
@activity('GetFileMetadata').output.exists
```

### Execution

**If True:**
- Copy the file.
- Run the transformation notebook.
- Load the curated dataset.

**If False:**
- Record the missing file.
- Send an alert if required.
- End or continue according to business requirements.

This design prevents downstream processing from running when the required input is unavailable.

---

## 18. Practical Scenario: Failure Handling

Suppose a pipeline contains:

```text
Copy Data
    |
    v
Transform Data
    |
    v
Load Data
```

If the Copy Activity fails, the transformation and load should not proceed.

A failure-handling branch can be configured:

```text
Copy Data
    |
    +---- Succeeded ----> Transform Data
    |
    +---- Failed -------> Log Error
                              |
                              v
                         Send Alert
```

For production pipelines, failure handling should capture useful information such as:

- Pipeline run ID.
- Activity name.
- Error message.
- Source and destination.
- Execution timestamp.
- Retry count, where available.

Avoid logging credentials, access tokens, or sensitive data.

---

## 19. Activity Retry and Timeout Settings

ADF activities can be configured with retry and timeout settings.

### Retry

Retry determines how many times ADF attempts an activity again after a failure.

### Retry Interval

Defines the waiting interval between retry attempts.

### Timeout

Defines how long an activity is allowed to run before ADF treats it as timed out.

### Example

A temporary network issue causes a Copy Activity to fail.

A retry may allow the operation to succeed without manual intervention.

### Important

Retries are useful for transient failures, but they do not fix permanent errors such as:

- Invalid credentials.
- Missing permissions.
- Incorrect source paths.
- Invalid SQL.
- Schema incompatibility.

Design activities and destinations to handle retries safely. Repeated execution should not create duplicate or inconsistent data.

---

## 20. How to Design a Reliable ADF Pipeline

A good production pipeline should include:

1. Clear activity responsibilities.
2. Correct dependency conditions.
3. Input validation.
4. Error handling.
5. Retry and timeout configuration.
6. Logging and monitoring.
7. Parameterization.
8. Idempotent processing where possible.
9. Appropriate parallelism.
10. Reusable child pipelines where useful.

### Example Design

```text
Trigger
   |
   v
Read Configuration
   |
   v
Validate Input
   |
   v
ForEach Dataset
   |
   v
Copy Data
   |
   +---- Succeeded ----> Transform
   |                       |
   |                       v
   |                    Validate
   |                       |
   |                       v
   |                     Load
   |
   +---- Failed -------> Log Error
                           |
                           v
                       Send Alert
```

This is a conceptual design. Actual activity connections and error paths depend on the pipeline's requirements.

---

## 21. Common Mistakes to Avoid

- Confusing an activity with a pipeline.
- Assuming Lookup is suitable for large data extraction.
- Assuming Get Metadata reads file contents.
- Using ForEach parallelism without considering source limits.
- Forgetting to pass the correct array to ForEach.
- Using `firstRow` when Lookup is configured to return multiple rows.
- Assuming Copy Activity performs complex transformations.
- Confusing ADF orchestration with Databricks processing.
- Using Completed when the next activity requires successful output.
- Assuming a failed activity automatically sends an alert.
- Ignoring retry and timeout settings.
- Retrying non-idempotent operations without checking duplicate risks.
- Forgetting that multiple dependencies can affect when an activity starts.
- Hardcoding file paths, table names, and credentials unnecessarily.

---

## 22. Common Interview Questions

### Q1. What is an activity in Azure Data Factory?

An activity is an individual task inside an ADF pipeline, such as copying data, retrieving metadata, or executing a Databricks notebook.

### Q2. What is the difference between a pipeline and an activity?

A pipeline is a logical workflow containing activities. An activity performs an individual task within that workflow.

### Q3. What is Copy Activity?

Copy Activity moves data from a supported source to a supported destination.

### Q4. What is the difference between Lookup and Get Metadata?

Lookup retrieves data or configuration values from a supported source. Get Metadata retrieves information about a file, folder, or dataset, such as existence, size, and last modified time.

### Q5. What is ForEach Activity?

ForEach iterates over a collection and executes its inner activities for each item.

### Q6. What is the difference between sequential and parallel ForEach?

Sequential ForEach processes items one after another. Parallel ForEach can process multiple iterations concurrently, subject to its batch count and available resources.

### Q7. What is If Condition Activity?

It evaluates a Boolean expression and executes either the True or False branch.

### Q8. What is Execute Pipeline Activity?

It invokes another pipeline, allowing reusable and modular pipeline design.

### Q9. What are activity dependencies?

They define when an activity can execute based on the outcome of another activity.

### Q10. What are the four dependency conditions?

Succeeded, Failed, Skipped, and Completed.

### Q11. What is the difference between Succeeded and Completed?

Succeeded means the activity finished successfully. Completed means it reached a terminal state, whether successful or failed.

### Q12. How do you handle activity failure in ADF?

I configure failure dependencies to execute logging or notification activities. I also use appropriate retries, timeouts, and monitoring.

### Q13. Can an activity depend on multiple activities?

Yes. An activity can have multiple upstream dependencies, and its execution depends on the configured conditions being satisfied.

### Q14. What is the difference between activity dependencies and triggers?

Dependencies control execution flow inside a pipeline. Triggers start pipelines based on schedules, events, or other configured conditions.

### Q15. How do you pass dynamic table names into Copy Activity?

I can use Lookup to retrieve table configurations, ForEach to iterate through them, and dynamic expressions such as `@item().table_name` to parameterize the source.

### Q16. Does ADF perform the Spark transformation when using Databricks Notebook Activity?

No. ADF orchestrates the notebook execution. Databricks performs the Spark processing.

### Q17. Why should ForEach parallelism be controlled?

Excessive parallelism can overload the source, destination, or compute resources and can increase failures and costs.

### Q18. Why are retries not enough to guarantee reliable data loading?

Retries can repeat an operation. Without idempotent processing or duplicate protection, a retry may produce duplicate or inconsistent results.

---

## 23. Quick Revision

| Concept | Remember |
|---|---|
| Activity | Individual pipeline task |
| Pipeline | Workflow containing activities |
| Copy | Moves data |
| Lookup | Retrieves data or configuration |
| Get Metadata | Retrieves file or folder metadata |
| ForEach | Iterates over a collection |
| If Condition | Executes a Boolean branch |
| Execute Pipeline | Calls another pipeline |
| Stored Procedure | Executes database logic |
| Databricks Notebook | Runs notebook processing |
| Succeeded | Previous activity succeeded |
| Failed | Previous activity failed |
| Skipped | Previous activity was skipped |
| Completed | Previous activity reached a terminal state |
| Retry | Attempts an activity again after failure |
| Timeout | Maximum configured execution duration |
| Dependency | Controls activity execution order |

---

## 24. Self-Assessment

Try answering these questions without looking at the notes:

1. What is an activity in ADF?
2. Explain the difference between a pipeline and an activity.
3. How does Copy Activity work?
4. When would you use Lookup Activity?
5. What is the difference between Lookup and Get Metadata?
6. How does ForEach work?
7. What is the difference between sequential and parallel execution?
8. How do you use Lookup output inside ForEach?
9. What is If Condition Activity?
10. What is Execute Pipeline Activity?
11. Explain the four activity dependency conditions.
12. What is the difference between Succeeded and Completed?
13. How do you handle activity failures?
14. How do retries and timeouts work?
15. How would you build a configuration-driven ingestion pipeline?
16. How would you validate that an input file exists before processing?
17. How do you control parallelism in a production pipeline?
18. Why is idempotency important when retrying pipeline activities?

**Interview readiness:** You should be able to explain common ADF activities, their purpose, how to pass dynamic values between them, how dependencies control execution, and how to design a reliable configuration-driven pipeline.