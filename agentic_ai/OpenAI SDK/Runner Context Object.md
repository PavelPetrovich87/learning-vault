---
status: stable
mastery: beginner
tags: [agentic-ai, openai, agents-sdk, context, runner, tools, state]
created: 2026-06-29
updated: 2026-10-21
difficulty: intermediate
---

# 🧠 Topic: Runner Context Object

> [!abstract] Executive Summary (ELI5)
> The `context` object is a typed bag of data you pass to `Runner.run(...)` so that every tool, guardrail, and hook in the agent loop can read the same request-scoped state — user IDs, database sessions, API clients, or feature flags — without falling back to global variables.

---

## 🎯 The "Why"
- **Problem Solved:** Agent tools often need request-specific dependencies: a user ID for authorization, a DB connection, an API token, or a tenant identifier. Without a context object, developers reach for globals or closures, which makes testing, concurrency, and multi-tenancy painful.
- **Alternatives:**
    - **Global variables:** Simple but not thread/async safe and hard to test.
    - **Closures / partial functions:** Bind state to each tool at definition time. Works, but couples tool registration to runtime state and makes reuse harder.
    - **Dependency-injection frameworks:** More powerful, but heavier than the SDK's built-in `context` mechanism.
- **Trade-offs:** The SDK context is lightweight, typed, and async-safe. The limitation is that it is request-scoped: it lives for one `Runner.run` invocation and must be threaded through every call manually.

## 💡 Mental Model & Analogy
- **Analogy:** The context object is like a **clipboard carried by a hospital nurse on a single patient round**. Every specialist who sees the patient during that round can read the clipboard, but the clipboard is created fresh for each round and never mixed between patients.
- **Visual/Diagram:**
```mermaid
flowchart TD
    U[User Request] --> R[Runner.run(agent, input, context=ctx)]
    R --> A[Agent]
    A --> T[Tool Call]
    T --> C[Tool reads ctx.context]
    A --> G[Guardrail reads ctx.context]
    A --> H[Hook reads ctx.context]
    C --> R2[Result]
    G --> R2
    H --> R2
    R2 --> O[Response]
```

## 🛠️ Core Principles & Code Snippets

### 1. Define a typed context dataclass
Using a dataclass or Pydantic model gives you autocomplete and runtime validation.

```python
from dataclasses import dataclass

@dataclass
class AppContext:
    user_id: str
    tenant_id: str
    api_token: str
    db_session: Any  # replace with your session type
```

### 2. Pass context into Runner.run
```python
from agents import Runner

ctx = AppContext(
    user_id="user-123",
    tenant_id="tenant-42",
    api_token="...",
    db_session=db_session,
)

result = await Runner.run(agent, "What is my account balance?", context=ctx)
```

### 3. Access context inside a tool
Tools receive a `RunContextWrapper` as their first argument. The user-defined context lives on `ctx.context`.

```python
from agents import function_tool, RunContextWrapper

@function_tool
async def fetch_balance(ctx: RunContextWrapper[AppContext], account_type: str) -> str:
    user_id = ctx.context.user_id
    tenant_id = ctx.context.tenant_id
    db = ctx.context.db_session

    balance = await db.fetch_one(
        "SELECT balance FROM accounts WHERE user_id = ? AND tenant_id = ?",
        user_id, tenant_id,
    )
    return f"{account_type} balance: ${balance}"
```

> [!tip] Implementation Note
> Always type-hint `RunContextWrapper[AppContext]`. It documents the contract and gives you IDE support. Forgetting the wrapper type leads to confusing errors when you try to access `ctx.context`.

### 4. Access context inside guardrails
Guardrails follow the same pattern. Input guardrails receive the input plus the wrapper; output guardrails receive the output plus the wrapper.

```python
from agents import input_guardrail, RunContextWrapper

@input_guardrail
async def tenant_guardrail(ctx: RunContextWrapper[AppContext], agent, input):
    if ctx.context.tenant_id not in allowed_tenants:
        return GuardrailFunctionOutput(
            output_info=None,
            tripwire_triggered=True,
        )
    return GuardrailFunctionOutput(tripwire_triggered=False)
```

### 5. Access context inside hooks
Lifecycle hooks also receive the wrapper.

```python
from agents import RunHooks

class AuditHooks(RunHooks):
    async def on_tool_start(self, ctx: RunContextWrapper[AppContext], agent, tool):
        print(f"[{ctx.context.user_id}] Calling {tool.name}")

agent = Agent(
    name="Banking agent",
    instructions="Help users with account questions.",
    tools=[fetch_balance],
    hooks=AuditHooks(),
)
```

## 🧰 Techniques for Managing Context

### Technique 1: Request-scoped context factory
Create a fresh context object for each incoming request. This is the safest pattern for multi-tenant or multi-user apps.

```python
async def handle_user_request(user_id: str, tenant_id: str, user_input: str):
    async with db.session() as session:
        ctx = AppContext(
            user_id=user_id,
            tenant_id=tenant_id,
            api_token=await token_store.get(user_id),
            db_session=session,
        )
        result = await Runner.run(agent, user_input, context=ctx)
        return result.final_output
```

### Technique 2: Context as a dependency container
Group related clients and configuration into the context so tools stay focused on business logic.

```python
@dataclass
class ServiceContext:
    db: Database
    cache: CacheClient
    audit: AuditLogger
    config: AppConfig

@function_tool
async def lookup_user(ctx: RunContextWrapper[ServiceContext], email: str):
    cached = await ctx.context.cache.get(email)
    if cached:
        return cached
    user = await ctx.context.db.users.find_by_email(email)
    await ctx.context.cache.set(email, user)
    await ctx.context.audit.log("user_lookup", email=email)
    return user
```

### Technique 3: Immutable context updates
Treat the context as immutable. If a tool needs to influence later behavior, return the value in the tool result and let the agent decide, or use a fresh context on the next `Runner.run` call.

```python
# Good: tool returns data, does not mutate context
@function_tool
async def get_budget(ctx: RunContextWrapper[AppContext]) -> str:
    return f"Budget: ${ctx.context.budget}"

# Risky: mutating the context inside a tool
@function_tool
async def deduct_budget(ctx: RunContextWrapper[AppContext], amount: int) -> str:
    ctx.context.budget -= amount  # side effect; hard to reason about
    return f"Remaining: ${ctx.context.budget}"
```

### Technique 4: Layered context with optional fields
For shared tooling across agents that need different data, use a single context with optional fields and validate at the point of use.

```python
@dataclass
class SharedContext:
    user_id: str
    admin_mode: bool = False
    override_model: str | None = None
```

### Technique 5: Context propagation through handoffs
When one agent hands off to another, the same `RunContextWrapper` travels with the run. The target agent and its tools receive the same context.

```python
triage_agent = Agent(
    name="Triage",
    instructions="Route the user.",
    handoffs=[billing_agent],
)

# billing_agent's tools will see the same AppContext
```

## ⚠️ Pitfalls & Anti-patterns
- **Using global state instead of context:** Globals leak between concurrent requests and break unit tests. Pass request-scoped data through `context`.
- **Untyped context (plain dict):** A `dict` works, but you lose autocomplete, validation, and refactoring safety. Prefer dataclasses or Pydantic models.
- **Storing mutable shared resources without care:** A database connection pool can live on the context, but a single connection should be request-scoped. Sharing connections across concurrent runs causes race conditions.
- **Mutating context inside tools or guardrails:** The SDK does not forbid it, but it makes runs harder to reason about and debug. Return data instead.
- **Forgetting the wrapper type:** Writing `def tool(ctx: AppContext, ...)` instead of `def tool(ctx: RunContextWrapper[AppContext], ...)` means you receive the wrapper object, not your data class, and `ctx.user_id` will fail.
- **When NOT to use context:** For truly global configuration that never changes per request, environment variables or a module-level config object are simpler. Context is for request-scoped state.

## 🔗 Deep Connections
- **Parent Topic:** [[OpenAI-Agent-SDK-Overview]], [[The Agent]]
- **Related Concepts:**
    - [[Handoffs]] — Context propagates automatically when agents hand off
    - [[Guardrails]] — Guardrails can use context for tenant/user-specific checks
    - [[Decorator-Based Tool Registry]] — Tools receive the context wrapper as their first argument
    - [[LLM Tool Use - Function Calling]] — General tool-call patterns
- **Prerequisites:** Python dataclasses, OpenAI Agents SDK basics, async/await

## ❓ Active Recall Questions
> [!faq]- What is the purpose of the `context` argument in `Runner.run(...)`?
> It provides a typed, request-scoped object that tools, guardrails, and hooks can access through `RunContextWrapper.context`.

> [!faq]- How do you access the context inside a tool?
> The first parameter of a tool is a `RunContextWrapper[YourContextType]`. Your custom object is available as `ctx.context`.

> [!faq]- Does the context object survive across multiple `Runner.run` calls?
> No, it is request-scoped. Each `Runner.run` invocation receives its own context object. If you need persistence, store state outside the run and pass it in next time.

> [!faq]- Why should the context be immutable inside tools?
> Mutating the context creates hidden side effects that are hard to trace, test, and reproduce. Return values through tool results instead.

> [!faq]- What happens to context during a handoff?
> The same `RunContextWrapper` is propagated to the target agent, so its tools and guardrails see the same request-scoped state.

---
## 📝 How to use this template effectively
1. **Never Copy-Paste:** If you find yourself copying documentation, stop. Read, close the tab, and write it from memory.
2. **Iterate:** Fill the 'ELI5' and 'Analogy' first. If you can't explain it simply, you don't understand it yet.
3. **The "Why" is King:** Seniority is defined by knowing *when* and *why* to use a tool, not just *how*.
