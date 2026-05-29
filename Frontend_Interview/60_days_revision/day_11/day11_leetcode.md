# Day 11 — LeetCode (Sliding Window — Product & Vowels)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 713 | [Subarray Product Less Than K](#1-subarray-product-less-than-k-713) | Medium | Count subarrays — variable window |
| 1456 | [Maximum Number of Vowels in a Substring](#2-maximum-number-of-vowels-in-a-substring-1456) | Medium | Fixed window k |
| 1590 | [Make Sum Divisible by P](#3-make-sum-divisible-by-p-1590) | Medium | Prefix + modular window |

> **Note:** LC 1590 is prefix/modulo; LC **1598** / substring problems — below also covers **3**-char **latest** substring pattern (1590 alternative: **1699** closest). Primary **1590** included as advanced; **1456**-style **latest substring** bonus in §4.

---

## Table of Contents

1. [Subarray Product Less Than K (713)](#1-subarray-product-less-than-k-713)
2. [Maximum Number of Vowels (1456)](#2-maximum-number-of-vowels-in-a-substring-1456)
3. [Make Sum Divisible by P (1590)](#3-make-sum-divisible-by-p-1590)
4. [Latest Substring with 3 Occurrences (pattern)](#4-latest-substring-with-3-occurrences-pattern)
5. [Pattern Cheat Sheet](#5-pattern-cheat-sheet)

---

## 1. Subarray Product Less Than K (713)

### Problem (short)

Count **contiguous subarrays** where product < `k`. `nums` positive integers.

```
nums = [10, 5, 2, 6], k = 100 → 8
```

### Hint — Variable window on product

Expand; while product >= k, divide by nums[left] and shrink.

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
function numSubarrayProductLessThanK(nums, k) {
  if (k <= 1) return 0;

  let left = 0;
  let product = 1;
  let count = 0;

  for (let right = 0; right < nums.length; right++) {
    product *= nums[right];
    while (product >= k) {
      product /= nums[left++];
    }
    count += right - left + 1;
  }
  return count;
}
```

### Why `count += right - left + 1`?

All subarrays ending at `right` with start in `[left, right]` are valid.

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(1) |

---

## 2. Maximum Number of Vowels (1456)

### Problem (short)

String `s`, integer `k`. Return max vowels in any substring of length **k**.

```
s = "abciiidef", k = 3 → 3
s = "aeiou", k = 2 → 2
```

### Fixed window

```js
/**
 * @param {string} s
 * @param {number} k
 * @return {number}
 */
function maxVowels(s, k) {
  const isVowel = (c) => "aeiou".includes(c);
  let count = 0;
  let max = 0;

  for (let i = 0; i < s.length; i++) {
    if (isVowel(s[i])) count++;
    if (i >= k) {
      if (isVowel(s[i - k])) count--;
    }
    if (i >= k - 1) max = Math.max(max, count);
  }
  return max;
}
```

---

## 3. Make Sum Divisible by P (1590)

### Problem (short)

Remove **shortest contiguous subarray** so remaining sum is divisible by `p`. Return length or -1.

```
nums = [3,1,4,2], p = 6 → 1  (remove [4] or [2] — shortest length 1)
nums = [6,3,5,2], p = 9 → 2
```

### Hint — Target remainder via prefix mod

`total % p` = rem. Need remove subarray with sum ≡ rem (mod p). Use hash map of prefix mod → earliest index; sliding window on mod sums.

```js
/**
 * @param {number[]} nums
 * @param {number} p
 * @return {number}
 */
function minSubarray(nums, p) {
  const total = nums.reduce((a, b) => a + b, 0);
  const rem = total % p;
  if (rem === 0) return 0;

  const need = rem;
  const modIndex = new Map([[0, -1]]);
  let prefix = 0;
  let minLen = nums.length;

  for (let i = 0; i < nums.length; i++) {
    prefix = (prefix + nums[i]) % p;
    const target = (prefix - need + p) % p;
    if (modIndex.has(target)) {
      minLen = Math.min(minLen, i - modIndex.get(target));
    }
    modIndex.set(prefix, i);
  }

  return minLen === nums.length ? -1 : minLen;
}
```

---

## 4. Latest Substring with 3 Occurrences (pattern)

Interview variant: find **latest** (rightmost) substring where each of 3 given chars appears at least once — fixed sliding window over sorted positions or expand from end.

```js
/** Latest minimum window containing all of a, b, c in order of rightmost end */
function latestTripleWindow(s) {
  const need = new Set(["a", "b", "c"]);
  let left = 0;
  const have = new Map();
  let formed = 0;
  let best = "";

  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    if (need.has(c)) {
      have.set(c, (have.get(c) ?? 0) + 1);
      if (have.get(c) === 1) formed++;
    }
    while (formed === 3) {
      best = s.slice(left, right + 1); // keep updating — last valid = latest longest end
      const l = s[left];
      if (need.has(l)) {
        have.set(l, have.get(l) - 1);
        if (have.get(l) === 0) formed--;
      }
      left++;
    }
  }
  return best;
}
```

---

## 5. Pattern Cheat Sheet

| Problem | Pattern |
|---------|---------|
| **713** | Variable window; count += window size |
| **1456** | Fixed k; slide vowel count |
| **1590** | Prefix mod + hash map |

---

*End of Day 11 LeetCode*
