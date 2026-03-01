# Refactoring

## What is Refactoring?

### Definition

Refactoring is restructuring code without changing its external behavior:

```python
# Refactoring improves:
# - Code readability
# - Maintainability
# - Testability
# - Performance (sometimes)

# Refactoring does NOT:
# - Add new features
# - Fix bugs (intentionally)
# - Change functionality
```

### Why Refactor?

```python
# Reasons to refactor:
# 1. Code is hard to understand
# 2. Duplicate code exists
# 3. Changes are difficult to make
# 4. Testing is painful
# 5. Performance is poor

# Benefits:
# - Easier maintenance
# - Faster development
# - Fewer bugs
# - Better design
```

## Code Smells

### Long Method

```python
# Bad: Method does too much
def process_order(order):
    # Validate
    if not order.items:
        raise ValueError("Empty order")
    if not order.customer:
        raise ValueError("No customer")

    # Calculate total
    subtotal = sum(item.price * item.quantity for item in order.items)
    tax = subtotal * 0.08
    shipping = 10 if subtotal < 100 else 0
    total = subtotal + tax + shipping

    # Apply discount
    if order.customer.is_premium:
        total *= 0.9

    # Process payment
    if total > 0:
        charge_credit_card(order.customer.card, total)

    # Update inventory
    for item in order.items:
        update_stock(item.product_id, -item.quantity)

    # Send notifications
    send_email(order.customer.email, "Order confirmed")
    send_sms(order.customer.phone, "Order confirmed")

    return total

# Good: Extract methods
def process_order(order):
    validate_order(order)
    total = calculate_total(order)
    process_payment(order.customer, total)
    update_inventory(order.items)
    send_notifications(order.customer)
    return total
```

### Large Class

```python
# Bad: Class does too many things
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    # User management
    def save_to_database(self):
        pass

    def load_from_database(self, user_id):
        pass

    # Authentication
    def check_password(self, password):
        pass

    def reset_password(self):
        pass

    # Notifications
    def send_welcome_email(self):
        pass

    def send_password_reset_email(self):
        pass

    # Reporting
    def generate_activity_report(self):
        pass

# Good: Split into focused classes
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserRepository:
    def save(self, user):
        pass

    def load(self, user_id):
        pass

class AuthenticationService:
    def check_password(self, user, password):
        pass

    def reset_password(self, user):
        pass

class NotificationService:
    def send_welcome_email(self, user):
        pass

    def send_password_reset_email(self, user):
        pass

class UserReportService:
    def generate_activity_report(self, user):
        pass
```

### Duplicate Code

```python
# Bad: Repeated logic
def calculate_employee_salary(employee):
    base = employee.base_salary
    bonus = base * 0.1
    tax = (base + bonus) * 0.2
    return base + bonus - tax

def calculate_contractor_payment(contractor):
    base = contractor.hourly_rate * contractor.hours
    bonus = base * 0.1
    tax = (base + bonus) * 0.2
    return base + bonus - tax

# Good: Extract common logic
def calculate_payment(base_amount):
    bonus = base_amount * 0.1
    tax = (base_amount + bonus) * 0.2
    return base_amount + bonus - tax

def calculate_employee_salary(employee):
    return calculate_payment(employee.base_salary)

def calculate_contractor_payment(contractor):
    base = contractor.hourly_rate * contractor.hours
    return calculate_payment(base)
```

### Long Parameter List

```python
# Bad: Too many parameters
def create_user(name, email, age, address, city, state, zip_code, country,
                phone, is_active, is_premium):
    pass

# Good: Use object
class UserData:
    def __init__(self, name, email, age):
        self.name = name
        self.email = email
        self.age = age

class Address:
    def __init__(self, street, city, state, zip_code, country):
        self.street = street
        self.city = city
        self.state = state
        self.zip_code = zip_code
        self.country = country

def create_user(user_data: UserData, address: Address, phone: str,
                is_active: bool = True, is_premium: bool = False):
    pass
```

### Primitive Obsession

```python
# Bad: Using primitives for domain concepts
def calculate_discount(price, user_type, coupon_code):
    if user_type == "premium":
        discount = 0.2
    elif user_type == "regular":
        discount = 0.1
    else:
        discount = 0

    if coupon_code == "SAVE10":
        discount += 0.1

    return price * (1 - discount)

# Good: Use domain objects
class UserType:
    def __init__(self, name, discount):
        self.name = name
        self.discount = discount

class Coupon:
    def __init__(self, code, discount):
        self.code = code
        self.discount = discount

class DiscountCalculator:
    def calculate(self, price, user_type: UserType, coupon: Coupon = None):
        discount = user_type.discount
        if coupon:
            discount += coupon.discount
        return price * (1 - discount)
```

## Refactoring Techniques

### Extract Method

```python
# Before
def print_invoice(order):
    print("***********************")
    print(f"Order #{order.id}")
    print("***********************")

    total = 0
    for item in order.items:
        price = item.quantity * item.price
        total += price
        print(f"{item.name}: ${price}")

    print(f"Total: ${total}")

# After
def print_invoice(order):
    print_header(order)
    total = print_items(order.items)
    print_total(total)

def print_header(order):
    print("***********************")
    print(f"Order #{order.id}")
    print("***********************")

def print_items(items):
    total = 0
    for item in items:
        price = item.quantity * item.price
        total += price
        print(f"{item.name}: ${price}")
    return total

def print_total(total):
    print(f"Total: ${total}")
```

### Rename Variable/Method

```python
# Before: Unclear names
def calc(d):
    return d * 0.08

# After: Clear names
def calculate_sales_tax(amount):
    TAX_RATE = 0.08
    return amount * TAX_RATE
```

### Extract Variable

```python
# Before: Complex expression
if (user.age >= 18 and user.has_valid_id and
    not user.is_banned and user.country in ['US', 'CA']):
    allow_access()

# After: Named intermediate variables
is_adult = user.age >= 18
has_credentials = user.has_valid_id and not user.is_banned
is_in_supported_region = user.country in ['US', 'CA']

if is_adult and has_credentials and is_in_supported_region:
    allow_access()
```

### Inline Method

```python
# Before: Unnecessary indirection
def get_rating(driver):
    return more_than_five_late_deliveries(driver) ? 2 : 1

def more_than_five_late_deliveries(driver):
    return driver.late_deliveries > 5

# After: Inline simple method
def get_rating(driver):
    return 2 if driver.late_deliveries > 5 else 1
```

### Replace Conditional with Polymorphism

```python
# Before: Type checking
class Bird:
    def __init__(self, bird_type):
        self.type = bird_type

    def fly(self):
        if self.type == 'european':
            return "European bird flying"
        elif self.type == 'african':
            return "African bird flying"
        elif self.type == 'norwegian_blue':
            return "Cannot fly (nailed to perch)"

# After: Polymorphism
class Bird(ABC):
    @abstractmethod
    def fly(self):
        pass

class EuropeanBird(Bird):
    def fly(self):
        return "European bird flying"

class AfricanBird(Bird):
    def fly(self):
        return "African bird flying"

class NorwegianBlueBird(Bird):
    def fly(self):
        return "Cannot fly (nailed to perch)"
```

## When to Refactor

### Rule of Three

```python
# First time: Write it
def calculate_price(item):
    return item.price * 1.08

# Second time: Duplicate it (acceptable)
def calculate_order_total(order):
    return sum(item.price * 1.08 for item in order.items)

# Third time: Refactor it
TAX_RATE = 1.08

def apply_tax(amount):
    return amount * TAX_RATE

def calculate_price(item):
    return apply_tax(item.price)

def calculate_order_total(order):
    subtotal = sum(item.price for item in order.items)
    return apply_tax(subtotal)
```

### Before Adding Features

```python
# Scenario: Adding a new payment method
# Before refactoring:
def process_payment(payment_type, amount):
    if payment_type == 'credit_card':
        # Complex credit card logic
        pass
    elif payment_type == 'paypal':
        # Complex PayPal logic
        pass

# Refactor first
class PaymentMethod(ABC):
    @abstractmethod
    def process(self, amount):
        pass

class CreditCardPayment(PaymentMethod):
    def process(self, amount):
        # Credit card logic
        pass

class PayPalPayment(PaymentMethod):
    def process(self, amount):
        # PayPal logic
        pass

# Now adding Bitcoin is easy
class BitcoinPayment(PaymentMethod):
    def process(self, amount):
        # Bitcoin logic
        pass
```

### During Code Review

```python
# Code review feedback triggers refactoring
# Original code:
def f(x, y):
    return (x + y) * 2 if x > 0 else 0

# After review feedback:
def calculate_double_sum(positive_value, additional_value):
    """Calculate double the sum if positive_value is positive."""
    if positive_value <= 0:
        return 0

    total = positive_value + additional_value
    return total * 2
```

## Refactoring Workflow

### Safe Refactoring Steps

```python
# 1. Ensure tests exist and pass
# Run: pytest

# 2. Make small changes
# Bad: Refactor entire class at once
# Good: Extract one method, test, commit

# 3. Test after each change
# Run tests frequently

# 4. Commit frequently
# git commit -m "Extract calculate_total method"

# 5. Use IDE refactoring tools
# - Rename (Shift+F6 in PyCharm)
# - Extract method (Ctrl+Alt+M)
# - Inline (Ctrl+Alt+N)
```

### Refactoring with Tests

```python
# Original code
def calculate_total(items):
    total = 0
    for item in items:
        total += item.price * item.quantity
    return total

# Test
def test_calculate_total():
    items = [
        Item(price=10, quantity=2),
        Item(price=5, quantity=3)
    ]
    assert calculate_total(items) == 35

# Refactor (tests still pass)
def calculate_total(items):
    return sum(item.price * item.quantity for item in items)

# Test still passes
def test_calculate_total():
    items = [
        Item(price=10, quantity=2),
        Item(price=5, quantity=3)
    ]
    assert calculate_total(items) == 35
```

## Common Refactorings

### Replace Magic Numbers with Constants

```python
# Before
def calculate_shipping(weight):
    if weight < 5:
        return weight * 2.5
    return weight * 2.0

# After
LIGHTWEIGHT_THRESHOLD = 5
LIGHTWEIGHT_RATE = 2.5
STANDARD_RATE = 2.0

def calculate_shipping(weight):
    if weight < LIGHTWEIGHT_THRESHOLD:
        return weight * LIGHTWEIGHT_RATE
    return weight * STANDARD_RATE
```

### Simplify Conditionals

```python
# Before
def get_discount(user):
    if user.is_premium:
        if user.years_member > 5:
            return 0.25
        else:
            return 0.15
    else:
        if user.years_member > 5:
            return 0.10
        else:
            return 0.05

# After
def get_discount(user):
    base_discount = 0.15 if user.is_premium else 0.05
    loyalty_bonus = 0.10 if user.years_member > 5 else 0
    return base_discount + loyalty_bonus
```

### Decompose Conditional

```python
# Before
if date.before(SUMMER_START) or date.after(SUMMER_END):
    charge = quantity * winter_rate + winter_service_charge
else:
    charge = quantity * summer_rate

# After
def is_winter(date):
    return date.before(SUMMER_START) or date.after(SUMMER_END)

def calculate_winter_charge(quantity):
    return quantity * winter_rate + winter_service_charge

def calculate_summer_charge(quantity):
    return quantity * summer_rate

if is_winter(date):
    charge = calculate_winter_charge(quantity)
else:
    charge = calculate_summer_charge(quantity)
```

### Replace Nested Conditionals with Guard Clauses

```python
# Before
def calculate_pay(employee):
    if employee.is_active:
        if employee.hours_worked > 0:
            if employee.hourly_rate > 0:
                return employee.hours_worked * employee.hourly_rate
    return 0

# After
def calculate_pay(employee):
    if not employee.is_active:
        return 0

    if employee.hours_worked <= 0:
        return 0

    if employee.hourly_rate <= 0:
        return 0

    return employee.hours_worked * employee.hourly_rate
```

## Summary

Refactoring principles and techniques:

**What is refactoring:**

- Improving code structure
- Maintaining functionality
- Continuous improvement

**Code smells:**

- Long methods
- Large classes
- Duplicate code
- Long parameter lists
- Primitive obsession

**Techniques:**

- Extract method
- Rename variables/methods
- Extract variable
- Inline method
- Replace conditional with polymorphism

**When to refactor:**

- Rule of three
- Before adding features
- During code review
- When code smells appear

**Workflow:**

- Ensure tests exist
- Make small changes
- Test frequently
- Commit often

Refactoring is ongoing maintenance, not a one-time activity.

## Next Steps

- Apply [SOLID Principles](solid-principles.md)
- Use [Design Patterns](design-patterns.md)
- Follow [Code Style Standards](code-style-and-standards.md)
