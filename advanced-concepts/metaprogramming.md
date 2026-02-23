# Metaprogramming

## Table of Contents

- [What is Metaprogramming?](#what-is-metaprogramming)
- [type() and Dynamic Class Creation](#type-and-dynamic-class-creation)
- [Metaclasses](#metaclasses)
- [**new** vs **init**](#__new__-vs-__init__)
- [Class Decorators vs Metaclasses](#class-decorators-vs-metaclasses)
- [Practical Applications](#practical-applications)
- [Summary](#summary)
- [Next Steps](#next-steps)

## What is Metaprogramming?

### Definition

Code that manipulates code - creating, modifying, or analyzing code at runtime.

**Python metaprogramming techniques:**

- Dynamic attribute access (`getattr`, `setattr`)
- Dynamic class creation (`type()`)
- Metaclasses
- Decorators
- `__getattr__`, `__setattr__`
- `exec()`, `eval()`

### Simple Example

```python
# Dynamic attribute access
class Person:
    def __init__(self, name):
        self.name = name

person = Person("Alice")

# Normal access
print(person.name)

# Dynamic access
attr_name = "name"
print(getattr(person, attr_name))  # Alice

# Dynamic setting
setattr(person, "age", 30)
print(person.age)  # 30
```

## type() and Dynamic Class Creation

### type() as Class Factory

`type()` can create classes at runtime:

```python
# Normal class definition
class Dog:
    def bark(self):
        return "Woof!"

# Equivalent dynamic creation
Dog = type('Dog', (), {'bark': lambda self: "Woof!"})

dog = Dog()
print(dog.bark())  # Woof!
```

**Syntax:** `type(name, bases, dict)`

- `name`: Class name
- `bases`: Tuple of base classes
- `dict`: Dictionary of attributes/methods

### With Base Classes

```python
class Animal:
    def sleep(self):
        return "Sleeping"

# Create class inheriting from Animal
Cat = type('Cat', (Animal,), {
    'meow': lambda self: "Meow!"
})

cat = Cat()
print(cat.meow())   # Meow!
print(cat.sleep())  # Sleeping (inherited)
```

### With Attributes

```python
# Class with attributes and methods
Person = type('Person', (), {
    'species': 'Homo sapiens',  # Class attribute
    '__init__': lambda self, name: setattr(self, 'name', name),
    'greet': lambda self: f"Hello, I'm {self.name}"
})

person = Person("Alice")
print(person.greet())   # Hello, I'm Alice
print(person.species)   # Homo sapiens
```

### Practical Use

```python
def create_model_class(table_name, fields):
    """Factory for database model classes."""

    def __init__(self, **kwargs):
        for field in fields:
            setattr(self, field, kwargs.get(field))

    def __repr__(self):
        field_str = ', '.join(f"{f}={getattr(self, f)}" for f in fields)
        return f"{table_name}({field_str})"

    return type(table_name, (), {
        '__init__': __init__,
        '__repr__': __repr__,
        'fields': fields
    })

# Create User class dynamically
User = create_model_class('User', ['id', 'name', 'email'])
user = User(id=1, name="Alice", email="alice@example.com")
print(user)  # User(id=1, name=Alice, email=alice@example.com)
```

## Metaclasses

### What are Metaclasses?

Classes that create classes. `type` is the default metaclass.

```python
# Every class is an instance of type
class MyClass:
    pass

print(type(MyClass))  # <class 'type'>
print(isinstance(MyClass, type))  # True

# MyClass is a class
# type is its metaclass
```

### Defining a Metaclass

```python
class Meta(type):
    """Custom metaclass."""

    def __new__(mcs, name, bases, attrs):
        """Called when creating class."""
        print(f"Creating class: {name}")
        return super().__new__(mcs, name, bases, attrs)

    def __init__(cls, name, bases, attrs):
        """Called after class is created."""
        print(f"Initializing class: {name}")
        super().__init__(name, bases, attrs)

class MyClass(metaclass=Meta):
    pass

# Output:
# Creating class: MyClass
# Initializing class: MyClass
```

### Metaclass **new**

```python
class UpperAttrMeta(type):
    """Convert all attribute names to uppercase."""

    def __new__(mcs, name, bases, attrs):
        uppercase_attrs = {}
        for attr_name, attr_value in attrs.items():
            if not attr_name.startswith('__'):
                uppercase_attrs[attr_name.upper()] = attr_value
            else:
                uppercase_attrs[attr_name] = attr_value

        return super().__new__(mcs, name, bases, uppercase_attrs)

class MyClass(metaclass=UpperAttrMeta):
    x = 1
    y = 2

    def method(self):
        return "hello"

obj = MyClass()
# Original names don't work
# print(obj.x)  # AttributeError

# Uppercase names work
print(obj.X)  # 1
print(obj.Y)  # 2
print(obj.METHOD())  # hello
```

### Validation Metaclass

```python
class ValidatedMeta(type):
    """Ensure class has required attributes."""

    def __new__(mcs, name, bases, attrs):
        if name != 'Base':  # Skip base class
            if 'validate' not in attrs:
                raise TypeError(f"{name} must have 'validate' method")

        return super().__new__(mcs, name, bases, attrs)

class Base(metaclass=ValidatedMeta):
    pass

class Valid(Base):
    def validate(self):
        return True

# class Invalid(Base):  # TypeError: Invalid must have 'validate' method
#     pass
```

### Singleton Metaclass

```python
class Singleton(type):
    """Ensure only one instance of class."""

    _instances = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=Singleton):
    def __init__(self):
        print("Connecting to database")

db1 = Database()  # Connecting to database
db2 = Database()  # No output - returns existing instance
print(db1 is db2)  # True
```

## **new** vs **init**

### **new** - Object Creation

```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        """Create and return new instance."""
        print(f"__new__ called for {cls}")
        instance = super().__new__(cls)
        return instance

    def __init__(self, value):
        """Initialize instance."""
        print(f"__init__ called with value={value}")
        self.value = value

obj = MyClass(42)
# __new__ called for <class '__main__.MyClass'>
# __init__ called with value=42
```

### Immutable Types

`__new__` required for immutable types:

```python
class PositiveInt(int):
    def __new__(cls, value):
        if value < 0:
            raise ValueError("Value must be positive")
        return super().__new__(cls, value)

# Can't use __init__ - int is immutable
num = PositiveInt(42)   # OK
# num = PositiveInt(-5)  # ValueError
```

### Controlling Instance Creation

```python
class LimitedInstances:
    _instance_count = 0
    _max_instances = 3

    def __new__(cls, *args, **kwargs):
        if cls._instance_count >= cls._max_instances:
            raise RuntimeError(f"Cannot create more than {cls._max_instances} instances")

        cls._instance_count += 1
        return super().__new__(cls)

    def __init__(self, name):
        self.name = name

obj1 = LimitedInstances("A")  # OK
obj2 = LimitedInstances("B")  # OK
obj3 = LimitedInstances("C")  # OK
# obj4 = LimitedInstances("D")  # RuntimeError
```

## Class Decorators vs Metaclasses

### Class Decorator

Simpler alternative to metaclasses:

```python
def add_id(cls):
    """Add ID class attribute."""
    import random
    cls.id = random.randint(1000, 9999)
    return cls

@add_id
class MyClass:
    pass

print(MyClass.id)  # Random 4-digit number
```

### Equivalent Metaclass

```python
class AddIdMeta(type):
    def __new__(mcs, name, bases, attrs):
        import random
        attrs['id'] = random.randint(1000, 9999)
        return super().__new__(mcs, name, bases, attrs)

class MyClass(metaclass=AddIdMeta):
    pass

print(MyClass.id)  # Random 4-digit number
```

### When to Use Each

**Class Decorator:**

- Simpler syntax
- Works on single class
- Applied after class creation
- Easier to understand

**Metaclass:**

- Affects inheritance hierarchy
- More powerful (controls class creation)
- Can validate/modify base classes
- More complex

### Combining Both

```python
class Meta(type):
    def __new__(mcs, name, bases, attrs):
        attrs['from_meta'] = True
        return super().__new__(mcs, name, bases, attrs)

def decorator(cls):
    cls.from_decorator = True
    return cls

@decorator
class MyClass(metaclass=Meta):
    pass

print(MyClass.from_meta)       # True
print(MyClass.from_decorator)  # True
```

## Practical Applications

### ORM Example

```python
class Field:
    def __init__(self, field_type):
        self.field_type = field_type

class ModelMeta(type):
    """Metaclass for ORM models."""

    def __new__(mcs, name, bases, attrs):
        # Find all Field instances
        fields = {}
        for key, value in list(attrs.items()):
            if isinstance(value, Field):
                fields[key] = value
                attrs.pop(key)  # Remove from class attributes

        attrs['_fields'] = fields
        return super().__new__(mcs, name, bases, attrs)

class Model(metaclass=ModelMeta):
    """Base model class."""

    def __init__(self, **kwargs):
        for field_name in self._fields:
            setattr(self, field_name, kwargs.get(field_name))

    def __repr__(self):
        field_strs = [f"{k}={getattr(self, k)}" for k in self._fields]
        return f"{self.__class__.__name__}({', '.join(field_strs)})"

class User(Model):
    id = Field(int)
    name = Field(str)
    email = Field(str)

user = User(id=1, name="Alice", email="alice@example.com")
print(user)  # User(id=1, name=Alice, email=alice@example.com)
print(user._fields)  # {'id': <Field>, 'name': <Field>, 'email': <Field>}
```

### Registry Pattern

```python
class RegistryMeta(type):
    """Auto-register classes."""

    registry = {}

    def __new__(mcs, name, bases, attrs):
        cls = super().__new__(mcs, name, bases, attrs)
        if name != 'Plugin':  # Don't register base class
            mcs.registry[name] = cls
        return cls

class Plugin(metaclass=RegistryMeta):
    pass

class EmailPlugin(Plugin):
    pass

class SMSPlugin(Plugin):
    pass

print(RegistryMeta.registry)
# {'EmailPlugin': <class 'EmailPlugin'>, 'SMSPlugin': <class 'SMSPlugin'>}

# Factory function
def get_plugin(name):
    return RegistryMeta.registry[name]()

email = get_plugin('EmailPlugin')
```

### Automatic Method Addition

```python
class AddMethodsMeta(type):
    """Add CRUD methods automatically."""

    def __new__(mcs, name, bases, attrs):
        # Add standard CRUD methods
        attrs['create'] = lambda self: f"{name}.create()"
        attrs['read'] = lambda self: f"{name}.read()"
        attrs['update'] = lambda self: f"{name}.update()"
        attrs['delete'] = lambda self: f"{name}.delete()"

        return super().__new__(mcs, name, bases, attrs)

class User(metaclass=AddMethodsMeta):
    pass

user = User()
print(user.create())  # User.create()
print(user.read())    # User.read()
```

## Summary

Metaprogramming in Python:

**Dynamic class creation:**

- `type(name, bases, dict)` creates classes at runtime
- Useful for factories and code generation

**Metaclasses:**

- Classes that create classes
- Override `__new__` to control class creation
- Override `__init__` to initialize classes
- Default metaclass is `type`

****new** vs **init**:**

- `__new__`: Creates instance (returns object)
- `__init__`: Initializes instance (returns None)
- `__new__` required for immutable types

**Class decorators:**

- Simpler alternative to metaclasses
- Applied after class creation
- Can't affect inheritance

**Use cases:**

- ORM frameworks (SQLAlchemy-like)
- Plugin systems
- Automatic registration
- Validation and constraints
- Adding methods/attributes
- Singleton pattern

**When to use:**

- Metaclasses: Complex frameworks, inheritance control
- Decorators: Simple modifications, single classes
- `type()`: Dynamic class generation

Metaprogramming is powerful but complex - use judiciously!

## Next Steps

- Explore [Descriptors and Properties](descriptors-and-properties.md)
- Learn [Abstract Base Classes](abstract-base-classes.md)
- Apply in [Design Patterns](../patterns_and_practices/design-patterns.md)
