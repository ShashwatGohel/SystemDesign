# Practice Question 09 - Design a Ride Sharing System (Uber / Ola)

## Interview Question

Design a scalable Ride Sharing Platform like Uber or Ola.

### Functional Requirements

- Users Can Request Rides
- Drivers Can Accept Rides
- Find Nearest Driver
- Live Driver Tracking
- Ride Status Updates
- Payments
- Ride History

### Non Functional Requirements

- High Availability
- Low Latency
- High Scalability
- Fault Tolerance
- Real-Time Tracking
- Eventual Consistency
- Support 100 Million Users

---

## Design The High-Level Architecture

Mention:

- Ride Service
- Driver Matching Service
- GeoHash
- Redis
- WebSockets
- Notification Service
- Payment Service
- Kafka
- Surge Pricing

Also Explain:

1. Driver Matching Flow
2. Ride Lifecycle
3. Live Tracking
4. Surge Pricing
5. Scaling Strategy

---

# Interview Solution

## Core Services

- Ride Service
- Driver Matching Service
- Tracking Service
- Payment Service
- Notification Service
- Pricing Service
- Analytics Service

Each service follows:

```text
Database Per Service Pattern
```

---

## GeoHash

Finding the nearest driver is the most important problem in a ride-sharing system.

Scanning millions of drivers for every request is expensive.

Therefore, I would use:

```text
GeoHash
```

Flow:

```text
Passenger Location
↓
GeoHash
↓
Nearby GeoHash Cells
↓
Available Drivers
```

Only nearby drivers are searched.

### Why GeoHash?

```text
Fast Location Queries

Efficient Driver Search

Reduced Search Space

Horizontal Scalability
```

---

## Redis

Driver locations continuously change.

Example:

```text
Every 5 Seconds
```

Redis stores:

```text
Driver Location

GeoHash

Availability Status
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

Low Latency

Real-Time Updates
```

---

## Driver Matching Service

Flow:

```text
Ride Request
↓
GeoHash Search
↓
Nearby Available Drivers
↓
Top K Drivers
```

Example:

```text
Nearest 5 Drivers
```

The request is sent to all selected drivers.

The first driver who accepts gets the ride.

### Why Top-K Matching?

```text
Fast Assignment

Higher Acceptance Rate

Lower Wait Time
```

---

## Ride Lifecycle

```text
RIDE_REQUESTED
↓
DRIVER_ASSIGNED
↓
DRIVER_ARRIVING
↓
RIDE_STARTED
↓
RIDE_COMPLETED
↓
PAYMENT_COMPLETED
```

---

## WebSockets

After ride assignment:

```text
Passenger App
↔
WebSocket
↔
Backend
↔
WebSocket
↔
Driver App
```

### Why WebSockets?

```text
Real-Time Updates

Bidirectional Communication

No Repeated HTTP Requests

Low Network Overhead
```

---

## Live Tracking

```text
Driver App
↓
Tracking Service
↓
Redis
↓
WebSocket
↓
Passenger App
```

Passengers receive live location updates instantly.

---

## Kafka

Kafka handles asynchronous communication.

Events:

```text
RIDE_REQUESTED

DRIVER_ASSIGNED

RIDE_STARTED

RIDE_COMPLETED

PAYMENT_COMPLETED
```

Consumers:

```text
Notification Service

Analytics Service

Payment Service
```

### Why Kafka?

```text
Loose Coupling

Scalability

Asynchronous Processing

Event Driven Architecture
```

---

## Notification Service

Sends:

```text
Ride Accepted

Driver Arriving

Ride Started

Ride Completed

Payment Success
```

---

## Payment Service

Flow:

```text
RIDE_COMPLETED
↓
Payment Service
↓
Payment Processing
↓
PAYMENT_COMPLETED
```

Supports:

```text
UPI

Credit Card

Wallets

Cash
```

---

## Surge Pricing

The Pricing Service calculates:

```text
Active Ride Requests

Available Drivers
```

for every GeoHash region.

Example:

```text
Demand = 1000 Riders

Supply = 100 Drivers

Surge Multiplier = 2x
```

### Why Surge Pricing?

```text
Balances Supply And Demand

Increases Driver Availability

Improves Ride Fulfillment
```

---

## Complete Ride Flow

```text
Passenger Requests Ride
↓
Ride Service
↓
GeoHash Search
↓
Driver Matching Service
↓
Top K Drivers
↓
Driver Accepts
↓
DRIVER_ASSIGNED Event
↓
Kafka
↓
Notification Service
↓
Tracking Service
↓
Redis + WebSockets
↓
Live Tracking
↓
RIDE_STARTED
↓
RIDE_COMPLETED
↓
Payment Service
↓
PAYMENT_COMPLETED
↓
Analytics Service
```

---

## Scaling Strategy

### Driver Search Scaling

```text
GeoHash

Redis

Horizontal Scaling
```

### Event Scaling

```text
Kafka Partitions

Consumer Groups

Multiple Consumers
```

### Tracking Scaling

```text
WebSocket Servers

Redis

Load Balancers
```

---

# Key Design Decisions

```text
GeoHash
→ Nearby Driver Search

Redis
→ Real-Time Driver Locations

Top-K Matching
→ Faster Driver Assignment

WebSockets
→ Live Tracking

Kafka
→ Event Driven Architecture

Payment Service
→ Fare Collection

Surge Pricing
→ Demand Supply Balancing

Notification Service
→ Real-Time Ride Updates
```
