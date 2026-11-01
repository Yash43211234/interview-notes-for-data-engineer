# Unity Catalog in Databricks

## 1. What Is Unity Catalog?

**Unity Catalog** is Databricks' centralized governance solution for managing, securing, discovering, and auditing data and AI assets.

It provides a unified way to manage access to data across supported Databricks workspaces.

With Unity Catalog, organizations can manage:

- Tables and views.
- Schemas and catalogs.
- Volumes and files.
- Data permissions.
- Data lineage.
- Auditing.
- External storage access.
- Data discovery.

**Simple way to remember:**

Unity Catalog = Centralized data governance + Access control + Data discovery + Lineage.

## 2. Why Do We Need Unity Catalog?

Suppose an organization has multiple teams working with different datasets.

- Data Engineers manage Bronze and Silver tables.
- Data Analysts work with Gold tables.
- Business Users access reporting data.
- Administrators manage security and governance.

Without centralized governance, managing permissions and tracking data across workspaces becomes difficult.

Unity Catalog helps solve these problems.

### Main Benefits

1. **Centralized governance:** Manage data assets through a unified governance model.
2. **Access control:** Control who can read, modify, or manage data.
3. **Data lineage:** Track where data comes from and where it goes.
4. **Data discovery:** Find and understand available data assets.
5. **Auditing:** Review supported access and activity events.
6. **External storage management:** Govern access to cloud storage.
7. **Scalability:** Apply consistent governance across supported workspaces.

## 3. Unity Catalog Architecture

Unity Catalog organizes data assets using a hierarchical structure.

```text
                 METASTORE
                     |
          +----------+----------+
          |                     |
          v                     v
       CATALOG A             CATALOG B
          |
          v
        SCHEMA
          |
    +-----+------+----------------+
    |            |                |
    v            v                v
  TABLE        VIEW            VOLUME
```

The main hierarchy is:

**Metastore → Catalog → Schema → Tables / Views / Volumes**

### 3.1 Metastore

A metastore is the top-level container for Unity Catalog governance.

It stores metadata and manages governance for supported data assets.

It provides a common governance layer for the workspaces attached to it.

### 3.2 Catalog

A catalog is the primary organizational container inside a metastore.

Organizations can create separate catalogs for different environments or business areas.

Example:

```text
production
development
finance
sales
```

### 3.3 Schema

A schema organizes related data objects inside a catalog.

Example:

```text
production
    |
    +-- bronze
    +-- silver
    +-- gold
```

### 3.4 Tables

Tables store structured data.

Example:

```sql
production.gold.daily_sales
```

### 3.5 Views

Views provide a logical representation of data using SQL queries.

Example:

```sql
CREATE VIEW production.gold.sales_summary AS
SELECT
    order_date,
    SUM(amount) AS total_sales
FROM production.silver.orders
GROUP BY order_date;
```

### 3.6 Volumes

Volumes provide governed access to files stored in cloud object storage.

They are useful for managing non-tabular data such as:

- CSV files.
- JSON files.
- Images.
- Documents.
- Other files used in data engineering workflows.

A volume can be managed or external.

## 4. Three-Level Namespace

Unity Catalog uses a three-level namespace to identify data objects.

```text
catalog.schema.table
```

Example:

```sql
SELECT *
FROM production.gold.daily_sales;
```

Here:

| Level | Value |
|---|---|
| Catalog | `production` |
| Schema | `gold` |
| Table | `daily_sales` |

Another example:

```sql
SELECT *
FROM main.bronze.orders;
```

This fully qualified name identifies the `orders` table inside the `bronze` schema of the `main` catalog.

**Interview point:** The three-level namespace helps uniquely identify data assets across catalogs and schemas.

## 5. Unity Catalog vs Hive Metastore

Databricks historically supported the Hive Metastore for organizing table metadata.

Unity Catalog provides a more centralized governance model.

| Feature | Hive Metastore | Unity Catalog |
|---|---|---|
| Governance | More limited and workspace-oriented | Centralized governance |
| Namespace | Catalog and schema conventions vary by environment | Three-level namespace |
| Cross-workspace governance | More limited | Supports shared governance across attached workspaces |
| Access control | Primarily workspace-oriented permissions | Fine-grained centralized permissions |
| Data lineage | Limited native governance capabilities | Built-in lineage capabilities for supported operations |
| Auditing | More limited | Centralized audit capabilities |
| External storage governance | Different configuration model | Storage credentials and external locations |

**Important:** Unity Catalog is the recommended governance solution for modern Databricks deployments. The two systems have different capabilities and should not be treated as interchangeable.

## 6. Managed Tables vs External Tables

Unity Catalog supports managed and external tables.

### 6.1 Managed Tables

For a managed table, Unity Catalog manages the table's metadata and data lifecycle.

Example:

```sql
CREATE TABLE main.gold.daily_sales (
    order_date DATE,
    total_sales DOUBLE
);
```

Databricks manages the table's storage location according to the configured managed storage.

When a managed table is dropped, its underlying data is also scheduled for deletion according to platform retention and cleanup behavior.

### 6.2 External Tables

An external table references data stored at a location managed outside the table's managed storage lifecycle.

Example:

```sql
CREATE TABLE main.bronze.orders
USING DELTA
LOCATION 'abfss://raw@storageaccount.dfs.core.windows.net/orders/';
```

The table metadata is registered in Unity Catalog, while the underlying data remains at the specified external location.

Dropping the external table removes its catalog registration but does not normally delete the underlying files.

### Comparison

| Feature | Managed Table | External Table |
|---|---|---|
| Metadata management | Unity Catalog | Unity Catalog |
| Data location | Managed storage | Specified external location |
| Data lifecycle | Managed by Databricks | Data managed separately |
| Drop table | Data is scheduled for deletion | Underlying files remain |
| Storage control | More Databricks-managed | More user-controlled |

**Remember:**

Managed table = Unity Catalog manages the table's data lifecycle.

External table = Unity Catalog manages metadata, but the external data location is managed separately.

## 7. Access Control in Unity Catalog

Unity Catalog uses privileges to control access to data objects.

Common privileges include:

| Privilege | Purpose |
|---|---|
| `USE CATALOG` | Access a catalog context |
| `USE SCHEMA` | Access a schema context |
| `SELECT` | Read table or view data |
| `MODIFY` | Modify table data |
| `CREATE TABLE` | Create tables |
| `CREATE SCHEMA` | Create schemas |
| `CREATE VIEW` | Create views |
| `READ VOLUME` | Read volume files |
| `WRITE VOLUME` | Write volume files |
| `MANAGE` | Manage privileges on supported objects |

### Example: Grant Read Access

```sql
GRANT USE CATALOG
ON CATALOG main
TO `data_analysts`;

GRANT USE SCHEMA
ON SCHEMA main.gold
TO `data_analysts`;

GRANT SELECT
ON SCHEMA main.gold
TO `data_analysts`;
```

This gives the group read access to supported tables and views in the Gold schema, subject to the applicable permission model.

### Example: Revoke Access

```sql
REVOKE SELECT
ON SCHEMA main.gold
FROM `data_analysts`;
```

**Important:** A user may still have access through another group, inherited privilege, ownership, or another grant.

## 8. Users, Groups, and Service Principals

Unity Catalog permissions can be assigned to different identities.

### Users

Individual people who access Databricks.

### Groups

Collections of users.

Example:

```text
data_engineers
data_analysts
finance_team
```

Groups make permission management easier.

### Service Principals

Application identities used by automated workloads.

Common use cases:

- Databricks Jobs.
- CI/CD pipelines.
- Automated data ingestion.
- Application-to-Databricks authentication.

**Best practice:** Use groups for team access and service principals for automated workloads.

## 9. Storage Credentials and External Locations

Unity Catalog provides a governed way to access cloud storage.

Two important concepts are:

1. Storage Credentials.
2. External Locations.

### 9.1 Storage Credentials

A storage credential represents an authentication mechanism used to access cloud storage.

Examples include supported:

- Azure managed identities.
- Azure service principals.
- AWS IAM roles.

### 9.2 External Locations

An external location combines:

- A cloud storage path.
- A storage credential.

Example:

```text
Storage Credential
        |
        v
External Location
        |
        v
ADLS Gen2 Container
        |
        v
Data Files
```

An external location can be used to govern access to supported external storage paths.

### Example

```sql
CREATE EXTERNAL LOCATION sales_data
URL 'abfss://raw@storageaccount.dfs.core.windows.net/'
WITH (STORAGE CREDENTIAL sales_storage_credential);
```

This is a conceptual example. The storage credential must already exist, and the user must have the required privileges.

### Common External Location Privileges

- `READ FILES`
- `WRITE FILES`
- `CREATE EXTERNAL TABLE`
- `CREATE EXTERNAL VOLUME`

**Important:** Unity Catalog permissions and cloud IAM permissions are separate security layers. Both may be needed.

## 10. Data Lineage in Unity Catalog

Data lineage tracks the origin, movement, and transformation of data.

Example:

```text
Bronze Layer
bronze.orders
      |
      v
Silver Layer
silver.orders
      |
      v
Gold Layer
gold.daily_sales
      |
      v
Power BI Dashboard
```

Unity Catalog can capture lineage for supported operations.

### Types of Lineage

**Table-level lineage**

Tracks relationships between tables.

Example:

```text
silver.orders
      |
      v
gold.daily_sales
```

**Column-level lineage**

Tracks how individual columns are derived.

Example:

```text
silver.orders.amount
          |
          v
      SUM(amount)
          |
          v
gold.daily_sales.total_sales
```

### Why Is Lineage Useful?

- Troubleshooting incorrect data.
- Understanding data dependencies.
- Impact analysis before schema changes.
- Auditing and traceability.
- Understanding upstream and downstream assets.

**Important:** Lineage capture depends on supported operations, data assets, and platform capabilities.

## 11. Auditing in Unity Catalog

Auditing helps organizations understand activities performed within Databricks.

Examples include:

- Who accessed a data asset?
- Which user or application performed an operation?
- When did a particular action happen?
- Which permissions or administrative activities occurred?

Databricks provides audit logs through supported audit system tables and platform logging capabilities, depending on workspace configuration and permissions.

### Example: Audit Use Cases

Suppose a sensitive customer table was accessed unexpectedly.

An administrator may investigate:

1. The identity that performed the action.
2. The time of access.
3. The operation performed.
4. The relevant data asset.
5. Whether the access was authorized.

**Important:** Audit logging and retention depend on the workspace, account configuration, cloud platform, and available system tables.

## 12. Row-Level and Column-Level Security

Unity Catalog supports fine-grained access-control capabilities for supported tables and configurations.

### Row-Level Security

Restricts which records a user can see.

Example:

```text
Sales Table

order_id | region | amount
---------------------------
101      | North  | 500
102      | South  | 700
103      | North  | 900
```

A North-region analyst should see only:

```text
order_id | region | amount
---------------------------
101      | North  | 500
103      | North  | 900
```

### Column-Level Security

Controls how sensitive column values are presented.

Example:

```text
customer_id | customer_name | phone
------------------------------------
501         | Rahul         | XXXXXXX210
502         | Priya         | XXXXXXX234
```

A column mask can hide or transform sensitive values for users who should not see the full information.

**Remember:**

- Row filter = Controls which rows are visible.
- Column mask = Controls how column values are displayed.

## 13. Unity Catalog Volumes

A volume is a governed object used to access files in cloud storage.

Unlike tables, which organize structured records, volumes are designed for file-based access.

Example:

```text
main
  |
  v
raw_files
  |
  v
customer_volume
  |
  +-- customers.csv
  +-- orders.json
  +-- documents/
```

### Types of Volumes

**Managed Volume**

Databricks manages the volume's storage location and lifecycle.

**External Volume**

The volume references files at an external storage location.

### Example: Read a File

```python
df = spark.read.csv(
    "/Volumes/main/raw_files/customer_volume/customers.csv",
    header=True,
    inferSchema=True
)
```

The volume path follows:

```text
/Volumes/<catalog>/<schema>/<volume>/<file>
```

### Why Are Volumes Useful?

- Govern file access.
- Organize non-tabular data.
- Support data engineering workflows.
- Provide controlled access to files through Unity Catalog.

## 14. Unity Catalog and Medallion Architecture

Unity Catalog can govern tables across the Bronze, Silver, and Gold layers.

Example:

```text
             UNITY CATALOG
                    |
                    v
                  main
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
      bronze      silver      gold
        |           |           |
        v           v           v
      Raw Data   Clean Data  Business Data
```

Example tables:

```text
main.bronze.orders
main.silver.orders
main.gold.daily_sales
```

Different permissions can be assigned to different teams.

| Team | Bronze | Silver | Gold |
|---|---|---|---|
| Data Engineers | Read/Write as required | Read/Write as required | Create/Modify as required |
| Data Analysts | Usually restricted | Read | Read |
| Business Users | No direct access | No direct access | Approved views |

This is an example governance design. Actual permissions should follow business requirements and least-privilege principles.

## 15. Practical Example: Creating a Governed Data Structure

Suppose we need to create a catalog for an e-commerce project.

### Step 1: Create a Catalog

```sql
CREATE CATALOG ecommerce;
```

### Step 2: Create Schemas

```sql
CREATE SCHEMA ecommerce.bronze;

CREATE SCHEMA ecommerce.silver;

CREATE SCHEMA ecommerce.gold;
```

### Step 3: Create a Bronze Table

```sql
CREATE TABLE ecommerce.bronze.orders (
    order_id INT,
    customer_id INT,
    amount DOUBLE
);
```

### Step 4: Create a Silver Table

```sql
CREATE TABLE ecommerce.silver.orders AS
SELECT DISTINCT
    order_id,
    customer_id,
    amount
FROM ecommerce.bronze.orders
WHERE amount IS NOT NULL;
```

### Step 5: Create a Gold Table

```sql
CREATE TABLE ecommerce.gold.daily_sales AS
SELECT
    SUM(amount) AS total_sales
FROM ecommerce.silver.orders;
```

### Step 6: Grant Analyst Access

```sql
GRANT USE CATALOG
ON CATALOG ecommerce
TO `data_analysts`;

GRANT USE SCHEMA
ON SCHEMA ecommerce.gold
TO `data_analysts`;

GRANT SELECT
ON SCHEMA ecommerce.gold
TO `data_analysts`;
```

Now the analyst group can read the Gold data, subject to the required privileges and applicable governance configuration.

## 16. Unity Catalog vs Delta Lake

These technologies are related but serve different purposes.

| Feature | Unity Catalog | Delta Lake |
|---|---|---|
| Main purpose | Data governance | Reliable table storage and transactions |
| Access control | Yes | Not its primary responsibility |
| Data lineage | Yes, for supported operations | Not its primary governance function |
| ACID transactions | Not its primary responsibility | Yes |
| Schema enforcement | Governance and metadata capabilities | Yes, for Delta tables |
| Time Travel | Can govern access to supported assets | Yes, through Delta transaction history |
| Storage management | Governed data and storage access | Manages Delta table data and transaction log |

### Simple Explanation

- Unity Catalog manages governance, permissions, discovery, and lineage.
- Delta Lake provides reliable table storage, ACID transactions, and Time Travel.

They work together in Databricks.

## 17. Best Practices

1. Use Unity Catalog for centralized data governance.
2. Organize data using meaningful catalogs and schemas.
3. Use fully qualified table names in production pipelines.
4. Follow the least-privilege principle.
5. Prefer group-based access management.
6. Use service principals for automated workloads.
7. Use storage credentials and external locations for governed cloud storage.
8. Monitor access through supported audit capabilities.
9. Review lineage before changing production tables.
10. Use row filters and column masks for sensitive data where appropriate.
11. Separate development, testing, and production assets.
12. Avoid exposing credentials in notebooks or source code.
13. Review ownership and permissions regularly.
14. Use managed tables when Databricks should manage the data lifecycle.
15. Use external tables when the data must remain at an externally managed storage location.

## 18. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Unity Catalog is a storage format | It is a centralized governance solution |
| Delta Lake and Unity Catalog are the same | Delta Lake manages reliable tables; Unity Catalog manages governance |
| Catalog is the top-level object | Metastore is the top-level governance container |
| Schema and catalog are identical | A schema belongs to a catalog |
| Managed and external tables behave identically | Their data lifecycle and storage management differ |
| External tables delete underlying files when dropped | Dropping the registration normally leaves external data files |
| Unity Catalog automatically tracks every external operation | Lineage capture has supported-operation boundaries |
| `SELECT` alone always gives table access | Parent usage privileges may also be required |
| Storage credentials and secret scopes are identical | They have different responsibilities |
| Volumes are the same as tables | Volumes govern file access; tables organize structured data |
| Unity Catalog replaces Delta Lake | They complement each other |

## 19. Interview Questions and Answers

**Q1. What is Unity Catalog?**

Unity Catalog is Databricks' centralized governance solution for managing, securing, discovering, and auditing data and AI assets.

**Q2. Why do we need Unity Catalog?**

It helps manage permissions, lineage, auditing, data discovery, and external storage access across supported workspaces.

**Q3. Explain the Unity Catalog hierarchy.**

The hierarchy is Metastore → Catalog → Schema → Tables, Views, Volumes, and other supported objects.

**Q4. What is a metastore?**

A metastore is the top-level container that manages metadata and governance for Unity Catalog assets.

**Q5. What is a catalog?**

A catalog is an organizational container inside a metastore that contains schemas.

**Q6. What is a schema?**

A schema organizes related tables, views, volumes, and other objects inside a catalog.

**Q7. What is the three-level namespace?**

It identifies a data object using:

```text
catalog.schema.table
```

For example:

```sql
SELECT *
FROM main.gold.daily_sales;
```

**Q8. What is the difference between Unity Catalog and Hive Metastore?**

Unity Catalog provides centralized governance, fine-grained permissions, lineage, and cross-workspace governance. Hive Metastore has a more limited, traditionally workspace-oriented governance model.

**Q9. What is the difference between managed and external tables?**

For managed tables, Databricks manages the table's data lifecycle. For external tables, Unity Catalog manages the metadata while the data remains at an externally managed location.

**Q10. What is an external location?**

An external location associates a cloud storage path with a storage credential and provides a governed way to access external storage.

**Q11. What is a storage credential?**

It represents an authentication mechanism used by Unity Catalog to access cloud storage.

**Q12. What is a volume?**

A volume is a Unity Catalog object used to govern access to files stored in cloud storage.

**Q13. What is data lineage?**

Data lineage tracks the origin, movement, and transformation of data across supported data assets and operations.

**Q14. What is the difference between upstream and downstream lineage?**

Upstream shows where data comes from. Downstream shows which assets depend on the data.

**Q15. What is row-level security?**

It restricts the rows a user can access based on an access policy.

**Q16. What is column-level security?**

It controls access to sensitive column values, often through masking.

**Q17. What is the difference between Unity Catalog and Delta Lake?**

Unity Catalog handles governance and access control. Delta Lake provides reliable table storage, ACID transactions, and Time Travel.

**Q18. Can Unity Catalog be shared across multiple workspaces?**

Yes. A metastore can be assigned to multiple supported workspaces, allowing centralized governance, subject to workspace and configuration requirements.

**Q19. How would you give analysts access only to Gold tables?**

I would create or use an analyst group, grant `USE CATALOG` and `USE SCHEMA` on the required objects, and grant `SELECT` on the approved Gold tables or schema.

**Q20. How would you investigate a data-quality issue using Unity Catalog?**

I would inspect the affected table's upstream lineage, identify source tables and transformations, review relevant jobs and data-quality checks, and trace where the incorrect data entered the pipeline.

## 20. Quick Revision

| Concept | Remember |
|---|---|
| Unity Catalog | Centralized governance |
| Metastore | Top-level governance container |
| Catalog | Contains schemas |
| Schema | Contains data objects |
| Three-level namespace | `catalog.schema.table` |
| Managed Table | Databricks-managed data lifecycle |
| External Table | Data remains at external location |
| Volume | Governed file access |
| Storage Credential | Cloud-storage authentication |
| External Location | Storage path + credential |
| GRANT | Provides privileges |
| REVOKE | Removes privileges |
| Data Lineage | Tracks data movement and transformations |
| Upstream | Data sources |
| Downstream | Data consumers |
| Row Filter | Restricts visible rows |
| Column Mask | Controls displayed values |
| Audit Logs | Record supported activities |
| Delta Lake | ACID transactions and reliable tables |

### One-Minute Interview Answer

Unity Catalog is Databricks' centralized governance solution. It helps organizations manage access control, data discovery, lineage, auditing, and cloud storage access.

Its hierarchy is Metastore, Catalog, Schema, and then objects such as Tables, Views, and Volumes. It uses a three-level namespace like `catalog.schema.table`.

We can manage permissions using privileges such as `SELECT`, `MODIFY`, `USE CATALOG`, and `USE SCHEMA`. For external storage, Unity Catalog provides storage credentials and external locations.

It also supports data lineage and fine-grained access-control capabilities such as row filters and column masks.

Unity Catalog works alongside Delta Lake: Unity Catalog manages governance, while Delta Lake provides reliable table storage and ACID transactions.

## 21. Self-Assessment

- [ ] Define Unity Catalog.
- [ ] Explain why centralized governance is required.
- [ ] Draw the Unity Catalog architecture.
- [ ] Explain metastore, catalog, and schema.
- [ ] Explain the three-level namespace.
- [ ] Differentiate Unity Catalog and Hive Metastore.
- [ ] Differentiate managed and external tables.
- [ ] Explain Unity Catalog permissions.
- [ ] Explain storage credentials and external locations.
- [ ] Explain data lineage and auditing.
- [ ] Explain row-level and column-level security.
- [ ] Explain managed and external volumes.
- [ ] Differentiate Unity Catalog and Delta Lake.
- [ ] Create a catalog and schemas using SQL.
- [ ] Grant read access to an analyst group.
- [ ] Explain how Unity Catalog supports the medallion architecture.

**Interview readiness:** You should be able to explain Unity Catalog's architecture, compare it with Delta Lake, describe managed versus external tables, and demonstrate how you would govern access to Bronze, Silver, and Gold data in a production environment.