# Databricks Compute and Cost Basics

## 1. What Is Compute in Databricks?

Compute refers to the processing resources used to execute workloads in Databricks.

These resources provide the CPU, memory, and execution environment required to run:

- PySpark code
- Spark SQL queries
- Notebooks
- Data Engineering pipelines
- Machine Learning workloads
- Structured Streaming jobs

Databricks uses compute resources to process data stored in locations such as cloud object storage.

**Simple example:**

You have 100 GB of customer data stored in Azure Data Lake Storage Gen2.

To process this data using PySpark, you need compute resources. Databricks uses those resources to read, transform, and write the data.

**Remember:** Storage and compute are different.

- Storage holds your data.
- Compute processes your data.

---

## 2. Main Compute Components

A typical Spark compute environment contains the following components:

| Component | Responsibility |
|---|---|
| Driver | Coordinates the Spark application |
| Worker nodes | Execute distributed tasks |
| Executors | Run tasks and maintain application data on workers |
| CPU | Performs computation |
| Memory | Holds data and intermediate results |
| Disk | Stores temporary data and spill files |

### Driver

The driver is responsible for:

- Creating the Spark application.
- Building and coordinating execution plans.
- Scheduling tasks.
- Tracking job execution.
- Communicating with executors.

### Worker Nodes

Worker nodes provide resources for distributed processing.

They execute tasks assigned by the Spark scheduler.

### Executors

Executors are processes associated with a Spark application.

They:
- Execute tasks.
- Store cached data.
- Manage intermediate computation data.
- Report execution information to the driver.

**Interview point:** The driver coordinates execution, while executors perform distributed work.

---

## 3. Databricks Compute Types

Databricks provides different compute options depending on the workload and platform configuration.

Common compute categories include:

### A. All-Purpose Compute

Used for interactive development and exploration.

Typical use cases:
- Developing PySpark notebooks.
- Running SQL and Python commands interactively.
- Debugging transformations.
- Exploring datasets.
- Collaborating during development.

### B. Job Compute

Used for executing scheduled or automated workloads.

Typical use cases:
- ETL pipelines.
- Scheduled data transformations.
- Batch processing.
- Production data workflows.

Job compute can be configured to be created for a job run and terminated afterward, depending on the job configuration.

### C. SQL Warehouses

Compute resources designed for SQL workloads.

Typical use cases:
- Running SQL queries.
- Building dashboards.
- Business intelligence.
- Supporting SQL users and BI tools.

### D. Serverless Compute

Databricks manages the underlying infrastructure for supported serverless workloads.

Typical benefits:
- Less infrastructure management.
- Simplified compute provisioning.
- Automatic resource management for supported workloads.

Availability, supported features, and billing vary by cloud, workspace configuration, and Databricks product edition.

---

## 4. Cluster and Compute Configuration

A compute configuration determines the resources available to a workload.

Common configuration elements include:

| Configuration | Meaning |
|---|---|
| Runtime version | Databricks Runtime and Spark environment |
| Node type | Machine type used for compute |
| Worker count | Number of worker nodes |
| Autoscaling | Automatically adjusts worker count |
| Auto-termination | Stops idle compute after a configured period |
| Access mode | Controls how users and workloads access compute |
| Photon | Databricks' accelerated execution engine for supported workloads |

### Example

Suppose you configure compute with:

- 1 driver node.
- 2 to 8 worker nodes.
- Autoscaling enabled.
- Auto-termination after 15 minutes.

The compute can scale its worker count within the configured limits based on workload requirements.

**Important:** A larger cluster is not automatically faster or more cost-efficient. Performance depends on workload characteristics, data volume, partitioning, and execution strategy.

---

## 5. What Is Databricks Runtime?

Databricks Runtime is the execution environment used by Databricks compute.

It includes components such as:

- Apache Spark.
- Databricks optimizations.
- Libraries and runtime dependencies.
- Platform-specific capabilities.

Different runtime versions may support different features.

### Why Does Runtime Version Matter?

A runtime upgrade can affect:

- Spark behavior.
- Library compatibility.
- SQL functionality.
- Performance.
- Security and bug fixes.

**Interview point:** Always check runtime compatibility before upgrading production workloads.

---

## 6. What Is Photon?

Photon is Databricks' native vectorized query engine.

It accelerates supported workloads by using optimized native execution instead of relying entirely on the standard JVM-based Spark execution path.

### Potential Benefits

- Faster SQL queries.
- Improved performance for supported DataFrame operations.
- Better resource utilization for suitable workloads.

### Important Points

- Photon does not accelerate every operation.
- Performance improvement depends on the workload.
- Unsupported operations may use other execution paths.
- Photon availability depends on compute type and configuration.

**Interview question:** What is Photon, and how does it improve query performance?

**Answer:** Photon is Databricks' native vectorized execution engine. It can improve performance for supported SQL and DataFrame workloads by using optimized native execution.

---

## 7. What Is DBU?

<badge color="info">Important interview concept</badge>

DBU stands for **Databricks Unit**.

It is a unit of processing capability used by Databricks for measuring and billing its services.

Databricks usage is commonly measured in DBUs, but the actual price depends on the product, workload, cloud provider, region, and pricing arrangement.

### What Influences DBU Consumption?

- Compute type.
- Workload type.
- Runtime and configuration.
- Number and size of compute resources.
- Duration of usage.
- Applicable pricing model.

**Important:** DBU is not the same as a virtual machine's hourly infrastructure cost.

Depending on the cloud and pricing arrangement, the total cost may include both Databricks service charges and underlying cloud infrastructure charges.

---

## 8. Understanding Databricks Cost

A simplified cost model is:

**Total Cost = Databricks Charges + Cloud Infrastructure Charges + Other Applicable Services**

For example, on a cloud deployment where the customer pays separately for infrastructure:

1. Databricks charges are based on applicable DBU consumption.
2. Cloud provider charges cover the underlying compute and related infrastructure.
3. Storage, networking, and other services may add additional costs.

The exact billing model depends on the deployment and commercial arrangement.

### Factors That Increase Cost

| Factor | Why It Increases Cost |
|---|---|
| Large compute | More resources are consumed |
| Long-running clusters | Compute remains active longer |
| Unnecessary jobs | Additional execution consumes resources |
| Inefficient queries | More processing time and resources |
| Excessive shuffles | Additional network and disk activity |
| Over-caching | Resources remain occupied unnecessarily |
| Idle compute | Resources remain active without useful work |
| Frequent small workloads | Repeated startup and execution overhead |

---

## 9. How to Reduce Databricks Cost

### 1. Enable Auto-Termination

Configure idle compute to stop automatically.

Example:

```text
Auto-termination: 15 minutes
```

This helps prevent unnecessary charges from idle interactive compute.

### 2. Use Job Compute for Automated Workloads

Use job-oriented compute for scheduled production workloads where appropriate.

Avoid keeping interactive compute running when it is not needed.

### 3. Right-Size Compute

Select a suitable machine type and worker count.

Do not allocate excessive resources without measuring workload requirements.

### 4. Use Autoscaling

Autoscaling adjusts worker count within configured limits.

It can help workloads that have varying processing requirements.

However, scaling does not guarantee lower costs. The minimum and maximum workers, scaling behavior, and workload duration all matter.

### 5. Optimize Spark Queries

Reduce unnecessary processing through:

- Predicate pushdown.
- Projection pushdown.
- Appropriate join strategies.
- Efficient partitioning.
- Data skew handling.
- Avoiding unnecessary shuffles.

Faster execution can reduce compute time, although the actual cost impact depends on resource usage and billing.

### 6. Use Delta Lake Optimization

For suitable Delta tables, consider:

```sql
OPTIMIZE orders;
```

This compacts small files into larger files.

For appropriate query patterns, data skipping can be improved using supported clustering or Z-Ordering techniques.

**Important:** `OPTIMIZE` itself consumes compute resources. Run it when the expected query and maintenance benefits justify the cost.

### 7. Avoid Unnecessary Caching

Caching can improve performance when the same DataFrame is reused.

However, caching data that is not reused wastes resources.

### 8. Monitor Compute Usage

Regularly review:
- Active compute.
- Idle time.
- Job duration.
- DBU consumption.
- Query execution time.
- Failed and repeated jobs.

---

## 10. Interactive Compute vs Job Compute

| Feature | All-Purpose Compute | Job Compute |
|---|---|---|
| Main purpose | Interactive development | Automated execution |
| Typical user | Developer or analyst | Pipeline or job |
| Workload | Exploration and debugging | Scheduled and production jobs |
| Lifecycle | May remain active until stopped or terminated | Often created and terminated around a job run |
| Cost control | Auto-termination and right-sizing | Job lifecycle and compute configuration |

**Interview question:** Why would you choose job compute instead of all-purpose compute for a production ETL pipeline?

**Answer:** Job compute is designed for automated workloads and can be configured to run only when required. This reduces unnecessary interactive compute usage and supports better workload isolation.

---

## 11. Serverless vs Classic Compute

| Feature | Serverless Compute | Classic Compute |
|---|---|---|
| Infrastructure management | Managed by Databricks | More infrastructure configuration by users |
| Provisioning | Managed provisioning | User-configured compute resources |
| Scaling | Managed for supported workloads | Depends on configured scaling |
| Control | Less direct infrastructure control | More configuration control |
| Availability | Depends on workload and region | Depends on workspace and cloud configuration |

**Important:** Serverless does not automatically mean cheaper. Compare the actual workload performance and billing model.

---

## 12. Cost Monitoring and Troubleshooting

When a Databricks workload becomes expensive, investigate the following:

### Compute Level

- Is compute running when nobody is using it?
- Is the cluster larger than necessary?
- Is autoscaling configured appropriately?
- Is auto-termination enabled?

### Job Level

- Is the same job running repeatedly?
- Are unnecessary tasks being executed?
- Are failed jobs being retried excessively?
- Is the schedule appropriate?

### Query Level

- Is there unnecessary data scanning?
- Are large shuffles occurring?
- Is there data skew?
- Are joins using suitable strategies?
- Are small files affecting query performance?

### Billing Level

- Which workload is consuming the most DBUs?
- Which compute configuration is most expensive?
- Are Databricks and cloud infrastructure charges being considered separately where applicable?

Use the available Databricks billing and usage information, compute monitoring, query history, and cloud billing tools to investigate.

---

## 13. Practical Scenario-Based Questions

### Scenario 1: Your cluster is running overnight even though no job is executing. What would you do?

**Answer:**

I would inspect the compute activity and configure auto-termination. I would also verify that no scheduled job or notebook session requires the compute to remain active.

### Scenario 2: A daily ETL pipeline takes 30 minutes and the compute remains active for several hours. How would you reduce cost?

**Answer:**

I would investigate why the compute remains active after the workload finishes. I would use an appropriate job compute configuration, enable suitable lifecycle management, and review job execution time.

### Scenario 3: A Spark job is slow even after increasing worker nodes. What would you investigate?

**Answer:**

I would inspect the execution plan and Spark UI. I would check shuffle size, data skew, partition distribution, join strategies, and memory spills. Adding more workers does not necessarily fix an inefficient query.

### Scenario 4: A developer uses a large interactive cluster for a small SQL query. What would you recommend?

**Answer:**

I would evaluate whether a smaller compute configuration or an appropriate SQL warehouse is sufficient. The compute should match the workload requirements.

### Scenario 5: Your team wants to run a production pipeline every day. Which compute would you consider?

**Answer:**

I would consider job-oriented compute configured for the workload, with appropriate resource sizing, lifecycle management, monitoring, and retry behavior.

### Scenario 6: Does Photon always reduce the cost of a workload?

**Answer:**

No. Photon can improve performance for supported workloads, but the actual cost depends on execution time, resource consumption, and applicable pricing. I would benchmark the workload before making a decision.

---

## 14. Common Interview Questions

**Q1. What is Databricks compute?**

It is the processing environment that provides resources to run Spark jobs, SQL queries, notebooks, and other workloads.

**Q2. What is DBU?**

DBU stands for Databricks Unit. It is a unit used to measure Databricks service usage for billing.

**Q3. What is the difference between storage and compute?**

Storage holds data, while compute processes data.

**Q4. What is the difference between all-purpose compute and job compute?**

All-purpose compute is mainly used for interactive development, while job compute is intended for automated workloads.

**Q5. What is auto-termination?**

It automatically stops idle compute after a configured period.

**Q6. What is autoscaling?**

It adjusts the number of worker nodes within configured limits according to workload requirements.

**Q7. What is Photon?**

Photon is Databricks' native vectorized execution engine for accelerating supported workloads.

**Q8. What is Databricks Runtime?**

It is the execution environment containing Spark and Databricks-specific runtime capabilities and optimizations.

**Q9. How can you reduce Databricks cost?**

Use appropriate compute sizing, auto-termination, job-oriented compute, query optimization, workload monitoring, and efficient storage practices.

**Q10. Is serverless always cheaper than classic compute?**

No. Cost depends on workload, resource consumption, execution time, and the applicable pricing model.

**Q11. Does increasing workers always improve Spark performance?**

No. Performance also depends on data distribution, partitioning, shuffle operations, skew, memory, and query design.

**Q12. What is the difference between DBU cost and cloud infrastructure cost?**

DBU cost represents Databricks service usage. Cloud infrastructure cost covers underlying cloud resources when billed separately.

---

## 15. Quick Revision Notes

| Term | Remember |
|---|---|
| Compute | Resources used to process data |
| Driver | Coordinates Spark execution |
| Worker | Provides distributed processing resources |
| Executor | Runs tasks and manages application data |
| All-purpose compute | Interactive development |
| Job compute | Automated workloads |
| SQL warehouse | SQL and BI workloads |
| Serverless | Databricks-managed infrastructure for supported workloads |
| Databricks Runtime | Spark and Databricks execution environment |
| Photon | Native vectorized execution engine |
| DBU | Databricks Unit |
| Autoscaling | Adjusts worker count |
| Auto-termination | Stops idle compute |
| Right-sizing | Matching resources to workload requirements |

---

## 16. Self-Assessment

Before moving to the next topic, make sure you can explain:

- [ ] What Databricks compute means.
- [ ] The roles of driver, worker, and executor.
- [ ] The difference between all-purpose and job compute.
- [ ] What a SQL warehouse is used for.
- [ ] What serverless compute means.
- [ ] What Databricks Runtime provides.
- [ ] What Photon does.
- [ ] What DBU means.
- [ ] The difference between Databricks and cloud infrastructure charges.
- [ ] How autoscaling and auto-termination work.
- [ ] How to reduce unnecessary compute costs.
- [ ] How to investigate an expensive Spark job.

**Final takeaway:** A Data Engineer should understand not only how to build Databricks pipelines but also how to select suitable compute, monitor resource usage, optimize execution, and control costs. The objective is to deliver reliable performance without wasting compute resources.