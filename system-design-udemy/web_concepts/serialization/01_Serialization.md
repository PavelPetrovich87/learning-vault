---
status: archived
tags: [system-design, udemy, web-concepts, serialization, research]
created: 2026-07-18
updated: 2026-10-21
---

# 01_Serialization

## Source
Udemy System Design Course — Web Concepts / Serialization section.
Companion source: `[[../Serialization_Detailed_Interview_Answers.pdf]]` (13 interview questions covering serialization fundamentals, format comparison, performance, caching/storage, and security).

## Key Concepts
- **Why serialization matters:** Distributed systems are made of independent processes on independent machines speaking different languages. The only thing that can travel between them is bytes — so any in-memory object must be flattened into a byte stream (and reconstructed on the other side) before it can be stored, cached, or sent over the network. See [[Serialization_Fundamentals]].
- **What is serialization:** Converting an object/data structure into a format that can be stored, transmitted, and reconstructed later. The inverse — reconstruction — is **deserialization**. The format choice drives size, speed, readability, and compatibility.
- **Common serialization formats:** JSON and XML (human-readable text), Protocol Buffers and Avro (binary, schema-driven), and BSON (binary JSON used by MongoDB). See [[Serialization_Formats]].
- **Trade-offs:** Readability (JSON/XML) vs efficiency (Protobuf/Avro) vs compatibility (schema-driven formats enable safe evolution). The format also drives CPU and memory cost because text formats are expensive to parse. See [[Serialization_Tradeoffs]].
- **Serialization in action — APIs:** REST uses JSON, gRPC uses Protocol Buffers, SOAP uses XML. The wire format is rarely arbitrary — it is chosen to match the protocol's constraints (streaming, schema enforcement, browser support). See [[Serialization_In_APIs]].
- **Serialization in caching & data storage:** Redis/Memcached values, MongoDB's BSON, and Avro on Hadoop/Kafka pipelines each pick a format tuned for the workload (lookup latency, indexing, schema evolution). See [[Serialization_In_Storage]].
- **Security:** Deserialization of untrusted input is a serious attack surface — it has caused real-world RCE (Java, .NET, Python pickle). Validate, schema-check, and sign/encrypt. See [[Serialization_Security]].

## Concept Notes
- [[Serialization_Fundamentals]]
- [[Serialization_Formats]]
- [[Serialization_Tradeoffs]]
- [[Serialization_In_APIs]]
- [[Serialization_In_Storage]]
- [[Serialization_Security]]

## Flashcards
See flashcards: [[Serialization-Flashcards]]

## Related Notes
- [[HTTP_Statelessness]]
- [[State_Management_Techniques]]
- [[Token_Based_Authentication]] *(JWT is itself a serialized, signed token)*
- [[Microservices_Communication]]
- [[Microservices_Overview]]

---

*Last updated: 2026-07-18*
