# 03 — Project Architecture

> Complete architecture overview. How all the pieces fit together.

---

## 🏗️ High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AI Studio Frontend                            │
└─────────────────────────────┬───────────────────────────────────────┘
                              │ REST API (HTTP/SSE)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   THIS SERVICE (FastAPI)                             │
│  ┌──────────┐  ┌───────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │Endpoints │→ │ Services  │→ │ LangGraph    │→ │ LLM (Bedrock)│  │
│  │(Routers) │  │(Business) │  │ (AI Engine)  │  │              │  │
│  └──────────┘  └───────────┘  └──────────────┘  └──────────────┘  │
│       │              │              │                    │           │
│       │              ▼              ▼                    │           │
│       │        ┌──────────┐  ┌──────────────┐          │           │
│       │        │PostgreSQL│  │ MCP Servers  │          │           │
│       │        │ (Config) │  │ (Tools)      │          │           │
│       │        └──────────┘  └──────────────┘          │           │
│       │              │                                   │           │
│       │              ▼                                   │           │
│       │        ┌──────────┐                             │           │
│       │        │  Redis   │ (Caching, Traces)           │           │
│       │        └──────────┘                             │           │
└───────┼─────────────────────────────────────────────────┼───────────┘
        │                                                  │
        ▼                                                  ▼
┌──────────────┐                                  ┌──────────────┐
│ Agent Registry│                                  │ AWS Bedrock  │
│ Service      │                                  │ (LLMs)       │
└──────────────┘                                  └──────────────┘
```

---

## 📁 Complete Folder Structure

```
agentic-ai-service-backend-develop/
│
├── app/                          # Main application code
│   ├── __init__.py
│   └── api/
│       ├── main.py               # 🚀 APP ENTRY POINT (FastAPI app creation)
│       ├── config.py             # ⚙️ All configuration (reads .env)
│       ├── db.py                 # 🗄️ Database connection setup
│       ├── constants.py          # Fixed values
│       │
│       ├── endpoints/            # 🌐 REST API routes (= Spring Controllers)
│       │   ├── agents.py         #   /agents CRUD + execute
│       │   ├── tools.py          #   /tools management
│       │   ├── guardrails.py     #   /guardrails config
│       │   ├── executions.py     #   /executions history
│       │   ├── custom_workflows.py  # /custom-workflows
│       │   ├── dynamic_workflows.py # /dynamic-workflows
│       │   ├── skills.py         #   /skills management
│       │   └── ...
│       │
│       ├── services/             # 💼 Business logic (= Spring Services)
│       │   ├── agent_service.py  #   ⭐ MAIN: Agent CRUD + execution
│       │   ├── agent/            #   Agent sub-modules:
│       │   │   ├── agent_creation.py   # Build LangGraph agent
│       │   │   ├── agent_execution.py  # Run agent
│       │   │   └── agent_models.py     # Internal models
│       │   ├── custom_workflow/   #   User-defined DAG workflows
│       │   ├── dynamic_workflow/  #   Supervisor-based workflows
│       │   ├── memory_services/   #   Agent memory management
│       │   ├── tool_service.py    #   MCP tool management
│       │   ├── execution_service.py    # Execution tracking
│       │   ├── guardrail_service.py    # Input/output guardrails
│       │   └── ...
│       │
│       ├── models/               # 📋 Pydantic schemas (= Java DTOs)
│       │   ├── agent.py          #   AgentCreate, AgentResponse, etc.
│       │   ├── tool.py           #   Tool schemas
│       │   ├── executions.py     #   Execution schemas
│       │   └── ...
│       │
│       ├── utils/                # 🔧 Utilities & helpers
│       │   ├── llm_providers.py  #   LLM connection factory
│       │   ├── mcp_tool_cache.py #   MCP tool caching
│       │   ├── checkpointer_utils.py  # LangGraph state persistence
│       │   ├── flyway_migrations.py   # DB migration runner
│       │   └── ...
│       │
│       ├── exceptions/           # ❌ Custom exceptions
│       │   └── custom_exceptions.py
│       │
│       ├── validators/           # ✅ Input validation
│       │   └── field_validators.py
│       │
│       ├── mcp_servers/          # 🧪 Sample MCP tool servers (for testing)
│       │   ├── math_server.py
│       │   ├── english_server.py
│       │   └── ...
│       │
│       └── catalogs/             # UI layout catalogs
│
├── logging_lib/                  # 📝 Custom logging library
│   ├── logger.py                 # Main logger
│   ├── logging_config.yaml       # Log configuration
│   └── mdc_keys.py              # Mapped Diagnostic Context
│
├── security_lib/                 # 🔒 Authentication/Authorization
│   ├── auth.py                   # Token verification
│   ├── token_verifier.py        # JWT validation
│   └── constants.py             # Security constants
│
├── migrations/                   # 🗄️ Database migrations (Flyway-style)
│   ├── V0__initial_schema.sql
│   ├── V1__add_guardrail_type.sql
│   └── V10_*__*.sql             # Versioned migrations
│
├── tests/                        # 🧪 Test suite
│   ├── conftest.py              # Test fixtures & mocks
│   ├── unit/                    # Unit tests (no real DB)
│   └── integration/             # Integration tests
│
├── scripts/                      # 🛠️ Dev scripts
│   └── docker-compose.db.yml   # Local DB containers
│
├── docs/                         # 📚 Documentation
│   ├── execution_flow_guide.md
│   ├── agent-skills.md
│   └── ...
│
├── pyproject.toml               # 📦 Dependencies (= pom.xml)
├── poetry.lock                  # 🔒 Locked versions
├── Dockerfile                   # 🐳 Container build
├── justfile                     # ⚡ Task runner
├── Makefile                     # ⚡ Alternative task runner
├── conftest.py                  # 🧪 Root test config
└── .env                         # ⚙️ Local environment vars
```

---

## 🔄 Three Execution Paths

This project supports THREE ways to run AI agents:

### Path 1: Standalone Agent Execution
```
POST /agents/{id}/execute
    → agent_service.execute_agent()
        → AgentCreation.build_agent_app()  (create LangGraph graph)
        → AgentExecution.execute()          (run the graph)
            → LangGraph → LLM (Bedrock/OpenAI)
                       → MCP Tools
```

### Path 2: Custom Workflows (User-Defined DAGs)
```
POST /custom-workflows/{id}/execute
    → custom_workflows_service.execute()
        → Build workflow graph (agent nodes connected by edges)
        → For each node: agent_service.run_agent_with_context()
            → Same LangGraph execution as standalone
```

### Path 3: Dynamic Workflows (Supervisor-Based)
```
POST /dynamic-workflows/{id}/execute
    → dynamic_workflow_service.execute()
        → Supervisor agent decides which sub-agents to call
        → Routes to sub-agents dynamically
            → Same LangGraph execution as standalone
```

---

## 🧩 Layer Responsibilities

### Endpoints Layer (`endpoints/`)
- **Purpose:** HTTP request/response handling
- **Java analog:** `@RestController` classes
- **Does:** Parse request → validate → call service → return response
- **Does NOT:** Business logic, DB access directly

### Services Layer (`services/`)
- **Purpose:** All business logic
- **Java analog:** `@Service` classes
- **Does:** Orchestration, validation, DB queries, agent execution
- **Pattern:** Singleton instances (module-level) — e.g., `agent_service = AgentService()`

### Models Layer (`models/`)
- **Purpose:** Data shapes (request/response schemas)
- **Java analog:** DTOs + validation annotations
- **Uses:** Pydantic `BaseModel` for auto-validation

### Utils Layer (`utils/`)
- **Purpose:** Cross-cutting helpers
- **Contains:** LLM provider setup, caching, migrations, middleware

---

## 🔐 Multi-Tenancy

Every API request includes authentication context:

```python
# Extracted from JWT token on every request:
tenant_id: str          # "tenant-123"
provider_tenant: str    # "provider-456" 
application: str        # "app-789"
namespace: str          # Optional grouping
```

All database queries filter by these fields:
```python
query = select(Agent).where(
    Agent.tenant_id == tenant_id,
    Agent.provider_tenant == provider_tenant,
    Agent.application == application
)
```

---

## 🔄 Request Lifecycle

```
1. HTTP Request arrives at FastAPI
2. Middleware runs (CORS, logging, security)
3. Security: JWT token validated → tenant info extracted
4. Router: Request matched to endpoint function
5. Pydantic: Request body validated against model
6. Depends(): Dependencies injected (DB session, auth)
7. Service: Business logic executed
8. Response: Pydantic model serialized to JSON
9. Middleware: Response headers, logging
10. HTTP Response sent
```

---

## 🗄️ Database Architecture

```
PostgreSQL (Primary)
├── agents              # Agent configurations
├── tools               # MCP tool definitions
├── guardrails          # Input/output guardrails
├── executions          # Execution history
├── execution_nodes     # Step-by-step execution details
├── custom_workflows    # Workflow definitions
├── output_formats      # Response format templates
├── skills              # Uploaded skill files
├── key_pairs           # Encryption keys
└── langgraph_*         # LangGraph checkpoint tables
```

Migrations in `migrations/` folder, auto-applied on startup.

---

## ▶️ Next: [04-fastapi-basics.md](./04-fastapi-basics.md) — Learn the FastAPI framework

