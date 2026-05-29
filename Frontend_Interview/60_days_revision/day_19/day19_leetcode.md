# Day 19 — LeetCode (Set / Two Pointers)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 349 | [Intersection of Two Arrays](#1-intersection-of-two-arrays-349) | Easy | Set intersection |
| 350 | [Intersection of Two Arrays II](#2-intersection-of-two-arrays-ii-350) | Easy | Map frequency |
| 457 | [Circular Array Loop](#3-circular-array-loop-457) | Medium | Set + simulation |

---

## Table of Contents

1. [Intersection of Two Arrays (349)](#1-intersection-of-two-arrays-349)
2. [Intersection of Two Arrays II (350)](#2-intersection-of-two-arrays-ii-350)
3. [Circular Array Loop (457)](#3-circular-array-loop-457)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Intersection of Two Arrays (349)

### Problem (short)

Return **unique** intersection of two integer arrays.

```
Input: nums1 = [1,2,2,1], nums2 = [2,2]
Output: [2]
```

### Hint 1 — Set for uniqueness

Put smaller array in Set; scan other for matches.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number[]}
 */
function intersection(nums1, nums2) {
  const set1 = new Set(nums1);
  const res = new Set();
  for (const n of nums2) {
    if (set1.has(n)) res.add(n);
  }
  return [...res];
}
```

| Time | Space |
|------|-------|
| O(n + m) | O(n) |

---

## 2. Intersection of Two Arrays II (350)

### Problem (short)

Return intersection with **multiplicity** — each element appears min(count1, count2) times.

```
Input: nums1 = [1,2,2,1], nums2 = [2,2]
Output: [2,2]
```

### Hint 1 — Frequency map

Count smaller array in Map; decrement when match found.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums1
 * @param {number[]} nums2
 * @return {number[]}
 */
function intersect(nums1, nums2) {
  if (nums1.length > nums2.length) return intersect(nums2, nums1);
  const freq = new Map();
  for (const n of nums1) freq.set(n, (freq.get(n) ?? 0) + 1);
  const res = [];
  for (const n of nums2) {
    const c = freq.get(n) ?? 0;
    if (c > 0) {
      res.push(n);
      freq.set(n, c - 1);
    }
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(n + m) | O(min(n,m)) |

### Sort + two pointers alternative

Sort both; advance pointers matching equal elements — O(n log n) time, O(1) extra if in-place.

---

## 3. Circular Array Loop (457)

### Problem (short)

Array of non-zero integers — each value is jump length (forward if positive, backward if negative). Circular index `(i + jump) % n`. Determine if there exists a **cycle** (same direction, length > 1).

```
Input: nums = [2,-1,1,2,2]
Output: true
```

### Hint 1 — Try each start + visited Set

For each index, simulate until loop, same direction break, or revisit.

### Hint 2 — Cycle length 1 invalid

If `next === i`, not a valid cycle.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @return {boolean}
 */
function circularArrayLoop(nums) {
  const n = nums.length;

  for (let start = 0; start < n; start++) {
    if (nums[start] === 0) continue;
    let slow = start, fast = start;
    const forward = nums[start] > 0;

    while (true) {
      slow = nextIdx(nums, slow);
      fast = nextIdx(nums, nextIdx(nums, fast));
      if (slow === fast) {
        if (slow === nextIdx(nums, slow)) break; // cycle len 1
        return true;
      }
      if ((nums[slow] > 0) !== forward || (nums[fast] > 0) !== forward) break;
    }
  }
  return false;
}

function nextIdx(nums, i) {
  const n = nums.length;
  return ((i + nums[i]) % n + n) % n;
}
```

### Mark visited (optimize)

Mark nums[i] = 0 after failed search from i to avoid redundant work.

| Time | Space |
|------|-------|
| O(n²) worst; O(n) with marking | O(1) |

---

## 4. Pattern Cheat Sheet

| Problem | Structure | Key idea |
|---------|-----------|----------|
| **349** | Set | Unique intersection |
| **350** | Map freq | Multiplicity intersection |
| **457** | Set / fast-slow | Simulate circular jumps |

### Sort + two pointers alternative (350)

```js
function intersectTwoPointers(nums1, nums2) {
  nums1.sort((a, b) => a - b);
  nums2.sort((a, b) => a - b);
  const res = [];
  let i = 0, j = 0;
  while (i < nums1.length && j < nums2.length) {
    if (nums1[i] === nums2[j]) {
      res.push(nums1[i]);
      i++; j++;
    } else if (nums1[i] < nums2[j]) i++;
    else j++;
  }
  return res;
}
```

### Follow-ups

| Question | Answer |
|----------|--------|
| 349 vs 350 | Unique vs with multiplicity |
| 457 cycle length 1 | Invalid — must move to different index |
| 457 direction change | All steps in cycle same sign |

---

*End of Day 19 LeetCode*
