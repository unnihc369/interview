# Day 28 — Machine Coding Revision

**React:** Heap autocomplete  
**React Native:** Background priority queue

---

## Table of Contents

### React (Web)

1. [Problem — Heap Autocomplete](#1-problem--heap-autocomplete)
2. [Top-K Prefix Suggestions](#2-top-k-prefix-suggestions)
3. [React Autocomplete UI](#3-react-autocomplete-ui)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [Background Priority Queue](#5-background-priority-queue)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Heap Autocomplete

**Task:** Search input with autocomplete. Suggestions ranked by **popularity score** — use min-heap size K to keep top matches for current prefix.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Prefix filter | case-insensitive startsWith |
| Rank | Higher score first |
| Top 5 | Heap/sort suggestions |
| Keyboard | Arrow up/down, Enter select |

---

## 2. Top-K Prefix Suggestions

```js
const CORPUS = [
  { term: "javascript", score: 100 },
  { term: "java", score: 80 },
  { term: "json", score: 60 },
  { term: "jest", score: 50 },
  { term: "jsx", score: 45 },
  { term: "jquery", score: 40 },
];

function topKMatches(query, k = 5) {
  const q = query.toLowerCase();
  const matches = CORPUS.filter((x) => x.term.startsWith(q));
  return matches
    .sort((a, b) => b.score - a.score || a.term.localeCompare(b.term))
    .slice(0, k);
}
```

### Trie + heap (production)

Trie for prefix lookup O(m); heap for top-K among branch results.

---

## 3. React Autocomplete UI

```tsx
import { useState, useMemo, useRef, useEffect } from "react";

const CORPUS = [
  { term: "javascript", score: 100 },
  { term: "java", score: 80 },
  { term: "json", score: 60 },
  { term: "jest", score: 50 },
  { term: "jsx", score: 45 },
  { term: "jquery", score: 40 },
];

export default function HeapAutocomplete() {
  const [query, setQuery] = useState("");
  const [active, setActive] = useState(0);
  const listRef = useRef<HTMLUListElement>(null);

  const suggestions = useMemo(() => {
    if (!query) return [];
    const q = query.toLowerCase();
    return CORPUS.filter((x) => x.term.startsWith(q))
      .sort((a, b) => b.score - a.score)
      .slice(0, 5);
  }, [query]);

  useEffect(() => setActive(0), [query]);

  const onKeyDown = (e: React.KeyboardEvent) => {
    if (!suggestions.length) return;
    if (e.key === "ArrowDown") {
      e.preventDefault();
      setActive((i) => Math.min(i + 1, suggestions.length - 1));
    } else if (e.key === "ArrowUp") {
      e.preventDefault();
      setActive((i) => Math.max(i - 1, 0));
    } else if (e.key === "Enter") {
      setQuery(suggestions[active].term);
    }
  };

  return (
    <div style={{ maxWidth: 360, margin: 24, position: "relative" }}>
      <h2>Autocomplete (Top-K)</h2>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        onKeyDown={onKeyDown}
        placeholder="Type ja…"
        style={{ width: "100%", padding: 10, fontSize: 16 }}
        aria-autocomplete="list"
        aria-expanded={suggestions.length > 0}
      />
      {suggestions.length > 0 && (
        <ul
          ref={listRef}
          role="listbox"
          style={{
            listStyle: "none",
            margin: 0,
            padding: 0,
            border: "1px solid #ccc",
            position: "absolute",
            width: "100%",
            background: "#fff",
            zIndex: 1,
          }}
        >
          {suggestions.map((s, i) => (
            <li
              key={s.term}
              role="option"
              aria-selected={i === active}
              onMouseDown={() => setQuery(s.term)}
              style={{
                padding: "8px 12px",
                background: i === active ? "#e3f2fd" : "#fff",
                cursor: "pointer",
              }}
            >
              {s.term} <span style={{ color: "#888", fontSize: 12 }}>({s.score})</span>
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Debounce autocomplete | 150–300ms for API; local corpus instant |
| onMouseDown vs onClick | mousedown before blur closes list |
| a11y | combobox pattern, aria-activedescendant |

---

# Part B — React Native

## 5. Background Priority Queue

**Task:** When app backgrounds, queue **sync jobs** by priority. On foreground, process min-heap order (critical first).

```tsx
import { useEffect, useRef, useState } from "react";
import { AppState, View, Text, Pressable, StyleSheet } from "react-native";

type Job = { id: string; label: string; priority: number };

export default function BackgroundPriorityQueue() {
  const [queue, setQueue] = useState<Job[]>([]);
  const [log, setLog] = useState<string[]>([]);
  const appState = useRef(AppState.currentState);

  useEffect(() => {
    const sub = AppState.addEventListener("change", (next) => {
      if (appState.current.match(/inactive|background/) && next === "active") {
        flushQueue();
      }
      appState.current = next;
    });
    return () => sub.remove();
  }, [queue]);

  const flushQueue = () => {
    setQueue((q) => {
      const sorted = [...q].sort((a, b) => a.priority - b.priority);
      sorted.forEach((j) =>
        setLog((l) => [`Ran: ${j.label} (P${j.priority})`, ...l.slice(0, 9)])
      );
      return [];
    });
  };

  const enqueue = (label: string, priority: number) => {
    setQueue((q) =>
      [...q, { id: String(Date.now()), label, priority }].sort(
        (a, b) => a.priority - b.priority
      )
    );
  };

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Background Job Queue</Text>
      <Text>Pending: {queue.length}</Text>
      <Pressable style={styles.btn} onPress={() => enqueue("Analytics flush", 2)}>
        <Text>Queue P2 job</Text>
      </Pressable>
      <Pressable style={styles.btn} onPress={() => enqueue("Auth refresh", 1)}>
        <Text>Queue P1 job</Text>
      </Pressable>
      <Pressable style={styles.btn} onPress={flushQueue}>
        <Text>Flush now (foreground)</Text>
      </Pressable>
      {log.map((l, i) => (
        <Text key={i} style={styles.log}>{l}</Text>
      ))}
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 16 },
  title: { fontSize: 18, fontWeight: "700", marginBottom: 12 },
  btn: { padding: 12, backgroundColor: "#eee", marginTop: 8, borderRadius: 8 },
  log: { fontSize: 12, color: "#666", marginTop: 4 },
});
```

### TaskManager / BackgroundFetch

Mention `expo-background-fetch` for OS-level background execution limits.

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| AppState | active / background / inactive |
| iOS background limits | ~30s unless audio/location |
| Priority on resume | Flush heap before UI interactive |

---

## Quick Revision — Day 28

```
Web:  prefix filter → score sort → top 5 autocomplete
RN:   AppState foreground → flush priority queue
Mock: 215, 347, 295 timed
```

---

*End of Day 28 machine coding*
