# 🏗️ Complete Project Flow — Agentic AI Service Backend

## Table of Contents
1. [Project Overview](#project-overview)
2. [Tech Stack](#tech-stack)
3. [Architecture Diagram](#architecture-diagram)
4. [Project Structure Deep Dive](#project-structure-deep-dive)
5. [Application Startup Flow](#application-startup-flow)
6. [Request Lifecycle](#request-lifecycle)
7. [Core Execution Paths](#core-execution-paths)
8. [Database Layer](#database-layer)
9. [Security & Authentication](#security--authentication)
10. [MCP Tool Integration](#mcp-tool-integration)
11. [Guardrails System](#guardrails-system)
12. [Dynamic Workflows (Supervisor)](#dynamic-workflows-supervisor)
13. [Custom Workflows](#custom-workflows)
14. [Skills System](#skills-system)
15. [Middleware Pipeline](#middleware-pipeline)
16. [Configuration Management](#configuration-management)
17. [Observability & Telemetry](#observability--telemetry)
18. [Key Design Patterns](#key-design-patterns)
19. [API Endpoints Summary](#api-endpoints-summary)
20. [Data Models](#data-models)

---

## Project Overview

This is a **multi-tenant AI Agent Orchestration Platform** built with FastAPI. It allows enterprises to:
- Create, configure, and execute AI agents powered by AWS Bedrock LLMs
- Connect agents to external tools via MCP (Model Context Protocol)
- Build workflows (sequential, parallel, conditional) connecting multiple agents
- Run dynamic workflows where a supervisor agent discovers and routes to sub-agents at runtime
- Apply guardrails (content filtering, PII detection) to agent inputs/outputs
- Support Human-in-the-Loop (HITL) approval gates
- Manage execution state, memory, and conversation checkpoints

**In simple words:** It's a backend service that lets you create AI agents, give them tools, connect them in workflows, and execute them safely with guardrails — all in a multi-tenant SaaS environment.

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Web Framework** | FastAPI 0.136.0 | Async REST API, auto-docs, dependency injection |
| **AI/LLM** | LangChain 1.2.0 + LangGraph 1.0.10 | Agent orchestration, state graphs, tool calling |
| **LLM Provider** | AWS Bedrock (Claude, Titan) | Language model inference |
| **Database** | PostgreSQL (primary), MySQL (legacy) | Application data, workflow state |
| **Checkpointing** | LangGraph Checkpoint Postgres | Conversation state persistence |
| **Cache** | Redis 7.3.0 | Model metadata cache, execution state |
| **Object Storage** | AWS S3 | Skills content, execution traces |
| **Message Queue** | AWS SQS | Agent registry events, deprecated model notifications |
| **Tool Protocol** | MCP (Model Context Protocol) | External tool connectivity |
| **Agent Protocol** | A2A SDK 0.3.22 | Agent-to-Agent communication |
| **Auth** | JWT + JWKS | Multi-tenant token verification |
| **Observability** | OpenTelemetry | Distributed tracing |
| **ORM** | SQLAlchemy 2.0.41 (async) | Database access |
| **Validation** | Pydantic 2.11.7 | Request/response schemas |
| **Server** | Uvicorn 0.35.0 | ASGI server (4 workers) |
| **Package Manager** | Poetry | Dependency management |
| **Migrations** | Flyway-style (custom) | SQL schema migrations |

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT (AI Studio UI / API Consumer)                     │
└──────────────────────────────────────┬──────────────────────────────────────────────┘
                                       │ HTTPS + JWT Bearer Token
                                       ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    FASTAPI APP                                       │
│  ┌──────────────┐  ┌───────────────┐  ┌──────────────┐  ┌───────────────────────┐  │
│  │  Middleware   │  │  Security     │  │  Exception   │  │   CORS + Headers      │  │
│  │  Pipeline     │  │  (JWT verify) │  │  Handlers    │  │   + XSS Sanitize      │  │
│  └──────┬───────┘  └───────┬───────┘  └──────┬───────┘  └───────────────────────┘  │
│         ▼                   ▼                  ▼                                      │
│  ┌──────────────────────────────────────────────────────────────────────────────┐    │
│  │                          API ENDPOINTS (Routers)                              │    │
│  │  /agents  /tools  /executions  /custom-workflows  /dynamic-workflows         │    │
│  │  /guardrails  /output-formats  /skills  /keys  /long-term-memories           │    │
│  └──────────────────────────────────────┬───────────────────────────────────────┘    │
│                                          ▼                                            │
│  ┌──────────────────────────────────────────────────────────────────────────────┐    │
│  │                           SERVICES LAYER                                      │    │
│  │  ┌─────────────┐ ┌──────────────────┐ ┌─────────────────────────────────┐   │    │
│  │  │AgentService │ │CustomWorkflow    │ │DynamicWorkflowService           │   │    │
│  │  │             │ │Service           │ │(Supervisor Orchestration)        │   │    │
│  │  └──────┬──────┘ └────────┬─────────┘ └────────────────┬────────────────┘   │    │
│  │         │                  │                             │                    │    │
│  │         ▼                  ▼                             ▼                    │    │
│  │  ┌─────────────────────────────────────────────────────────────────────┐     │    │
│  │  │                    AgentRunner (Execution Orchestrator)              │     │    │
│  │  │   ┌───────────┐  ┌──────────────┐  ┌─────────────────────────┐    │     │    │
│  │  │   │ProgressHook│  │GuardrailHook │  │HumanApprovalHook (HITL) │    │     │    │
│  │  │   └───────────┘  └──────────────┘  └─────────────────────────┘    │     │    │
│  │  └─────────────────────────────┬───────────────────────────────────────┘     │    │
│  │                                 ▼                                             │    │
│  │  ┌─────────────────────────────────────────────────────────────────────┐     │    │
│  │  │              AgentCreation → LangGraph StateGraph                    │     │    │
│  │  │   ┌─────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────────┐   │     │    │
│  │  │   │ LLM Node│  │Tool Nodes│  │ KB Search│  │ Skills Loading  │   │     │    │
│  │  │   └────┬────┘  └────┬─────┘  └────┬─────┘  └─────────────────┘   │     │    │
│  │  │        ▼             ▼              ▼                               │     │    │
│  │  └─────────────────────────────────────────────────────────────────────┘     │    │
│  └──────────────────────────────────────────────────────────────────────────────┘    │
└──────────────────────────────────────┬──────────────────────────────────────────────┘
                                       │
              ┌────────────────────────┼────────────────────────────┐
              ▼                        ▼                            ▼
┌──────────────────┐    ┌──────────────────────┐    ┌──────────────────────────┐
│  AWS Bedrock     │    │  PostgreSQL + Redis   │    │  MCP Servers (Tools)     │
│  (LLM Inference) │    │  (State + Cache)      │    │  (External Capabilities) │
└──────────────────┘    └──────────────────────┘    └──────────────────────────┘
```

---

## Project Structure Deep Dive

```
app/api/
├── main.py                 # FastAPI app initialization, lifespan, middleware, router registration
├── config.py               # All configuration from env vars (Pydantic BaseModel)
├── db.py                   # Database connection factory (Postgres preferred, MySQL fallback)
├── constants.py            # App-wide constants (limits, sizes, etc.)
├── init_db.py              # Database initialization helpers
│
├── endpoints/              # FastAPI routers (HTTP layer only)
│   ├── agents.py           # /agents - CRUD + execute
│   ├── tools.py            # /tools - MCP server management
│   ├── custom_workflows.py # /custom-workflows - DAG workflows
│   ├── dynamic_workflows.py# /dynamic-workflows - Supervisor workflows
│   ├── executions.py       # /executions - Status polling, resume
│   ├── guardrails.py       # /guardrails - Content filtering rules
│   ├── skills.py           # /skills - Reusable instruction packages
│   ├── output_formats.py   # /output-formats - Response structure templates
│   ├── keys.py             # /keys - RSA key pair management
│   └── __init__.py         # Exports all routers
│
├── services/               # Business logic layer
│   ├── agent_service.py    # Main agent service (4000+ lines) - CRUD + execution
│   ├── agent/              # Refactored agent internals
│   │   ├── agent_creation.py   # Builds LangGraph apps (2700+ lines)
│   │   ├── agent_execution.py  # Executes agents (sync/async/SSE)
│   │   └── agent_models.py     # GraphState, AgentApp dataclasses
│   ├── custom_workflow/    # Workflow graph building
│   │   ├── custom_workflow_creation.py  # DAG construction
│   │   ├── custom_workflow_execution.py # Graph execution
│   │   └── custom_workflow_models.py    # Workflow state models
│   ├── dynamic_workflow/   # Supervisor-based routing
│   │   ├── dynamic_workflow_execution.py # Supervisor loop
│   │   └── dynamic_workflow_synthesis_helpers.py # Final answer generation
│   ├── execution_service.py    # Execution state persistence
│   ├── tool_service.py         # MCP server management (3000+ lines)
│   ├── guardrail_service.py    # AWS Bedrock guardrails
│   ├── skill_service.py        # Skills CRUD + S3 storage
│   ├── memory_service.py       # Long-term memory management
│   ├── governance_service.py   # Approval workflows
│   └── ...
│
├── models/                 # Pydantic schemas (request/response)
│   ├── agent.py            # AgentCreate, AgentResponse, AgentStep, etc.
│   ├── custom_workflows.py # Workflow nodes, edges, execution models
│   ├── dynamic_workflows.py# Supervisor workflow models
│   ├── tool.py             # MCP server & tool models
│   ├── guardrails.py       # Guardrail configuration models
│   ├── executions.py       # Execution state/response models
│   └── ...
│
├── utils/                  # Shared utilities
│   ├── llm_providers.py    # AWS Bedrock client factory
│   ├── checkpointer_utils.py # LangGraph checkpoint pool
│   ├── mcp_tool_cache.py   # MCP session caching per request
│   ├── mcp_validator.py    # MCP server connectivity checks
│   ├── flyway_migrations.py # SQL migration runner
│   ├── execution_handler.py # Shared execution response building
│   └── ...
│
├── exceptions/             # Custom exception hierarchy
│   └── custom_exceptions.py # AgenticAIException base + 20+ specific exceptions
│
└── validators/             # Field validation utilities
    └── field_validators.py # Name, UUID, description validators

migrations/                 # Flyway SQL migrations (V0 through V10_22+)
security_lib/               # JWT token verification library
logging_lib/                # Structured JSON logging (MDC context)
tests/                      # Unit + Integration tests
```

---

## Application Startup Flow

```
1. OTel Auto-Instrumentation (before any import)
   └─ opentelemetry.instrumentation.auto_instrumentation.initialize()

2. setup_logging() → CustomUvicornFormatter (structured JSON)

3. Flyway Migrations (init_migrations())
   └─ Reads migrations/ directory → applies SQL in order

4. Agent Steps Migration (init_agent_steps_migration())
   └─ Migrates legacy MCP format to new array format

5. Import all routers from endpoints/__init__.py

6. FastAPI app = FastAPI(lifespan=lifespan, dependencies=[cleanup_mcp_sessions])

7. Lifespan startup:
   ├─ initialize_checkpointer() → AsyncPostgresSaver pool
   ├─ initialize_async_db_pool() → SQLAlchemy async engine
   ├─ Shared httpx client → MCP connections (500 max, 100 keepalive)
   ├─ deprecated_model_consumer.start() → SQS polling
   └─ agent_registry_event_consumer.start() → SQS polling

8. Register middlewares:
   ├─ SecurityHeadersMiddleware (HSTS, CSP, X-Frame)
   ├─ XSSInputSanitizationMiddleware (header sanitization)
   ├─ CORSMiddleware (Siemens cloud domain)
   ├─ validate_request middleware (error handling)
   └─ log_requests middleware (timing + logref)

9. Include all routers → app.include_router(...)

10. Uvicorn starts on 0.0.0.0:8000 with 4 workers
```

---

## Request Lifecycle

```
1. HTTP Request arrives at Uvicorn worker

2. ASGI Middleware Pipeline (order matters):
   ├─ log_requests → Sets logref UUID, measures duration
   ├─ validate_request → Catches unhandled errors
   ├─ CORSMiddleware → Adds CORS headers
   ├─ XSSInputSanitizationMiddleware → Cleans dangerous headers
   └─ SecurityHeadersMiddleware → Adds security response headers

3. FastAPI Dependency Injection:
   ├─ verify_token_with_scope() → JWT validation → TokenVerificationResult
   ├─ cleanup_mcp_sessions → Yields (cleanup happens AFTER response)
   └─ Route-specific dependencies

4. Endpoint Handler (e.g., agents.py → create_agent)
   ├─ Extracts tenant_id, provider_tenant, application from token
   ├─ Validates request body (Pydantic model)
   └─ Calls service layer

5. Service Layer (e.g., agent_service.create_agent)
   ├─ Business validation
   ├─ Database operations (async SQLAlchemy)
   ├─ External calls (Bedrock, MCP, Redis)
   └─ Returns response model

6. Response serialization (Pydantic → JSON)

7. MCP Session Cleanup (FastAPI dependency after-yield)
   └─ Closes all MCP sessions opened during request

8. Response sent to client
```

---

## Core Execution Paths

### Path 1: Standalone Agent Execution

```
POST /agents/{id}/execute
    │
    ▼
agents.py endpoint
    │ verify token, extract tenant info
    ▼
agent_service.execute_agent()
    │ Load agent from DB
    │ Validate model exists (Redis cache → LLM service)
    │ Check governance (approval required?)
    ▼
AgentExecution.execute()
    │ Generate execution_id, session_id
    │ Get persistent checkpointer
    ▼
AgentCreation.build_agent_app()
    │ ├─ Resolve MCP tools (discover from servers)
    │ ├─ Load knowledge bank tools
    │ ├─ Load skill tools (LoadSkillTool, LoadSkillResourceTool)
    │ ├─ Build system prompt (+ skills section + output format)
    │ ├─ Create LLM instance (Bedrock/OpenAI)
    │ ├─ Add middlewares (retry, summarization, call limit)
    │ ├─ Build LangGraph StateGraph
    │ └─ Compile with checkpointer
    ▼
Execute LangGraph (ainvoke/astream)
    │ Messages flow through graph nodes
    │ LLM decides tool calls → tools execute → results return
    │ Guardrails check input/output
    ▼
Persist execution state
    │ execution_service._store_execution_state()
    ▼
Return response (output, tools_used, token_usage, metrics)
```

### Path 2: Custom Workflow Execution

```
POST /custom-workflows/{id}/execute
    │
    ▼
CustomWorkflowService.execute_workflow()
    │ Load workflow + nodes + edges from DB
    │ Build LangGraph workflow graph:
    │   ├─ Agent nodes → each calls agent_service.run_agent_with_context()
    │   ├─ Conditional edges (LLM-based routing)
    │   └─ Parallel branches (fan-out/fan-in)
    ▼
LangGraph executes the DAG
    │ Each agent node:
    │   ├─ Creates AgentExecutionRequest + AgentExecutionContext
    │   ├─ Calls AgentRunner.run()
    │   └─ Merges output into workflow state
    ▼
Final node output → workflow response
```

### Path 3: Dynamic Workflow (Supervisor)

```
POST /dynamic-workflows/{id}/execute
    │
    ▼
DynamicWorkflowExecution.execute()
    │ Load supervisor agent + config
    │ Build supervisor LangGraph with:
    │   ├─ Supervisor node (LLM with routing tools)
    │   ├─ discover_entities tool (searches agent registry)
    │   ├─ HIL gate (human approval before execution)
    │   ├─ Entity executor (runs selected agent/workflow)
    │   └─ Summarizer (creates output summary)
    ▼
Supervisor Loop:
    │ 1. Supervisor analyzes request
    │ 2. Decides: discover_entities / select_entity / complete
    │ 3. If select_entity → HIL gate → Execute entity → Summarize
    │ 4. Loop back with updated history
    │ 5. Until "complete" or max iterations reached
    ▼
Final Synthesizer → Combines all outputs → Final answer
```

---

## Database Layer

### Connection Strategy
- **PostgreSQL** is preferred (set via `DB_TYPE=postgres` env var)
- **MySQL** is legacy fallback
- Async engine with connection pooling (`pool_pre_ping=True`, `pool_recycle=300`)
- Separate Postgres connection for LangGraph checkpoints

### Key Tables

| Table | Purpose |
|-------|---------|
| `agents` | Agent configurations (model, prompt, tools, steps) |
| `tools` | MCP server registrations |
| `custom_workflows` | Workflow definitions |
| `custom_workflow_edges` | DAG connections between nodes |
| `dynamic_workflows` | Supervisor workflow configs |
| `executions` | Execution state and history |
| `guardrails` | Content filtering rules |
| `output_formats` | Response structure templates |
| `skills` / `skill_versions` | Reusable instruction packages |
| `key_pairs` | RSA key pairs for auth |
| `a2a_agents` | Agent-to-Agent registrations |

### Migration System
- Located in `migrations/` directory
- Naming: `V{major}_{minor}__description.sql`
- Applied automatically on startup via `flyway_migrations.py`
- Tracks applied migrations in a metadata table

---

## Security & Authentication

### JWT Token Flow
```
Client → JWT Bearer Token → security_lib.verify_token_with_scope()
    │
    ├─ Fetches JWKS from identity provider (cached)
    ├─ Verifies signature + expiry
    ├─ Extracts: tenant_id, provider_tenant, application, scopes
    └─ Returns TokenVerificationResult
```

### Multi-tenancy
- Every request carries: `tenant_id` + `provider_tenant` + `application`
- ALL database queries filter by these three fields
- Optional `namespace` for sub-tenant isolation
- `TenancyType`: TENANT_SPECIFIC (default) or CROSS_TENANT (shared resources)

### Security Middlewares
1. **XSS Input Sanitization** — Strips HTML/script from client headers
2. **Security Headers** — HSTS, CSP, X-Frame-Options, nosniff
3. **CORS** — Restricted to Siemens cloud domain

---

## MCP Tool Integration

### What is MCP?
Model Context Protocol — a standard for connecting AI agents to external tools/services.

### Flow
```
1. Admin registers MCP server: POST /tools
   └─ Stores server URL, auth config, available tools

2. Agent creation: tools.mcp = [{mcpServerId, toolNames}]
   └─ Links agent to specific tools from specific servers

3. At execution time:
   ├─ validate_mcp_reachability() → Check server is alive
   ├─ MultiServerMCPClient connects to server(s)
   ├─ load_mcp_tools() → Discovers available tools
   ├─ Tools are added to LangGraph agent
   └─ LLM calls tools → MCP server executes → results return

4. After request:
   └─ cleanup_mcp_sessions → Closes all connections
```

### Tool Cache
- Per-request MCP session cache (`mcp_tool_cache`)
- Shared httpx client with connection pooling (500 connections)
- `NonClosingHttpxClient` wrapper prevents premature TLS close

---

## Guardrails System

### Architecture
- Powered by **AWS Bedrock Guardrails**
- Applied at two points: INPUT (before LLM) and OUTPUT (after LLM)
- Configurable per agent/workflow

### Types of Guardrails
1. **Topic Policy** — Block certain topics
2. **Content Policy** — Filter harmful content
3. **Word Policy** — Block specific words/phrases
4. **Sensitive Information** — PII detection + anonymization
5. **Contextual Grounding** — Verify factual accuracy

### Enforcement Modes
- `guardrail_enforcement_enabled=False` (default): Log violations only
- `guardrail_enforcement_enabled=True`: Block request on violation

---

## Dynamic Workflows (Supervisor)

### Supervisor System Prompt Architecture

The supervisor gets a structured system prompt with 6 sections:
1. **USER ROUTING POLICY** — Custom strategy per supervisor
2. **DEFAULT ROUTING POLICY** — Fallback if no custom policy
3. **ANALYSIS INSTRUCTIONS** — How to assess requests
4. **CONSTRAINTS** — Non-negotiable rules (single-task discovery, failure handling)
5. **CAPABILITIES** — Available tools and state descriptions
6. **OUTPUT FORMAT** — Required response structure (SupervisorDecision schema)

### Supervisor Decision Actions
- `select_entity` — Choose an agent/workflow to execute
- `discover_entities` — Search registry for capabilities
- `complete` — Task fully done, provide final answer
- `partial_complete` — Task partially done
- `override_rejected` — Override human rejection
- `none` — No action needed

---

## Custom Workflows

### Workflow Types
1. **Sequential** — Agents execute one after another
2. **Parallel** — Multiple agents execute simultaneously
3. **Conditional** — LLM-based routing (if/else paths)
4. **DAG** — Directed Acyclic Graph (any combination)

### Node Types
- **Agent Node** — Executes a specific agent
- **Condition Node** — Routes based on LLM evaluation
- **Start/End** — Entry and exit points

### Edge Types
- **Direct Edge** — Simple A → B connection
- **Conditional Edge** — A → B or A → C based on condition

---

## Skills System

### Overview
Reusable instruction packages (markdown with YAML frontmatter) stored in S3.

### Loading Tiers
| Tier | Name | When | How |
|------|------|------|-----|
| L1 | Discovery | Agent build time | Names + descriptions in system prompt |
| L2 | Activation | Runtime tool call | `load_skill(name)` → returns SKILL.md body |
| L3 | Resources | Runtime tool call | `load_skill_resource(name, path)` → returns file |

---

## Middleware Pipeline

### LangChain/LangGraph Middlewares (per agent)
1. **ModelRetryMiddleware** — Retries failed LLM calls (exponential backoff)
2. **SummarizationMiddleware** — Summarizes long conversations
3. **ModelCallLimitMiddleware** — Limits total LLM calls per execution
4. **PerToolRetryMiddleware** — Retries failed tool calls

---

## Configuration Management

All configuration loaded from environment variables via `config.py`:

```python
config = Config.from_env()  # Single source of truth

# Access: config.database.host, config.aws.region, config.redis.host, etc.
```

### Key Environment Variables
| Variable | Purpose |
|----------|---------|
| `DB_TYPE` | postgres or mysql |
| `POSTGRES_HOST/PORT/USERNAME/PASSWORD/DATABASE` | Primary DB |
| `AWS_REGION` | AWS region for Bedrock |
| `REDIS_HOST/PORT` | Cache layer |
| `HOST_REGION` | Deployment region (eu1/us1/ap1) |
| `RECURSION_LIMIT` | Max LangGraph iterations |
| `MCP_SERVER_TIMEOUT` | Tool connection timeout |
| `GUARDRAIL_ENFORCEMENT_ENABLED` | Block or log-only mode |

---

## Observability & Telemetry

- **OpenTelemetry** auto-instrumentation (traces all HTTP, DB, Redis calls)
- **Structured JSON logging** via `logging_lib` (MDC context, correlation IDs)
- **Execution traces** stored in S3 (tool calls, LLM responses, timings)
- **Redis Span Exporter** for real-time trace visibility

---

## Key Design Patterns

### 1. Service Singleton Pattern
```python
# At module level in service files:
agent_service = AgentService()
tool_service = ToolService()
# Used directly in endpoints
```

### 2. Hook Pattern (Pluggable Behavior)
```python
class ExecutionHook:
    async def before_run(self, request, context): ...
    async def after_run(self, request, context, state, response): ...
    async def on_error(self, request, context, error): ...

# Register hooks:
runner = AgentRunner(hooks=[ProgressHook(), GuardrailHook()])
```

### 3. Exception Hierarchy
```python
AgenticAIException (base, 500)
├── ResourceNotFoundException (404)
├── BadRequestException (400)
├── ConflictException (409)
├── ForbiddenException (403)
├── UnauthorizedException (401)
├── AgentCreationFailedException (500)
├── AgentExecutionFailedException (500)
├── ServerConnectionException (502)
└── ... (20+ specific exceptions)
```

### 4. Multi-tenancy Filter
```python
# Every query includes:
query = select(AgentDB).where(
    and_(
        AgentDB.tenant_id == tenant_id,
        AgentDB.provider_tenant == provider_tenant,
        AgentDB.application == application,
    )
)
```

### 5. Async-First
- All service methods are `async`
- SQLAlchemy async sessions
- httpx async client for MCP
- asyncio.create_task for background execution

---

## API Endpoints Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/agents` | List agents (paginated) |
| `POST` | `/agents` | Create agent |
| `GET` | `/agents/{id}` | Get agent details |
| `PUT` | `/agents/{id}` | Update agent |
| `DELETE` | `/agents/{id}` | Delete agent |
| `POST` | `/agents/{id}/execute` | Execute agent |
| `POST` | `/agents/{id}/resume` | Resume HITL-paused agent |
| `GET` | `/tools` | List MCP servers |
| `POST` | `/tools` | Register MCP server |
| `GET` | `/executions` | List executions |
| `GET` | `/executions/{id}` | Get execution status |
| `POST` | `/custom-workflows` | Create workflow |
| `POST` | `/custom-workflows/{id}/execute` | Execute workflow |
| `POST` | `/dynamic-workflows` | Create supervisor workflow |
| `POST` | `/dynamic-workflows/{id}/execute` | Execute supervisor workflow |
| `POST` | `/guardrails` | Create guardrail |
| `POST` | `/skills` | Upload skill |
| `GET` | `/health` | Health check |

---

## Data Models

### AgentCreate (Request)
```python
{
    "name": "my-agent",
    "version": "1.0",
    "description": "Agent that does X",
    "model": "anthropic.claude-sonnet-4-5-20250929-v1:0",
    "systemPrompt": "You are a helpful assistant...",
    "tools": {
        "mcp": [
            {"mcpServerId": "server-1", "toolNames": ["tool_a", "tool_b"]}
        ]
    },
    "temperature": 0.7,
    "maxTokens": 800,
    "guardrails": ["guardrail-id-1"],
    "steps": [...],  // Optional multi-step
    "skills": ["skill-id-1"]
}
```

### Execution Response
```python
{
    "executionId": "uuid",
    "sessionId": "uuid",
    "status": "COMPLETED",
    "output": "Agent's response text",
    "toolsUsed": [{"name": "tool_a", "mcpServerId": "server-1"}],
    "metrics": {
        "inputTokens": 150,
        "outputTokens": 320,
        "totalTokens": 470,
        "durationMs": 2340
    }
}
```

---

## Summary: End-to-End Agent Execution

```
1. User sends POST /agents/{id}/execute with instruction
2. JWT verified → tenant extracted
3. Agent loaded from PostgreSQL
4. Model validated via Redis cache (or LLM service)
5. MCP tools discovered from configured servers
6. LangGraph StateGraph built (LLM + tools + guardrails)
7. Input guardrails checked
8. LangGraph executes (LLM → tool calls → LLM → ... until done)
9. Output guardrails checked
10. Execution state persisted to PostgreSQL
11. Response returned with output + metrics + tool usage
12. MCP sessions cleaned up
```

**This is a production-grade, enterprise AI orchestration platform.** Understanding this flow is the foundation for contributing as a developer.

