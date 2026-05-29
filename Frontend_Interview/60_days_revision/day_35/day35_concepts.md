# Day 35 — Trees Week Revision

**Topics:** Recursion · DFS/BFS · Serialization · LCA · Morris · Path problems · Complexity recap

---

## Table of Contents

1. [Week 5 Concept Map](#1-week-5-concept-map)
2. [Traversal Decision Tree](#2-traversal-decision-tree)
3. [Pattern Templates](#3-pattern-templates)
4. [Complexity Reference](#4-complexity-reference)
5. [Common Mistakes](#5-common-mistakes)
6. [Mock Interview Prompts](#6-mock-interview-prompts)
7. [Interview Quick Index](#7-interview-quick-index)

---

## 1. Week 5 Concept Map

| Day | Core topic | Key LC |
|-----|------------|--------|
| 29 | Recursion vs iteration | 104, 226, 124 |
| 30 | Pre/in/post DFS | 144, 94, 145 |
| 31 | BFS level order | 102, 107, 199 |
| 32 | Serialization | 297, 449, 652 |
| 33 | LCA | 236, 235, 1644 |
| 34 | Morris / advanced paths | 114, 543, 687 |

---

## 2. Traversal Decision Tree

```
Need level grouping?     → BFS (queue + level size)
Need sorted BST order?   → Inorder
Clone / prefix order?    → Preorder
Bottom-up aggregation?   → Postorder
O(1) space inorder?      → Morris
Shortest path (unweighted)? → BFS
```

---

## 3. Pattern Templates

### Recursive DFS base

```ts
function solve(root: TreeNode | null): T {
  if (!root) return BASE;
  const left = solve(root.left);
  const right = solve(root.right);
  return COMBINE(root, left, right);
}
```

### BFS levels

```ts
while (queue.length) {
  const size = queue.length;
  for (let i = 0; i < size; i++) { /* process level */ }
}
```

### LCA

```ts
if (!root || root === p || root === q) return root;
const L = lca(root.left, p, q), R = lca(root.right, p, q);
return L && R ? root : L ?? R;
```

### Global max path (124 / 543 / 687)

Return **single arm** to parent; update **global** with combined arms at node.

---

## 4. Complexity Reference

| Operation | Time | Space |
|-----------|------|-------|
| Any traversal | O(n) | O(h) DFS / O(w) BFS |
| Serialize/deserialize | O(n) | O(n) |
| LCA general | O(n) | O(h) |
| LCA BST | O(h) | O(1) |
| Morris inorder | O(n) | O(1) |

---

## 5. Common Mistakes

| Mistake | Fix |
|---------|-----|
| Mix BFS levels without `size` | Capture `queue.length` first |
| Return arch sum in 124 | Return one arm only |
| Forget `Math.max(child, 0)` in path sum | Ignore negative arms |
| Deserialize without null markers | Preorder needs `"null"` |
| BST LCA using general algorithm | Use value comparison |

---

## 6. Mock Interview Prompts

1. Implement file explorer with expand/collapse — discuss recursive vs flat render.
2. Serialize tree to JSON and restore from localStorage.
3. Find LCA of two selected nodes in org chart UI.
4. Morris inorder — explain threading and restoration.
5. **Timed LC mock:** 102, 236, 124 (45 min).

---

## 7. Interview Quick Index

| Pattern | When |
|---------|------|
| Recursion | Default tree solution |
| Stack DFS | Avoid call stack limit |
| BFS | Levels, right view, shortest path |
| Serialize | Storage, duplicate subtrees |
| LCA | Shared ancestor, DOM, threads |
| Global + return | Path sum, diameter, univ path |

---

## Day 35 Cheat Sheet

```
Trees week → recursion, BFS/DFS, serialize, LCA, Morris
Pick traversal by output shape
Path problems → global max + return one arm
Mock: 102 + 236 + 124 under 45 min
```

---

*End of Day 35 concepts*
