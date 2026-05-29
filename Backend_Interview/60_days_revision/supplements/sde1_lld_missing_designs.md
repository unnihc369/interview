# SDE 1–2 Supplement — Missing LLD Designs

**Covers:** Rate Limiter · Restaurant · Deck of Cards · Amazon Locker · Notification System

**Related days:** Day 5 (patterns), Day 44 (Rate Limiter + Cache), Day 8 (Observer)

---

# 1. Rate Limiter

## Requirements

- `allowRequest(userId)` → true/false
- Max N requests per window (e.g. 100/minute)
- Thread-safe; optionally distributed (Redis)

## Algorithms

| Algorithm | Pros | Cons |
|-----------|------|------|
| **Token bucket** | Allows controlled bursts | Slightly complex |
| **Sliding window log** | Accurate | Memory per user |
| **Fixed window counter** | Simple | Burst at window boundary |
| **Sliding window counter** | Good balance | Approximate |

## Token Bucket (In-Memory)

```java
public class TokenBucketRateLimiter {
    private final long capacity;
    private final double refillRatePerSec;
    private double tokens;
    private long lastRefillNanos;

    public TokenBucketRateLimiter(long capacity, double refillRatePerSec) {
        this.capacity = capacity;
        this.refillRatePerSec = refillRatePerSec;
        this.tokens = capacity;
        this.lastRefillNanos = System.nanoTime();
    }

    public synchronized boolean allowRequest() {
        refill();
        if (tokens >= 1) {
            tokens -= 1;
            return true;
        }
        return false;
    }

    private void refill() {
        long now = System.nanoTime();
        double elapsed = (now - lastRefillNanos) / 1_000_000_000.0;
        tokens = Math.min(capacity, tokens + elapsed * refillRatePerSec);
        lastRefillNanos = now;
    }
}
```

## Per-User Limiter

```java
public class UserRateLimiter {
    private final ConcurrentHashMap<String, TokenBucketRateLimiter> buckets = new ConcurrentHashMap<>();

    public boolean allow(String userId) {
        return buckets.computeIfAbsent(userId, id -> new TokenBucketRateLimiter(100, 100.0 / 60))
                      .allowRequest();
    }
}
```

## Distributed (Redis)

```lua
-- KEYS[1] = key, ARGV[1] = limit, ARGV[2] = window_sec
local current = redis.call('INCR', KEYS[1])
if current == 1 then redis.call('EXPIRE', KEYS[1], ARGV[2]) end
return current <= tonumber(ARGV[1])
```

---

# 2. Restaurant / Food Ordering System

## Core Classes

```text
Restaurant
  ├── Menu
  │     └── MenuItem (id, name, price, category, available)
  ├── Order
  │     ├── OrderItem (menuItem, quantity, specialInstructions)
  │     └── OrderStatus (PLACED, PREPARING, READY, DELIVERED, CANCELLED)
  ├── Table (optional dine-in)
  └── Payment
```

## Key Flows

```text
Customer browses menu → adds to cart → places order
    → Restaurant accepts → Kitchen prepares → Ready → Delivered/Served
    → Payment processed
```

```java
public enum OrderStatus { PLACED, CONFIRMED, PREPARING, READY, OUT_FOR_DELIVERY, DELIVERED, CANCELLED }

public class Order {
    private final String id;
    private final String customerId;
    private final String restaurantId;
    private final List<OrderItem> items;
    private OrderStatus status;

    public void confirm() {
        if (status != OrderStatus.PLACED) throw new IllegalStateException();
        status = OrderStatus.CONFIRMED;
    }

    public void cancel() {
        if (status == OrderStatus.DELIVERED) throw new IllegalStateException();
        status = OrderStatus.CANCELLED;
    }
}
```

## Patterns

| Pattern | Use |
|---------|-----|
| **State** | OrderStatus transitions |
| **Strategy** | Payment (card, UPI, wallet) |
| **Observer** | Notify customer on status change |
| **Factory** | OrderItem from MenuItem |

## Interview Points

- Inventory: decrement on confirm, restore on cancel
- Concurrent orders: lock menu item stock or optimistic `@Version`
- Search/filter menu by category, price, veg/non-veg

---

# 3. Deck of Cards

## Requirements

- Standard 52-card deck
- Shuffle, deal N cards, reset
- Optional: Blackjack/ Poker hand evaluation

## Design

```java
public enum Suit { HEARTS, DIAMONDS, CLUBS, SPADES }
public enum Rank { TWO(2), THREE(3), /* ... */ ACE(14);
    private final int value;
    Rank(int value) { this.value = value; }
    public int getValue() { return value; }
}

public record Card(Suit suit, Rank rank) implements Comparable<Card> {
    public int compareTo(Card other) {
        return Integer.compare(this.rank.getValue(), other.rank.getValue());
    }
}

public class Deck {
    private final List<Card> cards = new ArrayList<>();

    public Deck() {
        for (Suit s : Suit.values())
            for (Rank r : Rank.values())
                cards.add(new Card(s, r));
    }

    public void shuffle() {
        Collections.shuffle(cards);
    }

    public Card deal() {
        if (cards.isEmpty()) throw new IllegalStateException("Empty deck");
        return cards.remove(cards.size() - 1);
    }

    public List<Card> deal(int n) {
        List<Card> hand = new ArrayList<>();
        for (int i = 0; i < n; i++) hand.add(deal());
        return hand;
    }
}
```

## Extensions (if time)

- Multiple decks for Blackjack
- `Hand` class with `evaluate()` for poker rank
- **Singleton** for game manager (optional)

---

# 4. Amazon Locker

## Requirements

- Customer orders package → assigned locker compartment
- OTP/code to open locker
- Time-limited pickup; return to warehouse if expired
- Multiple locker sizes (S/M/L)

## Classes

```text
LockerStation
  ├── Compartment (id, size, status: AVAILABLE/OCCUPIED/OUT_OF_SERVICE)
  ├── AccessCode (otp, expiry, compartmentId)
  └── Package (orderId, size, status)
```

```java
public enum CompartmentSize { SMALL, MEDIUM, LARGE }
public enum CompartmentStatus { AVAILABLE, OCCUPIED, MAINTENANCE }

public class LockerService {
    public AccessCode assignLocker(String orderId, CompartmentSize size) {
        Compartment comp = findAvailable(size)
            .orElseThrow(() -> new NoLockerAvailableException());
        comp.occupy(orderId);
        AccessCode code = AccessCode.generate(comp.getId(), Duration.ofHours(48));
        notificationService.send(code);
        return code;
    }

    public void pickup(String otp) {
        AccessCode code = codeStore.validate(otp);
        Compartment comp = station.getCompartment(code.compartmentId());
        comp.release();
        codeStore.invalidate(otp);
    }
}
```

## State Machine

```text
AVAILABLE → OCCUPIED (assign) → AVAILABLE (pickup or expiry)
```

## Interview Points

- Match package size to smallest fitting compartment
- Scheduled job for expired codes → release compartment
- Thread-safe compartment assignment (lock per station)

---

# 5. Notification System

## Requirements

- Send notification via Email, SMS, Push
- Template support
- Retry on failure
- User preferences (opt-out channels)

## Design

```java
public interface NotificationChannel {
    void send(Notification notification);
}

@Component
public class EmailChannel implements NotificationChannel { /* ... */ }

@Component
public class SmsChannel implements NotificationChannel { /* ... */ }

@Service
public class NotificationService {
    private final Map<ChannelType, NotificationChannel> channels;
    private final UserPreferenceRepository prefs;

    public void notify(String userId, NotificationTemplate template, Map<String, String> vars) {
        UserPreferences pref = prefs.findByUserId(userId);
        Notification msg = template.render(vars);

        for (ChannelType ch : pref.enabledChannels()) {
            channels.get(ch).send(msg);
        }
    }
}
```

## Patterns

| Pattern | Use |
|---------|-----|
| **Strategy** | Channel selection (Email/SMS/Push) |
| **Observer** | Event triggers notification |
| **Factory** | Create channel by type |
| **Template Method** | Base send with retry hook |

## Async / Scale

```text
Event → Message Queue (Kafka/SQS) → Worker pool → Channel adapters
```

Idempotency: dedupe by `(userId, eventId, channel)`.

---

# Interview Questions

## Q1. Token bucket vs sliding window?

Token bucket allows bursts up to capacity; sliding window enforces strict rate over rolling period.

## Q2. How to make rate limiter distributed?

Redis INCR + EXPIRE or Lua script for atomicity across instances.

## Q3. Restaurant order cancel after PREPARING?

Business rule — usually disallow or partial refund; use State pattern with valid transitions.

## Q4. Deck shuffle fairness?

`Collections.shuffle` uses Fisher-Yates — O(n), uniform distribution.

## Q5. Locker compartment size mismatch?

Find smallest available compartment ≥ package size (best-fit); else reject or alternate station.

---

# One-Line Revision

```text
Rate Limiter = token bucket + per-user map + Redis for distributed.
Restaurant = State (order) + Strategy (payment) + inventory.
Deck = Card/Suit/Rank + shuffle + deal.
Amazon Locker = compartment assignment + OTP + expiry job.
Notification = Strategy channels + Observer events + async queue.
```

---

*End of Missing LLD Designs Supplement*
