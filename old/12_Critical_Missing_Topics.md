# 🔥 CRITICAL MISSING TOPICS — What You MUST Add to Your Preparation
### Based on Resume Analysis of Ram Girhe | Updated April 2026

---

# ⚠️ IMPORTANT RESUME CORRECTIONS FIRST

```
Your resume says "2+ years" but you told me "3 years".
If it's now April 2026 and you joined January 2024:
  → That's 2 years 3 months.
  → Say "2.5+ years" or "close to 3 years" — don't round up to 3.
  → Interviewers WILL calculate: Jan 2024 to April 2026 = ~2.3 years
  → Lying about experience is an instant rejection.
```

---

# PART 1: PROJECTS YOU WORKED ON BUT NOT COVERED IN GUIDES

Your resume lists **6 projects**: Agentic AI, RAG/Vector, DAA, ATR, AM, TS.
The guides only cover Agentic AI and RAG. **Interviewers WILL ask about ALL of them.**

---

## 1. Unified Data Access (DAA) — You MUST be able to explain this

```
What is DAA?
  A service that provides a unified API to query data from multiple sources
  (Data Lake, databases, APIs) using a single query language.

What is AST-style Query DSL?
  AST = Abstract Syntax Tree
  Instead of writing raw SQL, you send a structured JSON query:

  Traditional SQL:
    SELECT department, AVG(salary) FROM employees
    WHERE age > 25 GROUP BY department ORDER BY AVG(salary) DESC

  AST-style Query DSL (JSON):
  {
    "dataset": "employees",
    "filters": [
      { "field": "age", "operator": "gt", "value": 25 }
    ],
    "aggregations": [
      { "field": "salary", "function": "avg", "alias": "avg_salary" }
    ],
    "groupBy": [
      { "field": "department" },
      { "field": "hire_date", "timeInterval": "MONTH" }
    ],
    "orderBy": [
      { "field": "avg_salary", "direction": "DESC" }
    ],
    "limit": 100
  }

Why AST-style?
  - Language-agnostic (works with any backend: SQL, NoSQL, Data Lake)
  - Validatable (JSON schema validation before execution)
  - Composable (build queries programmatically)
  - Secure (no SQL injection possible)
```

### DAA Test Cases You Should Know:
```
✅ POSITIVE:
1. Simple query with single filter → 200
2. Query with multiple aggregations (sum, count, min, max, avg) → correct results
3. GroupBy with timeInterval (MONTH, WEEK, DAY) → correct grouping
4. Multi-field orderBy → correct sorting
5. Async lifecycle: submit → poll (PENDING→RUNNING→COMPLETED) → get results
6. Export query results to file
7. Schema inference: API returns column names + types for a dataset

❌ NEGATIVE:
8. Invalid dataset name → 404
9. Invalid aggregation function → 400
10. Filter on non-existent field → 400
11. Query timeout (exceeds SLA) → proper error, not hang
12. Unauthorized dataset access → 403

🌍 MULTI-REGION:
13. Same query returns same results across US/EU/AP
14. Dataset available in US but not AP → proper error in AP
```

**Interview Q:** *"Tell me about the DAA project."*
> "DAA is a Unified Data Access service that lets applications query data from FDS Data Lake using an AST-style query DSL. Instead of raw SQL, clients send structured JSON queries with filters, aggregations, groupBy, and ordering. I automated dataset configuration APIs and the async query lifecycle — submit, poll for status transitions, and retrieve results. I also validated schema inference, export workflows, and cross-region consistency across US, EU, and AP."

---

## 2. ATR, AM, TS — Even if smaller, prepare 2-3 lines each

**You MUST know what these abbreviations stand for and what you tested.**

```
⚠️ If you can't explain a project on your resume, interviewers will doubt
   EVERYTHING on your resume.

For each project, prepare:
1. What it does (1 sentence)
2. What you tested (1-2 sentences)
3. One interesting bug you found (1 sentence)

Template:
"[Project] is a [what it does]. I automated [X endpoints] covering
[what types of tests]. One interesting issue I found was [brief bug]."
```

---

# PART 2: MICROSERVICES CONCEPTS (CRITICAL — asked in 80% of SDET interviews)

---

## What is Microservices Architecture?

```
Monolith:                    Microservices:
┌──────────────────┐         ┌─────────┐ ┌─────────┐ ┌─────────┐
│   ALL CODE       │         │ Agent   │ │ LLM     │ │ Vector  │
│   IN ONE APP     │         │ Service │ │ Gateway │ │ Service │
│   • Users        │    →    └────┬────┘ └────┬────┘ └────┬────┘
│   • Orders       │              │           │           │
│   • Payments     │         ┌────┴────┐ ┌────┴────┐ ┌────┴────┐
│   • Reports      │         │   DB    │ │   DB    │ │ VectorDB│
└──────────────────┘         └─────────┘ └─────────┘ └─────────┘

Benefits:
• Independent deployment (update Agent without touching LLM)
• Independent scaling (scale Vector service during heavy search)
• Technology freedom (Agent in Java, Vector in Python)
• Fault isolation (LLM crash doesn't kill Agent service)

Challenges (interviewers love asking these):
• Network latency between services
• Data consistency across services
• Distributed tracing & debugging
• Service discovery
• Circuit breaker pattern
```

### Key Microservice Concepts:

| Concept | Explanation | Your Project Example |
|---------|-------------|---------------------|
| **API Gateway** | Single entry point for all clients | Routes to Agent, Vector, DAA services |
| **Service Discovery** | Services find each other dynamically | Agent service finds LLM Gateway |
| **Circuit Breaker** | Stop calling a failing service, use fallback | LLM primary fails → fallback model |
| **Eventual Consistency** | Data syncs across services over time (not instant) | Embedding model swap → async re-indexing |
| **Idempotency** | Same request twice = same result | DELETE /agent/123 twice → first 204, second 404 |
| **Rate Limiting** | Limit requests per time window | 429 Too Many Requests |
| **Health Check** | `/health` endpoint returns service status | Test across all regions |
| **Distributed Tracing** | Track request across multiple services | Trace agent execution through LLM → tools → response |
| **Retry with Backoff** | Retry failed requests with increasing delay | 1s → 2s → 4s → give up |
| **Dead Letter Queue** | Store failed messages for later processing | Failed async executions |

---

## Interview Questions on Microservices:

**Q: How do you test microservices differently from monoliths?**
> "In monoliths, you test one application. In microservices, you also test:
> 1. **Contract testing** — does Service A's request match Service B's expectation?
> 2. **Integration testing** — do services communicate correctly?
> 3. **Failure testing** — what happens when one service is down?
> 4. **Data consistency** — is data synchronized across services?
> 5. **Network resilience** — timeouts, retries, circuit breakers work?"

**Q: What is contract testing?**
> "Contract testing verifies that two services agree on the API format. For example, if Agent service expects LLM Gateway to return `{response: string, tokens: number}`, the contract test verifies this shape is maintained even when LLM Gateway is updated independently."

**Q: What is the difference between integration testing and E2E testing?**
> "Integration tests verify two services work together (Agent → LLM). E2E tests verify the entire user journey (Register agent → Configure → Execute → Get streaming result → Verify guardrails). In my project, FIT testing is integration-level, while Agentic AI E2E covers the full flow."

---

# PART 3: API DESIGN & REST CONCEPTS (Frequently missed by candidates)

---

## REST API Maturity Model (Richardson)
```
Level 0: Single endpoint, POST everything (/api with action in body)
Level 1: Resources (/users, /orders — but only POST)
Level 2: HTTP verbs (GET /users, POST /users, DELETE /users/123)
Level 3: HATEOAS (response includes links to related actions)

Most APIs are Level 2. Know this for interviews.
```

## Pagination Patterns (you work with cursor-based in RAG)
```
Offset-based:  GET /items?page=2&limit=10
  ✅ Simple, supports jump to any page
  ❌ Inconsistent if data changes between pages

Cursor-based:  GET /items?cursor=eyJpZCI6MTB9&limit=10
  ✅ Consistent even when data changes
  ✅ Better performance on large datasets
  ❌ Can't jump to arbitrary page
  
Your project uses cursor-based for RAG semantic search results.
```

## API Versioning
```
URL:     /api/v1/agents, /api/v2/agents
Header:  Accept: application/vnd.api.v1+json
Query:   /api/agents?version=1

Your project uses URL versioning: /api/v1/mcp-servers
```

## Idempotency (asked very often)
```
Idempotent = calling multiple times has same effect as calling once.

GET    /users/123       → Always returns same user     → IDEMPOTENT ✅
PUT    /users/123       → Always replaces with same data → IDEMPOTENT ✅
DELETE /users/123       → First: 204, Second: 404       → IDEMPOTENT ✅
POST   /users           → Creates new user EACH time    → NOT IDEMPOTENT ❌
PATCH  /users/123       → Depends on implementation     → USUALLY IDEMPOTENT ✅

Interview Q: "How do you make POST idempotent?"
Answer: Use an idempotency key (unique ID in header). Server checks if
request with that key was already processed.
  Header: Idempotency-Key: abc-123-unique
```

---

# PART 4: AUTHENTICATION & SECURITY DEEP DIVE

---

## JWT Token Structure (You work with 3-level JWT)
```
JWT = Header.Payload.Signature

Header:   {"alg": "RS256", "typ": "JWT"}
Payload:  {
  "sub": "user-123",
  "iss": "auth.siemens.com",
  "aud": "agentic-ai",
  "exp": 1714000000,
  "iat": 1713996400,
  "scope": "agentic.admin",
  "tenant_level": "APPLICATION",    ← Your 3-level isolation
  "tenant_id": "teamcenter",
  "provider_id": "siemens"
}
Signature: HMACSHA256(base64(header) + "." + base64(payload), secret)

Key claims to test:
• exp (expiry) — expired token → 401
• scope — insufficient scope → 403
• tenant_id — wrong tenant → 403
• iss (issuer) — wrong issuer → 401
• Tampered payload → signature invalid → 401
```

## OAuth2 Flow (Your project uses client_credentials)
```
Client Credentials Flow (service-to-service):
┌──────────┐                          ┌──────────────┐
│  Test     │ ── client_id + secret ──→ │  Auth Server │
│  Client   │ ←── access_token ──────── │  (OAuth2)    │
│           │                          └──────────────┘
│           │ ── Bearer token ────────→ ┌──────────────┐
│           │ ←── API response ──────── │  Agentic AI  │
└──────────┘                          └──────────────┘

What to test:
✅ Valid credentials → token returned
✅ Invalid client_id → 401
✅ Invalid secret → 401
✅ Token used after expiry → 401
✅ Token used for wrong service → 403
✅ Token refresh before expiry → seamless
✅ Rate limiting on token requests → 429
```

---

# PART 5: PERFORMANCE & NON-FUNCTIONAL TESTING

---

## Response Time SLAs (from your resume)
```
Your projects have specific SLAs:
• RAG search availability: 1 second after ingestion
• Agent sync execution: < 30 seconds
• API CRUD operations: < 2 seconds
• Bulk ingestion (100 records): < 10 seconds

How to test response time in REST Assured:
.then()
    .time(lessThan(2000L))  // milliseconds

How to report:
1. Measure response times in test
2. Flag violations as defects
3. Track trends over sprints (is it getting slower?)
```

## Concurrent Request Testing
```
Scenarios to test:
1. 10 concurrent agent creations → no data corruption
2. 5 concurrent queries on same dataset → all return correct results
3. Simultaneous read + write → no dirty reads
4. Rate limit enforcement under load → 429 after threshold
```

---

# PART 6: MOST IMPORTANT INTERVIEW QUESTIONS YOU'RE MISSING

---

## Q1: "Walk me through your test automation framework architecture."
```
This is the #1 MOST ASKED question for SDET roles. You MUST answer this confidently.

Your framework:
┌─────────────────────────────────────────────────────────┐
│                 TEST AUTOMATION FRAMEWORK                 │
├──────────────┬──────────────┬──────────────┬────────────┤
│  Feature     │  Step        │  Utilities   │  Config    │
│  Files       │  Definitions │              │            │
│  (.feature)  │  (Java)      │  (Java)      │(.properties)│
├──────────────┼──────────────┼──────────────┼────────────┤
│ Cucumber     │ REST Assured │ AuthHelper   │ us.props   │
│ scenarios    │ API calls    │ PayloadUtil  │ eu.props   │
│ Given/When/  │ Assertions   │ ConfigReader │ ap.props   │
│ Then         │ Data extract │ TestDataGen  │ credentials│
├──────────────┴──────────────┴──────────────┴────────────┤
│                    EXECUTION LAYER                        │
│  Gradle │ TestNG/JUnit │ CI/CD Pipeline │ Region Matrix │
├─────────────────────────────────────────────────────────┤
│                    REPORTING LAYER                        │
│  Cucumber HTML │ Trends │ JIRA Integration │ Slack Notify│
└─────────────────────────────────────────────────────────┘

Key design decisions to explain:
1. BDD with Cucumber → business-readable tests
2. REST Assured → fluent API testing
3. Config-driven → same code, different regions
4. Reusable specs → RequestSpec/ResponseSpec
5. Data cleanup → tests create and delete their own data
6. Parallel execution → region matrix in CI/CD
7. Retry mechanism → rerun.txt for flaky tests
8. Reporting → automated HTML reports + trends
```

---

## Q2: "What is the most critical bug you found?"
```
Prepare 3 bug stories (interviewers may ask for multiple):

Bug 1 (Security — Most Impressive):
"I found a tenant isolation vulnerability in the Agentic AI service.
Application A's token could access Application B's agent configurations
in the AP region. This was because the AP deployment had a misconfigured
tenant filter in the middleware. If this reached production, customers
could see each other's AI agent configurations, which is a critical
data breach. I discovered it during cross-tenant testing."

Bug 2 (Functional — Shows Debugging Skill):
"The RAG semantic search returned 0 results in EU even though records
were ingested. US and AP worked fine. I traced it to the embedding model
endpoint being misconfigured for EU — it pointed to a US endpoint,
causing a region mismatch in vector storage. I identified it by comparing
the embedding model config across all 3 regions."

Bug 3 (Performance — Shows SLA Awareness):
"After bulk ingestion of 100 records, search results were not available
within the 1-second SLA. Investigation showed the indexing was synchronous
instead of asynchronous, blocking the API response. After the fix, search
became available within 800ms."
```

---

## Q3: "How do you decide what to automate vs what to test manually?"

```
Automate:
✅ Repetitive tests (run every sprint)
✅ Multi-region validation (same test, 3 regions)
✅ Regression suite
✅ Data-driven tests (many inputs, same flow)
✅ API tests (fast, stable, no UI dependency)

Manual:
✅ Exploratory testing (new features, first time)
✅ Usability testing
✅ One-time verifications
✅ Complex visual validations
✅ Tests that are too flaky to automate reliably

Decision framework:
  ROI = (Manual execution time × Frequency) - Automation cost
  If ROI > 0 within 3 sprints → automate
```

---

## Q4: "How do you handle test data across environments?"
```
Strategies we use:
1. CREATE before test, DELETE after test (self-contained)
2. Unique data per run: "agent_" + timestamp (avoid conflicts)
3. No shared test data between tests (independence)
4. Environment-specific config files (different tenants per region)
5. Factory pattern for payload generation

Example:
  String uniqueName = "mcp-server-" + System.currentTimeMillis();
  // Create → Test → Delete
  // Even if test fails, @After hook cleans up
```

---

## Q5: "What is Shift-Left Testing and how do you practice it?"
```
Shift-Left = Test EARLIER in the development lifecycle.

Traditional: Dev → Code Complete → QA Tests → Bugs Found Late 😢
Shift-Left:  Design Review → API Contract Review → Test During Dev → Early Bugs 😊

How you practice it:
1. Review API specs (Swagger/OpenAPI) before dev starts
2. Write Cucumber scenarios during sprint planning
3. Create test data requirements early
4. Participate in design discussions
5. Start automation as soon as first endpoint is deployed (not after all are done)
```

---

# PART 7: SOFT SKILLS & COMMUNICATION GAPS

---

## How to Answer "Do You Have Any Questions?" (BETTER VERSION)
```
DON'T ask generic questions. Ask SMART questions that show you've researched:

For a startup:
  "What's your current test automation coverage, and what's the target for this year?"

For a product company:
  "How does the QA team collaborate with developers during the sprint? 
   Is there a dedicated QA review in the PR process?"

For an AI/ML company:
  "How do you handle the non-deterministic nature of AI outputs in your test strategy?
   Do you use snapshot testing or threshold-based validation?"

For any company:
  "What does a typical first 90 days look like for someone in this role?"
  "What's the biggest quality challenge the team is facing right now?"
```

---

## How to Handle Salary Negotiation
```
For 2.5 years SDET with your skills (AI/ML testing, multi-region, REST Assured):

Market range (India, April 2026):
  Service companies: 8-12 LPA
  Product companies: 12-20 LPA
  Top-tier (FAANG-level): 18-30 LPA

Your differentiators (justify higher range):
  • AI/ML testing (rare skill)
  • Multi-region deployment testing
  • ISTQB certified
  • 320+ LeetCode
  • Agentic AI + RAG (cutting-edge)

Script:
  "Based on my experience in AI/ML service testing, multi-region automation,
   and ISTQB certification, I'm targeting [X-Y] LPA. However, I'm flexible
   based on the overall opportunity, learning path, and growth potential.
   What's the budget range for this role?"

RULE: Always let them state a number first if possible.
```

---

# PART 8: THINGS MOST CANDIDATES FORGET

---

## ✅ Pre-Interview Checklist
```
□ Research the company (products, tech stack, recent news)
□ Read the job description 3 times — match YOUR experience to THEIR requirements
□ Prepare company-specific "Why this company?" answer
□ Test your internet connection, camera, microphone
□ Have a glass of water nearby
□ Keep resume printed/open
□ Keep a notepad for notes
□ Dress professionally (even for video calls)
□ Join 5 minutes early
□ Smile when you greet
```

## ✅ During Interview
```
□ Listen FULLY before answering (don't interrupt)
□ Take 3-5 seconds to think — silence is okay
□ Use STAR format for behavioral questions
□ Quantify achievements ("50+ endpoints", "3 regions", "40% reduction")
□ If you don't know: "I haven't worked with that directly, but based on 
   my experience with [similar thing], I would approach it by..."
□ Ask clarifying questions for vague problems
□ Think out loud during coding — show your process
□ Don't badmouth current/previous company
□ End with "Do you have any questions for me?"
```

## ✅ After Interview
```
□ Send a thank-you email within 24 hours
□ Note down questions they asked (for future prep)
□ If rejected, ask for feedback
□ Keep applying — don't wait for one result
```

---

# PART 9: TECHNOLOGY TRENDS TO MENTION (Impress Interviewers)

---

```
Show that you're aware of industry trends:

1. "I've been following the MCP standard by Anthropic — it's becoming the 
    universal protocol for AI tool integration, and I have hands-on testing 
    experience with it."

2. "I'm seeing a shift towards AI-powered test generation — using LLMs to 
    generate test cases from API specs. I'm exploring how to validate the 
    quality of AI-generated tests."

3. "Playwright is rapidly replacing Selenium for new projects due to 
    auto-waiting, multi-browser support, and built-in API testing."

4. "Shift-left testing and contract testing are becoming essential in 
    microservices architectures."

5. "Observability (logs + metrics + traces) is complementing traditional 
    testing in production monitoring."
```

---

*These are the gaps that could cost you an offer. Study them before your interview!*

