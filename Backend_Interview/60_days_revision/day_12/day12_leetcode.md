# Day 12 — Stack & Monotonic Stack

---

# 1. Trapping Rain Water

## Problem Statement

Given elevation map `height[]` (non-negative), compute how much water can be trapped after raining.

---

# Example

```text
Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

---

# Approach Hints

- Brute: for each index, water += min(maxLeft, maxRight) - height[i] — O(n²).
- Prefix max arrays left[] and right[] — O(n) time O(n) space.
- Two pointers from both ends — O(n) time O(1) space.
- Monotonic stack — alternative O(n).

---

# Core Idea (Two Pointers)

1. `left` and `right` at ends; track `leftMax` and `rightMax`.
2. Move pointer with smaller max; water at that index = max - height.

---

# Java Solution

```java
class Solution {
    public int trap(int[] height) {
        int left = 0, right = height.length - 1;
        int leftMax = 0, rightMax = 0, water = 0;

        while (left < right) {
            if (height[left] < height[right]) {
                if (height[left] >= leftMax) leftMax = height[left];
                else water += leftMax - height[left];
                left++;
            } else {
                if (height[right] >= rightMax) rightMax = height[right];
                else water += rightMax - height[right];
                right--;
            }
        }
        return water;
    }
}
```

---

# JavaScript Solution

```javascript
function trap(height) {
  let left = 0, right = height.length - 1;
  let leftMax = 0, rightMax = 0, water = 0;

  while (left < right) {
    if (height[left] < height[right]) {
      if (height[left] >= leftMax) leftMax = height[left];
      else water += leftMax - height[left];
      left++;
    } else {
      if (height[right] >= rightMax) rightMax = height[right];
      else water += rightMax - height[right];
      right--;
    }
  }
  return water;
}
```

---

# Time & Space Complexity

```text
Time:  O(n)
Space: O(1)
```

---

# 2. Largest Rectangle in Histogram

## Problem Statement

Given array `heights` representing histogram bar heights, return area of largest rectangle in histogram.

---

# Example

```text
Input: [2,1,5,6,2,3]
Output: 10  (bars 5 and 6, width 2, height 5)
```

---

# Approach Hints

- Brute O(n²) — fix each bar as min height, expand width.
- Monotonic increasing stack of indices — when pop, calculate area with popped height as smallest.

---

# Core Idea (Monotonic Stack)

1. Push indices with increasing heights.
2. When current height < stack top, pop and compute area: `height[popped] * width`.
3. Width extends to current index and previous stack top.

---

# Java Solution

```java
class Solution {
    public int largestRectangleArea(int[] heights) {
        Deque<Integer> stack = new ArrayDeque<>();
        int max = 0;

        for (int i = 0; i <= heights.length; i++) {
            int h = (i == heights.length) ? 0 : heights[i];
            while (!stack.isEmpty() && h < heights[stack.peek()]) {
                int height = heights[stack.pop()];
                int width = stack.isEmpty() ? i : i - stack.peek() - 1;
                max = Math.max(max, height * width);
            }
            stack.push(i);
        }
        return max;
    }
}
```

---

# JavaScript Solution

```javascript
function largestRectangleArea(heights) {
  const stack = [];
  let max = 0;

  for (let i = 0; i <= heights.length; i++) {
    const h = i === heights.length ? 0 : heights[i];
    while (stack.length && h < heights[stack[stack.length - 1]]) {
      const height = heights[stack.pop()];
      const width = stack.length ? i - stack[stack.length - 1] - 1 : i;
      max = Math.max(max, height * width);
    }
    stack.push(i);
  }
  return max;
}
```

---

# Time & Space Complexity

```text
Time:  O(n)
Space: O(n)
```

---

# 3. Maximal Rectangle

## Problem Statement

Given binary matrix `matrix` of `'0'` and `'1'`, find largest rectangle containing only `'1'`s.

---

# Example

```text
Input:
[["1","0","1","0","0"],
 ["1","0","1","1","1"],
 ["1","1","1","1","1"],
 ["1","0","0","1","0"]]
Output: 6
```

---

# Approach Hints

- For each row, treat as histogram base heights (accumulate consecutive 1s per column).
- Run **Largest Rectangle in Histogram** on each row — O(rows × cols).

---

# Core Idea (Walkthrough)

1. `heights[j]` = number of consecutive 1s ending at current row in column j.
2. After updating heights for row i, call `largestRectangleArea(heights)`.
3. Track global max.

---

# Java Solution

```java
class Solution {
    public int maximalRectangle(char[][] matrix) {
        if (matrix.length == 0) return 0;
        int cols = matrix[0].length;
        int[] heights = new int[cols];
        int max = 0;

        for (char[] row : matrix) {
            for (int j = 0; j < cols; j++) {
                heights[j] = row[j] == '1' ? heights[j] + 1 : 0;
            }
            max = Math.max(max, largestRectangleArea(heights));
        }
        return max;
    }

    private int largestRectangleArea(int[] heights) {
        Deque<Integer> stack = new ArrayDeque<>();
        int max = 0;
        for (int i = 0; i <= heights.length; i++) {
            int h = (i == heights.length) ? 0 : heights[i];
            while (!stack.isEmpty() && h < heights[stack.peek()]) {
                int height = heights[stack.pop()];
                int width = stack.isEmpty() ? i : i - stack.peek() - 1;
                max = Math.max(max, height * width);
            }
            stack.push(i);
        }
        return max;
    }
}
```

---

# JavaScript Solution

```javascript
function maximalRectangle(matrix) {
  if (!matrix.length) return 0;
  const cols = matrix[0].length;
  const heights = Array(cols).fill(0);
  let max = 0;

  for (const row of matrix) {
    for (let j = 0; j < cols; j++) {
      heights[j] = row[j] === "1" ? heights[j] + 1 : 0;
    }
    max = Math.max(max, largestRectangleArea(heights));
  }
  return max;
}

function largestRectangleArea(heights) {
  const stack = [];
  let max = 0;
  for (let i = 0; i <= heights.length; i++) {
    const h = i === heights.length ? 0 : heights[i];
    while (stack.length && h < heights[stack[stack.length - 1]]) {
      const height = heights[stack.pop()];
      const width = stack.length ? i - stack[stack.length - 1] - 1 : i;
      max = Math.max(max, height * width);
    }
    stack.push(i);
  }
  return max;
}
```

---

# Time & Space Complexity

```text
Time:  O(rows * cols)
Space: O(cols)
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Trapping Rain Water | Two pointers / prefix max | O(n) |
| Largest Rectangle in Histogram | Monotonic stack | O(n) |
| Maximal Rectangle | Histogram per row | O(m×n) |

---

*End of Day 12 LeetCode*
