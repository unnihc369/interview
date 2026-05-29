# Day 19 — DP: House Robber & Decode Ways

**Topics:** House Robber I · House Robber II · Decode Ways

---

# 1. House Robber I

## Problem Statement

Given array `nums` where `nums[i]` is money in house `i`, rob houses such that **no two adjacent houses** are robbed. Return maximum money.

---

# Example

```text
Input: [1, 2, 3, 1]
Output: 4  (rob house 0 and 2: 1 + 3)

Input: [2, 7, 9, 3, 1]
Output: 12 (2 + 9 + 1)
```

---

# Brute Force

Try all subsets, check no adjacent. Exponential.

---

# Optimized Approach — DP

## Recurrence

At house `i`, two choices:
- Rob: `nums[i] + dp[i-2]`
- Skip: `dp[i-1]`

```text
dp[i] = max(dp[i-1], dp[i-2] + nums[i])
```

---

# Java Solution

```java
class Solution {
    public int rob(int[] nums) {
        if (nums.length == 1) return nums[0];

        int prev2 = nums[0];
        int prev1 = Math.max(nums[0], nums[1]);

        for (int i = 2; i < nums.length; i++) {
            int curr = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1;
            prev1 = curr;
        }
        return prev1;
    }
}
```

---

# JavaScript Solution

```javascript
function rob(nums) {
  if (nums.length === 1) return nums[0];
  let prev2 = nums[0], prev1 = Math.max(nums[0], nums[1]);
  for (let i = 2; i < nums.length; i++) {
    const curr = Math.max(prev1, prev2 + nums[i]);
    prev2 = prev1;
    prev1 = curr;
  }
  return prev1;
}
```

---

# Time Complexity

```text
O(n) time, O(1) space
```

---

# 2. House Robber II

## Problem Statement

Houses arranged in a **circle** — first and last houses are adjacent. Find maximum robbery amount.

---

# Example

```text
Input: [2, 3, 2]
Output: 3  (cannot rob 2 and 2 — they're adjacent in circle)

Input: [1, 2, 3, 1]
Output: 4  (rob 1 and 3)
```

---

# Optimized Approach

Break circle into two linear problems:

1. Rob houses `0` to `n-2` (exclude last)
2. Rob houses `1` to `n-1` (exclude first)

Return max of both.

---

# Java Solution

```java
class Solution {
    public int rob(int[] nums) {
        if (nums.length == 1) return nums[0];
        return Math.max(robLinear(nums, 0, nums.length - 2),
                        robLinear(nums, 1, nums.length - 1));
    }

    private int robLinear(int[] nums, int start, int end) {
        int prev2 = nums[start];
        int prev1 = Math.max(nums[start], nums[start + 1]);

        for (int i = start + 2; i <= end; i++) {
            int curr = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1;
            prev1 = curr;
        }
        return prev1;
    }
}
```

---

# Time Complexity

```text
O(n) time, O(1) space
```

---

# 3. Decode Ways

## Problem Statement

Given string containing digits, count number of ways to decode it.

Mapping: `1→A, 2→B, ..., 26→Z`

---

# Example

```text
Input: "12"
Output: 2  ("AB" or "L")

Input: "226"
Output: 3  ("BZ", "VF", "BBF")

Input: "06"
Output: 0  (invalid — '06' is not valid encoding)
```

---

# Optimized Approach — DP

```text
dp[i] = ways to decode s[0..i)

dp[i] = dp[i-1]           if s[i-1] is valid single digit (1-9)
      + dp[i-2]           if s[i-2..i-1] is valid two-digit (10-26)
```

---

# Java Solution

```java
class Solution {
    public int numDecodings(String s) {
        if (s.charAt(0) == '0') return 0;

        int n = s.length();
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;

        for (int i = 2; i <= n; i++) {
            int oneDigit = Integer.parseInt(s.substring(i - 1, i));
            int twoDigits = Integer.parseInt(s.substring(i - 2, i));

            if (oneDigit >= 1 && oneDigit <= 9) {
                dp[i] += dp[i - 1];
            }
            if (twoDigits >= 10 && twoDigits <= 26) {
                dp[i] += dp[i - 2];
            }
        }
        return dp[n];
    }
}
```

---

# Space-Optimized

```java
class Solution {
    public int numDecodings(String s) {
        if (s.charAt(0) == '0') return 0;
        int prev2 = 1, prev1 = 1;

        for (int i = 1; i < s.length(); i++) {
            int curr = 0;
            if (s.charAt(i) != '0') curr += prev1;
            int two = Integer.parseInt(s.substring(i - 1, i + 1));
            if (two >= 10 && two <= 26) curr += prev2;
            prev2 = prev1;
            prev1 = curr;
        }
        return prev1;
    }
}
```

---

# JavaScript Solution

```javascript
function numDecodings(s) {
  if (s[0] === '0') return 0;
  let prev2 = 1, prev1 = 1;
  for (let i = 1; i < s.length; i++) {
    let curr = 0;
    if (s[i] !== '0') curr += prev1;
    const two = parseInt(s.substring(i - 1, i + 1));
    if (two >= 10 && two <= 26) curr += prev2;
    prev2 = prev1;
    prev1 = curr;
  }
  return prev1;
}
```

---

# Time Complexity

```text
O(n) time, O(1) space (optimized)
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| House Robber I | Linear DP, take/skip adjacent | O(n) |
| House Robber II | Two linear passes (break circle) | O(n) |
| Decode Ways | DP string partitioning (1 or 2 chars) | O(n) |

---

# Interview Questions

## House Robber — relation to Fibonacci?

When all house values are 1, answer equals Fibonacci(n). Same recurrence structure.

## Decode Ways — handle leading zeros?

If `s[i] == '0'`, only valid as part of "10" or "20". Standalone '0' contributes 0 ways.

## House Robber II — why two passes work?

In circular arrangement, robber either takes first house (exclude last) or takes last (exclude first). These are mutually exclusive optimal subcases.

## Decode Ways vs Climbing Stairs?

Similar DP structure. Decode Ways has validity constraints (no '0' alone, max 26 for two digits).

---

*End of Day 19 LeetCode*
