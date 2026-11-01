# Azure Data Factory: Incremental Copy Pattern

## 1. What Is Incremental Loading?

Incremental loading is a data ingestion technique in which a pipeline processes only new or changed records instead of copying the entire source dataset every time.

It reduces:

- Data movement.
- Pipeline execution time.
- Source database load.
- Storage and processing costs.

### Example

Suppose a source table contains 10 million customer records.

Every day, only 5,000 records are inserted or updated.

**Full Load:** Copies all 10 million records.

**Incremental Load:** Copies only the 5,000 new or changed records, assuming the source tracking mechanism can identify them correctly.

```text
Source Table
     |
     v
Identify New or Changed Records
     |
     v
Copy Only Incremental Data
     |
     v
Target Table
```

Incremental loading is commonly used in production data engineering pipelines.

---

## 2. Full Load vs Incremental Load

| Feature | Full Load | Incremental Load |
|---|---|---|
| Data copied | Entire dataset | New or changed data |
| Execution time | Usually higher | Usually lower |
| Source load | Higher | Usually lower |
| Data movement | More | Less |
| Complexity | Simpler | Requires tracking |
| Common use | Initial load, small datasets | Regular production loads |

### Example

Source table:

| order_id | amount | updated_at |
|---|---:|---|
| 101 | 500 | 2026-10-20 10:00 |
| 102 | 800 | 2026-10-21 11:00 |
| 103 | 900 | 2026-10-22 12:00 |

Suppose the last successful watermark is:

`2026-10-21 00:00:00`

The next incremental load selects records updated after that watermark.

```sql
SELECT *
FROM dbo.Orders
WHERE updated_at > '2026-10-21 00:00:00';
```

The records for orders 102 and 103 are selected.

**Important:** This simple example assumes that `updated_at` is reliably maintained and that the watermark strategy accounts for late updates and timestamp precision.

---

## 3. What Is a Watermark?

A watermark is a stored value that tracks how far an incremental extraction has progressed.

It is commonly based on:

- Last processed timestamp.
- Maximum processed ID.
- Source change version.
- CDC position or log sequence number.

### Example

Suppose a pipeline loads orders based on `updated_at`.

| Pipeline Run | Watermark Before Run | Watermark After Successful Run |
|---|---|---|
| Run 1 | 2026-10-20 00:00 | 2026-10-21 00:00 |
| Run 2 | 2026-10-21 00:00 | 2026-10-22 00:00 |
| Run 3 | 2026-10-22 00:00 | 2026-10-23 00:00 |

The watermark identifies the extraction progress.

### Why Is It Important?

Without a watermark, the pipeline may repeatedly process old records or miss the intended incremental range.

A watermark helps identify which records should be extracted in the next run.

---

## 4. Architecture of an Incremental Copy Pipeline

A common ADF incremental copy pattern uses:

1. Lookup Activity.
2. Copy Activity.
3. Stored Procedure Activity or another mechanism to update the watermark.

### Architecture

```text
                ADF Pipeline
                     |
                     v
              Lookup Watermark
                     |
                     v
              Read Last Value
                     |
                     v
                Copy Activity
                     |
                     v
              Source Data Query
                     |
                     v
                Target Storage
                     |
                     v
             Update Watermark
```

### Execution Steps

1. Read the previous successful watermark.
2. Identify the upper boundary for the current extraction.
3. Extract records within the incremental range.
4. Write the extracted data to the destination.
5. Validate the write.
6. Update the watermark only after the target write succeeds.

The upper boundary is important because it creates a stable extraction window.

---

## 5. Watermark Table

A watermark table stores the last successfully processed value for each source table or dataset.

### Example SQL Table

```sql
CREATE TABLE dbo.WatermarkTable
(
    table_name VARCHAR(100) PRIMARY KEY,
    last_watermark DATETIME2
);
```

### Sample Data

| table_name | last_watermark |
|---|---|
| Customers | 2026-10-20 00:00:00 |
| Orders | 2026-10-21 00:00:00 |
| Products | 2026-10-19 00:00:00 |

Each table can maintain its own watermark.

This is useful in metadata-driven pipelines that process multiple source tables.

### Read Watermark

```sql
SELECT last_watermark
FROM dbo.WatermarkTable
WHERE table_name = 'Orders';
```

### Update Watermark

```sql
UPDATE dbo.WatermarkTable
SET last_watermark = '2026-10-22 00:00:00'
WHERE table_name = 'Orders';
```

In production, the new watermark should be derived from the successfully processed extraction boundary, not an arbitrary current timestamp.

---

## 6. High Watermark and Low Watermark

Incremental loading often uses two boundaries.

- **Low watermark:** The last successfully processed boundary.
- **High watermark:** The upper boundary selected for the current extraction.

The pipeline extracts records between these boundaries.

### Example

Low watermark:

`2026-10-20 00:00:00`

High watermark:

`2026-10-22 00:00:00`

Query:

```sql
SELECT *
FROM dbo.Orders
WHERE updated_at > '2026-10-20 00:00:00'
  AND updated_at <= '2026-10-22 00:00:00';
```

The next successful watermark becomes the high watermark.

### Why Use Both Boundaries?

Using a fixed upper boundary makes each extraction window predictable.

Without