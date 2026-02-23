# Descriptors and Properties

## Table of Contents

- [What are Descriptors?](#what-are-descriptors)
- [The Descriptor Protocol](#the-descriptor-protocol)
- [Data vs Non-Data Descriptors](#data-vs-non-data-descriptors)
- [property Decorator](#property-decorator)
- [Practical Descriptor Examples](#practical-descriptor-examples)
- [Descriptor Use Cases](#descriptor-use-cases)
- [Summary](#summary)
- [Next Steps](#next-steps)

## What are Descriptors?

### Definition

Objects that define how attribute access is handled. They power properties, methods, class methods, and static methods.

```python
class Descriptor:
    def __get__(self, obj, objtype=None):
        """Get attribute value."""
        return "value"

    def __set__(self, obj, value):
        """Set attribute value."""
        pass

    def __delete__(self, obj):
        """Delete attribute."""
        pass

class MyClass:
    attr = Descriptor()  # Descriptor instance

obj = MyClass()
obj.attr  # Calls Descriptor.__get__
obj.attr = 42  # Calls Descriptor.__set__
del obj.attr  # Calls Descriptor.__delete__
```

## The Descriptor Protocol

### **get** Method

```python
class LoggedAccess:
    def __init__(self, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        value = obj.__dict__.get(self.name)
        print(f"Getting {self.name}: {value}")
        return value

class MyClass:
    x = LoggedAccess('_x')

    def __init__(self, x):
        self._x = x

obj = MyClass(42)
print(obj.x)  # Getting _x: 42
              # 42
```

### **set** Method

```python
class TypedProperty:
    def __init__(self, name, expected_type):
        self.name = name
        self.expected_type = expected_type

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name)

    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"Expected {self.expected_type}, got {type(value)}")
        obj.__dict__[self.name] = value

class Person:
    name = TypedProperty('_name', str)
    age = TypedProperty('_age', int)

    def __init__(self, name, age):
        self.name = name
        self.age = age

person = Person("Alice", 30)
print(person.name)  # Alice
person.age = 31  # OK
# person.age = "thirty"  # TypeError
```

### **delete** Method

```python
class Undeletable:
    def __get__(self, obj, objtype=None):
        return obj.__dict__.get('_value')

    def __set__(self, obj, value):
        obj.__dict__['_value'] = value

    def __delete__(self, obj):
        raise AttributeError("Cannot delete this attribute")

class MyClass:
    value = Undeletable()

    def __init__(self, value):
        self.value = value

obj = MyClass(42)
obj.value = 100  # OK
# del obj.value  # AttributeError: Cannot delete this attribute
```

## Data vs Non-Data Descriptors

### Data Descriptors

Have both `__get__` and `__set__` (or `__delete__`):

```python
class DataDescriptor:
    def __get__(self, obj, objtype=None):
        return obj.__dict__.get('_value', None)

    def __set__(self, obj, value):
        obj.__dict__['_value'] = value

class MyClass:
    x = DataDescriptor()

obj = MyClass()
obj.x = 10

# Data descriptor takes precedence over instance __dict__
obj.__dict__['x'] = 99
print(obj.x)  # 10 (descriptor wins)
```

### Non-Data Descriptors

Only have `__get__`:

```python
class NonDataDescriptor:
    def __get__(self, obj, objtype=None):
        return "from descriptor"

class MyClass:
    x = NonDataDescriptor()

obj = MyClass()
print(obj.x)  # from descriptor

# Instance __dict__ takes precedence
obj.__dict__['x'] = "from dict"
print(obj.x)  # from dict (instance wins)
```

### Lookup Order

1. Data descriptors from class (and bases)
2. Instance `__dict__`
3. Non-data descriptors from class (and bases)
4. `__getattr__()` if defined

## property Decorator

### Basic Usage

`property` is a data descriptor:

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        """Getter."""
        return self._radius

    @radius.setter
    def radius(self, value):
        """Setter with validation."""
        if value < 0:
            raise ValueError("Radius cannot be negative")
        self._radius = value

    @radius.deleter
    def radius(self):
        """Deleter."""
        print("Deleting radius")
        del self._radius

circle = Circle(5)
print(circle.radius)  # 5 (calls getter)
circle.radius = 10    # Calls setter
# circle.radius = -5  # ValueError
del circle.radius     # Deleting radius
```

### Read-Only Property

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        """Computed property - no setter."""
        return self.width * self.height

rect = Rectangle(5, 10)
print(rect.area)  # 50
# rect.area = 100  # AttributeError - can't set
```

### Property with Caching

```python
class Fibonacci:
    def __init__(self, n):
        self.n = n
        self._cache = None

    @property
    def value(self):
        if self._cache is None:
            print("Computing...")
            self._cache = self._compute_fib(self.n)
        return self._cache

    def _compute_fib(self, n):
        if n < 2:
            return n
        return self._compute_fib(n-1) + self._compute_fib(n-2)

fib = Fibonacci(10)
print(fib.value)  # Computing... 55
print(fib.value)  # 55 (cached)
```

## Practical Descriptor Examples

### Validated Attribute

```python
class Validator:
    def __init__(self, min_value=None, max_value=None):
        self.min_value = min_value
        self.max_value = max_value

    def __set_name__(self, owner, name):
        """Called when descriptor is assigned to class attribute."""
        self.name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, None)

    def __set__(self, obj, value):
        if self.min_value is not None and value < self.min_value:
            raise ValueError(f"Value must be >= {self.min_value}")
        if self.max_value is not None and value > self.max_value:
            raise ValueError(f"Value must be <= {self.max_value}")
        setattr(obj, self.name, value)

class Product:
    price = Validator(min_value=0)
    quantity = Validator(min_value=0, max_value=1000)

    def __init__(self, price, quantity):
        self.price = price
        self.quantity = quantity

product = Product(19.99, 50)
# product.price = -10  # ValueError
# product.quantity = 2000  # ValueError
```

### Lazy Property

```python
class LazyProperty:
    def __init__(self, func):
        self.func = func
        self.name = func.__name__

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self

        # Compute value once and replace descriptor with value
        value = self.func(obj)
        setattr(obj, self.name, value)
        return value

class DataProcessor:
    def __init__(self, filename):
        self.filename = filename

    @LazyProperty
    def data(self):
        """Load data only when first accessed."""
        print(f"Loading data from {self.filename}")
        with open(self.filename) as f:
            return f.read()

processor = DataProcessor('data.txt')
print("Created processor")
print(processor.data)  # Loading data from data.txt
print(processor.data)  # No loading message - cached
```

### Type Checking Descriptor

```python
class Typed:
    def __init__(self, expected_type):
        self.expected_type = expected_type

    def __set_name__(self, owner, name):
        self.name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name)

    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(
                f"{self.name} must be {self.expected_type.__name__}, "
                f"got {type(value).__name__}"
            )
        setattr(obj, self.name, value)

class Person:
    name = Typed(str)
    age = Typed(int)
    email = Typed(str)

    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

person = Person("Alice", 30, "alice@example.com")
# person.age = "thirty"  # TypeError: _age must be int, got str
```

## Descriptor Use Cases

### Database Field Descriptors

```python
class Field:
    def __init__(self, field_type, default=None):
        self.field_type = field_type
        self.default = default

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return obj.__dict__.get(self.name, self.default)

    def __set__(self, obj, value):
        if value is not None and not isinstance(value, self.field_type):
            raise TypeError(f"{self.name} must be {self.field_type}")
        obj.__dict__[self.name] = value

class Model:
    """Base model class."""
    pass

class User(Model):
    id = Field(int)
    username = Field(str)
    email = Field(str)
    active = Field(bool, default=True)

    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            setattr(self, key, value)

user = User(id=1, username="alice", email="alice@example.com")
print(user.active)  # True (default)
```

### Method Binding

How methods work - they're descriptors:

```python
class Function:
    """Simplified function descriptor."""

    def __init__(self, func):
        self.func = func

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self  # Accessed from class

        # Create bound method
        def bound_method(*args, **kwargs):
            return self.func(obj, *args, **kwargs)

        return bound_method

class MyClass:
    @Function
    def method(self, x):
        return f"self={self}, x={x}"

obj = MyClass()
print(obj.method(42))  # Binds obj as self
```

### Cached Property

```python
class cached_property:
    """Cache property value after first access."""

    def __init__(self, func):
        self.func = func

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self

        # Cache value in instance __dict__
        # Non-data descriptor, so __dict__ takes precedence after caching
        value = self.func(obj)
        setattr(obj, self.func.__name__, value)
        return value

class WebPage:
    def __init__(self, url):
        self.url = url

    @cached_property
    def content(self):
        """Fetch content once and cache."""
        print(f"Fetching {self.url}")
        return f"Content from {self.url}"

page = WebPage("http://example.com")
print(page.content)  # Fetching http://example.com
print(page.content)  # Cached - no fetching
```

### Logging Descriptor

```python
class LoggedAttribute:
    def __init__(self, default=None):
        self.default = default

    def __set_name__(self, owner, name):
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        value = getattr(obj, self.private_name, self.default)
        print(f"GET {self.name} = {value}")
        return value

    def __set__(self, obj, value):
        print(f"SET {self.name} = {value}")
        setattr(obj, self.private_name, value)

class Account:
    balance = LoggedAttribute(default=0)

    def __init__(self, balance):
        self.balance = balance

account = Account(100)  # SET balance = 100
print(account.balance)  # GET balance = 100
account.balance = 200   # SET balance = 200
```

## Summary

Descriptors and properties in Python:

**Descriptor protocol:**

- `__get__(self, obj, objtype)`: Get attribute
- `__set__(self, obj, value)`: Set attribute
- `__delete__(self, obj)`: Delete attribute
- `__set_name__(self, owner, name)`: Called when descriptor assigned

**Types:**

- **Data descriptors:** Have `__set__` or `__delete__`
- **Non-data descriptors:** Only have `__get__`

**Lookup order:**

1. Data descriptors (class and bases)
2. Instance `__dict__`
3. Non-data descriptors (class and bases)
4. `__getattr__()` if defined

**property decorator:**

- Built on descriptors
- Provides getter, setter, deleter
- Common for computed attributes

**Use cases:**

- Type validation
- Lazy loading
- Caching
- Logging
- Method binding
- ORM fields

Descriptors are a powerful low-level feature that powers many Python features like properties, methods, and class methods.

## Next Steps

- Apply in [Object-Oriented Programming](object-oriented-programming.md)
- Use with [Abstract Base Classes](abstract-base-classes.md)
- Explore [Metaprogramming](metaprogramming.md) patterns
