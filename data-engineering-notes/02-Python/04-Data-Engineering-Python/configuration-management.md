# Configuration Management in Python

## 1. What Is Configuration Management?

**Configuration management** is the practice of storing and managing application settings separately from the application code.

Configuration values control how an application behaves without requiring changes to its core logic.

Examples of configuration:
- Database connection settings
- API endpoints
- File paths
- Logging levels
- Retry limits
- Environment names such as development, testing, and production
- Cloud storage locations

Instead of hardcoding these values, applications load them from configuration sources such as environment variables, JSON files, YAML files, or dedicated configuration systems.

### Example: Hardcoded configuration

```python
database_host = "prod-db.example.com"
database_port = 5432
```

Problems:
- Changing environments requires code changes.
- Sensitive information might accidentally be committed to Git.
- The same application becomes harder to deploy across environments.

### Better approach

```python
import os

database_host = os.getenv("DB_HOST", "localhost")
database_port = int(os.getenv("DB_PORT", "5432"))
```

Now, the configuration can change without modifying the Python code.

## 2. Why Configuration Management Matters

| Benefit | Explanation |
|---|---|
| Environment separation | Use different settings for development and production |
| Security | Keep passwords and API keys out of source code |
| Maintainability | Change settings without editing business logic |
| Reusability | Run the same application in different environments |
| Deployment flexibility | Configure jobs through environment-specific settings |
| Testing | Supply test settings without affecting production |

**Data Engineering example:** The same ETL pipeline may read from a development storage account during testing and a production storage account after deployment. Configuration lets the pipeline reuse the same processing logic.

## 3. Common Configuration Sources

### 3.1 Environment variables

Environment variables are values provided to a process by its execution environment.

Python can access them using the `os` module.

```python
import os

environment = os.getenv("APP_ENV", "development")
batch_size = int(os.getenv("BATCH_SIZE", "1000"))

print(environment)
print(batch_size)
```

If `APP_ENV` is not set, the application uses `"development"`.

If `BATCH_SIZE` is not set, it uses `1000`.

**Important:** Environment variables are strings. Convert them to the required type when necessary.

```python
import os

debug_mode = os.getenv("DEBUG", "false").lower() == "true"
retry_count = int(os.getenv("RETRY_COUNT", "3"))
```

For Boolean values, define accepted values explicitly when you need stricter validation.

```python
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
```

This avoids treating every non-empty string as `True`.

### 3.2 JSON configuration files

JSON is useful for simple, structured settings.

Example `config.json`:

```json
{
  "environment": "development",
  "batch_size": 1000,
  "retry_count": 3,
  "input_path": "data/input",
  "output_path": "data/output"
}
```

Load the configuration:

```python
import json

with open("config.json", "r", encoding="utf-8") as file:
    config = json.load(file)

print(config["batch_size"])
print(config["input_path"])
```

**Advantages:**
- Easy to read and widely supported.
- Supports nested objects, lists, strings, numbers, and Boolean values.

**Limitation:** JSON does not support comments, and it does not automatically validate that required settings are present or have the correct types.

### 3.3 YAML configuration files

YAML is often used for more readable configuration files, including application settings and deployment definitions.

Example `config.yaml`:

```yaml
environment: development
batch_size: 1000
retry_count: 3

storage:
  input_path: data/input
  output_path: data/output
```

Python can load it using PyYAML:

```python
import yaml

with open("config.yaml", "r", encoding="utf-8") as file:
    config = yaml.safe_load(file)

print(config["batch_size"])
print(config["storage"]["input_path"])
```

Install the dependency if needed:

```bash
pip install PyYAML
```

Use `yaml.safe_load()` for ordinary configuration loading rather than `yaml.load()` with an unsafe loader.

### 3.4 Command-line arguments

Command-line arguments are useful when users need to change settings each time they run a script.

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument("--environment", default="development")
parser.add_argument("--batch-size", type=int, default=1000)

args = parser.parse_args()

print(args.environment)
print(args.batch_size)
```

Run the script:

```bash
python pipeline.py --environment production --batch-size 5000
```

Use command-line arguments for runtime choices such as batch size, date range, or execution mode.

## 4. Choosing the Right Configuration Source

| Source | Best use |
|---|---|
| Environment variables | Deployment-specific settings and injected secrets |
| JSON | Simple structured configuration |
| YAML | Human-readable nested configuration |
| Command-line arguments | Per-run overrides |
| Secret manager | Credentials, API keys, and other sensitive values |
| Central configuration service | Settings shared across multiple applications or services |

These sources can be combined. For example, a pipeline might load ordinary defaults from a YAML file, override selected settings with environment variables, and retrieve credentials from a secret manager.

## 5. Separate Configuration from Business Logic

Consider an ETL pipeline.

### Poor approach

```python
def process_orders():
    input_path = "data/orders.csv"
    output_path = "data/processed_orders"
    batch_size = 1000

    # Read, transform, and write data
```

The settings are tied to the function.

### Better approach

```python
def process_orders(input_path, output_path, batch_size):
    # Read data from input_path
    # Transform data
    # Write data to output_path
    pass
```

The caller supplies the configuration:

```python
process_orders(
    input_path="data/orders.csv",
    output_path="data/processed_orders",
    batch_size=1000,
)
```

For larger applications, use a dedicated configuration object instead of passing many individual arguments.

## 6. Using a Configuration Dataclass

Python's `dataclasses` module provides a convenient way to represent related settings.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class PipelineConfig:
    environment: str
    batch_size: int
    retry_count: int
    input_path: str
    output_path: str
```

Create the configuration:

```python
config = PipelineConfig(
    environment="development",
    batch_size=1000,
    retry_count=3,
    input_path="data/input",
    output_path="data/output",
)

print(config.batch_size)
```

The `frozen=True` option prevents normal reassignment of fields after creation.

```python
# Raises dataclasses.FrozenInstanceError
config.batch_size = 5000
```

A dataclass organizes settings but does not automatically validate all their business rules. Add explicit validation or use a validation library when necessary.

## 7. Environment-Specific Configuration

Applications often need different settings for different environments.

| Setting | Development | Production |
|---|---|---|
| Database | Local or test database | Production database |
| Logging | DEBUG or INFO | INFO or WARNING |
| Data paths | Test locations | Production storage |
| Batch size | Smaller test batches | Workload-appropriate batches |
| Credentials | Test credentials | Managed production credentials |

One simple approach is to select settings using an environment variable.

```python
import os

environment = os.getenv("APP_ENV", "development").lower()

if environment == "development":
    batch_size = 100
elif environment == "production":
    batch_size = 5000
else:
    raise ValueError(f"Unsupported environment: {environment}")
```

For larger projects, keep environment-specific settings in separate configuration files or use a structured configuration system. Validate configuration at application startup so errors are discovered before processing begins.

## 8. Configuration Validation

Invalid configuration can cause pipeline failures, incorrect results, or unnecessary resource consumption.

Validate settings before using them.

```python
def validate_config(batch_size: int, retry_count: int) -> None:
    if batch_size <= 0:
        raise ValueError("batch_size must be greater than zero")

    if retry_count < 0:
        raise ValueError("retry_count cannot be negative")
```

Usage:

```python
batch_size = 1000
retry_count = 3

validate_config(batch_size, retry_count)
```

Other checks might include:
- Required values are present.
- Numeric values are within reasonable limits.
- Environment names are recognized.
- File paths and storage locations are valid.
- Required credentials are available.
- Conflicting settings are rejected.

Validate configuration as early as possible rather than discovering problems halfway through an ETL job.

## 9. Managing Secrets Securely

**Never commit passwords, API keys, access tokens, or private keys to source control.**

Bad example:

```python
# Do not do this
database_password = "MyRealPassword123"
```

A safer approach is to obtain the secret from an environment variable or a dedicated secret manager.

```python
import os

database_password = os.environ["DB_PASSWORD"]
```

Using `os.environ["DB_PASSWORD"]` raises `KeyError` if the variable is missing. This can be useful when the secret is mandatory.

For cloud workloads, prefer managed identity or workload identity where supported, together with an approved secret manager such as Azure Key Vault.

Additional good practices:
- Do not print secrets in logs.
- Restrict access using least privilege.
- Rotate credentials according to organizational policy.
- Avoid storing secrets in ordinary configuration files.
- Do not assume environment variables are automatically encrypted or secret.

## 10. A Practical ETL Configuration Example

Suppose a data pipeline reads orders, applies transformations, and writes results.

Example `config.json`:

```json
{
  "environment": "development",
  "batch_size": 1000,
  "input_path": "data/orders.csv",
  "output_path": "data/processed_orders"
}
```

Python implementation:

```python
import json
from dataclasses import dataclass
from pathlib import Path


@dataclass(frozen=True)
class PipelineConfig:
    environment: str
    batch_size: int
    input_path: str
    output_path: str


def load_config(path: str) -> PipelineConfig:
    with open(path, "r", encoding="utf-8") as file:
        data = json.load(file)

    config = PipelineConfig(
        environment=data["environment"],
        batch_size=int(data["batch_size"]),
        input_path=data["input_path"],
        output_path=data["output_path"],
    )

    if config.batch_size <= 0:
        raise ValueError("batch_size must be positive")

    if config.environment not in {"development", "testing", "production"}:
        raise ValueError("Unsupported environment")

    return config


config = load_config("config.json")

if not Path(config.input_path).exists():
    raise FileNotFoundError(f"Input file not found: {config.input_path}")

print(f"Environment: {config.environment}")
print(f"Batch size: {config.batch_size}")
print(f"Input: {config.input_path}")
print(f"Output: {config.output_path}")
```

This example demonstrates separation, loading, type conversion, and basic validation. In a real cloud pipeline, input paths may be URIs rather than local filesystem paths, so existence checks should use the appropriate storage API.

## 11. Configuration Management in Data Engineering

Configuration is especially useful for reusable data pipelines.

A typical pipeline may need:
- Source and destination locations.
- Table names and schema names.
- Processing dates.
- Batch sizes and partition settings.
- Retry counts and timeout limits.
- Environment names.
- Logging and monitoring settings.
- Feature flags controlling optional processing steps.

For example, a pipeline can use the same transformation code while receiving different input and output paths for testing and production.

In Azure-based workflows, configuration can be supplied through Azure Data Factory pipeline parameters, Databricks job parameters, environment variables, managed secret stores, and deployment settings. The correct choice depends on whether a value is a per-run parameter, an environment setting, or a sensitive credential.

**Important distinction:** Configuration management defines how settings are supplied, validated, and maintained. It is not the same as data transformation, orchestration, or secret storage, although it works alongside all three.

## 12. Common Mistakes

1. **Hardcoding environment-specific values.** This makes deployments harder to maintain.
2. **Committing secrets to Git.** Keep credentials out of source code and ordinary configuration files.
3. **Ignoring data types.** Environment variables are strings; convert and validate them.
4. **Using defaults for mandatory settings.** A missing production database address should normally cause a clear startup error, not silently fall back to a local database.
5. **Skipping validation.** Invalid values can fail jobs or produce incorrect behavior.
6. **Mixing configuration with processing logic.** Keep the settings separate from transformations.
7. **Logging sensitive settings.** Log safe configuration metadata, not passwords or tokens.
8. **Overcomplicating small applications.** Start with environment variables or a simple file; add a configuration framework only when it solves a real problem.
9. **Assuming configuration changes automatically update running jobs.** Many applications load configuration at startup, so a restart or new job run may be required.

## 13. Interview Questions

### Basic

1. What is configuration management?
2. Why should configuration be separated from application code?
3. What are environment variables?
4. How do you read environment variables in Python?
5. What is the difference between JSON and YAML configuration?
6. Why should secrets not be stored in source code?
7. What is the purpose of `os.getenv()`?
8. Why is configuration validation important?

### Intermediate

1. How would you use the same ETL pipeline in development and production?
2. How would you override a default batch size using an environment variable?
3. What happens if a required environment variable is missing?
4. How would you organize a large set of pipeline settings?
5. How would you handle configuration for multiple cloud environments?
6. What is the difference between configuration values and secrets?
7. How would you test code that depends on environment variables?
8. How can invalid configuration be prevented from reaching production?

### Data Engineering scenarios

1. A pipeline works in development but fails in production because a storage path is incorrect. How would you investigate it?
2. A database password has been committed to Git. What should you do?
3. A pipeline processes too much data because the batch size is incorrectly configured. How would you prevent this?
4. How would you configure different source and destination tables without duplicating transformation code?
5. Which settings belong in orchestration parameters, and which belong in a secret manager?

## 14. Quick Revision

| Concept | Key point |
|---|---|
| Configuration management | Manage application settings separately from code |
| `os.getenv()` | Read an environment variable with an optional default |
| `os.environ[]` | Read a required environment variable; raises `KeyError` if missing |
| JSON | Simple structured configuration |
| YAML | Readable configuration with nested structures |
| `argparse` | Parse command-line settings |
| `dataclass` | Organize related configuration values |
| Validation | Check required values, types, and allowed ranges |
| Secret manager | Store and retrieve sensitive credentials |
| Environment-specific settings | Support development, testing, and production |
| ETL benefit | Reuse pipeline logic across environments |

**Key takeaway:** A well-designed data pipeline keeps business logic separate from configuration, validates settings before processing, and manages secrets securely. This makes the pipeline easier to test, deploy, and maintain across environments.