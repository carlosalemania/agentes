# Python Best Practices - Examples

> **Ejemplos de código Python moderno e idiomático**

---

## Ejemplo 1: Dataclasses vs Manual Class

### Before - Manual Class
```python
class User:
    def __init__(self, name, email, age=0):
        self.name = name
        self.email = email
        self.age = age

    def __repr__(self):
        return f"User(name={self.name!r}, email={self.email!r}, age={self.age!r})"

    def __eq__(self, other):
        if not isinstance(other, User):
            return False
        return (self.name, self.email, self.age) == (other.name, other.email, other.age)
```

### After - Dataclass
```python
from dataclasses import dataclass

@dataclass
class User:
    name: str
    email: str
    age: int = 0

# Automatic __init__, __repr__, __eq__, etc.
```

---

## Ejemplo 2: Context Manager

```python
from contextlib import contextmanager

@contextmanager
def temporary_config(key, value):
    """Temporarily set config value."""
    old_value = config.get(key)
    config[key] = value
    try:
        yield
    finally:
        if old_value is not None:
            config[key] = old_value
        else:
            del config[key]

with temporary_config('debug', True):
    # debug is True here
    pass
# debug restored
```

---

## Ejemplo 3: Async/Await

```python
import asyncio
import aiohttp

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

async def fetch_all(urls):
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        return await asyncio.gather(*tasks)

urls = ['https://example.com', 'https://python.org']
results = asyncio.run(fetch_all(urls))
```

---

**Versión:** 1.0.0
