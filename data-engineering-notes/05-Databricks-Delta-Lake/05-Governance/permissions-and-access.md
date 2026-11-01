# Permissions and Access Control in Databricks

## 1. What Is Access Control?

Access control is the process of deciding who can access data and what actions they are allowed to perform.

In a data engineering environment, not every user should have permission to read, modify, or delete every table.

For example:

- Data engineers may need to create and modify tables.
- Data analysts may need permission to read Gold tables.
- Business users may only need access to reporting data.
- Administrators manage workspace and governance settings.

**Simple way to remember:**

Authentication = Who are you?

Authorization = What are you allowed to do?

## 2. Why Do We Need Permissions?

Suppose an organization stores customer information in Databricks.

Different users need different levels of access.

| User | Requirement |
|---|---|
| Data Engineer | Read Bronze and Silver; create Gold tables |
| Data Analyst | Read Silver and Gold |
| Business User | Read approved Gold tables |
| Administrator | Manage workspace and governance |

Without access control:

- Unauthorized users may view sensitive data.
- Users may accidentally modify production tables.
- Important data may be deleted.
- Organizations may fail to meet security and compliance requirements.

### Main Benefits

1. Protect sensitive data.
2. Prevent unauthorized changes.
3. Apply least-privilege access.
4. Separate responsibilities between teams.
5. Support auditing and compliance.
6. Manage access centrally.

## 3. Unity Catalog and Access Control

**Unity Catalog** is Databricks' centralized governance solution for managing data and AI assets.

It provides capabilities such as:

- Access control.
- Data discovery.
- Data lineage.
- Auditing.
- Centralized permissions management.
- Governance across supported workspaces.

Unity Catalog uses a hierarchy to organize data assets.

### Unity Catalog Hierarchy

```text
          METASTORE
               |
               v
            CATALOG
               |
               v
             SCHEMA
               |
               v
       TABLE / VIEW / VOLUME
```

Example:

```text
Metastore
   |
   v
main
   |
   +-- bronze
   |     |
   |     +-- orders
   |
   +-- silver
   |     |
   |     +-- orders
   |
   +-- gold
         |
         +-- daily_sales
```

Here:

- `main` is the catalog.
- `bronze`, `silver`, and `gold` are schemas.
- `orders` and `daily_sales` are tables.

The fully qualified table name is:

```sql
main.gold.daily_sales
```

This hierarchy allows permissions to be managed at different levels.

## 4. Types of Permissions in Unity Catalog

Unity Catalog uses privileges to control what users and groups can do with securable objects.

Common privileges include:

| Privilege | Purpose |
|---|---|
| `USE CATALOG` | Allows access to a catalog context |
| `USE SCHEMA` | Allows access to a schema context |
| `SELECT` | Read data from a table or view |
| `MODIFY` | Modify table data |
| `CREATE TABLE` | Create tables in a schema |
| `CREATE SCHEMA` | Create schemas in a catalog |
| `CREATE VIEW` | Create views in a schema |
| `READ VOLUME` | Read files from a volume |
| `WRITE VOLUME` | Write files to a volume |
| `MANAGE` | Manage permissions and ownership where supported |

**Important:** The exact privileges available depend on the securable object and platform capabilities.

## 5. Understanding USE CATALOG and USE SCHEMA

A user generally needs the appropriate usage privileges to access objects within a catalog and schema.

For example, granting `SELECT` on a table alone is not necessarily sufficient.

The user also needs access to the parent catalog and schema.

### Example

Suppose an analyst needs to read:

```sql
main.gold.daily_sales
```

The analyst needs:

1. `USE CATALOG` on `main`.
2. `USE SCHEMA` on `main.gold`.
3. `SELECT` on `main.gold.daily_sales`.

This allows the analyst to access the table without granting permission to modify it.

## 6. GRANT and REVOKE Commands

Databricks SQL supports `GRANT` and `REVOKE` for managing privileges.

### 6.1 GRANT

`GRANT` provides a privilege to a user, group, or service principal.

Example:

```sql
GRANT USE CATALOG
ON CATALOG main
TO `data_analysts`;
```

Grant schema access:

```sql
GRANT USE SCHEMA
ON SCHEMA main.gold
TO `data_analysts`;
```

Grant read access:

```sql
GRANT SELECT
ON TABLE main.gold.daily_sales
TO `data_analysts`;
```

Now the group can read the specified table, subject to the required parent permissions and other access conditions.

### 6.2 REVOKE

`REVOKE` removes a previously granted privilege.

Example:

```sql
REVOKE SELECT
ON TABLE main.gold.daily_sales
FROM `data_analysts`;
```

This removes the group's direct SELECT grant on that table.

**Important:** A user may still have access through another group, inherited privilege, ownership, or another grant. Revoking one grant does not necessarily remove every possible access path.

## 7. GRANT vs REVOKE

| Command | Purpose |
|---|---|
| `GRANT` | Provides a privilege |
| `REVOKE` | Removes a privilege |

Example:

```sql
-- Give read access
GRANT SELECT
ON TABLE main.gold.daily_sales
TO `finance_team`;

-- Remove read access
REVOKE SELECT
ON TABLE main.gold.daily_sales
FROM `finance_team`;
```

## 8. Understanding Inheritance

Unity Catalog supports privilege inheritance through its securable hierarchy.

For example, a privilege granted at a catalog or schema level may apply to supported child objects.

Conceptually:

```text
           CATALOG
              |
              v
            SCHEMA
              |
              v
            TABLE
```

Suppose a group receives appropriate read privileges at a schema level.

Those privileges may apply to current and future child tables, depending on the privilege and object type.

### Why Is Inheritance Useful?

- Reduces repetitive permission management.
- Makes team-level access easier.
- Helps manage permissions consistently.
- Supports scalable governance.

**Important:** Do not assume every privilege automatically applies to every child object. Check the specific privilege's inheritance behavior.

## 9. Users, Groups, and Service Principals

Databricks permissions can be assigned to different identities.

### 9.1 Users

Individual people who access the Databricks workspace.

Example:

```text
analyst@example.com
```

### 9.2 Groups

Groups contain multiple users and make permission management easier.

Example:

```text
data_engineers
data_analysts
finance_team
```

Instead of granting access individually, administrators can assign privileges to a group.

### 9.3 Service Principals

A service principal represents an application or automated workload.

It is commonly used for:

- Automated jobs.
- CI/CD pipelines.
- Application access.
- Machine-to-machine authentication.

**Best practice:** Prefer group-based permissions for teams and service principals for automated workloads instead of sharing individual user credentials.

## 10. Ownership and MANAGE Privilege

Ownership is an important part of Unity Catalog governance.

An object owner has significant control over that object, including the ability to manage its permissions.

The `MANAGE` privilege can also allow a principal to manage privileges on supported securable objects without being the owner.

Example:

```sql
GRANT MANAGE
ON SCHEMA main.gold
TO `data_platform_admins`;
```

**Important:**

- Ownership and `MANAGE` are not identical.
- `MANAGE` does not automatically provide every data access privilege.
- Exact behavior depends on the object and privilege model.

## 11. Row-Level and Column-Level Security

Sometimes users should have access to only part of a dataset.

For example, a company has sales data for multiple regions.

- North team should see North region records.
- South team should see South region records.
- Finance may need access to all approved records.

Unity Catalog supports fine-grained access-control capabilities, including row filters and column masks, subject to supported configurations and platform features.

### 11.1 Row-Level Security

Row filters restrict which records a user can see.

Example:

```text
Sales Table
--------------------------------
order_id | region | amount
--------------------------------
101      | North  | 500
102      | South  | 700
103      | North  | 900
104      | East   | 600
```

A North-region analyst should see:

```text
order_id | region | amount
--------------------------------
101      | North  | 500
103      | North  | 900
```

The user can query the table but sees only records allowed by the row-filter policy.

### 11.2 Column-Level Security

Column masks control how sensitive column values are displayed.

Example:

```text
Customer Table
--------------------------------
customer_id | customer_name | phone
--------------------------------
501         | Rahul         | 9876543210
502         | Priya         | 9876501234
```

An authorized user may see the full phone number.

Another user may see a masked value:

```text
customer_id | customer_name | phone
--------------------------------
501         | Rahul         | XXXXXXX210
502         | Priya         | XXXXXXX234
```

**Important:** A mask changes the value presented to users; it does not necessarily remove the original data from storage.

## 12. Views as an Access-Control Technique

Views can provide controlled access to selected columns or records.

Example:

```sql
CREATE VIEW main.gold.public_customers AS
SELECT
    customer_id,
    customer_name
FROM main.silver.customers;
```

Grant access:

```sql
GRANT SELECT
ON VIEW main.gold.public_customers
TO `business_users`;
```

This allows users to query the view without granting them direct access to the underlying table, provided the view's configured security behavior and ownership model permit it.

**Important:** A view is not automatically a security boundary in every configuration. Review its access mode, underlying object permissions, and supported security behavior.

## 13. Storage Credentials and External Locations

Databricks sometimes needs to access data stored outside managed table storage, such as files in Azure Data Lake Storage Gen2.

Unity Catalog provides storage credentials and external locations to manage access to cloud storage.

### Storage Credential

A storage credential represents the authentication mechanism used to access cloud storage.

### External Location

An external location combines:

- A cloud storage path.
- A storage credential.

Conceptually:

```text
        STORAGE CREDENTIAL
                |
                v
        EXTERNAL LOCATION
                |
                v
        ADLS GEN2 CONTAINER
                |
                v
           DATA FILES
```

Example:

```text
External Location: sales_data
URL: abfss://raw@storageaccount.dfs.core.windows.net/
```

Users need the appropriate Unity Catalog privileges to access supported external locations and their files.

Common external-location privileges include:

- `READ FILES`
- `WRITE FILES`
- `CREATE EXTERNAL TABLE`
- `CREATE EXTERNAL VOLUME`

**Important:** Cloud IAM permissions and Unity Catalog permissions are separate security layers. Both must be configured correctly for the intended access.

## 14. Least Privilege Principle

The principle of least privilege means giving users only the permissions they need to perform their work.

Example:

| Role | Appropriate Access |
|---|---|
| Data Engineer | Read source tables and modify approved transformation tables |
| Data Analyst | Read approved Silver and Gold tables |
| Business User | Read selected reporting views |
| Pipeline Service Principal | Access only the required source and target assets |
| Administrator | Manage governance and workspace settings |

Avoid giving every user broad permissions such as ownership or full administrative access.

### Why Is Least Privilege Important?

- Reduces unauthorized access.
- Limits accidental data modification.
- Protects sensitive information.
- Reduces the impact of compromised accounts.
- Supports security and compliance requirements.

## 15. Practical Example: Role-Based Access

Suppose an organization has three teams:

- Data Engineers
- Data Analysts
- Business Users

The organization wants:

- Engineers to manage transformation tables.
- Analysts to read Silver and Gold data.
- Business users to read only approved Gold views.

### Step 1: Create Groups

Groups are generally managed through the organization's identity provider or Databricks account administration.

```text
data_engineers
data_analysts
business_users
```

### Step 2: Grant Analyst Access

```sql
GRANT USE CATALOG
ON CATALOG main
TO `data_analysts`;

GRANT USE SCHEMA
ON SCHEMA main.silver
TO `data_analysts`;

GRANT USE SCHEMA
ON SCHEMA main.gold
TO `data_analysts`;

GRANT SELECT
ON SCHEMA main.silver
TO `data_analysts`;

GRANT SELECT
ON SCHEMA main.gold
TO `data_analysts`;
```

This grants the analyst group read access to supported tables and views in those schemas, subject to the applicable privilege model.

### Step 3: Grant Business User Access

```sql
GRANT USE CATALOG
ON CATALOG main
TO `business_users`;

GRANT USE SCHEMA
ON SCHEMA main.gold
TO `business_users`;

GRANT SELECT
ON VIEW main.gold.public_customers
TO `business_users`;
```

Business users receive access to the approved view rather than the entire Gold schema.

### Step 4: Review Permissions

Administrators should regularly review:

- Direct grants.
- Group memberships.
- Ownership.
- Inherited privileges.
- Service principal permissions.
- External storage access.

**Production consideration:** Validate the effective permissions with the actual user or group context. A direct grant or revoke may not tell the complete access story.

## 16. Common Mistakes

| Mistake | Correct Understanding |
|---|---|
| Authentication and authorization are the same | Authentication verifies identity; authorization checks permissions |
| `SELECT` alone always guarantees table access | Parent catalog and schema usage permissions may also be required |
| Giving access to a user is always best | Group-based access is easier to maintain |
| `REVOKE` removes all access | Other grants, groups, ownership, or inheritance may still provide access |
| `MANAGE` and ownership are identical | They are different permission concepts |
| Cloud IAM and Unity Catalog permissions are the same | They are separate security layers |
| Views always hide underlying data | Security behavior depends on view configuration and permissions |
| Row filters delete unauthorized records | They restrict which rows users can see |
| Column masks remove sensitive data from storage | They control the displayed values |
| Every user should receive broad access | Follow least privilege |

## 17. Interview Questions and Answers

**Q1. What is access control in Databricks?**

Access control defines which users, groups, or service principals can access data assets and what operations they can perform.

**Q2. What is Unity Catalog?**

Unity Catalog is Databricks' centralized governance solution for managing data and AI assets, including permissions, discovery, auditing, and lineage.

**Q3. What is the Unity Catalog hierarchy?**

Metastore → Catalog → Schema → Tables, views, volumes, and other securable objects.

**Q4. What is the difference between GRANT and REVOKE?**

GRANT provides a privilege, while REVOKE removes a previously granted privilege.

**Q5. What is the purpose of USE CATALOG and USE SCHEMA?**

They provide the required usage access to the parent catalog and schema so users can access objects within them.

**Q6. What is the difference between a user, group, and service principal?**

A user represents a person, a group represents multiple users, and a service principal represents an application or automated workload.

**Q7. What is the least privilege principle?**

It means providing only the minimum permissions required to perform a task.

**Q8. What is row-level security?**

It restricts the records a user can see based on an access policy.

**Q9. What is column-level security?**

It controls access to sensitive columns, often using column masks to protect displayed values.

**Q10. What is the difference between ownership and MANAGE?**

Ownership provides significant control over an object. MANAGE is a privilege that allows permission management on supported objects without necessarily owning them.

**Q11. What is an external location?**

An external location is a Unity Catalog securable object that associates a cloud storage path with a storage credential.

**Q12. Why are groups preferred over individual grants?**

Groups simplify permission management, improve consistency, and make onboarding and offboarding easier.

**Q13. Does REVOKE always remove a user's access?**

No. The user may still have access through another group, inherited privilege, ownership, or another grant.

**Q14. What is the difference between Unity Catalog permissions and cloud IAM permissions?**

Unity Catalog controls access through Databricks governance. Cloud IAM controls access at the cloud storage or infrastructure layer. Both may be required.

**Q15. How would you provide analysts read-only access to Gold tables?**

I would create or use an analyst group, grant `USE CATALOG` and `USE SCHEMA` on the required parent objects, and grant `SELECT` on the approved Gold tables or schema. I would avoid granting `MODIFY` unless required.

## 18. Quick Revision

| Concept | Remember |
|---|---|
| Authentication | Verifies identity |
| Authorization | Determines permitted actions |
| Unity Catalog | Centralized governance |
| Metastore | Top-level governance container |
| Catalog | Organizes schemas |
| Schema | Organizes data objects |
| GRANT | Provides privileges |
| REVOKE | Removes privileges |
| USE CATALOG | Required catalog usage access |
| USE SCHEMA | Required schema usage access |
| SELECT | Read data |
| MODIFY | Modify table data |
| MANAGE | Manage privileges where supported |
| Groups | Manage team permissions |
| Service Principal | Application or automated identity |
| Row Filter | Restricts visible rows |
| Column Mask | Controls displayed column values |
| External Location | Governed cloud storage path |
| Least Privilege | Give only necessary access |

### One-Minute Interview Answer

In Databricks, access control determines who can access data and what operations they can perform.

Unity Catalog provides centralized governance through a hierarchy of metastore, catalog, schema, and data objects.

We manage permissions using privileges such as `SELECT`, `MODIFY`, `USE CATALOG`, and `USE SCHEMA`, with `GRANT` and `REVOKE` commands.

For production environments, I prefer group-based access and service principals for automated pipelines. I also follow the least-privilege principle.

For sensitive data, I can use supported row filters, column masks, and controlled views. When accessing external cloud storage, I also consider cloud IAM permissions alongside Unity Catalog permissions.

## 19. Self-Assessment

- [ ] Explain authentication and authorization.
- [ ] Explain Unity Catalog's role in access control.
- [ ] Draw the Unity Catalog hierarchy.
- [ ] Explain common Unity Catalog privileges.
- [ ] Write GRANT and REVOKE commands.
- [ ] Explain USE CATALOG and USE SCHEMA.
- [ ] Explain privilege inheritance.
- [ ] Differentiate users, groups, and service principals.
- [ ] Explain ownership and MANAGE.
- [ ] Explain row-level security.
- [ ] Explain column-level security.
- [ ] Explain storage credentials and external locations.
- [ ] Differentiate Unity Catalog and cloud IAM permissions.
- [ ] Apply the least-privilege principle.
- [ ] Design read-only access for an analyst group.

**Interview readiness:** You should be able to explain the Unity Catalog permission hierarchy, write basic GRANT and REVOKE statements, and design secure access for data engineers, analysts, business users, and automated pipelines.