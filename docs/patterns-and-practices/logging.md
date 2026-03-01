# Logging

## Why Logging Matters

### Logging vs Print Statements

```python
# Bad: Using print for debugging
def process_order(order):
    print(f"Processing order {order.id}")  # Lost in production
    total = calculate_total(order)
    print(f"Total: {total}")  # No timestamp, no context
    return total

# Good: Using logging
import logging

logger = logging.getLogger(__name__)

def process_order(order):
    logger.info("Processing order %s for user %s", order.id, order.user_id)
    total = calculate_total(order)
    logger.debug("Calculated total: %.2f", total)
    return total
```

### The Purpose of Logging

```python
# Logging serves multiple purposes:

# 1. Debugging - Understand what happened
logger.debug("User input: %s", user_data)

# 2. Monitoring - Track application behavior
logger.info("Successfully processed payment of $%.2f", amount)

# 3. Auditing - Record important events
logger.info("User %s accessed sensitive resource %s", user_id, resource)

# 4. Error tracking - Capture failures
logger.error("Payment gateway timeout after 30s", exc_info=True)

# 5. Performance analysis - Identify bottlenecks
logger.info("Query executed in %.3f seconds", elapsed_time)
```

### Production vs Development

```python
# Development: Verbose debugging
logger.debug("Variable x = %s, y = %s", x, y)
logger.debug("Entering function with args: %s", args)

# Production: Important events only
logger.info("Application started successfully")
logger.warning("Database connection pool nearly exhausted: %d/%d", used, total)
logger.error("Failed to process payment for order %s", order_id)
```

## The Logging Module

### Basic Logging Example

```python
import logging

# Configure basic logging
logging.basicConfig(level=logging.INFO)

# Log messages at different levels
logging.debug("Detailed debug information")
logging.info("General information")
logging.warning("Warning message")
logging.error("Error message")
logging.critical("Critical error")
```

### Creating Named Loggers

```python
# Bad: Using root logger
import logging
logging.info("Application started")  # Uses root logger

# Good: Named logger per module
import logging

logger = logging.getLogger(__name__)
logger.info("Application started")  # Uses 'mymodule' logger

# In myapp/database/connection.py
logger = logging.getLogger(__name__)
# Logger name will be 'myapp.database.connection'
```

### Logger Hierarchy

```python
# Loggers form a hierarchy based on their names
import logging

# Root logger
root = logging.getLogger()

# Child loggers
app_logger = logging.getLogger('myapp')
db_logger = logging.getLogger('myapp.database')
api_logger = logging.getLogger('myapp.api')

# Hierarchy:
# root
#   └── myapp
#       ├── myapp.database
#       └── myapp.api

# Child loggers inherit settings from parents
app_logger.setLevel(logging.DEBUG)
# myapp.database and myapp.api also get DEBUG level
```

## Log Levels

### The Five Standard Levels

```python
import logging

# DEBUG (10): Detailed diagnostic information
logger.debug("Function called with args: %s", args)
logger.debug("Processing item %d of %d", i, total)

# INFO (20): Confirmation things are working
logger.info("Server started on port 8000")
logger.info("User %s logged in successfully", username)

# WARNING (30): Something unexpected, but app continues
logger.warning("API rate limit at 90%% capacity")
logger.warning("Disk space low: %d%% free", percent_free)

# ERROR (40): Serious problem, couldn't perform function
logger.error("Failed to connect to database: %s", error)
logger.error("Unable to process order %s", order_id)

# CRITICAL (50): Very serious error, app may shut down
logger.critical("Out of memory - shutting down")
logger.critical("All database connections failed")
```

### Choosing the Right Level

```python
# Use DEBUG for:
# - Variable values during development
# - Function entry/exit traces
# - Detailed diagnostic data

logger.debug("Received request: %s", request_data)

# Use INFO for:
# - Major application milestones
# - Important user actions
# - Successful operations

logger.info("Payment processed successfully for order %s", order_id)

# Use WARNING for:
# - Deprecation notices
# - Recoverable errors
# - Resource constraints

logger.warning("Using deprecated API method %s", method_name)

# Use ERROR for:
# - Caught exceptions
# - Failed operations
# - Data integrity issues

logger.error("Failed to update user record", exc_info=True)

# Use CRITICAL for:
# - System-level failures
# - Unrecoverable errors
# - Security breaches

logger.critical("Authentication system offline")
```

### Setting Log Levels

```python
import logging

logger = logging.getLogger(__name__)

# Set minimum level for this logger
logger.setLevel(logging.DEBUG)  # Show DEBUG and above

# Only INFO and above will be displayed
logger.setLevel(logging.INFO)

# Check if level is enabled before expensive operations
if logger.isEnabledFor(logging.DEBUG):
    logger.debug("Expensive data: %s", expensive_function())
```

## Basic Configuration

### Using basicConfig()

```python
import logging

# Simple configuration
logging.basicConfig(level=logging.INFO)

# More detailed configuration
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S',
    filename='app.log',
    filemode='a'  # Append mode
)

# Configuration options:
# - level: Minimum severity level
# - format: How messages are formatted
# - datefmt: Date/time format
# - filename: Log to file instead of console
# - filemode: 'a' (append) or 'w' (overwrite)
# - stream: Output stream (e.g., sys.stdout)
```

### Format Strings

```python
import logging

# Common format attributes
logging.basicConfig(
    format='%(asctime)s | %(levelname)-8s | %(name)s | %(message)s'
)
# Output: 2026-02-24 10:30:45,123 | INFO     | myapp | User logged in

# More detailed format
logging.basicConfig(
    format='%(asctime)s | %(levelname)s | %(name)s:%(lineno)d | %(funcName)s | %(message)s'
)
# Output: 2026-02-24 10:30:45,123 | INFO | myapp:42 | login | User logged in

# Available format attributes:
# %(asctime)s       - Human-readable time
# %(name)s          - Logger name
# %(levelname)s     - Text level (INFO, ERROR, etc.)
# %(levelno)s       - Numeric level
# %(message)s       - Log message
# %(filename)s      - Source filename
# %(lineno)d        - Line number
# %(funcName)s      - Function name
# %(module)s        - Module name
# %(pathname)s      - Full pathname
# %(process)d       - Process ID
# %(thread)d        - Thread ID
# %(threadName)s    - Thread name
```

### Date Formatting

```python
import logging

# Different date formats
logging.basicConfig(
    format='%(asctime)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S'
)
# Output: 2026-02-24 10:30:45 - Message

logging.basicConfig(
    format='%(asctime)s - %(message)s',
    datefmt='%Y/%m/%d %I:%M:%S %p'
)
# Output: 2026/02/24 10:30:45 AM - Message

logging.basicConfig(
    format='%(asctime)s - %(message)s',
    datefmt='%b %d %H:%M:%S'
)
# Output: Feb 24 10:30:45 - Message
```

## Loggers, Handlers, and Formatters

### The Logging Architecture

```python
# Logging flow:
# Logger → Handler → Formatter
#   ↓         ↓          ↓
#  emit    output     format

import logging

# 1. Create logger
logger = logging.getLogger('myapp')
logger.setLevel(logging.DEBUG)

# 2. Create handler (where logs go)
handler = logging.StreamHandler()  # Console output
handler.setLevel(logging.INFO)

# 3. Create formatter (how logs look)
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# 4. Connect them
handler.setFormatter(formatter)
logger.addHandler(handler)

# 5. Use the logger
logger.info("Application started")
```

### Multiple Handlers

```python
import logging
import logging.handlers

logger = logging.getLogger('myapp')
logger.setLevel(logging.DEBUG)

# Console handler - INFO and above
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)
console_formatter = logging.Formatter('%(levelname)s - %(message)s')
console_handler.setFormatter(console_formatter)

# File handler - DEBUG and above
file_handler = logging.FileHandler('debug.log')
file_handler.setLevel(logging.DEBUG)
file_formatter = logging.Formatter(
    '%(asctime)s - %(name)s:%(lineno)d - %(levelname)s - %(message)s'
)
file_handler.setFormatter(file_formatter)

# Error file handler - ERROR and above
error_handler = logging.FileHandler('errors.log')
error_handler.setLevel(logging.ERROR)
error_handler.setFormatter(file_formatter)

# Add all handlers
logger.addHandler(console_handler)
logger.addHandler(file_handler)
logger.addHandler(error_handler)

# Now:
# - DEBUG goes to debug.log only
# - INFO goes to console and debug.log
# - ERROR goes to all three destinations
```

### Handler Types

```python
import logging
import logging.handlers

# StreamHandler - Output to console
console = logging.StreamHandler()

# FileHandler - Write to a file
file = logging.FileHandler('app.log')

# RotatingFileHandler - Rotate when file gets too large
rotating = logging.handlers.RotatingFileHandler(
    'app.log',
    maxBytes=10*1024*1024,  # 10MB
    backupCount=5  # Keep 5 backup files
)

# TimedRotatingFileHandler - Rotate based on time
timed = logging.handlers.TimedRotatingFileHandler(
    'app.log',
    when='midnight',  # Rotate at midnight
    interval=1,       # Every day
    backupCount=7     # Keep 7 days
)

# SMTPHandler - Send logs via email
email = logging.handlers.SMTPHandler(
    mailhost='smtp.example.com',
    fromaddr='app@example.com',
    toaddrs=['admin@example.com'],
    subject='Application Error'
)

# SysLogHandler - Send to syslog
syslog = logging.handlers.SysLogHandler(address='/dev/log')

# HTTPHandler - Send to web server
http = logging.handlers.HTTPHandler(
    'localhost:8080',
    '/log',
    method='POST'
)
```

### Custom Formatters

```python
import logging

class CustomFormatter(logging.Formatter):
    """Custom formatter with colors for console output."""

    grey = "\x1b[38;21m"
    yellow = "\x1b[33;21m"
    red = "\x1b[31;21m"
    bold_red = "\x1b[31;1m"
    reset = "\x1b[0m"

    format_str = "%(asctime)s - %(name)s - %(levelname)s - %(message)s"

    FORMATS = {
        logging.DEBUG: grey + format_str + reset,
        logging.INFO: grey + format_str + reset,
        logging.WARNING: yellow + format_str + reset,
        logging.ERROR: red + format_str + reset,
        logging.CRITICAL: bold_red + format_str + reset
    }

    def format(self, record):
        log_fmt = self.FORMATS.get(record.levelno)
        formatter = logging.Formatter(log_fmt)
        return formatter.format(record)

# Usage
logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
handler.setFormatter(CustomFormatter())
logger.addHandler(handler)
```

## Configuration Patterns

### Production Configuration

```python
import logging
import logging.handlers

def setup_logging():
    """Configure logging for production use."""

    # Root logger configuration
    logger = logging.getLogger()
    logger.setLevel(logging.INFO)

    # Console handler for containerized environments
    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.INFO)
    console_formatter = logging.Formatter(
        '%(asctime)s | %(levelname)-8s | %(name)s | %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    console_handler.setFormatter(console_formatter)

    # Rotating file handler
    file_handler = logging.handlers.RotatingFileHandler(
        'logs/app.log',
        maxBytes=10*1024*1024,  # 10MB
        backupCount=5
    )
    file_handler.setLevel(logging.DEBUG)
    file_formatter = logging.Formatter(
        '%(asctime)s | %(levelname)-8s | %(name)s:%(lineno)d | %(funcName)s | %(message)s',
        datefmt='%Y-%m-%d %H:%M:%S'
    )
    file_handler.setFormatter(file_formatter)

    # Add handlers
    logger.addHandler(console_handler)
    logger.addHandler(file_handler)

if __name__ == '__main__':
    setup_logging()
    logger = logging.getLogger(__name__)
    logger.info("Application started")
```

### Dictionary Configuration

```python
import logging.config

LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': '%(asctime)s | %(levelname)-8s | %(name)s | %(message)s'
        },
        'detailed': {
            'format': '%(asctime)s | %(levelname)-8s | %(name)s:%(lineno)d | %(funcName)s | %(message)s'
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            'formatter': 'standard',
            'stream': 'ext://sys.stdout'
        },
        'file': {
            'class': 'logging.handlers.RotatingFileHandler',
            'level': 'DEBUG',
            'formatter': 'detailed',
            'filename': 'logs/app.log',
            'maxBytes': 10485760,
            'backupCount': 5
        },
        'error_file': {
            'class': 'logging.FileHandler',
            'level': 'ERROR',
            'formatter': 'detailed',
            'filename': 'logs/errors.log',
        },
    },
    'loggers': {
        '': {  # Root logger
            'level': 'INFO',
            'handlers': ['console', 'file']
        },
        'myapp': {
            'level': 'DEBUG',
            'handlers': ['console', 'file', 'error_file'],
            'propagate': False
        },
        'myapp.sensitive': {
            'level': 'WARNING',
            'handlers': ['file'],
            'propagate': False
        },
    }
}

# Apply configuration
logging.config.dictConfig(LOGGING_CONFIG)

# Use loggers
logger = logging.getLogger('myapp')
logger.info("Application started")
```

### Environment-Based Configuration

```python
import logging
import os

def configure_logging():
    """Configure logging based on environment."""

    env = os.getenv('ENVIRONMENT', 'development')

    if env == 'production':
        level = logging.INFO
        log_file = '/var/log/myapp/app.log'
    elif env == 'staging':
        level = logging.DEBUG
        log_file = 'logs/staging.log'
    else:  # development
        level = logging.DEBUG
        log_file = 'logs/dev.log'

    logging.basicConfig(
        level=level,
        format='%(asctime)s | %(levelname)-8s | %(name)s | %(message)s',
        handlers=[
            logging.StreamHandler(),
            logging.FileHandler(log_file)
        ]
    )

configure_logging()
logger = logging.getLogger(__name__)
```

## Best Practices

### Use Lazy Formatting

```python
# Bad: String formatting happens before logging
logger.debug(f"Processing item {item_id}")  # Always creates string
logger.info("User: " + username)  # Always concatenates

# Good: Formatting only happens if message is logged
logger.debug("Processing item %s", item_id)
logger.info("User: %s", username)

# Why it matters:
if logger.isEnabledFor(logging.DEBUG):
    # With f-strings: Already formatted
    # With %s: Only formats if level allows
    pass
```

### Include Context

```python
# Bad: Not enough context
logger.error("Failed to process")
logger.error("Invalid data")

# Good: Rich context
logger.error(
    "Failed to process order %s for user %s: %s",
    order_id, user_id, error_message
)
logger.error(
    "Invalid email format received: %s (user_id=%s)",
    email, user_id
)

# Even better: Use extra fields
logger.error(
    "Payment processing failed",
    extra={
        'order_id': order_id,
        'user_id': user_id,
        'amount': amount,
        'payment_method': method
    }
)
```

### Log Exceptions Properly

```python
# Bad: Lost exception information
try:
    result = risky_operation()
except Exception as e:
    logger.error("Operation failed: %s", str(e))

# Good: Include full traceback
try:
    result = risky_operation()
except Exception as e:
    logger.error("Operation failed", exc_info=True)

# Even better: Use logging.exception() at ERROR level
try:
    result = risky_operation()
except Exception as e:
    logger.exception("Operation failed")
    # Automatically includes exc_info=True

# For non-ERROR levels with traceback
try:
    result = optional_operation()
except Exception as e:
    logger.warning("Optional operation failed", exc_info=True)
```

### Don't Log Sensitive Data

```python
# Bad: Logging sensitive information
logger.info("User login: %s, password: %s", username, password)
logger.debug("Credit card: %s, CVV: %s", card_number, cvv)
logger.info("API key: %s", api_key)

# Good: Mask or omit sensitive data
logger.info("User login: %s", username)
logger.debug("Credit card ending in %s", card_number[-4:])
logger.info("API key starting with %s", api_key[:8])

# Create filtered loggers for sensitive modules
class SensitiveDataFilter(logging.Filter):
    """Filter out sensitive data from logs."""

    def filter(self, record):
        # Remove password fields
        if hasattr(record, 'password'):
            record.password = '***REDACTED***'
        return True

logger.addFilter(SensitiveDataFilter())
```

### Control Log Volume

```python
# Bad: Excessive logging
for i in range(1000000):
    logger.debug("Processing item %d", i)

# Good: Log at intervals
total = len(items)
for i, item in enumerate(items):
    process(item)
    if i % 1000 == 0:
        logger.info("Progress: %d/%d items processed", i, total)

logger.info("Completed processing %d items", total)

# Good: Aggregate information
errors = []
for item in items:
    try:
        process(item)
    except Exception as e:
        errors.append((item.id, str(e)))

if errors:
    logger.error("Failed to process %d items: %s", len(errors), errors[:5])
```

### Use Appropriate Naming

```python
# Bad: Generic logger names
logger = logging.getLogger('main')
logger = logging.getLogger('utils')

# Good: Use __name__ for automatic naming
logger = logging.getLogger(__name__)
# In myapp/services/payment.py -> 'myapp.services.payment'

# Good: Consistent naming across application
# myapp.web.views
# myapp.web.middleware
# myapp.services.email
# myapp.database.connection
```

### Configure Once, Use Everywhere

```python
# config/logging_config.py
import logging.config

def setup_logging():
    """Configure logging once at application startup."""
    LOGGING_CONFIG = {
        # ... configuration ...
    }
    logging.config.dictConfig(LOGGING_CONFIG)

# main.py
from config.logging_config import setup_logging

def main():
    # Configure logging once
    setup_logging()

    # Use loggers throughout application
    logger = logging.getLogger(__name__)
    logger.info("Application started")

    # Other modules just create loggers
    from myapp.services import process_data
    process_data()  # Uses its own logger

if __name__ == '__main__':
    main()

# myapp/services.py
import logging

logger = logging.getLogger(__name__)

def process_data():
    logger.info("Processing data")
    # Automatically uses configuration from main
```

## Structured Logging

### JSON Logging

```python
import logging
import json
from datetime import datetime

class JSONFormatter(logging.Formatter):
    """Format log records as JSON."""

    def format(self, record):
        log_data = {
            'timestamp': datetime.utcnow().isoformat(),
            'level': record.levelname,
            'logger': record.name,
            'message': record.getMessage(),
            'module': record.module,
            'function': record.funcName,
            'line': record.lineno,
        }

        # Include exception info if present
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)

        # Include extra fields
        if hasattr(record, 'user_id'):
            log_data['user_id'] = record.user_id
        if hasattr(record, 'request_id'):
            log_data['request_id'] = record.request_id

        return json.dumps(log_data)

# Usage
logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)

logger.info("User logged in", extra={'user_id': 12345, 'ip': '192.168.1.1'})
```

### Structured Logging Libraries

```python
# Using structlog (third-party library)
import structlog

# Configure structlog
structlog.configure(
    processors=[
        structlog.stdlib.add_log_level,
        structlog.stdlib.add_logger_name,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer()
    ]
)

logger = structlog.get_logger()

# Use structured logging
logger.info("user.login", user_id=12345, ip="192.168.1.1")
logger.error("payment.failed", order_id=67890, amount=99.99, reason="timeout")

# Output (JSON):
# {"event": "user.login", "user_id": 12345, "ip": "192.168.1.1", "timestamp": "2026-02-24T10:30:45Z", "level": "info"}
```

### Benefits of Structured Logging

```python
# Traditional logging: Hard to parse
logger.info("User 12345 from 192.168.1.1 logged in at 2026-02-24")

# Structured logging: Easy to parse and query
logger.info("user.login", extra={
    'user_id': 12345,
    'ip': '192.168.1.1',
    'timestamp': '2026-02-24T10:30:45Z'
})

# Benefits:
# 1. Machine-readable format
# 2. Easy to search and filter
# 3. Works with log aggregation tools (ELK, Splunk, Datadog)
# 4. Consistent field names
# 5. Type preservation (numbers stay numbers)
```

## Logging in Different Contexts

### In Applications

```python
# main.py
import logging

def main():
    # Configure logging at application entry point
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )

    logger = logging.getLogger(__name__)
    logger.info("Application started")

    try:
        run_application()
    except Exception as e:
        logger.critical("Application crashed", exc_info=True)
        raise

if __name__ == '__main__':
    main()
```

### In Libraries

```python
# mylib/core.py

# Bad: Configuring logging in a library
import logging
logging.basicConfig(level=logging.INFO)  # Don't do this!

# Good: Just create and use loggers
import logging

logger = logging.getLogger(__name__)

def library_function():
    """Library function that logs but doesn't configure."""
    logger.debug("Library function called")
    # ... function logic ...
    logger.info("Library function completed")

# Let the application configure logging
```

### In Web Applications

```python
# Flask example
from flask import Flask, request
import logging

app = Flask(__name__)

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

@app.before_request
def log_request():
    """Log each incoming request."""
    logger.info(
        "Request: %s %s",
        request.method,
        request.path,
        extra={
            'ip': request.remote_addr,
            'user_agent': request.user_agent.string
        }
    )

@app.route('/api/users')
def get_users():
    logger.info("Fetching users")
    try:
        users = fetch_users()
        return {'users': users}
    except Exception as e:
        logger.error("Failed to fetch users", exc_info=True)
        return {'error': 'Internal server error'}, 500
```

### In Tests

```python
import logging
import pytest

def test_logging_output(caplog):
    """Test that function produces expected log messages."""

    with caplog.at_level(logging.INFO):
        my_function()

    # Assert log messages
    assert "Expected message" in caplog.text
    assert any(record.levelname == "ERROR" for record in caplog.records)

def test_with_logging_disabled():
    """Test with logging temporarily disabled."""

    # Disable all logging
    logging.disable(logging.CRITICAL)

    try:
        my_function()  # Won't produce log output
    finally:
        logging.disable(logging.NOTSET)  # Re-enable

@pytest.fixture
def configure_test_logging():
    """Configure logging for tests."""
    logging.basicConfig(
        level=logging.DEBUG,
        format='%(levelname)s - %(message)s'
    )
```

### In Scripts

```python
#!/usr/bin/env python3
"""
Batch processing script with logging.
"""
import logging
import sys

def setup_logging(verbose=False):
    """Configure logging for script."""
    level = logging.DEBUG if verbose else logging.INFO

    logging.basicConfig(
        level=level,
        format='%(asctime)s - %(levelname)s - %(message)s',
        handlers=[
            logging.StreamHandler(sys.stdout),
            logging.FileHandler('script.log')
        ]
    )

def main():
    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument('-v', '--verbose', action='store_true')
    args = parser.parse_args()

    setup_logging(args.verbose)
    logger = logging.getLogger(__name__)

    logger.info("Script started")

    # Script logic
    try:
        process_data()
        logger.info("Script completed successfully")
    except Exception as e:
        logger.error("Script failed", exc_info=True)
        sys.exit(1)

if __name__ == '__main__':
    main()
```

## Common Pitfalls

### Pitfall 1: Multiple basicConfig Calls

```python
# Bad: basicConfig only works once
import logging

logging.basicConfig(level=logging.INFO)
logging.basicConfig(level=logging.DEBUG)  # Ignored!

# Good: Use force parameter (Python 3.8+)
logging.basicConfig(level=logging.INFO)
logging.basicConfig(level=logging.DEBUG, force=True)

# Better: Use dictConfig for complex setups
import logging.config
logging.config.dictConfig(LOGGING_CONFIG)
```

### Pitfall 2: Handler Duplication

```python
# Bad: Adding handlers multiple times
def setup_logging():
    logger = logging.getLogger('myapp')
    handler = logging.StreamHandler()
    logger.addHandler(handler)

setup_logging()
setup_logging()  # Adds handler again - duplicate logs!

# Good: Check for existing handlers
def setup_logging():
    logger = logging.getLogger('myapp')

    if not logger.handlers:
        handler = logging.StreamHandler()
        logger.addHandler(handler)

# Better: Clear handlers before adding
def setup_logging():
    logger = logging.getLogger('myapp')
    logger.handlers = []  # Clear existing

    handler = logging.StreamHandler()
    logger.addHandler(handler)
```

### Pitfall 3: Not Using propagate Correctly

```python
# Problem: Duplicate messages due to propagation
root_logger = logging.getLogger()
root_logger.addHandler(console_handler)

child_logger = logging.getLogger('myapp')
child_logger.addHandler(file_handler)

child_logger.info("Test")  # Appears in both console and file!

# Solution: Set propagate=False
child_logger = logging.getLogger('myapp')
child_logger.addHandler(file_handler)
child_logger.propagate = False  # Don't pass to parent

child_logger.info("Test")  # Only appears in file
```

### Pitfall 4: Using F-strings

```python
# Bad: F-strings always evaluate
logger.debug(f"Value: {expensive_function()}")
# expensive_function() is called even if debug is disabled!

# Good: Lazy % formatting
logger.debug("Value: %s", expensive_function())
# expensive_function() only called if debug is enabled

# Alternative: Check level first
if logger.isEnabledFor(logging.DEBUG):
    logger.debug(f"Value: {expensive_function()}")
```

### Pitfall 5: Logging in Loops

```python
# Bad: Millions of log messages
for item in huge_list:
    logger.info("Processing %s", item.id)

# Good: Log progress at intervals
total = len(huge_list)
for i, item in enumerate(huge_list):
    process(item)
    if i % 100 == 0:
        logger.info("Progress: %d/%d (%.1f%%)", i, total, i/total*100)

logger.info("Completed processing %d items", total)
```

### Pitfall 6: Missing exc_info

```python
# Bad: Lost exception details
try:
    risky_operation()
except Exception as e:
    logger.error("Failed: %s", e)  # Only shows exception message

# Good: Include full traceback
try:
    risky_operation()
except Exception as e:
    logger.error("Failed", exc_info=True)
    # or
    logger.exception("Failed")  # Automatically includes exc_info
```

## Summary

Logging best practices:

**Basic principles:**

- Use logging module, not print statements
- Choose appropriate log levels
- Use lazy % formatting, not f-strings
- Log exceptions with exc_info=True

**Configuration:**

- Configure once at application startup
- Use named loggers with `__name__`
- dictConfig for complex setups
- Multiple handlers for different destinations

**Message quality:**

- Include context and relevant data
- Don't log sensitive information
- Be mindful of log volume
- Use structured logging for production

**Architecture:**

- Logger → Handler → Formatter
- Handler per destination
- Formatter per output format
- Set levels at both logger and handler

**Common patterns:**

- Rotating file handlers for production
- JSON formatting for log aggregation
- Environment-based configuration
- Separate error logs

Effective logging is essential for debugging, monitoring, and maintaining production systems.

## Next Steps

- Organize with [Code Organization](code-organization.md)
- Document with [Documentation](documentation.md)
- Quality check with [Writing Clean Code](writing-clean-code.md)
