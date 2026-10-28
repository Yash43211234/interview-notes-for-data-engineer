# Primary Keys and Foreign Keys in SQL

## 1. What Is a Primary Key?

A primary key is a column or combination of columns that uniquely identifies every row in a table.

**Main rules:**
- Every primary key value must be unique.
- Primary key columns cannot contain `NULL`.
- A table can have only one primary key constraint.
- A primary key can contain multiple columns, forming a composite primary key.

### Example

```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100),
    city VARCHAR(50)
);
```

Here, `customer_id` uniquely identifies each customer.

Valid data:

| customer_id | customer_name | city |
|---:|---|---|
| 101 | Amit | Delhi |
| 102 | Priya | Mumbai |
| 103 | Rahul | Pune |

Invalid data:

| customer_id | customer_name | city |
|---:|---|---|
| 101 | Amit | Delhi |
| 101 | Priya | Mumbai |

The second row violates the primary key constraint because `customer_id = 101` already exists.

A row with `customer_id = NULL` would also violate the constraint.

## 2. What Is a Foreign Key?

A foreign key is a column or combination of columns that references a primary key or another eligible unique key in a referenced table.

It helps maintain **referential integrity**, ensuring that a non-null foreign key value refers to an existing key in the referenced table.

### Example

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
);
```

Here:
- `order_id` uniquely identifies each order.
- `customer_id` references a customer in the `customers` table.
- An order cannot reference a nonexistent customer when the foreign key constraint is enforced.

Valid data:

| order_id | customer_id | order_date |
|---:|---:|---|
| 1001 | 101 | 2026-01-10 |
| 1002 | 102 | 2026-01-11 |

Invalid data:

| order_id | customer_id | order_date |
|---:|---:|---|
| 1003 | 999 | 2026-01-12 |

If customer 999 does not exist, this insert violates the foreign key constraint.

**Important:** A foreign key can generally contain `NULL` unless the column is also defined as `NOT NULL`. A `NULL` foreign key does not reference a parent row.

## 3. Primary Key vs. Foreign Key

| Feature | Primary Key | Foreign Key |
|---|---|---|
| Purpose | Uniquely identifies a row | Establishes a relationship between tables |
| Uniqueness | Must be unique | Values can repeat |
| `NULL` values | Not allowed | Allowed unless restricted by `NOT NULL` |
| Number per table | One primary key constraint | Multiple foreign key constraints are allowed |
| References another table | Not required | References an eligible key |
| Main benefit | Entity integrity | Referential integrity |

## 4. How Primary Keys and Foreign Keys Work Together

Consider two tables: `customers` and `orders`.

<box border radius="lg" padding={3} gap={3}>
  <box background="surface-secondary" radius="md" padding={3} gap={1}>
    **customers**

    | customer_id (PK) | customer_name |
    |---:|---|
    | 101 | Amit |
    | 102 | Priya |
    | 103 | Rahul |
  </box>
  <row align="center" justify="center" gap={2}>
    <icon name="arrow-down" size="xl"/>
    <text color="secondary" size="sm">orders.customer_id references customers.customer_id</text>
  </row>
  <box background="surface" border radius="md" padding={3} gap={1}>
    **orders**

    | order_id (PK) | customer_id (FK) |
    |---:|---:|
    | 1001 | 101 |
    | 1002 | 101 |
    | 1003 | 102 |
  </box>
</box>

Customer 101 has two orders. Therefore, the foreign key can repeat.

This represents a **one-to-many relationship**: one customer can have many orders.

## 5. Composite Primary Key

A composite primary key uses two or more columns together to uniquely identify a row.

Example: A student can enroll in multiple courses, and each course can contain multiple students.

```sql
CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    PRIMARY KEY (student_id, course_id)
);
```

The combination of `student_id` and `course_id` must be unique.

Valid data:

| student_id | course_id | enrollment_date |
|---:|---:|---|
| 1 | 101 | 2026-01-10 |
| 1 | 102 | 2026-01-12 |
| 2 | 101 | 2026-01-15 |

The same student can enroll in different courses, but the same student-course combination cannot appear twice.

## 6. Composite Foreign Key

A composite foreign key references a candidate key consisting of multiple columns in another table.

```sql
CREATE TABLE course_sections (
    course_id INT,
    section_id INT,
    room_number VARCHAR(20),
    PRIMARY KEY (course_id, section_id)
);
```

A table that references a specific course section can use:

```sql
CREATE TABLE student_enrollments (
    student_id INT,
    course_id INT,
    section_id INT,
    PRIMARY KEY (student_id, course_id, section_id),
    FOREIGN KEY (course_id, section_id)
        REFERENCES course_sections(course_id, section_id)
);
```

The referenced column combination must be a valid referenced key under the database engine's rules.

## 7. Foreign Key Actions: ON DELETE and ON UPDATE

Foreign key constraints can specify what happens when a referenced key is deleted or updated.

### CASCADE

Changes or deletions in the parent propagate to matching child rows.

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
        ON DELETE CASCADE
);
```

Deleting a customer can delete that customer's associated orders.

### SET NULL

Sets the child foreign key to `NULL` when the referenced parent row is deleted or its key is updated, depending on the configured action.

```sql
FOREIGN KEY (customer_id)
    REFERENCES customers(customer_id)
    ON DELETE SET NULL
```

The foreign key column must allow `NULL`.

### RESTRICT / NO ACTION

Prevents a parent key from being deleted or changed when referencing child rows exist, subject to the database's constraint-checking rules.

**Interview point:** `CASCADE` should be used carefully because a single parent deletion may remove many child records.

## 8. Can a Foreign Key Reference a Non-Primary-Key Column?

Yes, in many relational databases, a foreign key can reference a column or column combination backed by an eligible `UNIQUE` constraint or unique key, provided the database supports it.

Example:

```sql
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_code VARCHAR(10) UNIQUE
);
```

A related table may reference `department_code`:

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    department_code VARCHAR(10),
    FOREIGN KEY (department_code)
        REFERENCES departments(department_code)
);
```

The exact requirements for referenced keys depend on the database engine.

## 9. Primary Key vs. UNIQUE Constraint

Both primary keys and unique constraints enforce uniqueness, but they differ in important ways.

| Feature | Primary Key | UNIQUE Constraint |
|---|---|---|
| Purpose | Main row identifier | Enforces uniqueness of a key |
| Multiple constraints per table | Only one primary key | Multiple unique constraints are possible |
| `NULL` handling | Not allowed | Database-specific behavior |
| Foreign key target | Can be referenced | Can often be referenced if eligible |

For example, `employee_id` may be the primary key, while `email` has a unique constraint.

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    email VARCHAR(150) UNIQUE,
    employee_name VARCHAR(100)
);
```

## 10. Do Foreign Keys Automatically Create Indexes?

Not universally.

- Primary keys are commonly backed by a unique index.
- Unique constraints are commonly backed by unique indexes.
- Foreign keys do not automatically create indexes on the referencing columns in every database.

An index on a foreign key can improve joins and help the database locate child rows during parent updates or deletions.

Example:

```sql
CREATE INDEX idx_orders_customer_id
ON orders(customer_id);
```

Consider existing indexes and the actual workload before creating additional ones.

## 11. Common Interview Questions

**Q1. Can a table have multiple primary keys?**

No. A table can have only one primary key constraint, although that key can contain multiple columns.

**Q2. Can a foreign key contain duplicate values?**

Yes. Multiple orders can reference the same customer.

**Q3. Can a foreign key contain `NULL`?**

Yes, unless the column is also defined as `NOT NULL`.

**Q4. Can a foreign key reference another table's foreign key?**

It can reference a column set that qualifies as a referenced key under the database engine's rules. A foreign key column is not automatically eligible simply because it is itself a foreign key.

**Q5. What happens if you insert a child row before its parent row?**

If the referenced parent key does not exist when the foreign key is checked, the insert fails. Some databases support deferred constraint checking within a transaction.

**Q6. What is referential integrity?**

It ensures that foreign key references remain valid according to the defined constraints.

**Q7. What is the difference between a primary key and a candidate key?**

A candidate key is any minimal set of columns that uniquely identifies a row. One candidate key is selected as the primary key.

## 12. Common Mistakes

- Assuming foreign key values must be unique.
- Assuming foreign keys cannot contain `NULL`.
- Assuming a foreign key automatically creates an index in every database.
- Forgetting that a composite key is unique only as a combination.
- Using `ON DELETE CASCADE` without considering downstream data loss.
- Assuming every unique-looking column is automatically a valid foreign key target.
- Confusing database constraints with indexes.

## 13. Interview Practice

1. Create `customers` and `orders` tables with primary and foreign keys.
2. Find customers who have no orders using a `LEFT JOIN`.
3. Explain why several orders can share the same `customer_id`.
4. Create a composite primary key for a student-course enrollment table.
5. Demonstrate how a foreign key prevents inserting an invalid customer ID.
6. Explain the difference between `ON DELETE CASCADE` and `ON DELETE SET NULL`.
7. Explain why indexing a foreign key may improve join performance.

## Quick Revision

- **Primary key:** Uniquely identifies a row; cannot be `NULL`.
- **Foreign key:** References an eligible key in another table.
- **Referential integrity:** Keeps references valid.
- **Composite key:** Uses multiple columns together.
- **`ON DELETE CASCADE`:** Deletes dependent child rows when the referenced parent row is deleted.
- **`UNIQUE`:** Enforces uniqueness without necessarily serving as the primary key.
- **Indexing:** Primary keys are commonly indexed; foreign key indexing depends on the database and workload.

**Core interview takeaway:** Primary keys identify records; foreign keys connect records and enforce valid relationships between tables.
