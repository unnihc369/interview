# Day 32 — Tree Serialization (JSON)

**Topics:** Preorder serialization · Null markers · JSON round-trip · N-ary encoding · localStorage persistence

---

## Table of Contents

1. [Why Serialize Trees](#1-why-serialize-trees)
2. [Preorder with Null Markers](#2-preorder-with-null-markers)
3. [JSON Format for Frontend](#3-json-format-for-frontend)
4. [Deserialize Algorithm](#4-deserialize-algorithm)
5. [N-ary Tree Encoding](#5-n-ary-tree-encoding)
6. [Interview Quick Index](#6-interview-quick-index)

---

## 1. Why Serialize Trees

Convert in-memory tree → **string/JSON** for:

- `localStorage` / API transport
- Undo/redo snapshots
- Diff/sync between client and server

---

## 2. Preorder with Null Markers

LeetCode 297 classic: `"1,2,null,null,3,4,null,null,5,null,null"`

```ts
function serialize(root: TreeNode | null): string {
  const parts: string[] = [];
  function dfs(node: TreeNode | null) {
    if (!node) { parts.push("null"); return; }
    parts.push(String(node.val));
    dfs(node.left);
    dfs(node.right);
  }
  dfs(root);
  return parts.join(",");
}
```

**Why preorder + nulls?** Uniquely reconstructs any binary tree.

---

## 3. JSON Format for Frontend

Human-readable nested JSON (like API responses):

```ts
type TreeJSON = { val: number; left: TreeJSON | null; right: TreeJSON | null };

function toJSON(root: TreeNode | null): TreeJSON | null {
  if (!root) return null;
  return {
    val: root.val,
    left: toJSON(root.left),
    right: toJSON(root.right),
  };
}

function fromJSON(json: TreeJSON | null): TreeNode | null {
  if (!json) return null;
  const node = new TreeNode(json.val);
  node.left = fromJSON(json.left);
  node.right = fromJSON(json.right);
  return node;
}
```

**localStorage:**

```ts
localStorage.setItem("tree", JSON.stringify(toJSON(root)));
const restored = fromJSON(JSON.parse(localStorage.getItem("tree")!));
```

---

## 4. Deserialize Algorithm

```ts
function deserialize(data: string): TreeNode | null {
  const tokens = data.split(",");
  let i = 0;

  function build(): TreeNode | null {
    const val = tokens[i++];
    if (val === "null") return null;
    const node = new TreeNode(Number(val));
    node.left = build();
    node.right = build();
    return node;
  }
  return build();
}
```

Uses **global index** or queue — preorder consume order matches serialize order.

---

## 5. N-ary Tree Encoding

LeetCode 449: `"1,3,2,4,null,null,null,5,null,null"`

```ts
function serializeNary(root: NaryNode | null): string {
  if (!root) return "";
  let s = root.val + "," + root.children.length;
  for (const child of root.children) s += "," + serializeNary(child);
  return s;
}
```

Or nested JSON with `children: []` array — natural for React state.

---

## 6. Interview Quick Index

| Question | Answer |
|----------|--------|
| Serialize binary tree? | Preorder + "null" markers |
| Deserialize? | Recursive build consuming tokens in order |
| JSON nested vs flat? | Nested = readable; flat = compact |
| N-ary? | Store child count or children array |
| Frontend storage? | `JSON.stringify` + localStorage |

---

## Day 32 Cheat Sheet

```
Binary     → preorder comma-separated + "null"
JSON       → { val, left, right } recursive
Deserialize → read val → build left → build right
N-ary      → children[] in JSON or val,count,children...
```

---

*End of Day 32 concepts*
