# Day 22 — LeetCode (Heaps)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 215 | [Kth Largest Element in an Array](#1-kth-largest-element-in-an-array-215) | Medium | Min-heap size K |
| 347 | [Top K Frequent Elements](#2-top-k-frequent-elements-347) | Medium | Hash + heap |
| 23 | [Merge k Sorted Lists](#3-merge-k-sorted-lists-23) | Hard | Min-heap of heads |

---

## Table of Contents

1. [Kth Largest Element (215)](#1-kth-largest-element-in-an-array-215)
2. [Top K Frequent Elements (347)](#2-top-k-frequent-elements-347)
3. [Merge k Sorted Lists (23)](#3-merge-k-sorted-lists-23)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Kth Largest Element in an Array (215)

### Problem (short)

Return the **kth largest** element in unsorted array (not kth distinct).

```
Input: nums = [3,2,1,5,6,4], k = 2
Output: 5
```

### Hint 1 — Sort

`nums.sort((a,b)=>b-a)[k-1]` — O(n log n).

### Hint 2 — Min-heap of size k

If heap size > k, pop min. Root is kth largest.

### Solution — JavaScript (min-heap simulation with sort bucket)

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number}
 */
function findKthLargest(nums, k) {
  const heap = [];
  const push = (v) => {
    heap.push(v);
    heap.sort((a, b) => a - b);
  };
  for (const n of nums) {
    push(n);
    if (heap.length > k) heap.shift();
  }
  return heap[0];
}
```

### Proper MinHeap class solution

```js
function findKthLargest(nums, k) {
  const h = new MinHeap();
  for (const n of nums) {
    h.push(n);
    if (h.size() > k) h.pop();
  }
  return h.peek();
}
```

| Time | Space |
|------|-------|
| O(n log k) | O(k) |

---

## 2. Top K Frequent Elements (347)

### Problem (short)

Return **k** most frequent elements (any order).

```
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1,2]
```

### Hint 1 — Count frequencies

`Map` value → count.

### Hint 2 — Min-heap size k on frequency

Keep k highest freq; root is lowest among top k.

### Solution — JavaScript

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number[]}
 */
function topKFrequent(nums, k) {
  const freq = new Map();
  for (const n of nums) freq.set(n, (freq.get(n) ?? 0) + 1);

  const entries = [...freq.entries()];
  entries.sort((a, b) => a[1] - b[1]);
  return entries.slice(-k).map(([num]) => num);
}
```

### Bucket sort O(n) alternative

```js
function topKFrequentBucket(nums, k) {
  const freq = new Map();
  for (const n of nums) freq.set(n, (freq.get(n) ?? 0) + 1);
  const buckets = Array.from({ length: nums.length + 1 }, () => []);
  for (const [num, f] of freq) buckets[f].push(num);
  const res = [];
  for (let i = buckets.length - 1; i >= 0 && res.length < k; i--) {
    res.push(...buckets[i]);
  }
  return res.slice(0, k);
}
```

| Time | Space |
|------|-------|
| O(n log k) heap; O(n) bucket | O(n) |

---

## 3. Merge k Sorted Lists (23)

### Problem (short)

Merge **k** sorted linked lists into one sorted list.

### Hint 1 — Min-heap of list heads

Compare `node.val`, pop smallest, push `node.next`.

### Solution — JavaScript

```js
/**
 * @param {ListNode[]} lists
 * @return {ListNode}
 */
function mergeKLists(lists) {
  const nodes = lists.filter(Boolean);
  nodes.sort((a, b) => a.val - b.val);

  const dummy = { val: 0, next: null };
  let tail = dummy;

  while (nodes.length) {
    nodes.sort((a, b) => a.val - b.val);
    const node = nodes.shift();
    tail.next = node;
    tail = tail.next;
    if (node.next) nodes.push(node.next);
  }
  return dummy.next;
}
```

### Production heap version (concept)

Push `{ val, node }` into min-heap; O(N log k) where N = total nodes.

| Time | Space |
|------|-------|
| O(N log k) | O(k) |

---

## 4. Pattern Cheat Sheet

| Problem | Heap | Key |
|---------|------|-----|
| **215** | Min size K | Root = kth largest |
| **347** | Min on freq | Or bucket sort |
| **23** | Min on node.val | Pop, push next |

### When you see…

| Signal | Think |
|--------|-------|
| Kth largest/smallest | Size-K heap (opposite type) |
| Top K by frequency | Map + heap or buckets |
| Merge K sorted | Heap of K heads |

---

*End of Day 22 LeetCode*
