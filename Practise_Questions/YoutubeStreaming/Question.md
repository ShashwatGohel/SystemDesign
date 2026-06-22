# Practice Question 02 - Design YouTube Video Upload & Streaming System

## Interview Question

Design a scalable YouTube Video Upload and Streaming System.

Requirements:

- 500 Million Users
- Users can upload videos
- Users can watch videos
- Users can like and comment on videos
- Users can subscribe to channels
- Subscribers should receive notifications for new uploads
- Videos should stream smoothly worldwide
- Recommended videos should load quickly

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

1. Video Upload Flow
2. Video Processing Flow
3. Video Streaming Flow
4. Notification Flow

---

# Interview Answer

For a platform serving hundreds of millions of users globally, I would use a Microservices Architecture because different services have different scaling requirements and should be independently deployable.

---

## Services

I would divide the system into:

```text
User Service

Video Service

Comment Service

Like Service

Subscription Service

Notification Service

Recommendation Service

Media Processing Service
```

Each service owns its own database.

This follows:

```text
Database Per Service Pattern
```

which improves scalability and fault isolation.

---

## Authentication

I would use JWT Authentication.

```text
Access Token

Refresh Token
```

The API Gateway validates tokens before forwarding requests.

---

## Load Balancer

A Load Balancer sits in front of API Gateway instances.

Responsibilities:

```text
Traffic Distribution

Failover

Health Checks

High Availability
```

---

## API Gateway

Acts as the single entry point.

Responsibilities:

```text
Authentication

Routing

Rate Limiting

Logging

Request Validation
```

---

## Object Storage

Videos should not be stored inside databases.

Instead:

```text
Video Service
↓
Object Storage
```

Examples:

```text
AWS S3

Google Cloud Storage
```

The database stores only metadata and video URLs.

---

## CDN

Videos should be served through a CDN.

Benefits:

```text
Reduced Latency

Global Delivery

Reduced Backend Load
```

Users stream videos from the nearest CDN edge location.

---

## Redis Cache

Redis caches:

```text
Trending Videos

Video Metadata

Channel Information

Recommendations
```

Benefits:

```text
Faster Reads

Lower Latency

Reduced Database Load
```

---

## Message Queue

Video processing is asynchronous.

Examples:

```text
Kafka

RabbitMQ
```

---

## Service Discovery

Services dynamically register themselves in a Service Registry.

Benefits:

```text
No Hardcoded IPs

Automatic Discovery

Easy Scaling
```

---

# Video Upload Flow

```text
Creator Uploads Video
↓
Load Balancer
↓
API Gateway
↓
Video Service
↓
Object Storage
↓
Store Video Metadata
↓
Video Database
↓
Publish Event To Message Queue
```

---

# Video Processing Flow

```text
Video Uploaded Event
↓
Message Queue
↓
Media Processing Service
↓
Generate Multiple Resolutions
↓
Store Processed Videos
↓
Object Storage
```

Example:

```text
1080p

720p

480p

360p
```

---

# Video Streaming Flow

```text
User Opens Video
↓
Video Service
↓
Redis Cache

(Cache Hit)
↓
Return Video Metadata

(Cache Miss)
↓
Database
↓
Redis Update
↓
Return Metadata
↓
CDN Streams Video
```

---

# Notification Flow

```text
Creator Uploads Video
↓
Message Queue
↓
Notification Service
↓
Subscribers Receive Notification
```

---

# Key Design Decisions

```text
Microservices
→ Independent Scaling

Object Storage
→ Video Storage

CDN
→ Global Streaming

Redis
→ Faster Reads

Message Queue
→ Asynchronous Processing

Service Discovery
→ Dynamic Communication
```
