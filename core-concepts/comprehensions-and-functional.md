# Comprehensions and Functional Tools

## Table of Contents

- [List Comprehensions](#list-comprehensions)
- [Dictionary Comprehensions](#dictionary-comprehensions)
- [Set Comprehensions](#set-comprehensions)
- [Generator Expressions](#generator-expressions)
- [Lambda Functions](#lambda-functions)
- [Map Function](#map-function)
- [Filter Function](#filter-function)
- [Reduce Function](#reduce-function)
- [Combining Functional Tools](#combining-functional-tools)
- [Performance Considerations](#performance-considerations)
- [Summary](#summary)
- [Next Steps](#next-steps)

## List Comprehensions

### Basic Syntax

Concise way to create lists from iterables:

```python
# Traditional approach
squares = []
for x in range(10):
    squares.append(x ** 2)

# List comprehension
squares = [x ** 2 for x in range(10)]
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

**Syntax:** `[expression for item in iterable]`

### With Conditionals

Filter items with `if`:

```python
# Only even numbers
evens = [x for x in range(20) if x % 2 == 0]
# [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

# Squares of odd numbers
odd_squares = [x ** 2 for x in range(10) if x % 2 == 1]
# [1, 9, 25, 49, 81]
```

### Nested Comprehensions

```python
# Flatten a matrix
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [num for row in matrix for num in row]
# [1, 2, 3, 4, 5, 6, 7, 8, 9]

# Create a matrix
matrix = [[i * j for j in range(3)] for i in range(3)]
# [[0, 0, 0], [0, 1, 2], [0, 2, 4]]
```

### String Operations

```python
# Convert to uppercase
words = ['hello', 'world', 'python']
upper = [word.upper() for word in words]
# ['HELLO', 'WORLD', 'PYTHON']

# Extract first letters
initials = [word[0] for word in words]
# ['h', 'w', 'p']

# Filter by length
long_words = [word for word in words if len(word) > 5]
# ['python']
```

### Multiple Conditions

```python
# Using if-else (ternary)
numbers = [1, 2, 3, 4, 5, 6]
result = ['even' if x % 2 == 0 else 'odd' for x in numbers]
# ['odd', 'even', 'odd', 'even', 'odd', 'even']

# Multiple filters
result = [x for x in range(20) if x % 2 == 0 if x % 3 == 0]
# [0, 6, 12, 18]
```

## Dictionary Comprehensions

### Basic Syntax

Create dictionaries from iterables:

```python
# Square numbers as dict
squares = {x: x ** 2 for x in range(6)}
# {0: 0, 1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# From two lists
keys = ['a', 'b', 'c']
values = [1, 2, 3]
d = {k: v for k, v in zip(keys, values)}
# {'a': 1, 'b': 2, 'c': 3}
```

**Syntax:** `{key_expr: value_expr for item in iterable}`

### Transforming Dictionaries

```python
# Swap keys and values
original = {'a': 1, 'b': 2, 'c': 3}
swapped = {v: k for k, v in original.items()}
# {1: 'a', 2: 'b', 3: 'c'}

# Filter dictionary
prices = {'apple': 0.50, 'banana': 0.30, 'orange': 0.60}
expensive = {k: v for k, v in prices.items() if v > 0.40}
# {'apple': 0.50, 'orange': 0.60}

# Transform values
doubled = {k: v * 2 for k, v in prices.items()}
# {'apple': 1.0, 'banana': 0.6, 'orange': 1.2}
```

### String Manipulation

```python
# Word lengths
words = ['hello', 'world', 'python']
lengths = {word: len(word) for word in words}
# {'hello': 5, 'world': 5, 'python': 6}

# Character counts
text = "hello"
char_count = {char: text.count(char) for char in text}
# {'h': 1, 'e': 1, 'l': 2, 'o': 1}
```

## Set Comprehensions

### Basic Syntax

Create sets from iterables:

```python
# Unique squares
squares = {x ** 2 for x in range(-5, 6)}
# {0, 1, 4, 9, 16, 25} - duplicates removed

# Unique characters
text = "hello world"
chars = {c for c in text if c.isalpha()}
# {'h', 'e', 'l', 'o', 'w', 'r', 'd'}
```

**Syntax:** `{expression for item in iterable}`

### Deduplication

```python
# Remove duplicates while transforming
numbers = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
unique_doubles = {x * 2 for x in numbers}
# {2, 4, 6, 8}

# Case-insensitive unique words
words = ['Hello', 'WORLD', 'hello', 'Python', 'world']
unique = {word.lower() for word in words}
# {'hello', 'world', 'python'}
```

## Generator Expressions

### Basic Syntax

Like list comprehensions but produce items lazily:

```python
# List comprehension - creates entire list in memory
squares_list = [x ** 2 for x in range(1000000)]

# Generator expression - produces one at a time
squares_gen = (x ** 2 for x in range(1000000))

# Use in iteration
for square in squares_gen:
    if square > 100:
        break
    print(square)
```

**Syntax:** `(expression for item in iterable)` - note parentheses instead of brackets

### Memory Efficiency

```python
# Sum large sequence efficiently
total = sum(x ** 2 for x in range(1000000))

# Generator passed directly to function
max_square = max(x ** 2 for x in range(100))

# Chain operations
result = sum(x for x in range(1000) if x % 3 == 0 if x % 5 == 0)
```

### Converting to Collections

```python
# Generator to list
gen = (x ** 2 for x in range(10))
squares = list(gen)
# [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Generator exhausted after one use
gen = (x for x in range(5))
list1 = list(gen)  # [0, 1, 2, 3, 4]
list2 = list(gen)  # [] - generator exhausted
```

## Lambda Functions

### Anonymous Functions

Small, one-expression functions:

```python
# Regular function
def square(x):
    return x ** 2

# Lambda equivalent
square = lambda x: x ** 2
print(square(5))  # 25
```

**Syntax:** `lambda parameters: expression`

### Multiple Parameters

```python
# Addition
add = lambda a, b: a + b
add(3, 5)  # 8

# Multiple operations
calculate = lambda x, y: (x + y, x - y, x * y, x / y)
calculate(10, 5)  # (15, 5, 50, 2.0)
```

### Common Use Cases

**Sorting with custom key:**

```python
# Sort by second element
pairs = [(1, 'one'), (3, 'three'), (2, 'two')]
sorted_pairs = sorted(pairs, key=lambda x: x[1])
# [(1, 'one'), (3, 'three'), (2, 'two')]

# Sort by length
words = ['python', 'is', 'awesome']
sorted_words = sorted(words, key=lambda w: len(w))
# ['is', 'python', 'awesome']

# Sort objects
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

people = [Person('Alice', 30), Person('Bob', 25)]
sorted_people = sorted(people, key=lambda p: p.age)
```

**Conditional logic:**

```python
# Max of two values
maximum = lambda a, b: a if a > b else b
maximum(10, 20)  # 20

# Absolute value
abs_val = lambda x: x if x >= 0 else -x
abs_val(-5)  # 5
```

### Limitations and Best Practices

**Limited to single expressions:**

```python
# Can't do this in lambda
def complex_func(x):
    result = x ** 2
    if result > 10:
        return result
    return 0

# This won't work as lambda
```

**When to use lambdas:**

- Simple, one-line operations
- Short-lived callback functions
- Sorting keys
- Functional programming patterns

**When to use regular functions:**

- Multiple statements needed
- Complex logic
- Need documentation
- Function used in multiple places

## Map Function

### Basic Usage

Apply function to every item in iterable:

```python
# Square all numbers
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x ** 2, numbers))
# [1, 4, 9, 16, 25]

# Convert to strings
nums = [1, 2, 3, 4]
strings = list(map(str, nums))
# ['1', '2', '3', '4']

# Uppercase strings
words = ['hello', 'world']
upper = list(map(str.upper, words))
# ['HELLO', 'WORLD']
```

**Syntax:** `map(function, iterable, ...)`

### Multiple Iterables

```python
# Add corresponding elements
a = [1, 2, 3]
b = [10, 20, 30]
result = list(map(lambda x, y: x + y, a, b))
# [11, 22, 33]

# Multiply pairs
nums1 = [1, 2, 3, 4]
nums2 = [2, 3, 4, 5]
products = list(map(lambda x, y: x * y, nums1, nums2))
# [2, 6, 12, 20]
```

### Map vs Comprehension

```python
# Using map
result = list(map(lambda x: x ** 2, range(10)))

# Using comprehension (often more Pythonic)
result = [x ** 2 for x in range(10)]

# Comprehension more readable for complex logic
result = [x ** 2 for x in range(20) if x % 2 == 0]
```

## Filter Function

### Basic Usage

Select items that match condition:

```python
# Get even numbers
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens = list(filter(lambda x: x % 2 == 0, numbers))
# [2, 4, 6, 8, 10]

# Filter None values
values = [1, None, 2, None, 3, 4]
clean = list(filter(None, values))  # None is default predicate
# [1, 2, 3, 4]

# Filter strings by length
words = ['hi', 'hello', 'hey', 'python']
long_words = list(filter(lambda w: len(w) > 3, words))
# ['hello', 'python']
```

**Syntax:** `filter(function, iterable)`

### Complex Filtering

```python
# Multiple conditions
numbers = range(1, 101)
result = list(filter(lambda x: x % 3 == 0 and x % 5 == 0, numbers))
# [15, 30, 45, 60, 75, 90]

# Filter objects
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age

people = [Person('Alice', 30), Person('Bob', 17), Person('Charlie', 25)]
adults = list(filter(lambda p: p.age >= 18, people))
```

### Filter vs Comprehension

```python
# Using filter
result = list(filter(lambda x: x % 2 == 0, range(20)))

# Using comprehension (more readable)
result = [x for x in range(20) if x % 2 == 0]
```

## Reduce Function

### Basic Usage

Reduce iterable to single value by applying function cumulatively:

```python
from functools import reduce

# Sum all numbers
numbers = [1, 2, 3, 4, 5]
total = reduce(lambda acc, x: acc + x, numbers)
# 15

# Product of all numbers
product = reduce(lambda acc, x: acc * x, numbers)
# 120

# Find maximum
maximum = reduce(lambda a, b: a if a > b else b, numbers)
# 5
```

**Syntax:** `reduce(function, iterable, [initial])`

### With Initial Value

```python
from functools import reduce

# Sum with initial value
numbers = [1, 2, 3, 4]
total = reduce(lambda acc, x: acc + x, numbers, 10)
# 20 (10 + 1 + 2 + 3 + 4)

# Build dictionary
items = [('a', 1), ('b', 2), ('c', 3)]
d = reduce(lambda acc, item: {**acc, item[0]: item[1]}, items, {})
# {'a': 1, 'b': 2, 'c': 3}
```

### Practical Examples

```python
from functools import reduce

# Flatten list of lists
lists = [[1, 2], [3, 4], [5, 6]]
flat = reduce(lambda acc, lst: acc + lst, lists, [])
# [1, 2, 3, 4, 5, 6]

# String concatenation
words = ['Hello', ' ', 'World', '!']
sentence = reduce(lambda acc, word: acc + word, words)
# 'Hello World!'

# Find GCD of multiple numbers
import math
numbers = [48, 64, 128, 256]
gcd = reduce(math.gcd, numbers)
# 16
```

### Alternatives to Reduce

Many uses have built-in alternatives:

```python
# Sum - use built-in
total = sum([1, 2, 3, 4, 5])  # Better than reduce

# Product - use math.prod (Python 3.8+)
import math
product = math.prod([1, 2, 3, 4, 5])

# Max/min - use built-in
maximum = max([1, 2, 3, 4, 5])
minimum = min([1, 2, 3, 4, 5])
```

## Combining Functional Tools

### Chaining Operations

```python
# Traditional approach
numbers = range(1, 11)
evens = filter(lambda x: x % 2 == 0, numbers)
squared = map(lambda x: x ** 2, evens)
result = list(squared)
# [4, 16, 36, 64, 100]

# Combined
result = list(map(lambda x: x ** 2, filter(lambda x: x % 2 == 0, range(1, 11))))

# Using comprehension (more readable)
result = [x ** 2 for x in range(1, 11) if x % 2 == 0]
```

### Complex Pipelines

```python
from functools import reduce

# Process data pipeline
data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

# Filter evens, square them, sum result
result = reduce(
    lambda acc, x: acc + x,
    map(lambda x: x ** 2, filter(lambda x: x % 2 == 0, data)),
    0
)
# 220

# Same with comprehension
result = sum(x ** 2 for x in data if x % 2 == 0)
```

### Real-World Example

```python
# Process user data
users = [
    {'name': 'Alice', 'age': 30, 'active': True},
    {'name': 'Bob', 'age': 17, 'active': True},
    {'name': 'Charlie', 'age': 25, 'active': False},
    {'name': 'Diana', 'age': 35, 'active': True},
]

# Get names of active adult users
# Using functional approach
adult_active_names = list(map(
    lambda u: u['name'],
    filter(lambda u: u['age'] >= 18 and u['active'], users)
))
# ['Alice', 'Diana']

# Using comprehension (more Pythonic)
adult_active_names = [
    u['name'] for u in users
    if u['age'] >= 18 and u['active']
]
```

## Performance Considerations

### Comprehensions vs map/filter

```python
import timeit

# List comprehension
def use_comprehension():
    return [x ** 2 for x in range(1000)]

# Map with lambda
def use_map():
    return list(map(lambda x: x ** 2, range(1000)))

# Comprehension is usually faster and more readable
```

### Memory: Lists vs Generators

```python
import sys

# List comprehension - stores all in memory
squares_list = [x ** 2 for x in range(10000)]
print(sys.getsizeof(squares_list))  # ~80KB

# Generator - produces on demand
squares_gen = (x ** 2 for x in range(10000))
print(sys.getsizeof(squares_gen))  # ~128 bytes

# Use generator when:
# - Processing large datasets
# - Only iterating once
# - Memory constrained

# Use list when:
# - Need to iterate multiple times
# - Need indexing or slicing
# - Dataset is small
```

### When to Use What

**List comprehensions:** Most Pythonic for simple transformations

```python
[x ** 2 for x in numbers if x > 0]
```

**Generator expressions:** Large datasets, one-time iteration

```python
sum(x ** 2 for x in huge_dataset)
```

**map/filter:** When you have existing functions

```python
list(map(str.upper, words))
list(filter(str.isalpha, chars))
```

**Lambda:** Short, throwaway functions

```python
sorted(items, key=lambda x: x[1])
```

**Regular functions:** Complex logic, reusability, documentation

```python
def process_item(item):
    """Process item with complex logic."""
    # Multiple steps...
    return result

list(map(process_item, items))
```

## Summary

Comprehensions and functional tools provide powerful, expressive ways to work with data:

**Comprehensions:**

- List: `[expr for item in iterable if condition]`
- Dict: `{key: value for item in iterable}`
- Set: `{expr for item in iterable}`
- Generator: `(expr for item in iterable)`

**Functional tools:**

- Lambda: `lambda params: expression` - anonymous functions
- Map: `map(func, iterable)` - transform each item
- Filter: `filter(func, iterable)` - select items
- Reduce: `reduce(func, iterable)` - combine to single value

**Best practices:**

- Prefer comprehensions for readability
- Use generators for large datasets
- Use built-ins (sum, max, min) over reduce
- Choose regular functions for complex logic

These tools enable concise, functional-style Python code while maintaining readability.

## Next Steps

- Apply OOP concepts in [Object-Oriented Programming](object-oriented-programming.md)
- Explore advanced function patterns in [Advanced Functions](advanced-functions.md)
- Learn lazy evaluation with [Iterators and Generators](iterators-and-generators.md)
