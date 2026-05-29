# Day 34 — Morris Traversal & Advanced Tree Patterns

**Topics:** Morris inorder O(1) space · Threaded tree · Subtree selection · Diameter / univ path preview

---

## Table of Contents

1. [Morris Traversal](#1-morris-traversal)
2. [Why O(1) Extra Space](#2-why-o1-extra-space)
3. [Flatten Tree Concept](#3-flatten-tree-concept)
4. [Diameter & Longest Univ Path](#4-diameter--longest-univ-path)
5. [Interview Quick Index](#5-interview-quick-index)

---

## 1. Morris Traversal

Inorder without stack/recursion by **threading** rightmost node of left subtree to current.

```ts
function morrisInorder(root: TreeNode | null): number[] {
  const res: number[] = [];
  let cur: TreeNode | null = root;

  while (cur) {
    if (!cur.left) {
      res.push(cur.val);
      cur = cur.right;
    } else {
      let pred = cur.left;
      while (pred.right && pred.right !== cur) pred = pred.right;

      if (!pred.right) {
        pred.right = cur;       // thread
        cur = cur.left;
      } else {
        pred.right = null;      // remove thread
        res.push(cur.val);
        cur = cur.right;
      }
    }
  }
  return res;
}
```

---

## 2. Why O(1) Extra Space

No auxiliary stack — uses **null right pointers** temporarily. Each edge traversed at most twice → O(n) time.

**Trade-off:** Mutates tree briefly (restored after). Not for concurrent reads.

---

## 3. Flatten Tree Concept

LC 114: Flatten to linked list in **preorder** (right = next).

```ts
function flatten(root: TreeNode | null): void {
  let cur: TreeNode | null = root;
  while (cur) {
    if (cur.left) {
      let pred = cur.left;
      while (pred.right) pred = pred.right;
      pred.right = cur.right;
      cur.right = cur.left;
      cur.left = null;
    }
    cur = cur.right;
  }
}
```

Same threading idea as Morris — move left subtree to right chain.

---

## 4. Diameter & Longest Univ Path

### Diameter (LC 543)

Longest path between any two nodes (may not pass root).

```ts
function diameterOfBinaryTree(root: TreeNode | null): number {
  let max = 0;
  function depth(node: TreeNode | null): number {
    if (!node) return 0;
    const L = depth(node.left), R = depth(node.right);
    max = Math.max(max, L + R);
    return 1 + Math.max(L, R);
  }
  depth(root);
  return max;
}
```

### Longest Univ Path (LC 687)

Longest path where all nodes share same value.

```ts
function longestUnivPath(root: TreeNode | null): number {
  let max = 0;
  function dfs(node: TreeNode | null): number {
    if (!node) return 0;
    let left = dfs(node.left), right = dfs(node.right);
    left = node.left?.val === node.val ? left + 1 : 0;
    right = node.right?.val === node.val ? right + 1 : 0;
    max = Math.max(max, left + right);
    return Math.max(left, right);
  }
  dfs(root);
  return max;
}
```

---

## 5. Interview Quick Index

| Question | Answer |
|----------|--------|
| Morris traversal? | Thread left subtree; O(1) space inorder |
| Flatten tree? | Preorder linked list via right pointers |
| Diameter? | At each node: L depth + R depth |
| Univ path? | Extend arm only if child val matches |

---

## Day 34 Cheat Sheet

```
Morris    → thread predecessor; inorder O(1) space
Flatten   → left subtree → right chain
Diameter  → max(Ldepth + Rdepth) at each node
Univ path → same val arms only
```

---

*End of Day 34 concepts*
