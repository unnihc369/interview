# Day 36 — LLD: Online Auction System

**Topics:** Requirements · Class Design · Bidding · Concurrency · Design Patterns

---

# 1. Problem Statement

Design an online auction platform where sellers list items, buyers place bids, and the highest bidder wins when the auction ends.

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| Create Auction | Seller sets start price, reserve, end time |
| Place Bid | Buyer bids if amount > current highest |
| Auto-extend | Optional anti-sniping (extend if bid in last N minutes) |
| Close Auction | Declare winner at end time |
| Watchlist | Users track auctions |

---

# 3. Non-Functional Requirements

- Strong consistency on bid acceptance (no lost highest bid)
- Handle concurrent bids on hot items
- Audit trail of all bids
- Notify outbid users in real time

---

# 4. Core Classes

```text
User
  ├── id, name, email

Auction
  ├── id, item, seller, startPrice, reservePrice
  ├── startTime, endTime, status (SCHEDULED, LIVE, CLOSED)
  └── currentHighestBid

Bid
  ├── id, auctionId, bidderId, amount, timestamp

AuctionService
  ├── createAuction(), placeBid(), closeAuction()

BidRepository / AuctionRepository
```

---

# 5. Enums

```java
public enum AuctionStatus {
    SCHEDULED, LIVE, CLOSED, CANCELLED
}

public enum BidResult {
    ACCEPTED, OUTBID, AUCTION_CLOSED, BID_TOO_LOW, RESERVE_NOT_MET
}
```

---

# 6. Place Bid — Sequence

```text
Buyer -> AuctionController.placeBid(auctionId, amount)
     -> AuctionService.placeBid()
         -> lock(auctionId)  // distributed lock or DB row lock
         -> validate auction LIVE and amount > currentHighest
         -> persist Bid
         -> update Auction.currentHighestBid
         -> publish OutbidEvent (Kafka/WebSocket)
     -> return BidResult.ACCEPTED
```

---

# 7. Concurrency Strategy

| Approach | Trade-off |
|----------|-----------|
| `SELECT FOR UPDATE` on auction row | Simple, DB bottleneck |
| Optimistic locking `@Version` | Retry on conflict |
| Redis distributed lock | Fast, needs lock TTL care |
| Single-partition Kafka per auction | Serialize bids per auction |

Interview answer: **pessimistic row lock** or **optimistic retry** for LLD; mention Redis at scale.

---

# 8. Key APIs

```java
public interface AuctionService {
    Auction createAuction(CreateAuctionRequest req);
    BidResult placeBid(Long auctionId, Long bidderId, BigDecimal amount);
    void closeAuction(Long auctionId);
    List<Bid> getBidHistory(Long auctionId);
}
```

---

# 9. Anti-Sniping Extension

```java
if (auction.getEndTime().minusMinutes(5).isBefore(Instant.now())) {
    auction.setEndTime(auction.getEndTime().plusMinutes(5));
}
```

Prevents last-second bid and immediate close.

---

# 10. Design Patterns

| Pattern | Use |
|---------|-----|
| Observer | Notify watchers on new bid |
| Strategy | Payment settlement after win |
| State | Auction lifecycle transitions |
| Repository | Persistence abstraction |

---

# 11. Class Diagram (Text)

```text
Seller 1 ---- * Auction
Buyer   1 ---- * Bid
Auction 1 ---- * Bid
Auction * ---- 1 Item

class AuctionService {
  + placeBid(auctionId, bidderId, amount): BidResult
  + closeAuction(auctionId): void
}
```

---

# 12. Edge Cases

- Bid equals current highest → reject (strictly greater)
- Auction ended mid-request → return `AUCTION_CLOSED`
- Reserve price not met → no winner or seller discretion
- Duplicate bid idempotency via client token

---

# Interview Questions

## Q1. How prevent lost updates on concurrent bids?

Row-level lock or atomic `UPDATE ... WHERE current_price < :bid` with affected-rows check.

## Q2. How scale to millions of auctions?

Shard by `auctionId`; hot auctions on dedicated partition; read replicas for browse; write master for bids.

## Q3. Real-time updates?

WebSocket/SSE to clients subscribed to `auctionId`; push on each accepted bid.

## Q4. Payment after win?

Separate `PaymentService`; saga or outbox pattern; do not block bid path on payment.

## Q5. Scheduler for auction close?

Quartz / `@Scheduled` scan ending auctions, or event-driven timer per auction in Redis.

---

# Quick Revision

```text
Online Auction → Auction + Bid + concurrent placeBid with lock/optimistic versioning.
Anti-sniping extends endTime; Observer/Kafka for outbid notifications.
```

---

*End of Day 36 LLD*
