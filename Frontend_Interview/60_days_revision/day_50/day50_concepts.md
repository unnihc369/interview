# Day 50 — Memoization (Top-Down DP)

**Week 8 — Dynamic Programming** · **Topics:** Top-down · Memo table · State definition · Overlapping subproblems

---

## Table of Contents

1. [What Is Dynamic Programming?](#1-what-is-dynamic-programming)
2. [Top-Down vs Bottom-Up](#2-top-down-vs-bottom-up)
3. [Memoization Pattern](#3-memoization-pattern)
4. [Fibonacci — The Canonical Example](#4-fibonacci--the-canonical-example)
5. [Climbing Stairs State Machine](#5-climbing-stairs-state-machine)
6. [When to Use Memoization](#6-when-to-use-memoization)
7. [JavaScript Memo Helpers](#7-javascript-memo-helpers)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. What Is Dynamic Programming?

DP applies when a problem has:

| Property | Meaning |
|----------|---------|
| **Optimal substructure** | Optimal answer built from optimal sub-answers |
| **Overlapping subproblems** | Same subproblem solved many times |

**Without memo:** exponential recursion (Fibonacci).  
**With memo:** each unique state computed once → polynomial time.

---

## 2. Top-Down vs Bottom-Up

| Approach | Also called | How |
|----------|---------------|-----|
| **Top-down** | Memoization | Recursion + cache lookup |
| **Bottom-up** | Tabulation | Iterative fill of DP table (Day 51) |

Top-down is often faster to **write in interviews** — start with brute recursion, add cache.

---

## 3. Memoization Pattern

```js
function solve(input) {
  const memo = new Map(); // or Array, or object

  function dp(state) {
    const key = serialize(state); // e.g. `${i},${j}` or just `n`
    if (memo.has(key)) return memo.get(key);

    // base case
    if (isBase(state)) return baseValue;

    const result = combine(dp(nextState1), dp(nextState2));
    memo.set(key, result);
    return result;
  }

  return dp(initialState);
}
```

### Key decisions

1. **What is the state?** (index, remaining capacity, position)
2. **What does `dp(state)` return?** (count, min cost, boolean)
3. **Transition:** how do sub-states combine?

---

## 4. Fibonacci — The Canonical Example

### Naive — O(2^n)

```js
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}
```

### Memoized — O(n)

```js
function fibMemo(n, memo = new Map()) {
  if (n <= 1) return n;
  if (memo.has(n)) return memo.get(n);
  const val = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
  memo.set(n, val);
  return val;
}
```

### Closure style (interview-friendly)

```js
function fib(n) {
  const memo = Array(n + 1).fill(-1);

  function dp(i) {
    if (i <= 1) return i;
    if (memo[i] !== -1) return memo[i];
    memo[i] = dp(i - 1) + dp(i - 2);
    return memo[i];
  }

  return dp(n);
}
```

---

## 5. Climbing Stairs State Machine

Reach step `n` from step `0`. Each move: +1 or +2.

```
dp(n) = dp(n-1) + dp(n-2)   // same recurrence as Fibonacci
dp(0) = 1, dp(1) = 1
```

```js
function climbStairs(n) {
  const memo = {};
  function dp(i) {
    if (i <= 1) return 1;
    if (memo[i] != null) return memo[i];
    memo[i] = dp(i - 1) + dp(i - 2);
    return memo[i];
  }
  return dp(n);
}
```

### Min cost variant (LC 746)

State = step index. Transition adds `cost[i]` when landing on step `i`.

```js
function minCostClimbingStairs(cost) {
  const n = cost.length;
  const memo = new Map();

  function dp(i) {
    if (i >= n) return 0;
    if (memo.has(i)) return memo.get(i);
    const val = cost[i] + Math.min(dp(i + 1), dp(i + 2));
    memo.set(i, val);
    return val;
  }

  return Math.min(dp(0), dp(1));
}
```

---

## 6. When to Use Memoization

| Signal | Example |
|--------|---------|
| "Count ways" / "minimum cost" | Climbing stairs, coin change |
| Recursive definition obvious | Fibonacci, tree DP |
| Sparse state space | Only some `(i,j)` visited |
| Need actual path reconstruction | Store parent pointers alongside memo |

**Avoid** when state space is huge and dense — prefer bottom-up or space optimization (Day 57).

---

## 7. JavaScript Memo Helpers

### Map with tuple key

```js
const key = (i, j) => `${i},${j}`;
memo.set(key(i, j), val);
```

### `useMemo` in React (not DP, but related)

Cache expensive pure computations by dependency array — different from algorithmic memoization but same idea.

### Generic memoize utility

```js
function memoize(fn, keyFn = (...args) => JSON.stringify(args)) {
  const cache = new Map();
  return (...args) => {
    const k = keyFn(...args);
    if (cache.has(k)) return cache.get(k);
    const result = fn(...args);
    cache.set(k, result);
    return result;
  };
}

const fibFast = memoize((n) => {
  if (n <= 1) return n;
  return fibFast(n - 1) + fibFast(n - 2);
});
```

---

## 8. Interview Quick Index

| Problem | State | Recurrence |
|---------|-------|------------|
| Fibonacci (509) | `n` | `f(n-1)+f(n-2)` |
| Tribonacci (1137) | `n` | sum of last 3 |
| Min cost stairs (746) | step `i` | `cost[i]+min(dp(i+1),dp(i+2))` |

**Template sentence:** "I'll define `dp(i)` as the answer for subproblem X, memoize on state, and combine children with min/max/sum."

---

**Next:** [Day 50 Machine Coding](day50_machine_coding.md) · [Day 50 LeetCode](day50_leetcode.md)
