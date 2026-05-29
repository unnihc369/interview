# Day 21 — Revision: Two Pointers

**Week 3 Review** · **Topics:** Opposite ends · Same direction · Fast/slow · Dutch flag · Merge · Intervals

---

## Table of Contents

1. [Pattern Taxonomy](#1-pattern-taxonomy)
2. [Opposite Ends](#2-opposite-ends)
3. [Same-Direction (Read/Write)](#3-same-direction-readwrite)
4. [Fast & Slow Pointers](#4-fast--slow-pointers)
5. [Dutch National Flag](#5-dutch-national-flag)
6. [Merge & Intervals](#6-merge--intervals)
7. [Sort + Two Pointers](#7-sort--two-pointers)
8. [Complexity Reference](#8-complexity-reference)
9. [Interview Quick Index](#9-interview-quick-index)

---

## 1. Pattern Taxonomy

| Pattern | Pointer setup | Example problems |
|---------|---------------|------------------|
| Opposite ends | `l=0`, `r=n-1` | 11, 15, 167, 977 |
| Read / write | `read`, `write` | 283, 27 |
| Fast / slow | `slow`, `fast=2*slow` | 141, 142, 876 |
| Dutch flag | `lo`, `mid`, `hi` | 75 |
| Merge | `i`, `j` on two arrays | 88, 986 |
| Fix + pair | outer `i`, inner `l/r` | 15, 18, 259 |

---

## 2. Opposite Ends

### When to use

- Input is **sorted** (or sortable)
- Pair/triplet sum problems
- Max area / container
- Palindrome check

### Template

```js
function oppositeTemplate(arr) {
  let l = 0, r = arr.length - 1;
  while (l < r) {
    // compare arr[l] and arr[r]
    if (/* need larger sum */) l++;
    else r--;
  }
}
```

### Key problems recap

| LC | Move rule |
|----|-----------|
| 11 | Move shorter height |
| 167 | sum vs target |
| 977 | Larger square from ends |

---

## 3. Same-Direction (Read/Write)

### When to use

- In-place removal / compaction
- Move zeroes / filter in-place

### Template

```js
function compact(arr, pred) {
  let write = 0;
  for (let read = 0; read < arr.length; read++) {
    if (pred(arr[read])) arr[write++] = arr[read];
  }
  return write;
}
```

---

## 4. Fast & Slow Pointers

### When to use

- Linked list cycle (141, 142)
- Middle of list (876)
- Happy number (202)

```js
function hasCycle(head) {
  let slow = head, fast = head;
  while (fast?.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }
  return false;
}
```

---

## 5. Dutch National Flag

Three-way partition in one pass — 0, 1, 2 or low/mid/high.

```js
function dutchFlag(nums) {
  let lo = 0, mid = 0, hi = nums.length - 1;
  while (mid <= hi) {
    if (nums[mid] === 0) { swap(lo++, mid++); }
    else if (nums[mid] === 1) { mid++; }
    else { swap(mid, hi--); }
  }
}
```

---

## 6. Merge & Intervals

### Merge sorted arrays

Backward write avoids overwrite — LC 88.

### Interval intersection

Advance pointer whose interval ends first — LC 986.

---

## 7. Sort + Two Pointers

| K-sum | Approach |
|-------|----------|
| 2Sum unsorted | Hash map O(n) |
| 2Sum sorted | Two pointers O(n) |
| 3Sum | Sort + fix i + L/R |
| 4Sum | Sort + fix i,j + L/R |
| 3Sum smaller | Sort + fix i + count |

Always **skip duplicates** at each fixed index.

---

## 8. Complexity Reference

| Pattern | Time | Space |
|---------|------|-------|
| Opposite ends | O(n) | O(1) |
| Sort + 2 pointers | O(n log n) or O(n²) | O(1) |
| Merge two lists | O(n+m) | O(1) in-place |
| Dutch flag | O(n) | O(1) |

---

## 9. Interview Quick Index

| Question | Section |
|----------|---------|
| When opposite vs hash map? | [§2](#2-opposite-ends) — sorted → two ptr |
| Move zeroes pattern | [§3](#3-same-direction-readwrite) |
| Cycle detection | [§4](#4-fast--slow-pointers) |
| 3Sum duplicate skip | [§7](#7-sort--two-pointers) |

---

## Day 21 Cheat Sheet

```
Sorted pair sum     → L/R opposite
In-place filter     → read/write
3-way partition     → lo/mid/hi
Merge               → from end or two heads
K-sum               → sort + (K-2) loops + L/R
Mock today          → 11, 15, 42 timed
```

---

*End of Day 21 concepts*
