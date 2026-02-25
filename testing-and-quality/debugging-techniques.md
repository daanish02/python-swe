# Debugging Techniques

## Table of Contents

- [Print Debugging](#print-debugging)
- [Python Debugger (pdb)](#python-debugger-pdb)
- [IDE Debugging](#ide-debugging)
- [Logging for Debugging](#logging-for-debugging)
- [Stack Traces and Exceptions](#stack-traces-and-exceptions)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Print Debugging

### Basic Print Statements

```python
def calculate_total(items):
    print(f"Items: {items}")  # Debug: see input
    total = 0
    for item in items:
        print(f"Processing item: {item}")  # Debug: see each iteration
        total += item['price'] * item['quantity']
    print(f"Total: {total}")  # Debug: see output
    return total

# Better: Use repr() for unambiguous output
def process_data(data):
    print(f"Data: {data!r}")  # Shows quotes for strings, etc.
```

### Pretty Printing

```python
from pprint import pprint

# Complex data structures
data = {
    'users': [
        {'id': 1, 'name': 'Alice', 'scores': [95, 87, 92]},
        {'id': 2, 'name': 'Bob', 'scores': [88, 91, 85]}
    ]
}

# Regular print - hard to read
print(data)

# Pretty print - formatted
pprint(data)
# Output:
# {'users': [{'id': 1, 'name': 'Alice', 'scores': [95, 87, 92]},
#            {'id': 2, 'name': 'Bob', 'scores': [88, 91, 85]}]}

# Control depth
pprint(data, depth=1)
# {'users': [...]}
```

### Debugging with f-strings

```python
# Show variable name and value
x = 42
y = 10
print(f"{x=}, {y=}")  # x=42, y=10

# With expressions
items = [1, 2, 3, 4, 5]
print(f"{len(items)=}")  # len(items)=5
print(f"{sum(items)=}")  # sum(items)=15

# Formatting
price = 19.99
print(f"{price=:.2f}")  # price=19.99
```

### Temporary Debug Markers

```python
# Use distinctive markers to find debug code later
def complex_function():
    # DEBUG: Check intermediate values
    print("🔍 DEBUG: Starting function")  # Easy to spot

    result = some_calculation()
    print(f"🔍 DEBUG: Result = {result}")

    return result

# Easy to grep: grep -r "🔍 DEBUG" .
# Or use: TODO, FIXME, XXX, DEBUG
```

## Python Debugger (pdb)

**pdb (Python Debugger)**: An interactive debugger that lets you pause code execution, inspect variables, and step through code line by line.

### Starting pdb

```python
import pdb

def buggy_function(x, y):
    result = x + y
    pdb.set_trace()  # Debugger stops here
    return result * 2

buggy_function(3, 4)

# Python 3.7+: Use breakpoint()
def buggy_function(x, y):
    result = x + y
    breakpoint()  # Better: uses PYTHONBREAKPOINT env var
    return result * 2
```

### pdb Commands

```python
# Once in pdb:

# Navigation
# (Pdb) n          - Next line (step over)
# (Pdb) s          - Step into function
# (Pdb) c          - Continue execution
# (Pdb) r          - Return from current function
# (Pdb) q          - Quit debugger

# Inspection
# (Pdb) p x        - Print variable x
# (Pdb) pp x       - Pretty print variable x
# (Pdb) type(x)    - Check type
# (Pdb) dir(x)     - List attributes
# (Pdb) locals()   - Show local variables
# (Pdb) globals()  - Show global variables

# Code
# (Pdb) l          - List code around current line
# (Pdb) ll         - List entire function
# (Pdb) w          - Where am I? (stack trace)

# Breakpoints
# (Pdb) b 10       - Set breakpoint at line 10
# (Pdb) b file.py:20  - Set breakpoint in file
# (Pdb) b function    - Set breakpoint at function
# (Pdb) tbreak 10     - Temporary breakpoint
# (Pdb) cl         - Clear all breakpoints
# (Pdb) cl 1       - Clear breakpoint 1
```

### pdb Example Session

```python
def calculate_average(numbers):
    total = sum(numbers)
    breakpoint()  # Stop here
    count = len(numbers)
    average = total / count
    return average

calculate_average([1, 2, 3, 4, 5])

# Session:
# > /path/to/file.py(3)calculate_average()
# -> count = len(numbers)
# (Pdb) p total
# 15
# (Pdb) p numbers
# [1, 2, 3, 4, 5]
# (Pdb) n
# > /path/to/file.py(4)calculate_average()
# -> average = total / count
# (Pdb) p count
# 5
# (Pdb) c
```

### Conditional Breakpoints

```python
def process_items(items):
    for i, item in enumerate(items):
        # Only break when item > 100
        if item > 100:
            breakpoint()
        process(item)

# Or with pdb command:
# (Pdb) b 5, item > 100  # Break at line 5 if item > 100
```

### Post-mortem Debugging

```python
import pdb

def buggy_function():
    x = 1
    y = 0
    return x / y  # ZeroDivisionError

try:
    buggy_function()
except:
    pdb.post_mortem()  # Debug at exception point

# Or run script with: python -m pdb script.py
# Then use 'pm' command after exception
```

## IDE Debugging

### VS Code Debugging

```python
# No code changes needed!

# 1. Set breakpoints: Click left of line number
# 2. Press F5 or Run > Start Debugging
# 3. Choose Python File

# Features:
# - Step over (F10)
# - Step into (F11)
# - Step out (Shift+F11)
# - Continue (F5)
# - Variables view
# - Watch expressions
# - Call stack
# - Debug console
```

```json
// .vscode/launch.json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Python: Current File",
      "type": "python",
      "request": "launch",
      "program": "${file}",
      "console": "integratedTerminal",
      "justMyCode": true // Don't step into library code
    },
    {
      "name": "Python: Module",
      "type": "python",
      "request": "launch",
      "module": "myapp.main",
      "args": ["--debug"]
    }
  ]
}
```

### PyCharm Debugging

```python
# Similar to VS Code:
# 1. Click gutter to set breakpoint
# 2. Right-click > Debug
# 3. Use debug toolbar

# Features:
# - Inline variable values
# - Evaluate expression
# - Watches
# - Frame navigation
# - Thread inspection
```

### Debugging Tests

```python
# pytest with pdb
pytest --pdb  # Drop into debugger on failure

pytest --pdb --pdbcls=IPython.terminal.debugger:Pdb  # Use IPython debugger

# VS Code: Debug specific test
# Add to launch.json:
{
    "name": "Python: Debug Tests",
    "type": "python",
    "request": "launch",
    "module": "pytest",
    "args": ["tests/", "-v"],
    "console": "integratedTerminal"
}
```

## Logging for Debugging

### Basic Logging

```python
import logging

# Configure logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

logger = logging.getLogger(__name__)

def process_data(data):
    logger.debug(f"Processing data: {data}")

    try:
        result = complex_operation(data)
        logger.info(f"Operation succeeded: {result}")
        return result
    except Exception as e:
        logger.error(f"Operation failed: {e}", exc_info=True)
        raise
```

### Log Levels

```python
import logging

logger = logging.getLogger(__name__)

# DEBUG: Detailed information for diagnosing problems
logger.debug("Variable x = %s", x)

# INFO: Confirmation that things are working
logger.info("User %s logged in", username)

# WARNING: Something unexpected but still working
logger.warning("Disk space low: %d%% remaining", percent)

# ERROR: Error occurred, but application continues
logger.error("Failed to save file: %s", filename)

# CRITICAL: Serious error, application may crash
logger.critical("Database connection lost!")
```

### Structured Logging

```python
import logging
import json

class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
        }
        return json.dumps(log_data)

# Usage
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger = logging.getLogger(__name__)
logger.addHandler(handler)
logger.setLevel(logging.DEBUG)

logger.info("User logged in", extra={'user_id': 123, 'ip': '192.168.1.1'})
```

### Debug Logging Only in Development

```python
import os
import logging

# Set log level based on environment
log_level = logging.DEBUG if os.getenv('DEBUG') else logging.INFO

logging.basicConfig(
    level=log_level,
    format='%(levelname)s: %(message)s'
)

logger = logging.getLogger(__name__)

def process():
    logger.debug("This only shows in DEBUG mode")
    logger.info("This always shows")
```

## Stack Traces and Exceptions

**Stack trace**: A report showing the sequence of function calls that led to an error, helping you trace back where things went wrong.

### Reading Stack Traces

```python
def level3():
    x = 1 / 0  # Error here

def level2():
    level3()

def level1():
    level2()

level1()

# Traceback (most recent call last):
#   File "script.py", line 10, in <module>
#     level1()                          # Start: level1 called
#   File "script.py", line 8, in level1
#     level2()                          # Then: level2 called
#   File "script.py", line 5, in level2
#     level3()                          # Then: level3 called
#   File "script.py", line 2, in level3
#     x = 1 / 0                         # Error: division by zero
# ZeroDivisionError: division by zero
```

### Better Exception Messages

```python
# Generic error
def process_user(user_id):
    user = get_user(user_id)
    if not user:
        raise ValueError("Invalid user")  # Not helpful!

# Better: Include context
def process_user(user_id):
    user = get_user(user_id)
    if not user:
        raise ValueError(f"User not found: user_id={user_id}")

# Best: Custom exception
class UserNotFoundError(Exception):
    def __init__(self, user_id):
        self.user_id = user_id
        super().__init__(f"User not found: user_id={user_id}")

def process_user(user_id):
    user = get_user(user_id)
    if not user:
        raise UserNotFoundError(user_id)
```

### Printing Full Stack Traces

```python
import traceback

try:
    risky_operation()
except Exception as e:
    # Print full traceback
    traceback.print_exc()

    # Or capture as string
    error_trace = traceback.format_exc()
    logger.error(f"Error occurred:\n{error_trace}")

    # Or get traceback details
    tb_lines = traceback.format_tb(e.__traceback__)
    for line in tb_lines:
        logger.debug(line)
```

### Exception Chaining

```python
class DatabaseError(Exception):
    pass

def save_user(user):
    try:
        database.save(user)
    except ConnectionError as e:
        # Chain exceptions - preserves original error
        raise DatabaseError("Failed to save user") from e

# Or suppress original cause
try:
    risky_operation()
except Exception as e:
    raise NewError("Something went wrong") from None  # Hides original
```

### sys.excepthook for Debugging

```python
import sys

def custom_excepthook(type, value, traceback):
    # Custom exception handler
    print("=== EXCEPTION OCCURRED ===")
    print(f"Type: {type.__name__}")
    print(f"Value: {value}")
    print(f"Traceback: {traceback}")
    # Could send to logging, monitoring service, etc.

sys.excepthook = custom_excepthook

# Now all uncaught exceptions go through custom handler
1 / 0  # Handled by custom_excepthook
```

## Summary

Debugging techniques:

**Print debugging:**

- Basic print statements
- pprint for complex data
- f-strings with {var=}
- Distinctive markers (🔍, DEBUG)

**pdb debugger:**

- breakpoint() to start
- Commands: n (next), s (step), c (continue)
- Inspect variables: p, pp, locals()
- Set breakpoints: b line_number
- Post-mortem debugging

**IDE debugging:**

- VS Code: Set breakpoints, F5 to run
- PyCharm: Similar workflow
- Debug tests directly
- launch.json configuration

**Logging:**

- Use logging module, not print
- Log levels: DEBUG, INFO, WARNING, ERROR, CRITICAL
- Structured logging with JSON
- Debug mode in development

**Stack traces:**

- Read from bottom up
- Include context in exceptions
- Custom exceptions
- Exception chaining
- traceback module for details

Effective debugging combines these techniques: logging for production, pdb for complex issues, IDE debugging for workflows, and print statements for quick checks.

## Next Steps

- Explore [Tools and Ecosystem](../tools_and_ecosystem/README.md)
- Optimize with [Performance and Optimization](../performance_and_optimization/README.md)
- Review [Testing Fundamentals](testing-fundamentals.md)
