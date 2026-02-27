# Profiling and Measurement

## Table of Contents

- [Why Profile](#why-profile)
- [Using timeit for Benchmarks](#using-timeit-for-benchmarks)
- [Profiling with cProfile](#profiling-with-cprofile)
- [Line-by-Line Profiling](#line-by-line-profiling)
- [Memory Profiling](#memory-profiling)
- [Scientific Measurement](#scientific-measurement)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Why Profile

**Profiling**: Measuring where your program spends time and resources to identify bottlenecks.

### Don't Guess, Measure

```python
# Wrong approach: Optimize based on intuition
def process_data(items):
    # "This loop must be slow, let me optimize it"
    # (Actually fast - not the bottleneck)
    return [transform(item) for item in items]

# Right approach: Profile to find actual bottlenecks
import cProfile
cProfile.run('main()')

# Results show:
# - process_data(): 0.1s (10% of time)
# - database_query(): 0.9s (90% of time)  ← Real bottleneck!

# Optimize database_query, not process_data
```

### What Profiling Reveals

```python
# Profile output shows:
# - Which functions take the most time
# - How many times functions are called
# - Call relationships (who calls what)
# - Hidden expensive operations

# Example insights:
# "validate_email() called 100,000 times" → Cache results
# "json.dumps() takes 60% of time" → Use faster serializer
# "list concatenation in loop" → Use list.extend() instead
```

## Using timeit for Benchmarks

**timeit**: Module for timing small code snippets, measures best of multiple runs.

### Basic timeit Usage

```python
import timeit

# Time a statement
time = timeit.timeit(
    '[x**2 for x in range(100)]',
    number=10000  # Repeat 10,000 times
)
print(f"Time: {time:.4f} seconds")

# Time with setup
time = timeit.timeit(
    stmt='result = func(data)',
    setup='data = list(range(1000))',
    number=1000
)
```

### Comparing Approaches

```python
import timeit

# Compare list comprehension vs map
data = list(range(10000))

list_comp = timeit.timeit(
    '[x * 2 for x in data]',
    globals={'data': data},
    number=1000
)

map_func = timeit.timeit(
    'list(map(lambda x: x * 2, data))',
    globals={'data': data},
    number=1000
)

print(f"List comp: {list_comp:.4f}s")
print(f"Map:       {map_func:.4f}s")
print(f"Winner: {'List comp' if list_comp < map_func else 'Map'}")
# List comp: 0.8234s
# Map:       1.2456s
# Winner: List comp
```

### Command-Line Usage

```bash
# Time statement from command line
python -m timeit '[x**2 for x in range(100)]'

# Output:
# 20000 loops, best of 5: 13.8 usec per loop

# With setup
python -m timeit -s 'import math' 'math.sqrt(144)'

# Multiple statements
python -m timeit -s 'data = list(range(1000))' 'sum(data)'
```

### Best Practices

```python
import timeit

# Bad: Including setup in timing
bad_time = timeit.timeit(
    '''
data = list(range(1000))  # This gets timed!
result = sum(data)
    ''',
    number=1000
)

# Good: Setup outside timing
good_time = timeit.timeit(
    stmt='result = sum(data)',
    setup='data = list(range(1000))',  # Not timed
    number=1000
)

# Repeat for statistical significance
times = timeit.repeat(
    stmt='sum(data)',
    setup='data = list(range(1000))',
    number=1000,
    repeat=5  # Run 5 times
)
print(f"Best: {min(times):.4f}s")
print(f"Average: {sum(times)/len(times):.4f}s")
```

### Timing Functions

```python
import timeit

def approach_1(data):
    return [x * 2 for x in data]

def approach_2(data):
    return list(map(lambda x: x * 2, data))

# Time functions
data = list(range(10000))

time_1 = timeit.timeit(
    'approach_1(data)',
    globals={'approach_1': approach_1, 'data': data},
    number=1000
)

time_2 = timeit.timeit(
    'approach_2(data)',
    globals={'approach_2': approach_2, 'data': data},
    number=1000
)

print(f"Approach 1: {time_1:.4f}s")
print(f"Approach 2: {time_2:.4f}s")
```

## Profiling with cProfile

**cProfile**: Built-in profiler that shows time spent in each function.

### Basic Profiling

```python
import cProfile

def slow_function():
    total = 0
    for i in range(1000000):
        total += i
    return total

def main():
    result1 = slow_function()
    result2 = slow_function()
    return result1 + result2

# Profile the main function
cProfile.run('main()')

# Output:
#          5 function calls in 0.123 seconds
#
#    ncalls  tottime  percall  cumtime  percall filename:lineno(function)
#         1    0.000    0.000    0.123    0.123 <string>:1(<module>)
#         1    0.000    0.000    0.123    0.123 script.py:7(main)
#         2    0.123    0.062    0.123    0.062 script.py:2(slow_function)
```

### Saving and Analyzing Profiles

```python
import cProfile
import pstats

# Save profile to file
cProfile.run('main()', 'profile_stats')

# Analyze saved profile
stats = pstats.Stats('profile_stats')

# Sort by cumulative time
stats.sort_stats('cumulative')
stats.print_stats(10)  # Top 10 functions

# Sort by total time (time in function, excluding subcalls)
stats.sort_stats('tottime')
stats.print_stats(10)

# Filter by filename
stats.sort_stats('cumulative')
stats.print_stats('mymodule.py')

# Print call relationships
stats.print_callers(10)  # Who calls these functions
stats.print_callees(10)  # What these functions call
```

### Profiling Decorator

```python
import cProfile
import pstats
from functools import wraps

def profile(func):
    """Decorator to profile a function"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        profiler = cProfile.Profile()
        profiler.enable()
        result = func(*args, **kwargs)
        profiler.disable()

        stats = pstats.Stats(profiler)
        stats.sort_stats('cumulative')
        stats.print_stats(20)

        return result
    return wrapper

@profile
def process_large_dataset():
    # Function automatically profiled when called
    data = load_data()
    results = analyze(data)
    return results
```

### Interpreting cProfile Output

```python
# Example output explained:
#
# ncalls  tottime  percall  cumtime  percall filename:lineno(function)
# 100     0.500    0.005    2.000    0.020   module.py:10(process)
#
# ncalls:  Number of times function was called (100)
# tottime: Total time in function (excluding subcalls) - 0.500s
# percall: tottime / ncalls - 0.005s per call
# cumtime: Total time including subcalls - 2.000s
# percall: cumtime / ncalls - 0.020s per call
#
# Large difference between tottime and cumtime means
# function spends most time calling other functions
```

## Line-by-Line Profiling

**line_profiler**: Third-party tool showing time spent on each line of code.

### Installing line_profiler

```bash
pip install line_profiler
```

### Using line_profiler

```python
# my_script.py
from line_profiler import profile

@profile
def process_items(items):
    # Each line will be profiled
    total = 0
    results = []

    for item in items:
        value = expensive_calculation(item)
        total += value
        results.append(value)

    average = total / len(items)
    return results, average

# Run with kernprof
# kernprof -l -v my_script.py
```

```bash
# Command line usage
kernprof -l -v my_script.py

# Output shows time per line:
# Line #      Hits         Time  Per Hit   % Time  Line Contents
# ==============================================================
#      3                                           @profile
#      4                                           def process_items(items):
#      5         1          2.0      2.0      0.0      total = 0
#      6         1          1.5      1.5      0.0      results = []
#      7
#      8      1001        523.0      0.5      2.1      for item in items:
#      9      1000      23456.0     23.5     95.2          value = expensive_calculation(item)
#     10      1000        345.0      0.3      1.4          total += value
#     11      1000        312.0      0.3      1.3          results.append(value)
#     12
#     13         1          2.5      2.5      0.0      average = total / len(items)
#     14         1          1.0      1.0      0.0      return results, average
```

### Programmatic Usage

```python
from line_profiler import LineProfiler

def process_data(data):
    result = []
    for item in data:
        processed = transform(item)
        result.append(processed)
    return result

# Profile specific function
profiler = LineProfiler()
profiler.add_function(process_data)
profiler.enable()

# Run code
result = process_data(my_data)

profiler.disable()
profiler.print_stats()
```

## Memory Profiling

**memory_profiler**: Tool for tracking memory usage line by line.

### Installing memory_profiler

```bash
pip install memory_profiler
```

### Using memory_profiler

```python
# my_script.py
from memory_profiler import profile

@profile
def load_large_data():
    # Each line's memory impact shown
    data = [i for i in range(1000000)]  # Large list
    filtered = [x for x in data if x % 2 == 0]  # Another large list
    result = sum(filtered)  # Small value
    return result

# Run: python -m memory_profiler my_script.py
```

```bash
# Output shows memory per line:
# Line #    Mem usage    Increment  Occurrences   Line Contents
# ================================================================
#      3   38.4 MiB   38.4 MiB           1       @profile
#      4                                         def load_large_data():
#      5   76.3 MiB   37.9 MiB           1           data = [i for i in range(1000000)]
#      6   95.1 MiB   18.8 MiB           1           filtered = [x for x in data if x % 2 == 0]
#      7   95.1 MiB    0.0 MiB           1           result = sum(filtered)
#      8   95.1 MiB    0.0 MiB           1           return result
```

### Tracking Memory Over Time

```python
from memory_profiler import memory_usage

def memory_intensive_function():
    large_list = [0] * (10**7)
    processed = [x * 2 for x in large_list]
    return sum(processed)

# Measure memory usage over time
mem_usage = memory_usage(memory_intensive_function, interval=0.1)

print(f"Peak memory: {max(mem_usage):.2f} MiB")
print(f"Average memory: {sum(mem_usage)/len(mem_usage):.2f} MiB")
```

### Finding Memory Leaks

```python
from memory_profiler import profile
import gc

@profile
def potential_leak():
    # Monitor for memory that doesn't get freed
    cache = []
    for i in range(1000):
        data = generate_data()
        cache.append(data)  # Memory keeps growing
        # Should periodically clear or use limited cache

    # Force garbage collection to see if memory is freed
    gc.collect()

    return len(cache)

# Run and check if memory drops after gc.collect()
```

## Scientific Measurement

### Statistical Significance

```python
import timeit
import statistics

def measure_with_stats(stmt, setup='pass', number=1000, repeat=10):
    """Measure with statistical analysis"""
    times = timeit.repeat(stmt, setup=setup, number=number, repeat=repeat)

    return {
        'min': min(times),
        'max': max(times),
        'mean': statistics.mean(times),
        'median': statistics.median(times),
        'stdev': statistics.stdev(times) if len(times) > 1 else 0,
    }

# Compare two approaches
results_a = measure_with_stats('[x*2 for x in data]', 'data=list(range(1000))')
results_b = measure_with_stats('list(map(lambda x: x*2, data))', 'data=list(range(1000))')

print(f"Approach A: {results_a['mean']:.4f}s ± {results_a['stdev']:.4f}s")
print(f"Approach B: {results_b['mean']:.4f}s ± {results_b['stdev']:.4f}s")
```

### Warm-up Runs

```python
import timeit

def benchmark_with_warmup(func, warmup=3, runs=10):
    """Run warmup iterations before timing"""
    # Warm up (JIT, cache, etc.)
    for _ in range(warmup):
        func()

    # Actual measurements
    times = []
    for _ in range(runs):
        start = timeit.default_timer()
        func()
        elapsed = timeit.default_timer() - start
        times.append(elapsed)

    return min(times), sum(times) / len(times)

def my_function():
    return sum(range(10000))

min_time, avg_time = benchmark_with_warmup(my_function)
print(f"Min: {min_time:.4f}s, Avg: {avg_time:.4f}s")
```

### Context and Reproducibility

```python
import sys
import platform
import timeit

def benchmark_context():
    """Report benchmark environment"""
    print("Benchmark Context:")
    print(f"  Python: {sys.version}")
    print(f"  Platform: {platform.platform()}")
    print(f"  Processor: {platform.processor()}")

    # Disable garbage collection for more stable results
    import gc
    gc_old = gc.isenabled()
    gc.disable()

    # Run benchmark
    time = timeit.timeit('sum(range(10000))', number=10000)

    # Restore GC state
    if gc_old:
        gc.enable()

    print(f"  Time: {time:.4f}s")

benchmark_context()
```

### Comparing Performance

```python
def compare_approaches(approaches, setup='', number=1000):
    """Compare multiple approaches fairly"""
    results = {}

    for name, stmt in approaches.items():
        time = timeit.timeit(stmt, setup=setup, number=number)
        results[name] = time

    # Find fastest
    fastest = min(results.values())

    # Print comparison
    print(f"{'Approach':<20} {'Time':>10} {'Relative':>10}")
    print("-" * 42)
    for name, time in sorted(results.items(), key=lambda x: x[1]):
        relative = time / fastest
        print(f"{name:<20} {time:>10.4f}s {relative:>9.2f}x")

# Example usage
compare_approaches({
    'List comp':   '[x*2 for x in data]',
    'Map':         'list(map(lambda x: x*2, data))',
    'For loop':    'result = []\nfor x in data:\n    result.append(x*2)',
    'Generator':   'list(x*2 for x in data)',
}, setup='data = list(range(1000))')

# Output:
# Approach             Time   Relative
# ------------------------------------------
# List comp          0.0523s      1.00x
# Generator          0.0548s      1.05x
# For loop           0.0891s      1.70x
# Map                0.1234s      2.36x
```

## Summary

Profiling and measurement tools:

**timeit:**

- Timing small code snippets
- Comparing different approaches
- Micro-benchmarks
- Use `timeit.repeat()` for multiple runs

**cProfile:**

- Function-level profiling
- Shows time per function
- Call counts and relationships
- Use `pstats` for analysis

**line_profiler:**

- Line-by-line timing
- Find exact slow lines
- Install: `pip install line_profiler`
- Use `@profile` decorator

**memory_profiler:**

- Line-by-line memory usage
- Find memory leaks
- Track memory over time
- Install: `pip install memory_profiler`

**Best practices:**

- Profile before optimizing
- Use appropriate tool for the job
- Measure with realistic data
- Run multiple times for reliability
- Report benchmark context
- Focus on the biggest bottlenecks

Don't optimize blindly -- profile first, then optimize.

## Next Steps

- Apply [Optimization Techniques](optimization-techniques.md)
- Understand [Memory Optimization](memory-optimization.md)
- Learn [Parallel and Concurrent Execution](parallel-and-concurrent.md)
