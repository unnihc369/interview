# Day 15 — Machine Coding Revision

**React:** Two-pointer drag-and-drop list reorder  
**React Native:** Swipe-to-delete list row

---

## Table of Contents

### React (Web)

1. [Problem — Reorderable List](#1-problem--reorderable-list)
2. [Two-Pointer Reorder Logic](#2-two-pointer-reorder-logic)
3. [Drag-and-Drop UI](#3-drag-and-drop-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Swipe-to-Delete Row](#5-swipe-to-delete-row)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Reorderable List

**Task:** Build a list where users **drag** an item and **drop** it at another index (playlist / task reorder).

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Drag start | Remember `dragIndex` |
| Drag over | Highlight `hoverIndex` |
| Drop | Move item from `dragIndex` → `hoverIndex` |
| Immutable state | New array reference for React re-render |

**Two-pointer mental model:** `from` and `to` indices — splice without mutating original.

---

## 2. Two-Pointer Reorder Logic

```ts
function reorder<T>(list: T[], fromIndex: number, toIndex: number): T[] {
  if (fromIndex === toIndex) return list;
  const next = [...list];
  const [moved] = next.splice(fromIndex, 1);
  next.splice(toIndex, 0, moved);
  return next;
}
```

### Alternative (two-pointer style on copy)

```ts
function reorderTwoPointer<T>(arr: T[], i: number, j: number): T[] {
  const out = [...arr];
  const item = out[i];
  if (i < j) {
    for (let k = i; k < j; k++) out[k] = out[k + 1];
    out[j] = item;
  } else {
    for (let k = i; k > j; k--) out[k] = out[k - 1];
    out[j] = item;
  }
  return out;
}
```

---

## 3. Drag-and-Drop UI

### HTML5 DnD (interview-friendly)

```tsx
import { useState } from "react";

type Item = { id: string; label: string };

export default function ReorderList() {
  const [items, setItems] = useState<Item[]>([
    { id: "1", label: "Track A" },
    { id: "2", label: "Track B" },
    { id: "3", label: "Track C" },
  ]);
  const [dragIndex, setDragIndex] = useState<number | null>(null);
  const [hoverIndex, setHoverIndex] = useState<number | null>(null);

  const onDragStart = (index: number) => setDragIndex(index);

  const onDragOver = (e: React.DragEvent, index: number) => {
    e.preventDefault();
    setHoverIndex(index);
  };

  const onDrop = (index: number) => {
    if (dragIndex === null) return;
    setItems((prev) => reorder(prev, dragIndex, index));
    setDragIndex(null);
    setHoverIndex(null);
  };

  return (
    <ul style={{ listStyle: "none", padding: 0 }}>
      {items.map((item, index) => (
        <li
          key={item.id}
          draggable
          onDragStart={() => onDragStart(index)}
          onDragOver={(e) => onDragOver(e, index)}
          onDrop={() => onDrop(index)}
          style={{
            padding: 12,
            marginBottom: 8,
            border: "1px solid #ccc",
            background: hoverIndex === index ? "#eef" : "#fff",
            cursor: "grab",
          }}
        >
          {item.label}
        </li>
      ))}
    </ul>
  );
}

function reorder<T>(list: T[], from: number, to: number): T[] {
  const next = [...list];
  const [moved] = next.splice(from, 1);
  next.splice(to, 0, moved);
  return next;
}
```

### Touch / production note

Mention **`@dnd-kit/core`** or **`react-beautiful-dnd`** for accessibility and mobile — HTML5 DnD is weak on touch.

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Why immutable reorder? | React compares by reference; in-place `sort` on state breaks predictability |
| `splice` twice? | Remove at `from`, insert at `to` (adjust `to` if `from < to` when using single splice) |
| Keys in list | Stable `id`, not array index, if list can reorder |
| Performance | `useCallback` handlers; virtualize long lists |

---

# Part B — React Native

## 5. Swipe-to-Delete Row

### Using `react-native-gesture-handler` Swipeable

```bash
npx expo install react-native-gesture-handler
```

```tsx
import { FlatList, Text, StyleSheet } from "react-native";
import { Swipeable } from "react-native-gesture-handler";
import { useState } from "react";

type Row = { id: string; title: string };

export default function SwipeDeleteList() {
  const [rows, setRows] = useState<Row[]>([
    { id: "1", title: "Inbox item 1" },
    { id: "2", title: "Inbox item 2" },
  ]);

  const deleteRow = (id: string) => {
    setRows((prev) => prev.filter((r) => r.id !== id));
  };

  const renderRightActions = (id: string) => (
    <Text
      onPress={() => deleteRow(id)}
      style={styles.deleteAction}
    >
      Delete
    </Text>
  );

  return (
    <FlatList
      data={rows}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <Swipeable renderRightActions={() => renderRightActions(item.id)}>
          <Text style={styles.row}>{item.title}</Text>
        </Swipeable>
      )}
    />
  );
}

const styles = StyleSheet.create({
  row: { padding: 16, backgroundColor: "#fff" },
  deleteAction: {
    backgroundColor: "#e53935",
    color: "#fff",
    width: 80,
    textAlign: "center",
    lineHeight: 48,
  },
});
```

### Two-pointer tie-in

After delete, indices shift — same as removing index `i` from array: `filter` or `splice` with immutable copy.

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Swipeable vs PanResponder? | Swipeable is higher-level; PanResponder for custom gestures |
| FlatList + Swipeable | One Swipeable per row; close open row on scroll (ref) |
| Undo pattern? | Keep deleted item in ref 5s before API call |

---

## Quick Revision — Day 15

```
Web:  dragIndex + hoverIndex → reorder([...], from, to)
RN:   Swipeable renderRightActions → filter id from state
Both: immutable array updates; stable keys
```

---

*End of Day 15 machine coding*
