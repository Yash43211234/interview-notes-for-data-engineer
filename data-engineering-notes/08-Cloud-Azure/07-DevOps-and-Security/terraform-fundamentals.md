

# Terraform Fundamentals for Data Engineers

## 1. What Is Terraform?

Terraform is an **Infrastructure as Code (IaC)** tool developed by HashiCorp.

It allows you to define, create, update, and manage cloud infrastructure using configuration files instead of manually creating resources through a cloud portal.

For example, a data engineering project may require:

- Azure Data Lake Storage Gen2
- Azure Data Factory
- Azure Databricks workspace
- Azure Key Vault
- Azure networking resources
- Azure role assignments

Instead of creating each resource manually, you can define the infrastructure in Terraform files and deploy it consistently.

### Simple definition

Terraform allows us to manage infrastructure using code.

---

## 2. Why Do We Need Terraform?

Imagine creating the same data engineering environment manually for three environments:

- Development
- Testing
- Production

Manually creating resources can lead to:

- Different configurations across environments.
- Human errors.
- Repeated work.
- Difficulty tracking infrastructure changes.
- Inconsistent deployments.

Terraform solves these problems by defining infrastructure in reusable configuration files.

### Benefits

1. **Automation:** Create infrastructure using code.
2. **Consistency:** Use the same configuration across environments.
3. **Version control:** Track infrastructure changes using Git.
4. **Reusability:** Reuse modules and configurations.
5. **Repeatability:** Recreate infrastructure when required.
6. **Reviewability:** Preview infrastructure changes before applying them.
7. **Multi-cloud support:** Manage supported resources across different providers.

---

## 3. What Is Infrastructure as Code (IaC)?

Infrastructure as Code means managing infrastructure through machine-readable configuration files.

### Without IaC

```text
Azure Portal
     |
     v
Manually create resources
     |
     v
Configure permissions
     |
     v
Repeat for each environment
```

### With IaC

```text
Terraform Configuration
          |
          v
     Terraform CLI
          |
          v
    Cloud Provider
          |
          v
  Infrastructure Created
```

### Types of IaC approaches

| Approach | Explanation |
|---|---|
| Declarative | Describe the desired final infrastructure state |
| Imperative | Specify the sequence of steps to create infrastructure |

Terraform primarily uses a **declarative approach**.

You describe what infrastructure should exist. Terraform determines the actions needed to reach that desired state.

---

## 4. How Terraform Works

Terraform follows a workflow:

```text
Write Configuration
        |
        v
terraform init
        |
        v
terraform validate
        |
        v
terraform plan
        |
        v
Review Proposed Changes
        |
        v
terraform apply
        |
        v
Infrastructure Created or Updated
```

### Step 1: Write configuration

Define the resources you want to create using Terraform configuration files.

### Step 2: Initialize

```bash
terraform init
```

Downloads required providers and initializes the working directory.

### Step 3: Validate

```bash
terraform validate
```

Checks whether the configuration is syntactically valid and internally consistent.

### Step 4: Plan

```bash
terraform plan
```

Shows the changes Terraform proposes to make.

### Step 5: Apply

```bash
terraform apply
```

Applies the planned changes to create, update, or delete infrastructure.

**Important:** Review the plan before applying changes, especially in production.

---

## 5. Terraform Architecture and Components

Terraform uses several important components.

### 5.1 Terraform Core

Terraform Core is responsible for:

- Reading configuration files.
- Comparing desired configuration with the current state.
- Building an execution plan.
- Coordinating infrastructure changes through providers.
- Managing Terraform state.

### 5.2 Providers

A provider is a plugin that allows Terraform to interact with a particular platform or service.

Examples:

- AzureRM provider for Azure resources.
- AWS provider for AWS resources.
- Google provider for Google Cloud resources.
- Databricks provider for Databricks resources.

Example:

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

The provider communicates with Azure APIs to manage supported resources.

### 5.3 Resources

A resource represents an infrastructure object that Terraform manages.

Examples:

- Storage account
- Resource group
- Data Factory
- Key Vault
- Databricks workspace

### 5.4 Data Sources

A data source reads information about an existing resource without managing its lifecycle as a Terraform resource.

Example:

```hcl
data "azurerm_resource_group" "existing" {
  name = "rg-existing-data"
}
```

This reads details of an existing resource group.

### Resource vs Data Source

| Resource | Data Source |
|---|---|
| Manages infrastructure lifecycle | Reads existing information |
| Can create or update resources | Does not create the referenced object |
| Can destroy managed resources | Does not destroy the referenced object |
| Represents desired infrastructure | Retrieves existing infrastructure details |

---

## 6. Terraform Configuration Language (HCL)

Terraform uses **HashiCorp Configuration Language (HCL)**.

Terraform files commonly use the `.tf` extension.

Example:

```text
terraform-project/
│
├── main.tf
├── providers.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
└── versions.tf
```

These filenames are conventions. Terraform loads configuration from all applicable `.tf` files in the working directory.

### Common file purposes

| File | Purpose |
|---|---|
| `main.tf` | Main resource definitions |
| `providers.tf` | Provider configuration |
| `variables.tf` | Input variable declarations |
| `outputs.tf` | Output values |
| `terraform.tfvars` | Variable values |
| `versions.tf` | Terraform and provider version constraints |

You can organize files differently depending on project requirements.

---

## 7. Terraform Blocks

Terraform configurations contain different types of blocks.

### 7.1 Terraform Block

Defines Terraform requirements, including provider requirements and version constraints.

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}
```

### 7.2 Provider Block

Configures the provider used to communicate with a platform.

```hcl
provider "azurerm" {
  features {}
}
```

### 7.3 Resource Block

Defines infrastructure that Terraform should manage.

General syntax:

```hcl
resource "provider_resource_type" "local_name" {
  argument = "value"
}
```

Example:

```hcl
resource "azurerm_resource_group" "data_rg" {
  name     = "rg-data-engineering"
  location = "Central India"
}
```

Here:

- `azurerm_resource_group` is the resource type.
- `data_rg` is the Terraform local name.
- `name` is the Azure resource group name.
- `location` is its Azure region.

### 7.4 Data Block

Reads an existing resource.

```hcl
data "azurerm_resource_group" "existing" {
  name = "rg-existing-data"
}
```

### 7.5 Variable Block

Defines input variables.

```hcl
variable "location" {
  description = "Azure deployment region"
  type        = string
  default     = "Central India"
}
```

### 7.6 Output Block

Exposes useful values after deployment.

```hcl
output "resource_group_name" {
  value = azurerm_resource_group.data_rg.name
}
```

---

## 8. Practical Example: Create Azure Resources Using Terraform

Let's create a basic Azure environment for a data engineering project.

Required resources:

1. Azure Resource Group
2. Azure Storage Account
3. Storage Container

### Project structure

```text
azure-data-platform/
│
├── main.tf
├── variables.tf
├── outputs.tf
└── terraform.tfvars
```

### Step 1: Configure provider and resources

`main.tf`

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "data_rg" {
  name     = "rg-data-platform"
  location = var.location
}

resource "azurerm_storage_account" "data_storage" {
  name                     = var.storage_account_name
  resource_group_name      = azurerm_resource_group.data_rg.name
  location                 = azurerm_resource_group.data_rg.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
  is_hns_enabled           = true
}

resource "azurerm_storage_container" "raw_container" {
  name                  = "raw"
  storage_account_id    = azurerm_storage_account.data_storage.id
  container_access_type = "private"
}
```

The storage account has hierarchical namespace enabled, making it suitable as the foundation for ADLS Gen2.

### Step 2: Define variables

`variables.tf`

```hcl
variable "location" {
  description = "Azure region"
  type        = string
  default     = "Central India"
}

variable "storage_account_name" {
  description = "Globally unique storage account name"
  type        = string
}
```

### Step 3: Provide variable values

`terraform.tfvars`

```hcl
storage_account_name = "stdataengdemo123"
```

Storage account names must follow Azure naming rules and be globally unique. Replace the example with an available name.

### Step 4: Define outputs

`outputs.tf`

```hcl
output "resource_group_name" {
  value = azurerm_resource_group.data_rg.name
}

output "storage_account_id" {
  value = azurerm_storage_account.data_storage.id
}

output "storage_container_name" {
  value = azurerm_storage_container.raw_container.name
}
```

### Step 5: Deploy

```bash
terraform init
terraform validate
terraform plan
terraform apply
```

Terraform creates the resources after the plan is approved and the apply operation succeeds.

---

## 9. Terraform Variables

Variables make Terraform configurations reusable.

Without variables, values are hardcoded in resource definitions.

With variables, you can supply different values for different environments.

### Variable types

| Type | Example |
|---|---|
| string | `"Central India"` |
| number | `3` |
| bool | `true` |
| list | `["dev", "test", "prod"]` |
| map | `{ environment = "dev" }` |
| object | Structured collection of named attributes |
| tuple | Ordered collection with potentially different element types |

Example:

```hcl
variable "environment" {
  type    = string
  default = "dev"
}

variable "worker_count" {
  type    = number
  default = 2
}

variable "enable_logging" {
  type    = bool
  default = true
}
```

### Variable precedence

Terraform can receive variable values from different sources.

Common sources include:

1. Command-line `-var` and `-var-file` options.
2. Automatically loaded variable files.
3. Environment variables.
4. Default values.

For exact precedence, command-line variable options take precedence over variable files and environment variables; defaults are used when no higher-precedence value is supplied.

Example:

```bash
terraform apply -var="environment=prod"
```

### Sensitive variables

```hcl
variable "database_password" {
  type      = string
  sensitive = true
}
```

The `sensitive` flag helps prevent the value from being displayed in normal CLI output.

**Important:** Marking a variable sensitive does not encrypt or remove it from Terraform state. Do not use it as a substitute for secure state storage.

---

## 10. Terraform Outputs

Outputs expose values from Terraform configuration.

Example:

```hcl
output "storage_account_name" {
  description = "Created storage account"
  value       = azurerm_storage_account.data_storage.name
}
```

View outputs:

```bash
terraform output
```

View a specific output:

```bash
terraform output storage_account_name
```

Outputs are useful for:

- Displaying resource IDs.
- Passing information between modules.
- Sharing deployment details with other automation.
- Connecting infrastructure deployment with application configuration.

Sensitive outputs should be marked `sensitive = true`.

---

## 11. Terraform State

Terraform state is one of the most important concepts.

Terraform maintains a state file to track the infrastructure it manages.

The default local state filename is:

```text
terraform.tfstate
```

### Why is state required?

Terraform needs to understand:

- Which resources it manages.
- The identifiers of those resources.
- The relationship between configuration and deployed infrastructure.
- Which changes are required during the next deployment.

### Example

Suppose your configuration defines:

```hcl
resource "azurerm_resource_group" "data_rg" {
  name     = "rg-data-platform"
  location = "Central India"
}
```

After applying the configuration, Terraform records information about the resource in its state.

During a future plan, Terraform compares the configuration with the state and the provider's view of the actual resource.

### Important commands

```bash
terraform state list
```

Lists resources tracked in state.

```bash
terraform state show azurerm_resource_group.data_rg
```

Displays details of a resource in state.

```bash
terraform state pull
```

Reads the current state from the configured backend.

### Important security point

Terraform state may contain sensitive values, including credentials or resource attributes.

Treat state as sensitive information.

Do not commit `terraform.tfstate` or state backups to a public Git repository.

---

## 12. Local State vs Remote State

### Local State

The state file is stored on the local machine.

Advantages:

- Simple to start.
- Useful for learning and individual experiments.
- No remote backend setup required.

Disadvantages:

- Difficult to collaborate.
- Risk of losing state.
- Difficult to coordinate multiple deployments.
- Requires secure local storage and backup practices.

### Remote State

State is stored in a shared remote backend.

For Azure, a common choice is an Azure Storage Account with a Blob container.

Advantages:

- Shared state for team members.
- Centralized storage.
- Supports state locking with the AzureRM backend.
- Easier collaboration and controlled access.

### Example: AzureRM Backend

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformstate123"
    container_name       = "tfstate"
    key                  = "data-platform.tfstate"
  }
}
```

This configuration identifies where Terraform should store the state.

The backend storage account and container must already exist before initialization.

Initialize the backend:

```bash
terraform init
```

### State locking

State locking helps prevent multiple Terraform operations from modifying the same state simultaneously.

The AzureRM backend supports state locking.

**Interview point:** Remote state is recommended for team-based environments because it provides shared state management and helps coordinate infrastructure changes.

---

## 13. Terraform Plan

`terraform plan` previews the changes Terraform intends to make.

Common plan symbols:

| Symbol | Meaning |
|---|---|
| `+` | Create resource |
| `-` | Destroy resource |
| `~` | Update resource in place |
| `-/+` | Destroy and recreate resource |
| `<=` | Read a data source |

Example:

```bash
terraform plan
```

Save a plan:

```bash
terraform plan -out=tfplan
```

Apply the saved plan:

```bash
terraform apply tfplan
```

Using a saved plan helps ensure the reviewed plan is the one being applied.

However, a saved plan may become stale if the configuration, state, or remote infrastructure changes.

---

## 14. Terraform Lifecycle

Terraform lifecycle meta-arguments control how Terraform handles resource changes.

Common lifecycle settings:

### 14.1 create_before_destroy

Creates a replacement resource before destroying the old one, when supported by the resource and dependency constraints.

```hcl
lifecycle {
  create_before_destroy = true
}
```

Useful when downtime must be minimized.

### 14.2 prevent_destroy

Prevents Terraform from planning destruction of a resource while this lifecycle rule remains in the configuration.

```hcl
lifecycle {
  prevent_destroy = true
}
```

Useful for protecting critical infrastructure.

It does not protect a resource if the lifecycle rule itself is removed and the configuration is applied.

### 14.3 ignore_changes

Tells Terraform to ignore changes to specified resource attributes when planning updates.

```hcl
lifecycle {
  ignore_changes = [
    tags
  ]
}
```

Useful when an external system or team manages a particular attribute.

Use it carefully because it can hide configuration drift.

---

## 15. Terraform Modules

A module is a reusable collection of Terraform configuration files.

Modules help organize infrastructure into logical components.

Example:

```text
terraform-project/
│
├── main.tf
├── variables.tf
├── outputs.tf
│
└── modules/
    ├── storage/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    │
    └── data-factory/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Why use modules?

- Reuse infrastructure definitions.
- Reduce duplicated code.
- Improve maintainability.
- Standardize resource configurations.
- Separate infrastructure into logical components.

### Calling a module

```hcl
module "storage" {
  source = "./modules/storage"

  storage_account_name = "stdataengdemo123"
  location             = "Central India"
}
```

### Module inputs and outputs

A module can accept input variables and expose output values.

For example:

```hcl
output "storage_account_id" {
  value = azurerm_storage_account.data_storage.id
}
```

The root module can reference it:

```hcl
output "created_storage_id" {
  value = module.storage.storage_account_id
}
```

### Root Module vs Child Module

| Root Module | Child Module |
|---|---|
| Main working configuration | Reusable module called by another module |
| Usually contains deployment entry point | Contains a logical infrastructure component |
| Calls other modules | Can call other modules |
| Runs as the main Terraform configuration | Used through a module block |

---

## 16. Terraform Workspaces

Terraform workspaces allow multiple state instances to be associated with one configuration.

For example:

```bash
terraform workspace new dev
terraform workspace new test
terraform workspace new prod
```

List workspaces:

```bash
terraform workspace list
```

Select a workspace:

```bash
terraform workspace select dev
```

Show the current workspace:

```bash
terraform workspace show
```

### Why use workspaces?

The same Terraform configuration can be applied with separate state for different instances.

### Important distinction

Terraform workspaces are not the same as Azure environments or Azure DevOps environments.

For complex production deployments, separate root configurations, backend keys, or repositories may provide clearer isolation than relying only on workspaces.

Do not assume separate workspaces automatically provide separate credentials, permissions, or cloud resources.

---

## 17. Terraform and Git

Terraform configuration should normally be stored in version control.

A typical workflow:

```text
Developer
    |
    v
Modify Terraform Files
    |
    v
Create Git Branch
    |
    v
Commit Changes
    |
    v
Open Pull Request
    |
    v
Run Terraform Format and Validate
    |
    v
Generate Terraform Plan
    |
    v
Review and Approve
    |
    v
Apply Infrastructure Changes
```

### Useful Git practices

- Keep Terraform files in Git.
- Review infrastructure changes through pull requests.
- Avoid committing state files.
- Do not commit secrets or credentials.
- Use `.gitignore` for local Terraform files.

Example `.gitignore`:

```gitignore
.terraform/
*.tfstate
*.tfstate.*
*.tfplan
crash.log
*.tfvars
```

If a `.tfvars` file contains only non-sensitive environment settings and the team intentionally versions it, it can be committed. Never commit files containing secrets.

The provider lock file, `.terraform.lock.hcl`, should normally be committed so the team uses consistent provider selections.

---

## 18. Terraform with CI/CD

Terraform can be integrated with GitHub Actions or Azure Pipelines.

### Typical CI/CD workflow

```text
Developer Pushes Code
          |
          v
       Git Repository
          |
          v
      CI/CD Pipeline
          |
          v
    terraform fmt -check
          |
          v
    terraform validate
          |
          v
     terraform plan
          |
          v
     Review and Approval
          |
          v
     terraform apply
          |
          v
    Azure Infrastructure
```

### Common pipeline stages

| Stage | Purpose |
|---|---|
| Format | Check consistent formatting |
| Validate | Check configuration validity |
| Plan | Preview infrastructure changes |
| Review | Review proposed changes |
| Apply | Deploy approved changes |

### Important security practices

- Use workload identity federation/OIDC where supported.
- Avoid storing long-lived credentials in pipeline variables.
- Use a remote state backend.
- Restrict who can approve production deployments.
- Protect production environments.
- Keep state access restricted.
- Do not expose secrets in pipeline logs.

**Important:** `terraform apply` changes real infrastructure. Production deployment should use an approved plan and controlled permissions.

---

## 19. Terraform vs ARM Templates vs Bicep

Azure provides native infrastructure deployment options, including ARM templates and Bicep.

| Feature | Terraform | ARM Templates | Bicep |
|---|---|---|---|
| Developer | HashiCorp | Microsoft | Microsoft |
| Language | HCL | JSON | Bicep language |
| Main purpose | IaC across supported providers | Azure infrastructure | Azure infrastructure |
| Multi-cloud | Yes, through providers | No | No |
| State management | Terraform state | Azure Resource Manager manages deployment state | Azure Resource Manager manages deployment state |
| Reusability | Modules | Linked/nested templates | Modules |
| Azure integration | Provider-based | Native | Native |

### When should you use Terraform?

Use Terraform when:

- Your organization manages infrastructure across multiple cloud providers.
- You want a common IaC workflow.
- Your team already uses Terraform modules and state management.
- You need consistent infrastructure automation across supported services.

### When should you use Bicep?

Use Bicep when:

- Your infrastructure is Azure-focused.
- You prefer native Azure deployment tooling.
- You want to avoid managing Terraform state.

Neither tool is universally better. Choose based on the organization's platform, governance, and operational requirements.

---

## 20. Terraform vs Ansible

These tools solve different but sometimes overlapping problems.

| Feature | Terraform | Ansible |
|---|---|---|
| Main purpose | Provision and manage infrastructure | Configure systems and automate tasks |
| Typical use | Create Azure Storage Account | Install and configure software |
| Approach | Declarative | Playbook-based task automation |
| State | Maintains Terraform state | Generally agentless; does not use Terraform-style infrastructure state |
| Example | Create a Databricks workspace | Configure a VM or application |

A team may use Terraform to provision infrastructure and Ansible to configure software running on that infrastructure.

---

## 21. Terraform in a Data Engineering Project

Consider a company building an Azure data platform.

Required infrastructure:

- Resource Group
- ADLS Gen2
- Azure Data Factory
- Azure Databricks
- Key Vault
- Managed Identity
- Role Assignments
- Networking

### Architecture

```text
                Terraform
                    |
                    v
             Azure Provider
                    |
       ---------------------------
       |       |       |         |
       v       v       v         v
     ADLS     ADF   Databricks  Key Vault
       |       |       |         |
       ---------------------------
                    |
                    v
          Managed Identity and
             Role Assignments
                    |
                    v
             Data Platform
                    |
                    v
         Data Ingestion and ETL
```

### Deployment workflow

1. Define the Azure resources in Terraform.
2. Configure provider authentication.
3. Create the resource group and storage account.
4. Enable hierarchical namespace for ADLS Gen2.
5. Create ADF and Databricks resources.
6. Configure managed identities.
7. Assign required permissions.
8. Configure Key Vault and secure access.
9. Review the Terraform plan.
10. Apply the configuration.
11. Deploy data pipelines and application code separately.

### Important distinction

Terraform primarily provisions and manages infrastructure. It does not replace ADF or Databricks as the data processing and orchestration engines.

A useful separation is:

- Terraform: Infrastructure provisioning.
- ADF: Data pipeline orchestration and data movement.
- Databricks: Distributed data processing.
- ADLS Gen2: Data storage.
- Key Vault: Secret management.
- GitHub Actions/Azure Pipelines: CI/CD automation.

---

## 22. Terraform Commands for Interviews

| Command | Purpose |
|---|---|
| `terraform init` | Initialize working directory and providers |
| `terraform fmt` | Format configuration files |
| `terraform fmt -check` | Check formatting without modifying files |
| `terraform validate` | Validate configuration |
| `terraform plan` | Preview changes |
| `terraform apply` | Apply infrastructure changes |
| `terraform destroy` | Destroy managed infrastructure |
| `terraform output` | Display outputs |
| `terraform state list` | List resources in state |
| `terraform state show` | Show a resource's state |
| `terraform providers` | Show provider requirements |
| `terraform workspace list` | List workspaces |
| `terraform workspace select` | Select a workspace |
| `terraform show` | Display current state or a saved plan |
| `terraform refresh` | Refresh state using a plan operation; legacy workflow, generally prefer `terraform plan -refresh-only` |

### Important command distinction

`terraform init` initializes the project.

`terraform plan` previews changes.

`terraform apply` performs the changes.

`terraform destroy` removes infrastructure managed by the configuration.

---

## 23. Common Terraform Errors

| Problem | Possible reason | What to check |
|---|---|---|
| Provider initialization failure | Provider unavailable or version issue | Provider source, version, network |
| Authentication failure | Invalid Azure credentials | Azure CLI login, service principal, federated identity |
| Resource already exists | Resource exists outside Terraform management | Import or reference it appropriately |
| State lock error | Another operation holds the state lock | Check active Terraform operations |
| Invalid configuration | Incorrect HCL syntax or arguments | Run `terraform fmt` and `terraform validate` |
| Permission denied | Identity lacks Azure permissions | Azure RBAC and target-service permissions |
| Resource replacement | An argument requires recreation | Review plan and provider documentation |
| State drift | Actual infrastructure differs from configuration/state | Refresh and review the plan |

### What is infrastructure drift?

Infrastructure drift occurs when actual infrastructure differs from the configuration or state Terraform expects.

Example:

A developer manually changes a storage account setting through the Azure Portal.

Terraform may detect this difference during a later plan.

You should review the proposed correction before applying it.

---

## 24. Common Mistakes to Avoid

- Assuming Terraform creates resources without authentication or permissions.
- Confusing a provider with a resource.
- Confusing resources with data sources.
- Assuming `terraform plan` changes infrastructure.
- Applying a production plan without review.
- Committing Terraform state files to Git.
- Storing passwords directly in `.tf` files.
- Assuming `sensitive = true` encrypts values in state.
- Using local state for a collaborative production deployment.
- Ignoring state locking.
- Assuming workspaces automatically isolate permissions and credentials.
- Running `terraform destroy` without checking the impact.
- Ignoring infrastructure drift.
- Assuming Terraform replaces ADF or Databricks.

---

## 25. Interview Questions and Answers

### Q1. What is Terraform?

Terraform is an Infrastructure as Code tool used to define, provision, and manage infrastructure through configuration files.

### Q2. What is Infrastructure as Code?

IaC means managing infrastructure through code instead of manually creating and configuring resources.

### Q3. Is Terraform declarative or imperative?

Terraform is primarily declarative. We define the desired infrastructure state, and Terraform determines the required operations.

### Q4. What is a Terraform provider?

A provider is a plugin that allows Terraform to communicate with a platform or service, such as Azure, AWS, or Databricks.

### Q5. What is a Terraform resource?

A resource represents infrastructure that Terraform manages, such as an Azure Storage Account or Resource Group.

### Q6. What is a data source?

A data source reads information about an existing resource without managing its lifecycle.

### Q7. What is Terraform state?

Terraform state records information about infrastructure managed by Terraform. It helps Terraform map configuration to actual resources and determine changes.

### Q8. Why is remote state important?

Remote state allows team members and automation pipelines to use shared infrastructure state. It also supports centralized access control and state locking when the backend provides it.

### Q9. What is the difference between `terraform plan` and `terraform apply`?

`terraform plan` previews proposed infrastructure changes. `terraform apply` executes the changes.

### Q10. What is a Terraform module?

A module is a reusable collection of Terraform configuration files used to organize and share infrastructure definitions.

### Q11. What is infrastructure drift?

Drift occurs when actual infrastructure changes outside Terraform and no longer matches the expected configuration or state.

### Q12. What is the difference between a variable and an output?

A variable passes input values into a configuration. An output exposes selected values from the configuration.

### Q13. What is the purpose of `terraform init`?

It initializes the working directory, downloads required providers, and configures the backend.

### Q14. What happens if you delete the Terraform state file?

Terraform loses its record of the resources it manages. It may propose creating resources that already exist. The existing resources are not automatically deleted, but the state must be recovered or resources imported carefully.

### Q15. How do you protect Terraform state?

Use a secure remote backend, restrict access, enable encryption and locking where supported, and avoid committing state files to Git.

### Q16. Can Terraform manage Azure Databricks?

Yes. Terraform supports Azure Databricks resources through the relevant providers. It can provision workspaces and manage supported Databricks resources.

### Q17. What is the difference between Terraform and Azure Data Factory?

Terraform provisions and manages infrastructure. ADF orchestrates data workflows and performs data movement and supported transformations.

### Q18. How would you deploy the same infrastructure across DEV, TEST, and PROD?

Use reusable modules and environment-specific configuration values, with separate state and controlled permissions for each environment. Separate root configurations or backend keys can provide stronger isolation.

### Q19. What is the purpose of `prevent_destroy`?

It prevents Terraform from planning destruction of a resource while that lifecycle rule is present in the configuration.

### Q20. How would you handle a resource created manually in Azure that Terraform needs to manage?

Use Terraform's import workflow to associate the existing resource with a Terraform resource address, then align the configuration with the actual resource. Review the plan before applying further changes.

---

## 26. Quick Revision

| Topic | Key Point |
|---|---|
| Terraform | Infrastructure as Code tool |
| HCL | Terraform configuration language |
| Provider | Connects Terraform to a platform |
| Resource | Infrastructure managed by Terraform |
| Data Source | Reads existing resource information |
| Variable | Input to configuration |
| Output | Exposes configuration result |
| State | Tracks managed infrastructure |
| Backend | Stores Terraform state |
| Module | Reusable Terraform configuration |
| Workspace | Separate state instance for a configuration |
| Plan | Previews changes |
| Apply | Executes changes |
| Drift | Actual infrastructure differs from expected configuration |
| Lifecycle | Controls resource replacement and change behavior |
| CI/CD | Automates validation, planning, review, and deployment |

### Five points to remember

1. Terraform is primarily a declarative Infrastructure as Code tool.
2. Providers allow Terraform to communicate with cloud platforms.
3. Terraform state is essential for tracking managed infrastructure.
4. Remote state and access control are important for team collaboration.
5. Terraform provisions infrastructure; ADF and Databricks perform data engineering workloads.

---

## 27. Self-Assessment

Try answering these questions without looking at the notes:

1. Explain Terraform and Infrastructure as Code.
2. Describe the complete Terraform workflow.
3. Explain the difference between a provider, resource, and data source.
4. Why does Terraform need a state file?
5. Compare local state and remote state.
6. What is the difference between `terraform plan` and `terraform apply`?
7. Explain Terraform modules with an Azure example.
8. What is infrastructure drift, and how can you detect it?
9. How would you securely manage Terraform state in a production project?
10. Design a Terraform deployment for ADLS Gen2, ADF, Databricks, and Key Vault.

