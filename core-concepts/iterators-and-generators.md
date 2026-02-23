# Iterators and Generators

## Table of Contents

- [The Iteration Protocol](#the-iteration-protocol)
- [Creating Iterators](#creating-iterators)
- [Generator Functions](#generator-functions)
- [Generator Expressions](#generator-expressions)
- [yield from](#yield-from)
- [Send and Two-Way Communication](#send-and-two-way-communication)
- [Practical Use Cases](#practical-use-cases)
- [Performance Benefits](#performance-benefits)
- [Summary](#summary)
- [Next Steps](#next-steps)

## The Iteration Protocol

### How Iteration Works

Behind the scenes of `for` loops:

```python
# What Python does with for loops
numbers = [1, 2, 3]
for num in numbers:
    print(num)

# Equivalent to:
numbers = [1, 2, 3]
iterator = iter(numbers)  # Get iterator
while True:
    try:
        num = next(iterator)  # Get next item
        print(num)
    except StopIteration:  # No more items
        break
```

### Manual Iteration

```python
numbers = [1, 2, 3, 4, 5]

# Get iterator
it = iter(numbers)

# Get items one at a time
print(next(it))  # 1
print(next(it))  # 2
print(next(it))  # 3

# Continue until exhausted
print(next(it))  # 4
print(next(it))  # 5
# print(next(it))  # StopIteration exception
```

### The Iterator Protocol

Two methods: `__iter__()` and `__next__()`:

```python
class CountDown:
    def __init__(self, start):
        self.current = start

    def __iter__(self):
        """Return iterator object (self)."""
        return self

    def __next__(self):
        """Return next item or raise StopIteration."""
        if self.current <= 0:
            raise StopIteration

        self.current -= 1
        return self.current + 1

# Use in for loop
for num in CountDown(5):
    print(num)
# 5, 4, 3, 2, 1
```

## Creating Iterators

### Basic Iterator Class

```python
class Range:
    """Custom range iterator."""
    def __init__(self, start, end):
        self.current = start
        self.end = end

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.end:
            raise StopIteration

        value = self.current
        self.current += 1
        return value

# Use like built-in range
for i in Range(0, 5):
    print(i)
# 0, 1, 2, 3, 4
```

### Iterable vs Iterator

**Iterable:** Has `__iter__()`, can be iterated over
**Iterator:** Has both `__iter__()` and `__next__()`

```python
class Squares:
    """Iterable that creates iterator."""
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        """Return new iterator."""
        return SquaresIterator(self.n)

class SquaresIterator:
    """Iterator."""
    def __init__(self, n):
        self.n = n
        self.i = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.i >= self.n:
            raise StopIteration

        result = self.i ** 2
        self.i += 1
        return result

# Can iterate multiple times
squares = Squares(5)
print(list(squares))  # [0, 1, 4, 9, 16]
print(list(squares))  # [0, 1, 4, 9, 16] - works again
```

### File Iterator

```python
class FileIterator:
    """Iterate over file lines."""
    def __init__(self, filename):
        self.file = open(filename, 'r')

    def __iter__(self):
        return self

    def __next__(self):
        line = self.file.readline()
        if not line:
            self.file.close()
            raise StopIteration
        return line.strip()

# Use (files are already iterable, this is for demonstration)
for line in FileIterator('data.txt'):
    print(line)
```

## Generator Functions

### Basic Generator

Use `yield` instead of `return`:

```python
def count_up_to(n):
    """Generator function."""
    count = 1
    while count <= n:
        yield count  # Pause here, resume on next()
        count += 1

# Creates generator object
counter = count_up_to(5)
print(type(counter))  # <class 'generator'>

# Use in for loop
for num in count_up_to(5):
    print(num)
# 1, 2, 3, 4, 5
```

### Generator State

Generators remember state between calls:

```python
def simple_generator():
    print("Starting")
    yield 1
    print("Between first and second")
    yield 2
    print("Between second and third")
    yield 3
    print("Ending")

gen = simple_generator()

print("Created generator")
print(next(gen))  # Starting, then 1
print("Got first value")
print(next(gen))  # Between first and second, then 2
print("Got second value")
print(next(gen))  # Between second and third, then 3
print("Got third value")
# next(gen)  # Ending, then StopIteration
```

### Infinite Generators

```python
def infinite_counter(start=0):
    """Generate infinite sequence."""
    count = start
    while True:
        yield count
        count += 1

# Use with caution - infinite!
counter = infinite_counter()
print(next(counter))  # 0
print(next(counter))  # 1
print(next(counter))  # 2

# Safe usage - limit iteration
for i, value in enumerate(infinite_counter()):
    if i >= 5:
        break
    print(value)
# 0, 1, 2, 3, 4
```

### Fibonacci Generator

```python
def fibonacci():
    """Generate Fibonacci sequence."""
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Get first 10 Fibonacci numbers
fib = fibonacci()
for _ in range(10):
    print(next(fib), end=' ')
# 0 1 1 2 3 5 8 13 21 34
```

## Generator Expressions

### Syntax

Like list comprehensions but with parentheses:

```python
# List comprehension - creates list immediately
squares_list = [x**2 for x in range(10)]

# Generator expression - creates generator
squares_gen = (x**2 for x in range(10))

print(type(squares_list))  # <class 'list'>
print(type(squares_gen))   # <class 'generator'>

# Consume generator
for sq in squares_gen:
    print(sq)
```

### Memory Efficiency

```python
import sys

# List - all in memory
numbers_list = [x for x in range(1000000)]
print(sys.getsizeof(numbers_list))  # ~8 MB

# Generator - produces on demand
numbers_gen = (x for x in range(1000000))
print(sys.getsizeof(numbers_gen))   # ~128 bytes

# Use generator in operations
total = sum(x**2 for x in range(1000000))  # Memory efficient
```

### Common Patterns

```python
# Filter and transform
even_squares = (x**2 for x in range(20) if x % 2 == 0)

# Nested generators
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = (item for row in matrix for item in row)

# With functions
lines = (line.strip() for line in open('file.txt'))
upper_lines = (line.upper() for line in lines)
```

## yield from

### Delegating to Sub-Generator

```python
def generator1():
    yield 1
    yield 2

def generator2():
    yield 3
    yield 4

# Without yield from
def combined():
    for value in generator1():
        yield value
    for value in generator2():
        yield value

# With yield from
def combined():
    yield from generator1()
    yield from generator2()

print(list(combined()))  # [1, 2, 3, 4]
```

### Flattening Nested Structures

```python
def flatten(nested):
    """Recursively flatten nested iterables."""
    for item in nested:
        if isinstance(item, (list, tuple)):
            yield from flatten(item)
        else:
            yield item

nested = [1, [2, 3, [4, 5]], 6, [7, [8, 9]]]
print(list(flatten(nested)))
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Tree Traversal

```python
class Node:
    def __init__(self, value, children=None):
        self.value = value
        self.children = children or []

    def traverse(self):
        """Depth-first traversal."""
        yield self.value
        for child in self.children:
            yield from child.traverse()

# Build tree
tree = Node(1, [
    Node(2, [Node(4), Node(5)]),
    Node(3, [Node(6), Node(7)])
])

print(list(tree.traverse()))  # [1, 2, 4, 5, 3, 6, 7]
```

## Send and Two-Way Communication

### Sending Values to Generators

```python
def echo():
    """Generator that echoes sent values."""
    while True:
        value = yield  # Receive value
        print(f"Received: {value}")

gen = echo()
next(gen)  # Prime the generator

gen.send("Hello")  # Received: Hello
gen.send(42)       # Received: 42
gen.send([1, 2])   # Received: [1, 2]
```

### Two-Way Communication

```python
def running_average():
    """Maintain running average."""
    total = 0
    count = 0
    average = None

    while True:
        value = yield average  # Send average, receive value
        total += value
        count += 1
        average = total / count

avg = running_average()
next(avg)  # Prime

print(avg.send(10))  # 10.0
print(avg.send(20))  # 15.0
print(avg.send(30))  # 20.0
```

### Coroutine Pattern

```python
def grep(pattern):
    """Search for pattern in sent values."""
    print(f"Looking for {pattern}")
    while True:
        line = yield
        if pattern in line:
            print(line)

search = grep("python")
next(search)  # Prime

search.send("I love python")      # I love python
search.send("Java is cool")       # (no output)
search.send("python is awesome")  # python is awesome
```

## Practical Use Cases

### Reading Large Files

```python
def read_large_file(filename):
    """Read file line by line."""
    with open(filename, 'r') as file:
        for line in file:
            yield line.strip()

# Memory efficient for huge files
for line in read_large_file('huge_file.txt'):
    process(line)
```

### Processing Streaming Data

```python
def parse_log_lines(filename):
    """Parse log file entries."""
    with open(filename, 'r') as file:
        for line in file:
            parts = line.strip().split(' - ')
            if len(parts) == 3:
                timestamp, level, message = parts
                yield {
                    'timestamp': timestamp,
                    'level': level,
                    'message': message
                }

# Process logs as they're read
for entry in parse_log_lines('app.log'):
    if entry['level'] == 'ERROR':
        alert(entry['message'])
```

### Pipeline Pattern

```python
def numbers(n):
    """Generate numbers 0 to n."""
    for i in range(n):
        yield i

def square(nums):
    """Square each number."""
    for num in nums:
        yield num ** 2

def filter_even(nums):
    """Keep only even numbers."""
    for num in nums:
        if num % 2 == 0:
            yield num

# Chain generators
pipeline = filter_even(square(numbers(10)))
print(list(pipeline))  # [0, 4, 16, 36, 64]
```

### Batch Processing

```python
def batch(iterable, size):
    """Yield batches of specified size."""
    batch = []
    for item in iterable:
        batch.append(item)
        if len(batch) == size:
            yield batch
            batch = []

    if batch:  # Yield remaining items
        yield batch

# Process data in batches
data = range(25)
for batch_data in batch(data, 10):
    print(batch_data)
# [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
# [10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
# [20, 21, 22, 23, 24]
```

## Performance Benefits

### Memory Comparison

```python
import sys

# List - all in memory
def get_squares_list(n):
    return [x**2 for x in range(n)]

# Generator - on demand
def get_squares_gen(n):
    for x in range(n):
        yield x**2

n = 1000000

squares_list = get_squares_list(n)
print(f"List size: {sys.getsizeof(squares_list) / 1024 / 1024:.2f} MB")

squares_gen = get_squares_gen(n)
print(f"Generator size: {sys.getsizeof(squares_gen)} bytes")
```

### Lazy Evaluation

```python
# Eager - all executed immediately
def eager_process():
    data = [expensive_operation(x) for x in range(1000)]
    return data[:10]  # Only need first 10!

# Lazy - only compute what's needed
def lazy_process():
    data = (expensive_operation(x) for x in range(1000))
    return list(itertools.islice(data, 10))  # Only compute 10
```

### Infinite Sequences

```python
def primes():
    """Generate prime numbers infinitely."""
    def is_prime(n):
        if n < 2:
            return False
        for i in range(2, int(n**0.5) + 1):
            if n % i == 0:
                return False
        return True

    n = 2
    while True:
        if is_prime(n):
            yield n
        n += 1

# Get first 20 primes
import itertools
first_20_primes = list(itertools.islice(primes(), 20))
print(first_20_primes)
```

## Summary

Iterators and generators in Python:

**Iteration protocol:**

- `__iter__()` returns iterator
- `__next__()` returns next item or raises `StopIteration`
- `for` loops use this protocol

**Creating iterators:**

- Implement `__iter__()` and `__next__()`
- Separate iterable (container) from iterator (state)

**Generators:**

- Functions with `yield` keyword
- Automatically implement iterator protocol
- Remember state between calls
- More concise than iterator classes

**Generator expressions:**

- `(expr for item in iterable)`
- Like list comprehensions but lazy
- Memory efficient for large datasets

**yield from:**

- Delegate to sub-generator
- Simplify nested generators
- Flatten recursive structures

**Practical benefits:**

- Memory efficiency for large datasets
- Lazy evaluation - compute on demand
- Pipeline processing
- Infinite sequences
- Clean, readable code

Generators are a powerful tool for working with sequences and streams of data efficiently.

## Next Steps

- Manage resources with [Context Managers](context-managers.md)
- Explore async generators in [Concurrency](concurrency.md)
- Apply to [Performance and Optimization](../performance_and_optimization/)
