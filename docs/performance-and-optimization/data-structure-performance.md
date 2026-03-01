# Data Structure Performance

## Lists in CPython

**Dynamic array**: Python lists are implemented as dynamic arrays in CPython, storing pointers to objects in contiguous memory.

### How Lists Work

```python
# CPython implementation details:
# - Array of pointers to objects
# - Dynamically resizes when full
# - Over-allocates to amortize cost

numbers = [1, 2, 3]
# Memory layout (simplified):
# [ptr1, ptr2, ptr3, empty, empty, ...]
#   ↓     ↓     ↓
# [int]  [int] [int]
```

### List Operation Costs

```python
# Fast operations (constant time):
items = [1, 2, 3, 4, 5]

items.append(6)          # Fast: Add to end
items.pop()              # Fast: Remove from end
items[2]                 # Fast: Access by index
items[1] = 10           # Fast: Update by index
len(items)              # Fast: Length is cached

# Slow operations (linear time):
items.insert(0, 99)     # Slow: Shifts all elements
items.pop(0)            # Slow: Shifts all elements
items.remove(3)         # Slow: Searches then shifts
3 in items              # Slow: Searches all elements
items + other_list      # Slow: Creates new list
```

### Practical Examples

```python
# Bad: Building list by inserting at beginning
def process_items(items):
    result = []
    for item in items:
        result.insert(0, process(item))  # Slow!
    return result

# Good: Append then reverse
def process_items(items):
    result = []
    for item in items:
        result.append(process(item))  # Fast
    result.reverse()  # Single operation
    return result

# Even better: Use deque
from collections import deque

def process_items(items):
    result = deque()
    for item in items:
        result.appendleft(process(item))  # Fast
    return list(result)
```

### List Growth Strategy

```python
# CPython over-allocates to avoid frequent resizing
import sys

items = []
for i in range(10):
    items.append(i)
    print(f"len={len(items)}, size={sys.getsizeof(items)}")

# Output shows jumps:
# len=1, size=88   (capacity: 4)
# len=2, size=88
# len=5, size=120  (capacity: 8)
# len=9, size=184  (capacity: 16)

# Growth pattern: 0, 4, 8, 16, 25, 35, 46, 58, 72, 88...
# Roughly: new_capacity = old_capacity + (old_capacity >> 3)
```

## Dictionaries and Hash Tables

**Hash table**: Python dicts are implemented as hash tables, providing fast key-based lookups.

### How Dicts Work

```python
# CPython implementation:
# - Hash function maps keys to indices
# - Handles collisions with open addressing
# - Maintains insertion order (since Python 3.7)

user = {'name': 'Alice', 'age': 30}

# Simplified process:
# hash('name') → index → store value
# hash('age')  → index → store value
```

### Dict Operation Costs

```python
user = {'name': 'Alice', 'age': 30, 'email': 'alice@example.com'}

# Fast operations (average constant time):
user['name']                    # Fast: Hash lookup
user['city'] = 'NYC'           # Fast: Hash insert
del user['age']                # Fast: Hash delete
'email' in user                # Fast: Hash check
len(user)                      # Fast: Size is cached

# Linear time operations:
list(user.keys())              # Slow: Creates list of all keys
list(user.values())            # Slow: Creates list of all values
user.copy()                    # Slow: Copies all items
```

### Hash Collisions

```python
# Keys must be hashable (immutable)
valid_keys = {
    'string': 1,
    42: 2,
    (1, 2): 3,          # Tuple is hashable
    frozenset([1, 2]): 4,  # Frozen set is hashable
}

# Invalid keys (not hashable):
# invalid = {
#     [1, 2]: 'value',    # Lists are mutable
#     {1: 2}: 'value',    # Dicts are mutable
# }

# Good hash functions distribute keys evenly
# Python's hash() handles this well for built-in types
```

### Practical Examples

```python
# Bad: Using list for lookups
def find_user(users_list, user_id):
    for user in users_list:
        if user['id'] == user_id:
            return user
    return None

# Good: Use dict for O(1) lookups
users_by_id = {user['id']: user for user in users_list}

def find_user(user_id):
    return users_by_id.get(user_id)

# 1000x faster for large datasets
```

### Dict Memory Usage

```python
# Dicts use more memory than lists
import sys

# List: ~64 bytes + 8 bytes per item
items_list = [1, 2, 3, 4, 5]
print(sys.getsizeof(items_list))  # ~104 bytes

# Dict: ~240 bytes + ~24 bytes per item
items_dict = {i: i for i in range(5)}
print(sys.getsizeof(items_dict))  # ~360 bytes

# Trade memory for speed
```

## Sets and Membership Testing

**Set**: Unordered collection implemented as hash table, optimized for membership testing.

### How Sets Work

```python
# Like dict keys without values
# Uses hash table for O(1) lookups

numbers = {1, 2, 3, 4, 5}
# Internally: hash table of unique elements
```

### Set Operation Costs

```python
numbers = {1, 2, 3, 4, 5}

# Fast operations (average constant time):
3 in numbers            # Fast: Hash lookup
numbers.add(6)          # Fast: Hash insert
numbers.remove(3)       # Fast: Hash delete
len(numbers)            # Fast: Size is cached

# Set operations (fast - proportional to smaller set):
other = {4, 5, 6, 7}
numbers & other         # Intersection: {4, 5}
numbers | other         # Union: {1, 2, 3, 4, 5, 6, 7}
numbers - other         # Difference: {1, 2, 3}
numbers ^ other         # Symmetric difference: {1, 2, 3, 6, 7}
```

### Membership Testing

```python
import timeit

# Compare membership testing performance
data = list(range(10000))

# List: O(n) - must check each element
list_time = timeit.timeit(
    '9999 in data',
    globals={'data': data},
    number=10000
)

# Set: O(1) - hash lookup
data_set = set(data)
set_time = timeit.timeit(
    '9999 in data',
    globals={'data': data_set},
    number=10000
)

print(f"List: {list_time:.4f}s")  # ~5.2s
print(f"Set:  {set_time:.4f}s")   # ~0.001s
# Set is 5000x faster!
```

### Practical Examples

```python
# Bad: Using list for filtering
def filter_valid_users(users, valid_ids):
    return [u for u in users if u.id in valid_ids]  # O(n*m)

# Good: Convert to set first
def filter_valid_users(users, valid_ids):
    valid_set = set(valid_ids)  # O(m)
    return [u for u in users if u.id in valid_set]  # O(n)

# Much faster for large lists
```

### When to Use Sets

```python
# Use sets for:
# - Membership testing
# - Removing duplicates
# - Set operations (union, intersection)
# - Tracking unique items

# Remove duplicates
items = [1, 2, 2, 3, 3, 3, 4]
unique = list(set(items))  # [1, 2, 3, 4] (order not preserved)

# Track seen items
def find_duplicates(items):
    seen = set()
    duplicates = set()
    for item in items:
        if item in seen:
            duplicates.add(item)
        seen.add(item)
    return duplicates
```

## Tuples and Immutability

**Tuple**: Immutable sequence stored as fixed-size array in CPython.

### Tuples vs Lists

```python
# Tuple: Fixed size, immutable
point = (10, 20)

# List: Dynamic size, mutable
numbers = [10, 20]

# Memory:
import sys
print(sys.getsizeof(point))    # 64 bytes
print(sys.getsizeof(numbers))  # 88 bytes

# Tuples are slightly smaller (no resize overhead)
```

### Tuple Performance

```python
import timeit

# Tuple creation slightly faster
tuple_time = timeit.timeit('(1, 2, 3, 4, 5)', number=1000000)
list_time = timeit.timeit('[1, 2, 3, 4, 5]', number=1000000)

print(f"Tuple: {tuple_time:.4f}s")  # ~0.08s
print(f"List:  {list_time:.4f}s")   # ~0.19s

# Access speed similar
data_tuple = (1, 2, 3, 4, 5)
data_list = [1, 2, 3, 4, 5]

tuple_access = timeit.timeit('data[2]', globals={'data': data_tuple}, number=1000000)
list_access = timeit.timeit('data[2]', globals={'data': data_list}, number=1000000)
# Nearly identical
```

### When to Use Tuples

```python
# Use tuples for:
# - Fixed collections (coordinates, RGB values)
# - Dict keys (must be immutable)
# - Function returns (multiple values)
# - Data that shouldn't change

# Coordinate
point = (10, 20)

# RGB color
color = (255, 128, 0)

# Dict key
locations = {
    (0, 0): 'origin',
    (10, 20): 'point A',
}

# Multiple returns
def get_user_info():
    return ('Alice', 30, 'alice@example.com')
```

## Collections Module Alternatives

### deque (Double-Ended Queue)

**deque**: Double-ended queue optimized for fast appends/pops from both ends.

```python
from collections import deque

# List: Fast at end, slow at beginning
items_list = [1, 2, 3]
items_list.append(4)      # Fast
items_list.insert(0, 0)   # Slow (shifts all)

# deque: Fast at both ends
items_deque = deque([1, 2, 3])
items_deque.append(4)     # Fast
items_deque.appendleft(0) # Fast (no shifting!)

# Use deque for:
# - Queue operations (FIFO)
# - Stack operations (LIFO)
# - Sliding windows
# - Recent history

# Queue example
queue = deque()
queue.append('task1')     # Enqueue
queue.append('task2')
task = queue.popleft()    # Dequeue (fast!)

# Maxlen for fixed-size buffer
recent = deque(maxlen=5)
for i in range(10):
    recent.append(i)  # Automatically removes oldest
print(recent)  # deque([5, 6, 7, 8, 9])
```

### Counter

**Counter**: Dict subclass for counting hashable objects.

```python
from collections import Counter

# Count occurrences
words = ['apple', 'banana', 'apple', 'cherry', 'banana', 'apple']

# Manual counting (verbose)
counts = {}
for word in words:
    counts[word] = counts.get(word, 0) + 1

# Counter (clean)
counts = Counter(words)
print(counts)  # Counter({'apple': 3, 'banana': 2, 'cherry': 1})

# Useful methods
counts.most_common(2)     # [('apple', 3), ('banana', 2)]
counts.total()            # 6
counts['apple']           # 3
counts['orange']          # 0 (default, not KeyError)

# Operations
c1 = Counter(['a', 'b', 'c'])
c2 = Counter(['b', 'c', 'd'])
c1 + c2  # Add counts
c1 - c2  # Subtract counts
```

### defaultdict

**defaultdict**: Dict that provides default values for missing keys.

```python
from collections import defaultdict

# Regular dict requires checking
groups = {}
for item in items:
    if item.category not in groups:
        groups[item.category] = []
    groups[item.category].append(item)

# defaultdict is cleaner
groups = defaultdict(list)
for item in items:
    groups[item.category].append(item)  # Auto-creates list

# Useful for:
# - Grouping items
# - Building graphs
# - Counting (use Counter instead)

# Example: Group by first letter
words = ['apple', 'banana', 'apricot', 'berry', 'cherry']
by_letter = defaultdict(list)
for word in words:
    by_letter[word[0]].append(word)
# {'a': ['apple', 'apricot'], 'b': ['banana', 'berry'], 'c': ['cherry']}
```

### OrderedDict

```python
from collections import OrderedDict

# Note: Regular dicts maintain insertion order since Python 3.7
# OrderedDict mainly useful for:
# - Backward compatibility
# - Equality comparison includes order
# - move_to_end() method

d1 = {'a': 1, 'b': 2}
d2 = {'b': 2, 'a': 1}
print(d1 == d2)  # True (order doesn't matter)

od1 = OrderedDict([('a', 1), ('b', 2)])
od2 = OrderedDict([('b', 2), ('a', 1)])
print(od1 == od2)  # False (order matters)

# move_to_end useful for LRU caches
cache = OrderedDict()
cache['key1'] = 'value1'
cache['key2'] = 'value2'
cache.move_to_end('key1')  # Move to end (most recent)
```

## Choosing the Right Structure

### Quick Reference

```python
# Fast lookups by key → dict
user_by_id = {user.id: user for user in users}
user = user_by_id[123]

# Fast membership testing → set
valid_ids = {1, 2, 3, 4, 5}
if user.id in valid_ids:  # Fast

# Sequential access → list
items = [1, 2, 3, 4, 5]
for item in items:  # Fast iteration

# Fast insertions at both ends → deque
from collections import deque
queue = deque()
queue.append(item)      # Fast
queue.appendleft(item)  # Fast

# Counting occurrences → Counter
from collections import Counter
counts = Counter(items)

# Fixed, immutable data → tuple
coordinate = (x, y, z)
```

### Performance Comparison

```python
# Membership testing (10,000 items, checking 10,000 times)
# List:  ~8 seconds
# Set:   ~0.001 seconds

# Insert at beginning (10,000 operations)
# List:  ~5 seconds
# deque: ~0.001 seconds

# Lookup by key (10,000 lookups)
# List:  ~8 seconds
# Dict:  ~0.001 seconds

# General rule:
# - Hash tables (dict, set) for lookups: O(1)
# - Dynamic arrays (list) for sequential: O(1) at end
# - deque for both ends: O(1)
```

### Real-World Example

```python
# Bad: Using wrong structures
def process_events(events):
    # List for lookups (slow)
    seen_ids = []

    # List for frequent removals (slow)
    pending = []

    for event in events:
        if event.id in seen_ids:  # O(n) - slow!
            continue
        seen_ids.append(event.id)

        if event.requires_processing:
            pending.insert(0, event)  # O(n) - slow!

# Good: Using right structures
def process_events(events):
    # Set for membership testing
    seen_ids = set()

    # deque for frequent insertions
    pending = deque()

    for event in events:
        if event.id in seen_ids:  # O(1) - fast!
            continue
        seen_ids.add(event.id)

        if event.requires_processing:
            pending.appendleft(event)  # O(1) - fast!
```

## Summary

Python data structure performance:

**Lists (dynamic arrays):**

- Fast: append, pop (end), indexing, iteration
- Slow: insert/pop (beginning), membership testing
- Use for: Sequential access, order matters

**Dicts (hash tables):**

- Fast: key lookup, insert, delete, membership
- Memory intensive, keys must be hashable
- Use for: Fast lookups, key-value mapping

**Sets (hash tables):**

- Fast: membership testing, add, remove, set operations
- Unordered, elements must be hashable
- Use for: Membership tests, uniqueness, set operations

**Tuples (immutable arrays):**

- Slightly faster creation, less memory
- Immutable, can be dict keys
- Use for: Fixed data, dict keys, returns

**Collections module:**

- deque: Fast both ends (queue, stack)
- Counter: Counting occurrences
- defaultdict: Auto-default values
- OrderedDict: Order-sensitive equality

Choose structures based on access patterns, not just familiarity.

## Next Steps

- Master [Profiling and Measurement](profiling-and-measurement.md)
- Learn [Optimization Techniques](optimization-techniques.md)
- Explore [Memory Optimization](memory-optimization.md)
