# Week 2 Journal — Scaffolding

## Progress Tracker

| Day | Topic                    | Status         | Date   |
| --- | ------------------------ | -------------- | ------ |
| 1   | Theory & Mental Models   | ✅ Done         | Mar 24 |
| 2   | Repo Architecture & Docs | 🟡 In progress | Mar 25 |
| 3   | Linting & Formatting     | ⬜ Not started  |        |
| 4   | Tailwind + shadcn/ui     | ⬜ Not started  |        |
| 5   | CI/CD + Storybook        | ⬜ Not started  |        |
| 6   | Agent Test Drive + Retro | ⬜ Not started  |        |

**Legend:** ⬜ Not started · 🟡 In progress · ✅ Done

---

## Day 1 — Theory & Mental Models
**Date:**

###  https://bitloops.com/resources/governance/architectural-constraints-for-ai-agents 
**Key Takeaways**

- Architectural decay is silent. In the AI era the code enthropy is very fast. Refactoring becomes harder. Architectural Debt makes code messy and ununderstandable for AI as well as for devs. But devs at least have their experience and feeling (if they work a certain amount of time with this codebase). AI - does not, it works with tokens.
- We need to put CLEAR ARCHITECTURAL CONSTRAINTS. Here are examples for DDD:
	  - **Layer boundary constraints**
	  - **Dependency direction constraints**
	  - **Module isolation constraints**
	  - **Naming and symbol constraints**
	  - **Circular dependency constraints**
	  - **Pattern enforcement constraints**
- Types of rules:
	  - **Static analysis rules** (ESLint with custom rules) 
	  - **Custom validators** are rules specific to your domain
	  - **Dependency graph analysis** builds a map of what depends on wha
	    
#### - Common Architectural Constraints in Practice
	  - **Presentation → Domain → Infrastructure**
	  - **Module isolation**
	  - **No circular dependencies**
	  - **Naming conventions as rules**
	  - **Security boundaries**
	  - **Service boundaries**
#### - Enforcement Mechanisms
	 - **Static analysis before merge** (The feedback needs to be specific)
	 - **Custom validators in the AI workflow** (The agent generates a function, a validator checks it against your rules) 
	  - **Runtime guards for critical constraints.** (middleware that audits requests)

## https://refine.dev/blog/shadcn-blog/ # AI-First UIs

 - "Code ownership" vs "dependency" как принцип scaffolding
 - Declarative > Imperative для AI (Tailwind-классы (text-blue-500,p-4) — это по сути декларативный язык: "что я хочу", а не "как это сделать". Агенту проще манипулировать строкой классов, чем вызыватьtheme.override({ palette: {primary: ... } }))
 - Reduced abstraction = reduced hallucination surface

#### https://dev.to/zetsubo/how-should-css-architecture-evolve-in-the-age-of-ai-coding-agents-317k ZETSUBO про конкретную имплементацию харнесс фреймоврка

- Конвенции - это хорошо, но у агентов ограниченный контекст и лучше давать им фидбек луп
- ЦСС легко дрифтует а с помощью агентов вообще моментально, если у человека что-то в голове есть какие-то конвенции - у ИИ агентов конвенции огут слетать галлюцинировать, а главное - они склонны делать задачу любой ценой
- Необходимо строить фидбэк системы
- Виды дрифта:
	- - **Component boundary ambiguity**: Where one component ends and another begins differs from person to person
	- - **Layout responsibility confusion**: Whether margin belongs to parent or child varies file by file
	- - **Naming and structural inconsistency**: Naming conventions and file organization vary across the project
- Необходимо выделить инварианты своей CSS архитектуры - вот какие проперти должны быть у инвариантов:
	1. **Binary evaluation**: Violation or compliance is determined unambiguously (no gray zones)
	2. **Syntax-verifiable**: Can be determined through source code syntax analysis alone (no runtime needed)
	3. **Locally verifiable**: Can be determined by looking at the target file alone (no project-wide scanning needed)
 - 2 подхода в фидбэке
	 - Предварительный прогрев
	 - непосредственно фидбэк
		1. **What's wrong**: `margin-top` is in the wrong place
		2. **Why it's wrong**: Item properties must be specified from the parent
		3. **How to fix it**: Write it in the parent Block's `> .child-name` selector
		Структура фидбэк лупа:
			1. Generate SCSS
			2. Run `stylelint`
			3. Parse output and fix according to error messages
			4. Run `stylelint` again → repeat until zero errors
### Questions / Gaps

А тепреь нам из всего этого надо создать метаанализ, а еще лучше применить на конкретной архитектуре FSD

### Day 1 — Final Takeaways (The Self-Enforcing Repository)

1. **Minimize hallucination surface** — fewer abstractions, declarative tools (Tailwind > CSS-in-JS), code ownership over dependencies (shadcn model). In FSD context: flat public APIs per slice, no deep re-exports.
2. **Define invariants, not conventions** — rules must be binary, syntax-verifiable, locally verifiable. In FSD: "feature/ cannot import from another feature/" is an invariant; "keep files small" is a convention → discard it.
3. **Build convergent feedback loops** — write → lint → error with fix instructions → fix → re-lint → zero errors. The agent doesn't need to memorize rules, it just needs to reach exit code 0.
4. **Drift is the enemy, not bugs** — AI amplifies architectural drift at scale. Every rule that isn't enforced by tooling WILL be violated. The harness must catch drift before merge.
5. **Apply to FSD**: Layer boundaries (app → pages → widgets → features → entities → shared) are perfect invariants. Cross-slice imports are detectable by static analysis. This is our Day 2 blueprint.

---

## Day 2 — Repo Architecture & Docs
**Date:**

### What I Built


### Decisions Made & Why


---

## Day 3 — Linting & Formatting
**Date:**

### Config Choices & Rationale


### Gotchas


---

## Day 4 — Tailwind + shadcn/ui
**Date:**

### Components Added


### Notes


---

## Day 5 — CI/CD + Storybook
**Date:**

### Pipeline Design


### Storybook Notes


---

## Day 6 — Agent Test Drive + Retrospective
**Date:**

### Prompt Given to Agent


### What Worked


### What Broke / Was Missing


### Changes for Next Time

