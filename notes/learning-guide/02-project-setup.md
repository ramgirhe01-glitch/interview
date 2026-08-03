# 02 — Project Setup & Running Locally

> Get the project running on your Windows machine step by step.

---

## 📋 Prerequisites

| Tool | Version | Purpose | Install |
|------|---------|---------|---------|
| Python | 3.11+ | Runtime | [python.org](https://www.python.org/downloads/) |
| Poetry | 2.x | Dependency manager (like Maven) | `pip install poetry` |
| Docker Desktop | Latest | Run MySQL/Redis locally | [docker.com](https://www.docker.com/products/docker-desktop/) |
| Git | Latest | Version control | [git-scm.com](https://git-scm.com/) |
| just | Latest | Task runner (like Make) | `winget install casey.just` |
| IDE | PyCharm or VS Code | Development | JetBrains / Microsoft |

---

## 🔧 Step 1: Install Python 3.11+

```powershell
# Check current Python version
python --version

# If not installed, download from python.org
# During install: ✅ Check "Add Python to PATH"
# After install, verify:
python --version   # Should show 3.11.x or higher
pip --version      # Should work
```

---

## 🔧 Step 2: Install Poetry (Dependency Manager)

Poetry = Maven/Gradle for Python. Manages dependencies via `pyproject.toml`.

```powershell
# Install Poetry
pip install poetry

# Verify
poetry --version

# Configure Poetry to create virtualenv in project folder (recommended)
poetry config virtualenvs.in-project true
```

---

## 🔧 Step 3: Clone & Install Dependencies

```powershell
# Navigate to project
cd "D:\AI backend\agentic-ai-service-backend-develop\agentic-ai-service-backend-develop"

# Install all dependencies (reads pyproject.toml → creates .venv/)
poetry install

# This creates a virtual environment with all packages
# Equivalent to: mvn install
```

### What `poetry install` Does:
1. Reads `pyproject.toml` (like `pom.xml`)
2. Creates `.venv/` folder (isolated Python environment)
3. Installs all dependencies into `.venv/`
4. Makes the project importable

---

## 🔧 Step 4: Set Up Environment Variables

Create a `.env` file in the project root:

```powershell
# Copy example or create new
# File: .env (in project root)
```

**Required `.env` contents:**
```env
# Database (MySQL - local)
DB_HOST=127.0.0.1
DB_PORT=3307
DB_USERNAME=admin
DB_PASSWORD=Password123
DB_NAME=agenticconfig
DB_TYPE=postgres

# PostgreSQL (for LangGraph checkpoints)
POSTGRES_HOST=127.0.0.1
POSTGRES_PORT=5432
POSTGRES_USERNAME=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DATABASE=langgraph_checkpoints

# AWS (use dummy for local dev)
AWS_DEFAULT_REGION=us-east-1
AWS_ACCESS_KEY_ID=test
AWS_SECRET_ACCESS_KEY=test

# Redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379

# Security
TOKEN_AUDIENCE=your-audience

# LLM Service
LLM_SERVICE_URL=http://localhost:8080
```

---

## 🔧 Step 5: Start Local Database & Redis

> 📘 **Need to install MySQL, PostgreSQL, or Redis first?**  
> See: [02a-install-databases.md](./02a-install-databases.md) — Full installation guide (Docker & Native)

```powershell
# Option A: Start ALL databases with one command (recommended)
docker compose -f scripts/docker-compose.all.yml up -d

# Option B: Using 'just' (MySQL only)
just db-up

# Option C: Using original Docker Compose (MySQL only)
docker compose -f scripts/docker-compose.db.yml up -d

# Verify containers are running
docker ps
```

---

## 🔧 Step 6: Run the Application

```powershell
# Option A: Using 'just' (recommended)
just dev

# Option B: Direct command
poetry run uvicorn app.api.main:app --reload --host 127.0.0.1 --port 8001 --log-level debug

# Option C: Using Makefile
make dev
```

### What This Does:
- Starts FastAPI server on `http://127.0.0.1:8001`
- `--reload` = hot reload on file changes (like Spring DevTools)
- Swagger UI at: `http://127.0.0.1:8001/docs`
- OpenAPI JSON at: `http://127.0.0.1:8001/openapi.json`

---

## 🔧 Step 7: Verify It Works

```powershell
# Health check
curl http://127.0.0.1:8001/health

# Open Swagger UI in browser
Start-Process "http://127.0.0.1:8001/docs"
```

---

## 🧪 Running Tests

```powershell
# Run ALL unit tests
poetry run pytest tests/unit/ -v

# Run specific test file
poetry run pytest tests/unit/services/test_agent_service.py -v

# Run with coverage
poetry run pytest tests/unit/ --cov=app --cov-report=html

# Run a single test by name
poetry run pytest tests/unit/ -k "test_create_agent" -v
```

---

## 🛠️ Useful Commands Cheat Sheet

| Command | Purpose |
|---------|---------|
| `just dev` | Start dev server with hot reload |
| `just db-up` | Start MySQL + Redis containers |
| `just db-down` | Stop database containers |
| `just db-reset` | Reset database (drop + recreate) |
| `just db-shell` | Open MySQL CLI |
| `poetry install` | Install/update dependencies |
| `poetry add <package>` | Add new dependency (like `mvn add`) |
| `poetry run pytest` | Run tests |
| `poetry shell` | Activate virtualenv in terminal |

---

## 🏗️ Project Root File Reference

| File | Purpose | Java Equivalent |
|------|---------|-----------------|
| `pyproject.toml` | Dependencies + build config | `pom.xml` / `build.gradle` |
| `poetry.lock` | Locked dependency versions | `pom.xml` resolved |
| `.env` | Environment variables | `application.properties` |
| `Makefile` | Build commands | Gradle tasks |
| `justfile` | Task runner commands | Gradle tasks (better) |
| `Dockerfile` | Container build | Same |
| `conftest.py` | Test configuration | Test base class |
| `requirements.txt` | Pip-style deps (for Docker) | — |

---

## ⚠️ Common Issues & Fixes

### "Module not found" errors
```powershell
# Make sure you're using poetry's virtualenv
poetry run python -c "import app; print('OK')"

# Or activate the virtualenv first
poetry shell
python -c "import app; print('OK')"
```

### Port already in use
```powershell
# Find what's using port 8001
netstat -ano | findstr :8001
# Kill it
taskkill /PID <pid> /F
```

### Docker not running
```powershell
# Start Docker Desktop, then:
docker ps  # Should list containers
```

---

## 📁 IDE Setup (PyCharm / IntelliJ)

1. **Open project folder** in PyCharm/IntelliJ
2. **Set Python Interpreter:**
   - File → Settings → Project → Python Interpreter
   - Select: `.venv/Scripts/python.exe` (in project root)
3. **Mark source roots:**
   - Right-click `app/` → Mark as → Sources Root
   - Right-click `app/api/` → Mark as → Sources Root
4. **Install Python plugin** (if using IntelliJ instead of PyCharm)

---

## ▶️ Next: [03-project-architecture.md](./03-project-architecture.md) — Understand the overall architecture

