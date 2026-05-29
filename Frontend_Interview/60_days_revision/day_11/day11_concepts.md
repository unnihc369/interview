# Day 11 — JavaScript & TypeScript Revision

**Topics:** Event delegation · Bubbling & capturing · Performance patterns

---

## Table of Contents

1. [Event Propagation](#1-event-propagation)
2. [Event Delegation](#2-event-delegation)
3. [Implementation Patterns](#3-implementation-patterns)
4. [React Event Delegation](#4-react-event-delegation)
5. [Interview Quick Index](#5-interview-quick-index)

---

## 1. Event Propagation

### Three phases

```
Capture (root → target) → Target → Bubble (target → root)
```

```js
element.addEventListener("click", handler);              // bubble (default)
element.addEventListener("click", handler, true);        // capture
element.addEventListener("click", handler, { capture: true });
```

### stopPropagation vs stopImmediatePropagation

| Method | Effect |
|--------|--------|
| `stopPropagation()` | Stops further propagation on this event type |
| `stopImmediatePropagation()` | Also blocks other listeners on same node |
| `preventDefault()` | Blocks default browser action (not propagation) |

---

## 2. Event Delegation

### Idea

Attach **one listener** on parent; handle events from children via `event.target`.

**Benefits:**

- Fewer listeners (memory + setup cost)
- Works for **dynamically added** children
- Centralized handler logic

```html
<ul id="list">
  <li data-id="1">Apple</li>
  <li data-id="2">Banana</li>
</ul>
```

```js
document.getElementById("list").addEventListener("click", (e) => {
  const li = e.target.closest("li");
  if (!li || !list.contains(li)) return;
  console.log("Selected", li.dataset.id);
});
```

### `closest()` vs `matches()`

```js
// closest — walk up from target
e.target.closest("[data-action]");

// matches — test target only
if (e.target.matches("button.delete")) { /* ... */ }
```

---

## 3. Implementation Patterns

### Delegated click actions

```js
function delegate(root, selector, event, handler) {
  root.addEventListener(event, (e) => {
    const match = e.target.closest(selector);
    if (match && root.contains(match)) {
      handler.call(match, e, match);
    }
  });
}

delegate(document.body, "[data-copy]", "click", (e, el) => {
  navigator.clipboard.writeText(el.dataset.copy);
});
```

### Table row selection

```js
tbody.addEventListener("click", (e) => {
  const row = e.target.closest("tr[data-row-id]");
  if (!row) return;
  row.classList.toggle("selected");
});
```

### Pitfalls

| Pitfall | Fix |
|---------|-----|
| Click on nested icon inside button | Use `closest("button")` |
| Hover on parent not child | Delegate `mouseenter` won't bubble — use `mouseover` or direct listeners |
| Shadow DOM | `event.composedPath()` |

---

## 4. React Event Delegation

React 17+ attaches listeners to **root container**, not `document` — still delegation internally.

```tsx
function TodoList({ items, onDelete }: Props) {
  const handleClick = (e: React.MouseEvent<HTMLUListElement>) => {
    const btn = (e.target as HTMLElement).closest("[data-delete-id]");
    if (!btn) return;
    onDelete(btn.getAttribute("data-delete-id")!);
  };

  return (
    <ul onClick={handleClick}>
      {items.map((item) => (
        <li key={item.id}>
          {item.text}
          <button type="button" data-delete-id={item.id}>
            ×
          </button>
        </li>
      ))}
    </ul>
  );
}
```

### React vs vanilla

| | Vanilla delegation | React |
|---|-------------------|-------|
| Dynamic nodes | Natural | Re-render updates DOM; delegation still valid |
| Synthetic events | N/A | Pooled/normalized in older React; 17+ native-like |
| Prefer | Large static lists | Usually explicit handlers per row is fine unless 1000+ rows |

---

## 5. Interview Quick Index

| Question | Section |
|----------|---------|
| Bubbling vs capturing | [§1](#1-event-propagation) |
| What is event delegation? | [§2](#2-event-delegation) |
| Why use it? | [§2](#2-event-delegation) |
| `closest` role | [§2–§3](#3-implementation-patterns) |
| React delegation | [§4](#4-react-event-delegation) |

---

## Day 11 Cheat Sheet

```
Propagation → capture → target → bubble
Delegation  → one parent listener + event.target.closest
Dynamic UI  → delegation avoids rebinding listeners
React 17+   → listeners on root; still synthetic events
```

---

*End of Day 11 revision*
