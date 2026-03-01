# Type Systems

## Type Hints Basics

### Why Type Hints?

**Benefits:**

- Better IDE autocomplete
- Catch bugs early with static analysis
- Self-documenting code
- Easier refactoring

**Note:** Type hints are optional and not enforced at runtime.

```python
def greet(name: str) -> str:
    return f"Hello, {name}!"

# Runtime doesn't enforce types
result = greet(123)  # Works at runtime, but type checker warns
```

### Basic Syntax

```python
# Variable annotations
age: int = 30
name: str = "Alice"
is_active: bool = True

# Function annotations
def add(a: int, b: int) -> int:
    return a + b

# No return value
def print_message(msg: str) -> None:
    print(msg)
```

## Built-in Types

### Simple Types

```python
# Basic types
count: int = 10
price: float = 19.99
name: str = "Alice"
is_valid: bool = True

# None type
def might_return_none() -> int | None:
    return None

# Any value (no type checking)
from typing import Any
value: Any = "can be anything"
```

### Collections

```python
# Lists
numbers: list[int] = [1, 2, 3]
names: list[str] = ["Alice", "Bob"]

# Tuples (fixed length)
point: tuple[int, int] = (10, 20)
record: tuple[str, int, bool] = ("Alice", 30, True)

# Tuples (variable length)
coords: tuple[float, ...] = (1.0, 2.0, 3.0, 4.0)

# Dictionaries
scores: dict[str, int] = {"Alice": 90, "Bob": 85}
mapping: dict[int, str] = {1: "one", 2: "two"}

# Sets
unique_ids: set[int] = {1, 2, 3}
```

### Optional Values

```python
# Python 3.10+ Union syntax
def find_user(id: int) -> User | None:
    pass

# Older syntax (still valid)
from typing import Optional
def find_user(id: int) -> Optional[User]:
    pass

# Multiple types
def process(value: int | str | float) -> bool:
    pass
```

## Typing Module

### Common Types

```python
from typing import List, Dict, Set, Tuple, Optional, Union

# Before Python 3.9, use typing versions
numbers: List[int] = [1, 2, 3]
mapping: Dict[str, int] = {"a": 1}
unique: Set[str] = {"a", "b"}

# Python 3.9+: use built-in types
numbers: list[int] = [1, 2, 3]
mapping: dict[str, int] = {"a": 1}
unique: set[str] = {"a", "b"}
```

### Callable

```python
from typing import Callable

# Function that takes int, str and returns bool
Validator = Callable[[int, str], bool]

def register_validator(validator: Validator) -> None:
    pass

def my_validator(id: int, name: str) -> bool:
    return id > 0 and len(name) > 0

register_validator(my_validator)
```

### Sequence and Iterable

```python
from typing import Sequence, Iterable

# Sequence (indexable)
def process_sequence(items: Sequence[int]) -> int:
    return items[0] + items[1]

# Works with lists, tuples, etc.
process_sequence([1, 2, 3])
process_sequence((1, 2, 3))

# Iterable (can be iterated)
def sum_items(items: Iterable[int]) -> int:
    return sum(items)

# Works with lists, tuples, generators, etc.
sum_items([1, 2, 3])
sum_items(x for x in range(10))
```

### TypeAlias

```python
from typing import TypeAlias

# Define type aliases for clarity
UserId: TypeAlias = int
UserName: TypeAlias = str
UserData: TypeAlias = dict[str, str | int]

def get_user(id: UserId) -> UserData:
    return {"name": "Alice", "age": 30}

def create_user(name: UserName, age: int) -> UserId:
    return 123
```

## Function Annotations

### Arguments and Returns

```python
def greet(name: str, age: int) -> str:
    return f"{name} is {age} years old"

# Multiple return values (tuple)
def min_max(numbers: list[int]) -> tuple[int, int]:
    return min(numbers), max(numbers)

# No return
def log_message(message: str) -> None:
    print(message)
```

### Default Arguments

```python
def greet(name: str, greeting: str = "Hello") -> str:
    return f"{greeting}, {name}!"

# Optional with default None
def process(data: str, logger: Logger | None = None) -> None:
    if logger:
        logger.log(data)
```

### \*args and \*\*kwargs

```python
def sum_all(*args: int) -> int:
    return sum(args)

def configure(**kwargs: str) -> dict[str, str]:
    return kwargs

# More specific kwargs
def create_user(**attrs: int | str | bool) -> User:
    pass
```

### Overload

Different signatures for same function:

```python
from typing import overload

@overload
def process(data: int) -> int: ...

@overload
def process(data: str) -> str: ...

def process(data: int | str) -> int | str:
    if isinstance(data, int):
        return data * 2
    return data.upper()
```

## Generics

### Generic Functions

```python
from typing import TypeVar

T = TypeVar('T')

def first(items: list[T]) -> T:
    return items[0]

# Type checker infers return type
numbers = [1, 2, 3]
num = first(numbers)  # Type is int

names = ["Alice", "Bob"]
name = first(names)  # Type is str
```

### Generic Classes

```python
from typing import Generic, TypeVar

T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self._items: list[T] = []

    def push(self, item: T) -> None:
        self._items.append(item)

    def pop(self) -> T:
        return self._items.pop()

    def is_empty(self) -> bool:
        return len(self._items) == 0

# Type-safe stack
int_stack: Stack[int] = Stack()
int_stack.push(1)
int_stack.push(2)
value: int = int_stack.pop()

str_stack: Stack[str] = Stack()
str_stack.push("hello")
```

### Constrained TypeVars

```python
from typing import TypeVar

# Only numeric types
Number = TypeVar('Number', int, float)

def add(a: Number, b: Number) -> Number:
    return a + b

add(1, 2)      # OK: int
add(1.5, 2.5)  # OK: float
# add("a", "b")  # Error: str not allowed
```

### Bounded TypeVars

```python
from typing import TypeVar

class Animal:
    def make_sound(self) -> str:
        return "Some sound"

class Dog(Animal):
    def make_sound(self) -> str:
        return "Woof"

# T must be Animal or subclass
T = TypeVar('T', bound=Animal)

def make_noise(animal: T) -> T:
    print(animal.make_sound())
    return animal

dog = Dog()
make_noise(dog)  # OK
# make_noise("cat")  # Error: str is not Animal
```

## Protocols

### Structural Subtyping

Duck typing with type safety:

```python
from typing import Protocol

class Drawable(Protocol):
    """Anything with a draw method."""
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

def render(shape: Drawable) -> None:
    shape.draw()

# Both work - no inheritance needed
render(Circle())
render(Square())
```

### Runtime Checkable Protocols

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Closeable(Protocol):
    def close(self) -> None: ...

class File:
    def close(self) -> None:
        print("Closing file")

f = File()
isinstance(f, Closeable)  # True
```

### Protocol with Properties

```python
from typing import Protocol

class Sized(Protocol):
    @property
    def size(self) -> int: ...

class Container:
    def __init__(self, items: list):
        self._items = items

    @property
    def size(self) -> int:
        return len(self._items)

def process(obj: Sized) -> int:
    return obj.size * 2

container = Container([1, 2, 3])
process(container)  # OK
```

## Type Checking Tools

Type checkers validate type hints without running code:

**Popular type checkers:**

- **mypy**: Most widely used, highly configurable
- **pyright**: Fast, excellent VS Code integration
- **pyre**: Facebook's type checker

```bash
# Basic usage
pip install mypy
mypy script.py
```

> **Note:** For comprehensive coverage of type checker configuration, usage patterns, and integration strategies, see:
>
> - [Type Checking](../testing_and_quality/type-checking.md) - Configuration and testing integration
> - [Code Quality Tools](../testing_and_quality/code-quality-tools.md) - Tools overview

### Type Checking in Code

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    # Only imported during type checking, not runtime
    from expensive_module import HeavyClass

def process(data: 'HeavyClass') -> None:
    # Forward reference as string
    pass
```

### Ignoring Type Errors

```python
# Ignore one line
result = some_function()  # type: ignore

# Ignore specific error
value = "text"  # type: ignore[assignment]

# Ignore for whole file
# type: ignore (at top of file)
```

## Gradual Typing

### Adopting Types Incrementally

```python
# Start with no types
def process(data):
    return data.upper()

# Add types to new code
def process_new(data: str) -> str:
    return data.upper()

# Gradually add to existing code
def process(data: str) -> str:  # Added types
    return data.upper()
```

### Any Type

```python
from typing import Any

# Escape hatch - no type checking
def legacy_function(data: Any) -> Any:
    return data.process().result()

# Use sparingly during migration
```

### cast Function

```python
from typing import cast

def get_value() -> object:
    return "hello"

# Tell type checker the actual type
value = cast(str, get_value())
print(value.upper())  # Type checker knows it's str
```

### Stub Files

Type hints in separate `.pyi` files:

**module.py:**

```python
def process(data):
    return data.upper()
```

**module.pyi:**

```python
def process(data: str) -> str: ...
```

## Summary

Type systems in Python:

**Type hints:**

- Optional annotations for variables and functions
- Not enforced at runtime
- Improve IDE support and catch bugs early

**Basic types:**

- Built-in: `int`, `str`, `bool`, `list`, `dict`, etc.
- Special: `None`, `Any`, `Union` (`|`)

**typing module:**

- `Callable`, `Sequence`, `Iterable`
- `Optional`, `Union`, `TypeAlias`
- Generic types with `TypeVar`

**Protocols:**

- Structural subtyping (duck typing with types)
- Define interfaces without inheritance
- Runtime checkable with decorator

**Type checkers:**

- mypy: Popular, configurable
- pyright: Fast, good VS Code integration
- Use in CI/CD for validation

**Gradual typing:**

- Add types incrementally
- Use `Any` during migration
- Stub files (`.pyi`) for untyped code

Type hints make Python code more maintainable and catch errors earlier in development.

## Next Steps

- Master type checking with [Type Checking](../testing_and_quality/type-checking.md)
- Apply in [Writing Clean Code](../patterns_and_practices/writing-clean-code.md)
- Use with [Testing](../testing_and_quality/unit-testing.md)
