# 📘 Scenario-Based & Java Coding Questions for SDET
### Ram Girhe | Practical Testing Scenarios + SDET Coding Problems

---

# SECTION A: API Testing Scenarios

---

## Q1: How would you test a Login API?

### Test Cases:
```
✅ POSITIVE TESTS:
1. Valid credentials → 200 + JWT token returned
2. Token format validation (header.payload.signature)
3. Token contains correct claims (userId, roles, exp)
4. Token expiry time is correct (e.g., 60 minutes)
5. Refresh token is returned alongside access token

❌ NEGATIVE TESTS:
6. Wrong password → 401 Unauthorized
7. Non-existent username → 401 (NOT 404 — don't reveal user exists!)
8. Empty username → 400 Bad Request
9. Empty password → 400 Bad Request
10. Null body → 400 Bad Request
11. SQL injection in username ("admin' OR '1'='1") → 400 (NOT 500!)
12. XSS in username ("<script>alert(1)</script>") → 400
13. Expired token used for protected endpoint → 401
14. Missing Content-Type header → 415 Unsupported Media Type
15. Invalid JSON body → 400

🔒 SECURITY TESTS:
16. Password NOT in response body
17. Token is HttpOnly and Secure flag set
18. Rate limiting after 5 failed attempts → 429 Too Many Requests
19. Account lockout after 10 failed attempts
20. Brute force protection (increasing delay)
21. HTTPS only (HTTP should redirect or reject)

⚡ PERFORMANCE TESTS:
22. Response time < 2 seconds under normal load
23. 100 concurrent login requests → no server crash
24. Token generation doesn't leak memory
```

---

## Q2: How would you test a File Upload API?

```
✅ POSITIVE:
1. Valid file (PDF, 5MB) → 201 Created
2. Response contains fileId, fileName, size, uploadedAt
3. Upload + immediately download → verify file integrity (checksum match)
4. Multiple file types: PDF, DOCX, PNG, JPG, CSV

❌ NEGATIVE:
5. Oversized file (>100MB) → 413 Payload Too Large
6. Invalid format (.exe, .bat) → 400 or 415
7. Empty file (0 bytes) → 400 Bad Request
8. Missing file in request → 400
9. Special characters in filename (test@#$.pdf) → handled gracefully
10. Very long filename (>255 chars) → 400 or truncated

🔒 SECURITY:
11. File with embedded malware → rejected or quarantined
12. Double extension (malware.pdf.exe) → rejected
13. MIME type mismatch (file says PDF but is actually EXE) → rejected

⚡ PERFORMANCE:
14. Concurrent uploads (10 users) → no data corruption
15. Large file upload doesn't timeout
16. Upload progress tracking works
```

---

## Q3: How would you test Pagination?

```
Setup: Database has 95 total records.

✅ POSITIVE:
1. GET /items?page=1&limit=10 → returns exactly 10 items
2. GET /items?page=10&limit=10 → returns 5 items (last page)
3. Total count header: X-Total-Count: 95
4. Response includes: totalPages=10, currentPage, hasNext, hasPrevious
5. Items are ordered consistently (default sort)

❌ NEGATIVE/EDGE:
6. GET /items?page=11&limit=10 → empty array [], not error
7. GET /items?page=0&limit=10 → 400 Bad Request
8. GET /items?page=-1&limit=10 → 400 Bad Request
9. GET /items?page=1&limit=0 → 400 Bad Request
10. GET /items?page=1&limit=1000 → capped at max (e.g., 100)
11. Missing page param → uses default (page=1)
12. Non-numeric page (?page=abc) → 400

🔄 CURSOR-BASED PAGINATION:
13. First request returns items + nextCursor
14. Using nextCursor returns next batch
15. No duplicate items across pages
16. No missing items across pages
17. Last page has no nextCursor (or nextCursor=null)
```

---

## Q4: How would you test a Search API?

```
✅ POSITIVE:
1. Exact match search → returns correct results
2. Partial match → returns relevant results
3. Case-insensitive search
4. Search with special characters (O'Brien, São Paulo)
5. Search with filters (category, date range, status)
6. Sort by relevance, date, name
7. Pagination works with search results

❌ NEGATIVE:
8. Empty search query → 400 or return all
9. Very long query (>1000 chars) → 400 or truncated
10. SQL injection in search term → 400
11. No results found → 200 with empty array
12. Invalid filter values → 400

⚡ PERFORMANCE:
13. Search response time < 1 second
14. Concurrent search requests handled
15. Search index is up-to-date (newly created items findable)
```

---

## Q5: How would you test a Payment/Transaction API?

```
✅ POSITIVE:
1. Valid payment → 200/201, transactionId returned
2. Correct amount deducted
3. Idempotency: same request twice → only one charge
4. Different payment methods (card, UPI, wallet)
5. Transaction status: PENDING → PROCESSING → COMPLETED

❌ NEGATIVE:
6. Insufficient balance → 402 Payment Required
7. Invalid card number → 400
8. Expired card → 400
9. Amount = 0 → 400
10. Negative amount → 400
11. Amount exceeds daily limit → 403

🔒 SECURITY:
12. Card number NOT stored in logs
13. PCI DSS compliance (masked card number in response)
14. Duplicate charge prevention
15. Timeout handling (payment gateway timeout → proper rollback)
```

---

## Q6: How would you test a DELETE API?

```
✅ POSITIVE:
1. DELETE /users/123 → 204 No Content
2. GET /users/123 after delete → 404 Not Found
3. Soft delete: GET /users/123?includeDeleted=true → returns with deleted flag

❌ NEGATIVE:
4. DELETE /users/999 (non-existent) → 404
5. DELETE without auth → 401
6. DELETE without permission → 403
7. DELETE /users/123 twice → first 204, second 404 (idempotent)
8. DELETE resource with dependencies → 409 Conflict or cascade delete

🔒 SECURITY:
9. User A cannot delete User B's resource
10. Audit log records who deleted what and when
```

---

# SECTION B: Java Coding Problems for SDET

---

## Q1: Reverse a String
```java
// Method 1: StringBuilder
public String reverse(String s) {
    return new StringBuilder(s).reverse().toString();
}

// Method 2: Two pointers (interviewers prefer this)
public String reverseManual(String s) {
    char[] chars = s.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        left++;
        right--;
    }
    return new String(chars);
}
```

---

## Q2: Find duplicate characters in a string
```java
public Map<Character, Integer> findDuplicates(String str) {
    Map<Character, Integer> map = new HashMap<>();
    for (char c : str.toCharArray())
        map.put(c, map.getOrDefault(c, 0) + 1);
    
    Map<Character, Integer> duplicates = new HashMap<>();
    map.forEach((k, v) -> { if (v > 1) duplicates.put(k, v); });
    return duplicates;
}
// Input: "programming" → {r=2, g=2, m=2}
```

---

## Q3: Check if two strings are anagrams
```java
// Method 1: Sort and compare
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    char[] a = s.toCharArray(), b = t.toCharArray();
    Arrays.sort(a);
    Arrays.sort(b);
    return Arrays.equals(a, b);
}

// Method 2: Frequency count (more efficient)
public boolean isAnagramV2(String s, String t) {
    if (s.length() != t.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < s.length(); i++) {
        count[s.charAt(i) - 'a']++;
        count[t.charAt(i) - 'a']--;
    }
    for (int c : count) if (c != 0) return false;
    return true;
}
// "listen" and "silent" → true
```

---

## Q4: FizzBuzz
```java
public List<String> fizzBuzz(int n) {
    List<String> result = new ArrayList<>();
    for (int i = 1; i <= n; i++) {
        if (i % 15 == 0) result.add("FizzBuzz");
        else if (i % 3 == 0) result.add("Fizz");
        else if (i % 5 == 0) result.add("Buzz");
        else result.add(String.valueOf(i));
    }
    return result;
}
```

---

## Q5: Find the second largest element in an array
```java
public int secondLargest(int[] arr) {
    int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int num : arr) {
        if (num > first) {
            second = first;
            first = num;
        } else if (num > second && num != first) {
            second = num;
        }
    }
    return second;
}
// [12, 35, 1, 10, 34, 1] → 34
```

---

## Q6: Check if a string is palindrome
```java
public boolean isPalindrome(String s) {
    s = s.toLowerCase().replaceAll("[^a-z0-9]", "");
    int left = 0, right = s.length() - 1;
    while (left < right) {
        if (s.charAt(left) != s.charAt(right)) return false;
        left++;
        right--;
    }
    return true;
}
// "A man, a plan, a canal: Panama" → true
```

---

## Q7: Remove duplicates from an array
```java
// Using Set
public int[] removeDuplicates(int[] arr) {
    return Arrays.stream(arr).distinct().toArray();
}

// Without extra space (sorted array)
public int removeDuplicatesSorted(int[] nums) {
    if (nums.length == 0) return 0;
    int i = 0;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i]) {
            i++;
            nums[i] = nums[j];
        }
    }
    return i + 1;
}
```

---

## Q8: Count word frequency in a sentence
```java
public Map<String, Integer> wordFrequency(String sentence) {
    Map<String, Integer> freq = new HashMap<>();
    for (String word : sentence.toLowerCase().split("\\s+"))
        freq.put(word, freq.getOrDefault(word, 0) + 1);
    return freq;
}
// "the cat sat on the mat" → {the=2, cat=1, sat=1, on=1, mat=1}
```

---

## Q9: Find missing number in array [1..n]
```java
public int missingNumber(int[] nums) {
    int n = nums.length;
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : nums) actualSum += num;
    return expectedSum - actualSum;
}
// [0, 1, 3] → 2
```

---

## Q10: Fibonacci series
```java
// Iterative (preferred)
public int fibonacci(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; i++) {
        int temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}
// 0, 1, 1, 2, 3, 5, 8, 13, 21, 34...
```

---

## Q11: Check if number is prime
```java
public boolean isPrime(int n) {
    if (n <= 1) return false;
    if (n <= 3) return true;
    if (n % 2 == 0 || n % 3 == 0) return false;
    for (int i = 5; i * i <= n; i += 6)
        if (n % i == 0 || n % (i + 2) == 0) return false;
    return true;
}
```

---

## Q12: Swap two numbers without temp variable
```java
public void swap(int a, int b) {
    a = a + b;  // a = 15
    b = a - b;  // b = 5 (original a)
    a = a - b;  // a = 10 (original b)
    // Or using XOR:
    // a = a ^ b; b = a ^ b; a = a ^ b;
}
```

---

## Q13: Java Collections — Common Interview Questions

```java
// ArrayList vs LinkedList
ArrayList<String> al = new ArrayList<>();   // Fast random access O(1), slow insert O(n)
LinkedList<String> ll = new LinkedList<>();  // Slow access O(n), fast insert O(1)

// HashMap vs TreeMap vs LinkedHashMap
HashMap<String, Integer> hm = new HashMap<>();        // Unordered, O(1) access
TreeMap<String, Integer> tm = new TreeMap<>();         // Sorted by key, O(log n)
LinkedHashMap<String, Integer> lhm = new LinkedHashMap<>(); // Insertion order, O(1)

// HashSet vs TreeSet
HashSet<String> hs = new HashSet<>();   // Unordered, O(1) add/contains
TreeSet<String> ts = new TreeSet<>();   // Sorted, O(log n) add/contains

// When to use what?
// Need order → LinkedHashMap / LinkedHashSet
// Need sorted → TreeMap / TreeSet
// Need speed → HashMap / HashSet
// Need FIFO → Queue (LinkedList)
// Need LIFO → Stack or Deque
```

---

## Q14: Java Streams — Common SDET Usage

```java
List<String> names = Arrays.asList("Ram", "Shyam", "Ram", "Sita", "Gita");

// Filter
List<String> filtered = names.stream()
    .filter(n -> n.startsWith("R"))
    .collect(Collectors.toList()); // [Ram, Ram]

// Distinct
List<String> unique = names.stream()
    .distinct()
    .collect(Collectors.toList()); // [Ram, Shyam, Sita, Gita]

// Map (transform)
List<String> upper = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// Count
long count = names.stream().filter(n -> n.length() > 3).count();

// Sort
List<String> sorted = names.stream().sorted().collect(Collectors.toList());

// Reduce
int totalLength = names.stream()
    .mapToInt(String::length)
    .sum();

// Group by
Map<Integer, List<String>> grouped = names.stream()
    .collect(Collectors.groupingBy(String::length));
```

---

# SECTION C: Testing Concepts Quick Reference

| Concept | Explanation | Example |
|---------|-------------|---------|
| **Smoke Testing** | Quick check critical paths work | Login, home page loads |
| **Sanity Testing** | Focused check on specific fix | Bug fix for login → test only login |
| **Regression Testing** | Verify nothing broke | Full suite after sprint |
| **FIT Testing** | Cross-service integration check | Agent → LLM → Response works |
| **BDD** | Tests in business language | Given/When/Then (Gherkin) |
| **TDD** | Write test first, then code | Red → Green → Refactor |
| **Boundary Value** | Test at edges | Age field: 0, 1, 17, 18, 99, 100 |
| **Equivalence Partitioning** | Group inputs, test one each | Valid: 18-60, Invalid: <18, >60 |
| **Severity vs Priority** | Impact vs Urgency | Typo on homepage = Low severity, High priority |
| **Defect Life Cycle** | Bug journey | New→Assigned→Open→Fixed→Retest→Closed |
| **Test Pyramid** | Testing layers | Unit(70%) → Integration(20%) → E2E(10%) |
| **Shift Left Testing** | Test early in SDLC | Test at design phase, not just after dev |
| **Exploratory Testing** | Unscripted, experience-based | Explore app like a real user |
| **Alpha vs Beta Testing** | Internal vs External users | Alpha=in-house, Beta=selected customers |
| **Black Box vs White Box** | Without/With code knowledge | Black=functional, White=structural |

---

*Practice writing code by hand on paper — many interviews still require this!*

