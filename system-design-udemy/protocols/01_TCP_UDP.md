---
status: researching
tags: [system-design, udemy, research, networking, tcp, udp]
created: 2026-07-02
updated: 2026-07-02
---

# 01 TCP and UDP

> [!abstract] TL;DR
> TCP is a connection-oriented, reliable transport protocol that guarantees ordered delivery through handshakes, sequence numbers, acknowledgments, and retransmissions. UDP is a connectionless, best-effort protocol optimized for speed and low latency, leaving reliability to the application. Choosing between them is a trade-off between reliability and performance.

---

## TCP (Transmission Control Protocol)

### What is TCP?
TCP is a connection-oriented transport-layer protocol that provides reliable, ordered, and error-checked delivery of a byte stream between applications.

### Three-Way Handshake
Before data is exchanged, TCP establishes a connection using a three-step handshake:
1. **SYN** — Client sends a SYN (synchronize) segment to the server.
2. **SYN-ACK** — Server responds with SYN-ACK (synchronize-acknowledge) and its own initial sequence number.
3. **ACK** — Client sends an ACK (acknowledge) to confirm the connection is established.

### Tracking Data
- Each byte is assigned a **sequence number**.
- The receiver sends back **ACK numbers** indicating the next expected byte.
- Senders maintain a **send buffer** and track unacknowledged segments.

### Ordering and Assembling
- TCP segments may arrive out of order due to network routing.
- The receiver uses sequence numbers to reassemble segments into the correct order.
- Out-of-order segments are buffered until the missing segment arrives.

### Reliability of TCP
- **Acknowledgments**: Receiver confirms receipt.
- **Retransmissions**: If an ACK is not received within a timeout, the sender retransmits.
- **Checksums**: Detect corrupted data.
- **Flow control**: Sliding window prevents overwhelming the receiver.
- **Congestion control**: Adjusts transmission rate based on network conditions.

### Trade-Offs
- **Pros**: Guaranteed delivery, ordering, error recovery, congestion avoidance.
- **Cons**: Higher latency (handshake + acknowledgments), higher overhead, slower throughput for small messages.

### TCP Use Cases
- Web browsing (HTTP/HTTPS)
- Email (SMTP, IMAP, POP3)
- File transfer (FTP, SFTP)
- Databases (PostgreSQL, MySQL)
- Any scenario where data integrity matters more than raw speed.

---

## UDP (User Datagram Protocol)

### What is UDP?
UDP is a connectionless transport-layer protocol that sends self-contained datagrams with minimal overhead and no guarantees.

### Goal of UDP
UDP's primary goal is to provide a simple, fast, and low-latency transport mechanism for messages where occasional loss is acceptable.

### Connection Establishment
UDP does **not** establish a connection. It sends datagrams directly to the destination without handshakes, teardown, or state maintenance. This makes it stateless and efficient.

### Advantages
- Low latency (no handshake).
- Minimal overhead (8-byte header vs. 20+ bytes for TCP).
- Supports multicast and broadcast.
- Efficient for small, frequent messages.

### Trade-Offs
- **Pros**: Fast, simple, scalable for one-to-many.
- **Cons**: No guaranteed delivery, no ordering, no built-in congestion control, no retransmission.

### UDP Use Cases
- DNS (most queries, with TCP fallback for large responses)
- VoIP and video conferencing
- Online gaming
- Live streaming
- IoT telemetry
- SNMP, DHCP

---

## TCP vs UDP Comparison Table

| Feature | TCP | UDP |
|---------|-----|-----|
| **Connection** | Connection-oriented (handshake) | Connectionless |
| **Reliability** | Guaranteed delivery | Best-effort, no guarantee |
| **Ordering** | Ordered by sequence numbers | No ordering |
| **Speed / Latency** | Slower due to overhead and handshake | Faster, lower latency |
| **Error Handling** | Retransmissions + checksums | Checksum only; corrupt packets dropped |
| **Overhead** | Higher (20–60 byte header) | Lower (8 byte header) |
| **Flow Control** | Yes (sliding window) | No |
| **Congestion Control** | Yes | No |
| **Packet Model** | Byte stream | Fixed-size datagrams |
| **Use Case Fit** | Accuracy over speed | Speed over accuracy |

---

## Use Cases Table

| Use Case | Preferred Protocol | Reason |
|----------|-------------------|--------|
| Web browsing / APIs | TCP | Data integrity, request-response |
| Email | TCP | Reliable delivery, no tolerance for loss |
| File transfer | TCP | Complete, ordered file reconstruction |
| DNS lookup | UDP (TCP fallback) | Fast, small queries |
| Voice / Video calls | UDP | Low latency, loss-tolerant |
| Online gaming | UDP | Real-time state updates |
| Live streaming | UDP | Reduced buffering |
| IoT / Telemetry | UDP (or TCP) | Frequent small messages |
| VPN tunneling | TCP or UDP | Depends on protocol (e.g., WireGuard uses UDP) |

---

## Related Notes
- [[IP Addresses]]
- [[DNS Deep Dive]]
- [[HTTP vs WebSocket]]

---

## Status History
- `2026-07-02`: researching — extracted TCP/UDP concepts, comparisons, and use cases.
