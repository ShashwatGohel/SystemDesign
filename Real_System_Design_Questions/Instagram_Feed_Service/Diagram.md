# Instagram Feed System Architecture

## High Level Architecture

```mermaid
flowchart TB

    USER[User]

    POST[Post Service]

    STORAGE[(Object Storage)]

    DB[(Post Database)]

    KAFKA[Kafka]

    FEED[Feed Service]

    REDIS[(Redis Feed Cache)]

    CDN[CDN]

    USER --> POST

    POST --> STORAGE

    POST --> DB

    POST --> KAFKA

    KAFKA --> FEED

    FEED --> REDIS

    STORAGE --> CDN

    CDN --> USER
```

---

## Post Creation Flow

```mermaid
flowchart LR

    USER[User]

    POST[Post Service]

    STORAGE[(Object Storage)]

    DB[(Post Database)]

    KAFKA[POST_CREATED Event]

    USER --> POST

    POST --> STORAGE

    STORAGE --> POST

    POST --> DB

    POST --> KAFKA
```

---

## Fan-Out On Write

```mermaid
flowchart TB

    POST[New Post]

    FEED[Feed Service]

    F1[Follower Feed 1]

    F2[Follower Feed 2]

    F3[Follower Feed 3]

    F4[Follower Feed N]

    POST --> FEED

    FEED --> F1

    FEED --> F2

    FEED --> F3

    FEED --> F4
```

---

## Fan-Out On Read

```mermaid
flowchart TB

    POST[Celebrity Post]

    DB[(Post Database)]

    USER[Follower Opens Feed]

    FEED[Feed Service]

    POST --> DB

    USER --> FEED

    FEED --> DB

    DB --> FEED
```

---

## Feed Loading Flow

```mermaid
flowchart LR

    USER[User]

    FEED[Feed Service]

    REDIS[(Redis)]

    DB[(Post Database)]

    USER --> FEED

    FEED --> REDIS

    REDIS --> FEED

    FEED --> USER

    FEED --> DB

    DB --> FEED
```

---

## Celebrity Handling

```mermaid
flowchart TB

    USER1[Normal User]

    USER2[Celebrity]

    WRITE[Fan-Out On Write]

    READ[Fan-Out On Read]

    USER1 --> WRITE

    USER2 --> READ
```

---

## Complete Feed Architecture

```mermaid
flowchart TB

    USER[User]

    CDN[CDN]

    STORAGE[(Object Storage)]

    POST[Post Service]

    FEED[Feed Service]

    REDIS[(Redis Feed Cache)]

    DB[(Post Database)]

    KAFKA[Kafka]

    USER --> POST

    POST --> STORAGE

    POST --> DB

    POST --> KAFKA

    KAFKA --> FEED

    FEED --> REDIS

    FEED --> DB

    STORAGE --> CDN

    CDN --> USER
```
