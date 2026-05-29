# Day 57 — LeetCode (Stock DP)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 121 | [Best Time to Buy and Sell Stock](#1-best-time-to-buy-and-sell-stock-121) | Easy | One pass min |
| 122 | [Best Time to Buy and Sell Stock II](#2-best-time-to-buy-and-sell-stock-ii-122) | Medium | Greedy / DP |
| 123 | [Best Time to Buy and Sell Stock III](#3-best-time-to-buy-and-sell-stock-iii-123) | Hard | 2 transactions |

---

## Table of Contents

1. [Best Time I (121)](#1-best-time-i-121)
2. [Best Time II (122)](#2-best-time-ii-122)
3. [Best Time III (123)](#3-best-time-iii-123)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Best Time I (121)

### Solution

```js
function maxProfit(prices) {
  let min = Infinity, ans = 0;
  for (const p of prices) {
    min = Math.min(min, p);
    ans = Math.max(ans, p - min);
  }
  return ans;
}
```

---

## 2. Best Time II (122)

### Solution

```js
function maxProfit(prices) {
  let ans = 0;
  for (let i = 1; i < prices.length; i++)
    if (prices[i] > prices[i - 1]) ans += prices[i] - prices[i - 1];
  return ans;
}
```

---

## 3. Best Time III (123)

### Solution

```js
function maxProfit(prices) {
  let b1 = -Infinity, s1 = 0, b2 = -Infinity, s2 = 0;
  for (const p of prices) {
    b1 = Math.max(b1, -p);
    s1 = Math.max(s1, b1 + p);
    b2 = Math.max(b2, s1 - p);
    s2 = Math.max(s2, b2 + p);
  }
  return s2;
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) |

---

## 4. Pattern Cheat Sheet

| k | Approach |
|---|----------|
| 1 | track min price |
| ∞ | sum positive deltas |
| 2 | four rolling states |
| k | buy[k], sell[k] arrays |

---

**Prev/Next:** [Concepts](day57_concepts.md) · [Machine Coding](day57_machine_coding.md)
