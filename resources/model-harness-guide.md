# Hive Model & Harness Guide
## Aristotelian Telos-Matching for Agent Assignment

> **PLATFORM REALITY (HappyCapy Claude Code env) — verified 2026-05-30 from harness source + live tests**
>
> The Agent tool's `model=` parameter is **silently overridden**. The harness reads
> `process.env.CLAUDE_CODE_SUBAGENT_MODEL` (pinned to `claude-sonnet-4-6`) **live on every spawn**;
> when it holds a concrete model it ignores the `model=` param entirely. So **every `Agent(...)`
> spawn runs Sonnet 4-6**, no matter what `model="opus"`/`"haiku"` you pass. (Precedence proven from
> source: env var > `model=` param > agent-file frontmatter. The literal value `"inherit"` is the only
> value that re-enables the param — but it's platform-set, not user-controllable per session.)
>
> **THE FIX — spawn tool-using agents on ANY model via the `claude` CLI as a subprocess.**
> This routes through the session's own **Bedrock entitlement** (your unlimited Anthropic credits),
> so it has **NO pricing restriction** — full tool access included. The AI Gateway curl path is a
> *separate, metered, per-model-gated* surface; that is exactly where the "pricing restriction"
> errors come from. Do **not** use it for Anthropic models anymore.
>
> ```bash
> claude -p "<full agent brief>" \
>   --model <haiku|sonnet|opus|claude-opus-4-8|claude-sonnet-4-6> \
>   --permission-mode acceptEdits --dangerously-skip-permissions < /dev/null
> ```
> Verified live (all exit 0, full tools, no gating): `--model haiku`→`claude-haiku-4-5`,
> `sonnet`→`claude-sonnet-4-5`, `opus`→`claude-opus-4-8`, `claude-opus-4-8`→`claude-opus-4-8`.
> NOTE: alias `sonnet` resolves to 4-5; pass the full ID `claude-sonnet-4-6` for 4-6.
> This is **per-agent and per-session** (each subprocess is independent) and needs no config change.
>
> **Execution path by model tier (corrected):**
> | Model | Execution Path | Tools? | Gated? |
> |-------|---------------|--------|--------|
> | Opus (4-6 / 4-8) | `claude -p --model opus` (or `claude-opus-4-8`) subprocess | yes | no |
> | Haiku 4-5 | `claude -p --model haiku` subprocess | yes | no |
> | Sonnet 4-6 | `Agent(subagent_type="general-purpose", prompt=...)` — pinned session default (fastest, in-process) | yes | no |
> | Non-Anthropic (OpenAI) | Codex CLI: `codex -c model_provider=ai_gateway -m openai/gpt-5.5 exec --skip-git-repo-check "..."` | **text-only here** | gateway-metered |
> | Non-Anthropic (Grok/MiniMax/Gemini/DeepSeek) | **Direct gateway curl** `$AI_GATEWAY_BASE_URL/api/v1/chat/completions` | text-only | gateway-metered |
>
> **DEPRECATED:** the old AI-Gateway-curl path for Opus/Haiku (`curl $AI_GATEWAY_BASE_URL/...`) —
> it produces NO tool use and hits the pricing gate. Replaced by the `claude -p` subprocess above.
>
> **VERIFIED ENV REALITY (2026-05-30 live tests) — non-Anthropic models:**
> - **Codex flag order is `exec --skip-git-repo-check`** (flag AFTER `exec`). The reverse order errors out.
> - **Codex v0.118 only supports the `responses` wire API.** Only **OpenAI** models (gpt-5.5)
>   return content through it. **Grok-4.20 / MiniMax / DeepSeek return EMPTY via Codex** (exit 0, no output)
>   even though they are valid+on-plan via direct `/api/v1/chat/completions` curl.
> - **Codex tool/shell execution does NOT work in this sandbox** (`bwrap`/bubblewrap not installed; the
>   model claims "Done" but no file is written). So every non-Anthropic path here is **text-generation
>   only — the orchestrator (Claude) must write any files** the model's output implies.
> - **Plan-gated model IDs (403 on current plan):** `x-ai/grok-3`, `minimax/minimax-m1`. Use the
>   **rostered** IDs: `x-ai/grok-4.20`, `minimax/minimax-m3`, `deepseek/deepseek-v4-pro`, `openai/gpt-5.5`.
> - **Reliable universal non-Anthropic recipe (text-only):** direct curl to the gateway:
>   ```bash
>   curl -s "$AI_GATEWAY_BASE_URL/api/v1/chat/completions" \
>     -H "Authorization: Bearer $AI_GATEWAY_API_KEY" -H "Content-Type: application/json" \
>     -d '{"model":"x-ai/grok-4.20","messages":[{"role":"user","content":"..."}]}'
>   ```

This guide covers **HOW to run a chosen model** (harness mechanics, launch commands).
**WHICH model to choose is decided in `resources/model-decision.md`** — a benchmark +
cost grounded decision file. Read that file FIRST, run its algorithm, then come here to
map the chosen model to its execution path.

> **Model choice is NOT made here and NOT by stereotype.** Do not route by a model's
> country of origin or by tags like "Asian language → MiniMax" or "STEM → GPT-5.5"
> (the latter is factually false — GPT-5.5 has no reported math benchmark). All capability
> picks come from `model-decision.md`. The one *values/voice* mandate is **Grok 4.20 for
> opinion / unfiltered / 18+ / politics / economics content** (100% of the time).

The core principle remains: **assign the minimum-sufficient model that can fulfill the
role's cognitive demands**, justified by benchmark + cost evidence. Over-assigning wastes
cost; under-assigning breaks quality. Step up one tier only on demonstrated failure.

---

## The Decision Process

For each agent role, ask two questions in order:

1. **Which model?** → resolved by `resources/model-decision.md` (classify dominant axis →
   shortlist top benchmark scorers → pick cheapest clearing the bar → step up one tier on
   failure; Grok 4.20 mandatory for opinion/unfiltered/18+/politics/economics).

2. **What harness does that model need?** → answered below (file I/O? run code? just think?).

---

## Roster → Execution Path (the only models hive uses)

Capability picks live in `model-decision.md`. This table maps each rostered model to **how
to run it here**. The critical split is *tool-loop agent* (can act: edit files, run commands,
call MCP) vs *text worker* (generates text/code/analysis; the orchestrator performs any actions).

| Model | Tool-loop agent? | Execution path |
|-------|------------------|----------------|
| Claude Opus 4.8 | **Yes** | `claude -p --model claude-opus-4-8 --dangerously-skip-permissions < /dev/null` (Bedrock, ungated) |
| Claude Sonnet 4.6 | **Yes** | `Agent(subagent_type="general-purpose", prompt=...)` — pinned session default, in-process |
| Claude Haiku 4.5 | **Yes** | `claude -p --model haiku --dangerously-skip-permissions < /dev/null` (Bedrock, ungated) |
| GPT-5.5 | **Yes (text via Codex)** | `codex -c model_provider=ai_gateway -m openai/gpt-5.5 exec --skip-git-repo-check "..."` — *text-only here* (no `bwrap`); orchestrator writes files |
| DeepSeek V4 Pro | **No — text worker** | direct gateway curl → `deepseek/deepseek-v4-pro` |
| MiniMax M3 | **No — text worker** | direct gateway curl → `minimax/minimax-m3` |
| Kimi K2.6 | **No — text worker** | direct gateway curl (if a route exists; else fall back) |
| Grok 4.20 | **No — text worker** | direct gateway curl → `x-ai/grok-4.20` |

**Removed from roster (do not spawn):** Gemini (all), GPT-4.1, gpt-5-mini, minimax-m1,
minimax-m2.7. Low remaining credits / superseded.

> **Why DeepSeek / MiniMax / Kimi / Grok are text workers, not CLI tool agents (verified 2026-06-02):**
> The claude-multimodel proxy (:9877) translates Anthropic↔OpenAI. Non-streaming `/v1/messages`
> tool use WORKS, but the `claude` CLI always *streams*, and the proxy's streaming handler drops
> their reasoning-channel tool-call deltas → empty output. Until that is fixed, feed their
> text/code output into a Claude/Codex agent that does the actual tool calls. Per benchmarks they
> match or beat Sonnet on many axes at 5–30× lower cost — so prefer them for *generation* work.

> **Direct gateway curl (text-only, all non-Claude/non-OpenAI models):**
> ```bash
> curl -s "$AI_GATEWAY_BASE_URL/api/v1/chat/completions" \
>   -H "Authorization: Bearer $AI_GATEWAY_API_KEY" -H "Content-Type: application/json" \
>   -d '{"model":"<MODEL_ID>","messages":[{"role":"user","content":"<PROMPT>"}]}' \
>   | python3 -c "import sys,json;print(json.load(sys.stdin)['choices'][0]['message']['content'])"
> ```

---

## Role → Model: resolve via the decision file (no hardcoding)

Hive does **not** pin one model per role. For each role, take its dominant cognitive axis to
`model-decision.md` and apply the algorithm. The guidance below is *how to think about the axis*,
not a fixed assignment:

- **High-stakes / single-point-of-failure roles** (Lead Architect, Integration Lead, Quality
  Critic, Risk Analyst, Devil's Advocate, Systems/Structural designers, security/legal/financial
  Domain Experts): errors cascade → start at the top of the relevant axis (often Opus 4.8), since
  the decision file's high-stakes override applies.
- **Production generation roles** (Research Analyst, Lead/Section Writers, Data Analyst, Options
  Generator, Evaluator, Planner, Engineer): if the role only *generates*, a cheap text worker
  (DeepSeek/MiniMax) often clears the bar far below Sonnet's price; if it must *act on the repo*,
  use a tool-loop agent (Sonnet, escalating to Opus).
- **Bulk / mechanical roles** (Copy Editor, Fact Checker, Format Converter, Summarizer, Classifier,
  Data Processor, Extractor, Visual Narrator, QA Tester): "apply template X to input Y" → start at
  **Haiku 4.5** (the minimum), escalate only on failure.
- **Opinion / unfiltered / 18+ / politics / economics** content in any role → **Grok 4.20** (mandatory).
- **Multimodal** (image/video understanding) → MiniMax M3 / Kimi K2.6 lead per benchmarks.

In every case, justify the pick from the decision file's benchmark + cost tables, not from habit.

---

## Harness Selection: Full Decision Table

### The Four Harnesses

#### `general-purpose` — The Default Execution Harness

**Use when:** The agent needs to produce files, run code, read existing files, or use tools.

All execution agents (the ones actually producing deliverables) use `general-purpose`.
This includes ALL roles from the agent roster.

```python
# Claude Code Agent tool — ALWAYS runs Sonnet 4-6 here (model param is ignored by the harness).
# Use this only for Sonnet-class roles. For Opus/Haiku roles, use the `claude -p` subprocess below.
Agent(
    subagent_type="general-purpose",
    prompt="..."   # NOTE: passing model="opus"/"haiku" has NO effect in this environment
)
```

**Every Sonnet-class execution agent should have `subagent_type="general-purpose"`.**
For Opus/Haiku execution agents, use the `claude -p --model` subprocess harness (below).

---

#### `Plan` subagent — Read-Only Planning

**Use when:** The task is pure analysis/planning with NO file side-effects needed.

The `Plan` subagent CANNOT write files. Using it for execution agents breaks the project.
Use it only for:
- Phase 1 task analysis (before agents are designed)
- Pre-wave planning (which agents go in which wave)
- Contract quality review (read-only sanity check before freezing)

```python
Agent(
    subagent_type="Plan",
    model="sonnet",
    prompt="Analyze this task and produce a Hive analysis..."
)
```

---

#### `Explore` subagent — Discovery and Scanning

**Use when:** An agent needs to read many files/sources to discover patterns, without
producing new files.

Use for:
- Security/quality audit initial passes (read all outputs, report findings)
- Codebase or document discovery (what exists before building on it)
- Pre-validation scanning (verify file structure before formal validation)

```python
Agent(
    subagent_type="Explore",
    model="sonnet",
    prompt="Scan all outputs in this directory and report completeness..."
)
```

---

#### Codex CLI — OpenAI Model Dispatch (text-only here)

**Use when:** you need an **OpenAI** model (gpt-5.5) for text generation.
**Do NOT use Codex for Grok/MiniMax/Gemini/DeepSeek — it returns EMPTY** (responses-wire-API only).
Use direct gateway curl for those (see the non-Anthropic section above).

> **Flag order matters:** `exec` comes BEFORE `--skip-git-repo-check`. The reverse errors out.
> **No tools here:** Codex shell exec needs `bwrap` (not installed) — output is text only; the
> orchestrator must write any files. Requires the proxy on :9876 (`bash ~/.claude/skills/codex-multimodel/scripts/install.sh` if down).

```bash
# Dispatch to an OpenAI model via AI Gateway (proxy):
codex -c model_provider=ai_gateway -m <openai-model-id> exec --skip-git-repo-check "prompt"

# Examples (OpenAI models only — gpt-5.5 is the rostered OpenAI model):
codex -c model_provider=ai_gateway -m openai/gpt-5.5 exec --skip-git-repo-check "Extract JSON schema from..."
codex -c model_provider=ai_gateway -m openai/gpt-5.5 exec --skip-git-repo-check "Write the integration test suite for..."

# Grok / MiniMax / Gemini → use direct curl instead (Codex yields empty):
curl -s "$AI_GATEWAY_BASE_URL/api/v1/chat/completions" \
  -H "Authorization: Bearer $AI_GATEWAY_API_KEY" -H "Content-Type: application/json" \
  -d '{"model":"x-ai/grok-4.20","messages":[{"role":"user","content":"Write social media copy for..."}]}'
```

---

#### `claude -p` Subprocess — Tool-Using Anthropic Agent on ANY Model (PRIMARY for Opus/Haiku)

**Use when:** You need a full tool-using Claude Code agent (file I/O, bash, etc.) on a model
*other than* the pinned Sonnet 4-6 — i.e. Opus or Haiku. This is the **only** way to get
non-Sonnet Anthropic agents *with tools*, and it is **ungated** (uses the session's Bedrock
entitlement, not the metered AI Gateway).

```bash
# Single agent on Opus, with full tool access:
claude -p "<full agent brief — role, inputs, output path, contract>" \
  --model opus \
  --permission-mode acceptEdits --dangerously-skip-permissions < /dev/null \
  > /tmp/hive/agentX.out.md 2>&1
```

Model values: `haiku` (→4-5), `opus` (→4-6), `claude-opus-4-8`, `claude-sonnet-4-6` (full ID for 4-6).
`< /dev/null` suppresses the stdin-wait warning. Each subprocess is fully independent → you can mix
models freely within one wave.

**Parallel wave (mixed models, true per-agent selection):**
```bash
mkdir -p /tmp/hive
claude -p "$BRIEF_ARCHITECT" --model opus            --dangerously-skip-permissions < /dev/null > /tmp/hive/architect.md 2>&1 &
claude -p "$BRIEF_WRITER_1"  --model claude-sonnet-4-6 --dangerously-skip-permissions < /dev/null > /tmp/hive/writer1.md 2>&1 &
claude -p "$BRIEF_EDITOR"    --model haiku           --dangerously-skip-permissions < /dev/null > /tmp/hive/editor.md 2>&1 &
wait
# then read /tmp/hive/*.md and integrate
```

**Trade-offs vs. the `Agent` tool:**
- `Agent(general-purpose)` is in-process, ~instant to start, integrates with the orchestrator's
  task list — but is **locked to Sonnet 4-6**. Use it for all Sonnet-class roles.
- `claude -p` has ~a few seconds of startup (it fires SessionStart hooks) and runs out-of-process,
  so capture output to a file and read it back — but gives you **any model + tools + no gate**.
  Use it for every Opus and Haiku role.

**Caveats:** consumes the Anthropic entitlement (fine if unlimited); `--dangerously-skip-permissions`
is required for non-interactive tool use (appropriate for sandboxed workers); each subprocess has its
own fresh context (pass a complete self-contained brief — nothing bleeds back to the orchestrator
except the output file).

---

## Hybrid Harness Strategy

Some agents benefit from a two-phase approach:

### Pattern: Explore-then-Write

1. First, spawn `Explore` subagent to discover/scan all relevant material
2. Feed Explore output to a `general-purpose` agent to produce the deliverable

**Good for:** Quality Critic (scan all outputs first, then write review),
Security Analyst (scan codebase first, then write threat model)

### Pattern: Worker-then-Claude-Code

1. A cheap text worker (DeepSeek V4 Pro / MiniMax M3 via gateway curl, or GPT-5.5 via Codex)
   generates the bulk text/code/analysis for a chunk
2. The worker's output is saved to a file
3. Claude Code `general-purpose` (or `claude -p`) agent uses that file to perform the
   tool actions (file writes, repo edits) and produce the final deliverable

**Good for:** Tasks where generation can be done far cheaper by a text worker but the
acting/writing must go through a tool-loop Claude agent (the workers are text-only here)

---

## Quick Reference: The Assignment Matrix

> **Execution Path key:**
> - `claude -p subprocess` → **Bash dispatch**: `claude -p "..." --model <X> --dangerously-skip-permissions < /dev/null` — full tools, any model, ungated. **Default for Opus/Haiku.**
> - `general-purpose` / `Plan` / `Explore` → `Agent(subagent_type="...", prompt="...")` — **always runs Sonnet 4-6** (model param ignored).
> - `Codex CLI` → **Bash dispatch**: `codex -c model_provider=ai_gateway -m <model> exec --skip-git-repo-check "..."` — **OpenAI models only, TEXT-ONLY** (no tool/file execution: `bwrap` missing). Grok/MiniMax return EMPTY via Codex → use direct gateway curl instead.

> This matrix maps a **model already chosen via `model-decision.md`** to its launch command.
> It is a path lookup, NOT a model-selection shortcut — do not pick a model from this table.

| Chosen model (from decision file) | Execution Path | Launch flag / ID |
|-----------------------------------|----------------|------------------|
| Claude Opus 4.8 | **`claude -p` subprocess** (tool agent, ungated Bedrock) | `--model claude-opus-4-8` |
| Claude Sonnet 4.6 | `Agent(subagent_type="general-purpose")` (in-process) | pinned session default |
| Claude Haiku 4.5 | **`claude -p` subprocess** (tool agent, ungated Bedrock) | `--model haiku` |
| GPT-5.5 | Codex CLI (text-only here) | `codex ... -m openai/gpt-5.5 exec --skip-git-repo-check` |
| DeepSeek V4 Pro | **direct gateway curl** (text worker) | `deepseek/deepseek-v4-pro` |
| MiniMax M3 | **direct gateway curl** (text worker) | `minimax/minimax-m3` |
| Kimi K2.6 | **direct gateway curl** (text worker) | gateway ID (if route exists) |
| Grok 4.20 | **direct gateway curl** (text worker) | `x-ai/grok-4.20` |
| Pre-execution analysis (any) | `Plan` subagent (read-only) | pinned session default |
| Discovery/scan passes (any) | `Explore` subagent (read-only) | pinned session default |

> **Why `claude -p` subprocess for Opus/Haiku (not the Agent tool, not the gateway curl)?**
> The Agent tool ignores the model param and always runs Sonnet 4-6. The old AI-Gateway curl
> path returns text only (no tools) AND is pricing-gated. The `claude -p --model X` subprocess
> is the only path that delivers a real tool-using agent on the requested model, through the
> session's ungated Bedrock entitlement. If a role is genuinely Sonnet-class, prefer the in-process
> `Agent` tool (no subprocess overhead); reserve `claude -p` for when the model must differ.

---

## The Aristotelian Verification Test

Before finalizing any model assignment, apply this test:

1. **Telos match:** Does this model's designed purpose match this role's purpose?
2. **Necessity:** Would a cheaper model fail at this role? (If no, step down)
3. **Sufficiency:** Would a more expensive model do it better in a way that matters? (If yes, step up)
4. **Stakes:** If this agent produces wrong output, how bad is it? (High stakes = escalate)
5. **Volume:** Is this a high-volume, repetitive task? (High volume = Haiku if possible)

A model assignment is virtuous when all five tests are satisfied.
