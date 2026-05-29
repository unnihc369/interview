# Day 22 — Binary Heap (Min / Max)

**Week 4 — Heaps** · **Topics:** Min-heap · Max-heap · Priority queue · Heapify · Top-K

---

## Table of Contents

1. [Heap Fundamentals](#1-heap-fundamentals)
2. [Array Representation](#2-array-representation)
3. [Min-Heap vs Max-Heap](#3-min-heap-vs-max-heap)
4. [Core Operations](#4-core-operations)
5. [JavaScript Heap Implementation](#5-javascript-heap-implementation)
6. [Top-K Pattern](#6-top-k-pattern)
7. [Merge K Sorted Lists Preview](#7-merge-k-sorted-lists-preview)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. Heap Fundamentals

### Definition

A **binary heap** is a complete binary tree satisfying the **heap property**:

| Type | Property |
|------|----------|
| **Min-heap** | Parent ≤ both children |
| **Max-heap** | Parent ≥ both children |

Root is always **minimum** (min-heap) or **maximum** (max-heap).

### Complete binary tree

All levels filled left-to-right except possibly last level — enables **array storage** without pointers.

### Use cases

| Use | Heap type |
|-----|-----------|
| Priority queue | min or max |
| Kth largest | min-heap size K |
| Kth smallest | max-heap size K |
| Merge K lists | min-heap of heads |
| Dijkstra / scheduling | min-heap |

---

## 2. Array Representation

For node at index `i`:

| Relation | Index |
|----------|-------|
| Parent | `Math.floor((i - 1) / 2)` |
| Left child | `2 * i + 1` |
| Right child | `2 * i + 2` |

```js
// Min-heap array example
const heap = [1, 3, 2, 7, 5, 4, 6];
//        1
//      /   \
//     3     2
//    / \   / \
//   7  5  4   6
```

---

## 3. Min-Heap vs Max-Heap

```js
// Min-heap peek
heap[0]; // smallest

// Max-heap — negate values or invert compare
class MaxHeap {
  compare(a, b) { return b - a; } // larger has higher priority
}
```

### Trick: get max with min-heap

Store negated values, or use custom comparator.

---

## 4. Core Operations

| Operation | Time | Description |
|-----------|------|-------------|
| `peek` | O(1) | Root element |
| `push` | O(log n) | Add at end, **sift up** |
| `pop` | O(log n) | Remove root, move last to root, **sift down** |
| `heapify` | O(n) | Build heap from array |

### Sift up

While node < parent, swap with parent and repeat.

### Sift down

Swap with smaller child (min-heap) until heap property restored.

---

## 5. JavaScript Heap Implementation

```js
class MinHeap {
  constructor(compare = (a, b) => a - b) {
    this.data = [];
    this.compare = compare;
  }

  size() { return this.data.length; }
  peek() { return this.data[0]; }

  push(val) {
    this.data.push(val);
    this._siftUp(this.data.length - 1);
  }

  pop() {
    if (this.data.length === 0) return undefined;
    const top = this.data[0];
    const last = this.data.pop();
    if (this.data.length) {
      this.data[0] = last;
      this._siftDown(0);
    }
    return top;
  }

  _siftUp(i) {
    while (i > 0) {
      const p = Math.floor((i - 1) / 2);
      if (this.compare(this.data[i], this.data[p]) >= 0) break;
      [this.data[i], this.data[p]] = [this.data[p], this.data[i]];
      i = p;
    }
  }

  _siftDown(i) {
    const n = this.data.length;
    while (true) {
      let smallest = i;
      const l = 2 * i + 1, r = 2 * i + 2;
      if (l < n && this.compare(this.data[l], this.data[smallest]) < 0) smallest = l;
      if (r < n && this.compare(this.data[r], this.data[smallest]) < 0) smallest = r;
      if (smallest === i) break;
      [this.data[i], this.data[smallest]] = [this.data[smallest], this.data[i]];
      i = smallest;
    }
  }
}
```

### Using in interviews

Many use **no library** — implement or use array + `sort` for small K (mention tradeoff).

---

## 6. Top-K Pattern

### Kth largest (LC 215)

Maintain **min-heap of size K** — root is Kth largest when done.

```js
function findKthLargest(nums, k) {
  const heap = new MinHeap();
  for (const n of nums) {
    heap.push(n);
    if (heap.size() > k) heap.pop();
  }
  return heap.peek();
}
```

| Approach | Time | Space |
|----------|------|-------|
| Sort | O(n log n) | O(1) |
| Min-heap size K | O(n log k) | O(k) |
| Quickselect | O(n) avg | O(1) |

---

## 7. Merge K Sorted Lists Preview

Push head of each list into min-heap keyed by node value — pop, append, push next — LC 23.

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| Parent/child index formulas | [§2](#2-array-representation) |
| Kth largest heap type | [§6](#6-top-k-pattern) — min-heap size K |
| push/pop complexity | [§4](#4-core-operations) — O(log n) |
| heapify vs n pushes | heapify O(n); n pushes O(n log n) |

---

## Day 22 Cheat Sheet

```
Min-heap  → parent ≤ children; root = min
Max-heap  → parent ≥ children; root = max
Array idx → parent (i-1)//2, children 2i+1, 2i+2
push      → sift up O(log n)
pop       → sift down O(log n)
K largest → min-heap size K
K smallest→ max-heap size K
```

---

*End of Day 22 concepts*
