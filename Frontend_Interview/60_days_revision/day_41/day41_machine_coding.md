# Day 41 — Machine Coding Revision

**React:** Auto-balancing folder tree · rebalance on bulk import  
**React Native:** Reorderable FlatList · drag reorder with balance hint

---

## Table of Contents

1. [Detect Imbalance in UI Tree](#1-detect-imbalance-in-ui-tree)
2. [Auto-Rebalance on Import — React](#2-auto-rebalance-on-import--react)
3. [Folder Tree Balance Indicator](#3-folder-tree-balance-indicator)
4. [Reorderable FlatList — RN](#4-reorderable-flatlist--rn)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. Detect Imbalance in UI Tree

```ts
function treeDepth(node: FileNode | null): number {
  if (!node) return 0;
  return 1 + Math.max(treeDepth(node.left ?? null), treeDepth(node.right ?? null));
}

function isBalancedUI(node: FileNode | null): boolean {
  if (!node) return true;
  const L = treeDepth(node.left ?? null);
  const R = treeDepth(node.right ?? null);
  if (Math.abs(L - R) > 1) return false;
  return isBalancedUI(node.left ?? null) && isBalancedUI(node.right ?? null);
}
```

For N-ary folder trees, compare max child depth vs min child depth as heuristic.

---

## 2. Auto-Rebalance on Import — React

```tsx
function FolderImporter() {
  const [root, setRoot] = useState<FileNode | null>(null);
  const [balanced, setBalanced] = useState(true);

  const importFlat = (flat: { id: string; name: string; parentId: string | null }[]) => {
    const names = flat.map((f) => f.name).sort();
    const bstRoot = sortedArrayToBST(names.map((n, i) => ({ val: i, name: n })));
    setRoot(bstToFolderTree(bstRoot, flat));
    setBalanced(true);
  };

  const importUnbalanced = (tree: FileNode) => {
    setRoot(tree);
    setBalanced(isBalancedUI(convertToBinary(tree)));
  };

  return (
    <div>
      {!balanced && (
        <div style={{ background: "#fef3c7", padding: 8 }}>
          Tree imbalanced — <button onClick={() => importFlat(flatten(root))}>Rebalance</button>
        </div>
      )}
      <TreeView root={root} />
    </div>
  );
}
```

---

## 3. Folder Tree Balance Indicator

```tsx
function BalanceBadge({ root }: { root: FileNode | null }) {
  const depth = treeDepth(convertToBinary(root));
  const nodeCount = countNodes(root);
  const idealDepth = Math.ceil(Math.log2(nodeCount + 1));
  const ratio = depth / idealDepth;

  return (
    <span style={{ color: ratio > 1.5 ? "red" : "green" }}>
      Depth {depth} (ideal ~{idealDepth})
    </span>
  );
}
```

---

## 4. Reorderable FlatList — RN

```tsx
import DraggableFlatList, { RenderItemParams } from "react-native-draggable-flatlist";

function ReorderableFolders({ items, onReorder }: Props) {
  return (
    <DraggableFlatList
      data={items}
      keyExtractor={(item) => item.id}
      onDragEnd={({ data }) => onReorder(data)}
      renderItem={({ item, drag, isActive }: RenderItemParams<FileNode>) => (
        <Pressable
          onLongPress={drag}
          style={{ padding: 16, backgroundColor: isActive ? "#e0e7ff" : "#fff" }}
        >
          <Text>{item.name}</Text>
        </Pressable>
      )}
    />
  );
}
```

After reorder, optionally **rebuild balanced index** from sorted order for O(log n) lookups.

**Package:** `react-native-draggable-flatlist` — mention in interview.

---

## 5. Interview Checklist

- [ ] Explain balance vs skewed performance
- [ ] Rebalance via inorder + mid split
- [ ] UI warning when tree too deep
- [ ] RN drag reorder list

---

*End of Day 41 machine coding*
