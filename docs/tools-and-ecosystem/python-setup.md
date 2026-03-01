# Python Setup and Installation

## Installing Python

**Python installation**: The process of getting the Python interpreter on your system so you can run Python code.

### Official Installation

```bash
# Download from python.org
# Windows: python-3.11.x-amd64.exe
# macOS: python-3.11.x-macosx11.pkg
# Linux: Usually pre-installed or via package manager

# Verify installation
python --version
# or
python3 --version
```

### What Gets Installed

```bash
# Python interpreter
python  # or python3

# Package manager
pip  # or pip3

# Standard library
# - Built-in modules (os, sys, pathlib, etc.)
# - Located in Python installation directory

# IDLE (optional)
# - Basic Python IDE
# - Useful for beginners
```

### Installation Options

```bash
# Windows installer options:
# ✓ Add Python to PATH (important!)
# ✓ Install pip
# ✓ Install for all users (optional)
# ✓ Precompile standard library

# Verify PATH setup
where python  # Windows
which python  # Unix/macOS

# Should show Python executable location
# C:\Users\...\Python311\python.exe (Windows)
# /usr/local/bin/python3 (macOS/Linux)
```

## Python Version Management

**Version manager**: A tool that lets you install and switch between multiple Python versions on one system.

### pyenv (Unix/macOS/WSL)

```bash
# Install pyenv
# macOS:
brew install pyenv

# Linux:
curl https://pyenv.run | bash

# List available versions
pyenv install --list

# Install specific version
pyenv install 3.11.5
pyenv install 3.12.0

# List installed versions
pyenv versions

# Set global Python version
pyenv global 3.11.5

# Set version for current directory
pyenv local 3.12.0  # Creates .python-version file

# Set version for current shell session
pyenv shell 3.11.5
```

### py launcher (Windows)

```bash
# Built into Python 3.3+
# Automatically manages multiple Python versions

# List installed versions
py --list

# Run specific version
py -3.11 script.py
py -3.12 script.py

# Use latest Python 3
py -3 script.py

# Install package with specific version
py -3.11 -m pip install requests

# Interactive shell with specific version
py -3.12
```

### asdf (Universal)

```bash
# Install asdf
# Follow: https://asdf-vm.com/guide/getting-started.html

# Add Python plugin
asdf plugin add python

# Install Python version
asdf install python 3.11.5

# Set global version
asdf global python 3.11.5

# Set local version (per directory)
asdf local python 3.12.0  # Creates .tool-versions
```

### uv (Modern, Fast)

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh  # Unix/macOS
irm https://astral.sh/uv/install.ps1 | iex        # Windows

# Or with pip
pip install uv

# Install Python versions
uv python install 3.11
uv python install 3.12.0
uv python install 3.13

# List installed versions
uv python list

# Pin Python version for project
uv python pin 3.11
# Creates .python-version file

# Use specific Python version
uv run --python 3.11 script.py

# Install with uv and create environment in one step
uv venv --python 3.11

# Advantages:
# - Extremely fast (10-100x faster than pip)
# - Manages both Python versions and packages
# - No separate pyenv needed
# - Unified tool for environment + package management
```

## System Python vs User Python

### System Python

```bash
# Pre-installed on macOS/Linux
# Usually /usr/bin/python or /usr/bin/python3

# DON'T:
# - Modify system Python
# - Install packages globally with sudo pip
# - Upgrade/downgrade system Python

# Risks:
# - Break system tools
# - Cause OS instability
# - Permission issues
```

### User Python

```bash
# Installed by you
# Managed independently from system

# Best practices:
# 1. Install separate Python from python.org or pyenv
# 2. Use virtual environments
# 3. Never use sudo with pip

# Check which Python you're using
which python3  # Should NOT be /usr/bin/python3 when working

# Use user Python
# Windows: C:\Users\...\Python311\python.exe
# macOS: /usr/local/bin/python3 or ~/.pyenv/...
# Linux: /usr/local/bin/python3 or ~/.pyenv/...
```

### --user Flag

```bash
# Install package for current user only
pip install --user requests

# Package location:
# ~/.local/lib/python3.11/site-packages (Unix)
# %APPDATA%\Python\Python311\site-packages (Windows)

# Better alternative: Use virtual environments!
```

## Platform-Specific Installation

### Windows

```bash
# Method 1: Official installer from python.org
# - Download .exe installer
# - Check "Add Python to PATH"
# - Install

# Method 2: Microsoft Store
# - Search "Python 3.11"
# - Click Install
# - Auto-updates

# Method 3: Winget (Windows Package Manager)
winget install Python.Python.3.11

# Method 4: Chocolatey
choco install python311

# Verify
py --version
python --version

# Common locations:
# %LOCALAPPDATA%\Programs\Python\Python311
# C:\Python311
```

### macOS

```bash
# Method 1: Official installer from python.org
# - Download .pkg installer
# - Run installer
# - Installs to /usr/local/bin/

# Method 2: Homebrew (recommended)
brew install python@3.11
brew install python@3.12

# Creates symlinks
# /usr/local/bin/python3
# /usr/local/bin/pip3

# Method 3: pyenv (best for multiple versions)
brew install pyenv
pyenv install 3.11.5

# Verify
python3 --version
which python3

# Note: macOS includes system Python in /usr/bin/
# Don't use it! Use Homebrew or pyenv version
```

### Linux

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install python3.11 python3.11-venv python3-pip

# Fedora/RHEL
sudo dnf install python3.11

# Arch
sudo pacman -S python

# Build from source (if needed)
sudo apt install build-essential zlib1g-dev libssl-dev
wget https://www.python.org/ftp/python/3.11.5/Python-3.11.5.tgz
tar -xzf Python-3.11.5.tgz
cd Python-3.11.5
./configure --enable-optimizations
make -j$(nproc)
sudo make altinstall  # altinstall preserves system Python

# Verify
python3.11 --version
which python3.11
```

### Docker

```dockerfile
# Dockerfile
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

# Interactive Python in container
docker run -it python:3.11 python
```

## Choosing Python Versions

### Version Numbers Explained

```bash
# Format: MAJOR.MINOR.PATCH
# Example: 3.11.5

# MAJOR (3): Major language changes, backward incompatible
# MINOR (11): New features, mostly backward compatible
# PATCH (5): Bug fixes, fully backward compatible
```

### Active Python Versions

```bash
# As of 2024:
# Python 3.12 - Latest stable (Oct 2023)
# Python 3.11 - Stable, fast (Oct 2022)
# Python 3.10 - Stable (Oct 2021)
# Python 3.9 - Stable (Oct 2020)
# Python 3.8 - Security fixes only
# Python 3.7 and earlier - End of life

# Check support status: https://devguide.python.org/versions/
```

### Which Version to Use?

```bash
# For new projects:
# ✓ Python 3.11 or 3.12 (latest stable)
# - Best performance (3.11+ is significantly faster)
# - Latest features
# - Long support

# For production:
# ✓ Python 3.10 or 3.11
# - Mature, tested
# - Good library support
# - LTS support available

# For compatibility:
# ✓ Python 3.9+
# - If supporting older systems
# - Check dependency requirements

# Avoid:
# ✗ Python 3.7 and earlier (end of life)
# ✗ Python 2.x (deprecated since 2020)
```

### Checking Compatibility

```python
# Check minimum Python version in code
import sys

if sys.version_info < (3, 9):
    raise RuntimeError("Python 3.9+ required")

# Specify in pyproject.toml
# [project]
# requires-python = ">=3.9"

# Or in setup.py
# python_requires=">=3.9"
```

### Feature Availability by Version

```python
# Python 3.10+: Pattern matching
match value:
    case 1:
        print("one")
    case _:
        print("other")

# Python 3.9+: Type hints improvements
def greet(name: str) -> str:
    items: list[str] = []  # No need for List from typing

# Python 3.8+: Assignment expressions
if (n := len(items)) > 10:
    print(f"Too many items: {n}")

# Python 3.7+: Dataclasses
from dataclasses import dataclass

@dataclass
class User:
    name: str
    age: int
```

## Summary

Python installation and setup:

**Installation methods:**

- Official installer from python.org
- Package managers (brew, apt, winget, choco)
- pyenv for version management
- uv for fast, modern installation
- Docker for containerized environments

**Version management:**

- uv (all platforms) - fastest, modern, unified tool
- pyenv (Unix/macOS/WSL) - most flexible
- py launcher (Windows) - built-in
- asdf (universal) - manages multiple languages

**Best practices:**

- Don't modify system Python
- Use virtual environments
- Install user Python separately
- Never use sudo with pip

**Choosing versions:**

- Python 3.11+ for new projects (fast!)
- Python 3.10+ for production stability
- Check library compatibility
- Avoid Python 3.7 and earlier

**Platform considerations:**

- Windows: Use official installer or py launcher
- macOS: Use Homebrew or pyenv, avoid system Python
- Linux: Use package manager or build from source
- Docker: Official Python images for containers

Proper Python setup is the foundation for all development work.

## Next Steps

- Set up [Development Environments](development-environments.md)
- Learn [Running Python Code](running-python.md)
- Master [Virtual Environments](virtual-environments.md)
