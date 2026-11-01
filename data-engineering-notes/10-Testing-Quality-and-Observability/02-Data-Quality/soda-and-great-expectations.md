

# Soda vs Great Expectations

## 1. Introduction

Soda and Great Expectations are data quality tools used to validate datasets, detect data quality issues, and monitor data reliability.

They help data engineering teams define rules and automatically check whether data meets business requirements.

For example, consider an e-commerce orders table:

| order_id | customer_id | amount | order_date |
|---:|---:|---:|---|
| 101 | 501 | 500.00 | 2026-10-01 |
| 102 | 502 | 300.00 | 2026-10-02 |
| 103 | NULL | 250.00 | 2026-10-03 |

Possible quality rules:

- `order_id` must not be NULL.
- `order_id` must be unique.
- `amount` must be greater than or equal to zero.
- `customer_id` must not be NULL.
- `order_date` must contain valid dates.

Instead of manually checking these conditions, Soda and Great Expectations allow us to define and automate them.

## 2. What Is Soda?

Soda is a data quality platform and toolset used to define, execute, and monitor data quality checks.

It helps teams identify problems such as:

- Missing values.
- Duplicate records.
- Unexpected row counts.
- Invalid values.
- Schema changes.
- Unusual data distributions.
- Data freshness problems.

Soda provides a declarative approach: users define checks, and Soda executes them against supported data sources.

### Main Components

| Component | Purpose |
|---|---|
| Soda Core | Open-source Python-based data quality library |
| SodaCL | YAML-based language for defining data quality checks |
| Soda Library | Product offering that includes data quality scanning capabilities |
| Soda Cloud | Platform for managing, monitoring, and collaborating on data quality, subject to product and plan availability |

Product names and packaging can change, so check the current Soda documentation when setting up a new project.

### How Soda Works

```text
Data Source
    |
    v
Soda Configuration
    |
    v
SodaCL Checks
    |
    v
Soda Scan
    |
    v
Validation Results
    |
    v
Alerts and Monitoring
```

### Example Soda Check

Suppose we want to validate an `orders` table.

A SodaCL checks file may look like this:

```yaml
checks for orders:
  - row_count > 0
  - missing_count(order_id) = 0
  - duplicate_count(order_id) = 0
  - missing_count(customer_id) = 0
  - min(amount) >= 0
```

Explanation:

| Check | Meaning |
|---|---|
| `row_count > 0` | Table must contain records |
| `missing_count(order_id) = 0` | No missing order IDs |
| `duplicate_count(order_id) = 0` | Order IDs must be unique |
| `missing_count(customer_id) = 0` | Customer ID cannot be missing |
| `min(amount) >= 0` | Amount cannot be negative |

The exact supported checks and syntax depend on the Soda version and data source.

## 3. What Is Great Expectations?

Great Expectations, commonly called GX, is a data quality framework used to define expectations and validate datasets.

It allows data engineers to define what valid data should look like and execute validations against actual data.

### Main Concepts

| Component | Purpose |
|---|---|
| Expectation | A rule that data must satisfy |
| Expectation Suite | A collection of expectations |
| Data Source | Connection to the underlying data system |
| Data Asset | A dataset or logical data object being validated |
| Batch | A selected portion of data being validated |
| Validation Definition | Connects a batch definition with an expectation suite |
| Checkpoint | Executes validation and can trigger follow-up actions |
| Data Docs | Presents validation results in a human-readable format |

The exact API and configuration workflow vary across Great Expectations versions. The concepts above are useful for understanding its validation model.

### How Great Expectations Works

```text
Data Source
    |
    v
Data Asset
    |
    v
Batch Definition
    |
    v
Expectation Suite
    |
    v
Validation Definition
    |
    v
Checkpoint
    |
    v
Validation Results
```

### Example Expectations

Suppose we have an orders dataset.

```python
import great_expectations as gx

context = gx.get_context()
```

This creates or loads a Great Expectations context.

Some examples of expectations include:

```python
# Illustrative expectation names:
expect_column_values_to_not_be_null
expect_column_values_to_be_unique
expect_column_values_to_be_between
expect_table_row_count_to_be_between
```

For example, the business rules could be:

| Column | Expectation |
|---|---|
| order_id | Values must not be NULL |
| order_id | Values must be unique |
| amount | Values must be between 0 and 100000 |
| customer_id | Values must not be NULL |
| order_date | Values must follow the expected date format |

The exact method signatures and execution setup should be taken from the documentation for the installed GX version. Avoid mixing legacy APIs with current APIs.

## 4. Soda vs Great Expectations

Both tools validate data quality, but their approaches and workflows differ.

| Feature | Soda | Great Expectations |
|---|---|---|
| Main purpose | Data quality checks and monitoring | Data validation and expectation management |
| Rule definition | SodaCL, YAML-based | Expectations, usually defined through Python APIs |
| Configuration style | Declarative | Programmatic and declarative project configuration |
| Validation | Soda scans | GX validations |
| Rule organization | Checks and check files | Expectation Suites |
| Results | Scan results and monitoring | Validation results and Data Docs |
| Data sources | Supported databases and data systems | Supported execution engines and data sources |
| Monitoring | Soda platform capabilities | Validation results and integrations |
| Customization | Checks, SQL and supported extensions | Custom expectations and validation workflows |
| Typical use | Data quality monitoring | Detailed data validation and testing |

### Simple Difference

- **Soda:** Define checks and run scans to monitor data quality.
- **Great Expectations:** Define expectations and validate datasets against them.

Both can be used in production data pipelines.

## 5. Implementing Data Quality Checks with Soda

Consider an orders table stored in a SQL database.

### Step 1: Install Soda

Installation depends on the Soda product and supported data source.

For a Soda Core setup, install the appropriate Soda package for the data source connector.

Example:

```bash
pip install soda-core-postgres
```

This example is for PostgreSQL. Other databases require their corresponding supported package.

### Step 2: Configure the Data Source

A Soda configuration file contains connection information.

Illustrative configuration:

```yaml
data_source orders_db:
  type: postgres
  connection:
    host: localhost
    port: 5432
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    database: ecommerce
```

Use environment variables or a secure secret manager for credentials. Do not commit passwords to Git.

### Step 3: Define Checks

Create a checks file:

```yaml
checks for orders:
  - row_count > 0
  - missing_count(order_id) = 0
  - duplicate_count(order_id) = 0
  - missing_count(customer_id) = 0
  - min(amount) >= 0
```

### Step 4: Run a Scan

A Soda scan executes the defined checks against the configured data source.

The command and configuration options depend on the installed Soda version.

Conceptually:

```text
soda scan
```

### Step 5: Review Results

The scan identifies which checks passed or failed.

Example:

| Check | Result |
|---|---|
| Row count greater than zero | Passed |
| No missing order IDs | Passed |
| No duplicate order IDs | Failed |
| No missing customer IDs | Failed |
| Minimum amount non-negative | Passed |

The team can investigate failed checks and decide whether to stop processing, quarantine records, or issue an alert.

## 6. Implementing Data Quality Checks with Great Expectations

Great Expectations can validate data through expectation suites and validation workflows.

### Step 1: Install GX

```bash
pip install great_expectations
```

### Step 2: Create or Load a Context

```python
import great_expectations as gx

context = gx.get_context()
```

A context manages the project's data sources, validation configuration, and related metadata.

### Step 3: Define Expectations

For an orders dataset, define rules such as:

```text
order_id must not be NULL
order_id must be unique
amount must be non-negative
customer_id must not be NULL
```

These rules can be organized into an Expectation Suite.

### Step 4: Configure Data Access

Connect GX to the relevant data source and define the dataset or batch to validate.

For example, the source might be:

- A PostgreSQL table.
- A Spark DataFrame.
- A file-based dataset.
- A supported cloud data source.

### Step 5: Execute Validation

Use a Validation Definition and Checkpoint workflow to run the expectation suite against a selected batch.

### Step 6: Review Results

GX produces validation results showing which expectations succeeded or failed.

Data Docs can present those results in a human-readable format when configured.

**Important:** This section describes the current conceptual workflow. Actual code for configuring data sources, assets, batches, and checkpoints depends on the GX version and execution environment.

## 7. Using Soda and Great Expectations in Data Engineering

Both tools can be integrated into ETL and ELT pipelines.

### Example: ADLS to Delta Lake

```text
          ADLS Gen2
              |
              v
         Bronze Layer
              |
              v
       PySpark Transform
              |
              v
         Silver Layer
              |
              v
      Data Quality Checks
         /          \
        /            \
     Passed         Failed
       |               |
       v               v
   Gold Layer       Quarantine
       |
       v
   Reporting
```

### Where Can We Use Them?

| Pipeline Stage | Example Check |
|---|---|
| Bronze | File exists, schema is available |
| Silver | No missing primary keys |
| Silver | Duplicate business keys are detected |
| Silver | Amount values follow business rules |
| Gold | KPI calculations are valid |
| Gold | Daily totals match expected aggregates |
| Production | Freshness and volume are within expected ranges |

### Example Pipeline Flow

1. Read raw files from ADLS Gen2.
2. Apply transformations using PySpark.
3. Write the transformed dataset to a Silver Delta table.
4. Run Soda or GX validations.
5. If critical checks pass, continue to the Gold layer.
6. If checks fail, record the failure and quarantine or hold the affected data.
7. Monitor validation results.

## 8. Integrating Data Quality with CI/CD

Data quality checks can be automated through CI/CD pipelines.

Example:

```text
Developer Changes Code
          |
          v
       Git Push
          |
          v
      CI Pipeline
          |
          v
   Unit and Integration Tests
          |
          v
   Data Quality Validation
          |
       /      \
      /        \
   Passed     Failed
     |           |
     v           v
  Deploy       Stop
     |
     v
 Production Monitoring
```

### Benefits

- Detect defects before deployment.
- Prevent invalid transformations from reaching production.
- Maintain consistent quality rules.
- Track changes to validation rules.
- Improve deployment confidence.

A CI pipeline may validate test datasets, while production scans validate actual production data. These are related but distinct activities.

## 9. When Should You Use Soda?

Soda may be a suitable choice when:

- You want to define checks in YAML.
- You need repeatable data quality scans.
- You want to monitor data quality across datasets.
- Your team prefers declarative checks.
- You want to integrate data quality checks into pipelines.

## 10. When Should You Use Great Expectations?

Great Expectations may be suitable when:

- You want to organize detailed expectations into suites.
- You need Python-oriented validation workflows.
- You want to validate datasets and batches.
- You need customizable expectations.
- You want human-readable validation documentation through Data Docs.

Neither tool is universally better. The choice depends on the data architecture, team skills, integrations, monitoring needs, and operational requirements.

## 11. Common Mistakes

1. **Defining rules without business context:** A technically valid value may still violate business requirements.
2. **Checking only NULL values:** Data quality includes uniqueness, validity, accuracy, consistency, and freshness.
3. **Ignoring failed checks:** Every failure should have an owner and response strategy.
4. **Hardcoding credentials:** Use environment variables or secret managers.
5. **Mixing API versions:** Great Expectations APIs have evolved; use documentation matching the installed version.
6. **Assuming all checks work on every data source:** Check support depends on the connector and execution engine.
7. **Treating every failure as critical:** Define severity and acceptable thresholds.
8. **Running expensive checks unnecessarily:** Full-table scans can increase processing cost.
9. **Ignoring historical trends:** A gradual rise in NULL values may be more important than one isolated failure.
10. **Assuming the tool fixes data automatically:** These tools detect and report problems; correction and recovery require pipeline logic.

## 12. Practical Interview Scenario

**Question: You are building an Azure data pipeline using ADLS Gen2, Azure Databricks, and Delta Lake. How would you use Soda or Great Expectations to maintain data quality?**

### Answer

I would define data quality rules based on the business requirements and execute them at appropriate pipeline stages.

**Step 1: Ingest data**

Read raw files from ADLS Gen2 into the Bronze layer.

**Step 2: Define expectations**

Create rules for:
- Required columns.
- NULL values.
- Duplicate business keys.
- Valid amount ranges.
- Valid dates.
- Referential integrity.

**Step 3: Apply transformations**

Use PySpark to clean and transform the data.

**Step 4: Run validations**

Use Soda or Great Expectations to execute the defined checks against the Silver dataset.

**Step 5: Handle failures**

If critical rules fail, stop downstream processing or quarantine invalid records. Record the failed rules and relevant metrics.

**Step 6: Continue processing**

Allow validated data to move into the Gold layer.

**Step 7: Monitor**

Track validation results and data quality trends across pipeline runs.

This approach makes data quality checks repeatable and helps prevent invalid data from reaching downstream reporting systems.

## 13. Common Interview Questions and Answers

### Q1. What is Soda?

Soda is a data quality platform and toolset used to define, execute, and monitor data quality checks.

### Q2. What is Great Expectations?

Great Expectations is a data quality framework that allows users to define expectations and validate datasets against them.

### Q3. What is SodaCL?

SodaCL is a YAML-based language used to define data quality checks in Soda.

### Q4. What is an Expectation Suite?

An Expectation Suite is a collection of data quality expectations in Great Expectations.

### Q5. What is a Checkpoint in Great Expectations?

A Checkpoint is a configured workflow for running validation against a selected batch and handling the results.

### Q6. What is Data Docs?

Data Docs presents Great Expectations validation results and related metadata in a human-readable format.

### Q7. What is the main difference between Soda and Great Expectations?

Soda emphasizes declarative checks and data quality monitoring, while Great Expectations organizes validation around expectations, suites, batches, and validation workflows.

### Q8. Can Soda and Great Expectations be used with PySpark?

Yes, depending on the tool version, supported execution engine, and integration setup.

### Q9. Can these tools validate data in Delta Lake?

They can validate supported data sources and execution paths involving Delta Lake. The exact integration depends on the product version and environment.

### Q10. Do these tools automatically clean bad data?

No. They detect and report data quality issues. The pipeline must define how invalid data is corrected, quarantined, or rejected.

### Q11. Can data quality checks run in CI/CD?

Yes. They can be included in automated test and deployment workflows.

### Q12. Which tool should you choose?

Choose based on the team's preferred rule-definition style, supported data sources, integrations, monitoring needs, and operational requirements.

## 14. Quick Revision

| Concept | Meaning |
|---|---|
| Soda | Data quality toolset and platform |
| Soda Core | Open-source Python-based data quality library |
| SodaCL | YAML-based check definition language |
| Soda Scan | Executes data quality checks |
| Great Expectations | Data quality validation framework |
| Expectation | A rule describing acceptable data |
| Expectation Suite | Collection of expectations |
| Data Source | System containing the data |
| Data Asset | Dataset being validated |
| Batch | Selected data to validate |
| Validation Definition | Connects a batch definition and expectation suite |
| Checkpoint | Runs validation workflow |
| Data Docs | Human-readable validation documentation |

### Remember These Five Points

1. Soda uses SodaCL to define data quality checks.
2. Great Expectations uses expectations and suites to define validation rules.
3. Both can automate data quality checks in data pipelines.
4. Both help identify invalid or unexpected data.
5. Neither automatically guarantees that bad data is corrected.

## 15. Self-Assessment

Try answering these questions without looking at the notes.

1. What is Soda?
2. What is Great Expectations?
3. What is SodaCL?
4. What is an Expectation Suite?
5. What is the purpose of a Checkpoint?
6. What are Data Docs?
7. Explain the main difference between Soda and Great Expectations.
8. How do you define a uniqueness check in Soda?
9. How are expectations organized in Great Expectations?
10. Can these tools be integrated with PySpark?
11. How would you validate a Delta Lake Silver table?
12. How do you handle failed data quality checks?
13. How can data quality checks be included in CI/CD?
14. What factors influence the choice between Soda and Great Expectations?
15. Design a data quality validation workflow for an ADLS-to-Delta pipeline.

**Interview readiness:** Be able to explain Soda, SodaCL, Great Expectations, Expectation Suites, Checkpoints, and how you would integrate data quality validation into an Azure Databricks pipeline.

