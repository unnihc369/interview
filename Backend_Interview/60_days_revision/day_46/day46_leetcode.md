# Day 46 — Mock DSA Round (Timed: 90 Minutes)

**Format:** 3 hard problems from Weeks 2–4 · 30 min each · Full solutions · Self-score rubric

**Topics:** Median of Two Sorted Arrays · Word Search II · Alien Dictionary

---

# Mock Interview Rules

```text
Total time:    90 minutes (strict)
Problem 1:     30 min — Binary Search (Hard)
Problem 2:     30 min — Trie + Backtracking (Hard)
Problem 3:     30 min — Graph Topological Sort (Hard)

Scoring:
  ✅ Working optimal solution     = 10 pts
  ⚠️  Correct approach, minor bug = 7 pts
  ❌ Brute force only              = 4 pts
  ❌ No progress in 30 min         = 0 pts

Target: 24+/30 to pass mock bar
```

---

# Problem 1: Median of Two Sorted Arrays (Hard)

*Source: Week 4, Day 22*

## Problem Statement

Given two sorted arrays `nums1` and `nums2` of size `m` and `n`, return the **median** of the two sorted arrays. Overall run time must be O(log (m+n)).

---

# Example

```text
Input: nums1 = [1,3], nums2 = [2]
Output: 2.0

Input: nums1 = [1,2], nums2 = [3,4]
Output: 2.5
```

---

# Interviewer Expectations (30 min)

| Minute | Goal |
|--------|------|
| 0–5 | Clarify: arrays sorted? empty? duplicates? |
| 5–10 | Reject merge O(m+n) — must be O(log) |
| 10–25 | Binary search on smaller array, partition logic |
| 25–30 | Handle odd/even total length, test edge cases |

---

# Optimized Approach (Binary Search on Partition)

## Core Idea

1. Binary search on smaller array for partition `i`
2. `j = (m + n + 1) / 2 - i` in second array
3. Valid partition: `maxLeft1 <= minRight2` AND `maxLeft2 <= minRight1`
4. Median from max of left halves or avg with min of right halves

---

# Java Solution

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

            int maxLeft1  = (i == 0) ? Integer.MIN_VALUE : nums1[i - 1];
            int minRight1 = (i == m) ? Integer.MAX_VALUE : nums1[i];
            int maxLeft2  = (j == 0) ? Integer.MIN_VALUE : nums2[j - 1];
            int minRight2 = (j == n) ? Integer.MAX_VALUE : nums2[j];

            if (maxLeft1 <= minRight2 && maxLeft2 <= minRight1) {
                if ((m + n) % 2 == 1) {
                    return Math.max(maxLeft1, maxLeft2);
                }
                return (Math.max(maxLeft1, maxLeft2) + Math.min(minRight1, minRight2)) / 2.0;
            } else if (maxLeft1 > minRight2) {
                hi = i - 1;
            } else {
                lo = i + 1;
            }
        }
        throw new IllegalArgumentException();
    }
}
```

---

# Time Complexity

```text
O(log(min(m, n)))
```

---

# Problem 2: Word Search II (Hard)

*Source: Week 4, Day 24*

## Problem Statement

Given an `m × n` board of characters and a list of strings `words`, return all words on the board. Each word must be constructed from letters of sequentially adjacent cells (no reuse per word).

---

# Example

```text
Input:
board = [["o","a","a","n"],
         ["e","t","a","e"],
         ["i","h","k","r"],
         ["i","f","l","v"]]
words = ["oath","pea","eat","rain"]

Output: ["eat","oath"]
```

---

# Interviewer Expectations (30 min)

| Minute | Goal |
|--------|------|
| 0–5 | Clarify: dictionary size? board size? |
| 5–10 | Naive: Word Search I for each word — too slow |
| 10–20 | Build Trie from words; DFS board with pruning |
| 20–30 | Remove found words from Trie; handle duplicates |

---

# Optimized Approach (Trie + DFS)

## Core Idea

1. Insert all words into Trie
2. DFS from each cell; follow Trie edges
3. When node `isWord`, add to result
4. Prune: remove Trie branch after all words found (optional optimization)

---

# Java Solution

```java
class Solution {
    class TrieNode {
        TrieNode[] children = new TrieNode[26];
        String word = null;
    }

    public List<String> findWords(char[][] board, String[] words) {
        TrieNode root = buildTrie(words);
        List<String> result = new ArrayList<>();
        int m = board.length, n = board[0].length;

        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++)
                dfs(board, r, c, root, result);

        return result;
    }

    private TrieNode buildTrie(String[] words) {
        TrieNode root = new TrieNode();
        for (String w : words) {
            TrieNode node = root;
            for (char ch : w.toCharArray()) {
                int idx = ch - 'a';
                if (node.children[idx] == null) node.children[idx] = new TrieNode();
                node = node.children[idx];
            }
            node.word = w;
        }
        return root;
    }

    private void dfs(char[][] board, int r, int c, TrieNode node, List<String> result) {
        char ch = board[r][c];
        if (ch == '#' || node.children[ch - 'a'] == null) return;

        node = node.children[ch - 'a'];
        if (node.word != null) {
            result.add(node.word);
            node.word = null; // avoid duplicates
        }

        board[r][c] = '#';
        int[][] dirs = {{0,1},{0,-1},{1,0},{-1,0}};
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nr < board.length && nc >= 0 && nc < board[0].length)
                dfs(board, nr, nc, node, result);
        }
        board[r][c] = ch;
    }
}
```

---

# Time Complexity

```text
O(m × n × 4^L) worst case, heavily pruned by Trie — O(m×n×words) practical
```

---

# Problem 3: Alien Dictionary (Hard)

*Source: Week 3, Day 17*

## Problem Statement

Given sorted dictionary of an alien language, derive the character order. Return `""` if invalid order (cycle).

---

# Example

```text
Input: words = ["wrt","wrf","er","ett","rftt"]
Output: "wertf"

Input: words = ["z","x"]
Output: "zx"

Input: words = ["z","x","z"]
Output: ""  (cycle)
```

---

# Interviewer Expectations (30 min)

| Minute | Goal |
|--------|------|
| 0–5 | Compare adjacent words only for first differing char |
| 5–15 | Build directed graph: char A before B → edge A→B |
| 15–25 | Topological sort (Kahn's BFS or DFS) |
| 25–30 | Detect cycle; handle prefix edge case ("abc","ab") |

---

# Optimized Approach (Topological Sort)

## Core Idea

1. Compare adjacent words, find first differing char → edge `c1 → c2`
2. Special case: if `word1.startsWith(word2) && word1.length > word2.length` → invalid
3. Kahn's algorithm: in-degree 0 queue
4. If sorted order size < unique chars → cycle

---

# Java Solution

```java
class Solution {
    public String alienOrder(String[] words) {
        Map<Character, Set<Character>> adj = new HashMap<>();
        Map<Character, Integer> indegree = new HashMap<>();

        for (String w : words)
            for (char c : w.toCharArray())
                adj.putIfAbsent(c, new HashSet<>());

        for (int i = 0; i < words.length - 1; i++) {
            String w1 = words[i], w2 = words[i + 1];
            if (w1.length() > w2.length() && w1.startsWith(w2)) return "";

            for (int j = 0; j < Math.min(w1.length(), w2.length()); j++) {
                char c1 = w1.charAt(j), c2 = w2.charAt(j);
                if (c1 != c2) {
                    if (!adj.get(c1).contains(c2)) {
                        adj.get(c1).add(c2);
                        indegree.put(c2, indegree.getOrDefault(c2, 0) + 1);
                    }
                    break;
                }
            }
        }

        for (char c : adj.keySet())
            indegree.putIfAbsent(c, 0);

        Queue<Character> queue = new LinkedList<>();
        for (char c : indegree.keySet())
            if (indegree.get(c) == 0) queue.offer(c);

        StringBuilder sb = new StringBuilder();
        while (!queue.isEmpty()) {
            char c = queue.poll();
            sb.append(c);
            for (char next : adj.get(c)) {
                indegree.put(next, indegree.get(next) - 1);
                if (indegree.get(next) == 0) queue.offer(next);
            }
        }
        return sb.length() == indegree.size() ? sb.toString() : "";
    }
}
```

---

# Time Complexity

```text
O(C) where C = total characters in all words
```

---

# Mock Debrief Checklist

After 90 minutes, score yourself:

- [ ] Stated brute force before optimizing?
- [ ] Wrote edge cases before coding?
- [ ] Communicated time/space complexity?
- [ ] Tested with provided examples + one custom case?
- [ ] Problem 1 partition logic correct?
- [ ] Problem 2 Trie pruning / no duplicate words?
- [ ] Problem 3 prefix invalid case handled?

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Median Two Sorted | Binary search partition | O(log(min(m,n))) |
| Word Search II | Trie + DFS backtrack | O(m×n×α) |
| Alien Dictionary | Graph + topological sort | O(C) |

---

# Interview Questions (Post-Mock)

## Why binary search on smaller array for median?

Keeps O(log(min(m,n))) and ensures partition indices stay in bounds.

## Word Search II — why Trie beats HashSet of words?

Shared prefixes collapse — early termination when path not in Trie.

## Alien Dictionary — why only adjacent word pairs?

Sorted order guarantees all constraints appear in some adjacent comparison (transitive order).

---

*End of Day 46 Mock DSA*
