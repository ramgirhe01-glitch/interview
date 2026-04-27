# 📘 Technical Interview Guide — SDET / QA Automation
### Ram Girhe | REST Assured, Cucumber BDD, Selenium, Playwright, CI/CD, AI/ML Testing

---

# SECTION A: REST Assured — Deep Dive

---

## Q1: How do you structure a REST Assured test?
```java
@Test
public void testGetUser() {
    given()
        .baseUri("https://api.example.com")
        .header("Authorization", "Bearer " + token)
        .queryParam("region", "US")
        .log().all()  // Log request for debugging
    .when()
        .get("/users/123")
    .then()
        .log().ifError()  // Log response only on failure
        .statusCode(200)
        .body("name", equalTo("Ram"))
        .body("roles", hasItem("admin"))
        .body("email", containsString("@"))
        .body("createdAt", notNullValue())
        .time(lessThan(2000L)); // Response time SLA
}
```

### Key REST Assured Methods:
| Method | Purpose | Example |
|--------|---------|---------|
| `given()` | Setup request (headers, params, body) | `.header("Auth", "Bearer x")` |
| `when()` | Send HTTP request | `.get("/users")` |
| `then()` | Validate response | `.statusCode(200)` |
| `extract()` | Extract data from response | `.extract().path("id")` |
| `body()` | Validate JSON body | `.body("name", equalTo("Ram"))` |
| `log()` | Log request/response | `.log().all()` |

---

## Q2: How do you handle OAuth2 in REST Assured?
```java
public class AuthHelper {
    private static String cachedToken;
    private static long tokenExpiry;

    public static String getOAuth2Token() {
        // Return cached token if still valid
        if (cachedToken != null && System.currentTimeMillis() < tokenExpiry) {
            return cachedToken;
        }
        
        cachedToken = given()
            .contentType("application/x-www-form-urlencoded")
            .formParam("grant_type", "client_credentials")
            .formParam("client_id", ConfigReader.get("client.id"))
            .formParam("client_secret", ConfigReader.get("client.secret"))
            .formParam("scope", "api.read api.write")
        .when()
            .post(ConfigReader.get("auth.url") + "/oauth/token")
        .then()
            .statusCode(200)
            .extract().path("access_token");
        
        // Cache for 55 minutes (token valid for 60 min)
        tokenExpiry = System.currentTimeMillis() + 55 * 60 * 1000;
        return cachedToken;
    }
}
```

### OAuth2 Grant Types You Should Know:
| Grant Type | Use Case | Who Uses It |
|-----------|----------|-------------|
| `client_credentials` | Service-to-service (no user) | Backend APIs, microservices |
| `authorization_code` | User login via browser | Web apps |
| `password` | Direct user login (deprecated) | Legacy apps |
| `refresh_token` | Renew expired token | Mobile/web apps |

---

## Q3: How do you validate JSON Schema?
```java
// Add dependency: io.rest-assured:json-schema-validator
.then()
    .assertThat()
    .body(matchesJsonSchemaInClasspath("schemas/user-schema.json"));
```

Example schema file (`user-schema.json`):
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["id", "name", "email"],
  "properties": {
    "id": { "type": "integer" },
    "name": { "type": "string", "minLength": 1 },
    "email": { "type": "string", "format": "email" },
    "roles": {
      "type": "array",
      "items": { "type": "string" }
    }
  }
}
```

---

## Q4: How do you do multi-region testing?

**Architecture:**
```
config/
  ├── us.properties    → baseUrl=https://us-api.siemens.com
  ├── eu.properties    → baseUrl=https://eu-api.siemens.com
  └── ap.properties    → baseUrl=https://ap-api.siemens.com
```

**ConfigReader pattern:**
```java
public class ConfigReader {
    private static Properties properties;
    
    static {
        String region = System.getProperty("region", "US");
        properties = new Properties();
        try (InputStream is = ConfigReader.class.getResourceAsStream(
                "/config/" + region.toLowerCase() + ".properties")) {
            properties.load(is);
        } catch (IOException e) { throw new RuntimeException(e); }
    }
    
    public static String get(String key) {
        return properties.getProperty(key);
    }
}
```

**What to validate across regions:**
- Same API returns consistent structure across US/EU/AP
- Data isolation: US tenant cannot access EU data
- Region-specific OAuth scopes work correctly
- Latency meets SLA per region
- Deployment version is consistent

---

## Q5: Explain your FIT testing approach.
```
FIT = Functional Integration Testing

Purpose: Validate that a new deployment didn't break core functionality.
Runs: After every deployment, before promoting to production.

FIT Test Checklist:
1. ✅ Auth token generation works
2. ✅ Core CRUD (Create, Read, Update, Delete) succeed
3. ✅ Cross-service calls work (Agent → LLM → Response)
4. ✅ Region-specific configs loaded correctly
5. ✅ Database connectivity verified
6. ✅ External service integrations alive (Bedrock, etc.)

FIT vs Smoke vs Regression:
| Type | Scope | When | Duration |
|------|-------|------|----------|
| Smoke | Critical paths only | After build | 5-10 min |
| FIT | Integration points | After deployment | 15-30 min |
| Regression | Full test suite | Nightly/weekly | 1-3 hours |
```

---

## Q6: How do you handle request/response logging?
```java
// Global logging configuration
RestAssured.filters(new RequestLoggingFilter(), new ResponseLoggingFilter());

// Or selective logging
given()
    .log().ifValidationFails()  // Only log when test fails
.when()
    .get("/users")
.then()
    .log().ifError()
    .statusCode(200);
```

---

## Q7: How do you do data-driven testing with REST Assured?
```java
@DataProvider(name = "userData")
public Object[][] userData() {
    return new Object[][] {
        {"admin", "admin@test.com", 201},
        {"", "test@test.com", 400},        // Empty name
        {"user", "invalid-email", 400},     // Bad email
        {"user", "", 400},                  // Empty email
    };
}

@Test(dataProvider = "userData")
public void testCreateUser(String name, String email, int expectedStatus) {
    String payload = String.format("{\"name\":\"%s\",\"email\":\"%s\"}", name, email);
    given()
        .contentType(ContentType.JSON)
        .body(payload)
    .when()
        .post("/users")
    .then()
        .statusCode(expectedStatus);
}
```

---

## Q8: How do you chain API calls (e.g., create then verify)?
```java
@Test
public void testCreateAndVerifyUser() {
    // Step 1: Create
    String userId = given()
        .contentType(ContentType.JSON)
        .body("{\"name\":\"Ram\",\"email\":\"ram@test.com\"}")
    .when()
        .post("/users")
    .then()
        .statusCode(201)
        .extract().path("id");

    // Step 2: Verify
    given()
    .when()
        .get("/users/" + userId)
    .then()
        .statusCode(200)
        .body("name", equalTo("Ram"));
    
    // Step 3: Cleanup
    given().when().delete("/users/" + userId).then().statusCode(204);
}
```

---

## Q9: Explain RequestSpecification and ResponseSpecification.
```java
// Reusable request spec
RequestSpecification requestSpec = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .addHeader("Authorization", "Bearer " + token)
    .setContentType(ContentType.JSON)
    .addFilter(new AllureRestAssured()) // Allure reporting
    .build();

// Reusable response spec
ResponseSpecification responseSpec = new ResponseSpecBuilder()
    .expectStatusCode(200)
    .expectContentType(ContentType.JSON)
    .expectResponseTime(lessThan(3000L))
    .build();

// Usage
given()
    .spec(requestSpec)
    .body(payload)
.when()
    .post("/users")
.then()
    .spec(responseSpec)
    .body("id", notNullValue());
```

---

# SECTION B: Cucumber BDD — Deep Dive

---

## Q10: Write a sample feature file for Agentic AI.
```gherkin
@AgenticAI @Smoke
Feature: MCP Server Registration

  Background:
    Given I have a valid OAuth2 token with scope "agentic.admin"

  Scenario: Register a Standard MCP Server
    Given I have a valid MCP server payload with type "STANDARD"
    When I send a POST request to "/api/v1/mcp-servers"
    Then the response status code should be 201
    And the response body should contain "serverId"
    And the server type should be "STANDARD"

  Scenario Outline: Register MCP Server with different types
    When I register an MCP server with type "<type>"
    Then the response status code should be <status>

    Examples:
      | type                   | status |
      | STANDARD               | 201    |
      | DESKTOP_TUNNEL         | 201    |
      | PRIVATE_NETWORK_TUNNEL | 201    |
      | INVALID                | 400    |
```

---

## Q11: How do you write step definitions?
```java
public class AgenticAISteps {
    private Response response;
    private String token;
    private String payload;

    @Given("I have a valid OAuth2 token with scope {string}")
    public void getTokenWithScope(String scope) {
        token = AuthHelper.getToken(scope);
        assertNotNull(token, "Token should not be null");
    }

    @Given("I have a valid MCP server payload with type {string}")
    public void createPayload(String type) {
        payload = PayloadBuilder.mcpServer(type);
    }

    @When("I send a POST request to {string}")
    public void sendPostRequest(String endpoint) {
        response = given()
            .header("Authorization", "Bearer " + token)
            .contentType(ContentType.JSON)
            .body(payload)
        .when()
            .post(ConfigReader.get("baseUrl") + endpoint);
    }

    @Then("the response status code should be {int}")
    public void verifyStatusCode(int expectedStatus) {
        assertEquals(expectedStatus, response.getStatusCode());
    }

    @Then("the response body should contain {string}")
    public void verifyBodyContains(String field) {
        assertNotNull(response.jsonPath().getString(field));
    }
}
```

---

## Q12: Cucumber Hooks — Setup and Teardown
```java
public class Hooks {
    @Before
    public void setUp(Scenario scenario) {
        System.out.println("Starting: " + scenario.getName());
        // Setup test data, token, etc.
    }

    @After
    public void tearDown(Scenario scenario) {
        if (scenario.isFailed()) {
            // Attach logs or screenshots
            scenario.attach(getResponseLog(), "text/plain", "API Response");
        }
        // Cleanup test data
    }

    @Before("@NeedsAuth")
    public void setupAuth() {
        // Only runs for scenarios tagged @NeedsAuth
        token = AuthHelper.getToken("default");
    }
}
```

---

## Q13: Key Cucumber Concepts
| Concept | Description | Example |
|---------|-------------|---------|
| Feature | Test file (.feature) | `Feature: User Management` |
| Scenario | Single test case | `Scenario: Create user` |
| Scenario Outline | Data-driven test | Uses `Examples:` table |
| Background | Runs before each scenario | Common setup steps |
| Tags | Categorize scenarios | `@Smoke`, `@Regression` |
| Hooks | Before/After execution | `@Before`, `@After` |
| Data Tables | Pass tabular data | Step with table below it |
| Doc Strings | Pass multi-line text | Triple quotes in step |

---

# SECTION C: Selenium / Playwright

---

## Q14: Explain Page Object Model.
```java
public class LoginPage {
    private WebDriver driver;

    @FindBy(id = "username") private WebElement usernameField;
    @FindBy(id = "password") private WebElement passwordField;
    @FindBy(id = "loginBtn") private WebElement loginButton;
    @FindBy(css = ".error-message") private WebElement errorMessage;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public DashboardPage login(String user, String pass) {
        usernameField.clear();
        usernameField.sendKeys(user);
        passwordField.clear();
        passwordField.sendKeys(pass);
        loginButton.click();
        return new DashboardPage(driver);
    }

    public String getErrorMessage() {
        return errorMessage.getText();
    }
}
```

**Why POM?**
- Separates test logic from page structure
- Easy maintenance — change locator in ONE place
- Readable tests: `loginPage.login("Ram", "pass123")`
- Reusable across multiple tests

---

## Q15: Selenium Waits — Implicit vs Explicit vs Fluent
```java
// Implicit Wait (applies globally — NOT recommended)
driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));

// Explicit Wait (targeted — RECOMMENDED)
WebDriverWait wait = new WebDriverWait(driver, Duration.ofSeconds(10));
WebElement element = wait.until(
    ExpectedConditions.visibilityOfElementLocated(By.id("result"))
);

// Fluent Wait (most control)
Wait<WebDriver> fluentWait = new FluentWait<>(driver)
    .withTimeout(Duration.ofSeconds(30))
    .pollingEvery(Duration.ofSeconds(2))
    .ignoring(NoSuchElementException.class);
WebElement el = fluentWait.until(d -> d.findElement(By.id("dynamic-content")));
```

**Rule: NEVER use Thread.sleep() in production test code.**

---

## Q16: Selenium vs Playwright — Key Differences
| Feature | Selenium | Playwright |
|---------|----------|------------|
| Language Support | Java, Python, C#, JS, Ruby | JS/TS, Python, Java, C# |
| Speed | Slower (HTTP/W3C protocol) | Faster (CDP/WebSocket) |
| Auto-wait | No (need explicit waits) | Yes (built-in smart waits) |
| Multi-tab/browser | Complex setup | Native support |
| Network Interception | Limited (proxy-based) | Built-in `route()` API |
| Parallel Execution | Via Selenium Grid | Native browser contexts |
| iFrame handling | switchTo().frame() | Built-in frame locators |
| File download | Complex | Built-in |
| Headless | Supported | Default mode |
| Community | Huge (20+ years) | Growing fast |
| Mobile testing | Via Appium | Experimental |

---

## Q17: Common Selenium Interview Questions

**Q: How do you handle dynamic elements?**
```java
// Use CSS/XPath with partial match
driver.findElement(By.cssSelector("[id*='user']"));      // contains
driver.findElement(By.cssSelector("[id^='user']"));      // starts with
driver.findElement(By.xpath("//div[contains(@class, 'active')]"));
```

**Q: How do you handle alerts?**
```java
Alert alert = driver.switchTo().alert();
alert.accept();    // OK
alert.dismiss();   // Cancel
alert.getText();   // Read message
alert.sendKeys("input"); // Prompt
```

**Q: How do you handle multiple windows?**
```java
String mainWindow = driver.getWindowHandle();
Set<String> allWindows = driver.getWindowHandles();
for (String window : allWindows) {
    if (!window.equals(mainWindow)) {
        driver.switchTo().window(window);
        // do stuff
        driver.close();
    }
}
driver.switchTo().window(mainWindow);
```

---

# SECTION D: CI/CD & DevOps Questions

---

## Q18: Describe your CI/CD pipeline.
```
Pipeline Architecture:
┌──────────┐   ┌──────────┐   ┌──────────────┐   ┌──────────┐   ┌──────────┐
│  Trigger  │ → │  Build   │ → │  Test Matrix │ → │  Report  │ → │  Notify  │
│ (PR/Push) │   │ (Gradle) │   │  US/EU/AP    │   │ (HTML)   │   │ (Slack)  │
└──────────┘   └──────────┘   └──────────────┘   └──────────┘   └──────────┘

Detailed Steps:
1. Developer pushes code → triggers pipeline
2. Build stage: `./gradlew compileTestJava` — compile test code
3. Test stage: Run with tags `./gradlew test -Dtags="@Smoke"`
4. Region matrix: Same tests run for US, EU, AP (parallel)
5. Retry stage: Failed tests rerun from rerun.txt
6. Report stage: Cucumber HTML report + trend analysis
7. Notification: Results pushed to JIRA/Slack/Email
```

---

## Q19: How do you handle flaky tests?
```
Root Cause Analysis Framework:
1. Is it timing? → Add explicit waits, not Thread.sleep
2. Is it data? → Create fresh data per test, don't share
3. Is it environment? → Compare across regions
4. Is it concurrency? → Run in isolation to confirm
5. Is it network? → Add retry with backoff

Prevention:
- Cucumber rerun plugin (rerun.txt) for auto-retry
- Tag flaky tests with @Flaky for tracking
- Weekly flaky test review meeting
- Monitor flaky rate trend over time
- Set threshold: >5% flaky rate = team action required
```

---

## Q20: Git Commands Every SDET Should Know
```bash
git clone <url>                    # Clone repo
git checkout -b feature/my-test   # Create branch
git add .                         # Stage changes
git commit -m "Add smoke tests"   # Commit
git push origin feature/my-test   # Push
git pull origin main              # Update from main
git stash                         # Temporarily save changes
git log --oneline -10             # View recent commits
git diff                          # See uncommitted changes
git rebase main                   # Rebase on main
git cherry-pick <commit-hash>     # Pick specific commit
```

---

# SECTION E: AI/ML Testing (Your Differentiator)

---

## Q21: How do you test an Agentic AI service?
```
Testing Layers:
┌─────────────────────────────────────────────┐
│ Layer 1: MCP Server Management              │
│ - CRUD for Standard, Desktop, Private Tunnel│
│ - Validate server registration/deregistration│
├─────────────────────────────────────────────┤
│ Layer 2: Agent Lifecycle                     │
│ - Create, configure, update, delete agents  │
│ - Assign tools, LLMs, guardrails            │
├─────────────────────────────────────────────┤
│ Layer 3: Execution                          │
│ - Sync execution (request → response)       │
│ - Async execution (submit → poll → result)  │
│ - SSE streaming (real-time events)          │
├─────────────────────────────────────────────┤
│ Layer 4: Security & Isolation               │
│ - 3-level JWT (Provider→App→Customer)       │
│ - Tenant isolation validation               │
│ - Guardrail enforcement                     │
├─────────────────────────────────────────────┤
│ Layer 5: Cross-Region Consistency           │
│ - US, EU, AP behave identically             │
│ - Config propagation verified               │
└─────────────────────────────────────────────┘
```

---

## Q22: How do you test SSE (Server-Sent Events)?
```java
Response response = given()
    .header("Accept", "text/event-stream")
    .header("Authorization", "Bearer " + token)
.when()
    .get("/api/v1/agents/execute-stream");

// Validate SSE format
String body = response.getBody().asString();
String[] events = body.split("\n\n");

// Assertions
assertTrue(events.length > 0, "Should have at least one event");
assertTrue(events[0].startsWith("event:"), "Each event should have type");
assertTrue(events[events.length - 1].contains("event: done"), "Last event should be 'done'");

// Validate event ordering
boolean foundStart = false, foundEnd = false;
for (String event : events) {
    if (event.contains("event: start")) foundStart = true;
    if (event.contains("event: done")) foundEnd = true;
}
assertTrue(foundStart && foundEnd, "Must have start and done events");
```

---

## Q23: What is RAG and how do you test it?
```
RAG = Retrieval-Augmented Generation

How it works:
1. Documents are converted to vector embeddings
2. Stored in vector database (Knowledge Bank)
3. User sends a query
4. System finds semantically similar documents (not keyword match!)
5. Retrieved context + user query → sent to LLM
6. LLM generates answer using the context

Testing approach:
┌──────────────────────────────────┐
│ 1. Ingestion Testing             │
│ - Ingest known documents         │
│ - Bulk ingestion (100/request)   │
│ - Duplicate handling             │
│ - Invalid format rejection       │
├──────────────────────────────────┤
│ 2. Search/Retrieval Testing      │
│ - Semantic search accuracy       │
│ - topK relevance tuning          │
│ - Metadata filtering             │
│ - Cursor-based pagination        │
│ - 1-second availability SLA      │
├──────────────────────────────────┤
│ 3. Model Management              │
│ - Embedding model hot-swap       │
│ - Async re-indexing after swap   │
│ - Model version consistency      │
├──────────────────────────────────┤
│ 4. Integration Testing           │
│ - RAG + LLM end-to-end          │
│ - Context window limits          │
│ - Fallback when KB is empty      │
└──────────────────────────────────┘
```

---

## Q24: HTTP Status Codes Every Tester Must Know
| Code | Meaning | When You'll See It |
|------|---------|-------------------|
| 200 | OK | Successful GET/PUT/PATCH |
| 201 | Created | Successful POST (resource created) |
| 204 | No Content | Successful DELETE |
| 301 | Moved Permanently | URL redirect |
| 400 | Bad Request | Invalid input / missing required field |
| 401 | Unauthorized | Missing or invalid token |
| 403 | Forbidden | Valid token but no permission |
| 404 | Not Found | Resource doesn't exist |
| 405 | Method Not Allowed | Wrong HTTP method |
| 409 | Conflict | Duplicate resource |
| 413 | Payload Too Large | File too big |
| 415 | Unsupported Media Type | Wrong Content-Type |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Server Error | Server crashed (BUG!) |
| 502 | Bad Gateway | Upstream service down |
| 503 | Service Unavailable | Server overloaded |

---

## Q25: PUT vs PATCH vs POST
| Method | Purpose | Idempotent? | Example |
|--------|---------|-------------|---------|
| POST | Create new resource | No | `POST /users` with body |
| PUT | Replace entire resource | Yes | `PUT /users/123` with full body |
| PATCH | Update partial resource | Yes | `PATCH /users/123` with partial body |
| DELETE | Remove resource | Yes | `DELETE /users/123` |

**Idempotent** = calling it multiple times has the same result as calling it once.

---

*Master these concepts and you'll ace any SDET technical interview!*

