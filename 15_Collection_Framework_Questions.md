# Java Collection Framework - Easy & Medium Questions with Answers (All Methods Covered)

---

## TABLE OF CONTENTS
1. [List (ArrayList, LinkedList)](#1-list)
2. [Set (HashSet, LinkedHashSet, TreeSet)](#2-set)
3. [Map (HashMap, LinkedHashMap, TreeMap)](#3-map)
4. [Queue & Deque (PriorityQueue, ArrayDeque)](#4-queue--deque)
5. [Stack](#5-stack)
6. [Collections Utility Class](#6-collections-utility-class)
7. [Easy Questions](#easy-questions)
8. [Medium Questions](#medium-questions)

---

## 1. LIST

### ArrayList Methods Reference
```java
List<String> list = new ArrayList<>();

// Add
list.add("A");                    // [A]
list.add(0, "B");                 // [B, A]
list.addAll(List.of("C", "D"));   // [B, A, C, D]
list.addAll(1, List.of("X"));     // [B, X, A, C, D]

// Access
list.get(0);                      // "B"
list.indexOf("A");                // 2
list.lastIndexOf("A");            // 2
list.contains("A");               // true
list.containsAll(List.of("A","B")); // true

// Modify
list.set(0, "Z");                 // replaces index 0 → [Z, X, A, C, D]

// Remove
list.remove(0);                   // removes by index
list.remove("A");                 // removes first occurrence
list.removeAll(List.of("C","D")); // removes all matching
list.removeIf(s -> s.startsWith("X")); // conditional remove
list.retainAll(List.of("Z"));    // keep only matching
list.clear();                     // remove all

// Info
list.size();
list.isEmpty();

// Conversion
Object[] arr = list.toArray();
String[] strArr = list.toArray(new String[0]);

// Iteration
list.forEach(System.out::println);
Iterator<String> it = list.iterator();
ListIterator<String> lit = list.listIterator();
ListIterator<String> lit2 = list.listIterator(1); // start from index

// Sublist
List<String> sub = list.subList(0, 2);

// Sort
list.sort(Comparator.naturalOrder());
list.sort(Comparator.reverseOrder());

// Stream
list.stream();
list.parallelStream();

// Replce all
list.replaceAll(String::toUpperCase);

// Immutable lists (Java 9+)
List<String> immutable = List.of("A", "B", "C");
List<String> copy = List.copyOf(list);
```

### LinkedList Additional Methods
```java
LinkedList<String> ll = new LinkedList<>();
ll.addFirst("A");     ll.addLast("B");
ll.getFirst();         ll.getLast();
ll.removeFirst();      ll.removeLast();
ll.peek();   ll.peekFirst();   ll.peekLast();
ll.poll();   ll.pollFirst();   ll.pollLast();
ll.offer("C");  ll.offerFirst("D");  ll.offerLast("E");
ll.push("F");   // addFirst
ll.pop();        // removeFirst
ll.descendingIterator();
```

---

## 2. SET

### HashSet / LinkedHashSet / TreeSet Methods
```java
Set<Integer> set = new HashSet<>();      // unordered
Set<Integer> lset = new LinkedHashSet<>(); // insertion order
Set<Integer> tset = new TreeSet<>();      // sorted order

// Add
set.add(1);
set.addAll(List.of(2, 3, 4));

// Remove
set.remove(1);
set.removeAll(List.of(2, 3));
set.removeIf(n -> n > 3);
set.retainAll(List.of(4));
set.clear();

// Query
set.contains(1);
set.containsAll(List.of(1, 2));
set.size();
set.isEmpty();

// Iteration
set.forEach(System.out::println);
set.iterator();

// Conversion
set.toArray();
set.stream();

// Immutable (Java 9+)
Set<Integer> immutable = Set.of(1, 2, 3);
Set<Integer> copy = Set.copyOf(set);
```

### TreeSet Additional Methods (NavigableSet)
```java
TreeSet<Integer> ts = new TreeSet<>(List.of(1, 3, 5, 7, 9));

ts.first();          // 1
ts.last();           // 9
ts.lower(5);         // 3 (strictly less)
ts.floor(5);         // 5 (less or equal)
ts.higher(5);        // 7 (strictly greater)
ts.ceiling(5);       // 5 (greater or equal)
ts.pollFirst();      // removes and returns 1
ts.pollLast();       // removes and returns 9
ts.headSet(5);       // {3} (elements < 5)
ts.headSet(5, true); // {3, 5}
ts.tailSet(5);       // {5, 7}
ts.tailSet(5, false);// {7}
ts.subSet(3, 7);     // {3, 5} (3 inclusive, 7 exclusive)
ts.subSet(3, true, 7, true); // {3, 5, 7}
ts.descendingSet();
ts.descendingIterator();
ts.comparator();     // null if natural ordering
```

---

## 3. MAP

### HashMap / LinkedHashMap / TreeMap Methods
```java
Map<String, Integer> map = new HashMap<>();
Map<String, Integer> lmap = new LinkedHashMap<>(); // insertion order
Map<String, Integer> tmap = new TreeMap<>();       // sorted by key

// Put
map.put("a", 1);
map.putAll(Map.of("b", 2, "c", 3));
map.putIfAbsent("a", 99);  // won't replace (key exists)

// Get
map.get("a");               // 1
map.getOrDefault("z", 0);   // 0

// Remove
map.remove("a");
map.remove("b", 2);         // remove only if value matches

// Replace
map.replace("c", 30);
map.replace("c", 30, 300);  // replace only if value matches

// Query
map.containsKey("a");
map.containsValue(1);
map.size();
map.isEmpty();

// Views
map.keySet();
map.values();
map.entrySet();

// Iteration
map.forEach((k, v) -> System.out.println(k + "=" + v));
for (Map.Entry<String, Integer> e : map.entrySet()) {
    e.getKey(); e.getValue(); e.setValue(99);
}

// Compute
map.compute("a", (k, v) -> v == null ? 1 : v + 1);
map.computeIfAbsent("d", k -> 4);
map.computeIfPresent("a", (k, v) -> v + 10);
map.merge("a", 1, Integer::sum);

// Replace all
map.replaceAll((k, v) -> v * 2);

// Clear
map.clear();

// Immutable (Java 9+)
Map<String, Integer> immutable = Map.of("a", 1, "b", 2);
Map<String, Integer> copy = Map.copyOf(map);
Map.Entry<String, Integer> entry = Map.entry("key", 1);
Map<String, Integer> fromEntries = Map.ofEntries(
    Map.entry("a", 1), Map.entry("b", 2)
);
```

### TreeMap Additional Methods (NavigableMap)
```java
TreeMap<Integer, String> tm = new TreeMap<>();
tm.put(1, "a"); tm.put(3, "c"); tm.put(5, "e"); tm.put(7, "g");

tm.firstKey();           // 1
tm.lastKey();            // 7
tm.firstEntry();         // 1=a
tm.lastEntry();          // 7=g
tm.lowerKey(5);          // 3
tm.floorKey(5);          // 5
tm.higherKey(5);         // 7
tm.ceilingKey(5);        // 5
tm.lowerEntry(5);        // 3=c
tm.pollFirstEntry();     // removes 1=a
tm.pollLastEntry();      // removes 7=g
tm.headMap(5);           // {3=c}
tm.tailMap(5);           // {5=e}
tm.subMap(3, 7);         // {3=c, 5=e}
tm.descendingMap();
tm.descendingKeySet();
tm.navigableKeySet();
```

---

## 4. QUEUE & DEQUE

### PriorityQueue
```java
PriorityQueue<Integer> pq = new PriorityQueue<>(); // min-heap
PriorityQueue<Integer> maxPq = new PriorityQueue<>(Comparator.reverseOrder());

pq.offer(3); pq.offer(1); pq.offer(2);
pq.add(4);           // same as offer but throws on capacity issues
pq.peek();            // 1 (doesn't remove)
pq.poll();            // 1 (removes)
pq.remove(3);         // removes specific element
pq.contains(2);       // true
pq.size();
pq.isEmpty();
pq.toArray();
pq.iterator();
pq.clear();
```

### ArrayDeque (Double-ended queue)
```java
ArrayDeque<String> dq = new ArrayDeque<>();

// Add
dq.addFirst("A");    dq.addLast("B");
dq.offerFirst("C");  dq.offerLast("D");
dq.push("E");        // addFirst

// Access
dq.getFirst();        dq.getLast();        // throws if empty
dq.peekFirst();       dq.peekLast();       // returns null if empty
dq.peek();            dq.element();

// Remove
dq.removeFirst();     dq.removeLast();     // throws if empty
dq.pollFirst();       dq.pollLast();       // returns null if empty
dq.poll();            dq.pop();            // removeFirst

dq.remove("A");
dq.removeFirstOccurrence("A");
dq.removeLastOccurrence("A");

dq.size(); dq.isEmpty(); dq.contains("B");
dq.iterator(); dq.descendingIterator();
```

---

## 5. STACK

```java
Stack<Integer> stack = new Stack<>();
stack.push(1);
stack.push(2);
stack.peek();     // 2 (no remove)
stack.pop();      // 2 (removes)
stack.empty();    // false
stack.search(1);  // 1 (1-based position from top)
stack.size();
stack.contains(1);

// Prefer ArrayDeque as stack:
Deque<Integer> stack2 = new ArrayDeque<>();
stack2.push(1); stack2.pop(); stack2.peek();
```

---

## 6. COLLECTIONS UTILITY CLASS

```java
List<Integer> list = new ArrayList<>(List.of(3, 1, 4, 1, 5));

// Sort
Collections.sort(list);
Collections.sort(list, Comparator.reverseOrder());

// Search
int idx = Collections.binarySearch(list, 4); // list must be sorted

// Reorder
Collections.reverse(list);
Collections.shuffle(list);
Collections.rotate(list, 2);     // rotate right by 2
Collections.swap(list, 0, 1);

// Replace / Fill
Collections.fill(list, 0);
Collections.replaceAll(list, 0, 9);

// Min / Max
Collections.min(list);
Collections.max(list);
Collections.min(list, Comparator.reverseOrder());

// Frequency / Disjoint
Collections.frequency(list, 1);   // count of 1
Collections.disjoint(list1, list2); // true if no common elements

// Unmodifiable
List<Integer> unmod = Collections.unmodifiableList(list);
Set<Integer> unmodSet = Collections.unmodifiableSet(set);
Map<String, Integer> unmodMap = Collections.unmodifiableMap(map);

// Synchronized
List<Integer> syncList = Collections.synchronizedList(list);
Set<Integer> syncSet = Collections.synchronizedSet(set);
Map<String, Integer> syncMap = Collections.synchronizedMap(map);

// Singleton
List<Integer> single = Collections.singletonList(1);
Set<Integer> singleSet = Collections.singleton(1);
Map<String, Integer> singleMap = Collections.singletonMap("a", 1);

// Empty
List<Integer> empty = Collections.emptyList();
Set<Integer> emptySet = Collections.emptySet();
Map<String, Integer> emptyMap = Collections.emptyMap();

// N Copies
List<Integer> copies = Collections.nCopies(5, 0); // [0,0,0,0,0]

// Checked
List<String> checked = Collections.checkedList(new ArrayList<>(), String.class);

// Enumeration
Enumeration<Integer> en = Collections.enumeration(list);
List<Integer> fromEnum = Collections.list(en);
```

---

## EASY QUESTIONS

---

### Q1: Remove Duplicates from a List
```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 2, 3, 3, 4));
List<Integer> unique = new ArrayList<>(new LinkedHashSet<>(list));
// [1, 2, 3, 4]
```

### Q2: Find Common Elements Between Two Lists
```java
List<Integer> a = List.of(1, 2, 3, 4);
List<Integer> b = List.of(3, 4, 5, 6);
List<Integer> common = new ArrayList<>(a);
common.retainAll(b); // [3, 4]
```

### Q3: Sort a Map by Values
```java
Map<String, Integer> map = Map.of("a", 3, "b", 1, "c", 2);
List<Map.Entry<String, Integer>> entries = new ArrayList<>(map.entrySet());
entries.sort(Map.Entry.comparingByValue());
// b=1, c=2, a=3
```

### Q4: Count Word Frequency
```java
String[] words = {"apple", "banana", "apple", "cherry", "banana", "apple"};
Map<String, Integer> freq = new LinkedHashMap<>();
for (String w : words) freq.merge(w, 1, Integer::sum);
// {apple=3, banana=2, cherry=1}
```

### Q5: Convert List to Map
```java
List<String> list = List.of("apple", "banana", "cherry");
Map<String, Integer> map = list.stream()
    .collect(Collectors.toMap(s -> s, String::length));
```

### Q6: Find Maximum Value in Map
```java
Map<String, Integer> map = Map.of("a", 10, "b", 30, "c", 20);
Map.Entry<String, Integer> max = Collections.max(
    map.entrySet(), Map.Entry.comparingByValue());
// b=30
```

### Q7: Iterate Map in Different Ways
```java
// 1. forEach
map.forEach((k, v) -> System.out.println(k + "=" + v));

// 2. entrySet
for (Map.Entry<String, Integer> e : map.entrySet()) {
    System.out.println(e.getKey() + "=" + e.getValue());
}

// 3. keySet
for (String key : map.keySet()) {
    System.out.println(key + "=" + map.get(key));
}

// 4. Stream
map.entrySet().stream().forEach(System.out::println);
```

### Q8: Convert Between Collections
```java
// List → Set
Set<Integer> set = new HashSet<>(list);

// Set → List
List<Integer> list2 = new ArrayList<>(set);

// Array → List
List<Integer> list3 = Arrays.asList(1, 2, 3); // fixed-size
List<Integer> list4 = new ArrayList<>(Arrays.asList(1, 2, 3)); // mutable

// List → Array
Integer[] arr = list.toArray(new Integer[0]);

// Map keys → List
List<String> keys = new ArrayList<>(map.keySet());

// Map values → List
List<Integer> vals = new ArrayList<>(map.values());
```

### Q9: Stack Implementation Using Deque
```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(10);
stack.push(20);
stack.push(30);
System.out.println(stack.peek()); // 30
System.out.println(stack.pop());  // 30
```

### Q10: Queue Implementation
```java
Queue<String> queue = new LinkedList<>();
queue.offer("first");
queue.offer("second");
queue.offer("third");
System.out.println(queue.peek()); // "first"
System.out.println(queue.poll()); // "first"
```

---

## MEDIUM QUESTIONS

---

### Q11: Group Anagrams
```java
public Map<String, List<String>> groupAnagrams(String[] words) {
    Map<String, List<String>> map = new HashMap<>();
    for (String w : words) {
        char[] ch = w.toCharArray();
        Arrays.sort(ch);
        String key = new String(ch);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(w);
    }
    return map;
}
// Input: ["eat","tea","tan","ate","nat","bat"]
// Output: {aet=[eat, tea, ate], ant=[tan, nat], abt=[bat]}
```

### Q12: Find First Non-Repeating Character Using LinkedHashMap
```java
public char firstNonRepeating(String s) {
    Map<Character, Integer> map = new LinkedHashMap<>();
    for (char c : s.toCharArray()) map.merge(c, 1, Integer::sum);
    return map.entrySet().stream()
        .filter(e -> e.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst().orElse('_');
}
```

### Q13: LRU Cache Using LinkedHashMap
```java
class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // accessOrder = true
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}

LRUCache<Integer, String> cache = new LRUCache<>(3);
cache.put(1, "a"); cache.put(2, "b"); cache.put(3, "c");
cache.get(1); // access 1
cache.put(4, "d"); // evicts 2
```

### Q14: Top K Frequent Elements Using PriorityQueue
```java
public List<Integer> topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> freq = new HashMap<>();
    for (int n : nums) freq.merge(n, 1, Integer::sum);
    
    PriorityQueue<Map.Entry<Integer, Integer>> pq =
        new PriorityQueue<>(Comparator.comparingInt(Map.Entry::getValue));
    
    for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
        pq.offer(e);
        if (pq.size() > k) pq.poll();
    }
    
    return pq.stream().map(Map.Entry::getKey).collect(Collectors.toList());
}
```

### Q15: Implement Stack Using Two Queues
```java
class StackUsingQueues<T> {
    private Queue<T> q1 = new LinkedList<>();
    private Queue<T> q2 = new LinkedList<>();
    
    public void push(T item) {
        q2.offer(item);
        while (!q1.isEmpty()) q2.offer(q1.poll());
        Queue<T> tmp = q1; q1 = q2; q2 = tmp;
    }
    
    public T pop()  { return q1.poll(); }
    public T peek() { return q1.peek(); }
    public boolean isEmpty() { return q1.isEmpty(); }
}
```

### Q16: Implement Queue Using Two Stacks
```java
class QueueUsingStacks<T> {
    private Deque<T> inbox = new ArrayDeque<>();
    private Deque<T> outbox = new ArrayDeque<>();
    
    public void enqueue(T item) { inbox.push(item); }
    
    public T dequeue() {
        if (outbox.isEmpty()) {
            while (!inbox.isEmpty()) outbox.push(inbox.pop());
        }
        return outbox.pop();
    }
    
    public T peek() {
        if (outbox.isEmpty()) {
            while (!inbox.isEmpty()) outbox.push(inbox.pop());
        }
        return outbox.peek();
    }
}
```

### Q17: Find All Pairs with Given Sum Using Set
```java
public List<int[]> findPairs(int[] arr, int target) {
    Set<Integer> seen = new HashSet<>();
    List<int[]> result = new ArrayList<>();
    for (int n : arr) {
        int complement = target - n;
        if (seen.contains(complement)) {
            result.add(new int[]{complement, n});
        }
        seen.add(n);
    }
    return result;
}
```

### Q18: Sort Map by Key and Value
```java
// Sort by Key
TreeMap<String, Integer> sortedByKey = new TreeMap<>(map);

// Sort by Value (ascending)
List<Map.Entry<String, Integer>> sorted = map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toList());

// Sort by Value (descending)
map.entrySet().stream()
    .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
    .forEach(System.out::println);

// Collect to LinkedHashMap to preserve order
Map<String, Integer> sortedMap = map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue,
        (e1, e2) -> e1, LinkedHashMap::new));
```

### Q19: Flatten Nested Lists
```java
List<List<Integer>> nested = List.of(
    List.of(1, 2), List.of(3, 4), List.of(5, 6)
);
List<Integer> flat = nested.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
// [1, 2, 3, 4, 5, 6]
```

### Q20: Detect Cycle in a List (Simulated with Collection)
```java
// Using Floyd's with LinkedList of references
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

### Q21: Partition List into Chunks
```java
public <T> List<List<T>> partition(List<T> list, int size) {
    List<List<T>> partitions = new ArrayList<>();
    for (int i = 0; i < list.size(); i += size) {
        partitions.add(new ArrayList<>(
            list.subList(i, Math.min(i + size, list.size()))
        ));
    }
    return partitions;
}
```

### Q22: Custom Object Sorting with Comparable and Comparator
```java
class Employee implements Comparable<Employee> {
    String name;
    int salary;
    
    @Override
    public int compareTo(Employee o) {
        return Integer.compare(this.salary, o.salary);
    }
}

List<Employee> employees = new ArrayList<>();
// Natural order (by salary)
Collections.sort(employees);

// Custom comparators
employees.sort(Comparator.comparing(Employee::getName));
employees.sort(Comparator.comparing(Employee::getSalary).reversed());
employees.sort(Comparator.comparing(Employee::getSalary)
    .thenComparing(Employee::getName));
```

### Q23: Stream Operations on Collections
```java
List<Integer> nums = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// filter + map + collect
List<Integer> evenSquares = nums.stream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .collect(Collectors.toList());

// reduce
int sum = nums.stream().reduce(0, Integer::sum);

// groupingBy
Map<Boolean, List<Integer>> partitioned = nums.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

// counting
Map<Integer, Long> freq = nums.stream()
    .collect(Collectors.groupingBy(n -> n % 3, Collectors.counting()));

// joining
String csv = List.of("a","b","c").stream()
    .collect(Collectors.joining(", ", "[", "]")); // "[a, b, c]"

// toUnmodifiableList (Java 10+)
List<Integer> immutable = nums.stream()
    .collect(Collectors.toUnmodifiableList());
```

### Q24: ConcurrentHashMap Basic Usage
```java
ConcurrentHashMap<String, Integer> cmap = new ConcurrentHashMap<>();
cmap.put("a", 1);
cmap.putIfAbsent("b", 2);
cmap.compute("a", (k, v) -> v + 1);
cmap.merge("a", 10, Integer::sum);

// Bulk operations
cmap.forEach(1, (k, v) -> System.out.println(k + "=" + v));
cmap.search(1, (k, v) -> v > 5 ? k : null);
cmap.reduce(1, (k, v) -> v, Integer::sum);
```

### Q25: EnumSet and EnumMap
```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

EnumSet<Day> weekdays = EnumSet.range(Day.MON, Day.FRI);
EnumSet<Day> weekend = EnumSet.of(Day.SAT, Day.SUN);
EnumSet<Day> all = EnumSet.allOf(Day.class);
EnumSet<Day> none = EnumSet.noneOf(Day.class);

EnumMap<Day, String> schedule = new EnumMap<>(Day.class);
schedule.put(Day.MON, "Work");
schedule.put(Day.SAT, "Rest");
```

---

