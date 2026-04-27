# ☕ Java Complete Interview Guide — SDET Focus
### Ram Girhe | 3 YOE | Exercises + Interview Questions + Deep Concepts

---

# PART 1: JAVA CORE CONCEPTS

---

## 1. OOP (Object-Oriented Programming)

### Four Pillars:

#### 1.1 Encapsulation — "Hide the data"
```java
public class Employee {
    private String name;     // private = hidden
    private double salary;

    // Getter
    public String getName() { return name; }

    // Setter with validation
    public void setSalary(double salary) {
        if (salary < 0) throw new IllegalArgumentException("Salary cannot be negative");
        this.salary = salary;
    }
}
```
**Interview Q:** *Why use encapsulation?*
> Protects data from unauthorized access, enforces validation, allows internal changes without breaking external code.

---

#### 1.2 Inheritance — "Reuse the code"
```java
// Parent class
public class Animal {
    protected String name;
    public void eat() { System.out.println(name + " is eating"); }
}

// Child class
public class Dog extends Animal {
    public void bark() { System.out.println(name + " is barking"); }

    @Override
    public void eat() { System.out.println(name + " is eating bones"); }
}

// Usage
Dog dog = new Dog();
dog.name = "Buddy";
dog.eat();  // "Buddy is eating bones" (overridden method)
dog.bark(); // "Buddy is barking"
```

**Interview Q:** *Does Java support multiple inheritance?*
> No, Java doesn't support multiple inheritance with classes (Diamond Problem). But it supports multiple inheritance through interfaces.

---

#### 1.3 Polymorphism — "One interface, many forms"

**Compile-time (Method Overloading):**
```java
public class Calculator {
    public int add(int a, int b) { return a + b; }
    public double add(double a, double b) { return a + b; }
    public int add(int a, int b, int c) { return a + b + c; }
}
```

**Runtime (Method Overriding):**
```java
Animal animal = new Dog();  // Parent reference, child object
animal.eat(); // Calls Dog's eat() — decided at RUNTIME
// animal.bark(); // ❌ Compile error — Animal doesn't have bark()
```

**Interview Q:** *What is the difference between overloading and overriding?*
| Feature | Overloading | Overriding |
|---------|-------------|------------|
| When | Compile-time | Runtime |
| Where | Same class | Parent-Child |
| Method signature | Different params | Same signature |
| Return type | Can differ | Must be same or covariant |
| Access modifier | Can differ | Cannot be more restrictive |

---

#### 1.4 Abstraction — "Show only what's needed"

**Abstract Class:**
```java
public abstract class Shape {
    abstract double area();       // No body — child MUST implement
    public void display() {       // Can have concrete methods too
        System.out.println("Area: " + area());
    }
}

public class Circle extends Shape {
    private double radius;
    public Circle(double radius) { this.radius = radius; }

    @Override
    double area() { return Math.PI * radius * radius; }
}
```

**Interface:**
```java
public interface Testable {
    void runTest();                    // abstract by default
    default void setup() {            // default method (Java 8+)
        System.out.println("Setting up...");
    }
    static void printVersion() {      // static method
        System.out.println("v1.0");
    }
}

public class ApiTest implements Testable {
    @Override
    public void runTest() {
        System.out.println("Running API test");
    }
}
```

**Interview Q:** *Abstract class vs Interface?*
| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Methods | Abstract + Concrete | Abstract + Default + Static |
| Variables | Any type | Only public static final |
| Constructor | Yes | No |
| Inheritance | Single (extends) | Multiple (implements) |
| When to use | "IS-A" with shared code | "CAN-DO" capability |

---

## 2. String Handling

```java
// String is IMMUTABLE
String s1 = "Hello";
String s2 = "Hello";
System.out.println(s1 == s2);       // true (String Pool)
System.out.println(s1.equals(s2));   // true (content comparison)

String s3 = new String("Hello");
System.out.println(s1 == s3);       // false (different objects)
System.out.println(s1.equals(s3));   // true (same content)

// StringBuilder is MUTABLE (not thread-safe, faster)
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // Modifies in-place
System.out.println(sb.toString()); // "Hello World"

// StringBuffer is MUTABLE (thread-safe, slower)
StringBuffer sbuf = new StringBuffer("Hello");
sbuf.append(" World");
```

**Interview Q:** *Why is String immutable in Java?*
> Security (used in class loading, network connections), Thread safety, String Pool optimization, Caching hashCode.

### Common String Methods:
```java
String s = "Hello World";
s.length();              // 11
s.charAt(0);             // 'H'
s.substring(0, 5);       // "Hello"
s.indexOf("World");      // 6
s.toLowerCase();          // "hello world"
s.toUpperCase();          // "HELLO WORLD"
s.trim();                 // Remove leading/trailing spaces
s.replace("World", "Java"); // "Hello Java"
s.split(" ");             // ["Hello", "World"]
s.contains("World");      // true
s.startsWith("Hello");    // true
s.isEmpty();              // false
s.toCharArray();          // char array
String.valueOf(123);      // "123"
```

---

## 3. Collections Framework

### 3.1 List (Ordered, Allows Duplicates)
```java
// ArrayList — fast random access, slow insert/delete in middle
List<String> list = new ArrayList<>();
list.add("Ram");
list.add("Shyam");
list.add("Ram");          // Duplicates allowed
list.get(0);              // "Ram" — O(1)
list.remove(1);           // Remove by index
list.contains("Ram");     // true
list.size();              // 2
list.indexOf("Ram");      // 0

// LinkedList — fast insert/delete, slow random access
List<String> linked = new LinkedList<>();
linked.add("A");
linked.addFirst("B");     // B, A
linked.addLast("C");      // B, A, C
```

### 3.2 Set (Unique, No Duplicates)
```java
// HashSet — unordered, O(1) operations
Set<String> set = new HashSet<>();
set.add("Java");
set.add("Python");
set.add("Java");          // Ignored — duplicate
System.out.println(set.size()); // 2

// LinkedHashSet — maintains insertion order
Set<String> ordered = new LinkedHashSet<>();

// TreeSet — sorted
Set<Integer> sorted = new TreeSet<>();
sorted.add(3); sorted.add(1); sorted.add(2);
System.out.println(sorted); // [1, 2, 3]
```

### 3.3 Map (Key-Value Pairs)
```java
// HashMap — unordered, O(1) operations
Map<String, Integer> map = new HashMap<>();
map.put("Ram", 90);
map.put("Shyam", 85);
map.put("Ram", 95);       // Overwrites previous value
map.get("Ram");            // 95
map.containsKey("Ram");    // true
map.containsValue(85);     // true
map.remove("Shyam");
map.size();                // 1
map.keySet();              // Set of keys
map.values();              // Collection of values
map.entrySet();            // Set of Map.Entry

// Iterate
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}

// getOrDefault
map.getOrDefault("unknown", 0); // 0

// putIfAbsent
map.putIfAbsent("Ram", 100);   // Won't overwrite existing

// computeIfAbsent (useful for grouping)
Map<Character, List<String>> groups = new HashMap<>();
groups.computeIfAbsent('R', k -> new ArrayList<>()).add("Ram");
```

### 3.4 Queue & Deque
```java
// Queue — FIFO
Queue<String> queue = new LinkedList<>();
queue.offer("First");     // Add to tail
queue.offer("Second");
queue.peek();              // "First" (view head, don't remove)
queue.poll();              // "First" (remove head)

// PriorityQueue — sorted order
PriorityQueue<Integer> pq = new PriorityQueue<>(); // Min-heap
pq.offer(3); pq.offer(1); pq.offer(2);
pq.poll(); // 1 (smallest first)

// Max-heap
PriorityQueue<Integer> maxPq = new PriorityQueue<>(Collections.reverseOrder());

// Deque — double-ended queue (can be used as Stack)
Deque<String> deque = new ArrayDeque<>();
deque.push("A");          // Stack: push to front
deque.push("B");
deque.pop();              // "B" — LIFO
```

---

## 4. Exception Handling

```java
// Checked Exception — must handle at compile time
try {
    FileReader file = new FileReader("test.txt");
} catch (FileNotFoundException e) {
    System.out.println("File not found: " + e.getMessage());
} finally {
    System.out.println("Always executes");
}

// Unchecked Exception — runtime, no forced handling
int[] arr = {1, 2, 3};
// arr[5]; // ArrayIndexOutOfBoundsException

// Multiple catch
try {
    // risky code
} catch (NullPointerException | ArrayIndexOutOfBoundsException e) {
    System.out.println("Caught: " + e.getClass().getSimpleName());
} catch (Exception e) {
    System.out.println("Generic: " + e.getMessage());
}

// Custom Exception
public class ApiTestException extends RuntimeException {
    private int statusCode;
    
    public ApiTestException(String message, int statusCode) {
        super(message);
        this.statusCode = statusCode;
    }
    
    public int getStatusCode() { return statusCode; }
}

// Throw custom exception
if (response.getStatusCode() != 200) {
    throw new ApiTestException("API failed", response.getStatusCode());
}

// try-with-resources (auto-close)
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line = br.readLine();
}  // br is automatically closed
```

**Interview Q:** *Checked vs Unchecked exceptions?*
| Type | Checked | Unchecked |
|------|---------|-----------|
| When | Compile-time | Runtime |
| Must handle? | Yes (try-catch or throws) | No |
| Extends | Exception | RuntimeException |
| Examples | IOException, SQLException | NullPointerException, ArrayIndexOutOfBounds |

**Interview Q:** *What is finally block? When does it NOT execute?*
> Always executes after try/catch. Does NOT execute if: `System.exit()` is called, JVM crashes, or thread is killed.

---

## 5. Java 8+ Features

### 5.1 Lambda Expressions
```java
// Before Java 8
Runnable r1 = new Runnable() {
    @Override
    public void run() { System.out.println("Old way"); }
};

// With Lambda
Runnable r2 = () -> System.out.println("Lambda way");

// With parameters
Comparator<String> comp = (a, b) -> a.length() - b.length();
List<String> names = Arrays.asList("Ram", "Shyam", "Gita");
names.sort(comp);
// Or inline:
names.sort((a, b) -> a.length() - b.length());
```

### 5.2 Functional Interfaces
```java
// Predicate — takes input, returns boolean
Predicate<Integer> isEven = n -> n % 2 == 0;
isEven.test(4); // true

// Function — takes input, returns output
Function<String, Integer> strLength = String::length;
strLength.apply("Hello"); // 5

// Consumer — takes input, returns nothing
Consumer<String> printer = System.out::println;
printer.accept("Hello"); // prints "Hello"

// Supplier — takes nothing, returns output
Supplier<LocalDate> today = LocalDate::now;
today.get(); // 2026-04-23
```

### 5.3 Streams API (VERY important for interviews)
```java
List<String> names = Arrays.asList("Ram", "Shyam", "Ram", "Sita", "Gita", "Raj");

// Filter — keep elements matching condition
List<String> startsWithR = names.stream()
    .filter(n -> n.startsWith("R"))
    .collect(Collectors.toList()); // [Ram, Ram, Raj]

// Map — transform each element
List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList()); // [3, 5, 3, 4, 4, 3]

// Distinct — remove duplicates
List<String> unique = names.stream()
    .distinct()
    .collect(Collectors.toList()); // [Ram, Shyam, Sita, Gita, Raj]

// Sorted
List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList()); // alphabetical

// Reduce — combine into single value
int totalLength = names.stream()
    .mapToInt(String::length)
    .sum(); // 22

// Count
long count = names.stream().filter(n -> n.length() > 3).count(); // 3

// anyMatch, allMatch, noneMatch
boolean hasRam = names.stream().anyMatch(n -> n.equals("Ram")); // true
boolean allShort = names.stream().allMatch(n -> n.length() < 10); // true

// findFirst
Optional<String> first = names.stream()
    .filter(n -> n.startsWith("S"))
    .findFirst(); // Optional[Shyam]

// Collectors.groupingBy
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
// {3=[Ram, Ram, Raj], 5=[Shyam], 4=[Sita, Gita]}

// Collectors.joining
String joined = names.stream().collect(Collectors.joining(", ")); // "Ram, Shyam, Ram, ..."

// Collectors.counting
Map<String, Long> frequency = names.stream()
    .collect(Collectors.groupingBy(n -> n, Collectors.counting()));
// {Ram=2, Shyam=1, Sita=1, Gita=1, Raj=1}

// Collectors.toMap
Map<String, Integer> nameToLength = names.stream()
    .distinct()
    .collect(Collectors.toMap(n -> n, String::length));

// flatMap — flatten nested lists
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2), Arrays.asList(3, 4), Arrays.asList(5)
);
List<Integer> flat = nested.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList()); // [1, 2, 3, 4, 5]
```

### 5.4 Optional (avoid NullPointerException)
```java
Optional<String> opt = Optional.ofNullable(getName()); // might be null
String name = opt.orElse("Unknown");           // default value
String name2 = opt.orElseThrow(() -> new RuntimeException("Name required"));
opt.ifPresent(n -> System.out.println("Name: " + n));
String upper = opt.map(String::toUpperCase).orElse("N/A");
```

---

## 6. Multithreading Basics

```java
// Way 1: Extend Thread
class MyThread extends Thread {
    public void run() { System.out.println("Thread: " + getName()); }
}
new MyThread().start();

// Way 2: Implement Runnable
Runnable task = () -> System.out.println("Runnable: " + Thread.currentThread().getName());
new Thread(task).start();

// Way 3: ExecutorService (preferred)
ExecutorService executor = Executors.newFixedThreadPool(3);
executor.submit(() -> System.out.println("Task 1"));
executor.submit(() -> System.out.println("Task 2"));
executor.shutdown();

// Synchronized (thread-safe)
public class Counter {
    private int count = 0;
    public synchronized void increment() { count++; }
    public int getCount() { return count; }
}
```

**Interview Q:** *What is the difference between Thread.start() and Thread.run()?*
> `start()` creates a new thread and calls `run()` in that thread. `run()` directly executes in the current thread — no new thread is created.

**Interview Q:** *What is a deadlock?*
> When two threads each hold a lock the other needs, both wait forever. Prevention: acquire locks in consistent order, use timeout.

---

## 7. Design Patterns (SDET-relevant)

### Singleton (one instance)
```java
public class ConfigReader {
    private static ConfigReader instance;
    private Properties props;
    
    private ConfigReader() {
        props = new Properties();
        // load properties
    }
    
    public static synchronized ConfigReader getInstance() {
        if (instance == null) instance = new ConfigReader();
        return instance;
    }
    
    public String get(String key) { return props.getProperty(key); }
}
```

### Builder (step-by-step object creation)
```java
public class TestPayload {
    private String name;
    private String type;
    private String region;
    
    private TestPayload() {}
    
    public static class Builder {
        private TestPayload payload = new TestPayload();
        public Builder name(String name) { payload.name = name; return this; }
        public Builder type(String type) { payload.type = type; return this; }
        public Builder region(String region) { payload.region = region; return this; }
        public TestPayload build() { return payload; }
    }
}
// Usage:
TestPayload p = new TestPayload.Builder().name("test").type("STANDARD").region("US").build();
```

### Factory (create objects without exposing creation logic)
```java
public class DriverFactory {
    public static WebDriver getDriver(String browser) {
        return switch (browser.toLowerCase()) {
            case "chrome" -> new ChromeDriver();
            case "firefox" -> new FirefoxDriver();
            case "edge" -> new EdgeDriver();
            default -> throw new IllegalArgumentException("Unknown browser: " + browser);
        };
    }
}
```

---

# PART 2: JAVA EXERCISES

---

## Exercise 1: Easy (15 min each)

### E1: Reverse each word in a sentence
```
Input: "Hello World Java"
Output: "olleH dlroW avaJ"
```
**Solution:**
```java
public String reverseWords(String sentence) {
    String[] words = sentence.split(" ");
    StringBuilder result = new StringBuilder();
    for (String word : words) {
        result.append(new StringBuilder(word).reverse()).append(" ");
    }
    return result.toString().trim();
}
```

### E2: Find the most frequent element in array
```
Input: [1, 3, 2, 1, 4, 1, 3]
Output: 1 (appears 3 times)
```
**Solution:**
```java
public int mostFrequent(int[] arr) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : arr) freq.put(n, freq.getOrDefault(n, 0) + 1);
    return freq.entrySet().stream()
        .max(Map.Entry.comparingByValue())
        .get().getKey();
}
```

### E3: Remove all vowels from a string
```
Input: "Hello World"
Output: "Hll Wrld"
```
**Solution:**
```java
public String removeVowels(String s) {
    return s.replaceAll("[aeiouAEIOU]", "");
}
```

### E4: Check if array is sorted
```java
public boolean isSorted(int[] arr) {
    for (int i = 1; i < arr.length; i++)
        if (arr[i] < arr[i - 1]) return false;
    return true;
}
```

### E5: Flatten a list of lists
```java
public <T> List<T> flatten(List<List<T>> nested) {
    return nested.stream()
        .flatMap(Collection::stream)
        .collect(Collectors.toList());
}
```

---

## Exercise 2: Medium (25 min each)

### M1: Find all pairs with given sum
```java
public List<int[]> findPairs(int[] arr, int target) {
    List<int[]> pairs = new ArrayList<>();
    Set<Integer> seen = new HashSet<>();
    for (int num : arr) {
        int complement = target - num;
        if (seen.contains(complement))
            pairs.add(new int[]{complement, num});
        seen.add(num);
    }
    return pairs;
}
```

### M2: Longest consecutive sequence
```java
public int longestConsecutive(int[] nums) {
    Set<Integer> set = new HashSet<>();
    for (int n : nums) set.add(n);
    int maxLen = 0;
    for (int n : set) {
        if (!set.contains(n - 1)) { // start of sequence
            int len = 1;
            while (set.contains(n + len)) len++;
            maxLen = Math.max(maxLen, len);
        }
    }
    return maxLen;
}
```

### M3: Implement a simple LRU Cache
```java
public class LRUCache extends LinkedHashMap<Integer, Integer> {
    private int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder=true
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}
```

### M4: Producer-Consumer with BlockingQueue
```java
public class ProducerConsumer {
    private BlockingQueue<Integer> queue = new LinkedBlockingQueue<>(5);
    
    public void produce() throws InterruptedException {
        for (int i = 0; i < 10; i++) {
            queue.put(i);
            System.out.println("Produced: " + i);
        }
    }
    
    public void consume() throws InterruptedException {
        while (true) {
            Integer item = queue.take();
            System.out.println("Consumed: " + item);
        }
    }
}
```

---

# PART 3: TOP 50 JAVA INTERVIEW QUESTIONS

---

### Basics
| # | Question | Key Answer |
|---|----------|------------|
| 1 | What is JVM, JRE, JDK? | JDK ⊃ JRE ⊃ JVM. JVM runs bytecode, JRE has libraries, JDK has compiler |
| 2 | What is platform independence? | Java compiles to bytecode → runs on any JVM |
| 3 | What is the difference between == and .equals()? | == compares references, .equals() compares content |
| 4 | What is autoboxing/unboxing? | Auto conversion between primitives and wrappers (int ↔ Integer) |
| 5 | What are access modifiers? | public (all), protected (package+subclass), default (package), private (class) |
| 6 | What is `static` keyword? | Belongs to class, not instance. Shared across all objects |
| 7 | What is `final` keyword? | Variable=constant, Method=can't override, Class=can't extend |
| 8 | What is `this` keyword? | Reference to current object |
| 9 | What is `super` keyword? | Reference to parent class |
| 10 | What is type casting? | Widening (auto): int→long. Narrowing (manual): double→int |

### OOP
| # | Question | Key Answer |
|---|----------|------------|
| 11 | What is method overloading? | Same method name, different parameters, same class |
| 12 | What is method overriding? | Same method name+params, parent-child classes, @Override |
| 13 | Can we override static methods? | No. Static methods are class-level, not instance-level |
| 14 | Can constructor be inherited? | No. But child calls parent constructor via super() |
| 15 | What is diamond problem? | Ambiguity in multiple inheritance. Java avoids with single class inheritance |
| 16 | What is composition vs inheritance? | Composition = "HAS-A" (preferred), Inheritance = "IS-A" |
| 17 | What is an anonymous class? | Class without name, defined and instantiated inline |

### Collections
| # | Question | Key Answer |
|---|----------|------------|
| 18 | ArrayList vs LinkedList? | AL: O(1) access, O(n) insert. LL: O(n) access, O(1) insert |
| 19 | HashMap internal working? | hashCode() → bucket index → linked list/tree in bucket |
| 20 | What happens if two keys have same hashCode? | Collision → stored in same bucket as linked list, compared with equals() |
| 21 | HashMap vs Hashtable? | HashMap: not synchronized, allows null key. Hashtable: synchronized, no null |
| 22 | HashMap vs ConcurrentHashMap? | CHM: thread-safe with segment locking, better performance than Hashtable |
| 23 | How does TreeMap work? | Red-Black tree, O(log n) operations, keys sorted |
| 24 | What is fail-fast vs fail-safe? | Fail-fast: ConcurrentModificationException (ArrayList). Fail-safe: uses copy (CopyOnWriteArrayList) |
| 25 | Comparable vs Comparator? | Comparable: natural order (compareTo in class). Comparator: custom order (external) |

### Java 8+
| # | Question | Key Answer |
|---|----------|------------|
| 26 | What is a lambda expression? | Anonymous function: (params) -> expression |
| 27 | What is a functional interface? | Interface with exactly one abstract method (@FunctionalInterface) |
| 28 | Stream vs Collection? | Stream: lazy, pipeline, single-use. Collection: eager, stored, reusable |
| 29 | What is Optional? | Container to avoid NullPointerException. ofNullable(), orElse(), map() |
| 30 | map() vs flatMap()? | map: 1-to-1 transform. flatMap: 1-to-many (flattens nested) |
| 31 | What are method references? | Shorthand for lambdas: String::length instead of s -> s.length() |
| 32 | What is a default method? | Method with body in interface (Java 8+). For backward compatibility |

### Exception Handling
| # | Question | Key Answer |
|---|----------|------------|
| 33 | Checked vs Unchecked? | Checked: compile-time (IOException). Unchecked: runtime (NPE) |
| 34 | throw vs throws? | throw: actually throws. throws: declares in method signature |
| 35 | Can finally block be skipped? | Only if System.exit(), JVM crash, or thread death |
| 36 | try-with-resources? | Auto-closes resources implementing AutoCloseable |

### Multithreading
| # | Question | Key Answer |
|---|----------|------------|
| 37 | Thread vs Runnable? | Thread: extends class (limits inheritance). Runnable: implements interface (preferred) |
| 38 | synchronized keyword? | Only one thread can access synchronized method/block at a time |
| 39 | volatile keyword? | Ensures variable is read from main memory, not thread cache |
| 40 | What is thread pool? | Pre-created threads reused for tasks. ExecutorService |

### SDET-Specific
| # | Question | Key Answer |
|---|----------|------------|
| 41 | How do you read a JSON file in Java? | Jackson: ObjectMapper.readValue() or Gson: new Gson().fromJson() |
| 42 | How do you read a properties file? | Properties.load(new FileInputStream("config.properties")) |
| 43 | How do you make HTTP calls in Java? | REST Assured, HttpClient (Java 11+), or OkHttp |
| 44 | Explain Singleton in test automation | One WebDriver instance, one ConfigReader instance |
| 45 | Explain Builder pattern in testing | Build complex payloads step by step |
| 46 | How do you handle test data? | JSON files, Excel (Apache POI), DB queries, API calls |
| 47 | What is Page Object Model? | Each page = Java class. Elements = fields. Actions = methods |
| 48 | How do you do parallel testing? | TestNG parallel="methods", Selenium Grid, JUnit5 parallel config |
| 49 | What is Maven/Gradle? | Build tools: manage dependencies, compile, run tests, generate reports |
| 50 | Explain your test framework architecture | Feature files → Step defs → Page Objects → Utils → Config → Reports |

---

# PART 4: JAVA FOR SDET — PRACTICAL PATTERNS

---

### Reading JSON payload from file
```java
// Using Jackson
ObjectMapper mapper = new ObjectMapper();
Map<String, Object> payload = mapper.readValue(
    new File("src/test/resources/payloads/create_user.json"),
    new TypeReference<>() {}
);

// Modify dynamically
payload.put("name", "Ram_" + System.currentTimeMillis());
String jsonString = mapper.writeValueAsString(payload);
```

### Reading from Properties file
```java
Properties props = new Properties();
try (InputStream is = getClass().getResourceAsStream("/config.properties")) {
    props.load(is);
}
String baseUrl = props.getProperty("base.url");
```

### Generating random test data
```java
public class TestDataGenerator {
    private static final Random random = new Random();
    
    public static String randomEmail() {
        return "test_" + System.currentTimeMillis() + "@test.com";
    }
    
    public static String randomName() {
        String[] names = {"Ram", "Shyam", "Sita", "Gita"};
        return names[random.nextInt(names.length)] + "_" + random.nextInt(1000);
    }
    
    public static int randomInt(int min, int max) {
        return random.nextInt(max - min + 1) + min;
    }
}
```

### Retry logic
```java
public static <T> T retry(Supplier<T> action, int maxRetries, long delayMs) {
    for (int i = 0; i < maxRetries; i++) {
        try {
            return action.get();
        } catch (Exception e) {
            if (i == maxRetries - 1) throw e;
            try { Thread.sleep(delayMs); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
        }
    }
    throw new RuntimeException("Should not reach here");
}

// Usage:
String result = retry(() -> apiCall(), 3, 2000);
```

---

*Master Java fundamentals + Streams + Collections = you'll ace 80% of SDET Java questions!*

