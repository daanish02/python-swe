# Testing Fundamentals

## Table of Contents

- [Why Testing Matters](#why-testing-matters)
- [Types of Tests](#types-of-tests)
- [Test Structure](#test-structure)
- [Writing Your First Tests](#writing-your-first-tests)
- [Test Organization](#test-organization)
- [Testing Best Practices](#testing-best-practices)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Why Testing Matters

### Benefits of Testing

```python
# Testing provides:
# - Confidence in code changes
# - Documentation of behavior
# - Safety net for refactoring
# - Early bug detection
# - Better design (testable code is well-designed)

# Without tests:
# - Fear of changing code
# - Regressions go unnoticed
# - Manual testing is time-consuming
# - Hard to onboard new developers
```

### Cost of Not Testing

```python
# Scenario: Bug found in production
# - Customer impact
# - Emergency hotfix required
# - Team pulled from other work
# - Reputation damage
# - Lost revenue

# With tests: Bug caught before deployment
# - Fix during development
# - No customer impact
# - Normal development cycle
```

## Types of Tests

### Unit Tests

**Unit test**: A test that verifies a single function, method, or class in isolation from other parts of the system.

Test individual functions or methods in isolation:

```python
# Unit test: Tests single function
def add(a, b):
    return a + b

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0

# Characteristics:
# - Fast (milliseconds)
# - Isolated (no dependencies)
# - Focused (one thing)
# - Many of these (hundreds to thousands)
```

### Integration Tests

**Integration test**: A test that verifies multiple components work correctly together (e.g., code + database, code + external API).

Test how components work together:

```python
# Integration test: Tests multiple components
class UserRepository:
    def save(self, user):
        # Database interaction
        pass

class UserService:
    def __init__(self, repository):
        self.repository = repository

    def create_user(self, name, email):
        user = User(name, email)
        self.repository.save(user)
        return user

def test_user_service_creates_user():
    # Test with real database
    repository = UserRepository(database)
    service = UserService(repository)

    user = service.create_user("Alice", "alice@example.com")

    # Verify user was saved to database
    saved_user = repository.find_by_email("alice@example.com")
    assert saved_user.name == "Alice"

# Characteristics:
# - Slower than unit tests
# - Tests interactions
# - Uses real dependencies (or close to real)
# - Fewer of these than unit tests
```

### End-to-End Tests

Test entire application flow:

```python
# E2E test: Tests complete user workflow
def test_user_registration_flow():
    # Open browser
    browser = Browser()
    browser.visit("http://localhost/register")

    # Fill form
    browser.fill("name", "Alice")
    browser.fill("email", "alice@example.com")
    browser.fill("password", "secret123")

    # Submit
    browser.click("Submit")

    # Verify success
    assert browser.text_contains("Registration successful")

    # Verify email sent
    assert email_was_sent_to("alice@example.com")

    # Verify can login
    browser.visit("http://localhost/login")
    browser.fill("email", "alice@example.com")
    browser.fill("password", "secret123")
    browser.click("Login")
    assert browser.text_contains("Welcome, Alice")

# Characteristics:
# - Slowest (seconds to minutes)
# - Tests entire system
# - Uses real infrastructure
# - Few of these (critical paths only)
```

### Testing Pyramid

```python
# Testing Pyramid (bottom to top):
#
#        /\
#       /  \     E2E Tests (Few)
#      /____\
#     /      \
#    /  INTEG \  Integration Tests (Some)
#   /__________\
#  /            \
# /   UNIT TESTS \ Unit Tests (Many)
#/________________\
#
# Majority of tests should be unit tests
# Some integration tests
# Few E2E tests
```

## Test Structure

### Arrange-Act-Assert (AAA)

```python
def test_user_creation():
    # Arrange: Set up test data and dependencies
    name = "Alice"
    email = "alice@example.com"
    repository = MockRepository()
    service = UserService(repository)

    # Act: Execute the code being tested
    user = service.create_user(name, email)

    # Assert: Verify the results
    assert user.name == "Alice"
    assert user.email == "alice@example.com"
    assert repository.save_called
```

### Given-When-Then (BDD Style)

```python
def test_user_with_invalid_email():
    # Given a user service
    service = UserService()

    # When creating user with invalid email
    with pytest.raises(ValueError):
        service.create_user("Alice", "invalid-email")

    # Then ValueError should be raised
```

## Writing Your First Tests

### Simple Function Test

```python
# calculator.py
def add(a, b):
    return a + b

def subtract(a, b):
    return a - b

def multiply(a, b):
    return a * b

def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# test_calculator.py
import pytest
from calculator import add, subtract, multiply, divide

def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0

def test_subtract():
    assert subtract(5, 3) == 2
    assert subtract(0, 5) == -5

def test_multiply():
    assert multiply(3, 4) == 12
    assert multiply(-2, 3) == -6

def test_divide():
    assert divide(6, 2) == 3
    assert divide(5, 2) == 2.5

def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(5, 0)
```

### Testing a Class

```python
# user.py
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email
        self.active = True

    def deactivate(self):
        self.active = False

    def activate(self):
        self.active = True

    def update_email(self, new_email):
        if '@' not in new_email:
            raise ValueError("Invalid email")
        self.email = new_email

# test_user.py
import pytest
from user import User

def test_user_creation():
    user = User("Alice", "alice@example.com")

    assert user.name == "Alice"
    assert user.email == "alice@example.com"
    assert user.active is True

def test_user_deactivation():
    user = User("Alice", "alice@example.com")

    user.deactivate()

    assert user.active is False

def test_user_activation():
    user = User("Alice", "alice@example.com")
    user.deactivate()

    user.activate()

    assert user.active is True

def test_update_email_with_valid_email():
    user = User("Alice", "alice@example.com")

    user.update_email("newemail@example.com")

    assert user.email == "newemail@example.com"

def test_update_email_with_invalid_email():
    user = User("Alice", "alice@example.com")

    with pytest.raises(ValueError):
        user.update_email("invalid-email")
```

## Test Organization

### File Structure

```
myproject/
├── src/
│   └── myproject/
│       ├── __init__.py
│       ├── calculator.py
│       └── user.py
└── tests/
    ├── __init__.py
    ├── test_calculator.py
    └── test_user.py

# Or mirror source structure:
myproject/
├── myproject/
│   ├── __init__.py
│   ├── calculator.py
│   ├── user.py
│   └── services/
│       ├── __init__.py
│       └── email_service.py
└── tests/
    ├── __init__.py
    ├── test_calculator.py
    ├── test_user.py
    └── services/
        ├── __init__.py
        └── test_email_service.py
```

### Naming Conventions

```python
# Test files: test_*.py or *_test.py
test_calculator.py
test_user.py
calculator_test.py

# Test functions: test_*
def test_add():
    pass

def test_user_creation():
    pass

# Test classes: Test*
class TestCalculator:
    def test_add(self):
        pass

    def test_subtract(self):
        pass

class TestUser:
    def test_creation(self):
        pass

    def test_deactivation(self):
        pass
```

### Grouping Tests

```python
# Group related tests in classes
class TestUserAuthentication:
    def test_login_with_valid_credentials(self):
        pass

    def test_login_with_invalid_password(self):
        pass

    def test_login_with_nonexistent_user(self):
        pass

class TestUserProfile:
    def test_update_profile(self):
        pass

    def test_upload_avatar(self):
        pass

    def test_delete_profile(self):
        pass
```

## Testing Best Practices

### Test One Thing

**Principle:** Each test should verify one specific behavior or scenario.

**How to decide if you're testing too much:**

1. Can you describe what the test does in one sentence?
   - If you need "and" or multiple sentences → Split it
2. If one assertion fails, would you still want to run the others?
   - If yes → They're testing different things → Split them
3. Does the test name contain "and"?
   - If yes → Probably testing multiple things

**Decision process:**

```
Look at your test:
├─ Does it test creation AND behavior? → Split
├─ Does it test multiple methods? → Split
├─ Does it test multiple scenarios? → Split
└─ Does it test one method with one scenario? → Good!
```

```python
# Bad: Test multiple things
def test_user():
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"  # Testing name
    assert user.email == "alice@example.com"  # Testing email
    user.deactivate()
    assert not user.active  # Testing deactivation
    user.update_email("new@example.com")
    assert user.email == "new@example.com"  # Testing update

# Problem: If deactivation breaks, you can't see if update_email works
# Problem: Test name doesn't describe what it tests
# Problem: Hard to understand what failed when it breaks

# Good: Separate tests
def test_user_has_correct_name():
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"

def test_user_has_correct_email():
    user = User("Alice", "alice@example.com")
    assert user.email == "alice@example.com"

def test_user_can_be_deactivated():
    user = User("Alice", "alice@example.com")
    user.deactivate()
    assert not user.active

def test_user_can_update_email():
    user = User("Alice", "alice@example.com")
    user.update_email("new@example.com")
    assert user.email == "new@example.com"

# Better: Each test has clear purpose
# Better: Failures are easy to diagnose
# Better: Tests can run independently
```

**Exception:** Multiple related assertions are OK if they verify the same behavior:

```python
# This is OK - all assertions verify the same behavior
def test_parse_name_returns_all_parts():
    result = parse_name("John Robert Smith")
    assert result["first"] == "John"
    assert result["middle"] == "Robert"
    assert result["last"] == "Smith"
    # All assertions verify parsing works correctly
```

### Use Descriptive Names

```python
# Bad: Unclear test names
def test_1():
    pass

def test_user():
    pass

def test_calc():
    pass

# Good: Clear test names
def test_add_returns_sum_of_two_numbers():
    pass

def test_user_deactivation_sets_active_to_false():
    pass

def test_divide_by_zero_raises_value_error():
    pass
```

### Test Edge Cases

```python
def test_add_with_positive_numbers():
    assert add(2, 3) == 5

def test_add_with_negative_numbers():
    assert add(-2, -3) == -5

def test_add_with_zero():
    assert add(0, 5) == 5
    assert add(5, 0) == 5
    assert add(0, 0) == 0

def test_add_with_large_numbers():
    assert add(1000000, 2000000) == 3000000

def test_add_with_floats():
    assert add(0.1, 0.2) == pytest.approx(0.3)
```

### Keep Tests Fast

```python
# Bad: Slow test
def test_user_creation():
    time.sleep(1)  # Don't do this!
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"

# Good: Fast test
def test_user_creation():
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"

# Use mocks for slow operations
def test_send_email():
    with patch('email_service.send') as mock_send:
        mock_send.return_value = True
        result = send_welcome_email("user@example.com")
        assert result is True
```

### Tests Should Be Independent

```python
# Bad: Tests depend on each other
user = None

def test_create_user():
    global user
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"

def test_deactivate_user():
    global user
    user.deactivate()  # Depends on test_create_user
    assert not user.active

# Good: Each test is independent
def test_create_user():
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"

def test_deactivate_user():
    user = User("Alice", "alice@example.com")
    user.deactivate()
    assert not user.active
```

### Test Discovery Checklist

When looking at a function or class, use this checklist to identify what tests to write:

**For any function:**

1. **Does it have conditional logic?** (if/else, match/case)
   - Test each branch
2. **Does it validate inputs?**
   - Test with invalid inputs (should raise errors)
3. **Does it have edge cases?** (empty list, zero, None, boundaries)
   - Test each edge case
4. **Does it return a value?**
   - Test the return value with typical inputs
5. **Does it have side effects?** (modifies state, writes files, calls APIs)
   - Test the side effects occurred

**For classes:**

1. **Initialization:** Test object creation with various parameters
2. **Each public method:** Apply the function checklist above
3. **State changes:** Test that methods modify state correctly
4. **Method interactions:** Test sequences of method calls

**Example application:**

```python
def calculate_discount(price, discount_percent):
    if price < 0:
        raise ValueError("Price cannot be negative")
    if discount_percent < 0 or discount_percent > 100:
        raise ValueError("Discount must be 0-100")
    return price * (discount_percent / 100)

# Checklist:
# Conditional logic? Yes (2 if statements) → Test each branch
# Validates inputs? Yes → Test invalid price, invalid discount
# Edge cases? Yes → Test 0 price, 0%, 100% discount
# Returns value? Yes → Test return with typical inputs
# Side effects? No

# Tests needed:
# 1. Normal case: typical price and discount
# 2. Edge: zero price
# 3. Edge: 0% discount
# 4. Edge: 100% discount
# 5. Error: negative price
# 6. Error: negative discount
# 7. Error: discount > 100

# That's 7 tests - comprehensive coverage!
```

**Use this process:**

1. Go through the checklist for each function
2. Write down the tests you need (don't write code yet!)
3. Review your list - missing anything?
4. Then write the actual test code

This systematic approach ensures you don't miss important test cases!

## Summary

Testing fundamentals:

**Why test:**

- Confidence in changes
- Early bug detection
- Documentation
- Better design

**Types of tests:**

- **Unit:** Fast, isolated, many
- **Integration:** Medium speed, component interaction, some
- **E2E:** Slow, full system, few

**Test structure:**

- Arrange-Act-Assert
- Given-When-Then
- Clear sections

**Best practices:**

- Test one thing per test
- Descriptive names
- Test edge cases
- Keep tests fast
- Tests are independent

**Organization:**

- Mirror source structure
- Clear naming conventions
- Group related tests

Testing is essential for building reliable, maintainable software.

## Next Steps

- Learn [Unit Testing](unit-testing.md) with pytest
- Explore [Testing Strategies](testing-strategies.md)
- Master [Mocking and Fixtures](mocking-and-fixtures.md)
