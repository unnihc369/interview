# Day 17 — Graph Patterns

**Topics:** Number of Connected Components · Graph Valid Tree · Alien Dictionary (Topological Sort)

---

# 1. Number of Connected Components

## Problem Statement

Given `n` nodes labeled `0` to `n-1` and undirected edges, return the number of connected components.

---

# Example

```text
Input: n = 5, edges = [[0,1],[1,2],[3,4]]
Output: 2
```

Components: {0,1,2} and {3,4}

---

# Optimized Approach (Union-Find)

## Core Idea

- Initialize each node as its own parent
- For each edge, union the two nodes
- Count distinct roots

---

# Java Solution

```java
class Solution {
    public int countComponents(int n, int[][] edges) {
        int[] parent = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;

        int components = n;
        for (int[] edge : edges) {
            if (union(parent, edge[0], edge[1])) {
                components--;
            }
        }
        return components;
    }

    private int find(int[] parent, int x) {
        if (parent[x] != x) parent[x] = find(parent, parent[x]);
        return parent[x];
    }

    private boolean union(int[] parent, int a, int b) {
        int rootA = find(parent, a);
        int rootB = find(parent, b);
        if (rootA == rootB) return false;
        parent[rootA] = rootB;
        return true;
    }
}
```

---

# Alternative — DFS

Build adjacency list, DFS from each unvisited node, increment component count.

```text
O(n + e) time
```

---

# Time Complexity (Union-Find)

```text
O(n + e * α(n)) ≈ O(n + e)
```

---

# 2. Graph Valid Tree

## Problem Statement

Given `n` nodes and undirected edges, determine if edges form a **valid tree**.

Valid tree conditions:
1. Exactly `n - 1` edges
2. No cycles
3. All nodes connected (single component)

---

# Example

```text
Input: n = 5, edges = [[0,1],[0,2],[0,3],[1,4]]
Output: true

Input: n = 5, edges = [[0,1],[1,2],[2,3],[1,3],[1,4]]
Output: false (cycle)
```

---

# Optimized Approach

## Quick Checks

```text
if (edges.length != n - 1) return false;
```

Then Union-Find or DFS — if no cycle and all connected → valid tree.

---

# Java Solution (Union-Find)

```java
class Solution {
    public boolean validTree(int n, int[][] edges) {
        if (edges.length != n - 1) return false;

        int[] parent = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;

        for (int[] edge : edges) {
            if (!union(parent, edge[0], edge[1])) return false;
        }
        return true;
    }

    private int find(int[] parent, int x) {
        if (parent[x] != x) parent[x] = find(parent, parent[x]);
        return parent[x];
    }

    private boolean union(int[] parent, int a, int b) {
        int rootA = find(parent, a);
        int rootB = find(parent, b);
        if (rootA == rootB) return false;
        parent[rootA] = rootB;
        return true;
    }
}
```

---

# JavaScript Solution

```javascript
function validTree(n, edges) {
  if (edges.length !== n - 1) return false;
  const parent = Array.from({ length: n }, (_, i) => i);
  function find(x) {
    if (parent[x] !== x) parent[x] = find(parent[x]);
    return parent[x];
  }
  for (const [a, b] of edges) {
    const ra = find(a), rb = find(b);
    if (ra === rb) return false;
    parent[ra] = rb;
  }
  return true;
}
```

---

# Time Complexity

```text
O(n + e)
```

---

# 3. Alien Dictionary (Topological Sort)

## Problem Statement

Given sorted words from an alien language, derive the character order. Return empty string if invalid (cycle detected).

---

# Example

```text
Input: ["wrt","wrf","er","ett","rftt"]
Output: "wertf"

Explanation:
wrt vs wrf → t before f
wrt vs er  → w before e
...
```

---

# Optimized Approach

## Core Idea

1. Compare adjacent words to find character precedence edges
2. Build directed graph + in-degree count
3. Kahn's algorithm (BFS topological sort)
4. If result length < unique chars → cycle

---

# Java Solution

```java
class Solution {
    public String alienOrder(String[] words) {
        Map<Character, Set<Character>> graph = new HashMap<>();
        Map<Character, Integer> inDegree = new HashMap<>();

        for (String word : words) {
            for (char c : word.toCharArray()) {
                graph.putIfAbsent(c, new HashSet<>());
                inDegree.putIfAbsent(c, 0);
            }
        }

        for (int i = 0; i < words.length - 1; i++) {
            String w1 = words[i], w2 = words[i + 1];
            if (w1.length() > w2.length() && w1.startsWith(w2)) return "";

            for (int j = 0; j < Math.min(w1.length(), w2.length()); j++) {
                char c1 = w1.charAt(j), c2 = w2.charAt(j);
                if (c1 != c2) {
                    if (!graph.get(c1).contains(c2)) {
                        graph.get(c1).add(c2);
                        inDegree.put(c2, inDegree.get(c2) + 1);
                    }
                    break;
                }
            }
        }

        Queue<Character> queue = new LinkedList<>();
        for (char c : inDegree.keySet()) {
            if (inDegree.get(c) == 0) queue.offer(c);
        }

        StringBuilder sb = new StringBuilder();
        while (!queue.isEmpty()) {
            char c = queue.poll();
            sb.append(c);
            for (char next : graph.get(c)) {
                inDegree.put(next, inDegree.get(next) - 1);
                if (inDegree.get(next) == 0) queue.offer(next);
            }
        }

        return sb.length() == inDegree.size() ? sb.toString() : "";
    }
}
```

---

# Edge Case — Invalid Input

```text
["abc", "ab"]  → invalid (longer word comes before prefix)
```

Must return `""`.

---

# Time Complexity

```text
O(C) where C = total characters across all words
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Connected Components | Union-Find or DFS | O(n+e) |
| Graph Valid Tree | n-1 edges + no cycle | O(n+e) |
| Alien Dictionary | Build graph + Topological Sort | O(C) |

---

# Interview Questions

## Tree vs Graph?

Tree: connected, acyclic, n-1 edges. Graph: may have cycles, may be disconnected.

## Union-Find vs DFS for components?

Union-Find better for dynamic connectivity (edges added online). DFS simpler for static graph one-shot.

## How detect cycle in alien dictionary?

Topological sort produces fewer chars than unique set → cycle exists.

---

*End of Day 17 LeetCode*
