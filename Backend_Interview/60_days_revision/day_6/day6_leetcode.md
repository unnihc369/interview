# Day 6 — LeetCode

## Topics
- Merge Intervals
- Meeting Rooms II
- Kth Largest Element

---

# 1. Merge Intervals

## Problem

Merge overlapping intervals.

---

## Java Solution

```java
class Solution {

    public int[][] merge(int[][] intervals) {

        Arrays.sort(intervals,
                (a, b) -> a[0] - b[0]);

        List<int[]> result = new ArrayList<>();

        int[] current = intervals[0];

        for (int i = 1; i < intervals.length; i++) {

            if (current[1] >= intervals[i][0]) {

                current[1] = Math.max(
                        current[1],
                        intervals[i][1]
                );

            } else {

                result.add(current);

                current = intervals[i];
            }
        }

        result.add(current);

        return result.toArray(new int[result.size()][]);
    }
}
```

---

# 2. Meeting Rooms II

## Pattern
Min Heap

---

## Java Solution

```java
class Solution {

    public int minMeetingRooms(int[][] intervals) {

        Arrays.sort(intervals,
                (a, b) -> a[0] - b[0]);

        PriorityQueue<Integer> pq =
                new PriorityQueue<>();

        for (int[] interval : intervals) {

            if (!pq.isEmpty()
                    && pq.peek() <= interval[0]) {

                pq.poll();
            }

            pq.offer(interval[1]);
        }

        return pq.size();
    }
}
```

---

# 3. Kth Largest Element

## Pattern
Heap

---

## Java Solution

```java
class Solution {

    public int findKthLargest(
            int[] nums,
            int k) {

        PriorityQueue<Integer> pq =
                new PriorityQueue<>();

        for (int num : nums) {

            pq.offer(num);

            if (pq.size() > k) {
                pq.poll();
            }
        }

        return pq.peek();
    }
}
```

---

# Pattern Cheat Sheet

| Problem | Pattern |
|---|---|
| Merge Intervals | Sorting |
| Meeting Rooms II | Min Heap |
| Kth Largest | Heap |

---

*End of Day 6 LeetCode*