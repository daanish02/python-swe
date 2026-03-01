# Control Flow

## Conditional Statements

### The `if` Statement

Execute code only when a condition is True:

```python
age = 18
if age >= 18:
    print("You are an adult")
    print("You can vote")
```

**Indentation is syntax:** Python uses indentation to define code blocks (4 spaces is standard).

```python
# Wrong - IndentationError
if age >= 18:
print("Adult")

# Right
if age >= 18:
    print("Adult")
```

### `if-else`

Execute different code based on condition:

```python
age = 15
if age >= 18:
    print("Adult")
else:
    print("Minor")
```

### `if-elif-else`

Multiple conditions evaluated in order:

```python
score = 85

if score >= 90:
    grade = 'A'
elif score >= 80:
    grade = 'B'
elif score >= 70:
    grade = 'C'
elif score >= 60:
    grade = 'D'
else:
    grade = 'F'

print(f"Grade: {grade}")
```

**Order matters:** First matching condition wins:

```python
x = 75

# This works
if x >= 70:
    print("C or better")  # Executes
elif x >= 80:
    print("B or better")  # Never checked

# Better structure
if x >= 80:
    print("B or better")
elif x >= 70:
    print("C or better")
```

### Nested Conditions

```python
age = 25
has_license = True

if age >= 18:
    if has_license:
        print("Can drive")
    else:
        print("Cannot drive - no license")
else:
    print("Cannot drive - too young")
```

**Prefer flat over nested when possible:**

=== "Nested"

    ```python
    if age >= 18:
        if has_license:
            print("Can drive")
        else:
            print("No license")
    else:
        print("Too young")
    ```

=== "Flat"

    ```python
    if age < 18:
        print("Too young")
    elif not has_license:
        print("No license")
    else:
        print("Can drive")
    ```

### Truthiness in Conditions

Don't need explicit comparisons:

```python
items = [1, 2, 3]

# Verbose
if len(items) > 0:
    print("Has items")

# Pythonic
if items:
    print("Has items")

# Check for empty
if not items:
    print("Empty")
```

**Falsy values:**

- `False`, `None`
- `0`, `0.0`
- `""` (empty string)
- `[]`, `()`, `{}` (empty collections)

**Everything else is truthy.**

### Chained Comparisons

```python
x = 5

# Verbose
if x > 0 and x < 10:
    print("Single digit positive")

# Pythonic (chained)
if 0 < x < 10:
    print("Single digit positive")

# Works with multiple chains
if 0 < x < y < 100:
    print("x and y between 0 and 100, x < y")
```

## Loops

### The `for` Loop

Iterate over sequences (lists, strings, ranges, etc.):

```python
# Iterate over list
fruits = ['apple', 'banana', 'cherry']
for fruit in fruits:
    print(fruit)

# Iterate over string
for char in "Python":
    print(char)

# Iterate over range
for i in range(5):  # 0, 1, 2, 3, 4
    print(i)

# Range with start and end
for i in range(1, 6):  # 1, 2, 3, 4, 5
    print(i)

# Range with step
for i in range(0, 10, 2):  # 0, 2, 4, 6, 8
    print(i)

# Backwards
for i in range(10, 0, -1):  # 10, 9, 8, ..., 1
    print(i)
```

### Iterating with Index

```python
fruits = ['apple', 'banana', 'cherry']

# Non-Pythonic
for i in range(len(fruits)):
    print(i, fruits[i])

# Pythonic - use enumerate()
for i, fruit in enumerate(fruits):
    print(i, fruit)

# Start index at different value
for i, fruit in enumerate(fruits, start=1):
    print(i, fruit)  # 1 apple, 2 banana, 3 cherry
```

### Iterating Multiple Sequences

```python
names = ['Alice', 'Bob', 'Charlie']
ages = [25, 30, 35]

# Use zip() to iterate in parallel
for name, age in zip(names, ages):
    print(f"{name} is {age} years old")

# zip stops at shortest sequence
numbers = [1, 2, 3, 4, 5]
letters = ['a', 'b', 'c']
for num, letter in zip(numbers, letters):
    print(num, letter)  # Only pairs (1,a), (2,b), (3,c)
```

### Iterating Dictionaries

```python
person = {'name': 'Alice', 'age': 25, 'city': 'NYC'}

# Iterate keys (default)
for key in person:
    print(key)

# Iterate keys explicitly
for key in person.keys():
    print(key)

# Iterate values
for value in person.values():
    print(value)

# Iterate key-value pairs
for key, value in person.items():
    print(f"{key}: {value}")
```

### The `while` Loop

Repeat while condition is True:

```python
count = 0
while count < 5:
    print(count)
    count += 1

# Common pattern: input validation
while True:
    response = input("Enter 'yes' or 'no': ").lower()
    if response in ['yes', 'no']:
        break
    print("Invalid input, try again")
```

**Warning:** Ensure the condition eventually becomes False to avoid infinite loops:

```python
# Infinite loop - condition never becomes False
while True:
    print("Forever!")  # Ctrl+C to stop

# Good - condition will become False
x = 10
while x > 0:
    print(x)
    x -= 1
```

## Loop Control

### `break` - Exit Loop

Immediately exit the innermost loop:

```python
# Find first even number
numbers = [1, 3, 5, 6, 7, 8]
for num in numbers:
    if num % 2 == 0:
        print(f"Found even number: {num}")
        break  # Stop searching

# Exit while loop
while True:
    response = input("Continue? (y/n): ")
    if response.lower() == 'n':
        break
```

### `continue` - Skip Iteration

Skip rest of current iteration, move to next:

```python
# Print only positive numbers
numbers = [5, -2, 8, -1, 3, -7]
for num in numbers:
    if num < 0:
        continue  # Skip negative numbers
    print(num)  # Only positive numbers printed

# Equivalent without continue
for num in numbers:
    if num >= 0:
        print(num)
```

### `else` with Loops

`else` clause executes if loop completes normally (not broken):

```python
# Check if all numbers are positive
numbers = [1, 2, 3, 4, 5]
for num in numbers:
    if num < 0:
        print("Found negative number")
        break
else:
    print("All numbers are positive")  # Executes if no break

# Search pattern
target = 7
numbers = [1, 3, 5, 9]
for num in numbers:
    if num == target:
        print(f"Found {target}")
        break
else:
    print(f"{target} not found")  # Executes if not found
```

### `pass` - Do Nothing

Placeholder that does nothing:

```python
# Empty block (syntax requires something)
for i in range(10):
    pass  # Do nothing, but valid syntax

# Stub for future implementation
def coming_soon():
    pass  # TODO: implement later

# Conditional placeholder
if condition:
    pass  # Handle this case later
else:
    do_something()
```

## Match Statements

Python 3.10+ introduced pattern matching with `match`/`case`:

### Basic Match

```python
status_code = 404

match status_code:
    case 200:
        print("OK")
    case 404:
        print("Not Found")
    case 500:
        print("Server Error")
    case _:  # default case
        print("Unknown status")
```

### Pattern Matching with Values

```python
point = (0, 5)

match point:
    case (0, 0):
        print("Origin")
    case (0, y):
        print(f"Y-axis at {y}")
    case (x, 0):
        print(f"X-axis at {x}")
    case (x, y):
        print(f"Point at ({x}, {y})")
```

### Match with Guards

```python
age = 25

match age:
    case n if n < 0:
        print("Invalid age")
    case n if n < 18:
        print("Minor")
    case n if n < 65:
        print("Adult")
    case _:
        print("Senior")
```

### Match with Types

```python
def process(data):
    match data:
        case int():
            print(f"Integer: {data}")
        case str():
            print(f"String: {data}")
        case list():
            print(f"List with {len(data)} items")
        case _:
            print(f"Unknown type: {type(data)}")
```

!!! note

    `match` is powerful but often an `if-elif-else` chain is simpler for basic cases.

## Conditional Expressions

### Ternary Operator

Conditional assignment in one line:

```python
# Standard if-else
if age >= 18:
    status = "adult"
else:
    status = "minor"

# Ternary (conditional expression)
status = "adult" if age >= 18 else "minor"

# General form: value_if_true if condition else value_if_false
max_value = a if a > b else b
```

**Use for simple cases only:**

```python
# Good - simple condition
message = "Pass" if score >= 60 else "Fail"

# Bad - too complex
result = "A" if score >= 90 else "B" if score >= 80 else "C" \
                if score >= 70 else "D" if score >= 60 else "F"

# Better - use regular if-elif-else for multiple conditions
```

### Short-Circuit Evaluation

Logical operators return values, not just booleans:

```python
# or returns first truthy value
name = user_name or "Guest"  # Use user_name if truthy, else "Guest"

# and returns last value if all truthy
value = validate() and process() and save()

# Careful with side effects
x = 0
y = x or 10  # y is 10 (x is falsy)
z = x and 10  # z is 0 (first falsy value)
```

## Summary

Control flow in Python is straightforward and readable:

**Conditionals:**

- `if condition:` - execute if True
- `if-elif-else` - multiple conditions
- Use truthiness instead of explicit comparisons
- Chain comparisons: `0 < x < 10`

**Loops:**

- `for item in sequence:` - iterate over items
- `while condition:` - repeat while True
- Use `enumerate()` for indexed iteration
- Use `zip()` for parallel iteration

**Control keywords:**

- `break` - exit loop
- `continue` - skip to next iteration
- `pass` - do nothing (placeholder)
- `else` with loops - executes if not broken

**Pattern matching:**

- `match/case` (Python 3.10+) for complex pattern matching

**Conditional expressions:**

- `value = x if condition else y` for simple cases

## Next Steps

- Apply control flow with [Data Structures](data-structures.md)
- Build reusable logic with [Functions](functions.md)
- Handle errors gracefully with [Error Handling](error-handling.md)
