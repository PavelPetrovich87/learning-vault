---
status: new
tags: [harness-engineering, agentic-workflows, reliability, opendev]
source: "The Model Upgrade Treadmill Nobody Talks About (Medium)"
created: 2026-03-23
difficulty: intermediate
---

# 🧠 Topic: Harness Engineering Primitives

> [!abstract] Executive Summary (ELI5)
> Think of an AI agent like a computer: the model is the brain (CPU), the memory is short-term (RAM), and the **Harness** is the Operating System (OS). To make it reliable, we need "safety rails" like checking its own work before finishing, stopping it if it gets stuck in a loop, and making sure it doesn't forget everything if it has to restart.

---

## 🎯 The "Why"
- **Problem Solved:** Raw models are "stateless" and "impulsive." They drift off-track during long tasks, hallucinate "done" states, and waste tokens repeating the same mistakes.
- **Alternatives:** Just "prompting harder" (ineffective for long horizons).
- **Core Insight:** Reliability lives in the **Harness** (infrastructure), not the model's raw intelligence.

## 💡 The OS Analogy
| Component | Computing Equivalent | Role in Agentic Systems |
| :--- | :--- | :--- |
| **Model** | **CPU** | Raw processing power/intelligence. |
| **Context Window** | **RAM** | Limited, volatile, and expensive short-term memory. |
| **Agent Harness** | **Operating System** | Curates context, handles boot sequences, provides drivers (tools), and manages state. |
| **Agent** | **Application** | The specific task or logic running on top. |

---

## 🛠️ Five Critical Reliability Primitives

### 1. Context Engineering (Proactive)
Don't wait for the agent to ask. Proactively deliver:
- Directory structures and available tools.
- Coding standards and environmental constraints.
- **Goal:** Give the model what it needs *before* it fumbles.

### 2. Self-Verification Loops
A "pre-completion checklist" that intercepts the agent before it declares victory.
- **Mechanism:** Force a verification pass against the original task spec.
- **Why:** Models are biased toward their first plausible solution ("close enough to fool myself").

### 3. Loop Detection (Doom Loop Breaker)
Track per-file edit counts or repetitive tool calls.
- **Mechanism:** After $N$ edits to the same file, inject a "reconsider your approach" signal into the context.
- **Why:** Prevents agents from burning tokens making tiny variations to a broken approach.

### 4. Durable State Management
Surviving context window resets or network hiccups.
- **Mechanism:** Use `git commit history` + structured `progress.md` files.
- **Why:** Ensures that an agent starting a fresh window can immediately orient itself without "amnesia."

### 5. Adaptive Reasoning Budgets ("The Reasoning Sandwich")
Optimizing compute/token spend based on task phase.
- **High Compute:** Planning (initial strategy).
- **Medium Compute:** Execution (writing code/running tools).
- **High Compute:** Final Verification (QA/Review).
- **Why:** Maxing out reasoning on every subtask causes timeouts; under-reasoning on planning causes failure.

---

## ⚠️ Pitfalls & Anti-patterns
- **Dumb RAG:** Dumping everything into the context and "hoping the model figures it out."
- **Ephemeral Execution:** Treating tasks as one-off function calls instead of long-running workflows.

## 🔗 Deep Connections
- [[Thinking-vs-Action-Phases]]
- [[Tool-Result-Optimization]]
- [[Context-Aware-System-Reminders]]

## ❓ Active Recall Questions
> [!faq]- In the OS Analogy, why is the Harness compared to the Operating System?
> Because it manages the lifecycle, provides the "drivers" (tools), and curates the "RAM" (context window) for the "CPU" (model).

> [!faq]- What is the "Reasoning Sandwich" and why is it used?
> It allocates high compute for planning and verification, but medium compute for execution, optimizing for cost and reliability.
