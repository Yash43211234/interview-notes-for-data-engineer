# Azure Blob Storage

## 1. What Is Azure Blob Storage?

Azure Blob Storage is Microsoft's object storage service for storing large amounts of unstructured data.

A blob (Binary Large Object) is an individual object stored in a container.

It is commonly used to store:

- Images and videos.
- Documents and backups.
- CSV, JSON, and XML files.
- Parquet and Avro files.
- Application logs.
- Data lake files.
- Media and archive data.

### Example

A company receives customer and transaction data from different systems.

It can store these files in Azure Blob Storage:

```text
Storage Account
    |
    v
Container: company-data
    |
    +---- customers/
    |       +---- customers.csv
    |
    +---- transactions/
    |       +---- transactions.json
    |
    +---- reports/
            +---- monthly-report.pdf
```

Blob Storage provides scalable storage without requiring the application to manage physical storage infrastructure.

---

## 2. Azure Blob Storage Architecture

The basic hierarchy consists of:

1. Storage account.
2. Container.
3. Blob.

```text
Azure Storage
    |
    v
Storage Account
    |
    +---- Container: raw-data
    |         |
    |         +---- customers.csv
    |         +---- orders.json
    |
    +---- Container: backups
              |
              +---- backup-01.zip
              +---- backup-02.zip
```

### Storage Account

A storage account is the top-level Azure resource that provides the namespace and configuration for stored data.

It controls settings such as:

- Storage redundancy.
- Network access.
- Security.
- Data protection.
- Performance and access configuration.

### Container

A container is a logical grouping of blobs inside a storage account.

It helps organize data and manage access.

### Blob

A blob is an individual object stored inside a container.

It contains the data and associated properties and metadata.

---

## 3. Types of Blobs

Azure Blob Storage supports three main blob types.

### 3.1 Block Blob

Block blobs are designed for storing general files and large objects.

Common examples:

- CSV files.
- JSON files.
- Parquet files.
- Images.
- Videos.
- Documents.

Block blobs are uploaded as blocks and committed to form the final blob.

They are commonly used in data engineering pipelines.

### 3.2 Append Blob

Append blobs are optimized for append operations.

New data blocks can be appended to the end of the blob.

Common examples:

- Application logs.
- Event logs.
- Sequential logging workloads.

They are useful when data is continuously appended rather than frequently overwritten.

### 3.3 Page Blob

Page blobs support random read and write operations over fixed-size pages.

They are commonly used for scenarios such as Azure virtual machine disks.

### Comparison

| Blob Type | Main Purpose | Example |
|---|---|---|
| Block Blob | General object and file storage | CSV, Parquet, images |
| Append Blob | Append-oriented workloads | Logs |
| Page Blob | Random read/write access | Virtual machine disks |

**Interview point:** Block blobs are commonly used for data lake files, while page blobs are associated with disk workloads.

---

## 4. Blob Storage vs ADLS Gen2

ADLS Gen2 is built on Azure Blob Storage with hierarchical namespace enabled.

Both provide cloud object storage, but their capabilities and typical use cases differ.

| Blob Storage without HNS | ADLS Gen2 with HNS |
|---|---|
| Object storage model | Object storage with hierarchical namespace |
| Containers and blobs | Containers/filesystems, directories, and files |
| Folder-like paths are generally virtual | Directories are managed hierarchically |
| Supports Azure RBAC and SAS | Supports RBAC, SAS, and path-level ACLs |
| Suitable for general object storage | Particularly useful for analytics and data lake workloads |

### Example

Without hierarchical namespace, this blob name can represent a folder-like path:

```text
raw/sales/2026/orders.csv
```

The path segments are part of the blob name.

With hierarchical namespace enabled, directories are managed as part of the filesystem namespace.

**Important:** ADLS Gen2 is not a completely separate storage engine. It adds hierarchical namespace capabilities to Blob Storage.

---

## 5. Blob Access Tiers

Azure Blob Storage provides access tiers to help optimize storage costs based on how frequently data is accessed.

Common access tiers:

1. Hot.
2. Cool.
3. Cold.
4. Archive.

### 5.1 Hot Tier

Designed for frequently accessed data.

Examples:

- Active application data.
- Frequently accessed reports.
- Current pipeline input files.

### 5.2 Cool Tier

Designed for infrequently accessed data that is still expected to be accessed occasionally.

Examples:

- Older reports.
- Historical datasets.
- Data retained for short-term reference.

### 5.3 Cold Tier

Designed for rarely accessed data that needs to remain online.

Examples:

- Older historical datasets.
- Data retained for compliance or occasional analysis.

### 5.4 Archive Tier

Designed for rarely accessed data that can tolerate offline storage and retrieval delays.

Examples:

- Long-term backups.
- Historical records.
- Compliance archives.

Archive data must be rehydrated before normal access.

### Comparison

| Tier | Access Pattern | Availability |
|---|---|---|
| Hot | Frequent | Online |
| Cool | Infrequent | Online |
| Cold | Rare | Online |
| Archive | Very rare | Offline; rehydration required |

### Important Cost Considerations

Storage cost is not the only cost.

Consider:

- Storage capacity charges.
- Read and write transaction charges.
- Data retrieval charges.
- Early deletion charges where applicable.
- Data transfer costs.
- Archive rehydration costs.

Cool, cold, and archive tiers generally have minimum retention periods. Deleting or moving data before the applicable period can result in early deletion charges.

Tier availability and pricing depend on the account type, region, redundancy configuration, and current Azure pricing.

---

## 6. Blob Storage Access Methods

Applications and services can access Blob Storage using several methods.

### 6.1 Microsoft Entra ID

Microsoft Entra ID provides identity-based authentication.

A user, application, or managed identity authenticates and then receives authorization through appropriate roles or other supported mechanisms.

### 6.2 Azure RBAC

Role-Based Access Control grants permissions at Azure resource scopes.

Common data roles:

| Role | Purpose |
|---|---|
| Storage Blob Data Reader | Read and list blob data |
| Storage Blob Data Contributor | Read, write, and delete blob data |
| Storage Blob Data Owner | Broad data access, including managing ownership and ACLs |

**Important:** The general `Reader` role provides management-plane read access. It does not automatically grant permission to read blob contents.

### 6.3 Shared Access Signature (SAS)

A SAS provides delegated access with specified permissions and validity constraints.

Example:

A company wants to share a report with an external application for a limited period.

It can generate a read-only SAS token with an expiry time.

### 6.4 Storage Account Keys

Storage account keys provide broad Shared Key access to the storage account.

They should be protected carefully and should not be hardcoded into application code.

For detailed authorization concepts, see `access-control-and-permissions.md`.

---

## 7. Types of SAS

Azure Storage supports three common SAS types.

| SAS Type | Description |
|---|---|
| User delegation SAS | Signed using a user delegation key obtained through Microsoft Entra ID |
| Service SAS | Provides delegated access to a specific storage service resource |
| Account SAS | Provides delegated access to supported services and resource types within a storage account |

### SAS Best Practices

- Grant only the required permissions.
- Set an appropriate expiry time.
- Use HTTPS.
- Apply IP restrictions where appropriate.
- Avoid sharing tokens publicly.
- Prefer user delegation SAS when supported and suitable.

A SAS token is a bearer credential. Anyone who obtains it may use it within its permissions and validity period.

---

## 8. Blob Storage Endpoints

A Blob Storage endpoint commonly follows this format:

```text
https://<storage-account>.blob.core.windows.net
```

Example:

```text
https://stcompanydata.blob.core.windows.net
```

A blob URL can include the container and blob name:

```text
https://stcompanydata.blob.core.windows.net/company-data/raw/orders.csv
```

### Understanding the URL

| Part | Meaning |
|---|---|
| `https://` | Secure HTTP protocol |
| `stcompanydata` | Storage account name |
| `blob.core.windows.net` | Blob Storage endpoint |
| `company-data` | Container |
| `raw/orders.csv` | Blob name |

When hierarchical namespace is enabled, the DFS endpoint is also available for filesystem operations:

```text
https://<storage-account>.dfs.core.windows.net
```

The Blob and DFS endpoints belong to the same storage account; they do not represent separate copies of the data.

---

## 9. Access Blob Storage Using Python

The Azure Storage Blob client library for Python can be used to upload, download, and manage blobs.

Install the library:

```bash
pip install azure-storage-blob azure-identity
```

### 9.1 Connect Using DefaultAzureCredential

```python
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

account_url = "https://stcompanydata.blob.core.windows.net"

credential = DefaultAzureCredential()

blob_service_client = BlobServiceClient(
    account_url=account_url,
    credential=credential
)
```

`DefaultAzureCredential` supports several authentication methods depending on the environment, such as developer credentials and managed identity.

The identity must have the required storage data permissions.

### 9.2 List Containers

```python
for container in blob_service_client.list_containers():
    print(container["name"])
```

### 9.3 Upload a File

```python
container_name = "company-data"
blob_name = "raw/orders.csv"
local_file = "orders.csv"

blob_client = blob_service_client.get_blob_client(
    container=container_name,
    blob=blob_name
)

with open(local_file, "rb") as data:
    blob_client.upload_blob(
        data,
        overwrite=True
    )
```

### 9.4 Download a File

```python
with open("downloaded_orders.csv", "wb") as file:
    download_stream = blob_client.download_blob()
    file.write(download_stream.readall())
```

### 9.5 Delete a Blob

```python
blob_client.delete_blob()
```

**Important:** Use secure authentication and grant only the required data-plane permissions.

---

## 10. Access Blob Storage from Azure Databricks

Databricks can read and write files stored in Blob Storage.

For a storage account without hierarchical namespace, a common URI format is:

```text
wasbs://<container>@<storage-account>.blob.core.windows.net/<path>
```

Example:

```python
source_path = (
    "wasbs://company-data@stcompanydata.blob.core.windows.net/raw/orders/"
)

df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load(source_path)

df.display()
```

For ADLS Gen2, the commonly used secure filesystem URI is:

```text
abfss://<container>@<storage-account>.dfs.core.windows.net/<path>
```

**Important:**

- `wasbs://` is a legacy Hadoop-compatible Blob Storage connector scheme.
- `abfss://` is commonly used for secure ADLS Gen2 filesystem access.
- The URI scheme does not provide authentication by itself.
- The exact connector and authentication support depend on the Databricks runtime and configuration.
- For modern Azure workloads, use supported Entra ID-based authentication and current platform recommendations.

---

## 11. Data Redundancy

Azure Storage provides redundancy options to protect stored data.

| Option | Meaning |
|---|---|
| LRS | Locally redundant storage |
| ZRS | Zone-redundant storage |
| GRS | Geo-redundant storage |
| RA-GRS | Read-access geo-redundant storage |
| GZRS | Geo-zone-redundant storage |
| RA-GZRS | Read-access geo-zone-redundant storage |

### Basic Understanding

- **LRS:** Maintains multiple copies at one physical location.
- **ZRS:** Replicates data across availability zones in the primary region.
- **GRS:** Provides local redundancy and asynchronous replication to a secondary region.
- **RA-GRS:** GRS with read access to the secondary region.
- **GZRS:** Combines zone redundancy in the primary region with geo-replication.
- **RA-GZRS:** GZRS with read access to the secondary region.

**Important:** Geo-replication is generally asynchronous. Recent writes may not yet be available in the secondary region.

Storage redundancy is not a replacement for a complete application disaster recovery plan.

---

## 12. Data Protection Features

Blob Storage provides data protection capabilities to help recover from accidental deletion or modification.

### 12.1 Soft Delete

Soft delete helps recover deleted blobs or containers during a configured retention period.

Instead of immediately permanently deleting the data, Azure retains it for recovery according to the configured policy.

### 12.2 Blob Versioning

Blob versioning automatically maintains previous versions of a blob when it is modified or overwritten.

Example:

```text
orders.csv
    |
    +---- Version 1
    |
    +---- Version 2
    |
    +---- Version 3 (Current)
```

Previous versions can be used to recover older data, subject to retention and configuration.

### 12.3 Point-in-Time Restore

Point-in-time restore can restore block blob data to an earlier point in time within the supported retention window and configuration.

It depends on features such as versioning, change feed, and soft delete.

### 12.4 Blob Snapshots

A snapshot captures a read-only version of a blob at a specific point in time.

It can help preserve a previous state.

### Comparison

| Feature | Purpose |
|---|---|
| Soft delete | Recover deleted blobs or containers |
| Versioning | Maintain previous blob versions |
| Point-in-time restore | Restore supported blob data to an earlier time |
| Snapshot | Capture a read-only point-in-time state |

These features have different prerequisites, costs, and limitations. They should be configured according to the required recovery strategy.

---

## 13. Blob Lifecycle Management

Lifecycle management automates supported actions based on configured rules.

A policy can use conditions such as:

- Blob age.
- Last modified time.
- Last access time, where supported and enabled.
- Blob prefix.
- Blob index tags.

### Example

A company wants to manage historical transaction files:

```text
New Files
    |
    v
Hot Tier
    |
    | After configured period
    v
Cool Tier
    |
    | After configured period
    v
Archive Tier
    |
    | After retention period
    v
Delete
```

This is an example policy, not a default automatic process.

The policy must be configured, and tiering and deletion must follow the service's supported conditions.

### Benefits

- Reduces storage costs.
- Automates data retention.
- Helps manage historical data.
- Reduces manual storage administration.

**Important:** Archive data must be rehydrated before normal access. Lifecycle policies should be designed carefully to avoid moving active data into unsuitable tiers.

---

## 14. Blob Storage in a Data Engineering Pipeline

Blob Storage is commonly used as a landing and storage layer in data engineering architectures.

### Example Architecture

```text
Source Systems
      |
      v
Azure Data Factory
      |
      v
Azure Blob Storage
      |
      v
Azure Databricks
      |
      v
Data Transformation
      |
      v
Processed Data
      |
      v
Curated Data
      |
      v
Reporting / Analytics
```

### Responsibilities

**Azure Data Factory**
- Connects to source systems.
- Copies data into Blob Storage.
- Orchestrates ingestion workflows.

**Azure Blob Storage**
- Stores raw source files.
- Stores historical data.
- Provides scalable object storage.

**Azure Databricks**
- Reads stored files.
- Processes data using PySpark.
- Writes transformed datasets.

**Analytics Services**
- Consume curated data for reporting and analysis.

### Example Folder Structure

```text
company-data/
    |
    +---- raw/
    |       +---- orders/
    |       +---- customers/
    |
    +---- processed/
    |       +---- orders/
    |       +---- customers/
    |
    +---- curated/
            +---- sales_summary/
            +---- customer_metrics/
```

---

## 15. Performance and Cost Considerations

### Performance

Consider the following when designing Blob Storage workloads:

1. Choose a suitable access tier.
2. Use efficient file formats such as Parquet for analytical workloads.
3. Avoid creating excessive small files.
4. Use appropriate partitioning for downstream query patterns.
5. Consider parallel uploads and downloads for large workloads.
6. Select a suitable redundancy configuration.
7. Monitor storage transactions and latency.

### Cost

Important cost components include:

- Stored data capacity.
- Read and write transactions.
- Data retrieval.
- Data transfer.
- Redundancy configuration.
- Data protection features.
- Early deletion charges for applicable tiers.

**Important:** There is no single cheapest tier for every workload. A tier with low storage cost may have higher retrieval or transaction costs.

---

## 16. Common Blob Storage Problems

### 16.1 Authentication Failure

Possible causes:

- Invalid credentials.
- Expired token.
- Incorrect managed identity configuration.
- Invalid SAS token.

### 16.2 Authorization Failure

Possible causes:

- Missing Storage Blob Data Reader or Contributor role.
- Incorrect RBAC scope.
- Insufficient permissions for the requested operation.

### 16.3 Network Access Failure

Possible causes:

- Storage firewall restrictions.
- Public network access disabled.
- Private endpoint or DNS configuration issue.
- Missing network connectivity.

### 16.4 Blob Not Found

Possible causes:

- Incorrect account name.
- Incorrect container.
- Incorrect blob name.
- Blob was deleted or moved.

### Troubleshooting Checklist

1. Verify the storage account endpoint.
2. Verify the container and blob name.
3. Confirm the authentication method.
4. Check data-plane RBAC permissions.
5. Check SAS permissions and expiry, if used.
6. Inspect storage firewall and private networking.
7. Check whether the blob exists.
8. Review application or pipeline logs.

---

## 17. Common Interview Questions

### Q1. What is Azure Blob Storage?

Azure Blob Storage is an object storage service for storing large amounts of unstructured data such as files, images, videos, backups, and data engineering datasets.

### Q2. What are the three types of blobs?

- Block blobs.
- Append blobs.
- Page blobs.

### Q3. What is the difference between block, append, and page blobs?

Block blobs are used for general object storage, append blobs are optimized for append operations, and page blobs support random read/write operations.

### Q4. What is the difference between Blob Storage and ADLS Gen2?

ADLS Gen2 is built on Blob Storage with hierarchical namespace enabled, providing directory and filesystem capabilities useful for analytics workloads.

### Q5. What are Blob access tiers?

Hot, Cool, Cold, and Archive. They support different access patterns and cost structures.

### Q6. Can we directly read data from Archive tier?

No. Archive is offline storage. The data must be rehydrated before normal access.

### Q7. What is a container?

A container is a logical grouping of blobs inside a storage account.

### Q8. What is a SAS token?

A SAS token provides delegated access to storage resources with defined permissions and validity constraints.

### Q9. What is the difference between a SAS token and a storage account key?

A SAS token provides limited delegated access. A storage account key provides broad Shared Key access to the storage account.

### Q10. What is the difference between LRS and ZRS?

LRS maintains multiple copies at one physical location. ZRS replicates data across availability zones in the primary region.

### Q11. What is Blob Versioning?

It maintains previous versions of a blob when the blob is modified or overwritten.

### Q12. What is Blob Soft Delete?

It allows deleted blobs or containers to be recovered during a configured retention period.

### Q13. What is lifecycle management?

It automates supported tiering and deletion actions based on configured conditions.

### Q14. How does Azure Databricks access Blob Storage?

Databricks uses a supported storage connector and authentication configuration to read and write blobs. Legacy Hadoop-compatible access may use `wasbs://`.

### Q15. How would you use Blob Storage in a data engineering project?

I would use it to land source files, retain raw data, and provide storage for downstream transformation services such as Databricks.

---

## 18. Common Mistakes to Avoid

- Confusing Blob Storage with ADLS Gen2.
- Assuming a container is the same as a directory.
- Assuming all blobs are block blobs.
- Confusing access tiers with storage redundancy.
- Assuming Archive data can be queried immediately.
- Assuming a SAS token provides unlimited access.
- Sharing storage account keys in code.
- Assuming Blob Storage automatically transforms data.
- Assuming versioning and soft delete are the same feature.
- Assuming geo-redundancy guarantees zero data loss.
- Ignoring retrieval and early deletion charges.
- Assuming `wasbs://` is the preferred access method for every modern Azure workload.

---

## 19. Quick Revision

| Topic | Key Point |
|---|---|
| Blob Storage | Azure object storage |
| Storage account | Top-level storage resource |
| Container | Logical grouping of blobs |
| Blob | Individual stored object |
| Block Blob | General file/object storage |
| Append Blob | Append-oriented storage |
| Page Blob | Random read/write storage |
| Hot | Frequently accessed data |
| Cool | Infrequently accessed data |
| Cold | Rarely accessed online data |
| Archive | Offline long-term storage |
| SAS | Delegated, limited access |
| Storage account key | Broad Shared Key credential |
| LRS | Local redundancy |
| ZRS | Zone redundancy |
| GRS | Local plus geo-redundancy |
| Soft delete | Recover deleted data |
| Versioning | Maintain previous blob versions |
| Snapshot | Read-only point-in-time blob state |
| Lifecycle management | Automated tiering and deletion |
| Blob endpoint | `blob.core.windows.net` |
| DFS endpoint | `dfs.core.windows.net` |

---

## 20. Self-Assessment

Try answering these questions without looking at the notes:

1. What is Azure Blob Storage?
2. Explain storage account, container, and blob.
3. What are the three blob types?
4. Explain block blob vs append blob vs page blob.
5. What is the difference between Blob Storage and ADLS Gen2?
6. Explain Hot, Cool, Cold, and Archive tiers.
7. Why must Archive data be rehydrated?
8. What is a SAS token?
9. Explain the difference between SAS and storage account keys.
10. What are LRS, ZRS, GRS, and GZRS?
11. Explain soft delete, versioning, and snapshots.
12. What is lifecycle management?
13. How can Databricks access Blob Storage?
14. How would you troubleshoot a Blob Storage access error?
15. Design a simple data engineering pipeline using ADF, Blob Storage, and Databricks.

**Interview readiness:** You should be able to explain Blob Storage architecture, blob types, access tiers, authentication, redundancy, data protection, lifecycle management, and its role in an Azure data engineering pipeline.