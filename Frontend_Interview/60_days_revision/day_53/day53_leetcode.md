# Day 53 — LeetCode (LCS / Edit Distance)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 1143 | [Longest Common Subsequence](#1-longest-common-subsequence-1143) | Medium | 2D string DP |
| 583 | [Delete Operation for Two Strings](#2-delete-operation-for-two-strings-583) | Medium | LCS transform |
| 72 | [Edit Distance](#3-edit-distance-72) | Medium | Levenshtein |

---

## Table of Contents

1. [Longest Common Subsequence (1143)](#1-longest-common-subsequence-1143)
2. [Delete Operation for Two Strings (583)](#2-delete-operation-for-two-strings-583)
3. [Edit Distance (72)](#3-edit-distance-72)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Longest Common Subsequence (1143)

### Problem (short)

Return length of longest subsequence common to both strings (not necessarily contiguous).

```
text1 = "abcde", text2 = "ace" → 3 ("ace")
```

### Solution

```js
function longestCommonSubsequence(text1, text2) {
  const m = text1.length, n = text2.length;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      dp[i][j] = text1[i - 1] === text2[j - 1]
        ? dp[i - 1][j - 1] + 1
        : Math.max(dp[i - 1][j], dp[i][j - 1]);
    }
  }
  return dp[m][n];
}
```

---

## 2. Delete Operation for Two Strings (583)

### Problem (short)

Min deletions (both strings) to make them equal.

### Hint 1

Keep LCS, delete the rest: `m + n - 2 * LCS`.

### Solution

```js
function minDistance(word1, word2) {
  const lcs = longestCommonSubsequence(word1, word2);
  return word1.length + word2.length - 2 * lcs;
}
```

---

## 3. Edit Distance (72)

### Problem (short)

Min insert/delete/replace to transform word1 → word2.

### Solution

```js
function minDistance(word1, word2) {
  const m = word1.length, n = word2.length;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (let i = 0; i <= m; i++) dp[i][0] = i;
  for (let j = 0; j <= n; j++) dp[0][j] = j;
  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      if (word1[i - 1] === word2[j - 1]) dp[i][j] = dp[i - 1][j - 1];
      else dp[i][j] = 1 + Math.min(dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]);
    }
  }
  return dp[m][n];
}
```

| Time | Space |
|------|-------|
| O(m×n) | O(m×n) |

---

## 4. Pattern Cheat Sheet

| Match? | Transition |
|--------|------------|
| Yes | diagonal + 1 (LCS) or diagonal (edit) |
| No | max(top, left) or min 3 neighbors + 1 |

---

**Prev/Next:** [Concepts](day53_concepts.md) · [Machine Coding](day53_machine_coding.md)
