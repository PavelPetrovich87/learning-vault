<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->
<!-- #flashcards/system-design -->

# 🎴 Code Quality as AI Infrastructure Flashcards

<!-- In the context of **AI-Assisted Engineering**, why is achieving exactly 100% test coverage considered a "phase change" compared to 95% coverage? -->
<!-- ? -->
<!-- At 95%, the agent must use its limited reasoning to guess if the uncovered 5% requires testing. At exactly 100%, ambiguity is eliminated: any uncovered code is a binary, non-negotiable failure, making the agent's goal deterministic. -->
<!--SR:!2026-04-30,7,210-->

---

<!-- How does using strict **Semantic Types** (e.g., `UserId` or `StripeInvoiceId`) instead of generic primitives (`string`) specifically improve the performance of an autonomous coding agent? -->
<!-- ? -->
<!-- It shrinks the LLM's "search space" of possible actions. Semantic types explicitly define domain boundaries and data flow, preventing the agent from having to infer context from surrounding code, thereby drastically reducing hallucinations. -->
<!--SR:!2026-04-25,2,210-->

---

<!-- When designing architecture for **AI Coding Agents**, what is the primary risk of using large, monolithic files? -->
<!-- ? -->
<!-- **Context Truncation/Summarization.** Large files force the LLM to summarize or truncate its context window, degrading performance. Small, well-scoped files ensure the *entire* logic fits into the active context, eliminating a whole class of reasoning errors. -->
<!--SR:!2026-05-21,29,230-->

---

<!-- In an **Agentic Workflow**, why is the directory structure (e.g., `./billing/invoices/` vs `./utils/`) more critical than in traditional development? -->
<!-- ? -->
<!-- Because **the filesystem is the UI for the agent**. Agents navigate codebases via directory structures and filenames; a logical, semantic folder hierarchy acts as a high-signal interface that provides immediate context and prevents reasoning failures. -->
<!--SR:!2026-06-22,55,250-->
