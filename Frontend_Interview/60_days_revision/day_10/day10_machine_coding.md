# Day 10 — Machine Coding Revision

**React:** Virtualized list — windowed rendering  
**React Native:** FlatList windowing thresholds

---

## Table of Contents

### React (Web)

1. [Problem — 10k Row List](#1-problem--10k-row-list)
2. [Windowed Rendering Concept](#2-windowed-rendering-concept)
3. [useVirtualWindow Hook](#3-usevirtualwindow-hook)
4. [VirtualList Component](#4-virtuallist-component)
5. [React Interview Points](#5-react-interview-points)

### React Native

6. [FlatList Windowing Props](#6-flatlist-windowing-props)
7. [Tuned FlatList Example](#7-tuned-flatlist-example)
8. [RN Interview Points](#8-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — 10k Row List

Render a list of **10,000 items** smoothly. Only mount DOM nodes for **visible window** + small overscan buffer.

| Requirement | Target |
|-------------|--------|
| Scroll FPS | ~60 |
| DOM nodes | ~20–40, not 10k |
| Variable row height | Bonus (use measure cache) |
| Scroll to index | Supported |

---

## 2. Windowed Rendering Concept

```
Total items:     [0 … 9999]
Viewport:              [visible]
Render window:    [overscan | visible | overscan]
```

**Key values:**

- `scrollTop` — scroll container offset
- `itemHeight` — fixed row height (simplify interview)
- `startIndex = floor(scrollTop / itemHeight) - overscan`
- `endIndex = startIndex + visibleCount + 2 * overscan`

---

## 3. useVirtualWindow Hook

```tsx
import { useCallback, useMemo, useState } from "react";

type Options = {
  itemCount: number;
  itemHeight: number;
  viewportHeight: number;
  overscan?: number;
};

export function useVirtualWindow({
  itemCount,
  itemHeight,
  viewportHeight,
  overscan = 3,
}: Options) {
  const [scrollTop, setScrollTop] = useState(0);

  const visibleCount = Math.ceil(viewportHeight / itemHeight);

  const range = useMemo(() => {
    const start = Math.max(0, Math.floor(scrollTop / itemHeight) - overscan);
    const end = Math.min(
      itemCount - 1,
      start + visibleCount + overscan * 2
    );
    return { start, end };
  }, [scrollTop, itemHeight, visibleCount, overscan, itemCount]);

  const totalHeight = itemCount * itemHeight;
  const offsetY = range.start * itemHeight;

  const onScroll = useCallback((e: React.UIEvent<HTMLDivElement>) => {
    setScrollTop(e.currentTarget.scrollTop);
  }, []);

  return { range, totalHeight, offsetY, onScroll, scrollTop };
}
```

---

## 4. VirtualList Component

```tsx
import { useVirtualWindow } from "./useVirtualWindow";

const ROW_H = 48;
const VIEW_H = 400;
const COUNT = 10_000;

export default function VirtualList() {
  const { range, totalHeight, offsetY, onScroll } = useVirtualWindow({
    itemCount: COUNT,
    itemHeight: ROW_H,
    viewportHeight: VIEW_H,
    overscan: 5,
  });

  const items = [];
  for (let i = range.start; i <= range.end; i++) {
    items.push(
      <div
        key={i}
        style={{
          height: ROW_H,
          display: "flex",
          alignItems: "center",
          padding: "0 16px",
          borderBottom: "1px solid #eee",
          boxSizing: "border-box",
        }}
      >
        Row {i + 1}
      </div>
    );
  }

  return (
    <div
      onScroll={onScroll}
      style={{
        height: VIEW_H,
        overflow: "auto",
        border: "1px solid #ccc",
        position: "relative",
      }}
    >
      <div style={{ height: totalHeight, position: "relative" }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>{items}</div>
      </div>
    </div>
  );
}
```

### Libraries (mention)

- `@tanstack/react-virtual` — headless, flexible
- `react-window` — fixed/variable size lists

---

## 5. React Interview Points

| Question | Answer |
|----------|--------|
| Why virtualize? | DOM cost — 10k nodes kill layout/paint |
| overscan | Extra rows above/below to reduce blank flash |
| Fixed vs variable height | Variable needs measure + position cache |
| Key stability | Use item id, not array index if reordering |

---

# Part B — React Native

## 6. FlatList Windowing Props

| Prop | Meaning | Typical |
|------|---------|---------|
| `initialNumToRender` | First paint batch | 10–15 |
| `maxToRenderPerBatch` | Per scroll batch | 5–10 |
| `windowSize` | Viewport multiples kept mounted | 5–10 |
| `updateCellsBatchingPeriod` | ms between batches | 50 |
| `removeClippedSubviews` | Unmount off-screen (Android) | `true` |
| `getItemLayout` | Skip measurement if fixed height | Required for scrollToIndex |

**`windowSize={5}`** → ~2.5 screen heights above + below kept in memory.

---

## 7. Tuned FlatList Example

```tsx
import { FlatList, Text, View } from "react-native";

const DATA = Array.from({ length: 10_000 }, (_, i) => ({
  id: String(i),
  title: `Item ${i + 1}`,
}));

const ITEM_H = 56;

export default function TunedFlatList() {
  return (
    <FlatList
      data={DATA}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <View
          style={{
            height: ITEM_H,
            justifyContent: "center",
            paddingHorizontal: 16,
            borderBottomWidth: 1,
            borderColor: "#eee",
          }}
        >
          <Text>{item.title}</Text>
        </View>
      )}
      getItemLayout={(_, index) => ({
        length: ITEM_H,
        offset: ITEM_H * index,
        index,
      })}
      initialNumToRender={12}
      maxToRenderPerBatch={8}
      windowSize={7}
      updateCellsBatchingPeriod={50}
      removeClippedSubviews
    />
  );
}
```

### FlashList (Shopify) — interview bonus

- Recycles views like native RecyclerView
- Drop-in FlatList replacement for large lists

---

## 8. RN Interview Points

| Question | Answer |
|----------|--------|
| FlatList vs ScrollView + map | FlatList virtualizes; ScrollView mounts all |
| Blank rows on fast scroll | Increase windowSize / initialNumToRender |
| `keyExtractor` | Stable unique keys — never Math.random |
| Memo `renderItem` | `React.memo` row component + `useCallback` |

---

## Quick Revision — Day 10

```
Web:  scrollTop → start/end index → translateY spacer
RN:   initialNumToRender, windowSize, getItemLayout, removeClippedSubviews
```

---

*End of Day 10 machine coding*
