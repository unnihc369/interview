# Day 42 — HLD: Facebook News Feed + MySQL Replication & Failover

**Topics:** News Feed Architecture · Fan-out · Ranking · MySQL Replication · Failover · Interview Questions

---

# Part 1: Facebook News Feed — HLD

---

# 1. Functional Requirements

- Users create posts (text, image, video)
- News feed shows posts from friends/pages followed
- Like, comment, share on posts
- Real-time feed updates (near real-time acceptable)
- Support billions of users, millions of posts/day

---

# 2. Non-Functional Requirements

- Feed load < 200ms p99
- High availability (99.99%)
- Eventual consistency acceptable for feed (seconds delay OK)
- Scalable write path for viral posts
- Personalized ranking

---

# 3. High-Level Architecture

```text
Client (Web/Mobile)
      ↓
CDN (media)
      ↓
Load Balancer
      ↓
API Gateway
      ↓
┌──────────────────────────────────────────────┐
│ Post Service    │ Feed Service │ Social Graph │
│ (write posts)   │ (read feed)  │ (follows)    │
└──────────────────────────────────────────────┘
      ↓                    ↓              ↓
  Posts DB            Feed Cache       Graph DB
  (Cassandra)         (Redis)          (TAO / Neo4j)
      ↓
  Fan-out Workers (Kafka)
      ↓
  Feed Storage (Redis / Cassandra per user)
      ↓
  Ranking Service (ML features)
```

---

# 4. Write Path — Create Post

```text
User creates post
      ↓
Post Service validates, stores in Posts DB (postId, userId, content, ts)
      ↓
Publish PostCreated event to Kafka
      ↓
Fan-out Service consumes event
      ↓
Fetch followers from Social Graph Service
      ↓
For each follower (or celebrity strategy):
  push postId into follower's feed cache (Redis sorted set)
      ↓
Ranking pipeline scores post for each feed (async)
```

---

# 5. Fan-out on Write vs Fan-out on Read

| Strategy | Pros | Cons |
|----------|------|------|
| **Fan-out on write** | Fast read — precomputed feed | Slow write for celebrities (millions followers) |
| **Fan-out on read** | Fast write | Slow read — merge at request time |

**Hybrid (Facebook approach):**

```text
Normal user (< 10K followers) → fan-out on write
Celebrity (> 10K followers)    → fan-out on read (merge at read time)
```

---

# 6. Feed Read Path

```text
GET /feed
      ↓
Feed Service checks Redis: feed:{userId} (sorted set of postIds by score/time)
      ↓
Cache miss → rebuild from DB + fan-out read for celebrity follows
      ↓
Batch fetch post content from Posts DB / CDN
      ↓
Ranking Service re-ranks top N posts (engagement, affinity, recency)
      ↓
Return paginated JSON to client
```

---

# 7. Feed Storage Design

**Redis sorted set:**

```text
Key: feed:user:{userId}
Score: timestamp or rank score
Member: postId
```

Trim to last 1000 entries. Older posts archived or fetched on scroll.

**Cassandra fallback** for users with huge feeds.

---

# 8. Social Graph Service

```text
follows table:
  follower_id, followee_id, created_at

Indexes:
  - followers of user X  (fan-out)
  - followees of user Y  (timeline sources)
```

At Facebook scale: **TAO** (graph cache) or distributed graph DB.

---

# 9. Ranking / News Feed Algorithm (Conceptual)

Signals:

```text
- Affinity: interactions with author
- Engagement: likes, comments, shares velocity
- Recency: time decay
- Content type: video boost, link penalty
- Negative signals: hide post, report
```

```text
score = w1*affinity + w2*engagement_rate + w3*recency_decay + ML_model(features)
```

ML model trained offline; features in **feature store** (Redis + batch).

---

# 10. Comments, Likes — Denormalization

```text
post_counters: { postId, likeCount, commentCount }  (Redis INCR)
comments sharded by postId
```

Feed shows counts without joining heavy tables.

---

# 11. Hot Post / Viral Content

```text
Viral post → skip full fan-out
Store in hot_posts cache
On feed read: merge hot posts from followed celebrities + user feed cache
```

---

# 12. Media Pipeline

```text
Upload → S3 → async transcoding (video) → CDN URLs in post metadata
```

Separate from feed critical path.

---

# Part 2: MySQL Replication & Failover

---

# 13. MySQL Replication Topologies

## Primary-Replica (Async)

```text
Primary (writes) ──binlog──→ Replica1 (reads)
                          └──→ Replica2 (reads)
```

- Default async replication — primary does not wait for replica ACK
- Fast writes; possible data loss on primary crash (unreplicated transactions)

## Semi-Synchronous

Primary waits for **at least one** replica to acknowledge receipt before commit ACK to client.

Reduces data loss window.

## Group Replication (MySQL InnoDB Cluster)

Multi-primary or single-primary quorum-based replication — automatic failover within cluster.

---

# 14. Binary Log (binlog)

```text
Primary writes → binlog events (ROW / STATEMENT / MIXED format)
Replica I/O thread pulls binlog → relay log
Replica SQL thread applies relay log
```

**ROW format** preferred for consistency (replicates actual row changes).

---

# 15. Replication Lag

Causes: heavy writes, slow replica disk, long transactions, parallel replication not tuned.

Monitor:

```sql
SHOW REPLICA STATUS\G
-- Seconds_Behind_Source (lag indicator)
```

Mitigation:

- Read scaling only on caught-up replicas
- Parallel replication (`replica_parallel_workers`)
- Split write/read — don't read stale for critical paths

---

# 16. Failover Scenarios

## Manual Failover

```text
1. Stop writes to primary (app maintenance mode)
2. Wait for replicas to catch up
3. Promote best replica: STOP REPLICA; RESET REPLICA ALL; read_only=OFF
4. Repoint application to new primary
5. Reconfigure old primary as replica (if recoverable)
```

## Automatic Failover (Orchestrator, MHA, ProxySQL, AWS RDS Multi-AZ)

```text
Health check fails on primary
      ↓
Leader election / orchestrator promotes replica
      ↓
VIP/DNS update (db.example.com → new primary)
      ↓
App reconnects via connection pool
```

---

# 17. Split-Brain Prevention

```text
Old primary still accepting writes + new primary promoted
→ divergent data
```

Solutions:

- **Fencing** — STONITH: isolate old primary (shutdown NIC / iptables)
- Quorum (Group Replication, Galera)
- Orchestrator `RecoveryAcknowledgement`

---

# 18. Connection Pool on Failover

```java
// HikariCP with JDBC URL pointing to cluster VIP or RDS endpoint
spring.datasource.url=jdbc:mysql://cluster-vip:3306/app
```

Use **short connection maxLifetime** so pool refreshes after failover.

Test: `socketTimeout`, retry logic for transient `Communications link failure`.

---

# 19. Read/Write Splitting

```java
@Transactional(readOnly = true)  // route to replica
public List<Post> getFeed(Long userId) { ... }

@Transactional  // route to primary
public Post createPost(PostRequest req) { ... }
```

Spring AbstractRoutingDataSource or middleware (ProxySQL, Vitess).

**Stale read risk** after write — route user's own writes to primary or use **read-your-writes** consistency token.

---

# 20. Multi-AZ vs Multi-Region

| Multi-AZ | Multi-Region |
|----------|--------------|
| Sync/semi-sync within region | Async cross-region replication |
| Low RPO in AZ failure | Higher lag, disaster recovery |
| RDS Multi-AZ automatic failover | Aurora Global Database |

---

# 21. Backup & PITR

```text
Full backup (mysqldump / Percona XtraBackup)
+ continuous binlog archiving
→ Point-in-time recovery to任意 second before mistake
```

---

# 22. News Feed + MySQL — When MySQL Fits

| Use MySQL | Use Specialized Store |
|-----------|----------------------|
| User profile, billing | Feed cache (Redis) |
| Small relational data | Posts at scale (Cassandra) |
| Strong consistency transactions | Graph (TAO) |

Facebook feed not stored in single MySQL table — MySQL for metadata; feed in cache/NoSQL. Interview: articulate **polyglot persistence**.

---

# Interview Questions

## How does Facebook handle celebrity posts?

Hybrid fan-out: skip write fan-out for high-follower accounts; merge on read from celebrity post cache.

## Feed consistency after posting?

Read-your-post from primary or insert into author's feed synchronously; followers eventual.

## Async vs semi-sync replication?

Async: faster, risk loss. Semi-sync: safer, slightly slower. Choose by RPO/RTO SLA.

## RPO vs RTO?

RPO = max acceptable data loss (time). RTO = max downtime to restore service.

## Orchestrator vs manual failover?

Orchestrator detects failure, promotes replica, updates topology — reduces human error and MTTR.

## Why binlog ROW format?

Avoids non-deterministic statement replay; exact row changes on replica.

## GTID benefit?

Global Transaction ID — simplifies failover: promote replica without guessing binlog position.

```sql
SHOW MASTER STATUS;  -- old
SELECT @@GLOBAL.gtid_executed;  -- GTID-based
```

---

# Quick Revision

```text
News Feed → fan-out on write (normal) + fan-out on read (celebrity); Redis feed cache; Kafka events; ranking ML.
MySQL → async replicas, semi-sync for RPO, Orchestrator failover, fencing vs split-brain, read/write split + lag awareness.
```

---

*End of Day 42 HLD + MySQL*
