# Day 19 — Machine Coding Revision

**React:** Array intersection UI  
**React Native:** List diff highlighter

---

## Table of Contents

### React (Web)

1. [Problem — Array Intersection UI](#1-problem--array-intersection-ui)
2. [Set-Based Intersection Logic](#2-set-based-intersection-logic)
3. [React Implementation](#3-react-implementation)
4. [React Interview Points](#4-react-interview-points)

### React Native

5. [List Diff Highlighter](#5-list-diff-highlighter)
6. [RN Interview Points](#6-rn-interview-points)

---

# Part A — React (Web)

## 1. Problem — Array Intersection UI

**Task:** Two multi-select lists (Team A / Team B). Show **intersection**, **A-only**, **B-only** with color coding. Updates live as user toggles members.

**Requirements:**

| Feature | Behavior |
|---------|----------|
| Two panels | Checkbox lists of names |
| Venn summary | Intersection / A \ B / B \ A counts |
| Highlight | Shared names in both panels green |
| Performance | Set ops O(n) on toggle |

---

## 2. Set-Based Intersection Logic

```js
function analyzeSets(setA, setB) {
  const intersection = new Set([...setA].filter((x) => setB.has(x)));
  const aOnly = new Set([...setA].filter((x) => !setB.has(x)));
  const bOnly = new Set([...setB].filter((x) => !setA.has(x)));
  return { intersection, aOnly, bOnly };
}
```

### Intersection II style (with counts) — LC 350 tie-in

```js
function intersectWithCounts(a, b) {
  const freq = new Map();
  for (const x of b) freq.set(x, (freq.get(x) ?? 0) + 1);
  const res = [];
  for (const x of a) {
    const c = freq.get(x) ?? 0;
    if (c > 0) {
      res.push(x);
      freq.set(x, c - 1);
    }
  }
  return res;
}
```

---

## 3. React Implementation

```tsx
import { useMemo, useState } from "react";

const TEAM_A = ["Ada", "Bob", "Chen", "Dia"];
const TEAM_B = ["Bob", "Dia", "Eve", "Finn"];

export default function ArrayIntersectionUI() {
  const [selA, setSelA] = useState<Set<string>>(new Set(["Ada", "Bob"]));
  const [selB, setSelB] = useState<Set<string>>(new Set(["Bob", "Eve"]));

  const { intersection, aOnly, bOnly } = useMemo(
    () => analyzeSets(selA, selB),
    [selA, selB]
  );

  const toggle = (set: Set<string>, setter: typeof setSelA, name: string) => {
    setter((prev) => {
      const next = new Set(prev);
      if (next.has(name)) next.delete(name);
      else next.add(name);
      return next;
    });
  };

  return (
    <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 24, padding: 24 }}>
      <Panel
        title="Team A"
        items={TEAM_A}
        selected={selA}
        highlight={intersection}
        onToggle={(n) => toggle(selA, setSelA, n)}
      />
      <Panel
        title="Team B"
        items={TEAM_B}
        selected={selB}
        highlight={intersection}
        onToggle={(n) => toggle(selB, setSelB, n)}
      />
      <div style={{ gridColumn: "1 / -1" }}>
        <h3>Summary</h3>
        <p>Intersection ({intersection.size}): {[...intersection].join(", ") || "—"}</p>
        <p>A only ({aOnly.size}): {[...aOnly].join(", ") || "—"}</p>
        <p>B only ({bOnly.size}): {[...bOnly].join(", ") || "—"}</p>
      </div>
    </div>
  );
}

function Panel({
  title,
  items,
  selected,
  highlight,
  onToggle,
}: {
  title: string;
  items: string[];
  selected: Set<string>;
  highlight: Set<string>;
  onToggle: (name: string) => void;
}) {
  return (
    <div>
      <h3>{title}</h3>
      {items.map((name) => (
        <label key={name} style={{ display: "block", marginBottom: 8 }}>
          <input
            type="checkbox"
            checked={selected.has(name)}
            onChange={() => onToggle(name)}
          />
          <span
            style={{
              marginLeft: 8,
              color: highlight.has(name) && selected.has(name) ? "#2e7d32" : "#000",
              fontWeight: highlight.has(name) ? 600 : 400,
            }}
          >
            {name}
          </span>
        </label>
      ))}
    </div>
  );
}

function analyzeSets(setA: Set<string>, setB: Set<string>) {
  const intersection = new Set([...setA].filter((x) => setB.has(x)));
  const aOnly = new Set([...setA].filter((x) => !setB.has(x)));
  const bOnly = new Set([...setB].filter((x) => !setA.has(x)));
  return { intersection, aOnly, bOnly };
}
```

---

## 4. React Interview Points

| Question | Answer |
|----------|--------|
| Set in state | New Set on each toggle — immutable update |
| useMemo for analysis? | Cheap O(n) but avoids recompute on unrelated renders |
| Array vs Set in state | Set for membership; convert for render `[...set]` |

---

# Part B — React Native

## 5. List Diff Highlighter

**Task:** Show two lists (old vs new sync). Highlight **added** (green), **removed** (red strikethrough), **unchanged** (default).

```tsx
import { FlatList, Text, View, StyleSheet } from "react-native";

type Item = { id: string; label: string };

function diffLists(oldItems: Item[], newItems: Item[]) {
  const oldMap = new Map(oldItems.map((x) => [x.id, x]));
  const newMap = new Map(newItems.map((x) => [x.id, x]));

  const added = newItems.filter((x) => !oldMap.has(x.id));
  const removed = oldItems.filter((x) => !newMap.has(x.id));
  const kept = newItems.filter((x) => oldMap.has(x.id));

  return { added, removed, kept };
}

export default function ListDiffHighlighter({
  oldItems,
  newItems,
}: {
  oldItems: Item[];
  newItems: Item[];
}) {
  const { added, removed, kept } = diffLists(oldItems, newItems);

  const rows = [
    ...removed.map((x) => ({ ...x, status: "removed" as const })),
    ...kept.map((x) => ({ ...x, status: "kept" as const })),
    ...added.map((x) => ({ ...x, status: "added" as const })),
  ];

  return (
    <FlatList
      data={rows}
      keyExtractor={(item) => `${item.status}-${item.id}`}
      renderItem={({ item }) => (
        <View style={styles.row}>
          <Text
            style={[
              item.status === "added" && styles.added,
              item.status === "removed" && styles.removed,
            ]}
          >
            {item.label} {item.status !== "kept" && `(${item.status})`}
          </Text>
        </View>
      )}
    />
  );
}

const styles = StyleSheet.create({
  row: { padding: 12, borderBottomWidth: 1, borderColor: "#eee" },
  added: { color: "#2e7d32", fontWeight: "600" },
  removed: { color: "#c62828", textDecorationLine: "line-through" },
});
```

### LCS / Myers diff (mention)

Production sync UIs use **longest common subsequence** for minimal edit script — Set/Map diff suffices for id-keyed lists.

---

## 6. RN Interview Points

| Question | Answer |
|----------|--------|
| Diff by id vs label | Always stable `id` |
| Animation on add | `LayoutAnimation` or Reanimated entering |
| FlashList | Drop-in for large diff lists |

---

## Quick Revision — Day 19

```
Web:  two Set state → intersection / aOnly / bOnly → Venn UI
RN:   Map id lookup → added/removed/kept → color styles
Both: O(n) Set/Map; LC 349/350 patterns
```

---

*End of Day 19 machine coding*
