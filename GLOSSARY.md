# // Repository Technical Glossary

Welcome to the centralized glossary. These terms are linked across the architecture specifications.

---

## :: A

### ACID
A set of database properties ensuring reliable transactions: Atomicity (all-or-nothing), Consistency (preserves rules), Isolation (independent runs), and Durability (survives crashes).

### Aggregate
A cluster of domain objects (Entities and Value Objects) in Domain-Driven Design that can be treated as a single transactional unit.

### API Gateway
A reverse proxy boundary server acting as a single entry point for routing, rate limiting, and securing incoming requests to internal services.

---

## :: B

### BASE
A transactional model for distributed systems prioritizing availability: Basically Available, Soft state (data floats), and Eventual consistency.

### Bloom Filter
A space-efficient probabilistic data structure used to test whether an element is a member of a set, preventing unnecessary backend lookups.

### Bounded Context
A logical boundary wrapping a specific domain model in Domain-Driven Design, ensuring terminology meanings remain isolated.

---

## :: C

### Cache Avalanche
An outage occurring when multiple cached items expire simultaneously, triggering a massive surge of concurrent queries directly to databases.

### Cache Stampede
An issue where a highly popular cache key expires, leading to many application threads attempting to recalculate and write it at the same time.

### Cache-Aside
A caching pattern where the application reads from the cache first, falling back to the database on a cache miss and updating the cache afterwards.

### CAP Theorem
A theorem stating that a distributed data system can guarantee at most two of three properties simultaneously: Consistency, Availability, and Partition Tolerance.

### CDN (Content Delivery Network)
A geographically distributed group of servers caching static assets close to end users.

### Circuit Breaker
A resilience pattern that fails requests immediately when error rates cross a threshold, giving struggling services time to recover.

### Clean Architecture
A software design pattern organizing code in concentric layers, enforcing that dependencies only point inwards toward the core business logic.

### Cold Start
The initialization delay occurring when a serverless function is triggered after a period of inactivity.

### Consistent Hashing
A sharding distribution algorithm using a circular hash ring, minimizing data relocation when servers are added or removed.

---

## :: D

### Database-per-Service
A microservice pattern where each service owns its data storage exclusively, exposing data only via public APIs.

### Delta / Parquet Format
Optimized, compressed columnar storage file formats designed to handle massive data read operations in analytical pipelines.

### DHT (Distributed Hash Table)
A decentralized storage system providing lookup services similar to a hash table without central servers.

### DirectQuery
A data visualization connection mode querying databases in real time instead of importing data into local RAM.

### Domain Event
An event capturing a state change of significance within the business domain model.

---

## :: E

### Edge Cache
A cache positioned at the network boundary near clients to return payloads without traversing to application backends.

### Entity
An object defined by a unique identity that persists across state updates, rather than by its attribute values.

### ERP (Enterprise Resource Planning)
Integrated software platforms used to manage day-to-day operations like finance, manufacturing, and supply chains.

### ESB (Enterprise Service Bus)
A centralized middleware architecture routing, transforming, and orchestrating messages between enterprise systems.

### Event-Driven Architecture (EDA)
A software model where services communicate asynchronously by producing, detecting, and consuming events.

---

## :: G

### gRPC
A high-performance remote procedure call framework running over HTTP/2, using binary Protocol Buffers for message serialization.

---

## :: H

### Hexagonal Architecture
A structural design pattern isolating core domain logic from external dependencies using interfaces called Ports and implementation Adapters.

### Horizontal Scaling (Scale-Out)
Expanding capacity by adding more standard servers to the cluster pool.

### HTTP
The application-layer protocol powering web requests, with newer versions optimizing multiplexing (HTTP/2) and connection handshakes (HTTP/3).

---

## :: I

### Idempotency
A property of an API or message consumer ensuring that processing the same request multiple times yields the same state outcome.

---

## :: J

### JWT (JSON Web Token)
A compact, cryptographically signed token standard used for securely transmitting claims between web clients and gateways.

---

## :: K

### Kafka
A distributed, persistent event log platform designed for real-time streaming pipelines.

### Key Generation Service (KGS)
A dedicated microservice pre-generating unique numerical sequence ranges to avoid key collision queries during data insertions.

### Kubernetes (K8s)
An open-source container orchestration engine managing container deployments, load balancing, and health cycles.

---

## :: L

### Lakehouse
A modern storage architecture merging raw data file storage (Data Lake) with relational ACID transaction reliability (Data Warehouse).

### Load Balancer
A traffic router distributing user connections across a group of application servers.

---

## :: M

### Medallion Architecture
A data curation pattern where data is progressively validated and structured as it flows through Bronze, Silver, and Gold layers.

### Microservices
An architectural style structuring applications as a collection of small, independent, and autonomous services.

### Monolithic Architecture
A traditional software design packaging the user interface, business rules, and database adapters as a single deployment unit.

---

## :: P

### PACELC
An extension of the CAP theorem stating that if there is a **P**artition, a system trades off **A**vailability vs. **C**onsistency; **E**lse, it trades off **L**atency vs. **C**onsistency.

---

## :: R

### RabbitMQ
A traditional AMQP-based message broker facilitating routing and workers queue distribution.

### Rate Limiting
A security pattern restricting the maximum requests a user can make within a specified window.

### Replication
Copying the same data across multiple database servers to guarantee high availability.

---

## :: S

### Saga Pattern
A design pattern coordinating distributed transactions across multiple microservices using a sequence of local steps and compensating tasks.

### Serverless (FaaS)
A cloud execution model running code on demand in ephemeral containers managed dynamically by the cloud vendor.

### Sharding
Partitioning database rows horizontally across multiple independent servers using a partition key.

### SOA (Service-Oriented Architecture)
An enterprise architectural pattern exposing business functions as reusable services through a centralized communication bus.

### Star Schema
A database modeling structure featuring a central transactional Fact Table surrounded by descriptive Dimension Tables.

---

## :: T

### TCP
A connection-oriented transport protocol guaranteeing packet ordering and transmission safety.

---

## :: U

### Ubiquitous Language
A strict common vocabulary shared by developers and domain experts to eliminate alignment discrepancies.

### UDP
A connectionless transport protocol prioritizing speed over delivery verification.

---

## :: V

### Value Object
An immutable object defined entirely by its structural values, having no unique conceptual identity.

### Vertical Scaling (Scale-Up)
Increasing capacity by adding CPU, RAM, or storage to a single server.

---

## :: W

### Write-Behind
A write pattern where application writes hit the cache first, flushing updates to the database asynchronously.

### Write-Through
A write pattern where writes update both the cache and the primary database in a single synchronous operation.
