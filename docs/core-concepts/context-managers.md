# Context Managers

## The with Statement

### Basic Usage

Automatic resource cleanup:

```python
# Without context manager
file = open('data.txt', 'r')
try:
    content = file.read()
    process(content)
finally:
    file.close()  # Must remember to close

# With context manager
with open('data.txt', 'r') as file:
    content = file.read()
    process(content)
# File automatically closed here
```

### Why Use Context Managers?

**Benefits:**

- Automatic cleanup (even if exceptions occur)
- Cleaner, more readable code
- Prevent resource leaks
- Guarantee setup and teardown

```python
# Guaranteed cleanup
with open('file.txt', 'w') as f:
    f.write('data')
    raise Exception("Something went wrong")
# File still properly closed despite exception
```

## Context Manager Protocol

### The Protocol

Two methods: `__enter__()` and `__exit__()`:

```python
class MyContext:
    def __enter__(self):
        """Setup - called when entering 'with' block."""
        print("Entering context")
        return self  # Returned value assigned to 'as' variable

    def __exit__(self, exc_type, exc_value, traceback):
        """Cleanup - called when exiting 'with' block."""
        print("Exiting context")
        return False  # Don't suppress exceptions

with MyContext() as ctx:
    print("Inside context")

# Output:
# Entering context
# Inside context
# Exiting context
```

### **exit** Parameters

```python
class DetailedContext:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        """
        exc_type: Exception class (or None)
        exc_value: Exception instance (or None)
        traceback: Traceback object (or None)
        """
        if exc_type is None:
            print("Exited normally")
        else:
            print(f"Exception occurred: {exc_type.__name__}: {exc_value}")

        return False  # Propagate exception

with DetailedContext():
    print("Working...")
# Exited normally

with DetailedContext():
    raise ValueError("Something wrong")
# Exception occurred: ValueError: Something wrong
# ValueError: Something wrong (exception propagated)
```

## Creating Context Managers with Classes

### File-like Context Manager

```python
class FileManager:
    """Manage file opening and closing."""

    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_value, traceback):
        if self.file:
            self.file.close()
        return False

with FileManager('test.txt', 'w') as f:
    f.write('Hello, World!')
```

### Database Connection

```python
class DatabaseConnection:
    """Manage database connection lifecycle."""

    def __init__(self, connection_string):
        self.connection_string = connection_string
        self.connection = None

    def __enter__(self):
        print(f"Opening connection to {self.connection_string}")
        self.connection = self._connect()
        return self.connection

    def __exit__(self, exc_type, exc_value, traceback):
        if self.connection:
            if exc_type is not None:
                print("Rolling back transaction")
                self.connection.rollback()
            else:
                print("Committing transaction")
                self.connection.commit()

            print("Closing connection")
            self.connection.close()

        return False

    def _connect(self):
        # Simulate connection
        return {"connected": True}

with DatabaseConnection("localhost:5432") as conn:
    # Do database operations
    pass
```

### Timer Context Manager

```python
import time

class Timer:
    """Measure execution time."""

    def __enter__(self):
        self.start = time.time()
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        self.end = time.time()
        self.elapsed = self.end - self.start
        print(f"Elapsed time: {self.elapsed:.4f} seconds")
        return False

with Timer():
    time.sleep(1)
    sum(range(1000000))
# Elapsed time: 1.0234 seconds
```

## contextlib Module

### @contextmanager Decorator

Simpler context manager creation using generators:

```python
from contextlib import contextmanager

@contextmanager
def file_manager(filename, mode):
    """Context manager using generator."""
    print(f"Opening {filename}")
    f = open(filename, mode)

    try:
        yield f  # Yield control to with block
    finally:
        print(f"Closing {filename}")
        f.close()

with file_manager('test.txt', 'w') as f:
    f.write('Hello!')
# Opening test.txt
# Closing test.txt
```

### How @contextmanager Works

```python
from contextlib import contextmanager

@contextmanager
def my_context():
    print("Setup")    # Before yield = __enter__
    yield "value"     # Yielded value = return from __enter__
    print("Cleanup")  # After yield = __exit__

with my_context() as value:
    print(f"Got: {value}")
# Setup
# Got: value
# Cleanup
```

### Catching Exceptions

```python
from contextlib import contextmanager

@contextmanager
def handled_context():
    try:
        print("Entering")
        yield
    except Exception as e:
        print(f"Caught exception: {e}")
        # Don't re-raise to suppress
    finally:
        print("Cleanup")

with handled_context():
    print("Working")
    raise ValueError("Oops")
# Entering
# Working
# Caught exception: Oops
# Cleanup
# (No exception propagated)
```

### contextlib.suppress

Suppress specified exceptions:

```python
from contextlib import suppress

# Without suppress
try:
    os.remove('nonexistent_file.txt')
except FileNotFoundError:
    pass

# With suppress
with suppress(FileNotFoundError):
    os.remove('nonexistent_file.txt')

# Multiple exceptions
with suppress(FileNotFoundError, PermissionError):
    os.remove(filename)
```

### contextlib.redirect_stdout

Redirect stdout:

```python
from contextlib import redirect_stdout
import io

# Capture print output
f = io.StringIO()
with redirect_stdout(f):
    print("Hello, World!")
    print("This goes to StringIO")

output = f.getvalue()
print(f"Captured: {output}")
```

## Multiple Context Managers

### Nested with Statements

```python
# Old style - nested
with open('input.txt', 'r') as infile:
    with open('output.txt', 'w') as outfile:
        outfile.write(infile.read())
```

### Comma-Separated

```python
# Modern style - comma separated
with open('input.txt', 'r') as infile, \
     open('output.txt', 'w') as outfile:
    outfile.write(infile.read())
```

### Multiple Resources

```python
from contextlib import contextmanager

@contextmanager
def resource(name):
    print(f"Acquiring {name}")
    try:
        yield name
    finally:
        print(f"Releasing {name}")

with resource('A') as a, resource('B') as b, resource('C') as c:
    print(f"Using {a}, {b}, {c}")
# Acquiring A
# Acquiring B
# Acquiring C
# Using A, B, C
# Releasing C
# Releasing B
# Releasing A
```

## Practical Examples

### Temporary Directory

```python
from contextlib import contextmanager
import tempfile
import shutil
import os

@contextmanager
def temporary_directory():
    """Create and clean up temporary directory."""
    temp_dir = tempfile.mkdtemp()
    print(f"Created temporary directory: {temp_dir}")

    try:
        yield temp_dir
    finally:
        print(f"Removing temporary directory: {temp_dir}")
        shutil.rmtree(temp_dir)

with temporary_directory() as temp_dir:
    # Create files in temp_dir
    filepath = os.path.join(temp_dir, 'test.txt')
    with open(filepath, 'w') as f:
        f.write('Temporary data')

    # Use temporary files
    print(os.listdir(temp_dir))
# Directory automatically cleaned up
```

### Change Directory

```python
from contextlib import contextmanager
import os

@contextmanager
def change_directory(new_dir):
    """Temporarily change working directory."""
    old_dir = os.getcwd()
    print(f"Changing from {old_dir} to {new_dir}")
    os.chdir(new_dir)

    try:
        yield
    finally:
        print(f"Changing back to {old_dir}")
        os.chdir(old_dir)

with change_directory('/tmp'):
    print(f"Current directory: {os.getcwd()}")
# Automatically returns to original directory
```

### Lock Context Manager

```python
import threading

class Lock:
    """Thread lock context manager."""

    def __init__(self):
        self._lock = threading.Lock()

    def __enter__(self):
        self._lock.acquire()
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        self._lock.release()
        return False

lock = Lock()

# Thread-safe operation
with lock:
    # Critical section
    shared_resource.modify()
```

### Environment Variables

```python
from contextlib import contextmanager
import os

@contextmanager
def environment_variable(key, value):
    """Temporarily set environment variable."""
    old_value = os.environ.get(key)
    os.environ[key] = value

    try:
        yield
    finally:
        if old_value is None:
            del os.environ[key]
        else:
            os.environ[key] = old_value

with environment_variable('DEBUG', 'true'):
    # Code that uses DEBUG environment variable
    print(os.environ['DEBUG'])  # true
# DEBUG restored to original value
```

## Error Handling

### Suppressing Exceptions

Return `True` from `__exit__` to suppress:

```python
class IgnoreErrors:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type is not None:
            print(f"Ignoring exception: {exc_value}")
            return True  # Suppress exception
        return False

with IgnoreErrors():
    print("Before error")
    raise ValueError("This will be suppressed")
    print("After error")  # Won't execute
print("Continues here")

# Output:
# Before error
# Ignoring exception: This will be suppressed
# Continues here
```

### Selective Exception Handling

```python
class HandleSpecific:
    def __enter__(self):
        return self

    def __exit__(self, exc_type, exc_value, traceback):
        if exc_type is ValueError:
            print(f"Handled ValueError: {exc_value}")
            return True  # Suppress ValueError only
        return False  # Propagate other exceptions

with HandleSpecific():
    raise ValueError("This is handled")
# Handled ValueError: This is handled
# (No exception raised)

with HandleSpecific():
    raise TypeError("This propagates")
# TypeError: This propagates
```

### Cleanup on Error

```python
from contextlib import contextmanager

@contextmanager
def transaction():
    """Database-like transaction."""
    print("BEGIN TRANSACTION")

    try:
        yield
    except Exception as e:
        print(f"ROLLBACK: {e}")
        raise  # Re-raise exception
    else:
        print("COMMIT")

with transaction():
    print("Operation 1")
    print("Operation 2")
# BEGIN TRANSACTION
# Operation 1
# Operation 2
# COMMIT

try:
    with transaction():
        print("Operation 1")
        raise ValueError("Error")
        print("Operation 2")
except ValueError:
    pass
# BEGIN TRANSACTION
# Operation 1
# ROLLBACK: Error
```

## Summary

Context managers in Python:

**The protocol:**

- `__enter__()`: Setup, returns value for `as` clause
- `__exit__(exc_type, exc_value, traceback)`: Cleanup
- Return `True` from `__exit__` to suppress exceptions

**Creation methods:**

- **Class-based:** Implement `__enter__` and `__exit__`
- **Generator-based:** Use `@contextmanager` decorator

**contextlib utilities:**

- `@contextmanager`: Create context managers with generators
- `suppress()`: Suppress specified exceptions
- `redirect_stdout()`: Redirect stdout
- `ExitStack`: Manage dynamic number of context managers

**Common uses:**

- File management
- Database connections
- Locks and synchronization
- Temporary resources
- Environment changes
- Timer and profiling

**Best practices:**

- Use for resource management
- Guarantee cleanup even with exceptions
- Make code more readable
- Avoid resource leaks

Context managers are essential for proper resource management in Python.

## Next Steps

- Apply in [File Operations](../fundamentals/file-operations.md)
- Use with [Concurrency](concurrency.md) for locks
- Explore [Performance and Optimization](../performance_and_optimization/)
