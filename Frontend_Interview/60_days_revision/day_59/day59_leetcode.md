# Day 59 — LeetCode (String DP)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 115 | [Distinct Subsequences](#1-distinct-subsequences-115) | Hard | Count subseq |
| 97 | [Interleaving String](#2-interleaving-string-97) | Medium | 2D boolean |
| 583 | [Delete Operation for Two Strings](#3-delete-operation-review-583) | Medium | Review — LCS |

---

## Table of Contents

1. [Distinct Subsequences (115)](#1-distinct-subsequences-115)
2. [Interleaving String (97)](#2-interleaving-string-97)
3. [Delete Operation Review (583)](#3-delete-operation-review-583)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Distinct Subsequences (115)

### Problem (short)

Return number of distinct subsequences of `s` which equal `t`.

```
s = "rabbbit", t = "rabbit" → 3
```

### Solution

```js
function numDistinct(s, t) {
  const m = s.length, n = t.length;
  const dp = Array(n + 1).fill(0);
  dp[0] = 1;
  for (let i = 1; i <= m; i++) {
    for (let j = n; j >= 1; j--) {
      if (s[i - 1] === t[j - 1]) dp[j] += dp[j - 1];
    }
  }
  return dp[n];
}
```

Space-optimized: iterate j backwards.

---

## 2. Interleaving String (97)

### Solution

```js
function isInterleave(s1, s2, s3) {
  const m = s1.length, n = s2.length;
  if (m + n !== s3.length) return false;
  const dp = Array(n + 1).fill(false);
  dp[0] = true;
  for (let j = 1; j <= n; j++) dp[j] = dp[j - 1] && s2[j - 1] === s3[j - 1];
  for (let i = 1; i <= m; i++) {
    dp[0] = dp[0] && s1[i - 1] === s3[i - 1];
    for (let j = 1; j <= n; j++) {
      dp[j] = (dp[j] && s1[i - 1] === s3[i + j - 1]) || (dp[j - 1] && s2[j - 1] === s3[i + j - 1]);
    }
  }
  return dp[n];
}
```

---

## 3. Delete Operation Review (583)

### Solution

```js
function minDistance(word1, word2) {
  const m = word1.length, n = word2.length;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      dp[i][j] = word1[i - 1] === word2[j - 1]
        ? dp[i - 1][j - 1] + 1
        : Math.max(dp[i - 1][j], dp[i][j - 1]);
    }
  }
  const lcs = dp[m][n];
  return m + n - 2 * lcs;
}
```

---

## 4. Pattern Cheat Sheet

| Match char | 115 | 97 |
|------------|-----|-----|
| Yes | add dp[i-1][j-1] | OR from top/left |
| No | keep dp[i-1][j] | skip |

---

**Prev/Next:** [Concepts](day59_concepts.md) · [Machine Coding](day59_machine_coding.md)
