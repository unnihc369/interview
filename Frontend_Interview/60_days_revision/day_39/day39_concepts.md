# Day 39 — Kth Smallest / Largest in BST

**Topics:** Inorder kth element · Order statistics · Successor · Closest value

---

## Table of Contents

1. [Kth Smallest — Inorder](#1-kth-smallest--inorder)
2. [Kth Largest — Reverse Inorder](#2-kth-largest--reverse-inorder)
3. [Inorder Successor](#3-inorder-successor)
4. [Closest Value in BST](#4-closest-value-in-bst)
5. [Leaderboard Analogy](#5-leaderboard-analogy)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Kth Smallest — Inorder

Kth smallest = kth node in **inorder** traversal (1-indexed).

```ts
function kthSmallest(root: TreeNode | null, k: number): number {
  const stack: TreeNode[] = [];
  let cur: TreeNode | null = root;
  let count = 0;

  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.left; }
    cur = stack.pop()!;
    count++;
    if (count === k) return cur.val;
    cur = cur.right;
  }
  return -1;
}
```

**Augmented BST:** Store subtree size at each node → O(log n) per query (advanced).

---

## 2. Kth Largest — Reverse Inorder

Visit right → root → left; decrement k.

```ts
function kthLargest(root: TreeNode | null, k: number): number {
  const stack: TreeNode[] = [];
  let cur: TreeNode | null = root;
  let count = 0;

  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.right; }
    cur = stack.pop()!;
    count++;
    if (count === k) return cur.val;
    cur = cur.left;
  }
  return -1;
}
```

---

## 3. Inorder Successor

**Node with parent pointers (LC 285):**

```ts
function inorderSuccessor(root: TreeNode, p: TreeNode): TreeNode | null {
  let succ: TreeNode | null = null;
  let cur: TreeNode | null = root;
  while (cur) {
    if (p.val < cur.val) { succ = cur; cur = cur.left; }
    else cur = cur.right;
  }
  return succ;
}
```

If no parent: inorder traversal until past p, or BST search pattern above.

---

## 4. Closest Value in BST

LC 272: Find node value closest to `target`.

```ts
function closestValue(root: TreeNode | null, target: number): number {
  let cur = root!;
  let closest = cur.val;

  while (cur) {
    if (Math.abs(cur.val - target) < Math.abs(closest - target)) {
      closest = cur.val;
    }
    cur = target < cur.val ? cur.left! : cur.right!;
  }
  return closest;
}
```

Walk toward target; track best seen (like binary search on values).

---

## 5. Leaderboard Analogy

Scores stored in BST by rank:
- **Insert score** → `insertIntoBST`
- **Kth rank** → kth smallest (or largest for top-k)
- **User rank** → count nodes smaller than user's score (inorder count)

```ts
function getRank(root: TreeNode | null, score: number): number {
  let rank = 0;
  const stack: TreeNode[] = [];
  let cur = root;
  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.left; }
    cur = stack.pop()!;
    rank++;
    if (cur.val === score) return rank;
    cur = cur.right;
  }
  return -1;
}
```

---

## 6. Interview Quick Index

| Question | Answer |
|----------|--------|
| Kth smallest? | Inorder, stop at k |
| Kth largest? | Reverse inorder |
| Successor? | BST search: go left if p < cur |
| Closest? | Walk toward target, track min diff |
| Time? | O(h + k) iterative / O(n) worst |

---

## Day 39 Cheat Sheet

```
Kth small → inorder left-first, count k
Kth large → reverse inorder
Successor → smallest in right subtree OR ancestor
Closest   → BST walk + track best
```

---

*End of Day 39 concepts*
