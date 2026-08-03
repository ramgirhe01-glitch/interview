
# 🎯 Complete Interview Preparation Guide — Ram Girhe
### Software Quality Engineer | 3 Years Experience | API & AI/ML Testing

---

# 📘 PART 1: DSA (Data Structures & Algorithms)

Since you have 320+ LeetCode problems solved, interviewers will expect solid fundamentals. Here's what to focus on for QA/SDET roles:

---

## 1. Arrays & Strings (Most Common)

### Pattern: Two Pointers
```java
// Reverse a string in-place
public void reverseString(char[] s) {
    int left = 0, right = s.length - 1;
    while (left < right) {
        char temp = s[left];
        s[left] = s[right];
        s[right] = temp;
        left++;
        right--;
    }
}
```

### Pattern: Sliding Window
```java
// Maximum sum subarray of size k
public int maxSumSubarray(int[] arr, int k) {
    int maxSum = 0, windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += arr[i];
    maxSum = windowSum;
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```

### Must-Practice Problems:
| Problem | Pattern | Difficulty |
|---------|---------|------------|
| Two Sum | HashMap | Easy |
| Best Time to Buy and Sell Stock | Sliding Window | Easy |
| Container With Most Water | Two Pointers | Medium |
| 3Sum | Two Pointers + Sort | Medium |
| Longest Substring Without Repeating Characters | Sliding Window | Medium |
| Product of Array Except Self | Prefix/Suffix | Medium |

---

## 2. HashMap / HashSet

```java
// Two Sum - Classic interview question
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement))
            return new int[]{map.get(complement), i};
        map.put(nums[i], i);
    }
    return new int[]{};
}
```

### Must-Practice:
- Group Anagrams
- Valid Anagram
- First Non-Repeating Character
- Frequency Count problems

---

## 3. Linked List

```java
// Reverse a Linked List (asked very often)
public ListNode reverseList(ListNode head) {
    ListNode prev = null, curr = head;
    while (curr != null) {
        ListNode next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

### Must-Practice:
- Detect cycle (Floyd's algorithm)
- Merge two sorted lists
- Remove Nth node from end
- Find middle of linked list

---

## 4. Stack & Queue

```java
// Valid Parentheses
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
        if (c == '(') stack.push(')');
        else if (c == '{') stack.push('}');
        else if (c == '[') stack.push(']');
        else if (stack.isEmpty() || stack.pop() != c) return false;
    }
    return stack.isEmpty();
}
```

---

## 5. Binary Tree / BST

```java
// Level Order Traversal (BFS)
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(root);
    while (!queue.isEmpty()) {
        int size = queue.size();
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {
            TreeNode node = queue.poll();
            level.add(node.val);
            if (node.left != null) queue.add(node.left);
            if (node.right != null) queue.add(node.right);
        }
        result.add(level);
    }
    return result;
}
```

---

## 6. Sorting & Searching

```java
// Binary Search
public int binarySearch(int[] arr, int target) {
    int lo = 0, hi = arr.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) lo = mid + 1;
        else hi = mid - 1;
    }
    return -1;
}
```

### Time Complexity Cheat Sheet:
| Algorithm | Best | Average | Worst | Space |
|-----------|------|---------|-------|-------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) |
| Binary Search | O(1) | O(log n) | O(log n) | O(1) |
| HashMap lookup | O(1) | O(1) | O(n) | O(n) |

---

## 7. Dynamic Programming (Medium-level expected)

```java
// Climbing Stairs
public int climbStairs(int n) {
    if (n <= 2) return n;
    int a = 1, b = 2;
    for (int i = 3; i <= n; i++) {
        int temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}
```

### Must-Practice DP:
- Fibonacci, Climbing Stairs
- Longest Common Subsequence
- Coin Change
- 0/1 Knapsack
- Maximum Subarray (Kadane's)

---

# 📘 PART 2: TECHNICAL INTERVIEW (Based on Your Resume)

---

## A. REST Assured — Deep Dive

### Q1: How do you structure a REST Assured test?
```java
@Test
public void testGetUser() {
    given()
        .baseUri("https://api.example.com")
        .header("Authorization", "Bearer " + token)
        .queryParam("region", "US")
    .when()
        .get("/users/123")
    .then()
        .statusCode(200)
        .body("name", equalTo("Ram"))
        .body("roles", hasItem("admin"))
        .time(lessThan(2000L)); // Response time SLA
}
```

### Q2: How do you handle OAuth2 in REST Assured?
```java
// Token generation method
public static String getOAuth2Token() {
    return given()
        .contentType("application/x-www-form-urlencoded")
        .formParam("grant_type", "client_credentials")
        .formParam("client_id", ConfigReader.get("client.id"))
        .formParam("client_secret", ConfigReader.get("client.secret"))
        .formParam("scope", "api.read api.write")
    .when()
        .post("https://auth.example.com/oauth/token")
    .then()
        .statusCode(200)
        .extract().path("access_token");
}
```

### Q3: How do you validate JSON Schema?
```java
.then()
    .assertThat()
    .body(matchesJsonSchemaInClasspath("schemas/user-schema.json"));
```

### Q4: How do you do multi-region testing?
```
Answer: We maintain region-specific config files (us.properties, eu.properties, ap.properties).
Each has its own base URL, OAuth credentials, and tenant IDs.
CI pipeline runs the same test suite 3 times with different -Dregion=US/EU/AP flag.
We validate data isolation — a tenant in US should NOT see EU data.
```

### Q5: Explain your FIT testing approach.
```
FIT = Functional Integration Testing.
After a new build is deployed, FIT tests run as a smoke check:
1. Auth token is valid
2. Core CRUD operations work
3. Cross-service calls succeed (e.g., Agent → LLM → Response)
4. Region-specific configs are loaded correctly
```

---

## B. Cucumber BDD — Deep Dive

### Q6: Write a sample feature file for Agentic AI.
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
      | type                  | status |
      | STANDARD              | 201    |
      | DESKTOP_TUNNEL        | 201    |
      | PRIVATE_NETWORK_TUNNEL| 201    |
      | INVALID               | 400    |
```

### Q7: How do you write step definitions?
```java
@Given("I have a valid OAuth2 token with scope {string}")
public void getTokenWithScope(String scope) {
    token = AuthHelper.getToken(scope);
    assertNotNull(token, "Token should not be null");
}

@When("I send a POST request to {string}")
public void sendPostRequest(String endpoint) {
    response = given()
        .header("Authorization", "Bearer " + token)
        .contentType(ContentType.JSON)
        .body(payload)
    .when()
        .post(BASE_URL + endpoint);
}

@Then("the response status code should be {int}")
public void verifyStatusCode(int expectedStatus) {
    assertEquals(expectedStatus, response.getStatusCode());
}
```

### Q8: What is the difference between Scenario and Scenario Outline?
```
Scenario: Runs once with fixed data.
Scenario Outline: Runs multiple times with data from Examples table.
Use Scenario Outline for data-driven testing (e.g., testing different regions, different input types).
```

---

## C. Selenium / Playwright

### Q9: Explain Page Object Model.
```java
public class LoginPage {
    private WebDriver driver;

    @FindBy(id = "username") private WebElement usernameField;
    @FindBy(id = "password") private WebElement passwordField;
    @FindBy(id = "loginBtn") private WebElement loginButton;

    public LoginPage(WebDriver driver) {
        this.driver = driver;
        PageFactory.initElements(driver, this);
    }

    public DashboardPage login(String user, String pass) {
        usernameField.sendKeys(user);
        passwordField.sendKeys(pass);
        loginButton.click();
        return new DashboardPage(driver);
    }
}
```

### Q10: Selenium vs Playwright — Key Differences
| Feature | Selenium | Playwright |
|---------|----------|------------|
| Language Support | Java, Python, C#, JS | JS/TS, Python, Java, C# |
| Speed | Slower (HTTP protocol) | Faster (CDP/WebSocket) |
| Auto-wait | No (need explicit waits) | Yes (built-in) |
| Multi-tab/browser | Complex | Native support |
| Network Interception | Limited | Built-in |
| Parallel Execution | Via Grid | Native contexts |
| Headless | Supported | Default |

---

## D. CI/CD & DevOps Questions

### Q11: Describe your CI/CD pipeline.
```
1. Developer pushes code → triggers pipeline
2. Build stage: Gradle compiles test code
3. Test stage: Runs Cucumber tests with -Dtags="@Smoke"
4. Region matrix: Same tests run for US, EU, AP
5. Report stage: Cucumber HTML report + trends published
6. Notification: Results pushed to JIRA/Slack
7. On failure: rerun.txt captures failed scenarios for retry
```

### Q12: How do you handle flaky tests?
```
1. Retry mechanism: Cucumber rerun plugin (rerun.txt)
2. Explicit waits instead of Thread.sleep
3. Isolate test data — each test creates its own data
4. Tag flaky tests with @Flaky, investigate root cause
5. Check if it's environment-specific (region, deployment timing)
```

---

## E. AI/ML Testing Questions (Based on Your Agentic AI Work)

### Q13: How do you test an Agentic AI service?
```
1. MCP Server Registration: CRUD operations, validate different tunnel types
2. Agent CRUD: Create, update, delete agents; validate tenant isolation
3. Guardrails: Test prompt injection detection, content filtering
4. LLM Integration: Verify fallback when primary model (Bedrock) fails
5. SSE Streaming: Validate real-time event stream format, ordering
6. Workflow Execution: Sync vs Async, validate task completion
7. Tenant Isolation: 3-level JWT (Provider → Application → Customer)
```

### Q14: How do you test SSE (Server-Sent Events)?
```java
// Using REST Assured with streaming
Response response = given()
    .header("Accept", "text/event-stream")
    .header("Authorization", "Bearer " + token)
.when()
    .get("/api/v1/agents/execute-stream");

// Parse SSE events
String body = response.getBody().asString();
String[] events = body.split("\n\n");
assertTrue(events.length > 0);
assertTrue(events[events.length - 1].contains("event: done"));
```

### Q15: What is RAG and how do you test it?
```
RAG = Retrieval-Augmented Generation
- Knowledge Bank stores documents as vector embeddings
- On query, system finds semantically similar records (not keyword match)
- Retrieved context is sent to LLM along with the user's question

Testing approach:
1. Ingest known documents into Knowledge Bank
2. Query with semantically similar (not exact) terms
3. Validate topK results contain expected records
4. Test relevance scoring threshold
5. Test metadata filtering
6. Validate 1-second SLA for search availability
7. Test bulk ingestion (100 records/request)
8. Test embedding model hot-swap + async re-indexing
```

---

# 📘 PART 3: HR & BEHAVIORAL QUESTIONS

---

## 🔹 1. Tell me about yourself.
> "I'm Ram Girhe, a Software Quality Engineer with 3 years of experience in test automation. I currently work at Expleo Solutions, deployed at Siemens Digital Industries Software. I specialize in API test automation using REST Assured and Cucumber BDD. My current focus is testing AI/ML microservices — including an Agentic AI platform, a RAG/Vector service, and a Unified Data Access service. I work across multi-region deployments (US, EU, AP), build CI/CD pipelines, and analyze daily test results. I'm also ISTQB certified and have solved 320+ LeetCode problems. I'm looking for opportunities where I can grow into more complex automation challenges."

## 🔹 2. Why are you looking for a change?
> "I've had a great learning experience at Expleo/Siemens. I've worked on 6+ microservices and gained deep expertise in API testing and AI/ML service validation. Now, I'm looking for a role where I can take on more ownership — perhaps lead a QA team, architect automation frameworks, or work on more diverse tech stacks. I want to grow both technically and in terms of responsibility."

## 🔹 3. Describe a challenging bug you found.
> "While testing the Agentic AI service across regions, I found that agent execution in the AP region was returning stale guardrail configurations. The root cause was a caching issue — the guardrail config was cached at deployment time and not refreshed when updated via API. I identified this by comparing responses across US (working) and AP (stale). I documented the issue with region-specific logs, curl commands to reproduce, and the expected vs actual behavior. The dev team fixed the cache invalidation logic within a sprint."

## 🔹 4. How do you handle tight deadlines?
> "I prioritize based on risk. I focus smoke/critical path tests first, then regression. I use tags in Cucumber (@Smoke, @Regression, @P1) to run subsets. I also automate report generation so I spend less time on reporting and more on actual testing. In my current role, I reduced reporting effort by 40% through automated pipelines."

## 🔹 5. Tell me about a time you disagreed with a team member.
> "A developer believed a particular API endpoint didn't need negative test cases since 'the frontend validates input.' I respectfully disagreed, explaining that APIs can be called directly (Postman, curl, other services) bypassing frontend validation. I demonstrated by sending an invalid payload directly and the API crashed with a 500 error instead of a proper 400. The team agreed to add server-side validation, and I added negative test cases for all endpoints."

## 🔹 6. What is your greatest strength?
> "Systematic debugging. When a test fails, I don't just rerun it — I check logs, compare regions, validate the test data, and isolate whether it's a test issue or a real bug. This approach has helped me catch several production-critical bugs that others missed."

## 🔹 7. What is your greatest weakness?
> "I sometimes spend too much time making automation code 'perfect' — adding extra validations, better logging, etc. I've learned to balance quality with delivery timelines by setting time-boxes for improvements."

## 🔹 8. Where do you see yourself in 5 years?
> "I see myself as a Senior QA Lead or Test Architect, designing automation strategies for large-scale distributed systems. I want to deepen my expertise in AI/ML testing and eventually contribute to building AI-powered testing tools."

## 🔹 9. Why should we hire you?
> "I bring a unique combination: strong API automation skills (50+ endpoints automated), experience with cutting-edge AI/ML testing (Agentic AI, RAG, LLMs), multi-region deployment expertise, and CI/CD pipeline creation. I'm also ISTQB certified and practice DSA regularly. I don't just write tests — I understand systems, find meaningful bugs, and improve processes."

## 🔹 10. Do you have any questions for us?
> Always ask:
> - "What does the QA team structure look like?"
> - "What tools and frameworks does the team currently use?"
> - "What's the biggest quality challenge the team faces right now?"
> - "How is test automation maturity measured here?"

---

# 📘 PART 4: SCENARIO-BASED QUESTIONS

## Q1: How would you test a Login API?
```
Positive:
- Valid credentials → 200 + token
- Token format (JWT) validation
- Token expiry time check

Negative:
- Wrong password → 401
- Empty username → 400
- SQL injection in username → 400 (not 500!)
- Expired token → 401
- Missing Content-Type header → 415

Performance:
- Response time < 2 seconds
- Concurrent login requests

Security:
- Password not in response body
- Token is HttpOnly, Secure
- Rate limiting after 5 failed attempts → 429
- Account lockout after 10 attempts
```

## Q2: How would you test a file upload API?
```
- Valid file (PDF, 5MB) → 201
- Oversized file (>100MB) → 413
- Invalid format (.exe) → 400/415
- Empty file → 400
- Special characters in filename → handled gracefully
- Concurrent uploads → no data corruption
- Upload + immediately download → verify integrity (checksum)
```

## Q3: How would you test pagination?
```
- First page: offset=0, limit=10 → returns 10 items
- Last page: returns remaining items (could be < limit)
- Beyond last page: returns empty array, not error
- limit=0 → 400 or empty
- Negative offset → 400
- Total count header matches actual data
- Cursor-based: validate next cursor, no duplicates across pages
```

---

# 📘 PART 5: JAVA/CODING QUESTIONS FOR SDET

## Q1: Reverse a String
```java
public String reverse(String s) {
    return new StringBuilder(s).reverse().toString();
}
```

## Q2: Find duplicate characters in a string
```java
public void findDuplicates(String str) {
    Map<Character, Integer> map = new HashMap<>();
    for (char c : str.toCharArray())
        map.put(c, map.getOrDefault(c, 0) + 1);
    map.entrySet().stream()
        .filter(e -> e.getValue() > 1)
        .forEach(e -> System.out.println(e.getKey() + ": " + e.getValue()));
}
```

## Q3: Check if two strings are anagrams
```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    char[] a = s.toCharArray(), b = t.toCharArray();
    Arrays.sort(a);
    Arrays.sort(b);
    return Arrays.equals(a, b);
}
```

## Q4: FizzBuzz
```java
for (int i = 1; i <= 100; i++) {
    if (i % 15 == 0) System.out.println("FizzBuzz");
    else if (i % 3 == 0) System.out.println("Fizz");
    else if (i % 5 == 0) System.out.println("Buzz");
    else System.out.println(i);
}
```

## Q5: Find the second largest element in an array
```java
public int secondLargest(int[] arr) {
    int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int num : arr) {
        if (num > first) { second = first; first = num; }
        else if (num > second && num != first) second = num;
    }
    return second;
}
```

---

# 📘 PART 6: TESTING CONCEPTS

| Concept | Explanation |
|---------|-------------|
| **Smoke Testing** | Quick check that critical paths work after a new build |
| **Regression Testing** | Verify existing functionality isn't broken by new changes |
| **FIT Testing** | Functional Integration Testing — validates cross-service interactions |
| **BDD** | Behavior-Driven Development — tests written in Gherkin (Given/When/Then) |
| **TDD** | Test-Driven Development — write test first, then code |
| **Boundary Value Analysis** | Test at edges: min, min+1, max-1, max |
| **Equivalence Partitioning** | Divide inputs into valid/invalid groups, test one from each |
| **Severity vs Priority** | Severity = impact; Priority = urgency. A typo on homepage = low severity, high priority |
| **Defect Life Cycle** | New → Assigned → Open → Fixed → Retest → Closed/Reopened |
| **Test Pyramid** | Unit (70%) → Integration (20%) → E2E (10%) |

---

# 🏁 QUICK REVISION CHECKLIST

- [ ] Practice top 20 LeetCode problems (Easy + Medium)
- [ ] Review REST Assured syntax & common assertions
- [ ] Practice writing Cucumber feature files from scratch
- [ ] Review OAuth2 flow (client_credentials, authorization_code)
- [ ] Understand your resume projects deeply — expect "tell me more" questions
- [ ] Prepare 3-4 STAR format stories (Situation, Task, Action, Result)
- [ ] Review HTTP status codes: 200, 201, 400, 401, 403, 404, 409, 429, 500, 503
- [ ] Know the difference between PUT vs PATCH, POST vs PUT
- [ ] Practice explaining your CI/CD pipeline on a whiteboard
- [ ] Review Java collections: List, Set, Map, Queue + when to use which

---

*Good luck with your interviews, Ram! 🚀*
