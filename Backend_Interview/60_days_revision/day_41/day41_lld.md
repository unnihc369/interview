# Day 41 — LLD: WhatsApp Chat Server

**Topics:** Requirements · Messaging Model · Delivery Semantics · Groups · Presence · Scale · Interview Questions

---

# 1. Problem Statement

Design a real-time chat system like WhatsApp supporting 1:1 messaging, group chats, delivery/read receipts, online presence, and media sharing.

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| 1:1 Chat | Send/receive text messages |
| Group Chat | Create group, add/remove members, admin roles |
| Delivery Status | Sent → Delivered → Read |
| Media | Images, videos, documents |
| Presence | Online / last seen |
| Message History | Paginated history on login |
| Push Notification | Notify offline users |

---

# 3. Non-Functional Requirements

- Low latency (< 100ms message delivery when online)
- High availability (99.99%)
- Message ordering per chat
- At-least-once delivery with deduplication
- End-to-end encryption (mention in interview — optional deep dive)
- Support billions of messages/day at scale

---

# 4. Core Entities

```text
User
  ├── userId, phone, name, profilePic, lastSeen

Chat (Conversation)
  ├── chatId, type (DIRECT, GROUP), participants, createdAt

Message
  ├── messageId, chatId, senderId, content, type (TEXT, IMAGE, ...)
  ├── timestamp, status (SENT, DELIVERED, READ)

Group
  ├── groupId, name, adminIds, memberIds

DeviceSession
  ├── userId, deviceId, connectionId, lastActive
```

---

# 5. High-Level Architecture (LLD + Scale Bridge)

```text
Mobile/Web Client
      ↓ (WebSocket / long polling)
Connection Gateway (stateful, sticky sessions)
      ↓
Chat Service (stateless)
      ↓
Message Queue (Kafka) ──→ Message Store (Cassandra/HBase)
      ↓
Push Notification Service (APNs/FCM)
      ↓
Media Service (S3 + CDN)
```

---

# 6. Message Send — Sequence Diagram

```text
UserA -> Gateway: SEND_MESSAGE(chatId, content)
Gateway -> ChatService: processMessage()
ChatService:
  1. Generate messageId (Snowflake / UUID)
  2. Persist to MessageStore (partition by chatId)
  3. Publish to Kafka topic chat-events
  4. Return ACK to UserA (status SENT)

Kafka Consumer (Delivery):
  1. Lookup online sessions for recipients
  2. If online → push via Gateway WebSocket (DELIVERED)
  3. If offline → enqueue Push Notification
  4. Update delivery status in DB
```

---

# 7. Chat ID Design

```text
Direct chat: hash(sorted(userA, userB)) → deterministic chatId
Group chat:  UUID on group creation
```

Deterministic direct chat ID avoids duplicate conversations.

```java
public static String directChatId(String u1, String u2) {
    String[] ids = {u1, u2};
    Arrays.sort(ids);
    return "d_" + DigestUtils.sha256Hex(ids[0] + ":" + ids[1]).substring(0, 16);
}
```

---

# 8. Message Storage Schema

**Cassandra-style (wide column):**

```text
Partition key: chatId
Clustering key: timestamp DESC
Columns: messageId, senderId, content, type, status
```

Query: `SELECT * FROM messages WHERE chatId = ? ORDER BY timestamp DESC LIMIT 50`

**User inbox index:**

```text
Partition: userId
Clustering: lastMessageTime DESC
Columns: chatId, preview, unreadCount
```

---

# 9. Delivery & Read Receipts

```java
public enum MessageStatus { SENT, DELIVERED, READ }

// Recipient ACK
void onDelivered(String messageId, String userId) {
    statusRepo.update(messageId, userId, DELIVERED);
    notifySender(messageId, DELIVERED);
}

// User opens chat
void onRead(String chatId, String userId, long upToTimestamp) {
    statusRepo.markRead(chatId, userId, upToTimestamp);
    notifySenderReadReceipt(chatId, userId);
}
```

Idempotency: client sends `clientMessageId`; server dedupes.

---

# 10. Group Chat Rules

| Action | Who |
|--------|-----|
| Add member | Admin |
| Remove member | Admin or self-leave |
| Change group name | Admin |
| Send message | All members |

Store `group_members` in Redis set for fast membership check.

---

# 11. Presence (Online / Last Seen)

```text
On connect:    SET presence:{userId} = online EX 60
Heartbeat:     EXPIRE refresh every 30s
On disconnect: DEL presence:{userId}; SET lastseen:{userId} = now()
```

Privacy setting: hide last seen from non-contacts.

---

# 12. Media Messages

```text
1. Client requests upload URL from MediaService
2. Upload to S3 (pre-signed PUT)
3. Send message with mediaUrl + thumbnail metadata
4. Recipients download via CDN
```

Do not route binary through chat gateway.

---

# 13. WebSocket Gateway Design

- **Sticky sessions** — user always hits same gateway node
- **Cross-node fan-out** — Redis Pub/Sub channel `user:{userId}` for delivery when recipient on different gateway
- Connection registry: `userId → Set<connectionId>`

---

# 14. Key Classes (LLD)

```java
public interface ChatService {
    Message sendMessage(SendMessageRequest req);
    List<Message> getHistory(String chatId, String cursor, int limit);
    void markRead(String chatId, String userId);
}

public interface ConnectionManager {
    void deliver(String userId, MessagePayload payload);
    boolean isOnline(String userId);
}

public interface MessageRepository {
    void save(Message message);
    List<Message> findByChatId(String chatId, Instant before, int limit);
}
```

---

# 15. End-to-End Encryption (Interview Bonus)

```text
Signal Protocol / Double Ratchet
Server stores ciphertext only
Keys on device — server cannot read content
Trade-off: no server-side search on message body
```

---

# 16. Edge Cases

- Duplicate message on retry → dedupe by `clientMessageId`
- User blocked → reject send with clear error
- Message ordering across regions → partition Kafka by `chatId`
- Deleted message → tombstone flag; sync to clients
- Large group (256+) → fan-out on write vs read (see HLD day 42 contrast)

---

# 17. Fan-out Strategy

| Model | When |
|-------|------|
| Fan-out on write | Small groups, WhatsApp groups |
| Fan-out on read | Celebrity broadcast, Twitter-like |

WhatsApp 1:1 and small groups: **fan-out on write** to per-user inbox.

---

# Interview Questions

## Q1. How guarantee message order?
Single partition per `chatId` in Kafka; monotonic `timestamp` or sequence number per chat.

## Q2. How handle offline messages?
Persist first; on reconnect client pulls `since lastSyncToken`; push for urgent.

## Q3. WebSocket vs MQTT?
WebSocket for mobile apps with custom protocol; MQTT for IoT — WhatsApp uses custom binary protocol on TCP.

## Q4. How scale connection gateway?
Millions of connections — horizontal gateway nodes + Redis pub/sub + connection registry sharded by userId.

## Q5. CAP trade-off for chat?
AP for message delivery (availability + partition tolerance); eventual consistency for read receipts acceptable.

## Q6. Delete for everyone?
Server marks message deleted; broadcast `DELETE_MESSAGE` event; clients remove from UI; retention policy for legal hold.

---

# Quick Revision

```text
WhatsApp LLD → WebSocket gateway + ChatService + Kafka + Cassandra(chatId partition).
Delivery: SENT→DELIVERED→READ; presence in Redis; media via S3; fan-out on write for groups.
```

---

*End of Day 41 LLD*
