# Day 4 - Arrays & Interview DSA Patterns

---

# 1. 3Sum

## Problem Statement

Given an integer array `nums`, return all unique triplets `[a, b, c]` such that:

```text
a + b + c = 0
```

Triplets should be unique (no duplicates).

---

# Example

```text
Input:
[-1,0,1,2,-1,-4]

Output:
[[-1,-1,2],[-1,0,1]]
```

---

# Brute Force Approach

Try all combinations of 3 indices.

```text
O(n^3)
```

Too slow for interview constraints.

---

# Optimized Approach

## Core Idea

1. Sort array
2. Fix one element `i`
3. Use two pointers (`left`, `right`) to find pairs summing to `-nums[i]`
4. Skip duplicates for `i`, `left`, `right`

---

# Java Solution

```java
class Solution {
    public List<List<Integer>> threeSum(int[] nums) {
        Arrays.sort(nums);
        List<List<Integer>> result = new ArrayList<>();

        for (int i = 0; i < nums.length - 2; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;

            int left = i + 1;
            int right = nums.length - 1;

            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];

                if (sum == 0) {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    left++;
                    right--;

                    while (left < right && nums[left] == nums[left - 1]) left++;
                    while (left < right && nums[right] == nums[right + 1]) right--;
                } else if (sum < 0) {
                    left++;
                } else {
                    right--;
                }
            }
        }

        return result;
    }
}
```

---

# JavaScript Solution

```javascript
function threeSum(nums) {
  nums.sort((a, b) => a - b);
  const result = [];

  for (let i = 0; i < nums.length - 2; i++) {
    if (i > 0 && nums[i] === nums[i - 1]) continue;

    let left = i + 1;
    let right = nums.length - 1;

    while (left < right) {
      const sum = nums[i] + nums[left] + nums[right];

      if (sum === 0) {
        result.push([nums[i], nums[left], nums[right]]);
        left++;
        right--;

        while (left < right && nums[left] === nums[left - 1]) left++;
        while (left < right && nums[right] === nums[right + 1]) right--;
      } else if (sum < 0) {
        left++;
      } else {
        right--;
      }
    }
  }

  return result;
}
```

---

# Time Complexity

```text
O(n^2)
```

---

# 2. Container With Most Water

## Problem Statement

Given an array `height`, each value represents vertical line height.
Find two lines that together form the container holding maximum water.

```text
Area = min(height[i], height[j]) * (j - i)
```

---

# Example

```text
Input:
[1,8,6,2,5,4,8,3,7]

Output:
49
```

---

# Brute Force

Try all pairs `(i, j)`.

```text
O(n^2)
```

---

# Optimized Approach (Two Pointers)

## Core Idea

- Start with widest container: `left = 0`, `right = n-1`
- Compute area
- Move the pointer having smaller height (only chance to increase min height)

---

# Java Solution

```java
class Solution {
    public int maxArea(int[] height) {
        int left = 0;
        int right = height.length - 1;
        int max = 0;

        while (left < right) {
            int h = Math.min(height[left], height[right]);
            int w = right - left;
            max = Math.max(max, h * w);

            if (height[left] < height[right]) left++;
            else right--;
        }

        return max;
    }
}
```

---

# JavaScript Solution

```javascript
function maxArea(height) {
  let left = 0;
  let right = height.length - 1;
  let max = 0;

  while (left < right) {
    const h = Math.min(height[left], height[right]);
    const w = right - left;
    max = Math.max(max, h * w);

    if (height[left] < height[right]) left++;
    else right--;
  }

  return max;
}
```

---

# Time Complexity

```text
O(n)
```

---

# 3. Search in Rotated Sorted Array

## Problem Statement

Array is sorted, then rotated at some pivot.
Find index of `target` in `O(log n)`.

If not found, return `-1`.

---

# Example

```text
Input:
nums = [4,5,6,7,0,1,2], target = 0

Output:
4
```

---

# Core Idea

Modified Binary Search:

At each step, one half is always sorted.

- If left half sorted, check if target lies there.
- Else right half sorted, check there.

---

# Java Solution

```java
class Solution {
    public int search(int[] nums, int target) {
        int left = 0, right = nums.length - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) return mid;

            if (nums[left] <= nums[mid]) {
                // Left half sorted
                if (target >= nums[left] && target < nums[mid]) {
                    right = mid - 1;
                } else {
                    left = mid + 1;
                }
            } else {
                // Right half sorted
                if (target > nums[mid] && target <= nums[right]) {
                    left = mid + 1;
                } else {
                    right = mid - 1;
                }
            }
        }

        return -1;
    }
}
```

---

# JavaScript Solution

```javascript
function search(nums, target) {
  let left = 0;
  let right = nums.length - 1;

  while (left <= right) {
    const mid = Math.floor((left + right) / 2);

    if (nums[mid] === target) return mid;

    if (nums[left] <= nums[mid]) {
      if (target >= nums[left] && target < nums[mid]) {
        right = mid - 1;
      } else {
        left = mid + 1;
      }
    } else {
      if (target > nums[mid] && target <= nums[right]) {
        left = mid + 1;
      } else {
        right = mid - 1;
      }
    }
  }

  return -1;
}
```

---

# Time Complexity

```text
O(log n)
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| 3Sum | Sort + Two Pointers | O(n²) |
| Container With Most Water | Two Pointers | O(n) |
| Search in Rotated Sorted Array | Modified Binary Search | O(log n) |

---

*End of Day 4 LeetCode*
