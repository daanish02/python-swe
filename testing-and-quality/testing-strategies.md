# Testing Strategies

## Table of Contents

- [Test-Driven Development](#test-driven-development)
- [Behavior-Driven Development](#behavior-driven-development)
- [Testing Patterns](#testing-patterns)
- [Test Coverage](#test-coverage)
- [Edge Cases and Boundary Conditions](#edge-cases-and-boundary-conditions)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Test-Driven Development

**TDD (Test-Driven Development)**: A development approach where you write tests before writing code, following the cycle: Red (failing test) → Green (make it pass) → Refactor (improve code).

### TDD Cycle: Red-Green-Refactor

```python
# 1. RED: Write failing test first
def test_add():
    assert add(2, 3) == 5  # Test fails - add() doesn't exist yet

# 2. GREEN: Write minimal code to pass
def add(a, b):
    return 5  # Hardcoded, but test passes!

# 3. REFACTOR: Improve implementation
def add(a, b):
    return a + b  # Proper implementation

# Repeat cycle for next feature
```

### TDD Example: Calculator

```python
# Step 1: Write test for addition
def test_add():
    calc = Calculator()
    assert calc.add(2, 3) == 5

# Step 2: Implement Calculator with add
class Calculator:
    def add(self, a, b):
        return a + b

# Step 3: Test passes, add more tests
def test_add_negative():
    calc = Calculator()
    assert calc.add(-2, -3) == -5

def test_add_zero():
    calc = Calculator()
    assert calc.add(0, 5) == 5

# Step 4: Add subtraction
def test_subtract():
    calc = Calculator()
    assert calc.subtract(5, 3) == 2

# Step 5: Implement subtract
class Calculator:
    def add(self, a, b):
        return a + b

    def subtract(self, a, b):
        return a - b

# Continue cycle...
```

### TDD Benefits

```python
# Benefits:
# - Tests before code ensures testability
# - Clear requirements (tests define behavior)
# - High test coverage by default
# - Confidence in refactoring
# - Forces small, focused changes

# TDD mindset:
# - What should this do? (Write test)
# - Does it work? (Run test - should fail)
# - Make it work (Write code)
# - Make it right (Refactor)
```

## Behavior-Driven Development

**BDD (Behavior-Driven Development)**: A testing approach that describes system behavior in plain language (Given-When-Then) that's readable by non-programmers.

### BDD with pytest-bdd

```python
# Install: pip install pytest-bdd

# features/calculator.feature
"""
Feature: Calculator
    As a user
    I want to perform basic arithmetic
    So that I can calculate results

Scenario: Add two numbers
    Given I have a calculator
    When I add 2 and 3
    Then the result should be 5

Scenario: Subtract two numbers
    Given I have a calculator
    When I subtract 3 from 5
    Then the result should be 2
"""

# tests/test_calculator.py
from pytest_bdd import scenarios, given, when, then
from calculator import Calculator

scenarios('../features/calculator.feature')

@given('I have a calculator')
def calculator():
    return Calculator()

@when('I add 2 and 3')
def add_numbers(calculator):
    calculator.result = calculator.add(2, 3)

@then('the result should be 5')
def check_result(calculator):
    assert calculator.result == 5
```

### BDD Style with pytest

```python
# Without pytest-bdd plugin, use descriptive names
def test_user_registration():
    # Given: A new user with valid data
    username = "alice"
    email = "alice@example.com"
    password = "secret123"

    # When: Registering the user
    user = register_user(username, email, password)

    # Then: User should be created and activated
    assert user.username == username
    assert user.email == email
    assert user.is_active is True

def test_user_login_with_correct_password():
    # Given: An existing user
    user = create_user("alice", "alice@example.com", "secret123")

    # When: Logging in with correct password
    result = login(user.username, "secret123")

    # Then: Login should succeed
    assert result.success is True
    assert result.user == user

def test_user_login_with_wrong_password():
    # Given: An existing user
    user = create_user("alice", "alice@example.com", "secret123")

    # When: Logging in with wrong password
    result = login(user.username, "wrong_password")

    # Then: Login should fail
    assert result.success is False
    assert result.error == "Invalid password"
```

## Testing Patterns

### Arrange-Act-Assert

```python
def test_order_total_calculation():
    # Arrange: Set up test data
    order = Order()
    order.add_item(Item(price=10, quantity=2))
    order.add_item(Item(price=5, quantity=3))

    # Act: Execute the behavior
    total = order.calculate_total()

    # Assert: Verify the result
    assert total == 35
```

### Builder Pattern for Test Data

```python
class UserBuilder:
    def __init__(self):
        self._name = "Default Name"
        self._email = "default@example.com"
        self._age = 18
        self._active = True

    def with_name(self, name):
        self._name = name
        return self

    def with_email(self, email):
        self._email = email
        return self

    def with_age(self, age):
        self._age = age
        return self

    def inactive(self):
        self._active = False
        return self

    def build(self):
        return User(self._name, self._email, self._age, self._active)

# Usage in tests
def test_adult_user():
    user = UserBuilder().with_age(25).build()
    assert user.is_adult()

def test_inactive_user():
    user = UserBuilder().inactive().build()
    assert not user.is_active
```

### Object Mother Pattern

```python
class UserMother:
    @staticmethod
    def create_default():
        return User("Alice", "alice@example.com", 30)

    @staticmethod
    def create_admin():
        user = User("Admin", "admin@example.com", 35)
        user.role = "admin"
        return user

    @staticmethod
    def create_inactive():
        user = User("Inactive", "inactive@example.com", 25)
        user.active = False
        return user

# Usage
def test_admin_permissions():
    admin = UserMother.create_admin()
    assert admin.has_permission("delete_user")

def test_inactive_user_cannot_login():
    user = UserMother.create_inactive()
    assert not user.can_login()
```

### Test Fixtures as Factories

```python
@pytest.fixture
def user_factory():
    def _create_user(name="Alice", email="alice@example.com", **kwargs):
        return User(name, email, **kwargs)
    return _create_user

def test_with_custom_user(user_factory):
    user = user_factory(name="Bob", email="bob@example.com")
    assert user.name == "Bob"

def test_with_default_user(user_factory):
    user = user_factory()
    assert user.name == "Alice"
```

## Test Coverage

**Test coverage**: A metric showing what percentage of code is executed by tests; higher coverage doesn't guarantee bug-free code.

### Measuring Coverage

```python
# Install: pip install pytest-cov

# Run tests with coverage
# pytest --cov=mymodule tests/

# Generate HTML report
# pytest --cov=mymodule --cov-report=html tests/

# View coverage report
# open htmlcov/index.html

# Example output:
# mymodule/calculator.py    100%
# mymodule/user.py          85%
# mymodule/service.py       70%
# TOTAL                     85%
```

### What Coverage Means

```python
# 100% coverage doesn't mean bug-free code!
def divide(a, b):
    return a / b  # Missing zero check!

def test_divide():
    assert divide(6, 2) == 3  # 100% coverage, but missing edge case!

# Better tests:
def test_divide_positive():
    assert divide(6, 2) == 3

def test_divide_negative():
    assert divide(-6, 2) == -3

def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(6, 0)
```

### What Coverage Doesn't Mean

```python
# High coverage doesn't guarantee:
# - All edge cases tested
# - Correct behavior
# - Good test quality
# - No bugs

# Focus on:
# - Testing important paths
# - Edge cases and boundaries
# - Error conditions
# - Integration points

# Coverage is a tool, not a goal
```

### Targeting Important Coverage

```python
# Focus on:
# 1. Business logic
def calculate_discount(user, order):
    # Important! Test thoroughly
    pass

# 2. Error handling
def process_payment(amount):
    # Test failure cases
    pass

# 3. Integration points
def save_to_database(data):
    # Test database interaction
    pass

# Less critical:
# - Simple getters/setters
# - Configuration code
# - Third-party library wrappers
```

## Edge Cases and Boundary Conditions

**Edge case**: An unusual or extreme input that might cause unexpected behavior (e.g., empty lists, zero, negative numbers, very large values).

### Boundary Testing

```python
# Test boundaries, not just typical values
def test_age_validation():
    # Boundary: 18 is minimum
    assert is_adult(17) is False  # Below boundary
    assert is_adult(18) is True   # At boundary
    assert is_adult(19) is True   # Above boundary

def test_string_length():
    # Boundary: max length 50
    assert validate_name("a" * 49)  # Below
    assert validate_name("a" * 50)  # At boundary
    with pytest.raises(ValueError):
        validate_name("a" * 51)  # Above
```

### Edge Cases

```python
# Empty collections
def test_sum_empty_list():
    assert sum_numbers([]) == 0

def test_average_empty_list():
    with pytest.raises(ValueError):
        average([])

# Null/None values
def test_process_none():
    with pytest.raises(ValueError):
        process_data(None)

# Zero values
def test_divide_by_zero():
    with pytest.raises(ZeroDivisionError):
        divide(5, 0)

# Negative values
def test_negative_quantity():
    with pytest.raises(ValueError):
        Order(quantity=-1)

# Large values
def test_large_number():
    result = factorial(20)
    assert result == 2432902008176640000

# Special characters
def test_special_chars_in_name():
    user = User("O'Brien", "test@example.com")
    assert user.name == "O'Brien"
```

### Equivalence Partitioning

```python
# Divide input space into partitions
# Test one value from each partition

# Age validation: child (0-17), adult (18-64), senior (65+)
def test_child():
    assert get_category(10) == "child"

def test_adult():
    assert get_category(30) == "adult"

def test_senior():
    assert get_category(70) == "senior"

# Plus boundaries
def test_boundaries():
    assert get_category(17) == "child"
    assert get_category(18) == "adult"
    assert get_category(64) == "adult"
    assert get_category(65) == "senior"
```

### Error Cases

```python
# Test error conditions
def test_invalid_email():
    with pytest.raises(ValueError):
        User("Alice", "invalid-email")

def test_negative_price():
    with pytest.raises(ValueError):
        Product(name="Item", price=-10)

def test_missing_required_field():
    with pytest.raises(ValueError):
        User(name="", email="test@example.com")

def test_duplicate_username():
    create_user("alice", "alice1@example.com")
    with pytest.raises(DuplicateError):
        create_user("alice", "alice2@example.com")
```

### State Transitions

```python
# Test state changes
def test_order_state_transitions():
    order = Order()

    # New -> Pending
    assert order.state == "new"
    order.submit()
    assert order.state == "pending"

    # Pending -> Processing
    order.process()
    assert order.state == "processing"

    # Processing -> Shipped
    order.ship()
    assert order.state == "shipped"

    # Processing -> Cancelled
    order2 = Order()
    order2.submit()
    order2.process()
    order2.cancel()
    assert order2.state == "cancelled"

# Test invalid transitions
def test_cannot_cancel_shipped_order():
    order = Order()
    order.submit()
    order.process()
    order.ship()

    with pytest.raises(InvalidStateError):
        order.cancel()
```

## Summary

Testing strategies:

**Test-Driven Development:**

- Red-Green-Refactor cycle
- Write tests first
- Ensures testability
- High coverage by design

**Behavior-Driven Development:**

- Given-When-Then structure
- Business-readable specs
- Collaboration with stakeholders

**Testing patterns:**

- Arrange-Act-Assert
- Builder pattern for test data
- Object Mother pattern
- Test fixtures as factories

**Test coverage:**

- Measure with pytest-cov
- Not a goal, a tool
- Focus on important code
- 100% ≠ bug-free

**Edge cases:**

- Boundary conditions
- Empty/null values
- Zero, negative, large values
- Special characters
- Error conditions
- State transitions

Good tests cover not just happy paths but edge cases and error conditions.

## Next Steps

- Master [Mocking and Fixtures](mocking-and-fixtures.md)
- Explore [Advanced Testing](advanced-testing.md)
- Improve with [Code Quality Tools](code-quality-tools.md)
