# Python Code Maturity Progression

## Purpose

This guide outlines the natural progression from a basic Python script to production-grade, enterprise-ready software. It consolidates 15 developmental stages into **7 practical phases**, providing both conceptual understanding and concrete implementation examples.

**Key insight**: Production readiness is not a single milestone but a progression. Most teams don't need all 7 phases—a startup might be perfectly fine at Phase 3-4. The "right" phase depends on your scale, team size, regulatory requirements, and business criticality.

**Important**: Move up phases incrementally as pain points emerge. Don't try to jump from Phase 1 to Phase 6—you'll spend months on infrastructure before delivering value. Build what you need today, and level up when necessary.

## The 7 Phases

Each phase builds upon the previous, adding capabilities that become essential as your code grows from personal scripts to production systems.

| Phase       | Focus                       | When You Need This                                      |
| ----------- | --------------------------- | ------------------------------------------------------- |
| **Phase 1** | Foundational Code           | Learning Python, rapid prototyping, one-off scripts     |
| **Phase 2** | Modular & Installable       | Code you'll maintain, share with team, or reuse         |
| **Phase 3** | Quality Engineering         | Code that needs reliability, team collaboration         |
| **Phase 4** | Production-Ready Config     | Deploying to multiple environments, real users          |
| **Phase 5** | Performance & Scale         | High traffic, performance critical, production loads    |
| **Phase 6** | Infrastructure & Deployment | Multiple deployments, zero-downtime updates, automation |
| **Phase 7** | Operations & Enterprise     | Mission-critical systems, compliance, multi-region      |

## Complete Example: URL Shortener Service

Throughout this guide, we'll build a **URL Shortener with Analytics**—progressing from a basic script to an enterprise-grade system. This example is ideal because it:

- Starts simple (generate short codes, redirect)
- Adds complexity naturally (analytics, users, API)
- Has real production concerns (performance, scale, monitoring)
- Touches databases, APIs, async operations, caching

## Phase 1: Foundational Code

**Covers: Basic Script → Structured Code**

### Characteristics

**Basic Script (Level 1)**

- Single `.py` file that runs
- Hardcoded values throughout
- No error handling
- Manual execution
- Works "on my machine"

**Structured Code (Level 2)**

- Multiple functions for organization
- Basic command-line arguments (argparse)
- Some error handling (try/except)
- `if __name__ == "__main__":` guard
- README with basic instructions

### When to Use Phase 1

- Learning Python fundamentals
- Quick prototypes and experiments
- One-off data analysis scripts
- Personal automation tasks

### Implementation: Basic Script

```python
# url_shortener.py

import hashlib

urls = {}

def shorten_url(long_url):

       short_code = hashlib.md5(long_url.encode()).hexdigest()[:6]
       urls[short_code] = long_url

       return short_code

def get_long_url(short_code):
       return urls.get(short_code)

# Test it
short = shorten_url("https://www.example.com/very/long/url")
print(f"Short code: {short}")
print(f"Original: {get_long_url(short)}")
```

**Run**: `python url_shortener.py`

### Implementation: Structured Code

```python
# url_shortener.py

import hashlib
import argparse
import sys

urls = {}

def shorten_url(long_url):
       """Generate short code for a URL"""
       if not long_url.startswith(('http://', 'https://')):
           raise ValueError("URL must start with http:// or https://")

       short_code = hashlib.md5(long_url.encode()).hexdigest()[:6]
       urls[short_code] = long_url

       return short_code

def get_long_url(short_code):
       """Retrieve original URL from short code"""
       url = urls.get(short_code)

       if not url:
           raise KeyError(f"Short code '{short_code}' not found")
       return url

def main():

       parser = argparse.ArgumentParser(description='URL Shortener')
       parser.add_argument('--shorten', help='URL to shorten')
       parser.add_argument('--expand', help='Short code to expand')
       args = parser.parse_args()

       try:
           if args.shorten:
               short_code = shorten_url(args.shorten)
               print(f"Short code: {short_code}")
           elif args.expand:
               long_url = get_long_url(args.expand)
               print(f"Original URL: {long_url}")
           else:
               parser.print_help()
       except (ValueError, KeyError) as e:
           print(f"Error: {e}", file=sys.stderr)
           sys.exit(1)

if __name__ == "__main__":
       main()
```

**Run**: `python url_shortener.py --shorten https://example.com`

### Transitioning to Phase 2

**Signs you need Phase 2**:

- You're copying code between scripts
- Multiple people need to use your code
- Data doesn't persist between runs
- Configuration changes require editing code
- You have no logs to debug issues

## Phase 2: Modular & Installable

**Covers: Modular Package → Installable Package**

### Characteristics

**Modular Package (Level 3)**

- Organized into modules/packages (`src/` directory structure)
- Configuration separated from code (config files or environment variables)
- `requirements.txt` for dependencies
- Basic logging instead of print statements
- Input validation and error messages

**Installable Package (Level 4)**

- `setup.py` or `pyproject.toml` for installation
- Entry points for CLI commands
- Virtual environment usage
- `.gitignore` for Python projects
- Basic documentation (docstrings)
- Type hints on functions

### When to Use Phase 2

- Code you'll maintain long-term
- Sharing code with teammates
- Multiple related scripts to organize
- Need dependency management

### Implementation: Modular Package

**Structure**:

```
url-shortener/
├── README.md
├── requirements.txt
├── config.ini
└── shortener/
       ├── __init__.py
       ├── core.py
       ├── storage.py
       └── cli.py
```

**requirements.txt**:

```
# No external dependencies yet
```

**config.ini**:

```ini
[DEFAULT]
storage_file = urls.json
base_url = http://localhost:8000
```

**shortener/storage.py**:

```python
import json
import os
from typing import Optional

class URLStorage:
       def __init__(self, filepath: str):
           self.filepath = filepath
           self.urls = self._load()

       def _load(self) -> dict:
           if os.path.exists(self.filepath):
               with open(self.filepath, 'r') as f:
                   return json.load(f)
           return {}

       def _save(self):
           with open(self.filepath, 'w') as f:
               json.dump(self.urls, f, indent=2)

       def add(self, short_code: str, long_url: str):
           self.urls[short_code] = long_url
           self._save()

       def get(self, short_code: str) -> Optional[str]:
           return self.urls.get(short_code)

       def exists(self, short_code: str) -> bool:
           return short_code in self.urls
```

**shortener/core.py**:

```python
import hashlib
import logging

from .storage import URLStorage

logger = logging.getLogger(__name__)

class URLShortener:

       def __init__(self, storage: URLStorage):

           self.storage = storage

       def shorten(self, long_url: str) -> str:
           if not long_url.startswith(('http://', 'https://')):
               raise ValueError("URL must start with http:// or https://")
           short_code = hashlib.md5(long_url.encode()).hexdigest()[:6]

           if self.storage.exists(short_code):
               logger.info(f"Short code {short_code} already exists")
           else:
               self.storage.add(short_code, long_url)
               logger.info(f"Created short code {short_code} for {long_url}")

           return short_code



       def expand(self, short_code: str) -> str:
           url = self.storage.get(short_code)
           if not url:
               raise KeyError(f"Short code '{short_code}' not found")
           return url
```

**shortener/cli.py**:

```python
import argparse
import logging
import configparser

from .core import URLShortener
from .storage import URLStorage

def setup_logging():
       logging.basicConfig(
           level=logging.INFO,
           format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
       )

def load_config():
       config = configparser.ConfigParser()
       config.read('config.ini')
       return config['DEFAULT']

def main():
       setup_logging()
       config = load_config()
       storage = URLStorage(config['storage_file'])
       shortener = URLShortener(storage)

       parser = argparse.ArgumentParser(description='URL Shortener')
       parser.add_argument('--shorten', help='URL to shorten')
       parser.add_argument('--expand', help='Short code to expand')

       args = parser.parse_args()

       try:
           if args.shorten:
               short_code = shortener.shorten(args.shorten)
               print(f"{config['base_url']}/{short_code}")
           elif args.expand:
               long_url = shortener.expand(args.expand)
               print(f"Original URL: {long_url}")
           else:
               parser.print_help()

       except (ValueError, KeyError) as e:
           logging.error(str(e))
           exit(1)
```

**Run**: `python -m shortener.cli --shorten https://example.com`

### Implementation: Installable Package

**Add pyproject.toml**:

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "url-shortener"
version = "0.1.0"
description = "A simple URL shortener"
authors = [{name = "Your Name", email = "you@example.com"}]
requires-python = ">=3.8"
dependencies = []

[project.scripts]
shorten = "shortener.cli:main"
```

**Install**: `pip install -e .`

**Run**: `shorten --shorten https://example.com`

### Transitioning to Phase 3

**Signs you need Phase 3**:

- Bugs are hard to track down
- Unsure if changes break existing functionality
- Code style inconsistencies across team
- No confidence in refactoring
- Production bugs could have been caught earlier

## Phase 3: Quality Engineering

**Covers: Tested Code → Quality Assurance → Robust Error Handling**

### Characteristics

**Tested Code (Level 5)**

- Unit tests with pytest
- Test coverage measurement (pytest-cov)
- Integration tests for main workflows
- Test fixtures and mocks for external dependencies
- CI/CD runs tests automatically (GitHub Actions, GitLab CI)

**Quality Assurance (Level 6)**

- Code formatting (Black, Ruff)
- Linting (pylint, flake8, Ruff)
- Static type checking (mypy)
- Pre-commit hooks enforce standards
- Code review process
- Dependency security scanning (Safety, Bandit)

**Robust Error Handling (Level 7)**

- Custom exception classes
- Comprehensive error handling with context
- Graceful degradation
- Retry logic with exponential backoff
- Circuit breakers for external services
- Structured logging with context (structlog)

### When to Use Phase 3

- Code used by others or in production
- Team collaboration requiring standards
- Need confidence in refactoring
- Debugging production issues
- Want to catch bugs before users do

### Implementation: Tested Code

**Structure**:

```
url-shortener/
├── tests/
│   ├── __init__.py
│   ├── test_core.py
│   └── test_storage.py
└── ...
```

**Update requirements.txt**:

```
pytest
pytest-cov
```

**tests/test_core.py**:

```python
import pytest
from shortener.core import URLShortener
from shortener.storage import URLStorage
import tempfile
import os

@pytest.fixture
def temp_storage():
       fd, path = tempfile.mkstemp(suffix='.json')
       os.close(fd)
       storage = URLStorage(path)
       yield storage
       os.unlink(path)

@pytest.fixture
def shortener(temp_storage):
       return URLShortener(temp_storage)

def test_shorten_url(shortener):
       url = "https://example.com"
       short_code = shortener.shorten(url)

       assert len(short_code) == 6
       assert shortener.expand(short_code) == url


def test_shorten_invalid_url(shortener):
       with pytest.raises(ValueError):
           shortener.shorten("not-a-url")

def test_expand_nonexistent(shortener):
       with pytest.raises(KeyError):
           shortener.expand("abc123")

def test_shorten_idempotent(shortener):
       url = "https://example.com"
       code1 = shortener.shorten(url)
       code2 = shortener.shorten(url)

       assert code1 == code2
```

**tests/test_storage.py**:

```python
import pytest
from shortener.storage import URLStorage
import tempfile
import os

@pytest.fixture
def temp_file():
       fd, path = tempfile.mkstemp(suffix='.json')
       os.close(fd)
       yield path
       if os.path.exists(path):
           os.unlink(path)

def test_storage_add_and_get(temp_file):
       storage = URLStorage(temp_file)
       storage.add("abc123", "https://example.com")

       assert storage.get("abc123") == "https://example.com"

def test_storage_persistence(temp_file):
       storage1 = URLStorage(temp_file)
       storage1.add("abc123", "https://example.com")
       storage2 = URLStorage(temp_file)

       assert storage2.get("abc123") == "https://example.com"

def test_storage_get_nonexistent(temp_file):
       storage = URLStorage(temp_file)

       assert storage.get("xyz789") is None
```

**Run tests**: `pytest --cov=shortener tests/`

### Implementation: Quality Assurance

**Update requirements.txt**:

```
pytest
pytest-cov
black
ruff
mypy
pre-commit
```

**.pre-commit-config.yaml**:

```yaml
repos:
  - repo: https://github.com/psf/black
    rev: 23.12.1
    hooks:
      - id: black
        language_version: python3

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9
    hooks:
      - id: ruff
        args: [--fix]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies: [types-all]
```

**Update pyproject.toml** (add):

```toml
[tool.black]
line-length = 88
[tool.ruff]
line-length = 88
select = ["E", "F", "I", "N", "W"]

[tool.mypy]
python_version = "3.8"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true
```

**Update code with type hints** (shortener/core.py):

```python
import hashlib
import logging
from typing import Optional

from .storage import URLStorage

logger = logging.getLogger(__name__)

class URLShortener:
       def __init__(self, storage: URLStorage) -> None:
           self.storage = storage

       def shorten(self, long_url: str) -> str:
           if not long_url.startswith(('http://', 'https://')):
               raise ValueError("URL must start with http:// or https://")

           short_code = hashlib.md5(long_url.encode()).hexdigest()[:6]

           if self.storage.exists(short_code):
               logger.info(f"Short code {short_code} already exists")
           else:
               self.storage.add(short_code, long_url)
               logger.info(f"Created short code {short_code} for {long_url}")

           return short_code

       def expand(self, short_code: str) -> str:
           url = self.storage.get(short_code)
           if not url:
               raise KeyError(f"Short code '{short_code}' not found")
           return url
```

**Setup**: `pre-commit install`

**Run**: `black .`, `ruff check .`, `mypy shortener/`

### Implementation: Robust Error Handling & Web API

**Update requirements.txt**:

```
flask
python-dotenv
structlog
tenacity
```

**Create .env**:

```
FLASK_ENV=development
STORAGE_FILE=urls.json
BASE_URL=http://localhost:5000
```

**shortener/exceptions.py**:

```python
class ShortenerError(Exception):
       """Base exception for shortener"""
       pass

class InvalidURLError(ShortenerError):
       """Raised when URL is invalid"""
       pass

class ShortCodeNotFoundError(ShortenerError):
       """Raised when short code doesn't exist"""
       pass

class StorageError(ShortenerError):
       """Raised when storage operation fails"""
       pass
```

**shortener/api.py**:

```python
from flask import Flask, request, jsonify, redirect
from typing import Tuple
import structlog
import os
from dotenv import load_dotenv

from .core import URLShortener
from .storage import URLStorage
from .exceptions import InvalidURLError, ShortCodeNotFoundError

load_dotenv()

logger = structlog.get_logger()

app = Flask(__name__)

# Initialize storage and shortener
storage = URLStorage(os.getenv('STORAGE_FILE', 'urls.json'))
shortener = URLShortener(storage)

@app.route('/api/shorten', methods=['POST'])
def api_shorten() -> Tuple[dict, int]:
       try:
           data = request.get_json()
           if not data or 'url' not in data:
               return jsonify({'error': 'Missing url parameter'}), 400

           long_url = data['url']
           short_code = shortener.shorten(long_url)

           base_url = os.getenv('BASE_URL', 'http://localhost:5000')
           short_url = f"{base_url}/{short_code}"

           logger.info("url_shortened", long_url=long_url, short_code=short_code)
           return jsonify({
               'short_code': short_code,
               'short_url': short_url,
               'long_url': long_url
           }), 201

       except InvalidURLError as e:
           logger.warning("invalid_url", error=str(e))
           return jsonify({'error': str(e)}), 400

       except Exception as e:
           logger.error("shorten_failed", error=str(e))
           return jsonify({'error': 'Internal server error'}), 500

@app.route('/<short_code>')
def redirect_url(short_code: str):
       try:
           long_url = shortener.expand(short_code)
           logger.info("url_redirected", short_code=short_code, long_url=long_url)

           return redirect(long_url, code=302)

       except ShortCodeNotFoundError:
           logger.warning("short_code_not_found", short_code=short_code)

           return jsonify({'error': 'Short code not found'}), 404

       except Exception as e:
           logger.error("redirect_failed", short_code=short_code, error=str(e))

           return jsonify({'error': 'Internal server error'}), 500

@app.route('/health')
def health():
       return jsonify({'status': 'healthy'}), 200


if __name__ == '__main__':
       app.run(debug=True)
```

**Run**: `python -m shortener.api`

**Test**: `curl -X POST http://localhost:5000/api/shorten -H "Content-Type: application/json" -d '{"url":"https://example.com"}'`

### Transitioning to Phase 4

**Signs you need Phase 4**:

- Deploying to multiple environments (dev, staging, prod)
- Configuration scattered across code and env vars
- No visibility into production behavior
- Can't debug production issues effectively
- Secrets hardcoded or insecurely managed

## Phase 4: Production-Ready Configuration

**Covers: Configuration Management → Observability**

### Characteristics

**Configuration Management (Level 8)**

- Environment-based configs (dev, staging, prod)
- Secret management (not in code or git)
- Feature flags for gradual rollouts
- Configuration validation (pydantic)
- `.env` files with python-dotenv

**Observability (Level 9)**

- Structured logging to centralized system (ELK, CloudWatch)
- Metrics collection (Prometheus, StatsD)
- Distributed tracing (OpenTelemetry)
- Health check endpoints
- Performance profiling instrumentation
- Alerting on critical errors

### When to Use Phase 4

- Deploying to production environments
- Multiple environments to manage
- Need to debug production issues
- Want to understand system behavior
- Compliance requires audit trails

### Implementation: Configuration Management

**Update requirements.txt**:

```
pydantic
pydantic-settings
```

**shortener/config.py**:

```python
from pydantic_settings import BaseSettings
from pydantic import Field, validator

class Settings(BaseSettings):
       # App settings
       environment: str = Field(default="development")
       debug: bool = Field(default=False)

       # Storage settings
       storage_file: str = Field(default="urls.json")

       # URL settings
       base_url: str = Field(default="http://localhost:5000")
       short_code_length: int = Field(default=6, ge=4, le=10)

       # Database settings (for future use)
       database_url: str = Field(default="sqlite:///urls.db")

       # Redis settings (for future use)
       redis_url: str = Field(default="redis://localhost:6379/0")

       # API settings
       rate_limit: int = Field(default=100)

       class Config:
           env_file = ".env"
           case_sensitive = False

       @validator('base_url')
       def validate_base_url(cls, v):
           if not v.startswith(('http://', 'https://')):
               raise ValueError('base_url must start with http:// or https://')

           return v.rstrip('/')

settings = Settings()
```

**Environment-specific configs**:

**.env.development**:

```
ENVIRONMENT=development
DEBUG=true
STORAGE_FILE=urls_dev.json
BASE_URL=http://localhost:5000
```

**.env.production**:

```
ENVIRONMENT=production
DEBUG=false
STORAGE_FILE=/var/data/urls.json
BASE_URL=https://short.example.com
DATABASE_URL=postgresql://user:pass@db:5432/urls
REDIS_URL=redis://redis:6379/0
```

**Update api.py to use settings**:

```python
from .config import settings

storage = URLStorage(settings.storage_file)
# ... rest of code uses settings.*
```

### Implementation: Observability

**Update requirements.txt**:

```
prometheus-client
opentelemetry-api
opentelemetry-sdk
opentelemetry-instrumentation-flask
```

**shortener/metrics.py**:

```python
from prometheus_client import Counter, Histogram, Gauge
import time

# Metrics
url_shortened_total = Counter(
       'url_shortened_total',
       'Total number of URLs shortened'
)

url_redirected_total = Counter(
       'url_redirected_total',
       'Total number of redirects'
)

url_not_found_total = Counter(
       'url_not_found_total',
       'Total number of 404s'
)

request_duration = Histogram(
       'request_duration_seconds',
       'Request duration in seconds',
       ['method', 'endpoint']
)

active_urls = Gauge(
       'active_urls_total',
       'Total number of active short URLs'
)
```

**Update api.py with observability**:

```python
from flask import Flask, request, jsonify, redirect
from prometheus_client import generate_latest, CONTENT_TYPE_LATEST
import structlog
import time
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, BatchSpanProcessor

from .metrics import (
       url_shortened_total,
       url_redirected_total,
       url_not_found_total,
       request_duration,
       active_urls
)

# Setup tracing
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)
trace.get_tracer_provider().add_span_processor(
       BatchSpanProcessor(ConsoleSpanExporter())
)

# ... existing code ...

@app.route('/api/shorten', methods=['POST'])
def api_shorten():
       with tracer.start_as_current_span("shorten_url"):
           start_time = time.time()
           try:
               # ... existing code ...
               url_shortened_total.inc()
               active_urls.set(len(storage.urls))
               return result, 201
           finally:
               request_duration.labels(method='POST', endpoint='/api/shorten').observe(
                   time.time() - start_time
               )

@app.route('/metrics')
def metrics():
       return generate_latest(), 200, {'Content-Type': CONTENT_TYPE_LATEST}

# ... rest of code ...
```

### Transitioning to Phase 5

**Signs you need Phase 5**:

- Response times too slow for users
- System can't handle current traffic
- Database is a bottleneck
- Need to scale horizontally
- Want consistent deployments across environments

## Phase 5: Performance & Scale

**Covers: Scalability & Performance → Containerization**

### Characteristics

**Scalability & Performance (Level 10)**

- Async/await for I/O-bound operations
- Connection pooling for databases
- Caching strategies (Redis, in-memory)
- Background job processing (Celery, RQ)
- Database query optimization
- Resource limits and rate limiting

**Containerization (Level 11)**

- Dockerfile with multi-stage builds
- Docker Compose for local development
- Optimized image size (Alpine, slim variants)
- `.dockerignore` file
- Container security scanning
- Non-root user in container

### When to Use Phase 5

- High traffic or performance-critical
- Need to scale horizontally
- Multiple services to coordinate
- Want consistent dev/prod environments
- Deploying to cloud or Kubernetes

### Implementation: Database & Caching

**Update requirements.txt**:

```
sqlalchemy
redis
aiohttp
asyncio
```

**shortener/models.py**:

```python
from sqlalchemy import Column, String, Integer, DateTime, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from datetime import datetime

Base = declarative_base()

class URL(Base):
       __tablename__ = 'urls'

       id = Column(Integer, primary_key=True)
       short_code = Column(String(10), unique=True, index=True, nullable=False)
       long_url = Column(String(2048), nullable=False)
       created_at = Column(DateTime, default=datetime.utcnow)
       click_count = Column(Integer, default=0)

       def __repr__(self):
           return f"<URL(short_code='{self.short_code}', long_url='{self.long_url}')>"
```

**shortener/database.py**:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session
from contextlib import contextmanager

from .models import Base, URL
from .config import settings
import redis

engine = create_engine(settings.database_url, echo=settings.debug)
SessionLocal = sessionmaker(bind=engine)

# Redis client
redis_client = redis.from_url(settings.redis_url, decode_responses=True)

def init_db():
       Base.metadata.create_all(bind=engine)

@contextmanager
def get_db() -> Session:
       db = SessionLocal()
       try:
           yield db
           db.commit()
       except Exception:
           db.rollback()
           raise
       finally:
           db.close()
```

**shortener/service.py**:

```python
from typing import Optional
import hashlib
import structlog

from .database import get_db, redis_client
from .models import URL
from .exceptions import InvalidURLError, ShortCodeNotFoundError
from .config import settings

logger = structlog.get_logger()

class URLService:
       def __init__(self):
           self.cache_ttl = 3600  # 1 hour

       def _generate_short_code(self, long_url: str) -> str:
           return hashlib.md5(long_url.encode()).hexdigest()[:settings.short_code_length]

       def shorten(self, long_url: str) -> str:
           if not long_url.startswith(('http://', 'https://')):
               raise InvalidURLError("URL must start with http:// or https://")

           short_code = self._generate_short_code(long_url)

           # Check cache first
           cached = redis_client.get(f"url:{short_code}")
           if cached:
               logger.info("cache_hit", short_code=short_code)
               return short_code

           # Check database
           with get_db() as db:
               existing = db.query(URL).filter(URL.short_code == short_code).first()

               if existing:
                   # Update cache
                   redis_client.setex(
                       f"url:{short_code}",
                       self.cache_ttl,
                       existing.long_url
                   )
                   return short_code

               # Create new
               url = URL(short_code=short_code, long_url=long_url)
               db.add(url)
               db.commit()

               # Cache it
               redis_client.setex(
                   f"url:{short_code}",
                   self.cache_ttl,
                   long_url
               )

               logger.info("url_created", short_code=short_code, long_url=long_url)
               return short_code

       def expand(self, short_code: str) -> str:
           # Check cache
           cached = redis_client.get(f"url:{short_code}")

           if cached:
               logger.info("cache_hit", short_code=short_code)
               self._increment_clicks(short_code)
               return cached

           # Check database
           with get_db() as db:
               url = db.query(URL).filter(URL.short_code == short_code).first()
               if not url:
                   raise ShortCodeNotFoundError(f"Short code '{short_code}' not found")

               # Cache it
               redis_client.setex(
                   f"url:{short_code}",
                   self.cache_ttl,
                   url.long_url
               )

               self._increment_clicks(short_code)
               return url.long_url

       def _increment_clicks(self, short_code: str):
           with get_db() as db:
               url = db.query(URL).filter(URL.short_code == short_code).first()
               if url:
                   url.click_count += 1
                   db.commit()
```

### Implementation: Containerization

**Dockerfile**:

```dockerfile
# Multi-stage build
FROM python:3.11-slim as builder
WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Final stage
FROM python:3.11-slim

# Create non-root user
RUN useradd -m -u 1000 appuser
WORKDIR /app

# Copy dependencies from builder
COPY --from=builder /root/.local /home/appuser/.local

# Copy application
COPY --chown=appuser:appuser . .

# Make sure scripts in .local are usable
ENV PATH=/home/appuser/.local/bin:$PATH
USER appuser
EXPOSE 5000
CMD ["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "shortener.api:app"]
```

**.dockerignore**:

```
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env/
venv/
.git
.gitignore
.pytest_cache
.coverage
*.db
*.json
.env
.env.*
tests/
```

**docker-compose.yml**:

```yaml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "5000:5000"
    environment:
      - DATABASE_URL=postgresql://postgres:password@db:5432/urlshortener
      - REDIS_URL=redis://redis:6379/0

    depends_on:
      - db
      - redis

    volumes:
      - ./:/app

  db:
    image: postgres:15-alpine

    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=urlshortener

    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

    ports:
      - "6379:6379"

  prometheus:
    image: prom/prometheus

    ports:
      - "9090:9090"

    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana

    ports:
      - "3000:3000"

    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

volumes:
  postgres_data:
```

**prometheus.yml**:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "url-shortener"

    static_configs:
      - targets: ["app:5000"]
```

**Run**: `docker-compose up --build`

### Transitioning to Phase 6

**Signs you need Phase 6**:

- Manual deployments are error-prone
- Need to deploy multiple times per day
- Want zero-downtime deployments
- Managing multiple environments manually
- Need infrastructure to be reproducible

## Phase 6: Infrastructure & Deployment

**Covers: Infrastructure as Code → Production Deployment**

### Characteristics

**Infrastructure as Code (Level 12)**

- Terraform or CloudFormation for resources
- Kubernetes manifests or Helm charts
- Service mesh configuration (Istio, Linkerd)
- Auto-scaling rules
- Load balancer configuration
- Database migration management (Alembic)

**Production Deployment (Level 13)**

- Blue-green or canary deployments
- Rolling updates with zero downtime
- Automated rollback on failure
- Database migration strategy
- Secrets rotation
- Backup and disaster recovery procedures

### When to Use Phase 6

- Frequent deployments needed
- Multiple production environments
- Need reproducible infrastructure
- Want automated deployment pipeline
- Zero-downtime requirement

### Implementation: Infrastructure as Code

**terraform/main.tf**:

```hcl
terraform {
     required_providers {
       aws = {
         source  = "hashicorp/aws"
         version = "~> 5.0"
       }
     }
}

provider "aws" {
     region = var.aws_region
}

# VPC
resource "aws_vpc" "main" {
     cidr_block = "10.0.0.0/16"
     tags = {
       Name = "url-shortener-vpc"
     }
}

# ECS Cluster
resource "aws_ecs_cluster" "main" {
     name = "url-shortener-cluster"
}

# RDS PostgreSQL
resource "aws_db_instance" "postgres" {
     identifier        = "url-shortener-db"
     engine            = "postgres"
     engine_version    = "15.3"
     instance_class    = "db.t3.micro"
     allocated_storage = 20

     db_name  = "urlshortener"
     username = var.db_username
     password = var.db_password

     skip_final_snapshot = true
}

# ElastiCache Redis
resource "aws_elasticache_cluster" "redis" {
     cluster_id      = "url-shortener-cache"
     engine          = "redis"
     node_type       = "cache.t3.micro"
     num_cache_nodes = 1
     port            = 6379
}

# Application Load Balancer
resource "aws_lb" "main" {
     name               = "url-shortener-alb"
     internal           = false
     load_balancer_type = "application"
     subnets            = aws_subnet.public[*].id
}

# ECS Task Definition
resource "aws_ecs_task_definition" "app" {
     family                   = "url-shortener"
     network_mode             = "awsvpc"
     requires_compatibilities = ["FARGATE"]
     cpu                      = "256"
     memory                   = "512"

     container_definitions = jsonencode([{
       name  = "url-shortener"
       image = "${var.ecr_repository}:latest"

       portMappings = [{
         containerPort = 5000
         protocol      = "tcp"
       }]

       environment = [
         {
           name  = "DATABASE_URL"
           value = "postgresql://${var.db_username}:${var.db_password}@${aws_db_instance.postgres.endpoint}/urlshortener"
         },
         {
           name  = "REDIS_URL"
           value = "redis://${aws_elasticache_cluster.redis.cache_nodes[0].address}:6379/0"
         }
       ]
     }])
}

# Auto Scaling
resource "aws_appautoscaling_target" "ecs" {
     max_capacity       = 10
     min_capacity       = 2
     resource_id        = "service/${aws_ecs_cluster.main.name}/${aws_ecs_service.app.name}"
     scalable_dimension = "ecs:service:DesiredCount"
     service_namespace  = "ecs"
}

resource "aws_appautoscaling_policy" "cpu" {
     name               = "cpu-autoscaling"
     policy_type        = "TargetTrackingScaling"
     resource_id        = aws_appautoscaling_target.ecs.resource_id
     scalable_dimension = aws_appautoscaling_target.ecs.scalable_dimension
     service_namespace  = aws_appautoscaling_target.ecs.service_namespace

     target_tracking_scaling_policy_configuration {
       predefined_metric_specification {
         predefined_metric_type = "ECSServiceAverageCPUUtilization"
       }
       target_value = 70.0
     }
}
```

**terraform/variables.tf**:

```hcl
variable "aws_region" {
     default = "us-east-1"
}

variable "db_username" {
     sensitive = true
}

variable "db_password" {
     sensitive = true
}

variable "ecr_repository" {
     description = "ECR repository URL"
}
```

**Run**: `terraform init && terraform plan && terraform apply`

### Implementation: CI/CD Pipeline

**.github/workflows/deploy.yml**:

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: "3.11"
      - name: Install dependencies
        run: |
          pip install -r requirements.txt
          pip install pytest pytest-cov
      - name: Run tests
        run: pytest --cov=shortener tests/
      - name: Run linting
        run: |
          pip install ruff black mypy
          black --check .
          ruff check .
          mypy shortener/

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build and push Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/url-shortener:$IMAGE_TAG .
          docker tag $ECR_REGISTRY/url-shortener:$IMAGE_TAG $ECR_REGISTRY/url-shortener:latest
          docker push $ECR_REGISTRY/url-shortener:$IMAGE_TAG
          docker push $ECR_REGISTRY/url-shortener:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to ECS
        run: |
          aws ecs update-service 
            --cluster url-shortener-cluster 
            --service url-shortener-service 
            --force-new-deployment
      - name: Wait for deployment
        run: |
          aws ecs wait services-stable 
            --cluster url-shortener-cluster 
            --services url-shortener-service
```

**Database Migration with Alembic**:

**alembic/versions/001_initial.py**:

```python
from alembic import op
import sqlalchemy as sa

def upgrade():
       op.create_table(
           'urls',
           sa.Column('id', sa.Integer(), nullable=False),
           sa.Column('short_code', sa.String(10), nullable=False),
           sa.Column('long_url', sa.String(2048), nullable=False),
           sa.Column('created_at', sa.DateTime(), nullable=True),
           sa.Column('click_count', sa.Integer(), nullable=True),
           sa.PrimaryKeyConstraint('id')
       )
       op.create_index('ix_urls_short_code', 'urls', ['short_code'], unique=True)

def downgrade():
       op.drop_index('ix_urls_short_code', table_name='urls')
       op.drop_table('urls')
```

### Transitioning to Phase 7

**Signs you need Phase 7**:

- System is business-critical
- Need 99.9%+ uptime SLAs
- Compliance requirements (SOC2, HIPAA, GDPR)
- Multiple regions for disaster recovery
- On-call team needs runbooks and alerts

## Phase 7: Operations & Enterprise

**Covers: Monitoring & Maintenance → Enterprise Grade**

### Characteristics

**Monitoring & Maintenance (Level 14)**

- SLA/SLO definitions with alerting
- On-call rotation and runbooks
- Incident response procedures
- Post-mortem process
- Performance regression detection
- Cost monitoring and optimization

**Enterprise Grade (Level 15)**

- Multi-region deployment
- Compliance certifications (SOC2, HIPAA, GDPR)
- Audit logging
- Role-based access control (RBAC)
- Data privacy and retention policies
- Chaos engineering (intentional failure testing)
- Comprehensive documentation (architecture diagrams, API docs, runbooks)
- Version compatibility guarantees
- Deprecation policies

### When to Use Phase 7

- Mission-critical production systems
- Enterprise customers requiring SLAs
- Compliance and regulatory requirements
- Multi-region availability needed
- Large distributed teams

### Implementation: Monitoring & Alerting

**prometheus_rules.yml**:

```yaml
groups:
  - name: url_shortener_alerts

    interval: 30s

    rules:
      - alert: HighErrorRate
        expr: rate(url_not_found_total[5m]) > 0.05
        for: 5m
        labels:
          severity: warning

        annotations:
          summary: High error rate detected

          description: Error rate is {{ $value }} errors/sec

      - alert: HighLatency
        expr: histogram_quantile(0.95, rate(request_duration_seconds_bucket[5m])) > 0.5
        for: 5m
        labels:
          severity: warning

        annotations:
          summary: High latency detected
          description: P95 latency is {{ $value }} seconds

      - alert: ServiceDown
        expr: up{job="url-shortener"} == 0
        for: 1m
        labels:
          severity: critical

        annotations:
          summary: Service is down

          description: URL shortener service is not responding
```

**RUNBOOK.md**:

```markdown
# URL Shortener Runbook

## Common Issues

### High Error Rate (404s)

**Symptoms**: Alert "HighErrorRate" firing

**Investigation**:

1. Check Grafana dashboard for spike in 404s
2. Look at logs: `kubectl logs -l app=url-shortener --tail=100`
3. Check if specific short codes are missing

**Resolution**:

- If database corruption: Restore from backup
- If cache issue: `redis-cli FLUSHDB` to clear cache
- If malicious scanning: Enable rate limiting

### High Latency

**Symptoms**: Alert "HighLatency" firing

**Investigation**:

1. Check database connection pool: Are connections exhausted?
2. Check Redis: `redis-cli INFO stats`
3. Check CPU/memory: `kubectl top pods`

**Resolution**:

- Scale up: `kubectl scale deployment url-shortener --replicas=5`
- Check slow queries in PostgreSQL
- Increase Redis memory if evicting too aggressively

### Service Down

**Symptoms**: Alert "ServiceDown" firing, health check failing

**Investigation**:

1. Check pod status: `kubectl get pods`
2. Check recent deployments: `kubectl rollout history deployment/url-shortener`
3. Check logs for crashes

**Resolution**:

- Rollback: `kubectl rollout undo deployment/url-shortener`
- Check resource limits if OOMKilled
- Review recent code changes

## Deployment Process

1. **Pre-deployment**
   - Run full test suite
   - Check staging environment
   - Notify team in #deployments channel

2. **Deploy**
   - Merge to main (triggers CI/CD)
   - Monitor deployment in GitHub Actions
   - Watch metrics dashboard

3. **Post-deployment**
   - Verify health check returns 200
   - Test shorten/expand functionality
   - Monitor error rates for 15 minutes
   - If issues: rollback immediately

## Database Maintenance

**Backup**: Automated daily backups to S3

**Restore**: `./scripts/restore_db.sh <backup-date>`

**Migrations**: `alembic upgrade head`

## Performance Tuning

**Cache Hit Rate Target**: >90%

**P95 Latency Target**: <100ms

**Error Rate Target**: <0.1%

Check current metrics: http://grafana.example.com/d/url-shortener
```

### Implementation: Enterprise Features

**Audit Logging**:

```python
# shortener/audit.py
import structlog
from datetime import datetime
from typing import Optional

audit_logger = structlog.get_logger("audit")

def log_audit_event(
       user_id: Optional[str],
       action: str,
       resource_type: str,
       resource_id: str,
       metadata: dict
):
       """Log audit trail for compliance"""
       audit_logger.info(
           "audit_event",
           timestamp=datetime.utcnow().isoformat(),
           user_id=user_id,
           action=action,
           resource_type=resource_type,
           resource_id=resource_id,
           metadata=metadata
       )
```

**RBAC Implementation**:

```python
# shortener/rbac.py
from enum import Enum
from typing import Set

class Role(Enum):
       VIEWER = "viewer"
       CREATOR = "creator"
       ADMIN = "admin"


class Permission(Enum):
       VIEW_URL = "view_url"
       CREATE_URL = "create_url"
       DELETE_URL = "delete_url"
       VIEW_ANALYTICS = "view_analytics"

ROLE_PERMISSIONS: dict[Role, Set[Permission]] = {
       Role.VIEWER: {Permission.VIEW_URL, Permission.VIEW_ANALYTICS},
       Role.CREATOR: {Permission.VIEW_URL, Permission.CREATE_URL, Permission.VIEW_ANALYTICS},
       Role.ADMIN: set(Permission),  # All permissions
}

def has_permission(user_role: Role, required_permission: Permission) -> bool:
       return required_permission in ROLE_PERMISSIONS.get(user_role, set())
```

### Grafana Dashboard Configuration

Create dashboards tracking:

- Request rate (requests/sec)
- Error rate by type (4xx, 5xx)
- Latency percentiles (p50, p95, p99)
- Cache hit rate
- Database connection pool usage
- Redis memory usage
- Pod CPU/memory
- Click count trends
- Active URLs count

## ML/AI Specific Additions

For machine learning applications, add these domain-specific considerations at appropriate phases:

### Phase 3-4: ML Development

- **Model Versioning**: Track experiments with MLflow, Weights & Biases, or DVC
- **Data Versioning**: Track dataset changes (DVC, LakeFS)
- **Experiment Tracking**: Log hyperparameters, metrics, artifacts

### Phase 5: ML Performance

- **Model Registry**: Centralized model storage with metadata
- **Feature Store**: Centralized feature engineering and serving
- **Model Optimization**: Quantization, pruning for inference speed

### Phase 6-7: ML Operations

- **A/B Testing**: Compare model versions in production
- **Model Monitoring**: Track drift, performance degradation, fairness metrics
- **Explainability**: SHAP values, feature importance dashboards
- **Retraining Pipelines**: Automated model retraining on new data

## Decision Framework: Which Phase Do You Need?

Use this framework to determine your appropriate maturity level:

### Choose Phase 1 if:

- ✅ Learning Python or prototyping
- ✅ One-off scripts or personal projects
- ✅ No other users or dependencies

### Choose Phase 2 if:

- ✅ Code will be maintained over time
- ✅ Sharing with teammates
- ✅ Need to manage dependencies
- ❌ Not deployed to production yet

### Choose Phase 3 if:

- ✅ Multiple developers collaborating
- ✅ Code quality matters
- ✅ Need confidence when refactoring
- ❌ Not handling production traffic

### Choose Phase 4 if:

- ✅ Deploying to production
- ✅ Multiple environments (dev/staging/prod)
- ✅ Need to debug production issues
- ❌ Performance not a concern yet

### Choose Phase 5 if:

- ✅ Performance is critical
- ✅ High traffic or concurrent users
- ✅ Need to scale horizontally
- ✅ Ready for containerization

### Choose Phase 6 if:

- ✅ Frequent deployments required
- ✅ Need zero-downtime updates
- ✅ Infrastructure should be reproducible
- ✅ Multiple production environments

### Choose Phase 7 if:

- ✅ Mission-critical system
- ✅ SLA commitments to customers
- ✅ Compliance requirements (SOC2, HIPAA, GDPR)
- ✅ Multi-region availability needed
- ✅ 24/7 on-call team

## Related Resources

### Python SWE Lab References

- [Code Organization](../patterns-and-practices/code-organization.md) - Module and package structure
- [Testing](../testing-and-quality/README.md) - Testing strategies and frameworks
- [Design Patterns](../patterns-and-practices/design-patterns.md) - Common design patterns
- [Performance Optimization](../performance-and-optimization/README.md) - Profiling and optimization
- [Tools and Ecosystem](../tools-and-ecosystem/README.md) - Development tools

### External Resources

- [The Twelve-Factor App](https://12factor.net/) - Methodology for building SaaS apps
- [Site Reliability Engineering](https://sre.google/) - Google's SRE practices
- [Python Packaging User Guide](https://packaging.python.org/) - Comprehensive packaging guide

## Summary Checklist

### Phase 1: Foundational Code

- [ ] Working script
- [ ] CLI with error handling
- [ ] Basic functions and structure

### Phase 2: Modular & Installable

- [ ] Modular package with logging
- [ ] Configuration separated from code
- [ ] Installable via pip

### Phase 3: Quality Engineering

- [ ] Comprehensive tests (70%+ coverage)
- [ ] Linting, formatting, type checking
- [ ] Pre-commit hooks
- [ ] Custom exceptions and error handling
- [ ] Structured logging

### Phase 4: Production-Ready Config

- [ ] Environment-based configuration
- [ ] Secret management
- [ ] Configuration validation
- [ ] Structured logging to centralized system
- [ ] Metrics collection (Prometheus)
- [ ] Health check endpoints

### Phase 5: Performance & Scale

- [ ] Database with connection pooling
- [ ] Redis caching
- [ ] Async operations where appropriate
- [ ] Multi-stage Docker build
- [ ] Docker Compose for local dev
- [ ] Optimized container image

### Phase 6: Infrastructure & Deployment

- [ ] Infrastructure as Code (Terraform/CloudFormation)
- [ ] Auto-scaling configuration
- [ ] CI/CD pipeline
- [ ] Automated tests in pipeline
- [ ] Blue-green or canary deployments
- [ ] Database migration strategy

### Phase 7: Operations & Enterprise

- [ ] SLA/SLO definitions
- [ ] Alert rules configured
- [ ] Runbooks for common issues
- [ ] Incident response process
- [ ] Audit logging
- [ ] RBAC implementation
- [ ] Comprehensive documentation
- [ ] Multi-region if required

**Remember**: The goal is not to reach Phase 7 as quickly as possible. The goal is to be at the _right_ phase for your current needs, and to know how to progress when those needs change.

Start simple. Add complexity only when you feel the pain of not having it. Each phase should solve real problems you're experiencing, not hypothetical future problems.
