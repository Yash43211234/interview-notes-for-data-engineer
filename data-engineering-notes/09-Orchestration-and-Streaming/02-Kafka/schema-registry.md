

# Kafka Schema Registry

## 1. Introduction

In Kafka, producers publish records and consumers read them. Both applications need to understand the structure and data types of those records.

For example, an order event might contain:

```json
{
  "order_id": 101,
  "customer_id": "C01",
  "amount": 500.0,
  "status": "PLACED"
}
```

Over time, the structure may change:
- A new field is added.
- A field is removed.
- A field's data type changes.
- A field becomes optional.

If producers and consumers use incompatible schemas, data processing can fail.

**Schema Registry** helps manage schemas and enforce compatibility rules between producers and consumers.

It is commonly used with Kafka and serialization formats such as Avro, Protobuf, and JSON Schema.

---

## 2. What Is Schema Registry?

Schema Registry is a centralized service that stores, manages, and versions schemas used by Kafka applications.

It helps producers and consumers agree on the structure of messages.

### Without Schema Registry

```text
Producer A                 Consumer A
    |                           |
    v                           v
Uses Schema A              Assumes Schema B
    |                           |
    └──────── Kafka ────────────┘
                  |
                  v
           Compatibility Error
```

The producer and consumer may interpret the same message differently.

### With Schema Registry

```text
              Schema Registry
              /              \
             v                v
       Producer             Consumer
             \                /
              \              /
               v            v
                Kafka Topic
```

The producer registers or retrieves the schema, and the consumer retrieves the appropriate schema to deserialize the record.

### Main responsibilities

1. Store schemas centrally.
2. Maintain schema versions.
3. Assign schema IDs.
4. Enforce compatibility rules.
5. Support schema evolution.
6. Help producers and consumers interpret records consistently.

**Important:** Schema Registry is not a Kafka broker and does not store the actual business records. Kafka stores the records; Schema Registry manages their schemas.

---

## 3. Why Do We Need Schema Registry?

Consider an e-commerce application.

Initially, the producer sends:

```json
{
  "order_id": 101,
  "amount": 500
}
```

Later, the business adds a customer field:

```json
{
  "order_id": 101,
  "amount": 500,
  "customer_id": "C01"
}
```

An older consumer may not understand the new schema if it expects a different structure or if the serialization format requires a field that is missing.

Schema Registry helps manage these changes through versioning and compatibility checks.

### Benefits

- Consistent data structure.
- Controlled schema evolution.
- Reduced serialization and deserialization errors.
- Centralized schema management.
- Better producer-consumer compatibility.
- Improved data governance.

---

## 4. What Is a Schema?

A schema defines the structure of a record.

It describes:
- Field names.
- Data types.
- Required or optional fields.
- Default values.
- Nested structures.
- Additional constraints supported by the format.

Example Avro schema:

```json
{
  "type": "record",
  "name": "Order",
  "namespace": "com.example",
  "fields": [
    {
      "name": "order_id",
      "type": "long"
    },
    {
      "name": "customer_id",
      "type": "string"
    },
    {
      "name": "amount",
      "type": "double"
    },
    {
      "name": "status",
      "type": "string"
    }
  ]
}
```

This schema defines an `Order` record containing four fields.

### Common schema formats

| Format | Description |
|---|---|
| Avro | Row-oriented serialization format with schema support |
| Protobuf | Language-neutral serialization format using message definitions |
| JSON Schema | Defines and validates JSON document structure |

All three can be used with compatible Schema Registry implementations, depending on the platform.

---

## 5. What Is Avro?

Apache Avro is a data serialization system commonly used with Kafka.

It uses schemas to define record structure and supports compact binary serialization.

### Example Avro schema

```json
{
  "type": "record",
  "name": "Customer",
  "fields": [
    {
      "name": "customer_id",
      "type": "string"
    },
    {
      "name": "name",
      "type": "string"
    },
    {
      "name": "age",
      "type": "int"
    }
  ]
}
```

### Why use Avro?

- Compact binary representation.
- Schema-based serialization.
- Supports schema evolution.
- Useful for structured event data.
- Widely used in Kafka ecosystems.

### Avro vs JSON

| Avro | JSON |
|---|---|
| Commonly uses binary encoding | Text-based representation |
| Uses an explicit schema | Structure may be inferred or validated separately |
| Usually more compact | Usually more verbose |
| Schema evolution supported through compatibility rules | Schema evolution requires separate validation and management |

**Important:** Avro does not automatically guarantee compatibility. Compatibility depends on the schemas and the configured rules.

---

## 6. How Schema Registry Works

A typical producer-consumer workflow:

```text
                Schema Registry
                       |
              Register / Retrieve
                       |
                       v
Producer ───────> Kafka Topic ───────> Consumer
    |                                     |
    v                                     v
Serialize using schema              Deserialize using schema
```

### Producer workflow

1. Producer creates a record.
2. Producer uses an Avro, Protobuf, or JSON Schema serializer.
3. Serializer checks Schema Registry for the schema.
4. If necessary, the schema is registered under a subject.
5. Registry returns a schema ID.
6. Serializer encodes the record with schema identification information.
7. Producer publishes the serialized record to Kafka.

### Consumer workflow

1. Consumer reads the serialized record.
2. Deserializer extracts the schema identification information.
3. Deserializer retrieves the writer schema from Schema Registry if it is not already cached.
4. The record is decoded.
5. The consumer obtains the application-level object.

### Important point

The schema is not normally copied into every message in full. With common Confluent-compatible wire formats, the message contains a schema ID, and the deserializer uses that ID to retrieve the schema.

---

## 7. Subject and Schema ID

Two important Schema Registry concepts are:

- **Subject:** A named scope under which schemas are versioned and compatibility is checked.
- **Schema ID:** An identifier assigned to a registered schema.

### Example

Suppose we have an `orders` topic.

A common subject naming strategy is:

```text
orders-value
```

This represents the value schema for the `orders` topic.

Another subject could be:

```text
orders-key
```

This represents the key schema.

### Subject naming strategies

Common strategies include:

| Strategy | Subject naming |
|---|---|
| TopicNameStrategy | `<topic>-key` or `<topic>-value` |
| RecordNameStrategy | Uses the fully qualified record name |
| TopicRecordNameStrategy | Combines topic name and record name |

The default in many Confluent serializers is `TopicNameStrategy`.

### Example

```text
Topic: orders

Subjects:
  orders-key
  orders-value

Versions:
  orders-value
    ├── Version 1
    ├── Version 2
    └── Version 3
```

Each subject maintains its own schema versions.

### Schema ID vs Version

| Schema ID | Schema version |
|---|---|
| Identifies a registered schema in the registry | Identifies a schema's position within a subject |
| Can be shared across subjects when the same schema is registered | Scoped to a subject |
| Used by serializers and deserializers | Used to manage schema evolution |

A schema ID and a subject version are not interchangeable.

---

## 8. What Is Schema Evolution?

Schema evolution means changing a schema over time while maintaining compatibility with existing producers and consumers.

Example:

### Version 1

```json
{
  "type": "record",
  "name": "Order",
  "fields": [
    {
      "name": "order_id",
      "type": "long"
    },
    {
      "name": "amount",
      "type": "double"
    }
  ]
}
```

### Version 2

A new optional field is added:

```json
{
  "type": "record",
  "name": "Order",
  "fields": [
    {
      "name": "order_id",
      "type": "long"
    },
    {
      "name": "amount",
      "type": "double"
    },
    {
      "name": "status",
      "type": ["null", "string"],
      "default": null
    }
  ]
}
```

The new field has a default value of `null`, allowing compatible readers to handle records that do not contain it.

### Common schema changes

- Adding a field.
- Removing a field.
- Renaming a field.
- Changing a data type.
- Changing a default value.
- Making a field optional or required.

Not every change is compatible. Compatibility depends on the schema format and configured compatibility mode.

---

## 9. Schema Compatibility Types

Schema Registry supports different compatibility modes.

The most important ones are:

1. BACKWARD
2. FORWARD
3. FULL
4. BACKWARD_TRANSITIVE
5. FORWARD_TRANSITIVE
6. FULL_TRANSITIVE
7. NONE

### A. BACKWARD compatibility

A new schema can read data written using the previous schema.

Example:

```text
Old Producer → Old Schema → Record
                              |
                              v
                       New Consumer
                       New Schema
```

The new consumer must be able to read old records.

For Avro, adding a field with a default value is a common backward-compatible change.

### B. FORWARD compatibility

The previous schema can read data written using the new schema.

Example:

```text
New Producer → New Schema → Record
                              |
                              v
                       Old Consumer
                       Old Schema
```

The old consumer must be able to read records produced with the new schema.

For Avro, removing a field can be forward-compatible when the old reader schema supplies an appropriate default for that field.

### C. FULL compatibility

Both backward and forward compatibility are required.

```text
Old Consumer ← New Producer
New Consumer ← Old Producer
```

Both directions must work.

### D. Transitive compatibility

Non-transitive compatibility checks the new schema against the latest registered version.

Transitive compatibility checks the new schema against all relevant previous versions.

| Mode | Compatibility check |
|---|---|
| BACKWARD | Latest previous version |
| BACKWARD_TRANSITIVE | All previous versions |
| FORWARD | Latest previous version |
| FORWARD_TRANSITIVE | All previous versions |
| FULL | Latest previous version in both directions |
| FULL_TRANSITIVE | All previous versions in both directions |
| NONE | No compatibility checks |

### E. NONE

No compatibility checks are performed before accepting a schema.

This gives flexibility but increases the risk of breaking consumers.

### Quick comparison

| Compatibility | New reader reads old data | Old reader reads new data |
|---|---|---|
| BACKWARD | Yes | Not required |
| FORWARD | Not required | Yes |
| FULL | Yes | Yes |

**Interview point:** Backward means new consumers can read old data. Forward means old consumers can read new data.

---

## 10. Schema Registry Compatibility Example

Suppose the original schema is:

```text
Version 1:
order_id
amount
```

A new version adds:

```text
Version 2:
order_id
amount
status (default = null)
```

For Avro:

- A new reader can read old records because it can supply the default for the missing `status` field.
- An old reader can generally read new records because it ignores the additional `status` field.

This change is typically both backward- and forward-compatible under the stated assumptions.

However, changing field types, removing fields, and changing defaults must be evaluated according to the specific format and reader/writer rules.

---

## 11. Schema Registry API and Operations

Schema Registry exposes REST APIs for managing schemas.

Common operations include:

- Register a schema.
- Retrieve a schema by ID.
- List subject versions.
- Retrieve a specific subject version.
- Check compatibility.
- Update compatibility settings.

### Example REST endpoints

Register a schema:

```http
POST /subjects/orders-value/versions
```

Get all versions:

```http
GET /subjects/orders-value/versions
```

Get a particular version:

```http
GET /subjects/orders-value/versions/2
```

Get a schema by ID:

```http
GET /schemas/ids/123
```

Check compatibility:

```http
POST /compatibility/subjects/orders-value/versions/latest
```

These are endpoint paths; actual requests also require the appropriate payload, headers, and authentication.

---

## 12. Schema Registry with Kafka Producer

A typical Confluent-compatible Avro producer uses a serializer that communicates with Schema Registry.

Illustrative Python example:

```python
from confluent_kafka import SerializingProducer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroSerializer
from confluent_kafka.serialization import StringSerializer

schema_str = """
{
  "type": "record",
  "name": "Order",
  "fields": [
    {"name": "order_id", "type": "long"},
    {"name": "amount", "type": "double"}
  ]
}
"""

schema_registry_conf = {
    "url": "http://localhost:8081"
}

schema_registry_client = SchemaRegistryClient(
    schema_registry_conf
)

avro_serializer = AvroSerializer(
    schema_registry_client,
    schema_str
)

producer_conf = {
    "bootstrap.servers": "localhost:9092",
    "key.serializer": StringSerializer("utf_8"),
    "value.serializer": avro_serializer
}

producer = SerializingProducer(producer_conf)

producer.produce(
    topic="orders",
    key="order-101",
    value={
        "order_id": 101,
        "amount": 500.0
    }
)

producer.flush()
```

### What happens here?

1. The application defines an Avro schema.
2. It creates a Schema Registry client.
3. It creates an Avro serializer.
4. The serializer registers or retrieves the schema as needed.
5. The producer serializes the record and publishes it.

**Note:** This is an illustrative local-development example. Production systems should configure authentication, TLS, error handling, delivery callbacks, and appropriate subject naming and compatibility rules. Exact library APIs can vary by version.

---

## 13. Schema Registry with Kafka Consumer

The consumer uses a compatible deserializer to decode records.

Illustrative Python example:

```python
from confluent_kafka import DeserializingConsumer
from confluent_kafka.schema_registry import SchemaRegistryClient
from confluent_kafka.schema_registry.avro import AvroDeserializer
from confluent_kafka.serialization import StringDeserializer

schema_registry_client = SchemaRegistryClient({
    "url": "http://localhost:8081"
})

avro_deserializer = AvroDeserializer(
    schema_registry_client
)

consumer_conf = {
    "bootstrap.servers": "localhost:9092",
    "group.id": "order-processors",
    "auto.offset.reset": "earliest",
    "key.deserializer": StringDeserializer("utf_8"),
    "value.deserializer": avro_deserializer
}

consumer = DeserializingConsumer(consumer_conf)

consumer.subscribe(["orders"])

try:
    while True:
        msg = consumer.poll(1.0)

        if msg is None:
            continue

        if msg.error():
            print(msg.error())
            continue

        order = msg.value()
        print(order)

finally:
    consumer.close()
```

### What happens here?

1. The consumer subscribes to the `orders` topic.
2. It fetches serialized records.
3. The Avro deserializer reads the schema ID.
4. It retrieves the writer schema from Schema Registry when needed.
5. It converts the bytes into a Python object.

---

## 14. Schema Registry vs Kafka

These are separate components.

| Kafka | Schema Registry |
|---|---|
| Distributed event-streaming platform | Schema management service |
| Stores and serves records | Stores and serves schemas |
| Manages topics and partitions | Manages subjects and schema versions |
| Handles producers and consumers | Supports serializers and deserializers |
| Provides replication and retention | Provides schema compatibility checks |

### Important point

Schema Registry does not replace Kafka. It works alongside Kafka to manage the structure of messages.

---

## 15. Schema Registry vs Data Contract

These concepts are related but not identical.

### Schema Registry

Manages technical message schemas and compatibility.

### Data contract

Defines broader expectations between data producers and consumers, potentially including:
- Schema.
- Data types.
- Required fields.
- Semantics and business definitions.
- Quality rules.
- Ownership.
- Service-level expectations.
- Change-management rules.

A schema can be registered successfully but still violate a business data contract.

Example:

A schema may allow `amount = -500`, even though the business rule says an order amount must be positive.

Schema compatibility alone does not validate every business rule.

---

## 16. Practical Data Engineering Scenario

Consider an e-commerce company sending order events through Kafka.

### Architecture

```text
E-commerce Application
          |
          v
       Producer
          |
          v
    Avro Serializer
          |
          v
   Schema Registry
          |
          v
    Kafka Topic: orders
          |
          v
       Consumer
          |
          v
    Avro Deserializer
          |
          v
   Schema Validation
          |
          v
   Spark / Databricks
          |
          v
       Delta Lake
```

### Scenario

Initially, the order event contains:

```text
order_id
amount
```

Later, the business adds:

```text
customer_id
status
```

The data engineering team should:

1. Define the new schema.
2. Check compatibility with existing versions.
3. Register the new schema.
4. Deploy compatible producers.
5. Ensure consumers can read the required versions.
6. Monitor deserialization errors.
7. Validate business rules after deserialization.

### Benefits

- Controlled schema changes.
- Fewer unexpected consumer failures.
- Better coordination between teams.
- Easier evolution of streaming pipelines.

---

## 17. Common Errors and Troubleshooting

| Problem | Possible cause | Approach |
|---|---|---|
| Schema registration fails | Compatibility violation | Check the subject's compatibility mode |
| Serialization fails | Record does not match schema | Validate field names and data types |
| Deserialization fails | Incorrect schema or incompatible data | Check schema ID and writer/reader schemas |
| Registry unavailable | Network or service issue | Check Registry health and connectivity |
| Unknown schema ID | Schema not available or wrong Registry | Verify schema ID and Registry endpoint |
| Authentication failure | Missing or invalid credentials | Check Registry authentication and permissions |
| New field causes errors | Missing default or incompatible type | Review schema evolution rules |
| Different applications use different schemas | Subject naming or deployment inconsistency | Review subject strategy and version management |

---

## 18. Best Practices

1. Define schemas before publishing production events.
2. Use a suitable serialization format such as Avro or Protobuf.
3. Configure compatibility rules for each subject.
4. Prefer backward compatibility when consumers need to read historical records.
5. Use transitive compatibility when all historical versions must remain readable.
6. Avoid unnecessary field renaming and type changes.
7. Use defaults carefully when adding fields.
8. Test producer and consumer compatibility before deployment.
9. Monitor schema registration and serialization failures.
10. Secure Schema Registry with appropriate authentication and TLS.
11. Document business meaning, not just field types.
12. Coordinate schema changes across producer and consumer teams.

---

## 19. Common Interview Questions and Answers

### Q1. What is Schema Registry?

A centralized service that stores, versions, and manages schemas used by Kafka producers and consumers.

### Q2. Why do we need Schema Registry?

It helps maintain consistent message structures and manage schema evolution without unexpectedly breaking consumers.

### Q3. Does Schema Registry store Kafka messages?

No. Kafka stores the records; Schema Registry stores the schemas.

### Q4. What is a schema?

A definition of record structure, including fields, data types, defaults, and other format-specific rules.

### Q5. What is Avro?

A schema-based serialization format that commonly uses compact binary encoding.

### Q6. What is a subject?

A named scope under which schemas are versioned and compatibility is checked.

### Q7. What is a schema ID?

An identifier assigned to a registered schema. Serializers can include it in the message so deserializers can retrieve the schema.

### Q8. What is the difference between schema ID and schema version?

A schema ID identifies a registered schema, while a version identifies its position within a subject.

### Q9. What is schema evolution?

Changing a schema over time while maintaining the required compatibility with existing applications.

### Q10. What is backward compatibility?

A new reader can read data written using an older schema.

### Q11. What is forward compatibility?

An older reader can read data written using a newer schema.

### Q12. What is full compatibility?

Both backward and forward compatibility are required.

### Q13. What is transitive compatibility?

It checks compatibility against all relevant previous versions rather than only the latest version.

### Q14. What happens if a producer sends data that does not match its schema?

The serializer can fail before publishing the record, depending on the serializer and configuration.

### Q15. Can Schema Registry validate business rules?

Not by itself. It manages schemas and compatibility; business validation generally belongs in the application or data-processing pipeline.

### Q16. Does Schema Registry guarantee that every consumer can process a message?

No. Compatibility rules reduce schema-related breakage, but consumers can still fail because of business logic, configuration, or implementation issues.

### Q17. What is the difference between Schema Registry and Kafka?

Kafka stores and distributes event records. Schema Registry manages the schemas used to serialize and deserialize those records.

### Q18. Can Schema Registry work with JSON?

Yes. Compatible implementations support JSON Schema as well as formats such as Avro and Protobuf.

### Q19. What is the purpose of a default value in Avro?

It allows a reader schema to supply a value when a field is absent from the writer schema, supporting certain schema evolution changes.

### Q20. What happens when an incompatible schema is registered?

The registry rejects registration when compatibility enforcement is enabled and the new schema violates the configured compatibility rules.

---

## 20. Common Mistakes to Avoid

1. **Confusing Kafka with Schema Registry:** Kafka stores records; Registry manages schemas.
2. **Confusing schema ID with version:** They have different scopes and purposes.
3. **Assuming every schema change is compatible:** Compatibility depends on format and configured rules.
4. **Confusing backward and forward compatibility:** Backward means new reader reads old data; forward means old reader reads new data.
5. **Assuming Registry validates business rules:** Schema compatibility is not complete business validation.
6. **Assuming the full schema is stored in every message:** Common wire formats use a schema ID.
7. **Assuming schema registration guarantees consumer success:** Application logic and deployment compatibility still matter.
8. **Ignoring subject naming strategy:** It affects how schemas are grouped and compatibility is enforced.
9. **Changing field types without testing:** Type changes can break readers.
10. **Ignoring transitive compatibility:** A schema compatible with the latest version may not be compatible with older versions.

---

## 21. Quick Revision

| Concept | Remember |
|---|---|
| Schema Registry | Centralized schema management |
| Schema | Defines record structure |
| Avro | Schema-based binary serialization format |
| Protobuf | Schema-based serialization format |
| JSON Schema | Defines JSON structure |
| Subject | Scope for schema versions and compatibility |
| Schema ID | Identifies a registered schema |
| Schema version | Version within a subject |
| Schema evolution | Controlled schema changes |
| BACKWARD | New reader reads old data |
| FORWARD | Old reader reads new data |
| FULL | Both directions |
| TRANSITIVE | Checks all relevant previous versions |
| NONE | No compatibility enforcement |
| Serializer | Converts records into bytes |
| Deserializer | Converts bytes into records |

### Remember these five points

1. Schema Registry manages schemas, not Kafka messages.
2. Producers serialize records using schemas.
3. Consumers use schemas to deserialize records.
4. Compatibility rules help control schema evolution.
5. Backward means new reader reads old data; forward means old reader reads new data.

---

## 22. Self-Assessment

Try answering these without looking at the notes.

1. What is Schema Registry, and why is it used?
2. What is the difference between Kafka and Schema Registry?
3. Explain the role of Avro.
4. What is a subject?
5. What is the difference between schema ID and schema version?
6. Explain the complete producer-to-consumer Schema Registry workflow.
7. What is schema evolution?
8. Explain backward, forward, and full compatibility.
9. What is the difference between transitive and non-transitive compatibility?
10. How would you add a new field without breaking existing consumers?
11. What happens if an incompatible schema is registered?
12. Does Schema Registry validate business rules?
13. How would you troubleshoot a deserialization error?
14. Design a Kafka pipeline that uses Avro and Schema Registry to write order events into Delta Lake.

**Interview readiness:** You should be able to explain why Schema Registry is needed, how schema IDs and subjects work, and how compatibility rules help producers and consumers evolve independently.

