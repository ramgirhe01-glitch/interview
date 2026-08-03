# Lesson 07 — Error Handling 🛟

Things go wrong: databases disconnect, users send bad data, files are missing. **Error handling**
lets your program respond gracefully instead of crashing.

---

## 📖 Concept

### try / except

```python
try:
    number = int("hello")        # this raises a ValueError
except ValueError:
    number = 0                   # recover instead of crashing
```

### The full shape

```python
try:
    risky()
except SomeError as e:      # catch a specific error, `e` holds details
    handle(e)
except (TypeError, KeyError):   # catch multiple types
    ...
else:
    print("ran only if no error")
finally:
    cleanup()               # ALWAYS runs (success or failure)
```

### Raising your own errors

```python
if agent is None:
    raise ResourceNotFoundException(code="agentNotFound", detail="Agent not found")
```

**Golden rules:**
- Catch **specific** exceptions, not bare `except:`.
- Use `finally` for cleanup that must always happen (closing connections, logging time).
- Create **custom exceptions** so errors carry meaningful info.

---

## 🔎 In Your Project

**Nested try/except with fallbacks** (`app/api/services/tool_service.py`):

```python
def run_async(self, coro):
    try:
        loop = asyncio.get_running_loop()
    except RuntimeError:                       # no running loop -> try another way
        try:
            loop = asyncio.get_event_loop()
        except RuntimeError:                   # still none -> create one
            loop = asyncio.new_event_loop()
            asyncio.set_event_loop(loop)
        return loop.run_until_complete(coro)
```

This gracefully handles three possible situations instead of crashing.

**`finally` guarantees logging** (`logging_lib/decorators.py`):

```python
try:
    result = func(*args, **kwargs)
    return result
finally:
    duration = (time.time() - start_time) * 1000   # runs even if func raised
    # ... log duration ...
```

**Custom exception in action** (per `AGENTS.md`):

```python
raise ResourceNotFoundException(code="agentNotFound", detail=f"Agent {id} not found")
```

An exception handler later converts this into a clean JSON response like
`cds.agenticai.agentNotFound`.

---

## 🏋️ Exercise

In `playground.py`:

```python
# 1. Safe integer parsing
def safe_int(text, default=0):
    try:
        return int(text)
    except ValueError:
        return default

print(safe_int("42"))       # 42
print(safe_int("oops"))     # 0

# 2. Use finally
def do_work():
    try:
        print("working...")
        raise RuntimeError("boom")
    except RuntimeError as e:
        print("caught:", e)
    finally:
        print("cleanup always runs")

do_work()

# 3. Custom exception (reuse AppException from Lesson 06)
class AppException(Exception):
    def __init__(self, code, detail, status_code=500):
        super().__init__(detail)
        self.code, self.detail, self.status_code = code, detail, status_code

def find_agent(agent_id, agents):
    if agent_id not in agents:
        raise AppException("agentNotFound", f"Agent {agent_id} not found", 404)
    return agents[agent_id]

try:
    find_agent("x", {"a": 1})
except AppException as e:
    print(e.status_code, e.detail)

# 4. Challenge: wrap safe_int to also catch TypeError (e.g. safe_int(None)).
```

---

## 🔗 Project Connection

Your whole `exceptions/` package + `exception_handlers.py` is built on this. Services `raise`
custom exceptions; handlers catch them and return proper HTTP errors. `finally` guarantees your
`log_me` decorator always records timing.

---

## ➡️ Next Step

Go to **[Lesson 08 — Files, Config & Environment](./08-files-config-env.md)**.

