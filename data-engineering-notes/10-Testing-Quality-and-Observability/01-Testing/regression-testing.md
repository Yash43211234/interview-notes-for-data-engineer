

# Regression Testing in Data Engineering

## 1. What Is Regression Testing?

Regression testing is the process of testing an existing system after a code change, configuration change, bug fix, or pipeline update to ensure that previously working functionality still works correctly.

The main objective is to identify unintended side effects introduced by a new change.

### Simple example

Suppose an existing data pipeline calculates daily sales revenue.

Initially:

```text
Total Revenue = 100,000
```

A developer modifies the transformation logic to add a discount calculation.

After the change, the pipeline produces:

```text
Total Revenue = 92,000
```

The reduction might be correct if discounts were previously excluded. However, it might also indicate that the existing revenue calculation was accidentally broken.

Regression testing helps verify that:
- Existing calculations still work.
- Previously supported records are processed correctly.
- Output schemas remain compatible.
- Existing business rules are preserved.
- New changes do not break downstream systems.

**Key idea:** Regression testing checks whether a new change has broken something that was already working.

---

## 2. Why Is Regression Testing Important in Data Engineering?

Data engineering pipelines contain multiple interconnected components:

```text
Source Systems
      |
      v
Data Ingestion
      |
      v
Raw Layer
      |
      v
Transformation
      |
      v
Curated Layer
      |
      v
Data Warehouse
      |
      v
Reports and Analytics
```

A small modification in one transformation can affect multiple downstream datasets.

### Example

Suppose a developer changes the customer transformation logic.

Previously:

```python
customers = customers.filter(
    customers["status"] == "ACTIVE"
)
```

The developer changes it to:

```python
customers = customers.filter(
    customers["status"] != "INACTIVE"
)
```

The new condition may include customers with NULL status values differently from the original logic.

This can affect:
- Customer counts.
- Revenue reports.
- Customer segmentation.
- Downstream joins.
- Business dashboards.

Regression testing helps detect these unintended changes before deployment.

### Main benefits

1. Prevents existing functionality from breaking.
2. Detects unintended changes in business logic.
3. Protects data quality.
4. Reduces production failures.
5. Builds confidence in code changes.
6. Supports safer CI/CD deployments.
7. Protects downstream consumers.

---

## 3. Regression Testing vs Other Testing Types

Regression testing is often confused with unit testing, integration testing, and system testing.

| Testing Type | Main Purpose |
|---|---|
| Unit Testing | Tests an individual function or component |
| Integration Testing | Tests interactions between components |
| System Testing | Tests the complete system |
| Regression Testing | Checks whether existing behavior still works after changes |
| Smoke Testing | Quickly checks whether critical functionality works |
| Acceptance Testing | Verifies whether the solution meets business requirements |
| Performance Testing | Evaluates speed, throughput, and resource usage |

### Example

Suppose a PySpark pipeline contains:
- A customer transformation.
- An order transformation.
- A customer-order join.
- A final revenue aggregation.

Different tests can be applied:

- **Unit testing:** Test the customer transformation function.
- **Integration testing:** Verify that customer and order datasets join correctly.
- **System testing:** Execute the complete pipeline.
- **Regression testing:** Compare the new pipeline results with expected existing behavior.
- **Smoke testing:** Verify that the pipeline starts and produces its essential output.

**Important:** These testing types are not mutually exclusive. Regression tests can include unit, integration, and end-to-end tests.

---

## 4. When Should We Perform Regression Testing?

Regression testing should be performed whenever a change could affect existing behavior.

Common situations include:

| Situation | Example |
|---|---|
| Code modification | Changing a PySpark transformation |
| Bug fix | Fixing incorrect revenue calculations |
| Schema change | Adding a new customer column |
| Business logic change | Modifying discount calculations |
| Library upgrade | Upgrading the Spark version |
| Configuration change | Changing partition settings |
| Pipeline modification | Adding a new ingestion step |
| Database migration | Moving from one database to another |
| Performance optimization | Replacing a join strategy |
| Production deployment | Releasing a new pipeline version |

### Practical scenario

A data engineer changes a SQL query to improve performance.

The query runs faster, but the result contains fewer records.

Regression testing should verify:
- Row counts.
- Aggregated values.
- Duplicate records.
- NULL handling.
- Join results.
- Business rules.

Performance improvement should not come at the cost of incorrect results.

---

## 5. Regression Testing in Data Pipelines

A data pipeline can be tested at different stages.

### A. Source-to-Raw Testing

Verify that source data is ingested correctly.

Checks:
- Expected files are processed.
- Records are not unintentionally lost.
- Source columns are preserved.
- Data types are handled correctly.

### B. Raw-to-Transformed Testing

Verify transformation logic.

Checks:
- Filters work correctly.
- Joins produce expected records.
- Aggregations are correct.
- NULL values are handled properly.
- Business rules remain unchanged.

### C. Transformed-to-Curated Testing

Verify the final output.

Checks:
- Expected schema.
- Correct row counts.
- No unexpected duplicates.
- Correct metrics.
- Referential integrity.

### D. Downstream Testing

Verify that downstream consumers still work.

Checks:
- Reports receive expected columns.
- Dashboards calculate metrics correctly.
- Applications can read the updated dataset.
- Existing queries remain compatible.

---

## 6. Data Validation Techniques for Regression Testing

Regression testing in data engineering often relies on comparing the old and new pipeline outputs.

### 6.1 Row Count Validation

Compare the number of records before and after a change.

```python
old_count = old_df.count()
new_count = new_df.count()

assert old_count == new_count
```

This is useful when the expected behavior is that the number of records remains unchanged.

However, matching row counts do not prove that the actual records are identical.

### 6.2 Schema Validation

Check whether the expected columns and data types remain unchanged.

```python
expected_columns = [
    "order_id",
    "customer_id",
    "amount",
    "order_date"
]

assert new_df.columns == expected_columns
```

For stronger validation, compare the complete schema:

```python
assert new_df.schema == expected_schema
```

This checks schema details such as field names, data types, and nullability.

### 6.3 Duplicate Validation

Check whether unexpected duplicates have been introduced.

```python
duplicate_count = (
    new_df.groupBy("order_id")
    .count()
    .filter("count > 1")
    .count()
)

assert duplicate_count == 0
```

This assumes `order_id` is expected to be unique.

### 6.4 NULL Validation

Check whether required columns contain unexpected NULL values.

```python
from pyspark.sql.functions import col

null_count = (
    new_df
    .filter(col("order_id").isNull())
    .count()
)

assert null_count == 0
```

### 6.5 Aggregate Validation

Compare business metrics.

```python
from pyspark.sql.functions import sum

old_revenue = old_df.agg(
    sum("amount")
).first()[0]

new_revenue = new_df.agg(
    sum("amount")
).first()[0]

assert old_revenue == new_revenue
```

Use exact equality only when the calculation and numeric representation make it appropriate. For floating-point calculations, use a defined tolerance.

### 6.6 Record-Level Comparison

Compare records using a stable business key.

```python
from pyspark.sql.functions import col

old_alias = old_df.alias("old")
new_alias = new_df.alias("new")

differences = (
    old_alias.join(
        new_alias,
        on="order_id",
        how="full_outer"
    )
    .filter(
        (col("old.amount") != col("new.amount")) |
        col("old.order_id").isNull() |
        col("new.order_id").isNull()
    )
)
```

This identifies missing or changed records for the illustrated non-NULL amount case.

In production, handle NULL-safe comparisons and duplicate keys explicitly. If multiple records can share a key, use a composite key or compare record multiplicities rather than assuming one-to-one matching.

---

## 7. Golden Dataset Testing

A golden dataset is a small, trusted dataset with known expected outputs.

It is used to verify that transformation logic continues to produce correct results after code changes.

### Example

Input:

| order_id | amount | discount |
|---|---:|---:|
| 101 | 1000 | 100 |
| 102 | 500 | 50 |
| 103 | 800 | 0 |

Expected output:

| order_id | final_amount |
|---|---:|
| 101 | 900 |
| 102 | 450 |
| 103 | 800 |

Transformation:

```python
from pyspark.sql.functions import col

result = orders.withColumn(
    "final_amount",
    col("amount") - col("discount")
)
```

Expected result:

```text
101 -> 900
102 -> 450
103 -> 800
```

If a developer changes the discount logic, the golden dataset test can detect an unexpected result.

### Benefits

- Repeatable testing.
- Known expected results.
- Easy bug detection.
- Useful for complex transformations.
- Suitable for CI/CD pipelines.

**Important:** Golden datasets should cover normal records, boundary conditions, NULL values, duplicates, and relevant edge cases.

---

## 8. Regression Testing in SQL

SQL transformations can also be regression-tested.

Suppose an existing query calculates monthly revenue.

### Original query

```sql
SELECT
    DATE_TRUNC('month', order_date) AS order_month,
    SUM(amount) AS total_revenue
FROM orders
GROUP BY DATE_TRUNC('month', order_date);
```

A developer changes the query to include only completed orders.

### Updated query

```sql
SELECT
    DATE_TRUNC('month', order_date) AS order_month,
    SUM(amount) AS total_revenue
FROM orders
WHERE status = 'COMPLETED'
GROUP BY DATE_TRUNC('month', order_date);
```

This is a business logic change, so the results are expected to differ.

Regression testing should verify that:
- Completed orders are included.
- Cancelled orders are excluded.
- NULL statuses are handled according to the requirement.
- Monthly aggregation remains correct.
- Existing output columns and types remain compatible.

### SQL comparison example

```sql
SELECT
    COALESCE(old.order_month, new.order_month) AS order_month,
    old.total_revenue AS old_revenue,
    new.total_revenue AS new_revenue
FROM old_monthly_revenue old
FULL OUTER JOIN new_monthly_revenue new
    ON old.order_month = new.order_month
WHERE old.total_revenue IS DISTINCT FROM new.total_revenue;
```

`IS DISTINCT FROM` provides NULL-safe comparison in SQL engines that support it. Syntax may differ across database platforms.

---

## 9. Regression Testing in PySpark

A common approach is to use `pytest` with small DataFrame fixtures.

### Example transformation

```python
from pyspark.sql.functions import col

def calculate_final_amount(df):
    return df.withColumn(
        "final_amount",
        col("amount") - col("discount")
    )
```

### Test example

```python
def test_calculate_final_amount(spark):
    input_data = [
        (101, 1000.0, 100.0),
        (102, 500.0, 50.0)
    ]

    input_df = spark.createDataFrame(
        input_data,
        ["order_id", "amount", "discount"]
    )

    result = calculate_final_amount(input_df)

    actual = {
        row["order_id"]: row["final_amount"]
        for row in result.collect()
    }

    expected = {
        101: 900.0,
        102: 450.0
    }

    assert actual == expected
```

### Why use pytest?

- Organizes test cases.
- Supports reusable fixtures.
- Provides readable assertions.
- Integrates with CI/CD.
- Makes automated regression testing easier.

For larger DataFrame comparisons, libraries such as `chispa` can help compare schemas and rows.

Remember that DataFrames are distributed and unordered. If row order matters to the test, explicitly sort the results before comparing them.

---

## 10. Regression Testing in Databricks and Delta Lake

Databricks pipelines commonly use Delta tables as intermediate and final datasets.

Regression testing can validate:
- Schema compatibility.
- Row counts.
- Duplicate records.
- Business metrics.
- Data changes between pipeline versions.
- Downstream table compatibility.

### Example: Compare two Delta table versions

Delta Lake supports time travel when the required transaction-log history and data files are retained.

```sql
SELECT *
FROM orders VERSION AS OF 10;
```

This reads an older table version.

Current version:

```sql
SELECT *
FROM orders;
```

### Compare aggregate results

```sql
SELECT
    SUM(amount) AS total_revenue,
    COUNT(*) AS total_orders
FROM orders VERSION AS OF 10;
```

Compare with the current table:

```sql
SELECT
    SUM(amount) AS total_revenue,
    COUNT(*) AS total_orders
FROM orders;
```

### Important

Time travel can help compare historical table states, but it is not a replacement for a proper test dataset.

Old versions may become unavailable after required files are removed through retention and VACUUM operations.

---

## 11. Regression Testing in CI/CD

Regression testing is especially useful before deploying pipeline changes to production.

### Typical workflow

```text
Developer Changes Code
          |
          v
       Git Commit
          |
          v
      Pull Request
          |
          v
     Build / Validate
          |
          v
     Run Unit Tests
          |
          v
   Run Regression Tests
          |
          v
   Integration Tests
          |
          v
       Approval
          |
          v
     Deploy to Production
          |
          v
    Post-Deployment Checks
```

### Example CI pipeline stages

| Stage | Purpose |
|---|---|
| Code Validation | Check formatting and basic code issues |
| Unit Tests | Test individual functions |
| Regression Tests | Check previously working behavior |
| Integration Tests | Test component interactions |
| Deployment | Release approved changes |
| Smoke Tests | Verify critical functionality after deployment |

### Why automate regression testing?

Manual testing becomes difficult when:
- Pipelines contain many transformations.
- Multiple developers make changes.
- Deployments happen frequently.
- Datasets are large.
- Downstream dependencies are numerous.

Automation helps run the same tests consistently for every relevant code change.

---

## 12. Regression Testing vs Data Quality Testing

These concepts are related but have different objectives.

### Data Quality Testing

Checks whether data meets defined quality rules.

Examples:
- Order IDs should not be NULL.
- Customer IDs should be valid.
- Amounts should not be negative.
- Business keys should be unique.

### Regression Testing

Checks whether a code or pipeline change has introduced unintended differences in previously working behavior.

Examples:
- Revenue calculation changed unexpectedly.
- A join now drops valid customers.
- A previously available column disappeared.
- A transformation produces different records after a library upgrade.

| Data Quality Testing | Regression Testing |
|---|---|
| Checks data against quality rules | Checks behavior after changes |
| Focuses on data correctness | Focuses on unintended changes |
| Can run continuously | Usually triggered by relevant changes and releases |
| Uses business rules and constraints | Uses baselines, expected outputs, and compatibility checks |

A regression test may also contain data quality assertions.

---

## 13. Challenges in Regression Testing

### A. Large Data Volumes

Comparing complete production datasets can be expensive.

Possible solutions:
- Use representative test datasets.
- Compare partition-level metrics.
- Use targeted record comparisons.
- Validate important business aggregates.

### B. Changing Business Requirements

Sometimes output differences are intentional.

The expected baseline must be updated only after the business change is reviewed and approved.

### C. Nondeterministic Results

Distributed processing can produce different row orders or floating-point aggregation results.

Solutions:
- Compare results without relying on row order.
- Use deterministic tie-breakers.
- Apply numeric tolerances where appropriate.
- Control random inputs in tests.

### D. Schema Evolution

Adding or changing columns can affect downstream consumers.

Solutions:
- Validate schema compatibility.
- Test downstream queries.
- Review breaking changes.
- Maintain versioned data contracts.

### E. Test Data Privacy

Production data may contain personal or confidential information.

Solutions:
- Use synthetic or masked data.
- Apply least-privilege access.
- Avoid copying sensitive production records unnecessarily.

---

## 14. Best Practices

1. Maintain a reliable baseline of expected outputs.
2. Create small, representative golden datasets.
3. Test business-critical transformations.
4. Validate schema and data types.
5. Compare important aggregates.
6. Check duplicates and NULL values.
7. Include boundary cases and edge cases.
8. Use stable keys for record comparisons.
9. Avoid relying on row order unless explicitly required.
10. Automate tests in CI/CD.
11. Test downstream compatibility.
12. Review intentional output changes with business stakeholders.
13. Keep test data separate from production data.
14. Preserve source data or reproducible inputs for reprocessing.
15. Run post-deployment smoke tests.

---

## 15. Practical Interview Scenario

**Question: You modified a PySpark transformation to improve performance. How would you ensure that the change does not break existing functionality?**

### Answer

I would perform regression testing before deploying the modified transformation.

My approach would be:

1. Use the same input dataset for the old and new implementations.
2. Compare output schemas.
3. Compare record counts.
4. Validate business-critical aggregations.
5. Check duplicates and NULL values.
6. Compare records using stable business keys.
7. Test important edge cases.
8. Run integration tests with downstream transformations.
9. Execute automated regression tests through CI/CD.
10. Deploy only after the expected results are verified.

If differences are intentional, I would validate them against the updated business requirements.

---

## 16. Common Interview Questions and Answers

### Q1. What is regression testing?

Regression testing verifies that existing functionality continues to work after code, configuration, or system changes.

### Q2. Why is regression testing important in data engineering?

A small transformation change can affect multiple downstream tables, reports, and business metrics. Regression testing helps identify unintended changes before production deployment.

### Q3. What is the difference between unit testing and regression testing?

Unit testing checks individual components. Regression testing checks whether existing behavior remains correct after changes. Unit tests can be part of a regression suite.

### Q4. How do you perform regression testing in PySpark?

I compare old and new outputs using schema checks, row counts, business aggregates, duplicate checks, and record-level comparisons. I also test edge cases using automated test frameworks.

### Q5. What is a golden dataset?

A golden dataset is a trusted test dataset with known expected results. It is used to verify that transformation logic continues to produce correct output.

### Q6. How do you compare old and new pipeline outputs?

I compare schemas, counts, aggregates, and records using stable business keys. I also handle NULL values and duplicate keys carefully.

### Q7. Can row-count comparison alone prove that a pipeline is correct?

No. Two datasets can have identical row counts but contain different records or incorrect values.

### Q8. How is regression testing used in CI/CD?

Regression tests run automatically after relevant code changes and before deployment. They help prevent changes that break existing pipeline behavior from reaching production.

### Q9. What is the difference between regression testing and data quality testing?

Data quality testing checks defined data rules. Regression testing checks whether changes have introduced unintended differences in existing behavior.

### Q10. How do you test a schema change?

I validate the expected schema, check compatibility with downstream consumers, test affected transformations, and verify that existing queries continue to work.

### Q11. How do you test a performance optimization?

I compare the old and new outputs using the same inputs, verify business results, and then measure execution time and resource usage.

### Q12. What is a regression baseline?

A regression baseline is a trusted reference representing expected system behavior or output against which new results are compared.

### Q13. Should regression tests always expect identical output?

No. If a business requirement intentionally changes, the expected output should change accordingly. Tests should verify the new approved behavior and protect unrelated existing functionality.

### Q14. How would you handle a regression failure?

I would identify the affected component, compare old and new results, inspect the code change, determine whether the difference is intentional, fix the issue, and rerun the relevant tests.

### Q15. What is the role of regression testing after a Spark upgrade?

It helps identify changes in output behavior, schema handling, query execution, library compatibility, and downstream integrations.

---

## 17. Common Mistakes to Avoid

1. **Checking only row counts:** Equal counts do not guarantee equal data.
2. **Ignoring schema changes:** A transformation may produce correct values but break downstream consumers.
3. **Comparing data without stable keys:** This can produce misleading results.
4. **Ignoring NULL handling:** NULL comparisons require explicit logic.
5. **Assuming every output difference is a bug:** Some changes are intentional business updates.
6. **Ignoring duplicate records:** Duplicate keys can make record comparisons ambiguous.
7. **Testing only happy-path data:** Boundary cases and invalid records also matter.
8. **Relying only on manual testing:** Automated tests are more repeatable.
9. **Ignoring downstream dependencies:** A locally correct transformation can still break reports or other pipelines.
10. **Using production data carelessly:** Protect sensitive information in test environments.

---

## 18. Quick Revision

| Concept | Meaning |
|---|---|
| Regression Testing | Checks existing behavior after changes |
| Baseline | Trusted reference output |
| Golden Dataset | Test data with known expected results |
| Unit Testing | Tests individual components |
| Integration Testing | Tests component interactions |
| Schema Validation | Checks expected columns and data types |
| Row Count Validation | Checks record counts |
| Record Comparison | Identifies missing or changed records |
| Data Quality Testing | Validates defined data rules |
| CI/CD Testing | Automates validation before deployment |

### Remember These Five Points

1. Regression testing protects existing functionality.
2. It is important after code, configuration, schema, and platform changes.
3. Compare schemas, counts, aggregates, and records.
4. Golden datasets provide repeatable expected results.
5. Automate regression tests in CI/CD to catch problems before production.

---

## 19. Self-Assessment

Try answering these questions without looking at the notes.

1. What is regression testing?
2. Why is it important in data engineering?
3. What is the difference between regression testing and unit testing?
4. When should regression testing be performed?
5. How would you compare old and new PySpark outputs?
6. Why is row-count validation insufficient by itself?
7. What is a golden dataset?
8. How can SQL transformations be regression-tested?
9. How can Delta Lake time travel help in regression testing?
10. How is regression testing integrated into CI/CD?
11. What is the difference between regression testing and data quality testing?
12. How would you test a Spark performance optimization?
13. How would you handle intentional output differences?
14. What challenges occur when comparing large datasets?
15. Design a regression testing strategy for an e-commerce order pipeline.

**Interview readiness:** You should be able to explain regression testing clearly, compare old and new pipeline outputs, describe golden datasets, and explain how you would automate regression testing for a PySpark or SQL pipeline.

