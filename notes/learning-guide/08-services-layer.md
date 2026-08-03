# 08 — Services Layer (Business Logic)

> Where all the real work happens. Like Spring `@Service` classes.

---

## 🎯 Service Layer Overview

```
endpoints/ (HTTP handling)
    │
    ▼ calls
services/ (ALL business logic lives here)
    │
    ├── agent_service.py          ← ⭐ Main service (4000+ lines!)
    ├── agent/                     ← Agent sub-modules
    │   ├── agent_creation.py     ← Build LangGraph agents
    │   ├── agent_execution.py    ← Run agents
    │   └── agent_models.py       ← Internal data models
    ├── tool_service.py            ← MCP tool management
    ├── execution_service.py       ← Track execution history
    ├── guardrail_service.py       ← Input/output guardrails
    ├── custom_workflow/           ← User-defined workflows
    ├── dynamic_workflow/          ← Supervisor-based workflows
    ├── memory_services/           ← Agent memory
    ├── skill_service.py           ← Uploaded skills
    └── ...
```

---

## 🏗️ Service Pattern (Singleton)

In Java, Spring manages service lifecycle. In this project, services are **module-level singletons**:

```python
# services/agent_service.py — Pattern used throughout

class AgentService:
    """Main agent service — CRUD + execution logic."""
    
    def __init__(self):
        self.agent_creation = AgentCreation(self)
        self.agent_execution = AgentExecution(self, self.agent_creation, execution_service)
    
    async def create_agent(self, data: AgentCreate, tenant_id: str, ...) -> dict:
        """Create a new agent configuration in DB."""
        ...
    
    async def get_agent(self, agent_id: str, tenant_id: str, ...) -> dict:
        """Retrieve agent by ID with tenant filtering."""
        ...
    
    async def execute_agent(self, agent_id: str, instruction: str, ...) -> dict:
        """Execute an agent with the given instruction."""
        ...

# ⭐ SINGLETON INSTANCE — imported by other modules
agent_service = AgentService()
```

**Java equivalent:**
```java
@Service
public class AgentService {
    // Spring creates single instance, injects where needed
}
```

**Python usage:**
```python
# In endpoints/agents.py
from services.agent_service import agent_service  # Import the singleton

@router.post("/agents/{agent_id}/execute")
async def execute(agent_id: str, request: ExecuteRequest):
    return await agent_service.execute_agent(agent_id, request.instruction)
```

---

## 📋 Key Services Explained

### 1. `agent_service.py` — The Main Service (⭐ Most Important)

**What it does:**
- Agent CRUD (Create, Read, Update, Delete)
- Agent execution orchestration
- LangGraph agent building
- Multi-tenant data access

**Key methods:**
```python
class AgentService:
    # CRUD
    async def create_agent(data, tenant_id, ...) -> dict
    async def get_agent(agent_id, tenant_id, ...) -> dict
    async def update_agent(agent_id, data, tenant_id, ...) -> dict
    async def delete_agent(agent_id, tenant_id, ...) -> None
    async def list_agents(tenant_id, ...) -> list
    
    # Execution
    async def execute_agent(agent_id, instruction, session_id, ...) -> dict
    async def execute_step_based_agent(...) -> dict
    async def run_agent_with_context(...) -> dict  # Used by workflows
```

### 2. `tool_service.py` — Tool Management

```python
class ToolService:
    """Manage MCP tools that agents can use."""
    
    async def create_tool(data, tenant_id, ...) -> dict
    async def validate_mcp_reachability(server_url) -> bool
    async def get_tools_for_agent(agent_id, ...) -> list
```

### 3. `execution_service.py` — Execution Tracking

```python
class ExecutionService:
    """Track and store execution history."""
    
    async def create_execution(agent_id, session_id, ...) -> dict
    async def update_execution_status(execution_id, status, ...) -> None
    async def get_execution(execution_id, ...) -> dict
    async def list_executions(agent_id, ...) -> list
```

### 4. `guardrail_service.py` — Safety Rules

```python
class GuardrailService:
    """Manage input/output guardrails for agents."""
    
    async def create_guardrail(data, ...) -> dict
    async def evaluate_input(input_text, guardrails) -> GuardrailResult
    async def evaluate_output(output_text, guardrails) -> GuardrailResult
```

### 5. `custom_workflow/` — User-Defined Workflows

```python
# custom_workflow_creation.py
class CustomWorkflowCreation:
    """Build LangGraph DAG from user-defined workflow config."""
    
    async def build_workflow_graph(workflow_config) -> CompiledGraph

# custom_workflow_execution.py
class CustomWorkflowExecution:
    """Execute a compiled workflow."""
    
    async def execute(workflow_graph, input_data) -> dict
```

### 6. `dynamic_workflow/` — Supervisor Workflows

```python
# dynamic_workflow_execution.py
class DynamicWorkflowExecution:
    """Supervisor-based dynamic agent routing."""
    
    async def execute(supervisor_config, sub_agents, instruction) -> dict
    # Supervisor LLM decides which sub-agent to call
```

---

## 🔄 Execution Flow (Tracing a Request)

### Example: `POST /agents/{id}/execute`

```
1. endpoints/agents.py
   └── execute_agent_endpoint(agent_id, request)
       
2. services/agent_service.py
   └── agent_service.execute_agent(agent_id, instruction, ...)
       ├── Validate agent exists (DB query)
       ├── Check governance (is execution allowed?)
       ├── Create execution record in DB
       │
       └── agent_execution.execute(...)
           
3. services/agent/agent_execution.py
   └── AgentExecution.execute(...)
       ├── agent_creation.build_agent_app(agent_config, tools)
       │   ├── Load LLM (Bedrock/OpenAI)
       │   ├── Load MCP tools
       │   └── Build LangGraph graph
       │
       └── Run the LangGraph agent
           ├── LLM processes instruction
           ├── If tool needed → call MCP server
           ├── If HITL needed → interrupt
           └── Return final response
           
4. Back to endpoint → Return HTTP response
```

---

## 🔧 Common Service Patterns

### Pattern 1: Database Query with Tenant Filtering
```python
async def get_agent(self, agent_id: str, tenant_id: str, provider_tenant: str, application: str):
    session_factory = await get_async_db_session(DatabaseType.POSTGRES)
    async with session_factory() as session:
        result = await session.execute(
            select(self.AgentTable).where(
                and_(
                    self.AgentTable.c.id == agent_id,
                    self.AgentTable.c.tenant_id == tenant_id,
                    self.AgentTable.c.provider_tenant == provider_tenant,
                    self.AgentTable.c.application == application,
                )
            )
        )
        row = result.fetchone()
        if not row:
            raise ResourceNotFoundException(
                code="agentNotFound",
                detail=f"Agent {agent_id} not found"
            )
        return row._mapping
```

### Pattern 2: Create with UUID
```python
async def create_agent(self, data: AgentCreate, tenant_id: str, ...):
    agent_id = str(uuid.uuid4())
    now = datetime.now(timezone.utc)
    
    session_factory = await get_async_db_session(DatabaseType.POSTGRES)
    async with session_factory() as session:
        await session.execute(
            self.AgentTable.insert().values(
                id=agent_id,
                name=data.name,
                system_prompt=data.system_prompt,
                tenant_id=tenant_id,
                created_at=now,
            )
        )
        await session.commit()
    
    return {"id": agent_id, "name": data.name, ...}
```

### Pattern 3: Error Handling
```python
from exceptions.custom_exceptions import (
    ResourceNotFoundException,
    ValidationException,
    InternalServerException,
)

async def execute_agent(self, agent_id: str, ...):
    # Validate
    agent = await self.get_agent(agent_id, ...)
    if not agent:
        raise ResourceNotFoundException(
            code="agentNotFound",
            detail=f"Agent {agent_id} not found"
        )
    
    if not agent.get("tools"):
        raise ValidationException(
            code="noToolsConfigured",
            detail="Agent has no tools configured"
        )
    
    try:
        result = await self._run_agent(agent, ...)
        return result
    except Exception as e:
        raise InternalServerException(
            code="executionFailed",
            detail=f"Agent execution failed: {str(e)}"
        )
```

---

## 🧩 Service Dependencies

```
agent_service ─────────┐
    ├── agent_creation  │
    ├── agent_execution │
    ├── execution_service ◄── tracks executions
    ├── tool_service ◄──── manages tools
    ├── guardrail_service ◄── evaluates guardrails
    └── memory_service ◄── manages agent memory

custom_workflows_service
    ├── agent_service ◄── runs individual agents
    └── execution_service

dynamic_workflow_service
    ├── agent_service ◄── runs sub-agents
    └── execution_service
```

---

## 💡 Key Takeaways

1. **Services are singletons** — one instance per module, imported elsewhere
2. **All DB access** goes through SQLAlchemy async sessions
3. **Multi-tenancy** — every query filters by tenant_id/provider_tenant/application
4. **Async everywhere** — all service methods are `async def`
5. **Exception handling** — custom exceptions with error codes
6. **agent_service.py** is the main file — start here when debugging

---

## ▶️ Next: [09-mcp-tools.md](./09-mcp-tools.md) — MCP tool integration

