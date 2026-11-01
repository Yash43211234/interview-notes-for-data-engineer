# Output Modes in Spark Structured Streaming

## 1. What Are Output Modes?

Output modes define how Spark Structured Streaming writes the results of a streaming query to its output sink.

They determine which rows are written when a new streaming trigger processes data.

Spark Structured Streaming supports three main output modes:

1. Append Mode
2. Update Mode
3. Complete Mode

**Key idea:** Output mode controls which results are written to the sink, not how frequently Spark reads incoming data.

---

## 2. Append Mode

Append mode writes only new rows that were added to the result table since the previous trigger.

Previously written rows are not written again by the query.

### Example

Suppose a streaming application receives order records.

| Batch | New order | Amount |
|---|---|---:|
| Batch 1 | Order 101 | 500 |
| Batch 2 | Order 102 | 700 |
| Batch 3 | Order 103 | 300 |

With Append mode:

- Batch 1 writes Order 101.
- Batch 2 writes Order 102.
- Batch 3 writes Order 103.

The output contains newly added rows.

### PySpark example

```python
query = (
    stream_df.writeStream
        .format("console")
        .outputMode("append")
        .start()
)
```

### When to use Append Mode

- Writing incoming events.
- Saving new transactions.
- Processing records that will not change after being added to the result.
- Writing finalized aggregation results when the required event-time watermark conditions are satisfied.

**Important:** Append mode is not supported for every streaming query. For example, aggregations that can update previously emitted results generally need a suitable watermark or another supported query design.

---

## 3. Update Mode

Update mode writes only the rows that were updated in the result table since the previous trigger.

It does not rewrite every row in the result table.

### Example

Suppose you want to calculate total sales by country.

Incoming data:

| Batch | Country | Amount |
|---|---|---:|
| Batch 1 | India | 500 |
| Batch 1 | USA | 300 |
| Batch 2 | India | 200 |
| Batch 3 | USA | 100 |

After Batch 1:

| Country | Total Sales |
|---|---:|
| India | 500 |
| USA | 300 |

After Batch 2:

| Country | Total Sales |
|---|---:|
| India | 700 |
| USA | 300 |

After Batch 3:

| Country | Total Sales |
|---|---:|
| India | 700 |
| USA | 400 |

With Update mode, Spark writes only the aggregate rows that changed during each trigger.

- Batch 1: India = 500, USA = 300.
- Batch 2: India = 700.
- Batch 3: USA = 400.

### PySpark example

```python
from pyspark.sql.functions import sum

aggregated_df = (
    stream_df
    .groupBy("country")
    .agg(sum("amount").alias("total_sales"))
)

query = (
    aggregated_df.writeStream
        .format("console")
        .outputMode("update")
        .start()
)
```

### When to use Update Mode

- Running aggregations.
- Maintaining continuously updated totals.
- Monitoring sales by country.
- Tracking changing metrics.

**Important:** Update mode writes changed result rows, but the sink must be able to handle updates correctly. A sink that simply appends every output row can contain multiple versions of the same aggregate key.

---

## 4. Complete Mode

Complete mode writes the entire result table after every trigger.

It includes both newly added and previously existing result rows.

### Example

Using the same sales aggregation:

After Batch 1:

| Country | Total Sales |
|---|---:|
| India | 500 |
| USA | 300 |

After Batch 2:

| Country | Total Sales |
|---|---:|
| India | 700 |
| USA | 300 |

After Batch 3:

| Country | Total Sales |
|---|---:|
| India | 700 |
| USA | 400 |

With Complete mode:

- Batch 1 writes the entire two-row result.
- Batch 2 writes the entire updated two-row result.
- Batch 3 writes the entire updated two-row result.

### PySpark example

```python
query = (
    aggregated_df.writeStream
        .format("console")
        .outputMode("complete")
        .start()
)
```

### When to use Complete Mode

- Small or manageable aggregation result tables.
- Applications that need the complete current result after every trigger.
- Monitoring dashboards that consume the full result.

**Important:** Complete mode can generate significant output as the result table grows because Spark writes the entire result table for each trigger. It is not supported for every query or sink.

---

## 5. Append vs Update vs Complete

| Feature | Append | Update | Complete |
|---|---|---|---|
| Writes new rows | Yes | Yes, if changed | Yes |
| Writes updated rows | No | Yes | Yes |
| Writes entire result | No | No | Yes |
| Rewrites unchanged rows | No | No | Yes |
| Common use | New events | Changed aggregates | Full aggregate result |
| Output volume | Usually lower | Depends on changed rows | Can be high |

### Easy way to remember

- Append → Only new result rows.
- Update → Only changed result rows.
- Complete → Entire result table.

---

## 6. Output Modes with Aggregations

Output modes become especially important when using aggregations in streaming queries.

Consider:

```python
from pyspark.sql.functions import sum

result = (
    stream_df
    .groupBy("country")
    .agg(sum("amount").alias("total_sales"))
)
```

As new records arrive, the total sales for a country can change.

For example:

```text
Batch 1:
India = 500

Batch 2:
India = 700

Batch 3:
India = 900
```

The aggregation result is updated over time.

- Append mode generally waits until an aggregation result is final before emitting it.
- Update mode emits the changed aggregate rows.
- Complete mode emits the entire current aggregation result.

For event-time aggregations, watermarks can help Spark determine when older windows are sufficiently complete.

---

## 7. What Is the Role of Watermarks?

A watermark helps Spark manage late-arriving data in event-time streaming queries.

It defines how long Spark should wait for late data before considering older event-time results complete.

### Example

```python
from pyspark.sql.functions import window, sum

result = (
    stream_df
    .withWatermark("event_time", "10 minutes")
    .groupBy(
        window("event_time", "5 minutes"),
        "country"
    )
    .agg(sum("amount").alias("total_sales"))
)
```

Here:

- The event-time window is 5 minutes.
- The watermark delay threshold is 10 minutes.

Once a window is considered complete under the watermark policy, Spark can emit its final result in Append mode.

**Important:**

- A watermark is not a guarantee that no later data will arrive.
- Data arriving later than the watermark threshold may be dropped from the relevant stateful aggregation.
- The actual behavior depends on the query and stateful operation.

---

## 8. Output Modes and Sinks

Not every sink supports every output mode.

The supported modes depend on the output format and query type.

For example:

| Sink | General considerations |
|---|---|
| Console | Useful for testing; supports output-mode demonstrations |
| Memory | Useful for testing and querying in-memory results |
| File-based sinks | Commonly support Append mode |
| Delta Lake | Supports streaming writes with sink-specific behavior and integration |
| Kafka | Writes output records to topics; update and complete modes have restrictions |

**Important:** Always check the documentation for the specific sink and Spark version. Do not assume that every sink supports all three output modes.

---

## 9. Output Mode vs Trigger

Output mode and trigger are different concepts.

### Output mode

Determines which rows are written.

Examples:
- Append.
- Update.
- Complete.

### Trigger

Determines when Spark executes the next processing cycle.

Example:

```python
query = (
    stream_df.writeStream
        .format("console")
        .outputMode("append")
        .trigger(processingTime="30 seconds")
        .start()
)
```

Here:
- Append mode controls which rows are written.
- The processing-time trigger requests a new processing cycle every 30 seconds.

**Remember:** Output mode controls what is written; trigger controls when processing occurs.

---

## 10. Common Mistakes

1. Confusing output modes with triggers.
2. Assuming Append mode writes every incoming record regardless of query semantics.
3. Assuming Update mode writes the entire result table.
4. Assuming Complete mode writes only changed rows.
5. Ignoring sink compatibility.
6. Assuming Append mode is always suitable for aggregations.
7. Forgetting that Update mode may require an upsert-capable destination.
8. Assuming watermarks guarantee that no late data will arrive.
9. Assuming output mode alone guarantees exactly-once delivery.

---

## 11. Interview Questions

**Q1. What are output modes in Structured Streaming?**

Output modes define which rows from the streaming query's result table are written to the sink.

**Q2. What are the three output modes?**

Append, Update, and Complete.

**Q3. What is Append mode?**

It writes only newly added result rows that have not previously been emitted.

**Q4. What is Update mode?**

It writes only result rows that changed since the previous trigger.

**Q5. What is Complete mode?**

It writes the entire current result table after every trigger.

**Q6. Which output mode is suitable for continuously updated aggregations?**

Update mode is commonly suitable because it writes only the aggregate rows that changed.

**Q7. Can Append mode be used with aggregations?**

Yes, for supported queries where results can be finalized, such as suitable event-time window aggregations with watermarks.

**Q8. Why can Complete mode be expensive?**

Because it writes the entire result table after every trigger, even when only a few rows have changed.

**Q9. What is the difference between Update and Complete mode?**

Update writes only changed rows, while Complete writes the entire current result table.

**Q10. What is the role of a watermark?**

It helps manage late-arriving event-time data and determine when older aggregation results can be considered complete.

**Q11. Is output mode the same as trigger?**

No. Output mode controls which rows are written; trigger controls when processing runs.

**Q12. Do all sinks support all output modes?**

No. Supported output modes depend on the sink, query type, and Spark version.

---

## 12. Quick Revision

| Concept | Remember |
|---|---|
| Append | Writes newly added result rows |
| Update | Writes changed result rows |
| Complete | Writes the entire result table |
| Watermark | Manages late event-time data |
| Trigger | Controls processing frequency |
| Sink | Destination for streaming output |
| Aggregation | Can produce changing results |
| Update-compatible sink | Must handle changed rows correctly |

### One-line revision

- Append → New rows only.
- Update → Changed rows only.
- Complete → Entire result table.
- Watermark → Handles late event-time data.
- Trigger → Controls when processing happens.

---

## 13. Self-Check

- [ ] Can I explain Append mode?
- [ ] Can I explain Update mode?
- [ ] Can I explain Complete mode?
- [ ] Can I differentiate all three modes?
- [ ] Can I explain output modes with an aggregation example?
- [ ] Can I explain why Append mode may require a watermark?
- [ ] Can I explain the role of a watermark?
- [ ] Can I differentiate output mode and trigger?
- [ ] Can I explain why sink compatibility matters?

**Final takeaway:** Append writes new result rows, Update writes changed result rows, and Complete writes the entire result table. Choose the output mode based on how your streaming query changes over time and what your sink supports.