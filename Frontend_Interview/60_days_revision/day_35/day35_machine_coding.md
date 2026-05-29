# Day 35 — Machine Coding Revision

**React:** Tree drag-and-drop reorder · dnd-kit / HTML5 DnD  
**React Native:** Long-press context menu on tree nodes

---

## Table of Contents

1. [Drag-and-Drop Tree — React](#1-drag-and-drop-tree--react)
2. [Reorder with @dnd-kit](#2-reorder-with-dnd-kit)
3. [Move Node Between Parents](#3-move-node-between-parents)
4. [Long-Press Menu — React Native](#4-long-press-menu--react-native)
5. [Revision Checklist](#5-revision-checklist)

---

## 1. Drag-and-Drop Tree — React

**Goal:** Reorder siblings or move nodes between folders.

```ts
type FileNode = { id: string; name: string; type: "folder" | "file"; children?: FileNode[] };

function removeNode(tree: FileNode, id: string): [FileNode, FileNode | null] {
  if (tree.id === id) return [tree, null];
  if (!tree.children) return [tree, null];
  for (let i = 0; i < tree.children.length; i++) {
    if (tree.children[i].id === id) {
      const [removed] = tree.children.splice(i, 1);
      return [{ ...tree, children: [...tree.children] }, removed];
    }
    const [updated, removed] = removeNode(tree.children[i], id);
    if (removed) {
      const children = [...tree.children];
      children[i] = updated;
      return [{ ...tree, children }, removed];
    }
  }
  return [tree, null];
}

function insertNode(tree: FileNode, parentId: string, node: FileNode, index: number): FileNode {
  if (tree.id === parentId) {
    const children = [...(tree.children ?? [])];
    children.splice(index, 0, node);
    return { ...tree, children };
  }
  return {
    ...tree,
    children: tree.children?.map((c) => insertNode(c, parentId, node, index)),
  };
}
```

---

## 2. Reorder with @dnd-kit

```tsx
import { DndContext, closestCenter, DragEndEvent } from "@dnd-kit/core";
import { SortableContext, verticalListSortingStrategy, useSortable } from "@dnd-kit/sortable";

function SortableTreeItem({ node }: { node: FileNode }) {
  const { attributes, listeners, setNodeRef, transform } = useSortable({ id: node.id });
  return (
    <div ref={setNodeRef} {...attributes} {...listeners} style={{ transform: CSS.Transform.toString(transform) }}>
      {node.name}
    </div>
  );
}

function TreeDnd({ tree, setTree }: Props) {
  const onDragEnd = (event: DragEndEvent) => {
    const { active, over } = event;
    if (!over || active.id === over.id) return;
    // reorder siblings in immutable tree update
  };

  return (
    <DndContext collisionDetection={closestCenter} onDragEnd={onDragEnd}>
      <SortableContext items={tree.children?.map((c) => c.id) ?? []} strategy={verticalListSortingStrategy}>
        {tree.children?.map((c) => <SortableTreeItem key={c.id} node={c} />)}
      </SortableContext>
    </DndContext>
  );
}
```

Mention **HTML5 drag API** as simpler alternative for interviews.

---

## 3. Move Node Between Parents

```tsx
function handleDrop(dragId: string, targetFolderId: string, index: number) {
  setTree((prev) => {
    const [without, node] = removeNode(prev, dragId);
    if (!node) return prev;
    return insertNode(without, targetFolderId, node, index);
  });
}
```

Validate: cannot drop folder into its own descendant (cycle check via DFS).

---

## 4. Long-Press Menu — React Native

```tsx
import { ActionSheetIOS, Alert, Platform, Pressable, Text } from "react-native";

function TreeNodeRN({ node, onRename, onDelete, onAddChild }: Props) {
  const showMenu = () => {
    const options = ["Rename", "Add Folder", "Delete", "Cancel"];
    const destructiveButtonIndex = 2;
    const cancelButtonIndex = 3;

    if (Platform.OS === "ios") {
      ActionSheetIOS.showActionSheetWithOptions(
        { options, destructiveButtonIndex, cancelButtonIndex },
        (idx) => {
          if (idx === 0) onRename(node.id);
          if (idx === 1) onAddChild(node.id);
          if (idx === 2) onDelete(node.id);
        }
      );
    } else {
      Alert.alert(node.name, "Choose action", [
        { text: "Rename", onPress: () => onRename(node.id) },
        { text: "Add Folder", onPress: () => onAddChild(node.id) },
        { text: "Delete", style: "destructive", onPress: () => onDelete(node.id) },
        { text: "Cancel", style: "cancel" },
      ]);
    }
  };

  return (
    <Pressable onLongPress={showMenu} delayLongPress={400}>
      <Text>{node.name}</Text>
    </Pressable>
  );
}
```

**Bonus:** `react-native-context-menu-view` or custom bottom sheet for Material Design.

---

## 5. Revision Checklist

Week 5 machine coding recap:

- [ ] Recursive tree render + expand/collapse
- [ ] Flatten for FlatList (comments)
- [ ] BFS level layout
- [ ] localStorage JSON persistence
- [ ] LCA highlight in family tree
- [ ] Checkbox subtree selection
- [ ] Drag-drop or long-press menu

---

*End of Day 35 machine coding*
