# Day 31 — BFS Level Order Traversal

**Topics:** Queue-based BFS · Level-by-level processing · Width vs depth · Zigzag variant

---

## Table of Contents

1. [BFS on Trees](#1-bfs-on-trees)
2. [Level Order Template](#2-level-order-template)
3. [Why Process by Level Size](#3-why-process-by-level-size)
4. [BFS vs DFS on Trees](#4-bfs-vs-dfs-on-trees)
5. [Common Variants](#5-common-variants)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. BFS on Trees

**Breadth-First Search** visits nodes **level by level** using a **queue** (FIFO).

```
        3          ← level 0
       / \
      9  20        ← level 1
        /  \
       15   7      ← level 2

BFS order: 3 → 9 → 20 → 15 → 7
Level groups: [[3], [9,20], [15,7]]
```

---

## 2. Level Order Template

```ts
function levelOrder(root: TreeNode | null): number[][] {
  if (!root) return [];
  const result: number[][] = [];
  const queue: TreeNode[] = [root];

  while (queue.length) {
    const size = queue.length;       // nodes at current level
    const level: number[] = [];

    for (let i = 0; i < size; i++) {
      const node = queue.shift()!;
      level.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
    result.push(level);
  }
  return result;
}
```

**Critical:** Inner loop uses `size` captured **before** processing — otherwise you mix levels.

---

## 3. Why Process by Level Size

Without fixed `size`, one iteration processes one node but may enqueue next level — hard to group.

```ts
// WRONG for grouping — processes one node per outer loop
while (queue.length) {
  const node = queue.shift()!;
  // ...
}

// RIGHT — batch entire level
const size = queue.length;
for (let i = 0; i < size; i++) { /* ... */ }
```

---

## 4. BFS vs DFS on Trees

| | BFS (level order) | DFS (pre/in/post) |
|--|-------------------|-------------------|
| Structure | Queue | Stack / recursion |
| Finds | Shortest path (unweighted) | Deep paths first |
| Space | O(w) max width | O(h) height |
| UI analog | Grid layout by depth | Drill-down navigation |

**Skewed tree:** BFS space O(1); DFS space O(n).

---

## 5. Common Variants

### Right side view — last node per level

```ts
function rightSideView(root: TreeNode | null): number[] {
  if (!root) return [];
  const res: number[] = [], queue = [root];
  while (queue.length) {
    const size = queue.length;
    for (let i = 0; i < size; i++) {
      const node = queue.shift()!;
      if (i === size - 1) res.push(node.val);
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
  return res;
}
```

### Zigzag — alternate push direction per level

```ts
if (depth % 2 === 1) level.reverse();
```

### Level with nulls (LC 513) — track last non-null at deepest level

---

## 6. Interview Quick Index

| Question | Answer |
|----------|--------|
| BFS data structure? | Queue |
| Level grouping trick? | `size = queue.length` before inner loop |
| BFS space? | O(w) max width |
| Shortest path in tree? | BFS from source |
| Right side view? | Last node in each level batch |

---

## Day 31 Cheat Sheet

```
BFS       → queue, FIFO, level by level
Template  → while queue: size = len; for size: shift + enqueue children
Variants  → zigzag reverse, right = last per level
Space     → O(max width)
```

---

*End of Day 31 concepts*
