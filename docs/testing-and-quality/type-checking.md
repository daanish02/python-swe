# Type Checking

> **Note:** This guide focuses on **using type checkers** (mypy, pyright) for code quality and testing. For **type hints syntax and language features**, see [Type Systems](../core_concepts/type-systems.md).

## Type Hints Basics

### Basic Types

```python
# Built-in types
def greet(name: str) -> str:
    return f"Hello, {name}"

def add(x: int, y: int) -> int:
    return x + y

def divide(x: float, y: float) -> float:
    return x / y

def is_valid(flag: bool) -> bool:
    return not flag

# Variables
age: int = 30
name: str = "Alice"
price: float = 19.99
active: bool = True
```

### Collections

```python
from typing import List, Dict, Set, Tuple

# Lists
names: List[str] = ["Alice", "Bob", "Charlie"]
numbers: List[int] = [1, 2, 3, 4, 5]

def sum_numbers(numbers: List[int]) -> int:
    return sum(numbers)

# Dictionaries
user: Dict[str, str] = {"name": "Alice", "email": "alice@example.com"}
scores: Dict[str, int] = {"Alice": 95, "Bob": 87}

def get_score(scores: Dict[str, int], name: str) -> int:
    return scores.get(name, 0)

# Sets
tags: Set[str] = {"python", "typing", "mypy"}

def unique_items(items: List[str]) -> Set[str]:
    return set(items)

# Tuples
point: Tuple[int, int] = (10, 20)
rgb: Tuple[int, int, int] = (255, 0, 0)

def get_coordinates() -> Tuple[float, float]:
    return (40.7128, -74.0060)

# Modern syntax (Python 3.9+)
names: list[str] = ["Alice", "Bob"]
scores: dict[str, int] = {"Alice": 95}
point: tuple[int, int] = (10, 20)
```

### Optional and Union

```python
from typing import Optional, Union

# Optional: value or None
def find_user(user_id: int) -> Optional[str]:
    if user_id == 1:
        return "Alice"
    return None

# Equivalent to Union[str, None]
def get_name() -> str | None:  # Python 3.10+
    return None

# Union: multiple possible types
def process_input(value: Union[int, str]) -> str:
    if isinstance(value, int):
        return str(value)
    return value

# Modern syntax (Python 3.10+)
def process_input(value: int | str) -> str:
    if isinstance(value, int):
        return str(value)
    return value
```

### Any and Type Variables

```python
from typing import Any, TypeVar

# Any: any type (opts out of type checking)
def log(message: Any) -> None:
    print(message)

log("string")  # OK
log(42)  # OK
log([1, 2, 3])  # OK

# TypeVar: generic type variable
T = TypeVar('T')

def first(items: List[T]) -> T:
    return items[0]

# Type checker infers return type from input
result: int = first([1, 2, 3])  # result is int
name: str = first(["Alice", "Bob"])  # name is str
```

## Mypy Configuration

### Basic Configuration

```toml
# pyproject.toml
[tool.mypy]
python_version = "3.11"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = false  # Start lenient
ignore_missing_imports = true

# Paths to check
files = ["src", "tests"]
exclude = [
    "migrations/",
    "build/",
]
```

### Strict Mode

```toml
[tool.mypy]
# Strict settings
strict = true

# Or enable individually:
disallow_untyped_calls = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true
disallow_untyped_decorators = true
warn_redundant_casts = true
warn_unused_ignores = true
warn_return_any = true
warn_unreachable = true
strict_equality = true
```

### Per-Module Configuration

```toml
[tool.mypy]
strict = true

# Relax rules for specific modules
[[tool.mypy.overrides]]
module = "tests.*"
disallow_untyped_defs = false

[[tool.mypy.overrides]]
module = "legacy.*"
ignore_errors = true

[[tool.mypy.overrides]]
module = "third_party.*"
ignore_missing_imports = true
```

### Ignoring Errors

```python
# Ignore single line
result = some_untyped_function()  # type: ignore

# Ignore specific error
value: int = "string"  # type: ignore[assignment]

# Ignore entire file (at top)
# mypy: ignore-errors

# Ignore specific rule for file
# mypy: disable-error-code="name-defined"
```

## Advanced Type Hints

### Generics

```python
from typing import TypeVar, Generic, List

T = TypeVar('T')

class Stack(Generic[T]):
    def __init__(self) -> None:
        self.items: List[T] = []

    def push(self, item: T) -> None:
        self.items.append(item)

    def pop(self) -> T:
        return self.items.pop()

# Usage
int_stack: Stack[int] = Stack()
int_stack.push(1)
int_stack.push(2)
value: int = int_stack.pop()

str_stack: Stack[str] = Stack()
str_stack.push("hello")
```

### Callable

```python
from typing import Callable

# Function that takes a function
def apply_twice(func: Callable[[int], int], value: int) -> int:
    return func(func(value))

def double(x: int) -> int:
    return x * 2

result = apply_twice(double, 5)  # Returns 20

# More complex callable
def create_processor(
    validator: Callable[[str], bool],
    transformer: Callable[[str], str]
) -> Callable[[str], str | None]:
    def process(text: str) -> str | None:
        if validator(text):
            return transformer(text)
        return None
    return process
```

### Literal and Final

```python
from typing import Literal, Final

# Literal: specific values only
def set_color(color: Literal["red", "green", "blue"]) -> None:
    pass

set_color("red")  # OK
set_color("yellow")  # Error: Argument must be "red", "green", or "blue"

# Final: constant value
MAX_SIZE: Final = 100
MAX_SIZE = 200  # Error: Cannot assign to final name

class Config:
    DEBUG: Final[bool] = True
```

### NewType

```python
from typing import NewType

# Create distinct types
UserId = NewType('UserId', int)
ProductId = NewType('ProductId', int)

def get_user(user_id: UserId) -> str:
    return f"User {user_id}"

def get_product(product_id: ProductId) -> str:
    return f"Product {product_id}"

# Must explicitly create
user_id = UserId(42)
product_id = ProductId(42)

get_user(user_id)  # OK
get_user(product_id)  # Error: wrong type
get_user(42)  # Error: expects UserId, not int
```

### TypedDict

```python
from typing import TypedDict

# Define dictionary structure
class User(TypedDict):
    id: int
    name: str
    email: str
    active: bool

def create_user(name: str, email: str) -> User:
    return {
        "id": 1,
        "name": name,
        "email": email,
        "active": True
    }

user: User = create_user("Alice", "alice@example.com")

# With optional fields
class UserOptional(TypedDict, total=False):
    id: int
    name: str  # Required
    email: str  # Required
    age: int  # Optional
```

## Gradual Typing

### Starting with Untyped Code

```python
# Start: No type hints
def process_data(data):
    result = []
    for item in data:
        result.append(item * 2)
    return result

# Step 1: Add basic types
def process_data(data: list) -> list:
    result = []
    for item in data:
        result.append(item * 2)
    return result

# Step 2: Add generic types
from typing import List

def process_data(data: List[int]) -> List[int]:
    result: List[int] = []
    for item in data:
        result.append(item * 2)
    return result

# Step 3: More precise
def process_data(data: List[int]) -> List[int]:
    return [item * 2 for item in data]
```

### Using Any for Migration

```python
from typing import Any, Dict

# Start with Any for complex types
def legacy_function(config: Dict[str, Any]) -> Any:
    # Complex logic
    return config.get("result")

# Gradually refine
class Config(TypedDict):
    host: str
    port: int
    debug: bool

def better_function(config: Config) -> str | None:
    return config.get("result")
```

### Type Stubs

```python
# For third-party libraries without types
# Create stubs in stubs/ directory

# stubs/external_lib.pyi
def process(data: str) -> int: ...

class Processor:
    def __init__(self, name: str) -> None: ...
    def run(self) -> bool: ...

# Configure in mypy.ini
# [mypy]
# mypy_path = stubs
```

## Protocols and Structural Typing

### Protocol Basics

```python
from typing import Protocol

# Define protocol (interface)
class Drawable(Protocol):
    def draw(self) -> None: ...

class Circle:
    def draw(self) -> None:
        print("Drawing circle")

class Square:
    def draw(self) -> None:
        print("Drawing square")

# Any object with draw() method works
def render(obj: Drawable) -> None:
    obj.draw()

render(Circle())  # OK
render(Square())  # OK
```

### Runtime Checkable Protocols

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Sized(Protocol):
    def __len__(self) -> int: ...

def process(obj: Sized) -> int:
    return len(obj)

# Can check at runtime
assert isinstance([1, 2, 3], Sized)  # True
assert isinstance(5, Sized)  # False
```

### Generic Protocols

```python
from typing import Protocol, TypeVar

T = TypeVar('T')

class Comparable(Protocol):
    def __lt__(self: T, other: T) -> bool: ...
    def __gt__(self: T, other: T) -> bool: ...

def find_max(items: List[Comparable]) -> Comparable:
    return max(items)

# Works with any comparable type
find_max([1, 2, 3])  # OK
find_max(["a", "b", "c"])  # OK
```

### Protocol Examples

```python
from typing import Protocol, Iterator

class SupportsRead(Protocol):
    def read(self, size: int = -1) -> str: ...

def process_file(file: SupportsRead) -> str:
    return file.read()

# Works with any object that has read()
with open("file.txt") as f:
    process_file(f)  # OK

from io import StringIO
process_file(StringIO("data"))  # OK

# Iterator protocol
class SupportsIter(Protocol[T]):
    def __iter__(self) -> Iterator[T]: ...

def sum_all(items: SupportsIter[int]) -> int:
    return sum(items)
```

## Summary

Type checking with Python:

**Basic types:**

- Built-in: int, str, float, bool
- Collections: List, Dict, Set, Tuple
- Optional and Union for alternatives
- Any for opting out, TypeVar for generics

**Mypy configuration:**

- Start lenient, gradually strict
- Per-module overrides
- Ignore specific errors when needed

**Advanced types:**

- Generic classes with TypeVar
- Callable for function types
- Literal for specific values
- NewType for distinct types
- TypedDict for structured dicts

**Gradual typing:**

- Add types incrementally
- Use Any during migration
- Create stubs for untyped libraries
- Refine types over time

**Protocols:**

- Structural subtyping (duck typing)
- Define interfaces without inheritance
- Runtime checkable
- Generic protocols

Type hints improve code quality, catch bugs early, and enhance IDE support without runtime overhead.

## Next Steps

- Learn type hints syntax in [Type Systems](../core_concepts/type-systems.md)
- Debug effectively with [Debugging Techniques](debugging-techniques.md)
- Use quality tools in [Code Quality Tools](code-quality-tools.md)
