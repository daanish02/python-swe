# Memory Optimization

## Python Memory Model

**Reference counting**: Python tracks how many references point to each object, freeing memory when count reaches zero.

### How Python Manages Memory

```python
# Every Python object has:
# - Type information
# - Reference count
# - Value/data

import sys

x = 42
print(sys.getsizeof(x))  # 28 bytes (just for storing int!)

# Python objects have overhead
# Small int needs ~28 bytes (vs 4 bytes in C)
```

### Everything is an Object

```python
# Python stores references, not values directly
a = [1, 2, 3]
b = a  # b references same list

# Modifying through one reference affects other
b.append(4)
print(a)  # [1, 2, 3, 4]

# Each list item is also a reference
numbers = [1, 2, 3]
# numbers doesn't contain integers
# It contains references to integer objects
```

### Memory Overhead

```python
import sys

# Small values have large overhead
print(sys.getsizeof(1))        # ~28 bytes
print(sys.getsizeof(True))     # ~28 bytes
print(sys.getsizeof(None))     # ~16 bytes

# Collections have base overhead
print(sys.getsizeof([]))       # ~56 bytes (empty list!)
print(sys.getsizeof({}))       # ~232 bytes (empty dict!)
print(sys.getsizeof(set()))    # ~216 bytes (empty set!)

# List with 3 integers
numbers = [1, 2, 3]
print(sys.getsizeof(numbers))  # ~88 bytes (just the list)
# Plus 28 bytes × 3 for the integers = ~172 bytes total
# In C: 12 bytes (4 bytes × 3 integers)
```

## Measuring Memory Usage

### Using sys.getsizeof()

```python
import sys

# Measure size of objects
data = list(range(1000))
print(f"List size: {sys.getsizeof(data)} bytes")

# Warning: Only measures object, not referenced objects
nested = [[1, 2, 3], [4, 5, 6]]
print(sys.getsizeof(nested))  # Only outer list!

# Recursive size calculation
def deep_getsizeof(obj, seen=None):
    """Calculate size including referenced objects"""
    if seen is None:
        seen = set()

    obj_id = id(obj)
    if obj_id in seen:
        return 0

    seen.add(obj_id)
    size = sys.getsizeof(obj)

    if isinstance(obj, dict):
        size += sum(deep_getsizeof(k, seen) + deep_getsizeof(v, seen)
                   for k, v in obj.items())
    elif isinstance(obj, (list, tuple, set, frozenset)):
        size += sum(deep_getsizeof(item, seen) for item in obj)

    return size

nested = [[1, 2, 3], [4, 5, 6]]
print(f"Shallow: {sys.getsizeof(nested)} bytes")
print(f"Deep: {deep_getsizeof(nested)} bytes")
```

### Using memory_profiler

```python
from memory_profiler import profile

@profile
def load_data():
    # Line-by-line memory usage
    data = [i for i in range(1000000)]  # ~8 MB
    doubled = [x * 2 for x in data]     # Another ~8 MB
    return doubled

# Run: python -m memory_profiler script.py
# Shows memory increase per line
```

### Tracking Memory Over Time

```python
import tracemalloc

# Start tracking
tracemalloc.start()

# Run memory-intensive code
data = [i for i in range(1000000)]

# Get current memory usage
current, peak = tracemalloc.get_traced_memory()
print(f"Current: {current / 1024 / 1024:.2f} MB")
print(f"Peak: {peak / 1024 / 1024:.2f} MB")

# Get top memory allocations
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

for stat in top_stats[:5]:
    print(stat)

tracemalloc.stop()
```

## Using **slots**

\***\*slots\*\***: Class attribute that reserves space for declared attributes, reducing memory overhead.

### Default Behavior

```python
class User:
    def __init__(self, name, age):
        self.name = name
        self.age = age

# Each instance has a __dict__
user = User("Alice", 30)
print(user.__dict__)  # {'name': 'Alice', 'age': 30}

import sys
print(sys.getsizeof(user.__dict__))  # ~240 bytes overhead!
```

### Using **slots**

```python
class User:
    __slots__ = ['name', 'age']  # Declare attributes

    def __init__(self, name, age):
        self.name = name
        self.age = age

user = User("Alice", 30)
# No __dict__!
# print(user.__dict__)  # AttributeError

import sys
# Roughly 50% less memory per instance
```

### Memory Savings

```python
import sys

# Without __slots__
class RegularUser:
    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

# With __slots__
class SlottedUser:
    __slots__ = ['name', 'age', 'email']

    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

# Compare memory usage
regular = RegularUser("Alice", 30, "alice@example.com")
slotted = SlottedUser("Alice", 30, "alice@example.com")

print(f"Regular: {sys.getsizeof(regular) + sys.getsizeof(regular.__dict__)} bytes")
print(f"Slotted: {sys.getsizeof(slotted)} bytes")

# For 1 million instances:
# Regular: ~300 MB
# Slotted: ~150 MB
# Saves ~150 MB!
```

### When to Use **slots**

```python
# Use __slots__ when:
# - Creating many instances (thousands+)
# - Fixed set of attributes
# - Memory is constrained

# Good use case: Data objects
class Point:
    __slots__ = ['x', 'y']

    def __init__(self, x, y):
        self.x = x
        self.y = y

points = [Point(i, i*2) for i in range(1000000)]
# Saves significant memory

# Don't use __slots__ when:
# - Only a few instances
# - Need dynamic attributes
# - Subclassing complex classes
# - Not sure about all attributes

# Bad: Dynamic attributes needed
class Config:
    __slots__ = ['host', 'port']  # Can't add new attributes!

config = Config()
config.timeout = 30  # AttributeError!
```

### Limitations of **slots**

```python
class Base:
    __slots__ = ['x']

# Subclass needs its own __slots__
class Derived(Base):
    __slots__ = ['y']  # Only for new attributes

# Can't use with multiple inheritance (usually)
# Can't have weakref (unless add '__weakref__' to __slots__)
# Can't have __dict__ (unless add '__dict__' to __slots__)
```

## Generators for Memory Efficiency

**Generator**: Yields values one at a time instead of creating full list in memory.

### List vs Generator Memory

```python
import sys

# List: All in memory
numbers_list = [x for x in range(1000000)]
print(f"List: {sys.getsizeof(numbers_list)} bytes")  # ~8 MB

# Generator: One value at a time
numbers_gen = (x for x in range(1000000))
print(f"Generator: {sys.getsizeof(numbers_gen)} bytes")  # ~200 bytes!
```

### Processing Large Files

```python
# Bad: Loads entire file into memory
def process_file_list(filename):
    with open(filename) as f:
        lines = f.readlines()  # All lines in memory

    for line in lines:
        process(line)

# For 10 GB file: Uses 10+ GB memory

# Good: Generator processes one line at a time
def process_file_gen(filename):
    with open(filename) as f:
        for line in f:  # One line at a time
            yield process(line)

for result in process_file_gen('huge.txt'):
    save(result)

# For 10 GB file: Uses ~constant memory
```

### Generator Pipelines

```python
# Chain generators for complex processing
def read_logs(filename):
    """Read log file line by line"""
    with open(filename) as f:
        for line in f:
            yield line.strip()

def filter_errors(lines):
    """Keep only error lines"""
    for line in lines:
        if 'ERROR' in line:
            yield line

def parse_errors(lines):
    """Parse error messages"""
    for line in lines:
        yield parse_line(line)

# Process 100 GB log file with minimal memory
pipeline = parse_errors(filter_errors(read_logs('app.log')))
for error in pipeline:
    report(error)

# Memory usage stays constant regardless of file size
```

### When to Use Generators

```python
# Use generators for:
# - Large datasets that don't fit in memory
# - Processing data once (streaming)
# - Lazy evaluation (compute on demand)

# Generator: Process large dataset
def get_large_dataset():
    for record in database.scan():  # Millions of records
        yield transform(record)

# List: Small, reused data
def get_menu_items():
    # Small, accessed multiple times
    return ['File', 'Edit', 'View']
```

## Memory-Efficient Data Structures

### Array Module

**array**: Compact storage for homogeneous numeric data (C-style arrays).

```python
from array import array
import sys

# List of integers
numbers_list = list(range(1000000))
print(f"List: {sys.getsizeof(numbers_list)} bytes")  # ~8 MB

# Array of integers
numbers_array = array('i', range(1000000))  # 'i' = signed int
print(f"Array: {sys.getsizeof(numbers_array)} bytes")  # ~4 MB

# Array uses half the memory!
# But only for homogeneous numeric types
```

### Using array

```python
from array import array

# Type codes:
# 'b': signed char (1 byte)
# 'i': signed int (4 bytes)
# 'f': float (4 bytes)
# 'd': double (8 bytes)

# Create array
temps = array('f', [72.5, 73.1, 71.8, 74.2])

# Array methods similar to list
temps.append(75.0)
temps.extend([76.1, 77.3])
print(temps[0])  # 72.5

# Much more memory-efficient for numeric data
# But slower than list for some operations
```

### Numpy Arrays

```python
import numpy as np
import sys

# Python list
py_list = list(range(1000000))
print(f"Python list: {sys.getsizeof(py_list) / 1024 / 1024:.2f} MB")

# Numpy array
np_array = np.arange(1000000)
print(f"Numpy array: {np_array.nbytes / 1024 / 1024:.2f} MB")

# Numpy is more memory-efficient
# Plus much faster for numeric operations
```

### Named Tuples

```python
from collections import namedtuple
import sys

# Regular class (with __dict__)
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# Named tuple (immutable, no __dict__)
PointTuple = namedtuple('Point', ['x', 'y'])

# Memory comparison
regular = Point(10, 20)
tuple_point = PointTuple(10, 20)

print(f"Class: {sys.getsizeof(regular) + sys.getsizeof(regular.__dict__)} bytes")
print(f"Namedtuple: {sys.getsizeof(tuple_point)} bytes")

# Named tuple uses less memory
# But immutable (can't change values)
```

## Garbage Collection

**Garbage collection**: Automatic process of reclaiming memory from objects no longer in use.

### Reference Counting

```python
import sys

x = [1, 2, 3]
print(sys.getrefcount(x))  # 2 (x + temporary for getrefcount)

y = x  # Create another reference
print(sys.getrefcount(x))  # 3

del y  # Remove reference
print(sys.getrefcount(x))  # 2

# When refcount reaches 0, memory is freed immediately
```

### Circular References

```python
# Problem: Circular references prevent immediate cleanup
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

a = Node(1)
b = Node(2)
a.next = b
b.next = a  # Circular reference!

del a
del b
# Memory not immediately freed (circular reference)
# Garbage collector will eventually clean up
```

### Manual Garbage Collection

```python
import gc

# Disable automatic GC (for performance-critical code)
gc.disable()

# ... run performance-critical code ...

# Manually trigger collection when appropriate
gc.collect()

# Re-enable automatic GC
gc.enable()

# Check GC stats
print(gc.get_count())  # Generations counts
print(gc.get_stats())  # Detailed statistics
```

### Weak References

**weakref**: Reference that doesn't prevent garbage collection.

```python
import weakref

class LargeObject:
    def __init__(self, data):
        self.data = data

# Strong reference (prevents GC)
obj = LargeObject([1, 2, 3])

# Weak reference (allows GC)
weak_obj = weakref.ref(obj)

# Access through weak reference
print(weak_obj())  # LargeObject instance

# Remove strong reference
del obj

# Weak reference now returns None
print(weak_obj())  # None (object was garbage collected)
```

### Memory Leaks to Avoid

```python
# Leak 1: Global caches that grow unbounded
cache = {}  # Never cleaned up!

def get_data(key):
    if key not in cache:
        cache[key] = expensive_operation(key)
    return cache[key]
# Use lru_cache with maxsize instead

# Leak 2: Closures holding references
def create_functions():
    functions = []
    for i in range(10000):
        data = load_large_data(i)  # Large object
        functions.append(lambda: process(data))  # Closure keeps reference!
    return functions
# data objects never freed (held by closures)

# Leak 3: Not closing files/resources
def process_files(filenames):
    for filename in filenames:
        f = open(filename)  # Never closed!
        process(f.read())
# Use 'with' statement

# Leak 4: Circular references with __del__
class Node:
    def __init__(self):
        self.ref = None

    def __del__(self):
        # __del__ can prevent GC of circular references
        cleanup()
```

## Summary

Python memory optimization:

**Memory model:**

- Everything is an object with overhead
- Reference counting for automatic cleanup
- Small values have large relative overhead
- Use `sys.getsizeof()` to measure

\***\*slots**:\*\*

- Reduces memory per instance by ~50%
- Use for many instances with fixed attributes
- Prevents dynamic attribute assignment
- Significant savings for millions of instances

**Generators:**

- Minimal memory (compute on demand)
- Use for large datasets
- Generator expressions: `(x for x in items)`
- Chain generators for complex pipelines

**Memory-efficient structures:**

- array module for homogeneous numeric data
- numpy for large numeric datasets
- namedtuple for immutable records
- Tuples use less memory than lists

**Garbage collection:**

- Automatic via reference counting
- Cyclic GC for circular references
- Use weakref to allow GC
- Avoid unbounded caches and circular references

Memory optimization matters when processing large datasets or creating many objects.

## Next Steps

- Learn [Parallel and Concurrent Execution](parallel-and-concurrent.md)
- Explore [Native Extensions](native-extensions.md)
- Review [Profiling and Measurement](profiling-and-measurement.md)
