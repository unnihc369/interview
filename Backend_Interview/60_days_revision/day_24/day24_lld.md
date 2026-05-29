# Day 24 - LLD: Stock Brokerage System

---

# 1. Requirements

## Functional

- User registration and portfolio view
- Place buy/sell orders (market and limit)
- Order matching and execution
- View order history and holdings
- Real-time or near-real-time price feed

## Non-Functional

- High throughput for order placement
- Strong consistency for balances and holdings
- Audit trail for all trades

---

# 2. Core Entities

```text
User 1 ---- * Account
Account 1 ---- * Order
Account 1 ---- * Holding
Order * ---- 1 Stock
Trade (execution record) links buy/sell orders
```

```java
enum OrderType { MARKET, LIMIT }
enum OrderSide { BUY, SELL }
enum OrderStatus { PENDING, PARTIAL, FILLED, CANCELLED }

class Stock {
    private String symbol;
    private String name;
    private BigDecimal lastPrice;
}

class Account {
    private Long id;
    private Long userId;
    private BigDecimal cashBalance;
}

class Holding {
    private Long accountId;
    private String symbol;
    private int quantity;
    private BigDecimal avgCost;
}

class Order {
    private Long id;
    private Long accountId;
    private String symbol;
    private OrderSide side;
    private OrderType type;
    private int quantity;
    private BigDecimal limitPrice; // null for market
    private OrderStatus status;
    private int filledQty;
    private LocalDateTime createdAt;
}

class Trade {
    private Long id;
    private Long buyOrderId;
    private Long sellOrderId;
    private String symbol;
    private int quantity;
    private BigDecimal price;
    private LocalDateTime executedAt;
}
```

---

# 3. Order Matching Engine (Core)

## Price-Time Priority

```text
Buy orders: max price first, then earliest time
Sell orders: min price first, then earliest time
```

Use two priority queues or order books per symbol:

```text
BUY book:  max-heap by price, FIFO at same price
SELL book: min-heap by price, FIFO at same price
```

---

# 4. Matching Logic

```java
public void match(String symbol) {
    PriorityQueue<Order> buys = buyBook.get(symbol);
    PriorityQueue<Order> sells = sellBook.get(symbol);

    while (!buys.isEmpty() && !sells.isEmpty()) {
        Order buy = buys.peek();
        Order sell = sells.peek();

        if (buy.getLimitPrice() != null && buy.getLimitPrice().compareTo(sell.getLimitPrice()) < 0) {
            break; // no match
        }

        int qty = Math.min(buy.getRemainingQty(), sell.getRemainingQty());
        BigDecimal price = sell.getLimitPrice(); // price-time priority

        executeTrade(buy, sell, qty, price);
        updateOrderStatus(buy, sells, buys);
        updateOrderStatus(sell, sells, sells);
    }
}
```

---

# 5. Service Layer

```java
interface OrderService {
    Order placeOrder(PlaceOrderRequest req);
    void cancelOrder(Long orderId);
}

interface PortfolioService {
    Portfolio getPortfolio(Long accountId);
    BigDecimal getAvailableCash(Long accountId);
}

interface MatchingEngine {
    void submitOrder(Order order);
    void match(String symbol);
}
```

---

# 6. Pre-Trade Validation

| Check | Rule |
|-------|------|
| Buy | `cash >= quantity * price` |
| Sell | `holding.quantity >= quantity` |
| Market hours | Optional trading window |
| Lot size | Quantity multiple of lot |

Reserve funds/shares on order placement (avoid overselling):

```text
Place BUY  → hold cash
Place SELL → hold shares
Cancel     → release hold
Fill       → transfer cash/shares
```

---

# 7. Observer Pattern — Price Updates

```java
interface PriceListener {
    void onPriceUpdate(String symbol, BigDecimal price);
}

class PriceFeedService {
    private final List<PriceListener> listeners = new CopyOnWriteArrayList<>();

    public void publish(String symbol, BigDecimal price) {
        listeners.forEach(l -> l.onPriceUpdate(symbol, price));
    }
}
```

Portfolio UI subscribes for real-time P&L.

---

# 8. Class Diagram

```text
OrderService --> MatchingEngine
OrderService --> AccountService
MatchingEngine --> OrderBook (per symbol)
OrderBook --> PriorityQueue<Order> buys/sells
PortfolioService --> HoldingRepository
TradeService --> TradeRepository
PriceFeedService ..> PriceListener
```

---

# 9. Sequence — Place Buy Order

```text
Client -> OrderController: POST /orders
OrderController -> OrderService: placeOrder(req)
OrderService -> AccountService: validateAndHoldCash()
OrderService -> OrderRepository: save(PENDING)
OrderService -> MatchingEngine: submitOrder()
MatchingEngine -> match(symbol)
MatchingEngine -> TradeService: recordTrade()
MatchingEngine -> AccountService: settle(buy, sell)
MatchingEngine -> OrderRepository: update status
OrderService --> OrderController: OrderResponse
```

---

# 10. API Design

```text
POST   /api/v1/orders
DELETE /api/v1/orders/{id}
GET    /api/v1/orders?accountId=
GET    /api/v1/portfolio/{accountId}
GET    /api/v1/stocks/{symbol}/quote
WS     /ws/prices (optional real-time)
```

---

# 11. Concurrency & Scale

| Concern | Approach |
|---------|----------|
| Single symbol hot spot | Dedicated thread per symbol order book |
| Distributed matching | Partition by symbol across nodes |
| Idempotency | Client order ID dedup |
| Audit | Append-only trade log |

---

# 12. Edge Cases

| Case | Handling |
|------|----------|
| Partial fill | Update filledQty, status PARTIAL |
| Market order no liquidity | Reject or queue |
| Cancel pending order | Remove from book, release hold |
| Circuit breaker | Halt trading on symbol |

---

# Common Interview Questions

## Q1. Market vs Limit order?

Market: execute at best available price immediately. Limit: only at limit price or better.

## Q2. How to prevent negative balance?

Hold/reserve cash on order placement; settle on fill.

## Q3. Single-threaded vs multi-threaded matching?

Single-threaded per symbol simplifies consistency; shard symbols for scale.

## Q4. Difference from payment gateway LLD?

Brokerage adds order book, matching engine, holdings — not just transfer.

---

# One-Line Revision

```text
Stock Brokerage = order book + price-time matching + cash/share holds + trade settlement audit.
```

---

*End of Day 24 LLD*
