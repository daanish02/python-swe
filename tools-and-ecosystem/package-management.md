# Package Management

## Table of Contents

- [Using pip](#using-pip)
- [Requirements Files](#requirements-files)
- [Understanding PyPI](#understanding-pypi)
- [Modern Tools: poetry and uv](#modern-tools-poetry-and-uv)
- [Managing Dependencies](#managing-dependencies)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Using pip

**pip**: Python's package installer, used to install and manage third-party libraries from PyPI (Python Package Index).

### Basic pip Commands

```bash
# Install a package
pip install requests

# Install specific version
pip install requests==2.28.0

# Install minimum version
pip install requests>=2.28.0

# Install version range
pip install requests>=2.28.0,<3.0.0

# Install latest
pip install --upgrade requests

# Uninstall
pip uninstall requests

# List installed packages
pip list

# Show package details
pip show requests

# Search packages (deprecated, use https://pypi.org)
# pip search requests
```

### Installing from Different Sources

```bash
# From PyPI (default)
pip install requests

# From Git repository
pip install git+https://github.com/psf/requests.git

# From specific branch
pip install git+https://github.com/psf/requests.git@main

# From specific commit
pip install git+https://github.com/psf/requests.git@abc123

# From local directory
pip install ./mypackage

# From local directory in editable mode
pip install -e ./mypackage

# From wheel file
pip install mypackage-1.0.0-py3-none-any.whl

# From requirements file
pip install -r requirements.txt
```

### Editable Installs

```bash
# Install package in editable/development mode
pip install -e .

# Useful during development:
# - Changes in source code immediately available
# - No need to reinstall after every change
# - Common for local package development
```

```python
# Project structure
myproject/
├── src/
│   └── mypackage/
│       ├── __init__.py
│       └── module.py
├── tests/
├── pyproject.toml
└── setup.py

# Install in editable mode
cd myproject
pip install -e .

# Now can import mypackage anywhere
# Changes to src/ immediately take effect
```

## Requirements Files

**requirements.txt**: A file listing all Python packages needed for a project, making it easy to recreate the environment.

### Basic requirements.txt

```txt
# requirements.txt
requests==2.28.2
numpy>=1.24.0
pandas>=2.0.0,<3.0.0
flask
```

```bash
# Install all requirements
pip install -r requirements.txt

# Generate requirements.txt from current environment
pip freeze > requirements.txt
```

### requirements-dev.txt

```txt
# requirements-dev.txt
# Development dependencies only

# Include production dependencies
-r requirements.txt

# Testing
pytest>=7.0.0
pytest-cov>=4.0.0
pytest-mock>=3.10.0

# Linting and formatting
ruff>=0.1.0
black>=23.0.0
mypy>=1.0.0

# Development tools
ipython>=8.0.0
ipdb>=0.13.0
```

```bash
# Install development dependencies
pip install -r requirements-dev.txt
```

### Pinning Versions

```txt
# Exact versions (most strict)
requests==2.28.2
numpy==1.24.3

# Minimum version
requests>=2.28.0

# Compatible release (recommended)
requests~=2.28.0  # Allows 2.28.x but not 2.29.0

# Version range
requests>=2.28.0,<3.0.0

# Any version (not recommended for production)
requests
```

### Comments and Organization

```txt
# requirements.txt

# Web framework
flask==2.3.0
flask-cors==4.0.0

# Database
sqlalchemy==2.0.15
psycopg2-binary==2.9.6

# Data processing
pandas==2.0.2
numpy==1.24.3

# Utilities
python-dotenv==1.0.0
requests==2.31.0
```

## Understanding PyPI

**PyPI (Python Package Index)**: The official repository of Python packages, hosting hundreds of thousands of open-source libraries.

### Browsing PyPI

```bash
# Visit: https://pypi.org

# Search for packages
# - Check package popularity (downloads, GitHub stars)
# - Read documentation
# - Check last update date
# - Review dependencies
```

### Package Information

```bash
# View package details
pip show requests

# Output:
# Name: requests
# Version: 2.31.0
# Summary: Python HTTP for Humans.
# Home-page: https://requests.readthedocs.io
# Author: Kenneth Reitz
# License: Apache 2.0
# Location: /path/to/site-packages
# Requires: charset-normalizer, idna, urllib3, certifi
# Required-by: package1, package2
```

### Security Considerations

```bash
# Check for known vulnerabilities
pip install safety
safety check

# Or use pip-audit (official tool)
pip install pip-audit
pip-audit
```

```bash
# Example output:
# requests (2.25.0): CVE-2021-1234
# - Vulnerability in older version
# - Upgrade to 2.28.0 or higher
```

## Modern Tools: poetry and uv

### poetry

**poetry**: A modern dependency management and packaging tool, combining pip and requirements.txt with better dependency resolution.

```bash
# Install poetry
curl -sSL https://install.python-poetry.org | python3 -

# Create new project
poetry new myproject

# Initialize existing project
poetry init

# Add dependency
poetry add requests

# Add development dependency
poetry add --group dev pytest

# Install all dependencies
poetry install

# Update dependencies
poetry update

# Show dependencies
poetry show

# Run script in poetry environment
poetry run python script.py
```

```toml
# pyproject.toml (created by poetry)
[tool.poetry]
name = "myproject"
version = "0.1.0"
description = ""
authors = ["Your Name <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.9"
requests = "^2.28.0"
pandas = "^2.0.0"

[tool.poetry.group.dev.dependencies]
pytest = "^7.0.0"
black = "^23.0.0"
ruff = "^0.1.0"
```

### uv

**uv**: An extremely fast Python package installer and resolver, written in Rust (10-100x faster than pip).

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Or with pip
pip install uv

# Use uv like pip
uv pip install requests

# Install from requirements.txt
uv pip install -r requirements.txt

# Create virtual environment
uv venv

# Sync exact dependencies (lockfile)
uv pip sync requirements.txt

# Compile requirements (with dependency resolution)
uv pip compile requirements.in -o requirements.txt
```

```txt
# requirements.in (high-level dependencies)
requests
pandas>=2.0.0
flask

# Generate locked requirements.txt
uv pip compile requirements.in -o requirements.txt
# Creates requirements.txt with ALL transitive dependencies pinned
```

### pyproject.toml with uv

```toml
# pyproject.toml
[project]
name = "myproject"
version = "0.1.0"
requires-python = ">=3.9"
dependencies = [
    "requests>=2.28.0",
    "pandas>=2.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "ruff>=0.1.0",
]
```

```bash
# Install project dependencies with uv
uv pip install -e .

# Install with dev dependencies
uv pip install -e ".[dev]"
```

## Managing Dependencies

### Dependency Resolution

```bash
# Problem: Conflicting versions
# Package A requires requests>=2.25.0
# Package B requires requests<2.28.0
# Solution: requests==2.27.1 (satisfies both)

# pip does basic resolution
# poetry and uv do advanced resolution

# Check dependency tree
pip install pipdeptree
pipdeptree

# Example output:
# requests==2.31.0
#   - certifi [required: >=2017.4.17]
#   - charset-normalizer [required: >=2,<4]
#   - idna [required: >=2.5,<4]
#   - urllib3 [required: >=1.21.1,<3]
```

### Upgrading Packages

```bash
# Upgrade single package
pip install --upgrade requests

# Upgrade all packages (NOT recommended)
pip list --outdated
pip install --upgrade package1 package2 ...

# Better: Use poetry or uv for safe upgrades
poetry update
uv pip compile requirements.in -o requirements.txt --upgrade
```

### Lock Files

**Lock file**: A file recording exact versions of all dependencies, ensuring reproducible builds.

```bash
# pip-tools (generates lock files from requirements.in)
pip install pip-tools

# requirements.in (your dependencies)
requests
pandas>=2.0.0

# Generate lockfile
pip-compile requirements.in
# Creates requirements.txt with pinned versions

# Install from lockfile
pip-sync requirements.txt
```

```txt
# requirements.txt (generated)
requests==2.31.0
  # via -r requirements.in
pandas==2.0.2
  # via -r requirements.in
numpy==1.24.3
  # via pandas
python-dateutil==2.8.2
  # via pandas
pytz==2023.3
  # via pandas
```

### Best Practices

```bash
# 1. Separate production and development dependencies
requirements.txt      # Production
requirements-dev.txt  # Development

# 2. Pin exact versions in production
requests==2.31.0  # Not requests>=2.31.0

# 3. Use compatible release operator for flexibility
requests~=2.31.0  # Allows 2.31.x

# 4. Document why specific versions are pinned
# requests==2.28.0  # Pinned due to bug in 2.29.0

# 5. Regularly update and test
pip list --outdated
# Review and update one at a time

# 6. Use lock files for reproducibility
# pip-compile, poetry.lock, uv.lock

# 7. Check security
pip-audit
safety check
```

### Handling Conflicts

```bash
# Conflict example
pip install package-a  # Requires numpy<1.24
pip install package-b  # Requires numpy>=1.24
# ERROR: Cannot install incompatible versions

# Solutions:

# 1. Update packages
pip install --upgrade package-a package-b

# 2. Find compatible versions
# Check package documentation

# 3. Use different packages
# Find alternatives with compatible dependencies

# 4. Create separate environments
# Use virtual environments for conflicting projects
```

## Summary

Package management in Python:

**pip basics:**

- `pip install package` - Install from PyPI
- `pip install -r requirements.txt` - Install from file
- `pip freeze` - List installed packages
- `pip install -e .` - Editable install for development

**requirements.txt:**

- Lists project dependencies
- requirements-dev.txt for development tools
- Pin versions for reproducibility
- Use comments for documentation

**PyPI:**

- Central repository for Python packages
- Check popularity, documentation, and updates
- Use safety/pip-audit for security checks

**Modern tools:**

- poetry: All-in-one dependency management
- uv: Extremely fast pip replacement
- Both provide better dependency resolution
- Support lock files for reproducibility

**Dependency management:**

- Pin exact versions in production
- Use lock files (pip-compile, poetry, uv)
- Separate production and dev dependencies
- Regularly check for updates and security issues
- Handle conflicts with virtual environments

Choose tools based on project needs: pip for simplicity, poetry for monorepos, uv for speed.

## Next Steps

- Set up [Virtual Environments](virtual-environments.md)
- Review [Python Setup](python-setup.md)
- Explore [Code Quality Tools](../testing_and_quality/code-quality-tools.md)
