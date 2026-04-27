# 📘 DSA (Data Structures & Algorithms) — Complete Guide for SDET
### Ram Girhe | 320+ LeetCode Problems | Target: 3-5 YOE Roles

---

## 🎯 What Interviewers Expect at 3 YOE

- **Easy problems**: Solve in 10-15 minutes with clean code
- **Medium problems**: Solve in 20-30 minutes, explain approach clearly
- **Hard problems**: Not usually expected for QA/SDET, but know concepts
- **Always explain**: Time complexity, Space complexity, Edge cases

---

## 1. Arrays & Strings (Most Asked — 40% of DSA rounds)

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
// Time: O(n), Space: O(1)
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
// Time: O(n), Space: O(1)
```

### Pattern: Prefix Sum
```java
// Product of Array Except Self
public int[] productExceptSelf(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    result[0] = 1;
    for (int i = 1; i < n; i++)
        result[i] = result[i - 1] * nums[i - 1];
    int suffix = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= suffix;
        suffix *= nums[i];
    }
    return result;
}
// Time: O(n), Space: O(1) excluding output
```

### Pattern: Kadane's Algorithm
```java
// Maximum Subarray Sum
public int maxSubArray(int[] nums) {
    int maxSum = nums[0], currentSum = nums[0];
    for (int i = 1; i < nums.length; i++) {
        currentSum = Math.max(nums[i], currentSum + nums[i]);
        maxSum = Math.max(maxSum, currentSum);
    }
    return maxSum;
}
// Time: O(n), Space: O(1)
```

### Must-Practice Problems (Arrays):
| # | Problem | Pattern | Difficulty | LeetCode # |
|---|---------|---------|------------|------------|
| 1 | Two Sum | HashMap | Easy | 1 |
| 2 | Best Time to Buy and Sell Stock | One Pass | Easy | 121 |
| 3 | Contains Duplicate | HashSet | Easy | 217 |
| 4 | Maximum Subarray | Kadane's | Medium | 53 |
| 5 | Container With Most Water | Two Pointers | Medium | 11 |
| 6 | 3Sum | Two Pointers + Sort | Medium | 15 |
| 7 | Longest Substring Without Repeating Characters | Sliding Window | Medium | 3 |
| 8 | Product of Array Except Self | Prefix/Suffix | Medium | 238 |
| 9 | Merge Intervals | Sort + Merge | Medium | 56 |
| 10 | Rotate Array | Reverse | Medium | 189 |

### Must-Practice Problems (Strings):
| # | Problem | Pattern | Difficulty | LeetCode # |
|---|---------|---------|------------|------------|
| 1 | Valid Palindrome | Two Pointers | Easy | 125 |
| 2 | Valid Anagram | Frequency Count | Easy | 242 |
| 3 | Longest Common Prefix | Vertical Scan | Easy | 14 |
| 4 | String to Integer (atoi) | Parsing | Medium | 8 |
| 5 | Longest Palindromic Substring | Expand Center | Medium | 5 |
| 6 | Group Anagrams | HashMap + Sort | Medium | 49 |

---

## 2. HashMap / HashSet (25% of DSA rounds)

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
// Time: O(n), Space: O(n)
```

```java
// First Non-Repeating Character
public int firstUniqChar(String s) {
    Map<Character, Integer> count = new HashMap<>();
    for (char c : s.toCharArray())
        count.put(c, count.getOrDefault(c, 0) + 1);
    for (int i = 0; i < s.length(); i++)
        if (count.get(s.charAt(i)) == 1) return i;
    return -1;
}
```

```java
// Group Anagrams
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> map = new HashMap<>();
    for (String s : strs) {
        char[] chars = s.toCharArray();
        Arrays.sort(chars);
        String key = new String(chars);
        map.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(map.values());
}
```

### When to use HashMap vs HashSet:
| Use Case | Data Structure |
|----------|---------------|
| Need key-value pairs | HashMap |
| Need only unique values | HashSet |
| Count frequency | HashMap<K, Integer> |
| Check existence quickly | HashSet |
| Two Sum pattern | HashMap<Value, Index> |

---

## 3. Linked List

```java
// Reverse a Linked List (asked VERY often)
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
// Time: O(n), Space: O(1)
```

```java
// Detect Cycle - Floyd's Tortoise & Hare
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

```java
// Merge Two Sorted Lists
public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    ListNode dummy = new ListNode(0);
    ListNode curr = dummy;
    while (l1 != null && l2 != null) {
        if (l1.val <= l2.val) { curr.next = l1; l1 = l1.next; }
        else { curr.next = l2; l2 = l2.next; }
        curr = curr.next;
    }
    curr.next = (l1 != null) ? l1 : l2;
    return dummy.next;
}
```

```java
// Find Middle of Linked List
public ListNode middleNode(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

```java
// Remove Nth Node From End
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode first = dummy, second = dummy;
    for (int i = 0; i <= n; i++) first = first.next;
    while (first != null) {
        first = first.next;
        second = second.next;
    }
    second.next = second.next.next;
    return dummy.next;
}
```

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

```java
// Min Stack - O(1) getMin
class MinStack {
    Stack<int[]> stack = new Stack<>(); // [value, currentMin]
    
    public void push(int val) {
        int min = stack.isEmpty() ? val : Math.min(val, stack.peek()[1]);
        stack.push(new int[]{val, min});
    }
    public void pop() { stack.pop(); }
    public int top() { return stack.peek()[0]; }
    public int getMin() { return stack.peek()[1]; }
}
```

```java
// Implement Queue using Two Stacks
class MyQueue {
    Stack<Integer> input = new Stack<>(), output = new Stack<>();
    
    public void push(int x) { input.push(x); }
    
    public int pop() {
        if (output.isEmpty())
            while (!input.isEmpty()) output.push(input.pop());
        return output.pop();
    }
    
    public int peek() {
        if (output.isEmpty())
            while (!input.isEmpty()) output.push(input.pop());
        return output.peek();
    }
    
    public boolean empty() { return input.isEmpty() && output.isEmpty(); }
}
```

---

## 5. Binary Tree / BST

```java
// Level Order Traversal (BFS) — very commonly asked
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

```java
// Inorder Traversal (Left → Root → Right) — gives sorted order for BST
public List<Integer> inorderTraversal(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    inorder(root, result);
    return result;
}
private void inorder(TreeNode node, List<Integer> result) {
    if (node == null) return;
    inorder(node.left, result);
    result.add(node.val);
    inorder(node.right, result);
}
```

```java
// Maximum Depth of Binary Tree
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

```java
// Validate BST
public boolean isValidBST(TreeNode root) {
    return validate(root, Long.MIN_VALUE, Long.MAX_VALUE);
}
private boolean validate(TreeNode node, long min, long max) {
    if (node == null) return true;
    if (node.val <= min || node.val >= max) return false;
    return validate(node.left, min, node.val) && validate(node.right, node.val, max);
}
```

---

## 6. Sorting & Searching

```java
// Binary Search — must know perfectly
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

```java
// Merge Sort — understand divide and conquer
public void mergeSort(int[] arr, int left, int right) {
    if (left < right) {
        int mid = (left + right) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}
private void merge(int[] arr, int left, int mid, int right) {
    int[] temp = new int[right - left + 1];
    int i = left, j = mid + 1, k = 0;
    while (i <= mid && j <= right) {
        if (arr[i] <= arr[j]) temp[k++] = arr[i++];
        else temp[k++] = arr[j++];
    }
    while (i <= mid) temp[k++] = arr[i++];
    while (j <= right) temp[k++] = arr[j++];
    System.arraycopy(temp, 0, arr, left, temp.length);
}
```

### Time Complexity Cheat Sheet:
| Algorithm | Best | Average | Worst | Space | Stable? |
|-----------|------|---------|-------|-------|---------|
| Bubble Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection Sort | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Binary Search | O(1) | O(log n) | O(log n) | O(1) | N/A |
| HashMap lookup | O(1) | O(1) | O(n) | O(n) | N/A |

---

## 7. Dynamic Programming

```java
// Climbing Stairs (Fibonacci variant)
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

```java
// Coin Change — classic DP
public int coinChange(int[] coins, int amount) {
    int[] dp = new int[amount + 1];
    Arrays.fill(dp, amount + 1);
    dp[0] = 0;
    for (int i = 1; i <= amount; i++)
        for (int coin : coins)
            if (coin <= i)
                dp[i] = Math.min(dp[i], dp[i - coin] + 1);
    return dp[amount] > amount ? -1 : dp[amount];
}
```

```java
// Longest Common Subsequence
public int longestCommonSubsequence(String text1, String text2) {
    int m = text1.length(), n = text2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            if (text1.charAt(i - 1) == text2.charAt(j - 1))
                dp[i][j] = dp[i - 1][j - 1] + 1;
            else
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
    return dp[m][n];
}
```

```java
// 0/1 Knapsack
public int knapsack(int[] weights, int[] values, int capacity) {
    int n = weights.length;
    int[][] dp = new int[n + 1][capacity + 1];
    for (int i = 1; i <= n; i++)
        for (int w = 0; w <= capacity; w++)
            if (weights[i - 1] <= w)
                dp[i][w] = Math.max(dp[i - 1][w], values[i - 1] + dp[i - 1][w - weights[i - 1]]);
            else
                dp[i][w] = dp[i - 1][w];
    return dp[n][capacity];
}
```

### DP Problem Recognition Tips:
| If the question says... | Think of... |
|------------------------|-------------|
| "Find minimum/maximum" | DP or Greedy |
| "Count number of ways" | DP |
| "Is it possible to..." | DP (boolean) |
| "Optimal substructure" | DP |
| Overlapping subproblems | Memoization/Tabulation |

---

## 8. Graph Basics (Know concepts, not deep implementation)

```java
// BFS on Graph
public void bfs(Map<Integer, List<Integer>> graph, int start) {
    Set<Integer> visited = new HashSet<>();
    Queue<Integer> queue = new LinkedList<>();
    queue.add(start);
    visited.add(start);
    while (!queue.isEmpty()) {
        int node = queue.poll();
        System.out.print(node + " ");
        for (int neighbor : graph.getOrDefault(node, List.of()))
            if (!visited.contains(neighbor)) {
                visited.add(neighbor);
                queue.add(neighbor);
            }
    }
}
```

---

## 🔥 Top 30 LeetCode Problems for SDET Interviews

| # | Problem | Topic | LC# |
|---|---------|-------|-----|
| 1 | Two Sum | Array/HashMap | 1 |
| 2 | Valid Parentheses | Stack | 20 |
| 3 | Merge Two Sorted Lists | LinkedList | 21 |
| 4 | Best Time to Buy/Sell Stock | Array | 121 |
| 5 | Valid Palindrome | String | 125 |
| 6 | Linked List Cycle | LinkedList | 141 |
| 7 | Reverse Linked List | LinkedList | 206 |
| 8 | Contains Duplicate | Array/Set | 217 |
| 9 | Valid Anagram | String | 242 |
| 10 | Binary Tree Level Order | Tree/BFS | 102 |
| 11 | Maximum Depth of Binary Tree | Tree/DFS | 104 |
| 12 | Climbing Stairs | DP | 70 |
| 13 | Maximum Subarray | DP/Kadane | 53 |
| 14 | 3Sum | Array/Two Pointers | 15 |
| 15 | Longest Substring Without Repeating | Sliding Window | 3 |
| 16 | Group Anagrams | HashMap | 49 |
| 17 | Product of Array Except Self | Prefix | 238 |
| 18 | Merge Intervals | Sort | 56 |
| 19 | Coin Change | DP | 322 |
| 20 | Min Stack | Stack Design | 155 |
| 21 | LRU Cache | HashMap+DLL | 146 |
| 22 | Search in Rotated Sorted Array | Binary Search | 33 |
| 23 | Find Median from Data Stream | Heap | 295 |
| 24 | Number of Islands | Graph/BFS | 200 |
| 25 | Validate BST | Tree | 98 |
| 26 | Implement Queue using Stacks | Stack | 232 |
| 27 | Longest Common Subsequence | DP | 1143 |
| 28 | Remove Nth Node From End | LinkedList | 19 |
| 29 | Rotate Array | Array | 189 |
| 30 | FizzBuzz | Basic | 412 |

---

*Practice 2-3 problems daily. Focus on understanding patterns, not memorizing solutions.*

