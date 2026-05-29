# Day 47 — Mock DSA Round: 3 Amazon Hards (Timed: 90 Minutes)

**Format:** Amazon-style hard problems · 30 min each · Full solutions · Pattern focus

**Topics:** Trapping Rain Water II · Sliding Window Maximum · First Missing Positive

---

# Mock Interview Rules

```text
Total time:    90 minutes
Problem 1:     30 min — Heap + BFS (Hard)
Problem 2:     30 min — Monotonic Deque (Hard)
Problem 3:     30 min — Array In-Place (Hard)

Amazon bar:    Optimal solution + clean code + edge cases + O() analysis
Communicate:   "I'll start with brute force, then optimize because..."
```

---

# Problem 1: Trapping Rain Water II (LeetCode 407)

## Problem Statement

Given an `m × n` height map, compute how much water can be trapped after raining.

---

# Example

```text
Input: heightMap = [[1,4,3,1,3,2],[3,2,1,3,2,4],[2,3,3,2,3,1]]
Output: 4
```

---

# Interviewer Expectations

| Minute | Goal |
|--------|------|
| 0–5 | Note: 1D version uses two pointers — 2D differs |
| 5–15 | Min heap on boundary cells; expand inward |
| 15–25 | Water at cell = max(0, boundaryHeight - cellHeight) |
| 25–30 | Visited set, push neighbors |

---

# Optimized Approach (Min Heap BFS)

## Core Idea

Water level at interior cell = min of shortest path to boundary (like Dijkstra).
Start heap with all border cells. Process lowest height first; update neighbors.

---

# Java Solution

```java
class Solution {
    private static final int[][] DIRS = {{0,1},{0,-1},{1,0},{-1,0}};

    public int trapRainWater(int[][] heightMap) {
        int m = heightMap.length, n = heightMap[0].length;
        if (m < 3 || n < 3) return 0;

        boolean[][] visited = new boolean[m][n];
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[2] - b[2]);

        for (int r = 0; r < m; r++) {
            pq.offer(new int[]{r, 0, heightMap[r][0]});
            pq.offer(new int[]{r, n - 1, heightMap[r][n - 1]});
            visited[r][0] = visited[r][n - 1] = true;
        }
        for (int c = 1; c < n - 1; c++) {
            pq.offer(new int[]{0, c, heightMap[0][c]});
            pq.offer(new int[]{m - 1, c, heightMap[m - 1][c]});
            visited[0][c] = visited[m - 1][c] = true;
        }

        int water = 0;
        while (!pq.isEmpty()) {
            int[] curr = pq.poll();
            int r = curr[0], c = curr[1], h = curr[2];

            for (int[] d : DIRS) {
                int nr = r + d[0], nc = c + d[1];
                if (nr < 0 || nr >= m || nc < 0 || nc >= n || visited[nr][nc]) continue;
                visited[nr][nc] = true;
                water += Math.max(0, h - heightMap[nr][nc]);
                pq.offer(new int[]{nr, nc, Math.max(h, heightMap[nr][nc])});
            }
        }
        return water;
    }
}
```

---

# Time Complexity

```text
O(m × n × log(m × n))
```

---

# Problem 2: Sliding Window Maximum (LeetCode 239)

## Problem Statement

Given array `nums` and window size `k`, return max in each sliding window.

---

# Example

```text
Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
```

---

# Optimized Approach (Monotonic Deque)

## Core Idea

Deque stores **indices** of candidates in decreasing value order.
Front = max of current window. Remove indices outside window from front.
Remove smaller elements from back before adding new.

---

# Java Solution

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        Deque<Integer> dq = new ArrayDeque<>();
        int[] result = new int[nums.length - k + 1];
        int ri = 0;

        for (int i = 0; i < nums.length; i++) {
            while (!dq.isEmpty() && dq.peekFirst() < i - k + 1)
                dq.pollFirst();
            while (!dq.isEmpty() && nums[dq.peekLast()] < nums[i])
                dq.pollLast();
            dq.offerLast(i);
            if (i >= k - 1) result[ri++] = nums[dq.peekFirst()];
        }
        return result;
    }
}
```

---

# Time Complexity

```text
O(n) — each element pushed/popped at most once
```

---

# Problem 3: First Missing Positive (LeetCode 41)

## Problem Statement

Given unsorted array, find the smallest missing **positive** integer in O(n) time, O(1) extra space.

---

# Example

```text
Input: nums = [3,4,-1,1]
Output: 2

Input: nums = [7,8,9,11,12]
Output: 1
```

---

# Interviewer Expectations

| Minute | Goal |
|--------|------|
| 0–5 | Answer must be in [1, n+1] where n = length |
| 5–15 | Index as hash: place nums[i] at index nums[i]-1 |
| 15–25 | Cycle swap, skip invalid (≤0 or >n) |
| 25–30 | Scan for first nums[i] != i+1 |

---

# Optimized Approach (Cycle Sort / Index Marking)

## Core Idea

For each value `v` in [1, n], it belongs at index `v-1`.
Swap until position correct. Answer = first index where `nums[i] != i+1`.

---

# Java Solution

```java
class Solution {
    public int firstMissingPositive(int[] nums) {
        int n = nums.length;

        for (int i = 0; i < n; i++) {
            while (nums[i] > 0 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
                int temp = nums[nums[i] - 1];
                nums[nums[i] - 1] = nums[i];
                nums[i] = temp;
            }
        }

        for (int i = 0; i < n; i++) {
            if (nums[i] != i + 1) return i + 1;
        }
        return n + 1;
    }
}
```

---

# Time Complexity

```text
O(n) time, O(1) space
```

---

# Amazon Mock Debrief

## Communication phrases that score well

```text
"I'll use a min-heap because we process the lowest boundary first..."
"The deque maintains monotonic decreasing values so front is always max..."
"Answer is bounded by n, so I can use the array as a hash table..."
```

## Common mistakes

| Problem | Mistake |
|---------|---------|
| Rain Water II | Using 1D two-pointer approach |
| Sliding Window Max | Storing values instead of indices in deque |
| First Missing Positive | Sorting O(n log n) when O(n) required |

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Trapping Rain Water II | Min heap BFS from boundary | O(mn log mn) |
| Sliding Window Maximum | Monotonic deque | O(n) |
| First Missing Positive | Index as hash / cycle sort | O(n) |

---

# Interview Questions

## Rain Water II vs I?

1D: two pointers from ends. 2D: multi-directional boundary — need priority queue.

## Why deque not heap for sliding max?

Amortized O(1) per element with deque vs O(log k) with heap — both acceptable; deque is optimal.

## First Missing Positive — why O(1) space?

Use input array indices as presence markers — values swapped into correct slots.

---

*End of Day 47 Mock DSA*
