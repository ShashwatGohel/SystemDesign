# Practice Question 06 - Design Instagram Feed System

## Interview Question

Design a scalable Instagram Feed System.

Requirements:

- Users Can Create Posts
- Users Can Follow Other Users
- Users Can Like Posts
- Users Can Comment On Posts
- Users Can View Home Feed
- Feed Should Load Quickly
- Support 100 Million Users
- Celebrity Accounts Can Have Millions Of Followers

Design the high-level architecture.

Mention:

- Feed Service
- Post Service
- Redis Cache
- Kafka
- Feed Generation
- Fan-Out On Write
- Fan-Out On Read
- Object Storage
- CDN
- Database
- Celebrity Handling

Also explain:

1. Post Creation Flow
2. Feed Loading Flow
3. Fan-Out On Write
4. Fan-Out On Read
5. Celebrity Handling

---

# Interview Answer

For an Instagram Feed System, the primary goals are:

```text
Low Latency

High Availability

Scalability

High Read Throughput

Fast Feed Loading
```

Instagram is primarily a read-heavy system because users read feeds much more frequently than they create posts.

---

## Post Service

The Post Service is responsible for:

```text
Creating Posts

Managing Post Metadata

Publishing Events
```

When a user uploads a post, the media is stored separately and only metadata is stored in the database.

---

## Object Storage

Images and videos should not be stored directly inside databases.

Instead:

```text
User Uploads Media
↓
Object Storage
↓
Generate URL
↓
Store URL In Database
```

Examples:

```text
AWS S3

Google Cloud Storage
```

### Why Object Storage?

```text
Cheap Storage

Massive Scalability

Supports Large Media Files
```

---

## CDN

Media files should be served through a CDN.

Flow:

```text
Object Storage
↓
CDN
↓
Users
```

### Why CDN?

```text
Lower Latency

Faster Image Loading

Reduced Backend Traffic
```

---

## Post Database

Stores:

```text
Post ID

User ID

Caption

Media URL

Timestamp
```

This acts as the source of truth.

---

## Kafka

After a post is created:

```text
POST_CREATED
```

event is published to Kafka.

Kafka enables:

```text
Loose Coupling

Asynchronous Processing

Scalability

Event Driven Architecture
```

Multiple services can independently consume the event.

Examples:

```text
Feed Service

Notification Service

Analytics Service
```

---

## Feed Service

The Feed Service is responsible for:

```text
Feed Generation

Feed Updates

Timeline Management
```

This is the core service of Instagram.

---

## Feed Cache (Redis)

Feed generation is expensive.

Generating feeds repeatedly requires:

```text
Fetch Posts

Merge Posts

Sort Posts

Rank Posts
```

Instead, feeds should be cached inside Redis.

Example:

```text
feed:101

Post 5001

Post 4999

Post 4998
```

### Why Redis?

```text
Fast Reads

Low Latency

Reduced Database Load
```

---

## Fan-Out On Write

Used for normal users.

Example:

```text
User Has 500 Followers
```

When a post is created:

```text
Create Post
↓
Update Feed Cache
Of All Followers
```

### Why Fan-Out On Write?

```text
Cheap Writes

Very Fast Feed Reads

Good For Normal Users
```

Feed is already precomputed when users open Instagram.

---

## Fan-Out On Read

Used for celebrity accounts.

Example:

```text
100 Million Followers
```

Updating every follower's feed cache would be extremely expensive.

Instead:

```text
Store Post Once
↓
Followers Open Feed
↓
Generate Feed Dynamically
```

### Why Fan-Out On Read?

```text
Avoids Massive Cache Updates

Handles Celebrity Accounts Efficiently

Reduces Write Amplification
```

---

## Celebrity Problem

Suppose:

```text
Celebrity
↓
100 Million Followers
```

Using Fan-Out On Write would require:

```text
100 Million Feed Updates
```

for a single post.

This is not practical.

Therefore:

```text
Normal Users
↓
Fan-Out On Write

Celebrities
↓
Fan-Out On Read
```

This approach is called:

```text
Hybrid Feed Architecture
```

---

## Feed Loading Flow

When a user opens Instagram:

```text
User Opens Feed
↓
Feed Service
↓
Redis

Cache Hit
↓
Return Feed

Cache Miss
↓
Generate Feed
↓
Database
↓
Update Redis
↓
Return Feed
```

---

## Cache Failure Handling

Redis is only a cache.

If Redis fails:

```text
Feed Service
↓
Database
↓
Generate Feed
↓
Return Feed
```

The system becomes slower but remains available.

### Interview Gold Statement

```text
Cache Failures Should Degrade Performance,
Not Availability
```

---

## Post Creation Flow

```text
User Creates Post
↓
Object Storage
↓
Generate Media URL
↓
Post Service
↓
Store Metadata In Database
↓
POST_CREATED Event
↓
Kafka
```

---

## Feed Update Flow

### Normal User

```text
POST_CREATED
↓
Kafka
↓
Feed Service
↓
Fan-Out On Write
↓
Update Followers Feed Cache
↓
Redis
```

---

### Celebrity User

```text
POST_CREATED
↓
Kafka
↓
Feed Service
↓
Store Post Only
↓
Fan-Out On Read
```

Feed generated when followers open Instagram.

---

# Key Design Decisions

```text
Object Storage
→ Media Storage

CDN
→ Fast Content Delivery

Kafka
→ Event Driven Architecture

Redis
→ Feed Cache

Fan-Out On Write
→ Normal Users

Fan-Out On Read
→ Celebrities

Hybrid Feed Architecture
→ Best Scalability
```
