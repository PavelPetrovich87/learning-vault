---
status: new
tags: [behavioral-steering, psychology, opendev]
source: OpenDev Technical Report
created: 2026-03-19
difficulty: intermediate
---

# 🧠 Topic: Behavioral Steering via Recency Bias

> [!abstract] Executive Summary (ELI5)
> AI models are like people who remember the last thing you told them much better than what you told them an hour ago. We use this "Recency Bias" to our advantage by putting the most important rules at the very bottom of the chat, right where the AI will see them before it speaks.

---

## 🎯 The "Why"
- **Problem Solved:** "Instruction Drift." As the chat history gets long, the agent starts to ignore the rules in the System Prompt (which is at the very top/oldest part of the context).
- **Alternatives:** Hardcoding logic in the backend (inflexible), Fine-tuning (expensive/slow).
- **Trade-offs:** 
    - **Pro:** High compliance with complex behaviors; zero code changes needed to update rules.
    - **Con:** Consumes more tokens than a static system prompt; can feel "repetitive" in the logs.
## 💡 The Steering Hierarchy: System vs. Behavior
To steer an agent effectively in 2026 (using **Claude 4.6 Sonnet**), you need both a reliable transport layer and high-quality content:

1. **System (The "Radio"):** The **Infrastructure layer** (`System Reminders`). It handles the *how* and *when*. It ensures the message is injected at the very last turn to exploit Recency Bias.
2. **Behavior (The "Order"):** The **Nudge layer**. It's the *what*. It provides tactical, rule-based steering based on state detection (e.g., "Empty Action detected").

> **Key Rule:** You can have the best Nudge (behavioral rule), but if the System (infrastructure) puts it at the top of the context, the agent will ignore it. **Pipes first, Water second.**

## 🛠️ Strategic Steering (Actual Rust Implementation)
...
The `opendev-agents` crate implements behavioral steering through a **"Nudge Budget"** and event-driven injection in the `ReactLoop`. The Harness acts as a **State Machine Monitor**, constantly comparing the *desired state* (Plan/Goal) with the *actual output* (Last response/Tool result).

### 1. Nudge Rule Implementation
Nudges are not hardcoded strings; they are **predicates** defined in the `opendev-context` composition engine. Each rule consists of a `Condition` (a function taking the current state) and a `Payload`.

```rust
// Conceptual implementation of a Nudge Rule in opendev-context
pub struct NudgeRule {
    pub name: String,
    pub condition: Box<dyn Fn(&AgentState) -> bool>, // The Trigger
    pub message: String,
    pub priority: i32, // Usually highest to be at the very bottom
}
```

### 2. Core Trigger Mechanisms
The Harness automatically injects a nudge when it detects one of these "Drifts":

| Trigger | Condition (Logic) | Nudge Content (Payload) |
| :--- | :--- | :--- |
| **Empty Action** | `if has_reasoning && tools.is_empty() && !is_finished` | "You provided reasoning but didn't execute any tool. Take action." |
| **Plan Drift** | `if intent == "finish" && plan.has_todo_items()` | "Wait, you have 3 unfinished items in your plan. Verify completion." |
| **Error Loop** | `if history.last_3_results().all_errors_identical()` | "You are repeating the same error. Change your strategy or tool parameters." |
| **Step Pressure** | `if current_step > (max_steps * 0.8)` | "Budget Warning: You are at step 16/20. Converge on the solution now." |

### 3. The Injection Process
1. **The Power of the Last Turn:** By using the `MessageClass::Nudge`, the `CompositionEngine` ensures these instructions are placed at the **absolute bottom** of the context. This exploits the model's *Attention Mechanism*, making the nudge the most salient instruction.
2. **Nudge Budgeting:** Limits (e.g., `max_nudge_attempts: 3`) prevent "Nudge-Response Loops" where a stubborn model ignores the reminder. If the budget is exhausted, the Harness yields to the user.
3. **Targeted Delivery:** In OpenDev's Compound Architecture, Nudges are often filtered to reach **only the Action Model**, keeping the Thinking Model's trace clean and focused on pure reasoning.

```rust
// Logic from crates/opendev-agents/src/react_loop/helpers/mod.rs
if consecutive_no_tool_calls >= self.config.max_nudge_attempts {
    debug!("Max nudge attempts reached, accepting completion");
} else {
    // Inject a nudge to remind the agent of its current strategy or state
    let nudge = get_reminder("implicit_completion_nudge", &[("task", &self.task)]);
    append_nudge(messages, &nudge);
}
```


## ⚠️ Pitfalls & Anti-patterns
- **Over-steering:** If you give too many "reminders," the agent gets confused and loses track of the original goal.
- **Context Bloat:** Each steering message adds tokens. Use them surgically.

## 🔗 Deep Connections
- [[Context-Aware System Reminders]]
- [[Attention Mechanism]]
- [[Prompt Engineering]]

## ❓ Active Recall Questions
> [!faq]- Why is the "Recency Bias" an engineering opportunity rather than just a bug?
> Because it allows us to dynamically override or reinforce behaviors exactly when they are needed, without bloating the static system prompt.

---
