# Code Style and Standards

## PEP 8 Style Guide

### What is PEP 8?

PEP 8 is Python's official style guide, providing conventions for writing readable code:

```python
# Bad: Inconsistent style
def calculateTotal(item_list,tax_rate):
    total=0
    for item in item_list:
        total+=item['price']
    return total*tax_rate

# Good: PEP 8 compliant
def calculate_total(item_list, tax_rate):
    total = 0
    for item in item_list:
        total += item['price']
    return total * tax_rate
```

### Key Principles

```python
# 1. Use 4 spaces per indentation level
def function():
    if condition:
        do_something()

# 2. Maximum line length: 79 characters (88 for black)
# Break long lines logically
result = some_function_with_a_long_name(
    argument1, argument2,
    argument3, argument4
)

# 3. Two blank lines before top-level functions and classes
class MyClass:
    pass


def my_function():
    pass


# 4. One blank line between methods
class Example:
    def method_one(self):
        pass

    def method_two(self):
        pass
```

## Naming Conventions

### Variable and Function Names

```python
# Use snake_case for variables and functions
user_name = "Alice"
total_count = 42

def calculate_average(numbers):
    return sum(numbers) / len(numbers)

# Avoid single-letter names (except in specific contexts)
# Bad
a = 10
b = 20
c = a + b

# Good
price = 10
quantity = 20
total = price * quantity

# Single letters OK in list comprehensions and lambdas
squares = [x**2 for x in range(10)]
sorted_items = sorted(items, key=lambda x: x.name)
```

### Class Names

```python
# Use PascalCase for class names
class UserAccount:
    pass

class OrderProcessor:
    pass

class HTTPClient:  # Acronyms in uppercase
    pass

# Bad
class user_account:
    pass

class orderprocessor:
    pass
```

### Constants

```python
# Use UPPER_CASE for constants
MAX_CONNECTIONS = 100
DEFAULT_TIMEOUT = 30
API_BASE_URL = "https://api.example.com"

# Module-level constants at the top
DATABASE_HOST = "localhost"
DATABASE_PORT = 5432
```

### Private and Protected

```python
class Example:
    def __init__(self):
        self.public = "accessible"
        self._protected = "internal use"  # Single underscore
        self.__private = "name mangled"   # Double underscore

    def public_method(self):
        """Public API."""
        pass

    def _internal_method(self):
        """Not part of public API."""
        pass

    def __private_method(self):
        """Name mangled to _Example__private_method."""
        pass
```

### Special Names

```python
# Dunder methods (double underscore)
class MyClass:
    def __init__(self):
        pass

    def __str__(self):
        return "MyClass instance"

    def __repr__(self):
        return "MyClass()"

# Avoid single trailing underscore (reserved)
class_ = "value"  # Avoid keyword conflict
type_ = "string"

# Avoid double leading and trailing underscore (reserved for Python)
# Don't create: __custom__
```

## Code Layout

### Imports

```python
# Order: standard library, third-party, local
# Grouped with blank lines between

# Standard library
import os
import sys
from pathlib import Path

# Third-party
import numpy as np
import pandas as pd
from flask import Flask, request

# Local
from mypackage import module1
from mypackage.subpackage import module2

# Avoid wildcard imports
# Bad
from module import *

# Good
from module import function1, function2
# Or
import module
```

### Import Formatting

```python
# One import per line for 'import'
# Bad
import sys, os

# Good
import os
import sys

# Multiple items OK for 'from'
from typing import List, Dict, Optional

# Long imports can be split
from package.subpackage import (
    LongClassName1,
    LongClassName2,
    LongClassName3,
)
```

### Module Structure

```python
#!/usr/bin/env python3
"""Module docstring.

Detailed description of what this module does.
"""

# Imports
import os
from typing import List

# Constants
MAX_SIZE = 100
DEFAULT_VALUE = "default"

# Exception classes
class CustomError(Exception):
    pass

# Classes
class MyClass:
    pass

# Functions
def my_function():
    pass

# Main execution
if __name__ == '__main__':
    main()
```

## Whitespace and Formatting

### Operators and Delimiters

```python
# Surround binary operators with spaces
x = 1 + 2
result = value * multiplier
is_valid = x == y

# No space for high-priority operators
z = x*2 + y*3

# Keyword arguments and default parameters
def function(arg1, arg2=None, arg3=False):
    pass

# Function calls
result = function(arg1, arg2=value)

# No spaces inside brackets
my_list = [1, 2, 3]
my_dict = {'key': 'value'}
my_tuple = (1, 2, 3)

# Bad
my_list = [ 1, 2, 3 ]
my_dict = { 'key': 'value' }
```

### Line Continuation

```python
# Implicit continuation (preferred)
long_list = [
    'item1', 'item2', 'item3',
    'item4', 'item5', 'item6',
]

result = function_call(
    argument1,
    argument2,
    argument3,
)

# Hanging indent
result = some_function(
    first_argument, second_argument,
    third_argument, fourth_argument
)

# Backslash continuation (avoid when possible)
total = value1 + value2 + \
        value3 + value4
```

### Comments

```python
# Inline comments: two spaces before #
x = 5  # Initialize counter

# Block comments: # followed by space
# This is a comment explaining
# a complex piece of logic that
# spans multiple lines

# TODO comments
# TODO: Implement error handling
# FIXME: This breaks when input is None
# NOTE: This assumes input is sorted
```

## Code Readability Principles

### Explicit is Better Than Implicit

```python
# Bad: Implicit
def process(data):
    return [x for x in data if x]

# Good: Explicit
def process_non_empty_items(data):
    """Return only non-empty items from data."""
    return [item for item in data if item]
```

### Flat is Better Than Nested

```python
# Bad: Deep nesting
def process_order(order):
    if order:
        if order.is_valid():
            if order.items:
                if order.total > 0:
                    return process_payment(order)
    return None

# Good: Early returns
def process_order(order):
    if not order:
        return None

    if not order.is_valid():
        return None

    if not order.items:
        return None

    if order.total <= 0:
        return None

    return process_payment(order)
```

### Simple is Better Than Complex

```python
# Bad: Overly complex
result = reduce(lambda x, y: x + y,
                filter(lambda x: x > 0,
                       map(lambda x: x * 2, numbers)))

# Good: Simple and clear
doubled = [x * 2 for x in numbers]
positive = [x for x in doubled if x > 0]
result = sum(positive)

# Or even better
result = sum(x * 2 for x in numbers if x * 2 > 0)
```

### Readability Counts

```python
# Bad: Cryptic
def f(x, y, z):
    return (x + y) * z if z else x

# Good: Readable
def calculate_weighted_sum(base_value, additional_value, weight):
    """Calculate (base + additional) * weight, or just base if weight is 0."""
    if weight == 0:
        return base_value

    return (base_value + additional_value) * weight
```

## Automated Tools

Several tools help enforce style standards automatically:

**Key tools:**

- **black**: Opinionated code formatter
- **ruff**: Fast all-in-one linter and formatter
- **isort**: Import statement organizer
- **flake8**: Style guide enforcer
- **mypy**: Static type checker

```bash
# Quick example
pip install black ruff isort

# Format code
black myfile.py

# Lint and auto-fix
ruff check --fix myfile.py

# Sort imports
isort myfile.py
```

**Example configuration:**

```toml
# pyproject.toml
[tool.black]
line-length = 88

[tool.ruff]
line-length = 88
select = ["E", "F", "I"]  # Errors, pyflakes, isort

[tool.isort]
profile = "black"
```

> **For comprehensive tool coverage**, see:
>
> - [Code Quality Tools](../testing_and_quality/code-quality-tools.md) - Complete reference for all quality tools
> - [Type Checking](../testing_and_quality/type-checking.md) - Type checker configuration and usage

## Summary

Code style and standards in Python:

**PEP 8:**

- Official Python style guide
- 4 spaces for indentation
- 79-88 character line length
- Consistent formatting

**Naming conventions:**

- snake_case: variables, functions
- PascalCase: classes
- UPPER_CASE: constants
- \_protected, \_\_private: visibility

**Code layout:**

- Import order: stdlib, third-party, local
- Logical grouping with blank lines
- Consistent module structure

**Readability:**

- Explicit over implicit
- Flat over nested
- Simple over complex
- Self-documenting code

**Automated tools:**

- black, ruff: Code formatting and linting
- isort: Import organization
- flake8, mypy: Style and type checking
- See [Code Quality Tools](../testing_and_quality/code-quality-tools.md) for details

Consistency and readability are more important than strict adherence to any single rule.

## Next Steps

- Apply to [Writing Clean Code](writing-clean-code.md)
- Document with [Documentation](documentation.md)
- Structure with [Code Organization](code-organization.md)
