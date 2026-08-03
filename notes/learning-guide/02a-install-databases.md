# 02a — Install MySQL, PostgreSQL & Redis (Windows)

> Step-by-step installation guide for all three databases used in this project.

---

## 🎯 What Each Database Does in This Project

| Database | Purpose | Port |
|----------|---------|------|
| **MySQL** | Main application config storage (agents, tools, workflows) | 3307 |
| **PostgreSQL** | LangGraph checkpoints (agent state/memory persistence) | 5432 |
| **Redis** | Caching (model metadata, execution traces, rate limiting) | 6379 |

---

## 🐳 Option A: Docker (Recommended — Easiest)

> Use Docker for all three. No native installs needed. One command to start everything.

### Step 1: Install Docker Desktop

```powershell
# Download from: https://www.docker.com/products/docker-desktop/
# Install → Restart PC → Open Docker Desktop → Wait for it to start

# Verify Docker is running:
docker --version
docker ps
```

### Step 2: Create a Combined Docker Compose File

The project already has MySQL in `scripts/docker-compose.db.yml`. Let's create a complete one with all three:

Create file: `scripts/docker-compose.all.yml`

```yaml
version: '3.9'

services:
  # ═══════════════════════════════════════════════
  # MySQL 8.0 — Main application database
  # ═══════════════════════════════════════════════
  mysql:
    container_name: agentic-mysql
    image: mysql:8.0
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: agenticconfig
      MYSQL_USER: admin
      MYSQL_PASSWORD: Password123
    ports:
      - "3307:3306"       # Access on localhost:3307
    command: ["--default-authentication-plugin=mysql_native_password", "--max-connections=200"]
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-pPassword123"]
      interval: 5s
      timeout: 3s
      retries: 20
      start_period: 5s
    volumes:
      - mysql_data:/var/lib/mysql

  # ═══════════════════════════════════════════════
  # PostgreSQL 16 — LangGraph checkpoints
  # ═══════════════════════════════════════════════
  postgres:
    container_name: agentic-postgres
    image: postgres:16
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: langgraph_checkpoints
    ports:
      - "5432:5432"       # Access on localhost:5432
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 3s
      retries: 20
      start_period: 5s
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # ═══════════════════════════════════════════════
  # Redis 7 — Caching & pub/sub
  # ═══════════════════════════════════════════════
  redis:
    container_name: agentic-redis
    image: redis:7-alpine
    restart: unless-stopped
    ports:
      - "6379:6379"       # Access on localhost:6379
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 20
    volumes:
      - redis_data:/data

volumes:
  mysql_data:
  postgres_data:
  redis_data:
```

### Step 3: Start All Services

```powershell
# Navigate to project scripts folder
cd "D:\AI backend\agentic-ai-service-backend-develop\agentic-ai-service-backend-develop\scripts"

# Start all three databases
docker compose -f docker-compose.all.yml up -d

# Check status (all should show "healthy")
docker ps

# Expected output:
# agentic-mysql      mysql:8.0        0.0.0.0:3307->3306   healthy
# agentic-postgres   postgres:16      0.0.0.0:5432->5432   healthy
# agentic-redis      redis:7-alpine   0.0.0.0:6379->6379   healthy
```

### Step 4: Verify Connections

```powershell
# Test MySQL
docker exec -it agentic-mysql mysql -uadmin -pPassword123 agenticconfig -e "SELECT 1;"

# Test PostgreSQL
docker exec -it agentic-postgres psql -U postgres -d langgraph_checkpoints -c "SELECT 1;"

# Test Redis
docker exec -it agentic-redis redis-cli ping
# Should return: PONG
```

### Step 5: Stop / Restart / Reset

```powershell
# Stop all (keeps data)
docker compose -f scripts/docker-compose.all.yml down

# Stop and DELETE all data (fresh start)
docker compose -f scripts/docker-compose.all.yml down -v

# Restart
docker compose -f scripts/docker-compose.all.yml restart

# View logs
docker logs agentic-mysql
docker logs agentic-postgres
docker logs agentic-redis
```

---

## 📋 Your `.env` File (After Docker Setup)

```env
# MySQL
DB_HOST=127.0.0.1
DB_PORT=3307
DB_USERNAME=admin
DB_PASSWORD=Password123
DB_NAME=agenticconfig
DB_TYPE=postgres

# PostgreSQL
POSTGRES_HOST=127.0.0.1
POSTGRES_PORT=5432
POSTGRES_USERNAME=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DATABASE=langgraph_checkpoints

# Redis
REDIS_HOST=127.0.0.1
REDIS_PORT=6379
```

---

---

## 💻 Option B: Native Installation (Without Docker)

> Only use this if Docker is not available on your machine.

---

### 🐬 Install MySQL 8.0 Natively

#### Download & Install:
1. Go to: https://dev.mysql.com/downloads/installer/
2. Download **MySQL Installer (Community)**
3. Run installer → Choose **"Custom"** install
4. Select: **MySQL Server 8.0** + **MySQL Workbench** (GUI tool)
5. Click Next → Execute

#### Configure During Install:
- Authentication: **Use Legacy Authentication** (mysql_native_password)
- Root password: `root`
- Add user: username=`admin`, password=`Password123`

#### After Install:
```powershell
# Verify MySQL is running
mysql --version

# Connect to MySQL
mysql -u admin -pPassword123

# Create the database
CREATE DATABASE agenticconfig;
SHOW DATABASES;
EXIT;
```

#### MySQL Workbench (GUI):
- Open MySQL Workbench
- Connect to: `localhost:3306` with `admin`/`Password123`
- You can browse tables, run queries visually

#### Start/Stop MySQL Service:
```powershell
# Check status
Get-Service -Name "MySQL80"

# Start
Start-Service -Name "MySQL80"

# Stop
Stop-Service -Name "MySQL80"
```

---

### 🐘 Install PostgreSQL 16 Natively

#### Download & Install:
1. Go to: https://www.postgresql.org/download/windows/
2. Download the **EDB installer** (includes pgAdmin GUI)
3. Run installer → Choose all components:
   - ✅ PostgreSQL Server
   - ✅ pgAdmin 4 (GUI tool)
   - ✅ Command Line Tools
4. Set superuser password: `postgres`
5. Port: `5432` (default)
6. Finish installation

#### After Install:
```powershell
# Verify PostgreSQL is running
psql --version

# Connect (will prompt for password: postgres)
psql -U postgres

# Create the checkpoint database
CREATE DATABASE langgraph_checkpoints;
\l          -- list databases
\q          -- quit
```

#### pgAdmin 4 (GUI):
- Open pgAdmin 4 from Start Menu
- Connect to: `localhost:5432` with `postgres`/`postgres`
- Right-click "Databases" → Create → Database → `langgraph_checkpoints`

#### Start/Stop PostgreSQL Service:
```powershell
# Check status
Get-Service -Name "postgresql*"

# Start
Start-Service -Name "postgresql-x64-16"

# Stop
Stop-Service -Name "postgresql-x64-16"
```

---

### 🔴 Install Redis Natively (Windows)

> Note: Redis doesn't officially support Windows. Use one of these options:

#### Option 1: Memurai (Redis-compatible for Windows) — Recommended
1. Go to: https://www.memurai.com/get-memurai
2. Download **Memurai Developer** (free)
3. Install → Runs as Windows service on port 6379
4. Test:
```powershell
memurai-cli ping
# Returns: PONG
```

#### Option 2: WSL2 (Windows Subsystem for Linux)
```powershell
# Enable WSL (run as Admin)
wsl --install

# Restart PC, then open Ubuntu terminal:
sudo apt update
sudo apt install redis-server
sudo service redis-server start
redis-cli ping
# Returns: PONG
```

#### Option 3: Redis via Docker (simplest)
```powershell
# Just run Redis in Docker even if you install MySQL/Postgres natively
docker run -d --name agentic-redis -p 6379:6379 redis:7-alpine
docker exec -it agentic-redis redis-cli ping
```

---

---

## 🔧 GUI Tools for Database Management

| Database | GUI Tool | Download |
|----------|----------|----------|
| MySQL | **MySQL Workbench** | Included in MySQL installer |
| MySQL | **DBeaver** (free, universal) | https://dbeaver.io/ |
| PostgreSQL | **pgAdmin 4** | Included in PostgreSQL installer |
| PostgreSQL | **DBeaver** (free, universal) | https://dbeaver.io/ |
| Redis | **RedisInsight** | https://redis.com/redis-enterprise/redis-insight/ |
| All | **DataGrip** (JetBrains, paid) | https://www.jetbrains.com/datagrip/ |

> 💡 **DBeaver** is recommended — it's free and works with MySQL + PostgreSQL + Redis all in one tool. Like a universal database IDE.

---

## ✅ Verification Script

Run this after setup to verify everything works:

```powershell
# Save as: verify-databases.ps1

Write-Host "=== Checking MySQL ===" -ForegroundColor Cyan
try {
    docker exec agentic-mysql mysql -uadmin -pPassword123 -e "SELECT 'MySQL OK' AS status;" 2>$null
    Write-Host "✅ MySQL is running on port 3307" -ForegroundColor Green
} catch {
    Write-Host "❌ MySQL is NOT running" -ForegroundColor Red
}

Write-Host ""
Write-Host "=== Checking PostgreSQL ===" -ForegroundColor Cyan
try {
    docker exec agentic-postgres psql -U postgres -c "SELECT 'PostgreSQL OK' AS status;" 2>$null
    Write-Host "✅ PostgreSQL is running on port 5432" -ForegroundColor Green
} catch {
    Write-Host "❌ PostgreSQL is NOT running" -ForegroundColor Red
}

Write-Host ""
Write-Host "=== Checking Redis ===" -ForegroundColor Cyan
try {
    $result = docker exec agentic-redis redis-cli ping 2>$null
    if ($result -eq "PONG") {
        Write-Host "✅ Redis is running on port 6379" -ForegroundColor Green
    }
} catch {
    Write-Host "❌ Redis is NOT running" -ForegroundColor Red
}

Write-Host ""
Write-Host "=== All checks complete ===" -ForegroundColor Yellow
```

---

## 🗺️ Connection Details Summary

| Service | Host | Port | Username | Password | Database |
|---------|------|------|----------|----------|----------|
| MySQL | 127.0.0.1 | 3307 | admin | Password123 | agenticconfig |
| PostgreSQL | 127.0.0.1 | 5432 | postgres | postgres | langgraph_checkpoints |
| Redis | 127.0.0.1 | 6379 | — | — | — |

---

## 🔄 Daily Workflow

```powershell
# Morning: Start databases
docker compose -f scripts/docker-compose.all.yml up -d

# Work: Run the app
just dev

# Evening: Stop databases (optional, they auto-start with Docker Desktop)
docker compose -f scripts/docker-compose.all.yml down
```

---

## ▶️ Back to: [02-project-setup.md](./02-project-setup.md) — Continue with project setup

