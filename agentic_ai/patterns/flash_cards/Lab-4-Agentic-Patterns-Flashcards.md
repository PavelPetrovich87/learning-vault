---
status: analyzing
mastery: beginner
tags: [flashcards, agentic-ai, llm, lab-4]
created: 2026-06-24
updated: 2026-08-21
---
<!-- ARCHIVED 2026-08-21: flashcards paused (outside the 4 active courses). Remove wrappers to restore. -->

# Lab 4 Agentic Patterns — Flashcards

<!-- Based on the concept notes for Lab 4 "Professionally You". -->

---

## LLM Tool Use / Function Calling

<!-- When should you prefer native function calling over structured output + hardcoded dispatch? -->
<!-- ? -->
<!-- When the LLM needs to reason about which action to take and adapt based on tool results. Structured output is enough when the action path is fixed. -->
<!-- #flashcards/agentic-ai -->

<!-- What makes a good tool description? -->
<!-- ? -->
<!-- It explains when to call the tool, what it does, and what each parameter means. Treat it as part of the prompt. -->
<!-- #flashcards/agentic-ai -->

<!-- Why must tool results be fed back to the LLM? -->
<!-- ? -->
<!-- The LLM needs the result to produce a final answer grounded in reality. Without it, the model would hallucinate the outcome. -->
<!-- #flashcards/agentic-ai -->

<!-- What is the runtime's responsibility when a tool is called? -->
<!-- ? -->
<!-- Validate arguments, execute the tool safely, handle errors, and return the result to the LLM. -->
<!-- #flashcards/agentic-ai -->

---

## ReAct Loop

<!-- What does ReAct stand for and what is the core loop? -->
<!-- ? -->
<!-- ReAct = Reason + Act. The LLM reasons, picks a tool, observes the result, and repeats until it can answer. -->
<!-- #flashcards/agentic-ai -->

<!-- How is ReAct different from Chain-of-Thought? -->
<!-- ? -->
<!-- Chain-of-Thought only produces reasoning text. ReAct couples reasoning with external actions and observations. -->
<!-- #flashcards/agentic-ai -->

<!-- Why is a max-iteration limit critical in a ReAct loop? -->
<!-- ? -->
<!-- Without it, a stuck or confused model can loop indefinitely, burning tokens and latency. -->
<!-- #flashcards/agentic-ai -->

<!-- What should a tool observation include to be useful? -->
<!-- ? -->
<!-- Clear, structured data that the LLM can interpret to drive the next reasoning step. -->
<!-- #flashcards/agentic-ai -->

---

## Lightweight Prompt-Injected RAG

<!-- When is prompt-injected RAG preferable over vector DB RAG? -->
<!-- ? -->
<!-- When the corpus is small, stable, and fits comfortably within the context window. -->
<!-- #flashcards/agentic-ai -->

<!-- What is the main scaling limitation of prompt-injected RAG? -->
<!-- ? -->
<!-- Context window size and cost. Every message resends the full corpus, and truncation loses facts. -->
<!-- #flashcards/agentic-ai -->

<!-- How do you prevent hallucination when the answer is not in the injected context? -->
<!-- ? -->
<!-- Give the model an explicit out, such as admitting ignorance or offering to record the question. -->
<!-- #flashcards/agentic-ai -->

<!-- What percentage of the context window should typically be reserved for the actual chat? -->
<!-- ? -->
<!-- A good rule of thumb is to reserve at least 30% of the context window for conversation history and response. -->
<!-- #flashcards/agentic-ai -->

---

## Persona Embodiment

<!-- What are the four components of a strong persona prompt? -->
<!-- ? -->
<!-- Identity, voice, goals, and boundaries. -->
<!-- #flashcards/agentic-ai -->

<!-- Why combine persona with RAG instead of relying on training data? -->
<!-- ? -->
<!-- Training data may be outdated or wrong about the person. RAG grounds answers in curated, current facts. -->
<!-- #flashcards/agentic-ai -->

<!-- What is persona drift and how do you mitigate it? -->
<!-- ? -->
<!-- Persona drift is when the model gradually stops acting in character. Mitigate by keeping the persona in the system prompt, using summaries, or re-injecting the persona. -->
<!-- #flashcards/agentic-ai -->

<!-- What is a common ethical risk of persona embodiment? -->
<!-- ? -->
<!-- Impersonating real people without consent or disclosure can mislead users and create legal liability. -->
<!-- #flashcards/agentic-ai -->

---

## Decorator-Based Tool Registry

<!-- What problem does a tool registry solve compared to if/elif dispatch? -->
<!-- ? -->
<!-- It decouples tool definition from execution, so adding a new tool doesn't require editing the dispatcher. -->
<!-- #flashcards/agentic-ai -->

<!-- Why keep the registry format generic instead of OpenAI-specific? -->
<!-- ? -->
<!-- Different LLM providers use different tool schemas. A generic registry lets you swap providers without rewriting tool definitions. -->
<!-- #flashcards/agentic-ai -->

<!-- What is a common testing issue with global registries? -->
<!-- ? -->
<!-- Tests can pollute each other because tools persist across test cases. Use per-instance registries or clear between tests. -->
<!-- #flashcards/agentic-ai -->

<!-- What should happen at import time in a decorator-based registry? -->
<!-- ? -->
<!-- Only registration metadata should be collected. Avoid I/O or heavy initialization. -->
<!-- #flashcards/agentic-ai -->

---

## Human-in-the-Loop for Agent Observability

<!-- What is the difference between observability and human-in-the-loop? -->
<!-- ? -->
<!-- Observability informs the human after the fact. Human-in-the-loop lets the human approve, reject, or modify actions. -->
<!-- #flashcards/agentic-ai -->

<!-- Why throttle notifications in a production agent? -->
<!-- ? -->
<!-- To prevent notification fatigue and reduce cost when an agent generates many events. -->
<!-- #flashcards/agentic-ai -->

<!-- What should a good agent notification include? -->
<!-- ? -->
<!-- What happened, what data was involved, when it happened, why it matters, and how to follow up. -->
<!-- #flashcards/agentic-ai -->

<!-- What is notification fatigue and how does it hurt an agent deployment? -->
<!-- ? -->
<!-- Too many pings cause the owner to mute the channel and miss real issues. -->
<!-- #flashcards/agentic-ai -->

---

## Cross-Cutting

<!-- What is the main advantage of combining ReAct with a tool registry? -->
<!-- ? -->
<!-- The agent can dynamically discover and call tools in a loop without hardcoded dispatch logic. -->
<!-- #flashcards/agentic-ai -->

<!-- In Lab 4, which pattern creates real-world observability for the owner? -->
<!-- ? -->
<!-- Human-in-the-loop notifications via Pushover fire on tool calls like recording a lead or unknown question. -->
<!-- #flashcards/agentic-ai -->

<!-- What is the simplest RAG approach used in Lab 4? -->
<!-- ? -->
<!-- Lightweight prompt-injected RAG: LinkedIn PDF and personal summary text are injected directly into the system prompt. -->
<!-- #flashcards/agentic-ai -->
