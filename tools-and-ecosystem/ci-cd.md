# CI/CD (Continuous Integration / Continuous Deployment)

## Table of Contents

- [Why CI/CD?](#why-cicd)
- [CI/CD Concepts](#cicd-concepts)
- [GitHub Actions](#github-actions)
- [Common GitHub Actions Workflows](#common-github-actions-workflows)
- [GitLab CI/CD](#gitlab-cicd)
- [Common Pipeline Patterns](#common-pipeline-patterns)
- [Environment Variables and Secrets](#environment-variables-and-secrets)
- [Best Practices](#best-practices)
- [Debugging CI/CD](#debugging-cicd)
- [Advanced Patterns](#advanced-patterns)
- [Practical Examples](#practical-examples)
- [Next Steps](#next-steps)
- [Resources](#resources)

## About This Document

CI/CD automates the process of testing, building, and deploying software. Continuous Integration (CI) automatically tests every code change. Continuous Deployment (CD) automatically deploys passing changes to production. This document covers CI/CD concepts, GitHub Actions, GitLab CI, and practical automation patterns.

Modern software development is impossible without automation. CI/CD transforms how teams ship software - from manual, error-prone processes to automated, reliable pipelines.

---

## Why CI/CD?

### Problems Without Automation

**Manual testing and deployment:**

- "It works on my machine" syndrome
- Inconsistent testing coverage
- Human error in deployment
- Long time between code and production
- Fear of deploying (because it's risky)
- Integration problems discovered late

### Benefits of CI/CD

1. **Fast Feedback**: Bugs caught minutes after commit, not days
2. **Consistency**: Same tests run every time, same deployment process
3. **Confidence**: Automated tests give confidence to deploy
4. **Speed**: Deploy multiple times per day instead of per month
5. **Quality**: Can't merge broken code (if configured properly)
6. **Documentation**: Pipeline as code documents your process

---

## CI/CD Concepts

### Continuous Integration (CI)

Automatically test every code change:

```
Developer → Git Push → CI System → Run Tests → Report Results
```

**Key practices:**

- Commit frequently (at least daily)
- Every commit triggers automated tests
- Keep the build fast (< 10 minutes)
- Fix broken builds immediately
- Test in production-like environment

### Continuous Deployment (CD)

Automatically deploy passing changes:

```
Tests Pass → Build Artifacts → Deploy to Staging → Deploy to Production
```

**Key practices:**

- Every commit to main can potentially deploy
- Automated deployment pipeline
- Zero-downtime deployments
- Easy rollback mechanism
- Monitoring and alerting

### Continuous Delivery vs Deployment

**Continuous Delivery**: Code is _always_ ready to deploy (manual trigger)
**Continuous Deployment**: Code _automatically_ deploys to production

```
Continuous Delivery:
  Commit → Test → Build → [Manual Approval] → Deploy

Continuous Deployment:
  Commit → Test → Build → Deploy (automatic)
```

---

## GitHub Actions

### Overview

GitHub Actions is GitHub's built-in CI/CD platform. Configuration is done in YAML files stored in `.github/workflows/`.

### Basic Workflow Structure

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

# When to run
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

# Jobs to run
jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov

      - name: Run tests
        run: pytest --cov=.
```

### Workflow Components

**Triggers (`on`):**

```yaml
# On push to specific branches
on:
  push:
    branches: [ main ]

# On pull request
on:
  pull_request:

# On schedule (cron)
on:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight

# Manual trigger
on:
  workflow_dispatch:

# Multiple triggers
on:
  push:
    branches: [ main ]
  pull_request:
  workflow_dispatch:
```

**Jobs:**

```yaml
jobs:
  # Job 1
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: pytest

  # Job 2 (runs in parallel by default)
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Run linter
        run: ruff check .

  # Job 3 (depends on test)
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: ./deploy.sh
```

**Runners:**

- `ubuntu-latest` - Ubuntu Linux
- `macos-latest` - macOS
- `windows-latest` - Windows
- Self-hosted runners for custom environments

---

## Common GitHub Actions Workflows

### Python Testing

```yaml
name: Python Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11", "3.12"]

    steps:
      - uses: actions/checkout@v3

      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
          pip install pytest pytest-cov ruff mypy

      - name: Lint with ruff
        run: ruff check .

      - name: Type check with mypy
        run: mypy .

      - name: Test with pytest
        run: |
          pytest --cov=. --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
```

### Linting and Formatting

```yaml
name: Code Quality

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"

      - name: Install linters
        run: |
          pip install ruff black isort mypy

      - name: Check formatting with black
        run: black --check .

      - name: Check import order with isort
        run: isort --check-only .

      - name: Lint with ruff
        run: ruff check .

      - name: Type check with mypy
        run: mypy .
```

### Build and Deploy

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      - run: |
          pip install -r requirements.txt
          pytest

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Deploy to production
        env:
          DEPLOY_KEY: ${{ secrets.DEPLOY_KEY }}
        run: |
          ./scripts/deploy.sh
```

### Matrix Testing

```yaml
name: Matrix Testing

on: [push, pull_request]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: ["3.9", "3.10", "3.11"]

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - run: |
          pip install -r requirements.txt
          pytest
```

---

## GitLab CI/CD

### Overview

GitLab has built-in CI/CD. Configuration is in `.gitlab-ci.yml` at repository root.

### Basic Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  image: python:3.11
  script:
    - pip install -r requirements.txt
    - pytest
  only:
    - merge_requests
    - main

build:
  stage: build
  script:
    - python setup.py build
  artifacts:
    paths:
      - dist/

deploy:
  stage: deploy
  script:
    - ./deploy.sh
  only:
    - main
  environment:
    name: production
```

### GitLab Pipeline Components

**Stages**: Defines execution order

```yaml
stages:
  - test # Runs first
  - build # Runs after test
  - deploy # Runs after build
```

**Jobs**: Tasks within stages

```yaml
unit_test:
  stage: test
  script:
    - pytest tests/unit

integration_test:
  stage: test
  script:
    - pytest tests/integration
# Both run in parallel (same stage)
```

**Images**: Docker images for jobs

```yaml
test:
  image: python:3.11-slim
  script:
    - pytest
```

**Artifacts**: Files passed between jobs

```yaml
build:
  script:
    - python setup.py bdist_wheel
  artifacts:
    paths:
      - dist/*.whl
    expire_in: 1 week
```

---

## Common Pipeline Patterns

### Testing Pipeline

```yaml
# GitHub Actions
name: Testing Pipeline

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      - run: |
          pip install -r requirements.txt
          pytest tests/unit

  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      - run: |
          pip install -r requirements.txt
          pytest tests/integration
```

### Multi-Stage Deployment

```yaml
# GitLab CI
stages:
  - test
  - deploy-staging
  - deploy-production

test:
  stage: test
  script:
    - pytest

deploy-staging:
  stage: deploy-staging
  script:
    - ./deploy.sh staging
  only:
    - main
  environment:
    name: staging
    url: https://staging.example.com

deploy-production:
  stage: deploy-production
  script:
    - ./deploy.sh production
  only:
    - main
  when: manual # Requires manual approval
  environment:
    name: production
    url: https://example.com
```

### Caching Dependencies

```yaml
# GitHub Actions
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"
          cache: "pip"

      - run: pip install -r requirements.txt
      - run: pytest

# GitLab CI
test:
  image: python:3.11
  cache:
    paths:
      - .pip-cache
  before_script:
    - pip install --cache-dir .pip-cache -r requirements.txt
  script:
    - pytest
```

---

## Environment Variables and Secrets

### GitHub Actions Secrets

**Adding secrets:**

1. Repository → Settings → Secrets → New repository secret
2. Add name and value (e.g., `API_KEY`, `DEPLOY_TOKEN`)

**Using secrets:**

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy with secret
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
        run: |
          echo "Deploying..."
          ./deploy.sh
```

### GitLab CI/CD Variables

**Adding variables:**
Settings → CI/CD → Variables → Add Variable

**Using variables:**

```yaml
deploy:
  script:
    - echo "API_KEY is $API_KEY"
    - ./deploy.sh
  variables:
    ENVIRONMENT: production
```

**Protected variables**: Only available on protected branches

---

## Best Practices

### Pipeline Design

1. **Keep pipelines fast**: < 10 minutes ideal

   ```yaml
   # Run fast tests first
   stages:
     - quick-tests # Unit tests (< 2 min)
     - slow-tests # Integration tests (< 10 min)
     - deploy
   ```

2. **Fail fast**: Run quickest tests first

   ```yaml
   - name: Quick syntax check
     run: python -m py_compile *.py

   - name: Full test suite
     run: pytest
   ```

3. **Parallelize when possible**:

   ```yaml
   strategy:
     matrix:
       test-group: [unit, integration, e2e]
   ```

4. **Cache dependencies**: Don't re-download every time
   ```yaml
   - uses: actions/cache@v3
     with:
       path: ~/.cache/pip
       key: ${{ runner.os }}-pip-${{ hashFiles('requirements.txt') }}
   ```

### Testing Strategy

```yaml
# Comprehensive testing pipeline
jobs:
  lint:
    # Fast: syntax and style (< 1 min)
    runs-on: ubuntu-latest
    steps:
      - run: ruff check .
      - run: black --check .

  type-check:
    # Fast: type checking (< 2 min)
    runs-on: ubuntu-latest
    steps:
      - run: mypy .

  unit-tests:
    # Medium: unit tests (< 5 min)
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/unit

  integration-tests:
    # Slow: integration tests (< 15 min)
    needs: [lint, type-check, unit-tests]
    runs-on: ubuntu-latest
    steps:
      - run: pytest tests/integration
```

### Security

1. **Never log secrets**:

   ```yaml
   # BAD
   - run: echo "Password is ${{ secrets.PASSWORD }}"

   # GOOD
   - run: echo "Deploying with credentials..."
   ```

2. **Use secrets for sensitive data**:

   ```yaml
   env:
     API_KEY: ${{ secrets.API_KEY }} # NOT hardcoded
   ```

3. **Restrict secret access**: Use environment protection rules

4. **Scan for vulnerabilities**:
   ```yaml
   - name: Security scan
     run: |
       pip install safety
       safety check
   ```

---

## Debugging CI/CD

### Common Issues

**"Tests pass locally but fail in CI":**

- Different Python version
- Missing environment variables
- Different dependencies
- Timezone differences
- File path issues (Windows vs Linux)

**Solution:**

```yaml
# Match local environment
- uses: actions/setup-python@v4
  with:
    python-version: "3.11.5" # Exact version

# Print debug info
- run: |
    python --version
    pip list
    printenv | sort
```

**"Pipeline is too slow":**

```yaml
# Before: 20 minutes
- run: pip install -r requirements.txt # 3 min every time

# After: 2 minutes
- uses: actions/cache@v3 # Cache dependencies
  with:
    path: ~/.cache/pip
    key: ${{ hashFiles('requirements.txt') }}
```

### Debugging Workflows

```yaml
# Enable debug logging
- name: Debug
  run: |
    echo "Event: ${{ github.event_name }}"
    echo "Ref: ${{ github.ref }}"
    echo "Actor: ${{ github.actor }}"
    ls -la
    env | sort
```

---

## Advanced Patterns

### Monorepo with Selective Testing

```yaml
name: Monorepo CI

on: [push]

jobs:
  changes:
    runs-on: ubuntu-latest
    outputs:
      backend: ${{ steps.filter.outputs.backend }}
      frontend: ${{ steps.filter.outputs.frontend }}
    steps:
      - uses: actions/checkout@v3
      - uses: dorny/paths-filter@v2
        id: filter
        with:
          filters: |
            backend:
              - 'backend/**'
            frontend:
              - 'frontend/**'

  test-backend:
    needs: changes
    if: needs.changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: pytest backend/

  test-frontend:
    needs: changes
    if: needs.changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
      - run: npm test
```

### Deployment with Approval

```yaml
# GitLab CI - Manual approval
deploy-production:
  stage: deploy
  script:
    - ./deploy.sh
  when: manual
  only:
    - main

# GitHub Actions - Environment protection rules
deploy:
  runs-on: ubuntu-latest
  environment:
    name: production
    # Requires approval from designated reviewers
  steps:
    - run: ./deploy.sh
```

### Matrix Strategy

```yaml
strategy:
  matrix:
    python-version: [3.9, 3.10, 3.11, 3.12]
    os: [ubuntu-latest, macos-latest, windows-latest]
    exclude:
      - os: macos-latest
        python-version: 3.9
  fail-fast: false # Don't cancel other jobs if one fails

steps:
  - uses: actions/setup-python@v4
    with:
      python-version: ${{ matrix.python-version }}
```

---

## Practical Examples

### Simple Python Project

```yaml
name: Python CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Check out code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"
          cache: "pip"

      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest ruff

      - name: Lint
        run: ruff check .

      - name: Test
        run: pytest
```

### Full Production Pipeline

```yaml
name: Production Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      - run: pip install ruff black mypy
      - run: black --check .
      - run: ruff check .
      - run: mypy .

  test:
    needs: quality
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.9", "3.10", "3.11"]
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - run: pip install -r requirements.txt pytest pytest-cov
      - run: pytest --cov=. --cov-report=xml
      - uses: codecov/codecov-action@v3

  deploy:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v3
      - name: Deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: ./scripts/deploy.sh
```

---

## Next Steps

- **[Version Control](version-control.md)** - Git fundamentals
- **[Git Workflows](git-workflows.md)** - Branching strategies
- **[Remote Repositories](remote-repositories.md)** - GitHub and GitLab collaboration

---

## Resources

### Documentation

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)

### Learning

- [GitHub Actions Tutorial](https://docs.github.com/en/actions/learn-github-actions)
- [GitLab CI/CD Examples](https://docs.gitlab.com/ee/ci/examples/)
- [Awesome Actions](https://github.com/sdras/awesome-actions)

### Tools

- [Act](https://github.com/nektos/act) - Run GitHub Actions locally
- [GitLab Runner](https://docs.gitlab.com/runner/) - Self-hosted GitLab CI
