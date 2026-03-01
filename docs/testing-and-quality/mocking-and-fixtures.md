# Mocking and Fixtures

## Test Doubles

**Test double**: A generic term for any fake object used in place of a real one during testing (includes dummies, stubs, fakes, mocks, and spies).

### Types of Test Doubles

```python
# Dummy: Passed but never used
def test_with_dummy():
    dummy_logger = None  # Not actually used
    service = Service(dummy_logger)
    service.do_something()

# Stub: Returns predetermined values
class StubDatabase:
    def get_user(self, user_id):
        return User(id=1, name="Alice")  # Always same response

# Fake: Working implementation, but simplified
class FakeDatabase:
    def __init__(self):
        self.users = {}

    def save(self, user):
        self.users[user.id] = user

    def get(self, user_id):
        return self.users.get(user_id)

# Mock: Verifies calls and interactions
from unittest.mock import Mock

mock_email = Mock()
service.send_welcome_email(user)
mock_email.send.assert_called_once_with(to=user.email)

# Spy: Records calls while maintaining real behavior
class SpyLogger:
    def __init__(self, real_logger):
        self.real_logger = real_logger
        self.calls = []

    def log(self, message):
        self.calls.append(message)
        self.real_logger.log(message)
```

### When to Use Each

```python
# Stub: When you need fixed responses
def test_user_service_with_stub():
    stub_repo = StubUserRepository()  # Returns fixed users
    service = UserService(stub_repo)
    user = service.get_user(1)
    assert user.name == "Alice"

# Fake: When you need realistic behavior
def test_with_fake_database():
    fake_db = FakeDatabase()
    service = UserService(fake_db)

    user = User(id=1, name="Alice")
    service.create_user(user)

    retrieved = service.get_user(1)
    assert retrieved.name == "Alice"

# Mock: When you need to verify interactions
def test_email_sent_on_registration():
    mock_email = Mock()
    service = UserService(email_service=mock_email)

    service.register_user("alice@example.com")

    mock_email.send_welcome.assert_called_once()
```

## unittest.mock Deep Dive

### Mock Objects

```python
from unittest.mock import Mock

# Basic mock
mock = Mock()
mock.some_method()  # Can call any method
mock.some_method.assert_called_once()

# Mock with return value
mock = Mock(return_value=42)
result = mock()
assert result == 42

# Mock with side effect (function)
def side_effect_func(x):
    return x * 2

mock = Mock(side_effect=side_effect_func)
assert mock(5) == 10

# Mock with side effect (list of values)
mock = Mock(side_effect=[1, 2, 3])
assert mock() == 1
assert mock() == 2
assert mock() == 3

# Mock with side effect (exception)
mock = Mock(side_effect=ValueError("Error!"))
with pytest.raises(ValueError):
    mock()
```

### Patching

```python
from unittest.mock import patch

# Patch a function
@patch('mymodule.external_api_call')
def test_service(mock_api):
    mock_api.return_value = {'status': 'success'}
    result = my_service.process()
    assert result == 'success'

# Patch as context manager
def test_service_context():
    with patch('mymodule.external_api_call') as mock_api:
        mock_api.return_value = {'status': 'success'}
        result = my_service.process()
        assert result == 'success'

# Patch multiple targets
@patch('mymodule.database')
@patch('mymodule.email_service')
def test_registration(mock_email, mock_db):
    # Decorators apply in reverse order
    register_user("alice@example.com")
    mock_db.save.assert_called_once()
    mock_email.send.assert_called_once()

# Patch object attribute
@patch.object(MyClass, 'method')
def test_method(mock_method):
    mock_method.return_value = 42
    obj = MyClass()
    assert obj.method() == 42
```

### MagicMock

```python
from unittest.mock import MagicMock

# MagicMock supports magic methods
mock = MagicMock()
mock.__len__.return_value = 5
assert len(mock) == 5

mock.__getitem__.return_value = 'item'
assert mock[0] == 'item'

# Context manager support
mock = MagicMock()
with mock as ctx:
    pass
mock.__enter__.assert_called_once()
mock.__exit__.assert_called_once()

# Iterator support
mock = MagicMock()
mock.__iter__.return_value = iter([1, 2, 3])
assert list(mock) == [1, 2, 3]
```

### Verifying Calls

```python
from unittest.mock import call

mock = Mock()

# Call tracking
mock.method(1, 2, key='value')
mock.method.assert_called_with(1, 2, key='value')
mock.method.assert_called_once_with(1, 2, key='value')

# Check if called at all
mock.other_method()
assert mock.other_method.called
assert mock.other_method.call_count == 1

# Check call arguments
mock.method(1, 2, 3)
args, kwargs = mock.method.call_args
assert args == (1, 2, 3)

# Check all calls
mock.method(1)
mock.method(2)
mock.method(3)
assert mock.method.call_args_list == [call(1), call(2), call(3)]

# Any order
mock.method(1)
mock.method(2)
mock.method.assert_any_call(1)
mock.method.assert_any_call(2)
```

### Mock Specifications

```python
# Spec: Mock must match interface
class RealClass:
    def real_method(self):
        pass

mock = Mock(spec=RealClass)
mock.real_method()  # OK
mock.fake_method()  # Raises AttributeError

# Spec_set: Stricter, prevents setting attributes
mock = Mock(spec_set=RealClass)
mock.new_attribute = 42  # Raises AttributeError

# Create autospec from real object
from unittest.mock import create_autospec

def real_function(x, y):
    return x + y

mock_func = create_autospec(real_function)
mock_func(1, 2)  # OK
mock_func(1)  # TypeError: missing required argument
```

## Pytest Fixtures Advanced

### Fixture Scope

```python
import pytest

# Function scope (default): Run for each test
@pytest.fixture
def user():
    return User("Alice")

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
    yield app
    app.cleanup()

# Session scope: Run once per test session
@pytest.fixture(scope="session")
def test_data():
    return load_test_data()
```

### Fixture Factories

```python
# Fixture that returns a factory function
@pytest.fixture
def user_factory():
    created_users = []

    def _create_user(name="Alice", email="alice@example.com"):
        user = User(name, email)
        created_users.append(user)
        return user

    yield _create_user

    # Cleanup all created users
    for user in created_users:
        user.delete()

# Usage
def test_multiple_users(user_factory):
    alice = user_factory(name="Alice")
    bob = user_factory(name="Bob")
    assert alice.name != bob.name
```

### Parametrized Fixtures

```python
# Parametrize fixtures
@pytest.fixture(params=["sqlite", "postgres", "mysql"])
def database(request):
    db = Database(request.param)
    db.connect()
    yield db
    db.disconnect()

# Test runs 3 times with different databases
def test_query(database):
    result = database.query("SELECT 1")
    assert result is not None

# Combine with marks
@pytest.fixture(params=[
    pytest.param("sqlite", marks=pytest.mark.fast),
    pytest.param("postgres", marks=pytest.mark.slow),
])
def database(request):
    return Database(request.param)
```

### Fixture Autouse

```python
# Automatically use fixture for all tests
@pytest.fixture(autouse=True)
def reset_database():
    database.clear()
    yield
    # Cleanup happens automatically

def test_something():
    # reset_database runs automatically
    pass

# Autouse with scope
@pytest.fixture(autouse=True, scope="module")
def setup_module():
    print("Setup module")
    yield
    print("Teardown module")
```

### Fixture Dependencies

```python
# Fixtures can depend on other fixtures
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

# Test depends on entire chain
def test_create_user(user_service):
    user = user_service.create("Alice", "alice@example.com")
    assert user.name == "Alice"
```

### conftest.py Organization

```python
# tests/conftest.py (root level - available to all tests)
@pytest.fixture
def app():
    return create_app()

# tests/unit/conftest.py (available to unit tests only)
@pytest.fixture
def mock_database():
    return Mock(spec=Database)

# tests/integration/conftest.py (available to integration tests only)
@pytest.fixture
def real_database():
    db = Database()
    db.connect()
    yield db
    db.disconnect()

# Fixtures from parent conftest.py are available in subdirectories
```

## Test Isolation

### Avoiding Shared State

```python
# BAD: Shared mutable state
shared_list = []

def test_append():
    shared_list.append(1)
    assert len(shared_list) == 1  # Fails if test_remove runs first

def test_remove():
    shared_list.clear()
    assert len(shared_list) == 0  # Depends on order

# GOOD: Fresh state per test
@pytest.fixture
def my_list():
    return []

def test_append(my_list):
    my_list.append(1)
    assert len(my_list) == 1

def test_remove(my_list):
    my_list.clear()
    assert len(my_list) == 0
```

### Resetting Singletons

```python
# Singleton pattern
class Cache:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance.data = {}
        return cls._instance

# Reset between tests
@pytest.fixture(autouse=True)
def reset_cache():
    Cache._instance = None
    yield
    Cache._instance = None

def test_cache_isolated():
    cache = Cache()
    cache.data['key'] = 'value'
    assert cache.data['key'] == 'value'
```

### Database Transactions

```python
# Rollback after each test
@pytest.fixture
def db_session():
    session = database.create_session()
    session.begin()
    yield session
    session.rollback()
    session.close()

def test_create_user(db_session):
    user = User(name="Alice")
    db_session.add(user)
    db_session.commit()  # Committed to session
    # But rolled back after test

def test_user_not_found(db_session):
    # Fresh database state
    user = db_session.query(User).filter_by(name="Alice").first()
    assert user is None  # Previous test's user was rolled back
```

### Temporary Files

```python
import tempfile
import os

@pytest.fixture
def temp_file():
    fd, path = tempfile.mkstemp()
    yield path
    os.close(fd)
    os.unlink(path)

def test_file_write(temp_file):
    with open(temp_file, 'w') as f:
        f.write("test")

    with open(temp_file, 'r') as f:
        assert f.read() == "test"
    # File deleted after test

# Or use pytest's tmp_path fixture
def test_with_tmp_path(tmp_path):
    file_path = tmp_path / "test.txt"
    file_path.write_text("test")
    assert file_path.read_text() == "test"
    # tmp_path is automatically cleaned up
```

## Mocking External Dependencies

### HTTP Requests

```python
# Using responses library
import responses
import requests

@responses.activate
def test_api_call():
    responses.add(
        responses.GET,
        'https://api.example.com/users/1',
        json={'id': 1, 'name': 'Alice'},
        status=200
    )

    response = requests.get('https://api.example.com/users/1')
    assert response.json()['name'] == 'Alice'

# Using requests-mock
import requests_mock

def test_api_with_mock():
    with requests_mock.Mocker() as m:
        m.get('https://api.example.com/users/1',
              json={'id': 1, 'name': 'Alice'})

        response = requests.get('https://api.example.com/users/1')
        assert response.json()['name'] == 'Alice'
```

### Database Operations

```python
# Mock database calls
@patch('mymodule.database.query')
def test_user_service(mock_query):
    mock_query.return_value = [{'id': 1, 'name': 'Alice'}]

    service = UserService()
    users = service.get_all_users()

    assert len(users) == 1
    assert users[0]['name'] == 'Alice'

# Or use fake database
class FakeDatabase:
    def __init__(self):
        self.users = []

    def insert(self, user):
        self.users.append(user)

    def query(self, condition):
        return [u for u in self.users if condition(u)]

@pytest.fixture
def fake_db():
    return FakeDatabase()

def test_with_fake_db(fake_db):
    service = UserService(fake_db)
    service.create_user(name="Alice")
    users = service.get_all_users()
    assert len(users) == 1
```

### File System

```python
# Mock file operations
@patch('builtins.open', create=True)
def test_read_file(mock_open):
    mock_open.return_value.__enter__.return_value.read.return_value = "content"

    result = read_file("path/to/file.txt")
    assert result == "content"

# Or use tmp_path for real files
def test_read_file_real(tmp_path):
    file_path = tmp_path / "test.txt"
    file_path.write_text("content")

    result = read_file(str(file_path))
    assert result == "content"
```

### Time and Dates

```python
from unittest.mock import patch
from datetime import datetime

# Mock datetime
@patch('mymodule.datetime')
def test_current_time(mock_datetime):
    mock_datetime.now.return_value = datetime(2024, 1, 1, 12, 0, 0)

    result = get_current_timestamp()
    assert result == "2024-01-01 12:00:00"

# Using freezegun
from freezegun import freeze_time

@freeze_time("2024-01-01 12:00:00")
def test_with_frozen_time():
    assert datetime.now().year == 2024
    assert datetime.now().month == 1
```

### Environment Variables

```python
@patch.dict('os.environ', {'API_KEY': 'test_key'})
def test_with_env_var():
    assert os.environ['API_KEY'] == 'test_key'

# Using monkeypatch fixture
def test_with_monkeypatch(monkeypatch):
    monkeypatch.setenv('API_KEY', 'test_key')
    assert os.environ['API_KEY'] == 'test_key'
    # Automatically restored after test
```

## Summary

Test doubles:

**Types:**

- Dummy: Unused placeholders
- Stub: Predetermined responses
- Fake: Simplified working implementation
- Mock: Verifies interactions
- Spy: Records calls while preserving behavior

**unittest.mock:**

- Mock objects with return_value and side_effect
- @patch decorator and context manager
- MagicMock for magic methods
- Verify calls with assert*called*\*
- Spec and autospec for type safety

**Pytest fixtures:**

- Scope: function, class, module, session
- Factories for flexible test data
- Parametrize fixtures for multiple scenarios
- Autouse for automatic setup
- Dependencies create fixture chains
- conftest.py for shared fixtures

**Test isolation:**

- Fresh state per test
- Reset singletons
- Database transactions
- Temporary files
- Avoid shared mutable state

**Mocking external dependencies:**

- HTTP requests (responses, requests-mock)
- Database operations
- File system (tmp_path)
- Time and dates (freezegun)
- Environment variables (monkeypatch)

Good mocking isolates tests from external dependencies while maintaining realistic behavior.

## Next Steps

- Learn [Advanced Testing](advanced-testing.md)
- Improve with [Code Quality Tools](code-quality-tools.md)
- Master [Type Checking](type-checking.md)
