# Day 50 — LeetCode (Memoization / 1D DP)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 509 | [Fibonacci Number](#1-fibonacci-number-509) | Easy | Memo / tabulation |
| 1137 | [N-th Tribonacci Number](#2-n-th-tribonacci-number-1137) | Easy | 1D DP rolling |
| 746 | [Min Cost Climbing Stairs](#3-min-cost-climbing-stairs-746) | Easy | Memo on index |

---

## Table of Contents

1. [Fibonacci Number (509)](#1-fibonacci-number-509)
2. [N-th Tribonacci Number (1137)](#2-n-th-tribonacci-number-1137)
3. [Min Cost Climbing Stairs (746)](#3-min-cost-climbing-stairs-746)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Fibonacci Number (509)

### Problem (short)

Return F(n) where F(0)=0, F(1)=1, F(n)=F(n-1)+F(n-2).

```
Input: n = 4 → Output: 3
```

### Hint 1

Classic memo: `dp(i)` = F(i), base `i<=1`.

### Hint 2

Bottom-up also works: fill array from 0..n.

### Solution — Memoization

```js
function fib(n) {
  const memo = new Map();
  function dp(i) {
    if (i <= 1) return i;
    if (memo.has(i)) return memo.get(i);
    const v = dp(i - 1) + dp(i - 2);
    memo.set(i, v);
    return v;
  }
  return dp(n);
}
```

### Solution — Bottom-up O(1) space

```js
function fib(n) {
  if (n <= 1) return n;
  let a = 0, b = 1;
  for (let i = 2; i <= n; i++) [a, b] = [b, a + b];
  return b;
}
```

| Time | Space |
|------|-------|
| O(n) | O(n) memo / O(1) iterative |

---

## 2. N-th Tribonacci Number (1137)

### Problem (short)

T(0)=0, T(1)=1, T(2)=1, T(n)=T(n-1)+T(n-2)+T(n-3).

### Hint 1

Extend Fibonacci: three previous values.

### Solution

```js
function tribonacci(n) {
  if (n === 0) return 0;
  if (n <= 2) return 1;
  let a = 0, b = 1, c = 1;
  for (let i = 3; i <= n; i++) {
    const next = a + b + c;
    a = b;
    b = c;
    c = next;
  }
  return c;
}
```

### Memo version

```js
function tribonacci(n) {
  const memo = {};
  function dp(i) {
    if (i === 0) return 0;
    if (i <= 2) return 1;
    if (memo[i] != null) return memo[i];
    return (memo[i] = dp(i - 1) + dp(i - 2) + dp(i - 3));
  }
  return dp(n);
}
```

---

## 3. Min Cost Climbing Stairs (746)

### Problem (short)

Each step has a cost. Start at index 0 or 1. Pay cost[i] when **stepping on** i. Reach top (index n) with minimum cost.

```
cost = [10,15,20] → Output: 15 (0→2 or 1→2)
```

### Hint 1

`dp(i)` = min cost to reach step i.

### Hint 2

`dp(i) = cost[i] + min(dp(i-1), dp(i-2))`. Answer: `min(dp(n-1), dp(n-2))`.

### Solution — Top-down

```js
function minCostClimbingStairs(cost) {
  const n = cost.length;
  const memo = new Map();

  function dp(i) {
    if (i < 0) return 0;
    if (i === 0 || i === 1) return cost[i];
    if (memo.has(i)) return memo.get(i);
    const v = cost[i] + Math.min(dp(i - 1), dp(i - 2));
    memo.set(i, v);
    return v;
  }

  return Math.min(dp(n - 1), dp(n - 2));
}
```

### Solution — Bottom-up

```js
function minCostClimbingStairs(cost) {
  const n = cost.length;
  const dp = new Array(n);
  dp[0] = cost[0];
  dp[1] = cost[1];
  for (let i = 2; i < n; i++) {
    dp[i] = cost[i] + Math.min(dp[i - 1], dp[i - 2]);
  }
  return Math.min(dp[n - 1], dp[n - 2]);
}
```

| Time | Space |
|------|-------|
| O(n) | O(n) |

---

## 4. Pattern Cheat Sheet

| Problem | State | Base | Transition |
|---------|-------|------|------------|
| 509 | index `n` | 0,1 | sum of 2 |
| 1137 | index `n` | 0,1,1 | sum of 3 |
| 746 | step `i` | cost[0], cost[1] | cost + min(prev 2) |

**Interview flow:** recursive idea → overlapping subproblems → memo → optional space optimize.

---

**Prev/Next:** [Concepts](day50_concepts.md) · [Machine Coding](day50_machine_coding.md)
