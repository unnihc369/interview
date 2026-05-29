# Day 50 — Final DSA Revision Guide + 3 Amazon Problems

**Week 8 · Monday · ~3 hours**  
**Focus:** Pattern recall under pressure · Amazon-style medium/hard · Java solutions

---

# Part 1 — 60-Minute Pattern Revision (No Code)

Use this as a spoken drill. If you cannot name **approach + complexity + edge case** in 30 seconds per row, revisit that week’s file.

---

## 1. Arrays & Hashing

| Pattern | When to use | Complexity | Gotcha |
|---------|-------------|------------|--------|
| Two pointers | Sorted array, pair sum, palindrome | O(n) | Move pointer that “locks” invariant |
| Prefix sum + map | Subarray sum = k | O(n) | Initialize map with `{0:1}` |
| Sliding window | Substring, max sum length k | O(n) | Shrink when constraint violated |
| Hash map frequency | Anagram, majority, two sum | O(n) | Key: what you store (count vs index) |

**Amazon favorites:** Two Sum, Group Anagrams, Product Except Self, Subarray Sum Equals K.

---

## 2. Stack & Queue

| Pattern | When | Gotcha |
|---------|------|--------|
| Monotonic stack | Next greater, histogram | Pop while `<=` or `<` based on problem |
| Min stack | GetMin O(1) | Auxiliary stack or pair stack |
| Deque BFS | Shortest path unweighted | Mark visited at **enqueue** not dequeue |

---

## 3. Binary Search

| Variant | Condition |
|---------|-----------|
| Classic | `lo <= hi`, mid on sorted array |
| Lower bound | First index where `arr[i] >= target` |
| Binary search on answer | Monotonic predicate on value (capacity, speed) |
| Two sorted arrays median | Partition so left sizes equal |

---

## 4. Trees

| Pattern | Template cue |
|---------|--------------|
| DFS post-order | LCA, max path sum, diameter |
| BFS level order | `size = queue.size()` inner loop |
| BST inorder | Kth smallest, validate BST (range) |

---

## 5. Graphs

| Pattern | Algorithm |
|---------|-----------|
| Connected components | DFS/BFS or Union-Find |
| Topological sort | Kahn BFS or DFS post-order |
| Shortest path (non-neg) | Dijkstra or BFS if unweighted |
| Multi-source BFS | Enqueue all sources day 0 (rotting oranges) |

---

## 6. Heap / Priority Queue

| Problem type | Heap type |
|--------------|-----------|
| K largest / K frequent | Min-heap size k |
| Merge K lists | Min-heap of list heads |
| Median stream | Two heaps |

---

## 7. Dynamic Programming

| Signal | Approach |
|--------|----------|
| Optimal substructure + overlapping | Top-down memo or bottom-up tabulation |
| Choice at index | `dp[i] = f(dp[i-1], dp[i-2], ...)` |
| Unbounded knapsack | Coin change inner loop forward |
| Grid path | `dp[r][c]` from top/left |

---

## 8. Backtracking

Template:

```text
void backtrack(state) {
  if (complete) { record; return; }
  for (choice : choices) {
    if (!valid) continue;
    apply(choice);
    backtrack(state);
    undo(choice);
  }
}
```

**Prune early:** N-Queens (column/diag sets), Word Search (visited mark).

---

## 9. Interview Delivery Checklist (Every Problem)

1. **Clarify** — constraints, duplicates, negative numbers, empty input  
2. **Examples** — normal + edge (empty, single, all same)  
3. **Brute force** — state complexity  
4. **Optimize** — name pattern  
5. **Code** — talk while writing  
6. **Test** — walk one example line by line  
7. **Follow-up** — space optimization, stream input, concurrency (say “out of scope unless required”)

---

# Part 2 — Problem 1: LRU Cache (LeetCode 146)

**Amazon frequency:** Very high · **Pattern:** HashMap + Doubly Linked List (or LinkedHashMap in practice, but interview wants DLL)

---

## Problem Statement

Design a data structure that supports `get(key)` and `put(key, value)` in O(1). When capacity exceeded, evict **least recently used** key.

---

## Approach

- `Map<Integer, Node>` for O(1) lookup  
- DLL: head = MRU, tail = LRU  
- On `get`/`put`: move node to head  
- On overflow: remove tail, delete from map  

---

## Java Solution

```java
class LRUCache {
    class Node {
        int key, val;
        Node prev, next;
        Node(int k, int v) { key = k; val = v; }
    }

    private final int capacity;
    private final Map<Integer, Node> map = new HashMap<>();
    private final Node head = new Node(0, 0);
    private final Node tail = new Node(0, 0);

    public LRUCache(int capacity) {
        this.capacity = capacity;
        head.next = tail;
        tail.prev = head;
    }

    public int get(int key) {
        if (!map.containsKey(key)) return -1;
        Node node = map.get(key);
        moveToHead(node);
        return node.val;
    }

    public void put(int key, int value) {
        if (map.containsKey(key)) {
            Node node = map.get(key);
            node.val = value;
            moveToHead(node);
            return;
        }
        Node node = new Node(key, value);
        map.put(key, node);
        addToHead(node);
        if (map.size() > capacity) {
            Node lru = tail.prev;
            remove(lru);
            map.remove(lru.key);
        }
    }

    private void moveToHead(Node node) {
        remove(node);
        addToHead(node);
    }

    private void addToHead(Node node) {
        node.next = head.next;
        node.prev = head;
        head.next.prev = node;
        head.next = node;
    }

    private void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
}
```

**Time:** O(1) per op · **Space:** O(capacity)

---

## Follow-ups

- Thread-safe LRU → `ReentrantReadWriteLock` or striped locks per segment  
- LFU Cache → `TreeMap` of frequencies or multiple DLLs per freq  

---

# Part 3 — Problem 2: Rotting Oranges (LeetCode 994)

**Amazon frequency:** High · **Pattern:** Multi-source BFS on grid

---

## Problem Statement

Grid: `0` empty, `1` fresh orange, `2` rotten. Each minute rotten oranges rot 4-directional neighbors. Return minimum minutes until no fresh orange remains, or `-1` if impossible.

---

## Approach

1. Enqueue all rotten cells, count fresh  
2. BFS level-by-level (minutes = levels)  
3. Each step rot neighbors, decrement fresh  
4. If `fresh > 0` after BFS → return -1  

---

## Java Solution

```java
class Solution {
    private static final int[][] DIRS = {{0,1},{0,-1},{1,0},{-1,0}};

    public int orangesRotting(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        Queue<int[]> q = new LinkedList<>();
        int fresh = 0;

        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == 2) q.offer(new int[]{r, c});
                else if (grid[r][c] == 1) fresh++;
            }
        }
        if (fresh == 0) return 0;

        int minutes = 0;
        while (!q.isEmpty()) {
            int size = q.size();
            boolean rotted = false;
            for (int i = 0; i < size; i++) {
                int[] cell = q.poll();
                for (int[] d : DIRS) {
                    int nr = cell[0] + d[0], nc = cell[1] + d[1];
                    if (nr < 0 || nr >= m || nc < 0 || nc >= n || grid[nr][nc] != 1) continue;
                    grid[nr][nc] = 2;
                    fresh--;
                    q.offer(new int[]{nr, nc});
                    rotted = true;
                }
            }
            if (rotted) minutes++;
        }
        return fresh == 0 ? minutes : -1;
    }
}
```

**Time:** O(m·n) · **Space:** O(m·n) queue worst case

---

# Part 4 — Problem 3: Accounts Merge (LeetCode 721)

**Amazon frequency:** High · **Pattern:** Union-Find or DFS on email graph

---

## Problem Statement

Each account: `["Name", "email1", "email2", ...]`. Merge accounts that share **any** email. Return merged lists sorted by email; names can differ per account but merged group uses first seen name.

---

## Approach (Union-Find)

1. Map email → index  
2. Union emails within same account  
3. Group emails by root parent  
4. Attach name from any account in group  

---

## Java Solution

```java
class Solution {
    public List<List<String>> accountsMerge(List<List<String>> accounts) {
        Map<String, Integer> emailId = new HashMap<>();
        int id = 0;
        for (List<String> acc : accounts) {
            for (int i = 1; i < acc.size(); i++) {
                emailId.putIfAbsent(acc.get(i), id++);
            }
        }

        UnionFind uf = new UnionFind(id);
        Map<String, String> emailToName = new HashMap<>();

        for (List<String> acc : accounts) {
            String name = acc.get(0);
            int first = emailId.get(acc.get(1));
            for (int i = 1; i < acc.size(); i++) {
                emailToName.put(acc.get(i), name);
                uf.union(first, emailId.get(acc.get(i)));
            }
        }

        Map<Integer, List<String>> groups = new HashMap<>();
        for (Map.Entry<String, Integer> e : emailId.entrySet()) {
            int root = uf.find(e.getValue());
            groups.computeIfAbsent(root, k -> new ArrayList<>()).add(e.getKey());
        }

        List<List<String>> result = new ArrayList<>();
        for (List<String> emails : groups.values()) {
            Collections.sort(emails);
            String name = emailToName.get(emails.get(0));
            List<String> merged = new ArrayList<>();
            merged.add(name);
            merged.addAll(emails);
            result.add(merged);
        }
        return result;
    }

    static class UnionFind {
        int[] parent, rank;
        UnionFind(int n) {
            parent = new int[n];
            for (int i = 0; i < n; i++) parent[i] = i;
            rank = new int[n];
        }
        int find(int x) {
            if (parent[x] != x) parent[x] = find(parent[x]);
            return parent[x];
        }
        void union(int a, int b) {
            int ra = find(a), rb = find(b);
            if (ra == rb) return;
            if (rank[ra] < rank[rb]) parent[ra] = rb;
            else if (rank[ra] > rank[rb]) parent[rb] = ra;
            else { parent[rb] = ra; rank[ra]++; }
        }
    }
}
```

**Time:** O(N · α(N)) · **Space:** O(N) emails

---

# Part 5 — Self-Test (45 min, timed)

| # | Problem | Target time |
|---|---------|-------------|
| 1 | LRU Cache | 25 min |
| 2 | Rotting Oranges | 15 min |
| 3 | Accounts Merge | 25 min |

If you finish early: implement **LFU Cache** sketch or **Word Ladder I** BFS.

---

# Interview Questions

**Why doubly linked list for LRU?**  
Singly linked list cannot remove arbitrary node in O(1) without knowing predecessor.

**BFS vs DFS for rotting oranges?**  
BFS gives shortest time (levels). DFS would need extra tracking for minimum time.

**Accounts merge without Union-Find?**  
Build graph: edge between emails in same account; DFS connected components.

---

*End of Day 50 — LeetCode*
