# Day 5 — Machine Coding Revision

**React:** Debounced search · request queue · cancellation  
**React Native:** Throttled scroll FlatList · infinite scroll

---

## Table of Contents

### React (Web)

1. [Problem — Debounced Search with Queue](#1-problem--debounced-search-with-queue)
2. [Debounce + AbortController Pattern](#2-debounce--abortcontroller-pattern)
3. [Request Queue with Cancellation](#3-request-queue-with-cancellation)
4. [Full Search Component](#4-full-search-component)
5. [React Interview Points](#5-react-interview-points)

### React Native

6. [Problem — Throttled Infinite Scroll](#6-problem--throttled-infinite-scroll)
7. [useThrottledCallback Hook](#7-usethrottledcallback-hook)
8. [FlatList Infinite Scroll](#8-flatlist-infinite-scroll)
9. [Full RN Solution](#9-full-rn-solution)
10. [RN Interview Points](#10-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Debounced Search with Queue

**Task:** Build a search input that:
- **Debounces** user input (300ms)
- Queues API requests sequentially OR cancels stale requests
- Shows loading, results, and error states
- Handles rapid typing without race conditions

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Debounce | Wait 300ms after last keystroke |
| Cancel stale | Abort previous fetch when new search starts |
| Loading state | Spinner while fetching |
| Empty state | Message when no results |
| Error handling | Show error with retry |

---

## 2. Debounce + AbortController Pattern

```tsx
import { useCallback, useEffect, useRef, useState } from "react";

type SearchResult = { id: string; title: string };

function useDebouncedSearch(query: string, delay = 300) {
  const [results, setResults] = useState<SearchResult[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const abortRef = useRef<AbortController | null>(null);
  const debounceRef = useRef<ReturnType<typeof setTimeout>>();

  useEffect(() => {
    if (!query.trim()) {
      setResults([]);
      setError(null);
      return;
    }

    debounceRef.current = setTimeout(async () => {
      // Cancel previous request
      abortRef.current?.abort();
      abortRef.current = new AbortController();

      setLoading(true);
      setError(null);

      try {
        const res = await fetch(
          `https://jsonplaceholder.typicode.com/posts?q=${encodeURIComponent(query)}`,
          { signal: abortRef.current.signal }
        );
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const data: SearchResult[] = await res.json();
        // Filter client-side for demo (API doesn't actually filter)
        const filtered = data
          .filter((p) =>
            p.title.toLowerCase().includes(query.toLowerCase())
          )
          .slice(0, 10)
          .map((p) => ({ id: String(p.id), title: p.title }));
        setResults(filtered);
      } catch (err) {
        if (err instanceof DOMException && err.name === "AbortError") return;
        setError(err instanceof Error ? err.message : "Search failed");
        setResults([]);
      } finally {
        setLoading(false);
      }
    }, delay);

    return () => {
      clearTimeout(debounceRef.current);
      abortRef.current?.abort();
    };
  }, [query, delay]);

  return { results, loading, error };
}
```

---

## 3. Request Queue with Cancellation

For interviews requiring **sequential queue** instead of abort:

```tsx
type QueuedRequest = {
  id: number;
  query: string;
  abort: AbortController;
};

function useSearchQueue() {
  const [results, setResults] = useState<SearchResult[]>([]);
  const [loading, setLoading] = useState(false);
  const queueRef = useRef<QueuedRequest[]>([]);
  const processingRef = useRef(false);
  const requestIdRef = useRef(0);

  const processQueue = useCallback(async () => {
    if (processingRef.current || queueRef.current.length === 0) return;
    processingRef.current = true;

    while (queueRef.current.length > 0) {
      const req = queueRef.current.shift()!;
      setLoading(true);

      try {
        const res = await fetch(
          `https://jsonplaceholder.typicode.com/posts?userId=1`,
          { signal: req.abort.signal }
        );
        const data = await res.json();
        const filtered = data
          .filter((p: { title: string }) =>
            p.title.toLowerCase().includes(req.query.toLowerCase())
          )
          .slice(0, 5)
          .map((p: { id: number; title: string }) => ({
            id: String(p.id),
            title: p.title,
          }));

        // Only apply if this is still the latest request
        if (req.id === requestIdRef.current) {
          setResults(filtered);
        }
      } catch (err) {
        if (!(err instanceof DOMException && err.name === "AbortError")) {
          console.error(err);
        }
      }
    }

    setLoading(false);
    processingRef.current = false;
  }, []);

  const enqueueSearch = useCallback(
    (query: string) => {
      // Cancel all pending in queue
      queueRef.current.forEach((r) => r.abort.abort());
      queueRef.current = [];

      const id = ++requestIdRef.current;
      const abort = new AbortController();
      queueRef.current.push({ id, query, abort });
      processQueue();
    },
    [processQueue]
  );

  return { results, loading, enqueueSearch };
}
```

---

## 4. Full Search Component

```tsx
export default function DebouncedSearch() {
  const [query, setQuery] = useState("");
  const { results, loading, error } = useDebouncedSearch(query);

  return (
    <div style={{ maxWidth: 500, margin: "0 auto", padding: 24 }}>
      <h2>Debounced Search</h2>

      <div style={{ position: "relative" }}>
        <input
          type="text"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Search posts..."
          style={{
            width: "100%",
            padding: "12px 16px",
            fontSize: 16,
            borderRadius: 8,
            border: "1px solid #ccc",
          }}
        />
        {loading && (
          <span style={{ position: "absolute", right: 12, top: 12 }}>
            ⏳
          </span>
        )}
      </div>

      {error && (
        <p style={{ color: "red", marginTop: 8 }}>
          {error}
          <button onClick={() => setQuery(query + " ")} style={{ marginLeft: 8 }}>
            Retry
          </button>
        </p>
      )}

      {!loading && query && results.length === 0 && !error && (
        <p style={{ color: "#666", marginTop: 16 }}>No results for "{query}"</p>
      )}

      <ul style={{ listStyle: "none", padding: 0, marginTop: 16 }}>
        {results.map((r) => (
          <li
            key={r.id}
            style={{
              padding: 12,
              borderBottom: "1px solid #eee",
            }}
          >
            {r.title}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Race condition explanation (interview gold)

```
User types "a" → request A starts
User types "ab" → request B starts, A aborted
User types "abc" → request C starts, B aborted
Only C's response applied → no stale "a" results
```

---

## 5. React Interview Points

| Question | Answer |
|----------|--------|
| Debounce in useEffect? | setTimeout in effect; cleanup clears timeout |
| AbortController purpose | Cancel in-flight fetch when query changes |
| AbortError handling | Ignore — expected when cancelling |
| Debounce vs queue? | Debounce reduces calls; queue serializes remaining |
| lodash.debounce? | OK to mention; show you know impl |
| stale closure in debounce? | useRef for latest callback |

### Checklist

- [ ] 300ms debounce on input
- [ ] AbortController cancels stale requests
- [ ] Loading / error / empty states
- [ ] Cleanup on unmount
- [ ] No race condition on results

---

# Part B — React Native

## 6. Problem — Throttled Infinite Scroll

**Task:** FlatList with **throttled** `onEndReached` for infinite scroll pagination.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Throttle | Max one loadMore per 500ms |
| Pagination | Load 20 items per page |
| Footer loader | ActivityIndicator while loading |
| End reached | Stop when no more data |
| Pull to refresh | Reset list |

---

## 7. useThrottledCallback Hook

```tsx
import { useCallback, useRef } from "react";

function useThrottledCallback<T extends (...args: never[]) => void>(
  callback: T,
  delay: number
): T {
  const lastCallRef = useRef(0);
  const timeoutRef = useRef<ReturnType<typeof setTimeout>>();
  const callbackRef = useRef(callback);
  callbackRef.current = callback;

  return useCallback(
    ((...args: Parameters<T>) => {
      const now = Date.now();
      const remaining = delay - (now - lastCallRef.current);

      if (remaining <= 0) {
        lastCallRef.current = now;
        callbackRef.current(...args);
      } else if (!timeoutRef.current) {
        timeoutRef.current = setTimeout(() => {
          lastCallRef.current = Date.now();
          timeoutRef.current = undefined;
          callbackRef.current(...args);
        }, remaining);
      }
    }) as T,
    [delay]
  );
}
```

---

## 8. FlatList Infinite Scroll

```tsx
function useInfiniteList(pageSize = 20) {
  const [items, setItems] = useState<Item[]>([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);
  const [hasMore, setHasMore] = useState(true);

  const loadPage = useCallback(async (pageNum: number) => {
    setLoading(true);
    await new Promise((r) => setTimeout(r, 600)); // simulate API

    const newItems = Array.from({ length: pageSize }, (_, i) => ({
      id: String((pageNum - 1) * pageSize + i + 1),
      title: `Item ${(pageNum - 1) * pageSize + i + 1}`,
    }));

    setItems((prev) => (pageNum === 1 ? newItems : [...prev, ...newItems]));
    setPage(pageNum);
    if (pageNum >= 5) setHasMore(false); // 5 pages max for demo
    setLoading(false);
  }, [pageSize]);

  const loadMore = useCallback(() => {
    if (loading || !hasMore) return;
    loadPage(page + 1);
  }, [loading, hasMore, page, loadPage]);

  const refresh = useCallback(() => {
    setHasMore(true);
    loadPage(1);
  }, [loadPage]);

  useEffect(() => {
    loadPage(1);
  }, []);

  return { items, loading, hasMore, loadMore, refresh };
}
```

---

## 9. Full RN Solution

```tsx
import React, { useCallback } from "react";
import {
  View,
  Text,
  FlatList,
  ActivityIndicator,
  RefreshControl,
  StyleSheet,
} from "react-native";

type Item = { id: string; title: string };

function ListItem({ item }: { item: Item }) {
  return (
    <View style={styles.item}>
      <Text style={styles.itemText}>{item.title}</Text>
    </View>
  );
}

export default function InfiniteScrollList() {
  const { items, loading, hasMore, loadMore, refresh } = useInfiniteList(20);
  const throttledLoadMore = useThrottledCallback(loadMore, 500);

  const renderFooter = useCallback(() => {
    if (!loading) return null;
    return (
      <View style={styles.footer}>
        <ActivityIndicator size="large" color="#007AFF" />
      </View>
    );
  }, [loading]);

  return (
    <FlatList
      data={items}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <ListItem item={item} />}
      onEndReached={throttledLoadMore}
      onEndReachedThreshold={0.5}
      ListFooterComponent={renderFooter}
      ListEmptyComponent={
        !loading ? (
          <Text style={styles.empty}>No items</Text>
        ) : null
      }
      refreshControl={
        <RefreshControl refreshing={loading && items.length === 0} onRefresh={refresh} />
      }
      contentContainerStyle={styles.list}
    />
  );
}

const styles = StyleSheet.create({
  list: { paddingVertical: 8 },
  item: {
    padding: 16,
    borderBottomWidth: 1,
    borderBottomColor: "#eee",
  },
  itemText: { fontSize: 16 },
  footer: { paddingVertical: 20 },
  empty: { textAlign: "center", padding: 32, color: "#999" },
});
```

### onEndReached gotchas

```tsx
// Guard against multiple fires
const loadMore = useCallback(() => {
  if (loading || !hasMore) return;  // essential
  // ...
}, [loading, hasMore]);

// onEndReachedThreshold: 0.5 = trigger when 50% from bottom
```

---

## 10. RN Interview Points

| Question | Answer |
|----------|--------|
| Why throttle onEndReached? | Fires multiple times rapidly on fast scroll |
| FlatList vs ScrollView | FlatList virtualizes — required for long lists |
| getItemLayout optimization | Skip measurement if fixed item height |
| maxToRenderPerBatch | Tune for scroll performance |
| RefreshControl | Pull down to reset pagination |

### Checklist

- [ ] Throttled loadMore (500ms)
- [ ] Loading footer indicator
- [ ] hasMore guard
- [ ] Pull to refresh
- [ ] keyExtractor on FlatList

---

## Quick Revision — Day 5 Machine Coding

```
React:
  debounce in useEffect + cleanup
  AbortController per fetch; abort on new query
  Ignore AbortError
  loading / error / empty UI

React Native:
  useThrottledCallback for onEndReached
  Guard: loading || !hasMore
  FlatList + ListFooterComponent spinner
  refreshControl for reset
```

---

*End of Day 5 machine coding*
