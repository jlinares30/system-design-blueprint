# ◇ SQL vs. NoSQL: Data Layer Architecture

## ▪ Comparison Matrix

| Criteria | Relational Databases (SQL) | Non-Relational Databases (NoSQL) |
| :--- | :--- | :--- |
| **Schema** | Strict, predefined, and structured (Tables with rows and columns). | Flexible, dynamic schema (JSON, key-value, column-family, graph). |
| **Relationships** | Native and optimized support for complex JOINS. | JOINS are not natively optimized; data is denormalized or queried separately. |
| **Scaling** | Primarily Vertical. Horizontal scaling (Sharding/Replication) is complex. | Designed to scale horizontally out-of-the-box using native partitioning. |
| **Transactions** | Strict **ACID** guarantees. | Flexible **BASE** guarantees (Eventual Consistency). |
| **Examples** | PostgreSQL, MySQL, Oracle, SQL Server. | MongoDB, Cassandra, Redis, Neo4j, DynamoDB. |

---

## ▪ Transactional Models

### ACID (Traditional SQL)
*   **Atomicity:** All-or-nothing. If any operation within a transaction fails, the entire transaction is rolled back.
*   **Consistency:** A transaction changes the database from one valid state to another, maintaining constraints and rules.
*   **Isolation:** Concurrent transactions execute without interfering with each other.
*   **Durability:** Once committed, transaction updates persist even in the event of system failures.

### BASE (Distributed NoSQL)
*   **Basically Available:** The system guarantees availability; every request receives a response (even if data is not immediately consistent).
*   **Soft State:** The state of the data can drift over time without active user interaction due to lazy replication.
*   **Eventual Consistency:** The system will eventually become consistent; once updates stop, all replicas will converge to the same state.

---

## ▪ NoSQL Database Categories

1. **Document Stores:** Stores data in semi-structured document formats (JSON/BSON).
    *   *Examples:* MongoDB, CouchDB.
    *   *Use Cases:* E-commerce catalogs, user profiles, content management.
2. **Key-Value Stores:** Highly optimized hash-table-like storage where unique keys map to binary or string values.
    *   *Examples:* Redis, Memcached, DynamoDB (basic configuration).
    *   *Use Cases:* Caching, session management, shopping carts.
3. **Wide-Column Stores:** Stores data in columns instead of rows, optimized for massive aggregations and queries on specific columns.
    *   *Examples:* Apache Cassandra, ScyllaDB, HBase.
    *   *Use Cases:* Time-series logs, telemetry data, audit trails.
4. **Graph Databases:** Represents data as nodes, properties, and edges (relationships).
    *   *Examples:* Neo4j, Amazon Neptune.
    *   *Use Cases:* Recommendation engines, social network mapping, fraud detection.

---

## ▪ Key Architectural Considerations

*   **Financial Transaction Ledgers:** Choose **SQL (e.g., PostgreSQL)** because strict consistency (ACID) is non-negotiable. Balances cannot temporarily drift or show inconsistencies between accounts.
*   **High-Throughput Social Feeds:** Choose **NoSQL (e.g., Cassandra or DynamoDB)** where rapid write/read capabilities and horizontal scalability take priority over immediate consistency.
