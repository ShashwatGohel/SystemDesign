# Day-22 : Microservices Architecture

## Important Connection From Previous Topics

Previously we learned:

```text
Load Balancers

Caching

Databases

Replication

Sharding

Message Queues

Authentication

API Gateway
```

---

Question:

```text
What Happens When
A Single Backend Application
Becomes Huge?
```

---

Problems:

```text
Huge Codebase

Difficult Scaling

Risky Deployments

Multiple Teams Working Together
```

---

Solution:

```text
Microservices Architecture
```

---

# Monolithic Architecture

## What Is A Monolith?

A Monolith is a single application where all business logic is deployed together.

---

Example:

```text
Instagram Backend

Login

Posts

Likes

Comments

Messages

Notifications

Search
```

inside one application.

---

Architecture:

```text
Client
 ↓
Monolithic Backend
 ↓
Database
```

---

# Problems With Monolith

## Problem 1 : Huge Codebase

```text
500 Developers
↓
Same Codebase
```

Results:

```text
Merge Conflicts

Difficult Maintenance

Slower Development
```

---

## Problem 2 : Scaling Problem

Suppose:

```text
Messages Feature
```

gets:

```text
10x Traffic
```

---

Need:

```text
Scale Entire Application
```

---

Even though only Messages need scaling.

---

## Problem 3 : Deployment Problem

Small Change:

```text
Notification Module
```

---

Requires:

```text
Deploy Entire Application
```

---

Risk:

```text
Whole System Can Break
```

---

# Microservices Architecture

## What Are Microservices?

Instead of:

```text
One Huge Backend
```

---

Split into:

```text
Small Independent Services
```

---

Example:

```text
User Service

Post Service

Comment Service

Like Service

Notification Service

Search Service

Message Service
```

---

Architecture:

```text
Client
 ↓
API Gateway
 ↓
 ├── User Service
 ├── Post Service
 ├── Comment Service
 ├── Like Service
 ├── Notification Service
 ├── Search Service
 └── Message Service
```

---

# Characteristics Of Microservices

Each Service Has:

```text
Own Code

Own Deployment

Own Logic

Own Database
```

---

Benefits:

```text
Independent Development

Independent Deployment

Independent Scaling
```

---

# Selective Scaling

Suppose:

```text
Message Service
```

gets:

```text
10x Traffic
```

---

Microservices:

```text
Scale Only Message Service
```

---

Monolith:

```text
Scale Entire Application
```

---

# Service Communication

Interview Favorite.

---

Question:

```text
How Do Services Communicate?
```

---

Answer:

```text
1. Synchronous Communication

2. Asynchronous Communication
```

---

# Synchronous Communication

Services directly communicate and wait for a response.

---

Examples:

```text
HTTP

REST APIs

gRPC
```

---

Architecture:

```text
User Service
 ↓
HTTP/gRPC
 ↓
Post Service
```

---

Use Cases:

```text
Get User Profile

Fetch Post Details

Get Comments
```

---

When Immediate Response Is Needed.

---

# Asynchronous Communication

Services communicate through a Message Queue.

---

Examples:

```text
Kafka

RabbitMQ

Message Queue
```

---

Architecture:

```text
Post Service
 ↓
Message Queue
 ↓
Notification Service
```

---

Use Cases:

```text
Send Notifications

Generate Thumbnails

Analytics Processing

Feed Updates
```

---

Benefits:

```text
Loose Coupling

Better Scalability

User Doesn't Wait
```

---

# gRPC

Interview Favorite.

---

## What Is gRPC?

gRPC stands for:

```text
Google Remote Procedure Call
```

---

Think:

```text
A Faster Alternative To REST
For Service-To-Service Communication
```

---

# REST Example

Request:

```http
GET /user/123
```

---

Response:

```json
{
  "id": 123,
  "name": "Shashwat"
}
```

---

Uses:

```text
JSON
```

---

Benefits:

```text
Human Readable

Easy To Debug
```

---

Drawbacks:

```text
Larger Payload

Slower Parsing
```

---

# gRPC Example

Instead Of:

```text
JSON
```

gRPC Uses:

```text
Binary Data
```

---

Benefits:

```text
Smaller Payload

Less Bandwidth

Lower Latency

Higher Throughput
```

---

# REST vs gRPC

| REST                     | gRPC                             |
| ------------------------ | -------------------------------- |
| JSON                     | Binary                           |
| Human Readable           | Not Human Readable               |
| Slower                   | Faster                           |
| Easy To Debug            | Harder To Debug                  |
| Common For Frontend APIs | Common For Service Communication |

---

# Easy Memory Rule

```text
Frontend ↔ Backend
↓
REST API

Backend ↔ Backend
↓
gRPC
```

---

# Database Per Service Pattern

Interview Favorite.

---

Bad Design:

```text
All Services
↓
One Database
```

---

Problems:

```text
Tight Coupling

Harder Scaling

Shared Failures
```

---

Preferred:

```text
User Service
↓
User DB

Post Service
↓
Post DB

Notification Service
↓
Notification DB
```

---

Benefits:

```text
Loose Coupling

Independent Scaling

Fault Isolation
```

---

# Why Database Per Service?

Suppose:

```text
Notification Database Down
```

---

Question:

Should Login Stop Working?

```text
No
```

---

Reason:

```text
Service Isolation
```

---

# API Gateway

Without Gateway:

```text
Client
↓
User Service

Client
↓
Post Service

Client
↓
Search Service
```

---

Messy.

---

With Gateway:

```text
Client
↓
API Gateway
↓
Services
```

---

Responsibilities:

```text
Authentication

Rate Limiting

Routing

Logging

Request Validation
```

---

# Challenges Of Microservices

## Challenge 1

Distributed Transactions

Example:

```text
Order Service

Payment Service
```

---

Payment Success.

Order Failure.

---

Result:

```text
Inconsistent State
```

---

## Challenge 2

Debugging

Monolith:

```text
One Place To Debug
```

---

Microservices:

```text
20 Services
```

---

Need:

```text
Tracing
```

---

## Challenge 3

Network Calls

Monolith:

```text
Function Call
```

---

Microservices:

```text
Network Call
```

---

Problems:

```text
Slower

Can Fail

More Complex
```

---

# Monolith vs Microservices

| Monolith                  | Microservices              |
| ------------------------- | -------------------------- |
| Single Application        | Multiple Services          |
| Easier Initially          | More Complex               |
| Single Deployment         | Independent Deployment     |
| Hard To Scale Selectively | Easy To Scale Individually |
| Easier Debugging          | Harder Debugging           |
| Less Network Calls        | More Network Calls         |

---

# When To Use Monolith?

Use When:

```text
Small Startup

Small Team

Few Users
```

---

Example:

```text
5 Developers

500 Users
```

---

Benefits:

```text
Simple

Fast Development

Less Operational Complexity
```

---

# When To Use Microservices?

Use When:

```text
Millions Of Users

Multiple Teams

Different Scaling Requirements
```

---

Benefits:

```text
Independent Scaling

Independent Deployments

Better Team Ownership
```

---

# Common Interview Questions

## Q1. What Is A Monolith?

### Answer

A Monolith is a single application where all business functionality is developed, deployed, and scaled together.

---

## Q2. What Is A Microservice?

### Answer

A Microservice is an independently deployable service responsible for a specific business capability. Multiple microservices together form the complete system.

---

## Q3. Advantages Of Microservices?

### Answer

```text
Independent Scaling

Independent Deployment

Better Team Ownership

Fault Isolation
```

---

## Q4. Disadvantages Of Microservices?

### Answer

```text
Higher Complexity

Network Failures

Distributed Transactions

Harder Debugging
```

---

## Q5. Why Database Per Service?

### Answer

Database Per Service prevents tight coupling, improves fault isolation, and allows independent scaling of services.

---

## Q6. How Do Services Communicate?

### Answer

Services communicate either:

```text
Synchronously
↓
HTTP
REST
gRPC

Asynchronously
↓
Kafka
RabbitMQ
Message Queues
```

---

## Q7. Why Use API Gateway?

### Answer

API Gateway acts as a single entry point and handles routing, authentication, rate limiting, logging, and request validation.

---

## Q8. When Would You Choose Monolith Over Microservices?

### Answer

For small startups with small teams and low traffic, Monolith is preferred because it is simpler, faster to build, and easier to maintain.

---

# Interview Practice Scenarios

## Scenario 1

### Situation

Startup:

```text
5 Developers

500 Users
```

A senior engineer suggests:

```text
50 Microservices
```

### Interview Answer

I would not recommend 50 microservices at this stage because the operational complexity would be significantly higher than the business needs.

A monolithic architecture would be simpler, faster to develop, easier to deploy, and more suitable for a startup with 5 developers and 500 users.

---

## Scenario 2

### Situation

Instagram Messaging Service traffic grows:

```text
10x
```

while Login traffic remains unchanged.

### Interview Answer

Microservices allow selective scaling.

Only the Message Service needs to be scaled instead of scaling the entire application, reducing infrastructure cost and improving efficiency.

---

## Scenario 3

### Situation

Notification Service goes down.

### Question

Should Login stop working?

### Interview Answer

No.

Login should continue working because services are isolated from each other. A failure in Notification Service should not directly affect User Service.

---

## Scenario 4

### Situation

A developer says:

```text
All Services
Should Share
One Database
```

### Interview Answer

I would not agree.

A shared database introduces tight coupling between services, making independent scaling and fault isolation difficult.

The preferred approach is the Database Per Service Pattern.

---

## Scenario 5

### Situation

Design Instagram Architecture.

### Interview Answer

Services:

```text
User Service
Post Service
Comment Service
Like Service
Notification Service
Feed Service
Search Service
Message Service
Media Service
```

---

Communication:

```text
Synchronous
↓
HTTP / REST / gRPC

Asynchronous
↓
Kafka / RabbitMQ / Message Queue
```

Examples:

```text
Fetch User Profile
↓
Synchronous

Send Notification
↓
Asynchronous
```

---

API Gateway:

```text
Authentication

Rate Limiting

Routing

Logging

Request Validation
```

---

Databases:

```text
Database Per Service Pattern
```

Each service owns its own database to ensure loose coupling, fault isolation, and independent scaling.

---

# Key Takeaways

1. Monolith is a single application.
2. Microservices split functionality into independent services.
3. Microservices enable selective scaling.
4. Services communicate synchronously or asynchronously.
5. gRPC is commonly used for service-to-service communication.
6. Database Per Service improves isolation.
7. API Gateway acts as a single entry point.
8. Microservices introduce operational complexity.
9. Distributed transactions are harder than monoliths.
10. Use Monolith for small startups and Microservices for large-scale systems.

---

# 30 Second Revision

```text
Monolith
↓
Single Application

Microservices
↓
Multiple Independent Services

Service Communication
↓
HTTP
REST
gRPC

OR

Kafka
RabbitMQ

Database Per Service
↓
Loose Coupling

API Gateway
↓
Authentication
Rate Limiting
Routing

Selective Scaling
↓
Scale Only Required Service
```

---

# Interview Keywords

```text
Monolith

Microservices

Service Isolation

Independent Deployment

Independent Scaling

Database Per Service

API Gateway

HTTP

REST

gRPC

Kafka

RabbitMQ

Message Queue

Synchronous Communication

Asynchronous Communication

Distributed Transactions

Fault Isolation
```
