# WhatsApp Chat System Architecture

## High Level Architecture

```mermaid
flowchart TB

    U1[User A]
    U2[User B]

    LB[Load Balancer]

    CS1[Chat Server 1]
    CS2[Chat Server 2]
    CS3[Chat Server N]

    KAFKA[Kafka]

    REDIS[(Redis Presence Cache)]

    DB[(Message Database)]

    U1 --> LB
    U2 --> LB

    LB --> CS1
    LB --> CS2
    LB --> CS3

    CS1 --> KAFKA
    CS2 --> KAFKA
    CS3 --> KAFKA

    CS1 --> DB
    CS2 --> DB
    CS3 --> DB

    CS1 --> REDIS
    CS2 --> REDIS
    CS3 --> REDIS
```

---

## Message Sending Flow

```mermaid
flowchart LR

    UA[User A]

    WS1[WebSocket]

    CHAT[Chat Service]

    DB[(Message Database)]

    KAFKA[Kafka]

    WS2[WebSocket]

    UB[User B]

    UA --> WS1

    WS1 --> CHAT

    CHAT --> DB

    CHAT --> KAFKA

    KAFKA --> CHAT

    CHAT --> WS2

    WS2 --> UB
```

---

## Offline Message Flow

```mermaid
flowchart TD

    UA[User A]

    CHAT[Chat Service]

    DB[(Message Database)]

    OFF[User B Offline]

    ONLINE[User B Reconnects]

    UA --> CHAT

    CHAT --> DB

    DB --> OFF

    OFF --> ONLINE

    ONLINE --> DB

    DB --> ONLINE
```

---

## Read Receipt Flow

```mermaid
flowchart LR

    SENT[Message Stored]

    DELIVERED[Delivered To Device]

    READ[Read By User]

    SENT --> DELIVERED

    DELIVERED --> READ
```

---

## Presence Service Flow

```mermaid
flowchart LR

    USER[User]

    WS[WebSocket Connection]

    PRESENCE[Presence Service]

    REDIS[(Redis)]

    USER --> WS

    WS --> PRESENCE

    PRESENCE --> REDIS
```

---

## Media Upload Flow

```mermaid
flowchart LR

    USER[User]

    CHAT[Chat Service]

    STORAGE[Object Storage]

    DB[(Message Database)]

    CDN[CDN]

    RECEIVER[Receiver]

    USER --> CHAT

    CHAT --> STORAGE

    STORAGE --> DB

    STORAGE --> CDN

    CDN --> RECEIVER
```

---

## Message Ordering Using Kafka

```mermaid
flowchart TB

    MSG1[Hello]

    MSG2[How Are You]

    MSG3[Lets Meet Tomorrow]

    PARTITION[Conversation Partition]

    MSG1 --> PARTITION
    MSG2 --> PARTITION
    MSG3 --> PARTITION
```

---

## Chat Service Scaling

```mermaid
flowchart TB

    USERS[100M Concurrent Users]

    LB[Load Balancer]

    CS1[Chat Server 1]

    CS2[Chat Server 2]

    CS3[Chat Server 3]

    CSN[Chat Server N]

    REDIS[(Redis)]

    KAFKA[Kafka]

    USERS --> LB

    LB --> CS1
    LB --> CS2
    LB --> CS3
    LB --> CSN

    CS1 --> REDIS
    CS2 --> REDIS
    CS3 --> REDIS
    CSN --> REDIS

    CS1 --> KAFKA
    CS2 --> KAFKA
    CS3 --> KAFKA
    CSN --> KAFKA
```

---

## Complete WhatsApp Architecture

```mermaid
flowchart TB

    USERA[User A]
    USERB[User B]

    LB[Load Balancer]

    CHAT1[Chat Server 1]
    CHAT2[Chat Server 2]
    CHATN[Chat Server N]

    REDIS[(Redis Presence Cache)]

    DB[(Message Database)]

    KAFKA[Kafka]

    STORAGE[(Object Storage)]

    CDN[CDN]

    USERA --> LB
    USERB --> LB

    LB --> CHAT1
    LB --> CHAT2
    LB --> CHATN

    CHAT1 --> REDIS
    CHAT2 --> REDIS
    CHATN --> REDIS

    CHAT1 --> DB
    CHAT2 --> DB
    CHATN --> DB

    CHAT1 --> KAFKA
    CHAT2 --> KAFKA
    CHATN --> KAFKA

    CHAT1 --> STORAGE
    CHAT2 --> STORAGE
    CHATN --> STORAGE

    STORAGE --> CDN

    CDN --> USERA
    CDN --> USERB
```
