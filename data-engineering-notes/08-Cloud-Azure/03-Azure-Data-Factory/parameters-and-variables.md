# Azure Data Factory: Parameters and Variables

## 1. Introduction

Parameters and variables are used in Azure Data Factory (ADF) to make pipelines dynamic, reusable, and easier to maintain.

They help avoid hardcoding values such as:

- Table names.
- File names.
- Folder paths.
- Database names.
- Processing dates.
- Environment-specific settings.
- Intermediate values during pipeline execution.

ADF mainly provides these types of parameters and variables:

1. Pipeline parameters.
2. Pipeline variables.
3. Dataset parameters.
4. Linked service parameters.

The most important distinction:

- **Parameter:** An input value that cannot be changed during the pipeline run.
- **Variable:** A value that can be updated during the pipeline run.

---

## 2. Pipeline Parameters

### What Are Pipeline Parameters?

Pipeline parameters are input values passed to a pipeline when it executes.

They make pipelines reusable by allowing different values for different runs.

### Example

Suppose you have a pipeline that copies data from a SQL table to ADLS Gen2.

Instead of hardcoding the table name, create a parameter:

```text
Parameter Name: tableName
Type: String
Default Value: Customers
```

Use the parameter in an expression:

```text
@pipeline().parameters.tableName
```

Now the same pipeline can process different tables.

Run 1:

```text
tableName = Customers
```

Run 2:

```text
tableName = Orders
```

Run 3:

```text
tableName = Products
```

### Creating Pipeline Parameters

1. Open Azure Data Factory Studio.
2. Open the required pipeline.
3. Select the Parameters tab.
4. Add a parameter.
5. Provide its name and data type.
6. Optionally provide a default value.
7. Reference it using an ADF expression.

### Parameter Data Types

Common parameter types include:

- String.
- Int.
- Float.
- Bool.
- Array.
- Object.
- SecureString.

The supported types depend on the ADF property or activity in which the parameter is used.

### Important Characteristics

- Parameters are supplied as pipeline inputs.
- They are read-only during execution.
- Different runs can use different values.
- They support reusable pipeline designs.

---

## 3. Pipeline Variables

### What Are Pipeline Variables?

Variables are values stored inside a pipeline that can be updated during execution.

They are useful when a pipeline needs to maintain temporary state.

### Example

Suppose a pipeline processes multiple tables and maintains a counter.

Create a variable:

```text
Variable Name: tableCount
Type: Integer
Default Value: 0
```

Use a Set Variable activity to update it:

```text
@add(variables('tableCount'), 1)
```

After processing three tables, the value can become:

```text
tableCount = 3
```

### Creating Pipeline Variables

1. Open the pipeline.
2. Select the Variables tab.
3. Add a variable.
4. Enter its name.
5. Select its data type.
6. Set an initial value.
7. Use Set Variable or Append Variable activities to update it.

### Variable Data Types

ADF pipeline variables support:

- String.
- Bool.
- Array.

They do not provide a general-purpose Integer variable type.

For numeric calculations, use expressions such as `add()` and store the result in a String variable if needed, or use another appropriate design.

### Example: String Variable

```text
Variable Name: currentTable
Type: String
Default Value: Customers
```

Update it using Set Variable:

```text
@pipeline().parameters.tableName
```

### Example: Array Variable

```text
Variable Name: processedTables
Type: Array
Default Value: []
```

Use Append Variable to add a value:

```text
@item().tableName
```

This expression assumes the activity is inside a ForEach loop iterating over objects that contain a `tableName` property.

---

## 4. Parameters vs Variables

This is one of the most common ADF interview questions.

| Feature | Parameters | Variables |
|---|---|---|
| Purpose | Receive pipeline inputs | Store temporary pipeline values |
| Can change during a run? | No | Yes |
| Defined in | Pipeline Parameters tab | Pipeline Variables tab |
| Common use | Table name, folder path, date | Intermediate values, counters, arrays |
| Activities | Read by expressions | Read and updated by supported activities |
| Scope | Pipeline input | Pipeline |

### Simple Example

Parameter:

```text
tableName = Customers
```

The pipeline receives this value when it starts.

Variable:

```text
currentStatus = Running
```

The pipeline can update it during execution.

### Remember

**Parameters are inputs; variables are mutable values.**

---

## 5. ADF Dynamic Content Expressions

ADF expressions allow you to use parameters, variables, activity outputs, and system variables dynamically.

Expressions are commonly used in:

- Dataset properties.
- Linked Service properties.
- Activity settings.
- If Condition expressions.
- Copy Activity source and sink settings.
- Pipeline control flow.

### Reference a Pipeline Parameter

```text
@pipeline().parameters.tableName
```

### Reference a Pipeline Variable

```text
@variables('currentTable')
```

### Reference a Pipeline Run ID

```text
@pipeline().RunId
```

### Reference the Pipeline Name

```text
@pipeline().Pipeline
```

### Reference the Current Date

```text
@formatDateTime(utcNow(), 'yyyy-MM-dd')
```

### Concatenate Strings

```text
@concat('raw/', pipeline().parameters.tableName, '.csv')
```

If `tableName` is `Customers`, the result is:

```text
raw/Customers.csv
```

### Convert a Value to an Integer

```text
@int('100')
```

### Add Two Values

```text
@add(10, 20)
```

Result:

```text
30
```

**Important:** ADF expressions are not Python or SQL expressions. Use ADF expression functions and the correct syntax for the property being configured.

---

## 6. Dataset Parameters

### What Are Dataset Parameters?

Dataset parameters allow a dataset to receive dynamic values at runtime.

They are useful when the same dataset must refer to different tables, files, or folders.

### Example

Suppose a database contains:

```text
Customers
Orders
Products
Payments
```

Instead of creating four separate datasets, create one parameterized dataset.

Dataset name:

```text
DS_SQL_Generic
```

Parameters:

```text
schemaName
tableName
```

Configure the dataset's schema and table properties using:

```text
@dataset().schemaName
```

```text
@dataset().tableName
```

### Pass Values from a Pipeline

Suppose the pipeline has a parameter:

```text
tableName = Customers
```

In the activity's dataset parameter section, pass:

```text
schemaName = dbo
tableName = @pipeline().parameters.tableName
```

The dataset now refers to:

```text
dbo.Customers
```

For another run, pass:

```text
tableName = Orders
```

The same dataset can refer to:

```text
dbo.Orders
```

### Benefits

- Reuse one dataset.
- Reduce duplicated configuration.
- Support metadata-driven pipelines.
- Process multiple tables dynamically.

**Important:** The connector must support the dynamic configuration being used. Dataset parameters do not automatically make every connection property dynamic.

---

## 7. Linked Service Parameters

### What Are Linked Service Parameters?

Linked Service parameters allow connection properties to be supplied dynamically.

They are useful when the same Linked Service configuration must connect to different compatible resources.

### Example

Suppose you have different storage accounts for different environments:

```text
Development: devstorage
Testing: teststorage
Production: prodstorage
```

Instead of maintaining completely separate connection definitions, you can parameterize a supported Linked Service property.

Example parameter:

```text
storageAccountName
```

Use it in the supported connection property:

```text
@{linkedService().storageAccountName}
```

The exact expression and property configuration depend on the connector.

### Passing the Parameter

A dataset or activity using the Linked Service can provide the required parameter value through its linked-service parameter configuration.

### Benefits

- Environment-based configuration.
- Reusable connection definitions.
- Less duplicated configuration.
- Easier maintenance.

**Important:** Parameterizing a Linked Service does not remove the need for authentication, authorization, or network connectivity.

---

## 8. System Variables

ADF provides system variables that expose information about the current pipeline execution and trigger.

### Common Pipeline System Variables

| System Variable | Purpose |
|---|---|
| `@pipeline().Pipeline` | Pipeline name |
| `@pipeline().RunId` | Pipeline run ID |
| `@pipeline().TriggerId` | Trigger ID, when available |
| `@pipeline().TriggerName` | Trigger name, when available |
| `@pipeline().TriggerTime` | Trigger time, when available |

### Example: Store Pipeline Run ID

```text
@pipeline().RunId
```

This can be passed to a logging activity to identify a particular execution.

### Trigger System Variables

For a schedule trigger, trigger-related properties can provide information about the scheduled execution.

For a tumbling window trigger, window-related properties can identify the processing interval.

Examples:

```text
@trigger().scheduledTime
```

```text
@trigger().startTime
```

```text
@trigger().endTime
```

The available properties depend on the trigger type. Do not assume that every trigger exposes every property.

---

## 9. Practical Example: Reusable Pipeline for Multiple Tables

### Requirement

A company has multiple tables in Azure SQL Database:

```text
Customers
Orders
Products
Payments
```

The requirement is to copy all tables into ADLS Gen2 using one reusable pipeline.

### Step 1: Create Pipeline Parameters

```text
schemaName: String
tableName: String
```

### Step 2: Create a Parameterized SQL Dataset

Dataset parameters:

```text
schemaName
tableName
```

Dataset properties:

```text
Schema: @dataset().schemaName
Table: @dataset().tableName
```

### Step 3: Create a Parameterized ADLS Dataset

Dataset parameters:

```text
folderName
fileName
```

Example path:

```text
@dataset().folderName/@dataset().fileName
```

### Step 4: Configure the Copy Activity

Source dataset parameters:

```text
schemaName = @pipeline().parameters.schemaName
tableName = @pipeline().parameters.tableName
```

Sink dataset parameters:

```text
folderName = raw
fileName = @concat(pipeline().parameters.tableName, '.csv')
```

### Step 5: Execute with Different Values

First run:

```text
schemaName = dbo
tableName = Customers
```

Output:

```text
raw/Customers.csv
```

Second run:

```text
schemaName = dbo
tableName = Orders
```

Output:

```text
raw/Orders.csv
```

The same pipeline and datasets can be reused.

---

## 10. Practical Example: ForEach with Metadata

For a metadata-driven pipeline, table names can be stored in an array or retrieved from a configuration table.

Example array:

```json
[
  {"schemaName": "dbo", "tableName": "Customers"},
  {"schemaName": "dbo", "tableName": "Orders"},
  {"schemaName": "dbo", "tableName": "Products"}
]
```

A ForEach activity iterates over the array.

Inside the loop, pass the current item's values to the Copy Activity.

Schema:

```text
@item().schemaName
```

Table:

```text
@item().tableName
```

### Execution Flow

```text
       Pipeline
           |
           v
    Get Table Metadata
           |
           v
        ForEach
           |
           v
      Copy Activity
           |
           v
     ADLS Gen2 Output
```

### Important

- `@item()` refers to the current item in a ForEach iteration.
- Sequential execution can be configured when required.
- Parallel execution can improve throughput but may overload the source or destination.
- Avoid updating shared pipeline variables from parallel ForEach iterations because concurrent updates can produce unexpected results.

---

## 11. Variable Scope and Concurrency

ADF pipeline variables are pipeline-scoped.

They are not independent variables for each ForEach iteration.

### Example Problem

Suppose a ForEach activity processes three tables in parallel and every iteration updates the same variable:

```text
processedTable = current table name
```

Multiple iterations may attempt to update the same variable concurrently.

The final value may not represent a predictable iteration order.

### Better Approach

- Use `@item()` to access the current iteration's data.
- Pass values directly to activities.
- Use Append Variable only when its behavior is appropriate for the execution design.
- Use sequential ForEach execution if shared mutable state is required.
- Prefer collecting results through a dedicated logging or storage mechanism for complex parallel workflows.

**Interview point:** Shared pipeline variables should be used carefully inside parallel ForEach activities.

---

## 12. Parameters, Variables, and Expressions: Common Mistakes

1. **Trying to update a parameter:** Pipeline parameters are read-only during execution.
2. **Using the wrong reference:** Use `pipeline().parameters.name` for pipeline parameters and `variables('name')` for variables.
3. **Confusing dataset and pipeline parameters:** Dataset parameters belong to a dataset; pipeline parameters belong to a pipeline.
4. **Hardcoding table names:** Use parameterized datasets when repeated configurations are required.
5. **Using an unsupported variable type:** ADF pipeline variables support String, Bool, and Array.
6. **Updating shared variables in parallel:** Concurrent ForEach iterations can cause unexpected results.
7. **Incorrect expression syntax:** Use ADF expression syntax, not Python or SQL syntax.
8. **Ignoring null values:** Expressions can fail or produce unexpected results when required inputs are NULL.
9. **Assuming every connector supports dynamic properties:** Check the connector's supported configuration.
10. **Putting secrets in ordinary parameters:** Use secure authentication and secret-management mechanisms instead.

---

## 13. Best Practices

- Use pipeline parameters for runtime inputs.
- Use variables only for values that must change during execution.
- Use dataset parameters for dynamic table and file references.
- Use Linked Service parameters for reusable connection configurations where supported.
- Use meaningful parameter and variable names.
- Set sensible defaults where appropriate.
- Validate parameter values before processing.
- Use metadata-driven pipelines for repeated table ingestion.
- Avoid hardcoding environment-specific values.
- Use Key Vault or managed identity for secure authentication.
- Avoid shared variable updates in parallel loops.
- Log pipeline parameters carefully without exposing sensitive values.
- Keep pipeline and dataset configuration reusable and easy to maintain.

---

## 14. Common Interview Questions

### Q1. What are parameters in Azure Data Factory?

Parameters are input values passed to a pipeline, dataset, or supported Linked Service configuration. They help make ADF components reusable and dynamic.

### Q2. What are variables in ADF?

Variables are pipeline-scoped values that can be updated during pipeline execution.

### Q3. What is the difference between parameters and variables?

Parameters are read-only inputs. Variables are mutable values that can change during execution.

### Q4. Can we change a pipeline parameter during execution?

No. Pipeline parameters are read-only during a pipeline run.

### Q5. What data types are supported by pipeline variables?

String, Bool, and Array.

### Q6. What is a dataset parameter?

A dataset parameter allows values such as table names, folder paths, and file names to be passed dynamically to a dataset.

### Q7. What is a Linked Service parameter?

It allows supported connection properties to be supplied dynamically, helping reuse connection configurations across compatible resources.

### Q8. How do you reference a pipeline parameter?

```text
@pipeline().parameters.parameterName
```

### Q9. How do you reference a variable?

```text
@variables('variableName')
```

### Q10. What is the use of `@item()`?

It returns the current item being processed inside a ForEach activity.

### Q11. How do you create a reusable pipeline for multiple tables?

I create pipeline parameters for schema and table names, use a parameterized dataset, and pass different values during each execution. For multiple tables, I can use a ForEach activity with a metadata list.

### Q12. Can a variable be updated inside a ForEach activity?

Yes, but shared variables should be used carefully when ForEach iterations run in parallel because concurrent updates can cause unexpected results.

### Q13. What is the difference between pipeline parameters and dataset parameters?

Pipeline parameters provide inputs to the pipeline. Dataset parameters provide dynamic configuration values to the dataset. Pipeline values can be passed into dataset parameters.

### Q14. Why use Linked Service parameters?

They help reuse connection configurations when supported properties, such as resource names or endpoints, need to vary.

### Q15. How do parameters support metadata-driven ingestion?

A pipeline can read table metadata from a configuration source and pass each table's name and schema dynamically to a reusable dataset and Copy Activity.

---

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Pipeline Parameter | Read-only pipeline input |
| Pipeline Variable | Mutable pipeline value |
| Dataset Parameter | Dynamic table, folder, or file configuration |
| Linked Service Parameter | Dynamic connection configuration |
| `pipeline().parameters` | Access pipeline parameters |
| `variables()` | Access pipeline variables |
| `dataset()` | Access dataset parameters |
| `linkedService()` | Access Linked Service parameters |
| `@item()` | Current ForEach item |
| `@pipeline().RunId` | Current pipeline run ID |
| ForEach | Iterates over a collection |
| Metadata-driven pipeline | Uses configuration data to control processing |

### One-Line Summary

**Parameters make ADF components configurable and reusable, while variables store values that can change during pipeline execution.**

---

## 16. Self-Assessment

Try answering these questions without looking at the notes:

1. What are pipeline parameters?
2. What are pipeline variables?
3. What is the main difference between parameters and variables?
4. Which data types are supported by pipeline variables?
5. How do you reference a pipeline parameter?
6. How do you reference a variable?
7. What are dataset parameters?
8. What are Linked Service parameters?
9. How can one dataset process multiple SQL tables?
10. How can ForEach help build a metadata-driven pipeline?
11. What does `@item()` return?
12. Why should shared variables be handled carefully in parallel ForEach activities?
13. What is the difference between pipeline and dataset parameters?
14. How can you use parameters to support development and production environments?
15. How would you design a reusable pipeline to ingest 100 tables?

**Interview readiness:** You should be able to explain pipeline parameters, variables, dataset parameters, and Linked Service parameters, write their common expressions, and design a reusable metadata-driven pipeline using ForEach.