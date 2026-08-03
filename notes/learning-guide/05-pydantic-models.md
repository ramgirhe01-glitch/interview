# 05 — Pydantic Models (Data Validation)

> Pydantic = Java DTOs + Bean Validation + Jackson combined. Used EVERYWHERE in this project.

---

## 🎯 What is Pydantic?

- **Data validation library** using Python type hints
- **Auto-validates** data on creation (like `@Valid` in Spring)
- **Auto-serializes** to/from JSON (like Jackson)
- **Generates JSON Schema** (for OpenAPI docs)
- Replaces: Java Records + Lombok + validation annotations

---

## 📋 Basic Model (Like a Java DTO)

```python
# models/agent.py
from pydantic import BaseModel, Field
from typing import Optional, List
from datetime import datetime

class AgentCreate(BaseModel):
    """Request body for creating an agent — auto-validated!"""
    
    name: str = Field(..., min_length=1, max_length=100)
    system_prompt: str = Field(..., min_length=1)
    model: str = Field(default="claude-3-sonnet")
    tools: List[str] = Field(default_factory=list)
    temperature: float = Field(default=0.7, ge=0.0, le=2.0)
    description: Optional[str] = None
```

**Java equivalent:**
```java
public record AgentCreate(
    @NotBlank @Size(max = 100) String name,
    @NotBlank String systemPrompt,
    String model,  // default "claude-3-sonnet"
    List<String> tools,
    @Min(0) @Max(2) Double temperature,
    @Nullable String description
) {}
```

---

## ✅ Validation Rules (Field Constraints)

```python
from pydantic import BaseModel, Field, field_validator
from typing import Optional

class AgentCreate(BaseModel):
    # Required field (... means required)
    name: str = Field(..., min_length=1, max_length=100, description="Agent name")
    
    # Optional with default
    model: str = Field(default="claude-3-sonnet", description="LLM model to use")
    
    # Numeric constraints
    temperature: float = Field(default=0.7, ge=0.0, le=2.0)
    max_tokens: int = Field(default=4096, gt=0, le=100000)
    
    # Optional (can be None)
    description: Optional[str] = Field(None, max_length=500)
    
    # List with constraints
    tools: List[str] = Field(default_factory=list, max_length=20)
```

### Custom Validators:
```python
from pydantic import field_validator, model_validator

class AgentCreate(BaseModel):
    name: str
    model: str
    tools: List[str] = []
    
    @field_validator('name')
    @classmethod
    def name_must_not_be_empty(cls, v: str) -> str:
        """Like @AssertTrue in Java"""
        if not v.strip():
            raise ValueError('Name cannot be blank')
        return v.strip()
    
    @field_validator('model')
    @classmethod
    def validate_model(cls, v: str) -> str:
        allowed = ['claude-3-sonnet', 'claude-3-haiku', 'gpt-4']
        if v not in allowed:
            raise ValueError(f'Model must be one of: {allowed}')
        return v
    
    @model_validator(mode='after')
    def check_consistency(self):
        """Cross-field validation (like class-level @Valid)"""
        if self.tools and not self.name:
            raise ValueError('Agent with tools must have a name')
        return self
```

---

## 📤 Response Models

```python
class AgentResponse(BaseModel):
    """What the API returns — controls serialization"""
    
    id: str
    name: str
    system_prompt: str
    model: str
    tools: List[str]
    created_at: datetime
    updated_at: Optional[datetime] = None
    
    class Config:
        from_attributes = True  # Can create from SQLAlchemy model (ORM mode)
    
    # Exclude sensitive fields from response
    # Use model_config to customize serialization
    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "id": "agent-123",
                    "name": "My Assistant",
                    "model": "claude-3-sonnet"
                }
            ]
        }
    }
```

### Usage in Endpoint:
```python
@router.get("/agents/{agent_id}", response_model=AgentResponse)
async def get_agent(agent_id: str):
    # FastAPI auto-serializes the return value using AgentResponse
    # Any extra fields are stripped, types are coerced
    return await agent_service.get(agent_id)
```

---

## 🔄 Nested Models

```python
class ToolConfig(BaseModel):
    name: str
    server_url: str
    auth_type: str = "none"

class GuardrailConfig(BaseModel):
    type: str  # "input" or "output"
    rule: str
    action: str = "block"

class AgentCreate(BaseModel):
    name: str
    tools: List[ToolConfig] = []          # Nested list of objects
    guardrails: List[GuardrailConfig] = []  # Another nested list
    metadata: Dict[str, Any] = {}          # Flexible dict
```

**Request JSON:**
```json
{
    "name": "My Agent",
    "tools": [
        {"name": "calculator", "server_url": "http://localhost:9000"}
    ],
    "guardrails": [
        {"type": "input", "rule": "no_profanity", "action": "block"}
    ]
}
```

---

## 🔀 Enums in Pydantic

```python
from enum import Enum

class AgentStatus(str, Enum):  # str + Enum = serializes as string
    ACTIVE = "active"
    INACTIVE = "inactive"
    DRAFT = "draft"

class ToolType(str, Enum):
    MCP = "mcp"
    FUNCTION = "function"
    API = "api"

class AgentCreate(BaseModel):
    name: str
    status: AgentStatus = AgentStatus.DRAFT
    tool_type: ToolType
```

---

## 🏭 Creating Instances

```python
# From dict (like JSON deserialization)
data = {"name": "Bot", "model": "claude-3-sonnet", "temperature": 0.5}
agent = AgentCreate(**data)  # Unpacks dict as keyword args
# OR
agent = AgentCreate.model_validate(data)  # Explicit validation

# From another object (ORM → Response)
db_agent = await db.query(AgentTable).get(id)
response = AgentResponse.model_validate(db_agent)  # from_attributes=True needed

# To dict
agent_dict = agent.model_dump()  # {"name": "Bot", "model": "claude-3-sonnet", ...}

# To JSON string
agent_json = agent.model_dump_json()  # '{"name":"Bot","model":"claude-3-sonnet",...}'

# Exclude None values
agent_dict = agent.model_dump(exclude_none=True)
```

---

## 📊 Discriminated Unions (Polymorphic Types)

```python
from typing import Literal, Union
from pydantic import BaseModel

class MCPTool(BaseModel):
    type: Literal["mcp"] = "mcp"
    server_url: str
    tool_name: str

class FunctionTool(BaseModel):
    type: Literal["function"] = "function"
    function_name: str
    module: str

# Union type — Pydantic picks the right one based on "type" field
ToolConfig = Union[MCPTool, FunctionTool]

class AgentCreate(BaseModel):
    name: str
    tools: List[ToolConfig]  # Can be mix of MCP and Function tools
```

---

## 🔗 Pydantic in This Project (Real Examples)

| File | Models | Purpose |
|------|--------|---------|
| `models/agent.py` | `AgentCreate`, `AgentUpdate`, `AgentResponse` | Agent CRUD |
| `models/tool.py` | `ToolCreate`, `ToolResponse`, `ToolType` | Tool management |
| `models/executions.py` | `ExecutionResponse`, `ExecutionNode` | Execution history |
| `models/guardrails.py` | `GuardrailConfig` | Guardrail rules |
| `models/custom_workflows.py` | `WorkflowCreate`, `WorkflowNode` | Workflow definitions |
| `config.py` | `DatabaseConfig`, `AWSConfig`, `RedisConfig` | App configuration |

---

## 💡 Key Differences from Java

| Java | Pydantic |
|------|----------|
| `@NotNull` | Field is required (no default) |
| `@Nullable` | `Optional[type] = None` |
| `@Size(min=1, max=100)` | `Field(min_length=1, max_length=100)` |
| `@Min(0) @Max(100)` | `Field(ge=0, le=100)` |
| `@Pattern(regexp)` | `Field(pattern="regex")` |
| `@Valid` (nested) | Automatic for nested BaseModel |
| `@JsonIgnore` | `Field(exclude=True)` |
| `@JsonProperty("name")` | `Field(alias="name")` |
| `ObjectMapper.readValue()` | `Model.model_validate(data)` |
| `ObjectMapper.writeValue()` | `model.model_dump_json()` |

---

## ▶️ Next: [06-sqlalchemy-database.md](./06-sqlalchemy-database.md) — Database layer with SQLAlchemy

