# Day 28 — Revision: Heaps

**Week 4 Review** · **Topics:** Min/max heap · Top-K · Two heaps · Merge K · Scheduling · Greedy + heap

---

## Table of Contents

1. [Heap Pattern Taxonomy](#1-heap-pattern-taxonomy)
2. [Top-K Problems](#2-top-k-problems)
3. [Two Heaps — Median](#3-two-heaps--median)
4. [Merge K Sorted](#4-merge-k-sorted)
5. [Greedy + Heap Scheduling](#5-greedy--heap-scheduling)
6. [Implementation Checklist](#6-implementation-checklist)
7. [Complexity Reference](#7-complexity-reference)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. Heap Pattern Taxonomy

| Pattern | Heap type | Examples |
|---------|-----------|----------|
| Kth largest | Min-heap size K | 215 |
| Kth smallest | Max-heap size K | — |
| Top K frequent | Min on freq | 347, 692 |
| Merge K sorted | Min on value | 23, 1167 |
| Running median | Max-lo + min-hi | 295 |
| Schedule by time | Min on timestamp | Day 27 MC |
| Greedy max avail | Max on freq | 767, 621 |

---

## 2. Top-K Problems

### Kth largest — min-heap size K

```js
for (const n of nums) {
  heap.push(n);
  if (heap.size() > k) heap.pop(); // pop min
}
return heap.peek();
```

### Top K frequent — bucket sort O(n)

When freq bounded by n, buckets indexed by count.

### K closest points — max-heap size K on distance²

---

## 3. Two Heaps — Median

Invariant:

- `maxLo` holds lower half (max at root)
- `minHi` holds upper half (min at root)
- `|maxLo| - |minHi| <= 1`

```js
addNum(n) {
  if (!maxLo.length || n <= -maxLo[0]) pushMax(n);
  else pushMin(n);
  rebalance();
}
findMedian() {
  return maxLo.length > minHi.length ? -maxLo[0] : (-maxLo[0] + minHi[0]) / 2;
}
```

---

## 4. Merge K Sorted

Template:

1. Push first element of each list with metadata (list id, index).
2. Pop min, append to result, push next from same list.

Complexity: O(N log k) where N = total elements, k = lists.

---

## 5. Greedy + Heap Scheduling

| Problem | Greedy rule |
|---------|-------------|
| 621 Task scheduler | Most frequent first; cooldown queue |
| 767 Reorganize | Max freq char not adjacent to last |
| 1353 Events | Attend event ending soonest |
| 1167 Connect sticks | Merge two smallest (Huffman) |

---

## 6. Implementation Checklist

Interview heap implementation must include:

- [ ] `push` — sift up
- [ ] `pop` — swap root/last, sift down
- [ ] `peek` — O(1)
- [ ] Custom comparator for objects `{ val, idx }`
- [ ] State which heap type for Kth largest vs smallest

### Common mistakes

| Mistake | Fix |
|---------|-----|
| Wrong heap for Kth largest | Use **min**-heap size K |
| Forget dedupe in merge K | Track visited (i,j) |
| Median imbalance | Rebalance after every add |

---

## 7. Complexity Reference

| Operation | Time |
|-----------|------|
| push / pop | O(log n) |
| heapify n items | O(n) |
| Top K with heap | O(n log k) |
| Merge K lists | O(N log k) |
| Median stream add | O(log n) |

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| Kth largest heap? | Min-heap size K [§2](#2-top-k-problems) |
| Median stream? | Two heaps [§3](#3-two-heaps--median) |
| Merge K lists? | Min-heap heads [§4](#4-merge-k-sorted) |
| Mock today | 215, 347, 295 timed |

---

## Day 28 Cheat Sheet

```
K largest     → min-heap size K
K smallest    → max-heap size K
Median        → maxLo + minHi balanced
Merge K       → pop min, push next
Schedule      → min-heap by time/freq
Mock          → 215, 347, 295 in 45 min
```

---

*End of Day 28 concepts*
