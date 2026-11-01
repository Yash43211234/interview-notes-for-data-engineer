# ADLS Gen2 Integration with Azure Databricks

## 1. What Is ADLS Gen2 Integration?

Azure Databricks integrates with Azure Data Lake Storage Gen2 (ADLS Gen2) to read, process, and write large volumes of data.

- **ADLS Gen2:** Stores data in the cloud.
- **Azure Databricks:** Uses Apache Spark to process and transform data.
- **Delta Lake:** Provides reliable table storage with features such as ACID transactions and schema enforcement.

A common data engineering workflow is:

1. Ingest source data into ADLS Gen2.
2. Read raw files from ADLS Gen2 using Databricks.
3. Clean and transform data using PySpark.
4. Write transformed data back to ADLS Gen2.
5. Store curated data as Delta tables for analytics.

## 2. Architecture

```text
          Source Systems
                 |
                 v
       Azure Data Factory
                 |
                 v
          ADLS Gen2
          /    |     \
         /     |      \
      Bronze  Silver  Gold
         |      |      |
         +------+------+
                |
                v
       Azure Databricks
           (Spark)
                |
                v
        Delta Lake Tables
                |
                v
       Analytics / Reporting
```

The architecture can vary. For example, Databricks can also ingest directly from source systems instead of receiving data through ADF.

### Common storage layers

| Layer | Purpose | Example |
|---|---|---|
| Bronze | Raw source data | Original CSV or JSON |
| Silver | Cleaned and validated data | Deduplicated customer records |
| Gold | Business-ready data | Daily sales summary |

## 3. ADLS Gen2 URI Format

Databricks commonly accesses ADLS Gen2 using the `abfss://` URI scheme.

```text
abfss://<container>@<storage-account>.dfs.core.windows.net/<path>
```

Example:

```text
abfss://bronze@stdataeng001.dfs.core.windows.net/orders/
```

| Component | Meaning |
|---|---|
| `abfss://` | Secure Hadoop-compatible filesystem scheme |
| `bronze` | Container or filesystem name |
| `stdataeng001` | Storage account name |
| `dfs.core.windows.net` | ADLS Gen2 DFS endpoint |
| `orders/` | Directory path |

### ABFS vs ABFSS

- `abfs://` uses the ABFS filesystem connector without explicitly requesting TLS through the URI scheme.
- `abfss://` explicitly requests secure TLS communication.

Prefer `abfss://` for secure ADLS Gen2 access.

**Important:** The URI identifies the storage location. It does not authenticate the user or grant access.

## 4. Authentication Methods

Databricks must authenticate before accessing protected ADLS Gen2 data.

Common approaches include:

1. Microsoft Entra ID service principal.
2. Managed identity.
3. Unity Catalog storage credentials and external locations.
4. SAS tokens or account keys in specific legacy or controlled scenarios.

### 4.1 Service Principal

A service principal is an application identity registered in Microsoft Entra ID.

It can be used by Databricks to authenticate to Azure Storage.

Typically, it requires:

- Tenant ID
- Application (client) ID
- Client secret or certificate
- Appropriate storage permissions

The identity must also be authorized to access the required storage data.

### 4.2 Managed Identity

A managed identity provides an Azure-managed identity to an Azure resource.

Benefits:

- No need to manage application secrets.
- Authentication is handled through Microsoft Entra ID.
- Access can be controlled using Azure roles and permissions.

There are two main types:

| Type | Description |
|---|---|
| System-assigned | Created for a resource and tied to its lifecycle |
| User-assigned | Created independently and can be assigned to multiple supported resources |

A managed identity does not automatically receive access to ADLS Gen2. It still needs appropriate permissions.

### 4.3 Unity Catalog

Unity Catalog provides centralized data governance for Databricks.

It uses:

- **Storage credentials:** Represent the authentication mechanism used to access cloud storage.
- **External locations:** Connect a storage path with a storage credential.
- **Privileges:** Control which users and groups can access external locations and data.

This is a preferred governance approach for many modern Databricks deployments.

## 5. Required Permissions

Authentication and authorization are different.

- Authentication confirms the identity.
- Authorization determines what the identity can access.

For ADLS Gen2, access may be controlled through Azure RBAC and POSIX-style ACLs.

### Common Azure roles

| Role | Purpose |
|---|---|
| Storage Blob Data Reader | Read blob data |
| Storage Blob Data Contributor | Read, write, and delete blob data |
| Storage Blob Data Owner | Full data access, including ownership and ACL management |

A general Azure `Reader` role does not automatically grant permission to read blob contents.

### ACL permissions

ADLS Gen2 supports:

- `r` — Read
- `w` — Write
- `x` — Execute or traverse directories

For example, reading a file may require:
- Execute permission on its parent directories.
- Read permission on the file.

**Interview point:** A successful login does not guarantee access to the file. The identity must have the necessary data-plane permissions.

## 6. Reading Data from ADLS Gen2

Once authentication and permissions are configured, Databricks can read files using Spark.

### Read CSV

```python
storage_path = (
    "abfss://bronze@stdataeng001.dfs.core.windows.net/orders/"
)

df = (
    spark.read
    .option("header", "true")
    .option("inferSchema", "true")
    .csv(storage_path)
)

display(df)
```

For production pipelines, prefer defining an explicit schema rather than relying on schema inference.

### Read JSON

```python
json_path = (
    "abfss://bronze@stdataeng001.dfs.core.windows.net/customers/"
)

df = spark.read.json(json_path)

display(df)
```

### Read Parquet

```python
parquet_path = (
    "abfss://silver@stdataeng001.dfs.core.windows.net/orders/"
)

df = spark.read.parquet(parquet_path)

display(df)
```

### Read Delta

```python
delta_path = (
    "abfss://gold@stdataeng001.dfs.core.windows.net/sales/"
)

df = spark.read.format("delta").load(delta_path)

display(df)
```

## 7. Writing Data to ADLS Gen2

Databricks can write transformed data to ADLS Gen2.

### Write CSV

```python
output_path = (
    "abfss://silver@stdataeng001.dfs.core.windows.net/orders/"
)

(
    df.write
    .mode("overwrite")
    .option("header", "true")
    .csv(output_path)
)
```

### Write Parquet

```python
output_path = (
    "abfss://silver@stdataeng001.dfs.core.windows.net/orders_parquet/"
)

(
    df.write
    .mode("overwrite")
    .parquet(output_path)
)
```

### Write Delta

```python
output_path = (
    "abfss://gold@stdataeng001.dfs.core.windows.net/sales_delta/"
)

(
    df.write
    .format("delta")
    .mode("overwrite")
    .save(output_path)
)
```

### Common write modes

| Mode | Behavior |
|---|---|
| `append` | Adds new data |
| `overwrite` | Replaces existing data at the target |
| `error` / `errorifexists` | Fails if the target exists |
| `ignore` | Does nothing if the target exists |

**Important:** `overwrite` can replace existing data. Use it carefully in production pipelines. For incremental processing, consider append or Delta Lake `MERGE`, depending on the requirement.

Spark generally writes output as a directory containing one or more part files, not as one ordinary file.

## 8. Using ADLS Gen2 with the Medallion Architecture

The medallion architecture organizes data into Bronze, Silver, and Gold layers.

### Bronze Layer

Stores raw or minimally processed source data.

```python
bronze_path = (
    "abfss://bronze@stdataeng001.dfs.core.windows.net/orders/"
)

raw_df = spark.read.option("header", "true").csv(bronze_path)
```

Typical responsibilities:
- Preserve source data.
- Maintain ingestion metadata.
- Support reprocessing.
- Avoid unnecessary business transformations.

### Silver Layer

Stores cleaned and validated data.

```python
from pyspark.sql.functions import col

silver_df = (
    raw_df
    .filter(col("order_id").isNotNull())
    .dropDuplicates(["order_id"])
)

silver_path = (
    "abfss://silver@stdataeng001.dfs.core.windows.net/orders/"
)

(
    silver_df.write
    .format("delta")
    .mode("overwrite")
    .save(silver_path)
)
```

Typical responsibilities:
- Handle nulls and invalid records.
- Remove duplicates based on business rules.
- Apply data type conversions.
- Standardize column names and values.

### Gold Layer

Stores business-ready datasets.

```python
from pyspark.sql.functions import sum

gold_df = (
    silver_df
    .groupBy("customer_id")
    .agg(sum("amount").alias("total_amount"))
)

gold_path = (
    "abfss://gold@stdataeng001.dfs.core.windows.net/customer_sales/"
)

(
    gold_df.write
    .format("delta")
    .mode("overwrite")
    .save(gold_path)
)
```

Typical responsibilities:
- Business aggregations.
- Reporting datasets.
- KPI calculations.
- Analytics-ready tables.

## 9. Delta Lake on ADLS Gen2

Delta Lake stores data files in Parquet format and maintains a transaction log in the `_delta_log` directory.

Example directory:

```text
gold/
  sales/
    part-00000-....parquet
    part-00001-....parquet
    _delta_log/
      00000000000000000000.json
      00000000000000000001.json
```

### Why use Delta Lake?

| Feature | Benefit |
|---|---|
| ACID transactions | Reliable table changes |
| Schema enforcement | Helps prevent incompatible writes |
| Schema evolution | Supports approved schema changes |
| Time Travel | Query eligible previous versions |
| MERGE | Supports inserts, updates, and deletes |
| Transaction log | Tracks table changes |

### Example: Delta MERGE

Suppose the source contains updated customer records.

```python
from delta.tables import DeltaTable

target_path = (
    "abfss://silver@stdataeng001.dfs.core.windows.net/customers/"
)

delta_table = DeltaTable.forPath(spark, target_path)

(
    delta_table.alias("target")
    .merge(
        source_df.alias("source"),
        "target.customer_id = source.customer_id"
    )
    .whenMatchedUpdateAll()
    .whenNotMatchedInsertAll()
    .execute()
)
```

Before running a MERGE, make sure the source does not contain multiple ambiguous rows for the same target key. Deduplicate or otherwise resolve multiple matches according to the business rules.

## 10. Unity Catalog and External Locations

In a Unity Catalog-enabled workspace, external locations provide a governed way to access cloud storage.

### Basic relationship

```text
Storage Account
      |
      v
ADLS Gen2 Path
      |
      v
Storage Credential
      |
      v
External Location
      |
      v
Unity Catalog Permissions
      |
      v
Databricks Users and Groups
```

### Important terms

| Term | Meaning |
|---|---|
| Storage credential | Defines the identity used to access cloud storage |
| External location | Connects a cloud storage path with a storage credential |
| External table | A Unity Catalog table whose data is stored at an external location |
| Managed table | A Unity Catalog table whose storage lifecycle is managed by Unity Catalog |

An external location is not the same as an external table. The external location governs access to a storage path; tables can be created within that governed path.

### Example SQL

The following is illustrative. The storage credential must already exist, and the executing principal must have the required privileges.

```sql
CREATE EXTERNAL LOCATION bronze_location
URL 'abfss://bronze@stdataeng001.dfs.core.windows.net/'
WITH (STORAGE CREDENTIAL bronze_credential);
```

Grant access:

```sql
GRANT READ FILES
ON EXTERNAL LOCATION bronze_location
TO `data_engineers`;
```

Actual privileges and supported operations depend on the object type and workspace configuration.

## 11. Databricks Access Approaches: Legacy vs Unity Catalog

Older Databricks workspaces may use direct storage configuration or mounts. Modern deployments commonly prefer Unity Catalog external locations.

| Approach | Description |
|---|---|
| Direct storage access | Access ADLS paths using configured credentials |
| DBFS mount | Exposes cloud storage through a mounted path |
| Unity Catalog external location | Governed access using storage credentials and permissions |

### DBFS mounts

A mount maps external storage to a path such as:

```text
/mnt/bronze
```

Then a notebook may access it using:

```python
df = spark.read.parquet("/mnt/bronze/orders/")
```

Mounts are a legacy access pattern in many modern Databricks architectures. They are not the preferred approach for new Unity Catalog deployments.

Do not assume that a mount automatically provides centralized Unity Catalog governance.

## 12. Common Integration Problems

| Error or symptom | Possible reason | What to check |
|---|---|---|
| 403 Forbidden | Missing authorization | RBAC, ACLs, Unity Catalog privileges |
| Authentication failed | Invalid identity or credentials | Tenant, client ID, secret, identity configuration |
| Path not found | Incorrect path | Container, account, directory, filename |
| DNS resolution failure | Network or DNS issue | Private DNS, VNet, endpoint |
| Connection timeout | Network restrictions | Firewall, NSG, private endpoint, routing |
| Cannot list directory | Missing directory permissions | Execute and read permissions |
| Delta table not found | Wrong path or missing Delta log | Table location and `_delta_log` |
| Write operation fails | Missing write permissions or conflicting operation | RBAC, ACL, table permissions, concurrent writes |

### Troubleshooting sequence

1. Verify the storage account name.
2. Verify the container and directory path.
3. Check whether the identity can authenticate.
4. Check the required data-plane permissions.
5. Verify ACL traversal permissions for parent directories.
6. Check network connectivity and firewall rules.
7. Confirm the correct URI scheme and endpoint.
8. Review Databricks and Azure diagnostic logs.

A private endpoint solves a network connectivity requirement; it does not grant storage data permissions.

## 13. Security Best Practices

- Prefer managed identities or Unity Catalog storage credentials where supported.
- Avoid hardcoding client secrets, account keys, or SAS tokens in notebooks.
- Use secret management when credentials are required.
- Apply least-privilege access.
- Separate access between Bronze, Silver, and Gold where appropriate.
- Use Unity Catalog for centralized governance.
- Restrict public network access when the architecture requires private connectivity.
- Rotate credentials when using secrets.
- Audit access and monitor failed authentication or authorization attempts.
- Avoid granting broad storage permissions to every workspace user.

## 14. Practical End-to-End Example

**Scenario:** A company receives daily order files in ADLS Gen2. Databricks must clean the data and produce a customer sales summary.

### Step 1: Read raw orders

```python
raw_path = (
    "abfss://bronze@stdataeng001.dfs.core.windows.net/orders/"
)

orders_df = (
    spark.read
    .option("header", "true")
    .option("inferSchema", "true")
    .csv(raw_path)
)
```

### Step 2: Clean the data

```python
from pyspark.sql.functions import col

clean_df = (
    orders_df
    .filter(col("order_id").isNotNull())
    .filter(col("customer_id").isNotNull())
    .dropDuplicates(["order_id"])
)
```

### Step 3: Write cleaned data to Silver

```python
silver_path = (
    "abfss://silver@stdataeng001.dfs.core.windows.net/orders/"
)

(
    clean_df.write
    .format("delta")
    .mode("overwrite")
    .save(silver_path)
)
```

### Step 4: Read Silver data

```python
silver_df = spark.read.format("delta").load(silver_path)
```

### Step 5: Create customer-level aggregation

```python
from pyspark.sql.functions import sum

customer_sales = (
    silver_df
    .groupBy("customer_id")
    .agg(sum("amount").alias("total_sales"))
)
```

### Step 6: Write Gold data

```python
gold_path = (
    "abfss://gold@stdataeng001.dfs.core.windows.net/customer_sales/"
)

(
    customer_sales.write
    .format("delta")
    .mode("overwrite")
    .save(gold_path)
)
```

### Final architecture

```text
Daily Order Files
       |
       v
ADLS Gen2 - Bronze
       |
       v
Databricks - Read and Clean
       |
       v
ADLS Gen2 - Silver (Delta)
       |
       v
Databricks - Aggregate
       |
       v
ADLS Gen2 - Gold (Delta)
       |
       v
Power BI / Analytics
```

For a production pipeline, consider incremental processing, idempotent writes, data quality checks, audit columns, and recovery handling rather than blindly overwriting the complete target each run.

## 15. Common Interview Questions and Answers

### Q1. How does Azure Databricks connect to ADLS Gen2?

Databricks uses the ABFS connector and an authentication mechanism such as a service principal, managed identity, or Unity Catalog storage credential. The identity must have the required storage permissions.

### Q2. What is the `abfss://` protocol?

It is the secure Hadoop-compatible filesystem URI scheme used to access Azure Data Lake Storage through the ABFS connector.

### Q3. Does `abfss://` provide authentication?

No. It identifies the storage access protocol and requests secure transport. Authentication and authorization must be configured separately.

### Q4. What is the difference between authentication and authorization?

Authentication verifies the identity. Authorization determines which resources and operations that identity can access.

### Q5. What is a service principal?

A service principal is an application identity in Microsoft Entra ID. It can authenticate applications and services to Azure resources.

### Q6. What is a managed identity?

A managed identity is an Azure-managed identity that allows supported resources to authenticate without managing application secrets directly.

### Q7. Does a managed identity automatically access ADLS Gen2?

No. It must be granted the required data-plane RBAC roles or ACL permissions.

### Q8. What is the difference between ADLS Gen2 and Delta Lake?

ADLS Gen2 is cloud storage. Delta Lake is a table storage framework that uses Parquet files and a transaction log to provide features such as ACID transactions and MERGE.

### Q9. Why use Delta instead of plain Parquet?

Delta provides transaction management, schema enforcement, schema evolution, time travel, and reliable updates and deletes through supported operations.

### Q10. What is Unity Catalog's role in ADLS integration?

Unity Catalog centralizes governance. Storage credentials define authentication, external locations govern storage paths, and privileges control access.

### Q11. What is the difference between an external location and an external table?

An external location connects a cloud storage path to a storage credential and governs access. An external table is a table whose data resides at an external storage location.

### Q12. What is the difference between RBAC and ACL in ADLS Gen2?

RBAC provides role-based access at Azure resource scopes. ACLs provide fine-grained file and directory permissions, including read, write, and execute.

### Q13. What causes a 403 error while reading ADLS data?

Common causes include missing data-plane permissions, insufficient ACL permissions, incorrect identity configuration, or missing Unity Catalog privileges.

### Q14. What is the difference between a private endpoint and storage permissions?

A private endpoint provides private network connectivity to a service. Storage permissions determine whether an authenticated identity can read or write data.

### Q15. How would you design a Databricks and ADLS Gen2 pipeline?

I would store raw data in Bronze, use Databricks to validate and transform it into Silver, and create business-ready Gold datasets. I would use Delta Lake for reliable table operations, Unity Catalog for governance, and monitoring and recovery mechanisms for production reliability.

## 16. Common Mistakes to Avoid

- Assuming that `abfss://` automatically authenticates the user.
- Confusing Azure management-plane roles with storage data-plane roles.
- Assuming a managed identity automatically has storage access.
- Treating ADLS Gen2 and Delta Lake as the same technology.
- Using account keys or secrets directly in notebooks.
- Assuming private endpoint connectivity grants data access.
- Using overwrite mode without understanding its impact.
- Assuming Spark writes a single output file.
- Confusing Unity Catalog external locations with external tables.
- Assuming legacy DBFS mounts are the preferred modern governance approach.
- Forgetting to deduplicate source records before a Delta MERGE.

## 17. Quick Revision

| Concept | Remember |
|---|---|
| ADLS Gen2 | Cloud data storage |
| Azure Databricks | Distributed data processing |
| Delta Lake | Reliable table framework on Parquet |
| `abfss://` | Secure ABFS filesystem URI |
| Service principal | Application identity |
| Managed identity | Azure-managed identity |
| RBAC | Role-based resource/data permissions |
| ACL | Fine-grained file and directory permissions |
| Unity Catalog | Centralized Databricks governance |
| Storage credential | Defines cloud authentication |
| External location | Governed cloud storage path |
| Bronze | Raw data |
| Silver | Cleaned data |
| Gold | Business-ready data |

## 18. Self-Assessment

Try answering these without looking at the notes:

1. Explain how Databricks reads data from ADLS Gen2.
2. What does `abfss://` represent?
3. Why are authentication and authorization separate?
4. Compare service principals and managed identities.
5. What permissions are required to read a file in ADLS Gen2?
6. Explain the role of Unity Catalog in storage integration.
7. Differentiate an external location from an external table.
8. Explain how Delta Lake stores and manages table changes.
9. How would you troubleshoot a 403 error?
10. Design a Bronze-to-Silver-to-Gold pipeline using Databricks and ADLS Gen2.

**Interview readiness target:** You should be able to explain the complete flow from identity authentication and storage permissions to reading, transforming, and writing data, without confusing storage, processing, and governance responsibilities.