# Practice Question 07 - Design a Food Delivery System (Swiggy / Zomato)

## Interview Question

Design a scalable Food Delivery System like Swiggy or Zomato.

Requirements:

- Users Can Browse Restaurants
- Users Can Search Nearby Restaurants
- Users Can Place Orders
- Restaurants Can Accept Orders
- Delivery Partners Can Deliver Orders
- Live Order Tracking
- Order Notifications
- Support 100 Million Users

Design the high-level architecture.

Mention:

- Order Service
- Restaurant Service
- Delivery Service
- GeoHash
- Redis
- Kafka
- Notification Service
- Tracking Service
- Order Lifecycle

Also explain:

1. Nearby Restaurant Search
2. Driver Matching
3. Order Lifecycle
4. Live Tracking
5. Scaling Strategy

---

# Interview Answer

For a Food Delivery System, the primary goals are:

```text
Low Latency

High Availability

Real-Time Tracking

Scalability

Reliability
```

The system consists of multiple independent services communicating through Kafka in an Event-Driven Architecture.

---

## Restaurant Service

The Restaurant Service manages:

```text
Restaurant Information

Menus

Availability

Restaurant Status
```

Users browse restaurants through this service.

---

## Nearby Restaurant Search

To efficiently find nearby restaurants, I would use:

```text
GeoHash
```

Restaurant locations are converted into GeoHash cells.

When a user opens the application:

```text
User Location
↓
GeoHash
↓
Nearby Cells
↓
Nearby Restaurants
```

### Why GeoHash?

```text
Fast Location Queries

Efficient Radius Search

Avoids Full Database Scans
```

---

## Order Service

The Order Service is responsible for:

```text
Order Creation

Order State Management

Order History
```

The database stores:

```text
Order ID

User ID

Restaurant ID

Driver ID

Timestamp

Current Status
```

---

## Order Lifecycle

Every order passes through multiple states:

```text
ORDER_PLACED
↓
ORDER_ACCEPTED
↓
ORDER_PREPARING
↓
ORDER_READY
↓
ORDER_PICKED_UP
↓
ORDER_DELIVERED
```

The current state is stored in the Order Database.

---

## Kafka

Every state change generates a Kafka event.

Example:

```text
ORDER_PLACED
```

↓

```text
ORDER_ACCEPTED
```

↓

```text
ORDER_PICKED_UP
```

↓

```text
ORDER_DELIVERED
```

Kafka enables:

```text
Loose Coupling

Asynchronous Processing

Scalability

Event Driven Architecture
```

---

## Event Driven Architecture

A service can act as both:

```text
Consumer

Producer
```

Example:

```text
Restaurant Service

Consumes:
ORDER_PLACED

Produces:
ORDER_ACCEPTED
```

---

```text
Delivery Service

Consumes:
ORDER_ACCEPTED

Produces:
ORDER_PICKED_UP
```

---

```text
Notification Service

Consumes:
ORDER_PLACED
ORDER_ACCEPTED
ORDER_PICKED_UP
ORDER_DELIVERED
```

---

## Driver Matching Service

When food is ready:

```text
ORDER_READY
```

event is published.

The Driver Matching Service:

```text
Calculates GeoHash

Finds Nearby Riders

Filters Available Riders

Assigns Closest Rider
```

---

## Redis

Redis stores:

```text
Driver Location

Driver Availability

Live Tracking Data
```

Example:

```text
driver:101

latitude

longitude

status=AVAILABLE
```

### Why Redis?

```text
Fast Reads

Fast Writes

Real-Time Updates
```

---

## Live Tracking

Delivery partners continuously update:

```text
Latitude

Longitude
```

The Tracking Service stores the latest location in Redis.

Flow:

```text
Driver
↓
Tracking Service
↓
Redis
↓
Customer App
```

This enables real-time order tracking.

---

## Notification Service

Consumes Kafka events and sends:

```text
Order Confirmed

Order Preparing

Order Picked Up

Order Delivered
```

notifications.

---

## Scaling Strategy

### Kafka Scaling

Use:

```text
Kafka Cluster
```

with:

```text
Multiple Brokers

Multiple Partitions

Replication
```

Benefits:

```text
Fault Tolerance

High Throughput

Scalability
```

---

### Redis Scaling

Use:

```text
Redis Cluster
```

for:

```text
Location Storage

Tracking Data

Caching
```

---

### Database Scaling

Use:

```text
Read Replicas

Sharding
```

for handling large-scale traffic.

---

## Complete Order Flow

```text
User Places Order
↓
Order Service
↓
Order Database
↓
ORDER_PLACED Event
↓
Kafka

Restaurant Service Consumes Event
↓
ORDER_ACCEPTED

Restaurant Prepares Food
↓
ORDER_READY

Driver Matching Service
↓
GeoHash Search
↓
Assign Rider

Driver Picks Up Food
↓
ORDER_PICKED_UP

Tracking Service Updates Redis

Order Delivered
↓
ORDER_DELIVERED

Notification Service
↓
Customer Notified

Rating Service
↓
Review Collection

Analytics Service
↓
Reporting
```

---

# Key Design Decisions

```text
GeoHash
→ Nearby Search

Redis
→ Live Location Tracking

Kafka
→ Event Driven Architecture

Order Service
→ State Management

Notification Service
→ Asynchronous Notifications

Tracking Service
→ Real-Time Tracking

Kafka Cluster
→ Scalability And Reliability
```
