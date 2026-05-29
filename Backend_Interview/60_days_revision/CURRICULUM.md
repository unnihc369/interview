# 60-Day Backend Interview Curriculum

**Week 1 (Days 1–7):** Foundations — OOP, DI, Collections, REST basics, Maven, Annotations  
**Weeks 2–8 (Days 8–56):** Advanced Java, Spring Data JPA, Security, Cloud, DSA patterns, LLD, HLD, MySQL

---

## Week 1 — Foundations (Days 1–7) ✅

| Day | DSA | Java | Spring Boot | LLD / Other |
|-----|-----|------|-------------|-------------|
| 1 | Two Sum, Valid Parentheses, Merge Lists | OOP, SOLID SRP/OCP | First REST controller | — |
| 2 | Stock, Max Subarray, etc. | Abstraction, Interfaces | Dependency Injection | SOLID LSP/ISP/DIP |
| 3 | Product Except Self, Max Product, Min Rotated | Collections | application.properties, profiles | SOLID DIP |
| 4 | 3Sum, Container Water, Search Rotated | Generics, Comparable/Comparator | POST/PUT/DELETE REST | UML |
| 5 | (Arrays revision) | Exceptions | (varies) | (varies) |
| 6 | Merge Intervals, Meeting Rooms, Kth Largest | (varies) | (varies) | (varies) |
| 7 | — | — | — | HLD: URL Shortener + MySQL Indexing |

---

## Week 2 — Advanced Java, Spring Data JPA, DSA Patterns (Days 8–14)

| Day | DSA | Java | Spring Boot | LLD |
|-----|-----|------|-------------|-----|
| 8 Mon | Number of Islands, Rotting Oranges, Course Schedule | Multithreading: Thread, Runnable, synchronized | Spring Data JPA: entities, CrudRepository, CRUD | Strategy, Observer |
| 9 Tue | Word Ladder, Pacific Atlantic, Clone Graph | ExecutorService, Callable, Future | JPA @OneToMany, @ManyToOne, @ManyToMany | Decorator, Command |
| 10 Wed | Top K Frequent, Merge K Lists, Median Stream | ConcurrentHashMap, AtomicInteger | @Query, Pageable | Vending Machine |
| 11 Thu | Longest Substring, Sliding Max, Min Window | Streams API | @Transactional, propagation, isolation | Adapter, Proxy |
| 12 Fri | Trapping Rain, Largest Histogram, Max Rectangle | Optional, java.time | Mini-Project: BookStore REST + JPA | Tic-Tac-Toe |
| 13 Sat | — | — | — | Elevator System |
| 14 Sun | — | — | — | HLD: Uber/Lyft + MySQL Joins/CTEs |

---

## Week 3 — Spring Security, Trees & Graphs (Days 15–21)

| Day | DSA | Java | Spring Boot | LLD |
|-----|-----|------|-------------|-----|
| 15 Mon | BT Level Order, LCA, Max Path Sum | Nested classes, static | Spring Security basics, password encoding | Library Management |
| 16 Tue | Validate BST, Kth Smallest BST, Serialize BT | Custom annotations | JWT, UserDetailsService | State, Memento |
| 17 Wed | Connected Components, Graph Valid Tree, Alien Dict | Reflection API | @PreAuthorize, @Secured | ATM Machine |
| 18 Thu | LIS, Coin Change, Word Break | Lambdas, functional interfaces | Actuator | Template Method, Visitor |
| 19 Fri | House Robber I/II, Decode Ways | GC basics | SLF4J + Logback | Chess Game |
| 20 Sat | — | — | — | Splitwise |
| 21 Sun | — | — | — | HLD: Amazon Search + ACID/Isolation |

---

## Week 4 — DP, Spring Cloud, LLD Round-up (Days 22–28)

| Day | DSA | Java | Spring Boot | LLD |
|-----|-----|------|-------------|-----|
| 22 Mon | Median Two Sorted, Merge/Insert Interval | CountDownLatch, CyclicBarrier | OpenAPI / Swagger | LLD pattern review |
| 23 Tue | Non-overlap Intervals, Meeting Rooms II, Employee Free Time | WeakHashMap, heap vs stack | @Cacheable, Redis | Hotel Booking |
| 24 Wed | Implement Trie, Word Search II, Add Search Word | Java I/O, NIO | @Scheduled, @Async | Stock Brokerage |
| 25 Thu | Regex Matching, Wildcard, Edit Distance | java.util.regex | @Conditional, profiles | Chain of Responsibility |
| 26 Fri | Reverse K Group, Copy Random List, LFU Cache | JUnit 5, Mockito | @WebMvcTest vs @DataJpaTest | Car Rental |
| 27 Sat | — | — | — | Movie Ticket Booking |
| 28 Sun | — | — | — | HLD: WhatsApp + Query Optimization |

---

## Week 5 — Graphs, Backtracking, Spring Cloud (Days 29–35)

| Day | DSA | Java | Spring Boot | LLD |
|-----|-----|------|-------------|-----|
| 29 Mon | Word Search, Generate Parentheses, Phone Letter Combos | Java Modules (JPMS) | Eureka Service Discovery | Ride Sharing (Uber LLD) |
| 30 Tue | Permutations, Subsets, Combination Sum | JVM: classloader, runtime areas | Spring Cloud Gateway | Flyweight, Bridge |
| 31 Wed | N-Queens, Sudoku, Remove Invalid Parens | Collectors advanced | Resilience4J circuit breaker | Parking Lot (extended) |
| 32 Thu | Shortest Path Binary Matrix, Network Delay, Cheapest Flights K | CompletableFuture | Sleuth + Zipkin | Logging Framework |
| 33 Fri | Min Height Trees, Reconstruct Itinerary, Critical Connections | G1GC, ZGC | @ConfigurationProperties | Coffee Vending Machine |
| 34 Sat | — | — | — | Snake and Ladder |
| 35 Sun | — | — | — | HLD: Netflix/YouTube + Partitions/Sharding |

---

## Week 6 — System Design Deep Dive (Days 36–42)

| Day | DSA | Java | Spring Boot | LLD |
|-----|-----|------|-------------|-----|
| 36 Mon | Reorganize String, Task Scheduler, Partition Labels | Bytecode (ASM intro) | Kafka producer/consumer | Online Auction |
| 37 Tue | Max Freq Stack, Design Twitter, Time Key-Value | Bean Validation | WebSockets | Mediator, Interpreter |
| 38 Wed | Encode TinyURL, GetRandom O(1) Dup, Range Sum 2D | KeyStore, Cipher | OAuth2 Resource Server | Meeting Scheduler |
| 39 Thu | Brick Wall, Subarray Divisible K, Longest Subarray Limit | Java 9+ var, List.of | REST Docs vs Swagger | Airline Check-in |
| 40 Fri | Min Arrows Balloons, Queue Reconstruction, Dota2 Senate | Java 17 records, sealed | Spring Native / GraalVM | Task Management (Trello) |
| 41 Sat | — | — | — | WhatsApp chat server (LLD) |
| 42 Sun | — | — | — | HLD: Facebook News Feed + Replication |

---

## Week 7 — Mock Interview Prep (Days 43–49)

| Day | DSA | Java | Spring Boot | LLD |
|-----|-----|------|-------------|-----|
| 43 Mon | Serialize N-ary Tree, All O(1) DS, Stay in Place Steps | Deadlock, livelock, starvation | @SpringBootTest, MockMvc | LLD pattern review |
| 44 Tue | Race Car, Strange Printer, Min Cost Valid Path | JFR, JMC profiling | @ConditionalOnProperty | Cache System LRU/LFU |
| 45 Wed | Longest Happy String, Form Target String, Max Events | MethodHandle | Test slices | Payment Gateway |
| 46 Thu | Mock DSA (3 timed problems) | Java 8+ review | Annotations review | Parking Lot extended |
| 47 Fri | Mock DSA (3 new Amazon) | Concurrency review | Security + JWT review | Library concurrent reservations |
| 48 Sat | — | — | — | Chess full code mock |
| 49 Sun | — | — | — | HLD: Prime Video + Transactions/deadlocks |

---

## Week 8 — Final Revision (Days 50–56)

| Day | Focus |
|-----|-------|
| 50 Mon | DSA revision + LLD: Vending Machine, Elevator |
| 51 Tue | HLD: URL Shortener, Uber + MySQL EXPLAIN |
| 52 Wed | Full CRUD app: JPA + Security + Actuator + tests |
| 53 Thu | Mock DSA: 3 hards (1 hour) |
| 54 Fri | Mock LLD (ATM) + HLD (Twitter) |
| 55 Sat | Amazon Leadership Principles (STAR) |
| 56 Sun | Light review + rest |

---

## File naming convention

```
day_N/
  dayN_leetcode.md      — 3 DSA problems (Mon–Fri)
  dayN_java.md          — Java topic(s)
  dayN_spring_boot.md   — Spring Boot topic(s)
  dayN_lld.md           — LLD / design patterns
  dayN_hld_mysql.md     — Sat/Sun only (HLD + MySQL)
```

---

## SDE 1–2 Supplement Files

Gap topics are **woven into existing days** and collected in:

| File | Topics |
|------|--------|
| [supplements/sde1_spring_mvc_aop.md](supplements/sde1_spring_mvc_aop.md) | Spring AOP, Filter vs Interceptor vs AOP, CORS, multipart upload, Application Events |
| [supplements/sde1_lld_missing_designs.md](supplements/sde1_lld_missing_designs.md) | Rate Limiter, Restaurant, Deck of Cards, Amazon Locker, Notification System |

Integrated into days: String/equals/hashCode (Day 3), pass-by-value (Day 5), bean scopes/MVC (Day 2), normalization (Day 7), JPA @EntityGraph/@Version/Flyway (Day 9), volatile/ThreadLocal (Day 8), GROUP BY/HAVING (Day 14), HikariCP (Day 25), creational/structural patterns (Day 5 LLD).

---

*Follow daily files in `day_8/` through `day_56/` for detailed revision content.*
