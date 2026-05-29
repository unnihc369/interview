# Day 52 — LeetCode (Knapsack)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 416 | [Partition Equal Subset Sum](#1-partition-equal-subset-sum-416) | Medium | 0/1 subset sum |
| 494 | [Target Sum](#2-target-sum-494) | Medium | Count subsets |
| 322 | [Coin Change](#3-coin-change-322) | Medium | Unbounded min |

---

## Table of Contents

1. [Partition Equal Subset Sum (416)](#1-partition-equal-subset-sum-416)
2. [Target Sum (494)](#2-target-sum-494)
3. [Coin Change (322)](#3-coin-change-322)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Partition Equal Subset Sum (416)

### Problem (short)

Can nums be partitioned into two subsets with equal sum?

### Hint 1

Target = total/2. 0/1 knapsack: can we reach target?

### Solution

```js
function canPartition(nums) {
  const sum = nums.reduce((a, b) => a + b, 0);
  if (sum % 2) return false;
  const target = sum / 2;
  const dp = Array(target + 1).fill(false);
  dp[0] = true;
  for (const n of nums) {
    for (let t = target; t >= n; t--) {
      dp[t] = dp[t] || dp[t - n];
    }
  }
  return dp[target];
}
```

---

## 2. Target Sum (494)

### Problem (short)

Assign `+` or `-` to each integer to reach `target`. Count ways.

### Hint 1

Reduce to subset sum: `(target + sum) / 2` must be integer.

### Solution

```js
function findTargetSumWays(nums, target) {
  const sum = nums.reduce((a, b) => a + b, 0);
  if ((target + sum) % 2 !== 0 || Math.abs(target) > sum) return 0;
  const sub = (target + sum) / 2;
  let dp = Array(sub + 1).fill(0);
  dp[0] = 1;
  for (const n of nums) {
    for (let s = sub; s >= n; s--) dp[s] += dp[s - n];
  }
  return dp[sub];
}
```

---

## 3. Coin Change (322)

### Problem (short)

Fewest coins to make `amount`. Return -1 if impossible.

### Solution

```js
function coinChange(coins, amount) {
  const dp = Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  for (let a = 1; a <= amount; a++) {
    for (const c of coins) {
      if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
    }
  }
  return dp[amount] === Infinity ? -1 : dp[amount];
}
```

| Time | Space |
|------|-------|
| O(amount × coins) | O(amount) |

---

## 4. Pattern Cheat Sheet

| Problem | Loop order | dp type |
|---------|------------|---------|
| 416 | w descending | boolean |
| 494 | w descending | count |
| 322 | a ascending | min |

---

**Prev/Next:** [Concepts](day52_concepts.md) · [Machine Coding](day52_machine_coding.md)
