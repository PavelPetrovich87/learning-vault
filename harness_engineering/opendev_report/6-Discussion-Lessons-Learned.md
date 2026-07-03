# 3 Discussion (Lessons Learned)

This section examines five cross-cutting design tensions and extracts transferable lessons for builders of agentic systems.

### 3.1 Context Pressure as the Central Design Constraint

Context utilization is the single most important metric for agent longevity. Tool outputs (file contents, logs) often consume 70–80% of the context, dwarfing the system prompt.

- **Lesson: Treat context as a budget, not a buffer.** Use graduated reduction stages (pruning, masking) rather than a single emergency compaction.
- **Lesson: Offload large outputs to the filesystem.** Transform a context-consumption problem into a retrieval problem.
- **Lesson: Calibrate from API-reported token counts, not local estimates.** Provider-side formatting and injections make local counting systematically underestimate actual usage.

### 3.2 Steering Behavior Over Long Horizons

System prompt influence decays as conversations grow. Behavioral guidance is a signal-to-noise engineering problem.

- **Lesson: Inject reminders at the point of decision, not upfront.** Short, targeted user-role reminders at maximum recency are more effective than long system prompt sections.
- **Lesson: Separate thinking from action.** A separate thinking phase with no tool access produces better reasoning traces by removing the pressure of immediate action.

### 3.3 Safety Through Architectural Constraints

- **Lesson: Make unsafe tools invisible, not blocked.** Schema gating (removing tools from the agent's set) is more robust than checking permissions at call time. The model cannot reason about capabilities it doesn't know exist.
- **Approval persistence.** Users must be able to mark rules as “always allow” to prevent approval fatigue.
- **Lifecycle hooks.** Early design of hooks (blocking vs. non-blocking) is necessary for meaningful extensibility.

### 3.4 Designing for Approximate Outputs

LLMs produce approximately-correct outputs (trailing whitespace, minor indentation drift).

- **Lesson: Design tools to absorb LLM imprecision.** Use fuzzy matching chains (like in `edit_file`) to bridge the gap between intent and literal output.
- **Lesson: Adapt recovery hints to the agent’s available tool set.** Hints must suggest strategies the agent actually has access to.

### 3.5 Lazy Loading and Bounded Growth

- **Lesson: Bound every resource that grows with session length.** Caps on undo history, concurrent calls, and nudge budgets prevent resource exhaustion.
- **Lesson: Prefer empirical threshold tuning over first-principles calculation.** Use iterative failure analysis to find optimal values for caps and thresholds.
- **Self-healing indexes.** Design data structures so that loss triggers regeneration rather than failure.
- **Deterministic operations.** Routing simple operations (session management, mode switching) outside the agent loop avoids token cost and unnecessary nondeterminism.
