# Java String - Easy & Medium Questions with Answers (All Methods Covered)

---

## EASY QUESTIONS

---

### Q1: How do you find the length of a string?
```java
String s = "Hello World";
System.out.println(s.length()); // 11
```

### Q2: How do you get a character at a specific index?
```java
String s = "Hello";
char ch = s.charAt(1); // 'e'
// Throws StringIndexOutOfBoundsException if index < 0 or >= length()
```

### Q3: How do you convert a string to char array?
```java
String s = "Hello";
char[] arr = s.toCharArray(); // ['H','e','l','l','o']
```

### Q4: How do you compare two strings?
```java
String a = "hello";
String b = "Hello";

a.equals(b);             // false (case-sensitive)
a.equalsIgnoreCase(b);   // true
a.compareTo(b);          // positive (lexicographic comparison)
a.compareToIgnoreCase(b); // 0
```

### Q5: How do you check if a string is empty or blank?
```java
String s1 = "";
String s2 = "   ";

s1.isEmpty();  // true (length == 0)
s2.isBlank();  // true (only whitespace, Java 11+)
```

### Q6: How do you convert case?
```java
String s = "Hello World";
s.toUpperCase(); // "HELLO WORLD"
s.toLowerCase(); // "hello world"
```

### Q7: How do you trim whitespace?
```java
String s = "  Hello  ";
s.trim();  // "Hello" (removes leading/trailing spaces <= U+0020)
s.strip(); // "Hello" (Unicode-aware, Java 11+)
s.stripLeading();  // "Hello  "
s.stripTrailing(); // "  Hello"
```

### Q8: How do you check if a string contains a substring?
```java
String s = "Hello World";
s.contains("World"); // true
```

### Q9: How do you find the index of a character or substring?
```java
String s = "Hello World Hello";
s.indexOf('o');          // 4
s.indexOf("World");      // 6
s.indexOf('o', 5);       // 7 (search from index 5)
s.lastIndexOf('o');      // 16
s.lastIndexOf("Hello");  // 12
```

### Q10: How do you check prefix and suffix?
```java
String s = "HelloWorld";
s.startsWith("Hello");    // true
s.startsWith("World", 5); // true (from offset 5)
s.endsWith("World");      // true
```

### Q11: How do you extract a substring?
```java
String s = "Hello World";
s.substring(6);     // "World"
s.substring(0, 5);  // "Hello" (endIndex exclusive)
```

### Q12: How do you replace characters/substrings?
```java
String s = "Hello World";
s.replace('l', 'r');         // "Herro Worrd"
s.replace("World", "Java");  // "Hello Java"
s.replaceFirst("l", "L");    // "HeLlo World" (regex-based)
s.replaceAll("l", "L");      // "HeLLo WorLd" (regex-based)
```

### Q13: How do you split a string?
```java
String s = "a,b,c,d";
String[] parts = s.split(",");    // ["a","b","c","d"]
String[] parts2 = s.split(",", 2); // ["a","b,c,d"] (limit)
```

### Q14: How do you join strings?
```java
String result = String.join("-", "a", "b", "c"); // "a-b-c"
String result2 = String.join(",", List.of("x","y")); // "x,y"
```

### Q15: How do you concatenate strings?
```java
String s = "Hello";
s.concat(" World"); // "Hello World"
// Also: s + " World" or String.format("%s %s", "Hello", "World")
```

### Q16: How do you convert other types to String?
```java
String.valueOf(123);       // "123"
String.valueOf(true);      // "true"
String.valueOf('c');       // "c"
String.valueOf(new char[]{'a','b'}); // "ab"
Integer.toString(123);     // "123"
```

### Q17: How do you use `format` and `formatted`?
```java
String s = String.format("Name: %s, Age: %d", "John", 25);
// "Name: John, Age: 25"

// Java 15+
String s2 = "Name: %s".formatted("John"); // "Name: John"
```

### Q18: How do you convert String to bytes and back?
```java
String s = "Hello";
byte[] bytes = s.getBytes();                    // default charset
byte[] utf8 = s.getBytes(StandardCharsets.UTF_8);
String back = new String(bytes);
String back2 = new String(utf8, StandardCharsets.UTF_8);
```

### Q19: How do you use `intern()`?
```java
String s1 = new String("hello");
String s2 = s1.intern(); // returns reference from string pool
String s3 = "hello";
System.out.println(s2 == s3); // true
```

### Q20: How do you check with `matches()` (regex)?
```java
String s = "hello123";
s.matches("[a-z]+\\d+"); // true
```

### Q21: How do you use `getChars()` and `codePointAt()`?
```java
String s = "Hello";
char[] dest = new char[3];
s.getChars(0, 3, dest, 0); // dest = ['H','e','l']

int cp = s.codePointAt(0); // 72 (Unicode code point of 'H')
int cpBefore = s.codePointBefore(1); // 72
int cpCount = s.codePointCount(0, 5); // 5
```

### Q22: How do you use `chars()` and `codePoints()` streams? (Java 9+)
```java
"Hello".chars().forEach(c -> System.out.print((char) c + " "));
// H e l l o

"Hello".codePoints().forEach(System.out::println);
```

### Q23: How do you repeat a string? (Java 11+)
```java
String s = "ab".repeat(3); // "ababab"
```

### Q24: How do you use `contentEquals()`?
```java
String s = "Hello";
s.contentEquals(new StringBuilder("Hello")); // true
s.contentEquals(new StringBuffer("Hello"));  // true
```

### Q25: How do you use `regionMatches()`?
```java
String s = "Hello World";
s.regionMatches(6, "World", 0, 5);       // true
s.regionMatches(true, 0, "HELLO", 0, 5); // true (ignoreCase=true)
```

### Q26: How to use `toCharArray()` + `copyValueOf()`?
```java
char[] arr = {'J','a','v','a'};
String s = String.copyValueOf(arr);        // "Java"
String s2 = String.copyValueOf(arr, 0, 2); // "Ja"
```

### Q27: How do you use `subSequence()`?
```java
String s = "Hello";
CharSequence cs = s.subSequence(1, 4); // "ell"
```

### Q28: `hashCode()` and `toString()`
```java
String s = "Hello";
s.hashCode();  // integer hash
s.toString();  // returns itself
```

### Q29: How do you use `lines()`, `indent()`, `translateEscapes()`? (Java 11-15+)
```java
"line1\nline2\nline3".lines().forEach(System.out::println);
// line1
// line2
// line3

"Hello".indent(4); // "    Hello\n"

"Hello\\nWorld".translateEscapes(); // "Hello\nWorld" (Java 15+)
```

---

## MEDIUM QUESTIONS

---

### Q30: Reverse a String
```java
public String reverse(String s) {
    return new StringBuilder(s).reverse().toString();
}

// Without built-in:
public String reverseManual(String s) {
    char[] arr = s.toCharArray();
    int l = 0, r = arr.length - 1;
    while (l < r) {
        char tmp = arr[l];
        arr[l++] = arr[r];
        arr[r--] = tmp;
    }
    return new String(arr);
}
```

### Q31: Check if a String is a Palindrome
```java
public boolean isPalindrome(String s) {
    s = s.toLowerCase().replaceAll("[^a-z0-9]", "");
    int l = 0, r = s.length() - 1;
    while (l < r) {
        if (s.charAt(l++) != s.charAt(r--)) return false;
    }
    return true;
}
```

### Q32: Count Character Occurrences
```java
public Map<Character, Integer> charCount(String s) {
    Map<Character, Integer> map = new LinkedHashMap<>();
    for (char c : s.toCharArray()) {
        map.merge(c, 1, Integer::sum);
    }
    return map;
}
```

### Q33: Find First Non-Repeating Character
```java
public char firstUnique(String s) {
    Map<Character, Integer> map = new LinkedHashMap<>();
    for (char c : s.toCharArray()) map.merge(c, 1, Integer::sum);
    for (Map.Entry<Character, Integer> e : map.entrySet()) {
        if (e.getValue() == 1) return e.getKey();
    }
    return '_';
}
```

### Q34: Check if Two Strings are Anagrams
```java
public boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    char[] c1 = a.toLowerCase().toCharArray();
    char[] c2 = b.toLowerCase().toCharArray();
    Arrays.sort(c1);
    Arrays.sort(c2);
    return Arrays.equals(c1, c2);
}

// Optimized with frequency array:
public boolean isAnagramOpt(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] freq = new int[26];
    for (int i = 0; i < a.length(); i++) {
        freq[a.charAt(i) - 'a']++;
        freq[b.charAt(i) - 'a']--;
    }
    for (int f : freq) if (f != 0) return false;
    return true;
}
```

### Q35: Find All Permutations of a String
```java
public List<String> permutations(String s) {
    List<String> result = new ArrayList<>();
    permute(s.toCharArray(), 0, result);
    return result;
}

private void permute(char[] arr, int idx, List<String> result) {
    if (idx == arr.length - 1) {
        result.add(new String(arr));
        return;
    }
    for (int i = idx; i < arr.length; i++) {
        char tmp = arr[idx]; arr[idx] = arr[i]; arr[i] = tmp;
        permute(arr, idx + 1, result);
        tmp = arr[idx]; arr[idx] = arr[i]; arr[i] = tmp; // backtrack
    }
}
```

### Q36: Find Longest Substring Without Repeating Characters
```java
public int longestUnique(String s) {
    Map<Character, Integer> map = new HashMap<>();
    int max = 0, start = 0;
    for (int end = 0; end < s.length(); end++) {
        char c = s.charAt(end);
        if (map.containsKey(c)) {
            start = Math.max(start, map.get(c) + 1);
        }
        map.put(c, end);
        max = Math.max(max, end - start + 1);
    }
    return max;
}
```

### Q37: Reverse Words in a String
```java
public String reverseWords(String s) {
    String[] words = s.trim().split("\\s+");
    StringBuilder sb = new StringBuilder();
    for (int i = words.length - 1; i >= 0; i--) {
        sb.append(words[i]);
        if (i > 0) sb.append(" ");
    }
    return sb.toString();
}
```

### Q38: String Compression (aabcccaaa → a2b1c3a3)
```java
public String compress(String s) {
    StringBuilder sb = new StringBuilder();
    int i = 0;
    while (i < s.length()) {
        char c = s.charAt(i);
        int count = 0;
        while (i < s.length() && s.charAt(i) == c) { i++; count++; }
        sb.append(c).append(count);
    }
    return sb.length() < s.length() ? sb.toString() : s;
}
```

### Q39: Check if String is Rotation of Another
```java
public boolean isRotation(String s1, String s2) {
    return s1.length() == s2.length() && (s1 + s1).contains(s2);
}
```

### Q40: Longest Common Prefix
```java
public String longestCommonPrefix(String[] strs) {
    if (strs.length == 0) return "";
    String prefix = strs[0];
    for (int i = 1; i < strs.length; i++) {
        while (!strs[i].startsWith(prefix)) {
            prefix = prefix.substring(0, prefix.length() - 1);
        }
    }
    return prefix;
}
```

### Q41: Remove Duplicates from String (Preserve Order)
```java
public String removeDuplicates(String s) {
    LinkedHashSet<Character> set = new LinkedHashSet<>();
    for (char c : s.toCharArray()) set.add(c);
    StringBuilder sb = new StringBuilder();
    set.forEach(sb::append);
    return sb.toString();
}
```

### Q42: Count Vowels and Consonants
```java
public void countVowelsConsonants(String s) {
    long vowels = s.toLowerCase().chars()
        .filter(c -> "aeiou".indexOf(c) != -1).count();
    long consonants = s.toLowerCase().chars()
        .filter(c -> c >= 'a' && c <= 'z' && "aeiou".indexOf(c) == -1).count();
    System.out.println("Vowels: " + vowels + ", Consonants: " + consonants);
}
```

### Q43: Convert String to Integer (Custom atoi)
```java
public int myAtoi(String s) {
    s = s.trim();
    if (s.isEmpty()) return 0;
    int sign = 1, i = 0;
    if (s.charAt(0) == '-') { sign = -1; i++; }
    else if (s.charAt(0) == '+') i++;
    long result = 0;
    while (i < s.length() && Character.isDigit(s.charAt(i))) {
        result = result * 10 + (s.charAt(i++) - '0');
        if (result * sign > Integer.MAX_VALUE) return Integer.MAX_VALUE;
        if (result * sign < Integer.MIN_VALUE) return Integer.MIN_VALUE;
    }
    return (int) (result * sign);
}
```

### Q44: Find All Substrings of a String
```java
public List<String> allSubstrings(String s) {
    List<String> result = new ArrayList<>();
    for (int i = 0; i < s.length(); i++) {
        for (int j = i + 1; j <= s.length(); j++) {
            result.add(s.substring(i, j));
        }
    }
    return result;
}
```

### Q45: Longest Palindromic Substring
```java
public String longestPalindrome(String s) {
    String result = "";
    for (int i = 0; i < s.length(); i++) {
        String odd = expand(s, i, i);
        String even = expand(s, i, i + 1);
        if (odd.length() > result.length()) result = odd;
        if (even.length() > result.length()) result = even;
    }
    return result;
}

private String expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
    return s.substring(l + 1, r);
}
```

---

## StringBuilder / StringBuffer Methods Quick Reference

```java
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");       // "Hello World"
sb.insert(5, ",");         // "Hello, World"
sb.delete(5, 6);           // "Hello World"
sb.deleteCharAt(4);        // "Hell World"
sb.replace(0, 4, "Hey");   // "Hey World"
sb.reverse();              // "dlroW yeH"
sb.charAt(0);              // 'd'
sb.setCharAt(0, 'D');      // "DlroW yeH"
sb.length();               // 9
sb.capacity();             // initial 16 + original length
sb.ensureCapacity(50);
sb.trimToSize();
sb.substring(0, 4);        // "Dlro"
sb.indexOf("roW");         // 2
sb.lastIndexOf("W");       // 4
sb.toString();              // convert to String
```

> **StringBuffer** has the same methods but is **synchronized** (thread-safe).

---

