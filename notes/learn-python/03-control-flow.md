# Lesson 03 — Control Flow 🔀

Control flow decides **which** code runs and **how many times**.

---

## 📖 Concept

### if / elif / else — make decisions

```python
if strength > 0.8:
    level = "HIGH"
elif strength > 0.5:
    level = "MEDIUM"
else:
    level = "LOW"
```

Indentation (4 spaces) defines what's "inside" the block. Python has no `{}` braces.

### for — repeat over items

```python
for tool in ["add", "subtract"]:
    print(tool)
```

### while — repeat until a condition is false

```python
count = 0
while count < 3:
    count += 1
```

### Comprehensions — build a list/dict in one line

```python
ports = [int(p) for p in "8010,8011".split(",")]     # list comprehension
squares = [n * n for n in range(5)]                  # [0,1,4,9,16]
evens = [n for n in range(10) if n % 2 == 0]         # with a filter
```

Read it as: *"give me `int(p)` **for** each `p` **in** the split string."*

---

## 🔎 In Your Project

**Multi-condition `if`** (`app/api/db.py`) — decide whether Postgres is configured:

```python
if (
    config.postgres.host
    and config.postgres.port
    and config.postgres.username
    and config.database.database
):
    return f"postgresql+psycopg://{config.postgres.username}:..."
return None
```

All four must be truthy (`and`) before building the connection string.

**Loop + conditionals** (`app/api/services/execution_service.py`):

```python
for msg in messages:
    tool_calls_list = None
    if hasattr(msg, "tool_calls") and isinstance(getattr(msg, "tool_calls"), list):
        tool_calls_list = getattr(msg, "tool_calls")
    elif isinstance(msg, dict) and isinstance(msg.get("tool_calls"), list):
        tool_calls_list = msg.get("tool_calls")
```

This loops over messages and handles **two shapes** of data (an object vs. a dict).

**List comprehension** (`app/api/config.py`):

```python
kb_server_ports = [int(p) for p in os.getenv("KB_SERVER_PORTS", "8010,8011").split(",")]
```

---

## 🏋️ Exercise

In `playground.py`:

```python
# 1. Classify guardrail strength (like a real filter in your models)
for strength in [0.2, 0.6, 0.9]:
    if strength > 0.8:
        print(strength, "-> HIGH")
    elif strength > 0.5:
        print(strength, "-> MEDIUM")
    else:
        print(strength, "-> LOW")

# 2. Comprehension: parse ports
ports = [int(p) for p in "8010,8011,8012".split(",")]
print(ports)

# 3. Filter: keep only even ports
even_ports = [p for p in ports if p % 2 == 0]
print(even_ports)

# 4. Challenge: given tools = ["add","","multiply",None,"add"]
#    build a list of only the truthy, unique tool names.
tools = ["add", "", "multiply", None, "add"]
# your code here
```

---

## 🔗 Project Connection

Your `db.py` uses `if` to pick a database; `execution_service.py` loops over AI messages to
extract which tools were used; `config.py` uses comprehensions to parse settings. These three
patterns show up constantly.

---

## ➡️ Next Step

Go to **[Lesson 04 — Functions](./04-functions.md)**.

