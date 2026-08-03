# Lesson 10 — Apply It: Build a Real Feature 🚀

Time to combine everything. This capstone walks you through adding a small feature to your
project the way a professional would, reusing every concept from Lessons 01–09.

> ⚠️ This lesson is a **guided plan**, not code to blindly paste. Do it step by step, testing as
> you go. Start on a new git branch: `git checkout -b learn/health-feature`.

---

## 🎯 The feature: a `/health/details` endpoint

A simple endpoint that reports whether key config is present (region, DB host, Redis host).
It touches: type hints, dicts, control flow, functions, modules, a Pydantic model, error
handling, config reading, and FastAPI — all ten lessons.

---

## Step 1 — Design the response shape (Pydantic — Lesson 09 & 02)

Create `app/api/models/health.py`:

```python
from pydantic import BaseModel, Field

class HealthDetails(BaseModel):
    """Response describing the service's configuration health."""
    region: str = Field(..., description="Configured host region")
    database_configured: bool = Field(..., description="Is a DB host set?")
    redis_configured: bool = Field(..., description="Is Redis host set?")
    status: str = Field(..., description="'ok' or 'degraded'")
```

*Concepts:* class + inheritance (`BaseModel`), type hints, `Field`.

---

## Step 2 — Write the logic (functions, dict, control flow — Lessons 02–04)

Create `app/api/services/health_service.py`:

```python
from app.api.config import config
from app.api.models.health import HealthDetails


class HealthService:
    """Reports configuration health of the service."""

    def get_details(self) -> HealthDetails:
        region = config.global_settings.region
        db_ok = bool(config.database.host)          # truthy check -> bool
        redis_ok = bool(config.redis.host)

        # control flow: overall status
        status = "ok" if (db_ok and redis_ok and region != "UNSET-REGION") else "degraded"

        return HealthDetails(
            region=region,
            database_configured=db_ok,
            redis_configured=redis_ok,
            status=status,
        )


# singleton (Lesson 06) — created once, imported everywhere
health_service = HealthService()
```

*Concepts:* class + singleton, type hints, `bool()`, conditional expression, config reading.

> 🔎 Check the exact attribute names in your `app/api/config.py` (e.g. `config.database.host`,
> `config.redis.host`) and adjust if they differ. This is real debugging practice!

---

## Step 3 — Expose the endpoint (FastAPI — Lesson 09)

Create `app/api/endpoints/health.py`:

```python
from fastapi import APIRouter
from app.api.services.health_service import health_service
from app.api.models.health import HealthDetails

router = APIRouter(prefix="/health", tags=["health"])


@router.get("/details", response_model=HealthDetails)
async def health_details() -> HealthDetails:
    """Return configuration health details."""
    return health_service.get_details()
```

*Concepts:* imports/modules, `async def`, FastAPI routing, response model.

---

## Step 4 — Register the router (modules — Lesson 05)

Following your `AGENTS.md` "New Endpoint Pattern":

1. Add to `app/api/endpoints/__init__.py` so the router is exported.
2. In `app/api/main.py`, import it and call `app.include_router(health_router)`.

Look at how existing routers (e.g. `agents`) are wired and copy that pattern exactly.

---

## Step 5 — Add error handling (Lesson 07)

Make the service defensive:

```python
    def get_details(self) -> HealthDetails:
        try:
            region = config.global_settings.region
        except AttributeError:
            region = "UNSET-REGION"
        # ...rest as before...
```

---

## Step 6 — Run and test it

```powershell
# start the dev server
just dev
# or: poetry run uvicorn app.api.main:app --reload --host 127.0.0.1 --port 8001
```

Then open in your browser: `http://127.0.0.1:8001/docs` — find `GET /health/details`, click
**Try it out**, and **Execute**. You should see your JSON response.

---

## Step 7 — Write a unit test (Testing — from AGENTS.md)

Create `tests/unit/services/test_health_service.py`:

```python
from app.api.services.health_service import health_service


def test_health_details_returns_status():
    details = health_service.get_details()
    assert details.status in ("ok", "degraded")
    assert isinstance(details.region, str)
```

Run it:

```powershell
poetry run pytest tests/unit/services/test_health_service.py -v
```

---

## ✅ Concept checklist — you just used ALL of them

| Lesson | Where you used it |
|--------|-------------------|
| 01 Types | type hints on every function |
| 02 Data structures | dict/bool fields in the model |
| 03 Control flow | the `status = "ok" if ... else "degraded"` |
| 04 Functions | `get_details`, `async def health_details` |
| 05 Modules | imports + registering the router |
| 06 OOP | `HealthService` class + singleton |
| 07 Errors | `try/except AttributeError` |
| 08 Config | reading from `config` |
| 09 Frameworks | FastAPI + Pydantic |
| 10 Apply | this whole feature 🎉 |

---

## 🏁 Where to go next

1. **Extend it:** add `mcp_configured` and `postgres_configured` fields.
2. **Read real code:** now re-read `app/api/endpoints/agents.py` — it should make far more sense.
3. **Pick a real task:** find a small improvement in your repo and try it on a branch.
4. **Deepen fundamentals:** the official tutorial at https://docs.python.org/3/tutorial/ and
   FastAPI docs at https://fastapi.tiangolo.com/tutorial/.

You've gone from Python basics to shipping a feature in your own production-style codebase.
Keep coding a little every day. 💪

