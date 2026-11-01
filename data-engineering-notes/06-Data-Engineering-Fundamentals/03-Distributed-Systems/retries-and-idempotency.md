# Retries and Idempotency in Data Engineering

## 1. Introduction

Retries and idempotency are two important techniques for building reliable data pipelines.

- **Retry** means attempting an operation again after it fails.
- **Idempotency** means repeating an operation under defined conditions produces the same final effect as performing it once.

Retries help a system recover from temporary failures. Idempotency helps prevent repeated attempts from creating duplicate or incorrect results.

### Simple Example

Suppose a pipeline reads an order from Kafka and writes it to a Delta table.

The write succeeds, but the application fails before recording that the operation completed. When the pipeline retries, it may write the same order again.

A retry alone does not prevent duplication. An idempotent write strategy can ensure that the order is represented only once in the target table.

**Key idea:** Retries improve recovery; idempotency makes repeated operations safe.

---

## 2. Why Are Retries Needed?

Distributed systems can experience temporary failures because of:

- Network interruptions.
- Temporary storage unavailability.
- Service throttling.
- Executor or worker failures.
- Connection timeouts.
- Temporary resource shortages.

Some failures disappear when the operation is attempted again.

### Example

A pipeline writes data to cloud storage, but the request fails because of a temporary network interruption.

The pipeline retries the write and succeeds.

Without retries, a temporary failure might stop the entire pipeline unnecessarily.

However, retries are not a solution for every error. Invalid SQL, incompatible schemas, missing permissions, and deterministic code bugs usually require correcting the underlying cause.

---

## 3. Types of Retry Strategies

### 3.1 Immediate Retry

The system retries immediately after a failure.

```text
Operation
    |
    v
 Failed
    |
    v
Retry Immediately
    |
    v
Success or Failure
```

This can work for very short-lived failures, but repeated requests may worsen an overloaded service.

### 3.2 Fixed-Delay Retry

The system waits for a fixed interval before retrying.

Example:

```text
Attempt 1 → Failed
Wait 5 seconds
Attempt 2 → Failed
Wait 5 seconds
Attempt 3 → Success
```

A fixed delay is simple to implement, but many clients retrying at the same interval can create additional load.

### 3.3 Exponential Backoff

The waiting time increases after each failed attempt.

For example:

```text
Attempt 1 → Failed → Wait 1 second
Attempt 2 → Failed → Wait 2 seconds
Attempt 3 → Failed → Wait 4 seconds
Attempt 4 → Failed → Wait 8 seconds
```

A simplified formula is:

\[
d_n = \min(d_{\max}, d_0 \times 2^{n-1})
\]

Where:
- \(d_n\) = delay before retry \(n\)
- \(d_0\) = initial delay
- \(d_{\max}\) = maximum delay

Actual retry policies may use a different formula.

### 3.4 Exponential Backoff with Jitter

Jitter adds randomness to retry delays so that multiple clients do not all retry simultaneously.

This is especially useful when many jobs depend on the same service.

### Best Practices

- Set a maximum number of attempts or a clear retry deadline.
- Use backoff for transient failures.
- Respect service throttling guidance, such as `Retry-After`, where applicable.
- Avoid retrying permanent failures without changing the cause.
- Make repeated operations safe before adding automatic retries.

---

## 4. When Should an Operation Be Retried?

Not all failures should be treated the same way.

| Failure | Retry? | Reason |
|---|---|---|
| Temporary network timeout | Often | The connection may recover |
| Temporary service throttling | Often, with backoff | The service may accept requests later |
| Temporary storage unavailability | Often | Availability may recover |
| Invalid SQL syntax | Usually not | The query must be corrected |
| Missing permissions | Usually not | Access must be fixed |
| Invalid schema | Usually not without correction | The input or schema must be addressed |
| Unknown write outcome | Carefully | The write may already have succeeded |
| Duplicate business record | Not simply by retrying | Deduplication or idempotent logic is needed |

**Important:** A timeout does not necessarily mean the operation failed. The server may have completed the operation even though the client did not receive the response.

This uncertainty is one of the main reasons idempotency matters.

---

## 5. What Is Idempotency?

An operation is idempotent when repeating it under the same defined conditions has the same final effect as performing it once.

In simplified mathematical notation:

\[
f(f(x)) = f(x)
\]

This equation illustrates the idea of applying an idempotent operation twice. For data pipelines, the important question is whether repeated execution produces the intended final data state.

### Example: Idempotent Operation

Suppose an order table contains:

| order_id | amount |
|---|---:|
| 101 | 500 |

A pipeline receives order `101` with amount `500`.

If it uses a suitable upsert keyed by `order_id`, retrying the same operation leaves one logical order record with amount `500`.

### Example: Non-Idempotent Operation

Suppose a pipeline appends the same order on every attempt.

First attempt:

| order_id | amount |
|---|---:|
| 101 | 500 |

After a retry:

| order_id | amount |
|---|---:|
| 101 | 500 |
| 101 | 500 |

The retry created a duplicate.

### Important Qualification

Idempotency depends on the operation and its assumptions. A write keyed by `order_id` may be idempotent for updates to the current order state, but it may not be correct if each incoming event represents a separate legitimate transaction.

Choose a stable key and define the intended business behavior first.

---

## 6. Common Ways to Make Data Pipelines Idempotent

### 6.1 Use a Stable Business Key

Identify records using a stable key, such as:
- `order_id`
- `customer_id`
- `transaction_id`
- `event_id`

The key should uniquely identify the entity or event for the intended operation.

Do not assume a column is unique unless the source and business rules support that assumption.

### 6.2 Use Upsert or MERGE

An upsert inserts a new record or updates an existing record.

Example using Delta Lake SQL:

```sql
MERGE INTO target_orders AS target
USING source_orders AS source
ON target.order_id = source.order_id

WHEN MATCHED THEN
  UPDATE SET
    target.amount = source.amount,
    target.status = source.status

WHEN NOT MATCHED THEN
  INSERT (order_id, amount, status)
  VALUES (source.order_id, source.amount, source.status);
```

If the same source state is applied again, the target can remain unchanged after the first successful application.

However:
- The merge key must match the intended business identity.
- The source should be deduplicated when multiple source rows can match the same target row.
- If multiple events for one key have different values, define which event wins.
- Concurrent writes and transaction conflicts must be handled appropriately.

### 6.3 Deduplicate Input Records

If the same event can arrive multiple times, deduplicate using a stable event identifier.

Example in PySpark:

```python
deduplicated_df = df.dropDuplicates(["event_id"])
```

This removes duplicate combinations of `event_id` within the DataFrame.

**Important:** If duplicate event IDs have different values in other columns, `dropDuplicates()` does not guarantee which row is retained. Use deterministic ordering and a window function when the latest or highest-priority record must be selected.

Deduplicating one batch also does not automatically prevent duplicates across separate pipeline runs. The target design must address that case.

### 6.4 Use Batch Identifiers or Idempotency Keys

For APIs or systems that support idempotency keys, send a stable identifier with the request.

The destination can recognize repeated requests with the same key and avoid repeating the operation's effect, according to its documented behavior.

For data pipelines, a stable batch identifier can help track whether a batch has already been committed.

The identifier should be reused for retries of the same logical operation, not regenerated for each attempt.

### 6.5 Use Transactions Where Supported

Transactions can atomically commit a supported group of changes.

For example, a Delta Lake write can commit a table update as a transaction.

However, a transaction in one table does not automatically make a workflow involving a table, an email, and an external API atomic. Each external side effect needs its own appropriate reliability strategy.

---

## 7. Retries and Idempotency in Apache Spark

Spark can retry failed tasks, but task retries and application-level retries are not identical.

### Task Retries

Spark may rerun a failed task according to the configured retry policy.

For transformations that compute results from input data, recomputing a task is generally part of Spark's execution model.

### Why Side Effects Need Care

Consider a Spark task that sends an email or calls an external API for every row.

A task might perform some external calls and then fail. If Spark retries the task, those calls may happen again.

This can produce duplicate external side effects.

### Better Design

- Prefer Spark transformations that compute data rather than directly performing external side effects.
- Use supported transactional or idempotent sink mechanisms.
- Give external operations stable idempotency keys when supported.
- Avoid assuming that a successful task attempt means all external side effects occurred exactly once.

Spark task retry settings control retry behavior; they do not automatically make arbitrary external operations idempotent.

---

## 8. Retries and Idempotency in Spark Structured Streaming

Structured Streaming uses checkpoints to support recovery of query progress and state.

A failure may cause a micro-batch or supported part of the processing workflow to be retried during recovery.

The result depends on the source, sink, and how the query writes data.

### Example

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

This example writes a stream to a Delta table using a persistent checkpoint location.

Checkpointing helps recover the query, but the example alone does not deduplicate repeated business events. If the source legitimately contains the same order event multiple times, the target may still contain multiple rows unless the pipeline handles those duplicates.

### Key Distinctions

- **Checkpointing:** Helps recover source progress and state.
- **Event deduplication:** Identifies repeated events using business rules or event IDs.
- **Idempotent sink:** Prevents retries from creating unintended repeated effects.
- **Exactly-once effective results:** Depend on the complete source-processing-sink design.

These mechanisms complement one another but are not interchangeable.

---

## 9. The Watermark Problem in Incremental Pipelines

Many pipelines track progress using a timestamp or an increasing identifier.

Example:

```sql
SELECT *
FROM source_orders
WHERE updated_at > :last_watermark;
```

Here, `last_watermark` represents the previously stored extraction boundary.

This approach can miss data if:
- A record arrives late.
- An existing record is updated with an earlier or unchanged timestamp.
- The watermark is advanced before the target write succeeds.
- Multiple records share the same timestamp and the boundary is handled incorrectly.

### Safer Design

1. Read data using a defined extraction boundary.
2. Use a lookback window or a reliable CDC mechanism when required.
3. Write the data using idempotent logic.
4. Confirm the target write has succeeded.
5. Advance the extraction watermark only after successful processing.
6. Make recovery safe if the target write succeeds but the watermark update fails.

If the target write succeeds but the watermark update fails, the same data may be read again. Idempotent writes help make that replay safe.

**Important:** Advancing the watermark after a successful write is necessary, but not sufficient by itself. The pipeline must also handle partial failures and repeated execution correctly.

---

## 10. Common Failure Scenarios

### Scenario 1: The Request Times Out

The client sends a write request. The server commits the data, but the response is lost.

The client cannot know from the timeout alone whether the write succeeded.

**Solution:** Check the operation's status where possible, or retry using an idempotency key or idempotent write strategy.

### Scenario 2: The Target Write Succeeds but the Job Crashes

The target data is committed, but the job fails before recording its completion.

The job may retry the same batch.

**Solution:** Use a transaction-aware or idempotent destination design and durable progress tracking.

### Scenario 3: A Task Fails After Calling an External API

The API call succeeds, but the Spark task fails before reporting success.

Spark retries the task, potentially repeating the API call.

**Solution:** Use API-supported idempotency keys or move side effects to a controlled delivery process.

### Scenario 4: A Pipeline Reruns Yesterday's Data

A scheduled job is rerun after a failure or for a correction.

If it blindly appends the same records, duplicates may appear.

**Solution:** Use deterministic batch boundaries, deduplication, upserts, or a carefully defined partition replacement strategy.

### Scenario 5: Two Jobs Update the Same Key

Two concurrent jobs attempt to update the same business record.

**Solution:** Define conflict handling, ensure source keys are unique within each merge operation, and retry supported transaction conflicts when appropriate. A retry should re-read or recompute from valid inputs when necessary.

---

## 11. Retries vs Idempotency vs Checkpointing

| Concept | Main Purpose | Example |
|---|---|---|
| Retry | Attempt failed work again | Retry a temporary storage error |
| Idempotency | Make repeated operations safe | Upsert an order by a stable key |
| Checkpointing | Save progress or state for recovery | Structured Streaming checkpoint |
| Deduplication | Identify and remove repeated records | Deduplicate by `event_id` |
| Transaction | Commit supported changes atomically | Delta Lake table commit |
| Watermark | Track extraction progress or event-time progress, depending on context | Incremental extraction boundary |

The term *watermark* has different meanings in batch incremental extraction and streaming event-time processing. Do not confuse an extraction watermark with a Structured Streaming event-time watermark.

A reliable pipeline often combines several of these techniques.

---

## 12. Designing a Retry-Safe Data Pipeline

Suppose a pipeline ingests order events, transforms them using PySpark, and writes the latest order state into a Delta table.

### Recommended Flow

1. Read from a durable source.
2. Identify records using stable event and business keys.
3. Validate the input.
4. Deduplicate repeated events using deterministic rules.
5. Apply transformations.
6. Upsert into the target using the intended business key.
7. Record successful progress.
8. Monitor failures and retry transient errors.
9. Test replay and recovery scenarios.

### Example Architecture

```text
Durable Source
      |
      v
Read Batch or Stream
      |
      v
Validate and Deduplicate
      |
      v
Transform Data
      |
      v
Idempotent Target Write
      |
      v
Confirm Successful Commit
      |
      v
Advance Progress / Continue
```

The precise commit and progress-recording mechanism depends on the orchestration system, source, and sink.

### Reliability Checklist

- Is the retry policy bounded?
- Are transient and permanent errors distinguished?
- Can the same batch run twice without incorrect results?
- Are business keys stable and appropriate?
- Are source duplicates handled deterministically?
- Can the pipeline recover from a crash after the target write?
- Is progress updated only at the appropriate point?
- Are external side effects protected against repeated calls?
- Are retry counts, failures, and data-quality issues monitored?

---

## 13. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Retrying always fixes failures | Permanent errors need correction |
| A timeout proves the operation failed | The operation may have succeeded without the response reaching the client |
| Idempotency means an operation runs only once | The operation may run multiple times but produce the same final effect |
| Checkpointing automatically prevents duplicate business records | Checkpointing tracks progress and state; business deduplication may still be required |
| `dropDuplicates()` always keeps the latest record | It does not guarantee which row is retained when other column values differ |
| A Delta `MERGE` is automatically safe for every input | Keys, source uniqueness, concurrency, and business rules still matter |
| Spark task retries guarantee exactly-once API calls | External side effects can be repeated |
| Updating a watermark is enough for retry safety | The target write and progress update must be designed for partial failures |
| Every failure should be retried | Retry only when appropriate for the failure and operation |
| A transaction makes the entire pipeline atomic | A transaction protects only the operations within its supported transaction boundary |

---

## 14. Interview Questions and Answers

### Q1. What is a retry?

A retry is another attempt to perform an operation after a failure. Retries are useful for transient errors but may not help with permanent errors.

### Q2. What is idempotency?

Idempotency means repeating an operation under defined conditions produces the same final effect as performing it once.

### Q3. Why are retries and idempotency used together?

Retries help recover from failures, while idempotency prevents repeated attempts from creating unintended duplicate or inconsistent results.

### Q4. What is exponential backoff?

Exponential backoff increases the waiting time between successive retries, helping reduce pressure on an overloaded service.

### Q5. What is jitter?

Jitter adds randomness to retry delays so that many clients do not retry at exactly the same time.

### Q6. How can you make a data pipeline idempotent?

Use stable keys, deterministic transformations, deduplication where appropriate, upserts or transactional writes, and retry-safe progress tracking.

### Q7. Does `dropDuplicates()` make a pipeline fully idempotent?

No. It can remove duplicate keys within a DataFrame, but it does not automatically prevent duplicates across pipeline runs or make external side effects safe.

### Q8. Why can Spark task retries create duplicate API calls?

A task may perform an external call successfully and then fail before reporting completion. A retry may repeat the call.

### Q9. Does a streaming checkpoint guarantee exactly-once output?

Not for every sink or external operation. Checkpointing supports recovery, but end-to-end guarantees depend on the source, processing logic, and sink behavior.

### Q10. What happens if the target write succeeds but the watermark update fails?

The next run may read the same records again. An idempotent target write allows the pipeline to process them again without unintended duplicate effects.

### Q11. What is the difference between retry and deduplication?

Retry repeats an operation after failure. Deduplication identifies repeated records and removes or resolves them according to defined rules.

### Q12. How would you make an API request retry-safe?

Use a stable idempotency key if the API supports it, retry transient failures with bounded backoff, and check the operation status when the result is uncertain.

### Q13. Is an upsert always idempotent?

Not automatically. It can be idempotent when keys, source data, and update logic are deterministic and repeated application produces the intended same target state.

### Q14. How would you handle a batch that may have partially succeeded?

Use durable progress tracking, transactional or idempotent writes, and a recovery procedure that can safely reprocess the batch. Verify the sink's behavior during partial failures.

---

## 15. Quick Revision

- **Retry:** Attempt failed work again.
- **Idempotency:** Repeated operation has the same final effect under defined conditions.
- **Exponential backoff:** Increase the wait between retries.
- **Jitter:** Randomize retry delays.
- **Stable key:** Identifies the same entity or event consistently.
- **Upsert:** Insert a new record or update an existing one.
- **Deduplication:** Identify and resolve repeated records.
- **Checkpointing:** Save progress or state for recovery.
- **Watermark:** Track progress; its meaning depends on the pipeline context.
- **Unknown write outcome:** The operation may have succeeded even if the response was lost.
- **Exactly-once effective results:** Depend on the complete source-processing-sink design.
- **Best practice:** Design for safe replay before enabling automatic retries.

### One-Minute Interview Answer

Retries allow a pipeline to recover from temporary failures, while idempotency ensures that repeating an operation does not create an unintended additional effect.

For example, a Spark pipeline may retry after a failed task, but if the task already called an external API, the call could be repeated. I would make the operation retry-safe using stable idempotency keys or an appropriate destination design.

For data pipelines, I would use deterministic transformations, stable business keys, deduplication where needed, transactional or idempotent writes, and durable progress tracking. I would also use bounded retries with backoff for transient failures. Checkpointing helps recovery, but it does not guarantee exactly-once effects for arbitrary external sinks.

---

## 16. Self-Assessment

- [ ] Define retry and idempotency.
- [ ] Explain why a timeout does not prove that a write failed.
- [ ] Compare fixed-delay and exponential-backoff retries.
- [ ] Explain jitter and bounded retries.
- [ ] Design an idempotent upsert.
- [ ] Explain the limitations of `dropDuplicates()`.
- [ ] Describe how Spark task retries can repeat external side effects.
- [ ] Explain the role and limitations of streaming checkpoints.
- [ ] Explain safe watermark updates in incremental ingestion.
- [ ] Design a pipeline that can safely rerun a partially completed batch.

**Interview readiness:** Be able to explain this scenario clearly: *A data write succeeds, but the application crashes before recording success. How will you retry without creating duplicate records?*