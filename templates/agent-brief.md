# Hive Agent Brief Template

Fill every `{{PLACEHOLDER}}` before sending. Agents given partial context produce significantly
more coordination errors. Every execution agent receives the FULL contract, not just their section.

---

```
CRUCIBLE AGENT BRIEF
====================

Agent Name:     {{AGENT_NAME}}
Role:           {{AGENT_ROLE}}
Model Class:    {{MODEL_CLASS}}
Model Rationale: {{MODEL_RATIONALE}}
Harness:        {{HARNESS_TYPE}}
Wave:           {{WAVE_NUMBER}} of {{TOTAL_WAVES}}

---

## YOUR CONTRACT (Shared source of truth — all agents receive this)

{{CONTRACT}}

---

## YOUR SPECIFIC ASSIGNMENT

{{ASSIGNMENT}}

Deliverable(s) you must produce:
{{DELIVERABLE_LIST}}

Output location(s):
{{OUTPUT_PATHS}}

---

## DEPENDENCIES (What you consume)

{{DEPENDENCIES}}

If a dependency file is listed but not yet available, check the output paths listed
in the contract. If still missing, note it in your output and proceed with what you have.

---

## EXPORTS (What you produce for other agents)

{{EXPORTS}}

Other agents are waiting for these outputs. Produce them in the exact format specified
in the contract. Do not deviate from the format — format mismatches cascade silently.

---

## QUALITY CRITERIA

{{QUALITY_CRITERIA}}

Your output passes when all criteria are satisfied. If you cannot satisfy a criterion,
document why and what you achieved instead — partial quality with clear documentation
is better than silence.

---

## CONSTRAINTS

- Stay within your section boundaries as defined in the contract.
- Do not modify files owned by other agents.
- Do not add scope beyond your assignment — breadth creep costs other agents integration time.
- If you discover a gap or contradiction in the contract, note it in your output header,
  then proceed with the best reasonable interpretation. Do not halt.

---

## AGENT-TO-AGENT COMMUNICATION (A2A)

A2A Status: {{A2A_ENABLED}}
Team Name:  {{TEAM_NAME}}
Your Peers: {{PEER_AGENTS}}

{{#if A2A_ENABLED}}
You are operating in **Hybrid Wave mode** with direct peer messaging enabled.
Use SendMessage to contact peers — but ONLY for these three triggers:

1. **Interface deviation** — if your output format must deviate from the contract spec:
   Message the consumers listed in your EXPORTS section immediately.
   Format: "Interface change at [path]: [what changed vs contract spec]. Adjust your dependency."

2. **Partial output available** — if a peer is blocked waiting for your output and you
   have a usable section ready before full completion:
   Message them with the path and which section is safe to consume.
   Format: "Partial output ready at [path]: section [X] is final. Do not consume [Y] yet."

3. **Peer-resolvable blocker** — if you are blocked and the blocker is data that another
   agent owns and could provide early:
   Message that specific peer only.
   Format: "Blocked on: [specific gap]. Can you write [X] to [path] before completing your deliverable?"

**Limit A2A messages to only the necessary triggers.** If you find yourself wanting to send
more than 2–3 messages, stop — that signal means the coupling score was underestimated.
Notify the orchestrator by appending a note to your output file header, not by sending more messages.

**Do NOT message for:** status, confirmations, general questions, or anything the contract answers.
**Do NOT broadcast** unless the issue affects every peer equally (extremely rare).

The orchestrator (team lead) receives all messages automatically — no need to report upward.
{{else}}
A2A is disabled for this wave. This is a Contract-First Parallel run — components are
independent. Coordinate only through the contract and your output files.
{{/if}}

---

## EXECUTION NOTES

Harness context: {{HARNESS_TYPE}}
- general-purpose: You have full tool access (Read, Write, Edit, Bash, web search).
  Use them freely to produce your deliverables.
- Plan: Read-only. Produce your analysis as text output only — no file writes.
- Explore: Discovery mode. Read and scan; report findings; no writes.
- Codex CLI (OpenAI models only, TEXT-ONLY): You were dispatched via
  `codex exec --skip-git-repo-check`. You have NO tool/file execution in this env
  (`bwrap` missing) — produce your output as text to stdout; the orchestrator writes
  any files. Note flag order: `exec` comes BEFORE `--skip-git-repo-check`. Grok/MiniMax/
  Gemini return EMPTY via Codex; the orchestrator reaches those via direct gateway curl.

Begin immediately. Do not re-introduce yourself or recap this brief.
Produce your deliverable(s) and stop.
```

---

## Placeholder Reference

| Placeholder | Content | Example |
|-------------|---------|---------|
| `{{AGENT_NAME}}` | Unique name for this agent instance | `Research Analyst Alpha` |
| `{{AGENT_ROLE}}` | Role from the agent roster | `Research Analyst` |
| `{{MODEL_CLASS}}` | Model assigned | `anthropic/claude-sonnet-4.6` |
| `{{MODEL_RATIONALE}}` | One sentence why this model was chosen | `Sonnet for synthesis work with reasonable depth` |
| `{{HARNESS_TYPE}}` | Execution environment | `general-purpose` |
| `{{WAVE_NUMBER}}` | Which wave this agent runs in | `2` |
| `{{TOTAL_WAVES}}` | Total number of waves | `4` |
| `{{CONTRACT}}` | Full contract text verbatim | *(see contract template)* |
| `{{ASSIGNMENT}}` | One paragraph describing this agent's task | `Synthesize the three source documents...` |
| `{{DELIVERABLE_LIST}}` | Bulleted list of outputs | `- outputs/research-findings.md` |
| `{{OUTPUT_PATHS}}` | File paths where outputs go | `outputs/research-findings.md` |
| `{{DEPENDENCIES}}` | Files/outputs this agent needs | `outputs/contract.md (this wave), none (Wave 1)` |
| `{{EXPORTS}}` | Files this agent produces for others | `outputs/research-findings.md → Strategy Agent` |
| `{{QUALITY_CRITERIA}}` | Measurable done criteria from contract | `- Covers all 5 topic areas specified` |
| `{{A2A_ENABLED}}` | `true` (Hybrid Wave) or `false` (Parallel) | `true` |
| `{{TEAM_NAME}}` | Team name from TeamCreate (Hybrid Wave only) | `hive-wave-run-1747312` |
| `{{PEER_AGENTS}}` | Named peers in this wave with their deliverables | `researcher → outputs/findings.md, analyst → outputs/data.md` |

---

## Quick-Fill Example: Research Analyst

```
Agent Name:     Research Analyst
Role:           Research Analyst
Model Class:    anthropic/claude-sonnet-4.6
Model Rationale: Synthesis of multiple sources requires evaluation depth that Haiku cannot
                 provide; Opus is not justified for standard research without extreme stakes.
Harness:        general-purpose
Wave:           2 of 4

CONTRACT:
[paste full contract here]

YOUR SPECIFIC ASSIGNMENT:
Research the competitive landscape for electric vehicle charging infrastructure
in Europe, focusing on market leaders, pricing models, and regulatory environment.
Produce an annotated findings document — not raw data dumps, but evaluated synthesis.

Deliverable(s) you must produce:
- outputs/research-findings.md (primary deliverable)
- outputs/source-list.md (secondary: all sources used with credibility ratings)

Output location(s):
- outputs/research-findings.md
- outputs/source-list.md

DEPENDENCIES:
- outputs/contract.md (already exists)
- No outputs from other agents required for this wave.

EXPORTS:
- outputs/research-findings.md → consumed by: Strategist, Risk Analyst (Wave 3)
- outputs/source-list.md → consumed by: Fact Checker (Wave 3), Integration Lead (Wave 4)

QUALITY CRITERIA:
- Covers: market leaders (top 5), pricing models (at least 3 categories), regulatory environment (EU + 3 major markets)
- Each finding is attributed to a source
- No raw data dumps — every section is evaluated synthesis, not copy-paste
- Document length: 800–1500 words
```
