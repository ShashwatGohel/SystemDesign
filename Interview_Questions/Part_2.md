# System Design Interview Revision Questions - Part 2

---

# 16. Event Driven Architecture

1. What is Event Driven Architecture?
   **Answer:** Event-Driven Architecture (EDA) is a software design pattern where decoupled services communicate asynchronously by producing, detecting, and consuming events that represent system state changes.

2. Why do we use Event Driven Systems?
   **Answer:** We use EDA to eliminate synchronous blocking calls, reduce direct dependencies between services, improve horizontal scalability, and increase system resilience (if a consumer crashes, the producer can keep running).

3. What is an Event?
   **Answer:** An event is an immutable, lightweight record of a state change or an action that has already occurred in the system (e.g., `OrderCreated`, `PaymentFailed`).

4. What is a Producer?
   **Answer:** A producer is the component or service that detects a state change and publishes the corresponding event message to an event broker or channel.

5. What is a Consumer?
   **Answer:** A consumer is a component or service that subscribes to an event broker, retrieves incoming event messages, and processes them by executing its local business logic.

6. How does Event Driven Architecture improve scalability?
   **Answer:** By allowing services to operate asynchronously. Producers and consumers do not block each other; thus, if consumer demand spikes, we can horizontally scale the consumer instances to handle the message backlog at their own pace without affecting producer performance.

7. What is Loose Coupling?
   **Answer:** Loose coupling is a design goal where services operate with minimal direct knowledge of each other. In EDA, the producer does not know who the consumers are, how they process the events, or even how many consumers exist.

8. What are the advantages of asynchronous communication?
   **Answer:**
   * **Low Latency:** The sender does not wait for a response; it returns immediately.
   * **Load Leveling:** Event brokers act as buffers to absorb sudden traffic spikes.
   * **Failure Isolation:** Temporary consumer failures do not block the producer.

9. Can a service both produce and consume events?
   **Answer:** Yes. In a typical event pipeline, a service consumes an event (e.g., `OrderPlaced`), performs its work, and then publishes a new event (e.g., `PaymentSucceeded`) to trigger the next step.

10. What are the disadvantages of Event Driven Systems?
    **Answer:**
    * **Complexity:** Difficult to debug and trace events across distributed networks.
    * **Data Consistency:** Requires handling eventual consistency and out-of-order event delivery.
    * **Idempotency Requirements:** Consumers must be idempotent to handle duplicate event delivery safely.

---

# 17. Saga Pattern

1. What problem does Saga Pattern solve?
   **Answer:** The Saga pattern solves the challenge of maintaining data consistency across multiple microservices with their own independent databases in a distributed transaction flow without using slow, blocking distributed locks.

2. Why can't ACID transactions work across microservices?
   **Answer:** Because microservices utilize the "Database per Service" pattern. Executing an ACID transaction across distinct databases requires a Two-Phase Commit (2PC) protocol, which blocks resources, suffers from high network latency, and introduces a single point of failure, severely limiting scalability.

3. What is a Compensating Transaction?
   **Answer:** A compensating transaction is an explicit rollback action designed to undo the changes made by a previously successful local transaction in the Saga, triggered when a subsequent step in the workflow fails.

4. What is Choreography-based Saga?
   **Answer:** A decentralized workflow where each microservice executes a local transaction and publishes events. Other services listen to those events and decide what action to take next, without any central controller.

5. What is Orchestration-based Saga?
   **Answer:** A centralized workflow where a dedicated coordinator (orchestrator) service explicitly instructs participating microservices on which local transactions to execute and manages the rollback sequence if any failure is encountered.

6. How does rollback happen in Saga?
   **Answer:** When a step fails, the Saga runtime (or orchestrator) stops the forward progress and triggers the predefined compensating transactions in reverse chronological order for all previously completed steps.

7. What happens when Payment fails after Inventory Reservation?
   **Answer:** The Saga orchestrator detects the payment failure and publishes an event or makes a direct call to trigger the compensating transaction in the Inventory Service (e.g., `ReleaseInventory`), returning the reserved stock.

8. Saga Pattern vs Two Phase Commit?
   **Answer:**
   * **Saga** is non-blocking, relies on eventual consistency, scales horizontally, but requires complex compensating logic.
   * **Two-Phase Commit (2PC)** guarantees strong consistency, blocks database locks until all nodes commit, and does not scale well in high-throughput distributed systems.

9. What are the advantages of Saga Pattern?
   **Answer:**
   * High performance and scalability due to non-blocking local transactions.
   * Supports long-lived business processes.
   * Decouples the transaction boundaries of individual microservices.

10. When should Saga Pattern be used?
    **Answer:** It should be used in distributed, multi-step workflows (like booking trips involving flights, hotels, and cars, or complex e-commerce order processing) where eventual consistency is acceptable.

---

# 18. Idempotency

1. What is Idempotency?
   **Answer:** Idempotency is the property of an operation where executing it multiple times with the same inputs produces the exact same system state and returns the same response as executing it once.

2. Why is Idempotency important in Payments?
   **Answer:** To prevent duplicate billing. If a client submits a payment request and encounters a network timeout, they will retry the request. Idempotency ensures that the second request does not charge the customer's credit card again.

3. What is an Idempotency Key?
   **Answer:** A unique identifier (usually a UUID generated by the client) passed in the request header that allows the server to identify retried requests and avoid executing duplicate business logic.

4. How do you implement Idempotency?
   **Answer:**
   * The server intercepts the request and checks a database/cache (like Redis) for the idempotency key.
   * If the key exists, the server returns the cached response from the first request.
   * If the key does not exist, the server locks the key, processes the request, saves the response, releases the lock, and returns the response.

5. What problems does Idempotency solve?
   **Answer:**
   * Double-spending and duplicate charges.
   * Creation of duplicate records (e.g., creating two orders instead of one).
   * Data corruption due to network-level retries.

6. What happens if a payment request is retried?
   **Answer:** The system matches the incoming idempotency key in its transaction cache, bypasses the payment processor, and instantly returns the cached status of the original transaction.

7. Is GET request idempotent?
   **Answer:** Yes, GET requests are idempotent and safe. By definition, making multiple GET requests to the same URL should only retrieve data and never modify the underlying state of the server.

8. Is POST request idempotent?
   **Answer:** No, POST requests are not idempotent by default. Sending the same POST request multiple times will typically create a new resource each time (e.g., adding multiple comments or placing multiple separate orders) unless specifically decorated with an idempotency mechanism.

9. Idempotency vs Deduplication?
   **Answer:**
   * **Deduplication** is the mechanical process of finding and filtering out duplicate requests before they are processed.
   * **Idempotency** is the semantic guarantee that regardless of how many times a request is processed, the system ends up in the same logical state and yields the same outcome.

10. Where is Idempotency commonly used?
    **Answer:** Payment processing gateways, e-commerce checkout checkouts, distributed message queue consumers, email delivery engines, and financial accounting ledgers.

---

# 19. Rate Limiting

1. Why is Rate Limiting required?
   **Answer:** To protect infrastructure from Denial of Service (DoS) attacks, prevent resource exhaustion, protect against brute-force attempts, control costs (API usage), and enforce fair-use limits across users.

2. What is Token Bucket Algorithm?
   **Answer:** An algorithm where tokens are added to a bucket of capacity $N$ at a constant rate. Each API request consumes a token. If the bucket has tokens, the request is allowed; otherwise, it is dropped. This algorithm allows short bursts of traffic.

3. What is Leaky Bucket Algorithm?
   **Answer:** An algorithm where requests enter a queue (the bucket) and are processed (leak out) at a constant, uniform rate. If the queue overflows, new requests are rejected. This enforces a smooth, constant output flow.

4. What is Fixed Window Rate Limiting?
   **Answer:** An algorithm that counts requests within fixed time windows (e.g., 60-second intervals). If the counter exceeds the limit within a window, requests are blocked until the window resets. It can suffer from double-traffic bursts at window boundaries.

5. What is Sliding Window Rate Limiting?
   **Answer:** An algorithm that tracks requests relative to a moving time window (e.g., the last 60 seconds from the current timestamp). It prevents boundary bursts by calculating request rates dynamically across overlapping window segments.

6. Token Bucket vs Leaky Bucket?
   **Answer:**
   * **Token Bucket** allows bursts of traffic (up to the bucket size) while enforcing a long-term average rate.
   * **Leaky Bucket** smooths out traffic, preventing bursts entirely by forcing a constant, metered processing rate.

7. Fixed Window vs Sliding Window?
   **Answer:** Fixed Window is simpler and cheaper to store but allows double the rate limit at window boundaries. Sliding Window is more accurate and prevents boundary surges, but requires more CPU and memory to track timestamps.

8. How do you implement Rate Limiting using Redis?
   **Answer:**
   * **Fixed Window:** Store a key like `user:rate_limit:window` and use `INCR` and `EXPIRE`.
   * **Sliding Window:** Store a Redis Sorted Set (ZSET) for each user. The key is the user ID, and the elements/scores are the request timestamps. We remove timestamps older than the window, count the set size, and add the new request if the count is within limits.

9. What race conditions occur in Distributed Rate Limiters?
   **Answer:** When multiple concurrent requests hit different app instances at the same millisecond, they may read the same current request count from Redis, approve the requests, and increment the counter, resulting in exceeding the rate limit.

10. How do Lua Scripts solve Rate Limiter race conditions?
    **Answer:** Redis executes Lua scripts atomically on a single thread. By running the rate-limiting check and increment operations within a single Lua script, we ensure no other operation can interleave, eliminating race conditions without expensive distributed locks.

---

# 20. API Design

1. What is a REST API?
   **Answer:** A REST (Representational State Transfer) API is an architectural style for designing stateless network services based on HTTP, where resources are identified by URIs and manipulated using standard HTTP verbs.

2. What are the common HTTP methods?
   **Answer:**
   * `GET`: Retrieve a resource.
   * `POST`: Create a new resource.
   * `PUT`: Replace an existing resource entirely.
   * `PATCH`: Update a resource partially.
   * `DELETE`: Remove a resource.

3. PUT vs PATCH?
   **Answer:** `PUT` replaces the target resource entirely with the request payload. `PATCH` applies partial modifications to the resource (only updating the fields provided in the payload).

4. GET vs POST?
   **Answer:** `GET` is safe and idempotent, used for fetching data, and passes parameters via URL query parameters. `POST` is unsafe and non-idempotent, used for creating/updating resources, and passes data in the request body.

5. What are RESTful principles?
   **Answer:**
   * Client-server separation.
   * Stateless communication.
   * Cacheability of resources.
   * Uniform Interface (resource identification, self-describing messages).
   * Layered system architecture.

6. What HTTP status codes are commonly used?
   **Answer:**
   * `200 OK`, `201 Created`, `204 No Content`
   * `304 Not Modified`
   * `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests`
   * `500 Internal Server Error`, `502 Bad Gateway`, `503 Service Unavailable`

7. What is API Versioning?
   **Answer:** The practice of managing changes to an API to prevent breaking existing clients. Common strategies include URI versioning (e.g., `/api/v1/`), query parameter versioning (e.g., `?version=1`), or header versioning (`Accept: application/vnd.company.v1+json`).

8. What is Idempotency in APIs?
   **Answer:** The guarantee that making multiple identical API requests will leave the system in the same state and return the same response as a single request, crucial for write endpoints.

9. How do you design scalable APIs?
   **Answer:** By keeping services stateless, implementing pagination for list endpoints, utilizing caching and CDNs, compression, rate-limiting, and offloading heavy tasks to asynchronous background queues.

10. REST vs gRPC?
    **Answer:**
    * **REST** relies on JSON over HTTP/1.1; it is human-readable, universal, but slow and verbose.
    * **gRPC** uses Protocol Buffers over HTTP/2; it is a binary protocol, strongly-typed, supports bidirectional streaming, and offers extremely high performance for inter-service communication.

---

# 21. Authentication

1. What is Authentication?
   **Answer:** Authentication is the security process of verifying the identity of a user, service, or device attempting to access a system (answering "Who are you?").

2. Authentication vs Authorization?
   **Answer:**
   * **Authentication** is proving your identity (e.g., verifying a username and password).
   * **Authorization** is verifying if the authenticated identity has permission to perform a specific action (e.g., checking if the user is an administrator).

3. What is JWT?
   **Answer:** A JSON Web Token (JWT) is an open standard (RFC 7519) that defines a compact, self-contained method for securely transmitting information between parties as a digitally signed JSON object.

4. How does JWT work?
   **Answer:** Upon successful login, the server signs a JWT containing user claims and returns it to the client. The client stores it (e.g., in HttpOnly cookies) and sends it in the `Authorization` header for subsequent requests. The server validates the signature mathematically, avoiding database lookups.

5. What are the parts of a JWT token?
   **Answer:** A JWT consists of three base64url-encoded parts separated by dots:
   * **Header:** Specifies the token type and signing algorithm (e.g., HMAC SHA256).
   * **Payload:** Contains the claims (e.g., user ID, roles, expiration time).
   * **Signature:** Created by hashing the header, payload, and a secret key to ensure integrity.

6. What is OAuth?
   **Answer:** OAuth is an open-standard authorization framework that allows third-party applications to obtain limited access to user accounts on an HTTP service (like Google or GitHub) without exposing user passwords.

7. What is Multi-Factor Authentication?
   **Answer:** Multi-Factor Authentication (MFA) requires users to provide two or more independent verification factors to gain access, combining something they know (password), something they have (authenticator app), or something they are (biometrics).

8. What are Refresh Tokens?
   **Answer:** Refresh tokens are long-lived, securely stored credentials used to request new, short-lived access tokens when they expire, reducing the risk of active token interception and improving user experience.

9. Session-based Authentication vs JWT?
   **Answer:**
   * **Session-based:** Stateful. Sessions are stored in a database/cache on the server; easy to revoke instantly but requires database lookup on every request.
   * **JWT:** Stateless. The token is stored on the client; scales extremely well for distributed microservices but is hard to revoke before expiration without a blacklist cache.

10. What are common Authentication vulnerabilities?
    **Answer:**
    * Brute-force attacks due to weak password policies.
    * JWT signature bypass (e.g., exploiting the `none` algorithm).
    * Session hijacking/fixation.
    * Storing tokens in unsafe client locations vulnerable to XSS.

---

# 22. Authorization

1. What is Authorization?
   **Answer:** Authorization is the security process that determines the access rights and permissions of an authenticated user, defining what resources they can access and what actions they can perform (answering "What are you allowed to do?").

2. Authentication vs Authorization?
   **Answer:** Authentication is the step of identifying a user (e.g., entering a PIN). Authorization happens next, checking if that user has permission to perform an action (e.g., transferring funds).

3. What is RBAC?
   **Answer:** Role-Based Access Control (RBAC) is an authorization model where permissions are grouped into specific roles (e.g., Admin, Editor, Viewer), and users are assigned to those roles.

4. What is ABAC?
   **Answer:** Attribute-Based Access Control (ABAC) is a dynamic authorization model where access decisions are evaluated based on attributes of the user (e.g., department), the resource (e.g., classification), and context (e.g., time, location).

5. What are Roles and Permissions?
   **Answer:**
   * **Permissions** are granular rules defining allowed actions on resources (e.g., `delete:user`).
   * **Roles** are collections of these permissions (e.g., an `Admin` role contains `create:user`, `read:user`, and `delete:user`).

6. How do you implement Authorization in APIs?
   **Answer:** By using middleware that decodes the user's roles/permissions from their token (e.g., JWT) and checks them against the route's required permissions. If the user lacks the necessary permission, a `403 Forbidden` status is returned.

7. What is Least Privilege Principle?
   **Answer:** A security design practice where users, accounts, and computing processes are granted only the minimum level of access rights and permissions necessary to perform their tasks.

8. What is Resource-Level Authorization?
   **Answer:** The practice of verifying access permissions at the individual resource instance level rather than just the endpoint level (e.g., verifying `current_user.id == post.author_id` before allowing a post deletion).

9. What is Policy-Based Authorization?
   **Answer:** A design where authorization logic is separated from application code, written as declarative policies that combine multiple rules (e.g., checking user age, location, and consent before granting access).

10. How do large systems manage permissions?
    **Answer:** By using centralized, high-performance authorization microservices (such as Google Zanzibar implementations, Open Policy Agent, or AWS Verified Permissions) that model permissions as a directed graph of relationships.

---

# 23. Session Management

1. What is a Session?
   **Answer:** A session is a stateful conversation history stored on the server to keep track of a user's logged-in state across multiple stateless HTTP requests.

2. How does Session Management work?
   **Answer:** Upon successful login, the server creates a unique session record in its store and sends a Session ID cookie to the client. The client browser automatically includes this cookie in subsequent requests. The server reads the cookie, looks up the session record, and reconstructs the user's state.

3. Session vs JWT?
   **Answer:** Sessions are stateful, stored on the server side, easily revocable, but require database/cache lookups. JWT is stateless, stored on the client, difficult to revoke instantly, but highly scalable because it requires no server-side lookup.

4. Where are Sessions stored?
   **Answer:**
   * In-memory on the application server (not scalable).
   * Relational databases (slow and heavy).
   * In-memory distributed caches like Redis or Memcached (fast, scalable, and volatile).

5. What is Session Expiry?
   **Answer:** A security configuration that automatically invalidates a session after a period of user inactivity (idle timeout) or a set time from creation (absolute timeout) to mitigate session hijacking risks.

6. What is Sticky Session?
   **Answer:** A load balancer configuration that routes all HTTP requests from a specific client to the exact same backend server instance that created their session.

7. Why can Sticky Sessions create problems?
   **Answer:** They break the stateless scaling of the application layer, lead to uneven traffic distribution (if one server handles many active sessions), and cause users to be logged out if their target server crashes.

8. How can Redis be used for Session Management?
   **Answer:** By acting as a shared, fast, in-memory session database. App servers write session JSON objects to Redis keys (with TTLs) and query them by Session ID, enabling all app servers to remain stateless.

9. How do you scale Session Management?
   **Answer:** By storing sessions in a highly available, sharded Redis or Memcached cluster, ensuring that any stateless application server can query session states instantly.

10. What happens when a session store crashes?
    **Answer:** If the session store crashes and is not replicated, all active users will lose their sessions and be forced to log in again. In production, this is prevented by running a replicated Redis Cluster with persistent backups.

---

# 24. WebSockets

1. What are WebSockets?
   **Answer:** WebSockets is a protocol (RFC 6455) that enables persistent, bidirectional, low-latency, full-duplex communication channels over a single long-lived TCP connection.

2. How are WebSockets different from HTTP?
   **Answer:** HTTP is unidirectional (client-pull), stateless, and closes after a request-response cycle. WebSockets are bidirectional (either side can push data at any time), stateful, and keep the connection open indefinitely.

3. What is Full Duplex Communication?
   **Answer:** A communication channel where both the client and the server can send and receive messages simultaneously, without waiting for the other party to finish or initiate.

4. How is a WebSocket connection established?
   **Answer:** The client initiates a standard HTTP request with an `Upgrade: websocket` header. If the server supports the protocol, it completes the handshake by returning an HTTP `101 Switching Protocols` status code, upgrading the existing TCP connection to a WebSocket.

5. Why are WebSockets better than Polling?
   **Answer:** Polling sends periodic HTTP requests, creating massive network overhead from headers and TCP handshakes. WebSockets open a single connection, allowing raw data frames to be sent with minimal header overhead, reducing latency and bandwidth.

6. What is Long Polling?
   **Answer:** A fallback technique where the client requests data from the server, and the server intentionally holds the request open until new data is available or a timeout occurs, simulating a push before closing the request.

7. When should WebSockets be used?
   **Answer:** For real-time applications such as instant messaging chat rooms, multiplayer games, real-time financial trading dashboards, collaborative document editing, and live sports tickers.

8. How do WebSockets scale?
   **Answer:** WebSocket scaling is challenging due to persistent TCP connections. We use load balancers that support connection upgrades, scale WebSocket servers horizontally, and use a Pub/Sub message broker (like Redis or Kafka) to distribute events to the correct server instances hosting target clients.

9. What happens when a WebSocket server crashes?
   **Answer:** All TCP connections held by that server are dropped. The client application must detect the drop and immediately try to reconnect, utilizing an exponential backoff jitter strategy to avoid DDOSing the remaining servers.

10. Examples of systems that use WebSockets?
    **Answer:** Discord, Slack, Figma, Binance, Uber (for real-time driver tracking).

---

# 25. GeoHash

1. What is GeoHash?
   **Answer:** GeoHash is a geocoding system that encodes latitude and longitude coordinates into a short string of alphanumeric characters using a hierarchical grid structure.

2. Why do Uber and Zomato use GeoHash?
   **Answer:** To partition geographic space into predictable rectangular grids. This allows them to store and query location data (such as nearby drivers or restaurants) using rapid string prefix match queries.

3. How does GeoHash work?
   **Answer:** It recursively divides the 2D world map into grid cells. Longitude and latitude are converted to binary sequences, interleaved, and encoded into a Base32 string. Longer strings represent smaller, more precise grid cells.

4. How do you find nearby drivers using GeoHash?
   **Answer:** By converting the user's location into a Geohash string of the desired search resolution (e.g., 6 characters) and querying the database for drivers whose Geohash values start with that prefix.

5. What are neighboring GeoHash cells?
   **Answer:** The 8 grid squares immediately adjacent to a target Geohash cell. Querying neighboring cells is essential because a user near the edge of their cell might be physically closer to a driver in the next cell than one in their own.

6. Why not scan the entire database?
   **Answer:** Scanning the entire database requires calculating distance formulas (like the Haversine formula) for every single entry, which is an $O(N)$ operation. For millions of records, this would crash the database.

7. How does GeoHash improve scalability?
   **Answer:** It transforms a complex 2D spatial search into a 1D string lookup. Database engines can index the `geohash` string column, reducing query complexity from $O(N)$ to $O(\log N)$.

8. What are the limitations of GeoHash?
   **Answer:**
   * **Boundary Problems:** Two locations can be extremely close physically but on opposite sides of a grid boundary, resulting in completely different Geohashes.
   * **Inflexible Resolution:** The grid size steps are fixed and cannot be customized smoothly.

9. GeoHash vs 2D Sphere Index?
   **Answer:**
   * **GeoHash** is a database-independent string representation that can be indexed easily in any store (NoSQL, SQL, Redis).
   * **2D Sphere Index** is a native database feature (e.g., R-Tree) that calculates precise distances on a sphere, supporting complex polygons but is database-dependent.

10. What systems commonly use GeoHash?
    **Answer:** Ridesharing apps (Uber, Lyft), food delivery platforms (DoorDash, Zomato), and location-based social networks (Tinder).

---

# 26. 2D Sphere Index

1. What is a 2D Sphere Index?
   **Answer:** A database-level geospatial index that calculates geometries on an earth-like sphere, allowing you to index and query locations stored as GeoJSON objects (Points, Polygons, Lines).

2. Why do we need a Geospatial Index?
   **Answer:** To perform spatial operations—like finding points of interest inside a polygon, calculating precise distances, or performing radius searches—without running heavy trigonometric math on application servers.

3. How does MongoDB's 2D Sphere Index work?
   **Answer:** It uses Google's S2 library to project the earth's sphere onto a cube and subdivide it hierarchically into cells, converting coordinates into 64-bit integer cell IDs indexed via a B-Tree.

4. How do you find nearby restaurants?
   **Answer:** By running a geospatial query (e.g., MongoDB's `$near` operator) on the indexed location field, specifying the coordinate and the maximum distance constraint.

5. GeoHash vs 2D Sphere Index?
   **Answer:** Geohash is a simple string prefix representation that works on any database but suffers from grid edge cases. 2D Sphere Index is a powerful database-native index that handles complex shapes and exact spherical distances but is computationally heavier.

6. What is Radius Search?
   **Answer:** A spatial query that finds all indexed database records located within a specified circular distance (e.g., 2000 meters) from a central set of latitude/longitude coordinates.

7. How do you query locations within 5 km?
   **Answer:** In MongoDB, you query the indexed field using `$near` with `$maxDistance: 5000`. In PostgreSQL with PostGIS, you use `ST_DWithin(geog, ST_MakePoint(lon, lat), 5000)`.

8. What are the limitations of Geospatial Queries?
   **Answer:** They are CPU-intensive, consume significant memory, and can experience latency spikes in highly dense geographic areas (e.g., searching for taxi drivers in downtown Manhattan).

9. Why is indexing important in location-based systems?
   **Answer:** Without a geospatial index, the database engine must execute a full table scan and compute spherical geometry distance equations on every record, causing severe resource starvation.

10. When should you use GeoHash instead?
    **Answer:** When you need a lightweight, database-agnostic solution, when caching query results as simple key-value pairs is critical, or when you are using databases that lack native geospatial indices (like Redis or DynamoDB).

---

# 27. Microservices

1. What are Microservices?
   **Answer:** Microservices is an architectural style that structures an application as a collection of small, autonomous, loosely coupled services, each responsible for a single business capability and deployable independently.

2. Monolith vs Microservices?
   **Answer:** A Monolith is a single, unified codebase and deployable package; it is easy to build initially but hard to scale and maintain. Microservices partition the application into decoupled, network-connected services, enabling fast, independent deployment cycles but adding network latency and operational complexity.

3. What are the advantages of Microservices?
   **Answer:**
   * **Independent Deployments:** Teams can release updates without coordinating.
   * **Targeted Scalability:** Scale only the resource-heavy services.
   * **Technology Agnostic:** Different services can use different programming languages or databases.
   * **Fault Isolation:** A crash in a non-critical service does not down the entire system.

4. What are the disadvantages of Microservices?
   **Answer:**
   * **Operational Complexity:** Requires robust CI/CD, service meshes, and container orchestration (Kubernetes).
   * **Network Latency:** Inter-service calls introduce network delay.
   * **Data Consistency:** Managing distributed transactions (eventual consistency) is difficult.
   * **Harder Debugging:** Tracing requests across multiple servers requires distributed logging (Zipkin, Jaeger).

5. How do Microservices communicate?
   **Answer:**
   * **Synchronous:** REST APIs (HTTP/JSON), gRPC.
   * **Asynchronous:** Message brokers (RabbitMQ, Amazon SQS), event streams (Apache Kafka).

6. Why do Microservices improve scalability?
   **Answer:** Because services can be scaled independently. If the order placement service experiences high traffic, we can scale only that service, leaving the catalog and payment services running on their baseline capacities.

7. What is Service Independence?
   **Answer:** The concept where a microservice can be developed, tested, deployed, and scaled by its owning team without relying on, coordinating with, or breaking other services.

8. What are common challenges in Microservices?
   **Answer:** Maintaining data consistency, managing network partitions, routing requests (service discovery), implementing distributed transactions, and compiling unified logs.

9. How do Microservices handle failures?
   **Answer:** By applying resilience patterns such as **Circuit Breakers** (halting calls to failing services), **Retries with Jitter**, **Rate Limiters**, and providing **Graceful Degradation** (returning cached or fallback values).

10. When should you choose a Monolith instead?
    **Answer:** For early-stage startups building MVPs, small development teams, simple applications, or when rapid prototyping and minimal operational overhead are prioritized.

---

# 28. Database Per Service Pattern

1. What is Database Per Service Pattern?
   **Answer:** A microservices design pattern where each microservice owns its private database. Other services cannot access this data directly; they must interact via the owner service's public APIs.

2. Why is sharing a database between services bad?
   **Answer:** It creates tight coupling. A schema modification by one team can break another team's service, it bypasses API contracts, and creates a database-level performance bottleneck and single point of failure.

3. How does Database Per Service improve independence?
   **Answer:** It allows teams to modify, optimize, and choose their database systems (e.g., SQL for transactions, NoSQL for logs) without coordinating with other teams or risking breaking external services.

4. What are the challenges of Database Per Service?
   **Answer:**
   * Inability to perform standard database joins across service boundaries.
   * Maintaining transactional consistency across multiple databases.
   * Managing duplicated data and syncing reference entities.

5. How do services communicate data?
   **Answer:** By exposing API endpoints to query details, or asynchronously by publishing events to an event bus (e.g., Kafka) when local database records change.

6. Why is Event Driven Architecture common with this pattern?
   **Answer:** Because it allows services to maintain local read-optimized copies of necessary external data asynchronously, avoiding slow, blocking synchronous API calls to other services.

7. What is data duplication in Microservices?
   **Answer:** The deliberate duplication of common fields (e.g., storing `user_name` alongside `order_id` in an Order database) to avoid making network requests to the User service every time order details are requested.

8. How do joins work across services?
   **Answer:**
   * **API Composition:** An API gateway or consumer service queries multiple microservices and joins their JSON payloads in memory.
   * **CQRS (Command Query Responsibility Segregation):** A read-optimized database subscribes to events from all services, aggregating and joining the data in advance.

9. How do you maintain consistency?
   **Answer:** By utilizing eventual consistency patterns, primarily the Saga Pattern, backed by the Transactional Outbox pattern to guarantee event publishing.

10. When should this pattern be used?
    **Answer:** In mature microservice architectures where scaling, team autonomy, and independent database performance are absolute requirements.

---

# 29. Fan-Out On Write

1. What is Fan-Out On Write?
   **Answer:** Fan-out on write (or the Push model) is a feed generation technique where a user's new post is immediately written (pushed) to the home feed caches of all their followers.

2. How does Fan-Out On Write work?
   **Answer:** When a user creates a post: the system writes the post to the database, queries the user's followers, and pushes the post ID into the feed cache (e.g., Redis List) of every follower in the background.

3. Why is Fan-Out On Write used in social media feeds?
   **Answer:** Because social media platforms are highly read-heavy (users check their feeds far more often than they post). Precomputing feeds on write makes feed retrieval extremely fast for the end user.

4. What are the advantages of Fan-Out On Write?
   **Answer:** Extremely fast read latency ($O(1)$ lookup time) when a user opens their application, as the feed is already pre-assembled in memory.

5. What are the disadvantages of Fan-Out On Write?
   **Answer:**
   * **Write Amplification:** A single post from a user with 1 million followers results in 1 million database/cache write operations.
   * High memory storage requirements for holding precomputed feeds for inactive users.

6. Why do celebrity accounts create problems?
   **Answer:** If a celebrity with 50 million followers posts, writing to 50 million feed caches in real time will block workers, exhaust memory, and create massive system-wide lag (the "celebrity problem").

7. How is feed precomputation performed?
   **Answer:** By using asynchronous workers (e.g., RabbitMQ or Celery tasks) that pick up the post event, retrieve the list of followers, and update the followers' Redis feed lists in batches.

8. What storage is commonly used for feed caches?
   **Answer:** In-memory databases like Redis, utilizing Lists or Sorted Sets (ZSETs) capped at a fixed size (e.g., storing the latest 500 post IDs).

9. How does Fan-Out On Write improve read performance?
   **Answer:** It eliminates the need to query follower graphs and perform database sort and join operations when a user requests their feed.

10. When should Fan-Out On Write be avoided?
    **Answer:** For users with very high follower counts (celebrities) or in systems where write throughput is high and read throughput is low.

---

# 30. Fan-Out On Read

1. What is Fan-Out On Read?
   **Answer:** Fan-out on read (or the Pull model) is a feed generation technique where a user's home feed is assembled dynamically in real-time when they query it.

2. How does Fan-Out On Read work?
   **Answer:** When a user opens their home page, the system queries the follower database to get the list of who they follow, fetches the latest posts from those accounts, merges and sorts them by timestamp in memory, and returns the feed.

3. Why is Fan-Out On Read used for celebrities?
   **Answer:** To prevent write amplification. When a celebrity posts, the post is saved once in the celebrity's timeline database. No background writes are performed to their millions of followers.

4. Fan-Out On Read vs Fan-Out On Write?
   **Answer:**
   * **Fan-Out On Read (Pull):** Cheap writes (write once), expensive reads (merge on demand).
   * **Fan-Out On Write (Push):** Expensive writes (write to all), cheap reads (instant fetch).

5. What are the advantages of Fan-Out On Read?
   **Answer:**
   * No write amplification.
   * Simple post creation logic.
   * Significantly less storage overhead since we don't store precomputed feeds.

6. What are the disadvantages of Fan-Out On Read?
   **Answer:** Slow read latency. Fetching and sorting posts from hundreds of followed accounts at request time can result in long page-load delays.

7. How does feed generation happen at request time?
   **Answer:** The system retrieves the list of IDs the user follows, queries their latest post timelines, performs a K-way merge sort in memory, filters out spam, and returns the sorted array of posts.

8. Why does Fan-Out On Read increase read cost?
   **Answer:** Because every page refresh triggers multiple database queries and computational sorting, consuming high database CPU and I/O bandwidth.

9. How do companies use Hybrid Feed Generation?
   **Answer:** They use a hybrid model: Fan-out on write (push) is used for normal users, while Fan-out on read (pull) is used for celebrity accounts. When a user requests their feed, the system pulls the precomputed push feed and merges it with the dynamic timelines of any celebrities they follow.

10. When should Fan-Out On Read be preferred?
    **Answer:** When the system has very high follower counts (like Twitter/X or Instagram), or when storage space is constrained.
