# Day 30 - Backtracking: Subsets & Combinations

---

# 1. Permutations

## Problem Statement

Given an array `nums` of distinct integers, return all possible permutations.

---

# Example

```text
Input: [1,2,3]
Output: [[1,2,3],[1,3,2],[2,1,3],[2,3,1],[3,1,2],[3,2,1]]
```

---

# Core Idea

Swap-based backtracking or use `used[]` array. At each level, pick unused element.

---

# Java Solution

```java
class Solution {
    public List<List<Integer>> permute(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, new ArrayList<>(), new boolean[nums.length], result);
        return result;
    }

    private void backtrack(int[] nums, List<Integer> path,
                           boolean[] used, List<List<Integer>> result) {
        if (path.size() == nums.length) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i = 0; i < nums.length; i++) {
            if (used[i]) continue;
            used[i] = true;
            path.add(nums[i]);
            backtrack(nums, path, used, result);
            path.remove(path.size() - 1);
            used[i] = false;
        }
    }
}
```

---

# JavaScript Solution

```javascript
function permute(nums) {
  const result = [];
  const used = new Array(nums.length).fill(false);

  function backtrack(path) {
    if (path.length === nums.length) {
      result.push([...path]);
      return;
    }
    for (let i = 0; i < nums.length; i++) {
      if (used[i]) continue;
      used[i] = true;
      path.push(nums[i]);
      backtrack(path);
      path.pop();
      used[i] = false;
    }
  }

  backtrack([]);
  return result;
}
```

---

# Time Complexity

```text
O(n! * n)
Space: O(n)
```

---

# 2. Subsets

## Problem Statement

Given an integer array `nums` of unique elements, return all possible subsets (power set).

---

# Example

```text
Input: [1,2,3]
Output: [[],[1],[2],[1,2],[3],[1,3],[2,3],[1,2,3]]
```

---

# Core Idea

For each element: **include** or **exclude** (or iterate start index to avoid duplicates in combination problems).

---

# Java Solution

```java
class Solution {
    public List<List<Integer>> subsets(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        backtrack(nums, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] nums, int start, List<Integer> path,
                           List<List<Integer>> result) {
        result.add(new ArrayList<>(path));
        for (int i = start; i < nums.length; i++) {
            path.add(nums[i]);
            backtrack(nums, i + 1, path, result);
            path.remove(path.size() - 1);
        }
    }
}
```

---

# JavaScript Solution

```javascript
function subsets(nums) {
  const result = [];

  function backtrack(start, path) {
    result.push([...path]);
    for (let i = start; i < nums.length; i++) {
      path.push(nums[i]);
      backtrack(i + 1, path);
      path.pop();
    }
  }

  backtrack(0, []);
  return result;
}
```

---

# Time Complexity

```text
O(2^n * n)
```

---

# 3. Combination Sum

## Problem Statement

Given distinct integers `candidates` and target `target`, return all unique combinations where chosen numbers sum to `target`. Same number may be used unlimited times.

---

# Example

```text
Input: candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]
```

---

# Core Idea

Sort candidates. Backtrack from `start` index; allow reuse by passing `i` (not `i+1`) when including same element.

---

# Java Solution

```java
class Solution {
    public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> result = new ArrayList<>();
        backtrack(candidates, target, 0, new ArrayList<>(), result);
        return result;
    }

    private void backtrack(int[] candidates, int remain, int start,
                           List<Integer> path, List<List<Integer>> result) {
        if (remain == 0) {
            result.add(new ArrayList<>(path));
            return;
        }
        for (int i = start; i < candidates.length; i++) {
            if (candidates[i] > remain) break; // pruning
            path.add(candidates[i]);
            backtrack(candidates, remain - candidates[i], i, path, result);
            path.remove(path.size() - 1);
        }
    }
}
```

---

# JavaScript Solution

```javascript
function combinationSum(candidates, target) {
  candidates.sort((a, b) => a - b);
  const result = [];

  function backtrack(start, remain, path) {
    if (remain === 0) {
      result.push([...path]);
      return;
    }
    for (let i = start; i < candidates.length; i++) {
      if (candidates[i] > remain) break;
      path.push(candidates[i]);
      backtrack(i, remain - candidates[i], path);
      path.pop();
    }
  }

  backtrack(0, target, []);
  return result;
}
```

---

# Time Complexity

```text
O(2^target) worst case (branching on target with 1s)
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Permutations | used[] + path building | O(n!·n) |
| Subsets | start index, include/exclude tree | O(2^n·n) |
| Combination Sum | start index + reuse (stay at i) | exponential |

---

# Interview Questions

## Permutations II (with duplicates)?

Sort + skip same value at same level: `if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue`.

## Subsets vs Subsets II?

Subsets II: skip duplicate branches when `i > start && nums[i] == nums[i-1]`.

## Combination Sum II (each number once)?

Pass `i + 1` after pick; skip duplicates at same recursion level.

## Permutations: swap-based alternative?

```java
void backtrack(int[] nums, int first) {
    if (first == nums.length) { add copy; return; }
    for (int i = first; i < nums.length; i++) {
        swap(nums, first, i);
        backtrack(nums, first + 1);
        swap(nums, first, i);
    }
}
```

---

*End of Day 30 LeetCode*
