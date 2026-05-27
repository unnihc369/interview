# Day 3 — LeetCode (Arrays · Binary Search)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 238 | Product of Array Except Self | Medium | Prefix + Suffix |
| 152 | Maximum Product Subarray | Medium | Kadane Variant |
| 153 | Find Minimum in Rotated Sorted Array | Medium | Binary Search |

---

# 1. Product of Array Except Self (238)

## Problem

Given an integer array `nums`, return an array `answer` such that:

```text
answer[i] = product of all elements except nums[i]
```

Constraints:
- Solve without division
- Time Complexity should be O(n)

---

## Example

```text
Input:  [1,2,3,4]
Output: [24,12,8,6]
```

---

## Intuition

For every index:
- Multiply all elements on left side
- Multiply all elements on right side

```text
result[i] = leftProduct * rightProduct
```

---

## Prefix and Suffix Idea

For:
```text
[1,2,3,4]
```

Prefix:
```text
[1,1,2,6]
```

Suffix traversal gives:
```text
[24,12,8,6]
```

---

## Algorithm

1. Store prefix products in result array
2. Traverse from right side using suffix variable
3. Multiply prefix and suffix

---

## Java Solution

```java
class Solution {

    public int[] productExceptSelf(int[] nums) {

        int n = nums.length;

        int[] result = new int[n];

        result[0] = 1;

        // Prefix product
        for (int i = 1; i < n; i++) {
            result[i] = result[i - 1] * nums[i - 1];
        }

        int suffix = 1;

        // Suffix product
        for (int i = n - 1; i >= 0; i--) {
            result[i] *= suffix;
            suffix *= nums[i];
        }

        return result;
    }
}
```

---

## JavaScript Solution

```javascript
function productExceptSelf(nums) {

    let n = nums.length;

    let result = new Array(n).fill(1);

    for (let i = 1; i < n; i++) {
        result[i] = result[i - 1] * nums[i - 1];
    }

    let suffix = 1;

    for (let i = n - 1; i >= 0; i--) {
        result[i] *= suffix;
        suffix *= nums[i];
    }

    return result;
}
```

---

## Complexity

| Time | Space |
|---|---|
| O(n) | O(1) extra |

---

## Frequently Asked Interview Questions

### Why not use division?
Because array can contain zero.

### Why prefix and suffix?
To avoid repeated multiplication.

### Can this be solved in-place?
Yes using result array + suffix variable.

---

# 2. Maximum Product Subarray (152)

## Problem

Find contiguous subarray having largest product.

---

## Example

```text
Input: [2,3,-2,4]
Output: 6
```

Subarray:
```text
[2,3]
```

---

## Key Observation

Negative numbers can flip:
- minimum → maximum
- maximum → minimum

So maintain:
- currentMax
- currentMin

---

## Java Solution

```java
class Solution {

    public int maxProduct(int[] nums) {

        int currentMax = nums[0];
        int currentMin = nums[0];
        int result = nums[0];

        for (int i = 1; i < nums.length; i++) {

            int temp = currentMax;

            currentMax = Math.max(
                nums[i],
                Math.max(currentMax * nums[i],
                         currentMin * nums[i])
            );

            currentMin = Math.min(
                nums[i],
                Math.min(temp * nums[i],
                         currentMin * nums[i])
            );

            result = Math.max(result, currentMax);
        }

        return result;
    }
}
```

---

## Complexity

| Time | Space |
|---|---|
| O(n) | O(1) |

---

## Interview Questions

### Why track minimum product?
Negative × negative = positive.

### Difference between Kadane and this?
Kadane tracks maximum sum.
This tracks max and min products.

---

# 3. Find Minimum in Rotated Sorted Array (153)

## Problem

Find minimum element in rotated sorted array.

---

## Example

```text
Input: [4,5,6,7,0,1,2]
Output: 0
```

---

## Binary Search Intuition

Minimum always lies in unsorted half.

Compare:
```java
nums[mid] with nums[right]
```

---

## Java Solution

```java
class Solution {

    public int findMin(int[] nums) {

        int left = 0;
        int right = nums.length - 1;

        while (left < right) {

            int mid = left + (right - left) / 2;

            if (nums[mid] > nums[right]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }

        return nums[left];
    }
}
```

---

## Complexity

| Time | Space |
|---|---|
| O(log n) | O(1) |

---

## Interview Questions

### Why compare with nums[right]?
To identify sorted half.

### What if duplicates exist?
Logic changes slightly.

---

# Pattern Cheat Sheet

| Problem | Pattern | Key Idea |
|---|---|---|
| Product Except Self | Prefix + Suffix | Store left/right products |
| Maximum Product Subarray | Kadane Variant | Track min and max |
| Rotated Sorted Array | Binary Search | Minimum in unsorted half |

---

*End of Day 3 LeetCode*