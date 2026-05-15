# React Complete Notes (Basics → Advanced)

## Table of Contents

1. [Introduction to React](#1-introduction-to-react)
2. [Advantages of React](#2-advantages-of-react)
3. [React vs HTML](#3-react-vs-html)
4. [React vs Next.js](#4-react-vs-nextjs)
5. [Library vs Framework](#5-library-vs-framework)
6. [Key Features of React](#6-key-features-of-react)
7. [JSX in Detail](#7-jsx-in-detail)
8. [DOM and Virtual DOM](#8-dom-and-virtual-dom)
9. [How Virtual DOM Works](#9-how-virtual-dom-works)
10. [React Reconciliation Algorithm](#10-react-reconciliation-algorithm)
11. [Diffing Algorithm in Detail](#11-diffing-algorithm-in-detail)
12. [React Rendering Flow](#12-react-rendering-flow)
13. [Render Phase vs Commit Phase](#13-render-phase-vs-commit-phase)
14. [React Fiber Architecture](#14-react-fiber-architecture)
15. [Optimizations React Does Internally](#15-optimizations-react-does-internally)
16. [useEffect and Reconciliation](#16-how-useeffect-fits-into-reconciliation)
17. [Class Components vs Functional Components](#17-class-components-vs-functional-components)
18. [setState vs useState](#18-setstate-vs-usestate)
19. [Challenges While Migrating Class → Functional](#19-challenges-while-migrating-class--functional)
20. [When Class Components Still Have Advantages](#20-when-class-components-still-have-advantages)
21. [Limitations of React Reconciliation](#21-limitations-of-react-reconciliation)
22. [Optimizing Large Scale React Applications](#22-optimizing-large-scale-react-applications)
23. [Real DOM vs Virtual DOM](#23-real-dom-vs-virtual-dom)
24. [Important Interview Questions Summary](#24-important-interview-questions-summary)

---

## 1. Introduction to React

React is a **JavaScript library** used to build User Interfaces (UI), especially **Single Page Applications (SPA)**.

- Developed by **Meta Platforms (Facebook)**
- Lets developers create **reusable UI components** and efficiently update the UI when data changes

**Example:**

```jsx
function App() {
  return <h1>Hello React</h1>;
}
```

---

## 2. Advantages of React

### 1. Component-Based Architecture

UI is divided into reusable components.

```jsx
<Header />
<Sidebar />
<Content />
<Footer />
```

**Benefits:** Reusability, maintainability, easier debugging, better scalability

### 2. Virtual DOM Improves Performance

React updates only changed parts instead of re-rendering the entire page. This reduces reflows, repaints, and expensive DOM operations.

### 3. Declarative Syntax

You describe **what** the UI should look like; React handles **how** to update it.

```jsx
const isLoggedIn = true;

return (
  <div>
    {isLoggedIn ? <Home /> : <Login />}
  </div>
);
```

### 4. Reusable Components

```jsx
<Button text="Save" />
<Button text="Delete" />
```

### 5. One-Way Data Binding

Data flows **Parent → Child**. Predictable data flow improves debugging.

```jsx
<Child name={userName} />
```

### 6. Hooks Support

Hooks allow state and lifecycle management in functional components: `useState`, `useEffect`, `useMemo`, `useCallback`, `useRef`

### 7. Strong Ecosystem

Huge community, libraries, DevTools, state management support

### 8. SEO Support with SSR

Using frameworks like **Next.js** improves SEO via Server-Side Rendering.

---

## 3. React vs HTML

| Feature         | HTML              | React                    |
|-----------------|-------------------|--------------------------|
| Type            | Markup Language   | JavaScript Library       |
| Dynamic UI      | Difficult         | Easy                     |
| Reusability     | No                | Yes                      |
| State Management| No                | Yes                      |
| Components      | No                | Yes                      |
| DOM Updates     | Manual            | Automatic                |
| Routing         | Manual            | Library based            |
| Performance     | Slower (dynamic)  | Faster (Virtual DOM)     |

---

## 4. React vs Next.js

| Feature              | React        | Next.js              |
|----------------------|--------------|----------------------|
| Type                 | UI Library   | Full-stack Framework |
| Routing              | Manual       | Built-in             |
| SSR                  | Not built-in | Supported            |
| SEO                  | Limited      | Excellent            |
| Backend APIs         | No           | Yes                  |
| File-based routing   | No           | Yes                  |
| Static Site Generation | No         | Yes                  |
| Full-stack support   | No           | Yes                  |

### When to Use React

- Building SPA
- Internal dashboards
- Client-side apps

### When to Use Next.js

- SEO is important
- SSR needed
- Full-stack app needed
- Fast loading required

---

## 5. Library vs Framework

### Library

A library gives tools. **You** control the flow.

**Example:** React — you call the library.

### Framework

The framework controls the flow. The framework calls **your** code.

**Examples:** Next.js, Angular

This is called **Inversion of Control**.

---

## 6. Key Features of React

### Component-Based Architecture

```jsx
function Header() {
  return <h1>Header</h1>;
}
```

### Declarative UI

```jsx
return count > 0 ? <Success /> : <Empty />;
```

### JSX

JavaScript XML — HTML-like syntax inside JavaScript.

```jsx
const element = <h1>Hello</h1>;
```

### One-Way Data Binding

```jsx
<Profile name="John" />
```

### React Hooks

```jsx
const [count, setCount] = useState(0);
```

### Virtual DOM

React creates lightweight copies of the DOM, compares old vs new, and updates only changed nodes.

---

## 7. JSX in Detail

### What is JSX?

**JSX = JavaScript XML**

```jsx
const element = <h1>Hello</h1>;
```

Internally converted to:

```js
React.createElement("h1", null, "Hello");
```

### Why JSX?

- Easier UI writing
- Better readability
- HTML-like syntax
- JavaScript power inside UI

### JSX vs HTML

| HTML              | JSX                          |
|-------------------|------------------------------|
| `class`           | `className`                  |
| `for`             | `htmlFor`                    |
| `onclick`         | `onClick`                    |
| `style="color:red"` | `style={{ color: "red" }}` |

```jsx
<label htmlFor="name">Name</label>
```

### Why JSX is Considered Type-Safe

JSX catches many errors at compile time (missing closing tags, invalid expressions). With TypeScript:

```tsx
type Props = { name: string };

function User({ name }: Props) {
  return <h1>{name}</h1>;
}
```

Incorrect props show **compile-time** errors.

### JSX Prevents XSS

React automatically escapes values.

```jsx
const userInput = "<script>alert(1)</script>";
return <div>{userInput}</div>;
```

Output: text is escaped — no script execution.

### dangerouslySetInnerHTML

Used to inject raw HTML (risky — XSS if untrusted):

```jsx
<div dangerouslySetInnerHTML={{ __html: html }} />
```

Use **only** for trusted HTML (CMS, markdown renderers).

---

## 8. DOM and Virtual DOM

### What is DOM?

**DOM = Document Object Model** — browser converts HTML into a tree.

```html
<body>
  <div>
    <h1>Hello</h1>
  </div>
</body>
```

```
body
 └── div
      └── h1
```

### Actual DOM Problems

Updating the DOM is expensive: layout, reflow, repaint, composite.

### What is Virtual DOM?

Lightweight **JavaScript object** representation of the Real DOM.

```js
{
  type: "div",
  props: { children: "Hello" }
}
```

### Why Virtual DOM is Lightweight

Contains: element type, props, children, attributes.

Does **not** contain: layout info, paint info, CSS engine details.

Creation and comparison in memory are fast.

---

## 9. How Virtual DOM Works

### Step 1: Initial Render

React creates Virtual DOM tree and Real DOM tree.

### Step 2: State Change

```jsx
setCount(count + 1);
```

Triggers new Virtual DOM creation.

### Step 3: Diffing

React compares **Old Virtual DOM** vs **New Virtual DOM** → **Reconciliation**

### Step 4: Find Differences

```jsx
// Old: <h1>Hello</h1>
// New: <h1>Hello World</h1>
// Only text changed
```

### Step 5: Update Real DOM

React updates only the changed node — not the entire page.

### Step 6: Browser Paint

Browser performs layout → paint → composite (minimal work).

---

## 10. React Reconciliation Algorithm

**Reconciliation** = process React uses to compare old and new Virtual DOM trees for efficient UI updates.

### Why Not O(n³)?

General tree comparison is O(n³). For 1000 elements: 1,000,000,000 comparisons — too slow for real-time UI.

### React Optimization → O(n)

**Assumption 1:** Same element types have similar structure.

```jsx
<div>Hello</div>
<div>World</div>
```

React compares children directly.

**Assumption 2:** Different types → different trees.

```jsx
<div />  →  <span />
```

React destroys old tree and creates new one.

**Assumption 3:** Keys identify stable siblings.

```jsx
users.map(user => <User key={user.id} />)
```

### DFS Traversal Example

```
App
 ├── Header
 ├── Main
 │    ├── Card1
 │    └── Card2
 └── Footer
```

Traversal: App → Header → Main → Card1 → Card2 → Footer  
**Time complexity:** O(n) — each node visited once.

---

## 11. Diffing Algorithm in Detail

### Case 1: Different Element Type

Old: `<div />` → New: `<span />`  
React removes old DOM and creates new DOM (entire subtree recreated).

### Case 2: Same Element Type

Old: `<div className="a" />` → New: `<div className="b" />`  
React updates only changed attributes.

### Case 3: Text Content Changed

Old: `<h1>Hello</h1>` → New: `<h1>Hello World</h1>`  
Only text node updated.

### Case 4: Lists with Keys

**Without keys** — inserting at start makes React think all items changed (bad performance).

**With keys:**

```jsx
<li key="1">A</li>
<li key="2">B</li>
```

React identifies nodes correctly → efficient updates.

---

## 12. React Rendering Flow

```
State Change
    ↓
Render Phase
    ↓
Virtual DOM Creation
    ↓
Diffing / Reconciliation
    ↓
Commit Phase
    ↓
Real DOM Update
    ↓
Browser Layout → Paint → Composite
```

### Browser Rendering Pipeline

1. **Parse HTML** → DOM tree  
2. **Parse CSS** → CSSOM tree  
3. **Render Tree** — DOM + CSSOM  
4. **Layout** — positions and sizes  
5. **Paint** — pixels drawn  
6. **Composite** — layers on screen  

---

## 13. Render Phase vs Commit Phase

### Render Phase

- Calls components
- Creates Virtual DOM
- Calculates changes
- **No DOM updates yet**
- Can be **interrupted** (React 18)

### Commit Phase

- Updates actual DOM
- Runs effects
- **Cannot be interrupted**

---

## 14. React Fiber Architecture

**Fiber** is React’s modern reconciliation engine for better scheduling, concurrent rendering, and interruptible work.

### Before Fiber

Rendering was synchronous — large updates blocked the UI.

### Fiber Features

1. **Incremental rendering** — work split into chunks  
2. **Priority scheduling** — e.g. typing > background render  
3. **Pause and resume** rendering  
4. **Concurrent rendering** (React 18): `startTransition`, `Suspense`

### Fiber Node Structure

Each component becomes a Fiber node with: `type`, `props`, `state`, `child`, `sibling`, `parent`

---

## 15. Optimizations React Does Internally

### Reconciliation and Diffing

Efficient tree comparison — **O(n)** complexity.

### Batching Updates

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
// Single re-render, final count +2
```

### Memoization

**React.memo** — prevents unnecessary re-render:

```jsx
export default React.memo(Button);
```

**useMemo** — caches expensive calculations:

```jsx
const total = useMemo(() => calc(), [deps]);
```

**useCallback** — caches function references:

```jsx
const onClick = useCallback(() => {}, [deps]);
```

### Lazy Loading & Code Splitting

```jsx
const Home = React.lazy(() => import("./Home"));
```

### Concurrent Rendering

Prioritizes important updates.

### Fragments

```jsx
<>
  <h1 />
</>
```

Avoids extra DOM nodes.

### Suspense

```jsx
<Suspense fallback={<Loader />}>
  <App />
</Suspense>
```

### Skipping Unnecessary Re-renders

`memo`, `PureComponent`, `shouldComponentUpdate`

---

## 16. How useEffect Fits into Reconciliation

`useEffect` runs **after** the commit phase.

```
Render Phase
    ↓
Commit DOM Changes
    ↓
Browser Paint
    ↓
useEffect Runs
```

```jsx
useEffect(() => {
  console.log("DOM updated");
}, []);
```

### useLayoutEffect

Runs **before** browser paint — useful for measurements and animations.

---

## 17. Class Components vs Functional Components

| Feature          | Class Component      | Functional Component |
|------------------|----------------------|----------------------|
| Syntax           | Complex              | Simple               |
| State            | `this.state`         | `useState`           |
| Lifecycle        | lifecycle methods    | hooks                |
| `this` keyword   | Required             | Not needed           |
| Performance      | Slightly heavier     | Lightweight          |
| Reusability      | Lower                | Higher               |
| Logic sharing    | HOC / Render Props   | Hooks                |

### Class Component Example

```jsx
class App extends React.Component {
  state = { count: 0 };

  render() {
    return <h1>{this.state.count}</h1>;
  }
}
```

### Functional Component Example

```jsx
function App() {
  const [count, setCount] = useState(0);
  return <h1>{count}</h1>;
}
```

---

## 18. setState vs useState

### `this.setState`

```jsx
this.setState({ count: 1 });
```

- Merges state partially  
- Async batching  
- Class-based  

### `useState`

```jsx
setCount(1);
```

- Replaces state (or functional update)  
- Hooks-based  

### Batching Example

```jsx
setCount(c => c + 1);
setCount(c => c + 1);
// Single render, final count +2
```

---

## 19. Challenges While Migrating Class → Functional

1. **Lifecycle conversion** — `componentDidMount` → `useEffect(() => {}, [])`  
2. **`this` removal** — no `this.handleClick`  
3. **State splitting** — one big object → smaller hook states  
4. **Memoization** — need `useCallback` / `useMemo` when functions recreate each render  
5. **Dependency arrays** — wrong deps → infinite loops or **stale closures**

---

## 20. When Class Components Still Have Advantages

- **Explicit lifecycle** — `componentDidMount`, `componentDidUpdate`, `componentWillUnmount` for complex flows  
- **Error boundaries** — still mostly class-based (`componentDidCatch`)  
- **Legacy enterprise apps** — large codebases not yet migrated  

---

## 21. Limitations of React Reconciliation

1. **Inefficient list handling** — bad keys → re-renders, state mismatch  
2. **Unnecessary re-renders** — parent update re-renders children (use `memo`, `useMemo`, `useCallback`)  
3. **Loss of state** — remounting resets state  
4. **Large component trees** — longer reconciliation time  
5. **Animation challenges** — use Framer Motion, React Spring  
6. **Context re-renders** — all consumers update; mitigate with splitting, memoization, Zustand/Redux selectors  

---

## 22. Optimizing Large Scale React Applications

- **Proper keys** — never use array index for dynamic lists  
- **Memoization** — `React.memo`, `useMemo`, `useCallback` (use carefully)  
- **Virtualization** — `react-window`, `react-virtualized`  
- **Code splitting** — `React.lazy`, `Suspense`  
- **State colocation** — keep state close to usage  
- **Avoid frequent re-renders** — selectors, split components  
- **Concurrent features** — `startTransition`, `Suspense`  

---

## 23. Real DOM vs Virtual DOM

| Feature        | Real DOM              | Virtual DOM           |
|----------------|-----------------------|------------------------|
| Speed          | Slower                | Faster                 |
| Updates        | Entire tree may update| Minimal updates        |
| Memory         | Heavy                 | Lightweight            |
| Operations     | Expensive             | Cheap (in-memory)      |
| Re-rendering   | Full recalculation    | Selective              |
| Manipulation   | Direct browser work   | JS objects + diffing   |

### Why Real DOM is Slow

Layout recalculation, paint, reflow, composite.

### Why Virtual DOM is Faster

In-memory objects, efficient diffing, minimal real DOM updates.

---

## 24. Important Interview Questions Summary

Questions are grouped by level. Each item links to the section where the answer is covered in detail.

### Basics

| Question | Answer in section |
|----------|-------------------|
| [What is React?](#1-introduction-to-react) | [§1 Introduction to React](#1-introduction-to-react) |
| [Why use React?](#2-advantages-of-react) | [§2 Advantages of React](#2-advantages-of-react) |
| [What is JSX?](#7-jsx-in-detail) | [§7 JSX in Detail](#7-jsx-in-detail) |
| [What is Virtual DOM?](#8-dom-and-virtual-dom) | [§8 DOM and Virtual DOM](#8-dom-and-virtual-dom) · [§9 How Virtual DOM Works](#9-how-virtual-dom-works) |
| [Difference between Real DOM and Virtual DOM?](#23-real-dom-vs-virtual-dom) | [§23 Real DOM vs Virtual DOM](#23-real-dom-vs-virtual-dom) |
| [What are hooks?](#6-key-features-of-react) | [§6 Key Features of React](#6-key-features-of-react) (Hooks) · [§17 Class vs Functional](#17-class-components-vs-functional-components) |

### Intermediate

| Question | Answer in section |
|----------|-------------------|
| [Explain reconciliation.](#10-react-reconciliation-algorithm) | [§10 React Reconciliation Algorithm](#10-react-reconciliation-algorithm) |
| [Explain diffing algorithm.](#11-diffing-algorithm-in-detail) | [§11 Diffing Algorithm in Detail](#11-diffing-algorithm-in-detail) |
| [What is batching?](#15-optimizations-react-does-internally) | [§15 Optimizations](#15-optimizations-react-does-internally) · [§18 setState vs useState](#18-setstate-vs-usestate) |
| [Difference between useEffect and useLayoutEffect?](#16-how-useeffect-fits-into-reconciliation) | [§16 useEffect and Reconciliation](#16-how-useeffect-fits-into-reconciliation) |
| [Explain React Fiber.](#14-react-fiber-architecture) | [§14 React Fiber Architecture](#14-react-fiber-architecture) |
| [Explain React rendering lifecycle.](#12-react-rendering-flow) | [§12 React Rendering Flow](#12-react-rendering-flow) · [§13 Render vs Commit Phase](#13-render-phase-vs-commit-phase) |

### Advanced

| Question | Answer in section |
|----------|-------------------|
| [How does React achieve O(n) diffing?](#10-react-reconciliation-algorithm) | [§10 Reconciliation](#10-react-reconciliation-algorithm) · [§11 Diffing](#11-diffing-algorithm-in-detail) |
| [Explain concurrent rendering.](#14-react-fiber-architecture) | [§14 React Fiber Architecture](#14-react-fiber-architecture) |
| [How does React scheduling work?](#14-react-fiber-architecture) | [§14 React Fiber Architecture](#14-react-fiber-architecture) (priority, pause/resume) |
| [How to optimize large React apps?](#22-optimizing-large-scale-react-applications) | [§22 Optimizing Large Scale Apps](#22-optimizing-large-scale-react-applications) |
| [Explain memoization deeply.](#15-optimizations-react-does-internally) | [§15 Optimizations](#15-optimizations-react-does-internally) |
| [Why are keys important?](#11-diffing-algorithm-in-detail) | [§11 Diffing — Lists with Keys](#11-diffing-algorithm-in-detail) |
| [Explain render phase vs commit phase.](#13-render-phase-vs-commit-phase) | [§13 Render Phase vs Commit Phase](#13-render-phase-vs-commit-phase) |
| [How does React prevent unnecessary renders?](#15-optimizations-react-does-internally) | [§15 Optimizations](#15-optimizations-react-does-internally) |
| [What are stale closures?](#19-challenges-while-migrating-class--functional) | [§19 Migration Challenges](#19-challenges-while-migrating-class--functional) (dependency arrays) |
| [Explain React internals and Fiber architecture.](#14-react-fiber-architecture) | [§14 React Fiber Architecture](#14-react-fiber-architecture) · [§12 Rendering Flow](#12-react-rendering-flow) |

### Quick revision checklist

- [ ] Can explain Virtual DOM flow end-to-end → [§9](#9-how-virtual-dom-works)  
- [ ] Can draw reconciliation + diffing cases → [§10](#10-react-reconciliation-algorithm), [§11](#11-diffing-algorithm-in-detail)  
- [ ] Know render vs commit + where `useEffect` runs → [§13](#13-render-phase-vs-commit-phase), [§16](#16-how-useeffect-fits-into-reconciliation)  
- [ ] Know when to use `memo` / `useMemo` / `useCallback` → [§15](#15-optimizations-react-does-internally), [§22](#22-optimizing-large-scale-react-applications)  

---

*End of document*
