# Code Quality Tools

> **Comprehensive Reference:** This is the main guide for code quality tools. For style principles, see [Code Style and Standards](../patterns_and_practices/code-style-and-standards.md). For type hints syntax, see [Type Systems](../core_concepts/type-systems.md).

## Table of Contents

- [Linters](#linters)
- [Formatters](#formatters)
- [Type Checkers](#type-checkers)
- [Code Complexity](#code-complexity)
- [Pre-commit Hooks](#pre-commit-hooks)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Linters

**Linter**: A tool that analyzes code to find potential errors, style violations, and code smells without running the code.

### Ruff

```bash
# Install
pip install ruff

# Run linter
ruff check .

# Auto-fix issues
ruff check --fix .

# Ignore specific rules
ruff check --ignore E501,F401 .
```

```python
# Configuration: pyproject.toml
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "N",   # pep8-naming
    "UP",  # pyupgrade
]
ignore = ["E501"]  # Line too long

# Per-file ignores
[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]  # Unused imports OK in __init__.py
```

```python
# Example issues ruff catches

# Unused import
import os  # F401: unused import
import sys

# Undefined name
result = undefined_variable  # F821

# Unused variable
def func():
    x = 5  # F841: assigned but never used
    return 10

# Multiple statements on one line
x = 1; y = 2  # E702

# Missing whitespace
x=1+2  # E225: missing whitespace around operator

# Line too long (if not ignored)
very_long_line = "This is a very long line that exceeds the maximum line length limit"  # E501
```

### Pylint

```bash
# Install
pip install pylint

# Run pylint
pylint mymodule.py

# With configuration
pylint --rcfile=.pylintrc mymodule.py

# Disable specific warnings
pylint --disable=C0103,C0114 mymodule.py
```

```python
# Configuration: .pylintrc
[MASTER]
ignore=migrations
jobs=4

[MESSAGES CONTROL]
disable=C0103,  # Invalid name
        C0114,  # Missing module docstring
        R0903   # Too few public methods

[FORMAT]
max-line-length=88

[BASIC]
good-names=i,j,k,_,x,y,id

[DESIGN]
max-args=7
max-locals=15
```

```python
# Example issues pylint catches

# Invalid name
def MyFunction():  # C0103: Function name should be lowercase
    pass

# Missing docstring
class MyClass:  # C0115: Missing class docstring
    pass

# Too many arguments
def func(a, b, c, d, e, f, g, h):  # R0913: Too many arguments
    pass

# Unused argument
def func(x, y):  # W0613: Unused argument 'y'
    return x

# Duplicate code
def func1():
    x = 1
    y = 2
    z = 3
    return x + y + z

def func2():
    x = 1  # R0801: Similar lines in func1
    y = 2
    z = 3
    return x * y * z
```

### Flake8

```bash
# Install
pip install flake8

# Run flake8
flake8 .

# With specific plugins
pip install flake8-docstrings flake8-bugbear
flake8 .
```

```python
# Configuration: .flake8 or setup.cfg
[flake8]
max-line-length = 88
exclude = .git,__pycache__,migrations
ignore = E203,W503  # Conflicts with black

# Per-file ignores
per-file-ignores =
    __init__.py:F401
```

## Formatters

**Formatter**: A tool that automatically rewrites code to follow consistent style rules (spacing, indentation, line breaks).

### Black

```bash
# Install
pip install black

# Format files
black .

# Check without modifying
black --check .

# Show diff
black --diff myfile.py

# Format single file
black myfile.py
```

```python
# Configuration: pyproject.toml
[tool.black]
line-length = 88
target-version = ['py311']
include = '\.pyi?$'
exclude = '''
/(
    \.git
  | \.venv
  | migrations
)/
'''
```

```python
# Before black
def my_function(x,y,z):
    result=x+y+z
    if result>10:return True
    else:return False

# After black
def my_function(x, y, z):
    result = x + y + z
    if result > 10:
        return True
    else:
        return False

# Or more concisely:
def my_function(x, y, z):
    result = x + y + z
    return result > 10
```

### isort

```bash
# Install
pip install isort

# Sort imports
isort .

# Check without modifying
isort --check-only .

# Show diff
isort --diff myfile.py
```

```python
# Configuration: pyproject.toml
[tool.isort]
profile = "black"  # Compatible with black
line_length = 88
multi_line_output = 3
include_trailing_comma = true
```

```python
# Before isort
import sys
from myapp import models
import os
from typing import List
from myapp.utils import helper

# After isort
import os
import sys
from typing import List

from myapp import models
from myapp.utils import helper
```

### Ruff Format

```bash
# Ruff can also format (alternative to black)
ruff format .

# Check without modifying
ruff format --check .
```

## Type Checkers

**Type checker**: A static analysis tool that validates type hints in code to catch type-related bugs before runtime.

### Mypy

```bash
# Install
pip install mypy

# Run type checker
mypy .

# Check specific file
mypy mymodule.py

# Strict mode
mypy --strict mymodule.py
```

```python
# Configuration: mypy.ini or pyproject.toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

# Per-module configuration
[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false
```

```python
# Example type checking

# Type annotations
def greet(name: str) -> str:
    return f"Hello, {name}"

# Mypy catches type errors
greet(123)  # Error: Argument 1 has incompatible type "int"

# Optional types
from typing import Optional

def find_user(user_id: int) -> Optional[User]:
    user = database.get(user_id)
    return user if user else None

# Generic types
from typing import List, Dict

def process_items(items: List[str]) -> Dict[str, int]:
    return {item: len(item) for item in items}

# Protocol for structural subtyping
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()  # OK with any object that has draw()
```

### Pyright

```bash
# Install
npm install -g pyright
# Or use in VS Code with Pylance

# Run type checker
pyright

# Configuration in pyrightconfig.json
```

```json
// pyrightconfig.json
{
  "include": ["src"],
  "exclude": ["**/node_modules", "**/__pycache__"],
  "typeCheckingMode": "strict",
  "reportMissingImports": true,
  "reportMissingTypeStubs": false,
  "pythonVersion": "3.11"
}
```

## Code Complexity

**Code complexity**: A measure of how difficult code is to understand and maintain, often measured by number of decision paths (if/else, loops).

### Radon

```bash
# Install
pip install radon

# Cyclomatic complexity
radon cc mymodule.py

# With minimum threshold
radon cc mymodule.py -n B  # Show B grade and worse

# Maintainability index
radon mi mymodule.py

# Raw metrics
radon raw mymodule.py
```

```python
# Example complexity analysis

# Low complexity (A): 1-5
def simple_function(x):
    return x * 2

# Medium complexity (B): 6-10
def medium_function(x):
    if x > 0:
        if x > 10:
            return "high"
        return "low"
    return "negative"

# High complexity (C+): 11+
def complex_function(x, y, z):
    if x > 0:
        if y > 0:
            if z > 0:
                if x > y:
                    if y > z:
                        return "case1"
                    return "case2"
                return "case3"
            return "case4"
        return "case5"
    return "case6"
# Refactor high complexity functions!
```

### McCabe

```bash
# Install (comes with flake8)
pip install mccabe

# Check complexity with flake8
flake8 --max-complexity=10 mymodule.py
```

```python
# Configuration in .flake8
[flake8]
max-complexity = 10
```

## Pre-commit Hooks

**Pre-commit hook**: An automated script that runs quality checks (linting, formatting, tests) before code is committed to version control.

### Setup Pre-commit

```bash
# Install
pip install pre-commit

# Create .pre-commit-config.yaml
touch .pre-commit-config.yaml

# Install hooks
pre-commit install

# Run manually
pre-commit run --all-files
```

### Configuration

```yaml
# .pre-commit-config.yaml
repos:
  # Ruff
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.0
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  # Black (alternative to ruff-format)
  - repo: https://github.com/psf/black
    rev: 23.10.0
    hooks:
      - id: black

  # isort
  - repo: https://github.com/pycqa/isort
    rev: 5.12.0
    hooks:
      - id: isort

  # Mypy
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.6.0
    hooks:
      - id: mypy
        additional_dependencies: [types-requests]

  # Basic checks
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-json
      - id: check-added-large-files
      - id: check-merge-conflict
      - id: debug-statements

  # Security
  - repo: https://github.com/PyCQA/bandit
    rev: 1.7.5
    hooks:
      - id: bandit
        args: ["-c", "pyproject.toml"]
```

### Running Hooks

```bash
# Hooks run automatically on git commit
git add myfile.py
git commit -m "Update file"  # Hooks run here

# Run on all files
pre-commit run --all-files

# Run specific hook
pre-commit run black --all-files

# Skip hooks (not recommended)
git commit --no-verify -m "Skip hooks"

# Update hooks to latest versions
pre-commit autoupdate
```

### Custom Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: local
    hooks:
      - id: pytest
        name: pytest
        entry: pytest
        language: system
        pass_filenames: false
        always_run: true
        stages: [commit]

      - id: custom-script
        name: Custom validation
        entry: python scripts/validate.py
        language: system
        types: [python]
```

## Summary

Code quality tools:

**Linters:**

- Ruff: Fast, modern all-in-one linter
- Pylint: Comprehensive Python linter
- Flake8: Style guide enforcement
- Catch errors, style issues, complexity

**Formatters:**

- Black: Opinionated code formatter
- isort: Import statement organizer
- Ruff format: Alternative to black
- Consistent code style

**Type checkers:**

- Mypy: Static type checker
- Pyright: Fast type checker (VS Code)
- Catch type errors before runtime
- Better IDE support

**Complexity:**

- Radon: Complexity metrics
- McCabe: Cyclomatic complexity
- Identify code that needs refactoring
- Maintainability index

**Pre-commit hooks:**

- Automated quality checks
- Run before commits
- Prevent bad code from entering repo
- Combine multiple tools

Automated quality tools catch issues early and maintain consistent code standards.

## Next Steps

- Master [Type Checking](type-checking.md)
- Debug effectively with [Debugging Techniques](debugging-techniques.md)
- Explore [Tools and Ecosystem](../tools_and_ecosystem/README.md)
