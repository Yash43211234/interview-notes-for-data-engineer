# Database Connectivity in Python

## 1. What Is Database Connectivity?

**Database connectivity** is the process of establishing communication between an application and a database so that the application can read, insert, update, and delete data.

In Python, database connectivity is commonly used to:
- Extract data from relational databases.
- Load transformed data into destination tables.
- Execute SQL queries from Python scripts.
- Automate ETL and ELT pipelines.
- Validate source and target data.
- Schedule database operations.

For a Data Engineer, database connectivity is an important skill because data pipelines frequently move data between databases, files, APIs, and cloud storage systems.

## 2. How Python Connects to a Database

A typical database interaction follows this sequence:

1. Import a database driver or library.
2. Provide connection settings.
3. Establish a connection.
4. Create a cursor or execute a query through the library.
5. Execute SQL statements.
6. Fetch results or commit changes.
7. Close the cursor and connection.

The exact steps depend on the database library.

For example, Python can connect to PostgreSQL using `psycopg`, MySQL using `mysql-connector-python`, and SQL Server using `pyodbc`.

## 3. Database Drivers and Libraries

A **database driver** allows an application to communicate with a particular database.

| Database | Common Python library |
|---|---|
| PostgreSQL | `psycopg` |
| MySQL | `mysql-connector-python` |
| SQL Server | `pyodbc` |
| SQLite | Built-in `sqlite3` |
| Oracle Database | `python-oracledb` |

Install the library appropriate for your database.

For example:

```bash
pip install "psycopg[binary]"
```

For a production environment, follow your organization's approved dependency and driver installation practices.

## 4. Connecting to SQLite

SQLite is useful for learning because it is built into Python and does not require a separate database server.

### Establish a connection

```python
import sqlite3

connection = sqlite3.connect("company.db")

print("Database connected")
```

If the file does not exist, SQLite normally creates it.

### Create a table and insert data

```python
import sqlite3

connection = sqlite3.connect("company.db")

try:
    cursor = connection.cursor()

    cursor.execute("""
        CREATE TABLE IF NOT EXISTS employees (
            employee_id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            salary REAL
        )
    """)

    cursor.execute(
        "INSERT INTO employees (employee_id, name, salary) VALUES (?, ?, ?)",
        (1, "Yash", 50000)
    )

    connection.commit()

finally:
    connection.close()
```

The `?` placeholders are SQLite parameter markers. The values are passed separately from the SQL statement.

**Important:** Running this insert repeatedly with the same primary key causes a uniqueness error. In a reusable script, handle duplicates or use an appropriate upsert strategy.

### Fetch data

```python
import sqlite3

connection = sqlite3.connect("company.db")

try:
    cursor = connection.cursor()

    cursor.execute("SELECT employee_id, name, salary FROM employees")

    rows = cursor.fetchall()

    for row in rows:
        print(row)

finally:
    connection.close()
```

Example output:

```text
(1, 'Yash', 50000.0)
```

## 5. Understanding Connections and Cursors

### Connection

A connection represents an active session between the Python application and the database.

It is used to:
- Establish communication.
- Manage transactions.
- Create cursors.
- Commit or roll back changes.
- Close the session.

### Cursor

A cursor is an object used by many database drivers to execute SQL statements and retrieve query results.

```python
cursor = connection.cursor()

cursor.execute("SELECT * FROM employees")
rows = cursor.fetchall()
```

A cursor does not itself represent a separate database server connection.

**Interview distinction:** A connection manages the database session, while a cursor is commonly used to execute statements and process their results.

Some libraries provide higher-level APIs that do not require explicit cursor creation.

## 6. Fetching Query Results

Database drivers commonly provide three fetch methods.

### `fetchone()`

Returns the next row, or `None` when no row remains.

```python
cursor.execute("SELECT * FROM employees")

row = cursor.fetchone()
print(row)
```

### `fetchmany()`

Returns up to the requested number of rows.

```python
cursor.execute("SELECT * FROM employees")

rows = cursor.fetchmany(100)
print(rows)
```

This is useful when processing results in batches.

### `fetchall()`

Returns all remaining rows.

```python
cursor.execute("SELECT * FROM employees")

rows = cursor.fetchall()
print(rows)
```

**Data Engineering tip:** Avoid `fetchall()` for extremely large datasets because it may consume substantial memory. Use batching or streaming where supported.

## 7. Connecting to PostgreSQL

PostgreSQL is a widely used relational database.

Install the driver:

```bash
pip install "psycopg[binary]"
```

Example connection:

```python
import os
import psycopg

connection = psycopg.connect(
    host=os.environ["DB_HOST"],
    port=int(os.getenv("DB_PORT", "5432")),
    dbname=os.environ["DB_NAME"],
    user=os.environ["DB_USER"],
    password=os.environ["DB_PASSWORD"],
)

try:
    with connection.cursor() as cursor:
        cursor.execute("SELECT version()")
        print(cursor.fetchone())

finally:
    connection.close()
```

Set the required environment variables before running the script. This avoids hardcoding credentials in the code.

For production deployments, use an approved secrets-management approach and configure TLS according to your database's security requirements.

## 8. Connecting to MySQL

Install the driver:

```bash
pip install mysql-connector-python
```

Example:

```python
import os
import mysql.connector

connection = mysql.connector.connect(
    host=os.environ["DB_HOST"],
    port=int(os.getenv("DB_PORT", "3306")),
    database=os.environ["DB_NAME"],
    user=os.environ["DB_USER"],
    password=os.environ["DB_PASSWORD"],
)

try:
    cursor = connection.cursor()

    cursor.execute("SELECT DATABASE()")
    print(cursor.fetchone())

finally:
    cursor.close()
    connection.close()
```

Connection parameter names and supported options vary across libraries. Follow the documentation for the driver you use.

## 9. Connecting to SQL Server Using `pyodbc`

SQL Server is common in enterprise data environments.

Install the Python library:

```bash
pip install pyodbc
```

You also need a compatible Microsoft ODBC driver installed on the machine running the Python code.

Example:

```python
import os
import pyodbc

connection_string = (
    "DRIVER={ODBC Driver 18 for SQL Server};"
    f"SERVER={os.environ['DB_HOST']};"
    f"DATABASE={os.environ['DB_NAME']};"
    f"UID={os.environ['DB_USER']};"
    f"PWD={os.environ['DB_PASSWORD']};"
    "Encrypt=yes;"
    "TrustServerCertificate=no;"
)

connection = pyodbc.connect(connection_string)

try:
    cursor = connection.cursor()

    cursor.execute("SELECT GETDATE()")
    print(cursor.fetchone())

finally:
    connection.close()
```

The required driver name depends on what is installed. Use a valid certificate and the correct trust settings for your environment; do not disable certificate verification as a general troubleshooting shortcut.

In enterprise environments, integrated authentication or Microsoft Entra authentication may be preferable to username/password authentication, depending on the approved setup.

## 10. Parameterized Queries

**Always use parameterized queries for values supplied dynamically.** Do not construct SQL by directly concatenating user input.

### Unsafe approach

```python
employee_id = input("Enter employee ID: ")

query = (
    "SELECT * FROM employees WHERE employee_id = "
    + employee_id
)
```

String construction can introduce SQL injection vulnerabilities and formatting errors.

### Safer approach with SQLite

```python
employee_id = 1

cursor.execute(
    "SELECT * FROM employees WHERE employee_id = ?",
    (employee_id,)
)

rows = cursor.fetchall()
```

PostgreSQL with `psycopg` uses `%s` placeholders for ordinary values:

```python
cursor.execute(
    "SELECT * FROM employees WHERE employee_id = %s",
    (employee_id,)
)
```

The placeholder syntax is **driver-specific**. Use the syntax supported by your library.

Parameters represent values, not arbitrary SQL identifiers such as table names or column names. If identifiers must vary dynamically, use an appropriate identifier-composition API or a strict allowlist.

## 11. Transactions, Commit, and Rollback

A transaction groups database operations into a unit of work.

### `commit()`

Makes pending transactional changes durable according to the database's transaction semantics.

### `rollback()`

Reverts pending changes in the current transaction.

Example:

```python
import sqlite3

connection = sqlite3.connect("company.db")

try:
    cursor = connection.cursor()

    cursor.execute(
        "UPDATE employees SET salary = ? WHERE employee_id = ?",
        (55000, 1)
    )

    connection.commit()

except Exception:
    connection.rollback()
    raise

finally:
    connection.close()
```

If an error occurs before the commit, the script attempts to roll back the transaction and re-raises the exception.

**Important:** Transaction behavior differs by driver and database. Some libraries use autocommit settings, and some operations may have special transaction behavior.

## 12. Use Context Managers for Safer Resource Handling

Context managers help ensure that resources are handled correctly, even when an exception occurs.

For SQLite:

```python
import sqlite3

with sqlite3.connect("company.db") as connection:
    cursor = connection.cursor()

    cursor.execute(
        "SELECT employee_id, name FROM employees"
    )

    rows = cursor.fetchall()
    print(rows)
```

For SQLite, the connection context manager handles transaction commit or rollback; it does **not** automatically close the connection. If explicit closure is required, use `contextlib.closing()` or close the connection in a `finally` block.

For libraries with different connection and cursor context-manager behavior, consult the driver's documentation.

## 13. SQLAlchemy: Higher-Level Database Connectivity

SQLAlchemy provides a higher-level SQL toolkit and optional ORM. It supports multiple relational databases through appropriate drivers.

Install it:

```bash
pip install sqlalchemy
```

Example with SQLite:

```python
from sqlalchemy import create_engine, text

engine = create_engine("sqlite:///company.db")

with engine.connect() as connection:
    result = connection.execute(
        text("SELECT employee_id, name FROM employees")
    )

    for row in result:
        print(row)
```

For a transaction that writes data:

```python
from sqlalchemy import create_engine, text

engine = create_engine("sqlite:///company.db")

with engine.begin() as connection:
    connection.execute(
        text(
            "UPDATE employees "
            "SET salary = :salary "
            "WHERE employee_id = :employee_id"
        ),
        {"salary": 60000, "employee_id": 1},
    )
```

The `engine.begin()` context manager manages the transaction: it commits on successful completion and rolls back if an exception occurs.

SQLAlchemy engines can also manage connection pooling. Pooling reduces the overhead of repeatedly creating database connections, particularly in applications with frequent database operations.

## 14. Loading Query Results into Pandas

Pandas is useful for small-to-medium extracts that fit comfortably in memory.

```python
import sqlite3
import pandas as pd

connection = sqlite3.connect("company.db")

try:
    df = pd.read_sql_query(
        "SELECT employee_id, name, salary FROM employees",
        connection
    )

    print(df.head())

finally:
    connection.close()
```

For a large result, use chunks:

```python
import sqlite3
import pandas as pd

connection = sqlite3.connect("company.db")

try:
    chunks = pd.read_sql_query(
        "SELECT employee_id, name, salary FROM employees",
        connection,
        chunksize=10000
    )

    for chunk in chunks:
        # Transform or write each chunk
        print(len(chunk))

finally:
    connection.close()
```

Chunking reduces the amount of data held in memory at once, although total memory usage depends on what the pipeline does with each chunk.

## 15. Database Connectivity in Data Engineering Pipelines

A typical database-to-database ETL process follows this pattern:

1. Read connection settings securely.
2. Connect to the source database.
3. Execute an extraction query.
4. Read the results in manageable batches.
5. Transform and validate the data.
6. Write to the destination database.
7. Commit changes at appropriate transaction boundaries.
8. Record metrics and errors.
9. Close connections and release resources.

Example conceptual flow:

```text
Source Database
       |
       v
Python Database Driver
       |
       v
Extract Data in Batches
       |
       v
Transform and Validate
       |
       v
Destination Database
       |
       v
Commit and Record Metrics
```

In large-scale Data Engineering systems, use a tool appropriate to the volume and architecture. Spark JDBC, managed data integration services, bulk-loading utilities, or database-native replication may be more suitable than processing every row through a Python cursor.

## 16. Common Connectivity Errors

| Error or symptom | Possible cause | What to check |
|---|---|---|
| Connection refused | Server unavailable or wrong port | Host, port, server status |
| Authentication failed | Incorrect credentials or authentication mode | User, password, permissions, authentication configuration |
| Database not found | Incorrect database name | Database name and server |
| Timeout | Network issue, firewall, or overloaded server | Connectivity, firewall rules, timeout settings |
| Driver not found | Missing or incompatible driver | Installed driver and architecture |
| SSL or certificate error | TLS configuration or certificate trust problem | Certificate chain, server name, TLS settings |
| Permission denied | Insufficient database privileges | Grants and access policies |
| Too many connections | Connections are leaking or pool size is too high | Resource cleanup and pooling configuration |
| Query timeout | Slow query or insufficient timeout | Query plan, indexes, workload, timeout settings |
| Memory exhaustion | Fetching too many rows at once | Batch fetching, streaming, query filters |

Avoid automatically retrying every error. Retry transient failures with bounded retries and backoff; correct permanent problems such as invalid credentials or missing permissions instead.

## 17. Security and Performance Best Practices

### Security

- Use environment variables or a secret manager for credentials.
- Use parameterized SQL statements.
- Grant only the permissions the application needs.
- Use encrypted connections where required.
- Avoid logging credentials and sensitive query parameters.
- Keep database drivers and dependencies updated.

### Performance

- Select only the columns required.
- Filter data at the database when appropriate.
- Fetch large results in batches.
- Use connection pooling for repeated connections.
- Use bulk insert or batch operations when supported.
- Inspect query plans for slow queries.
- Avoid opening a new connection for every row.
- Use incremental extraction for recurring data pipelines when the source supports it.

## 18. Interview Questions

### Basic

1. What is database connectivity?
2. What is the difference between a connection and a cursor?
3. What is a database driver?
4. What is the difference between `fetchone()`, `fetchmany()`, and `fetchall()`?
5. What do `commit()` and `rollback()` do?
6. Why should database connections be closed?
7. What is a parameterized query?
8. What is SQL injection?

### Intermediate

1. How would you connect Python to PostgreSQL?
2. How would you read millions of rows without loading all of them into memory?
3. What is connection pooling?
4. How would you handle database connection failures in an ETL pipeline?
5. What is the difference between a database driver and SQLAlchemy?
6. Why might a database connection succeed while a query fails?
7. How would you manage database credentials securely?
8. How would you troubleshoot a connection timeout?
9. What is the purpose of a transaction?
10. How would you avoid inserting duplicate records when rerunning a pipeline?

### Data Engineering scenarios

1. A pipeline extracts ten million rows. How would you design extraction to control memory consumption?
2. A job fails after inserting some rows. How would you prevent inconsistent data?
3. A pipeline opens hundreds of database connections. What would you investigate?
4. A source database query is slow. What would you check before increasing the timeout?
5. How would you load transformed records into a target database efficiently?
6. When would you choose Spark JDBC or a managed ingestion service instead of a simple Python driver?

## 19. Quick Revision

| Concept | Remember |
|---|---|
| Connection | Establishes a database session |
| Cursor | Executes SQL and processes results in cursor-based drivers |
| `execute()` | Runs a SQL statement |
| `fetchone()` | Retrieves one row |
| `fetchmany(n)` | Retrieves up to `n` rows |
| `fetchall()` | Retrieves all remaining rows |
| `commit()` | Commits pending transactional changes |
| `rollback()` | Reverts pending transactional changes |
| Parameterized query | Separates SQL structure from values |
| Connection pooling | Reuses database connections |
| SQLAlchemy | Higher-level SQL toolkit and optional ORM |
| Pandas chunks | Process query results in smaller batches |
| Secret manager | Protects credentials and other sensitive values |

**Key takeaway:** Learn how to connect, execute parameterized queries, fetch data in batches, manage transactions, handle errors, and protect credentials. These are the core database-connectivity skills required to build reliable Python-based data pipelines.