# Lesson 04 — Functions ⚙️

Functions are **reusable blocks of code** with a name. They take **inputs** (parameters) and
usually **return** an output.

---

## 📖 Concept

### Basic function

```python
def build_url(host: str, port: int) -> str:
    return f"http://{host}:{port}"

url = build_url("localhost", 8010)   # "http://localhost:8010"
```

### Default arguments

```python
def add_tool(name: str, server_id: str = None):   # server_id is optional
    ...
```

### `*args` and `**kwargs` — accept any number of arguments

```python
def wrapper(*args, **kwargs):
    # args  -> a tuple of positional arguments
    # kwargs-> a dict of keyword arguments
    ...
```

### Decorators — wrap a function to add behavior

A decorator is a function that takes another function and returns a new one. You apply it with
`@`:

```python
@log_me            # <-- decorator
def do_work():
    ...
```

### `async def` — asynchronous functions

Web servers handle many requests at once. `async def` lets a function **pause** (with `await`)
while waiting (e.g. for the database) so the server can do other work meanwhile.

```python
async def get_data():
    result = await db.query(...)   # pause here, don't block others
    return result
```

Your FastAPI endpoints and services are almost all `async def`.

---

## 🔎 In Your Project

**A real decorator with `*args/**kwargs`** (`logging_lib/decorators.py`):

```python
def log_me(func: Callable) -> Callable:
    """Decorator that logs the execution time of a method."""
    @functools.wraps(func)
    def wrapper(*args, **kwargs) -> Any:
        start_time = time.time()
        try:
            result = func(*args, **kwargs)
            return result
        finally:
            duration = (time.time() - start_time) * 1000
            # ... log the duration ...
    return wrapper
```

- `wrapper` accepts *any* arguments via `*args, **kwargs`, so it works on any function.
- `functools.wraps(func)` keeps the original function's name/docstring.
- `finally` (Lesson 07) always runs, so timing is always logged.

**An async function** (`app/api/services` style):

```python
async def get_checkpointer_text_data(self, thread_id, date):
    data = await self._load_from_db(thread_id, date)
    return data
```

---

## 🏋️ Exercise

In `playground.py`:

```python
import functools, time

# 1. Basic function with a default arg
def build_url(host: str, port: int = 8010) -> str:
    return f"http://{host}:{port}"

print(build_url("localhost"))          # uses default port
print(build_url("localhost", 9000))    # overrides it

# 2. Write your OWN simple decorator that prints before/after
def announce(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}...")
        result = func(*args, **kwargs)
        print("Done!")
        return result
    return wrapper

@announce
def add(a, b):
    return a + b

print(add(2, 3))

# 3. Challenge: add timing to `announce` (like log_me does with time.time()).
```

**Note on async:** to run an `async def` function you need `asyncio.run(my_async_fn())`. Try:

```python
import asyncio
async def hello():
    return "hi from async"
print(asyncio.run(hello()))
```

---

## 🔗 Project Connection

Your logging library (`log_me`) is a decorator applied across services. Every endpoint and
service method is `async def` so your server can handle many AI requests concurrently. Knowing
decorators and `async/await` is essential to reading almost any file in `app/api/`.

---

## ➡️ Next Step

Go to **[Lesson 05 — Modules & Packages](./05-modules-and-packages.md)**.

