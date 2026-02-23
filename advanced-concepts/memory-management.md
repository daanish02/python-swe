# Memory Management

## Table of Contents

- [Python Memory Model](#python-memory-model)
- [Reference Counting](#reference-counting)
- [Garbage Collection](#garbage-collection)
- [Memory Profiling](#memory-profiling)
- [Memory Optimization](#memory-optimization)
- [Weak References](#weak-references)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Python Memory Model

### Everything is an Object

Every value in Python is an object stored in memory:

```python
import sys

x = 42
print(type(x))  # <class 'int'>
print(id(x))    # Memory address (e.g., 140234567890)
print(sys.getsizeof(x))  # Size in bytes (e.g., 28)

# Even functions are objects
def func():
    pass

print(type(func))  # <class 'function'>
print(id(func))    # Memory address
```

### Object Identity

```python
# id() returns memory address
a = [1, 2, 3]
b = a
c = [1, 2, 3]

print(id(a))  # e.g., 140234567890
print(id(b))  # Same as a
print(id(c))  # Different from a and b

print(a is b)  # True (same object)
print(a is c)  # False (different objects)
print(a == c)  # True (same content)
```

### Integer Caching

Python caches small integers (-5 to 256):

```python
a = 100
b = 100
print(a is b)  # True (cached)

x = 1000
y = 1000
print(x is y)  # False (not cached)

# In interactive mode:
# >>> a = 1000
# >>> b = 1000
# >>> a is b
# False

# But in a script:
# a = 1000
# b = 1000
# print(a is b)  # True (compiler optimization)
```

### String Interning

Python interns some strings:

```python
a = "hello"
b = "hello"
print(a is b)  # True (interned)

c = "hello world"
d = "hello world"
print(c is d)  # May be True or False (depends on content)

# Force interning
import sys
e = sys.intern("hello world")
f = sys.intern("hello world")
print(e is f)  # True
```

## Reference Counting

### How it Works

Each object has a reference count tracking how many references point to it:

```python
import sys

a = []
print(sys.getrefcount(a))  # 2 (a + temporary reference in getrefcount)

b = a
print(sys.getrefcount(a))  # 3

c = [a, a]
print(sys.getrefcount(a))  # 5 (a, b, c[0], c[1], temp)

del b
print(sys.getrefcount(a))  # 4

del c
print(sys.getrefcount(a))  # 2
```

### Creating References

```python
import sys

x = [1, 2, 3]
print(sys.getrefcount(x))  # 2

# Create more references
y = x          # Assignment
z = [x]        # Container
d = {'key': x} # Dictionary

print(sys.getrefcount(x))  # 6 (x, y, z[0], d['key'], temp, frame)
```

### Removing References

```python
import sys

class MyClass:
    pass

obj = MyClass()
print(sys.getrefcount(obj))  # 2

refs = [obj, obj, obj]
print(sys.getrefcount(obj))  # 5

# Remove references
refs.clear()
print(sys.getrefcount(obj))  # 2

del obj  # Reference count drops to 0, object deallocated
```

### Reference Cycles

Problem: objects referencing each other:

```python
import sys

class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

# Create cycle
a = Node(1)
b = Node(2)
a.next = b
b.next = a  # Cycle!

print(sys.getrefcount(a))  # 3 (a, b.next, temp)

# Delete variables
del a, b
# Objects still exist (reference cycle)
# Garbage collector will eventually clean up
```

## Garbage Collection

### Generational Garbage Collection

Python uses generational GC to detect cycles:

```python
import gc

# Get GC stats
print(gc.get_count())  # (generation0, generation1, generation2)

# Get GC thresholds
print(gc.get_threshold())  # (threshold0, threshold1, threshold2)

# Manually trigger collection
collected = gc.collect()
print(f"Collected {collected} objects")
```

### Tracking Objects

```python
import gc

class MyClass:
    pass

# Create objects with cycles
objects = []
for i in range(5):
    obj = MyClass()
    obj.self_ref = obj  # Cycle
    objects.append(obj)

# Check tracked objects
tracked = len(gc.get_objects())
print(f"Tracked objects: {tracked}")

# Clear references
objects.clear()

# Collect garbage
collected = gc.collect()
print(f"Collected: {collected}")
```

### GC Callbacks

```python
import gc

def callback(phase, info):
    print(f"GC {phase}: {info}")

# Set callback
gc.callbacks.append(callback)

# Create garbage
for _ in range(1000):
    a = []
    a.append(a)

# Force collection
gc.collect()

# Remove callback
gc.callbacks.clear()
```

### Disabling GC

```python
import gc

# Disable automatic GC
gc.disable()

# Create garbage
for _ in range(1000):
    a = []
    a.append(a)

# Nothing collected automatically
print(f"Count: {gc.get_count()}")

# Manual collection
collected = gc.collect()
print(f"Collected: {collected}")

# Re-enable
gc.enable()
```

## Memory Profiling

### memory_profiler

```python
# Install: pip install memory-profiler

from memory_profiler import profile

@profile
def memory_intensive_function():
    # Create large list
    data = [0] * (10 ** 6)

    # Create large dict
    mapping = {i: i for i in range(10 ** 5)}

    return data, mapping

# Run with: python -m memory_profiler script.py
# Output shows line-by-line memory usage
```

### tracemalloc

Built-in memory tracking:

```python
import tracemalloc

# Start tracking
tracemalloc.start()

# Allocate memory
data = [0] * (10 ** 6)
more_data = {i: i for i in range(10 ** 5)}

# Get current usage
current, peak = tracemalloc.get_traced_memory()
print(f"Current: {current / 10**6:.2f} MB")
print(f"Peak: {peak / 10**6:.2f} MB")

# Get top memory allocations
snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

print("\nTop 3 allocations:")
for stat in top_stats[:3]:
    print(stat)

# Stop tracking
tracemalloc.stop()
```

### sys.getsizeof

Get size of objects:

```python
import sys

# Simple types
print(sys.getsizeof(42))              # 28 bytes
print(sys.getsizeof(3.14))            # 24 bytes
print(sys.getsizeof("hello"))         # 54 bytes

# Collections
print(sys.getsizeof([]))              # 56 bytes (empty list)
print(sys.getsizeof([1, 2, 3]))       # 80 bytes
print(sys.getsizeof({}))              # 64 bytes (empty dict)
print(sys.getsizeof({1: 2, 3: 4}))    # 232 bytes

# Custom objects
class MyClass:
    def __init__(self):
        self.x = 1
        self.y = 2

obj = MyClass()
print(sys.getsizeof(obj))             # Object overhead
print(sys.getsizeof(obj.__dict__))    # Instance dict
```

### Recursive Size

```python
import sys

def get_size(obj, seen=None):
    """Recursively calculate object size."""
    size = sys.getsizeof(obj)

    if seen is None:
        seen = set()

    obj_id = id(obj)
    if obj_id in seen:
        return 0

    seen.add(obj_id)

    if isinstance(obj, dict):
        size += sum(get_size(k, seen) + get_size(v, seen) for k, v in obj.items())
    elif hasattr(obj, '__dict__'):
        size += get_size(obj.__dict__, seen)
    elif hasattr(obj, '__iter__') and not isinstance(obj, (str, bytes, bytearray)):
        size += sum(get_size(item, seen) for item in obj)

    return size

# Test
data = {
    'list': [1, 2, 3, 4, 5],
    'dict': {'a': 1, 'b': 2},
    'nested': [[1, 2], [3, 4]]
}
print(f"Total size: {get_size(data)} bytes")
```

## Memory Optimization

### **slots**

Reduce memory overhead for classes:

```python
import sys

# Without __slots__
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# With __slots__
class PointSlots:
    __slots__ = ['x', 'y']

    def __init__(self, x, y):
        self.x = x
        self.y = y

# Compare sizes
p1 = Point(1, 2)
p2 = PointSlots(1, 2)

print(f"Point size: {sys.getsizeof(p1) + sys.getsizeof(p1.__dict__)} bytes")
print(f"PointSlots size: {sys.getsizeof(p2)} bytes")

# Memory savings for many instances
points = [Point(i, i) for i in range(10000)]
points_slots = [PointSlots(i, i) for i in range(10000)]
```

### Generators vs Lists

```python
import sys

# List - loads all in memory
numbers_list = [i for i in range(1000000)]
print(f"List size: {sys.getsizeof(numbers_list)} bytes")

# Generator - computes on demand
numbers_gen = (i for i in range(1000000))
print(f"Generator size: {sys.getsizeof(numbers_gen)} bytes")

# Generator saves memory
def read_large_file(filename):
    with open(filename) as f:
        for line in f:  # Generator - one line at a time
            yield line.strip()

# Instead of:
# return [line.strip() for line in f]  # Loads entire file
```

### Interning Strings

```python
import sys

# Without interning
strings = ["user_" + str(i) for i in range(1000)]
size_without = sum(sys.getsizeof(s) for s in strings)

# With interning
strings_interned = [sys.intern("user_" + str(i)) for i in range(1000)]
size_with = sum(sys.getsizeof(s) for s in strings_interned)

print(f"Without interning: {size_without} bytes")
print(f"With interning: {size_with} bytes")
```

### array Module

More efficient than lists for numeric data:

```python
import sys
import array

# List of integers
list_data = [i for i in range(10000)]
print(f"List size: {sys.getsizeof(list_data)} bytes")

# Array of integers
array_data = array.array('i', range(10000))
print(f"Array size: {sys.getsizeof(array_data)} bytes")

# Array is more memory-efficient
```

## Weak References

### weakref Module

References that don't increase ref count:

```python
import weakref
import sys

class MyClass:
    pass

# Strong reference
obj = MyClass()
print(sys.getrefcount(obj))  # 2

# Weak reference
weak = weakref.ref(obj)
print(sys.getrefcount(obj))  # Still 2 (weak ref doesn't count)

# Access object
print(weak())  # <__main__.MyClass object at 0x...>

# Delete strong reference
del obj
print(weak())  # None (object was deleted)
```

### WeakValueDictionary

Cache that doesn't prevent garbage collection:

```python
import weakref

class ExpensiveObject:
    def __init__(self, name):
        self.name = name
        print(f"Creating {name}")

    def __del__(self):
        print(f"Deleting {self.name}")

# Regular dict keeps objects alive
cache = {}
cache['obj1'] = ExpensiveObject('Object 1')
del cache['obj1']  # Object still exists until cache is cleared

# WeakValueDictionary allows GC
weak_cache = weakref.WeakValueDictionary()
obj = ExpensiveObject('Object 2')
weak_cache['obj2'] = obj

print("Before del")
del obj  # Object deleted immediately
print("After del")
print(weak_cache.get('obj2'))  # None
```

### Circular Reference with Weak References

```python
import weakref

class Node:
    def __init__(self, value):
        self.value = value
        self._parent = None  # Weak reference

    @property
    def parent(self):
        return self._parent() if self._parent else None

    @parent.setter
    def parent(self, node):
        self._parent = weakref.ref(node) if node else None

# Create nodes
root = Node("root")
child = Node("child")
child.parent = root  # Weak reference

# No circular reference issue
print(child.parent.value)  # root

# Delete root
del root
print(child.parent)  # None (automatically cleaned up)
```

### WeakSet

```python
import weakref

class Item:
    def __init__(self, name):
        self.name = name

# Track items without preventing GC
items = weakref.WeakSet()

item1 = Item("Item 1")
item2 = Item("Item 2")

items.add(item1)
items.add(item2)

print(len(items))  # 2

del item1
print(len(items))  # 1 (item1 was garbage collected)
```

## Summary

Python memory management:

**Memory model:**

- Everything is an object
- Objects have identity (id), type, and value
- Small integers and strings are cached/interned

**Reference counting:**

- Each object tracks reference count
- Object deallocated when count reaches 0
- Fast but can't handle cycles

**Garbage collection:**

- Generational GC detects cycles
- Three generations (0, 1, 2)
- Can be controlled with `gc` module

**Memory profiling:**

- `memory_profiler`: Line-by-line profiling
- `tracemalloc`: Built-in memory tracking
- `sys.getsizeof()`: Object size

**Optimization:**

- `__slots__`: Reduce instance overhead
- Generators: Lazy evaluation
- `array` module: Efficient numeric arrays
- String interning: Share identical strings

**Weak references:**

- Don't increase reference count
- Allow garbage collection
- Useful for caches and avoiding cycles

## Next Steps

- Profile with [Performance Optimization](../performance_and_optimization/profiling.md)
- Apply in [Python Internals](python-internals.md)
- Optimize [Concurrency](concurrency.md) patterns
