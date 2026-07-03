---
status: new
tags: [context-engineering, optimization, opendev]
source: OpenDev Technical Report
created: 2026-03-19
difficulty: intermediate
---

# 🧠 Topic: Tool Result Optimization (Summarization)

> [!abstract] Executive Summary (ELI5)
> If you ask an AI to read a book, don't show it every single word in the book every time you ask a question. Just give it the summary of each chapter so it has room to think about your new questions.

---

## 🎯 The "Why"
- **Problem Solved:** "Context Garbage." Raw tool outputs like `npm install` logs, 5000-line directory listings, or full file reads quickly fill up the model's memory with useless text.
- **Alternatives:** Truncation (cutting the end off), which often removes the most important info (like the error at the bottom of a log).
- **Trade-offs:** 
    - **Pro:** Saves 50-80% of tokens; keeps the agent focused on the task instead of the logs.
    - **Con:** Risk of "Over-summarization" where critical details (like a specific line number in an error) are lost.

## 💡 Mental Model & Analogy
- **Analogy:** "The Executive Briefing." The CEO doesn't read every email; they read a 1-page summary with the most important numbers and "Action Needed" notes.

## 🛠️ Core Principles (Actual Rust Implementation)
The `opendev-tools-impl` crate implements a sophisticated truncation and overflow system in `truncation.rs`.

```rust
// Examples from crates/opendev-tools-impl/src/truncation.rs

pub const MAX_LINES: usize = 2000;
pub const MAX_BYTES: usize = 50 * 1024; // 50 KB

pub enum TruncateDirection {
    Head, // Keep first N lines (for file reads)
    Tail, // Keep last N lines (for build/test logs)
}

pub fn truncate_output(...) -> TruncateResult {
    // If output exceeds limits:
    // 1. Save full content to ~/.opendev/tool-output/tool_{timestamp}_{uuid}
    // 2. Return a truncated preview string to the agent
    // 3. Inject a HINT: "Full output saved to: {path}. Use Grep or Read offset."
}
```

1. **Type-Specific Truncation:**
    - **Head Truncation:** Used for file reads or directory listings where the beginning is most relevant.
    - **Tail Truncation:** Used for shell commands and test logs where the error message usually appears at the very end.
2. **FileSystem Overflow:** Large results are never lost; they are offloaded to `~/.opendev/tool-output/`, keeping the LLM's context window clean while preserving data for human or subagent review.
3. **Automated Cleanup:** Files in the overflow directory are automatically deleted after 7 days (`RETENTION_SECS`) to prevent unbounded disk usage.
4. **Recovery Guidance:** Every truncated response includes a specific "Recovery Hint" instructing the agent on how to retrieve the missing information using other tools (e.g., `grep_search` on the overflow file).


## ⚠️ Pitfalls & Anti-patterns
- **Summarizing Errors:** Never hide the *full* text of an error message unless it's repetitive. Errors are the most valuable context for self-correction.
- **Lost Anchors:** If you summarize a file read and the agent needs to edit line 142, it needs the content of line 142. Don't summarize a file the agent is *currently* editing.

## 🔗 Deep Connections
- [[Adaptive Context Compaction (ACC)]]
- [[Tokenization]]
- [[Information Theory]]

## ❓ Active Recall Questions
> [!faq]- Why is "Head-Tail" truncation better than simple "Head" truncation for logs?
> Because logs usually put the "setup" at the top and the "result/error" at the bottom. Simple truncation loses the most important part (the ending).

---
