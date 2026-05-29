# Day 29 — Trees: Recursion vs Iteration

**Topics:** Tree node model · Recursion on trees · Iterative stack/queue · When to use each · Complexity

---

## Table of Contents

1. [Tree Fundamentals](#1-tree-fundamentals)
2. [Recursion on Trees](#2-recursion-on-trees)
3. [Iteration with Stack & Queue](#3-iteration-with-stack--queue)
4. [Recursion vs Iteration — Decision Guide](#4-recursion-vs-iteration--decision-guide)
5. [Interview Quick Index](#5-interview-quick-index)

---

## 1. Tree Fundamentals

### Definition

A **binary tree** is a hierarchical structure where each node has at most **two children** (`left`, `right`).

```ts
class TreeNode {
  val: number;
  left: TreeNode | null;
  right: TreeNode | null;
  constructor(val = 0, left: TreeNode | null = null, right: TreeNode | null = null) {
    this.val = val;
    this.left = left;
    this.right = right;
  }
}
```

### Key terms

| Term | Meaning |
|------|---------|
| **Root** | Top node |
| **Leaf** | Node with no children |
| **Height / depth** | Longest path root → leaf (or node → leaf) |
| **Subtree** | Node + all descendants |

### Tree as nested data (frontend mental model)

```ts
type FolderNode = {
  id: string;
  name: string;
  children: FolderNode[];
};

const tree: FolderNode = {
  id: "root",
  name: "src",
  children: [
    { id: "c1", name: "components", children: [] },
    { id: "c2", name: "hooks", children: [{ id: "c3", name: "useStack.ts", children: [] }] },
  ],
};
```

> UI trees (folders, comments, menus) are the same abstraction as binary trees — often **N-ary** (array of children).

---

## 2. Recursion on Trees

### Pattern

Every tree problem follows:

1. **Base case** — `node === null` → return sentinel (`0`, `[]`, `null`)
2. **Recurse** — compute result for `left` and `right`
3. **Combine** — merge child results at current node

```ts
function maxDepth(root: TreeNode | null): number {
  if (!root) return 0;
  return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

### Why recursion fits trees naturally

- Each subtree is a **smaller instance** of the same problem.
- Call stack mirrors **depth-first** descent.
- Code stays short and readable for interviews.

### Tail recursion (conceptual)

```ts
function sumTree(root: TreeNode | null, acc = 0): number {
  if (!root) return acc;
  return sumTree(root.right, sumTree(root.left, acc + root.val));
}
```

> JS engines do **not** guarantee tail-call optimization — treat as clarity pattern, not performance hack.

### Common recursive templates

| Problem | Base | Combine |
|---------|------|---------|
| Max depth | `null → 0` | `1 + max(L, R)` |
| Count nodes | `null → 0` | `1 + count(L) + count(R)` |
| Invert | `null → null` | swap children, recurse |
| Path sum | `null → false` | check leaf + target |

---

## 3. Iteration with Stack & Queue

### DFS with explicit stack

Simulates recursion without call stack growth risk.

```ts
function maxDepthIterative(root: TreeNode | null): number {
  if (!root) return 0;
  const stack: { node: TreeNode; depth: number }[] = [{ node: root, depth: 1 }];
  let max = 0;

  while (stack.length) {
    const { node, depth } = stack.pop()!;
    max = Math.max(max, depth);
    if (node.right) stack.push({ node: node.right, depth: depth + 1 });
    if (node.left) stack.push({ node: node.left, depth: depth + 1 });
  }
  return max;
}
```

### BFS with queue (level order preview)

```ts
function levelOrder(root: TreeNode | null): number[][] {
  if (!root) return [];
  const result: number[][] = [];
  const queue: TreeNode[] = [root];

  while (queue.length) {
    const size = queue.length;
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

### Inorder iterative (stack)

```ts
function inorder(root: TreeNode | null): number[] {
  const out: number[] = [];
  const stack: TreeNode[] = [];
  let cur: TreeNode | null = root;

  while (cur || stack.length) {
    while (cur) {
      stack.push(cur);
      cur = cur.left;
    }
    cur = stack.pop()!;
    out.push(cur.val);
    cur = cur.right;
  }
  return out;
}
```

---

## 4. Recursion vs Iteration — Decision Guide

| Factor | Recursion | Iteration |
|--------|-----------|-----------|
| **Readability** | Excellent for trees | More boilerplate |
| **Stack overflow** | Risk on skewed trees (depth = n) | Controlled explicit stack |
| **Interview default** | Preferred unless asked | Show you know both |
| **Space** | O(h) call stack | O(h) or O(w) explicit structure |
| **Morris / O(1) space** | N/A | Special pointer tricks (Day 34) |

### When interviewers want iteration

- "Solve without recursion"
- Very deep tree / production safety
- Follow-up after recursive solution

### Converting recursion → iteration

1. Identify what the call stack stores (node + extra state).
2. Push root with initial state onto stack/queue.
3. Loop: pop, process, push children with updated state.

---

## 5. Interview Quick Index

| Question | Answer |
|----------|--------|
| Tree node structure? | `val`, `left`, `right` (nullable) |
| Recursive tree template? | base null → recurse children → combine |
| DFS iterative? | Stack, push right then left |
| BFS? | Queue, process level by level |
| Time for full traversal? | O(n) — visit each node once |
| Space recursive DFS? | O(h) — height; O(n) worst skewed |
| Recursion vs iteration? | Same logic; iteration avoids call stack limit |

---

## Day 29 Cheat Sheet

```
Tree        → hierarchical; each node ≤ 2 children
Recursion   → null base + left/right + combine
DFS stack   → push nodes; pop to visit
BFS queue   → FIFO; level-by-level
Complexity  → O(n) time; O(h) space typical
```

---

*End of Day 29 concepts*
