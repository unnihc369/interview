# Day 30 — DFS Traversals (Pre / In / Post)

**Topics:** Preorder · Inorder · Postorder · Iterative stack variants · When each order matters

---

## Table of Contents

1. [Three DFS Orders](#1-three-dfs-orders)
2. [Preorder — Root First](#2-preorder--root-first)
3. [Inorder — Root Middle](#3-inorder--root-middle)
4. [Postorder — Root Last](#4-postorder--root-last)
5. [Iterative Implementations](#5-iterative-implementations)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Three DFS Orders

All visit every node once — difference is **when you process the root**.

| Order | Sequence | Mnemonic |
|-------|----------|----------|
| **Preorder** | Root → Left → Right | "Copy tree" |
| **Inorder** | Left → Root → Right | "BST sorted" |
| **Postorder** | Left → Right → Root | "Delete bottom-up" |

---

## 2. Preorder — Root First

```ts
function preorder(root: TreeNode | null, out: number[] = []): number[] {
  if (!root) return out;
  out.push(root.val);           // visit root FIRST
  preorder(root.left, out);
  preorder(root.right, out);
  return out;
}
```

**Frontend analog:** Render parent folder before children (breadcrumb-first navigation).

---

## 3. Inorder — Root Middle

```ts
function inorder(root: TreeNode | null, out: number[] = []): number[] {
  if (!root) return out;
  inorder(root.left, out);
  out.push(root.val);           // visit root MIDDLE
  inorder(root.right, out);
  return out;
}
```

**BST property:** Inorder traversal of a valid BST → **ascending sorted** values.

---

## 4. Postorder — Root Last

```ts
function postorder(root: TreeNode | null, out: number[] = []): number[] {
  if (!root) return out;
  postorder(root.left, out);
  postorder(root.right, out);
  out.push(root.val);           // visit root LAST
  return out;
}
```

**Use case:** Compute folder sizes — need child totals before parent.

---

## 5. Iterative Implementations

### Preorder (stack — push right before left)

```ts
function preorderIter(root: TreeNode | null): number[] {
  if (!root) return [];
  const stack = [root], res: number[] = [];
  while (stack.length) {
    const node = stack.pop()!;
    res.push(node.val);
    if (node.right) stack.push(node.right);
    if (node.left) stack.push(node.left);
  }
  return res;
}
```

### Inorder (stack — go left until null)

```ts
function inorderIter(root: TreeNode | null): number[] {
  const res: number[] = [];
  const stack: TreeNode[] = [];
  let cur: TreeNode | null = root;
  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.left; }
    cur = stack.pop()!;
    res.push(cur.val);
    cur = cur.right;
  }
  return res;
}
```

### Postorder (stack with visited flag)

```ts
function postorderIter(root: TreeNode | null): number[] {
  if (!root) return [];
  const res: number[] = [];
  const stack: [TreeNode, boolean][] = [[root, false]];
  while (stack.length) {
    const [node, visited] = stack.pop()!;
    if (visited) res.push(node.val);
    else {
      stack.push([node, true]);
      if (node.right) stack.push([node.right, false]);
      if (node.left) stack.push([node.left, false]);
    }
  }
  return res;
}
```

---

## 6. Interview Quick Index

| Question | Answer |
|----------|--------|
| Preorder use? | Serialize, clone, prefix expr |
| Inorder on BST? | Sorted ascending |
| Postorder use? | Delete tree, postfix, aggregate from leaves |
| All traversals time? | O(n) |
| Space? | O(h) stack |

---

## Day 30 Cheat Sheet

```
Preorder  → root, left, right
Inorder   → left, root, right (BST = sorted)
Postorder → left, right, root
Iterative → stack; inorder = left chain + pop
```

---

*End of Day 30 concepts*
