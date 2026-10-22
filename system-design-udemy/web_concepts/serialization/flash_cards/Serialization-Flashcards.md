---
tags: [flashcards, system-design, web-concepts, serialization]
created: 2026-07-18
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Serialization — Flashcards

<!-- > One card per interview question (Q1–Q13) from `../Serialization_Detailed_Interview_Answers.pdf`, plus synthesis cards on the headline trade-offs. Source notes: [[Serialization_Fundamentals]], [[Serialization_Formats]], [[Serialization_Tradeoffs]], [[Serialization_In_APIs]], [[Serialization_In_Storage]], [[Serialization_Security]]. -->

<!-- What is serialization, and why is it needed in system design? -->
<!-- ? -->
<!-- Serialization is converting an in-memory object/data structure into a byte format that can be stored, transmitted, and reconstructed later (the inverse is deserialization). It's needed for **data exchange** (APIs, RPC, queues), **storage** (DB rows, files), **networking** (the wire only carries bytes), and **interoperability** (a Python service talking to a Go service via a language-neutral format like JSON or Protobuf). -->
<!-- #flashcards/system-design -->

<!-- How does serialization impact data exchange and storage? -->
<!-- ? -->
<!-- The format choice drives four things: **size** (Protobuf is compact, XML is verbose), **speed** (binary encodes/decodes faster than text), **compatibility** (human-readable vs schema-driven), and **data fidelity** (naive formats lose precision on floats/big-ints/timestamps; schema-driven formats reduce that risk). Same data, very different real-world cost depending on the pick. -->
<!-- #flashcards/system-design -->

<!-- How would you compare JSON, XML, Protocol Buffers, and Avro? -->
<!-- ? -->
<!-- **Human-readable:** JSON/XML yes, Protobuf/Avro no. **Schema:** JSON/XML optional, Protobuf/Avro required. **Encoding:** JSON/XML text, Protobuf/Avro binary. **Size:** XML large > JSON medium > Protobuf/Avro small. **Speed:** XML slow > JSON moderate > Protobuf/Avro fast. **Use cases:** JSON for web APIs, XML for SOAP/configs, Protobuf for gRPC/microservices, Avro for Hadoop/Kafka big data. -->
<!-- #flashcards/system-design -->

<!-- When would you choose Protocol Buffers over JSON? -->
<!-- ? -->
<!-- When **performance** (smaller payload, faster encode/decode), **strong typing** (compile-time contract checks), **safe schema evolution** (field numbers give backward/forward compatibility), or **gRPC** (Protobuf is its default wire format) outweigh human readability. Stay with JSON when humans must read/debug payloads in the browser or when ad-hoc schemaless flexibility matters more. -->
<!-- #flashcards/system-design -->

<!-- How does serialization affect API performance and efficiency? -->
<!-- ? -->
<!-- Three ways: (1) **Response time** — lighter formats reduce payload size, cutting both serialization CPU and time-on-wire; (2) **Processing overhead** — JSON/XML parsing dominates CPU on hot paths, binary formats cut it sharply; (3) **Bandwidth** — efficient serialization reduces egress cost, which is often the cheapest way to scale an API further. -->
<!-- #flashcards/system-design -->

<!-- Why is Protocol Buffers the default wire format in gRPC instead of JSON? -->
<!-- ? -->
<!-- Four reasons: (1) **binary format** — much smaller and faster than JSON; (2) **schema enforcement** — the `.proto` guarantees backward/forward compatibility via field numbers; (3) **efficient serialization** — minimal overhead, optimized for network transmission; (4) **structured RPC fit** — gRPC's typed stubs, metadata, and bidirectional streaming model assume a strongly-typed message format, and Protobuf provides exactly that. -->
<!-- #flashcards/system-design -->

<!-- How does serialization affect caching strategies in systems like Redis? -->
<!-- ? -->
<!-- Redis/Memcached are opaque key-value stores over bytes — you serialize before `SET` and deserialize after `GET`, so the format drives **latency** (compact formats cut decode time), **storage efficiency** (you can pre-compress blobs), and **lookup behavior** (smaller values mean less memory pressure and eviction). JSON is the debug-friendly default; MessagePack/Protobuf/Avro win at high cardinality or for hot values. -->
<!-- #flashcards/system-design -->

<!-- What are the trade-offs between readability, efficiency, and compatibility in serialization formats? -->
<!-- ? -->
<!-- **Readability:** JSON/XML are easy to read but large and slow to parse. **Efficiency:** Protobuf/Avro are compact and fast but require schema management and aren't human-readable. **Compatibility:** JSON is flexible and ubiquitous, but flexibility ≠ compatibility — schema-driven formats (Protobuf/Avro) are what *guarantee* safe evolution. You optimize one axis at the cost of the others. -->
<!-- #flashcards/system-design -->

<!-- How does serialization impact CPU and memory usage? -->
<!-- ? -->
<!-- Text formats (JSON, XML) burn more CPU on string parsing and object construction, and large/deeply nested payloads spike memory because the parser builds a full tree before you can stream it. Binary formats (Protobuf, Avro) have lower CPU and memory footprints thanks to fixed-width fields, length prefixes, and precompiled decoders. At scale, inefficient serialization is a common hidden CPU bottleneck. -->
<!-- #flashcards/system-design -->

<!-- How is Avro beneficial in big data systems like Hadoop and Kafka? -->
<!-- ? -->
<!-- Three reasons: (1) **schema evolution** — fields can be added/removed/renamed without breaking readers, because writer and reader schemas are reconciled at decode time; (2) **schema travels with data** — Avro files embed the writer's schema, and Kafka pairs each message with a schema ID from a registry, so records stay readable for years; (3) **binary and compact** — small on disk, fast to process. Big data's defining property is that records outlive the code that produced them, and Avro is engineered for exactly that. -->
<!-- #flashcards/system-design -->

<!-- Why do some databases like MongoDB use BSON instead of JSON? -->
<!-- ? -->
<!-- BSON is JSON's data model encoded as binary with length prefixes and richer types (Date, int32/int64, BinData, Decimal128). The length prefixes let MongoDB **skip through a document without fully parsing it**, which makes indexing and projections fast. The richer types remove JSON's date/binary/decimal workarounds. Net effect: faster queries, efficient indexing, faithful type fidelity on disk. -->
<!-- #flashcards/system-design -->

<!-- What security risks are associated with serialization? -->
<!-- ? -->
<!-- Three families: (1) **deserialization attacks** — insecure deserialization of native object formats can lead to RCE; (2) **injection vulnerabilities** — poorly validated serialized data can carry SQL/LDAP/template-injection payloads that fire downstream; (3) **man-in-the-middle** — intercepted serialized data can be tampered with in transit if it isn't encrypted and signed. -->
<!-- #flashcards/system-design -->

<!-- How can improper deserialization lead to vulnerabilities? -->
<!-- ? -->
<!-- Three failure modes: **RCE** — attackers craft a "gadget chain" of legitimate classes whose constructors/setters run attacker code (Java Apache Commons Collections, .NET `BinaryFormatter`, Python `pickle`, PHP `unserialize`); **DoS** — payloads engineered to blow up during parsing (deeply nested JSON, hash-collision keys, decompression bombs); **Data tampering** — unsigned serialized data can be edited to escalate privileges or alter balances. Mitigations: avoid native object deserialization at boundaries, validate against a strict schema, sign/encrypt sensitive data, bound payload size and nesting, patch dependencies. -->
<!-- #flashcards/system-design -->

<!-- Synthesis: text formats vs binary formats — when do you pick which? -->
<!-- ? -->
<!-- Pick **text (JSON/XML)** when humans need to read/debug payloads, the browser must consume them natively, traffic is low-to-moderate, or schema flexibility matters most. Pick **binary (Protobuf/Avro/BSON)** when CPU/bandwidth is the constraint (high QPS internal traffic, big-data pipelines, storage indexes), when you need a strict typed contract, or when records must outlive the code that produced them. A pragmatic split: JSON at the edge, Protobuf/gRPC internally, Avro in the data lake, BSON in the document DB. -->
<!-- #flashcards/system-design -->

<!-- Synthesis: why is deserialization a security attack surface, and what's the single most important defense? -->
<!-- ? -->
<!-- Because some formats (Java `Serializable`, .NET `BinaryFormatter`, Python `pickle`, Ruby `Marshal`, PHP `unserialize`) can reconstruct *arbitrary objects* from bytes, an attacker who controls the payload can chain legitimate classes ("gadget chain") into RCE. The single most important defense: **don't deserialize native objects from untrusted input — use data-only formats (JSON/Protobuf/Avro) at every trust boundary.** Layer schema validation, signatures/encryption, payload-size caps, and patched dependencies on top. -->
<!-- #flashcards/system-design -->

<!-- Synthesis: a partner service sends you Java `Serializable` objects over a socket. How do you de-risk it without breaking the integration? -->
<!-- ? -->
<!-- In priority order: (1) negotiate moving the boundary to a data-only format (JSON/Protobuf) so no native reconstruction happens; (2) if Java serialization is unavoidable short-term, add an `ObjectInputFilter` allow-list of permitted classes and reject everything else; (3) require integrity (HMAC/signature) and confidentiality (TLS/envelope encryption); (4) cap payload size and nesting to defeat DoS; (5) keep dependencies patched to remove known gadget chains. Treat this as a transition state, not steady state. -->
<!-- #flashcards/system-design -->
