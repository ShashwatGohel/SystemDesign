## Day-1 : What is System Design 
System Design is the process of designing the architecture, components, data flow, and interactions of a software system.
---
System Design = Planning how software works internally
---

System Design helps us understand how a software system should be built and how it will continue to work efficiently as the number of users and requests grows.

---
Components of a System : 
-Client : example Mobile App, Website, Desktop App
-Backend Server : handles the business logic , authentication, APIs
-Database : Storing of the data
-Cache : Store frequently accessed data
-Storage : stores images, Videos, Documents
-Load Balancer : Distribution of the Traffic
-Queue : Handle Backend tasks 

---

### High Level Design v/s Low Level Design : ##
1. High Level Design : 
    -Service
    -Databases
    -Caches
    -LoadBalancer
    -Queues
2. Low Level Design :
    -Classes
    -Objects
    -Interfaces
    -Patterns

---
### Steps For Answering System Design Question : 

1. Gather Requirements
2. Estimate Scale
3. Design APIs
4. Design Database
5. Design Architecture
6. Identify Bottlenecks
7. Improve System
8. Discuss Trade-offs

---
### Whenever we see a System ask : 
1. What are the Users Doing 
2. Where is the Data Stored
3. What becomes slow first 
4. How will it scale

### Real world Example : College Canteen System 
- User : Student 
- Client : Order Counter
- Backend : Workiers (Canteen Staff) 
- Database : Register Book
- Queue : pending Orders
- Load Balancer : Multiple Counters for Serving 

---
### Common Question for Interview 

1. What is System Design ?
System Design is the process of designing the architecture, components, data flow, and scalability of a software system.

2. Difference between HLD and LLD ? 
HLD deals with Services, Databas, Cache, Load Balancer, Queses, whereas LLD deals with the classes, Objects, Interfaces and the patterns.

3. What are the Major components of System Design ?
The major components of System Design are 
- Client 
- Backend
- Database
- Queue
- LoadBalancer
- Storage
- Cache

4. Design the URL Shortner ? 
- User: Url Creator, URL Visitor
- data : OriginalLink, TimeStamp,clickCount, shortCode
- APIs : POST to create a short link, GET api to open the short link 
- database : ONe Database is enough (MySQL, PostgreSQL, MongoDB) we need to store the Key - Value pair only.

## Important Point : 
### When the Question is about , what happens when the system grows from 100 users to 10Million Users : 
Potential Bottelnecks are: 
- Application Server
- Database 
- Network 
- Storage

Sample answer forNotes app, from 100 Users to 10M users : 
Initially, the application server may become the bottleneck because a single server can handle only a limited number of concurrent requests. As traffic increases further, the database usually becomes the biggest bottleneck due to heavy read and write operations. Eventually, network bandwidth and storage can also become bottlenecks.

---
### Why Do Companies Ask System Design?

- To evaluate architecture thinking
- To evaluate problem decomposition skills
- To evaluate scalability understanding
- To evaluate trade-off analysis
- To evaluate communication skills

---
### Difference Between System Design and Coding Interviews

Coding Interview:
- Focus on algorithms and data structures
- Requires writing code

System Design Interview:
- Focus on architecture and scalability
- Requires discussing components, bottlenecks, and trade-offs
- Usually no production code is required

---
### Key Takeaways

1. System Design is about designing scalable systems.
2. System Design is not diagram drawing.
3. Every design starts with requirement gathering.
4. Bottlenecks are the first thing to identify when a system grows.
5. Trade-offs are more important than perfect solutions.
6. HLD focuses on architecture, LLD focuses on implementation.


