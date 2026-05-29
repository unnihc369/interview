# Day 59 — DP String Edit Distance

**Week 8 — Dynamic Programming** · **Topics:** Distinct subsequences · Interleaving · Edit distance review · Autocorrect

---

## Table of Contents

1. [String DP Recap](#1-string-dp-recap)
2. [Distinct Subsequences (115)](#2-distinct-subsequences-115)
3. [Interleaving String (97)](#3-interleaving-string-97)
4. [Delete Operations Review (583)](#4-delete-operations-review-583)
5. [Autocorrect Cost Model](#5-autocorrect-cost-model)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. String DP Recap

Two pointers on prefixes → 2D table `dp[i][j]` comparing `s[0..i-1]` and `t[0..j-1]`.

| Problem | dp[i][j] |
|---------|----------|
| LCS / edit | min/max/count ops |
| Distinct subseq | # ways s forms t |
| Interleaving | can s1+s2 form s3 |

---

## 2. Distinct Subsequences (115)

Count ways `s` has subsequence equal to `t`.

```js
function numDistinct(s, t) {
  const m = s.length, n = t.length;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(0));
  for (let i = 0; i <= m; i++) dp[i][0] = 1;

  for (let i = 1; i <= m; i++) {
    for (let j = 1; j <= n; j++) {
      dp[i][j] = dp[i - 1][j];
      if (s[i - 1] === t[j - 1]) dp[i][j] += dp[i - 1][j - 1];
    }
  }
  return dp[m][n];
}
```

If match: use char (+ dp[i-1][j-1]) or skip (+ dp[i-1][j]).

---

## 3. Interleaving String (97)

Can `s3` be formed by interleaving `s1` and `s2`?

```js
function isInterleave(s1, s2, s3) {
  const m = s1.length, n = s2.length;
  if (m + n !== s3.length) return false;
  const dp = Array.from({ length: m + 1 }, () => Array(n + 1).fill(false));
  dp[0][0] = true;

  for (let i = 0; i <= m; i++) {
    for (let j = 0; j <= n; j++) {
      if (i > 0 && s1[i - 1] === s3[i + j - 1]) dp[i][j] ||= dp[i - 1][j];
      if (j > 0 && s2[j - 1] === s3[i + j - 1]) dp[i][j] ||= dp[i][j - 1];
    }
  }
  return dp[m][n];
}
```

---

## 4. Delete Operations Review (583)

```js
function minDistance(word1, word2) {
  const lcs = longestCommonSubsequence(word1, word2);
  return word1.length + word2.length - 2 * lcs;
}
```

---

## 5. Autocorrect Cost Model

Keyboard distance + edit distance → suggest minimum-cost correction:

```js
function suggestionCost(typed, candidate) {
  return minDistance(typed, candidate); // Levenshtein
}

function rankSuggestions(typed, dictionary) {
  return dictionary
    .map((word) => ({ word, cost: suggestionCost(typed, word) }))
    .filter((x) => x.cost <= 2)
    .sort((a, b) => a.cost - b.cost || a.word.localeCompare(b.word));
}
```

---

## 6. Interview Quick Index

| LC | dp[i][j] meaning |
|----|------------------|
| 115 | # subseq ways |
| 97 | interleave possible? |
| 583 | m+n-2×LCS (review) |

---

**Next:** [Machine Coding](day59_machine_coding.md) · [LeetCode](day59_leetcode.md)
