# Failure Recovery in Data Engineering

## 1. Introduction

**Failure recovery is the process of restoring a data pipeline or system to a correct working state after an unexpected failure.**

Data pipelines can fail because of application errors, network interruptions, unavailable storage, resource shortages, invalid records, or infrastructure problems.

A reliable pipeline should not only detect failures but also recover without losing required data, creating unintended duplicates, or producing inconsistent results.

### Simple Example

Suppose a daily pipeline performs these operations:

1. Reads order data from cloud storage.
2. Cleans and transforms the records.
3. Writes the results to a Delta table.
4. Updates the pipeline's processing status.

The Delta write succeeds, but the application crashes before updating the processing status.

When the pipeline runs again, it may process the same data.

If the pipeline uses an unsafe append operation, duplicate records may appear.

A recovery-safe design uses:
- Durable source data.
- Progress tracking.
- Idempotent or transactional writes.
- Appropriate retry mechanisms.
- Validation after recovery.

**Key idea:** Failure recovery means restoring correct processing, not simply restarting a failed job.

---

## 2. Why Is Failure Recovery Important?

Data pipelines often process business-critical information.

Without proper recovery mechanisms, failures can cause:

- Missing records.
- Duplicate records.
- Incomplete reports.
- Incorrect business calculations.
- Inconsistent tables.
- Delayed data delivery.
- Repeated manual intervention.
- Increased infrastructure costs.

### Example: E-Commerce Pipeline

Suppose an e-commerce company processes 10 million order events daily.

The pipeline fails after processing 6 million records.

The system needs to determine:
- How much data was successfully committed?
- Where did processing stop?
- Can the remaining records be processed?
- Could already processed records be duplicated?
- Are downstream tables consistent?

A good recovery design answers these questions through reliable progress tracking, safe writes, and validation.

---

## 3. Common Causes of Pipeline Failures

### 3.1 Application Failure

The application crashes because of:
- Programming errors.
- Invalid configuration.
- Missing dependencies.
- Incorrect SQL or PySpark logic.
- Unexpected input values.

**Recovery:** Identify and correct the cause, then rerun the affected work safely.

### 3.2 Data Failure

Input data contains:
- Missing required values.
- Invalid data types.
- Duplicate records.
- Corrupted files.
- Unexpected schema changes.

**Recovery:** Validate the input, quarantine invalid records where appropriate, and process corrected data.

### 3.3 Infrastructure Failure

Examples:
- Executor failure.
- Driver failure.
- Machine restart.
- Container termination.
- Cluster unavailability.

**Recovery:** Use the platform's supported restart and task-retry mechanisms. Streaming applications may also use checkpoints to recover progress and state.

### 3.4 Network Failure

Examples:
- Temporary connection loss.
- Request timeout.
- Service throttling.
- Unavailable remote storage.

**Recovery:** Retry transient operations with bounded retries and suitable backoff. An unknown request outcome must be handled carefully because the remote operation may already have succeeded.

### 3.5 Storage Failure

Examples:
- Cloud storage temporarily unavailable.
- Database connection failure.
- Insufficient storage capacity.
- Missing input files.

**Recovery:** Restore availability, verify data integrity, and rerun affected processing where possible.

### 3.6 Resource Failure

Examples:
- Out-of-memory errors.
- Disk spill or insufficient disk capacity.
- Executor resource exhaustion.
- Excessive concurrent jobs.

**Recovery:** Investigate resource usage, data skew, partition sizes, and workload configuration before restarting.

Repeatedly restarting without fixing the underlying resource problem may cause the same failure again.

---

## 4. Failure Detection

Before recovering from a failure, the system must detect and identify it.

### Common Detection Methods

**1. Job status**

Check whether the pipeline completed, failed, or was cancelled.

**2. Logs**

Inspect application and task logs for exceptions and error messages.

**3. Metrics**

Monitor:
- Processing duration.
- Input and output records.
- Failed tasks.
- Memory usage.
- CPU utilization.
- Shuffle activity.
- Storage and network errors.

**4. Data-quality checks**

A job can finish successfully while producing incorrect data.

Check:
- Null values.
- Duplicate business keys.
- Unexpected record counts.
- Invalid data types.
- Business-rule violations.

**5. Freshness checks**

Verify that data arrived within the expected time window.

For example, a daily sales table may be expected to contain data through the previous business day.

### Important Point

Job success and data correctness are different.

A pipeline should monitor both operational health and data quality.

---

## 5. Failure Recovery Strategies

### 5.1 Retry

Retry means attempting a failed operation again.

Useful for temporary problems such as:
- Network interruptions.
- Temporary service unavailability.
- Transient storage errors.
- Resource allocation issues that may recover.

Example:

```text
Operation
    |
    v
  Failed
    |
    v
 Wait and Retry
    |
    v
Success or Failure
```

Use:
- Bounded retries.
- Exponential backoff.
- Jitter when appropriate.
- Error classification.

Do not repeatedly retry permanent failures such as invalid SQL or missing permissions without correcting the cause.

### 5.2 Restart

Restart means starting a failed application, job, or service again.

A restart may be appropriate after:
- A temporary infrastructure problem.
- A corrected configuration.
- An application crash.

However, a restart alone does not guarantee that processing resumes from the exact previous position.

Recovery behavior depends on the application, progress tracking, and source and sink capabilities.

### 5.3 Checkpointing

Checkpointing stores progress or state needed for recovery.

In Spark Structured Streaming, checkpoints can store:
- Source progress.
- Query metadata.
- Stateful processing information.

Example:

```python
query = (
    events.writeStream
    .format("delta")
    .option(
        "checkpointLocation",
        "/checkpoints/orders"
    )
    .outputMode("append")
    .toTable("sales.orders")
)
```

When a compatible query is restarted with its checkpoint, Spark can use the saved information to recover according to the source and sink guarantees.

**Important:** Checkpointing does not automatically guarantee exactly-once effects for arbitrary external destinations.

### 5.4 Reprocessing

Reprocessing means processing previously available data again.

Useful when:
- Transformation logic was incorrect.
- Historical records were corrected.
- A business rule changed.
- A target table needs rebuilding.

Example:

If a revenue calculation was incorrect for October, the pipeline may need to recalculate October's results.

### 5.5 Backfill

Backfill means loading missing or unprocessed historical data.

Example:

If the October 10 pipeline failed and no records were loaded for that date, the pipeline can backfill October 10.

Backfill and reprocessing are related, but not identical:
- Backfill fills historical gaps.
- Reprocessing regenerates existing results.

### 5.6 Failover

Failover moves processing or service responsibility to another available resource when the primary resource becomes unavailable.

Examples:
- Switching to a healthy service instance.
- Using a standby database.
- Restarting a workload on another available worker.

Failover depends on platform support and system configuration. It does not automatically protect against every form of data loss.

### 5.7 Restore from Backup

Backups are used when data has been lost, corrupted, or accidentally deleted and cannot be recovered through normal retries or recomputation.

A backup and restore strategy should define:
- Backup frequency.
- Retention period.
- Recovery location.
- Recovery time objective.
- Recovery point objective.
- Restore testing.

---

## 6. Progress Tracking and Recovery

A pipeline needs to know what work has completed and what remains.

Progress can be tracked using:
- Checkpoints.
- Source offsets.
- Watermarks.
- Batch IDs.
- Execution status tables.
- Transaction logs.

### Example: Batch Progress Table

| batch_id | start_date | end_date | status |
|---|---|---|---|
| B001 | 2026-10-01 | 2026-10-02 | SUCCESS |
| B002 | 2026-10-02 | 2026-10-03 | SUCCESS |
| B003 | 2026-10-03 | 2026-10-04 | FAILED |

The pipeline can identify B003 as the failed batch.

A progress table is useful for orchestration and recovery, but it does not itself make the target write safe to repeat.

### Recommended Workflow

1. Identify the failed batch.
2. Inspect its logs and execution status.
3. Determine whether the target write partially or fully succeeded.
4. Correct the failure cause.
5. Rerun the batch using retry-safe logic.
6. Validate the output.
7. Mark the batch successful only after the required checks pass.

### Important Scenario

The target write may succeed while the progress-table update fails.

In this case, the batch may be processed again.

Use idempotent writes or a transaction design that appropriately coordinates the relevant operations.

---

## 7. Failure Recovery in Apache Spark

Spark provides several mechanisms that help recover from processing failures.

### 7.1 Task Retry

Spark can retry failed tasks according to the configured retry policy.

This is useful for temporary task failures.

A persistent programming error or repeated memory failure may still cause the stage or job to fail.

### 7.2 Lineage-Based Recovery

Spark tracks the transformations and dependencies used to produce distributed data.

If a partition is lost, Spark may recompute it from its lineage, provided the required inputs remain available.

### 7.3 Stage Re-Execution

If required shuffle data is lost, Spark may need to rerun upstream tasks to regenerate it.

The amount of work depends on the execution plan and which intermediate data remains available.

### 7.4 Executor Recovery

If an executor becomes unavailable, Spark can reschedule eligible work on available resources.

Cached data on the lost executor may need to be recomputed.

### 7.5 Driver Failure

Driver failure is more serious because the driver coordinates the Spark application.

Recovery depends on deployment mode and platform capabilities.

For long-running streaming queries, a compatible restart using the same checkpoint can help restore progress and state.

### Important Distinction

Spark task retry, lineage-based recomputation, and application restart are different recovery mechanisms. They address different failure scopes.

---

## 8. Failure Recovery in Spark Structured Streaming

Structured Streaming uses checkpoints to support recovery of streaming query progress and state.

### Conceptual Workflow

```text
Source Events
      |
      v
Spark Streaming Query
      |
      v
Process Micro-Batch
      |
      v
Write Output
      |
      v
Record Progress
      |
      v
Checkpoint
```

This is a conceptual view. Exact commit and recovery behavior depends on the source, sink, and query implementation.

### What Happens After a Failure?

1. The streaming query stops or becomes unavailable.
2. The failure is investigated.
3. The query is restarted with its compatible checkpoint.
4. Spark uses the checkpoint and supported source recovery mechanisms.
5. Processing resumes according to the source and sink guarantees.
6. Output correctness is verified.

### Important Best Practices

- Use a persistent checkpoint location.
- Use a separate checkpoint for each independent query.
- Keep the checkpoint location stable for recovery of the same query.
- Do not delete checkpoints casually.
- Make custom external writes retry-safe.
- Monitor query progress and processing delays.

A checkpoint supports recovery, but it does not automatically protect arbitrary external side effects from duplication.

---

## 9. Failure Recovery in Delta Lake

Delta Lake provides transaction-based table updates that help maintain consistent committed table states.

### Example

Suppose a pipeline writes transformed records into a Delta table.

```python
(
    transformed_df.write
    .format("delta")
    .mode("append")
    .saveAsTable("sales.orders")
)
```

Delta commits a successful table transaction so readers see a consistent committed version.

However, if the application crashes after the commit but before recording its batch status, the pipeline may retry the same data.

An append operation may create duplicate logical records.

### Safer Approaches

- Use MERGE for suitable upsert workloads.
- Use stable business keys.
- Deduplicate source data.
- Use transaction-aware batch identifiers where supported.
- Validate the target after recovery.

### Important Distinction

Delta Lake transactions protect the table commit. They do not automatically make an entire multi-system pipeline atomic.

For example, a workflow involving Delta, an external database, and an API needs separate coordination or idempotency strategies.

---

## 10. Failure Recovery and Exactly-Once Processing

Exactly-once processing is often discussed in distributed data systems.

However, it is important to understand its scope.

**Exactly-once effective results** means the final committed output reflects each intended operation once within the supported processing and transaction guarantees.

It does not necessarily mean that every physical computation or API call happens only once.

### Example

A streaming pipeline reads an event and writes it to a target.

If the application fails after the target write but before recording progress, the event may be processed again.

A retry-safe sink can prevent the repeated attempt from creating an additional logical result.

### Components That May Be Needed

- Durable source data.
- Progress tracking.
- Checkpointing.
- Transactional output.
- Idempotent operations.
- Deduplication where appropriate.

### Interview Point

Checkpointing alone does not guarantee exactly-once output for arbitrary sinks. End-to-end guarantees depend on the source, processing engine, and destination.

---

## 11. Recovery Objectives: RTO and RPO

Two important disaster-recovery concepts are Recovery Time Objective and Recovery Point Objective.

### 11.1 Recovery Time Objective (RTO)

RTO is the maximum acceptable time to restore a service after a disruption.

Example:

If the RTO is two hours, the system should be restored within two hours of the incident.

### 11.2 Recovery Point Objective (RPO)

RPO is the maximum acceptable amount of data loss measured in time.

Example:

If the RPO is 15 minutes, the business is willing to lose at most 15 minutes of data in the defined disaster scenario.

### Comparison

| Feature | RTO | RPO |
|---|---|---|
| Meaning | Target recovery time | Acceptable data-loss window |
| Focus | Service availability | Data recovery point |
| Example | Restore within 2 hours | Lose no more than 15 minutes of data |
| Common mechanisms | Failover, restart, recovery automation | Backups, replication, durable logs |

RTO and RPO are business recovery objectives. They should be defined for the system and disaster scenario, not assumed from the presence of a backup or checkpoint.

---

## 12. Designing a Failure-Resilient Data Pipeline

Suppose a company processes order events from Kafka, transforms them using Spark, and writes the results to a Delta table.

### Recommended Design

**Step 1: Durable input**

Configure Kafka replication, retention, and producer reliability appropriately.

**Step 2: Fault-tolerant processing**

Use Spark Structured Streaming with a persistent checkpoint location.

**Step 3: Data validation**

Validate required fields, data types, event identifiers, and business rules.

**Step 4: Retry-safe output**

Use a transactional or idempotent write strategy to prevent unintended duplicate effects.

**Step 5: Progress tracking**

Track completed batches or source progress using the supported recovery mechanism.

**Step 6: Monitoring**

Monitor job status, source lag, processing time, failures, and data quality.

**Step 7: Recovery testing**

Test executor failure, application restart, destination failure, and historical replay.

### Architecture

```text
Kafka
  |
  v
Spark Structured Streaming
  |
  +------> Checkpoint
  |
  v
Validation and Transformation
  |
  v
Idempotent / Transactional Write
  |
  v
Delta Table
  |
  v
Data Quality and Monitoring
```

The design should define how each component recovers and how partial failures are handled.

---

## 13. Practical Recovery Scenario

### Scenario

A daily order pipeline processes October 24 data.

The pipeline:
1. Reads source records.
2. Transforms the data.
3. Writes to the Silver Delta table.
4. Updates the batch status to `SUCCESS`.

The Delta write succeeds, but the application crashes before updating the status.

### What Can Go Wrong?

When the pipeline restarts, the batch status still appears as `FAILED` or `IN_PROGRESS`.

If the job blindly appends the same records, duplicate orders may appear.

### Recovery Approach

1. Identify the failed batch using its batch ID.
2. Check whether the target write committed.
3. Verify the target records for the affected batch.
4. Rerun using an idempotent MERGE or another safe write strategy.
5. Validate record counts and business totals.
6. Update the batch status to `SUCCESS`.

### Interview Answer

I would not assume that a failed job means the target write also failed. First, I would check whether the write was committed. Then I would rerun the batch using an idempotent write strategy so that repeated processing does not create duplicates. Finally, I would validate the output and update the execution status.

---

## 14. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Restarting a job always resumes from the exact failure point | Recovery depends on progress tracking and application design |
| A successful job means correct data | Data-quality and completeness checks are also required |
| Every failure should be retried | Permanent failures require correcting the cause |
| Checkpointing prevents every duplicate | Sink and external side-effect behavior also matter |
| Spark can always recompute lost data | Required inputs and dependencies must remain available |
| Delta transactions make the whole pipeline atomic | They protect supported table transactions, not every external system |
| A batch status table guarantees recovery | It tracks progress but does not automatically make writes idempotent |
| Backfill and restart are the same | Restart recovers an execution; backfill processes a historical range |
| Replication replaces backups | Replication and backup protect against different failure scenarios |
| RTO and RPO mean the same thing | RTO concerns recovery time; RPO concerns acceptable data loss |

---

## 15. Interview Questions and Answers

### Q1. What is failure recovery?

Failure recovery is the process of restoring a pipeline or system to a correct working state after a failure, while protecting required data and processing guarantees.

### Q2. What are common causes of pipeline failures?

Common causes include application bugs, invalid input data, network problems, storage unavailability, executor failures, driver crashes, and resource shortages.

### Q3. What is the difference between retry and restart?

Retry attempts a failed operation again. Restart starts a failed application or job again. A restart may involve recovery from saved progress if the application supports it.

### Q4. How does Spark recover from executor failure?

Spark can reschedule affected tasks and recompute lost partitions using lineage when required inputs remain available.

### Q5. How does Spark Structured Streaming recover after a failure?

A compatible restarted query can use its persistent checkpoint and supported source recovery mechanisms to restore progress and state.

### Q6. Does checkpointing guarantee exactly-once processing?

Not for every destination. Checkpointing supports recovery, but exactly-once effective results depend on the source, processing logic, and sink guarantees.

### Q7. What happens if a target write succeeds but the job crashes?

The target may already contain committed data even though the job reports failure. A retry may process the same data again, so the pipeline needs idempotent or transaction-aware write logic.

### Q8. How would you recover a failed batch pipeline?

I would identify the failed batch, inspect logs and target state, correct the underlying issue, rerun the batch safely, validate the output, and then mark it successful.

### Q9. What is the difference between backfill and failure recovery?

Failure recovery is the broader process of restoring correct operation after a failure. Backfill is one possible recovery activity used to load missing historical data.

### Q10. What is the difference between RTO and RPO?

RTO is the target time to restore a service. RPO is the maximum acceptable data-loss window.

### Q11. How do you prevent duplicate records during recovery?

I would use stable keys, deterministic transformations, source deduplication, and idempotent writes or suitable transactional mechanisms.

### Q12. What should be monitored during failure recovery?

I would monitor job status, retry counts, task failures, source lag, input and output record counts, processing duration, data freshness, and data-quality results.

### Q13. How would you test a pipeline's recovery process?

I would simulate supported failure scenarios, such as executor loss, application restart, temporary storage unavailability, and destination write failure. Then I would verify that processing resumes correctly without missing or duplicate results.

### Q14. What is the role of backups in failure recovery?

Backups provide a recovery source when data is lost or corrupted and cannot be reconstructed through normal processing or replication.

---

## 16. Quick Revision

| Concept | Remember |
|---|---|
| Failure recovery | Restore correct operation after a failure |
| Retry | Attempt a failed operation again |
| Restart | Start a failed application or job again |
| Checkpointing | Save progress or state for recovery |
| Lineage | Recompute lost derived data when possible |
| Replication | Maintain multiple copies |
| Backup | Preserve data for restoration |
| Reprocessing | Run existing data through processing again |
| Backfill | Load missing historical data |
| Idempotency | Make repeated operations safe |
| RTO | Target time to restore service |
| RPO | Maximum acceptable data-loss window |
| Monitoring | Detect operational and data-quality problems |

### One-Minute Interview Answer

Failure recovery is the process of restoring a data pipeline to a correct working state after an unexpected failure.

In Spark, recovery can involve task retries, lineage-based recomputation, and application restarts. For Structured Streaming, checkpoints help recover query progress and state. In Delta Lake, transactions help maintain consistent committed table versions.

For a reliable pipeline, I would combine durable input data, appropriate retries, checkpointing where needed, idempotent or transactional writes, progress tracking, and data-quality validation. I would also test failure scenarios and define recovery objectives such as RTO and RPO. Importantly, restarting a job does not automatically guarantee that the output is complete or free of duplicates.

---

## 17. Self-Assessment

- [ ] Define failure recovery.
- [ ] Identify common pipeline failure types.
- [ ] Explain how to detect failures using logs and metrics.
- [ ] Compare retry and restart.
- [ ] Explain checkpointing and lineage-based recovery.
- [ ] Describe Structured Streaming recovery.
- [ ] Explain the role of Delta transactions.
- [ ] Handle a crash after a successful target write.
- [ ] Explain RTO and RPO.
- [ ] Design a recovery-safe batch pipeline.
- [ ] Design a recovery-safe streaming pipeline.
- [ ] Explain how to validate data after recovery.

**Interview readiness:** Be able to answer this scenario: *Your pipeline failed after writing data to the target but before updating its batch status. How would you recover it without losing records or creating duplicates?*