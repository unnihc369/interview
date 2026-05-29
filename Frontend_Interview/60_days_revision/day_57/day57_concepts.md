# Day 57 — DP Rolling Array (Stock)

**Week 8 — Dynamic Programming** · **Topics:** Space optimization · Rolling variables · Stock series · k transactions

---

## Table of Contents

1. [Rolling Array Technique](#1-rolling-array-technique)
2. [Best Time I — One Transaction](#2-best-time-i--one-transaction)
3. [Best Time II — Unlimited Transactions](#3-best-time-ii--unlimited-transactions)
4. [Best Time III — Two Transactions](#4-best-time-iii--two-transactions)
5. [General k Transactions](#5-general-k-transactions)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Rolling Array Technique

Instead of `dp[i][...]` table, keep only previous row or few scalars:

```js
// Before: dp[i] depends on dp[i-1]
let prev = base;
for (let i = 1; i <= n; i++) {
  const curr = transition(prev);
  prev = curr;
}
```

Stock problems use states: **hold** vs **cash** (not holding).

---

## 2. Best Time I — One Transaction

Track min price so far; max profit = max(price[i] - minSoFar).

```js
function maxProfit(prices) {
  let minPrice = Infinity, maxP = 0;
  for (const p of prices) {
    minPrice = Math.min(minPrice, p);
    maxP = Math.max(maxP, p - minPrice);
  }
  return maxP;
}
```

O(1) space — no DP table needed.

---

## 3. Best Time II — Unlimited Transactions

Capture every upward move.

```js
function maxProfit(prices) {
  let profit = 0;
  for (let i = 1; i < prices.length; i++) {
    if (prices[i] > prices[i - 1]) profit += prices[i] - prices[i - 1];
  }
  return profit;
}
```

Equivalent DP: `cash = max(cash, hold + price)`, `hold = max(hold, cash - price)`.

---

## 4. Best Time III — Two Transactions

```js
function maxProfit(prices) {
  let buy1 = -Infinity, sell1 = 0;
  let buy2 = -Infinity, sell2 = 0;
  for (const p of prices) {
    buy1 = Math.max(buy1, -p);
    sell1 = Math.max(sell1, buy1 + p);
    buy2 = Math.max(buy2, sell1 - p);
    sell2 = Math.max(sell2, buy2 + p);
  }
  return sell2;
}
```

State machine: buy1 → sell1 → buy2 → sell2.

---

## 5. General k Transactions

```js
function maxProfit(k, prices) {
  if (k >= prices.length / 2) return maxProfitUnlimited(prices);
  let buy = Array(k + 1).fill(-Infinity);
  let sell = Array(k + 1).fill(0);
  for (const p of prices) {
    for (let j = k; j >= 1; j--) {
      sell[j] = Math.max(sell[j], buy[j] + p);
      buy[j] = Math.max(buy[j], sell[j - 1] - p);
    }
  }
  return sell[k];
}
```

---

## 6. Interview Quick Index

| LC | Transactions | Space |
|----|--------------|-------|
| 121 | 1 | O(1) |
| 122 | unlimited | O(1) |
| 123 | 2 | O(1) four vars |

---

**Next:** [Machine Coding](day57_machine_coding.md) · [LeetCode](day57_leetcode.md)
