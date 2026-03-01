# Functions

## Defining Functions

### Basic Function Syntax

```python
def function_name(parameters):
    """Docstring describing the function"""
    # Function body
    return result
```

**Example:**

```python
def greet(name):
    """Return a greeting message."""
    return f"Hello, {name}!"

# Call the function
message = greet("Alice")
print(message)  # Hello, Alice!
```

### Functions Without Return

If no `return` statement, function returns `None`:

```python
def print_greeting(name):
    print(f"Hello, {name}!")
    # No return statement

result = print_greeting("Bob")  # Prints: Hello, Bob!
print(result)  # None
```

### Multiple Statements

```python
def calculate_circle_area(radius):
    """Calculate and return the area of a circle."""
    pi = 3.14159
    area = pi * radius ** 2
    return area

area = calculate_circle_area(5)
print(area)  # 78.53975
```

## Parameters and Arguments

### Positional Parameters

Arguments passed by position:

```python
def divide(a, b):
    return a / b

divide(10, 2)  # 5.0 - a=10, b=2
divide(2, 10)  # 0.2 - a=2, b=10 (order matters!)
```

### Keyword Arguments

Arguments passed by name:

```python
def describe_pet(animal, name):
    print(f"I have a {animal} named {name}.")

# Positional
describe_pet("dog", "Buddy")

# Keyword (order doesn't matter)
describe_pet(name="Buddy", animal="dog")
describe_pet(animal="cat", name="Whiskers")

# Mix positional and keyword (positional must come first)
describe_pet("hamster", name="Nibbles")
```

### Parameter vs Argument

- **Parameter**: Variable in function definition
- **Argument**: Actual value passed when calling

```python
def greet(name):  # 'name' is a parameter
    return f"Hello, {name}!"

greet("Alice")  # "Alice" is an argument
```

## Return Values

### Single Return Value

```python
def square(x):
    return x ** 2

result = square(5)  # 25
```

### Multiple Return Values

Functions can return multiple values as a tuple:

```python
def min_max(numbers):
    return min(numbers), max(numbers)

minimum, maximum = min_max([1, 5, 3, 9, 2])
print(minimum, maximum)  # 1 9

# Actually returns a tuple
result = min_max([1, 5, 3, 9, 2])
print(result)  # (1, 9)
print(type(result))  # <class 'tuple'>
```

### Early Returns

```python
def find_item(items, target):
    """Return index of target, or -1 if not found."""
    for i, item in enumerate(items):
        if item == target:
            return i  # Early return
    return -1  # If loop completes without finding

index = find_item([1, 2, 3, 4], 3)  # 2
index = find_item([1, 2, 3, 4], 10)  # -1
```

### Return None Explicitly

```python
def process(value):
    if value < 0:
        return None  # Explicit None
    return value * 2

result = process(-5)  # None
result = process(5)   # 10
```

## Scope and Namespaces

### Local Scope

Variables created inside a function are local:

```python
def my_function():
    x = 10  # Local variable
    print(x)

my_function()  # 10
print(x)  # NameError - x doesn't exist outside function
```

### Global Scope

Variables defined outside functions are global:

```python
x = 100  # Global variable

def my_function():
    print(x)  # Can read global

my_function()  # 100
```

### Modifying Global Variables

Use `global` keyword to modify global variables:

```python
counter = 0

def increment():
    global counter  # Declare we're using global variable
    counter += 1

increment()
print(counter)  # 1

increment()
print(counter)  # 2
```

**Without `global`:**

```python
counter = 0

def increment():
    counter = 1  # Creates local variable, doesn't modify global

increment()
print(counter)  # 0 (unchanged)
```

### LEGB Rule

Python searches for variables in this order:

1. **L**ocal - inside current function
2. **E**nclosing - in enclosing function(s)
3. **G**lobal - module level
4. **B**uilt-in - Python built-ins

```python
x = "global"

def outer():
    x = "enclosing"

    def inner():
        x = "local"
        print(x)  # Searches: local → enclosing → global → built-in

    inner()  # Prints: local
    print(x)  # Prints: enclosing

outer()
print(x)  # Prints: global
```

## Default Arguments

### Basic Default Values

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

greet("Alice")  # Hello, Alice! (uses default)
greet("Bob", "Hi")  # Hi, Bob! (overrides default)
greet("Charlie", greeting="Hey")  # Hey, Charlie!
```

### Multiple Defaults

```python
def connect(host="localhost", port=8080, timeout=30):
    print(f"Connecting to {host}:{port} (timeout: {timeout}s)")

connect()  # All defaults
connect("example.com")  # Custom host, default port and timeout
connect("example.com", 443)  # Custom host and port
connect(port=9000)  # Custom port, keyword argument
```

### Mutable Default Argument Trap

**Danger:** Mutable defaults are created once and shared:

```python
def add_item(item, items=[]):  # BAD - mutable default
    items.append(item)
    return items

list1 = add_item("apple")  # ['apple']
list2 = add_item("banana")  # ['apple', 'banana'] - UNEXPECTED!
```

**Fix:** Use `None` and create new object inside:

```python
def add_item(item, items=None):  # GOOD
    if items is None:
        items = []
    items.append(item)
    return items

list1 = add_item("apple")  # ['apple']
list2 = add_item("banana")  # ['banana'] - correct!
```

## Variable-Length Arguments

### \*args - Variable Positional Arguments

Collect extra positional arguments into a tuple:

```python
def sum_all(*numbers):
    """Sum any number of arguments."""
    total = 0
    for num in numbers:
        total += num
    return total

sum_all(1, 2, 3)  # 6
sum_all(1, 2, 3, 4, 5)  # 15
sum_all()  # 0

# numbers is a tuple
def print_args(*args):
    print(type(args))  # <class 'tuple'>
    print(args)

print_args(1, 2, 3)  # (1, 2, 3)
```

### Mixing Regular and \*args

```python
def greet(greeting, *names):
    """Greet multiple people."""
    for name in names:
        print(f"{greeting}, {name}!")

greet("Hello", "Alice", "Bob", "Charlie")
# Hello, Alice!
# Hello, Bob!
# Hello, Charlie!
```

### Unpacking Arguments with \*

```python
def add(a, b, c):
    return a + b + c

numbers = [1, 2, 3]
result = add(*numbers)  # Unpacks list into arguments
print(result)  # 6

# Equivalent to:
result = add(numbers[0], numbers[1], numbers[2])
```

## Keyword Arguments

### \*\*kwargs - Variable Keyword Arguments

Collect extra keyword arguments into a dictionary:

```python
def print_info(**kwargs):
    """Print any number of keyword arguments."""
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_info(name="Alice", age=30, city="NYC")
# name: Alice
# age: 30
# city: NYC

# kwargs is a dictionary
def show_kwargs(**kwargs):
    print(type(kwargs))  # <class 'dict'>
    print(kwargs)

show_kwargs(a=1, b=2)  # {'a': 1, 'b': 2}
```

### Combining \*args and \*\*kwargs

```python
def func(a, b, *args, **kwargs):
    print(f"a={a}, b={b}")
    print(f"args={args}")
    print(f"kwargs={kwargs}")

func(1, 2, 3, 4, 5, x=10, y=20)
# a=1, b=2
# args=(3, 4, 5)
# kwargs={'x': 10, 'y': 20}
```

**Order matters:**

1. Regular positional parameters
2. \*args
3. Keyword-only parameters (after \*)
4. \*\*kwargs

```python
def func(pos1, pos2, *args, kw_only, **kwargs):
    pass

func(1, 2, 3, 4, kw_only=5, extra=6)
```

### Unpacking Keyword Arguments with \*\*

```python
def greet(name, age):
    print(f"{name} is {age} years old")

person = {"name": "Alice", "age": 30}
greet(**person)  # Unpacks dict into keyword arguments
# Alice is 30 years old

# Equivalent to:
greet(name=person["name"], age=person["age"])
```

## Function Documentation

### Docstrings

First string in function body documents the function:

```python
def calculate_area(radius):
    """
    Calculate the area of a circle.

    Args:
        radius: The radius of the circle.

    Returns:
        The area of the circle.
    """
    return 3.14159 * radius ** 2

# Access docstring
print(calculate_area.__doc__)
help(calculate_area)
```

### Docstring Styles

**One-line docstring:**

```python
def square(x):
    """Return the square of x."""
    return x ** 2
```

**Multi-line docstring (Google style):**

```python
def divide(a, b):
    """
    Divide two numbers.

    Args:
        a: The numerator.
        b: The denominator.

    Returns:
        The quotient of a divided by b.

    Raises:
        ValueError: If b is zero.
    """
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

**Type hints in docstrings:**

```python
def greet(name: str) -> str:
    """
    Return a greeting message.

    Args:
        name: Person's name (str).

    Returns:
        Greeting message (str).
    """
    return f"Hello, {name}!"
```

## First-Class Functions

Functions are objects - can be assigned, passed, and returned:

### Assigning Functions to Variables

```python
def greet(name):
    return f"Hello, {name}!"

# Assign to variable
say_hello = greet
print(say_hello("Alice"))  # Hello, Alice!
```

### Passing Functions as Arguments

```python
def apply_operation(func, value):
    """Apply a function to a value."""
    return func(value)

def square(x):
    return x ** 2

def cube(x):
    return x ** 3

result = apply_operation(square, 5)  # 25
result = apply_operation(cube, 5)    # 125
```

### Returning Functions

```python
def make_multiplier(n):
    """Return a function that multiplies by n."""
    def multiplier(x):
        return x * n
    return multiplier

times_two = make_multiplier(2)
times_three = make_multiplier(3)

print(times_two(5))    # 10
print(times_three(5))  # 15
```

### Storing Functions in Data Structures

```python
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

# Dictionary of functions
operations = {
    '+': add,
    '-': subtract,
    '*': multiply
}

result = operations['+'](5, 3)  # 8
result = operations['*'](5, 3)  # 15
```

## Summary

Functions promote code reuse, organization, and abstraction - essential for writing maintainable programs.

**Defining functions:**

- `def name(params):` with optional `return`
- Docstrings for documentation

**Parameters and arguments:**

- Positional and keyword arguments
- Default parameter values
- `*args` for variable positional arguments
- `**kwargs` for variable keyword arguments

**Scope:**

- LEGB rule: Local → Enclosing → Global → Built-in
- Use `global` to modify global variables

**Lambda functions:**

- `lambda params: expression` for simple anonymous functions
- Best for short, one-time-use functions

**First-class functions:**

- Functions are objects
- Can be assigned, passed, and returned
- Enable functional programming patterns

## Next Steps

- Apply functions to [File Operations](file-operations.md)
- Use with [Error Handling](error-handling.md) for robust code
- Explore advanced function concepts in [Core Concepts](../core_concepts/advanced-functions.md)
