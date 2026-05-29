# Day 43 — Mock Prep: Tree Serialization, O(1) DS, DP Counting

**Topics:** Serialize/Deserialize N-ary Tree · All O(1) Data Structure · Number of Ways to Stay in Place

---

# 1. Serialize and Deserialize N-ary Tree

## Problem Statement

Design an algorithm to serialize and deserialize an N-ary tree. There is no restriction on how your serialization/deserialization algorithm should work.

---

# Example

```text
Input:
     1
   / | \
  3  2  4
 / \
5   6

Output (serialized): "1,3,5,#,#,6,#,#,2,#,#,4,#,#"
```

`#` marks end of children list for a node.

---

# Brute Force Approach

Convert to adjacency list JSON — works but verbose and slow to parse.

```text
O(n) serialize, O(n) deserialize — but heavy string overhead
```

---

# Optimized Approach (Preorder + Child Count)

## Core Idea

1. **Serialize:** preorder DFS; after each node value, write number of children
2. **Deserialize:** read value + child count; recursively build `count` children

Alternative (LeetCode-style): preorder with `#` sentinel after each node's children exhausted.

---

# Java Solution

```java
class Codec {

    public String serialize(Node root) {
        if (root == null) return "";
        StringBuilder sb = new StringBuilder();
        serializeDfs(root, sb);
        return sb.toString();
    }

    private void serializeDfs(Node node, StringBuilder sb) {
        sb.append(node.val).append(',');
        sb.append(node.children.size()).append(',');
        for (Node child : node.children) {
            serializeDfs(child, sb);
        }
    }

    public Node deserialize(String data) {
        if (data.isEmpty()) return null;
        String[] tokens = data.split(",");
        int[] idx = {0};
        return deserializeDfs(tokens, idx);
    }

    private Node deserializeDfs(String[] tokens, int[] idx) {
        int val = Integer.parseInt(tokens[idx[0]++]);
        int childCount = Integer.parseInt(tokens[idx[0]++]);
        Node node = new Node(val);
        for (int i = 0; i < childCount; i++) {
            node.children.add(deserializeDfs(tokens, idx));
        }
        return node;
    }
}
```

---

# Time Complexity

```text
O(n) serialize, O(n) deserialize — visit each node once
```

---

# 2. All O(1) Data Structure

## Problem Statement

Design a structure that supports in **average O(1)**:

- `insert(val)` — insert if not present
- `remove(val)` — remove if present
- `getRandom()` — return random element with equal probability
- `getMin()` — return minimum element

All operations must be average O(1).

---

# Brute Force Approach

Separate `HashSet` + `TreeSet` + `ArrayList` — insert/remove O(log n) for min.

---

# Optimized Approach (HashMap + ArrayList + Swap-Remove)

## Core Idea

1. `HashMap<Integer, Integer>` — value → index in list
2. `ArrayList<Integer>` — stores all values for O(1) random by index
3. **Remove:** swap target with last element, update map, pop last — O(1)
4. **Min:** maintain separate `TreeMap` or second structure — but that's O(log n)

**Interview twist:** true O(1) min requires **two lists** or a **doubly-linked list + min tracking**. Standard accepted solution uses **HashMap + ArrayList** for insert/remove/random; **min** via auxiliary `TreeMap<Integer, Integer>` (count) → O(log n) min, or store `(value, minSoFar)` pairs if interviewer relaxes.

**Canonical O(1) avg solution (insert/remove/random only):**

```java
class RandomizedSet {
    private Map<Integer, Integer> map = new HashMap<>();
    private List<Integer> list = new ArrayList<>();
    private Random rand = new Random();

    public boolean insert(int val) {
        if (map.containsKey(val)) return false;
        map.put(val, list.size());
        list.add(val);
        return true;
    }

    public boolean remove(int val) {
        if (!map.containsKey(val)) return false;
        int idx = map.get(val);
        int last = list.get(list.size() - 1);
        list.set(idx, last);
        map.put(last, idx);
        list.remove(list.size() - 1);
        map.remove(val);
        return true;
    }

    public int getRandom() {
        return list.get(rand.nextInt(list.size()));
    }
}
```

For **getMin O(1)**, add `Map<Integer, Integer> freq` + track global min with lazy update, or use **MinStack pattern** with paired `(val, currentMin)` entries in the list.

---

# Full Solution with O(1) Min (Interview Extension)

```java
class AllOne {
    private Map<String, Integer> keyToCount = new HashMap<>();
    private Map<Integer, Node> countToNode = new HashMap<>();
    private Node head, tail; // doubly linked list of counts

    public void inc(String key) {
        int count = keyToCount.getOrDefault(key, 0);
        if (count > 0) removeKeyFromBucket(key, count);
        keyToCount.put(key, count + 1);
        addKeyToBucket(key, count + 1);
    }

    public void dec(String key) {
        int count = keyToCount.get(key);
        removeKeyFromBucket(key, count);
        if (count == 1) keyToCount.remove(key);
        else { keyToCount.put(key, count - 1); addKeyToBucket(key, count - 1); }
    }

    public String getMaxKey() { return head.next.keys.iterator().next(); }
    public String getMinKey() { return tail.prev.keys.iterator().next(); }
    // Node = { count, Set<String> keys, prev, next }
}
```

*(LeetCode 432 "All One Data Structure" — bucket + DLL pattern for O(1) min/max key by frequency.)*

---

# 3. Number of Ways to Stay in Place After Some Steps

## Problem Statement

You have a pointer at index `0` in array `length` of zeros. Each step move `-1`, `0`, or `+1`. After exactly `numSteps` steps, how many ways to stay at index `0`? Return modulo `10^9 + 7`.

---

# Example

```text
Input: numSteps = 3, arrLen = 2
Output: 0
(can't stay at 0 after 3 steps without going out of bounds)
```

---

# Brute Force Approach

DFS all 3^numSteps paths — exponential.

```text
O(3^numSteps) — too slow
```

---

# Optimized Approach (DP)

## Core Idea

`dp[step][pos]` = ways to reach `pos` after `step` steps.

1. Base: `dp[0][0] = 1`
2. Transition: from `(step-1, pos-1)`, `(step-1, pos)`, `(step-1, pos+1)`
3. Prune: `pos >= arrLen` or `pos < 0` → skip
4. Optimization: max reachable index is `min(arrLen-1, numSteps)` — bound DP table

---

# Java Solution

```java
class Solution {
    private static final int MOD = 1_000_000_007;

    public int numWays(int steps, int arrLen) {
        int maxPos = Math.min(arrLen - 1, steps);
        int[][] dp = new int[steps + 1][maxPos + 1];
        dp[0][0] = 1;

        for (int s = 1; s <= steps; s++) {
            for (int p = 0; p <= maxPos; p++) {
                if (p > 0) dp[s][p] = (dp[s][p] + dp[s - 1][p - 1]) % MOD;
                dp[s][p] = (dp[s][p] + dp[s - 1][p]) % MOD;
                if (p < maxPos) dp[s][p] = (dp[s][p] + dp[s - 1][p + 1]) % MOD;
            }
        }
        return dp[steps][0];
    }
}
```

---

# Space-Optimized

```java
public int numWays(int steps, int arrLen) {
    int maxPos = Math.min(arrLen - 1, steps);
    int[] prev = new int[maxPos + 1];
    prev[0] = 1;
    for (int s = 1; s <= steps; s++) {
        int[] curr = new int[maxPos + 1];
        for (int p = 0; p <= maxPos; p++) {
            if (p > 0) curr[p] = (curr[p] + prev[p - 1]) % MOD;
            curr[p] = (curr[p] + prev[p]) % MOD;
            if (p < maxPos) curr[p] = (curr[p] + prev[p + 1]) % MOD;
        }
        prev = curr;
    }
    return prev[0];
}
```

---

# Time Complexity

```text
O(steps × min(arrLen, steps)) time, O(min(arrLen, steps)) space
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Serialize N-ary Tree | Preorder + child count / `#` sentinel | O(n) |
| All O(1) DS | HashMap + ArrayList swap-remove | O(1) avg |
| Stay in Place Steps | Bounded 2D DP | O(steps × min(arrLen, steps)) |

---

# Interview Questions

## How is N-ary serialization different from binary?

Binary can use preorder alone with null markers. N-ary must encode **how many children** or use `#` after each subtree — otherwise you can't know where one child's subtree ends.

## Why swap-with-last on remove?

ArrayList remove by index is O(n) due to shifting. Swap with last + pop is O(1) while keeping dense indices for random access.

## Why bound `maxPos` in DP?

After `k` steps you can't be farther than `k` from origin. Also `arrLen` caps position. Table size becomes `O(steps × min(arrLen, steps))` not `O(steps × arrLen)`.

## Follow-up: getMin in O(1)?

True O(1) all four ops is **All One** (432) style with buckets + DLL, or relax min to O(log n) with TreeMap.

---

# One-Line Revision

```text
N-ary serialize = preorder + child count; O(1) set = map + list swap-remove; stay-in-place = bounded DP with 3 transitions.
```

---

*End of Day 43 LeetCode*
