# Day 9 — Graph BFS Advanced

---

# 1. Word Ladder

## Problem Statement

Given `beginWord`, `endWord`, and a word list, find the length of shortest transformation sequence where only one letter changes at a time and each intermediate word must exist in the list. Return `0` if no sequence exists.

---

# Example

```text
beginWord = "hit", endWord = "cog"
wordList = ["hot","dot","dog","lot","log","cog"]
Output: 5   (hit → hot → dot → dog → cog)
```

---

# Approach Hints

- BFS on implicit graph: words are nodes; edge if one char differs.
- Preprocess: build adjacency from patterns with wildcard (`h*t`) OR compare each word pair (O(n²) words).
- Bidirectional BFS optimizes large inputs.

---

# Core Idea (Walkthrough)

1. BFS from `beginWord`, level = 1.
2. For each word, try changing each character a-z; if in set and unvisited, enqueue.
3. When `endWord` reached, return current level.

---

# Java Solution

```java
class Solution {
    public int ladderLength(String beginWord, String endWord, List<String> wordList) {
        Set<String> dict = new HashSet<>(wordList);
        if (!dict.contains(endWord)) return 0;

        Queue<String> q = new LinkedList<>();
        q.offer(beginWord);
        Set<String> visited = new HashSet<>();
        visited.add(beginWord);

        int level = 1;
        while (!q.isEmpty()) {
            int size = q.size();
            for (int i = 0; i < size; i++) {
                char[] word = q.poll().toCharArray();
                for (int j = 0; j < word.length; j++) {
                    char orig = word[j];
                    for (char c = 'a'; c <= 'z'; c++) {
                        word[j] = c;
                        String next = new String(word);
                        if (next.equals(endWord)) return level + 1;
                        if (dict.contains(next) && visited.add(next)) {
                            q.offer(next);
                        }
                    }
                    word[j] = orig;
                }
            }
            level++;
        }
        return 0;
    }
}
```

---

# JavaScript Solution

```javascript
function ladderLength(beginWord, endWord, wordList) {
  const dict = new Set(wordList);
  if (!dict.has(endWord)) return 0;

  const q = [beginWord];
  const visited = new Set([beginWord]);
  let level = 1;

  while (q.length) {
    const size = q.length;
    for (let i = 0; i < size; i++) {
      const chars = [...q.shift()];
      for (let j = 0; j < chars.length; j++) {
        const orig = chars[j];
        for (let c = 97; c <= 122; c++) {
          chars[j] = String.fromCharCode(c);
          const next = chars.join("");
          if (next === endWord) return level + 1;
          if (dict.has(next) && !visited.has(next)) {
            visited.add(next);
            q.push(next);
          }
        }
        chars[j] = orig;
      }
    }
    level++;
  }
  return 0;
}
```

---

# Time & Space Complexity

```text
Time:  O(M² × N)  (M = word length, N = word list size)
Space: O(N)
```

---

# 2. Pacific Atlantic Water Flow

## Problem Statement

Heights matrix: water flows from cell to neighbor with equal or lower height. Find cells that can flow to both Pacific (top/left edges) and Atlantic (bottom/right edges).

---

# Example

```text
Input: [[1,2,2,3,5],[3,2,3,4,4],[2,4,5,3,1],[6,7,1,4,5],[5,1,1,2,4]]
Output: [[0,4],[1,3],[1,4],[2,2],[3,0],[3,1],[4,0]]
```

---

# Approach Hints

- Reverse thinking: start DFS/BFS from ocean borders inward (water can flow "up" to higher/equal cells).
- Two visited sets: reachable from Pacific, reachable from Atlantic.
- Intersection of both sets = answer.

---

# Core Idea (Walkthrough)

1. DFS from all Pacific border cells; mark reachable.
2. DFS from all Atlantic border cells; mark reachable.
3. Cells in both sets satisfy condition.

---

# Java Solution

```java
class Solution {
    public List<List<Integer>> pacificAtlantic(int[][] heights) {
        int rows = heights.length, cols = heights[0].length;
        boolean[][] pac = new boolean[rows][cols];
        boolean[][] atl = new boolean[rows][cols];

        for (int c = 0; c < cols; c++) {
            dfs(heights, pac, 0, c);
            dfs(heights, atl, rows - 1, c);
        }
        for (int r = 0; r < rows; r++) {
            dfs(heights, pac, r, 0);
            dfs(heights, atl, r, cols - 1);
        }

        List<List<Integer>> res = new ArrayList<>();
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (pac[r][c] && atl[r][c]) res.add(List.of(r, c));
            }
        }
        return res;
    }

    private void dfs(int[][] h, boolean[][] vis, int r, int c) {
        vis[r][c] = true;
        int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
        for (int[] d : dirs) {
            int nr = r + d[0], nc = c + d[1];
            if (nr >= 0 && nc >= 0 && nr < h.length && nc < h[0].length
                && !vis[nr][nc] && h[nr][nc] >= h[r][c]) {
                dfs(h, vis, nr, nc);
            }
        }
    }
}
```

---

# JavaScript Solution

```javascript
function pacificAtlantic(heights) {
  const rows = heights.length;
  const cols = heights[0].length;
  const pac = Array.from({ length: rows }, () => Array(cols).fill(false));
  const atl = Array.from({ length: rows }, () => Array(cols).fill(false));

  function dfs(vis, r, c) {
    vis[r][c] = true;
    const dirs = [[1, 0], [-1, 0], [0, 1], [0, -1]];
    for (const [dr, dc] of dirs) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nc >= 0 && nr < rows && nc < cols
          && !vis[nr][nc] && heights[nr][nc] >= heights[r][c]) {
        dfs(vis, nr, nc);
      }
    }
  }

  for (let c = 0; c < cols; c++) {
    dfs(pac, 0, c);
    dfs(atl, rows - 1, c);
  }
  for (let r = 0; r < rows; r++) {
    dfs(pac, r, 0);
    dfs(atl, r, cols - 1);
  }

  const res = [];
  for (let r = 0; r < rows; r++) {
    for (let c = 0; c < cols; c++) {
      if (pac[r][c] && atl[r][c]) res.push([r, c]);
    }
  }
  return res;
}
```

---

# Time & Space Complexity

```text
Time:  O(m * n)
Space: O(m * n)
```

---

# 3. Clone Graph

## Problem Statement

Given a reference to a node in a connected undirected graph, return a deep copy. Each node has `val` and list of neighbors.

---

# Example

```text
Input: adjList = [[2,4],[1,3],[2,4],[1,3]]
Output: deep copy with same structure
```

---

# Approach Hints

- BFS or DFS + HashMap: `originalNode -> cloneNode`.
- Create clone when first visiting; wire neighbors from map.

---

# Core Idea (Walkthrough)

1. If node null, return null.
2. Map old → new node.
3. BFS: poll node, for each neighbor create clone if missing, add to neighbor list.

---

# Java Solution

```java
class Solution {
    public Node cloneGraph(Node node) {
        if (node == null) return null;

        Map<Node, Node> map = new HashMap<>();
        Queue<Node> q = new LinkedList<>();
        q.offer(node);
        map.put(node, new Node(node.val));

        while (!q.isEmpty()) {
            Node cur = q.poll();
            for (Node nei : cur.neighbors) {
                if (!map.containsKey(nei)) {
                    map.put(nei, new Node(nei.val));
                    q.offer(nei);
                }
                map.get(cur).neighbors.add(map.get(nei));
            }
        }
        return map.get(node);
    }
}
```

---

# JavaScript Solution

```javascript
function cloneGraph(node) {
  if (!node) return null;

  const map = new Map();
  const q = [node];
  map.set(node, new Node(node.val));

  while (q.length) {
    const cur = q.shift();
    for (const nei of cur.neighbors) {
      if (!map.has(nei)) {
        map.set(nei, new Node(nei.val));
        q.push(nei);
      }
      map.get(cur).neighbors.push(map.get(nei));
    }
  }
  return map.get(node);
}
```

---

# Time & Space Complexity

```text
Time:  O(V + E)
Space: O(V)
```

---

# Pattern Quick Summary

| Problem | Pattern | Time |
|---------|---------|------|
| Word Ladder | BFS shortest path | O(M²N) |
| Pacific Atlantic | Multi-source DFS from borders | O(m×n) |
| Clone Graph | BFS/DFS + HashMap | O(V+E) |

---

*End of Day 9 LeetCode*
