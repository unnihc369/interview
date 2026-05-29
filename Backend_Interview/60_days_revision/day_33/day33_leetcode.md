# Day 33 - Graph Advanced

---

# 1. Minimum Height Trees (MHT / Tree Centroids)

## Problem Statement

Tree with `n` nodes (edges `n-1`). Return all **roots** that minimize tree height. MHT roots are tree **centroids** (1 or 2 nodes).

---

# Core Idea

Topological peel: remove leaf nodes layer by layer until ≤2 nodes remain. Those are MHT roots.

---

# Java Solution

```java
class Solution {
    public List<Integer> findMinHeightTrees(int n, int[][] edges) {
        if (n == 1) return List.of(0);

        List<Set<Integer>> adj = new ArrayList<>();
        int[] degree = new int[n];
        for (int i = 0; i < n; i++) adj.add(new HashSet<>());
        for (int[] e : edges) {
            adj.get(e[0]).add(e[1]);
            adj.get(e[1]).add(e[0]);
            degree[e[0]]++;
            degree[e[1]]++;
        }

        Queue<Integer> leaves = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            if (degree[i] == 1) leaves.offer(i);
        }

        int remaining = n;
        while (remaining > 2) {
            int size = leaves.size();
            remaining -= size;
            for (int i = 0; i < size; i++) {
                int leaf = leaves.poll();
                for (int neighbor : adj.get(leaf)) {
                    if (--degree[neighbor] == 1) leaves.offer(neighbor);
                }
            }
        }
        return new ArrayList<>(leaves);
    }
}
```

---

# JavaScript Solution

```javascript
function findMinHeightTrees(n, edges) {
  if (n === 1) return [0];
  const adj = Array.from({ length: n }, () => new Set());
  const degree = Array(n).fill(0);
  for (const [u, v] of edges) {
    adj[u].add(v); adj[v].add(u);
    degree[u]++; degree[v]++;
  }
  let leaves = [];
  for (let i = 0; i < n; i++) if (degree[i] === 1) leaves.push(i);

  let remaining = n;
  while (remaining > 2) {
    const next = [];
    remaining -= leaves.length;
    for (const leaf of leaves) {
      for (const nb of adj[leaf]) {
        if (--degree[nb] === 1) next.push(nb);
      }
    }
    leaves = next;
  }
  return leaves;
}
```

---

# Time Complexity

```text
O(n)
```

---

# 2. Reconstruct Itinerary (Eulerian Path)

## Problem Statement

Given tickets `[from, to]`, reconstruct itinerary starting from `"JFK"`. Use all tickets exactly once. Smallest lexical order if multiple valid.

---

# Core Idea

Hierholzer's algorithm for Eulerian path. Sort destinations lexicographically. DFS, post-order append to result, reverse at end.

---

# Java Solution

```java
class Solution {
    public List<String> findItinerary(List<List<String>> tickets) {
        Map<String, PriorityQueue<String>> graph = new HashMap<>();
        for (List<String> t : tickets) {
            graph.computeIfAbsent(t.get(0), k -> new PriorityQueue<>()).add(t.get(1));
        }

        LinkedList<String> route = new LinkedList<>();
        dfs("JFK", graph, route);
        return route;
    }

    private void dfs(String airport, Map<String, PriorityQueue<String>> graph,
                   LinkedList<String> route) {
        PriorityQueue<String> dests = graph.get(airport);
        while (dests != null && !dests.isEmpty()) {
            dfs(dests.poll(), graph, route);
        }
        route.addFirst(airport);
    }
}
```

---

# JavaScript Solution

```javascript
function findItinerary(tickets) {
  const graph = {};
  for (const [from, to] of tickets) {
    if (!graph[from]) graph[from] = [];
    graph[from].push(to);
  }
  for (const k in graph) graph[k].sort();

  const route = [];
  function dfs(airport) {
    const dests = graph[airport];
    while (dests && dests.length) {
      dfs(dests.shift());
    }
    route.push(airport);
  }
  dfs('JFK');
  return route.reverse();
}
```

---

# Time Complexity

```text
O(E log E) — sorting edges
```

---

# 3. Critical Connections (Bridges in Graph)

## Problem Statement

Given `n` servers and bidirectional connections, return **critical connections** (bridges) — removing which disconnects the graph.

---

# Core Idea

Tarjan's algorithm: DFS with `disc` (discovery time) and `low` (lowest reachable). Bridge when `low[neighbor] > disc[node]`.

---

# Java Solution

```java
class Solution {
    private int time = 0;
    private List<List<Integer>> result;

    public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
        List<List<Integer>> graph = new ArrayList<>();
        for (int i = 0; i < n; i++) graph.add(new ArrayList<>());
        for (List<Integer> c : connections) {
            graph.get(c.get(0)).add(c.get(1));
            graph.get(c.get(1)).add(c.get(0));
        }

        int[] disc = new int[n];
        int[] low = new int[n];
        Arrays.fill(disc, -1);
        result = new ArrayList<>();
        dfs(0, -1, graph, disc, low);
        return result;
    }

    private void dfs(int node, int parent, List<List<Integer>> graph,
                     int[] disc, int[] low) {
        disc[node] = low[node] = time++;

        for (int neighbor : graph.get(node)) {
            if (neighbor == parent) continue;
            if (disc[neighbor] == -1) {
                dfs(neighbor, node, graph, disc, low);
                low[node] = Math.min(low[node], low[neighbor]);
                if (low[neighbor] > disc[node]) {
                    result.add(List.of(node, neighbor));
                }
            } else {
                low[node] = Math.min(low[node], disc[neighbor]);
            }
        }
    }
}
```

---

# JavaScript Solution

```javascript
function criticalConnections(n, connections) {
  const graph = Array.from({ length: n }, () => []);
  for (const [u, v] of connections) {
    graph[u].push(v); graph[v].push(u);
  }
  const disc = Array(n).fill(-1), low = Array(n).fill(0);
  const result = [];
  let time = 0;

  function dfs(node, parent) {
    disc[node] = low[node] = time++;
    for (const nb of graph[node]) {
      if (nb === parent) continue;
      if (disc[nb] === -1) {
        dfs(nb, node);
        low[node] = Math.min(low[node], low[nb]);
        if (low[nb] > disc[node]) result.push([node, nb]);
      } else {
        low[node] = Math.min(low[node], disc[nb]);
      }
    }
  }
  dfs(0, -1);
  return result;
}
```

---

# Time Complexity

```text
O(V + E)
```

---

# Pattern Quick Summary

| Problem | Algorithm | Time |
|---------|-----------|------|
| Minimum Height Trees | Leaf peeling (topological) | O(n) |
| Reconstruct Itinerary | Eulerian path (Hierholzer) | O(E log E) |
| Critical Connections | Tarjan bridges | O(V+E) |

---

# Interview Questions

## MHT — why at most 2 roots?

Tree centroids are 1 or 2 nodes (middle of longest path).

## Itinerary — why add airport post-order?

Post-order DFS consumes edges; reverse gives JFK-first route.

## Bridge vs articulation point?

Bridge is critical **edge**. Articulation point is critical **node** (`low[child] >= disc[node]`).

## Critical Connections in distributed systems?

Bridges = single points of failure in network topology; add redundant links.

---

*End of Day 33 LeetCode*
