# Day 2 - Arrays & Interview DSA Patterns

---

# 1. Best Time to Buy and Sell Stock

## Problem Statement

Given an array where:

```text
prices[i] = stock price on ith day
```

Find maximum profit by:

- Buying once
- Selling once

You must buy before selling.

---

# Example

```text
Input:
[7,1,5,3,6,4]

Output:
5
```

Explanation:

- Buy at 1
- Sell at 6
- Profit = 5

---

# Brute Force Approach

## Idea

Try every pair:

```text
Buy at i
Sell at j
```

Find maximum profit.

---

# Time Complexity

```text
O(n²)
```

---

# Optimized Approach

## Core Idea

Track:

- Minimum price seen so far
- Maximum profit

---

# Intuition

If current price is greater than minimum price:

```text
profit = currentPrice - minPrice
```

Update maximum profit.

---

# Java Solution

```java
class Solution {
    public int maxProfit(int[] prices) {

        int minPrice = Integer.MAX_VALUE;
        int maxProfit = 0;

        for(int price : prices) {

            if(price < minPrice) {
                minPrice = price;
            }

            int profit = price - minPrice;

            if(profit > maxProfit) {
                maxProfit = profit;
            }
        }

        return maxProfit;
    }
}
```

---

# JavaScript Solution

```javascript
function maxProfit(prices) {
  let minPrice = Infinity;
  let maxProfit = 0;

  for (let price of prices) {
    if (price < minPrice) {
      minPrice = price;
    }

    let profit = price - minPrice;

    if (profit > maxProfit) {
      maxProfit = profit;
    }
  }

  return maxProfit;
}
```

---

# Time Complexity

| Operation | Complexity |
| --------- | ---------- |
| Traversal | O(n)       |
| Space     | O(1)       |

---

# Frequently Asked Interview Questions

## Why do we track minimum price?

Because best profit comes from:

```text
current price - smallest previous price
```

---

## Can stock be sold before buying?

❌ No

Must buy first.

---

## What if prices always decrease?

Return:

```text
0
```

No profit possible.

---

# Dry Run

```text
prices = [7,1,5,3,6,4]
```

| Price | Min Price | Profit | Max Profit |
| ----- | --------- | ------ | ---------- |
| 7     | 7         | 0      | 0          |
| 1     | 1         | 0      | 0          |
| 5     | 1         | 4      | 4          |
| 3     | 1         | 2      | 4          |
| 6     | 1         | 5      | 5          |
| 4     | 1         | 3      | 5          |

---

# Pattern Used

## Sliding Window / Greedy

---

# 2. Maximum Subarray (Kadane’s Algorithm)

---

# Problem Statement

Find contiguous subarray with maximum sum.

---

# Example

```text
Input:
[-2,1,-3,4,-1,2,1,-5,4]

Output:
6
```

Subarray:

```text
[4,-1,2,1]
```

---

# Brute Force

Generate all subarrays.

Time Complexity:

```text
O(n²)
```

---

# Kadane's Algorithm

## Core Idea

If current sum becomes negative:

```text
Discard it
```

Because negative sum reduces future sum.

---

# Formula

```text
currentSum = max(nums[i], currentSum + nums[i])
```

---

# Java Solution

```java
class Solution {
    public int maxSubArray(int[] nums) {

        int currentSum = nums[0];
        int maxSum = nums[0];

        for(int i = 1; i < nums.length; i++) {

            currentSum = Math.max(nums[i], currentSum + nums[i]);

            maxSum = Math.max(maxSum, currentSum);
        }

        return maxSum;
    }
}
```

---

# JavaScript Solution

```javascript
function maxSubArray(nums) {
  let currentSum = nums[0];
  let maxSum = nums[0];

  for (let i = 1; i < nums.length; i++) {
    currentSum = Math.max(nums[i], currentSum + nums[i]);

    maxSum = Math.max(maxSum, currentSum);
  }

  return maxSum;
}
```

---

# Dry Run

```text
[-2,1,-3,4,-1,2,1,-5,4]
```

| Num | Current Sum | Max Sum |
| --- | ----------- | ------- |
| -2  | -2          | -2      |
| 1   | 1           | 1       |
| -3  | -2          | 1       |
| 4   | 4           | 4       |
| -1  | 3           | 4       |
| 2   | 5           | 5       |
| 1   | 6           | 6       |
| -5  | 1           | 6       |
| 4   | 5           | 6       |

---

# Time Complexity

| Operation | Complexity |
| --------- | ---------- |
| Traversal | O(n)       |
| Space     | O(1)       |

---

# Frequently Asked Interview Questions

## Why reset when sum becomes negative?

Because negative prefix decreases future sums.

---

## Can Kadane work for all negative arrays?

✅ Yes

Initialize with first element.

---

# Pattern Used

## Dynamic Programming

---

# 3. Contains Duplicate

---

# Problem Statement

Check whether array contains duplicates.

---

# Example

```text
Input:
[1,2,3,1]

Output:
true
```

---

# Optimized Approach

Use HashSet.

---

# Java Solution

```java
import java.util.*;

class Solution {
    public boolean containsDuplicate(int[] nums) {

        HashSet<Integer> set = new HashSet<>();

        for(int num : nums) {

            if(set.contains(num)) {
                return true;
            }

            set.add(num);
        }

        return false;
    }
}
```

---

# JavaScript Solution

```javascript
function containsDuplicate(nums) {
  let set = new Set();

  for (let num of nums) {
    if (set.has(num)) {
      return true;
    }

    set.add(num);
  }

  return false;
}
```

---

# Time Complexity

| Operation      | Complexity |
| -------------- | ---------- |
| Traversal      | O(n)       |
| Set Operations | O(1)       |
| Space          | O(n)       |

---

# Frequently Asked Interview Questions

## Why use HashSet?

For:

```text
O(1)
```

lookup.

---

## Alternative Solution?

Sort array.

But complexity becomes:

```text
O(n log n)
```

---

# Pattern Used

## Hashing

---

# Interview Revision Notes

| Problem                       | Pattern     |
| ----------------------------- | ----------- |
| Best Time to Buy & Sell Stock | Greedy      |
| Maximum Subarray              | Kadane / DP |
| Contains Duplicate            | Hashing     |

---

# Common Interview Follow-Ups

## Best Time to Buy and Sell Stock II

Multiple transactions allowed.

---

## Maximum Product Subarray

Variation of Kadane.

---

## Contains Nearby Duplicate

Uses sliding window + hashing.
