---
status: analyzing
tags: [agentic-ai, openai-sdk, pydantic, validation, structured-output]
created: 2026-07-01
updated: 2026-07-01
difficulty: intermediate
source: https://docs.pydantic.dev/latest/
---

# 🧠 Topic: Pydantic

> [!abstract] Executive Summary (ELI5)
> Pydantic is a Python library that lets you define data shapes using classes. When data arrives, it checks the types, converts values when safe, and raises clear errors if something doesn't match. It is the de facto contract language for OpenAI structured outputs.

---

## 🎯 The "Why"
- **Problem Solved:** Python is dynamically typed. Passing raw dictionaries from an LLM into your business logic leads to `KeyError`, wrong types, and silent bugs. Pydantic gives you runtime validation, type coercion, and self-documenting schemas.
- **Alternatives:**
  - `dataclasses` + manual validation — lighter, but you write all coercion and error handling yourself.
  - `attrs` — powerful, but less ecosystem integration with OpenAI SDK and FastAPI.
  - JSON Schema directly — works for OpenAI API, but harder to read and maintain in Python.
- **Trade-offs:**
  - Pydantic adds a dependency and a small runtime cost for model validation.
  - Very strict defaults can be surprising for newcomers (e.g., extra fields rejected by default in V2).
  - Version 2 changed internals significantly; libraries must be compatible with `pydantic>=2`.

## 💡 Mental Model & Analogy
- **Analogy:** Pydantic is like a bouncer at a club. The bouncer has a guest list (the schema), checks IDs (types), turns away people who don't qualify (validation errors), and only lets in the right guests (validated objects).
- **Visual/Diagram:**
```mermaid
graph LR
    A[Raw Input dict | JSON] --> B[Pydantic Model]
    B -->|valid| C[Typed Object]
    B -->|invalid| D[ValidationError]
```

## 🛠️ Core Principles & Code Snippets
*Focus on "The Right Way" (Best Practices).*

```python
from pydantic import BaseModel, Field, validator
from typing import Literal

class RiskLevel(BaseModel):
    level: Literal["low", "medium", "high"]
    score: int = Field(ge=0, le=100, description="Risk score from 0 to 100")

class ComplianceCheck(BaseModel):
    passed: bool
    risk: RiskLevel
    notes: str | None = None

# Validation happens automatically
check = ComplianceCheck.model_validate({
    "passed": True,
    "risk": {"level": "low", "score": 15},
    "notes": "No issues found."
})
```

> [!tip] Implementation Note
> Prefer `model_validate` (Pydantic V2) or `parse_obj` (V1) over direct instantiation when the input is a dictionary. It runs the full validation pipeline and gives better error messages.

Key principles:
1. **Models are schemas.** Each field is a type, a default, and a description. Use `Field(...)` for constraints and documentation.
2. **Composability.** Nest models inside other models to build complex structures cleanly. This is exactly how OpenAI structured output schemas should be built.
3. **Validation is not parsing.** Pydantic coerces compatible types (`"42"` -> `42` for `int`) but rejects truly wrong data. Do not rely on it as a security sanitizer.
4. **Use `Literal` for enums.** It is concise, type-safe, and serializes cleanly to JSON Schema for the LLM.
5. **Custom validators.** Use `@field_validator` (V2) for cross-field logic, but keep most rules declarative to keep schemas readable by the model.

## ⚠️ Pitfalls & Anti-patterns
- **Common Mistakes:**
  - Using overly permissive types like `Any` or `dict` in fields you want the LLM to populate — defeats the purpose of structured output.
  - Forgetting that V2 uses `model_dump` instead of V1's `dict`, and `model_validate` instead of `parse_obj`.
  - Over-engineering with `@model_validator` for simple constraints that `Field` already supports.
- **When NOT to use:**
  - Static config objects where a `TypedDict` or simple dataclass is enough.
  - Hot paths where microseconds matter and you can trust the input shape.
- **Performance/Security considerations:**
  - Large nested models can be expensive to validate at high throughput.
  - Validate untrusted input *before* it reaches your database or downstream services.
  - Be careful with `json.loads` on raw LLM output — validate through Pydantic rather than trusting the JSON structure.

## 🔗 Deep Connections
- **Parent Topic:** [[OpenAI-Agent-SDK-Overview]]
- **Related Concepts:** [[Structured Output]], [[LLM Tool Use - Function Calling]], [[Decorator-Based Tool Registry]]
- **Prerequisites:** Python type hints, JSON Schema basics, OpenAI SDK client setup

## ❓ Active Recall Questions
> [!faq]- Why does the OpenAI SDK use Pydantic models for `response_format`?
> Pydantic models can be automatically converted to JSON Schema, enforced at runtime, and returned as typed Python objects.

> [!faq]- What happens if a Pydantic model field fails validation during OpenAI structured output?
> The SDK raises a `ValidationError` or the API refuses the schema if it cannot be represented as JSON Schema. In your code, catch `ValidationError` and log/escape the bad response.

> [!faq]- What is a common migration pitfall between Pydantic V1 and V2?
> Method names changed: `dict()` -> `model_dump()`, `parse_obj()` -> `model_validate()`, and `validator` -> `field_validator`. Also V2 validates extra fields more strictly by default.
