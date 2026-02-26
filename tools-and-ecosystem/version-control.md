# Version Control with Git

## Table of Contents

- [Why Version Control?](#why-version-control)
- [Git Mental Model](#git-mental-model)
- [Essential Git Commands](#essential-git-commands)
- [Branching Basics](#branching-basics)
- [Example Commit Messages](#example-commit-messages)
- [Ignoring Files](#ignoring-files)
- [Git Configuration](#git-configuration)
- [Common Workflows](#common-workflows)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Next Steps](#next-steps)
- [Resources](#resources)

## About This Document

Version control is the foundation of modern software development. Git enables you to track changes, collaborate with others, experiment safely, and maintain a complete history of your project. This document covers Git fundamentals - understanding commits, branches, and the core mental model that makes Git powerful.

Mastering Git means understanding it as a directed acyclic graph of commits, not just memorizing commands. This conceptual foundation will make every Git operation intuitive.

---

## Why Version Control?

### The Problems It Solves

**Without version control:**

```
my_project/
  script.py
  script_old.py
  script_final.py
  script_final_v2.py
  script_actually_final.py
  script_use_this_one.py
```

**With version control:**

```
my_project/
  script.py  # Complete history tracked by Git
```

### Core Benefits

1. **History**: Complete record of every change, who made it, and why
2. **Collaboration**: Multiple people work on the same codebase without conflicts
3. **Experimentation**: Try risky changes safely with branches
4. **Backup**: Distributed copies protect against data loss
5. **Blame/Credit**: Understand when and why code changed
6. **Rollback**: Recover from mistakes by reverting to previous states

---

## Git Mental Model

### The Three Trees

Git manages your code through three main states:

1. **Working Directory**: Files on disk you're currently editing
2. **Staging Area (Index)**: Changes marked for the next commit
3. **Repository (HEAD)**: Committed history stored in `.git/`

```
Working Directory  →  Staging Area  →  Repository
   (modified)         (git add)        (git commit)
```

### Commits are Snapshots

- Each commit is a complete snapshot of your project at a point in time
- Commits are immutable - once created, they never change
- Commits form a directed acyclic graph (DAG) with parent-child relationships
- A branch is just a pointer to a commit

```
Commit Graph:

  A ← B ← C ← D (main)
       ↖
         E ← F (feature)
```

---

## Essential Git Commands

### Repository Initialization

**Starting a new repository:**

```bash
# Initialize Git in current directory
git init

# Initialize with specific branch name
git init --initial-branch=main
```

**Cloning an existing repository:**

```bash
# Clone from remote
git clone https://github.com/username/repo.git

# Clone into specific directory
git clone https://github.com/username/repo.git my-project
```

### Checking Status

```bash
# See current status
git status

# Compact status
git status -s

# See what branch you're on
git branch --show-current
```

### Staging Changes

```bash
# Stage specific file
git add filename.py

# Stage all changes in current directory
git add .

# Stage all changes in repository
git add -A

# Stage parts of a file interactively
git add -p filename.py

# Unstage a file
git reset filename.py
```

### Committing Changes

```bash
# Commit staged changes
git commit -m "Add user authentication"

# Commit with multi-line message
git commit -m "Add user authentication

- Implement login endpoint
- Add password hashing
- Create user session management"

# Stage and commit in one step (tracked files only)
git commit -am "Fix typo in README"

# Amend last commit (change message or add forgotten files)
git add forgotten_file.py
git commit --amend
```

### Viewing History

```bash
# View commit history
git log

# Compact one-line format
git log --oneline

# Show graph structure
git log --oneline --graph --all

# Last 5 commits
git log -5

# Show changes in each commit
git log -p

# Show commits by specific author
git log --author="Danish"

# Show commits affecting specific file
git log filename.py
```

### Viewing Changes

```bash
# Show unstaged changes
git diff

# Show staged changes
git diff --staged

# Compare with specific commit
git diff abc123

# Show changes in specific file
git diff filename.py

# Show changes between branches
git diff main..feature-branch
```

### Undoing Changes

```bash
# Discard changes in working directory
git checkout -- filename.py

# Unstage a file (keep changes in working directory)
git reset filename.py

# Undo last commit, keep changes staged
git reset --soft HEAD~1

# Undo last commit, unstage changes
git reset HEAD~1

# Undo last commit, discard all changes (DANGEROUS)
git reset --hard HEAD~1

# Revert a commit by creating a new commit
git revert abc123
```

---

## Branching Basics

### Understanding Branches

A branch is a lightweight pointer to a commit. Creating a branch is instant and cheap.

```bash
# List branches
git branch

# Create new branch
git branch feature-login

# Switch to branch
git checkout feature-login

# Create and switch in one command
git checkout -b feature-login

# Modern alternative (Git 2.23+)
git switch feature-login
git switch -c feature-login
```

### Merging Branches

```bash
# Merge feature-login into current branch
git merge feature-login

# Merge with commit even if fast-forward possible
git merge --no-ff feature-login

# Abort merge if conflicts
git merge --abort
```

### Deleting Branches

```bash
# Delete merged branch
git branch -d feature-login

# Force delete unmerged branch
git branch -D feature-login
```

---

## Example Commit Messages

### The Anatomy of a Good Commit Message

```
Short summary (50 chars or less)

Detailed explanation of what changed and why. Wrap at 72 characters.
Focus on the "why" more than the "what" since the diff shows the what.

- Use bullet points for multiple changes
- Separate concerns in different commits
- Reference issue numbers: Fixes #123
```

### Real Examples

**Simple feature addition:**

```
Add user profile page

Implement basic user profile view with username, email, and bio.
Uses existing authentication to show current user's profile.
```

**Bug fix:**

```
Fix null pointer exception in data loader

Check if dataset is empty before accessing first element.
Adds defensive programming to prevent crashes with empty data.

Fixes #456
```

**Refactoring:**

```
Extract database connection logic to separate module

Move database connection code from api.py to db.py for better
separation of concerns and reusability across multiple endpoints.

No functional changes.
```

**Documentation:**

```
Add installation instructions to README

Include Python version requirements, dependency installation with pip,
and virtual environment setup steps.
```

**Performance improvement:**

```
Optimize query performance for user search

Add database index on users.username column and refactor query
to use parameterized search. Reduces search time from 2s to 50ms
for datasets with 100k+ users.
```

---

## Ignoring Files

### The `.gitignore` File

Create a `.gitignore` file to exclude files from version control:

```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
.venv/
pip-log.txt

# Virtual environments
ENV/
.env

# IDEs
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Project specific
data/
*.log
secrets.yml
.env.local
```

**Common patterns:**

- `*.log` - All files ending in .log
- `temp/` - Entire temp directory
- `/config.local` - config.local in root only
- `!important.log` - Exception to a previous pattern

---

## Git Configuration

### User Configuration

```bash
# Set name and email (required for commits)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default editor
git config --global core.editor "code --wait"

# View all configuration
git config --list

# View specific setting
git config user.name
```

### Useful Aliases

```bash
# Create shortcuts
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg 'log --oneline --graph --all'
```

Now use them:

```bash
git st        # Instead of git status
git lg        # Beautiful log graph
```

---

## Common Workflows

### Daily Development Flow

```bash
# 1. Start working
git status                          # Check current state
git pull origin main                # Get latest changes

# 2. Create feature branch
git checkout -b feature-add-search

# 3. Make changes
# ... edit files ...

# 4. Stage and commit
git add .
git commit -m "Add search functionality"

# 5. More changes and commits
# ... edit files ...
git commit -am "Fix search edge cases"

# 6. Ready to share
git push origin feature-add-search
```

### Experiment Safely

```bash
# Try something risky
git checkout -b experimental-refactor

# Make changes
# ... edit files ...

# If it works
git checkout main
git merge experimental-refactor

# If it doesn't work
git checkout main
git branch -D experimental-refactor  # Discard experiment
```

---

## Best Practices

### Commit Often, Perfect Later

- Make small, frequent commits while working
- Use `git rebase -i` to clean up history before sharing
- Each commit should be a logical unit of change

### Write Meaningful Commit Messages

- First line: Imperative mood ("Add feature" not "Added feature")
- 50 characters or less for summary
- Blank line, then detailed explanation
- Explain _why_, not just _what_

### Keep History Clean

- One commit per logical change
- Avoid commits like "fix typo", "oops", "forgot file"
- Use `git commit --amend` for small corrections
- Rebase to clean up before merging to main

### Never Commit

- Secrets (API keys, passwords, tokens)
- Large binary files (unless necessary)
- Generated files (compiled code, build artifacts)
- Environment-specific configs
- Sensitive data

---

## Troubleshooting

### "I committed to the wrong branch"

```bash
# Move last commit to correct branch
git log --oneline  # Note the commit hash
git checkout correct-branch
git cherry-pick abc123  # Apply commit to correct branch
git checkout wrong-branch
git reset --hard HEAD~1  # Remove from wrong branch
```

### "I want to undo my last commit but keep the changes"

```bash
git reset --soft HEAD~1
```

### "I made changes but want to discard everything"

```bash
git reset --hard HEAD
git clean -fd  # Remove untracked files
```

### "I need to see what changed in a commit"

```bash
git show abc123
```

### "Who changed this line?"

```bash
git blame filename.py
```

---

## Next Steps

Once you're comfortable with Git basics:

1. **[Git Workflows](git-workflows.md)** - Branching strategies, collaboration patterns
2. **[Remote Repositories](remote-repositories.md)** - GitHub, GitLab, pushing and pulling
3. **[CI/CD](ci-cd.md)** - Automation with GitHub Actions and GitLab CI

---

## Resources

### Official Documentation

- [Git Official Documentation](https://git-scm.com/doc)
- [Pro Git Book](https://git-scm.com/book/en/v2) (free online)

### Interactive Learning

- [Learn Git Branching](https://learngitbranching.js.org/) (interactive visualizations)
- [GitHub Git Handbook](https://guides.github.com/introduction/git-handbook/)

### References

- [Git Cheat Sheet](https://education.github.com/git-cheat-sheet-education.pdf)
- [Oh Shit, Git!?!](https://ohshitgit.com/) (fixing common mistakes)
