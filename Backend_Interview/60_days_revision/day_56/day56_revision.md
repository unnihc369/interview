# Day 56 — Light Review + Rest

**Week 8 · Sunday · ~90 min light study, then rest**

Final day before interviews: **recall, not cram**. Sleep and confidence matter more than one extra problem.

---

# Schedule

| Time | Activity |
|------|----------|
| 30 min | Section 1–3 below (DSA light) |
| 20 min | Section 4 (Spring Boot tips) |
| 20 min | Section 5 (checklist walkthrough) |
| Rest of day | **No heavy coding** — walk, review notes only if anxious |

---

# 1. Backtracking — 10-Min Recall

## Template

```java
void backtrack(/* state */) {
    if (isComplete(state)) {
        result.add(copy(state));
        return;
    }
    for (Choice c : choices(state)) {
        if (!isValid(c, state)) continue;
        apply(c, state);
        backtrack(state);
        undo(c, state);
    }
}
```

## Classic problems (name pattern only)

| Problem | Pruning trick |
|---------|---------------|
| Subsets / Subsets II | Sort + skip duplicate index |
| Permutations | `used[]` boolean |
| Combination Sum | Start index, reuse allowed |
| N-Queens | Column + diag sets |
| Word Search | Mark visited, unmark on undo |
| Generate Parentheses | Open count ≥ close count |

**Complexity:** Often O(2^n) or O(n!); state space is the interview talking point.

---

# 2. Dynamic Programming — 10-Min Recall

## Decision tree

```text
Can you define optimal substructure on prefix/index/grid?
  → Yes: try dp[i] or dp[i][j]
Unbounded use of same item? → inner loop forward (coin change)
Two sequences? → 2D DP (LCS, edit distance)
```

## One-liners

| Problem | Recurrence |
|---------|------------|
| Climbing stairs | `dp[i] = dp[i-1] + dp[i-2]` |
| House robber | `dp[i] = max(dp[i-1], nums[i] + dp[i-2])` |
| Coin change | `dp[a] = min(dp[a], 1 + dp[a-coin])` |
| LIS | Patience sorting O(n log n) or `dp[i]` inner loop |
| Word break | `dp[i] = any(dp[j] && dict.contains(s[j:i]))` |
| Unique paths | `dp[r][c] = dp[r-1][c] + dp[r][c-1]` |

**Space optimization:** rolling array when only previous row/column needed.

---

# 3. Trees — 10-Min Recall

| Task | Approach |
|------|----------|
| LCA (BT) | Post-order: if left/right both non-null, root is LCA |
| Validate BST | Inorder or range (min, max) |
| Kth smallest BST | Inorder iterative |
| Max path sum | Post-order gain, global max |
| Serialize | Preorder with null markers |
| Level order | BFS `size = queue.size()` |

**Don’t confuse:** diameter vs max path sum (negative values).

---

# 4. Spring Boot — Interview Tips (15 min)

## Must be able to explain

| Topic | One sentence |
|-------|--------------|
| `@Transactional` | Proxy wraps method; rollback on unchecked default |
| JWT flow | Stateless; filter validates token → SecurityContext |
| `@WebMvcTest` | Controller slice; mock services with `@MockBean` |
| `@DataJpaTest` | JPA + in-memory DB; no full context |
| Actuator | Health/metrics for K8s probes |
| N+1 problem | Lazy load in loop → fix with `JOIN FETCH` or `@EntityGraph` |

## REST checklist

- Correct HTTP verbs and status codes (201 create, 204 delete)  
- Validation on DTOs, not entities  
- Global `@RestControllerAdvice` for errors  
- Pagination with `Pageable`  

## Security checklist

- BCrypt passwords, never store plain text  
- `permitAll` only for auth + public health  
- CORS configured explicitly for front-end origin  

---

# 5. Master Rest Checklist

## DSA

- [ ] Can do two-sum, LRU, level-order in 20 min each  
- [ ] Know merge K lists heap template  
- [ ] Median two sorted: partition diagram  
- [ ] Graph: BFS, DFS, topo sort when to use  

## LLD

- [ ] State pattern: vending, ATM  
- [ ] Elevator stop sets + direction  
- [ ] Parking lot / chess — name main classes  

## HLD

- [ ] URL shortener: read cache, base62, sharding  
- [ ] Uber: trip FSM, matching GEO, location Kafka  
- [ ] Twitter: fan-out push/pull/hybrid  
- [ ] CAP tradeoff one example  

## MySQL

- [ ] EXPLAIN `type` column order  
- [ ] Composite index left prefix  
- [ ] ACID + isolation levels (read committed vs repeatable read)  

## Java

- [ ] `ConcurrentHashMap` vs `HashMap`  
- [ ] `ExecutorService` vs raw `Thread`  
- [ ] Streams vs loops — when streams hurt readability  

## Spring Boot

- [ ] Built CRUD + JWT once (Day 52)  
- [ ] Mockito + slice tests  

## Behavioural

- [ ] 8+ STAR stories with metrics  
- [ ] “Tell me about yourself” rehearsed  

## Logistics

- [ ] IDE/environment tested  
- [ ] Quiet space, water, camera if virtual  
- [ ] Questions for interviewer ready  

---

# 6. Week 8 File Map (Quick Navigation)

| Day | Files |
|-----|-------|
| 50 | `day50_leetcode.md`, `day50_lld.md` |
| 51 | `day51_hld_mysql.md` |
| 52 | `day52_spring_boot.md`, `day52_java.md` |
| 53 | `day53_leetcode.md` |
| 54 | `day54_lld.md`, `day54_hld_mysql.md` |
| 55 | `day55_behavioural.md` |
| 56 | `day56_revision.md` (this file) |

---

# 7. Mindset

- **Interview is a conversation**, not an exam  
- Clarify before coding; narrate while coding  
- If stuck, state brute force, then optimize  
- After 60 days: you have breadth — trust repetition from Weeks 1–7  

---

**Congratulations on completing the 60-day backend interview revision track.**

Take the rest of Day 56 off. You are prepared to perform.

---

*End of Day 56 — Final Review*
