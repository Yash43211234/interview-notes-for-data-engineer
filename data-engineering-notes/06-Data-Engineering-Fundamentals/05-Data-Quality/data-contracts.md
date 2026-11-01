# Data Contracts in Data Engineering

## 1. Introduction

A **data contract is an agreement between a data producer and data consumer that defines how data should be structured, delivered, and maintained.**

It specifies expectations such as:

- Schema and column names.
- Data types and required fields.
- Data quality rules.
- Business definitions.
- Delivery frequency and freshness.
- Ownership and responsibilities.
- Compatibility and change-management rules.

Data contracts help prevent unexpected changes in source systems from breaking downstream data pipelines.

### Example

Suppose an application team provides order data to a data engineering team.

The application team agrees to provide:

| Column | Data Type | Requirement |
|---|---|---|
| order_id | STRING | Required and unique |
| customer_id | STRING | Required |
| amount | DECIMAL(12,2) | Must be non-negative |
| order_date | TIMESTAMP | Required |
| status | STRING | Must follow approved values |

The agreement also states that data will arrive every hour.

If the application team renames `order_id` or changes `amount` from decimal to string without informing the data engineering team, downstream pipelines may fail.

A data contract establishes expectations and a process for managing such changes.

**Key idea:** A data contract makes data producers and consumers agree on what data means, how it is structured, and what quality and delivery standards it must meet.

---

## 2. Why Are Data Contracts Needed?

In a data platform, multiple teams may produce and consume data.

For example:

- Application teams generate transactional data.
- Data engineering teams ingest and transform it.
- Analytics teams build reports.
- Data science teams use it for machine learning.

Without clear agreements, changes made by one team can affect several downstream systems.

### Common Problems Without Data Contracts

1. A source column is renamed unexpectedly.
2. A data type changes and causes pipeline failures.
3. Required fields start containing NULL values.
4. Data arrives later than expected.
5. Business definitions differ between teams.
6. Source systems change without notifying consumers.
7. Downstream reports produce incorrect results.

### Benefits

- Clear expectations between teams.
- Fewer unexpected pipeline failures.
- Better data quality.
- Controlled schema changes.
- Clear ownership and accountability.
- Improved trust in data.
- Easier debugging and incident management.

---

## 3. Main Components of a Data Contract

A data contract commonly contains the following sections.

### 3.1 Schema Definition

Defines the expected structure of the dataset.

Example:

```text
orders
------
order_id       STRING
customer_id    STRING
amount         DECIMAL(12,2)
order_date     TIMESTAMP
status         STRING
```

It may also define:
- Column names.
- Data types.
- Nullable or required fields.
- Nested structures.
- Primary or business keys.

### 3.2 Data Quality Rules

Defines the quality requirements that incoming data must satisfy.

Example:

| Column | Rule |
|---|---|
| order_id | Must not be NULL |
| customer_id | Must not be NULL |
| amount | Must be greater than or equal to zero |
| status | Must belong to an approved list |
| order_id | Must be unique |

### 3.3 Freshness and Delivery Expectations

Defines when data should be available.

Example:

```text
Dataset: orders
Expected delivery: Every hour
Maximum acceptable delay: 30 minutes
```

Freshness expectations should be measurable and agreed upon by both teams.

### 3.4 Business Definitions

Defines what each field means.

Example:

| Field | Business Meaning |
|---|---|
| order_amount | Total order value before applicable adjustments |
| order_date | Timestamp when the order was placed |
| customer_id | Unique identifier assigned to a customer |
| order_status | Current business status of the order |

This prevents different teams from interpreting the same column differently.

### 3.5 Ownership and Responsibilities

Defines who is responsible for the dataset.

Example:

| Responsibility | Owner |
|---|---|
| Generate source data | Application team |
| Maintain source schema | Application team |
| Ingest and transform data | Data engineering team |
| Monitor downstream reports | Analytics team |

### 3.6 Change-Management Rules

Defines how schema and data changes should be communicated.

Example:

- Additive changes must be documented.
- Breaking changes require advance notice.
- Consumers must have time to update their pipelines.
- New schema versions should be tested before production deployment.

### 3.7 Security and Access Rules

A contract may also define:
- Sensitive fields.
- Personal information classification.
- Permitted data consumers.
- Masking requirements.
- Retention requirements.
- Access restrictions.

These rules should align with the organization's security and governance policies.

---

## 4. Data Contract Example

Consider an order dataset shared by an e-commerce application.

### Contract Definition

```yaml
dataset: orders

owner:
  producer: ecommerce_application
  consumer: data_engineering

schema:
  - name: order_id
    type: string
    required: true
    unique: true

  - name: customer_id
    type: string
    required: true

  - name: amount
    type: decimal(12,2)
    required: true
    minimum: 0

  - name: order_date
    type: timestamp
    required: true

  - name: status
    type: string
    required: true
    allowed_values:
      - Pending
      - Shipped
      - Delivered
      - Cancelled

delivery:
  frequency: hourly
  freshness_sla: 30 minutes

change_policy:
  breaking_changes_require_notice: true
  schema_versioning: true
```

This is an illustrative contract format, not a universal standard.

Different organizations may use YAML, JSON, schema registries, platform-specific tools, or custom metadata systems.

### What Does This Contract Define?

- Expected columns and data types.
- Required fields.
- Uniqueness requirements.
- Business validation rules.
- Delivery frequency.
- Freshness expectation.
- Ownership.
- Change-management expectations.

---

## 5. Producer and Consumer Responsibilities

A data contract creates shared responsibilities.

### Data Producer

The producer is the system or team that creates and publishes the data.

Responsibilities:

- Follow the agreed schema.
- Provide valid data.
- Meet delivery expectations.
- Communicate schema changes.
- Document business definitions.
- Maintain source data reliability.

Example: An e-commerce application generating order transactions.

### Data Consumer

The consumer is the system or team that uses the published data.

Responsibilities:

- Follow the agreed data structure.
- Validate incoming data.
- Handle supported schema versions.
- Report contract violations.
- Avoid assuming undocumented behavior.
- Communicate consumer requirements.

Example: A data engineering team ingesting orders into a Delta Lake table.

### Comparison

| Producer | Consumer |
|---|---|
| Generates data | Uses data |
| Publishes the schema | Validates the schema |
| Maintains source quality | Applies downstream quality checks |
| Communicates changes | Updates dependent pipelines |
| Meets delivery expectations | Monitors data availability |

Both sides should agree on the contract rather than treating it as a one-sided document.

---

## 6. Data Contracts and Schema Evolution

Data contracts are closely related to schema evolution.

Schema evolution handles changes in data structure. A data contract defines which changes are allowed and how they should be communicated.

### Example

Initial schema:

```text
order_id
customer_id
amount
```

New schema:

```text
order_id
customer_id
amount
payment_method
```

Adding `payment_method` may be an acceptable additive change if the contract allows it.

Now consider this change:

```text
customer_id → customer_number
```

Renaming a field may break downstream consumers.

The contract should define whether this is a breaking change and how consumers will be migrated.

### Types of Changes

| Change | Typical Impact | Contract Consideration |
|---|---|---|
| Add nullable column | Often additive | Document and communicate |
| Add required column | May break consumers or producers | Define default or migration plan |
| Remove column | Breaking | Provide migration period |
| Rename column | Breaking for existing references | Version or migration strategy |
| Change data type | Depends on compatibility | Validate before release |
| Change business meaning | Potentially serious | Document semantic change |

A change is not automatically safe just because the physical schema remains compatible. Business meaning can change without a column name or data type changing.

---

## 7. Data Contracts vs Schema Registry

These concepts are related but not identical.

### Data Contract

Defines the complete agreement between producers and consumers.

It can include:
- Schema.
- Data quality.
- Business definitions.
- Ownership.
- Freshness.
- Security.
- Change-management rules.

### Schema Registry

A schema registry is a service that stores and manages schemas for data producers and consumers.

It is commonly used with serialization formats such as Avro, Protobuf, and JSON Schema.

It can support:
- Schema registration.
- Schema versioning.
- Compatibility checks.
- Schema retrieval by applications.

### Comparison

| Data Contract | Schema Registry |
|---|---|
| Defines the broader data agreement | Stores and manages schemas |
| Includes quality and business rules | Focuses primarily on schema management |
| Defines ownership and delivery expectations | Supports schema versioning and compatibility |
| Can use a schema registry | Does not replace the full contract |

**Remember:** A schema registry can help enforce part of a data contract, but it is not the entire contract.

---

## 8. Data Contracts in a Data Pipeline

A data contract can be checked at multiple stages of a pipeline.

### Example Architecture

```text
Source Application
        |
        v
Published Data Contract
        |
        v
Data Ingestion
        |
        v
Contract Validation
        |
        +----------------------+
        |                      |
        v                      v
Valid Data              Contract Violations
        |                      |
        v                      v
Bronze / Silver          Quarantine / Alerts
        |
        v
Transformation
        |
        v
Gold Tables
        |
        v
Analytics and Reports
```

### Step-by-Step Process

**Step 1: Define the Contract**

Producer and consumer agree on the schema, quality rules, delivery expectations, and ownership.

**Step 2: Publish the Data**

The producer generates data according to the agreed contract.

**Step 3: Validate Incoming Data**

The ingestion pipeline checks:
- Required columns.
- Data types.
- Required values.
- Allowed values.
- Uniqueness.
- Freshness.

**Step 4: Handle Violations**

Depending on severity:
- Reject the batch.
- Quarantine invalid records.
- Send alerts.
- Accept with warnings when permitted.

**Step 5: Monitor Compliance**

Track violations, schema changes, and freshness metrics.

**Step 6: Manage Changes**

Communicate changes, test compatibility, and coordinate consumer updates.

---

## 9. Implementing Contract Checks in PySpark

A data contract can be implemented using validation logic in PySpark.

Suppose the contract requires:

- `order_id` must not be NULL.
- `customer_id` must not be NULL.
- `amount` must be non-negative.
- `status` must belong to an approved list.

### Example

```python
from pyspark.sql import functions as F

valid_statuses = [
    "Pending",
    "Shipped",
    "Delivered",
    "Cancelled"
]

# Required-field validation
required_valid = (
    F.col("order_id").isNotNull() &
    F.col("customer_id").isNotNull() &
    F.col("amount").isNotNull()
)

# Business-rule validation
business_valid = (
    (F.col("amount") >= 0) &
    F.col("status").isin(valid_statuses)
)

# Separate valid and invalid records
valid_df = df.filter(
    required_valid & business_valid
)

invalid_df = df.filter(
    ~(required_valid & business_valid)
)
```

### Duplicate-Key Validation

```python
duplicate_keys = (
    df.groupBy("order_id")
    .count()
    .filter(F.col("count") > 1)
    .select("order_id")
)
```

### Freshness Validation

For a dataset with an `updated_at` timestamp:

```python
latest_record = (
    df.agg(
        F.max("updated_at").alias("latest_timestamp")
    )
)

latest_record.show()
```

The pipeline can compare the latest available timestamp with the expected freshness threshold.

**Important:** These are illustrative checks. A production implementation should also handle missing columns, unexpected data types, NULL logic, duplicate-record policy, time zones, and empty datasets.

---

## 10. Data Contracts in a Medallion Architecture

Data contracts can apply across Bronze, Silver, and Gold layers.

### Bronze Layer

Focus:
- Source schema expectations.
- Ingestion format.
- Source metadata.
- Delivery frequency.
- Raw-data preservation.

### Silver Layer

Focus:
- Required-field validation.
- Data types.
- Uniqueness.
- Validity rules.
- Standardized business definitions.

### Gold Layer

Focus:
- Business-level definitions.
- Reporting metrics.
- Consumer-facing schemas.
- Freshness and availability requirements.

### Example

```text
Source Contract
      |
      v
Bronze
(Raw Source Data)
      |
      v
Silver Contract
(Cleaned and Validated Data)
      |
      v
Gold Contract
(Business-Ready Data)
      |
      v
Reports and Analytics
```

Different layers can have different contracts because they serve different consumers and purposes.

---

## 11. Data Contracts vs Data Quality Rules vs SLAs

These concepts are connected but serve different purposes.

### Data Contract

Defines the overall agreement about data structure, meaning, quality, ownership, and delivery.

### Data Quality Rules

Define checks used to evaluate whether data meets expected standards.

Examples:
- Customer ID cannot be NULL.
- Order ID must be unique.
- Amount cannot be negative.

### SLA (Service-Level Agreement)

Defines an agreed service expectation or target.

For data pipelines, this may include:
- Data available by a specified time.
- Maximum acceptable delay.
- Pipeline availability.
- Incident response expectations.

### Comparison

| Concept | Main Purpose | Example |
|---|---|---|
| Data contract | Overall producer-consumer agreement | Orders schema and ownership |
| Data quality rule | Validate data correctness | Amount must be non-negative |
| SLA | Define service expectations | Orders available by 9 AM |

A data contract may include data quality rules and SLA requirements.

---

## 12. Common Challenges and Best Practices

### Common Challenges

1. Different teams interpret fields differently.
2. Producers change schemas without notice.
3. Consumers depend on undocumented columns.
4. Business definitions are unclear.
5. Quality checks are not automated.
6. Multiple schema versions exist.
7. Ownership is not clearly assigned.
8. Contract violations are not monitored.

### Best Practices

**1. Define Contracts Early**

Agree on expectations before integrating data sources.

**2. Keep Contracts Versioned**

Maintain a clear history of schema and rule changes.

**3. Assign Ownership**

Identify the producer, consumer, and responsible data owner.

**4. Automate Validation**

Run contract checks as part of ingestion and transformation pipelines.

**5. Define Breaking Changes**

Document which changes require consumer updates.

**6. Communicate Changes**

Give consumers sufficient notice before production changes.

**7. Monitor Compliance**

Track schema violations, quality failures, and freshness.

**8. Document Business Meaning**

Explain what each important field represents.

**9. Protect Sensitive Data**

Define classification, access, masking, and retention expectations.

**10. Test Before Deployment**

Validate new contract versions against existing consumers.

---

## 13. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| A data contract is only a schema | It also covers quality, ownership, semantics, and delivery |
| A schema registry is the same as a data contract | A registry primarily manages schemas |
| Data contracts prevent all pipeline failures | They reduce unexpected failures but cannot eliminate every issue |
| Adding a column is always safe | Consumer behavior and contract rules matter |
| A producer owns every downstream quality issue | Producers and consumers have shared responsibilities |
| Successful ingestion proves contract compliance | Explicit validation is still required |
| Data quality rules and data contracts are identical | Quality rules are one component of a broader contract |
| Every schema change requires a new major version | Versioning policy depends on compatibility and organizational rules |
| Data contracts are only useful for streaming | They are useful in batch and streaming pipelines |

---

## 14. Interview Questions and Answers

### Q1. What is a data contract?

A data contract is an agreement between data producers and consumers that defines the expected schema, quality, business meaning, delivery requirements, ownership, and change-management rules.

### Q2. Why are data contracts important?

They reduce unexpected schema changes, improve data quality, establish ownership, and help prevent downstream pipeline failures.

### Q3. What are the main components of a data contract?

Schema definition, data quality rules, business definitions, freshness expectations, ownership, security requirements, and change-management policies.

### Q4. Who is responsible for a data contract?

The producer and consumer should agree on it. The producer is responsible for publishing data according to the agreement, while the consumer validates and uses it according to the defined expectations.

### Q5. What happens when a producer changes a column name?

The change may break downstream consumers. It should be communicated, assessed for compatibility, and deployed using an agreed migration or versioning strategy.

### Q6. What is the difference between a data contract and a schema registry?

A data contract defines the broader agreement between teams. A schema registry manages schemas and can support schema versioning and compatibility checks.

### Q7. How do you validate a data contract in PySpark?

I check required columns, data types, NULL values, duplicate business keys, allowed values, business rules, and data freshness.

### Q8. How are data contracts related to schema evolution?

Schema evolution handles structural changes. Data contracts define which changes are acceptable, how they are communicated, and how consumers should handle them.

### Q9. Can data contracts be used in batch pipelines?

Yes. They can define the expected schema, quality, and delivery requirements for both batch and streaming datasets.

### Q10. What is the difference between a data contract and an SLA?

A data contract defines the overall data agreement. An SLA defines measurable service expectations, such as data availability and delivery time.

### Q11. How do data contracts help in a medallion architecture?

They define expectations for incoming Bronze data, validated Silver data, and business-ready Gold datasets. Each layer can have a contract suited to its consumers.

### Q12. What would you do if incoming data violated a contract?

I would identify the violation, assess its severity, and apply the agreed policy: reject the batch, quarantine affected records, raise an alert, or accept with a warning when permitted. I would also notify the responsible owner.

---

## 15. Quick Revision

| Concept | Remember |
|---|---|
| Data contract | Agreement between producer and consumer |
| Producer | Creates and publishes data |
| Consumer | Uses published data |
| Schema definition | Defines columns and data types |
| Quality rules | Define acceptable data |
| Freshness | Defines expected data availability |
| Ownership | Defines responsibility |
| Change management | Controls schema and business changes |
| Schema registry | Stores and manages schemas |
| Schema evolution | Handles changes in data structure |
| SLA | Defines measurable service expectations |
| Contract validation | Checks incoming data against the agreement |

### One-Minute Interview Answer

A data contract is an agreement between a data producer and a consumer that defines how data should be structured, delivered, and maintained.

It typically includes the schema, data types, required fields, data quality rules, business definitions, freshness expectations, ownership, and change-management policies.

For example, an e-commerce application may provide order data to a data engineering pipeline. The contract can specify that `order_id` must be unique, `customer_id` cannot be NULL, `amount` must be non-negative, and data should arrive every hour.

I can implement these checks using PySpark and validate incoming data before loading it into Delta tables. If the contract is violated, I follow the agreed policy, such as rejecting the batch, quarantining invalid records, or raising an alert.

---

## 16. Self-Assessment

- [ ] Define a data contract.
- [ ] Explain why data contracts are needed.
- [ ] Identify the main components of a contract.
- [ ] Explain producer and consumer responsibilities.
- [ ] Understand schema and quality expectations.
- [ ] Explain freshness and delivery requirements.
- [ ] Differentiate data contracts and schema registries.
- [ ] Explain how contracts support schema evolution.
- [ ] Implement basic contract checks in PySpark.
- [ ] Explain contract validation in a medallion architecture.
- [ ] Differentiate data contracts, quality rules, and SLAs.
- [ ] Describe how to handle contract violations.

**Interview readiness:** Be able to explain this scenario: *An application team changes the `amount` column from DECIMAL to STRING without informing the data engineering team. How would a data contract help detect the issue, protect downstream pipelines, and coordinate the change?*