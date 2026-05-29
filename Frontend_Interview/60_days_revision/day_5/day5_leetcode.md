# Day 5 — LeetCode (Stack Span & Priority Queue)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 901 | [Online Stock Span](#1-online-stock-span-901) | Medium | Monotonic Stack |
| 946 | [Validate Stack Sequences](#2-validate-stack-sequences-946) | Medium | Stack Simulation |
| 407 | [Trapping Rain Water II](#3-trapping-rain-water-ii-407) | Hard | Min-Heap + BFS |

---

## Table of Contents

1. [Online Stock Span (901)](#1-online-stock-span-901)
2. [Validate Stack Sequences (946)](#2-validate-stack-sequences-946)
3. [Trapping Rain Water II (407)](#3-trapping-rain-water-ii-407)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Online Stock Span (901)

### Problem (short)

Design class that computes **stock span**: consecutive days (including today) where price was **≤ today's price**.

**Example**

```
StockSpan span = new StockSpan();
span.next(100); // 1
span.next(80);  // 1
span.next(60);  // 1
span.next(70);  // 2  (60, 70)
span.next(60);  // 1
span.next(75);  // 4  (60, 70, 60, 75)
span.next(85);  // 6
```

### Hint 1 — Monotonic decreasing stack

Stack stores `(price, span)` pairs in decreasing price order.

### Hint 2 — On new price

```
span = 1
while stack not empty AND price >= stack.top.price:
  span += stack.top.span
  pop
push (price, span)
return span
```

### Walkthrough — prices 100, 80, 60, 70

```
100 → stack [(100,1)] → span 1
80  → stack [(100,1), (80,1)] → span 1
60  → stack [(100,1), (80,1), (60,1)] → span 1
70  → 70>=60 pop span+=1; 70<80 stop → push (70,2) → span 2
```

### Complexity

| next() Time | Space |
|-------------|-------|
| O(1) amortized | O(n) |

### JavaScript

```js
class StockSpanner {
  constructor() {
    this.stack = []; // [price, span]
  }

  /**
   * @param {number} price
   * @return {number}
   */
  next(price) {
    let span = 1;
    while (
      this.stack.length > 0 &&
      this.stack[this.stack.length - 1][0] <= price
    ) {
      span += this.stack.pop()[1];
    }
    this.stack.push([price, span]);
    return span;
  }
}
```

### TypeScript

```ts
class StockSpanner {
  private stack: [number, number][] = [];

  next(price: number): number {
    let span = 1;
    while (this.stack.length > 0 && this.stack[this.stack.length - 1][0] <= price) {
      span += this.stack.pop()![1];
    }
    this.stack.push([price, span]);
    return span;
  }
}
```

---

## 2. Validate Stack Sequences (946)

### Problem (short)

Given `pushed` and `popped` arrays, return whether `popped` could be result of stack operations on `pushed`.

**Example**

```
Input:  pushed = [1,2,3,4,5], popped = [4,5,3,2,1]
Output: true

Input:  pushed = [1,2,3,4,5], popped = [4,3,5,1,2]
Output: false
```

### Hint 1 — Simulate with stack

Push from `pushed`. After each push, try popping while stack top === popped[i].

### Hint 2 — Algorithm

```
stack = []
j = 0  (index in popped)

for each x in pushed:
  stack.push(x)
  while stack not empty AND stack.top == popped[j]:
    stack.pop()
    j++

return stack is empty
```

### Walkthrough — pushed [1,2,3], popped [2,3,1]

```
push 1 → [1], top≠2
push 2 → [1,2], top=2 pop j=1
top=1≠3, push 3 → [1,3], top=3 pop j=2
top=1=1 pop j=3
stack empty ✓
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

### JavaScript

```js
/**
 * @param {number[]} pushed
 * @param {number[]} popped
 * @return {boolean}
 */
function validateStackSequences(pushed, popped) {
  const stack = [];
  let j = 0;

  for (const x of pushed) {
    stack.push(x);
    while (stack.length > 0 && stack[stack.length - 1] === popped[j]) {
      stack.pop();
      j++;
    }
  }

  return stack.length === 0;
}
```

### TypeScript

```ts
function validateStackSequences(pushed: number[], popped: number[]): boolean {
  const stack: number[] = [];
  let j = 0;

  for (const x of pushed) {
    stack.push(x);
    while (stack.length > 0 && stack[stack.length - 1] === popped[j]) {
      stack.pop();
      j++;
    }
  }

  return stack.length === 0;
}
```

---

## 3. Trapping Rain Water II (407)

### Problem (short)

Given `m x n` height map, compute total trapped rain water (3D version of LC 42).

**Example**

```
Input: heightMap = [
  [1,4,3,1,3,2],
  [3,2,1,3,2,4],
  [2,3,3,2,3,1]
]
Output: 4
```

### Hint 1 — Not simple two-pointer (that's 1D)

Need **priority queue (min-heap)** + BFS from boundary.

### Hint 2 — Intuition

Water level at cell = min of max heights on path from boundary.  
Process cells from **lowest boundary** inward (like Dijkstra).

### Hint 3 — Algorithm

```
1. Push all boundary cells into min-heap; mark visited
2. While heap not empty:
   pop cell (h, r, c)
   for each neighbor (nr, nc):
     if not visited:
       water += max(0, h - heightMap[nr][nc])
       push (max(h, heightMap[nr][nc]), nr, nc)
       mark visited
3. return total water
```

### Why min-heap?

Always expand from **lowest wall** — guarantees water level calculation is correct.

### Complexity

| Time | Space |
|------|-------|
| O(m·n·log(m·n)) | O(m·n) |

### JavaScript

```js
/**
 * @param {number[][]} heightMap
 * @return {number}
 */
function trapRainWater(heightMap) {
  const m = heightMap.length;
  if (m <= 2) return 0;
  const n = heightMap[0].length;
  if (n <= 2) return 0;

  const visited = Array.from({ length: m }, () => new Array(n).fill(false));
  const heap = new MinHeap((a, b) => a[0] - b[0]); // [height, r, c]

  // Push boundaries
  for (let r = 0; r < m; r++) {
    for (let c = 0; c < n; c++) {
      if (r === 0 || r === m - 1 || c === 0 || c === n - 1) {
        heap.push([heightMap[r][c], r, c]);
        visited[r][c] = true;
      }
    }
  }

  const dirs = [[0, 1], [0, -1], [1, 0], [-1, 0]];
  let water = 0;

  while (heap.size() > 0) {
    const [h, r, c] = heap.pop();
    for (const [dr, dc] of dirs) {
      const nr = r + dr;
      const nc = c + dc;
      if (nr < 0 || nr >= m || nc < 0 || nc >= n || visited[nr][nc]) continue;
      visited[nr][nc] = true;
      water += Math.max(0, h - heightMap[nr][nc]);
      heap.push([Math.max(h, heightMap[nr][nc]), nr, nc]);
    }
  }
  return water;
}

// Simple min-heap helper
class MinHeap {
  constructor(compare) {
    this.data = [];
    this.compare = compare;
  }
  push(val) {
    this.data.push(val);
    this._bubbleUp(this.data.length - 1);
  }
  pop() {
    const top = this.data[0];
    const last = this.data.pop();
    if (this.data.length) {
      this.data[0] = last;
      this._sinkDown(0);
    }
    return top;
  }
  size() { return this.data.length; }
  _bubbleUp(i) {
    while (i > 0) {
      const parent = (i - 1) >> 1;
      if (this.compare(this.data[i], this.data[parent]) >= 0) break;
      [this.data[i], this.data[parent]] = [this.data[parent], this.data[i]];
      i = parent;
    }
  }
  _sinkDown(i) {
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

### TypeScript

```ts
function trapRainWater(heightMap: number[][]): number {
  const m = heightMap.length;
  if (m <= 2) return 0;
  const n = heightMap[0].length;
  if (n <= 2) return 0;

  const visited: boolean[][] = Array.from({ length: m }, () =>
    new Array(n).fill(false)
  );
  type Cell = [number, number, number]; // height, r, c
  const heap = new MinHeap<Cell>((a, b) => a[0] - b[0]);

  for (let r = 0; r < m; r++) {
    for (let c = 0; c < n; c++) {
      if (r === 0 || r === m - 1 || c === 0 || c === n - 1) {
        heap.push([heightMap[r][c], r, c]);
        visited[r][c] = true;
      }
    }
  }

  const dirs = [[0, 1], [0, -1], [1, 0], [-1, 0]] as const;
  let water = 0;

  while (heap.size() > 0) {
    const [h, r, c] = heap.pop()!;
    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr < 0 || nr >= m || nc < 0 || nc >= n || visited[nr][nc]) continue;
      visited[nr][nc] = true;
      water += Math.max(0, h - heightMap[nr][nc]);
      heap.push([Math.max(h, heightMap[nr][nc]), nr, nc]);
    }
  }
  return water;
}
```

### Common mistakes

- Using 1D two-pointer approach on 2D grid.
- Not using `max(h, neighborHeight)` when pushing neighbor.
- Forgetting to mark visited before pushing.

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | Key |
|---------|---------|-----|
| **901 Stock Span** | Monotonic stack | Accumulate span when popping lower prices |
| **946 Validate Stack** | Stack simulation | Pop while top === next in popped |
| **407 Rain Water II** | Min-heap BFS | Expand from lowest boundary inward |

### When you see…

| Signal | Think |
|--------|-------|
| Consecutive previous smaller/equal | Monotonic stack with accumulated count |
| Can this pop sequence happen? | Simulate push/pop |
| 2D water trapping | Min-heap from boundary |

### Solve order (revision)

1. **901** — Pop lower/equal prices; add their span.
2. **946** — Push all; pop matching popped[j].
3. **407** — Boundary into heap; BFS with water += max(0, h - neighbor).

---

*End of Day 5 LeetCode*
