# Writing Clean Code

## Meaningful Names

### Variable Names

```python
# Bad: Cryptic abbreviations
d = 86400  # What is d?
tm = time.time()  # tm?
usr_lst = []  # usr_lst?

# Good: Clear and descriptive
seconds_per_day = 86400
current_timestamp = time.time()
active_users = []

# Bad: Non-descriptive
data = fetch_data()
temp = calculate(data)
result = process(temp)

# Good: Intent-revealing
user_orders = fetch_user_orders()
order_total = calculate_order_total(user_orders)
invoice = generate_invoice(order_total)
```

### Function Names

```python
# Bad: Vague names
def handle_data(data):
    pass

def do_stuff(x, y):
    pass

# Good: Verb + noun, describes what it does
def validate_email_address(email):
    pass

def calculate_monthly_payment(principal, rate, months):
    pass

# Boolean functions should sound like yes/no questions
def is_valid_email(email):
    return '@' in email

def has_permission(user, resource):
    return user.role == 'admin'

def can_delete_post(user, post):
    return user.id == post.author_id or user.is_admin
```

### Class Names

```python
# Bad: Vague or too generic
class Manager:
    pass

class Data:
    pass

class Helper:
    pass

# Good: Specific and descriptive
class UserAccountManager:
    pass

class OrderData:
    pass

class EmailValidationHelper:
    pass

# Even better: More specific
class DatabaseConnectionPool:
    pass

class ShoppingCartItem:
    pass

class PDFReportGenerator:
    pass
```

### Context Matters

```python
# Bad: Redundant context
class User:
    def __init__(self, user_name, user_email, user_age):
        self.user_name = user_name
        self.user_email = user_email
        self.user_age = user_age

# Good: Context implied by class
class User:
    def __init__(self, name, email, age):
        self.name = name
        self.email = email
        self.age = age

# Usage is clear
user = User(name="Alice", email="alice@example.com", age=30)
print(user.name)  # Not user.user_name
```

## Functions Should Do One Thing

### Single Responsibility

```python
# Bad: Function does too many things
def process_order(order):
    # Validate
    if not order.items:
        raise ValueError("Order has no items")

    # Calculate total
    total = sum(item.price * item.quantity for item in order.items)

    # Apply discount
    if order.user.is_premium:
        total *= 0.9

    # Process payment
    payment_result = payment_gateway.charge(order.user.card, total)

    # Update inventory
    for item in order.items:
        inventory.decrease(item.product_id, item.quantity)

    # Send email
    email.send(order.user.email, "Order confirmed")

    return payment_result

# Good: Separate concerns
def process_order(order):
    validate_order(order)
    total = calculate_order_total(order)
    payment_result = charge_payment(order.user, total)
    update_inventory(order.items)
    send_confirmation_email(order.user)
    return payment_result

def validate_order(order):
    if not order.items:
        raise ValueError("Order has no items")

def calculate_order_total(order):
    total = sum(item.price * item.quantity for item in order.items)
    return apply_discount(order.user, total)

def apply_discount(user, total):
    if user.is_premium:
        return total * 0.9
    return total
```

### Small Functions

```python
# Bad: Long function
def generate_report(users):
    report = []
    for user in users:
        if user.is_active:
            orders = user.get_orders()
            total_spent = 0
            for order in orders:
                if order.status == 'completed':
                    for item in order.items:
                        total_spent += item.price * item.quantity
            if total_spent > 1000:
                report.append({
                    'user': user.name,
                    'email': user.email,
                    'total': total_spent,
                    'category': 'premium'
                })
    return report

# Good: Small, focused functions
def generate_report(users):
    active_users = filter_active_users(users)
    premium_users = filter_premium_users(active_users)
    return format_user_report(premium_users)

def filter_active_users(users):
    return [user for user in users if user.is_active]

def filter_premium_users(users):
    return [user for user in users if calculate_total_spent(user) > 1000]

def calculate_total_spent(user):
    completed_orders = get_completed_orders(user)
    return sum(calculate_order_total(order) for order in completed_orders)

def get_completed_orders(user):
    return [order for order in user.get_orders() if order.status == 'completed']

def calculate_order_total(order):
    return sum(item.price * item.quantity for item in order.items)

def format_user_report(users):
    return [format_user_entry(user) for user in users]

def format_user_entry(user):
    return {
        'user': user.name,
        'email': user.email,
        'total': calculate_total_spent(user),
        'category': 'premium'
    }
```

## KISS Principle

### Keep It Simple, Stupid

```python
# Bad: Over-engineered
class CalculatorFactory:
    @staticmethod
    def create_calculator(calc_type):
        if calc_type == "basic":
            return BasicCalculator()

def add_numbers(a, b):
    factory = CalculatorFactory()
    calc = factory.create_calculator("basic")
    return calc.add(a, b)

# Good: Simple and direct
def add_numbers(a, b):
    return a + b

# Choose simplicity: solve the problem at hand, not imaginary future problems
```

## DRY Principle

### Don't Repeat Yourself

```python
# Bad: Repeated code
def send_welcome_email(user):
    subject = "Welcome!"
    message = f"Hello {user.name}, welcome to our platform!"
    email_client.send(user.email, subject, message)
    log.info(f"Sent welcome email to {user.email}")

def send_password_reset_email(user):
    subject = "Password Reset"
    message = f"Hello {user.name}, click here to reset your password."
    email_client.send(user.email, subject, message)
    log.info(f"Sent password reset email to {user.email}")

def send_order_confirmation_email(user, order):
    subject = "Order Confirmed"
    message = f"Hello {user.name}, your order #{order.id} is confirmed."
    email_client.send(user.email, subject, message)
    log.info(f"Sent order confirmation email to {user.email}")

# Good: Extract common functionality
def send_email(user, subject, message):
    email_client.send(user.email, subject, message)
    log.info(f"Sent '{subject}' email to {user.email}")

def send_welcome_email(user):
    send_email(
        user,
        subject="Welcome!",
        message=f"Hello {user.name}, welcome to our platform!"
    )

def send_password_reset_email(user):
    send_email(
        user,
        subject="Password Reset",
        message=f"Hello {user.name}, click here to reset your password."
    )

def send_order_confirmation_email(user, order):
    send_email(
        user,
        subject="Order Confirmed",
        message=f"Hello {user.name}, your order #{order.id} is confirmed."
    )
```

### Extract Common Patterns

```python
# Bad: Repeated validation logic
def create_user(username, email, age):
    if not username or len(username) < 3:
        raise ValueError("Invalid username")
    if not email or '@' not in email:
        raise ValueError("Invalid email")
    if not age or age < 18:
        raise ValueError("Invalid age")
    return User(username, email, age)

def update_user(user, username, email, age):
    if not username or len(username) < 3:
        raise ValueError("Invalid username")
    if not email or '@' not in email:
        raise ValueError("Invalid email")
    if not age or age < 18:
        raise ValueError("Invalid age")
    user.username = username
    user.email = email
    user.age = age

# Good: Extract validation
def validate_username(username):
    if not username or len(username) < 3:
        raise ValueError("Invalid username")

def validate_email(email):
    if not email or '@' not in email:
        raise ValueError("Invalid email")

def validate_age(age):
    if not age or age < 18:
        raise ValueError("Invalid age")

def validate_user_data(username, email, age):
    validate_username(username)
    validate_email(email)
    validate_age(age)

def create_user(username, email, age):
    validate_user_data(username, email, age)
    return User(username, email, age)

def update_user(user, username, email, age):
    validate_user_data(username, email, age)
    user.username = username
    user.email = email
    user.age = age
```

## Reducing Cognitive Complexity

### Avoid Deep Nesting

```python
# Bad: Deep nesting
def process_data(data):
    if data:
        if data.is_valid():
            if data.has_items():
                for item in data.items:
                    if item.is_active:
                        if item.price > 0:
                            process_item(item)

# Good: Early returns and flat structure
def process_data(data):
    if not data:
        return

    if not data.is_valid():
        return

    if not data.has_items():
        return

    for item in data.items:
        if should_process_item(item):
            process_item(item)

def should_process_item(item):
    return item.is_active and item.price > 0
```

### Use Guard Clauses

```python
# Bad: Nested conditions
def calculate_discount(user, order):
    if user:
        if user.is_active:
            if order:
                if order.total > 100:
                    if user.is_premium:
                        return order.total * 0.2
                    else:
                        return order.total * 0.1
    return 0

# Good: Guard clauses
def calculate_discount(user, order):
    if not user or not user.is_active:
        return 0

    if not order or order.total <= 100:
        return 0

    discount_rate = 0.2 if user.is_premium else 0.1
    return order.total * discount_rate
```

### Extract Complex Conditions

```python
# Bad: Complex inline conditions
if user.age >= 18 and user.has_valid_id and not user.is_banned and (user.country == 'US' or user.country == 'CA'):
    allow_access()

# Good: Extract to named function
def can_access_service(user):
    is_adult = user.age >= 18
    has_valid_credentials = user.has_valid_id and not user.is_banned
    is_in_supported_region = user.country in ['US', 'CA']

    return is_adult and has_valid_credentials and is_in_supported_region

if can_access_service(user):
    allow_access()
```

### Simplify Boolean Logic

```python
# Bad: Redundant boolean logic
def is_eligible(user):
    if user.age >= 18:
        return True
    else:
        return False

# Good: Direct return
def is_eligible(user):
    return user.age >= 18

# Bad: Unnecessary negation
if not is_invalid(data):
    process(data)

# Good: Positive naming
if is_valid(data):
    process(data)
```

## Self-Documenting Code

### Choose Expressive Names Over Comments

```python
# Bad: Need comments to explain
# Check if user can make purchase
if u.b > p and u.l > 0 and u.s == 'a':
    process()

# Good: Code explains itself
if user_can_make_purchase(user, price):
    process_purchase()

def user_can_make_purchase(user, price):
    has_sufficient_balance = user.balance > price
    has_credit_limit = user.credit_limit > 0
    is_active_account = user.status == 'active'

    return has_sufficient_balance and has_credit_limit and is_active_account
```

### Use Constants for Magic Numbers

```python
# Bad: Magic numbers
def calculate_shipping(weight):
    if weight < 5:
        return weight * 2.5
    elif weight < 20:
        return weight * 2.0
    else:
        return weight * 1.5

# Good: Named constants
LIGHTWEIGHT_THRESHOLD = 5
MEDIUM_WEIGHT_THRESHOLD = 20
LIGHTWEIGHT_RATE = 2.5
MEDIUM_WEIGHT_RATE = 2.0
HEAVYWEIGHT_RATE = 1.5

def calculate_shipping(weight):
    if weight < LIGHTWEIGHT_THRESHOLD:
        return weight * LIGHTWEIGHT_RATE
    elif weight < MEDIUM_WEIGHT_THRESHOLD:
        return weight * MEDIUM_WEIGHT_RATE
    else:
        return weight * HEAVYWEIGHT_RATE
```

### Meaningful Return Values

```python
# Bad: Unclear return values
def check_user(user):
    if user.is_active and user.email_verified:
        return 1
    elif user.is_active:
        return 2
    else:
        return 0

# Good: Explicit return values
from enum import Enum

class UserStatus(Enum):
    INACTIVE = "inactive"
    ACTIVE_UNVERIFIED = "active_unverified"
    ACTIVE_VERIFIED = "active_verified"

def get_user_status(user):
    if not user.is_active:
        return UserStatus.INACTIVE

    if not user.email_verified:
        return UserStatus.ACTIVE_UNVERIFIED

    return UserStatus.ACTIVE_VERIFIED
```

## Error Handling

### Use Exceptions, Not Return Codes

```python
# Bad: Return codes
def divide(a, b):
    if b == 0:
        return None, "Division by zero"
    return a / b, None

result, error = divide(10, 0)
if error:
    print(f"Error: {error}")

# Good: Exceptions
def divide(a, b):
    if b == 0:
        raise ValueError("Division by zero")
    return a / b

try:
    result = divide(10, 0)
except ValueError as e:
    print(f"Error: {e}")
```

### Specific Exceptions

```python
# Bad: Generic exceptions
def get_user(user_id):
    if not user_id:
        raise Exception("Invalid input")

    user = database.find(user_id)
    if not user:
        raise Exception("Not found")

    return user

# Good: Specific exceptions
class InvalidUserIdError(ValueError):
    pass

class UserNotFoundError(LookupError):
    pass

def get_user(user_id):
    if not user_id:
        raise InvalidUserIdError(f"Invalid user ID: {user_id}")

    user = database.find(user_id)
    if not user:
        raise UserNotFoundError(f"User not found: {user_id}")

    return user

# Caller can handle specific errors
try:
    user = get_user(user_id)
except InvalidUserIdError:
    # Handle invalid input
    pass
except UserNotFoundError:
    # Handle missing user
    pass
```

### Fail Fast

```python
# Bad: Late validation
def process_order(order):
    # Do lots of work
    calculate_total(order)
    apply_discounts(order)
    reserve_inventory(order)

    # Validate at the end
    if not order.user:
        raise ValueError("Order must have a user")

# Good: Validate early
def process_order(order):
    # Validate first
    if not order.user:
        raise ValueError("Order must have a user")

    # Then proceed
    calculate_total(order)
    apply_discounts(order)
    reserve_inventory(order)
```

## Summary

Writing clean code principles:

**Meaningful names:**

- Descriptive variable and function names
- Intent-revealing names
- Avoid abbreviations
- Context-appropriate naming

**Single responsibility:**

- Functions do one thing
- Small, focused functions
- Descriptive function names

**KISS principle:**

- Keep it simple
- Avoid over-engineering
- Solve current problems, not imaginary ones

**DRY principle:**

- Don't repeat yourself
- Extract common functionality
- Create reusable components

**Cognitive complexity:**

- Avoid deep nesting
- Use guard clauses
- Extract complex conditions
- Simplify boolean logic

**Self-documenting:**

- Expressive names over comments
- Constants for magic numbers
- Clear return values

**Error handling:**

- Use exceptions, not return codes
- Specific exception types
- Fail fast

Clean code is code that's easy to read, understand, and modify.

## Next Steps

- Document with [Documentation](documentation.md)
- Organize with [Code Organization](code-organization.md)
- Apply [Design Patterns](design-patterns.md)
