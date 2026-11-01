# Databricks Repos and Git Integration

## 1. What Is Git?

Git is a distributed version control system used to track changes in source code and collaborate with other developers.

In data engineering, Git helps us manage:

- Databricks notebooks
- Python scripts
- SQL files
- Configuration files
- ETL pipeline code
- Shared project code

**Example:**

Suppose three data engineers are working on the same ETL project. Git helps them work on separate changes, review modifications, and combine their work without manually copying files.

## 2. What Is a Git Repository?

A Git repository (repo) is a location where project files and their version history are stored.

A repository tracks changes made to files over time.

Common repository platforms:

- GitHub
- GitLab
- Bitbucket
- Azure DevOps Repos

Example project structure:

```text
sales-data-pipeline/
│
├── notebooks/
│   ├── ingestion.py
│   ├── transformation.py
│   └── validation.py
│
├── src/
│   └── common_functions.py
│
├── sql/
│   └── create_tables.sql
│
├── tests/
│   └── test_transformations.py
│
└── README.md
```

## 3. What Are Databricks Repos?

Databricks Repos provide Git integration inside the Databricks workspace.

They allow developers to connect a remote Git repository and work with its files directly in Databricks.

They help data engineers:

- Clone or connect a Git repository.
- Work with notebooks and source files.
- Create and switch branches.
- Commit changes.
- Push changes to a remote repository.
- Pull updates made by other developers.
- Collaborate using Git workflows.

**Remember:** Databricks Repos connect the Databricks development environment with a Git repository.

> Databricks has evolved its Git-folder experience over time. The interface may refer to this functionality as Git folders rather than Repos.

## 4. Why Do Data Engineers Need Git Integration?

Without version control, developers may manually copy notebooks or overwrite each other's changes.

Git integration provides a controlled development workflow.

### Benefits

1. **Version control:** Track changes to notebooks and scripts.
2. **Collaboration:** Multiple developers can work on the same project.
3. **Branching:** Develop features independently.
4. **Code review:** Review changes before merging.
5. **Rollback:** Return to an earlier version when needed.
6. **Deployment:** Promote tested code through development, testing, and production environments.

## 5. Git Repository vs. Databricks Workspace

These are related but different concepts.

| Git Repository | Databricks Workspace |
|---|---|
| Stores source code and version history | Provides an environment for working with Databricks resources |
| Supports Git branches and commits | Organizes notebooks and other workspace objects |
| Usually hosted on GitHub, GitLab, or Azure DevOps | Managed within a Databricks workspace |
| Used for collaboration and version control | Used for development, execution, and administration |

**Interview point:** A Git repository manages source code history, while the Databricks workspace provides the environment in which developers work and run their code.

## 6. Connecting a Git Repository to Databricks

A typical setup involves these steps:

1. Create a Git repository on GitHub, GitLab, or another supported provider.
2. Open the Databricks workspace.
3. Configure Git integration in the workspace settings if required.
4. Provide the required Git authentication.
5. Create a Git folder by connecting the remote repository.
6. Select the branch you want to work on.
7. Open and edit project files in Databricks.

The exact setup depends on workspace permissions, Git provider, and authentication configuration.

### Authentication

Databricks may support authentication methods such as:

- Personal access tokens
- OAuth-based authentication
- Other provider-supported authentication methods

**Important:** Never hardcode access tokens or credentials in notebooks. Use an approved secure credential-management approach.

## 7. Important Git Concepts

### 7.1 Clone

Clone means creating a local or workspace copy of a remote Git repository.

Example:

```bash
git clone https://github.com/company/sales-pipeline.git
```

In Databricks, creating a Git folder connected to a remote repository provides a similar development experience.

### 7.2 Branch

A branch is an independent line of development.

It allows developers to work on new features without directly changing the main branch.

Example:

```text
main
 |
 +---- feature/order-ingestion
 |
 +---- feature/customer-cleaning
```

Common branch names:

- `main`
- `develop`
- `feature/order-ingestion`
- `bugfix/null-handling`

### 7.3 Commit

A commit records a set of changes in the Git repository's history.

Example:

```bash
git add .
git commit -m "Add order ingestion notebook"
```

A good commit message should describe what changed.

### 7.4 Push

Push uploads local commits to the remote repository.

```bash
git push origin feature/order-ingestion
```

### 7.5 Pull

Pull fetches remote changes and integrates them into the current local branch.

```bash
git pull origin main
```

### 7.6 Merge

Merge combines changes from one branch into another.

Example:

```text
feature/order-ingestion
          |
          v
        Review
          |
          v
         main
```

In team workflows, changes are commonly merged through a pull request after review.

### 7.7 Pull Request (PR)

A pull request is a request to merge changes from one branch into another.

It allows team members to:

- Review code.
- Discuss changes.
- Run automated checks.
- Approve or request modifications.
- Merge approved changes.

**Remember:** A pull request is a collaboration and review process provided by Git hosting platforms, not a Git command itself.

## 8. Common Git Commands

| Command | Purpose |
|---|---|
| `git clone` | Clone a remote repository |
| `git status` | Check working-tree status |
| `git branch` | List or manage branches |
| `git switch` | Switch branches |
| `git checkout` | Switch branches or restore files |
| `git add` | Stage changes |
| `git commit` | Record staged changes |
| `git push` | Upload commits |
| `git pull` | Fetch and integrate remote changes |
| `git fetch` | Download remote updates without integrating them |
| `git merge` | Combine branches |
| `git log` | View commit history |
| `git diff` | Compare changes |

## 9. Typical Databricks Git Workflow

A common team workflow looks like this:

```text
             Remote Git Repository
                     |
                     v
              Create Git Folder
                     |
                     v
               Create Branch
                     |
                     v
              Develop in Databricks
                     |
                     v
               Test the Code
                     |
                     v
                 Commit
                     |
                     v
                  Push
                     |
                     v
              Create Pull Request
                     |
                     v
                Code Review
                     |
                     v
              Merge into Main
                     |
                     v
              Deploy to Production
```

### Practical example

Suppose you are developing a customer ingestion pipeline.

1. Create a feature branch named `feature/customer-ingestion`.
2. Open the project in Databricks.
3. Develop the ingestion notebook.
4. Test the notebook using development data.
5. Commit the changes.
6. Push the branch to the remote repository.
7. Create a pull request.
8. Get approval from another developer.
9. Merge the changes into the main branch.
10. Deploy the approved version through the team's deployment process.

## 10. Handling Merge Conflicts

A merge conflict occurs when Git cannot automatically combine changes.

Example:

- Developer A changes a line in `ingestion.py`.
- Developer B changes the same line in another branch.
- Both branches are merged.

Git may not know which version should be retained.

### How to resolve a conflict

1. Identify the conflicting files.
2. Open the files and inspect the conflicting sections.
3. Decide which changes should be retained.
4. Edit the file to remove conflict markers.
5. Test the corrected code.
6. Commit the resolution.
7. Push the updated branch.

Example conflict markers:

```python
<<<<<<< HEAD
source_path = "/data/dev/orders"
=======
source_path = "/data/test/orders"
>>>>>>> feature-branch
```

After resolving the conflict:

```python
source_path = "/data/dev/orders"
```

The correct value depends on the intended application behavior.

**Best practice:** Pull or fetch recent changes regularly and avoid unnecessary overlapping edits.

## 11. Databricks Git Folders and Version Control

Databricks Git folders allow developers to work with Git-backed files in the workspace.

Common operations include:

- Viewing changed files.
- Creating branches.
- Switching branches.
- Committing changes.
- Pulling remote changes.
- Pushing local commits.

A Git folder is a development workspace connected to a Git repository. It should not be confused with a production deployment mechanism.

**Important:** Not every Databricks workspace object is automatically version-controlled through Git folders. Workspace-specific resources and deployment configurations may need separate management.

## 12. Git Integration in CI/CD

CI/CD stands for:

- CI: Continuous Integration
- CD: Continuous Delivery or Continuous Deployment

Git integration is an important part of a data engineering CI/CD workflow.

### Typical workflow

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
Automated Tests
    |
    v
Code Review
    |
    v
Merge into Main
    |
    v
Deployment Pipeline
    |
    v
Databricks Production
```

### Continuous Integration

CI automatically checks code changes.

Examples:

- Python syntax checks
- Unit tests
- Code quality checks
- Validation of project configuration

### Continuous Delivery / Deployment

CD moves approved code into the target environment.

Examples:

- Deploy notebooks and source files.
- Update Databricks Job definitions.
- Configure production parameters.
- Run deployment validation.

Databricks Asset Bundles, now also referred to as Declarative Automation Bundles, can help define and deploy Databricks resources such as Jobs and pipelines through a version-controlled project.

## 13. Development, Testing, and Production Branches

A team may use separate branches or other Git workflows to control changes.

Example:

```text
main
 |
 +---- develop
         |
         +---- feature/order-pipeline
         |
         +---- feature/customer-pipeline
```

A typical environment promotion process is:

| Environment | Purpose |
|---|---|
| Development | Build and test new code |
| Testing / QA | Validate functionality and data quality |
| Production | Execute approved workloads |

**Important:** Branching strategy and environment deployment strategy are separate decisions. A team may use different branching models depending on its CI/CD process.

## 14. Best Practices

1. Use meaningful branch names.
2. Write clear commit messages.
3. Create feature branches instead of directly changing the main branch.
4. Review code before merging.
5. Test notebooks before committing.
6. Avoid committing credentials, tokens, and sensitive data.
7. Keep notebooks and source files organized.
8. Resolve merge conflicts carefully.
9. Use automated tests where possible.
10. Keep development and production configurations separate.
11. Use a controlled deployment process.
12. Avoid storing large generated data files in Git repositories.

## 15. Common Interview Questions

### Q1. What is Git?

Git is a distributed version control system used to track changes in source code and support collaboration.

### Q2. What are Databricks Repos?

Databricks Repos, now commonly represented through Git folders, provide Git integration in Databricks so developers can work with remote repository files.

### Q3. Why is Git important in data engineering?

Git helps manage notebook and script versions, collaborate with other engineers, review changes, and support CI/CD workflows.

### Q4. What is the difference between a repository and a branch?

A repository contains project files and their history. A branch is an independent line of development within that repository.

### Q5. What is the difference between `git fetch` and `git pull`?

`git fetch` downloads remote updates without integrating them into the current branch.

`git pull` fetches remote updates and integrates them into the current branch.

### Q6. What is a pull request?

A pull request asks team members to review proposed changes before merging them into another branch.

### Q7. What is a merge conflict?

A merge conflict occurs when Git cannot automatically combine changes from different branches.

### Q8. How do you use Git with Databricks notebooks?

Connect a Git repository through a Databricks Git folder, create a feature branch, edit and test notebooks, commit changes, push them, and create a pull request.

### Q9. What is CI/CD?

CI/CD automates code integration, testing, validation, and deployment.

### Q10. Can Git alone deploy a Databricks Job?

No. Git stores and versions source code. A deployment process or tool is needed to deploy and configure Databricks resources.

### Q11. Why should we use feature branches?

Feature branches allow developers to work independently without directly affecting the main branch.

### Q12. How do you protect credentials in Git?

Never commit credentials or tokens. Use approved secret-management solutions and secure authentication methods.

## 16. Quick Revision

| Concept | Remember |
|---|---|
| Git | Version control system |
| Repository | Project files and history |
| Databricks Git folder | Git-connected development environment |
| Clone | Create a repository copy |
| Branch | Independent development line |
| Commit | Record changes |
| Push | Upload commits |
| Fetch | Download remote updates |
| Pull | Fetch and integrate changes |
| Merge | Combine branches |
| Pull Request | Request review and merge |
| Merge Conflict | Git cannot automatically combine changes |
| CI | Automated integration and testing |
| CD | Controlled delivery or deployment |

## 17. Self-Assessment

Before moving to the next topic, make sure you can explain:

- [ ] What is Git?
- [ ] What is a Git repository?
- [ ] What are Databricks Repos / Git folders?
- [ ] Why do data engineers need Git?
- [ ] What is the difference between a repository and a branch?
- [ ] What is the difference between commit, push, and pull?
- [ ] What is the difference between fetch and pull?
- [ ] What is a pull request?
- [ ] How do you resolve merge conflicts?
- [ ] How does Git integration support CI/CD?
- [ ] How would you manage Databricks notebooks in a team project?

**Interview readiness:** You should be able to explain how multiple data engineers collaborate on Databricks notebooks using Git branches, commits, pull requests, code reviews, and CI/CD deployment.