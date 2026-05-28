# ◇ Serverless and Peer-to-Peer (P2P)

## ▪ Serverless Computing (FaaS)

Serverless is a cloud execution model where the cloud provider dynamically manages the allocation and provisioning of servers. Developers bundle code into ephemeral functions (Function-as-a-Service or FaaS) that run in response to specific triggers.

### Advantages
*   **No Infrastructure Management:** Zero OS patching, server provisioning, or infrastructure capacity planning.
*   **Elastic Auto-scaling:** Scales automatically from zero to thousands of concurrent executions.
*   **Granular Billing:** Charged purely by execution time in milliseconds and memory allocated, avoiding idle server costs.

### Disadvantages
*   **Cold Starts:** Initial latency when a function is invoked after being idle, as the platform provisions container resources and spins up runtime dependencies.
*   **Execution Time Limits:** Cloud providers impose maximum runtime execution constraints (e.g., AWS Lambda has a 15-minute timeout).
*   **Stateless Execution:** FaaS instances are transient. Any persistent state must be stored in external systems like caches or databases.

---

## ▪ Peer-to-Peer (P2P) Architectures

Unlike the traditional Client-Server architecture, a Peer-to-Peer (P2P) network consists of decentralized, self-organizing nodes (peers) that act as both clients and servers of resources, communicating directly.

```mermaid
graph TD
    subgraph Client-Server
        C1[Client 1] --> Server((Central Server))
        C2[Client 2] --> Server
        C3[Client 3] --> Server
    end

    subgraph Peer-to-Peer (P2P)
        P1[Peer A] <--> P2[Peer B]
        P2 <--> P3[Peer C]
        P3 <--> P1
    end
```

### Core P2P Concepts
1. **Distributed Hash Table (DHT):** A decentralized system providing a lookup service similar to a hash table. It maps keys (e.g., file hashes) to values (e.g., node IP addresses) without a central index.
2. **Resilience:** There is no Single Point of Failure (SPOF). The system continues operating even if several nodes disconnect.
3. **Use Cases:** File sharing (BitTorrent), blockchain protocols, peer-to-peer CDNs, and real-time communication protocols (WebRTC).

---

## ▪ Key Architectural Considerations

*   **Cold Start Mitigation:**
    *   **Keep-Alives:** Schedule periodic "ping" invocations or use provider features like Provisioned Concurrency.
    *   **Bundle Optimization:** Minimize package sizes (tree shaking, removing unnecessary dependencies).
    *   **Language Selection:** Prefer fast-booting environments (e.g., Node.js, Go) over heavier runtimes (e.g., Java, .NET) for latency-sensitive APIs.
