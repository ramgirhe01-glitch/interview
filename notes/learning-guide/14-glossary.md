# 14 — Glossary & Terminology

> Quick reference for terms used in this project.

---

## 🤖 AI / Agent Terms

| Term | Definition |
|------|-----------|
| **LLM** | Large Language Model — the AI brain (Claude, GPT-4) |
| **Agent** | An LLM + tools + system prompt + configuration |
| **System Prompt** | Instructions that define agent behavior (invisible to user) |
| **Tool** | A function/API the agent can call (calculator, search, etc.) |
| **Guardrail** | Safety rule that checks input/output (blocks harmful content) |
| **Execution** | One run of an agent (user sends instruction → agent responds) |
| **Session** | A conversation thread (multiple executions with memory) |
| **Token** | Unit of text for LLMs (~4 chars). Determines cost & limits |
| **Temperature** | Randomness control (0 = deterministic, 2 = creative) |
| **Hallucination** | When AI generates incorrect/fabricated information |
| **RAG** | Retrieval-Augmented Generation — giving AI access to documents |
| **Embedding** | Converting text to numbers for similarity search |

---

## 🔄 Workflow Terms

| Term | Definition |
|------|-----------|
| **Standalone Agent** | Single agent execution (no workflow) |
| **Custom Workflow** | User-defined graph of connected agents (DAG) |
| **Dynamic Workflow** | Supervisor agent that routes to sub-agents automatically |
| **Supervisor** | A special agent that decides which other agents to call |
| **DAG** | Directed Acyclic Graph — workflow with no loops |
| **Node** | A step in a workflow (usually one agent) |
| **Edge** | Connection between workflow nodes |
| **Conditional Edge** | Connection that depends on a condition |

---

## 🛠️ Technology Terms

| Term | Definition |
|------|-----------|
| **MCP** | Model Context Protocol — standard for AI-tool communication |
| **LangChain** | Python framework for building AI applications |
| **LangGraph** | Library for building agent state machines |
| **Checkpoint** | Saved agent state (for resuming/memory) |
| **SSE** | Server-Sent Events — streaming response protocol |
| **HITL** | Human-in-the-Loop — agent pauses for human approval |
| **A2A** | Agent-to-Agent — protocol for agents calling other agents |
| **Bedrock** | AWS service that provides LLM access |
| **OpenTelemetry** | Observability framework (tracing, metrics) |

---

## 🏗️ Architecture Terms

| Term | Definition |
|------|-----------|
| **Multi-tenant** | One service serves multiple isolated customers |
| **Tenant ID** | Unique identifier for a customer/organization |
| **Provider Tenant** | The platform provider's tenant identifier |
| **Application** | Logical grouping within a tenant |
| **Namespace** | Optional sub-grouping for resources |
| **Endpoint** | An API route (URL + method) |
| **Router** | Group of related endpoints (like Spring Controller) |
| **Service** | Business logic layer (all operations) |
| **Middleware** | Code that runs on every request (logging, CORS, auth) |
| **Dependency Injection** | Providing resources to functions automatically |

---

## 🐍 Python Terms

| Term | Definition |
|------|-----------|
| **virtualenv / .venv** | Isolated Python environment (like Maven local repo) |
| **Poetry** | Dependency manager (like Maven/Gradle) |
| **pyproject.toml** | Project config file (like pom.xml) |
| **async/await** | Non-blocking execution (like CompletableFuture) |
| **Coroutine** | An async function (can be suspended/resumed) |
| **Event Loop** | Manages async execution (like NIO event loop) |
| **Decorator** | Function that wraps another function (@something) |
| **Generator** | Function that yields values one at a time |
| **Context Manager** | Object for `with` statement (auto-cleanup) |
| **Type Hint** | Optional type annotation (`x: int = 5`) |
| **f-string** | Format string (`f"Hello {name}"`) |
| **List Comprehension** | Compact list creation (`[x*2 for x in items]`) |
| **dunder** | Double-underscore methods (`__init__`, `__str__`) |
| **pip** | Basic package installer (Poetry is better) |

---

## 🗄️ Database Terms

| Term | Definition |
|------|-----------|
| **ORM** | Object-Relational Mapping (SQLAlchemy) |
| **Migration** | Versioned database schema change |
| **Flyway** | Migration tool/pattern (version-ordered SQL files) |
| **Session** | Database connection wrapper (like EntityManager) |
| **DSN** | Data Source Name (connection string) |
| **Pool** | Connection pool (reuse DB connections) |
| **Async Engine** | Non-blocking database engine |

---

## 🔐 Security Terms

| Term | Definition |
|------|-----------|
| **JWT** | JSON Web Token (contains user identity) |
| **Bearer Token** | Auth header format: `Authorization: Bearer <token>` |
| **OAuth2** | Authorization framework |
| **OIDC** | OpenID Connect (identity layer on OAuth2) |
| **Scope** | Permission level in a token |
| **CORS** | Cross-Origin Resource Sharing (browser security) |

---

## 📁 File/Folder Reference

| Name | What It Is |
|------|-----------|
| `__init__.py` | Makes a folder a Python package (can be empty) |
| `conftest.py` | pytest configuration/fixtures file |
| `.env` | Environment variables (not committed to git) |
| `pyproject.toml` | Project dependencies and metadata |
| `poetry.lock` | Exact locked dependency versions |
| `Dockerfile` | Container build instructions |
| `docker-compose.yml` | Multi-container orchestration |
| `justfile` / `Makefile` | Task runner commands |
| `migrations/` | Database schema evolution files |

---

## 🔗 Acronyms

| Acronym | Full Form |
|---------|-----------|
| API | Application Programming Interface |
| ASGI | Asynchronous Server Gateway Interface |
| CRUD | Create, Read, Update, Delete |
| DAG | Directed Acyclic Graph |
| DI | Dependency Injection |
| DTO | Data Transfer Object |
| HITL | Human In The Loop |
| LLM | Large Language Model |
| MCP | Model Context Protocol |
| ORM | Object-Relational Mapping |
| OTEL | OpenTelemetry |
| RAG | Retrieval-Augmented Generation |
| REST | Representational State Transfer |
| SSE | Server-Sent Events |
| UUID | Universally Unique Identifier |

---

## 📚 Where to Learn More

| Topic | Resource |
|-------|----------|
| Python | [python.org tutorial](https://docs.python.org/3/tutorial/) |
| FastAPI | [fastapi.tiangolo.com](https://fastapi.tiangolo.com/) |
| Pydantic | [docs.pydantic.dev](https://docs.pydantic.dev/) |
| SQLAlchemy | [docs.sqlalchemy.org](https://docs.sqlalchemy.org/) |
| LangChain | [python.langchain.com](https://python.langchain.com/) |
| LangGraph | [langchain-ai.github.io/langgraph](https://langchain-ai.github.io/langgraph/) |
| pytest | [docs.pytest.org](https://docs.pytest.org/) |
| Poetry | [python-poetry.org](https://python-poetry.org/) |
| MCP | [modelcontextprotocol.io](https://modelcontextprotocol.io/) |
| AWS Bedrock | [AWS docs](https://docs.aws.amazon.com/bedrock/) |

---

## 🎉 Congratulations!

You've completed the learning guide! Here's your next steps:

1. **Start coding** — pick a small bug or feature
2. **Read real code** — `agent_service.py` is the heart
3. **Run tests** — understand by modifying tests
4. **Ask questions** — reference this guide for terminology
5. **Build a feature** — best way to learn!

---

> 💡 **Tip:** Keep this glossary open while reading code. Most confusion comes from unfamiliar terms.

