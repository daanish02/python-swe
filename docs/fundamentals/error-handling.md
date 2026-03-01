# Error Handling

## Understanding Exceptions

### What are Exceptions?

Exceptions are runtime errors that disrupt normal program flow. When Python encounters an error, it _raises_ an exception.

```python
# This raises a ZeroDivisionError
result = 10 / 0

# This raises a TypeError
result = "hello" + 5

# This raises a ValueError
number = int("abc")
```

!!! failure "**Unhandled Exceptions**"

    Without error handling, exceptions crash the program:

    ```python
    def divide(a, b):
        return a / b

    result = divide(10, 0)  # ZeroDivisionError - program stops
    print("This never prints")
    ```

## Try-Except Blocks

### Basic Exception Handling

```python
try:
    # Code that might raise an exception
    result = 10 / 0
except:
    # Handle any exception
    print("An error occurred")

print("Program continues")
```

**Output:**

```bash
An error occurred
Program continues
```

### Accessing Exception Details

```python
try:
    result = 10 / 0
except Exception as e:
    print(f"Error: {e}")
    print(f"Type: {type(e).__name__}")
```

**Output:**

```bash
Error: division by zero
Type: ZeroDivisionError
```

## Catching Specific Exceptions

### Why Catch Specific Exceptions?

Catching all exceptions is too broad - handle different errors differently:

```python
def read_number_from_file(filename):
    try:
        with open(filename, 'r') as f:
            content = f.read()
            number = int(content)
            return number
    except FileNotFoundError:
        print(f"File {filename} not found")
        return None
    except ValueError:
        print("File content is not a valid number")
        return None
```

### Specific Exception Syntax

```python
try:
    # Risky operation
    result = some_operation()
except SpecificError:
    # Handle SpecificError
    handle_error()
```

**Example:**

```python
def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        print("Cannot divide by zero")
        return None
    except TypeError:
        print("Arguments must be numbers")
        return None

safe_divide(10, 0)      # Cannot divide by zero
safe_divide(10, "2")    # Arguments must be numbers
safe_divide(10, 2)      # 5.0
```

## Multiple Except Blocks

### Sequential Exception Handlers

```python
try:
    file = open('data.txt', 'r')
    content = file.read()
    number = int(content)
    result = 100 / number
except FileNotFoundError:
    print("File not found")
except ValueError:
    print("File contains invalid number")
except ZeroDivisionError:
    print("Number cannot be zero")
finally:
    try:
        file.close()
    except:
        pass
```

### Catching Multiple Exceptions Together

```python
try:
    # Operation that might fail
    result = process_data()
except (ValueError, TypeError) as e:
    print(f"Invalid data: {e}")

# Equivalent to:
try:
    result = process_data()
except ValueError as e:
    print(f"Invalid data: {e}")
except TypeError as e:
    print(f"Invalid data: {e}")
```

## Else and Finally

### The `else` Clause

Executes if no exception occurred:

```python
try:
    number = int(input("Enter a number: "))
except ValueError:
    print("Invalid number")
else:
    print(f"You entered: {number}")
    # This only runs if no exception occurred
```

**Use else for:**

- Code that should only run if try succeeded
- Keeping try block minimal (only code that might raise exception)

```python
try:
    file = open('data.txt', 'r')
except FileNotFoundError:
    print("File not found")
else:
    # Only runs if file opened successfully
    content = file.read()
    process(content)
    file.close()
```

### The `finally` Clause

Always executes, regardless of exceptions:

```python
try:
    file = open('data.txt', 'r')
    content = file.read()
except FileNotFoundError:
    print("File not found")
finally:
    # This always runs
    print("Cleanup complete")
```

**Use finally for:**

- Cleanup operations (closing files, connections)
- Releasing resources
- Operations that must execute

```python
file = None
try:
    file = open('data.txt', 'r')
    content = file.read()
    process(content)
except FileNotFoundError:
    print("File not found")
finally:
    if file:
        file.close()  # Always close the file
```

### Complete Structure

```python
try:
    # Code that might raise exception
    risky_operation()
except SpecificError:
    # Handle specific error
    handle_error()
else:
    # Runs if no exception
    continue_processing()
finally:
    # Always runs
    cleanup()
```

## Raising Exceptions

### Basic `raise`

```python
def divide(a, b):
    if b == 0:
        raise ValueError("Denominator cannot be zero")
    return a / b

divide(10, 0)  # ValueError: Denominator cannot be zero
```

### Raising Without Arguments

Re-raise the current exception:

```python
try:
    risky_operation()
except SomeError as e:
    log_error(e)
    raise  # Re-raise the same exception
```

### Raising Different Exception

```python
try:
    result = process_data()
except ValueError as e:
    # Catch one exception, raise another
    raise RuntimeError(f"Processing failed: {e}")
```

### Exception Chaining

Preserve original exception:

```python
try:
    result = int("abc")
except ValueError as e:
    raise TypeError("Invalid type") from e

# Shows both exceptions:
# TypeError: Invalid type
# The above exception was the direct cause of the following exception:
# ValueError: invalid literal for int()
```

## Common Built-in Exceptions

### ValueError

Raised when operation receives correct type but inappropriate value:

```python
int("abc")  # ValueError: invalid literal
int("123")  # OK

# Custom validation
def set_age(age):
    if age < 0:
        raise ValueError("Age cannot be negative")
    return age
```

### TypeError

Raised when operation applied to wrong type:

```python
"hello" + 5  # TypeError: can only concatenate str
len(5)       # TypeError: object of type 'int' has no len()

# Type checking
def add(a, b):
    if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
        raise TypeError("Arguments must be numbers")
    return a + b
```

### KeyError

Raised when dictionary key doesn't exist:

```python
d = {'a': 1, 'b': 2}
d['c']  # KeyError: 'c'

# Safe access
try:
    value = d['c']
except KeyError:
    value = None

# Better: use get()
value = d.get('c', None)
```

### IndexError

Raised when sequence index is out of range:

```python
items = [1, 2, 3]
items[5]  # IndexError: list index out of range

# Safe access
try:
    item = items[5]
except IndexError:
    item = None
```

### FileNotFoundError

Raised when file doesn't exist:

```python
with open('nonexistent.txt', 'r') as f:
    # FileNotFoundError
    pass

# Handle gracefully
try:
    with open('data.txt', 'r') as f:
        content = f.read()
except FileNotFoundError:
    print("File not found, using defaults")
    content = ""
```

### AttributeError

Raised when attribute doesn't exist:

```python
"hello".nonexistent_method()  # AttributeError

# Check before accessing
if hasattr(obj, 'method'):
    obj.method()
```

### ZeroDivisionError

Raised when dividing by zero:

```python
10 / 0  # ZeroDivisionError

def safe_divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return float('inf')  # Or None, or handle differently
```

## Exception Hierarchy

### Base Classes

```
BaseException
├── SystemExit
├── KeyboardInterrupt
└── Exception
    ├── ValueError
    ├── TypeError
    ├── KeyError
    ├── IndexError
    ├── FileNotFoundError
    ├── ZeroDivisionError
    ├── AttributeError
    └── ... (many more)
```

### Catching Base Exception

```python
# Catch all exceptions (usually too broad)
try:
    risky_code()
except Exception as e:
    # Catches most exceptions, but not SystemExit or KeyboardInterrupt
    handle_error(e)

# Catch absolutely everything (rarely needed)
try:
    risky_code()
except BaseException as e:
    # Catches everything, including Ctrl+C
    handle_error(e)
```

### Best Practice

Catch specific exceptions when possible:

```python
# Good - specific
try:
    number = int(user_input)
except ValueError:
    print("Invalid number")

# Bad - too broad
try:
    number = int(user_input)
except Exception:
    print("Something went wrong")
```

## Best Practices

### Keep Try Blocks Small

=== "Bad"

    ```python
    try:
        file = open('data.txt')
        content = file.read()
        data = json.loads(content)
        result = process(data)
        output = format_result(result)
        print(output)
    except Exception as e:
        print(f"Error: {e}")
    ```

=== "Good"

    ```python
    try:
        file = open('data.txt')
    except FileNotFoundError:
        print("File not found")
        return

    try:
        data = json.loads(file.read())
    except json.JSONDecodeError:
        print("Invalid JSON")
        return

    result = process(data)
    output = format_result(result)
    print(output)
    ```

### Don't Silence Exceptions

=== "Bad"

    ```python
    # Hides errors
    try:
        risky_operation()
    except:
        pass  # Silent failure
    ```

=== "Good"

    ```python
    # Handle or log
    try:
        risky_operation()
    except Exception as e:
        logger.error(f"Operation failed: {e}")
    # Or re-raise, or provide default
    ```

### Use Specific Exceptions

=== "Bad"

    ```python
    if something_wrong:
        raise Exception("Error")
    ```

=== "Good"

    ```python
    if value < 0:
        raise ValueError("Value must be non-negative")
    ```

### Don't Use Exceptions for Control Flow

=== "Bad"

    ```python
    # Exception for normal flow
    try:
        while True:
            item = iterator.next()
            process(item)
    except StopIteration:
        pass
    ```

=== "Good"

    ```python
    # Use proper iteration
    for item in iterator:
        process(item)
    ```

### Document Exceptions

```python
def divide(a, b):
    """
    Divide a by b.

    Args:
        a: Numerator
        b: Denominator

    Returns:
        The quotient

    Raises:
        ValueError: If b is zero
        TypeError: If arguments are not numbers
    """
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

## Summary

Proper error handling makes programs robust and easier to debug.

**Basic structure:**

- `try` - code that might raise exception
- `except` - handle specific exceptions
- `else` - runs if no exception (optional)
- `finally` - always runs (optional)

**Key concepts:**

- Catch specific exceptions when possible
- Use `else` for code that should only run on success
- Use `finally` for cleanup that must always occur
- Raise exceptions with descriptive messages

**Common exceptions:**

- `ValueError` - wrong value
- `TypeError` - wrong type
- `KeyError` - missing dictionary key
- `IndexError` - index out of range
- `FileNotFoundError` - file doesn't exist

**Best practices:**

- Keep try blocks minimal
- Don't silence exceptions
- Use specific exception types
- Document what exceptions functions raise

## Next Steps

- Organize code with [Modules and Imports](modules-and-imports.md)
- Apply error handling in [File Operations](file-operations.md)
- Explore advanced exception patterns in [Core Concepts](../core_concepts/)
