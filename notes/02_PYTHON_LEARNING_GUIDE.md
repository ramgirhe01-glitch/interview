# 🐍 Python Mastery: From Basics to This Project Level (Interview Ready)

## Table of Contents
1. [Python Fundamentals](#1-python-fundamentals)
2. [Object-Oriented Programming](#2-object-oriented-programming)
3. [Advanced Python](#3-advanced-python)
4. [Async Programming (Critical for this project)](#4-async-programming)
5. [Type Hints & Pydantic](#5-type-hints--pydantic)
6. [FastAPI Framework](#6-fastapi-framework)
7. [SQLAlchemy (Async ORM)](#7-sqlalchemy-async-orm)
8. [LangChain & LangGraph](#8-langchain--langgraph)
9. [Design Patterns in This Project](#9-design-patterns-in-this-project)
10. [Testing with Pytest](#10-testing-with-pytest)
11. [Interview Questions & Answers](#11-interview-questions--answers)
12. [Common Interview Coding Challenges](#12-common-interview-coding-challenges)

---

## 1. Python Fundamentals

### Variables & Data Types
```python
# Python is dynamically typed
name = "Agent"           # str
version = 1.0            # float
is_active = True         # bool
tools = ["search", "calc"]  # list
config = {"model": "claude"}  # dict
agent_ids = {uuid1, uuid2}   # set
coordinates = (10, 20)       # tuple (immutable)
```

### String Formatting (Used Everywhere in This Project)
```python
# f-strings (preferred in this project)
agent_id = "abc-123"
logger.log(f"Agent {agent_id} not found")  # ← Used in every file

# Format for error codes
code = f"cds.agenticai.{error_code}"  # ← Pattern from custom_exceptions.py
```

### Collections & Comprehensions
```python
# List comprehension (used in services)
tool_names = [tool["name"] for tool in tools_used if tool.get("name")]

# Dict comprehension
port_map = {f"server_{i}": port for i, port in enumerate(ports)}

# Generator expression (memory efficient)
total = sum(msg.token_count for msg in messages if msg.role == "assistant")
```

### Functions & Arguments
```python
# This project uses heavily:
def create_agent(
    name: str,                          # Positional
    *,                                   # Force keyword-only after this
    model: str = "claude-3",            # Default value
    tools: Optional[List[str]] = None,  # Optional with None default
    **kwargs                            # Catch-all keyword args
) -> AgentResponse:
    """Docstring - Google style used in project."""
    pass
```

### Exception Handling (Project Pattern)
```python
# This project's exception pattern:
try:
    result = await db_session.execute(query)
except SQLAlchemyError as e:
    logger.log(logging.ERROR, f"Database error: {e}", LOGGING_FILE)
    raise InternalServerException(
        code="databaseError",
        detail=f"Failed to fetch agent: {str(e)}"
    )
except Exception as e:
    # Generic catch - always last
    raise InternalServerException(code="internalError", detail=str(e))
```

### Interview Q&A: Fundamentals

**Q: What's the difference between `is` and `==`?**
> `==` checks value equality. `is` checks identity (same object in memory). `None` checks should always use `is None`.

**Q: What are mutable vs immutable types?**
> Mutable: list, dict, set. Immutable: str, int, float, tuple, frozenset. Important for function default arguments — never use `def f(x=[])`, use `def f(x=None)`.

**Q: Explain `*args` and `**kwargs`**
> `*args` collects extra positional args as tuple. `**kwargs` collects extra keyword args as dict. Used in this project for passing dynamic config to LLM providers.

---

## 2. Object-Oriented Programming

### Classes (As Used in This Project)
```python
# From agent_service.py - Real project pattern:
class AgentService:
    def __init__(self, llm_provider: LLMProvider | None = None):
        # Dependency injection pattern
        self._llm_provider = llm_provider or LangChainBedrockProvider()
        self.agent_creation = AgentCreation(self)  # Composition
        self.agent_execution = AgentExecution(self, self.agent_creation, execution_service)
    
    @staticmethod
    def get_inference_profile_prefix() -> Optional[str]:
        """Static method - no self needed."""
        pass
    
    async def execute_agent(self, agent, input_data, **kwargs):
        """Instance method - async!"""
        pass

# Singleton pattern used in this project:
agent_service = AgentService()  # Module-level singleton
```

### Inheritance (Exception Hierarchy)
```python
# From custom_exceptions.py:
class AgenticAIException(Exception):
    """Base exception for all custom exceptions."""
    def __init__(self, code: str, detail: str, status_code: int = 500):
        self.code = code
        self.detail = detail
        self.status_code = status_code
        super().__init__(self.detail)

class ResourceNotFoundException(AgenticAIException):
    """404 - Resource not found."""
    def __init__(self, code="resourceNotFound", detail="Not found"):
        super().__init__(code=code, detail=detail, status_code=404)

class BadRequestException(AgenticAIException):
    """400 - Invalid request."""
    def __init__(self, code="badRequest", detail="Invalid request"):
        super().__init__(code=code, detail=detail, status_code=400)
```

### Abstract Classes & Protocols
```python
# Hook pattern from this project:
class ExecutionHook:
    """Base class for execution lifecycle hooks."""
    async def before_run(self, request, context):
        pass  # Override in subclass
    
    async def after_run(self, request, context, state, response):
        pass

class ProgressHook(ExecutionHook):
    """Persists execution state."""
    async def after_run(self, request, context, state, response):
        await execution_service._store_execution_state(...)
```

### Dataclasses & Named Tuples
```python
from dataclasses import dataclass, field
from typing import Optional, Dict, Any

# Used in guardrail_service.py:
@dataclass
class GuardrailValidationResult:
    filtered_content: Optional[str] = None
    assessments: List[Dict[str, Any]] = field(default_factory=list)
```

### Interview Q&A: OOP

**Q: Explain `@staticmethod` vs `@classmethod` vs instance method.**
> - Instance method: Takes `self`, accesses instance state
> - `@classmethod`: Takes `cls`, accesses class state, can be factory method
> - `@staticmethod`: No `self`/`cls`, just logically belongs to the class (like `get_inference_profile_prefix`)

**Q: What is composition vs inheritance?**
> Composition: "has-a" (AgentService HAS an AgentCreation). Inheritance: "is-a" (ResourceNotFoundException IS AN AgenticAIException). This project prefers composition.

**Q: What's the Singleton pattern? How is it used here?**
> Single instance for entire app. In this project: `agent_service = AgentService()` at module level. Python's module import system ensures it's created once.

---

## 3. Advanced Python

### Decorators (Used Heavily)
```python
# Function decorator (common in FastAPI):
from functools import wraps

def log_execution(func):
    @wraps(func)
    async def wrapper(*args, **kwargs):
        logger.info(f"Executing {func.__name__}")
        result = await func(*args, **kwargs)
        logger.info(f"Completed {func.__name__}")
        return result
    return wrapper

# Pydantic field_validator (from models/agent.py):
class MCPToolSelection(BaseModel):
    toolNames: List[str]
    
    @field_validator('toolNames')
    @classmethod
    def validate_tool_names(cls, v):
        if '*' in v and len(v) > 1:
            raise ValueError("Wildcard cannot be mixed with specific names")
        return v
```

### Context Managers
```python
# Used for database sessions and MCP connections:
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # STARTUP
    await initialize_checkpointer()
    yield  # App runs here
    # SHUTDOWN
    await shutdown_checkpointer()

# Usage in MCP:
async with MultiServerMCPClient(servers) as client:
    tools = await load_mcp_tools(client)
```

### Generators & Iterators
```python
# SSE streaming uses async generators:
async def stream_agent_response(agent_app, state, config):
    async for event in agent_app.astream(state, config):
        if "messages" in event:
            yield f"data: {json.dumps(event)}\n\n"
    yield "data: [DONE]\n\n"
```

### Metaclasses & `__init_subclass__`
```python
# SQLAlchemy uses declarative base (metaclass magic):
Base = declarative_base()  # Creates metaclass

class AgentDB(Base):
    __tablename__ = "agents"  # Metaclass reads this
    id = Column(String(36), primary_key=True)
    name = Column(String(255))
```

### Enum Classes
```python
# From this project:
from enum import Enum

class DiscoveryMode(str, Enum):
    OFF = "OFF"
    ON = "ON"

class GovernanceMode(str, Enum):
    OFF = "OFF"
    APPROVAL = "APPROVAL"

class TenancyType(str, Enum):
    TENANT_SPECIFIC = "TENANT_SPECIFIC"
    CROSS_TENANT = "CROSS_TENANT"
```

### Interview Q&A: Advanced

**Q: What are decorators? How do they work?**
> Decorators wrap a function/class with additional behavior. They're syntactic sugar for `func = decorator(func)`. Used for logging, authentication, validation, caching.

**Q: Explain Python's GIL. How does async help?**
> GIL (Global Interpreter Lock) prevents true parallel threading for CPU-bound tasks. But for I/O-bound tasks (DB queries, HTTP calls, LLM inference), `asyncio` allows concurrent execution by switching between coroutines during I/O waits. This project is I/O-heavy, so async is critical.

**Q: What's the difference between `__str__` and `__repr__`?**
> `__str__` is for users (readable). `__repr__` is for developers (unambiguous, should be valid Python if possible). `SecurityContext.__repr__` in this project shows tenant info but hides the token.

---

## 4. Async Programming (Critical for this project)

### Why Async Matters Here
This project makes MANY I/O calls per request:
- Database queries (PostgreSQL)
- Redis cache lookups
- AWS Bedrock LLM calls (slow! 2-30 seconds)
- MCP server tool calls
- S3 operations
- SQS messages

Without async, each would block the entire server. With async, hundreds of requests run concurrently.

### Core Concepts
```python
import asyncio

# Coroutine (async function):
async def fetch_agent(agent_id: str) -> AgentResponse:
    async with self.SessionLocal() as session:
        result = await session.execute(select(AgentDB).where(AgentDB.id == agent_id))
        return result.scalar_one_or_none()

# Awaiting (pauses coroutine, lets others run):
agent = await fetch_agent("abc-123")

# Running multiple coroutines concurrently:
agent, tools, guardrails = await asyncio.gather(
    fetch_agent(agent_id),
    fetch_tools(agent_id),
    fetch_guardrails(agent_id),
)

# Background tasks (fire and forget):
bg_task = asyncio.create_task(
    self._execute_in_background(agent_app, state, ...)
)

# Async generators (SSE streaming):
async def stream():
    async for chunk in llm.astream(messages):
        yield chunk
```

### Async Patterns from This Project
```python
# Pattern 1: Async context manager for DB sessions
async with self.SessionLocal() as session:
    result = await session.execute(query)
    await session.commit()

# Pattern 2: Semaphore for rate limiting
_discovery_semaphore = asyncio.Semaphore(50)

async def discover_tools(server_url):
    async with _discovery_semaphore:  # Max 50 concurrent discoveries
        return await _fetch_tools(server_url)

# Pattern 3: Lock for thundering herd protection
_model_fetch_locks: Dict[str, asyncio.Lock] = {}

async def get_model_with_cache(model_id):
    lock = _get_model_fetch_lock(model_id)
    async with lock:
        # Only one coroutine fetches at a time
        cached = await redis.get(model_id)
        if cached:
            return cached
        result = await fetch_from_service(model_id)
        await redis.set(model_id, result)
        return result

# Pattern 4: Event for signaling (MCP session cleanup)
close_event = asyncio.Event()
close_event.set()  # Signal holder task to exit
await holder_task  # Wait for cleanup
```

### Interview Q&A: Async

**Q: What's the difference between threading and asyncio?**
> Threading: OS manages context switches, shared memory risks, GIL limits CPU parallelism. Asyncio: Single thread, cooperative multitasking, coroutines voluntarily yield at `await` points. Better for I/O-bound workloads (this project), lower overhead.

**Q: What happens if you call `await` on a non-coroutine?**
> TypeError. You can only await coroutines, Tasks, or Futures.

**Q: How does `asyncio.gather()` differ from sequential awaits?**
> `gather()` runs all coroutines concurrently. Sequential awaits run one after another. For 3 DB queries taking 100ms each: sequential = 300ms, gather = ~100ms.

**Q: What is `asyncio.create_task()` used for?**
> Schedules a coroutine to run in the background. Used in this project for async execution mode — returns immediately to client while agent runs in background.

---

## 5. Type Hints & Pydantic

### Python Type Hints (Used Everywhere)
```python
from typing import List, Optional, Dict, Any, Tuple, Union

# Basic type hints:
def process(name: str, count: int = 0) -> bool: ...

# Complex types:
def get_agents(
    tenant_id: str,
    page: int = 0,
    size: int = 100,
) -> Tuple[List[AgentResponse], Dict[str, int]]: ...

# Optional (can be None):
model: Optional[str] = None  # Same as: str | None

# Union types (Python 3.10+):
def get_provider(config: LLMProvider | None = None): ...

# TypedDict (structured dicts):
from typing_extensions import TypedDict
class GraphState(TypedDict):
    messages: List[AnyMessage]
    tools_used: List[Dict[str, str]]
```

### Pydantic Models (Request/Response Validation)
```python
from pydantic import BaseModel, Field, field_validator, model_validator

# Request model (from models/agent.py):
class AgentCreate(BaseModel):
    name: str = Field(..., min_length=1, max_length=255, description="Agent name")
    version: str = Field(..., description="Agent version")
    model: str = Field(..., description="LLM model ID")
    systemPrompt: str = Field(..., min_length=1, description="System instructions")
    temperature: Optional[float] = Field(0.7, ge=0.0, le=1.0)
    maxTokens: Optional[int] = Field(800, ge=1, le=200000)
    tools: Optional[ToolGroup] = None
    
    @field_validator('name')
    @classmethod
    def validate_name(cls, v):
        if not re.match(r'^[a-zA-Z0-9_-]+$', v):
            raise ValueError('Name must be alphanumeric')
        return v
    
    model_config = {
        "json_schema_extra": {
            "example": {
                "name": "my-agent",
                "version": "1.0",
                "model": "anthropic.claude-sonnet-4-5-20250929-v1:0",
                "systemPrompt": "You are helpful."
            }
        }
    }

# Response model:
class AgentResponse(BaseModel):
    id: str
    name: str
    version: str
    model: str
    createdAt: datetime
    updatedAt: datetime
    
    class Config:
        from_attributes = True  # Allows creating from SQLAlchemy objects

# Nested models:
class AgentResponseWrapper(BaseModel):
    data: AgentResponse
```

### Pydantic Settings (Configuration)
```python
from pydantic import BaseModel

class Config(BaseModel):
    database: DatabaseConfig
    aws: AWSConfig
    redis: RedisConfig
    
    @classmethod
    def from_env(cls) -> "Config":
        return cls(
            database=DatabaseConfig(
                host=os.getenv("DB_HOST", "localhost"),
                port=int(os.getenv("DB_PORT", "5432")),
            ),
            ...
        )

config = Config.from_env()  # Loaded once at module import
```

### Interview Q&A: Types & Pydantic

**Q: Why use Pydantic over plain dataclasses?**
> Pydantic provides: automatic validation, JSON serialization, field constraints, custom validators, OpenAPI schema generation. FastAPI relies on Pydantic for request parsing and response serialization.

**Q: What's the difference between `Field(...)` and `Field(None)`?**
> `Field(...)` means the field is REQUIRED (no default). `Field(None)` means it's optional with None as default.

**Q: How does Pydantic v2 differ from v1?**
> v2: Written in Rust (much faster), uses `model_validator`/`field_validator` decorators, `model_config` dict instead of `Config` class, `from_attributes` instead of `orm_mode`.

---

## 6. FastAPI Framework

### Core Concepts Used in This Project

```python
from fastapi import FastAPI, APIRouter, Depends, Query, HTTPException, Request
from fastapi.responses import JSONResponse, StreamingResponse
from fastapi.security import HTTPBearer

# App creation with lifespan:
app = FastAPI(
    title="Agentic AI Services",
    lifespan=lifespan,
    dependencies=[Depends(cleanup_mcp_sessions)],  # Global dependency
)

# Router pattern (each endpoint file):
router = APIRouter(prefix="/agents", tags=["Agents"])

@router.get("", response_model=AgentListResponseWrapper)
async def list_agents(
    page: int = Query(0, ge=0),           # Query parameter with validation
    size: int = Query(100, ge=1, le=200),
    token_info: TokenVerificationResult = Depends(  # Dependency injection
        verify_token_with_scope("aiservice.read")
    ),
):
    agents = await agent_service.get_agents(...)
    return AgentListResponseWrapper(data=agents)

@router.post("", status_code=201, response_model=AgentResponseWrapper)
async def create_agent(
    create_agent_data: CreateAgentRequestWrapper,  # Body auto-parsed by Pydantic
    token_info: TokenVerificationResult = Depends(
        verify_token_with_scope("aiservice.write")
    ),
    credentials: HTTPAuthorizationCredentials = Depends(security),
):
    result = await agent_service.create_agent(...)
    return AgentResponseWrapper(data=result)
```

### Dependency Injection (Key FastAPI Feature)
```python
# Security dependency (from this project):
def verify_token_with_scope(required_scope: str):
    """Returns a FastAPI dependency that validates JWT and scope."""
    async def verify(credentials: HTTPAuthorizationCredentials = Depends(HTTPBearer())):
        token = credentials.credentials
        # Verify JWT signature, expiry, audience
        # Check required scope exists
        return TokenVerificationResult(tenant, app_name, provider)
    return verify

# Usage in endpoint:
@router.get("/agents")
async def list_agents(
    token_info: TokenVerificationResult = Depends(verify_token_with_scope("aiservice.read"))
):
    tenant_id = token_info.tenant  # Extracted from JWT
```

### Middleware (ASGI Pattern)
```python
# Custom ASGI middleware (from this project):
class SecurityHeadersMiddleware:
    def __init__(self, app):
        self.app = app

    async def __call__(self, scope, receive, send):
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return

        async def send_with_headers(message):
            if message["type"] == "http.response.start":
                headers = MutableHeaders(scope=message)
                headers["X-Content-Type-Options"] = "nosniff"
                headers["X-Frame-Options"] = "SAMEORIGIN"
            await send(message)

        await self.app(scope, receive, send_with_headers)

# Register:
app.add_middleware(SecurityHeadersMiddleware)
```

### SSE (Server-Sent Events) Streaming
```python
# From this project - streaming agent responses:
from sse_starlette import ServerSentEvent

@router.post("/agents/{id}/execute")
async def execute_agent(..., response_mode: str = "sync"):
    if response_mode == "sse":
        return StreamingResponse(
            stream_generator(agent_app, state),
            media_type="text/event-stream"
        )
```

### Exception Handlers
```python
# From exception_handlers.py:
def register_exception_handlers(app: FastAPI):
    @app.exception_handler(AgenticAIException)
    async def handle_agentic_exception(request, exc):
        return JSONResponse(
            status_code=exc.status_code,
            content=exc.to_dict()
        )
    
    @app.exception_handler(RequestValidationError)
    async def handle_validation_error(request, exc):
        return JSONResponse(status_code=422, content={"errors": [...]})
```

### Interview Q&A: FastAPI

**Q: How does FastAPI achieve high performance?**
> Built on Starlette (ASGI), uses async/await natively, Pydantic v2 (Rust core) for fast serialization, uvicorn with multiple workers. Can handle thousands of concurrent connections.

**Q: What's the difference between `Depends()` and middleware?**
> Middleware: Runs on EVERY request (cross-cutting concerns like logging, CORS). Depends: Runs only for routes that declare it (targeted like auth, DB sessions). Dependencies can be nested and share state.

**Q: Explain FastAPI's request lifecycle.**
> Request → Middleware stack → Route matching → Dependency resolution → Request body parsing (Pydantic) → Handler function → Response serialization → Middleware stack (reverse) → Response.

---

## 7. SQLAlchemy (Async ORM)

### Model Definition (From This Project)
```python
from sqlalchemy import Column, String, Text, DateTime, JSON, Boolean, Float, Integer
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.dialects.postgresql import ARRAY

Base = declarative_base()

class AgentDB(Base):
    __tablename__ = "agents"

    id = Column(String(36), primary_key=True)
    tenant_id = Column(String(255))
    provider_tenant = Column(String(255))
    name = Column(String(255), nullable=False)
    model = Column(String(100), nullable=False)
    system_prompt = Column(Text, nullable=False)
    tools = Column(JSON)           # Stored as JSONB in Postgres
    guardrails = Column(JSON)
    steps = Column(JSON)
    temperature = Column(Float, default=0.7)
    max_tokens = Column(Integer, default=800)
    skills = Column(JSON, nullable=True)
    memory_ids = Column(ARRAY(String))  # PostgreSQL array type
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
```

### Async Queries (From This Project)
```python
from sqlalchemy import select, and_, or_, func
from sqlalchemy.ext.asyncio import AsyncSession

# SELECT with multi-tenant filter:
async def get_agent(self, agent_id, tenant_id, provider_tenant, application):
    async with self.SessionLocal() as session:
        query = select(AgentDB).where(
            and_(
                AgentDB.id == agent_id,
                AgentDB.tenant_id == tenant_id,
                AgentDB.provider_tenant == provider_tenant,
                AgentDB.application == application,
            )
        )
        result = await session.execute(query)
        return result.scalar_one_or_none()

# INSERT:
async def create_agent(self, agent_data):
    async with self.SessionLocal() as session:
        db_agent = AgentDB(id=str(uuid4()), **agent_data)
        session.add(db_agent)
        await session.commit()
        await session.refresh(db_agent)
        return db_agent

# Pagination:
async def list_agents(self, tenant_id, page=0, size=100):
    async with self.SessionLocal() as session:
        # Count total
        count_query = select(func.count()).select_from(AgentDB).where(...)
        total = (await session.execute(count_query)).scalar()
        
        # Fetch page
        query = select(AgentDB).where(...).offset(page * size).limit(size)
        results = (await session.execute(query)).scalars().all()
        
        return results, {"totalElements": total, "totalPages": ceil(total/size)}
```

### Interview Q&A: SQLAlchemy

**Q: What's the difference between `session.execute()` and `session.query()`?**
> `session.query()` is legacy (1.x style). `session.execute(select(...))` is 2.0 style — more explicit, works with async, and is the standard in this project.

**Q: How does connection pooling work?**
> SQLAlchemy maintains a pool of database connections. `pool_pre_ping=True` checks connections are alive before use. `pool_recycle=300` recreates connections after 5 minutes to avoid stale TCP connections.

---

## 8. LangChain & LangGraph

### LangChain Concepts (Used in This Project)
```python
from langchain_core.messages import HumanMessage, AIMessage, ToolMessage
from langchain_core.tools import BaseTool, StructuredTool
from langchain_aws import ChatBedrock

# Creating LLM instance:
llm = ChatBedrock(
    model_id="anthropic.claude-sonnet-4-5-20250929-v1:0",
    region_name="eu-central-1",
    model_kwargs={"temperature": 0.7, "max_tokens": 800},
)

# Binding tools to LLM:
llm_with_tools = llm.bind_tools(tools)

# Messages (conversation):
messages = [
    HumanMessage(content="Search for Python tutorials"),
    AIMessage(content="", tool_calls=[{"name": "search", "args": {"q": "Python"}}]),
    ToolMessage(content="Found 10 results...", tool_call_id="call_123"),
]
```

### LangGraph (State Machine for Agents)
```python
from langgraph.graph import StateGraph, END
from langgraph.prebuilt import create_react_agent
from typing_extensions import TypedDict

# Define state:
class GraphState(TypedDict):
    messages: list  # Conversation messages
    tools_used: list
    metrics: dict

# Build graph (simplified version of what AgentCreation does):
graph = StateGraph(GraphState)

# Add nodes:
graph.add_node("agent", agent_node)      # LLM reasoning
graph.add_node("tools", tool_node)       # Tool execution

# Add edges:
graph.add_edge("__start__", "agent")
graph.add_conditional_edges("agent", should_continue, {
    "tools": "tools",
    "end": END,
})
graph.add_edge("tools", "agent")

# Compile with checkpointer:
app = graph.compile(checkpointer=memory_checkpointer)

# Execute:
result = await app.ainvoke(
    {"messages": [HumanMessage(content="Hello")]},
    config={"configurable": {"thread_id": session_id}}
)
```

### How This Project Uses LangGraph
```
AgentCreation.build_agent_app():
1. Discover MCP tools → List[BaseTool]
2. Load KB tools → SearchKBTool instances
3. Load Skill tools → LoadSkillTool, LoadSkillResourceTool
4. Create LLM with middlewares (retry, summarization, call limit)
5. Build StateGraph:
   - agent_node: LLM decides what to do
   - tool_node: Executes selected tools
   - Conditional edges: Continue or END
6. Compile with checkpointer → AgentApp
7. Return compiled app ready for execution
```

### Interview Q&A: LangChain/LangGraph

**Q: What is a ReAct agent?**
> Reasoning + Acting. The LLM reasons about what to do, acts (calls a tool), observes the result, then reasons again. Loops until task is complete. LangGraph implements this as a state machine.

**Q: What's the difference between LangChain and LangGraph?**
> LangChain: Library of components (LLMs, tools, chains). LangGraph: Framework for building stateful, multi-step agent workflows as directed graphs. LangGraph uses LangChain components but adds state management, checkpointing, and complex routing.

**Q: What is a checkpointer?**
> Persists the state of a LangGraph execution. Enables: conversation memory (resume sessions), HITL (pause/resume), error recovery. This project uses AsyncPostgresSaver to store checkpoints in PostgreSQL.

---

## 9. Design Patterns in This Project

### 1. Repository Pattern (Services as Repositories)
```python
class AgentService:
    """Encapsulates all agent data access and business logic."""
    async def get_agents(self, tenant_id, ...): ...
    async def create_agent(self, agent_data, ...): ...
    async def execute_agent(self, agent, input_data, ...): ...
```

### 2. Strategy Pattern (LLM Providers)
```python
class LLMProvider:
    """Base interface for LLM providers."""
    async def create_llm(self, model_id, **kwargs): ...

class LangChainBedrockProvider(LLMProvider):
    """AWS Bedrock implementation."""
    async def create_llm(self, model_id, **kwargs):
        return ChatBedrock(model_id=model_id, ...)

class OpenAIProvider(LLMProvider):
    """OpenAI implementation (BYOM)."""
    async def create_llm(self, model_id, **kwargs):
        return ChatOpenAI(model=model_id, ...)
```

### 3. Builder Pattern (Agent App Construction)
```python
# AgentCreation.build_agent_app() builds complex objects step by step:
async def build_agent_app(self, agent, checkpointer, bearer_token, ...):
    tools = await self._resolve_tools(agent)          # Step 1
    llm = await self._create_llm(agent)               # Step 2
    llm = self._add_middlewares(llm)                   # Step 3
    graph = self._build_graph(llm, tools)              # Step 4
    app = graph.compile(checkpointer=checkpointer)     # Step 5
    return AgentApp(app=app, tools=tools)
```

### 4. Observer/Hook Pattern
```python
# Hooks observe execution lifecycle:
class AgentRunner:
    def __init__(self, hooks: List[ExecutionHook]):
        self.hooks = hooks
    
    async def run(self, request, context):
        for hook in self.hooks:
            await hook.before_run(request, context)
        
        result = await self._execute(request)
        
        for hook in self.hooks:
            await hook.after_run(request, context, result)
        
        return result
```

### 5. Factory Pattern (Database Connection)
```python
def get_async_db_session(db_type: DatabaseType, echo=False):
    """Factory: creates appropriate session based on config."""
    if db_type == DatabaseType.POSTGRES:
        dsn = _postgres_dsn()
    else:
        dsn = _mysql_dsn()
    engine = _get_or_create_async_engine(dsn, echo)
    return async_sessionmaker(engine)
```

---

## 10. Testing with Pytest

### Test Structure
```python
import pytest
from unittest.mock import AsyncMock, MagicMock, patch

# Async test:
@pytest.mark.asyncio
async def test_create_agent():
    # Arrange
    mock_session = AsyncMock()
    agent_data = AgentCreate(name="test", version="1.0", ...)
    
    # Act
    with patch.object(agent_service, 'SessionLocal', return_value=mock_session):
        result = await agent_service.create_agent(agent_data, "tenant1", "provider1", "app1")
    
    # Assert
    assert result.name == "test"
    mock_session.add.assert_called_once()
    mock_session.commit.assert_awaited_once()

# Fixture for mocked DB:
@pytest.fixture
def mock_db_session():
    session = AsyncMock()
    session.execute = AsyncMock(return_value=MagicMock(scalar_one_or_none=MagicMock(return_value=mock_agent)))
    return session
```

### Conftest Pattern (From This Project)
```python
# tests/conftest.py - Mocks external dependencies:
import sys
from unittest.mock import MagicMock

# Mock a2a module before any imports:
sys.modules['a2a'] = MagicMock()
sys.modules['mcp.shared'] = MagicMock()

@pytest.fixture(autouse=True)
def mock_aws_credentials(monkeypatch):
    monkeypatch.setenv("AWS_DEFAULT_REGION", "us-east-1")
    monkeypatch.setenv("AWS_ACCESS_KEY_ID", "testing")
```

---

## 11. Interview Questions & Answers

### Python Core

**Q: What are Python's memory management mechanisms?**
> Reference counting (primary), cyclic garbage collector (handles circular refs), memory pools (pymalloc for small objects). `del` decrements ref count; when it hits 0, memory is freed.

**Q: Explain Python's MRO (Method Resolution Order).**
> Uses C3 linearization. For multiple inheritance, Python creates a consistent order to search for methods. Check with `ClassName.__mro__`.

**Q: What are slots? When would you use them?**
> `__slots__` restricts instance attributes to a fixed set, saving memory (no `__dict__`). Use for classes with many instances. Not used in this project (flexibility preferred).

### FastAPI/Web

**Q: How would you handle rate limiting in FastAPI?**
> Use middleware or dependency: track requests per IP/token in Redis, return 429 when limit exceeded. This project uses semaphores for internal rate limiting (e.g., `_discovery_semaphore`).

**Q: How do you handle long-running tasks?**
> This project uses three approaches: 1) Background tasks (`asyncio.create_task`), 2) SSE streaming (real-time updates), 3) Polling (client checks `/executions/{id}` for status).

**Q: Explain CORS and why it matters.**
> Cross-Origin Resource Sharing. Browsers block requests from different origins by default. Backend must explicitly allow specific origins. This project allows only `https://cloud.eu1.sws.siemens.com`.

### Architecture

**Q: How would you design a multi-tenant system?**
> Every request carries tenant context (from JWT). All DB queries filter by tenant. Data isolation at query level (not separate DBs). This project uses `tenant_id + provider_tenant + application` triple.

**Q: How do you handle distributed tracing?**
> OpenTelemetry auto-instrumentation traces HTTP calls, DB queries, Redis ops. Correlation IDs (`logref`) link logs across services. This project uses `logging_lib` with MDC context.

**Q: Explain the difference between sync and async execution modes in this project.**
> Sync: Client waits for complete response. Async: Server returns `executionId` immediately, runs agent in background. Client polls `/executions/{id}` for results. SSE: Real-time streaming of partial results.

---

## 12. Common Interview Coding Challenges

### Challenge 1: Implement a simple async cache with TTL
```python
import asyncio
import time
from typing import Any, Optional

class AsyncTTLCache:
    def __init__(self, ttl_seconds: int = 300):
        self._cache: dict = {}
        self._ttl = ttl_seconds
        self._lock = asyncio.Lock()
    
    async def get(self, key: str) -> Optional[Any]:
        async with self._lock:
            if key in self._cache:
                value, timestamp = self._cache[key]
                if time.time() - timestamp < self._ttl:
                    return value
                del self._cache[key]
            return None
    
    async def set(self, key: str, value: Any):
        async with self._lock:
            self._cache[key] = (value, time.time())
```

### Challenge 2: Implement retry with exponential backoff
```python
import asyncio
import random

async def retry_with_backoff(
    func,
    max_retries: int = 3,
    initial_delay: float = 1.0,
    backoff_factor: float = 2.0,
    jitter: bool = True,
):
    """Retry pattern used throughout this project."""
    for attempt in range(max_retries + 1):
        try:
            return await func()
        except Exception as e:
            if attempt == max_retries:
                raise
            delay = initial_delay * (backoff_factor ** attempt)
            if jitter:
                delay += random.uniform(0, delay * 0.1)
            await asyncio.sleep(delay)
```

### Challenge 3: Implement a simple dependency injection container
```python
class Container:
    """Simplified version of FastAPI's DI."""
    _instances = {}
    _factories = {}
    
    @classmethod
    def register(cls, interface, factory):
        cls._factories[interface] = factory
    
    @classmethod
    def resolve(cls, interface):
        if interface not in cls._instances:
            factory = cls._factories.get(interface)
            if factory:
                cls._instances[interface] = factory()
        return cls._instances[interface]
```

---

## Quick Reference: This Project's Python Features

| Feature | Where Used | Why |
|---------|-----------|-----|
| `async/await` | All services | Non-blocking I/O |
| Pydantic models | `models/` | Request validation |
| SQLAlchemy async | `services/` | DB access |
| Type hints | Everywhere | IDE support, docs |
| Decorators | Validators, routes | Cross-cutting concerns |
| Context managers | DB sessions, MCP | Resource cleanup |
| Generators | SSE streaming | Memory-efficient streaming |
| Enums | Models | Type-safe constants |
| Dataclasses | Utils | Simple data containers |
| ABC/Protocols | Hooks, providers | Pluggable interfaces |
| f-strings | Logging, errors | String formatting |
| Comprehensions | Data transforms | Concise collection ops |
| `asyncio.gather` | Parallel fetches | Concurrent I/O |
| `asyncio.Lock` | Cache protection | Thread safety |
| `asyncio.Semaphore` | Rate limiting | Concurrency control |

---

**Master these concepts and you'll be ready for any Python backend developer interview, especially for AI/ML platform roles.**

