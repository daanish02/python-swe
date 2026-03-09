# How to Write Tests: A Complete Beginner's Guide

## Table of Contents

- [Your First Test - Complete Walkthrough](#your-first-test---complete-walkthrough)
- [How to Choose What to Test](#how-to-choose-what-to-test)
- [How to Choose Test Values](#how-to-choose-test-values)
- [How to Determine Assertions](#how-to-determine-assertions)
- [How Many Tests Do I Need?](#how-many-tests-do-i-need)
- [Common Beginner Mistakes](#common-beginner-mistakes)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
- [Practice Exercises](#practice-exercises)

## About This Guide

This guide teaches you **the thinking process** for writing tests from scratch. If you've never written a test before, this is where you should start.

You'll learn:

- How to decide what functions need tests
- How to choose test values systematically
- How to determine what to assert
- How to know when you're done
- How to avoid common mistakes

**Prerequisites:** Basic Python knowledge (functions, classes, basic data types)

**Not prerequisites:** Any testing experience! We'll start from zero.

---

## Your First Test - Complete Walkthrough

Let's write tests for a real function, showing every thought step along the way.

### The Function

```python
# discount.py
def calculate_discount(price, discount_percent):
    """Calculate discounted price.

    Args:
        price: Original price (must be positive)
        discount_percent: Discount percentage (0-100)

    Returns:
        Discounted price

    Raises:
        ValueError: If price is negative or discount is invalid
    """
    if price < 0:
        raise ValueError("Price cannot be negative")

    if discount_percent < 0 or discount_percent > 100:
        raise ValueError("Discount must be between 0 and 100")

    discount_amount = price * (discount_percent / 100)
    return price - discount_amount
```

### Step 1: Read and Understand

**Think:** What does this function do?

- Takes a price and discount percentage
- Returns the price after applying the discount
- Has validation: price can't be negative, discount must be 0-100
- Can raise ValueError for invalid inputs

### Step 2: Identify What to Test

**Think:** What behaviors should I verify?

1. **Normal case:** Valid price and discount returns correct result
2. **Edge cases for discount:** 0% discount, 100% discount, middle values
3. **Edge cases for price:** Zero price, small price, large price
4. **Error cases:** Negative price, negative discount, discount > 100

### Step 3: Choose Test Values

**Think:** For each test, what specific values should I use?

1. Normal case: price=100, discount=20 (easy to calculate: should be 80)
2. Zero discount: price=100, discount=0 (should return 100)
3. 100% discount: price=50, discount=100 (should return 0)
4. Zero price: price=0, discount=10 (should return 0)
5. Negative price: price=-10, discount=20 (should raise ValueError)
6. Negative discount: price=100, discount=-5 (should raise ValueError)
7. Discount > 100: price=100, discount=150 (should raise ValueError)

**Why these values?**

- Round numbers make it easy to verify correctness
- Cover all validation branches
- Include boundaries (0, 100)

### Step 4: Write the Tests

```python
# test_discount.py
import pytest
from discount import calculate_discount

def test_calculate_discount_normal_case():
    """Test discount calculation with typical values."""
    # Arrange: Set up test data
    price = 100
    discount_percent = 20

    # Act: Call the function
    result = calculate_discount(price, discount_percent)

    # Assert: Verify the result
    assert result == 80  # 100 - (100 * 0.20) = 80

def test_calculate_discount_zero_discount():
    """Test that 0% discount returns original price."""
    result = calculate_discount(100, 0)
    assert result == 100  # No discount applied

def test_calculate_discount_full_discount():
    """Test that 100% discount returns zero."""
    result = calculate_discount(50, 100)
    assert result == 0  # Everything is discounted

def test_calculate_discount_zero_price():
    """Test with zero price."""
    result = calculate_discount(0, 10)
    assert result == 0  # 0 minus anything is still 0

def test_calculate_discount_negative_price_raises_error():
    """Test that negative price raises ValueError."""
    with pytest.raises(ValueError, match="Price cannot be negative"):
        calculate_discount(-10, 20)
    # Why pytest.raises? Because we expect an exception, not a return value

def test_calculate_discount_negative_discount_raises_error():
    """Test that negative discount raises ValueError."""
    with pytest.raises(ValueError, match="Discount must be between 0 and 100"):
        calculate_discount(100, -5)

def test_calculate_discount_excessive_discount_raises_error():
    """Test that discount > 100 raises ValueError."""
    with pytest.raises(ValueError, match="Discount must be between 0 and 100"):
        calculate_discount(100, 150)
```

### Step 5: Run the Tests

```bash
# Install pytest if you haven't
pip install pytest

# Run the tests
pytest test_discount.py -v
```

Expected output:

```
test_discount.py::test_calculate_discount_normal_case PASSED
test_discount.py::test_calculate_discount_zero_discount PASSED
test_discount.py::test_calculate_discount_full_discount PASSED
test_discount.py::test_calculate_discount_zero_price PASSED
test_discount.py::test_calculate_discount_negative_price_raises_error PASSED
test_discount.py::test_calculate_discount_negative_discount_raises_error PASSED
test_discount.py::test_calculate_discount_excessive_discount_raises_error PASSED

7 passed in 0.03s
```

### Key Takeaways from This Example

1. **Read the function carefully** - understand what it does and its constraints
2. **Identify scenarios** - normal case, edge cases, error cases
3. **Choose specific values** - use numbers that are easy to verify
4. **Use descriptive names** - test name should say what it tests
5. **Follow Arrange-Act-Assert** - makes tests easy to read
6. **Test errors too** - use `pytest.raises` for expected exceptions

---

## How to Choose What to Test

Not everything needs tests. Here's a decision framework:

### Decision Flowchart

```
Does the code contain logic (if/else, loops, calculations)?
├─ YES → HIGH PRIORITY: Write tests
└─ NO → Is it a simple getter/setter?
    ├─ YES → LOW PRIORITY: Probably skip
    └─ NO → Does it interact with external systems (DB, API, files)?
        ├─ YES → HIGH PRIORITY: Write tests
        └─ NO → Is it configuration or constants?
            ├─ YES → SKIP: No tests needed
            └─ NO → Does it have error handling?
                ├─ YES → MEDIUM PRIORITY: Write tests
                └─ NO → LOW PRIORITY: Consider skipping
```

### Test Priority Matrix

**HIGH PRIORITY (Must Test):**

- Business logic with calculations
- Functions with conditionals (if/else)
- Error handling and validation
- Functions that transform data
- Integration points (database, API, file I/O)
- Security-related code (authentication, authorization)

**MEDIUM PRIORITY (Should Test):**

- Functions with multiple parameters
- String processing and formatting
- List/dict manipulation
- State-changing methods

**LOW PRIORITY (Can Skip):**

- Simple getters and setters without logic
- Property accessors
- Constants and configuration
- Direct pass-throughs to libraries

**NEVER TEST:**

- Standard library functions (already tested)
- Third-party libraries (their responsibility)
- Trivial one-liners with no logic

### Examples with Decisions

```python
# ✅ HIGH PRIORITY: Test this
def calculate_tax(amount, tax_rate, tax_type):
    """Complex business logic - MUST TEST."""
    if tax_type == "inclusive":
        return amount * tax_rate / (1 + tax_rate)
    elif tax_type == "exclusive":
        return amount * tax_rate
    else:
        raise ValueError(f"Unknown tax type: {tax_type}")

# ✅ HIGH PRIORITY: Test this
def validate_email(email):
    """Validation logic - MUST TEST."""
    if not email or "@" not in email:
        return False
    parts = email.split("@")
    return len(parts) == 2 and all(parts)

# ✅ MEDIUM PRIORITY: Test this
def format_name(first_name, last_name, middle_initial=None):
    """Multiple parameters and formatting - SHOULD TEST."""
    if middle_initial:
        return f"{first_name} {middle_initial}. {last_name}"
    return f"{first_name} {last_name}"

# ⚠️ LOW PRIORITY: Probably skip
class User:
    def __init__(self, name):
        self._name = name

    @property
    def name(self):
        """Simple getter - LOW PRIORITY."""
        return self._name

    @name.setter
    def name(self, value):
        """Simple setter - LOW PRIORITY."""
        self._name = value

# ❌ SKIP: Don't test this
def get_config():
    """Just returns a dict - SKIP."""
    return {
        "api_url": "https://api.example.com",
        "timeout": 30
    }

# ❌ SKIP: Don't test this
def save_user_wrapper(user):
    """Just calls library function - SKIP."""
    return database.save(user)  # Test the actual logic, not the wrapper
```

### "Should I Test This?" Quick Guide

| Code Type             | Test It? | Why / Why Not                          |
| --------------------- | -------- | -------------------------------------- |
| Business calculations | ✅ YES   | Core logic, high value if wrong        |
| Input validation      | ✅ YES   | Protects from bad data                 |
| Error handling        | ✅ YES   | Ensures failures are handled correctly |
| String formatting     | ✅ YES   | Edge cases like empty strings          |
| Database queries      | ✅ YES   | Integration testing important          |
| API endpoints         | ✅ YES   | Contract testing needed                |
| Simple getters        | ❌ NO    | No logic to test                       |
| Constants             | ❌ NO    | Can't be wrong                         |
| Third-party wrappers  | ❌ NO    | Test your logic, not theirs            |
| Logging statements    | ❌ NO    | Testing output is low value            |

### When in Doubt

Ask yourself:

1. **What could go wrong?** If many things → test it
2. **Is this code complex?** If yes → test it
3. **Would a bug here be costly?** If yes → test it
4. **Can I think of multiple scenarios?** If yes → test it
5. **Is this just passing data through?** If yes → skip it

**Rule of thumb:** If you're wondering whether to test something, lean toward testing it. Writing a test takes 5 minutes; debugging a production bug takes hours.

---

## How to Choose Test Values

You have a function. What values should you pass to it? Follow this systematic process.

### The 4-Step Process

**Step 1: Identify the input types**

Look at the function signature and determine what each parameter expects.

```python
def process_order(items: list, discount_code: str, user_age: int) -> float:
    pass
```

Inputs:

- `items`: list
- `discount_code`: string
- `user_age`: integer

**Step 2: For each input, categorize test values**

For every input, consider these categories:

| Category       | Description              | Example                             |
| -------------- | ------------------------ | ----------------------------------- |
| **Normal**     | Typical, expected values | List with 3 items, "SAVE10", age=25 |
| **Edge cases** | Extreme but valid values | Empty list, empty string, age=0     |
| **Boundaries** | Just above/below limits  | age=17, age=18 (if 18+ matters)     |
| **Invalid**    | Should cause errors      | None values, negative age           |

**Step 3: Apply equivalence partitioning**

Group similar inputs into "partitions" and test one value from each partition.

Example: Age validation (must be 18+)

Partitions:

- **Children:** 0-17 (test with 10)
- **Adults:** 18-64 (test with 30)
- **Seniors:** 65+ (test with 70)
- **Boundaries:** Test 17, 18, 64, 65
- **Invalid:** Test -1, None

**Step 4: Apply boundary value analysis**

If there are limits or boundaries, test:

- Just below the boundary
- At the boundary
- Just above the boundary

Example: String length must be 1-50 characters

- Test 0 chars (below, should fail)
- Test 1 char (at lower boundary, should pass)
- Test 2 chars (just above lower boundary, should pass)
- Test 49 chars (just below upper boundary, should pass)
- Test 50 chars (at upper boundary, should pass)
- Test 51 chars (above, should fail)

### Quick Reference Table: Input Type → Test Values

| Input Type        | Test Values to Consider                                                                                                                                                     |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Integer**       | • Positive (e.g., 42)<br>• Negative (e.g., -5)<br>• Zero<br>• Large number (e.g., 999999)<br>• Boundaries (if range specified)                                              |
| **Float**         | • Positive (e.g., 3.14)<br>• Negative (e.g., -2.5)<br>• Zero (0.0)<br>• Very small (e.g., 0.0001)<br>• Very large<br>• Special: inf, -inf, NaN (if relevant)                |
| **String**        | • Empty string ""<br>• Single character "a"<br>• Typical string "hello"<br>• Long string<br>• Special characters "!@#$"<br>• Unicode "こんにちは"<br>• None (if allowed)    |
| **List**          | • Empty list []<br>• Single item [1]<br>• Multiple items [1, 2, 3]<br>• Large list (100+ items)<br>• None (if allowed)<br>• List with duplicates<br>• List with None values |
| **Dict**          | • Empty dict {}<br>• Single key {"a": 1}<br>• Multiple keys<br>• Missing keys (if expected keys)<br>• None (if allowed)                                                     |
| **Boolean**       | • True<br>• False<br>• None (if allowed)                                                                                                                                    |
| **None/Optional** | • None<br>• Valid value<br>• Empty value (e.g., "", [], {})                                                                                                                 |

### Common Patterns with Templates

#### Pattern 1: Numeric Function

```python
def square_root(n):
    """Returns square root of n."""
    if n < 0:
        raise ValueError("Cannot calculate square root of negative number")
    return n ** 0.5

# Test values:
def test_square_root_positive():
    assert square_root(4) == 2.0  # Normal case

def test_square_root_zero():
    assert square_root(0) == 0.0  # Edge case: boundary

def test_square_root_large():
    assert square_root(1000000) == 1000.0  # Large number

def test_square_root_fractional():
    assert square_root(0.25) == 0.5  # Fractional input

def test_square_root_negative():
    with pytest.raises(ValueError):  # Invalid case
        square_root(-1)
```

#### Pattern 2: String Function

```python
def truncate_string(s, max_length):
    """Truncate string to max_length, adding '...'."""
    if max_length < 3:
        raise ValueError("max_length must be at least 3")

    if len(s) <= max_length:
        return s
    return s[:max_length - 3] + "..."

# Test values:
def test_truncate_shorter_than_max():
    assert truncate_string("hello", 10) == "hello"  # No truncation

def test_truncate_exactly_max():
    assert truncate_string("hello", 5) == "hello"  # Boundary

def test_truncate_longer_than_max():
    assert truncate_string("hello world", 8) == "hello..."  # Truncation

def test_truncate_empty_string():
    assert truncate_string("", 10) == ""  # Edge case

def test_truncate_single_char():
    assert truncate_string("a", 10) == "a"  # Edge case

def test_truncate_max_too_small():
    with pytest.raises(ValueError):  # Invalid
        truncate_string("hello", 2)
```

#### Pattern 3: List Function

```python
def find_duplicates(items):
    """Return list of items that appear more than once."""
    seen = set()
    duplicates = set()

    for item in items:
        if item in seen:
            duplicates.add(item)
        else:
            seen.add(item)

    return list(duplicates)

# Test values:
def test_find_duplicates_with_duplicates():
    assert find_duplicates([1, 2, 2, 3, 3, 3]) == [2, 3]  # Normal

def test_find_duplicates_no_duplicates():
    assert find_duplicates([1, 2, 3]) == []  # No duplicates

def test_find_duplicates_empty_list():
    assert find_duplicates([]) == []  # Edge case

def test_find_duplicates_single_item():
    assert find_duplicates([1]) == []  # Edge case

def test_find_duplicates_all_same():
    assert find_duplicates([5, 5, 5]) == [5]  # All duplicates
```

#### Pattern 4: Multiple Parameters

```python
def calculate_bmi(weight_kg, height_m):
    """Calculate Body Mass Index."""
    if weight_kg <= 0:
        raise ValueError("Weight must be positive")
    if height_m <= 0:
        raise ValueError("Height must be positive")

    return weight_kg / (height_m ** 2)

# Test combinations:
def test_bmi_normal_values():
    assert calculate_bmi(70, 1.75) == pytest.approx(22.86, rel=0.01)

def test_bmi_minimum_valid():
    assert calculate_bmi(0.1, 0.1) == pytest.approx(10.0)  # Edge case

def test_bmi_large_values():
    assert calculate_bmi(150, 2.0) == pytest.approx(37.5)

def test_bmi_zero_weight():
    with pytest.raises(ValueError):
        calculate_bmi(0, 1.75)

def test_bmi_negative_weight():
    with pytest.raises(ValueError):
        calculate_bmi(-70, 1.75)

def test_bmi_zero_height():
    with pytest.raises(ValueError):
        calculate_bmi(70, 0)

def test_bmi_negative_height():
    with pytest.raises(ValueError):
        calculate_bmi(70, -1.75)
```

### Decision Tree: "What Values Should I Test?"

```
Look at the function parameter:

1. What type is it? (int, str, list, etc.)
   → Use the Quick Reference Table above

2. Are there documented constraints? (must be positive, max length, etc.)
   → Test at boundaries: below, at, above

3. Are there different "categories" of values? (child/adult, small/large, etc.)
   → Test one value from each category (equivalence partitioning)

4. What happens with empty/zero/None?
   → Test these edge cases

5. What should cause an error?
   → Test these with pytest.raises

6. Combine parameters:
   → Test normal case with all valid
   → Test each parameter invalid separately
```

---

## How to Determine Assertions

You've called the function. Now what should you check? Here's how to decide:

### Decision Tree for Assertions

```
What does the function do?

├─ Returns a value?
│  ├─ Specific value expected? → assert result == expected
│  ├─ Type matters? → assert isinstance(result, ExpectedType)
│  ├─ Range of values? → assert min_val <= result <= max_val
│  └─ Floating point? → assert result == pytest.approx(expected)
│
├─ Changes object state?
│  ├─ Check attribute: assert obj.attribute == expected
│  └─ Check multiple attributes: assert all conditions
│
├─ Raises an exception?
│  └─ with pytest.raises(ExceptionType, match="message"):
│
├─ Has side effects? (writes file, calls API)
│  ├─ Use mock: mock_func.assert_called_once()
│  └─ Check observable output: assert file_exists()
│
└─ Returns None but does something?
   └─ Check what it did (state change, side effect)
```

### Pattern 1: Simple Return Value

```python
def add(a, b):
    return a + b

# Ask: What should this return?
# Answer: Sum of a and b
def test_add():
    result = add(2, 3)
    assert result == 5  # Direct equality
```

### Pattern 2: Return Value with Type

```python
def get_user(user_id):
    # Returns User object or None
    if user_id in database:
        return User(database[user_id])
    return None

# Ask: What should this return?
# Answer: User instance or None
def test_get_user_exists():
    result = get_user(123)
    assert isinstance(result, User)  # Check type
    assert result.id == 123  # Check property

def test_get_user_not_found():
    result = get_user(999)
    assert result is None  # Check None
```

### Pattern 3: Floating Point Comparison

```python
def calculate_average(numbers):
    return sum(numbers) / len(numbers)

# Ask: What should this return?
# Answer: Average (might have floating point imprecision)
def test_calculate_average():
    result = calculate_average([1, 2, 3])
    assert result == pytest.approx(2.0)  # Use approx for floats!

    # Why not assert result == 2.0?
    # Because: 0.1 + 0.2 == 0.30000000000000004 in Python
```

### Pattern 4: State Change

```python
class BankAccount:
    def __init__(self, balance=0):
        self.balance = balance

    def deposit(self, amount):
        self.balance += amount

# Ask: What should this do?
# Answer: Increase balance
def test_deposit():
    account = BankAccount(100)

    # Check before state
    assert account.balance == 100

    account.deposit(50)

    # Check after state
    assert account.balance == 150  # State changed
```

### Pattern 5: Exception Raised

```python
def divide(a, b):
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# Ask: What should this do?
# Answer: Raise ValueError
def test_divide_by_zero():
    with pytest.raises(ValueError) as exc_info:
        divide(10, 0)

    # Optional: Check error message
    assert "Cannot divide by zero" in str(exc_info.value)
```

### Pattern 6: Multiple Assertions

```python
def parse_name(full_name):
    parts = full_name.split()
    return {
        "first": parts[0],
        "last": parts[-1],
        "middle": " ".join(parts[1:-1]) if len(parts) > 2 else None
    }

# Ask: What should this return?
# Answer: Dict with first, last, middle
def test_parse_name_with_middle():
    result = parse_name("John Robert Smith")

    # Multiple related assertions are OK
    assert result["first"] == "John"
    assert result["middle"] == "Robert"
    assert result["last"] == "Smith"

    # Or check entire structure
    assert result == {
        "first": "John",
        "middle": "Robert",
        "last": "Smith"
    }
```

### Pattern 7: List/Collection Content

```python
def filter_even(numbers):
    return [n for n in numbers if n % 2 == 0]

# Ask: What should this return?
# Answer: List containing only even numbers
def test_filter_even():
    result = filter_even([1, 2, 3, 4, 5, 6])

    # Check exact list
    assert result == [2, 4, 6]

    # Or check properties
    assert len(result) == 3
    assert all(n % 2 == 0 for n in result)

    # Or check membership
    assert 2 in result
    assert 1 not in result
```

### Common Assertion Patterns

```python
# Equality
assert result == expected

# Identity
assert result is None
assert result is not None

# Type checking
assert isinstance(result, str)
assert type(result) == int

# Membership
assert item in collection
assert item not in collection

# Comparison
assert result > 0
assert result <= 100
assert min_val < result < max_val

# Boolean
assert condition
assert not condition

# String matching
assert "substring" in result
assert result.startswith("prefix")
assert result.endswith("suffix")

# Length
assert len(result) == 5
assert len(result) > 0

# Exceptions
with pytest.raises(ValueError):
    function_call()

with pytest.raises(ValueError, match="specific message"):
    function_call()

# Floating point
assert result == pytest.approx(3.14159, rel=1e-3)

# Multiple conditions
assert all([cond1, cond2, cond3])
assert any([cond1, cond2, cond3])
```

### "What Should I Assert?" Checklist

When writing assertions, ask:

- ✅ **Is this the expected result?** → assert equality
- ✅ **Could the type be wrong?** → assert isinstance()
- ✅ **Are there multiple properties to check?** → assert each one
- ✅ **Is this a floating point number?** → use pytest.approx()
- ✅ **Should this raise an error?** → use pytest.raises()
- ✅ **Did state change?** → assert before and after
- ✅ **Is the result a collection?** → check length, contents, membership
- ⚠️ **Am I asserting too many unrelated things?** → Split into separate tests
- ⚠️ **Am I testing implementation details?** → Focus on behavior

---

## How Many Tests Do I Need?

You've written some tests. Are you done? Here's how to decide.

### Minimum Coverage Rule

**Every function should have at least:**

1. **One normal case test** - typical usage
2. **One edge case test** - unusual but valid input
3. **One error case test** - invalid input (if applicable)

Example:

```python
def absolute_value(n):
    return -n if n < 0 else n

# Minimum 3 tests:
def test_absolute_positive():  # Normal case
    assert absolute_value(5) == 5

def test_absolute_zero():  # Edge case
    assert absolute_value(0) == 0

def test_absolute_negative():  # Another normal case (acts as error coverage)
    assert absolute_value(-5) == 5
```

### Complexity-Based Guide

| Function Complexity                        | Recommended Tests | Example                                |
| ------------------------------------------ | ----------------- | -------------------------------------- |
| **Trivial** (1-2 lines, no logic)          | 1-2 tests         | Simple getter, constant return         |
| **Simple** (Single if/else)                | 2-4 tests         | Basic validation, simple calculation   |
| **Moderate** (Multiple conditions)         | 5-8 tests         | Multiple validations, several branches |
| **Complex** (Loops, nested logic)          | 8-15 tests        | Data processing, algorithms            |
| **Very Complex** (Multiple methods, state) | 15+ tests         | Classes with multiple methods          |

### "Am I Done?" Checklist

Go through this checklist for each function:

```
✓ Tested normal/common case?
✓ Tested with empty inputs (empty string, empty list, zero, None)?
✓ Tested at boundaries (min/max values, limits)?
✓ Tested all error conditions (invalid inputs)?
✓ Tested all if/else branches?
✓ Tested edge cases specific to the domain?
✓ Can you think of another way it could break?
  ├─ If YES → Write another test
  └─ If NO → You're probably done

Optional (for important functions):
✓ Tested with large inputs (performance, memory)?
✓ Tested with special characters or unusual data?
✓ Tested concurrent usage (if relevant)?
```

### Examples: Simple vs Complex

#### Simple Function (3 tests is enough)

```python
def is_even(n):
    return n % 2 == 0

# Tests:
def test_is_even_with_even_number():
    assert is_even(4) is True

def test_is_even_with_odd_number():
    assert is_even(3) is False

def test_is_even_with_zero():
    assert is_even(0) is True

# Done! Only 3 tests needed - function is simple
```

#### Moderate Function (7 tests feels right)

```python
def format_phone(phone):
    """Format phone as (XXX) XXX-XXXX."""
    digits = ''.join(c for c in phone if c.isdigit())

    if len(digits) != 10:
        raise ValueError("Must be 10 digits")

    return f"({digits[:3]}) {digits[3:6]}-{digits[6:]}"

# Tests:
def test_format_phone_digits_only():
    assert format_phone("1234567890") == "(123) 456-7890"

def test_format_phone_with_dashes():
    assert format_phone("123-456-7890") == "(123) 456-7890"

def test_format_phone_with_spaces():
    assert format_phone("123 456 7890") == "(123) 456-7890"

def test_format_phone_with_parentheses():
    assert format_phone("(123) 456-7890") == "(123) 456-7890"

def test_format_phone_too_short():
    with pytest.raises(ValueError):
        format_phone("123456789")

def test_format_phone_too_long():
    with pytest.raises(ValueError):
        format_phone("12345678901")

def test_format_phone_no_digits():
    with pytest.raises(ValueError):
        format_phone("abc")

# 7 tests - covers all branches and edge cases
```

#### Complex Class (12+ tests needed)

```python
class ShoppingCart:
    def __init__(self):
        self.items = []

    def add_item(self, name, price, quantity=1):
        if price < 0:
            raise ValueError("Price cannot be negative")
        if quantity < 1:
            raise ValueError("Quantity must be at least 1")

        self.items.append({"name": name, "price": price, "quantity": quantity})

    def remove_item(self, name):
        self.items = [item for item in self.items if item["name"] != name]

    def get_total(self):
        return sum(item["price"] * item["quantity"] for item in self.items)

    def apply_discount(self, percent):
        if not 0 <= percent <= 100:
            raise ValueError("Discount must be 0-100")

        for item in self.items:
            item["price"] *= (1 - percent / 100)

# Tests needed:
# - Add item: normal, negative price, zero quantity
# - Remove item: exists, doesn't exist, empty cart
# - Get total: empty cart, single item, multiple items, with quantities
# - Apply discount: 0%, 50%, 100%, invalid discount
# - Integration: Add, discount, get total

# Total: 12-15 tests for thorough coverage
```

### When to STOP Writing Tests

**Stop when:**

1. ✅ All branches are covered
2. ✅ All edge cases are tested
3. ✅ All error conditions are tested
4. ✅ You can't think of new scenarios
5. ✅ Coverage tool shows 80-100% coverage (if using one)

**Don't stop just because:**

- ❌ You reached a specific number of tests
- ❌ You're tired (take a break, come back)
- ❌ Tests are passing (might be missing cases)

**Warning signs you need MORE tests:**

- 🚩 Function has 5 if statements but only 3 tests
- 🚩 Function handles errors but no error tests
- 🚩 You found a bug that existing tests didn't catch
- 🚩 Code review feedback points out missing cases
- 🚩 Coverage is below 70%

**Warning signs you have TOO MANY tests:**

- 🚩 Tests are nearly identical (consolidate them)
- 🚩 Testing trivial getters/setters
- 🚩 Testing the same thing multiple times
- 🚩 Tests take minutes to run for simple functions

---

## Common Beginner Mistakes

Learn from these common pitfalls and how to avoid them.

### Mistake 1: Testing Too Many Things in One Test

**Bad:**

```python
def test_user():
    user = User("Alice", "alice@example.com", 25)

    # Testing many unrelated things
    assert user.name == "Alice"
    assert user.email == "alice@example.com"
    assert user.age == 25

    user.deactivate()
    assert not user.active

    user.activate()
    assert user.active

    user.update_email("new@example.com")
    assert user.email == "new@example.com"

# Problem: If ANY assertion fails, you don't know which one
# Problem: Hard to understand what this test is actually verifying
```

**Good:**

```python
def test_user_creation_sets_name():
    user = User("Alice", "alice@example.com", 25)
    assert user.name == "Alice"

def test_user_creation_sets_email():
    user = User("Alice", "alice@example.com", 25)
    assert user.email == "alice@example.com"

def test_user_deactivation():
    user = User("Alice", "alice@example.com", 25)
    user.deactivate()
    assert not user.active

def test_user_activation():
    user = User("Alice", "alice@example.com", 25)
    user.deactivate()
    user.activate()
    assert user.active

def test_user_email_update():
    user = User("Alice", "alice@example.com", 25)
    user.update_email("new@example.com")
    assert user.email == "new@example.com"

# Better: Each test has a clear purpose
# Better: Failures are easy to diagnose
```

**How to recognize:** Test name contains "and" or tests multiple behaviors  
**Fix:** Split into multiple focused tests

### Mistake 2: Unclear Test Names

**Bad:**

```python
def test_1():
    assert add(2, 3) == 5

def test_user():
    user = User("Alice", "alice@example.com")
    user.deactivate()
    assert not user.active

def test_calc():
    result = calculate_discount(100, 20)
    assert result == 80
```

**Good:**

```python
def test_add_returns_sum_of_two_positive_numbers():
    assert add(2, 3) == 5

def test_user_deactivation_sets_active_to_false():
    user = User("Alice", "alice@example.com")
    user.deactivate()
    assert not user.active

def test_calculate_discount_reduces_price_by_percentage():
    result = calculate_discount(100, 20)
    assert result == 80

# Better: Test name tells you exactly what it tests
# Better: When it fails, you know immediately what broke
```

**How to recognize:** Test name doesn't describe what's being tested  
**Fix:** Name format: `test_<function>_<condition>_<expected_result>`

### Mistake 3: Tests Depending on Each Other

**Bad:**

```python
# Global state shared between tests!
user = None

def test_create_user():
    global user
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"

def test_deactivate_user():
    global user
    user.deactivate()  # Depends on test_create_user running first!
    assert not user.active

# Problem: Tests must run in order
# Problem: If test_create_user fails, test_deactivate_user also fails
# Problem: Can't run tests in isolation
```

**Good:**

```python
def test_create_user():
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"

def test_deactivate_user():
    user = User("Alice", "alice@example.com")  # Create fresh user
    user.deactivate()
    assert not user.active

# Better: Each test is independent
# Better: Tests can run in any order
# Better: Tests can be run individually
```

**How to recognize:** Tests fail when run individually but pass when run together  
**Fix:** Set up everything each test needs, don't share state

### Mistake 4: Not Testing Edge Cases

**Bad:**

```python
def test_divide():
    assert divide(6, 2) == 3

# Only tested the happy path!
# What about divide by zero?
# What about negative numbers?
# What about floats?
```

**Good:**

```python
def test_divide_positive_numbers():
    assert divide(6, 2) == 3

def test_divide_results_in_float():
    assert divide(5, 2) == 2.5

def test_divide_negative_numbers():
    assert divide(-6, 2) == -3
    assert divide(6, -2) == -3

def test_divide_by_zero_raises_error():
    with pytest.raises(ZeroDivisionError):
        divide(5, 0)

# Better: Tests important edge cases
```

**How to recognize:** Only 1-2 tests for a non-trivial function  
**Fix:** Use "How to Choose Test Values" section to identify edge cases

### Mistake 5: Testing Implementation Instead of Behavior

**Bad:**

```python
class UserService:
    def create_user(self, name, email):
        user = User(name, email)
        self._users.append(user)  # Implementation detail
        return user

def test_create_user():
    service = UserService()
    user = service.create_user("Alice", "alice@example.com")

    # Testing implementation: checking internal list
    assert user in service._users  # BAD: accessing private attribute

    # Problem: If implementation changes (use dict instead of list),
    # test breaks even though behavior is correct
```

**Good:**

```python
def test_create_user_returns_user_with_correct_attributes():
    service = UserService()
    user = service.create_user("Alice", "alice@example.com")

    # Testing behavior: what the function returns
    assert user.name == "Alice"
    assert user.email == "alice@example.com"

def test_created_user_can_be_retrieved():
    service = UserService()
    user = service.create_user("Alice", "alice@example.com")

    # Testing behavior: public API
    retrieved = service.get_user(user.id)
    assert retrieved == user

    # Better: Tests what the service DOES, not HOW it does it
```

**How to recognize:** Tests access private attributes (\_variable) or check internal state  
**Fix:** Test public API and observable behavior only

### Mistake 6: Not Using Arrange-Act-Assert Structure

**Bad:**

```python
def test_deposit():
    account = BankAccount(100)
    account.deposit(50)
    assert account.balance == 150
    account.deposit(25)
    assert account.balance == 175

    # Hard to read: what's setup and what's being tested?
```

**Good:**

```python
def test_deposit_increases_balance():
    # Arrange: Set up test data
    account = BankAccount(100)

    # Act: Perform the action
    account.deposit(50)

    # Assert: Verify the result
    assert account.balance == 150

def test_multiple_deposits():
    # Arrange
    account = BankAccount(100)

    # Act
    account.deposit(50)
    account.deposit(25)

    # Assert
    assert account.balance == 175

# Better: Clear structure, easy to understand
```

**How to recognize:** Test is hard to read, actions and assertions are mixed  
**Fix:** Organize with blank lines: Arrange, Act, Assert

### Mistake 7: Copy-Pasting Tests Without Understanding

**Bad:**

```python
# Copied from internet without understanding
def test_user_creation():
    with patch('database.save') as mock_save:
        user = User("Alice", "alice@example.com")
        service.create_user(user)
        mock_save.assert_called_once()

    # But I don't even have a database!
    # I don't understand what patch does!
    # Test doesn't work and I don't know why!
```

**Good:**

```python
# Start simple, understand each part
def test_user_has_correct_name():
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"

# Once you understand basics, then learn mocking:
# 1. Read documentation
# 2. Understand why mocking is needed
# 3. Start with simple mock examples
# 4. Build up complexity gradually
```

**How to recognize:** You can't explain what the test does or why  
**Fix:** Start with simple tests, learn concepts one at a time

### Quick Recognition Guide

| Symptom                                  | Likely Mistake                    | Fix                            |
| ---------------------------------------- | --------------------------------- | ------------------------------ |
| Test name contains "and"                 | Testing too much                  | Split into multiple tests      |
| Test fails when run alone                | Tests depend on each other        | Make tests independent         |
| Only 1-2 tests for complex function      | Missing edge cases                | Apply value selection process  |
| Test breaks when refactoring             | Testing implementation            | Test behavior, not internals   |
| Can't understand test purpose            | Unclear naming                    | Use descriptive names          |
| Test is hard to read                     | No AAA structure                  | Organize: Arrange, Act, Assert |
| Test doesn't work and you don't know why | Copy-pasted without understanding | Start simple, learn gradually  |

---

## Quick Reference Cheat Sheet

### Function Analysis Workflow

```
1. Read function → Understand what it does
2. Identify inputs → Note types and constraints
3. Identify outputs → Return value, side effects, or exceptions
4. Choose test scenarios:
   ├─ Normal case (typical usage)
   ├─ Edge cases (empty, zero, None, boundaries)
   └─ Error cases (invalid inputs)
5. For each scenario:
   ├─ Choose specific test values
   ├─ Determine expected result
   └─ Write test with descriptive name
6. Check: Did I cover all branches? All edge cases?
```

### Test Value Selection Cheat Sheet

| Input Type    | Must Test                              | Consider Testing                      |
| ------------- | -------------------------------------- | ------------------------------------- |
| **int/float** | Positive, Negative, Zero               | Large, Boundaries, Special (inf, NaN) |
| **string**    | Empty "", Normal "hello"               | Single char, Long, Special chars      |
| **list**      | Empty [], Single [1], Multiple [1,2,3] | Large, Duplicates, None values        |
| **dict**      | Empty {}, With keys                    | Missing expected keys                 |
| **bool**      | True, False                            | None (if allowed)                     |
| **None**      | None, Valid value                      | Edge values                           |

### Assertion Pattern Selector

| What Function Does     | Use This Assertion                                   |
| ---------------------- | ---------------------------------------------------- |
| Returns specific value | `assert result == expected`                          |
| Returns float          | `assert result == pytest.approx(expected)`           |
| Returns object         | `assert isinstance(result, Type)`                    |
| Raises exception       | `with pytest.raises(ExceptionType):`                 |
| Changes state          | `assert obj.attribute == expected`                   |
| Returns collection     | `assert len(result) == X` or `assert item in result` |

### Common Test Patterns

```python
# Pattern: Normal case
def test_function_with_typical_input():
    result = function(typical_input)
    assert result == expected_output

# Pattern: Edge case
def test_function_with_empty_input():
    result = function([])
    assert result == expected_for_empty

# Pattern: Error case
def test_function_with_invalid_input_raises_error():
    with pytest.raises(ValueError):
        function(invalid_input)

# Pattern: State change
def test_method_changes_object_state():
    obj = MyClass()
    obj.method()
    assert obj.attribute == expected_new_value

# Pattern: Multiple assertions (related)
def test_function_returns_dict_with_expected_structure():
    result = function(input)
    assert result["key1"] == value1
    assert result["key2"] == value2
```

### Test Count Estimator

- **Simple function** (1 if/else): 2-4 tests
- **Moderate function** (multiple conditions): 5-8 tests
- **Complex function** (loops, many branches): 8-15 tests
- **Class with multiple methods**: 3-5 tests per method

### "Am I Done?" Quick Check

```
✓ Tested normal case?
✓ Tested edge cases (empty, zero, None)?
✓ Tested boundaries?
✓ Tested error cases?
✓ All if/else branches covered?
✓ Can't think of how else it could break?

If all YES → You're done!
```

---

## Practice Exercises

Test your understanding with these exercises. Try them before looking at the solutions!

### Exercise 1: Calculate Age

Write tests for this function:

```python
def calculate_age(birth_year, current_year):
    """Calculate age from birth year and current year.

    Args:
        birth_year: Year of birth (must be 1900-current_year)
        current_year: Current year

    Returns:
        Age in years

    Raises:
        ValueError: If birth_year is invalid
    """
    if birth_year < 1900:
        raise ValueError("Birth year must be 1900 or later")

    if birth_year > current_year:
        raise ValueError("Birth year cannot be in the future")

    return current_year - birth_year
```

**Tasks:**

1. How many tests should you write?
2. What test values would you choose?
3. Write the tests!

<details>
<summary>Click to see solution</summary>

**Answer:**

You should write about 5-7 tests:

```python
def test_calculate_age_normal_case():
    """Test with typical values."""
    assert calculate_age(1990, 2024) == 34

def test_calculate_age_boundary_1900():
    """Test minimum valid year."""
    assert calculate_age(1900, 2024) == 124

def test_calculate_age_same_year():
    """Test when born in current year (age 0)."""
    assert calculate_age(2024, 2024) == 0

def test_calculate_age_before_1900():
    """Test that birth year before 1900 raises error."""
    with pytest.raises(ValueError, match="must be 1900 or later"):
        calculate_age(1899, 2024)

def test_calculate_age_future_year():
    """Test that future birth year raises error."""
    with pytest.raises(ValueError, match="cannot be in the future"):
        calculate_age(2025, 2024)

def test_calculate_age_one_year_old():
    """Test age of 1."""
    assert calculate_age(2023, 2024) == 1

def test_calculate_age_old_person():
    """Test with large age."""
    assert calculate_age(1920, 2024) == 104
```

**Why these tests?**

- Normal case: typical values (1990, 2024)
- Boundaries: 1900, same year (age 0)
- Errors: before 1900, future year
- Edge cases: age 1, old age
</details>

### Exercise 2: Find Maximum

Write tests for this function:

```python
def find_max(numbers):
    """Find maximum number in list.

    Args:
        numbers: List of numbers

    Returns:
        Maximum number

    Raises:
        ValueError: If list is empty
    """
    if not numbers:
        raise ValueError("List cannot be empty")

    max_num = numbers[0]
    for num in numbers[1:]:
        if num > max_num:
            max_num = num
    return max_num
```

**Tasks:**

1. What edge cases should you test?
2. Write at least 5 tests

<details>
<summary>Click to see solution</summary>

**Answer:**

```python
def test_find_max_multiple_numbers():
    """Test with typical list."""
    assert find_max([3, 1, 4, 1, 5, 9, 2]) == 9

def test_find_max_single_number():
    """Test with single item."""
    assert find_max([42]) == 42

def test_find_max_all_negative():
    """Test with all negative numbers."""
    assert find_max([-5, -2, -8, -1]) == -1

def test_find_max_with_duplicates():
    """Test when max appears multiple times."""
    assert find_max([5, 1, 5, 3, 5]) == 5

def test_find_max_first_is_max():
    """Test when first element is maximum."""
    assert find_max([10, 2, 3, 4]) == 10

def test_find_max_last_is_max():
    """Test when last element is maximum."""
    assert find_max([1, 2, 3, 10]) == 10

def test_find_max_empty_list():
    """Test that empty list raises error."""
    with pytest.raises(ValueError, match="cannot be empty"):
        find_max([])
```

**Edge cases identified:**

- Single item
- All negative
- Duplicates
- Max at start/end
- Empty list (error case)
</details>

### Exercise 3: Validate Password

Given this function, identify what tests you would write (don't write the full code, just list the test cases):

```python
def validate_password(password):
    """Validate password meets requirements.

    Requirements:
    - At least 8 characters
    - Contains at least one uppercase letter
    - Contains at least one lowercase letter
    - Contains at least one digit

    Args:
        password: Password string

    Returns:
        True if valid, False if invalid
    """
    if len(password) < 8:
        return False

    has_upper = any(c.isupper() for c in password)
    has_lower = any(c.islower() for c in password)
    has_digit = any(c.isdigit() for c in password)

    return has_upper and has_lower and has_digit
```

**Task:** List out test cases you would write (test name + input + expected result)

<details>
<summary>Click to see solution</summary>

**Answer:**

Test cases to write:

1. `test_validate_password_meets_all_requirements`
   - Input: "Passw0rd"
   - Expected: True

2. `test_validate_password_too_short`
   - Input: "Pass1"
   - Expected: False

3. `test_validate_password_exactly_8_chars`
   - Input: "Passw0rd" (8 chars)
   - Expected: True

4. `test_validate_password_no_uppercase`
   - Input: "password123"
   - Expected: False

5. `test_validate_password_no_lowercase`
   - Input: "PASSWORD123"
   - Expected: False

6. `test_validate_password_no_digit`
   - Input: "Password"
   - Expected: False

7. `test_validate_password_empty_string`
   - Input: ""
   - Expected: False

8. `test_validate_password_long_valid`
   - Input: "ThisIsAVeryLongPassword123"
   - Expected: True

9. `test_validate_password_special_chars_allowed`
   - Input: "P@ssw0rd!"
   - Expected: True

Total: 9 tests covering normal case, each requirement violation, boundaries, and edge cases.

</details>

### Exercise 4: Debug This Test

What's wrong with this test? How would you fix it?

```python
def test_user():
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"
    assert user.email == "alice@example.com"

    user.set_age(25)
    assert user.age == 25

    user.deactivate()
    assert user.active == False

    user.update_email("newemail@example.com")
    assert user.email == "newemail@example.com"
```

Click to see solution

**Problems:**

1. **Testing too many things** - This test verifies creation, age setting, deactivation, and email update
2. **Unclear test name** - "test_user" doesn't describe what's being tested
3. **Hard to debug** - If any assertion fails, you don't know which behavior broke

**Fixed version:**

```python
def test_user_creation_sets_name_and_email():
    """Test that user is created with correct name and email."""
    user = User("Alice", "alice@example.com")
    assert user.name == "Alice"
    assert user.email == "alice@example.com"

def test_user_set_age_updates_age_attribute():
    """Test that set_age updates the age."""
    user = User("Alice", "alice@example.com")
    user.set_age(25)
    assert user.age == 25

def test_user_deactivate_sets_active_to_false():
    """Test that deactivate sets active flag to False."""
    user = User("Alice", "alice@example.com")
    user.deactivate()
    assert user.active is False

def test_user_update_email_changes_email():
    """Test that update_email changes the email."""
    user = User("Alice", "alice@example.com")
    user.update_email("newemail@example.com")
    assert user.email == "newemail@example.com"
```

**Improvements:**

- Each test has one clear purpose
- Descriptive test names
- Easy to debug - failures point to specific functionality
- Tests are independent

---

## Summary

**You've learned the practical process of writing tests:**

1. **Choose what to test** using the decision flowchart
2. **Choose test values** with the 4-step systematic process
3. **Determine assertions** based on what the function does
4. **Know when you're done** using the checklist
5. **Avoid common mistakes** by recognizing patterns

**Key principles to remember:**

- Test behavior, not implementation
- Each test should verify one thing
- Use descriptive names
- Follow Arrange-Act-Assert
- Tests should be independent
- Focus on important code (business logic, validation, error handling)

**Next steps:**

- Practice with [Test Examples Walkthrough](test-examples-walkthrough.md)
- Learn more concepts in [Testing Fundamentals](testing-fundamentals.md)
- Master test frameworks in [Unit Testing](unit-testing.md)

**Remember:** Testing is a skill that improves with practice. Your first tests won't be perfect, and that's okay! Start simple, learn from mistakes, and keep practicing.
