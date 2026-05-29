# Day 24 — requestAnimationFrame

**Week 4 — Heaps / Scheduling** · **Topics:** rAF · vs setTimeout · FPS · Median streaming · Animation loop

---

## Table of Contents

1. [requestAnimationFrame Basics](#1-requestanimationframe-basics)
2. [Animation Loop Pattern](#2-animation-loop-pattern)
3. [rAF vs setTimeout vs setInterval](#3-raf-vs-settimeout-vs-setinterval)
4. [cancelAnimationFrame](#4-cancelanimationframe)
5. [FPS & Delta Time](#5-fps--delta-time)
6. [Streaming Median Tie-In](#6-streaming-median-tie-in)
7. [React Integration](#7-react-integration)
8. [Interview Quick Index](#8-interview-quick-index)

---

## 1. requestAnimationFrame Basics

### Syntax

```js
const id = requestAnimationFrame(callback);
// callback receives DOMHighResTimeStamp (ms since navigation start)
```

Browser schedules callback **before next repaint** (~60fps = ~16.67ms), synced with display refresh.

```js
requestAnimationFrame((timestamp) => {
  console.log("Frame at", timestamp);
});
```

### Why use rAF?

| Benefit | Detail |
|---------|--------|
| Smooth animation | Aligned with GPU compositor |
| Battery | Pauses in background tabs (most browsers) |
| No drift | Adapts to refresh rate (120Hz displays) |

---

## 2. Animation Loop Pattern

```js
let start = null;

function step(timestamp) {
  if (!start) start = timestamp;
  const elapsed = timestamp - start;
  const progress = Math.min(elapsed / 1000, 1); // 1 second anim
  element.style.transform = `translateX(${progress * 200}px)`;
  if (progress < 1) requestAnimationFrame(step);
}

requestAnimationFrame(step);
```

### Recursive loop

```js
function loop(timestamp) {
  update(timestamp);
  render();
  requestAnimationFrame(loop);
}
requestAnimationFrame(loop);
```

---

## 3. rAF vs setTimeout vs setInterval

| API | Timing | Use |
|-----|--------|-----|
| **rAF** | Before paint ~16ms | Visual updates, DOM reads/writes batch |
| **setTimeout** | Fixed delay macrotask | Debounce, delayed work |
| **setInterval** | Fixed repeat | Polling (prefer rAF for UI) |

### Don't animate with setInterval(16)

Interval ignores frame budget, causes jank, runs in background tabs.

---

## 4. cancelAnimationFrame

```js
let frameId;
function animate() {
  // ...
  frameId = requestAnimationFrame(animate);
}
cancelAnimationFrame(frameId);
```

React cleanup:

```js
useEffect(() => {
  let id;
  const tick = (t) => { /* ... */ id = requestAnimationFrame(tick); };
  id = requestAnimationFrame(tick);
  return () => cancelAnimationFrame(id);
}, []);
```

---

## 5. FPS & Delta Time

```js
let last = 0;
function gameLoop(now) {
  const dt = (now - last) / 1000; // seconds since last frame
  last = now;
  position += velocity * dt; // frame-rate independent
  requestAnimationFrame(gameLoop);
}
```

### Measure FPS

```js
let frames = 0, lastFpsTime = 0;
function measure(now) {
  frames++;
  if (now - lastFpsTime >= 1000) {
    console.log("FPS:", frames);
    frames = 0;
    lastFpsTime = now;
  }
  requestAnimationFrame(measure);
}
```

---

## 6. Streaming Median Tie-In

**Median tracker** (Day 24 MC): two heaps — max-heap for lower half, min-heap for upper half. rAF batches chart redraws when new samples arrive.

```js
class MedianFinder {
  constructor() {
    this.lo = []; // max-heap (negate for JS)
    this.hi = []; // min-heap
  }
  addNum(num) {
    this.lo.push(-num);
    this.lo.sort((a, b) => a - b);
    this.hi.push(-this.lo.shift());
    this.hi.sort((a, b) => a - b);
    if (this.hi.length > this.lo.length) {
      this.lo.push(-this.hi.shift());
      this.lo.sort((a, b) => a - b);
    }
  }
  findMedian() {
    if (this.lo.length > this.hi.length) return -this.lo[0];
    return (-this.lo[0] + this.hi[0]) / 2;
  }
}
```

Use rAF to coalesce multiple `addNum` calls into one chart update per frame.

---

## 7. React Integration

### Prefer CSS transitions for simple UI

rAF for canvas, custom physics, or latency graph (LC 295 visualizer).

```tsx
useLayoutEffect(() => {
  let id: number;
  const draw = (t: number) => {
    canvasRef.current && paintChart(t);
    id = requestAnimationFrame(draw);
  };
  id = requestAnimationFrame(draw);
  return () => cancelAnimationFrame(id);
}, [data]);
```

### react-spring / framer-motion

Use libraries internally optimized — mention you know rAF under the hood.

---

## 8. Interview Quick Index

| Question | Section |
|----------|---------|
| rAF vs setInterval animate | [§3](#3-raf-vs-settimeout-vs-setinterval) |
| Background tab behavior | [§1](#1-requestanimationframe-basics) — paused |
| Frame-rate independent motion | [§5](#5-fps--delta-time) — delta time |
| Cleanup | [§4](#4-cancelanimationframe) |

---

## Day 24 Cheat Sheet

```
requestAnimationFrame(cb) → before next paint; ~60fps
cancelAnimationFrame(id)  → stop loop
Use for: DOM animation, canvas, charts
Avoid: setInterval(16) for visual updates
Delta time: position += v * dt
Median stream: two heaps + rAF batch render
```

---

*End of Day 24 concepts*
