# Hive Contract Template

The contract is the single source of truth for all agents in a swarm. A weak contract cascades
into swarm failure. Generate this with Opus 4.6. Do not compromise on contract quality — if
the contract is ambiguous, parallel agents drift independently in ways that are expensive to fix.

Replace every `{{PLACEHOLDER}}` below.

---

```
CRUCIBLE CONTRACT
=================
Project:        {{PROJECT_NAME}}
Task Family:    {{TASK_FAMILY}}
Mode:           {{COORDINATION_MODE}}
Total Agents:   {{AGENT_COUNT}}
Total Waves:    {{WAVE_COUNT}}
Generated:      {{TIMESTAMP}}

---

## 1. OBJECTIVE

{{OBJECTIVE_ONE_SENTENCE}}

Success means: {{SUCCESS_DEFINITION}}

---

## 2. DELIVERABLE MANIFEST

Every output produced by this swarm, its owner, and its format.

| # | Deliverable | Owner Agent | Format | Output Path | Consumed By |
|---|-------------|-------------|--------|-------------|-------------|
{{DELIVERABLE_TABLE_ROWS}}

Example row:
| 1 | Competitive analysis findings | Research Analyst | Markdown | outputs/research-findings.md | Strategist, Integration Lead |

No deliverable exists unless it is in this manifest. No agent produces deliverables not listed here.

---

## 3. SHARED REFERENCES

Definitions, terminology, facts, and conventions that all agents must align on.
These are not open to interpretation — agents cannot substitute their own definitions.

### Terminology
{{SHARED_TERMINOLOGY}}

Example:
- "market leader": company with >20% market share in their primary segment, not just revenue rank
- "competitive threat": direct product substitution risk, not adjacency

### Style Conventions
{{STYLE_CONVENTIONS}}

Example:
- Tone: professional but not academic; avoid jargon unless defined
- Tense: present tense for current state, past for historical, future for projections
- Numbers: always include units; percentages to one decimal place
- Citations: [Author, Year] inline; full reference at document end

### Factual Axioms
{{FACTUAL_AXIOMS}}

Example — facts all agents treat as given without re-researching:
- Scope is limited to: {{GEOGRAPHIC_SCOPE}} / {{TIME_SCOPE}} / {{DOMAIN_SCOPE}}
- Base year for all financial projections: {{BASE_YEAR}}
- Primary target audience: {{AUDIENCE}}

---

## 4. AGENT ROSTER AND WAVE SCHEDULE

| Agent Name | Role | Model | Harness | Wave | Status |
|------------|------|-------|---------|------|--------|
{{AGENT_ROSTER_ROWS}}

Example rows:
| Lead Architect | Lead Architect | anthropic/claude-opus-4.8 | general-purpose | 1 | ✓ Complete |
| Research Analyst | Research Analyst | anthropic/claude-sonnet-4.6 | general-purpose | 2 | Pending |

Wave execution order:
{{WAVE_EXECUTION_DESCRIPTION}}

---

## 5. INTERFACE DEFINITIONS

How outputs from one agent connect to inputs of another. This is where coordination
either works or breaks. Be specific — vague interfaces cause agents to produce
incompatible outputs that the Integration Lead cannot reconcile.

{{INTERFACE_DEFINITIONS}}

Example:
**Research → Strategy Interface:**
- Research Analyst produces: `outputs/research-findings.md`
  - Format: H2 sections per topic area; each finding as a bullet with evidence link
  - Required fields per finding: [claim, evidence source, confidence: high/med/low]
- Strategist consumes: above file
  - Strategist reads the `confidence: high` findings first when forming recommendations
  - Strategist MUST reference specific research finding IDs in their rationale

**Research → Risk Analyst Interface:**
- Risk Analyst reads `outputs/research-findings.md`
  - Focuses on `confidence: low` items as uncertainty sources
  - Does not reproduce the research; only references it

---

## 6. SECTION BOUNDARIES

What each agent owns exclusively. Prevents overlap and contradiction.

{{SECTION_BOUNDARIES}}

Example:
- Research Analyst owns: all factual claims about current market state
- Strategist owns: all forward-looking recommendations and rationale
- Risk Analyst owns: all risk identification and mitigation proposals
- NO agent writes in another agent's section
- Integration Lead may restructure sections but must not change substance

---

## 7. QUALITY CRITERIA

Measurable done criteria for each deliverable.

{{QUALITY_CRITERIA}}

Example:
**Research Findings:**
- [ ] Covers all 5 topic areas specified in the objective
- [ ] Each claim has a source; no unsupported assertions
- [ ] Confidence ratings applied to all findings
- [ ] 800–2000 words (not a dump, not too thin)

**Strategy Document:**
- [ ] At least 3 distinct strategic options presented
- [ ] Each option has: rationale, risk summary, resource requirements
- [ ] Final recommendation justified by research evidence
- [ ] References specific research findings by section

**Final Integrated Report:**
- [ ] All deliverables present and synthesized
- [ ] No contradictions between sections
- [ ] Reads as unified voice, not a patchwork
- [ ] Within target length: {{TARGET_LENGTH}}

---

## 8. A2A COMMUNICATION PROTOCOL

A2A Status: {{A2A_ENABLED}}  *(true = Hybrid Wave mode, false = Parallel mode)*

{{#if A2A_ENABLED}}
### Permitted Channels

Agents may message each other directly using `SendMessage(to="<agent-name>", ...)`.
The team name for this swarm is: **{{TEAM_NAME}}**

**Permitted triggers only — agents must not message for any other reason:**

| Trigger | Sender → Recipient | Required message content |
|---------|--------------------|--------------------------|
| Interface deviation | Deviating agent → its consumers | Path + exact delta from contract spec |
| Partial output ready | Producer → blocked consumer | Path + which section is final vs. in-progress |
| Peer-resolvable blocker | Blocked agent → owner of blocking data | Specific gap + what data is needed + path to write it |

### A2A Rules

- Message the **specific peer** affected — do not broadcast unless every peer is equally affected (rare)
- Keep messages **factual and brief** — one paragraph maximum
- The orchestrator (team lead) receives all messages automatically — do not CC them
- A2A does not replace the contract. If the answer is in the contract, do not send a message
- After resolving via A2A, update your output file to reflect the resolution
- **Maximum 2 A2A messages per agent per wave** — if you need more, the coupling score was
  underestimated and the orchestrator should be notified to re-evaluate the mode

### A2A Message Log

Agents that send or receive A2A messages should append a brief log entry to their output file:

```
A2A LOG
-------
[sent|received] from/to: [agent name] | trigger: [interface-deviation|partial-ready|blocker]
summary: [one sentence]
resolved: [yes/no] | action taken: [what changed]
```
{{else}}
A2A is disabled. This is a Contract-First Parallel run. All coordination is via the contract
and output files only. Agents must not message each other — the contract is sufficient.
{{/if}}

---

## 9. ESCALATION PROTOCOL

If an agent encounters a blocking problem, they should:
1. Document the blocker clearly in their output file header
2. If A2A is enabled and the blocker is peer-resolvable — send one targeted message first
3. Produce as much of their deliverable as they can
4. Mark uncertain/incomplete sections explicitly
5. Do NOT halt — partial output with clear documentation is better than silence

Integration Lead resolves residual blockers during the final wave.

---

## 10. CONTRACT SIGNATURE

This contract was generated by the Lead Architect agent.
All agents receive this contract in full before beginning work.

Lead Architect: {{LEAD_ARCHITECT_NOTES}}
Team Name (A2A): {{TEAM_NAME}}
```

---

## Contract Generation Instructions

To generate a contract for a new project using Opus (via the `claude -p` subprocess — full tools, ungated; do NOT route Anthropic models through Codex):

```bash
claude -p "You are the Lead Architect for a Hive swarm. Generate a complete Hive Contract for:

PROJECT: {{PROJECT_DESCRIPTION}}
TASK FAMILY: {{TASK_FAMILY}}
COMPONENTS: {{COMPONENT_LIST}}
AGENTS: {{AGENT_LIST}}
QUALITY TIER: {{QUALITY_TIER}}

Use the Hive Contract template structure exactly. Every section must be filled.
Be specific in the interface definitions — vague interfaces cause coordination failures.
The contract is the single source of truth; completeness matters more than brevity.

Write the finished contract to ./tmp/hive_contract.md using the Write tool." \
  --model opus --permission-mode acceptEdits --dangerously-skip-permissions < /dev/null
```

Or if the current session is already Opus, generate the contract inline following the
template above without a subprocess.

## What Makes a Contract Good

**Good contract signals:**
- Every deliverable has an exact output path and format spec
- Every interface says exactly what field names/structure is expected
- Section boundaries say what each agent must NOT touch, not just what they own
- Quality criteria are checkboxes, not vague adjectives ("comprehensive" is not a criterion; "covers all 5 areas specified" is)

**Weak contract signals:**
- "Agent B uses Agent A's output" (no format spec, no field names)
- "High quality output" (not measurable)
- Missing section boundaries (agents overlap and contradict each other)
- Terminology undefined (agents interpret the same word differently)

A weak contract is the leading cause of Hive swarm failure. Invest in contract quality first.
