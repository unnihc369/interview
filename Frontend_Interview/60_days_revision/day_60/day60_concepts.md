# Day 60 — Full Mock Interview & Final Revision

**Week 8 Finale** · **Topics:** DP mastery review · Interview simulation · Pattern consolidation

---

## Table of Contents

1. [60-Day Journey Recap](#1-60-day-journey-recap)
2. [DP Pattern Master Index](#2-dp-pattern-master-index)
3. [Mock Interview Structure (2 Hours)](#3-mock-interview-structure-2-hours)
4. [Behavioral + System Design Warm-Up](#4-behavioral--system-design-warm-up)
5. [Coding Round Strategy](#5-coding-round-strategy)
6. [Complexity Cheat Sheet](#6-complexity-cheat-sheet)
7. [Final Revision Checklist](#7-final-revision-checklist)
8. [Post-60-Day Maintenance Plan](#8-post-60-day-maintenance-plan)

---

## 1. 60-Day Journey Recap

| Week | Theme | Days |
|------|-------|------|
| 1 | Stacks & Queues | 1–7 |
| 2 | Sliding Window | 8–14 |
| 3 | Two Pointers | 15–21 |
| 4 | Heaps | 22–28 |
| 5 | Trees | 29–35 |
| 6 | BST | 36–42 |
| 7 | Tries | 43–49 |
| 8 | Dynamic Programming | 50–60 |

---

## 2. DP Pattern Master Index

| Pattern | Days | Key LC |
|---------|------|--------|
| Memoization 1D | 50 | 509, 746 |
| Grid tabulation | 51 | 62, 64 |
| Knapsack | 52 | 416, 322 |
| LCS / edit | 53, 59 | 1143, 72 |
| LIS | 54 | 300, 354 |
| Tree DP | 55 | 337, 124 |
| Bitmask | 56 | 847 |
| Rolling / stock | 57 | 121–123 |
| Interval | 58 | 312 |
| String DP | 59 | 115, 97 |

### Recognition signals

| Phrase in problem | Likely pattern |
|-------------------|----------------|
| "Count ways" | DP count, often 0/1 knapsack |
| "Minimum cost/path" | Grid or knapsack |
| "Longest subsequence" | LIS or LCS |
| "Two strings" | 2D string DP |
| "Tree, no adjacent" | Tree DP |
| "Visit all nodes, small n" | Bitmask |
| "Burst / split range" | Interval DP |
| "At most k transactions" | Stock rolling |

---

## 3. Mock Interview Structure (2 Hours)

### Round 1 — DSA (45 min)

1. **5 min** — Clarify problem, examples, edge cases.
2. **5 min** — Brute force + identify overlapping subproblems.
3. **25 min** — Implement optimal (memo or tabulation).
4. **10 min** — Test, state complexity, discuss follow-ups.

### Round 2 — Machine Coding (45 min)

1. **5 min** — Requirements, component breakdown.
2. **30 min** — Working UI with core feature.
3. **10 min** — Polish, accessibility, edge cases.

### Round 3 — JS/React Deep Dive (30 min)

- Closures, event loop, React reconciliation, hooks rules.
- One question from Weeks 1–4 concepts.

---

## 4. Behavioral + System Design Warm-Up

**STAR format:** Situation → Task → Action → Result (2 min each).

Sample prompts:

- "Tell me about a performance optimization you shipped."
- "How do you handle conflicting PR feedback?"
- "Design a typeahead with 10M queries/day." (Trie + cache + debounce)

---

## 5. Coding Round Strategy

### DP interview script

```
1. "This has optimal substructure because..."
2. "State: dp(i) = ..."
3. "Base case: ..."
4. "Transition: ..."
5. "I'll start top-down with memo, then tabulate if time."
6. "Time O(...), space O(...), can optimize to O(...) with rolling array."
```

### When stuck

- Draw small example (n=4).
- Write brute recursion first.
- Look for knapsack / LCS / interval template match.

---

## 6. Complexity Cheat Sheet

| Pattern | Typical time | Typical space |
|---------|--------------|---------------|
| 1D DP | O(n) | O(n) → O(1) |
| 2D grid | O(m×n) | O(m×n) → O(n) |
| Knapsack | O(n×W) | O(W) |
| LIS n log n | O(n log n) | O(n) |
| Tree DP | O(n) | O(h) stack |
| Bitmask | O(2^n × n) | O(2^n × n) |
| Interval | O(n³) | O(n²) |

---

## 7. Final Revision Checklist

### DSA (Week 8)

- [ ] Write Fibonacci memo + tabulation from memory
- [ ] Unique paths with obstacles
- [ ] Partition equal subset sum
- [ ] LCS and edit distance
- [ ] LIS O(n log n)
- [ ] House Robber III
- [ ] Stock III (4 variables)
- [ ] Burst balloons interval template

### Machine Coding

- [ ] Build autocomplete with debounce (Week 7 tie-in)
- [ ] Implement drag-and-drop list (Week 3)
- [ ] RN FlatList with pull-to-refresh
- [ ] Error boundary + loading states
- [ ] **Modal + Dropdown** from [classics supplement](../supplements/sde1_machine_coding_classics.md)
- [ ] **Toast notification** system

### JavaScript / React / Security

- [ ] Event loop: microtasks vs macrotasks
- [ ] `useMemo` vs `useCallback` vs memoization DP
- [ ] Map/Set complexity
- [ ] Generator iteration
- [ ] XSS / CSRF / token storage ([security supplement](../supplements/sde1_frontend_security.md))
- [ ] CORS + AbortController ([networking supplement](../supplements/sde1_frontend_networking.md))
- [ ] TanStack Query basics ([state supplement](../supplements/sde1_state_management.md))
- [ ] RTL test one component ([testing supplement](../supplements/sde1_testing_rtl.md))

### TypeScript

- [ ] Pick / Omit / Partial / Record ([TS utilities](../supplements/sde1_typescript_utilities.md))
- [ ] Discriminated unions for API state

### SDE 1–2 Supplement Index

| File | Topics |
|------|--------|
| [sde1_frontend_security.md](../supplements/sde1_frontend_security.md) | XSS, CSRF, CSP, tokens |
| [sde1_react_internals_patterns.md](../supplements/sde1_react_internals_patterns.md) | VDOM, hooks, patterns |
| [sde1_react_performance.md](../supplements/sde1_react_performance.md) | memo, Core Web Vitals |
| [sde1_frontend_networking.md](../supplements/sde1_frontend_networking.md) | HTTP, CORS, WebSocket |
| [sde1_testing_rtl.md](../supplements/sde1_testing_rtl.md) | Jest, RTL |
| [sde1_state_management.md](../supplements/sde1_state_management.md) | Redux, Zustand, RQ |
| [sde1_machine_coding_classics.md](../supplements/sde1_machine_coding_classics.md) | Modal, Table, Toast |
| [sde1_typescript_utilities.md](../supplements/sde1_typescript_utilities.md) | Pick, Omit, satisfies |
| [sde1_accessibility.md](../supplements/sde1_accessibility.md) | ARIA, keyboard |
| [sde1_ssr_nextjs.md](../supplements/sde1_ssr_nextjs.md) | SSR, hydration, Next.js |

---

## 8. Post-60-Day Maintenance Plan

| Frequency | Activity |
|-----------|----------|
| Daily | 1 LC medium (15 min) |
| Weekly | 1 timed mock (3 problems) |
| Monthly | 1 full machine coding build |
| Before interview | Review this checklist + Day 60 LeetCode mock |

---

**Next:** [Machine Coding — Full Stack Project](day60_machine_coding.md) · [LeetCode Mock Contest](day60_leetcode.md)
