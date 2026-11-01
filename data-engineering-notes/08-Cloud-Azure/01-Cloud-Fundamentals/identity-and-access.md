# Identity and Access Management (IAM)

## 1. What Is Identity and Access Management?

Identity and Access Management (IAM) is the process of managing **who can access a system, what they can access, and what actions they are allowed to perform**.

In cloud environments, IAM helps organizations securely manage access to:

- Users
- Groups
- Applications
- Virtual machines
- Storage accounts
- Databases
- Data pipelines
- Cloud services

**Example:**

Suppose a data engineering team works with Azure Data Lake Storage Gen2.

- A data engineer needs permission to read and write files.
- A data analyst needs read-only access.
- A pipeline needs permission to access storage without using a personal user account.
- An administrator manages access permissions.

IAM ensures that each identity receives only the permissions required for its job.

---

## 2. Why Is IAM Important?

Without proper access management, unauthorized users or applications could access sensitive data or modify important resources.

IAM helps organizations:

1. Prevent unauthorized access.
2. Protect confidential data.
3. Control who can read, write, or delete resources.
4. Manage employee access.
5. Secure application-to-application communication.
6. Track and audit access.
7. Apply least-privilege access.
8. Meet security and compliance requirements.

For a data engineer, IAM is important because data pipelines frequently access storage accounts, databases, APIs, and other cloud services.

---

## 3. Core IAM Concepts

Four important concepts are:

| Concept | Meaning |
|---|---|
| Identity | Who or what is requesting access |
| Authentication | Verifying the identity |
| Authorization | Determining what the identity can do |
| Auditing | Recording and reviewing access and activities |

### 3.1 Identity

An identity represents a user, application, service, or other entity that needs access to a resource.

Examples:

- A human user.
- A service principal.
- A managed identity.
- An application or workload identity.

### 3.2 Authentication

Authentication verifies that an identity is genuine.

Examples:

- Username and password.
- Multi-factor authentication (MFA).
- Security keys.
- Certificates.
- Access tokens.

**Question:** Who are you?

Authentication answers this question.

### 3.3 Authorization

Authorization determines which actions an authenticated identity is permitted to perform.

Examples:

- Read a file.
- Upload a file.
- Delete a table.
- Start a pipeline.
- Manage a storage account.

**Question:** What are you allowed to do?

Authorization answers this question.

### 3.4 Auditing

Auditing records relevant activities so organizations can investigate access and changes.

Examples:

- User sign-ins.
- Permission changes.
- Resource modifications.
- Data access activities, where supported and configured.

Authentication, authorization, and auditing serve different purposes.

---

## 4. Authentication vs Authorization

This is a common interview question.

| Authentication | Authorization |
|---|---|
| Verifies identity | Determines permissions |
| Happens before access is granted | Evaluates permitted actions |
| Answers: Who are you? | Answers: What can you do? |
| Uses passwords, MFA, certificates, tokens | Uses roles, policies, and permissions |

### Example

A data engineer signs in to Azure using their account and MFA.

1. Azure verifies the user's identity. This is authentication.
2. Azure checks whether the user has permission to access a storage account. This is authorization.
3. The user can read files only if the required permissions are granted.

**Remember:** Successful authentication does not automatically mean access to every resource.

---

## 5. Microsoft Entra ID

Microsoft Entra ID, formerly Azure Active Directory (Azure AD), is Microsoft's cloud identity and access management service.

It helps manage:

- Users.
- Groups.
- Applications.
- Service principals.
- Authentication.
- Single sign-on.
- Conditional Access.
- Identity governance.

### Important capabilities

**User management:** Create and manage employee identities.

**Group management:** Assign access to groups instead of managing every user individually.

**Single Sign-On (SSO):** Allow users to access multiple supported applications using one identity.

**Multi-Factor Authentication (MFA):** Require additional verification beyond a password.

**Conditional Access:** Apply access decisions based on conditions such as user, device, location, and risk.

**Application identities:** Manage identities used by applications to authenticate.

### Example

An organization has 50 data engineers.

Instead of granting storage permissions individually, it can create a Microsoft Entra security group called `Data-Engineers` and assign the required permissions to that group.

When a new engineer joins, adding the user to the group can provide the required access.

Actual access still depends on the assigned roles and other applicable policies.

---

## 6. Important Identity Types in Azure

A data engineer should understand these identity types.

### 6.1 User Identity

A user identity represents a human being.

Example:

A data engineer signs in to Azure Portal using their organization account.

Use cases:
- Accessing Azure Portal.
- Working with Databricks.
- Managing resources.
- Running authorized queries.

### 6.2 Group Identity

A group contains multiple user identities.

Instead of assigning permissions to every user separately, administrators can assign permissions to a group.

Example:

`Data-Engineering-Team`

Members receive the permissions assigned to that group, subject to the applicable authorization system.

### 6.3 Service Principal

A service principal is an application identity in a Microsoft Entra tenant.

It allows an application or automation process to authenticate and access resources without signing in as a human user.

Example:

An automated application needs to access ADLS Gen2.

It can authenticate using a service principal with an appropriately configured credential or certificate.

Typical use cases:
- CI/CD pipelines.
- Automated scripts.
- Application-to-application authentication.
- Scheduled integration workloads.

### 6.4 Managed Identity

A managed identity is an identity managed by Microsoft Entra ID for an Azure resource.

It allows supported Azure services to authenticate to other services without developers managing credentials directly.

Example:

An Azure Data Factory pipeline needs to access ADLS Gen2.

Instead of storing a username and password in code, the pipeline can use its managed identity if the integration and target service support it.

Two common types:

**System-assigned managed identity**
- Created for a specific Azure resource.
- Its lifecycle is tied to that resource.
- Deleted when the associated resource is deleted.

**User-assigned managed identity**
- Created as a separate Azure resource.
- Can be associated with multiple supported resources.
- Has an independent lifecycle.

### Service Principal vs Managed Identity

| Service principal | Managed identity |
|---|---|
| Application identity | Azure-managed identity |
| Credentials may need management | Credentials are managed by Azure |
| Can use secret, certificate, or supported federated credentials | Uses supported managed identity authentication |
| Can be used beyond a single Azure resource | Designed for supported Azure workloads |
| Credential rotation may be required | No application-managed secret rotation for its managed identity credentials |

**Important:** A managed identity is represented through a service principal in Microsoft Entra ID, but the terms are not interchangeable in day-to-day usage.

A managed identity avoids manually managing its authentication credentials. It does not automatically grant access to every Azure resource.

---

## 7. Role-Based Access Control (RBAC)

Role-Based Access Control (RBAC) is an authorization mechanism that assigns permissions through roles.

Instead of giving individual permissions one by one, administrators assign a predefined or custom role to an identity at a particular scope.

### Example

Suppose a data engineer needs to upload and read files in an ADLS Gen2 storage account.

An administrator can assign an appropriate storage data role to the engineer or their group.

### Common Azure RBAC roles

| Role | General purpose |
|---|---|
| Owner | Manage resources and assign access |
| Contributor | Manage resources but cannot grant access through Azure RBAC |
| Reader | View resources without modifying them |
| Storage Blob Data Reader | Read blob data |
| Storage Blob Data Contributor | Read, write, and delete blob data |
| Storage Blob Data Owner | Broad control over blob data, including access control |

The exact permissions of a role should be checked before assigning it.

### Important distinction

Azure management roles and data roles are different.

For example:

- `Reader` generally provides visibility into Azure resource configuration.
- `Storage Blob Data Reader` provides permission to read blob data.

Having permission to view a storage account resource does not automatically mean having permission to read the files inside it.

---

## 8. RBAC Scope

Azure RBAC assignments apply at a defined scope.

Common scopes are:

1. Management group.
2. Subscription.
3. Resource group.
4. Individual resource.

### Example

Suppose an organization has:

```text
Azure Subscription
    |
    +-- Resource Group: Data-RG
            |
            +-- Storage Account: SalesStorage
            |
            +-- Storage Account: CustomerStorage
```

If a role is assigned at the resource group level, it can apply to resources within that resource group through scope inheritance.

If a role is assigned only to `SalesStorage`, it does not automatically apply to `CustomerStorage`.

**Interview point:** Assign access at the narrowest scope that satisfies the requirement.

---

## 9. Azure RBAC vs Microsoft Entra Roles

These two are often confused.

| Azure RBAC | Microsoft Entra roles |
|---|---|
| Controls access to Azure resources | Controls access to identity directory features |
| Examples: Reader, Contributor, Owner | Examples: Global Administrator, User Administrator |
| Scope can be subscription, resource group, or resource | Scope is generally the Entra tenant or supported directory administrative scope |
| Used for resource management and data access through Azure roles | Used for managing users, groups, applications, and directory settings |

Example:

- Managing an Azure storage account: Azure RBAC.
- Creating and managing Entra users: Microsoft Entra directory roles.

Some services also have their own authorization systems, so not every permission is controlled by Azure RBAC alone.

---

## 10. Azure Storage Access: RBAC and ACLs

ADLS Gen2 supports two important authorization mechanisms:

1. Azure RBAC.
2. POSIX-style access control lists (ACLs).

### 10.1 Azure RBAC

RBAC provides access at broader resource scopes, such as a storage account or container.

Example:

Assigning `Storage Blob Data Reader` at the storage account scope can provide read access across the relevant blob data covered by the role assignment.

### 10.2 Access Control Lists (ACLs)

ACLs provide more granular directory and file-level permissions.

Common permissions:

| Permission | Meaning |
|---|---|
| Read (`r`) | Read file contents or list directory contents |
| Write (`w`) | Write or modify file contents; create entries in a directory |
| Execute (`x`) | Traverse a directory |

An ACL can define:
- Owner permissions.
- Owning group permissions.
- Named user permissions.
- Named group permissions.
- Other permissions.

### Example

Suppose the storage layout is:

```text
datalake/
    bronze/
    silver/
    gold/
```

The organization wants:

- Ingestion team: Write access to Bronze.
- Transformation team: Read Bronze and write Silver.
- Analytics team: Read Gold.

ACLs can help implement directory-level access when the relevant storage authorization model and identity configuration support it.

### Important ACL concepts

**Access ACL:** Defines access for the current directory or file.

**Default ACL:** Defines inherited default permissions for newly created children of a directory.

Default ACLs apply to newly created child items; they do not automatically modify existing items.

### RBAC vs ACL

| RBAC | ACL |
|---|---|
| Role-based permissions | File and directory-level permissions |
| Applied at Azure resource scopes | Applied to individual directories and files |
| Suitable for broad access | Suitable for granular access |
| Uses role assignments | Uses read, write, and execute permissions |

### Important ADLS Gen2 point

For an operation to succeed, the identity must have the required permissions under the applicable authorization model.

A broad data-plane RBAC assignment can provide access that makes a more restrictive ACL ineffective for that access path. Evaluate the complete permission configuration instead of assuming ACLs always override RBAC.

---

## 11. Managed Identity in a Data Engineering Pipeline

Consider this scenario:

An Azure Data Factory pipeline needs to copy data from a source into ADLS Gen2.

A secure approach is:

1. Enable the Data Factory managed identity.
2. Identify the required storage permissions.
3. Assign the appropriate storage data role or configure ACLs.
4. Configure the linked service to use managed identity authentication.
5. Run the pipeline.
6. Verify access and review logs.

### Architecture

```text
Azure Data Factory
        |
        | Managed Identity
        v
Microsoft Entra ID
        |
        | Issues authentication token
        v
ADLS Gen2
        |
        | Checks authorization
        v
Read / Write Data
```

Authentication and authorization are separate:

- Managed identity helps the service authenticate.
- RBAC or ACL permissions determine what it can access.

### Why is this better than storing credentials?

- No application-managed password or client secret.
- Reduced risk of accidental credential exposure.
- No manual rotation of the managed identity credentials.
- Easier identity lifecycle management for supported Azure resources.

---

## 12. Tokens and Authentication Basics

After successful authentication, an identity-based system can issue tokens that applications use to access protected resources.

### Access Token

An access token is presented to a resource or service to authorize a request.

Its audience, permissions, and validity must match the target service's requirements.

### Refresh Token

A refresh token can be used by supported authentication flows to obtain new access tokens without requiring the user to sign in again immediately.

Not every authentication flow or identity type uses refresh tokens in the same way.

### Token-based authentication flow

```text
Application
     |
     v
Microsoft Entra ID
     |
     | Authentication
     v
Access Token
     |
     v
Target Azure Service
     |
     | Authorization Check
     v
Access Granted or Denied
```

**Important:** Possessing a valid token does not mean the identity can access every resource. The target service still evaluates authorization.

---

## 13. Security Best Practices

A data engineer should follow these practices when designing pipelines and accessing cloud resources.

### 13.1 Least Privilege

Grant only the permissions required for the task.

Example:

A reporting application that only reads Gold tables should not receive permission to delete Bronze data.

### 13.2 Prefer Managed Identity

Use managed identity for supported Azure-to-Azure authentication scenarios instead of storing secrets in notebooks or scripts.

### 13.3 Use Groups

Assign access to appropriate groups instead of maintaining many individual role assignments.

### 13.4 Enable MFA

Require multi-factor authentication for human users according to organizational security policy.

### 13.5 Use Conditional Access

Apply access rules based on relevant identity, device, location, and risk conditions.

### 13.6 Avoid Hardcoded Credentials

Never store passwords, client secrets, or access keys directly in source code or Git repositories.

### 13.7 Review Permissions

Periodically review access assignments and remove unnecessary permissions.

### 13.8 Audit Access

Use supported logs and monitoring systems to investigate sign-ins, permission changes, and resource activity.

### 13.9 Separate Environments

Use appropriate access boundaries for development, testing, and production.

### 13.10 Protect Privileged Accounts

Limit administrator access and use privileged identity controls where appropriate.

---

## 14. Common Data Engineering Scenarios

| Scenario | Suitable approach |
|---|---|
| ADF needs access to ADLS Gen2 | Managed identity with appropriate storage permissions |
| Databricks needs access to governed cloud storage | Unity Catalog storage credentials and external locations, using supported identity configuration |
| A developer needs to read a storage account | Appropriate storage data role or ACL permissions |
| A pipeline needs application authentication | Managed identity or service principal, depending on the environment |
| A user needs to manage Azure resources | Appropriate Azure RBAC management role |
| A team needs access to multiple resources | Entra group with scoped role assignments |
| An application needs access to a secret | Azure Key Vault with an appropriate identity and permission model |

### Important Databricks connection

In a Databricks environment using Unity Catalog:

- A storage credential defines the authentication mechanism for cloud storage.
- An external location combines a cloud storage path with a storage credential.
- Unity Catalog permissions control which users and groups can use the governed external location.

This is different from simply granting a user a storage account role.

---

## 15. Common Interview Questions

### Q1. What is IAM?

IAM manages identities and controls their access to systems and resources.

### Q2. What is the difference between authentication and authorization?

Authentication verifies identity. Authorization determines which actions that identity is permitted to perform.

### Q3. What is Microsoft Entra ID?

Microsoft Entra ID is Microsoft's cloud identity and access management service for users, groups, applications, and authentication.

### Q4. What is a service principal?

A service principal is an application identity in a Microsoft Entra tenant that allows applications and automation to authenticate and access resources.

### Q5. What is a managed identity?

A managed identity is an Azure-managed identity for a supported resource. It allows the resource to authenticate to other services without the application managing credentials directly.

### Q6. What is the difference between system-assigned and user-assigned managed identity?

- System-assigned: Lifecycle is tied to one Azure resource.
- User-assigned: Separate Azure resource that can be associated with multiple supported resources.

### Q7. What is RBAC?

RBAC is an authorization mechanism that grants permissions through roles assigned to identities at specific scopes.

### Q8. What is the difference between Azure RBAC and Entra roles?

Azure RBAC controls access to Azure resources. Entra roles control administrative access to identity directory features.

### Q9. What is the difference between Reader and Storage Blob Data Reader?

Reader generally allows viewing Azure resource configuration. Storage Blob Data Reader allows reading blob data.

### Q10. What is the principle of least privilege?

Grant only the minimum permissions required to perform a task.

### Q11. What is the difference between RBAC and ACL in ADLS Gen2?

RBAC provides role-based access at broader Azure resource scopes. ACLs provide granular directory and file-level permissions.

### Q12. How can ADF access ADLS Gen2 securely?

Use the Data Factory managed identity, configure the linked service for managed identity authentication, and grant the identity the required storage permissions.

### Q13. Does managed identity automatically provide access to storage?

No. It provides an identity for authentication. The identity must still receive the required authorization permissions.

### Q14. What is MFA?

Multi-Factor Authentication requires additional verification beyond a single authentication factor, such as a password.

### Q15. What is Conditional Access?

Conditional Access applies access decisions based on configured conditions such as identity, device, location, and risk.

### Q16. Why should we avoid hardcoded credentials?

Hardcoded credentials can be exposed through source code, logs, or repositories. Use managed identity or an appropriate secure secret-management mechanism.

### Q17. What happens if a user is authenticated but has no storage permissions?

Authentication can succeed, but the storage service can deny the requested operation because authorization is missing.

---

## 16. Common Mistakes to Avoid

- Treating authentication and authorization as the same concept.
- Assuming managed identity automatically grants access.
- Confusing a service principal with a managed identity.
- Assuming Azure Reader can read files inside a storage account.
- Giving Owner or Contributor roles when data-only access is sufficient.
- Ignoring the scope of an RBAC assignment.
- Assuming ACLs always override RBAC.
- Storing credentials in notebooks or Git repositories.
- Giving every pipeline the same permissions.
- Confusing Azure RBAC with Microsoft Entra directory roles.
- Assuming a valid access token grants access to every resource.

---

## 17. Quick Revision

| Term | Key meaning |
|---|---|
| IAM | Identity and access management |
| Identity | User or application requesting access |
| Authentication | Verifying identity |
| Authorization | Checking permitted actions |
| Auditing | Recording and reviewing activities |
| Microsoft Entra ID | Cloud identity service |
| User | Human identity |
| Group | Collection of identities |
| Service principal | Application identity |
| Managed identity | Azure-managed identity |
| RBAC | Role-based authorization |
| ACL | File and directory-level permissions |
| Least privilege | Minimum required permissions |
| MFA | Additional authentication factor |
| Conditional Access | Access decisions based on conditions |
| Access token | Token used to access a protected resource |
| Scope | Level at which a role assignment applies |

---

## 18. Self-Assessment

Try answering these questions without looking at the notes:

1. What is IAM and why is it important?
2. Explain authentication and authorization with an Azure example.
3. What is Microsoft Entra ID?
4. What is a service principal?
5. What is a managed identity?
6. Explain system-assigned and user-assigned managed identities.
7. What is Azure RBAC?
8. Explain the four common RBAC scopes.
9. What is the difference between Azure RBAC and Entra roles?
10. What is the difference between Reader and Storage Blob Data Reader?
11. Explain RBAC and ACL in ADLS Gen2.
12. How would you securely allow ADF to access ADLS Gen2?
13. Why does managed identity still need permissions?
14. What is the principle of least privilege?
15. How does IAM help secure a data engineering pipeline?

**Interview readiness:** You should be able to explain identity types, authentication versus authorization, RBAC, ADLS Gen2 ACLs, and how a managed identity securely connects Azure data services.