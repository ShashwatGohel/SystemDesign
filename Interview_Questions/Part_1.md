# System Design Interview Revision Questions - Part 1

---

# 1. Requirements Gathering

1. **What are Functional Requirements?**
   
   **Answer:** Functional requirements define the core behaviors, features, and capabilities of the system—effectively, what the system must do. In an interview, I identify these as the direct actions users can perform (e.g., "A user should be able to upload a video," "A user should be able to search for friends," or "The system must send a verification email upon registration"). They are the concrete functional APIs we need to design.

2. **What are Non-Functional Requirements?**
   
   **Answer:** Non-functional requirements (NFRs) describe the operational constraints and quality attributes of the system—how it should perform. These set the SLAs and engineering boundaries, covering aspects such as latency (e.g., p99 write latency < 200ms), availability (e.g., 99.99% uptime), consistency (e.g., eventual consistency for timelines vs. strong consistency for transactions), durability, security, and scalability.

3. **Why are requirements important before designing a system?**
   
   **Answer:** Requirements define the scope and prevent either over-engineering or under-engineering. In a system design interview, building without requirements is a red flag. For instance, designing a global, multi-region database with strong consistency is an expensive waste of effort if the system only needs to support 1,000 local users with eventual consistency. Clarifying requirements sets the constraints for QPS, storage, and architecture.

4. **How do you estimate system scale?**
   
   **Answer:** I start with the Daily Active Users (DAU). From there, I estimate the read/write ratio to calculate the read QPS and write QPS. Next, I multiply the write QPS by the average payload size to determine the daily data ingestion rate and bandwidth requirements (ingress/egress). Finally, I project this storage requirement over a standard period, such as 3 or 5 years, adjusting for peak-to-average traffic ratios.

5. **What questions should you ask before starting a system design interview?**
   
   **Answer:** I always ask:
   * Who are the users, and what are the core use cases (MVP)?
   * What is the expected scale (DAU, active writers/readers)?
   * What is the read-to-write ratio?
   * What are the latency targets (e.g., p99 response time) and availability requirements (how many 9s)?
   * Do we prioritize strong consistency over availability, or is eventual consistency acceptable?

6. **What is the difference between user requirements and system requirements?**
   
   **Answer:** User requirements are high-level, natural-language descriptions of what the user expects the product to do from their perspective (e.g., "I want to see my friends' updates in real time"). System requirements translate those needs into technical, actionable specifications for engineers (e.g., "The server must push update events to connected clients via WebSockets within 500ms of publish, maintaining an open connection limit of 50k per pod").

7. **How do you identify bottlenecks from requirements?**
   
   **Answer:** I analyze the resource constraints implied by the requirements. If the write volume is extremely high (e.g., logs ingestion), the database write bandwidth and disk I/O are the bottlenecks. If it is a read-heavy search system, database query performance, cache hits, or network egress are the bottlenecks. If we are processing video uploads, CPU/GPU transcoders and temporary storage become the bottlenecks.

8. **What metrics would you collect during requirement gathering?**
   
   **Answer:**
   * Daily Active Users (DAU) and Monthly Active Users (MAU).
   * Average and peak QPS (Queries Per Second) for both read and write APIs.
   * Average size of the data items being created or retrieved.
   * Target latency percentiles (p95 and p99 response times).
   * Data retention requirements (e.g., "store indefinitely" vs. "purge after 30 days").

9. **How do requirements influence database selection?**
   
   **Answer:** The choice depends heavily on data structure and transactional needs. If the requirements specify highly structured data, complex relations, and a strict need for ACID transactions (e.g., financial ledger transactions), I will opt for a relational database (SQL). If the requirements demand high write throughput, flexible schemas, or simple key-value/document-based access over partitioned datasets, I will choose NoSQL.

10. **How do requirements influence architecture decisions?**
   
    **Answer:** If low latency is a strict NFR, I must architect for caching, CDNs, and edge deployments. If high availability is prioritized over strong consistency (an AP system under the CAP theorem), I will opt for asynchronous replication and message queues to decouple services. If the system needs to support a global user base, I will design a multi-region active-active deployment.

---

# 2. Scalability

1. **What is scalability?**
   
   **Answer:** Scalability is the ability of a system to handle an increasing amount of load (such as higher QPS, larger datasets, or more concurrent users) by adding hardware resources, without degrading performance or reliability.

2. **What is Vertical Scaling?**
   
   **Answer:** Vertical scaling (scaling up) means increasing the capacity of an individual server by adding more resources, such as upgrading to a faster CPU, increasing RAM, or installing larger SSDs.

3. **What is Horizontal Scaling?**
   
   **Answer:** Horizontal scaling (scaling out) means adding more machines/nodes to the resource pool and distributing the workload across them.

4. **Vertical vs Horizontal Scaling?**
   
   **Answer:** Vertical scaling is simple to implement because it doesn't introduce distributed system challenges, but it has a hard hardware ceiling and creates a single point of failure (SPOF). Horizontal scaling offers virtually infinite capacity and eliminates SPOFs, but it introduces network latency, data consistency issues (like split-brain), and operational complexity.

5. **Why do modern systems prefer horizontal scaling?**
   
   **Answer:** Modern systems prefer horizontal scaling because it is more cost-effective (using commodity servers rather than expensive specialized hardware), avoids the physical limits of single-server hardware upgrades, and enables high availability and fault tolerance (if one server fails, others continue running).

6. **What are the limitations of vertical scaling?**
   
   **Answer:**
   * Physical hardware limits (there is a maximum amount of RAM and CPU cores a single motherboard can support).
   * Non-linear cost scaling (high-end hardware cost increases exponentially compared to performance).
   * Single Point of Failure (SPOF).
   * Downtime is typically required to upgrade hardware.

7. **How does horizontal scaling improve availability?**
   
   **Answer:** It eliminates single points of failure. By running multiple redundant instances of an application behind a load balancer, the failure of a single node does not bring down the entire service; the remaining healthy nodes continue to process incoming requests.

8. **How do load balancers help scalability?**
   
   **Answer:** Load balancers act as the entry point for incoming traffic, distributing requests across a pool of horizontally scaled backend servers. This prevents any single server from becoming overloaded and allows us to dynamically add or remove servers from the pool without interrupting the client.

9. **How do databases scale horizontally?**
   
   **Answer:** Databases scale horizontally through two primary techniques:
   * **Replication:** Directing read queries to read replicas while keeping a primary node for writes.
   * **Sharding (Partitioning):** Splitting the database rows across multiple physical machines based on a partition key, so that each machine only holds a subset of the data.

10. **How would you scale a system from 1K users to 100M users?**
   
    **Answer:** I would take a phased approach:
    * **1K Users:** A single monolithic app server and one database on the same instance.
    * **10K Users:** Move the database to a dedicated instance and introduce an in-memory cache (like Redis) for read-heavy queries.
    * **100K Users:** Introduce a load balancer and spin up multiple stateless application instances. Set up database read replicas.
    * **1M Users:** Decouple services using message queues for asynchronous tasks, and store static assets in a CDN.
    * **10M - 100M Users:** Microservices architecture, database sharding, consistent hashing, and multi-region active-active deployments.

---

# 3. Load Balancers

1. **Why do we need a Load Balancer?**
   
   **Answer:** We need a load balancer to distribute incoming traffic evenly across backend servers, prevent any single instance from becoming a bottleneck, hide our internal server topology from the public, perform health checks to route around failed nodes, and handle SSL termination.

2. **What is Layer 4 Load Balancing?**
   
   **Answer:** L4 load balancing operates at the Transport layer of the OSI model (TCP/UDP). It routes traffic based on network packet headers (source/destination IP and port) without looking inside the application payload. It is highly efficient and fast because it does not inspect HTTP headers or payload.

3. **What is Layer 7 Load Balancing?**
   
   **Answer:** L7 load balancing operates at the Application layer (HTTP/HTTPS). It inspects the actual payload, headers, cookies, and URL path to make intelligent routing decisions (e.g., routing requests from `/images` to an image server and `/api` to an application server).

4. **Round Robin vs Least Connections?**
   
   **Answer:**
   * **Round Robin:** Passes requests sequentially to each server in the pool. It works well when all servers have similar specifications and tasks take a similar amount of time to complete.
   * **Least Connections:** Routes new requests to the server with the fewest active connections. This is preferred when request processing times vary significantly.

5. **What are Health Checks?**
   
   **Answer:** Health checks are periodic ping or HTTP requests sent by the load balancer to the backend servers. If a server fails to respond within a timeout or returns an error status code, the load balancer marks it unhealthy and stops routing traffic to it.

6. **What happens when a server fails behind a Load Balancer?**
   
   **Answer:** The load balancer detects the failure via its health check mechanism. It immediately stops forwarding new requests to the failed server and redistributes the incoming traffic to the remaining healthy servers in the pool.

7. **What are Sticky Sessions?**
   
   **Answer:** Sticky sessions (session affinity) configure the load balancer to route all requests from a specific user to the same backend server for the duration of their session. This is usually implemented using a session cookie. It simplifies state management but can lead to uneven load distribution.

8. **Can a Load Balancer become a bottleneck?**
   
   **Answer:** Yes, if the throughput of incoming network packets or concurrent connections exceeds the load balancer's hardware capacity. To mitigate this, we scale the load balancing layer horizontally using DNS Round Robin (Anycast DNS) to route to multiple physical load balancer instances.

9. **How do companies make Load Balancers highly available?**
   
   **Answer:** By deploying them in active-passive or active-active clusters. In an active-passive setup, a passive load balancer monitors the active one; if the active one fails, the passive one takes over its virtual IP address (using protocols like VRRP or Keepalived) instantly.

10. **Where should a Load Balancer be placed in the architecture?**
   
    **Answer:** Load balancers should be placed at multiple entry points:
    * Between the client and the web servers/CDN.
    * Between the web servers and internal microservices.
    * Between the microservices and database read replica clusters.

---

# 4. Caching

1. **Why is caching needed?**
   
   **Answer:** Caching is needed to reduce data retrieval latency by storing frequently requested data in fast, in-memory storage (RAM). This decreases the read load on downstream databases, reduces network hops, and improves overall system response time.

2. **What is a Cache Hit?**
   
   **Answer:** A cache hit occurs when the application requests data, and that data is successfully found in the cache and returned immediately to the client without querying the source database.

3. **What is a Cache Miss?**
   
   **Answer:** A cache miss occurs when requested data is not found in the cache. The application must then query the primary database, return the data to the user, and typically write that data to the cache for future requests.

4. **What is Cache Aside Pattern?**
   
   **Answer:** In the cache-aside (lazy loading) pattern, the application is responsible for interacting with both the cache and the database. The app checks the cache first; if it's a hit, it returns the data. If it's a miss, the app queries the database, updates the cache with the new data, and returns the response.

5. **What is Read Through Cache?**
   
   **Answer:** In a read-through pattern, the application treats the cache as the main data store. On a cache miss, the cache provider itself (not the application code) fetches the data from the database, updates itself, and returns the data back to the application.

6. **What is Write Through Cache?**
   
   **Answer:** In a write-through pattern, data is written to the cache and the database synchronously. A write operation is only considered complete when both the cache and the database have been updated, ensuring high data consistency at the expense of higher write latency.

7. **What is Write Back Cache?**
   
   **Answer:** Write-back (write-behind) writes data directly to the cache. The cache then asynchronously writes the data back to the primary database, often in batches. This provides extremely low write latency but carries a risk of data loss if the cache node crashes before the data is persisted to the database.

8. **What is Cache Invalidation?**
   
   **Answer:** Cache invalidation is the process of removing or updating cached data when the underlying database source changes. This is critical for preventing the application from serving stale, outdated data.

9. **What problems can stale cache create?**
   
   **Answer:** Serving stale cache can lead to functional consistency bugs—such as showing incorrect account balances, displaying out-of-stock items as purchasable, or presenting outdated configuration values, which degrades user trust.

10. **How do you decide what data should be cached?**
   
    **Answer:** I look for data that is read frequently but modified infrequently (high read-to-write ratio), data that is expensive to compute (like complex aggregations or report queries), and static configuration or metadata.

---

# 5. Redis

1. **What is Redis?**
   
   **Answer:** Redis (Remote Dictionary Server) is an open-source, in-memory, key-value data structure store. It is widely used as a cache, database, message broker, and rate limiter.

2. **Why is Redis faster than traditional databases?**
   
   **Answer:** Redis is exceptionally fast because it stores all data in memory (RAM), avoiding disk seek time overhead. It also utilizes highly optimized in-memory data structures and a single-threaded event loop mechanism (using non-blocking I/O multiplexing), which eliminates thread context switching and lock contention.

3. **What data structures does Redis support?**
   
   **Answer:** Redis supports strings, lists, sets, sorted sets (ZSETs), hashes, bitmaps, HyperLogLogs, geospatial indexes, and streams.

4. **When should Redis be used?**
   
   **Answer:** Redis should be used for high-throughput caching, session storage, distributed locking, real-time analytics, Leaderboards (using Sorted Sets), API rate limiting, and pub/sub message brokering.

5. **Redis vs Memcached?**
   
   **Answer:** Redis supports rich data structures, persistence options (RDB and AOF), replication, and clustering. Memcached is a simpler, multi-threaded, purely in-memory key-value cache that does not support complex data types, native replication, or persistence.

6. **How does Redis support caching?**
   
   **Answer:** By storing key-value pairs in memory and providing TTL (Time-To-Live) expirations. When memory limits are reached, it automatically evicts keys based on configured eviction policies like LRU (Least Recently Used) or LFU (Least Frequently Used).

7. **How does Redis support rate limiting?**
   
   **Answer:** Rate limiting can be implemented by using the `INCR` command and setting an `EXPIRE` window on a key, or by using Sorted Sets (ZSET) to implement a sliding window log, tracking timestamped request counts.

8. **What happens if Redis crashes?**
   
   **Answer:** If persistence is enabled, Redis recovers data from RDB snapshots or the AOF (Append Only File) log on startup. To prevent downtime and data loss, a production environment should use Redis Sentinel (for automatic failover) or Redis Cluster.

9. **How do Redis Replicas work?**
   
   **Answer:** Redis uses leader-follower (master-replica) replication. Write operations are performed on the master node, which then asynchronously replicates the commands to the follower nodes, enabling horizontal scale-out for read queries.

10. **What is Redis Cluster?**
   
    **Answer:** Redis Cluster is a distributed implementation of Redis that automatically shards data across multiple nodes using 16,384 hashing slots. It provides linear scalability and automatic failover without requiring Sentinel.

---

# 6. CDN

1. **What is a CDN?**
   
   **Answer:** A Content Delivery Network (CDN) is a geographically distributed network of proxy servers (Points of Presence) that cache content closer to end-users at the network edge.

2. **Why do we need a CDN?**
   
   **Answer:** We use a CDN to reduce latency for global users, offload traffic and bandwidth costs from our origin servers, and absorb sudden traffic spikes or DDoS attacks.

3. **What type of data is stored in a CDN?**
   
   **Answer:** Mainly static assets such as HTML, CSS, JavaScript, images, and videos. It can also cache dynamic API responses that have appropriate cache-control headers.

4. **What is Edge Caching?**
   
   **Answer:** Edge caching is the process of storing content on servers situated at the edge of the Internet, physically close to the user's ISP, minimizing the number of network hops required to retrieve resources.

5. **How does CDN reduce latency?**
   
   **Answer:** By serving data from the physically closest edge server, minimizing the Round Trip Time (RTT) of network packets. Instead of a request traveling across an ocean to an origin server, it is resolved at a local edge server.

6. **CDN vs Redis?**
   
   **Answer:** A CDN caches static/web content globally at the edge of the network close to the end-users. Redis is an application-level in-memory cache located within the data center, close to the application servers, to speed up database queries.

7. **What is a Cache Hit in a CDN?**
   
   **Answer:** A cache hit occurs when a CDN edge server has a valid, non-expired copy of the requested resource in its local storage and serves it directly to the user.

8. **What is a Cache Miss in a CDN?**
   
   **Answer:** A cache miss occurs when the CDN edge server does not have the resource (or the resource has expired), requiring the CDN to fetch it from the origin server, cache it locally, and return it to the user.

9. **How do CDNs handle global traffic?**
   
   **Answer:** CDNs use Anycast routing, which allows multiple edge servers to share the same IP address. Routers automatically send the user's packets to the path that is topologically closest.

10. **What happens when CDN data becomes stale?**
   
    **Answer:** The CDN edge server makes a conditional request (using headers like `If-Modified-Since` or `ETag`) to the origin server. If the origin returns a `304 Not Modified`, the CDN marks the cached version fresh; otherwise, it pulls the new file and updates its cache.

---

# 7. Database Fundamentals

1. **What is a Database?**
   
   **Answer:** A database is an organized collection of data, structured to be easily accessed, managed, updated, and persisted electronically within a computer system.

2. **What is a Primary Key?**
   
   **Answer:** A primary key is a column or set of columns in a table that uniquely identifies each row in that table. It must contain unique values and cannot contain null values.

3. **What is a Foreign Key?**
   
   **Answer:** A foreign key is a column or group of columns in one table that references the primary key of another table, establishing a link between the data in the two tables and enforcing referential integrity.

4. **What is an Index?**
   
   **Answer:** An index is a database data structure (typically a B-Tree) created on one or more columns of a table. It acts like a book's index, enabling the database engine to find records rapidly without performing a slow full table scan.

5. **What is Normalization?**
   
   **Answer:** Normalization is the process of organizing database tables to minimize data redundancy and dependency (e.g., splitting one large table into multiple related tables). This reduces data anomalies during writes.

6. **What is Denormalization?**
   
   **Answer:** Denormalization is the practice of intentionally introducing redundant data or grouping tables together to optimize query read performance by avoiding expensive relational join operations.

7. **What are ACID Properties?**
   
   **Answer:** ACID represents the key guarantees of a transaction:
   * **Atomicity:** All operations in a transaction succeed, or the entire transaction is rolled back.
   * **Consistency:** A transaction moves the database from one valid state to another, maintaining all constraints.
   * **Isolation:** Concurrent transactions execute without interfering with each other.
   * **Durability:** Once a transaction commits, the changes are permanently saved in non-volatile storage.

8. **What is a Transaction?**
   
   **Answer:** A transaction is a sequence of database operations executed as a single unit of work. It is treated as an all-or-nothing operation, satisfying the ACID properties.

9. **What is Data Consistency?**
   
   **Answer:** Consistency means that data conforms to all active rules, constraints, triggers, and foreign keys. In distributed systems, consistency also refers to all nodes seeing the same data at the same time.

10. **What is Data Integrity?**
   
    **Answer:** Data integrity refers to the overall accuracy, completeness, and reliability of data. It is maintained through constraints (like NOT NULL, UNIQUE, CHECK) and referential integrity (foreign keys).

---

# 8. SQL vs NoSQL

1. **What is SQL?**
   
   **Answer:** SQL (Structured Query Language) is the standard language used to interact with Relational Database Management Systems (RDBMS). Relational databases organize data into tables with pre-defined schemas and establish relationships between them.

2. **What is NoSQL?**
   
   **Answer:** NoSQL (Not Only SQL) databases are non-relational databases designed to store unstructured or semi-structured data. They use flexible, dynamic schemas and are built to scale horizontally.

3. **SQL vs NoSQL?**
   
   **Answer:** SQL databases are relational, use structured tables with rigid schemas, scale vertically, and guarantee ACID transactions. NoSQL databases are non-relational, have flexible schemas (document, key-value, graph, wide-column), scale horizontally, and trade strict consistency for availability and performance (often following the BASE model).

4. **When would you choose SQL?**
   
   **Answer:** I would choose SQL when data relations are complex and highly structured, the schema is stable, and strict ACID transaction compliance is required (e.g., financial ledgers, checkout flows).

5. **When would you choose NoSQL?**
   
   **Answer:** I would choose NoSQL when handling massive volumes of data, requiring horizontal scaling, storing unstructured or polymorphic data, or when fast write speeds are critical (e.g., real-time chats, user analytics, IoT telemetry).

6. **Why is SQL preferred for transactions?**
   
   **Answer:** Because RDBMS systems natively implement robust isolation levels (like Serializable) and strict write coordination locks to ensure that concurrent transactions do not corrupt shared state.

7. **Why is NoSQL preferred for scalability?**
   
   **Answer:** Because NoSQL databases are designed to partition data across independent nodes without relying on centralized coordination, which eliminates the need for expensive cross-network joins or distributed locking.

8. **What are examples of SQL databases?**
   
   **Answer:** PostgreSQL, MySQL, SQLite, Microsoft SQL Server, Oracle.

9. **What are examples of NoSQL databases?**
   
   **Answer:** MongoDB (Document), Cassandra (Wide-column), Redis (Key-value), Neo4j (Graph), DynamoDB (Key-value/Document).

10. **Can SQL and NoSQL be used together?**
   
    **Answer:** Yes, this is known as polyglot persistence. For example, in an e-commerce application, PostgreSQL could handle the transactional checkout system, while MongoDB handles the dynamic product catalog, and Redis stores user sessions.

---

# 9. Database Indexing

1. **What is an Index?**
   
   **Answer:** An index is a specialized data structure (most commonly a B-Tree or B+ Tree) that stores pointers to table rows, sorted by specific column values. It acts as a shortcut to bypass scanning the whole table.

2. **Why do we need indexing?**
   
   **Answer:** We need indexing to speed up data retrieval. Without indexes, the database must perform a sequential full table scan ($O(N)$ complexity) to find matching records, which becomes unacceptably slow as the table grows.

3. **How does indexing improve performance?**
   
   **Answer:** By reducing the search complexity from a linear scan ($O(N)$) to a logarithmic tree traversal ($O(\log N)$). The database engine can find the specific row keys in the tree and jump directly to the data blocks on disk.

4. **What is a B-Tree Index?**
   
   **Answer:** A B-Tree (Balanced Tree) is a self-balancing search tree structure that keeps keys in sorted order. This allows the database to perform fast lookups, range queries, inserts, and deletes in logarithmic time.

5. **What is a Composite Index?**
   
   **Answer:** A composite index is an index created on multiple columns of a table. The order of columns matters because the database can only search starting from the leftmost column in the index (left-to-right prefix rule).

6. **What is a Clustered Index?**
   
   **Answer:** A clustered index determines the physical order of data storage on disk. A table can have only one clustered index because data rows themselves can only be sorted in one order (usually the primary key).

7. **What is a Non-Clustered Index?**
   
   **Answer:** A non-clustered index is a separate structure containing index keys pointing to the physical locations of the data rows (such as pointers to the clustered index). A table can have multiple non-clustered indexes.

8. **What are the disadvantages of indexing?**
   
   **Answer:**
   * **Storage Overhead:** Indexes require disk space to store the tree nodes.
   * **Write Performance Penalty:** Every write operation (`INSERT`, `UPDATE`, `DELETE`) requires modifying both the table data and updating the index structures.

9. **Why do writes become slower with more indexes?**
   
   **Answer:** Whenever a row is written, the database engine must modify the B-Trees of every index associated with that table. This involves tree rebalancing, node splits, and additional random disk writes.

10. **How do you choose columns for indexing?**
   
    **Answer:** I index columns that are frequently used in `WHERE` clauses, `JOIN` conditions, or `ORDER BY` operations, prioritizing columns with high cardinality (many unique values) to ensure high search selectivity.

---

# 10. Replication

1. **What is Database Replication?**
   
   **Answer:** Database replication is the process of copying data from one database server (the primary) to one or more database servers (the replicas) to keep the dataset synchronized across multiple instances.

2. **Why do we need Replication?**
   
   **Answer:** We use replication to:
   * Offload read traffic from the primary node to read replicas (scaling reads).
   * Ensure high availability (if the primary crashes, a replica can be promoted).
   * Decrease latency by placing read replicas closer to users geographically.
   * Run heavy analytical queries or backups without impacting write throughput.

3. **What is a Primary Database?**
   
   **Answer:** The primary database is the master node in a replication setup. It accepts write operations, logs the changes (e.g., WAL log), and broadcasts them to the read replicas.

4. **What is a Read Replica?**
   
   **Answer:** A read replica is a read-only copy of the primary database. It receives replication updates from the primary and processes read queries for clients.

5. **How does replication improve availability?**
   
   **Answer:** If the primary database goes offline, we can promote one of the healthy read replicas to act as the new primary, allowing the system to resume write operations with minimal downtime.

6. **What is Replication Lag?**
   
   **Answer:** Replication lag is the delay between a write operation being committed on the primary database and that write being applied and made visible on a replica.

7. **What happens when the Primary DB fails?**
   
   **Answer:** An automatic or manual failover occurs. A consensus mechanism (or monitoring tool) detects the failure, promotes a healthy replica with the least replication lag to be the new primary, and updates client connection routing.

8. **Synchronous vs Asynchronous Replication?**
   
   **Answer:**
   * **Synchronous:** The primary commits the write and blocks until at least one replica acknowledges receiving it. This prevents data loss on failover but increases write latency.
   * **Asynchronous:** The primary commits the write immediately and acknowledges the client. The data is replicated to replicas in the background. This is faster but risks data loss if the primary crashes before the replica receives the updates.

9. **Can replicas handle writes?**
   
   **Answer:** Typically, replicas are configured to be read-only to avoid split-brain and write conflicts. Multi-primary (active-active) replication allows multiple nodes to handle writes, but requires complex conflict resolution strategies.

10. **How do companies scale read-heavy systems?**
   
    **Answer:** By deploying a pool of read replicas behind a load balancer and utilizing caching layers (like Redis or Memcached) to intercept reads before they ever reach the databases.

---

# 11. Sharding

1. **What is Sharding?**
   
   **Answer:** Sharding is a database architecture pattern where database tables are split horizontally across multiple independent database servers (shards), with each server holding a subset of the rows.

2. **Why do we need Sharding?**
   
   **Answer:** Sharding is necessary when a dataset is too large to fit on a single physical machine, or when the write QPS exceeds the capacity of a single database primary node, making horizontal scaling at the storage tier a requirement.

3. **What is a Shard Key?**
   
   **Answer:** A shard key is a column (or columns) in the table that determines which shard stores a given row. The database runs a routing function (such as hashing) on the shard key value to locate the target shard.

4. **How do you choose a Shard Key?**
   
   **Answer:** I look for a key that:
   * Distributes writes and reads evenly across all shards (to avoid hotspots).
   * Frequently appears in query filters, so we can route requests directly to a single shard (preventing cross-shard broadcast queries).
   * Typically, keys like `user_id` or `tenant_id` are good candidates.

5. **What is Hot Sharding?**
   
   **Answer:** Hot sharding (or a hot spot) occurs when a disproportionate amount of read/write requests target a single shard. This happens when the shard key is poorly chosen (e.g., sharding by date or celebrity ID), overloading that machine while other shards sit idle.

6. **What is Re-Sharding?**
   
   **Answer:** Re-sharding is the process of altering the shard layout—either by splitting existing shards or adding new shard servers—and redistributing the database rows to fit the new partition structure.

7. **What happens when one shard becomes overloaded?**
   
   **Answer:** It leads to latency spikes, query queue timeouts, or connection failures for users whose data resides on that shard. It requires splitting the overloaded shard or migrating data to new shards.

8. **Sharding vs Replication?**
   
   **Answer:**
   * **Replication** duplicates the *same* dataset across multiple nodes to scale reads and ensure high availability.
   * **Sharding** splits a dataset into *different* subsets across nodes to scale write capacity and database storage limits.

9. **What are the challenges of sharding?**
   
   **Answer:**
   * **Complex Queries:** Joins across shards are extremely slow and complex to orchestrate.
   * **Referential Integrity:** Enforcing foreign key constraints across shards is not supported by databases natively.
   * **Re-sharding:** Migrating data live without downtime is a massive engineering effort.

10. **How do large systems rebalance shards?**
   
    **Answer:** They use consistent hashing algorithms or a central configuration directory service (like ZooKeeper) that tracks shard mappings, allowing them to migrate subsets of hash ranges with minimal data movement.

---

# 12. Consistent Hashing

1. **What problem does Consistent Hashing solve?**
   
   **Answer:** It solves the massive data-migration problem in distributed caching and storage systems. With standard modulo hashing ($hash(key) \pmod N$), adding or removing a node changes the destination of almost all keys, triggering global cache misses or massive re-indexing.

2. **How does Consistent Hashing work?**
   
   **Answer:** It maps both servers and data keys onto a circular ring using a hash function. A key is assigned to the first server it encounters when traversing the ring in a clockwise direction.

3. **What is a Hash Ring?**
   
   **Answer:** A hash ring is a representation of the output range of a hash function (e.g., from 0 to $2^{32} - 1$) connected in a circle. Both node identifiers and keys are mapped to positions on this circle.

4. **What are Virtual Nodes?**
   
   **Answer:** Virtual nodes (vnodes) map a single physical server to multiple locations on the hash ring. This ensures a more uniform distribution of keys across physical nodes and prevents hot spots on any single server.

5. **What happens when a new server is added?**
   
   **Answer:** Only a fraction of the keys ($1/N$, where $N$ is the number of nodes) are moved. Specifically, the new node only takes ownership of keys that lie between it and its counter-clockwise neighbor.

6. **What happens when a server is removed?**
   
   **Answer:** Only the keys that were stored on the removed server need to be reassigned to the next clockwise server. The keys mapped to all other servers remain completely unaffected.

7. **Why is Consistent Hashing better than Modulo Hashing?**
   
   **Answer:** Modulo hashing ($key \pmod N$) forces the remapping of nearly $100\%$ of keys when $N$ changes. Consistent hashing minimizes key movement to $1/N$ of keys, preventing massive system-wide database stampedes.

8. **Where is Consistent Hashing used?**
   
   **Answer:** In distributed databases like Amazon DynamoDB, Apache Cassandra, and Voldemort, as well as distributed caches like Redis Cluster and Memcached.

9. **How does it help Redis Clusters?**
   
   **Answer:** It allows Redis nodes to be dynamically scaled out or scaled in during runtime, automatically re-routing keys to the new nodes without significant impact on system availability.

10. **How does it reduce data movement?**
   
    **Answer:** By localizing key ownership changes. When a node is modified, only the immediate adjacent range of the hash ring is affected; all other segments of the circle remain routed to the exact same servers.

---

# 13. Storage Systems

1. **What is Object Storage?**
   
   **Answer:** Object storage organizes data as distinct, flat objects containing the raw payload, user-defined custom metadata, and a unique identifier. It is accessed via HTTP/HTTPS APIs and is designed for massive scale-out of unstructured data.

2. **What is Block Storage?**
   
   **Answer:** Block storage splits files into raw, fixed-size blocks, each with its own address, without any metadata or file structure. Accessed by the operating system over protocols like iSCSI or Fibre Channel, it behaves like a physical hard drive and is optimized for low-latency databases and VMs.

3. **What is File Storage?**
   
   **Answer:** File storage stores data in a hierarchical directory tree structure of files and folders (e.g., NFS, SMB). It is easy to use for shared local access but becomes a performance bottleneck at high volumes.

4. **When should Object Storage be used?**
   
   **Answer:** It should be used for storing unstructured, read-heavy data that is written once and read many times, such as user-uploaded images, videos, logs, static assets, and massive system backups.

5. **Why are images stored in Object Storage?**
   
   **Answer:** Because images are static, unstructured binary blobs. Storing them in object storage is highly cost-effective, allows direct retrieval via CDN edges, and keeps the database lean.

6. **What is a Blob?**
   
   **Answer:** A Blob (Binary Large Object) is a collection of binary data stored as a single entity, typically used to refer to images, videos, audio, or compiled executable files.

7. **Why not store large files directly in a database?**
   
   **Answer:** Storing large files in databases causes severe database bloat, decreases backup speeds, consumes precious database RAM (which should be used for indexing), and increases write lock contention.

8. **How does Object Storage scale?**
   
   **Answer:** It separates data storage from the metadata index service. The system scales horizontally by spreading raw object files across vast clusters of commodity storage nodes while using a distributed directory service for index resolution.

9. **What is the difference between metadata and file data?**
   
   **Answer:** File data is the raw binary content itself (e.g., the actual video file). Metadata is structural information about the file (e.g., content-type, creation date, user ID, resolution) used for search and access control.

10. **Examples of popular Object Storage systems?**
   
    **Answer:** Amazon S3, Google Cloud Storage, Azure Blob Storage, MinIO.

---

# 14. Message Queues

1. **What is a Message Queue?**
   
   **Answer:** A Message Queue (MQ) is an asynchronous, point-to-point communication channel that allows different software components to send and receive messages without being directly connected or online simultaneously.

2. **Why do we need Message Queues?**
   
   **Answer:** We need message queues to:
   * Decouple microservices, preventing failure propagation.
   * Smooth out traffic spikes (rate leveling) by buffering excess requests.
   * Enable asynchronous execution of time-consuming processes.
   * Improve overall system fault tolerance.

3. **What is Asynchronous Processing?**
   
   **Answer:** A design pattern where a service triggers a task (e.g., generating a PDF report) and immediately returns a success status back to the client, while a separate background worker process performs the actual task in the background.

4. **Message Queue vs Direct API Call?**
   
   **Answer:** A direct API call is synchronous; if the receiver is down or slow, the caller blocks or fails. A message queue is asynchronous; the sender drops the message in the queue and continues. The receiver consumes it when available, ensuring the request succeeds even if downstream systems are temporarily offline.

5. **What is Producer and Consumer?**
   
   **Answer:**
   * **Producer:** The application component that creates messages and writes them to the queue.
   * **Consumer:** The application component that reads messages from the queue and executes the corresponding tasks.

6. **What happens when a consumer is down?**
   
   **Answer:** The messages remain safely buffered in the queue. When the consumer comes back online, it reads the messages from the queue, preventing any data loss.

7. **What is a Dead Letter Queue?**
   
   **Answer:** A Dead Letter Queue (DLQ) is a dedicated queue where messages that fail to be processed successfully after a pre-defined number of retry attempts are sent. This isolates them for manual inspection and debugging without blocking the main queue.

8. **How do retries work in MQ systems?**
   
   **Answer:** If a consumer fails to process a message, the consumer returns a negative acknowledgment (NACK). The queue system then re-queues the message, often applying a retry count limit and an exponential backoff delay before offering it to consumers again.

9. **What are the advantages of decoupling services?**
   
   **Answer:**
   * Services can scale independently based on their resource bottlenecks (CPU vs Memory).
   * Fault isolation: a bug in one service does not crash the entire application flow.
   * Development flexibility: teams can write different services in different languages.

10. **Examples of Message Queue systems?**
   
    **Answer:** RabbitMQ, Apache ActiveMQ, Amazon SQS, IronMQ.

---

# 15. Kafka

1. **What is Kafka?**
   
   **Answer:** Apache Kafka is a distributed event streaming platform designed to handle high-throughput, low-latency, real-time data feeds. It functions as an append-only distributed log.

2. **Why do we use Kafka?**
   
   **Answer:** We use Kafka when we need massive real-time throughput (millions of events per second), persistent event logs that can be replayed, real-time stream processing, or central event-sourcing hubs.

3. **What is a Topic?**
   
   **Answer:** A Topic is a logical category or feed name to which messages are published. It acts like a table in a database or a folder in a file system.

4. **What is a Partition?**
   
   **Answer:** A partition is a single physical append-only log file belonging to a topic. Topics are split into multiple partitions distributed across Kafka brokers to allow parallel writes and reads.

5. **What is a Consumer Group?**
   
   **Answer:** A consumer group is a set of consumers that cooperate to read messages from a topic. Kafka ensures that each partition in a topic is consumed by only one member of the group, enabling parallel processing without duplicate consumption.

6. **Why does Kafka guarantee ordering only within a partition?**
   
   **Answer:** Because maintaining a global ordering of messages across multiple partitions on different physical brokers would require complex cross-node locking, which would severely degrade Kafka's throughput and scalability.

7. **What is an Offset?**
   
   **Answer:** An offset is a unique, sequential integer identifier assigned to each message as it is appended to a partition. It acts as the local cursor position for consumers reading that partition.

8. **What is Log Compaction?**
   
   **Answer:** Log compaction is a retention policy that ensures Kafka retains at least the last known value for each key within a partition log. It removes older records with duplicate keys to save space.

9. **What is Kafka Retention?**
   
   **Answer:** Kafka retention determines how long messages are kept in the log before being automatically deleted. This can be configured based on time (e.g., 7 days) or total storage size (e.g., 50 GB).

10. **Kafka vs RabbitMQ?**
   
    **Answer:**
    * **Kafka** is a distributed log; it uses a pull model where clients track their own offsets, enabling message replay and massive event throughput.
    * **RabbitMQ** is a traditional message broker; it uses a push model with complex routing logic, where messages are typically deleted immediately after the consumer acknowledges them.
