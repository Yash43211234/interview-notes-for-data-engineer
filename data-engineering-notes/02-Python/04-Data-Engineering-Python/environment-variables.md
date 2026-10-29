# Environment Variables in Python

## 1. What Are Environment Variables?

**Environment variables** are key-value pairs provided to a running process by its operating system or execution environment.

They allow an application to access configuration values without hardcoding them into the source code.

Examples include:

- Database hostnames and usernames
- API endpoints
- Application environment (`development`, `testing`, `production`)
- Logging levels
- Batch sizes and retry counts
- File paths and storage locations
- References to secrets or credentials

Example:

```text
APP_ENV=production
BATCH_SIZE=5000
DB_HOST=database.example.com
LOG_LEVEL=INFO
```

In Python, the built-in `os` module provides access to environment variables.

```python
import os

environment = os.getenv("APP_ENV", "development")
print(environment)
```

If `APP_ENV` is set to `production`, the output is `production`. Otherwise, the default value is `development`.

## 2. Why Are Environment Variables Important?

Environment variables help applications run in different environments without changing the source code.

For example, the same ETL pipeline might use different database servers:

| Setting | Development | Production |
|---|---|---|
| `APP_ENV` | `development` | `production` |
| `DB_HOST` | Test database | Production database |
| `BATCH_SIZE` | `100` | `5000` |
| `LOG_LEVEL` | `DEBUG` | `INFO` |

The values above are illustrative; actual batch sizes and logging settings depend on the workload.

**Main benefits:**

1. Separate configuration from business logic.
2. Support different deployment environments.
3. Avoid committing credentials directly to source code.
4. Make scripts easier to deploy and reuse.
5. Allow configuration to be changed without editing Python files.

Important: Environment variables are not automatically secret or encrypted. Use a secrets manager and appropriate access controls for sensitive values.

## 3. Accessing Environment Variables in Python

Python provides two commonly used approaches.

### 3.1 `os.getenv()`

Returns the value of an environment variable, or `None` if the variable is missing and no default is provided.

```python
import os

value = os.getenv("APP_ENV")
print(value)
```

Provide a default value when appropriate:

```python
environment = os.getenv("APP_ENV", "development")
print(environment)
```

If the variable is missing, the output is `development`.

Use `getenv()` when a setting is optional or has a sensible default.

### 3.2 `os.environ`

`os.environ` behaves like a mapping of the current process's environment variables.

```python
import os

environment = os.environ["APP_ENV"]
print(environment)
```

If `APP_ENV` is missing, Python raises a `KeyError`.

Use this approach when a setting is mandatory and the application should fail clearly if it is missing.

You can also check whether a variable exists:

```python
if "APP_ENV" in os.environ:
    print("APP_ENV is configured")
else:
    print("APP_ENV is missing")
```

### Difference between `os.getenv()` and `os.environ[]`

| Feature | `os.getenv()` | `os.environ[]` |
|---|---|---|
| Reads environment variables | Yes | Yes |
| Missing variable, no default | Returns `None` | Raises `KeyError` |
| Supports a default value | Yes | Not directly |
| Best use | Optional settings | Required settings |

## 4. Environment Variables Are Strings

Environment variable values are read as strings, even when they represent numbers or Boolean values.

Example:

```python
import os

os.environ["BATCH_SIZE"] = "1000"

batch_size = os.getenv("BATCH_SIZE")

print(batch_size)        # '1000'
print(type(batch_size))  # <class 'str'>
```

Convert values to the required type before using them.

### Integer

```python
batch_size = int(os.getenv("BATCH_SIZE", "1000"))
```

### Float

```python
timeout = float(os.getenv("TIMEOUT", "30.5"))
```

### Boolean

Avoid this common mistake:

```python
debug = bool(os.getenv("DEBUG", "false"))
print(debug)  # True
```

The result is `True` because the string `"false"` is non-empty.

Instead, parse the accepted values explicitly:

```python
import os

def get_bool_env(name: str, default: bool = False) -> bool:
    value = os.getenv(name)

    if value is None:
        return default

    value = value.strip().lower()

    if value in {"true", "1", "yes"}:
        return True

    if value in {"false", "0", "no"}:
        return False

    raise ValueError(f"Invalid Boolean value for {name}")

debug = get_bool_env("DEBUG")
```

This makes invalid settings easier to detect.

## 5. Setting Environment Variables

The method depends on the operating system, shell, or deployment platform.

### 5.1 Set a variable in Python

```python
import os

os.environ["APP_ENV"] = "testing"

print(os.getenv("APP_ENV"))
```

This updates the environment mapping for the current Python process and its subsequently created child processes. It does not permanently configure the operating system or automatically update the parent shell.

### 5.2 Windows Command Prompt

Set a variable for the current Command Prompt session:

```bat
set APP_ENV=development
python app.py
```

### 5.3 Windows PowerShell

```powershell
$env:APP_ENV = "development"
python app.py
```

### 5.4 Linux or macOS terminal

```bash
export APP_ENV=development
python app.py
```

These shell assignments normally apply to the current shell session and processes launched from it. Persistent configuration requires the appropriate operating-system or deployment configuration.

## 6. Using a `.env` File

A `.env` file is a convenient way to store local development configuration as key-value pairs.

Example `.env`:

```text
APP_ENV=development
BATCH_SIZE=1000
LOG_LEVEL=DEBUG
```

Python's `os.getenv()` does **not** automatically load a `.env` file. You need a library such as `python-dotenv`, or another configuration mechanism.

Install the library:

```bash
pip install python-dotenv
```

Load the file:

```python
import os
from dotenv import load_dotenv

load_dotenv()

environment = os.getenv("APP_ENV", "development")
batch_size = int(os.getenv("BATCH_SIZE", "1000"))

print(environment)
print(batch_size)
```

By default, `load_dotenv()` looks for a `.env` file using its discovery rules.

### Important `.env` practices

- Add local `.env` files containing secrets to `.gitignore`.
- Commit a `.env.example` file containing placeholder values if useful.
- Do not put real passwords or tokens in public repositories.
- Do not assume `.env` files are encrypted.
- In production, use the approved deployment configuration or secrets-management system.

Example `.gitignore` entry:

```text
.env
```

A `.env` file is a development convenience, not a replacement for a production secrets manager.

## 7. Required and Optional Configuration

Choose defaults carefully.

### Optional configuration

```python
import os

log_level = os.getenv("LOG_LEVEL", "INFO")
```

A default is reasonable because the application can function with standard logging.

### Required configuration

```python
import os

database_host = os.environ["DB_HOST"]
```

If the database host is missing, the application fails immediately rather than silently connecting to an unintended default.

For a clearer error message:

```python
import os

database_host = os.getenv("DB_HOST")

if not database_host:
    raise ValueError("Required environment variable DB_HOST is missing")
```

This check also rejects an empty string.

**Best practice:** Validate mandatory configuration at application startup, before running expensive processing or modifying data.

## 8. Environment Variables in a Data Engineering Pipeline

Consider a pipeline that extracts orders from a database and writes processed data to storage.

```python
import os

app_env = os.getenv("APP_ENV", "development")
batch_size = int(os.getenv("BATCH_SIZE", "1000"))
source_table = os.getenv("SOURCE_TABLE", "orders")
output_path = os.environ["OUTPUT_PATH"]

if batch_size <= 0:
    raise ValueError("BATCH_SIZE must be greater than zero")

print(f"Environment: {app_env}")
print(f"Batch size: {batch_size}")
print(f"Source table: {source_table}")
print(f"Output path: {output_path}")
```

This example separates runtime settings from the pipeline logic.

The source table and output path can change between environments without editing the transformation code.

For a real pipeline, validate table identifiers, storage paths, permissions, and other relevant settings before processing begins.

## 9. Using Environment Variables for Database Connections

Never hardcode database passwords in source code.

```python
import os
import sqlite3

database_path = os.getenv("DB_PATH", "company.db")

connection = sqlite3.connect(database_path)

try:
    with connection:
        cursor = connection.cursor()
        cursor.execute("SELECT 1")
        print(cursor.fetchone())
finally:
    connection.close()
```

For a server-based database, credentials can be supplied through the environment:

```python
import os

db_host = os.environ["DB_HOST"]
db_name = os.environ["DB_NAME"]
db_user = os.environ["DB_USER"]
db_password = os.environ["DB_PASSWORD"]
```

These values can then be passed to the appropriate database driver.

In production, prefer managed identity or workload identity where supported. Otherwise, obtain credentials from an approved secrets manager and apply least-privilege access.

Do not print `db_password` or include it in logs and error messages.

## 10. Environment Variables in Cloud Data Engineering

Environment variables are commonly used in local Python applications, containers, virtual machines, and job execution environments.

In cloud data platforms, configuration may also be supplied through platform-specific mechanisms.

| Platform or tool | Typical configuration mechanism |
|---|---|
| Azure Data Factory | Pipeline parameters, linked-service settings, secure parameters |
| Azure Databricks | Job parameters, environment variables where supported, secret scopes or approved secret integrations |
| Azure Key Vault | Secure retrieval of secrets |
| Docker | Environment settings supplied when running containers |
| GitHub Actions | Repository or environment variables and secrets |
| Linux services | Service or process environment configuration |

These mechanisms serve related but different purposes. For example, a pipeline parameter can represent a processing date, while a secret manager stores a database password.

Do not assume that setting an environment variable on your laptop automatically makes it available to a cloud job. Configure the variable in the environment where the job actually runs.

## 11. Environment Variables and Configuration Precedence

Applications may obtain settings from multiple sources:

- Built-in defaults
- Configuration files
- Environment variables
- Command-line arguments
- Secret managers

A common design is to let higher-priority sources override lower-priority defaults. The precise precedence must be defined by the application.

Example:

```python
import os

default_batch_size = 1000
batch_size = int(os.getenv("BATCH_SIZE", str(default_batch_size)))
```

Here, `BATCH_SIZE` overrides the default when supplied.

For a larger application, document precedence clearly and validate the final resolved configuration. This prevents unexpected behavior when different sources provide conflicting values.

## 12. Common Mistakes

1. **Treating all values as the correct type.** Convert strings to integers, floats, or Booleans as needed.
2. **Using `bool()` on a string.** The string `"false"` evaluates to `True`.
3. **Assuming `.env` files load automatically.** Load them explicitly or use a deployment mechanism.
4. **Hardcoding secrets.** Keep credentials out of source code and Git.
5. **Using defaults for mandatory production settings.** This can hide missing configuration.
6. **Changing `os.environ` and expecting permanent changes.** Python process-level changes do not automatically persist in the parent shell.
7. **Ignoring missing variables.** Validate required settings before processing.
8. **Logging sensitive configuration.** Never expose passwords, tokens, or private keys.
9. **Assuming local settings apply in the cloud.** Configure each execution environment separately.
10. **Using unvalidated values in SQL.** Environment variables are still input; use parameterized queries for SQL values and safe methods for dynamic identifiers.

## 13. Interview Questions

### Basic

1. What are environment variables?
2. Why are environment variables useful in Python?
3. What is the difference between `os.getenv()` and `os.environ[]`?
4. What happens when a required environment variable is missing?
5. Why are environment variable values treated as strings?
6. How do you set an environment variable in Windows and Linux?
7. What is a `.env` file?
8. Does Python automatically load `.env` files?

### Intermediate

1. How would you parse a Boolean environment variable safely?
2. How would you validate configuration before running a pipeline?
3. What is the difference between configuration and secrets?
4. Why should production credentials not be hardcoded?
5. How would you configure the same ETL pipeline for development and production?
6. What happens when a Python process modifies `os.environ`?
7. How can configuration values override defaults?
8. Why might a locally working pipeline fail when deployed to a cloud platform?

### Data Engineering scenarios

1. Your pipeline cannot find `DB_HOST` in production. How would you troubleshoot it?
2. A pipeline unexpectedly uses a batch size of `1000` instead of `5000`. What would you check?
3. A database password was accidentally committed to Git. What immediate steps would you take?
4. How would you pass different storage paths to the same pipeline in testing and production?
5. Where should a database password be stored in an Azure-based workflow?
6. How would you prevent invalid configuration from causing a partial data load?

## 14. Quick Revision

| Concept | Key point |
|---|---|
| Environment variable | A key-value setting supplied to a process |
| `os.getenv("KEY")` | Returns the value or `None` if missing |
| `os.getenv("KEY", "default")` | Returns a default when the variable is missing |
| `os.environ["KEY"]` | Raises `KeyError` if the key is missing |
| Type conversion | Convert strings into the required data type |
| `.env` file | Local configuration file that needs to be loaded |
| `python-dotenv` | Library that can load `.env` files |
| Secrets manager | Secure mechanism for managing sensitive values |
| Configuration validation | Checks settings before processing starts |
| Cloud deployment | Configure variables in the actual job execution environment |

**Key takeaway:** Environment variables help make Python applications reusable across environments. For Data Engineering, learn to read and validate settings, convert their types, keep secrets secure, and configure variables in the environment where each pipeline actually runs.