# Day 41 — Balanced BST & AVL Basics

**Topics:** Balance factor · Height-balanced tree · Rebalance on insert · Deepest leaves sum

---

## Table of Contents

1. [What is a Balanced BST](#1-what-is-a-balanced-balance)
2. [Check Balance — LC 110](#2-check-balance--lc-110)
3. [AVL Rotations (Conceptual)](#3-avl-rotations-conceptual)
4. [Balance BST — LC 1382](#4-balance-bst--lc-1382)
5. [Deepest Leaves Sum — LC 1302](#5-deepest-leaves-sum--lc-1302)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. What is a Balanced BST

**Height-balanced:** For every node, `|height(left) - height(right)| ≤ 1`.

| Structure | Height | Search |
|-----------|--------|--------|
| Balanced | O(log n) | O(log n) |
| Skewed (linked list) | O(n) | O(n) |

**AVL tree:** Self-balancing BST — rebalance via **rotations** after insert/delete.

---

## 2. Check Balance — LC 110

```ts
function isBalanced(root: TreeNode | null): boolean {
  function height(node: TreeNode | null): number {
    if (!node) return 0;
    const L = height(node.left);
    if (L === -1) return -1;
    const R = height(node.right);
    if (R === -1) return -1;
    if (Math.abs(L - R) > 1) return -1;
    return 1 + Math.max(L, R);
  }
  return height(root) !== -1;
}
```

Return `-1` as sentinel for "unbalanced" — single pass O(n).

---

## 3. AVL Rotations (Conceptual)

| Case | Fix |
|------|-----|
| **LL** (left-left heavy) | Right rotation |
| **RR** (right-right heavy) | Left rotation |
| **LR** | Left rotate child, then right rotate |
| **RL** | Right rotate child, then left rotate |

```ts
function rotateRight(y: AVLNode): AVLNode {
  const x = y.left!;
  y.left = x.right;
  x.right = y;
  // update heights
  return x;
}
```

Interview: explain **why** rotations preserve BST order while fixing height.

---

## 4. Balance BST — LC 1382

Given BST, return **balanced** BST with same values.

**Approach:** Inorder → sorted array → `sortedArrayToBST`.

```ts
function balanceBST(root: TreeNode | null): TreeNode | null {
  const nums: number[] = [];
  (function inorder(n: TreeNode | null) {
    if (!n) return;
    inorder(n.left);
    nums.push(n.val);
    inorder(n.right);
  })(root);
  return sortedArrayToBST(nums);
}
```

---

## 5. Deepest Leaves Sum — LC 1302

Sum of nodes at **maximum depth** — BFS until last level.

```ts
function deepestLeavesSum(root: TreeNode | null): number {
  if (!root) return 0;
  let queue = [root];
  let sum = root.val;

  while (queue.length) {
    const next: TreeNode[] = [];
    sum = 0;
    for (const node of queue) {
      if (node.left) { next.push(node.left); sum += node.left.val; }
      if (node.right) { next.push(node.right); sum += node.right.val; }
    }
    if (next.length) queue = next;
  }
  return sum;
}
```

---

## 6. Interview Quick Index

| Question | Answer |
|----------|--------|
| Balanced definition? | \|h(left) - h(right)\| ≤ 1 at every node |
| Check balanced O(n)? | Return -1 sentinel from height DFS |
| AVL fix? | Rotations after insert/delete |
| Rebalance BST? | Inorder + mid rebuild |
| Deepest leaves? | BFS, reset sum each level |

---

## Day 41 Cheat Sheet

```
Balanced  → height diff ≤ 1 everywhere
Check     → DFS height with -1 sentinel
AVL       → LL/RR/LR/RL rotations
Rebalance → inorder → sortedArrayToBST
Deepest   → BFS last level sum
```

---

*End of Day 41 concepts*
