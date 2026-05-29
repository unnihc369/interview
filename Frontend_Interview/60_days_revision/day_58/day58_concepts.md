# Day 58 — DP Interval (Burst Balloons)

**Week 8 — Dynamic Programming** · **Topics:** Interval DP · Burst balloons · Matrix chain · Divide & conquer on range

---

## Table of Contents

1. [Interval DP Framework](#1-interval-dp-framework)
2. [Burst Balloons (312)](#2-burst-balloons-312)
3. [Minimum Score Triangulation (1039)](#3-minimum-score-triangulation-1039)
4. [Remove Boxes (546)](#4-remove-boxes-546)
5. [Fill Order — Length Increasing](#5-fill-order--length-increasing)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Interval DP Framework

`dp[i][j]` = optimal answer for subarray/subproblem on range `[i, j]`.

Fill by **increasing interval length**:

```js
for (let len = 2; len <= n; len++) {
  for (let i = 0; i + len - 1 < n; i++) {
    const j = i + len - 1;
    dp[i][j] = combine over split point k in (i..j);
  }
}
```

---

## 2. Burst Balloons (312)

Burst **last** balloon k in range — coins = left × mid × right.

```js
function maxCoins(nums) {
  const arr = [1, ...nums, 1];
  const n = arr.length;
  const dp = Array.from({ length: n }, () => Array(n).fill(0));

  for (let len = 3; len <= n; len++) {
    for (let i = 0; i + len - 1 < n; i++) {
      const j = i + len - 1;
      for (let k = i + 1; k < j; k++) {
        dp[i][j] = Math.max(
          dp[i][j],
          dp[i][k] + dp[k][j] + arr[i] * arr[k] * arr[j]
        );
      }
    }
  }
  return dp[0][n - 1];
}
```

---

## 3. Minimum Score Triangulation (1039)

Split polygon triangle at k; score = product of three vertices.

```js
function minScoreTriangulation(values) {
  const n = values.length;
  const dp = Array.from({ length: n }, () => Array(n).fill(0));
  for (let len = 3; len <= n; len++) {
    for (let i = 0; i + len - 1 < n; i++) {
      const j = i + len - 1;
      dp[i][j] = Infinity;
      for (let k = i + 1; k < j; k++) {
        dp[i][j] = Math.min(
          dp[i][j],
          dp[i][k] + dp[k][j] + values[i] * values[k] * values[j]
        );
      }
    }
  }
  return dp[0][n - 1];
}
```

---

## 4. Remove Boxes (546)

Hard: `dp[i][j][k]` = max points removing boxes[i..j] with k same-color boxes attached to left of i.

State explosion — know the pattern; full solution uses 3D memo.

---

## 5. Fill Order — Length Increasing

Always iterate `len` from small to large so `dp[i][k]` and `dp[k][j]` are ready before `dp[i][j]`.

---

## 6. Interview Quick Index

| LC | Split at | Cost at k |
|----|----------|-----------|
| 312 | last burst k | arr[i]*arr[k]*arr[j] |
| 1039 | triangle k | v[i]*v[k]*v[j] |
| 546 | 3D interval+count | box color runs |

---

**Next:** [Machine Coding](day58_machine_coding.md) · [LeetCode](day58_leetcode.md)
