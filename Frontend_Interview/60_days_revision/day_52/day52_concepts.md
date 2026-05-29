# Day 52 — Knapsack (0/1 & Unbounded)

**Week 8 — Dynamic Programming** · **Topics:** 0/1 knapsack · Unbounded knapsack · Subset sum · Coin change

---

## Table of Contents

1. [Knapsack Family](#1-knapsack-family)
2. [0/1 Knapsack](#2-01-knapsack)
3. [Partition Equal Subset Sum](#3-partition-equal-subset-sum)
4. [Target Sum (+ / −)](#4-target-sum--)
5. [Unbounded Knapsack — Coin Change](#5-unbounded-knapsack--coin-change)
6. [Iteration Order Matters](#6-iteration-order-matters)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Knapsack Family

| Variant | Use each item | Question |
|---------|---------------|----------|
| **0/1** | At most once | Subset sum, partition |
| **Unbounded** | Unlimited times | Coin change (min coins) |
| **Bounded** | Limited count | Rare in FE interviews |

State: `dp[capacity]` or `dp[i][w]` = best value achievable.

---

## 2. 0/1 Knapsack

`dp[w]` = can we make sum `w`?

```js
function subsetSum(nums, target) {
  const dp = Array(target + 1).fill(false);
  dp[0] = true;
  for (const num of nums) {
    for (let w = target; w >= num; w--) { // reverse!
      dp[w] = dp[w] || dp[w - num];
    }
  }
  return dp[target];
}
```

**Reverse loop** prevents using same item twice in one pass.

---

## 3. Partition Equal Subset Sum

Can array split into two equal-sum subsets? → subset sum to `total/2`.

```js
function canPartition(nums) {
  const sum = nums.reduce((a, b) => a + b, 0);
  if (sum % 2) return false;
  return subsetSum(nums, sum / 2);
}
```

---

## 4. Target Sum (+ / −)

Assign `+` or `-` to each number to reach target.

Transform: `sum(positive) - sum(negative) = target`  
→ find subset with sum `(target + total) / 2`.

```js
function findTargetSumWays(nums, target) {
  const total = nums.reduce((a, b) => a + b, 0);
  if ((target + total) % 2 || Math.abs(target) > total) return 0;
  const sub = (target + total) / 2;
  const dp = Array(sub + 1).fill(0);
  dp[0] = 1;
  for (const n of nums) {
    for (let s = sub; s >= n; s--) {
      dp[s] += dp[s - n];
    }
  }
  return dp[sub];
}
```

---

## 5. Unbounded Knapsack — Coin Change

Min coins to make amount — each coin reusable.

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

**Forward loop** on amount allows reuse.

---

## 6. Iteration Order Matters

| Problem | Inner loop direction |
|---------|---------------------|
| 0/1 knapsack | `w` from high → low |
| Unbounded | `a` from low → high |
| Count ways 0/1 | high → low |
| Count ways unbounded | low → high |

---

## 7. Interview Quick Index

| LC | Pattern | dp meaning |
|----|---------|------------|
| 416 | 0/1 boolean | can make sum/2 |
| 494 | 0/1 count ways | # subsets sum to S |
| 322 | unbounded min | min coins for amount |

---

**Next:** [Machine Coding](day52_machine_coding.md) · [LeetCode](day52_leetcode.md)
