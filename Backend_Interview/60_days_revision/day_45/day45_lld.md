# Day 45 — LLD: Online Payment Gateway (Idempotency)

**Topics:** Payment flow · Idempotency keys · Webhooks · Failure handling · Interview Q&A

---

# 1. Problem Statement

Design an online payment gateway integration for an e-commerce platform:

- Process card/UPI payments
- **Idempotent** charge requests (retry-safe)
- Handle async webhook callbacks
- Support refunds
- Prevent double-charging

---

# 2. Functional Requirements

| Feature | Description |
|---------|-------------|
| charge | Debit customer for order |
| refund | Reverse partial/full payment |
| getStatus | Query payment by idempotency key or txn ID |
| webhook | Receive provider status updates |

---

# 3. Non-Functional Requirements

- Exactly-once **effect** (at-least-once delivery with idempotency)
- PCI compliance — never store raw card (use tokenization)
- Audit trail for all state transitions
- 99.99% availability for status queries

---

# 4. Core Entities

```text
PaymentRequest
  ├── idempotencyKey (client-generated UUID)
  ├── orderId
  ├── amount, currency
  └── paymentMethodToken

Payment
  ├── paymentId
  ├── idempotencyKey (unique index)
  ├── orderId
  ├── amount, status (PENDING|SUCCESS|FAILED|REFUNDED)
  ├── providerTxnId
  └── createdAt, updatedAt

PaymentEvent (audit)
  ├── eventId, paymentId, oldStatus, newStatus, source, timestamp
```

---

# 5. Idempotency — Core Design

## Why Idempotency?

```text
Client sends charge → network timeout → client retries
Without idempotency → double charge
With idempotency key → same result on retry
```

---

# Flow

```text
1. Client generates idempotencyKey (UUID) per logical operation
2. POST /payments with Idempotency-Key header
3. Server:
   a. Check if key exists → return cached response (200/201 same body)
   b. If not → create PENDING payment, call provider
   c. Store result keyed by idempotencyKey
4. Retries with same key → step 3a
```

---

# Java Implementation Sketch

```java
@Service
@Transactional
public class PaymentService {

    private final PaymentRepository paymentRepo;
    private final PaymentGatewayClient gateway;
    private final IdempotencyStore idempotencyStore;

    public PaymentResponse charge(ChargeRequest req, String idempotencyKey) {
        // Step 1: fast path — already processed
        Optional<PaymentResponse> cached = idempotencyStore.get(idempotencyKey);
        if (cached.isPresent()) return cached.get();

        // Step 2: acquire lock on idempotency key (prevent concurrent duplicate)
        idempotencyStore.acquireLock(idempotencyKey);

        try {
            cached = idempotencyStore.get(idempotencyKey);
            if (cached.isPresent()) return cached.get();

            // Step 3: create payment record PENDING
            Payment payment = paymentRepo.save(Payment.pending(req, idempotencyKey));

            // Step 4: call external gateway
            GatewayResult result = gateway.charge(
                req.getAmount(), req.getToken(), idempotencyKey);

            payment.applyResult(result);
            paymentRepo.save(payment);

            PaymentResponse response = PaymentResponse.from(payment);
            idempotencyStore.save(idempotencyKey, response, Duration.ofHours(24));
            return response;

        } finally {
            idempotencyStore.releaseLock(idempotencyKey);
        }
    }
}
```

---

# Idempotency Store Options

| Store | Pros | Cons |
|-------|------|------|
| DB unique constraint | Durable, simple | Slower |
| Redis SET NX + TTL | Fast lock | Need durability fallback |
| Hybrid | Redis lock + DB record | Production pattern |

```sql
CREATE TABLE idempotency_keys (
    idempotency_key VARCHAR(64) PRIMARY KEY,
    request_hash    VARCHAR(64) NOT NULL,
    response_body   JSON NOT NULL,
    status          VARCHAR(20) NOT NULL,
    created_at      TIMESTAMP NOT NULL,
    expires_at      TIMESTAMP NOT NULL
);
CREATE UNIQUE INDEX idx_payment_idempotency ON payments(idempotency_key);
```

---

# Conflict Detection

Same key + **different** request body → return `409 Conflict`.

```java
if (existing != null && !existing.requestHash().equals(hash(req))) {
    throw new IdempotencyConflictException(idempotencyKey);
}
```

---

# 6. State Machine

```text
PENDING ──success──→ SUCCESS ──refund──→ REFUNDED
   │                      │
   └──failure──→ FAILED   └──partial refund──→ PARTIALLY_REFUNDED
```

Use **State pattern** or enum transitions with validation.

---

# 7. Webhook Handler (Async Confirmation)

```java
@RestController
public class WebhookController {

    @PostMapping("/webhooks/stripe")
    public ResponseEntity<Void> handleStripe(
            @RequestBody String payload,
            @RequestHeader("Stripe-Signature") String sig) {

        if (!signatureVerifier.verify(payload, sig)) {
            return ResponseEntity.status(401).build();
        }

        WebhookEvent event = parser.parse(payload);
        webhookProcessor.process(event);  // idempotent by provider event ID
        return ResponseEntity.ok().build();
    }
}

@Service
public class WebhookProcessor {
    public void process(WebhookEvent event) {
        if (processedEventRepo.exists(event.getEventId())) return; // dedup

        Payment payment = paymentRepo.findByProviderTxnId(event.getTxnId())
            .orElseThrow();
        payment.transition(event.getStatus());
        paymentRepo.save(payment);
        processedEventRepo.save(new ProcessedEvent(event.getEventId()));
    }
}
```

---

# 8. Failure Scenarios

| Scenario | Handling |
|----------|----------|
| Gateway timeout after charge | Reconcile via webhook or polling; idempotency key on retry |
| Duplicate webhook | Dedup by provider event ID |
| Partial failure (DB save fails) | Transaction rollback; client retries with same key |
| Refund on pending | Reject — invalid state transition |

---

# 9. API Design

```text
POST /api/v1/payments
  Header: Idempotency-Key: <uuid>
  Body: { orderId, amount, currency, paymentMethodToken }
  → 201 Created | 200 OK (replay) | 409 Conflict

GET /api/v1/payments/{paymentId}
POST /api/v1/payments/{paymentId}/refund
  Header: Idempotency-Key: <uuid-for-refund>
```

---

# Interview Questions

## Q1. Idempotency key — client or server generated?

Client generates per logical operation (Stripe pattern). Server validates uniqueness.

## Q2. How long to store idempotency keys?

24–72 hours typical. Stripe keeps 24h. Longer for B2B reconciliation.

## Q3. Idempotency vs distributed transaction?

Idempotency handles retry semantics. 2PC/Saga handles cross-service atomicity — complementary.

## Q4. What if gateway succeeds but DB fails?

Orphan charge — reconciliation job polls gateway by idempotency key, aligns DB. Mention this proactively.

## Q5. @Transactional on charge method?

Yes for DB consistency. External gateway call **outside** transaction or use outbox pattern to avoid long-held locks.

---

# One-Line Revision

```text
Payment gateway LLD = idempotency key + unique DB index + lock + cached response; webhooks deduped by event ID; state machine for PENDING→SUCCESS→REFUNDED.
```

---

*End of Day 45 LLD*
