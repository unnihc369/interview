# Day 11 — Sliding Window

---

# 1. Longest Substring Without Repeating Characters

## Problem Statement

Given a string `s`, find the length of the longest substring without repeating characters.

---

# Example

```text
Input: "abcabcbb"
Output: 3   ("abc")
```

---

# Approach Hints

- Sliding window `[left, right]` with HashMap/Set of char → last index.
- When duplicate found, move `left` past previous occurrence.
- Track max window size.

---

# Core Idea (Walkthrough)

1. `right` expands window; store char index in map.
2. If char seen and index >= `left`, shift `left` to `map.get(c) + 1`.
3. Update max = `right - left + 1`.

---

# Java Solution

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        Map<Character, Integer> last = new HashMap<>();
        int left = 0, max = 0;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            if (last.containsKey(c) && last.get(c) >= left) {
                left = last.get(c) + 1;
            }
            last.put(c, right);
            max = Math.max(max, right - left + 1);
        }
        return max;
    }
}
```

---

# JavaScript Solution

```javascript
function lengthOfLongestSubstring(s) {
  const last = new Map();
  let left = 0, max = 0;

  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    if (last.has(c) && last.get(c) >= left) left = last.get(c) + 1;
    last.set(c, right);
    max = Math.max(max, right - left + 1);
  }
  return max;
}
```

---

# Time & Space Complexity

```text
Time:  O(n)
Space: O(min(n, alphabet))
```

---

# 2. Sliding Window Maximum

## Problem Statement

Given array `nums` and window size `k`, return max in each sliding window of size `k`.

---

# Example

```text
Input: nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]
```

---

# Approach Hints

- Brute: O(n*k) — scan each window.
- Optimal: deque storing indices of useful candidates (monotonic decreasing values).
- Front of deque = index of max for current window.

---

# Core Idea (Walkthrough)

1. Deque stores indices; values at indices decreasing.
2. Remove indices outside window from front.
3. Before adding `i`, remove smaller values from back.
4. If `i >= k-1`, append `nums[deque.front]`.

---

# Java Solution

```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        Deque<Integer> dq = new ArrayDeque<>();
        int[] res = new int[nums.length - k + 1];
        int idx = 0;

        for (int i = 0; i < nums.length; i++) {
            while (!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();
            while (!dq.isEmpty() && nums[dq.peekLast()] < nums[i]) dq.pollLast();
            dq.offerLast(i);
            if (i >= k - 1) res[idx++] = nums[dq.peekFirst()];
        }
        return res;
    }
}
```

---

# JavaScript Solution

```javascript
function maxSlidingWindow(nums, k) {
  const dq = [];
  const res = [];

  for (let i = 0; i < nums.length; i++) {
    while (dq.length && dq[0] <= i - k) dq.shift();
    while (dq.length && nums[dq[dq.length - 1]] < nums[i]) dq.pop();
    dq.push(i);
    if (i >= k - 1) res.push(nums[dq[0]]);
  }
  return res;
}
```

---

# Time & Space Complexity

```text
Time:  O(n)
Space: O(k)
```

---

# 3. Minimum Window Substring

## Problem Statement

Given strings `s` and `t`, return the minimum window substring of `s` such that every character in `t` (including multiplicity) is included. Return `""` if none exists.

---

# Example

```text
Input: s = "ADOBECODEBANC", t = "ABC"
Output: "BANC"
```

---

# Approach Hints

- Expand `right` until window valid (all chars of t covered).
- Shrink `left` while still valid to minimize length.
- Track `need` and `have` counts with HashMap.

---

# Core Idea (Walkthrough)

1. Build frequency map for `t`.
2. Expand window; when all required chars satisfied, update answer and shrink from left.

---

# Java Solution

```java
class Solution {
    public String minWindow(String s, String t) {
        if (t.isEmpty()) return "";

        Map<Character, Integer> need = new HashMap<>();
        for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);

        int have = 0, required = need.size();
        Map<Character, Integer> window = new HashMap<>();
        int left = 0;
        int[] res = {-1, -1};
        int resLen = Integer.MAX_VALUE;

        for (int right = 0; right < s.length(); right++) {
            char c = s.charAt(right);
            window.merge(c, 1, Integer::sum);

            if (need.containsKey(c) && window.get(c).intValue() == need.get(c).intValue()) {
                have++;
            }

            while (have == required) {
                if (right - left + 1 < resLen) {
                    res[0] = left;
                    res[1] = right;
                    resLen = right - left + 1;
                }
                char leftChar = s.charAt(left);
                window.merge(leftChar, -1, Integer::sum);
                if (need.containsKey(leftChar) &&
                    window.get(leftChar) < need.get(leftChar)) {
                    have--;
                }
                left++;
            }
        }

        return resLen == Integer.MAX_VALUE ? "" : s.substring(res[0], res[1] + 1);
    }
}
```

---

# JavaScript Solution

```javascript
function minWindow(s, t) {
  if (!t.length) return "";

  const need = new Map();
  for (const c of t) need.set(c, (need.get(c) || 0) + 1);

  let have = 0, required = need.size;
  const window = new Map();
  let left = 0;
  let res = [-1, -1];
  let resLen = Infinity;

  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    window.set(c, (window.get(c) || 0) + 1);

    if (need.has(c) && window.get(c) === need.get(c)) have++;

    while (have === required) {
      if (right - left + 1 < resLen) {
        res = [left, right];
        resLen = right - left + 1;
      }
      const lc = s[left];
      window.set(lc, window.get(lc) - 1);
      if (need.has(lc) && window.get(lc) < need.get(lc)) have--;
      left++;
    }
  }

  return resLen === Infinity ? "" : s.slice(res[0], res[1] + 1);
}
```

---

# Time & Space Complexity

```text
Time:  O(|s| + |t|)
Space: O(|s| + |t|)
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Longest Substring Without Repeating | Sliding window + HashMap | O(n) |
| Sliding Window Maximum | Monotonic deque | O(n) |
| Minimum Window Substring | Expand/shrink window | O(n) |

---

*End of Day 11 LeetCode*
