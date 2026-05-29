# Day 2 — LeetCode (Monotonic Stack & Deque)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 739 | [Daily Temperatures](#1-daily-temperatures-739) | Medium | Monotonic Stack |
| 735 | [Asteroid Collision](#2-asteroid-collision-735) | Medium | Stack Simulation |
| 239 | [Sliding Window Maximum](#3-sliding-window-maximum-239) | Hard | Monotonic Deque |

---

## Table of Contents

1. [Daily Temperatures (739)](#1-daily-temperatures-739)
2. [Asteroid Collision (735)](#2-asteroid-collision-735)
3. [Sliding Window Maximum (239)](#3-sliding-window-maximum-239)
4. [Pattern Cheat Sheet](#4-pattern-cheat-sheet)

---

## 1. Daily Temperatures (739)

### Problem (short)

Given daily temperatures, return an array `answer` where `answer[i]` is the number of days until a **warmer** temperature. If none, `0`.

**Example**

```
Input:  temperatures = [73, 74, 75, 71, 69, 72, 76, 73]
Output: [1, 1, 4, 2, 1, 1, 0, 0]

Day 0 (73°) → next warmer is day 1 (74°) → wait 1 day
Day 2 (75°) → next warmer is day 6 (76°) → wait 4 days
```

### Hint 1 — Brute force is O(n²)

For each `i`, scan forward until `temperatures[j] > temperatures[i]`. Works but too slow for large `n`.

### Hint 2 — Monotonic decreasing stack

Keep a stack of **indices** where temperatures are in **decreasing order** (top = most recent / hottest in stack).

When current temp is **warmer** than stack top → we found the answer for that index.

```
Stack stores indices waiting for a warmer day
Pop while current > temperatures[stack.top]
  → answer[popped] = i - popped
Push current index i
```

### Hint 3 — Walkthrough `[73, 74, 75, 71, 69, 72, 76, 73]`

```
i=0 (73): stack [0]
i=1 (74): 74>73 → pop 0, ans[0]=1-0=1; push 1 → [1]
i=2 (75): 75>74 → pop 1, ans[1]=1; push 2 → [2]
i=3 (71): push 3 → [2,3]
i=4 (69): push 4 → [2,3,4]
i=5 (72): 72>69 pop 4 ans[4]=1; 72>71 pop 3 ans[3]=2; push 5 → [2,5]
i=6 (76): pop 5,3,2 → ans[5]=1, ans[3]=4, ans[2]=4; push 6 → [6]
i=7 (73): push 7 → [6,7]
Remaining indices → 0 (default)
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

Each index pushed and popped at most once.

### JavaScript

```js
/**
 * @param {number[]} temperatures
 * @return {number[]}
 */
function dailyTemperatures(temperatures) {
  const n = temperatures.length;
  const answer = new Array(n).fill(0);
  const stack = []; // indices

  for (let i = 0; i < n; i++) {
    while (
      stack.length > 0 &&
      temperatures[i] > temperatures[stack[stack.length - 1]]
    ) {
      const prev = stack.pop();
      answer[prev] = i - prev;
    }
    stack.push(i);
  }
  return answer;
}
```

### TypeScript

```ts
function dailyTemperatures(temperatures: number[]): number[] {
  const n = temperatures.length;
  const answer = new Array<number>(n).fill(0);
  const stack: number[] = [];

  for (let i = 0; i < n; i++) {
    while (
      stack.length > 0 &&
      temperatures[i] > temperatures[stack[stack.length - 1]]
    ) {
      const prev = stack.pop()!;
      answer[prev] = i - prev;
    }
    stack.push(i);
  }
  return answer;
}
```

### Common mistakes

- Storing temperatures in stack instead of **indices** (can't compute day difference).
- Using increasing stack (wrong direction for "next greater").
- Forgetting to initialize answer array with zeros.

---

## 2. Asteroid Collision (735)

### Problem (short)

Asteroids move in a line. Positive = moving right, negative = moving left. Same size → both destroy. Smaller destroyed by larger. Return final state.

**Example**

```
Input:  asteroids = [5, 10, -5]
Output: [5, 10]

Input:  asteroids = [8, -8]
Output: []   (both destroyed)

Input:  asteroids = [10, 2, -5]
Output: [10]  (2 and -5 collide, -5 wins; then 10 beats -5)
```

### Hint 1 — Simulate with stack

Process left to right. Stack = surviving asteroids so far.

### Hint 2 — Collision rules

Only collide when: **stack top > 0** AND **current < 0** (right-moving meets left-moving).

```
While stack not empty AND top > 0 AND current < 0:
  if |top| < |current|  → pop top, current survives (continue loop)
  if |top| > |current|  → current destroyed (skip push)
  if |top| == |current| → both destroyed (skip push)
If current not destroyed → push current
```

### Hint 3 — Walkthrough `[10, 2, -5]`

```
10 → stack [10]
2  → stack [10, 2]
-5 → collide with 2: |2| < |-5| → pop 2
   → collide with 10: |10| > |-5| → -5 destroyed
Final: [10]
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

### JavaScript

```js
/**
 * @param {number[]} asteroids
 * @return {number[]}
 */
function asteroidCollision(asteroids) {
  const stack = [];

  for (const ast of asteroids) {
    let alive = true;

    while (
      alive &&
      stack.length > 0 &&
      stack[stack.length - 1] > 0 &&
      ast < 0
    ) {
      const top = stack[stack.length - 1];
      if (top < -ast) {
        stack.pop();
      } else if (top === -ast) {
        stack.pop();
        alive = false;
      } else {
        alive = false;
      }
    }

    if (alive) stack.push(ast);
  }
  return stack;
}
```

### TypeScript

```ts
function asteroidCollision(asteroids: number[]): number[] {
  const stack: number[] = [];

  for (const ast of asteroids) {
    let alive = true;

    while (
      alive &&
      stack.length > 0 &&
      stack[stack.length - 1] > 0 &&
      ast < 0
    ) {
      const top = stack[stack.length - 1];
      if (top < -ast) {
        stack.pop();
      } else if (top === -ast) {
        stack.pop();
        alive = false;
      } else {
        alive = false;
      }
    }

    if (alive) stack.push(ast);
  }
  return stack;
}
```

### Common mistakes

- Not handling equal-size mutual destruction.
- Pushing destroyed asteroid anyway.
- Forgetting that only **positive top + negative current** collides.

---

## 3. Sliding Window Maximum (239)

### Problem (short)

Given array `nums` and window size `k`, return max in each sliding window position.

**Example**

```
Input:  nums = [1,3,-1,-3,5,3,6,7], k = 3
Output: [3,3,5,5,6,7]

Windows: [1,3,-1]→3, [3,-1,-3]→3, [-1,-3,5]→5, ...
```

### Hint 1 — Brute O(n·k)

For each window, scan k elements for max. Too slow.

### Hint 2 — Monotonic deque of indices

Deque stores **indices** of candidates for max, in **decreasing value order** (front = current max index).

For each new index `i`:
1. Remove indices **out of window** from front (`i - k`).
2. Remove from **back** while `nums[back] <= nums[i]` (they'll never be max).
3. Push `i` to back.
4. If `i >= k - 1`, record `nums[deque.front]`.

### Hint 3 — Why deque?

- Front always has index of max in current window.
- Smaller elements behind larger ones are useless — pop them.

### Walkthrough — first few steps, k=3

```
i=0 (1): deque [0]
i=1 (3): pop 0 (1<=3), deque [1]
i=2 (-1): deque [1,2] → window complete → max nums[1]=3
i=3 (-3): pop 2 (-1<=-3)? no; deque [1,3] → max 3
...
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(k) |

Each index added/removed from deque at most once.

### JavaScript

```js
/**
 * @param {number[]} nums
 * @param {number} k
 * @return {number[]}
 */
function maxSlidingWindow(nums, k) {
  const deque = []; // indices
  const result = [];

  for (let i = 0; i < nums.length; i++) {
    // Remove out-of-window indices
    while (deque.length > 0 && deque[0] <= i - k) {
      deque.shift();
    }
    // Remove smaller elements from back
    while (deque.length > 0 && nums[deque[deque.length - 1]] <= nums[i]) {
      deque.pop();
    }
    deque.push(i);

    if (i >= k - 1) {
      result.push(nums[deque[0]]);
    }
  }
  return result;
}
```

### TypeScript

```ts
function maxSlidingWindow(nums: number[], k: number): number[] {
  const deque: number[] = [];
  const result: number[] = [];

  for (let i = 0; i < nums.length; i++) {
    while (deque.length > 0 && deque[0] <= i - k) {
      deque.shift();
    }
    while (deque.length > 0 && nums[deque[deque.length - 1]] <= nums[i]) {
      deque.pop();
    }
    deque.push(i);

    if (i >= k - 1) {
      result.push(nums[deque[0]]);
    }
  }
  return result;
}
```

### Optimization note

Use a **doubly-linked list** or custom deque for O(1) `shift`. In interviews, mention that `Array.shift()` is O(n) in JS but algorithm is still O(n) amortized if you use index pointers instead.

```js
// Index-based deque (no shift cost)
class Deque {
  constructor() { this.items = []; this.head = 0; }
  pushBack(x) { this.items.push(x); }
  popBack() { return this.items.pop(); }
  popFront() {
    if (this.head >= this.items.length) return undefined;
    return this.items[this.head++];
  }
  front() { return this.items[this.head]; }
  get length() { return this.items.length - this.head; }
}
```

### Common mistakes

- Storing values instead of indices (can't evict out-of-window).
- Forgetting to record result only when `i >= k - 1`.
- Using increasing deque (need **decreasing** for max).

---

## 4. Pattern Cheat Sheet

| Problem | Structure | Key insight |
|---------|-----------|-------------|
| **739 Daily Temp** | Monotonic stack (indices) | Pop when current > stack top → next greater element |
| **735 Asteroid** | Stack simulation | Only +/− collision at stack top |
| **239 Sliding Max** | Monotonic deque | Front = max; evict stale + smaller |

### When you see…

| Signal | Think |
|--------|-------|
| Next greater / warmer / smaller element | Monotonic stack |
| Sliding window min/max | Monotonic deque |
| Sequential destroy/collide rules | Stack simulation |

### Monotonic stack template

```js
const stack = [];
for (let i = 0; i < n; i++) {
  while (stack.length && condition(nums[i], nums[stack.at(-1)])) {
    const idx = stack.pop();
    // process idx — found answer for idx
  }
  stack.push(i);
}
```

### Solve order (revision)

1. **739** — Draw stack of indices; pop when warmer found.
2. **735** — Trace collision at stack top only.
3. **239** — Deque front = max; trim back when new element is larger.

---

*End of Day 2 LeetCode*
