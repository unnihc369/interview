# Day 8 — LeetCode (Sliding Window — Fixed & Variable)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 3 | [Longest Substring Without Repeating Characters](#1-longest-substring-without-repeating-characters-3) | Medium | Variable window + Set/Map |
| 904 | [Fruit Into Baskets](#2-fruit-into-baskets-904) | Medium | At most K distinct (K=2) |
| 76 | [Minimum Window Substring](#3-minimum-window-substring-76) | Hard | Expand/shrink + frequency match |

---

## Table of Contents

1. [Longest Substring Without Repeating Characters (3)](#1-longest-substring-without-repeating-characters-3)
2. [Fruit Into Baskets (904)](#2-fruit-into-baskets-904)
3. [Minimum Window Substring (76)](#3-minimum-window-substring-76)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Longest Substring Without Repeating Characters (3)

### Problem (short)

Given string `s`, return length of **longest substring** without repeating characters.

```
"abcabcbb" → 3  ("abc")
"bbbbb"    → 1
"pwwkew"   → 3  ("wke")
```

### Hint 1 — Brute force → window

Check every substring O(n³). Use **two pointers**: expand `right`, shrink `left` when duplicate.

### Hint 2 — Track last seen index

Map `char → last index`. When duplicate at `right`, move `left` to `max(left, lastIndex + 1)`.

### Walkthrough — `"pwwkew"`

```
right=0 'p' map{p:0} len=1
right=1 'w' map{p:0,w:1} len=2
right=2 'w' dup → left=max(0,1+1)=2 map{w:2} len=1
right=3 'k' len=2
right=4 'e' len=3
right=5 'w' left=3 len=3 → max=3
```

### JavaScript

```js
/**
 * @param {string} s
 * @return {number}
 */
function lengthOfLongestSubstring(s) {
  const last = new Map();
  let left = 0;
  let maxLen = 0;

  for (let right = 0; right < s.length; right++) {
    const ch = s[right];
    if (last.has(ch) && last.get(ch) >= left) {
      left = last.get(ch) + 1;
    }
    last.set(ch, right);
    maxLen = Math.max(maxLen, right - left + 1);
  }
  return maxLen;
}
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(min(n, charset)) |

### Common mistakes

- Moving `left` only +1 on duplicate (Set approach) — still works but slower; index map is O(1) jump.
- Forgetting `left = max(left, ...)` when chars repeat outside current window.

---

## 2. Fruit Into Baskets (904)

### Problem (short)

Array `fruits` — each element is fruit type. You have **2 baskets** (at most **2 distinct types** in window). Return **max number of fruits** you can pick (contiguous subarray).

```
[1,2,1]        → 3
[0,1,2,2]      → 3  ([1,2,2] or [2,2] — actually [1,2,2] length 3)
[1,2,3,2,2]    → 4  ([2,3,2,2])
```

### Hint 1 — K=2 distinct template

Classic **sliding window with frequency map**. Expand `right`; while `distinct > 2`, shrink `left`.

### JavaScript

```js
/**
 * @param {number[]} fruits
 * @return {number}
 */
function totalFruit(fruits) {
  const count = new Map();
  let left = 0;
  let maxLen = 0;

  for (let right = 0; right < fruits.length; right++) {
    const f = fruits[right];
    count.set(f, (count.get(f) ?? 0) + 1);

    while (count.size > 2) {
      const l = fruits[left++];
      count.set(l, count.get(l) - 1);
      if (count.get(l) === 0) count.delete(l);
    }
    maxLen = Math.max(maxLen, right - left + 1);
  }
  return maxLen;
}
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(1) — at most 3 keys in map |

---

## 3. Minimum Window Substring (76)

### Problem (short)

Strings `s` and `t`. Return **smallest substring** of `s` containing all chars of `t` (including multiplicity). Return `""` if none.

```
s = "ADOBECODEBANC", t = "ABC" → "BANC"
s = "a", t = "a" → "a"
```

### Hint 1 — Two counters

- `need`: freq map for `t`
- `have`: how many unique chars in window satisfy required count
- `required = Object.keys(need).length`

### Hint 2 — Expand then shrink

Expand `right` until `have === required`, then shrink `left` while still valid to minimize window.

### JavaScript

```js
/**
 * @param {string} s
 * @param {string} t
 * @return {string}
 */
function minWindow(s, t) {
  if (!t.length) return "";

  const need = new Map();
  for (const ch of t) need.set(ch, (need.get(ch) ?? 0) + 1);

  let have = 0;
  const required = need.size;
  const window = new Map();
  let left = 0;
  let res = [-1, 0, 0]; // [len, left, right]

  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    window.set(c, (window.get(c) ?? 0) + 1);

    if (need.has(c) && window.get(c) === need.get(c)) have++;

    while (have === required) {
      const len = right - left + 1;
      if (res[0] === -1 || len < res[0]) res = [len, left, right];

      const l = s[left];
      window.set(l, window.get(l) - 1);
      if (need.has(l) && window.get(l) < need.get(l)) have--;
      left++;
    }
  }

  return res[0] === -1 ? "" : s.slice(res[1], res[2] + 1);
}
```

### Complexity

| Time | Space |
|------|-------|
| O(\|s\| + \|t\|) | O(\|s\| + \|t\|) |

### Common mistakes

- Counting `have` on every char add (should increment only when `window[c] === need[c]`)
- Not updating best window **inside** shrink loop

---

## 4. Pattern Cheat Sheet

| Problem | Window type | Shrink when |
|---------|-------------|-------------|
| **3** | Variable | Duplicate char in window |
| **904** | Variable | Distinct types > 2 |
| **76** | Variable | Valid → shrink for min length |

### Template (variable window)

```js
for (let right = 0; right < n; right++) {
  // add s[right] to window
  while (windowInvalid) {
    // remove s[left]
    left++;
  }
  // update answer
}
```

### Solve order

1. **3** — Map last index; max length.
2. **904** — Map freq; at most 2 distinct.
3. **76** — need/have; shrink when valid.

---

*End of Day 8 LeetCode*
