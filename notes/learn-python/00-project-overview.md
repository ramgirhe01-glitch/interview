# Lesson 00 — Understanding Your Project 🗺️

Before learning Python syntax, let's understand **what you're building**. Everything you learn
will connect back to this.

---

## 📖 What is this project?

Your repo, `agentic-ai-service-backend`, is a **web backend** — a program that runs on a server
and answers requests over the internet (via HTTP). It's built with **FastAPI**, a popular Python
web framework.

Its job: **orchestrate AI agents**. An "agent" is an AI (powered by AWS Bedrock LLMs) that can
use tools and follow instructions. Your service lets multiple customers ("tenants") create and
run these agents.

There are **three ways** agents run (from your `AGENTS.md`):
1. **Standalone Agents** — run one agent directly.
2. **Custom Workflows** — connect multiple agents into a user-defined graph (DAG).
3. **Dynamic Workflows** — a "supervisor" agent decides which sub-agents to call at runtime.

---

## 📁 How the code is organized

```
app/api/
├── endpoints/   # The "doors" — HTTP routes users call (agents.py, custom_workflows.py)
├── services/    # The "brains" — business logic (agent_service, execution_service, ...)
├── models/      # Data shapes — Pydantic schemas for requests/responses
├── utils/       # Helpers — LLM providers, checkpointer, tool cache
├── exceptions/  # Custom error types (all extend AgenticAIException)
├── config.py    # Reads settings from environment variables
└── db.py        # Database connection setup (PostgreSQL / MySQL)
```

Think of it like a restaurant:
- **endpoints/** = the waiters taking orders.
- **services/** = the kitchen doing the actual cooking.
- **models/** = the menu, defining exactly what an order looks like.
- **db.py** = the pantry/storage.
- **exceptions/** = "we're out of that dish" messages, done politely.

---

## 🔎 The request lifecycle (big picture)

```
User sends HTTP request
        │
        ▼
endpoints/agents.py   ── validates token, reads request body (a Pydantic model)
        │
        ▼
services/agent_service.py   ── does the real work
        │
        ▼
db.py / LangGraph / Bedrock   ── stores data, runs the AI
        │
        ▼
Response (JSON) sent back to user
```

---

## 🏋️ Exercise

Open your project and just **explore** — don't change anything yet:

1. Open `app/api/main.py`. This is where the app starts. Skim it.
2. Open `app/api/endpoints/agents.py`. Find a line starting with `@router.get` or
   `@router.post`. That's an endpoint. How many can you count?
3. Open `app/api/services/` folder. List 3 service files you see.

Write your answers as comments in a new file `learn-python/playground.py`:

```python
# main.py starts the FastAPI app
# I found ___ endpoints in agents.py
# Three services: ______, ______, ______
```

---

## 🔗 Project Connection

Every lesson after this will pull code from these exact folders. By knowing the map now,
you'll always understand *where* a concept lives and *why* it's there.

---

## ➡️ Next Step

Go to **[Lesson 01 — Syntax, Variables & Type Hints](./01-syntax-variables-typehints.md)** to
start writing and reading Python.

