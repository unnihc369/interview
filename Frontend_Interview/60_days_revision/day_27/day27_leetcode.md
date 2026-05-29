# Day 27 — LeetCode (Heaps / Greedy)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 1046 | [Last Stone Weight](#1-last-stone-weight-1046) | Easy | Max-heap simulation |
| 1054 | [Distant Barcodes](#2-distant-barcodes-1054) | Medium | Max-heap + queue |
| 1353 | [Maximum Number of Events That Can Be Attended](#3-maximum-number-of-events-that-can-be-attended-1353) | Medium | Min-heap by end day |

---

## Table of Contents

1. [Last Stone Weight (1046)](#1-last-stone-weight-1046)
2. [Distant Barcodes (1054)](#2-distant-barcodes-1054)
3. [Maximum Number of Events (1353)](#3-maximum-number-of-events-that-can-be-attended-1353)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Last Stone Weight (1046)

### Problem (short)

Smash two heaviest stones; if equal both gone, else difference remains. Last weight or 0.

```
Input: stones = [2,7,4,1,8,1]
Output: 1
```

### Hint 1 — Max-heap

Always pop two largest, push difference if non-zero.

### Solution — JavaScript

```js
/**
 * @param {number[]} stones
 * @return {number}
 */
function lastStoneWeight(stones) {
  const heap = [...stones].sort((a, b) => b - a);

  while (heap.length > 1) {
    const a = heap.shift();
    const b = heap.shift();
    if (a !== b) {
      heap.push(a - b);
      heap.sort((a, b) => b - a);
    }
  }
  return heap.length ? heap[0] : 0;
}
```

| Time | Space |
|------|-------|
| O(n log n) | O(n) |

---

## 2. Distant Barcodes (1054)

### Problem (short)

Rearrange barcodes so no two adjacent equal. Return any valid arrangement.

Similar to LC 767 — max-heap on frequency with adjacency constraint.

### Solution — JavaScript

```js
/**
 * @param {number[]} barcodes
 * @return {number[]}
 */
function rearrangeBarcodes(barcodes) {
  const freq = new Map();
  for (const b of barcodes) freq.set(b, (freq.get(b) ?? 0) + 1);

  const heap = [...freq.entries()].sort((a, b) => b[1] - a[1]);
  const res = [];

  while (heap.length) {
    heap.sort((a, b) => b[1] - a[1]);
    let [val, count] = heap.shift();
    if (res.length && res[res.length - 1] === val) {
      if (!heap.length) return [];
      heap.sort((a, b) => b[1] - a[1]);
      const [v2, c2] = heap.shift();
      res.push(v2);
      if (c2 > 1) heap.push([v2, c2 - 1]);
      heap.push([val, count]);
    } else {
      res.push(val);
      if (count > 1) heap.push([val, count - 1]);
    }
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(n log n) | O(n) |

---

## 3. Maximum Number of Events That Can Be Attended (1353)

### Problem (short)

Events `[start, end]`. Attend one event per day. Maximize count.

### Hint 1 — Sort by start day

### Hint 2 — Min-heap of end days

For each day, push events starting that day. Pop events that ended before today. Attend event with **smallest end** (greedy).

### Solution — JavaScript

```js
/**
 * @param {number[][]} events
 * @return {number}
 */
function maxEvents(events) {
  events.sort((a, b) => a[0] - b[0]);
  const heap = [];
  let day = 0, i = 0, attended = 0;
  const n = events.length;

  while (i < n || heap.length) {
    if (!heap.length) day = events[i][0];
    while (i < n && events[i][0] <= day) {
      heap.push(events[i][1]);
      heap.sort((a, b) => a - b);
      i++;
    }
    while (heap.length && heap[0] < day) heap.shift();
    if (heap.length) {
      heap.shift();
      attended++;
    }
    day++;
  }
  return attended;
}
```

| Time | Space |
|------|-------|
| O(n log n) | O(n) |

---

## 4. Pattern Cheat Sheet

| Problem | Heap | Key |
|---------|------|-----|
| **1046** | Max | Smash top two |
| **1054** | Max freq | Like reorganize string |
| **1353** | Min end day | Greedy earliest deadline |

### Last stone — proper MaxHeap loop

```js
function lastStoneWeightHeap(stones) {
  const h = [...stones].sort((a, b) => b - a);
  while (h.length > 1) {
    const y = h.shift();
    const x = h.shift();
    if (y !== x) {
      h.push(y - x);
      h.sort((a, b) => b - a);
    }
  }
  return h[0] ?? 0;
}
```

### Follow-ups

| Question | Answer |
|----------|--------|
| 1046 one stone left | Return weight or 0 |
| 1054 impossible | Most freq > ceil(n/2) — but problem guarantees solvable |
| 1353 same day events | Can attend one per day; heap picks min end |

### Complexity summary

| Problem | Time | Space |
|---------|------|-------|
| 1046 | O(n log n) | O(n) |
| 1054 | O(n log n) | O(n) |
| 1353 | O(n log n) | O(n) |

---

*End of Day 27 LeetCode*
