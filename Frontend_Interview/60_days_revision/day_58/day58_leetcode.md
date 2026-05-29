# Day 58 — LeetCode (Interval DP)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 312 | [Burst Balloons](#1-burst-balloons-312) | Hard | Interval — last burst |
| 1039 | [Minimum Score Triangulation](#2-minimum-score-triangulation-1039) | Medium | Interval polygon |
| 546 | [Remove Boxes](#3-remove-boxes-546) | Hard | 3D interval DP |

---

## Table of Contents

1. [Burst Balloons (312)](#1-burst-balloons-312)
2. [Minimum Score Triangulation (1039)](#2-minimum-score-triangulation-1039)
3. [Remove Boxes (546)](#3-remove-boxes-546)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Burst Balloons (312)

### Hint 1

Think **last** balloon to burst in range (i,j), not first.

### Solution

```js
function maxCoins(nums) {
  const a = [1, ...nums, 1];
  const n = a.length;
  const dp = Array.from({ length: n }, () => Array(n).fill(0));
  for (let len = 3; len <= n; len++) {
    for (let i = 0; i + len - 1 < n; i++) {
      const j = i + len - 1;
      for (let k = i + 1; k < j; k++) {
        dp[i][j] = Math.max(dp[i][j], dp[i][k] + dp[k][j] + a[i] * a[k] * a[j]);
      }
    }
  }
  return dp[0][n - 1];
}
```

---

## 2. Minimum Score Triangulation (1039)

### Solution

```js
function minScoreTriangulation(values) {
  const n = values.length;
  const dp = Array.from({ length: n }, () => Array(n).fill(0));
  for (let len = 3; len <= n; len++) {
    for (let i = 0; i + len - 1 < n; i++) {
      const j = i + len - 1;
      dp[i][j] = Infinity;
      for (let k = i + 1; k < j; k++) {
        dp[i][j] = Math.min(dp[i][j], dp[i][k] + dp[k][j] + values[i] * values[k] * values[j]);
      }
    }
  }
  return dp[0][n - 1];
}
```

---

## 3. Remove Boxes (546)

### Hint

`dp[l][r][k]` — max points for boxes l..r with k extra same-color boxes to the left.

### Solution sketch

```js
function removeBoxes(boxes) {
  const n = boxes.length;
  const memo = new Map();
  function dp(l, r, k) {
    if (l > r) return 0;
    const key = `${l},${r},${k}`;
    if (memo.has(key)) return memo.get(key);
    let ans = (k + 1) * (k + 1) * boxes[r] + dp(l, r - 1, 0);
    for (let i = l; i < r; i++) {
      if (boxes[i] === boxes[r]) {
        ans = Math.max(ans, dp(l, i, k + 1) + dp(i + 1, r - 1, 0));
      }
    }
    memo.set(key, ans);
    return ans;
  }
  return dp(0, n - 1, 0);
}
```

---

## 4. Pattern Cheat Sheet

| Step | Action |
|------|--------|
| 1 | Define dp on [i,j] |
| 2 | Loop len = 2..n |
| 3 | Try all split k |
| 4 | Combine sub-intervals + cost at k |

---

**Prev/Next:** [Concepts](day58_concepts.md) · [Machine Coding](day58_machine_coding.md)
