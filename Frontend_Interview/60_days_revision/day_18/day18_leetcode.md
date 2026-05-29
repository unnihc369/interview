# Day 18 — LeetCode (Two Pointers)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 75 | [Sort Colors](#1-sort-colors-75) | Medium | Dutch national flag |
| 283 | [Move Zeroes](#2-move-zeroes-283) | Easy | Two pointers write |
| 845 | [Longest Mountain in Array](#3-longest-mountain-in-array-845) | Medium | Expand from peak |

---

## Table of Contents

1. [Sort Colors (75)](#1-sort-colors-75)
2. [Move Zeroes (283)](#2-move-zeroes-283)
3. [Longest Mountain in Array (845)](#3-longest-mountain-in-array-845)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Sort Colors (75)

### Problem (short)

Sort array of `0`, `1`, `2` **in-place** (one-pass preferred).

```
Input: nums = [2,0,2,1,1,0]
Output: [0,0,1,1,2,2]
```

### Hint 1 — Counting sort works

Three passes count 0/1/2 — O(n) but two passes. Can we one-pass?

### Hint 2 — Dutch National Flag (Dijkstra)

Three pointers: `lo` (next 0 slot), `mid` (current), `hi` (next 2 slot from end).

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @return {void} Do not return anything, modify nums in-place.
 */
function sortColors(nums) {
  let lo = 0, mid = 0, hi = nums.length - 1;

  while (mid <= hi) {
    if (nums[mid] === 0) {
      [nums[lo], nums[mid]] = [nums[mid], nums[lo]];
      lo++;
      mid++;
    } else if (nums[mid] === 1) {
      mid++;
    } else {
      [nums[mid], nums[hi]] = [nums[hi], nums[mid]];
      hi--;
    }
  }
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) |

### Why `mid++` after swap with `lo` but not after `hi`?

Swapped-from-`hi` value is unclassified — re-examine at `mid`. From `lo` we know it's 0.

---

## 2. Move Zeroes (283)

### Problem (short)

Move all `0`s to end while maintaining relative order of non-zero elements **in-place**.

```
Input: nums = [0,1,0,3,12]
Output: [1,3,12,0,0]
```

### Hint 1 — Write pointer

`write` = next position for non-zero. Scan `read`; copy non-zero to `write++`.

### Hint 2 — Fill rest with zeros

After scan, fill `write..n-1` with 0.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @return {void}
 */
function moveZeroes(nums) {
  let write = 0;
  for (let read = 0; read < nums.length; read++) {
    if (nums[read] !== 0) {
      nums[write++] = nums[read];
    }
  }
  while (write < nums.length) nums[write++] = 0;
}
```

### Swap variant (single pass, preserves order)

```js
function moveZeroesSwap(nums) {
  let write = 0;
  for (let read = 0; read < nums.length; read++) {
    if (nums[read] !== 0) {
      [nums[write], nums[read]] = [nums[read], nums[write]];
      write++;
    }
  }
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) |

---

## 3. Longest Mountain in Array (845)

### Problem (short)

Longest subarray that is **mountain**: strictly increase then strictly decrease, length ≥ 3.

```
Input: arr = [2,1,4,7,3,2,5]
Output: 5  → [1,4,7,3,2]
```

### Hint 1 — Brute O(n²)

Every peak try expand left/right — acceptable but O(n²).

### Hint 2 — Two pointers from each peak

Find peak where `arr[i-1] < arr[i] > arr[i+1]`, expand `left` and `right` while strictly monotonic.

### Solution — JavaScript

```js
/**
 * @param {number[]} arr
 * @return {number}
 */
function longestMountain(arr) {
  let best = 0;
  const n = arr.length;

  for (let peak = 1; peak < n - 1; peak++) {
    if (arr[peak - 1] < arr[peak] && arr[peak] > arr[peak + 1]) {
      let left = peak - 1;
      while (left > 0 && arr[left - 1] < arr[left]) left--;
      let right = peak + 1;
      while (right < n - 1 && arr[right] > arr[right + 1]) right++;
      best = Math.max(best, right - left + 1);
    }
  }
  return best;
}
```

### Optimized two-pass (optional)

Precompute `up[i]` = length of increasing ending at i, `down[i]` = decreasing starting at i — O(n).

| Time | Space |
|------|-------|
| O(n) per peak worst O(n²); optimized O(n) | O(1) or O(n) |

---

## 4. Pattern Cheat Sheet

| Problem | Pointers | Key move |
|---------|----------|----------|
| **75** | lo / mid / hi | 0→lo, 1→mid++, 2→hi |
| **283** | read / write | Write non-zeros; pad zeros |
| **845** | Expand from peak | Strict inc then dec |

### When you see…

| Signal | Think |
|--------|-------|
| 3-way partition | Dutch flag |
| In-place filter / compact | Read-write two pointers |
| Peak / valley subarray | Expand from local peak |

---

*End of Day 18 LeetCode*
