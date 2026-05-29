# Day 8 — Machine Coding Revision

**React:** Live character counter (variable max length / sliding window)  
**React Native:** Windowed data loading with batch updates

---

## Table of Contents

### React (Web)

1. [Problem — Live Character Counter](#1-problem--live-character-counter)
2. [Variable Window Rules](#2-variable-window-rules)
3. [Full React Implementation](#3-full-react-implementation)
4. [useCharacterLimit Hook](#4-usecharacterlimit-hook)
5. [React Interview Points](#5-react-interview-points)

### React Native

6. [Problem — Windowed Feed Loader](#6-problem--windowed-feed-loader)
7. [Batch Update Strategy](#7-batch-update-strategy)
8. [FlatList Windowed Loader](#8-flatlist-windowed-loader)
9. [RN Interview Points](#9-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Live Character Counter

Build a textarea with:

| Feature | Behavior |
|---------|----------|
| Live count | Updates on every keystroke |
| Max length | Configurable (e.g. 280) |
| Variable window | Warn at 90%, error at 100% |
| Trim paste | Optional: truncate pasted text to fit |
| Accessibility | `aria-live` for screen readers |

**"Variable window"** = visual zones change as user approaches limit (green → yellow → red), similar to Twitter/X compose.

---

## 2. Variable Window Rules

```ts
type Zone = "safe" | "warn" | "danger" | "blocked";

function getZone(length: number, max: number): Zone {
  if (length >= max) return "blocked";
  if (length >= max * 0.95) return "danger";
  if (length >= max * 0.8) return "warn";
  return "safe";
}
```

| Zone | Threshold (max=280) | UI |
|------|---------------------|-----|
| safe | 0–223 | Gray counter |
| warn | 224–265 | Amber + "N left" |
| danger | 266–279 | Red pulse |
| blocked | 280 | Disable input / truncate |

---

## 3. Full React Implementation

```tsx
import { useCallback, useMemo, useState } from "react";

const MAX = 280;

type Zone = "safe" | "warn" | "danger" | "blocked";

function getZone(len: number, max: number): Zone {
  if (len >= max) return "blocked";
  if (len >= max * 0.95) return "danger";
  if (len >= max * 0.8) return "warn";
  return "safe";
}

const zoneStyles: Record<Zone, React.CSSProperties> = {
  safe: { color: "#666" },
  warn: { color: "#b8860b", fontWeight: 600 },
  danger: { color: "#c0392b", fontWeight: 700 },
  blocked: { color: "#c0392b" },
};

export default function CharacterCounter() {
  const [text, setText] = useState("");
  const len = text.length;
  const zone = getZone(len, MAX);
  const remaining = MAX - len;

  const handleChange = useCallback(
    (e: React.ChangeEvent<HTMLTextAreaElement>) => {
      const next = e.target.value;
      if (next.length <= MAX) setText(next);
      else setText(next.slice(0, MAX)); // hard cap
    },
    []
  );

  const handlePaste = useCallback(
    (e: React.ClipboardEvent<HTMLTextAreaElement>) => {
      e.preventDefault();
      const pasted = e.clipboardData.getData("text");
      setText((prev) => (prev + pasted).slice(0, MAX));
    },
    []
  );

  const counterLabel = useMemo(() => {
    if (zone === "blocked") return "Limit reached";
    if (zone === "danger" || zone === "warn") return `${remaining} left`;
    return `${len} / ${MAX}`;
  }, [zone, remaining, len]);

  return (
    <div style={{ maxWidth: 480, padding: 16 }}>
      <label htmlFor="compose">Compose</label>
      <textarea
        id="compose"
        rows={5}
        value={text}
        onChange={handleChange}
        onPaste={handlePaste}
        aria-describedby="char-count"
        style={{ width: "100%", padding: 8, boxSizing: "border-box" }}
      />
      <div
        id="char-count"
        role="status"
        aria-live="polite"
        style={{ marginTop: 8, ...zoneStyles[zone] }}
      >
        {counterLabel}
      </div>
      {zone === "blocked" && (
        <p style={{ fontSize: 12, color: "#c0392b" }}>
          Remove {len - MAX + 1} character(s) to post.
        </p>
      )}
    </div>
  );
}
```

---

## 4. useCharacterLimit Hook

```tsx
import { useCallback, useMemo, useState } from "react";

export function useCharacterLimit(max: number, initial = "") {
  const [text, setText] = useState(initial);
  const length = text.length;
  const remaining = max - length;
  const isFull = length >= max;

  const setCapped = useCallback(
    (value: string) => setText(value.slice(0, max)),
    [max]
  );

  const append = useCallback(
    (chunk: string) => setText((prev) => (prev + chunk).slice(0, max)),
    [max]
  );

  const zone = useMemo(() => {
    if (length >= max) return "blocked" as const;
    if (length >= max * 0.95) return "danger" as const;
    if (length >= max * 0.8) return "warn" as const;
    return "safe" as const;
  }, [length, max]);

  return { text, setText: setCapped, append, length, remaining, isFull, zone };
}
```

---

## 5. React Interview Points

| Question | Answer |
|----------|--------|
| Controlled vs uncontrolled | Controlled: `value` + `onChange` — needed for live count |
| Why intercept paste? | Browser may paste over limit before onChange fires |
| Debounce count? | Usually no — count is cheap; debounce autosave instead |
| Variable window meaning | Threshold zones (80/95/100%), not sliding-window DSA |
| a11y | `aria-live="polite"` on counter |

---

# Part B — React Native

## 6. Problem — Windowed Feed Loader

Load a large list in **batches** as user scrolls — only fetch/render a **window** of items around the viewport.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Initial batch | Load first 20 items |
| On end reached | Fetch next batch (cursor/page) |
| Batch merge | Append without duplicating IDs |
| Pull to refresh | Reset window to batch 1 |
| Loading states | Footer spinner, empty state |

---

## 7. Batch Update Strategy

```ts
type Item = { id: string; title: string };

type Page = { items: Item[]; nextCursor: string | null };

async function fetchPage(cursor?: string): Promise<Page> {
  const res = await fetch(
    `https://api.example.com/feed?cursor=${cursor ?? ""}&limit=20`
  );
  return res.json();
}

// Merge — dedupe by id (sliding window of loaded IDs in memory)
function mergeItems(prev: Item[], incoming: Item[]): Item[] {
  const seen = new Set(prev.map((i) => i.id));
  const unique = incoming.filter((i) => !seen.has(i.id));
  return [...prev, ...unique];
}
```

**Window mental model:** Visible viewport ≈ 5–8 rows; **loaded window** grows in batches; **virtualization** (Day 10) trims rendered nodes.

---

## 8. FlatList Windowed Loader

```tsx
import { useCallback, useState } from "react";
import {
  ActivityIndicator,
  FlatList,
  RefreshControl,
  Text,
  View,
} from "react-native";

type Item = { id: string; title: string };

export default function WindowedFeed() {
  const [items, setItems] = useState<Item[]>([]);
  const [cursor, setCursor] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const loadBatch = useCallback(async (reset = false) => {
    if (loading) return;
    setLoading(true);
    try {
      const page = await fetchPage(reset ? undefined : cursor ?? undefined);
      setItems((prev) =>
        reset ? page.items : mergeItems(prev, page.items)
      );
      setCursor(page.nextCursor);
      setHasMore(page.nextCursor != null);
    } finally {
      setLoading(false);
    }
  }, [cursor, loading]);

  const onRefresh = useCallback(async () => {
    setRefreshing(true);
    setCursor(null);
    setHasMore(true);
    await loadBatch(true);
    setRefreshing(false);
  }, [loadBatch]);

  const onEndReached = useCallback(() => {
    if (hasMore && !loading) loadBatch(false);
  }, [hasMore, loading, loadBatch]);

  return (
    <FlatList
      data={items}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <View style={{ padding: 16, borderBottomWidth: 1, borderColor: "#eee" }}>
          <Text>{item.title}</Text>
        </View>
      )}
      onEndReached={onEndReached}
      onEndReachedThreshold={0.3}
      ListFooterComponent={
        loading ? <ActivityIndicator style={{ padding: 16 }} /> : null
      }
      refreshControl={
        <RefreshControl refreshing={refreshing} onRefresh={onRefresh} />
      }
      ListEmptyComponent={
        !loading ? <Text style={{ padding: 16 }}>No items</Text> : null
      }
    />
  );
}

async function fetchPage(cursor?: string) {
  // mock — replace with API
  await new Promise((r) => setTimeout(r, 400));
  const start = cursor ? Number(cursor) : 0;
  const batch = Array.from({ length: 20 }, (_, i) => ({
    id: String(start + i),
    title: `Post ${start + i}`,
  }));
  const next = start + 20 >= 100 ? null : String(start + 20);
  return { items: batch, nextCursor: next };
}

function mergeItems(prev: Item[], incoming: Item[]) {
  const seen = new Set(prev.map((i) => i.id));
  return [...prev, ...incoming.filter((i) => !seen.has(i.id))];
}
```

### Optimizations (mention in interview)

- `initialNumToRender={10}`, `maxToRenderPerBatch={10}`, `windowSize={5}`
- Cancel in-flight fetch on unmount (`AbortController`)
- Optimistic batch prepend for pull-to-refresh

---

## 9. RN Interview Points

| Question | Answer |
|----------|--------|
| `onEndReachedThreshold` | Fraction of list length from end (0.3 = 30%) |
| Duplicate keys? | Dedupe merge; stable `keyExtractor` |
| Memory with 10k items | Pair with virtualization; don't map all to DOM |
| Refresh vs pagination | Reset cursor + items vs append |

---

## Quick Revision — Day 8

```
React:     zones at 80/95/100%, controlled textarea, paste cap
RN:        FlatList + cursor pagination + mergeItems dedupe
Link:      variable UI window ↔ sliding window DSA (Week 2)
```

---

*End of Day 8 machine coding*
