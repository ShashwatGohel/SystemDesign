# Ride Sharing System Architecture

## High Level Architecture

```mermaid
flowchart TB

    USER[Passenger]
    DRIVER[Driver]

    RS[Ride Service]
    DMS[Driver Matching Service]
    TS[Tracking Service]
    PS[Payment Service]
    NS[Notification Service]
    AS[Analytics Service]

    REDIS[(Redis)]
    DB[(Ride Database)]
    KAFKA[Kafka Cluster]

    USER --> RS
    DRIVER --> DMS

    RS --> DMS
    DMS --> REDIS

    RS --> DB
    RS --> KAFKA

    KAFKA --> NS
    KAFKA --> PS
    KAFKA --> AS
```

---

## Driver Matching Flow

```mermaid
flowchart TD

    USER[Ride Request]

    GEO[GeoHash Search]

    REDIS[(Redis)]

    TOPK[Top K Drivers]

    DRIVER[Driver Assigned]

    USER --> GEO
    GEO --> REDIS
    REDIS --> TOPK
    TOPK --> DRIVER
```

---

## Ride Lifecycle

```mermaid
flowchart LR

    REQUESTED[RIDE_REQUESTED]

    ASSIGNED[DRIVER_ASSIGNED]

    ARRIVING[DRIVER_ARRIVING]

    STARTED[RIDE_STARTED]

    COMPLETED[RIDE_COMPLETED]

    PAYMENT[PAYMENT_COMPLETED]

    REQUESTED --> ASSIGNED
    ASSIGNED --> ARRIVING
    ARRIVING --> STARTED
    STARTED --> COMPLETED
    COMPLETED --> PAYMENT
```

---

## Live Tracking Flow

```mermaid
flowchart LR

    DRIVER[Driver App]

    TRACK[Tracking Service]

    REDIS[(Redis)]

    WS[WebSocket]

    USER[Passenger App]

    DRIVER --> TRACK
    TRACK --> REDIS
    REDIS --> WS
    WS --> USER
```

---

## Kafka Event Flow

```mermaid
flowchart TB

    RIDE[Ride Service]

    KAFKA[Kafka]

    NOTIFY[Notification Service]

    PAYMENT[Payment Service]

    ANALYTICS[Analytics Service]

    RIDE --> KAFKA

    KAFKA --> NOTIFY
    KAFKA --> PAYMENT
    KAFKA --> ANALYTICS
```

---

## Surge Pricing Flow

```mermaid
flowchart TD

    DEMAND[Ride Requests]

    SUPPLY[Available Drivers]

    PRICING[Pricing Service]

    SURGE[Surge Multiplier]

    DEMAND --> PRICING
    SUPPLY --> PRICING

    PRICING --> SURGE
```

---

## Complete End-To-End Architecture

```mermaid
flowchart TB

    USER[Passenger]
    DRIVER[Driver]

    RS[Ride Service]
    DMS[Driver Matching Service]
    TS[Tracking Service]
    PS[Payment Service]
    NS[Notification Service]
    AS[Analytics Service]

    REDIS[(Redis)]
    DB[(Ride Database)]
    KAFKA[Kafka Cluster]

    USER --> RS

    RS --> DMS

    DMS --> REDIS

    DRIVER --> DMS

    RS --> DB

    RS --> KAFKA

    KAFKA --> NS
    KAFKA --> PS
    KAFKA --> AS

    DRIVER --> TS

    TS --> REDIS

    REDIS --> USER
```
