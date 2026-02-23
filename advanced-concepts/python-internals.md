# Python Internals

## Table of Contents

- [CPython Implementation](#cpython-implementation)
- [Python Bytecode](#python-bytecode)
- [The dis Module](#the-dis-module)
- [Object Internals](#object-internals)
- [Function Internals](#function-internals)
- [Class and Method Internals](#class-and-method-internals)
- [Import System](#import-system)
- [Summary](#summary)
- [Next Steps](#next-steps)

## CPython Implementation

### What is CPython?

CPython is the reference implementation of Python written in C:

```python
# Python code is:
# 1. Parsed into an Abstract Syntax Tree (AST)
# 2. Compiled to bytecode
# 3. Executed by the Python Virtual Machine (PVM)

# Example: x = 1 + 2
# AST: Assign(target=Name('x'), value=BinOp(left=1, op=Add, right=2))
# Bytecode: LOAD_CONST, LOAD_CONST, BINARY_ADD, STORE_NAME
# PVM: Executes bytecode instructions
```

### Python Virtual Machine

```python
# PVM is a stack-based machine
# - Uses evaluation stack to perform operations
# - Executes bytecode instructions one at a time
# - Maintains call stack for function calls

# Example execution:
# 1 + 2
# Stack: []
# LOAD_CONST 1    → Stack: [1]
# LOAD_CONST 2    → Stack: [1, 2]
# BINARY_ADD      → Stack: [3]
```

## Python Bytecode

### What is Bytecode?

Intermediate representation between source code and machine code:

```python
# Python source (.py) → Bytecode (.pyc) → Execution

# Bytecode is:
# - Platform independent
# - Cached in __pycache__/ directory
# - .pyc files contain compiled bytecode
```

### Viewing Bytecode

```python
import dis

def add(a, b):
    return a + b

# Disassemble function
dis.dis(add)

# Output:
#   2           0 LOAD_FAST                0 (a)
#               2 LOAD_FAST                1 (b)
#               4 BINARY_ADD
#               6 RETURN_VALUE
```

### Bytecode Instructions

```python
# Common instructions:

# Loading and storing
# LOAD_CONST: Load constant
# LOAD_FAST: Load local variable
# STORE_FAST: Store local variable
# LOAD_GLOBAL: Load global variable
# STORE_GLOBAL: Store global variable

# Operations
# BINARY_ADD, BINARY_MULTIPLY, etc.
# COMPARE_OP: Comparison operators
# UNARY_NOT, UNARY_NEGATIVE, etc.

# Control flow
# JUMP_FORWARD, JUMP_ABSOLUTE
# POP_JUMP_IF_FALSE, POP_JUMP_IF_TRUE

# Function calls
# CALL_FUNCTION
# RETURN_VALUE
```

## The dis Module

### Disassembling Code

```python
import dis

# Disassemble simple expression
dis.dis("x = 1 + 2")

# Disassemble function
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

dis.dis(factorial)

# Disassemble method
class MyClass:
    def method(self, x):
        return x * 2

dis.dis(MyClass.method)
```

### Understanding Output

```python
import dis

def example():
    x = 10
    y = 20
    return x + y

dis.dis(example)

# Output explanation:
#   Line    Offset  Opcode          Argument
#   2       0       LOAD_CONST      1 (10)        # Load constant 10
#           2       STORE_FAST      0 (x)         # Store in variable x
#   3       4       LOAD_CONST      2 (20)        # Load constant 20
#           6       STORE_FAST      1 (y)         # Store in variable y
#   4       8       LOAD_FAST       0 (x)         # Load variable x
#           10      LOAD_FAST       1 (y)         # Load variable y
#           12      BINARY_ADD                    # Add x and y
#           14      RETURN_VALUE                  # Return result
```

### Bytecode Optimization

```python
import dis

# Constant folding
def unoptimized():
    return 1 + 2 + 3 + 4

def optimized():
    return 10  # Compiler optimizes 1+2+3+4 to 10

dis.dis(unoptimized)
# Shows LOAD_CONST 10, RETURN_VALUE (already optimized!)

# The compiler performs peephole optimization
```

### Code Object

```python
def my_function(a, b):
    c = a + b
    return c

# Get code object
code = my_function.__code__

print(f"Name: {code.co_name}")
print(f"Args: {code.co_argcount}")
print(f"Local vars: {code.co_nlocals}")
print(f"Stack size: {code.co_stacksize}")
print(f"Bytecode: {code.co_code}")
print(f"Constants: {code.co_consts}")
print(f"Variable names: {code.co_varnames}")
print(f"Filename: {code.co_filename}")
print(f"First line: {code.co_firstlineno}")
```

## Object Internals

### Object Structure

Every Python object has:

```python
class MyClass:
    pass

obj = MyClass()

# Internal structure (simplified):
# - ob_refcnt: Reference count
# - ob_type: Pointer to type object
# - ... type-specific data

print(type(obj))  # <class '__main__.MyClass'>
print(id(obj))    # Memory address
print(obj.__class__)  # Same as type(obj)
```

### Type Object

```python
# Everything has a type
print(type(42))           # <class 'int'>
print(type("hello"))      # <class 'str'>
print(type([1, 2, 3]))    # <class 'list'>

# Types are objects too
print(type(int))          # <class 'type'>
print(type(type))         # <class 'type'>

# type is its own type!
print(type(type) is type) # True
```

### Object Attributes

```python
class MyClass:
    class_var = "class"

    def __init__(self):
        self.instance_var = "instance"

obj = MyClass()

# Instance __dict__
print(obj.__dict__)  # {'instance_var': 'instance'}

# Class __dict__
print(MyClass.__dict__)  # {..., 'class_var': 'class', ...}

# Attribute lookup order:
# 1. Instance __dict__
# 2. Class __dict__
# 3. Parent class __dict__ (if any)
# 4. __getattr__ (if defined)
```

## Function Internals

### Function Objects

```python
def my_function(x, y=10):
    """Docstring."""
    return x + y

# Function attributes
print(f"Name: {my_function.__name__}")
print(f"Doc: {my_function.__doc__}")
print(f"Module: {my_function.__module__}")
print(f"Defaults: {my_function.__defaults__}")  # (10,)
print(f"Code: {my_function.__code__}")
print(f"Globals: {my_function.__globals__}")  # Global namespace
print(f"Closure: {my_function.__closure__}")  # None (not a closure)
```

### Closure Internals

```python
def outer(x):
    def inner(y):
        return x + y
    return inner

closure = outer(10)

# Closure cells
print(closure.__closure__)  # (<cell at 0x...: int object at 0x...>,)
print(closure.__closure__[0].cell_contents)  # 10

# Code object shows free variables
print(closure.__code__.co_freevars)  # ('x',)
```

### Call Stack

```python
import sys

def function_a():
    function_b()

def function_b():
    function_c()

def function_c():
    # Get current frame
    frame = sys._getframe()

    # Walk up the call stack
    while frame:
        print(f"Function: {frame.f_code.co_name}")
        print(f"  Line: {frame.f_lineno}")
        print(f"  Locals: {list(frame.f_locals.keys())}")
        frame = frame.f_back

function_a()
```

## Class and Method Internals

### Class Creation

```python
# Class definition
class MyClass:
    x = 10

    def method(self):
        return self.x

# Equivalent to:
def method(self):
    return self.x

MyClass = type('MyClass', (), {'x': 10, 'method': method})

# type() creates classes dynamically
```

### Method Resolution Order (MRO)

```python
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):
    pass

# MRO: D -> B -> C -> A -> object
print(D.__mro__)
print(D.mro())

obj = D()
print(obj.method())  # "B" (first in MRO)
```

### Descriptor Protocol

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        print("__get__ called")
        return 42

    def __set__(self, obj, value):
        print("__set__ called")

    def __delete__(self, obj):
        print("__delete__ called")

class MyClass:
    x = Descriptor()

obj = MyClass()
obj.x        # __get__ called
obj.x = 100  # __set__ called
del obj.x    # __delete__ called
```

### Bound vs Unbound Methods

```python
class MyClass:
    def method(self):
        return "called"

# Unbound - accessed from class
unbound = MyClass.method
print(type(unbound))  # <class 'function'>

# Bound - accessed from instance
obj = MyClass()
bound = obj.method
print(type(bound))  # <class 'method'>
print(bound.__self__)  # obj
print(bound.__func__)  # Original function

# Calling
bound()  # Same as: unbound(obj)
```

## Import System

### Import Mechanics

```python
# import module
# 1. Check sys.modules (cache)
# 2. Find module (using sys.meta_path)
# 3. Load module (read file, create module object)
# 4. Execute module code
# 5. Add to sys.modules
# 6. Bind to namespace

import sys

# Already imported modules
print(list(sys.modules.keys())[:10])

# Import paths
print(sys.path)
```

### Module Objects

```python
import math

# Module attributes
print(math.__name__)      # 'math'
print(math.__file__)      # Path to module file
print(math.__package__)   # Package name
print(math.__doc__)       # Module docstring
print(math.__dict__.keys())  # Module namespace
```

### Import Hooks

```python
import sys
import importlib.abc
import importlib.machinery

class CustomFinder(importlib.abc.MetaPathFinder):
    def find_spec(self, fullname, path, target=None):
        print(f"Trying to import: {fullname}")
        return None  # Let other finders handle it

# Add custom finder
finder = CustomFinder()
sys.meta_path.insert(0, finder)

# Now imports go through our finder
# import some_module  # Prints: Trying to import: some_module

# Remove finder
sys.meta_path.remove(finder)
```

### Package Initialization

```python
# Package structure:
# mypackage/
#   __init__.py      # Executed when package is imported
#   module1.py
#   module2.py

# __init__.py can:
# - Initialize package
# - Set __all__
# - Import submodules
# - Provide package-level API

# Example __init__.py:
# from .module1 import func1
# from .module2 import func2
# __all__ = ['func1', 'func2']
```

## Summary

Python internals:

**CPython:**

- Reference implementation in C
- Compiles Python to bytecode
- Executes bytecode in PVM (Python Virtual Machine)

**Bytecode:**

- Intermediate representation
- Platform independent
- Cached in .pyc files
- View with `dis` module

**dis module:**

- Disassemble Python code
- Shows bytecode instructions
- Helps understand optimization
- Access code objects

**Objects:**

- Everything is an object
- Has type, identity, value
- type is its own type
- Attributes stored in `__dict__`

**Functions:**

- First-class objects
- Have `__code__`, `__globals__`, `__closure__`
- Closures capture free variables

**Classes:**

- Created with `type()`
- MRO determines method resolution
- Descriptors control attribute access
- Methods are functions bound to instances

**Import system:**

- Caches in `sys.modules`
- Uses `sys.path` and `sys.meta_path`
- Can be customized with import hooks
- `__init__.py` initializes packages

## Next Steps

- Optimize with [Memory Management](memory-management.md)
- Apply [Metaprogramming](metaprogramming.md) techniques
- Profile with [Performance Optimization](../performance_and_optimization/profiling.md)
