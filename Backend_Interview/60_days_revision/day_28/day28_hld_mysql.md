# Day 28 — HLD + MySQL: WhatsApp Messenger

## Topics
- WhatsApp High Level Design
- Messaging Architecture
- Real-time Delivery
- MySQL Query Optimization & Indexes

---

# 1. WhatsApp — Functional Requirements

## Core Features

- One-to-one messaging
- Group messaging (up to 256 members)
- Online/offline status (last seen)
- Message delivery status (sent, delivered, read)
- Media sharing (images, video, documents)
- End-to-end encryption (mention in interview)

## Out of Scope (Initial Version)

- Voice/video calls
- Stories/status
- Payments

---

# 2. Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Low latency | Message delivery < 500ms |
| High availability | 99.99% |
| Scalability | Billions of messages/day |
| Durability | Messages never lost |
| Ordering | Per-chat message order preserved |

---

# 3. High Level Architecture

```text
                    ┌─────────────┐
                    │   Clients   │
                    │ (Mobile/Web)│
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │     CDN     │  (media delivery)
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │Load Balancer│
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         │                 │                 │
  ┌──────▼──────┐  ┌───────▼──────┐  ┌──────▼──────┐
  │ Chat Service│  │Presence Svc  │  │ Media Service│
  └──────┬──────┘  └───────┬──────┘  └──────┬──────┘
         │                 │                 │
  ┌──────▼──────┐  ┌───────▼──────┐  ┌──────▼──────┐
  │ Message Queue│  │    Redis     │  │ Object Store│
  │   (Kafka)    │  │ (presence)   │  │  (S3/Blob)  │
  └──────┬──────┘  └──────────────┘  └─────────────┘
         │
  ┌──────▼──────┐
  │  Message DB  │  (Cassandra / sharded MySQL)
  │  (per chat)  │
  └─────────────┘
```

---

# 4. Message Flow — One-to-One

```text
1. User A sends message to User B
2. Chat Service validates auth, stores message
3. If User B online → push via WebSocket/long polling
4. If User B offline → store + push notification (APNs/FCM)
5. User B ACK → update status to DELIVERED
6. User B reads → update status to READ
```

---

# 5. Real-Time Connection Model

## WebSocket Gateway

```text
User connects → Gateway assigns connection server
Connection server registry in Redis:
  userId → serverId + connectionId
```

When message arrives for User B:

```text
Lookup userId in Redis → find serverId
Forward message to that WebSocket server → deliver to client
```

---

# 6. Message Storage Design

## Option A — Cassandra (WhatsApp actual)

- Partition by `chat_id`
- Cluster by `message_id` (time-ordered UUID)
- Optimized for write-heavy, time-range reads

## Option B — Sharded MySQL (Interview Friendly)

```sql
-- Shard key: chat_id % num_shards

CREATE TABLE messages (
    id          BIGINT PRIMARY KEY,
    chat_id     BIGINT NOT NULL,
    sender_id   BIGINT NOT NULL,
    content     TEXT,
    media_url   VARCHAR(512),
    msg_type    ENUM('TEXT','IMAGE','VIDEO','DOC'),
    status      ENUM('SENT','DELIVERED','READ'),
    created_at  TIMESTAMP(3) NOT NULL,
    INDEX idx_chat_time (chat_id, created_at DESC)
);
```

---

# 7. Chat & User Schema (MySQL)

```sql
CREATE TABLE users (
    id         BIGINT PRIMARY KEY,
    phone      VARCHAR(20) UNIQUE,
    name       VARCHAR(100),
    last_seen  TIMESTAMP,
    INDEX idx_phone (phone)
);

CREATE TABLE chats (
    id         BIGINT PRIMARY KEY,
    type       ENUM('DIRECT','GROUP'),
    created_at TIMESTAMP
);

CREATE TABLE chat_members (
    chat_id    BIGINT,
    user_id    BIGINT,
    joined_at  TIMESTAMP,
    PRIMARY KEY (chat_id, user_id),
    INDEX idx_user_chats (user_id, chat_id)
);

CREATE TABLE group_info (
    chat_id    BIGINT PRIMARY KEY,
    name       VARCHAR(200),
    admin_id   BIGINT
);
```

---

# 8. Group Messaging

```text
Message to group chat_id
  → Store once in messages table
  → Fan-out to online members via WebSocket
  → Offline members: push notification only (no per-user message copy)
```

For very large groups, use **pull model** (client fetches on open) instead of push fan-out.

---

# 9. Media Handling

```text
1. Client uploads media to Media Service
2. Store in S3/Blob; return media_url
3. Message contains media_url reference (not binary in DB)
4. CDN serves media on download
5. Generate thumbnail async
```

---

# 10. Message Status & Read Receipts

```sql
CREATE TABLE message_receipts (
    message_id  BIGINT,
    user_id     BIGINT,
    status      ENUM('DELIVERED','READ'),
    updated_at  TIMESTAMP,
    PRIMARY KEY (message_id, user_id)
);
```

For direct chat: update main message status.  
For groups: per-user receipt table.

---

# 11. Presence / Last Seen

Store in Redis (fast, ephemeral):

```text
Key: presence:{userId}
Value: { online: true, lastSeen: timestamp }
TTL: refresh on heartbeat
```

Don't hit DB for every online check.

---

# 12. End-to-End Encryption (Mention)

```text
Server stores encrypted payload
Keys exchanged client-side (Signal protocol)
Server cannot read message content
```

Shows security awareness in HLD round.

---

# 13. Scaling Strategies

| Component | Scale Approach |
|-----------|----------------|
| Chat Service | Horizontal pods behind LB |
| WebSocket | Sticky sessions + Redis registry |
| Message DB | Shard by chat_id |
| Media | Object store + CDN |
| Push notifications | Async queue workers |
| Hot users/groups | Dedicated partition, rate limit |

---

# 14. Bottlenecks

- WebSocket connection count per server
- Group fan-out for large groups
- Media upload bandwidth
- Message ordering across regions
- DB hot partitions (celebrity chats)

---

# 15. MySQL Query Optimization

---

# Slow Query Identification

```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1;

-- Find queries with EXPLAIN
EXPLAIN ANALYZE
SELECT * FROM messages
WHERE chat_id = 12345
ORDER BY created_at DESC
LIMIT 50;
```

---

# Index Design for Messenger Queries

## Query: Recent messages in chat

```sql
SELECT * FROM messages
WHERE chat_id = ?
ORDER BY created_at DESC
LIMIT 50;
```

**Index:**

```sql
CREATE INDEX idx_chat_created ON messages (chat_id, created_at DESC);
```

Covers WHERE + ORDER BY — avoids filesort.

---

## Query: User's chat list with last message

```sql
SELECT c.id, c.type, m.content, m.created_at
FROM chat_members cm
JOIN chats c ON c.id = cm.chat_id
LEFT JOIN messages m ON m.id = (
    SELECT id FROM messages
    WHERE chat_id = cm.chat_id
    ORDER BY created_at DESC LIMIT 1
)
WHERE cm.user_id = ?
ORDER BY m.created_at DESC;
```

**Indexes:**

```sql
INDEX idx_user_chats (user_id, chat_id)   -- on chat_members
INDEX idx_chat_created (chat_id, created_at DESC)  -- on messages
```

Consider **denormalized `chat_last_message` table** for inbox view (write on new message).

---

## Query: Unread count

```sql
SELECT COUNT(*) FROM messages
WHERE chat_id = ? AND created_at > ? AND sender_id != ?;
```

**Index:**

```sql
INDEX idx_chat_unread (chat_id, created_at, sender_id)
```

Or maintain unread counter in Redis/DB per user per chat.

---

# 16. Index Types & Rules

| Index Type | Use Case |
|------------|----------|
| PRIMARY KEY (clustered) | Row lookup by id |
| Composite | Multi-column filters + sort |
| Covering index | Query answered from index alone |
| UNIQUE | phone, email dedup |

---

# Leftmost Prefix Rule

```sql
INDEX (chat_id, created_at, sender_id)
```

Works for:

```sql
WHERE chat_id = ?
WHERE chat_id = ? AND created_at > ?
WHERE chat_id = ? AND created_at > ? AND sender_id = ?
```

Does NOT work for:

```sql
WHERE sender_id = ?   -- skips leading chat_id
WHERE created_at > ?  -- skips leading chat_id
```

---

# 17. EXPLAIN Key Columns

| Column | Meaning |
|--------|---------|
| `type` | `ALL`=full scan (bad), `ref`/`range`=index, `const`=best |
| `key` | Index actually used |
| `rows` | Estimated rows examined |
| `Extra` | `Using filesort`, `Using temporary` = optimize |

---

# 18. Optimization Techniques

| Technique | Example |
|-----------|---------|
| Add composite index | `(chat_id, created_at DESC)` |
| Avoid SELECT * | Fetch only needed columns |
| Pagination | Cursor-based: `WHERE created_at < ? LIMIT 50` |
| Denormalization | `chat_inbox` table with last_msg preview |
| Partitioning | `PARTITION BY HASH(chat_id)` for large tables |
| Read replicas | Read-heavy inbox queries |
| Cache | Redis for recent 50 messages per chat |

---

# 19. Cursor Pagination (Better than OFFSET)

```sql
-- BAD: OFFSET grows slow
SELECT * FROM messages WHERE chat_id = 1 ORDER BY created_at DESC LIMIT 50 OFFSET 10000;

-- GOOD: keyset pagination
SELECT * FROM messages
WHERE chat_id = 1 AND created_at < '2026-05-29 10:00:00'
ORDER BY created_at DESC
LIMIT 50;
```

Uses index efficiently regardless of page depth.

---

# 20. Interview Questions

## Q1. How does WhatsApp deliver messages in real time?

WebSocket connection + Redis user-to-server mapping + push to correct gateway.

## Q2. How to store billions of messages?

Shard by chat_id; time-series friendly store (Cassandra) or partitioned MySQL.

## Q3. Why not one MySQL table without sharding?

Write/read hotspot, storage limits, backup/restore time.

## Q4. Group message fan-out problem?

For small groups push; for large groups pull on open; avoid N copies in DB.

## Q5. Index for chat message history?

Composite `(chat_id, created_at DESC)`.

## Q6. Why denormalize inbox?

Avoid expensive subquery for last message on every inbox load.

## Q7. Read vs write optimization?

Messenger is write-heavy; optimize inserts (batch, async) and recent-read cache.

## Q8. How to handle message ordering?

Monotonic ID per chat (snowflake/time-UUID); single partition per chat.

---

# Quick Revision

```text
WhatsApp HLD = WebSocket gateway + message queue + sharded storage + Redis presence + CDN media
MySQL tuning = composite indexes + EXPLAIN + cursor pagination + denormalize hot reads
```

---

*End of Day 28 HLD + MySQL*
