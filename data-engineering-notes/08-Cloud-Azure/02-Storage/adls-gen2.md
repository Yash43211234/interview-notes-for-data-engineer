# Azure Data Lake Storage Gen2 (ADLS Gen2)

## 1. What Is Azure Data Lake Storage Gen2?

Azure Data Lake Storage Gen2 (ADLS Gen2) is Microsoft's cloud-based data lake storage service built on Azure Blob Storage.

It is designed to store and manage large amounts of structured, semi-structured, and unstructured data for analytics and data engineering workloads.

It supports:

- Large-scale data storage.
- Hierarchical directory structures.
- Big data analytics.
- Fine-grained access control.
- Integration with Azure Data Factory, Azure Databricks, and other analytics services.

### Example

A company receives sales data from multiple sources.

It stores the data in ADLS Gen2:

```text
company-data/
    |
    +---- raw/
    |       |
    |       +---- sales/
    |       +---- customers/
    |
    +---- processed/
    |       |
    |       +---- sales/
    |       +---- customers/
    |
    +---- curated/
            |
            +---- sales_summary/
            +---- customer_analytics/
```

- `raw/`: Original ingested data.
- `processed/`: Cleaned and transformed data.
- `curated/`: Business-ready data for reporting and analytics.

This structure is commonly used in a medallion-style architecture.

---

## 2. ADLS Gen2 Architecture

ADLS Gen2 combines Azure Blob Storage capabilities with a hierarchical namespace.

Its foundation is Azure Blob Storage, while hierarchical namespace provides directory and file-system-like capabilities.

### Main Components

1. Storage account.
2. Container (also called a filesystem in ADLS Gen2 contexts).
3. Directories.
4. Files.
5. Hierarchical namespace.
6. Access control.

### Simplified Structure

```text
Storage Account
    |
    v
Container / Filesystem
    |
    +---- raw/
    |       |
    |       +---- orders/
    |       +---- customers/
    |
    +---- processed/
    |
    +---- curated/
```

### Important Terms

| Component | Meaning |
|---|---|
| Storage account | Top-level Azure Storage resource |
| Container | Logical grouping of blobs |
| Filesystem | Common ADLS Gen2 term for a container |
| Directory | Folder-like structure |
| File | Stored data object |
| Hierarchical namespace | Enables directory and file-system semantics |

---

## 3. What Is Hierarchical Namespace (HNS)?

Hierarchical Namespace is a capability that organizes data into directories and files instead of treating every object only as an independent blob name.

Without HNS, Blob Storage can represent folders using virtual directory names in blob paths.

With HNS enabled, directories are managed as part of the storage namespace.

### Example

Without HNS, an object might have a name such as:

```text
raw/sales/2026/orders.csv
```

The slash-separated segments represent a virtual folder structure.

With HNS enabled, the directory structure is managed hierarchically, enabling file-system-style operations and directory-level access control.

### Benefits of HNS

- Directory-level organization.
- Efficient directory operations.
- Fine-grained ACL permissions.
- Better compatibility with big data analytics workloads.
- File-system-style operations such as rename and directory management.

**Important:** Hierarchical Namespace is enabled at the storage account level. It is not a setting that can be independently enabled for one container.

---

## 4. ADLS Gen2 vs Azure Blob Storage

ADLS Gen2 is built on Blob Storage, but it adds capabilities for analytics workloads.

| Azure Blob Storage | ADLS Gen2 |
|---|---|
| Object storage service | Blob Storage with hierarchical namespace |
| Supports object-based organization | Supports directory and file-system semantics |
| Uses containers and blobs | Uses containers/filesystems, directories, and files |
| Supports Azure RBAC and SAS | Supports RBAC, SAS, and ACL-based access control |
| Suitable for general object storage | Particularly useful for analytics and data lake workloads |

**Interview point:** ADLS Gen2 is not a completely separate storage engine from Blob Storage. It is Blob Storage with hierarchical namespace capabilities.

---

## 5. Storage Account and Container

### Storage Account

A storage account provides the namespace and configuration boundary for Azure Storage resources.

It defines important settings such as:

- Storage redundancy.
- Networking access.
- Security configuration.
- Data protection options.
- Hierarchical namespace support.

Example:

```text
Storage Account: stcompanydata
```

### Container / Filesystem

A container is a logical grouping of blobs inside a storage account.

In ADLS Gen2, the term filesystem is commonly used for the container.

Example:

```text
stcompanydata
    |
    +---- company-data
    |
    +---- backup-data
```

The container can hold directories and files.

---

## 6. Common Data Lake Folder Structure

A data engineering platform often organizes data into layers.

```text
company-data/
    |
    +---- raw/
    |       |
    |       +---- source_system_1/
    |       +---- source_system_2/
    |
    +---- processed/
    |       |
    |       +---- customers/
    |       +---- orders/
    |
    +---- curated/
            |
            +---- sales/
            +---- finance/
```

### Raw Layer

Stores source data with minimal modification.

Purpose:

- Preserve source data.
- Support reprocessing.
- Maintain an ingestion history.

### Processed Layer

Contains validated, cleaned, and transformed data.

Purpose:

- Handle null values and invalid records.
- Standardize formats.
- Remove or manage duplicates.
- Apply business transformations.

### Curated Layer

Contains business-ready datasets.

Purpose:

- Support analytics and reporting.
- Provide trusted data to downstream consumers.
- Store aggregated or modeled datasets.

The exact layer names and responsibilities depend on the organization's architecture.

---

## 7. File Formats Used in ADLS Gen2

ADLS Gen2 can store many file formats.

| Format | Type | Common Use |
|---|---|---|
| CSV | Text-based, tabular | Simple data exchange |
| JSON | Semi-structured | API and event data |
| XML | Semi-structured | Document and system integration |
| Parquet | Columnar | Analytics and big data processing |
| Avro | Row-oriented binary | Data exchange and event pipelines |
| ORC | Columnar | Analytical workloads |

### Why Is Parquet Common in Data Lakes?

Parquet is a columnar file format.

Benefits:

- Reads only required columns when supported by the query engine.
- Supports compression and encoding.
- Stores schema information.
- Works well with analytical queries.

Example:

```text
curated/sales/
    |
    +---- part-00000.parquet
    +---- part-00001.parquet
    +---- part-00002.parquet
```

Spark commonly writes output as a directory containing multiple part files.

**Important:** ADLS Gen2 stores the files. Spark, Databricks, or another processing engine reads and transforms them.

---

## 8. Access Control in ADLS Gen2

ADLS Gen2 supports multiple authorization mechanisms.

### 8.1 Azure RBAC

Role-Based Access Control grants permissions at Azure resource scopes.

Common data roles:

- Storage Blob Data Reader.
- Storage Blob Data Contributor.
- Storage Blob Data Owner.

### 8.2 Access Control Lists (ACLs)

ACLs provide granular permissions for directories and files.

Basic permissions:

| Permission | Meaning |
|---|---|
| Read (`r`) | Read file contents or list directory contents |
| Write (`w`) | Modify files or create/delete directory entries, as applicable |
| Execute (`x`) | Traverse directories |

### Example

```text
company-data/
    |
    +---- raw/
    |
    +---- processed/
    |
    +---- curated/
```

Possible permissions:

- Ingestion pipeline: write to `raw/`.
- Transformation pipeline: read `raw/`, write `processed/`.
- Analytics team: read `curated/`.

### Important

- RBAC provides role-based permissions at resource scopes.
- ACLs provide path-level permissions.
- Broad data-plane RBAC permissions can grant access independently of restrictive ACL entries.
- Parent-directory execute permissions are required to traverse the directory path.
- Default ACLs affect newly created children, not existing files and directories.

For detailed RBAC and ACL concepts, see `access-control-and-permissions.md`.

---

## 9. How to Access ADLS Gen2

Data engineering services can access ADLS Gen2 using different authentication mechanisms.

Common options include:

1. Managed Identity.
2. Service Principal with Microsoft Entra ID.
3. User delegation SAS.
4. Account key or connection string where required.

### Managed Identity

Managed Identity is often preferred for Azure-to-Azure service authentication because it avoids storing credentials in application code.

Example:

```text
Azure Data Factory
        |
        v
Managed Identity
        |
        v
Microsoft Entra ID
        |
        v
Azure Storage
        |
        v
ADLS Gen2
```

The identity must also receive the necessary storage permissions.

**Important:** Successful authentication does not automatically grant data access.

---

## 10. Access ADLS Gen2 from Azure Databricks

Databricks can read and write ADLS Gen2 data.

A common URI format is:

```text
abfss://<container>@<storage-account>.dfs.core.windows.net/<path>
```

Example:

```python
source_path = (
    "abfss://company-data@stcompanydata.dfs.core.windows.net/raw/orders/"
)

df = spark.read.format("parquet").load(source_path)

df.display()
```

### Write Data

```python
target_path = (
    "abfss://company-data@stcompanydata.dfs.core.windows.net/processed/orders/"
)

df.write \
    .format("parquet") \
    .mode("overwrite") \
    .save(target_path)
```

### Understanding the URI

| Part | Meaning |
|---|---|
| `abfss://` | Secure ADLS Gen2 filesystem URI scheme |
| `company-data` | Container/filesystem name |
| `stcompanydata` | Storage account name |
| `dfs.core.windows.net` | ADLS Gen2 DFS endpoint |
| `/raw/orders/` | Directory path |

### Why Use `abfss://`?

`abfss://` is the secure filesystem URI scheme commonly used by Hadoop-compatible clients to access ADLS Gen2 over HTTPS.

**Important:** The URI does not authenticate the caller. Databricks must also have a valid authentication configuration and the required storage permissions.

---

## 11. ADLS Gen2 Endpoints

Azure Storage provides different endpoints for different operations.

Common endpoint formats:

```text
Blob endpoint:
https://<storage-account>.blob.core.windows.net

DFS endpoint:
https://<storage-account>.dfs.core.windows.net
```

### Blob Endpoint

Used for Blob Storage operations.

### DFS Endpoint

Used for Data Lake Storage Gen2 filesystem operations.

It supports hierarchical namespace operations such as directory and file management.

**Important:** Both endpoints belong to the same storage account. They do not represent two separate copies of the data.

---

## 12. Storage Redundancy

Azure Storage offers different redundancy options to improve data durability and availability.

| Option | Meaning |
|---|---|
| LRS | Locally redundant storage |
| ZRS | Zone-redundant storage |
| GRS | Geo-redundant storage |
| RA-GRS | Read-access geo-redundant storage |
| GZRS | Geo-zone-redundant storage |
| RA-GZRS | Read-access geo-zone-redundant storage |

### Basic Understanding

- LRS: Multiple copies at one physical location.
- ZRS: Replication across availability zones in the primary region.
- GRS: Local redundancy plus asynchronous replication to a secondary region.
- RA-GRS: GRS with read access to the secondary region.
- GZRS: Zone redundancy in the primary region plus geo-replication.
- RA-GZRS: GZRS with read access to the secondary region.

The availability of redundancy options depends on storage account configuration, region, and service support.

---

## 13. Lifecycle Management

Azure Storage lifecycle management helps manage data based on age or other supported conditions.

For example, a company may:

- Keep frequently accessed data in a hot tier.
- Move older data to a cooler tier.
- Archive data that is rarely accessed.
- Delete data after a retention period.

### Common Blob Access Tiers

| Tier | Typical Use |
|---|---|
| Hot | Frequently accessed data |
| Cool | Infrequently accessed data |
| Cold | Rarely accessed data with longer retention |
| Archive | Offline data that may require rehydration before access |

Not all account types, regions, and operations support every tier in the same way.

### Example

```text
New Data
   |
   v
Hot Tier
   |
   v
Cool Tier
   |
   v
Cold Tier
   |
   v
Archive Tier
```

This is an illustrative lifecycle, not an automatic sequence. Lifecycle policies must be configured, and tiering eligibility and access costs should be considered.

**Important:** Archive data is offline and cannot be read directly like online hot, cool, or cold data. It must be rehydrated before normal access.

---

## 14. ADLS Gen2 Integration with Data Engineering Services

ADLS Gen2 is commonly used as the storage layer in Azure data engineering architectures.

### Azure Data Factory

ADF can:

- Copy data from source systems into ADLS Gen2.
- Orchestrate ingestion pipelines.
- Move data between storage locations.
- Trigger downstream processing.

### Azure Databricks

Databricks can:

- Read raw files.
- Clean and transform data using PySpark.
- Write processed datasets.
- Create Delta tables.
- Perform batch and streaming processing.

### Azure Synapse Analytics

Synapse can:

- Query data in supported lake formats.
- Run analytical SQL workloads.
- Integrate lake data with analytics solutions.

### Simplified Architecture

```text
Source Systems
      |
      v
Azure Data Factory
      |
      v
ADLS Gen2 - Raw
      |
      v
Azure Databricks
      |
      v
ADLS Gen2 - Processed
      |
      v
ADLS Gen2 - Curated
      |
      v
Synapse / BI / Analytics
```

---

## 15. ADLS Gen2 vs Delta Lake

These technologies are related but serve different purposes.

| ADLS Gen2 | Delta Lake |
|---|---|
| Cloud storage service | Storage/table format and transaction framework |
| Stores files and directories | Organizes data files as reliable tables |
| Provides storage and access control | Provides ACID transactions and table-level capabilities |
| Can store CSV, JSON, Parquet, and other files | Commonly stores data in Parquet with a transaction log |
| Does not independently provide Delta table transactions | Uses a transaction log to manage table versions and changes |

### Example

ADLS Gen2 can store a Delta table directory:

```text
curated/sales_delta/
    |
    +---- _delta_log/
    |
    +---- part-00000-....parquet
    +---- part-00001-....parquet
```

- Parquet files contain the data.
- `_delta_log` contains transaction and table metadata.
- Delta Lake provides table-level capabilities such as ACID transactions and time travel.

**Important:** ADLS Gen2 is the storage service. Delta Lake is the table format and transaction layer built on top of storage.

---

## 16. Common ADLS Gen2 Problems

### 16.1 Authentication Failure

Possible causes:

- Invalid credentials.
- Incorrect service principal configuration.
- Managed identity not configured.
- Expired or invalid token.

### 16.2 Authorization Failure

Possible causes:

- Missing Storage Blob Data Reader or Contributor role.
- Insufficient ACL permissions.
- Missing execute permission on a parent directory.
- Incorrect RBAC scope.

### 16.3 Network Connectivity Failure

Possible causes:

- Storage firewall restrictions.
- Public network access disabled.
- Private endpoint configuration issue.
- Private DNS resolution issue.
- Missing network connectivity from the compute environment.

### 16.4 File Not Found

Possible causes:

- Incorrect storage account.
- Incorrect container/filesystem.
- Wrong directory or file name.
- Incorrect endpoint or path.

### 16.5 Slow Data Processing

Possible causes:

- Too many small files.
- Poor partitioning strategy.
- Reading unnecessary columns.
- Inefficient file format.
- Too many unnecessary data scans.
- Insufficient or poorly configured compute resources.

### Troubleshooting Checklist

1. Verify the storage account name.
2. Verify the container/filesystem name.
3. Check the DFS endpoint and file path.
4. Confirm the identity used by the workload.
5. Verify RBAC and ACL permissions.
6. Check storage firewall and private networking.
7. Check file existence and format.
8. Inspect Spark logs and execution metrics if processing is slow.

---

## 17. Common Interview Questions

### Q1. What is ADLS Gen2?

ADLS Gen2 is Azure's data lake storage capability built on Blob Storage with hierarchical namespace support. It is designed for large-scale analytics and data engineering workloads.

### Q2. What is hierarchical namespace?

It provides directory and file-system semantics, including efficient directory operations and ACL-based access control.

### Q3. Is ADLS Gen2 a separate storage service from Blob Storage?

It is built on Azure Blob Storage. Enabling hierarchical namespace adds data lake capabilities.

### Q4. What is the difference between a container and a directory?

A container is a logical grouping of blobs. A directory organizes files and subdirectories within a hierarchical namespace.

### Q5. What is the difference between the Blob and DFS endpoints?

The Blob endpoint supports Blob Storage operations. The DFS endpoint supports ADLS Gen2 filesystem operations.

### Q6. What is the `abfss://` URI?

It is a secure filesystem URI scheme commonly used by Hadoop-compatible clients to access ADLS Gen2.

### Q7. How does Databricks connect to ADLS Gen2?

Databricks uses a supported authentication mechanism, such as Managed Identity or a service principal, and accesses storage using an appropriate URI and permissions.

### Q8. Does Managed Identity automatically grant access to ADLS Gen2?

No. The identity must be assigned appropriate RBAC roles or ACL permissions.

### Q9. What is the difference between ADLS Gen2 and Delta Lake?

ADLS Gen2 provides cloud storage. Delta Lake adds table-level transaction and reliability capabilities on top of data files stored in the lake.

### Q10. Why is ADLS Gen2 useful for data engineering?

It provides scalable storage, hierarchical organization, fine-grained access control, and integration with Azure data processing services.

### Q11. What is the difference between LRS and ZRS?

LRS maintains multiple copies at one physical location. ZRS replicates data across availability zones in the primary region.

### Q12. What is the purpose of lifecycle management?

It automates supported data-tiering and deletion actions based on configured rules, helping manage storage costs and retention.

### Q13. How would you organize a data lake?

I would separate raw, processed, and curated datasets, organize them by source or business domain, and apply appropriate access controls and retention policies.

### Q14. How would you troubleshoot an ADLS Gen2 access error?

I would verify the identity, authentication method, RBAC role, ACL permissions, parent-directory traversal, storage network rules, and requested path.

---

## 18. Common Mistakes to Avoid

- Thinking ADLS Gen2 is completely separate from Blob Storage.
- Confusing containers with directories.
- Assuming hierarchical namespace is enabled separately for each container.
- Assuming a storage URI provides authentication.
- Assuming Managed Identity automatically grants permissions.
- Confusing the DFS endpoint with the Blob endpoint.
- Confusing ADLS Gen2 with Delta Lake.
- Assuming every file in a data lake must be Parquet.
- Assuming lifecycle management automatically moves data through every tier.
- Forgetting that archive data must be rehydrated before normal access.
- Ignoring storage networking when troubleshooting authorization errors.
- Treating storage redundancy as a complete disaster recovery solution.

---

## 19. Quick Revision

| Topic | Key Point |
|---|---|
| ADLS Gen2 | Azure data lake storage capability |
| Foundation | Azure Blob Storage |
| HNS | Hierarchical Namespace |
| Storage account | Top-level storage resource |
| Container | Logical grouping of blobs |
| Filesystem | Common ADLS Gen2 term for a container |
| Directory | Folder-like structure |
| DFS endpoint | Filesystem operations |
| Blob endpoint | Blob Storage operations |
| `abfss://` | Secure ADLS Gen2 filesystem URI |
| RBAC | Role-based resource access |
| ACL | Granular directory/file permissions |
| Raw | Source data |
| Processed | Cleaned and transformed data |
| Curated | Business-ready data |
| LRS | Local redundancy |
| ZRS | Zone redundancy |
| GRS | Local plus geo-redundancy |
| Lifecycle management | Policy-based data tiering and deletion |
| Delta Lake | Table and transaction layer on storage |

---

## 20. Self-Assessment

Try answering these questions without looking at the notes:

1. What is ADLS Gen2?
2. Explain its architecture.
3. What is Hierarchical Namespace?
4. How is ADLS Gen2 related to Blob Storage?
5. Explain storage account, container, directory, and file.
6. What is the difference between Blob and DFS endpoints?
7. What is the purpose of `abfss://`?
8. How can Databricks access ADLS Gen2?
9. What is the difference between RBAC and ACL?
10. How would you organize raw, processed, and curated data?
11. Explain LRS, ZRS, GRS, and GZRS.
12. What is storage lifecycle management?
13. What is the difference between ADLS Gen2 and Delta Lake?
14. How would you troubleshoot an authorization failure?
15. Design a simple Azure data lake architecture using ADF, ADLS Gen2, and Databricks.

**Interview readiness:** You should be able to explain ADLS Gen2 architecture, hierarchical namespace, storage endpoints, authentication, access control, file formats, storage redundancy, and its role in an Azure data engineering pipeline.