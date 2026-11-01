# Azure Interview Questions and Answers for Data Engineers

## 1. Azure Cloud Fundamentals

### Q1. What Is Microsoft Azure?

Microsoft Azure is Microsoft's cloud computing platform. It provides services for computing, storage, networking, databases, analytics, security, and application development.

For Data Engineers, Azure provides services to:
- Ingest data from different sources.
- Store structured and unstructured data.
- Process large datasets.
- Build ETL and ELT pipelines.
- Orchestrate data workflows.
- Secure and monitor data platforms.

### Q2. What Are the Main Cloud Service Models?

| Model | Description | Example |
|---|---|---|
| IaaS | Infrastructure managed through cloud resources | Azure Virtual Machines |
| PaaS | Managed platform for applications and data services | Azure SQL Database |
| SaaS | Ready-to-use software managed by the provider | Microsoft 365 |

**IaaS:** You manage more of the operating system and infrastructure.

**PaaS:** The provider manages the underlying infrastructure and platform.

**SaaS:** You use the application without managing its underlying platform.

### Q3. What Are the Types of Cloud Deployment?

- **Public cloud:** Services run on a cloud provider's shared infrastructure.
- **Private cloud:** Infrastructure dedicated to an organization.
- **Hybrid cloud:** Combines on-premises infrastructure and cloud services.
- **Multi-cloud:** Uses services from multiple cloud providers.

### Q4. What Is the Difference Between Scalability and Elasticity?

**Scalability** means increasing or decreasing system capacity to handle workload changes.

**Elasticity** means automatically or dynamically adjusting resources as demand changes.

Example:
- Increasing cluster capacity to handle a growing dataset is scalability.
- Automatically adding and removing compute resources based on workload is elasticity.

### Q5. What Is High Availability?

High availability means designing a system to remain operational with minimal downtime.

It can involve:
- Redundant resources.
- Availability Zones.
- Load balancing.
- Health monitoring.
- Automatic failover.

### Q6. What Is Fault Tolerance?

Fault tolerance is the ability of a system to continue operating when one or more components fail.

High availability and fault tolerance are related, but not identical. Fault tolerance focuses on continued operation despite failures, while high availability focuses on minimizing downtime.

### Q7. What Are RPO and RTO?

**RPO – Recovery Point Objective**

The maximum acceptable amount of data loss measured in time.

Example: An RPO of 15 minutes means the recovery process should lose no more than approximately 15 minutes of data.

**RTO – Recovery Time Objective**

The maximum acceptable time to restore a service after a failure.

Example: An RTO of 2 hours means the service should be restored within 2 hours.

### Q8. What Is an Azure Region?

An Azure region is a geographic area containing one or more Azure datacenters.

Organizations select regions based on:
- Data residency requirements.
- Latency.
- Service availability.
- Compliance.
- Cost.
- Disaster recovery requirements.

### Q9. What Is an Availability Zone?

An Availability Zone is a physically separate location within a supported Azure region.

Zones have independent infrastructure such as power, cooling, and networking.

Using multiple zones can improve availability when a zone fails, provided the service and architecture support zonal deployment.

### Q10. What Is the Difference Between an Availability Set and Availability Zones?

| Availability Set | Availability Zones |
|---|---|
| Distributes VMs across fault and update domains | Distributes resources across physically separate zones |
| Helps protect against localized hardware and maintenance failures | Helps protect against zone-level failures |
| Does not provide the same physical separation as zones | Provides stronger physical isolation where supported |

### Q11. What Is an Azure Resource Group?

A Resource Group is a logical container for Azure resources.

Example:

```text
rg-data-platform
    |
    |-- adf-data-pipeline
    |-- storage-data-lake
    |-- databricks-workspace
    |-- key-vault
```

Resource groups help organize, manage, monitor, and control access to related resources.

### Q12. What Is an Azure Subscription?

An Azure subscription is a management, billing, and access boundary for Azure resources.

A subscription contains resource groups, and resource groups contain resources.

```text
Microsoft Entra Tenant
        |
        v
Azure Subscription
        |
        v
Resource Groups
        |
        v
Azure Resources
```

A tenant and subscription are different concepts. A tenant represents an identity directory; a subscription is a resource and billing boundary.

---

## 2. Azure Storage and ADLS Gen2

### Q13. What Is Azure Blob Storage?

Azure Blob Storage is Microsoft's object storage service for storing large amounts of unstructured data.

Examples:
- CSV files.
- JSON files.
- Parquet files.
- Images.
- Videos.
- Backups.
- Log files.

It is commonly used as the storage foundation for data lakes.

### Q14. What Is ADLS Gen2?

Azure Data Lake Storage Gen2 is Azure Blob Storage with hierarchical namespace enabled.

It provides data-lake capabilities such as:
- Directory and file semantics.
- Access control lists.
- Large-scale analytical storage.
- Integration with analytics services.
- Hadoop-compatible access through supported connectors.

ADLS Gen2 is commonly used for Bronze, Silver, and Gold data layers.

### Q15. What Is the Difference Between Blob Storage and ADLS Gen2?

| Blob Storage | ADLS Gen2 |
|---|---|
| Object storage | Object storage with hierarchical namespace |
| Traditional flat namespace | True directory and file hierarchy |
| Supports object-level access controls | Supports RBAC and POSIX-style ACLs |
| General-purpose object storage | Designed with data-lake workloads in mind |

ADLS Gen2 is not a completely separate storage engine. It builds on Azure Blob Storage.

### Q16. What Is Hierarchical Namespace?

Hierarchical namespace (HNS) enables a real directory and file hierarchy in a storage account.

Without HNS, slash-separated paths are generally interpreted as object names with virtual directories.

With HNS, directories and files are first-class entities with filesystem-like operations and ACL support.

HNS is enabled at the storage-account level.

### Q17. What Is the Difference Between a Container and a Directory in ADLS Gen2?

**Container / Filesystem:** A top-level grouping inside a storage account.

**Directory:** A folder-like structure inside a container.

Example:

```text
storageaccount
    |
    |-- container: data
            |
            |-- bronze
            |     |-- customers
            |
            |-- silver
            |     |-- customers
            |
            |-- gold
                  |-- sales
```

### Q18. What Are the Common Access Tiers in Azure Blob Storage?

| Tier | Typical Use |
|---|---|
| Hot | Frequently accessed data |
| Cool | Infrequently accessed data |
| Cold | Rarely accessed data |
| Archive | Long-term, rarely accessed data |

Archive data is offline and generally needs rehydration before normal reading. Access and retention costs depend on the tier and operations.

### Q19. What Are the Types of Azure Storage Redundancy?

| Type | Description |
|---|---|
| LRS | Locally Redundant Storage |
| ZRS | Zone-Redundant Storage |
| GRS | Geo-Redundant Storage |
| RA-GRS | Read-Access Geo-Redundant Storage |
| GZRS | Geo-Zone-Redundant Storage |
| RA-GZRS | Read-Access Geo-Zone-Redundant Storage |

**LRS:** Maintains multiple copies within one datacenter or local storage location.

**ZRS:** Replicates data across availability zones in a supported region.

**GRS:** Replicates data to a secondary region, generally asynchronously.

**RA-GRS:** Provides read access to the secondary region.

**Important:** Geo-replication does not necessarily mean immediate replication or automatic application failover.

### Q20. What Is the Difference Between Blob Endpoint and DFS Endpoint?

ADLS Gen2 commonly uses two endpoints:

```text
Blob:
https://<account>.blob.core.windows.net

DFS:
https://<account>.dfs.core.windows.net
```

The DFS endpoint supports hierarchical filesystem operations.

The Blob endpoint supports Blob Storage APIs.

The correct endpoint depends on the application and connector.

### Q21. What Is the `abfss://` Protocol?

`abfss://` is a secure Hadoop-compatible URI scheme commonly used by Spark and Hadoop connectors to access ADLS Gen2.

Example:

```text
abfss://data@storageaccount.dfs.core.windows.net/bronze/customers/
```

It identifies a storage location. It does not authenticate the user or grant permissions by itself.

### Q22. What Is the Difference Between `abfss://` and `wasbs://`?

| abfss:// | wasbs:// |
|---|---|
| Used with ADLS Gen2 | Legacy Azure Blob Hadoop connector scheme |
| Commonly uses DFS endpoint | Uses Blob endpoint |
| Supports ADLS Gen2 filesystem operations | Does not provide the same HNS filesystem semantics |

For new ADLS Gen2 analytical workloads, `abfss://` is commonly preferred.

### Q23. What Is the Difference Between Soft Delete, Versioning, and Snapshots?

- **Soft delete:** Retains deleted data for a configured recovery period.
- **Versioning:** Maintains previous versions of a blob when it is modified.
- **Snapshots:** Capture a point-in-time state of a blob.

These features have different capabilities and configuration requirements. None should automatically be treated as a complete disaster-recovery strategy.

---

## 3. Azure Identity and Security

### Q24. What Is Microsoft Entra ID?

Microsoft Entra ID, formerly Azure Active Directory, is Microsoft's cloud identity and access management service.

It supports:
- User authentication.
- Group management.
- Application identities.
- Single sign-on.
- Conditional access.
- Identity-based authorization integrations.

### Q25. What Is the Difference Between Authentication and Authorization?

**Authentication:** Verifies who you are.

**Authorization:** Determines what you are allowed to access.

Example:

A Databricks job authenticates using a managed identity. Azure authorization determines whether that identity can read files from ADLS Gen2.

### Q26. What Is Azure RBAC?

Azure Role-Based Access Control manages permissions to Azure resources.

It can assign roles at different scopes:
- Management group.
- Subscription.
- Resource group.
- Resource.

Example roles include:
- Owner.
- Contributor.
- Reader.
- Storage Blob Data Reader.
- Storage Blob Data Contributor.

The role and scope determine the effective permissions.

### Q27. What Is the Difference Between Azure RBAC and ADLS ACLs?

| Azure RBAC | ADLS ACL |
|---|---|
| Role-based permission model | Fine-grained filesystem permission model |
| Assigned at resource scopes | Assigned to files and directories |
| Useful for broad data access | Useful for directory- and file-level access |
| Uses Azure role assignments | Uses owner, group, and other permission entries |

For ADLS Gen2, both can contribute to authorization. Broad data-plane RBAC permissions may grant access beyond the restrictions expressed by individual ACL entries.

### Q28. What Are ADLS Gen2 ACL Permissions?

ADLS Gen2 ACLs support:

- **Read (r):** Read file content or list directory contents.
- **Write (w):** Write or modify file content; directory write supports creating and deleting children.
- **Execute (x):** Traverse a directory.

Example:

```text
rwx
```

This means read, write, and execute permissions.

For directory access, execute permission on parent directories is important.

### Q29. What Is a Service Principal?

A service principal is an identity representing an application or service in Microsoft Entra ID.

It allows an application to authenticate and access Azure resources without using an individual user's identity.

It commonly uses:
- Client secret.
- Certificate.
- Federated identity credential.

### Q30. What Is a Managed Identity?

A managed identity is an identity managed by Azure that allows an Azure resource to authenticate to supported services without storing credentials in application code.

Types:

**System-assigned managed identity**
- Created with an Azure resource.
- Lifecycle is tied to that resource.

**User-assigned managed identity**
- Created as an independent Azure resource.
- Can be assigned to multiple supported resources.

Managed identities are represented by service principals in Microsoft Entra ID.

### Q31. What Is the Difference Between a Service Principal and a Managed Identity?

| Service Principal | Managed Identity |
|---|---|
| Application identity | Azure-managed identity |
| May require secret or certificate management | Avoids storing application credentials for supported Azure authentication |
| Can be used outside Azure | Commonly used by Azure-hosted resources |
| Credential lifecycle is managed by the application or administrator | Credential lifecycle is managed by Azure |

### Q32. What Is Azure Key Vault?

Azure Key Vault securely stores and controls access to:
- Secrets.
- Encryption keys.
- Certificates.

Data Engineering applications can use it to retrieve secrets without hardcoding them in notebooks or pipeline definitions.

### Q33. Why Should You Avoid Hardcoding Storage Account Keys?

Hardcoded keys:
- Can be exposed through source code or notebook history.
- Are difficult to rotate safely.
- Can grant broad access.
- Make access auditing harder.

Prefer managed identities or other supported identity-based authentication methods where possible.

---

## 4. Azure Data Factory (ADF)

### Q34. What Is Azure Data Factory?

Azure Data Factory is a cloud-based data integration and orchestration service.

It is used to:
- Extract data from sources.
- Move data between systems.
- Schedule and orchestrate pipelines.
- Execute transformation activities.
- Monitor pipeline runs.

### Q35. What Is an ADF Pipeline?

A pipeline is a logical grouping of activities that together perform a data integration workflow.

Example:

```text
Copy Data
    |
    v
Validate Data
    |
    v
Run Databricks Notebook
    |
    v
Load Target Table
```

### Q36. What Is an Activity in ADF?

An activity is an individual operation inside a pipeline.

Examples:
- Copy Activity.
- Lookup Activity.
- Get Metadata.
- ForEach.
- If Condition.
- Execute Pipeline.
- Databricks Notebook Activity.

### Q37. What Is a Linked Service?

A linked service defines the connection information used by ADF to connect to an external system.

Examples:
- Azure SQL Database.
- ADLS Gen2.
- Azure Databricks.
- REST API.
- On-premises SQL Server.

A linked service is similar to a connection definition.

### Q38. What Is a Dataset in ADF?

A dataset represents the data structure or location that an activity reads from or writes to.

Example:

A CSV dataset may define:
- Storage linked service.
- Container.
- File path.
- File format.
- Column structure.

**Difference:**

- Linked service: How to connect.
- Dataset: Which data or location to use.
- Activity: What operation to perform.
- Pipeline: How activities are organized.

### Q39. What Is an Integration Runtime?

Integration Runtime (IR) is the compute infrastructure used by ADF to provide data integration capabilities.

Main types:

**1. Azure Integration Runtime**
- Cloud-based data movement and supported activity execution.
- Can connect to supported cloud data stores.

**2. Self-hosted Integration Runtime**
- Installed on an on-premises or private network machine.
- Supports connectivity to private or on-premises systems.

**3. Azure-SSIS Integration Runtime**
- Runs SQL Server Integration Services packages in Azure.

### Q40. What Is Copy Activity?

Copy Activity moves data from a source to a destination.

Example:

```text
Azure SQL Database
        |
        v
   Copy Activity
        |
        v
    ADLS Gen2
```

It supports many connectors and can perform supported format conversions.

### Q41. What Is a Parameter in ADF?

A parameter is an input value passed to a pipeline, dataset, or linked service.

Parameters are useful for reusable pipelines.

Example:

```text
pipeline parameter:
    source_table = customers
```

The same pipeline can process multiple tables by passing different parameter values.

### Q42. What Is a Variable in ADF?

A variable is a pipeline-level value that can be set and updated during pipeline execution.

Difference:

- Parameters are inputs and are generally immutable during a pipeline run.
- Variables can be updated during the run.

### Q43. What Is the Difference Between Parameters and Variables?

| Parameters | Variables |
|---|---|
| Receive input values | Store pipeline execution values |
| Generally immutable during a run | Can be updated |
| Support reusable pipelines | Support intermediate logic |

### Q44. What Is a Trigger in ADF?

A trigger starts a pipeline based on a defined condition.

Common types:

- Schedule trigger.
- Tumbling window trigger.
- Event-based trigger.

**Schedule trigger:** Starts a pipeline at configured times.

**Tumbling window trigger:** Processes fixed, continuous time windows and supports dependency management and retry behavior.

**Event-based trigger:** Starts a pipeline when a configured event occurs, such as a supported storage event.

### Q45. What Is the Difference Between Schedule and Tumbling Window Triggers?

| Schedule Trigger | Tumbling Window Trigger |
|---|---|
| Starts pipelines according to a schedule | Processes defined time windows |
| Does not inherently provide window dependency management | Supports window dependencies |
| Useful for simple periodic execution | Useful for time-window-based data processing |

### Q46. What Is ForEach Activity?

ForEach iterates over a collection of items and executes activities for each item.

Example:

```text
Tables:
- customers
- products
- orders

ForEach:
    Process each table
```

It supports sequential or parallel execution, subject to configuration and activity limits.

### Q47. What Is Lookup Activity?

Lookup Activity retrieves data or configuration values from a supported source.

It is commonly used to:
- Read metadata.
- Fetch table names.
- Retrieve control-table values.
- Drive dynamic pipelines.

### Q48. What Is Get Metadata Activity?

Get Metadata retrieves metadata about a dataset or supported storage object.

Examples:
- File name.
- File size.
- Last modified time.
- Child items.
- Column structure where supported.

### Q49. How Do You Implement an Incremental Load in ADF?

A common approach uses a watermark column.

1. Read the last successful watermark.
2. Query source records newer than the watermark.
3. Copy or transform the records.
4. Write them to the destination.
5. Validate successful completion.
6. Update the watermark only after the target write succeeds.

Example source query:

```sql
SELECT *
FROM customers
WHERE modified_at > @last_watermark
  AND modified_at <= @current_watermark;
```

The exact parameter syntax depends on the source connector and query context.

**Important:** Timestamp-based incremental loads can miss late updates and deletes. CDC, overlap windows, and reconciliation may be required.

### Q50. What Is a Watermark Table?

A watermark table stores the latest successfully processed source position.

Example:

| table_name | last_watermark |
|---|---|
| customers | 2026-10-31 10:00:00 |
| orders | 2026-10-31 11:30:00 |

It helps the pipeline identify which records should be processed in the next run.

### Q51. How Do You Handle Pipeline Failures in ADF?

- Configure retry policies where appropriate.
- Capture activity failure details.
- Use failure dependencies for error handling.
- Log pipeline and activity run IDs.
- Preserve source progress.
- Make target writes idempotent.
- Notify support teams.
- Restart from a safe point.

Retries alone do not guarantee correct results. A failed activity may have written some data before reporting failure.

### Q52. How Do You Monitor ADF Pipelines?

Use:
- ADF Monitor hub.
- Pipeline run history.
- Activity run details.
- Azure Monitor.
- Diagnostic settings.
- Log Analytics.
- Alerts.

Monitor failure rate, execution duration, data volume, and freshness.

---

## 5. Azure Databricks

### Q53. What Is Azure Databricks?

Azure Databricks is a managed data and AI platform based on Apache Spark.

It supports:
- Distributed data processing.
- Data engineering.
- Data science.
- Streaming.
- Delta Lake.
- SQL analytics.
- Machine learning.

### Q54. What Is a Databricks Cluster?

A cluster is a compute environment used to run workloads.

A typical Spark cluster contains:
- Driver node.
- Worker nodes.
- Executors running on worker nodes.

The driver coordinates Spark execution, while executors process tasks.

The exact compute model depends on the selected Databricks compute type.

### Q55. What Is the Difference Between All-Purpose and Job Compute?

| All-Purpose Compute | Job Compute |
|---|---|
| Commonly used for interactive development | Used for scheduled or automated workloads |
| Can be shared by users where configured | Created or selected for job execution |
| May remain active between tasks | Often terminates after job completion |
| Useful for exploration and debugging | Useful for production jobs |

Databricks compute options, names, and billing behavior can change by workspace and product configuration.

### Q56. What Is a Databricks Notebook?

A notebook is an interactive environment for writing and executing code.

Supported languages commonly include:
- Python.
- SQL.
- Scala.
- R.

Notebooks are used for development, data exploration, transformation, and workflow tasks.

### Q57. What Is Delta Lake?

Delta Lake is a storage and table framework that provides ACID transactions and other table-management capabilities on top of data files, commonly Parquet.

Features include:
- ACID transactions.
- Schema enforcement.
- Schema evolution.
- Time Travel.
- MERGE, UPDATE, and DELETE.
- Transaction history.

### Q58. What Is the Delta Transaction Log?

The Delta transaction log is stored in the `_delta_log` directory.

It records committed table changes and allows readers to identify a consistent table snapshot.

It supports:
- Transaction history.
- Schema and protocol information.
- Added and removed files.
- Table-state reconstruction.

JSON commit files and checkpoints are used to maintain and reconstruct table state.

### Q59. What Is Time Travel in Delta Lake?

Time Travel allows users to query an earlier version of a Delta table.

Example:

```sql
SELECT *
FROM customers VERSION AS OF 5;
```

Or:

```sql
SELECT *
FROM customers TIMESTAMP AS OF '2026-10-01 10:00:00';
```

Time Travel requires the relevant transaction-log history and data files to remain available. VACUUM can make older versions unavailable.

### Q60. What Is VACUUM?

VACUUM removes unreferenced data files that are eligible under the configured retention rules.

Example:

```sql
VACUUM customers;
```

Before running VACUUM, understand the retention policy and recovery requirements.

Do not casually disable retention safety checks.

### Q61. What Is OPTIMIZE?

OPTIMIZE compacts small files in a Delta table to improve file layout and query performance.

```sql
OPTIMIZE sales;
```

For supported configurations, ZORDER can improve data skipping for selected columns:

```sql
OPTIMIZE sales
ZORDER BY (customer_id);
```

ZORDER is not a traditional database index. The best layout depends on query patterns and platform capabilities.

### Q62. What Is Unity Catalog?

Unity Catalog is Databricks' centralized governance solution.

It supports:
- Data discovery.
- Access control.
- Catalog and schema organization.
- Data lineage.
- Auditing.
- Centralized governance.

A common object hierarchy:

```text
Catalog
   |
   v
Schema
   |
   v
Table / View / Volume
```

### Q63. What Is a Databricks Workflow or Job?

A Databricks Job coordinates automated tasks.

It can include:
- Notebook tasks.
- Python tasks.
- SQL tasks.
- JAR tasks.
- Pipeline tasks.
- Dependencies between tasks.

Jobs can be scheduled and monitored.

### Q64. How Do You Pass Parameters to a Databricks Notebook?

Notebook widgets are one way to receive parameters.

```python
dbutils.widgets.text("source_table", "customers")

source_table = dbutils.widgets.get("source_table")

print(source_table)
```

Databricks Jobs can also pass task parameters through supported task configuration.

### Q65. What Is the Difference Between `dbutils.notebook.run()` and `%run`?

**`%run`:**
- Executes another notebook in the current notebook context.
- Makes definitions from the called notebook available in the current context.
- Useful for code organization and reuse.

**`dbutils.notebook.run()`:**
- Executes another notebook as a separate notebook run.
- Supports passing parameters and returning a string result.
- Has separate execution behavior and limitations.

For larger production workflows, Databricks Jobs with task dependencies are often easier to monitor and manage.

---

## 6. Azure Networking Fundamentals

### Q66. What Is a Virtual Network (VNet)?

An Azure Virtual Network is a logically isolated network in Azure.

It enables Azure resources to communicate with:
- Other resources in the same VNet.
- Connected VNets.
- On-premises networks.
- Supported external services.

### Q67. What Is a Subnet?

A subnet divides a VNet into smaller IP address ranges.

Example:

```text
VNet: 10.0.0.0/16

Subnet 1: 10.0.1.0/24
Subnet 2: 10.0.2.0/24
Subnet 3: 10.0.3.0/24
```

Subnets help organize workloads and apply network controls.

### Q68. What Is an NSG?

A Network Security Group filters inbound and outbound network traffic using rules.

Rules can specify:
- Source.
- Destination.
- Port.
- Protocol.
- Allow or deny.
- Priority.

NSGs are stateful. Lower numerical priority values are evaluated before higher numerical priority values.

### Q69. What Is an Azure Firewall?

Azure Firewall is a managed, stateful network security service.

It supports centralized traffic filtering and network protection.

It differs from an NSG:
- NSGs filter traffic at subnet or network-interface scope.
- Azure Firewall provides centralized network and application traffic control.

### Q70. What Is a Private Endpoint?

A Private Endpoint provides a private IP address in a VNet for supported Azure services through Azure Private Link.

It allows private connectivity to services such as supported storage and database services.

A private endpoint does not automatically grant access to the data. Identity and authorization permissions are still required.

### Q71. What Is a Service Endpoint?

A Service Endpoint extends a VNet's identity to supported Azure services.

It allows a supported service to apply network rules for traffic originating from the VNet.

Unlike a private endpoint, it does not assign a private IP address to the service inside the VNet.

### Q72. Private Endpoint vs Service Endpoint

| Private Endpoint | Service Endpoint |
|---|---|
| Uses a private IP in the VNet | Uses the service's public endpoint with VNet identity |
| Uses Azure Private Link | Uses VNet service endpoint integration |
| Can provide private connectivity | Does not provide a private IP to the service |
| Requires DNS configuration for private name resolution in many designs | Uses the service endpoint network integration |

### Q73. What Is VPN Gateway?

VPN Gateway connects Azure VNets or on-premises networks through encrypted VPN tunnels over the internet.

It is useful for secure connectivity without dedicated private circuit connectivity.

### Q74. What Is ExpressRoute?

ExpressRoute provides private connectivity between on-premises infrastructure and Microsoft cloud services through a connectivity provider.

It does not traverse the public internet in the same way as an ordinary internet VPN connection.

Do not assume ExpressRoute automatically encrypts every traffic flow at the application layer.

### Q75. What Is Azure Load Balancer?

Azure Load Balancer distributes network traffic across backend resources at Layer 4.

It supports TCP and UDP traffic.

For HTTP/HTTPS application-level routing, services such as Application Gateway or Azure Front Door may be more suitable.

---

## 7. Azure Monitoring and Cost Management

### Q76. What Is Azure Monitor?

Azure Monitor is Microsoft's monitoring platform for collecting and analyzing telemetry from Azure resources and applications.

It supports:
- Metrics.
- Logs.
- Alerts.
- Dashboards.
- Diagnostic data.

### Q77. What Is Log Analytics?

Log Analytics is used to query and analyze log data collected in a Log Analytics workspace.

It commonly uses Kusto Query Language (KQL).

Example:

```kusto
AzureActivity
| where TimeGenerated > ago(24h)
| summarize TotalEvents = count()
    by ResourceProvider
```

The table and available columns depend on the data sources configured in the workspace.

### Q78. What Is Application Insights?

Application Insights is an application performance monitoring feature in Azure Monitor.

It can help track:
- Requests.
- Dependencies.
- Exceptions.
- Application performance.
- Availability.

### Q79. How Would You Monitor a Data Engineering Pipeline in Azure?

Monitor different layers:

**ADF**
- Pipeline success and failure.
- Activity duration.
- Retry count.
- Data movement volume.

**Databricks**
- Job status.
- Spark UI metrics.
- Executor failures.
- Shuffle and spill.
- Processing duration.

**ADLS Gen2**
- Storage usage.
- Access failures.
- Request metrics.
- Data growth.

**Data quality**
- Row counts.
- Null rates.
- Duplicate keys.
- Schema changes.
- Freshness.

Use Azure Monitor, diagnostic settings, Log Analytics, and service-specific monitoring.

### Q80. How Do You Reduce Azure Data Engineering Costs?

- Use suitable compute sizes.
- Avoid leaving interactive clusters running unnecessarily.
- Use job compute or serverless options when suitable.
- Process only required data.
- Use incremental loads.
- Optimize Spark transformations.
- Compact small files when beneficial.
- Choose storage tiers based on access patterns.
- Monitor unused resources.
- Review data retention and replication costs.

Cost optimization should not compromise reliability, security, or required performance.

---

## 8. Practical Azure Data Engineering Scenarios

### Q81. Design an Azure Data Engineering Pipeline.

**Requirement:** Read customer and order data from Azure SQL Database and store transformed data in ADLS Gen2.

Possible architecture:

```text
Azure SQL Database
        |
        v
Azure Data Factory
        |
        v
ADLS Gen2 - Bronze
        |
        v
Azure Databricks
        |
        v
ADLS Gen2 - Silver
        |
        v
Databricks Transformations
        |
        v
ADLS Gen2 - Gold
        |
        v
Power BI / SQL Analytics
```

**ADF responsibilities:**
- Connect to the source.
- Copy data.
- Pass parameters.
- Orchestrate processing.
- Monitor pipeline execution.

**Databricks responsibilities:**
- Clean and validate data.
- Deduplicate records.
- Apply business transformations.
- Build Delta tables.
- Handle incremental processing.

**ADLS responsibilities:**
- Store raw and processed datasets.
- Organize Bronze, Silver, and Gold layers.
- Provide scalable storage.

### Q82. How Would You Secure an Azure Data Pipeline?

1. Use managed identities where supported.
2. Grant least-privilege Azure RBAC permissions.
3. Configure ADLS Gen2 ACLs where fine-grained access is needed.
4. Store required secrets in Key Vault.
5. Use private networking where required.
6. Enable monitoring and audit logging.
7. Restrict production access.
8. Avoid exposing credentials in notebooks and pipeline parameters.

### Q83. ADF Can Connect to Azure SQL but Cannot Access ADLS Gen2. What Would You Check?

Possible causes:
- Incorrect storage path.
- Missing storage permissions.
- Incorrect authentication method.
- Storage firewall restrictions.
- Private endpoint or DNS problems.
- Integration Runtime connectivity.

Investigation:
1. Verify the linked service.
2. Check the authentication identity.
3. Check Storage Blob Data roles.
4. Check ADLS ACLs.
5. Check network and DNS connectivity.
6. Inspect the activity error details.

### Q84. A Databricks Notebook Can Read ADLS but Cannot Write. What Would You Check?

Possible causes:
- Read permission exists but write permission is missing.
- Missing directory execute permission.
- Incorrect target path.
- Storage firewall restrictions.
- Invalid authentication or expired credentials.

Check:
- Managed identity or service principal.
- Azure RBAC data roles.
- ADLS ACL read, write, and execute permissions.
- Parent directory traversal.
- Network access.

### Q85. An ADF Pipeline Runs Successfully but Loads Duplicate Records. What Would You Do?

Possible causes:
- Pipeline retries after a partial write.
- Append-only target logic.
- Incorrect watermark management.
- Source contains duplicates.
- Multiple pipeline runs overlap.

Solutions:
- Define the target business key.
- Deduplicate source data deterministically.
- Use idempotent writes or MERGE where appropriate.
- Control overlapping pipeline execution.
- Update the watermark only after successful target completion.
- Reconcile target data.

### Q86. A Databricks Job Is Slow Even After Increasing Cluster Size. What Would You Investigate?

Increasing compute does not necessarily solve the bottleneck.

Check:
- Spark UI stages and tasks.
- Data skew.
- Shuffle volume.
- Disk spill.
- Join strategy.
- Small files.
- Source and sink I/O.
- DataFrame execution plan.

Apply the optimization that addresses the measured bottleneck rather than repeatedly increasing cluster size.

### Q87. How Would You Implement a Metadata-Driven ADF Pipeline?

Instead of creating one pipeline for every table, maintain a configuration table.

Example:

| source_table | target_path | load_type | watermark_column |
|---|---|---|---|
| customers | bronze/customers | incremental | modified_at |
| products | bronze/products | full | |
| orders | bronze/orders | incremental | updated_at |

Pipeline flow:

```text
Read Configuration
        |
        v
    ForEach Table
        |
        v
Determine Load Type
        |
        v
Copy / Incremental Load
        |
        v
Validate Output
        |
        v
Update Watermark
```

Benefits:
- Reusability.
- Less duplicated pipeline code.
- Easier onboarding of new tables.
- Centralized configuration.

The framework should still handle table-specific exceptions, schema changes, retries, and data-quality rules.

### Q88. How Would You Recover a Failed Incremental Pipeline?

1. Identify the failed pipeline run.
2. Check whether the target write completed or partially completed.
3. Check the last successfully committed watermark.
4. Verify whether the source data is still available.
5. Rerun from the last safe source position.
6. Use idempotent writes to avoid duplicates.
7. Reconcile source and target.
8. Advance the watermark only after successful completion.

Do not simply move the watermark forward to bypass a failure.

---

## 9. Important Azure Service Comparison Table

| Service | Main Purpose |
|---|---|
| Azure Data Factory | Data integration and orchestration |
| Azure Data Lake Storage Gen2 | Data lake storage |
| Azure Blob Storage | Object storage |
| Azure Databricks | Distributed processing and analytics |
| Azure Synapse Analytics | Analytics and data warehousing platform |
| Azure SQL Database | Managed relational database |
| Azure Event Hubs | Event ingestion |
| Azure Functions | Serverless event-driven compute |
| Azure Key Vault | Secrets, keys, and certificates |
| Microsoft Entra ID | Identity and access management |
| Azure Monitor | Monitoring and telemetry |
| Log Analytics | Log querying and analysis |
| Azure Purview / Microsoft Purview | Data governance and catalog capabilities |
| Power BI | Business intelligence and reporting |

---

## 10. Common Azure Interview Mistakes

1. Confusing Azure RBAC with ADLS ACLs.
2. Assuming managed identity automatically has storage access.
3. Confusing a linked service with a dataset in ADF.
4. Treating Integration Runtime as a pipeline activity.
5. Confusing ADF orchestration with Databricks processing.
6. Assuming ADLS Gen2 is a separate storage engine from Blob Storage.
7. Assuming private endpoints grant data permissions.
8. Confusing authentication and authorization.
9. Updating incremental watermarks before confirming successful writes.
10. Assuming every pipeline retry is safe.
11. Treating a successful ADF run as proof of correct data.
12. Assuming geo-replication is synchronous.
13. Confusing RPO with RTO.
14. Assuming increasing cluster size always improves Spark performance.
15. Hardcoding storage keys and passwords in notebooks.
16. Treating all Azure services as available in every region or configuration.

---

## 11. Quick Revision

| Concept | Remember |
|---|---|
| Azure | Microsoft's cloud platform |
| IaaS | Infrastructure as a Service |
| PaaS | Platform as a Service |
| SaaS | Software as a Service |
| Region | Geographic Azure area |
| Availability Zone | Physically separate location in a region |
| RPO | Maximum acceptable data loss measured in time |
| RTO | Maximum acceptable recovery time |
| Resource Group | Logical resource container |
| Subscription | Resource and billing boundary |
| Blob Storage | Object storage |
| ADLS Gen2 | Blob Storage with hierarchical namespace |
| HNS | Hierarchical Namespace |
| RBAC | Role-based resource/data access |
| ACL | Fine-grained filesystem permissions |
| Managed Identity | Azure-managed application identity |
| Key Vault | Secure secrets and key management |
| ADF | Data integration and orchestration |
| Pipeline | Group of activities |
| Linked Service | Connection definition |
| Dataset | Data structure or location |
| Integration Runtime | Data integration infrastructure |
| Watermark | Last successfully processed source position |
| Databricks | Managed Spark-based data platform |
| Delta Lake | Transactional table framework |
| Unity Catalog | Centralized Databricks governance |
| Private Endpoint | Private IP connectivity using Private Link |
| NSG | Network traffic filtering |
| Azure Monitor | Monitoring and telemetry |

---

## 12. Interview Preparation Checklist

### Azure Fundamentals
- [ ] Explain IaaS, PaaS, and SaaS.
- [ ] Explain scalability and elasticity.
- [ ] Explain regions and availability zones.
- [ ] Explain RPO and RTO.
- [ ] Explain subscriptions and resource groups.

### Storage
- [ ] Explain Blob Storage and ADLS Gen2.
- [ ] Explain hierarchical namespace.
- [ ] Explain containers and directories.
- [ ] Explain storage redundancy.
- [ ] Explain `abfss://`.
- [ ] Explain storage access tiers.

### Security
- [ ] Explain Entra ID.
- [ ] Differentiate authentication and authorization.
- [ ] Explain Azure RBAC and ADLS ACLs.
- [ ] Explain service principals and managed identities.
- [ ] Explain Key Vault.

### Azure Data Factory
- [ ] Explain pipelines and activities.
- [ ] Differentiate linked services and datasets.
- [ ] Explain Integration Runtime types.
- [ ] Explain Copy Activity.
- [ ] Explain parameters and variables.
- [ ] Explain trigger types.
- [ ] Implement incremental loading conceptually.
- [ ] Explain metadata-driven pipelines.
- [ ] Explain retries and failure recovery.

### Databricks
- [ ] Explain clusters and notebooks.
- [ ] Explain Delta Lake.
- [ ] Explain transaction logs.
- [ ] Explain Time Travel and VACUUM.
- [ ] Explain OPTIMIZE and ZORDER.
- [ ] Explain Unity Catalog.
- [ ] Explain Jobs and notebook parameters.

### Networking and Monitoring
- [ ] Explain VNet and subnet.
- [ ] Explain NSG and Firewall.
- [ ] Differentiate private and service endpoints.
- [ ] Explain VPN Gateway and ExpressRoute.
- [ ] Explain Azure Monitor and Log Analytics.
- [ ] Explain pipeline monitoring and cost optimization.

**Preparation priority:** Focus first on ADLS Gen2, ADF architecture, linked services, datasets, Integration Runtime, incremental loading, managed identities, RBAC versus ACLs, Databricks, and Delta Lake. These concepts are especially important when explaining an end-to-end Azure Data Engineering pipeline.