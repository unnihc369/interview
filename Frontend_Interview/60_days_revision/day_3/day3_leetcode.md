# Day 3 — LeetCode (Stack & Queue Patterns)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 402 | [Remove K Digits](#1-remove-k-digits-402) | Medium | Monotonic Stack |
| 933 | [Number of Recent Calls](#2-number-of-recent-calls-933) | Easy | Queue / Sliding Window |
| 85 | [Maximal Rectangle](#3-maximal-rectangle-85) | Hard | Stack + DP |

---

## Table of Contents

1. [Remove K Digits (402)](#1-remove-k-digits-402)
2. [Number of Recent Calls (933)](#2-number-of-recent-calls-933)
3. [Maximal Rectangle (85)](#3-maximal-rectangle-85)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Remove K Digits (402)

### Problem (short)

Given non-negative integer string `num` and integer `k`, remove **k digits** so the remaining number is **smallest possible**. Return as string (no leading zeros; `"0"` if empty).

**Example**

```
Input:  num = "1432219", k = 3
Output: "1219"
Remove 4, 3, 2 → smallest is 1219

Input:  num = "10200", k = 1
Output: "200"   (remove leading 1, not middle 0)
```

### Hint 1 — Greedy with stack

To minimize number, remove **leftmost larger digits** when a smaller digit follows.

Use **monotonic increasing stack** of digits (as chars).

### Hint 2 — Algorithm

```
stack = []
for each digit d in num:
  while k > 0 AND stack not empty AND stack.top > d:
    pop stack, k--
  push d

If k still > 0 → remove from end of stack (remaining digits are increasing)
Join stack, strip leading zeros
If empty → "0"
```

### Hint 3 — Walkthrough `"1432219", k=3`

```
'1' → [1]
'4' → [1,4]
'3' → 4>3 pop, k=2 → [1,3]
'2' → 3>2 pop, k=1 → [1,2]
'2' → [1,2,2]
'1' → 2>1 pop, k=0 → [1,2,1]
'9' → [1,2,1,9]
Result: "1219" ✓
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

### JavaScript

```js
/**
 * @param {string} num
 * @param {number} k
 * @return {string}
 */
function removeKdigits(num, k) {
  const stack = [];

  for (const digit of num) {
    while (k > 0 && stack.length > 0 && stack[stack.length - 1] > digit) {
      stack.pop();
      k--;
    }
    stack.push(digit);
  }

  // Remove remaining from end if k > 0
  while (k > 0) {
    stack.pop();
    k--;
  }

  // Strip leading zeros
  let result = stack.join("").replace(/^0+/, "");
  return result === "" ? "0" : result;
}
```

### TypeScript

```ts
function removeKdigits(num: string, k: number): string {
  const stack: string[] = [];

  for (const digit of num) {
    while (k > 0 && stack.length > 0 && stack[stack.length - 1] > digit) {
      stack.pop();
      k--;
    }
    stack.push(digit);
  }

  while (k > 0) {
    stack.pop();
    k--;
  }

  const result = stack.join("").replace(/^0+/, "");
  return result === "" ? "0" : result;
}
```

### Common mistakes

- Removing from wrong end when k remains (pop from **end** of increasing sequence).
- Not handling leading zeros (`"10", k=1` → `"0"` not `"0"` from `"00"`).
- Forgetting `"0"` when result is empty.

---

## 2. Number of Recent Calls (933)

### Problem (short)

Implement `RecentCounter` class:
- `ping(t)` adds request at time `t` (strictly increasing)
- Returns count of requests in **[t-3000, t]** (inclusive)

**Example**

```
RecentCounter rc = new RecentCounter();
rc.ping(1);    // 1
rc.ping(100);  // 2
rc.ping(3001); // 3  (range [1, 3001])
rc.ping(3002); // 3  (range [2, 3002] — 1 dropped)
```

### Hint 1 — Queue (FIFO)

Store timestamps in queue. On each ping, remove timestamps **< t - 3000** from front.

### Hint 2 — Amortized O(1)

Each timestamp enqueued once, dequeued once → O(1) amortized per ping.

### Complexity

| ping() Time | Space |
|-------------|-------|
| O(1) amortized | O(W) where W = window size ≤ 3001 |

### JavaScript

```js
class RecentCounter {
  constructor() {
    this.queue = [];
  }

  /**
   * @param {number} t
   * @return {number}
   */
  ping(t) {
    this.queue.push(t);
    const cutoff = t - 3000;
    while (this.queue[0] < cutoff) {
      this.queue.shift();
    }
    return this.queue.length;
  }
}
```

### TypeScript

```ts
class RecentCounter {
  private queue: number[] = [];

  ping(t: number): number {
    this.queue.push(t);
    const cutoff = t - 3000;
    while (this.queue[0] < cutoff) {
      this.queue.shift();
    }
    return this.queue.length;
  }
}
```

### Optimization — avoid shift()

```ts
class RecentCounter {
  private queue: number[] = [];
  private head = 0;

  ping(t: number): number {
    this.queue.push(t);
    const cutoff = t - 3000;
    while (this.queue[this.head] < cutoff) {
      this.head++;
    }
    return this.queue.length - this.head;
  }
}
```

### Common mistakes

- Using `<` vs `<=` wrong for cutoff (range is **[t-3000, t]** inclusive).
- Not removing stale entries before returning count.

---

## 3. Maximal Rectangle (85)

### Problem (short)

Given binary matrix of `'0'` and `'1'`, find area of largest rectangle containing only `'1'`.

**Example**

```
Input:
[
  ["1","0","1","0","0"],
  ["1","0","1","1","1"],
  ["1","1","1","1","1"],
  ["1","0","0","1","0"]
]
Output: 6  (bottom row of 1s, width 3... actually 2x3 or 3x2 = 6)
```

### Hint 1 — Reduce to "Largest Rectangle in Histogram" (LC 84)

For each row, compute **height** of consecutive 1s above (including current row).
Run histogram max-rectangle algorithm on each row's heights.

### Hint 2 — Build heights row by row

```js
heights[r][c] = matrix[r][c] === '1'
  ? (heights[r-1][c] || 0) + 1
  : 0
```

### Hint 3 — Largest rectangle in histogram (stack)

For each row's `heights` array:
- Stack stores **indices** with increasing heights
- When current height < stack top → pop and compute area

```
area = height[popped] * (i - stack.top - 1)
// if stack empty: width = i
```

### Walkthrough — row 3 heights `[1, 0, 1, 2, 1]`

```
i=0 h=1: stack [0]
i=1 h=0: pop 0, area=1*1=1; stack []
i=2 h=1: stack [2]
i=3 h=2: stack [2,3]
i=4 h=1: 1<2 pop 3 area=2*1=2; 1<=1 pop 2 area=1*3=3
max so far updated...
```

### Complexity

| Time | Space |
|------|-------|
| O(rows × cols) | O(cols) |

### JavaScript

```js
/**
 * @param {character[][]} matrix
 * @return {number}
 */
function maximalRectangle(matrix) {
  if (!matrix.length || !matrix[0].length) return 0;

  const cols = matrix[0].length;
  const heights = new Array(cols).fill(0);
  let maxArea = 0;

  for (const row of matrix) {
    for (let c = 0; c < cols; c++) {
      heights[c] = row[c] === "1" ? heights[c] + 1 : 0;
    }
    maxArea = Math.max(maxArea, largestRectangleArea(heights));
  }
  return maxArea;
}

function largestRectangleArea(heights) {
  const stack = [];
  let max = 0;

  for (let i = 0; i <= heights.length; i++) {
    const h = i === heights.length ? 0 : heights[i];
    while (stack.length > 0 && heights[stack[stack.length - 1]] > h) {
      const idx = stack.pop();
      const height = heights[idx];
      const width = stack.length === 0 ? i : i - stack[stack.length - 1] - 1;
      max = Math.max(max, height * width);
    }
    stack.push(i);
  }
  return max;
}
```

### TypeScript

```ts
function maximalRectangle(matrix: string[][]): number {
  if (!matrix.length || !matrix[0].length) return 0;

  const cols = matrix[0].length;
  const heights = new Array<number>(cols).fill(0);
  let maxArea = 0;

  for (const row of matrix) {
    for (let c = 0; c < cols; c++) {
      heights[c] = row[c] === "1" ? heights[c] + 1 : 0;
    }
    maxArea = Math.max(maxArea, largestRectangleArea(heights));
  }
  return maxArea;
}

function largestRectangleArea(heights: number[]): number {
  const stack: number[] = [];
  let max = 0;

  for (let i = 0; i <= heights.length; i++) {
    const h = i === heights.length ? 0 : heights[i];
    while (stack.length > 0 && heights[stack[stack.length - 1]] > h) {
      const idx = stack.pop()!;
      const height = heights[idx];
      const width = stack.length === 0 ? i : i - stack[stack.length - 1] - 1;
      max = Math.max(max, height * width);
    }
    stack.push(i);
  }
  return max;
}
```

### Common mistakes

- Not resetting height to 0 when cell is `'0'`.
- Forgetting sentinel height `0` at end of histogram loop.
- Off-by-one in width calculation.

---

## 4. Pattern Cheat Sheet

| Problem | Pattern | Key |
|---------|---------|-----|
| **402 Remove K** | Monotonic increasing stack | Pop larger digits when smaller comes |
| **933 Recent Calls** | Queue sliding window | Dequeue timestamps outside [t-3000, t] |
| **85 Max Rectangle** | DP heights + histogram stack | Each row → LC 84 subproblem |

### When you see…

| Signal | Think |
|--------|-------|
| Remove k elements for optimal result | Monotonic stack (greedy) |
| Time-window counting | Queue with expiry |
| Max rectangle in binary matrix | Row histogram + stack |

### Solve order (revision)

1. **402** — Stack pop while top > current digit.
2. **933** — Queue; trim front when < t-3000.
3. **85** — Build heights per row; run histogram algorithm.

---

*End of Day 3 LeetCode*
