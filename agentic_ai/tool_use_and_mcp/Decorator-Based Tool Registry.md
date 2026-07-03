---
status: analyzing
mastery: beginner
tags: [agentic-ai, llm, tool-registry, decorator, patterns, evergreen]
source: Lab 4 "Professionally You" — Agentic Engineering
created: 2026-06-24
updated: 2026-06-24
difficulty: intermediate
---

# Decorator-Based Tool Registry

> [!abstract] Executive Summary (ELI5)
> Instead of writing a long `if/elif` chain to decide which tool to run, you mark each tool function with a decorator. The agent automatically discovers every marked function, builds the list of tools, and knows how to call them.

---

## 🎯 The "Why"

- **Problem Solved:** As the number of tools grows, hardcoded dispatch becomes a maintenance nightmare. A registry centralizes discovery, schema generation, and execution so adding a new tool is just adding a new decorated function.
- **Alternatives:**
  - **Hardcoded `if/elif` dispatch** — simple for 1–2 tools, brittle for many.
  - **Manual tool list assembly** — each tool is registered by hand in a central list. Easy to forget one.
  - **Plugin/discovery via file system** — scan a directory for modules. More decoupled but harder to control load order and dependencies.
  - **Framework-managed registries** (LangChain, LlamaIndex) — batteries included, but add coupling to the framework.
- **Trade-offs:** A decorator registry adds a small amount of upfront complexity but pays off quickly beyond three tools. It keeps tool logic co-located with metadata, which improves discoverability.

## 💡 Mental Model & Analogy

- **Analogy:** "This is like a restaurant kitchen where every chef hangs their specialty on a board as soon as they arrive. The expediter doesn't need to know every chef personally; they just read the board and route orders."
- **Visual/Diagram:**

```mermaid
graph LR
    A[@register_tool] -->|decorates| B[Tool Function]
    B -->|auto-discovered| C[Registry]
    C -->|schemas| D[LLM]
    D -->|tool_call| C
    C -->|dispatches| B
    B -->|result| E[Agent Loop]
```

## 🛠️ Core Principles & Code Snippets

### 1. The registry object

A simple global registry collects decorated tools and their schemas.

```python
from functools import wraps

REGISTRY = {}


def register_tool(name=None, description=None):
    def decorator(func):
        tool_name = name or func.__name__
        tool_description = description or func.__doc__ or ""
        REGISTRY[tool_name] = {
            "function": func,
            "name": tool_name,
            "description": tool_description,
            "parameters": infer_schema(func),  # or typed dict
        }
        return func
    return decorator
```

### 2. Declaring a tool

The business logic and metadata live in one place.

```python
@register_tool(
    name="record_user_details",
    description="Record contact details when a visitor wants to get in touch."
)
def record_user_details(email: str, name: str = None, phone: str = None):
    if not is_valid_email(email):
        return {"error": "Invalid email"}
    save_to_crm(email=email, name=name, phone=phone)
    send_push_notification(f"New lead: {email}")
    return {"status": "recorded"}
```

### 3. Building the OpenAI-compatible tool list

Convert the registry to the format the LLM expects.

```python
def get_openai_tools():
    return [
        {
            "type": "function",
            "function": {
                "name": meta["name"],
                "description": meta["description"],
                "parameters": meta["parameters"]
            }
        }
        for meta in REGISTRY.values()
    ]
```

### 4. Dispatch

Execute the right function by name without any `if/elif`.

```python
def execute_tool(call):
    meta = REGISTRY.get(call.function.name)
    if not meta:
        return {"error": f"Unknown tool: {call.function.name}"}
    args = json.loads(call.function.arguments)
    return meta["function"](**args)
```

## 🏗️ Complexity Spectrum

| Level | Approach | When to Use |
|-------|----------|-------------|
| **Level 1: Hardcoded dispatch** | `if name == "x": ...` | 1–2 tools, prototypes |
| **Level 2: Decorator registry** | `@register_tool` auto-discovery | 3–20 tools, production agents |
| **Level 3: Class-based tools** | Each tool is a class with `run()` and `schema()` | Complex tools with shared state |
| **Level 4: Plugin system** | Tools loaded from separate packages or remote services | Platform with third-party tools |

## 🏢 Production Examples

- **Internal admin copilot** — 15+ tools for reading logs, restarting services, checking user accounts.
- **Customer support agent** — tools for order lookup, refund, escalation, and KB search.
- **Lab 4 chatbot** — `record_user_details` and `record_unknown_question` registered via decorator.
- **Data analysis assistant** — tools for SQL queries, chart generation, and CSV export.
- **DevOps bot** — tools for Terraform plan, Kubernetes rollout, and incident creation.

## ⚠️ Pitfalls & Anti-patterns

- **Global registry abuse:** A global `REGISTRY` is fine for small apps, but can make testing hard. Allow explicit registry instances.
- **Name collisions:** Two tools with the same name overwrite each other. Fail fast or namespace tools.
- **Heavy functions in decorators:** Decorators run at import time. Don’t perform I/O or slow initialization there.
- **Schema drift:** If you infer schemas from type hints, ensure optional fields and descriptions are accurate.
- **Tight coupling to one LLM format:** Keep the registry format generic and convert to OpenAI/Anthropic/Google formats at the edges.

## 🔗 Deep Connections

- **Parent Topic:** [[Agentic Workflow Patterns Overview]], [[LLM System Design]]
- **Related Concepts:** [[LLM Tool Use / Function Calling]], [[ReAct Loop]], [[Plugin Architecture]], [[Dependency Injection]]
- **Prerequisites:** Python decorators, JSON schema, function dispatch patterns

## ❓ Active Recall Questions

> [!faq]- What problem does a tool registry solve compared to if/elif dispatch?
> It decouples tool definition from tool execution. Adding a new tool doesn’t require editing the dispatcher.

> [!faq]- Why keep the registry format generic instead of OpenAI-specific?
> Different LLM providers use slightly different tool schemas. A generic registry lets you swap providers without rewriting tool definitions.

> [!faq]- What is a common testing issue with global registries?
> Tests can pollute each other because tools persist across test cases. Use per-instance registries or clear the registry between tests.

## Related Notes

- [[LLM Tool Use / Function Calling]]
- [[ReAct Loop]]
- [[Agentic Workflow Patterns Overview]]
