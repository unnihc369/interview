# Day 35 — HLD: Netflix / YouTube + MySQL Partitions & Sharding

**Topics:** Video Streaming HLD · CDN · Transcoding · MySQL Partitioning · Sharding · Interview Questions

---

# Part 1: Netflix / YouTube — High-Level Design

---

# 1. Functional Requirements

- Upload videos (creators)
- Stream videos on demand (adaptive bitrate)
- Search and recommendations
- User profiles, subscriptions, watch history
- Comments, likes, playlists
- Live streaming (extension)

---

# 2. Non-Functional Requirements

- **Scale:** billions of hours watched/month
- **Latency:** video start < 2s
- **Availability:** 99.99% for playback
- **Global:** users worldwide
- **Consistency:** eventual for views/likes; strong for payments

---

# 3. High-Level Architecture

```text
                    ┌─────────────────────────────────────┐
                    │           CDN (Edge caches)          │
                    │   video segments (.ts / .m4s chunks) │
                    └──────────────────▲──────────────────┘
                                       │
Client (Web/TV/Mobile)                 │
      │                                │
      ▼                                │
┌─────────────┐    metadata/API   ┌────┴────────────┐
│ API Gateway │ ◄────────────────►│  App Services   │
└─────────────┘                   │  User, Catalog, │
                                  │  Upload, Reco   │
                                  └────────┬────────┘
                                           │
              ┌────────────────────────────┼────────────────────────┐
              ▼                            ▼                        ▼
        ┌──────────┐              ┌──────────────┐          ┌─────────────┐
        │ MySQL /  │              │ Object Store │          │ Kafka       │
        │ Cassandra│              │ S3 / GCS     │          │ (events)    │
        │ (meta)   │              │ raw + segments│         └─────────────┘
        └──────────┘              └──────────────┘
                                           ▲
                                  ┌────────┴────────┐
                                  │ Transcoding     │
                                  │ Workers (K8s)   │
                                  └─────────────────┘
```

---

# 4. Upload Flow

```text
1. Creator requests upload URL (pre-signed S3 PUT)
2. Client uploads raw video directly to object storage
3. Upload service publishes "video.uploaded" to Kafka
4. Transcoding pipeline:
   - Extract audio/video
   - Encode 360p, 480p, 720p, 1080p, 4K
   - Package HLS/DASH segments
   - Generate thumbnails
5. Store segment URLs + metadata in DB
6. Invalidate/warm CDN cache for popular content
```

---

# 5. Playback Flow

```text
1. User clicks video → API returns manifest URL + auth token
2. Client requests HLS/DASH manifest (.m3u8)
3. Player selects bitrate (ABR) based on bandwidth
4. Client fetches segments from nearest CDN edge
5. Heartbeat events → watch history / recommendations
```

**Adaptive Bitrate (ABR):** switch quality per segment based on buffer and bandwidth.

---

# 6. Key Components Deep Dive

### Metadata DB

```text
videos(id, creator_id, title, description, status, created_at)
video_formats(video_id, resolution, codec, manifest_url)
users, subscriptions, watch_history
```

High write on watch events → Cassandra or sharded MySQL + Kafka consumers.

### Object Storage

Raw uploads and transcoded segments. Cheap, durable. Lifecycle policies move old raw to Glacier.

### CDN

CloudFront/Akamai cache segments at edge. Cache key = segment path. TTL long for immutable segments.

### Recommendation

Offline: Spark jobs on watch history → feature store.  
Online: rank candidates from precomputed embeddings + real-time signals.

### Search

Elasticsearch inverted index on title, tags, transcripts (ASR).

---

# 7. Capacity Estimation (Interview)

```text
Assume:
  500M DAU, 1 hour avg watch/day
  = 500M hours/day ≈ 6M hours/second peak (with factor)

Storage per 10-min 1080p video ≈ 1-2 GB raw, ~500 MB transcoded all formats
1M uploads/day × 1 GB avg = 1 PB/year order of magnitude

CDN egress dominates cost — cache hit ratio critical
```

---

# 8. Bottlenecks & Mitigations

| Bottleneck | Mitigation |
|------------|------------|
| Transcoding backlog | Auto-scale workers, priority queue |
| Hot videos | CDN + multi-region replication |
| DB write load | Sharding, async aggregation |
| Upload failures | Multipart upload, resume |
| Copyright | Content ID fingerprinting |

---

# Part 2: MySQL Partitioning

---

# 9. What is Partitioning?

Split one logical table into **physical segments** within same MySQL instance. Query optimizer **prunes** irrelevant partitions.

```sql
CREATE TABLE orders (
    id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    amount DECIMAL(10,2),
    created_at DATETIME NOT NULL,
    PRIMARY KEY (id, created_at)
)
PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

---

# 10. Partition Types

| Type | Use Case |
|------|----------|
| **RANGE** | Dates, incremental IDs |
| **LIST** | Region, status enum |
| **HASH** | Even distribution, no range queries |
| **KEY** | MySQL internal hashing (like HASH) |

---

# 11. Benefits & Limits

**Benefits:**

- Faster queries with partition key in WHERE
- Easy drop old data: `ALTER TABLE DROP PARTITION p2022`
- Maintenance per partition (index rebuild)

**Limits:**

- All unique keys must include partition key
- Cross-partition queries can be slower
- Not a replacement for indexing
- Max 8192 partitions

---

# 12. Partition Pruning Example

```sql
SELECT * FROM orders
WHERE created_at BETWEEN '2024-06-01' AND '2024-06-30';
-- Only scans p2024 partition
```

Without partition key in query → scans all partitions (full table scan behavior).

---

# Part 3: MySQL Sharding

---

# 13. Partitioning vs Sharding

| | Partitioning | Sharding |
|---|--------------|----------|
| Scope | Single MySQL server | Multiple servers |
| Transparency | MySQL native | Application / middleware |
| Scale | Disk/IO on one machine | Horizontal across nodes |
| Cross-shard JOIN | N/A (same server) | Expensive / avoided |

---

# 14. Sharding Strategies

```text
1. Range-based: user_id 1-1M → shard1, 1M-2M → shard2
   Pros: simple range queries  Cons: hot spots

2. Hash-based: shard = hash(user_id) % N
   Pros: even distribution  Cons: range queries hard

3. Directory-based: lookup table maps key → shard
   Pros: flexible rebalancing  Cons: lookup service SPOF
```

---

# 15. Sharding for Video Platform

```text
users          → shard by user_id (hash)
videos         → shard by video_id (hash)
watch_history  → shard by user_id (co-locate with user)
comments       → shard by video_id

Avoid cross-shard JOIN:
  - Denormalize creator_name on videos table
  - Application-level aggregation
  - Fan-out queries to all shards + merge (expensive)
```

---

# 16. Sharding Middleware

| Tool | Notes |
|------|-------|
| Vitess | YouTube uses; MySQL sharding, resharding |
| ProxySQL | Routing, read/write split |
| Application layer | Shard key in DAO — full control |
| Citus (Postgres) | Reference for distributed SQL |

---

# 17. Challenges

```text
1. Resharding when adding shards → consistent hashing, dual-write migration
2. Distributed transactions → avoid; use Saga per shard
3. Auto-increment IDs → UUID or snowflake IDs globally unique
4. Aggregates (COUNT global) → map-reduce per shard or OLAP warehouse
5. Referential integrity across shards → application enforced
```

---

# 18. Consistent Hashing (Interview)

```text
Place shards and keys on hash ring
Key maps to next clockwise shard
Adding shard moves only adjacent keys — minimal remapping
Virtual nodes improve balance
```

Used in: Cassandra, DynamoDB, CDN, cache clusters.

---

# Part 4: Combined Design Example

---

# 19. watch_history Table Design

```sql
-- Single server: partition by month
CREATE TABLE watch_history (
    user_id BIGINT NOT NULL,
    video_id BIGINT NOT NULL,
    watched_at DATETIME NOT NULL,
    progress_sec INT,
    PRIMARY KEY (user_id, video_id, watched_at)
)
PARTITION BY RANGE (TO_DAYS(watched_at)) (...);

-- At scale: shard by user_id % 64
-- Shard 0 on db-shard-0, Shard 1 on db-shard-1, ...
```

Write path: API → Kafka → consumer batches insert to correct shard.

---

# Interview Questions

## Why CDN for video but not API?

Video segments large, immutable, read-heavy — perfect for edge cache. API personalized, dynamic — harder to cache at edge.

## SQL vs NoSQL for metadata?

SQL fine until single-node limits; then shard MySQL or use Cassandra for high write (watch events).

## How does YouTube handle viral video?

Aggressive CDN caching, origin shield, possibly dedicated hot-spot handling.

## Partition vs index on created_at?

Index helps find rows; partition eliminates reading other partitions' files entirely.

## When to shard?

Single MySQL CPU/IO saturated, replication lag too high for writes, dataset exceeds practical single-node storage (~TB scale depending on hardware).

## Vitess role?

MySQL-compatible sharding layer — VTGate routes queries, VTablet manages shards, supports resharding online.

---

# Quick Revision

```text
Upload → S3 → Kafka → Transcode → segments → CDN
Playback → manifest → ABR → CDN segments
Partition: RANGE/LIST/HASH within one MySQL
Shard: hash(user_id) across DB servers
Avoid cross-shard JOIN; UUID IDs; consistent hashing
YouTube/Netflix: CDN + object store + async pipeline
```

---

*End of Day 35 HLD Netflix/YouTube + MySQL Partitions & Sharding*
