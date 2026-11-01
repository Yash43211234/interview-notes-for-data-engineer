# Azure SQL Database

## 1. What Is Azure SQL Database?

**Azure SQL Database** is a fully managed relational database service provided by Microsoft Azure. It is based on the Microsoft SQL Server database engine.

It allows applications and data engineering systems to store, query, and manage relational data without managing the underlying database servers, operating system, or most routine maintenance tasks.

It supports:
- SQL queries and relational data.
- Tables, views, stored procedures, and indexes.
- Transactions and constraints.
- Automated backups.
- High availability.
- Security and access control.
- Scaling compute and storage according to the selected service tier.

**Simple explanation:** Azure SQL Database is a cloud-hosted SQL database where Microsoft manages much of the infrastructure and database maintenance.

---

## 2. Why Do We Need Azure SQL Database?

Consider an e-commerce company that stores:

- Customer information.
- Product details.
- Orders.
- Payments.
- Inventory.

The company needs a relational database to manage this structured information and support transactional queries.

Instead of installing and maintaining SQL Server on its own virtual machines, the company can use Azure SQL Database.

### Basic architecture

```text
       Application / Data Pipeline
                   |
                   v
            Azure SQL Database
                   |
          +--------+--------+
          |        |        |
          v        v        v
       Customers  Orders  Products
          |        |        |
          +--------+--------+
                   |
                   v
             SQL Queries
```

### Common use cases

- Application databases.
- Transactional workloads.
- Customer and order management.
- Operational reporting.
- Staging data for ETL pipelines.
- Relational data integration.
- Serving structured data to applications and analytics systems.

---

## 3. Azure SQL Database Architecture

Azure SQL Database is a Platform as a Service (PaaS) offering.

Microsoft manages much of the underlying infrastructure, including:
- Database engine infrastructure.
- Routine patching and maintenance.
- Automated backups.
- High-availability mechanisms.
- Infrastructure monitoring.

The customer remains responsible for:
- Database schema design.
- Query and index design.
- Data modeling.
- User access and permissions.
- Application logic.
- Data governance.
- Workload and cost management.

### Logical architecture

```text
              Client Application
                     |
                 SQL Client
                     |
                     v
             Azure SQL Database
                     |
              SQL Database Engine
                     |
          +----------+----------+
          |          |          |
          v          v          v
        Tables     Indexes   Procedures
          |          |          |
          +----------+----------+
                     |
                     v
               Database Storage
```

**Important:** Azure SQL Database is a managed database service. You do not normally manage the underlying SQL Server operating system or physical database servers.

---

## 4. Azure SQL Database vs SQL Server vs SQL Managed Instance

These three options are related but provide different levels of management and compatibility.

### 4.1 SQL Server on Azure Virtual Machines

SQL Server runs on an Azure Virtual Machine.

You manage:
- Operating system.
- SQL Server installation and configuration.
- Patching and maintenance.
- Backup configuration.
- High availability setup.
- Database performance and tuning.

This provides the most infrastructure-level control.

### 4.2 Azure SQL Managed Instance

Azure SQL Managed Instance is a managed PaaS database service with broad SQL Server compatibility.

It is useful when an organization wants to migrate existing SQL Server databases to Azure with fewer application changes.

It supports many SQL Server features that may not be available in Azure SQL Database.

### 4.3 Azure SQL Database

Azure SQL Database is a managed relational database service designed for modern cloud applications.

It provides database-level management without requiring customers to manage the underlying SQL Server infrastructure.

### Comparison

| Feature | Azure SQL Database | SQL Managed Instance | SQL Server on Azure VM |
|---|---|---|---|
| Service type | PaaS | PaaS | IaaS |
| Infrastructure management | Microsoft | Microsoft | Customer |
| OS access | No | No | Yes |
| SQL Server compatibility | Database-level compatibility | Broad instance-level compatibility | Full SQL Server control |
| Instance-level features | Limited compared with SQL Server | Broad support | Full control |
| Maintenance | Mostly managed | Mostly managed | Customer-managed |
| Typical use | Cloud applications | SQL Server migration | Full infrastructure control |

**Interview point:** Choose Azure SQL Database for cloud-native relational applications, Managed Instance for broader SQL Server compatibility, and SQL Server on an Azure VM when full infrastructure control is required.

---

## 5. Deployment and Purchasing Models

Azure SQL Database offers different deployment options.

### 5.1 Single Database

A single database is an isolated database resource with its own compute and storage configuration.

Suitable for:
- Individual application databases.
- Independent workloads.
- Applications with separate performance requirements.

### 5.2 Elastic Pool

An elastic pool allows multiple databases to share a pool of compute resources.

It is useful when several databases have varying or unpredictable resource usage.

Example:

```text
             Elastic Pool
                  |
          Shared Compute Resources
                  |
       +----------+----------+
       |          |          |
       v          v          v
     DB_One    DB_Two     DB_Three
```

Instead of assigning separate dedicated compute capacity to every database, the databases share resources within the pool.

**Example:** A SaaS application has 100 customer databases. Most customers use very little compute, while a few experience temporary spikes. An elastic pool can be more cost-effective than independently provisioning large compute capacity for every database.

### 5.3 Serverless Compute Tier

The serverless compute tier automatically scales compute resources based on workload demand.

Characteristics:
- Automatic compute scaling.
- Can pause after an inactivity period when configured and supported.
- Compute billing based on actual usage under its pricing model.
- Suitable for intermittent or unpredictable workloads.
- Resume activity may introduce latency.

Serverless is a compute option within the applicable Azure SQL Database purchasing model, not a separate database engine.

### 5.4 Provisioned Compute Tier

In the provisioned model, compute resources are allocated in advance.

Suitable for:
- Consistent workloads.
- Predictable performance requirements.
- Applications with steady database usage.

### Comparison

| Model | Main idea | Suitable workload |
|---|---|---|
| Single database | Independent database resource | Individual applications |
| Elastic pool | Multiple databases share compute | Many databases with variable demand |
| Serverless | Compute adjusts to workload | Intermittent usage |
| Provisioned | Compute allocated in advance | Consistent workloads |

---

## 6. Compute and Service Tiers

Azure SQL Database provides different purchasing and service-tier options.

### 6.1 DTU-Based Purchasing Model

DTU stands for **Database Transaction Unit**.

It is a bundled measure of compute, memory, and data I/O resources.

The DTU model provides predefined performance levels.

It is a simplified way to choose database performance capacity.

### 6.2 vCore-Based Purchasing Model

The vCore model provides more control over compute and storage configuration.

You choose resources such as:
- Number of virtual cores.
- Memory and compute characteristics associated with the selected hardware.
- Storage capacity.
- Service tier.

It also provides clearer separation between compute and storage costs.

### 6.3 General Purpose

The General Purpose tier is designed for typical business workloads.

It balances cost and performance and is suitable for many standard application databases.

### 6.4 Business Critical

The Business Critical tier is designed for workloads requiring high performance and low latency.

It uses a different storage and availability architecture from General Purpose.

### 6.5 Hyperscale

Hyperscale is designed for very large databases and workloads requiring scalable storage architecture.

It supports database sizes and scaling characteristics beyond typical smaller database workloads.

### Service-tier comparison

| Tier | Main focus | Common use |
|---|---|---|
| General Purpose | Balanced cost and performance | Standard applications |
| Business Critical | High performance and low latency | Performance-sensitive applications |
| Hyperscale | Large-scale database storage and scalability | Very large databases |

**Note:** Exact limits, pricing, availability, and feature support depend on the region, hardware generation, purchasing model, and current Azure offering.

---

## 7. Connecting to Azure SQL Database

Applications and data engineering tools can connect to Azure SQL Database using supported SQL drivers and authentication methods.

### Common connection options

- SQL Server Management Studio (SSMS).
- Azure Data Studio, where supported.
- Python applications.
- Azure Data Factory.
- Azure Databricks.
- Power BI.
- Other SQL clients and applications.

### Connection details

A typical connection requires:

| Parameter | Meaning |
|---|---|
| Server name | Azure SQL logical server hostname |
| Database name | Target database |
| Authentication | Identity or authentication method |
| Username | Required for SQL authentication |
| Password | Required for SQL authentication |
| Encryption | Secure connection configuration |

Example server name:

```text
myserver.database.windows.net
```

Example Python connection using `pyodbc`:

```python
import pyodbc

connection_string = (
    "Driver={ODBC Driver 18 for SQL Server};"
    "Server=tcp:myserver.database.windows.net,1433;"
    "Database=SalesDB;"
    "Authentication=ActiveDirectoryInteractive;"
    "Encrypt=yes;"
    "TrustServerCertificate=no;"
)

connection = pyodbc.connect(connection_string)

cursor = connection.cursor()

cursor.execute(
    "SELECT TOP 10 * FROM dbo.Customers"
)

for row in cursor.fetchall():
    print(row)

connection.close()
```

This example uses interactive Microsoft Entra authentication. The required ODBC driver and authentication configuration must be installed and available in the execution environment.

For production workloads, use an appropriate non-interactive identity and secure credential management.

---

## 8. Authentication and Security

Azure SQL Database supports multiple authentication approaches.

### 8.1 SQL Authentication

SQL authentication uses a database login and password.

It is useful for applications that require SQL-based authentication, but credentials must be securely stored and managed.

### 8.2 Microsoft Entra Authentication

Microsoft Entra ID authentication allows users and applications to authenticate using Entra identities.

It supports centralized identity management and integration with organizational access policies.

### 8.3 Managed Identity

Managed Identity allows supported Azure services to authenticate to Azure SQL Database without storing passwords in application code.

Example:

```text
          Azure Data Factory
                  |
            Managed Identity
                  |
                  v
           Microsoft Entra ID
                  |
                  v
          Azure SQL Database
                  |
          Database Permissions
                  |
                  v
              Data Access
```

The identity must be configured as a database user and granted the required permissions.

**Important:** A managed identity authenticates the workload. It does not automatically grant database access.

### 8.4 Network Security

Azure SQL Database supports network security controls such as:

- Server-level firewall rules.
- Database-level firewall rules.
- Private endpoints.
- Virtual network integration options.
- Microsoft Entra authentication.

### 8.5 Encryption

Important security features include:

- Encryption in transit using TLS.
- Transparent Data Encryption (TDE) for data at rest.
- Always Encrypted for supported client-side column encryption scenarios.
- Dynamic Data Masking for limiting exposure of selected data.
- Auditing and threat detection capabilities.

These features solve different security problems; they are not interchangeable.

---

## 9. Firewall and Private Endpoint

### Firewall Rules

Azure SQL Database uses firewall rules to control which network sources can connect.

A connection may fail when the client's IP address is not allowed by the applicable firewall configuration.

### Private Endpoint

A private endpoint provides a private IP address in an Azure Virtual Network for accessing Azure SQL Database through Azure Private Link.

It helps keep database connectivity on private network paths.

### Architecture

```text
          Azure Virtual Network
                   |
              Private IP
                   |
                   v
             Private Endpoint
                   |
                   v
           Azure SQL Database
```

**Important:** A private endpoint does not automatically grant database permissions. Authentication and authorization are still required.

---

## 10. Backup and High Availability

Azure SQL Database provides automated backup and high-availability capabilities.

### 10.1 Automated Backups

Azure SQL Database automatically creates backups according to the service's backup policy.

These backups support:
- Point-in-time restore.
- Long-term retention when configured.
- Recovery from accidental changes or deletion.

### 10.2 Point-in-Time Restore

Point-in-time restore allows a database to be restored to an earlier point within the available backup retention period.

Example:

```text
10:00 AM  Database working
11:00 AM  Incorrect UPDATE executed
12:00 PM  Problem discovered
                  |
                  v
          Point-in-Time Restore
                  |
                  v
       Restore to before 11:00 AM
```

The restore operation generally creates a new database rather than directly rewinding the existing database in place.

### 10.3 High Availability

Azure SQL Database includes built-in high-availability capabilities designed to reduce downtime during infrastructure failures and maintenance.

The exact architecture and availability behavior depend on the selected service tier.

### 10.4 Geo-Replication

Active geo-replication allows databases to replicate data to secondary databases in other Azure regions.

It can support:
- Disaster recovery.
- Regional resilience.
- Read-only secondary workloads, where configured.

Replication is generally asynchronous, so some data loss may occur during failover.

### Important distinction

| Feature | Purpose |
|---|---|
| Automated backup | Recovery from accidental changes or data loss |
| Point-in-time restore | Restore a database to an earlier time |
| High availability | Reduce downtime during failures |
| Geo-replication | Maintain a secondary database in another region |
| Failover group | Manage replication and failover for one or more databases |

High availability and disaster recovery are related but different goals.

---

## 11. Performance Optimization

Database performance depends on query design, indexing, workload characteristics, and available resources.

### 11.1 Indexing

Indexes help SQL Server locate rows efficiently.

Example:

```sql
CREATE INDEX idx_orders_customer_id
ON dbo.Orders(customer_id);
```

This index can help queries that filter or join using `customer_id`.

However, unnecessary indexes increase storage and write overhead.

### 11.2 Query Optimization

Avoid retrieving unnecessary columns.

Less efficient:

```sql
SELECT *
FROM dbo.Orders;
```

More targeted:

```sql
SELECT
    order_id,
    customer_id,
    order_date,
    amount
FROM dbo.Orders
WHERE order_date >= '2026-01-01';
```

Other optimization practices:
- Filter rows early.
- Use appropriate joins.
- Avoid unnecessary conversions on filtered columns.
- Review execution plans.
- Avoid repeated expensive queries.
- Use suitable indexes.
- Monitor CPU, memory, and data I/O.

### 11.3 Query Performance Monitoring

Useful tools include:
- Query Store.
- Execution plans.
- Azure Monitor metrics.
- Dynamic Management Views (DMVs).
- Automatic tuning features where supported.

### 11.4 Query Store

Query Store captures query, plan, and runtime-statistics history.

It helps identify:
- Regressed queries.
- Changing execution plans.
- High-resource queries.
- Performance trends.

### 11.5 Automatic Tuning

Azure SQL Database provides automatic tuning capabilities for supported scenarios.

Depending on configuration and availability, it can help identify or apply certain performance improvements.

Do not assume automatic tuning replaces proper query and index design.

---

## 12. Azure SQL Database in Data Engineering

Azure SQL Database can act as a relational source, staging database, or serving layer in a data engineering architecture.

### Example architecture

```text
             Application
                  |
                  v
          Azure SQL Database
                  |
                  v
          Azure Data Factory
                  |
                  v
             ADLS Gen2
                  |
                  v
          Azure Databricks
                  |
          Transform / Clean
                  |
                  v
           Delta Lake Tables
                  |
                  v
             Power BI
```

### Responsibilities of each service

| Service | Responsibility |
|---|---|
| Azure SQL Database | Store relational application data |
| Azure Data Factory | Extract and orchestrate data movement |
| ADLS Gen2 | Store raw and processed files |
| Azure Databricks | Perform large-scale transformations |
| Delta Lake | Provide reliable analytical tables |
| Power BI | Build reports and dashboards |

### Example scenario

An e-commerce company stores orders in Azure SQL Database.

The data engineering team needs to build a daily analytics pipeline.

1. ADF connects to Azure SQL Database.
2. ADF extracts new or changed records.
3. Raw data is stored in ADLS Gen2.
4. Databricks reads the raw data.
5. PySpark performs cleaning and transformations.
6. Curated data is written to Delta tables.
7. Power BI uses the curated data for reporting.

Azure SQL Database acts as the operational relational source, while Databricks performs the large-scale analytical processing.

---

## 13. Azure SQL Database vs Azure Synapse Analytics

These services are designed for different workload patterns.

| Feature | Azure SQL Database | Azure Synapse Analytics |
|---|---|---|
| Main purpose | Relational application database | Analytics and data warehousing |
| Workload | OLTP-oriented | OLAP-oriented |
| Query patterns | Frequent transactional operations | Large analytical queries |
| Data | Relational application data | Large-scale analytical data |
| Common use | Orders, customers, applications | Reporting, analytics, warehousing |
| Architecture | Managed relational database | Integrated analytics platform |

### Example

**Azure SQL Database:**

```sql
INSERT INTO dbo.Orders
(
    order_id,
    customer_id,
    amount
)
VALUES
(
    101,
    5001,
    2500
);
```

This represents a transactional operation.

**Analytical workload:**

```sql
SELECT
    customer_id,
    SUM(amount) AS total_spent
FROM dbo.Orders
GROUP BY customer_id;
```

This query aggregates order data for analysis.

The query itself can run in Azure SQL Database, but large-scale analytical workloads may be better suited to a dedicated analytical platform.

---

## 14. Common Errors and Troubleshooting

| Error or problem | Possible reason | What to check |
|---|---|---|
| Cannot connect to server | Incorrect server name or network issue | Connection details and connectivity |
| Login failed | Incorrect credentials or authentication setup | Login and identity configuration |
| Firewall blocked | Client IP not allowed | Firewall rules |
| Permission denied | Database user lacks required permission | Database roles and grants |
| Query is slow | Missing index or inefficient query | Execution plan and Query Store |
| DTU/vCore utilization is high | Insufficient capacity or inefficient workload | Resource metrics and query performance |
| Connection timeout | Network issue or resource pressure | Network, database health, query duration |
| Private endpoint connection fails | DNS or network configuration problem | Private DNS, VNet, endpoint status |

### Troubleshooting sequence

1. Verify the server and database names.
2. Confirm network connectivity.
3. Check firewall or private endpoint configuration.
4. Verify authentication.
5. Check database-level permissions.
6. Inspect query execution plans.
7. Review Query Store and Azure Monitor.
8. Check resource utilization.

---

## 15. Common Interview Questions and Answers

### Q1. What is Azure SQL Database?

Azure SQL Database is a fully managed relational database service in Azure, based on the SQL Server database engine.

### Q2. Is Azure SQL Database PaaS or IaaS?

It is a Platform as a Service (PaaS) offering.

### Q3. What is the difference between Azure SQL Database and SQL Server on an Azure VM?

Azure SQL Database is managed by Microsoft at the infrastructure level, whereas SQL Server on an Azure VM requires the customer to manage the operating system and much of the database infrastructure.

### Q4. What is Azure SQL Managed Instance?

It is a managed SQL database service that provides broad SQL Server compatibility, making it useful for migrating existing SQL Server workloads.

### Q5. What is an elastic pool?

An elastic pool allows multiple Azure SQL databases to share compute resources, making it useful when individual databases have variable resource usage.

### Q6. What is the difference between DTU and vCore?

DTU combines compute, memory, and I/O into a bundled performance measure. The vCore model provides more direct control over compute and storage configuration.

### Q7. What is the difference between General Purpose and Business Critical?

General Purpose targets balanced cost and performance. Business Critical targets workloads requiring higher performance and lower latency.

### Q8. What is Hyperscale?

Hyperscale is an Azure SQL Database service tier designed for very large databases and scalable storage requirements.

### Q9. How does Azure SQL Database support security?

It supports Microsoft Entra authentication, SQL authentication, managed identities, firewall rules, private endpoints, encryption, auditing, and database permissions.

### Q10. What is a private endpoint?

A private endpoint provides private network connectivity to an Azure service through a private IP address in a virtual network.

### Q11. What is Point-in-Time Restore?

It allows a database to be restored to an earlier point within the available backup retention period.

### Q12. What is Query Store?

Query Store stores query and execution-plan history along with runtime statistics to help analyze performance and troubleshoot query regressions.

### Q13. How can ADF connect to Azure SQL Database?

ADF can use an Azure SQL Database linked service with an appropriate authentication method and Integration Runtime configuration.

### Q14. How can Azure Databricks access Azure SQL Database?

Databricks can connect using supported JDBC drivers and an appropriate authentication method. Credentials should be managed securely.

### Q15. Can Azure SQL Database replace Azure Databricks?

No. Azure SQL Database is a relational database service, while Databricks provides distributed data processing and analytics capabilities.

---

## 16. Common Mistakes to Avoid

1. Confusing Azure SQL Database with SQL Server installed on a VM.
2. Assuming Azure SQL Database provides full instance-level SQL Server compatibility.
3. Confusing Azure SQL Database with Azure Synapse Analytics.
4. Thinking an elastic pool combines multiple databases into one database.
5. Assuming DTU and vCore are the same purchasing model.
6. Thinking a private endpoint automatically grants database access.
7. Assuming managed identity automatically has database permissions.
8. Confusing backup with high availability.
9. Assuming geo-replication is always synchronous.
10. Creating too many indexes without considering write overhead.
11. Using Azure SQL Database as the primary engine for massive distributed transformations.
12. Assuming Microsoft manages database schema, query optimization, and application-level security decisions.

---

## 17. Quick Revision

| Concept | Remember |
|---|---|
| Azure SQL Database | Managed relational database service |
| Service model | PaaS |
| Single database | Independent database resource |
| Elastic pool | Shared compute across databases |
| Serverless | Compute adjusts to workload |
| Provisioned | Compute allocated in advance |
| DTU | Bundled performance measure |
| vCore | Compute-focused purchasing model |
| General Purpose | Balanced performance and cost |
| Business Critical | High performance and low latency |
| Hyperscale | Very large database workloads |
| Managed Identity | Secure identity-based authentication |
| Private Endpoint | Private network connectivity |
| TDE | Encryption at rest |
| Point-in-Time Restore | Restore to an earlier time |
| Query Store | Query and plan performance history |
| Geo-replication | Secondary database in another region |

### Remember these five points

1. Azure SQL Database is a managed relational PaaS service.
2. It is primarily suited to transactional relational workloads.
3. Elastic pools allow multiple databases to share compute.
4. Managed Identity, firewall rules, and private endpoints help secure access.
5. ADF can extract data from Azure SQL Database, while Databricks can perform large-scale transformations.

---

## 18. Self-Assessment

Try answering these questions without looking at the notes:

1. What is Azure SQL Database?
2. Why is it classified as PaaS?
3. Explain Azure SQL Database vs SQL Managed Instance vs SQL Server on an Azure VM.
4. What is an elastic pool?
5. What is the difference between DTU and vCore?
6. Explain General Purpose, Business Critical, and Hyperscale.
7. How can an application connect to Azure SQL Database?
8. How does Managed Identity work with Azure SQL Database?
9. What is the purpose of a private endpoint?
10. Explain automated backups and Point-in-Time Restore.
11. What is the difference between high availability and disaster recovery?
12. What is Query Store?
13. How can ADF extract data from Azure SQL Database?
14. How can Databricks connect to Azure SQL Database?
15. When would you choose Azure SQL Database instead of Synapse Analytics?

**Interview preparation priority:** Focus on Azure SQL Database vs Managed Instance vs SQL Server on VM, elastic pools, DTU vs vCore, service tiers, authentication, private endpoints, backup and restore, Query Store, and integration with ADF and Databricks.