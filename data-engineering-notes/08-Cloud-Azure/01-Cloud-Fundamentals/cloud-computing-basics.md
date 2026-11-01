# Cloud Computing Basics

## 1. What Is Cloud Computing?

Cloud computing is the delivery of computing resources—such as servers, storage, databases, networking, and software—over the internet.

Instead of purchasing and maintaining physical infrastructure, organizations can rent computing resources from cloud providers and pay according to their usage or subscription model.

**Example:**

Suppose a company needs a server to process customer data.

**Traditional approach:**
- Purchase physical servers.
- Set up networking and storage.
- Maintain hardware and power.
- Upgrade infrastructure when capacity increases.
- Pay for infrastructure even when it is underused.

**Cloud approach:**
- Create a virtual machine using a cloud provider.
- Configure storage and networking.
- Increase or decrease resources when needed.
- Pay according to the selected pricing model.

Cloud computing reduces the need to manage physical infrastructure, but it does not eliminate infrastructure management entirely.

---

## 2. Why Do Companies Use Cloud Computing?

Common reasons include:

| Benefit | Explanation |
|---|---|
| Scalability | Increase or decrease resources based on workload |
| Cost flexibility | Avoid large upfront hardware purchases |
| Availability | Use services designed for high availability |
| Global access | Deploy applications across geographic regions |
| Managed services | Let providers manage parts of the infrastructure |
| Faster deployment | Provision resources in minutes instead of purchasing hardware |
| Backup and recovery | Use cloud storage and recovery services |
| Automation | Provision and manage infrastructure programmatically |

### Example

An e-commerce company experiences heavy traffic during a festival sale.

With cloud infrastructure, it can increase computing capacity to handle additional traffic and reduce capacity when demand returns to normal.

This is more flexible than purchasing enough physical servers for the highest possible traffic throughout the year.

---

## 3. Traditional Data Center vs Cloud Computing

| Traditional data center | Cloud computing |
|---|---|
| Company purchases hardware | Company rents cloud resources |
| High initial investment | Often usage-based or subscription pricing |
| Hardware maintenance by company | Provider manages physical infrastructure |
| Capacity upgrades can take time | Resources can often be provisioned quickly |
| Physical infrastructure is company-managed | Infrastructure responsibilities are shared |
| Expansion requires hardware planning | Scaling can be automated or configured |

Cloud computing does not always cost less. Poor resource sizing, unnecessary services, data transfer, and idle resources can increase cloud expenses.

---

## 4. Characteristics of Cloud Computing

Five commonly taught characteristics are:

### 4.1 On-Demand Self-Service

Users can provision computing resources when required without manually contacting the provider for every request.

Example: Creating a virtual machine through the Azure Portal.

### 4.2 Broad Network Access

Cloud services can be accessed over a network using different devices and standard interfaces.

Example: Accessing Azure Storage from an application or notebook.

### 4.3 Resource Pooling

Cloud providers pool physical computing resources and allocate them to multiple customers.

This is commonly implemented through multi-tenant infrastructure.

Customers' resources and data are logically isolated according to the service's security design.

### 4.4 Rapid Elasticity

Resources can scale up or down quickly based on demand.

Example: Increasing compute capacity during a large data-processing workload.

### 4.5 Measured Service

Cloud usage is monitored and measured.

Customers are billed according to the service's pricing model, which may depend on compute time, storage, requests, transactions, or other usage metrics.

---

## 5. Cloud Deployment Models

Cloud deployment models describe where cloud infrastructure is deployed and who can use it.

### 5.1 Public Cloud

Cloud infrastructure is owned and operated by a cloud provider and made available to multiple customers.

Examples:
- Microsoft Azure
- Amazon Web Services (AWS)
- Google Cloud Platform (GCP)

**Characteristics:**
- Shared underlying infrastructure.
- Logical isolation between customers.
- Flexible resource provisioning.
- Provider-managed physical data centers.

### 5.2 Private Cloud

Cloud infrastructure is dedicated to a single organization.

It may be hosted in the organization's own data center or by a third-party provider.

**Characteristics:**
- Dedicated environment.
- Greater control over infrastructure configuration.
- Can support organization-specific security requirements.
- Requires management and operational responsibility.

A private cloud is not automatically more secure than a public cloud. Security depends on architecture, configuration, and operations.

### 5.3 Hybrid Cloud

Hybrid cloud combines private infrastructure with public cloud services.

Example:

A bank may keep certain sensitive systems in its private environment while using Azure for selected analytics workloads.

**Benefits:**
- Flexibility in workload placement.
- Gradual cloud migration.
- Integration with existing infrastructure.

### 5.4 Multi-Cloud

Multi-cloud means using services from multiple cloud providers.

Example:

An organization uses Azure for data processing and AWS for another application.

**Why companies use it:**
- Avoid dependence on a single provider.
- Use services suited to different workloads.
- Support geographic or business requirements.

Multi-cloud also increases operational complexity, including identity management, networking, monitoring, and cost management.

---

## 6. Cloud Service Models

Cloud service models describe how much of the technology stack the provider manages.

The three foundational models are IaaS, PaaS, and SaaS.

### 6.1 Infrastructure as a Service (IaaS)

IaaS provides fundamental computing resources such as virtual machines, networking, and storage.

The customer manages the operating system and software installed on the infrastructure.

**Example:** Azure Virtual Machines.

Typical customer responsibilities:
- Operating system configuration.
- Runtime installation.
- Application deployment.
- Data and application security.
- Resource configuration.

Typical provider responsibilities:
- Physical servers.
- Data center facilities.
- Physical networking.
- Virtualization infrastructure.

### 6.2 Platform as a Service (PaaS)

PaaS provides a managed platform for building and running applications or workloads.

The provider manages more of the underlying infrastructure and platform components.

**Examples:**
- Azure App Service.
- Azure SQL Database.
- Azure Data Factory (managed data integration service).

Typical customer responsibilities:
- Application or pipeline configuration.
- Data and access policies.
- Business logic.
- Workload monitoring.

The exact responsibility split depends on the service.

### 6.3 Software as a Service (SaaS)

SaaS provides ready-to-use software over the internet.

The provider manages the application and underlying infrastructure.

**Examples:**
- Microsoft 365.
- Salesforce.
- Google Workspace.

Users mainly configure and use the software.

### Service Model Comparison

| Area | IaaS | PaaS | SaaS |
|---|---|---|---|
| Physical infrastructure | Provider | Provider | Provider |
| Virtualization | Provider | Provider | Provider |
| Operating system | Customer | Provider | Provider |
| Runtime/platform | Customer | Provider | Provider |
| Application | Customer | Customer | Provider |
| Application data and usage | Customer | Customer | Shared responsibility, depending on service |

### Easy way to remember

- **IaaS:** Rent infrastructure.
- **PaaS:** Use a managed platform.
- **SaaS:** Use ready-made software.

The more managed the service, the less infrastructure the customer generally needs to maintain.

---

## 7. Shared Responsibility Model

Cloud security is a shared responsibility between the cloud provider and the customer.

The exact responsibilities depend on the service model.

For example:

| Responsibility | IaaS | PaaS | SaaS |
|---|---|---|---|
| Physical data center security | Provider | Provider | Provider |
| Physical hardware | Provider | Provider | Provider |
| Operating system patching | Customer | Provider | Provider |
| Application configuration | Customer | Customer | Provider manages application |
| User access management | Customer | Customer | Customer |
| Data classification and appropriate use | Customer | Customer | Customer |

This is a simplified comparison. Actual responsibility boundaries vary by cloud service and contract.

**Important interview point:** Moving workloads to the cloud does not mean the cloud provider is responsible for every security issue.

Customers remain responsible for areas such as identity management, access configuration, data protection, and secure workload usage.

---

## 8. Important Cloud Computing Concepts

### 8.1 Scalability

Scalability is the ability to increase or decrease resources to handle workload changes.

Two common types:

**Vertical scaling (Scale Up/Down)**

Increase or decrease the capacity of an existing resource.

Example:
- Increase a virtual machine from 4 CPU cores to 8 CPU cores.
- Increase memory from 16 GB to 32 GB.

**Horizontal scaling (Scale Out/In)**

Increase or decrease the number of resource instances.

Example:
- Increase the number of application instances from 2 to 6.
- Reduce them from 6 to 2 when demand decreases.

| Vertical scaling | Horizontal scaling |
|---|---|
| Changes resource capacity | Changes instance count |
| Often requires a larger machine | Adds more machines or instances |
| Limited by machine capacity | Can support distributed workloads |
| Application may need downtime depending on service | Requires workload support for distribution |

### 8.2 Elasticity

Elasticity is the ability to automatically or dynamically adjust resources according to changing demand.

**Difference:**
- Scalability: Ability to handle increased or decreased workload.
- Elasticity: Ability to adjust resources as demand changes, often automatically.

### 8.3 High Availability

High availability means designing a system to remain accessible despite certain failures.

Common techniques:
- Deploy across availability zones.
- Use load balancing.
- Maintain redundant instances.
- Implement health checks and failover.

High availability does not mean downtime is impossible.

### 8.4 Fault Tolerance

Fault tolerance is the ability of a system to continue operating when a component fails.

Example:

If one processing node fails, a distributed processing system may continue using other nodes and recover lost work.

High availability focuses on maintaining service availability; fault tolerance focuses on continuing operation despite failures. They overlap but are not identical.

### 8.5 Disaster Recovery

Disaster recovery is the process of restoring systems and data after a major failure or disruption.

Common techniques:
- Backups.
- Replication.
- Cross-region recovery.
- Recovery procedures and testing.

Two important terms:

**RPO (Recovery Point Objective)**

The maximum acceptable amount of data loss measured in time.

Example: An RPO of 1 hour means the recovery strategy should aim to lose no more than approximately one hour of data.

**RTO (Recovery Time Objective)**

The target time within which a service should be restored after disruption.

Example: An RTO of 2 hours means the service should be restored within two hours.

### 8.6 Pay-As-You-Go

Customers pay according to the pricing model and resources they consume.

Examples:
- Virtual machine running time.
- Storage capacity.
- Data processing.
- Database usage.
- Network data transfer.

Not every cloud service is billed strictly per second or per request. Pricing models vary.

---

## 9. Regions and Availability Zones

Cloud providers operate data centers in different geographic locations.

### Region

A region is a geographic area containing one or more cloud data centers.

Example: Azure Central India or South India.

### Availability Zone

An availability zone is a physically separate location within a supported cloud region, designed to provide independent infrastructure and fault isolation.

A region may contain multiple availability zones.

### Why are they important?

Suppose an application runs only in one availability zone.

If that zone experiences a failure, the application may become unavailable.

Deploying suitable resources across multiple zones can improve availability.

**Region vs Availability Zone**

| Region | Availability Zone |
|---|---|
| Geographic area | Separate location within a region |
| Contains one or more data center locations | Provides fault isolation within a region |
| Used for geographic deployment | Used for resilience within a region |

Availability zone support and service behavior vary by region and service.

---

## 10. Cloud Computing in Data Engineering

Cloud computing is important for data engineering because it provides scalable storage, processing, ingestion, orchestration, and analytics services.

A data engineer can use cloud services to build end-to-end data pipelines.

### Example Azure Data Engineering Architecture

```text
Source Systems
     |
     v
Azure Data Factory
     |
     v
Azure Data Lake Storage Gen2
     |
     v
Azure Databricks
     |
     v
Delta Lake
     |
     v
Gold Tables / Azure Synapse Analytics
     |
     v
Power BI
```

### Important Azure Services

| Service | Purpose |
|---|---|
| Azure Data Factory | Data integration and pipeline orchestration |
| Azure Data Lake Storage Gen2 | Scalable cloud data storage |
| Azure Databricks | Distributed data processing and analytics |
| Azure Synapse Analytics | Analytics and data warehousing capabilities |
| Azure SQL Database | Managed relational database |
| Azure Event Hubs | Event ingestion |
| Azure Key Vault | Secrets and key management |
| Microsoft Entra ID | Identity and access management |
| Azure Monitor | Monitoring and diagnostics |

A service can support multiple use cases. For example, Azure Data Factory is primarily a data integration and orchestration service, not a general-purpose distributed Spark processing engine.

### Example: Processing Daily Sales Data

Suppose an organization receives sales files every day.

1. Data Factory schedules the pipeline.
2. Source files are copied into ADLS Gen2.
3. Databricks reads the files.
4. PySpark cleans and transforms the data.
5. Delta tables store the processed results.
6. Gold tables provide business-ready sales metrics.
7. Power BI uses the Gold data for reporting.

This architecture separates ingestion, storage, transformation, and reporting responsibilities.

---

## 11. Cloud Cost Management

Cloud resources can become expensive when they are not managed properly.

Important practices:

1. Stop or terminate unused compute resources.
2. Configure auto-termination where suitable.
3. Choose appropriate VM sizes.
4. Monitor storage growth.
5. Use lifecycle policies for older data.
6. Monitor data transfer costs.
7. Set budgets and alerts.
8. Review resource utilization.
9. Use reserved capacity or savings plans when workloads and commitments justify them.

**Important:** A cloud service being managed does not mean it is automatically inexpensive.

For data engineering, monitor:
- Databricks compute usage.
- Data Factory activity and integration runtime usage.
- Storage capacity and transactions.
- Data transfer.
- SQL warehouse or database compute.

---

## 12. Common Cloud Computing Interview Questions

### Q1. What is cloud computing?

Cloud computing is the delivery of computing resources such as servers, storage, networking, databases, and software over the internet.

### Q2. What are the main benefits of cloud computing?

Scalability, flexible pricing, rapid provisioning, availability, managed services, and global access.

### Q3. What are the three cloud service models?

IaaS, PaaS, and SaaS.

- IaaS provides infrastructure.
- PaaS provides a managed platform.
- SaaS provides ready-to-use software.

### Q4. What is the difference between public and private cloud?

Public cloud infrastructure is provided to multiple customers with logical isolation. Private cloud infrastructure is dedicated to one organization.

### Q5. What is hybrid cloud?

Hybrid cloud combines private infrastructure with public cloud services.

### Q6. What is multi-cloud?

Multi-cloud means using services from multiple cloud providers.

### Q7. What is the difference between scalability and elasticity?

Scalability is the ability to handle workload changes by adjusting resources. Elasticity is the ability to dynamically adjust resources as demand changes.

### Q8. What is the difference between vertical and horizontal scaling?

Vertical scaling increases the capacity of an existing machine. Horizontal scaling increases the number of machines or instances.

### Q9. What is high availability?

High availability is the design goal of keeping a service accessible despite certain component failures.

### Q10. What is fault tolerance?

Fault tolerance is the ability of a system to continue operating when components fail.

### Q11. What is disaster recovery?

Disaster recovery is the process of restoring systems and data after a major failure.

### Q12. What is the shared responsibility model?

It describes how cloud security responsibilities are divided between the cloud provider and the customer.

### Q13. What is an availability zone?

An availability zone is a physically separate location within a supported cloud region, designed to provide infrastructure fault isolation.

### Q14. What is the difference between RPO and RTO?

- RPO: Maximum acceptable data loss measured in time.
- RTO: Target time to restore the service.

### Q15. Why is cloud computing useful for data engineers?

It provides scalable storage, distributed processing, managed ingestion, orchestration, databases, and analytics services.

---

## 13. Common Mistakes to Avoid

- Thinking cloud computing always costs less than on-premises infrastructure.
- Assuming the cloud provider manages every security responsibility.
- Confusing scalability with elasticity.
- Confusing high availability with fault tolerance.
- Treating a region and an availability zone as the same thing.
- Assuming private cloud is automatically more secure.
- Assuming PaaS means no configuration or operational responsibility.
- Thinking horizontal scaling always requires application changes to be minimal.
- Assuming all cloud services use pay-as-you-go pricing.
- Confusing a cloud provider with an individual cloud service.

---

## 14. Quick Revision

| Concept | Meaning |
|---|---|
| Cloud computing | Computing resources delivered over a network |
| Public cloud | Provider-operated infrastructure for multiple customers |
| Private cloud | Cloud infrastructure dedicated to one organization |
| Hybrid cloud | Combination of private and public cloud |
| Multi-cloud | Use of multiple cloud providers |
| IaaS | Infrastructure as a Service |
| PaaS | Platform as a Service |
| SaaS | Software as a Service |
| Scalability | Ability to handle workload changes |
| Elasticity | Dynamic resource adjustment |
| Vertical scaling | Increase or decrease machine capacity |
| Horizontal scaling | Increase or decrease instance count |
| High availability | Keep services accessible despite failures |
| Fault tolerance | Continue operating despite component failures |
| Disaster recovery | Restore systems after major disruption |
| RPO | Acceptable data-loss window |
| RTO | Target service-restoration time |
| Region | Geographic cloud deployment area |
| Availability zone | Separate fault-isolated location within a region |
| Shared responsibility | Security responsibilities divided between provider and customer |

---

## 15. Self-Assessment

Try answering these questions without looking at the notes:

1. What is cloud computing?
2. Why do companies move workloads to the cloud?
3. Explain IaaS, PaaS, and SaaS with examples.
4. What is the difference between public, private, hybrid, and multi-cloud?
5. Explain vertical and horizontal scaling.
6. What is the difference between scalability and elasticity?
7. Explain high availability and fault tolerance.
8. What are RPO and RTO?
9. What is the difference between a region and an availability zone?
10. Explain the shared responsibility model.
11. Why is cloud computing useful for data engineering?
12. Describe a basic Azure data engineering architecture.

**Interview readiness:** You should be able to explain the cloud fundamentals, service models, deployment models, scaling, availability, and how cloud services fit into a data engineering pipeline.