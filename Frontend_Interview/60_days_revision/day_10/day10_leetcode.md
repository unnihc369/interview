# Day 10 — LeetCode (Sliding Window — Subarray Sum)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 209 | [Minimum Size Subarray Sum](#1-minimum-size-subarray-sum-209) | Medium | Variable window — sum ≥ target |
| 1004 | [Max Consecutive Ones III](#2-max-consecutive-ones-iii-1004) | Medium | Variable — at most K flips |
| 992 | [Subarrays with K Different Integers](#3-subarrays-with-k-different-integers-992) | Hard | At most K trick |

---

## Table of Contents

1. [Minimum Size Subarray Sum (209)](#1-minimum-size-subarray-sum-209)
2. [Max Consecutive Ones III (1004)](#2-max-consecutive-ones-iii-1004)
3. [Subarrays with K Different Integers (992)](#3-subarrays-with-k-different-integers-992)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Minimum Size Subarray Sum (209)

### Problem (short)

Array of **positive** integers `nums`, integer `target`. Return **minimal length** of subarray with sum ≥ `target`, or 0 if none.

```
target = 7, nums = [2,3,1,2,4,3] → 2  ([4,3])
target = 4, nums = [1,4,4] → 1
target = 11, nums = [1,1,1,1,1,1,1,1] → 0
```

### Hint — Expand until sum ≥ target, then shrink

```js
/**
 * @param {number} target
 * @param {number[]} nums
 * @return {number}
 */
function minSubArrayLen(target, nums) {
  let left = 0;
  let sum = 0;
  let minLen = Infinity;

  for (let right = 0; right < nums.length; right++) {
    sum += nums[right];
    while (sum >= target) {
      minLen = Math.min(minLen, right - left + 1);
      sum -= nums[left++];
    }
  }
  return minLen === Infinity ? 0 : minLen;
}
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(1) |

---

## 2. Max Consecutive Ones III (1004)

### Problem (short)

Binary array `nums`, integer `k`. You may flip at most **k** zeros to ones. Return max length of contiguous subarray of 1s.

```
nums = [1,1,1,0,0,0,1,1,1,1,0], k = 2 → 6
nums = [0,0,1,1,0,0,1,1,1,0,1,1,1,1,0,0,0,1,1,1,1,1], k = 3 → 10
```

### Hint — Window with zero count

Expand; while zeros > k, shrink left.

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
function longestOnes(nums, k) {
  let left = 0;
  let zeros = 0;
  let maxLen = 0;

  for (let right = 0; right < nums.length; right++) {
    if (nums[right] === 0) zeros++;
    while (zeros > k) {
      if (nums[left] === 0) zeros--;
      left++;
    }
    maxLen = Math.max(maxLen, right - left + 1);
  }
  return maxLen;
}
```

---

## 3. Subarrays with K Different Integers (992)

### Problem (short)

Return number of **subarrays** with exactly **K** different integers.

```
nums = [1,2,1,2,3], k = 2 → 7
nums = [1,2,1,3,4], k = 3 → 3
```

### Hint — exactly K = atMost(K) - atMost(K-1)

```js
function subarraysWithKDistinct(nums, k) {
  return atMost(nums, k) - atMost(nums, k - 1);
}

function atMost(nums, k) {
  const count = new Map();
  let left = 0;
  let total = 0;

  for (let right = 0; right < nums.length; right++) {
    const x = nums[right];
    count.set(x, (count.get(x) ?? 0) + 1);

    while (count.size > k) {
      const l = nums[left++];
      count.set(l, count.get(l) - 1);
      if (count.get(l) === 0) count.delete(l);
    }
    total += right - left + 1; // all subarrays ending at right
  }
  return total;
}
```

### Why add `right - left + 1`?

Each valid window `[left, right]` contributes all subarrays ending at `right` with start in `[left, right]`.

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(k) |

---

## 4. Pattern Cheat Sheet

| Problem | Condition | Shrink when |
|---------|-----------|-------------|
| **209** | sum ≥ target | sum ≥ target (minimize) |
| **1004** | at most k zeros | zeros > k |
| **992** | exactly k distinct | atMost(k) - atMost(k-1) |

---

*End of Day 10 LeetCode*
