# System Design Interview Revision Questions - Part 3

---

# 31. Inventory Reservation

1. What is Inventory Reservation?
   **Answer:** Inventory reservation is a design pattern where stock for an item is temporarily blocked or locked for a user during the checkout flow (before payment completes), guaranteeing that the item is held for them while they pay.

2. Why is Inventory Reservation needed?
   **Answer:** It is critical to prevent overselling (selling more items than are physically in the warehouse) and to avoid the poor user experience of a customer completing a payment only to find out the item is out of stock.

3. What is the Overselling Problem?
   **Answer:** A race condition where multiple concurrent purchase requests check the available stock of an item (e.g., stock = 1), find it positive, and all proceed to charge the users, resulting in more sales than physical inventory.

4. Why not reserve inventory when a user adds to cart?
   **Answer:** Because cart abandonment rates are high (typically around 70%). Reserving stock on cart addition would lead to "artificial stockouts," where items are unavailable to actual buyers because they are sitting in abandoned carts.

5. When should inventory be reserved?
   **Answer:** When the user initiates the checkout transaction (clicks "Place Order" or "Proceed to Payment"), creating a temporary lease/hold (e.g., 10-15 minutes) on the inventory.

6. What happens if payment fails after reservation?
   **Answer:** The Saga orchestrator or database scheduler detects the payment failure (or reservation timeout) and triggers a compensating transaction to release the held inventory back to the available pool.

7. How is inventory released?
   **Answer:** By updating the inventory table to decrement the `reserved_quantity` and increment the `available_quantity` for that item, or by deleting the corresponding temporary lock/lease record.

8. How does Inventory Reservation improve consistency?
   **Answer:** By dividing stock into three distinct states: `available_stock`, `reserved_stock`, and `sold_stock`. This allows the system to run atomic checks and updates (using database locks or transactions) at checkout time.

9. Inventory Reservation vs Direct Stock Deduction?
   **Answer:**
   * **Inventory Reservation** places a temporary lease on stock during checkout; if payment fails, the lease is easily reverted.
   * **Direct Stock Deduction** immediately subtracts stock; if payment fails, the system must issue refunds and manual adjustments, which is highly error-prone.

10. Which systems commonly use Inventory Reservation?
    **Answer:** E-commerce platforms (Amazon), flash sale portals, concert ticket booking sites (Ticketmaster), and airline reservation systems.

---

# 32. Distributed Transactions

1. What is a Distributed Transaction?
   **Answer:** A transaction that involves updates across multiple physical databases, network nodes, or microservices, while trying to maintain the classic ACID guarantees globally.

2. Why are Distributed Transactions difficult?
   **Answer:** Due to network latency, the possibility of independent node failures, network partitions, and the high overhead of coordinating state agreements across a distributed system (CAP Theorem constraints).

3. Why can't ACID transactions scale across microservices?
   **Answer:** Microservices use the "Database per Service" pattern. Executing global ACID transactions across multiple distinct databases requires distributed locking mechanisms (like 2PC) that block resources, increase latency exponentially, and risk deadlocks.

4. What is Eventual Consistency?
   **Answer:** A consistency model where database replicas or dependent services receive updates asynchronously. The system does not guarantee that all nodes see the same data immediately, but guarantees they will eventually reconcile to the same state.

5. What is the Saga Pattern?
   **Answer:** An eventual consistency pattern that manages distributed transactions as a sequence of local transactions. Each microservice executes its own local transaction and publishes an event; if a step fails, the Saga runs compensating transactions in reverse order.

6. What is a Compensating Transaction?
   **Answer:** A local transaction executed to undo the side effects of a previously completed step in a Saga when a subsequent step in the workflow fails.

7. Distributed Transaction vs Local Transaction?
   **Answer:**
   * **Local Transaction:** Managed inside a single database engine, supports native ACID, has high performance, and minimal complexity.
   * **Distributed Transaction:** Spans multiple network-connected databases, requires complex coordination protocols, degrades throughput, and is highly prone to network-related failures.

8. What is Two Phase Commit?
   **Answer:** A consensus protocol (2PC) that ensures atomic distributed commits in two steps:
   * **Prepare Phase:** The coordinator node asks all participant databases if they are ready to commit and locks resources.
   * **Commit Phase:** If all participants agree, the coordinator commands them to write the changes permanently.

9. Why is Two Phase Commit rarely used at scale?
   **Answer:** It is a synchronous, blocking protocol. If the coordinator or network fails during the commit phase, participant nodes remain locked, rendering parts of the system unavailable. It degrades system throughput.

10. How do modern microservices handle Distributed Transactions?
    **Answer:** By designing for eventual consistency using the Saga Pattern (either Orchestration or Choreography) and utilizing the Transactional Outbox pattern to ensure reliable message delivery.

---

# 33. Dead Letter Queue (DLQ)

1. What is a Dead Letter Queue?
   **Answer:** A DLQ is a specialized queue within a messaging system (like RabbitMQ or SQS) where messages that cannot be processed successfully by consumers are sent for isolation and debugging.

2. Why do we need a Dead Letter Queue?
   **Answer:** To prevent "poison pills"—corrupted or unprocessable messages that cause consumer crashes—from blocking the main queue and halting the entire message processing pipeline.

3. When does a message enter the DLQ?
   **Answer:**
   * When it fails processing repeatedly and exceeds the maximum retry limit.
   * When the message payload fails parsing or validation.
   * When the message's Time-To-Live (TTL) expires.

4. How do retries work before DLQ?
   **Answer:** When a consumer encounters an error, it returns a negative acknowledgment (NACK). The message broker puts the message back into the queue, typically waiting for a configured backoff delay before offering it to consumers again.

5. What happens to messages inside the DLQ?
   **Answer:** They remain stored in the DLQ. Engineers monitor the DLQ, debug the root cause of the failures, patch the consumer code, and then replay the messages back into the primary queue for processing.

6. How does DLQ improve reliability?
   **Answer:** It isolates failing messages without throwing them away, ensuring that healthy messages continue to flow smoothly while preserving faulty messages for forensic analysis.

7. What are common use cases of DLQ?
   **Answer:** Stripe webhook processing, e-commerce order ingestion pipelines, SMS/Email delivery notifications, and financial ledger postings.

8. How do you monitor DLQs?
   **Answer:** By setting up alerts (using Prometheus/Grafana or AWS CloudWatch) on the queue depth metric (number of messages in the DLQ). Any message in the DLQ should trigger an alert for investigation.

9. Kafka DLQ vs RabbitMQ DLQ?
   **Answer:**
   * **RabbitMQ** has native support where the broker automatically routes rejected or expired messages to a configured Dead Letter Exchange (DLX).
   * **Kafka** has no native DLQ concept; it must be implemented in application code by catching consumer exceptions and writing the bad event to a dedicated "dead-letter" topic.

10. What problems can occur if DLQs are ignored?
    **Answer:** Silent data loss, unresolved application bugs, and broker disk space exhaustion due to the unchecked accumulation of dead messages.

---

# 34. Notification Preferences

1. What are Notification Preferences?
   **Answer:** User configuration settings that dictate which types of notifications (e.g., marketing, security, transactional) they want to receive and through which channels (Email, SMS, Push notifications).

2. Why are Notification Preferences important?
   **Answer:** To respect user choices, prevent spam complaints, comply with regulatory requirements (GDPR, CAN-SPAM), reduce API costs (SMS), and prevent users from blocking app notifications entirely.

3. How would you store user preferences?
   **Answer:** In a relational database table mapping `user_id` to various categories and channels, and caching this configuration in Redis as a JSON string or Bitmask for high-performance runtime lookups.

4. Why is Redis useful for Notification Preferences?
   **Answer:** Before dispatching any notification, the system must check the user's preferences. Querying a relational database for every single notification at high scale creates a major database bottleneck. Redis provides sub-millisecond, high-throughput lookups.

5. How do you support Email, SMS, and Push preferences?
   **Answer:** By defining structured user preference records (e.g., `{"security": {"email": true, "sms": true, "push": true}, "marketing": {"email": false, "sms": false, "push": true}}`) that the routing service evaluates before sending.

6. How do you disable marketing notifications?
   **Answer:** By setting the active preference flag for the `marketing` category to `false` in the database and updating the Redis cache.

7. How do you allow critical notifications?
   **Answer:** By bypassing preference checks entirely for critical or transactional alerts (like OTPs or fraud alerts) by classifying them under a protected `security` or `system` category.

8. How do Notification Preferences affect scalability?
   **Answer:** If not cached properly, preference checks will overwhelm the database during notification spikes. Implementing a cache-aside pattern with Redis is essential to scale the preference-checking layer.

9. How would you design preference management APIs?
   * `GET /users/{id}/preferences` to retrieve settings.
   * `PUT /users/{id}/preferences` to update preferences, which updates the SQL database and invalidates/updates the Redis cache.

10. What is the difference between system notifications and marketing notifications?
    **Answer:**
    * **System/Transactional:** Essential notifications required for the core app functionality (OTPs, password resets). These cannot be disabled by the user.
    * **Marketing:** Promotional alerts (discounts, recommendation alerts) that are optional and must legally support opting out.

---

# 35. Surge Pricing

1. What is Surge Pricing?
   **Answer:** A dynamic pricing strategy where prices increase in real-time when customer demand outstrips the supply of available service providers (e.g., ride-hailing drivers during rush hour or rain).

2. Why is Surge Pricing needed?
   **Answer:** To balance the marketplace: it encourages more drivers to come online or travel to high-demand areas (increasing supply) and tempers demand from price-sensitive riders.

3. How is Surge Pricing calculated?
   **Answer:** By measuring the ratio of active ride requests (demand) to unassigned, active drivers (supply) within a specific geographic area over a moving time window.

4. Why is supply-demand ratio important?
   **Answer:** It is the primary metric that determines the surge multiplier. A high ratio (many riders, few drivers) triggers a higher multiplier, whereas a balanced ratio keeps the price at baseline.

5. How does GeoHash help Surge Pricing?
   **Answer:** It divides the physical map into a grid of discrete cells. The system can aggregate supply and demand numbers inside each individual Geohash cell to compute localized surge values rather than applying a single rate to a whole city.

6. How do ride-sharing apps calculate surge by area?
   **Answer:** By running stream processing jobs (e.g., Apache Flink) that aggregate GPS locations of drivers and rider requests in real-time, grouped by Geohash cells (usually resolution 6), and outputting a surge multiplier for each cell.

7. What are the advantages of Surge Pricing?
   **Answer:**
   * Incentivizes drivers to log on during peak hours, increasing supply.
   * Maximizes platform throughput and matching efficiency.
   * Allocates limited rides to users who need them most.

8. What are the disadvantages of Surge Pricing?
   **Answer:** It causes customer dissatisfaction due to high price volatility and can lead to brand perception issues if perceived as unfair.

9. How does Surge Pricing increase driver availability?
   **Answer:** The driver app displays a heat map showing high-surge zones with fare multipliers. Drivers migrate to these highlighted areas to maximize their earnings.

10. How do Uber and Ola implement Surge Pricing?
    **Answer:** Drivers stream GPS locations via WebSockets to a location service. This location data is ingested into Kafka and processed by a stream engine (like Flink) to calculate demand and supply per Geohash, updating a Redis cache queried by the pricing service.

---

# 36. Top-K Matching

1. What is Top-K Matching?
   **Answer:** An algorithm used to select the "K" best resources (e.g., the 5 closest and highest-rated drivers) from a larger pool to fulfill a specific request.

2. Why is Top-K Matching used in ride-sharing systems?
   **Answer:** To find the most optimal drivers for a trip request based on ETA, route constraints, and driver status, ensuring a high likelihood of acceptance and a short pickup time.

3. Why not notify all drivers?
   **Answer:** It would lead to massive network overhead, driver distraction, duplicate assignment conflicts (race conditions), and a frustrating user experience when drivers repeatedly lose matches to faster clickers.

4. Why not notify only one driver?
   **Answer:** If the single selected driver declines or ignores the request, the passenger experiences a significant delay. Batching notifications to the Top-K drivers optimizes acceptance speed.

5. How is K selected?
   **Answer:** K is determined based on historical acceptance rates, driver density, and current traffic conditions. Typically, $K$ ranges between 3 and 5.

6. What are the benefits of Top-K Matching?
   **Answer:** It minimizes rider wait times, optimizes driver matching quality, prevents driver notification fatigue, and simplifies concurrency control.

7. How does Top-K Matching reduce rider wait time?
   **Answer:** By simultaneously or sequentially offering the ride to a small pool of the closest available drivers, maximizing the chance of an immediate acceptance.

8. What happens if none of the Top-K drivers accept?
   **Answer:** The system expands the search radius to neighboring Geohash cells, recalculates the next Top-K drivers, and sends a new round of notifications.

9. How does Top-K Matching scale?
   **Answer:** By storing driver locations in an in-memory spatial index (like Redis Geospatial indexes or R-Trees), enabling $O(\log N)$ nearest-neighbor lookups instead of calculating distances for all drivers ($O(N)$).

10. Which real-world systems use Top-K Matching?
    **Answer:** Uber, Lyft, DoorDash, and dispatch dispatchers in logistics management systems.

---

# 37. URL Shortener

1. How do you generate unique short URLs?
   **Answer:** By generating a unique 64-bit ID using a distributed generator (like Twitter Snowflake) or a Token Range Generator, and then encoding that integer ID into a Base62 string.

2. Why use Base62 Encoding?
   **Answer:** Base62 uses characters `[0-9, a-z, A-Z]`, which are URL-safe. A 7-character Base62 string provides $62^7 \approx 3.5$ trillion unique keys, making the shortened URL extremely compact.

3. Why is a URL Shortener read-heavy?
   **Answer:** A short link is generated once, but can be clicked millions of times by users globally. The read-to-write ratio is typically 100:1 or higher.

4. How does Redis help URL Shorteners?
   **Answer:** Redis caches the mapping of `short_key -> long_url` in memory. Redirection requests can be served directly from Redis in under 10ms, bypassing the primary database.

5. What database would you choose and why?
   **Answer:** A NoSQL database like DynamoDB or MongoDB because the data structure is a simple key-value lookup (`short_key` to `long_url`) and does not require complex relational joins.

6. How do you handle Hot URLs?
   **Answer:** By implementing multi-level caching: placing a CDN (like Cloudflare) in front of the redirect service to cache redirects at the edge, and using Redis with an LRU eviction policy.

7. How do you collect click analytics?
   **Answer:** When a user accesses the short link, the redirect service sends an analytics event containing the timestamp, IP, and User-Agent to a message broker (like Kafka) asynchronously, before returning the HTTP 302 redirect response.

8. Why use Kafka for analytics?
   **Answer:** To ingest high-velocity click events asynchronously without blocking the user's redirect request, decoupling the redirect engine from the analytics processing pipelines.

9. How would you shard URL data?
   **Answer:** By sharding the database using the hash of the `short_key` as the shard key, ensuring that read and write traffic is distributed evenly across database partitions.

10. What happens if Redis fails?
    **Answer:** The system falls back to querying the primary NoSQL/SQL database. To prevent the database from being overwhelmed, Redis is deployed in a highly available cluster with replication.

---

# 38. Notification System

1. How would you design a Notification System?
   **Answer:** I would design a microservices-based system:
   * **API Gateway** for notification requests.
   * **User Preference Service** to check if notifications are allowed.
   * **Message Queues (Kafka)** to buffer requests.
   * **Worker Pool** to call third-party delivery services (APNS, FCM, Twilio, SendGrid).

2. Why use Kafka for Notifications?
   **Answer:** To handle high traffic spikes, prioritize different message types (e.g., separate topics for OTPs vs marketing notifications), and buffer requests when third-party APIs are down or rate-limited.

3. What is Notification Preference?
   **Answer:** The user configuration determining which notification channels (SMS, Email, Push) they accept for different classes of notifications.

4. How do you send Email, SMS, and Push Notifications?
   **Answer:** By using specialized worker pools that integrate with external gateway APIs: SendGrid/Amazon SES for Email, Twilio for SMS, and APNS (Apple) / FCM (Google) for Push notifications.

5. What is a Dead Letter Queue?
   **Answer:** A backup queue where notification payloads that repeatedly fail to deliver (due to invalid parameters, expired tokens, etc.) are moved for manual review.

6. How do you retry failed notifications?
   **Answer:** By catching delivery failures in the worker, publishing the message back to a retry queue with a delay (using exponential backoff), and limiting the retry count before sending it to a DLQ.

7. Why use asynchronous processing?
   **Answer:** Sending notifications involves making network calls to external APIs. Asynchronous processing prevents these slow network operations from blocking user-facing application threads.

8. What database would you choose and why?
   **Answer:** PostgreSQL or MongoDB to store notification templates, user preferences, and historical delivery logs, while using Redis as a high-performance cache.

9. How do you scale notification workers?
   **Answer:** By packaging workers in containers (like Docker/Kubernetes) and auto-scaling them based on CPU utilization or Kafka consumer group lag.

10. How do you prevent notification spam?
    **Answer:** By implementing a rate-limiting service (e.g., using Redis token bucket) that enforces frequency caps on marketing notifications per user (e.g., maximum 3 pushes per day).

---

# 39. WhatsApp / Chat System

1. Why use WebSockets for Chat Systems?
   **Answer:** To maintain a persistent, bidirectional, full-duplex TCP connection, enabling the server to push new messages to the client instantly without the overhead of HTTP request/response loops.

2. How do you store offline messages?
   **Answer:** In a database like Apache Cassandra or DynamoDB, marked with an `undelivered` status. When the recipient client reconnects, the server fetches these messages and pushes them over the WebSocket.

3. How do message delivery ticks work?
   **Answer:**
   * **Single Tick (Sent):** The message has reached the chat server.
   * **Double Tick (Delivered):** The recipient client acknowledges receipt of the message over their WebSocket connection.
   * **Blue Ticks (Read):** The recipient opens the chat screen, triggering a read event back to the server.

4. How do you maintain message ordering?
   **Answer:** By generating a sequential message ID (using Snowflake IDs or client-side sequence numbers combined with server timestamps) for each chat room, allowing the client UI to sort messages correctly.

5. Why use Kafka in Chat Systems?
   **Answer:** To act as a high-throughput messaging backbone that decouples the WebSocket connection servers from the offline storage engines, push notification workers, and analytical systems.

6. How is user presence managed?
   **Answer:** The client sends periodic heartbeat pings over the WebSocket connection. The presence service updates the user's status; if heartbeats cease, the user is marked offline.

7. Why use Redis for online/offline status?
   **Answer:** Presence changes frequently. Redis handles the high write-volume of heartbeats in memory, and allows rapid status queries (e.g., "Is user X online?") with sub-millisecond latency.

8. How do you store media files?
   **Answer:** The sender uploads the media to an Object Store (like Amazon S3) via a pre-signed URL, and then sends the S3 media link, thumbnail, and file metadata as a standard text message over WebSockets.

9. How do you scale chat servers?
   **Answer:** By keeping track of active connections in a routing table (e.g., `user_id -> WebSocket server IP` in Redis). When User A sends a message to User B, the server queries the routing table to find User B's server and forwards the message.

10. How does WhatsApp support billions of messages?
    **Answer:** By decentralizing the architecture: messages are stored locally on user devices, using Erlang-based chat servers that handle millions of concurrent connections per machine, and using end-to-end encryption.

---

# 40. Instagram Feed

1. What is Fan-Out On Write?
   **Answer:** Pre-generating user feeds by immediately pushing a new post ID to the home feed cache (e.g., Redis List) of all followers when the creator posts.

2. What is Fan-Out On Read?
   **Answer:** Generating the home feed on demand when a user logs in by querying the database for all users they follow, pulling their latest posts, and merging them.

3. Why do celebrity accounts create feed generation problems?
   **Answer:** Pushing a celebrity's post to all of their millions of followers on write (Fan-Out on Write) causes write amplification, overloading the cache cluster and creating delivery lag.

4. How do you generate a home feed?
   **Answer:** By using a hybrid approach: pulling the precomputed feed of normal users from Redis, and merging it in real-time with the dynamic timelines of any followed celebrities.

5. Why use Redis for feed caching?
   **Answer:** Because feeds are read-heavy and require fast sorted lookups. Redis Sorted Sets (ZSET) store post IDs sorted by creation time, allowing $O(1)$ page fetches.

6. What is Feed Precomputation?
   **Answer:** Asassembling and caching a user's feed in the background before they log in, ensuring the feed displays instantly when they open the app.

7. How do you rank posts in a feed?
   **Answer:** By using a machine learning model that scores posts based on user engagement metrics, post age, affinity with the creator, and user demographics.

8. How do you scale feed generation?
   **Answer:** By implementing a hybrid fan-out model, limiting precomputation only to active users (evicting inactive feeds from the cache), and using asynchronous workers.

9. Fan-Out On Write vs Fan-Out On Read?
   **Answer:**
   * **Fan-Out On Write:** Fast reads ($O(1)$), slow writes (write amplification). Best for standard users.
   * **Fan-Out On Read:** Slow reads (heavy query on read), fast writes (write once). Best for celebrities.

10. How would you design Instagram end-to-end?
    **Answer:** Using microservices: CDN + Object Storage (S3) for media, Cassandra/PostgreSQL for post metadata, Redis for feed caches, Graph DB for follower graphs, and a hybrid fan-out pipeline with Kafka.

---

# 41. Food Delivery System

1. How do you find nearby restaurants?
   **Answer:** By indexing restaurant coordinates using GeoHash or MongoDB's 2D Sphere Index, allowing the system to query for restaurants within a specific geographic range.

2. What is a 2D Sphere Index?
   **Answer:** A database index designed for calculating geometry on an earth-like sphere, enabling rapid coordinates and boundary searches.

3. How do you find nearby delivery partners?
   **Answer:** By storing active driver coordinates in an in-memory spatial index (like Redis Geo) and executing a `GEORADIUS` query around the restaurant's coordinates.

4. Why use GeoHash?
   **Answer:** To discretize the world map into grid cells, which enables fast string-prefix searches for locations and optimizes spatial query caching.

5. How is order tracking implemented?
   **Answer:** The delivery partner's app streams GPS coordinates via WebSockets/gRPC to a location ingestion service, which updates Redis. The customer client reads these coordinates from Redis via WebSockets.

6. Why use Kafka in Food Delivery Systems?
   **Answer:** To handle high-velocity events (location streams, order status updates) asynchronously, decoupling ingestion from matching engines and analytical systems.

7. How do order status updates work?
   **Answer:** When status changes (e.g., `Order Prepared`), the service publishes an event to Kafka. Subscribed services update the database, trigger push notifications, and notify the delivery partner.

8. Why store live location in Redis?
   **Answer:** Because delivery partners update their locations every few seconds. Storing these transient, high-volume updates in a disk database would exhaust I/O. Redis handles these writes in-memory.

9. How do you scale delivery partner matching?
   **Answer:** By dividing the city into Geohash zones and running separate matching loops (using Top-K matching) for each zone independently, preventing a single global matching bottleneck.

10. How would you design Zomato/Swiggy end-to-end?
    **Answer:** With a microservice architecture: Restaurant Service, Order Service (PostgreSQL + Saga), Location Ingress (Go/Redis Geo), Matcher Engine, and Notification Service, using WebSockets for tracking.

---

# 42. E-Commerce System

1. What is the Overselling Problem?
   **Answer:** A race condition where multiple concurrent purchase requests check and modify the stock of a limited item at the same time, leading to sales exceeding the actual stock.

2. What is Inventory Reservation?
   **Answer:** A temporary hold placed on stock during the checkout stage (e.g., for 10 minutes), allowing the user to pay without risking the item being sold to someone else.

3. Why not reserve inventory on Add To Cart?
   **Answer:** Because high cart abandonment rates (70%) would lock up inventory unnecessarily, preventing active buyers from making purchases.

4. How do you prevent duplicate payments?
   **Answer:** By enforcing the client to generate a unique idempotency key for each checkout session, which the payment gateway and order services check before processing.

5. Why use Idempotency Keys?
   **Answer:** To guarantee that retrying a request (due to a network dropout or user double-click) will not result in duplicate credit card charges or duplicate order creations.

6. How does Saga Pattern help E-Commerce?
   **Answer:** It manages multi-service transactions (Order, Payment, Inventory, Shipping) by orchestrating local transactions and executing compensating actions (like refunding payment and releasing stock) if a step fails.

7. Why store cart data in Redis?
   **Answer:** Cart data is read and modified frequently during a session. Redis Hashed structures provide sub-millisecond access and prevent unnecessary load on the primary SQL database.

8. Product Cache vs Inventory Database?
   **Answer:**
   * **Product Cache:** Caches read-heavy, static product descriptions and images.
   * **Inventory Database:** Manages highly dynamic, write-heavy stock counts requiring strong transactional consistency.

9. How do you scale during Big Billion Days?
    **Answer:** By pre-warming caches, auto-scaling services, throttling reviews/recommendations, using queue-based payment processing, and utilizing Redis for distributed locks on hot inventory.

10. How would you design Amazon/Flipkart end-to-end?
    **Answer:** Using microservices: Search (Elasticsearch), Catalog (Cassandra + Redis), Cart (Redis), Orders (PostgreSQL + Saga), Payments (Idempotent API), and Notifications (Kafka).

---

# 43. Ride Sharing System

1. How do you find the nearest driver?
   **Answer:** By querying a real-time driver spatial index in Redis (`GEOSEARCH`) using the passenger's latitude and longitude coordinates.

2. Why use GeoHash?
   **Answer:** To partition geographic coordinates into localized grid cells, simplifying spatial searches and enabling fast driver-rider matching.

3. Why store driver locations in Redis?
   **Answer:** Because drivers send GPS updates every 3-4 seconds. Redis handles these high-volume, transient location writes in-memory, avoiding database disk bottlenecks.

4. What is Top-K Driver Matching?
   **Answer:** An optimization where the dispatch engine selects the $K$ closest, most appropriate drivers to offer the ride, rather than notifying all drivers in the region.

5. How do you implement live tracking?
   **Answer:** The driver's mobile client streams GPS coordinates via WebSockets to a location service. This service updates Redis and pushes the coordinates to the rider's active WebSocket connection.

6. Why use WebSockets in Uber?
   **Answer:** To maintain persistent, bidirectional connections that allow real-time location streaming, instant ride dispatch requests, and instant driver response updates.

7. How does Surge Pricing work?
   **Answer:** By measuring the real-time supply of available drivers versus the demand of ride requests inside a Geohash cell and applying a pricing multiplier when demand exceeds supply.

8. How do you scale ride matching?
   **Answer:** By running localized matching workers partitioned by Geohash zones, limiting matching computations to small, independent geographic pools.

9. What events would Kafka handle?
   **Answer:** Ride requests, driver acceptances, trip starts/ends, fare payments, and historical driver coordinate updates for traffic analysis.

10. How would you design Uber/Ola end-to-end?
    **Answer:** Using microservices: Location Ingest (Go/WebSockets), Driver Matcher (Redis Geo), Ride Manager (Saga orchestration), Surge Engine (Kafka + Flink), and Payment Service.
