# Azure Cosmos DB

## 1. What Is Azure Cosmos DB?

**Azure Cosmos DB** is a fully managed, globally distributed NoSQL database service provided by Microsoft Azure.

It is designed for applications that need:
- Low-latency data access.
- High availability.
- Global data distribution.
- Horizontal scalability.
- Flexible data models.
- High-throughput workloads.

Unlike traditional relational databases, Cosmos DB supports non-relational data models and APIs designed for different application requirements.

**Simple explanation:** Azure Cosmos DB is a globally distributed NoSQL database service that helps applications store and access data quickly across different regions.

### Common use cases

- Real-time applications.
- IoT applications.
- Gaming platforms.
- User profiles and session data.
- Product catalogs.
- Personalization systems.
- Applications serving users across multiple countries.
- Event-driven applications.

---

## 2. Why Do We Need Cosmos DB?

Consider an e-commerce application with customers in India, the United States, and Europe.

The application stores:
- Customer profiles.
- Shopping cart information.
- Product details.
- User preferences.
- Application activity.

If all application requests must access a database in one distant region, users in other regions may experience higher latency.

Cosmos DB can distribute data across multiple Azure regions so applications can access data from geographically closer locations.

### Basic architecture

```text
             Global Application
                     |
          +----------+----------+
          |          |          |
          v          v          v
       India        USA       Europe
          |          |          |
          v          v          v
       Cosmos DB  Cosmos DB  Cosmos DB
          +----------+----------+
                     |
             Globally Distributed
                 Data Service
```

The database can replicate data across configured regions according to its consistency and replication settings.

---

## 3. Main Features of Azure Cosmos DB

### 3.1 Global Distribution

Cosmos DB allows a database to be distributed across multiple Azure regions.

Benefits:
- Lower latency for geographically distributed users.
- Regional availability.
- Read access from multiple regions.
- Support for regional failover.

### 3.2 Low Latency

Cosmos DB is designed for low-latency data access.

Its performance depends on:
- Data model.
- Partition key.
- Request complexity.
- Provisioned throughput.
- Region selection.
- Consistency level.
- Network conditions.

### 3.3 Horizontal Scalability

Cosmos DB distributes data across partitions.

As the data volume and throughput requirements increase, the service can scale across additional physical partitions.

### 3.4 Flexible Data Model

Cosmos DB supports multiple APIs and data models, including:
- NoSQL document data.
- MongoDB-compatible workloads.
- Cassandra-compatible workloads.
- Table data.
- Gremlin graph data.

API availability and capabilities depend on the selected Cosmos DB offering.

### 3.5 High Availability

Cosmos DB provides replication and availability capabilities.

Its behavior depends on:
- Configured regions.
- Consistency level.
- Replication settings.
- Availability configuration.
- Application retry and failover handling.

---

## 4. Cosmos DB Architecture

A Cosmos DB account is the top-level resource.

The hierarchy for the Azure Cosmos DB for NoSQL API is:

```text
              Cosmos DB Account
                      |
                      v
                   Database
                      |
                      v
                  Container
                      |
                      v
                    Items
```

### 4.1 Cosmos DB Account

The account is the top-level resource.

It defines important settings such as:
- API type.
- Geographic regions.
- Consistency configuration.
- Networking.
- Account-level access.

### 4.2 Database

A database is a logical container for one or more containers.

It can help organize application data.

### 4.3 Container

A container stores items and is a key unit for partitioning and throughput configuration.

For the NoSQL API, a container is broadly comparable to a collection in a document database.

### 4.4 Items

Items are the individual records stored in a container.

For the NoSQL API, items are JSON documents.

Example:

```json
{
  "id": "ORD1001",
  "customerId": "C101",
  "product": "Laptop",
  "amount": 65000,
  "status": "Delivered"
}
```

Each item represents an individual record.

**Important:** Cosmos DB uses different terminology depending on the API. For example, the NoSQL API uses containers and items, while MongoDB-compatible APIs use collections and documents.

---

## 5. Azure Cosmos DB APIs

Cosmos DB offers different APIs to support different application data models and compatibility requirements.

### 5.1 Azure Cosmos DB for NoSQL

This is the native JSON document API.

Characteristics:
- Stores JSON items.
- Uses SQL-like query syntax.
- Supports partitioned containers.
- Supports indexing.
- Uses request units for throughput measurement.

Example query:

```sql
SELECT *
FROM c
WHERE c.status = "Delivered"
```

This query retrieves items whose status is `Delivered`.

### 5.2 Azure Cosmos DB for MongoDB

Provides a MongoDB-compatible interface for applications using supported MongoDB drivers and features.

It is useful for applications designed around MongoDB-compatible document operations.

Compatibility depends on the specific Cosmos DB API version and supported features.

### 5.3 Azure Cosmos DB for Apache Cassandra

Provides a Cassandra-compatible interface.

It is useful for applications that use Cassandra Query Language (CQL) and supported Cassandra client tools.

### 5.4 Azure Cosmos DB for Table

Provides a key-value table API compatible with Azure Table Storage-style operations.

It is suitable for simple structured key-value data.

### 5.5 Azure Cosmos DB for Apache Gremlin

Supports graph data models and Gremlin queries.

It is useful when relationships between entities are central to the application.

### API comparison

| API | Data model | Common use |
|---|---|---|
| NoSQL | JSON documents | Modern document-based applications |
| MongoDB | Documents | MongoDB-compatible applications |
| Cassandra | Wide-column | Cassandra-compatible workloads |
| Table | Key-value | Simple structured key-value data |
| Gremlin | Graph | Relationship-oriented applications |

**Interview point:** Select the API based on the application's data model, client compatibility, and required features. Do not assume every API supports identical functionality.

---

## 6. Partitioning in Cosmos DB

Partitioning is one of the most important concepts in Cosmos DB.

It allows data to be distributed across multiple physical partitions to support scalability and throughput.

### 6.1 Partition Key

A partition key is a property selected when creating a container.

Cosmos DB uses the partition key to distribute items among logical partitions.

Example:

```json
{
  "id": "ORD1001",
  "customerId": "C101",
  "amount": 65000,
  "status": "Delivered"
}
```

If `customerId` is selected as the partition key:

```text
Partition Key: /customerId
```

Items with the same customer ID belong to the same logical partition.

### 6.2 Logical Partition

A logical partition contains all items with the same partition-key value.

Example:

```text
Container: Orders
Partition Key: customerId

Logical Partition C101
    ORD1001
    ORD1002
    ORD1003

Logical Partition C102
    ORD1004
    ORD1005

Logical Partition C103
    ORD1006
```

### 6.3 Physical Partition

A physical partition is the underlying storage and compute partition managed by Cosmos DB.

Multiple logical partitions can be placed on a physical partition.

Cosmos DB manages the distribution of logical partitions across physical partitions.

### 6.4 Choosing a Good Partition Key

A good partition key should generally provide:
- High cardinality.
- Even distribution of data.
- Even distribution of request volume.
- Good alignment with common queries.

A poor partition key can create:
- Hot partitions.
- Uneven data distribution.
- Throughput bottlenecks.
- Increased cross-partition queries.

### Example

Suppose an orders container has millions of records.

Possible partition keys:

| Partition key | Potential issue |
|---|---|
| `status` | Low cardinality; a few values may become hotspots |
| `country` | May distribute unevenly if most users are in one country |
| `customerId` | Often better when customers and queries are distributed |
| `orderId` | High cardinality, but may not support common customer-based queries efficiently |

There is no universally best partition key. Choose based on data distribution and access patterns.

---

## 7. Request Units (RU/s)

Cosmos DB uses **Request Units (RUs)** to measure the resources required to perform database operations.

An RU represents a normalized measure of database request cost.

Operations consume different amounts of RUs.

Examples:
- Reading an item.
- Inserting an item.
- Updating an item.
- Deleting an item.
- Running a query.

The RU consumption depends on factors such as:
- Item size.
- Query complexity.
- Number of items scanned.
- Index usage.
- Consistency level.
- Partition distribution.

### RU/s

RU/s means Request Units per second.

It represents the throughput capacity allocated to a Cosmos DB container or database.

### Example

Suppose an application has 1,000 RU/s provisioned.

If the workload requires more throughput than the available capacity, requests may receive a `429 Request Rate Too Large` response.

This indicates that the request was throttled.

Applications should use suitable retry handling and respect the retry guidance provided by the service.

### Throughput models

Cosmos DB supports different throughput options, including:

- Provisioned throughput.
- Autoscale throughput.
- Serverless capacity for supported account configurations.

**Provisioned throughput:** You allocate a throughput capacity in RU/s.

**Autoscale:** Throughput can automatically scale within the configured maximum.

**Serverless:** You pay based on consumed request units rather than provisioning throughput in advance.

Availability and limitations depend on the selected account configuration and API.

---

## 8. Consistency Levels

Consistency defines how updates are observed by clients across replicas.

Cosmos DB provides five consistency levels.

### 8.1 Strong Consistency

Strong consistency guarantees that reads return the most recent committed write.

It provides the strongest consistency guarantee but can involve higher latency and availability trade-offs.

### 8.2 Bounded Staleness

Bounded staleness allows reads to lag behind writes within a configured limit.

The limit is expressed using:
- A maximum number of versions.
- A maximum time interval.

### 8.3 Session Consistency

Session consistency provides read-your-writes and related session guarantees.

It is commonly useful for user-facing applications.

Example:

A user updates their profile and then reads it again within the same session. Session consistency helps ensure the user sees their own update.

### 8.4 Consistent Prefix

Consistent Prefix ensures that reads do not observe writes out of order.

However, some recent writes may not yet be visible.

### 8.5 Eventual Consistency

Eventual consistency provides the weakest consistency guarantee.

If no new updates occur, replicas eventually converge to the same value.

It can offer greater flexibility for globally distributed workloads.

### Consistency comparison

| Level | Main guarantee |
|---|---|
| Strong | Reads return the latest committed write |
| Bounded Staleness | Reads may lag within a configured bound |
| Session | Read-your-writes within a session |
| Consistent Prefix | Writes are observed in order |
| Eventual | Replicas eventually converge |

### Simple example

Suppose a user updates their account balance in one region.

Different consistency levels determine how quickly and under what guarantees that update becomes visible to reads from other regions.

**Interview point:** Strong consistency offers the strongest read guarantee, while eventual consistency offers weaker guarantees that may suit some globally distributed workloads.

---

## 9. Global Distribution and Replication

Cosmos DB supports distributing an account across multiple Azure regions.

### Example architecture

```text
                 Global Application
                         |
              +----------+----------+
              |          |          |
              v          v          v
          India Region  US Region  Europe Region
              |          |          |
              +----------+----------+
                         |
                  Cosmos DB Account
                         |
                 Replicated Data
```

### Multi-region configuration

Depending on the account configuration, Cosmos DB can support:
- One or multiple read regions.
- Multiple write regions.
- Regional failover.
- Automatic or manual failover options.

### Single-region writes

In a single-write-region configuration, one region handles writes.

Other configured regions can serve reads.

### Multi-region writes

With multi-region writes, applications can write to multiple configured regions.

This can reduce write latency for geographically distributed applications.

Conflict resolution is important when concurrent updates occur.

### Failover

If a region becomes unavailable, Cosmos DB can use configured failover behavior to continue serving requests from available regions.

The exact behavior depends on the account's regional and failover configuration.

---

## 10. Indexing in Cosmos DB

Indexing helps Cosmos DB retrieve data efficiently.

For the NoSQL API, indexing is automatic by default for properties in JSON items, subject to the configured indexing policy.

### Benefits

- Faster queries.
- Efficient filtering.
- Improved query performance.
- Reduced unnecessary data scanning.

### Indexing policy

An indexing policy controls which paths are indexed.

It can be customized to include or exclude paths depending on query requirements.

### Example

Suppose documents contain:

```json
{
  "id": "ORD1001",
  "customerId": "C101",
  "status": "Delivered",
  "amount": 65000,
  "metadata": {
    "source": "mobile"
  }
}
```

If queries frequently filter on `status` and `customerId`, the indexing policy should support those access patterns.

### Important points

- Indexing consumes storage and can add write overhead.
- Indexing policies should match query patterns.
- Cross-partition queries may still be expensive even when indexes exist.
- Indexing behavior and available index types vary by API.

---

## 11. Querying Cosmos DB

The NoSQL API supports a SQL-like query language for JSON documents.

### Example data

```json
{
  "id": "ORD1001",
  "customerId": "C101",
  "amount": 65000,
  "status": "Delivered"
}
```

### Retrieve all items

```sql
SELECT *
FROM c
```

### Filter records

```sql
SELECT *
FROM c
WHERE c.status = "Delivered"
```

### Select specific fields

```sql
SELECT
    c.id,
    c.customerId,
    c.amount
FROM c
```

### Aggregate records

```sql
SELECT
    c.status,
    COUNT(1) AS total_orders
FROM c
GROUP BY c.status
```

### Filter by partition key

If `customerId` is the partition key:

```sql
SELECT *
FROM c
WHERE c.customerId = "C101"
```

Providing the partition key can help Cosmos DB target the relevant partition rather than fan out across multiple partitions.

Query efficiency depends on the query, partition key, indexes, and data distribution.

---

## 12. Change Feed

The **Cosmos DB Change Feed** provides a persistent record of changes to items in a container.

It allows applications to process changes as they become available.

### Common use cases

- Event-driven processing.
- Data synchronization.
- Incremental data pipelines.
- Search index updates.
- Triggering downstream applications.
- Maintaining analytical copies of operational data.

### Example architecture

```text
             Application
                  |
                  v
             Cosmos DB
                  |
              New / Updated
                  |
                  v
             Change Feed
                  |
                  v
         Change Feed Processor
                  |
          +-------+-------+
          |               |
          v               v
       ADLS Gen2       Event Hub
          |               |
          v               v
      Databricks      Consumers
```

### How it works

1. An application inserts or updates an item.
2. Cosmos DB records the change in its Change Feed.
3. A consumer reads the available changes.
4. The consumer processes the changed items.
5. The consumer maintains progress using its supported checkpoint mechanism.

### Important points

- Change Feed is useful for incremental processing.
- It is not the same as a full database export.
- The NoSQL API's default latest-version mode primarily exposes inserts and updates.
- Deletes require additional consideration; all-versions-and-deletes mode has specific prerequisites and retention constraints.
- Consumers should be designed to handle retries and duplicate processing safely.

---

## 13. Azure Cosmos DB in Data Engineering

Cosmos DB is commonly used as an operational NoSQL source or serving database.

It can integrate with data engineering services for downstream analytics.

### Example architecture

```text
              Application
                   |
                   v
               Cosmos DB
                   |
              Change Feed
                   |
                   v
            Incremental Ingestion
                   |
                   v
               ADLS Gen2
                   |
                   v
            Azure Databricks
                   |
             PySpark Processing
                   |
                   v
              Delta Lake
                   |
                   v
                Power BI
```

### Example scenario

An online shopping application stores customer activity and shopping cart data in Cosmos DB.

The data engineering team wants to analyze customer behavior.

1. The application writes activity data to Cosmos DB.
2. Change Feed captures new and updated items.
3. A downstream consumer processes the changes.
4. Data is stored in ADLS Gen2.
5. Databricks cleans and transforms the data.
6. Curated Delta tables are created.
7. Power BI uses the curated data for analysis.

### Why use Cosmos DB?

- It supports high-volume application workloads.
- It provides flexible JSON document storage.
- It supports global distribution.
- Change Feed enables incremental downstream processing.

**Important:** Cosmos DB is generally an operational database, not a replacement for a data warehouse or distributed analytical processing engine.

---

## 14. Cosmos DB vs Azure SQL Database

These services support different data models and workload patterns.

| Feature | Cosmos DB | Azure SQL Database |
|---|---|---|
| Database type | NoSQL | Relational |
| Data model | Documents and other API-specific models | Tables and relational structures |
| Schema | Flexible, depending on API and application design | Structured relational schema |
| Querying | API-specific query language | T-SQL |
| Scaling | Horizontal partitioning and distributed throughput | Managed relational compute and storage scaling |
| Global distribution | Core capability | Different geo-replication and availability options |
| Transactions | API-specific transaction scope and capabilities | Relational transactions |
| Common use | Globally distributed applications | Relational transactional applications |

### When to choose Cosmos DB

Choose Cosmos DB when:
- The application needs flexible document storage.
- Global distribution is important.
- Low-latency access across regions is required.
- Workloads need horizontal scaling.
- Application access patterns align with partitioned data.

### When to choose Azure SQL Database

Choose Azure SQL Database when:
- Data has clear relational structure.
- Relationships and joins are central.
- Relational constraints and transactions are important.
- Applications rely on T-SQL and SQL Server compatibility.

Neither database is universally better. Choose based on application requirements and access patterns.

---

## 15. Cosmos DB vs Azure Data Lake Storage

These services are not direct replacements.

| Feature | Cosmos DB | ADLS Gen2 |
|---|---|---|
| Main purpose | Operational database | Data lake storage |
| Data access | Database APIs and queries | File and directory access |
| Typical data | Application records and documents | Raw, processed, and analytical files |
| Workload | Low-latency application access | Large-scale data storage and analytics |
| Query processing | Database query engine | External engines such as Spark and Synapse |
| Common use | Application serving layer | Data engineering storage layer |

### Example

A company may store customer profiles in Cosmos DB and export activity data into ADLS Gen2 for analytical processing.

```text
        Application
             |
             v
          Cosmos DB
             |
        Change Feed
             |
             v
          ADLS Gen2
             |
             v
         Databricks
             |
             v
        Delta Lake
```

---

## 16. Common Errors and Troubleshooting

| Problem | Possible reason | What to check |
|---|---|---|
| Request throttled | RU/s capacity exceeded | RU consumption and retry behavior |
| High query cost | Inefficient query or cross-partition scan | Query metrics and partition key |
| Hot partition | Uneven partition-key distribution | Partition-key choice and traffic |
| High latency | Region distance or expensive query | Region configuration and query design |
| Access denied | Missing identity or permissions | Authentication and data-plane roles |
| Data not visible immediately | Consistency behavior or replication delay | Configured consistency level |
| Duplicate downstream records | Consumer retries or repeated processing | Idempotency and checkpointing |
| High storage usage | Data growth or indexing overhead | Item size, retention, indexing policy |

### Troubleshooting approach

1. Check request charge in RUs.
2. Review query metrics.
3. Verify the partition key.
4. Inspect partition distribution and throughput.
5. Check the configured consistency level.
6. Review region and network configuration.
7. Verify authentication and permissions.
8. Inspect Change Feed consumer progress and retry behavior.

---

## 17. Common Interview Questions and Answers

### Q1. What is Azure Cosmos DB?

Azure Cosmos DB is a fully managed, globally distributed NoSQL database service designed for low-latency, highly available, and horizontally scalable applications.

### Q2. Is Cosmos DB SQL or NoSQL?

Cosmos DB is a NoSQL database service. Its NoSQL API supports a SQL-like query language for querying JSON documents.

### Q3. What are the main components of Cosmos DB?

For the NoSQL API, the main hierarchy is Account, Database, Container, and Item.

### Q4. What is a partition key?

A partition key is a property used to distribute items across logical partitions. A good partition key helps balance storage and request traffic.

### Q5. What is the difference between logical and physical partitions?

A logical partition contains items with the same partition-key value. A physical partition is the underlying storage and compute partition managed by Cosmos DB.

### Q6. What happens if the partition key is poorly selected?

It can cause uneven data distribution, hot partitions, increased query cost, and throughput bottlenecks.

### Q7. What are Request Units?

Request Units measure the normalized resource consumption of database operations. RU/s represents the throughput capacity allocated to the workload.

### Q8. What is a 429 error?

A 429 response indicates that a request was throttled because the workload exceeded available throughput at that time. Applications should follow the service's retry guidance.

### Q9. What are the five consistency levels?

Strong, Bounded Staleness, Session, Consistent Prefix, and Eventual.

### Q10. What is Session consistency?

Session consistency provides read-your-writes guarantees within a session, helping users see their own updates.

### Q11. What is Change Feed?

Change Feed provides a way to consume changes to items in a Cosmos DB container for downstream processing and synchronization.

### Q12. How is Cosmos DB useful in data engineering?

It can act as an operational source, while Change Feed enables incremental ingestion into ADLS Gen2, Databricks, or other analytical systems.

### Q13. What is the difference between provisioned and autoscale throughput?

Provisioned throughput allocates a configured RU/s capacity. Autoscale adjusts throughput within a configured maximum based on workload demand.

### Q14. What is the difference between Cosmos DB and Azure SQL Database?

Cosmos DB is a NoSQL database designed for distributed application workloads, while Azure SQL Database is a managed relational database suited to structured relational workloads.

### Q15. How do you reduce RU consumption?

Use an appropriate partition key, optimize queries, retrieve only required fields, use indexes effectively, and avoid unnecessary cross-partition operations.

---

## 18. Common Mistakes to Avoid

1. Thinking Cosmos DB is a relational database.
2. Confusing a database with a container.
3. Confusing logical and physical partitions.
4. Choosing a partition key without considering query patterns.
5. Assuming high cardinality alone guarantees a good partition key.
6. Ignoring hot partitions.
7. Treating RU/s as the same thing as CPU utilization.
8. Assuming every query is automatically inexpensive because indexing is enabled.
9. Confusing consistency levels with replication regions.
10. Assuming multi-region replication means every read immediately returns the latest write.
11. Treating Change Feed as a complete database backup.
12. Assuming the default Change Feed mode automatically provides every delete event.
13. Using Cosmos DB as a replacement for a data warehouse.
14. Assuming all Cosmos DB APIs have identical features and query behavior.

---

## 19. Quick Revision

| Concept | Remember |
|---|---|
| Azure Cosmos DB | Globally distributed NoSQL database |
| Account | Top-level Azure resource |
| Database | Logical grouping of containers |
| Container | Stores and partitions items |
| Item | Individual JSON record in NoSQL API |
| Partition key | Distributes items |
| Logical partition | Items sharing a partition-key value |
| Physical partition | Underlying managed partition |
| RU | Request Unit |
| RU/s | Throughput capacity |
| Strong consistency | Latest committed write guarantee |
| Session consistency | Read-your-writes guarantee |
| Eventual consistency | Replicas eventually converge |
| Change Feed | Incremental stream of item changes |
| Autoscale | Adjusts throughput within configured maximum |
| Global distribution | Replication across Azure regions |
| Hot partition | Uneven traffic or storage concentration |

### Remember these five points

1. Cosmos DB is a globally distributed NoSQL database service.
2. Its NoSQL API stores JSON items in containers.
3. Partition-key design is critical for scalability and performance.
4. RU/s measures allocated throughput capacity.
5. Change Feed supports incremental data integration into downstream systems.

---

## 20. Self-Assessment

Try answering these questions without looking at the notes:

1. What is Azure Cosmos DB?
2. Why is Cosmos DB considered a NoSQL database?
3. Explain Account, Database, Container, and Item.
4. What are the different Cosmos DB APIs?
5. What is a partition key?
6. Explain logical and physical partitions.
7. What makes a good partition key?
8. What is a hot partition?
9. What are Request Units and RU/s?
10. What happens when a workload exceeds its available RU/s?
11. Explain the five consistency levels.
12. What is the difference between provisioned and autoscale throughput?
13. What is Change Feed?
14. How can Change Feed support incremental ingestion into ADLS Gen2?
15. Compare Cosmos DB with Azure SQL Database.
16. When would you choose Cosmos DB instead of ADLS Gen2?
17. How would you troubleshoot high RU consumption?

**Interview preparation priority:** Focus on Cosmos DB fundamentals, APIs, partitioning, partition-key selection, RU/s, consistency levels, global distribution, Change Feed, and integration with ADLS Gen2 and Databricks.