# Model Decision File (Authoritative)

> **READ THIS FIRST, then decide.** The orchestrator MUST consult this file before
> spawning any agent. Pick the model by matching the task's dominant axis to the
> benchmark evidence below, then choosing the **cheapest model that clears the bar**.
> Decisions are grounded in benchmarks + price, NOT in the model's country of origin,
> NOT in stereotypes, and NOT in a blanket "default model".
>
> Data: June 2026 benchmark set (see `uploads/model-benchmarks/` in workspace, or the
> tables reproduced below). All numbers are from official system cards / technical
> papers / Artificial Analysis / Scale Labs.

---

## 1. Roster (the only models hive uses)

| Model | Gateway / harness ID | Role in hive |
|-------|----------------------|--------------|
| Claude Opus 4.8 | `claude-opus-4-8` (Bedrock via `claude -p`) | Top-tier tool agent; high-stakes / hardest reasoning |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` (Agent tool default) | Balanced tool agent; mid-tier work |
| Claude Haiku 4.5 | `claude-haiku-4-5` (Bedrock via `claude -p`) | Cheapest Claude tool agent; mass / low-stakes work |
| DeepSeek V4 Pro | `deepseek/deepseek-v4-pro` (gateway curl) | Cheap text/reasoning/code worker |
| MiniMax M3 | `minimax/minimax-m3` (gateway curl) | Cheap text/reasoning/code/multimodal worker |
| Kimi K2.6 | `minimax/...`→ N/A; gateway curl | Cheap multimodal + swarm-coordination worker |
| GPT-5.5 | `openai/gpt-5.5` (Codex CLI) | Tool agent when OpenAI ecosystem / its coding profile is needed |
| Grok 4.20 | gateway curl | **Mandatory** for opinion / unfiltered / 18+ / politics / economics |

**Removed from hive:** Gemini, GPT-4.1, gpt-5-mini, minimax-m1, minimax-m2.7
(low remaining credits / superseded). Do not spawn these.

> Note on Kimi: there is no dedicated gateway alias verified for Kimi in this sandbox;
> use it as a *text/reasoning worker via gateway curl* only where its specific
> advantages (multimodal MMMU, swarm design reasoning) matter and a gateway route exists.
> If no route, fall back to the next-best model on that axis.

---

## 2. Price (output $/MTok — the cost that dominates generation)

Cheapest → most expensive:

| Rank | Model | Input | Output | Note |
|------|-------|-------|--------|------|
| 1 | DeepSeek V4 Pro | $0.44 | **$0.87** | 75% promo; full = $1.74/$3.48 |
| 2 | MiniMax M3 | $0.60 | **$2.40** | ≤512K tier; OpenRouter $0.30/$1.20 |
| 3 | Kimi K2.6 | $0.95 | **$4.00** | open weights |
| 4 | Claude Sonnet 4.6 | $3.00 | **$15.00** | |
| 5 | Claude Opus 4.8 | $5.00 | **$25.00** | |
| 6 | GPT-5.5 | $5.00 | **$30.00** | most expensive output |

Haiku 4.5 / Grok 4.20: no benchmark file; Haiku = cheapest Claude tool agent,
Grok = values/voice pick (not a capability pick).

---

## 3. Benchmark evidence by axis (the facts decisions rest on)

**SWE-bench Verified (real-world coding):** Opus 88.6 > DeepSeek 80.6 ≈ Kimi 80.2 ≈ Sonnet 79.6 ≈ MiniMax ~79. GPT-5.5 not reported.
**SWE-bench Pro (hard multi-file):** Opus 69.2 > MiniMax 59.0 > Kimi 58.6 > DeepSeek 55.4.
**LiveCodeBench v6 (competitive):** DeepSeek 93.5 > Kimi 89.6. (Opus/Sonnet/MiniMax/GPT not reported.)
**Codeforces ELO:** DeepSeek 3206 (beats GPT-5.4 ~3168) > Gemini 3052.
**Math:** Kimi AIME2026 96.4 · Opus USAMO2026 96.7 · Sonnet AIME2025 95.6 · DeepSeek HMMT2026 95.2. **GPT-5.5 has NO reported math score.**
**GPQA Diamond (reasoning):** Opus 93.6 > Kimi 90.5 > DeepSeek 90.1 > Sonnet 89.9.
**HLE w/ tools:** Opus 57.9 > Kimi 54.0 > Sonnet 49.0 > DeepSeek 48.2.
**MCP-Atlas (tool use):** Opus 82.2 > MiniMax 74.2 > DeepSeek 73.6 >> Sonnet 61.3.
**OSWorld (GUI/OS agent):** Opus 83.4 > Kimi 73.1 > Sonnet 72.5.
**Terminal-Bench:** Opus 74.6 > DeepSeek 67.9 > Kimi 66.7 > MiniMax 66.0 > Sonnet 59.1.
**BrowseComp (web):** Opus 84.3 ≈ MiniMax 83.5 ≈ DeepSeek 83.4 ≈ Kimi 83.2 (tied within 1.1 pt).
**GDPval-AA ELO (real professional work):** Opus 1890 >> Sonnet 1633 > DeepSeek 1554.
**MMMU-Pro (multimodal):** Kimi 79.4 > Sonnet 74.5. DeepSeek = text-only (no vision).
**Multi-agent swarm:** Kimi K2.6 uniquely coordinates 300 sub-agents / 4000+ steps.
**Context:** all 1M except Kimi 256K. DeepSeek largest max output (384K).
**No benchmark data:** Grok 4.20, Haiku 4.5 (treat as judgment picks).

---

## 4. Execution-path reality (HappyCapy sandbox) — capability gate

A model can only be a **tool-using agent** if its harness supports a tool loop here.
Verified state (2026-06-02):

| Model | Tool-loop agent? | How to run |
|-------|------------------|------------|
| Opus 4.8 / Sonnet 4.6 / Haiku 4.5 | **Yes** | Agent tool (→Sonnet), or `claude -p --model <id> --dangerously-skip-permissions < /dev/null` |
| GPT-5.5 | **Yes** | Codex CLI: `codex -c model_provider=ai_gateway -m openai/gpt-5.5 exec ...` |
| DeepSeek V4 Pro | **Text/reasoning/code only** | gateway curl `$AI_GATEWAY_BASE_URL/api/v1/chat/completions` |
| MiniMax M3 | **Text/reasoning/code only** | gateway curl |
| Kimi K2.6 | **Text/reasoning only** | gateway curl (if route exists) |
| Grok 4.20 | **Text only** | gateway curl |

**Why DeepSeek/MiniMax/Kimi are NOT tool agents here:** the claude-multimodel proxy
(port 9877) translates Anthropic↔OpenAI. Non-streaming `/v1/messages` tool use WORKS,
but the `claude` CLI always *streams*, and the proxy's streaming handler drops
reasoning-channel tool-call deltas → empty output. Until that is fixed, treat these
three as **workers that produce text/code/analysis**, fed into a Claude/Codex agent that
does the actual tool calls (file writes, command execution). This is the "verify then
rely" boundary — do not advertise them as autonomous CLI agents.

**Consequence for decisions:** if a step needs to *act* (edit files, run commands, call
MCP tools) it must go to a tool-loop agent (Claude tier or GPT-5.5). If a step only
needs to *generate* (write text, reason, draft code, analyze), the cheap text workers
are first-class — and per benchmarks they match or beat Sonnet on many axes at 5–30× lower cost.

---

## 5. Decision algorithm

```
1. CONTENT-TYPE OVERRIDE (check first, non-negotiable):
   - Opinion / untraditional / diverse-viewpoint / 18+ / politics / economics /
     "real human opinion" / unfiltered  →  GROK 4.20  (100% of the time).

2. CLASSIFY the step's dominant axis:
   coding-real | coding-competitive | math/proof | reasoning/knowledge |
   tool-use/agentic | web-research | os-gui | multimodal | long-context |
   swarm-coordination | bulk/low-stakes | general.

3. NEEDS-TO-ACT?  Does the step call tools (edit files, run cmds, MCP)?
   - YES → candidate set = tool-loop agents only {Opus, Sonnet, Haiku, GPT-5.5}.
   - NO  → candidate set = all roster models (cheap text workers included).

4. SHORTLIST by benchmark: take the top scorers on the dominant axis (Section 3).

5. PICK CHEAPEST that clears the quality bar (Section 2 price order).
   - Minimum-sufficient rule: start at the cheapest qualifying model;
     step up exactly ONE tier only on demonstrated failure (bad output / verification fail).
   - "Use Haiku minimum" for bulk/low-stakes: start at Haiku, escalate only if it fails.

6. HIGH-STAKES / IRREVERSIBLE override: if an error is costly or hard to reverse,
   start at Opus 4.8 regardless of cost.
```

---

## 6. Axis → recommended pick (apply algorithm, don't hardcode)

| Dominant axis | Needs to act? | First pick (cheapest clearing bar) | Step-up |
|---------------|---------------|-----------------------------------|---------|
| Real-world coding (generate code) | no | DeepSeek V4 Pro (80.6, $0.87) | MiniMax → Sonnet → Opus |
| Real-world coding (edit/run repo) | yes | Sonnet 4.6 (tool agent, 79.6) | Opus 4.8 (88.6) |
| Competitive / algorithmic coding | no | DeepSeek V4 Pro (LC 93.5, CF 3206) | Kimi K2.6 |
| Math / proof | no | DeepSeek/Kimi (cheap, ~95) | Opus (USAMO 96.7) |
| Reasoning / knowledge | no | DeepSeek V4 Pro (GPQA 90.1, $0.87) | Kimi → Opus (93.6) |
| Tool use / agentic (must act) | yes | Sonnet 4.6 | Opus 4.8 (MCP 82.2) |
| Web research / browsing | depends | DeepSeek/MiniMax/Kimi (~83, tied) | Opus (84.3) |
| OS / GUI automation | yes | Sonnet 4.6 (72.5) | Opus 4.8 (83.4) |
| Multimodal (image/video) | no | MiniMax M3 / Kimi K2.6 (79.4) | Sonnet 4.6 (74.5) |
| Long-context (1M, big output) | no | DeepSeek V4 Pro (384K out, MRCR 83.5) | Opus 4.8 (MRCR 92.9) |
| Multi-agent swarm design | no | Kimi K2.6 (300-agent design) | Opus 4.8 |
| Bulk / low-stakes / mass info | yes | **Haiku 4.5** (minimum) | Sonnet 4.6 |
| Opinion / 18+ / politics / econ | text | **Grok 4.20** (mandatory) | — |
| General, no clear axis | yes | Sonnet 4.6 (balanced) | Opus / down to Haiku |

> There is **no blanket default**. "General" defaults to Sonnet only because it is the
> balanced tool-loop agent, not because Sonnet is privileged. If the step doesn't need
> to act and has a clear axis, a cheaper worker usually wins.

---

## 7. Anti-bias rules (explicit)

- **Never** route by the model's country / lab ("Chinese lab → Asian language"). That is
  a debunked stereotype and is banned. MiniMax/DeepSeek/Kimi are chosen on benchmarks + price.
- **Never** claim "STEM/math → GPT-5.5": GPT-5.5 has **no reported math benchmark**;
  Kimi/Opus/Sonnet/DeepSeek lead math. Route math to them.
- **Never** keep a model as "default" just by habit. Justify every pick from Sections 2–3.
- Grok 4.20 is the one *values/voice* mandate (opinion/unfiltered/18+/politics/economics),
  and that is a deliberate scope decision, not a capability ranking.
