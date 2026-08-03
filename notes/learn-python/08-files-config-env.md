# Lesson 08 — Files, Config & Environment 🗂️

Real apps read configuration from **environment variables** and **files** instead of hard-coding
secrets. Your project does this in `config.py`.

---

## 📖 Concept

### Reading environment variables

```python
import os

region = os.getenv("HOST_REGION")            # None if not set
port = os.getenv("DB_PORT", "5432")          # "5432" is the default
```

Environment variables are always **strings** — convert as needed (`int(...)`).

### `.env` files

During local development, you keep settings in a `.env` file. The `python-dotenv` library loads
them into the environment:

```python
from dotenv import load_dotenv
load_dotenv()          # now os.getenv() can see values from .env
```

### Reading/writing files

```python
with open("data.txt", "r") as f:      # "with" auto-closes the file
    content = f.read()

with open("out.txt", "w") as f:
    f.write("hello")
```

The `with` block guarantees the file closes even if an error happens.

### Checking a path exists

```python
if os.path.exists(env_path):
    ...
```

---

## 🔎 In Your Project

**Loading `.env` conditionally** (`app/api/config.py`):

```python
project_root = os.path.abspath(os.path.join(os.path.dirname(__file__), "..", "..", ".."))
env_path = os.path.join(project_root, ".env")

if os.path.exists(env_path):
    load_dotenv(dotenv_path=env_path, override=True)
else:
    load_dotenv(override=True)
```

- `os.path.dirname(__file__)` = the folder this file lives in.
- `os.path.join(...)` builds a cross-platform path (works on Windows & Linux).
- It checks the file exists before loading it.

**Validating config after loading** (`app/api/config.py`):

```python
config = Config.from_env()
if config.global_settings.region == "UNSET-REGION":
    raise ValueError("HOST_REGION is not set. Set it to eu1, us1, or ap1.")
if config.global_settings.region not in ["eu1", "us1", "ap1"]:
    raise ValueError(f"Invalid region '{config.global_settings.region}'.")
```

The app **refuses to start** with bad config — failing early and loudly is good design.

---

## 🏋️ Exercise

In `playground.py`:

```python
import os

# 1. Read an env var with a default
db_port = os.getenv("DB_PORT", "5432")
print("DB port:", db_port, "type:", type(db_port))   # note: it's a str!

# 2. Convert & validate
port_num = int(db_port)
if not (1 <= port_num <= 65535):
    raise ValueError("Port out of range")
print("Valid port:", port_num)

# 3. Write then read a file (like a mini log)
with open("learn-python/note.txt", "w") as f:
    f.write("I learned file I/O!\n")

with open("learn-python/note.txt", "r") as f:
    print(f.read())

# 4. Build a cross-platform path
here = os.path.dirname(os.path.abspath(__file__))
print("This script lives in:", here)
```

Set a temporary env var and re-run to see it change:

```powershell
$env:DB_PORT = "9999"; poetry run python learn-python/playground.py
```

**Challenge:** Read your project's `AGENTS.md` file and print how many lines it has
(`len(f.readlines())`).

---

## 🔗 Project Connection

`config.py` is the entry point for *all* settings in your app — database hosts, AWS region,
Redis, ports. Understanding `os.getenv`, `load_dotenv`, path building, and early validation lets
you safely add new settings.

---

## ➡️ Next Step

Go to **[Lesson 09 — Libraries & Frameworks](./09-libraries-frameworks.md)**.

