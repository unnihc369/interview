# Day 53 — Full Mock DSA (1 Hour Timed)

**Week 8 · Thursday · Strict 60 minutes + 10 min review**

| # | Problem | Target | Difficulty |
|---|---------|--------|------------|
| 1 | Median of Two Sorted Arrays | 25 min | Hard |
| 2 | Merge k Sorted Lists | 20 min | Hard |
| 3 | Word Ladder II | 15 min sketch + discuss | Hard |

**Rules:** No IDE autocomplete. Timer per problem. If stuck 5 min, read hint only, then solution.

---

# Mock Protocol

```text
00:00 — Read all 3 statements (3 min)
00:03 — Problem 1 start
00:28 — Problem 2 start (stop P1 even if incomplete)
00:48 — Problem 3 start
01:00 — STOP coding
01:10 — Grade with rubric below
```

---

# Problem 1 — Median of Two Sorted Arrays (LeetCode 4)

## Statement

Given sorted arrays `nums1` and `nums2` of size m and n, return the median of the merged sorted array in **O(log(m+n))** time.

---

## Hints (use only if stuck)

1. Median = element(s) at partition index `(m+n+1)/2`  
2. Binary search on **smaller** array partition `i`; `j = (m+n+1)/2 - i`  
3. Correct partition: `maxLeft <= minRight` on both sides  

---

## Approach

Binary search partition in `nums1` such that:

- Left half has `(m+n+1)/2` elements  
- `nums1[i-1] <= nums2[j]` and `nums2[j-1] <= nums1[i]`  

If partition too far right in nums1 → move `hi = i-1`.

---

## Java Solution

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if (nums1.length > nums2.length) {
            return findMedianSortedArrays(nums2, nums1);
        }
        int m = nums1.length, n = nums2.length;
        int lo = 0, hi = m;

        while (lo <= hi) {
            int i = (lo + hi) / 2;
            int j = (m + n + 1) / 2 - i;

            int maxLeft1 = (i == 0) ? Integer.MIN_VALUE : nums1[i - 1];
            int minRight1 = (i == m) ? Integer.MAX_VALUE : nums1[i];
            int maxLeft2 = (j == 0) ? Integer.MIN_VALUE : nums2[j - 1];
            int minRight2 = (j == n) ? Integer.MAX_VALUE : nums2[j];

            if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {
                if ((m + n) % 2 == 0) {
                    return (Math.max(maxLeft1, maxLeft2) + Math.min(minRight1, minRight2)) / 2.0;
                }
                return Math.max(maxLeft1, maxLeft2);
            } else if (maxLeft1 > minRight2) {
                hi = i - 1;
            } else {
                lo = i + 1;
            }
        }
        throw new IllegalArgumentException("Input not sorted");
    }
}
```

**Time:** O(log min(m,n)) · **Space:** O(1)

---

## Rubric — Problem 1

| Score | Criteria |
|-------|----------|
| 5 | Correct binary search partition, handles even/odd |
| 3 | Merge approach O(m+n) with correct median |
| 1 | Brute force merge attempted |

---

# Problem 2 — Merge k Sorted Lists (LeetCode 23)

## Statement

Given array of `k` linked lists (each sorted), merge into one sorted list.

---

## Hints

1. Min-heap of size k with list heads  
2. Alternative: divide and conquer merge pairs  

---

## Approach (Min-Heap)

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a.val));
        for (ListNode head : lists) {
            if (head != null) pq.offer(head);
        }
        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;
        while (!pq.isEmpty()) {
            ListNode node = pq.poll();
            tail.next = node;
            tail = tail.next;
            if (node.next != null) pq.offer(node.next);
        }
        return dummy.next;
    }
}
```

**Time:** O(N log k) where N = total nodes · **Space:** O(k)

---

## Divide & Conquer (Alternative)

```java
ListNode mergeKLists(ListNode[] lists) {
    if (lists == null || lists.length == 0) return null;
    return mergeRange(lists, 0, lists.length - 1);
}

ListNode mergeRange(ListNode[] lists, int lo, int hi) {
    if (lo == hi) return lists[lo];
    int mid = lo + (hi - lo) / 2;
    return mergeTwo(mergeRange(lists, lo, mid), mergeRange(lists, mid + 1, hi));
}
```

**Time:** O(N log k) · Mention in interview as cleaner for very large k.

---

## Rubric — Problem 2

| Score | Criteria |
|-------|----------|
| 5 | Heap or D&C with correct complexity |
| 3 | Sequential merge two-at-a-time (O(kN)) |
| 1 | Brute collect + sort |

---

# Problem 3 — Word Ladder II (LeetCode 126)

## Statement

Given `beginWord`, `endWord`, word list `wordList`, return **all shortest** transformation sequences from `beginWord` to `endWord`. Each step change one letter; each intermediate word must be in `wordList`.

---

## Hints

1. BFS from `beginWord` to find shortest distance to every word  
2. Build adjacency / parent map during BFS  
3. DFS backtrack from `endWord` to collect paths  

---

## Approach Outline

**Phase 1 — BFS**

```text
Queue: beginWord
dist[beginWord] = 0
While queue:
  For each word, try all 26 letters per position
  If neighbor in wordSet and not visited at this level:
    record distance, parents
```

**Phase 2 — Backtrack**

```java
void dfs(String word, List<String> path, List<List<String>> result) {
    if (word.equals(beginWord)) {
        List<String> seq = new ArrayList<>(path);
        Collections.reverse(seq);
        result.add(seq);
        return;
    }
    for (String parent : parents.get(word)) {
        path.add(parent);
        dfs(parent, path, result);
        path.remove(path.size() - 1);
    }
}
```

---

## Java Solution (Combined Sketch)

```java
class Solution {
    public List<List<String>> findLadders(String beginWord, String endWord, List<String> wordList) {
        Set<String> dict = new HashSet<>(wordList);
        List<List<String>> result = new ArrayList<>();
        if (!dict.contains(endWord)) return result;

        Map<String, List<String>> graph = new HashMap<>();
        Map<String, Integer> dist = new HashMap<>();
        Queue<String> q = new LinkedList<>();
        q.offer(beginWord);
        dist.put(beginWord, 0);
        boolean found = false;

        while (!q.isEmpty() && !found) {
            int size = q.size();
            Set<String> visited = new HashSet<>();
            for (int i = 0; i < size; i++) {
                String word = q.poll();
                int d = dist.get(word);
                char[] arr = word.toCharArray();
                for (int j = 0; j < arr.length; j++) {
                    char orig = arr[j];
                    for (char c = 'a'; c <= 'z'; c++) {
                        arr[j] = c;
                        String next = new String(arr);
                        if (!dict.contains(next)) continue;
                        graph.computeIfAbsent(next, k -> new ArrayList<>()).add(word);
                        if (!dist.containsKey(next)) {
                            dist.put(next, d + 1);
                            if (next.equals(endWord)) found = true;
                            else { q.offer(next); visited.add(next); }
                        }
                    }
                    arr[j] = orig;
                }
            }
        }
        if (!dist.containsKey(endWord)) return result;
        List<String> path = new ArrayList<>();
        path.add(endWord);
        backtrack(endWord, beginWord, graph, path, result);
        return result;
    }

    private void backtrack(String word, String begin, Map<String, List<String>> graph,
                           List<String> path, List<List<String>> result) {
        if (word.equals(begin)) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (String parent : graph.getOrDefault(word, List.of())) {
            path.add(parent);
            backtrack(parent, begin, graph, path, result);
            path.remove(path.size() - 1);
        }
    }
}
```

*Full production code may need level-by-level BFS to avoid duplicate edges — practice on LeetCode.*

**Time:** O(N · L · 26) BFS · **Space:** O(N) for graph

---

## Rubric — Problem 3

| Score | Criteria |
|-------|----------|
| 5 | BFS shortest + backtrack all paths |
| 3 | BFS finds one shortest path |
| 1 | DFS without shortest guarantee |

---

# Post-Mock Review (10 min)

| Question | Your answer |
|----------|-------------|
| Which problem lost most time? | |
| Did you clarify constraints upfront? | |
| Median: can you draw partition diagram? | |
| Merge K: heap size at any time? | |
| Word Ladder II: BFS vs bi-directional BFS? | |

**Bi-directional BFS** for Word Ladder I/II: start from both ends, stop when frontiers meet — reduces explored nodes.

---

# Retake Schedule

If score < 8/15 total rubric points:

- Day 54 morning: redo Median + Merge K only (45 min)  
- Word Ladder II: study solution, rewrite from memory  

---

*End of Day 53 — Mock DSA*
