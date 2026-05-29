# Day 39 — Prefix Sum & Sliding Window Advanced

**Topics:** Brick Wall · Subarray Sums Divisible by K · Longest Continuous Subarray With Absolute Diff Limit

---

# 1. Brick Wall

## Problem Statement

Find vertical line that crosses fewest bricks. Line can be at grid edge. Each row is brick widths summing to same total.

---

# Optimized Approach

For each row, accumulate prefix positions (except final edge). Count frequency of prefix sums — min crosses = `rows - maxFrequency`.

```java
class Solution {
    public int leastBricks(List<List<Integer>> wall) {
        Map<Integer, Integer> count = new HashMap<>();
        int max = 0;
        for (List<Integer> row : wall) {
            int prefix = 0;
            for (int i = 0; i < row.size() - 1; i++) {
                prefix += row.get(i);
                max = Math.max(max, count.merge(prefix, 1, Integer::sum));
            }
        }
        return wall.size() - max;
    }
}
```

```text
Time: O(total bricks)   Space: O(rows)
```

---

# 2. Subarray Sums Divisible by K

## Problem Statement

Return count of contiguous subarrays whose sum is divisible by `k` (handles negative numbers).

---

# Optimized Approach (Prefix Sum Modulo)

```text
If (prefix[j] - prefix[i]) % k == 0
→ prefix[j] % k == prefix[i] % k
```

Track frequency of prefix mod `k`; for each new prefix, add `count[mod]`.

```java
class Solution {
    public int subarraysDivByK(int[] nums, int k) {
        Map<Integer, Integer> count = new HashMap<>();
        count.put(0, 1);
        int prefix = 0, result = 0;

        for (int num : nums) {
            prefix += num;
            int mod = ((prefix % k) + k) % k; // handle negative mod
            result += count.getOrDefault(mod, 0);
            count.merge(mod, 1, Integer::sum);
        }
        return result;
    }
}
```

---

# JavaScript Solution

```javascript
function subarraysDivByK(nums, k) {
  const count = new Map([[0, 1]]);
  let prefix = 0, result = 0;
  for (const num of nums) {
    prefix += num;
    const mod = ((prefix % k) + k) % k;
    result += count.get(mod) || 0;
    count.set(mod, (count.get(mod) || 0) + 1);
  }
  return result;
}
```

---

# 3. Longest Continuous Subarray With Absolute Diff ≤ Limit

## Problem Statement

Given array and `limit`, find longest subarray where `max - min ≤ limit`.

---

# Optimized Approach (Sliding Window + Monotonic Deque)

Maintain window `[left, right]` with deques tracking min and max.

```java
class Solution {
    public int longestSubarray(int[] nums, int limit) {
        Deque<Integer> maxQ = new ArrayDeque<>();
        Deque<Integer> minQ = new ArrayDeque<>();
        int left = 0, best = 0;

        for (int right = 0; right < nums.length; right++) {
            while (!maxQ.isEmpty() && nums[maxQ.peekLast()] <= nums[right]) maxQ.pollLast();
            maxQ.offerLast(right);
            while (!minQ.isEmpty() && nums[minQ.peekLast()] >= nums[right]) minQ.pollLast();
            minQ.offerLast(right);

            while (nums[maxQ.peekFirst()] - nums[minQ.peekFirst()] > limit) {
                left++;
                if (maxQ.peekFirst() < left) maxQ.pollFirst();
                if (minQ.peekFirst() < left) minQ.pollFirst();
            }
            best = Math.max(best, right - left + 1);
        }
        return best;
    }
}
```

Alternative: TreeMap/multiset window — O(n log n).

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Brick Wall | Prefix sum frequency | O(bricks) |
| Subarray Div K | Prefix mod + hashmap | O(n) |
| Longest Subarray Limit | Sliding window + deque | O(n) |

---

# Interview Questions

## Why normalize negative mod in divisible K?
Java `%` can be negative; `((prefix % k) + k) % k` gives consistent remainder in `[0, k-1)`.

## Brick Wall — why skip last brick per row?
Line at total width is edge — does not cross any brick.

## Deque vs multiset for sliding window max?
Both O(n); deque is O(1) amortized per element; multiset O(log n) simpler to code.

---

*End of Day 39 LeetCode*
