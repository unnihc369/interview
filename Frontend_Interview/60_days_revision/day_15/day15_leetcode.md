# Day 15 — LeetCode (Two Pointers)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 11 | [Container With Most Water](#1-container-with-most-water-11) | Medium | Opposite ends |
| 15 | [3Sum](#2-3sum-15) | Medium | Sort + two pointers |
| 42 | [Trapping Rain Water](#3-trapping-rain-water-42) | Hard | Two pointers / stack |

---

## Table of Contents

1. [Container With Most Water (11)](#1-container-with-most-water-11)
2. [3Sum (15)](#2-3sum-15)
3. [Trapping Rain Water (42)](#3-trapping-rain-water-42)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Container With Most Water (11)

### Problem (short)

Given `height[i]` as vertical lines, find two lines that form a container holding the **maximum** water.

```
Input: height = [1,8,6,2,5,4,8,3,7]
Output: 49  (lines at index 1 and 8)
```

### Hint 1 — Brute force is O(n²)

Interview: state you can do better with **two pointers** at both ends.

### Hint 2 — Move the shorter side

Area = `min(h[l], h[r]) * (r - l)`.  
If you move the **taller** side, width decreases and height cannot exceed the shorter side → area can only shrink.

### Solution — JavaScript

```js
/**
 * @param {number[]} height
 * @return {number}
 */
function maxArea(height) {
  let l = 0, r = height.length - 1, best = 0;
  while (l < r) {
    const h = Math.min(height[l], height[r]);
    best = Math.max(best, h * (r - l));
    if (height[l] < height[r]) l++;
    else r--;
  }
  return best;
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) |

---

## 2. 3Sum (15)

### Problem (short)

Return all unique triplets `[a, b, c]` such that `a + b + c = 0`.

### Hint 1 — Sort first

Sorting enables two-pointer on the remainder after fixing one element.

### Hint 2 — Skip duplicates

When `nums[i] === nums[i-1]` (and `i > 0`), skip to avoid duplicate triplets.

### Algorithm

1. Sort `nums`.
2. For each `i`, set `l = i+1`, `r = n-1`.
3. While `l < r`: if sum === 0, record and skip duplicate `l`/`r`; if sum < 0, `l++`; else `r--`.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
function threeSum(nums) {
  nums.sort((a, b) => a - b);
  const res = [];

  for (let i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    let l = i + 1, r = nums.length - 1;

    while (l < r) {
      const sum = nums[i] + nums[l] + nums[r];
      if (sum === 0) {
        res.push([nums[i], nums[l], nums[r]]);
        while (l < r && nums[l] === nums[l + 1]) l++;
        while (l < r && nums[r] === nums[r - 1]) r--;
        l++;
        r--;
      } else if (sum < 0) l++;
      else r--;
    }
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(n²) | O(1) extra (excluding output) |

---

## 3. Trapping Rain Water (42)

### Problem (short)

Elevation map `height[]` — how much water can be trapped after rain?

```
Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

### Approach A — Two pointers (optimal space)

Track `leftMax`, `rightMax`. Move pointer at the **smaller max** side and add water above current bar.

```js
function trap(height) {
  let l = 0, r = height.length - 1;
  let leftMax = 0, rightMax = 0, water = 0;

  while (l < r) {
    if (height[l] < height[r]) {
      leftMax = Math.max(leftMax, height[l]);
      water += leftMax - height[l];
      l++;
    } else {
      rightMax = Math.max(rightMax, height[r]);
      water += rightMax - height[r];
      r--;
    }
  }
  return water;
}
```

### Approach B — Stack (good to mention)

Monotonic stack for bar-by-bar pooling — O(n) time, O(n) space.

| Time | Space (two ptr) |
|------|-----------------|
| O(n) | O(1) |

---

## 4. Pattern Cheat Sheet

| Problem | Pointers | Key move |
|---------|----------|----------|
| **11** | L/R ends | Move shorter height |
| **15** | Fix i, L/R on rest | Sort + skip dupes |
| **42** | L/R + max heights | Add water at smaller-max side |

### When you see…

| Signal | Think |
|--------|-------|
| Max pair / area between ends | Opposite two pointers |
| Triplets with sum K | Sort + for loop + two pointers |
| Trapped water / elevation | Two pointers or stack |

---

*End of Day 15 LeetCode*
