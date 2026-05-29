# Day 14 — Machine Coding Revision (Sunday)

**React:** Optimize windowed list (10k items) — full solution review  
**React Native:** Profile screen with sliding tabs

---

## Table of Contents

### React (Web)

1. [Mock Task — 10k List Optimization](#1-mock-task--10k-list-optimization)
2. [Checklist & Scoring](#2-checklist--scoring)
3. [Optimized VirtualList (Complete)](#3-optimized-virtuallist-complete)
4. [Further Optimizations](#4-further-optimizations)

### React Native

5. [Profile Sliding Tabs](#5-profile-sliding-tabs)
6. [Full Implementation](#6-full-implementation)
7. [Mock Rubric](#7-mock-rubric)

---

# Part A — React (Web)

## 1. Mock Task — 10k List Optimization

**Prompt (45 min):** You have a scrollable list of 10,000 items. Current implementation lags. Optimize.

**Starting point (intentionally bad):**

```tsx
// ❌ DO NOT ship this
export function SlowList({ items }: { items: string[] }) {
  return (
    <div style={{ height: 400, overflow: "auto" }}>
      {items.map((item, i) => (
        <div key={i} style={{ height: 48, padding: 8 }}>
          {item}
        </div>
      ))}
    </div>
  );
}
```

**Expected solution axes:**

1. Windowed rendering (only visible rows)
2. Stable keys (`id` not index)
3. Memoized row component
4. Optional: `requestAnimationFrame` for scroll handler
5. Mention `@tanstack/react-virtual` for production

---

## 2. Checklist & Scoring

| Criteria | Points |
|----------|--------|
| Explains why 10k DOM nodes fail | 10 |
| Correct start/end index math | 25 |
| Spacer / translateY technique | 25 |
| overscan mentioned | 10 |
| Memoized row + stable keys | 15 |
| Scroll performance (passive listener) | 5 |
| Accessibility (keyboard scroll) | 10 |

---

## 3. Optimized VirtualList (Complete)

```tsx
import React, { memo, useCallback, useMemo, useState } from "react";

const ROW_HEIGHT = 48;
const VIEWPORT = 400;
const OVERSCAN = 5;

type RowProps = { index: number; label: string };

const Row = memo(function Row({ index, label }: RowProps) {
  return (
    <div
      style={{
        height: ROW_HEIGHT,
        display: "flex",
        alignItems: "center",
        padding: "0 16px",
        borderBottom: "1px solid #eee",
      }}
    >
      {label}
    </div>
  );
});

const ITEMS = Array.from({ length: 10_000 }, (_, i) => ({
  id: `row-${i}`,
  label: `Item ${i + 1}`,
}));

export default function OptimizedList() {
  const [scrollTop, setScrollTop] = useState(0);

  const { start, end, offsetY, totalHeight } = useMemo(() => {
    const visible = Math.ceil(VIEWPORT / ROW_HEIGHT);
    const start = Math.max(0, Math.floor(scrollTop / ROW_HEIGHT) - OVERSCAN);
    const end = Math.min(
      ITEMS.length - 1,
      start + visible + OVERSCAN * 2
    );
    return {
      start,
      end,
      offsetY: start * ROW_HEIGHT,
      totalHeight: ITEMS.length * ROW_HEIGHT,
    };
  }, [scrollTop]);

  const onScroll = useCallback((e: React.UIEvent<HTMLDivElement>) => {
    setScrollTop(e.currentTarget.scrollTop);
  }, []);

  const slice = useMemo(() => {
    const nodes = [];
    for (let i = start; i <= end; i++) {
      nodes.push(<Row key={ITEMS[i].id} index={i} label={ITEMS[i].label} />);
    }
    return nodes;
  }, [start, end]);

  return (
    <div
      role="list"
      aria-label="Optimized 10k items"
      onScroll={onScroll}
      style={{ height: VIEWPORT, overflow: "auto", border: "1px solid #ccc" }}
    >
      <div style={{ height: totalHeight, position: "relative" }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>{slice}</div>
      </div>
    </div>
  );
}
```

---

## 4. Further Optimizations

```tsx
// @tanstack/react-virtual — production default
import { useVirtualizer } from "@tanstack/react-virtual";

function TanStackList() {
  const parentRef = useRef<HTMLDivElement>(null);
  const virtualizer = useVirtualizer({
    count: 10_000,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 48,
    overscan: 8,
  });
  // map virtualizer.getVirtualItems() → rows
}
```

| Technique | Benefit |
|-----------|---------|
| `content-visibility: auto` | CSS skip off-screen layout (limited support) |
| Variable row height | Measure + cache positions |
| Web Worker | Sort/filter 10k without blocking main thread |

---

# Part B — React Native

## 5. Profile Sliding Tabs

**Mock:** Profile header + horizontally swipable tabs (Posts / Photos / About). Tab bar syncs with swipe.

**Stack:** `react-native-tab-view` or manual `FlatList` paging + `ScrollView`.

---

## 6. Full Implementation

```tsx
import { useState } from "react";
import { Dimensions, Pressable, Text, View } from "react-native";
import { TabView, SceneMap, TabBar } from "react-native-tab-view";

const W = Dimensions.get("window").width;

const Posts = () => (
  <View style={{ padding: 16 }}>
  {Array.from({ length: 20 }, (_, i) => (
    <Text key={i} style={{ marginBottom: 12 }}>Post {i + 1}</Text>
  ))}
  </View>
);

const Photos = () => (
  <View style={{ padding: 16, flexDirection: "row", flexWrap: "wrap", gap: 8 }}>
    {Array.from({ length: 12 }, (_, i) => (
      <View
        key={i}
        style={{ width: 100, height: 100, backgroundColor: `#${((i * 40) % 255).toString(16)}88` }}
      />
    ))}
  </View>
);

const About = () => (
  <View style={{ padding: 16 }}>
    <Text>Bio, location, links…</Text>
  </View>
);

const renderScene = SceneMap({
  posts: Posts,
  photos: Photos,
  about: About,
});

export default function ProfileSlidingTabs() {
  const [index, setIndex] = useState(0);
  const [routes] = useState([
    { key: "posts", title: "Posts" },
    { key: "photos", title: "Photos" },
    { key: "about", title: "About" },
  ]);

  return (
    <View style={{ flex: 1 }}>
      <View style={{ padding: 24, alignItems: "center", backgroundColor: "#f5f5f5" }}>
        <View
          style={{
            width: 80,
            height: 80,
            borderRadius: 40,
            backgroundColor: "#4a90d9",
            marginBottom: 8,
          }}
        />
        <Text style={{ fontSize: 20, fontWeight: "700" }}>Jane Doe</Text>
        <Text style={{ color: "#666" }}>@jane</Text>
      </View>

      <TabView
        navigationState={{ index, routes }}
        renderScene={renderScene}
        onIndexChange={setIndex}
        initialLayout={{ width: W }}
        renderTabBar={(props) => (
          <TabBar
            {...props}
            indicatorStyle={{ backgroundColor: "#4a90d9", height: 3 }}
            style={{ backgroundColor: "#fff" }}
            activeColor="#333"
            inactiveColor="#999"
          />
        )}
        lazy
        swipeEnabled
      />
    </View>
  );
}
```

```bash
npm install react-native-tab-view react-native-pager-view
```

### `lazy` prop

Only mount tab scene when first visited — saves memory (windowed tab content).

---

## 7. Mock Rubric

| Area | Pass |
|------|------|
| Web 10k | Virtualization + explain complexity |
| RN tabs | Swipe + tab sync + lazy scenes |
| Code quality | Types, keys, no inline 10k map |
| Communication | Tradeoffs stated aloud |

---

## Quick Revision — Day 14

```
Sunday mock:  OptimizedList + ProfileSlidingTabs + timed LC
Remember:     explain before coding; test edge cases (empty, 1 item)
```

---

*End of Day 14 machine coding*
