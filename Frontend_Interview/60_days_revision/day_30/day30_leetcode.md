# Day 30 — LeetCode (DFS Traversals)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 144 | [Binary Tree Preorder Traversal](#1-binary-tree-preorder-traversal-144) | Easy | DFS / stack |
| 94 | [Binary Tree Inorder Traversal](#2-binary-tree-inorder-traversal-94) | Easy | DFS / stack |
| 145 | [Binary Tree Postorder Traversal](#3-binary-tree-postorder-traversal-145) | Easy | DFS / stack |

---

## 1. Binary Tree Preorder Traversal (144)

### Problem

Return node values in **preorder** (root → left → right).

### JavaScript — Recursive

```js
function preorderTraversal(root) {
  const res = [];
  function dfs(node) {
    if (!node) return;
    res.push(node.val);
    dfs(node.left);
    dfs(node.right);
  }
  dfs(root);
  return res;
}
```

### JavaScript — Iterative

```js
function preorderTraversal(root) {
  if (!root) return [];
  const stack = [root], res = [];
  while (stack.length) {
    const node = stack.pop();
    res.push(node.val);
    if (node.right) stack.push(node.right);
    if (node.left) stack.push(node.left);
  }
  return res;
}
```

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## 2. Binary Tree Inorder Traversal (94)

### Problem

Return values in **inorder** (left → root → right).

### JavaScript — Iterative (interview favorite)

```js
function inorderTraversal(root) {
  const res = [], stack = [];
  let cur = root;
  while (cur || stack.length) {
    while (cur) { stack.push(cur); cur = cur.left; }
    cur = stack.pop();
    res.push(cur.val);
    cur = cur.right;
  }
  return res;
}
```

### Morris traversal preview (O(1) space — Day 34)

Thread left subtree to avoid stack — advanced follow-up.

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## 3. Binary Tree Postorder Traversal (145)

### Problem

Return values in **postorder** (left → right → root).

### JavaScript — Iterative with visited flag

```js
function postorderTraversal(root) {
  if (!root) return [];
  const res = [], stack = [[root, false]];
  while (stack.length) {
    const [node, visited] = stack.pop();
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

### Reverse preorder trick (left-right-root)

Modify preorder to visit **right before left**, then reverse result → postorder.

| Time | Space |
|------|-------|
| O(n) | O(h) |

---

## Pattern Cheat Sheet

| Traversal | Recursive | Iterative key |
|-----------|-----------|---------------|
| Preorder | Visit before children | Stack; push right first |
| Inorder | Visit between children | Left chain + pop |
| Postorder | Visit after children | Visited flag or reverse trick |

---

*End of Day 30 LeetCode*
