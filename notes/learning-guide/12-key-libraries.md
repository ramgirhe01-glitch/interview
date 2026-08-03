# 12 — Key Libraries & Dependencies

> Understanding the major packages used in this project.

---

## 📦 Core Libraries Map

```
pyproject.toml dependencies
│
├── 🌐 Web Framework
│   ├── fastapi          → Web framework (like Spring Boot)
│   ├── uvicorn          → ASGI server (like Tomcat)
│   ├── starlette        → Low-level HTTP (FastAPI built on this)
│   └── sse-starlette    → Server-Sent Events streaming
│
├── 📋 Data & Validation
│   ├── pydantic         → Data validation (like Bean Validation)
│   ├── pydantic-settings → Configuration from env vars
│   └── orjson           → Fast JSON serialization
│
├── 🗄️ Database
│   ├── sqlalchemy       → ORM (like Hibernate)
│   ├── psycopg          → PostgreSQL async driver
│   ├── psycopg2-binary  → PostgreSQL sync driver
│   └── PyMySQL          → MySQL driver
│
├── 🤖 AI / LLM
│   ├── langchain        → LLM framework
│   ├── langchain-aws    → AWS Bedrock integration
│   ├── langchain-openai → OpenAI integration
│   ├── langgraph        → Agent state machine
│   ├── langgraph-checkpoint-postgres → State persistence
│   └── langsmith        → LLM observability/tracing
│
├── 🔧 Tools & Protocols
│   ├── mcp              → Model Context Protocol
│   ├── fastmcp          → MCP server framework
│   ├── langchain-mcp-adapters → MCP ↔ LangChain bridge
│   └── a2a-sdk          → Agent-to-Agent protocol
│
├── ☁️ AWS
│   ├── boto3            → AWS SDK (like aws-sdk-java)
│   ├── botocore         → AWS low-level SDK
│   └── azure-identity   → Azure auth (if needed)
│
├── 🔒 Security
│   ├── authlib          → OAuth2/OIDC
│   ├── python-jose      → JWT handling
│   ├── PyJWT            → JWT encoding/decoding
│   └── cryptography     → Encryption utilities
│
├── 📊 Observability
│   ├── opentelemetry-api    → Tracing API
│   ├── opentelemetry-sdk    → Tracing implementation
│   └── opentelemetry-instrumentation-langchain → AI tracing
│
├── 🗃️ Caching
│   └── redis            → Redis client (caching, pub/sub)
│
└── 🛠️ Utilities
    ├── httpx            → Async HTTP client (like Apache HttpClient)
    ├── python-dotenv    → Load .env files
    ├── PyYAML           → YAML parsing
    ├── tenacity         → Retry logic
    └── requests         → Sync HTTP client
```

---

## 🔑 Most Important Libraries (Deep Dive)

### 1. FastAPI + Uvicorn
```
FastAPI = The web framework (handles routes, validation, docs)
Uvicorn = The server (handles network connections, concurrency)

Together: Like Spring Boot + Tomcat
```

### 2. Pydantic
```
Purpose: Validate data, serialize/deserialize JSON
Used for: Request bodies, response models, configuration
Like: Lombok + Jackson + Bean Validation combined
```

### 3. SQLAlchemy
```
Purpose: Database ORM + query builder
Used for: All database operations (async mode)
Like: JPA/Hibernate
Key: Uses "Core" style (not ORM models) in this project
```

### 4. LangChain + LangGraph
```
LangChain: Framework for building AI applications
  - Messages (HumanMessage, AIMessage)
  - LLM connections (Bedrock, OpenAI)
  - Tools (function calling)
  - Prompts (template management)

LangGraph: Orchestration engine for multi-step AI agents
  - StateGraph (define workflow)
  - Nodes (processing steps)
  - Edges (transitions)
  - Checkpointing (state persistence)
```

### 5. boto3 (AWS SDK)
```
Purpose: Interact with AWS services
Used for: Bedrock (LLMs), SQS (queues), Secrets Manager, KMS
Like: aws-sdk-java
```

### 6. Redis
```
Purpose: In-memory data store
Used for: Caching model metadata, execution traces, rate limiting
Like: Spring Data Redis
```

### 7. httpx
```
Purpose: Async HTTP client
Used for: Calling external services, MCP servers
Like: Apache HttpClient / WebClient
Key: Supports async, connection pooling, streaming
```

### 8. OpenTelemetry
```
Purpose: Distributed tracing & metrics
Used for: Request tracing, LLM call monitoring
Like: Micrometer + Zipkin/Jaeger
```

---

## 📊 Library Version Quick Reference

| Library | Version | Notes |
|---------|---------|-------|
| Python | ≥3.11 | Required |
| FastAPI | 0.136.0 | Web framework |
| Pydantic | 2.11.7 | Validation (v2!) |
| SQLAlchemy | 2.0.41 | ORM (v2!) |
| LangChain | 1.2.0 | AI framework |
| LangGraph | 1.0.10 | Agent orchestration |
| boto3 | 1.42.63 | AWS SDK |
| Redis | 7.3.0 | Cache client |
| uvicorn | 0.35.0 | ASGI server |
| Poetry | 2.3.4 | Package manager |

---

## 🔄 How Libraries Connect

```
Request → FastAPI (routing) 
        → Pydantic (validate request body)
        → Service (business logic)
        → SQLAlchemy (database query)
        → LangChain (build AI prompt)
        → LangGraph (orchestrate agent)
        → boto3/Bedrock (call LLM)
        → MCP/httpx (call tools)
        → Redis (cache results)
        → OpenTelemetry (trace everything)
        → Pydantic (serialize response)
        → FastAPI (send HTTP response)
```

---

## 💡 When You See These Imports

```python
# Seeing these? Here's what they do:

from fastapi import FastAPI, APIRouter, Depends, HTTPException
# → Web framework stuff (routing, DI, errors)

from pydantic import BaseModel, Field, field_validator
# → Data validation models

from sqlalchemy import select, and_, Column, String
# → Database queries

from langchain_core.messages import HumanMessage, AIMessage
# → AI conversation messages

from langgraph.graph import StateGraph, START, END
# → Agent workflow graph

from langchain_aws import ChatBedrock
# → AWS LLM connection

import boto3
# → AWS services (S3, SQS, Secrets, etc.)

from redis.asyncio import Redis
# → Redis cache operations

import httpx
# → HTTP calls to external services

from opentelemetry import trace
# → Distributed tracing
```

---

## ▶️ Next: [13-common-patterns.md](./13-common-patterns.md) — Code patterns & conventions

