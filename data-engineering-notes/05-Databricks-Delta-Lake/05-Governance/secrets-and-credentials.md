# Secrets and Credentials in Databricks

## 1. What Are Secrets and Credentials?

In data engineering, pipelines often need to connect to external systems such as:

- Azure Data Lake Storage (ADLS Gen2)
- Azure SQL Database
- AWS S3
- REST APIs
- Databases
- Message brokers

These systems may require sensitive authentication information, such as:

- Passwords
- API keys
- Access tokens
- Client secrets
- Connection strings
- Service account credentials

A **secret** is sensitive information that should not be exposed in source code, notebooks, logs, or configuration files.

**Credentials** are the authentication information or mechanisms used to prove identity and obtain access to a system.

### Simple Example

Avoid hardcoding credentials:

```python
password = "MyDatabasePassword123"
```

Instead, retrieve the secret securely:

```python
password = dbutils.secrets.get(
    scope="database_scope",
    key="db_password"
)
```

**Remember:**

Secrets protect sensitive values. Credentials are used to authenticate to systems.

## 2. Why Do We Need Secret Management?

Suppose a Databricks pipeline connects to an Azure SQL Database.

The pipeline needs:

- Server name
- Database name
- Username
- Password

If the password is written directly in a notebook, anyone with access to that notebook may discover it.

This creates security risks.

### Main Benefits

1. **Security:** Prevent credentials from being exposed in code.
2. **Centralized management:** Store sensitive values separately from notebooks.
3. **Access control:** Restrict which users and workloads can retrieve secrets.
4. **Maintainability:** Update credentials without changing application code.
5. **Reduced exposure:** Avoid placing passwords and tokens in source files.
6. **Automation:** Allow jobs and applications to retrieve required secrets.

## 3. Databricks Secret Management

Databricks provides a secret management interface through **secret scopes**.

A secret scope is a collection of secrets.

Example:

```text
Secret Scope: database_scope

Secrets:
    db_username
    db_password
    db_server
    api_key
```

A notebook can retrieve a secret using its scope and key.

### Secret Scope Structure

```text
          SECRET SCOPE
               |
      +--------+--------+
      |        |        |
      v        v        v
   Username Password  API Key
```

Two commonly discussed scope types are:

1. Databricks-backed secret scopes.
2. Azure Key Vault-backed secret scopes.

Availability and behavior depend on the Databricks workspace and cloud configuration.

## 4. Databricks-Backed Secret Scope

A Databricks-backed secret scope stores secrets in Databricks-managed secret storage.

Example:

```text
Scope Name: database_scope

Secret Keys:
    db_username
    db_password
```

### Creating a Secret Scope

A secret scope can be created through supported Databricks CLI commands or APIs.

Example using the Databricks CLI:

```bash
databricks secrets create-scope database_scope
```

The exact CLI syntax and authentication configuration depend on the installed CLI version.

### Adding a Secret

A secret can be added using the CLI:

```bash
databricks secrets put-secret database_scope db_password
```

The command may prompt for the secret value or support input through the configured CLI interface.

Avoid putting actual passwords directly into shell commands because command history, process information, or logs may expose them.

### Listing Secret Scopes

```bash
databricks secrets list-scopes
```

This lists available scopes, subject to the caller's permissions.

### Listing Secret Keys

```bash
databricks secrets list-secrets database_scope
```

This lists secret metadata such as key names, not the actual secret values.

## 5. Azure Key Vault-Backed Secret Scope

In Azure environments, Databricks can integrate with **Azure Key Vault**.

Azure Key Vault is a cloud service used to securely store and manage:

- Secrets
- Encryption keys
- Certificates

Instead of managing sensitive values only inside Databricks, an organization can manage them in Azure Key Vault.

### Architecture

```text
       AZURE KEY VAULT
              |
              | Stores secrets
              |
              v
      DATABRICKS SECRET SCOPE
              |
              | Retrieves secret
              |
              v
        DATABRICKS NOTEBOOK
              |
              v
       EXTERNAL DATABASE
```

### Example

Suppose Azure Key Vault contains:

```text
db-username
db-password
```

Databricks can access those values through a configured Key Vault-backed secret scope.

The notebook retrieves the secret using:

```python
username = dbutils.secrets.get(
    scope="azure-kv-scope",
    key="db-username"
)

password = dbutils.secrets.get(
    scope="azure-kv-scope",
    key="db-password"
)
```

**Important:** The secret scope must be configured correctly, and the relevant identity must have the required permissions in Azure Key Vault.

## 6. Retrieving Secrets in Databricks Notebooks

Databricks provides `dbutils.secrets` for retrieving secret values from configured secret scopes.

### Retrieve a Single Secret

```python
password = dbutils.secrets.get(
    scope="database_scope",
    key="db_password"
)
```

### Retrieve Multiple Secrets

```python
username = dbutils.secrets.get(
    scope="database_scope",
    key="db_username"
)

password = dbutils.secrets.get(
    scope="database_scope",
    key="db_password"
)
```

### Use Secrets in a Database Connection

```python
jdbc_url = (
    "jdbc:sqlserver://myserver.database.windows.net:1433;"
    "databaseName=salesdb"
)

username = dbutils.secrets.get(
    scope="database_scope",
    key="db_username"
)

password = dbutils.secrets.get(
    scope="database_scope",
    key="db_password"
)

df = (
    spark.read
    .format("jdbc")
    .option("url", jdbc_url)
    .option("dbtable", "dbo.orders")
    .option("user", username)
    .option("password", password)
    .option("driver", "com.microsoft.sqlserver.jdbc.SQLServerDriver")
    .load()
)
```

Here:

- The JDBC URL identifies the database.
- The username and password are retrieved securely.
- Spark reads the database table into a DataFrame.

The database driver must be available to the compute environment.

**Important:** Do not print the password or include it in exception messages, logs, or notebook output.

## 7. Secret Scopes and Permissions

Creating a secret scope does not mean every workspace user should be able to retrieve its secrets.

Databricks provides scope-level permissions to control secret access.

Common permission levels include:

| Permission | Meaning |
|---|---|
| `READ` | Read secret values |
| `WRITE` | Create or update secrets |
| `MANAGE` | Manage scope permissions and secrets |

The exact permission behavior depends on the scope type and workspace configuration.

### Example: Grant Read Access

```bash
databricks secrets put-acl \
  database_scope \
  data_engineers \
  READ
```

### Example: Grant Manage Access

```bash
databricks secrets put-acl \
  database_scope \
  platform_admins \
  MANAGE
```

These examples use Databricks CLI syntax; verify the syntax against the CLI version used in your environment.

### Recommended Access Model

| Team | Access |
|---|---|
| Data Engineers | READ on required pipeline secrets |
| Platform Administrators | MANAGE where necessary |
| Business Users | No access to database credentials |
| Automated Workloads | Access through an approved identity and configuration |

**Best practice:** Give users and workloads access only to the secrets they actually need.

## 8. Secret Redaction and Its Limitations

Databricks attempts to redact certain secret values when they appear in notebook output.

For example, a secret retrieved through `dbutils.secrets.get()` may be displayed as redacted output in supported notebook contexts.

However, **redaction is not a complete security boundary**.

Sensitive information can still be exposed through:

- Logging statements.
- External libraries.
- Error messages.
- Data transformations.
- Network requests.
- Encoded or modified secret values.
- Other output or debugging mechanisms.

Example of unsafe code:

```python
password = dbutils.secrets.get(
    scope="database_scope",
    key="db_password"
)

print(password)
```

Even when notebook redaction is available, never intentionally print secrets.

### Safe Practice

```python
password = dbutils.secrets.get(
    scope="database_scope",
    key="db_password"
)

print("Database credential retrieved successfully")
```

The message confirms retrieval without exposing the value.

## 9. Credentials for Azure Data Lake Storage Gen2

A Data Engineer may need to access ADLS Gen2 from Databricks.

Common authentication approaches include:

1. Microsoft Entra ID-based authentication.
2. Managed identities where supported.
3. Service principals.
4. Unity Catalog storage credentials and external locations.

### Service Principal Authentication

A service principal represents an application identity in Microsoft Entra ID.

It commonly uses:

- Tenant ID
- Client ID
- Client secret or certificate

A simplified configuration example:

```python
client_id = dbutils.secrets.get(
    scope="azure_scope",
    key="client_id"
)

client_secret = dbutils.secrets.get(
    scope="azure_scope",
    key="client_secret"
)

tenant_id = dbutils.secrets.get(
    scope="azure_scope",
    key="tenant_id"
)

storage_account = "mystorageaccount"

spark.conf.set(
    f"fs.azure.account.auth.type.{storage_account}.dfs.core.windows.net",
    "OAuth"
)

spark.conf.set(
    f"fs.azure.account.oauth.provider.type.{storage_account}.dfs.core.windows.net",
    "org.apache.hadoop.fs.azurebfs.oauth2.ClientCredsTokenProvider"
)

spark.conf.set(
    f"fs.azure.account.oauth2.client.id.{storage_account}.dfs.core.windows.net",
    client_id
)

spark.conf.set(
    f"fs.azure.account.oauth2.client.secret.{storage_account}.dfs.core.windows.net",
    client_secret
)

spark.conf.set(
    f"fs.azure.account.oauth2.client.endpoint.{storage_account}.dfs.core.windows.net",
    f"https://login.microsoftonline.com/{tenant_id}/oauth2/token"
)
```

This is a traditional service-principal configuration pattern. Exact configuration support depends on the Databricks runtime, cloud environment, and storage setup.

**Production recommendation:** For Unity Catalog-governed storage access, prefer supported storage credentials and external locations, often using managed identities or service principals, rather than configuring credentials independently in every notebook.

## 10. Unity Catalog Storage Credentials

A storage credential is a Unity Catalog securable object representing an authentication mechanism for accessing cloud storage.

It can use supported authentication mechanisms such as:

- Azure managed identities.
- Azure service principals.
- AWS IAM roles.
- Other supported cloud identity mechanisms.

### Architecture

```text
        DATABRICKS
             |
             v
       UNITY CATALOG
             |
             v
     STORAGE CREDENTIAL
             |
             v
      EXTERNAL LOCATION
             |
             v
      CLOUD STORAGE
```

### Storage Credential vs Secret Scope

| Feature | Secret Scope | Storage Credential |
|---|---|---|
| Main purpose | Store and retrieve sensitive values | Provide governed cloud-storage authentication |
| Managed through | Databricks secret management | Unity Catalog |
| Common use | API keys, database passwords, tokens | ADLS Gen2, S3, cloud storage |
| Access control | Secret-scope permissions | Unity Catalog privileges |
| Used by | Notebooks and applications | Governed storage operations |

**Interview point:** A secret scope stores sensitive values. A storage credential represents an authentication mechanism for accessing governed cloud storage.

They are related to security but are not interchangeable.

## 11. Secret Scope vs Environment Variables vs Hardcoding

| Approach | Security | Recommendation |
|---|---|---|
| Hardcoded credentials | Unsafe | Avoid |
| Environment variables | Depends on how they are injected and protected | Useful in appropriate deployment environments |
| Databricks secret scopes | Centralized secret storage with scope-level access control | Useful for notebook and application secrets |
| Azure Key Vault | Dedicated cloud secret-management service | Useful for centralized Azure secret management |
| Unity Catalog storage credentials | Governed cloud-storage authentication | Preferred for supported Unity Catalog storage access |

Environment variables are not automatically secure. Their security depends on how values are supplied, stored, logged, and accessed.

## 12. Secret Rotation

Secret rotation means replacing an existing credential with a new one periodically or when a security event requires it.

Example:

```text
Old Database Password
          |
          v
Generate New Password
          |
          v
Update Secret Store
          |
          v
Validate Database Connection
          |
          v
Retire Old Password
```

### Why Is Rotation Important?

- Reduces the impact of leaked credentials.
- Supports security policies.
- Limits the lifetime of exposed credentials.
- Helps respond to suspected compromise.

### Recommended Rotation Process

1. Generate a new credential in the target system.
2. Update the secret in the approved secret store.
3. Test the connection using the new credential.
4. Update dependent applications if necessary.
5. Revoke the old credential after successful validation.
6. Monitor jobs and pipelines for authentication failures.

**Important:** Plan rotation carefully. Updating a secret in a store does not automatically guarantee that every running application or cached session immediately uses the new value.

## 13. Best Practices

1. Never hardcode passwords, API keys, or tokens.
2. Use secret scopes or an approved external secret manager.
3. Prefer Azure Key Vault integration when centralized Azure secret management is required.
4. Use Unity Catalog storage credentials for supported governed cloud-storage access.
5. Apply least-privilege permissions.
6. Prefer service principals or managed identities for automated workloads where appropriate.
7. Rotate credentials according to security requirements.
8. Never print secrets in notebooks.
9. Avoid passing credentials through notebook parameters or job arguments.
10. Do not commit secrets to Git repositories.
11. Avoid exposing credentials in exception messages and application logs.
12. Review secret access permissions regularly.
13. Use separate credentials for development, testing, and production.
14. Revoke compromised or unused credentials promptly.

## 14. Common Interview Mistakes

| Mistake | Correct Understanding |
|---|---|
| Secret and credential mean exactly the same thing | A secret is sensitive information; a credential is used to authenticate |
| Secret scopes automatically secure every credential | Access permissions and safe usage practices are still required |
| Redaction guarantees complete protection | Redaction has limitations |
| Every user should access every secret | Follow least privilege |
| Secret scopes and storage credentials are interchangeable | They serve different purposes |
| Environment variables are always secure | Their safety depends on how they are managed |
| Updating a secret immediately updates every running connection | Applications may need to reconnect or refresh credentials |
| Cloud storage access requires only a Databricks secret | Cloud permissions and authentication configuration must also be correct |

## 15. Interview Questions and Answers

**Q1. What is a secret in Databricks?**

A secret is sensitive information such as a password, API key, or token that should not be exposed in notebooks or source code. Databricks secret scopes provide a way to store and retrieve such values.

**Q2. What is a secret scope?**

A secret scope is a collection of secrets that can be accessed by authorized users or workloads.

**Q3. What are the two commonly discussed secret scope types?**

Databricks-backed secret scopes and Azure Key Vault-backed secret scopes.

**Q4. How do you retrieve a secret in PySpark?**

Using `dbutils.secrets.get()`.

```python
password = dbutils.secrets.get(
    scope="database_scope",
    key="db_password"
)
```

**Q5. Why should credentials not be hardcoded?**

Hardcoded credentials can be exposed through source code, notebooks, Git repositories, logs, or shared files.

**Q6. What is Azure Key Vault?**

Azure Key Vault is a cloud service used to securely store and manage secrets, encryption keys, and certificates.

**Q7. What is the difference between Databricks-backed and Key Vault-backed secret scopes?**

A Databricks-backed scope stores secrets in Databricks-managed secret storage. A Key Vault-backed scope integrates with Azure Key Vault, allowing secrets to be managed through that service.

**Q8. What is secret redaction?**

Secret redaction attempts to hide certain secret values when they appear in supported notebook outputs. It should not be treated as a complete security mechanism.

**Q9. How do you control access to secrets?**

Use secret-scope permissions and grant only the required access to users or groups.

**Q10. What is secret rotation?**

Secret rotation is the process of replacing existing credentials with new ones to reduce security risks.

**Q11. What is the difference between a secret scope and a storage credential?**

A secret scope stores sensitive values. A storage credential represents an authentication mechanism used by Unity Catalog to access governed cloud storage.

**Q12. How would you securely connect Databricks to Azure SQL Database?**

I would store the database credentials in an approved secret manager, retrieve them securely in the pipeline, use an authenticated JDBC connection, and avoid logging or printing the credentials.

**Q13. How would you securely connect Databricks to ADLS Gen2?**

I would prefer a supported Unity Catalog storage credential and external location, using managed identity or service-principal authentication where appropriate. I would also ensure the required cloud permissions are configured.

**Q14. What should you do if a secret is accidentally exposed?**

Treat it as compromised, rotate or revoke the credential, update dependent applications, investigate possible unauthorized access, and review logs and permissions.

**Q15. Are environment variables always secure?**

No. Their security depends on how they are injected, stored, exposed, and accessed.

## 16. Quick Revision

| Concept | Remember |
|---|---|
| Secret | Sensitive information |
| Credential | Used to authenticate |
| Secret Scope | Collection of secrets |
| Databricks-backed Scope | Secrets stored in Databricks-managed storage |
| Key Vault-backed Scope | Integrates with Azure Key Vault |
| `dbutils.secrets.get()` | Retrieves a secret |
| `READ` | Read secret values |
| `WRITE` | Create or update secrets |
| `MANAGE` | Manage scope access and secrets |
| Azure Key Vault | Azure secret-management service |
| Service Principal | Application identity |
| Managed Identity | Azure-managed identity |
| Storage Credential | Governed cloud-storage authentication |
| Secret Rotation | Replacing credentials |
| Secret Redaction | Hides certain secret values in supported outputs |
| Least Privilege | Give only necessary access |

### One-Minute Interview Answer

In Databricks, secrets and credentials are used to securely authenticate pipelines to external systems such as databases, APIs, and cloud storage.

Databricks provides secret scopes to store sensitive values such as passwords, API keys, and tokens. We can retrieve them in notebooks using `dbutils.secrets.get()` instead of hardcoding credentials.

In Azure environments, we can integrate with Azure Key Vault for centralized secret management.

For cloud storage access through Unity Catalog, we can use storage credentials and external locations with supported authentication mechanisms such as managed identities or service principals.

In production, I follow least privilege, avoid printing secrets, rotate credentials, and use separate credentials for different environments.

## 17. Self-Assessment

- [ ] Explain secrets and credentials.
- [ ] Explain why hardcoding credentials is unsafe.
- [ ] Define a Databricks secret scope.
- [ ] Differentiate Databricks-backed and Key Vault-backed scopes.
- [ ] Explain `dbutils.secrets.get()`.
- [ ] Explain secret-scope permissions.
- [ ] Explain secret redaction and its limitations.
- [ ] Explain Azure Key Vault.
- [ ] Explain service principal authentication.
- [ ] Explain Unity Catalog storage credentials.
- [ ] Differentiate secret scopes and storage credentials.
- [ ] Explain secret rotation.
- [ ] Explain least-privilege access.
- [ ] Describe a secure database connection.
- [ ] Describe a secure ADLS Gen2 connection.

**Interview readiness:** You should be able to explain how Databricks stores and retrieves secrets, how Azure Key Vault integrates with Databricks, and why Unity Catalog storage credentials are useful for governed cloud-storage access.