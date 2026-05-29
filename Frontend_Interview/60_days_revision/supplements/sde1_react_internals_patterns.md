# SDE 1–2 Supplement — React Internals & Component Patterns

**Topics:** Virtual DOM · Reconciliation · Hooks rules · Error Boundaries · Lazy/Suspense · Compound components · HOC · Render props · Context

**Related days:** Day 1 (hooks), Day 2 (useReducer), Day 14 (performance)

---

## Table of Contents

1. [Virtual DOM & Reconciliation](#1-virtual-dom--reconciliation)
2. [The `key` Prop](#2-the-key-prop)
3. [Rules of Hooks](#3-rules-of-hooks)
4. [Controlled vs Uncontrolled](#4-controlled-vs-uncontrolled)
5. [useEffect Pitfalls](#5-useeffect-pitfalls)
6. [Error Boundaries](#6-error-boundaries)
7. [Code Splitting: lazy + Suspense](#7-code-splitting-lazy--suspense)
8. [React 18: Batching & Concurrent Features](#8-react-18-batching--concurrent-features)
9. [Component Patterns](#9-component-patterns)
10. [Interview Q&A](#10-interview-qa)

---

## 1. Virtual DOM & Reconciliation

React keeps a **lightweight JS tree** (Virtual DOM). On state change:

```text
State change → render() produces new VDOM tree
            → diff old vs new (reconciliation)
            → apply minimal DOM updates
```

**Fiber** (React 16+): reconciliation is incremental — work can pause/resume for responsiveness.

| Concept | Meaning |
|---------|---------|
| Render | Call component function → return JSX (describes UI) |
| Commit | Apply DOM changes to browser |
| Re-render | Parent re-render → child re-renders by default |

---

## 2. The `key` Prop

Keys help React identify **which list items changed**.

```tsx
// ❌ index as key — bugs on reorder/delete/insert
items.map((item, i) => <Row key={i} data={item} />);

// ✅ stable unique id
items.map((item) => <Row key={item.id} data={item} />);
```

**Bug with index key:** Delete first item → all keys shift → React reuses wrong component state.

---

## 3. Rules of Hooks

1. Only call hooks at **top level** (not in if/for/nested functions)
2. Only call from **React functions** (components or custom hooks)

```tsx
// ❌ conditional hook
if (loggedIn) useEffect(() => {}, []);

// ✅ hook always called; condition inside
useEffect(() => {
  if (!loggedIn) return;
  fetchUser();
}, [loggedIn]);
```

Custom hooks must start with `use` — they compose other hooks.

---

## 4. Controlled vs Uncontrolled

| | Controlled | Uncontrolled |
|---|------------|--------------|
| Value source | React state | DOM ref |
| Single source of truth | Yes | DOM is truth |
| Validation | Easy on change | On submit |

```tsx
// Controlled
const [email, setEmail] = useState("");
<input value={email} onChange={(e) => setEmail(e.target.value)} />

// Uncontrolled
const inputRef = useRef<HTMLInputElement>(null);
<input ref={inputRef} defaultValue="" />
// read: inputRef.current?.value
```

**Interview:** Prefer controlled for forms with validation; uncontrolled for simple file inputs or integrating non-React libs.

---

## 5. useEffect Pitfalls

```tsx
// Stale closure — count always logs old value if deps wrong
useEffect(() => {
  const id = setInterval(() => console.log(count), 1000);
  return () => clearInterval(id);
}, []); // ❌ missing count

// ✅ include count OR use functional update / ref
useEffect(() => {
  const id = setInterval(() => setCount((c) => c + 1), 1000);
  return () => clearInterval(id);
}, []);

// Cleanup on unmount — subscriptions, timers, AbortController
useEffect(() => {
  const ctrl = new AbortController();
  fetch(url, { signal: ctrl.signal });
  return () => ctrl.abort();
}, [url]);
```

| Dependency mistake | Symptom |
|--------------------|---------|
| Missing dep | Stale data |
| Object/array in deps | Infinite re-fetch (new ref each render) |
| No cleanup | Memory leak, race conditions |

---

## 6. Error Boundaries

Class component (or `react-error-boundary` lib) catching **render** errors in children.

```tsx
class ErrorBoundary extends React.Component<
  { fallback: React.ReactNode; children: React.ReactNode },
  { hasError: boolean }
> {
  state = { hasError: false };

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    logToService(error, info.componentStack);
  }

  render() {
    return this.state.hasError ? this.props.fallback : this.props.children;
  }
}

// Usage
<ErrorBoundary fallback={<p>Something went wrong</p>}>
  <UserDashboard />
</ErrorBoundary>
```

**Does NOT catch:** event handlers, async code, SSR errors, errors in boundary itself.

---

## 7. Code Splitting: lazy + Suspense

```tsx
import { lazy, Suspense } from "react";

const AdminPanel = lazy(() => import("./AdminPanel"));

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <AdminPanel />
    </Suspense>
  );
}
```

Webpack/Vite creates separate chunk — loaded on demand → smaller initial bundle.

---

## 8. React 18: Batching & Concurrent Features

**Automatic batching:** Multiple `setState` in same event handler → one re-render (even in async/setTimeout in React 18).

```tsx
import { useTransition, useDeferredValue } from "react";

// useTransition — mark update as non-urgent
const [isPending, startTransition] = useTransition();
startTransition(() => setFilteredItems(hugeList.filter(...)));

// useDeferredValue — lag behind fast-changing value
const deferredQuery = useDeferredValue(query);
const results = useMemo(() => search(deferredQuery), [deferredQuery]);
```

---

## 9. Component Patterns

### Compound Components

```tsx
const TabsContext = createContext<{ active: string; setActive: (id: string) => void } | null>(null);

function Tabs({ children, defaultId }: { children: React.ReactNode; defaultId: string }) {
  const [active, setActive] = useState(defaultId);
  return (
    <TabsContext.Provider value={{ active, setActive }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

function TabList({ children }: { children: React.ReactNode }) {
  return <div role="tablist">{children}</div>;
}

function Tab({ id, children }: { id: string; children: React.ReactNode }) {
  const ctx = useContext(TabsContext)!;
  return (
    <button role="tab" aria-selected={ctx.active === id} onClick={() => ctx.setActive(id)}>
      {children}
    </button>
  );
}

function TabPanel({ id, children }: { id: string; children: React.ReactNode }) {
  const ctx = useContext(TabsContext)!;
  if (ctx.active !== id) return null;
  return <div role="tabpanel">{children}</div>;
}

Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

// Usage
<Tabs defaultId="a">
  <Tabs.List>
    <Tabs.Tab id="a">Tab A</Tabs.Tab>
    <Tabs.Tab id="b">Tab B</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panel id="a">Content A</Tabs.Panel>
  <Tabs.Panel id="b">Content B</Tabs.Panel>
</Tabs>
```

### Render Props

```tsx
function MouseTracker({ render }: { render: (pos: { x: number; y: number }) => React.ReactNode }) {
  const [pos, setPos] = useState({ x: 0, y: 0 });
  useEffect(() => {
    const handler = (e: MouseEvent) => setPos({ x: e.clientX, y: e.clientY });
    window.addEventListener("mousemove", handler);
    return () => window.removeEventListener("mousemove", handler);
  }, []);
  return <>{render(pos)}</>;
}

<MouseTracker render={({ x, y }) => <p>Mouse: {x}, {y}</p>} />
```

### HOC (Higher-Order Component)

```tsx
function withAuth<P extends object>(Wrapped: React.ComponentType<P>) {
  return function AuthComponent(props: P) {
    const { user } = useAuth();
    if (!user) return <LoginPrompt />;
    return <Wrapped {...props} />;
  };
}

const ProtectedDashboard = withAuth(Dashboard);
```

**Modern preference:** Custom hooks (`useAuth`) over HOCs — simpler composition.

### Context + Provider

```tsx
const ThemeContext = createContext<"light" | "dark">("light");

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<"light" | "dark">("light");
  return (
    <ThemeContext.Provider value={theme}>
      {children}
      <button onClick={() => setTheme((t) => (t === "light" ? "dark" : "light"))}>Toggle</button>
    </ThemeContext.Provider>
  );
}
```

**Split contexts** by update frequency to avoid unnecessary re-renders.

---

## 10. Interview Q&A

**Q: Virtual DOM benefits?**  
A: Declarative UI, batched updates, cross-browser abstraction, diff minimizes DOM ops.

**Q: Why not mutate state directly?**  
A: React won't detect change → no re-render; breaks time-travel debugging.

**Q: forwardRef use case?**  
A: Pass ref to child DOM node (focus input, measure element).

**Q: Compound vs prop drilling?**  
A: Compound shares implicit state via context; flexible API like `<Select><Option /></Select>`.

---

*End of React Internals & Patterns Supplement*
