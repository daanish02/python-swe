\# Python Code Maturity Progression



\## Purpose



This guide outlines the natural progression from a basic Python script to production-grade, enterprise-ready software. It consolidates 15 developmental stages into \*\*7 practical phases\*\*, providing both conceptual understanding and concrete implementation examples.



\*\*Key insight\*\*: Production readiness is not a single milestone but a progression. Most teams don't need all 7 phases—a startup might be perfectly fine at Phase 3-4. The "right" phase depends on your scale, team size, regulatory requirements, and business criticality.



\*\*Important\*\*: Move up phases incrementally as pain points emerge. Don't try to jump from Phase 1 to Phase 6—you'll spend months on infrastructure before delivering value. Build what you need today, and level up when necessary.



\## The 7 Phases



Each phase builds upon the previous, adding capabilities that become essential as your code grows from personal scripts to production systems.



| Phase       | Focus                       | When You Need This                                      |

| ----------- | --------------------------- | ------------------------------------------------------- |

| \*\*Phase 1\*\* | Foundational Code           | Learning Python, rapid prototyping, one-off scripts     |

| \*\*Phase 2\*\* | Modular \& Installable       | Code you'll maintain, share with team, or reuse         |

| \*\*Phase 3\*\* | Quality Engineering         | Code that needs reliability, team collaboration         |

| \*\*Phase 4\*\* | Production-Ready Config     | Deploying to multiple environments, real users          |

| \*\*Phase 5\*\* | Performance \& Scale         | High traffic, performance critical, production loads    |

| \*\*Phase 6\*\* | Infrastructure \& Deployment | Multiple deployments, zero-downtime updates, automation |

| \*\*Phase 7\*\* | Operations \& Enterprise     | Mission-critical systems, compliance, multi-region      |



\## Complete Example: URL Shortener Service



Throughout this guide, we'll build a \*\*URL Shortener with Analytics\*\*—progressing from a basic script to an enterprise-grade system. This example is ideal because it:



\- Starts simple (generate short codes, redirect)

\- Adds complexity naturally (analytics, users, API)

\- Has real production concerns (performance, scale, monitoring)

\- Touches databases, APIs, async operations, caching



---



\## Phase 1: Foundational Code



\*\*Covers: Basic Script → Structured Code\*\*



\### Characteristics



\*\*Basic Script (Level 1)\*\*



\- Single `.py` file that runs

\- Hardcoded values throughout

\- No error handling

\- Manual execution

\- Works "on my machine"



\*\*Structured Code (Level 2)\*\*



\- Multiple functions for organization

\- Basic command-line arguments (argparse)

\- Some error handling (try/except)

\- `if \_\_name\_\_ == "\_\_main\_\_":` guard

\- README with basic instructions



\### When to Use Phase 1



\- Learning Python fundamentals

\- Quick prototypes and experiments

\- One-off data analysis scripts

\- Personal automation tasks



\### Implementation: Basic Script



```python

\# url\_shortener.py

import hashlib



urls = {}



def shorten\_url(long\_url):

&nbsp;   short\_code = hashlib.md5(long\_url.encode()).hexdigest()\[:6]

&nbsp;   urls\[short\_code] = long\_url

&nbsp;   return short\_code



def get\_long\_url(short\_code):

&nbsp;   return urls.get(short\_code)



\# Test it

short = shorten\_url("https://www.example.com/very/long/url")

print(f"Short code: {short}")

print(f"Original: {get\_long\_url(short)}")

```



\*\*Run\*\*: `python url\_shortener.py`



\### Implementation: Structured Code



```python

\# url\_shortener.py

import hashlib

import argparse

import sys



urls = {}



def shorten\_url(long\_url):

&nbsp;   """Generate short code for a URL"""

&nbsp;   if not long\_url.startswith(('http://', 'https://')):

&nbsp;       raise ValueError("URL must start with http:// or https://")



&nbsp;   short\_code = hashlib.md5(long\_url.encode()).hexdigest()\[:6]

&nbsp;   urls\[short\_code] = long\_url

&nbsp;   return short\_code



def get\_long\_url(short\_code):

&nbsp;   """Retrieve original URL from short code"""

&nbsp;   url = urls.get(short\_code)

&nbsp;   if not url:

&nbsp;       raise KeyError(f"Short code '{short\_code}' not found")

&nbsp;   return url



def main():

&nbsp;   parser = argparse.ArgumentParser(description='URL Shortener')

&nbsp;   parser.add\_argument('--shorten', help='URL to shorten')

&nbsp;   parser.add\_argument('--expand', help='Short code to expand')



&nbsp;   args = parser.parse\_args()



&nbsp;   try:

&nbsp;       if args.shorten:

&nbsp;           short\_code = shorten\_url(args.shorten)

&nbsp;           print(f"Short code: {short\_code}")

&nbsp;       elif args.expand:

&nbsp;           long\_url = get\_long\_url(args.expand)

&nbsp;           print(f"Original URL: {long\_url}")

&nbsp;       else:

&nbsp;           parser.print\_help()

&nbsp;   except (ValueError, KeyError) as e:

&nbsp;       print(f"Error: {e}", file=sys.stderr)

&nbsp;       sys.exit(1)



if \_\_name\_\_ == "\_\_main\_\_":

&nbsp;   main()

```



\*\*Run\*\*: `python url\_shortener.py --shorten https://example.com`



\### Transitioning to Phase 2



\*\*Signs you need Phase 2\*\*:



\- You're copying code between scripts

\- Multiple people need to use your code

\- Data doesn't persist between runs

\- Configuration changes require editing code

\- You have no logs to debug issues



---



\## Phase 2: Modular \& Installable



\*\*Covers: Modular Package → Installable Package\*\*



\### Characteristics



\*\*Modular Package (Level 3)\*\*



\- Organized into modules/packages (`src/` directory structure)

\- Configuration separated from code (config files or environment variables)

\- `requirements.txt` for dependencies

\- Basic logging instead of print statements

\- Input validation and error messages



\*\*Installable Package (Level 4)\*\*



\- `setup.py` or `pyproject.toml` for installation

\- Entry points for CLI commands

\- Virtual environment usage

\- `.gitignore` for Python projects

\- Basic documentation (docstrings)

\- Type hints on functions



\### When to Use Phase 2



\- Code you'll maintain long-term

\- Sharing code with teammates

\- Multiple related scripts to organize

\- Need dependency management



\### Implementation: Modular Package



\*\*Structure\*\*:



```

url-shortener/

├── README.md

├── requirements.txt

├── config.ini

└── shortener/

&nbsp;   ├── \_\_init\_\_.py

&nbsp;   ├── core.py

&nbsp;   ├── storage.py

&nbsp;   └── cli.py

```



\*\*requirements.txt\*\*:



```

\# No external dependencies yet

```



\*\*config.ini\*\*:



```ini

\[DEFAULT]

storage\_file = urls.json

base\_url = http://localhost:8000

```



\*\*shortener/storage.py\*\*:



```python

import json

import os

from typing import Optional



class URLStorage:

&nbsp;   def \_\_init\_\_(self, filepath: str):

&nbsp;       self.filepath = filepath

&nbsp;       self.urls = self.\_load()



&nbsp;   def \_load(self) -> dict:

&nbsp;       if os.path.exists(self.filepath):

&nbsp;           with open(self.filepath, 'r') as f:

&nbsp;               return json.load(f)

&nbsp;       return {}



&nbsp;   def \_save(self):

&nbsp;       with open(self.filepath, 'w') as f:

&nbsp;           json.dump(self.urls, f, indent=2)



&nbsp;   def add(self, short\_code: str, long\_url: str):

&nbsp;       self.urls\[short\_code] = long\_url

&nbsp;       self.\_save()



&nbsp;   def get(self, short\_code: str) -> Optional\[str]:

&nbsp;       return self.urls.get(short\_code)



&nbsp;   def exists(self, short\_code: str) -> bool:

&nbsp;       return short\_code in self.urls

```



\*\*shortener/core.py\*\*:



```python

import hashlib

import logging

from .storage import URLStorage



logger = logging.getLogger(\_\_name\_\_)



class URLShortener:

&nbsp;   def \_\_init\_\_(self, storage: URLStorage):

&nbsp;       self.storage = storage



&nbsp;   def shorten(self, long\_url: str) -> str:

&nbsp;       if not long\_url.startswith(('http://', 'https://')):

&nbsp;           raise ValueError("URL must start with http:// or https://")



&nbsp;       short\_code = hashlib.md5(long\_url.encode()).hexdigest()\[:6]



&nbsp;       if self.storage.exists(short\_code):

&nbsp;           logger.info(f"Short code {short\_code} already exists")

&nbsp;       else:

&nbsp;           self.storage.add(short\_code, long\_url)

&nbsp;           logger.info(f"Created short code {short\_code} for {long\_url}")



&nbsp;       return short\_code



&nbsp;   def expand(self, short\_code: str) -> str:

&nbsp;       url = self.storage.get(short\_code)

&nbsp;       if not url:

&nbsp;           raise KeyError(f"Short code '{short\_code}' not found")

&nbsp;       return url

```



\*\*shortener/cli.py\*\*:



```python

import argparse

import logging

import configparser

from .core import URLShortener

from .storage import URLStorage



def setup\_logging():

&nbsp;   logging.basicConfig(

&nbsp;       level=logging.INFO,

&nbsp;       format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'

&nbsp;   )



def load\_config():

&nbsp;   config = configparser.ConfigParser()

&nbsp;   config.read('config.ini')

&nbsp;   return config\['DEFAULT']



def main():

&nbsp;   setup\_logging()

&nbsp;   config = load\_config()



&nbsp;   storage = URLStorage(config\['storage\_file'])

&nbsp;   shortener = URLShortener(storage)



&nbsp;   parser = argparse.ArgumentParser(description='URL Shortener')

&nbsp;   parser.add\_argument('--shorten', help='URL to shorten')

&nbsp;   parser.add\_argument('--expand', help='Short code to expand')



&nbsp;   args = parser.parse\_args()



&nbsp;   try:

&nbsp;       if args.shorten:

&nbsp;           short\_code = shortener.shorten(args.shorten)

&nbsp;           print(f"{config\['base\_url']}/{short\_code}")

&nbsp;       elif args.expand:

&nbsp;           long\_url = shortener.expand(args.expand)

&nbsp;           print(f"Original URL: {long\_url}")

&nbsp;       else:

&nbsp;           parser.print\_help()

&nbsp;   except (ValueError, KeyError) as e:

&nbsp;       logging.error(str(e))

&nbsp;       exit(1)

```



\*\*Run\*\*: `python -m shortener.cli --shorten https://example.com`



\### Implementation: Installable Package



\*\*Add pyproject.toml\*\*:



```toml

\[build-system]

requires = \["setuptools>=61.0", "wheel"]

build-backend = "setuptools.build\_meta"



\[project]

name = "url-shortener"

version = "0.1.0"

description = "A simple URL shortener"

authors = \[{name = "Your Name", email = "you@example.com"}]

requires-python = ">=3.8"

dependencies = \[]



\[project.scripts]

shorten = "shortener.cli:main"

```



\*\*Install\*\*: `pip install -e .`



\*\*Run\*\*: `shorten --shorten https://example.com`



\### Transitioning to Phase 3



\*\*Signs you need Phase 3\*\*:



\- Bugs are hard to track down

\- Unsure if changes break existing functionality

\- Code style inconsistencies across team

\- No confidence in refactoring

\- Production bugs could have been caught earlier



---



\## Phase 3: Quality Engineering



\*\*Covers: Tested Code → Quality Assurance → Robust Error Handling\*\*



\### Characteristics



\*\*Tested Code (Level 5)\*\*



\- Unit tests with pytest

\- Test coverage measurement (pytest-cov)

\- Integration tests for main workflows

\- Test fixtures and mocks for external dependencies

\- CI/CD runs tests automatically (GitHub Actions, GitLab CI)



\*\*Quality Assurance (Level 6)\*\*



\- Code formatting (Black, Ruff)

\- Linting (pylint, flake8, Ruff)

\- Static type checking (mypy)

\- Pre-commit hooks enforce standards

\- Code review process

\- Dependency security scanning (Safety, Bandit)



\*\*Robust Error Handling (Level 7)\*\*



\- Custom exception classes

\- Comprehensive error handling with context

\- Graceful degradation

\- Retry logic with exponential backoff

\- Circuit breakers for external services

\- Structured logging with context (structlog)



\### When to Use Phase 3



\- Code used by others or in production

\- Team collaboration requiring standards

\- Need confidence in refactoring

\- Debugging production issues

\- Want to catch bugs before users do



\### Implementation: Tested Code



\*\*Structure\*\*:



```

url-shortener/

├── tests/

│   ├── \_\_init\_\_.py

│   ├── test\_core.py

│   └── test\_storage.py

└── ...

```



\*\*Update requirements.txt\*\*:



```

pytest

pytest-cov

```



\*\*tests/test\_core.py\*\*:



```python

import pytest

from shortener.core import URLShortener

from shortener.storage import URLStorage

import tempfile

import os



@pytest.fixture

def temp\_storage():

&nbsp;   fd, path = tempfile.mkstemp(suffix='.json')

&nbsp;   os.close(fd)

&nbsp;   storage = URLStorage(path)

&nbsp;   yield storage

&nbsp;   os.unlink(path)



@pytest.fixture

def shortener(temp\_storage):

&nbsp;   return URLShortener(temp\_storage)



def test\_shorten\_url(shortener):

&nbsp;   url = "https://example.com"

&nbsp;   short\_code = shortener.shorten(url)

&nbsp;   assert len(short\_code) == 6

&nbsp;   assert shortener.expand(short\_code) == url



def test\_shorten\_invalid\_url(shortener):

&nbsp;   with pytest.raises(ValueError):

&nbsp;       shortener.shorten("not-a-url")



def test\_expand\_nonexistent(shortener):

&nbsp;   with pytest.raises(KeyError):

&nbsp;       shortener.expand("abc123")



def test\_shorten\_idempotent(shortener):

&nbsp;   url = "https://example.com"

&nbsp;   code1 = shortener.shorten(url)

&nbsp;   code2 = shortener.shorten(url)

&nbsp;   assert code1 == code2

```



\*\*tests/test\_storage.py\*\*:



```python

import pytest

from shortener.storage import URLStorage

import tempfile

import os



@pytest.fixture

def temp\_file():

&nbsp;   fd, path = tempfile.mkstemp(suffix='.json')

&nbsp;   os.close(fd)

&nbsp;   yield path

&nbsp;   if os.path.exists(path):

&nbsp;       os.unlink(path)



def test\_storage\_add\_and\_get(temp\_file):

&nbsp;   storage = URLStorage(temp\_file)

&nbsp;   storage.add("abc123", "https://example.com")

&nbsp;   assert storage.get("abc123") == "https://example.com"



def test\_storage\_persistence(temp\_file):

&nbsp;   storage1 = URLStorage(temp\_file)

&nbsp;   storage1.add("abc123", "https://example.com")



&nbsp;   storage2 = URLStorage(temp\_file)

&nbsp;   assert storage2.get("abc123") == "https://example.com"



def test\_storage\_get\_nonexistent(temp\_file):

&nbsp;   storage = URLStorage(temp\_file)

&nbsp;   assert storage.get("xyz789") is None

```



\*\*Run tests\*\*: `pytest --cov=shortener tests/`



\### Implementation: Quality Assurance



\*\*Update requirements.txt\*\*:



```

pytest

pytest-cov

black

ruff

mypy

pre-commit

```



\*\*.pre-commit-config.yaml\*\*:



```yaml

repos:

&nbsp; - repo: https://github.com/psf/black

&nbsp;   rev: 23.12.1

&nbsp;   hooks:

&nbsp;     - id: black

&nbsp;       language\_version: python3



&nbsp; - repo: https://github.com/astral-sh/ruff-pre-commit

&nbsp;   rev: v0.1.9

&nbsp;   hooks:

&nbsp;     - id: ruff

&nbsp;       args: \[--fix]



&nbsp; - repo: https://github.com/pre-commit/mirrors-mypy

&nbsp;   rev: v1.8.0

&nbsp;   hooks:

&nbsp;     - id: mypy

&nbsp;       additional\_dependencies: \[types-all]

```



\*\*Update pyproject.toml\*\* (add):



```toml

\[tool.black]

line-length = 88



\[tool.ruff]

line-length = 88

select = \["E", "F", "I", "N", "W"]



\[tool.mypy]

python\_version = "3.8"

warn\_return\_any = true

warn\_unused\_configs = true

disallow\_untyped\_defs = true

```



\*\*Update code with type hints\*\* (shortener/core.py):



```python

import hashlib

import logging

from typing import Optional

from .storage import URLStorage



logger = logging.getLogger(\_\_name\_\_)



class URLShortener:

&nbsp;   def \_\_init\_\_(self, storage: URLStorage) -> None:

&nbsp;       self.storage = storage



&nbsp;   def shorten(self, long\_url: str) -> str:

&nbsp;       if not long\_url.startswith(('http://', 'https://')):

&nbsp;           raise ValueError("URL must start with http:// or https://")



&nbsp;       short\_code = hashlib.md5(long\_url.encode()).hexdigest()\[:6]



&nbsp;       if self.storage.exists(short\_code):

&nbsp;           logger.info(f"Short code {short\_code} already exists")

&nbsp;       else:

&nbsp;           self.storage.add(short\_code, long\_url)

&nbsp;           logger.info(f"Created short code {short\_code} for {long\_url}")



&nbsp;       return short\_code



&nbsp;   def expand(self, short\_code: str) -> str:

&nbsp;       url = self.storage.get(short\_code)

&nbsp;       if not url:

&nbsp;           raise KeyError(f"Short code '{short\_code}' not found")

&nbsp;       return url

```



\*\*Setup\*\*: `pre-commit install`



\*\*Run\*\*: `black .`, `ruff check .`, `mypy shortener/`



\### Implementation: Robust Error Handling \& Web API



\*\*Update requirements.txt\*\*:



```

flask

python-dotenv

structlog

tenacity

```



\*\*Create .env\*\*:



```

FLASK\_ENV=development

STORAGE\_FILE=urls.json

BASE\_URL=http://localhost:5000

```



\*\*shortener/exceptions.py\*\*:



```python

class ShortenerError(Exception):

&nbsp;   """Base exception for shortener"""

&nbsp;   pass



class InvalidURLError(ShortenerError):

&nbsp;   """Raised when URL is invalid"""

&nbsp;   pass



class ShortCodeNotFoundError(ShortenerError):

&nbsp;   """Raised when short code doesn't exist"""

&nbsp;   pass



class StorageError(ShortenerError):

&nbsp;   """Raised when storage operation fails"""

&nbsp;   pass

```



\*\*shortener/api.py\*\*:



```python

from flask import Flask, request, jsonify, redirect

from typing import Tuple

import structlog

import os

from dotenv import load\_dotenv



from .core import URLShortener

from .storage import URLStorage

from .exceptions import InvalidURLError, ShortCodeNotFoundError



load\_dotenv()



logger = structlog.get\_logger()



app = Flask(\_\_name\_\_)



\# Initialize storage and shortener

storage = URLStorage(os.getenv('STORAGE\_FILE', 'urls.json'))

shortener = URLShortener(storage)



@app.route('/api/shorten', methods=\['POST'])

def api\_shorten() -> Tuple\[dict, int]:

&nbsp;   try:

&nbsp;       data = request.get\_json()

&nbsp;       if not data or 'url' not in data:

&nbsp;           return jsonify({'error': 'Missing url parameter'}), 400



&nbsp;       long\_url = data\['url']

&nbsp;       short\_code = shortener.shorten(long\_url)



&nbsp;       base\_url = os.getenv('BASE\_URL', 'http://localhost:5000')

&nbsp;       short\_url = f"{base\_url}/{short\_code}"



&nbsp;       logger.info("url\_shortened", long\_url=long\_url, short\_code=short\_code)



&nbsp;       return jsonify({

&nbsp;           'short\_code': short\_code,

&nbsp;           'short\_url': short\_url,

&nbsp;           'long\_url': long\_url

&nbsp;       }), 201



&nbsp;   except InvalidURLError as e:

&nbsp;       logger.warning("invalid\_url", error=str(e))

&nbsp;       return jsonify({'error': str(e)}), 400

&nbsp;   except Exception as e:

&nbsp;       logger.error("shorten\_failed", error=str(e))

&nbsp;       return jsonify({'error': 'Internal server error'}), 500



@app.route('/<short\_code>')

def redirect\_url(short\_code: str):

&nbsp;   try:

&nbsp;       long\_url = shortener.expand(short\_code)

&nbsp;       logger.info("url\_redirected", short\_code=short\_code, long\_url=long\_url)

&nbsp;       return redirect(long\_url, code=302)

&nbsp;   except ShortCodeNotFoundError:

&nbsp;       logger.warning("short\_code\_not\_found", short\_code=short\_code)

&nbsp;       return jsonify({'error': 'Short code not found'}), 404

&nbsp;   except Exception as e:

&nbsp;       logger.error("redirect\_failed", short\_code=short\_code, error=str(e))

&nbsp;       return jsonify({'error': 'Internal server error'}), 500



@app.route('/health')

def health():

&nbsp;   return jsonify({'status': 'healthy'}), 200



if \_\_name\_\_ == '\_\_main\_\_':

&nbsp;   app.run(debug=True)

```



\*\*Run\*\*: `python -m shortener.api`



\*\*Test\*\*: `curl -X POST http://localhost:5000/api/shorten -H "Content-Type: application/json" -d '{"url":"https://example.com"}'`



\### Transitioning to Phase 4



\*\*Signs you need Phase 4\*\*:



\- Deploying to multiple environments (dev, staging, prod)

\- Configuration scattered across code and env vars

\- No visibility into production behavior

\- Can't debug production issues effectively

\- Secrets hardcoded or insecurely managed



---



\## Phase 4: Production-Ready Configuration



\*\*Covers: Configuration Management → Observability\*\*



\### Characteristics



\*\*Configuration Management (Level 8)\*\*



\- Environment-based configs (dev, staging, prod)

\- Secret management (not in code or git)

\- Feature flags for gradual rollouts

\- Configuration validation (pydantic)

\- `.env` files with python-dotenv



\*\*Observability (Level 9)\*\*



\- Structured logging to centralized system (ELK, CloudWatch)

\- Metrics collection (Prometheus, StatsD)

\- Distributed tracing (OpenTelemetry)

\- Health check endpoints

\- Performance profiling instrumentation

\- Alerting on critical errors



\### When to Use Phase 4



\- Deploying to production environments

\- Multiple environments to manage

\- Need to debug production issues

\- Want to understand system behavior

\- Compliance requires audit trails



\### Implementation: Configuration Management



\*\*Update requirements.txt\*\*:



```

pydantic

pydantic-settings

```



\*\*shortener/config.py\*\*:



```python

from pydantic\_settings import BaseSettings

from pydantic import Field, validator



class Settings(BaseSettings):

&nbsp;   # App settings

&nbsp;   environment: str = Field(default="development")

&nbsp;   debug: bool = Field(default=False)



&nbsp;   # Storage settings

&nbsp;   storage\_file: str = Field(default="urls.json")



&nbsp;   # URL settings

&nbsp;   base\_url: str = Field(default="http://localhost:5000")

&nbsp;   short\_code\_length: int = Field(default=6, ge=4, le=10)



&nbsp;   # Database settings (for future use)

&nbsp;   database\_url: str = Field(default="sqlite:///urls.db")



&nbsp;   # Redis settings (for future use)

&nbsp;   redis\_url: str = Field(default="redis://localhost:6379/0")



&nbsp;   # API settings

&nbsp;   rate\_limit: int = Field(default=100)



&nbsp;   class Config:

&nbsp;       env\_file = ".env"

&nbsp;       case\_sensitive = False



&nbsp;   @validator('base\_url')

&nbsp;   def validate\_base\_url(cls, v):

&nbsp;       if not v.startswith(('http://', 'https://')):

&nbsp;           raise ValueError('base\_url must start with http:// or https://')

&nbsp;       return v.rstrip('/')



settings = Settings()

```



\*\*Environment-specific configs\*\*:



\*\*.env.development\*\*:



```

ENVIRONMENT=development

DEBUG=true

STORAGE\_FILE=urls\_dev.json

BASE\_URL=http://localhost:5000

```



\*\*.env.production\*\*:



```

ENVIRONMENT=production

DEBUG=false

STORAGE\_FILE=/var/data/urls.json

BASE\_URL=https://short.example.com

DATABASE\_URL=postgresql://user:pass@db:5432/urls

REDIS\_URL=redis://redis:6379/0

```



\*\*Update api.py to use settings\*\*:



```python

from .config import settings



storage = URLStorage(settings.storage\_file)

\# ... rest of code uses settings.\*

```



\### Implementation: Observability



\*\*Update requirements.txt\*\*:



```

prometheus-client

opentelemetry-api

opentelemetry-sdk

opentelemetry-instrumentation-flask

```



\*\*shortener/metrics.py\*\*:



```python

from prometheus\_client import Counter, Histogram, Gauge

import time



\# Metrics

url\_shortened\_total = Counter(

&nbsp;   'url\_shortened\_total',

&nbsp;   'Total number of URLs shortened'

)



url\_redirected\_total = Counter(

&nbsp;   'url\_redirected\_total',

&nbsp;   'Total number of redirects'

)



url\_not\_found\_total = Counter(

&nbsp;   'url\_not\_found\_total',

&nbsp;   'Total number of 404s'

)



request\_duration = Histogram(

&nbsp;   'request\_duration\_seconds',

&nbsp;   'Request duration in seconds',

&nbsp;   \['method', 'endpoint']

)



active\_urls = Gauge(

&nbsp;   'active\_urls\_total',

&nbsp;   'Total number of active short URLs'

)

```



\*\*Update api.py with observability\*\*:



```python

from flask import Flask, request, jsonify, redirect

from prometheus\_client import generate\_latest, CONTENT\_TYPE\_LATEST

import structlog

import time

from opentelemetry import trace

from opentelemetry.sdk.trace import TracerProvider

from opentelemetry.sdk.trace.export import ConsoleSpanExporter, BatchSpanProcessor



from .metrics import (

&nbsp;   url\_shortened\_total,

&nbsp;   url\_redirected\_total,

&nbsp;   url\_not\_found\_total,

&nbsp;   request\_duration,

&nbsp;   active\_urls

)



\# Setup tracing

trace.set\_tracer\_provider(TracerProvider())

tracer = trace.get\_tracer(\_\_name\_\_)

trace.get\_tracer\_provider().add\_span\_processor(

&nbsp;   BatchSpanProcessor(ConsoleSpanExporter())

)



\# ... existing code ...



@app.route('/api/shorten', methods=\['POST'])

def api\_shorten():

&nbsp;   with tracer.start\_as\_current\_span("shorten\_url"):

&nbsp;       start\_time = time.time()

&nbsp;       try:

&nbsp;           # ... existing code ...

&nbsp;           url\_shortened\_total.inc()

&nbsp;           active\_urls.set(len(storage.urls))

&nbsp;           return result, 201

&nbsp;       finally:

&nbsp;           request\_duration.labels(method='POST', endpoint='/api/shorten').observe(

&nbsp;               time.time() - start\_time

&nbsp;           )



@app.route('/metrics')

def metrics():

&nbsp;   return generate\_latest(), 200, {'Content-Type': CONTENT\_TYPE\_LATEST}



\# ... rest of code ...

```



\### Transitioning to Phase 5



\*\*Signs you need Phase 5\*\*:



\- Response times too slow for users

\- System can't handle current traffic

\- Database is a bottleneck

\- Need to scale horizontally

\- Want consistent deployments across environments



---



\## Phase 5: Performance \& Scale



\*\*Covers: Scalability \& Performance → Containerization\*\*



\### Characteristics



\*\*Scalability \& Performance (Level 10)\*\*



\- Async/await for I/O-bound operations

\- Connection pooling for databases

\- Caching strategies (Redis, in-memory)

\- Background job processing (Celery, RQ)

\- Database query optimization

\- Resource limits and rate limiting



\*\*Containerization (Level 11)\*\*



\- Dockerfile with multi-stage builds

\- Docker Compose for local development

\- Optimized image size (Alpine, slim variants)

\- `.dockerignore` file

\- Container security scanning

\- Non-root user in container



\### When to Use Phase 5



\- High traffic or performance-critical

\- Need to scale horizontally

\- Multiple services to coordinate

\- Want consistent dev/prod environments

\- Deploying to cloud or Kubernetes



\### Implementation: Database \& Caching



\*\*Update requirements.txt\*\*:



```

sqlalchemy

redis

aiohttp

asyncio

```



\*\*shortener/models.py\*\*:



```python

from sqlalchemy import Column, String, Integer, DateTime, create\_engine

from sqlalchemy.ext.declarative import declarative\_base

from sqlalchemy.orm import sessionmaker

from datetime import datetime



Base = declarative\_base()



class URL(Base):

&nbsp;   \_\_tablename\_\_ = 'urls'



&nbsp;   id = Column(Integer, primary\_key=True)

&nbsp;   short\_code = Column(String(10), unique=True, index=True, nullable=False)

&nbsp;   long\_url = Column(String(2048), nullable=False)

&nbsp;   created\_at = Column(DateTime, default=datetime.utcnow)

&nbsp;   click\_count = Column(Integer, default=0)



&nbsp;   def \_\_repr\_\_(self):

&nbsp;       return f"<URL(short\_code='{self.short\_code}', long\_url='{self.long\_url}')>"

```



\*\*shortener/database.py\*\*:



```python

from sqlalchemy import create\_engine

from sqlalchemy.orm import sessionmaker, Session

from contextlib import contextmanager

from .models import Base, URL

from .config import settings

import redis



engine = create\_engine(settings.database\_url, echo=settings.debug)

SessionLocal = sessionmaker(bind=engine)



\# Redis client

redis\_client = redis.from\_url(settings.redis\_url, decode\_responses=True)



def init\_db():

&nbsp;   Base.metadata.create\_all(bind=engine)



@contextmanager

def get\_db() -> Session:

&nbsp;   db = SessionLocal()

&nbsp;   try:

&nbsp;       yield db

&nbsp;       db.commit()

&nbsp;   except Exception:

&nbsp;       db.rollback()

&nbsp;       raise

&nbsp;   finally:

&nbsp;       db.close()

```



\*\*shortener/service.py\*\*:



```python

from typing import Optional

import hashlib

import structlog

from .database import get\_db, redis\_client

from .models import URL

from .exceptions import InvalidURLError, ShortCodeNotFoundError

from .config import settings



logger = structlog.get\_logger()



class URLService:

&nbsp;   def \_\_init\_\_(self):

&nbsp;       self.cache\_ttl = 3600  # 1 hour



&nbsp;   def \_generate\_short\_code(self, long\_url: str) -> str:

&nbsp;       return hashlib.md5(long\_url.encode()).hexdigest()\[:settings.short\_code\_length]



&nbsp;   def shorten(self, long\_url: str) -> str:

&nbsp;       if not long\_url.startswith(('http://', 'https://')):

&nbsp;           raise InvalidURLError("URL must start with http:// or https://")



&nbsp;       short\_code = self.\_generate\_short\_code(long\_url)



&nbsp;       # Check cache first

&nbsp;       cached = redis\_client.get(f"url:{short\_code}")

&nbsp;       if cached:

&nbsp;           logger.info("cache\_hit", short\_code=short\_code)

&nbsp;           return short\_code



&nbsp;       # Check database

&nbsp;       with get\_db() as db:

&nbsp;           existing = db.query(URL).filter(URL.short\_code == short\_code).first()

&nbsp;           if existing:

&nbsp;               # Update cache

&nbsp;               redis\_client.setex(

&nbsp;                   f"url:{short\_code}",

&nbsp;                   self.cache\_ttl,

&nbsp;                   existing.long\_url

&nbsp;               )

&nbsp;               return short\_code



&nbsp;           # Create new

&nbsp;           url = URL(short\_code=short\_code, long\_url=long\_url)

&nbsp;           db.add(url)

&nbsp;           db.commit()



&nbsp;           # Cache it

&nbsp;           redis\_client.setex(

&nbsp;               f"url:{short\_code}",

&nbsp;               self.cache\_ttl,

&nbsp;               long\_url

&nbsp;           )



&nbsp;           logger.info("url\_created", short\_code=short\_code, long\_url=long\_url)

&nbsp;           return short\_code



&nbsp;   def expand(self, short\_code: str) -> str:

&nbsp;       # Check cache

&nbsp;       cached = redis\_client.get(f"url:{short\_code}")

&nbsp;       if cached:

&nbsp;           logger.info("cache\_hit", short\_code=short\_code)

&nbsp;           self.\_increment\_clicks(short\_code)

&nbsp;           return cached



&nbsp;       # Check database

&nbsp;       with get\_db() as db:

&nbsp;           url = db.query(URL).filter(URL.short\_code == short\_code).first()

&nbsp;           if not url:

&nbsp;               raise ShortCodeNotFoundError(f"Short code '{short\_code}' not found")



&nbsp;           # Cache it

&nbsp;           redis\_client.setex(

&nbsp;               f"url:{short\_code}",

&nbsp;               self.cache\_ttl,

&nbsp;               url.long\_url

&nbsp;           )



&nbsp;           self.\_increment\_clicks(short\_code)

&nbsp;           return url.long\_url



&nbsp;   def \_increment\_clicks(self, short\_code: str):

&nbsp;       with get\_db() as db:

&nbsp;           url = db.query(URL).filter(URL.short\_code == short\_code).first()

&nbsp;           if url:

&nbsp;               url.click\_count += 1

&nbsp;               db.commit()

```



\### Implementation: Containerization



\*\*Dockerfile\*\*:



```dockerfile

\# Multi-stage build

FROM python:3.11-slim as builder



WORKDIR /app



\# Install dependencies

COPY requirements.txt .

RUN pip install --no-cache-dir --user -r requirements.txt



\# Final stage

FROM python:3.11-slim



\# Create non-root user

RUN useradd -m -u 1000 appuser



WORKDIR /app



\# Copy dependencies from builder

COPY --from=builder /root/.local /home/appuser/.local



\# Copy application

COPY --chown=appuser:appuser . .



\# Make sure scripts in .local are usable

ENV PATH=/home/appuser/.local/bin:$PATH



USER appuser



EXPOSE 5000



CMD \["gunicorn", "--bind", "0.0.0.0:5000", "--workers", "4", "shortener.api:app"]

```



\*\*.dockerignore\*\*:



```

\_\_pycache\_\_

\*.pyc

\*.pyo

\*.pyd

.Python

env/

venv/

.git

.gitignore

.pytest\_cache

.coverage

\*.db

\*.json

.env

.env.\*

tests/

```



\*\*docker-compose.yml\*\*:



```yaml

version: "3.8"



services:

&nbsp; app:

&nbsp;   build: .

&nbsp;   ports:

&nbsp;     - "5000:5000"

&nbsp;   environment:

&nbsp;     - DATABASE\_URL=postgresql://postgres:password@db:5432/urlshortener

&nbsp;     - REDIS\_URL=redis://redis:6379/0

&nbsp;   depends\_on:

&nbsp;     - db

&nbsp;     - redis

&nbsp;   volumes:

&nbsp;     - ./:/app



&nbsp; db:

&nbsp;   image: postgres:15-alpine

&nbsp;   environment:

&nbsp;     - POSTGRES\_USER=postgres

&nbsp;     - POSTGRES\_PASSWORD=password

&nbsp;     - POSTGRES\_DB=urlshortener

&nbsp;   volumes:

&nbsp;     - postgres\_data:/var/lib/postgresql/data



&nbsp; redis:

&nbsp;   image: redis:7-alpine

&nbsp;   ports:

&nbsp;     - "6379:6379"



&nbsp; prometheus:

&nbsp;   image: prom/prometheus

&nbsp;   ports:

&nbsp;     - "9090:9090"

&nbsp;   volumes:

&nbsp;     - ./prometheus.yml:/etc/prometheus/prometheus.yml



&nbsp; grafana:

&nbsp;   image: grafana/grafana

&nbsp;   ports:

&nbsp;     - "3000:3000"

&nbsp;   environment:

&nbsp;     - GF\_SECURITY\_ADMIN\_PASSWORD=admin



volumes:

&nbsp; postgres\_data:

```



\*\*prometheus.yml\*\*:



```yaml

global:

&nbsp; scrape\_interval: 15s



scrape\_configs:

&nbsp; - job\_name: "url-shortener"

&nbsp;   static\_configs:

&nbsp;     - targets: \["app:5000"]

```



\*\*Run\*\*: `docker-compose up --build`



\### Transitioning to Phase 6



\*\*Signs you need Phase 6\*\*:



\- Manual deployments are error-prone

\- Need to deploy multiple times per day

\- Want zero-downtime deployments

\- Managing multiple environments manually

\- Need infrastructure to be reproducible



---



\## Phase 6: Infrastructure \& Deployment



\*\*Covers: Infrastructure as Code → Production Deployment\*\*



\### Characteristics



\*\*Infrastructure as Code (Level 12)\*\*



\- Terraform or CloudFormation for resources

\- Kubernetes manifests or Helm charts

\- Service mesh configuration (Istio, Linkerd)

\- Auto-scaling rules

\- Load balancer configuration

\- Database migration management (Alembic)



\*\*Production Deployment (Level 13)\*\*



\- Blue-green or canary deployments

\- Rolling updates with zero downtime

\- Automated rollback on failure

\- Database migration strategy

\- Secrets rotation

\- Backup and disaster recovery procedures



\### When to Use Phase 6



\- Frequent deployments needed

\- Multiple production environments

\- Need reproducible infrastructure

\- Want automated deployment pipeline

\- Zero-downtime requirement



\### Implementation: Infrastructure as Code



\*\*terraform/main.tf\*\*:



```hcl

terraform {

&nbsp; required\_providers {

&nbsp;   aws = {

&nbsp;     source  = "hashicorp/aws"

&nbsp;     version = "~> 5.0"

&nbsp;   }

&nbsp; }

}



provider "aws" {

&nbsp; region = var.aws\_region

}



\# VPC

resource "aws\_vpc" "main" {

&nbsp; cidr\_block = "10.0.0.0/16"



&nbsp; tags = {

&nbsp;   Name = "url-shortener-vpc"

&nbsp; }

}



\# ECS Cluster

resource "aws\_ecs\_cluster" "main" {

&nbsp; name = "url-shortener-cluster"

}



\# RDS PostgreSQL

resource "aws\_db\_instance" "postgres" {

&nbsp; identifier        = "url-shortener-db"

&nbsp; engine            = "postgres"

&nbsp; engine\_version    = "15.3"

&nbsp; instance\_class    = "db.t3.micro"

&nbsp; allocated\_storage = 20



&nbsp; db\_name  = "urlshortener"

&nbsp; username = var.db\_username

&nbsp; password = var.db\_password



&nbsp; skip\_final\_snapshot = true

}



\# ElastiCache Redis

resource "aws\_elasticache\_cluster" "redis" {

&nbsp; cluster\_id      = "url-shortener-cache"

&nbsp; engine          = "redis"

&nbsp; node\_type       = "cache.t3.micro"

&nbsp; num\_cache\_nodes = 1

&nbsp; port            = 6379

}



\# Application Load Balancer

resource "aws\_lb" "main" {

&nbsp; name               = "url-shortener-alb"

&nbsp; internal           = false

&nbsp; load\_balancer\_type = "application"

&nbsp; subnets            = aws\_subnet.public\[\*].id

}



\# ECS Task Definition

resource "aws\_ecs\_task\_definition" "app" {

&nbsp; family                   = "url-shortener"

&nbsp; network\_mode             = "awsvpc"

&nbsp; requires\_compatibilities = \["FARGATE"]

&nbsp; cpu                      = "256"

&nbsp; memory                   = "512"



&nbsp; container\_definitions = jsonencode(\[{

&nbsp;   name  = "url-shortener"

&nbsp;   image = "${var.ecr\_repository}:latest"



&nbsp;   portMappings = \[{

&nbsp;     containerPort = 5000

&nbsp;     protocol      = "tcp"

&nbsp;   }]



&nbsp;   environment = \[

&nbsp;     {

&nbsp;       name  = "DATABASE\_URL"

&nbsp;       value = "postgresql://${var.db\_username}:${var.db\_password}@${aws\_db\_instance.postgres.endpoint}/urlshortener"

&nbsp;     },

&nbsp;     {

&nbsp;       name  = "REDIS\_URL"

&nbsp;       value = "redis://${aws\_elasticache\_cluster.redis.cache\_nodes\[0].address}:6379/0"

&nbsp;     }

&nbsp;   ]

&nbsp; }])

}



\# Auto Scaling

resource "aws\_appautoscaling\_target" "ecs" {

&nbsp; max\_capacity       = 10

&nbsp; min\_capacity       = 2

&nbsp; resource\_id        = "service/${aws\_ecs\_cluster.main.name}/${aws\_ecs\_service.app.name}"

&nbsp; scalable\_dimension = "ecs:service:DesiredCount"

&nbsp; service\_namespace  = "ecs"

}



resource "aws\_appautoscaling\_policy" "cpu" {

&nbsp; name               = "cpu-autoscaling"

&nbsp; policy\_type        = "TargetTrackingScaling"

&nbsp; resource\_id        = aws\_appautoscaling\_target.ecs.resource\_id

&nbsp; scalable\_dimension = aws\_appautoscaling\_target.ecs.scalable\_dimension

&nbsp; service\_namespace  = aws\_appautoscaling\_target.ecs.service\_namespace



&nbsp; target\_tracking\_scaling\_policy\_configuration {

&nbsp;   predefined\_metric\_specification {

&nbsp;     predefined\_metric\_type = "ECSServiceAverageCPUUtilization"

&nbsp;   }

&nbsp;   target\_value = 70.0

&nbsp; }

}

```



\*\*terraform/variables.tf\*\*:



```hcl

variable "aws\_region" {

&nbsp; default = "us-east-1"

}



variable "db\_username" {

&nbsp; sensitive = true

}



variable "db\_password" {

&nbsp; sensitive = true

}



variable "ecr\_repository" {

&nbsp; description = "ECR repository URL"

}

```



\*\*Run\*\*: `terraform init \&\& terraform plan \&\& terraform apply`



\### Implementation: CI/CD Pipeline



\*\*.github/workflows/deploy.yml\*\*:



```yaml

name: Deploy to Production



on:

&nbsp; push:

&nbsp;   branches: \[main]



jobs:

&nbsp; test:

&nbsp;   runs-on: ubuntu-latest

&nbsp;   steps:

&nbsp;     - uses: actions/checkout@v3



&nbsp;     - name: Set up Python

&nbsp;       uses: actions/setup-python@v4

&nbsp;       with:

&nbsp;         python-version: "3.11"



&nbsp;     - name: Install dependencies

&nbsp;       run: |

&nbsp;         pip install -r requirements.txt

&nbsp;         pip install pytest pytest-cov



&nbsp;     - name: Run tests

&nbsp;       run: pytest --cov=shortener tests/



&nbsp;     - name: Run linting

&nbsp;       run: |

&nbsp;         pip install ruff black mypy

&nbsp;         black --check .

&nbsp;         ruff check .

&nbsp;         mypy shortener/



&nbsp; build:

&nbsp;   needs: test

&nbsp;   runs-on: ubuntu-latest

&nbsp;   steps:

&nbsp;     - uses: actions/checkout@v3



&nbsp;     - name: Configure AWS credentials

&nbsp;       uses: aws-actions/configure-aws-credentials@v2

&nbsp;       with:

&nbsp;         aws-access-key-id: ${{ secrets.AWS\_ACCESS\_KEY\_ID }}

&nbsp;         aws-secret-access-key: ${{ secrets.AWS\_SECRET\_ACCESS\_KEY }}

&nbsp;         aws-region: us-east-1



&nbsp;     - name: Login to Amazon ECR

&nbsp;       id: login-ecr

&nbsp;       uses: aws-actions/amazon-ecr-login@v1



&nbsp;     - name: Build and push Docker image

&nbsp;       env:

&nbsp;         ECR\_REGISTRY: ${{ steps.login-ecr.outputs.registry }}

&nbsp;         IMAGE\_TAG: ${{ github.sha }}

&nbsp;       run: |

&nbsp;         docker build -t $ECR\_REGISTRY/url-shortener:$IMAGE\_TAG .

&nbsp;         docker tag $ECR\_REGISTRY/url-shortener:$IMAGE\_TAG $ECR\_REGISTRY/url-shortener:latest

&nbsp;         docker push $ECR\_REGISTRY/url-shortener:$IMAGE\_TAG

&nbsp;         docker push $ECR\_REGISTRY/url-shortener:latest



&nbsp; deploy:

&nbsp;   needs: build

&nbsp;   runs-on: ubuntu-latest

&nbsp;   steps:

&nbsp;     - uses: actions/checkout@v3



&nbsp;     - name: Deploy to ECS

&nbsp;       run: |

&nbsp;         aws ecs update-service \\

&nbsp;           --cluster url-shortener-cluster \\

&nbsp;           --service url-shortener-service \\

&nbsp;           --force-new-deployment



&nbsp;     - name: Wait for deployment

&nbsp;       run: |

&nbsp;         aws ecs wait services-stable \\

&nbsp;           --cluster url-shortener-cluster \\

&nbsp;           --services url-shortener-service

```



\*\*Database Migration with Alembic\*\*:



\*\*alembic/versions/001\_initial.py\*\*:



```python

from alembic import op

import sqlalchemy as sa



def upgrade():

&nbsp;   op.create\_table(

&nbsp;       'urls',

&nbsp;       sa.Column('id', sa.Integer(), nullable=False),

&nbsp;       sa.Column('short\_code', sa.String(10), nullable=False),

&nbsp;       sa.Column('long\_url', sa.String(2048), nullable=False),

&nbsp;       sa.Column('created\_at', sa.DateTime(), nullable=True),

&nbsp;       sa.Column('click\_count', sa.Integer(), nullable=True),

&nbsp;       sa.PrimaryKeyConstraint('id')

&nbsp;   )

&nbsp;   op.create\_index('ix\_urls\_short\_code', 'urls', \['short\_code'], unique=True)



def downgrade():

&nbsp;   op.drop\_index('ix\_urls\_short\_code', table\_name='urls')

&nbsp;   op.drop\_table('urls')

```



\### Transitioning to Phase 7



\*\*Signs you need Phase 7\*\*:



\- System is business-critical

\- Need 99.9%+ uptime SLAs

\- Compliance requirements (SOC2, HIPAA, GDPR)

\- Multiple regions for disaster recovery

\- On-call team needs runbooks and alerts



---



\## Phase 7: Operations \& Enterprise



\*\*Covers: Monitoring \& Maintenance → Enterprise Grade\*\*



\### Characteristics



\*\*Monitoring \& Maintenance (Level 14)\*\*



\- SLA/SLO definitions with alerting

\- On-call rotation and runbooks

\- Incident response procedures

\- Post-mortem process

\- Performance regression detection

\- Cost monitoring and optimization



\*\*Enterprise Grade (Level 15)\*\*



\- Multi-region deployment

\- Compliance certifications (SOC2, HIPAA, GDPR)

\- Audit logging

\- Role-based access control (RBAC)

\- Data privacy and retention policies

\- Chaos engineering (intentional failure testing)

\- Comprehensive documentation (architecture diagrams, API docs, runbooks)

\- Version compatibility guarantees

\- Deprecation policies



\### When to Use Phase 7



\- Mission-critical production systems

\- Enterprise customers requiring SLAs

\- Compliance and regulatory requirements

\- Multi-region availability needed

\- Large distributed teams



\### Implementation: Monitoring \& Alerting



\*\*prometheus\_rules.yml\*\*:



```yaml

groups:

&nbsp; - name: url\_shortener\_alerts

&nbsp;   interval: 30s

&nbsp;   rules:

&nbsp;     - alert: HighErrorRate

&nbsp;       expr: rate(url\_not\_found\_total\[5m]) > 0.05

&nbsp;       for: 5m

&nbsp;       labels:

&nbsp;         severity: warning

&nbsp;       annotations:

&nbsp;         summary: High error rate detected

&nbsp;         description: Error rate is {{ $value }} errors/sec



&nbsp;     - alert: HighLatency

&nbsp;       expr: histogram\_quantile(0.95, rate(request\_duration\_seconds\_bucket\[5m])) > 0.5

&nbsp;       for: 5m

&nbsp;       labels:

&nbsp;         severity: warning

&nbsp;       annotations:

&nbsp;         summary: High latency detected

&nbsp;         description: P95 latency is {{ $value }} seconds



&nbsp;     - alert: ServiceDown

&nbsp;       expr: up{job="url-shortener"} == 0

&nbsp;       for: 1m

&nbsp;       labels:

&nbsp;         severity: critical

&nbsp;       annotations:

&nbsp;         summary: Service is down

&nbsp;         description: URL shortener service is not responding

```



\*\*RUNBOOK.md\*\*:



```markdown

\# URL Shortener Runbook



\## Common Issues



\### High Error Rate (404s)



\*\*Symptoms\*\*: Alert "HighErrorRate" firing



\*\*Investigation\*\*:



1\. Check Grafana dashboard for spike in 404s

2\. Look at logs: `kubectl logs -l app=url-shortener --tail=100`

3\. Check if specific short codes are missing



\*\*Resolution\*\*:



\- If database corruption: Restore from backup

\- If cache issue: `redis-cli FLUSHDB` to clear cache

\- If malicious scanning: Enable rate limiting



\### High Latency



\*\*Symptoms\*\*: Alert "HighLatency" firing



\*\*Investigation\*\*:



1\. Check database connection pool: Are connections exhausted?

2\. Check Redis: `redis-cli INFO stats`

3\. Check CPU/memory: `kubectl top pods`



\*\*Resolution\*\*:



\- Scale up: `kubectl scale deployment url-shortener --replicas=5`

\- Check slow queries in PostgreSQL

\- Increase Redis memory if evicting too aggressively



\### Service Down



\*\*Symptoms\*\*: Alert "ServiceDown" firing, health check failing



\*\*Investigation\*\*:



1\. Check pod status: `kubectl get pods`

2\. Check recent deployments: `kubectl rollout history deployment/url-shortener`

3\. Check logs for crashes



\*\*Resolution\*\*:



\- Rollback: `kubectl rollout undo deployment/url-shortener`

\- Check resource limits if OOMKilled

\- Review recent code changes



\## Deployment Process



1\. \*\*Pre-deployment\*\*

&nbsp;  - Run full test suite

&nbsp;  - Check staging environment

&nbsp;  - Notify team in #deployments channel



2\. \*\*Deploy\*\*

&nbsp;  - Merge to main (triggers CI/CD)

&nbsp;  - Monitor deployment in GitHub Actions

&nbsp;  - Watch metrics dashboard



3\. \*\*Post-deployment\*\*

&nbsp;  - Verify health check returns 200

&nbsp;  - Test shorten/expand functionality

&nbsp;  - Monitor error rates for 15 minutes

&nbsp;  - If issues: rollback immediately



\## Database Maintenance



\*\*Backup\*\*: Automated daily backups to S3

\*\*Restore\*\*: `./scripts/restore\_db.sh <backup-date>`

\*\*Migrations\*\*: `alembic upgrade head`



\## Performance Tuning



\*\*Cache Hit Rate Target\*\*: >90%

\*\*P95 Latency Target\*\*: <100ms

\*\*Error Rate Target\*\*: <0.1%



Check current metrics: http://grafana.example.com/d/url-shortener

```



\### Implementation: Enterprise Features



\*\*Audit Logging\*\*:



```python

\# shortener/audit.py

import structlog

from datetime import datetime

from typing import Optional



audit\_logger = structlog.get\_logger("audit")



def log\_audit\_event(

&nbsp;   user\_id: Optional\[str],

&nbsp;   action: str,

&nbsp;   resource\_type: str,

&nbsp;   resource\_id: str,

&nbsp;   metadata: dict

):

&nbsp;   """Log audit trail for compliance"""

&nbsp;   audit\_logger.info(

&nbsp;       "audit\_event",

&nbsp;       timestamp=datetime.utcnow().isoformat(),

&nbsp;       user\_id=user\_id,

&nbsp;       action=action,

&nbsp;       resource\_type=resource\_type,

&nbsp;       resource\_id=resource\_id,

&nbsp;       metadata=metadata

&nbsp;   )

```



\*\*RBAC Implementation\*\*:



```python

\# shortener/rbac.py

from enum import Enum

from typing import Set



class Role(Enum):

&nbsp;   VIEWER = "viewer"

&nbsp;   CREATOR = "creator"

&nbsp;   ADMIN = "admin"



class Permission(Enum):

&nbsp;   VIEW\_URL = "view\_url"

&nbsp;   CREATE\_URL = "create\_url"

&nbsp;   DELETE\_URL = "delete\_url"

&nbsp;   VIEW\_ANALYTICS = "view\_analytics"



ROLE\_PERMISSIONS: dict\[Role, Set\[Permission]] = {

&nbsp;   Role.VIEWER: {Permission.VIEW\_URL, Permission.VIEW\_ANALYTICS},

&nbsp;   Role.CREATOR: {Permission.VIEW\_URL, Permission.CREATE\_URL, Permission.VIEW\_ANALYTICS},

&nbsp;   Role.ADMIN: set(Permission),  # All permissions

}



def has\_permission(user\_role: Role, required\_permission: Permission) -> bool:

&nbsp;   return required\_permission in ROLE\_PERMISSIONS.get(user\_role, set())

```



\### Grafana Dashboard Configuration



Create dashboards tracking:



\- Request rate (requests/sec)

\- Error rate by type (4xx, 5xx)

\- Latency percentiles (p50, p95, p99)

\- Cache hit rate

\- Database connection pool usage

\- Redis memory usage

\- Pod CPU/memory

\- Click count trends

\- Active URLs count



---



\## ML/AI Specific Additions



For machine learning applications, add these domain-specific considerations at appropriate phases:



\### Phase 3-4: ML Development



\- \*\*Model Versioning\*\*: Track experiments with MLflow, Weights \& Biases, or DVC

\- \*\*Data Versioning\*\*: Track dataset changes (DVC, LakeFS)

\- \*\*Experiment Tracking\*\*: Log hyperparameters, metrics, artifacts



\### Phase 5: ML Performance



\- \*\*Model Registry\*\*: Centralized model storage with metadata

\- \*\*Feature Store\*\*: Centralized feature engineering and serving

\- \*\*Model Optimization\*\*: Quantization, pruning for inference speed



\### Phase 6-7: ML Operations



\- \*\*A/B Testing\*\*: Compare model versions in production

\- \*\*Model Monitoring\*\*: Track drift, performance degradation, fairness metrics

\- \*\*Explainability\*\*: SHAP values, feature importance dashboards

\- \*\*Retraining Pipelines\*\*: Automated model retraining on new data



---



\## Decision Framework: Which Phase Do You Need?



Use this framework to determine your appropriate maturity level:



\### Choose Phase 1 if:



\- ✅ Learning Python or prototyping

\- ✅ One-off scripts or personal projects

\- ✅ No other users or dependencies



\### Choose Phase 2 if:



\- ✅ Code will be maintained over time

\- ✅ Sharing with teammates

\- ✅ Need to manage dependencies

\- ❌ Not deployed to production yet



\### Choose Phase 3 if:



\- ✅ Multiple developers collaborating

\- ✅ Code quality matters

\- ✅ Need confidence when refactoring

\- ❌ Not handling production traffic



\### Choose Phase 4 if:



\- ✅ Deploying to production

\- ✅ Multiple environments (dev/staging/prod)

\- ✅ Need to debug production issues

\- ❌ Performance not a concern yet



\### Choose Phase 5 if:



\- ✅ Performance is critical

\- ✅ High traffic or concurrent users

\- ✅ Need to scale horizontally

\- ✅ Ready for containerization



\### Choose Phase 6 if:



\- ✅ Frequent deployments required

\- ✅ Need zero-downtime updates

\- ✅ Infrastructure should be reproducible

\- ✅ Multiple production environments



\### Choose Phase 7 if:



\- ✅ Mission-critical system

\- ✅ SLA commitments to customers

\- ✅ Compliance requirements (SOC2, HIPAA, GDPR)

\- ✅ Multi-region availability needed

\- ✅ 24/7 on-call team



\## Related Resources



\### Python SWE Lab References



\- \[Code Organization](../patterns-and-practices/code-organization.md) - Module and package structure

\- \[Testing](../testing-and-quality/README.md) - Testing strategies and frameworks

\- \[Design Patterns](../patterns-and-practices/design-patterns.md) - Common design patterns

\- \[Performance Optimization](../performance-and-optimization/README.md) - Profiling and optimization

\- \[Tools and Ecosystem](../tools-and-ecosystem/README.md) - Development tools



\### External Resources



\- \[The Twelve-Factor App](https://12factor.net/) - Methodology for building SaaS apps

\- \[Site Reliability Engineering](https://sre.google/) - Google's SRE practices

\- \[Python Packaging User Guide](https://packaging.python.org/) - Comprehensive packaging guide



---



\## Summary Checklist



\### Phase 1: Foundational Code



\- \[ ] Working script

\- \[ ] CLI with error handling

\- \[ ] Basic functions and structure



\### Phase 2: Modular \& Installable



\- \[ ] Modular package with logging

\- \[ ] Configuration separated from code

\- \[ ] Installable via pip



\### Phase 3: Quality Engineering



\- \[ ] Comprehensive tests (70%+ coverage)

\- \[ ] Linting, formatting, type checking

\- \[ ] Pre-commit hooks

\- \[ ] Custom exceptions and error handling

\- \[ ] Structured logging



\### Phase 4: Production-Ready Config



\- \[ ] Environment-based configuration

\- \[ ] Secret management

\- \[ ] Configuration validation

\- \[ ] Structured logging to centralized system

\- \[ ] Metrics collection (Prometheus)

\- \[ ] Health check endpoints



\### Phase 5: Performance \& Scale



\- \[ ] Database with connection pooling

\- \[ ] Redis caching

\- \[ ] Async operations where appropriate

\- \[ ] Multi-stage Docker build

\- \[ ] Docker Compose for local dev

\- \[ ] Optimized container image



\### Phase 6: Infrastructure \& Deployment



\- \[ ] Infrastructure as Code (Terraform/CloudFormation)

\- \[ ] Auto-scaling configuration

\- \[ ] CI/CD pipeline

\- \[ ] Automated tests in pipeline

\- \[ ] Blue-green or canary deployments

\- \[ ] Database migration strategy



\### Phase 7: Operations \& Enterprise



\- \[ ] SLA/SLO definitions

\- \[ ] Alert rules configured

\- \[ ] Runbooks for common issues

\- \[ ] Incident response process

\- \[ ] Audit logging

\- \[ ] RBAC implementation

\- \[ ] Comprehensive documentation

\- \[ ] Multi-region if required



---



\*\*Remember\*\*: The goal is not to reach Phase 7 as quickly as possible. The goal is to be at the \_right\_ phase for your current needs, and to know how to progress when those needs change.



Start simple. Add complexity only when you feel the pain of not having it. Each phase should solve real problems you're experiencing, not hypothetical future problems.



