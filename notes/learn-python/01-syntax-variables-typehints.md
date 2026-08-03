# Lesson 01 — Syntax, Variables & Type Hints ✍️

---

## 📖 Concept

**Variables** store values. You don't declare a type — Python figures it out:

```python
name = "math-server"     # str  (text)
port = 8010              # int  (whole number)
strength = 0.8           # float (decimal)
is_ready = True          # bool (True/False)
nothing = None           # None (absence of a value)
```

**Type hints** are optional labels that say what type a variable/parameter *should* be. Python
doesn't enforce them at runtime, but they make code readable and let tools catch bugs:

```python
port: int = 8010
name: str = "math-server"

def greet(user: str) -> str:   # takes a str, returns a str
    return "Hello " + user
```

- `user: str` → the parameter `user` should be a string.
- `-> str` → the function returns a string.

Your codebase uses type hints **everywhere**. That's professional Python.

---

## 🔎 In Your Project

**Type conversion from environment variables** (`app/api/config.py`):

```python
max_mcp_servers = int(os.getenv("MAXIMUM_MCP_SERVERS_PER_AGENT", "5"))
```

Environment variables always arrive as **strings** (`"5"`), so `int(...)` converts `"5"` → `5`.
The second argument `"5"` is a **default** used if the variable isn't set.

**Type hints on a function** (from your services):

```python
def add_tool(name: Optional[str], server_id: Optional[str] = None):
    ...
```

- `Optional[str]` means "a string **or** `None`".
- `server_id: Optional[str] = None` gives it a **default value** of `None`.

---

## 🏋️ Exercise

In `learn-python/playground.py`, write:

```python
# 1. Create variables describing an MCP server
server_name: str = "math-server"
server_port: int = 8010
enabled: bool = True

# 2. Convert a string to a number (like config.py does)
raw = "42"
number = int(raw)
print(number + 8)          # should print 50

# 3. Write a typed function
def describe(name: str, port: int) -> str:
    return f"{name} runs on port {port}"

print(describe(server_name, server_port))
```

Run it:

```powershell
poetry run python learn-python/playground.py
```

**Challenge:** What happens if you write `int("hello")`? Try it and read the error.

---

## 🔗 Project Connection

Your `config.py` reads dozens of settings from the environment and converts them to the right
types (`int`, list, etc.). Understanding `int(...)`, defaults, and `Optional` lets you read and
extend that file confidently.

---

## ➡️ Next Step

Go to **[Lesson 02 — Data Structures](./02-data-structures.md)** to learn how Python groups many
values together.

