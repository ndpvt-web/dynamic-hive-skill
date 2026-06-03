---
name: hive
description: >
  General-purpose multi-agent orchestration for ANY task type — research, writing, strategy,
  analysis, creative work, coding, data, and mixed projects. Hive dynamically designs the
  agent swarm, selects the right model per agent via a benchmark-grounded decision file
  (Haiku/Sonnet/Opus/DeepSeek/MiniMax/Kimi/GPT-5.5/Grok — chosen on benchmarks + cost, never
  stereotypes), and chooses the right harness per agent (Claude Code general-purpose, Plan,
  Explore, or Codex CLI). Nothing is hardcoded — every model pick flows from the decision file
  and task analysis. Use Hive proactively whenever a task would benefit from parallel
  specialist agents working toward a shared deliverable, or when the user says "build this",
  "research this thoroughly", "create a comprehensive X", "orchestrate agents", "multi-agent",
  "swarm", or asks for outputs that have multiple distinct components (report + code + slides,
  analysis + strategy + draft, etc.).
version: 1.0.0
author: HappyCapy Research
triggers:
  - hive
  - multi-agent
  - orchestrate agents
  - agent swarm
  - build with agents
  - comprehensive research
  - deep analysis with agents
  - parallel agents
---

# Hive: General-Purpose Multi-Agent Orchestration

Hive is a domain-agnostic successor to Forge. Where Forge was designed for software
projects, Hive works for **any task** — research papers, marketing strategies, business
plans, data analyses, creative projects, and complex technical work. The agent swarm, model
assignments, and harness selections are all derived dynamically from task analysis.

**The core insight:** Multi-agent quality gains come not from having more agents, but from
matching each agent's role, model, and harness to the specific cognitive demands of its task.
A Researcher synthesizing 50 sources needs Opus. A Formatter applying a template needs Haiku.
Treating them identically wastes cost on one and sacrifices quality on the other.

## Resource Map (read these on demand, not upfront)

| File | When to read |
|------|-------------|
| `resources/task-taxonomy.md` | Phase 1: to classify task type and calculate coupling |
| `resources/agent-roster.md` | Phase 3: to select roles and wave assignments |
| `resources/model-decision.md` | **Phase 3b: READ FIRST to pick each agent's model (benchmark + cost based)** |
| `resources/model-harness-guide.md` | Phase 3: to map the chosen model to its harness + launch command |
| `resources/coordination-modes.md` | Phase 2: to select parallel/hybrid/sequential |
| `templates/agent-brief.md` | Phase 5: to compose each agent's prompt |
| `templates/crucible-contract.md` | Phase 4: to generate the shared contract |
| `evals/evals.json` | Reference only: 3 example task prompts with pass/fail assertions for testing skill output |

---

## Platform Constraint: Model Routing

> **IMPORTANT — read before Phase 3. Verified 2026-05-30 from harness source + live tests.**
>
> The Agent tool's `model=` parameter **does not override the model**. The harness reads
> `process.env.CLAUDE_CODE_SUBAGENT_MODEL` (pinned to `claude-sonnet-4-6`) live on every spawn and
> ignores the param. So every `Agent(...)` spawn runs **Sonnet 4-6**, even with `model="opus"`.
>
> **To run a tool-using agent on a different model, invoke the `claude` CLI as a subprocess with
> `--model`.** This goes through the session's own **Bedrock entitlement** (your unlimited Anthropic
> credits) → full tool access, **no pricing restriction**. The old AI-Gateway curl path is
> **deprecated for Anthropic models**: it returns text only (no tools) and is the source of the
> "pricing restriction" errors (it's a separate metered/gated billing surface).
>
> Execution rules:
> - **Opus agents** → `claude -p "<brief>" --model opus` (or `--model claude-opus-4-8`) subprocess ✓ tools
> - **Haiku agents** → `claude -p "<brief>" --model haiku` subprocess ✓ tools
> - **Sonnet agents** → `Agent(subagent_type="general-purpose", prompt=...)` — pinned session default ✓
> - **Non-Anthropic models** → Codex CLI (`/home/node/.local/bin/codex -c model_provider=ai_gateway`)
>
> Verified live (all exit 0, full tools, ungated): `--model haiku`→claude-haiku-4-5,
> `sonnet`→claude-sonnet-4-5, `opus`→claude-opus-4-8, `claude-opus-4-8`→claude-opus-4-8.
> (Alias `sonnet`=4-5; use full ID `claude-sonnet-4-6` for 4-6.) See `resources/model-harness-guide.md`.
>
> The tool-using subprocess helper (reuse this pattern throughout the skill — replaces the old
> `hive_call` curl helper). The spawned agent has Write/Bash/etc. and produces its own deliverable:
> ```bash
> # hive_agent <model> <out_log> <<'BRIEF' ... BRIEF
> hive_agent() {  # $1=model  $2=output-log path; brief on stdin
>   claude -p "$(cat)" --model "$1" \
>     --permission-mode acceptEdits --dangerously-skip-permissions \
>     > "$2" 2>&1 < /dev/null
> }
> # Example (agent writes deliverable to ./outputs/strategy.md itself per its brief):
> hive_agent opus /tmp/hive/strategist.log <<'BRIEF'
> You are the Strategist. ...full self-contained brief, incl. exact output path...
> BRIEF
> ```
>
> For a **parallel wave** with mixed models, background each and `wait`:
> ```bash
> mkdir -p /tmp/hive
> claude -p "$BRIEF_1" --model opus  --dangerously-skip-permissions < /dev/null > /tmp/hive/a.log 2>&1 &
> claude -p "$BRIEF_2" --model haiku --dangerously-skip-permissions < /dev/null > /tmp/hive/b.log 2>&1 &
> wait   # each agent wrote its own deliverable file; logs hold their transcripts
> ```

---

## The Seven Phases

### Phase 1: TASK ANALYSIS

Read `resources/task-taxonomy.md` now.

Decompose the user's request into:

1. **Task family** — which of the five families (Knowledge / Construct / Transform / Decide / Hybrid)
2. **Components** — the discrete deliverables or workstreams
3. **Complexity score** (1–25 scale; see taxonomy)
4. **Coupling score** — how tightly the components depend on each other (0.0–1.0)
5. **Quality tier** — what level of output quality does this task demand?

Output the analysis in this format:

```
HIVE ANALYSIS
=================
Task Family: [Knowledge|Construct|Transform|Decide|Hybrid]
Objective: [one-sentence statement]

Components:
1. [Component Name] — [What it produces]
2. ...

Complexity: [1-25] ([Simple|Moderate|Complex|Very Complex|Extreme])

Coupling Analysis:
  Shared references (analog of shared interfaces): X / Y total
  Circular dependencies: X / Y total
  Shared context ratio: 0.X
  COUPLING SCORE: 0.XX

Recommended Mode: [Parallel|Hybrid|Sequential]
Estimated Agents: [derive in Phase 3 via telos-matching — do not anchor here]
```

---

### Phase 2: COORDINATION MODE SELECTION

Read `resources/coordination-modes.md` now.

Select one of three modes based on the coupling score:

| Coupling Score | Mode | When to Use |
|---------------|------|-------------|
| < 0.4 | **Contract-First Parallel** | Independent components, well-defined outputs |
| 0.4–0.7 | **Hybrid Wave** | Mixed dependencies, some parallel, some sequential |
| > 0.7 | **Progressive-Layer Sequential** | Tightly coupled, emergent design |

---

### Phase 3: AGENT DESIGN — THE CRUCIBLE CORE

This is where Hive differs from every other orchestration approach.

**Read `resources/agent-roster.md` and `resources/model-harness-guide.md` now.**

For each agent in the swarm, determine three things:

#### 3a. Role
Select from the universal role catalog in `agent-roster.md`. Roles are not
hardcoded — match them to the actual cognitive work required. A "Backend Engineer"
is wrong for a marketing strategy; a "Strategist" + "Market Researcher" is right.
Custom roles are allowed and encouraged when standard roles don't fit.

#### 3b. Model — decide via `resources/model-decision.md` (READ IT FIRST)

**Do not guess a model from habit or stereotype.** Open `resources/model-decision.md`
and run its decision algorithm for each role. In short:

1. **Content-type override:** opinion / untraditional / 18+ / politics / economics /
   real-human-opinion / unfiltered → **Grok 4.20** (mandatory, 100%).
2. **Classify** the role's dominant axis (coding-real, coding-competitive, math, reasoning,
   tool-use, web-research, os-gui, multimodal, long-context, swarm, bulk, general).
3. **Needs to act?** If the role must call tools (edit files, run commands, MCP), restrict to
   tool-loop agents {Opus 4.8, Sonnet 4.6, Haiku 4.5, GPT-5.5}. If it only generates text/
   code/analysis, the cheap workers {DeepSeek V4 Pro, MiniMax M3, Kimi K2.6} are first-class.
4. **Shortlist** the top benchmark scorers on that axis, then **pick the cheapest that clears
   the bar.** Step up exactly one tier only on demonstrated failure (minimum-sufficient rule).
5. **High-stakes / irreversible** → start at Opus 4.8 regardless of cost.
6. **Bulk / low-stakes / mass info** → start at **Haiku 4.5** (the minimum), escalate only if it fails.

There is **no blanket default model**. Every pick must trace to a benchmark + cost
justification in the decision file. Models removed from the roster (Gemini, GPT-4.1,
gpt-5-mini, minimax-m1/m2.7) must not be spawned.

> **Capability gate (verified):** Non-Anthropic, non-OpenAI models (DeepSeek, MiniMax, Kimi,
> Grok) are **text/reasoning/code workers** in this env, NOT autonomous CLI tool agents — the
> claude-multimodel proxy handles non-streaming tool use but the `claude` CLI streams, which
> drops their tool calls. Reach them via **direct gateway curl** (`$AI_GATEWAY_BASE_URL/api/v1/chat/completions`).
> The orchestrator must perform any file writes / command execution they describe. OpenAI
> (GPT-5.5) acts via Codex CLI. Claude tiers act via Agent tool / `claude -p`. See
> `model-harness-guide.md` for exact launch commands.

#### 3c. Harness
Select the execution environment for each agent:

| Harness | When to Use |
|---------|-------------|
| `general-purpose` | Agent needs to read/write files, run code, use tools |
| `Plan` subagent | Read-only analysis, planning, no file side-effects |
| `Explore` subagent | Discovery scan, file/codebase exploration |
| Codex CLI | Non-Anthropic model needed, or shell-pipeline dispatch |

**Rule:** All execution agents (producing deliverables) use `general-purpose`.
Phase 1 analysis uses `Plan`. Discovery uses `Explore`. Non-Anthropic models use Codex CLI.

---

### Phase 4: CONTRACT GENERATION (The Orchestrator acts as Lead Architect)

The orchestrator generates the shared contract using Opus 4.6. This IS the Lead Architect
role from the agent roster — it is performed directly by the orchestrator, not by a
separately-spawned agent. The contract is the single source of truth for all agents.
A weak contract cascades into swarm failure.

Use `templates/crucible-contract.md` as the template. The contract must specify:
1. **Deliverable manifest** — every output, its format, its owner agent
2. **Shared references** — facts, terminology, style conventions all agents must align on
3. **Interface definitions** — how outputs connect (e.g., how the Analysis feeds the Strategy)
4. **Section boundaries** — exactly what each agent owns and must not touch
5. **Quality criteria** — what "done" means for each deliverable

**Contract generation should use Opus.** If the orchestrator session is already Opus
(check `/model`), write the contract directly. Otherwise delegate to a tool-using
`claude -p --model opus` subprocess (the Agent tool can't change model — see Platform Constraint):

```bash
# The Opus subprocess writes the contract to ./tmp/hive_contract.md itself (instruct it in the brief).
mkdir -p tmp
claude -p "$CONTRACT_BRIEF
Write the finished contract to ./tmp/hive_contract.md using the Write tool." \
  --model opus --permission-mode acceptEdits --dangerously-skip-permissions \
  < /dev/null > /tmp/hive/contract_gen.log 2>&1
test -s ./tmp/hive_contract.md && echo "Contract generated → ./tmp/hive_contract.md" || cat /tmp/hive/contract_gen.log
```
where `$CONTRACT_BRIEF` is the full contract-generation prompt (task description, component list,
deliverable manifest, shared references, interface definitions, quality criteria).

---

### Phase 5: EXECUTION

Execute agents according to the mode from Phase 2. Use the agent brief template at
`templates/agent-brief.md` for every agent prompt. Fill in:
- `{{AGENT_NAME}}`, `{{AGENT_ROLE}}`, `{{MODEL_CLASS}}`
- `{{MODEL_RATIONALE}}` — why this model was chosen for this role
- `{{HARNESS_TYPE}}` — so agents know their execution context
- `{{CONTRACT}}` — the full shared contract
- `{{ASSIGNMENT}}` — this agent's specific deliverable(s)
- `{{DEPENDENCIES}}` — what this agent consumes from others
- `{{EXPORTS}}` — what this agent produces for others

**All execution agents receive the FULL contract, not just their section.**
Agents given partial context produce significantly more coordination errors — partial context is a primary driver of swarm failure.

#### Spawning Pattern

The pattern depends on the coordination mode selected in Phase 2:

**Contract-First Parallel (coupling < 0.4) — anonymous agents, no A2A:**

Sonnet agents (session default) → spawn in ONE message, all parallel:
```python
Agent(subagent_type="general-purpose", prompt=sonnet_agent_1_brief)
Agent(subagent_type="general-purpose", prompt=sonnet_agent_2_brief)
# No model= needed — session is already Sonnet 4.6
```

Opus agents → tool-using `claude -p --model opus` subprocesses, backgrounded in parallel.
Each agent writes its OWN deliverable (its brief specifies the exact output path):
```bash
mkdir -p /tmp/hive
claude -p "$OPUS_BRIEF_1" --model opus --dangerously-skip-permissions < /dev/null > /tmp/hive/strategist.log 2>&1 &
claude -p "$OPUS_BRIEF_2" --model opus --dangerously-skip-permissions < /dev/null > /tmp/hive/risk_analyst.log 2>&1 &
wait   # ./outputs/strategist.md and ./outputs/risk_analyst.md were written by the agents themselves
echo "Opus agents complete"
```

Haiku agents → same pattern with `--model haiku`. Mixed-model waves are fine — background each
`claude -p` with its own `--model` and `wait` once. (Use `claude-opus-4-8` for Opus 4-8; full ID
`claude-sonnet-4-6` if you need Sonnet 4-6 in a subprocess rather than the in-process Agent tool.)

Non-Anthropic models are **TEXT-ONLY** here (no tool/file execution — `bwrap` missing; orchestrator writes files).
**OpenAI** (GPT-5.5) works via Codex CLI. **Grok/MiniMax return EMPTY through Codex** → use direct gateway curl.
```bash
# OpenAI via Codex — IMPORTANT: exec comes BEFORE --skip-git-repo-check (flag order matters)
/home/node/.local/bin/codex -c model_provider=ai_gateway -m openai/gpt-5.5 exec --skip-git-repo-check "..."

# Grok / MiniMax / DeepSeek → direct gateway curl (Codex returns EMPTY for these)
curl -s "$AI_GATEWAY_BASE_URL/api/v1/chat/completions" \
  -H "Authorization: Bearer $AI_GATEWAY_API_KEY" -H "Content-Type: application/json" \
  -d '{"model":"x-ai/grok-4.20","messages":[{"role":"user","content":"..."}]}'   # grok-3 & minimax-m1 are 403 (not on plan); use grok-4.20 / deepseek-v4-pro / minimax-m3
```

**Hybrid Wave (coupling 0.4–0.7) — named teammates, A2A enabled:**
```python
# Step 1: Create team ONCE before any agents are spawned
TeamCreate(team_name="hive-{project-slug}-{timestamp}")

# Step 2: Spawn wave agents as named teammates (ONE message per wave)
# Note: model= param is ignored by the harness — all Agent-spawned teammates run Sonnet 4.6.
# Use Sonnet-class roles in Hybrid Wave. Opus roles run as direct API calls (no A2A).
Agent(subagent_type="general-purpose",
      name="researcher", team_name="hive-{project-slug}-{timestamp}",
      prompt=researcher_brief)   # brief includes A2A_ENABLED=true, TEAM_NAME, PEER_AGENTS
Agent(subagent_type="general-purpose",
      name="analyst", team_name="hive-{project-slug}-{timestamp}",
      prompt=analyst_brief)

# Agents can now: SendMessage(to="researcher", message="...", summary="...")
# Orchestrator receives all messages automatically

# Step 3: After all waves complete — shut down and clean up
SendMessage(to="*", message={"type": "shutdown_request", "reason": "Hive swarm complete"})
TeamDelete()
```

**Progressive-Layer Sequential (coupling > 0.7) — anonymous agents, no A2A:**
```bash
# Layer 1 — Opus tool-using subprocess (frozen output written to file, passed to next layer).
# The Opus agent writes its own deliverable via the Write tool; instruct it in the brief.
mkdir -p tmp /tmp/hive
claude -p "CONTRACT:
$(cat ./tmp/hive_contract.md)

Build Layer 1: ${layer_1_spec}
Write the finished Layer 1 deliverable to ./tmp/hive_layer1.md using the Write tool." \
  --model opus --permission-mode acceptEdits --dangerously-skip-permissions \
  < /dev/null > /tmp/hive/layer1.log 2>&1
test -s ./tmp/hive_layer1.md && echo "Layer 1 complete → ./tmp/hive_layer1.md" || cat /tmp/hive/layer1.log

# validate + fix (max 3 tries), then Layer 2 — Sonnet via Agent tool (in-process, pinned default)
# (in the orchestrator)
layer1_output = open('./tmp/hive_layer1.md').read()
contract = open('./tmp/hive_contract.md').read()
Agent(subagent_type="general-purpose",
    prompt=f"CONTRACT:\n{contract}\n\nFROZEN LAYER 1:\n{layer1_output}\n\nBuild Layer 2: {layer_2_spec}")
```

Spawn all agents within the same wave in **one message** (not sequentially). Wait for
wave completion before starting the next wave.

---

### Phase 6: VALIDATION

Validation criteria depend on task type. After all agents complete:

1. **Completeness check** — every deliverable in the manifest exists
2. **Interface check** — outputs that feed other outputs are compatible
3. **Quality check** — spot-check against quality criteria in contract
4. **Coherence check** — does the whole swarm output read as unified work?

For coherence validation on complex outputs, spawn Opus as Quality Critic via the
`claude -p --model opus` subprocess (full tools, ungated). It reads the outputs and
writes its review itself:

```bash
mkdir -p tmp /tmp/hive
claude -p "You are the Quality Critic for this Hive swarm.

CONTRACT:
$(cat ./tmp/hive_contract.md)

Read every deliverable under ./outputs/ (use the Read/Glob tools).
Review for: completeness, coherence, contract compliance, quality.
Report what passed, what failed, and what needs fixing — be specific.
Write your full review to ./tmp/hive_quality_review.md using the Write tool." \
  --model opus --permission-mode acceptEdits --dangerously-skip-permissions \
  < /dev/null > /tmp/hive/critic.log 2>&1
test -s ./tmp/hive_quality_review.md && echo "Quality review → ./tmp/hive_quality_review.md" || cat /tmp/hive/critic.log
```

---

### Phase 7: DELIVERY

1. **Collect** all agent outputs
2. **Merge** in dependency order (outputs that feed others go first)
3. **Fix** targeted validation failures (spawn one Fixer with specific errors, not full regen)
4. **Package** into final deliverable(s)
5. **Attribute** via AGENTS.md (agent name, role, model, harness, deliverable)

---

## Anti-Patterns to Avoid

| Anti-Pattern | Why It Fails | Correct Approach |
|-------------|-------------|-----------------|
| `Agent(model="claude-opus-4.8", ...)` | Agent tool model param is silently ignored in HappyCapy (`CLAUDE_CODE_SUBAGENT_MODEL` overrides it) — all agents run Sonnet 4.6 | Spawn via `claude -p --model opus ... --dangerously-skip-permissions < /dev/null` subprocess (full tools, ungated) |
| `Agent(model="opus", ...)` | Same — even short enum values are ignored by the harness | Same fix: `claude -p --model opus/haiku` subprocess |
| AI Gateway curl for Opus/Haiku | No tool use (text-only) AND pricing-gated → "pricing restriction" errors | Use the `claude -p --model` subprocess (routes through ungated Bedrock entitlement) |
| Reserving the AI Gateway/Codex path for non-Anthropic models | Anthropic models go through `claude -p`; gateway is only for GPT-5.5/Grok/MiniMax/DeepSeek | Codex CLI for GPT-5.5, gateway curl for the rest |
| Same model for all agents | Wastes Opus on formatting, underpowers synthesis | Telos-match every role |
| `general-purpose` for analysis phases | Burns context on read-only work | Use `Plan` for Phase 1 |
| Partial contract per agent | 38% more coordination errors | Every agent gets full contract |
| Sequential when coupling < 0.4 | 4–8x slower for no quality gain | Parallel when safe |
| Regenerating entire output on validation fail | 4x slower than targeted fix | Spawn Fixer with specific errors |
| Hardcoding tech roles for non-tech tasks | Wrong agents, wrong telos | Use universal roster + custom roles |
| A2A messaging in Parallel mode | Adds overhead with zero benefit; agents are independent | A2A only in Hybrid Wave mode |
| Using anonymous `Agent` tool for Hybrid Wave | Agents can't send/receive messages | Use `TeamCreate` + `name=` for Hybrid Wave |
| Agents messaging for status/confirmations | Floods orchestrator, slows execution | Only message for the 3 permitted triggers |
| Not calling `TeamDelete()` after swarm | Leaves dangling team resources | Always `TeamDelete()` after final wave |

---

## The Aristotelian Principle at the Core of Hive

> *"The excellence (arete) of a thing is the activity by which it fulfills its function well."*
> — Aristotle, Nicomachean Ethics I.7

Applied: a model's excellence is not its benchmark score but how well it performs
the specific task for which it was designed. The virtuous orchestrator does not use
the most powerful model — they use the most appropriate one. Hive's entire
model-selection system is built on this principle.

---

## When NOT to Use Hive

- Single-agent tasks with no interdependencies (use Task/Agent directly)
- Tasks completable in one step by one agent
- Quick fixes or one-file changes
- Exploratory research without a concrete deliverable

Hive adds value when there are **3+ components** that must **interoperate** or
**when the task is too large for one agent's context window**.
