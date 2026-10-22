---
status: archived
tags: [system-design, web-concepts, serialization, security, deserialization, rce]
source: Udemy System Design Course
created: 2026-07-18
updated: 2026-10-21
difficulty: intermediate
---

# 🧠 Topic: Serialization Security

> [!abstract] Executive Summary (ELI5)
> Deserializing untrusted bytes is one of the most dangerous things a service can do — it has caused countless **Remote Code Execution (RCE)** vulnerabilities. The risk comes from formats that can instruct the runtime to *instantiate arbitrary classes and run their constructors* (Java `Serializable`, .NET `BinaryFormatter`, Python `pickle`, Ruby `Marshal`, PHP `unserialize`). The defense is layered: don't deserialize native objects from untrusted input, validate against a strict schema, sign/encrypt anything sensitive, and prefer portable data-only formats (JSON, Protobuf, Avro) at every trust boundary.

---

## 🎯 The "Why"
- **Problem Solved:** Serialization is a way to make a process *reconstruct objects from bytes*. If the bytes are attacker-controlled and the format can carry instructions ("instantiate class X, then class Y, then call method Z"), deserialization becomes a code-execution primitive.
- **Alternatives:** Data-only formats (JSON, Protobuf, Avro) reconstruct plain values, not arbitrary objects — they remove the entire class of "gadget chain" attacks. The alternative *to* avoiding native deserialization is heavy mitigation (allow-lists, integrity checks), which is fragile.
- **Trade-offs:** Native serializers are convenient (no schema, reconstructs your real types) but unsafe at any trust boundary. Safe formats require a schema and explicit mapping to your domain types.

## 💡 Mental Model & Analogy
- **Analogy:** "Accepting a serialized object from the internet is like accepting a piece of flat-pack furniture from a stranger. A safe format (JSON/Protobuf) is a box of labeled parts — you decide what to build. An unsafe format (`pickle`, Java `Serializable`) is a box that *contains a robot that assembles itself and starts running* — and the stranger chose which robot. Don't let strangers ship you self-assembling robots."
- **Visual/Diagram — the trust boundary:**
```mermaid
graph LR
    A[Attacker] -->|crafted bytes| DS[Deserialization entrypoint]
    DS -->|"Native format: instantiate arbitrary classes<br/>→ Gadget chain → RCE"| RCE💥
    DS -.->|"Data-only format: plain values only<br/>+ schema validation"| SAFE[Safe reconstruction]
    style RCE fill:#fdd,stroke:#a00
    style SAFE fill:#dfd,stroke:#0a0
```

## 🛠️ Core Principles

### The Three Risk Families (PDF Q12)
- **Deserialization attacks:** malicious payloads exploit insecure deserialization to run code, escalate privileges, or pivot inside a network.
- **Injection vulnerabilities:** poorly validated serialized data can carry injection payloads (e.g., JSON values that become SQL, LDAP, or template injection when re-emitted).
- **Man-in-the-middle (MITM):** if serialized data transits an unencrypted channel or is unsigned, an attacker who intercepts it can tamper — even with a safe format.

### How Improper Deserialization Becomes a Vulnerability (PDF Q13)
- **Remote Code Execution (RCE):** attackers craft a "gadget chain" — a sequence of legitimate classes already on the classpath whose constructors/setters, when triggered in order, execute attacker-controlled code. Famous families include Java's Apache Commons Collections gadget chains, .NET `BinaryFormatter`/`DataContractSerializer` exploits, Python `pickle` (which is documented as unsafe for untrusted input), Ruby `Marshal`, and PHP object-injection gadgets.
- **Denial of Service (DoS):** payloads engineered to blow up during deserialization — e.g., deeply nested JSON ("billion laughs"-style), huge strings, hash-collision keys that turn a hash map into O(n²), or decompression bombs.
- **Data tampering:** unsigned serialized data can be modified in transit or at rest (privilege escalation by flipping a `role` field, balance changes, etc.).

### Mitigation Strategies

> [!warning] Hierarchy of controls
> The list below is in priority order — the first items remove the risk entirely; later ones are defense-in-depth on top.

1. **Don't deserialize native objects from untrusted input.** This single rule eliminates the gadget-chain class. Use data-only formats (JSON, Protobuf, Avro) at every trust boundary.
2. **Use a strict schema.** Protobuf/Avro/JSON Schema reject unexpected fields, wrong types, and shape drift before they reach your domain logic.
3. **Validate input against the schema *and* business rules** before acting on it — well-formed ≠ semantically valid.
4. **Integrity + confidentiality:** sign (HMAC or asymmetric signature) anything that mustn't be tampered with; encrypt anything sensitive (tokens, PII). TLS protects in transit; signatures protect at rest and end-to-end.
5. **Bound resource consumption:** cap nesting depth, payload size, and parse time to defeat DoS payloads. Most parsers expose these limits — turn them on.
6. **Pin types / allow-lists** if you absolutely must use a native serializer (e.g., Java's `ObjectInputFilter`, .NET's `XmlSerializer`-with-known-types) — but treat this as a last resort, not a license.
7. **Patch your dependencies** — gadget chains exist because of specific vulnerable libraries; keep them updated and run dependency scanners (OWASP Dependency-Check, Snyk, etc.).

### Format-by-Format Risk Posture

| Format | Risk profile | Why |
|---|---|---|
| **JSON** | Low (data-only) | Reconstructs plain values, no code execution. Still vulnerable to DoS (deep nesting) and to *injection downstream* if values are concatenated into SQL/HTML. |
| **Protobuf / Avro** | Low (data-only + schema) | Strict schema rejects unexpected types; no object instantiation. |
| **BSON** | Low (data-only, binary) | Same data model as JSON; parser bugs are the main residual risk — keep drivers patched. |
| **Java `Serializable`** | **High — avoid at boundaries** | Designed to reconstruct arbitrary serializable objects; gadget chains are well-documented. |
| **.NET `BinaryFormatter`** | **High — deprecated** | Microsoft itself marked it obsolete and recommends `JsonSerializer`/`XmlSerializer` instead. |
| **Python `pickle`** | **High — documented unsafe** | Docs explicitly say "never unpickle from untrusted sources." |
| **Ruby `Marshal`** | **High — documented unsafe** | Same story as pickle. |
| **PHP `unserialize`** | **High** | Object-injection gadgets abound; prefer `json_encode`/`json_decode`. |

## ⚠️ Pitfalls & Anti-patterns
- **Trusting "it's only internal traffic."** Trust boundaries shift; lateral movement is real. Treat any cross-process bytes as untrusted unless *both* endpoints are inside the same hardened boundary.
- **Disabling parser limits** for "convenience" — the day someone sends you a 50 MB nested JSON, your service falls over.
- **Signing data but not binding it to a context** — a signed token from *service A* shouldn't be accepted by *service B*; bind signatures to an audience/issuer.
- **Conflating encryption with integrity** — encrypted-but-unsigned data can still be tampered with (bit-flipping). Use authenticated encryption (AES-GCM) or sign separately.
- **Hand-rolling your own crypto or signature scheme** — use established libraries and standard formats (JWS/JWE, PASETO, TLS).
- **Forgetting that *downstream* injection is still your problem** — JSON is safe to deserialize, but if you concatenate a JSON field into SQL, you've reintroduced a vulnerability at a different layer.

## 🔗 Deep Connections
- **Parent Topic:** [[01_Serialization]]
- **Related Concepts:** [[Serialization_Fundamentals]] (the inverse operation, deserialization, is where the risk lives), [[Serialization_Formats]] (data-only vs native formats), [[Session_Security]] (tokens, signatures, tampering), [[Token_Based_Authentication]] (JWTs are signed serialized payloads — tampering is detected via signature, not prevented).
- **Prerequisites:** Trust boundaries, signatures vs encryption, the concept of a "gadget chain."

## ❓ Active Recall Questions

> [!faq]- What security risks are associated with serialization? *(PDF Q12)*
> Three families: (1) **deserialization attacks** — insecure deserialization of native object formats can lead to RCE; (2) **injection vulnerabilities** — poorly validated serialized data can carry SQL/LDAP/template-injection payloads that fire downstream; (3) **man-in-the-middle** — intercepted serialized data can be modified in transit if it's not encrypted and not signed.

> [!faq]- How can improper deserialization lead to vulnerabilities? *(PDF Q13)*
> Three failure modes: **Remote Code Execution** — attackers craft a "gadget chain" of legitimate classes whose constructors/setters execute attacker code when deserialized (Java Apache Commons Collections, .NET `BinaryFormatter`, Python `pickle`, PHP `unserialize`); **Denial of Service** — payloads engineered to blow up during parsing (deeply nested JSON, hash-collision keys, decompression bombs); **Data tampering** — unsigned serialized data can be edited to escalate privileges or alter balances.

> [!faq]- You inherit a service that accepts Java `Serializable` objects over a TCP socket from a partner system. How do you make it safe?
> In priority order: (1) negotiate with the partner to move the boundary to a data-only format (JSON/Protobuf) so no native object reconstruction happens; (2) if you must keep Java serialization short-term, add an `ObjectInputFilter` allow-list of permitted classes and reject everything else; (3) require integrity (sign the stream) and confidentiality (TLS or envelope encryption); (4) cap payload size and nesting to defeat DoS; (5) keep dependencies patched to remove known gadget chains. Treat this as a temporary state, not a steady state.

> [!faq]- If JSON is a "safe" format, why can a JSON API still be exploited through serialization?
> Because JSON is safe to *parse*, but unsafe to *use*. Deserializing JSON never runs code, but if your code concatenates a JSON field into SQL, HTML, or a shell command, you've reintroduced injection at the application layer. JSON is also vulnerable to DoS (deeply nested payloads) unless parser limits are set. Safe format ≠ safe system — validation, parameterization, and output encoding still matter.

---

*Last updated: 2026-07-18*
