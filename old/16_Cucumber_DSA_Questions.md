# Cucumber + DSA - Questions with Answers for Test Automation

---

## TABLE OF CONTENTS
1. [String DSA in Cucumber Step Definitions](#1-string-dsa-in-cucumber)
2. [Array/List DSA in Cucumber](#2-arraylist-dsa-in-cucumber)
3. [Map/Set DSA in Cucumber](#3-mapset-dsa-in-cucumber)
4. [Sorting & Searching in Cucumber](#4-sorting--searching)
5. [Data Table & DataTable Transforms](#5-data-table-transforms)
6. [Real-World Cucumber DSA Scenarios](#6-real-world-scenarios)
7. [Cucumber Framework DSA Interview Questions](#7-interview-questions)

---

## 1. STRING DSA IN CUCUMBER

### Q1: Validate Dynamic Text with Regex in Step Definition
**Feature:**
```gherkin
Scenario: Validate order ID format
  Given I place an order
  Then the order ID should match pattern "ORD-[0-9]{6}"
```
**Step Definition:**
```java
@Then("the order ID should match pattern {string}")
public void validateOrderId(String pattern) {
    String orderId = context.get("orderId");
    assertTrue(orderId.matches(pattern),
        "Order ID " + orderId + " doesn't match " + pattern);
}
```

### Q2: Compare Two API Responses (String Diff)
```gherkin
Scenario: Compare response bodies
  Given I call API "/v1/user" and store response as "response1"
  And I call API "/v2/user" and store response as "response2"
  Then responses "response1" and "response2" should have same keys
```
```java
@Then("responses {string} and {string} should have same keys")
public void compareResponseKeys(String r1, String r2) {
    JsonObject json1 = JsonParser.parseString(context.get(r1)).getAsJsonObject();
    JsonObject json2 = JsonParser.parseString(context.get(r2)).getAsJsonObject();
    Set<String> keys1 = json1.keySet();
    Set<String> keys2 = json2.keySet();
    
    Set<String> missing = new HashSet<>(keys1);
    missing.removeAll(keys2); // keys in v1 but not v2
    
    Set<String> extra = new HashSet<>(keys2);
    extra.removeAll(keys1); // keys in v2 but not v1
    
    assertTrue(missing.isEmpty() && extra.isEmpty(),
        "Missing: " + missing + ", Extra: " + extra);
}
```

### Q3: Parse and Validate CSV Data
```gherkin
Scenario: Validate CSV row
  Given a CSV line "John,Doe,30,Engineer"
  Then field 1 should be "John"
  And field 3 should be "30"
```
```java
@Given("a CSV line {string}")
public void parseCsv(String line) {
    context.put("csvFields", line.split(","));
}

@Then("field {int} should be {string}")
public void validateField(int index, String expected) {
    String[] fields = (String[]) context.get("csvFields");
    assertEquals(expected, fields[index - 1]);
}
```

---

## 2. ARRAY/LIST DSA IN CUCUMBER

### Q4: Sort and Verify Table Data
```gherkin
Scenario: Verify products are sorted by price
  Given the following products:
    | name    | price |
    | Widget  | 25.99 |
    | Gadget  | 15.50 |
    | Tool    | 35.00 |
  Then products should be sortable by price ascending
```
```java
@Then("products should be sortable by price ascending")
public void verifySortByPrice() {
    List<Map<String, String>> products = context.get("products");
    List<Double> prices = products.stream()
        .map(p -> Double.parseDouble(p.get("price")))
        .collect(Collectors.toList());
    
    List<Double> sorted = new ArrayList<>(prices);
    Collections.sort(sorted);
    
    // Verify original can be sorted
    assertEquals(sorted, prices.stream().sorted().collect(Collectors.toList()));
}
```

### Q5: Find Duplicates in Test Data
```gherkin
Scenario: Check for duplicate entries
  Given the following user emails:
    | email              |
    | john@test.com      |
    | jane@test.com      |
    | john@test.com      |
  Then I should find 1 duplicate email
```
```java
@Then("I should find {int} duplicate email(s)")
public void findDuplicates(int expectedCount) {
    List<String> emails = context.get("emails");
    Set<String> seen = new HashSet<>();
    Set<String> duplicates = new HashSet<>();
    for (String email : emails) {
        if (!seen.add(email)) duplicates.add(email);
    }
    assertEquals(expectedCount, duplicates.size());
}
```

### Q6: Binary Search in Sorted Test Data
```gherkin
Scenario: Search for a product in sorted catalog
  Given a sorted product catalog with IDs: 101, 205, 310, 415, 520
  When I search for product ID 310
  Then the product should be found at position 3
```
```java
@When("I search for product ID {int}")
public void searchProduct(int id) {
    int[] catalog = context.get("catalog");
    int index = Arrays.binarySearch(catalog, id);
    context.put("searchResult", index);
}

@Then("the product should be found at position {int}")
public void verifyPosition(int expected) {
    int result = context.get("searchResult");
    assertEquals(expected - 1, result); // 0-based index
}
```

### Q7: Paginate Results
```gherkin
Scenario: Paginate search results
  Given 25 search results
  When I request page 2 with page size 10
  Then I should get results 11 through 20
```
```java
@When("I request page {int} with page size {int}")
public void paginate(int page, int pageSize) {
    List<String> allResults = context.get("results");
    int start = (page - 1) * pageSize;
    int end = Math.min(start + pageSize, allResults.size());
    List<String> pageResults = allResults.subList(start, end);
    context.put("pageResults", pageResults);
    context.put("startIndex", start + 1);
    context.put("endIndex", end);
}
```

---

## 3. MAP/SET DSA IN CUCUMBER

### Q8: Store and Verify Key-Value Pairs from API Response
```gherkin
Scenario: Validate JSON response fields
  Given I receive the following API response:
    | field     | expected     |
    | status    | active       |
    | name      | John Doe     |
    | role      | admin        |
  Then all fields should match expected values
```
```java
@Then("all fields should match expected values")
public void validateAllFields() {
    Map<String, String> expected = context.get("expectedFields");
    Map<String, String> actual = context.get("responseFields");
    
    Map<String, String> mismatches = new LinkedHashMap<>();
    for (Map.Entry<String, String> e : expected.entrySet()) {
        String actualVal = actual.getOrDefault(e.getKey(), "MISSING");
        if (!actualVal.equals(e.getValue())) {
            mismatches.put(e.getKey(),
                "expected=" + e.getValue() + " actual=" + actualVal);
        }
    }
    assertTrue(mismatches.isEmpty(), "Mismatches: " + mismatches);
}
```

### Q9: Frequency Count for Validation
```gherkin
Scenario: Verify status distribution
  Given the following order statuses:
    | status    |
    | PENDING   |
    | SHIPPED   |
    | PENDING   |
    | DELIVERED |
    | SHIPPED   |
    | PENDING   |
  Then status frequency should be:
    | status    | count |
    | PENDING   | 3     |
    | SHIPPED   | 2     |
    | DELIVERED | 1     |
```
```java
@Then("status frequency should be:")
public void verifyFrequency(DataTable dataTable) {
    List<String> statuses = context.get("statuses");
    Map<String, Long> actual = statuses.stream()
        .collect(Collectors.groupingBy(s -> s, Collectors.counting()));
    
    List<Map<String, String>> expected = dataTable.asMaps();
    for (Map<String, String> row : expected) {
        String status = row.get("status");
        long expectedCount = Long.parseLong(row.get("count"));
        assertEquals(expectedCount, actual.getOrDefault(status, 0L),
            "Mismatch for status: " + status);
    }
}
```

### Q10: Set Operations for Test Data Validation
```gherkin
Scenario: Validate permissions
  Given user has permissions: READ, WRITE, DELETE
  And required permissions are: READ, WRITE, EXECUTE
  Then missing permissions should be: EXECUTE
  And extra permissions should be: DELETE
```
```java
@Then("missing permissions should be: {}")
public void checkMissing(String expectedMissing) {
    Set<String> userPerms = context.get("userPermissions");
    Set<String> required = context.get("requiredPermissions");
    
    Set<String> missing = new TreeSet<>(required);
    missing.removeAll(userPerms);
    
    Set<String> expected = Arrays.stream(expectedMissing.split(",\\s*"))
        .collect(Collectors.toSet());
    assertEquals(expected, missing);
}
```

---

## 4. SORTING & SEARCHING

### Q11: Custom Sort for UI Table Verification
```gherkin
Scenario: Verify table sorted by multiple columns
  Given the employee table:
    | name  | dept   | salary |
    | Alice | IT     | 70000  |
    | Bob   | HR     | 60000  |
    | Carol | IT     | 65000  |
  When I sort by dept ascending then salary descending
  Then the order should be:
    | name  |
    | Bob   |
    | Alice |
    | Carol |
```
```java
@When("I sort by dept ascending then salary descending")
public void multiSort() {
    List<Map<String, String>> employees = context.get("employees");
    employees.sort(
        Comparator.comparing((Map<String, String> m) -> m.get("dept"))
            .thenComparing(m -> Integer.parseInt(m.get("salary")),
                Comparator.reverseOrder())
    );
    context.put("employees", employees);
}
```

### Q12: Find Element Using HashMap for O(1) Lookup in Tests
```gherkin
Scenario: Quick lookup test user by email
  Given 10000 registered users
  When I lookup user by email "user5000@test.com"
  Then the lookup should complete in under 10ms
```
```java
@Given("{int} registered users")
public void createUsers(int count) {
    Map<String, User> userMap = new HashMap<>();
    for (int i = 1; i <= count; i++) {
        String email = "user" + i + "@test.com";
        userMap.put(email, new User(email, "User" + i));
    }
    context.put("userMap", userMap);
}

@When("I lookup user by email {string}")
public void lookupUser(String email) {
    long start = System.nanoTime();
    Map<String, User> userMap = context.get("userMap");
    User user = userMap.get(email); // O(1)
    long elapsed = (System.nanoTime() - start) / 1_000_000;
    context.put("lookupTime", elapsed);
    context.put("foundUser", user);
}
```

---

## 5. DATA TABLE TRANSFORMS

### Q13: DataTable to List of Maps
```gherkin
Scenario: Process data table
  Given the following test data:
    | username | password | role  |
    | admin    | admin123 | ADMIN |
    | user1    | pass123  | USER  |
```
```java
@Given("the following test data:")
public void processDataTable(DataTable dataTable) {
    // As List of Maps
    List<Map<String, String>> data = dataTable.asMaps(String.class, String.class);
    
    // As List of Lists
    List<List<String>> raw = dataTable.asLists(String.class);
    
    // As Map (single key-value column)
    // Map<String, String> map = dataTable.asMap(String.class, String.class);
    
    // Transform to POJO
    List<UserCredential> users = data.stream()
        .map(row -> new UserCredential(
            row.get("username"), row.get("password"), row.get("role")))
        .collect(Collectors.toList());
    
    context.put("testUsers", users);
}
```

### Q14: Dynamic Scenario Outline with Data Processing
```gherkin
Scenario Outline: Validate input combinations
  Given input array "<input>"
  When I find the <operation>
  Then result should be <expected>
  
  Examples:
    | input         | operation | expected |
    | 1,5,3,9,2     | max       | 9        |
    | 1,5,3,9,2     | min       | 1        |
    | 10,20,30      | sum       | 60       |
    | 4,2,7,1       | sorted    | 1,2,4,7  |
```
```java
@Given("input array {string}")
public void parseInput(String input) {
    int[] arr = Arrays.stream(input.split(","))
        .mapToInt(Integer::parseInt).toArray();
    context.put("inputArray", arr);
}

@When("I find the max")
public void findMax() {
    int[] arr = context.get("inputArray");
    context.put("result", String.valueOf(Arrays.stream(arr).max().getAsInt()));
}

@When("I find the sorted")
public void findSorted() {
    int[] arr = context.get("inputArray");
    Arrays.sort(arr);
    String result = Arrays.stream(arr)
        .mapToObj(String::valueOf)
        .collect(Collectors.joining(","));
    context.put("result", result);
}
```

---

## 6. REAL-WORLD SCENARIOS

### Q15: Retry Logic with Queue
```java
// Retry failed test steps using a Queue
public class RetryQueue {
    private Queue<Runnable> failedSteps = new LinkedList<>();
    private int maxRetries = 3;
    
    public void addForRetry(Runnable step) {
        failedSteps.offer(step);
    }
    
    public void processRetries() {
        int retries = 0;
        while (!failedSteps.isEmpty() && retries < maxRetries) {
            Runnable step = failedSteps.poll();
            try {
                step.run();
            } catch (Exception e) {
                failedSteps.offer(step); // re-add if still failing
                retries++;
            }
        }
    }
}
```

### Q16: Test Data Builder with Map
```java
public class TestDataBuilder {
    private Map<String, Object> data = new LinkedHashMap<>();
    
    public TestDataBuilder with(String key, Object value) {
        data.put(key, value);
        return this;
    }
    
    public TestDataBuilder withDefaults() {
        data.putIfAbsent("name", "Test User");
        data.putIfAbsent("email", "test@example.com");
        data.putIfAbsent("status", "active");
        return this;
    }
    
    public Map<String, Object> build() {
        return Collections.unmodifiableMap(data);
    }
}

// In step definition:
@Given("a default user with role {string}")
public void createUser(String role) {
    Map<String, Object> user = new TestDataBuilder()
        .withDefaults()
        .with("role", role)
        .build();
    context.put("testUser", user);
}
```

### Q17: Validate Sorted Table Column from UI
```java
@Then("the {string} column should be sorted {string}")
public void verifyColumnSorted(String column, String order) {
    List<WebElement> cells = driver.findElements(
        By.cssSelector("table td." + column));
    List<String> values = cells.stream()
        .map(WebElement::getText).collect(Collectors.toList());
    
    List<String> sorted = new ArrayList<>(values);
    if (order.equals("ascending")) {
        Collections.sort(sorted);
    } else {
        sorted.sort(Comparator.reverseOrder());
    }
    assertEquals(sorted, values, column + " not sorted " + order);
}
```

### Q18: Compare Two DataTables (Diff Algorithm)
```java
@Then("the response should match expected data:")
public void compareData(DataTable expected) {
    List<Map<String, String>> expectedRows = expected.asMaps();
    List<Map<String, String>> actualRows = context.get("actualData");
    
    // Find missing rows
    List<Map<String, String>> missing = new ArrayList<>(expectedRows);
    missing.removeAll(actualRows);
    
    // Find extra rows
    List<Map<String, String>> extra = new ArrayList<>(actualRows);
    extra.removeAll(expectedRows);
    
    StringBuilder diff = new StringBuilder();
    if (!missing.isEmpty()) diff.append("Missing: ").append(missing).append("\n");
    if (!extra.isEmpty()) diff.append("Extra: ").append(extra).append("\n");
    
    assertTrue(missing.isEmpty() && extra.isEmpty(), diff.toString());
}
```

---

## 7. INTERVIEW QUESTIONS

### Q19: How do you handle large test data in Cucumber?
**Answer:**
```java
// 1. Use external files (CSV/JSON) instead of inline DataTables
@Given("users from file {string}")
public void loadFromFile(String filename) throws IOException {
    List<Map<String, String>> users = Files.lines(Path.of("testdata/" + filename))
        .skip(1) // header
        .map(line -> {
            String[] parts = line.split(",");
            Map<String, String> map = new HashMap<>();
            map.put("name", parts[0]);
            map.put("email", parts[1]);
            return map;
        })
        .collect(Collectors.toList());
    context.put("users", users);
}

// 2. Use streaming for large datasets to avoid OOM
@Then("all users should have valid emails")
public void validateEmails() throws IOException {
    long invalid = Files.lines(Path.of("testdata/users.csv"))
        .skip(1)
        .map(line -> line.split(",")[1])
        .filter(email -> !email.matches("^[\\w.-]+@[\\w.-]+\\.[a-z]{2,}$"))
        .count();
    assertEquals(0, invalid, invalid + " invalid emails found");
}
```

### Q20: How do you share state between steps efficiently?
**Answer:**
```java
// Use a thread-safe ScenarioContext with HashMap
public class ScenarioContext {
    private final Map<String, Object> context = new ConcurrentHashMap<>();
    
    @SuppressWarnings("unchecked")
    public <T> T get(String key) {
        return (T) context.get(key);
    }
    
    public void put(String key, Object value) {
        context.put(key, value);
    }
    
    public boolean containsKey(String key) {
        return context.containsKey(key);
    }
    
    public void clear() {
        context.clear();
    }
}

// Inject via PicoContainer or Spring DI
public class StepDefs {
    private final ScenarioContext context;
    
    public StepDefs(ScenarioContext context) {
        this.context = context;
    }
}
```

### Q21: How do you deduplicate test scenarios?
**Answer:**
```java
// Use Scenario Outline + Examples for parameterized tests
// Use tagged hooks for setup/teardown
// Use Background for common Given steps
// Use helper methods to avoid code duplication

// Example: Reusable validation method
public class ValidationHelper {
    private static final Map<String, Predicate<String>> VALIDATORS = Map.of(
        "email", s -> s.matches("^[\\w.-]+@[\\w.-]+\\.[a-z]{2,}$"),
        "phone", s -> s.matches("^\\+?\\d{10,13}$"),
        "zip",   s -> s.matches("^\\d{5}(-\\d{4})?$")
    );
    
    public static boolean validate(String type, String value) {
        return VALIDATORS.getOrDefault(type, s -> false).test(value);
    }
}

@Then("the {string} field {string} should be valid")
public void validateField(String type, String value) {
    assertTrue(ValidationHelper.validate(type, value));
}
```

### Q22: How do you handle test data cleanup with Collections?
```java
@After
public void cleanup(Scenario scenario) {
    // Use a Stack (LIFO) for cleanup — reverse order of creation
    Deque<Runnable> cleanupStack = context.get("cleanupActions");
    if (cleanupStack != null) {
        while (!cleanupStack.isEmpty()) {
            try {
                cleanupStack.pop().run();
            } catch (Exception e) {
                log.warn("Cleanup failed: " + e.getMessage());
            }
        }
    }
}

// Register cleanup during test
@Given("I create a temporary user {string}")
public void createTempUser(String name) {
    String userId = apiClient.createUser(name);
    Deque<Runnable> cleanup = context.getOrDefault("cleanupActions", new ArrayDeque<>());
    cleanup.push(() -> apiClient.deleteUser(userId));
    context.put("cleanupActions", cleanup);
}
```

### Q23: Time Complexity Awareness in Test Automation
| Operation | ArrayList | LinkedList | HashMap | TreeMap | HashSet |
|-----------|-----------|------------|---------|---------|---------|
| Access by index | O(1) | O(n) | - | - | - |
| Search | O(n) | O(n) | O(1) | O(log n) | O(1) |
| Insert | O(n)* | O(1)** | O(1) | O(log n) | O(1) |
| Delete | O(n) | O(1)** | O(1) | O(log n) | O(1) |
| Sort | O(n log n) | O(n log n) | - | Sorted | - |

> *O(1) amortized at end; **O(1) if you have the node reference

**When it matters in testing:**
- Use **HashMap** for O(1) lookups when validating large datasets
- Use **TreeMap** when you need sorted keys (e.g., time-series data)
- Use **LinkedHashSet** to remove duplicates while preserving order
- Use **PriorityQueue** for priority-based test execution
- Avoid **nested loops** (O(n²)) — use **Set/Map** to reduce to O(n)

---

