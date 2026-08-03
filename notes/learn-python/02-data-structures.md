# Lesson 02 — Data Structures 📦

---

## 📖 Concept

Python has 4 core ways to group values:

| Type | Looks like | Ordered? | Duplicates? | Changeable? | Use when… |
|------|-----------|----------|-------------|-------------|-----------|
| **list** | `[1, 2, 3]` | ✅ | ✅ | ✅ | a sequence you'll modify |
| **tuple** | `(1, 2, 3)` | ✅ | ✅ | ❌ | a fixed group |
| **dict** | `{"key": "val"}` | ✅* | keys unique | ✅ | label → value pairs |
| **set** | `{1, 2, 3}` | ❌ | ❌ | ✅ | unique items / fast "is it in here?" |

```python
tools = ["add", "subtract"]        # list
tools.append("multiply")           # -> ["add", "subtract", "multiply"]

point = (10, 20)                   # tuple (can't be changed)

server = {"name": "math", "port": 8010}   # dict
print(server["name"])              # -> "math"

seen = set()                       # set
seen.add("add")
print("add" in seen)               # -> True  (very fast lookup)
```

---

## 🔎 In Your Project

**Set + list working together** (`app/api/services/execution_service.py`):

```python
tools_used = []        # a list: the final answer, keeps order
seen_tools = set()     # a set: fast "have I seen this already?"

def add_tool(name, server_id=None):
    if name and name not in seen_tools:   # set makes this check fast
        seen_tools.add(name)
        tools_used.append({"name": name, "mcpServerId": server_id or ""})
```

This is a classic **deduplication** pattern: use a `set` to avoid duplicates, a `list` to keep
order, and each item is a `dict` with labeled fields.

**Building a list from a string** (`app/api/config.py`):

```python
kb_server_ports = [int(p) for p in os.getenv("KB_SERVER_PORTS", "8010,8011").split(",")]
# "8010,8011" -> ["8010","8011"] -> [8010, 8011]
```

(That `[... for ...]` is a *list comprehension* — you'll master it in Lesson 03.)

---

## 🏋️ Exercise

In `playground.py`:

```python
# 1. A list of tool names
tools = ["add", "subtract", "add", "multiply", "add"]

# 2. Use a set to find the UNIQUE tools
unique_tools = set(tools)
print(unique_tools)          # order not guaranteed

# 3. A dict describing an agent
agent = {"name": "Calculator", "model": "bedrock", "tools": tools}
print(agent["model"])

# 4. Add a new key
agent["enabled"] = True
print(agent)

# 5. Challenge: count how many times "add" appears WITHOUT a loop
print(tools.count("add"))
```

**Bonus:** Recreate the dedup pattern from your `execution_service.py` yourself using a `set`
and a `list`.

---

## 🔗 Project Connection

Your services constantly pass around lists of tools, dicts of agent config, and use sets to
avoid duplicates. Pydantic models (Lesson 09) are basically structured dicts with validation.

---

## ➡️ Next Step

Go to **[Lesson 03 — Control Flow](./03-control-flow.md)**.

