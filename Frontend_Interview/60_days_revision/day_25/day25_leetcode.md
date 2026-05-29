# Day 25 — LeetCode (Heaps)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 378 | [Kth Smallest Element in a Sorted Matrix](#1-kth-smallest-element-in-a-sorted-matrix-378) | Medium | Min-heap / binary search |
| 264 | [Ugly Number II](#2-ugly-number-ii-264) | Medium | Min-heap triple pointers |
| 373 | [Find K Pairs with Smallest Sums](#3-find-k-pairs-with-smallest-sums-373) | Medium | Min-heap pairs |

---

## Table of Contents

1. [Kth Smallest in Sorted Matrix (378)](#1-kth-smallest-element-in-a-sorted-matrix-378)
2. [Ugly Number II (264)](#2-ugly-number-ii-264)
3. [Find K Pairs with Smallest Sums (373)](#3-find-k-pairs-with-smallest-sums-373)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Kth Smallest Element in a Sorted Matrix (378)

### Problem (short)

`n x n` matrix sorted row/col ascending. Kth smallest element.

```
Input: matrix = [[1,5,9],[10,11,13],[12,13,15]], k = 8
Output: 13
```

### Hint 1 — Min-heap of first column cells

Push `[matrix[i][0], i, 0]` for each row; pop k times pushing next in row.

### Hint 2 — Binary search on value range

Count elements <= mid — O(n log(max-min)).

### Solution — JavaScript (heap style)

```js
/**
 * @param {number[][]} matrix
 * @param {number} k
 * @return {number}
 */
function kthSmallest(matrix, k) {
  const n = matrix.length;
  const heap = [];
  for (let i = 0; i < n; i++) heap.push({ val: matrix[i][0], r: i, c: 0 });
  heap.sort((a, b) => a.val - b.val);

  for (let t = 0; t < k - 1; t++) {
    heap.sort((a, b) => a.val - b.val);
    const { r, c } = heap.shift();
    if (c + 1 < n) heap.push({ val: matrix[r][c + 1], r, c: c + 1 });
  }
  heap.sort((a, b) => a.val - b.val);
  return heap[0].val;
}
```

| Time | Space |
|------|-------|
| O(k log n) heap | O(n) |

---

## 2. Ugly Number II (264)

### Problem (short)

Ugly number has prime factors only 2, 3, 5. Return nth ugly number.

```
Input: n = 10
Output: 12
Sequence: 1,2,3,4,5,6,8,9,10,12,...
```

### Hint 1 — Generate in order with min-heap

Start heap `{1}`; pop min, push `x*2`, `x*3`, `x*5` (dedupe).

### Hint 2 — Three pointers (better)

```js
/**
 * @param {number} n
 * @return {number}
 */
function nthUglyNumber(n) {
  const ugly = [1];
  let i2 = 0, i3 = 0, i5 = 0;

  while (ugly.length < n) {
    const next = Math.min(ugly[i2] * 2, ugly[i3] * 3, ugly[i5] * 5);
    ugly.push(next);
    if (next === ugly[i2] * 2) i2++;
    if (next === ugly[i3] * 3) i3++;
    if (next === ugly[i5] * 5) i5++;
  }
  return ugly[n - 1];
}
```

| Time | Space |
|------|-------|
| O(n) | O(n) |

---

## 3. Find K Pairs with Smallest Sums (373)

### Problem (short)

Two sorted arrays `nums1`, `nums2`. K pairs `(u,v)` with smallest `u+v`.

### Hint 1 — Heap of (sum, i, j)

Start `(nums1[0]+nums2[0], 0, 0)`; push `(i+1,j)` and `(i,j+1)` with visited set.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @param {number} k
 * @return {number[][]}
 */
function kSmallestPairs(nums1, nums2, k) {
  if (!nums1.length || !nums2.length) return [];
  const heap = [{ sum: nums1[0] + nums2[0], i: 0, j: 0 }];
  const visited = new Set(["0,0"]);
  const res = [];

  while (heap.length && res.length < k) {
    heap.sort((a, b) => a.sum - b.sum);
    const { i, j } = heap.shift();
    res.push([nums1[i], nums2[j]]);

    if (i + 1 < nums1.length && !visited.has(`${i + 1},${j}`)) {
      visited.add(`${i + 1},${j}`);
      heap.push({ sum: nums1[i + 1] + nums2[j], i: i + 1, j });
    }
    if (j + 1 < nums2.length && !visited.has(`${i},${j + 1}`)) {
      visited.add(`${i},${j + 1}`);
      heap.push({ sum: nums1[i] + nums2[j + 1], i, j: j + 1 });
    }
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(k log k) | O(k) |

---

## 4. Pattern Cheat Sheet

| Problem | Heap use | Key |
|---------|----------|-----|
| **378** | Row heads | Pop k, push next col |
| **264** | Generate sorted | 3 pointers or heap dedupe |
| **373** | Pair sums | Visited (i,j) states |

### Binary search alternative (378)

```js
function kthSmallestBinarySearch(matrix, k) {
  const n = matrix.length;
  let lo = matrix[0][0], hi = matrix[n - 1][n - 1];
  while (lo < hi) {
    const mid = Math.floor((lo + hi) / 2);
    let count = 0, j = n - 1;
    for (let i = 0; i < n; i++) {
      while (j >= 0 && matrix[i][j] > mid) j--;
      count += j + 1;
    }
    if (count < k) lo = mid + 1;
    else hi = mid;
  }
  return lo;
}
```

### Follow-ups

| Question | Answer |
|----------|--------|
| 264 heap dedupe | Use Set when pushing 2x, 3x, 5x |
| 373 empty nums1 | Return [] early |
| 378 row-only sorted? | Different problem — need different approach |

---

*End of Day 25 LeetCode*
