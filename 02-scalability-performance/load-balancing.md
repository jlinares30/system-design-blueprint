# ◇ Load Balancing

Load balancers distribute incoming network traffic across a pool of servers, ensuring high availability, performance, and resource optimization.

---

## ▪ Load Balancing Algorithms

Load balancers use predefined algorithms to distribute incoming requests:

*   **Round Robin:** Distributes requests sequentially across the list of servers.
    *   *Weighted Round Robin:* Assigns traffic weights based on server capacities (e.g., a server with double the CPU capacity receives twice as many requests).
*   **Least Connections:** Routes traffic to the server with the fewest active connections. Ideal for tasks with highly variable execution times.
*   **IP Hash:** Computes a hash of the client's IP address to map them to a specific server. This ensures session persistence (*sticky sessions*).
*   **Least Response Time:** Sends requests to the server with the fastest response time and fewest active connections.

---

## ▪ Layer 4 (L4) vs. Layer 7 (L7) Load Balancing

Load balancers operate at different levels of the OSI network model:

| Feature | Layer 4 (L4) Load Balancing | Layer 7 (L7) Load Balancing |
| :--- | :--- | :--- |
| **OSI Layer** | Transport (TCP/UDP) | Application (HTTP/HTTPS/gRPC) |
| **Routing Context** | IP addresses and Ports. | IP, Port, HTTP headers, cookies, URL paths, message payload. |
| **Performance** | Extremely high throughput and low latency (no packet decryption/inspection). | Higher processing overhead (requires SSL/TLS decryption and payload inspection). |
| **Flexibility** | Limited. Simple port forwarding. | High. Enables path-based routing (e.g., `/api/v1/users` to Service A, `/static/` to Service B). |
| **Common Tools** | HAProxy (TCP mode), NGINX (stream), AWS NLB (Network Load Balancer). | HAProxy (HTTP mode), NGINX, Envoy, AWS ALB (Application Load Balancer). |

---

## ▪ Key Architectural Considerations

*   **Eliminating Load Balancer SPOFs:** Since the load balancer is the gateway to the system, it must not become a single point of failure. Deploy redundant load balancers in an **Active-Passive** or **Active-Active** configuration. Using **DNS Round Robin** combined with **keepalived / VRRP** (Virtual Router Redundancy Protocol) allows a virtual IP (VIP) to failover instantly to a passive load balancer if the primary node goes offline.
