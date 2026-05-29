# Day 14 — JavaScript & TypeScript Revision (Sunday)

**Topics:** Week 2 consolidation · Sliding window ↔ frontend patterns · Mock prep

---

## Table of Contents

1. [Week 2 Concept Recap](#1-week-2-concept-recap)
2. [DSA ↔ UI "Window" Mapping](#2-dsa--ui-window-mapping)
3. [Mock Interview Checklist](#3-mock-interview-checklist)
4. [Rapid Fire Q&A](#4-rapid-fire-qa)
5. [Day 14 Study Plan](#5-day-14-study-plan)

---

## 1. Week 2 Concept Recap

| Day | JS/TS | Machine coding | DSA |
|-----|-------|----------------|-----|
| 8 | map/filter/reduce | char counter zones | 3, 904, 76 |
| 9 | currying, partial | ResizeObserver | 438, 567, 480 |
| 10 | Proxy, Reflect | virtual list | 209, 1004, 992 |
| 11 | event delegation | debounced stats | 713, 1456, 1590 |
| 12 | memoize, LRU | LRU visualizer | 487, 1297, 1052 |
| 13 | WeakMap, WeakSet | price chart SMA | revision |
| 14 | **mock** | 10k list + tabs | 3, 76, 480 timed |

---

## 2. DSA ↔ UI "Window" Mapping

| UI pattern | Sliding window idea |
|------------|---------------------|
| Virtual list viewport | Only render indices [start, end] |
| FlatList `windowSize` | Off-screen buffer around viewport |
| Image preload ±1 | Fixed window on slide index |
| SMA chart | Fixed window on last N ticks |
| LRU access log `slice(-8)` | Last 8 ops window |
| Debounced scroll stats | Visible range [scrollTop, scrollTop + height] |

**Interview line:** "I treat the viewport as a sliding window over the data array — same two-pointer intuition as substring problems."

---

## 3. Mock Interview Checklist

### JS (10 min)

- [ ] Implement `curry(fn)` 
- [ ] Explain WeakMap vs Map with DOM example
- [ ] Event delegation with `closest`
- [ ] Proxy validation trap

### Machine coding (45 min)

- [ ] Virtualized list OR character counter from scratch
- [ ] Mention a11y + performance
- [ ] RN: FlatList tuning props if mobile role

### DSA (45 min timed)

- [ ] **3** — 15 min
- [ ] **76** — 20 min
- [ ] **480** — 25 min (approach + partial code OK)

---

## 4. Rapid Fire Q&A

**Q: When is sliding window not suitable?**  
A: Need all subarrays with complex non-monotonic constraint; DP on non-contiguous; negative numbers break simple product window.

**Q: `useMemo` vs memoize?**  
A: `useMemo` caches render computation; `memoize` caches pure function calls.

**Q: Why ResizeObserver for dashboard?**  
A: Container width changes without window resize (sidebar, split pane).

**Q: FlatList blank flash?**  
A: Increase `windowSize`, `initialNumToRender`; use `getItemLayout` for fixed height.

**Q: `React.memo` vs `useMemo` vs `useCallback`?**  
A: memo = skip component re-render; useMemo = cache value; useCallback = cache function ref.

**Q: Core Web Vitals?**  
A: LCP (load), INP (interaction), CLS (layout shift) — see [performance supplement](../supplements/sde1_react_performance.md).

---

## 5b. Performance & a11y Reminders

Before mock — review:

- [ ] Virtualized list for 10k rows ([day14_machine_coding](day14_machine_coding.md))
- [ ] `React.memo` + stable callbacks for list rows
- [ ] Modal/Dropdown keyboard support ([machine_coding_classics](../supplements/sde1_machine_coding_classics.md))
- [ ] `aria-live` for dynamic search results ([a11y supplement](../supplements/sde1_accessibility.md))

---

## 5. Day 14 Study Plan

| Time | Activity |
|------|----------|
| AM | Re-read [day13_leetcode](../day_13/day13_leetcode.md) templates |
| AM | Machine mock: optimize 10k list (day14_machine_coding) |
| PM | Timed LC mock (day14_leetcode) |
| PM | 30-min behavioral + explain one project |

---

## Day 14 Cheat Sheet

```
Review:  templates + Week 2 map + UI↔DSA parallels
Mock:    1 web component + 3 LC timed + explain tradeoffs
Weak:    drill 480 / 992 extra if time
```

---

*End of Day 14 revision*
