# 04 — FastAPI Basics

> FastAPI is to Python what Spring Boot is to Java. This guide covers everything you need.

---

## 🎯 What is FastAPI?

- **Modern Python web framework** (like Spring Boot)
- **Async-first** (built on Starlette + Pydantic)
- **Auto-generates OpenAPI/Swagger docs**
- **Type-hint driven** — your type hints become API documentation
- **Very fast** — one of the fastest Python frameworks

---

## 🏗️ App Structure (in this project)

```python
# app/api/main.py — The application entry point

from fastapi import FastAPI

# Create the FastAPI app (like SpringApplication)
app = FastAPI(
    title="Agentic AI Service",
    version="1.0.0",
    docs_url="/docs",      # Swagger UI
    redoc_url="/redoc",    # ReDoc alternative
)

# Register routers (like Spring component scanning)
app.include_router(agents_router, prefix="/api/v1")
app.include_router(tools_router, prefix="/api/v1")
```

**Java equivalent:**
```java
@SpringBootApplication
public class AgenticAiServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(AgenticAiServiceApplication.class, args);
    }
}
```

---

## 🌐 Defining Endpoints (Routes)

### Basic CRUD Example

```python
# app/api/endpoints/agents.py
from fastapi import APIRouter, HTTPException, Depends
from models.agent import AgentCreate, AgentResponse

router = APIRouter(prefix="/agents", tags=["Agents"])

# GET /agents/{agent_id}  —  Like @GetMapping("/{id}")
@router.get("/{agent_id}", response_model=AgentResponse)
async def get_agent(agent_id: str):
    agent = await agent_service.get_agent(agent_id)
    if not agent:
        raise HTTPException(status_code=404, detail="Agent not found")
    return agent

# POST /agents  —  Like @PostMapping
@router.post("/", response_model=AgentResponse, status_code=201)
async def create_agent(request: AgentCreate):  # Auto-validated by Pydantic!
    return await agent_service.create(request)

# PUT /agents/{agent_id}  —  Like @PutMapping
@router.put("/{agent_id}", response_model=AgentResponse)
async def update_agent(agent_id: str, request: AgentUpdate):
    return await agent_service.update(agent_id, request)

# DELETE /agents/{agent_id}  —  Like @DeleteMapping
@router.delete("/{agent_id}", status_code=204)
async def delete_agent(agent_id: str):
    await agent_service.delete(agent_id)
```

**Java equivalent:**
```java
@RestController
@RequestMapping("/agents")
public class AgentController {
    
    @GetMapping("/{agentId}")
    public ResponseEntity<AgentResponse> getAgent(@PathVariable String agentId) { ... }
    
    @PostMapping
    public ResponseEntity<AgentResponse> createAgent(@Valid @RequestBody AgentCreate request) { ... }
}
```

---

## 📥 Request Parameters

### Path Parameters
```python
@router.get("/agents/{agent_id}/tools/{tool_id}")
async def get_tool(agent_id: str, tool_id: str):
    # agent_id and tool_id extracted from URL path
    pass
```

### Query Parameters
```python
@router.get("/agents")
async def list_agents(
    page: int = 1,                    # ?page=2
    limit: int = 20,                  # ?limit=50
    name: Optional[str] = None,       # ?name=MyAgent (optional)
    status: str = "active",           # ?status=active (default)
):
    pass
```

### Request Body (JSON)
```python
from pydantic import BaseModel

class AgentCreate(BaseModel):
    name: str
    system_prompt: str
    model: str = "claude-3-sonnet"  # default value
    tools: list[str] = []

@router.post("/agents")
async def create_agent(request: AgentCreate):  # Auto-parsed from JSON body
    # request.name, request.system_prompt, etc. — fully validated
    pass
```

### Headers
```python
from fastapi import Header

@router.get("/agents")
async def list_agents(
    authorization: str = Header(...),   # Required header
    x_tenant_id: str = Header(None),    # Optional header
):
    pass
```

---

## 🔌 Dependency Injection with `Depends()`

FastAPI's DI system (simpler than Spring's but very powerful):

```python
from fastapi import Depends

# 1. Define a dependency function
async def get_current_user(authorization: str = Header(...)) -> dict:
    """Verify JWT and return user info"""
    token = authorization.replace("Bearer ", "")
    user = verify_jwt(token)
    if not user:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user

# 2. Use it in endpoints
@router.get("/agents")
async def list_agents(user: dict = Depends(get_current_user)):
    # 'user' is automatically injected
    tenant_id = user["tenant_id"]
    return await agent_service.list_by_tenant(tenant_id)
```

### Dependency Chain (This Project's Pattern):
```python
# security_lib/auth.py
async def verify_token_with_scope(request: Request) -> TokenPayload:
    """Extract and verify JWT token"""
    ...

# endpoints/agents.py
@router.get("/agents")
async def list_agents(
    token: TokenPayload = Depends(verify_token_with_scope),  # Auth
    db: AsyncSession = Depends(get_db_session),              # Database
):
    return await agent_service.list(token.tenant_id, db)
```

---

## 🧱 Middleware (Like Spring Filters/Interceptors)

```python
# In main.py
from fastapi.middleware.cors import CORSMiddleware

# CORS middleware (like Spring's CorsConfiguration)
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Custom middleware
@app.middleware("http")
async def log_requests(request: Request, call_next):
    """Log every request (like Spring HandlerInterceptor)"""
    start_time = time.time()
    response = await call_next(request)
    duration = time.time() - start_time
    logger.info(f"{request.method} {request.url.path} - {response.status_code} ({duration:.2f}s)")
    return response
```

---

## ❌ Exception Handling

```python
# exceptions/custom_exceptions.py
class AgenticAIException(Exception):
    """Base exception for all service exceptions"""
    def __init__(self, code: str, detail: str, status_code: int = 500):
        self.code = code
        self.detail = detail
        self.status_code = status_code

class ResourceNotFoundException(AgenticAIException):
    def __init__(self, code: str, detail: str):
        super().__init__(code=code, detail=detail, status_code=404)

# Register exception handler in main.py
@app.exception_handler(AgenticAIException)
async def agentic_exception_handler(request: Request, exc: AgenticAIException):
    return JSONResponse(
        status_code=exc.status_code,
        content={"error": f"cds.agenticai.{exc.code}", "message": exc.detail}
    )
```

**Java equivalent:**
```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) { ... }
}
```

---

## 🔄 Lifespan Events (Startup/Shutdown)

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    """Run on startup and shutdown"""
    # STARTUP (like @PostConstruct)
    logger.info("Starting application...")
    await initialize_database()
    await connect_redis()
    
    yield  # Application runs here
    
    # SHUTDOWN (like @PreDestroy)
    logger.info("Shutting down...")
    await close_database()
    await disconnect_redis()

app = FastAPI(lifespan=lifespan)
```

---

## 📡 Server-Sent Events (SSE) — Streaming Responses

This project uses SSE for streaming agent execution:

```python
from sse_starlette.sse import EventSourceResponse

@router.post("/agents/{agent_id}/execute")
async def execute_agent(agent_id: str, request: ExecuteRequest):
    
    async def event_generator():
        """Stream execution events to client"""
        async for event in agent_service.execute_stream(agent_id, request):
            yield {
                "event": event.type,
                "data": json.dumps(event.data)
            }
    
    return EventSourceResponse(event_generator())
```

---

## 📋 Response Models

```python
from pydantic import BaseModel
from typing import Optional, List
from datetime import datetime

class AgentResponse(BaseModel):
    id: str
    name: str
    system_prompt: str
    model: str
    tools: List[str]
    created_at: datetime
    updated_at: Optional[datetime] = None

    class Config:
        from_attributes = True  # Allow creating from ORM objects
```

---

## 🔗 Key FastAPI Concepts Summary

| FastAPI | Spring Boot | This Project Location |
|---------|-------------|----------------------|
| `FastAPI()` | `SpringApplication` | `main.py` |
| `APIRouter` | `@RestController` | `endpoints/` |
| `@router.get/post` | `@GetMapping/@PostMapping` | `endpoints/*.py` |
| `Depends()` | `@Autowired` | Throughout |
| `BaseModel` | `@Valid @RequestBody DTO` | `models/` |
| `HTTPException` | `ResponseStatusException` | `services/` |
| `@app.middleware` | `Filter/Interceptor` | `main.py` |
| `lifespan` | `@PostConstruct/@PreDestroy` | `main.py` |
| `/docs` | Swagger UI (SpringDoc) | Auto-generated |

---

## ▶️ Next: [05-pydantic-models.md](./05-pydantic-models.md) — Data validation with Pydantic

