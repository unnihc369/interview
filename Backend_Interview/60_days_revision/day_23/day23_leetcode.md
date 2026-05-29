# Day 23 - Greedy Intervals & Heap

---

# 1. Non-overlapping Intervals

## Problem Statement

Given intervals, return the **minimum number of intervals to remove** so the rest are non-overlapping.

---

# Example

```text
Input:  [[1,2],[2,3],[3,4],[1,3]]
Output: 1
(remove [1,3] to keep others)
```

---

# Core Idea — Greedy by End Time

Sort by **end time** ascending. Keep interval with earliest end; remove any that overlap.

```text
Equivalent to: maximum number of non-overlapping intervals
Answer = n - maxNonOverlapping
```

---

# Java Solution

```java
class Solution {
    public int eraseOverlapIntervals(int[][] intervals) {
        if (intervals.length == 0) return 0;

        Arrays.sort(intervals, (a, b) -> a[1] - b[1]);
        int count = 1;
        int end = intervals[0][1];

        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] >= end) {
                count++;
                end = intervals[i][1];
            }
        }
        return intervals.length - count;
    }
}
```

---

# Time Complexity

```text
O(n log n)
```

---

# 2. Meeting Rooms II

## Problem Statement

Given meeting time intervals, return the **minimum number of conference rooms** required.

---

# Example

```text
Input: [[0,30],[5,10],[15,20]]
Output: 2
```

---

# Approach 1 — Min Heap on End Times

1. Sort by start time
2. Min-heap stores end times of ongoing meetings
3. If new start ≥ smallest end → reuse room (poll heap)
4. Else → need new room (push new end)

---

# Java Solution

```java
class Solution {
    public int minMeetingRooms(int[][] intervals) {
        Arrays.sort(intervals, (a, b) -> a[0] - b[0]);
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();

        for (int[] interval : intervals) {
            if (!minHeap.isEmpty() && interval[0] >= minHeap.peek()) {
                minHeap.poll();
            }
            minHeap.offer(interval[1]);
        }
        return minHeap.size();
    }
}
```

---

# Approach 2 — Sweep Line (Advanced)

Separate start and end events; track concurrent count.

```text
+1 at start, -1 at end → max concurrent = answer
```

Same O(n log n) time, useful when explaining "timeline" intuition.

---

# Time Complexity

```text
O(n log n)
```

---

# 3. Employee Free Time

## Problem Statement

Given a list of employees, each with sorted disjoint intervals representing working hours, return **common free time** for all employees.

*(LeetCode Premium — common in interviews)*

---

# Example

```text
Employee1: [[1,3],[6,7]]
Employee2: [[2,4]]
Employee3: [[2,5],[9,12]]

Output: [[5,6],[7,9]]
```

---

# Core Idea

1. Merge all intervals into one sorted list
2. Gaps between merged intervals = free time

---

# Java Solution

```java
class Solution {
    public List<Interval> employeeFreeTime(List<List<Interval>> schedule) {
        List<Interval> all = new ArrayList<>();
        for (List<Interval> emp : schedule) {
            all.addAll(emp);
        }
        all.sort((a, b) -> a.start - b.start);

        List<Interval> merged = new ArrayList<>();
        Interval prev = all.get(0);
        for (int i = 1; i < all.size(); i++) {
            Interval curr = all.get(i);
            if (prev.end >= curr.start) {
                prev.end = Math.max(prev.end, curr.end);
            } else {
                merged.add(prev);
                prev = curr;
            }
        }
        merged.add(prev);

        List<Interval> free = new ArrayList<>();
        for (int i = 1; i < merged.size(); i++) {
            if (merged.get(i - 1).end < merged.get(i).start) {
                free.add(new Interval(merged.get(i - 1).end, merged.get(i).start));
            }
        }
        return free;
    }
}
```

---

# Time Complexity

```text
O(N log N) where N = total intervals across employees
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Non-overlapping Intervals | Greedy (sort by end) | O(n log n) |
| Meeting Rooms II | Min Heap / Sweep Line | O(n log n) |
| Employee Free Time | Merge all + find gaps | O(N log N) |

---

# Common Interview Questions

## Q1. Why sort by end for non-overlapping?

Earliest finishing interval leaves maximum room for future intervals — classic greedy proof.

## Q2. Meeting Rooms I vs II?

I: can one person attend all? (check overlap). II: min rooms = max concurrent meetings.

## Q3. Heap size at end = answer?

Yes for Meeting Rooms II — heap tracks active meeting end times.

## Q4. Employee Free Time without merging all?

Use priority queue of interval heads across k lists (merge k sorted lists), then find gaps.

---

# One-Line Revision

```text
Interval greedy = sort by end; Room scheduling = min-heap on end times; Free time = merge then gaps.
```

---

*End of Day 23 LeetCode*
