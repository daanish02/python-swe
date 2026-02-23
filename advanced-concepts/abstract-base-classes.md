# Abstract Base Classes

## Table of Contents

- [What are Abstract Base Classes?](#what-are-abstract-base-classes)
- [The abc Module](#the-abc-module)
- [Creating ABCs](#creating-abcs)
- [Abstract Methods](#abstract-methods)
- [Virtual Subclasses](#virtual-subclasses)
- [ABCs in the Standard Library](#abcs-in-the-standard-library)
- [Design Patterns with ABCs](#design-patterns-with-abcs)
- [Summary](#summary)
- [Next Steps](#next-steps)

## What are Abstract Base Classes?

### Definition

Abstract Base Classes (ABCs) define interfaces in Python. They cannot be instantiated directly and require subclasses to implement specific methods.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        """Calculate area."""
        pass

    @abstractmethod
    def perimeter(self):
        """Calculate perimeter."""
        pass

# shape = Shape()  # TypeError: Can't instantiate abstract class

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14159 * self.radius ** 2

    def perimeter(self):
        return 2 * 3.14159 * self.radius

circle = Circle(5)  # OK - all abstract methods implemented
```

### Why Use ABCs?

```python
# Without ABCs - no enforcement
class Animal:
    def make_sound(self):
        raise NotImplementedError("Subclasses must implement")

class Dog(Animal):
    pass  # No error until runtime

# dog = Dog()
# dog.make_sound()  # NotImplementedError

# With ABCs - error at instantiation
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        pass

class Dog(Animal):
    pass

# dog = Dog()  # TypeError at instantiation time
```

## The abc Module

### ABC Base Class

```python
from abc import ABC, abstractmethod

# Method 1: Inherit from ABC
class MyABC(ABC):
    @abstractmethod
    def my_method(self):
        pass

# Method 2: Use ABCMeta metaclass
from abc import ABCMeta

class MyABC(metaclass=ABCMeta):
    @abstractmethod
    def my_method(self):
        pass
```

### abstractmethod Decorator

```python
from abc import ABC, abstractmethod

class DataProcessor(ABC):
    @abstractmethod
    def process(self, data):
        """Must be implemented by subclasses."""
        pass

class CSVProcessor(DataProcessor):
    def process(self, data):
        return f"Processing CSV: {data}"

class JSONProcessor(DataProcessor):
    def process(self, data):
        return f"Processing JSON: {data}"

processors = [CSVProcessor(), JSONProcessor()]
for proc in processors:
    print(proc.process("sample data"))
```

## Creating ABCs

### Basic ABC

```python
from abc import ABC, abstractmethod

class Storage(ABC):
    """Abstract storage interface."""

    @abstractmethod
    def save(self, key, value):
        """Save value with key."""
        pass

    @abstractmethod
    def load(self, key):
        """Load value by key."""
        pass

    @abstractmethod
    def delete(self, key):
        """Delete value by key."""
        pass

class DictStorage(Storage):
    def __init__(self):
        self._data = {}

    def save(self, key, value):
        self._data[key] = value

    def load(self, key):
        return self._data.get(key)

    def delete(self, key):
        self._data.pop(key, None)

storage = DictStorage()
storage.save("key1", "value1")
print(storage.load("key1"))  # value1
```

### ABC with Concrete Methods

ABCs can provide concrete methods:

```python
from abc import ABC, abstractmethod

class Vehicle(ABC):
    def __init__(self, brand):
        self.brand = brand

    @abstractmethod
    def start_engine(self):
        """Abstract - must be implemented."""
        pass

    def display_info(self):
        """Concrete - inherited by all subclasses."""
        return f"{self.brand} {self.__class__.__name__}"

class Car(Vehicle):
    def start_engine(self):
        return f"{self.brand} car engine started"

car = Car("Toyota")
print(car.start_engine())  # Toyota car engine started
print(car.display_info())  # Toyota Car
```

### ABC with Properties

```python
from abc import ABC, abstractmethod

class Employee(ABC):
    @property
    @abstractmethod
    def salary(self):
        """Must be implemented as a property."""
        pass

    @abstractmethod
    def calculate_bonus(self):
        """Must be implemented as a method."""
        pass

class FullTimeEmployee(Employee):
    def __init__(self, base_salary):
        self._base_salary = base_salary

    @property
    def salary(self):
        return self._base_salary

    def calculate_bonus(self):
        return self.salary * 0.1

emp = FullTimeEmployee(50000)
print(emp.salary)  # 50000
print(emp.calculate_bonus())  # 5000.0
```

## Abstract Methods

### Class Methods and Static Methods

```python
from abc import ABC, abstractmethod

class Database(ABC):
    @classmethod
    @abstractmethod
    def connect(cls, connection_string):
        """Create connection."""
        pass

    @staticmethod
    @abstractmethod
    def validate_query(query):
        """Validate SQL query."""
        pass

    @abstractmethod
    def execute(self, query):
        """Execute query."""
        pass

class PostgreSQL(Database):
    @classmethod
    def connect(cls, connection_string):
        return cls(connection_string)

    @staticmethod
    def validate_query(query):
        return len(query) > 0

    def __init__(self, connection_string):
        self.connection_string = connection_string

    def execute(self, query):
        return f"Executing on PostgreSQL: {query}"

db = PostgreSQL.connect("postgresql://localhost")
print(PostgreSQL.validate_query("SELECT * FROM users"))  # True
print(db.execute("SELECT 1"))  # Executing on PostgreSQL: SELECT 1
```

### Multiple Abstract Methods

```python
from abc import ABC, abstractmethod

class Repository(ABC):
    @abstractmethod
    def create(self, entity):
        pass

    @abstractmethod
    def read(self, entity_id):
        pass

    @abstractmethod
    def update(self, entity_id, data):
        pass

    @abstractmethod
    def delete(self, entity_id):
        pass

    def exists(self, entity_id):
        """Concrete helper method."""
        return self.read(entity_id) is not None

class UserRepository(Repository):
    def __init__(self):
        self.users = {}
        self.next_id = 1

    def create(self, entity):
        user_id = self.next_id
        self.next_id += 1
        self.users[user_id] = entity
        return user_id

    def read(self, entity_id):
        return self.users.get(entity_id)

    def update(self, entity_id, data):
        if entity_id in self.users:
            self.users[entity_id].update(data)

    def delete(self, entity_id):
        self.users.pop(entity_id, None)

repo = UserRepository()
user_id = repo.create({"name": "Alice", "email": "alice@example.com"})
print(repo.exists(user_id))  # True
```

## Virtual Subclasses

### Register Method

Make a class a "virtual subclass" without inheritance:

```python
from abc import ABC

class Sized(ABC):
    @abstractmethod
    def __len__(self):
        pass

# Register as virtual subclass
@Sized.register
class MyList:
    def __init__(self):
        self._items = []

    def __len__(self):
        return len(self._items)

# Check isinstance
obj = MyList()
print(isinstance(obj, Sized))  # True
print(issubclass(MyList, Sized))  # True

# But not in __mro__
print(MyList.__mro__)  # (<class '__main__.MyList'>, <class 'object'>)
```

### Checking Subclasses

```python
from abc import ABC, abstractmethod

class Drawable(ABC):
    @abstractmethod
    def draw(self):
        pass

class Circle(Drawable):
    def draw(self):
        return "Drawing circle"

@Drawable.register
class Square:
    def draw(self):
        return "Drawing square"

# Both are subclasses
print(issubclass(Circle, Drawable))  # True
print(issubclass(Square, Drawable))  # True

# Get all registered subclasses
print(Drawable.__subclasses__())  # [<class '__main__.Circle'>]
```

## ABCs in the Standard Library

### collections.abc Module

Common ABCs for collections:

```python
from collections.abc import Iterable, Sized, Container

class MyCollection:
    def __init__(self, items):
        self._items = items

    def __iter__(self):
        return iter(self._items)

    def __len__(self):
        return len(self._items)

    def __contains__(self, item):
        return item in self._items

col = MyCollection([1, 2, 3, 4, 5])
print(isinstance(col, Iterable))  # True
print(isinstance(col, Sized))  # True
print(isinstance(col, Container))  # True
```

### Sequence ABC

```python
from collections.abc import Sequence

class MySequence(Sequence):
    def __init__(self, items):
        self._items = items

    def __getitem__(self, index):
        return self._items[index]

    def __len__(self):
        return len(self._items)

seq = MySequence([1, 2, 3, 4, 5])
print(seq[2])  # 3
print(len(seq))  # 5
print(3 in seq)  # True (provided by Sequence)
print(list(reversed(seq)))  # [5, 4, 3, 2, 1] (provided by Sequence)
```

### Mapping ABC

```python
from collections.abc import MutableMapping

class CaseInsensitiveDict(MutableMapping):
    def __init__(self):
        self._data = {}

    def __getitem__(self, key):
        return self._data[key.lower()]

    def __setitem__(self, key, value):
        self._data[key.lower()] = value

    def __delitem__(self, key):
        del self._data[key.lower()]

    def __iter__(self):
        return iter(self._data)

    def __len__(self):
        return len(self._data)

d = CaseInsensitiveDict()
d['Name'] = 'Alice'
print(d['name'])  # Alice (case insensitive)
print(d['NAME'])  # Alice
```

## Design Patterns with ABCs

### Strategy Pattern

```python
from abc import ABC, abstractmethod

class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data):
        pass

class QuickSort(SortStrategy):
    def sort(self, data):
        if len(data) <= 1:
            return data
        pivot = data[len(data) // 2]
        left = [x for x in data if x < pivot]
        middle = [x for x in data if x == pivot]
        right = [x for x in data if x > pivot]
        return self.sort(left) + middle + self.sort(right)

class BubbleSort(SortStrategy):
    def sort(self, data):
        data = data.copy()
        n = len(data)
        for i in range(n):
            for j in range(0, n-i-1):
                if data[j] > data[j+1]:
                    data[j], data[j+1] = data[j+1], data[j]
        return data

class Sorter:
    def __init__(self, strategy: SortStrategy):
        self.strategy = strategy

    def sort(self, data):
        return self.strategy.sort(data)

data = [3, 1, 4, 1, 5, 9, 2, 6]
sorter = Sorter(QuickSort())
print(sorter.sort(data))  # [1, 1, 2, 3, 4, 5, 6, 9]

sorter = Sorter(BubbleSort())
print(sorter.sort(data))  # [1, 1, 2, 3, 4, 5, 6, 9]
```

### Template Method Pattern

```python
from abc import ABC, abstractmethod

class DataParser(ABC):
    def parse_file(self, filename):
        """Template method."""
        data = self.read_file(filename)
        parsed = self.parse_data(data)
        self.validate(parsed)
        return self.transform(parsed)

    def read_file(self, filename):
        """Concrete method."""
        with open(filename) as f:
            return f.read()

    @abstractmethod
    def parse_data(self, data):
        """Must be implemented."""
        pass

    def validate(self, parsed_data):
        """Hook method - can be overridden."""
        pass

    def transform(self, parsed_data):
        """Hook method - can be overridden."""
        return parsed_data

class CSVParser(DataParser):
    def parse_data(self, data):
        lines = data.strip().split('\n')
        return [line.split(',') for line in lines]

    def validate(self, parsed_data):
        for row in parsed_data:
            if not row:
                raise ValueError("Empty row found")

class JSONParser(DataParser):
    def parse_data(self, data):
        import json
        return json.loads(data)

    def validate(self, parsed_data):
        if not isinstance(parsed_data, (dict, list)):
            raise ValueError("Invalid JSON structure")
```

### Factory Pattern

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self):
        pass

class Dog(Animal):
    def speak(self):
        return "Woof!"

class Cat(Animal):
    def speak(self):
        return "Meow!"

class AnimalFactory:
    @staticmethod
    def create_animal(animal_type: str) -> Animal:
        if animal_type == "dog":
            return Dog()
        elif animal_type == "cat":
            return Cat()
        else:
            raise ValueError(f"Unknown animal type: {animal_type}")

dog = AnimalFactory.create_animal("dog")
print(dog.speak())  # Woof!

cat = AnimalFactory.create_animal("cat")
print(cat.speak())  # Meow!
```

### Observer Pattern

```python
from abc import ABC, abstractmethod
from typing import List

class Observer(ABC):
    @abstractmethod
    def update(self, subject):
        pass

class Subject(ABC):
    def __init__(self):
        self._observers: List[Observer] = []

    def attach(self, observer: Observer):
        self._observers.append(observer)

    def detach(self, observer: Observer):
        self._observers.remove(observer)

    def notify(self):
        for observer in self._observers:
            observer.update(self)

class Stock(Subject):
    def __init__(self, symbol, price):
        super().__init__()
        self.symbol = symbol
        self._price = price

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        self._price = value
        self.notify()

class StockDisplay(Observer):
    def __init__(self, name):
        self.name = name

    def update(self, subject: Stock):
        print(f"{self.name}: {subject.symbol} is now ${subject.price}")

stock = Stock("AAPL", 150)
display1 = StockDisplay("Display 1")
display2 = StockDisplay("Display 2")

stock.attach(display1)
stock.attach(display2)

stock.price = 155  # Both displays notified
# Display 1: AAPL is now $155
# Display 2: AAPL is now $155
```

## Summary

Abstract Base Classes (ABCs) provide:

**Key concepts:**

- Cannot be instantiated directly
- Define interfaces through abstract methods
- Enforce implementation in subclasses
- Use `abc.ABC` or `abc.ABCMeta`

**Abstract methods:**

- `@abstractmethod`: Must be implemented
- Works with methods, properties, class methods, static methods
- Error at instantiation if not implemented

**Virtual subclasses:**

- Register classes without inheritance
- Use `@ABC.register` decorator
- Satisfy `isinstance()` and `issubclass()`

**Standard library ABCs:**

- `collections.abc`: Iterable, Sized, Sequence, Mapping, etc.
- Provide mixin methods
- Enable duck typing with isinstance checks

**Design patterns:**

- Strategy pattern
- Template method pattern
- Factory pattern
- Observer pattern

ABCs are crucial for defining clear interfaces and ensuring consistent implementations across related classes.

## Next Steps

- Combine with [Descriptors](descriptors-and-properties.md)
- Apply in [Object-Oriented Programming](object-oriented-programming.md)
- Use with [Type Systems](type-systems.md) protocols
