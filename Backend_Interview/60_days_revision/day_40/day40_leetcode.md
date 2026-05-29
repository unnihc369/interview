# Day 40 — Greedy Intervals & Simulation

**Topics:** Minimum Arrows to Burst Balloons · Queue Reconstruction by Height · Dota2 Senate

---

# 1. Minimum Number of Arrows to Burst Balloons

## Problem Statement

Balloons are intervals `[xstart, xend]`. One arrow at `x` bursts all balloons where `xstart ≤ x ≤ xend`. Return minimum arrows.

---

# Optimized Approach (Greedy — Activity Selection)

Sort by end coordinate. Shoot arrow at first balloon's end; skip balloons covered; repeat.

```java
class Solution {
    public int findMinArrowShots(int[][] points) {
        Arrays.sort(points, Comparator.comparingInt(a -> a[1]));
        int arrows = 1;
        int arrowPos = points[0][1];

        for (int i = 1; i < points.length; i++) {
            if (points[i][0] > arrowPos) {
                arrows++;
                arrowPos = points[i][1];
            }
        }
        return arrows;
    }
}
```

Same pattern as **Non-overlapping Intervals** — maximize balloons burst per arrow.

---

# 2. Queue Reconstruction by Height

## Problem Statement

Each person `[h, k]` — `h` = height, ``k` = count of people in front with height ≥ h. Reconstruct queue.

---

# Optimized Approach

1. Sort by height descending, then by `k` ascending
2. Insert each person at index `k` in result list (LinkedList)

```java
class Solution {
    public int[][] reconstructQueue(int[][] people) {
        Arrays.sort(people, (a, b) ->
            a[0] != b[0] ? b[0] - a[0] : a[1] - b[1]);

        List<int[]> result = new LinkedList<>();
        for (int[] person : people) {
            result.add(person[1], person);
        }
        return result.toArray(new int[0][]);
    }
}
```

```text
Time: O(n^2) due to LinkedList insert — acceptable for interview
```

---

# 3. Dota2 Senate

## Problem Statement

Senate string `R` and `D` vote to ban opposing party. Each senator bans nearest opponent in circular order. Predict winning party.

---

# Optimized Approach (Two Queues)

Store indices of R and D in queues. Compare front indices — smaller bans larger; winner re-enters at `i + n`.

```java
class Solution {
    public String predictPartyVictory(String senate) {
        Queue<Integer> r = new LinkedList<>();
        Queue<Integer> d = new LinkedList<>();
        int n = senate.length();

        for (int i = 0; i < n; i++) {
            if (senate.charAt(i) == 'R') r.offer(i);
            else d.offer(i);
        }

        while (!r.isEmpty() && !d.isEmpty()) {
            int ri = r.poll(), di = d.poll();
            if (ri < di) {
                r.offer(ri + n); // R bans D (di eliminated)
            } else {
                d.offer(di + n); // D bans R (ri eliminated)
            }
        }
        return r.isEmpty() ? "Dire" : "Radiant";
    }
}
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Min Arrows | Greedy sort by end | O(n log n) |
| Queue Reconstruction | Sort + insert at k | O(n²) |
| Dota2 Senate | Two queues simulation | O(n) |

---

# Interview Questions

## Min Arrows vs merge intervals?
Same greedy: pick point that covers most intervals (end of earliest-ending).

## Queue reconstruction — why sort tall first?
Shorter people don't affect taller people's `k`; inserting at `k` preserves constraint.

## Senate — why add `n` to index?
Simulates next round in circular senate without modulo confusion.

---

*End of Day 40 LeetCode*
