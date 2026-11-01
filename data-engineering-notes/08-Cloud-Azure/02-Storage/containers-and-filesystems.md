# Azure Containers and Filesystems

## 1. What Are Containers and Filesystems?

In Azure Storage, a **container** is a logical grouping of blobs inside a storage account.

In Azure Data Lake Storage Gen2 (ADLS Gen2), the term **filesystem** is commonly used to refer to a container.

A container or filesystem provides a boundary for organizing data and managing access.

### Example

```text
Storage Account: stcompanydata
    |
    +---- Container: raw-data
    |       |
    |       +---- customers/
    |       +---- orders/
    |
    +---- Container: processed-data
    |       |
    |       +---- customers/
    |       +---- orders/
    |
    +---- Container: curated-data
            |
            +---- sales/
            +---- finance/
```

Here:

- `stcompanydata` is the storage account.
- `raw-data`, `processed-data`, and `curated-data` are containers.
- The directories organize files inside each container.

---

## 2. What Is a Container?

A container is a logical grouping of blobs in an Azure Storage account.

It helps organize objects and provides a scope for certain storage operations and access controls.

### Main Characteristics

- A storage account can contain multiple containers.
- A container stores blobs.
- Blob names can contain slash-separated paths.
- Containers can have access policies and metadata.
- Access can be controlled using Azure RBAC, SAS, and other supported mechanisms.

### Example

```text
Storage Account
    |
    v
Container: sales
    |
    +---- orders.csv
    +---- customers.csv
    +---- reports/
            |
            +---- monthly.csv
```

The container is the logical grouping. The names and paths identify the blobs stored inside it.

---

## 3. What Is a Filesystem in ADLS Gen2?

In ADLS Gen2, a filesystem is the data lake terminology commonly used for a container.

It provides a logical namespace in which directories and files can be organized.

Example:

```text
Storage Account
    |
    v
Filesystem: company-data
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

The filesystem is the top-level grouping, while directories organize the data within it.

**Important:** A filesystem in ADLS Gen2 is not the same as a local computer filesystem. It is a cloud storage namespace that provides filesystem-style operations.

---

## 4. Container vs Filesystem

| Container | Filesystem |
|---|---|
| Azure Blob Storage terminology | Common ADLS Gen2 terminology |
| Logical grouping of blobs | Logical grouping of files and directories |
| Used with Blob Storage | Used with hierarchical namespace |
| Can contain blob names with virtual paths | Supports hierarchical directory and file operations |
| Supports Azure Storage access mechanisms | Supports RBAC, SAS, and ACL-based access control |

**Interview point:** In ADLS Gen2, a filesystem is essentially a container with hierarchical namespace capabilities available through the storage account.

---

## 5. What Is Hierarchical Namespace?

Hierarchical Namespace (HNS) is a storage account capability that provides directory and file-system semantics.

Without HNS, slash-separated blob names can represent virtual folders.

With HNS, directories and files are managed as part of a hierarchical namespace.

### Without HNS

```text
Container: company-data

Blob name:
raw/sales/2026/orders.csv
```

The slash-separated path is part of the blob name. Directories are generally virtual.

### With HNS

```text
Filesystem: company-data
    |
    +---- raw/
            |
            +---- sales/
                    |
                    +---- 2026/
                            |
                            +---- orders.csv
```

Directories are managed hierarchically.

### Benefits of HNS

1. Supports directory-level organization.
2. Enables efficient directory operations.
3. Supports ACL-based access control.
4. Provides filesystem-style rename and delete operations.
5. Supports analytics workloads that use Hadoop-compatible filesystem interfaces.

**Important:** HNS is enabled at the storage account level, not separately for each container.

---

## 6. Container, Directory, and File

These are three different levels in an ADLS Gen2 structure.

| Component | Meaning |
|---|---|
| Container / Filesystem | Top-level logical grouping |
| Directory | Organizes files and subdirectories |
| File | Stores the actual data |

### Example

```text
Storage Account: stcompanydata
    |
    v
Filesystem: company-data
    |
    +---- raw/                  <- Directory
    |       |
    |       +---- orders/       <- Directory
    |               |
    |               +---- orders.csv  <- File
    |
    +---- processed/            <- Directory
    |
    +---- curated/              <- Directory
```

A common interview mistake is to call every folder a container. A container is the top-level grouping; folders inside it are directories.

---

## 7. Designing Containers for a Data Lake

A data engineering project can organize data using separate containers or a single container with multiple directories.

Both approaches can be valid. The decision depends on security, ownership, lifecycle, and operational requirements.

### Approach 1: Separate Containers

```text
Storage Account
    |
    +---- raw-data
    |
    +---- processed-data
    |
    +---- curated-data
```

**Advantages:**

- Clear separation between data layers.
- Easier to apply container-level access policies.
- Different containers can have different configurations where supported.
- Clearer ownership boundaries.

**Disadvantages:**

- More containers to manage.
- Cross-container access may require additional configuration.
- Shared conventions must be maintained.

### Approach 2: One Container with Directories

```text
Storage Account
    |
    +---- company-data
            |
            +---- raw/
            |
            +---- processed/
            |
            +---- curated/
```

**Advantages:**

- Simpler top-level organization.
- Easy to navigate a single data lake namespace.
- ACLs can provide directory-level access separation.

**Disadvantages:**

- Requires careful ACL design.
- More attention is needed for directory traversal permissions.
- Poorly managed permissions can expose unintended data.

### Which Approach Should You Choose?

Use separate containers when data layers or teams require clear access and management boundaries.

Use directories when the layers belong to one logical data lake and directory-level access control is sufficient.

There is no universal rule that every data lake must use three separate containers.

---

## 8. Access Control at Different Levels

Azure Storage supports multiple authorization mechanisms.

### 8.1 Azure RBAC

Azure RBAC grants permissions to identities at resource scopes.

For example:

```text
Principal: Databricks managed identity
Role: Storage Blob Data Contributor
Scope: Required container
```

This provides data access within the assigned scope.

### 8.2 Access Control Lists (ACLs)

ADLS Gen2 ACLs provide more granular access to directories and files.

Example:

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

- Ingestion team: write to `raw/`.
- Engineering team: read `raw/`, write `processed/`.
- Analytics team: read `curated/`.

### 8.3 SAS

A Shared Access Signature provides delegated access to a storage resource with specified permissions and validity constraints.

### Important

- RBAC and ACLs serve different purposes.
- Broad data-plane RBAC permissions can independently grant data access.
- ACLs control access at the directory and file level.
- Execute permission on parent directories is required to traverse a path.
- Default ACLs apply to newly created children, not existing files.

For detailed concepts, see `access-control-and-permissions.md`.

---

## 9. Working with Containers and Filesystems in Azure Portal

A typical workflow for creating an ADLS Gen2 storage structure is:

1. Open the Azure Portal.
2. Create or select a storage account.
3. Ensure hierarchical namespace is enabled if ADLS Gen2 capabilities are required.
4. Open the storage account.
5. Navigate to Data Storage.
6. Open Containers.
7. Create a container.
8. Open the container.
9. Create directories.
10. Upload files into the required directories.
11. Configure access permissions.

Example structure:

```text
company-data/
    |
    +---- raw/
    +---- processed/
    +---- curated/
```

The exact portal labels may vary as the Azure interface changes.

---

## 10. Accessing Containers and Filesystems with Azure CLI

Azure CLI can be used to manage containers and directories.

### 10.1 Create a Container

```bash
az storage container create \
    --name company-data \
    --account-name stcompanydata \
    --auth-mode login
```

### 10.2 List Containers

```bash
az storage container list \
    --account-name stcompanydata \
    --auth-mode login \
    --output table
```

### 10.3 Create a Directory in ADLS Gen2

```bash
az storage fs directory create \
    --name raw/orders \
    --file-system company-data \
    --account-name stcompanydata \
    --auth-mode login
```

### 10.4 List Filesystems

```bash
az storage fs list \
    --account-name stcompanydata \
    --auth-mode login \
    --output table
```

### 10.5 List Directory Contents

```bash
az storage fs file list \
    --path raw/orders \
    --file-system company-data \
    --account-name stcompanydata \
    --auth-mode login \
    --output table
```

**Important:** These commands use Microsoft Entra ID login. The signed-in identity must have the required permissions. Azure CLI command behavior and options can vary by CLI version.

---

## 11. Accessing ADLS Gen2 with Python

The Azure Data Lake Storage Gen2 Python SDK supports filesystem and directory operations.

Install the required packages:

```bash
pip install azure-storage-file-datalake azure-identity
```

### 11.1 Create a Filesystem Client

```python
from azure.identity import DefaultAzureCredential
from azure.storage.filedatalake import DataLakeServiceClient

account_url = "https://stcompanydata.dfs.core.windows.net"

credential = DefaultAzureCredential()

service_client = DataLakeServiceClient(
    account_url=account_url,
    credential=credential
)
```

### 11.2 Create a Filesystem

```python
filesystem_client = service_client.create_file_system(
    file_system="company-data"
)
```

If the filesystem already exists, handle the existing-resource case rather than assuming creation will always succeed.

### 11.3 Get an Existing Filesystem

```python
filesystem_client = service_client.get_file_system_client(
    file_system="company-data"
)
```

### 11.4 Create a Directory

```python
directory_client = filesystem_client.create_directory(
    "raw/orders"
)
```

### 11.5 List Paths

```python
for path in filesystem_client.get_paths(path="raw"):
    print(path.name)
```

### 11.6 Upload a File

```python
file_client = filesystem_client.get_file_client(
    "raw/orders/orders.csv"
)

with open("orders.csv", "rb") as data:
    file_client.upload_data(
        data,
        overwrite=True
    )
```

The identity used by `DefaultAzureCredential` must have the required data permissions.

---

## 12. Accessing Containers and Filesystems from Databricks

Databricks can access ADLS Gen2 using a secure filesystem URI.

Common format:

```text
abfss://<filesystem>@<storage-account>.dfs.core.windows.net/<path>
```

Example:

```python
source_path = (
    "abfss://company-data@stcompanydata.dfs.core.windows.net/raw/orders/"
)

df = spark.read \
    .format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load(source_path)

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
| `abfss://` | Secure filesystem URI scheme |
| `company-data` | Filesystem/container name |
| `stcompanydata` | Storage account |
| `dfs.core.windows.net` | ADLS Gen2 endpoint |
| `/raw/orders/` | Directory path |

**Important:** The URI identifies the storage location. Authentication and authorization must be configured separately.

---

## 13. Container-Level vs Directory-Level Organization

Consider this requirement:

- The ingestion team can write raw data.
- The engineering team can transform raw data.
- The analytics team can read curated data.

Two possible designs are:

### Design A: Separate Containers

```text
Storage Account
    |
    +---- raw-data
    |
    +---- processed-data
    |
    +---- curated-data
```

Access can be assigned at the container scope using suitable Azure RBAC roles.

### Design B: One Container with Directories

```text
Storage Account
    |
    +---- company-data
            |
            +---- raw/
            +---- processed/
            +---- curated/
```

Access can be managed with directory-level ACLs, together with appropriately scoped RBAC roles.

### Interview Answer

"I would choose the organization based on security boundaries and operational requirements. Separate containers can simplify broad access separation, while a single container with directories can provide a unified data lake structure with ACL-based permissions."

---

## 14. Common Problems and Troubleshooting

### 14.1 Filesystem Not Found

Possible causes:

- Incorrect filesystem name.
- Filesystem was not created.
- Wrong storage account.
- Incorrect endpoint.

### 14.2 Directory Not Found

Possible causes:

- Incorrect directory path.
- Directory was deleted or renamed.
- Wrong filesystem.
- Case or naming mismatch.

### 14.3 Authorization Failure

Possible causes:

- Missing data-plane RBAC role.
- Insufficient ACL permissions.
- Missing execute permission on a parent directory.
- Incorrect role assignment scope.
- Role assignment propagation delay.

### 14.4 Authentication Failure

Possible causes:

- Invalid credentials.
- Managed identity not configured.
- Expired or invalid token.
- Incorrect service principal configuration.

### 14.5 Network Failure

Possible causes:

- Storage firewall restrictions.
- Private endpoint misconfiguration.
- Private DNS resolution issue.
- Missing network connectivity.

### Troubleshooting Checklist

1. Verify the storage account.
2. Verify the container/filesystem name.
3. Verify the directory and file path.
4. Confirm the identity used by the application.
5. Check authentication.
6. Check RBAC roles and scope.
7. Inspect ACLs and parent-directory execute permissions.
8. Check network and private endpoint configuration.
9. Review Azure and application logs.

---

## 15. Common Interview Questions

### Q1. What is a container in Azure Storage?

A container is a logical grouping of blobs inside a storage account.

### Q2. What is a filesystem in ADLS Gen2?

A filesystem is the ADLS Gen2 terminology commonly used for a container. It provides a namespace for organizing directories and files.

### Q3. What is the difference between a container and a directory?

A container is the top-level logical grouping. A directory organizes files and subdirectories inside it.

### Q4. Is a filesystem the same as a local filesystem?

No. An ADLS Gen2 filesystem is a cloud storage namespace that provides filesystem-style operations.

### Q5. What is hierarchical namespace?

HNS provides hierarchical directory and file semantics within an Azure Storage account.

### Q6. Can HNS be enabled separately for each container?

No. HNS is enabled at the storage account level.

### Q7. Can one storage account have multiple containers?

Yes. A storage account can contain multiple containers, subject to Azure service limits.

### Q8. How would you organize a data lake?

I would organize data by business domain, source system, or processing layer, using containers and directories based on access and operational requirements.

### Q9. When would you use separate containers instead of directories?

When different data areas require distinct access boundaries, ownership, or management policies.

### Q10. How do you access ADLS Gen2 from Databricks?

Using a supported authentication mechanism and a secure filesystem URI such as `abfss://`.

### Q11. What is the difference between RBAC and ACL in ADLS Gen2?

RBAC provides role-based access at Azure resource scopes. ACLs provide granular permissions on directories and files.

### Q12. Does creating a directory automatically grant access to it?

No. The caller needs appropriate authorization. Access can depend on RBAC, ACLs, and parent-directory traversal permissions.

### Q13. What is the difference between Blob Storage and ADLS Gen2 directory structures?

Blob Storage without HNS generally uses virtual directory paths in blob names. ADLS Gen2 with HNS manages directories and files hierarchically.

---

## 16. Common Mistakes to Avoid

- Treating a container and directory as the same thing.
- Assuming a filesystem is a local disk.
- Assuming every slash-separated blob path represents a physical directory.
- Assuming HNS can be enabled separately for each container.
- Assuming a container automatically grants data access.
- Confusing the storage account with a container.
- Assuming a storage URI handles authentication.
- Assuming ACLs replace all RBAC permissions.
- Forgetting parent-directory execute permissions.
- Creating too many containers without a clear access or management requirement.

---

## 17. Quick Revision

| Topic | Key Point |
|---|---|
| Storage account | Top-level Azure Storage resource |
| Container | Logical grouping of blobs |
| Filesystem | Common ADLS Gen2 term for a container |
| Directory | Organizes files and subdirectories |
| File | Stores actual data |
| HNS | Hierarchical Namespace |
| HNS scope | Storage account level |
| Blob path without HNS | Slash-separated blob name |
| ADLS Gen2 path | Hierarchical directory/file path |
| RBAC | Role-based access at resource scopes |
| ACL | Fine-grained directory/file permissions |
| `abfss://` | Secure ADLS Gen2 filesystem URI |
| DFS endpoint | ADLS Gen2 filesystem operations |

---

## 18. Self-Assessment

Try answering these questions without looking at the notes:

1. What is a container?
2. What is a filesystem in ADLS Gen2?
3. Explain container vs filesystem.
4. Differentiate container, directory, and file.
5. What is hierarchical namespace?
6. Can HNS be enabled at the container level?
7. How would you organize raw, processed, and curated data?
8. When would you use separate containers?
9. When would you use directories inside one container?
10. How do RBAC and ACLs control access?
11. How can you create a filesystem using Azure CLI?
12. How can Python access ADLS Gen2?
13. How does Databricks access a filesystem using `abfss://`?
14. How would you troubleshoot a filesystem authorization error?

**Interview readiness:** You should be able to explain the storage account hierarchy, containers, filesystems, directories, HNS, access control, and how data engineering tools interact with ADLS Gen2.