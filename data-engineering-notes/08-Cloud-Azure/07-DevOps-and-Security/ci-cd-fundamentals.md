# CI/CD Fundamentals for Data Engineering

## 1. What Is CI/CD?

CI/CD is a software development practice that automates how code is built, tested, validated, and deployed across environments.

CI/CD stands for:

- **CI — Continuous Integration**
- **CD — Continuous Delivery or Continuous Deployment**

In data engineering, CI/CD helps teams safely develop and deploy:

- SQL scripts.
- PySpark code.
- Databricks notebooks and jobs.
- Azure Data Factory pipelines.
- Data transformation logic.
- Infrastructure configurations.
- Data quality tests.

**Example:**

A data engineer updates a PySpark transformation in a Git repository. When the code is pushed, an automated pipeline runs tests and checks. If validation succeeds, the approved changes can be deployed to the development, testing, or production environment.

---

## 2. Why Do We Need CI/CD?

Without CI/CD, developers may manually:

1. Copy code between environments.
2. Upload notebooks individually.
3. Change production configurations.
4. Run tests manually.
5. Deploy changes without proper review.

This can cause:

- Deployment mistakes.
- Inconsistent environments.
- Broken production pipelines.
- Difficult rollbacks.
- Poor collaboration.
- Untracked changes.

CI/CD helps make deployments repeatable, controlled, and reliable.

### Benefits

| Benefit | Explanation |
|---|---|
| Automation | Reduces repetitive manual work |
| Consistency | Uses repeatable deployment steps |
| Quality | Runs automated tests before deployment |
| Collaboration | Supports team development through Git |
| Traceability | Tracks code changes and deployment history |
| Faster delivery | Reduces deployment effort |
| Reliability | Helps identify issues before production |

---

## 3. Continuous Integration (CI)

**Continuous Integration** means frequently integrating code changes into a shared repository and automatically validating those changes.

When a developer pushes code or creates a pull request, a CI pipeline can run:

1. Code checkout.
2. Syntax validation.
3. Code quality checks.
4. Unit tests.
5. Data transformation tests.
6. Build or packaging steps.

### Example: PySpark CI Workflow

A data engineer modifies a PySpark transformation.

```text
Developer
    |
    v
Writes PySpark Code
    |
    v
Pushes Changes to Git
    |
    v
CI Pipeline Starts
    |
    v
Code Quality Checks
    |
    v
Unit Tests
    |
    v
Validation Result
```

If tests fail, the team can fix the issue before merging the changes.

### Common CI Checks

| Check | Purpose |
|---|---|
| Syntax check | Detects invalid code |
| Linting | Checks coding standards |
| Unit testing | Tests individual functions |
| Integration testing | Tests interactions between components |
| Security scanning | Detects known vulnerabilities or exposed secrets |
| Build validation | Verifies that the application or package can be built |

**Important:** CI validates code changes. It does not necessarily deploy them to production.

---

## 4. Continuous Delivery vs Continuous Deployment

Both are commonly represented by CD, but they have different meanings.

### Continuous Delivery

Code is automatically built, tested, and prepared for release.

Production deployment generally requires an explicit approval or manual decision.

### Continuous Deployment

Every change that successfully passes the required automated checks is automatically deployed to production.

### Comparison

| Feature | Continuous Delivery | Continuous Deployment |
|---|---|---|
| Build and test | Automated | Automated |
| Deployment preparation | Automated | Automated |
| Production approval | Usually required | Not normally required |
| Production deployment | Approved release | Automatic after checks |
| Control | More human control | More automation |

**Interview point:** Continuous Delivery keeps software ready for release. Continuous Deployment automatically releases validated changes to production.

---

## 5. CI/CD Pipeline Architecture

A typical CI/CD pipeline contains several stages.

```text
Developer
    |
    v
Git Repository
    |
    v
Continuous Integration
    |
    +--> Code Validation
    |
    +--> Unit Testing
    |
    +--> Security Checks
    |
    v
Build / Package
    |
    v
Deployment Pipeline
    |
    v
Development Environment
    |
    v
Testing Environment
    |
    v
Approval (if required)
    |
    v
Production Environment
    |
    v
Monitoring
```

### Explanation of Each Stage

**1. Source Control**

Code is stored in a version control system such as Git.

**2. Validation**

The pipeline checks syntax, coding standards, dependencies, and configuration.

**3. Testing**

Automated tests verify that the code behaves as expected.

**4. Build or Packaging**

The code and required dependencies are prepared for deployment.

**5. Deployment**

Validated artifacts or configurations are deployed to the target environment.

**6. Approval**

A person or authorized process may approve production deployment.

**7. Monitoring**

The team monitors deployed pipelines, application health, and data quality.

---

## 6. Git and Version Control

Git is a distributed version control system used to track changes in code.

In data engineering, Git can manage:

- Python and PySpark files.
- SQL scripts.
- Databricks notebooks and project files.
- ADF pipeline definitions.
- Configuration files.
- Infrastructure-as-code files.
- Automated tests.

### Common Git Commands

```bash
# Check working tree status
git status

# Download remote repository changes
git pull

# Create a new branch
git checkout -b feature/customer-transformation

# Stage changes
git add .

# Commit changes
git commit -m "Add customer transformation"

# Push branch to remote repository
git push -u origin feature/customer-transformation
```

### Typical Git Workflow

```text
main
 |
 +---- feature/customer-transformation
 |
 +---- feature/order-validation
 |
 +---- bugfix/null-handling
```

Developers create separate branches for their work and merge approved changes into the main branch.

### Pull Request (PR)

A pull request is a request to review and merge changes from one branch into another.

A typical PR process includes:

1. Developer pushes a feature branch.
2. Developer creates a pull request.
3. Team members review the code.
4. Automated CI checks run.
5. Reviewers approve the changes.
6. The branch is merged.

**Important:** A pull request is a collaboration and review mechanism. It is not itself a deployment.

---

## 7. Environments: Development, Testing, and Production

Most data engineering projects use multiple environments.

| Environment | Purpose |
|---|---|
| Development (DEV) | Develop and test initial changes |
| Testing (TEST/QA) | Validate changes before release |
| Production (PROD) | Run live business workloads |

### Example

Suppose a team changes an order-processing pipeline.

**Development:**
- Update PySpark transformation.
- Test using development data.
- Validate output.

**Testing:**
- Deploy the approved changes.
- Run integration and regression tests.
- Verify data quality.

**Production:**
- Deploy after approval.
- Execute the production workflow.
- Monitor results.

### Why Separate Environments?

- Prevents untested code from affecting production.
- Supports controlled testing.
- Separates data and credentials.
- Reduces deployment risk.
- Makes rollback and troubleshooting easier.

**Important:** Environment separation should include access controls, configuration, secrets, and appropriate data isolation—not just separate folders or Git branches.

---

## 8. CI/CD in Azure

Azure provides several services and tools for CI/CD.

### Azure DevOps

Azure DevOps provides tools for managing software development and delivery.

Common components:

| Component | Purpose |
|---|---|
| Azure Repos | Git repositories |
| Azure Pipelines | Build, test, and deployment automation |
| Azure Boards | Work-item and project tracking |
| Azure Test Plans | Test management |
| Azure Artifacts | Package and artifact management |

### GitHub Actions

GitHub Actions is an automation platform integrated with GitHub repositories.

It can automate:

- Code validation.
- Unit testing.
- Build processes.
- Deployment workflows.
- Scheduled jobs.

### Azure DevOps vs GitHub Actions

| Feature | Azure DevOps | GitHub Actions |
|---|---|---|
| Source control | Azure Repos | GitHub repositories |
| CI/CD | Azure Pipelines | Workflows and actions |
| Project tracking | Azure Boards | GitHub Issues and Projects |
| Automation | YAML pipelines and tasks | YAML workflows |
| Integration | Microsoft ecosystem | GitHub ecosystem |

Both can implement CI/CD for Azure data engineering projects.

---

## 9. YAML in CI/CD

YAML is a human-readable format commonly used to define pipeline configurations.

It describes stages, jobs, steps, triggers, variables, and deployment conditions.

### Example: Azure Pipelines YAML

```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: ubuntu-latest

stages:
  - stage: Validate
    jobs:
      - job: Test
        steps:
          - checkout: self

          - script: |
              python --version
              pip install -r requirements.txt
              pytest tests/
            displayName: Run Python tests
```

### Understanding the YAML

| Element | Meaning |
|---|---|
| `trigger` | Defines when the pipeline starts |
| `branches` | Specifies branches that trigger the pipeline |
| `pool` | Selects the execution environment |
| `stages` | Groups major pipeline phases |
| `jobs` | Defines units of work within a stage |
| `steps` | Individual tasks or commands |
| `checkout` | Retrieves repository code |
| `script` | Runs shell commands |

**Important:** YAML is a configuration language. It defines the workflow; the CI/CD platform executes it.

---

## 10. CI/CD for Azure Data Factory

Azure Data Factory supports Git integration and publishing workflows.

### How It Works

1. Connect ADF to an Azure DevOps Git repository or supported GitHub repository.
2. Developers work in feature branches.
3. Changes are reviewed through pull requests.
4. Approved changes are merged into the collaboration branch.
5. Publish generates deployment artifacts, including the ARM template-based deployment files.
6. The deployment process applies the changes to the target ADF environment.

### ADF CI/CD Architecture

```text
             ADF Development
                    |
                    v
                Git Branch
                    |
                    v
               Pull Request
                    |
                    v
              Merge Changes
                    |
                    v
             ADF Publish
                    |
                    v
          ARM Template Artifacts
                    |
                    v
            Deployment Pipeline
                    |
           +--------+--------+
           |                 |
           v                 v
        ADF TEST          ADF PROD
```

### Important ADF Concepts

**Git Mode:** Used for collaborative development and version control.

**Live Mode:** Represents the currently published factory configuration.

**Publish:** Validates and publishes changes from the collaboration branch to the development factory and generates deployment artifacts.

**ARM Templates:** Azure Resource Manager templates describe Azure resources and configurations for deployment.

### Environment-Specific Configuration

Different environments may have different:

- Storage accounts.
- Key Vaults.
- SQL databases.
- Integration Runtime configurations.
- Linked service settings.

These values should be parameterized or managed through environment-specific deployment configuration.

**Important:** ADF Git integration does not mean every change is automatically deployed to production. Deployment is a separate controlled process.

---

## 11. CI/CD for Azure Databricks

Databricks projects can use CI/CD to deploy notebooks, source code, jobs, and related configurations.

Common approaches include:

- Databricks Git folders for development.
- Databricks Asset Bundles (formerly Databricks Bundles).
- Databricks CLI.
- REST APIs.
- Azure DevOps Pipelines.
- GitHub Actions.

### Example Workflow

```text
Developer
    |
    v
Databricks Project
    |
    v
Git Repository
    |
    v
Pull Request
    |
    v
Automated Tests
    |
    v
Deploy to DEV
    |
    v
Integration Testing
    |
    v
Approval
    |
    v
Deploy to PROD
```

### Databricks Asset Bundles

Databricks Asset Bundles provide a way to define and deploy Databricks resources as a project.

They can manage resources such as:

- Jobs.
- Pipelines.
- Notebooks and source files.
- Related configuration.

A bundle configuration commonly uses YAML.

### Simplified Example

```yaml
bundle:
  name: customer-data-pipeline

targets:
  dev:
    mode: development
    workspace:
      host: https://<dev-workspace-url>

  prod:
    mode: production
    workspace:
      host: https://<prod-workspace-url>
```

This is an illustrative configuration. Actual resource definitions, authentication, and deployment settings must be added for a working project.

### Why Use Bundles?

- Keeps deployment configuration in Git.
- Supports repeatable deployments.
- Separates development and production targets.
- Helps manage jobs and pipelines consistently.
- Supports automated CI/CD workflows.

---

## 12. Testing in Data Engineering CI/CD

Testing is essential because successful code execution does not necessarily mean correct data.

### 12.1 Unit Testing

Tests individual functions or transformation logic.

Example:

```python
def calculate_total(quantity, price):
    return quantity * price


def test_calculate_total():
    assert calculate_total(3, 100) == 300
```

### 12.2 Integration Testing

Tests whether multiple components work together.

Example:

- Databricks reads data from ADLS.
- PySpark transforms the data.
- The result is written to a Delta table.

### 12.3 Data Quality Testing

Checks the correctness and reliability of the output data.

Common checks:

- Null validation.
- Duplicate validation.
- Uniqueness.
- Referential integrity.
- Accepted value ranges.
- Record-count reconciliation.

Example:

```python
from pyspark.sql import functions as F

# Check for null customer IDs
null_count = (
    df.filter(F.col("customer_id").isNull())
      .count()
)

assert null_count == 0
```

### 12.4 Regression Testing

Checks whether new changes have broken existing functionality.

Example:

A change to customer transformation logic should not unexpectedly change previously validated order totals.

### Testing Comparison

| Test Type | What It Checks |
|---|---|
| Unit | Individual functions |
| Integration | Interaction between components |
| Data quality | Correctness of data |
| Regression | Existing functionality after changes |
| End-to-end | Complete pipeline behavior |

---

## 13. Deployment Strategies

### 13.1 Rolling Deployment

Replaces instances or components gradually rather than updating everything simultaneously.

Useful for services that support multiple running instances.

### 13.2 Blue-Green Deployment

Maintains two production environments:

- Blue — current production version.
- Green — new version.

The new version is tested in Green. Traffic or workload is switched after validation.

### 13.3 Canary Deployment

Releases a new version to a small subset of users or workloads first.

If it performs correctly, the rollout expands.

### Application to Data Engineering

These strategies are more common in application deployments, but their principles can help with data platform releases.

For data pipelines, similar safety techniques include:

- Deploying a new job alongside the old job.
- Testing against a limited data range.
- Writing to a separate target table.
- Validating output before switching downstream consumers.

**Important:** A data pipeline deployment must also consider schema compatibility, duplicate writes, backfills, and downstream dependencies.

---

## 14. Secrets and Configuration Management

CI/CD pipelines often need credentials to access Azure resources.

Examples:

- Azure service connections.
- Databricks authentication.
- Storage credentials.
- Database credentials.
- API tokens.

### Best Practices

1. Never hardcode credentials in YAML.
2. Use Azure Key Vault for secrets.
3. Use Managed Identity or workload identity federation where supported.
4. Restrict pipeline permissions.
5. Separate development and production credentials.
6. Avoid printing secret values in logs.
7. Rotate credentials when required.

### Example Architecture

```text
Azure DevOps Pipeline
        |
        | Secure identity
        v
    Azure Key Vault
        |
        | Retrieves required secret
        v
   Deployment Process
        |
        v
 Azure Data Platform
```

**Important:** Access to a Key Vault secret and access to the target Azure resource are separate permissions.

---

## 15. Infrastructure as Code (IaC)

Infrastructure as Code means defining infrastructure using configuration or code instead of creating every resource manually.

Common Azure IaC tools include:

- ARM Templates.
- Bicep.
- Terraform.

### Example Resources

A data engineering project may define:

- Azure Data Factory.
- Storage accounts.
- Databricks workspaces.
- Key Vault.
- Networking resources.
- Role assignments.

### Benefits

- Repeatable infrastructure deployment.
- Version-controlled configuration.
- Consistent environments.
- Easier disaster recovery.
- Reduced manual configuration errors.

### IaC vs CI/CD

| Concept | Purpose |
|---|---|
| Infrastructure as Code | Defines infrastructure |
| CI | Validates and integrates code changes |
| CD | Delivers or deploys validated changes |

IaC can be used within a CI/CD pipeline.

---

## 16. Rollback and Recovery

A deployment may introduce an issue.

Examples:

- Incorrect PySpark transformation.
- Broken ADF pipeline.
- Invalid schema change.
- Incorrect Databricks job configuration.
- Production pipeline failure.

### Rollback Options

- Redeploy a previously validated code version.
- Restore an earlier configuration.
- Revert a Git commit and deploy the corrected version.
- Restore compatible infrastructure settings.
- Use Delta Lake time travel or restore capabilities where appropriate for data recovery.

### Important Difference

**Code rollback is not the same as data rollback.**

For example, deploying an older PySpark script does not automatically reverse incorrect records already written to a production Delta table.

A safe recovery plan must consider:

1. Which code version was deployed?
2. Which data was affected?
3. Can the pipeline be rerun safely?
4. Is a backfill required?
5. Is the target table recoverable?
6. Are downstream consumers affected?

---

## 17. CI/CD vs Git vs Orchestration

These concepts are related but serve different purposes.

| Concept | Main Purpose | Example |
|---|---|---|
| Git | Tracks code changes | GitHub, Azure Repos |
| CI | Validates integrated code | Run PySpark unit tests |
| CD | Deploys validated changes | Deploy Databricks job |
| Orchestration | Coordinates data pipeline execution | ADF pipeline |
| IaC | Defines infrastructure | Terraform, Bicep |
| Monitoring | Observes deployed workloads | Azure Monitor |

**Example:**

Git tracks a PySpark change. CI tests it. CD deploys it to Databricks. A Databricks job or ADF pipeline orchestrates its execution. Monitoring checks whether the job succeeds and the output data is correct.

---

## 18. Practical End-to-End Scenario

### Scenario: Deploying a PySpark Customer Pipeline

A company has a Databricks job that processes customer data from ADLS Gen2 and writes curated Delta tables.

A developer modifies the transformation logic to handle duplicate customer records.

### Step 1: Development

The developer updates the PySpark code in a feature branch.

### Step 2: Source Control

The developer commits and pushes the changes to Git.

### Step 3: Pull Request

A pull request is created for review.

### Step 4: Continuous Integration

The CI pipeline:

- Checks Python syntax.
- Runs linting.
- Executes unit tests.
- Validates expected output.
- Checks for exposed credentials.

### Step 5: Merge

After successful checks and approval, the change is merged into the main branch.

### Step 6: Deployment to Development

The CD pipeline deploys the updated Databricks job configuration and code to the development workspace.

### Step 7: Integration Testing

The team runs the job using development data and validates the resulting Delta table.

### Step 8: Production Approval

An authorized person approves the production deployment.

### Step 9: Production Deployment

The CD pipeline deploys the approved version to the production workspace.

### Step 10: Monitoring

The team monitors:

- Job status.
- Execution duration.
- Failed tasks.
- Data quality.
- Record counts.
- Downstream dependencies.

### Architecture

```text
Developer
    |
    v
Git Feature Branch
    |
    v
Pull Request
    |
    v
CI: Tests and Validation
    |
    v
Merge to Main
    |
    v
CD: Deploy to DEV
    |
    v
Integration and Data Quality Tests
    |
    v
Production Approval
    |
    v
CD: Deploy to PROD
    |
    v
Databricks Job / ADF Pipeline
    |
    v
Monitoring and Alerts
```

---

## 19. Common Interview Questions and Answers

### Q1. What is CI/CD?

CI/CD is a set of practices that automates code integration, validation, testing, and deployment.

### Q2. What is Continuous Integration?

CI means frequently integrating code changes into a shared repository and automatically validating them through checks and tests.

### Q3. What is Continuous Delivery?

Continuous Delivery automatically builds, tests, and prepares code for release. Production deployment generally requires approval.

### Q4. What is Continuous Deployment?

Continuous Deployment automatically deploys every change that passes the required checks to production.

### Q5. What is the difference between CI and CD?

CI focuses on integrating and validating code changes. CD focuses on preparing or deploying validated changes to target environments.

### Q6. What is a CI/CD pipeline?

It is an automated workflow containing stages such as source checkout, validation, testing, packaging, deployment, and monitoring.

### Q7. What is Git's role in CI/CD?

Git tracks source code changes and supports collaboration through branches, commits, and pull requests.

### Q8. What is a pull request?

A pull request asks reviewers to examine proposed changes before merging them into another branch.

### Q9. What is the difference between DEV, TEST, and PROD?

DEV is used for development, TEST for validation, and PROD for live workloads.

### Q10. What is Azure DevOps?

Azure DevOps is a development platform that provides services such as Repos, Pipelines, Boards, Test Plans, and Artifacts.

### Q11. What is YAML used for in CI/CD?

YAML defines pipeline configuration, including triggers, stages, jobs, steps, variables, and deployment conditions.

### Q12. How does CI/CD work with Azure Data Factory?

ADF integrates with Git for collaborative development. Changes can be reviewed and merged, published as deployment artifacts, and deployed to other factories through a controlled pipeline.

### Q13. How does CI/CD work with Databricks?

Databricks projects can use Git, automated tests, Databricks Asset Bundles, CLI, and CI/CD platforms to deploy code and resources across workspaces.

### Q14. What is a Databricks Asset Bundle?

It is a project-based deployment mechanism that defines Databricks resources and their configuration for repeatable deployment across targets.

### Q15. Why is testing important in data engineering CI/CD?

Testing helps detect transformation errors, schema issues, integration failures, and data quality problems before changes reach production.

### Q16. What is Infrastructure as Code?

IaC defines infrastructure using code or configuration files so that resources can be deployed consistently and repeatedly.

### Q17. What is the difference between rollback and redeployment?

Rollback restores a previously working version or configuration. Redeployment is the process of deploying a version; it may deploy either a new or an earlier version.

### Q18. How do you manage secrets in a CI/CD pipeline?

Use Key Vault or another approved secret manager, identity-based authentication where supported, and least-privilege permissions. Avoid hardcoding credentials.

### Q19. Does successful deployment guarantee correct data?

No. Deployment success only confirms that deployment steps completed. Data quality and pipeline behavior must be validated separately.

### Q20. How would you design CI/CD for a Databricks data pipeline?

I would store the code and configuration in Git, use pull requests for review, run automated unit and quality tests, deploy to development, run integration tests, require production approval, and deploy using a repeatable mechanism such as Databricks Asset Bundles. After deployment, I would monitor job execution and data quality.

---

## 20. Common Mistakes to Avoid

- Confusing Continuous Delivery with Continuous Deployment.
- Assuming Git and CI/CD are the same thing.
- Deploying directly to production without testing.
- Hardcoding secrets in YAML or source code.
- Assuming a successful deployment guarantees correct output data.
- Treating ADF orchestration as CI/CD.
- Confusing code rollback with data rollback.
- Ignoring environment-specific configuration.
- Deploying notebooks without validating dependencies and permissions.
- Running tests only after production deployment.
- Assuming every Databricks deployment must use the same CI/CD tool.

---

## 21. Quick Revision

| Concept | Remember |
|---|---|
| CI | Integrate and validate code changes |
| CD | Deliver or deploy validated changes |
| Continuous Delivery | Production release usually needs approval |
| Continuous Deployment | Automatic production deployment after checks |
| Git | Version control |
| Branch | Isolated line of development |
| Commit | Recorded code change |
| Pull Request | Review and merge request |
| Azure Repos | Git repository hosting |
| Azure Pipelines | CI/CD automation |
| GitHub Actions | GitHub automation workflows |
| YAML | Pipeline configuration |
| DEV | Development environment |
| TEST | Testing environment |
| PROD | Production environment |
| ADF Publish | Publishes development changes and generates deployment artifacts |
| Databricks Asset Bundles | Define and deploy Databricks resources |
| Unit Testing | Tests individual logic |
| Integration Testing | Tests component interactions |
| Data Quality Testing | Validates data correctness |
| IaC | Defines infrastructure through code |
| Rollback | Restores an earlier working version or configuration |

---

## 22. Self-Assessment

Try answering these without looking at the notes:

1. Explain CI/CD in simple terms.
2. What happens during Continuous Integration?
3. Differentiate Continuous Delivery and Continuous Deployment.
4. Explain the role of Git, branches, commits, and pull requests.
5. Why do we need DEV, TEST, and PROD environments?
6. What are Azure Repos and Azure Pipelines?
7. What is YAML, and why is it used?
8. Explain the ADF CI/CD workflow.
9. How can Databricks Asset Bundles help with deployment?
10. What types of tests should run before deployment?
11. How should secrets be managed in a pipeline?
12. Explain rollback and why code rollback does not automatically reverse data changes.

**Interview-ready summary:**

CI/CD automates the process of integrating, testing, and deploying data engineering code. Git provides version control, CI runs automated checks and tests, and CD deploys validated code and configurations across environments. In Azure data engineering, this can be implemented using Azure DevOps or GitHub Actions with services such as Azure Data Factory and Databricks. A good CI/CD process includes code review, automated testing, environment-specific configuration, secure secret management, controlled production deployment, and post-deployment monitoring.