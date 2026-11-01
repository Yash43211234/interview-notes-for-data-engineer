# Azure Data Engineering – Quick Revision Cheat Sheet

> **Purpose:** Fast revision for Data Engineer interviews covering Azure fundamentals, ADLS Gen2, Azure Data Factory, Databricks, security, networking, and monitoring.

---

## 1. Azure Fundamentals

| Concept | Quick Explanation |
|---|---|
| Azure | Microsoft's cloud computing platform |
| Region | Geographic area containing Azure datacenters |
| Availability Zone | Separate physical datacenter location within a supported region |
| Subscription | Billing and resource-management boundary |
| Resource Group | Logical container for Azure resources |
| IaaS | Infrastructure as a Service |
| PaaS | Platform as a Service |
| SaaS | Software as a Service |
| Scalability | Ability to handle changing workload |
| Elasticity | Automatically or dynamically adjusting resources |
| High Availability | Keeping a service accessible |
| Fault Tolerance | Continuing operation despite certain failures |
| RPO | Maximum acceptable data loss measured in time |
| RTO | Target time to restore service |

### Cloud Service Models

| Model | Example |
|---|---|
| IaaS | Azure Virtual Machines |
| PaaS | Azure SQL Database |
| SaaS | Microsoft 365 |

### Remember

- Region = geographic location.
- Availability Zone = physically separate location within a region.
- Resource Group = logical resource container.
- Subscription = billing and management boundary.
- RPO = acceptable data loss.
- RTO = acceptable recovery time.

---

## 2. Azure Storage

### Storage Services

| Service | Purpose |
|---|---|
| Azure Blob Storage | Object storage |
| ADLS Gen2 | Analytics-oriented storage built on Blob Storage with HNS |
| Azure Files | Managed file shares |
| Azure Queue Storage | Message storage |
| Azure Table Storage | NoSQL key-value storage |

### ADLS Gen2

**ADLS Gen2 = Azure Blob Storage + Hierarchical Namespace (HNS).**

Important points:
- Designed for analytics workloads.
- Supports directories and files.
- Supports directory-level access control through ACLs.
- Works with Azure Databricks, Synapse, and other analytics services.
- HNS is enabled at the storage-account level.

### Storage Hierarchy

```text
Storage Account
      |
      v
Container / Filesystem
      |
      v
Directory
      |
      v
File
```

### Blob Storage vs ADLS Gen2

| Blob Storage without HNS | ADLS Gen2 with HNS |
|---|---|
| Object storage | Object storage with hierarchical namespace |
| Directory paths are generally virtual | Directories are first-class objects |
| Blob-level access controls | Supports POSIX-style ACLs |
| General object-storage workloads | Analytics and big-data workloads |

### Storage Access Tiers

| Tier | Use |
|---|---|
| Hot | Frequently accessed data |
| Cool | Infrequently accessed data |
| Cold | Rarely accessed data |
| Archive | Offline long-term storage |

Archive data generally needs rehydration before normal access.

### Redundancy

| Type | Meaning |
|---|---|
| LRS | Locally Redundant Storage |
| ZRS | Zone-Redundant Storage |
| GRS | Geo-Redundant Storage |
| RA-GRS | Read-Access Geo-Redundant Storage |
| GZRS | Geo-Zone-Redundant Storage |
| RA-GZRS | Read-Access Geo-Zone-Redundant Storage |

Geo-replication is generally asynchronous. A secondary read endpoint does not automatically mean the secondary region has taken over.

### Storage URI Formats

```text
abfss://container@account.dfs.core.windows.net/path/
```

- `abfss://` = secure ADLS Gen2 Hadoop-compatible URI.
- `dfs.core.windows.net` = ADLS Gen2 DFS endpoint.
- `blob.core.windows.net` = Blob Storage endpoint.
- `wasbs://` = legacy Azure Blob Hadoop connector URI.

**Important:** A URI identifies a storage location; it does not authenticate the user.

---

## 3. Azure Data Factory (ADF)

### What Is ADF?

Azure Data Factory is a cloud-based data integration and orchestration service.

It helps:
- Extract data.
- Copy data between systems.
- Schedule pipelines.
- Coordinate activities.
- Manage dependencies.
- Monitor pipeline execution.

### Core Components

| Component | Meaning |
|---|---|
| Pipeline | Logical group of activities |
| Activity | Individual operation |
| Linked Service | Connection information |
| Dataset | Data structure and location reference |
| Integration Runtime | Compute/integration infrastructure |
| Trigger | Starts a pipeline |
| Parameter | Input value passed to a pipeline |
| Variable | Pipeline-level mutable value |

### ADF Integration Runtime Types

| Type | Purpose |
|---|---|
| Azure IR | Cloud data movement and activity execution |
| Self-hosted IR | Access to private networks or on-premises systems |
| Azure-SSIS IR | Running SSIS packages |

### Common ADF Activities

| Activity | Purpose |
|---|---|
| Copy Activity | Move data |
| Lookup | Retrieve configuration or data |
| Get Metadata | Read file or dataset metadata |
| ForEach | Iterate over items |
| If Condition | Conditional execution |
| Set Variable | Assign variable value |
| Execute Pipeline | Invoke another pipeline |
| Databricks Notebook | Run a Databricks notebook |
| Stored Procedure | Execute a database procedure |

### Trigger Types

| Trigger | Purpose |
|---|---|
| Schedule | Run at scheduled times |
| Tumbling Window | Fixed, contiguous time windows with dependency support |
| Event-Based | Start on supported events, such as storage events |

### Important ADF Concepts

- Pipeline = workflow.
- Activity = task.
- Linked Service = connection.
- Dataset = data reference.
- Integration Runtime = connectivity and execution infrastructure.
- Trigger = execution start condition.
- Parameters = reusable inputs.
- Variables = values that can change during execution.

### ADF Incremental Loading

Typical flow:

```text
Read Previous Watermark
          |
          v
Read Current Watermark
          |
          v
Extract Changed Records
          |
          v
Copy to Target
          |
          v
Validate Successful Write
          |
          v
Update Watermark
```

**Important:** Update the watermark only after successful target processing. Timestamp-based loads can miss late updates and physical deletes; CDC, lookback windows, or reconciliation may be needed.

---

## 4. Azure Databricks

### What Is Databricks?

Azure Databricks is a managed data and AI platform built around Apache Spark.

Used for:
- Large-scale data processing.
- ETL/ELT.
- Batch processing.
- Structured Streaming.
- Machine learning.
- Delta Lake workloads.

### Core Components

| Component | Meaning |
|---|---|
| Workspace | Collaborative environment |
| Notebook | Interactive development interface |
| Cluster / Compute | Resources used to execute workloads |
| Driver | Coordinates Spark application execution |
| Executor | Executes tasks and stores cached data |
| Job | Scheduled or triggered workload |
| Delta Lake | Transactional table layer |
| Unity Catalog | Data governance and centralized access management |

### Driver vs Executor

| Driver | Executor |
|---|---|
| Coordinates application | Executes tasks |
| Builds execution plans | Processes partitions |
| Schedules tasks | Performs distributed work |
| Tracks job execution | Reports results and metrics |

### Databricks Compute

- All-purpose compute: Interactive development.
- Job compute: Workload-oriented execution.
- SQL warehouse: SQL analytics and BI workloads.
- Serverless compute: Managed compute where supported.

Availability, configuration, and billing vary by workspace, region, and plan.

### Notebook Basics

```python
# Python cell
print("Hello Databricks")
```

```sql
-- SQL cell
SELECT * FROM orders;
```

Common notebook utilities:

```python
dbutils.widgets.text("run_date", "")

run_date = dbutils.widgets.get("run_date")
```

- `%python` = Python cell.
- `%sql` = SQL cell.
- `%scala` = Scala cell.
- `%run` = Include another notebook in the current notebook context.
- `dbutils.notebook.run()` = Execute another notebook as a separate notebook run.

---

## 5. Delta Lake

### What Is Delta Lake?

Delta Lake is a transactional table layer commonly built using Parquet data files and a transaction log.

### Key Features

| Feature | Purpose |
|---|---|
| ACID Transactions | Reliable table operations |
| Schema Enforcement | Validate incoming schema |
| Schema Evolution | Support approved schema changes |
| MERGE | Insert, update, and delete matching records |
| Time Travel | Access earlier table versions |
| OPTIMIZE | Compact small files |
| ZORDER | Improve data skipping |
| VACUUM | Remove eligible unreferenced data files |

### Delta Architecture

```text
Delta Table
    |
    |--- Parquet Data Files
    |
    |--- _delta_log
             |
             |--- JSON Commits
             |
             |--- Checkpoints
```

### Important Operations

**MERGE**

```sql
MERGE INTO target AS t
USING source AS s
ON t.id = s.id
WHEN MATCHED THEN UPDATE SET *
WHEN NOT MATCHED THEN INSERT *;
```

**Time Travel**

```sql
SELECT *
FROM orders VERSION AS OF 5;
```

**History**

```sql
DESCRIBE HISTORY orders;
```

**Optimization**

```sql
OPTIMIZE orders;
```

**ZORDER**

```sql
OPTIMIZE orders
ZORDER BY (customer_id);
```

**Vacuum**

```sql
VACUUM orders DRY RUN;
```

### Remember

- Delta = Parquet data + transaction log.
- OPTIMIZE = file compaction.
- ZORDER = data skipping improvement.
- VACUUM = remove eligible unreferenced files.
- Time Travel requires the necessary transaction-log history and data files.
- VACUUM can make older versions unavailable.
- MERGE source records should be deduplicated deterministically.

---

## 6. Medallion Architecture

| Layer | Purpose |
|---|---|
| Bronze | Raw or minimally processed data |
| Silver | Cleaned, validated, standardized data |
| Gold | Business-ready analytical data |

```text
Source
  |
  v
Bronze
  |
  v
Silver
  |
  v
Gold
  |
  v
BI / Analytics
```

### Bronze

- Preserve source data.
- Maintain ingestion metadata.
- Support reprocessing.
- Keep raw history where required.

### Silver

- Remove or handle invalid records.
- Standardize data types.
- Deduplicate using business rules.
- Apply data-quality checks.
- Join and enrich data.

### Gold

- Build facts and dimensions.
- Apply business definitions.
- Create aggregations.
- Serve reports and analytical applications.

---

## 7. Azure Identity and Access Management

### Important Services

| Service | Purpose |
|---|---|
| Microsoft Entra ID | Identity and authentication |
| Azure RBAC | Role-based resource permissions |
| ADLS ACL | File and directory permissions |
| Managed Identity | Azure-managed application identity |
| Service Principal | Application identity |
| Key Vault | Secrets, keys, and certificates |

### Authentication vs Authorization

| Authentication | Authorization |
|---|---|
| Who are you? | What can you access? |
| Identity verification | Permission evaluation |
| Entra ID | RBAC and ACLs |

### Managed Identity

Types:
- System-assigned: Lifecycle tied to the Azure resource.
- User-assigned: Independent identity reusable across resources.

**Important:** Managed Identity authenticates a resource; it does not automatically grant access to storage or other services.

### RBAC vs ACL

| RBAC | ACL |
|---|---|
| Role-based permissions | File/directory-level permissions |
| Assigned at supported scopes | Assigned to filesystem paths |
| Controls broad resource/data access | Supports fine-grained access |

For ADLS Gen2, both RBAC and ACLs can affect access. Directory traversal requires execute permission on relevant parent directories.

### Key Vault

Used to store:
- Passwords.
- API keys.
- Connection strings.
- Certificates.
- Cryptographic keys.

Avoid hardcoding secrets in notebooks, source code, or pipeline parameters.

---

## 8. Azure Networking

| Concept | Quick Meaning |
|---|---|
| VNet | Private network in Azure |
| Subnet | Subdivision of a VNet |
| NSG | Filters network traffic |
| Azure Firewall | Managed network security service |
| Private Endpoint | Private IP connectivity to a supported service |
| Service Endpoint | Extends VNet identity to supported Azure services |
| VPN Gateway | Encrypted connectivity over VPN |
| ExpressRoute | Private connectivity through a connectivity provider |
| Load Balancer | Distributes network traffic |
| Private DNS | Resolves private service names |

### Private Endpoint vs Service Endpoint

| Private Endpoint | Service Endpoint |
|---|---|
| Uses a private IP in your VNet | Does not give the service a private IP |
| Uses Private Link | Extends VNet identity |
| Private DNS commonly required | DNS generally remains public service endpoint |
| Private connectivity to service | Network access restriction mechanism |

**Important:** A Private Endpoint does not grant data permissions. Identity and authorization are still required.

### NSG

- Filters inbound and outbound traffic.
- Rules use priorities.
- Lower priority number is evaluated first.
- Stateful: Return traffic for allowed connections is automatically permitted.

### Load Balancer Types

| Service | Layer |
|---|---|
| Azure Load Balancer | Layer 4 |
| Application Gateway | Layer 7 |
| Azure Front Door | Layer 7 global application delivery |

---

## 9. Azure Monitoring

| Service | Purpose |
|---|---|
| Azure Monitor | Collects and analyzes monitoring data |
| Log Analytics | Queries logs using KQL |
| Application Insights | Application performance monitoring |
| Azure Monitor Alerts | Notifies when configured conditions occur |
| Diagnostic Settings | Routes resource logs and metrics |

### What Should a Data Engineer Monitor?

**Operational Metrics**
- Pipeline success/failure.
- Execution duration.
- Retry count.
- Compute utilization.
- Job failures.

**Data Metrics**
- Row counts.
- Null counts.
- Duplicate records.
- Schema changes.
- Source-to-target reconciliation.

**Business Metrics**
- Revenue totals.
- Order counts.
- Customer counts.
- Data freshness.
- Business-rule violations.

### KQL Example

```kusto
AzureDiagnostics
| where TimeGenerated > ago(24h)
| summarize FailureCount = count() by Resource
| order by FailureCount desc
```

This is an illustrative query. Available tables and column names depend on diagnostic configuration and resource type.

---

## 10. Common Azure Data Engineering Architecture

```text
       Source Systems
       /     |      \
      DB    API    Files
       \     |      /
        Azure Data Factory
                |
                v
           ADLS Gen2
             Bronze
                |
                v
          Azure Databricks
             PySpark
                |
                v
             Silver
                |
                v
          Delta Lake / Gold
                |
                v
          Power BI / SQL
```

### Component Responsibilities

| Component | Responsibility |
|---|---|
| Source | Provides business data |
| ADF | Ingestion and orchestration |
| ADLS Gen2 | Data storage |
| Databricks | Distributed processing |
| PySpark | Transformations |
| Delta Lake | Transactional analytical tables |
| Unity Catalog | Governance and access control |
| Power BI | Reporting and visualization |

---

## 11. Common Interview Comparisons

| Comparison | Key Difference |
|---|---|
| Blob vs ADLS Gen2 | ADLS Gen2 enables hierarchical namespace |
| ADF vs Databricks | Orchestration/integration vs distributed processing |
| Dataset vs Linked Service | Data reference vs connection |
| Azure IR vs Self-hosted IR | Cloud-managed vs customer-managed runtime |
| Managed Identity vs Service Principal | Azure-managed identity vs application identity |
| RBAC vs ACL | Role-based access vs path-level permissions |
| Private Endpoint vs Service Endpoint | Private IP connectivity vs VNet identity extension |
| Parquet vs Delta | File format vs transactional table layer |
| OPTIMIZE vs VACUUM | Compaction vs removal of eligible unreferenced files |
| Bronze vs Silver vs Gold | Raw vs cleaned vs business-ready |

---

## 12. Common Interview Traps

1. ADLS Gen2 is not a completely separate storage engine; it is Blob Storage with HNS enabled.
2. ADF is not the same as Databricks. ADF orchestrates; Databricks processes data.
3. A storage URI does not authenticate a user.
4. Managed Identity does not automatically grant storage permissions.
5. RBAC and ACLs are not interchangeable.
6. A Private Endpoint does not automatically authorize data access.
7. Geo-replication is generally asynchronous.
8. A successful pipeline does not prove data correctness.
9. ADF retries should not blindly repeat non-idempotent writes.
10. Delta Lake is not simply another name for Parquet.
11. ZORDER is not a traditional database index.
12. VACUUM can remove files required for older Time Travel versions.
13. A watermark should not advance before successful target processing.
14. Timestamp-based incremental loads can miss late updates and deletes.
15. Availability Zones and Regions are different concepts.

---

## 13. Last-Minute Revision – 30 Key Facts

1. Azure = Cloud platform.
2. Region = Geographic area.
3. Availability Zone = Separate physical location.
4. Resource Group = Logical resource container.
5. ADLS Gen2 = Blob Storage + HNS.
6. HNS = Hierarchical Namespace.
7. `abfss://` = Secure ADLS Gen2 URI scheme.
8. ADF = Data integration and orchestration.
9. Pipeline = Group of activities.
10. Activity = Individual operation.
11. Linked Service = Connection.
12. Dataset = Data reference.
13. Integration Runtime = Connectivity and execution infrastructure.
14. Self-hosted IR = Private/on-premises connectivity.
15. Databricks = Managed data and AI platform.
16. Driver = Coordinates Spark execution.
17. Executor = Runs Spark tasks.
18. Delta = Parquet + transaction log.
19. MERGE = Insert/update/delete matching data.
20. OPTIMIZE = Compact files.
21. ZORDER = Improve data skipping.
22. VACUUM = Remove eligible unreferenced files.
23. Bronze = Raw.
24. Silver = Cleaned.
25. Gold = Business-ready.
26. Entra ID = Identity service.
27. RBAC = Role-based permissions.
28. ACL = Path-level permissions.
29. Key Vault = Secrets and keys.
30. Azure Monitor = Monitoring platform.

---

## 14. Final Interview Checklist

- [ ] Explain Azure service models.
- [ ] Explain regions and availability zones.
- [ ] Explain Blob Storage and ADLS Gen2.
- [ ] Explain storage redundancy options.
- [ ] Explain ADF components.
- [ ] Explain Integration Runtime types.
- [ ] Explain ADF incremental loading.
- [ ] Explain Databricks architecture.
- [ ] Explain Delta Lake and transaction logs.
- [ ] Explain MERGE, OPTIMIZE, ZORDER, and VACUUM.
- [ ] Explain Bronze, Silver, and Gold.
- [ ] Explain Entra ID, RBAC, ACL, and Managed Identity.
- [ ] Explain Private Endpoint and Service Endpoint.
- [ ] Explain Azure Monitor and Log Analytics.
- [ ] Design an end-to-end Azure data pipeline.

**Final reminder:** Focus on understanding each service's responsibility, when to use it, how it connects to other services, and what can go wrong in production.