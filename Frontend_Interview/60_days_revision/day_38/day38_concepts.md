# Day 38 — Validate BST & Inorder

**Topics:** Bounds validation · Strictly increasing inorder · Recover BST · BST Iterator

---

## Table of Contents

1. [Two Validation Approaches](#1-two-validation-approaches)
2. [Inorder Must Be Strictly Increasing](#2-inorder-must-be-strictly-increasing)
3. [Recover BST (Two Swapped Nodes)](#3-recover-bst-two-swapped-nodes)
4. [BST Iterator](#4-bst-iterator)
5. [Form Validation Tree Analogy](#5-form-validation-tree-analogy)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Two Validation Approaches

### Min/Max bounds (DFS)

```ts
function isValidBST(root: TreeNode | null, min = -Infinity, max = Infinity): boolean {
  if (!root) return true;
  if (root.val <= min || root.val >= max) return false;
  return isValidBST(root.left, min, root.val) &&
         isValidBST(root.right, root.val, max);
}
```

### Inorder scan

Track `prev` — each value must be **strictly greater** than previous.

---

## 2. Inorder Must Be Strictly Increasing

```ts
function isValidBSTInorder(root: TreeNode | null): boolean {
  let prev = -Infinity;
  const stack: TreeNode[] = [];
  let cur: TreeNode | null = root;

  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.left; }
    cur = stack.pop()!;
    if (cur.val <= prev) return false;
    prev = cur.val;
    cur = cur.right;
  }
  return true;
}
```

**Interview trap:** `[2, 2, 2]` — use `<=` not `<` for left bound depending on duplicate policy.

---

## 3. Recover BST (Two Swapped Nodes)

Exactly **two nodes** swapped → inorder has two "violations".

```ts
function recoverTree(root: TreeNode | null): void {
  let first: TreeNode | null = null;
  let second: TreeNode | null = null;
  let prev: TreeNode | null = null;

  const stack: TreeNode[] = [];
  let cur: TreeNode | null = root;

  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.left; }
    cur = stack.pop()!;
    if (prev && cur.val < prev.val) {
      if (!first) first = prev;
      second = cur;
    }
    prev = cur;
    cur = cur.right;
  }
  if (first && second) [first.val, second.val] = [second.val, first.val];
}
```

---

## 4. BST Iterator

LC 173: `next()` and `hasNext()` in O(h) amortized.

```ts
class BSTIterator {
  private stack: TreeNode[] = [];

  constructor(root: TreeNode | null) {
    this.pushLeft(root);
  }

  next(): number {
    const node = this.stack.pop()!;
    this.pushLeft(node.right);
    return node.val;
  }

  hasNext(): boolean {
    return this.stack.length > 0;
  }

  private pushLeft(node: TreeNode | null) {
    while (node) {
      this.stack.push(node);
      node = node.left;
    }
  }
}
```

Same as iterative inorder — lazy evaluation.

---

## 5. Form Validation Tree Analogy

Nested form fields = tree; validate **parent constraints before children**:

```ts
type FieldNode = {
  name: string;
  value: unknown;
  rules: ((v: unknown) => string | null)[];
  children: FieldNode[];
};

function validateField(node: FieldNode, parentValid = true): Record<string, string> {
  const errors: Record<string, string> = {};
  if (!parentValid) return errors;

  for (const rule of node.rules) {
    const err = rule(node.value);
    if (err) { errors[node.name] = err; return errors; }
  }
  for (const child of node.children) {
    Object.assign(errors, validateField(child, true));
  }
  return errors;
}
```

Like BST bounds: invalid parent → skip/delegate child validation.

---

## 6. Interview Quick Index

| Question | Answer |
|----------|--------|
| Validate BST? | Bounds or inorder strictly increasing |
| Why not just check children? | Deep subtree can violate |
| Recover BST? | Find two inorder violations, swap vals |
| Iterator? | Stack of left spine |
| `next()` complexity? | Amortized O(1), worst O(h) |

---

## Day 38 Cheat Sheet

```
Validate  → min/max bounds OR inorder prev
Recover   → two inorder drops → swap nodes
Iterator  → stack + pushLeft chain
Duplicates → define policy (strict < or <=)
```

---

*End of Day 38 concepts*
