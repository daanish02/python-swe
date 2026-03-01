# Design Patterns

## What are Design Patterns?

### Definition

Design patterns are reusable solutions to common software design problems:

```python
# Design patterns provide:
# - Proven solutions to recurring problems
# - Common vocabulary for developers
# - Best practices and tested approaches
# - Flexibility and maintainability

# Not:
# - Copy-paste code
# - One-size-fits-all solutions
# - Substitutes for good design
```

### When to Use Patterns

```python
# Use patterns when:
# - Problem fits the pattern's intent
# - Benefits outweigh complexity
# - Team understands the pattern

# Don't use patterns when:
# - Simpler solution exists
# - Over-engineering for current needs
# - Pattern doesn't fit the problem
```

## Creational Patterns

### Singleton Pattern

Ensure a class has only one instance:

```python
# Basic Singleton
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# Usage
s1 = Singleton()
s2 = Singleton()
assert s1 is s2  # Same instance

# Thread-safe Singleton
import threading

class ThreadSafeSingleton:
    _instance = None
    _lock = threading.Lock()

    def __new__(cls):
        if cls._instance is None:
            with cls._lock:
                if cls._instance is None:
                    cls._instance = super().__new__(cls)
        return cls._instance

# Singleton with decorator
def singleton(cls):
    instances = {}

    def get_instance(*args, **kwargs):
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return get_instance

@singleton
class DatabaseConnection:
    def __init__(self):
        self.connection = "Connected"

# Pythonic Singleton (module-level)
# database.py
class _Database:
    def __init__(self):
        self.connection = "Connected"

    def query(self, sql):
        return f"Executing: {sql}"

database = _Database()  # Single instance

# Usage: from database import database
```

### Factory Pattern

Create objects without specifying exact class:

```python
# Simple Factory
class Animal:
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
    def create_animal(animal_type):
        if animal_type == "dog":
            return Dog()
        elif animal_type == "cat":
            return Cat()
        else:
            raise ValueError(f"Unknown animal type: {animal_type}")

# Usage
factory = AnimalFactory()
dog = factory.create_animal("dog")
print(dog.speak())  # Woof!

# Factory Method Pattern
from abc import ABC, abstractmethod

class Document(ABC):
    @abstractmethod
    def render(self):
        pass

class PDFDocument(Document):
    def render(self):
        return "Rendering PDF"

class WordDocument(Document):
    def render(self):
        return "Rendering Word document"

class DocumentCreator(ABC):
    @abstractmethod
    def create_document(self):
        pass

    def process(self):
        doc = self.create_document()
        return doc.render()

class PDFCreator(DocumentCreator):
    def create_document(self):
        return PDFDocument()

class WordCreator(DocumentCreator):
    def create_document(self):
        return WordDocument()

# Usage
creator = PDFCreator()
print(creator.process())  # Rendering PDF
```

### Builder Pattern

Construct complex objects step by step:

```python
# Builder Pattern
class Pizza:
    def __init__(self):
        self.size = None
        self.cheese = False
        self.pepperoni = False
        self.mushrooms = False

    def __str__(self):
        toppings = []
        if self.cheese:
            toppings.append("cheese")
        if self.pepperoni:
            toppings.append("pepperoni")
        if self.mushrooms:
            toppings.append("mushrooms")

        return f"{self.size} pizza with {', '.join(toppings)}"

class PizzaBuilder:
    def __init__(self):
        self.pizza = Pizza()

    def set_size(self, size):
        self.pizza.size = size
        return self

    def add_cheese(self):
        self.pizza.cheese = True
        return self

    def add_pepperoni(self):
        self.pizza.pepperoni = True
        return self

    def add_mushrooms(self):
        self.pizza.mushrooms = True
        return self

    def build(self):
        return self.pizza

# Usage
pizza = (PizzaBuilder()
         .set_size("large")
         .add_cheese()
         .add_pepperoni()
         .build())

print(pizza)  # large pizza with cheese, pepperoni

# Pythonic Builder (using kwargs)
class PizzaSimple:
    def __init__(self, size, cheese=False, pepperoni=False, mushrooms=False):
        self.size = size
        self.cheese = cheese
        self.pepperoni = pepperoni
        self.mushrooms = mushrooms

pizza = PizzaSimple(size="large", cheese=True, pepperoni=True)
```

## Structural Patterns

### Adapter Pattern

Convert interface of a class to another interface:

```python
# Target interface
class USPlug:
    def provide_power(self):
        return "110V"

# Incompatible interface
class EuropeanSocket:
    def get_power(self):
        return "220V"

# Adapter
class PlugAdapter:
    def __init__(self, european_socket):
        self.socket = european_socket

    def provide_power(self):
        power = self.socket.get_power()
        # Convert 220V to 110V
        return f"{power} converted to 110V"

# Usage
european_socket = EuropeanSocket()
adapter = PlugAdapter(european_socket)
print(adapter.provide_power())  # 220V converted to 110V

# Real-world example
class LegacyAPI:
    def old_request(self, data):
        return f"Old API: {data}"

class ModernAPI:
    def request(self, data):
        pass

class APIAdapter(ModernAPI):
    def __init__(self, legacy_api):
        self.legacy_api = legacy_api

    def request(self, data):
        # Adapt new interface to old
        return self.legacy_api.old_request(data)

legacy = LegacyAPI()
adapter = APIAdapter(legacy)
print(adapter.request("data"))  # Old API: data
```

### Decorator Pattern

Add behavior to objects dynamically:

```python
# Component
class Coffee:
    def cost(self):
        return 5

    def description(self):
        return "Coffee"

# Decorator
class CoffeeDecorator:
    def __init__(self, coffee):
        self._coffee = coffee

    def cost(self):
        return self._coffee.cost()

    def description(self):
        return self._coffee.description()

# Concrete decorators
class MilkDecorator(CoffeeDecorator):
    def cost(self):
        return self._coffee.cost() + 1

    def description(self):
        return self._coffee.description() + ", Milk"

class SugarDecorator(CoffeeDecorator):
    def cost(self):
        return self._coffee.cost() + 0.5

    def description(self):
        return self._coffee.description() + ", Sugar"

# Usage
coffee = Coffee()
print(f"{coffee.description()}: ${coffee.cost()}")
# Coffee: $5

coffee = MilkDecorator(coffee)
print(f"{coffee.description()}: ${coffee.cost()}")
# Coffee, Milk: $6

coffee = SugarDecorator(coffee)
print(f"{coffee.description()}: ${coffee.cost()}")
# Coffee, Milk, Sugar: $6.5

# Python decorator (function)
def logging_decorator(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@logging_decorator
def add(a, b):
    return a + b

add(2, 3)  # Logs before and after
```

### Facade Pattern

Provide simplified interface to complex subsystem:

```python
# Complex subsystem
class CPU:
    def freeze(self):
        print("CPU: Freezing")

    def execute(self):
        print("CPU: Executing")

class Memory:
    def load(self, position, data):
        print(f"Memory: Loading {data} at {position}")

class HardDrive:
    def read(self, sector, size):
        return f"Data from sector {sector}"

# Facade
class ComputerFacade:
    def __init__(self):
        self.cpu = CPU()
        self.memory = Memory()
        self.hard_drive = HardDrive()

    def start(self):
        """Simplified interface for starting computer."""
        self.cpu.freeze()
        data = self.hard_drive.read(0, 1024)
        self.memory.load(0, data)
        self.cpu.execute()

# Usage
computer = ComputerFacade()
computer.start()  # Simple interface hides complexity

# Real-world example
class DatabaseFacade:
    def __init__(self):
        self.connection = DatabaseConnection()
        self.query_builder = QueryBuilder()
        self.result_parser = ResultParser()

    def get_user(self, user_id):
        """Simple method hides complex query logic."""
        query = self.query_builder.select('users').where(id=user_id)
        result = self.connection.execute(query)
        return self.result_parser.parse(result)
```

## Behavioral Patterns

### Strategy Pattern

Define family of algorithms, make them interchangeable:

```python
# Strategy interface
from abc import ABC, abstractmethod

class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data):
        pass

# Concrete strategies
class BubbleSort(SortStrategy):
    def sort(self, data):
        data = data.copy()
        n = len(data)
        for i in range(n):
            for j in range(0, n-i-1):
                if data[j] > data[j+1]:
                    data[j], data[j+1] = data[j+1], data[j]
        return data

class QuickSort(SortStrategy):
    def sort(self, data):
        if len(data) <= 1:
            return data
        pivot = data[len(data) // 2]
        left = [x for x in data if x < pivot]
        middle = [x for x in data if x == pivot]
        right = [x for x in data if x > pivot]
        return self.sort(left) + middle + self.sort(right)

# Context
class Sorter:
    def __init__(self, strategy: SortStrategy):
        self.strategy = strategy

    def sort(self, data):
        return self.strategy.sort(data)

# Usage
data = [3, 1, 4, 1, 5, 9, 2, 6]

sorter = Sorter(BubbleSort())
print(sorter.sort(data))  # [1, 1, 2, 3, 4, 5, 6, 9]

sorter = Sorter(QuickSort())
print(sorter.sort(data))  # [1, 1, 2, 3, 4, 5, 6, 9]

# Pythonic approach (pass function)
def bubble_sort(data):
    # ... implementation
    pass

def quick_sort(data):
    # ... implementation
    pass

class SimpleSorter:
    def __init__(self, sort_func):
        self.sort_func = sort_func

    def sort(self, data):
        return self.sort_func(data)

sorter = SimpleSorter(quick_sort)
```

### Observer Pattern

Define one-to-many dependency, notify all dependents of state changes:

```python
# Subject
class Subject:
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def detach(self, observer):
        self._observers.remove(observer)

    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

# Observer interface
class Observer(ABC):
    @abstractmethod
    def update(self, message):
        pass

# Concrete observers
class EmailNotifier(Observer):
    def update(self, message):
        print(f"Email: {message}")

class SMSNotifier(Observer):
    def update(self, message):
        print(f"SMS: {message}")

class LogNotifier(Observer):
    def update(self, message):
        print(f"Log: {message}")

# Usage
subject = Subject()

email = EmailNotifier()
sms = SMSNotifier()
log = LogNotifier()

subject.attach(email)
subject.attach(sms)
subject.attach(log)

subject.notify("New event occurred!")
# Email: New event occurred!
# SMS: New event occurred!
# Log: New event occurred!

# Real-world example: Stock price monitoring
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
        self.notify(f"{self.symbol} price changed to ${value}")

class StockDisplay(Observer):
    def __init__(self, name):
        self.name = name

    def update(self, message):
        print(f"{self.name}: {message}")

stock = Stock("AAPL", 150)
display1 = StockDisplay("Display 1")
display2 = StockDisplay("Display 2")

stock.attach(display1)
stock.attach(display2)

stock.price = 155
# Display 1: AAPL price changed to $155
# Display 2: AAPL price changed to $155
```

### Command Pattern

Encapsulate request as object:

```python
# Command interface
class Command(ABC):
    @abstractmethod
    def execute(self):
        pass

    @abstractmethod
    def undo(self):
        pass

# Receiver
class Light:
    def on(self):
        print("Light is ON")

    def off(self):
        print("Light is OFF")

# Concrete commands
class LightOnCommand(Command):
    def __init__(self, light):
        self.light = light

    def execute(self):
        self.light.on()

    def undo(self):
        self.light.off()

class LightOffCommand(Command):
    def __init__(self, light):
        self.light = light

    def execute(self):
        self.light.off()

    def undo(self):
        self.light.on()

# Invoker
class RemoteControl:
    def __init__(self):
        self.history = []

    def execute_command(self, command):
        command.execute()
        self.history.append(command)

    def undo_last(self):
        if self.history:
            command = self.history.pop()
            command.undo()

# Usage
light = Light()
remote = RemoteControl()

on_command = LightOnCommand(light)
off_command = LightOffCommand(light)

remote.execute_command(on_command)   # Light is ON
remote.execute_command(off_command)  # Light is OFF
remote.undo_last()                   # Light is ON (undo off)
```

## Python-Specific Patterns

### Context Manager Pattern

```python
# Using class
class FileManager:
    def __init__(self, filename, mode):
        self.filename = filename
        self.mode = mode
        self.file = None

    def __enter__(self):
        self.file = open(self.filename, self.mode)
        return self.file

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()

# Usage
with FileManager('file.txt', 'w') as f:
    f.write('Hello')

# Using contextlib
from contextlib import contextmanager

@contextmanager
def file_manager(filename, mode):
    f = open(filename, mode)
    try:
        yield f
    finally:
        f.close()

with file_manager('file.txt', 'w') as f:
    f.write('Hello')
```

### Borg Pattern (Monostate)

Share state across all instances:

```python
class Borg:
    _shared_state = {}

    def __init__(self):
        self.__dict__ = self._shared_state

class Config(Borg):
    def __init__(self):
        super().__init__()

    def set_config(self, key, value):
        self.__dict__[key] = value

# Usage
config1 = Config()
config1.set_config('debug', True)

config2 = Config()
print(config2.debug)  # True (shared state)
```

### Mixin Pattern

```python
# Mixins add functionality to classes
class JSONMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__)

class XMLMixin:
    def to_xml(self):
        # ... XML serialization
        pass

class User(JSONMixin, XMLMixin):
    def __init__(self, name, email):
        self.name = name
        self.email = email

user = User("Alice", "alice@example.com")
print(user.to_json())  # {"name": "Alice", "email": "alice@example.com"}
```

## Summary

Design patterns in Python:

**Creational:**

- **Singleton:** Single instance
- **Factory:** Create objects without specifying class
- **Builder:** Construct complex objects step by step

**Structural:**

- **Adapter:** Convert interface
- **Decorator:** Add behavior dynamically
- **Facade:** Simplified interface to complex system

**Behavioral:**

- **Strategy:** Interchangeable algorithms
- **Observer:** Notify dependents of changes
- **Command:** Encapsulate requests as objects

**Python-specific:**

- Context managers
- Borg (monostate)
- Mixins

Use patterns to solve recurring design problems, but don't over-engineer.

## Next Steps

- Apply [SOLID Principles](solid-principles.md)
- Follow [Code Organization](code-organization.md)
- Improve with [Refactoring](refactoring.md)
