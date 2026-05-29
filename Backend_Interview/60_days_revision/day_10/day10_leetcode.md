# Day 10 — Heap & Priority Queue

---

# 1. Top K Frequent Elements

## Problem Statement

Given an integer array `nums` and integer `k`, return the `k` most frequent elements. Answer may be in any order.

---

# Example

```text
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1,2]
```

---

# Approach Hints

- Count frequencies with HashMap.
- Min-heap of size `k` on frequency (keep top k largest).
- Bucket sort: index = frequency (O(n) when range small).

---

# Core Idea (Walkthrough)

1. Build frequency map.
2. Min-heap stores `(freq, num)`; if size > k, poll smallest freq.
3. Extract all nums from heap.

---

# Java Solution

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> freq = new HashMap<>();
        for (int n : nums) freq.merge(n, 1, Integer::sum);

        PriorityQueue<int[]> minHeap = new PriorityQueue<>(
            (a, b) -> a[0] - b[0] // compare by frequency
        );

        for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
            minHeap.offer(new int[]{e.getValue(), e.getKey()});
            if (minHeap.size() > k) minHeap.poll();
        }

        int[] res = new int[k];
        for (int i = 0; i < k; i++) res[i] = minHeap.poll()[1];
        return res;
    }
}
```

---

# JavaScript Solution

```javascript
function topKFrequent(nums, k) {
  const freq = new Map();
  for (const n of nums) freq.set(n, (freq.get(n) || 0) + 1);

  const heap = [];
  const push = (item) => {
    heap.push(item);
    heap.sort((a, b) => a[0] - b[0]);
    if (heap.length > k) heap.shift();
  };

  for (const [num, count] of freq) push([count, num]);
  return heap.map((x) => x[1]);
}
```

---

# Time & Space Complexity

```text
Time:  O(n log k)
Space: O(n)
```

---

# 2. Merge K Sorted Lists

## Problem Statement

Given an array of `k` linked lists, each sorted in ascending order, merge all into one sorted linked list.

---

# Example

```text
Input: lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]
```

---

# Approach Hints

- Min-heap of size k: always pick smallest head among lists.
- Divide and conquer: merge pairs (like merge sort).
- Brute: collect all values, sort O(N log N) — acceptable but not optimal interview answer.

---

# Core Idea (Walkthrough)

1. Push head of each non-null list into min-heap (compare by node value).
2. Poll smallest, append to result, push its `next` if exists.

---

# Java Solution

```java
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        PriorityQueue<ListNode> pq = new PriorityQueue<>(
            (a, b) -> a.val - b.val
        );

        for (ListNode head : lists) {
            if (head != null) pq.offer(head);
        }

        ListNode dummy = new ListNode(0);
        ListNode tail = dummy;

        while (!pq.isEmpty()) {
            ListNode node = pq.poll();
            tail.next = node;
            tail = tail.next;
            if (node.next != null) pq.offer(node.next);
        }

        return dummy.next;
    }
}
```

---

# JavaScript Solution

```javascript
function mergeKLists(lists) {
  const nodes = lists.filter(Boolean).sort((a, b) => a.val - b.val);
  const dummy = { val: 0, next: null };
  let tail = dummy;

  while (nodes.length) {
    nodes.sort((a, b) => a.val - b.val);
    const node = nodes.shift();
    tail.next = node;
    tail = node;
    if (node.next) nodes.push(node.next);
  }

  return dummy.next;
}
```

---

# Time & Space Complexity

```text
Time:  O(N log k)   (N = total nodes)
Space: O(k)
```

---

# 3. Find Median from Data Stream

## Problem Statement

Design a structure that supports `addNum(int)` and `findMedian()` — median of all elements so far.

---

# Example

```text
addNum(1), addNum(2) → median 1.5
addNum(3) → median 2.0
```

---

# Approach Hints

- Two heaps: max-heap for lower half, min-heap for upper half.
- Balance sizes so max-heap has equal or one more element than min-heap.
- Median = top of max-heap or average of both tops.

---

# Core Idea (Walkthrough)

1. Add to max-heap (lower), move max to min-heap (upper).
2. If min-heap larger, move min back to max-heap.
3. Median from heap tops.

---

# Java Solution

```java
class MedianFinder {
    private final PriorityQueue<Integer> lower = new PriorityQueue<>(Collections.reverseOrder());
    private final PriorityQueue<Integer> upper = new PriorityQueue<>();

    public void addNum(int num) {
        lower.offer(num);
        upper.offer(lower.poll());
        if (lower.size() < upper.size()) {
            lower.offer(upper.poll());
        }
    }

    public double findMedian() {
        if (lower.size() > upper.size()) return lower.peek();
        return (lower.peek() + upper.peek()) / 2.0;
    }
}
```

---

# JavaScript Solution

```javascript
class MedianFinder {
  constructor() {
    this.lower = []; // max heap via negate
    this.upper = [];
  }

  addNum(num) {
    this.lower.push(-num);
    this.lower.sort((a, b) => a - b);
    this.upper.push(-this.lower.shift());
    this.upper.sort((a, b) => a - b);
    if (this.lower.length < this.upper.length) {
      this.lower.push(-this.upper.shift());
      this.lower.sort((a, b) => a - b);
    }
  }

  findMedian() {
    if (this.lower.length > this.upper.length) return -this.lower[0];
    return (-this.lower[0] + this.upper[0]) / 2;
  }
}
```

---

# Time & Space Complexity

```text
addNum:      O(log n)
findMedian:  O(1)
Space:       O(n)
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Top K Frequent | HashMap + Min-heap size k | O(n log k) |
| Merge K Lists | Min-heap of k heads | O(N log k) |
| Find Median Stream | Two heaps | O(log n) per add |

---

*End of Day 10 LeetCode*
