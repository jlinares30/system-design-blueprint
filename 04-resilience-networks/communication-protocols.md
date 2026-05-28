# ◇ Communication Protocols

Selecting the right protocol governs performance, latencies, and connectivity patterns between clients and servers, as well as internal microservice-to-microservice traffic.

---

## ▪ Transport Layer: TCP vs. UDP

| Feature | Transmission Control Protocol (TCP) | User Datagram Protocol (UDP) |
| :--- | :--- | :--- |
| **Connection** | Connection-oriented (requires 3-way handshake). | Connectionless (sends packets without verifying target state). |
| **Reliability** | Guaranteed. Retransmits lost packets and ensures in-order delivery. | Unreliable. Packets may be lost, duplicated, or arrive out of order. |
| **Flow Control** | Yes. Regulates transmission rates based on network congestion. | No. Broadcasts packets as fast as bandwidth allows. |
| **Speed** | Slower (due to larger headers, handshakes, and ACK checks). | Extremely fast and lightweight. |
| **Common Uses** | Web browsing (HTTP), Email (SMTP), File transfer (FTP). | Video streaming, online gaming, DNS queries, VoIP. |

---

## ▪ Application Layer: HTTP vs. WebSockets vs. gRPC

### 1. HTTP (HyperText Request-Response)
*   **Model:** Client-initiated Request-Response pattern.
*   **HTTP/1.1:** Utilizes persistent connections (`Keep-Alive`), but suffers from *Head-of-Line (HoL) Blocking* (slow requests block subsequent ones on the same connection).
*   **HTTP/2:** Introduces multiplexing (concurrent streams over a single TCP connection) and HPACK header compression.
*   **HTTP/3:** Replaces TCP with the **QUIC** protocol (running over UDP), eliminating transport-layer HoL blocking and optimizing reconnection times for mobile clients.

### 2. WebSockets
*   **Model:** Persistent, bidirectional, full-duplex communication channel established over a single TCP connection.
*   **Mechanism:** Starts as an HTTP handshake request, upgrading to the WebSocket protocol.
*   **Use Cases:** Real-time chat messaging, financial ticker updates, collaborative canvases, and live dashboard tracking.

### 3. gRPC (Google Remote Procedure Call)
*   **Model:** High-performance, open-source RPC framework running over **HTTP/2**.
*   **Serialization:** Uses **Protocol Buffers (Protobuf)** instead of JSON. Translates data structures into highly compressed binary payloads.
*   **Streaming Support:** Supports client-streaming, server-streaming, and bidirectional streaming.
*   **Use Cases:** Internal communication between services within microservice architectures.

---

## ▪ Key Architectural Considerations

*   **Protobuf vs. JSON serialization:** Protobuf reduces payload sizes and eliminates CPU serialization overhead compared to parsing raw JSON strings. It enforces strong API contracts using `.proto` interface definitions, reducing integration bugs in distributed environments.
