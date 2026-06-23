# Food Delivery System Architecture

## High Level Architecture

```mermaid
flowchart TB

    USER[Customer]

    RS[Restaurant Service]

    OS[Order Service]

    DS[Delivery Service]

    NS[Notification Service]

    TS[Tracking Service]

    KAFKA[Kafka Cluster]

    REDIS[(Redis)]

    DB[(Order Database)]

    USER --> OS

    OS --> DB

    OS --> KAFKA

    KAFKA --> RS
    KAFKA --> DS
    KAFKA --> NS
    KAFKA --> TS

    TS --> REDIS
```

---

## Nearby Restaurant Search

```mermaid
flowchart LR

    USER[User Location]

    GH[GeoHash]

    CELL[Nearby GeoHash Cells]

    REST[Restaurants]

    USER --> GH

    GH --> CELL

    CELL --> REST
```

---

## Driver Matching Flow

```mermaid
flowchart TD

    READY[ORDER_READY]

    MATCH[Driver Matching Service]

    GEO[GeoHash Search]

    REDIS[(Redis)]

    DRIVER[Nearest Available Driver]

    READY --> MATCH

    MATCH --> GEO

    GEO --> REDIS

    REDIS --> DRIVER
```

---

## Order Lifecycle

```mermaid
flowchart LR

    PLACED[ORDER_PLACED]

    ACCEPTED[ORDER_ACCEPTED]

    PREPARING[ORDER_PREPARING]

    READY[ORDER_READY]

    PICKED[ORDER_PICKED_UP]

    DELIVERED[ORDER_DELIVERED]

    PLACED --> ACCEPTED

    ACCEPTED --> PREPARING

    PREPARING --> READY

    READY --> PICKED

    PICKED --> DELIVERED
```

---

## Event Driven Architecture

```mermaid
flowchart TB

    ORDER[Order Service]

    KAFKA[Kafka]

    REST[Restaurant Service]

    DELIVERY[Delivery Service]

    NOTIFY[Notification Service]

    ANALYTICS[Analytics Service]

    ORDER --> KAFKA

    KAFKA --> REST

    REST --> KAFKA

    KAFKA --> DELIVERY

    DELIVERY --> KAFKA

    KAFKA --> NOTIFY

    KAFKA --> ANALYTICS
```

---

## Live Tracking Flow

```mermaid
flowchart LR

    DRIVER[Driver App]

    TRACK[Tracking Service]

    REDIS[(Redis)]

    CUSTOMER[Customer App]

    DRIVER --> TRACK

    TRACK --> REDIS

    REDIS --> CUSTOMER
```

---

## Complete Food Delivery Architecture

```mermaid
flowchart TB

    USER[Customer]

    RESTAURANT[Restaurant]

    DRIVER[Delivery Partner]

    OS[Order Service]

    RS[Restaurant Service]

    DS[Delivery Service]

    TS[Tracking Service]

    NS[Notification Service]

    KAFKA[Kafka Cluster]

    REDIS[(Redis)]

    DB[(Order Database)]

    USER --> OS

    OS --> DB

    OS --> KAFKA

    KAFKA --> RS

    RS --> RESTAURANT

    RS --> KAFKA

    KAFKA --> DS

    DS --> DRIVER

    DRIVER --> TS

    TS --> REDIS

    REDIS --> USER

    KAFKA --> NS
```
