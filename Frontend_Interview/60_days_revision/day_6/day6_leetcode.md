# Day 6 — LeetCode Saturday Revision

| # | Problem | Difficulty | Pattern | Status |
|---|---------|------------|---------|--------|
| 84 | [Largest Rectangle in Histogram](#1-largest-rectangle-in-histogram-84) | Hard | Monotonic Stack | ⬜ Not done |
| 316 | [Remove Duplicate Letters](#2-remove-duplicate-letters-316) | Medium | Monotonic Stack + Greedy | ⬜ Not done |
| 42 | [Trapping Rain Water](#3-trapping-rain-water-42) | Hard | Two Pointer / Stack | ⬜ Not done |

**Target:** 2 Medium + 1 Hard — complete all three today.

---

## Table of Contents

1. [Largest Rectangle in Histogram (84)](#1-largest-rectangle-in-histogram-84)
2. [Remove Duplicate Letters (316)](#2-remove-duplicate-letters-316)
3. [Trapping Rain Water (42)](#3-trapping-rain-water-42)
4. [Saturday Revision Plan](#4-saturday-revision-plan)
5. [Pattern Cheat Sheet](#5-pattern-cheat-sheet)

---

## 1. Largest Rectangle in Histogram (84)

### Problem (short)

Given array `heights` representing histogram bar heights, return area of largest rectangle.

**Example**

```
Input:  heights = [2, 1, 5, 6, 2, 3]
Output: 10  (bars 5 and 6, width 2)

Input:  heights = [2, 4]
Output: 4
```

### Hint 1 — Brute O(n²) too slow

For each bar, expand left and right while height ≥ current. Need O(n).

### Hint 2 — Monotonic increasing stack of indices

When current height < stack top → popped bar's **right boundary** is current index.

```
area = heights[popped] * (i - stackTop - 1)
// if stack empty: width = i
```

### Hint 3 — Sentinel at end

Process height `0` at end to flush remaining stack.

### Walkthrough — `[2, 1, 5, 6, 2, 3]`

```
i=0 h=2: stack [0]
i=1 h=1: 1<2 pop 0 area=2*1=2; push 1 → [1]
i=2 h=5: push 2 → [1,2]
i=3 h=6: push 3 → [1,2,3]
i=4 h=2: pop 3 area=6*1=6; pop 2 area=5*2=10; push 4
i=5 h=3: push 5
i=6 h=0: flush all...
max = 10 ✓
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(n) |

### JavaScript

```js
/**
 * @param {number[]} heights
 * @return {number}
 */
function largestRectangleArea(heights) {
  const stack = [];
  let maxArea = 0;

  for (let i = 0; i <= heights.length; i++) {
    const h = i === heights.length ? 0 : heights[i];
    while (stack.length > 0 && heights[stack[stack.length - 1]] > h) {
      const idx = stack.pop();
      const height = heights[idx];
      const width = stack.length === 0 ? i : i - stack[stack.length - 1] - 1;
      maxArea = Math.max(maxArea, height * width);
    }
    stack.push(i);
  }
  return maxArea;
}
```

### TypeScript

```ts
function largestRectangleArea(heights: number[]): number {
  const stack: number[] = [];
  let maxArea = 0;

  for (let i = 0; i <= heights.length; i++) {
    const h = i === heights.length ? 0 : heights[i];
    while (stack.length > 0 && heights[stack[stack.length - 1]] > h) {
      const idx = stack.pop()!;
      const height = heights[idx];
      const width = stack.length === 0 ? i : i - stack[stack.length - 1] - 1;
      maxArea = Math.max(maxArea, height * width);
    }
    stack.push(i);
  }
  return maxArea;
}
```

---

## 2. Remove Duplicate Letters (316)

### Problem (short)

Remove duplicate letters so each letter appears once. Result must be **smallest in lexicographic order**.

**Example**

```
Input:  "bcabc"  → Output: "abc"
Input:  "cbacdcbc" → Output: "acdb"
```

### Hint 1 — Greedy + monotonic stack

Similar to "Remove K Digits" (402) but must keep at least one of each char.

### Hint 2 — Track remaining counts

Only pop stack top if:
- stack.top > current char (can improve lex order)
- stack.top appears **later** in string (can remove safely)

### Hint 3 — Algorithm

```
lastIndex[char] = last position
remaining[char] = count
stack = []
visited = set of chars in stack

for each char c:
  remaining[c]--
  if c in visited: continue
  while stack not empty AND stack.top > c AND remaining[stack.top] > 0:
    visited.remove(stack.top)
    stack.pop()
  stack.push(c)
  visited.add(c)

return stack.join('')
```

### Walkthrough — `"cbacdcbc"`

```
Process c,b,a,c,d,c,b,c → stack builds to "acdb"
```

### Complexity

| Time | Space |
|------|-------|
| O(n) | O(1) — alphabet size 26 |

### JavaScript

```js
/**
 * @param {string} s
 * @return {string}
 */
function removeDuplicateLetters(s) {
  const lastIndex = {};
  const remaining = {};
  const stack = [];
  const inStack = new Set();

  for (let i = 0; i < s.length; i++) {
    const c = s[i];
    lastIndex[c] = i;
    remaining[c] = (remaining[c] || 0) + 1;
  }

  for (let i = 0; i < s.length; i++) {
    const c = s[i];
    remaining[c]--;

    if (inStack.has(c)) continue;

    while (
      stack.length > 0 &&
      stack[stack.length - 1] > c &&
      remaining[stack[stack.length - 1]] > 0
    ) {
      inStack.delete(stack.pop());
    }

    stack.push(c);
    inStack.add(c);
  }

  return stack.join("");
}
```

### TypeScript

```ts
function removeDuplicateLetters(s: string): string {
  const lastIndex: Record<string, number> = {};
  const remaining: Record<string, number> = {};
  const stack: string[] = [];
  const inStack = new Set<string>();

  for (let i = 0; i < s.length; i++) {
    const c = s[i];
    lastIndex[c] = i;
    remaining[c] = (remaining[c] || 0) + 1;
  }

  for (let i = 0; i < s.length; i++) {
    const c = s[i];
    remaining[c]--;

    if (inStack.has(c)) continue;

    while (
      stack.length > 0 &&
      stack[stack.length - 1] > c &&
      remaining[stack[stack.length - 1]] > 0
    ) {
      inStack.delete(stack.pop()!);
    }

    stack.push(c);
    inStack.add(c);
  }

  return stack.join("");
}
```

---

## 3. Trapping Rain Water (42)

### Problem (short)

Given elevation map (non-negative integers), compute total trapped rain water.

**Example**

```
Input:  height = [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6

       █
   █~~~█~█~~█
 █~█~~~█~█~~█~█
```

### Hint 1 — Two pointer O(n) O(1)

```
left = 0, right = n-1
leftMax, rightMax
while left < right:
  if height[left] <= height[right]:
    water += max(0, leftMax - height[left])
    update leftMax, left++
  else:
    water += max(0, rightMax - height[right])
    update rightMax, right--
```

### Hint 2 — Why it works

Process shorter side — water level determined by **min of max left and max right**.

### Hint 3 — Stack approach O(n)

Monotonic decreasing stack of indices; when current > stack top → trap water between.

### Two-pointer JavaScript

```js
/**
 * @param {number[]} height
 * @return {number}
 */
function trap(height) {
  let left = 0;
  let right = height.length - 1;
  let leftMax = 0;
  let rightMax = 0;
  let water = 0;

  while (left < right) {
    if (height[left] <= height[right]) {
      leftMax = Math.max(leftMax, height[left]);
      water += leftMax - height[left];
      left++;
    } else {
      rightMax = Math.max(rightMax, height[right]);
      water += rightMax - height[right];
      right--;
    }
  }
  return water;
}
```

### TypeScript

```ts
function trap(height: number[]): number {
  let left = 0;
  let right = height.length - 1;
  let leftMax = 0;
  let rightMax = 0;
  let water = 0;

  while (left < right) {
    if (height[left] <= height[right]) {
      leftMax = Math.max(leftMax, height[left]);
      water += leftMax - height[left];
      left++;
    } else {
      rightMax = Math.max(rightMax, height[right]);
      water += rightMax - height[right];
      right--;
    }
  }
  return water;
}
```

### Complexity (two-pointer)

| Time | Space |
|------|-------|
| O(n) | O(1) |

---

## 4. Saturday Revision Plan

### Suggested order (2M + 1H)

| Order | Problem | Time budget | Focus |
|-------|---------|-------------|-------|
| 1 | **316** Remove Duplicate Letters | 25 min | Warm-up; stack + greedy |
| 2 | **42** Trapping Rain Water | 35 min | Two-pointer technique |
| 3 | **84** Largest Rectangle | 45 min | Hard capstone; connects to 85 |

### Self-check after each

- [ ] Can explain approach in 2 minutes without code
- [ ] Implemented without looking at solution
- [ ] Stated time/space complexity
- [ ] Identified edge cases (empty, single element)

### Connection to week problems

| Today | Related earlier |
|-------|-----------------|
| 84 | Day 3 problem 85 (max rectangle in matrix) |
| 316 | Day 3 problem 402 (remove k digits) |
| 42 | Day 5 problem 407 (2D rain water) |

---

## 5. Pattern Cheat Sheet

| Problem | Pattern | One-liner |
|---------|---------|-----------|
| **84 Histogram** | Monotonic stack + sentinel 0 | Pop when shorter; area = h × width |
| **316 Remove Dup** | Stack + remaining counts | Pop if top > c and top appears later |
| **42 Rain Water** | Two pointers from ends | Process shorter side; add max - height |

### Week 1 stack patterns summary

```
Next greater/warmer     → 739
Collision simulation    → 735
Sliding window max      → 239
Remove for optimal      → 402, 316
Histogram rectangle     → 84, 85
Validate sequence       → 946
Path simplify           → 71
Calculator              → 224
```

---

*End of Day 6 LeetCode — mark ⬜ → ✅ when complete*
