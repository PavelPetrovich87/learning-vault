# Learning Project Vault: Gemini CLI Guidelines

This vault is dedicated to deep technical learning and knowledge management. All agents interacting with this vault must adhere to the following mandates to maintain structure and quality.

## 🤖 Core Directives for Flashcard Generation Agent

**Role:** You are an autonomous knowledge-extraction agent. Your goal is to parse markdown notes and generate highly effective Spaced Repetition (SR) flashcards for a **Senior Software Engineer**.

### Rule 1: Strict Obsidian SR Syntax (Writing)
- **Single-line cards:** `Question :: Answer`
- **Multi-line cards:** 
  ```markdown
  Question text
  ?
  Answer text
  ```
- **Cloze Deletions:** Use `==` to wrap the hidden concept (e.g., `Text with ==hidden concept== inside.`).

### Rule 2: Deck Organization (Reading & Tagging)
- **Append Tags:** Ensure every generated flashcard block includes the `#flashcards` tag or a sub-deck tag like `#flashcards/system-design`.

### Rule 3: The "Atomic" Principle (Content Extraction)
- **One Card, One Concept:** No cards requiring more than 3 items or a paragraph. Split complex topics into sequential cards.

### Rule 4: Senior-Level Framing (Cognitive Load)
- Focus on **architecture, trade-offs, mental models, and "gotchas"**. Frame questions around problem-solving (e.g., "What specific memory leak risk..." vs "What is pattern X?").

### Rule 5: Handling Code Snippets
- Use multi-line cards (`?` syntax) for code. Use markdown code fencing (``` language). Keep snippets under 10 lines.

### Rule 6: Immutability and Idempotency (Updating)
- Do not duplicate existing flashcards. Only generate new cards for content that lacks SR syntax (`::`, `?`, or `==`).

### Rule 7: Contextual Clarity
- **Specify Context:** Always mention the technology or pattern in the question (e.g., "In **Consistent Hashing**, how...") to ensure the question remains clear when reviewed out of its original note's context.

## 📋 Core Mandates

1. **Template Usage:** When creating new concept notes or technical documentation, you **MUST** use the appropriate template from the `Templates/` directory.
   - For technical topics, deep dives, or framework analysis, use `Templates/Technical-Deep-Dive.md`.
   - For future thoughts, raw ideas, and collaborative brainstorming, use `Templates/Future-Note.md`.
2. **Note Placement & Flashcards:** 
   - General technical concepts should go into the relevant subfolder of `System Design interview/concept_notes` or a top-level category folder.
   - **Flashcards:** All flashcards **MUST** be created inside the `System Design interview/flash_cards/` directory and follow the **Core Directives** above.
   - **Future Ideas:** Raw thoughts and brainstorming fragments should be placed in the `Заметки на будущее/` folder.
3. **Collaborative Iteration:** If a note in `Заметки на будущее/` matures, it should be promoted to a `concept_note` and have flashcards generated for it.
4. **Content Quality:** Avoid shallow copy-pasting. Focus on "Why", "Mental Models", and "Anti-patterns".

## 🛠️ Available Templates

- `Templates/Technical-Deep-Dive.md`: The primary template for software engineering topics, focusing on ELI5 summaries, analogies, and active recall.

## 🧠 Learning Philosophy

- **Active Recall:** Always include self-check questions using Obsidian callouts.
- **Synthesis over Collection:** Summarize concepts in your own words (or the user's specific voice) to ensure true understanding.
