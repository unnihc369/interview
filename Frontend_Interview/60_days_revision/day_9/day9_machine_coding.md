# Day 9 — Machine Coding Revision

**React:** Window resize observer + responsive layout  
**React Native:** Horizontal swiper with dynamic window

---

## Table of Contents

### React (Web)

1. [Problem — Responsive Dashboard](#1-problem--responsive-dashboard)
2. [ResizeObserver Hook](#2-resizeobserver-hook)
3. [Responsive Grid Component](#3-responsive-grid-component)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Problem — Horizontal Swiper](#5-problem--horizontal-swiper)
6. [Dynamic Window Swiper](#6-dynamic-window-swiper)
7. [RN Interview Points](#7-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Responsive Dashboard

Build a layout that:

| Breakpoint | Columns | Behavior |
|------------|---------|----------|
| < 480px | 1 | Stack cards |
| 480–768px | 2 | Side by side |
| > 768px | 3 | Full grid |

Use **ResizeObserver** (not only `window.resize`) to measure **container** width — important for sidebars/modals.

---

## 2. ResizeObserver Hook

```tsx
import { useEffect, useRef, useState } from "react";

type Size = { width: number; height: number };

export function useResizeObserver<T extends HTMLElement>() {
  const ref = useRef<T>(null);
  const [size, setSize] = useState<Size>({ width: 0, height: 0 });

  useEffect(() => {
    const el = ref.current;
    if (!el) return;

    const ro = new ResizeObserver((entries) => {
      const { width, height } = entries[0].contentRect;
      setSize({ width, height });
    });
    ro.observe(el);
    return () => ro.disconnect();
  }, []);

  return { ref, ...size };
}

export function getColumns(width: number): number {
  if (width < 480) return 1;
  if (width < 768) return 2;
  return 3;
}
```

### Why ResizeObserver over window.resize?

| | `window.resize` | `ResizeObserver` |
|---|-----------------|------------------|
| Tracks | Viewport | Any element |
| Fires when | Window changes | Element box changes |
| Sidebar toggle | Misses | Catches |

---

## 3. Responsive Grid Component

```tsx
import { useMemo } from "react";
import { getColumns, useResizeObserver } from "./useResizeObserver";

const CARDS = Array.from({ length: 9 }, (_, i) => ({
  id: i,
  title: `Card ${i + 1}`,
}));

export default function ResponsiveDashboard() {
  const { ref, width } = useResizeObserver<HTMLDivElement>();
  const columns = useMemo(() => getColumns(width), [width]);

  return (
    <div ref={ref} style={{ padding: 16, width: "100%" }}>
      <header style={{ marginBottom: 12 }}>
        <strong>Dashboard</strong>
        <span style={{ marginLeft: 8, color: "#666" }}>
          {width}px · {columns} col
        </span>
      </header>

      <div
        style={{
          display: "grid",
          gridTemplateColumns: `repeat(${columns}, 1fr)`,
          gap: 12,
        }}
      >
        {CARDS.map((card) => (
          <article
            key={card.id}
            style={{
              padding: 16,
              borderRadius: 8,
              background: "#f5f5f5",
              minHeight: 80,
            }}
          >
            {card.title}
          </article>
        ))}
      </div>
    </div>
  );
}
```

### Debounced stats (bonus — ties to Day 11)

```tsx
import { useEffect, useState } from "react";

function useDebouncedValue<T>(value: T, delay: number): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);
  return debounced;
}

// In dashboard: const stableWidth = useDebouncedValue(width, 150);
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| ResizeObserver vs matchMedia | RO = element size; matchMedia = CSS breakpoints |
| Cleanup | `ro.disconnect()` in useEffect return |
| SSR | width 0 on first paint — avoid layout flash with default |
| Performance | Debounce if updating heavy charts on resize |

---

# Part B — React Native

## 5. Problem — Horizontal Swiper

Carousel / onboarding with:

- Horizontal paging
- **Dynamic window**: render current ±1 slide (preload neighbors)
- Dots indicator synced to active index
- Works with 100+ slides (don't mount all)

---

## 6. Dynamic Window Swiper

```tsx
import { useCallback, useRef, useState } from "react";
import {
  Dimensions,
  FlatList,
  NativeScrollEvent,
  NativeSyntheticEvent,
  Text,
  View,
} from "react-native";

const { width: SCREEN_W } = Dimensions.get("window");
const WINDOW_RADIUS = 1; // render index ± 1 fully; others placeholder

type Slide = { id: string; color: string; label: string };

const SLIDES: Slide[] = Array.from({ length: 50 }, (_, i) => ({
  id: String(i),
  color: `hsl(${(i * 40) % 360}, 70%, 60%)`,
  label: `Slide ${i + 1}`,
}));

export default function DynamicSwiper() {
  const [activeIndex, setActiveIndex] = useState(0);
  const listRef = useRef<FlatList>(null);

  const onMomentumScrollEnd = useCallback(
    (e: NativeSyntheticEvent<NativeScrollEvent>) => {
      const idx = Math.round(e.nativeEvent.contentOffset.x / SCREEN_W);
      setActiveIndex(idx);
    },
    []
  );

  const renderItem = useCallback(
    ({ item, index }: { item: Slide; index: number }) => {
      const inWindow = Math.abs(index - activeIndex) <= WINDOW_RADIUS;
      return (
        <View
          style={{
            width: SCREEN_W,
            height: 240,
            backgroundColor: inWindow ? item.color : "#ddd",
            justifyContent: "center",
            alignItems: "center",
          }}
        >
          <Text style={{ fontSize: 24, color: "#fff" }}>
            {inWindow ? item.label : "…"}
          </Text>
        </View>
      );
    },
    [activeIndex]
  );

  return (
    <View>
      <FlatList
        ref={listRef}
        data={SLIDES}
        horizontal
        pagingEnabled
        showsHorizontalScrollIndicator={false}
        keyExtractor={(s) => s.id}
        renderItem={renderItem}
        onMomentumScrollEnd={onMomentumScrollEnd}
        getItemLayout={(_, index) => ({
          length: SCREEN_W,
          offset: SCREEN_W * index,
          index,
        })}
        initialNumToRender={3}
        maxToRenderPerBatch={2}
        windowSize={3}
        removeClippedSubviews
      />

      <View style={{ flexDirection: "row", justifyContent: "center", gap: 6, marginTop: 12 }}>
        {SLIDES.slice(
          Math.max(0, activeIndex - 2),
          Math.min(SLIDES.length, activeIndex + 3)
        ).map((_, i) => {
          const dotIndex = Math.max(0, activeIndex - 2) + i;
          return (
            <View
              key={dotIndex}
              style={{
                width: 8,
                height: 8,
                borderRadius: 4,
                backgroundColor: dotIndex === activeIndex ? "#333" : "#ccc",
              }}
            />
          );
        })}
      </View>
    </View>
  );
}
```

### Production alternatives

- `react-native-pager-view` — native paging
- `react-native-reanimated-carousel` — gestures + windowing

---

## 7. RN Interview Points

| Question | Answer |
|----------|--------|
| `getItemLayout` | Required for performant scrollToIndex / known widths |
| `windowSize` | Multiplier of viewport height (3 = 1.5 screens each side) |
| Dynamic render | Only heavy content for `|index - active| <= R` |
| horizontal FlatList | `pagingEnabled` + fixed item width = screen |

---

## Quick Revision — Day 9

```
Web:  ResizeObserver → container width → grid columns
RN:   horizontal FlatList + activeIndex window + getItemLayout
```

---

*End of Day 9 machine coding*
