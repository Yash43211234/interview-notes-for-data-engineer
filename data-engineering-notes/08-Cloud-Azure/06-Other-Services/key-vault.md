# Azure Key Vault

## 1. What Is Azure Key Vault?

**Azure Key Vault** is a cloud service used to securely store and manage sensitive information such as:

- Secrets — passwords, connection strings, API keys, and tokens.
- Keys — cryptographic keys used for encryption and decryption.
- Certificates — digital certificates used for authentication and secure communication.

It helps applications and data pipelines access sensitive information without hardcoding credentials in source code or configuration files.

**Example:**

A Databricks notebook needs a database password to connect to Azure SQL Database.

Instead of writing the password directly in the notebook, store it in Azure Key Vault and retrieve it securely when the notebook runs.

---

## 2. Why Do We Need Key Vault?

Without Key Vault, developers may store credentials in:

- Python scripts.
- Configuration files.
- Notebook cells.
- Pipeline parameters.
- Git repositories.

This creates security risks.

Key Vault helps by providing:

1. Centralized secret management.
2. Controlled access through identities and permissions.
3. Secret versioning.
4. Secret expiration and rotation support.
5. Auditing and monitoring.
6. Integration with Azure services.

**Important:** Key Vault stores sensitive information. It does not automatically grant an application permission to use the resource protected by that information.

---

## 3. Core Components

Key Vault manages three main types of objects.

### 3.1 Secrets

Secrets are sensitive values that applications need at runtime.

Examples:

- Database passwords.
- API keys.
- Connection strings.
- Access tokens.

Example secret names:

```text
sql-database-password
api-key
storage-connection-string
```

A secret contains a value and associated metadata, such as its activation date, expiration date, and version.

### 3.2 Keys

Keys are cryptographic objects used for operations such as:

- Encryption.
- Decryption.
- Signing.
- Signature verification.
- Key wrapping and unwrapping.

Keys can be software-protected or backed by a Hardware Security Module (HSM), depending on the key type and vault configuration.

**Difference:** A secret stores a sensitive value, while a key performs cryptographic operations.

### 3.3 Certificates

Certificates help establish identity and secure communication.

They are commonly used for:

- HTTPS/TLS.
- Application authentication.
- Service-to-service communication.
- Certificate lifecycle management.

A certificate generally contains a public key and identity information, and may be associated with a private key managed by Key Vault.

---

## 4. Key Vault Architecture

A typical architecture looks like this:

```text
             Azure Application
                    |
                    | Authenticates using
                    v
          Microsoft Entra ID
                    |
                    | Issues access token
                    v
              Azure Key Vault
                    |
          Retrieves authorized secret
                    |
                    v
             Application
                    |
                    | Uses credential
                    v
             Azure SQL Database
```

### How It Works

1. An application or Azure service starts.
2. It authenticates using Microsoft Entra ID.
3. Entra ID provides an access token.
4. The application sends a request to Key Vault.
5. Key Vault checks whether the identity has the required permission.
6. If authorized, Key Vault returns the requested secret or performs the requested key operation.
7. The application uses the returned information to connect to another service.

**Important:** Authentication establishes who the application is. Authorization determines what it can access.

---

## 5. Authentication and Authorization

Key Vault uses Microsoft Entra ID for identity-based authentication.

Common identities include:

| Identity | Purpose |
|---|---|
| User | A person accessing Key Vault |
| Service principal | An application identity |
| System-assigned managed identity | Identity attached to an Azure resource |
| User-assigned managed identity | Independent identity that can be assigned to multiple resources |

### Managed Identity

A managed identity allows an Azure resource to authenticate to another Azure service without storing credentials in application code.

For example:

```text
Azure Data Factory
       |
       | Managed Identity
       v
   Azure Key Vault
       |
       | Reads database password
       v
   Azure SQL Database
```

The identity still needs permission to access the secret.

**Remember:** Managed Identity provides authentication, not automatic authorization.

---

## 6. Key Vault Access Control

Azure Key Vault supports two permission models:

### 6.1 Azure Role-Based Access Control (RBAC)

RBAC assigns permissions through Azure roles.

Common Key Vault data-plane roles include:

| Role | Purpose |
|---|---|
| Key Vault Administrator | Full control over Key Vault data-plane operations |
| Key Vault Secrets Officer | Manage secrets |
| Key Vault Secrets User | Read secret contents |
| Key Vault Reader | Read Key Vault metadata, but not secret contents |
| Key Vault Crypto Officer | Manage keys |
| Key Vault Crypto User | Perform permitted cryptographic operations |

The exact permissions depend on the role definition.

### 6.2 Vault Access Policies

Access policies are an older permission model that grants permissions directly for secrets, keys, and certificates.

For example, a policy can allow an identity to:

- Get secrets.
- List secrets.
- Set secrets.
- Delete secrets.

### RBAC vs Access Policies

| Feature | Azure RBAC | Access Policies |
|---|---|---|
| Permission model | Azure roles | Individual object permissions |
| Assignment | Azure role assignment | Vault access policy |
| Scope | Supports Azure resource scopes | Configured within the vault |
| Management | Centralized Azure IAM | Key Vault-specific |
| Recommendation | Preferred for new designs | Legacy option |

A Key Vault uses a configured permission model. Do not assume both models control data access simultaneously.

**Interview point:** For new implementations, Azure RBAC is generally preferred because it provides centralized permission management and consistent role assignments.

---

## 7. Secret Management and Versioning

Key Vault maintains versions when a secret is updated.

Example:

```text
Secret: sql-password

Version 1 → Old password
Version 2 → Updated password
Version 3 → Latest password
```

When an application requests a secret without specifying a version, it normally receives the latest enabled version.

### Why Versioning Matters

- Supports controlled credential updates.
- Helps applications transition to new credentials.
- Provides historical versions while they remain available.
- Helps with troubleshooting and recovery.

### Secret Rotation

Secret rotation means periodically changing a credential and updating the applications that use it.

Example:

1. Generate a new database password.
2. Update the database account.
3. Store the new password in Key Vault.
4. Allow the application to retrieve the updated secret.
5. Verify the application connection.
6. Retire the old credential when it is no longer needed.

**Important:** Updating a secret in Key Vault does not automatically change the password in the database. Both systems must be coordinated.

---

## 8. Integrating Key Vault with Azure Data Factory

Azure Data Factory can use Key Vault to retrieve credentials for linked services.

### Example Scenario

An ADF pipeline copies data from Azure SQL Database to ADLS Gen2.

The SQL connection requires a password.

Instead of storing the password directly in the linked service, reference a secret from Key Vault.

### Architecture

```text
             Azure Data Factory
                      |
                      | Managed Identity
                      v
                Azure Key Vault
                      |
                      | Retrieves SQL password
                      v
               Azure SQL Database
                      |
                      | Copy Activity
                      v
                  ADLS Gen2
```

### Configuration Steps

1. Create an Azure Key Vault.
2. Store the database password as a secret.
3. Enable the ADF managed identity.
4. Grant that identity permission to read the required secret.
5. Open the ADF linked service configuration.
6. Configure the Key Vault linked service.
7. Select the required secret.
8. Test the connection and publish the changes.

### Example

A linked service can reference a Key Vault secret rather than containing a literal password.

Conceptually:

```text
Password:
    Azure Key Vault Secret Reference

Secret name:
    sql-database-password
```

ADF retrieves the secret when it needs the credential.

**Interview point:** ADF uses its managed identity to authenticate to Key Vault. The identity must have permission to retrieve the secret.

---

## 9. Integrating Key Vault with Azure Databricks

Databricks applications and notebooks may need credentials to access external systems.

Examples:

- Azure SQL Database.
- REST APIs.
- External databases.
- Other services requiring credentials.

There are multiple ways to integrate secrets with Databricks. One approach is a Key Vault-backed secret scope.

### Key Vault-Backed Secret Scope

A secret scope provides a logical way to retrieve secrets from a configured secret backend.

Conceptual flow:

```text
Databricks Notebook
        |
        | Requests secret
        v
Databricks Secret Scope
        |
        | Retrieves from
        v
Azure Key Vault
        |
        v
Secret Value
```

### Example: Retrieve a Secret

```python
database_password = dbutils.secrets.get(
    scope="production-scope",
    key="sql-password"
)
```

The returned value can be used by the notebook to establish a database connection.

### Important Security Practices

- Never print secret values.
- Never hardcode passwords in notebooks.
- Restrict access to secret scopes.
- Use separate secrets for development and production.
- Avoid exposing credentials in logs or exception messages.
- Prefer managed identities or other supported identity-based authentication where possible.

**Important:** A secret scope is an access interface, not a replacement for Key Vault permissions. Access to a scope and access to the underlying secret must be configured correctly.

---

## 10. Key Vault Networking

Key Vault access can be controlled using network security features.

Common options include:

### Public Network Access

The vault can be accessed through its public endpoint, subject to configured network rules and authentication.

### Firewall Rules

Firewall rules can restrict which network sources can reach the vault.

### Private Endpoint

A private endpoint provides a private IP address in a Virtual Network for accessing Key Vault through Azure Private Link.

```text
Azure Databricks
       |
       v
   Virtual Network
       |
       v
  Private Endpoint
       |
       v
   Azure Key Vault
```

### Private Endpoint vs Firewall

| Feature | Firewall Rules | Private Endpoint |
|---|---|---|
| Main purpose | Restrict network access | Provide private connectivity |
| Private IP in VNet | Not by itself | Yes |
| Network isolation | Depends on allowed networks | Uses Private Link |
| DNS considerations | Depends on endpoint | Private DNS configuration is important |

**Remember:** A private endpoint does not grant permission to read secrets. Identity-based authorization is still required.

---

## 11. Monitoring and Auditing

Key Vault can be integrated with Azure Monitor and diagnostic logging.

Monitoring helps identify:

- Secret access requests.
- Unauthorized access attempts.
- Key operations.
- Certificate operations.
- Configuration and availability issues.

Diagnostic settings can send supported logs to destinations such as:

- Log Analytics workspace.
- Storage account.
- Event Hub.

### Why Auditing Matters

Suppose a production application unexpectedly accesses a secret.

Audit logs can help investigate:

- Which identity made the request.
- When the request occurred.
- Which operation was performed.
- Whether the request succeeded or failed.

Logging availability and event details depend on the configured diagnostic settings and service capabilities.

---

## 12. Key Vault vs Azure App Configuration

These services solve different problems.

| Feature | Azure Key Vault | Azure App Configuration |
|---|---|---|
| Main purpose | Manage sensitive information | Manage application settings |
| Typical data | Passwords, keys, certificates | Feature flags, URLs, configuration values |
| Secret storage | Yes | Can reference Key Vault secrets |
| Versioning | Supported for secrets, keys, and certificates | Supported for configuration values |
| Main use | Security and credential management | Application configuration management |

**Example:**

- Database password → Key Vault.
- Application environment name → App Configuration.
- Feature flag → App Configuration.
- API key → Key Vault.

App Configuration can reference a Key Vault secret instead of storing the secret value directly.

---

## 13. Key Vault vs Managed Identity

These are not competing services.

| Feature | Azure Key Vault | Managed Identity |
|---|---|---|
| Type | Secret/key/certificate management service | Identity mechanism |
| Main purpose | Securely store and manage sensitive data | Authenticate Azure resources |
| Stores passwords | Yes | No |
| Provides authentication | No, it relies on Entra ID | Yes, through Entra ID |
| Requires authorization | Yes | Still needs permissions on target resources |

**Example:**

A Databricks job uses a managed identity to authenticate to Azure Key Vault and retrieve a database credential.

---

## 14. Common Problems and Troubleshooting

### Problem 1: Access Denied

**Possible reasons:**

- Managed identity is not enabled.
- Identity does not have the required role.
- Wrong permission model is configured.
- Role assignment has not propagated yet.
- The requested operation is not allowed by the assigned role.

**Solution:**

1. Confirm which identity is being used.
2. Check the vault's permission model.
3. Verify the role assignment or access policy.
4. Confirm the scope and requested operation.
5. Retry after allowing time for permission changes to propagate.

### Problem 2: Secret Not Found

**Possible reasons:**

- Incorrect secret name.
- Incorrect vault configuration.
- Secret is disabled.
- Wrong secret version is requested.
- Secret has expired.

**Solution:** Verify the vault, secret name, version, and secret status.

### Problem 3: Network Connection Failure

**Possible reasons:**

- Firewall restrictions.
- Private endpoint configuration issue.
- Incorrect private DNS resolution.
- Missing network connectivity.

**Solution:** Check firewall rules, VNet connectivity, private endpoint status, and DNS resolution.

### Problem 4: ADF Cannot Retrieve a Secret

**Possible reasons:**

- ADF managed identity is not authorized.
- Key Vault linked service is incorrectly configured.
- Secret name is incorrect.
- Network restrictions block access.

**Solution:** Verify the ADF identity, Key Vault permissions, linked service configuration, and network settings.

### Problem 5: Application Uses an Old Password

**Possible reasons:**

- Application cached the secret.
- Application explicitly requests an older version.
- Secret rotation was incomplete.
- The application did not refresh its connection.

**Solution:** Check the requested secret version, refresh behavior, and credential update process.

---

## 15. Security Best Practices

1. Never hardcode passwords or API keys.
2. Use Managed Identity wherever supported.
3. Apply least-privilege access.
4. Prefer Azure RBAC for new Key Vault implementations.
5. Enable diagnostic logging and audit monitoring.
6. Configure expiration and rotation policies where appropriate.
7. Use private endpoints when private network access is required.
8. Separate development, testing, and production secrets.
9. Avoid printing secret values in logs.
10. Restrict who can manage permissions and delete secrets.
11. Use soft delete and purge protection to protect against accidental or malicious deletion.
12. Review access permissions periodically.

### Soft Delete and Purge Protection

**Soft delete:** Deleted Key Vault objects can be recovered during the configured retention period.

**Purge protection:** Prevents permanently purging a deleted object before its retention period expires.

These features help protect secrets, keys, and certificates from accidental or malicious deletion.

---

## 16. Practical Data Engineering Scenario

### Scenario: Secure SQL-to-ADLS Pipeline

A company wants to copy customer data from Azure SQL Database into ADLS Gen2 using Azure Data Factory.

The SQL database requires a password.

### Architecture

```text
             Azure Data Factory
                     |
                     | Managed Identity
                     v
               Azure Key Vault
                     |
                     | SQL password
                     v
              Azure SQL Database
                     |
                     | Copy Activity
                     v
                  ADLS Gen2
```

### Execution Flow

1. ADF pipeline starts.
2. ADF authenticates using its managed identity.
3. ADF retrieves the SQL password from Key Vault through the configured linked service.
4. ADF connects to Azure SQL Database.
5. Copy Activity reads customer data.
6. Data is written to ADLS Gen2.
7. Pipeline execution details are available in ADF monitoring.

### Benefits

- No password hardcoded in the pipeline.
- Centralized credential management.
- Controlled identity-based access.
- Easier credential rotation.
- Better auditing and security.

---

## 17. Common Interview Questions and Answers

### Q1. What is Azure Key Vault?

Azure Key Vault is a service for securely storing and managing secrets, cryptographic keys, and certificates.

### Q2. What are the three main objects managed by Key Vault?

Secrets, keys, and certificates.

### Q3. What is a secret?

A secret is sensitive information such as a password, API key, token, or connection string.

### Q4. What is the difference between a secret and a key?

A secret stores a sensitive value, whereas a key is used for cryptographic operations such as encryption, decryption, and signing.

### Q5. What is the difference between authentication and authorization?

Authentication verifies identity. Authorization determines what that identity is allowed to access.

### Q6. How does Managed Identity help with Key Vault?

Managed Identity allows an Azure resource to authenticate to Key Vault without storing credentials in application code. The identity must still receive the required permissions.

### Q7. How can ADF access a Key Vault secret?

ADF can use its managed identity through a Key Vault linked service. The managed identity must have permission to retrieve the secret.

### Q8. How can Databricks access Key Vault secrets?

A Databricks workspace can use a Key Vault-backed secret scope, where notebooks retrieve secrets through `dbutils.secrets.get()`.

### Q9. What is the difference between RBAC and Access Policies?

RBAC uses Azure role assignments to control access. Access Policies use Key Vault-specific permissions for secrets, keys, and certificates.

### Q10. What is secret rotation?

Secret rotation is the process of periodically replacing credentials and ensuring dependent applications use the updated values.

### Q11. Does updating a secret in Key Vault automatically update the database password?

No. The credential must also be changed in the database or external system. Both changes must be coordinated.

### Q12. What is a Private Endpoint?

A Private Endpoint provides private IP-based connectivity to Key Vault through Azure Private Link.

### Q13. Does a Private Endpoint provide permission to access secrets?

No. It provides network connectivity. Authentication and authorization are still required.

### Q14. What is soft delete?

Soft delete allows deleted Key Vault objects to be recovered during the configured retention period.

### Q15. What is purge protection?

Purge protection prevents permanent deletion of a soft-deleted object before its retention period expires.

### Q16. How would you troubleshoot an ADF Key Vault access error?

I would verify the ADF managed identity, Key Vault permission model, role assignment or access policy, secret name, linked service configuration, and network connectivity.

### Q17. Why should we avoid hardcoding credentials?

Hardcoded credentials can be exposed through source code, Git repositories, notebook output, logs, or configuration files. Key Vault provides centralized and controlled secret management.

### Q18. What is the difference between Key Vault and App Configuration?

Key Vault manages sensitive values such as passwords and keys. App Configuration manages application settings and feature flags and can reference Key Vault secrets.

---

## 18. Common Mistakes to Avoid

- Assuming Managed Identity automatically grants access to Key Vault.
- Confusing secrets with cryptographic keys.
- Assuming a private endpoint replaces authorization.
- Storing passwords directly in notebooks or source code.
- Assuming secret rotation automatically changes the external database password.
- Confusing Azure RBAC with Key Vault Access Policies.
- Assuming App Configuration and Key Vault have the same purpose.
- Printing secrets during debugging.
- Forgetting to configure monitoring and recovery protections.

---

## 19. Quick Revision

| Concept | Remember |
|---|---|
| Azure Key Vault | Securely manages secrets, keys, and certificates |
| Secret | Password, API key, token, connection string |
| Key | Cryptographic operations |
| Certificate | Digital identity and secure communication |
| Authentication | Verifies identity |
| Authorization | Grants or denies access |
| Managed Identity | Credential-free Azure resource authentication |
| RBAC | Azure role-based permissions |
| Access Policy | Key Vault-specific permission model |
| Secret Versioning | Maintains versions of secrets |
| Secret Rotation | Replaces credentials |
| Private Endpoint | Private network connectivity |
| Soft Delete | Recovery of deleted objects |
| Purge Protection | Prevents premature permanent deletion |
| ADF Integration | Key Vault linked service |
| Databricks Integration | Key Vault-backed secret scope |

---

## 20. Self-Assessment

Try answering these without looking at the notes:

1. What is Azure Key Vault, and why do data engineers use it?
2. Explain secrets, keys, and certificates with examples.
3. How does Managed Identity authenticate to Key Vault?
4. What is the difference between authentication and authorization?
5. Explain RBAC vs Access Policies.
6. How does ADF retrieve a SQL password from Key Vault?
7. How can Databricks access a Key Vault-backed secret scope?
8. What happens when a secret is rotated?
9. What is the difference between a Private Endpoint and authorization?
10. How would you troubleshoot an access-denied error?

**Interview-ready summary:**

Azure Key Vault is used to securely manage secrets, keys, and certificates. In data engineering, it helps services such as Azure Data Factory and Azure Databricks access sensitive credentials without hardcoding them. Azure services can authenticate using Managed Identity, while Key Vault permissions control which secrets or operations they can access. This improves security, centralized management, auditing, and credential rotation.