

# Secrets and Managed Identities in Azure

## 1. Overview

Data engineering pipelines often connect to databases, storage accounts, APIs, and other Azure services. These connections require authentication.

Applications need a secure way to access services without exposing passwords, connection strings, or access keys in code.

Azure provides two important mechanisms:

- **Secrets:** Securely store sensitive values such as passwords, API keys, and connection strings.
- **Managed Identities:** Allow Azure resources to authenticate to supported services without managing credentials manually.

They solve different problems and are often used together.

---

## 2. What Is a Secret?

A secret is sensitive information that should not be exposed in application code, configuration files, notebooks, or logs.

Examples include:

- Database passwords
- API keys
- Connection strings
- Service principal client secrets
- Access tokens

### Why should secrets be stored securely?

Consider this example:

```python
password = "MyDatabasePassword123"
```

Hardcoding credentials creates risks:

- Anyone with access to the source code may see them.
- Credentials can accidentally appear in Git commits.
- Changing a password requires updating the code or configuration.
- The same credentials may be reused across environments.

A better approach is to store secrets in **Azure Key Vault** and retrieve them securely when needed.

---

## 3. Azure Key Vault

Azure Key Vault is a managed Azure service for securely storing and controlling access to sensitive information.

It supports three main object types:

| Object | Purpose | Example |
|---|---|---|
| Secrets | Store sensitive values | Database password |
| Keys | Perform cryptographic operations | Encryption and signing |
| Certificates | Manage digital certificates | TLS/SSL certificate |

### Key Vault architecture

A typical architecture looks like this:

```text
Data Engineering Application
           |
           | Authenticates using
           | Managed Identity
           v
     Microsoft Entra ID
           |
           | Issues access token
           v
       Azure Key Vault
           |
           | Returns authorized secret
           v
   Application uses the secret
           |
           v
       Target Database
```

The application authenticates with its managed identity. Key Vault checks whether that identity has permission to read the requested secret.

### Important distinction

Key Vault stores and protects secrets. It does not automatically authenticate an application to every service.

The application retrieves the secret and uses it to authenticate to the target system.

---

## 4. What Is a Managed Identity?

A Managed Identity is an identity provided by Microsoft Entra ID for an Azure resource.

It allows an Azure resource to authenticate to other services without requiring you to store and manage credentials such as client secrets.

Examples:

- Azure Data Factory accessing Azure Key Vault
- Azure Databricks accessing ADLS Gen2
- Azure Functions accessing Azure SQL Database
- An Azure VM accessing a storage account

### How it works

1. Enable a managed identity on an Azure resource.
2. Azure associates the resource with an identity in Microsoft Entra ID.
3. The application requests an access token for a target service.
4. Microsoft Entra ID issues a token if authentication succeeds.
5. The target service checks the identity's permissions.
6. Access is granted or denied.

**Important:** A managed identity provides authentication, not automatic authorization. You must still grant it the required permissions on the target resource.

---

## 5. Types of Managed Identities

Azure supports two types:

### 5.1 System-Assigned Managed Identity

A system-assigned identity is created as part of an Azure resource.

Characteristics:

- Associated with one Azure resource.
- Its lifecycle is tied to that resource.
- Deleting the resource also deletes its system-assigned identity.
- It is suitable when an identity is needed by only one resource.

Example:

An Azure Data Factory instance uses its system-assigned managed identity to read a secret from Key Vault.

```text
Azure Data Factory
       |
       | System-assigned identity
       v
 Microsoft Entra ID
       |
       v
   Azure Key Vault
```

### 5.2 User-Assigned Managed Identity

A user-assigned identity is created as a separate Azure resource.

Characteristics:

- Exists independently of the Azure resource using it.
- Can be assigned to multiple supported Azure resources.
- Its lifecycle is independent of those resources.
- Useful when several applications need the same identity.

Example:

```text
          User-Assigned Identity
                    |
          -----------------------
          |          |          |
         ADF     Azure VM    Function
          |          |          |
          -----------------------
                    |
                    v
             Azure Key Vault
```

### System-Assigned vs User-Assigned

| Feature | System-Assigned | User-Assigned |
|---|---|---|
| Creation | Enabled on the resource | Created separately |
| Lifecycle | Tied to resource | Independent |
| Resource association | One resource | Multiple resources |
| Reusability | Limited | Reusable |
| Deletion | Deleted with resource | Remains until separately deleted |
| Common use | One application | Multiple applications |

**Interview point:** Use system-assigned identity when the identity belongs to one resource. Use user-assigned identity when multiple resources need a shared identity or when the identity lifecycle must be managed independently.

---

## 6. Managed Identity vs Service Principal

Both can represent an application identity in Microsoft Entra ID, but credential management differs.

### Service Principal

A service principal represents an application or workload identity in a particular Entra tenant.

A common authentication method uses:

- Tenant ID
- Client ID
- Client secret or certificate

Example:

```python
from azure.identity import ClientSecretCredential

credential = ClientSecretCredential(
    tenant_id="<tenant-id>",
    client_id="<client-id>",
    client_secret="<client-secret>"
)
```

The client secret must be stored securely and rotated when required.

### Managed Identity

A managed identity is an Azure-managed identity associated with an Azure resource.

The platform manages its credentials and token acquisition. The application does not need to maintain a client secret.

Example:

```python
from azure.identity import ManagedIdentityCredential

credential = ManagedIdentityCredential()
```

For a user-assigned identity, the client ID can be supplied:

```python
credential = ManagedIdentityCredential(
    client_id="<user-assigned-managed-identity-client-id>"
)
```

### Comparison

| Feature | Managed Identity | Service Principal |
|---|---|---|
| Identity type | Entra application identity | Entra application identity |
| Credential management | Managed by Azure | Managed by application/team |
| Client secret required | No | Commonly, but certificate or other methods are possible |
| Azure resource association | Yes | Not required |
| Suitable for Azure workloads | Excellent when supported | Yes |
| Suitable for external applications | Not generally | Yes |
| Credential rotation | Managed by platform | Must be managed when using secrets/certificates |

**Interview point:** Prefer managed identity for supported Azure-hosted workloads because it avoids storing and rotating application credentials. Service principals remain useful for external systems, CI/CD, and workloads that cannot use managed identities.

---

## 7. Authentication vs Authorization

These two concepts are frequently asked in interviews.

### Authentication

Authentication verifies the identity of the user, application, or service.

Question:

**Who are you?**

Example: An Azure Data Factory instance authenticates using its managed identity.

### Authorization

Authorization determines what an authenticated identity is allowed to do.

Question:

**What are you allowed to access?**

Example: The managed identity has permission to read secrets from a Key Vault.

### Example

Suppose Azure Data Factory needs to retrieve a database password.

1. ADF authenticates using its managed identity.
2. Microsoft Entra ID issues an access token.
3. Key Vault checks the identity's permissions.
4. If authorized, Key Vault returns the secret.

Authentication alone does not guarantee access.

---

## 8. Azure Key Vault Access Control

Key Vault supports two access-control models:

### 8.1 Azure RBAC Permission Model

Azure Role-Based Access Control manages permissions through role assignments.

Common Key Vault roles include:

| Role | Purpose |
|---|---|
| Key Vault Administrator | Full management of Key Vault data-plane operations |
| Key Vault Secrets Officer | Manage secrets |
| Key Vault Secrets User | Read secret contents |
| Key Vault Reader | Read Key Vault metadata, not secret contents |

For an application that only needs to retrieve secret values, **Key Vault Secrets User** is generally the relevant role.

The exact role should follow the application's required access.

### 8.2 Vault Access Policy Model

The older access policy model grants permissions directly to identities.

For example:

- Get: Read a secret.
- List: List secrets.
- Set: Create or update a secret.
- Delete: Delete a secret.

### RBAC vs Access Policies

| Feature | Azure RBAC | Access Policies |
|---|---|---|
| Permission management | Role assignments | Individual access policies |
| Scope | Supports Azure resource scopes | Configured on the vault |
| Centralized management | Integrates with Azure RBAC | Separate Key Vault permission model |
| Common recommendation | Preferred for new deployments | Legacy or existing deployments |

**Important:** A Key Vault is configured to use either the RBAC permission model or the access policy model. Do not assume both models grant permissions simultaneously.

Also, a role such as Contributor over the Key Vault management resource does not automatically grant permission to read secret values.

---

## 9. Secret Versioning and Rotation

Key Vault supports multiple versions of a secret.

Suppose a database password changes:

```text
DatabasePassword
    |
    |-- Version 1: Old password
    |
    |-- Version 2: New password
    |
    |-- Version 3: Latest password
```

Applications can retrieve the latest version or a specific version when required.

### What is secret rotation?

Secret rotation means periodically changing a credential to reduce the risk of compromise.

Example:

1. Create a new database password.
2. Update the database account with the new password.
3. Store the new password as a new Key Vault secret version.
4. Ensure applications retrieve and use the new version.
5. Verify successful connections.
6. Retire the old credential when safe.

**Important:** Creating a new Key Vault secret version does not automatically change the password in the database. The target system and consuming applications must also be updated.

### Best practices

- Rotate credentials according to organizational policy.
- Avoid permanent credentials where possible.
- Use managed identity instead of client secrets for supported Azure workloads.
- Monitor secret expiration and access.
- Avoid logging secret values.
- Test rotation before removing old credentials.

---

## 10. Using Managed Identity with Azure Data Factory

Azure Data Factory can use its managed identity to access Key Vault.

### Example scenario

An ADF pipeline needs a password to connect to an external database.

```text
        Azure Data Factory
                 |
                 | Managed Identity
                 v
          Microsoft Entra ID
                 |
                 v
            Azure Key Vault
                 |
                 | Database password
                 v
        External Database
```

### Configuration steps

1. Open the Azure Data Factory resource.
2. Enable or locate its system-assigned managed identity.
3. Open the Key Vault.
4. Grant the ADF identity the required permission to retrieve secrets.
5. Configure an ADF Key Vault linked service.
6. Configure the database linked service to retrieve the password from Key Vault.
7. Test the linked service connection.

### Why use this approach?

- No password needs to be hardcoded in the pipeline.
- The identity is managed by Azure.
- Access can be controlled through permissions.
- Credential changes can be handled centrally.

**Interview point:** ADF's managed identity authenticates to Key Vault. Key Vault authorization determines whether ADF can retrieve the requested secret.

---

## 11. Using Managed Identity with Azure Databricks and ADLS Gen2

Databricks workloads often need access to ADLS Gen2.

A secure architecture can use a managed identity with Unity Catalog.

```text
       Azure Databricks
               |
               | Unity Catalog
               v
       Storage Credential
               |
               | References
               v
       Managed Identity
               |
               | Authorized through
               v
       ADLS Gen2 Storage
```

### Important components

**Storage Credential:** A Unity Catalog object representing an authentication mechanism for accessing cloud storage.

**External Location:** A Unity Catalog object that combines a cloud storage path with a storage credential.

**Managed Identity:** The identity used to authenticate to the storage account.

### High-level setup

1. Create or configure a managed identity.
2. Grant the identity the required access to ADLS Gen2.
3. Create a Unity Catalog storage credential that uses the supported managed identity configuration.
4. Create an external location using the storage credential and an ADLS Gen2 path.
5. Grant users or groups the required Unity Catalog privileges.
6. Access the external location through governed tables or volumes.

### Important distinction

There are two separate permission layers:

- Azure permissions control access to the underlying storage.
- Unity Catalog privileges control which Databricks users can use the governed storage objects.

A user may have Unity Catalog permission but the underlying storage identity may lack Azure permissions. The operation can still fail.

---

## 12. Key Vault Integration with Azure Databricks

Databricks can also retrieve secrets from Azure Key Vault through a Key Vault-backed secret scope.

A simplified flow:

```text
       Databricks Notebook
                |
                v
          Secret Scope
                |
                v
          Azure Key Vault
                |
                v
          Secret Value
```

Example:

```python
database_password = dbutils.secrets.get(
    scope="production-secrets",
    key="database-password"
)
```

Use the retrieved value when creating a secure connection. Do not print it or include it in logs.

### Important notes

- Secret scopes provide a way to access secrets from notebooks.
- Access to a secret scope should be restricted.
- Key Vault-backed secret scopes integrate with Azure Key Vault.
- A secret scope is not the same thing as a managed identity.
- Databricks secret redaction is a safeguard, not a replacement for secure coding.
- The exact secret-scope capabilities and configuration depend on the Databricks workspace and platform setup.

For new storage access patterns, consider Unity Catalog storage credentials and external locations rather than relying on legacy mounts.

---

## 13. Managed Identity in Azure Functions

Azure Functions can use a managed identity to access other Azure services.

Example use cases:

- Read a secret from Key Vault.
- Read or write files in ADLS Gen2.
- Send messages to Event Hubs.
- Connect to Azure SQL Database.

Python example using `DefaultAzureCredential`:

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

credential = DefaultAzureCredential()

client = SecretClient(
    vault_url="https://<vault-name>.vault.azure.net/",
    credential=credential
)

secret = client.get_secret("database-password")

database_password = secret.value
```

`DefaultAzureCredential` can use different authentication methods depending on the environment. In a correctly configured Azure-hosted application, it can use the resource's managed identity.

For local development, it may use a developer credential such as Azure CLI authentication.

---

## 14. Managed Identity and Azure SQL Database

An Azure-hosted application can authenticate to Azure SQL Database using a managed identity, provided the identity is configured in the database and has the required database permissions.

High-level process:

1. Enable managed identity on the application resource.
2. Configure Microsoft Entra authentication for Azure SQL.
3. Create a database user for the managed identity.
4. Grant the required database permissions.
5. Configure the application to request an Azure SQL access token.
6. Connect using the token.

Example database configuration:

```sql
CREATE USER [my-managed-identity] FROM EXTERNAL PROVIDER;

ALTER ROLE db_datareader
ADD MEMBER [my-managed-identity];
```

The exact database user name and permissions depend on the identity and application requirements.

**Important:** Azure RBAC roles on the SQL resource do not replace SQL database permissions for ordinary database queries.

---

## 15. Security Best Practices

Follow these practices when designing production data engineering systems.

### Identity management

- Prefer managed identities for supported Azure-hosted workloads.
- Use separate identities for different environments when appropriate.
- Avoid sharing one powerful identity across unrelated applications.
- Remove unused identities and role assignments.
- Apply least-privilege access.

### Secret management

- Store passwords and API keys in Key Vault.
- Never hardcode credentials in notebooks, scripts, or Git repositories.
- Avoid printing secrets in logs.
- Rotate credentials according to policy.
- Monitor secret access and expiration.
- Use short-lived credentials where supported.

### Access control

- Grant only the permissions required.
- Assign roles at the narrowest practical scope.
- Review permissions periodically.
- Separate development, testing, and production access.
- Remember that authentication and authorization are separate.

### Network security

- Use private endpoints where appropriate.
- Configure private DNS correctly.
- Restrict public network access when the architecture allows it.
- Remember that network access does not automatically grant data permissions.

---

## 16. Common Errors and Troubleshooting

| Error or symptom | Possible reason | What to check |
|---|---|---|
| 401 Unauthorized | Authentication failed or token invalid | Identity configuration, token audience, credential |
| 403 Forbidden | Identity lacks permission | Key Vault role or access policy |
| Secret not found | Incorrect secret name or version | Secret name, version, vault |
| ADF cannot retrieve secret | Managed identity not authorized | ADF identity and Key Vault permissions |
| Databricks cannot access ADLS | Storage identity lacks permission | Storage credential, RBAC, ACL, external location |
| SQL login failed | Database identity or permissions incorrect | Entra authentication and database user |
| Works locally but fails in Azure | Local credential differs from deployed identity | Managed identity configuration and target permissions |

### A useful troubleshooting sequence

1. Confirm which identity the application is using.
2. Confirm that the identity exists and is enabled.
3. Confirm the requested resource and token audience.
4. Check network connectivity and firewall rules.
5. Check the target service's authorization settings.
6. Verify the secret name, version, or storage path.
7. Check diagnostic logs without exposing sensitive values.

**Remember:** A 403 response often indicates an authorization problem, but network restrictions and service-specific policies can also cause access failures.

---

## 17. Managed Identity vs Key Vault

These services are complementary, not alternatives.

| Feature | Managed Identity | Azure Key Vault |
|---|---|---|
| Main purpose | Authenticate an Azure workload | Securely store secrets, keys, and certificates |
| Stores passwords | No | Yes |
| Provides application identity | Yes | No |
| Avoids managing client secrets | Yes, for supported authentication | Stores secrets securely when needed |
| Controls access | Identity is granted permissions at target services | Controls access to vault objects |
| Example | ADF authenticating to Key Vault | Key Vault storing a database password |

### Simple way to remember

- **Managed Identity:** Who is accessing the service?
- **Key Vault:** Where are sensitive credentials stored?
- **RBAC/Access Policy:** What is the identity allowed to access?

---

## 18. Practical Data Engineering Scenario

### Scenario: ADF pipeline loads data from an external database into ADLS Gen2

Requirements:

- ADF connects to an external database.
- The database password must not be hardcoded.
- Data is copied into ADLS Gen2.
- Access should follow least-privilege principles.

### Recommended architecture

```text
                 Azure Data Factory
                         |
              System-Assigned Identity
                         |
              -----------------------
              |                     |
              v                     v
         Azure Key Vault        ADLS Gen2
              |                     |
              |                     |
       Database Password       Raw Data Files
              |
              v
       External Database
```

### Execution flow

1. ADF starts the pipeline.
2. ADF uses its managed identity to authenticate to Key Vault.
3. Key Vault checks whether the identity can retrieve the database password.
4. ADF retrieves the password through the configured linked service.
5. ADF connects to the external database.
6. Copy Activity reads the required data.
7. ADF writes the data into ADLS Gen2.
8. The storage access configuration authorizes the write operation.
9. Pipeline monitoring records execution status and activity metrics.

### Security considerations

- ADF should have only the required Key Vault secret permissions.
- ADF should have only the required storage data permissions.
- Database credentials should not appear in pipeline logs.
- Use separate identities or permissions for development and production where appropriate.
- Configure private connectivity if required by the security architecture.

---

## 19. Interview Questions and Answers

### Q1. What is Azure Key Vault?

Azure Key Vault is a managed Azure service used to securely store and control access to secrets, keys, and certificates.

### Q2. What is a managed identity?

A managed identity is an identity provided through Microsoft Entra ID for an Azure resource. It allows the resource to authenticate to supported services without manually managing credentials.

### Q3. What are the two types of managed identities?

- System-assigned managed identity
- User-assigned managed identity

### Q4. What is the difference between system-assigned and user-assigned identities?

A system-assigned identity is tied to one Azure resource and is deleted when that resource is deleted. A user-assigned identity is independent and can be associated with multiple supported resources.

### Q5. Does managed identity automatically provide access to Azure Storage?

No. It provides authentication. The identity must also receive the required storage permissions through Azure RBAC, ACLs, or the applicable authorization mechanism.

### Q6. What is the difference between authentication and authorization?

Authentication verifies who the identity is. Authorization determines what that identity can access or perform.

### Q7. What is the difference between Key Vault and managed identity?

Key Vault securely stores sensitive values. Managed identity provides an identity that an Azure resource can use to authenticate to services.

### Q8. Why should we not hardcode passwords in notebooks?

Hardcoded credentials can be exposed through source code, Git repositories, logs, or shared notebooks. Secure secret management reduces this risk.

### Q9. Can multiple Azure resources use the same managed identity?

Yes. Multiple supported Azure resources can use a user-assigned managed identity. A system-assigned identity is tied to its individual resource.

### Q10. What is secret rotation?

Secret rotation is the process of changing a credential periodically or when needed, updating the secure store and dependent applications, and retiring the old credential safely.

### Q11. Does changing a Key Vault secret automatically update the database password?

No. The database credential must also be changed in the database. Applications must then retrieve and use the new credential.

### Q12. What is the difference between Key Vault RBAC and access policies?

RBAC uses Azure role assignments to control access to Key Vault data. Access policies grant specific data permissions directly to identities. A vault uses one of these permission models.

### Q13. Can an Azure Data Factory pipeline use a managed identity to access Key Vault?

Yes. ADF can use its managed identity to authenticate to Key Vault, provided that identity has the required secret permissions.

### Q14. Can Azure Databricks use managed identity to access ADLS Gen2?

Yes. A supported configuration can use a managed identity through Unity Catalog storage credentials and external locations. The identity also needs the required Azure storage permissions.

### Q15. What is the difference between a service principal and a managed identity?

Both represent application identities in Microsoft Entra ID. A service principal commonly uses a client secret or certificate that the team manages. Managed identity credentials are managed by Azure for supported Azure resources.

### Q16. What would you check if an application receives a 403 error from Key Vault?

I would verify the identity being used, its Key Vault permission model, its assigned role or access policy, the scope, and any network restrictions.

### Q17. Why is least privilege important?

It limits the damage caused by compromised identities or application errors by ensuring each identity has only the permissions it needs.

### Q18. Is a Key Vault secret scope the same as a managed identity?

No. A secret scope is a Databricks mechanism for accessing secrets. A managed identity is an identity used for authentication.

---

## 20. Common Mistakes to Avoid

- Assuming managed identity automatically grants storage or Key Vault access.
- Treating authentication and authorization as the same thing.
- Confusing Key Vault with Microsoft Entra ID.
- Assuming Contributor on a Key Vault grants permission to read secret values.
- Using a client secret when a managed identity is supported and appropriate.
- Storing passwords in notebook source code.
- Printing secret values during debugging.
- Assuming a new Key Vault secret version automatically updates the target database.
- Confusing system-assigned and user-assigned identity lifecycles.
- Assuming a private endpoint grants authorization.
- Ignoring the difference between Azure RBAC and SQL database permissions.

---

## 21. Quick Revision

| Concept | Remember |
|---|---|
| Azure Key Vault | Stores secrets, keys, and certificates |
| Secret | Sensitive value such as a password or API key |
| Managed Identity | Azure-managed workload identity |
| System-assigned | Tied to one Azure resource |
| User-assigned | Independent and reusable |
| Service Principal | Application identity with credentials managed by the team or platform |
| Authentication | Verifies identity |
| Authorization | Determines permissions |
| Azure RBAC | Role-based permission management |
| Access Policy | Key Vault's alternative data permission model |
| Secret Rotation | Changes credentials and updates dependent systems |
| Least Privilege | Grants only required permissions |
| 401 | Often an authentication issue |
| 403 | Often an authorization or access-policy issue |

### Five points to remember for interviews

1. Managed identity removes the need to manage credentials for supported Azure workloads.
2. Managed identity does not automatically grant access; permissions must be assigned.
3. Key Vault securely stores secrets, keys, and certificates.
4. System-assigned identity follows the resource lifecycle; user-assigned identity is independent.
5. Key Vault and managed identity commonly work together in secure Azure data pipelines.

---

## 22. Self-Assessment

Try answering these without looking at the notes:

1. Explain the difference between Key Vault and managed identity.
2. Compare system-assigned and user-assigned managed identities.
3. Explain authentication and authorization using an ADF example.
4. How would you securely retrieve a database password in ADF?
5. How can Databricks access ADLS Gen2 using a managed identity?
6. What is the difference between a service principal and a managed identity?
7. What would you check if Key Vault returns a 403 error?
8. Why does secret rotation require coordination with the target database?
9. Explain the difference between Key Vault RBAC and access policies.
10. Design a secure architecture for an ADF pipeline that reads from a database and writes to ADLS Gen2.

