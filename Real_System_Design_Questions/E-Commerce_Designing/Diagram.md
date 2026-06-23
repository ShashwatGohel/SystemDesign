# E-Commerce System Architecture

## High Level Architecture

```mermaid
flowchart TB

    USER[User]

    PRODUCT[Product Service]

    CART[Cart Service]

    INVENTORY[Inventory Service]

    ORDER[Order Service]

    PAYMENT[Payment Service]

    SHIPPING[Shipping Service]

    NOTIFY[Notification Service]

    TRACKING[Tracking Service]

    ANALYTICS[Analytics Service]

    REDIS[(Redis)]

    DB[(Databases)]

    KAFKA[Kafka Cluster]

    USER --> PRODUCT
    USER --> CART
    USER --> ORDER

    PRODUCT --> REDIS
    PRODUCT --> DB

    CART --> REDIS

    ORDER --> INVENTORY
    ORDER --> PAYMENT

    ORDER --> KAFKA

    KAFKA --> NOTIFY
    KAFKA --> SHIPPING
    KAFKA --> ANALYTICS

    SHIPPING --> TRACKING
```

---

## Product Browsing Flow

```mermaid
flowchart LR

    USER[User]

    REDIS[(Redis Cache)]

    DB[(Product Database)]

    USER --> REDIS

    REDIS --> USER

    REDIS --> DB

    DB --> REDIS
```

---

## Cart Flow

```mermaid
flowchart LR

    USER[User]

    CART[Cart Service]

    REDIS[(Redis)]

    USER --> CART

    CART --> REDIS
```

---

## Inventory Reservation Flow

```mermaid
flowchart TD

    CHECKOUT[Checkout Started]

    INVENTORY[Inventory Service]

    RESERVE[Reserve Inventory]

    PAYMENT[Payment Service]

    CHECKOUT --> INVENTORY

    INVENTORY --> RESERVE

    RESERVE --> PAYMENT
```

---

## Order Confirmation Flow

```mermaid
flowchart LR

    PAYMENT[Payment Success]

    ORDER[Order Service]

    KAFKA[ORDER_CONFIRMED]

    PAYMENT --> ORDER

    ORDER --> KAFKA
```

---

## Event Driven Architecture

```mermaid
flowchart TB

    ORDER[Order Service]

    KAFKA[Kafka]

    NOTIFY[Notification Service]

    SHIPPING[Shipping Service]

    ANALYTICS[Analytics Service]

    ORDER --> KAFKA

    KAFKA --> NOTIFY

    KAFKA --> SHIPPING

    KAFKA --> ANALYTICS
```

---

## Shipping Flow

```mermaid
flowchart LR

    CONFIRM[ORDER_CONFIRMED]

    SHIP[ORDER_SHIPPED]

    OUT[OUT_FOR_DELIVERY]

    DELIVER[ORDER_DELIVERED]

    CONFIRM --> SHIP

    SHIP --> OUT

    OUT --> DELIVER
```

---

## Saga Pattern Rollback

```mermaid
flowchart TD

    INV[Inventory Reserved]

    ORDER[Order Created]

    PAY[Payment Failed]

    CANCEL[Cancel Order]

    RELEASE[Release Inventory]

    INV --> ORDER

    ORDER --> PAY

    PAY --> CANCEL

    CANCEL --> RELEASE
```

---

## Complete End-To-End Flow

```mermaid
flowchart TB

    USER[User]

    PRODUCT[Product Service]

    REDIS[(Redis Cache)]

    INVENTORY[Inventory Service]

    PAYMENT[Payment Service]

    ORDER[Order Service]

    KAFKA[Kafka]

    NOTIFY[Notification Service]

    SHIPPING[Shipping Service]

    TRACKING[Tracking Service]

    ANALYTICS[Analytics Service]

    USER --> PRODUCT

    PRODUCT --> REDIS

    USER --> INVENTORY

    INVENTORY --> PAYMENT

    PAYMENT --> ORDER

    ORDER --> KAFKA

    KAFKA --> NOTIFY

    KAFKA --> SHIPPING

    SHIPPING --> TRACKING

    KAFKA --> ANALYTICS
```
