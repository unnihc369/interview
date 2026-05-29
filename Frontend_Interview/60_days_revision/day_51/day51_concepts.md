# Day 51 — Tabulation (Bottom-Up DP)

**Week 8 — Dynamic Programming** · **Topics:** Bottom-up · 2D grid DP · Path counting · Min path sum

---

## Table of Contents

1. [Bottom-Up Tabulation](#1-bottom-up-tabulation)
2. [2D Grid DP Framework](#2-2d-grid-dp-framework)
3. [Unique Paths — Counting](#3-unique-paths--counting)
4. [Obstacles — Conditional Transitions](#4-obstacles--conditional-transitions)
5. [Min Path Sum — Optimization](#5-min-path-sum--optimization)
6. [Space Optimization](#6-space-optimization)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Bottom-Up Tabulation

Fill a table from **smallest subproblems** to the target.

```js
const dp = new Array(n + 1).fill(0);
dp[0] = base;
for (let i = 1; i <= n; i++) {
  dp[i] = transition(dp[i - 1], ...);
}
return dp[n];
```

| Top-down | Bottom-up |
|----------|-----------|
| Recursive + memo | Iterative loops |
| May skip unused states | Fills full table |
| Stack depth risk | No recursion limit |

---

## 2. 2D Grid DP Framework

`dp[i][j]` = answer for cell `(i,j)`.

```js
const m = grid.length, n = grid[0].length;
const dp = Array.from({ length: m }, () => Array(n).fill(0));

for (let i = 1; i < m; i++) {
  for (let j = 1; j < n; j++) {
    dp[i][j] = combine(dp[i - 1][j], dp[i][j - 1]);
  }
}
```

---

## 3. Unique Paths — Counting

Robot at `(0,0)` → `(m-1,n-1)`, only right/down.

```js
function uniquePaths(m, n) {
  const dp = Array.from({ length: m }, () => Array(n).fill(1));
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
    }
  }
  return dp[m - 1][n - 1];
}
```

---

## 4. Obstacles — Conditional Transitions

If `grid[i][j] === 1`: `dp[i][j] = 0`.

```js
function uniquePathsWithObstacles(grid) {
  const m = grid.length, n = grid[0].length;
  if (grid[0][0] === 1) return 0;
  const dp = Array.from({ length: m }, () => Array(n).fill(0));
  dp[0][0] = 1;
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (grid[i][j] === 1) { dp[i][j] = 0; continue; }
      if (i > 0) dp[i][j] += dp[i - 1][j];
      if (j > 0) dp[i][j] += dp[i][j - 1];
    }
  }
  return dp[m - 1][n - 1];
}
```

---

## 5. Min Path Sum — Optimization

```js
function minPathSum(grid) {
  const m = grid.length, n = grid[0].length;
  const dp = grid.map((row) => [...row]);
  for (let i = 1; i < m; i++) dp[i][0] += dp[i - 1][0];
  for (let j = 1; j < n; j++) dp[0][j] += dp[0][j - 1];
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[i][j] += Math.min(dp[i - 1][j], dp[i][j - 1]);
    }
  }
  return dp[m - 1][n - 1];
}
```

---

## 6. Space Optimization

```js
function uniquePaths(m, n) {
  const dp = Array(n).fill(1);
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) dp[j] += dp[j - 1];
  }
  return dp[n - 1];
}
```

---

## 7. Interview Quick Index

| Problem | dp[i][j] | Transition |
|---------|----------|------------|
| 62 | # paths to cell | sum top + left |
| 63 | # paths (obstacles) | 0 if blocked |
| 64 | min cost to cell | grid[i][j] + min(top, left) |

---

**Next:** [Machine Coding](day51_machine_coding.md) · [LeetCode](day51_leetcode.md)
