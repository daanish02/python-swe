# SOLID Principles

## Table of Contents

- [What are SOLID Principles?](#what-are-solid-principles)
- [Single Responsibility Principle](#single-responsibility-principle)
- [Open/Closed Principle](#openclosed-principle)
- [Liskov Substitution Principle](#liskov-substitution-principle)
- [Interface Segregation Principle](#interface-segregation-principle)
- [Dependency Inversion Principle](#dependency-inversion-principle)
- [Summary](#summary)
- [Next Steps](#next-steps)

## What are SOLID Principles?

### Definition

SOLID is an acronym for five design principles that make software more maintainable, flexible, and scalable:

```python
# S - Single Responsibility Principle
# O - Open/Closed Principle
# L - Liskov Substitution Principle
# I - Interface Segregation Principle
# D - Dependency Inversion Principle

# Benefits:
# - Easier to understand and maintain
# - More flexible and extensible
# - Better testability
# - Reduced coupling
```

## Single Responsibility Principle

### A class should have one, and only one, reason to change

```python
# Bad: Multiple responsibilities
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def save_to_database(self):
        # Database logic
        pass

    def send_welcome_email(self):
        # Email logic
        pass

    def generate_report(self):
        # Reporting logic
        pass

# Good: Single responsibility per class
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserRepository:
    def save(self, user):
        # Database logic
        pass

class EmailService:
    def send_welcome_email(self, user):
        # Email logic
        pass

class UserReportGenerator:
    def generate_report(self, user):
        # Reporting logic
        pass
```

### Real-World Example

```python
# Bad: Order class does too much
class Order:
    def __init__(self, items):
        self.items = items

    def calculate_total(self):
        return sum(item.price * item.quantity for item in self.items)

    def save_to_database(self):
        # Database code
        pass

    def send_confirmation_email(self):
        # Email code
        pass

    def generate_invoice_pdf(self):
        # PDF generation code
        pass

# Good: Separate concerns
class Order:
    def __init__(self, items):
        self.items = items

    def calculate_total(self):
        return sum(item.price * item.quantity for item in self.items)

class OrderRepository:
    def save(self, order):
        # Database code
        pass

class OrderNotificationService:
    def send_confirmation(self, order):
        # Email code
        pass

class InvoiceGenerator:
    def generate_pdf(self, order):
        # PDF generation code
        pass

# Usage
order = Order(items)
total = order.calculate_total()

repository = OrderRepository()
repository.save(order)

notifier = OrderNotificationService()
notifier.send_confirmation(order)

invoice = InvoiceGenerator()
invoice.generate_pdf(order)
```

## Open/Closed Principle

### Software entities should be open for extension but closed for modification

```python
# Bad: Must modify class to add new shape
class AreaCalculator:
    def calculate_area(self, shapes):
        total_area = 0
        for shape in shapes:
            if shape.type == 'rectangle':
                total_area += shape.width * shape.height
            elif shape.type == 'circle':
                total_area += 3.14 * shape.radius ** 2
            # Must modify to add triangle, etc.
        return total_area

# Good: Open for extension, closed for modification
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        return 3.14 * self.radius ** 2

class Triangle(Shape):
    def __init__(self, base, height):
        self.base = base
        self.height = height

    def area(self):
        return 0.5 * self.base * self.height

class AreaCalculator:
    def calculate_area(self, shapes):
        return sum(shape.area() for shape in shapes)

# Add new shapes without modifying AreaCalculator
shapes = [Rectangle(5, 10), Circle(7), Triangle(4, 6)]
calculator = AreaCalculator()
print(calculator.calculate_area(shapes))
```

### Real-World Example: Payment Processing

```python
# Bad: Must modify to add new payment methods
class PaymentProcessor:
    def process_payment(self, payment_type, amount):
        if payment_type == 'credit_card':
            # Credit card processing
            pass
        elif payment_type == 'paypal':
            # PayPal processing
            pass
        elif payment_type == 'bitcoin':
            # Bitcoin processing
            pass

# Good: Extensible payment processing
class PaymentMethod(ABC):
    @abstractmethod
    def process(self, amount):
        pass

class CreditCardPayment(PaymentMethod):
    def __init__(self, card_number):
        self.card_number = card_number

    def process(self, amount):
        print(f"Processing ${amount} via credit card")

class PayPalPayment(PaymentMethod):
    def __init__(self, email):
        self.email = email

    def process(self, amount):
        print(f"Processing ${amount} via PayPal")

class BitcoinPayment(PaymentMethod):
    def __init__(self, wallet_address):
        self.wallet_address = wallet_address

    def process(self, amount):
        print(f"Processing ${amount} via Bitcoin")

class PaymentProcessor:
    def process_payment(self, payment_method: PaymentMethod, amount):
        payment_method.process(amount)

# Easy to add new payment methods
processor = PaymentProcessor()
processor.process_payment(CreditCardPayment("1234"), 100)
processor.process_payment(PayPalPayment("user@email.com"), 50)
```

## Liskov Substitution Principle

### Objects should be replaceable with instances of their subtypes without altering correctness

```python
# Bad: Subclass violates base class contract
class Bird:
    def fly(self):
        return "Flying"

class Sparrow(Bird):
    def fly(self):
        return "Sparrow flying"

class Ostrich(Bird):
    def fly(self):
        raise Exception("Can't fly")  # Violates LSP!

# Using Bird interface
def make_bird_fly(bird: Bird):
    return bird.fly()

sparrow = Sparrow()
make_bird_fly(sparrow)  # OK

ostrich = Ostrich()
make_bird_fly(ostrich)  # Exception! LSP violated

# Good: Correct abstraction
class Bird:
    pass

class FlyingBird(Bird):
    def fly(self):
        return "Flying"

class Sparrow(FlyingBird):
    def fly(self):
        return "Sparrow flying"

class Ostrich(Bird):
    def run(self):
        return "Running fast"

# Now we can substitute correctly
def make_flying_bird_fly(bird: FlyingBird):
    return bird.fly()

make_flying_bird_fly(Sparrow())  # OK
# Ostrich is not a FlyingBird, type checker catches this
```

### Real-World Example: Rectangle and Square

```python
# Bad: Square breaks rectangle's behavior
class Rectangle:
    def __init__(self, width, height):
        self._width = width
        self._height = height

    @property
    def width(self):
        return self._width

    @width.setter
    def width(self, value):
        self._width = value

    @property
    def height(self):
        return self._height

    @height.setter
    def height(self, value):
        self._height = value

    def area(self):
        return self._width * self._height

class Square(Rectangle):
    def __init__(self, size):
        super().__init__(size, size)

    @Rectangle.width.setter
    def width(self, value):
        self._width = value
        self._height = value  # Breaks Rectangle behavior!

    @Rectangle.height.setter
    def height(self, value):
        self._width = value
        self._height = value

# Problem
def resize_rectangle(rect: Rectangle):
    rect.width = 5
    rect.height = 10
    assert rect.area() == 50  # Works for Rectangle

rect = Rectangle(2, 3)
resize_rectangle(rect)  # OK

square = Square(4)
resize_rectangle(square)  # Fails! area() is 100, not 50

# Good: Separate abstractions
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, size):
        self.size = size

    def area(self):
        return self.size * self.size
```

## Interface Segregation Principle

### Clients shouldn't depend on interfaces they don't use

```python
# Bad: Fat interface
class Worker(ABC):
    @abstractmethod
    def work(self):
        pass

    @abstractmethod
    def eat(self):
        pass

    @abstractmethod
    def sleep(self):
        pass

class Human(Worker):
    def work(self):
        print("Human working")

    def eat(self):
        print("Human eating")

    def sleep(self):
        print("Human sleeping")

class Robot(Worker):
    def work(self):
        print("Robot working")

    def eat(self):
        pass  # Robots don't eat!

    def sleep(self):
        pass  # Robots don't sleep!

# Good: Segregated interfaces
class Workable(ABC):
    @abstractmethod
    def work(self):
        pass

class Eatable(ABC):
    @abstractmethod
    def eat(self):
        pass

class Sleepable(ABC):
    @abstractmethod
    def sleep(self):
        pass

class Human(Workable, Eatable, Sleepable):
    def work(self):
        print("Human working")

    def eat(self):
        print("Human eating")

    def sleep(self):
        print("Human sleeping")

class Robot(Workable):
    def work(self):
        print("Robot working")

# Functions depend on minimal interfaces
def make_work(worker: Workable):
    worker.work()

make_work(Human())  # OK
make_work(Robot())  # OK
```

### Real-World Example: Document Interface

```python
# Bad: Bloated interface
class Document(ABC):
    @abstractmethod
    def open(self):
        pass

    @abstractmethod
    def save(self):
        pass

    @abstractmethod
    def print(self):
        pass

    @abstractmethod
    def fax(self):
        pass

class ModernDocument(Document):
    def open(self):
        print("Opening document")

    def save(self):
        print("Saving document")

    def print(self):
        print("Printing document")

    def fax(self):
        raise NotImplementedError("No fax support")

# Good: Segregated interfaces
class Openable(ABC):
    @abstractmethod
    def open(self):
        pass

class Saveable(ABC):
    @abstractmethod
    def save(self):
        pass

class Printable(ABC):
    @abstractmethod
    def print(self):
        pass

class Faxable(ABC):
    @abstractmethod
    def fax(self):
        pass

class ModernDocument(Openable, Saveable, Printable):
    def open(self):
        print("Opening document")

    def save(self):
        print("Saving document")

    def print(self):
        print("Printing document")

class LegacyDocument(Openable, Saveable, Printable, Faxable):
    def open(self):
        print("Opening document")

    def save(self):
        print("Saving document")

    def print(self):
        print("Printing document")

    def fax(self):
        print("Faxing document")
```

## Dependency Inversion Principle

### Depend on abstractions, not concretions

```python
# Bad: High-level depends on low-level
class MySQLDatabase:
    def connect(self):
        print("Connecting to MySQL")

    def query(self, sql):
        print(f"MySQL query: {sql}")

class UserService:
    def __init__(self):
        self.db = MySQLDatabase()  # Tight coupling!

    def get_user(self, user_id):
        return self.db.query(f"SELECT * FROM users WHERE id={user_id}")

# Can't easily switch to PostgreSQL or test with mock

# Good: Both depend on abstraction
class Database(ABC):
    @abstractmethod
    def connect(self):
        pass

    @abstractmethod
    def query(self, sql):
        pass

class MySQLDatabase(Database):
    def connect(self):
        print("Connecting to MySQL")

    def query(self, sql):
        print(f"MySQL query: {sql}")
        return []

class PostgreSQLDatabase(Database):
    def connect(self):
        print("Connecting to PostgreSQL")

    def query(self, sql):
        print(f"PostgreSQL query: {sql}")
        return []

class UserService:
    def __init__(self, database: Database):
        self.db = database  # Depend on abstraction

    def get_user(self, user_id):
        return self.db.query(f"SELECT * FROM users WHERE id={user_id}")

# Easy to switch implementations
mysql_service = UserService(MySQLDatabase())
postgres_service = UserService(PostgreSQLDatabase())

# Easy to test with mock
class MockDatabase(Database):
    def connect(self):
        pass

    def query(self, sql):
        return [{"id": 1, "name": "Test User"}]

test_service = UserService(MockDatabase())
```

### Real-World Example: Notification System

```python
# Bad: Tight coupling
class EmailSender:
    def send(self, message):
        print(f"Sending email: {message}")

class NotificationService:
    def __init__(self):
        self.sender = EmailSender()  # Tight coupling

    def notify(self, message):
        self.sender.send(message)

# Can't easily add SMS or push notifications

# Good: Dependency inversion
class MessageSender(ABC):
    @abstractmethod
    def send(self, message):
        pass

class EmailSender(MessageSender):
    def send(self, message):
        print(f"Sending email: {message}")

class SMSSender(MessageSender):
    def send(self, message):
        print(f"Sending SMS: {message}")

class PushNotificationSender(MessageSender):
    def send(self, message):
        print(f"Sending push notification: {message}")

class NotificationService:
    def __init__(self, senders: list[MessageSender]):
        self.senders = senders

    def notify(self, message):
        for sender in self.senders:
            sender.send(message)

# Flexible configuration
email = EmailSender()
sms = SMSSender()
push = PushNotificationSender()

service = NotificationService([email, sms])
service.notify("Hello!")

# Or just email
service = NotificationService([email])
service.notify("Email only")
```

## Summary

SOLID principles in Python:

**Single Responsibility:**

- One class, one responsibility
- One reason to change
- Easier to understand and maintain

**Open/Closed:**

- Open for extension
- Closed for modification
- Use abstraction and polymorphism

**Liskov Substitution:**

- Subtypes must be substitutable
- Don't break base class contracts
- Design correct abstractions

**Interface Segregation:**

- Small, focused interfaces
- Clients depend on what they need
- Avoid fat interfaces

**Dependency Inversion:**

- Depend on abstractions, not concretions
- High-level and low-level depend on abstractions
- Enables flexibility and testability

SOLID principles lead to more maintainable, flexible, and testable code.

## Next Steps

- Apply in [Design Patterns](design-patterns.md)
- Improve with [Refactoring](refactoring.md)
- Organize with [Code Organization](code-organization.md)
