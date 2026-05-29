# Day 11 — Machine Coding Revision

**React:** Debounced resize/scroll + window stats panel  
**React Native:** SectionList with sticky headers

---

## Table of Contents

### React (Web)

1. [Problem — Scroll & Resize Stats](#1-problem--scroll--resize-stats)
2. [useDebouncedWindowStats Hook](#2-usedebouncedwindowstats-hook)
3. [StatsPanel Component](#3-statspanel-component)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [SectionList Sticky Headers](#5-sectionlist-sticky-headers)
6. [Full SectionList Example](#6-full-sectionlist-example)
7. [RN Interview Points](#7-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Scroll & Resize Stats

Dashboard showing **live debounced** metrics:

| Metric | Source |
|--------|--------|
| Viewport W×H | `window` resize |
| Scroll Y | scroll container |
| Scroll depth % | `scrollTop / (scrollHeight - clientHeight)` |
| Visible window | `[scrollTop, scrollTop + clientHeight]` |

Debounce **150ms** to avoid thrashing during drag-resize.

---

## 2. useDebouncedWindowStats Hook

```tsx
import { useEffect, useState } from "react";

type WindowStats = {
  width: number;
  height: number;
  scrollY: number;
  scrollDepth: number;
  visibleStart: number;
  visibleEnd: number;
};

function useDebouncedValue<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}

export function useWindowStats(scrollRef: React.RefObject<HTMLElement>) {
  const [stats, setStats] = useState<WindowStats>({
    width: window.innerWidth,
    height: window.innerHeight,
    scrollY: 0,
    scrollDepth: 0,
    visibleStart: 0,
    visibleEnd: window.innerHeight,
  });

  useEffect(() => {
    const read = () => {
      const el = scrollRef.current;
      const scrollY = el?.scrollTop ?? window.scrollY;
      const scrollHeight = el?.scrollHeight ?? document.documentElement.scrollHeight;
      const clientHeight = el?.clientHeight ?? window.innerHeight;
      const max = Math.max(scrollHeight - clientHeight, 1);

      setStats({
        width: window.innerWidth,
        height: window.innerHeight,
        scrollY,
        scrollDepth: scrollY / max,
        visibleStart: scrollY,
        visibleEnd: scrollY + clientHeight,
      });
    };

    read();
    window.addEventListener("resize", read);
    const el = scrollRef.current;
    el?.addEventListener("scroll", read, { passive: true });

    return () => {
      window.removeEventListener("resize", read);
      el?.removeEventListener("scroll", read);
    };
  }, [scrollRef]);

  return useDebouncedValue(stats, 150);
}
```

---

## 3. StatsPanel Component

```tsx
import { useRef } from "react";
import { useWindowStats } from "./useWindowStats";

export default function ScrollStatsDemo() {
  const scrollRef = useRef<HTMLDivElement>(null);
  const stats = useWindowStats(scrollRef);

  return (
    <div>
      <aside
        style={{
          position: "fixed",
          top: 8,
          right: 8,
          padding: 12,
          background: "#1e1e1e",
          color: "#0f0",
          fontFamily: "monospace",
          fontSize: 12,
          borderRadius: 8,
          zIndex: 10,
        }}
      >
        <div>Viewport: {stats.width} × {stats.height}</div>
        <div>Scroll Y: {Math.round(stats.scrollY)}</div>
        <div>Depth: {(stats.scrollDepth * 100).toFixed(1)}%</div>
        <div>
          Window: [{Math.round(stats.visibleStart)},{" "}
          {Math.round(stats.visibleEnd)}]
        </div>
      </aside>

      <div
        ref={scrollRef}
        style={{ height: "100vh", overflow: "auto", padding: 16 }}
      >
        {Array.from({ length: 200 }, (_, i) => (
          <p key={i} style={{ margin: "24px 0" }}>
            Paragraph {i + 1} — scroll to update debounced stats.
          </p>
        ))}
      </div>
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Debounce vs throttle | Debounce = after pause; throttle = max once per interval |
| passive scroll listener | `{ passive: true }` — can't preventDefault, smoother scroll |
| ResizeObserver here? | Optional for container; window stats use resize event |

---

# Part B — React Native

## 5. SectionList Sticky Headers

`SectionList` = grouped data + optional **sticky section headers** (stay pinned while section scrolls).

| Prop | Purpose |
|------|---------|
| `sections` | `{ title, data }[]` |
| `renderSectionHeader` | Header component |
| `stickySectionHeadersEnabled` | Pin headers (default `true` iOS) |
| `keyExtractor` | Stable keys per row |

---

## 6. Full SectionList Example

```tsx
import { SectionList, Text, View, StyleSheet } from "react-native";

type Item = { id: string; name: string };
type Section = { title: string; data: Item[] };

const SECTIONS: Section[] = [
  {
    title: "Fruits",
    data: [
      { id: "1", name: "Apple" },
      { id: "2", name: "Banana" },
    ],
  },
  {
    title: "Vegetables",
    data: [
      { id: "3", name: "Carrot" },
      { id: "4", name: "Broccoli" },
      { id: "5", name: "Spinach" },
    ],
  },
  {
    title: "Dairy",
    data: [{ id: "6", name: "Milk" }],
  },
];

export default function StickySectionList() {
  return (
    <SectionList
      sections={SECTIONS}
      keyExtractor={(item) => item.id}
      stickySectionHeadersEnabled
      renderSectionHeader={({ section: { title } }) => (
        <View style={styles.header}>
          <Text style={styles.headerText}>{title}</Text>
        </View>
      )}
      renderItem={({ item }) => (
        <View style={styles.row}>
          <Text>{item.name}</Text>
        </View>
      )}
      ItemSeparatorComponent={() => <View style={styles.separator} />}
      SectionSeparatorComponent={() => <View style={{ height: 8 }} />}
    />
  );
}

const styles = StyleSheet.create({
  header: {
    paddingHorizontal: 16,
    paddingVertical: 10,
    backgroundColor: "#f0f0f0",
    borderBottomWidth: 1,
    borderColor: "#ddd",
  },
  headerText: { fontWeight: "700", fontSize: 16 },
  row: { padding: 16, backgroundColor: "#fff" },
  separator: { height: 1, backgroundColor: "#eee", marginLeft: 16 },
});
```

### Scroll-to-section

```tsx
const listRef = useRef<SectionList>(null);
listRef.current?.scrollToLocation({
  sectionIndex: 1,
  itemIndex: 0,
  animated: true,
});
```

---

## 7. RN Interview Points

| Question | Answer |
|----------|--------|
| FlatList vs SectionList | SectionList for grouped sections + headers |
| Sticky on Android | `stickySectionHeadersEnabled` — test both platforms |
| Performance | `renderItem` memoization; avoid inline heavy headers |

---

## Quick Revision — Day 11

```
Web:  debounce resize+scroll → visible window range stats
RN:   SectionList + stickySectionHeadersEnabled + renderSectionHeader
```

---

*End of Day 11 machine coding*
