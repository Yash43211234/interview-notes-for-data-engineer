# Data Compression in Data Engineering

## 1. What Is Data Compression?

**Data compression** is the process of reducing the amount of storage space required to store data.

It works by representing data more efficiently, removing repeated patterns, or encoding information using fewer bits.

In data engineering, compression is important because pipelines often process and store large volumes of data.

### Simple Example

Suppose a company stores 1 GB of raw data.

After compression, the same data may require only 300 MB.

```text
       ORIGINAL DATA
            |
            v
          1 GB
            |
            v
       COMPRESSION
            |
            v
        300 MB
```

The actual compression ratio depends on the data and compression algorithm.

### Why Is Compression Important?

- Reduces storage requirements.
- Reduces data transfer over networks.
- Can reduce cloud storage costs.
- Can improve query performance by reducing disk I/O.
- Helps manage large datasets efficiently.

**Important:** Compression reduces storage and transfer size, but decompression requires CPU resources. A highly compressed file is not always faster to process.

## 2. Types of Data Compression

There are two major categories:

1. Lossless Compression.
2. Lossy Compression.

### 2.1 Lossless Compression

Lossless compression reduces file size without losing the original information.

After decompression, the original data can be reconstructed exactly.

**Example:**

Original data:

```text
AAAAABBBBBCCCCCDDDDD
```

A simplified run-length representation:

```text
5A 5B 5C 5D
```

The original sequence can be reconstructed from the encoded representation.

This is useful for data engineering because analytical data generally needs to preserve its original values.

### Common Lossless Compression Algorithms

- Snappy.
- Gzip.
- Zstandard (ZSTD).
- LZ4.
- DEFLATE.

### 2.2 Lossy Compression

Lossy compression reduces file size by permanently removing some information.

The original data cannot be reconstructed exactly.

**Examples:**

- JPEG images.
- MP3 audio.
- Some video compression formats.

Lossy compression is commonly used for multimedia where a small loss in quality may be acceptable.

### Comparison

| Feature | Lossless | Lossy |
|---|---|---|
| Original data | Fully recoverable | Not fully recoverable |
| Data loss | No | Yes |
| Compression | Depends on data and algorithm | Can achieve much smaller sizes |
| Common use | Databases, logs, analytical datasets | Images, audio, video |
| Examples | Snappy, Gzip, ZSTD | JPEG, MP3 |

**Remember:** Lossless compression is generally preferred for analytical datasets where exact values must be preserved.

## 3. How Does Compression Work?

Compression algorithms identify patterns or represent information more efficiently.

Different algorithms use different techniques.

### 3.1 Run-Length Encoding (RLE)

RLE stores consecutive repeated values as a value and its count.

Original:

```text
AAAAABBBBCC
```

Compressed representation:

```text
5A 4B 2C
```

This works well when data contains long sequences of repeated values.

### 3.2 Dictionary Encoding

Dictionary encoding replaces repeated values with smaller identifiers.

Original values:

```text
IT, HR, IT, Finance, HR
```

Dictionary:

```text
0 = IT
1 = HR
2 = Finance
```

Encoded values:

```text
0, 1, 0, 2, 1
```

This is particularly useful for columns with relatively few distinct values.

### 3.3 Delta Encoding

Delta encoding stores differences between consecutive values instead of storing every value independently.

Original values:

```text
100, 105, 110, 115
```

Delta representation:

```text
100, +5, +5, +5
```

This can be effective for sorted or gradually changing numeric values.

### 3.4 Bit Packing

Bit packing stores values using only the number of bits needed to represent them.

For example, integers from 0 to 7 require only 3 bits each rather than 32 bits when stored as standard 32-bit integers.

### Important Distinction

Dictionary encoding, RLE, delta encoding, and bit packing are encoding techniques.

Snappy, Gzip, ZSTD, and LZ4 are compression algorithms or codecs.

Columnar formats such as Parquet can combine encoding techniques with compression codecs.

## 4. Common Compression Algorithms

| Algorithm | Type | General Characteristics |
|---|---|---|
| Snappy | Lossless | Fast compression and decompression |
| Gzip | Lossless | Good compression ratio, often slower than Snappy |
| Zstandard (ZSTD) | Lossless | Good balance of compression ratio and speed |
| LZ4 | Lossless | Very fast compression and decompression |
| DEFLATE | Lossless | Combines LZ77 and Huffman coding |

### 4.1 Snappy

Snappy is designed for fast compression and decompression.

Characteristics:
- Fast processing.
- Moderate compression ratio.
- Useful for data processing workloads.
- Commonly supported by Parquet.

### 4.2 Gzip

Gzip is a widely used lossless compression format.

Characteristics:
- Often provides better compression than Snappy.
- Can require more CPU time.
- Common in file transfer and archival workflows.
- Supported by many data processing tools.

### 4.3 Zstandard (ZSTD)

Zstandard provides configurable compression levels and aims to balance speed with compression efficiency.

Characteristics:
- Good compression ratio.
- Fast decompression.
- Supports multiple compression levels.
- Useful for modern data storage and analytics workloads.

### 4.4 LZ4

LZ4 focuses on very fast compression and decompression.

Characteristics:
- High processing speed.
- Generally lower compression ratio than stronger codecs.
- Useful when processing speed is important.

### Practical Comparison

| Requirement | Possible Choice |
|---|---|
| Fast compression and decompression | Snappy or LZ4 |
| Stronger compression | Gzip or ZSTD |
| Balanced speed and compression | ZSTD |
| Broad compatibility | Gzip |

These are general tendencies, not guaranteed performance rankings. Actual results depend on data, codec settings, hardware, and processing engine.

## 5. Compression in Columnar File Formats

Compression is especially important in columnar formats such as:

- Apache Parquet.
- Apache ORC.

These formats organize data by columns and can apply encoding and compression to column data.

### Example

Consider an employee dataset:

| employee_id | department | salary |
|---:|---|---:|
| 101 | IT | 50000 |
| 102 | IT | 60000 |
| 103 | HR | 55000 |
| 104 | IT | 52000 |

The department column contains repeated values.

A columnar format may use dictionary encoding:

```text
Dictionary:
0 = IT
1 = HR

Encoded values:
0, 0, 1, 0
```

A compression codec can then compress the encoded data further.

### Why Is This Useful?

1. Similar values are stored together.
2. Repeated values can be encoded efficiently.
3. Compression can reduce storage size.
4. Analytical queries can read only required columns.
5. Less data may need to be transferred from storage.

**Important:** Columnar storage and compression are related but different concepts. Columnar storage describes data organization; compression describes how the data is encoded into a smaller representation.

## 6. Compression in Apache Parquet

Parquet is a columnar file format commonly used in Spark and data lake architectures.

It supports multiple compression codecs.

Common options include:

- Snappy.
- Gzip.
- ZSTD.
- LZ4_RAW.
- Uncompressed.

### Writing Parquet with Snappy

```python
df.write \
    .mode("overwrite") \
    .option("compression", "snappy") \
    .parquet("/data/output/employees/")
```

### Writing Parquet with Gzip

```python
df.write \
    .mode("overwrite") \
    .option("compression", "gzip") \
    .parquet("/data/output/employees_gzip/")
```

### Writing Parquet with ZSTD

```python
df.write \
    .mode("overwrite") \
    .option("compression", "zstd") \
    .parquet("/data/output/employees_zstd/")
```

### Reading Parquet

```python
df = spark.read.parquet("/data/output/employees/")

df.show()
```

Spark reads the file and handles decompression automatically.

The codec must be supported by the Spark runtime and file configuration.

## 7. Compression in Apache ORC

ORC is another columnar file format that supports compression.

Common codecs include:

- ZLIB.
- Snappy.
- ZSTD.
- LZ4.
- LZO, where supported.

### Writing ORC with Snappy

```python
df.write \
    .mode("overwrite") \
    .option("compression", "snappy") \
    .orc("/data/output/employees_orc/")
```

### Writing ORC with ZSTD

```python
df.write \
    .mode("overwrite") \
    .option("compression", "zstd") \
    .orc("/data/output/employees_orc_zstd/")
```

### Reading ORC

```python
df = spark.read.orc("/data/output/employees_orc/")

df.show()
```

## 8. Compression in PySpark

PySpark supports compression for several file formats.

### CSV with Gzip

```python
df.write \
    .mode("overwrite") \
    .option("compression", "gzip") \
    .option("header", True) \
    .csv("/data/output/employees_csv/")
```

### JSON with Gzip

```python
df.write \
    .mode("overwrite") \
    .option("compression", "gzip") \
    .json("/data/output/employees_json/")
```

### Parquet with Snappy

```python
df.write \
    .mode("overwrite") \
    .option("compression", "snappy") \
    .parquet("/data/output/employees_parquet/")
```

**Important:** Compression support and option names can vary by file format and Spark version. Parquet and ORC commonly use Snappy by default in standard Spark configurations, but platform settings can override defaults.

## 9. How to Choose a Compression Algorithm?

The best compression algorithm depends on the workload.

Consider these factors:

### 9.1 Compression Ratio

How much does the algorithm reduce the original file size?

A higher compression ratio means less storage is required.

### 9.2 Compression Speed

How quickly can the algorithm compress data?

This matters when pipelines process large volumes frequently.

### 9.3 Decompression Speed

How quickly can the compressed data be read?

This is especially important for analytical queries.

### 9.4 CPU Usage

Compression and decompression require CPU resources.

Stronger compression can increase CPU usage.

### 9.5 Compatibility

The selected codec must be supported by the storage format and processing engine.

### Example Decision

| Scenario | Consideration |
|---|---|
| Frequent analytical reads | Fast decompression |
| Long-term archival | Stronger compression |
| High-throughput pipelines | Compression speed |
| Cloud storage cost reduction | Compression ratio and total compute cost |
| Cross-platform data exchange | Codec compatibility |

There is no universally best codec. Benchmark representative data using the actual workload.

## 10. Compression Ratio and Space Savings

Two useful measurements are compression ratio and space savings.

### Compression Ratio

\[
\text{Compression Ratio} =
\frac{\text{Original Size}}{\text{Compressed Size}}
\]

Example:

- Original size = 1000 MB.
- Compressed size = 250 MB.

\[
\text{Compression Ratio} = \frac{1000}{250} = 4:1
\]

This means the compressed data is one-fourth of its original size.

### Space Savings

\[
\text{Space Savings} =
\frac{\text{Original Size} - \text{Compressed Size}}
{\text{Original Size}} \times 100
\]

Using the same example:

\[
\frac{1000-250}{1000}\times100 = 75\%
\]

<text>**Result:** 75% storage space saved.</text>

### Important Difference

- Compression ratio = 4:1.
- Space savings = 75%.

Do not confuse these two measurements.

## 11. Compression and Cloud Data Engineering

Compression can reduce the amount of data stored in cloud platforms.

Consider a data pipeline using Azure:

```text
        SOURCE DATABASE
               |
               v
        AZURE DATA FACTORY
               |
               v
         ADLS GEN2
               |
               v
        PARQUET FILES
        (SNAPPY / ZSTD)
               |
               v
          DATABRICKS
               |
               v
        DELTA TABLES
               |
               v
          ANALYTICS
```

### Benefits

**1. Storage Cost Reduction**

Smaller files generally require less storage capacity.

**2. Lower Data Transfer**

Compressed files can reduce the amount of data transferred over a network.

**3. Query Performance**

Queries may read less data from storage, although decompression also consumes CPU.

**4. Better Storage Efficiency**

Columnar formats and compression techniques work together to store analytical datasets efficiently.

### Important Consideration

Cloud cost is not determined by storage size alone.

It can also depend on:
- Compute usage.
- Data transfer.
- Request operations.
- Storage tier.
- Retention period.
- Platform pricing.

Compression can reduce some costs, but it does not guarantee lower total pipeline cost.

## 12. Compression in Delta Lake

Delta Lake uses Parquet data files along with a transaction log.

Therefore, Parquet compression settings are relevant to Delta table storage.

### Example

```python
df.write \
    .format("delta") \
    .mode("overwrite") \
    .option("compression", "snappy") \
    .save("/data/delta/employees/")
```

Read the Delta table:

```python
df = spark.read.format("delta").load("/data/delta/employees/")

df.show()
```

### Important Points

- Delta data files are Parquet files.
- Compression applies to the underlying data files.
- The transaction log tracks table changes and metadata.
- Compression does not replace Delta Lake's transaction capabilities.
- `OPTIMIZE` reorganizes Delta data files; it is not simply a compression operation.

## 13. Compression vs Encoding

These concepts are related but not identical.

### Encoding

Encoding changes how values are represented.

Examples:
- Dictionary encoding.
- Run-length encoding.
- Delta encoding.
- Bit packing.

### Compression

Compression reduces the size of encoded data using a compression algorithm.

Examples:
- Snappy.
- Gzip.
- ZSTD.
- LZ4.

### Comparison

| Feature | Encoding | Compression |
|---|---|---|
| Purpose | Represent values efficiently | Reduce data size |
| Examples | Dictionary, RLE, Delta | Snappy, Gzip, ZSTD |
| Main benefit | Efficient representation | Smaller storage and transfer size |
| Relationship | Can prepare data for compression | Can compress encoded data |

A columnar format may apply one or more encoding techniques and then use a compression codec.

## 14. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Compression always improves query speed | It reduces I/O but adds decompression CPU cost |
| Gzip is always better than Snappy | Gzip often compresses more, while Snappy is generally faster |
| Snappy provides the highest compression ratio | Snappy prioritizes speed |
| Parquet and compression are the same | Parquet is a file format; compression is one of its storage capabilities |
| Dictionary encoding is a compression codec | It is an encoding technique |
| Lossy compression preserves exact data | Lossy compression permanently removes some information |
| All compression algorithms work with every file format | Codec support depends on the format and runtime |
| Delta Lake is a compression algorithm | Delta Lake is a table format and storage framework |
| OPTIMIZE is the same as compression | OPTIMIZE reorganizes data files; compression is a separate storage technique |
| Higher compression always means lower total cost | CPU, I/O, storage, and execution costs must all be considered |

## 15. Interview Questions and Answers

**Q1. What is data compression?**

Data compression is the process of reducing the amount of storage space required to represent data. It can also reduce the amount of data transferred over a network.

**Q2. What are the two main types of compression?**

Lossless and lossy compression.

Lossless compression preserves the original information, while lossy compression permanently removes some information.

**Q3. What is lossless compression?**

Lossless compression reduces data size without losing information. The original data can be reconstructed exactly after decompression.

**Q4. What is lossy compression?**

Lossy compression reduces file size by removing some information permanently. JPEG and MP3 are common examples.

**Q5. What is Snappy?**

Snappy is a lossless compression codec designed for fast compression and decompression. It is commonly used with Parquet.

**Q6. What is the difference between Snappy and Gzip?**

Snappy generally prioritizes speed, while Gzip often provides a higher compression ratio at the cost of more CPU time.

**Q7. What is ZSTD?**

Zstandard is a lossless compression algorithm that provides configurable compression levels and a balance between compression ratio and speed.

**Q8. Why is compression useful in data engineering?**

It reduces storage requirements and data transfer. It can also improve analytical performance by reducing disk I/O.

**Q9. Why is Parquet commonly used with compression?**

Parquet is a columnar format. Similar values are stored together, allowing efficient encoding and compression. This is useful for analytical workloads.

**Q10. What is dictionary encoding?**

Dictionary encoding stores unique values in a dictionary and replaces repeated values with smaller identifiers.

**Q11. What is run-length encoding?**

Run-length encoding represents consecutive repeated values using a value and its repetition count.

**Q12. What is the difference between encoding and compression?**

Encoding changes how values are represented, while compression reduces the size of the encoded representation.

**Q13. Does compression always improve performance?**

No. Compression can reduce disk I/O and network transfer, but decompression consumes CPU. Performance depends on the workload and codec.

**Q14. How do you enable compression in PySpark?**

For example, while writing Parquet:

```python
df.write \
    .option("compression", "snappy") \
    .parquet("/data/output/")
```

**Q15. What is a compression ratio?**

Compression ratio is the original data size divided by the compressed data size.

For example, compressing 1000 MB into 250 MB gives a 4:1 compression ratio.

**Q16. How does compression help in cloud data platforms?**

It reduces storage requirements and can reduce data transfer. It may also improve query performance by reducing the amount of data read from storage.

**Q17. Is Delta Lake a compression format?**

No. Delta Lake is a table format and storage framework built on Parquet data files and a transaction log. Parquet supports compression codecs.

## 16. Quick Revision

| Concept | Remember |
|---|---|
| Data Compression | Reduces data size |
| Lossless | Original data can be reconstructed exactly |
| Lossy | Some information is permanently removed |
| Snappy | Fast compression and decompression |
| Gzip | Often stronger compression, more CPU |
| ZSTD | Configurable balance of speed and compression |
| LZ4 | Very fast compression and decompression |
| Dictionary Encoding | Replaces values with dictionary IDs |
| RLE | Stores repeated values with counts |
| Delta Encoding | Stores differences between values |
| Bit Packing | Uses fewer bits for values |
| Parquet | Columnar file format |
| ORC | Columnar file format |
| Compression Ratio | Original size / compressed size |
| Space Savings | Percentage of storage reduced |
| Delta Lake | Uses Parquet data files and transaction log |

### One-Minute Interview Answer

Data compression is the process of reducing the storage size of data. It is important in data engineering because it helps reduce storage requirements and data transfer.

There are two main types: lossless and lossy compression. Lossless compression preserves the original information, while lossy compression permanently removes some information.

Common lossless codecs include Snappy, Gzip, ZSTD, and LZ4. Snappy is generally preferred when speed is important, while Gzip can provide stronger compression. ZSTD offers a configurable balance between speed and compression ratio.

In data engineering, compression is commonly used with columnar formats such as Parquet and ORC. For example, when writing Parquet files using PySpark, we can specify Snappy or ZSTD compression.

The choice depends on compression ratio, CPU usage, read performance, compatibility, and workload requirements.

## 17. Self-Assessment

- [ ] Define data compression.
- [ ] Explain why compression is important.
- [ ] Differentiate lossless and lossy compression.
- [ ] Explain Snappy, Gzip, ZSTD, and LZ4.
- [ ] Explain dictionary encoding.
- [ ] Explain run-length encoding.
- [ ] Explain delta encoding.
- [ ] Differentiate encoding and compression.
- [ ] Explain compression in Parquet.
- [ ] Explain compression in ORC.
- [ ] Write PySpark code to compress Parquet files.
- [ ] Explain compression in Delta Lake.
- [ ] Calculate compression ratio.
- [ ] Calculate storage space savings.
- [ ] Explain the trade-off between CPU usage and storage savings.

**Interview readiness:** You should be able to explain lossless and lossy compression, compare common codecs, describe how Parquet uses encoding and compression, and write a basic PySpark example using Snappy or ZSTD.