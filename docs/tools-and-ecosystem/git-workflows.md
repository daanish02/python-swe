# Git Workflows

## About This Document

Git workflows are structured approaches to using Git in team environments. While Git provides the tools (branches, commits, merges), workflows provide the patterns for how to use those tools effectively. This document covers common branching strategies, collaboration patterns, and best practices for working with Git in teams.

The right workflow depends on team size, release cadence, and project needs. Understanding multiple workflows helps you choose or adapt the best approach for your context.

---

## Why Workflows Matter

### Problems Without Structure

Without a defined workflow:

- Unclear which branch to develop on
- Conflicts about when to merge
- Confusion about what's ready for production
- Difficulty tracking releases
- Merge conflicts and integration headaches

### Benefits of a Defined Workflow

1. **Clarity**: Everyone knows where to commit, when to branch, when to merge
2. **Stability**: Main/production branches stay stable
3. **Collaboration**: Reduced conflicts, clear integration points
4. **Release Management**: Controlled, predictable releases
5. **Code Quality**: Built-in review and testing checkpoints

---

## Feature Branch Workflow

### Overview

The simplest team workflow. All feature development happens in dedicated branches that are merged back to `main`.

**Branches:**

- `main` - Stable, deployable code
- `feature/*` - Individual features

### Workflow Steps

```bash
# 1. Start with updated main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature-user-auth

# 3. Develop and commit
git add .
git commit -m "Add login endpoint"
git commit -m "Add password hashing"

# 4. Push feature branch
git push origin feature-user-auth

# 5. Create pull request (on GitHub/GitLab)
# 6. After review and approval, merge to main
# 7. Delete feature branch
git branch -d feature-user-auth
git push origin --delete feature-user-auth
```

### Visual Representation

```
main:     A---B---C-------G---H
               \         /     \
feature-1:      D---E---F       \
                                 \
feature-2:                        I---J (in progress)
```

### When to Use

- Small to medium teams
- Continuous deployment
- Simple release process
- Short-lived features (days, not weeks)

### Pros and Cons

**Pros:**

- Simple and easy to understand
- Low overhead
- Encourages small, focused changes
- Works well with CI/CD

**Cons:**

- Less control over releases
- Main branch is always "latest" (may not be "stable")
- Harder to maintain multiple versions

---

## GitHub Flow

### Overview

A simplified workflow optimized for continuous deployment. Anything on `main` is deployable.

**Rules:**

1. `main` is always deployable
2. Create descriptive branch names
3. Commit often and push frequently
4. Open pull requests early for feedback
5. Deploy immediately after merge

### Workflow Steps

```bash
# 1. Create branch from main
git checkout main
git pull origin main
git checkout -b add-search-feature

# 2. Make changes and commit
git add .
git commit -m "Add basic search endpoint"
git push origin add-search-feature

# 3. Open pull request for early feedback
# (Continue development)

git commit -m "Add search filtering"
git push origin add-search-feature

# 4. Get review, iterate
# 5. Merge to main
# 6. Deploy immediately
# 7. Delete branch
```

### Branch Naming Conventions

```bash
# Feature
feature/user-profile
add-search-functionality

# Bug fix
fix/login-error
bugfix/null-pointer-in-search

# Hotfix
hotfix/security-vulnerability

# Refactor
refactor/database-connection

# Documentation
docs/api-documentation
```

### When to Use

- Web applications with continuous deployment
- Projects that can deploy multiple times per day
- Small teams with fast review cycles
- SaaS products

### Pros and Cons

**Pros:**

- Extremely simple
- Fast feedback through early PRs
- Encourages small, frequent deploys
- Great for continuous deployment

**Cons:**

- Requires excellent test coverage
- Need reliable automated testing
- Less control over release timing
- Challenging for scheduled releases

---

## Git Flow

### Overview

A structured workflow with multiple long-lived branches for handling releases, hotfixes, and ongoing development.

**Branches:**

- `main` - Production-ready code
- `develop` - Integration branch for features
- `feature/*` - Individual features
- `release/*` - Release preparation
- `hotfix/*` - Emergency production fixes

### Workflow Steps

**Feature Development:**

```bash
# 1. Create feature from develop
git checkout develop
git pull origin develop
git checkout -b feature/user-dashboard

# 2. Develop feature
git commit -m "Add dashboard layout"
git commit -m "Add dashboard widgets"

# 3. Finish feature
git checkout develop
git merge --no-ff feature/user-dashboard
git branch -d feature/user-dashboard
git push origin develop
```

**Release Preparation:**

```bash
# 1. Create release branch
git checkout develop
git checkout -b release/v1.2.0

# 2. Prepare release (version bumps, changelog, bug fixes)
git commit -m "Bump version to 1.2.0"
git commit -m "Update CHANGELOG"

# 3. Merge to main and develop
git checkout main
git merge --no-ff release/v1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"

git checkout develop
git merge --no-ff release/v1.2.0

# 4. Delete release branch
git branch -d release/v1.2.0
```

**Hotfix:**

```bash
# 1. Create hotfix from main
git checkout main
git checkout -b hotfix/security-patch

# 2. Fix critical issue
git commit -m "Fix SQL injection vulnerability"

# 3. Merge to main and develop
git checkout main
git merge --no-ff hotfix/security-patch
git tag -a v1.2.1 -m "Hotfix: Security patch"

git checkout develop
git merge --no-ff hotfix/security-patch

# 4. Delete hotfix branch
git branch -d hotfix/security-patch
```

### Visual Representation

```
main:     A-----------E-------G (v1.0)
           \         /       /
develop:    B---C---D---F---H---I---J
             \     /         \     /
feature-1:    K---L           \   /
                              M-N
                            feature-2
```

### When to Use

- Scheduled releases (monthly, quarterly)
- Teams maintaining multiple versions
- Enterprise software
- Projects requiring release branches

### Pros and Cons

**Pros:**

- Clear separation of development and production
- Structured release process
- Supports multiple versions
- Handles emergency hotfixes well

**Cons:**

- Complex for beginners
- More overhead than simpler workflows
- Two long-lived branches to maintain
- Overkill for continuous deployment

---

## Trunk-Based Development

### Overview

All developers commit to a single branch (`main`/`trunk`) with very short-lived feature branches or direct commits.

**Principles:**

- Feature branches live < 1 day
- Commit directly to main (with CI checks)
- Use feature flags for incomplete work
- Extremely frequent integration

### Workflow Steps

```bash
# Small change - direct to main
git checkout main
git pull origin main
# ... make small change ...
git commit -m "Fix typo in error message"
git push origin main
# (CI runs, auto-deploys if passing)

# Larger change - short-lived branch
git checkout -b quick-refactor
# ... make changes ...
git commit -m "Refactor user validation"
git push origin quick-refactor
# Open PR, get quick review, merge within hours
```

### Feature Flags

```python
# Using feature flags for incomplete features
if feature_flag_enabled('new_search_algorithm'):
    result = new_search(query)
else:
    result = old_search(query)
```

### When to Use

- Highly skilled teams
- Excellent automated testing
- Strong CI/CD pipeline
- Projects requiring rapid iteration

### Pros and Cons

**Pros:**

- Minimal merge conflicts
- Fast feedback
- Simple branching model
- True continuous integration

**Cons:**

- Requires mature engineering practices
- Need feature flags for incomplete work
- Requires excellent test coverage
- Can be scary for less experienced teams

---

## Forking Workflow

### Overview

Used primarily in open-source projects. Each developer has a server-side fork of the repository.

### Workflow Steps

```bash
# 1. Fork repository on GitHub/GitLab
# 2. Clone your fork
git clone https://github.com/yourusername/project.git
cd project

# 3. Add upstream remote
git remote add upstream https://github.com/original/project.git

# 4. Create feature branch
git checkout -b fix-documentation

# 5. Make changes and commit
git commit -m "Fix typos in README"

# 6. Push to your fork
git push origin fix-documentation

# 7. Open pull request to upstream

# 8. Keep fork updated
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### When to Use

- Open-source projects
- Large number of contributors
- Contributors without write access
- Public projects accepting external contributions

### Pros and Cons

**Pros:**

- No write access needed to contribute
- Clean separation between official and contributor repos
- Maintainer has full control
- Scales to many contributors

**Cons:**

- More complex setup
- Need to keep fork synchronized
- Additional remote to manage

---

## Pull Request Best Practices

### Creating Good Pull Requests

**Title:**

```
Add user authentication with JWT

Fix null pointer exception in data loader

Refactor database connection module
```

**Description Template:**

```markdown
## What This PR Does

Brief summary of changes

## Why

Explanation of motivation

## How to Test

1. Step-by-step testing instructions
2. Expected behavior

## Screenshots (if UI changes)

[images]

## Checklist

- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No breaking changes (or documented)
- [ ] Follows code style guidelines

Closes #123
```

### Code Review Guidelines

**As Author:**

- Keep PRs small (< 400 lines when possible)
- One logical change per PR
- Self-review before requesting review
- Respond to all comments
- Don't take feedback personally

**As Reviewer:**

- Review promptly (within 24 hours)
- Focus on important issues first
- Be specific and constructive
- Ask questions rather than demand changes
- Approve when acceptable, not perfect

### Common PR Patterns

**Draft PR for early feedback:**

```bash
# Open draft PR while still developing
git push origin feature-branch
# Mark PR as draft
# Continue development
# Mark ready for review when complete
```

**WIP (Work In Progress) commits:**

```bash
# Quick save point
git commit -m "WIP: Experimenting with new approach"

# Later, clean up before review
git rebase -i HEAD~3
# Squash WIP commits into meaningful commits
```

---

## Merge Strategies

### Merge Commit (--no-ff)

```bash
git merge --no-ff feature-branch
```

**Result:**

```
main:   A---B-------E (merge commit)
             \     /
feature:      C---D
```

**Pros:** Preserves complete history, clear feature boundaries
**Cons:** Cluttered history with many branches

### Fast-Forward (default)

```bash
git merge feature-branch  # Fast-forward if possible
```

**Result:**

```
main:   A---B---C---D (linear history)
```

**Pros:** Clean linear history
**Cons:** Loses information about feature boundaries

### Squash Merge

```bash
git merge --squash feature-branch
git commit -m "Add complete feature"
```

**Result:**

```
main:   A---B---C (single commit with all feature changes)
```

**Pros:** Clean history, one commit per feature
**Cons:** Loses individual commit history from feature branch

### Rebase

```bash
git checkout feature-branch
git rebase main
```

**Result:**

```
Before:
main:     A---B---C
               \
feature:        D---E

After:
main:     A---B---C
                   \
feature:            D'---E' (rebased)
```

**Pros:** Clean linear history, no merge commits
**Cons:** Rewrites history (don't rebase public branches)

### When to Use Each

| Strategy          | Use Case                                                      |
| ----------------- | ------------------------------------------------------------- |
| **Merge --no-ff** | Preserving feature history, release branches                  |
| **Fast-forward**  | Simple updates, single commits                                |
| **Squash**        | Feature branches with messy history, one feature = one commit |
| **Rebase**        | Keeping feature branch updated, cleaning up before merge      |

---

## Practical Examples

### Example 1: Simple Feature

```bash
# Day 1: Start feature
git checkout main
git pull origin main
git checkout -b feature-add-pagination

# Commit 1
git add pagination.py
git commit -m "Add pagination utility functions"

# Commit 2
git add api.py
git commit -m "Integrate pagination in user list endpoint"

# Push for backup
git push origin feature-add-pagination

# Day 2: Continue
git add tests/test_pagination.py
git commit -m "Add pagination tests"

# Ready to merge
git push origin feature-add-pagination
# Open PR, get review, merge
```

### Example 2: Keeping Feature Branch Updated

```bash
# Start feature
git checkout -b feature-large-refactor

# Work for a few days
git commit -m "Refactor user model"
git commit -m "Update user endpoints"

# Meanwhile, main has advanced
# Option 1: Merge main into feature (preserves history)
git checkout feature-large-refactor
git merge main

# Option 2: Rebase feature onto main (cleaner history)
git checkout feature-large-refactor
git rebase main
# Resolve conflicts if any
git push --force-with-lease origin feature-large-refactor
```

### Example 3: Emergency Hotfix

```bash
# Production bug discovered!
git checkout main
git pull origin main
git checkout -b hotfix-critical-bug

# Quick fix
git add fixed_file.py
git commit -m "Fix critical null pointer exception"

# Test locally
python -m pytest

# Push and create PR marked as hotfix
git push origin hotfix-critical-bug
# Fast-track review
# Merge and deploy immediately

# Clean up
git checkout main
git pull origin main
git branch -d hotfix-critical-bug
```

---

## Collaborative Patterns

### Pair Programming with Git

```bash
# Technique 1: Shared branch
git checkout -b feature-pair-work
# Both developers push to same branch
git pull --rebase origin feature-pair-work
# ... make changes ...
git push origin feature-pair-work

# Technique 2: Co-authored commits
git commit -m "Implement search feature

Co-authored-by: Partner Name <partner@email.com>"
```

### Handling Conflicts

```bash
# Conflict during merge
git merge feature-branch
# CONFLICT in file.py

# 1. Check conflict
git status

# 2. Open file, resolve conflicts
# <<<<<<< HEAD
# Your changes
# =======
# Their changes
# >>>>>>> feature-branch

# 3. Stage resolved file
git add file.py

# 4. Complete merge
git commit

# 5. Push
git push origin main
```

### Reviewing Someone's Work Locally

```bash
# Fetch all branches
git fetch origin

# Check out PR branch
git checkout -b review-feature origin/feature-branch

# Test locally
python -m pytest

# Return to your branch
git checkout main
git branch -d review-feature
```

---

## Choosing a Workflow

### Decision Matrix

| Project Type                            | Recommended Workflow          |
| --------------------------------------- | ----------------------------- |
| Solo project                            | Feature Branch or GitHub Flow |
| Small team (2-5), continuous deployment | GitHub Flow                   |
| Small team, scheduled releases          | Feature Branch                |
| Medium team (5-15), scheduled releases  | Git Flow                      |
| Large team, rapid releases              | Trunk-Based Development       |
| Open source                             | Forking Workflow              |

### Questions to Ask

1. **How often do you release?**
   - Multiple times/day → Trunk-Based or GitHub Flow
   - Weekly/monthly → Feature Branch or Git Flow

2. **Do you need to support multiple versions?**
   - Yes → Git Flow
   - No → Simpler workflows

3. **How large is your team?**
   - 1-5 → Simple workflows
   - 10+ → More structure needed

4. **What's your testing maturity?**
   - Excellent automated tests → Trunk-Based
   - Manual testing → More controlled workflows

---

## Next Steps

- **[Version Control](version-control.md)** - Git fundamentals and core commands
- **[Remote Repositories](remote-repositories.md)** - Working with GitHub and GitLab
- **[CI/CD](ci-cd.md)** - Automating workflows with GitHub Actions

---

## Resources

- [Git Flow Original Post](https://nvie.com/posts/a-successful-git-branching-model/)
- [GitHub Flow Guide](https://guides.github.com/introduction/flow/)
- [Trunk Based Development](https://trunkbaseddevelopment.com/)
- [Atlassian Git Workflows](https://www.atlassian.com/git/tutorials/comparing-workflows)
