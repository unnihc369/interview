# Day 49 — HLD: Amazon Prime Video + MySQL Transactions & Locking

**Topics:** Video streaming architecture · CDN · Adaptive bitrate · ACID · Deadlocks · Optimistic vs Pessimistic locking · Interview Q&A

---

# Part 1: Amazon Prime Video — HLD Mock

---

# 1. Functional Requirements

- Browse catalog (movies, series, genres)
- Search titles
- Stream video with adaptive quality (360p → 4K)
- Resume playback from last position
- User profiles (multi-profile per account)
- Watchlist, recommendations
- DRM-protected content
- Subtitles / multiple audio tracks

---

# 2. Non-Functional Requirements

- **Latency:** Start playback < 2 seconds
- **Availability:** 99.99% for streaming
- **Scale:** 200M+ subscribers, millions concurrent streams
- **Global:** Low latency worldwide via CDN
- **Security:** DRM, geo-restriction, anti-piracy

---

# 3. High-Level Architecture

```text
Client (Web / Mobile / Smart TV)
        ↓
   CDN (CloudFront / Akamai) — static assets, video segments
        ↓
   API Gateway + Load Balancer
        ↓
┌───────────────────────────────────────────────────────┐
│  Backend Services                                     │
│  ├── Catalog Service (metadata, search)               │
│  ├── User Service (profiles, subscriptions)           │
│  ├── Playback Service (session, resume, entitlements) │
│  ├── Recommendation Service (ML ranking)              │
│  ├── Encoding Pipeline (transcode → HLS/DASH)         │
│  └── Analytics Service (view events)                  │
└───────────────────────────────────────────────────────┘
        ↓                    ↓                    ↓
   Elasticsearch        MySQL / DynamoDB       S3 (video segments)
   (search index)       (metadata, users)      (origin storage)
        ↓
   Redis (session cache, hot catalog, resume position)
        ↓
   Kafka (view events → recommendations, billing)
```

---

# 4. Video Streaming Flow

```text
1. User selects title
2. Playback Service checks entitlement (subscription, geo, parental controls)
3. Returns manifest URL (HLS .m3u8 or DASH .mpd) — signed, time-limited
4. Client CDN fetches video segments (.ts / .mp4 chunks)
5. ABR (Adaptive Bitrate) switches quality based on bandwidth
6. Heartbeat every 30s → update watch progress in Redis + async persist
7. View event → Kafka → Recommendation engine
```

---

# 5. Encoding Pipeline

```text
Raw upload (ProRes) → S3
        ↓
   MediaConvert / custom transcode farm
        ↓
   Multiple renditions: 360p, 480p, 720p, 1080p, 4K
        ↓
   Package as HLS (segment 2–6 sec each)
        ↓
   S3 + CDN edge cache
```

---

# 6. Key Design Decisions

| Decision | Choice | Why |
|----------|--------|-----|
| Video storage | S3 + CDN | Cheap storage, edge delivery |
| Metadata DB | MySQL + read replicas | ACID for subscriptions/billing |
| Search | Elasticsearch | Full-text, faceted browse |
| Resume position | Redis (hot) + MySQL (cold) | Fast read/write during playback |
| Manifest URLs | Signed CloudFront URLs | Expire in 1–4 hours, prevent hotlinking |
| DRM | Widevine / FairPlay / PlayReady | Studio content requirements |

---

# 7. Data Model (Core Tables)

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    subscription_tier ENUM('FREE','PRIME','PREMIUM'),
    created_at TIMESTAMP
);

CREATE TABLE profiles (
    id BIGINT PRIMARY KEY,
    user_id BIGINT REFERENCES users(id),
    name VARCHAR(100),
    is_kids BOOLEAN DEFAULT FALSE
);

CREATE TABLE titles (
    id BIGINT PRIMARY KEY,
    title VARCHAR(500),
    type ENUM('MOVIE','SERIES'),
    duration_sec INT,
    release_year INT
);

CREATE TABLE watch_progress (
    profile_id BIGINT,
    title_id BIGINT,
    episode_id BIGINT NULL,
    position_sec INT,
    updated_at TIMESTAMP,
    PRIMARY KEY (profile_id, title_id, episode_id)
);

CREATE TABLE entitlements (
    user_id BIGINT,
    title_id BIGINT,
    valid_from TIMESTAMP,
    valid_until TIMESTAMP,
    PRIMARY KEY (user_id, title_id)
);
```

---

# 8. Resume Playback — Caching Strategy

```text
Write path:
  Client heartbeat → Redis SET progress:{profile}:{title} TTL 7d
                  → Kafka → async batch write to MySQL every 60s

Read path:
  GET progress → Redis first → fallback MySQL → populate Redis
```

---

# 9. Recommendations (Brief)

```text
View events → Kafka → Flink/Spark batch
        ↓
Collaborative filtering + content-based features
        ↓
Pre-computed "Because you watched X" stored in Redis/DynamoDB
        ↓
Real-time layer: session context boost at request time
```

---

# Part 2: MySQL Transactions, Deadlocks & Locking

---

# 10. ACID in Prime Video Context

| Property | Example |
|----------|---------|
| **Atomicity** | Subscribe + charge + grant entitlement — all or nothing |
| **Consistency** | User can't stream without valid entitlement |
| **Isolation** | Concurrent profile updates don't corrupt watch history |
| **Durability** | Payment record persisted before access granted |

---

# 11. Transaction Example — Subscription Purchase

```java
@Service
public class SubscriptionService {

    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void subscribe(Long userId, String planId) {
        User user = userRepo.findByIdForUpdate(userId);  // pessimistic lock
        if (user.hasActiveSubscription()) throw new AlreadySubscribedException();

        PaymentResult payment = paymentGateway.charge(user, planId);
        if (!payment.isSuccess()) throw new PaymentFailedException();

        Subscription sub = subscriptionRepo.save(new Subscription(userId, planId));
        entitlementService.grantAllPrimeContent(userId);
        auditLog.record("SUBSCRIBE", userId, sub.getId());
    }
}
```

```sql
-- Repository: pessimistic row lock
SELECT * FROM users WHERE id = ? FOR UPDATE;
```

---

# 12. Isolation Levels Quick Reference

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|-------|------------|---------------------|--------------|
| READ UNCOMMITTED | Yes | Yes | Yes |
| READ COMMITTED | No | Yes | Yes |
| REPEATABLE READ (InnoDB default) | No | No | No* |
| SERIALIZABLE | No | No | No |

*InnoDB REPEATABLE READ prevents most phantoms via next-key locks.

---

# 13. Pessimistic Locking

## When to Use

- High contention on same row (inventory, subscription slot, wallet balance)
- Must prevent concurrent modification definitively

```sql
-- Row lock until transaction commits
SELECT * FROM entitlements
WHERE user_id = 123 AND title_id = 456
FOR UPDATE;

-- Shared lock (others can read, not write)
SELECT * FROM titles WHERE id = 789 LOCK IN SHARE MODE;
```

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT e FROM Entitlement e WHERE e.userId = :uid AND e.titleId = :tid")
Optional<Entitlement> findForUpdate(@Param("uid") Long uid, @Param("tid") Long tid);
```

---

# 14. Optimistic Locking

## When to Use

- Low contention, read-heavy (watch progress, profile name)
- Better throughput — no DB lock held

```java
@Entity
public class WatchProgress {
    @Id
    private Long id;

    @Version
    private Long version;

    private int positionSec;
    private Instant updatedAt;
}
```

```java
@Transactional
public void updateProgress(Long profileId, Long titleId, int position) {
    WatchProgress wp = progressRepo.findByProfileAndTitle(profileId, titleId)
        .orElse(new WatchProgress(profileId, titleId));
    wp.setPositionSec(position);
    wp.setUpdatedAt(Instant.now());
    progressRepo.save(wp);  // UPDATE ... WHERE version = ? → 0 rows → OptimisticLockException
}
```

**On conflict:** retry with merge (take max position) or last-write-wins policy.

---

# 15. Deadlock Scenario — Entitlement Grant

```text
Transaction A:
  LOCK entitlements row (user=1, title=100)
  WAIT  entitlements row (user=1, title=200)

Transaction B:
  LOCK entitlements row (user=1, title=200)
  WAIT  entitlements row (user=1, title=100)

→ Deadlock → InnoDB kills one transaction (ERROR 1213)
```

---

# 16. Deadlock Prevention

| Strategy | Application |
|----------|-------------|
| Consistent lock order | Always lock title IDs in ascending order |
| Short transactions | Don't call payment gateway inside long DB txn |
| Retry on deadlock | `@Retryable` on `DeadlockLoserDataAccessException` |
| Reduce lock scope | Update one row per transaction when possible |

```java
@Retryable(retryFor = DeadlockLoserDataAccessException.class, maxAttempts = 3,
           backoff = @Backoff(delay = 100))
@Transactional
public void batchGrantEntitlements(Long userId, List<Long> titleIds) {
    titleIds.stream().sorted().forEach(tid -> grantOne(userId, tid));
}
```

---

# 17. Pessimistic vs Optimistic — Decision Matrix

| Scenario | Lock Type | Why |
|----------|-----------|-----|
| Subscribe / payment | Pessimistic | Must not double-charge |
| Watch progress update | Optimistic | High frequency, low conflict |
| Concurrent seat booking (live event) | Pessimistic | Limited seats |
| Profile name edit | Optimistic | Rare conflicts |
| Bulk catalog import | Batch + no lock | Offline job |

---

# 18. Transaction Propagation in Streaming Context

```java
@Transactional(propagation = Propagation.REQUIRED)
public PlaybackSession startSession(Long profileId, Long titleId) {
    entitlementService.verify(profileId, titleId);  // REQUIRED — same txn
    return sessionRepo.save(new PlaybackSession(profileId, titleId));
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void logViewEvent(ViewEvent event) {
    // Separate txn — analytics failure doesn't rollback playback
    analyticsRepo.save(event);
}
```

---

# 19. Read Replica Considerations

```text
Catalog browse     → read replica (eventual consistency OK)
Entitlement check  → primary (must be consistent)
Search             → Elasticsearch (async index from MySQL binlog)
```

Use `@Transactional(readOnly = true)` on replica routes for browse APIs.

---

# Interview Questions

## Q1. How does Prime Video start playback in < 2 seconds?

CDN edge cache for first segments, small initial segment size (2s), ABR starts at lower quality, manifest pre-fetched on title page load.

## Q2. Where store 4K video?

S3 origin, CDN edge caching for popular content, cold content fetched from origin on first request in region.

## Q3. Pessimistic vs optimistic for subscription?

Pessimistic — two concurrent subscribe requests must not both succeed. `SELECT FOR UPDATE` on user row.

## Q4. Deadlock in MySQL — what happens?

InnoDB detects wait cycle, rolls back one transaction (deadlock victim). Application retries.

## Q5. Watch progress — lost update problem?

Two devices same profile: optimistic lock with retry, or merge strategy (keep max position).

## Q6. How handle geo-restriction?

Playback Service checks IP → GeoIP DB → compare with title rights table before signing manifest URL.

## Q7. Eventual consistency in recommendations?

View event async via Kafka — "Continue watching" row may lag seconds — acceptable UX.

---

# Quick Revision

```text
Prime Video → S3 segments + CDN + HLS/DASH + signed URLs + Redis resume + Kafka events
Subscribe   → @Transactional + SELECT FOR UPDATE (pessimistic)
Progress    → @Version optimistic lock + retry
Deadlock    → consistent lock order + short txns + retry 1213
Isolation   → InnoDB default REPEATABLE READ
```

---

*End of Day 49 HLD + MySQL*
