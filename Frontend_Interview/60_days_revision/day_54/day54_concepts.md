# Day 54 — Longest Increasing Subsequence (LIS)

**Week 8 — Dynamic Programming** · **Topics:** LIS · Patience sorting · Binary search · Envelope nesting

---

## Table of Contents

1. [LIS Problem Variants](#1-lis-problem-variants)
2. [O(n²) DP Solution](#2-on²-dp-solution)
3. [O(n log n) Patience Sorting](#3-on-log-n-patience-sorting)
4. [Russian Doll Envelopes](#4-russian-doll-envelopes)
5. [Number of LIS (673)](#5-number-of-lis-673)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. LIS Problem Variants

| Variant | Output |
|---------|--------|
| Length only | 300 |
| Count of LIS | 673 |
| 2D nesting | 354 |

**Subsequence** = not necessarily contiguous. **Subarray** = contiguous (different problem).

---

## 2. O(n²) DP Solution

`dp[i]` = length of LIS ending at index `i`.

```js
function lengthOfLIS(nums) {
  const n = nums.length;
  const dp = Array(n).fill(1);
  let best = 1;
  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
    }
    best = Math.max(best, dp[i]);
  }
  return best;
}
```

---

## 3. O(n log n) Patience Sorting

Maintain `tails[k]` = smallest tail of increasing subsequence of length `k+1`.

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

Binary search finds position to replace or extend.

---

## 4. Russian Doll Envelopes

Sort by width ascending; if same width, sort height **descending** (can't nest same width). LIS on heights.

```js
function maxEnvelopes(envelopes) {
  envelopes.sort((a, b) => a[0] === b[0] ? b[1] - a[1] : a[0] - b[0]);
  return lengthOfLIS(envelopes.map((e) => e[1]));
}
```

---

## 5. Number of LIS (673)

Track `count[i]` = number of LIS ending at `i`, `len[i]` = length.

```js
function findNumberOfLIS(nums) {
  const n = nums.length;
  const len = Array(n).fill(1);
  const count = Array(n).fill(1);
  for (let i = 1; i < n; i++) {
    for (let j = 0; j < i; j++) {
      if (nums[j] < nums[i]) {
        if (len[j] + 1 > len[i]) {
          len[i] = len[j] + 1;
          count[i] = count[j];
        } else if (len[j] + 1 === len[i]) {
          count[i] += count[j];
        }
      }
    }
  }
  const maxLen = Math.max(...len);
  return count.reduce((s, c, i) => s + (len[i] === maxLen ? c : 0), 0);
}
```

---

## 6. Interview Quick Index

| LC | Technique |
|----|-----------|
| 300 | O(n²) DP or O(n log n) tails |
| 354 | sort + LIS on 2nd dim |
| 673 | LIS + count arrays |

---

**Next:** [Machine Coding](day54_machine_coding.md) · [LeetCode](day54_leetcode.md)
