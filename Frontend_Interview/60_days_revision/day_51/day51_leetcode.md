# Day 51 — LeetCode (2D Grid DP)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 62 | [Unique Paths](#1-unique-paths-62) | Medium | 2D count paths |
| 63 | [Unique Paths II](#2-unique-paths-ii-63) | Medium | Obstacles |
| 64 | [Minimum Path Sum](#3-minimum-path-sum-64) | Medium | Min cost grid |

---

## Table of Contents

1. [Unique Paths (62)](#1-unique-paths-62)
2. [Unique Paths II (63)](#2-unique-paths-ii-63)
3. [Minimum Path Sum (64)](#3-minimum-path-sum-64)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Unique Paths (62)

### Problem (short)

`m×n` grid. Start top-left, end bottom-right. Move only right or down. Count paths.

```
Input: m = 3, n = 7 → Output: 28
```

### Hint 1

`dp[i][j] = dp[i-1][j] + dp[i][j-1]`.

### Solution

```js
function uniquePaths(m, n) {
  const dp = Array(n).fill(1);
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      dp[j] += dp[j - 1];
    }
  }
  return dp[n - 1];
}
```

| Time | Space |
|------|-------|
| O(m×n) | O(n) |

---

## 2. Unique Paths II (63)

### Problem (short)

Grid with obstacles (`1` = blocked). Count paths.

### Hint 1

If `grid[i][j]===1`, `dp[i][j]=0`.

### Solution

```js
function uniquePathsWithObstacles(grid) {
  const m = grid.length, n = grid[0].length;
  if (grid[0][0] === 1) return 0;
  const dp = Array(n).fill(0);
  dp[0] = 1;
  for (let i = 0; i < m; i++) {
    for (let j = 0; j < n; j++) {
      if (grid[i][j] === 1) dp[j] = 0;
      else if (j > 0) dp[j] += dp[j - 1];
    }
  }
  return dp[n - 1];
}
```

---

## 3. Minimum Path Sum (64)

### Problem (short)

Non-negative grid. Min sum path top-left to bottom-right.

### Solution

```js
function minPathSum(grid) {
  const m = grid.length, n = grid[0].length;
  for (let i = 1; i < m; i++) grid[i][0] += grid[i - 1][0];
  for (let j = 1; j < n; j++) grid[0][j] += grid[0][j - 1];
  for (let i = 1; i < m; i++) {
    for (let j = 1; j < n; j++) {
      grid[i][j] += Math.min(grid[i - 1][j], grid[i][j - 1]);
    }
  }
  return grid[m - 1][n - 1];
}
```

---

## 4. Pattern Cheat Sheet

| Problem | dp meaning | Blocked cell |
|---------|------------|--------------|
| 62 | path count | N/A |
| 63 | path count | dp=0 |
| 64 | min cost | N/A |

**Grid DP checklist:** init row 0 / col 0 → double loop → answer at bottom-right.

---

**Prev/Next:** [Concepts](day51_concepts.md) · [Machine Coding](day51_machine_coding.md)
