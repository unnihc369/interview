# Day 18 — Dynamic Programming Patterns

**Topics:** Longest Increasing Subsequence · Coin Change · Word Break

---

# 1. Longest Increasing Subsequence (LIS)

## Problem Statement

Given an integer array `nums`, return the length of the longest strictly increasing subsequence.

Subsequence: elements in order, not necessarily contiguous.

---

# Example

```text
Input: [10, 9, 2, 5, 3, 7, 101, 18]
Output: 4  (subsequence [2, 3, 7, 101])
```

---

# Brute Force

Try all 2^n subsequences, check if increasing.

```text
O(2^n)
```

---

# Optimized Approach 1 — DP O(n²)

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int n = nums.length;
        int[] dp = new int[n];
        Arrays.fill(dp, 1);
        int max = 1;

        for (int i = 1; i < n; i++) {
            for (int j = 0; j < i; j++) {
                if (nums[j] < nums[i]) {
                    dp[i] = Math.max(dp[i], dp[j] + 1);
                }
            }
            max = Math.max(max, dp[i]);
        }
        return max;
    }
}
```

---

# Optimized Approach 2 — Patience Sorting O(n log n)

Maintain `tails[i]` = smallest tail of increasing subsequence of length i+1.

```java
class Solution {
    public int lengthOfLIS(int[] nums) {
        int[] tails = new int[nums.length];
        int size = 0;

        for (int num : nums) {
            int lo = 0, hi = size;
            while (lo < hi) {
                int mid = lo + (hi - lo) / 2;
                if (tails[mid] < num) lo = mid + 1;
                else hi = mid;
            }
            tails[lo] = num;
            if (lo == size) size++;
        }
        return size;
    }
}
```

---

# Time Complexity

```text
O(n²) DP  |  O(n log n) binary search approach
```

---

# 2. Coin Change

## Problem Statement

Given coin denominations and amount, return minimum coins needed. Return `-1` if impossible.

---

---

# Example

```text
Input: coins = [1, 2, 5], amount = 11
Output: 3  (5 + 5 + 1)
```

---

# Optimized Approach — Bottom-Up DP

```java
class Solution {
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1);
        dp[0] = 0;

        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (coin <= i) {
                    dp[i] = Math.min(dp[i], dp[i - coin] + 1);
                }
            }
        }
        return dp[amount] > amount ? -1 : dp[amount];
    }
}
```

---

# Recurrence

```text
dp[i] = min(dp[i - coin] + 1) for each coin where coin <= i
dp[0] = 0
```

---

# JavaScript Solution

```javascript
function coinChange(coins, amount) {
  const dp = Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  for (let i = 1; i <= amount; i++) {
    for (const coin of coins) {
      if (coin <= i) dp[i] = Math.min(dp[i], dp[i - coin] + 1);
    }
  }
  return dp[amount] === Infinity ? -1 : dp[amount];
}
```

---

# Time Complexity

```text
O(amount * number of coins)
```

---

# 3. Word Break

## Problem Statement

Given string `s` and dictionary `wordDict`, determine if `s` can be segmented into space-separated words from dictionary.

---

# Example

```text
Input: s = "leetcode", wordDict = ["leet","code"]
Output: true

Input: s = "catsandog", wordDict = ["cats","dog","sand","and","cat"]
Output: false
```

---

# Optimized Approach — DP

```java
class Solution {
    public boolean wordBreak(String s, List<String> wordDict) {
        Set<String> dict = new HashSet<>(wordDict);
        boolean[] dp = new boolean[s.length() + 1];
        dp[0] = true;

        for (int i = 1; i <= s.length(); i++) {
            for (int j = 0; j < i; j++) {
                if (dp[j] && dict.contains(s.substring(j, i))) {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.length()];
    }
}
```

---

# Core Idea

```text
dp[i] = true if there exists j < i where dp[j] == true
        AND s[j..i) is in dictionary
```

---

# BFS Alternative

Treat as graph: each word match is an edge. BFS from index 0 to index n.

---

# Time Complexity

```text
O(n² * m) where n = string length, m = avg word length for substring check
```

With Trie optimization: O(n²)

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| LIS | DP / Patience sorting + binary search | O(n²) / O(n log n) |
| Coin Change | Unbounded knapsack DP | O(amount × coins) |
| Word Break | DP substring segmentation | O(n²) |

---

# Interview Questions

## Coin Change — greedy works?

Not always. Coins `[1, 3, 4]`, amount 6: greedy gives 4+1+1=3, optimal is 3+3=2.

## Word Break II follow-up?

Return all possible sentences — backtracking on top of DP table.

## LIS — return actual subsequence?

Track parent indices during DP, reconstruct path.

## Space optimize Coin Change?

Only need previous row — O(amount) space instead of 2D.

---

*End of Day 18 LeetCode*
