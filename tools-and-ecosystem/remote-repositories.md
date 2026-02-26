# Remote Repositories

## Table of Contents

- [Understanding Remotes](#understanding-remotes)
- [Remote Commands](#remote-commands)
- [Pushing and Pulling](#pushing-and-pulling)
- [GitHub](#github)
- [GitLab](#gitlab)
- [Collaboration Workflows](#collaboration-workflows)
- [Branch Protection](#branch-protection)
- [Tags and Releases](#tags-and-releases)
- [Working with Multiple Remotes](#working-with-multiple-remotes)
- [Cloning Strategies](#cloning-strategies)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Platform Comparison](#platform-comparison)
- [Next Steps](#next-steps)
- [Resources](#resources)

## About This Document

Remote repositories are hosted versions of your Git repository that enable collaboration, backup, and deployment. GitHub, GitLab, and Bitbucket are the most popular platforms for hosting remote repositories. This document covers working with remotes, collaboration workflows, platform-specific features, and best practices.

Understanding remotes transforms Git from a local version control system into a powerful collaboration platform.

---

## Understanding Remotes

### What is a Remote?

A remote is a version of your repository hosted on a server (GitHub, GitLab, your own server). It's a reference to another copy of your repository that you can push to and pull from.

**Common remotes:**

- `origin` - Default name for your main remote (convention)
- `upstream` - Original repository (in forking workflow)
- `production` - Deployment server
- Custom names for any other remotes

### Local vs Remote

```
Your Computer:          Remote Server (GitHub):
  .git/                   your-repo.git
  main branch             main branch
  feature branch          feature branch
  commits                 commits
```

---

## Remote Commands

### Viewing Remotes

```bash
# List remotes
git remote

# List with URLs
git remote -v

# Show details about a remote
git remote show origin
```

### Adding Remotes

```bash
# Add a remote
git remote add origin https://github.com/username/repo.git

# Add with SSH
git remote add origin git@github.com:username/repo.git

# Add upstream (forking workflow)
git remote add upstream https://github.com/original/repo.git
```

### Changing Remotes

```bash
# Change remote URL
git remote set-url origin https://github.com/username/new-repo.git

# Rename remote
git remote rename origin github

# Remove remote
git remote remove origin
```

---

## Pushing and Pulling

### Pushing Changes

```bash
# Push current branch to origin
git push origin main

# Push and set upstream (first time)
git push -u origin main
# Now you can just use: git push

# Push all branches
git push origin --all

# Push all tags
git push origin --tags

# Force push (DANGEROUS - rewrites history)
git push --force origin main

# Safer force push (fails if remote changed)
git push --force-with-lease origin main

# Delete remote branch
git push origin --delete feature-branch
```

### Pulling Changes

```bash
# Pull changes from origin/main
git pull origin main

# Pull with rebase instead of merge
git pull --rebase origin main

# Pull from configured upstream
git pull

# Fetch without merging
git fetch origin

# Fetch all remotes
git fetch --all

# Fetch and prune deleted branches
git fetch --prune
```

### Fetch vs Pull

**`git fetch`**: Downloads changes but doesn't merge

```bash
git fetch origin
git log origin/main  # View remote changes
git merge origin/main  # Merge when ready
```

**`git pull`**: Downloads and merges in one step

```bash
git pull origin main
# Equivalent to:
# git fetch origin
# git merge origin/main
```

---

## GitHub

### Repository Setup

**Creating a new repository:**

1. Go to github.com → New Repository
2. Choose name, visibility (public/private)
3. Optional: README, .gitignore, license

**Connecting local repository:**

```bash
# Method 1: New local repo
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/username/repo.git
git push -u origin main

# Method 2: Clone existing repo
git clone https://github.com/username/repo.git
```

### Authentication

**HTTPS (using Personal Access Token):**

```bash
git clone https://github.com/username/repo.git
# Username: your-username
# Password: ghp_yourPersonalAccessToken
```

**SSH (recommended):**

```bash
# 1. Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# 2. Add to SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# 3. Add public key to GitHub
# Copy ~/.ssh/id_ed25519.pub
# GitHub → Settings → SSH and GPG keys → New SSH key

# 4. Clone with SSH
git clone git@github.com:username/repo.git
```

### Pull Requests (PRs)

**Creating a PR:**

```bash
# 1. Push feature branch
git checkout -b feature-new-feature
git commit -m "Implement new feature"
git push origin feature-new-feature

# 2. On GitHub:
# - Go to repository
# - Click "Compare & pull request"
# - Fill in title and description
# - Assign reviewers
# - Submit PR
```

**Updating a PR:**

```bash
# Make more commits
git commit -m "Address review feedback"
git push origin feature-new-feature
# PR automatically updates
```

**PR Review Workflow:**

```bash
# As reviewer, check out PR locally
git fetch origin pull/123/head:pr-123
git checkout pr-123
# Test changes
# Return to your branch
git checkout main
```

### Issues

**Linking commits to issues:**

```bash
# Reference issue in commit
git commit -m "Fix login bug

Fixes #42"

# Or use keywords: Fixes, Closes, Resolves
git commit -m "Add search feature (Closes #15)"
```

### GitHub CLI

```bash
# Install GitHub CLI: gh

# Authenticate
gh auth login

# Clone repository
gh repo clone username/repo

# Create PR
gh pr create --title "New Feature" --body "Description"

# List PRs
gh pr list

# Check out PR
gh pr checkout 123

# View PR
gh pr view 123

# Merge PR
gh pr merge 123

# Create issue
gh issue create --title "Bug" --body "Description"

# List issues
gh issue list
```

---

## GitLab

### Repository Setup

**Similar to GitHub:**

1. Create repository on gitlab.com
2. Connect local repository

```bash
git remote add origin https://gitlab.com/username/project.git
git push -u origin main
```

### Authentication

**Personal Access Token:**

```bash
# Settings → Access Tokens → Create personal access token
# Use as password when pushing

# Or configure credential helper
git config --global credential.helper store
```

**SSH:**

```bash
# Same as GitHub - add SSH key to GitLab
# Settings → SSH Keys
```

### Merge Requests (MRs)

GitLab's equivalent to Pull Requests.

```bash
# Create branch and push
git checkout -b feature-branch
git push origin feature-branch

# On GitLab:
# - Create Merge Request
# - Assign reviewers, set labels
# - Link to issues
```

**GitLab-specific features:**

- Merge request approvals
- Merge request dependencies
- Draft merge requests

### GitLab CI/CD Integration

```yaml
# .gitlab-ci.yml
stages:
  - test
  - deploy

test:
  stage: test
  script:
    - python -m pytest

deploy:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
```

---

## Collaboration Workflows

### Fork and Pull Request

**For open-source contributions:**

```bash
# 1. Fork repository on GitHub/GitLab

# 2. Clone your fork
git clone https://github.com/yourusername/repo.git
cd repo

# 3. Add upstream remote
git remote add upstream https://github.com/original/repo.git

# 4. Create feature branch
git checkout -b fix-typo

# 5. Make changes and commit
git commit -m "Fix typo in README"

# 6. Push to your fork
git push origin fix-typo

# 7. Create pull request
# From your fork to upstream

# 8. Keep fork synchronized
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### Shared Repository

**For team projects:**

```bash
# 1. Clone shared repository
git clone https://github.com/team/project.git

# 2. Create feature branch
git checkout -b feature-auth

# 3. Make changes and push
git push origin feature-auth

# 4. Create PR
# No fork needed - branch in same repository

# 5. After merge, update local main
git checkout main
git pull origin main
git branch -d feature-auth
```

### Keeping Branch Updated

```bash
# Update feature branch with latest main
git checkout feature-branch
git fetch origin
git merge origin/main
# Or: git rebase origin/main

# Resolve conflicts if any
git push origin feature-branch
```

---

## Branch Protection

### GitHub Branch Protection Rules

**Settings → Branches → Add rule:**

- **Require pull request reviews**: Force code review before merge
- **Require status checks**: CI must pass
- **Require signed commits**: GPG signature verification
- **Include administrators**: Apply rules to admins too
- **Restrict who can push**: Limit direct pushes

```bash
# Now direct push to main fails
git push origin main
# Error: protected branch requires pull request
```

### GitLab Protected Branches

**Settings → Repository → Protected Branches:**

- Master/main branch protected by default
- Configure who can merge and push
- Require MR approval

---

## Tags and Releases

### Creating Tags

```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag specific commit
git tag -a v1.0.0 abc123 -m "Release 1.0.0"

# Push tags to remote
git push origin v1.0.0

# Push all tags
git push origin --tags

# List tags
git tag
git tag -l "v1.*"

# Delete tag locally
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0
```

### GitHub Releases

```bash
# 1. Create and push tag
git tag -a v1.0.0 -m "Version 1.0.0"
git push origin v1.0.0

# 2. On GitHub: Releases → Create a new release
# - Choose tag
# - Add release notes
# - Attach binaries (optional)
# - Publish

# Or using GitHub CLI
gh release create v1.0.0 --notes "Release notes"
```

---

## Working with Multiple Remotes

### Common Scenarios

**Open source contribution:**

```bash
# Your fork
git remote add origin https://github.com/you/repo.git

# Original repository
git remote add upstream https://github.com/original/repo.git

# Fetch from both
git fetch origin
git fetch upstream

# Push to your fork
git push origin feature-branch

# Pull from upstream
git pull upstream main
```

**Multiple deployment environments:**

```bash
git remote add origin https://github.com/team/app.git
git remote add staging https://staging-server.com/app.git
git remote add production https://production-server.com/app.git

# Deploy to staging
git push staging main

# Deploy to production
git push production main
```

---

## Cloning Strategies

### Clone Types

**Full clone:**

```bash
git clone https://github.com/username/repo.git
# Downloads complete history
```

**Shallow clone (faster):**

```bash
# Clone with limited history
git clone --depth 1 https://github.com/username/repo.git

# Clone single branch
git clone --single-branch --branch main https://github.com/username/repo.git
```

**Clone to specific directory:**

```bash
git clone https://github.com/username/repo.git my-project
```

**Clone with submodules:**

```bash
git clone --recursive https://github.com/username/repo.git
```

---

## Best Practices

### Remote Workflow Best Practices

1. **Pull before push**: Always get latest changes first

   ```bash
   git pull origin main
   git push origin main
   ```

2. **Use SSH for authentication**: More secure and convenient

   ```bash
   git clone git@github.com:username/repo.git
   ```

3. **Never force push to shared branches**: Rewrites history

   ```bash
   # NEVER on main/shared branches
   git push --force origin main

   # OK on your feature branch
   git push --force-with-lease origin feature-branch
   ```

4. **Delete merged branches**: Keep repository clean

   ```bash
   git branch -d feature-branch
   git push origin --delete feature-branch
   ```

5. **Use meaningful branch names**: Clear, descriptive

   ```bash
   # Good
   git checkout -b fix-login-validation

   # Bad
   git checkout -b temp
   ```

### Security Best Practices

1. **Never commit secrets**:

   ```bash
   # Add to .gitignore
   .env
   secrets.yml
   *.key
   credentials.json
   ```

2. **Use environment variables**:

   ```python
   import os
   API_KEY = os.getenv('API_KEY')  # Not hardcoded
   ```

3. **Rotate exposed credentials immediately**:
   If you accidentally commit a secret:

   ```bash
   # 1. Rotate/change the secret
   # 2. Remove from history (complex - better to prevent)
   # 3. Force push (if private repo and necessary)
   ```

4. **Use signed commits** (GPG):

   ```bash
   # Configure GPG
   git config --global user.signingkey YOUR_GPG_KEY
   git config --global commit.gpgsign true

   # Sign commit
   git commit -S -m "Signed commit"
   ```

---

## Troubleshooting

### "Failed to push - rejected"

```bash
# Remote has changes you don't have locally
# Solution: Pull first
git pull origin main
# Resolve conflicts if any
git push origin main
```

### "Repository not found"

```bash
# Check remote URL
git remote -v

# Update URL if wrong
git remote set-url origin https://github.com/username/correct-repo.git

# Verify access permissions
```

### "Permission denied (publickey)"

```bash
# SSH key not configured
# 1. Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# 2. Add to SSH agent
ssh-add ~/.ssh/id_ed25519

# 3. Add public key to GitHub/GitLab

# Test connection
ssh -T git@github.com
```

### Syncing Fork

```bash
# Fork is behind upstream
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

---

## Platform Comparison

| Feature                | GitHub               | GitLab                  | Bitbucket           |
| ---------------------- | -------------------- | ----------------------- | ------------------- |
| **Hosting**            | Cloud, Enterprise    | Cloud, Self-hosted      | Cloud, Data Center  |
| **CI/CD**              | GitHub Actions       | GitLab CI/CD (built-in) | Bitbucket Pipelines |
| **Code Review**        | Pull Requests        | Merge Requests          | Pull Requests       |
| **Issue Tracking**     | Issues               | Issues, Epics           | Jira integration    |
| **Project Management** | Projects, Milestones | Boards, Roadmaps        | Jira integration    |
| **Free Private Repos** | Unlimited            | Unlimited               | Up to 5 users       |
| **CLI**                | `gh`                 | `glab`                  | -                   |

---

## Next Steps

- **[Version Control](version-control.md)** - Git fundamentals
- **[Git Workflows](git-workflows.md)** - Collaboration patterns
- **[CI/CD](ci-cd.md)** - Automation with GitHub Actions and GitLab CI

---

## Resources

### Documentation

- [GitHub Docs](https://docs.github.com/)
- [GitLab Docs](https://docs.gitlab.com/)
- [GitHub CLI Manual](https://cli.github.com/manual/)

### Guides

- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Forking Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/forking-workflow)
- [SSH Key Setup](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
