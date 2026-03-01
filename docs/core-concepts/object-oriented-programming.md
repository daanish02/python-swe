# Object-Oriented Programming

## Classes and Objects

### Defining Classes

A class is a blueprint for creating objects:

```python
class Dog:
    pass

# Create instance (object)
my_dog = Dog()
print(type(my_dog))  # <class '__main__.Dog'>
```

### Basic Class Structure

```python
class Person:
    """Represents a person."""

    def __init__(self, name, age):
        """Initialize person with name and age."""
        self.name = name
        self.age = age

    def greet(self):
        """Return greeting message."""
        return f"Hello, I'm {self.name}"

# Create instances
alice = Person("Alice", 30)
bob = Person("Bob", 25)

print(alice.greet())  # Hello, I'm Alice
print(bob.greet())    # Hello, I'm Bob
```

### Understanding `self`

`self` refers to the instance being operated on:

```python
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1  # Modifies this instance's count

    def get_count(self):
        return self.count

c1 = Counter()
c2 = Counter()

c1.increment()
c1.increment()
print(c1.get_count())  # 2
print(c2.get_count())  # 0 - separate instance
```

## Methods and Attributes

### Instance Attributes

Belong to individual objects:

```python
class Car:
    def __init__(self, make, model):
        self.make = make      # Instance attribute
        self.model = model    # Instance attribute
        self.odometer = 0     # Instance attribute

car1 = Car("Toyota", "Camry")
car2 = Car("Honda", "Civic")

print(car1.make)  # Toyota
print(car2.make)  # Honda - different instance
```

### Class Attributes

Shared by all instances:

```python
class Dog:
    species = "Canis familiaris"  # Class attribute

    def __init__(self, name):
        self.name = name  # Instance attribute

dog1 = Dog("Buddy")
dog2 = Dog("Max")

print(dog1.species)  # Canis familiaris
print(dog2.species)  # Canis familiaris - same for all

# Shared across all instances
Dog.species = "Canis lupus"
print(dog1.species)  # Canis lupus
print(dog2.species)  # Canis lupus
```

### Instance Methods

Regular methods that operate on instance:

```python
class BankAccount:
    def __init__(self, balance=0):
        self.balance = balance

    def deposit(self, amount):
        """Instance method - uses self."""
        self.balance += amount
        return self.balance

    def withdraw(self, amount):
        if amount <= self.balance:
            self.balance -= amount
            return amount
        return 0

account = BankAccount(100)
account.deposit(50)   # 150
account.withdraw(30)  # 120
```

## The **init** Method

### Constructor Basics

Called when creating new instance:

```python
class Rectangle:
    def __init__(self, width, height):
        """Initialize rectangle dimensions."""
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

rect = Rectangle(5, 10)  # __init__ called
print(rect.area())  # 50
```

### Default Parameters

```python
class User:
    def __init__(self, username, email, active=True):
        self.username = username
        self.email = email
        self.active = active

user1 = User("alice", "alice@example.com")
user2 = User("bob", "bob@example.com", active=False)

print(user1.active)  # True (default)
print(user2.active)  # False
```

### Validation in **init**

```python
class Person:
    def __init__(self, name, age):
        if not name:
            raise ValueError("Name cannot be empty")
        if age < 0:
            raise ValueError("Age cannot be negative")

        self.name = name
        self.age = age

# Valid
person = Person("Alice", 30)

# Invalid
# person = Person("", 30)    # ValueError: Name cannot be empty
# person = Person("Bob", -5)  # ValueError: Age cannot be negative
```

## Instance vs Class vs Static Methods

### Instance Methods

Most common - operate on instance data:

```python
class Dog:
    def __init__(self, name):
        self.name = name

    def bark(self):  # Instance method
        return f"{self.name} says woof!"

dog = Dog("Buddy")
print(dog.bark())  # Buddy says woof!
```

### Class Methods

Operate on class, not instance. Use `@classmethod` decorator:

=== "main.py"

    ```python
    from date import Date

    # Regular constructor
    date1 = Date(2024, 1, 15)

    # Class method as alternative constructor
    date2 = Date.from_string("2024-01-15")

    # Class method as factory
    date3 = Date.today()
    ```

=== "date.py"

    ```python
    class Date:
        def __init__(self, year, month, day):
            self.year = year
            self.month = month
            self.day = day

        @classmethod
        def from_string(cls, date_string):
            """Alternative constructor."""
            year, month, day = map(int, date_string.split('-'))
            return cls(year, month, day)  # cls is Date class

        @classmethod
        def today(cls):
            """Factory method to create today's date."""
            import datetime
            today = datetime.date.today()
            return cls(today.year, today.month, today.day)
    ```

### Static Methods

Don't operate on instance or class. Use `@staticmethod`:

=== "main.py"

    ```python
    from mathutils import MathUtils

    # Call without creating instance
    result = MathUtils.add(5, 3)  # 8
    print(MathUtils.is_even(4))   # True

    # Can also call from instance (but unnecessary)
    utils = MathUtils()
    result = utils.add(2, 3)  # 5
    ```

=== "mathutils.py"

    ```python
    class MathUtils:
        @staticmethod
        def add(a, b):
            """Add two numbers - no need for instance or class."""
            return a + b

        @staticmethod
        def is_even(num):
            return num % 2 == 0
    ```

### When to Use Each

=== "main.py"

    ```python
    from pizza import Pizza

    # Instance method
    pizza = Pizza('medium', ['tomato', 'mozzarella'])
    print(pizza.get_price())  # 13.0

    # Class method
    margherita = Pizza.margherita('large')

    # Static method
    print(Pizza.validate_topping('pepperoni'))  # True
    ```

=== "pizza.py"

    ```python
    class Pizza:
        def __init__(self, size, toppings):
            self.size = size
            self.toppings = toppings

        # Instance method - operates on instance data
        def get_price(self):
            base = {'small': 8, 'medium': 10, 'large': 12}
            return base[self.size] + len(self.toppings) * 1.5

        # Class method - alternative constructor
        @classmethod
        def margherita(cls, size):
            return cls(size, ['tomato', 'mozzarella', 'basil'])

        # Static method - utility function
        @staticmethod
        def validate_topping(topping):
            valid = ['tomato', 'mozzarella', 'basil', 'pepperoni']
            return topping in valid
    ```

## Special Methods

### String Representation

=== "main.py"

    ```python
    from book import Book

    book = Book("1984", "George Orwell")
    print(book)         # 1984 by George Orwell (uses __str__)
    print(repr(book))   # Book('1984', 'George Orwell') (uses __repr__)
    print([book])       # [Book('1984', 'George Orwell')] (uses __repr__)
    ```

=== "book.py"

    ```python
    class Book:
        def __init__(self, title, author):
            self.title = title
            self.author = author

        def __str__(self):
            """User-friendly string - used by print()."""
            return f"{self.title} by {self.author}"

        def __repr__(self):
            """Developer-friendly string - used by repr()."""
            return f"Book('{self.title}', '{self.author}')"
    ```

### Comparison Operators

=== "main.py"

    ```python
    from rectangle import Rectangle

    rect1 = Rectangle(3, 4)  # area = 12
    rect2 = Rectangle(2, 6)  # area = 12
    rect3 = Rectangle(5, 5)  # area = 25

    print(rect1 == rect2)  # True
    print(rect1 < rect3)   # True
    print(rect3 > rect1)   # True
    ```

=== "rectangle.py"

    ```python
    class Rectangle:
        def __init__(self, width, height):
            self.width = width
            self.height = height

        def area(self):
            return self.width * self.height

        def __eq__(self, other):
            """Equal to =="""
            return self.area() == other.area()

        def __lt__(self, other):
            """Less than <"""
            return self.area() < other.area()

        def __le__(self, other):
            """Less than or equal <="""
            return self.area() <= other.area()
    ```

### Arithmetic Operators

=== "main.py"

    ```python
    from vector import Vector

    v1 = Vector(1, 2)
    v2 = Vector(3, 4)

    print(v1 + v2)  # Vector(4, 6)
    print(v2 - v1)  # Vector(2, 2)
    print(v1 * 3)   # Vector(3, 6)
    ```

=== "vector.py"

    ```python
    class Vector:
        def __init__(self, x, y):
            self.x = x
            self.y = y

        def __add__(self, other):
            """Addition: v1 + v2"""
            return Vector(self.x + other.x, self.y + other.y)

        def __sub__(self, other):
            """Subtraction: v1 - v2"""
            return Vector(self.x - other.x, self.y - other.y)

        def __mul__(self, scalar):
            """Multiplication by scalar: v * 3"""
            return Vector(self.x * scalar, self.y * scalar)

        def __str__(self):
            return f"Vector({self.x}, {self.y})"
    ```

### Container Methods

=== "main.py"

    ```python
    from playlist import Playlist

    playlist = Playlist()
    playlist.add_song("Song1")
    playlist.add_song("Song2")
    playlist.add_song("Song3")

    print(len(playlist))           # 3
    print(playlist[0])             # Song1
    print("Song2" in playlist)     # True
    for song in playlist:          # Iteration works
        print(song)
    ```

=== "playlist.py"

    ```python
    class Playlist:
        def __init__(self):
            self.songs = []

        def add_song(self, song):
            self.songs.append(song)

        def __len__(self):
            """len() support"""
            return len(self.songs)

        def __getitem__(self, index):
            """Indexing support: playlist[0]"""
            return self.songs[index]

        def __contains__(self, song):
            """'in' operator support"""
            return song in self.songs

        def __iter__(self):
            """Iteration support"""
            return iter(self.songs)
        print(song)
    ```

## Inheritance

### Basic Inheritance

=== "main.py"

    ```python
    from dog import Dog
    from cat import Cat

    dog = Dog("Buddy")
    cat = Cat("Whiskers")

    print(dog.speak())  # Buddy says woof!
    print(cat.speak())  # Whiskers says meow!
    ```

=== "dog.py"

    ```python
    from animal import Animal

    class Dog(Animal):  # Dog inherits from Animal
        def speak(self):  # Override parent method
            return f"{self.name} says woof!"
    ```

=== "cat.py"

    ```python
    from animal import Animal

    class Cat(Animal):
        def speak(self):
            return f"{self.name} says meow!"
    ```

=== "animal.py"

    ```python
    class Animal:
        def __init__(self, name):
            self.name = name

        def speak(self):
            return "Some sound"
    ```

### Calling Parent Methods

=== "main.py"

    ```python
    from car import Car

    car = Car("Toyota", "Camry", 4)
    print(car.info())  # Toyota Camry (4 doors)
    ```

=== "car.py"

    ```python
    from vehicle import Vehicle

    class Car(Vehicle):
        def __init__(self, make, model, num_doors):
            super().__init__(make, model)  # Call parent __init__
            self.num_doors = num_doors

        def info(self):
            base_info = super().info()  # Call parent method
            return f"{base_info} ({self.num_doors} doors)"
    ```

=== "vehicle.py"

    ```python
    class Vehicle:
        def __init__(self, make, model):
            self.make = make
            self.model = model

        def info(self):
            return f"{self.make} {self.model}"
    ```

### Multiple Inheritance

```python
class Flyable:
    def fly(self):
        return "Flying"

class Swimmable:
    def swim(self):
        return "Swimming"

class Duck(Flyable, Swimmable):  # Inherits from both
    def quack(self):
        return "Quack!"

duck = Duck()
print(duck.fly())    # Flying
print(duck.swim())   # Swimming
print(duck.quack())  # Quack!
```

## Composition

### Has-A Relationship

Prefer composition over inheritance when appropriate:

=== "main.py"

    ```python
    from car import Car

    car = Car("Toyota", "Camry", 200)
    print(car.start())  # Toyota Camry: Engine started
    ```

=== "car.py"

    ```python
    from engine import Engine

    class Car:
        def __init__(self, make, model, horsepower):
            self.make = make
            self.model = model
            self.engine = Engine(horsepower)  # Car HAS-AN Engine

        def start(self):
            return f"{self.make} {self.model}: {self.engine.start()}"
    ```

=== "engine.py"

    ```python
    class Engine:
        def __init__(self, horsepower):
            self.horsepower = horsepower

        def start(self):
            return "Engine started"
    ```

### Composition vs Inheritance

=== "Inheritance"

    ```python
    # IS-A: Rectangle IS-A Shape
    class Shape:
        pass

    class Rectangle(Shape):
        pass
    ```

=== "Composition"

    ```python
    # HAS-A: Car HAS-A Engine
    class Engine:
        pass

    class Car:
        def __init__(self):
            self.engine = Engine()
    ```

**When to use:**

- **Inheritance:** "is-a" relationship, extending behavior
- **Composition:** "has-a" relationship, building with components

## Polymorphism

### Method Overriding

Different classes with same interface:

=== "main.py"

    ```python
    from circle import Circle
    from rectangle import Rectangle

    # Polymorphism - same interface, different implementations
    shapes = [Circle(5), Rectangle(3, 4), Circle(3)]

    for shape in shapes:
        print(f"Area: {shape.area()}")

    # Area: 78.53975
    # Area: 12
    # Area: 28.27431
    ```

=== "circle.py"

    ```python
    from shape import Shape

    class Circle(Shape):
        def __init__(self, radius):
            self.radius = radius

        def area(self):
            return 3.14159 * self.radius ** 2
    ```

=== "rectangle.py"

    ```python
    from shape import Shape

    class Rectangle(Shape):
        def __init__(self, width, height):
            self.width = width
            self.height = height

        def area(self):
            return self.width * self.height
    ```

=== "shape.py"

    ```python
    class Shape:
        def area(self):
            raise NotImplementedError("Subclass must implement")
    ```

### Duck Typing

```python
class Dog:
    def speak(self):
        return "Woof!"

class Cat:
    def speak(self):
        return "Meow!"

class Duck:
    def speak(self):
        return "Quack!"

def make_it_speak(animal):
    """Doesn't care about type, just that it has speak()."""
    return animal.speak()


# All work - no inheritance needed
print(make_it_speak(Dog()))   # Woof!
print(make_it_speak(Cat()))   # Meow!
print(make_it_speak(Duck()))  # Quack!

# If it walks like a duck and quacks like a duck, it's a duck
```

## Method Resolution Order

### MRO Basics

Order Python searches for methods in inheritance hierarchy:

```python
class A:
    def method(self):
        return "A"

class B(A):
    def method(self):
        return "B"

class C(A):
    def method(self):
        return "C"

class D(B, C):  # Multiple inheritance
    pass

d = D()
print(d.method())  # B

# View MRO
print(D.__mro__)
# (<class '__main__.D'>, <class '__main__.B'>, <class '__main__.C'>,
#  <class '__main__.A'>, <class 'object'>)
```

### C3 Linearization

Python uses C3 algorithm for MRO:

```python
class Base:
    def process(self):
        return "Base"

class Left(Base):
    def process(self):
        return "Left"

class Right(Base):
    def process(self):
        return "Right"

class Child(Left, Right):  # Order matters!
    pass

child = Child()
print(child.process())  # Left (first in parent list)

print(Child.mro())
# [Child, Left, Right, Base, object]
```

## Property Decorators

### Basic Property

Managed attribute access:

=== "main.py"

    ```python
    from person import Person

    person = Person("Alice")
    print(person.name)  # Calls getter: Alice
    person.name = "Bob"  # Calls setter: Bob
    # person.name = ""  # ValueError
    ```

=== "person.py"

    ```python
    class Person:
        def __init__(self, name):
            self._name = name  # "Private" convention

        @property
        def name(self):
            """Getter"""
            return self._name

        @name.setter
        def name(self, value):
            """Setter with validation"""
            if not value:
                raise ValueError("Name cannot be empty")
            self._name = value
    ```

### Computed Properties

=== "main.py"

    ```python
    from rectangle import Rectangle

    rect = Rectangle(5, 10)
    print(rect.area)       # 50 (computed)
    print(rect.perimeter)  # 30 (computed)

    # No setter - read-only
    # rect.area = 100  # AttributeError
    ```

=== "rectangle.py"

    ```python
    class Rectangle:
        def __init__(self, width, height):
            self.width = width
            self.height = height

        @property
        def area(self):
            """Computed on access."""
            return self.width * self.height

        @property
        def perimeter(self):
            return 2 * (self.width + self.height)
    ```

### Read-Only Properties

=== "main.py"

    ```python
    from circle import Circle

    circle = Circle(5)
    print(circle.area)  # 78.53975
    # circle.area = 100  # AttributeError - no setter
    ```

=== "circle.py"

    ```python
    class Circle:
        def __init__(self, radius):
            self._radius = radius

        @property
        def radius(self):
            return self._radius

        @property
        def area(self):
            """Read-only computed property."""
            return 3.14159 * self._radius ** 2
    ```

## Class Design Principles

### Single Responsibility

Each class should have one reason to change:

=== "Bad"

    ```python
    # Multiple responsibilities
    class User:
        def __init__(self, name, email):
            self.name = name
            self.email = email

        def save_to_database(self):
            pass  # Database logic

        def send_email(self):
            pass  # Email logic
    ```

=== "Good"

    ```python
    # Separate concerns
    class User:
        def __init__(self, name, email):
            self.name = name
            self.email = email

    class UserRepository:
        def save(self, user):
            pass  # Database logic

    class EmailService:
        def send_email(self, user):
            pass  # Email logic
    ```

### Encapsulation

Hide internal details:

=== "main.py"

    ```python
    from bank_account import BankAccount

    account = BankAccount(100)
    account.deposit(50)
    print(account.get_balance())  # 150
    # print(account.__balance)  # AttributeError - private
    ```

=== "bank_account.py"

    ```python
    class BankAccount:
        def __init__(self, balance):
            self.__balance = balance  # Name mangling (private)

        def deposit(self, amount):
            if amount > 0:
                self.__balance += amount

        def withdraw(self, amount):
            if 0 < amount <= self.__balance:
                self.__balance -= amount
                return amount
            return 0

        def get_balance(self):
            return self.__balance
    ```

### Favor Composition

=== "Inheritance"

    ```python
    # Can be rigid
    class Employee(Person, Payable, Taxable):
        pass  # Complex hierarchy
    ```

=== "Composition"

    ```python
    # Is flexible
    class Employee:
        def __init__(self, person, payment_processor, tax_calculator):
            self.person = person
            self.payment_processor = payment_processor
            self.tax_calculator = tax_calculator
    ```

## Summary

OOP enables organizing code into reusable, maintainable structures.

**Core concepts:**

- Classes define blueprints, objects are instances
- `self` refers to the instance
- `__init__` initializes new objects

**Method types:**

- Instance methods: operate on instance (`self`)
- Class methods: operate on class (`@classmethod`, `cls`)
- Static methods: utility functions (`@staticmethod`)

**Special methods:**

- `__str__`, `__repr__`: string representation
- `__eq__`, `__lt__`: comparisons
- `__add__`, `__mul__`: arithmetic
- `__len__`, `__getitem__`: containers

**Inheritance and composition:**

- Inheritance: "is-a" relationship
- Composition: "has-a" relationship
- Prefer composition for flexibility

**Design principles:**

- Single responsibility
- Encapsulation
- Polymorphism through duck typing

## Next Steps

- Explore decorators in [Advanced Functions](advanced-functions.md)
- Learn lazy evaluation with [Iterators and Generators](iterators-and-generators.md)
- Manage resources with [Context Managers](context-managers.md)
