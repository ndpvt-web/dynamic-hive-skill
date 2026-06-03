# Hive Universal Agent Roster

This roster covers all task families. Roles are not hardcoded — select the ones that
match the actual cognitive work required. Custom roles are encouraged.

> **MODEL SELECTION IS NOT PINNED HERE.** Each role below lists its **dominant cognitive
> axis** and a *first-pick suggestion*, but the actual model is resolved by running the
> algorithm in **`resources/model-decision.md`** (read it FIRST). The suggestions are the
> cheapest model that typically clears the bar for that axis — step up exactly one tier on
> demonstrated failure, and apply the high-stakes override (start at Opus 4.8) for any role
> whose error is costly or hard to reverse. Two hard rules from the decision file:
> (1) **Opinion / unfiltered / 18+ / politics / economics content in ANY role → Grok 4.20.**
> (2) Never route by a model's lab/country or by a blanket default. Removed models
> (Gemini, GPT-4.1, gpt-5-mini, MiniMax M1, MiniMax M2.7) must not be assigned.
>
> **Needs-to-act gate:** if a role must edit files / run commands / call tools, its model
> must be a tool-loop agent (Opus / Sonnet / Haiku via `claude -p` or the Agent tool, or
> GPT-5.5 via Codex). If a role only *generates* text/code/analysis, the cheap text workers
> (DeepSeek V4 Pro, MiniMax M3) are first-class and usually win on cost — the orchestrator
> then performs any file writes. See `model-harness-guide.md` for launch commands.

## Table of Contents
1. [Universal Roles (always available)](#universal-roles)
2. [Knowledge Roles](#knowledge-roles)
3. [Construct Roles](#construct-roles)
4. [Transform Roles](#transform-roles)
5. [Decide Roles](#decide-roles)
6. [Technical Roles (when project has code/data)](#technical-roles)
7. [Role Selection Algorithm](#role-selection-algorithm)
8. [Wave Assignment Patterns](#wave-assignment-patterns)

---

## Universal Roles

These roles appear in almost every Hive project regardless of task family.

### Lead Architect (Phase 4 — performed by the orchestrator directly)
**Responsibility:** Designs the overall approach, generates the contract, defines deliverable
manifest, sets shared conventions (terminology, style, format).
**Axis → first pick:** high-stakes / single-point-of-failure → **Opus 4.8** (high-stakes override; errors cascade to every downstream agent).
**Harness:** The orchestrator (Claude Code) performs this role directly in Phase 4, either
inline (if current session is Opus) or via `claude -p --model claude-opus-4-8`. This is NOT a separately-spawned agent.
**Execution agents start at Wave 1 AFTER the contract exists.** The Lead Architect is complete
before any execution agents are spawned.

### Integration Lead (always final wave, solo)
**Responsibility:** Combines all agent outputs into the final unified deliverable. Resolves
inconsistencies, ensures stylistic coherence, validates completeness.
**Axis → first pick:** high-stakes synthesis + must act (writes the final files) → **Opus 4.8** (last line of defense; coherence errors are user-visible).
**Harness:** `general-purpose` (tool-loop; or `claude -p --model claude-opus-4-8`)
**Required when:** 3+ execution agents. Optional for simpler projects.

### Quality Critic
**Responsibility:** Reviews all outputs against quality criteria in the contract. Identifies
gaps, inconsistencies, errors. Does not fix — reports with specific, actionable findings.
**Axis → first pick:** high-stakes reasoning/critique → **Opus 4.8** (highest GPQA/HLE; a missed defect ships to the user).
**Harness:** `general-purpose` (or `Explore` to scan first, then `general-purpose`/`claude -p` to write the review)
**Wave:** Second-to-last (after all content agents, before Integration Lead)

---

## Knowledge Roles

### Research Analyst
**Responsibility:** Discovers, evaluates, and synthesizes information from multiple sources.
Primary intelligence-gathering role. Produces annotated findings, not raw data dumps.
**Axis → first pick:** web-research + must act (web search tools + write findings) → tool-loop agent, **Sonnet 4.6** (BrowseComp is tied within ~1pt across the top models, so the cheapest tool-capable agent suffices); step up to **Opus 4.8** for hard/high-stakes research. If the work is pure synthesis of already-gathered sources (no tools), a cheap text worker (DeepSeek V4 Pro) can generate and the orchestrator writes.
**Harness:** `general-purpose` (web search + write findings)
**Wave:** Early (Wave 2 in most projects)

### Domain Expert (instantiate with specific domain)
**Responsibility:** Deep expertise in a specific field. Named per domain:
"Climate Science Expert", "Financial Markets Expert", "Regulatory Expert", etc.
The specialization should be in the agent's name AND prompt.
**Axis → first pick:** reasoning/knowledge → start cheap (DeepSeek V4 Pro GPQA 90.1 if generation-only) or **Sonnet 4.6** if it must use tools; **high-stakes override → Opus 4.8** for security/legal/financial/medical domains where an error is costly. **Opinion/politics/economics domain → Grok 4.20.**
**Harness:** `general-purpose` (or gateway curl if generation-only)
**Wave:** Parallel with Research Analyst

### Fact Checker
**Responsibility:** Verifies claims, checks sources, flags unsubstantiated assertions.
High-volume verification work. Does not synthesize — reports pass/fail per claim.
**Axis → first pick:** bulk / low-stakes / mechanical (must act: searches to verify) → **Haiku 4.5** (the minimum); step up to Sonnet only if verification quality fails.
**Harness:** `general-purpose`
**Wave:** After content is generated, before Quality Critic

### Literature Reviewer
**Responsibility:** Surveys existing work in a field. Maps what is known, what is contested,
what is unknown. Produces a structured literature landscape.
**Axis → first pick:** reasoning/knowledge synthesis → DeepSeek V4 Pro (generation-only) or **Sonnet 4.6** if it must search/read with tools; step up to Opus for contested/high-stakes fields.
**Harness:** `general-purpose` (or gateway curl if generation-only)
**Wave:** Early, parallel with Research Analyst

### Data Analyst
**Responsibility:** Quantitative and statistical analysis. Works with numbers, trends,
correlations. Produces charts, tables, and numeric insights.
**Axis → first pick:** quantitative reasoning + must act (runs analysis scripts, writes output) → tool-loop agent, **Sonnet 4.6**; step up to **Opus 4.8** for high-stakes statistical conclusions. If the math/stats is generation-only (derive results, no scripts), **DeepSeek V4 Pro** (HMMT 95.2, $0.87) is the cheap first pick and the orchestrator writes the files.
**Harness:** `general-purpose` (runs analysis scripts, writes output)
**Wave:** Parallel with Research Analyst if data is pre-available

---

## Construct Roles

### Lead Writer
**Responsibility:** Primary prose creation. Owns the main narrative. Writes from the
structural outline created by Lead Architect.
**Axis → first pick:** prose generation (construct) → if writing to files via tools, **Sonnet 4.6**; for high-stakes/publication-grade prose step up to **Opus 4.8** (GDPval-AA 1890). Generation-only drafting can use a cheap worker, but Claude's writing quality usually justifies Sonnet here. **Opinion/op-ed/unfiltered voice → Grok 4.20.**
**Harness:** `general-purpose`
**Wave:** After structure is set

### Section Writer (instantiate per section)
**Responsibility:** Owns one specific section of a longer document. Named per section:
"Executive Summary Writer", "Methods Section Writer", "Case Studies Writer".
**Axis → first pick:** prose generation (construct) → **Sonnet 4.6** when writing to files via tools; generation-only sections can use a cheap worker (DeepSeek V4 Pro / MiniMax M3) with the orchestrator writing. Step up to **Opus 4.8** for high-stakes sections. **Opinion/op-ed voice → Grok 4.20.**
**Harness:** `general-purpose`
**Wave:** Parallel with other section writers (they're independent if contract is clear)

### Copy Editor
**Responsibility:** Grammar, style consistency, voice alignment, clarity improvements.
Does not change substance — only expression. Pattern-application work.
**Axis → first pick:** bulk / mechanical (apply style template to prose) → **Haiku 4.5** (the minimum); step up to Sonnet only if voice/clarity edits fail.
**Harness:** `general-purpose`
**Wave:** After all writing is complete

### Structural Designer
**Responsibility:** Designs the document architecture: sections, flow, hierarchy, narrative arc.
Produces an outline that all writers follow.
**Axis → first pick:** high-stakes design / single-point-of-failure → **Opus 4.8** (structure cascades to every downstream writer; an error here is costly to reverse).
**Harness:** `general-purpose`
**Wave:** Wave 2 (after Lead Architect, before writers)

### Creative Director
**Responsibility:** Defines the creative vision, tone, and thematic approach. Sets the
"feel" that all other Construct agents must align with.
**Axis → first pick:** creative direction (construct) → **Sonnet 4.6** (balanced taste + tool access). **Opinion/edgy/unfiltered brand voice → Grok 4.20.** Step up to Opus for high-stakes brand work.
**Harness:** `general-purpose`
**Wave:** Wave 2 (parallel with Structural Designer)

### Visual Narrator
**Responsibility:** Describes charts, diagrams, infographics, and visual elements.
Writes alt-text, captions, and data visualization specifications.
**Axis → first pick:** bulk / structured description → **Haiku 4.5** (the minimum). For charts that require *reading* an image, use a multimodal model (MiniMax M3 generation-only, or Sonnet 4.6 if it must act).
**Harness:** `general-purpose`
**Wave:** After content agents, parallel with Copy Editor

---

## Transform Roles

### Data Processor
**Responsibility:** Cleans, normalizes, and transforms raw data. Handles missing values,
type conversions, schema alignment. High-volume mechanical work.
**Axis → first pick:** bulk / mechanical + must act (runs processing scripts) → **Haiku 4.5** (the minimum); step up to Sonnet only on failure.
**Harness:** `general-purpose` (runs processing scripts)
**Wave:** Early

### Format Converter
**Responsibility:** Changes the representation format of content. Markdown → DOCX, CSV → JSON,
transcript → structured notes. Mechanical transformation.
**Axis → first pick:** bulk / mechanical + must act (runs conversion scripts) → **Haiku 4.5** (the minimum); step up to Sonnet only on failure.
**Harness:** `general-purpose`
**Wave:** Early or parallel

### Summarizer (instantiate per corpus)
**Responsibility:** Condenses a specific body of material. Named per corpus:
"Market Reports Summarizer", "Interview Transcripts Summarizer".
**Axis → first pick:** condensation (transform) → **Haiku 4.5** for short-to-medium content (the minimum); step up to **Sonnet 4.6** for complex/nuanced corpora. For very large generation-only corpora, a cheap long-context worker (DeepSeek V4 Pro, 1M context / 384K max output, $0.87) wins on cost — the orchestrator writes the file.
**Harness:** `general-purpose`
**Wave:** Parallel when inputs are independent

### Classifier / Tagger
**Responsibility:** Applies taxonomies, labels, or categories to items.
Bulk categorical work.
**Axis → first pick:** bulk / categorical → **Haiku 4.5** (the minimum); step up to Sonnet only if labeling accuracy fails. Generation-only labeling of a large set can use a cheap worker (DeepSeek V4 Pro / MiniMax M3).
**Harness:** `general-purpose`
**Wave:** Early; others may depend on classification output

### Extractor
**Responsibility:** Pulls specific entities, facts, or structured data from unstructured
sources. Named per target: "Entity Extractor", "Key Quote Extractor".
**Axis → first pick:** bulk / mechanical extraction → **Haiku 4.5** (the minimum). If strict structured JSON output is required and must be acted on, GPT-5.5 via Codex is an option; for generation-only structured extraction a cheap worker (DeepSeek V4 Pro) suffices.
**Harness:** `general-purpose`
**Wave:** Early

---

## Decide Roles

### Strategist
**Responsibility:** Synthesizes analysis into strategic recommendations. Applies frameworks
(SWOT, Porter's Five Forces, Jobs-to-be-Done, etc.). Highest-stakes Decide role.
**Axis → first pick:** high-stakes strategic reasoning → **Opus 4.8** (GDPval-AA 1890, GPQA 93.6; a wrong recommendation is costly and hard to reverse). **Economics/politics/opinion-laden strategy → Grok 4.20.**
**Harness:** `general-purpose`
**Wave:** After research/analysis agents

### Risk Analyst
**Responsibility:** Identifies risks, failure modes, and mitigation paths. Adversarial
framing — looks for what could go wrong.
**Axis → first pick:** high-stakes adversarial reasoning → **Opus 4.8** (under-identified risks are costly; deepest reasoning needed to surface non-obvious failure modes).
**Harness:** `general-purpose`
**Wave:** Parallel with Strategist

### Options Generator
**Responsibility:** Produces a structured set of alternatives or options for evaluation.
Divergent thinking. Breadth over depth.
**Axis → first pick:** divergent reasoning (generation-only) → **DeepSeek V4 Pro** (GPQA 90.1, $0.87) generates the option set and the orchestrator writes; use **Sonnet 4.6** if it must read repo/tool context. Step up to Opus for high-stakes option framing. **Opinion/politics/economics options → Grok 4.20.**
**Harness:** `general-purpose` (or gateway curl if generation-only)
**Wave:** Early in Decide projects (others evaluate these options)

### Evaluator / Scorer
**Responsibility:** Applies criteria to options and scores them. Comparative analysis.
Convergent thinking.
**Axis → first pick:** convergent reasoning (generation-only) → **DeepSeek V4 Pro** (GPQA 90.1, $0.87) when scoring against a fixed rubric and the orchestrator writes; **Sonnet 4.6** if it must read other agents' files via tools. Step up to **Opus 4.8** when the scoring drives a high-stakes decision.
**Harness:** `general-purpose` (or gateway curl if generation-only)
**Wave:** After Options Generator

### Devil's Advocate
**Responsibility:** Challenges the leading recommendations. Argues the opposing case.
Stress-tests assumptions.
**Axis → first pick:** high-stakes adversarial reasoning → **Opus 4.8** (needs depth to find non-obvious counterarguments; a weak rebuttal lets a flawed recommendation through). **Opinion/politics/economics counter-case → Grok 4.20.**
**Harness:** `general-purpose`
**Wave:** After Strategist and Evaluator

### Planner / Roadmap Builder
**Responsibility:** Translates decisions into sequenced action plans. Timelines, milestones,
dependencies.
**Axis → first pick:** sequencing/planning reasoning (generation-only) → **DeepSeek V4 Pro** (GPQA 90.1, $0.87) drafts the roadmap and the orchestrator writes; **Sonnet 4.6** if it must read repo/artifacts via tools. Step up to Opus for high-stakes program plans.
**Harness:** `general-purpose` (or gateway curl if generation-only)
**Wave:** After decisions are made

---

## Technical Roles

Include these when the project has code, data engineering, or systems design components.
For full technical projects, see the Forge skill which has a more complete technical roster.

### Engineer / Developer
**Responsibility:** Implements code, scripts, pipelines, or technical components.
Domain specialized: "Python Data Engineer", "React Frontend Developer", etc.
**Axis → first pick:** real-world coding + must act (edits/runs repo) → **Sonnet 4.6** (tool agent, SWE-bench 79.6); step up to **Opus 4.8** (SWE-bench 88.6, SWE-Pro 69.2) for hard multi-file work. If the work is *generation-only* (write code to stdout, orchestrator commits), **DeepSeek V4 Pro** (SWE-bench 80.6, LiveCodeBench 93.5, Codeforces 3206, $0.87) is the cheap first pick.
**Harness:** `general-purpose` (or gateway curl if generation-only)

### Systems Architect
**Responsibility:** Designs technical systems, APIs, data models. High-stakes design work.
**Axis → first pick:** high-stakes design / single-point-of-failure → **Opus 4.8** (architecture cascades to every downstream engineer; an error here is costly to reverse).
**Harness:** `general-purpose`

### QA / Tester
**Responsibility:** Tests, verifies, validates technical outputs.
**Axis → first pick:** bulk / pattern-based test generation + must act (runs the suite) → **Haiku 4.5** (the minimum); step up to **Sonnet 4.6** only if coverage/quality fails.
**Harness:** `general-purpose`

---

## Role Selection Algorithm

```
START
  |
  v
NOTE: Contract generation (Lead Architect role) is performed by the orchestrator in
Phase 4 BEFORE this algorithm runs. Do not spawn a Lead Architect agent.
  |
  v
EVALUATE TASK FAMILY:
  |
  ├─> Knowledge? → ADD: Research Analyst + [Domain Expert if specialized field]
  |
  ├─> Construct? → ADD: Structural Designer + Lead Writer (or Section Writers if long-form)
  |
  ├─> Transform? → ADD: appropriate Transform roles based on input/output types
  |
  ├─> Decide? → ADD: Options Generator + Evaluator + Strategist + Risk Analyst
  |
  └─> Hybrid? → ADD roles from each applicable family
  |
  v
EVALUATE QUALITY NEEDS:
  |
  ├─> High stakes / publication-grade? → ADD: Quality Critic + Fact Checker
  |
  ├─> Long-form (>5000 words)? → ADD: Copy Editor
  |
  └─> Decision output? → ADD: Devil's Advocate
  |
  v
IF agent_count >= 3:
  ADD: Integration Lead (final wave)
  |
  v
DONE: Return agent list with role + dominant axis (resolve model via model-decision.md) + harness for each
```

---

## Wave Assignment Patterns

**Important:** These patterns are illustrative scaffolds, not prescriptive counts. Actual
agent count is always derived from telos-matching in Phase 3. Use these as starting shapes —
add or remove roles based on what the task actually requires, not to match a target number.
Wave 1 in all patterns below is the FIRST EXECUTION WAVE (after Phase 4 contract generation).

### Pattern 1: Pure Knowledge (illustrative — typical range 3–5 execution agents)
```
Wave 1: Lead Architect
Wave 2: Research Analyst || Domain Expert || Literature Reviewer (parallel)
Wave 3: Data Analyst (if quantitative) || Fact Checker
Wave 4: Quality Critic
Wave 5: Integration Lead
```

### Pattern 2: Construct Document (illustrative — typical range 4–7 execution agents)
```
Wave 1: Lead Architect
Wave 2: Structural Designer || Research Analyst (parallel)
Wave 3: Section Writers (all parallel)
Wave 4: Copy Editor || Fact Checker || Visual Narrator (parallel)
Wave 5: Quality Critic
Wave 6: Integration Lead
```

### Pattern 3: Decision / Strategy (illustrative — typical range 5–8 execution agents)
```
Wave 1: Lead Architect
Wave 2: Research Analyst || Domain Expert (parallel)
Wave 3: Options Generator
Wave 4: Evaluator || Risk Analyst (parallel)
Wave 5: Strategist
Wave 6: Devil's Advocate || Planner (parallel)
Wave 7: Quality Critic
Wave 8: Integration Lead
```

### Pattern 4: Hybrid Complex (illustrative — typical range 8–14 execution agents)
```
Wave 1: Lead Architect
Wave 2: Research + Domain experts (parallel)
Wave 3: Analysis agents (parallel)
Wave 4: Content creation agents (parallel)
Wave 5: Strategy/Decision agents (parallel)
Wave 6: Quality + Editing agents (parallel)
Wave 7: Integration Lead
```

### Pattern 5: Transform Pipeline (illustrative — typical range 2–5 execution agents)
```
Wave 1: Lead Architect
Wave 2: Processor/Extractor/Classifier (parallel when inputs are independent)
Wave 3: Format Converter || Summarizer
Wave 4: Integration Lead (if 3+ agents)
```
