# 13 — Common Patterns & Conventions

> Code patterns and conventions used throughout this project.

---

## 🏗️ Pattern 1: Singleton Services

Every service is a single instance created at module level:

```python
# services/tool_service.py
class ToolService:
    async def create_tool(self, ...): ...
    async def get_tool(self, ...): ...
    async def delete_tool(self, ...): ...

# Singleton created at bottom of file
tool_service = ToolService()
```

**Usage elsewhere:**
```python
from services.tool_service import tool_service

# Use directly — no DI container needed
result = await tool_service.get_tool(tool_id)
```

---

## 🏗️ Pattern 2: Multi-Tenant Queries

EVERY database query includes tenant filtering:

```python
async def get_resource(self, resource_id, tenant_id, provider_tenant, application):
    result = await session.execute(
        select(ResourceTable).where(
            and_(
                ResourceTable.c.id == resource_id,
                ResourceTable.c.tenant_id == tenant_id,
                ResourceTable.c.provider_tenant == provider_tenant,
                ResourceTable.c.application == application,
            )
        )
    )
    row = result.fetchone()
    if not row:
        raise ResourceNotFoundException(...)
    return row._mapping
```

---

## 🏗️ Pattern 3: Error Code Convention

All errors follow the format: `cds.agenticai.{code}`

```python
# Raising errors:
raise ResourceNotFoundException(
    code="agentNotFound",           # becomes cds.agenticai.agentNotFound
    detail="Agent abc-123 not found"
)

raise ValidationException(
    code="invalidToolConfig",
    detail="Tool server URL is not reachable"
)

raise InternalServerException(
    code="executionFailed",
    detail="LLM returned an error"
)
```

**JSON response:**
```json
{
    "error": "cds.agenticai.agentNotFound",
    "message": "Agent abc-123 not found"
}
```

---

## 🏗️ Pattern 4: Async Context Managers for DB

```python
# Getting a database session (async with = try-with-resources)
session_factory = await get_async_db_session(DatabaseType.POSTGRES)
async with session_factory() as session:
    # Auto-commits on success, auto-rollbacks on exception
    result = await session.execute(query)
    await session.commit()
```

---

## 🏗️ Pattern 5: UUID for All IDs

```python
import uuid

async def create_resource(self, data):
    resource_id = str(uuid.uuid4())  # Always generate UUID
    now = datetime.now(timezone.utc)  # Always UTC timestamps
    
    await session.execute(
        Table.insert().values(
            id=resource_id,
            created_at=now,
            ...
        )
    )
```

---

## 🏗️ Pattern 6: Logging with Source

```python
from logging_lib import logger
import logging

LOG_SOURCE = "services.agent_service"  # Module identifier

# Usage
logger.log(logging.INFO, "Creating agent", LOG_SOURCE)
logger.log(logging.ERROR, f"Failed to execute: {error}", LOG_SOURCE)
logger.log(logging.DEBUG, f"Agent config: {config}", LOG_SOURCE)
```

---

## 🏗️ Pattern 7: Configuration from Environment

```python
# config.py — All config reads from environment variables
class Config:
    database = DatabaseConfig(
        host=os.getenv("DB_HOST", "localhost"),
        port=int(os.getenv("DB_PORT", "3307")),
        username=os.getenv("DB_USERNAME", "admin"),
        password=os.getenv("DB_PASSWORD", ""),
        database=os.getenv("DB_NAME", "agenticconfig"),
    )
    aws = AWSConfig(
        region=os.getenv("AWS_DEFAULT_REGION", "us-east-1"),
    )

config = Config()  # Singleton, imported everywhere
```

**Usage:**
```python
from config import config

region = config.aws.region
db_host = config.database.host
```

---

## 🏗️ Pattern 8: Endpoint → Service → DB Flow

```python
# 1. Endpoint (thin — just HTTP handling)
@router.post("/agents", status_code=201)
async def create_agent(
    request: AgentCreate,
    token: TokenPayload = Depends(verify_token),
):
    result = await agent_service.create_agent(
        data=request,
        tenant_id=token.tenant_id,
        provider_tenant=token.provider_tenant,
        application=token.application,
    )
    return result

# 2. Service (all logic)
async def create_agent(self, data, tenant_id, provider_tenant, application):
    # Validate
    await self._validate_agent_name_unique(data.name, tenant_id, ...)
    
    # Create
    agent_id = str(uuid.uuid4())
    session_factory = await get_async_db_session(DatabaseType.POSTGRES)
    async with session_factory() as session:
        await session.execute(self.AgentTable.insert().values(...))
        await session.commit()
    
    return {"id": agent_id, "name": data.name, ...}
```

---

## 🏗️ Pattern 9: Optional Fields with Partial Updates

```python
# PATCH /agents/{id} — only update provided fields
class AgentUpdate(BaseModel):
    name: Optional[str] = None
    system_prompt: Optional[str] = None
    model: Optional[str] = None
    tools: Optional[List[str]] = None

async def update_agent(self, agent_id, data: AgentUpdate, ...):
    # Only build SET clause for non-None fields
    update_values = {}
    if data.name is not None:
        update_values["name"] = data.name
    if data.system_prompt is not None:
        update_values["system_prompt"] = data.system_prompt
    
    # Or more Pythonic:
    update_values = data.model_dump(exclude_none=True)
    
    if update_values:
        await session.execute(
            Table.update().where(Table.c.id == agent_id).values(**update_values)
        )
```

---

## 🏗️ Pattern 10: Governance Check Before Execution

```python
from services.governance_service import Operation

async def execute_agent(self, agent_id, instruction, ...):
    # Check if operation is allowed
    governance_result = await self._check_governance(
        operation=Operation.EXECUTE,
        agent_id=agent_id,
        tenant_id=tenant_id,
    )
    if not governance_result.allowed:
        raise ForbiddenException(
            code="operationNotAllowed",
            detail=governance_result.reason
        )
    
    # Proceed with execution
    ...
```

---

## 🏗️ Pattern 11: Background Task for Async Execution

```python
import asyncio

async def execute_agent_async(self, agent_id, instruction, ...):
    """Start execution in background, return immediately."""
    execution_id = str(uuid.uuid4())
    
    # Create execution record with "running" status
    await execution_service.create_execution(execution_id, status="running")
    
    # Schedule background task
    task = asyncio.create_task(
        self._run_agent_background(execution_id, agent_id, instruction)
    )
    
    # Return immediately with execution ID
    return {"execution_id": execution_id, "status": "running"}
```

---

## 🏗️ Pattern 12: Streaming with SSE

```python
from sse_starlette.sse import EventSourceResponse

@router.post("/agents/{agent_id}/execute/stream")
async def execute_stream(agent_id: str, request: ExecuteRequest):
    
    async def event_stream():
        async for chunk in agent_service.execute_streaming(agent_id, request):
            yield {
                "event": chunk.event_type,
                "data": json.dumps(chunk.data)
            }
    
    return EventSourceResponse(event_stream())
```

---

## 📝 Naming Conventions

| Type | Convention | Example |
|------|-----------|---------|
| Files | `snake_case.py` | `agent_service.py` |
| Classes | `PascalCase` | `AgentService` |
| Functions | `snake_case` | `create_agent()` |
| Variables | `snake_case` | `agent_id` |
| Constants | `UPPER_SNAKE` | `MAX_TOKENS` |
| Private | `_prefix` | `_internal_method()` |
| Test files | `test_*.py` | `test_agent_service.py` |
| Test funcs | `test_*` | `test_create_agent()` |

---

## ▶️ Next: [14-glossary.md](./14-glossary.md) — Terminology reference

