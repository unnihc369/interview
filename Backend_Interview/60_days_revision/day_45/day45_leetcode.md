# Day 45 — Mock Prep: Greedy Heap, DP String, Priority Queue Events

**Topics:** Longest Happy String · Form Target String With Dictionary · Maximum Number of Events That Can Be Attended

---

# 1. Longest Happy String

## Problem Statement

String is **happy** if: no `aaa`, no `bbb`, no `ccc`. Given `a`, `b`, `c` counts, return longest happy string. If tie, return lexicographically smallest.

---

# Example

```text
Input: a = 1, b = 1, c = 7
Output: "ccaccbcc"
```

---

# Brute Force Approach

Generate all permutations — exponential.

---

# Optimized Approach (Greedy + Max Heap)

## Core Idea

1. Max heap by remaining count
2. Append up to 2 of the most frequent char (if last two same, pick second-most)
3. Never append 3rd identical consecutive char

---

# Java Solution

```java
class Solution {
    public String longestDiverseString(int a, int b, int c) {
        PriorityQueue<int[]> pq = new PriorityQueue<>((x, y) -> y[1] - x[1]);
        if (a > 0) pq.offer(new int[]{'a', a});
        if (b > 0) pq.offer(new int[]{'b', b});
        if (c > 0) pq.offer(new int[]{'c', c});

        StringBuilder sb = new StringBuilder();
        while (!pq.isEmpty()) {
            int[] first = pq.poll();
            int len = sb.length();
            if (len >= 2 && sb.charAt(len - 1) == first[0] && sb.charAt(len - 2) == first[0]) {
                if (pq.isEmpty()) break;
                int[] second = pq.poll();
                sb.append((char) second[0]);
                if (--second[1] > 0) pq.offer(second);
                pq.offer(first);
            } else {
                int count = Math.min(2, first[1]);
                for (int i = 0; i < count; i++) sb.append((char) first[0]);
                first[1] -= count;
                if (first[1] > 0) pq.offer(first);
            }
        }
        return sb.toString();
    }
}
```

---

# Time Complexity

```text
O((a+b+c) log 3) = O(n)
```

---

# 2. Form Target String With Dictionary

## Problem Statement

Given strings `words` (dictionary) and target, count number of ways to form `target` by concatenating dictionary words (words reusable). Return modulo `10^9 + 7`.

---

# Example

```text
Input: words = ["ab","abc","b"], target = "abc"
Output: 4
```

---

# Brute Force Approach

DFS try all word combinations — exponential.

---

# Optimized Approach (DP + Trie/Prefix Optimization)

## Core Idea

`dp[i]` = ways to form `target[0..i-1]`.

For each position `i`, try each word `w` where `target` substring ending at `i` matches `w`.

Optimization: group words by first character or build trie for faster matching.

---

# Java Solution

```java
class Solution {
    private static final int MOD = 1_000_000_007;

    public int numWays(String[] words, String target) {
        int tLen = target.length();
        int[] dp = new int[tLen + 1];
        dp[0] = 1;

        // Precompute: for each position in target, which words match ending here
        List<int[]>[] match = new List[tLen + 1];
        for (int i = 0; i <= tLen; i++) match[i] = new ArrayList<>();

        for (String w : words) {
            int wLen = w.length();
            for (int i = wLen; i <= tLen; i++) {
                if (target.substring(i - wLen, i).equals(w)) {
                    match[i].add(new int[]{i - wLen, wLen});
                }
            }
        }

        for (int i = 1; i <= tLen; i++) {
            for (int[] m : match[i]) {
                int start = m[0];
                dp[i] = (dp[i] + dp[start]) % MOD;
            }
        }
        return dp[tLen];
    }
}
```

---

# Optimized with Character Frequency (LeetCode 1639 style)

Precompute `count[wordIndex][charIndex][letter]` — how many words have char `c` at position `j`. Then DP over target position and word prefix length.

---

# Time Complexity

```text
O(tLen × wordLen × alphabet) with preprocessing
```

---

# 3. Maximum Number of Events That Can Be Attended

## Problem Statement

Given events `[startDay, endDay]`, you can attend one event per day. Return max events you can attend.

---

# Example

```text
Input: events = [[1,2],[2,3],[3,4]]
Output: 3
```

---

# Brute Force Approach

Try all subsets — exponential.

---

# Optimized Approach (Sort + Min Heap)

## Core Idea

1. Sort events by start day
2. Iterate day by day (or sweep through sorted starts)
3. Min heap stores **end days** of active events
4. For current day: add all events starting today; remove expired (end < day); pick event with **earliest end** (greedy)

---

# Java Solution

```java
class Solution {
    public int maxEvents(int[][] events) {
        Arrays.sort(events, (a, b) -> a[0] - b[0]);
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        int i = 0, n = events.length, day = 0, attended = 0;

        while (i < n || !minHeap.isEmpty()) {
            if (minHeap.isEmpty()) day = events[i][0];

            while (i < n && events[i][0] <= day) {
                minHeap.offer(events[i][1]);
                i++;
            }
            minHeap.poll(); // attend event ending soonest
            attended++;
            day++;

            while (!minHeap.isEmpty() && minHeap.peek() < day) minHeap.poll();
        }
        return attended;
    }
}
```

---

# Time Complexity

```text
O(n log n) — sort + heap operations
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Longest Happy String | Greedy + max heap | O(n) |
| Form Target (Dictionary) | DP + prefix match | O(t × w) |
| Max Events Attended | Sort + min heap (earliest deadline) | O(n log n) |

---

# Interview Questions

## Why max 2 same chars in Happy String?

Third would create `aaa`/`bbb`/`ccc`. Greedy: use 2 of dominant char when safe.

## Form Target — why DP?

Overlapping subproblems: ways to form prefix reused. `dp[i]` builds from shorter prefixes.

## Max Events — why earliest end day?

Classic **interval scheduling** greedy — pick job that frees resource soonest (same as meeting rooms).

---

# One-Line Revision

```text
Happy String = greedy heap avoid triple; Form Target = DP on prefix; Max Events = sort by start + min-heap on end days.
```

---

*End of Day 45 LeetCode*
