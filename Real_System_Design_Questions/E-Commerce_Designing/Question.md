# Practice Question 08 - Design an E-Commerce System (Amazon / Flipkart)

## Interview Question

Design a scalable E-Commerce Platform like Amazon or Flipkart.

Requirements:

- Users Can Browse Products
- Users Can Search Products
- Users Can Add Products To Cart
- Users Can Place Orders
- Users Can Make Payments
- Inventory Should Be Updated
- Users Can Track Orders
- Support 100 Million Users

Design the high-level architecture.

Mention:

- Product Service
- Cart Service
- Inventory Service
- Order Service
- Payment Service
- Notification Service
- Shipping Service
- Redis
- Kafka
- Saga Pattern

Also explain:

1. Product Browsing Flow
2. Cart Flow
3. Inventory Reservation
4. Payment Flow
5. Order Lifecycle
6. Failure Handling

---

# Interview Answer

For an E-Commerce Platform, the primary goals are:

```text
High Availability

Low Latency

Scalability

Inventory Consistency

Reliable Payments
```

Since inventory and payments are involved, correctness is extremely important.

---

## Services

The system can be divided into:

```text
Product Service

Cart Service

Inventory Service

Order Service

Payment Service

Notification Service

Shipping Service

Tracking Service

Analytics Service
```

Each service owns its own database following:

```text
Database Per Service Pattern
```

---

## Product Service

Responsible for:

```text
Product Catalog

Product Details

Pricing

Reviews

Ratings
```

---

## Redis Product Cache

During events such as:

```text
Big Billion Days

Prime Day

Festive Sale
```

product pages receive extremely high traffic.

Frequently accessed data should be cached:

```text
Product Name

Description

Images

Ratings

Reviews

Price

Discount
```

### Why Redis?

```text
Fast Reads

Low Latency

Reduced Database Load
```

---

## Inventory Service

Responsible for:

```text
Stock Management

Inventory Reservation

Inventory Updates
```

Inventory Database remains:

```text
Source Of Truth
```

for stock information.

### Why Not Store Inventory Only In Redis?

```text
Stale Data

Overselling Risk

Incorrect Stock Counts
```

---

## Cart Service

Cart data is temporary and changes frequently.

Cart information is stored in:

```text
Redis
```

Example:

```text
cart:user123

iPhone 17

AirPods

Laptop
```

Redis TTL automatically removes abandoned carts.

### Why Redis?

```text
Fast Access

Frequent Updates

Automatic Expiry
```

---

## Overselling Problem

Suppose:

```text
Stock = 1
```

Two users attempt to purchase simultaneously.

Without protection:

```text
2 Orders

1 Product
```

Overselling occurs.

---

## Inventory Reservation

During checkout:

```text
Checkout Started
↓
Inventory Reserved
```

The stock is temporarily locked.

If checkout succeeds:

```text
Reservation
↓
Permanent Deduction
```

If checkout fails:

```text
Reservation Released
```

---

## Payment Service

Handles:

```text
Payment Processing

Refunds

Payment Verification
```

### Idempotency

To prevent duplicate charges:

```text
Idempotency Key
```

is used.

Even if the client retries:

```text
Same Request
↓
Single Payment Execution
```

---

## Kafka

After payment succeeds:

```text
ORDER_CONFIRMED
```

event is published.

Kafka provides:

```text
Loose Coupling

Scalability

Asynchronous Processing

Event Driven Architecture
```

---

## Event Driven Architecture

Multiple services consume:

```text
ORDER_CONFIRMED
```

Example:

```text
Notification Service

Shipping Service

Analytics Service
```

A service may consume an event and publish another event.

---

## Notification Service

Sends:

```text
Email

SMS

Push Notifications
```

Examples:

```text
Order Confirmed

Order Shipped

Out For Delivery

Order Delivered
```

---

## Shipping Service

Responsible for:

```text
Shipment Creation

Logistics

Delivery Workflow
```

Events:

```text
ORDER_SHIPPED

OUT_FOR_DELIVERY

ORDER_DELIVERED
```

---

## Tracking Service

Consumes shipping events.

Displays:

```text
Live Shipment Status

Tracking Information
```

to customers.

---

## Saga Pattern

Microservices own separate databases.

Traditional ACID transactions across services are difficult.

Instead:

```text
Saga Pattern
```

is used.

Example:

```text
Inventory Reserved
✓

Order Created
✓

Payment Failed
✗
```

Compensating Actions:

```text
Release Inventory

Cancel Order
```

---

## Complete End-To-End Order Flow

During a Big Billion Days sale, product pages receive extremely high traffic. Therefore, frequently accessed data such as:

```text
Product Details

Images

Ratings

Reviews

Prices

Discounts
```

are cached in Redis.

Inventory remains in the Inventory Database because stale inventory data can lead to overselling.

---

### Product Browsing

```text
User Opens Product Page
↓
Redis Cache

(Cache Hit)
↓
Return Product Details

(Cache Miss)
↓
Database
↓
Update Redis
↓
Return Product Details
```

---

### Add To Cart

```text
User Adds Product
↓
Cart Service
↓
Redis
```

Redis TTL automatically removes abandoned carts.

---

### Checkout & Inventory Reservation

```text
Checkout Started
↓
Inventory Service
↓
Reserve Inventory
```

Inventory is temporarily locked.

---

### Payment Processing

```text
Payment Service
↓
Process Payment
```

Idempotency Keys ensure:

```text
No Duplicate Charges
```

---

### Order Confirmation

```text
Payment Success
↓
Order Service
↓
ORDER_CONFIRMED
↓
Kafka
```

Consumers:

```text
Notification Service

Shipping Service

Analytics Service
```

---

### Shipping Flow

```text
ORDER_CONFIRMED
↓
Shipping Service
↓
ORDER_SHIPPED
↓
OUT_FOR_DELIVERY
↓
ORDER_DELIVERED
```

---

### Tracking

```text
Tracking Service
↓
Updates Customer Screen
```

---

### Delivery Completed

```text
ORDER_DELIVERED
↓
Kafka
```

Consumers:

```text
Analytics Service

Reporting Service

Recommendation Service
```

---

### Failure Handling

Suppose:

```text
Inventory Reserved
✓

Payment Failed
✗
```

Saga Pattern executes:

```text
Release Inventory

Cancel Order
```

System returns to a consistent state.

---

# Key Design Decisions

```text
Redis
→ Product Cache

Redis TTL
→ Cart Expiry

Inventory Database
→ Source Of Truth

Inventory Reservation
→ Prevent Overselling

Idempotency Keys
→ Prevent Duplicate Payments

Kafka
→ Event Driven Architecture

Saga Pattern
→ Distributed Transactions

Tracking Service
→ Shipment Visibility
```
