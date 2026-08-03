# Java Array - Easy & Medium Questions with Answers (All Methods Covered)

---

## ARRAYS CLASS METHODS REFERENCE

```java
import java.util.Arrays;

int[] arr = {5, 3, 1, 4, 2};

// Sort
Arrays.sort(arr);                          // [1, 2, 3, 4, 5]
Arrays.sort(arr, 0, 3);                   // sort range [0, 3)
String[] sa = {"banana","apple","cherry"};
Arrays.sort(sa);                           // [apple, banana, cherry]
Arrays.sort(sa, Comparator.reverseOrder()); // [cherry, banana, apple]
Arrays.parallelSort(arr);                  // parallel sort (Java 8+)

// Search
Arrays.binarySearch(arr, 3);              // index (array must be sorted)
Arrays.binarySearch(arr, 1, 4, 3);        // search in range

// Fill
Arrays.fill(arr, 0);                      // [0, 0, 0, 0, 0]
Arrays.fill(arr, 1, 3, 9);                // fill range with 9

// Copy
int[] copy = Arrays.copyOf(arr, 10);       // copy with new length
int[] range = Arrays.copyOfRange(arr, 1, 4); // copy range [1, 4)

// Compare
Arrays.equals(arr, copy);                 // element-wise comparison
int[][] a2 = {{1,2},{3,4}};
int[][] b2 = {{1,2},{3,4}};
Arrays.deepEquals(a2, b2);                // true (deep comparison)
Arrays.compare(new int[]{1,2}, new int[]{1,3}); // -1 (Java 9+)
Arrays.mismatch(new int[]{1,2,3}, new int[]{1,2,5}); // 2 (Java 9+)

// Convert
Arrays.toString(arr);                     // "[0, 0, 0, 0, 0]"
Arrays.deepToString(a2);                  // "[[1, 2], [3, 4]]"
List<Integer> list = Arrays.asList(1,2,3); // fixed-size list
Arrays.stream(arr);                        // IntStream

// Hash
Arrays.hashCode(arr);
Arrays.deepHashCode(a2);

// Set All (Java 8+)
Arrays.setAll(arr, i -> i * 2);           // [0, 2, 4, 6, 8]
Arrays.parallelSetAll(arr, i -> i * 3);
Arrays.parallelPrefix(arr, Integer::sum); // running sum
```

---

## EASY QUESTIONS

---

### Q1: Find the Largest Element
```java
public int findMax(int[] arr) {
    int max = arr[0];
    for (int n : arr) max = Math.max(max, n);
    return max;
}
// Using streams:
int max = Arrays.stream(arr).max().getAsInt();
```

### Q2: Find the Smallest Element
```java
public int findMin(int[] arr) {
    return Arrays.stream(arr).min().getAsInt();
}
```

### Q3: Find Sum and Average
```java
int sum = Arrays.stream(arr).sum();
double avg = Arrays.stream(arr).average().getAsDouble();
```

### Q4: Reverse an Array
```java
public void reverse(int[] arr) {
    int l = 0, r = arr.length - 1;
    while (l < r) {
        int tmp = arr[l]; arr[l] = arr[r]; arr[r] = tmp;
        l++; r--;
    }
}
```

### Q5: Check if Array is Sorted
```java
public boolean isSorted(int[] arr) {
    for (int i = 1; i < arr.length; i++) {
        if (arr[i] < arr[i - 1]) return false;
    }
    return true;
}
```

### Q6: Count Occurrences of an Element
```java
public long count(int[] arr, int target) {
    return Arrays.stream(arr).filter(x -> x == target).count();
}
```

### Q7: Find Second Largest
```java
public int secondLargest(int[] arr) {
    int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int n : arr) {
        if (n > first) { second = first; first = n; }
        else if (n > second && n != first) second = n;
    }
    return second;
}
```

### Q8: Remove Duplicates from Sorted Array (In-Place)
```java
public int removeDuplicates(int[] arr) {
    if (arr.length == 0) return 0;
    int j = 0;
    for (int i = 1; i < arr.length; i++) {
        if (arr[i] != arr[j]) arr[++j] = arr[i];
    }
    return j + 1;
}
```

### Q9: Linear Search
```java
public int linearSearch(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) return i;
    }
    return -1;
}
```

### Q10: Copy Array
```java
int[] copy1 = Arrays.copyOf(arr, arr.length);
int[] copy2 = arr.clone();
int[] copy3 = new int[arr.length];
System.arraycopy(arr, 0, copy3, 0, arr.length);
```

### Q11: Merge Two Arrays
```java
public int[] merge(int[] a, int[] b) {
    int[] result = new int[a.length + b.length];
    System.arraycopy(a, 0, result, 0, a.length);
    System.arraycopy(b, 0, result, a.length, b.length);
    return result;
}
```

### Q12: Find Even and Odd Numbers
```java
int[] evens = Arrays.stream(arr).filter(x -> x % 2 == 0).toArray();
int[] odds  = Arrays.stream(arr).filter(x -> x % 2 != 0).toArray();
```

### Q13: Rotate Array Left by K Positions
```java
public void rotateLeft(int[] arr, int k) {
    int n = arr.length;
    k = k % n;
    reverse(arr, 0, k - 1);
    reverse(arr, k, n - 1);
    reverse(arr, 0, n - 1);
}

private void reverse(int[] arr, int l, int r) {
    while (l < r) {
        int tmp = arr[l]; arr[l] = arr[r]; arr[r] = tmp;
        l++; r--;
    }
}
```

### Q14: Find Frequency of Each Element
```java
public Map<Integer, Integer> frequency(int[] arr) {
    Map<Integer, Integer> map = new LinkedHashMap<>();
    for (int n : arr) map.merge(n, 1, Integer::sum);
    return map;
}
```

### Q15: Check if Two Arrays are Equal
```java
boolean equal = Arrays.equals(arr1, arr2);
```

---

## MEDIUM QUESTIONS

---

### Q16: Two Sum (Find pair with given sum)
```java
public int[] twoSum(int[] arr, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < arr.length; i++) {
        int complement = target - arr[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(arr[i], i);
    }
    return new int[]{};
}
```

### Q17: Find Missing Number (1 to N)
```java
public int missingNumber(int[] arr, int n) {
    int expectedSum = n * (n + 1) / 2;
    int actualSum = Arrays.stream(arr).sum();
    return expectedSum - actualSum;
}
```

### Q18: Move All Zeros to End
```java
public void moveZeros(int[] arr) {
    int j = 0;
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] != 0) {
            int tmp = arr[j]; arr[j] = arr[i]; arr[i] = tmp;
            j++;
        }
    }
}
```

### Q19: Find Duplicate Element
```java
// Using HashSet
public int findDuplicate(int[] arr) {
    Set<Integer> seen = new HashSet<>();
    for (int n : arr) {
        if (!seen.add(n)) return n;
    }
    return -1;
}

// Floyd's Cycle Detection (for 1..n range)
public int findDuplicateFloyd(int[] arr) {
    int slow = arr[0], fast = arr[0];
    do {
        slow = arr[slow];
        fast = arr[arr[fast]];
    } while (slow != fast);
    slow = arr[0];
    while (slow != fast) {
        slow = arr[slow];
        fast = arr[fast];
    }
    return slow;
}
```

### Q20: Kadane's Algorithm (Maximum Subarray Sum)
```java
public int maxSubarraySum(int[] arr) {
    int maxSum = arr[0], currentSum = arr[0];
    for (int i = 1; i < arr.length; i++) {
        currentSum = Math.max(arr[i], currentSum + arr[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
```

### Q21: Merge Two Sorted Arrays
```java
public int[] mergeSorted(int[] a, int[] b) {
    int[] result = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length) {
        result[k++] = (a[i] <= b[j]) ? a[i++] : b[j++];
    }
    while (i < a.length) result[k++] = a[i++];
    while (j < b.length) result[k++] = b[j++];
    return result;
}
```

### Q22: Dutch National Flag (Sort 0s, 1s, 2s)
```java
public void sortColors(int[] arr) {
    int low = 0, mid = 0, high = arr.length - 1;
    while (mid <= high) {
        if (arr[mid] == 0) {
            swap(arr, low++, mid++);
        } else if (arr[mid] == 1) {
            mid++;
        } else {
            swap(arr, mid, high--);
        }
    }
}
```

### Q23: Find Intersection of Two Arrays
```java
public int[] intersection(int[] a, int[] b) {
    Set<Integer> setA = Arrays.stream(a).boxed().collect(Collectors.toSet());
    return Arrays.stream(b).filter(setA::contains).distinct().toArray();
}
```

### Q24: Find Union of Two Arrays
```java
public int[] union(int[] a, int[] b) {
    Set<Integer> set = new TreeSet<>();
    for (int n : a) set.add(n);
    for (int n : b) set.add(n);
    return set.stream().mapToInt(Integer::intValue).toArray();
}
```

### Q25: Subarray with Given Sum (Sliding Window)
```java
public int[] subarraySum(int[] arr, int target) {
    int start = 0, sum = 0;
    for (int end = 0; end < arr.length; end++) {
        sum += arr[end];
        while (sum > target && start <= end) sum -= arr[start++];
        if (sum == target) return new int[]{start, end};
    }
    return new int[]{-1, -1};
}
```

### Q26: Product of Array Except Self
```java
public int[] productExceptSelf(int[] arr) {
    int n = arr.length;
    int[] result = new int[n];
    result[0] = 1;
    for (int i = 1; i < n; i++) result[i] = result[i-1] * arr[i-1];
    int right = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= right;
        right *= arr[i];
    }
    return result;
}
```

### Q27: Find Majority Element (appears > n/2 times) — Boyer-Moore
```java
public int majorityElement(int[] arr) {
    int candidate = arr[0], count = 1;
    for (int i = 1; i < arr.length; i++) {
        if (count == 0) { candidate = arr[i]; count = 1; }
        else if (arr[i] == candidate) count++;
        else count--;
    }
    return candidate;
}
```

### Q28: Spiral Traversal of 2D Array
```java
public List<Integer> spiralOrder(int[][] matrix) {
    List<Integer> result = new ArrayList<>();
    int top = 0, bottom = matrix.length - 1;
    int left = 0, right = matrix[0].length - 1;
    while (top <= bottom && left <= right) {
        for (int i = left; i <= right; i++) result.add(matrix[top][i]);
        top++;
        for (int i = top; i <= bottom; i++) result.add(matrix[i][right]);
        right--;
        if (top <= bottom)
            for (int i = right; i >= left; i--) result.add(matrix[bottom][i]);
        bottom--;
        if (left <= right)
            for (int i = bottom; i >= top; i--) result.add(matrix[i][left]);
        left++;
    }
    return result;
}
```

### Q29: Transpose a Matrix
```java
public int[][] transpose(int[][] matrix) {
    int m = matrix.length, n = matrix[0].length;
    int[][] result = new int[n][m];
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            result[j][i] = matrix[i][j];
    return result;
}
```

### Q30: Trapping Rain Water
```java
public int trap(int[] height) {
    int l = 0, r = height.length - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    while (l < r) {
        if (height[l] < height[r]) {
            leftMax = Math.max(leftMax, height[l]);
            water += leftMax - height[l];
            l++;
        } else {
            rightMax = Math.max(rightMax, height[r]);
            water += rightMax - height[r];
            r--;
        }
    }
    return water;
}
```

---

