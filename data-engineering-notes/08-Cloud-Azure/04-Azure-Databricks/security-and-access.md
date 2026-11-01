# Azure Databricks Security and Access Management

## 1. What Is Security and Access Management?

Security and access management in Azure Databricks controls who can access the workspace, use compute resources, and read or modify data.

A secure Databricks environment must protect:

- Workspace resources.
- Notebooks and jobs.
- Compute resources.
- Storage accounts and data files.
- Tables and other governed data assets.
- Credentials, secrets, and service identities.

Security is not controlled by one permission system. It involves identity, authentication, authorization, networking, compute, and data governance.

## 2. Security Architecture

A typical security architecture includes the following layers:

```text
                Users / Applications
                         |
                         v
                Microsoft Entra ID
                  Authentication
                         |
                         v
                Databricks Workspace
                  Workspace Access
                         |
                         v
                  Compute Resources
                  Compute Permissions
                         |
                         v
                   Unity Catalog
                    Data Access
                         |
                         v
                    ADLS Gen2
                 Storage Permissions
                         |
                         v
                 Data and Delta Tables
```

Other security components, such as Azure Key Vault, private networking, and audit logs, support the architecture.

## 3. Authentication vs Authorization

These are two fundamental security concepts.

### Authentication

Authentication verifies the identity of a user or application.

Examples:
- Username and password.
- Single sign-on (SSO).
- Microsoft Entra ID authentication.
- Service principal authentication.
- Managed identity authentication.

### Authorization

Authorization determines what an authenticated identity is allowed to do.

Examples:
- Open a workspace.
- Run a notebook.
- Use a compute resource.
- Read a table.
- Create or modify a table.
- Access a storage location.

### Difference

| Authentication | Authorization |
|---|---|
| Verifies identity | Checks permissions |
| Answers: Who are you? | Answers: What can you do? |
| Login and identity verification | Roles, privileges, and permissions |
| Usually happens before authorization | Evaluated when accessing resources |

**Interview example:** A user may successfully log in to Databricks but still receive a permission error when trying to read a protected table.

## 4. Microsoft Entra ID Integration

Microsoft Entra ID, formerly Azure Active Directory, provides identity and authentication services.

Databricks can integrate with Entra ID to manage user access.

Common identity types include:

| Identity | Purpose |
|---|---|
| User | Individual employee or developer |
| Group | Collection of users |
| Service principal | Application identity |
| Managed identity | Azure-managed identity for supported resources |

### Why use groups?

Instead of granting permissions individually to every user, organizations can assign permissions to groups.

Example:

```text
Data Engineering Team
        |
        +-- Developer A
        +-- Developer B
        +-- Developer C
```

The group can receive the permissions needed for data engineering work.

Benefits:
- Easier administration.
- Consistent access.
- Simpler onboarding and offboarding.
- Reduced permission-management effort.

## 5. Databricks Workspace Access Control

Workspace access control determines who can access workspace resources and perform workspace-level operations.

Examples of protected resources include:
- Notebooks.
- Folders.
- Jobs and workflows.
- Compute resources.
- Repositories.
- SQL-related workspace objects.

### Common workspace permissions

The exact permission names and available levels depend on the resource type.

Typical examples include:

| Permission concept | Purpose |
|---|---|
| Read | View an object |
| Run | Execute an object |
| Edit | Modify an object |
| Manage | Manage permissions or configuration |

For example, a developer may be allowed to run a notebook but not modify it.

### Workspace access vs data access

Workspace access does not automatically grant access to all underlying data.

A user may:
- Open the workspace.
- Open a notebook.
- Attach to permitted compute.

But still be unable to read a protected ADLS Gen2 path or Unity Catalog table.

These are separate authorization layers.

## 6. Compute Access Control

Compute access control determines who can use or manage compute resources.

Depending on the compute type and configuration, permissions may allow users to:
- Attach to compute.
- Use compute.
- Restart compute.
- Edit compute settings.
- Manage compute permissions.

### Example

Suppose a company has:

| User | Access |
|---|---|
| Developer A | Can use development compute |
| Developer B | Can use development compute |
| Platform Admin | Can manage compute configuration |

This separation helps prevent unauthorized infrastructure changes.

**Important:** Permission behavior differs across classic compute, SQL warehouses, and serverless compute. Do not assume that one compute permission model applies identically to every type.

## 7. Unity Catalog: Centralized Data Governance

Unity Catalog is Databricks' centralized governance solution for data and AI assets.

It provides:
- Data discovery.
- Access control.
- Centralized permissions.
- Data lineage.
- Auditing capabilities.
- Governance across supported workspaces.

### Unity Catalog hierarchy

```text
Metastore
    |
    v
  Catalog
    |
    v
  Schema
    |
    +----------+
    |          |
    v          v
  Table      View
```

The hierarchy is:

1. Metastore
2. Catalog
3. Schema
4. Table or View

Unity Catalog also governs other securable objects, including volumes, functions, and external locations.

### Example

Suppose an organization has:

```text
Metastore
   |
   +-- sales_catalog
          |
          +-- bronze
          |     |
          |     +-- raw_orders
          |
          +-- silver
          |     |
          |     +-- clean_orders
          |
          +-- gold
                |
                +-- customer_sales
```

Permissions can be assigned at different levels depending on the access requirement.

## 8. Unity Catalog Privileges

Unity Catalog uses privileges to control access to governed objects.

Common privileges include:

| Privilege | Purpose |
|---|---|
| USE CATALOG | Allows use of a catalog |
| USE SCHEMA | Allows use of a schema |
| SELECT | Read data from a table or view |
| MODIFY | Modify table data |
| CREATE TABLE | Create tables in a schema |
| CREATE SCHEMA | Create schemas in a catalog |
| CREATE VIEW | Create views in a schema |
| READ FILES | Read files through an external location |
| WRITE FILES | Write files through an external location |

The exact privileges depend on the securable object.

### Example: Grant table access

```sql
GRANT USE CATALOG
ON CATALOG sales_catalog
TO `data_analysts`;

GRANT USE SCHEMA
ON SCHEMA sales_catalog.gold
TO `data_analysts`;

GRANT SELECT
ON TABLE sales_catalog.gold.customer_sales
TO `data_analysts`;
```

These permissions allow the group to use the catalog and schema and read the specified table.

### Example: Grant data engineering permissions

```sql
GRANT USE CATALOG
ON CATALOG sales_catalog
TO `data_engineers`;

GRANT USE SCHEMA
ON SCHEMA sales_catalog.silver
TO `data_engineers`;

GRANT SELECT, MODIFY
ON TABLE sales_catalog.silver.clean_orders
TO `data_engineers`;
```

The executing principal must have the necessary authority to grant these privileges.

**Important:** `SELECT` on a table alone may not be sufficient. The principal generally also needs `USE CATALOG` and `USE SCHEMA` on the parent objects.

## 9. Storage Credentials and External Locations

Unity Catalog uses storage credentials and external locations to govern access to cloud storage.

### Storage Credential

A storage credential represents the authentication mechanism used to access cloud storage.

It can use supported identity mechanisms such as:
- Azure managed identity.
- Azure service principal.

### External Location

An external location connects:
- A cloud storage path.
- A storage credential.

Example:

```text
ADLS Gen2
    |
    v
Storage Credential
    |
    v
External Location
    |
    v
Unity Catalog Permissions
```

### Example SQL

```sql
CREATE EXTERNAL LOCATION bronze_location
URL 'abfss://bronze@stdataeng001.dfs.core.windows.net/'
WITH (STORAGE CREDENTIAL bronze_credential);
```

Grant permission:

```sql
GRANT READ FILES
ON EXTERNAL LOCATION bronze_location
TO `data_engineers`;
```

This example assumes that the storage credential already exists and that the executing principal has the required privileges.

### External Location vs External Table

| External Location | External Table |
|---|---|
| Governs a cloud storage path | Represents a table stored at an external location |
| Uses a storage credential | Uses a table definition and storage location |
| Controls file access through supported privileges | Provides table-level data access |
| Not itself a table | A governed table object |

## 10. ADLS Gen2 Access Control

Databricks may access data stored in ADLS Gen2.

Storage authorization can involve:
- Azure RBAC data-plane roles.
- ADLS Gen2 ACLs.
- Unity Catalog storage credentials and external locations.

### Common RBAC roles

| Role | Purpose |
|---|---|
| Storage Blob Data Reader | Read blob data |
| Storage Blob Data Contributor | Read, write, and delete blob data |
| Storage Blob Data Owner | Full data access, including ownership and ACL management |

A general Azure `Reader` role does not automatically grant permission to read blob contents.

### ADLS Gen2 ACL permissions

| Permission | Meaning |
|---|---|
| Read (`r`) | Read file contents or list directory contents |
| Write (`w`) | Write or modify data |
| Execute (`x`) | Traverse directories |

For example, reading a file can require:
- Execute permission on parent directories.
- Read permission on the file.

Default ACLs apply to newly created children. They do not automatically update existing files and directories.

### RBAC vs ACL

| RBAC | ACL |
|---|---|
| Role-based access | File and directory-level access |
| Applied at Azure resource scopes | Applied to filesystem paths |
| Useful for broader access | Useful for fine-grained permissions |
| Uses Azure role assignments | Uses POSIX-style permission entries |

**Important:** Broad data-plane RBAC permissions can grant access beyond restrictive ACL entries. Understand the authorization model before relying on ACLs to restrict an identity.

## 11. Secret Management with Azure Key Vault

Applications sometimes need credentials to access external systems.

Examples:
- Database passwords.
- API keys.
- Service principal secrets.
- Third-party application credentials.

Hardcoding secrets in notebooks is unsafe.

Azure Key Vault helps store and manage sensitive information.

### Basic architecture

```text
Azure Key Vault
       |
       v
Secret Management
       |
       v
Databricks
       |
       v
Notebook / Job
       |
       v
External Service
```

Databricks can integrate with supported secret-management mechanisms to retrieve credentials securely.

### Example using Databricks secrets

```python
client_secret = dbutils.secrets.get(
    scope="production-secrets",
    key="storage-client-secret"
)
```

This example assumes the secret scope and key already exist and that the user has permission to access them.

**Best practices:**
- Never print secrets in notebook output.
- Do not commit secrets to Git.
- Restrict secret-scope permissions.
- Rotate credentials.
- Prefer managed identities or other secretless authentication where supported.

## 12. Managed Identity and Service Principal

Both can be used for application authentication, but their lifecycle and management differ.

### Service Principal

A service principal represents an application identity in Microsoft Entra ID.

It can authenticate using:
- Client secret.
- Certificate.
- Other supported credential mechanisms.

### Managed Identity

A managed identity is an identity managed by Azure for supported resources.

It avoids directly managing application secrets.

### Comparison

| Feature | Service Principal | Managed Identity |
|---|---|---|
| Identity | Application identity | Azure-managed identity |
| Secret management | May require secret or certificate | No application secret to manage |
| Lifecycle | Managed independently | System-assigned or user-assigned lifecycle |
| Typical use | Applications and automation | Supported Azure resource authentication |

A managed identity is represented by a service principal in Microsoft Entra ID.

**Important:** Neither identity automatically has permission to access ADLS Gen2. Appropriate storage authorization must still be granted.

## 13. Network Security

Security also requires controlling how Databricks connects to Azure resources.

Common networking controls include:

- Virtual networks.
- Network security groups.
- Private endpoints.
- Private DNS.
- Firewalls.
- Secure connectivity configurations.

### Private Endpoint

A private endpoint provides a private IP address in a virtual network for a supported Azure service through Azure Private Link.

For example, Databricks compute can use private connectivity to reach an ADLS Gen2 storage account when the architecture is configured correctly.

### Important distinction

A private endpoint:
- Provides private network connectivity.
- Does not authenticate the user.
- Does not grant storage permissions.

Authentication, authorization, and network connectivity are separate security requirements.

## 14. Data Protection and Governance

A production Databricks environment should protect data throughout its lifecycle.

Important practices include:

### Data Access

- Use least-privilege permissions.
- Grant access through groups where appropriate.
- Restrict sensitive tables.
- Review permissions regularly.

### Data Governance

- Use Unity Catalog.
- Maintain data ownership.
- Track data lineage.
- Audit access and changes.

### Data Protection

- Avoid exposing sensitive information in notebook output.
- Mask or filter sensitive data when required.
- Use approved encryption and network security configurations.
- Follow organizational retention and compliance policies.

### Development and Production Separation

Keep development and production access separate.

For example:

| Environment | Typical access |
|---|---|
| Development | Developers can build and test |
| Testing | Approved users validate changes |
| Production | Restricted execution and administration |

Use controlled deployment processes to move code between environments.

## 15. Common Security Problems

| Problem | Possible cause | What to check |
|---|---|---|
| Cannot access workspace | User not provisioned or workspace access missing | Entra identity and workspace permissions |
| Cannot attach to compute | Missing compute permission | Compute access control |
| Cannot read table | Missing Unity Catalog privileges | Catalog, schema, and table grants |
| 403 from ADLS | Missing storage authorization | RBAC and ACL |
| Secret access denied | Missing secret-scope permission | Secret scope access control |
| Authentication failure | Invalid identity or credential | Entra configuration |
| Private endpoint connection failure | DNS or network configuration | Private DNS, routing, firewall |
| Can open notebook but cannot read data | Workspace access exists, data access is missing | Unity Catalog or storage permissions |

### Troubleshooting sequence

1. Confirm the correct user or application identity.
2. Check workspace access.
3. Check compute permissions.
4. Check Unity Catalog permissions.
5. Check storage credentials and external locations.
6. Verify ADLS Gen2 RBAC and ACL permissions.
7. Check network connectivity.
8. Review audit records and error messages.

## 16. Best Practices

1. Use Microsoft Entra ID for identity management.
2. Use groups to manage permissions consistently.
3. Apply least-privilege access.
4. Prefer Unity Catalog for centralized data governance.
5. Use managed identities or supported secretless authentication where possible.
6. Avoid hardcoding credentials in notebooks.
7. Store required secrets securely.
8. Separate workspace, compute, and data permissions.
9. Use private connectivity when required by the security architecture.
10. Review permissions regularly.
11. Enable appropriate auditing and monitoring.
12. Separate development and production environments.
13. Protect sensitive data in notebooks and logs.
14. Use controlled processes for granting and revoking access.

## 17. Practical Interview Scenario

**Question:** A data engineer can log in to Azure Databricks and run a notebook, but receives a 403 Forbidden error while reading an ADLS Gen2 file. How would you troubleshoot it?

### Step 1: Verify authentication

Confirm which identity the Databricks workload uses:
- Managed identity.
- Service principal.
- Unity Catalog storage credential.

### Step 2: Verify storage authorization

Check whether the identity has the required data-plane RBAC role or ACL permissions.

### Step 3: Check ACL traversal

For ADLS Gen2, verify execute permission on parent directories and read permission on the target file, where ACL-based access applies.

### Step 4: Check Unity Catalog

If the access is governed through Unity Catalog, verify:
- Storage credential configuration.
- External location configuration.
- Required privileges.
- Catalog and schema permissions if accessing a table.

### Step 5: Check networking

Verify firewall rules, private endpoint connectivity, private DNS, and routing.

### Step 6: Review logs

Inspect the error details and relevant Azure or Databricks audit logs.

**Key lesson:** Being able to log in to a Databricks workspace does not mean the same identity has permission to read every external data source.

## 18. Common Interview Questions and Answers

### Q1. What is the difference between authentication and authorization?

Authentication verifies identity. Authorization determines which resources and operations that identity is permitted to access.

### Q2. What is Unity Catalog?

Unity Catalog is Databricks' centralized governance solution for managing data access, permissions, discovery, and lineage.

### Q3. What is the Unity Catalog hierarchy?

The main hierarchy is Metastore → Catalog → Schema → Table or View.

### Q4. What is the difference between workspace permissions and Unity Catalog permissions?

Workspace permissions control access to workspace resources such as notebooks, jobs, and compute. Unity Catalog permissions control access to governed data assets.

### Q5. What is a storage credential?

A storage credential represents the authentication mechanism used by Unity Catalog to access cloud storage.

### Q6. What is an external location?

An external location connects a cloud storage path with a storage credential and governs access to that path.

### Q7. What is the difference between RBAC and ACL in ADLS Gen2?

RBAC provides role-based permissions at Azure resource scopes. ACLs provide fine-grained file and directory permissions.

### Q8. Does a managed identity automatically get access to ADLS Gen2?

No. It must be granted the appropriate data-plane RBAC role or ACL permissions.

### Q9. Why should secrets not be hardcoded in notebooks?

Hardcoded secrets can be exposed through source code, notebook output, logs, or Git repositories. Secure secret management reduces this risk.

### Q10. What is Azure Key Vault?

Azure Key Vault is a service for securely storing and managing secrets, keys, and certificates.

### Q11. What is a private endpoint?

A private endpoint provides private network connectivity to a supported Azure service through a private IP address in a virtual network.

### Q12. Does a private endpoint grant access to storage data?

No. It provides network connectivity. Authentication and authorization are still required.

### Q13. What is least privilege?

Least privilege means giving users and applications only the permissions necessary to perform their responsibilities.

### Q14. How would you secure a production Databricks workspace?

I would integrate identity management with Microsoft Entra ID, use group-based access, apply least-privilege workspace and compute permissions, govern data through Unity Catalog, secure storage credentials, use private networking where required, and monitor access through audit logs.

### Q15. Why can a user open a notebook but fail to read a table?

Notebook access and data access are separate. The user may have workspace permissions but lack the required Unity Catalog or underlying storage permissions.

## 19. Quick Revision

| Concept | Key point |
|---|---|
| Authentication | Verifies identity |
| Authorization | Determines allowed actions |
| Microsoft Entra ID | Identity and authentication |
| Workspace access control | Controls workspace resources |
| Compute access control | Controls compute usage and management |
| Unity Catalog | Centralized data governance |
| Metastore | Top-level Unity Catalog container |
| Catalog | Organizes schemas |
| Schema | Organizes tables and other objects |
| Storage credential | Defines cloud authentication |
| External location | Governed cloud storage path |
| RBAC | Role-based Azure permissions |
| ACL | File and directory permissions |
| Managed identity | Azure-managed identity |
| Service principal | Application identity |
| Azure Key Vault | Secure secret and key management |
| Private endpoint | Private network connectivity |
| Least privilege | Minimum required permissions |

## 20. Self-Assessment

Try answering these questions without looking at the notes:

1. Explain authentication and authorization.
2. How does Microsoft Entra ID integrate with Databricks?
3. Differentiate workspace access and data access.
4. Explain Unity Catalog hierarchy.
5. What are storage credentials and external locations?
6. Compare Azure RBAC and ADLS Gen2 ACLs.
7. Differentiate managed identity and service principal.
8. How would you securely manage credentials?
9. What is the purpose of a private endpoint?
10. How would you troubleshoot a 403 error from ADLS Gen2?
11. Why is least-privilege access important?
12. Explain how you would secure a production Databricks environment.

**Interview readiness target:** You should be able to explain how identity, workspace permissions, compute permissions, Unity Catalog, ADLS Gen2 authorization, and network security work together to protect a production data engineering platform.