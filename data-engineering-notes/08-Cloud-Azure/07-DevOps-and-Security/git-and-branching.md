# Git and Branching for Data Engineering

## 1. What Is Git?

**Git** is a distributed version control system used to track changes in files and coordinate work between developers.

In data engineering, Git helps manage:

- Python and PySpark scripts.
- SQL queries and stored procedures.
- Databricks notebooks and project files.
- Azure Data Factory pipeline definitions.
- Configuration files.
- Automated tests.
- Infrastructure-as-Code files.

**Example:**

Suppose two data engineers are working on the same data pipeline. One updates customer transformations, while the other modifies order processing.

Git helps them work independently, track changes, review code, and combine their work safely.

---

## 2. Why Do We Need Git?

Without version control, developers may:

- Overwrite each other's changes.
- Lose previous working code.
- Struggle to identify who changed a file.
- Find it difficult to compare versions.
- Have no reliable history of modifications.
- Accidentally deploy unreviewed changes.

### Benefits of Git

| Benefit | Explanation |
|---|---|
| Version history | Tracks changes over time |
| Collaboration | Multiple developers can work on a project |
| Branching | Supports independent development |
| Rollback | Helps restore earlier code versions |
| Code review | Supports reviewing changes before merging |
| Traceability | Records commits and their authors |
| Integration | Works with CI/CD platforms |

---

## 3. Git vs GitHub vs Azure Repos

These terms are related but different.

| Term | Meaning |
|---|---|
| Git | Version control system |
| GitHub | Cloud platform for hosting Git repositories and collaboration |
| Azure Repos | Azure DevOps service for hosting Git repositories |
| GitLab | Platform for Git repository hosting and DevOps workflows |

Git is the underlying version control technology. GitHub, Azure Repos, and GitLab provide platforms for hosting and collaborating on Git repositories.

---

## 4. Important Git Terminology

### Repository

A repository is a project directory tracked by Git.

It contains files and the history of committed changes.

### Working Directory

The working directory contains the files you currently edit.

### Staging Area

The staging area contains changes selected for the next commit.

### Commit

A commit is a recorded snapshot of staged changes.

### Branch

A branch is an independent line of development.

### Merge

A merge combines changes from one branch into another.

### Remote Repository

A remote repository is another copy of the repository, usually hosted on GitHub or Azure Repos.

### Clone

Clone creates a local copy of a remote repository, including its Git history.

### Pull

Pull fetches remote changes and integrates them into the current local branch.

### Push

Push sends local commits to a remote repository.

---

## 5. Understanding the Three Main Git Areas

Git commonly works with three important areas:

```text
Working Directory
       |
       | git add
       v
Staging Area
       |
       | git commit
       v
Local Repository
       |
       | git push
       v
Remote Repository
```

### 1. Working Directory

You modify files here.

Example:

```python
def calculate_total(quantity, price):
    return quantity * price
```

### 2. Staging Area

You select which changes should be included in the next commit.

```bash
git add customer_transformation.py
```

### 3. Local Repository

You record the staged changes.

```bash
git commit -m "Add customer transformation"
```

### 4. Remote Repository

You upload your local commits.

```bash
git push origin feature/customer-transformation
```

**Remember:**

- `git add` stages changes.
- `git commit` records changes locally.
- `git push` uploads commits to the remote repository.

---

## 6. Basic Git Setup

### Configure Your Identity

```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

Check the configuration:

```bash
git config --global --list
```

Git uses this information to identify the author of commits.

### Initialize a Repository

```bash
mkdir data-engineering-project
cd data-engineering-project

git init
```

`git init` creates a new local Git repository.

### Clone an Existing Repository

```bash
git clone <repository-url>
```

Example:

```bash
git clone https://github.com/example/data-engineering-project.git
```

Cloning downloads the repository and its history.

---

## 7. Common Git Commands

### Check Repository Status

```bash
git status
```

Shows:

- Modified files.
- Untracked files.
- Staged changes.
- Current branch.

### Add Changes

```bash
# Stage one file
git add customer.py

# Stage all changes in the current directory
git add .
```

### Commit Changes

```bash
git commit -m "Add customer data validation"
```

A good commit message briefly explains what changed.

### View Commit History

```bash
# Compact history
git log --oneline

# Detailed history
git log
```

### Compare Changes

```bash
# Compare working directory changes
git diff

# Compare staged changes
git diff --staged
```

### Push Changes

```bash
git push origin feature/customer-validation
```

### Pull Changes

```bash
git pull origin main
```

This fetches changes from the remote branch and integrates them into the current branch.

### Fetch Changes

```bash
git fetch origin
```

Fetch downloads remote updates without integrating them into your current branch.

**Important difference:**

- `git fetch` downloads remote information.
- `git pull` fetches and integrates remote changes.

---

## 8. What Is Branching?

Branching allows developers to work on different features or fixes without directly changing the main development branch.

Example:

```text
                    main
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
       feature/    feature/     bugfix/
       customer     orders      null-values
```

Each developer can work on a separate branch.

### Why Do We Need Branches?

1. Multiple developers can work independently.
2. Incomplete changes do not need to enter the main branch.
3. Features can be reviewed before merging.
4. Bugs can be fixed separately.
5. Releases can be managed more safely.

---

## 9. Common Branch Types

Branch naming conventions vary between organizations.

| Branch | Purpose |
|---|---|
| `main` | Stable or production-ready code |
| `develop` | Integration branch in some workflows |
| `feature/*` | New feature development |
| `bugfix/*` | Bug fixes |
| `hotfix/*` | Urgent production fixes |
| `release/*` | Preparing a release |

Not every team uses all these branches. Some teams work directly with short-lived feature branches and `main`.

### Example Branch Names

```text
main
develop
feature/customer-pipeline
feature/order-transformation
bugfix/null-handling
hotfix/production-failure
release/v1.0
```

---

## 10. Creating and Switching Branches

### Create a Branch

```bash
git branch feature/customer-pipeline
```

This creates a branch but does not switch to it.

### Switch to a Branch

```bash
git switch feature/customer-pipeline
```

### Create and Switch in One Command

```bash
git switch -c feature/customer-pipeline
```

Older equivalent:

```bash
git checkout -b feature/customer-pipeline
```

### List Branches

```bash
# Local branches
git branch

# Local and remote-tracking branches
git branch -a
```

### Delete a Branch

```bash
# Delete a branch that has been merged
git branch -d feature/customer-pipeline

# Force-delete a local branch
git branch -D feature/customer-pipeline
```

**Caution:** Force deletion can discard unmerged work.

---

## 11. Typical Feature Branch Workflow

Suppose you need to create a new PySpark customer transformation.

### Step 1: Update the Main Branch

```bash
git switch main
git pull origin main
```

### Step 2: Create a Feature Branch

```bash
git switch -c feature/customer-transformation
```

### Step 3: Write or Update Code

```python
from pyspark.sql import functions as F

cleaned_df = (
    customers_df
    .filter(F.col("customer_id").isNotNull())
    .dropDuplicates(["customer_id"])
)
```

### Step 4: Check Changes

```bash
git status
git diff
```

### Step 5: Stage Changes

```bash
git add customer_transformation.py
```

### Step 6: Commit Changes

```bash
git commit -m "Add customer data transformation"
```

### Step 7: Push the Branch

```bash
git push -u origin feature/customer-transformation
```

The `-u` option sets the upstream tracking branch. Future pushes and pulls can often use `git push` and `git pull` without specifying the remote and branch.

### Step 8: Create a Pull Request

Open a pull request from:

```text
feature/customer-transformation
                |
                v
               main
```

The team reviews and validates the changes.

### Step 9: Merge the Pull Request

After approval and required checks pass, the feature branch is merged into the target branch.

### Step 10: Update Your Local Main Branch

```bash
git switch main
git pull origin main
```

### Step 11: Delete the Feature Branch

```bash
git branch -d feature/customer-transformation
```

Delete the remote branch through the repository platform or:

```bash
git push origin --delete feature/customer-transformation
```

---

## 12. Merge in Git

A merge combines changes from one branch into another.

Example:

```bash
git switch main
git merge feature/customer-transformation
```

This merges the feature branch into the current `main` branch.

### Types of Merge

#### 1. Fast-Forward Merge

A fast-forward merge is possible when the target branch has not moved forward since the feature branch was created.

Git moves the target branch pointer forward.

```text
Before:

A --- B --- C  (feature)
      |
      main

After:

A --- B --- C  (main, feature)
```

No separate merge commit is required.

#### 2. Three-Way Merge

A three-way merge is used when both branches have new commits after they diverged.

Git uses:

- The common ancestor.
- The target branch changes.
- The source branch changes.

It combines the changes and may create a merge commit.

```text
Before:

       B --- C  (feature)
      /
A ---+
      \
       D --- E  (main)

After:

       B --- C
      /       \
A ---+         M  (main)
      \       /
       D --- E
```

The merge commit `M` records the integration.

**Important:** A merge does not always create a new commit. Fast-forward merges simply move a branch pointer.

---

## 13. Merge Conflicts

A merge conflict occurs when Git cannot automatically combine changes.

Example:

Two developers modify the same line in a SQL file.

Developer A:

```sql
SELECT customer_id, customer_name
FROM customers;
```

Developer B:

```sql
SELECT customer_id, email
FROM customers;
```

Git may not know which version should be retained.

### Conflict Markers

A conflicted file may contain:

```text
<<<<<<< HEAD
SELECT customer_id, customer_name
=======
SELECT customer_id, email
>>>>>>> feature/customer-email
```

### How to Resolve a Conflict

1. Open the conflicted file.
2. Understand both changes.
3. Decide which changes should remain.
4. Edit the file to create the correct final version.
5. Remove the conflict markers.
6. Stage the resolved file.
7. Complete the merge.

Commands:

```bash
git status

# After manually resolving the conflict
git add customer.sql

# Complete the merge
git commit
```

For a merge that is in progress, `git merge --abort` can usually return the branch to its pre-merge state.

**Important:** Do not blindly choose one side. Understand the business logic before resolving a data transformation or SQL conflict.

---

## 14. Merge vs Rebase

Both merge and rebase integrate changes from another branch, but they work differently.

### Merge

Merge combines branch histories and can create a merge commit.

```bash
git switch feature/customer-pipeline
git merge main
```

### Rebase

Rebase moves or replays your branch commits on top of another branch.

```bash
git switch feature/customer-pipeline
git rebase main
```

### Comparison

| Feature | Merge | Rebase |
|---|---|---|
| Purpose | Integrates branch changes | Replays commits on a new base |
| History | Preserves branch history | Creates rewritten commit history |
| Merge commit | May create one | Does not create a merge commit for the rebase itself |
| Commit IDs | Existing commits remain | Replayed commits get new IDs |
| Shared branches | Generally safe | Requires caution |

### Example

Before rebase:

```text
       B --- C  (feature)
      /
A ---+
      \
       D --- E  (main)
```

After rebasing feature onto main:

```text
A --- D --- E --- B' --- C'  (feature)
```

`B'` and `C'` are new commits based on the updated history.

**Important:** Avoid rebasing commits that other developers are already using unless the team has agreed on that workflow. Rewriting shared history can create problems for collaborators.

---

## 15. Git Reset vs Revert vs Restore

These commands serve different purposes.

### 15.1 git restore

Used to discard or restore changes in files.

Discard unstaged changes in one file:

```bash
git restore customer.py
```

Unstage a file while keeping its working-directory changes:

```bash
git restore --staged customer.py
```

### 15.2 git reset

Moves the current branch pointer and can change the staging area or working directory.

Common modes:

| Mode | Branch | Staging Area | Working Directory |
|---|---|---|---|
| `--soft` | Moves | Keeps changes staged | Keeps changes |
| `--mixed` | Moves | Resets staging | Keeps changes |
| `--hard` | Moves | Resets staging | Resets tracked files |

Examples:

```bash
# Undo the latest commit but keep changes staged
git reset --soft HEAD~1

# Undo the latest commit and unstage changes
git reset --mixed HEAD~1

# Destructively reset tracked files and branch to previous commit
git reset --hard HEAD~1
```

**Caution:** `git reset --hard` can permanently discard uncommitted changes. Use it carefully.

### 15.3 git revert

Creates a new commit that reverses the changes introduced by an earlier commit.

```bash
git revert <commit-id>
```

This is generally safer for undoing changes that have already been pushed to a shared branch because it does not rewrite existing history.

### Comparison

| Command | Purpose |
|---|---|
| `restore` | Restore file content or unstage changes |
| `reset` | Move branch pointer and optionally reset staged/working changes |
| `revert` | Create a new commit that reverses an earlier commit |

---

## 16. Fetch vs Pull

These commands both retrieve updates from a remote repository.

### Git Fetch

```bash
git fetch origin
```

- Downloads remote commits and references.
- Updates remote-tracking branches.
- Does not automatically integrate changes into the current branch.

### Git Pull

```bash
git pull origin main
```

- Fetches remote changes.
- Integrates them into the current branch using merge or, when configured/requested, rebase.

### Comparison

| Feature | Fetch | Pull |
|---|---|---|
| Downloads remote changes | Yes | Yes |
| Updates remote-tracking references | Yes | Yes |
| Integrates changes into current branch | No | Yes |
| Lets you inspect before integration | Yes | Possible, but fetch is more explicit |

A safe workflow is:

```bash
git fetch origin
git log --oneline --graph --all
git merge origin/main
```

This lets you inspect remote changes before integrating them.

---

## 17. .gitignore

`.gitignore` specifies intentionally untracked files that Git should ignore.

It is useful for excluding:

- Python cache files.
- Temporary files.
- Local environment files.
- Logs.
- Generated output.
- Local secrets and credentials.

### Example `.gitignore`

```gitignore
# Python
__pycache__/
*.pyc
.venv/

# Environment and secrets
.env
.env.*
!.env.example

# Logs
*.log

# Temporary files
*.tmp

# IDE settings
.vscode/
.idea/
```

**Important:** `.gitignore` does not automatically stop tracking a file that has already been committed. If a sensitive file was committed, remove it from tracking and address the exposure. Deleting it from the latest commit alone may not remove it from repository history.

Example:

```bash
git rm --cached .env
git commit -m "Stop tracking environment file"
```

If a real credential was exposed, rotate or revoke it immediately.

---

## 18. Git Tags

A tag marks a specific commit, commonly to identify a release.

Example:

```bash
# Create a lightweight tag
git tag v1.0.0

# Create an annotated tag
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push a tag
git push origin v1.0.0

# List tags
git tag
```

Tags are useful for:

- Release management.
- Identifying production versions.
- Tracking deployment milestones.
- Reproducing earlier releases.

---

## 19. Git in Data Engineering Projects

Consider a data engineering project containing:

```text
data-engineering-project/
|
+-- src/
|   +-- customer_transformation.py
|   +-- order_transformation.py
|
+-- sql/
|   +-- create_tables.sql
|   +-- customer_quality_checks.sql
|
+-- tests/
|   +-- test_customer.py
|
+-- databricks/
|   +-- job_config.yml
|
+-- adf/
|   +-- pipeline-definitions/
|
+-- requirements.txt
|
+-- .gitignore
```

Git can track changes to the project files.

### Example Team Workflow

Suppose three engineers work on the project:

| Engineer | Branch | Work |
|---|---|---|
| Engineer A | `feature/customer` | Customer transformation |
| Engineer B | `feature/orders` | Order transformation |
| Engineer C | `bugfix/null-values` | Fix null handling |

Each engineer:

1. Creates a branch.
2. Updates the required files.
3. Commits changes.
4. Pushes the branch.
5. Creates a pull request.
6. Gets the code reviewed.
7. Merges approved changes.

After merging, CI/CD can test and deploy the updated project.

### Important Data Engineering Practices

- Store source code and configuration in Git.
- Avoid committing raw production data.
- Never commit passwords or access keys.
- Keep environment-specific settings outside hardcoded source code.
- Use pull requests for review.
- Test transformation logic before merging.
- Coordinate changes to shared schemas and pipeline definitions.
- Keep production deployment controlled.

---

## 20. Branching Strategies

A branching strategy defines how a team organizes branches and manages code changes.

### 20.1 Feature Branch Workflow

Each feature or bug fix gets a separate branch.

```text
main
 |
 +--- feature/customer
 |
 +--- feature/orders
 |
 +--- bugfix/null-handling
```

Developers merge completed work through pull requests.

**Suitable for:** Many small and medium-sized teams.

### 20.2 Gitflow

Gitflow is a structured branching model that commonly uses:

- `main` — production releases.
- `develop` — integration of completed features.
- `feature/*` — feature development.
- `release/*` — release preparation.
- `hotfix/*` — urgent production fixes.

**Suitable for:** Teams that need structured release management.

### 20.3 Trunk-Based Development

Developers integrate small changes frequently into a shared main or trunk branch.

Short-lived branches may be used, with automated testing and feature flags helping manage releases.

**Suitable for:** Teams practicing frequent integration and automated delivery.

### Comparison

| Strategy | Main Idea | Complexity |
|---|---|---|
| Feature Branch | Separate branch for each feature | Low to medium |
| Gitflow | Multiple branches for development and releases | Higher |
| Trunk-Based | Frequent integration into the main branch | Low branching complexity |

There is no single best strategy for every organization. The team should choose one that fits its release process and deployment requirements.

---

## 21. Common Git Problems

### Problem 1: Forgot to Create a Branch

You started making changes directly on `main`.

If the changes are uncommitted, you can usually create a new branch and carry them with you:

```bash
git switch -c feature/customer-pipeline
```

The current working-directory changes remain available on the new branch.

### Problem 2: Committed to the Wrong Branch

If the commit has not been pushed, one possible recovery is:

```bash
# Create a branch at the current commit
git branch feature/customer-pipeline

# Move main back one commit
git reset --hard HEAD~1

# Switch to the feature branch
git switch feature/customer-pipeline
```

Use this only after confirming the commit and working tree state. Do not reset a shared branch without team coordination.

### Problem 3: Push Rejected

Possible reasons:

- Remote branch has newer commits.
- Local branch is behind the remote branch.
- Branch protection rules prevent direct pushes.
- Authentication or permission problems.

Possible workflow:

```bash
git fetch origin
git status
git pull --rebase origin feature/customer-pipeline
git push origin feature/customer-pipeline
```

Resolve any conflicts before pushing.

### Problem 4: Accidentally Staged a File

```bash
git restore --staged customer.py
```

This unstages the file but keeps its changes.

### Problem 5: Need to Undo a Pushed Commit

For a shared branch, consider:

```bash
git revert <commit-id>
git push origin main
```

This creates a new commit that reverses the earlier change.

---

## 22. Common Interview Questions and Answers

### Q1. What is Git?

Git is a distributed version control system used to track changes, maintain project history, and support collaboration.

### Q2. What is a Git repository?

A repository is a project directory managed by Git, including its files and commit history.

### Q3. What is the difference between Git and GitHub?

Git is the version control system. GitHub is a platform that hosts Git repositories and provides collaboration features.

### Q4. What is a branch?

A branch is an independent line of development that allows developers to work on changes without directly modifying another branch.

### Q5. What is a commit?

A commit is a recorded snapshot of staged changes in the local repository.

### Q6. What is the staging area?

It is the intermediate area where developers select changes to include in the next commit.

### Q7. What is the difference between `git add` and `git commit`?

`git add` stages changes. `git commit` records the staged changes in the local repository.

### Q8. What is the difference between `git push` and `git pull`?

Push uploads local commits to a remote repository. Pull fetches remote changes and integrates them into the current branch.

### Q9. What is the difference between fetch and pull?

Fetch downloads remote updates without integrating them into the current branch. Pull fetches and integrates them.

### Q10. What is a merge conflict?

A merge conflict occurs when Git cannot automatically combine changes from different branches. A developer must resolve the conflicting content.

### Q11. What is the difference between merge and rebase?

Merge combines branch histories and may create a merge commit. Rebase replays commits on a new base and rewrites their commit identities.

### Q12. What is the difference between reset and revert?

Reset moves the current branch pointer and may change the staging area or working directory. Revert creates a new commit that reverses an earlier commit.

### Q13. What is `.gitignore`?

It specifies untracked files and directories that Git should ignore.

### Q14. What is a pull request?

A pull request is a request to review and merge changes from one branch into another.

### Q15. What is a hotfix branch?

A hotfix branch is used for urgent fixes, often for production issues.

### Q16. What is Gitflow?

Gitflow is a branching model that uses branches such as `main`, `develop`, `feature`, `release`, and `hotfix`.

### Q17. What is trunk-based development?

It is a workflow in which developers integrate small changes frequently into a shared main branch or trunk.

### Q18. How do multiple data engineers collaborate using Git?

Each engineer works on a separate branch, commits changes, pushes them to a remote repository, and creates pull requests. After review and automated checks, approved changes are merged.

### Q19. How do you undo a change in a shared production branch?

I would generally use `git revert` to create a new commit that reverses the change, rather than rewriting shared history with reset.

### Q20. How do you prevent credentials from entering Git?

Use `.gitignore` for local secret files, secret managers such as Azure Key Vault, identity-based authentication, and secret-scanning checks. If a credential is exposed, rotate it immediately.

---

## 23. Common Mistakes to Avoid

- Confusing Git with GitHub.
- Assuming `git add` saves changes permanently.
- Forgetting to commit before pushing.
- Working directly on `main` when the team requires feature branches.
- Using `git reset --hard` without checking for uncommitted work.
- Rebasing shared branches without coordination.
- Assuming `.gitignore` removes previously committed files.
- Committing passwords, API keys, or production data.
- Resolving merge conflicts without understanding the code.
- Using `git pull` without considering local changes or the team's merge/rebase policy.
- Deleting branches before confirming that important work is merged.

---

## 24. Quick Revision

| Command / Concept | Purpose |
|---|---|
| `git init` | Initialize a repository |
| `git clone` | Copy a remote repository locally |
| `git status` | Check working tree status |
| `git add` | Stage changes |
| `git commit` | Record staged changes |
| `git push` | Upload commits |
| `git fetch` | Download remote updates |
| `git pull` | Fetch and integrate updates |
| `git branch` | List or create branches |
| `git switch` | Switch branches |
| `git merge` | Combine branch changes |
| `git rebase` | Replay commits on a new base |
| `git restore` | Restore file content or unstage changes |
| `git reset` | Move branch pointer and reset areas |
| `git revert` | Reverse a commit with a new commit |
| `git log` | View commit history |
| `git diff` | Compare changes |
| `.gitignore` | Ignore specified untracked files |
| `git tag` | Mark a specific commit |
| Pull Request | Review and merge proposed changes |

---

## 25. Self-Assessment

Try answering these without looking at the notes:

1. What is Git, and why is it important for data engineers?
2. Explain the working directory, staging area, and local repository.
3. What is a branch, and why do teams use feature branches?
4. Explain the difference between `git add`, `git commit`, and `git push`.
5. What is the difference between fetch and pull?
6. Explain fast-forward and three-way merge.
7. What causes a merge conflict, and how do you resolve one?
8. Explain merge vs rebase.
9. Differentiate restore, reset, and revert.
10. What is `.gitignore`, and what happens if a secret was already committed?
11. Explain the feature branch workflow from creation to merging.
12. Which branching strategy would you choose for a data engineering team, and why?

**Interview-ready summary:**

Git is a distributed version control system that helps data engineers track code changes and collaborate safely. Developers create branches for features or fixes, stage and commit their changes, push them to a remote repository, and use pull requests for review. Approved changes are merged into the main branch and can then be validated and deployed through CI/CD. Understanding branching, merging, conflict resolution, and safe recovery commands is essential for working on production data engineering projects.