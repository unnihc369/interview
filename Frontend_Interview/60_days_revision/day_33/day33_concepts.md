# Day 33 — Lowest Common Ancestor (LCA)

**Topics:** LCA definition · Recursive post-order · BST shortcut · Parent pointers (LC 1644)

---

## Table of Contents

1. [What is LCA](#1-what-is-lca)
2. [Binary Tree LCA — Recursive](#2-binary-tree-lca--recursive)
3. [BST LCA — Use Ordering](#3-bst-lca--use-ordering)
4. [LCA with Parent Pointers](#4-lca-with-parent-pointers)
5. [Frontend Analogies](#5-frontend-analogies)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. What is LCA

**Lowest Common Ancestor** of nodes `p` and `q` = deepest node that has **both** as descendants (a node can be ancestor of itself).

```
        3
       / \
      5   1
     / \
    6   2
       / \
      7   4

LCA(5, 1) = 3
LCA(5, 4) = 5
```

---

## 2. Binary Tree LCA — Recursive

Post-order: if current is p or q, or found in both subtrees → current is LCA.

```ts
function lowestCommonAncestor(
  root: TreeNode | null,
  p: TreeNode,
  q: TreeNode
): TreeNode | null {
  if (!root || root === p || root === q) return root;

  const left = lowestCommonAncestor(root.left, p, q);
  const right = lowestCommonAncestor(root.right, p, q);

  if (left && right) return root;
  return left ?? right;
}
```

**Intuition:** First node where paths to p and q **diverge and reunite**.

---

## 3. BST LCA — Use Ordering

```ts
function lowestCommonAncestorBST(root: TreeNode, p: TreeNode, q: TreeNode): TreeNode {
  let cur: TreeNode | null = root;
  while (cur) {
    if (p.val < cur.val && q.val < cur.val) cur = cur.left!;
    else if (p.val > cur.val && q.val > cur.val) cur = cur.right!;
    else return cur;
  }
  return root;
}
```

O(h) time, O(1) space — no recursion needed.

---

## 4. LCA with Parent Pointers

LeetCode 1644: nodes have `parent` reference.

**Two-pointer technique** (like linked list intersection):

```ts
function lowestCommonAncestorWithParent(p: Node, q: Node): Node {
  const ancestors = new Set<Node>();
  let a: Node | null = p;
  while (a) { ancestors.add(a); a = a.parent; }

  let b: Node | null = q;
  while (b) {
    if (ancestors.has(b)) return b;
    b = b.parent;
  }
  throw new Error("No LCA");
}
```

Alternative: walk both to root, align depths, then step together.

---

## 5. Frontend Analogies

| Domain | LCA meaning |
|--------|-------------|
| **Folder tree** | Deepest shared parent folder of two files |
| **DOM** | `commonAncestor` of two elements |
| **Message thread** | Fork point where two replies share ancestry |
| **Org chart** | Lowest manager overseeing both employees |

```ts
function findFolderLCA(root: Folder, idA: string, idB: string): Folder | null {
  if (!root) return null;
  if (root.id === idA || root.id === idB) return root;
  const left = findFolderLCA(root.left, idA, idB);
  const right = findFolderLCA(root.right, idA, idB);
  if (left && right) return root;
  return left ?? right;
}
```

---

## 6. Interview Quick Index

| Question | Answer |
|----------|--------|
| General LCA? | Post-order; both subtrees non-null → root |
| BST LCA? | Compare values, walk left/right |
| Parent pointers? | Hash ancestors of p; walk q upward |
| Time general? | O(n) |
| Time BST? | O(h) |

---

## Day 33 Cheat Sheet

```
General LCA → post-order; p/q at split = answer
BST LCA     → while both on same side, move; else stop
Parent ptr  → Set ancestors or two-pointer to root
```

---

*End of Day 33 concepts*
