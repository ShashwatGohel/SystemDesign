# Practice Question 01 - Design a URL Shortener (Bit.ly / TinyURL)

## Interview Question

Design a scalable URL Shortener like Bit.ly or TinyURL.

Requirements:

- Users can submit a long URL
- System generates a unique short URL
- Users can access the original URL using the short URL
- System should support billions of URLs
- Redirects should be fast
- System should support click analytics

Design the high-level architecture.

Mention:

- Load Balancer
- URL Service
- Redis Cache
- Database
- Analytics System
- Kafka
- Sharding
- Replication

Also explain:

1. URL Creation Flow
2. URL Redirect Flow
3. Analytics Flow

---

# Interview Answer

For a URL Shortener, the primary goals are:

```text
Low Latency

High Availability

Scalability

Durability
```

The system is highly read-heavy because redirect requests are significantly larger than URL creation requests.

---

## URL Generation Strategy

When a user submits a long URL:

```text
youtube.com/watch?v=abc123
```

The URL is first stored in the database.

The database generates a unique auto-increment ID.

Example:

```text
1001
```

The ID is then converted into a Base62 string.

```text
1001
↓
g9
```

The final shortened URL becomes:

```text
bit.ly/g9
```

### Why Base62?

```text
Short URLs

Guaranteed Uniqueness

Simple Implementation

No Collision Handling
```

---

## Database Choice

The URL Shortener mainly performs:

```text
shortCode → originalURL
```

lookups.

This is essentially a key-value access pattern.

Therefore, at large scale I would choose:

```text
Cassandra

DynamoDB
```

because they support horizontal scaling and high throughput.

For smaller systems:

```text
PostgreSQL

MySQL
```

would also work.

---

## Redis Cache

Since redirects are read-heavy, Redis should be placed in front of the database.

Redis stores:

```text
shortCode → originalURL
```

Example:

```text
g9
↓
youtube.com/watch?v=abc123
```

### Why Redis?

```text
Reduces Database Load

Improves Response Time

Supports Millions Of Reads
```

---

## Load Balancer

A Load Balancer distributes incoming requests across URL Service instances.

Responsibilities:

```text
Traffic Distribution

Health Checks

Failover

High Availability
```

---

## URL Service

The URL Service handles:

```text
URL Creation

URL Lookup

Redirect Generation
```

Responsibilities:

```text
Generate Short Codes

Query Redis

Query Database

Return Redirect Response
```

---

## Analytics System

Users often want analytics such as:

```text
Click Count

Country

Browser

Device Type
```

Analytics should not be processed inside the redirect path.

Instead, analytics events should be processed asynchronously.

---

## Kafka

Whenever a user clicks a shortened URL:

```text
URL_CLICKED
```

event is published to Kafka.

Example:

```json
{
  "shortCode": "g9",
  "country": "India",
  "device": "Mobile",
  "browser": "Chrome"
}
```

Analytics Service consumes these events and updates analytics databases periodically.

### Why Kafka?

```text
Decouples Analytics

High Throughput

Event Replay

Asynchronous Processing
```

---

## URL Creation Flow

```text
User Submits Long URL
↓
Load Balancer
↓
URL Service
↓
Database Insert
↓
Generate Auto Increment ID
↓
Base62 Encoding
↓
Generate Short Code
↓
Store Mapping
↓
(Optional) Cache In Redis
↓
Return Tiny URL
```

---

## URL Redirect Flow

```text
User Opens Tiny URL
↓
Load Balancer
↓
URL Service
↓
Redis Lookup

(Cache Hit)
↓
Return Original URL
↓
301 Redirect

(Cache Miss)
↓
Database Lookup
↓
Update Redis
↓
301 Redirect
```

---

## Analytics Flow

```text
User Clicks Short URL
↓
Redirect User Immediately
↓
Publish URL_CLICKED Event
↓
Kafka
↓
Analytics Service
↓
Analytics Database
```

This keeps analytics outside the critical request path.

---

## Scaling Strategy

### Database Scaling

```text
Sharding
```

Shard Key:

```text
shortCode

or

URL ID
```

---

### Read Scaling

```text
Read Replicas
```

for handling redirect traffic.

---

### Cache Scaling

```text
Redis Cluster

Consistent Hashing
```

---

## Common Bottlenecks

### Redis Failure

```text
Fallback To Database

Higher Latency
```

---

### Database Failure

```text
New URLs Cannot Be Created

Cached Redirects Continue Working
```

---

### Viral URL

```text
Hot Key

Hot Shard
```

Solutions:

```text
Redis Replicas

Hot Key Replication

CDN Caching
```

---

# Key Design Decisions

```text
Base62 Encoding
→ Short Unique URLs

Redis
→ Fast Redirects

NoSQL
→ Horizontal Scaling

Kafka
→ Analytics Processing

Sharding
→ Database Scalability

Replication
→ High Availability
```
