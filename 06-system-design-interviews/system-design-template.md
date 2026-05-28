# ◇ Large-Scale System Design Template

This template provides a step-by-step structural engineering framework to design, estimate, and evaluate large-scale distributed systems.

---

## ▪ Proposed Design Schedule

| Phase | Target Duration | Objective |
| :--- | :--- | :--- |
| **1. Scoping & Requirements** | 15% of time | Define functional scope, non-functional constraints, and assumptions. |
| **2. Scale Estimation** | 10% of time | Calculate query volumes, storage footprints, and bandwidth requirements. |
| **3. High-Level Design** | 30% of time | Create architecture diagrams, component routing, and API definitions. |
| **4. Architectural Deep Dive** | 35% of time | Resolve bottleneck issues, sharding, caching layers, and reliability. |
| **5. Core Trade-offs Summary** | 10% of time | Conclude with alternate designs and compromises chosen. |

---

## ▪ Engineering Design Framework

### Step 1: Requirements Scoping
Clarify the boundaries of the system before proposing architectures.

*   **Functional Requirements:**
    *   [ ] What are the primary user workflows?
    *   [ ] What are the supported media formats (e.g., text, rich media)?
    *   [ ] Is real-time delivery required?
*   **Non-Functional Requirements:**
    *   [ ] Availability target (e.g., 99.99% uptime).
    *   [ ] Latency parameters (e.g., read latency < 100ms).
    *   [ ] Consistency constraints (e.g., strong consistency vs. eventual consistency).
*   **Out of Scope:**
    *   [ ] Identify systems or components that will not be addressed (e.g., third-party auth, payment gateways).

### Step 2: Scale Estimations (Back-of-the-envelope)
Perform quick sizing calculations to estimate data center requirements.

*   **Traffic Scale:**
    *   Active Users: `________` Daily Active Users (DAU).
    *   Average Queries Per Second (QPS) (Read/Write): `________`.
    *   Peak QPS: `________` (usually calculated as `2 * Average QPS`).
*   **Storage Requirements:**
    *   Average payload size: `________` bytes.
    *   Accumulated storage over 5 years: `________` Terabytes (TB) or Petabytes (PB).
*   **Bandwidth Capacity:**
    *   Incoming (Ingress): `________` MB/s.
    *   Outgoing (Egress): `________` MB/s.

### Step 3: High-Level Architecture
Outline the main structural blocks and interfaces.

*   **System Diagram:**
    *   Map connections from clients through edge proxies (CDNs, Load Balancers) to stateless microservices and stateful data tiers.
*   **API Interface Specifications:**
    *   Define core endpoints, HTTP methods, and payload structures.
    *   `POST /v1/resource` -> Returns `201 Created`
    *   `GET /v1/resource/{id}` -> Returns `200 OK`

### Step 4: Architectural Deep Dive
Analyze specific bottlenecks and scaling strategies.

*   **Focal Points:**
    *   **Data Tier Design:** Schema layouts, indexing strategies, replication patterns, and sharding algorithms.
    *   **Caching Layers:** Cache placement (CDN, application, distributed key-value), TTL heuristics, and invalidation mechanisms.
    *   **Resilience Patterns:** Incorporate message brokers, fallback fallbacks, rate limiters, and circuit breakers.

### Step 5: Trade-offs Analysis
Conclude by stating the compromises made and how they impact system behaviors under stress.

*   *Example:* "We prioritized availability (AP) for the comments service using Cassandra, accepting that comments may take a few seconds to propagate to all users during network partitions."
