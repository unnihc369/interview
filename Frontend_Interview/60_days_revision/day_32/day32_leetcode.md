# Day 32 — LeetCode (Serialization)

| # | Problem | Difficulty | Pattern |
|---|---------|------------|---------|
| 297 | [Serialize and Deserialize Binary Tree](#1-serialize-and-deserialize-binary-tree-297) | Hard | Preorder + null |
| 449 | [Serialize and Deserialize N-ary Tree](#2-serialize-and-deserialize-n-ary-tree-449) | Hard | Preorder + count |
| 652 | [Find Duplicate Subtrees](#3-find-duplicate-subtrees-652) | Medium | Serialize + hash |

---

## 1. Serialize and Deserialize Binary Tree (297)

### Problem

Design codec to convert tree ↔ string without ambiguity.

### JavaScript

```js
function serialize(root) {
  const parts = [];
  (function dfs(node) {
    if (!node) { parts.push("null"); return; }
    parts.push(String(node.val));
    dfs(node.left);
    dfs(node.right);
  })(root);
  return parts.join(",");
}

function deserialize(data) {
  const tokens = data.split(",");
  let i = 0;
  function build() {
    const val = tokens[i++];
    if (val === "null") return null;
    const node = new TreeNode(+val);
    node.left = build();
    node.right = build();
    return node;
  }
  return build();
}
```

| Time | Space |
|------|-------|
| O(n) | O(n) |

---

## 2. Serialize and Deserialize N-ary Tree (449)

### Problem

Encode N-ary tree to string and decode back.

### JavaScript (val + child count)

```js
function serialize(root) {
  if (!root) return "";
  let res = root.val + "," + root.children.length;
  for (const c of root.children) res += "," + serialize(c);
  return res;
}

function deserialize(data) {
  if (!data) return null;
  const tokens = data.split(",").filter(Boolean);
  let i = 0;
  function build() {
    const val = +tokens[i++];
    const count = +tokens[i++];
    const node = new Node(val);
    for (let j = 0; j < count; j++) node.children.push(build());
    return node;
  }
  return build();
}
```

---

## 3. Find Duplicate Subtrees (652)

### Problem

Return all **duplicate subtrees** (same structure + values). Each duplicate group reported once.

### Hint

Serialize each subtree (postorder string like `"L,R,root"`), hash in Map, collect when count hits 2.

### JavaScript

```js
function findDuplicateSubtrees(root) {
  const map = new Map();
  const res = [];

  function dfs(node) {
    if (!node) return "#";
    const key = `${dfs(node.left)},${dfs(node.right)},${node.val}`;
    map.set(key, (map.get(key) || 0) + 1);
    if (map.get(key) === 2) res.push(node);
    return key;
  }
  dfs(root);
  return res;
}
```

| Time | Space |
|------|-------|
| O(n) | O(n) |

---

## Pattern Cheat Sheet

| Problem | Encoding |
|---------|----------|
| **297** | Preorder + null |
| **449** | val, childCount, children... |
| **652** | Postorder string as subtree fingerprint |

---

*End of Day 32 LeetCode*
