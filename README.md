# Hive — General-Purpose Multi-Agent Orchestration

Hive dynamically designs and executes a swarm of specialist AI agents for **any task type** — research, writing, strategy, analysis, data, creative work, or hybrid projects. It selects the right model and execution harness per agent based on benchmarks and cost, never habit or stereotype.

---

## Core Principle

> Assign the **minimum-sufficient model** that can fulfill a role's cognitive demands. Justify every pick from benchmarks and cost. Step up exactly one tier on demonstrated failure.

Multi-agent quality gains come from matching each agent's **role, model, and harness** to the specific demands of its task — not from using more agents or more powerful models uniformly.

---

## When to Use Hive

**Use it when:**
- The task has 3+ distinct components that must interoperate
- The task is too large for one agent's context window
- You need parallel workstreams with a unified final deliverable
- User says: "build this", "research this thoroughly", "create a comprehensive X", "multi-agent", "swarm"

**Do not use it for:**
- Single-agent tasks with no interdependencies
- Tasks completable in one step
- Quick fixes or one-file changes

---

## The Seven Phases

### Phase 1: Task Analysis
Read `resources/task-taxonomy.md`.

Classify the task into one of five families: **Knowledge, Construct, Transform, Decide, or Hybrid**. Calculate complexity (1–25 scale) and a coupling score (0.0–1.0) measuring how tightly components depend on each other.

Output format:
```
HIVE ANALYSIS
=================
Task Family: [Knowledge|Construct|Transform|Decide|Hybrid]
Objective: [one sentence]
Components: [list]
Complexity: [1-25]
Coupling Score: [0.00]
Recommended Mode: [Parallel|Hybrid|Sequential]
```

### Phase 2: Coordination Mode Selection
Read `resources/coordination-modes.md`.

| Coupling Score | Mode | Use When |
|---|---|---|
| < 0.4 | Contract-First Parallel | Independent components — 4–8x speedup |
| 0.4–0.7 | Hybrid Wave | Mixed dependencies — 2–5x speedup |
| > 0.7 | Progressive-Layer Sequential | Everything depends on everything |

### Phase 3: Agent Design
Read `resources/agent-roster.md` and `resources/model-decision.md`.

For each agent determine: **Role** (from universal roster) → **Model** (from decision file algorithm) → **Harness** (execution path).

### Phase 4: Contract Generation
The orchestrator acts as Lead Architect and generates a shared contract using `templates/crucible-contract.md`. All agents receive the **full contract** — partial context causes coordination failures.

### Phase 5: Execution
Spawn agents per the coordination mode using `templates/agent-brief.md` for each prompt.

### Phase 6: Validation
Check completeness, interface compatibility, quality, and coherence against the contract.

### Phase 7: Delivery
Merge outputs in dependency order, fix targeted failures, package the final deliverable.

---

## Model Selection

Read `resources/model-decision.md` before assigning any model. The decision algorithm:

1. **Content-type override:** Opinion / politics / economics / 18+ / unfiltered → **Grok 4.20** (mandatory, no exceptions).
2. **Classify** the role's dominant axis: `coding-real`, `coding-competitive`, `math`, `reasoning`, `tool-use`, `web-research`, `os-gui`, `multimodal`, `long-context`, `swarm`, `bulk`, `general`.
3. **Needs to act?** If the role must call tools (edit files, run commands): restrict to `{Opus 4.8, Sonnet 4.6, Haiku 4.5, GPT-5.5}`. If it only generates text/code/analysis: cheap text workers (DeepSeek V4 Pro, MiniMax M3) are first-class.
4. **Shortlist** top benchmark scorers on that axis, then **pick the cheapest that clears the bar**.
5. **High-stakes / irreversible** roles → start at Opus 4.8 regardless.
6. **Bulk / low-stakes / mechanical** roles → start at Haiku 4.5 (the minimum).

### Active Model Roster

| Model | Tool Agent? | How to Run |
|---|---|---|
| Claude Opus 4.8 | Yes | `claude -p --model claude-opus-4-8 --dangerously-skip-permissions < /dev/null` |
| Claude Sonnet 4.6 | Yes | `Agent(subagent_type="general-purpose", prompt=...)` — session default |
| Claude Haiku 4.5 | Yes | `claude -p --model haiku --dangerously-skip-permissions < /dev/null` |
| GPT-5.5 | Text-only | `codex -c model_provider=ai_gateway -m openai/gpt-5.5 exec --skip-git-repo-check "..."` |
| DeepSeek V4 Pro | Text-only | Direct gateway curl → `deepseek/deepseek-v4-pro` |
| MiniMax M3 | Text-only | Direct gateway curl → `minimax/minimax-m3` |
| Grok 4.20 | Text-only | Direct gateway curl → `x-ai/grok-4.20` |

**Do not use:** Gemini (all), GPT-4.1, gpt-5-mini, minimax-m1, minimax-m2.7 — superseded or low credits.

### Critical Platform Constraint

The `Agent(model="opus")` parameter is **silently ignored**. The harness pins all `Agent(...)` spawns to Sonnet 4.6 regardless. To run Opus or Haiku with full tools:

```bash
# Opus with tools (ungated Bedrock entitlement — no pricing restriction)
claude -p "full agent brief" --model claude-opus-4-8 \
  --permission-mode acceptEdits --dangerously-skip-permissions < /dev/null \
  > /tmp/hive/agent.log 2>&1

# Parallel wave — background each and wait
claude -p "$BRIEF_A" --model opus  --dangerously-skip-permissions < /dev/null > /tmp/hive/a.log 2>&1 &
claude -p "$BRIEF_B" --model haiku --dangerously-skip-permissions < /dev/null > /tmp/hive/b.log 2>&1 &
wait
```

---

## Coordination Modes in Detail

### Mode 1: Contract-First Parallel (coupling < 0.4)
Spawn all agents in **one message** simultaneously. Each works independently from the contract spec. 4–8x faster than sequential.

```python
Agent(subagent_type="general-purpose", prompt=agent_1_brief)
Agent(subagent_type="general-purpose", prompt=agent_2_brief)
Agent(subagent_type="general-purpose", prompt=agent_3_brief)
# All in the same turn
```

### Mode 2: Hybrid Wave (coupling 0.4–0.7)
Group components into waves. Agents within a wave run in parallel; waves execute sequentially. Agents can message each other (A2A) only for: interface deviations, partial output availability, or peer-resolvable blockers.

```python
TeamCreate(team_name="hive-{project}-{timestamp}")

# Wave 1
Agent(subagent_type="general-purpose", name="researcher", team_name="...", prompt=w1_brief)
Agent(subagent_type="general-purpose", name="analyst",    team_name="...", prompt=w1_brief)
# --- wait ---

# Wave 2
Agent(subagent_type="general-purpose", name="strategist", team_name="...", prompt=w2_brief)

# Teardown
SendMessage(to="*", message={"type": "shutdown_request", "reason": "Hive swarm complete"})
TeamDelete()
```

### Mode 3: Progressive-Layer Sequential (coupling > 0.7)
Build layer by layer. Each layer's actual output (not spec) becomes the foundation for the next. Max 3 fix iterations per layer.

```bash
# Layer 1 — Opus subprocess writes its own deliverable
claude -p "$LAYER1_BRIEF" --model opus --dangerously-skip-permissions < /dev/null > /tmp/hive/l1.log 2>&1
# Layer 2 — receives frozen Layer 1 output
Agent(subagent_type="general-purpose", prompt="FROZEN LAYER 1: $(cat ./tmp/layer1.md)\n\nBuild Layer 2: ...")
```

---

## Agent Roles

See `resources/agent-roster.md` for the full catalog. Key roles:

**Universal (every project):**
- **Lead Architect** — generates the contract (Phase 4, performed by orchestrator directly using Opus)
- **Integration Lead** — final wave, merges all outputs into unified deliverable (Opus)
- **Quality Critic** — reviews all outputs against contract criteria (Opus)

**Knowledge:** Research Analyst, Domain Expert, Fact Checker, Data Analyst

**Construct:** Lead Writer, Section Writers, Structural Designer, Copy Editor, Creative Director

**Transform:** Data Processor, Format Converter, Summarizer, Classifier, Extractor

**Decide:** Strategist, Risk Analyst, Options Generator, Evaluator, Devil's Advocate, Planner

**Technical:** Engineer/Developer, Systems Architect, QA Tester

---

## Harness Selection

| Harness | Use When |
|---|---|
| `general-purpose` | Agent produces files, runs code, uses tools — all execution agents |
| `Plan` subagent | Pure analysis/planning, no file writes (Phase 1) |
| `Explore` subagent | Read-only discovery/scanning across many files |
| Codex CLI | OpenAI (GPT-5.5) text generation only |
| `claude -p` subprocess | Opus or Haiku with full tools (primary path for non-Sonnet Claude) |

---

## File Structure

```
hive/
├── SKILL.md                        # Main skill definition and 7-phase walkthrough
├── resources/
│   ├── task-taxonomy.md            # Phase 1: classify task, calculate coupling
│   ├── coordination-modes.md       # Phase 2: parallel / hybrid / sequential
│   ├── agent-roster.md             # Phase 3a: universal role catalog
│   ├── model-decision.md           # Phase 3b: benchmark-grounded model selection
│   └── model-harness-guide.md      # Phase 3c: execution path per model
├── templates/
│   ├── crucible-contract.md        # Phase 4: shared contract template
│   └── agent-brief.md              # Phase 5: per-agent prompt template
└── evals/
    └── evals.json                  # 3 example tasks with pass/fail assertions
```

Read resources **on demand** per phase — do not load all upfront.

---

## Anti-Patterns

| Anti-Pattern | Why It Fails | Fix |
|---|---|---|
| `Agent(model="opus", ...)` | Model param silently ignored — always runs Sonnet 4.6 | Use `claude -p --model opus` subprocess |
| AI Gateway curl for Claude models | Text-only, pricing-gated → "pricing restriction" errors | Use `claude -p --model` (Bedrock entitlement, ungated) |
| Same model for all agents | Wastes Opus on formatting, underpowers synthesis | Telos-match every role via model-decision.md |
| Partial contract per agent | 38% more coordination errors | Every agent gets the full contract |
| `general-purpose` for read-only analysis | Wastes context | Use `Plan` for analysis-only phases |
| Sequential when coupling < 0.4 | 4–8x slower for no quality gain | Use parallel mode |
| Regenerating entire output on failure | 4x slower | Spawn targeted Fixer with specific errors |
| A2A messaging in Parallel mode | Zero benefit, adds overhead | A2A only in Hybrid Wave mode |
| Skipping `TeamDelete()` after swarm | Leaves dangling resources | Always call `TeamDelete()` after final wave |
| Codex CLI for Grok/MiniMax/DeepSeek | Returns empty output | Use direct gateway curl for these models |

---

## Quick Start Example

```python
# Task: Competitive analysis + go-to-market strategy

# Phase 1-2: Analyze (coupling ~0.55 → Hybrid Wave)
# Phase 3: Design agents
# Phase 4: Generate contract (Opus subprocess)

import subprocess
contract_brief = "..."
subprocess.run(["claude", "-p", contract_brief, "--model", "claude-opus-4-8",
                "--dangerously-skip-permissions"], input=b"")

# Phase 5: Execute — Wave 1 (research, parallel)
TeamCreate(team_name="hive-companalysis-001")
Agent(subagent_type="general-purpose", name="researcher", team_name="hive-companalysis-001", prompt=research_brief)
Agent(subagent_type="general-purpose", name="data-analyst", team_name="hive-companalysis-001", prompt=data_brief)

# Wave 2 (strategy, after Wave 1 completes)
Agent(subagent_type="general-purpose", name="strategist", team_name="hive-companalysis-001", prompt=strategy_brief)

# Final wave (integration — Opus subprocess)
# claude -p "$INTEGRATION_BRIEF" --model claude-opus-4-8 --dangerously-skip-permissions < /dev/null

SendMessage(to="*", message={"type": "shutdown_request", "reason": "Hive swarm complete"})
TeamDelete()
```
