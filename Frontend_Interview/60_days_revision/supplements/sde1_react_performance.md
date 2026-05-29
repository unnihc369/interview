# SDE 1–2 Supplement — React Performance

**Topics:** React.memo · useMemo · useCallback · Profiler · Core Web Vitals · Bundle optimization

**Related days:** Day 12 (memoize), Day 14 (10k list optimization)

---

## 1. When Does React Re-render?

```text
State change in component     → that component re-renders
Props change                  → child re-renders
Parent re-renders             → ALL children re-render (default)
Context value change          → all consumers re-render
```

---

## 2. React.memo · useMemo · useCallback

| Tool | What it memoizes | When to use |
|------|------------------|-------------|
| `React.memo(Component)` | Entire component output | Child gets same props often; expensive render |
| `useMemo(() => compute(), [deps])` | **Value** from computation | Expensive filter/sort; stable object for deps |
| `useCallback(fn, [deps])` | **Function reference** | Pass callback to memoized child |

```tsx
const ExpensiveList = memo(function ExpensiveList({
  items,
  onSelect,
}: {
  items: Item[];
  onSelect: (id: string) => void;
}) {
  return items.map((item) => (
    <button key={item.id} onClick={() => onSelect(item.id)}>{item.name}</button>
  ));
});

function Parent() {
  const [items, setItems] = useState<Item[]>([]);
  const [filter, setFilter] = useState("");

  const filtered = useMemo(
    () => items.filter((i) => i.name.includes(filter)),
    [items, filter]
  );

  const handleSelect = useCallback((id: string) => {
    console.log(id);
  }, []);

  return <ExpensiveList items={filtered} onSelect={handleSelect} />;
}
```

### When NOT to memoize (interview trap)

- Cheap renders — memo overhead > savings
- Props always new reference anyway
- Premature optimization before measuring

**Rule:** Profile first → optimize hot paths.

---

## 3. Other Performance Patterns

```tsx
// 1. Lazy load routes
const Settings = lazy(() => import("./Settings"));

// 2. Virtualize long lists (Day 10, 14)
// @tanstack/react-virtual or react-window

// 3. Debounce/throttle expensive handlers (Day 5)

// 4. Split context — don't put everything in one Provider

// 5. Children as composition (avoid inline object props)
<Child config={CONFIG} />  // stable ref
// vs config={{ theme: "dark" }}  // new object every render
```

---

## 4. React DevTools Profiler

```text
Record → interact → stop → see flame graph
Look for: components with long render time, frequent re-renders
"Why did this render?" — props/state/context changed
```

Programmatic:

```tsx
import { Profiler } from "react";

<Profiler id="Search" onRender={(id, phase, actualDuration) => {
  if (actualDuration > 16) console.warn(`${id} slow: ${actualDuration}ms`);
}}>
  <SearchResults />
</Profiler>
```

Target: **< 16ms** per frame (60fps).

---

## 5. Core Web Vitals

| Metric | Measures | Good threshold |
|--------|----------|----------------|
| **LCP** (Largest Contentful Paint) | Main content load | ≤ 2.5s |
| **INP** (Interaction to Next Paint) | Responsiveness to input | ≤ 200ms |
| **CLS** (Cumulative Layout Shift) | Visual stability | ≤ 0.1 |

### Frontend fixes

| Vital | Fixes |
|-------|-------|
| LCP | Optimize hero image (WebP, preload, CDN), reduce blocking JS |
| INP | Break long tasks, use `startTransition`, Web Workers |
| CLS | Set width/height on images, reserve space for ads, avoid inserting content above fold |

**Lighthouse** in Chrome DevTools → Performance + Accessibility audit.

---

## 6. Bundle Optimization

| Technique | Effect |
|-----------|--------|
| Code splitting (`lazy`) | Smaller initial JS |
| Tree shaking | Remove unused exports (ES modules) |
| Dynamic imports | Load feature on demand |
| Analyze bundle | `vite-bundle-visualizer`, `webpack-bundle-analyzer` |
| Avoid huge libraries | date-fns per-function vs moment entire lib |

---

## Interview Q&A

**Q: useMemo vs useCallback?**  
A: useMemo caches a **value**; useCallback caches a **function**.

**Q: Does memo prevent all re-renders?**  
A: Only if props are shallow-equal to previous. New object/array props → still re-renders.

**Q: How to optimize 10k item list?**  
A: Virtualization — only render visible rows + buffer (Day 14).

---

*End of React Performance Supplement*
