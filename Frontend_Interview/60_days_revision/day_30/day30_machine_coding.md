# Day 30 — Machine Coding Revision

**React:** File explorer expand/collapse all · keyboard navigation  
**React Native:** Org chart · pinch zoom · recursive layout

---

## Table of Contents

1. [Expand/Collapse All — React](#1-expandcollapse-all--react)
2. [Keyboard Tree Navigation](#2-keyboard-tree-navigation)
3. [Org Chart — React Native](#3-org-chart--react-native)
4. [Pinch-to-Zoom Wrapper](#4-pinch-to-zoom-wrapper)
5. [Interview Checklist](#5-interview-checklist)

---

## 1. Expand/Collapse All — React

```tsx
function useTreeExpansion(allFolderIds: string[], defaultOpen: string[] = []) {
  const [expanded, setExpanded] = useState(() => new Set(defaultOpen));

  return {
    isExpanded: (id: string) => expanded.has(id),
    toggle: (id: string) =>
      setExpanded((prev) => {
        const next = new Set(prev);
        next.has(id) ? next.delete(id) : next.add(id);
        return next;
      }),
    expandAll: () => setExpanded(new Set(allFolderIds)),
    collapseAll: () => setExpanded(new Set()),
  };
}

function collectFolderIds(node: FileNode): string[] {
  if (node.type !== "folder" || !node.children?.length) return [];
  return [node.id, ...node.children.flatMap(collectFolderIds)];
}
```

```tsx
export function FileExplorer({ tree }: { tree: FileNode }) {
  const folderIds = useMemo(() => collectFolderIds(tree), [tree]);
  const { isExpanded, toggle, expandAll, collapseAll } = useTreeExpansion(folderIds, ["root"]);

  return (
    <div>
      <div style={{ display: "flex", gap: 8, marginBottom: 8 }}>
        <button onClick={expandAll}>Expand All</button>
        <button onClick={collapseAll}>Collapse All</button>
      </div>
      <TreeNode node={tree} isExpanded={isExpanded} onToggle={toggle} />
    </div>
  );
}
```

---

## 2. Keyboard Tree Navigation

```tsx
// ArrowRight → expand folder; ArrowLeft → collapse
// Enter → select file
const onKeyDown = (e: React.KeyboardEvent, node: FileNode) => {
  if (e.key === "ArrowRight" && node.type === "folder") onToggle(node.id);
  if (e.key === "ArrowLeft" && node.type === "folder") onToggle(node.id);
  if (e.key === "Enter" && node.type === "file") onSelect(node.name);
};
```

Mention `role="tree"`, `aria-expanded`, `tabIndex={0}` for accessibility.

---

## 3. Org Chart — React Native

```tsx
type OrgNode = { id: string; name: string; title: string; reports: OrgNode[] };

function OrgNodeView({ node, scale }: { node: OrgNode; scale: number }) {
  return (
    <View style={{ alignItems: "center", transform: [{ scale }] }}>
      <View style={{ padding: 12, borderWidth: 1, borderRadius: 8, minWidth: 100, alignItems: "center" }}>
        <Text style={{ fontWeight: "700" }}>{node.name}</Text>
        <Text style={{ fontSize: 12, color: "#666" }}>{node.title}</Text>
      </View>
      {node.reports.length > 0 && (
        <View style={{ flexDirection: "row", marginTop: 20, gap: 12 }}>
          {node.reports.map((r) => (
            <OrgNodeView key={r.id} node={r} scale={scale} />
          ))}
        </View>
      )}
    </View>
  );
}
```

**Layout tip:** Horizontal `ScrollView` wrapping vertical scroll for wide org trees.

---

## 4. Pinch-to-Zoom Wrapper

```tsx
import { PinchGestureHandler } from "react-native-gesture-handler";
import Animated, { useAnimatedGestureHandler, useSharedValue, useAnimatedStyle } from "react-native-reanimated";

function Zoomable({ children }: { children: React.ReactNode }) {
  const scale = useSharedValue(1);
  const handler = useAnimatedGestureHandler({
    onActive: (e) => { scale.value = Math.min(Math.max(e.scale, 0.5), 2.5); },
  });
  const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }));

  return (
    <PinchGestureHandler onGestureEvent={handler}>
      <Animated.View style={style}>{children}</Animated.View>
    </PinchGestureHandler>
  );
}
```

Interview: mention `react-native-gesture-handler` + `reanimated` for smooth zoom.

---

## 5. Interview Checklist

- [ ] Expand/collapse all with Set of IDs
- [ ] Recursive org chart component
- [ ] ScrollView for overflow
- [ ] Optional pinch zoom
- [ ] Unique keys on recursive children

---

*End of Day 30 machine coding*
