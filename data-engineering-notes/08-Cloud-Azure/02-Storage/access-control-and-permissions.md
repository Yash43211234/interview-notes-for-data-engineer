# Azure Storage Access Control and Permissions

## 1. What Is Access Control?

Access control determines **who can access a resource and what actions they can perform**.

In Azure Storage, access control helps protect data stored in services such as:

- Azure Blob Storage
- Azure Data Lake Storage Gen2
- Azure Files
- Azure Queue Storage
- Azure Table Storage

For a data engineer, access control is important because pipelines, applications, and users need the correct permissions to read, write, update, and manage data.

### Example

Suppose an organization stores data in ADLS Gen2:

```text
Storage Account
    |
    v
Container: sales
    |
    +---- raw/
    |
    +---- processed/
    |
    +---- curated/
```

Different teams may need different permissions:

| User or identity | Required access |
|---|---|
| Data ingestion pipeline | Write to raw |
| Transformation pipeline | Read raw, write processed |
| Analytics team | Read curated |
| Administrator | Manage storage resources |

The goal is to grant only the permissions required for each task.

---

## 2. Main Access Control Mechanisms in Azure Storage

Azure Storage commonly uses these access control mechanisms:

1. Microsoft Entra ID authentication.
2. Azure Role-Based Access Control (Azure RBAC).
3. Access Control Lists (ACLs) for ADLS Gen2.
4. Shared Access Signatures (SAS).
5. Storage account keys.

These mechanisms serve different purposes. They should not be treated as interchangeable.

---

## 3. Microsoft Entra ID Authentication

Microsoft Entra ID (formerly Azure Active Directory) provides identity and authentication services.

It helps verify the identity of:

- Human users.
- Applications.
- Services.
- Managed identities.

### Example

A Databricks job needs to access an ADLS Gen2 storage account.

The job can authenticate using a managed identity or another supported Entra identity mechanism.

After authentication, Azure Storage evaluates the permissions assigned to that identity.

**Important:** Authentication proves who the caller is. It does not automatically grant access to storage data.

---

## 4. Azure RBAC (Role-Based Access Control)

Azure RBAC controls what an identity can do on an Azure resource.

You assign a role to an identity at a particular scope.

### Main Components

- **Security principal:** The user, group, service principal, or managed identity receiving access.
- **Role definition:** The set of permitted actions.
- **Scope:** The resource or resource hierarchy where permissions apply.
- **Role assignment:** Connects the principal, role, and scope.

### Example

A managed identity needs to read files from an ADLS Gen2 storage account.

You can assign:

```text
Principal: Databricks managed identity
Role: Storage Blob Data Reader
Scope: Required storage account or container
```

The identity can then perform the data operations allowed by that role within its scope.

### Common Azure Storage Roles

| Role | Purpose |
|---|---|
| Storage Blob Data Reader | Read and list blob data |
| Storage Blob Data Contributor | Read, write, and delete blob data |
| Storage Blob Data Owner | Broad data access, including managing ownership and ACLs |
| Reader | View Azure resource configuration, not necessarily read stored data |
| Contributor | Manage Azure resources, but does not automatically grant blob data access |
| Owner | Manage resources and assign Azure RBAC roles; data access still depends on applicable data permissions |

**Important interview point:**

`Reader` and `Storage Blob Data Reader` are different roles.

- `Reader` provides management-plane read access to Azure resource configuration.
- `Storage Blob Data Reader` provides data-plane read access to blob data.

Similarly, the general `Contributor` role does not automatically grant access to read or write blob contents.

---

## 5. Management Plane vs Data Plane

This distinction is especially important in Azure Storage.

### Management Plane

The management plane controls the Azure resource itself.

Examples:

- Create a storage account.
- Delete a storage account.
- Change storage account configuration.
- Configure networking settings.
- Manage resource properties.

### Data Plane

The data plane controls the actual data stored inside the resource.

Examples:

- Read a file.
- Upload a file.
- Delete a blob.
- List files in a container.
- Read data from ADLS Gen2.

### Comparison

| Management Plane | Data Plane |
|---|---|
| Manages Azure resources | Accesses stored data |
| Uses management roles | Uses data roles or other data authorization mechanisms |
| Example: Reader | Example: Storage Blob Data Reader |
| Example: Contributor | Example: Storage Blob Data Contributor |

### Interview Scenario

A user has the `Contributor` role on a storage account but cannot read a CSV file inside a container.

Why?

Because Contributor provides management permissions but does not automatically grant blob data access.

The user may need an appropriate data-plane role, such as Storage Blob Data Reader.

---

## 6. Role Scope and Inheritance

Azure RBAC assignments can be made at different scopes.

The main scopes are:

1. Management group.
2. Subscription.
3. Resource group.
4. Individual resource.

A role assignment at a higher scope can be inherited by resources below that scope.

### Example

```text
Subscription
    |
    v
Resource Group
    |
    +---- Storage Account A
    |         |
    |         +---- Container: raw
    |
    +---- Storage Account B
```

If a user receives a suitable role at the resource group scope, that assignment can apply to resources within the resource group.

### Least Privilege

Grant the minimum permissions required to perform a task.

For example:

If a pipeline only needs to read from one container, avoid granting broad access across an entire subscription.

---

## 7. Access Control Lists (ACLs) in ADLS Gen2

Azure Data Lake Storage Gen2 supports hierarchical namespace and POSIX-style access control lists.

ACLs provide more granular permissions for directories and files.

They are especially useful when different teams need access to different folders inside the same storage account.

### Basic ACL Permissions

| Permission | Meaning |
|---|---|
| Read (`r`) | Read file contents or list directory contents |
| Write (`w`) | Modify file contents or create/delete children in a directory, subject to applicable rules |
| Execute (`x`) | Traverse a directory or access a path component |

### Example Directory Structure

```text
sales/
    |
    +---- raw/
    |
    +---- processed/
    |
    +---- curated/
```

Possible access requirements:

| Team | Folder | Permission |
|---|---|---|
| Ingestion team | raw | Read, Write, Execute |
| Transformation team | raw | Read, Execute |
| Transformation team | processed | Read, Write, Execute |
| Analytics team | curated | Read, Execute |

The exact ACLs must also account for directory traversal and inherited permissions.

### File vs Directory Permissions

For a file:

- `r` allows reading file contents.
- `w` allows modifying file contents.
- `x` is not used as a normal file execution permission in the same way as directory traversal.

For a directory:

- `r` allows listing its contents.
- `w` allows creating and deleting entries, subject to the required permissions.
- `x` allows traversing the directory.

**Important:** A user may have permission on a file but still be unable to reach it if they lack execute permission on a parent directory.

---

## 8. ACL Types: Access ACL and Default ACL

ADLS Gen2 supports two important ACL types.

### 8.1 Access ACL

An Access ACL defines permissions on the current file or directory.

Example:

```text
processed/
    Access ACL:
        DataEngineer: rwx
        Analyst: r-x
```

These permissions apply to the current directory.

### 8.2 Default ACL

A Default ACL is available on directories.

It provides inherited permission entries for newly created child files and directories.

Example:

```text
processed/
    Default ACL:
        Analyst: r-x
```

When new children are created, the default ACL can be inherited according to the ACL inheritance rules.

**Important:**

- Default ACLs apply to newly created children.
- They do not automatically update existing files and directories.
- The final permissions of newly created children can also be affected by creation mode and permission inheritance rules.

---

## 9. ACL Permission Evaluation

When accessing ADLS Gen2 data, Azure evaluates the applicable authorization mechanisms.

A simplified ACL evaluation order is:

1. Check whether the caller matches the owning user.
2. Check whether the caller belongs to the owning group.
3. Check the named user entry, if applicable.
4. Check matching named group entries.
5. Check the other entry.

The ACL mask can limit the effective permissions of the owning group, named users, and named groups.

### Example ACL

```text
user::rwx
user:analyst:r-x
group::r-x
mask::r-x
other::---
```

Here:

- The owner has `rwx`.
- The named analyst has `r-x`.
- The owning group has `r-x`.
- The mask limits applicable group-class permissions to `r-x`.
- Other users have no permissions.

**Important:** ACL evaluation is more detailed than simply checking whether a username appears in an ACL. Group membership, the mask, parent-directory traversal, and applicable RBAC permissions all matter.

---

## 10. RBAC vs ACL

This is one of the most important Azure Storage interview topics.

| Azure RBAC | ACL |
|---|---|
| Role-based authorization | Fine-grained path-based authorization |
| Assigned to an identity at a scope | Assigned to users and groups on directories/files |
| Can cover broad storage resources | Can control access to specific paths |
| Example: Storage Blob Data Reader | Example: Read access to `curated/` |
| Useful for broad data access | Useful for folder-level permissions |

### When to Use RBAC

Use RBAC when:

- A pipeline needs broad read/write access.
- A team needs access across a container or storage account.
- You want centralized role assignments.

### When to Use ACL

Use ACL when:

- Different teams need access to different folders.
- Folder-level permissions are required.
- You need more granular access within a data lake.

### Important Interaction Between RBAC and ACL

For ADLS Gen2, Azure RBAC and ACLs work together, but broad data-plane RBAC permissions can grant access independently of restrictive ACL entries.

For example, a principal with `Storage Blob Data Contributor` at the storage account scope has broad data access. Restricting that principal's ACL on one directory does not necessarily remove the access granted by RBAC.

Therefore, use appropriately scoped roles and carefully designed ACLs.

---

## 11. Shared Access Signature (SAS)

A Shared Access Signature provides delegated, time-limited access to storage resources.

It allows a client to access a resource with specified permissions without sharing the storage account key directly.

A SAS can define:

- Allowed permissions.
- Start time.
- Expiry time.
- Allowed protocols.
- Optional IP restrictions.
- Resource scope.

### Example

A company wants to allow an external application to download a report from Blob Storage for a limited period.

It can generate a read-only SAS token with an expiry time.

### Types of SAS

| Type | Meaning |
|---|---|
| User delegation SAS | Signed using a user delegation key obtained through Microsoft Entra ID |
| Service SAS | Grants access to a specific storage service resource |
| Account SAS | Can grant access across supported storage services and resource types in an account |

### SAS Best Practices

- Grant only required permissions.
- Set a short expiry time.
- Use HTTPS.
- Restrict IP addresses when appropriate.
- Avoid sharing SAS tokens publicly.
- Prefer user delegation SAS where supported and suitable.

**Important:** A SAS token is a bearer credential. Anyone who obtains it may use it within its permissions and validity period.

---

## 12. Storage Account Keys

Storage account keys provide broad access to the storage account through Shared Key authorization.

A storage account generally has two keys:

- Key 1
- Key 2

These keys support key rotation.

### Example

An application authenticates to Azure Storage using a connection string containing the account name and key.

This approach is powerful but creates security risks if keys are exposed.

### Best Practices

- Avoid hardcoding account keys in source code.
- Store secrets in Azure Key Vault when Shared Key authentication is necessary.
- Rotate keys periodically and after suspected exposure.
- Prefer Microsoft Entra ID and managed identities when supported.
- Consider disabling Shared Key authorization if the workload does not require it and the environment supports that configuration.

---

## 13. Managed Identity for Storage Access

Managed Identity allows Azure resources to authenticate to supported services without storing credentials in application code.

Types:

1. System-assigned managed identity.
2. User-assigned managed identity.

### System-Assigned Managed Identity

- Created for a particular Azure resource.
- Its lifecycle is tied to that resource.
- Deleting the resource also removes its identity.

### User-Assigned Managed Identity

- Created as a separate Azure resource.
- Can be assigned to multiple supported resources.
- Has an independent lifecycle.

### Example: Data Factory Accessing ADLS Gen2

```text
Azure Data Factory
        |
        v
System-Assigned Managed Identity
        |
        v
Microsoft Entra ID
        |
        v
Azure Storage Authorization
        |
        v
ADLS Gen2
```

A suitable role or ACL permission must be assigned to the managed identity.

**Important:** Managed Identity handles authentication. It does not automatically provide storage permissions.

---

## 14. Access Control Example for a Data Lake

Suppose a company has this ADLS Gen2 structure:

```text
company-data/
    |
    +---- raw/
    |
    +---- processed/
    |
    +---- curated/
```

The company has three teams.

| Team | Raw | Processed | Curated |
|---|---|---|---|
| Ingestion | Write | No access | No access |
| Engineering | Read | Read/Write | Read |
| Analytics | No access | No access | Read |

### Possible Design

- Use managed identities for pipelines and applications.
- Use appropriately scoped Azure RBAC roles for broad access.
- Use ACLs for folder-level permissions where required.
- Give analytics users read-only access to curated data.
- Avoid granting unnecessary access to raw or processed data.
- Review group membership and role assignments regularly.

This design should be tested with the actual identities and access paths. Effective permissions can depend on RBAC scope, ACL inheritance, group membership, and networking restrictions.

---

## 15. Common Access Errors

### 15.1 AuthorizationPermissionMismatch

The caller is authenticated but lacks the required permission for the requested operation.

Possible reasons:

- Missing data-plane role.
- Insufficient ACL permissions.
- Missing execute permission on a parent directory.
- Incorrect scope.
- Role assignment has not yet propagated.

### 15.2 AuthenticationFailed

Authentication failed.

Possible reasons:

- Invalid credentials.
- Expired or invalid SAS token.
- Incorrect account key.
- Invalid token or identity configuration.

### 15.3 ResourceNotFound

The requested resource does not exist or cannot be found at the specified path.

Possible reasons:

- Incorrect storage account.
- Wrong container or filesystem.
- Incorrect directory or file path.
- Resource was deleted or renamed.

### 15.4 Network Access Denied

The identity may have valid permissions, but the storage account's network configuration blocks the request.

Possible reasons:

- Firewall restrictions.
- Public network access disabled.
- Private endpoint or DNS misconfiguration.
- Missing VNet or network access configuration.

### Troubleshooting Checklist

When a pipeline cannot access ADLS Gen2:

1. Confirm the storage account and file path.
2. Verify which identity the pipeline is using.
3. Confirm that authentication succeeds.
4. Check the required data-plane RBAC role.
5. Inspect ACLs on the target path and parent directories.
6. Check group membership and ACL masks where applicable.
7. Check role assignment scope.
8. Check storage firewall and network settings.
9. Allow for role-assignment propagation.
10. Test again using the same identity as the pipeline.

**Interview point:** Authentication, authorization, and network connectivity are separate checks. A failure in any one of them can prevent access.

---

## 16. Security Best Practices

1. Follow the principle of least privilege.
2. Prefer managed identities over stored credentials.
3. Use Microsoft Entra ID authentication where supported.
4. Avoid sharing storage account keys.
5. Use short-lived SAS tokens with limited permissions.
6. Assign roles at the narrowest practical scope.
7. Use ACLs for granular ADLS Gen2 folder permissions.
8. Review role assignments and ACLs regularly.
9. Use private endpoints and appropriate network restrictions when required.
10. Store unavoidable secrets in Azure Key Vault.
11. Monitor storage access and authorization failures.
12. Remove access when users or applications no longer need it.

---

## 17. Common Interview Questions

### Q1. What is Azure Storage access control?

It determines which identities can access storage resources and what operations they can perform.

### Q2. What is the difference between authentication and authorization?

- Authentication verifies identity.
- Authorization determines what that identity can access or do.

### Q3. What is Azure RBAC?

Azure RBAC is a role-based authorization system that assigns permissions to identities at defined scopes.

### Q4. What is the difference between Reader and Storage Blob Data Reader?

Reader provides management-plane read access to Azure resource configuration. Storage Blob Data Reader provides read access to blob data.

### Q5. What is the difference between RBAC and ACL?

RBAC provides role-based access at Azure resource scopes. ACLs provide granular permissions on ADLS Gen2 directories and files.

### Q6. When would you use ACLs instead of broad RBAC roles?

When different users or teams require different permissions on specific directories or files.

### Q7. What are the three basic ACL permissions?

Read (`r`), Write (`w`), and Execute (`x`).

### Q8. What is the difference between Access ACL and Default ACL?

Access ACL applies to the current file or directory. Default ACL is defined on a directory and provides inherited permissions for newly created children.

### Q9. Does a Default ACL update existing files?

No. It applies to newly created children, not existing files and directories.

### Q10. What is the purpose of the ACL mask?

It limits the effective permissions of the owning group, named users, and named groups in the ACL.

### Q11. What is a SAS token?

A SAS token provides delegated access to storage resources with specified permissions and validity constraints.

### Q12. What is the difference between a SAS token and a storage account key?

A SAS grants limited access according to its scope and permissions. An account key provides broad Shared Key access to the storage account.

### Q13. What is Managed Identity?

Managed Identity is an identity provided for an Azure resource so it can authenticate to supported services without storing credentials in code.

### Q14. Does Managed Identity automatically provide access to ADLS Gen2?

No. It must be granted suitable permissions through RBAC, ACLs, or another supported authorization mechanism.

### Q15. A pipeline can authenticate successfully but cannot read a file. What would you check?

I would verify the data-plane role, ACL permissions, parent-directory execute permissions, role scope, and storage network restrictions.

### Q16. Why might a user with Storage Blob Data Contributor still have broad access despite restrictive ACLs?

Because broad data-plane RBAC permissions can independently grant data access. Restrictive ACL entries do not necessarily override that access.

### Q17. How would you secure an Azure Data Factory pipeline accessing ADLS Gen2?

I would use a managed identity, assign the minimum required data permissions, configure the storage network access appropriately, and avoid hardcoded credentials.

---

## 18. Quick Revision

| Topic | Key Point |
|---|---|
| Authentication | Verifies identity |
| Authorization | Determines permissions |
| Microsoft Entra ID | Identity and authentication platform |
| Azure RBAC | Role-based access at resource scopes |
| ACL | Fine-grained file and directory permissions |
| Management Plane | Manages Azure resources |
| Data Plane | Reads and modifies stored data |
| Reader | Management-plane read access |
| Storage Blob Data Reader | Reads blob data |
| Storage Blob Data Contributor | Reads, writes, and deletes blob data |
| Access ACL | Permissions on current file/directory |
| Default ACL | Inherited by newly created children |
| `r` | Read |
| `w` | Write |
| `x` | Execute/traverse |
| ACL Mask | Limits effective group-class permissions |
| SAS | Delegated, time-limited access |
| Storage Account Key | Broad Shared Key credential |
| Managed Identity | Azure-managed identity for authentication |
| Least Privilege | Grant only required permissions |

---

## 19. Self-Assessment

Try answering these questions without looking at the notes:

1. Explain Azure Storage access control.
2. Differentiate authentication and authorization.
3. Explain management plane vs data plane.
4. What is Azure RBAC?
5. Explain the components of an RBAC role assignment.
6. Differentiate Reader and Storage Blob Data Reader.
7. What is the purpose of ACLs in ADLS Gen2?
8. Explain read, write, and execute permissions.
9. Differentiate Access ACL and Default ACL.
10. What is an ACL mask?
11. Explain RBAC vs ACL.
12. What is SAS and what are its types?
13. Differentiate SAS and storage account keys.
14. Explain system-assigned vs user-assigned managed identity.
15. Why does a managed identity need separate storage permissions?
16. How would you troubleshoot an ADLS Gen2 authorization failure?
17. How would you design folder-level access for raw, processed, and curated data?

**Interview readiness:** You should be able to explain RBAC, ACLs, SAS, managed identities, management-plane vs data-plane access, and how to troubleshoot a pipeline that cannot access ADLS Gen2.