# Day 20 — LeetCode (Merge / Two Pointers)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 88 | [Merge Sorted Array](#1-merge-sorted-array-88) | Easy | Merge from end |
| 986 | [Interval List Intersections](#2-interval-list-intersections-986) | Medium | Two pointer intervals |
| 259 | [3Sum Smaller](#3-3sum-smaller-259) | Medium | Sort + two pointers count |

---

## Table of Contents

1. [Merge Sorted Array (88)](#1-merge-sorted-array-88)
2. [Interval List Intersections (986)](#2-interval-list-intersections-986)
3. [3Sum Smaller (259)](#3-3sum-smaller-259)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Merge Sorted Array (88)

### Problem (short)

Merge `nums2` into `nums1` of size `m + n` where first `m` elements of `nums1` are valid. **In-place.**

```
Input: nums1 = [1,2,3,0,0,0], m = 3, nums2 = [2,5,6], n = 3
Output: [1,2,2,3,5,6]
```

### Hint 1 — Merge from front needs shifting

Use **write pointer from end** — classic two-pointer backward merge.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums1
 * @param {number} m
 * @param {number[]} nums2
 * @param {number} n
 * @return {void}
 */
function merge(nums1, m, nums2, n) {
  let p1 = m - 1, p2 = n - 1, write = m + n - 1;
  while (p2 >= 0) {
    if (p1 >= 0 && nums1[p1] > nums2[p2]) nums1[write--] = nums1[p1--];
    else nums1[write--] = nums2[p2--];
  }
}
```

| Time | Space |
|------|-------|
| O(m + n) | O(1) |

---

## 2. Interval List Intersections (986)

### Problem (short)

Two lists of **disjoint sorted** intervals. Return intersection intervals.

```
Input: A = [[0,2],[5,10]], B = [[1,5],[8,12]]
Output: [[1,2],[5,5],[8,10]]
```

### Hint 1 — Two pointers on interval lists

Compare `A[i]` and `B[j]`. Intersection is `[max(starts), min(ends)]` if valid.

### Hint 2 — Advance shorter end

Whoever ends first cannot intersect current pair again — increment that pointer.

### Solution — JavaScript

```js
/**
 * @param {number[][]} firstList
 * @param {number[][]} secondList
 * @return {number[][]}
 */
function intervalIntersection(firstList, secondList) {
  const res = [];
  let i = 0, j = 0;

  while (i < firstList.length && j < secondList.length) {
    const [a1, a2] = firstList[i];
    const [b1, b2] = secondList[j];
    const start = Math.max(a1, b1);
    const end = Math.min(a2, b2);
    if (start <= end) res.push([start, end]);
    if (a2 < b2) i++;
    else j++;
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(n + m) | O(1) excluding output |

---

## 3. 3Sum Smaller (259)

### Problem (short)

Given sorted `nums` and `target`, count triplets `i < j < k` with sum **< target**.

```
Input: nums = [-2,0,1,3], target = 2
Output: 2  → (-2,0,1), (-2,0,3) wait check: (-2,0,1)= -1 < 2, (-2,1,3)=2 not < 2
Actually: (-2,0,1) only? Problem examples vary — algorithm counts all i,j,k with sum < target
```

### Hint 1 — Fix i, two-pointer j,k

For fixed `i`, if `nums[i]+nums[j]+nums[k] < target`, all k between j and current k work.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @param {number} target
 * @return {number}
 */
function threeSumSmaller(nums, target) {
  nums.sort((a, b) => a - b);
  let count = 0;
  const n = nums.length;

  for (let i = 0; i < n - 2; i++) {
    let l = i + 1, r = n - 1;
    while (l < r) {
      if (nums[i] + nums[l] + nums[r] < target) {
        count += r - l;
        l++;
      } else {
        r--;
      }
    }
  }
  return count;
}
```

| Time | Space |
|------|-------|
| O(n²) | O(1) |

---

## 4. Pattern Cheat Sheet

| Problem | Pointers | Key move |
|---------|----------|----------|
| **88** | p1, p2 from end | Larger goes to write |
| **986** | i, j on intervals | Intersect; advance earlier end |
| **259** | fix i, l/r | sum < target → count r-l, l++ |

### Follow-ups

| Question | Answer |
|----------|--------|
| 88 why merge from end? | Empty space at tail avoids overwrite |
| 986 disjoint assumption | If overlapping intervals in one list, merge first |
| 259 vs 15 | Count `< target` vs exact `=== 0` |

### Test cases

```js
// 88
const m = [1,2,3,0,0,0]; merge(m, 3, [2,5,6], 3); // [1,2,2,3,5,6]

// 986
intervalIntersection([[0,2],[5,10],[13,23]], [[1,5],[8,12],[15,24]]);
// [[1,2],[5,5],[8,10],[15,23]]

// 259
threeSumSmaller([-2,0,1,3], 2); // 2
```

### Complexity summary

| Problem | Time | Space |
|---------|------|-------|
| 88 | O(m+n) | O(1) |
| 986 | O(n+m) | O(1) |
| 259 | O(n²) | O(1) |

---

*End of Day 20 LeetCode*
