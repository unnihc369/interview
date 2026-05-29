# Day 22 - Binary Search & Intervals

---

# 1. Median of Two Sorted Arrays

## Problem Statement

Given two sorted arrays `nums1` and `nums2` of size `m` and `n`, return the median of the two sorted arrays in **O(log(m+n))** time.

---

# Example

```text
Input: nums1 = [1,3], nums2 = [2]
Output: 2.0

Input: nums1 = [1,2], nums2 = [3,4]
Output: 2.5
```

---

# Brute Force

Merge both arrays and pick middle element.

```text
Time: O(m + n)
Space: O(m + n)
```

Not acceptable for interview follow-up.

---

# Optimized Approach — Binary Search on Partition

## Core Idea

We are not searching for a value — we are searching for a **correct partition** in both arrays such that:

- Left half has `(m + n + 1) / 2` elements
- All elements in left half ≤ all elements in right half

```text
nums1: [ ... leftA | rightA ... ]
nums2: [ ... leftB | rightB ... ]
```

If partition is valid:

- Odd total length → max(leftA, leftB)
- Even total length → (max(leftA, leftB) + min(rightA, rightB)) / 2

---

# Java Solution

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if (nums1.length > nums2.length) {
            return findMedianSortedArrays(nums2, nums1);
        }

        int m = nums1.length, n = nums2.length;
        int left = 0, right = m;
        int half = (m + n + 1) / 2;

        while (left <= right) {
            int i = (left + right) / 2; // partition in nums1
            int j = half - i;           // partition in nums2

            int maxLeftA  = (i == 0) ? Integer.MIN_VALUE : nums1[i - 1];
            int minRightA = (i == m) ? Integer.MAX_VALUE : nums1[i];
            int maxLeftB  = (j == 0) ? Integer.MIN_VALUE : nums2[j - 1];
            int minRightB = (j == n) ? Integer.MAX_VALUE : nums2[j];

            if (maxLeftA <= minRightB && maxLeftB <= minRightA) {
                if ((m + n) % 2 == 1) {
                    return Math.max(maxLeftA, maxLeftB);
                }
                return (Math.max(maxLeftA, maxLeftB) + Math.min(minRightA, minRightB)) / 2.0;
            } else if (maxLeftA > minRightB) {
                right = i - 1;
            } else {
                left = i + 1;
            }
        }
        throw new IllegalArgumentException("Input arrays not sorted");
    }
}
```

---

# Time & Space Complexity

```text
Time:  O(log(min(m, n)))
Space: O(1)
```

---

# Interview Tips

- Always binary search on **smaller array**
- Handle edge cases when partition is at boundary (`i == 0`, `i == m`)
- Explain why partition logic works (left half always smaller than right half)

---

# 2. Merge Intervals

## Problem Statement

Given an array of intervals where `intervals[i] = [start, end]`, merge all overlapping intervals and return the non-overlapping result.

---

# Example

```text
Input:  [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
```

---

# Core Idea

1. Sort by start time
2. Iterate and merge if current overlaps with last merged interval
3. Overlap condition: `current.start <= last.end`

---

# Java Solution

```java
class Solution {
    public int[][] merge(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
        List<int[]> merged = new ArrayList<>();
        int[] current = intervals[0];

        for (int i = 1; i < intervals.length; i++) {
            if (current[1] >= intervals[i][0]) {
                current[1] = Math.max(current[1], intervals[i][1]);
            } else {
                merged.add(current);
                current = intervals[i];
            }
        }
        merged.add(current);
        return merged.toArray(new int[merged.size()][]);
    }
}
```

---

# Time Complexity

```text
O(n log n) — sorting dominates
Space: O(n) for result list
```

---

# 3. Insert Interval

## Problem Statement

Given a sorted list of non-overlapping intervals and a new interval, insert the new interval and merge if necessary.

---

# Example

```text
Input:  intervals = [[1,3],[6,9]], newInterval = [2,5]
Output: [[1,5],[6,9]]
```

---

# Core Idea

Three phases in one pass:

1. Add all intervals ending before new interval starts
2. Merge all overlapping intervals with new interval
3. Add remaining intervals

---

# Java Solution

```java
class Solution {
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> result = new ArrayList<>();
        int i = 0, n = intervals.length;

        while (i < n && intervals[i][1] < newInterval[0]) {
            result.add(intervals[i++]);
        }

        while (i < n && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        result.add(newInterval);

        while (i < n) {
            result.add(intervals[i++]);
        }

        return result.toArray(new int[result.size()][]);
    }
}
```

---

# Alternative — Reuse Merge Intervals

Add new interval to list, then call merge logic. Same complexity, cleaner if merge is already known.

---

# Time Complexity

```text
O(n)
Space: O(n)
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Median Two Sorted Arrays | Binary Search on Partition | O(log(min(m,n))) |
| Merge Intervals | Sort + Linear Merge | O(n log n) |
| Insert Interval | Three-Phase Linear Scan | O(n) |

---

# Common Interview Questions

## Q1. Why binary search on smaller array for median?

Partition size in smaller array is bounded; reduces search space.

## Q2. How to detect interval overlap?

`start1 <= end2 && start2 <= end1`

## Q3. Insert Interval vs Merge Intervals difference?

Insert is online/single insertion; merge processes full batch.

## Q4. Can median be solved in O(m+n)?

Yes by merging, but interviewer expects O(log) follow-up.

---

# One-Line Revision

```text
Median = partition binary search; Intervals = sort + merge overlapping ranges.
```

---

*End of Day 22 LeetCode*
