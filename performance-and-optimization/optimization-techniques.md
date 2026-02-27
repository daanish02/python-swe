# Optimization Techniques

## Table of Contents

- [Caching and Memoization](#caching-and-memoization)
- [Using functools.lru_cache](#using-functoolslru_cache)
- [Generators and Lazy Evaluation](#generators-and-lazy-evaluation)
- [List Comprehensions vs Loops](#list-comprehensions-vs-loops)
- [Avoiding Unnecessary Work](#avoiding-unnecessary-work)
- [String Operations](#string-operations)
- [Common Python Patterns](#common-python-patterns)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Caching and Memoization

**Memoization**: Storing function results to avoid recomputing with same inputs.

### Manual Caching

```python
# Without caching: Slow for repeated calculations
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# fibonacci(35) takes ~5 seconds
# Recalculates same values many times

# With manual caching
_fib_cache = {}

def fibonacci_cached(n):
    if n in _fib_cache:
        return _fib_cache[n]

    if n < 2:
        result = n
    else:
        result = fibonacci_cached(n-1) + fibonacci_cached(n-2)

    _fib_cache[n] = result
    return result

# fibonacci_cached(35) takes <0.001 seconds
# 1000x+ faster!
```

### When to Cache

```python
# Cache when:
# - Function is called repeatedly with same inputs
# - Function is expensive (slow computation, I/O, API calls)
# - Results don't change (pure function)

# Good candidate: Expensive calculation
def calculate_distance(point1, point2):
    # Complex trigonometric calculations
    return expensive_math_operation(point1, point2)

# Good candidate: API call
def fetch_user_data(user_id):
    # Network request
    return requests.get(f'/api/users/{user_id}').json()

# Bad candidate: Random results
def get_random_number():
    return random.randint(1, 100)  # Should NOT be cached

# Bad candidate: Changes frequently
def get_current_time():
    return datetime.now()  # Should NOT be cached
```

## Using functools.lru_cache

**lru_cache**: Built-in decorator for automatic memoization with LRU (Least Recently Used) eviction.

### Basic Usage

```python
from functools import lru_cache

# Without cache
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# fibonacci(35): ~5 seconds

# With lru_cache (just add decorator!)
@lru_cache
def fibonacci_fast(n):
    if n < 2:
        return n
    return fibonacci_fast(n-1) + fibonacci_fast(n-2)

# fibonacci_fast(35): <0.001 seconds
```

### Cache Size

```python
from functools import lru_cache

# Unlimited cache
@lru_cache(maxsize=None)
def expensive_function(x):
    return x ** 2

# Limited cache (default: 128)
@lru_cache(maxsize=100)
def fetch_data(key):
    # Only keeps 100 most recent results
    # Evicts least recently used when full
    return database.get(key)

# Small cache for memory-constrained
@lru_cache(maxsize=10)
def transform(data):
    return process_large_data(data)
```

### Cache Statistics

```python
from functools import lru_cache

@lru_cache(maxsize=100)
def expensive_calculation(n):
    return n ** 2

# Use the function
for i in range(200):
    expensive_calculation(i % 50)  # Repeats values

# Check cache performance
print(expensive_calculation.cache_info())
# CacheInfo(hits=150, misses=50, maxsize=100, currsize=50)

# hits: Number of cache hits (150)
# misses: Number of cache misses (50)
# maxsize: Maximum cache size (100)
# currsize: Current cache size (50)

# Clear cache if needed
expensive_calculation.cache_clear()
```

### Real-World Example

```python
from functools import lru_cache
import requests

# Cache API responses
@lru_cache(maxsize=100)
def fetch_user(user_id):
    """Fetch user data from API"""
    response = requests.get(f'https://api.example.com/users/{user_id}')
    return response.json()

# First call: Makes API request
user1 = fetch_user(123)  # Slow (network request)

# Subsequent calls: Returns cached result
user2 = fetch_user(123)  # Instant (from cache)
user3 = fetch_user(123)  # Instant (from cache)

# Different argument: New API request
user4 = fetch_user(456)  # Slow (network request)

print(fetch_user.cache_info())
# CacheInfo(hits=2, misses=2, maxsize=100, currsize=2)
```

### Caching with Multiple Arguments

```python
from functools import lru_cache

@lru_cache(maxsize=100)
def calculate_distance(x1, y1, x2, y2):
    """Calculate Euclidean distance"""
    return ((x2 - x1)**2 + (y2 - y1)**2) ** 0.5

# Works with multiple arguments
d1 = calculate_distance(0, 0, 3, 4)  # Miss: 5.0
d2 = calculate_distance(0, 0, 3, 4)  # Hit: 5.0

# Arguments must be hashable (immutable)
# Works: int, float, str, tuple
# Doesn't work: list, dict, set
```

## Generators and Lazy Evaluation

**Generator**: Function that yields values one at a time, computing on demand rather than all at once.

### Memory-Efficient Processing

```python
# Memory-intensive: Loads everything into memory
def read_large_file_list(filename):
    with open(filename) as f:
        return [line.strip() for line in f]  # All lines in memory

lines = read_large_file_list('huge_file.txt')  # Uses gigabytes!
for line in lines:
    process(line)

# Memory-efficient: Generator yields one at a time
def read_large_file_gen(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()  # Yields one line at a time

lines = read_large_file_gen('huge_file.txt')  # Minimal memory
for line in lines:
    process(line)  # Processes one line at a time
```

### Generator Expressions

```python
# List comprehension: Creates full list
squares_list = [x**2 for x in range(1000000)]  # 8+ MB memory
print(sum(squares_list))

# Generator expression: Lazy evaluation
squares_gen = (x**2 for x in range(1000000))  # <100 bytes!
print(sum(squares_gen))  # Computes on-the-fly

# Functionally equivalent, vastly different memory usage
```

### Chaining Generators

```python
# Compose generators for memory efficiency
def read_file(filename):
    with open(filename) as f:
        for line in f:
            yield line.strip()

def filter_comments(lines):
    for line in lines:
        if not line.startswith('#'):
            yield line

def parse_data(lines):
    for line in lines:
        yield parse_line(line)

# Chain generators: Minimal memory usage
pipeline = parse_data(filter_comments(read_file('data.txt')))
for item in pipeline:
    process(item)

# Processes gigabyte files with constant memory
```

### When to Use Generators

```python
# Use generators when:
# - Processing large datasets
# - Don't need all results at once
# - Results used once (streaming)

# Generator: Good for large files
def process_log_file(filename):
    with open(filename) as f:
        for line in f:
            if 'ERROR' in line:
                yield parse_error(line)

# List: Good for small, reused data
def get_menu_items():
    # Small, fixed dataset, accessed multiple times
    return ['File', 'Edit', 'View', 'Help']
```

## List Comprehensions vs Loops

### Performance Comparison

```python
import timeit

# For loop
def with_loop(n):
    result = []
    for i in range(n):
        result.append(i * 2)
    return result

# List comprehension
def with_comprehension(n):
    return [i * 2 for i in range(n)]

# Benchmark
n = 10000
loop_time = timeit.timeit(lambda: with_loop(n), number=1000)
comp_time = timeit.timeit(lambda: with_comprehension(n), number=1000)

print(f"Loop: {loop_time:.4f}s")           # ~0.85s
print(f"Comprehension: {comp_time:.4f}s")  # ~0.52s
# List comprehension is ~40% faster
```

### When to Use Each

```python
# List comprehension: Simple transformations
squares = [x**2 for x in range(10)]

# List comprehension: With filter
evens = [x for x in range(10) if x % 2 == 0]

# For loop: Complex logic
result = []
for item in items:
    if complex_condition(item):
        processed = multi_step_process(item)
        if processed:
            result.append(processed)
# Too complex for comprehension

# For loop: Side effects (printing, logging, updating state)
for user in users:
    send_email(user)  # Side effect
    log_action(user)  # Side effect
# Don't use comprehension for side effects
```

### Dictionary and Set Comprehensions

```python
# Dict comprehension
squares_dict = {x: x**2 for x in range(10)}
# {0: 0, 1: 1, 2: 4, 3: 9, ...}

# Set comprehension
unique_lengths = {len(word) for word in words}

# Faster than manual construction
result = {}
for x in range(10):
    result[x] = x**2
```

## Avoiding Unnecessary Work

### Don't Recompute

```python
# Bad: Recomputing length every iteration
def process_items(items):
    for i in range(len(items)):  # len() called many times
        if i < len(items) - 1:  # len() called again!
            process(items[i])

# Good: Compute once
def process_items(items):
    length = len(items)  # Compute once
    for i in range(length):
        if i < length - 1:
            process(items[i])

# Better: Don't need length at all
def process_items(items):
    for item in items[:-1]:  # Pythonic, no len() needed
        process(item)
```

### Short-Circuit Evaluation

```python
# Python evaluates left to right, stops when result is known

# Bad: Always checks both conditions
def is_valid(user):
    return is_authenticated(user) and has_permission(user)
# If is_authenticated() is False, still calls has_permission()

# Good: Second check skipped if first is False
def is_valid(user):
    if not is_authenticated(user):
        return False
    return has_permission(user)

# Use 'any' for OR logic
def has_any_permission(user, permissions):
    return any(user.has_permission(p) for p in permissions)
# Stops checking as soon as one is True

# Use 'all' for AND logic
def has_all_permissions(user, permissions):
    return all(user.has_permission(p) for p in permissions)
# Stops checking as soon as one is False
```

### Avoid Repeated Lookups

```python
# Bad: Repeated dict lookup
def process_user_orders(orders, users):
    for order in orders:
        user = users[order.user_id]  # Lookup
        print(f"{users[order.user_id].name}: {order.total}")  # Lookup again!

# Good: Lookup once
def process_user_orders(orders, users):
    for order in orders:
        user = users[order.user_id]  # Lookup once
        print(f"{user.name}: {order.total}")  # Use cached reference
```

## String Operations

### String Concatenation

```python
import timeit

# Bad: Repeated concatenation (creates new string each time)
def bad_concat(items):
    result = ""
    for item in items:
        result += str(item) + ", "  # New string every iteration
    return result

# Good: Join with list
def good_concat(items):
    return ", ".join(str(item) for item in items)

# Benchmark with 10000 items
items = range(10000)
bad_time = timeit.timeit(lambda: bad_concat(items), number=10)
good_time = timeit.timeit(lambda: good_concat(items), number=10)

print(f"Bad: {bad_time:.4f}s")   # ~8.5s
print(f"Good: {good_time:.4f}s")  # ~0.02s
# join() is 400x+ faster!
```

### String Formatting

```python
# Slowest: Old-style formatting
result = "User: %s, Age: %d" % (name, age)

# Faster: str.format()
result = "User: {}, Age: {}".format(name, age)

# Fastest: f-strings (Python 3.6+)
result = f"User: {name}, Age: {age}"

# f-strings also most readable and flexible
result = f"User: {user.name.upper()}, Balance: ${account.balance:.2f}"
```

### String Membership

```python
# Multiple checks: Use set for large collections
# Bad: Checking in string repeatedly
def contains_keyword(text, keywords):
    # keywords is a list
    return any(keyword in text for keyword in keywords)

# Still bad if keywords is long
# Each 'in' operation is O(n*m)

# Good: For many keywords, use regex or aho-corasick
import re

def contains_keyword(text, keywords):
    pattern = '|'.join(re.escape(k) for k in keywords)
    return bool(re.search(pattern, text))

# Much faster for many keywords
```

## Common Python Patterns

### Use Built-in Functions

```python
# Slow: Manual implementation
def sum_squares(numbers):
    total = 0
    for n in numbers:
        total += n * n
    return total

# Fast: Built-in functions (implemented in C)
def sum_squares(numbers):
    return sum(n * n for n in numbers)

# Other fast built-ins:
min(items)       # Faster than manual loop
max(items)       # Faster than manual loop
any(conditions)  # Short-circuits
all(conditions)  # Short-circuits
```

### Local Variable Lookup

```python
# Slower: Global/attribute lookup in loop
import math

def calculate_values(numbers):
    result = []
    for n in numbers:
        result.append(math.sqrt(n))  # Looks up math.sqrt every time
    return result

# Faster: Cache lookup as local variable
def calculate_values(numbers):
    sqrt = math.sqrt  # Local reference
    result = []
    for n in numbers:
        result.append(sqrt(n))  # Local lookup is faster
    return result

# Or use comprehension (automatically optimized)
def calculate_values(numbers):
    return [math.sqrt(n) for n in numbers]
```

### Set/Dict Operations

```python
# Finding duplicates
# Slow: Nested loops - O(n²)
def find_duplicates_slow(items):
    duplicates = []
    for i, item in enumerate(items):
        for j, other in enumerate(items[i+1:]):
            if item == other and item not in duplicates:
                duplicates.append(item)
    return duplicates

# Fast: Using set - O(n)
def find_duplicates_fast(items):
    seen = set()
    duplicates = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        seen.add(item)
    return list(duplicates)
```

### Using Collections Module

```python
from collections import Counter, defaultdict, deque

# Counter for counting
# Slow: Manual counting
counts = {}
for item in items:
    counts[item] = counts.get(item, 0) + 1

# Fast: Counter
counts = Counter(items)

# defaultdict for grouping
# Slow: Checking and initializing
groups = {}
for item in items:
    key = item.category
    if key not in groups:
        groups[key] = []
    groups[key].append(item)

# Fast: defaultdict
groups = defaultdict(list)
for item in items:
    groups[item.category].append(item)

# deque for queue operations
# Slow: List popleft is O(n)
queue = [1, 2, 3]
first = queue.pop(0)  # Shifts all elements!

# Fast: deque popleft is O(1)
queue = deque([1, 2, 3])
first = queue.popleft()  # No shifting!
```

## Summary

Python optimization techniques:

**Caching and memoization:**

- Use `@lru_cache` for expensive, repeated calculations
- Check `cache_info()` for hit/miss stats
- Cache pure functions (same input → same output)
- Clear cache with `cache_clear()` if needed

**Generators and lazy evaluation:**

- Use generators for large datasets
- Generator expressions: `(x for x in items)`
- Chain generators for memory efficiency
- Streaming data processing

**List comprehensions:**

- ~40% faster than equivalent loops
- Use for simple transformations
- Dict/set comprehensions also fast
- Use loops for complex logic or side effects

**Avoid unnecessary work:**

- Don't recompute values
- Use short-circuit evaluation (any/all)
- Cache lookups (especially in loops)
- Local variable access is faster

**String operations:**

- Use `str.join()` not `+=` for concatenation
- f-strings fastest for formatting
- Built-in string methods are optimized

**Common patterns:**

- Use built-in functions (implemented in C)
- Set/dict for membership testing
- Counter for counting
- defaultdict for grouping
- deque for queues

Profile first, then apply these techniques where they matter.

## Next Steps

- Learn [Memory Optimization](memory-optimization.md)
- Explore [Parallel and Concurrent Execution](parallel-and-concurrent.md)
- Review [Data Structure Performance](data-structure-performance.md)
