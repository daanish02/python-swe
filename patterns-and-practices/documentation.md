# Documentation

## Table of Contents

- [Why Documentation Matters](#why-documentation-matters)
- [Docstrings](#docstrings)
- [Comments Best Practices](#comments-best-practices)
- [README Files](#readme-files)
- [API Documentation](#api-documentation)
- [Documentation Tools](#documentation-tools)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Why Documentation Matters

### The Purpose of Documentation

```python
# Documentation serves multiple audiences:
# 1. Future you (in 6 months)
# 2. Your team members
# 3. External users of your code
# 4. Contributors to your project

# Good documentation:
# - Explains WHY, not just WHAT
# - Provides examples
# - Is kept up to date
# - Is easy to find
```

### When to Document

```python
# Always document:
# - Public APIs
# - Complex algorithms
# - Non-obvious design decisions
# - Assumptions and constraints

# Don't document:
# - Obvious code
# - Implementation details that may change
# - Redundant information already in code
```

## Docstrings

### Function Docstrings

```python
# Bad: No docstring
def calculate_total(items, tax_rate):
    return sum(item.price for item in items) * (1 + tax_rate)

# Good: Clear docstring
def calculate_total(items, tax_rate):
    """Calculate total price including tax.

    Args:
        items: List of items with price attribute
        tax_rate: Tax rate as decimal (e.g., 0.08 for 8%)

    Returns:
        Total price with tax applied

    Raises:
        ValueError: If tax_rate is negative
    """
    if tax_rate < 0:
        raise ValueError("Tax rate cannot be negative")

    subtotal = sum(item.price for item in items)
    return subtotal * (1 + tax_rate)
```

### Google Style Docstrings

```python
def fetch_user_data(user_id, include_orders=False, timeout=30):
    """Fetch user data from the database.

    Retrieves user information and optionally their order history.
    Implements connection pooling and automatic retry logic.

    Args:
        user_id (int): The unique identifier for the user.
        include_orders (bool, optional): Whether to include order history.
            Defaults to False.
        timeout (int, optional): Request timeout in seconds. Defaults to 30.

    Returns:
        dict: A dictionary containing user data with the following keys:
            - id (int): User identifier
            - name (str): User's full name
            - email (str): User's email address
            - orders (list): List of orders if include_orders is True

    Raises:
        UserNotFoundError: If user_id doesn't exist in database.
        TimeoutError: If request exceeds timeout limit.
        DatabaseError: If database connection fails.

    Example:
        >>> user = fetch_user_data(123, include_orders=True)
        >>> print(user['name'])
        'Alice Smith'

    Note:
        This function uses connection pooling. Connections are automatically
        returned to the pool after use.
    """
    pass
```

### NumPy/SciPy Style Docstrings

```python
def calculate_statistics(data, weights=None, ddof=0):
    """
    Calculate descriptive statistics for a dataset.

    Computes mean, variance, and standard deviation with optional
    weighted values and degrees of freedom adjustment.

    Parameters
    ----------
    data : array_like
        Input data, can be a list, tuple, or numpy array.
    weights : array_like, optional
        Weights for each data point. Must have same length as data.
        Default is None (equal weights).
    ddof : int, optional
        Degrees of freedom adjustment for variance calculation.
        Default is 0.

    Returns
    -------
    dict
        Dictionary with keys 'mean', 'variance', and 'std_dev'.

    Raises
    ------
    ValueError
        If data is empty or weights length doesn't match data length.
    TypeError
        If data or weights contain non-numeric values.

    See Also
    --------
    numpy.mean : Compute the arithmetic mean.
    numpy.var : Compute the variance.

    Examples
    --------
    >>> data = [1, 2, 3, 4, 5]
    >>> stats = calculate_statistics(data)
    >>> print(stats['mean'])
    3.0

    >>> weights = [1, 1, 2, 2, 2]
    >>> stats = calculate_statistics(data, weights=weights)
    >>> print(stats['mean'])
    3.5

    Notes
    -----
    The weighted mean is calculated as:

    .. math:: \\bar{x} = \\frac{\\sum w_i x_i}{\\sum w_i}

    References
    ----------
    .. [1] Wikipedia, "Weighted arithmetic mean"
           https://en.wikipedia.org/wiki/Weighted_arithmetic_mean
    """
    pass
```

### Class Docstrings

```python
class ShoppingCart:
    """Shopping cart for e-commerce application.

    Manages items in a user's shopping cart, calculates totals,
    and applies discounts.

    Attributes:
        user_id (int): Identifier for the cart owner.
        items (list): List of CartItem objects.
        discount_code (str): Applied discount code, if any.

    Example:
        >>> cart = ShoppingCart(user_id=123)
        >>> cart.add_item(product_id=456, quantity=2)
        >>> print(cart.total)
        39.98
    """

    def __init__(self, user_id):
        """Initialize shopping cart.

        Args:
            user_id (int): The ID of the user owning this cart.
        """
        self.user_id = user_id
        self.items = []
        self.discount_code = None

    def add_item(self, product_id, quantity=1):
        """Add an item to the cart.

        Args:
            product_id (int): The product identifier.
            quantity (int, optional): Number of items to add. Defaults to 1.

        Raises:
            ValueError: If quantity is not positive.
        """
        pass
```

### Module Docstrings

```python
"""User authentication and authorization module.

This module provides functions and classes for user authentication,
session management, and permission checking in the application.

The module includes:
- User login/logout functionality
- JWT token generation and validation
- Role-based access control (RBAC)
- Password hashing and verification

Example:
    Basic authentication usage::

        from auth import authenticate_user, create_session

        user = authenticate_user(username, password)
        token = create_session(user)

Note:
    This module requires the 'cryptography' package for password hashing.
    Install with: pip install cryptography

Todo:
    * Add support for OAuth2
    * Implement two-factor authentication
    * Add rate limiting for login attempts
"""

import hashlib
from typing import Optional
```

## Comments Best Practices

### Good Comments

```python
# Explain WHY, not WHAT
# Bad
x = x + 1  # Increment x

# Good
x = x + 1  # Adjust for 1-based indexing

# Explain non-obvious decisions
def process_data(data):
    # Use binary search instead of linear because dataset is large
    # and sorted. Reduces O(n) to O(log n).
    result = binary_search(data, target)

    # Cache results for 5 minutes to reduce database load during
    # peak hours. Stale data is acceptable per business requirements.
    cache.set(key, result, timeout=300)

# Explain complex algorithms
def calculate_hash(data):
    # Using SHA-256 for cryptographic hashing
    # Alternative considered: MD5 (rejected due to security vulnerabilities)
    # See: https://example.com/security-policy
    return hashlib.sha256(data.encode()).hexdigest()
```

### Comments to Avoid

```python
# Bad: Stating the obvious
# Create a user
user = User()

# Set the name
user.name = "Alice"

# Save to database
user.save()

# Bad: Commented-out code (use version control instead)
# def old_function():
#     return "deprecated"

# Bad: Redundant comments
# This function adds two numbers together
def add(a, b):
    # Return the sum of a and b
    return a + b
```

### TODO Comments

```python
# TODO: Implement error handling for network failures
def fetch_data(url):
    return requests.get(url)

# FIXME: This breaks when input is None
def process(data):
    return data.upper()

# HACK: Temporary workaround for API rate limiting
# Remove after API v2 is deployed
def rate_limited_call():
    time.sleep(1)
    return api.call()

# NOTE: This assumes input is already sorted
def binary_search(arr, target):
    pass

# WARNING: This function is not thread-safe
global_counter = 0
def increment():
    global global_counter
    global_counter += 1
```

### Documenting Gotchas

```python
def parse_date(date_string):
    """Parse date string into datetime object.

    Args:
        date_string: Date in format 'YYYY-MM-DD'

    Returns:
        datetime object

    Warning:
        Input must be in UTC. Local timezone is not supported.
        Will raise ValueError for invalid formats.
    """
    pass

def calculate_discount(price, percentage):
    """Calculate discounted price.

    Important:
        percentage should be a decimal (0.1 for 10%), not an integer.
        Using 10 instead of 0.1 will result in incorrect calculation.
    """
    return price * (1 - percentage)
```

## README Files

### Project README Structure

````markdown
# Project Name

Brief description of what the project does and why it exists.

## Features

- Feature 1
- Feature 2
- Feature 3

## Installation

```bash
pip install package-name
```
````

Or install from source:

```bash
git clone https://github.com/user/repo.git
cd repo
pip install -e .
```

## Quick Start

```python
from package import main_function

result = main_function(arg1, arg2)
print(result)
```

## Usage

### Basic Example

```python
# Example code showing common use case
```

### Advanced Example

```python
# More complex example
```

## Configuration

Explain configuration options and environment variables.

## Documentation

Link to full documentation: https://docs.example.com

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

MIT License - see [LICENSE](LICENSE) file.

## Support

- Issue tracker: https://github.com/user/repo/issues
- Email: support@example.com

````

### Module-Level README

```python
# my_package/
#   README.md          # Package overview
#   module1/
#     README.md        # Module1 specific docs
#     __init__.py
#   module2/
#     README.md        # Module2 specific docs
#     __init__.py

# Example module README
"""
# Module: Data Processing

This module handles all data transformation and validation logic.

## Components

- `validators.py`: Input validation functions
- `transformers.py`: Data transformation classes
- `processors.py`: Main processing pipeline

## Usage

```python
from my_package.data_processing import DataProcessor

processor = DataProcessor()
result = processor.process(data)
````

## Design Decisions

- Uses streaming processing to handle large datasets
- Implements lazy evaluation for better performance
- Thread-safe for concurrent processing
  """

````

## API Documentation

### Documenting APIs

```python
class UserAPI:
    """REST API for user management.

    Base URL: /api/v1/users

    Authentication:
        All endpoints require JWT token in Authorization header:
        Authorization: Bearer <token>

    Rate Limiting:
        100 requests per minute per user
    """

    def get_user(self, user_id):
        """Get user by ID.

        Endpoint: GET /api/v1/users/{user_id}

        Args:
            user_id (int): User identifier

        Returns:
            dict: User data
            {
                "id": 123,
                "name": "Alice",
                "email": "alice@example.com",
                "created_at": "2024-01-01T00:00:00Z"
            }

        Raises:
            UserNotFoundError: 404 if user doesn't exist
            UnauthorizedError: 401 if token is invalid

        Example:
            >>> api = UserAPI()
            >>> user = api.get_user(123)
            >>> print(user['name'])
            'Alice'
        """
        pass

    def create_user(self, name, email):
        """Create new user.

        Endpoint: POST /api/v1/users

        Request Body:
            {
                "name": "string",
                "email": "string"
            }

        Response:
            201 Created
            {
                "id": 124,
                "name": "Bob",
                "email": "bob@example.com",
                "created_at": "2024-01-02T00:00:00Z"
            }

        Errors:
            400 Bad Request: Invalid input
            409 Conflict: Email already exists
        """
        pass
````

## Documentation Tools

### Sphinx

```python
# Install Sphinx
# pip install sphinx sphinx-rtd-theme

# Generate documentation structure
# sphinx-quickstart docs

# conf.py configuration
# extensions = [
#     'sphinx.ext.autodoc',
#     'sphinx.ext.napoleon',  # For Google/NumPy style docstrings
#     'sphinx.ext.viewcode',
# ]

# Build documentation
# cd docs
# make html

# Example RST file (index.rst)
"""
Welcome to My Project
=====================

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   installation
   usage
   api
   contributing

API Reference
=============

.. automodule:: mypackage.module
   :members:
   :undoc-members:
   :show-inheritance:
"""
```

### MkDocs

```yaml
# mkdocs.yml
site_name: My Project
theme:
  name: material

nav:
  - Home: index.md
  - Getting Started: getting-started.md
  - API Reference: api.md
  - Contributing: contributing.md

plugins:
  - search
  - mkdocstrings:
      handlers:
        python:
          options:
            show_source: true
```

### pdoc

```python
# Install pdoc
# pip install pdoc

# Generate documentation
# pdoc --html mypackage

# Or serve live documentation
# pdoc --http localhost:8080 mypackage

# pdoc uses docstrings directly, no configuration needed
```

### Type Hints for Documentation

```python
from typing import List, Dict, Optional, Union

def process_users(
    users: List[Dict[str, str]],
    filter_active: bool = True,
    limit: Optional[int] = None
) -> List[Dict[str, Union[str, int]]]:
    """Process user list with filtering.

    Type hints provide additional documentation and enable
    type checking with mypy.
    """
    pass
```

## Summary

Documentation best practices:

**Docstrings:**

- Use for all public APIs
- Include Args, Returns, Raises
- Provide examples
- Google or NumPy style

**Comments:**

- Explain WHY, not WHAT
- Document non-obvious decisions
- Avoid redundant comments
- Use TODO/FIXME appropriately

**README files:**

- Project overview
- Installation instructions
- Quick start examples
- Links to full documentation

**API documentation:**

- Endpoint descriptions
- Request/response formats
- Error codes
- Authentication requirements

**Tools:**

- Sphinx: Comprehensive documentation
- MkDocs: Modern, user-friendly
- pdoc: Simple, automatic
- Type hints: Enhanced documentation

Good documentation is maintained, accurate, and helpful to your target audience.

## Next Steps

- Organize with [Code Organization](code-organization.md)
- Apply [Code Style Standards](code-style-and-standards.md)
- Structure with [Design Patterns](design-patterns.md)
