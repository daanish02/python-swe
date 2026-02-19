# Modules and Imports

## Table of Contents

- [What are Modules?](#what-are-modules)
- [Importing Modules](#importing-modules)
- [Import Variations](#import-variations)
- [The import System](#the-import-system)
- [Understanding **name**](#understanding-__name__)
- [Creating Your Own Modules](#creating-your-own-modules)
- [Module Search Path](#module-search-path)
- [Best Practices](#best-practices)
- [Summary](#summary)
- [Next Steps](#next-steps)

## What are Modules?

### Definition

A **module** is a Python file (`.py`) containing Python definitions and statements. Modules let you organize code into reusable components.

**Benefits:**

- Code reusability
- Namespace organization
- Easier maintenance
- Collaboration

### Example Module

`math_utils.py`:

```python
"""Utility functions for mathematical operations."""

def add(a, b):
    """Return the sum of a and b."""
    return a + b

def multiply(a, b):
    """Return the product of a and b."""
    return a * b

PI = 3.14159
```

## Importing Modules

### Basic Import

```python
import math

# Use module name as prefix
result = math.sqrt(16)  # 4.0
print(math.pi)          # 3.141592653589793
```

### Import Multiple Modules

```python
import math
import random
import datetime

print(math.pi)
print(random.randint(1, 10))
print(datetime.datetime.now())
```

**Or on one line (less preferred):**

```python
import math, random, datetime
```

## Import Variations

### From Import

Import specific names from a module:

```python
from math import sqrt, pi

# Use directly without module prefix
result = sqrt(16)  # 4.0
print(pi)          # 3.141592653589793

# Can't access other math functions
# math.sin(0)  # NameError - math not imported
```

### Import All (Avoid!)

```python
from math import *

# Imports everything from math
result = sqrt(16)
print(pi)
```

**Why avoid:**

- Pollutes namespace
- Unclear where names come from
- Risk of name conflicts

```python
# Bad - name collision
from math import *
from numpy import *  # Both have 'sqrt', 'pi', etc.

# Which sqrt? Which pi? Unclear!
result = sqrt(16)
```

### Import with Alias

Rename module or function:

```python
# Module alias
import numpy as np
arr = np.array([1, 2, 3])

# Function alias
from datetime import datetime as dt
now = dt.now()

# Avoid conflicts
from math import sqrt as math_sqrt
from numpy import sqrt as np_sqrt

result1 = math_sqrt(16)
result2 = np_sqrt([16, 25, 36])
```

## The import System

### What Happens During Import?

1. **Search** for the module in `sys.path`
2. **Compile** to bytecode (if not already cached)
3. **Execute** the module code
4. **Create** module object
5. **Bind** name in current namespace

```python
import my_module  # Executes my_module.py

# my_module.py executed line by line
# Top-level code runs
# Functions and classes defined
# Variables assigned
```

### Import Caching

Modules are imported once per program:

```python
import my_module  # Executes my_module.py
import my_module  # Uses cached version, doesn't re-execute

# Verify: module stored in sys.modules
import sys
print('my_module' in sys.modules)  # True
```

### Reimporting During Development

```python
import my_module

# Made changes to my_module.py?
import importlib
importlib.reload(my_module)  # Force re-import
```

## Understanding **name**

### The **name** Variable

Every module has a `__name__` attribute:

```python
# When imported: __name__ is module name
import math
print(math.__name__)  # 'math'

# When run directly: __name__ is '__main__'
# In my_module.py:
print(__name__)
# If imported: prints 'my_module'
# If run directly: prints '__main__'
```

### The if **name** == "**main**" Pattern

Distinguish between import and direct execution:

`my_module.py`:

```python
def greet(name):
    return f"Hello, {name}!"

def main():
    print(greet("World"))

# Only runs when script executed directly
if __name__ == "__main__":
    main()
```

**When imported:**

```python
import my_module  # Nothing prints
result = my_module.greet("Alice")  # Use the function
```

**When run directly:**

```bash
python my_module.py  # Prints: Hello, World!
```

### Why Use This Pattern?

Makes modules both **importable** and **executable**:

```python
# calculator.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

# Test code runs only when executed directly
if __name__ == "__main__":
    print(f"2 + 3 = {add(2, 3)}")
    print(f"5 - 2 = {subtract(5, 2)}")
```

**As library:**

```python
import calculator
result = calculator.add(10, 5)  # Test code doesn't run
```

**As script:**

```bash
python calculator.py
# Output:
# 2 + 3 = 5
# 5 - 2 = 3
```

## Creating Your Own Modules

### Simple Module

`greetings.py`:

```python
"""A module for greeting people."""

def hello(name):
    """Return a hello greeting."""
    return f"Hello, {name}!"

def goodbye(name):
    """Return a goodbye message."""
    return f"Goodbye, {name}!"

# Module-level variable
DEFAULT_NAME = "World"
```

**Using the module:**

```python
import greetings

print(greetings.hello("Alice"))
print(greetings.DEFAULT_NAME)
```

### Module with Class

`person.py`:

```python
"""Person class module."""

class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def greet(self):
        return f"Hi, I'm {self.name}"
```

**Using the module:**

```python
from person import Person

p = Person("Alice", 30)
print(p.greet())
```

### Module Organization

Good module structure:

```python
"""
my_module.py - Brief description

Detailed explanation of what the module does.
"""

# Imports (standard library first, then third-party, then local)
import os
import sys
from datetime import datetime

# Constants
DEFAULT_VALUE = 100
MAX_RETRIES = 3

# Functions
def public_function():
    """Public function - part of module API."""
    return _private_helper()

def _private_helper():
    """Private function - internal use only."""
    # Convention: prefix with _ for "private"
    pass

# Classes
class MyClass:
    pass

# Main execution
if __name__ == "__main__":
    # Test/demo code
    print(public_function())
```

## Module Search Path

### sys.path

Python searches for modules in directories listed in `sys.path`:

```python
import sys
print(sys.path)
# ['', '/usr/lib/python3.9', '/usr/lib/python3.9/lib-dynload', ...]
```

**Order:**

1. Current directory (`''`)
2. `PYTHONPATH` environment variable
3. Standard library
4. Site-packages (third-party packages)

### Adding to Search Path

```python
import sys
sys.path.append('/path/to/my/modules')

# Now can import from that directory
import my_custom_module
```

### Relative Paths

If module is in current directory:

```
my_project/
    main.py
    utils.py
```

`main.py`:

```python
import utils  # Works - same directory
```

## Best Practices

### Import Organization

**PEP 8 convention:**

```python
# 1. Standard library imports
import os
import sys
from datetime import datetime

# 2. Third-party imports
import numpy as np
import pandas as pd

# 3. Local application imports
from my_package import my_module
from . import sibling_module  # Relative import
```

### Avoid Circular Imports

**Problem:**

`module_a.py`:

```python
from module_b import func_b

def func_a():
    return func_b()
```

`module_b.py`:

```python
from module_a import func_a

def func_b():
    return func_a()  # Circular dependency!
```

**Solutions:**

1. Restructure code to remove circular dependency
2. Import inside function (delayed import)
3. Refactor common code to a third module

```python
# Delayed import
def func_a():
    from module_b import func_b  # Import here, not at top
    return func_b()
```

### Prefer Explicit Imports

```python
# Good - clear what's being used
from math import sqrt, pi, sin

# Avoid - unclear and can cause conflicts
from math import *
```

### Use Aliases Consistently

```python
# Common conventions
import numpy as np        # Standard alias
import pandas as pd       # Standard alias
import matplotlib.pyplot as plt  # Standard alias

# Custom aliases should be meaningful
from my_long_module_name import function as fn  # OK if used often
```

### Module Naming

- Use lowercase
- Use underscores for multi-word names
- Avoid names that conflict with standard library

```python
# Good
my_module.py
data_processor.py
utils.py

# Bad
MyModule.py      # Not PEP 8
data-processor.py  # Hyphens not allowed
string.py        # Conflicts with standard library
```

### Public vs Private

Use naming conventions to indicate intent:

```python
# my_module.py

def public_function():
    """Part of module's public API."""
    pass

def _private_function():
    """Internal use only - don't import."""
    pass

# When importing
from my_module import public_function  # Good
from my_module import _private_function  # Bad practice

# Or
import my_module
my_module.public_function()  # Good
my_module._private_function()  # Possible but indicates "don't use"
```

## Summary

Modules in Python:

**Importing:**

- `import module` - import entire module
- `from module import name` - import specific name
- `import module as alias` - import with alias
- Avoid `from module import *`

**The import system:**

- Modules executed on first import
- Cached in `sys.modules`
- Search path in `sys.path`

**`__name__` pattern:**

- `if __name__ == "__main__":` for executable modules
- Makes code both importable and runnable

**Creating modules:**

- Any `.py` file is a module
- Organize code logically
- Use docstrings
- Follow PEP 8 conventions

**Best practices:**

- Organize imports (stdlib → third-party → local)
- Use explicit imports
- Avoid circular dependencies
- Use naming conventions (public vs \_private)

Modules are fundamental to organizing Python code and building larger applications.

## Next Steps

- Organize larger projects with packages in [Core Concepts](../core_concepts/modules-and-packages.md)
- Apply modular design in [Patterns and Practices](../patterns_and_practices/code-organization.md)
- Build reusable components with [Functions](functions.md)
