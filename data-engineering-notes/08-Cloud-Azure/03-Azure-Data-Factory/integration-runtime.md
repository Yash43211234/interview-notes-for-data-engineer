# Azure Data Factory: Integration Runtime

## 1. What Is Integration Runtime?

**Integration Runtime (IR)** is the compute infrastructure used by Azure Data Factory to provide data integration capabilities across different network environments.

It provides the connectivity and execution environment required for supported ADF activities.

It is used for:

- Moving data between supported data stores.
- Connecting to cloud and on-premises data sources.
- Dispatching transformation activities.
- Executing SSIS packages through the Azure-SSIS Integration Runtime.

### Simple Example

Suppose customer data is stored in an on-premises SQL Server database and must be copied to Azure Data Lake Storage Gen2.

```text
On-Premises SQL Server
          |
          v
  Self-hosted IR
          |
          v
  Azure Data Factory
          |
          v
      ADLS Gen2
```

The Self-hosted Integration Runtime provides the connectivity needed to access the on-premises source.

**Important:** Integration Runtime is not a dataset, linked service, or pipeline. It provides the integration infrastructure used by supported activities.

---

## 2. Why Do We Need Integration Runtime?

Data sources can exist in different environments:

- Azure cloud.
- Other cloud platforms.
- On-premises data centers.
- Private virtual networks.
- Hybrid environments.

ADF needs an appropriate execution and connectivity environment to access these systems.

Integration Runtime helps ADF perform these operations.

### Main Responsibilities

1. **Data Movement:** Provides the infrastructure for supported copy operations.
2. **Connectivity:** Connects to supported data sources and destinations.
3. **Transformation Dispatch:** Provides the environment for supported transformation activities.
4. **Network Access:** Supports private and on-premises connectivity through Self-hosted IR.
5. **SSIS Execution:** Runs SSIS packages through Azure-SSIS IR.

---

## 3. Types of Integration Runtime

Azure Data Factory supports three main Integration Runtime types:

1. Azure Integration Runtime.
2. Self-hosted Integration Runtime.
3. Azure-SSIS Integration Runtime.

<AsyncImageGroup query={["Azure Data Factory Azure Integration Runtime architecture diagram","Azure Data Factory self hosted integration runtime on premises architecture diagram","Azure Data Factory Azure SSIS integration runtime architecture diagram"]} aspectRatio="5:4" layout="carousel"/>

### 3.1 Azure Integration Runtime

Azure Integration Runtime is a fully managed integration infrastructure provided by Azure.

ADF manages the underlying infrastructure.

Common use cases:

- Copying data between cloud data stores.
- Connecting to supported cloud services.
- Running supported Mapping Data Flow activities.
- Dispatching supported external transformation activities.

### Example

Copy data from Azure SQL Database to ADLS Gen2.

```text
Azure SQL Database
        |
        v
  Azure Integration
      Runtime
        |
        v
      ADLS Gen2
```

### Characteristics

- Managed by Azure.
- No need to install or maintain a local runtime machine.
- Supports cloud data integration.
- Can be configured with networking options for supported private connectivity scenarios.

**Interview point:** Azure IR is commonly used for cloud-to-cloud data movement.

---

### 3.2 Self-hosted Integration Runtime

Self-hosted Integration Runtime is installed on a machine managed by the organization.

It is commonly used when ADF needs to access data sources inside a private network or on-premises environment.

### Example

A company has a SQL Server database inside its office network.

ADF cannot directly access that private database from the public cloud.

A Self-hosted IR can be installed on a machine that has network access to SQL Server.

```text
On-Premises Network
+-------------------------+
|                         |
|   SQL Server            |
|       |                 |
|       v                 |
|   Self-hosted IR        |
|                         |
+-------------------------+
          |
          | Outbound connection
          v
   Azure Data Factory
          |
          v
       ADLS Gen2
```

### Characteristics

- Installed on a Windows machine in the organization's environment.
- Managed by the organization.
- Can access supported on-premises and private-network data sources.
- Requires network connectivity and appropriate firewall configuration.
- Can be configured with multiple nodes for availability and scalability.

### Important

The Self-hosted IR machine must be able to reach the source and destination systems required by the activity.

It commonly establishes outbound communication with Azure rather than requiring Azure to initiate an inbound connection to the machine.

### Multiple Nodes

For production workloads, multiple Self-hosted IR nodes can be configured in a shared runtime.

Benefits:

- Higher availability.
- Better workload distribution.
- Reduced dependency on a single machine.

**Interview point:** Self-hosted IR is used when data sources are not directly reachable through the available cloud integration environment.

---

### 3.3 Azure-SSIS Integration Runtime

Azure-SSIS Integration Runtime is used to run existing SQL Server Integration Services (SSIS) packages in Azure Data Factory.

It is useful when an organization wants to migrate existing SSIS workloads to Azure.

### Example

An organization has an existing SSIS ETL package that extracts data from SQL Server, transforms it, and loads a warehouse.

Instead of completely rewriting the package, the organization can deploy and execute supported SSIS packages using Azure-SSIS IR.

```text
Existing SSIS Package
         |
         v
Azure-SSIS Integration Runtime
         |
         v
   Data Destination
```

### Characteristics

- Designed for SSIS package execution.
- Requires provisioning and configuration.
- Uses Azure compute resources.
- Can integrate with supported SQL Server and Azure services.

**Interview point:** Azure-SSIS IR is not the normal runtime for Copy Activity. It is specifically designed to execute SSIS packages.

---

## 4. Comparison of Integration Runtime Types

| Feature | Azure IR | Self-hosted IR | Azure-SSIS IR |
|---|---|---|---|
| Management | Azure-managed | Organization-managed | Azure-managed infrastructure |
| Installation | No local installation | Installed on a machine | Provisioned in Azure |
| Main purpose | Cloud integration | Private/on-premises connectivity | SSIS package execution |
| Cloud data movement | Yes | Yes, where supported | Not its primary purpose |
| On-premises connectivity | Depends on supported networking configuration | Yes | Depends on package connectivity configuration |
| SSIS execution | No | No | Yes |

### Easy Way to Remember

- Azure IR → Cloud integration.
- Self-hosted IR → Private network and on-premises connectivity.
- Azure-SSIS IR → Run SSIS packages.

---

## 5. How Integration Runtime Works with Linked Services

A Linked Service defines the connection information for a data store or compute service.

An Integration Runtime provides the execution and connectivity infrastructure for supported activities.

### Example

Suppose you want to copy data from an on-premises SQL Server to ADLS Gen2.

You configure:

**1. Linked Service for SQL Server**

Contains the connection details and authentication configuration for SQL Server.

**2. Linked Service for ADLS Gen2**

Contains the connection details and authentication configuration for the storage account.

**3. Integration Runtime**

Uses the appropriate runtime configuration to access the source and destination.

**4. Copy Activity**

Moves the data.

### Architecture

```text
Azure Data Factory
        |
        v
   Copy Activity
        |
        +--------------------+
        |                    |
        v                    v
SQL Server Linked      ADLS Linked
Service                Service
        |                    |
        v                    v
   Self-hosted IR       Azure IR
        |                    |
        v                    v
 On-Prem SQL Server      ADLS Gen2
```

This illustrates a hybrid integration scenario. The exact runtime configuration and data movement path depend on the connectors and networking setup.

---

## 6. Integration Runtime Selection

Choosing the correct Integration Runtime depends on the source, destination, network configuration, and activity type.

### Scenario 1: Azure SQL Database to ADLS Gen2

Use Azure Integration Runtime when the services are accessible through the configured cloud connectivity.

```text
Azure SQL Database
        |
        v
      Azure IR
        |
        v
     ADLS Gen2
```

### Scenario 2: On-Premises SQL Server to ADLS Gen2

Use Self-hosted IR when the SQL Server is inside a private network that requires a self-hosted connectivity component.

```text
On-Prem SQL Server
        |
        v
    Self-hosted IR
        |
        v
     ADLS Gen2
```

### Scenario 3: Existing SSIS Package Migration

Use Azure-SSIS IR to execute supported SSIS packages in Azure.

```text
SSIS Package
     |
     v
Azure-SSIS IR
     |
     v
Data Destination
```

### Scenario 4: Databricks Notebook Execution

ADF can use a Databricks linked service and Notebook Activity to orchestrate a notebook run.

The notebook's Spark computation is performed by Databricks compute, not by the ADF Integration Runtime.

---

## 7. Azure Integration Runtime: Important Configuration

Azure IR can be configured with different capabilities depending on the integration requirement.

Common configuration considerations include:

- Region.
- Compute properties for supported activities.
- Network connectivity.
- Data movement requirements.
- Private endpoint or managed virtual network options where supported.

### Region

The region can affect:

- Data movement performance.
- Network latency.
- Data residency.
- Cost.

For suitable workloads, placing services in the same or nearby region can reduce latency and potentially avoid some cross-region transfer costs.

### Managed Virtual Network

ADF can use a managed virtual network with supported Azure Integration Runtime configurations.

It helps isolate integration workloads and supports managed private endpoint connectivity to supported services.

### Important

Managed virtual network and private endpoint support depend on the service, connector, and configuration. Creating a private endpoint does not automatically grant data permissions.

---

## 8. Self-hosted Integration Runtime: Architecture

Self-hosted IR is particularly important in hybrid data engineering environments.

### Example Architecture

```text
                  Azure Cloud
+-----------------------------------------+
|                                         |
|          Azure Data Factory             |
|                  |                      |
+------------------|----------------------+
                   |
                   | Outbound communication
                   |
+------------------|----------------------+
|                  v                      |
|         Self-hosted IR Machine          |
|                  |                      |
|                  v                      |
|          On-Premises SQL Server         |
|                                         |
+-----------------------------------------+
```

### Installation Steps

1. Open Azure Data Factory Studio.
2. Navigate to Manage.
3. Select Integration Runtimes.
4. Create a new Integration Runtime.
5. Select Self-hosted.
6. Download the installer.
7. Install it on a machine with access to the required data sources.
8. Register the runtime using the authentication key provided by ADF.
9. Verify the runtime status.
10. Configure the relevant Linked Service to use it.

### Important Requirements

- The host machine must remain available when the runtime is needed.
- It must have network access to the required source and destination.
- Firewall and proxy settings must permit required communication.
- Credentials and permissions must be configured correctly.
- The runtime software should be kept updated.

---

## 9. Self-hosted IR: High Availability and Scaling

A single Self-hosted IR node can become a single point of failure.

For production environments, multiple nodes can be registered to the same Self-hosted IR.

### Single Node

```text
ADF
 |
 v
Self-hosted IR Node 1
 |
 v
Source System
```

If the machine becomes unavailable, the integration workload may fail.

### Multiple Nodes

```text
             Azure Data Factory
                     |
                     v
             Self-hosted IR
                /       \
               v         v
          Node 1       Node 2
               \         /
                v       v
              Data Source
```

### Benefits

- Better availability.
- Workload distribution.
- Maintenance flexibility.
- Reduced dependency on one machine.

### Important

Multiple nodes do not automatically solve every network or source-system bottleneck. Each node must have suitable connectivity, permissions, and capacity.

---

## 10. Integration Runtime and Data Movement

Copy Activity uses Integration Runtime infrastructure to move data.

The runtime choice affects how ADF connects to and transfers data between the source and sink.

### Example

Copy data from an on-premises SQL Server to Azure Blob Storage.

```text
On-Prem SQL Server
        |
        v
Self-hosted IR
        |
        v
Azure Blob Storage
```

The Self-hosted IR can provide the required access to the on-premises source.

For cloud-to-cloud copying, Azure IR is commonly used when the required connectivity is available.

### Performance Factors

Copy performance depends on:

- Source database performance.
- Destination performance.
- Network bandwidth and latency.
- Integration Runtime capacity.
- Parallel copy settings.
- Data volume and file sizes.
- Connector limitations.
- Source and sink throttling.

Increasing parallelism does not always improve performance. It can overload the source or destination.

---

## 11. Integration Runtime and Mapping Data Flows

Mapping Data Flows provide a visual, code-free transformation experience in ADF.

They run on managed Spark compute associated with Azure Integration Runtime.

### Example

```text
ADLS Gen2
    |
    v
Mapping Data Flow
    |
    +---- Filter
    |
    +---- Join
    |
    +---- Aggregate
    |
    v
ADLS Gen2
```

### Important

Mapping Data Flows use Spark-based compute, but they are not the same as running a Databricks notebook.

- Mapping Data Flows are designed and orchestrated inside ADF.
- Databricks notebooks run on Databricks compute.
- ADF can orchestrate both types of transformation workloads.

---

## 12. Integration Runtime and Databricks

ADF can execute a Databricks notebook through a Databricks Notebook Activity.

The responsibilities are separate:

| Component | Responsibility |
|---|---|
| ADF | Pipeline orchestration |
| Databricks Linked Service | Connection to the Databricks workspace |
| Notebook Activity | Starts notebook execution |
| Databricks Compute | Executes Spark transformations |
| ADLS Gen2 | Stores input and output data |

### Example

```text
ADF Pipeline
     |
     v
Databricks Notebook Activity
     |
     v
Databricks Compute
     |
     v
PySpark Processing
     |
     v
ADLS Gen2
```

**Interview point:** ADF Integration Runtime does not execute the Spark transformations inside a Databricks notebook.

---

## 13. Common Integration Runtime Errors

| Error | Possible Cause |
|---|---|
| Self-hosted IR unavailable | Host machine stopped or runtime service unavailable |
| Connection timeout | Network or firewall issue |
| Authentication failure | Incorrect credentials or permissions |
| Source connection failure | Incorrect server, port, or connection settings |
| Slow Copy Activity | Network, source, sink, or runtime bottleneck |
| Private endpoint connection failure | DNS, network, or endpoint configuration issue |
| SSIS package execution failure | Package, dependency, configuration, or runtime issue |

### Troubleshooting Steps

1. Check the Integration Runtime status in ADF Studio.
2. Confirm the runtime machine is running.
3. Check the Self-hosted IR service.
4. Verify source and destination connectivity.
5. Review firewall and proxy settings.
6. Validate authentication and permissions.
7. Check activity run details.
8. Review Copy Activity performance metrics where available.
9. Check whether the source or sink is throttling requests.

---

## 14. Integration Runtime vs Linked Service vs Dataset

This is a common interview comparison.

| Component | Purpose | Example |
|---|---|---|
| Integration Runtime | Provides execution and connectivity infrastructure | Self-hosted IR |
| Linked Service | Defines connection information | SQL Server connection |
| Dataset | Identifies data structure or location | `dbo.Customers` table |

### Easy Analogy

Imagine copying a file from a database to a storage account.

- Linked Service tells ADF how to connect.
- Dataset identifies the data to read or write.
- Integration Runtime provides the infrastructure to perform the integration.
- Copy Activity performs the copy operation.
- Pipeline coordinates the overall workflow.

---

## 15. Best Practices

1. Select the runtime based on source and destination connectivity.
2. Use Azure IR for suitable cloud integration workloads.
3. Use Self-hosted IR when private or on-premises connectivity requires it.
4. Use multiple Self-hosted IR nodes for suitable production availability requirements.
5. Monitor Self-hosted IR health and availability.
6. Keep runtime machines and software maintained.
7. Configure firewall and proxy access carefully.
8. Use least-privilege permissions.
9. Avoid overloading source and destination systems with excessive parallelism.
10. Monitor Copy Activity throughput and duration.
11. Use appropriate regions to reduce unnecessary latency and transfer costs.
12. Do not confuse ADF runtime compute with Databricks compute.
13. Use Azure-SSIS IR only when SSIS package execution is required.
14. Review private networking and DNS configuration when using private endpoints.

---

## 16. Common Interview Questions

### Q1. What is Integration Runtime in Azure Data Factory?

Integration Runtime is the compute infrastructure used by ADF to provide data integration capabilities, including data movement, connectivity, and supported transformation execution.

### Q2. What are the three types of Integration Runtime?

Azure Integration Runtime, Self-hosted Integration Runtime, and Azure-SSIS Integration Runtime.

### Q3. What is Azure Integration Runtime?

It is Azure-managed integration infrastructure commonly used for cloud-based data integration and supported transformation activities.

### Q4. What is Self-hosted Integration Runtime?

It is an integration runtime installed on an organization's machine to provide connectivity to supported private-network and on-premises data sources.

### Q5. When would you use Self-hosted IR?

When ADF needs to access a database or system inside an on-premises or private network that is not directly reachable through the available cloud integration environment.

### Q6. What is Azure-SSIS IR?

It is an Integration Runtime used to execute supported SQL Server Integration Services packages in Azure Data Factory.

### Q7. Can Azure IR connect to on-premises systems?

It depends on the network configuration and supported connectivity options. When a source requires a machine inside a private or on-premises network, Self-hosted IR is commonly used.

### Q8. What is the difference between Azure IR and Self-hosted IR?

Azure IR is managed by Azure and is commonly used for cloud integration. Self-hosted IR is installed and managed by the organization to provide private-network or on-premises connectivity.

### Q9. Can Self-hosted IR have multiple nodes?

Yes. Multiple nodes can be registered to the same Self-hosted IR to improve availability and distribute workloads.

### Q10. Does Self-hosted IR require inbound connectivity from Azure?

It commonly communicates with Azure using outbound connections. Required network and firewall rules must still be configured.

### Q11. Does Integration Runtime perform Databricks Spark transformations?

No. ADF orchestrates Databricks notebook execution, while Databricks compute performs the Spark transformations.

### Q12. Which Integration Runtime is used for Mapping Data Flows?

Mapping Data Flows run on managed Spark compute associated with Azure Integration Runtime.

### Q13. What affects Copy Activity performance?

Source and sink performance, network bandwidth, latency, runtime capacity, parallelism, data volume, and connector limitations.

### Q14. How do you troubleshoot a Self-hosted IR failure?

Check runtime status, host availability, service health, network connectivity, firewall rules, authentication, and activity error details.

### Q15. What is the difference between Integration Runtime and Linked Service?

Linked Service defines connection information. Integration Runtime provides the execution and connectivity infrastructure used by supported activities.

---

## 17. Quick Revision

| Topic | Key Point |
|---|---|
| Integration Runtime | ADF integration infrastructure |
| Azure IR | Managed cloud integration |
| Self-hosted IR | Private and on-premises connectivity |
| Azure-SSIS IR | Executes SSIS packages |
| Linked Service | Connection configuration |
| Dataset | Data structure or location |
| Copy Activity | Moves data |
| Mapping Data Flow | Visual Spark-based transformation |
| Databricks Activity | Orchestrates notebook execution |
| Multiple SHIR nodes | Improve availability and distribute workload |
| Outbound communication | Common Self-hosted IR connectivity pattern |
| Performance | Depends on network, source, sink, and runtime capacity |

---

## 18. Self-Assessment

Try answering these questions without looking at the notes:

1. What is Integration Runtime?
2. Why does ADF need Integration Runtime?
3. Explain the three types of Integration Runtime.
4. When would you choose Azure IR?
5. When would you choose Self-hosted IR?
6. What is Azure-SSIS IR?
7. What is the difference between Azure IR and Self-hosted IR?
8. How does Self-hosted IR connect to an on-premises SQL Server?
9. Why would you configure multiple Self-hosted IR nodes?
10. How does Integration Runtime work with Copy Activity?
11. What is the relationship between Integration Runtime and Linked Service?
12. Does ADF Integration Runtime execute Databricks Spark code?
13. How do Mapping Data Flows use Azure IR?
14. What factors affect Copy Activity performance?
15. How would you troubleshoot an unavailable Self-hosted IR?

**Interview readiness:** You should be able to explain the three Integration Runtime types, choose the correct runtime for cloud and hybrid scenarios, describe Self-hosted IR architecture, and distinguish ADF integration infrastructure from Databricks compute.