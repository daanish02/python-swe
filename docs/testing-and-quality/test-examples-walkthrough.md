# Test Examples Walkthrough

## About This Guide

This guide provides **complete examples** showing the entire thought process from function to tests. Each example shows:

1. 📝 **The Function** - Code to test
2. 🧠 **Analysis** - Understanding what it does
3. 🎯 **Test Identification** - What scenarios to test
4. 🎲 **Value Selection** - Why specific test values
5. ✅ **Complete Tests** - Final test code with rationale

Work through these examples to internalize the testing process.

**How to use this guide:**
- Read the function first
- Try to think through what tests YOU would write
- Then compare with the analysis
- Notice the thinking patterns

---

## Table of Contents

- [Example 1: Calculate Tax (Simple Math)](#example-1-calculate-tax)
- [Example 2: Validate Email (String Validation)](#example-2-validate-email)
- [Example 3: Remove Duplicates (List Processing)](#example-3-remove-duplicates)
- [Example 4: Calculate Shipping (Business Logic)](#example-4-calculate-shipping)
- [Example 5: Format Currency (String Formatting)](#example-5-format-currency)
- [Example 6: Find Median (Algorithm)](#example-6-find-median)
- [Example 7: Parse CSV Line (String Parsing)](#example-7-parse-csv-line)
- [Example 8: Grade Calculator (Conditionals)](#example-8-grade-calculator)
- [Example 9: BankAccount Class (Object State)](#example-9-bankaccount-class)
- [Example 10: Merge Sorted Lists (Algorithm)](#example-10-merge-sorted-lists)

---

## Example 1: Calculate Tax

### 📝 The Function

```python
def calculate_tax(amount, tax_rate):
    """Calculate tax on an amount.
    
    Args:
        amount: Amount to calculate tax on (must be non-negative)
        tax_rate: Tax rate as decimal (e.g., 0.15 for 15%)
    
    Returns:
        Tax amount
    
    Raises:
        ValueError: If amount is negative or tax_rate is invalid
    """
    if amount < 0:
        raise ValueError("Amount cannot be negative")
    
    if tax_rate < 0 or tax_rate > 1:
        raise ValueError("Tax rate must be between 0 and 1")
    
    return amount * tax_rate
```

### 🧠 Analysis

**What does this do?**
- Multiplies amount by tax_rate to get tax
- Validates inputs: amount ≥ 0, tax_rate between 0 and 1
- Raises ValueError for invalid inputs

**What could go wrong?**
- Negative amount
- Tax rate outside 0-1 range
- Edge cases: zero amount, zero tax rate, 100% tax

### 🎯 Test Identification

**Scenarios to test:**
1. Normal case: typical amount and tax rate
2. Edge case: zero amount
3. Edge case: zero tax rate (0% tax)
4. Edge case: maximum tax rate (100% tax)
5. Error case: negative amount
6. Error case: negative tax rate
7. Error case: tax rate > 1

### 🎲 Value Selection

**Why these specific values?**

- **amount=100, tax_rate=0.15**: Round numbers, easy to verify (expect 15.0)
- **amount=0**: Edge case for amount boundary
- **tax_rate=0**: Edge case for tax rate boundary
- **tax_rate=1**: Maximum valid tax rate
- **amount=-50**: Invalid input to trigger error
- **tax_rate=-0.1**: Invalid rate (negative)
- **tax_rate=1.5**: Invalid rate (> 1)

### ✅ Complete Tests

```python
import pytest
from tax import calculate_tax

def test_calculate_tax_typical_values():
    """Test tax calculation with normal values."""
    # Why 100 and 0.15? Easy to verify: 100 * 0.15 = 15
    result = calculate_tax(100, 0.15)
    assert result == 15.0

def test_calculate_tax_zero_amount():
    """Test that zero amount returns zero tax."""
    # Edge case: 0 multiplied by anything is 0
    result = calculate_tax(0, 0.15)
    assert result == 0.0

def test_calculate_tax_zero_rate():
    """Test that zero tax rate returns zero tax."""
    # Edge case: 0% tax means no tax
    result = calculate_tax(100, 0)
    assert result == 0.0

def test_calculate_tax_maximum_rate():
    """Test with 100% tax rate."""
    # Edge case: 100% tax means tax equals amount
    result = calculate_tax(100, 1.0)
    assert result == 100.0

def test_calculate_tax_small_amount():
    """Test with small decimal amount."""
    # Real-world case: prices with cents
    result = calculate_tax(19.99, 0.08)
    assert result == pytest.approx(1.5992)  # Use approx for floats!

def test_calculate_tax_negative_amount_raises_error():
    """Test that negative amount raises ValueError."""
    # Invalid input: can't have negative amount
    with pytest.raises(ValueError, match="Amount cannot be negative"):
        calculate_tax(-50, 0.15)

def test_calculate_tax_negative_rate_raises_error():
    """Test that negative tax rate raises ValueError."""
    # Invalid input: can't have negative tax rate
    with pytest.raises(ValueError, match="must be between 0 and 1"):
        calculate_tax(100, -0.1)

def test_calculate_tax_rate_above_one_raises_error():
    """Test that tax rate > 1 raises ValueError."""
    # Invalid input: 150% tax doesn't make sense
    with pytest.raises(ValueError, match="must be between 0 and 1"):
        calculate_tax(100, 1.5)
```

**Rationale:**
- 8 tests cover all branches and edge cases
- Used round numbers where possible for clarity
- Used pytest.approx for floating-point comparison
- Each error case has descriptive match parameter

---

## Example 2: Validate Email

### 📝 The Function

```python
def validate_email(email):
    """Check if email address is valid.
    
    Simple validation: must contain exactly one @ with text before and after.
    
    Args:
        email: Email string to validate
    
    Returns:
        True if valid, False otherwise
    """
    if not email:
        return False
    
    if email.count("@") != 1:
        return False
    
    local, domain = email.split("@")
    
    if not local or not domain:
        return False
    
    if "." not in domain:
        return False
    
    return True
```

### 🧠 Analysis

**What does this do?**
- Validates email has exactly one @
- Checks there's text before and after @
- Checks domain has at least one dot
- Returns boolean (no exceptions)

**What could go wrong?**
- Empty string
- No @
- Multiple @
- @ at start or end
- Domain without dot

### 🎯 Test Identification

**Scenarios to test:**
1. Valid email: typical format
2. Valid email: with subdomain
3. Invalid: empty string
4. Invalid: no @
5. Invalid: multiple @
6. Invalid: @ at start
7. Invalid: @ at end
8. Invalid: domain without dot
9. Invalid: None (if function should handle it)

### 🎲 Value Selection

**Why these specific values?**

- **"user@example.com"**: Classic valid email format
- **"name@mail.company.org"**: Valid with subdomain
- **""**: Empty string edge case
- **"userexample.com"**: Missing @ entirely
- **"user@@example.com"**: Multiple @ symbols
- **"@example.com"**: @ at start (no local part)
- **"user@"**: @ at end (no domain)
- **"user@example"**: Domain without dot

### ✅ Complete Tests

```python
from email_validator import validate_email

def test_validate_email_typical_valid():
    """Test with standard valid email."""
    # Why this value? Classic email format everyone recognizes
    assert validate_email("user@example.com") is True

def test_validate_email_with_subdomain():
    """Test valid email with subdomain."""
    # Real-world case: many emails have subdomains
    assert validate_email("user@mail.company.org") is True

def test_validate_email_with_dots_in_local():
    """Test valid email with dots in local part."""
    # Common pattern: first.last@example.com
    assert validate_email("first.last@example.com") is True

def test_validate_email_empty_string():
    """Test that empty string is invalid."""
    # Edge case: empty input
    assert validate_email("") is False

def test_validate_email_no_at_symbol():
    """Test that email without @ is invalid."""
    # Missing required character
    assert validate_email("userexample.com") is False

def test_validate_email_multiple_at_symbols():
    """Test that multiple @ symbols make email invalid."""
    # Invalid format: only one @ allowed
    assert validate_email("user@@example.com") is False
    assert validate_email("user@domain@example.com") is False

def test_validate_email_at_start():
    """Test that @ at start is invalid."""
    # No local part means invalid
    assert validate_email("@example.com") is False

def test_validate_email_at_end():
    """Test that @ at end is invalid."""
    # No domain part means invalid
    assert validate_email("user@") is False

def test_validate_email_domain_without_dot():
    """Test that domain without dot is invalid."""
    # Technically invalid: domains need TLD
    assert validate_email("user@example") is False

def test_validate_email_only_at_symbol():
    """Test that just @ is invalid."""
    # Edge case: minimal invalid input
    assert validate_email("@") is False

def test_validate_email_spaces():
    """Test that spaces make email invalid."""
    # Emails shouldn't have spaces
    assert validate_email("user @example.com") is False
    assert validate_email("user@ example.com") is False
```

**Rationale:**
- 11 tests cover valid and invalid cases
- Covers all validation rules
- Tests common real-world formats
- Tests edge cases (empty, just @)

---

## Example 3: Remove Duplicates

### 📝 The Function

```python
def remove_duplicates(items):
    """Remove duplicate items while preserving order.
    
    Args:
        items: List of items
    
    Returns:
        New list with duplicates removed, first occurrence kept
    """
    seen = set()
    result = []
    
    for item in items:
        if item not in seen:
            seen.add(item)
            result.append(item)
    
    return result
```

### 🧠 Analysis

**What does this do?**
- Removes duplicates from list
- Preserves original order
- Keeps first occurrence of each item
- Returns new list (doesn't modify original)

**What could go wrong?**
- Empty list
- No duplicates
- All duplicates
- Order not preserved

### 🎯 Test Identification

**Scenarios to test:**
1. Normal case: list with some duplicates
2. Edge case: empty list
3. Edge case: single item
4. Edge case: no duplicates
5. Edge case: all same items
6. Verify: order is preserved
7. Verify: first occurrence kept, not last
8. Works with different types (strings, numbers)

### 🎲 Value Selection

```python
# [1, 2, 2, 3, 1, 4] → [1, 2, 3, 4]
# Why? Shows duplicates scattered, tests order preservation

# [] → []
# Why? Edge case for empty

# [5] → [5]
# Why? Edge case for single item

# [1, 2, 3] → [1, 2, 3]
# Why? No duplicates case

# [7, 7, 7] → [7]
# Why? All duplicates case
```

### ✅ Complete Tests

```python
from duplicates import remove_duplicates

def test_remove_duplicates_with_duplicates():
    """Test removing duplicates from list with some."""
    # Mixed: shows it works with duplicates scattered throughout
    result = remove_duplicates([1, 2, 2, 3, 1, 4])
    assert result == [1, 2, 3, 4]

def test_remove_duplicates_empty_list():
    """Test with empty list."""
    # Edge case: empty input should return empty output
    result = remove_duplicates([])
    assert result == []

def test_remove_duplicates_single_item():
    """Test with single item."""
    # Edge case: single item has no duplicates
    result = remove_duplicates([5])
    assert result == [5]

def test_remove_duplicates_no_duplicates():
    """Test list with no duplicates."""
    # All unique: should return same items in same order
    result = remove_duplicates([1, 2, 3, 4, 5])
    assert result == [1, 2, 3, 4, 5]

def test_remove_duplicates_all_same():
    """Test with all identical items."""
    # Edge case: all duplicates should result in single item
    result = remove_duplicates([7, 7, 7, 7])
    assert result == [7]

def test_remove_duplicates_preserves_order():
    """Test that original order is maintained."""
    # Important: first occurrence of each item should remain
    result = remove_duplicates([3, 1, 4, 1, 5, 9, 2, 6, 5])
    assert result == [3, 1, 4, 5, 9, 2, 6]
    # Note: 1 appears once (first at index 1), 5 appears once (first at index 4)

def test_remove_duplicates_keeps_first_occurrence():
    """Test that first occurrence is kept, not last."""
    # If we later need to change to keep last, this test will catch it
    items = ["a", "b", "a", "c"]
    result = remove_duplicates(items)
    assert result == ["a", "b", "c"]  # First 'a' at index 0

def test_remove_duplicates_with_strings():
    """Test with string items."""
    # Works with different types
    result = remove_duplicates(["apple", "banana", "apple", "cherry"])
    assert result == ["apple", "banana", "cherry"]

def test_remove_duplicates_does_not_modify_original():
    """Test that original list is not modified."""
    # Important: function should not have side effects
    original = [1, 2, 2, 3]
    result = remove_duplicates(original)
    assert original == [1, 2, 2, 3]  # Original unchanged
    assert result == [1, 2, 3]
```

**Rationale:**
- 9 tests cover functionality and edge cases
- Tests behavior (order preservation) not just output
- Tests side effects (or lack thereof)
- Tests with different data types

---

## Example 4: Calculate Shipping

### 📝 The Function

```python
def calculate_shipping(weight_kg, distance_km, express=False):
    """Calculate shipping cost.
    
    Rules:
    - Base rate: $5
    - Weight: $2 per kg
    - Distance: $0.50 per km over 100km
    - Express: 50% surcharge on total
    
    Args:
        weight_kg: Package weight in kg (must be > 0)
        distance_km: Shipping distance in km (must be > 0)
        express: Whether express shipping
    
    Returns:
        Total shipping cost
    
    Raises:
        ValueError: If weight or distance is not positive
    """
    if weight_kg <= 0:
        raise ValueError("Weight must be positive")
    
    if distance_km <= 0:
        raise ValueError("Distance must be positive")
    
    # Base rate
    cost = 5.0
    
    # Weight charge
    cost += weight_kg * 2.0
    
    # Distance charge (only over 100km)
    if distance_km > 100:
        cost += (distance_km - 100) * 0.50
    
    # Express surcharge
    if express:
        cost *= 1.5
    
    return cost
```

### 🧠 Analysis

**What does this do?**
- Calculates shipping cost with multiple factors
- Base rate + weight charge + distance charge
- Distance charge only applies > 100km
- Express adds 50% to total
- Validates positive weight and distance

**Business logic to verify:**
- Base rate always included
- Weight calculation correct
- Distance threshold (100km) works
- Express multiplier correct
- All factors combine correctly

### 🎯 Test Identification

**Scenarios to test:**
1. Normal case: typical shipping
2. Small distance (< 100km): no distance charge
3. Exactly 100km: boundary for distance charge
4. Over 100km: distance charge applies
5. Express shipping: verify 50% surcharge
6. Express + long distance: all factors
7. Very light package: minimal weight
8. Heavy package: significant weight charge
9. Zero weight: error
10. Negative weight: error
11. Zero distance: error
12. Negative distance: error

### 🎲 Value Selection

**Why these values?**

```python
# weight=10, distance=50, express=False
# → $5 base + $20 weight + $0 distance = $25
# Why? Simple calculation, distance < 100

# weight=5, distance=100, express=False
# → $5 + $10 + $0 = $15
# Why? Exactly at 100km boundary

# weight=5, distance=150, express=False
# → $5 + $10 + $25 (50km * $0.50) = $40
# Why? Tests distance charge

# weight=10, distance=50, express=True
# → ($5 + $20) * 1.5 = $37.50
# Why? Tests express multiplier
```

### ✅ Complete Tests

```python
import pytest
from shipping import calculate_shipping

def test_calculate_shipping_basic():
    """Test basic shipping calculation."""
    # 10kg, 50km, not express
    # $5 base + $20 weight + $0 distance = $25
    result = calculate_shipping(10, 50)
    assert result == 25.0

def test_calculate_shipping_short_distance():
    """Test that distance under 100km has no distance charge."""
    # Distance < 100km means no distance surcharge
    result = calculate_shipping(5, 50)
    # $5 base + $10 weight = $15
    assert result == 15.0

def test_calculate_shipping_exactly_100km():
    """Test boundary: exactly 100km."""
    # At 100km, still no distance charge
    result = calculate_shipping(5, 100)
    # $5 + $10 = $15
    assert result == 15.0

def test_calculate_shipping_over_100km():
    """Test that distance over 100km adds distance charge."""
    # 150km means 50km over limit
    result = calculate_shipping(5, 150)
    # $5 base + $10 weight + $25 distance (50 * 0.50) = $40
    assert result == 40.0

def test_calculate_shipping_express():
    """Test express shipping adds 50% surcharge."""
    # Express multiplies total by 1.5
    result = calculate_shipping(10, 50, express=True)
    # ($5 + $20) * 1.5 = $37.50
    assert result == 37.50

def test_calculate_shipping_express_long_distance():
    """Test express with long distance combines all factors."""
    # All factors: base + weight + distance, then express multiplier
    result = calculate_shipping(10, 150, express=True)
    # ($5 + $20 + $25) * 1.5 = $75
    assert result == 75.0

def test_calculate_shipping_light_package():
    """Test with very light package."""
    # Minimum weight case
    result = calculate_shipping(0.5, 50)
    # $5 + $1 = $6
    assert result == 6.0

def test_calculate_shipping_heavy_package():
    """Test with heavy package."""
    # Weight is major cost factor
    result = calculate_shipping(50, 50)
    # $5 + $100 = $105
    assert result == 105.0

def test_calculate_shipping_zero_weight_raises_error():
    """Test that zero weight raises ValueError."""
    with pytest.raises(ValueError, match="Weight must be positive"):
        calculate_shipping(0, 100)

def test_calculate_shipping_negative_weight_raises_error():
    """Test that negative weight raises ValueError."""
    with pytest.raises(ValueError, match="Weight must be positive"):
        calculate_shipping(-5, 100)

def test_calculate_shipping_zero_distance_raises_error():
    """Test that zero distance raises ValueError."""
    with pytest.raises(ValueError, match="Distance must be positive"):
        calculate_shipping(10, 0)

def test_calculate_shipping_negative_distance_raises_error():
    """Test that negative distance raises ValueError."""
    with pytest.raises(ValueError, match="Distance must be positive"):
        calculate_shipping(10, -50)
```

**Rationale:**
- 12 tests cover complex business logic
- Each business rule tested individually
- Combination scenarios tested
- Boundary values tested (100km threshold)
- All validation tested

---

## Example 5: Format Currency

### 📝 The Function

```python
def format_currency(amount, currency="USD"):
    """Format amount as currency string.
    
    Args:
        amount: Numeric amount
        currency: Currency code (USD, EUR, GBP)
    
    Returns:
        Formatted string (e.g., "$1,234.56")
    
    Raises:
        ValueError: If currency is not supported
    """
    symbols = {
        "USD": "$",
        "EUR": "€",
        "GBP": "£"
    }
    
    if currency not in symbols:
        raise ValueError(f"Unsupported currency: {currency}")
    
    # Format with thousands separator and 2 decimals
    formatted_amount = f"{abs(amount):,.2f}"
    
    # Add symbol and handle negative
    if amount < 0:
        return f"-{symbols[currency]}{formatted_amount}"
    else:
        return f"{symbols[currency]}{formatted_amount}"
```

### 🧠 Analysis

**What does this do?**
- Formats numbers as currency strings
- Supports USD, EUR, GBP
- Adds appropriate symbol
- Thousands separator (commas)
- Always 2 decimal places
- Handles negative amounts

**Formatting rules to test:**
- Correct symbol for currency
- Thousands separators
- Decimal places
- Negative number formatting
- Zero handling

### 🎯 Test Identification

1. USD with typical amount
2. EUR and GBP: verify other currencies
3. Large amount: thousands separators
4. Small amount: decimal handling
5. Zero
6. Negative amount
7. Very large number
8. Amount with many decimal places (rounding)
9. Unsupported currency: error

### 🎲 Value Selection

```python
# 1234.56, "USD" → "$1,234.56"
# Why? Tests thousands separator and decimals

# 0, "USD" → "$0.00"
# Why? Edge case for zero

# -100, "USD" → "-$100.00"
# Why? Tests negative formatting

# 1234567.89, "USD" → "$1,234,567.89"
# Why? Multiple thousands separators

# 1234.567, "USD" → "$1,234.57"
# Why? Tests rounding to 2 decimals
```

### ✅ Complete Tests

```python
import pytest
from currency import format_currency

def test_format_currency_usd_typical():
    """Test USD formatting with typical amount."""
    # Shows thousands separator and cents
    assert format_currency(1234.56, "USD") == "$1,234.56"

def test_format_currency_eur():
    """Test EUR uses correct symbol."""
    assert format_currency(100.00, "EUR") == "€100.00"

def test_format_currency_gbp():
    """Test GBP uses correct symbol."""
    assert format_currency(100.00, "GBP") == "£100.00"

def test_format_currency_default_is_usd():
    """Test that default currency is USD."""
    # No currency specified should use USD
    assert format_currency(100.00) == "$100.00"

def test_format_currency_zero():
    """Test formatting zero amount."""
    # Edge case: zero should still show .00
    assert format_currency(0, "USD") == "$0.00"

def test_format_currency_negative():
    """Test negative amount formatting."""
    # Negative should have minus before symbol
    assert format_currency(-100.50, "USD") == "-$100.50"

def test_format_currency_large_amount():
    """Test large amount with multiple thousands separators."""
    # Shows multiple commas
    assert format_currency(1234567.89, "USD") == "$1,234,567.89"

def test_format_currency_small_amount():
    """Test small amount under 1."""
    # Less than $1
    assert format_currency(0.99, "USD") == "$0.99"

def test_format_currency_rounds_to_two_decimals():
    """Test that amount is rounded to 2 decimal places."""
    # 3 decimal places should round
    assert format_currency(1234.567, "USD") == "$1,234.57"
    assert format_currency(1234.564, "USD") == "$1,234.56"

def test_format_currency_no_decimals_shows_zeroes():
    """Test whole number shows .00."""
    # Important for currency display
    assert format_currency(100, "USD") == "$100.00"

def test_format_currency_large_negative():
    """Test large negative amount."""
    assert format_currency(-9999.99, "USD") == "-$9,999.99"

def test_format_currency_unsupported_currency_raises_error():
    """Test that unsupported currency raises ValueError."""
    with pytest.raises(ValueError, match="Unsupported currency: JPY"):
        format_currency(100, "JPY")

def test_format_currency_invalid_currency_code():
    """Test with invalid currency code."""
    with pytest.raises(ValueError, match="Unsupported currency"):
        format_currency(100, "INVALID")
```

**Rationale:**
- 13 tests cover all formatting rules
- Tests each supported currency
- Tests edge cases (zero, negative, rounding)
- Tests string formatting details
- Tests error handling

---

## Example 6: Find Median

### 📝 The Function

```python
def find_median(numbers):
    """Find median value in list of numbers.
    
    Args:
        numbers: List of numbers
    
    Returns:
        Median value (middle value for odd length,
        average of two middle values for even length)
    
    Raises:
        ValueError: If list is empty
    """
    if not numbers:
        raise ValueError("Cannot find median of empty list")
    
    sorted_nums = sorted(numbers)
    n = len(sorted_nums)
    
    if n % 2 == 1:
        # Odd length: return middle value
        return sorted_nums[n // 2]
    else:
        # Even length: return average of two middle values
        mid1 = sorted_nums[n // 2 - 1]
        mid2 = sorted_nums[n // 2]
        return (mid1 + mid2) / 2
```

### 🧠 Analysis

**What does this do?**
- Sorts numbers
- Returns middle value (odd length) or average of two middles (even length)
- Handles empty list error

**Algorithm behavior to verify:**
- Odd length: returns exact middle
- Even length: returns average
- Sorts first (order doesn't matter in input)
- Works with unsorted input
- Doesn't modify original list

### 🎯 Test Identification

1. Odd length list: simple case
2. Even length list: average case
3. Single item
4. Two items
5. Already sorted
6. Reverse sorted
7. Duplicates
8. All same values
9. Negative numbers
10. Mixed positive/negative
11. Empty list: error

### 🎲 Value Selection

```python
# [1, 3, 5] → 3
# Why? Odd length, middle is clear

# [1, 2, 3, 4] → 2.5
# Why? Even length, average of 2 and 3

# [5] → 5
# Why? Edge case single item

# [3, 1, 4, 1, 5, 9, 2] → 3
# Why? Unsorted, tests sorting

# [-5, 0, 5] → 0
# Why? Tests with negative numbers
```

### ✅ Complete Tests

```python
import pytest
from median import find_median

def test_find_median_odd_length():
    """Test median with odd number of elements."""
    # 3 elements: middle is 3
    assert find_median([1, 3, 5]) == 3

def test_find_median_even_length():
    """Test median with even number of elements."""
    # 4 elements: average of 2 and 3
    assert find_median([1, 2, 3, 4]) == 2.5

def test_find_median_single_element():
    """Test with single element."""
    # Edge case: single element is the median
    assert find_median([42]) == 42

def test_find_median_two_elements():
    """Test with two elements."""
    # Edge case: average of both
    assert find_median([1, 3]) == 2.0

def test_find_median_unsorted():
    """Test that function works with unsorted list."""
    # Input order shouldn't matter
    assert find_median([5, 1, 3]) == 3
    assert find_median([3, 1, 4, 1, 5, 9, 2]) == 3

def test_find_median_reverse_sorted():
    """Test with reverse sorted list."""
    # Should sort first
    assert find_median([5, 4, 3, 2, 1]) == 3

def test_find_median_with_duplicates():
    """Test with duplicate values."""
    # Duplicates shouldn't affect algorithm
    assert find_median([1, 2, 2, 3, 4]) == 2

def test_find_median_all_same():
    """Test when all values are the same."""
    # Edge case: all same means median is that value
    assert find_median([5, 5, 5, 5, 5]) == 5

def test_find_median_negative_numbers():
    """Test with negative numbers."""
    assert find_median([-5, -2, -8]) == -5

def test_find_median_mixed_positive_negative():
    """Test with mix of positive and negative."""
    assert find_median([-5, 0, 5]) == 0

def test_find_median_large_list():
    """Test with larger list."""
    # 7 elements, median should be 4
    assert find_median([1, 2, 3, 4, 5, 6, 7]) == 4

def test_find_median_floats():
    """Test with floating point numbers."""
    result = find_median([1.5, 2.7, 3.2])
    assert result == pytest.approx(2.7)

def test_find_median_empty_list_raises_error():
    """Test that empty list raises ValueError."""
    with pytest.raises(ValueError, match="Cannot find median of empty list"):
        find_median([])

def test_find_median_does_not_modify_original():
    """Test that original list is not modified."""
    # Important: sorting should not affect input
    original = [3, 1, 2]
    result = find_median(original)
    assert original == [3, 1, 2]  # Still unsorted
    assert result == 2
```

**Rationale:**
- 14 tests cover algorithm thoroughly
- Tests both odd and even cases
- Tests sorting behavior
- Tests edge cases (single, two elements, all same)
- Verifies no side effects

---

## Example 7: Parse CSV Line

### 📝 The Function

```python
def parse_csv_line(line):
    """Parse a CSV line into list of fields.
    
    Handles quoted fields with commas inside.
    
    Args:
        line: CSV line string
    
    Returns:
        List of field values
    
    Example:
        parse_csv_line('name,age,"city, state"')
        → ['name', 'age', 'city, state']
    """
    fields = []
    current_field = []
    in_quotes = False
    
    for char in line:
        if char == '"':
            in_quotes = not in_quotes
        elif char == ',' and not in_quotes:
            fields.append(''.join(current_field))
            current_field = []
        else:
            current_field.append(char)
    
    # Add last field
    fields.append(''.join(current_field))
    
    return fields
```

### 🧠 Analysis

**What does this do?**
- Splits CSV line by commas
- Respects quotes: commas inside quotes are kept
- Handles quoted and unquoted fields
- Returns list of parsed field values

**Parse rules to test:**
- Simple fields (no quotes)
- Quoted fields
- Commas inside quotes
- Empty fields
- Multiple quoted fields
- Single field

### 🎯 Test Identification

1. Simple unquoted fields
2. Field with quotes containing comma
3. Multiple quoted fields
4. Empty fields
5. Single field
6. Empty string
7. Field with quotes at start/end
8. No commas

### 🎲 Value Selection

```python
# 'name,age,city' → ['name', 'age', 'city']
# Why? Simple case, no quotes

# '"Smith, John",30,NYC' → ['Smith, John', '30', 'NYC']
# Why? Tests comma inside quotes

# 'a,,c' → ['a', '', 'c']
# Why? Tests empty field

# 'single' → ['single']
# Why? No commas case
```

### ✅ Complete Tests

```python
from csv_parser import parse_csv_line

def test_parse_csv_line_simple():
    """Test parsing simple unquoted fields."""
    # Basic case: no quotes or special chars
    result = parse_csv_line('name,age,city')
    assert result == ['name', 'age', 'city']

def test_parse_csv_line_with_quoted_field():
    """Test field with quotes containing comma."""
    # Comma inside quotes should be kept
    result = parse_csv_line('name,"city, state",age')
    assert result == ['name', 'city, state', 'age']

def test_parse_csv_line_multiple_quoted():
    """Test multiple quoted fields."""
    result = parse_csv_line('"first, name","last, name","city, state"')
    assert result == ['first, name', 'last, name', 'city, state']

def test_parse_csv_line_empty_fields():
    """Test with empty fields."""
    # Consecutive commas mean empty field
    result = parse_csv_line('a,,c')
    assert result == ['a', '', 'c']

def test_parse_csv_line_single_field():
    """Test with single field (no commas)."""
    result = parse_csv_line('onlyfield')
    assert result == ['onlyfield']

def test_parse_csv_line_empty_string():
    """Test with empty string."""
    # Edge case: empty input
    result = parse_csv_line('')
    assert result == ['']

def test_parse_csv_line_two_fields():
    """Test with two fields."""
    result = parse_csv_line('first,second')
    assert result == ['first', 'second']

def test_parse_csv_line_quoted_empty_field():
    """Test quoted empty field."""
    result = parse_csv_line('a,"",c')
    assert result == ['a', '', 'c']

def test_parse_csv_line_numbers():
    """Test with numeric values."""
    result = parse_csv_line('1,2,3')
    assert result == ['1', '2', '3']

def test_parse_csv_line_mixed():
    """Test realistic CSV line with mix of quoted and unquoted."""
    result = parse_csv_line('John,30,"New York, NY",engineer')
    assert result == ['John', '30', 'New York, NY', 'engineer']
```

**Rationale:**
- 10 tests cover parsing logic
- Tests the quote-handling algorithm
- Tests edge cases (empty, single field)
- Tests realistic CSV data

---

## Example 8: Grade Calculator

### 📝 The Function

```python
def calculate_grade(score):
    """Convert numeric score to letter grade.
    
    Grade scale:
    - 90-100: A
    - 80-89: B
    - 70-79: C
    - 60-69: D
    - Below 60: F
    
    Args:
        score: Numeric score (0-100)
    
    Returns:
        Letter grade string
    
    Raises:
        ValueError: If score is out of range
    """
    if score < 0 or score > 100:
        raise ValueError("Score must be between 0 and 100")
    
    if score >= 90:
        return "A"
    elif score >= 80:
        return "B"
    elif score >= 70:
        return "C"
    elif score >= 60:
        return "D"
    else:
        return "F"
```

### 🧠 Analysis

**What does this do?**
- Maps score to letter grade
- Multiple threshold conditions
- Validates score in range 0-100

**Thresholds to test:**
- Each grade range
- Boundary values (90, 80, 70, 60)
- Just above and below boundaries
- Min (0) and max (100)
- Out of range values

### 🎯 Test Identification

1. Each letter grade: A, B, C, D, F
2. Boundaries: 90, 80, 70, 60, 0, 100
3. Just above boundaries: 91, 81, 71, 61
4. Just below boundaries: 89, 79, 69, 59
5. Invalid: negative, > 100

### 🎲 Value Selection

Using equivalence partitioning and boundary testing:

```
A range: [90-100]
  Test: 90 (boundary), 95 (middle), 100 (boundary)

B range: [80-89]
  Test: 80 (boundary), 85 (middle), 89 (boundary)

C range: [70-79]
  Test: 70 (boundary), 75 (middle), 79 (boundary)

D range: [60-69]
  Test: 60 (boundary), 65 (middle), 69 (boundary)

F range: [0-59]
  Test: 0 (boundary), 30 (middle), 59 (boundary)

Invalid: -1, 101
```

### ✅ Complete Tests

```python
import pytest
from grades import calculate_grade

# Test each grade letter
def test_calculate_grade_a():
    """Test score in A range."""
    assert calculate_grade(95) == "A"

def test_calculate_grade_b():
    """Test score in B range."""
    assert calculate_grade(85) == "B"

def test_calculate_grade_c():
    """Test score in C range."""
    assert calculate_grade(75) == "C"

def test_calculate_grade_d():
    """Test score in D range."""
    assert calculate_grade(65) == "D"

def test_calculate_grade_f():
    """Test score in F range."""
    assert calculate_grade(30) == "F"

# Test boundaries
def test_calculate_grade_exactly_90():
    """Test boundary: 90 should be A."""
    assert calculate_grade(90) == "A"

def test_calculate_grade_exactly_80():
    """Test boundary: 80 should be B."""
    assert calculate_grade(80) == "B"

def test_calculate_grade_exactly_70():
    """Test boundary: 70 should be C."""
    assert calculate_grade(70) == "C"

def test_calculate_grade_exactly_60():
    """Test boundary: 60 should be D."""
    assert calculate_grade(60) == "D"

def test_calculate_grade_exactly_0():
    """Test boundary: 0 should be F."""
    assert calculate_grade(0) == "F"

def test_calculate_grade_exactly_100():
    """Test boundary: 100 should be A."""
    assert calculate_grade(100) == "A"

# Test just above and below boundaries
def test_calculate_grade_89():
    """Test 89 is B (just below A threshold)."""
    assert calculate_grade(89) == "B"

def test_calculate_grade_91():
    """Test 91 is A (just above threshold)."""
    assert calculate_grade(91) == "A"

def test_calculate_grade_79():
    """Test 79 is C (just below B threshold)."""
    assert calculate_grade(79) == "C"

def test_calculate_grade_81():
    """Test 81 is B (just above threshold)."""
    assert calculate_grade(81) == "B"

def test_calculate_grade_59():
    """Test 59 is F (just below D threshold)."""
    assert calculate_grade(59) == "F"

def test_calculate_grade_61():
    """Test 61 is D (just above threshold)."""
    assert calculate_grade(61) == "D"

# Test invalid inputs
def test_calculate_grade_negative_raises_error():
    """Test negative score raises ValueError."""
    with pytest.raises(ValueError, match="must be between 0 and 100"):
        calculate_grade(-1)

def test_calculate_grade_over_100_raises_error():
    """Test score > 100 raises ValueError."""
    with pytest.raises(ValueError, match="must be between 0 and 100"):
        calculate_grade(101)
```

**Rationale:**
- 19 tests thoroughly cover all conditions
- Every threshold tested at boundary
- Just above and below each boundary tested
- This is critical for grading (no mistakes allowed!)
- All error cases tested

---

## Example 9: BankAccount Class

### 📝 The Function

```python
class BankAccount:
    """Simple bank account."""
    
    def __init__(self, initial_balance=0):
        """Initialize account with balance."""
        if initial_balance < 0:
            raise ValueError("Initial balance cannot be negative")
        self.balance = initial_balance
        self.transaction_count = 0
    
    def deposit(self, amount):
        """Deposit money."""
        if amount <= 0:
            raise ValueError("Deposit amount must be positive")
        self.balance += amount
        self.transaction_count += 1
    
    def withdraw(self, amount):
        """Withdraw money."""
        if amount <= 0:
            raise ValueError("Withdrawal amount must be positive")
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
        self.transaction_count += 1
    
    def get_balance(self):
        """Get current balance."""
        return self.balance
```

### 🧠 Analysis

**What does this do?**
- Manages account balance
- Tracks transaction count
- Validates all operations
- Multiple methods with state changes

**State to test:**
- Initial state
- Balance after deposits
- Balance after withdrawals
- Transaction count
- Error states

### 🎯 Test Identification

**For __init__:**
1. Default balance (0)
2. Positive initial balance
3. Negative initial balance: error

**For deposit:**
4. Single deposit
5. Multiple deposits
6. Zero deposit: error
7. Negative deposit: error

**For withdraw:**
8. Single withdrawal
9. Multiple withdrawals
10. Insufficient funds: error
11. Zero withdrawal: error
12. Negative withdrawal: error

**For transaction_count:**
13. Increments on deposit
14. Increments on withdraw
15. Not incremented on errors

### ✅ Complete Tests

```python
import pytest
from bank import BankAccount

# Test initialization
def test_bank_account_default_balance():
    """Test account starts with zero balance by default."""
    account = BankAccount()
    assert account.get_balance() == 0
    assert account.transaction_count == 0

def test_bank_account_with_initial_balance():
    """Test account with initial balance."""
    account = BankAccount(100)
    assert account.get_balance() == 100
    assert account.transaction_count == 0

def test_bank_account_negative_initial_balance_raises_error():
    """Test that negative initial balance raises error."""
    with pytest.raises(ValueError, match="cannot be negative"):
        BankAccount(-50)

# Test deposit
def test_deposit_increases_balance():
    """Test deposit increases balance."""
    account = BankAccount(100)
    account.deposit(50)
    assert account.get_balance() == 150

def test_deposit_increments_transaction_count():
    """Test deposit increments transaction count."""
    account = BankAccount(100)
    account.deposit(50)
    assert account.transaction_count == 1

def test_multiple_deposits():
    """Test multiple deposits."""
    account = BankAccount(100)
    account.deposit(50)
    account.deposit(30)
    assert account.get_balance() == 180
    assert account.transaction_count == 2

def test_deposit_zero_raises_error():
    """Test that depositing zero raises error."""
    account = BankAccount(100)
    with pytest.raises(ValueError, match="must be positive"):
        account.deposit(0)

def test_deposit_negative_raises_error():
    """Test that negative deposit raises error."""
    account = BankAccount(100)
    with pytest.raises(ValueError, match="must be positive"):
        account.deposit(-50)

# Test withdraw
def test_withdraw_decreases_balance():
    """Test withdrawal decreases balance."""
    account = BankAccount(100)
    account.withdraw(30)
    assert account.get_balance() == 70

def test_withdraw_increments_transaction_count():
    """Test withdrawal increments transaction count."""
    account = BankAccount(100)
    account.withdraw(30)
    assert account.transaction_count == 1

def test_multiple_withdrawals():
    """Test multiple withdrawals."""
    account = BankAccount(100)
    account.withdraw(30)
    account.withdraw(20)
    assert account.get_balance() == 50
    assert account.transaction_count == 2

def test_withdraw_entire_balance():
    """Test withdrawing entire balance."""
    account = BankAccount(100)
    account.withdraw(100)
    assert account.get_balance() == 0

def test_withdraw_insufficient_funds_raises_error():
    """Test insufficient funds raises error."""
    account = BankAccount(50)
    with pytest.raises(ValueError, match="Insufficient funds"):
        account.withdraw(100)

def test_withdraw_zero_raises_error():
    """Test that withdrawing zero raises error."""
    account = BankAccount(100)
    with pytest.raises(ValueError, match="must be positive"):
        account.withdraw(0)

def test_withdraw_negative_raises_error():
    """Test that negative withdrawal raises error."""
    account = BankAccount(100)
    with pytest.raises(ValueError, match="must be positive"):
        account.withdraw(-30)

# Test transaction count
def test_transaction_count_includes_both_operations():
    """Test transaction count tracks both deposits and withdrawals."""
    account = BankAccount(100)
    account.deposit(50)
    account.withdraw(30)
    account.deposit(20)
    assert account.transaction_count == 3

def test_failed_transaction_does_not_increment_count():
    """Test that failed operations don't increment count."""
    account = BankAccount(50)
    try:
        account.withdraw(100)
    except ValueError:
        pass
    assert account.transaction_count == 0  # No successful transaction

def test_failed_transaction_does_not_change_balance():
    """Test that failed operations don't change balance."""
    account = BankAccount(50)
    try:
        account.withdraw(100)
    except ValueError:
        pass
    assert account.get_balance() == 50  # Balance unchanged

# Integration test
def test_realistic_account_usage():
    """Test realistic sequence of operations."""
    account = BankAccount(1000)
    account.deposit(500)      # 1500
    account.withdraw(200)     # 1300
    account.withdraw(300)     # 1000
    account.deposit(100)      # 1100
    assert account.get_balance() == 1100
    assert account.transaction_count == 4
```

**Rationale:**
- 19 tests cover class thoroughly
- Each method tested independently
- State changes verified
- Error conditions tested
- Transaction count verified
- Integration test for realistic usage

---

## Example 10: Merge Sorted Lists

### 📝 The Function

```python
def merge_sorted_lists(list1, list2):
    """Merge two sorted lists into one sorted list.
    
    Args:
        list1: First sorted list
        list2: Second sorted list
    
    Returns:
        New sorted list containing all elements from both lists
    
    Example:
        merge_sorted_lists([1, 3, 5], [2, 4, 6]) 
        → [1, 2, 3, 4, 5, 6]
    """
    result = []
    i, j = 0, 0
    
    # Merge while both lists have elements
    while i < len(list1) and j < len(list2):
        if list1[i] <= list2[j]:
            result.append(list1[i])
            i += 1
        else:
            result.append(list2[j])
            j += 1
    
    # Add remaining elements
    result.extend(list1[i:])
    result.extend(list2[j:])
    
    return result
```

### 🧠 Analysis

**What does this do?**
- Merges two sorted lists
- Maintains sort order
- Uses two-pointer algorithm
- Handles lists of different lengths

**Algorithm properties:**
- Result is sorted
- Contains all elements from both lists
- No elements duplicated (unless in input)
- Efficient merge

### 🎯 Test Identification

1. Equal length lists
2. Different length lists
3. One empty list
4. Both empty
5. Single element lists
6. One list much longer
7. Overlapping ranges
8. Non-overlapping ranges
9. Duplicate values
10. Verifies original lists unchanged

### 🎲 Value Selection

```python
# [1, 3, 5], [2, 4, 6] → [1, 2, 3, 4, 5, 6]
# Why? Equal length, interleaved values

# [1, 2], [3, 4, 5, 6] → [1, 2, 3, 4, 5, 6]
# Why? Different lengths

# [], [1, 2, 3] → [1, 2, 3]
# Why? One empty

# [1, 5, 9], [2, 3, 4] → [1, 2, 3, 4, 5, 9]
# Why? Tests algorithm when one list "catches up"
```

### ✅ Complete Tests

```python
from merge import merge_sorted_lists

def test_merge_sorted_lists_equal_length():
    """Test merging two equal-length lists."""
    result = merge_sorted_lists([1, 3, 5], [2, 4, 6])
    assert result == [1, 2, 3, 4, 5, 6]

def test_merge_sorted_lists_different_lengths():
    """Test merging lists of different lengths."""
    result = merge_sorted_lists([1, 2], [3, 4, 5, 6])
    assert result == [1, 2, 3, 4, 5, 6]

def test_merge_sorted_lists_first_empty():
    """Test when first list is empty."""
    result = merge_sorted_lists([], [1, 2, 3])
    assert result == [1, 2, 3]

def test_merge_sorted_lists_second_empty():
    """Test when second list is empty."""
    result = merge_sorted_lists([1, 2, 3], [])
    assert result == [1, 2, 3]

def test_merge_sorted_lists_both_empty():
    """Test when both lists are empty."""
    result = merge_sorted_lists([], [])
    assert result == []

def test_merge_sorted_lists_single_elements():
    """Test with single-element lists."""
    result = merge_sorted_lists([1], [2])
    assert result == [1, 2]

def test_merge_sorted_lists_one_much_longer():
    """Test when one list is much longer."""
    result = merge_sorted_lists([1], [2, 3, 4, 5, 6])
    assert result == [1, 2, 3, 4, 5, 6]

def test_merge_sorted_lists_non_overlapping():
    """Test lists with non-overlapping ranges."""
    # All of list1 before list2
    result = merge_sorted_lists([1, 2, 3], [4, 5, 6])
    assert result == [1, 2, 3, 4, 5, 6]

def test_merge_sorted_lists_overlapping():
    """Test lists with overlapping ranges."""
    result = merge_sorted_lists([1, 5, 9], [2, 3, 4])
    assert result == [1, 2, 3, 4, 5, 9]

def test_merge_sorted_lists_with_duplicates():
    """Test with duplicate values."""
    result = merge_sorted_lists([1, 3, 5], [1, 3, 5])
    assert result == [1, 1, 3, 3, 5, 5]

def test_merge_sorted_lists_all_same_value():
    """Test when all values are the same."""
    result = merge_sorted_lists([5, 5], [5, 5, 5])
    assert result == [5, 5, 5, 5, 5]

def test_merge_sorted_lists_negative_numbers():
    """Test with negative numbers."""
    result = merge_sorted_lists([-5, -3, -1], [-4, -2, 0])
    assert result == [-5, -4, -3, -2, -1, 0]

def test_merge_sorted_lists_does_not_modify_inputs():
    """Test that original lists are not modified."""
    list1 = [1, 3, 5]
    list2 = [2, 4, 6]
    result = merge_sorted_lists(list1, list2)
    # Originals unchanged
    assert list1 == [1, 3, 5]
    assert list2 == [2, 4, 6]
    assert result == [1, 2, 3, 4, 5, 6]

def test_merge_sorted_lists_result_is_sorted():
    """Test that result is properly sorted."""
    result = merge_sorted_lists([1, 5, 9], [2, 6, 7])
    # Verify sorting by checking each adjacent pair
    for i in range(len(result) - 1):
        assert result[i] <= result[i + 1]
```

**Rationale:**
- 14 tests cover merge algorithm
- Tests various length combinations
- Tests edge cases (empty, single elements)
- Tests algorithm correctness (overlapping, non-overlapping)
- Verifies no side effects
- Tests sortedness property

---

## Summary

You've now seen **10 complete examples** showing the thinking process from function to tests.

**Key patterns you've learned:**

1. **Always start with analysis** - understand what the function does
2. **Identify all scenarios** - normal, edge, error cases
3. **Choose specific values** - with clear rationale
4. **Test boundaries** - thresholds, limits, transitions
5. **Verify behavior, not implementation** - test what it does, not how
6. **Document your thinking** - clear test names and comments

**Common test patterns seen:**

- ✅ Normal case first (typical usage)
- ✅ Edge cases (empty, zero, single item, boundaries)
- ✅ Error cases with pytest.raises
- ✅ Verify state changes (before/after)
- ✅ Check side effects (or lack thereof)
- ✅ Progressive complexity (simple → complex)

**Practice tips:**

1. Work through each example again, covering the tests and trying to write them yourself
2. Find similar functions in your code and apply these patterns
3. Compare your tests with the examples - are you missing cases?
4. Notice how the thinking process is similar across different function types

**Next steps:**

- Apply these patterns to your own code
- Review [How to Write Tests](how-to-write-tests.md) for the complete workflow
- Study [Testing Fundamentals](testing-fundamentals.md) for concepts
- Practice with real functions from your projects

Remember: **The thinking process is more important than memorizing patterns.** Understand WHY these tests were chosen, and you'll be able to generate tests for any function.

Happy testing! 🎯
