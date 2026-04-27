# Cucumber Step Data Flow - Complete Guide

## Table of Contents
1. [Data Flow Architecture Overview](#1-data-flow-architecture-overview)
2. [ScenarioContext - Central Data Hub](#2-scenariocontext---central-data-hub)
3. [Data Flow Between Steps](#3-data-flow-between-steps)
4. [Feature File Data Sources](#4-feature-file-data-sources)
5. [API Step Data Flow](#5-api-step-data-flow)
6. [UI Step Data Flow](#6-ui-step-data-flow)
7. [Variable Substitution Flow](#7-variable-substitution-flow)
8. [End-to-End Data Flow Examples](#8-end-to-end-data-flow-examples)
9. [Config & Environment Data Flow](#9-config--environment-data-flow)
10. [Data Flow Diagrams](#10-data-flow-diagrams)

---

## 1. Data Flow Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CUCUMBER DATA FLOW ARCHITECTURE                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────────────┐  │
│  │ .feature     │───>│ Step         │───>│ ScenarioContext      │  │
│  │ Files        │    │ Definitions  │    │ (Shared State)       │  │
│  │              │    │              │    │                      │  │
│  │ - DataTable  │    │ - @Given     │    │ - set(key, value)    │  │
│  │ - DocString  │    │ - @When      │    │ - get(key)           │  │
│  │ - Examples   │    │ - @Then      │    │ - setResponse(resp)  │  │
│  │ - Parameters │    │ - @And       │    │ - getResponse()      │  │
│  │ - Variables  │    │              │    │ - getData()           │  │
│  └──────────────┘    └──────────────┘    │ - delete(key)        │  │
│                                          └──────────────────────┘  │
│                              │                     │               │
│                              ▼                     ▼               │
│                    ┌──────────────┐    ┌──────────────────────┐    │
│                    │ Page Objects │    │ Config / Properties  │    │
│                    │ / API Helper │    │ / JSON Test Data     │    │
│                    └──────────────┘    └──────────────────────┘    │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 2. ScenarioContext - Central Data Hub

The `ScenarioContext` is the **single source of truth** for passing data between steps within a scenario.

### How It Works

```java
// ScenarioContext is injected via constructor (PicoContainer DI)
public class MyStepDefinitions extends XFApiStepDefinitionBase {

    public MyStepDefinitions(ScenarioContext scenarioContext) {
        super(scenarioContext);  // Passes context to base class
    }
}
```

### Core Operations

| Operation | Method | Description |
|-----------|--------|-------------|
| **Store** | `scenarioContext.set("key", "value")` | Save any data |
| **Retrieve** | `scenarioContext.get("key")` | Get stored value |
| **Store Response** | `scenarioContext.setResponse(response)` | Save API response |
| **Get Response** | `scenarioContext.getResponse()` | Get last API response |
| **Get All Data** | `scenarioContext.getData()` | Get entire data map |
| **Delete** | `scenarioContext.delete("key")` | Remove a stored key |

### Data Lifecycle

```
Scenario Start
    │
    ▼
ScenarioContext created (empty)
    │
    ▼
@Before Hook → Sets platform, region, env, environment
    │
    ▼
Step 1 (Given) → scenarioContext.set("userId", "12345")
    │
    ▼
Step 2 (When)  → String id = scenarioContext.get("userId")  // "12345"
    │                scenarioContext.setResponse(apiResponse)
    │
    ▼
Step 3 (Then)  → Response resp = scenarioContext.getResponse()
    │                // Validate response
    │
    ▼
@After Hook → Cleanup
    │
    ▼
ScenarioContext destroyed
```

---

## 3. Data Flow Between Steps

### 3.1 Direct Parameter Passing (Feature → Step)

```gherkin
# String parameters
Given "casxfuser1" user logged in into "devconsole" service

# Integer parameters
Then the response status code should be 200

# DataTable parameters
And I select application settings details as below[UI]
  | Type            | FDSGatewayEntryPoint |
  | Web application | false                |
```

**Step Definition receives it:**

```java
@Given("{string} user logged in into {string} service")
public void userLoggedIn(String userKey, String service) {
    // userKey = "casxfuser1", service = "devconsole"
}

@And("I select application settings details as below[UI]")
public void selectAppSettings(DataTable dataTable) {
    List<Map<String, String>> data = dataTable.asMaps();
    String type = data.get(0).get("Type");  // "Web application"
}
```

### 3.2 Step-to-Step via ScenarioContext

```gherkin
# Step 1: Generate and store
Given I generate a random "numeric" type variable "apiid" of length 6

# Step 2: Use stored value via ${variable}
Given I set variable "appName" value as "xt123567${apiid}"

# Step 3: Use in DataTable
And I fill general information details as below[UI]
  | DisplayName | InternalName |
  | ${appName}  | ${appName}   |
```

**Behind the scenes:**

```java
// Step 1
@Given("I generate a random {string} type variable {string} of length {int}")
public void generateRandom(String type, String varName, int length) {
    String value = RandomStringUtils.randomNumeric(length);  // e.g., "483920"
    scenarioContext.set(varName, value);  // set("apiid", "483920")
}

// Step 2
@Given("I set variable {string} value as {string}")
public void setVariable(String varName, String rawValue) {
    // rawValue = "xt123567${apiid}"
    String resolved = stepUtil.updateVariableByValue(rawValue, scenarioContext.getData());
    // resolved = "xt123567483920"
    scenarioContext.set(varName, resolved);  // set("appName", "xt123567483920")
}

// Step 3 - DataTable values with ${} are auto-resolved using scenarioContext
```

### 3.3 API Response → Next Step

```gherkin
When I call the API to create a user
Then I store response field "data.id" into variable "createdUserId"
And I call the API to get user with id "${createdUserId}"
```

```java
// Store response
@When("I call the API to create a user")
public void callCreateUser() {
    Response response = apiHelper.post("/users", body);
    scenarioContext.setResponse(response);  // Store entire response
}

// Extract from response and store in context
@Then("I store response field {string} into variable {string}")
public void storeResponseField(String jsonPath, String variable) {
    String value = scenarioContext.getResponse().jsonPath().getString(jsonPath);
    scenarioContext.set(variable, value);  // set("createdUserId", "42")
}

// Use in next API call - ${createdUserId} gets resolved
@And("I call the API to get user with id {string}")
public void getUser(String userId) {
    // userId input = "${createdUserId}" → resolved to "42"
    String resolved = stepUtil.updateVariableByValue(userId, scenarioContext.getData());
    Response response = apiHelper.get("/users/" + resolved);
    scenarioContext.setResponse(response);
}
```

---

## 4. Feature File Data Sources

### 4.1 Inline Parameters

```gherkin
Given the user "john" with role "admin"
When timeout is set to 30 seconds
```

### 4.2 DataTable (Single Row = Key-Value Map)

```gherkin
And I fill general information details as below[UI]
  | DisplayName | InternalName | Version | Description          |
  | ${appName}  | ${appName}   |         | Test api Application |
```

**Usage in Java:**

```java
public void fillInfo(DataTable table) {
    Map<String, String> data = table.asMaps().get(0);
    String displayName = data.get("DisplayName");  // resolved "${appName}"
    String version = data.get("Version");           // "" (empty)
}
```

### 4.3 DataTable (Multi Row = List of Maps)

```gherkin
When I fill components details as below[UI]
  | Name       | URL          | SkipAuthorizationHeader |
  | Component1 | multicomZero | true                    |
  | Component2 | multicomOne  | false                   |
  | Component3 | multicomTwo  | true                    |
```

**Usage in Java:**

```java
public void fillComponents(DataTable table) {
    List<Map<String, String>> rows = table.asMaps();
    for (Map<String, String> row : rows) {
        String name = row.get("Name");
        String url = row.get("URL");
        boolean skipAuth = Boolean.parseBoolean(row.get("SkipAuthorizationHeader"));
    }
}
```

### 4.4 DocString (JSON/XML Body)

```gherkin
When I send a POST request to "/api/users" with body:
  """
  {
    "name": "John Doe",
    "email": "john@example.com"
  }
  """
```

**Usage in Java:**

```java
@When("I send a POST request to {string} with body:")
public void postWithBody(String endpoint, String docString) {
    // docString = the entire JSON block
    Response response = apiHelper.post(endpoint, docString);
    scenarioContext.setResponse(response);
}
```

### 4.5 Scenario Outline + Examples

```gherkin
Scenario Outline: Login with different users
  Given "<userKey>" user logged in into "<service>" service
  When I perform action "<action>"
  Then the response should contain "<expected>"

  Examples:
    | userKey    | service    | action | expected |
    | casxfuser1 | devconsole | create | success  |
    | casxfuser2 | iam        | read   | data     |
```

### 4.6 External JSON File Data

```gherkin
And I call API using request file "api/iam/create-user.json" with request path "request"
```

**JSON file (`api/iam/create-user.json`):**

```json
{
  "request": {
    "method": "POST",
    "path": "/api/v3/Users",
    "headers": {
      "Content-Type": "application/json",
      "Authorization": "Bearer ${accessToken}"
    },
    "body": {
      "name": "${userName}",
      "email": "${userEmail}"
    }
  }
}
```

**Java resolves `${}` variables from ScenarioContext before calling API.**

---

## 5. API Step Data Flow

### Complete API Test Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                      API STEP DATA FLOW                          │
│                                                                  │
│  Feature File                                                    │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ Given I get auth token for "casxfuser1"              │       │
│  │ When I call API "api/iam/get-users.json" "request"   │       │
│  │ Then response status is 200                          │       │
│  │ And I store "data[0].id" into "userId"               │       │
│  │ When I call API "api/iam/get-user.json" "request"    │       │
│  └───────────┬──────────────────────────────────────────┘       │
│              │                                                   │
│              ▼                                                   │
│  Step Definition                                                 │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ 1. Get token → scenarioContext.set("accessToken",t)  │       │
│  │ 2. Read JSON file → resolve ${accessToken}           │       │
│  │ 3. Build RequestSpec → Execute API                   │       │
│  │ 4. scenarioContext.setResponse(response)             │       │
│  │ 5. Extract jsonPath → scenarioContext.set("userId")  │       │
│  │ 6. Read next JSON → resolve ${userId}                │       │
│  │ 7. Execute next API call                             │       │
│  └───────────┬──────────────────────────────────────────┘       │
│              │                                                   │
│              ▼                                                   │
│  ScenarioContext (State)                                         │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ accessToken  = "Bearer eyJhbGciOi..."                │       │
│  │ userId       = "42"                                  │       │
│  │ response     = { status:200, body:{...} }            │       │
│  │ platform     = "us1_int"                             │       │
│  │ region       = "US"                                  │       │
│  │ env          = "INT"                                 │       │
│  └──────────────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────────────┘
```

### API Data Flow Code Pattern

```java
// === STEP 1: Authentication ===
@Given("I get auth token for {string}")
public void getAuthToken(String userKey) {
    // Load user credentials from config
    String clientId = AppConfig.getInstance().getEnvPropertyValue(userKey + ".clientId");
    String clientSecret = AppConfig.getInstance().getEnvPropertyValue(userKey + ".clientSecret");
    
    // Call auth endpoint
    Response tokenResponse = given()
        .formParam("grant_type", "client_credentials")
        .formParam("client_id", clientId)
        .formParam("client_secret", clientSecret)
        .post("/oauth/token");
    
    // Store token in context for subsequent steps
    String token = tokenResponse.jsonPath().getString("access_token");
    scenarioContext.set("accessToken", token);
}

// === STEP 2: Call API using JSON file ===
@When("I call API using file {string} with path {string}")
public void callApiUsingFile(String filePath, String jsonPath, String service) {
    // Read JSON request template
    String requestJson = JsonUtil.readJsonFile(filePath);
    
    // Resolve all ${variables} from ScenarioContext
    // e.g., ${accessToken} → actual token value
    String resolved = stepUtil.updateVariableByValue(requestJson, scenarioContext.getData());
    
    // Build and execute request
    RequestBuilder request = new RequestBuilder();
    RequestSpecBuilder spec = request.buildRequest(resolved, scenarioContext);
    Response response = stepUtil.executeApiCall(request.getHttpMethod(), spec);
    
    // Store response for next steps
    scenarioContext.setResponse(response);
}

// === STEP 3: Validate Response ===
@Then("the response status code should be {int}")
public void verifyStatusCode(int expected) {
    int actual = scenarioContext.getResponse().getStatusCode();
    assertEquals(expected, actual);
}

// === STEP 4: Extract and Store from Response ===
@And("I store response jsonpath {string} into variable {string}")
public void storeFromResponse(String jsonPath, String variable) {
    String value = scenarioContext.getResponse().jsonPath().getString(jsonPath);
    scenarioContext.set(variable, value);
    // Now ${variable} is available for all subsequent steps
}
```

### SAM Auth Signature Flow

```java
// Special flow for SAM (S3-signed) API calls
public void callApiUsingFile(String requestFilePath, String requestJsonPath, String service) {
    String id = scenarioContext.get("samAccessKey");    // From prior auth step
    String key = scenarioContext.get("samSecretKey");   // From prior auth step
    Signature signature = new Signature(id, key);
    
    // Read & resolve request JSON
    String requestDetails = stepUtil.getUpdatedJsonValueFromFile(requestFilePath, requestJsonPath);
    JSONObject requestJson = new JSONObject(requestDetails);
    
    // Extract method, path, headers, body
    String method = requestJson.getString("method");
    String path = requestJson.getString("path");
    String body = requestJson.getJSONObject("body").toString();
    
    // Generate S3 signature
    String authorization = signature.getSignature(method, path, contentType, body);
    String token = "S3 " + authorization;
    
    // Store token & call API
    scenarioContext.set("accessToken", token);
    callServiceUsingRequestDetails(requestDetails, body);
    scenarioContext.delete("accessToken");  // Cleanup after call
}
```

---

## 6. UI Step Data Flow

### Complete UI Test Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                       UI STEP DATA FLOW                          │
│                                                                  │
│  Feature File                                                    │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ Given "casxfuser1" user logged in "devconsole" [UI]  │       │
│  │ And I navigate to "Create App Step By Step" [UI]     │       │
│  │ Given I generate random variable "apiid" length 6    │       │
│  │ Given I set variable "appName" as "xt${apiid}"       │       │
│  │ And I fill general info as below[UI]                 │       │
│  │   | DisplayName | InternalName |                     │       │
│  │   | ${appName}  | ${appName}   |                     │       │
│  │ And I capture screenshot [UI]                        │       │
│  └───────────┬──────────────────────────────────────────┘       │
│              │                                                   │
│              ▼                                                   │
│  Step Definition + Page Objects                                  │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ 1. Login → Browser opens, stores session in context  │       │
│  │ 2. Navigate → Page object clicks menu                │       │
│  │ 3. Generate → scenarioContext.set("apiid","483920")  │       │
│  │ 4. Set var → scenarioContext.set("appName","xt4839") │       │
│  │ 5. Fill form → Resolve ${appName} from context       │       │
│  │    → Page.enterDisplayName("xt483920")               │       │
│  │ 6. Screenshot → Attach to Cucumber report            │       │
│  └──────────────────────────────────────────────────────┘       │
│                                                                  │
│  Config Properties (data source)                                 │
│  ┌──────────────────────────────────────────────────────┐       │
│  │ casxfuser1.username = user@siemens.com               │       │
│  │ casxfuser1.password = encrypted_pass                 │       │
│  │ devconsole.url = https://devconsole.example.com      │       │
│  │ dcOneTenant2 = tenant-id-123                         │       │
│  └──────────────────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────────────────┘
```

### UI Data Flow Code Pattern

```java
// === Login Step: Config → Browser → Context ===
@Given("{string} user logged in into {string} service on {string} tenant in browser window:{string}[UI]")
public void loginUser(String userKey, String service, String tenant, String window) {
    // Data from config.properties
    String username = AppConfig.getInstance().getEnvPropertyValue(userKey + ".username");
    String password = AppConfig.getInstance().getEnvPropertyValue(userKey + ".password");
    String url = AppConfig.getInstance().getEnvPropertyValue(service + ".url");
    
    // Open browser & login
    DriverFactory.initDriver("chrome");
    LoginPage loginPage = new LoginPage();
    loginPage.navigateTo(url);
    loginPage.login(username, password);
    
    // Store in context for later steps
    scenarioContext.set("currentUser", userKey);
    scenarioContext.set("currentService", service);
    scenarioContext.set("currentTenant", tenant);
}

// === Form Fill: Context variables → UI ===
@And("I fill general information details as below[UI]")
public void fillGeneralInfo(DataTable dataTable) {
    Map<String, String> data = dataTable.asMaps().get(0);
    
    // Resolve ${variables} from ScenarioContext
    String displayName = resolveVariable(data.get("DisplayName"));  // "${appName}" → "xt483920"
    String internalName = resolveVariable(data.get("InternalName"));
    
    // Use Page Object to fill form
    GeneralInfoPage page = new GeneralInfoPage();
    page.enterDisplayName(displayName);
    page.enterInternalName(internalName);
}

private String resolveVariable(String value) {
    if (value != null && value.contains("${")) {
        return stepUtil.updateVariableByValue(value, scenarioContext.getData());
    }
    return value;
}
```

### Desktop App Data Flow

```java
// Desktop → Browser handoff via ScenarioContext
@And("I get access token from desktop app")
public void getAccessTokenFromDesktop() {
    // Desktop app generates PKCE codes
    String[] result = generatePKCE();
    scenarioContext.set("codeVerifier", result[0]);
    scenarioContext.set("codeChallenge", result[1]);
    
    // Desktop opens browser for auth
    String appCodeUrl = buildAuthUrl(scenarioContext.get("codeChallenge"));
    scenarioContext.set("appCodeUrl", appCodeUrl);
    
    // After browser auth, capture auth code
    String authCode = extractAuthCode();
    scenarioContext.set("authCode", authCode);
    
    // Exchange code for token
    String token = exchangeCodeForToken(
        scenarioContext.get("authCode"),
        scenarioContext.get("codeVerifier")
    );
    scenarioContext.set("accessToken", token);
}
```

---

## 7. Variable Substitution Flow

### How `${variable}` Resolution Works

```
Input String: "Hello ${firstName}, your ID is ${userId}"

ScenarioContext Data:
  firstName = "John"
  userId    = "42"

Resolution:
  stepUtil.updateVariableByValue(input, scenarioContext.getData())
  
Output: "Hello John, your ID is 42"
```

### Resolution Chain

```
┌─────────────────────┐
│ Feature File         │
│ ${appName}          │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ Step Definition      │   Checks if value contains "${"
│ resolveVariable()    │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ stepUtil             │   Iterates scenarioContext.getData()
│ .updateVariableBy   │   Replaces ${key} with value
│  Value()            │
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ ScenarioContext      │   getData() returns HashMap
│ { appName: "xt123" }│
└────────┬────────────┘
         │
         ▼
┌─────────────────────┐
│ Resolved Value       │
│ "xt123"             │
└─────────────────────┘
```

### Nested Variable Resolution

```gherkin
Given I set variable "env" value as "staging"
Given I set variable "baseUrl" value as "https://${env}.example.com"
# Result: baseUrl = "https://staging.example.com"

Given I set variable "fullUrl" value as "${baseUrl}/api/users"
# Result: fullUrl = "https://staging.example.com/api/users"
```

### Variable Sources (Priority Order)

| Priority | Source | Example |
|----------|--------|---------|
| 1 | System Property | `-DuserKey=admin` |
| 2 | ScenarioContext | `scenarioContext.set("userKey", "admin")` |
| 3 | Config Properties | `config.properties: userKey=admin` |
| 4 | Environment Properties | `staging.properties: userKey=admin` |
| 5 | Feature File Inline | `"admin"` (direct string) |

---

## 8. End-to-End Data Flow Examples

### Example 1: Full API CRUD Flow

```gherkin
@api @crud
Scenario: Create, Read, Update, Delete a user

  # STEP 1: Auth → token stored in context
  Given I authenticate as "casxfuser1" on "iam" service
  # Context: { accessToken: "eyJ..." }

  # STEP 2: Create → response stored, ID extracted
  When I call API using file "api/iam/create-user.json" with "request"
  Then the response status code should be 201
  And I store jsonpath "id" into variable "createdUserId"
  # Context: { accessToken: "eyJ...", createdUserId: "usr-42" }

  # STEP 3: Read → uses ${createdUserId} from context
  When I call API using file "api/iam/get-user.json" with "request"
  # JSON file has: "path": "/api/users/${createdUserId}"
  # Resolved to: "/api/users/usr-42"
  Then the response status code should be 200
  And response jsonpath "name" should be "John Doe"

  # STEP 4: Update
  When I call API using file "api/iam/update-user.json" with "request"
  Then the response status code should be 200

  # STEP 5: Delete
  When I call API using file "api/iam/delete-user.json" with "request"
  Then the response status code should be 204
```

### Example 2: UI + API Mixed Flow

```gherkin
@ui @api @e2e
Scenario: Create app via UI, verify via API

  # UI Steps - create app
  Given "casxfuser1" user logged in into "devconsole" on "dcOneTenant2" in browser:"first"[UI]
  Given I generate a random "numeric" type variable "apiid" of length 6
  Given I set variable "appName" value as "testapp${apiid}"
  # Context: { apiid: "483920", appName: "testapp483920" }
  
  And I navigate to "Create App Step By Step" menu[UI]
  And I fill general information details as below[UI]
    | DisplayName     | InternalName    |
    | ${appName}      | ${appName}      |
  # Page Object receives resolved "testapp483920"
  
  And I click on Submit button[UI]
  And I capture screenshot [UI]
  
  # API Steps - verify creation
  Given I authenticate as "casxfuser1" on "devconsole" service
  When I call GET "/api/apps?name=${appName}"
  # Resolved: "/api/apps?name=testapp483920"
  Then the response status code should be 200
  And response jsonpath "data[0].name" should be "${appName}"
```

### Example 3: Desktop + Browser + API Flow

```gherkin
@desktop @ui @api
Scenario: Desktop SDK authentication flow

  # Desktop generates PKCE and opens browser
  Given I launch Java SDK app with following data
    | Client-ID             | Scope | BrowserType | Port      | UserKey    |
    | deskappusint-tide0723 | 0     | 0           | 4065,4066 | casxfuser1 |
  # Context: { clientId: "deskapp...", port: "4065,4066", userKey: "casxfuser1" }

  And I get access token from desktop app
  # Context: + { codeVerifier, codeChallenge, appCodeUrl, authCode, accessToken }

  And I verify user logged in successfully[Desktop]
  # Uses: scenarioContext.get("accessToken") to verify

  And I create headless tech user
    | UserName | Description | NonceRequired |
    | dgfhdhg  | dshffsdhg   | true          |
  # API call using ${accessToken} from context
  # Context: + { techUserId, techUserSecret }

  And I perform forceful secret rotation new secret "rolF2Fy..."
  # Uses: ${techUserId} from context

  And I logout from desktop app
  And I verify user logged out successfully[Desktop]
  Then I clear all cookies for current browser [UI]
```

---

## 9. Config & Environment Data Flow

### Config Loading Chain

```
┌─────────────────┐     ┌──────────────────────┐     ┌────────────┐
│ System Property │ ──> │ AppConfig.getInstance │ ──> │ Step Def   │
│ -Denv=staging   │     │                      │     │ Constructor│
└─────────────────┘     │  getPlatform()       │     │            │
                        │  getRegion()         │     │ set(PLAT)  │
┌─────────────────┐     │  getEnv()            │     │ set(REGION)│
│ config.properties│ ──>│  getEnvironment()    │     │ set(ENV)   │
│ platform=us1    │     │  getEnvPropertyValue │     │ set(ENVIR) │
│ region=US       │     │                      │     │            │
└─────────────────┘     └──────────────────────┘     └────────────┘
         │
         ▼
┌─────────────────┐
│ env-specific     │
│ us1_int.props    │
│ us1_staging.props│
└─────────────────┘
```

### Base Step Definition Constructor (Auto-Init)

```java
// Every step definition class auto-loads these into context
public XFApiStepDefinitionBase(ScenarioContext scenarioContext) {
    super(scenarioContext);
    this.scenarioContext.set(UTAFConstant.PLATFORM, AppConfig.getInstance().getPlatform());
    this.scenarioContext.set(UTAFConstant.REGION, AppConfig.getInstance().getRegion());
    this.scenarioContext.set(UTAFConstant.ENV, AppConfig.getInstance().getEnv());
    this.scenarioContext.set(UTAFConstant.ENVIRONMENT, AppConfig.getInstance().getEnvironment());
    this.scenarioContext.set("envUniqueKey", AppConfig.getInstance().getEnvPropertyValue("envUniqueKey"));
}
```

### Available in Every Scenario Automatically

| Key | Example Value | Source |
|-----|---------------|--------|
| `platform` | `us1_int` | `config.properties` |
| `region` | `US` | `config.properties` |
| `env` | `INT` | `config.properties` |
| `environment` | `us1_int` | `config.properties` |
| `envUniqueKey` | `usint1` | env-specific properties |

---

## 10. Data Flow Diagrams

### Complete Data Flow Summary

```
┌─────────────────────────────────────────────────────────────┐
│                    DATA INPUT SOURCES                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Feature File        2. JSON Request Files               │
│  ┌─────────────────┐   ┌───────────────────────┐           │
│  │ Parameters       │   │ api/iam/create.json   │           │
│  │ DataTable        │   │ { "path": "/users",   │           │
│  │ DocString        │   │   "body": {           │           │
│  │ Examples         │   │     "name":"${name}"  │           │
│  │ ${variables}     │   │   }                   │           │
│  └────────┬────────┘   │ }                     │           │
│           │             └──────────┬────────────┘           │
│           │                        │                        │
│  3. Config Properties   4. System Properties                │
│  ┌─────────────────┐   ┌───────────────────────┐           │
│  │ config.properties│   │ -Dbrowser=chrome      │           │
│  │ staging.props    │   │ -Denv=staging         │           │
│  │ user credentials │   │ -Dcucumber.filter.tags│           │
│  └────────┬────────┘   └──────────┬────────────┘           │
│           │                        │                        │
└───────────┼────────────────────────┼────────────────────────┘
            │                        │
            ▼                        ▼
┌─────────────────────────────────────────────────────────────┐
│                   PROCESSING LAYER                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Step Definitions                                           │
│  ┌─────────────────────────────────────────────────┐       │
│  │ 1. Receive data from feature file               │       │
│  │ 2. Resolve ${variables} via ScenarioContext      │       │
│  │ 3. Load config values via AppConfig              │       │
│  │ 4. Execute action (API call / UI action)         │       │
│  │ 5. Store results back in ScenarioContext          │       │
│  └─────────────────────────────────────────────────┘       │
│                         │                                   │
│                         ▼                                   │
│  ScenarioContext (Shared State)                             │
│  ┌─────────────────────────────────────────────────┐       │
│  │ { platform, region, env, environment,            │       │
│  │   accessToken, userId, appName, apiid,           │       │
│  │   response: { status, body, headers } }          │       │
│  └─────────────────────────────────────────────────┘       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────────────┐
│                    OUTPUT TARGETS                             │
├─────────────────────────────────────────────────────────────┤
│  ┌────────────┐ ┌────────────┐ ┌────────────────────────┐  │
│  │ API Server │ │ Browser UI │ │ Desktop App            │  │
│  │ (RestAssur)│ │ (Selenium/ │ │ (WinAppDriver)         │  │
│  │            │ │ Playwright)│ │                        │  │
│  └────────────┘ └────────────┘ └────────────────────────┘  │
│  ┌────────────┐ ┌────────────┐ ┌────────────────────────┐  │
│  │ Reports    │ │ Screenshots│ │ Logs                   │  │
│  │ (JSON/HTML)│ │ (PNG)      │ │ (Log4j2)               │  │
│  └────────────┘ └────────────┘ └────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### Quick Reference: All Data Passing Methods

| Method | Direction | Example |
|--------|-----------|---------|
| **Inline Parameter** | Feature → Step | `"casxfuser1"` |
| **DataTable** | Feature → Step | `\| key \| value \|` |
| **DocString** | Feature → Step | `"""JSON body"""` |
| **Scenario Outline** | Feature → Step | `Examples:` table |
| **ScenarioContext.set()** | Step → Context | `set("userId", "42")` |
| **ScenarioContext.get()** | Context → Step | `get("userId")` → `"42"` |
| **ScenarioContext.setResponse()** | Step → Context | Store API response |
| **ScenarioContext.getResponse()** | Context → Step | Retrieve response |
| **${variable}** | Context → Feature/JSON | Auto-resolved from context |
| **AppConfig** | Config → Step | `getEnvPropertyValue("key")` |
| **JSON File** | File → Step | Request templates with `${}` |
| **System Property** | CLI → Step | `-Dbrowser=chrome` |

