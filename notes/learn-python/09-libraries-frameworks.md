# Lesson 09 — Libraries & Frameworks 🧰

You rarely build everything from scratch. Your project stands on powerful libraries. Here's what
each does and how to read it.

---

## 📖 Concept: the big players in your repo

| Library | Role in your project |
|---------|----------------------|
| **FastAPI** | The web framework — defines HTTP endpoints (`@router.get`, `@router.post`). |
| **Pydantic** | Data validation — defines request/response shapes (`models/`). |
| **SQLAlchemy** | Talks to the database using Python classes instead of raw SQL. |
| **LangGraph / LangChain** | Orchestrate the AI agents and their message flow. |
| **python-dotenv** | Loads `.env` config (Lesson 08). |

They're installed and pinned via **Poetry** (`pyproject.toml` / `poetry.lock`).

---

## 🔎 In Your Project

### FastAPI endpoint (`app/api/endpoints/agents.py`)

```python
@router.get("/checkpointer/text")
async def get_checkpointer_text_history(
    thread_id: str = Query(..., description="Thread/session ID"),
    date: Optional[str] = Query(None, description="Optional ISO date filter"),
    token_info: TokenVerificationResult = Depends(
        verify_token_with_scope(f"{config.security.audience}.read")
    ),
):
    """Return checkpoint text history for a thread."""
    data = await agent_service.get_checkpointer_text_data(thread_id, date)
    return {"data": data}
```

What each piece means:
- `@router.get("/checkpointer/text")` → responds to `GET` requests at that URL.
- `Query(...)` → read a value from the URL query string; `...` means **required**.
- `Depends(verify_token_with_scope(...))` → run auth **before** the function; FastAPI injects
  the result. This is **dependency injection**.
- returns a dict → FastAPI turns it into JSON automatically.

### Pydantic model (`app/api/models/agent.py`)

```python
class MCPToolSelection(BaseModel):
    mcpServerId: str = Field(..., description="ID of the MCP server")
    toolNames: List[str] = Field(..., min_length=1, description="Tools to enable")

    @field_validator("toolNames")
    @classmethod
    def validate_tool_names(cls, v):
        if v and "*" in v and len(v) > 1:
            raise ValueError("Wildcard '*' must be the only element in toolNames.")
        return v
```

- Inheriting `BaseModel` gives you **automatic validation**. If a request is missing
  `mcpServerId`, Pydantic rejects it before your code runs.
- `Field(..., min_length=1)` → required, at least one item.
- `@field_validator` → custom validation rule.

### SQLAlchemy model (`app/api/services/tool_service.py`)

```python
class ToolDB(Base):
    __tablename__ = "tools"
    id = Column(String(36), primary_key=True)
    name = Column(String(255), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
```

Each class = one database table; each `Column` = one column. You query with Python, not SQL.

### LangGraph / LangChain (`app/api/services/execution_service.py`)

```python
from langgraph.types import Command
from langchain_core.messages import HumanMessage, AIMessage
from langgraph.checkpoint.postgres.aio import AsyncPostgresSaver
```

These provide message types (`HumanMessage`, `AIMessage`) and checkpointing (saving an agent's
progress so it can resume) — the heart of your agent execution.

---

## 🏋️ Exercise

You can experiment without running the whole server.

```python
# 1. Pydantic validation in isolation
from pydantic import BaseModel, Field, field_validator
from typing import List

class ToolSelection(BaseModel):
    server_id: str = Field(..., description="MCP server id")
    tool_names: List[str] = Field(..., min_length=1)

    @field_validator("tool_names")
    @classmethod
    def no_mixed_wildcard(cls, v):
        if "*" in v and len(v) > 1:
            raise ValueError("'*' must be the only element")
        return v

# valid
print(ToolSelection(server_id="s1", tool_names=["add", "subtract"]))

# invalid -> raises a validation error, try it:
# print(ToolSelection(server_id="s1", tool_names=["*", "add"]))
# print(ToolSelection(server_id="s1", tool_names=[]))
```

Run: `poetry run python learn-python/playground.py`

**Challenge:** Open `app/api/models/agent.py` and find another Pydantic model. Identify which
fields are required (`...`) vs optional (`Optional[...] = None`).

---

## 🔗 Project Connection

These four libraries *are* your architecture: FastAPI = the doors, Pydantic = the contracts,
SQLAlchemy = storage, LangGraph/LangChain = the AI engine. Reading any file now means
recognizing which library's patterns you're looking at.

---

## ➡️ Next Step

Go to **[Lesson 10 — Apply It: Build a Feature](./10-apply-to-project.md)** — the capstone.

