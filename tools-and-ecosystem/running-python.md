# Running Python Code

## Table of Contents

- [Python Scripts](#python-scripts)
- [Interactive Mode](#interactive-mode)
- [Modules vs Scripts](#modules-vs-scripts)
- [The if \_\_name\_\_ == '\_\_main\_\_' Pattern](#the-if-__name__--__main__-pattern)
- [Different Execution Methods](#different-execution-methods)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Python Scripts

**Script**: A Python file (.py) containing code that performs specific tasks when executed.

### Creating and Running a Script

```python
# hello.py
print("Hello, World!")
name = input("What's your name? ")
print(f"Nice to meet you, {name}!")
```

```bash
# Run the script
python hello.py
# or
python3 hello.py

# With arguments
python script.py arg1 arg2
```

### Script with Arguments

```python
# greet.py
import sys

if len(sys.argv) < 2:
    print("Usage: python greet.py <name>")
    sys.exit(1)

name = sys.argv[1]
print(f"Hello, {name}!")
```

```bash
# Run with argument
python greet.py Alice
# Output: Hello, Alice!
```

### Using argparse

```python
# calculator.py
import argparse

def main():
    parser = argparse.ArgumentParser(description="Simple calculator")
    parser.add_argument("x", type=float, help="First number")
    parser.add_argument("y", type=float, help="Second number")
    parser.add_argument(
        "--operation",
        choices=["add", "sub", "mul", "div"],
        default="add",
        help="Operation to perform"
    )

    args = parser.parse_args()

    if args.operation == "add":
        result = args.x + args.y
    elif args.operation == "sub":
        result = args.x - args.y
    elif args.operation == "mul":
        result = args.x * args.y
    elif args.operation == "div":
        result = args.x / args.y

    print(f"Result: {result}")

if __name__ == "__main__":
    main()
```

```bash
# Usage examples
python calculator.py 5 3 --operation add
# Result: 8.0

python calculator.py 10 2 --operation div
# Result: 5.0

python calculator.py --help
# Shows usage information
```

### Shebang Line (Unix/Linux/macOS)

```python
#!/usr/bin/env python3
# script.py

print("Hello from Python!")
```

```bash
# Make executable
chmod +x script.py

# Run directly
./script.py
```

## Interactive Mode

**Interactive mode**: Running Python commands one at a time in a shell, seeing results immediately.

### Starting Interactive Mode

```bash
# Start Python interpreter
python
# or
python3

# You'll see:
# Python 3.11.5 (main, ...)
# Type "help", "copyright", "credits" or "license" for more information.
# >>>
```

### Interactive Session Example

```python
>>> 2 + 2
4
>>> name = "Alice"
>>> age = 30
>>> f"{name} is {age} years old"
'Alice is 30 years old'
>>> def greet(name):
...     return f"Hello, {name}!"
...
>>> greet("Bob")
'Hello, Bob!'
>>> import math
>>> math.sqrt(16)
4.0
>>> exit()  # or Ctrl+D (Unix) / Ctrl+Z+Enter (Windows)
```

### Underscore Variable

```python
>>> 10 + 5
15
>>> _ + 3  # _ holds last result
18
>>> result = _
>>> result
18
```

### Help System

```python
>>> help(len)
# Shows documentation for len()

>>> help(str.split)
# Shows documentation for str.split()

>>> help()  # Interactive help mode
# help> list
# Shows list documentation
# help> quit
```

### dir() Function

```python
>>> import os
>>> dir(os)
# Lists all attributes and methods of os module

>>> dir()
# Lists names in current scope

>>> dir(str)
# Lists all str methods
```

## Modules vs Scripts

### Module

**Module**: A Python file designed to be imported and reused by other code.

```python
# math_utils.py (module)
"""Utility functions for mathematical operations."""

def add(a, b):
    """Add two numbers."""
    return a + b

def multiply(a, b):
    """Multiply two numbers."""
    return a * b

PI = 3.14159
```

```python
# Using the module
import math_utils

result = math_utils.add(5, 3)
print(result)  # 8
```

### Script

**Script**: A Python file designed to be executed directly to perform a task.

```python
# process_data.py (script)
"""Process data files and generate report."""

import sys
from pathlib import Path

def process_file(filename):
    with open(filename) as f:
        return f.read()

def main():
    if len(sys.argv) < 2:
        print("Usage: python process_data.py <filename>")
        sys.exit(1)

    filename = sys.argv[1]
    data = process_file(filename)
    print(f"Processed {len(data)} characters")

# Run only if executed as script
if __name__ == "__main__":
    main()
```

### Hybrid: Module and Script

```python
# calculator.py (both module and script)

def add(a, b):
    """Add two numbers."""
    return a + b

def multiply(a, b):
    """Multiply two numbers."""
    return a * b

def main():
    """CLI interface."""
    import sys
    if len(sys.argv) < 3:
        print("Usage: python calculator.py <x> <y>")
        return

    x = float(sys.argv[1])
    y = float(sys.argv[2])
    print(f"Add: {add(x, y)}")
    print(f"Multiply: {multiply(x, y)}")

if __name__ == "__main__":
    main()
```

```python
# Can be imported
from calculator import add, multiply
result = add(5, 3)

# Can be run as script
# python calculator.py 5 3
```

## The if \_\_name\_\_ == '\_\_main\_\_' Pattern

### Understanding \_\_name\_\_

```python
# module.py
print(f"__name__ = {__name__}")

def greet():
    print("Hello!")

if __name__ == "__main__":
    print("Running as script")
    greet()
```

```bash
# Run as script
python module.py
# Output:
# __name__ = __main__
# Running as script
# Hello!
```

```python
# Import as module
import module
# Output:
# __name__ = module
# (Only this prints, not "Running as script")

module.greet()
# Output: Hello!
```

### Why Use This Pattern?

```python
# utils.py

def process_data(data):
    """Function to be used by other modules."""
    return data.upper()

# Testing code (only runs when script is executed)
if __name__ == "__main__":
    test_data = "hello"
    result = process_data(test_data)
    print(f"Test: {result}")
    assert result == "HELLO"
    print("All tests passed!")
```

**Benefits:**

- Module can be imported without running test code
- Script can be executed to run tests or demos
- Separates reusable code from execution code

### Common Pattern

```python
# app.py

import sys
from typing import List

def process(items: List[str]) -> int:
    """Business logic - can be imported."""
    return sum(len(item) for item in items)

def main(args: List[str]) -> int:
    """Entry point - runs when script is executed."""
    if len(args) < 2:
        print("Usage: python app.py <items...>")
        return 1

    items = args[1:]
    result = process(items)
    print(f"Total length: {result}")
    return 0

if __name__ == "__main__":
    sys.exit(main(sys.argv))
```

## Different Execution Methods

### Direct Execution

```bash
# Basic
python script.py

# With Python version selector (Windows)
py -3.11 script.py

# With specific Python interpreter
/usr/bin/python3 script.py
~/.pyenv/versions/3.11.5/bin/python script.py
```

### Module Execution

```bash
# Run module as script
python -m module_name

# Examples:
python -m http.server 8000  # Built-in HTTP server
python -m json.tool file.json  # Format JSON
python -m pip install requests  # Run pip as module
python -m pytest tests/  # Run pytest
python -m pdb script.py  # Run with debugger
```

### Why Use -m?

```bash
# Ensures correct Python/pip pairing
python -m pip install package
# vs
pip install package  # Might use different Python!

# Run package's __main__.py
mypackage/
├── __init__.py
└── __main__.py  # Runs with: python -m mypackage
```

### Standard Input/Output

```python
# read_stdin.py
import sys

print("Reading from stdin...")
for line in sys.stdin:
    print(f"Got: {line.strip()}")
```

```bash
# Pipe input
echo "Hello" | python read_stdin.py

# Redirect input
python read_stdin.py < input.txt

# Redirect output
python script.py > output.txt

# Chain commands
cat data.txt | python process.py | python analyze.py
```

### Running Code Strings

```bash
# Execute Python code directly
python -c "print('Hello, World!')"

# Multiple statements
python -c "import sys; print(sys.version)"

# Useful for quick calculations
python -c "print(sum(range(1, 101)))"
# Output: 5050
```

### Interactive After Script

```bash
# Run script then drop to interactive mode
python -i script.py

# Useful for debugging
# - Script variables remain available
# - Can inspect state interactively
```

### Background Execution

```bash
# Unix/Linux/macOS
python long_running.py &
# Returns process ID

# Check background jobs
jobs

# Bring to foreground
fg

# Kill process
kill <pid>
```

```bash
# Using nohup (no hangup)
nohup python long_running.py &
# Keeps running after terminal closes
# Output goes to nohup.out
```

## Summary

Running Python code:

**Scripts:**

- .py files executed with `python script.py`
- Use sys.argv or argparse for arguments
- Shebang line makes scripts directly executable (Unix)

**Interactive mode:**

- REPL: read-eval-print loop
- Great for testing and exploration
- Use help() and dir() for discovery
- \_ holds last result

**Modules vs Scripts:**

- Module: Designed to be imported
- Script: Designed to be executed
- Hybrid: Use `if __name__ == "__main__"`

**\_\_name\_\_ pattern:**

- `__name__ == "__main__"` when run as script
- `__name__ == "module_name"` when imported
- Separates reusable code from execution

**Execution methods:**

- Direct: `python script.py`
- Module: `python -m module`
- String: `python -c "code"`
- Interactive: `python -i script.py`
- Background: `python script.py &`

**Best practices:**

- Always use `if __name__ == "__main__"` guard
- Organize code into main() function
- Use argparse for command-line interfaces
- Return exit codes from main()

Understanding execution modes helps you write code that works both as a library and as a standalone tool.

## Next Steps

- Master [Virtual Environments](virtual-environments.md)
- Learn [Package Management](package-management.md)
- Review [Testing Fundamentals](../testing_and_quality/testing-fundamentals.md)
