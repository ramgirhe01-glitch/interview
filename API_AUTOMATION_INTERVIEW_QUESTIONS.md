# API Automation Interview Questions — Complete Guide

## Table of Contents
1. [API & REST Fundamentals](#1-api--rest-fundamentals)
2. [HTTP Methods & Status Codes](#2-http-methods--status-codes)
3. [RestAssured Framework](#3-restassured-framework)
4. [Authentication & Authorization](#4-authentication--authorization)
5. [Request & Response Validation](#5-request--response-validation)
6. [JSON & XML Parsing](#6-json--xml-parsing)
7. [Schema Validation](#7-schema-validation)
8. [API Chaining & Data-Driven](#8-api-chaining--data-driven)
9. [Serialization & Deserialization (POJO)](#9-serialization--deserialization-pojo)
10. [Cucumber + RestAssured Integration](#10-cucumber--restassured-integration)
11. [Headers, Cookies & Query Params](#11-headers-cookies--query-params)
12. [File Upload/Download via API](#12-file-uploaddownload-via-api)
13. [API Performance & Security Testing](#13-api-performance--security-testing)
14. [Mocking & Stubbing APIs](#14-mocking--stubbing-apis)
15. [Framework Design for API Testing](#15-framework-design-for-api-testing)
16. [Scenario-Based & Coding Questions](#16-scenario-based--coding-questions)
17. [Tricky & Advanced Questions](#17-tricky--advanced-questions)

---

## 1. API & REST Fundamentals

### Q1: What is an API? What is a REST API?

**API (Application Programming Interface):** A set of rules/contracts that allows two software applications to communicate with each other.

**REST (Representational State Transfer):** An architectural style for designing APIs over HTTP.

**REST Principles:**

| Principle | Meaning |
|-----------|---------|
| **Client-Server** | Client and server are separate; communicate via HTTP |
| **Stateless** | Each request contains all info needed; server stores no client state |
| **Cacheable** | Responses can be cached for performance |
| **Uniform Interface** | Consistent URL patterns, standard HTTP methods |
| **Layered System** | Client doesn't know if it talks to end server or intermediary |
| **Resource-Based** | Everything is a resource identified by URI |

```
Client (Browser/App)  ──HTTP Request──>  Server (API)
                      <──HTTP Response──
```

---

### Q2: What is the difference between REST and SOAP?

| Feature | REST | SOAP |
|---------|------|------|
| Protocol | HTTP only | HTTP, SMTP, TCP, etc. |
| Data Format | JSON, XML, Text, HTML | XML only |
| Speed | ⚡ Faster (lightweight) | 🔴 Slower (XML overhead) |
| WSDL | Not required | Required (strict contract) |
| Stateless | Yes | Can be stateful |
| Caching | ✅ Supported | ❌ Not supported |
| Security | HTTPS, OAuth, JWT | WS-Security (built-in) |
| Error Handling | HTTP status codes | SOAP Fault element |
| Bandwidth | Low | High (XML verbose) |
| Use Case | Web/Mobile APIs, Microservices | Banking, Enterprise, Legacy |

---

### Q3: What is the difference between API and Web Service?

| Feature | API | Web Service |
|---------|-----|-------------|
| Scope | Broader — any interface | Subset of API |
| Network | Can be local or remote | Always over a network |
| Protocol | Any | HTTP, SOAP, REST |
| Examples | OS API, Library API, REST API | REST API, SOAP service |

**All web services are APIs, but not all APIs are web services.**

---

### Q4: What is an Endpoint, Base URI, and Resource?

```
https://api.example.com/v3/users/42/orders?status=active
|__________________________|____|__|______|______________|
      Base URI             Ver  Resource  Path Param  Query Param

Full URL = Endpoint
```

| Term | Example | Meaning |
|------|---------|---------|
| **Base URI** | `https://api.example.com` | Root URL of the API |
| **Resource** | `/users`, `/orders` | Entity being accessed |
| **Endpoint** | `https://api.example.com/v3/users/42` | Full URL to access a resource |
| **Path Param** | `/users/42` — `42` is the param | Identifies a specific resource |
| **Query Param** | `?status=active&page=2` | Filters/modifies the response |

---

### Q5: What is the difference between Path Parameter and Query Parameter?

| Feature | Path Parameter | Query Parameter |
|---------|---------------|-----------------|
| Syntax | `/users/{id}` → `/users/42` | `/users?role=admin` |
| Purpose | Identify specific resource | Filter/sort/paginate |
| Required? | Usually required | Usually optional |
| Position | Part of URL path | After `?` in URL |
| Multiple | `/users/42/orders/7` | `?role=admin&status=active` |

```java
// Path parameter
given().pathParam("id", 42)
    .when().get("/users/{id}");      // /users/42

// Query parameter
given().queryParam("role", "admin")
       .queryParam("page", 1)
    .when().get("/users");           // /users?role=admin&page=1
```

---

### Q6: What is Idempotency? Which HTTP methods are idempotent?

**Idempotent:** Making the same request multiple times produces the **same result** as making it once.

| Method | Idempotent? | Explanation |
|--------|-------------|-------------|
| **GET** | ✅ Yes | Reading data doesn't change it |
| **PUT** | ✅ Yes | Replace resource — same result every time |
| **DELETE** | ✅ Yes | Delete once or 100 times — resource is gone |
| **PATCH** | ❌ No* | Partial update may have different effects |
| **POST** | ❌ No | Creates new resource each time |

*PATCH **can** be idempotent depending on implementation, but is not guaranteed.

---

## 2. HTTP Methods & Status Codes

### Q7: Explain all HTTP methods with examples.

| Method | CRUD | Purpose | Request Body | Example |
|--------|------|---------|-------------|---------|
| **GET** | Read | Retrieve resource | ❌ No | `GET /users/42` |
| **POST** | Create | Create new resource | ✅ Yes | `POST /users` + body |
| **PUT** | Update (Full) | Replace entire resource | ✅ Yes | `PUT /users/42` + full body |
| **PATCH** | Update (Partial) | Update specific fields | ✅ Yes | `PATCH /users/42` + partial body |
| **DELETE** | Delete | Remove resource | ❌ Usually no | `DELETE /users/42` |
| **HEAD** | — | Same as GET but no body | ❌ No | Check if resource exists |
| **OPTIONS** | — | Get allowed methods | ❌ No | CORS preflight check |

---

### Q8: What is the difference between PUT and PATCH?

```json
// Current resource: GET /users/42
{
  "id": 42,
  "name": "John",
  "email": "john@test.com",
  "role": "user"
}

// PUT /users/42 — REPLACES entire resource (must send ALL fields)
{
  "id": 42,
  "name": "John",
  "email": "john.doe@test.com",
  "role": "admin"
}
// If you omit "role", it becomes null!

// PATCH /users/42 — Updates ONLY specified fields
{
  "email": "john.doe@test.com"
}
// Only email changes. name and role remain unchanged.
```

| Feature | PUT | PATCH |
|---------|-----|-------|
| Update Type | Full replacement | Partial update |
| Body | Must include ALL fields | Only changed fields |
| Missing Fields | Set to null/default | Unchanged |
| Idempotent | ✅ Yes | ❌ Not guaranteed |

---

### Q9: Explain all HTTP status codes. ⭐ Most Asked

#### 1xx — Informational
| Code | Meaning |
|------|---------|
| 100 | Continue |
| 101 | Switching Protocols |

#### 2xx — Success ✅
| Code | Meaning | When |
|------|---------|------|
| **200** | OK | GET/PUT/PATCH success |
| **201** | Created | POST — new resource created |
| **202** | Accepted | Request accepted, processing async |
| **204** | No Content | DELETE success — no body returned |

#### 3xx — Redirection 🔄
| Code | Meaning | When |
|------|---------|------|
| **301** | Moved Permanently | URL changed permanently |
| **302** | Found (Temporary Redirect) | URL changed temporarily |
| **304** | Not Modified | Cached version is still valid |

#### 4xx — Client Error ❌
| Code | Meaning | When |
|------|---------|------|
| **400** | Bad Request | Invalid request body/params |
| **401** | Unauthorized | Missing or invalid authentication |
| **403** | Forbidden | Authenticated but no permission |
| **404** | Not Found | Resource doesn't exist |
| **405** | Method Not Allowed | Wrong HTTP method (POST on GET-only endpoint) |
| **408** | Request Timeout | Server timed out waiting for request |
| **409** | Conflict | Resource conflict (duplicate entry) |
| **413** | Payload Too Large | Request body exceeds limit |
| **415** | Unsupported Media Type | Wrong Content-Type |
| **422** | Unprocessable Entity | Validation error |
| **429** | Too Many Requests | Rate limit exceeded |

#### 5xx — Server Error 💥
| Code | Meaning | When |
|------|---------|------|
| **500** | Internal Server Error | Unhandled server exception |
| **502** | Bad Gateway | Upstream server returned invalid response |
| **503** | Service Unavailable | Server is down/overloaded |
| **504** | Gateway Timeout | Upstream server didn't respond in time |

---

### Q10: What is the difference between 401 and 403?

| Code | 401 Unauthorized | 403 Forbidden |
|------|-----------------|---------------|
| Meaning | "Who are you?" | "I know who you are, but NO" |
| Cause | Missing/invalid/expired token | Valid token but insufficient permissions |
| Fix | Provide valid credentials | Get higher role/permissions |
| Example | No Bearer token in header | Regular user trying admin endpoint |

```java
// 401 — No token
given().get("/admin/users");  // 401

// 401 — Expired token
given().header("Authorization", "Bearer expired_token").get("/admin/users");  // 401

// 403 — Valid token but no admin role
given().header("Authorization", "Bearer user_token").get("/admin/users");  // 403

// 200 — Valid admin token
given().header("Authorization", "Bearer admin_token").get("/admin/users");  // 200
```

---

### Q11: What is the difference between 200 and 201?

| Code | 200 OK | 201 Created |
|------|--------|-------------|
| When | GET, PUT, PATCH success | POST — new resource created |
| Body | Resource data | Newly created resource |
| Header | — | Often includes `Location` header with new resource URL |

```
POST /users → 201 Created
Location: /users/42

GET /users/42 → 200 OK
{ "id": 42, "name": "John" }
```

---

## 3. RestAssured Framework

### Q12: What is RestAssured? Why use it?

**RestAssured** is a Java library for testing REST APIs. It provides a fluent (readable) DSL for HTTP requests.

**Why RestAssured?**
- ✅ Fluent BDD-style syntax (`given().when().then()`)
- ✅ Built-in JSON/XML parsing (JsonPath, XmlPath)
- ✅ Schema validation
- ✅ Supports all HTTP methods
- ✅ Easy authentication (Basic, OAuth, Bearer)
- ✅ Seamless integration with JUnit, TestNG, Cucumber
- ✅ Request/response logging
- ✅ Java-based — works in your Gradle/Maven project

---

### Q13: Explain the RestAssured request structure (given-when-then).

```java
import static io.restassured.RestAssured.*;
import static org.hamcrest.Matchers.*;

given()          // PRECONDITIONS — headers, auth, body, params
    .baseUri("https://api.example.com")
    .header("Authorization", "Bearer token123")
    .contentType(ContentType.JSON)
    .queryParam("page", 1)
    .body("{ \"name\": \"John\" }")

.when()          // ACTION — HTTP method + endpoint
    .post("/users")

.then()          // VALIDATION — status, body, headers
    .statusCode(201)
    .body("name", equalTo("John"))
    .body("id", notNullValue())
    .header("Content-Type", containsString("json"))
    .time(lessThan(3000L))   // Response time < 3 seconds
    .log().all();            // Print full response
```

---

### Q14: How do you extract values from a response?

```java
// Method 1: Extract entire response
Response response = given()
    .when().get("/users/42")
    .then().statusCode(200)
    .extract().response();

// Get values from response
String name = response.jsonPath().getString("name");
int id = response.jsonPath().getInt("id");
List<String> emails = response.jsonPath().getList("data.email");
String header = response.getHeader("Content-Type");
int statusCode = response.getStatusCode();
long time = response.getTime();
String body = response.getBody().asString();

// Method 2: Extract single value directly
String name = given()
    .when().get("/users/42")
    .then().extract().path("name");

// Method 3: JsonPath standalone
JsonPath json = new JsonPath(response.asString());
String city = json.getString("address.city");
List<Integer> ids = json.getList("data.id");
Map<String, Object> firstUser = json.getMap("data[0]");
```

---

### Q15: How do you log request and response in RestAssured?

```java
// Log everything
given().log().all()          // Log full request
    .when().get("/users")
    .then().log().all();     // Log full response

// Selective logging
given().log().headers()      // Log only headers
       .log().body()         // Log only body
       .log().params()       // Log only parameters
       .log().method()       // Log HTTP method
       .log().uri()          // Log full URI
    .when().get("/users")
    .then().log().status()   // Log only status line
           .log().body()     // Log only response body
           .log().headers(); // Log only response headers

// Log only on failure
given()
    .when().get("/users")
    .then().log().ifError()              // Log if status >= 400
           .log().ifStatusCodeIsEqualTo(500)  // Log if 500
           .log().ifValidationFails();   // Log if assertion fails

// Global logging (applies to all requests)
RestAssured.enableLoggingOfRequestAndResponseIfValidationFails();

// Log with filters (hide sensitive data)
RestAssured.filters(new RequestLoggingFilter(), new ResponseLoggingFilter());
```

---

### Q16: What is RequestSpecification and ResponseSpecification?

**Reusable request/response templates** to avoid repetition.

```java
// === RequestSpecification — reusable request setup ===
RequestSpecification authRequest = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .addHeader("Authorization", "Bearer token123")
    .setContentType(ContentType.JSON)
    .addFilter(new RequestLoggingFilter())
    .addFilter(new ResponseLoggingFilter())
    .build();

// Use in multiple tests
given().spec(authRequest)
    .when().get("/users")
    .then().statusCode(200);

given().spec(authRequest)
    .body(payload)
    .when().post("/users")
    .then().statusCode(201);

// === ResponseSpecification — reusable response validation ===
ResponseSpecification successResponse = new ResponseSpecBuilder()
    .expectStatusCode(200)
    .expectContentType(ContentType.JSON)
    .expectResponseTime(lessThan(5000L))
    .expectHeader("Server", notNullValue())
    .build();

// Use in multiple tests
given().spec(authRequest)
    .when().get("/users")
    .then().spec(successResponse)           // Apply reusable validation
           .body("data", not(empty()));     // Plus extra validation

given().spec(authRequest)
    .when().get("/products")
    .then().spec(successResponse);
```

---

### Q17: How do you set a global base URI in RestAssured?

```java
// Method 1: Static field
RestAssured.baseURI = "https://api.example.com";
RestAssured.port = 443;
RestAssured.basePath = "/v3";

// Now all requests use this base
given().when().get("/users");  // → https://api.example.com:443/v3/users

// Method 2: In RequestSpecification
RequestSpecification spec = new RequestSpecBuilder()
    .setBaseUri("https://api.example.com")
    .setBasePath("/v3")
    .build();

// Method 3: Reset after tests
RestAssured.reset();
```

---

## 4. Authentication & Authorization

### Q18: How do you handle different authentication types in RestAssured?

```java
// === 1. Basic Auth ===
given().auth().basic("username", "password")
    .when().get("/secure/resource");

// Preemptive Basic Auth (sends credentials in first request itself)
given().auth().preemptive().basic("username", "password")
    .when().get("/secure/resource");

// === 2. Bearer Token ===
given().header("Authorization", "Bearer eyJhbGciOi...")
    .when().get("/api/users");

// === 3. OAuth 2.0 ===
given().auth().oauth2("access_token_here")
    .when().get("/api/users");

// === 4. API Key (Header) ===
given().header("X-API-Key", "your-api-key-123")
    .when().get("/api/data");

// === 5. API Key (Query Param) ===
given().queryParam("api_key", "your-api-key-123")
    .when().get("/api/data");

// === 6. OAuth 1.0 ===
given().auth().oauth("consumerKey", "consumerSecret", "accessToken", "tokenSecret")
    .when().get("/api/resource");

// === 7. Digest Auth ===
given().auth().digest("username", "password")
    .when().get("/secure/resource");

// === 8. Certificate / Mutual TLS ===
given().keyStore("keystore.jks", "password")
       .trustStore("truststore.jks", "password")
    .when().get("/secure/endpoint");
```

---

### Q19: What is OAuth 2.0? Explain the flow.

**OAuth 2.0** is an authorization framework that lets apps get limited access to user accounts.

```
┌──────────┐                              ┌──────────────┐
│  Client  │ ──1. Request Auth Code──────> │ Auth Server  │
│  (App)   │ <──2. Auth Code────────────── │ (e.g., Okta) │
│          │ ──3. Exchange Code+Secret───> │              │
│          │ <──4. Access Token + Refresh─ │              │
│          │ ──5. API Call + Token────────>│──────────────│
│          │                               │ Resource     │
│          │ <──6. Protected Data───────── │ Server (API) │
└──────────┘                              └──────────────┘
```

**Common Grant Types:**

| Grant Type | Use Case |
|------------|----------|
| **Authorization Code** | Web apps (most secure) |
| **Client Credentials** | Server-to-server (no user) |
| **Password** | Trusted apps (legacy) |
| **Implicit** | SPAs (deprecated) |
| **PKCE** | Mobile/Desktop apps |

**Client Credentials flow (most common in API testing):**

```java
// Step 1: Get access token
Response tokenResponse = given()
    .contentType("application/x-www-form-urlencoded")
    .formParam("grant_type", "client_credentials")
    .formParam("client_id", "my-client-id")
    .formParam("client_secret", "my-client-secret")
    .formParam("scope", "read write")
    .when().post("https://auth.example.com/oauth/token");

String accessToken = tokenResponse.jsonPath().getString("access_token");

// Step 2: Use token in API calls
given().header("Authorization", "Bearer " + accessToken)
    .when().get("/api/users")
    .then().statusCode(200);
```

---

### Q20: What is the difference between Authentication and Authorization?

| Feature | Authentication (AuthN) | Authorization (AuthZ) |
|---------|----------------------|---------------------|
| Question | "Who are you?" | "What can you do?" |
| Verifies | Identity | Permissions |
| How | Username/password, Token, Certificate | Roles, Scopes, Policies |
| HTTP Code on Failure | **401 Unauthorized** | **403 Forbidden** |
| Example | Login with credentials | Admin vs User access |
| Happens | First | After authentication |

---

### Q21: What is JWT (JSON Web Token)? How do you validate it?

**JWT Structure:** `header.payload.signature`

```
eyJhbGciOiJSUzI1NiJ9.eyJzdWIiOiIxMjM0NTYiLCJuYW1lIjoiSm9obiJ9.signature_here
|______________________|___________________________________________|________________|
       Header                        Payload                         Signature
    (algorithm)              (claims: sub, name, exp)           (verification)
```

```java
// Decode JWT payload (no library needed)
String jwt = "eyJhbGci...";
String[] parts = jwt.split("\\.");
String payload = new String(Base64.getUrlDecoder().decode(parts[1]));
// {"sub":"123456","name":"John","exp":1700000000}

// Validate JWT fields
JsonPath payloadJson = new JsonPath(payload);
assertEquals("123456", payloadJson.getString("sub"));
assertTrue(payloadJson.getLong("exp") > System.currentTimeMillis() / 1000);  // Not expired

// Verify in API response
given().auth().oauth2(token)
    .when().get("/api/profile")
    .then().statusCode(200)
    .body("sub", equalTo("123456"));
```

---

## 5. Request & Response Validation

### Q22: How do you validate the response body in RestAssured?

```java
given()
    .when().get("/users/42")
    .then().statusCode(200)

    // String assertions
    .body("name", equalTo("John"))
    .body("name", containsString("Jo"))
    .body("name", startsWith("J"))
    .body("name", not(emptyString()))

    // Numeric assertions
    .body("age", equalTo(30))
    .body("age", greaterThan(18))
    .body("age", lessThanOrEqualTo(65))
    .body("salary", closeTo(50000.0, 100.0))

    // Null checks
    .body("id", notNullValue())
    .body("deletedAt", nullValue())

    // Collection assertions
    .body("hobbies", hasSize(3))
    .body("hobbies", hasItem("coding"))
    .body("hobbies", hasItems("coding", "reading"))
    .body("hobbies", containsInAnyOrder("reading", "coding", "gaming"))
    .body("roles", everyItem(not(emptyString())))

    // Nested object
    .body("address.city", equalTo("New York"))
    .body("address.zip", matchesPattern("\\d{5}"))

    // Array element
    .body("data[0].name", equalTo("John"))
    .body("data.size()", equalTo(10))
    .body("data.name", hasItem("John"))

    // Boolean
    .body("active", equalTo(true))

    // Response time
    .time(lessThan(3000L));
```

---

### Q23: How do you validate response headers?

```java
given()
    .when().get("/users")
    .then()
    .header("Content-Type", "application/json; charset=utf-8")
    .header("Content-Type", containsString("json"))
    .header("Cache-Control", notNullValue())
    .header("X-Request-Id", matchesPattern("[a-f0-9-]+"))
    .headers("Content-Type", containsString("json"),
             "Server", notNullValue());

// Extract header
Response response = given().when().get("/users").then().extract().response();
String contentType = response.getHeader("Content-Type");
Headers allHeaders = response.getHeaders();
```

---

### Q24: How do you validate response time?

```java
// In assertion
given().when().get("/users")
    .then().time(lessThan(2000L));  // Less than 2 seconds

// Extract response time
Response response = given().when().get("/users").then().extract().response();
long timeMs = response.getTime();
long timeMs2 = response.getTimeIn(TimeUnit.MILLISECONDS);

// Custom assertion
assertThat(response.getTime()).isLessThan(3000);
```

---

### Q25: How do you use Hamcrest Matchers with RestAssured?

```java
import static org.hamcrest.Matchers.*;

// Equality
equalTo("John")           // Exact match
not(equalTo("Jane"))      // Not equal
is("John")                // Same as equalTo

// String
containsString("oh")      // Contains substring
startsWith("Jo")          // Starts with
endsWith("hn")            // Ends with
emptyString()             // Empty string
matchesPattern("\\d+")    // Regex match

// Numeric
greaterThan(10)
greaterThanOrEqualTo(10)
lessThan(100)
lessThanOrEqualTo(100)
closeTo(50.0, 0.5)        // Within range

// Null
nullValue()
notNullValue()

// Collection
hasSize(5)                 // List size
hasItem("apple")           // Contains item
hasItems("apple", "banana")// Contains multiple items
containsInAnyOrder("b","a")// All items in any order
everyItem(greaterThan(0))  // Every item matches
empty()                    // Empty collection

// Logical
allOf(greaterThan(0), lessThan(100))   // AND
anyOf(equalTo("admin"), equalTo("user"))// OR
not(equalTo("banned"))                  // NOT

// Type
instanceOf(String.class)
```

---

## 6. JSON & XML Parsing

### Q26: How do you parse JSON response using JsonPath?

```java
// Given response:
// {
//   "page": 1,
//   "total": 100,
//   "data": [
//     { "id": 1, "name": "John", "email": "john@test.com", "address": { "city": "NYC" } },
//     { "id": 2, "name": "Jane", "email": "jane@test.com", "address": { "city": "LA" } }
//   ]
// }

Response response = given().when().get("/users");
JsonPath json = response.jsonPath();

// Simple values
int page = json.getInt("page");                    // 1
int total = json.getInt("total");                  // 100

// Array elements
String firstName = json.getString("data[0].name"); // "John"
String lastEmail = json.getString("data[-1].email");// "jane@test.com" (last)

// Nested objects
String city = json.getString("data[0].address.city"); // "NYC"

// Lists
List<String> names = json.getList("data.name");        // ["John", "Jane"]
List<Integer> ids = json.getList("data.id");           // [1, 2]
List<Map<String, Object>> allData = json.getList("data");

// Find with condition (Groovy GPath)
List<String> nycUsers = json.getList("data.findAll { it.address.city == 'NYC' }.name");
String firstAdmin = json.getString("data.find { it.id == 1 }.name");
int maxId = json.getInt("data.max { it.id }.id");
int count = json.getInt("data.size()");

// Map
Map<String, Object> user = json.getMap("data[0]");
```

---

### Q27: What is GPath in RestAssured? Give examples.

**GPath** is a Groovy-based path expression syntax used by RestAssured for JSON/XML querying.

```java
// Given response:
// { "data": [
//   { "id": 1, "name": "John", "age": 30, "dept": "IT" },
//   { "id": 2, "name": "Jane", "age": 25, "dept": "HR" },
//   { "id": 3, "name": "Bob",  "age": 35, "dept": "IT" }
// ]}

// Find all names where dept = IT
json.getList("data.findAll { it.dept == 'IT' }.name");  // ["John", "Bob"]

// Find first user older than 28
json.getString("data.find { it.age > 28 }.name");       // "John"

// Sum of ages
json.getInt("data.sum { it.age }");                      // 90

// Min / Max
json.getInt("data.min { it.age }.age");                  // 25
json.getInt("data.max { it.age }.age");                  // 35

// Collect (transform)
json.getList("data.collect { it.name.toUpperCase() }");  // ["JOHN", "JANE", "BOB"]

// Count
json.getInt("data.count { it.dept == 'IT' }");           // 2

// Every / Any
json.getBoolean("data.every { it.age > 20 }");           // true
json.getBoolean("data.any { it.dept == 'Finance' }");    // false

// Sort
json.getList("data.sort { a, b -> a.age <=> b.age }.name"); // ["Jane", "John", "Bob"]
```

---

### Q28: How do you parse XML response?

```java
// Given XML response:
// <users>
//   <user id="1"><name>John</name><email>john@test.com</email></user>
//   <user id="2"><name>Jane</name><email>jane@test.com</email></user>
// </users>

Response response = given().when().get("/users.xml");
XmlPath xml = response.xmlPath();

// Extract values
String name = xml.getString("users.user[0].name");        // "John"
String id = xml.getString("users.user[0].@id");            // "1" (attribute)
List<String> names = xml.getList("users.user.name");       // ["John", "Jane"]

// In-line assertion
given().when().get("/users.xml")
    .then()
    .body("users.user[0].name", equalTo("John"))
    .body("users.user.size()", equalTo(2));
```

---

## 7. Schema Validation

### Q29: What is JSON Schema Validation? How to implement it?

**JSON Schema** defines the structure, types, and constraints of a JSON response.

**Step 1: Create schema file** `src/test/resources/schemas/user_schema.json`
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["id", "name", "email"],
  "properties": {
    "id": {
      "type": "integer",
      "minimum": 1
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 100
    },
    "email": {
      "type": "string",
      "format": "email"
    },
    "age": {
      "type": "integer",
      "minimum": 0,
      "maximum": 150
    },
    "roles": {
      "type": "array",
      "items": { "type": "string" },
      "minItems": 1
    },
    "address": {
      "type": "object",
      "properties": {
        "city": { "type": "string" },
        "zip": { "type": "string", "pattern": "^\\d{5}$" }
      }
    }
  },
  "additionalProperties": false
}
```

**Step 2: Validate in test**
```java
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchemaInClasspath;
import static io.restassured.module.jsv.JsonSchemaValidator.matchesJsonSchema;

// From classpath
given().when().get("/users/42")
    .then().statusCode(200)
    .body(matchesJsonSchemaInClasspath("schemas/user_schema.json"));

// From file
File schemaFile = new File("src/test/resources/schemas/user_schema.json");
given().when().get("/users/42")
    .then().body(matchesJsonSchema(schemaFile));

// From string
String schema = "{ \"type\": \"object\", \"required\": [\"id\"] }";
given().when().get("/users/42")
    .then().body(matchesJsonSchema(schema));
```

**Gradle dependency:**
```groovy
implementation 'io.rest-assured:json-schema-validator:5.5.0'
```

---

### Q30: Why is schema validation important in API testing?

| Reason | Explanation |
|--------|-------------|
| **Contract Testing** | Ensures API response matches agreed contract |
| **Catch Breaking Changes** | Detects missing fields, type changes, new required fields |
| **Data Type Safety** | Validates string/int/boolean/array types |
| **Required Fields** | Ensures mandatory fields are always present |
| **Format Validation** | Email, date, UUID patterns |
| **Regression Safety** | Catches unintended API changes early |

---

## 8. API Chaining & Data-Driven

### Q31: What is API Chaining? Give an example.

**API Chaining:** Output of one API call becomes input of the next.

```java
// Step 1: Create user → get user ID
Response createResponse = given()
    .contentType(ContentType.JSON)
    .body("{ \"name\": \"John\", \"email\": \"john@test.com\" }")
    .when().post("/users")
    .then().statusCode(201)
    .extract().response();

int userId = createResponse.jsonPath().getInt("id");  // 42

// Step 2: Get user by ID (using ID from Step 1)
given()
    .pathParam("id", userId)
    .when().get("/users/{id}")
    .then().statusCode(200)
    .body("name", equalTo("John"));

// Step 3: Update user
given()
    .contentType(ContentType.JSON)
    .pathParam("id", userId)
    .body("{ \"name\": \"John Updated\" }")
    .when().put("/users/{id}")
    .then().statusCode(200)
    .body("name", equalTo("John Updated"));

// Step 4: Delete user
given()
    .pathParam("id", userId)
    .when().delete("/users/{id}")
    .then().statusCode(204);

// Step 5: Verify deletion
given()
    .pathParam("id", userId)
    .when().get("/users/{id}")
    .then().statusCode(404);
```

---

### Q32: How do you do data-driven API testing?

```java
// Method 1: Scenario Outline in Cucumber
// user_api.feature
// Scenario Outline: Create users with different data
//   When I create user with name "<name>" and email "<email>"
//   Then response status should be <status>
//   Examples:
//     | name  | email           | status |
//     | John  | john@test.com   | 201    |
//     | ""    | invalid         | 400    |
//     | Jane  | jane@test.com   | 201    |

// Method 2: External JSON file
String testData = new String(Files.readAllBytes(Paths.get("testdata/users.json")));
List<Map<String, Object>> users = JsonPath.from(testData).getList("$");
for (Map<String, Object> user : users) {
    given().contentType(ContentType.JSON)
        .body(user)
        .when().post("/users")
        .then().statusCode(201);
}

// Method 3: Excel data (Apache POI)
ExcelReader reader = new ExcelReader("testdata/users.xlsx");
List<Map<String, String>> data = reader.getSheetData("Users");
for (Map<String, String> row : data) {
    given().contentType(ContentType.JSON)
        .body(Map.of("name", row.get("Name"), "email", row.get("Email")))
        .when().post("/users")
        .then().statusCode(Integer.parseInt(row.get("ExpectedStatus")));
}
```

---

## 9. Serialization & Deserialization (POJO)

### Q33: What is Serialization and Deserialization in API testing?

| Term | Direction | Meaning |
|------|-----------|---------|
| **Serialization** | Java Object → JSON | Convert POJO to request body |
| **Deserialization** | JSON → Java Object | Convert response to POJO |

```java
// === POJO Class ===
@Data  // Lombok generates getters, setters, toString, equals, hashCode
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private int id;
    private String name;
    private String email;
    private String job;
}

// === Serialization (Java → JSON) ===
User user = new User(0, "John", "john@test.com", "QA Engineer");

Response response = given()
    .contentType(ContentType.JSON)
    .body(user)                          // Auto-serialized to JSON
    .when().post("/users")
    .then().statusCode(201)
    .extract().response();

// === Deserialization (JSON → Java) ===
User createdUser = response.as(User.class);   // Auto-deserialized
System.out.println(createdUser.getName());     // "John"
System.out.println(createdUser.getId());       // 42

// === Deserialize list ===
List<User> users = given()
    .when().get("/users")
    .then().extract()
    .jsonPath().getList("data", User.class);

// === Using Jackson ObjectMapper directly ===
ObjectMapper mapper = new ObjectMapper();
String json = mapper.writeValueAsString(user);          // Serialize
User fromJson = mapper.readValue(jsonString, User.class); // Deserialize
```

---

### Q34: What is the difference between Jackson and Gson?

| Feature | Jackson | Gson |
|---------|---------|------|
| Speed | ⚡ Faster | 🟡 Slower |
| Default in RestAssured | ✅ Yes | Needs config |
| Annotations | `@JsonProperty`, `@JsonIgnore` | `@SerializedName`, `@Expose` |
| Streaming API | ✅ Yes | ✅ Yes |
| Tree Model | `JsonNode` | `JsonElement` |
| Null Handling | Configurable | Skips nulls by default |
| Community | Larger | Simpler API |

```java
// Jackson annotation
public class User {
    @JsonProperty("user_name")   // Maps JSON "user_name" to Java "name"
    private String name;

    @JsonIgnore                  // Excluded from JSON
    private String password;

    @JsonFormat(pattern = "yyyy-MM-dd")
    private Date createdAt;
}

// Gson annotation
public class User {
    @SerializedName("user_name")
    private String name;

    @Expose(serialize = false)   // Excluded from serialization
    private String password;
}
```

---

## 10. Cucumber + RestAssured Integration

### Q35: How do you integrate Cucumber with RestAssured? Show the complete flow.

**Feature File:**
```gherkin
@api @users
Feature: User API CRUD Operations

  @smoke
  Scenario: Create and retrieve a user
    Given the API base URL is configured
    And I set the request header "Content-Type" to "application/json"
    When I send a POST request to "/api/users" with body:
      """
      {
        "name": "John Doe",
        "job": "QA Engineer"
      }
      """
    Then the response status code should be 201
    And the response should contain "name" as "John Doe"
    And I store response field "id" into variable "userId"
    When I send a GET request to "/api/users/${userId}"
    Then the response status code should be 200
```

**Step Definitions:**
```java
public class UserApiSteps {
    private ScenarioContext context;
    private RequestSpecBuilder requestSpec;
    private Response response;

    public UserApiSteps(ScenarioContext context) {
        this.context = context;
    }

    @Given("the API base URL is configured")
    public void configureBaseUrl() {
        requestSpec = new RequestSpecBuilder()
            .setBaseUri(ConfigReader.get("api.base.url"))
            .setContentType(ContentType.JSON);
    }

    @And("I set the request header {string} to {string}")
    public void setHeader(String key, String value) {
        requestSpec.addHeader(key, value);
    }

    @When("I send a POST request to {string} with body:")
    public void sendPost(String endpoint, String body) {
        String resolvedBody = resolveVariables(body);
        response = given().spec(requestSpec.build())
            .body(resolvedBody)
            .when().post(endpoint);
        context.setResponse(response);
    }

    @When("I send a GET request to {string}")
    public void sendGet(String endpoint) {
        String resolvedEndpoint = resolveVariables(endpoint);
        response = given().spec(requestSpec.build())
            .when().get(resolvedEndpoint);
        context.setResponse(response);
    }

    @Then("the response status code should be {int}")
    public void verifyStatusCode(int expected) {
        assertEquals(expected, response.getStatusCode());
    }

    @And("the response should contain {string} as {string}")
    public void verifyField(String key, String value) {
        assertEquals(value, response.jsonPath().getString(key));
    }

    @And("I store response field {string} into variable {string}")
    public void storeField(String jsonPath, String variable) {
        String value = response.jsonPath().getString(jsonPath);
        context.set(variable, value);
    }

    private String resolveVariables(String input) {
        // Replace ${variable} with context values
        for (Map.Entry<String, Object> entry : context.getData().entrySet()) {
            input = input.replace("${" + entry.getKey() + "}", entry.getValue().toString());
        }
        return input;
    }
}
```

---

### Q36: How do you share data between API and UI steps in Cucumber?

```java
// API Step: Create user and store token
@Given("I authenticate via API as {string}")
public void authenticateViaApi(String userKey) {
    String token = getAuthToken(userKey);
    scenarioContext.set("accessToken", token);    // Store in shared context
}

// UI Step: Use token from API
@When("I set auth cookie in browser")
public void setAuthCookie() {
    String token = scenarioContext.get("accessToken");  // Retrieve from context
    Cookie cookie = new Cookie("auth_token", token);
    driver.manage().addCookie(cookie);
    driver.navigate().refresh();                        // Now logged in via UI
}

// API Step: Verify UI action via API
@Then("I verify via API that the user was created")
public void verifyViaApi() {
    String userId = scenarioContext.get("createdUserId");  // Set by UI step
    given().auth().oauth2(scenarioContext.get("accessToken"))
        .when().get("/api/users/" + userId)
        .then().statusCode(200);
}
```

---

## 11. Headers, Cookies & Query Params

### Q37: How do you work with Headers in RestAssured?

```java
// Single header
given().header("Authorization", "Bearer token123")
       .header("Accept-Language", "en-US");

// Multiple headers via Map
Map<String, String> headers = new HashMap<>();
headers.put("Authorization", "Bearer token123");
headers.put("X-Request-Id", UUID.randomUUID().toString());
headers.put("Accept", "application/json");
given().headers(headers);

// Content-Type shortcut
given().contentType(ContentType.JSON);       // application/json
given().contentType(ContentType.XML);        // application/xml
given().contentType("multipart/form-data");

// Accept shortcut
given().accept(ContentType.JSON);
```

---

### Q38: How do you work with Cookies?

```java
// Send cookie
given().cookie("session_id", "abc123")
    .when().get("/dashboard");

// Send multiple cookies
given().cookies("session_id", "abc123", "theme", "dark")
    .when().get("/dashboard");

// Extract cookies from response
Response response = given().when().post("/login");
String sessionId = response.getCookie("session_id");
Map<String, String> allCookies = response.getCookies();
Cookie detailedCookie = response.getDetailedCookie("session_id");
// detailedCookie.getPath(), .getDomain(), .getExpiryDate(), .isHttpOnly()
```

---

### Q39: How do you send Form Data vs JSON body?

```java
// JSON body (Content-Type: application/json)
given()
    .contentType(ContentType.JSON)
    .body("{ \"name\": \"John\", \"email\": \"john@test.com\" }")
    .when().post("/api/users");

// Form data (Content-Type: application/x-www-form-urlencoded)
given()
    .contentType("application/x-www-form-urlencoded")
    .formParam("grant_type", "client_credentials")
    .formParam("client_id", "my-app")
    .formParam("client_secret", "secret123")
    .when().post("/oauth/token");

// Multipart form data (file upload)
given()
    .multiPart("file", new File("test.pdf"))
    .multiPart("description", "Test document")
    .when().post("/upload");
```

---

## 12. File Upload/Download via API

### Q40: How do you upload and download files via API?

```java
// === FILE UPLOAD ===
// Single file
given()
    .multiPart("file", new File("src/test/resources/test.pdf"))
    .when().post("/api/upload")
    .then().statusCode(200)
    .body("filename", equalTo("test.pdf"));

// Multiple files
given()
    .multiPart("files", new File("file1.pdf"))
    .multiPart("files", new File("file2.pdf"))
    .multiPart("description", "Batch upload")
    .when().post("/api/upload/batch")
    .then().statusCode(200);

// File with custom content type
given()
    .multiPart("file", new File("image.png"), "image/png")
    .when().post("/api/upload");

// === FILE DOWNLOAD ===
byte[] fileBytes = given()
    .when().get("/api/download/report.pdf")
    .then().statusCode(200)
    .header("Content-Type", "application/pdf")
    .extract().asByteArray();

// Save to file
Files.write(Paths.get("downloads/report.pdf"), fileBytes);

// Verify file size
assertTrue(fileBytes.length > 0);

// Download as InputStream
InputStream stream = given()
    .when().get("/api/download/report.pdf")
    .then().extract().asInputStream();
```

---

## 13. API Performance & Security Testing

### Q41: What are the key API security tests you should automate?

| Test | What to Verify | Example |
|------|---------------|---------|
| **Authentication** | Endpoints reject unauthenticated requests | Call without token → 401 |
| **Authorization** | Users can't access others' data | User A can't GET User B's data → 403 |
| **SQL Injection** | API handles malicious input | Send `' OR 1=1 --` in params |
| **XSS** | API sanitizes output | Send `<script>alert(1)</script>` → encoded in response |
| **Rate Limiting** | API enforces rate limits | Send 100 requests/sec → 429 |
| **HTTPS** | API uses encryption | Verify TLS certificate |
| **CORS** | Proper origin restrictions | Check `Access-Control-Allow-Origin` header |
| **Input Validation** | Rejects invalid data | Oversized payload → 413 |
| **Sensitive Data** | Passwords not in response | GET /users → no password field |
| **Broken Object Access** | Can't access other's resources by guessing ID | GET /users/999 → 403 not 200 |

```java
// Test: Unauthenticated request should return 401
@Test
public void testNoAuthReturns401() {
    given().when().get("/api/users")
        .then().statusCode(401);
}

// Test: SQL Injection attempt
@Test
public void testSqlInjection() {
    given().queryParam("name", "' OR 1=1 --")
        .when().get("/api/users")
        .then().statusCode(anyOf(is(400), is(200)))
        .body("data.size()", not(greaterThan(1)));  // Should not return all records
}

// Test: Rate limiting
@Test
public void testRateLimiting() {
    for (int i = 0; i < 150; i++) {
        Response r = given().when().get("/api/data");
        if (r.getStatusCode() == 429) {
            assertTrue(i > 50);  // Rate limit kicked in after ~100 requests
            return;
        }
    }
}
```

---

### Q42: How do you test API response time / performance?

```java
// Simple response time check
given().when().get("/api/users")
    .then().time(lessThan(2000L));  // Under 2 seconds

// Measure and log
Response response = given().when().get("/api/users").then().extract().response();
long responseTime = response.getTime();
System.out.println("Response time: " + responseTime + "ms");

// Concurrent requests (basic load test)
ExecutorService executor = Executors.newFixedThreadPool(10);
List<Future<Long>> times = new ArrayList<>();
for (int i = 0; i < 50; i++) {
    times.add(executor.submit(() -> {
        return given().when().get("/api/users").then().extract().response().getTime();
    }));
}
long avgTime = times.stream().mapToLong(f -> {
    try { return f.get(); } catch (Exception e) { return 0; }
}).average().orElse(0);
System.out.println("Average response time: " + avgTime + "ms");
assertTrue(avgTime < 3000);
```

---

## 14. Mocking & Stubbing APIs

### Q43: What is API Mocking? When and how do you use it?

**Mocking** = Creating a fake API server that returns predefined responses.

**When to Mock:**
- Dependent API is not yet built
- Third-party API has rate limits
- Need predictable test data
- Test error scenarios (500, timeout)
- Offline testing

**Tools:** WireMock, MockServer, Postman Mock, JSON Server

```java
// === WireMock Example ===
// Dependency: com.github.tomakehurst:wiremock-jre8:3.0.0

@Rule
WireMockRule wireMock = new WireMockRule(8089);

@Test
public void testWithMock() {
    // Setup mock
    stubFor(get(urlEqualTo("/api/users/1"))
        .willReturn(aResponse()
            .withStatus(200)
            .withHeader("Content-Type", "application/json")
            .withBody("{ \"id\": 1, \"name\": \"John\" }")));

    // Test against mock
    given().baseUri("http://localhost:8089")
        .when().get("/api/users/1")
        .then().statusCode(200)
        .body("name", equalTo("John"));

    // Mock error scenario
    stubFor(get(urlEqualTo("/api/users/999"))
        .willReturn(aResponse()
            .withStatus(404)
            .withBody("{ \"error\": \"User not found\" }")));

    // Mock slow response
    stubFor(get(urlEqualTo("/api/slow"))
        .willReturn(aResponse()
            .withStatus(200)
            .withFixedDelay(5000)));  // 5 second delay
}
```

---

## 15. Framework Design for API Testing

### Q44: How did you design your API automation framework?

**Answer:**

```
┌─────────────────────────────────────────────────────────┐
│              API Test Framework Architecture              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  .feature files                                         │
│  ┌──────────────────────────────┐                      │
│  │ @api @users                  │                      │
│  │ Feature: User API            │                      │
│  │   Scenario: Create user      │                      │
│  │     Given base URL configured│                      │
│  │     When POST /users         │                      │
│  │     Then status 201          │                      │
│  └──────────────┬───────────────┘                      │
│                 │                                       │
│                 ▼                                       │
│  Step Definitions                                      │
│  ┌──────────────────────────────┐                      │
│  │ UserApiSteps.java            │                      │
│  │ AuthApiSteps.java            │                      │
│  │ CommonApiSteps.java          │                      │
│  └──────────────┬───────────────┘                      │
│                 │                                       │
│        ┌────────┼────────┐                             │
│        ▼        ▼        ▼                             │
│  ┌──────────┐ ┌──────┐ ┌─────────────┐               │
│  │ApiHelper │ │POJO  │ │ScenarioCtx  │               │
│  │(RestAss.)│ │Models│ │(Shared Data)│               │
│  └──────────┘ └──────┘ └─────────────┘               │
│        │                                               │
│        ▼                                               │
│  ┌─────────────────────────────┐                      │
│  │ Request JSON Files          │                      │
│  │ api/iam/create-user.json    │                      │
│  │ api/iam/get-user.json       │                      │
│  └─────────────────────────────┘                      │
│                                                         │
│  Utilities: ConfigReader, JsonUtil, SchemaValidator     │
│  Reports: Cucumber HTML, Extent, Allure                │
└─────────────────────────────────────────────────────────┘
```

**Key Design Decisions:**
1. **RequestSpecification** for reusable request setup
2. **ResponseSpecification** for common validations
3. **ScenarioContext** for data sharing between steps
4. **POJO classes** for serialization/deserialization
5. **JSON schema files** for contract testing
6. **External JSON files** for request templates with `${variable}` substitution
7. **ApiHelper wrapper** for clean fluent API calls
8. **Environment-based config** (`staging.properties`, `prod.properties`)

---

### Q45: What is the difference between API Testing, Integration Testing, and Contract Testing?

| Type | What It Tests | Tool |
|------|--------------|------|
| **API Testing** | Individual API endpoint functionality | RestAssured, Postman |
| **Integration Testing** | Multiple APIs working together | RestAssured + test framework |
| **Contract Testing** | API response matches agreed schema/contract | Pact, JSON Schema Validator |
| **E2E Testing** | Full flow: UI → API → DB → Response | Selenium + RestAssured |

---

## 16. Scenario-Based & Coding Questions

### Q46: Write a complete CRUD test for a User API.

```java
public class UserCrudTest {

    private static final String BASE_URL = "https://reqres.in";
    private int userId;

    @Test
    @Order(1)
    public void createUser() {
        User user = new User("John Doe", "QA Lead");

        Response response = given()
            .baseUri(BASE_URL)
            .contentType(ContentType.JSON)
            .body(user)
            .log().all()
        .when()
            .post("/api/users")
        .then()
            .log().all()
            .statusCode(201)
            .body("name", equalTo("John Doe"))
            .body("id", notNullValue())
            .time(lessThan(3000L))
            .extract().response();

        userId = response.jsonPath().getInt("id");
        assertThat(userId).isGreaterThan(0);
    }

    @Test
    @Order(2)
    public void getUser() {
        given()
            .baseUri(BASE_URL)
        .when()
            .get("/api/users/2")
        .then()
            .statusCode(200)
            .body("data.id", equalTo(2))
            .body("data.email", containsString("@"))
            .body("data.first_name", notNullValue());
    }

    @Test
    @Order(3)
    public void updateUser() {
        given()
            .baseUri(BASE_URL)
            .contentType(ContentType.JSON)
            .body("{ \"name\": \"Updated\", \"job\": \"Director\" }")
        .when()
            .put("/api/users/2")
        .then()
            .statusCode(200)
            .body("name", equalTo("Updated"))
            .body("updatedAt", notNullValue());
    }

    @Test
    @Order(4)
    public void deleteUser() {
        given()
            .baseUri(BASE_URL)
        .when()
            .delete("/api/users/2")
        .then()
            .statusCode(204);
    }
}
```

---

### Q47: How do you test pagination?

```java
@Test
public void testPagination() {
    int page = 1;
    int totalPages;
    List<String> allUserNames = new ArrayList<>();

    do {
        Response response = given()
            .queryParam("page", page)
            .queryParam("per_page", 10)
            .when().get("/api/users")
            .then().statusCode(200)
            .extract().response();

        totalPages = response.jsonPath().getInt("total_pages");
        List<String> names = response.jsonPath().getList("data.first_name");
        allUserNames.addAll(names);

        // Verify page metadata
        assertEquals(page, response.jsonPath().getInt("page"));
        assertTrue(names.size() <= 10);

        page++;
    } while (page <= totalPages);

    // Verify no duplicates
    assertEquals(allUserNames.size(), new HashSet<>(allUserNames).size());
}
```

---

### Q48: How do you test negative scenarios?

```java
// 1. Missing required fields
given().contentType(ContentType.JSON)
    .body("{}")  // Empty body
    .when().post("/api/users")
    .then().statusCode(400)
    .body("error", containsString("required"));

// 2. Invalid data types
given().contentType(ContentType.JSON)
    .body("{ \"age\": \"not-a-number\" }")
    .when().post("/api/users")
    .then().statusCode(400);

// 3. Non-existent resource
given().when().get("/api/users/999999")
    .then().statusCode(404);

// 4. Invalid method
given().when().patch("/api/readonly-resource")
    .then().statusCode(405);

// 5. Invalid auth
given().header("Authorization", "Bearer invalid-token")
    .when().get("/api/secure")
    .then().statusCode(401);

// 6. Duplicate creation
given().contentType(ContentType.JSON)
    .body("{ \"email\": \"existing@test.com\" }")
    .when().post("/api/users")
    .then().statusCode(409);  // Conflict

// 7. Exceeding field length
String longName = "A".repeat(10001);
given().contentType(ContentType.JSON)
    .body("{ \"name\": \"" + longName + "\" }")
    .when().post("/api/users")
    .then().statusCode(anyOf(is(400), is(413)));

// 8. Special characters / XSS
given().contentType(ContentType.JSON)
    .body("{ \"name\": \"<script>alert('xss')</script>\" }")
    .when().post("/api/users")
    .then().statusCode(anyOf(is(400), is(201)))
    .body("name", not(containsString("<script>")));

// 9. Wrong content type
given().contentType(ContentType.XML)
    .body("<user><name>John</name></user>")
    .when().post("/api/users")
    .then().statusCode(415);  // Unsupported Media Type

// 10. Empty/null values
given().contentType(ContentType.JSON)
    .body("{ \"name\": null, \"email\": \"\" }")
    .when().post("/api/users")
    .then().statusCode(400);
```

---

### Q49: How do you test an API that requires another API's token?

```java
// Reusable method to get token
public String getAuthToken(String clientId, String clientSecret) {
    return given()
        .contentType("application/x-www-form-urlencoded")
        .formParam("grant_type", "client_credentials")
        .formParam("client_id", clientId)
        .formParam("client_secret", clientSecret)
    .when()
        .post("/oauth/token")
    .then()
        .statusCode(200)
        .extract().jsonPath().getString("access_token");
}

// Use in tests
@BeforeAll
public void setup() {
    String token = getAuthToken("myApp", "secret123");
    RequestSpecification authSpec = new RequestSpecBuilder()
        .addHeader("Authorization", "Bearer " + token)
        .setContentType(ContentType.JSON)
        .setBaseUri("https://api.example.com")
        .build();
    RestAssured.requestSpecification = authSpec;
}

@Test
public void testSecureEndpoint() {
    // Token is automatically included via spec
    given().when().get("/api/users")
        .then().statusCode(200);
}
```

---

### Q50: How do you test an API with dynamic/random test data?

```java
import com.github.javafaker.Faker;

Faker faker = new Faker();

// Generate random test data
String randomName = faker.name().fullName();        // "John Smith"
String randomEmail = faker.internet().emailAddress(); // "john.smith@gmail.com"
String randomPhone = faker.phoneNumber().phoneNumber();
String randomAddress = faker.address().fullAddress();
int randomAge = faker.number().numberBetween(18, 65);
String randomUUID = UUID.randomUUID().toString();

// Use in API request
Map<String, Object> payload = new HashMap<>();
payload.put("name", randomName);
payload.put("email", randomEmail);
payload.put("age", randomAge);

Response response = given()
    .contentType(ContentType.JSON)
    .body(payload)
    .when().post("/api/users")
    .then().statusCode(201)
    .extract().response();

// Verify the values match what we sent
assertEquals(randomName, response.jsonPath().getString("name"));
assertEquals(randomEmail, response.jsonPath().getString("email"));
```

---

## 17. Tricky & Advanced Questions

### Q51: What is the difference between `body()` and `content()` in RestAssured?

They are **identical**. `content()` is an alias for `body()`. Use `body()` — it's more common.

---

### Q52: How do you handle SSL certificates in RestAssured?

```java
// Disable SSL validation (test environments only!)
RestAssured.useRelaxedHTTPSValidation();

// OR per request
given().relaxedHTTPSValidation()
    .when().get("https://self-signed.example.com/api");

// Use specific truststore
given().trustStore("path/to/truststore.jks", "password")
    .when().get("https://secure.example.com/api");

// Use keystore (mutual TLS / client certificate)
given().keyStore("path/to/keystore.p12", "password")
    .when().get("https://mtls.example.com/api");
```

---

### Q53: How do you handle async APIs?

```java
// Pattern: Submit → Poll until complete → Get result

// Step 1: Submit async request
Response submitResponse = given()
    .body(payload)
    .when().post("/api/reports/generate")
    .then().statusCode(202)  // Accepted
    .extract().response();

String jobId = submitResponse.jsonPath().getString("jobId");
String statusUrl = submitResponse.getHeader("Location");  // /api/jobs/abc123

// Step 2: Poll until complete
String status = "PENDING";
int maxRetries = 30;
int retryCount = 0;

while (!status.equals("COMPLETED") && retryCount < maxRetries) {
    Thread.sleep(2000);  // Wait 2 seconds between polls

    Response pollResponse = given()
        .when().get("/api/jobs/" + jobId)
        .then().extract().response();

    status = pollResponse.jsonPath().getString("status");
    retryCount++;

    if (status.equals("FAILED")) {
        fail("Job failed: " + pollResponse.jsonPath().getString("error"));
    }
}

assertEquals("COMPLETED", status);

// Step 3: Get result
given().when().get("/api/reports/" + jobId + "/result")
    .then().statusCode(200)
    .body("data", notNullValue());
```

---

### Q54: What is the difference between `given().body(obj)` with Map vs POJO vs String?

```java
// 1. String — raw JSON
given().body("{ \"name\": \"John\" }");
// Pro: Simple. Con: No compile-time validation, messy escaping.

// 2. Map — key-value pairs
Map<String, Object> body = Map.of("name", "John", "age", 30);
given().body(body);
// Pro: Dynamic, no class needed. Con: No type safety.

// 3. POJO — Java object
User user = new User("John", 30);
given().body(user);
// Pro: Type safe, reusable, IDE support. Con: Need class for each model.

// 4. JSONObject (org.json)
JSONObject json = new JSONObject();
json.put("name", "John");
given().body(json.toString());

// 5. File
given().body(new File("src/test/resources/request.json"));
```

---

### Q55: How do you handle GraphQL APIs with RestAssured?

```java
// GraphQL is just a POST with a specific JSON body structure

String query = """
    {
      "query": "query GetUser($id: ID!) { user(id: $id) { name email } }",
      "variables": { "id": "42" }
    }
    """;

given()
    .contentType(ContentType.JSON)
    .body(query)
    .when().post("/graphql")
    .then().statusCode(200)
    .body("data.user.name", equalTo("John"))
    .body("errors", nullValue());  // No GraphQL errors
```

---

### Q56: How do you test WebSocket APIs?

```java
// WebSocket testing needs a different library — RestAssured doesn't support it
// Use: Java-WebSocket or OkHttp

// Example with OkHttp:
OkHttpClient client = new OkHttpClient();
Request request = new Request.Builder().url("ws://api.example.com/ws").build();

WebSocket ws = client.newWebSocket(request, new WebSocketListener() {
    @Override
    public void onOpen(WebSocket webSocket, okhttp3.Response response) {
        webSocket.send("{\"type\": \"subscribe\", \"channel\": \"updates\"}");
    }

    @Override
    public void onMessage(WebSocket webSocket, String text) {
        System.out.println("Received: " + text);
        // Assert message content
    }

    @Override
    public void onFailure(WebSocket webSocket, Throwable t, okhttp3.Response response) {
        fail("WebSocket failure: " + t.getMessage());
    }
});
```

---

### Q57: What are idempotency keys and how do you test them?

**Idempotency key** = A unique ID sent with POST requests to prevent duplicate operations.

```java
String idempotencyKey = UUID.randomUUID().toString();

// First request — creates resource
Response first = given()
    .header("Idempotency-Key", idempotencyKey)
    .contentType(ContentType.JSON)
    .body(payload)
    .when().post("/api/payments")
    .then().statusCode(201)
    .extract().response();

// Second request with SAME key — should return same result, NOT create duplicate
Response second = given()
    .header("Idempotency-Key", idempotencyKey)
    .contentType(ContentType.JSON)
    .body(payload)
    .when().post("/api/payments")
    .then().statusCode(201)  // Same status
    .extract().response();

// Both should return the same resource ID
assertEquals(first.jsonPath().getString("id"), second.jsonPath().getString("id"));
```

---

### Q58: How to verify that a field does NOT exist in the response?

```java
// Method 1: Check for null
given().when().get("/users/42")
    .then().body("password", nullValue());

// Method 2: Check key doesn't exist using Hamcrest
given().when().get("/users/42")
    .then().body("$", not(hasKey("password")));

// Method 3: Extract and check
Response response = given().when().get("/users/42").then().extract().response();
assertFalse(response.jsonPath().getMap("$").containsKey("password"));

// Method 4: Check array doesn't contain
given().when().get("/users")
    .then().body("data.password", everyItem(nullValue()));
```

---

### Q59: Tell me about a challenging API testing scenario you solved.

**STAR Method Answer:**

> **Situation:** "Our microservices architecture had 15+ APIs with complex chains — creating an application required calling 8 APIs in sequence, each depending on the previous response."
>
> **Task:** "Automate the complete app creation flow including OAuth token management, SAM signature generation, and verification across services."
>
> **Action:**
> - Created a **ScenarioContext** based framework to pass data between steps
> - Built **external JSON request templates** with `${variable}` substitution
> - Implemented **SAM S3 signature generation** utility for signed API calls
> - Created **reusable ApiHelper** wrapper with auto-logging and token injection
> - Used **Cucumber tags** to run API tests independently of UI tests
> - Added **JSON schema validation** for contract testing
>
> **Result:** "Achieved 90% API test coverage across all microservices. Test execution time dropped from 45 minutes (manual) to 8 minutes (automated). The framework was adopted across 5 teams."

---

### Q60: What is the difference between Postman and RestAssured?

| Feature | Postman | RestAssured |
|---------|---------|-------------|
| Type | GUI tool | Java library (code) |
| Language | JavaScript | Java |
| Version Control | Collections (JSON) | Git-friendly Java files |
| CI/CD | Newman CLI | Gradle/Maven native |
| Dynamic Data | Pre/Post scripts | Full Java power |
| Assertion | Chai.js (`pm.expect`) | Hamcrest + AssertJ |
| Debugging | Built-in console | IDE debugger |
| Reporting | Basic | Cucumber/Extent/Allure |
| Learning Curve | Easy | Medium |
| Best For | Quick exploration, manual testing | Automated CI/CD pipeline |
| Team Collaboration | Postman Workspaces | Standard Git workflows |

---

## Quick Interview Prep Checklist

| Topic | Must-Know Questions |
|-------|-------------------|
| **REST Basics** ⭐ | Q1–Q6 (always asked) |
| **HTTP Methods & Status Codes** ⭐ | Q7–Q11 (always asked) |
| **RestAssured Syntax** ⭐ | Q12–Q17 |
| **Authentication** ⭐ | Q18–Q21 (OAuth, JWT, Bearer) |
| **Response Validation** | Q22–Q25 (Hamcrest matchers) |
| **JSON Parsing / GPath** | Q26–Q28 |
| **Schema Validation** | Q29–Q30 |
| **API Chaining** ⭐ | Q31–Q32 |
| **POJO / Serialization** | Q33–Q34 |
| **Cucumber + RestAssured** | Q35–Q36 |
| **Headers / Cookies** | Q37–Q39 |
| **Framework Design** ⭐ | Q44–Q45 |
| **Negative Testing** ⭐ | Q48 |
| **Coding** | Q46–Q50 (practice these!) |
| **Advanced** | Q51–Q59 (senior level) |

