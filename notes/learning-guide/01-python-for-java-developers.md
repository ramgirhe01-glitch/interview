# 01 — Python for Java Developers

> This guide maps Python concepts to Java equivalents you already know.

---

## 📦 Basic Syntax Differences

### Variables & Types

```java
// Java
String name = "Agent";
int count = 5;
final double PI = 3.14;
List<String> items = new ArrayList<>();
```

```python
# Python — no type declarations needed (but you CAN add type hints)
name = "Agent"          # str (like String)
count = 5               # int
PI = 3.14               # float (no "final", use UPPER_CASE convention)
items: list[str] = []   # type hint (optional but used in this project)
```

### Type Hints (Used Heavily in This Project)

```python
from typing import Optional, List, Dict, Any

def create_agent(name: str, tools: List[str], config: Optional[Dict[str, Any]] = None) -> str:
    """Create an agent and return its ID."""
    return str(uuid.uuid4())
```

**Java equivalent:**
```java
public String createAgent(String name, List<String> tools, @Nullable Map<String, Object> config) {
    return UUID.randomUUID().toString();
}
```

---

## 🏛️ Classes & OOP

### Basic Class

```java
// Java
public class Agent {
    private String id;
    private String name;
    
    public Agent(String id, String name) {
        this.id = id;
        this.name = name;
    }
    
    public String getName() { return name; }
}
```

```python
# Python
class Agent:
    def __init__(self, id: str, name: str):  # Constructor = __init__
        self.id = id        # No private keyword, use _prefix convention
        self.name = name
    
    @property             # Like a getter
    def display_name(self) -> str:
        return self.name.upper()
```

### Inheritance

```python
# Python inheritance
class BaseService:
    def __init__(self, db_session):
        self.db = db_session
    
    async def get_by_id(self, id: str):
        raise NotImplementedError  # Like Java abstract method

class AgentService(BaseService):  # extends BaseService
    async def get_by_id(self, id: str):
        return await self.db.query(Agent).get(id)
```

### No Interfaces — Use Abstract Base Classes or Protocols

```python
from abc import ABC, abstractmethod

# Like a Java interface
class Repository(ABC):
    @abstractmethod
    async def find_by_id(self, id: str) -> Any:
        pass
    
    @abstractmethod
    async def save(self, entity: Any) -> None:
        pass
```

---

## ⚡ Async/Await (Like CompletableFuture but simpler)

This project is **heavily async**. In Java you'd use `CompletableFuture` or reactive; in Python it's `async/await`:

```java
// Java (CompletableFuture)
CompletableFuture<Agent> future = agentService.findById(id);
Agent agent = future.get();  // blocks
```

```python
# Python (async/await) — used EVERYWHERE in this project
async def execute_agent(agent_id: str, instruction: str) -> dict:
    agent = await agent_service.get_agent(agent_id)      # non-blocking wait
    result = await run_langgraph(agent, instruction)      # non-blocking wait
    return result

# You MUST use 'await' when calling async functions
# You MUST mark a function 'async' if it uses 'await' inside
```

### Key Rules:
1. `async def` = this function is a coroutine (runs in event loop)
2. `await` = pause here until result is ready (non-blocking)
3. You can only `await` inside an `async def` function
4. FastAPI handles the event loop for you

---

## 📂 Modules & Imports (Like Java Packages)

```java
// Java
import com.myapp.services.AgentService;
import com.myapp.models.Agent;
```

```python
# Python — files ARE modules, folders with __init__.py ARE packages
from app.api.services.agent_service import agent_service  # import specific thing
from app.api.models.agent import AgentCreate, AgentResponse  # multiple imports
import logging  # import whole module
```

### Project Import Style (This Project):
```python
# Relative imports (within same package)
from .agent_creation import AgentCreation
from ..utils.mcp_tool_cache import mcp_tool_cache

# Absolute imports (from project root)
from app.api.services.agent_service import agent_service
from config import config
```

---

## 🗃️ Data Structures

| Java | Python | Example |
|------|--------|---------|
| `HashMap<K,V>` | `dict` | `{"name": "bot", "id": 1}` |
| `ArrayList<T>` | `list` | `[1, 2, 3]` or `["a", "b"]` |
| `HashSet<T>` | `set` | `{1, 2, 3}` |
| `Tuple (record)` | `tuple` | `(1, "hello", True)` |
| `null` | `None` | `if result is None:` |
| `true/false` | `True/False` | Capital letters! |

### Dictionary Operations (Most Common in This Project)

```python
# Creating
agent_config = {
    "name": "MyAgent",
    "model": "claude-3",
    "tools": ["calculator", "search"]
}

# Accessing
name = agent_config["name"]           # Throws KeyError if missing
name = agent_config.get("name", "")   # Returns "" if missing (safe)

# Checking
if "tools" in agent_config:
    print(agent_config["tools"])

# Unpacking (very common in this project)
def process(**kwargs):  # accepts any keyword arguments as dict
    tenant_id = kwargs.get("tenant_id")
```

---

## 🎭 Decorators (Like Java Annotations)

Decorators are **EVERYWHERE** in this project:

```java
// Java annotations
@RestController
@RequestMapping("/agents")
@Transactional
public class AgentController { ... }
```

```python
# Python decorators — same concept!
@router.post("/agents")          # Like @PostMapping
@require_auth                    # Like @Secured
async def create_agent(request: AgentCreate):
    ...

# Common decorators in this project:
@router.get("/agents/{agent_id}")      # HTTP GET endpoint
@router.post("/agents")                # HTTP POST endpoint
@pytest.mark.asyncio                   # Mark test as async
@property                              # Getter method
```

### How Decorators Work:
```python
# A decorator is just a function that wraps another function
def log_execution(func):
    async def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = await func(*args, **kwargs)
        print(f"Done {func.__name__}")
        return result
    return wrapper

@log_execution
async def my_function():
    pass
# Same as: my_function = log_execution(my_function)
```

---

## 🏗️ Dependency Injection (Simpler Than Spring)

Java uses `@Autowired` or constructor injection with a DI container. Python/FastAPI uses `Depends()`:

```java
// Java Spring
@Service
public class AgentService {
    @Autowired
    private AgentRepository repository;
}
```

```python
# Python FastAPI — Depends() for DI
from fastapi import Depends

async def get_db():
    """Dependency that provides database session"""
    session = await get_async_db_session()
    try:
        yield session  # 'yield' = provide value, then cleanup after
    finally:
        await session.close()

@router.get("/agents/{agent_id}")
async def get_agent(
    agent_id: str,
    db: AsyncSession = Depends(get_db),       # Injected!
    token: dict = Depends(verify_token),       # Injected!
):
    return await agent_service.get(agent_id, db)
```

### Singleton Pattern (Used in This Project):
```python
# services/agent_service.py — bottom of file
# No DI container needed, just module-level instance
agent_service = AgentService()  # Singleton! Imported by other modules
```

---

## 🧪 Exception Handling

```java
// Java
try {
    agent = service.find(id);
} catch (NotFoundException e) {
    throw new ResponseStatusException(HttpStatus.NOT_FOUND, e.getMessage());
} finally {
    cleanup();
}
```

```python
# Python
try:
    agent = await service.find(id)
except ResourceNotFoundException as e:  # 'as' instead of variable declaration
    raise HTTPException(status_code=404, detail=str(e))
except (ValueError, TypeError) as e:   # catch multiple
    raise HTTPException(status_code=400, detail=str(e))
finally:
    await cleanup()
```

---

## 📝 String Formatting (f-strings)

```python
# f-strings — most common in this project (Python 3.6+)
agent_id = "abc-123"
message = f"Agent {agent_id} not found"  # Agent abc-123 not found
url = f"http://{host}:{port}/api/v1/agents/{agent_id}"

# Multi-line f-strings
query = f"""
    SELECT * FROM agents 
    WHERE tenant_id = '{tenant_id}'
    AND id = '{agent_id}'
"""
```

---

## 🔄 List Comprehensions (Very Pythonic — No Java Equivalent)

```python
# Instead of:
tool_names = []
for tool in agent.tools:
    tool_names.append(tool.name)

# Python way (list comprehension):
tool_names = [tool.name for tool in agent.tools]

# With filter:
active_tools = [t for t in agent.tools if t.is_active]

# Dict comprehension:
tool_map = {t.id: t.name for t in agent.tools}
```

---

## 📋 Context Managers (Like Java try-with-resources)

```java
// Java
try (Connection conn = dataSource.getConnection()) {
    // use conn
}  // auto-closed
```

```python
# Python
async with get_connection() as conn:
    # use conn
# auto-closed

# The 'with' statement calls __enter__/__exit__ (or __aenter__/__aexit__ for async)
```

---

## 🎯 Key Python Concepts Used in This Project

| Concept | Where Used | Quick Explanation |
|---------|-----------|-------------------|
| `async/await` | Everywhere | Non-blocking I/O |
| `Pydantic BaseModel` | models/ | Data validation classes |
| `SQLAlchemy` | db.py, services | ORM (like Hibernate) |
| `Depends()` | endpoints/ | Dependency injection |
| `yield` | Dependencies, generators | Lazy evaluation / cleanup |
| `**kwargs` | Many functions | Capture extra keyword args as dict |
| `*args` | Some functions | Capture extra positional args as tuple |
| `@property` | Models | Computed getter |
| `__init__.py` | Every package | Makes folder a Python package |
| `Optional[X]` | Type hints | Value can be X or None |

---

## ▶️ Next: [02-project-setup.md](./02-project-setup.md) — Set up your local environment

