# Performance Fundamentals

## Table of Contents

- [When to Optimize](#when-to-optimize)
- [Premature Optimization](#premature-optimization)
- [Performance-Aware Programming](#performance-aware-programming)
- [Understanding Trade-offs](#understanding-trade-offs)
- [Measuring Before Optimizing](#measuring-before-optimizing)
- [Summary](#summary)
- [Next Steps](#next-steps)

## When to Optimize

**Performance optimization**: The process of making code run faster or use less memory while maintaining correctness.

### Optimize When It Matters

```python
# Optimize when:
# - Users experience noticeable slowness
# - Processing large datasets
# - Running frequently (hot paths)
# - Hitting resource limits (memory, CPU)
# - Meeting performance requirements

# Example: User-facing API endpoint
@app.route('/search')
def search():
    # This runs thousands of times per day
    # Users wait for response
    # Optimization matters here
    results = optimized_search(query)
    return jsonify(results)

# Example: One-time data migration
def migrate_old_data():
    # Runs once
    # No users waiting
    # Optimization not critical
    for record in old_records:
        process_and_save(record)
```

### Don't Optimize When

```python
# Don't optimize:
# - Code runs rarely
# - Already fast enough
# - Makes code unreadable
# - Without measurements

# Example: Configuration loading (once at startup)
def load_config():
    # Runs once when app starts
    # Simple, readable code is better
    with open('config.yaml') as f:
        config = yaml.safe_load(f)
    return config

# Don't make it "faster" with complex caching
# Readability > speed here

# Example: Admin script
def generate_monthly_report():
    # Runs once a month by admin
    # Taking 5 seconds vs 0.5 seconds doesn't matter
    # Keep it simple and maintainable
    data = fetch_all_data()
    report = create_report(data)
    save_report(report)
```

### The 80/20 Rule

```python
# 80% of execution time spent in 20% of code
# Focus optimization on that 20%

# Example profile output:
# Function              Calls    Time
# process_items()       100000   8.5s   ← Hot path (optimize this)
# validate_email()      1000     0.8s
# log_message()         50000    0.5s
# format_date()         1000     0.2s   ← Cold path (don't bother)

# Profile first, then optimize hot paths
```

## Premature Optimization

**Premature optimization**: Optimizing code before knowing if it's actually slow, often making code harder to maintain.

### The Classic Mistake

```python
# Bad: Premature optimization
def calculate_total(items):
    # "Optimized" with manual loop instead of sum()
    # Harder to read, minimal speed gain
    total = 0
    length = len(items)
    i = 0
    while i < length:
        total += items[i]
        i += 1
    return total

# Good: Clear, readable, fast enough
def calculate_total(items):
    return sum(item.price for item in items)

# Only optimize if profiling shows this is slow
# (It won't be - sum() is fast)
```

### Making Code Worse

```python
# Bad: Complex "optimization" without benefit
class UserCache:
    """Premature caching that adds complexity"""
    def __init__(self):
        self._cache = {}
        self._timestamps = {}
        self._access_counts = {}

    def get_user(self, user_id):
        # Complex cache logic
        # But users are fetched once per request anyway
        if user_id in self._cache:
            if time.time() - self._timestamps[user_id] < 300:
                self._access_counts[user_id] += 1
                return self._cache[user_id]

        user = db.get_user(user_id)
        self._cache[user_id] = user
        self._timestamps[user_id] = time.time()
        self._access_counts[user_id] = 1
        return user

# Good: Simple, clear, maintainable
def get_user(user_id):
    return db.get_user(user_id)

# Add caching only if measurements show it helps
```

### When Optimization Backfires

```python
# Problem: Optimized for speed, broke functionality
def fast_search(items, keyword):
    # "Optimized" by skipping validation
    # Now crashes on edge cases
    return [item for item in items if keyword in item]

# What we lost:
# - Error handling
# - Case-insensitive search
# - Empty string handling
# - Type checking

# Better: Keep it correct first
def search(items, keyword):
    if not keyword:
        return []

    keyword_lower = keyword.lower()
    return [
        item for item in items
        if keyword_lower in item.name.lower()
    ]

# Optimize only if profiling shows this is too slow
```

## Performance-Aware Programming

**Performance-aware programming**: Writing clean code that naturally performs well, without sacrificing readability.

### Choose Efficient Patterns

```python
# Inefficient: Repeatedly checking membership in list
def filter_valid(items, valid_ids):
    # O(n*m) - slow for large lists
    return [item for item in items if item.id in valid_ids]

# Efficient: Convert list to set for O(1) lookups
def filter_valid(items, valid_ids):
    # O(n+m) - much faster
    valid_set = set(valid_ids)
    return [item for item in items if item.id in valid_set]

# Still readable, naturally faster
```

### Avoid Unnecessary Work

```python
# Inefficient: Processing same data multiple times
def analyze_data(records):
    total = sum(r.value for r in records)
    average = sum(r.value for r in records) / len(records)  # Duplicate work
    maximum = max(r.value for r in records)  # Duplicate work
    return total, average, maximum

# Efficient: Process once
def analyze_data(records):
    values = [r.value for r in records]
    total = sum(values)
    return total, total / len(values), max(values)

# Or even better: Single pass
def analyze_data(records):
    values = [r.value for r in records]
    return {
        'total': sum(values),
        'average': sum(values) / len(values),
        'maximum': max(values),
    }
```

### Use Appropriate Data Structures

```python
# Inefficient: Wrong data structure
def find_user(users, user_id):
    # O(n) - linear search through list
    for user in users:
        if user.id == user_id:
            return user
    return None

# Efficient: Use dict for lookups
users_by_id = {user.id: user for user in users}

def find_user(user_id):
    # O(1) - instant lookup
    return users_by_id.get(user_id)

# Same functionality, better performance
```

## Understanding Trade-offs

### Speed vs Readability

```python
# Readable but slower
def process_items(items):
    return [transform(item) for item in items if is_valid(item)]

# Faster but less clear
def process_items(items):
    result = []
    for item in items:
        if is_valid(item):
            result.append(transform(item))
    return result

# Choose readable unless profiling shows speed matters
# Usually the speed difference is negligible
```

### Memory vs Speed

```python
# Memory-efficient: Generator (lazy)
def read_large_file(filename):
    with open(filename) as f:
        for line in f:
            yield process_line(line)

# Fast but memory-intensive: Load all
def read_large_file(filename):
    with open(filename) as f:
        return [process_line(line) for line in f]

# Choose based on data size and available memory
```

### Simplicity vs Performance

```python
# Simple: Use library
import statistics
def calculate_stats(numbers):
    return {
        'mean': statistics.mean(numbers),
        'median': statistics.median(numbers),
        'stdev': statistics.stdev(numbers),
    }

# Faster but more complex: Manual calculation
def calculate_stats(numbers):
    n = len(numbers)
    mean = sum(numbers) / n
    sorted_nums = sorted(numbers)
    median = sorted_nums[n // 2]
    variance = sum((x - mean) ** 2 for x in numbers) / n
    stdev = variance ** 0.5
    return {'mean': mean, 'median': median, 'stdev': stdev}

# Use library unless profiling shows it's too slow
```

### Development Time vs Runtime

```python
# Quick to write, might be slow
def process_data(data):
    # Simple, obvious, maintainable
    return pandas_dataframe.apply(complex_function)

# Faster runtime, slower to develop
def process_data(data):
    # Manually optimized
    # More code, harder to maintain
    # But 10x faster for large datasets
    result = numpy.empty(len(data))
    for i, value in enumerate(data):
        result[i] = optimized_calculation(value)
    return result

# Start simple, optimize if needed
```

## Measuring Before Optimizing

### Use timeit for Quick Tests

```python
import timeit

# Compare two approaches
list_approach = timeit.timeit(
    '[x for x in range(1000) if x % 2 == 0]',
    number=10000
)

filter_approach = timeit.timeit(
    'list(filter(lambda x: x % 2 == 0, range(1000)))',
    number=10000
)

print(f"List comp: {list_approach:.4f}s")
print(f"Filter: {filter_approach:.4f}s")
# List comp: 0.4523s
# Filter: 0.6891s
# List comprehension wins
```

### Profile Real Usage

```python
import cProfile
import pstats

# Profile actual code
cProfile.run('main()', 'profile_stats')

# Analyze results
stats = pstats.Stats('profile_stats')
stats.sort_stats('cumulative')
stats.print_stats(10)

# Shows where time is actually spent
# Optimize based on data, not guesses
```

### Measure With Context

```python
import time

def measure_function(func, *args):
    """Simple timing helper"""
    start = time.perf_counter()
    result = func(*args)
    elapsed = time.perf_counter() - start
    print(f"{func.__name__}: {elapsed:.4f}s")
    return result

# Test with realistic data
data = load_real_dataset()
result = measure_function(process_data, data)

# Don't optimize based on toy examples
```

## Summary

Performance optimization fundamentals:

**When to optimize:**

- User-facing code with noticeable slowness
- Frequently executed hot paths
- Processing large datasets
- Hitting resource limits
- Don't optimize rarely-run code

**Premature optimization:**

- Optimizing before knowing it's slow
- Makes code harder to maintain
- Profile first, then optimize
- "Premature optimization is the root of all evil"

**Performance-aware programming:**

- Choose efficient data structures (set vs list)
- Avoid unnecessary work (don't recompute)
- Use appropriate patterns naturally
- Stay readable while being smart

**Trade-offs:**

- Speed vs readability (usually choose readability)
- Memory vs speed (depends on data size)
- Simplicity vs performance (start simple)
- Development time vs runtime (optimize when proven slow)

**Measure before optimizing:**

- Use timeit for micro-benchmarks
- Profile real code with cProfile
- Test with realistic data
- Optimize based on measurements, not assumptions

Make it work, make it right, then make it fast -- in that order.

## Next Steps

- Learn [Data Structure Performance](data-structure-performance.md)
- Master [Profiling and Measurement](profiling-and-measurement.md)
- Explore [Optimization Techniques](optimization-techniques.md)
