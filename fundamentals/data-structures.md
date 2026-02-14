# Data Structures

## Table of Contents

- [TL;DR](#tldr)
- [Lists](#lists)
- [Tuples](#tuples)
- [Dictionaries](#dictionaries)
- [Sets](#sets)
- [Choosing the Right Data Structure](#choosing-the-right-data-structure)
- [Summary](#summary)
- [Next Steps](#next-steps)

## TL;DR

Python's built-in data structures: **Lists** - ordered, mutable sequences `[1, 2, 3]`, support indexing/slicing, methods like append/extend/insert/remove. **Tuples** - ordered, immutable sequences `(1, 2, 3)`, faster than lists, used for fixed data. **Dictionaries** - key-value pairs `{'a': 1, 'b': 2}`, O(1) lookup, keys must be immutable. **Sets** - unordered, unique elements `{1, 2, 3}`, O(1) membership testing, support union/intersection/difference. Use lists for ordered mutable data, tuples for immutable records, dicts for lookups, sets for unique collections and membership tests.

## Lists

### Creating Lists

```python
# Empty list
empty = []
also_empty = list()

# List literal
numbers = [1, 2, 3, 4, 5]
mixed = [1, "hello", 3.14, True]

# From other iterables
chars = list("abc")  # ['a', 'b', 'c']
range_list = list(range(5))  # [0, 1, 2, 3, 4]

# Nested lists
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
```

### Accessing Elements

```python
fruits = ['apple', 'banana', 'cherry', 'date']

# Indexing (0-based)
fruits[0]   # 'apple'
fruits[2]   # 'cherry'

# Negative indexing (from end)
fruits[-1]  # 'date'
fruits[-2]  # 'cherry'

# Slicing [start:end:step]
fruits[1:3]    # ['banana', 'cherry']
fruits[:2]     # ['apple', 'banana']
fruits[2:]     # ['cherry', 'date']
fruits[::2]    # ['apple', 'cherry'] (every 2nd)
fruits[::-1]   # ['date', 'cherry', 'banana', 'apple'] (reversed)

# Length
len(fruits)  # 4
```

### Modifying Lists

Lists are **mutable** - can be changed in place:

```python
numbers = [1, 2, 3]

# Change element
numbers[0] = 10  # [10, 2, 3]

# Change slice
numbers[1:3] = [20, 30, 40]  # [10, 20, 30, 40]

# Append (add to end)
numbers.append(50)  # [10, 20, 30, 40, 50]

# Extend (add multiple items)
numbers.extend([60, 70])  # [10, 20, 30, 40, 50, 60, 70]
numbers += [80]  # Same as extend

# Insert at position
numbers.insert(0, 5)  # [5, 10, 20, 30, 40, 50, 60, 70, 80]

# Remove by value (first occurrence)
numbers.remove(30)  # [5, 10, 20, 40, 50, 60, 70, 80]

# Remove by index (and return it)
item = numbers.pop(2)  # item=20, list=[5, 10, 40, 50, 60, 70, 80]
last = numbers.pop()   # last=80, list=[5, 10, 40, 50, 60, 70]

# Clear all elements
numbers.clear()  # []

# Delete with del
items = [1, 2, 3, 4, 5]
del items[0]     # [2, 3, 4, 5]
del items[1:3]   # [2, 5]
```

### List Operations

```python
# Concatenation
[1, 2] + [3, 4]  # [1, 2, 3, 4]

# Repetition
[1, 2] * 3  # [1, 2, 1, 2, 1, 2]

# Membership
2 in [1, 2, 3]  # True
5 not in [1, 2, 3]  # True

# Count occurrences
[1, 2, 2, 3, 2].count(2)  # 3

# Find index (first occurrence)
[1, 2, 3, 2].index(2)  # 1

# Sorting
numbers = [3, 1, 4, 1, 5]
numbers.sort()  # Sorts in place: [1, 1, 3, 4, 5]
numbers.sort(reverse=True)  # [5, 4, 3, 1, 1]

# Sort returns None!
result = numbers.sort()  # result is None

# sorted() returns new list
original = [3, 1, 4]
sorted_list = sorted(original)  # [1, 3, 4]
# original unchanged: [3, 1, 4]

# Reverse
numbers = [1, 2, 3]
numbers.reverse()  # [3, 2, 1] (in place)

# reversed() returns iterator
rev = reversed([1, 2, 3])  # iterator
list(rev)  # [3, 2, 1]
```

### List Comprehensions

Create lists concisely:

```python
# Basic form: [expression for item in iterable]
squares = [x**2 for x in range(5)]  # [0, 1, 4, 9, 16]

# With condition: [expression for item in iterable if condition]
evens = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8]

# Transform and filter
words = ["hello", "world", "python"]
upper_long = [w.upper() for w in words if len(w) > 5]  # ['PYTHON']

# Nested comprehensions
matrix = [[1, 2, 3], [4, 5, 6]]
flattened = [item for row in matrix for item in row]  # [1, 2, 3, 4, 5, 6]
```

### Copying Lists

```python
original = [1, 2, 3]

# Reference (not a copy!)
ref = original
ref.append(4)
print(original)  # [1, 2, 3, 4] - modified!

# Shallow copy
copy1 = original.copy()
copy2 = original[:]
copy3 = list(original)

copy1.append(5)
print(original)  # [1, 2, 3, 4] - unchanged

# Deep copy (for nested structures)
import copy
nested = [[1, 2], [3, 4]]
shallow = nested.copy()
deep = copy.deepcopy(nested)

shallow[0].append(99)
print(nested)  # [[1, 2, 99], [3, 4]] - inner list shared!

deep[1].append(88)
print(nested)  # [[1, 2, 99], [3, 4]] - unchanged
```

## Tuples

### Creating Tuples

```python
# Empty tuple
empty = ()
also_empty = tuple()

# Tuple literal
numbers = (1, 2, 3, 4, 5)
mixed = (1, "hello", 3.14, True)

# Single element tuple (note the comma!)
single = (42,)  # Tuple with one element
not_tuple = (42)  # Just an int in parentheses

# Without parentheses (tuple packing)
coords = 10, 20  # (10, 20)

# From iterable
letters = tuple("abc")  # ('a', 'b', 'c')

# Nested tuples
nested = ((1, 2), (3, 4), (5, 6))
```

### Accessing Tuples

Same as lists - indexing and slicing:

```python
point = (10, 20, 30)

point[0]   # 10
point[-1]  # 30
point[1:3] # (20, 30)

len(point)  # 3
```

### Tuple Immutability

Tuples cannot be modified after creation:

```python
coords = (10, 20)

# These all fail with TypeError
coords[0] = 15  # Can't assign
coords.append(30)  # No append method
del coords[0]  # Can't delete element

# But can create new tuple
coords = (15, 20)  # Rebind name to new tuple
```

**Note:** If tuple contains mutable objects, those can be modified:

```python
tuple_with_list = ([1, 2], [3, 4])
tuple_with_list[0].append(99)  # OK - modifying the list
print(tuple_with_list)  # ([1, 2, 99], [3, 4])

tuple_with_list[0] = [5, 6]  # TypeError - can't reassign tuple element
```

### Tuple Operations

```python
# Concatenation
(1, 2) + (3, 4)  # (1, 2, 3, 4)

# Repetition
(1, 2) * 3  # (1, 2, 1, 2, 1, 2)

# Membership
2 in (1, 2, 3)  # True

# Count and index
(1, 2, 2, 3).count(2)  # 2
(1, 2, 3, 2).index(2)  # 1
```

### Tuple Unpacking

```python
# Basic unpacking
point = (10, 20)
x, y = point  # x=10, y=20

# Multiple assignment
a, b, c = 1, 2, 3  # a=1, b=2, c=3

# Swap values
a, b = b, a

# * for remaining items
first, *rest = [1, 2, 3, 4]  # first=1, rest=[2, 3, 4]
first, *middle, last = [1, 2, 3, 4, 5]  # first=1, middle=[2, 3, 4], last=5

# Ignore values with _
x, _, z = (1, 2, 3)  # Ignore the 2
```

### When to Use Tuples

**Use tuples for:**

- Fixed data that shouldn't change (e.g., coordinates, RGB colors)
- Dictionary keys (must be immutable)
- Returning multiple values from functions
- Unpacking assignments

**Use lists for:**

- Collections that may grow or shrink
- Data that needs modification
- When order matters and you need mutability

**Performance:** Tuples are slightly faster than lists and use less memory.

## Dictionaries

### Creating Dictionaries

```python
# Empty dictionary
empty = {}
also_empty = dict()

# Dictionary literal
person = {"name": "Alice", "age": 30, "city": "NYC"}

# dict() constructor
person = dict(name="Alice", age=30, city="NYC")

# From key-value pairs
pairs = [("a", 1), ("b", 2), ("c", 3)]
d = dict(pairs)  # {'a': 1, 'b': 2, 'c': 3}

# Dictionary comprehension
squares = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

### Accessing Values

```python
person = {"name": "Alice", "age": 30}

# Direct access (KeyError if missing)
person["name"]  # 'Alice'
person["city"]  # KeyError!

# get() method (returns None or default if missing)
person.get("name")  # 'Alice'
person.get("city")  # None
person.get("city", "Unknown")  # 'Unknown'

# Check if key exists
"name" in person  # True
"city" not in person  # True
```

### Modifying Dictionaries

```python
person = {"name": "Alice", "age": 30}

# Add or update
person["city"] = "NYC"  # Add new key
person["age"] = 31  # Update existing key

# Update multiple items
person.update({"age": 32, "job": "Engineer"})

# Remove by key (returns value)
age = person.pop("age")  # age=32, key removed

# Remove by key (doesn't return value)
del person["city"]

# Remove and return arbitrary item
person["x"] = 1
item = person.popitem()  # ('x', 1) in Python 3.7+

# Clear all items
person.clear()  # {}

# setdefault - get value, set if missing
counts = {}
counts.setdefault("a", 0)  # Returns 0, sets if key missing
counts.setdefault("a", 5)  # Returns 0 (already exists)
```

### Dictionary Methods

```python
person = {"name": "Alice", "age": 30, "city": "NYC"}

# Get all keys
person.keys()  # dict_keys(['name', 'age', 'city'])
list(person.keys())  # ['name', 'age', 'city']

# Get all values
person.values()  # dict_values(['Alice', 30, 'NYC'])

# Get key-value pairs
person.items()  # dict_items([('name', 'Alice'), ('age', 30), ('city', 'NYC')])

# Iterate
for key in person:
    print(key, person[key])

for key, value in person.items():
    print(f"{key}: {value}")

# Copy (shallow)
person_copy = person.copy()
```

### Dictionary Comprehensions

```python
# Basic
{x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# With condition
{x: x**2 for x in range(10) if x % 2 == 0}  # {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}

# Swap keys and values
original = {"a": 1, "b": 2}
swapped = {v: k for k, v in original.items()}  # {1: 'a', 2: 'b'}
```

### Dictionary Ordering

Python 3.7+ maintains insertion order:

```python
d = {}
d['b'] = 2
d['a'] = 1
d['c'] = 3

list(d.keys())  # ['b', 'a', 'c'] - order preserved
```

### Nested Dictionaries

```python
users = {
    "alice": {"age": 30, "city": "NYC"},
    "bob": {"age": 25, "city": "LA"}
}

users["alice"]["age"]  # 30
users["charlie"] = {"age": 35, "city": "Chicago"}
```

## Sets

### Creating Sets

```python
# Empty set (must use set(), not {})
empty = set()

# Set literal
numbers = {1, 2, 3, 4, 5}

# From iterable
letters = set("hello")  # {'h', 'e', 'l', 'o'} - duplicates removed

# Set comprehension
evens = {x for x in range(10) if x % 2 == 0}  # {0, 2, 4, 6, 8}
```

**Note:** Sets are **unordered** and contain **unique elements only**.

```python
{1, 2, 2, 3, 3, 3}  # {1, 2, 3} - duplicates removed
```

### Set Operations

```python
# Add element
numbers = {1, 2, 3}
numbers.add(4)  # {1, 2, 3, 4}
numbers.add(2)  # {1, 2, 3, 4} - no effect, already exists

# Remove element
numbers.remove(3)  # {1, 2, 4} (KeyError if not found)
numbers.discard(10)  # No error if not found

# Pop random element
item = numbers.pop()

# Clear all
numbers.clear()  # set()

# Membership (O(1) - very fast!)
3 in {1, 2, 3, 4}  # True

# Length
len({1, 2, 3})  # 3
```

### Mathematical Set Operations

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

# Union (all elements from both)
a | b  # {1, 2, 3, 4, 5, 6}
a.union(b)

# Intersection (elements in both)
a & b  # {3, 4}
a.intersection(b)

# Difference (in a but not in b)
a - b  # {1, 2}
a.difference(b)

# Symmetric difference (in either but not both)
a ^ b  # {1, 2, 5, 6}
a.symmetric_difference(b)

# Subset
{1, 2} < {1, 2, 3}  # True
{1, 2}.issubset({1, 2, 3})

# Superset
{1, 2, 3} > {1, 2}  # True
{1, 2, 3}.issuperset({1, 2})

# Disjoint (no common elements)
{1, 2}.isdisjoint({3, 4})  # True
```

### Frozen Sets

Immutable version of sets (can be dictionary keys):

```python
fs = frozenset([1, 2, 3])

# Can't modify
fs.add(4)  # AttributeError

# Can use as dict key
d = {fs: "value"}

# Can perform read operations
fs | {4, 5}  # Returns new frozenset: frozenset({1, 2, 3, 4, 5})
```

## Choosing the Right Data Structure

| Use Case                             | Data Structure | Why                                      |
| ------------------------------------ | -------------- | ---------------------------------------- |
| Ordered collection, need to modify   | **List**       | Mutable, ordered, allows duplicates      |
| Ordered collection, shouldn't change | **Tuple**      | Immutable, ordered, faster than list     |
| Key-value lookups                    | **Dictionary** | O(1) access by key                       |
| Unique elements, membership tests    | **Set**        | O(1) membership, automatic uniqueness    |
| Mathematical set operations          | **Set**        | Union, intersection, difference built-in |
| Sequence of same type (large data)   | **List**       | Simple, efficient for homogeneous data   |
| Multiple return values               | **Tuple**      | Conventional, unpacks easily             |
| Config data that shouldn't change    | **Dictionary** | Named access, clear structure            |

**Performance comparison:**

```python
# Membership testing
'x' in my_list  # O(n) - slow
'x' in my_set   # O(1) - fast

# Lookup by key
my_dict['key']  # O(1) - fast

# Append
my_list.append(x)  # O(1) - fast

# Insert at beginning
my_list.insert(0, x)  # O(n) - slow (shifts all elements)
```

## Summary

Python's core data structures serve different purposes:

**Lists:**

- Ordered, mutable sequences
- Support indexing, slicing, sorting
- Use for ordered collections that change

**Tuples:**

- Ordered, immutable sequences
- Faster than lists, less memory
- Use for fixed data, multiple returns, dict keys

**Dictionaries:**

- Key-value pairs, O(1) lookup
- Ordered (Python 3.7+)
- Use for lookups, mappings, structured data

**Sets:**

- Unordered, unique elements, O(1) membership
- Mathematical set operations
- Use for uniqueness, membership tests, set math

Choose based on your needs: mutability, ordering, lookup speed, and whether you need keys or just values.

## Next Steps

- Use these structures with [Functions](functions.md) for complex operations
- Combine with [Control Flow](control-flow.md) for data processing
- Learn more advanced patterns in [Core Concepts](../core_concepts/)
