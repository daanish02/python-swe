# Code Organization

## Table of Contents

- [Project Structure](#project-structure)
- [Module Design](#module-design)
- [Package Organization](#package-organization)
- [Separation of Concerns](#separation-of-concerns)
- [Import Management](#import-management)
- [Configuration Management](#configuration-management)
- [Summary](#summary)
- [Next Steps](#next-steps)

## Project Structure

### Basic Project Layout

```
myproject/
├── README.md
├── LICENSE
├── setup.py / pyproject.toml
├── requirements.txt
├── .gitignore
├── docs/
│   ├── index.md
│   └── api.md
├── tests/
│   ├── __init__.py
│   ├── test_module1.py
│   └── test_module2.py
├── src/
│   └── myproject/
│       ├── __init__.py
│       ├── main.py
│       ├── config.py
│       └── utils.py
└── scripts/
    └── deploy.sh
```

### Application Project Structure

```
webapp/
├── README.md
├── requirements.txt
├── setup.py
├── .env.example
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── order.py
│   ├── views/
│   │   ├── __init__.py
│   │   ├── auth.py
│   │   └── api.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── email.py
│   │   └── payment.py
│   ├── repositories/
│   │   ├── __init__.py
│   │   └── user_repository.py
│   └── utils/
│       ├── __init__.py
│       ├── validators.py
│       └── helpers.py
├── tests/
│   ├── conftest.py
│   ├── test_models/
│   ├── test_views/
│   └── test_services/
├── migrations/
│   └── versions/
└── static/
    └── css/
```

### Library Project Structure

```
mylibrary/
├── README.md
├── LICENSE
├── pyproject.toml
├── setup.py
├── docs/
│   ├── conf.py
│   ├── index.rst
│   └── api.rst
├── src/
│   └── mylibrary/
│       ├── __init__.py
│       ├── core.py
│       ├── utils.py
│       └── exceptions.py
├── tests/
│   ├── __init__.py
│   ├── test_core.py
│   └── test_utils.py
└── examples/
    ├── basic_usage.py
    └── advanced_usage.py
```

## Module Design

### Single Responsibility

```python
# Bad: Module does too many things
# users.py
class User:
    pass

class UserRepository:
    pass

class UserValidator:
    pass

class EmailService:
    pass

class PaymentProcessor:
    pass

# Good: Separate modules by responsibility
# models/user.py
class User:
    pass

# repositories/user_repository.py
class UserRepository:
    pass

# validators/user_validator.py
class UserValidator:
    pass

# services/email_service.py
class EmailService:
    pass

# services/payment_service.py
class PaymentProcessor:
    pass
```

### Module Size

```python
# Keep modules focused and manageable
# Bad: 2000+ line module with everything

# Good: Multiple focused modules
# database/
#   connection.py      (50 lines)
#   queries.py         (150 lines)
#   migrations.py      (100 lines)
#   models.py          (200 lines)

# Each file has clear responsibility
```

### Module Interface

```python
# mymodule/__init__.py
"""Public API for mymodule."""

from .core import MainClass, helper_function
from .utils import utility_function
from .exceptions import CustomError

# Define public API
__all__ = [
    'MainClass',
    'helper_function',
    'utility_function',
    'CustomError',
]

# Users import from package, not submodules
# from mymodule import MainClass  # Good
# from mymodule.core import MainClass  # Implementation detail
```

## Package Organization

### Flat vs Nested Packages

```python
# Flat (better for small projects)
mypackage/
├── __init__.py
├── module1.py
├── module2.py
├── module3.py
└── utils.py

# Nested (better for large projects)
mypackage/
├── __init__.py
├── core/
│   ├── __init__.py
│   ├── engine.py
│   └── processor.py
├── models/
│   ├── __init__.py
│   ├── base.py
│   └── user.py
├── services/
│   ├── __init__.py
│   └── api.py
└── utils/
    ├── __init__.py
    ├── validators.py
    └── helpers.py
```

### Feature-Based Organization

```python
# Organize by feature, not by type
# Bad: Type-based
myapp/
├── models/
│   ├── user.py
│   ├── order.py
│   └── product.py
├── views/
│   ├── user_view.py
│   ├── order_view.py
│   └── product_view.py
└── services/
    ├── user_service.py
    ├── order_service.py
    └── product_service.py

# Good: Feature-based
myapp/
├── users/
│   ├── __init__.py
│   ├── models.py
│   ├── views.py
│   ├── services.py
│   └── tests.py
├── orders/
│   ├── __init__.py
│   ├── models.py
│   ├── views.py
│   ├── services.py
│   └── tests.py
└── products/
    ├── __init__.py
    ├── models.py
    ├── views.py
    ├── services.py
    └── tests.py
```

### Layer Architecture

```python
# Organize by architectural layers
myapp/
├── presentation/      # UI layer
│   ├── api/
│   ├── views/
│   └── templates/
├── application/       # Application logic
│   ├── services/
│   └── use_cases/
├── domain/           # Business logic
│   ├── models/
│   ├── entities/
│   └── value_objects/
├── infrastructure/   # External concerns
│   ├── database/
│   ├── cache/
│   └── external_apis/
└── shared/          # Common utilities
    ├── utils/
    └── exceptions/
```

## Separation of Concerns

### MVC Pattern

```python
# Model - Data and business logic
# models/user.py
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def validate(self):
        if not self.email or '@' not in self.email:
            raise ValueError("Invalid email")

# View - Presentation logic
# views/user_view.py
class UserView:
    def render_user(self, user):
        return {
            'name': user.name,
            'email': user.email
        }

    def render_error(self, error):
        return {'error': str(error)}

# Controller - Request handling
# controllers/user_controller.py
class UserController:
    def __init__(self, user_service, user_view):
        self.user_service = user_service
        self.user_view = user_view

    def create_user(self, request):
        try:
            user = self.user_service.create_user(
                name=request.name,
                email=request.email
            )
            return self.user_view.render_user(user)
        except ValueError as e:
            return self.user_view.render_error(e)
```

### Repository Pattern

```python
# Separate data access from business logic

# repositories/user_repository.py
class UserRepository:
    """Handles data persistence."""

    def __init__(self, database):
        self.db = database

    def find_by_id(self, user_id):
        return self.db.query(User).filter_by(id=user_id).first()

    def save(self, user):
        self.db.add(user)
        self.db.commit()

    def delete(self, user):
        self.db.delete(user)
        self.db.commit()

# services/user_service.py
class UserService:
    """Contains business logic."""

    def __init__(self, user_repository):
        self.repository = user_repository

    def create_user(self, name, email):
        user = User(name, email)
        user.validate()
        self.repository.save(user)
        return user

    def get_user(self, user_id):
        user = self.repository.find_by_id(user_id)
        if not user:
            raise UserNotFoundError(f"User {user_id} not found")
        return user
```

### Dependency Injection

```python
# Bad: Hard-coded dependencies
class UserService:
    def __init__(self):
        self.repository = UserRepository()  # Tight coupling
        self.email_service = EmailService()  # Hard to test

# Good: Inject dependencies
class UserService:
    def __init__(self, repository, email_service):
        self.repository = repository
        self.email_service = email_service

# Usage
repository = UserRepository(database)
email_service = EmailService(smtp_config)
user_service = UserService(repository, email_service)

# Easy to test with mocks
mock_repo = MockUserRepository()
mock_email = MockEmailService()
service = UserService(mock_repo, mock_email)
```

## Import Management

### Absolute vs Relative Imports

```python
# Absolute imports (preferred)
from mypackage.module import function
from mypackage.subpackage.module import Class

# Relative imports (for intra-package references)
from . import sibling_module
from .sibling_module import function
from .. import parent_module
from ..parent_module import Class

# Example structure
mypackage/
├── __init__.py
├── module_a.py
└── subpackage/
    ├── __init__.py
    └── module_b.py

# In module_b.py
from mypackage.module_a import func  # Absolute (preferred)
from ..module_a import func          # Relative (OK within package)
```

### Organizing Imports

```python
# Standard library
import os
import sys
from pathlib import Path
from typing import List, Dict

# Third-party
import numpy as np
import pandas as pd
from flask import Flask, request
from sqlalchemy import create_engine

# Local application
from myapp.models import User, Order
from myapp.services import UserService
from myapp.utils import validate_email

# Group and sort within groups
```

### Avoiding Circular Imports

```python
# Problem: Circular dependency
# module_a.py
from module_b import ClassB

class ClassA:
    def __init__(self):
        self.b = ClassB()

# module_b.py
from module_a import ClassA  # Circular import!

class ClassB:
    def __init__(self):
        self.a = ClassA()

# Solution 1: Restructure (move shared code to new module)
# module_c.py
class BaseClass:
    pass

# module_a.py
from module_c import BaseClass

class ClassA(BaseClass):
    pass

# module_b.py
from module_c import BaseClass

class ClassB(BaseClass):
    pass

# Solution 2: Import inside function
# module_a.py
class ClassA:
    def create_b(self):
        from module_b import ClassB  # Import when needed
        return ClassB()

# Solution 3: Use dependency injection
# module_a.py
class ClassA:
    def __init__(self, b_factory):
        self.b_factory = b_factory

    def create_b(self):
        return self.b_factory()
```

## Configuration Management

### Configuration Files

```python
# config.py - Base configuration
class Config:
    """Base configuration."""
    DEBUG = False
    TESTING = False
    SECRET_KEY = os.environ.get('SECRET_KEY')
    DATABASE_URI = os.environ.get('DATABASE_URI')

class DevelopmentConfig(Config):
    """Development configuration."""
    DEBUG = True
    DATABASE_URI = 'sqlite:///dev.db'

class ProductionConfig(Config):
    """Production configuration."""
    DATABASE_URI = os.environ.get('DATABASE_URI')

class TestingConfig(Config):
    """Testing configuration."""
    TESTING = True
    DATABASE_URI = 'sqlite:///:memory:'

# Usage
config_by_name = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'testing': TestingConfig,
}

config = config_by_name[os.environ.get('FLASK_ENV', 'development')]
```

### Environment Variables

```python
# .env.example (template)
SECRET_KEY=your-secret-key
DATABASE_URI=postgresql://user:pass@localhost/db
API_KEY=your-api-key
DEBUG=true

# .env (actual values, not in version control)
SECRET_KEY=actual-secret-key
DATABASE_URI=postgresql://user:pass@localhost/mydb
API_KEY=actual-api-key
DEBUG=true

# Load with python-dotenv
from dotenv import load_dotenv
import os

load_dotenv()

SECRET_KEY = os.environ.get('SECRET_KEY')
DATABASE_URI = os.environ.get('DATABASE_URI')
DEBUG = os.environ.get('DEBUG', 'false').lower() == 'true'
```

### Settings Module

```python
# settings/
#   __init__.py
#   base.py
#   development.py
#   production.py

# settings/base.py
class BaseSettings:
    PROJECT_NAME = "MyApp"
    API_VERSION = "1.0"

    @property
    def database_url(self):
        return os.environ.get('DATABASE_URI')

# settings/development.py
from .base import BaseSettings

class DevelopmentSettings(BaseSettings):
    DEBUG = True
    LOG_LEVEL = "DEBUG"

# settings/production.py
from .base import BaseSettings

class ProductionSettings(BaseSettings):
    DEBUG = False
    LOG_LEVEL = "INFO"

# settings/__init__.py
import os
from .development import DevelopmentSettings
from .production import ProductionSettings

ENV = os.environ.get('ENV', 'development')

settings = {
    'development': DevelopmentSettings(),
    'production': ProductionSettings(),
}[ENV]
```

## Summary

Code organization principles:

**Project structure:**

- Clear directory layout
- Separate source, tests, docs
- Application vs library structure

**Module design:**

- Single responsibility
- Manageable size (100-300 lines)
- Clear public interface

**Package organization:**

- Flat for simple, nested for complex
- Feature-based or layer-based
- Consistent conventions

**Separation of concerns:**

- MVC pattern
- Repository pattern
- Dependency injection

**Import management:**

- Absolute imports preferred
- Organized by type
- Avoid circular dependencies

**Configuration:**

- Environment-based config
- Environment variables
- Separate settings module

Good organization makes code easier to navigate, understand, and maintain.

## Next Steps

- Apply [Design Patterns](design-patterns.md)
- Follow [SOLID Principles](solid-principles.md)
- Improve with [Refactoring](refactoring.md)
