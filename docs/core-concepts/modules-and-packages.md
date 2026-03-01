# Modules and Packages

## Module System Deep Dive

### Module Objects

Every imported module is an object:

```python
import math

# Module is an object
print(type(math))  # <class 'module'>
print(math.__name__)  # math
print(math.__file__)  # /path/to/math.py

# Has __dict__ with all names
print(math.__dict__.keys())
```

### sys.modules Cache

Modules are cached after first import:

```python
import sys

# Check if module is cached
'math' in sys.modules  # False

import math
'math' in sys.modules  # True

# Same object on re-import
import math as m1
import math as m2
print(m1 is m2)  # True - same object
```

### Reloading Modules

```python
import importlib
import my_module

# Make changes to my_module.py...

# Reload to see changes
importlib.reload(my_module)
```

## Package Structure

### What is a Package?

A directory with `__init__.py` file:

```bash
mypackage/
    __init__.py       # Makes it a package
    module1.py
    module2.py
    subpackage/
        __init__.py   # Sub-package
        module3.py
```

### Simple Package Example

=== "Directory structure"

    ```bash
    mathtools/
        __init__.py
        basic.py
        advanced.py
    ```

=== "mathtools/basic.py"

    ```python
    def add(a, b):
        return a + b

    def subtract(a, b):
        return a - b
    ```

=== "mathtools/advanced.py"

    ```python
    def power(base, exp):
        return base ** exp

    def factorial(n):
        if n <= 1:
            return 1
        return n * factorial(n - 1)
    ```

**Usage:**

```python
# Import whole module
import mathtools.basic
result = mathtools.basic.add(3, 5)

# Import specific function
from mathtools.basic import add
result = add(3, 5)

# Import with alias
from mathtools import advanced as adv
result = adv.power(2, 10)
```

## **init**.py Files

### Purpose

1. **Mark directory as package**
2. **Initialize package** (run code on import)
3. **Define package-level imports**
4. **Control `from package import *`**

### Empty **init**.py

Minimal package:

```python
# mathtools/__init__.py
# Empty file - just marks as package
```

### Package Initialization

Run code when package is imported:

```python
# mathtools/__init__.py
print("Initializing mathtools package")

# Define package-level variables
VERSION = "1.0.0"

# Import from submodules
from .basic import add, subtract
from .advanced import power, factorial

# Now can use directly
import mathtools
mathtools.add(3, 5)  # Works without .basic
```

### \_\_all\_\_ Variable

Control `from package import *`:

=== "mathtools/\_\_init\_\_.py"

    ```python
    from .basic import add, subtract
    from .advanced import power, factorial

    # Only these will be imported with *
    __all__ = ['add', 'subtract', 'power']

    # Using it
    from mathtools import *
    add(1, 2)      # OK
    power(2, 3)    # OK
    factorial(5)   # NameError - not in __all__
    ```

### Lazy Imports

Defer imports until needed:

=== "mathtools/\_\_init\_\_.py"

    ```python
    def get_advanced():
        """Import advanced module only when needed."""
        from . import advanced
        return advanced

    # Usage
    import mathtools
    adv = mathtools.get_advanced()
    adv.factorial(5)
    ```

## Relative vs Absolute Imports

### Absolute Imports

Full path from project root:

```python
# Always clear where module comes from
from mathtools.basic import add
from mathtools.advanced import power
import mathtools.subpackage.module
```

### Relative Imports

Relative to current module:

```python
# mathtools/advanced.py
from .basic import add  # Same package
from ..utils import helper  # Parent package
from .subpackage import module  # Sub-package
```

**Relative import syntax:**

- `.` = current package
- `..` = parent package
- `...` = grandparent package

### Complete Example

**Project structure:**

```bash
project/
    main.py
    package/
        __init__.py
        module_a.py
        module_b.py
        subpackage/
            __init__.py
            module_c.py
```

=== "package/subpackage/module_c.py"

    ```python
    # Import from parent package
    from ..module_a import func_a
    from ..module_b import func_b

    def func_c():
        return func_b() + " and Function C"
    ```

=== "package/module_b.py"

    ```python
    # Absolute import
    from package.module_a import func_a

    # Relative import (preferred within package)
    from .module_a import func_a

    def func_b():
        return func_a() + " and Function B"
    ```

=== "package/module_a.py"

    ```python
    def func_a():
        return "Function A"
    ```

### When to Use Each

**Absolute imports:**

- Clear and explicit
- Work from any context
- Recommended for large projects

**Relative imports:**

- More concise within packages
- Make packages more portable
- Only work within packages (not in scripts)

## Namespace Packages

### PEP 420 Namespace Packages

Packages without `__init__.py` (Python 3.3+):

```bash
namespace_package/
    subpackage1/
        module1.py   # No __init__.py needed
    subpackage2/
        module2.py
```

### Split Across Directories

Same namespace in multiple locations:

```bash
/path1/mynamespace/
    module_a.py

/path2/mynamespace/
    module_b.py

# Both contribute to same namespace
import mynamespace.module_a
import mynamespace.module_b
```

### Use Cases

- Plugin systems
- Extending packages across installations
- Framework extensions

## Import Mechanics

### The Import Process

1. **Check sys.modules** (cache)
2. **Find module** (using sys.path)
3. **Create module object**
4. **Execute module code**
5. **Cache in sys.modules**
6. **Bind in local namespace**

### sys.path

Where Python looks for modules:

```python
import sys

# View search path
print(sys.path)
# ['', '/usr/lib/python3.9', '/usr/lib/python3.9/lib-dynload', ...]

# Add custom path
sys.path.append('/my/custom/path')

# Now can import from that path
```

### Import Hooks

Customize import behavior:

```python
import sys

class CustomImporter:
    def find_module(self, fullname, path=None):
        print(f"Trying to import: {fullname}")
        return None  # Fall back to default

# Add to meta_path
sys.meta_path.insert(0, CustomImporter())

import some_module  # Prints: Trying to import: some_module
```

### **import** Function

Programmatic imports:

```python
# Dynamic import
module_name = "math"
module = __import__(module_name)
result = module.sqrt(16)

# Better way: importlib
import importlib
module = importlib.import_module(module_name)
```

## Circular Imports

### The Problem

```python
# module_a.py
from module_b import func_b

def func_a():
    return func_b()

# module_b.py
from module_a import func_a  # Circular!

def func_b():
    return func_a()

# import module_a  # ImportError or AttributeError
```

### Why It Happens

1. Import `module_a`
2. `module_a` imports `module_b`
3. `module_b` imports `module_a` (not fully loaded yet)
4. Error or incomplete import

### Solutions

**1. Restructure code:**

```python
# utils.py - common dependencies
def shared_func():
    pass

# module_a.py
from utils import shared_func

# module_b.py
from utils import shared_func
```

**2. Import at function level:**

```python
# module_a.py
def func_a():
    from module_b import func_b  # Import inside function
    return func_b()

# module_b.py
def func_b():
    from module_a import func_a
    return func_a()
```

**3. Import module, not function:**

```python
# module_a.py
import module_b  # Import module, not function

def func_a():
    return module_b.func_b()
```

## Best Practices

### Import Organization

Follow PEP 8 order:

```python
# 1. Standard library imports
import os
import sys
from pathlib import Path

# 2. Related third-party imports
import numpy as np
import pandas as pd

# 3. Local application imports
from mypackage import module1
from mypackage.subpackage import module2
from . import sibling_module
```

### Avoid Wildcard Imports

```python
# Bad - unclear what's imported
from module import *

# Good - explicit
from module import func1, func2, MyClass
```

### Package Layout

```
myproject/
    README.md
    setup.py
    mypackage/
        __init__.py
        core/
            __init__.py
            module1.py
            module2.py
        utils/
            __init__.py
            helpers.py
        tests/
            __init__.py
            test_module1.py
```

### **init**.py Best Practices

```python
# mypackage/__init__.py

# Version info
__version__ = "1.0.0"
__author__ = "Your Name"

# Import commonly used items
from .core.module1 import ImportantClass
from .core.module2 import useful_function

# Control wildcard imports
__all__ = ['ImportantClass', 'useful_function']

# Package-level initialization (if needed)
def _init():
    """Private initialization."""
    pass

_init()
```

### Avoid Circular Dependencies

- Keep modules focused (Single Responsibility)
- Use dependency injection
- Create common utilities module
- Import at function level as last resort

### Module Naming

```python
# Good - lowercase, underscores
my_module.py
data_processor.py

# Bad
MyModule.py        # PascalCase for classes
data-processor.py  # Hyphens not allowed
```

## Summary

Modules and packages in Python:

**Modules:**

- Any `.py` file
- Cached in `sys.modules`
- Found via `sys.path`

**Packages:**

- Directories with `__init__.py`
- Can have sub-packages
- `__init__.py` controls package behavior

**Import types:**

- **Absolute:** Full path from root
- **Relative:** Relative to current module (`.`, `..`, etc.)
- **Namespace packages:** Without `__init__.py` (PEP 420)

**Import mechanics:**

- Module search path (`sys.path`)
- Import caching
- Import hooks for customization

**Circular imports:**

- Caused by mutual dependencies
- Solve by restructuring or delayed imports

**Best practices:**

- Organize imports (stdlib → third-party → local)
- Avoid wildcard imports
- Use `__all__` for public API
- Clear package structure
- Explicit imports

Understanding the module system is crucial for organizing larger Python projects.

## Next Steps

- Apply to [Code Organization](../patterns_and_practices/code-organization.md)
- Explore [Type Systems](type-systems.md) for type hints
- Learn [Project Structure](../patterns_and_practices/) patterns
