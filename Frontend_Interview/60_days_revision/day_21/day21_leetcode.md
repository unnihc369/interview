# Day 21 — LeetCode (Timed Mock: Two Pointers)

**Format:** Timed practice — 3 problems · **45 min total** · No peeking at solutions until attempt complete

| # | Problem | Difficulty | Target time |
|---|---------|------------|-------------|
| 11 | [Container With Most Water](#mock-1--container-with-most-water-11) | Medium | 15 min |
| 15 | [3Sum](#mock-2--3sum-15) | Medium | 15 min |
| 42 | [Trapping Rain Water](#mock-3--trapping-rain-water-42) | Hard | 15 min |

---

## Table of Contents

1. [Mock Rules & Scoring](#1-mock-rules--scoring)
2. [Mock 1 — Container With Most Water (11)](#mock-1--container-with-most-water-11)
3. [Mock 2 — 3Sum (15)](#mock-2--3sum-15)
4. [Mock 3 — Trapping Rain Water (42)](#mock-3--trapping-rain-water-42)
5. [Post-Mock Review Checklist](#5-post-mock-review-checklist)
6. [Solutions (Reveal After Attempt)](#6-solutions-reveal-after-attempt)
7. [Pattern Cheat Sheet](#7-pattern-cheat-sheet)

---

## 1. Mock Rules & Scoring

### Before you start

1. Set timer for **45 minutes**.
2. Open a blank editor — JavaScript only.
3. For each problem: restate in your own words → examples → brute force → optimize → code → test.

### Scoring rubric

| Criteria | Points |
|----------|--------|
| Correct optimal approach | 3 |
| Working bug-free code | 3 |
| Stated time/space complexity | 2 |
| Handled edge cases | 2 |
| **Per problem max** | **10** |

| Total | Grade |
|-------|-------|
| 25–30 | Strong — ready for interview |
| 18–24 | Good — review hints below |
| < 18 | Re-study Day 15–20 concepts |

---

## Mock 1 — Container With Most Water (11)

### Problem (short)

`height[i]` vertical lines. Max water between two lines.

```
Input: [1,8,6,2,5,4,8,3,7]
Output: 49
```

### Self-check hints (use only if stuck 5+ min)

<details>
<summary>Hint 1</summary>
Two pointers at both ends.
</details>

<details>
<summary>Hint 2</summary>
Always move the **shorter** line inward.
</details>

### Your workspace

```js
/**
 * @param {number[]} height
 * @return {number}
 */
function maxArea(height) {
  // YOUR CODE
}
```

### Test cases

```js
console.assert(maxArea([1,8,6,2,5,4,8,3,7]) === 49);
console.assert(maxArea([1,1]) === 1);
console.assert(maxArea([4,3,2,1,4]) === 16);
```

---

## Mock 2 — 3Sum (15)

### Problem (short)

All unique triplets summing to 0.

```
Input: [-1,0,1,2,-1,-4]
Output: [[-1,-1,2],[-1,0,1]]
```

### Self-check hints

<details>
<summary>Hint 1</summary>
Sort the array first.
</details>

<details>
<summary>Hint 2</summary>
Fix index `i`, two-pointer `l` and `r` on remainder. Skip duplicates at i, l, r.
</details>

### Your workspace

```js
/**
 * @param {number[]} nums
 * @return {number[][]}
 */
function threeSum(nums) {
  // YOUR CODE
}
```

### Test cases

```js
const r1 = threeSum([-1,0,1,2,-1,-4]);
console.assert(r1.length === 2);
console.assert(threeSum([0,0,0]).length === 1);
console.assert(threeSum([]).length === 0);
```

---

## Mock 3 — Trapping Rain Water (42)

### Problem (short)

Elevation map — total trapped water?

```
Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

### Self-check hints

<details>
<summary>Hint 1</summary>
Two pointers with `leftMax` and `rightMax`.
</details>

<details>
<summary>Hint 2</summary>
Process the side with **smaller** max height.
</details>

### Your workspace

```js
/**
 * @param {number[]} height
 * @return {number}
 */
function trap(height) {
  // YOUR CODE
}
```

### Test cases

```js
console.assert(trap([0,1,0,2,1,0,1,3,2,1,2,1]) === 6);
console.assert(trap([4,2,0,3,2,5]) === 9);
console.assert(trap([1,2,3]) === 0);
```

---

## 5. Post-Mock Review Checklist

After timer stops, for each problem ask:

- [ ] Did I identify the two-pointer pattern within 2 minutes?
- [ ] Did I write the move rule before coding?
- [ ] Did I test empty / single / duplicate inputs?
- [ ] Could I explain **why** the greedy move is safe (11, 42)?

---

## 6. Solutions (Reveal After Attempt)

### 11 — Container With Most Water

```js
function maxArea(height) {
  let l = 0, r = height.length - 1, best = 0;
  while (l < r) {
    const h = Math.min(height[l], height[r]);
    best = Math.max(best, h * (r - l));
    if (height[l] < height[r]) l++;
    else r--;
  }
  return best;
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) |

### 15 — 3Sum

```js
function threeSum(nums) {
  nums.sort((a, b) => a - b);
  const res = [];
  for (let i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue;
    let l = i + 1, r = nums.length - 1;
    while (l < r) {
      const sum = nums[i] + nums[l] + nums[r];
      if (sum === 0) {
        res.push([nums[i], nums[l], nums[r]]);
        while (l < r && nums[l] === nums[l + 1]) l++;
        while (l < r && nums[r] === nums[r - 1]) r--;
        l++; r--;
      } else if (sum < 0) l++;
      else r--;
    }
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(n²) | O(1) extra |

### 42 — Trapping Rain Water

```js
function trap(height) {
  let l = 0, r = height.length - 1;
  let leftMax = 0, rightMax = 0, water = 0;
  while (l < r) {
    if (height[l] < height[r]) {
      leftMax = Math.max(leftMax, height[l]);
      water += leftMax - height[l];
      l++;
    } else {
      rightMax = Math.max(rightMax, height[r]);
      water += rightMax - height[r];
      r--;
    }
  }
  return water;
}
```

| Time | Space |
|------|-------|
| O(n) | O(1) |

---

## 7. Pattern Cheat Sheet

| Problem | Pattern | Key insight |
|---------|---------|-------------|
| **11** | Opposite ends | Move shorter side |
| **15** | Sort + fix + L/R | Skip duplicates |
| **42** | Opposite + running max | Add water at smaller-max side |

---

*End of Day 21 LeetCode mock*
