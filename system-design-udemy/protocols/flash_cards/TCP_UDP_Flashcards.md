---
status: researching
tags: [system-design, udemy, flashcards, tcp, udp]
created: 2026-07-02
updated: 2026-07-02
---

# TCP/UDP Flashcards

## Basic Questions

What is the difference between TCP and UDP?
?
TCP is connection-oriented, reliable, and ordered; it uses handshakes, sequence numbers, and acknowledgments. UDP is connectionless, best-effort, and unordered; it sends datagrams with minimal overhead and no delivery guarantees.
#flashcards/system-design

Why is TCP considered a reliable protocol?
?
TCP guarantees delivery, ordering, and error-free data through sequence numbers, acknowledgments, retransmissions, checksums, flow control, and congestion control.
#flashcards/system-design

When would you choose UDP over TCP?
?
Choose UDP when low latency, speed, and small message overhead matter more than guaranteed delivery, such as in gaming, VoIP, live streaming, DNS, and IoT telemetry.
#flashcards/system-design

## Technical and Deep-Dive

How does TCP ensure reliable data transmission?
?
TCP uses sequence numbers to track bytes, acknowledgments to confirm receipt, retransmission timers for lost data, checksums to detect corruption, and flow/congestion control to manage sending rates.
#flashcards/system-design

What is the three-way handshake in TCP?
?
A three-step connection setup: SYN from client, SYN-ACK from server, ACK from client. It synchronizes sequence numbers and establishes a reliable connection.
#flashcards/system-design

How does UDP handle packet loss?
?
UDP does not handle packet loss itself. It provides best-effort delivery; the application must implement retries, FEC, or accept loss if needed.
#flashcards/system-design

## Use Case and System Design

Which protocol would you use for a real-time multiplayer game? Why?
?
UDP, because low latency is critical and occasional packet loss is acceptable; state updates can be sent continuously without waiting for handshakes or retransmissions.
#flashcards/system-design

How does TCP handle congestion control?
?
TCP monitors network conditions using mechanisms like slow start, congestion avoidance, fast retransmit, and fast recovery to adjust the sending rate and prevent network collapse.
#flashcards/system-design

Why is DNS implemented over UDP instead of TCP?
?
DNS queries are typically small and require fast response times; UDP's connectionless nature avoids handshake overhead. TCP is used as a fallback for large responses (e.g., zone transfers or DNSSEC).
#flashcards/system-design

If a video streaming service is buffering too much, would switching from TCP to UDP help?
?
Switching to UDP can reduce latency and buffering because it removes retransmissions and head-of-line blocking, but it requires the application to handle packet loss, reordering, and quality degradation.
#flashcards/system-design
