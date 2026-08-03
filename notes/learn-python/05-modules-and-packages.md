# Lesson 05 — Modules & Packages 📦🔌

As projects grow, you split code across many files. **Modules** and **packages** let files use
each other's code.

---

## 📖 Concept

- A **module** is a single `.py` file.
- A **package** is a folder containing an `__init__.py` file (which can be empty). It groups
  related modules.
- `import` brings code from one module into another.

```python
import json                          # whole standard-library module
import uuid

from datetime import datetime        # one name from a module
from typing import List, Optional    # multiple names

import numpy as np                   # import with an alias

from app.api.services import agent_service   # from YOUR package
```

**Why so many `from app.api...` imports?** Your project is one big package. `app/api/services`
is a sub-package. Because each folder has `__init__.py`, Python treats them as importable.

Rule of thumb:
- **Standard library** (built into Python): `os`, `json`, `datetime`, `asyncio`.
- **Third-party** (installed via Poetry): `fastapi`, `pydantic`, `sqlalchemy`, `langgraph`.
- **Your own code**: `app.api...`, `logging_lib`, `security_lib`.

---

## 🔎 In Your Project

**Mixed imports** (`app/api/services/agent_service.py`):

```python
import json
import uuid
import asyncio
from datetime import datetime, timezone
from typing import List, Optional, Dict, Any, Awaitable
from pydantic import ValidationError                 # third-party
from sqlalchemy.dialects.postgresql import ARRAY     # third-party
from app.api.services import agent_registry_service  # your own package
from fastapi import HTTPException                     # third-party
from logging_lib import logger                        # your own package
```

Notice the ordering convention: **standard library** first, then **third-party**, then
**local** imports. This is a widely used style (enforced by tools like `isort`).

**Registering a new router** (from `AGENTS.md` — "New Endpoint Pattern"):

```python
# app/api/endpoints/__init__.py exposes routers
# app/api/main.py imports and includes them
```

---

## 🏋️ Exercise

1. Create a new file `learn-python/mymodule.py`:

```python
# mymodule.py
def greet(name: str) -> str:
    return f"Hello, {name}!"

PI = 3.14159
```

2. In `playground.py`, import and use it:

```python
from mymodule import greet, PI      # only works if run from learn-python/ folder

print(greet("Python learner"))
print(PI)
```

Run from the folder:

```powershell
cd learn-python
poetry run python playground.py
cd ..
```

3. **Challenge:** Open `app/api/services/agent_service.py` and list which of its imports are
   (a) standard library, (b) third-party, (c) your own code. Write your answer as comments.

---

## 🔗 Project Connection

When you add a feature (per `AGENTS.md`), you create a router in `endpoints/`, a service in
`services/`, register it in `endpoints/__init__.py`, and import it in `main.py`. That whole
flow **is** modules and packages in action.

---

## ➡️ Next Step

Go to **[Lesson 06 — OOP: Classes & Inheritance](./06-oop-classes.md)**.

