# 4 Context Engineering and Persistence

For long-horizon tasks across iterative sessions, context management is critical. Claude Code employs specialized mechanisms to track conversational state, handle cost metrics, and preserve system constraints efficiently.

## 4.1 Context Collection (`context.ts`)
The context system is responsible for aggregating environment data before hitting the LLM reasoning loop.
- Core implementation (`context.ts` and `context/`) handles concatenating user prompts, file inclusion, command-line arguments, and historical conversational memory.
- Handles trimming and compaction logic to keep the sequence of events within the Anthropic context limits while ensuring that task instructions don’t suffer from "attention decay."

## 4.2 State and History Management
- **Memory Directory (`memdir/`):** Dedicated persistence mechanism mapped to local storage to retain cross-session memory and learned behaviors for the repository.
- **`history.ts` and `state/`:** Manages the conversational turns, serialized as observations and actions, and optionally supports saving/resuming via commands like `/resume` or `/thinkback`.
- **Migrations (`migrations/`):** Infrastructure indicating that the local schema for stored context and memory can safely upgrade as the local Claude Code binary is updated over time without losing user state.

## 4.3 Cost Tracking
- **`cost-tracker.ts` and `costHook.ts`:** Actively manages and computes token usage per-call, converting API utilization into visible developer costs.
- Directly ties into user warnings for high-spend operations and commands like `/cost`.

## 4.4 Project Onboarding State
- **`projectOnboardingState.ts`:** A specialized capability to detect the environment (node, python, go, rust) and potentially inject initial system prompts or run indexers upon the first invocation in a new repository. This seeds the system with a baseline understanding of the repo rather than starting from complete scratch.
