# Virtual Environments

## Table of Contents

- [Why Virtual Environments Matter](#why-virtual-environments-matter)
- [Using venv](#using-venv)
- [virtualenv vs venv](#virtualenv-vs-venv)
- [Conda Environments](#conda-environments)
- [Modern Alternatives: uv](#modern-alternatives-uv)
- [Environment Management Strategies](#environment-management-strategies)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Why Virtual Environments Matter

**Virtual environment**: An isolated Python environment with its own set of packages, separate from the system Python.

### The Problem Without Virtual Environments

```bash
# System-wide installation (BAD)
pip install django==3.2
pip install requests==2.25.0

# Another project needs different versions
pip install django==4.2  # Overwrites Django 3.2!
pip install requests==2.31.0  # Overwrites requests 2.25.0!

# Result: Project 1 breaks because it needs Django 3.2
```

### Benefits of Virtual Environments

```bash
# Isolation:
# - Each project has its own packages
# - No version conflicts between projects
# - Safe to experiment without breaking system

# Reproducibility:
# - Lock exact package versions
# - Share requirements.txt with team
# - Everyone gets identical environment

# Clean system:
# - Don't pollute system Python
# - Easy to delete (just remove folder)
# - Start fresh anytime
```

## Using venv

**venv**: Python's built-in tool for creating virtual environments (available since Python 3.3).

### Creating Virtual Environments

```bash
# Create virtual environment
python -m venv myenv

# Or with specific Python version
python3.11 -m venv myenv

# Directory structure created:
myenv/
├── bin/           # Unix: Scripts and executables
├── Scripts/       # Windows: Scripts and executables
├── include/       # C headers
├── lib/           # Installed packages
└── pyvenv.cfg     # Configuration
```

### Activating and Deactivating

```bash
# Activate (Unix/macOS)
source myenv/bin/activate

# Activate (Windows Command Prompt)
myenv\Scripts\activate.bat

# Activate (Windows PowerShell)
myenv\Scripts\Activate.ps1

# After activation, prompt changes:
(myenv) user@machine:~/project$

# Check Python location
which python  # Unix
where python  # Windows
# Shows: /path/to/myenv/bin/python

# Deactivate
deactivate
```

### Using the Virtual Environment

```bash
# After activation:

# Install packages (goes to myenv, not system)
pip install requests

# Check installed packages
pip list
# Only shows packages in this environment

# Run Python
python script.py
# Uses myenv's Python and packages

# Generate requirements.txt
pip freeze > requirements.txt
```

### Common Workflow

```bash
# 1. Create project directory
mkdir myproject
cd myproject

# 2. Create virtual environment
python -m venv venv

# 3. Activate
source venv/bin/activate  # Unix
venv\Scripts\activate     # Windows

# 4. Install dependencies
pip install -r requirements.txt
# Or install packages individually
pip install requests pandas flask

# 5. Work on project
python app.py

# 6. Save dependencies
pip freeze > requirements.txt

# 7. Deactivate when done
deactivate
```

### Ignoring venv in Git

```bash
# .gitignore
venv/
env/
.env/
.venv/
ENV/
*.pyc
__pycache__/
```

## virtualenv vs venv

**virtualenv**: Third-party tool (predecessor to venv) with more features and faster creation.

### Comparison

```bash
# venv (built-in)
python -m venv myenv
# Pros: Built-in, no installation needed
# Cons: Slower, fewer options

# virtualenv (third-party)
pip install virtualenv
virtualenv myenv
# Pros: Faster, more features, works on older Python
# Cons: Requires installation
```

### virtualenv Features

```bash
# Install
pip install virtualenv

# Create environment
virtualenv myenv

# Faster creation
virtualenv --always-copy myenv

# Specify Python version
virtualenv -p python3.11 myenv
virtualenv --python=/usr/bin/python3.11 myenv

# Without pip (minimal)
virtualenv --no-pip myenv

# With system packages accessible
virtualenv --system-site-packages myenv
```

### When to Use Which

```bash
# Use venv when:
# - Python 3.3+ available
# - Standard workflow sufficient
# - Don't want extra dependencies

# Use virtualenv when:
# - Need faster environment creation
# - Working with Python 2.7 (legacy)
# - Need advanced options
# - Want --system-site-packages
```

## Conda Environments

**Conda**: Package and environment manager from Anaconda, handling Python and non-Python dependencies.

### Installing Conda

```bash
# Download Miniconda (minimal)
# https://docs.conda.io/en/latest/miniconda.html

# Or Anaconda (full, includes many packages)
# https://www.anaconda.com/download

# Verify installation
conda --version
```

### Creating Conda Environments

```bash
# Create environment
conda create -n myenv python=3.11

# Activate
conda activate myenv

# Deactivate
conda deactivate

# List environments
conda env list
# Output:
# base                     /home/user/miniconda3
# myenv                  * /home/user/miniconda3/envs/myenv

# Remove environment
conda remove -n myenv --all
```

### Installing Packages with Conda

```bash
# Install from conda repositories
conda install numpy pandas

# Install specific version
conda install numpy=1.24.0

# Install from conda-forge (community channel)
conda install -c conda-forge requests

# Mix conda and pip (use conda first)
conda install numpy pandas
pip install some-package-only-on-pypi
```

### environment.yml

```yaml
# environment.yml
name: myproject
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.11
  - numpy=1.24.3
  - pandas=2.0.2
  - scikit-learn=1.3.0
  - pip
  - pip:
      - requests==2.31.0
      - flask==2.3.2
```

```bash
# Create from environment.yml
conda env create -f environment.yml

# Update existing environment
conda env update -f environment.yml

# Export environment
conda env export > environment.yml

# Export without builds (more portable)
conda env export --no-builds > environment.yml

# Export minimal (only explicitly installed)
conda env export --from-history > environment.yml
```

### Conda vs venv

```bash
# Conda advantages:
# - Installs non-Python dependencies (C libraries, system tools)
# - Better for data science (numpy, scipy, pandas optimized)
# - Cross-platform binary packages
# - Can manage multiple Python versions easily

# venv advantages:
# - Built into Python
# - Faster
# - Simpler
# - Better pip compatibility

# Use conda for:
# - Data science projects
# - Need system libraries
# - Complex dependencies

# Use venv for:
# - Web development
# - Pure Python projects
# - Simpler dependencies
```

## Modern Alternatives: uv

**uv**: Extremely fast Python package and environment manager written in Rust.

### Creating Environments with uv

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# Create virtual environment
uv venv

# Or with specific name
uv venv myenv

# Or with specific Python version
uv venv --python 3.11

# Activate (same as venv)
source .venv/bin/activate  # Unix
.venv\Scripts\activate     # Windows
```

### Installing Packages with uv

```bash
# After activating environment:

# Install packages (10-100x faster than pip)
uv pip install requests

# Install from requirements.txt
uv pip install -r requirements.txt

# Sync exact dependencies (remove unused)
uv pip sync requirements.txt

# Compile dependencies with resolution
uv pip compile requirements.in -o requirements.txt
```

### uv Advantages

```bash
# Speed:
# - 10-100x faster than pip
# - Parallel downloads
# - Better caching

# Reliability:
# - Better dependency resolution
# - Avoids conflicts

# Compatibility:
# - Drop-in replacement for pip
# - Works with existing tools

# Example speed comparison:
# pip install pandas numpy scipy scikit-learn
# → 45 seconds

# uv pip install pandas numpy scipy scikit-learn
# → 3 seconds
```

## Environment Management Strategies

### Per-Project Environments

```bash
# Best practice: One environment per project

myproject/
├── venv/              # Virtual environment
├── src/
├── tests/
├── requirements.txt
└── README.md

anotherproject/
├── venv/              # Different environment
├── src/
├── requirements.txt
└── README.md

# Each project isolated
# No conflicts
# Easy to manage
```

### Naming Conventions

```bash
# Common environment names:
venv        # Most common
.venv       # Hidden directory
env
virtualenv

# Use .venv for:
# - Hidden from ls (Unix)
# - Standard VS Code detection
# - Consistent across projects

# Create .venv
python -m venv .venv
source .venv/bin/activate
```

### Environment in IDEs

```bash
# VS Code (.vscode/settings.json)
{
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
  "python.terminal.activateEnvironment": true
}

# PyCharm:
# - File → Settings → Project → Python Interpreter
# - Add Interpreter → Existing Environment
# - Select .venv/bin/python
```

### Multiple Python Versions

```bash
# Using pyenv + venv
pyenv install 3.11.0
pyenv local 3.11.0
python -m venv venv

# Using virtualenv
virtualenv -p python3.11 venv

# Using conda
conda create -n myenv python=3.11
```

### Sharing Environments

```bash
# 1. Generate requirements.txt
pip freeze > requirements.txt

# 2. Commit to Git
git add requirements.txt
git commit -m "Add requirements"

# 3. Team members recreate
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# For conda:
conda env export > environment.yml
# Team: conda env create -f environment.yml
```

### Cleaning Up

```bash
# Remove virtual environment
deactivate  # First deactivate
rm -rf venv  # Unix
rmdir /s venv  # Windows

# Recreate if needed
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# With conda
conda remove -n myenv --all
```

### Docker Alternative

```dockerfile
# Dockerfile (alternative to virtual environments)
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "app.py"]
```

```bash
# Build and run
docker build -t myapp .
docker run myapp

# Pros:
# - Complete isolation (OS level)
# - Reproducible across any machine
# - Includes system dependencies

# Cons:
# - More complex
# - Slower iteration during development
# - Larger footprint
```

## Summary

Virtual environments provide isolation for Python projects:

**Why use them:**

- Avoid version conflicts between projects
- Keep system Python clean
- Easy reproducibility
- Safe experimentation

**venv (built-in):**

- `python -m venv myenv` - Create environment
- `source myenv/bin/activate` - Activate (Unix)
- `myenv\Scripts\activate` - Activate (Windows)
- `deactivate` - Deactivate
- Built-in, simple, sufficient for most projects

**virtualenv (third-party):**

- Faster environment creation
- More options
- Works with older Python versions
- Install with `pip install virtualenv`

**Conda:**

- Manages Python and non-Python dependencies
- Great for data science
- `conda create -n myenv python=3.11`
- `conda activate myenv`
- Use environment.yml for sharing

**uv (modern):**

- 10-100x faster than pip
- `uv venv` - Create environment
- `uv pip install` - Install packages
- Drop-in pip replacement

**Best practices:**

- One environment per project
- Name it `.venv` for consistency
- Add to .gitignore
- Share via requirements.txt or environment.yml
- Delete and recreate when needed

Always use virtual environments -- never install packages system-wide unless absolutely necessary.

## Next Steps

- Learn [Package Management](package-management.md)
- Review [Python Setup](python-setup.md)
- Explore [Running Python](running-python.md)
