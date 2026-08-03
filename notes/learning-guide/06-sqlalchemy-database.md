# 06 — SQLAlchemy & Database Layer

> SQLAlchemy = Python's Hibernate/JPA. This project uses it for all database operations.

---

## 🎯 What is SQLAlchemy?

- **ORM (Object Relational Mapper)** — maps Python classes to database tables
- **Like Hibernate/JPA** for Java
- Supports: PostgreSQL, MySQL, SQLite, Oracle, MSSQL
- This project uses **async SQLAlchemy** (non-blocking DB operations)

---

## 🗄️ Database Setup (db.py in This Project)

```python
# app/api/db.py — Simplified view
from sqlalchemy.ext.asyncio import async_sessionmaker, create_async_engine

# Create engine (like DataSource in Java)
engine = create_async_engine(
    "postgresql+psycopg://user:pass@localhost:5432/dbname",
    pool_pre_ping=True,    # Check connection before use
    pool_recycle=300,       # Recycle connections every 5 min
    echo=False,            # Set True to see SQL queries
)

# Session factory (like EntityManagerFactory)
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)
```

### This Project's DB Selection:
```python
# db.py supports both PostgreSQL and MySQL
class DatabaseType(enum.Enum):
    POSTGRES = "postgres"
    MYSQL = "mysql"

# Config determines which one to use (DB_TYPE env var)
```

---

## 📋 Defining Tables (Like JPA @Entity)

```python
# In agent_service.py — this project defines tables inline
from sqlalchemy import Column, String, Text, DateTime, JSON, Boolean, Float, Integer
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Agent(Base):
    """Database table mapping — like @Entity in JPA"""
    __tablename__ = "agents"
    
    # Columns — like @Column in JPA
    id = Column(String(36), primary_key=True)           # @Id
    name = Column(String(100), nullable=False)           # @Column(nullable=false)
    system_prompt = Column(Text, nullable=False)
    model = Column(String(50), default="claude-3-sonnet")
    temperature = Column(Float, default=0.7)
    tools = Column(JSON, default=list)                   # JSON column (like @Type(JsonType))
    tenant_id = Column(String(100), nullable=False)      # Multi-tenancy
    provider_tenant = Column(String(100), nullable=False)
    application = Column(String(100), nullable=False)
    status = Column(String(20), default="active")
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, onupdate=datetime.utcnow)
```

**Java JPA equivalent:**
```java
@Entity
@Table(name = "agents")
public class Agent {
    @Id
    private String id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(columnDefinition = "TEXT")
    private String systemPrompt;
    
    @Type(JsonType.class)
    private List<String> tools;
}
```

---

## 🔍 Querying (Like JPA Criteria API)

### Basic Queries:

```python
from sqlalchemy import select, and_, or_

# SELECT * FROM agents WHERE id = ?
async def get_by_id(session, agent_id: str):
    result = await session.execute(
        select(Agent).where(Agent.id == agent_id)
    )
    return result.scalar_one_or_none()  # Returns Agent or None

# SELECT * FROM agents WHERE tenant_id = ? AND status = 'active'
async def list_active(session, tenant_id: str):
    result = await session.execute(
        select(Agent).where(
            and_(
                Agent.tenant_id == tenant_id,
                Agent.status == "active"
            )
        )
    )
    return result.scalars().all()  # Returns List[Agent]

# SELECT * FROM agents WHERE name LIKE '%bot%' ORDER BY created_at DESC LIMIT 20
async def search(session, query: str):
    result = await session.execute(
        select(Agent)
        .where(Agent.name.ilike(f"%{query}%"))
        .order_by(Agent.created_at.desc())
        .limit(20)
    )
    return result.scalars().all()
```

**Java JPA equivalent:**
```java
// Spring Data JPA
List<Agent> findByTenantIdAndStatus(String tenantId, String status);
```

---

## ✏️ CRUD Operations

### Create (INSERT):
```python
async def create_agent(session, data: AgentCreate) -> Agent:
    agent = Agent(
        id=str(uuid.uuid4()),
        name=data.name,
        system_prompt=data.system_prompt,
        model=data.model,
        tenant_id=data.tenant_id,
    )
    session.add(agent)        # Stage for insert (like persist())
    await session.commit()    # Execute INSERT
    await session.refresh(agent)  # Reload from DB (get generated values)
    return agent
```

### Update:
```python
async def update_agent(session, agent_id: str, data: AgentUpdate):
    result = await session.execute(
        select(Agent).where(Agent.id == agent_id)
    )
    agent = result.scalar_one_or_none()
    if not agent:
        raise ResourceNotFoundException(code="agentNotFound", detail=f"Agent {agent_id} not found")
    
    # Update fields
    if data.name is not None:
        agent.name = data.name
    if data.system_prompt is not None:
        agent.system_prompt = data.system_prompt
    
    await session.commit()
    return agent
```

### Delete:
```python
async def delete_agent(session, agent_id: str):
    result = await session.execute(
        select(Agent).where(Agent.id == agent_id)
    )
    agent = result.scalar_one_or_none()
    if agent:
        await session.delete(agent)
        await session.commit()
```

---

## 🔄 Session Management (Like @Transactional)

```python
# Pattern used in this project
async def get_async_db_session():
    """Get a database session — injected via Depends()"""
    async with AsyncSessionLocal() as session:
        async with session.begin():  # Auto-commit/rollback (like @Transactional)
            yield session

# Usage in endpoint:
@router.post("/agents")
async def create_agent(
    request: AgentCreate,
    session = Depends(get_async_db_session)  # Session injected
):
    return await agent_service.create(session, request)
```

### Manual Transaction Control:
```python
async def complex_operation(session):
    try:
        agent = Agent(name="New Agent", ...)
        session.add(agent)
        
        tool = Tool(agent_id=agent.id, ...)
        session.add(tool)
        
        await session.commit()  # Both saved atomically
    except Exception:
        await session.rollback()  # Both rolled back
        raise
```

---

## 🗄️ This Project's Database Pattern

The project uses a **service-level session pattern**:

```python
# In agent_service.py — typical pattern
class AgentService:
    async def get_agent(self, agent_id: str, tenant_id: str, ...):
        """Get agent with tenant filtering"""
        session_factory = await get_async_db_session(DatabaseType.POSTGRES)
        async with session_factory() as session:
            result = await session.execute(
                select(Agent).where(
                    and_(
                        Agent.id == agent_id,
                        Agent.tenant_id == tenant_id,
                        Agent.provider_tenant == provider_tenant,
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

---

## 📂 Migrations (Flyway-Style)

This project uses **Flyway-style SQL migrations** (not Alembic):

```
migrations/
├── V0__initial_schema.sql          # Initial tables
├── V1__add_guardrail_type.sql      # Add column
├── V10_1__DDL_optional_visibility.sql
├── V10_25__add_skills_feature.sql  # New feature
└── ...
```

### Migration Naming Convention:
```
V{major}_{minor}__{description}.sql
```

### Applied automatically on startup:
```python
# utils/flyway_migrations.py
async def apply_migrations():
    """Auto-apply pending migrations (like Flyway auto-migrate)"""
    # Reads migrations/ folder
    # Tracks applied versions in flyway_schema_history table
    # Applies new ones in order
```

---

## 🔗 SQLAlchemy ↔ Java ORM Comparison

| SQLAlchemy | JPA/Hibernate | Purpose |
|-----------|---------------|---------|
| `Base = declarative_base()` | `@Entity` | Table mapping |
| `Column(String(100))` | `@Column(length=100)` | Column definition |
| `session.add(obj)` | `entityManager.persist(obj)` | Insert |
| `session.commit()` | `@Transactional` commit | Save |
| `select(Model).where(...)` | `CriteriaQuery` | Query building |
| `session.execute(query)` | `entityManager.createQuery()` | Execute |
| `.scalar_one_or_none()` | `.getSingleResult()` | Get one |
| `.scalars().all()` | `.getResultList()` | Get list |
| `session.rollback()` | Transaction rollback | Undo |
| `create_async_engine()` | `DataSource` | Connection pool |

---

## ▶️ Next: [07-langchain-langgraph.md](./07-langchain-langgraph.md) — The AI engine

