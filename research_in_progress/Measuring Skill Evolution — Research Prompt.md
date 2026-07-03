---
status: researching
tags: [research, skill-evolution, metrics, evaluation, agent-harness]
created: 2026-06-10
updated: 2026-06-10
source:
  title: "Measuring Skill Evolution: A Deep Research Prompt"
  author: Self-directed research
  url: n/a
  date_read: 2026-06-10
---

# Measuring Skill Evolution — Research Prompt

> [!abstract] TL;DR
> Deep research prompt on how to measure the evolution and improvement of LLM-based skills (specifically agent-readiness classifier skills). This note frames the research questions, identifies measurement challenges, and outlines evaluation frameworks applicable to skills that produce structured text output (not code, not files).

---

## 🔬 Фаза 1: EXTRACTION — Контекст и постановка проблемы

### What Is Being Measured

We are building a **skill** (set of LLM instructions) that:
- Reads a repository's file structure and key files
- Classifies the repository across 10 axes (project_class, language, framework, build_system, test_framework, package_manager, monorepo, agents_md_quality, doc_richness, conventions)
- Detects the current maturity level (L0 Blind → L1 Oriented → L2 Self-Correcting → L3 Self-Maintaining)
- Identifies gaps between current and target level
- Generates a prioritized roadmap (1–5 items)
- Writes 2-3 paragraphs of actionable advice

**Output format:** Structured markdown text in chat (not files, not JSON).

**Skill lifecycle:** P0 (foundations) → P1 (full classifier) → P2 (walking skeleton) → P3 (maturity engine) → P4 (context authoring) → P5+ (sub-skills for specific axes).

### The Core Problem

**How do we know the skill is getting better with each iteration?**

Unlike traditional software where tests pass/fail, LLM-based skills produce:
- **Subjective output** (advice quality, relevance)
- **Probabilistic output** (same input → different outputs)
- **Context-dependent output** (what's "good advice" varies by repo)
- **Multi-dimensional output** (classification + gap detection + roadmap + advice)

We need a measurement framework that can:
1. Compare version N vs version N+1 of the skill
2. Detect regressions (new version is worse in some dimension)
3. Measure progress toward the "north star" (mature-reference repo should get minimal, accurate gaps)
4. Attribute improvement to specific changes (did P3 upgrade help?)

---

## 🧠 Фаза 2: ANALYSIS — Вопросы для глубокого исследования

### 2.1 Classification Accuracy Metrics

How do we measure whether the classifier correctly identifies repository characteristics?

**Questions:**
- [ ] What metrics apply to multi-label classification with 10 axes where some values are categorical (language: typescript|javascript|python) and some are ordinal (agents_md_quality: none|minimal|partial|comprehensive)?
- [ ] Is simple accuracy sufficient, or do we need per-axis precision/recall/F1?
- [ ] How do we handle "acceptable synonyms" (e.g., `api-backend` vs `backend-api` vs `rest-api`)?
- [ ] What about `undetermined` — is it a valid output or a failure mode?
- [ ] How do we aggregate per-axis scores into a single "classification quality" score?
- [ ] Can we use confusion matrices for categorical axes? What about ordinal axes?

**Research directions:**
- Look at Named Entity Recognition (NER) evaluation metrics — similar to multi-axis classification
- Study hierarchical classification evaluation (axes have dependencies: monorepo type affects agents_md_quality assessment)
- Investigate "soft matching" for categorical values (semantic similarity vs exact match)

### 2.2 Gap Detection Quality

How do we measure whether identified gaps are real, relevant, and complete?

**Questions:**
- [ ] What is the "ground truth" for gaps? Our Test Corpus Specification defines expected gaps, but is this sufficient?
- [ ] How do we measure **false positives** (gap reported but not real)?
- [ ] How do we measure **false negatives** (real gap missed)?
- [ ] How do we handle **priority ordering** — is gap #1 more important than gap #2? Can we measure ranking quality (NDCG, Kendall's tau)?
- [ ] What about **gap specificity** — "add tests" vs "add Vitest with at least one integration test for the API routes"?
- [ ] How do we measure **actionability** — can a human actually act on this gap?

**Research directions:**
- Study information retrieval metrics (precision@k, recall@k, MAP)
- Look at checklist evaluation frameworks (medical checklists, code review checklists)
- Investigate "task-oriented evaluation" — can the gap be turned into a concrete task?

### 2.3 Level Recommendation Accuracy

How do we measure whether the recommended next level is appropriate?

**Questions:**
- [ ] Is level recommendation a classification problem (predict correct next level) or a regression problem (predict maturity score)?
- [ ] How do we define "appropriate" — is it what WE think, or what an expert would think?
- [ ] What about **over-recommendation** (L0→L2) vs **under-recommendation** (L2→L1)? Which is worse?
- [ ] Can we define a "cost matrix" for wrong recommendations (e.g., over-recommending is more harmful than under-recommending)?
- [ ] How do we validate the ladder itself — are L0→L1→L2→L3 transitions actually the right granularity?

**Research directions:**
- Study ordinal regression evaluation (our levels are ordered)
- Look at recommendation system evaluation (precision/recall for level "items")
- Investigate "human preference alignment" — do experts agree with the skill's recommendations?

### 2.4 Advice Quality (The Hardest Problem)

How do we measure the quality of 2-3 paragraphs of actionable advice?

**Questions:**
- [ ] Can we use **LLM-as-a-judge** — another LLM evaluates the advice? What are the failure modes of this approach?
- [ ] What dimensions matter: accuracy, relevance, specificity, actionability, completeness, conciseness, tone?
- [ ] How do we measure **consistency** — same repo, same skill version, does advice vary wildly?
- [ ] Can we use **reference-based metrics** (BLEU, ROUGE) comparing to human-written "golden advice"?
- [ ] Are **reference-free metrics** better (e.g., BERTScore, GPT-4 evaluation)?
- [ ] How do we measure **impact** — does following the advice actually help an agent work better?

**Research directions:**
- Deep dive into LLM evaluation methodologies (LMSYS, AlpacaEval, MT-bench)
- Study text generation evaluation beyond n-gram overlap (BERTScore, BARTScore)
- Investigate "human-in-the-loop" evaluation frameworks (how many human evaluators needed?)
- Look at "advice quality" research in expert systems and decision support

### 2.5 Consistency and Reproducibility

How do we ensure the skill produces consistent output?

**Questions:**
- [ ] What is acceptable variance? If 2 runs on the same repo produce slightly different evidence wording, is that OK?
- [ ] How do we measure **structural consistency** (same axes, same gap count) vs **semantic consistency** (same meaning, different words)?
- [ ] Can we use **embeddings** to measure semantic similarity between outputs?
- [ ] What temperature/settings should we use for evaluation runs?
- [ ] How many runs per repo per version do we need for statistical significance?

**Research directions:**
- Study test-retest reliability in psychometrics
- Look at prompt engineering consistency research
- Investigate "deterministic LLM output" techniques (seed, temperature=0, constrained decoding)

### 2.6 Signal-to-Noise Ratio

How do we measure whether the skill produces signal vs noise?

**Questions:**
- [ ] Define **signal** — accurate classification, real gaps, actionable advice
- [ ] Define **noise** — hallucinated axes, fake gaps, generic advice, irrelevant recommendations
- [ ] Can we create a **noise taxonomy**? (hallucination, over-generalization, scope creep, ladder violation)
- [ ] How do we measure **signal density** — useful information per token?
- [ ] Can we use an **information-theoretic approach** (entropy, mutual information)?

**Research directions:**
- Study hallucination detection in LLMs
- Look at "fluency vs accuracy" trade-offs in text generation
- Investigate "confidence calibration" — can the skill express uncertainty?

### 2.7 Regression Detection

How do we catch when a new version is worse?

**Questions:**
- [ ] What is our **regression test suite** — all 6 repos from Test Corpus, or a subset?
- [ ] How do we define **"worse"** — lower score on any metric, or only on weighted aggregate?
- [ ] Can we use **diff-based evaluation** — compare outputs side-by-side and flag changes?
- [ ] Should we maintain a **golden output set** — expected output for each repo at each version?
- [ ] How do we handle **intentional changes** — if we update the ladder, expected output changes too?
- [ ] Can we automate regression detection, or does it require human review?

**Research directions:**
- Study snapshot testing methodologies
- Look at "approval testing" (text-based regression testing)
- Investigate differential testing for LLMs

### 2.8 Evaluation Frameworks and Tools

What existing frameworks can we adopt or adapt?

**Questions:**
- [ ] **LLM-as-a-judge:** Can we build a meta-prompt that evaluates skill output? What are the prompt engineering challenges?
- [ ] **Human evaluation:** How many evaluators? What rubric? How to reduce bias and inter-rater disagreement?
- [ ] **Automated test corpus:** Is our 6-repo corpus sufficient? How to expand it? How to maintain ground truth?
- [ ] **A/B testing:** Can we run two skill versions side-by-side and compare? What's the statistical framework?
- [ ] **Trace analysis:** Can we analyze the skill's "thinking process" (if exposed) to find failure modes?
- [ ] **Benchmarking:** Can we contribute to/create an open benchmark for repo-readiness skills?

**Research directions:**
- Survey LLM evaluation frameworks (EleutherAI, Hugging Face Evaluate, OpenAI Evals)
- Study MT-bench / AlpacaEval / Arena methodology
- Look at "structured generation evaluation" (our output has schema)
- Investigate "eval-driven development" (EDD) for LLM applications

### 2.9 Longitudinal Tracking

How do we track skill evolution over time?

**Questions:**
- [ ] What **dashboard/metrics** do we need? (time-series of scores per axis?)
- [ ] How do we correlate **skill changes** (git commits to SKILL.md) with **metric changes**?
- [ ] What is the **minimum viable tracking** — spreadsheet, or dedicated tool?
- [ ] Can we use the skill on **real repositories** (not just test corpus) and track feedback?
- [ ] How do we incorporate **user feedback** (did the advice help?) into the metric?

**Research directions:**
- Study MLOps monitoring practices (model drift, performance degradation)
- Look at "prompt version control" best practices
- Investigate "feedback loops" in LLM systems

---

## 🛠️ Фаза 3: APPLICATION — Как применить к нашему проекту

### Immediate Application (P2 Stage)

**What we can do now with minimal effort:**
1. **Manual evaluation rubric** — Human reviews P2 output on greenfield-nextjs against a checklist
2. **Binary pass/fail** — Does output match expected format? Does it violate anti-patterns?
3. **Version snapshot** — Save expected output for each test repo; future versions must match or improve

### Short-Term (P1–P3)

**What to build next:**
1. **Automated scoring script** — Parse skill output, compare to expected values per axis
2. **LLM-as-judge meta-prompt** — A second LLM evaluates the quality of advice against rubric
3. **Consistency test** — Run skill 5× on same repo, measure variance
4. **Regression suite** — Run on all 6 repos after every SKILL.md change

### Long-Term (P4+)

**What to research:**
1. **Human evaluation protocol** — Recruit 3-5 developers to rate advice quality
2. **Impact measurement** — Track whether repos that follow advice become more "agent-friendly"
3. **Benchmark contribution** — Package test corpus + rubric as open benchmark

### Metrics We Need to Define

| Metric | Definition | Measurement Method | Target |
|--------|-----------|-------------------|--------|
| Classification Accuracy | % of axes with correct value | Automated comparison to ground truth | ≥90% |
| Gap Precision | % of reported gaps that are real | Human review (LLM-as-judge) | ≥85% |
| Gap Recall | % of real gaps that are reported | Human review (LLM-as-judge) | ≥80% |
| Level Accuracy | % of correct level recommendations | Automated (exact match) | ≥95% |
| Advice Quality | Holistic score 1-5 | Human eval or LLM-as-judge | ≥4.0 |
| Consistency Score | Semantic similarity across runs | Embedding cosine similarity | ≥0.90 |
| Anti-Pattern Violations | Count of forbidden recommendations | Automated pattern matching | 0 |
| Signal-to-Noise | Useful info / total tokens | Heuristic or LLM eval | ≥0.70 |

---

## ✅ Фаза 4: ACTION — Что дальше

### Research Tasks

- [ ] **Task 1:** Survey LLM evaluation methodologies (2-3 papers/frameworks)
  - Focus on: structured output evaluation, LLM-as-judge, text generation metrics
  - Deliverable: Summary note with applicable techniques

- [ ] **Task 2:** Design evaluation rubric for advice quality
  - Focus on: dimensions, scale, inter-rater reliability
  - Deliverable: Rubric document + example evaluations

- [ ] **Task 3:** Build automated scoring for classification axes
  - Focus on: parsing skill output, comparing to ground truth, generating score
  - Deliverable: Script + test run on 6-repo corpus

- [ ] **Task 4:** Test consistency (multiple runs, same repo)
  - Focus on: variance measurement, acceptable thresholds
  - Deliverable: Variance report + recommendations for settings

- [ ] **Task 5:** Design regression test protocol
  - Focus on: golden outputs, diff detection, approval workflow
  - Deliverable: Protocol document + PoC

### Open Questions for Deep Research

1. **Can we use the skill itself to evaluate itself?** (meta-cognitive loop — skill critiques its own output)
2. **What is the minimum test corpus size?** 6 repos, 20, 100? What statistical power do we need?
3. **How do we weight metrics?** Is classification accuracy more important than advice quality?
4. **Can we build a "skill fitness function"?** Single scalar that captures overall skill quality
5. **How do we measure "usefulness to the end user"?** Not just correctness, but value

### Related Notes

- [[Test Corpus Specification]] — ground truth definitions
- [[Test Corpus Workflow]] — validation process
- [[Agent Harness Skill — Implementation Plan]] — skill evolution roadmap
- [[P0 — Foundations Decisions]] — ladder definition, contracts

---

## 🗣️ Разговор с собой

*Это исследование критически важно, потому что без измерения мы не знаем, улучшаем ли скилл или деградируем. Особенно опасно при переходе от P2 к P1 — добавление новых осей может сломать то, что работало. Нужен "парашют" в виде regression suite.*

*Главный инсайт: для LLM-based skills традиционные unit-tests не работают. Нужен гибрид: automated structure checks + LLM-as-judge for semantic quality + human spot-checks for edge cases. Это больше похоже на evaluation of expert systems, чем на тестирование ПО.*

*Вопрос, который меня мучает: можем ли мы создать "self-improving" loop, где скилл сам оценивает свои ошибки и предлагает правки к SKILL.md? Это meta-skill — skill that improves skills.*