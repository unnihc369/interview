# Day 12 — LeetCode (Sliding Window — Consecutive & Grumpy)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 487 | [Max Consecutive Ones II](#1-max-consecutive-ones-ii-487) | Medium | Variable — at most 1 flip |
| 1297 | [Maximum Number of Occurrences of a Substring](#2-maximum-number-of-occurrences-of-a-substring-1297) | Medium | Fixed window + freq cap |
| 1052 | [Grumpy Bookstore Owner](#3-grumpy-bookstore-owner-1052) | Medium | Fixed window on bonus minutes |

---

## Table of Contents

1. [Max Consecutive Ones II (487)](#1-max-consecutive-ones-ii-487)
2. [Maximum Occurrences Substring (1297)](#2-maximum-number-of-occurrences-of-a-substring-1297)
3. [Grumpy Bookstore Owner (1052)](#3-grumpy-bookstore-owner-1052)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Max Consecutive Ones II (487)

### Problem (short)

Binary array — flip at most **one** 0 to 1. Return max consecutive 1s.

```
[1,0,1,1,0,1] → 4
[1,0,1,0,1] → 4
```

### Solution — longestOnes(nums, 1)

```js
/**
 * @param {number[]} nums
 * @return {number}
 */
function findMaxConsecutiveOnes(nums) {
  let left = 0;
  let zeros = 0;
  let maxLen = 0;

  for (let right = 0; right < nums.length; right++) {
    if (nums[right] === 0) zeros++;
    while (zeros > 1) {
      if (nums[left] === 0) zeros--;
      left++;
    }
    maxLen = Math.max(maxLen, right - left + 1);
  }
  return maxLen;
}
```

---

## 2. Maximum Occurrences Substring (1297)

### Problem (short)

String `s`, `maxLetters` distinct chars, `minSize` and `maxSize` window bounds. Return **max frequency** of any valid substring occurrence.

**Key insight:** Optimal substring length is **minSize** (provable — interview mention).

```js
/**
 * @param {string} s
 * @param {number} maxLetters
 * @param {number} minSize
 * @param {number} maxSize
 * @return {number}
 */
function maxFreq(s, maxLetters, minSize, maxSize) {
  const k = minSize;
  const count = new Map();
  const freq = new Map();
  let left = 0;
  let maxOcc = 0;

  for (let right = 0; right < s.length; right++) {
    const c = s[right];
    count.set(c, (count.get(c) ?? 0) + 1);

    if (right - left + 1 > k) {
      const l = s[left++];
      count.set(l, count.get(l) - 1);
      if (count.get(l) === 0) count.delete(l);
    }

    if (right - left + 1 === k && count.size <= maxLetters) {
      const sub = s.slice(left, right + 1);
      freq.set(sub, (freq.get(sub) ?? 0) + 1);
      maxOcc = Math.max(maxOcc, freq.get(sub));
    }
  }
  return maxOcc;
}
```

---

## 3. Grumpy Bookstore Owner (1052)

### Problem (short)

`customers[i]` visitors, `grumpy[i]` 1 if owner grumpy (unsatisfied). You have **minutes** to use technique → owner not grumpy. Max satisfied customers?

```
customers = [1,0,1,2,1,1,7,5], grumpy = [0,1,0,1,0,1,0,0], minutes = 3 → 16
```

### Fixed window on unsatisfied only

```js
/**
 * @param {number[]} customers
 * @param {number[]} grumpy
 * @param {number} minutes
 * @return {number}
 */
function maxSatisfied(customers, grumpy, minutes) {
  let base = 0;
  for (let i = 0; i < customers.length; i++) {
    if (grumpy[i] === 0) base += customers[i];
  }

  let bonus = 0;
  for (let i = 0; i < minutes; i++) {
    if (grumpy[i]) bonus += customers[i];
  }

  let maxBonus = bonus;
  for (let i = minutes; i < customers.length; i++) {
    if (grumpy[i]) bonus += customers[i];
    if (grumpy[i - minutes]) bonus -= customers[i - minutes];
    maxBonus = Math.max(maxBonus, bonus);
  }
  return base + maxBonus;
}
```

---

## 4. Pattern Cheat Sheet

| Problem | Window | Trick |
|---------|--------|-------|
| **487** | Variable | ≤ 1 zero (1004 with k=1) |
| **1297** | Fixed minSize | Only check minSize windows |
| **1052** | Fixed minutes | Base + sliding bonus |

---

*End of Day 12 LeetCode*
