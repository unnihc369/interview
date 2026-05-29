# Day 37 — Stack & Design Data Structures

**Topics:** Maximum Frequency Stack · Design Twitter · Time Based Key-Value Store

---

# 1. Maximum Frequency Stack

## Problem Statement

Design stack `push`, `pop`, `popMax` where `popMax` removes and returns the most frequent element; on tie, return closest to stack top.

---

# Example

```text
push(5), push(7), push(5), push(4), push(5)
popMax() → 5 (freq 3, nearest top)
pop()    → 4
popMax() → 5
```

---

# Optimized Approach

Maintain:

- `freq` map: value → count
- `group` map: frequency → stack of values at that freq
- `maxFreq` current maximum frequency

On `popMax`: pop from `group.get(maxFreq)`, decrement freq, decrease `maxFreq` if stack empty.

---

# Java Solution

```java
class FreqStack {
    private final Map<Integer, Integer> freq = new HashMap<>();
    private final Map<Integer, Deque<Integer>> group = new HashMap<>();
    private int maxFreq = 0;

    public void push(int x) {
        int f = freq.merge(x, 1, Integer::sum);
        maxFreq = Math.max(maxFreq, f);
        group.computeIfAbsent(f, k -> new ArrayDeque<>()).push(x);
    }

    public int pop() {
        Deque<Integer> stack = group.get(maxFreq);
        int x = stack.pop();
        freq.put(x, freq.get(x) - 1);
        if (stack.isEmpty()) maxFreq--;
        return x;
    }
}
```

---

# Time Complexity

```text
push/pop: O(1) average
```

---

# 2. Design Twitter (News Feed)

## Problem Statement

Implement `postTweet`, `getNewsFeed`, `follow`, `unfollow`. Feed shows 10 most recent tweets from self + followed users.

---

# Optimized Approach

- `userTweets`: userId → list of (tweetId, timestamp) — prepend new tweets
- `followers`: userId → Set of followeeIds
- `getNewsFeed`: merge recent tweets from self + follows (max 10) — use **min-heap of size 10** across followees or merge k sorted lists

Simple approach: collect all candidate tweets, sort by time, take 10 — O(F * T log(F*T)).

Heap approach for follow-up scale: O(F log 10).

---

# Java Solution (Simplified)

```java
class Twitter {
    private final Map<Integer, List<int[]>> tweets = new HashMap<>();
    private final Map<Integer, Set<Integer>> following = new HashMap<>();
    private int time = 0;

    public void postTweet(int userId, int tweetId) {
        tweets.computeIfAbsent(userId, k -> new ArrayList<>())
              .add(0, new int[]{tweetId, time++});
    }

    public List<Integer> getNewsFeed(int userId) {
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);
        following.computeIfAbsent(userId, k -> new HashSet<>()).add(userId);

        for (int uid : following.get(userId)) {
            List<int[]> list = tweets.getOrDefault(uid, List.of());
            for (int i = 0; i < Math.min(10, list.size()); i++) {
                pq.offer(list.get(i));
                if (pq.size() > 10) pq.poll();
            }
        }
        List<Integer> res = new ArrayList<>();
        while (!pq.isEmpty()) res.add(0, pq.poll()[0]);
        return res;
    }

    public void follow(int followerId, int followeeId) {
        following.computeIfAbsent(followerId, k -> new HashSet<>()).add(followeeId);
    }

    public void unfollow(int followerId, int followeeId) {
        following.getOrDefault(followerId, Set.of()).remove(followeeId);
    }
}
```

---

# 3. Time Based Key-Value Store

## Problem Statement

`set(key, value, timestamp)` and `get(key, timestamp)` return value with largest timestamp ≤ query timestamp.

---

# Optimized Approach

`Map<String, TreeMap<Integer, String>>` — binary search on timestamps per key.

---

# Java Solution

```java
class TimeMap {
    private final Map<String, TreeMap<Integer, String>> map = new HashMap<>();

    public void set(String key, String value, int timestamp) {
        map.computeIfAbsent(key, k -> new TreeMap<>()).put(timestamp, value);
    }

    public String get(String key, int timestamp) {
        TreeMap<Integer, String> tree = map.get(key);
        if (tree == null) return "";
        Map.Entry<Integer, String> entry = tree.floorEntry(timestamp);
        return entry == null ? "" : entry.getValue();
    }
}
```

---

# JavaScript Solution (TimeMap)

```javascript
class TimeMap {
  constructor() { this.map = new Map(); }

  set(key, value, timestamp) {
    if (!this.map.has(key)) this.map.set(key, []);
    this.map.get(key).push([timestamp, value]);
  }

  get(key, timestamp) {
    const arr = this.map.get(key);
    if (!arr) return "";
    let lo = 0, hi = arr.length - 1, ans = "";
    while (lo <= hi) {
      const mid = (lo + hi) >> 1;
      if (arr[mid][0] <= timestamp) {
        ans = arr[mid][1];
        lo = mid + 1;
      } else hi = mid - 1;
    }
    return ans;
  }
}
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Freq Stack | freq + group stacks | O(1) |
| Design Twitter | HashMap + heap merge | O(F log 10) |
| TimeMap | TreeMap floorEntry / binary search | O(log n) per op |

---

# Interview Questions

## Freq Stack — why group by frequency?
O(1) access to "most frequent nearest top" without scanning entire stack.

## Twitter — how scale getNewsFeed?
Fan-out on write (push to followers' feeds) vs fan-out on read (merge at read time). Twitter uses hybrid.

## TimeMap — can timestamps be duplicate?
Problem usually allows strictly increasing timestamps per key; if duplicates, store list at timestamp.

---

*End of Day 37 LeetCode*
