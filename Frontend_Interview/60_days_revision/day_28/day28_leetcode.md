# Day 28 — LeetCode (Timed Mock: Heaps)

**Format:** Timed practice — 3 problems · **45 min total** · No peeking at solutions until attempt complete

| # | Problem | Difficulty | Target time |
|---|---------|------------|-------------|
| 215 | [Kth Largest Element in an Array](#mock-1--kth-largest-element-215) | Medium | 15 min |
| 347 | [Top K Frequent Elements](#mock-2--top-k-frequent-elements-347) | Medium | 15 min |
| 295 | [Find Median from Data Stream](#mock-3--find-median-from-data-stream-295) | Hard | 15 min |

---

## Table of Contents

1. [Mock Rules & Scoring](#1-mock-rules--scoring)
2. [Mock 1 — Kth Largest (215)](#mock-1--kth-largest-element-215)
3. [Mock 2 — Top K Frequent (347)](#mock-2--top-k-frequent-elements-347)
4. [Mock 3 — Median from Data Stream (295)](#mock-3--find-median-from-data-stream-295)
5. [Post-Mock Review Checklist](#5-post-mock-review-checklist)
6. [Solutions (Reveal After Attempt)](#6-solutions-reveal-after-attempt)
7. [Pattern Cheat Sheet](#7-pattern-cheat-sheet)

---

## 1. Mock Rules & Scoring

### Before you start

1. Timer: **45 minutes** total.
2. Implement `MedianFinder` as a **class** for problem 295.
3. State heap type and size invariant before coding.

### Scoring rubric

| Criteria | Points |
|----------|--------|
| Correct heap pattern | 3 |
| Working code | 3 |
| Complexity stated | 2 |
| Edge cases | 2 |

| Total | Grade |
|-------|-------|
| 25–30 | Heap patterns mastered |
| 18–24 | Review Day 22–24 |
| < 18 | Re-read heap cheat sheets |

---

## Mock 1 — Kth Largest Element (215)

### Problem (short)

Return the kth largest element in unsorted array.

```
Input: nums = [3,2,1,5,6,4], k = 2
Output: 5
```

### Hints (if stuck 5+ min)

<details>
<summary>Hint 1</summary>
Min-heap of size k — not max-heap of all.
</details>

<details>
<summary>Hint 2</summary>
When heap.size > k, pop smallest.
</details>

### Your workspace

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
function findKthLargest(nums, k) {
  // YOUR CODE
}
```

### Tests

```js
console.assert(findKthLargest([3,2,1,5,6,4], 2) === 5);
console.assert(findKthLargest([3,2,3,1,2,4,5,5,6], 4) === 4);
console.assert(findKthLargest([1], 1) === 1);
```

---

## Mock 2 — Top K Frequent Elements (347)

### Problem (short)

Return k most frequent elements.

```
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1,2]
```

### Hints

<details>
<summary>Hint 1</summary>
Count with Map first.
</details>

<details>
<summary>Hint 2</summary>
Min-heap size k on frequency, or bucket sort O(n).
</details>

### Your workspace

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number[]}
 */
function topKFrequent(nums, k) {
  // YOUR CODE
}
```

### Tests

```js
const r = topKFrequent([1,1,1,2,2,3], 2);
console.assert(r.length === 2 && r.includes(1) && r.includes(2));
console.assert(topKFrequent([1], 1)[0] === 1);
```

---

## Mock 3 — Find Median from Data Stream (295)

### Problem (short)

Implement:

- `addNum(num)` — insert from stream
- `findMedian()` — return median

```
addNum(1) → addNum(2) → findMedian() = 1.5
addNum(3) → findMedian() = 2
```

### Hints

<details>
<summary>Hint 1</summary>
Two heaps: max for lower half, min for upper.
</details>

<details>
<summary>Hint 2</summary>
Keep sizes balanced: maxLo.size >= minHi.size and diff <= 1.
</details>

### Your workspace

```js
class MedianFinder {
  constructor() {
    // YOUR CODE
  }
  addNum(num) {
    // YOUR CODE
  }
  findMedian() {
    // YOUR CODE
  }
}
```

### Tests

```js
const mf = new MedianFinder();
mf.addNum(1);
mf.addNum(2);
console.assert(mf.findMedian() === 1.5);
mf.addNum(3);
console.assert(mf.findMedian() === 2);
```

---

## 5. Post-Mock Review Checklist

- [ ] Said "min-heap size K" for 215 before coding?
- [ ] Used Map for frequencies in 347?
- [ ] Rebalanced heaps after every addNum in 295?
- [ ] Handled k = 1 and single element streams?

---

## 6. Solutions (Reveal After Attempt)

### 215 — Kth Largest

```js
function findKthLargest(nums, k) {
  const heap = [];
  const push = (v) => { heap.push(v); heap.sort((a, b) => a - b); };
  for (const n of nums) {
    push(n);
    if (heap.length > k) heap.shift();
  }
  return heap[0];
}
```

| Time | Space |
|------|-------|
| O(n log k) | O(k) |

### 347 — Top K Frequent

```js
function topKFrequent(nums, k) {
  const freq = new Map();
  for (const n of nums) freq.set(n, (freq.get(n) ?? 0) + 1);
  return [...freq.entries()]
    .sort((a, b) => b[1] - a[1])
    .slice(0, k)
    .map(([n]) => n);
}
```

| Time | Space |
|------|-------|
| O(n log n) | O(n) |

### 295 — Median Finder

```js
class MedianFinder {
  constructor() {
    this.lo = [];
    this.hi = [];
  }
  addNum(num) {
    if (!this.lo.length || num <= -this.lo[0]) {
      this.lo.push(-num);
      this.lo.sort((a, b) => a - b);
    } else {
      this.hi.push(num);
      this.hi.sort((a, b) => a - b);
    }
    if (this.lo.length > this.hi.length + 1) {
      this.hi.push(-this.lo.shift());
      this.hi.sort((a, b) => a - b);
    } else if (this.hi.length > this.lo.length) {
      this.lo.push(-this.hi.shift());
      this.lo.sort((a, b) => a - b);
    }
  }
  findMedian() {
    if (this.lo.length > this.hi.length) return -this.lo[0];
    return (-this.lo[0] + this.hi[0]) / 2;
  }
}
```

| addNum | findMedian |
|--------|------------|
| O(log n) | O(1) |

---

## 7. Pattern Cheat Sheet

| Problem | Pattern | One-liner |
|---------|---------|-----------|
| **215** | Min-heap size K | Root after scan |
| **347** | Map + top K | Freq desc slice |
| **295** | Two heaps | Balance lo/hi sizes |

---

*End of Day 28 LeetCode mock*
