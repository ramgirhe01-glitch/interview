# 🚀 Agentic AI Service Backend — Complete Learning Guide

> **For:** Java developers transitioning to Python & this project  
> **Time Estimate:** ~2-3 weeks for deep understanding  
> **Python Version:** 3.11+  
> **Framework:** FastAPI + LangGraph + LangChain

---

## 📖 Reading Order (Follow This Path)

| # | File | Topic | Time |
|---|------|-------|------|
| 1 | [01-python-for-java-developers.md](./01-python-for-java-developers.md) | Python syntax, OOP, async — mapped from Java concepts | 2-3 days |
| 2 | [02-project-setup.md](./02-project-setup.md) | Environment setup, running the app locally | 1-2 hours |
| 2a | [02a-install-databases.md](./02a-install-databases.md) | Install MySQL, PostgreSQL & Redis (Docker + Native) | 30 min |
| 3 | [03-project-architecture.md](./03-project-architecture.md) | High-level architecture, folder structure, request flow | 1 day |
| 4 | [04-fastapi-basics.md](./04-fastapi-basics.md) | FastAPI framework (like Spring Boot for Python) | 1 day |
| 5 | [05-pydantic-models.md](./05-pydantic-models.md) | Data validation & serialization (like Java DTOs + validation) | Half day |
| 6 | [06-sqlalchemy-database.md](./06-sqlalchemy-database.md) | ORM layer (like JPA/Hibernate) | 1 day |
| 7 | [07-langchain-langgraph.md](./07-langchain-langgraph.md) | AI Agent framework — core of this project | 2 days |
| 8 | [08-services-layer.md](./08-services-layer.md) | Business logic, service patterns | 1 day |
| 9 | [09-mcp-tools.md](./09-mcp-tools.md) | MCP (Model Context Protocol) tool integration | Half day |
| 10 | [10-testing-guide.md](./10-testing-guide.md) | Unit tests, mocking, pytest | 1 day |
| 11 | [11-deployment-docker.md](./11-deployment-docker.md) | Docker, CI/CD, deployment | Half day |
| 12 | [12-key-libraries.md](./12-key-libraries.md) | Important dependencies explained | Half day |
| 13 | [13-common-patterns.md](./13-common-patterns.md) | Patterns & conventions used in code | Half day |
| 14 | [14-glossary.md](./14-glossary.md) | Terminology reference | Reference |
| 15 | [15-hands-on-new-feature.md](./15-hands-on-new-feature.md) | Build a real feature end-to-end | 1 hour |
| 16 | [16-cheat-sheet.md](./16-cheat-sheet.md) | One-page quick reference (print this!) | Reference |

---

## 🏗️ What This Project Does (30-Second Summary)

This is a **multi-tenant backend service** that:
1. Lets users **create AI agents** (with LLMs, tools, guardrails)
2. **Executes agents** via LangGraph (standalone, in workflows, or dynamically)
3. Supports **MCP tools** (external tool servers), **memory**, **human-in-the-loop**
4. Exposes a **REST API** (FastAPI) consumed by a frontend (AI Studio)

---

## 🗺️ Quick Architecture Map

```
Client Request (REST API)
       │
       ▼
┌─────────────────┐
│  FastAPI Router  │  ← endpoints/ (like Spring Controllers)
│  (endpoints/)   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Service Layer   │  ← services/ (like Spring Services)
│  (services/)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  LangGraph/AI    │  ← Agent execution engine
│  (agent/)       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Database        │  ← PostgreSQL (SQLAlchemy ORM)
│  (db.py)        │
└─────────────────┘
```

---

## 💡 Java → Python Mental Model

| Java | Python (This Project) |
|------|----------------------|
| Spring Boot | FastAPI |
| Spring Controllers | FastAPI Routers (endpoints/) |
| Spring Services | Service classes (services/) |
| JPA/Hibernate | SQLAlchemy |
| DTOs + Bean Validation | Pydantic Models |
| Maven/Gradle | Poetry (pyproject.toml) |
| application.yml | .env + config.py |
| JUnit + Mockito | pytest + pytest-mock |
| CompletableFuture | async/await (asyncio) |
| Jackson JSON | Pydantic + orjson |
| Lombok @Data | Python dataclasses / Pydantic BaseModel |

---

## 🎯 First Week Goals

- [ ] Set up local environment (Python 3.11+, Poetry, Docker)
- [ ] Run the app locally (`just dev`)
- [ ] Read through `app/api/main.py` — understand startup
- [ ] Trace one API call: Create Agent → Execute Agent
- [ ] Run unit tests (`poetry run pytest tests/unit/ -v`)
- [ ] Modify something small, see it work

---

## 📁 Key Files to Read First

1. `app/api/main.py` — Application entry point
2. `app/api/config.py` — All configuration
3. `app/api/endpoints/agents.py` — Agent CRUD endpoints
4. `app/api/services/agent_service.py` — Core agent logic
5. `app/api/models/agent.py` — Agent data models
6. `AGENTS.md` — Developer guide (project root)

---

> **Tip:** Open each guide file in order. Each builds on the previous one. Code examples reference actual files in this project.


