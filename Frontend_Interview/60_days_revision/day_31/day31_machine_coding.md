# Day 31 — Machine Coding Revision

**React:** Level-order grid layout · tree to CSS grid  
**React Native:** Breadth-first feed · paginated levels

---

## Table of Contents

1. [Level-Order Grid Layout — React](#1-level-order-grid-layout--react)
2. [BFS to Position Map](#2-bfs-to-position-map)
3. [CSS Grid Tree Visualizer](#3-css-grid-tree-visualizer)
4. [RN Breadth-First Feed](#4-rn-breadth-first-feed)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. Level-Order Grid Layout — React

Render tree nodes in rows by depth — like org chart or mind map.

```ts
type PositionedNode = {
  node: TreeNode;
  row: number;
  col: number;
};

function bfsWithPositions(root: TreeNode | null): PositionedNode[] {
  if (!root) return [];
  const result: PositionedNode[] = [];
  const queue: { node: TreeNode; row: number; col: number }[] = [
    { node: root, row: 0, col: 0 },
  ];

  while (queue.length) {
    const size = queue.length;
    for (let i = 0; i < size; i++) {
      const { node, row, col } = queue.shift()!;
      result.push({ node, row, col });
      const nextRow = row + 1;
      if (node.left) queue.push({ node: node.left, row: nextRow, col: col * 2 });
      if (node.right) queue.push({ node: node.right, row: nextRow, col: col * 2 + 1 });
    }
  }
  return result;
}
```

---

## 2. BFS to Position Map

```tsx
function TreeGrid({ root }: { root: TreeNode }) {
  const positioned = useMemo(() => bfsWithPositions(root), [root]);
  const maxRow = Math.max(...positioned.map((p) => p.row));

  return (
    <div
      style={{
        display: "grid",
        gridTemplateRows: `repeat(${maxRow + 1}, 60px)`,
        gridTemplateColumns: `repeat(${2 ** (maxRow + 1)}, 1fr)`,
        gap: 4,
      }}
    >
      {positioned.map(({ node, row, col }) => (
        <div
          key={node.val + row}
          style={{
            gridRow: row + 1,
            gridColumn: col + 2 ** maxRow,
            border: "1px solid #ccc",
            borderRadius: 8,
            display: "flex",
            alignItems: "center",
            justifyContent: "center",
          }}
        >
          {node.val}
        </div>
      ))}
    </div>
  );
}
```

---

## 3. CSS Grid Tree Visualizer

Alternative: flex rows per level (simpler for interviews).

```tsx
function LevelRows({ levels }: { levels: TreeNode[][] }) {
  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 24, alignItems: "center" }}>
      {levels.map((level, i) => (
        <div key={i} style={{ display: "flex", gap: 16 }}>
          {level.map((node) => (
            <div key={node.val} style={{ width: 40, height: 40, border: "1px solid", borderRadius: "50%", display: "flex", alignItems: "center", justifyContent: "center" }}>
              {node.val}
            </div>
          ))}
        </div>
      ))}
    </div>
  );
}
```

---

## 4. RN Breadth-First Feed

Load content level-by-level (e.g. categories then items).

```tsx
function useBFSLevels<T>(root: CategoryNode<T>) {
  const [levels, setLevels] = useState<CategoryNode<T>[][]>([]);
  const [loadedDepth, setLoadedDepth] = useState(0);

  useEffect(() => {
    const queue: { node: CategoryNode<T>; depth: number }[] = [{ node: root, depth: 0 }];
    const grouped: CategoryNode<T>[][] = [];

    while (queue.length) {
      const size = queue.length;
      const level: CategoryNode<T>[] = [];
      for (let i = 0; i < size; i++) {
        const { node, depth } = queue.shift()!;
        if (depth <= loadedDepth) level.push(node);
        node.children?.forEach((c) => queue.push({ node: c, depth: depth + 1 }));
      }
      if (level.length) grouped.push(level);
    }
    setLevels(grouped);
  }, [root, loadedDepth]);

  return { levels, loadNextLevel: () => setLoadedDepth((d) => d + 1) };
}
```

**Pattern:** Paginate BFS — show level 0, tap "Load more" for level 1+.

---

## 5. Interview Checklist

- [ ] BFS groups levels with fixed `size`
- [ ] Grid or flex layout per row
- [ ] RN: lazy load deeper levels
- [ ] Mention O(w) space for queue

---

*End of Day 31 machine coding*
