# Day 24 — LeetCode (Heaps / Greedy)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 295 | [Find Median from Data Stream](#1-find-median-from-data-stream-295) | Hard | Two heaps |
| 502 | [IPO](#2-ipo-502) | Hard | Greedy + max-heap |
| 1167 | [Minimum Cost to Connect Sticks](#3-minimum-cost-to-connect-sticks-1167) | Medium | Min-heap merge |

---

## Table of Contents

1. [Find Median from Data Stream (295)](#1-find-median-from-data-stream-295)
2. [IPO (502)](#2-ipo-502)
3. [Minimum Cost to Connect Sticks (1167)](#3-minimum-cost-to-connect-sticks-1167)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Find Median from Data Stream (295)

### Problem (short)

Implement `MedianFinder`: `addNum(num)` and `findMedian()` in O(log n) add.

### Hint 1 — Two heaps

Max-heap stores lower half; min-heap stores upper half. Balance sizes.

### Hint 2 — Invariant

`maxLo.size >= minHi.size` and `maxLo.size <= minHi.size + 1`.

### Solution — JavaScript

```js
class MedianFinder {
  constructor() {
    this.lo = [];
    this.hi = [];
  }

  _pushLo(x) {
    this.lo.push(-x);
    this.lo.sort((a, b) => a - b);
  }
  _pushHi(x) {
    this.hi.push(x);
    this.hi.sort((a, b) => a - b);
  }

  addNum(num) {
    if (!this.lo.length || num <= -this.lo[0]) this._pushLo(num);
    else this._pushHi(num);

    if (this.lo.length > this.hi.length + 1) {
      this._pushHi(-this.lo.shift());
    } else if (this.hi.length > this.lo.length) {
      this._pushLo(-this.hi.shift());
    }
  }

  findMedian() {
    if (this.lo.length > this.hi.length) return -this.lo[0];
    return (-this.lo[0] + this.hi[0]) / 2;
  }
}
```

| addNum | findMedian | Space |
|--------|------------|-------|
| O(log n) | O(1) | O(n) |

---

## 2. IPO (502)

### Problem (short)

`k` projects max, `w` initial capital. Each project `(capitalRequired, profit)`. Pick up to k projects maximizing final capital.

### Hint 1 — Greedy

Always do affordable project with **max profit**.

### Hint 2 — Two heaps or sort + max-heap

Sort by capital; push affordable to max-heap by profit.

### Solution — JavaScript

```js
/**
 * @param {number} k
 * @param {number} w
 * @param {number[]} profits
 * @param {number[]} capital
 * @return {number}
 */
function findMaximizedCapital(k, w, profits, capital) {
  const projects = capital.map((c, i) => [c, profits[i]]);
  projects.sort((a, b) => a[0] - b[0]);
  const heap = [];
  let i = 0;

  for (let j = 0; j < k; j++) {
    while (i < projects.length && projects[i][0] <= w) {
      heap.push(projects[i][1]);
      heap.sort((a, b) => b - a);
      i++;
    }
    if (!heap.length) break;
    w += heap.shift();
  }
  return w;
}
```

| Time | Space |
|------|-------|
| O(n log n) | O(n) |

---

## 3. Minimum Cost to Connect Sticks (1167)

### Problem (short)

Combine sticks; cost = sum of lengths each merge. Minimize total cost (Huffman-like).

```
Input: sticks = [2,4,3]
Output: 14  → merge 2+3=5 (cost 5), then 5+4=9 (cost 9) → 14
```

### Hint 1 — Always merge two smallest

Min-heap — same as LC 1167 / Huffman coding.

### Solution — JavaScript

```js
/**
 * @param {number[]} sticks
 * @return {number}
 */
function connectSticks(sticks) {
  const heap = [...sticks].sort((a, b) => a - b);
  let cost = 0;

  while (heap.length > 1) {
    const a = heap.shift();
    const b = heap.shift();
    const sum = a + b;
    cost += sum;
    heap.push(sum);
    heap.sort((a, b) => a - b);
  }
  return cost;
}
```

| Time | Space |
|------|-------|
| O(n log n) | O(n) |

---

## 4. Pattern Cheat Sheet

| Problem | Heap | Key |
|---------|------|-----|
| **295** | Max-lo + min-hi | Balance sizes |
| **502** | Max profit heap | Greedy affordable |
| **1167** | Min-heap | Merge two smallest |

### Follow-ups

| Question | Answer |
|----------|--------|
| 295 why max on lower half? | Need instant access to largest of lower 50% |
| 502 greedy proof | Max profit among affordable always optimal |
| 1167 Huffman link | Same as optimal prefix codes |

### Test cases

```js
const mf = new MedianFinder();
mf.addNum(1); mf.addNum(2);
console.assert(mf.findMedian() === 1.5);

findMaximizedCapital(2, 0, [1,2,3], [0,1,1]); // 4
connectSticks([2,4,3]); // 14
```

### Complexity summary

| Problem | add / run | Space |
|---------|-----------|-------|
| 295 | O(log n) / O(1) | O(n) |
| 502 | O(n log n) | O(n) |
| 1167 | O(n log n) | O(n) |

---

*End of Day 24 LeetCode*
