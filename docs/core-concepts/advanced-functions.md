# Advanced Functions

## First-Class Functions

### Functions as Objects

Functions are objects that can be passed around:

```python
def greet(name):
    return f"Hello, {name}!"

# Assign to variable
say_hello = greet
print(say_hello("Alice"))  # Hello, Alice!

# Store in data structures
functions = [greet, str.upper, len]
for func in functions:
    print(func)
```

### Passing Functions as Arguments

```python
def apply_operation(func, value):
    """Apply function to value."""
    return func(value)

def square(x):
    return x ** 2

def cube(x):
    return x ** 3

print(apply_operation(square, 5))  # 25
print(apply_operation(cube, 5))    # 125
print(apply_operation(len, "hello"))  # 5
```

### Returning Functions

```python
def get_multiplier(n):
    """Return function that multiplies by n."""
    def multiplier(x):
        return x * n
    return multiplier

times_two = get_multiplier(2)
times_three = get_multiplier(3)

print(times_two(5))    # 10
print(times_three(5))  # 15
```

## Nested Functions

### Defining Functions Inside Functions

```python
def outer(x):
    """Outer function."""

    def inner(y):
        """Inner function - only accessible inside outer."""
        return x + y

    return inner(10)

print(outer(5))  # 15
# print(inner(5))  # NameError - inner not accessible
```

### Helper Functions

```python
def process_data(data):
    """Process data with helper functions."""

    def validate(item):
        """Local helper - not exposed outside."""
        return item is not None and item > 0

    def transform(item):
        """Another local helper."""
        return item ** 2

    # Use helpers
    valid_data = [item for item in data if validate(item)]
    return [transform(item) for item in valid_data]

result = process_data([1, -2, 3, None, 5])
print(result)  # [1, 9, 25]
```

## Closures

### Capturing Outer Variables

Function that remembers variables from enclosing scope:

```python
def make_counter():
    """Return counter function."""
    count = 0  # Enclosed variable

    def counter():
        nonlocal count  # Modify enclosed variable
        count += 1
        return count

    return counter

# Each counter has its own enclosed state
counter1 = make_counter()
counter2 = make_counter()

print(counter1())  # 1
print(counter1())  # 2
print(counter1())  # 3

print(counter2())  # 1 - separate state
print(counter2())  # 2
```

### Closure Properties

```python
def make_multiplier(factor):
    """Return function that multiplies by factor."""
    def multiply(x):
        return x * factor  # Captures 'factor'
    return multiply

times_two = make_multiplier(2)
times_three = make_multiplier(3)

print(times_two(5))    # 10
print(times_three(5))  # 15

# Inspect closure
print(times_two.__closure__)  # Shows captured variables
print(times_two.__closure__[0].cell_contents)  # 2
```

### Practical Example

```python
def make_logger(prefix):
    """Return logging function with prefix."""
    def log(message):
        print(f"[{prefix}] {message}")
    return log

info_log = make_logger("INFO")
error_log = make_logger("ERROR")

info_log("Application started")   # [INFO] Application started
error_log("Connection failed")    # [ERROR] Connection failed
```

## Function Decorators

### Basic Decorator

Modify or enhance function behavior:

```python
def uppercase_decorator(func):
    """Decorator that uppercases function result."""
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result.upper()
    return wrapper

@uppercase_decorator
def greet(name):
    return f"hello, {name}!"

print(greet("alice"))  # HELLO, ALICE!

# Equivalent to:
# greet = uppercase_decorator(greet)
```

### Multiple Decorators

```python
def bold(func):
    def wrapper(*args, **kwargs):
        return f"<b>{func(*args, **kwargs)}</b>"
    return wrapper

def italic(func):
    def wrapper(*args, **kwargs):
        return f"<i>{func(*args, **kwargs)}</i>"
    return wrapper

@bold
@italic
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))  # <b><i>Hello, Alice!</i></b>

# Applied bottom to top:
# greet = bold(italic(greet))
```

### Timing Decorator

```python
import time

def timer(func):
    """Measure function execution time."""
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "Done"

result = slow_function()
# slow_function took 1.0023 seconds
```

### Preserving Metadata

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # Preserves original function metadata
    def wrapper(*args, **kwargs):
        """Wrapper docstring."""
        return func(*args, **kwargs)
    return wrapper

@my_decorator
def greet(name):
    """Greet someone by name."""
    return f"Hello, {name}!"

print(greet.__name__)  # greet (not wrapper)
print(greet.__doc__)   # Greet someone by name.
```

## Decorator with Arguments

### Parameterized Decorators

```python
def repeat(times):
    """Decorator that repeats function execution."""
    def decorator(func):
        def wrapper(*args, **kwargs):
            results = []
            for _ in range(times):
                results.append(func(*args, **kwargs))
            return results
        return wrapper
    return decorator

@repeat(times=3)
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))
# ['Hello, Alice!', 'Hello, Alice!', 'Hello, Alice!']
```

### Flexible Decorator

```python
def debug(prefix="DEBUG"):
    """Decorator with optional argument."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"[{prefix}] Calling {func.__name__}")
            result = func(*args, **kwargs)
            print(f"[{prefix}] {func.__name__} returned {result}")
            return result
        return wrapper
    return decorator

@debug(prefix="INFO")
def add(a, b):
    return a + b

print(add(3, 5))
# [INFO] Calling add
# [INFO] add returned 8
# 8
```

### Validation Decorator

```python
def validate_args(**validations):
    """Validate function arguments."""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # Validate kwargs
            for param, validator in validations.items():
                if param in kwargs:
                    if not validator(kwargs[param]):
                        raise ValueError(f"Invalid {param}")
            return func(*args, **kwargs)
        return wrapper
    return decorator

@validate_args(
    age=lambda x: x >= 0,
    name=lambda x: len(x) > 0
)
def create_user(name, age):
    return f"User: {name}, Age: {age}"

print(create_user(name="Alice", age=30))  # OK
# create_user(name="", age=30)  # ValueError
# create_user(name="Bob", age=-5)  # ValueError
```

## Class Decorators

### Decorating Classes

```python
def singleton(cls):
    """Ensure class has only one instance."""
    instances = {}

    @wraps(cls)
    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return get_instance

@singleton
class Database:
    def __init__(self):
        print("Database initialized")

db1 = Database()  # Database initialized
db2 = Database()  # No output - same instance
print(db1 is db2)  # True
```

### Adding Methods

```python
def add_repr(cls):
    """Add __repr__ method to class."""
    def __repr__(self):
        attrs = ', '.join(f"{k}={v}" for k, v in self.__dict__.items())
        return f"{cls.__name__}({attrs})"

    cls.__repr__ = __repr__
    return cls

@add_repr
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)
print(p)  # Point(x=3, y=4)
```

### Registering Classes

```python
registry = {}

def register(name):
    """Register class in registry."""
    def decorator(cls):
        registry[name] = cls
        return cls
    return decorator

@register('user')
class User:
    pass

@register('product')
class Product:
    pass

print(registry)  # {'user': <class 'User'>, 'product': <class 'Product'>}

# Factory pattern
def create(name):
    return registry[name]()

user = create('user')
```

## `functools` Module

### @lru_cache

Memoization for expensive functions:

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    """Cached fibonacci - much faster."""
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

print(fibonacci(100))  # Fast with caching

# Check cache info
print(fibonacci.cache_info())
# CacheInfo(hits=98, misses=101, maxsize=128, currsize=101)

# Clear cache
fibonacci.cache_clear()
```

### @wraps

Preserve function metadata:

```python
from functools import wraps

def my_decorator(func):
    @wraps(func)  # Copy metadata from func
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

### @total_ordering

Implement comparison operators:

=== "main.py"

    ```python
    from student import Student

    alice = Student("Alice", 90)
    bob = Student("Bob", 85)

    print(alice > bob)   # True
    print(alice <= bob)  # False
    ```

=== "student.py"

    ```python
    from functools import total_ordering

    @total_ordering
    class Student:
        def __init__(self, name, grade):
            self.name = name
            self.grade = grade

        def __eq__(self, other):
            return self.grade == other.grade

        def __lt__(self, other):
            return self.grade < other.grade

        # Other comparisons (>, <=, >=) auto-generated
    ```

## Partial Functions

### Creating Partial Functions

Pre-fill some arguments:

```python
from functools import partial

def power(base, exponent):
    return base ** exponent

# Create specialized functions
square = partial(power, exponent=2)
cube = partial(power, exponent=3)

print(square(5))  # 25
print(cube(5))    # 125
```

### Practical Examples

```python
from functools import partial

# Base logger
def log(message, level="INFO"):
    print(f"[{level}] {message}")

# Specialized loggers
info = partial(log, level="INFO")
error = partial(log, level="ERROR")
debug = partial(log, level="DEBUG")

info("Application started")    # [INFO] Application started
error("Connection failed")     # [ERROR] Connection failed
debug("Variable value: 42")    # [DEBUG] Variable value: 42
```

## Function Attributes

### Custom Attributes

Functions can have attributes:

```python
def counter():
    counter.calls += 1
    return counter.calls

counter.calls = 0

print(counter())  # 1
print(counter())  # 2
print(counter())  # 3

print(counter.calls)  # 3
```

### Using for Memoization

=== "main.py"

    ```python
    from memoized_fib import memoized_fib

    print(memoized_fib(100))
    print(len(memoized_fib.cache))  # 101 entries cached
    ```

=== "memoized_fib.py"

    ```python
    def memoized_fib(n):
        """Fibonacci with manual memoization."""
        if not hasattr(memoized_fib, 'cache'):
            memoized_fib.cache = {}

        if n in memoized_fib.cache:
            return memoized_fib.cache[n]

        if n < 2:
            result = n
        else:
            result = memoized_fib(n-1) + memoized_fib(n-2)

        memoized_fib.cache[n] = result
        return result
    ```

### Introspection

```python
def greet(name: str) -> str:
    """Greet someone by name."""
    return f"Hello, {name}!"

# Function metadata
print(greet.__name__)          # greet
print(greet.__doc__)           # Greet someone by name.
print(greet.__module__)        # __main__
print(greet.__annotations__)   # {'name': <class 'str'>, 'return': <class 'str'>}

# Get signature
import inspect
sig = inspect.signature(greet)
print(sig)  # (name: str) -> str
```

## Summary

Advanced functions enable powerful metaprogramming and code organization.

**First-class functions:**

- Functions are objects
- Can be passed, returned, stored

**Closures:**

- Functions that capture variables from enclosing scope
- Enable data hiding and factory patterns

**Decorators:**

- Modify function behavior
- `@decorator_name` syntax
- Can accept arguments
- Use `@wraps` to preserve metadata

**Class decorators:**

- Modify or enhance classes
- Singleton pattern, registration, etc.

**functools module:**

- `@lru_cache`: memoization
- `@wraps`: preserve metadata
- `@total_ordering`: comparison operators
- `partial`: pre-fill arguments

**Function attributes:**

- Functions can have custom attributes
- Useful for state, caching, introspection

## Next Steps

- Explore [Iterators and Generators](iterators-and-generators.md) for lazy evaluation
- Learn [Context Managers](context-managers.md) for resource management
- Study [Metaprogramming](metaprogramming.md) for advanced techniques
