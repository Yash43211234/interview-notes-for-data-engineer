# Data Engineering System Design – Interview Questions

## 1. What Is System Design?

System design is the process of defining the architecture, components, data flow, technologies, and interactions required to build a system that meets business and technical requirements.

For Data Engineers, system design focuses on building reliable, scalable, secure, and maintainable data platforms.

A typical data system includes:

```text
Data Sources
     |
     v
Ingestion
     |
     v
Message Broker / Storage
     |
     v
Processing Engine
     |
     v
Data Warehouse / Lakehouse
     |
     v
Analytics / Reporting
```

A Data Engineer should be able to explain:
- Where data comes from.
- How data is ingested.
- Where data is stored.
- How data is processed.
- How data quality is maintained.
- How failures are recovered.
- How the system scales.
- How access is controlled.
- How performance and cost are managed.

---

## 2. How to Approach a System Design Interview

Do not start by selecting technologies.

First understand the problem and its requirements.

### Step 1: Clarify Functional Requirements

Functional requirements describe what the system must do.

Examples:
- Ingest customer and order data.
- Process transaction events.
- Store historical records.
- Generate daily reports.
- Support customer-level analytics.

Ask:
- What are the data sources?
- What data needs to be processed?
- Who consumes the data?
- What outputs are expected?
- Are historical records required?

### Step 2: Clarify Non-Functional Requirements

Non-functional requirements describe how well the system should operate.

Important requirements:

| Requirement | Meaning |
|---|---|
| Scalability | Handles increasing workload |
| Availability | Remains accessible |
| Reliability | Produces dependable results |
| Latency | Time taken to process or return data |
| Throughput | Amount of data processed per unit of time |
| Durability | Protects committed data from loss |
| Security | Protects data and access |
| Maintainability | Can be operated and changed easily |
| Cost | Stays within the budget |

Ask:
- How much data arrives daily?
- How quickly must data be available?
- What is the expected growth?
- What happens if a component fails?
- What are the recovery requirements?

### Step 3: Estimate Data Volume

Estimate:
- Events or records per second.
- Daily data volume.
- Average record size.
- Storage growth.
- Peak traffic.
- Retention period.

Example:

Assume:
- 1 million records per day.
- Average record size = 1 KB.

Approximate raw daily data:

\[
1,000,000 \times 1\text{ KB} \approx 1\text{ GB}
\]

Approximate raw yearly data:

\[
1\text{ GB} \times 365 \approx 365\text{ GB}
\]

This is a simplified estimate. Actual storage depends on file formats, compression, metadata, replication, and retained copies.

### Step 4: Draw the High-Level Architecture

Identify:
1. Sources.
2. Ingestion mechanism.
3. Raw storage.
4. Processing engine.
5. Curated storage.
6. Serving layer.
7. Orchestration.
8. Monitoring and governance.

### Step 5: Explain Important Design Decisions

For each component, explain why you selected it.

Examples:
- Batch or streaming?
- ADF or event-driven ingestion?
- ADLS Gen2 or a warehouse?
- Spark or SQL transformations?
- Full load, incremental load, or CDC?
- Partitioning or clustering?
- Managed identity or service principal?

### Step 6: Discuss Reliability and Failure Recovery

Explain:
- Retries.
- Idempotency.
- Checkpointing.
- Watermarks.
- Backfills.
- Dead-letter handling.
- Data reconciliation.
- Disaster recovery.

### Step 7: Discuss Security, Monitoring, and Cost

Explain:
- Authentication and authorization.
- Encryption and network access.
- Data quality.
- Freshness monitoring.
- Pipeline alerts.
- Compute optimization.
- Storage retention.

---

## 3. Functional and Non-Functional Requirements

### Functional Requirements

Examples for an e-commerce platform:
- Ingest orders and payments.
- Maintain customer and product data.
- Calculate daily revenue.
- Track order status.
- Provide historical reports.

### Non-Functional Requirements

Examples:
- Daily reports available by 7 AM.
- Near-real-time order visibility.
- Support increasing data volume.
- Recover from pipeline failures.
- Prevent duplicate financial transactions.
- Restrict access to sensitive information.

### Interview Tip

Translate vague requirements into measurable targets.

Instead of saying:

"The pipeline should be fast."

Say:

"The Gold sales dataset should be refreshed within 30 minutes of the agreed source-data availability time."

---

## 4. Batch vs Streaming Architecture

### Batch Architecture

```text
Source Database
      |
      v
Scheduled Extraction
      |
      v
Raw Data Lake
      |
      v
Batch Processing
      |
      v
Data Warehouse
      |
      v
Reporting
```

Use batch processing when:
- Data can arrive periodically.
- Hourly or daily freshness is acceptable.
- Historical processing is important.
- Simpler operations are preferred.

### Streaming Architecture

```text
Applications
      |
      v
Kafka / Event Hubs
      |
      v
Streaming Processor
      |
      v
Streaming Storage
      |
      v
Real-Time Analytics
```

Use streaming when:
- Events must be processed continuously.
- Low latency is important.
- Event-driven actions are required.
- Continuous aggregation or monitoring is needed.

### Batch vs Streaming Comparison

| Batch | Streaming |
|---|---|
| Processes bounded data | Processes continuously arriving events |
| Scheduled or triggered | Continuously running or periodically triggered |
| Often higher latency | Can provide lower latency |
| Easier historical replay of bounded input | Requires event retention and progress tracking |
| Suitable for daily reporting | Suitable for event-driven analytics |

**Important:** Streaming may use micro-batch execution. A streaming requirement does not always require event-by-event processing.

---

## 5. Data Storage Architecture

### Data Lake

A data lake stores large amounts of raw and processed data in its original or transformed formats.

Common storage:
- Azure Data Lake Storage Gen2.
- Amazon S3.
- Google Cloud Storage.

Advantages:
- Flexible data storage.
- Supports multiple data formats.
- Suitable for large-scale processing.
- Preserves historical source data.

### Data Warehouse

A data warehouse stores structured, curated data optimized for analytical queries and reporting.

Common examples:
- Azure Synapse Analytics.
- Snowflake.
- Amazon Redshift.
- BigQuery.

### Lakehouse

A lakehouse combines data-lake storage with table-management and analytical capabilities.

A typical lakehouse uses:
- Object storage.
- Parquet data files.
- Transactional table formats such as Delta Lake.
- Metadata and governance.
- Distributed processing engines.

### Data Lake vs Data Warehouse vs Lakehouse

| Feature | Data Lake | Data Warehouse | Lakehouse |
|---|---|---|---|
| Data | Raw and processed | Curated and structured | Raw and curated |
| Storage | Object storage | Warehouse-managed storage | Object storage with table layer |
| Schema | Often applied during processing | Usually enforced by the warehouse | Managed through table formats |
| Workloads | Data engineering, ML, exploration | BI and SQL analytics | Engineering and analytics |
| Transaction support | Depends on storage/table layer | Supported by warehouse | Supported by table format and platform |

---

## 6. Medallion Architecture

Medallion architecture organizes data into progressive quality layers.

### Bronze Layer

Stores raw or minimally processed source data.

Responsibilities:
- Preserve original data.
- Record ingestion metadata.
- Support reprocessing.
- Track source information.

### Silver Layer

Stores cleaned and standardized data.

Responsibilities:
- Handle duplicates.
- Validate schemas.
- Standardize data types.
- Apply CDC.
- Resolve data-quality issues.

### Gold Layer

Stores business-ready datasets.

Responsibilities:
- Build fact and dimension tables.
- Calculate business metrics.
- Create aggregated datasets.
- Support reporting and analytics.

```text
Source Systems
      |
      v
   Bronze
  Raw Data
      |
      v
   Silver
Clean Data
      |
      v
    Gold
Business Data
      |
      v
BI / Analytics
```

**Important:** Bronze, Silver, and Gold are logical data layers. Their exact implementation and transformation rules depend on the platform and business requirements.

---

## 7. Data Ingestion Design

### What Is Data Ingestion?

Data ingestion is the process of bringing data from source systems into a data platform.

Common sources:
- Databases.
- APIs.
- Files.
- Application events.
- Message brokers.
- SaaS platforms.

### Full Load

Reads the complete source dataset.

Useful for:
- Initial loads.
- Small reference tables.
- Controlled historical rebuilds.

### Incremental Load

Reads new or changed records.

Common approaches:
- Timestamp-based extraction.
- Increasing ID.
- CDC.
- Source change logs.

### CDC-Based Ingestion

Captures inserts, updates, and deletes from a source system.

Useful when:
- Tables are large.
- Changes are frequent.
- Deletes must be captured.
- Source transaction logs are available.

### What Should a Good Ingestion Framework Support?

- Configurable source and target.
- Authentication and authorization.
- Schema validation.
- Full and incremental loading.
- Retry and error handling.
- Idempotent writes.
- Watermark or offset management.
- Logging and monitoring.
- Reprocessing.

---

## 8. Processing Engine Design

### What Is a Distributed Processing Engine?

A distributed processing engine processes data across multiple compute resources.

Examples:
- Apache Spark.
- Apache Flink.
- Distributed SQL engines.

### Why Use Apache Spark?

Spark is useful when:
- Data volume exceeds convenient single-machine processing.
- Large joins and aggregations are required.
- Parallel processing is needed.
- Batch or streaming workloads must be handled.

### Important Spark Design Considerations

- Partition count.
- Data skew.
- Shuffle volume.
- Join strategy.
- Executor and driver memory.
- File sizes.
- Caching.
- Query execution plan.
- Adaptive Query Execution.

**Interview point:** Distributed processing introduces coordination, network transfer, and shuffle costs. A distributed engine is not automatically faster for every workload.

---

## 9. Data Modeling in System Design

### What Is Dimensional Modeling?

Dimensional modeling organizes analytical data into fact and dimension tables.

**Fact table:** Stores measurable business events or observations.

**Dimension table:** Stores descriptive attributes used to analyze facts.

Example:

```text
             DimCustomer
                  |
                  |
DimProduct -- FactSales -- DimDate
                  |
                  |
              DimStore
```

### What Is Grain?

Grain defines exactly what one row in a fact table represents.

Example:

One row per order item.

| order_id | product_id | quantity | amount |
|---|---|---:|---:|
| 1001 | P01 | 2 | 500 |
| 1001 | P02 | 1 | 300 |

This table has order-item grain, not order grain.

### Why Is Grain Important?

It determines:
- What one row means.
- Which measures are valid.
- Which dimensions can be joined.
- How aggregation should work.
- Whether joins can cause duplicate totals.

### What Is SCD Type 2?

Slowly Changing Dimension Type 2 preserves historical changes by creating a new dimension version.

Example:

| customer_key | customer_id | city | effective_from | effective_to |
|---|---|---|---|---|
| 101 | C01 | Kolkata | Jan 1 | Mar 31 |
| 205 | C01 | Bengaluru | Apr 1 | Current |

Facts should reference the correct dimension version for the event time.

A common interval rule is:

\[
effective\_from \leq event\_time < effective\_to
\]

---

## 10. Scalability and Performance

### What Is Scalability?

Scalability is the ability of a system to handle increasing workload by increasing its resources or improving its architecture.

### Vertical Scaling

Increasing the capacity of one machine.

Example:
- More CPU.
- More RAM.
- Faster storage.

Advantages:
- Often simpler to implement.
- Less distributed coordination.

Disadvantages:
- Hardware limits.
- Can become expensive.
- May leave a single point of failure.

### Horizontal Scaling

Adding more machines or workers.

Advantages:
- Can handle larger workloads.
- Supports distributed processing.
- Can improve fault isolation.

Disadvantages:
- Requires coordination.
- Introduces network and shuffle costs.
- Can increase operational complexity.

### How Would You Optimize a Large Data Pipeline?

1. Identify the slowest stage.
2. Check source and target I/O.
3. Filter unnecessary records early.
4. Select only required columns.
5. Optimize joins.
6. Handle skew.
7. Reduce unnecessary shuffles.
8. Review partitioning.
9. Compact small files where beneficial.
10. Tune compute based on measured bottlenecks.

---

## 11. Reliability and Fault Tolerance

### What Is Reliability?

Reliability is the ability of a system to operate correctly and consistently over time.

### What Is Fault Tolerance?

Fault tolerance is the ability to continue operating when components fail.

### What Is Idempotency?

Idempotency means repeated execution produces the same intended final result under defined conditions.

It is important because production pipelines can fail after partially writing data.

Common techniques:
- MERGE using business keys.
- Deterministic deduplication.
- Controlled partition replacement.
- Transactional writes.
- Stable event IDs.

### What Is Checkpointing?

Checkpointing stores processing progress and, for stateful streaming workloads, recovery state.

It helps a streaming job resume after failures.

However, checkpointing alone does not guarantee exactly-once results for every external sink.

### What Is Disaster Recovery?

Disaster recovery is the process of restoring systems and data after a major failure.

It may involve:
- Backups.
- Geo-replication.
- Recovery procedures.
- Alternate compute environments.
- Data restoration.
- Failover.

Important measures:
- RPO: Maximum acceptable data loss measured in time.
- RTO: Maximum acceptable recovery time.

---

## 12. Security and Governance

A production data platform should include:

### Authentication
Verifies the identity of a user, application, or service.

### Authorization
Determines which resources or data an identity can access.

### Encryption
Protects data:
- At rest.
- In transit.

### Secret Management
Use secure secret-management systems such as Azure Key Vault instead of hardcoding credentials.

### Data Governance
Defines:
- Data ownership.
- Data classification.
- Data lineage.
- Access policies.
- Retention.
- Audit requirements.

### Data Privacy

Sensitive information should be protected through:
- Least-privilege access.
- Masking or tokenization where appropriate.
- Encryption.
- Audit logging.
- Retention policies.

---

## 13. Monitoring and Observability

A production data platform needs monitoring across multiple layers.

| Layer | What to Monitor |
|---|---|
| Ingestion | Records received, ingestion failures, source lag |
| Storage | Capacity, file count, access failures |
| Processing | Duration, task failures, shuffle, spill |
| Orchestration | Pipeline status, retries, dependencies |
| Data Quality | Nulls, duplicates, invalid records |
| Serving | Query latency, report freshness |
| Business | Revenue totals, order counts, KPI correctness |

### What Is Data Freshness?

Data freshness measures how recently the data has been updated and made available to consumers.

### What Is Data Lineage?

Data lineage tracks where data originated and how it moves through the platform.

### What Is Observability?

Observability is the ability to understand system behavior through:
- Logs.
- Metrics.
- Traces.
- Data-quality measurements.

---

# 14. Important System Design Interview Questions

## Q1. Design an E-Commerce Data Platform

**Requirements:**
- Ingest customers, products, orders, and payments.
- Support daily reporting.
- Provide near-real-time order visibility.
- Maintain historical data.
- Support business analytics.

**Architecture:**

```text
        E-Commerce Applications
          /       |       \
         v        v        v
      Orders   Payments  Customers
         \        |        /
          v       v       v
        Event Hubs / ADF
                |
                v
          ADLS Gen2
           Bronze
                |
                v
          Databricks
                |
                v
            Silver
                |
                v
       Facts and Dimensions
                |
                v
             Gold
                |
                v
          Power BI / SQL
```

**Design considerations:**
- Use batch or CDC for database tables.
- Use event ingestion for order events.
- Preserve raw data in Bronze.
- Deduplicate events using event IDs.
- Apply CDC to customer and product tables.
- Separate order, payment, and refund facts.
- Define revenue calculations clearly.
- Monitor late-arriving events.
- Use idempotent writes.

**Important:** An order event and a payment event are different business events. Do not treat them as interchangeable records.

## Q2. Design a Real-Time Clickstream Analytics Platform

**Requirements:**
- Capture user activity.
- Process page views and clicks.
- Track sessions.
- Calculate active users.
- Analyze conversion funnels.

**Architecture:**

```text
Web / Mobile Applications
          |
          v
     Event Hubs / Kafka
          |
          v
   Spark Structured Streaming
          |
          v
      ADLS Bronze
          |
          v
   Validation and Deduplication
          |
          v
      ADLS Silver
          |
          v
 Sessionization / Aggregation
          |
          v
       Gold Tables
          |
          v
    BI / Analytics
```

**Design considerations:**
- Include event IDs.
- Preserve event timestamps.
- Handle late-arriving events.
- Deduplicate events.
- Define session timeout rules.
- Use watermarks for bounded state where appropriate.
- Track source lag and processing delay.

## Q3. Design a Daily Sales Reporting Platform

**Requirements:**
- Ingest sales data from multiple databases.
- Generate daily revenue and order reports.
- Support historical analysis.
- Refresh reports by 7 AM.

**Architecture:**

```text
Multiple Databases
        |
        v
   Azure Data Factory
        |
        v
    ADLS Bronze
        |
        v
   Databricks / SQL
        |
        v
    ADLS Silver
        |
        v
 FactSales + Dimensions
        |
        v
     Gold Layer
        |
        v
      Power BI
```

**Design considerations:**
- Use incremental extraction.
- Maintain watermarks.
- Handle late updates and deletes.
- Validate source-to-target counts and values.
- Define the grain of sales facts.
- Handle refunds and cancellations.
- Monitor report freshness.

## Q4. Design a Customer 360 Platform

**Requirements:**
- Combine customer data from multiple systems.
- Maintain customer history.
- Provide a unified customer view.
- Support analytics.

**Architecture:**

```text
CRM       Website       Orders
 |            |            |
 v            v            v
          Ingestion
              |
              v
          Bronze
              |
              v
       Standardization
              |
              v
          Silver
              |
              v
     Identity Resolution
              |
              v
     Customer Dimensions
              |
              v
            Gold
              |
              v
       Customer Analytics
```

**Design considerations:**
- Identify matching customer records.
- Define a trusted customer key.
- Handle conflicting attributes.
- Preserve historical changes with SCD Type 2 where needed.
- Apply privacy and access controls.
- Track source lineage.

## Q5. Design a Large-Scale Reporting Platform

**Requirements:**
- Process billions of historical records.
- Support concurrent reporting users.
- Refresh datasets on a defined schedule.
- Maintain consistent business metrics.

**Design considerations:**
- Use distributed storage and processing.
- Separate ingestion from serving.
- Use columnar formats.
- Model facts and dimensions at clear grains.
- Pre-aggregate frequently used metrics where justified.
- Optimize file layout and query patterns.
- Monitor concurrency and query latency.
- Define a consistent semantic layer.
- Support historical backfills.

## Q6. Design an Incremental Data Ingestion Framework

**Requirements:**
- Support many source tables.
- Handle full and incremental loads.
- Maintain watermarks.
- Support retries and recovery.

**Architecture:**

```text
Configuration Table
        |
        v
  Orchestration Engine
        |
        v
    Read Metadata
        |
        v
   Select Load Type
      /       \
     v         v
 Full Load  Incremental
      \       /
       v     v
     Raw Storage
          |
          v
     Validation
          |
          v
      Target Write
          |
          v
    Update Watermark
```

**Design considerations:**
- Store source and target metadata.
- Define primary or business keys.
- Use CDC where available.
- Advance watermarks only after successful writes.
- Make retries idempotent.
- Support schema changes.
- Maintain per-table execution status.

## Q7. Design a Data Pipeline That Handles 1 TB of Data Daily

**Approach:**

1. Understand the source and data format.
2. Determine whether the 1 TB is raw or compressed.
3. Check whether all data must be processed daily.
4. Identify incremental and CDC opportunities.
5. Estimate ingestion throughput.
6. Choose scalable storage.
7. Use distributed processing if justified.
8. Select partitioning based on query patterns.
9. Optimize joins and transformations.
10. Monitor throughput, cost, and SLA.

Do not choose compute size based only on the daily data volume. File sizes, transformation complexity, skew, compression, concurrency, and latency requirements also matter.

## Q8. Design a Reliable Streaming Pipeline

**Requirements:**
- Process events continuously.
- Recover from failures.
- Avoid incorrect duplicate results.
- Handle late events.

**Architecture:**

```text
Event Producers
       |
       v
 Kafka / Event Hubs
       |
       v
Streaming Processor
       |
       v
Checkpoint + State
       |
       v
Idempotent Sink
       |
       v
Curated Tables
```

**Design considerations:**
- Stable event identifiers.
- Source offset management.
- Checkpoint storage.
- Event-time processing.
- Watermarks.
- Stateful recovery.
- Idempotent sink operations.
- Dead-letter handling.
- Monitoring and reconciliation.

---

## 15. Common System Design Trade-Offs

| Decision | Trade-Off |
|---|---|
| Batch vs streaming | Simplicity and cost vs lower latency |
| SQL vs Spark | Simpler relational processing vs distributed transformations |
| ETL vs ELT | Pre-load transformation control vs target-platform flexibility |
| Data lake vs warehouse | Flexible storage vs managed analytical serving |
| Full load vs incremental | Simplicity vs efficiency |
| CDC vs timestamps | More complete change capture vs implementation complexity |
| Strong consistency vs eventual consistency | Immediate consistency vs availability and performance trade-offs |
| Vertical vs horizontal scaling | Simpler infrastructure vs distributed complexity |
| Partitioning vs no partitioning | Data skipping vs file and metadata overhead |
| Strict validation vs quarantine | Data correctness enforcement vs pipeline continuity |
| Pre-aggregation vs raw querying | Faster reports vs extra storage and maintenance |

A strong design explains why a trade-off is acceptable for the stated requirements.

---

## 16. Common System Design Mistakes

1. Selecting technologies before understanding requirements.
2. Ignoring expected data volume.
3. Confusing latency with throughput.
4. Choosing streaming when batch is sufficient.
5. Ignoring late-arriving records.
6. Failing to define fact-table grain.
7. Joining facts at incompatible grains.
8. Assuming checkpoints guarantee exactly-once processing.
9. Ignoring idempotency.
10. Forgetting delete handling in incremental loads.
11. Ignoring data-quality validation.
12. Overlooking security and governance.
13. Increasing compute without investigating bottlenecks.
14. Ignoring recovery and backfill requirements.
15. Failing to discuss cost and operational complexity.

---

## 17. How to Present Your System Design Answer

Use this structure during an interview:

**1. Requirements**

"I would first clarify the business use case, source systems, expected volume, freshness, and consumers."

**2. Scale Estimation**

"I would estimate daily volume, peak ingestion rate, retention, and expected growth."

**3. Architecture**

"I would explain the source, ingestion, storage, processing, and serving components."

**4. Data Modeling**

"I would define the grain, facts, dimensions, keys, and historical data requirements."

**5. Reliability**

"I would explain idempotency, retries, checkpointing, watermarks, and recovery."

**6. Performance**

"I would discuss partitioning, file formats, distributed processing, and query optimization."

**7. Security**

"I would cover identity, permissions, encryption, and sensitive-data handling."

**8. Monitoring**

"I would define pipeline health, data quality, freshness, and business-level metrics."

**9. Trade-Offs**

"I would explain why the selected architecture fits the requirements and what limitations it introduces."

---

## 18. Quick Revision

| Concept | Remember |
|---|---|
| System Design | Architecture based on requirements |
| Functional Requirement | What the system must do |
| Non-Functional Requirement | How well it must operate |
| Latency | Time taken to respond or process |
| Throughput | Amount processed per unit time |
| Scalability | Ability to handle increasing workload |
| Availability | System accessibility |
| Reliability | Consistent correct operation |
| Fault Tolerance | Continue operating despite failures |
| Batch | Bounded data processing |
| Streaming | Continuous event processing |
| ETL | Transform before loading |
| ELT | Transform after loading |
| CDC | Captures inserts, updates, and deletes |
| Data Lake | Flexible data storage |
| Data Warehouse | Curated analytical storage |
| Lakehouse | Lake storage with table and analytical capabilities |
| Bronze | Raw data |
| Silver | Cleaned data |
| Gold | Business-ready data |
| Grain | Meaning of one fact row |
| SCD Type 2 | Preserves dimension history |
| Idempotency | Safe repeated execution |
| Checkpointing | Saves processing progress and recovery state |
| RPO | Maximum acceptable data loss measured in time |
| RTO | Maximum acceptable recovery time |
| Data Lineage | Tracks data origin and movement |
| Observability | Understands behavior using telemetry |

---

## 19. Self-Assessment Checklist

### Fundamentals
- [ ] Explain functional and non-functional requirements.
- [ ] Explain latency, throughput, and scalability.
- [ ] Estimate data volume and storage.
- [ ] Explain batch and streaming architecture.
- [ ] Explain ETL and ELT.

### Data Architecture
- [ ] Design a data lake and lakehouse.
- [ ] Explain Bronze, Silver, and Gold.
- [ ] Design an ingestion framework.
- [ ] Explain CDC and incremental loads.
- [ ] Explain data modeling and grain.

### Performance
- [ ] Explain vertical and horizontal scaling.
- [ ] Explain partitioning.
- [ ] Explain distributed processing.
- [ ] Explain Spark optimization.
- [ ] Discuss storage and query performance.

### Reliability
- [ ] Explain idempotency.
- [ ] Explain checkpointing.
- [ ] Explain retry and recovery.
- [ ] Explain RPO and RTO.
- [ ] Design a backfill strategy.

### Production Design
- [ ] Explain security and governance.
- [ ] Explain monitoring and observability.
- [ ] Design an e-commerce data platform.
- [ ] Design a clickstream platform.
- [ ] Design a daily reporting platform.
- [ ] Explain cost and architecture trade-offs.

**Preparation priority:** Start with the end-to-end data platform design, incremental ingestion framework, e-commerce platform, and daily reporting platform. For each case, practice drawing the architecture and explaining requirements, data flow, storage, transformations, data modeling, reliability, monitoring, security, and trade-offs without relying on memorized technology names.