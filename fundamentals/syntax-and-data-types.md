# Syntax and Data Types

## Table of Contents

- [Variables and Assignment](#variables-and-assignment)
- [Basic Data Types](#basic-data-types)
- [Operators](#operators)
- [Type Conversion](#type-conversion)
- [Comments](#comments)
- [Naming Conventions](#naming-conventions)
- [Basic Input and Output](#basic-input-and-output)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Variables and Assignment

### What is a Variable?

In Python, a variable is a **name that references an object**. It's not a container that holds a value - it's a label that points to data stored in memory.

```python
x = 42  # x is a name that refers to the integer object 42
```

### Assignment Syntax

```python
# Simple assignment
name = "Alice"
age = 30
height = 5.6

# Multiple assignment
x, y, z = 1, 2, 3

# Unpacking
coordinates = (10, 20)
x, y = coordinates

# Same value to multiple variables
a = b = c = 0
```

### Dynamic Typing

Python is dynamically typed - you don't declare types explicitly. The type is determined by the value assigned:

```python
x = 42          # x refers to an int
x = "hello"     # x now refers to a str
x = [1, 2, 3]   # x now refers to a list
```

The same name can refer to different types at different times.

### Assignment is Reference

When you assign a variable to another variable, both names reference the same object:

```python
a = [1, 2, 3]
b = a           # b references the same list as a

b.append(4)
print(a)        # [1, 2, 3, 4] - both see the change
```

## Basic Data Types

### Integers (`int`)

Whole numbers without decimal points. Python 3 integers have unlimited precision.

```python
x = 42
y = -17
z = 0

# Large numbers (no limit!)
big_num = 123456789012345678901234567890

# Underscores for readability
million = 1_000_000
```

**Common operations:**

```python
10 + 5   # 15 - addition
10 - 5   # 5  - subtraction
10 * 5   # 50 - multiplication
10 / 5   # 2.0 - division (always returns float)
10 // 3  # 3  - floor division (integer result)
10 % 3   # 1  - modulo (remainder)
10 ** 2  # 100 - exponentiation
```

### Floats (`float`)

Numbers with decimal points. Use IEEE 754 double precision.

```python
x = 3.14
y = -0.5
z = 2.0

# Scientific notation
speed_of_light = 3.0e8  # 300,000,000
tiny = 1.5e-10          # 0.00000000015
```

**Important:** Floats have precision limitations:

```python
0.1 + 0.2  # 0.30000000000000004 (not exactly 0.3!)
```

For exact decimal arithmetic, use the `decimal` module:

```python
from decimal import Decimal
Decimal('0.1') + Decimal('0.2')  # Decimal('0.3')
```

### Strings (`str`)

Text data enclosed in quotes. Strings are **immutable** - cannot be changed after creation.

```python
# Single or double quotes
name = 'Alice'
message = "Hello, World!"

# Triple quotes for multi-line strings
long_text = """
This is a
multi-line string
"""

# Escaping special characters
quote = "He said, \"Hello!\""
path = "C:\\Users\\Name"

# Raw strings (backslashes are literal)
regex = r"\d+\.\d+"
```

**String operations:**

```python
# Concatenation
"Hello" + " " + "World"  # "Hello World"

# Repetition
"Ha" * 3  # "HaHaHa"

# Indexing (0-based)
text = "Python"
text[0]    # 'P'
text[-1]   # 'n' (negative indices count from end)

# Slicing [start:end:step]
text[0:3]   # 'Pyt'
text[2:]    # 'thon'
text[:4]    # 'Pyth'
text[::2]   # 'Pto' (every 2nd character)
text[::-1]  # 'nohtyP' (reversed)

# Length
len(text)  # 6

# Common methods
text.upper()        # 'PYTHON'
text.lower()        # 'python'
text.startswith('P')  # True
text.endswith('on')   # True
text.replace('P', 'J')  # 'Jython'
text.split('t')     # ['Py', 'hon']
```

**String formatting:**

```python
name = "Alice"
age = 30

# f-strings (Python 3.6+, recommended)
f"My name is {name} and I'm {age} years old"

# Format with expressions
f"Next year I'll be {age + 1}"

# Format method
"My name is {} and I'm {} years old".format(name, age)

# Old style (avoid in new code)
"My name is %s and I'm %d years old" % (name, age)
```

### Booleans (`bool`)

Truth values: `True` or `False`. Note the capital T and F.

```python
is_valid = True
is_empty = False

# Boolean operations
True and False   # False
True or False    # True
not True         # False
```

**Truthiness:** Many values evaluate to boolean in conditional contexts:

```python
# Falsy values (evaluate to False)
bool(0)         # False
bool(0.0)       # False
bool("")        # False (empty string)
bool([])        # False (empty list)
bool({})        # False (empty dict)
bool(None)      # False

# Truthy values (evaluate to True)
bool(1)         # True
bool("text")    # True
bool([1, 2])    # True
bool(-1)        # True (any non-zero number)
```

### None Type (`None`)

Represents the absence of a value. Similar to `null` in other languages.

```python
x = None

# Often used as default value
def greet(name=None):
    if name is None:
        return "Hello, stranger!"
    return f"Hello, {name}!"

# Check for None with 'is'
if x is None:
    print("x is None")

# Not the same as False or 0
None == False  # False
None == 0      # False
```

## Operators

### Arithmetic Operators

```python
a, b = 10, 3

a + b   # 13 - addition
a - b   # 7  - subtraction
a * b   # 30 - multiplication
a / b   # 3.333... - division (always float)
a // b  # 3  - floor division (rounds down)
a % b   # 1  - modulo (remainder)
a ** b  # 1000 - exponentiation

# Compound assignment
x = 5
x += 3   # x = x + 3, now x is 8
x -= 2   # x = x - 2, now x is 6
x *= 4   # x = x * 4, now x is 24
x /= 3   # x = x / 3, now x is 8.0
```

### Comparison Operators

Return boolean values:

```python
5 == 5   # True  - equal to
5 != 3   # True  - not equal to
5 > 3    # True  - greater than
5 < 10   # True  - less than
5 >= 5   # True  - greater than or equal
5 <= 4   # False - less than or equal

# Chain comparisons
x = 5
1 < x < 10  # True (same as: 1 < x and x < 10)
```

### Logical Operators

```python
# and - True if both are True
True and True   # True
True and False  # False

# or - True if at least one is True
True or False   # True
False or False  # False

# not - negates boolean
not True   # False
not False  # True

# Practical examples
age = 25
if age >= 18 and age < 65:
    print("Working age")

is_weekend = True
is_holiday = False
if is_weekend or is_holiday:
    print("Day off!")
```

**Short-circuit evaluation:** Python doesn't evaluate more than necessary:

```python
# If first part is False, second part isn't evaluated
False and expensive_function()  # expensive_function not called

# If first part is True, second part isn't evaluated
True or expensive_function()    # expensive_function not called
```

### Identity Operators

Compare object identity (memory location), not value:

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

a == b   # True  - same value
a is b   # False - different objects

a is c   # True  - same object
a == c   # True  - same value

# Commonly used with None
x = None
if x is None:  # Correct
    pass

if x == None:  # Works but not idiomatic
    pass
```

### Membership Operators

Check if a value exists in a sequence:

```python
# in
'a' in 'abc'       # True
3 in [1, 2, 3]     # True
'x' in 'abc'       # False

# not in
'x' not in 'abc'   # True
4 not in [1, 2, 3] # True
```

### Bitwise Operators

Operate on integers at the binary level:

```python
a = 60   # 60 = 0011 1100 in binary
b = 13   # 13 = 0000 1101 in binary

# Bitwise AND
a & b    # 12 = 0000 1100

# Bitwise OR
a | b    # 61 = 0011 1101

# Bitwise XOR (exclusive OR)
a ^ b    # 49 = 0011 0001

# Bitwise NOT (inverts bits)
~a       # -61 = 1100 0011

# Left shift (multiply by 2^n)
a << 2   # 240 = 1111 0000 (60 * 4)

# Right shift (divide by 2^n)
a >> 2   # 15 = 0000 1111 (60 // 4)

# Compound bitwise assignment
x = 8
x &= 3   # x = x & 3
x |= 4   # x = x | 4
x ^= 1   # x = x ^ 1
x <<= 2  # x = x << 2
x >>= 1  # x = x >> 1
```

**Common use cases:**

```python
# Set flags / permissions
READ = 1 << 0    # 0001
WRITE = 1 << 1   # 0010
EXECUTE = 1 << 2 # 0100

permissions = READ | WRITE  # 0011
if permissions & WRITE:
    print("Has write permission")

# Swap values without temp variable
a, b = 5, 10
a ^= b
b ^= a
a ^= b
print(a, b)  # 10, 5

# Check if number is power of 2
def is_power_of_two(n):
    return n > 0 and (n & (n - 1)) == 0

is_power_of_two(8)   # True
is_power_of_two(10)  # False

# Get/set/clear specific bits
num = 0b1010  # 10 in decimal

# Get bit at position 1 (0-indexed from right)
bit = (num >> 1) & 1  # 1

# Set bit at position 0
num |= (1 << 0)  # 0b1011 = 11

# Clear bit at position 3
num &= ~(1 << 3)  # 0b0011 = 3

# Toggle bit at position 2
num ^= (1 << 2)  # Flip the bit
```

## Type Conversion

### Explicit Conversion

```python
# To integer
int(3.14)      # 3
int("42")      # 42
int(True)      # 1
int(False)     # 0

# To float
float(42)      # 42.0
float("3.14")  # 3.14
float(True)    # 1.0

# To string
str(42)        # "42"
str(3.14)      # "3.14"
str(True)      # "True"

# To boolean
bool(1)        # True
bool(0)        # False
bool("text")   # True
bool("")       # False
```

### Implicit Conversion

Python automatically converts types in some operations:

```python
# Integer + Float → Float
3 + 2.5   # 5.5 (int 3 converted to float)

# Integer in division → Float
10 / 2    # 5.0 (result is always float)
```

### Type Checking

```python
x = 42
type(x)  # <class 'int'>

isinstance(x, int)     # True
isinstance(x, float)   # False
isinstance(x, (int, float))  # True (checks multiple types)
```

## Comments

### Single-line Comments

```python
# This is a comment
x = 5  # Comment after code

# Comments explain WHY, not WHAT
# Bad: x = x + 1  # increment x
# Good: x = x + 1  # account for zero-indexing
```

### Multi-line Comments

```python
# Use multiple single-line comments
# for multi-line commentary
# on complex logic

# Or triple quotes (though technically a string literal)
"""
This is a multi-line string
that can serve as a comment
"""
```

### Docstrings

Special comments for documenting modules, classes, and functions:

```python
def greet(name):
    """
    Return a greeting message for the given name.

    Args:
        name: The name to greet

    Returns:
        A greeting string
    """
    return f"Hello, {name}!"
```

## Naming Conventions

Following PEP 8 style guide:

### Variables and Functions

Use `snake_case` - lowercase with underscores:

```python
# Good
user_name = "Alice"
total_count = 42
def calculate_sum(a, b):
    return a + b

# Avoid
userName = "Alice"      # camelCase (not Python style)
TotalCount = 42         # PascalCase (for classes)
calculate-sum = lambda a, b: a + b  # hyphens not allowed
```

### Constants

Use `UPPER_CASE` with underscores:

```python
MAX_SIZE = 100
DEFAULT_TIMEOUT = 30
PI = 3.14159
```

### Classes

Use `PascalCase` - capitalize first letter of each word:

```python
class UserAccount:
    pass

class HTTPConnection:
    pass
```

### Private Conventions

Prefix with underscore to indicate internal use:

```python
_internal_value = 42     # "Private" variable
def _helper_function():  # "Private" function
    pass
```

### Naming Best Practices

```python
# Descriptive names
user_count = 10        # Good
n = 10                 # Too short

# Avoid single letters except in loops
for i in range(10):    # OK
for item in items:     # Better when possible

# Boolean names suggest True/False
is_valid = True
has_permission = False
can_edit = True

# Avoid Python keywords
list = [1, 2, 3]       # Bad (shadows built-in)
my_list = [1, 2, 3]    # Good
```

## Basic Input and Output

### Output with `print()`

```python
# Basic output
print("Hello, World!")

# Multiple arguments (space-separated)
print("Hello", "World")  # Hello World

# Custom separator
print("A", "B", "C", sep="-")  # A-B-C

# Custom ending (default is newline)
print("Hello", end=" ")
print("World")  # Hello World (on same line)

# Printing variables
name = "Alice"
print("Name:", name)
print(f"Name: {name}")  # f-string (preferred)

# Print to file or stderr
with open('output.txt', 'w') as f:
    print("Hello", file=f)

import sys
print("Error!", file=sys.stderr)
```

### Input with `input()`

```python
# Get string input
name = input("Enter your name: ")
print(f"Hello, {name}!")

# Convert to other types
age = int(input("Enter your age: "))
height = float(input("Enter height in meters: "))

# Handle conversion errors
try:
    number = int(input("Enter a number: "))
except ValueError:
    print("That's not a valid number!")
```

**Note:** `input()` always returns a string. You must convert it explicitly:

```python
# Wrong
x = input("Enter a number: ")
y = x + 5  # TypeError if user enters "10"

# Right
x = int(input("Enter a number: "))
y = x + 5  # Works correctly
```

## Summary

Python's basic syntax is designed for readability:

- **Variables** are references to objects, dynamically typed
- **Basic types**: `int`, `float`, `str`, `bool`, `None`
- **Operators**: arithmetic, comparison, logical, identity, membership
- **Type conversion**: explicit with `int()`, `float()`, `str()`, etc.
- **Comments**: use `#` for single-line, docstrings for documentation
- **Naming**: snake_case for variables/functions, PascalCase for classes, UPPER_CASE for constants
- **I/O**: `print()` for output, `input()` for input (returns string)

These building blocks appear in every Python program. Understanding them deeply makes everything else easier.

## Next Steps

- Learn [Control Flow](control-flow.md) to make decisions and repeat operations
- Explore [Data Structures](data-structures.md) for organizing collections of data
- Master [Functions](functions.md) for code reuse and organization
