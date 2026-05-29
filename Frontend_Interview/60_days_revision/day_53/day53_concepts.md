# Day 53 — Longest Common Subsequence (LCS)

**Week 8 — Dynamic Programming** · **Topics:** LCS · Edit distance · String DP · 2D table on strings

---

## Table of Contents

1. [String DP Overview](#1-string-dp-overview)
2. [LCS Definition & Recurrence](#2-lcs-definition--recurrence)
3. [LCS Table Construction](#3-lcs-table-construction)
4. [Delete Operation for Two Strings](#4-delete-operation-for-two-strings)
5. [Edit Distance (Levenshtein)](#5-edit-distance-levenshtein)
6. [Space Optimization](#6-space-optimization)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. String DP Overview

Two strings `s`, `t` → 2D DP where `dp[i][j]` compares prefixes `s[0..i-1]` and `t[0..j-1]`.

| Problem | dp[i][j] meaning |
|---------|------------------|
| LCS | length of LCS of prefixes |
| Edit distance | min ops to convert prefix s → prefix t |
| Delete ops | min deletes to make equal |

---

## 2. LCS Definition & Recurrence

If `s[i-1] === t[j-1]`: extend LCS → `dp[i][j] = dp[i-1][j-1] + 1`  
Else: skip one char → `dp[i][j] = max(dp[i-1][j], dp[i][j-1])`

```js
function longestCommonSubsequence(text1, text2) {
  const m = text1.length, n = text2.length;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (text1[i - 1] === text2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1] + 1;
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
      }
    }
  }
  return dp[m][n];
}
```

---

## 3. LCS Table Construction

Backtrack to reconstruct LCS string:

```js
function lcsString(a, b, dp) {
  let i = a.length, j = b.length, out = "";
  while (i > 0 && j > 0) {
    if (a[i - 1] === b[j - 1]) {
      out = a[i - 1] + out;
      i--; j--;
    } else if (dp[i - 1][j] >= dp[i][j - 1]) i--;
    else j--;
  }
  return out;
}
```

---

## 4. Delete Operation for Two Strings

Min deletions to make strings equal = `(m + n) - 2 * LCS`.

```js
function minDistance(word1, word2) {
  const lcs = longestCommonSubsequence(word1, word2);
  return word1.length + word2.length - 2 * lcs;
}
```

---

## 5. Edit Distance (Levenshtein)

Operations: insert, delete, replace — each cost 1.

```js
function minDistance(word1, word2) {
  const m = word1.length, n = word2.length;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (let i = 0; i <= m; i++) dp[i][0] = i;
  for (let j = 0; j <= n; j++) dp[0][j] = j;

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) {
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        dp[i][j] = 1 + Math.min(
          dp[i - 1][j],     // delete
          dp[i][j - 1],     // insert
          dp[i - 1][j - 1]  // replace
        );
      }
    }
  }
  return dp[m][n];
}
```

---

## 6. Space Optimization

Keep two rows (prev/current) → O(min(m,n)) space.

```js
function lcsSpaceOptimized(a, b) {
  if (a.length < b.length) [a, b] = [b, a];
  let prev = Array(b.length + 1).fill(0);
  let curr = Array(b.length + 1).fill(0);
  for (let i = 1; i <= a.length; i++) {
    for (let j = 1; j <= b.length; j++) {
      curr[j] = a[i - 1] === b[j - 1]
        ? prev[j - 1] + 1
        : Math.max(prev[j], curr[j - 1]);
    }
    [prev, curr] = [curr, prev.fill(0)];
  }
  return prev[b.length];
}
```

---

## 7. Interview Quick Index

| LC | Relation to LCS |
|----|-----------------|
| 1143 | LCS length |
| 583 | m + n - 2×LCS |
| 72 | edit distance (3 ops) |

---

**Next:** [Machine Coding](day53_machine_coding.md) · [LeetCode](day53_leetcode.md)
