# Practice Question 01 - Design Instagram Post Upload System

## Interview Question

Design a scalable Instagram Post Upload System.

Requirements:

- 100 Million Users
- Users can upload images and videos
- Users can like posts
- Users can comment on posts
- Followers should receive notifications when a user uploads a post
- Feed should load quickly
- Users are distributed globally

Design the high-level architecture.

Mention:

- Load Balancer
- API Gateway
- Authentication
- Microservices
- Databases
- Redis Cache
- Object Storage
- CDN
- Message Queue
- Service Discovery

Also explain:

1. Upload Flow
2. Notification Flow
3. Feed Loading Flow

---

# Interview Answer

For a system with more than 100 million users, I would use a Microservices Architecture because different components have different scaling requirements and should be independently deployable.

---

## Services

I would divide the system into the following services:

```text
User Service
Post Service
Comment Service
Like Service
Notification Service
Feed Service
Media Service
```

Each service would be responsible for a single business capability and would own its own database.

This follows the:

```text
Database Per Service Pattern
```

which provides loose coupling and fault isolation.

---

## Authentication

I would use JWT-based authentication.

```text
Access Token
+
Refresh Token
```

The API Gateway validates JWT tokens before forwarding requests to backend services.

---

## Load Balancer

A Load Balancer would sit in front of the API Gateway cluster.

Responsibilities:

```text
Traffic Distribution

Health Checks

Failover

High Availability
```

This prevents any single API Gateway instance from becoming a bottleneck.

---

## API Gateway

The API Gateway acts as the single entry point for all client requests.

Responsibilities:

```text
Authentication

Routing

Rate Limiting

Logging

Request Validation
```

Examples:

```text
/upload
→ Media Service

/comment
→ Comment Service

/like
→ Like Service
```

---

## Object Storage

Images and videos should not be stored directly in databases.

Instead:

```text
Media Service
↓
Object Storage
```

Examples:

```text
AWS S3

Google Cloud Storage
```

The generated media URL is stored in the Post Database.

---

## CDN

Since users are globally distributed, media should be delivered through a CDN.

Benefits:

```text
Lower Latency

Faster Image Loading

Reduced Backend Traffic
```

The CDN caches media closer to users worldwide.

---

## Redis Cache

Instagram is heavily read-oriented.

Redis can cache:

```text
Popular Posts

Trending Posts

Feed Metadata

User Profiles
```

Benefits:

```text
Reduced Database Load

Lower Latency

Improved Performance
```

---

## Message Queue

Notifications should be processed asynchronously.

Flow:

```text
Post Created
↓
Message Queue
↓
Notification Service
```

Examples:

```text
Kafka

RabbitMQ
```

This prevents users from waiting for notifications to be sent.

---

## Service Discovery

Since services scale dynamically and instances may change IP addresses, services should discover each other through a Service Registry.

Benefits:

```text
No Hardcoded IPs

Automatic Discovery

Automatic Recovery
```

---

# Upload Flow

```text
User Uploads Image
↓
Load Balancer
↓
API Gateway
↓
Media Service
↓
Object Storage
↓
Media URL Generated
↓
Post Service Stores Metadata
↓
Post Database
↓
Message Queue
```

---

# Notification Flow

```text
Post Service
↓
Message Queue
↓
Notification Service
↓
Followers Receive Notification
```

---

# Feed Loading Flow

```text
User Opens Feed
↓
Feed Service
↓
Redis Cache

(Cache Hit)
↓
Return Feed

(Cache Miss)
↓
Database
↓
Redis Update
↓
Return Feed
```

Media URLs are returned to the client, and actual images/videos are served through the CDN.

---

# Key Design Decisions

```text
Microservices
→ Independent Scaling

Database Per Service
→ Loose Coupling

Object Storage
→ Large Media Storage

CDN
→ Global Content Delivery

Redis
→ Faster Reads

Message Queue
→ Asynchronous Processing

Service Discovery
→ Dynamic Service Communication
```
