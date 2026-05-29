# Day 54 — HLD Mock: Twitter / X (45 Minutes)

**Week 8 · Friday · Timed 45 minutes**

Design a simplified **Twitter feed** system (post tweets, follow users, home timeline).

---

# Mock Timer

| Segment | Min |
|---------|-----|
| Requirements + estimates | 8 |
| High-level diagram | 10 |
| Tweet write path | 8 |
| Timeline read path | 12 |
| Deep dive (fan-out) | 5 |
| Tradeoffs + Q&A prep | 2 |

---

# 1. Functional Requirements

- Users post tweets (text, optional media metadata)  
- Follow / unfollow users  
- Home timeline: recent tweets from people I follow  
- User profile timeline: my tweets  
- Like, retweet (mention as extension)  
- Search users/tweets (optional, lower priority)  

---

# 2. Non-Functional Requirements

| NFR | Target |
|-----|--------|
| Read latency | Home feed < 200ms p99 |
| Write latency | Post tweet < 500ms |
| Availability | 99.9% |
| Scale | 300M MAU, 500M tweets/day |
| Consistency | Eventual for timeline; strong for tweet write ack |

---

# 3. Capacity Estimation

```text
500M tweets/day ≈ 6K tweets/sec average (~50K peak)
Read:feed >> write  (100:1 typical)
Avg tweet 300 bytes → 150 GB/day raw text → object store for media
```

**Fan-out math:**  
Celebrity with 10M followers → 10M writes per tweet if push model → **must use hybrid fan-out**.

---

# 4. High-Level Architecture

```text
                    ┌─────────────┐
Clients ───────────►│ API Gateway │
                    └──────┬──────┘
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
    ┌────────────┐  ┌────────────┐  ┌────────────┐
    │ User Svc   │  │ Tweet Svc  │  │ Timeline   │
    └────────────┘  └─────┬──────┘  │ Service    │
                          │         └──────┬─────┘
                          ▼                ▼
                    ┌──────────┐    ┌──────────────┐
                    │ Tweet DB │    │ Redis caches │
                    │ (sharded)│    │ (timelines)  │
                    └──────────┘    └──────────────┘
                          │
                    ┌─────▼─────┐
                    │ Kafka     │  tweet.created, follow events
                    └───────────┘
```

---

# 5. Data Model

## Tweets table (sharded by `tweet_id` or `user_id`)

```sql
CREATE TABLE tweets (
    tweet_id     BIGINT PRIMARY KEY,
    user_id      BIGINT NOT NULL,
    content      VARCHAR(280),
    created_at   TIMESTAMP,
    INDEX idx_user_time (user_id, created_at DESC)
);
```

## Follow graph

```sql
CREATE TABLE follows (
    follower_id  BIGINT,
    followee_id  BIGINT,
    created_at   TIMESTAMP,
    PRIMARY KEY (follower_id, followee_id)
);
```

## Social graph at scale

- **Adjacency list** in SQL for normal users  
- **Graph DB** or cache for “who follows whom” hot paths (optional)  

---

# 6. Write Path — Post Tweet

```text
1. Client POST /tweets {content}
2. Tweet Service validates, assigns tweet_id (Snowflake)
3. INSERT tweet DB (source of truth)
4. Publish event tweet.created → Kafka
5. Return 201 to client

Async consumers:
  - Fan-out worker (for normal users)
  - Search indexer
  - Notification service (mentions)
```

---

# 7. Read Path — Home Timeline

## Strategy A: Fan-out on write (push)

On tweet from user U, push `tweet_id` into Redis list `timeline:{follower_id}` for each follower.

**Pros:** Fast reads O(1) fetch  
**Cons:** Celebrity problem — millions of Redis writes per tweet

## Strategy B: Fan-out on read (pull)

On home timeline request, fetch followees, query recent tweets per user, merge-sort.

**Pros:** Cheap writes  
**Cons:** Slow reads for users following many accounts

## Strategy C: Hybrid (Twitter’s approach)

| User type | Strategy |
|-------------|----------|
| Normal (< 10K followers) | Fan-out on write to follower timelines |
| Celebrity | Fan-out on read + cache celebrity tweets separately |

```text
timeline:{userId}  → Redis LIST of tweet_ids (precomputed)
celebrity_tweets   → recent tweets from hot accounts (cached)
```

**Read algorithm:**

```text
1. GET timeline:userId from Redis (last N ids)
2. If stale or first page: merge celebrity pull + cached list
3. Hydrate tweet_ids → Tweet Service batch GET
4. Return ranked by time
```

---

# 8. Caching Layers

| Cache | Key | TTL |
|-------|-----|-----|
| Home timeline | `tl:{userId}` | Minutes |
| Tweet body | `tweet:{id}` | Hours |
| User profile | `user:{id}` | Hours |
| Follow list | `following:{userId}` | Minutes |

**Cache invalidation:** On unfollow, trim timeline entries (lazy) or background rebuild.

---

# 9. Sharding

- **Tweets:** shard by `user_id` (profile reads local)  
- **Tweet ID global:** Snowflake for cross-shard lookup by id  
- **Timeline Redis:** hash by `user_id` across cluster  

---

# 10. Media (Extension)

```text
Upload → presigned S3 URL → store URL in tweet metadata
CDN serves images/video
```

---

# 11. Failure & Edge Cases

| Scenario | Handling |
|----------|----------|
| Fan-out lag | Stale feed OK briefly; show “new tweets available” |
| Hot key (celebrity) | Pull model + aggressive CDN |
| Duplicate tweet post | Idempotency key from client |
| Deleted tweet | Tombstone in DB; purge from timelines async |

---

# 12. 45-Min Whiteboard Checklist

| # | Item | ✓ |
|---|------|---|
| 1 | Functional vs non-functional separated | |
| 2 | Rough QPS / storage estimate | |
| 3 | Tweet write path diagram | |
| 4 | Home timeline read path | |
| 5 | Push vs pull vs hybrid explained | |
| 6 | Celebrity / hot user problem | |
| 7 | Kafka for async fan-out | |
| 8 | Redis for timelines | |
| 9 | DB schema for tweets + follows | |
| 10 | Sharding mention | |

---

# Interview Follow-ups

**How is timeline sorted?**  
Merge k sorted lists of tweet_ids by timestamp — heap of size = number of followees fetched (cap followees per request).

**Consistency when following new user?**  
Backfill recent tweets async into `timeline:{userId}`.

**Compare to Instagram feed?**  
Similar fan-out; ranking ML layer heavier on IG.

**Rate limiting?**  
Token bucket per user on post; prevent spam.

---

# Relation to LeetCode “Design Twitter” (LC 355)

LC 355 is **in-memory** follow + recent 10 tweets — practice for **data structure** thinking:

- `Map<userId, Set<followee>>`  
- `Map<userId, List<tweetId>>` with global timestamp  

HLD mock adds distributed storage, fan-out, and scale.

---

*End of Day 54 — HLD Mock*
