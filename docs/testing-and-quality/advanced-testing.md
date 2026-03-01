# Advanced Testing

## Integration Testing

### Testing Components Together

```python
# Unit test: Test in isolation
def test_user_service_create():
    mock_repo = Mock()
    service = UserService(mock_repo)
    service.create_user("Alice", "alice@example.com")
    mock_repo.save.assert_called_once()

# Integration test: Test with real database
def test_user_service_integration(db_session):
    repo = UserRepository(db_session)
    service = UserService(repo)

    user = service.create_user("Alice", "alice@example.com")

    # Verify in database
    retrieved = service.get_user(user.id)
    assert retrieved.name == "Alice"
    assert retrieved.email == "alice@example.com"
```

### API Integration Tests

```python
from fastapi.testclient import TestClient
from myapp import app

client = TestClient(app)

def test_create_user_endpoint():
    response = client.post(
        "/users",
        json={"name": "Alice", "email": "alice@example.com"}
    )
    assert response.status_code == 201
    data = response.json()
    assert data["name"] == "Alice"
    assert "id" in data

def test_get_user_endpoint():
    # Create user
    create_response = client.post(
        "/users",
        json={"name": "Alice", "email": "alice@example.com"}
    )
    user_id = create_response.json()["id"]

    # Get user
    response = client.get(f"/users/{user_id}")
    assert response.status_code == 200
    assert response.json()["name"] == "Alice"

def test_user_not_found():
    response = client.get("/users/999999")
    assert response.status_code == 404
```

### Testing External Services

```python
import pytest
import requests

@pytest.mark.integration
def test_external_api():
    # Skip if API not available
    try:
        response = requests.get("https://api.example.com/health", timeout=1)
    except requests.exceptions.RequestException:
        pytest.skip("External API not available")

    # Test actual integration
    response = requests.get("https://api.example.com/users/1")
    assert response.status_code == 200
    assert "name" in response.json()

# Run only integration tests:
# pytest -m integration
```

### Contract Testing

```python
# Define expected contract
USER_SCHEMA = {
    "type": "object",
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"},
        "email": {"type": "string", "format": "email"}
    },
    "required": ["id", "name", "email"]
}

# Test provider conforms to contract
def test_api_contract():
    response = client.get("/users/1")
    data = response.json()

    from jsonschema import validate
    validate(instance=data, schema=USER_SCHEMA)

# Test consumer can handle contract
def test_consumer_contract():
    # Mock response following contract
    mock_response = {
        "id": 1,
        "name": "Alice",
        "email": "alice@example.com"
    }

    user = parse_user_response(mock_response)
    assert user.name == "Alice"
```

## Property-Based Testing

**Property-based testing**: A testing approach that automatically generates many test inputs to verify that code properties always hold true (e.g., "reversing a list twice returns original").

### Hypothesis Basics

```python
# Install: pip install hypothesis
from hypothesis import given
import hypothesis.strategies as st

# Test properties that should always hold
@given(st.integers(), st.integers())
def test_addition_commutative(a, b):
    assert a + b == b + a

@given(st.lists(st.integers()))
def test_reverse_twice(lst):
    assert list(reversed(list(reversed(lst)))) == lst

@given(st.text())
def test_string_length(s):
    assert len(s) >= 0
```

### Strategies

```python
from hypothesis import given
import hypothesis.strategies as st

# Basic strategies
@given(st.integers(min_value=0, max_value=100))
def test_with_bounded_int(n):
    assert 0 <= n <= 100

@given(st.floats(min_value=0.0, max_value=1.0))
def test_with_float(x):
    assert 0.0 <= x <= 1.0

@given(st.text(min_size=1, max_size=10))
def test_with_text(s):
    assert 1 <= len(s) <= 10

# Composite strategies
@given(st.lists(st.integers(), min_size=1, max_size=10))
def test_with_list(lst):
    assert 1 <= len(lst) <= 10

# Custom strategies
@st.composite
def user_strategy(draw):
    name = draw(st.text(min_size=1, max_size=50))
    age = draw(st.integers(min_value=0, max_value=120))
    return {"name": name, "age": age}

@given(user_strategy())
def test_with_user(user):
    assert len(user["name"]) > 0
    assert 0 <= user["age"] <= 120
```

### Finding Edge Cases

```python
from hypothesis import given, example
import hypothesis.strategies as st

# Hypothesis finds edge cases automatically
@given(st.lists(st.integers()))
def test_sort_idempotent(lst):
    once = sorted(lst)
    twice = sorted(once)
    assert once == twice
    # Hypothesis will test: [], [1], [1,1], negative numbers, etc.

# Add explicit examples
@given(st.integers())
@example(0)
@example(-1)
@example(2**31 - 1)
def test_abs_positive(n):
    assert abs(n) >= 0
```

### Stateful Testing

```python
from hypothesis.stateful import RuleBasedStateMachine, rule
import hypothesis.strategies as st

class StackMachine(RuleBasedStateMachine):
    def __init__(self):
        super().__init__()
        self.stack = []

    @rule(value=st.integers())
    def push(self, value):
        self.stack.append(value)

    @rule()
    def pop(self):
        if self.stack:
            self.stack.pop()

    @rule()
    def check_invariant(self):
        # Stack should never be negative length
        assert len(self.stack) >= 0

# Run stateful test
TestStack = StackMachine.TestCase
```

## Testing Async Code

### Async Test Functions

```python
import pytest
import asyncio

# Mark test as async
@pytest.mark.asyncio
async def test_async_function():
    result = await async_add(2, 3)
    assert result == 5

@pytest.mark.asyncio
async def test_async_api_call():
    async with AsyncClient() as client:
        response = await client.get("/users/1")
        assert response.status_code == 200
```

### Testing Coroutines

```python
import asyncio

async def fetch_data(url):
    await asyncio.sleep(0.1)  # Simulate API call
    return {"url": url, "data": "result"}

@pytest.mark.asyncio
async def test_fetch_data():
    result = await fetch_data("https://example.com")
    assert result["url"] == "https://example.com"
    assert "data" in result

# Test multiple concurrent calls
@pytest.mark.asyncio
async def test_concurrent_fetches():
    urls = ["https://example.com/1", "https://example.com/2"]
    results = await asyncio.gather(*[fetch_data(url) for url in urls])
    assert len(results) == 2
```

### Mocking Async Functions

```python
from unittest.mock import AsyncMock, patch

@pytest.mark.asyncio
async def test_with_async_mock():
    mock = AsyncMock(return_value={"status": "success"})
    result = await mock()
    assert result["status"] == "success"
    mock.assert_awaited_once()

@pytest.mark.asyncio
@patch('mymodule.async_api_call', new_callable=AsyncMock)
async def test_with_patched_async(mock_api):
    mock_api.return_value = {"data": "test"}

    result = await my_async_function()

    mock_api.assert_awaited_once()
    assert result["data"] == "test"
```

### Testing Event Loops

```python
@pytest.fixture
def event_loop():
    loop = asyncio.new_event_loop()
    yield loop
    loop.close()

@pytest.mark.asyncio
async def test_with_custom_loop(event_loop):
    task = event_loop.create_task(async_function())
    result = await task
    assert result is not None
```

### Async Fixtures

```python
@pytest.fixture
async def async_client():
    client = AsyncClient()
    await client.connect()
    yield client
    await client.disconnect()

@pytest.mark.asyncio
async def test_with_async_fixture(async_client):
    response = await async_client.get("/users/1")
    assert response.status_code == 200
```

## Database Testing

### Test Database Setup

```python
import pytest
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

@pytest.fixture(scope="session")
def engine():
    # Use in-memory SQLite for tests
    engine = create_engine("sqlite:///:memory:")
    Base.metadata.create_all(engine)
    yield engine
    engine.dispose()

@pytest.fixture
def db_session(engine):
    Session = sessionmaker(bind=engine)
    session = Session()
    yield session
    session.rollback()
    session.close()
```

### Testing Queries

```python
def test_create_user(db_session):
    user = User(name="Alice", email="alice@example.com")
    db_session.add(user)
    db_session.commit()

    assert user.id is not None

def test_query_user(db_session):
    # Setup
    user = User(name="Alice", email="alice@example.com")
    db_session.add(user)
    db_session.commit()

    # Test
    result = db_session.query(User).filter_by(name="Alice").first()
    assert result is not None
    assert result.email == "alice@example.com"

def test_delete_user(db_session):
    user = User(name="Alice", email="alice@example.com")
    db_session.add(user)
    db_session.commit()
    user_id = user.id

    db_session.delete(user)
    db_session.commit()

    result = db_session.query(User).filter_by(id=user_id).first()
    assert result is None
```

### Testing Relationships

```python
def test_user_posts_relationship(db_session):
    user = User(name="Alice")
    post1 = Post(title="Post 1", content="Content 1")
    post2 = Post(title="Post 2", content="Content 2")

    user.posts.append(post1)
    user.posts.append(post2)

    db_session.add(user)
    db_session.commit()

    # Verify relationship
    retrieved_user = db_session.query(User).filter_by(name="Alice").first()
    assert len(retrieved_user.posts) == 2
    assert retrieved_user.posts[0].title == "Post 1"
```

### Database Migrations

```python
def test_migration_up():
    # Apply migration
    alembic_command.upgrade(alembic_config, "head")

    # Verify schema changes
    inspector = inspect(engine)
    tables = inspector.get_table_names()
    assert "users" in tables

    columns = [col["name"] for col in inspector.get_columns("users")]
    assert "email" in columns

def test_migration_down():
    # Apply and revert migration
    alembic_command.upgrade(alembic_config, "head")
    alembic_command.downgrade(alembic_config, "-1")

    # Verify rollback
    inspector = inspect(engine)
    tables = inspector.get_table_names()
    # Check expected state after downgrade
```

## Performance Testing

### Timing Tests

```python
import time

def test_function_performance():
    start = time.time()

    result = expensive_function()

    duration = time.time() - start
    assert duration < 1.0  # Should complete in under 1 second
    assert result is not None

# Using pytest-benchmark
def test_benchmark(benchmark):
    result = benchmark(expensive_function)
    assert result is not None
    # Benchmark automatically measures and reports timing
```

### Memory Usage

```python
import sys

def test_memory_usage():
    data = create_large_dataset()
    size = sys.getsizeof(data)
    assert size < 1024 * 1024  # Less than 1 MB

# Using memory_profiler
from memory_profiler import profile

@profile
def test_memory_with_profiler():
    data = []
    for i in range(10000):
        data.append(i)
    # View memory usage in output
```

### Load Testing

```python
import concurrent.futures

def test_concurrent_requests():
    def make_request():
        return api_call()

    # Test with 100 concurrent requests
    with concurrent.futures.ThreadPoolExecutor(max_workers=100) as executor:
        futures = [executor.submit(make_request) for _ in range(100)]
        results = [f.result() for f in concurrent.futures.as_completed(futures)]

    assert len(results) == 100
    assert all(r.status == "success" for r in results)

# Using locust for load testing
from locust import HttpUser, task, between

class WebsiteUser(HttpUser):
    wait_time = between(1, 2)

    @task
    def get_users(self):
        self.client.get("/users")

    @task(3)  # 3x weight
    def get_user(self):
        self.client.get("/users/1")

# Run: locust -f locustfile.py
```

### Performance Regression Tests

```python
import pytest

@pytest.mark.benchmark
def test_search_performance(benchmark):
    data = create_test_data(10000)

    result = benchmark(search_function, data, "target")

    # Assert functional correctness
    assert result is not None

    # Benchmark plugin tracks performance over time
    # and can detect regressions

# Run with: pytest --benchmark-only
```

## Summary

Advanced testing techniques:

**Integration testing:**

- Test components together
- API integration tests
- External service testing
- Contract testing

**Property-based testing:**

- Hypothesis library
- Test properties, not examples
- Automatic edge case discovery
- Stateful testing

**Async testing:**

- @pytest.mark.asyncio
- Testing coroutines
- AsyncMock for mocking
- Async fixtures

**Database testing:**

- Test database setup (in-memory)
- Transaction rollback
- Testing queries and relationships
- Migration testing

**Performance testing:**

- Timing tests
- Memory usage
- Load testing
- Regression detection

Advanced tests ensure robustness, catch edge cases, and verify system behavior under various conditions.

## Next Steps

- Improve with [Code Quality Tools](code-quality-tools.md)
- Master [Type Checking](type-checking.md)
- Debug effectively with [Debugging Techniques](debugging-techniques.md)
