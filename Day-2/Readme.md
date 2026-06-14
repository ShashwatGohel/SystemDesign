# Day-2 : Functional Requirements vs Non-Functional Requirements

## Why Do We Gather Requirements?

Requirement gathering helps us understand what needs to be built before jumping into architecture design.

Without gathering requirements, we may end up designing unnecessary features and wasting interview time.

---

## What are Requirements?

Requirements define:

- What the system should do.
- How well the system should perform.

Requirements are divided into:

1. Functional Requirements (FR)
2. Non-Functional Requirements (NFR)

---

# Functional Requirements (FR)

## Definition

Functional Requirements describe what features the system should provide.

In simple words:

> What should the system do?

---

## Examples

### Instagram

- User can sign up
- User can login
- User can upload photos
- User can like posts
- User can comment on posts
- User can follow users

### WhatsApp

- Send messages
- Receive messages
- Create groups
- Show online status
- Deliver notifications

### URL Shortener

- Create short URL
- Redirect short URL
- Track click count

---

# Non-Functional Requirements (NFR)

## Definition

Non-Functional Requirements describe how well the system should perform.

In simple words:

> How should the system behave?

---

## Examples

- Scalability
- Availability
- Reliability
- Latency
- Throughput
- Security
- Data Durability

---

## Examples

### Instagram

- Support millions of users
- High availability
- Fast feed loading
- Reliable storage of photos
- Low latency

### WhatsApp

- Fast message delivery
- High availability
- Support millions of users
- Reliable message delivery
- Low latency

### URL Shortener

- Redirect within milliseconds
- High availability
- Handle millions of requests
- Reliable URL mapping

---

# Functional vs Non-Functional Requirements

| Functional Requirements | Non-Functional Requirements             |
| ----------------------- | --------------------------------------- |
| User can upload photo   | Upload should be fast                   |
| User can send message   | Message delivery should be low latency  |
| User can shorten URL    | Service should support millions of URLs |
| User can order food     | Service should be highly available      |

---

# Requirement Gathering Framework

Whenever the interviewer says:

> Design X

Follow this framework:

### Step 1: Clarify Functional Requirements

Ask:

- What features are required?
- What operations are supported?
- Who are the users?

---

### Step 2: Clarify Non-Functional Requirements

Ask:

- Expected scale?
- Availability requirements?
- Latency requirements?
- Reliability requirements?
- Security requirements?

---

### Step 3: Clarify Scope

Ask:

> What features are NOT required?

This prevents wasting time designing unnecessary functionality.

---

# In Scope vs Out of Scope

## In Scope

Features that must be designed.

### Example: Movie Ticket Booking

- Search Movies
- View Show Timings
- View Seat Availability
- Book Seats
- Make Payments

---

## Out of Scope

Features that are not required for the current design discussion.

### Example: Movie Ticket Booking

- Reviews
- Ratings
- Recommendation Engine
- Loyalty Programs
- Gift Cards
- Sports Events
- Live Concerts

---

# Why Are Non-Functional Requirements Important?

Non-Functional Requirements are important because they define how well the system should perform and directly influence architecture decisions related to scalability, availability, reliability, security, and performance.

### Example

#### Support 100 Users

```text
User
 |
Backend
 |
Database
```

Simple architecture is enough.

#### Support 100 Million Users

```text
User
 |
Load Balancer
 |
Multiple Servers
 |
Cache
 |
Database Replication
 |
CDN
```

The architecture changes completely because of Non-Functional Requirements.

---

# Real World Example

## Food Delivery Application

### Functional Requirements

- User Registration
- Search Restaurants
- Place Order
- Track Order
- Make Payment

### Non-Functional Requirements

- High Availability
- Scalability
- Fast Search
- Secure Payments
- Reliability

---

# Common Clarification Questions

## Functional Questions

- What features are required?
- Who are the users?
- What operations are supported?

---

## Scale Questions

- How many users?
- How many requests per second?
- How much data will be stored?

---

## Performance Questions

- Expected latency?
- Availability requirements?
- Reliability requirements?

---

## Scope Questions

- What features are out of scope?

---

# YouTube Example

## Functional Requirements

- Upload Videos
- Watch Videos
- Search Videos
- Subscribe to Channels
- Like and Comment on Videos

## Non-Functional Requirements

- Scalability
- High Availability
- Low Latency Video Playback
- Fast Search Response
- Reliability and Data Durability

---

# Online Movie Ticket Booking Example

## Functional Requirements

- Search Movies
- View Show Timings
- View Seat Availability
- Book Seats
- Make Payments
- Cancel Bookings
- Purchase Food and Beverages

## Non-Functional Requirements

- Scalability
- High Availability
- Fast Movie Search
- Secure Payments
- Reliability
- Data Durability
- Strong Consistency for Seat Booking

---

# Common Interview Mistakes

1. Not asking requirements.
2. Assuming features without clarification.
3. Ignoring Non-Functional Requirements.
4. Ignoring scale.
5. Starting architecture too early.
6. Not identifying out-of-scope features.

---

# Key Takeaways

1. Functional Requirements describe what the system should do.
2. Non-Functional Requirements describe how the system should behave.
3. Requirement gathering is the first step of every System Design interview.
4. Always identify in-scope and out-of-scope features.
5. Non-Functional Requirements drive architecture decisions.
6. Never start drawing architecture before clarifying requirements.
7. Requirement gathering saves time and prevents incorrect assumptions.

---

# Interview Questions

### Q1. What are Functional Requirements?

Functional Requirements describe the features and operations that a system should provide.

### Q2. What are Non-Functional Requirements?

Non-Functional Requirements describe how well the system should perform, including scalability, availability, reliability, latency, and security.

### Q3. Why are Non-Functional Requirements important?

They influence architecture decisions and determine whether a system can handle real-world scale and performance expectations.

### Q4. Why should we identify out-of-scope features?

To avoid designing unnecessary functionality and keep the discussion focused on the required system.

### Q5. What is the first step in a System Design interview?

Gathering the Requirement before starting to design the system.

## Revision Notes

### Must Remember

- Functional Requirements = What the system should do.
- Non-Functional Requirements = How well the system should perform.
- Always ask for requirements before designing.
- Clarify in-scope and out-of-scope features.
- NFRs drive architecture decisions.
