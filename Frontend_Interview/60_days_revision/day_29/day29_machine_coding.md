# Day 29 — Machine Coding Revision

**React:** Tree viewer (nested folders) · expand/collapse · recursive rendering  
**React Native:** Nested comments thread · reply UI · flat list from tree

---

## Table of Contents

### React (Web)

1. [Problem — Nested Folder Tree Viewer](#1-problem--nested-folder-tree-viewer)
2. [Tree Data Model & Types](#2-tree-data-model--types)
3. [Recursive TreeNode Component](#3-recursive-treenode-component)
4. [Expand/Collapse State](#4-expandcollapse-state)
5. [Full FolderTree Solution](#5-full-foldertree-solution)

### React Native

6. [Nested Comments Model](#6-nested-comments-model)
7. [Flatten Tree for FlatList](#7-flatten-tree-for-flatlist)
8. [CommentItem with Reply](#8-commentitem-with-reply)
9. [RN Interview Points](#9-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Nested Folder Tree Viewer

**Task:** Render a nested folder structure like VS Code's file explorer.

| Feature | Behavior |
|---------|----------|
| Display hierarchy | Indent children |
| Expand/collapse | Toggle folder visibility |
| Icons | Folder open/closed, file leaf |
| Click file | `onSelect(path)` callback |

---

## 2. Tree Data Model & Types

```ts
export type FileNode = {
  id: string;
  name: string;
  type: "folder" | "file";
  children?: FileNode[];
};

export const sampleTree: FileNode = {
  id: "root",
  name: "project",
  type: "folder",
  children: [
    {
      id: "src",
      name: "src",
      type: "folder",
      children: [
        { id: "app", name: "App.tsx", type: "file" },
        {
          id: "hooks",
          name: "hooks",
          type: "folder",
          children: [{ id: "useStack", name: "useStack.ts", type: "file" }],
        },
      ],
    },
    { id: "pkg", name: "package.json", type: "file" },
  ],
};
```

---

## 3. Recursive TreeNode Component

```tsx
import { useState } from "react";
import type { FileNode } from "./types";

type Props = {
  node: FileNode;
  depth?: number;
  onSelect?: (path: string) => void;
};

export function TreeNode({ node, depth = 0, onSelect }: Props) {
  const [open, setOpen] = useState(depth < 2); // auto-expand first levels
  const isFolder = node.type === "folder" && node.children?.length;

  const handleClick = () => {
    if (isFolder) setOpen((o) => !o);
    else onSelect?.(node.name);
  };

  return (
    <div>
      <div
        role="treeitem"
        aria-expanded={isFolder ? open : undefined}
        style={{
          paddingLeft: depth * 16,
          cursor: "pointer",
          display: "flex",
          alignItems: "center",
          gap: 6,
        }}
        onClick={handleClick}
      >
        <span>{isFolder ? (open ? "📂" : "📁") : "📄"}</span>
        <span>{node.name}</span>
      </div>

      {isFolder && open &&
        node.children!.map((child) => (
          <TreeNode key={child.id} node={child} depth={depth + 1} onSelect={onSelect} />
        ))}
    </div>
  );
}
```

---

## 4. Expand/Collapse State

### Per-node local state (simple)

Each folder manages its own `open` — good for machine coding.

### Centralized Set of expanded IDs (scalable)

```tsx
function useExpanded(defaultIds: string[] = []) {
  const [expanded, setExpanded] = useState(() => new Set(defaultIds));

  const toggle = (id: string) =>
    setExpanded((prev) => {
      const next = new Set(prev);
      next.has(id) ? next.delete(id) : next.add(id);
      return next;
    });

  const isExpanded = (id: string) => expanded.has(id);
  return { isExpanded, toggle };
}
```

Pass `isExpanded(node.id)` and `toggle(node.id)` down instead of local `useState`.

---

## 5. Full FolderTree Solution

```tsx
import { sampleTree } from "./data";
import { TreeNode } from "./TreeNode";

export default function FolderTree() {
  return (
    <div role="tree" style={{ fontFamily: "monospace", padding: 16 }}>
      <h3>Project Files</h3>
      <TreeNode
        node={sampleTree}
        onSelect={(path) => console.log("Selected:", path)}
      />
    </div>
  );
}
```

### Interview checklist

- [ ] Recursive component with `depth` indent
- [ ] Folder vs file handling
- [ ] Expand/collapse toggle
- [ ] Unique `key={child.id}`
- [ ] Optional: `role="tree"` accessibility

---

# Part B — React Native

## 6. Nested Comments Model

```ts
export type Comment = {
  id: string;
  author: string;
  text: string;
  replies: Comment[];
};

export const comments: Comment[] = [
  {
    id: "1",
    author: "Alice",
    text: "Great post!",
    replies: [
      {
        id: "1-1",
        author: "Bob",
        text: "Thanks!",
        replies: [{ id: "1-1-1", author: "Alice", text: "Welcome", replies: [] }],
      },
    ],
  },
];
```

---

## 7. Flatten Tree for FlatList

RN `FlatList` prefers a **flat array**. Flatten tree with depth for indentation.

```ts
type FlatComment = Comment & { depth: number };

function flattenComments(nodes: Comment[], depth = 0): FlatComment[] {
  const out: FlatComment[] = [];
  for (const node of nodes) {
    out.push({ ...node, depth });
    if (node.replies.length) {
      out.push(...flattenComments(node.replies, depth + 1));
    }
  }
  return out;
}
```

**Alternative:** Recursive `View` nesting (fine for shallow threads; FlatList better for long lists).

---

## 8. CommentItem with Reply

```tsx
import { FlatList, Pressable, Text, View } from "react-native";
import { flattenComments, type Comment } from "./types";

function CommentRow({
  item,
  onReply,
}: {
  item: Comment & { depth: number };
  onReply: (id: string) => void;
}) {
  return (
    <View style={{ paddingLeft: item.depth * 16, paddingVertical: 8 }}>
      <Text style={{ fontWeight: "600" }}>{item.author}</Text>
      <Text>{item.text}</Text>
      <Pressable onPress={() => onReply(item.id)}>
        <Text style={{ color: "#007AFF", marginTop: 4 }}>Reply</Text>
      </Pressable>
    </View>
  );
}

export default function CommentThread({ data }: { data: Comment[] }) {
  const flat = flattenComments(data);

  return (
    <FlatList
      data={flat}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <CommentRow item={item} onReply={(id) => console.log("Reply to", id)} />
      )}
    />
  );
}
```

### Collapsible replies (bonus)

Track `collapsedIds: Set<string>` — when collapsed, skip flattening that subtree.

---

## 9. RN Interview Points

| Question | Answer |
|----------|--------|
| Render nested tree in RN? | Recursive Views or flatten + FlatList |
| Why flatten? | Virtualization, performance on long threads |
| Indentation | `paddingLeft: depth * 16` |
| Add reply | Append to parent's `replies`, re-render |
| State shape | Tree in state; immutable update with spread |

---

## Quick Revision — Day 29 Machine Coding

```
React:     recursive TreeNode + depth indent + open state
RN:        Comment tree → flattenComments → FlatList
Pattern:   tree data → recursive or flat render
Key:       immutable updates when adding replies
```

---

*End of Day 29 machine coding*
