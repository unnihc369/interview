# Day 9 — LeetCode (Sliding Window — Anagrams & Median)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 438 | [Find All Anagrams in a String](#1-find-all-anagrams-in-a-string-438) | Medium | Fixed window + freq match |
| 567 | [Permutation in String](#2-permutation-in-string-567) | Medium | Fixed window (same as 438) |
| 480 | [Sliding Window Median](#3-sliding-window-median-480) | Hard | Fixed window + two heaps |

---

## Table of Contents

1. [Find All Anagrams in a String (438)](#1-find-all-anagrams-in-a-string-438)
2. [Permutation in String (567)](#2-permutation-in-string-567)
3. [Sliding Window Median (480)](#3-sliding-window-median-480)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Find All Anagrams in a String (438)

### Problem (short)

Given `s` and `p`, return **start indices** of all anagrams of `p` in `s`.

```
s = "cbaebabacd", p = "abc" → [0, 6]
s = "abab", p = "ab" → [0, 1, 2]
```

### Hint 1 — Fixed window size = p.length

Slide window of length `k = p.length`. Compare frequency maps.

### Hint 2 — matches counter

Track `matches` = number of chars where `window[c] === need[c]`. When `matches === need.size`, record index.

### JavaScript

```js
/**
 * @param {string} s
 * @param {string} p
 * @return {number[]}
 */
function findAnagrams(s, p) {
  if (p.length > s.length) return [];

  const need = new Map();
  for (const ch of p) need.set(ch, (need.get(ch) ?? 0) + 1);

  const k = p.length;
  const window = new Map();
  let matches = 0;
  const res = [];

  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    window.set(c, (window.get(c) ?? 0) + 1);
    if (need.has(c) && window.get(c) === need.get(c)) matches++;

    if (right >= k) {
      const leftIdx = right - k;
      if (matches === need.size) res.push(leftIdx);

      const l = s[leftIdx];
      if (need.has(l) && window.get(l) === need.get(l)) matches--;
      window.set(l, window.get(l) - 1);
    }
  }
  return res;
}
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(1) — 26 letters |

---

## 2. Permutation in String (567)

### Problem (short)

Return `true` if `s2` contains a **permutation** of `s1` as substring.

```
s1 = "ab", s2 = "eidbaooo" → true
s1 = "ab", s2 = "eidboaoo" → false
```

### Solution

Same as 438 — fixed window; return true if any index found.

```js
/**
 * @param {string} s1
 * @param {string} s2
 * @return {boolean}
 */
function checkInclusion(s1, s2) {
  const indices = findAnagrams(s2, s1);
  return indices.length > 0;
}
```

Or inline with early exit when `matches === need.size`.

---

## 3. Sliding Window Median (480)

### Problem (short)

Array `nums`, window size `k`. Return median of each window (sorted middle; even → average of two middles).

```
nums = [1,3,-1,-3,5,3,6,7], k = 3
→ [1, -1, -1, 3, 5, 6]
```

### Hint 1 — Naive O(n·k log k)

Sort each window — too slow for interview follow-up.

### Hint 2 — Two heaps (lazy deletion)

| Heap | Role |
|------|------|
| Max-heap (low) | Lower half |
| Min-heap (high) | Upper half |

Balance sizes: `low.size >= high.size` and `low.size - high.size <= 1`.

### Hint 3 — JS has no built-in heap

Use sorted structure + delay-delete map, or `SortedList` pattern with binary search insert.

### Practical JS (Multiset with arrays + lazy delete)

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number[]}
 */
function medianSlidingWindow(nums, k) {
  const sorted = [];

  function insert(x) {
    let lo = 0,
      hi = sorted.length;
    while (lo < hi) {
      const mid = (lo + hi) >> 1;
      if (sorted[mid] < x) lo = mid + 1;
      else hi = mid;
    }
    sorted.splice(lo, 0, x);
  }

  function remove(x) {
    const i = sorted.indexOf(x);
    if (i >= 0) sorted.splice(i, 1);
  }

  function median() {
    const mid = Math.floor(sorted.length / 2);
    if (sorted.length % 2) return sorted[mid];
    return (sorted[mid - 1] + sorted[mid]) / 2;
  }

  const res = [];
  for (let i = 0; i < nums.length; i++) {
    insert(nums[i]);
    if (i >= k) remove(nums[i - k]);
    if (i >= k - 1) res.push(median());
  }
  return res;
}
```

> Interview: explain two-heap approach; implement sorted multiset if allowed. Mention O(n log k) with proper heap.

### Two-heap sketch (Java-style logic)

```text
add num → push to appropriate heap → rebalance
remove num (leaving window) → lazy delete from heaps
median → top of max-heap (odd k) or average of tops (even k)
```

### Complexity (two heaps)

| Time | Space |
|------|-------|
| O(n log k) | O(k) |

---

## 4. Pattern Cheat Sheet

| Problem | Window | Key |
|---------|--------|-----|
| **438** | Fixed k | Freq maps + matches count |
| **567** | Fixed k | Same as 438, boolean |
| **480** | Fixed k | Two heaps / sorted multiset |

### Fixed window template

```js
for (let right = 0; right < n; right++) {
  add(nums[right]);
  if (right >= k - 1) {
    updateAnswer();
    remove(nums[right - k + 1]); // or left index
  }
}
```

---

*End of Day 9 LeetCode*
