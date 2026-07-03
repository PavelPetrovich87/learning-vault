---
status: new
tags: [architecture, agent_loop, react, tools]
source: Claude Code source code
created: 2026-04-03
difficulty: intermediate
---

# 🧠 Topic: Agent Tool Execution Loop 

> [!abstract] Executive Summary (ELI5)
> An AI agent doesn't just "think" all at once. It enters a loop where it reads the prompt, decides to pick up a tool (like a file reader), pauses its thinking to let the tool run, reads the result, and then continues thinking until the job is done.

---

## 🎯 The "Why"
- **Problem Solved:** LLMs are static stateless responders. To interact with the world, they need a deterministic execution harness to pause generation, invoke real-world code, and resume with the new observation.
- **Alternatives:** 
  - Single-shot generation (guessing the solution without verifying).
- **Trade-offs:** Tool looping requires parsing LLM outputs to detect structured tool calls, running local system code, and securely updating the context history. This introduces latency, but grants actual autonomy.

## 💡 Mental Model & Analogy
- **Analogy:** "This is like a chef reading a recipe. They read a step, pause to fetch an ingredient from the fridge (the tool), observe what they fetched, and then read the next step based on what they found."
- **Visual/Diagram:** 
```mermaid
sequenceDiagram
    participant Model
    participant Loop
    participant System
    Loop->>Model: Context + Prompt
    Model->>Loop: Text Stream + ToolCall(Bash)
    Loop->>Loop: Halt Stream
    Loop->>System: exec "ls -la"
    System-->>Loop: Results
    Loop->>Model: Observation(Results)
```

## 🛠️ Core Principles & Code Snippets
- **Strict Schemas:** Tools must define rigid input schemas (Zod/JSON Schema) so the LLM formats arguments predictably.
- **Permission gating:** Not all tools should auto-run. Pure read tools (like `grep`) might auto-run, but a `git push` via Bash requires a prompt injection in the exact tool execution loop interrupting the cycle to ask the human.

```typescript
// Conceptual Tool Interface
interface Tool {
  name: string;
  schema: ZodSchema;
  requiresApproval(args: unknown): boolean;
  execute(args: unknown): Promise<string>;
}

// In the QueryEngine loop
if (llmResponse.hasToolCall) {
  const tool = registry.get(llmResponse.toolName);
  if (tool.requiresApproval(args)) {
    await requestUserVeto(); // Pauses loops for human CLI input
  }
  const observation = await tool.execute(args);
  history.push(observation);
}
```

> [!tip] Implementation Note
> Abstract every action into a `Tool` class. Even internal actions like triggering a "plan mode" toggle should be exposed to the LLM as a tool.

## ⚠️ Pitfalls & Anti-patterns
- **Common Mistakes:** Letting the LLM generate unconstrained JSON which fails the parser, dropping the agent out of the loop.
- **When NOT to use:** For deterministic CLI commands (like `/help`). Use Slash Commands which bypass the loop entirely.
- **Performance/Security considerations:** Unapproved Bash executions. Tool loops must inject strict security sandbox checks before returning control to the host system.

## 🔗 Deep Connections
- **Parent Topic:** [[Agent Reasoning Loop]]
- **Related Concepts:** [[ReAct Pattern]], [[Tool Calling]]
- **Prerequisites:** JSON Schema parsing, understanding of conversational context tracking.

## ❓ Active Recall Questions
> [!faq]- Why must the generation stream halt when a tool is called?
> Because the LLM cannot proceed effectively until it observes the outcome of the tool. If it guesses the outcome, it will hallucinate file contents or command responses.

> [!faq]- How does the loop handle human permissions?
> The loop yields control back to the UI thread (via Ink) before executing the tool payload, awaiting a keypress. If vetoed, the loop feeds an "Error: User Denied" observation back to the LLM and continues.
