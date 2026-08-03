# 11 — Docker & Deployment

> How this project is containerized and deployed.

---

## 🐳 Docker Overview

The project uses a **multi-stage Docker build** for production:

```dockerfile
# Dockerfile — Simplified explanation

# Stage 1: Get system libraries
FROM debian:trixie-slim AS libmagic-source
RUN apt-get install -y libmagic1

# Stage 2: Build application image
FROM python:3.13-trixie

# Set working directory
WORKDIR /app

# Install Poetry
RUN pip install poetry==2.3.4

# Copy dependency files first (cache layer)
COPY pyproject.toml ./

# Install dependencies (without project code = Docker layer caching!)
RUN poetry install --no-root --only main

# Copy application code
COPY app/ ./app/
COPY logging_lib/ ./logging_lib/
COPY security_lib/ ./security_lib/
COPY migrations/ ./migrations/

# Run the application
CMD ["uvicorn", "app.api.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

---

## 🏗️ Docker Concepts for Java Developers

| Docker | Java Equivalent | Purpose |
|--------|----------------|---------|
| `Dockerfile` | Gradle `jib` config | Build image definition |
| `FROM python:3.13` | `FROM openjdk:17` | Base image |
| `COPY pyproject.toml` | Copy `pom.xml` first | Dependency caching |
| `RUN poetry install` | `mvn dependency:resolve` | Install deps |
| `COPY app/` | Copy `.jar` | Application code |
| `CMD ["uvicorn"...]` | `CMD ["java", "-jar"...]` | Start command |
| `docker-compose.yml` | Same | Multi-container setup |

---

## 🗄️ Local Docker Compose (`scripts/docker-compose.db.yml`)

For local development, Docker Compose runs supporting services:

```yaml
# scripts/docker-compose.db.yml
version: '3.8'
services:
  mysql:
    image: mysql:8.0
    container_name: agentic-mysql
    ports:
      - "3307:3306"
    environment:
      MYSQL_DATABASE: agenticconfig
      MYSQL_USER: admin
      MYSQL_PASSWORD: Password123
      MYSQL_ROOT_PASSWORD: root
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 5s
      timeout: 5s
      retries: 10

  redis:
    image: redis:7
    container_name: agentic-redis
    ports:
      - "6379:6379"
```

### Running Local Services:
```powershell
# Start MySQL + Redis
just db-up
# OR
docker compose -f scripts/docker-compose.db.yml up -d

# Stop
just db-down

# Reset (delete data and restart)
just db-reset
```

---

## 🚀 Building & Running Docker Image

```powershell
# Build the image
docker build -t agentic-ai-service .

# Run the container
docker run -p 8000:8000 \
  -e DB_HOST=host.docker.internal \
  -e DB_PORT=3307 \
  -e DB_USERNAME=admin \
  -e DB_PASSWORD=Password123 \
  -e DB_NAME=agenticconfig \
  -e AWS_DEFAULT_REGION=us-east-1 \
  -e REDIS_HOST=host.docker.internal \
  -e REDIS_PORT=6379 \
  agentic-ai-service
```

---

## 📦 Deployment Architecture (Production)

```
┌─────────────────────────────────────────────┐
│              AWS ECS / EKS                    │
│                                              │
│  ┌──────────────────────────────────────┐   │
│  │  Service: agentic-ai-service          │   │
│  │  ┌──────┐ ┌──────┐ ┌──────┐         │   │
│  │  │Pod 1 │ │Pod 2 │ │Pod 3 │ (auto)  │   │
│  │  └──────┘ └──────┘ └──────┘         │   │
│  └──────────────────────────────────────┘   │
│                     │                        │
│                     ▼                        │
│  ┌──────────────────────────────────────┐   │
│  │  AWS RDS (PostgreSQL)                 │   │
│  └──────────────────────────────────────┘   │
│                                              │
│  ┌──────────────────────────────────────┐   │
│  │  AWS ElastiCache (Redis)              │   │
│  └──────────────────────────────────────┘   │
│                                              │
│  ┌──────────────────────────────────────┐   │
│  │  AWS Bedrock (LLMs)                   │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

---

## ⚙️ Environment Variables (Production)

```env
# Database
DB_HOST=rds-endpoint.amazonaws.com
DB_PORT=5432
DB_USERNAME=app_user
DB_PASSWORD=<from-secrets-manager>
DB_NAME=agenticconfig
DB_TYPE=postgres

# PostgreSQL (Checkpoints)
POSTGRES_HOST=rds-endpoint.amazonaws.com
POSTGRES_PORT=5432
POSTGRES_USERNAME=checkpoint_user
POSTGRES_PASSWORD=<from-secrets-manager>
POSTGRES_DATABASE=langgraph_checkpoints

# AWS
AWS_DEFAULT_REGION=us-east-1
# (IAM role assumed — no keys needed in production)

# Redis
REDIS_HOST=elasticache-endpoint.amazonaws.com
REDIS_PORT=6379

# Security
TOKEN_AUDIENCE=production-audience

# OpenTelemetry (Tracing)
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
OTEL_SERVICE_NAME=agentic-ai-service
```

---

## 🔄 CI/CD Pipeline (GitLab)

The project uses GitLab CI/CD (similar to GitHub Actions):

```
1. Developer pushes code
2. CI Pipeline triggers:
   ├── Lint (flake8)
   ├── Unit Tests (pytest)
   ├── Security Scan (SAST)
   ├── Build Docker Image
   └── Push to Container Registry
3. CD Pipeline deploys to environments:
   ├── Dev (auto-deploy)
   ├── Staging (auto-deploy)
   └── Production (manual approval)
```

---

## 🗄️ Database Migrations in Production

Migrations run **automatically on app startup**:

```python
# utils/flyway_migrations.py
# Called during app lifespan startup
async def apply_migrations():
    """
    1. Check flyway_schema_history table
    2. Find unapplied migrations in migrations/ folder
    3. Apply them in order
    4. Record in history table
    """
```

---

## 💡 Key Deployment Points

1. **Multi-worker mode** — Production runs with `--workers 4` (like Tomcat thread pool)
2. **Health checks** — `/health` endpoint for load balancer
3. **Graceful shutdown** — Lifespan events clean up connections
4. **Secrets** — AWS Secrets Manager (not in environment variables)
5. **Observability** — OpenTelemetry for traces, structured JSON logging
6. **Auto-scaling** — Based on CPU/memory metrics

---

## ▶️ Next: [12-key-libraries.md](./12-key-libraries.md) — Important dependencies explained

