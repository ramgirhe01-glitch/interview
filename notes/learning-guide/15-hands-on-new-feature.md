# 15 — Hands-On: Add a New Feature (Step-by-Step)

> Walk-through of adding a real feature to this project, so you can replicate the pattern.

---

## 🎯 Scenario: Add a "Tags" Feature to Agents

We'll add the ability to tag agents with labels (e.g., "production", "experimental", "math").

---

## Step 1: Database Migration

Create a new migration file:

```sql
-- migrations/V10_30__add_tags_to_agents.sql

ALTER TABLE agents ADD COLUMN tags JSON DEFAULT '[]';

-- Add index for searching by tags (PostgreSQL)
CREATE INDEX idx_agents_tags ON agents USING GIN (tags);
```

**Naming rule:** `V{major}_{minor}__{description}.sql`  
The system auto-applies this on next startup.

---

## Step 2: Pydantic Model Update

Update the request/response models:

```python
# models/agent.py — Add to existing models

class AgentCreate(BaseModel):
    name: str
    system_prompt: str
    model: str = "claude-3-sonnet"
    tools: List[str] = []
    tags: List[str] = Field(default_factory=list, max_length=10)  # ← NEW
    
    @field_validator('tags')
    @classmethod
    def validate_tags(cls, v):
        """Each tag must be 1-30 chars, alphanumeric + hyphens"""
        for tag in v:
            if not re.match(r'^[a-z0-9-]{1,30}$', tag):
                raise ValueError(f'Invalid tag: {tag}. Use lowercase letters, numbers, hyphens.')
        return v

class AgentResponse(BaseModel):
    id: str
    name: str
    tags: List[str] = []  # ← NEW
    # ... rest of fields
```

---

## Step 3: Service Layer

Update the service to handle tags:

```python
# services/agent_service.py

async def create_agent(self, data: AgentCreate, tenant_id: str, ...):
    agent_id = str(uuid.uuid4())
    
    session_factory = await get_async_db_session(DatabaseType.POSTGRES)
    async with session_factory() as session:
        await session.execute(
            self.AgentTable.insert().values(
                id=agent_id,
                name=data.name,
                system_prompt=data.system_prompt,
                tags=json.dumps(data.tags),  # ← Store as JSON
                tenant_id=tenant_id,
                # ... other fields
            )
        )
        await session.commit()
    
    return {"id": agent_id, "name": data.name, "tags": data.tags}

# NEW: Filter agents by tag
async def list_agents_by_tag(self, tag: str, tenant_id: str, ...):
    session_factory = await get_async_db_session(DatabaseType.POSTGRES)
    async with session_factory() as session:
        # PostgreSQL JSON contains query
        result = await session.execute(
            select(self.AgentTable).where(
                and_(
                    self.AgentTable.c.tenant_id == tenant_id,
                    self.AgentTable.c.tags.contains(f'["{tag}"]'),
                )
            )
        )
        return [dict(row._mapping) for row in result.fetchall()]
```

---

## Step 4: Endpoint

Add or update the endpoint:

```python
# endpoints/agents.py

@router.get("/agents", response_model=List[AgentResponse])
async def list_agents(
    tag: Optional[str] = None,  # ← NEW query param: ?tag=production
    token: TokenPayload = Depends(verify_token),
):
    if tag:
        return await agent_service.list_agents_by_tag(
            tag=tag,
            tenant_id=token.tenant_id,
            provider_tenant=token.provider_tenant,
            application=token.application,
        )
    return await agent_service.list_agents(
        tenant_id=token.tenant_id,
        provider_tenant=token.provider_tenant,
        application=token.application,
    )
```

---

## Step 5: Unit Test

```python
# tests/unit/services/test_agent_service_tags.py
import pytest
from unittest.mock import AsyncMock, patch, MagicMock

class TestAgentTags:
    
    @pytest.mark.asyncio
    @patch('services.agent_service.get_async_db_session')
    async def test_create_agent_with_tags(self, mock_db):
        """Test creating an agent with tags"""
        # Arrange
        mock_session = AsyncMock()
        mock_session.execute = AsyncMock()
        mock_session.commit = AsyncMock()
        mock_db.return_value = AsyncMock(
            __aenter__=AsyncMock(return_value=mock_session),
            __aexit__=AsyncMock()
        )
        
        data = AgentCreate(
            name="Math Bot",
            system_prompt="You do math",
            tags=["math", "production"]
        )
        
        # Act
        result = await agent_service.create_agent(
            data=data,
            tenant_id="tenant-1",
            provider_tenant="provider-1",
            application="app-1"
        )
        
        # Assert
        assert result["tags"] == ["math", "production"]
        mock_session.execute.assert_called_once()
    
    def test_invalid_tag_raises_error(self):
        """Test that invalid tags are rejected by Pydantic"""
        with pytest.raises(ValidationError):
            AgentCreate(
                name="Bot",
                system_prompt="test",
                tags=["INVALID TAG WITH SPACES!"]  # Should fail
            )
```

---

## Step 6: Run & Verify

```powershell
# Run tests
poetry run pytest tests/unit/services/test_agent_service_tags.py -v

# Start the server
just dev

# Test via curl
curl -X POST http://localhost:8001/api/v1/agents \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <token>" \
  -d '{"name": "Math Bot", "system_prompt": "You do math", "tags": ["math", "production"]}'

# Filter by tag
curl http://localhost:8001/api/v1/agents?tag=production \
  -H "Authorization: Bearer <token>"
```

---

## 📋 Checklist for Any New Feature

- [ ] **Migration** — `migrations/V{x}_{y}__description.sql`
- [ ] **Model** — Add fields to Pydantic models in `models/`
- [ ] **Service** — Business logic in `services/`
- [ ] **Endpoint** — HTTP route in `endpoints/`
- [ ] **Tests** — Unit tests in `tests/unit/`
- [ ] **Validation** — Input constraints (Pydantic validators)
- [ ] **Multi-tenancy** — Filter by tenant_id in queries
- [ ] **Error handling** — Proper exception codes

---

## 🔄 File Touch Order

```
1. migrations/V10_30__add_tags.sql     (DB schema)
2. models/agent.py                      (Data shapes)
3. services/agent_service.py            (Logic)
4. endpoints/agents.py                  (HTTP route)
5. tests/unit/services/test_*.py        (Tests)
```

This is the same order every feature follows in this project.

---

> 💡 **Tip:** Look at recent Git commits to see how the team adds features. Each commit usually follows this exact pattern.

