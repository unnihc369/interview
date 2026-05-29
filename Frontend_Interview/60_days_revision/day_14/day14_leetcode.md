# Day 14 — LeetCode (Sunday: Timed Mock)

**Format:** 3 problems · **60 minutes total** · no hints until after mock

| # | Problem | Target time | Difficulty |
|---|---------|-------------|------------|
| 3 | Longest Substring Without Repeating Characters | 15 min | Medium |
| 76 | Minimum Window Substring | 20 min | Hard |
| 480 | Sliding Window Median | 25 min | Hard |

---

## Mock Rules

1. Set timer for **60 min**.
2. Write **JavaScript** solutions in a single file.
3. No IDE autocomplete for templates — recall from memory.
4. After time: compare with [Day 8](../day_8/day8_leetcode.md) and [Day 9](../day_9/day9_leetcode.md).
5. Score: **2/3** working = pass · **3/3** = strong.

---

## Problem Statements (copy to scratchpad)

### 1. (3) Longest Substring Without Repeating Characters

Given string `s`, find length of longest substring without repeating characters.

```
Input: "abcabcbb"  → 3
Input: "bbbbb"     → 1
Input: "pwwkew"    → 3
```

### 2. (76) Minimum Window Substring

Given `s` and `t`, return minimum window in `s` containing all characters of `t`. Return `""` if none.

```
s = "ADOBECODEBANC", t = "ABC"  → "BANC"
```

### 3. (480) Sliding Window Median

Given `nums` and window size `k`, return array of medians for each window.

```
nums = [1,3,-1,-3,5,3,6,7], k = 3
→ [1.00000,-1.00000,-1.00000,3.00000,5.00000,6.00000]
```

---

## Scratch Space (your attempt)

```js
// Problem 3 — start: ___:___  end: ___:___

// Problem 76 — start: ___:___  end: ___:___

// Problem 480 — start: ___:___  end: ___:___
```

---

## Self-Grading Rubric

| Score | Criteria |
|-------|----------|
| **3** | All pass LeetCode-style tests; discuss complexity |
| **2** | 3 + 76 correct; 480 correct approach |
| **1** | 3 correct only — review 76 template |
| **0** | Re-do Day 8–9 with templates |

---

## Reference Solutions (after mock only)

<details>
<summary>Click after timed attempt — Problem 3</summary>

```js
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

</details>

<details>
<summary>Click after timed attempt — Problem 76</summary>

```js
function minWindow(s, t) {
  if (!t.length) return "";
  const need = new Map();
  for (const c of t) need.set(c, (need.get(c) ?? 0) + 1);
  let have = 0;
  const required = need.size;
  const window = new Map();
  let left = 0;
  let res = [-1, 0, 0];

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

</details>

<details>
<summary>Click after timed attempt — Problem 480</summary>

```js
function medianSlidingWindow(nums, k) {
  const sorted = [];
  const insert = (x) => {
    let lo = 0, hi = sorted.length;
    while (lo < hi) {
      const mid = (lo + hi) >> 1;
      if (sorted[mid] < x) lo = mid + 1;
      else hi = mid;
    }
    sorted.splice(lo, 0, x);
  };
  const remove = (x) => {
    const i = sorted.indexOf(x);
    if (i >= 0) sorted.splice(i, 1);
  };
  const median = () => {
    const m = Math.floor(sorted.length / 2);
    return sorted.length % 2
      ? sorted[m]
      : (sorted[m - 1] + sorted[m]) / 2;
  };

  const res = [];
  for (let i = 0; i < nums.length; i++) {
    insert(nums[i]);
    if (i >= k) remove(nums[i - k]);
    if (i >= k - 1) res.push(median());
  }
  return res;
}
```

**Interview upgrade:** two heaps + lazy deletion for O(n log k).

</details>

---

## Post-Mock Review Questions

1. For **3**, why `left = max(left, last.get(ch) + 1)`?
2. For **76**, when does `have` increment?
3. For **480**, state time complexity of sorted-array vs two-heap approach.

---

## Next Week Preview

**Week 3 — Two Pointers** (Days 15–21). Rest today if mock was heavy.

---

*End of Day 14 LeetCode mock*
