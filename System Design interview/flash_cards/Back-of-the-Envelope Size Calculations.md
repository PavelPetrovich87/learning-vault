---
status: analyzing
mastery: beginner
tags: [flashcards, system-design, sizing, estimation]
created: 2026-06-24
updated: 2026-06-24
---

# 🎴 Back-of-the-Envelope Size Calculations

What is the "golden rule" that bridges powers of two and powers of ten in system design interviews?
?
`2^10 ≈ 10^3` (1024 ≈ 1000)
#flashcards/system-design/sizing

---

How much is `2^20`, `2^30`, and `2^40` in decimal magnitude?
?
- `2^20 ≈ 10^6` (million)
- `2^30 ≈ 10^9` (billion)
- `2^40 ≈ 10^12` (trillion)
#flashcards/system-design/sizing

---

What are the first two steps of the napkin algorithm for estimating a power like `26^8`?
?
1. Round the base to the nearest power of two (`26 → 32 = 2^5`).
2. Rewrite the expression using that power of two (`32^8 = (2^5)^8`).
#flashcards/system-design/sizing

---

Approximate `26^8` using the golden rule.
?
`26^8 ≈ 32^8 = (2^5)^8 = 2^40 = (2^10)^4 ≈ (10^3)^4 = 10^12` (1 trillion)
#flashcards/system-design/sizing

---

What is the common mistake when computing `(2^a)^b`?
?
People add exponents instead of multiplying. The correct rule is `(2^a)^b = 2^(a×b)`.
#flashcards/system-design/sizing

---

What does `10^9` represent: 1 billion or 9 billion?
?
`10^9` = 1 billion (1 followed by 9 zeros). The exponent is the number of zeros, not a multiplier.
#flashcards/system-design/sizing

---

10 million DAU each make 6 requests per day. What is the approximate RPS?
?
`10^7 × 6 = 6 × 10^7` requests/day. Divide by `10^5` seconds/day → ~600 RPS.
#flashcards/system-design/sizing

---

A WhatsApp-scale system has 1B users sending 50 messages/day, 100 bytes each. How much raw storage is needed for 5 years?
?
- Messages/day: `10^9 × 50 = 5 × 10^10`
- Bytes/day: `5 × 10^10 × 100 = 5 × 10^12` = 5 TB/day
- 5 years: `5 TB × 365 × 5 ≈ 10 PB`
#flashcards/system-design/sizing

---

What is the interview shortcut for the number of seconds in a day?
?
~10^5 seconds (actual: 86,400).
#flashcards/system-design/sizing

---

When should you *not* use back-of-the-envelope approximation?
?
When exactness matters: billing, financial ledgers, safety-critical systems, medical dosing, compliance, and final capacity purchase orders.
#flashcards/system-design/sizing

---

What is the URL-shortener storage estimate for an 8-character key space if each mapping takes 1 KB?
?
`26^8 ≈ 32^8 = 2^40 ≈ 10^12` keys. `10^12 × 1 KB = 10^15` bytes = 1 PB at full saturation.
#flashcards/system-design/sizing

---

What does it mean to "focus on order of magnitude, not exact numbers" in a system design interview?
?
The interviewer wants the scale (GB vs TB vs PB, hundreds vs millions of RPS), not a precise calculation.
#flashcards/system-design/sizing
