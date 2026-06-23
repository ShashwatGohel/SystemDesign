# Practice Question 05 - Design a WhatsApp / Chat System

## Interview Question

Design a scalable WhatsApp-like Chat System.

Requirements:

- One-to-One Messaging
- Group Chats
- Online / Offline Status
- Message Delivery Status
- Read Receipts
- Image / Video Sharing
- 100 Million Users

Design the high-level architecture.

Mention:

- WebSockets
- Chat Service
- Kafka
- Redis
- Message Database
- Presence Service
- Object Storage
- CDN
- Load Balancer
- Horizontal Scaling

Also explain:

1. Message Sending Flow
2. Offline User Flow
3. Read Receipt Flow
4. Presence Service Flow
5. Scaling Strategy

---

# Interview Answer

For a Chat System, the primary goals are:

```text
Real-Time Communication

Low Latency

Scalability

High Availability

Reliability
```

Unlike traditional applications, chat systems require instant message delivery between users.

---

## WebSockets

I would use WebSockets instead of HTTP because chat applications require real-time bidirectional communication.

A WebSocket connection is established once using an HTTP handshake and then remains open.

### Why WebSockets?

```text
Persistent Connection

Real-Time Communication

Low Latency

Full Duplex Communication
```

This eliminates repeated polling requests and enables instant message delivery.

---

## Chat Service

The Chat Service is responsible for:

```text
Sending Messages

Receiving Messages

Message Routing

Group Messaging

Message Status Updates
```

The Chat Service acts as the central communication layer between users.

### Why Chat Service?

```text
Centralized Message Handling

Easy Scaling

Supports Multiple Chat Features
```

---

## Message Ordering

Message ordering is critical in chat systems.

Example:

```text
Hello

How are you?

Let's meet tomorrow
```

must arrive in the same order.

To preserve ordering:

```text
Conversation ID
↓
Kafka Partition Key
```

Messages belonging to the same conversation are routed to the same Kafka partition.

### Why Kafka?

```text
Message Ordering

Durability

Scalability

Fault Tolerance
```

---

## Message Database

Messages should always be persisted before delivery.

The database stores:

```text
Message ID

Conversation ID

Sender ID

Receiver ID

Message Content

Timestamp

Message Status
```

### Why Message Database?

```text
Durability

Chat History

Offline Support

Message Recovery
```

The database acts as the source of truth.

---

## Offline User Handling

If User B is offline:

```text
Store Message
↓
Message Database
↓
Wait For Reconnection
↓
Deliver Message
```

Messages should never be lost.

When the user reconnects, undelivered messages are fetched and delivered through the WebSocket connection.

---

## Read Receipts

I would maintain three message states.

### Sent

```text
User A
↓
Chat Service
↓
Database
```

Status:

```text
✓
```

---

### Delivered

Recipient device acknowledges receipt.

Status:

```text
✓✓
```

---

### Read

Recipient opens the message and sends a read receipt.

Status:

```text
✓✓ Blue
```

### Why Read Receipts?

```text
Better User Experience

Message Tracking

Delivery Visibility
```

---

## Presence Service

The Presence Service tracks:

```text
Online Status

Offline Status

Last Seen Timestamp
```

Presence information changes frequently and should not be stored in a database for every request.

---

## Redis

Presence information is stored in Redis.

Example:

```text
user:101

status = online

last_seen = null
```

or

```text
user:101

status = offline

last_seen = 10:45 PM
```

### Why Redis?

```text
Fast Reads

Fast Writes

Low Latency

Frequently Updated Data
```

---

## Media Sharing

Images, videos, audio files, and documents should not be stored directly inside the Message Database.

Instead:

```text
Media Upload
↓
Object Storage
↓
Generate URL
↓
Store Metadata In Database
```

The Message Database stores:

```text
File URL

File Type

Timestamp

Sender ID

Receiver ID
```

### Why Object Storage?

```text
Massive Scalability

Cheap Storage

Supports Large Files
```

Examples:

```text
AWS S3

Google Cloud Storage
```

---

## CDN

Media should be served through a CDN.

Flow:

```text
Object Storage
↓
CDN
↓
User
```

### Why CDN?

```text
Lower Latency

Faster Downloads

Reduced Backend Load
```

---

## Load Balancer

A Load Balancer sits in front of multiple Chat Servers.

Responsibilities:

```text
Traffic Distribution

Health Checks

Failover

High Availability
```

---

## Scaling Strategy

To support 100 million concurrent users, I would deploy multiple Chat Server instances behind a Load Balancer.

Since WebSocket connections are stateful, Sticky Sessions should be used so that a user remains connected to the same Chat Server throughout the session.

Presence information would be stored in Redis, while Kafka would be used for message routing and ordering.

### Why Sticky Sessions?

```text
Persistent WebSocket Connections

Session Consistency

Reliable Communication
```

### Why Kafka?

```text
Message Ordering

Cross Server Communication

Scalable Processing

Durability
```

### Chat Service Scaling Answer

To support 100 million concurrent users, I would horizontally scale the Chat Service using multiple WebSocket servers behind a Load Balancer. Sticky Sessions would ensure that users remain connected to the same Chat Server throughout their WebSocket session. Redis would store presence information, while Kafka would be used for message routing, persistence, and maintaining message ordering across distributed Chat Servers.

---

## Message Sending Flow

```text
User A
↓
WebSocket
↓
Chat Service
↓
Message Database
↓
Kafka
↓
Chat Service
↓
WebSocket
↓
User B
```

---

## Offline User Flow

```text
User A Sends Message
↓
Message Database
↓
User B Offline
↓
Store Message
↓
User B Reconnects
↓
Fetch Undelivered Messages
↓
Deliver Through WebSocket
```

---

## Read Receipt Flow

```text
Message Sent
↓
✓

Delivered To Device
↓
✓✓

User Opens Chat
↓
Read Receipt Event
↓
✓✓ Blue
```

---

## Presence Flow

```text
User Connects
↓
Presence Service
↓
Redis
↓
Mark Online

User Disconnects
↓
Presence Service
↓
Update Last Seen
↓
Mark Offline
```

---

# Key Design Decisions

```text
WebSockets
→ Real-Time Communication

Kafka
→ Message Ordering And Scalability

Redis
→ Presence Management

Message Database
→ Source Of Truth

Object Storage
→ Media Storage

CDN
→ Fast Media Delivery

Load Balancer
→ Horizontal Scaling

Sticky Sessions
→ Persistent Connections
```
