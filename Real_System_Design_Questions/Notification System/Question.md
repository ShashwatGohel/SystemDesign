# Practice Question 02 - Design a Notification System

## Interview Question

Design a scalable Notification System.

Requirements:

- Support Push Notifications
- Support Email Notifications
- Support SMS Notifications
- Users should be able to configure notification preferences
- System should support retries for failed notifications
- System should support millions of notifications per minute
- Notification failures should not affect other services

Design the high-level architecture.

Mention:

- Kafka
- Notification Service
- Preference Service
- Push Service
- Email Service
- SMS Service
- Redis
- SQL Database
- Dead Letter Queue
- Consumer Groups
- Horizontal Scaling

Also explain:

1. Notification Flow
2. Retry Flow
3. User Preference Flow
4. Scaling Strategy

---

# Interview Answer

For a Notification System, the primary goals are:

```text
High Availability

Scalability

Reliability

Fault Tolerance

Low Latency
```

The system should be capable of delivering millions of notifications while remaining independent from the core business services.

---

## Event Driven Architecture

I would use Event Driven Architecture with Kafka.

Instead of:

```text
Order Service
↓
Direct API Call
↓
Notification Service
```

I would use:

```text
Order Service
↓
Kafka
↓
Notification Service
```

### Why Kafka?

```text
Loose Coupling

Asynchronous Processing

Fault Tolerance

High Throughput

Easy Scalability
```

Business services should not wait for notifications to be delivered.

For example:

```text
Order Created

Payment Success

Order Shipped
```

should complete successfully even if the Notification System is temporarily unavailable.

---

## Notification Service

The Notification Service acts as the central coordinator.

Responsibilities:

```text
Consume Events

Check User Preferences

Generate Notification Jobs

Route Notifications
```

The Notification Service does not directly send Push, Email, or SMS notifications.

Instead, it creates notification jobs and publishes them for downstream workers.

### Why Separate Notification Service?

```text
Centralized Logic

Easy Maintenance

Supports Multiple Channels

Loose Coupling
```

---

## Preference Service

Users should be able to control which notifications they receive.

Examples:

```text
Marketing Emails

Order Updates

Payment Alerts

OTP Messages
```

Before sending any notification, the Notification Service checks user preferences.

### Why Preference Service?

```text
User Control

Prevents Spam

Improves User Experience

Supports Opt-In / Opt-Out
```

---

## SQL Database

I would use PostgreSQL or MySQL.

The database stores:

```text
User Preferences

Notification History

Notification Templates

Delivery Status
```

### Why SQL?

```text
Structured Data

Reliable Storage

Easy Querying

Reporting And Auditing
```

The data has a fixed schema and is frequently queried for reporting purposes.

---

## Redis Cache

Notification preference checks occur very frequently.

Instead of querying the database every time:

```text
Notification
↓
Database Lookup
```

I would cache preferences inside Redis.

Example:

```text
user:101

Push = True

Email = False

SMS = True
```

### Why Redis?

```text
Fast Reads

Reduced Database Load

Low Latency
```

---

## Push Service

Responsible for sending:

```text
Mobile Push Notifications
```

Examples:

```text
Firebase Cloud Messaging

Apple Push Notification Service
```

---

## Email Service

Responsible for sending:

```text
Emails
```

Examples:

```text
SendGrid

Amazon SES
```

---

## SMS Service

Responsible for sending:

```text
SMS Messages
```

Examples:

```text
Twilio

MSG91
```

---

## Fan-Out Strategy

Suppose a celebrity uploads a post.

Followers:

```text
50 Million
```

Creating all notifications inside a single request is not feasible.

Instead:

```text
POST_CREATED
↓
Kafka
↓
Notification Service
↓
Fan-Out Workers
```

The followers are divided into smaller batches and processed in parallel.

### Why Fan-Out?

```text
Parallel Processing

Massive Scale

Fast Notification Delivery
```

---

## Retry Mechanism

Notification delivery may fail because of:

```text
Network Issues

Provider Failures

Rate Limits
```

Instead of immediately discarding notifications, retries should be attempted.

Example:

```text
Attempt 1

Attempt 2

Attempt 3
```

using exponential backoff.

### Why Retry?

```text
Temporary Failures

Better Reliability

Higher Delivery Rate
```

---

## Dead Letter Queue (DLQ)

If a notification still fails after maximum retries:

```text
Retry Failed
↓
Dead Letter Queue
```

The notification is stored for later inspection and replay.

### Why DLQ?

```text
Prevents Message Loss

Supports Recovery

Does Not Block Processing
```

---

## Notification Flow

```text
Order Service
↓
Kafka
↓
Notification Service
↓
Check Preferences
↓
Create Notification Jobs
↓
Push / Email / SMS Topics
↓
Workers
↓
External Providers
↓
User Receives Notification
```

---

## User Preference Flow

```text
Notification Event
↓
Notification Service
↓
Redis Lookup

(Cache Hit)
↓
Use Preferences

(Cache Miss)
↓
SQL Database
↓
Update Redis
↓
Continue Processing
```

---

## Retry Flow

```text
Notification Worker
↓
Provider Failure
↓
Retry Queue
↓
Retry
↓
Retry
↓
Retry

Still Failed?
↓
Dead Letter Queue
```

---

## Scaling Strategy

To handle 50 million notifications during a sale event, I would increase the number of Kafka partitions and horizontally scale the notification workers. All workers would belong to the same consumer group so that Kafka can distribute partitions among them. This allows notifications to be processed in parallel across multiple instances. As traffic increases, we can add more partitions and more worker instances, enabling the system to scale horizontally without changing the application logic.

### Why Kafka Partitions?

```text
Parallel Processing

Higher Throughput

Better Resource Utilization
```

### Why Consumer Groups?

```text
Automatic Load Distribution

Horizontal Scaling

Fault Tolerance
```

### Why Multiple Workers?

```text
Faster Processing

Independent Scaling

Better Availability
```

---

# Key Design Decisions

```text
Kafka
→ Event Driven Architecture

Redis
→ Fast Preference Lookup

SQL Database
→ Preferences And History Storage

Fan-Out
→ Large Scale Notification Delivery

Retry Mechanism
→ Reliability

Dead Letter Queue
→ Failure Recovery

Kafka Partitions
→ Horizontal Scaling

Consumer Groups
→ Load Distribution
```
