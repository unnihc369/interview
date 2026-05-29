# Day 54 — LeetCode (LIS)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 300 | [Longest Increasing Subsequence](#1-longest-increasing-subsequence-300) | Medium | LIS DP / binary search |
| 354 | [Russian Doll Envelopes](#2-russian-doll-envelopes-354) | Hard | Sort + LIS |
| 673 | [Number of Longest Increasing Subsequence](#3-number-of-longest-increasing-subsequence-673) | Medium | LIS + count |

---

## Table of Contents

1. [Longest Increasing Subsequence (300)](#1-longest-increasing-subsequence-300)
2. [Russian Doll Envelopes (354)](#2-russian-doll-envelopes-354)
3. [Number of LIS (673)](#3-number-of-lis-673)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Longest Increasing Subsequence (300)

### Solution — O(n log n)

```js
function lengthOfLIS(nums) {
  const tails = [];
  for (const x of nums) {
    let lo = 0, hi = tails.length;
    while (lo < hi) {
      const mid = (lo + hi) >> 1;
      if (tails[mid] < x) lo = mid + 1;
      else hi = mid;
    }
    tails[lo] = x;
  }
  return tails.length;
}
```

### Solution — O(n²)

```js
function lengthOfLIS(nums) {
  const dp = nums.map(() => 1);
  for (let i = 1; i < nums.length; i++)
    for (let j = 0; j < i; j++)
      if (nums[j] < nums[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
  return Math.max(...dp);
}
```

---

## 2. Russian Doll Envelopes (354)

### Solution

```js
function maxEnvelopes(envelopes) {
  envelopes.sort((a, b) => (a[0] === b[0] ? b[1] - a[1] : a[0] - b[0]));
  const heights = envelopes.map((e) => e[1]);
  return lengthOfLIS(heights);
}
```

---

## 3. Number of LIS (673)

### Solution

```js
function findNumberOfLIS(nums) {
  const n = nums.length;
  const len = Array(n).fill(1);
  const cnt = Array(n).fill(1);
  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) {
        if (len[j] + 1 > len[i]) { len[i] = len[j] + 1; cnt[i] = cnt[j]; }
        else if (len[j] + 1 === len[i]) cnt[i] += cnt[j];
      }
    }
  }
  const mx = Math.max(...len);
  return cnt.reduce((s, c, i) => s + (len[i] === mx ? c : 0), 0);
}
```

---

## 4. Pattern Cheat Sheet

| Problem | Key trick |
|---------|-----------|
| 300 | tails + binary search |
| 354 | sort w asc, h desc on tie |
| 673 | parallel len[] and cnt[] |

---

**Prev/Next:** [Concepts](day54_concepts.md) · [Machine Coding](day54_machine_coding.md)
