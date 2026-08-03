# 🏗️ Java Complete Hierarchy — Array, Arrays, String, Character & Collections
### Every Method, Return Type & Flow for Quick Reference

---

## VISUAL HIERARCHY

```
Java Core
│
├── Array (primitive)
│   └── int[], String[], Object[] — fixed size, index-based
│
├── Arrays (utility class — java.util.Arrays)
│   ├── Arrays.sort()
│   ├── Arrays.binarySearch()
│   ├── Arrays.fill()
│   ├── Arrays.copyOf()
│   ├── Arrays.asList()
│   └── Arrays.stream()
│
├── String (immutable — java.lang.String)
│   ├── charAt(), length(), substring()
│   ├── equals(), contains(), startsWith()
│   ├── split(), replace(), trim()
│   ├── toCharArray(), valueOf()
│   └── compareTo(), indexOf()
│
├── StringBuilder (mutable — java.lang.StringBuilder)
│   ├── append(), insert(), delete()
│   ├── reverse(), replace()
│   └── toString()
│
├── Character (wrapper — java.lang.Character)
│   ├── isLetter(), isDigit()
│   ├── isUpperCase(), isLowerCase()
│   ├── toUpperCase(), toLowerCase()
│   └── isWhitespace(), isAlphabetic()
│
└── Collections Framework (java.util)
    ├── List (interface)
    │   ├── ArrayList (dynamic array)
    │   └── LinkedList (doubly linked)
    ├── Set (interface)
    │   ├── HashSet (unordered, no duplicates)
    │   ├── LinkedHashSet (insertion order)
    │   └── TreeSet (sorted)
    ├── Map (interface)
    │   ├── HashMap (unordered key-value)
    │   ├── LinkedHashMap (insertion order)
    │   └── TreeMap (sorted by key)
    ├── Queue (interface)
    │   ├── PriorityQueue (min-heap)
    │   ├── LinkedList (also a Queue)
    │   └── ArrayDeque (double-ended)
    ├── Stack (legacy — use Deque instead)
    └── Collections (utility class)
        ├── Collections.sort()
        ├── Collections.reverse()
        ├── Collections.max(), min()
        └── Collections.frequency()
```

---

# 1️⃣ ARRAY — Primitive Fixed-Size Container

```java
// Declaration + Initialization
int[] arr = new int[5];                      // size 5, default 0
int[] arr = {1, 2, 3, 4, 5};                // literal
String[] names = {"Ram", "John", "Alice"};
int[][] matrix = new int[3][3];              // 2D array
int[][] jagged = {{1,2}, {3,4,5}, {6}};     // jagged array
```

| Operation | Syntax | Return Type | Description |
|-----------|--------|-------------|-------------|
| Access | `arr[0]` | `element type` | Get element at index |
| Assign | `arr[0] = 10` | `void` | Set element at index |
| Length | `arr.length` | `int` | Array size (NOT a method — no parentheses!) |
| Iterate | `for(int x : arr)` | — | Enhanced for-each loop |
| Clone | `arr.clone()` | `int[]` | Shallow copy |

### Flow — Common Array Operations:
```java
// Declare
int[] arr = {5, 3, 1, 4, 2};

// Access
int first = arr[0];              // 5
int last = arr[arr.length - 1];  // 2

// Modify
arr[2] = 99;                     // {5, 3, 99, 4, 2}

// Iterate
for (int i = 0; i < arr.length; i++) { }     // index-based
for (int num : arr) { }                       // for-each

// 2D Array
int[][] grid = new int[3][4];
grid[0][0] = 1;
for (int i = 0; i < grid.length; i++) {           // rows
    for (int j = 0; j < grid[i].length; j++) {    // columns
    }
}
```

### ⚠️ Array Limitations:
- Fixed size — cannot grow/shrink
- No built-in methods (use `Arrays` utility class)
- No `add()`, `remove()`, `contains()` — use `ArrayList` instead

---

# 2️⃣ ARRAYS — Utility Class (java.util.Arrays)

```java
import java.util.Arrays;
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `Arrays.sort(arr)` | `void` | Sort array ascending (in-place) |
| `Arrays.sort(arr, from, to)` | `void` | Sort partial array [from, to) |
| `Arrays.sort(arr, Comparator)` | `void` | Sort with custom comparator |
| `Arrays.parallelSort(arr)` | `void` | Sort using parallel threads |
| `Arrays.binarySearch(arr, key)` | `int` | Find index (array must be sorted!) |
| `Arrays.fill(arr, value)` | `void` | Fill all elements with value |
| `Arrays.fill(arr, from, to, val)` | `void` | Fill range [from, to) |
| `Arrays.copyOf(arr, newLength)` | `T[]` | Copy with new size (truncate/pad) |
| `Arrays.copyOfRange(arr, from, to)` | `T[]` | Copy range [from, to) |
| `Arrays.equals(arr1, arr2)` | `boolean` | Compare two arrays element-by-element |
| `Arrays.deepEquals(arr1, arr2)` | `boolean` | Compare multi-dimensional arrays |
| `Arrays.toString(arr)` | `String` | Convert to string: "[1, 2, 3]" |
| `Arrays.deepToString(arr2D)` | `String` | Convert 2D array to string |
| `Arrays.asList(arr)` | `List<T>` | Convert to fixed-size List (⚠️ can't add/remove) |
| `Arrays.stream(arr)` | `Stream<T>` | Convert to Stream |
| `Arrays.compare(arr1, arr2)` | `int` | Lexicographic comparison |
| `Arrays.mismatch(arr1, arr2)` | `int` | First index where arrays differ (-1 if equal) |

### Flow — Common Usage:
```java
int[] arr = {5, 3, 1, 4, 2};

// Sort
Arrays.sort(arr);                          // {1, 2, 3, 4, 5}

// Sort descending (Integer[] only, not int[])
Integer[] boxed = {5, 3, 1, 4, 2};
Arrays.sort(boxed, Collections.reverseOrder());  // {5, 4, 3, 2, 1}

// Sort partial
Arrays.sort(arr, 1, 4);                    // sort index 1 to 3

// Binary Search (MUST be sorted first!)
Arrays.sort(arr);
int idx = Arrays.binarySearch(arr, 3);     // returns index of 3

// Fill
int[] zeros = new int[10];
Arrays.fill(zeros, -1);                    // all elements = -1

// Copy
int[] copy = Arrays.copyOf(arr, 10);       // copy + extend to size 10
int[] sub = Arrays.copyOfRange(arr, 1, 4); // elements at index 1,2,3

// Compare
boolean same = Arrays.equals(arr1, arr2);

// Print
System.out.println(Arrays.toString(arr));  // [1, 2, 3, 4, 5]

// Convert to List
List<Integer> list = Arrays.asList(1, 2, 3);        // fixed-size list
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));  // mutable list

// Stream operations
int sum = Arrays.stream(arr).sum();
int max = Arrays.stream(arr).max().getAsInt();
int min = Arrays.stream(arr).min().getAsInt();
long count = Arrays.stream(arr).filter(x -> x > 3).count();
```

### Custom Sort with Comparator:
```java
// Sort strings by length
String[] words = {"banana", "cat", "apple", "do"};
Arrays.sort(words, (a, b) -> a.length() - b.length());  // {"do", "cat", "apple", "banana"}

// Sort 2D array by first element
int[][] intervals = {{3,5}, {1,4}, {2,6}};
Arrays.sort(intervals, (a, b) -> a[0] - b[0]);  // {{1,4}, {2,6}, {3,5}}

// Sort by second element descending
Arrays.sort(intervals, (a, b) -> b[1] - a[1]);
```

---

# 3️⃣ STRING — Immutable Character Sequence

```java
String s = "Hello World";
String s = new String("Hello");
```

## 3a. Basic Info

| Method | Return Type | Description |
|--------|-------------|-------------|
| `s.length()` | `int` | Number of characters |
| `s.isEmpty()` | `boolean` | Is length 0? |
| `s.isBlank()` | `boolean` | Is empty or only whitespace? (Java 11+) |
| `s.charAt(int index)` | `char` | Character at index |
| `s.toCharArray()` | `char[]` | Convert to char array |
| `s.getBytes()` | `byte[]` | Convert to byte array |

## 3b. Comparison

| Method | Return Type | Description |
|--------|-------------|-------------|
| `s.equals(Object o)` | `boolean` | Exact equality (case-sensitive) |
| `s.equalsIgnoreCase(String s2)` | `boolean` | Equality ignoring case |
| `s.compareTo(String s2)` | `int` | Lexicographic: <0, 0, >0 |
| `s.compareToIgnoreCase(String s2)` | `int` | Compare ignoring case |
| `s.contentEquals(CharSequence cs)` | `boolean` | Compare with StringBuilder etc. |

### compareTo() logic:
```java
"apple".compareTo("banana")   // negative (a < b)
"banana".compareTo("apple")   // positive (b > a)
"apple".compareTo("apple")    // 0 (equal)
```

## 3c. Searching

| Method | Return Type | Description |
|--------|-------------|-------------|
| `s.contains(CharSequence cs)` | `boolean` | Contains substring? |
| `s.startsWith(String prefix)` | `boolean` | Starts with prefix? |
| `s.endsWith(String suffix)` | `boolean` | Ends with suffix? |
| `s.indexOf(String str)` | `int` | First occurrence index (-1 if not found) |
| `s.indexOf(String str, int from)` | `int` | First occurrence from index |
| `s.indexOf(char ch)` | `int` | First occurrence of char |
| `s.lastIndexOf(String str)` | `int` | Last occurrence index |
| `s.lastIndexOf(char ch)` | `int` | Last occurrence of char |
| `s.matches(String regex)` | `boolean` | Matches entire regex pattern? |

## 3d. Extraction

| Method | Return Type | Description |
|--------|-------------|-------------|
| `s.substring(int begin)` | `String` | From begin to end |
| `s.substring(int begin, int end)` | `String` | From begin to end-1 (exclusive end) |
| `s.subSequence(int begin, int end)` | `CharSequence` | Same as substring |

### Substring logic:
```java
"Hello World".substring(6)      // "World"
"Hello World".substring(0, 5)   // "Hello" (index 0,1,2,3,4)
"Hello World".substring(6, 11)  // "World"
```

## 3e. Modification (returns NEW String — original unchanged!)

| Method | Return Type | Description |
|--------|-------------|-------------|
| `s.toLowerCase()` | `String` | All lowercase |
| `s.toUpperCase()` | `String` | All uppercase |
| `s.trim()` | `String` | Remove leading/trailing whitespace |
| `s.strip()` | `String` | Remove whitespace (Unicode-aware, Java 11+) |
| `s.stripLeading()` | `String` | Remove leading whitespace |
| `s.stripTrailing()` | `String` | Remove trailing whitespace |
| `s.replace(char old, char new)` | `String` | Replace all occurrences of char |
| `s.replace(CharSequence old, CharSequence new)` | `String` | Replace all occurrences of string |
| `s.replaceAll(String regex, String rep)` | `String` | Replace all matching regex |
| `s.replaceFirst(String regex, String rep)` | `String` | Replace first matching regex |
| `s.concat(String s2)` | `String` | Concatenate (prefer + operator) |
| `s.repeat(int count)` | `String` | Repeat N times (Java 11+) |
| `s.indent(int n)` | `String` | Add/remove indentation (Java 12+) |

## 3f. Splitting & Joining

| Method | Return Type | Description |
|--------|-------------|-------------|
| `s.split(String regex)` | `String[]` | Split by regex delimiter |
| `s.split(String regex, int limit)` | `String[]` | Split with max parts |
| `String.join(delim, elements)` | `String` | Join with delimiter |
| `String.join(delim, Iterable)` | `String` | Join collection with delimiter |

### Split/Join examples:
```java
String[] parts = "a,b,c".split(",");          // ["a", "b", "c"]
String[] words = "hello world".split(" ");     // ["hello", "world"]
String[] lines = text.split("\\n");            // split by newline
String[] limited = "a,b,c,d".split(",", 2);   // ["a", "b,c,d"] (max 2 parts)

String joined = String.join("-", "a", "b", "c");        // "a-b-c"
String joined = String.join(", ", listOfStrings);        // "item1, item2, item3"
```

## 3g. Conversion

| Method | Return Type | Description |
|--------|-------------|-------------|
| `String.valueOf(int n)` | `String` | int → String |
| `String.valueOf(char[] arr)` | `String` | char[] → String |
| `String.valueOf(boolean b)` | `String` | boolean → String |
| `String.valueOf(Object o)` | `String` | Any object → String |
| `String.format(format, args)` | `String` | Formatted string (like printf) |
| `Integer.parseInt(s)` | `int` | String → int |
| `Integer.valueOf(s)` | `Integer` | String → Integer object |
| `Double.parseDouble(s)` | `double` | String → double |

### Format examples:
```java
String.format("Name: %s, Age: %d", "Ram", 25);    // "Name: Ram, Age: 25"
String.format("Price: %.2f", 9.99);                 // "Price: 9.99"
String.format("%05d", 42);                          // "00042" (pad with zeros)
```

## 3h. Static Methods

| Method | Return Type | Description |
|--------|-------------|-------------|
| `String.valueOf(x)` | `String` | Convert anything to String |
| `String.join(delim, ...)` | `String` | Join strings |
| `String.format(fmt, ...)` | `String` | Printf-style formatting |
| `String.copyValueOf(char[])` | `String` | char[] to String |

## 3i. Most Used Patterns:
```java
String s = "Hello World";

// Reverse a string
String rev = new StringBuilder(s).reverse().toString();

// Check palindrome
boolean isPalin = s.equals(new StringBuilder(s).reverse().toString());

// Count character occurrences
long count = s.chars().filter(c -> c == 'l').count();

// Remove all spaces
String noSpaces = s.replaceAll("\\s+", "");

// First non-repeating char
char first = s.chars()
    .mapToObj(c -> (char) c)
    .filter(c -> s.indexOf(c) == s.lastIndexOf(c))
    .findFirst().orElse('\0');

// Convert to char frequency map
Map<Character, Integer> freq = new HashMap<>();
for (char c : s.toCharArray()) {
    freq.put(c, freq.getOrDefault(c, 0) + 1);
}

// String → int
int num = Integer.parseInt("123");

// int → String
String str = String.valueOf(123);    // or Integer.toString(123) or "" + 123

// char[] → String
String fromChars = new String(charArray);
String fromChars = String.valueOf(charArray);

// String → char[]
char[] chars = s.toCharArray();
```

---

# 4️⃣ STRINGBUILDER — Mutable String (for modifications)

```java
StringBuilder sb = new StringBuilder();
StringBuilder sb = new StringBuilder("Hello");
StringBuilder sb = new StringBuilder(100);  // initial capacity
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `sb.append(x)` | `StringBuilder` | Add to end (any type: int, char, String) |
| `sb.insert(int offset, x)` | `StringBuilder` | Insert at position |
| `sb.delete(int start, int end)` | `StringBuilder` | Delete range [start, end) |
| `sb.deleteCharAt(int index)` | `StringBuilder` | Delete single char |
| `sb.replace(int start, int end, String str)` | `StringBuilder` | Replace range with string |
| `sb.reverse()` | `StringBuilder` | Reverse entire sequence |
| `sb.charAt(int index)` | `char` | Get char at index |
| `sb.setCharAt(int index, char ch)` | `void` | Set char at index |
| `sb.length()` | `int` | Current length |
| `sb.capacity()` | `int` | Current capacity |
| `sb.indexOf(String str)` | `int` | First occurrence |
| `sb.lastIndexOf(String str)` | `int` | Last occurrence |
| `sb.substring(int start)` | `String` | Substring from start |
| `sb.substring(int start, int end)` | `String` | Substring [start, end) |
| `sb.toString()` | `String` | Convert to String |

### Flow:
```java
StringBuilder sb = new StringBuilder();
sb.append("Hello");          // "Hello"
sb.append(" ");              // "Hello "
sb.append("World");          // "Hello World"
sb.insert(5, ",");           // "Hello, World"
sb.delete(5, 6);             // "Hello World"
sb.reverse();                // "dlroW olleH"
sb.replace(0, 5, "Java");   // "Java olleH"
sb.deleteCharAt(4);          // "JavaolleH"
String result = sb.toString();

// Chain methods (returns itself)
String result = new StringBuilder("Hello")
    .append(" World")
    .reverse()
    .toString();             // "dlroW olleH"
```

### ⚠️ String vs StringBuilder:
```
String:        Immutable. Every modification creates NEW object. O(n) per concat.
StringBuilder: Mutable. Modifies in-place. O(1) amortized append.

Use StringBuilder when:
- Building string in a loop
- Many concatenations (50+)
- Performance matters
```

---

# 5️⃣ CHARACTER — Wrapper & Utility (java.lang.Character)

```java
char ch = 'A';
Character chObj = 'A';  // auto-boxing
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `Character.isLetter(char ch)` | `boolean` | Is a-z or A-Z? |
| `Character.isDigit(char ch)` | `boolean` | Is 0-9? |
| `Character.isLetterOrDigit(char ch)` | `boolean` | Is letter or digit? |
| `Character.isAlphabetic(int cp)` | `boolean` | Is alphabetic (Unicode-aware)? |
| `Character.isUpperCase(char ch)` | `boolean` | Is A-Z? |
| `Character.isLowerCase(char ch)` | `boolean` | Is a-z? |
| `Character.isWhitespace(char ch)` | `boolean` | Is space/tab/newline? |
| `Character.isSpaceChar(char ch)` | `boolean` | Is Unicode space? |
| `Character.toUpperCase(char ch)` | `char` | Convert to uppercase |
| `Character.toLowerCase(char ch)` | `char` | Convert to lowercase |
| `Character.getNumericValue(char ch)` | `int` | Get numeric value ('9' → 9) |
| `Character.forDigit(int digit, int radix)` | `char` | int to char digit |
| `Character.compare(char x, char y)` | `int` | Compare two chars |
| `Character.toString(char ch)` | `String` | char → String |
| `Character.valueOf(char ch)` | `Character` | char → Character object |
| `Character.hashCode(char ch)` | `int` | Hash code of char |

### Char Arithmetic (chars are numbers!):
```java
char ch = 'A';

// Char to int
int ascii = (int) ch;                    // 65
int digit = ch - '0';                    // for '5' → 5
int position = ch - 'a';                 // for 'c' → 2 (0-indexed)

// Int to char
char fromInt = (char) 65;               // 'A'
char fromDigit = (char) ('0' + 5);      // '5'
char fromPos = (char) ('a' + 2);        // 'c'

// Check ranges manually
boolean isUpper = ch >= 'A' && ch <= 'Z';
boolean isLower = ch >= 'a' && ch <= 'z';
boolean isDigit = ch >= '0' && ch <= '9';

// Toggle case
char toggled = Character.isUpperCase(ch) 
    ? Character.toLowerCase(ch) 
    : Character.toUpperCase(ch);
```

### Common Patterns:
```java
// Count vowels
String s = "Hello World";
int vowels = 0;
for (char c : s.toCharArray()) {
    if ("aeiouAEIOU".indexOf(c) != -1) vowels++;
}

// Check if string is all digits
boolean allDigits = s.chars().allMatch(Character::isDigit);

// Check if string is alphanumeric
boolean alphaNum = s.chars().allMatch(Character::isLetterOrDigit);

// Convert char array to frequency array (lowercase letters)
int[] freq = new int[26];
for (char c : s.toCharArray()) {
    if (Character.isLetter(c)) {
        freq[Character.toLowerCase(c) - 'a']++;
    }
}
```

---

# 6️⃣ LIST — Ordered, Indexed, Allows Duplicates

## 6a. ArrayList (Most Used)

```java
import java.util.ArrayList;
import java.util.List;

List<String> list = new ArrayList<>();
List<Integer> nums = new ArrayList<>(Arrays.asList(1, 2, 3));
List<String> list = List.of("a", "b", "c");    // immutable (Java 9+)
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `list.add(E element)` | `boolean` | Add to end |
| `list.add(int index, E element)` | `void` | Insert at index |
| `list.addAll(Collection c)` | `boolean` | Add all from another collection |
| `list.addAll(int index, Collection c)` | `boolean` | Insert all at index |
| `list.get(int index)` | `E` | Get element at index |
| `list.set(int index, E element)` | `E` | Replace element, returns OLD value |
| `list.remove(int index)` | `E` | Remove by index, returns removed |
| `list.remove(Object o)` | `boolean` | Remove first occurrence |
| `list.removeAll(Collection c)` | `boolean` | Remove all that exist in c |
| `list.retainAll(Collection c)` | `boolean` | Keep only elements in c |
| `list.removeIf(Predicate p)` | `boolean` | Remove matching condition |
| `list.clear()` | `void` | Remove all elements |
| `list.size()` | `int` | Number of elements |
| `list.isEmpty()` | `boolean` | Is size 0? |
| `list.contains(Object o)` | `boolean` | Contains element? |
| `list.containsAll(Collection c)` | `boolean` | Contains all? |
| `list.indexOf(Object o)` | `int` | First occurrence (-1 if not found) |
| `list.lastIndexOf(Object o)` | `int` | Last occurrence |
| `list.toArray()` | `Object[]` | Convert to array |
| `list.toArray(T[] arr)` | `T[]` | Convert to typed array |
| `list.subList(int from, int to)` | `List<E>` | View of range [from, to) |
| `list.sort(Comparator c)` | `void` | Sort with comparator |
| `list.iterator()` | `Iterator<E>` | Get iterator |
| `list.listIterator()` | `ListIterator<E>` | Bidirectional iterator |
| `list.stream()` | `Stream<E>` | Convert to Stream |
| `list.forEach(Consumer c)` | `void` | Apply action to each |
| `list.replaceAll(UnaryOperator op)` | `void` | Transform each element |

### Flow:
```java
List<String> list = new ArrayList<>();

// Add
list.add("Apple");                    // [Apple]
list.add("Banana");                   // [Apple, Banana]
list.add(1, "Cherry");                // [Apple, Cherry, Banana]

// Access
String first = list.get(0);           // "Apple"
int size = list.size();               // 3

// Modify
list.set(0, "Avocado");              // [Avocado, Cherry, Banana]

// Remove
list.remove(0);                       // [Cherry, Banana] — by index
list.remove("Banana");                // [Cherry] — by value

// Search
boolean has = list.contains("Cherry"); // true
int idx = list.indexOf("Cherry");      // 0

// Sort
list.sort(Comparator.naturalOrder());          // ascending
list.sort(Comparator.reverseOrder());          // descending
list.sort((a, b) -> a.length() - b.length()); // by length

// Iterate
for (String s : list) { }
list.forEach(System.out::println);
list.forEach(s -> System.out.println(s));

// Remove conditionally
list.removeIf(s -> s.startsWith("A"));

// Convert
String[] arr = list.toArray(new String[0]);
Object[] objArr = list.toArray();

// Sublist (view — changes affect original!)
List<String> sub = list.subList(1, 3);
```

### ⚠️ Integer List — remove() ambiguity:
```java
List<Integer> nums = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
nums.remove(2);              // removes INDEX 2 → removes element "3"
nums.remove(Integer.valueOf(2)); // removes VALUE 2
```

---

## 6b. LinkedList (also implements Deque)

Extra methods over ArrayList:

| Method | Return Type | Description |
|--------|-------------|-------------|
| `list.addFirst(E e)` | `void` | Add to beginning |
| `list.addLast(E e)` | `void` | Add to end |
| `list.getFirst()` | `E` | Get first element |
| `list.getLast()` | `E` | Get last element |
| `list.removeFirst()` | `E` | Remove + return first |
| `list.removeLast()` | `E` | Remove + return last |
| `list.peek()` | `E` | View first (null if empty) |
| `list.poll()` | `E` | Remove + return first (null if empty) |
| `list.offer(E e)` | `boolean` | Add to end |
| `list.push(E e)` | `void` | Add to front (stack behavior) |
| `list.pop()` | `E` | Remove from front (stack behavior) |

---

# 7️⃣ SET — No Duplicates

## 7a. HashSet (Unordered, O(1) operations)

```java
import java.util.HashSet;
import java.util.Set;

Set<String> set = new HashSet<>();
Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 3));
Set<String> set = Set.of("a", "b", "c");    // immutable (Java 9+)
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `set.add(E e)` | `boolean` | Add element (false if already exists) |
| `set.remove(Object o)` | `boolean` | Remove element |
| `set.contains(Object o)` | `boolean` | Contains element? O(1) |
| `set.size()` | `int` | Number of elements |
| `set.isEmpty()` | `boolean` | Is empty? |
| `set.clear()` | `void` | Remove all |
| `set.addAll(Collection c)` | `boolean` | Union |
| `set.retainAll(Collection c)` | `boolean` | Intersection |
| `set.removeAll(Collection c)` | `boolean` | Difference |
| `set.containsAll(Collection c)` | `boolean` | Is subset? |
| `set.toArray()` | `Object[]` | Convert to array |
| `set.stream()` | `Stream<E>` | Convert to Stream |
| `set.forEach(Consumer c)` | `void` | Apply action to each |
| `set.removeIf(Predicate p)` | `boolean` | Remove matching |
| `set.iterator()` | `Iterator<E>` | Get iterator |

### Flow:
```java
Set<Integer> set = new HashSet<>();
set.add(1);                     // [1] — returns true
set.add(2);                     // [1, 2]
set.add(1);                     // [1, 2] — returns false (duplicate!)

set.contains(1);                // true — O(1)
set.remove(2);                  // [1]
set.size();                     // 1

// Set operations
Set<Integer> a = new HashSet<>(Arrays.asList(1, 2, 3));
Set<Integer> b = new HashSet<>(Arrays.asList(2, 3, 4));

// Union: a ∪ b
Set<Integer> union = new HashSet<>(a);
union.addAll(b);                // {1, 2, 3, 4}

// Intersection: a ∩ b
Set<Integer> inter = new HashSet<>(a);
inter.retainAll(b);             // {2, 3}

// Difference: a - b
Set<Integer> diff = new HashSet<>(a);
diff.removeAll(b);              // {1}

// Check duplicates in array
int[] arr = {1, 2, 3, 2, 1};
Set<Integer> seen = new HashSet<>();
for (int n : arr) {
    if (!seen.add(n)) {
        System.out.println(n + " is duplicate");
    }
}
```

## 7b. LinkedHashSet (Maintains Insertion Order)
```java
Set<String> set = new LinkedHashSet<>();  // same methods as HashSet
// Elements iterate in insertion order
```

## 7c. TreeSet (Sorted Order, O(log n))

Extra methods:

| Method | Return Type | Description |
|--------|-------------|-------------|
| `set.first()` | `E` | Smallest element |
| `set.last()` | `E` | Largest element |
| `set.lower(E e)` | `E` | Greatest element strictly less than e |
| `set.higher(E e)` | `E` | Smallest element strictly greater than e |
| `set.floor(E e)` | `E` | Greatest element ≤ e |
| `set.ceiling(E e)` | `E` | Smallest element ≥ e |
| `set.pollFirst()` | `E` | Remove + return smallest |
| `set.pollLast()` | `E` | Remove + return largest |
| `set.headSet(E to)` | `SortedSet<E>` | Elements < to |
| `set.tailSet(E from)` | `SortedSet<E>` | Elements ≥ from |
| `set.subSet(E from, E to)` | `SortedSet<E>` | Elements in [from, to) |
| `set.descendingSet()` | `NavigableSet<E>` | Reverse order view |

```java
TreeSet<Integer> ts = new TreeSet<>(Arrays.asList(5, 1, 3, 7, 2));
// Auto-sorted: [1, 2, 3, 5, 7]

ts.first();       // 1
ts.last();        // 7
ts.lower(5);      // 3 (strictly less)
ts.higher(5);     // 7 (strictly greater)
ts.floor(4);      // 3 (≤ 4)
ts.ceiling(4);    // 5 (≥ 4)
```

---

# 8️⃣ MAP — Key-Value Pairs

## 8a. HashMap (Most Used)

```java
import java.util.HashMap;
import java.util.Map;

Map<String, Integer> map = new HashMap<>();
Map<String, Integer> map = Map.of("a", 1, "b", 2);  // immutable (Java 9+)
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `map.put(K key, V value)` | `V` | Add/update entry, returns OLD value (or null) |
| `map.putIfAbsent(K key, V value)` | `V` | Add only if key doesn't exist |
| `map.get(K key)` | `V` | Get value (null if not found) |
| `map.getOrDefault(K key, V default)` | `V` | Get value or default if not found |
| `map.remove(K key)` | `V` | Remove by key, returns value |
| `map.remove(K key, V value)` | `boolean` | Remove only if key maps to value |
| `map.replace(K key, V value)` | `V` | Replace value if key exists |
| `map.replace(K key, V old, V new)` | `boolean` | Replace only if current value = old |
| `map.containsKey(K key)` | `boolean` | Has this key? |
| `map.containsValue(V value)` | `boolean` | Has this value? |
| `map.size()` | `int` | Number of entries |
| `map.isEmpty()` | `boolean` | Is empty? |
| `map.clear()` | `void` | Remove all entries |
| `map.keySet()` | `Set<K>` | All keys as Set |
| `map.values()` | `Collection<V>` | All values |
| `map.entrySet()` | `Set<Map.Entry<K,V>>` | All key-value pairs |
| `map.merge(K key, V val, BiFunction)` | `V` | Merge with existing value |
| `map.compute(K key, BiFunction)` | `V` | Compute new value from key + old value |
| `map.computeIfAbsent(K key, Function)` | `V` | Compute only if key absent |
| `map.computeIfPresent(K key, BiFunction)` | `V` | Compute only if key present |
| `map.forEach(BiConsumer)` | `void` | Iterate key-value pairs |
| `map.replaceAll(BiFunction)` | `void` | Transform all values |
| `map.putAll(Map m)` | `void` | Copy all from another map |

### Flow:
```java
Map<String, Integer> map = new HashMap<>();

// Add
map.put("apple", 3);           // {apple=3}
map.put("banana", 5);          // {apple=3, banana=5}
map.put("apple", 7);           // {apple=7, banana=5} — overwrites!

// Get
int val = map.get("apple");                    // 7
int val = map.getOrDefault("grape", 0);        // 0 (not found)

// Check
map.containsKey("apple");      // true
map.containsValue(7);          // true

// Remove
map.remove("banana");          // {apple=7}

// Size
map.size();                    // 1

// Iterate
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    int value = entry.getValue();
}

for (String key : map.keySet()) { }
for (int value : map.values()) { }

map.forEach((k, v) -> System.out.println(k + "=" + v));
```

### Most Used Patterns:

```java
// Frequency count
String s = "hello";
Map<Character, Integer> freq = new HashMap<>();
for (char c : s.toCharArray()) {
    freq.put(c, freq.getOrDefault(c, 0) + 1);
}
// {h=1, e=1, l=2, o=1}

// Using merge()
for (char c : s.toCharArray()) {
    freq.merge(c, 1, Integer::sum);
}

// Group elements
Map<Integer, List<String>> grouped = new HashMap<>();
for (String word : words) {
    grouped.computeIfAbsent(word.length(), k -> new ArrayList<>()).add(word);
}

// Two Sum pattern
Map<Integer, Integer> seen = new HashMap<>();  // value → index
for (int i = 0; i < nums.length; i++) {
    int complement = target - nums[i];
    if (seen.containsKey(complement)) {
        return new int[]{seen.get(complement), i};
    }
    seen.put(nums[i], i);
}

// Sort map by value
map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .forEach(e -> System.out.println(e.getKey() + "=" + e.getValue()));

// Sort map by value descending
map.entrySet().stream()
    .sorted(Map.Entry.<String, Integer>comparingByValue().reversed())
    .collect(Collectors.toList());
```

## 8b. LinkedHashMap (Maintains Insertion Order)
```java
Map<String, Integer> map = new LinkedHashMap<>();  // same methods
// Iterates in insertion order
// Use for LRU cache (access order mode)
```

## 8c. TreeMap (Sorted by Key, O(log n))

Extra methods:

| Method | Return Type | Description |
|--------|-------------|-------------|
| `map.firstKey()` | `K` | Smallest key |
| `map.lastKey()` | `K` | Largest key |
| `map.lowerKey(K key)` | `K` | Greatest key strictly < given |
| `map.higherKey(K key)` | `K` | Smallest key strictly > given |
| `map.floorKey(K key)` | `K` | Greatest key ≤ given |
| `map.ceilingKey(K key)` | `K` | Smallest key ≥ given |
| `map.firstEntry()` | `Map.Entry<K,V>` | Smallest entry |
| `map.lastEntry()` | `Map.Entry<K,V>` | Largest entry |
| `map.pollFirstEntry()` | `Map.Entry<K,V>` | Remove + return smallest |
| `map.pollLastEntry()` | `Map.Entry<K,V>` | Remove + return largest |
| `map.headMap(K to)` | `SortedMap<K,V>` | Entries with key < to |
| `map.tailMap(K from)` | `SortedMap<K,V>` | Entries with key ≥ from |
| `map.subMap(K from, K to)` | `SortedMap<K,V>` | Entries in [from, to) |
| `map.descendingMap()` | `NavigableMap<K,V>` | Reverse order view |

---

# 9️⃣ QUEUE & DEQUE — FIFO & Double-Ended

## 9a. Queue (FIFO — First In, First Out)

```java
Queue<Integer> queue = new LinkedList<>();
Queue<Integer> queue = new ArrayDeque<>();   // preferred (faster)
```

| Method | Return Type | Throws Exception? | Description |
|--------|-------------|-------------------|-------------|
| `queue.add(E e)` | `boolean` | ✅ IllegalStateException | Add to tail |
| `queue.offer(E e)` | `boolean` | ❌ returns false | Add to tail (safe) |
| `queue.remove()` | `E` | ✅ NoSuchElementException | Remove from head |
| `queue.poll()` | `E` | ❌ returns null | Remove from head (safe) |
| `queue.element()` | `E` | ✅ NoSuchElementException | View head |
| `queue.peek()` | `E` | ❌ returns null | View head (safe) |
| `queue.size()` | `int` | — | Number of elements |
| `queue.isEmpty()` | `boolean` | — | Is empty? |

### ⚠️ Use `offer/poll/peek` (safe) over `add/remove/element` (throws)

### Flow:
```java
Queue<Integer> queue = new LinkedList<>();
queue.offer(1);         // [1]
queue.offer(2);         // [1, 2]
queue.offer(3);         // [1, 2, 3]
queue.peek();           // 1 (view front, don't remove)
queue.poll();           // 1 (remove front) → queue = [2, 3]
queue.poll();           // 2 → queue = [3]
queue.size();           // 1
```

## 9b. PriorityQueue (Min-Heap by default)

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();                    // min-heap
PriorityQueue<Integer> pq = new PriorityQueue<>(Collections.reverseOrder()); // max-heap
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[0] - b[0]); // custom
```

Same methods as Queue. Elements ordered by natural order or Comparator.

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
minHeap.offer(5);
minHeap.offer(1);
minHeap.offer(3);
minHeap.peek();         // 1 (smallest)
minHeap.poll();         // 1 (removes smallest) → [3, 5]
minHeap.poll();         // 3

// Max-Heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Collections.reverseOrder());
maxHeap.offer(5);
maxHeap.offer(1);
maxHeap.offer(3);
maxHeap.peek();         // 5 (largest)

// Top K elements pattern
PriorityQueue<Integer> minHeap = new PriorityQueue<>();
for (int num : arr) {
    minHeap.offer(num);
    if (minHeap.size() > k) minHeap.poll();  // keep only k largest
}
// minHeap.peek() = kth largest element
```

## 9c. Deque (Double-Ended Queue — Stack + Queue)

```java
Deque<Integer> deque = new ArrayDeque<>();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `deque.offerFirst(E e)` | `boolean` | Add to front |
| `deque.offerLast(E e)` | `boolean` | Add to back |
| `deque.pollFirst()` | `E` | Remove from front |
| `deque.pollLast()` | `E` | Remove from back |
| `deque.peekFirst()` | `E` | View front |
| `deque.peekLast()` | `E` | View back |
| `deque.push(E e)` | `void` | Add to front (Stack behavior) |
| `deque.pop()` | `E` | Remove from front (Stack behavior) |
| `deque.size()` | `int` | Size |
| `deque.isEmpty()` | `boolean` | Is empty? |

### Use Deque as Stack (LIFO):
```java
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);          // [1]
stack.push(2);          // [2, 1]
stack.push(3);          // [3, 2, 1]
stack.peek();           // 3 (top)
stack.pop();            // 3 (remove top) → [2, 1]
stack.pop();            // 2 → [1]
```

### Use Deque as Queue (FIFO):
```java
Deque<Integer> queue = new ArrayDeque<>();
queue.offer(1);         // [1]       (add to back)
queue.offer(2);         // [1, 2]
queue.poll();           // 1         (remove from front)
```

---

# 🔟 STACK (Legacy — use Deque instead)

```java
Stack<Integer> stack = new Stack<>();  // ⚠️ Legacy, prefer ArrayDeque
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `stack.push(E e)` | `E` | Push to top |
| `stack.pop()` | `E` | Remove + return top |
| `stack.peek()` | `E` | View top (don't remove) |
| `stack.isEmpty()` | `boolean` | Is empty? |
| `stack.size()` | `int` | Size |
| `stack.search(Object o)` | `int` | 1-based position from top (-1 if not found) |

---

# 1️⃣1️⃣ COLLECTIONS — Utility Class (java.util.Collections)

```java
import java.util.Collections;
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `Collections.sort(List)` | `void` | Sort ascending |
| `Collections.sort(List, Comparator)` | `void` | Sort with custom comparator |
| `Collections.reverse(List)` | `void` | Reverse list |
| `Collections.shuffle(List)` | `void` | Random shuffle |
| `Collections.swap(List, i, j)` | `void` | Swap elements at i and j |
| `Collections.fill(List, value)` | `void` | Fill all with value |
| `Collections.copy(dest, src)` | `void` | Copy src into dest |
| `Collections.max(Collection)` | `E` | Maximum element |
| `Collections.min(Collection)` | `E` | Minimum element |
| `Collections.frequency(Collection, Object)` | `int` | Count occurrences |
| `Collections.disjoint(Collection, Collection)` | `boolean` | No common elements? |
| `Collections.binarySearch(List, key)` | `int` | Search in sorted list |
| `Collections.nCopies(int n, Object o)` | `List<T>` | List with n copies of o |
| `Collections.singleton(T o)` | `Set<T>` | Immutable single-element set |
| `Collections.singletonList(T o)` | `List<T>` | Immutable single-element list |
| `Collections.emptyList()` | `List<T>` | Immutable empty list |
| `Collections.emptySet()` | `Set<T>` | Immutable empty set |
| `Collections.emptyMap()` | `Map<K,V>` | Immutable empty map |
| `Collections.unmodifiableList(List)` | `List<T>` | Read-only view |
| `Collections.unmodifiableSet(Set)` | `Set<T>` | Read-only view |
| `Collections.unmodifiableMap(Map)` | `Map<K,V>` | Read-only view |
| `Collections.synchronizedList(List)` | `List<T>` | Thread-safe wrapper |
| `Collections.reverseOrder()` | `Comparator<T>` | Descending comparator |
| `Collections.reverseOrder(Comparator)` | `Comparator<T>` | Reverse given comparator |
| `Collections.rotate(List, int distance)` | `void` | Rotate elements by distance |
| `Collections.replaceAll(List, old, new)` | `boolean` | Replace all occurrences |

### Flow:
```java
List<Integer> list = new ArrayList<>(Arrays.asList(5, 3, 1, 4, 2));

Collections.sort(list);                    // [1, 2, 3, 4, 5]
Collections.reverse(list);                 // [5, 4, 3, 2, 1]
Collections.shuffle(list);                 // random order
int max = Collections.max(list);           // 5
int min = Collections.min(list);           // 1
int count = Collections.frequency(list, 3); // how many 3s
Collections.swap(list, 0, 4);             // swap first and last
Collections.fill(list, 0);                // [0, 0, 0, 0, 0]
```

---

# 1️⃣2️⃣ ITERATOR — Traverse Collections

```java
Iterator<String> it = list.iterator();
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `it.hasNext()` | `boolean` | More elements? |
| `it.next()` | `E` | Get next element |
| `it.remove()` | `void` | Remove current element (safe during iteration) |

### Flow:
```java
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c", "d"));

// Remove during iteration (ConcurrentModification-safe)
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    if (s.equals("b")) {
        it.remove();  // safe! (unlike list.remove() in for-each)
    }
}
```

---

# 1️⃣3️⃣ STREAM API — Functional Operations (Java 8+)

```java
import java.util.stream.*;
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| **Creating** | | |
| `list.stream()` | `Stream<T>` | From collection |
| `Arrays.stream(arr)` | `Stream<T>` | From array |
| `Stream.of(a, b, c)` | `Stream<T>` | From values |
| `IntStream.range(0, 10)` | `IntStream` | 0 to 9 |
| `IntStream.rangeClosed(1, 10)` | `IntStream` | 1 to 10 |
| **Intermediate (return Stream)** | | |
| `.filter(Predicate)` | `Stream<T>` | Keep matching elements |
| `.map(Function)` | `Stream<R>` | Transform each element |
| `.mapToInt(ToIntFunction)` | `IntStream` | Map to int stream |
| `.flatMap(Function)` | `Stream<R>` | Flatten nested streams |
| `.distinct()` | `Stream<T>` | Remove duplicates |
| `.sorted()` | `Stream<T>` | Sort natural order |
| `.sorted(Comparator)` | `Stream<T>` | Sort with comparator |
| `.limit(long n)` | `Stream<T>` | Take first n elements |
| `.skip(long n)` | `Stream<T>` | Skip first n elements |
| `.peek(Consumer)` | `Stream<T>` | Debug — view without modifying |
| **Terminal (produce result)** | | |
| `.forEach(Consumer)` | `void` | Apply action to each |
| `.collect(Collector)` | `R` | Collect to List/Set/Map |
| `.toList()` | `List<T>` | Collect to list (Java 16+) |
| `.count()` | `long` | Count elements |
| `.sum()` | `int/long/double` | Sum (IntStream/LongStream) |
| `.min()` | `Optional<T>` | Minimum |
| `.max()` | `Optional<T>` | Maximum |
| `.average()` | `OptionalDouble` | Average (IntStream) |
| `.reduce(identity, BinaryOperator)` | `T` | Reduce to single value |
| `.findFirst()` | `Optional<T>` | First element |
| `.findAny()` | `Optional<T>` | Any element |
| `.anyMatch(Predicate)` | `boolean` | Any element matches? |
| `.allMatch(Predicate)` | `boolean` | All elements match? |
| `.noneMatch(Predicate)` | `boolean` | No element matches? |
| `.toArray()` | `Object[]` | Convert to array |

### Flow — Common Patterns:
```java
List<Integer> nums = Arrays.asList(5, 3, 1, 4, 2, 3, 5);

// Filter + Collect
List<Integer> evens = nums.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());          // [4, 2]

// Map + Collect
List<String> strings = nums.stream()
    .map(n -> "Item" + n)
    .collect(Collectors.toList());          // ["Item5", "Item3", ...]

// Sum
int sum = nums.stream().mapToInt(Integer::intValue).sum();   // 23

// Max/Min
int max = nums.stream().mapToInt(Integer::intValue).max().getAsInt();
int min = Collections.min(nums);

// Distinct + Sort
List<Integer> unique = nums.stream()
    .distinct()
    .sorted()
    .collect(Collectors.toList());          // [1, 2, 3, 4, 5]

// Count with condition
long count = nums.stream().filter(n -> n > 3).count();  // 3

// Find first matching
Optional<Integer> first = nums.stream().filter(n -> n > 3).findFirst();

// Reduce (product)
int product = nums.stream().reduce(1, (a, b) -> a * b);

// Join strings
String joined = list.stream().collect(Collectors.joining(", "));  // "a, b, c"

// Group by
Map<Integer, List<String>> grouped = words.stream()
    .collect(Collectors.groupingBy(String::length));

// Frequency map
Map<Integer, Long> freq = nums.stream()
    .collect(Collectors.groupingBy(n -> n, Collectors.counting()));

// Partition (split into two groups)
Map<Boolean, List<Integer>> parts = nums.stream()
    .collect(Collectors.partitioningBy(n -> n > 3));

// String char stream
"hello".chars()                            // IntStream of char codes
    .mapToObj(c -> (char) c)               // Stream<Character>
    .filter(Character::isLetter)
    .collect(Collectors.toList());

// Array to stream and back
int[] arr = {1, 2, 3, 4, 5};
int[] filtered = Arrays.stream(arr).filter(n -> n > 2).toArray();  // [3, 4, 5]
```

---

# 1️⃣4️⃣ COMPARATOR — Custom Sorting

```java
import java.util.Comparator;
```

| Method | Return Type | Description |
|--------|-------------|-------------|
| `Comparator.naturalOrder()` | `Comparator<T>` | Ascending (a, b, c / 1, 2, 3) |
| `Comparator.reverseOrder()` | `Comparator<T>` | Descending |
| `Comparator.comparing(Function)` | `Comparator<T>` | Compare by extracted key |
| `Comparator.comparingInt(ToIntFunction)` | `Comparator<T>` | Compare by int key |
| `comparator.reversed()` | `Comparator<T>` | Reverse this comparator |
| `comparator.thenComparing(Function)` | `Comparator<T>` | Secondary sort |
| `Comparator.nullsFirst(Comparator)` | `Comparator<T>` | Nulls go first |
| `Comparator.nullsLast(Comparator)` | `Comparator<T>` | Nulls go last |

### Common Sort Patterns:
```java
List<String> words = Arrays.asList("banana", "cat", "apple", "do");

// Sort by length
words.sort(Comparator.comparingInt(String::length));         // [do, cat, apple, banana]

// Sort by length descending
words.sort(Comparator.comparingInt(String::length).reversed());

// Sort by length, then alphabetically
words.sort(Comparator.comparingInt(String::length)
    .thenComparing(Comparator.naturalOrder()));

// Lambda comparator
words.sort((a, b) -> a.length() - b.length());              // ascending by length
words.sort((a, b) -> b.length() - a.length());              // descending by length

// Sort objects by field
List<Person> people = ...;
people.sort(Comparator.comparing(Person::getName));
people.sort(Comparator.comparingInt(Person::getAge).reversed());
people.sort(Comparator.comparing(Person::getCity)
    .thenComparingInt(Person::getAge));
```

---

# 1️⃣5️⃣ CONVERSION CHEAT SHEET

## Array ↔ List

```java
// Array → List
int[] arr = {1, 2, 3};
List<Integer> list = Arrays.stream(arr).boxed().collect(Collectors.toList());
// OR
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3));  // for Integer[]

String[] strArr = {"a", "b", "c"};
List<String> list = new ArrayList<>(Arrays.asList(strArr));

// List → Array
Integer[] arr = list.toArray(new Integer[0]);
int[] primitiveArr = list.stream().mapToInt(Integer::intValue).toArray();
String[] strArr = list.toArray(new String[0]);
```

## Array ↔ Set

```java
// Array → Set
Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 3));
Set<Integer> set = Arrays.stream(arr).boxed().collect(Collectors.toSet());

// Set → Array
Integer[] arr = set.toArray(new Integer[0]);
```

## List ↔ Set

```java
// List → Set (removes duplicates)
Set<Integer> set = new HashSet<>(list);

// Set → List
List<Integer> list = new ArrayList<>(set);
```

## String ↔ char[]

```java
// String → char[]
char[] chars = str.toCharArray();

// char[] → String
String str = new String(chars);
String str = String.valueOf(chars);
```

## String ↔ int

```java
// String → int
int num = Integer.parseInt("123");

// int → String
String str = String.valueOf(123);
String str = Integer.toString(123);
String str = "" + 123;
```

## int[] ↔ Integer[]

```java
// int[] → Integer[]
Integer[] boxed = Arrays.stream(arr).boxed().toArray(Integer[]::new);

// Integer[] → int[]
int[] primitive = Arrays.stream(boxed).mapToInt(Integer::intValue).toArray();
```

## String → List<Character>

```java
List<Character> chars = str.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.toList());
```

## Map → List of entries sorted by value

```java
List<Map.Entry<String, Integer>> sorted = map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toList());
```

---

# 1️⃣6️⃣ QUICK REFERENCE — When to Use What

| Need | Use | Why |
|------|-----|-----|
| Fixed size, fast access | `int[]` / `String[]` | O(1) access, no overhead |
| Dynamic size, indexed access | `ArrayList` | O(1) get, amortized O(1) add |
| Frequent add/remove at ends | `LinkedList` / `ArrayDeque` | O(1) add/remove at head/tail |
| No duplicates, fast lookup | `HashSet` | O(1) add/contains/remove |
| No duplicates, sorted | `TreeSet` | O(log n), auto-sorted |
| No duplicates, insertion order | `LinkedHashSet` | O(1) + maintains order |
| Key-value, fast lookup | `HashMap` | O(1) put/get/containsKey |
| Key-value, sorted keys | `TreeMap` | O(log n), sorted by key |
| Key-value, insertion order | `LinkedHashMap` | O(1) + maintains order |
| FIFO queue | `ArrayDeque` (as Queue) | O(1) offer/poll |
| LIFO stack | `ArrayDeque` (as Stack) | O(1) push/pop |
| Priority ordering | `PriorityQueue` | O(log n) offer/poll, O(1) peek |
| String building in loop | `StringBuilder` | O(1) amortized append |
| Immutable text | `String` | Thread-safe, interning |
| Character checking | `Character` static methods | Utility |
| Array operations | `Arrays` static methods | Sort, search, copy |
| Collection operations | `Collections` static methods | Sort, reverse, max, min |

---

# 1️⃣7️⃣ TIME COMPLEXITY CHEAT SHEET

| Data Structure | Access | Search | Insert | Delete |
|----------------|--------|--------|--------|--------|
| Array | O(1) | O(n) | O(n) | O(n) |
| ArrayList | O(1) | O(n) | O(1)* | O(n) |
| LinkedList | O(n) | O(n) | O(1)** | O(1)** |
| HashSet | — | O(1) | O(1) | O(1) |
| TreeSet | — | O(log n) | O(log n) | O(log n) |
| HashMap | — | O(1) | O(1) | O(1) |
| TreeMap | — | O(log n) | O(log n) | O(log n) |
| PriorityQueue | O(1)*** | O(n) | O(log n) | O(log n) |
| Stack/Deque | O(1)**** | O(n) | O(1) | O(1) |

```
* ArrayList: O(1) amortized add to end, O(n) add to middle
** LinkedList: O(1) if you have the node, O(n) to find it first
*** PriorityQueue: O(1) peek (min/max), O(n) to find arbitrary element
**** Stack/Deque: O(1) top/front only
```

---

# 1️⃣8️⃣ INTERVIEW PATTERNS — Which Collection to Use

| Pattern | Data Structure | Example |
|---------|---------------|---------|
| Two Sum | `HashMap<Integer, Integer>` | value → index |
| Frequency count | `HashMap<Character, Integer>` | char → count |
| Unique elements | `HashSet<Integer>` | seen.add(n) returns false = duplicate |
| Sliding window | `HashMap` or `int[26]` | frequency in window |
| Stack problems | `ArrayDeque<Integer>` | brackets, monotonic stack |
| BFS | `Queue<int[]>` (ArrayDeque) | level-order, shortest path |
| Top K elements | `PriorityQueue` | min-heap of size k |
| Sorted insertion | `TreeMap` / `TreeSet` | floor/ceiling queries |
| Ordered iteration | `LinkedHashMap` | LRU cache |
| Graph adjacency | `Map<Integer, List<Integer>>` | node → neighbors |
| Anagram grouping | `Map<String, List<String>>` | sorted key → anagram list |
| Interval merging | `int[][]` sorted + merge | sort by start |
| Matrix traversal | `boolean[][]` visited | DFS/BFS on grid |
| String building | `StringBuilder` | reverse, build in loop |
| Palindrome check | Two pointers on `char[]` | compare i and n-1-i |

---

*Complete Java hierarchy with every method, return type, and flow. Use this as your daily coding reference.*
*Last updated: April 30, 2026*

