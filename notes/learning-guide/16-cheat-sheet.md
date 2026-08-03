# 16 — Quick Reference Cheat Sheet

> One-page reference. Print this or keep it open while coding.

---

## ⚡ Commands

```powershell
# Start dev server
just dev

# Run tests
poetry run pytest tests/unit/ -v

# Install dependencies
poetry install

# Add a package
poetry add <package-name>

# Start DB
just db-up

# Reset DB
just db-reset

# Format check
poetry run flake8 app/

# Single test
poetry run pytest tests/unit/services/test_agent_service.py -k "test_create" -v
```

---

## 📁 Where Things Live

| Need to... | Go to... |
|------------|----------|
| Add/change API route | `app/api/endpoints/` |
| Add business logic | `app/api/services/` |
| Add/change data model | `app/api/models/` |
| Change DB schema | `migrations/V{n}__desc.sql` |
| Add utility function | `app/api/utils/` |
| Add/change config | `app/api/config.py` + `.env` |
| Write tests | `tests/unit/services/` |
| Fix auth issues | `security_lib/` |
| Fix logging | `logging_lib/` |

---

## 🐍 Python Quick Syntax

```python
# Variable
name: str = "value"
items: list[str] = ["a", "b"]
config: dict[str, Any] = {"key": "value"}
maybe: Optional[str] = None

# Function
async def my_func(arg1: str, arg2: int = 10) -> dict:
    result = await some_async_call()
    return {"data": result}

# Class
class MyService:
    def __init__(self):
        self.cache = {}
    
    async def do_something(self, id: str) -> dict:
        ...

# Error handling
try:
    result = await risky_operation()
except NotFoundException as e:
    raise HTTPException(404, str(e))
except Exception as e:
    logger.error(f"Failed: {e}")
    raise

# List comprehension
names = [agent["name"] for agent in agents if agent["status"] == "active"]

# Dict from list
agent_map = {a["id"]: a for a in agents}
```

---

## 🌐 FastAPI Patterns

```python
# GET with query params
@router.get("/items")
async def list_items(page: int = 1, limit: int = 20):
    ...

# POST with body
@router.post("/items", status_code=201)
async def create_item(request: ItemCreate):
    ...

# Path + query + auth
@router.get("/items/{item_id}")
async def get_item(
    item_id: str,
    include_details: bool = False,
    token: dict = Depends(verify_token),
):
    ...

# Raise HTTP error
raise HTTPException(status_code=404, detail="Not found")
```

---

## 🗄️ Database Patterns

```python
# Query one
result = await session.execute(select(Table).where(Table.c.id == id))
row = result.fetchone()  # or scalar_one_or_none()

# Query many
result = await session.execute(select(Table).where(Table.c.status == "active"))
rows = result.fetchall()

# Insert
await session.execute(Table.insert().values(id=uuid4(), name="test"))
await session.commit()

# Update
await session.execute(
    Table.update().where(Table.c.id == id).values(name="new")
)
await session.commit()

# Delete
await session.execute(Table.delete().where(Table.c.id == id))
await session.commit()
```

---

## 🧪 Test Patterns

```python
import pytest
from unittest.mock import AsyncMock, patch, MagicMock

# Async test
@pytest.mark.asyncio
async def test_something():
    result = await my_async_function()
    assert result == expected

# Mock a dependency
@patch('services.my_service.external_call')
async def test_with_mock(mock_call):
    mock_call.return_value = {"data": "mocked"}
    result = await service.method()
    assert result["data"] == "mocked"
    mock_call.assert_called_once()

# Test exception
with pytest.raises(ResourceNotFoundException):
    await service.get("nonexistent-id")

# Fixture
@pytest.fixture
def sample_data():
    return {"id": "123", "name": "test"}
```

---

## 🔑 Import Patterns (This Project)

```python
# Services
from services.agent_service import agent_service
from services.tool_service import tool_service
from services.execution_service import execution_service

# Models
from models.agent import AgentCreate, AgentResponse, AgentUpdate
from models.tool import ToolCreate, ToolType

# Config
from config import config

# Database
from app.api.db import get_async_db_session, DatabaseType

# Exceptions
from exceptions.custom_exceptions import ResourceNotFoundException, ValidationException

# Logging
from logging_lib import logger

# Auth
from security_lib.auth import verify_token_with_scope

# FastAPI
from fastapi import APIRouter, Depends, HTTPException, Query
```

---

## 🎯 Error Code Format

```python
# Always: cds.agenticai.{camelCaseCode}
raise ResourceNotFoundException(code="agentNotFound", detail="Agent X not found")
# → {"error": "cds.agenticai.agentNotFound", "message": "Agent X not found"}

# Common codes:
# agentNotFound, toolNotFound, workflowNotFound
# invalidRequest, validationError
# executionFailed, internalError
# operationNotAllowed, unauthorized
```

---

## 📊 HTTP Status Codes Used

| Code | When |
|------|------|
| 200 | Success (GET, PUT) |
| 201 | Created (POST) |
| 204 | No content (DELETE) |
| 400 | Bad request (validation error) |
| 401 | Unauthorized (no/invalid token) |
| 403 | Forbidden (no permission) |
| 404 | Not found |
| 409 | Conflict (duplicate) |
| 500 | Internal server error |

---

## 🔄 Java → Python Quick Map

| Java | Python |
|------|--------|
| `null` | `None` |
| `true/false` | `True/False` |
| `this` | `self` |
| `new Object()` | `Object()` |
| `instanceof` | `isinstance(obj, Class)` |
| `object.toString()` | `str(object)` |
| `.equals()` | `==` |
| `System.out.println()` | `print()` |
| `String.format()` | `f"text {var}"` |
| `throws Exception` | (no declaration needed) |
| `final` | (use UPPER_CASE convention) |
| `private` | `_prefix` (convention only) |
| `interface` | `ABC` or `Protocol` |
| `@Override` | (just define method, no annotation) |
| `ArrayList` | `list` |
| `HashMap` | `dict` |
| `Optional<T>` | `Optional[T]` (from typing) |
| `var x = ...` | `x = ...` (always inferred) |

---

> 📌 Bookmark this file — you'll reference it daily!

