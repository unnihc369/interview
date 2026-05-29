# Day 36 — Greedy & Heap Scheduling Patterns

**Topics:** Reorganize String · Task Scheduler · Partition Labels

---

# 1. Reorganize String

## Problem Statement

Given string `s`, rearrange characters so no two adjacent characters are the same. Return any valid string or `""` if impossible.

---

# Example

```text
Input:  "aab"
Output: "aba"

Input:  "aaab"
Output: ""  (impossible)
```

---

# Brute Force Approach

Backtrack all permutations; check adjacency constraint.

```text
O(n! * n) — not interview-viable
```

---

# Optimized Approach (Max Heap / Greedy)

## Core Idea

1. Count frequencies with `HashMap`
2. If max freq > `(n+1)/2` → impossible (pigeonhole)
3. Max-heap by frequency; always place most frequent char that is **not** same as last placed
4. If top equals last, pop second-most and place it

Alternative: sort chars by freq descending, place in even indices first then odd.

---

# Java Solution

```java
class Solution {
    public String reorganizeString(String s) {
        int[] freq = new int[26];
        for (char c : s.toCharArray()) freq[c - 'a']++;

        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> b[1] - a[1]);
        for (int i = 0; i < 26; i++) {
            if (freq[i] > 0) pq.offer(new int[]{i, freq[i]});
        }

        StringBuilder sb = new StringBuilder();
        while (!pq.isEmpty()) {
            int[] top = pq.poll();
            if (sb.length() > 0 && sb.charAt(sb.length() - 1) == (char) ('a' + top[0])) {
                if (pq.isEmpty()) return "";
                int[] second = pq.poll();
                sb.append((char) ('a' + second[0]));
                second[1]--;
                if (second[1] > 0) pq.offer(second);
                pq.offer(top);
            } else {
                sb.append((char) ('a' + top[0]));
                top[1]--;
                if (top[1] > 0) pq.offer(top);
            }
        }
        return sb.toString();
    }
}
```

---

# JavaScript Solution

```javascript
function reorganizeString(s) {
  const map = new Map();
  for (const c of s) map.set(c, (map.get(c) || 0) + 1);

  const max = Math.max(...map.values());
  if (max > Math.floor((s.length + 1) / 2)) return "";

  const arr = [...map.entries()].sort((a, b) => b[1] - a[1]);
  const res = new Array(s.length);
  let idx = 0;
  for (const [ch, count] of arr) {
    for (let i = 0; i < count; i++) {
      res[idx] = ch;
      idx += 2;
      if (idx >= s.length) idx = 1;
    }
  }
  return res.join("");
}
```

---

# Time Complexity

```text
O(n log k) — k = distinct chars (≤ 26)
```

---

# 2. Task Scheduler

## Problem Statement

Given tasks (each letter A–Z) and cooldown `n`, find minimum time units to finish all tasks. Same task must wait at least `n` intervals before running again (idle slots allowed).

---

# Example

```text
Input: tasks = ["A","A","A","B","B","B"], n = 2
Output: 8
Explanation: A -> B -> idle -> A -> B -> idle -> A -> B
```

---

# Brute Force Approach

Simulate with queue tracking next-available time per task.

```text
O(total_time) — acceptable but heap formula is cleaner
```

---

# Optimized Approach (Math + Greedy)

## Core Idea

Most frequent task `M` with count `maxCount` drives schedule.

```text
slots = (maxCount - 1) * (n + 1) + count_of_tasks_with_max_freq
answer = max(slots, tasks.length)
```

Intuition: fill `(n+1)`-wide rows with most frequent tasks; idle fills gaps.

---

# Java Solution

```java
class Solution {
    public int leastInterval(char[] tasks, int n) {
        int[] freq = new int[26];
        for (char t : tasks) freq[t - 'A']++;

        Arrays.sort(freq);
        int maxCount = freq[25];
        int maxFreqTasks = 0;
        for (int f : freq) {
            if (f == maxCount) maxFreqTasks++;
        }

        int slots = (maxCount - 1) * (n + 1) + maxFreqTasks;
        return Math.max(slots, tasks.length);
    }
}
```

---

# JavaScript Solution

```javascript
function leastInterval(tasks, n) {
  const freq = Array(26).fill(0);
  for (const t of tasks) freq[t.charCodeAt(0) - 65]++;

  freq.sort((a, b) => b - a);
  const maxCount = freq[0];
  const maxFreqTasks = freq.filter(f => f === maxCount).length;

  const slots = (maxCount - 1) * (n + 1) + maxFreqTasks;
  return Math.max(slots, tasks.length);
}
```

---

# Time Complexity

```text
O(26 log 26) ≈ O(1) for fixed alphabet
```

---

# 3. Partition Labels

## Problem Statement

Partition string `s` into as many parts as possible so each letter appears in **at most one** part. Return sizes of each part.

---

# Example

```text
Input:  "ababcbacadefegdehijhklij"
Output: [9, 7, 8]
```

---

# Brute Force Approach

Try every split point; verify each letter appears in only one segment.

```text
O(n^2) or worse
```

---

# Optimized Approach (Greedy / Last Occurrence)

## Core Idea

1. Record last index of each character
2. Scan left to right; `end` = max last index seen so far
3. When `i == end`, close current partition

---

# Java Solution

```java
class Solution {
    public List<Integer> partitionLabels(String s) {
        int[] last = new int[26];
        for (int i = 0; i < s.length(); i++) {
            last[s.charAt(i) - 'a'] = i;
        }

        List<Integer> result = new ArrayList<>();
        int start = 0, end = 0;

        for (int i = 0; i < s.length(); i++) {
            end = Math.max(end, last[s.charAt(i) - 'a']);
            if (i == end) {
                result.add(end - start + 1);
                start = i + 1;
            }
        }
        return result;
    }
}
```

---

# JavaScript Solution

```javascript
function partitionLabels(s) {
  const last = {};
  for (let i = 0; i < s.length; i++) last[s[i]] = i;

  const result = [];
  let start = 0, end = 0;

  for (let i = 0; i < s.length; i++) {
    end = Math.max(end, last[s[i]]);
    if (i === end) {
      result.push(end - start + 1);
      start = i + 1;
    }
  }
  return result;
}
```

---

# Time Complexity

```text
O(n) — two passes
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Reorganize String | Max heap / interleave by freq | O(n log k) |
| Task Scheduler | Math on max frequency | O(1) alphabet |
| Partition Labels | Greedy last-occurrence window | O(n) |

---

# Interview Questions

## Why is max freq > (n+1)/2 impossible for reorganize?
If one char dominates more than half the string (rounded up), two adjacent slots cannot separate all copies — pigeonhole principle.

## Task Scheduler — when is simulation needed?
Math formula suffices for count-only problems. Follow-up with different task durations requires priority queue simulation.

## Partition Labels — relation to merge intervals?
Each char's span `[first, last]` is an interval; greedy merge when scan index reaches current max end — same spirit as interval merging.

## Can reorganize use counting sort instead of heap?
Yes — fixed 26 letters; bucket by frequency then fill even/odd indices.

---

*End of Day 36 LeetCode*
