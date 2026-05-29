# Day 16 — LeetCode (Two Pointers)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 167 | [Two Sum II - Input Array Is Sorted](#1-two-sum-ii-167) | Medium | Opposite ends |
| 977 | [Squares of a Sorted Array](#2-squares-of-a-sorted-array-977) | Easy | Merge from ends |
| 18 | [4Sum](#3-4sum-18) | Medium | Sort + two loops + two pointers |

---

## Table of Contents

1. [Two Sum II (167)](#1-two-sum-ii-167)
2. [Squares of a Sorted Array (977)](#2-squares-of-a-sorted-array-977)
3. [4Sum (18)](#3-4sum-18)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Two Sum II (167)

### Problem (short)

Given a **1-indexed** sorted array `numbers` and `target`, return indices `[index1, index2]` such that `numbers[index1] + numbers[index2] === target`. Exactly one solution exists.

```
Input: numbers = [2,7,11,15], target = 9
Output: [1, 2]
```

### Hint 1 — Binary search or two pointers?

Sorted array → **two pointers** at both ends is O(n) vs O(n log n) per element with binary search.

### Hint 2 — Which pointer to move?

If `sum < target`, need larger sum → `left++`.  
If `sum > target`, need smaller sum → `right--`.

### Solution — JavaScript

```js
/**
 * @param {number[]} numbers
 * @param {number} target
 * @return {number[]}
 */
function twoSum(numbers, target) {
  let l = 0, r = numbers.length - 1;

  while (l < r) {
    const sum = numbers[l] + numbers[r];
    if (sum === target) return [l + 1, r + 1]; // 1-indexed
    if (sum < target) l++;
    else r--;
  }
  return [-1, -1];
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) |

### Follow-ups (interview)

| Question | Answer |
|----------|--------|
| Why not hash map? | Works O(n) but ignores sorted property; two pointers uses O(1) space |
| Duplicates allowed? | Problem guarantees unique solution pair |
| Return values not indices? | Same logic |

---

## 2. Squares of a Sorted Array (977)

### Problem (short)

Given sorted `nums` (may include negatives), return array of squares **sorted** in non-decreasing order.

```
Input: [-4,-1,0,3,10]
Output: [0,1,9,16,100]
```

### Hint 1 — Don't sort after squaring

Squaring then sorting is O(n log n). Can we do O(n)?

### Hint 2 — Largest squares at the ends

Negative numbers squared can be large. Compare `|nums[l]|` vs `|nums[r]|`, place larger square at write position moving inward.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @return {number[]}
 */
function sortedSquares(nums) {
  const n = nums.length;
  const res = new Array(n);
  let l = 0, r = n - 1, k = n - 1;

  while (l <= r) {
    const leftSq = nums[l] * nums[l];
    const rightSq = nums[r] * nums[r];
    if (leftSq > rightSq) {
      res[k--] = leftSq;
      l++;
    } else {
      res[k--] = rightSq;
      r--;
    }
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(n) | O(n) for output |

### Visual intuition

```
nums:  [-4, -1,  0,  3, 10]
         ↑              ↑
         l              r
Compare |−4|² vs |10|² → 100 goes to res[4]
```

---

## 3. 4Sum (18)

### Problem (short)

Find all **unique** quadruplets `[a, b, c, d]` such that `a + b + c + d === target`.

```
Input: nums = [1,0,-1,0,-2,2], target = 0
Output: [[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]]
```

### Hint 1 — Extend 3Sum

Sort array. Fix `i`, then `j`, then two-pointer on `l` and `r` for remaining sum.

### Hint 2 — Skip duplicates at every level

When `nums[i] === nums[i-1]` (and same for `j`, `l`, `r`), skip to avoid duplicate quadruplets.

### Algorithm

1. Sort `nums`.
2. For `i` from `0` to `n-4`:
   - Skip duplicate `i`.
   - For `j` from `i+1` to `n-3`:
     - Skip duplicate `j`.
     - `l = j+1`, `r = n-1`.
     - While `l < r`: if sum === target, push quad and skip dup `l`/`r`; adjust pointers.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number[][]}
 */
function fourSum(nums, target) {
  nums.sort((a, b) => a - b);
  const res = [];
  const n = nums.length;

  for (let i = 0; i < n - 3; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    for (let j = i + 1; j < n - 2; j++) {
      if (j > i + 1 && nums[j] === nums[j - 1]) continue;
      let l = j + 1, r = n - 1;

      while (l < r) {
        const sum = nums[i] + nums[j] + nums[l] + nums[r];
        if (sum === target) {
          res.push([nums[i], nums[j], nums[l], nums[r]]);
          while (l < r && nums[l] === nums[l + 1]) l++;
          while (l < r && nums[r] === nums[r - 1]) r--;
          l++;
          r--;
        } else if (sum < target) l++;
        else r--;
      }
    }
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(n³) | O(1) extra (excluding output / sort) |

### Optimization — early break

```js
if (nums[i] + nums[i+1] + nums[i+2] + nums[i+3] > target) break;
if (nums[i] + nums[n-3] + nums[n-2] + nums[n-1] < target) continue;
```

---

## 4. Pattern Cheat Sheet

| Problem | Pointers | Key move |
|---------|----------|----------|
| **167** | L/R on sorted | sum vs target → move one side |
| **977** | L/R, fill from back | Place larger square first |
| **18** | Fix i,j + L/R | Sort + skip duplicates at 4 levels |

### When you see…

| Signal | Think |
|--------|-------|
| Sorted + pair sum | Two pointers opposite ends |
| Sorted with negatives → squares | Merge from ends like merge sort |
| K-sum (K fixed) | Sort + (K-2) nested loops + two pointers |

### Complexity ladder

| K in K-Sum | Typical time |
|------------|--------------|
| 2 | O(n) two pointers |
| 3 | O(n²) |
| 4 | O(n³) |
| general K | O(n^(K-1)) two-pointer last step |

---

*End of Day 16 LeetCode*
