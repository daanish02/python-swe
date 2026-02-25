# Unit Testing

## Table of Contents

- [Testing Frameworks](#testing-frameworks)
- [Test Discovery](#test-discovery)
- [Fixtures](#fixtures)
- [Parameterized Tests](#parameterized-tests)
- [Test Isolation](#test-isolation)
- [Mocking Dependencies](#mocking-dependencies)
- [Assertions](#assertions)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Testing Frameworks

### pytest

Modern, powerful testing framework:

```python
# Install
# pip install pytest

# Simple test
def test_addition():
    assert 1 + 1 == 2

def test_string_length():
    assert len("hello") == 5

# Run tests
# pytest
# pytest test_file.py
# pytest -v  # Verbose
# pytest -k "test_addition"  # Run specific test
```

### unittest

Built-in testing framework:

```python
import unittest

class TestMath(unittest.TestCase):
    def test_addition(self):
        self.assertEqual(1 + 1, 2)

    def test_subtraction(self):
        self.assertEqual(5 - 3, 2)

    def test_multiplication(self):
        self.assertEqual(3 * 4, 12)

if __name__ == '__main__':
    unittest.main()
```

### pytest vs unittest

```python
# pytest: More concise
def test_something():
    assert value == expected

# unittest: More verbose
class TestSomething(unittest.TestCase):
    def test_something(self):
        self.assertEqual(value, expected)

# pytest advantages:
# - Simple assert statements
# - Better fixture system
# - Parameterized tests
# - Better output
# - Plugin ecosystem
```

## Test Discovery

### pytest Discovery Rules

```python
# pytest discovers:
# - Files: test_*.py or *_test.py
# - Functions: test_*
# - Classes: Test* (without __init__)
# - Methods: test_*

# Example structure:
tests/
├── test_calculator.py
├── test_user.py
└── integration/
    └── test_api.py

# Run all tests
# pytest

# Run specific directory
# pytest tests/integration/

# Run specific file
# pytest tests/test_calculator.py

# Run specific test
# pytest tests/test_calculator.py::test_add
```

### Organizing Tests

```python
# tests/test_calculator.py
class TestBasicOperations:
    def test_add(self):
        assert add(2, 3) == 5

    def test_subtract(self):
        assert subtract(5, 3) == 2

class TestAdvancedOperations:
    def test_power(self):
        assert power(2, 3) == 8

    def test_sqrt(self):
        assert sqrt(16) == 4

# Run specific class
# pytest tests/test_calculator.py::TestBasicOperations

# Run specific method
# pytest tests/test_calculator.py::TestBasicOperations::test_add
```

## Fixtures

**Fixture**: Reusable setup code that prepares test data or environment before tests run (e.g., creating test users, database connections).

### Basic Fixtures

```python
import pytest

@pytest.fixture
def sample_user():
    """Create a sample user for testing."""
    return User("Alice", "alice@example.com")

def test_user_name(sample_user):
    assert sample_user.name == "Alice"

def test_user_email(sample_user):
    assert sample_user.email == "alice@example.com"
```

### Fixture Scope

```python
# Function scope (default): Run for each test
@pytest.fixture
def user():
    return User("Alice", "alice@example.com")

# Class scope: Run once per test class
@pytest.fixture(scope="class")
def database():
    db = Database()
    db.connect()
    yield db
    db.disconnect()

# Module scope: Run once per module
@pytest.fixture(scope="module")
def app():
    app = create_app()
    return app

# Session scope: Run once per test session
@pytest.fixture(scope="session")
def browser():
    browser = Browser()
    yield browser
    browser.close()
```

### Fixture Teardown

```python
# Using yield for setup/teardown
@pytest.fixture
def temp_file():
    # Setup
    file = open('temp.txt', 'w')
    file.write('test data')
    file.close()

    # Provide fixture value
    yield 'temp.txt'

    # Teardown
    os.remove('temp.txt')

def test_file_exists(temp_file):
    assert os.path.exists(temp_file)
```

### Fixture Dependencies

```python
@pytest.fixture
def database():
    db = Database()
    db.connect()
    yield db
    db.disconnect()

@pytest.fixture
def user_repository(database):
    return UserRepository(database)

@pytest.fixture
def user_service(user_repository):
    return UserService(user_repository)

def test_create_user(user_service):
    user = user_service.create_user("Alice", "alice@example.com")
    assert user.name == "Alice"
```

### conftest.py

```python
# tests/conftest.py
# Fixtures available to all tests in directory and subdirectories

import pytest

@pytest.fixture
def sample_user():
    return User("Alice", "alice@example.com")

@pytest.fixture
def database():
    db = Database()
    db.connect()
    yield db
    db.disconnect()

# All tests can now use these fixtures
```

## Parameterized Tests

### Basic Parameterization

```python
import pytest

@pytest.mark.parametrize("a, b, expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (100, 200, 300),
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```

### Multiple Parameters

```python
@pytest.mark.parametrize("input,expected", [
    ("hello", 5),
    ("", 0),
    ("a", 1),
    ("test string", 11),
])
def test_string_length(input, expected):
    assert len(input) == expected
```

### Combining Parameters

```python
@pytest.mark.parametrize("x", [0, 1, 2])
@pytest.mark.parametrize("y", [10, 20])
def test_combinations(x, y):
    # Runs 6 tests (3 * 2)
    assert x < y
```

### Parameterizing with IDs

```python
@pytest.mark.parametrize("input,expected", [
    ("hello", 5),
    ("", 0),
    ("test", 4),
], ids=["normal", "empty", "short"])
def test_length(input, expected):
    assert len(input) == expected

# Output:
# test_length[normal] PASSED
# test_length[empty] PASSED
# test_length[short] PASSED
```

## Test Isolation

### Independent Tests

```python
# Bad: Shared state
class TestUserService:
    user_service = UserService()  # Shared!

    def test_create_user(self):
        self.user_service.create_user("Alice", "alice@example.com")
        assert len(self.user_service.users) == 1

    def test_delete_user(self):
        # Depends on test_create_user
        self.user_service.delete_user(0)
        assert len(self.user_service.users) == 0

# Good: Fresh state
class TestUserService:
    def test_create_user(self):
        service = UserService()
        service.create_user("Alice", "alice@example.com")
        assert len(service.users) == 1

    def test_delete_user(self):
        service = UserService()
        service.create_user("Alice", "alice@example.com")
        service.delete_user(0)
        assert len(service.users) == 0
```

### Using Fixtures for Isolation

```python
@pytest.fixture
def user_service():
    """Fresh UserService for each test."""
    return UserService()

def test_create_user(user_service):
    user_service.create_user("Alice", "alice@example.com")
    assert len(user_service.users) == 1

def test_delete_user(user_service):
    user_service.create_user("Alice", "alice@example.com")
    user_service.delete_user(0)
    assert len(user_service.users) == 0
```

## Mocking Dependencies

### unittest.mock

```python
from unittest.mock import Mock, patch

# Mock object
def test_send_email():
    email_service = Mock()
    email_service.send.return_value = True

    result = email_service.send("user@example.com", "Hello")

    assert result is True
    email_service.send.assert_called_once_with("user@example.com", "Hello")
```

### Patching

```python
# patch decorator
@patch('module.external_api_call')
def test_with_mock_api(mock_api):
    mock_api.return_value = {'status': 'success'}

    result = function_that_calls_api()

    assert result['status'] == 'success'
    mock_api.assert_called_once()

# patch context manager
def test_with_mock_api():
    with patch('module.external_api_call') as mock_api:
        mock_api.return_value = {'status': 'success'}

        result = function_that_calls_api()

        assert result['status'] == 'success'
```

### Mock Side Effects

```python
# Return different values on successive calls
mock = Mock()
mock.side_effect = [1, 2, 3]

assert mock() == 1
assert mock() == 2
assert mock() == 3

# Raise exception
mock = Mock()
mock.side_effect = ValueError("Error!")

with pytest.raises(ValueError):
    mock()
```

## Assertions

**Assertion**: A statement that checks if a condition is true; if false, the test fails. Used to verify expected behavior in tests.

### pytest Assertions

```python
# Basic assertions
assert value == expected
assert value != other
assert value > 0
assert value in collection
assert value is None

# Boolean
assert condition
assert not condition

# Collections
assert len(items) == 3
assert 'key' in dictionary
assert item in list

# Exceptions
with pytest.raises(ValueError):
    function_that_raises()

with pytest.raises(ValueError, match="Invalid"):
    function_that_raises()
```

### unittest Assertions

```python
class TestExample(unittest.TestCase):
    def test_equality(self):
        self.assertEqual(value, expected)
        self.assertNotEqual(value, other)

    def test_boolean(self):
        self.assertTrue(condition)
        self.assertFalse(condition)

    def test_none(self):
        self.assertIsNone(value)
        self.assertIsNotNone(value)

    def test_collections(self):
        self.assertIn(item, collection)
        self.assertNotIn(item, collection)

    def test_exceptions(self):
        with self.assertRaises(ValueError):
            function_that_raises()
```

### Floating Point Comparisons

```python
# Bad: Direct comparison
assert 0.1 + 0.2 == 0.3  # May fail!

# Good: Use approx
import pytest
assert 0.1 + 0.2 == pytest.approx(0.3)

# With tolerance
assert value == pytest.approx(expected, abs=0.01)
assert value == pytest.approx(expected, rel=0.01)
```

### Custom Assertions

```python
def assert_user_valid(user):
    """Custom assertion for user validation."""
    assert user is not None, "User is None"
    assert user.name, "User has no name"
    assert user.email, "User has no email"
    assert '@' in user.email, f"Invalid email: {user.email}"

def test_user():
    user = create_user("Alice", "alice@example.com")
    assert_user_valid(user)
```

## Summary

Unit testing in Python:

**Frameworks:**

- pytest: Modern, concise, powerful
- unittest: Built-in, verbose, class-based

**Test discovery:**

- Naming conventions
- Directory structure
- Run specific tests

**Fixtures:**

- Setup/teardown
- Scopes (function, class, module, session)
- Fixture dependencies
- conftest.py for shared fixtures

**Parameterization:**

- Multiple test cases from one function
- Clear test IDs
- Reduces duplication

**Isolation:**

- Independent tests
- Fresh state for each test
- No shared state

**Mocking:**

- Mock external dependencies
- unittest.mock module
- Patching and side effects

**Assertions:**

- Simple assert in pytest
- Rich assertions in unittest
- Floating point comparisons

## Next Steps

- Explore [Testing Strategies](testing-strategies.md)
- Master [Mocking and Fixtures](mocking-and-fixtures.md)
- Try [Advanced Testing](advanced-testing.md)
