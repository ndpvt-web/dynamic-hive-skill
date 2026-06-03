# Hive Coordination Modes

Three modes, selected by coupling score. The coupling score is a continuous measure
of how tightly the project's components depend on each other.

## Mode Decision

```
coupling_score < 0.4  →  CONTRACT-FIRST PARALLEL
coupling_score 0.4-0.7  →  HYBRID WAVE
coupling_score > 0.7  →  PROGRESSIVE-LAYER SEQUENTIAL

Override: if any component cannot start without another's actual output
          (not just its spec), add 0.2 to coupling_score before deciding.
```

---

## Mode 1: Contract-First Parallel (coupling < 0.4)

**When the components are largely independent.** Each agent can work from the contract
spec alone, without waiting for another agent's actual output.

**Execution:**
1. Generate contract (Opus)
2. Spawn ALL agents in ONE message simultaneously
3. Each agent receives the full contract + its specific assignment
4. Wait for all to complete
5. Validate and merge

**Typical projects:**
- Parallel research threads on different subtopics
- Multi-platform content (same content adapted for blog / Twitter / email)
- Independent transform jobs on separate datasets
- Modular document sections with minimal cross-references

**Speedup:** 4–8x vs sequential.

**Risk:** Contract must be comprehensive. If the contract is incomplete, all agents
drift independently. Invest heavily in contract quality for this mode.

**Spawning pattern:**
```python
# All agents in ONE message
Agent(subagent_type="general-purpose", model="sonnet", prompt=agent_1_brief)
Agent(subagent_type="general-purpose", model="haiku", prompt=agent_2_brief)
Agent(subagent_type="general-purpose", model="sonnet", prompt=agent_3_brief)
# ...all in the same turn
```

---

## Mode 2: Hybrid Wave (coupling 0.4–0.7)

**When some components depend on others, but not everything.** Group independent
components into waves. Waves execute sequentially; agents within a wave run in parallel.

**A2A Communication is active in this mode.** Agents within the same wave can message
each other directly when they encounter interface deviations, partial-output availability,
or blockers that a peer can resolve — without routing through the orchestrator.

**Execution:**
1. Generate contract (Opus)
2. Sort components into waves using dependency graph
3. **Create team** with `TeamCreate` (once, before any agents are spawned)
4. For each wave:
   a. Spawn all wave agents simultaneously (one message), each with `name=` + `team_name=`
   b. Agents communicate directly as needed during execution (see A2A rules below)
   c. Wait for all wave agents to complete
   d. Validate wave outputs
   e. Freeze outputs (make them available to next wave)
5. Final integration
6. Shut down team with `SendMessage(to="*", message={"type": "shutdown_request", "reason": "Hive swarm complete"})`

**Wave formation rules:**
- Wave 1: Components with no dependencies on other components
- Wave N+1: Components whose ALL dependencies are in Wave N or earlier
- Integration Lead always goes in the final wave, alone

**Typical projects:**
- Full research → strategy documents (research Wave 1, strategy Wave 2)
- Multi-section reports where later sections cite earlier ones
- Analysis → visualization → presentation pipelines
- Foundation → extensions patterns

**Speedup:** 2–5x vs sequential.

**Between-wave validation:** After each wave, check that:
- All promised outputs exist
- Interface contracts (what one wave promised to deliver) are honored
- No wave broke the style conventions set in the contract

---

### A2A Rules for Hybrid Wave

Agents should message peers **only** for these three triggers — not for general chatter:

| Trigger | Who sends | Message content |
|---------|-----------|----------------|
| Interface deviation | The deviating agent → consumers | "My output at `path` differs from contract spec: [exact delta]. Adjust your dependency." |
| Partial output ready | Producer → blocked consumer | "Partial output available at `path` — you can begin consuming section X now." |
| Peer-resolvable blocker | Blocked agent → the peer who owns the blocking data | "Blocked on: [specific gap]. Can you produce [X] before completing your main deliverable?" |

**Do NOT message for:** status updates, acknowledgments, general questions, or anything
the contract already answers. The contract is the primary coordination layer. A2A is
the exception channel for real-time deviations only.

**Orchestrator always hears it all.** Because Claude Code is the team lead, all
`SendMessage` calls from agents arrive as orchestrator notifications automatically.
The orchestrator does not need to poll — but should not interrupt unless a message
signals a contract-level conflict requiring resolution.

---

### Spawning Pattern (Hybrid Wave — A2A enabled)

```python
# Step 1: Create the team ONCE before spawning any agents
TeamCreate(team_name="hive-wave-run-{timestamp}")

# Step 2: Wave 1 — spawn as named teammates in ONE message
Agent(
    subagent_type="general-purpose", model="sonnet", name="researcher",
    team_name="hive-wave-run-{timestamp}", prompt=w1_researcher_brief
)
Agent(
    subagent_type="general-purpose", model="sonnet", name="analyst",
    team_name="hive-wave-run-{timestamp}", prompt=w1_analyst_brief
)
# Now researcher can: SendMessage(to="analyst", message="...")
# And analyst can: SendMessage(to="researcher", message="...")

# --- Wait for Wave 1 to complete ---

# Step 3: Wave 2 — new agents, same team, receive frozen Wave 1 outputs
Agent(
    subagent_type="general-purpose", model="opus", name="strategist",
    team_name="hive-wave-run-{timestamp}", prompt=w2_strategist_brief_with_wave1_outputs
)
Agent(
    subagent_type="general-purpose", model="sonnet", name="risk-analyst",
    team_name="hive-wave-run-{timestamp}", prompt=w2_risk_brief_with_wave1_outputs
)

# Step 4: After final wave, shut down team
SendMessage(to="*", message={"type": "shutdown_request", "reason": "Hive swarm complete"})
TeamDelete()
```

**Parallel mode (coupling < 0.4) does NOT use TeamCreate.** Anonymous `Agent` calls
are sufficient when components are truly independent — adding A2A messaging overhead
to fully parallel work provides no benefit.

---

## Mode 3: Progressive-Layer Sequential (coupling > 0.7)

**When everything depends on everything.** Build layer by layer; each layer's actual
output (not just its spec) becomes the foundation for the next.

**Execution:**
1. Generate initial contract (Opus — may be incomplete, that's OK)
2. Identify Layer 1: the foundational components with no dependencies
3. For each layer:
   a. Spawn ONE agent per layer component (or all in parallel if within-layer is independent)
   b. Validate output
   c. If issues: up to 3 fix iterations, then move on
   d. Freeze layer output
   e. Update contract if emergent insights require it
4. Final integration

### Within-Layer Parallel (when layer components are independent)

When a layer has multiple independent components, spawn them in parallel using the same
pattern as Hybrid Wave — with TeamCreate and named agents for A2A communication.

```python
# Multiple independent components in Layer 2 — use TeamCreate for within-layer A2A
TeamCreate(team_name="hive-{project-slug}-layer2")

Agent(subagent_type="general-purpose", model="claude-sonnet-4.6",
      name="section-a", team_name="hive-{project-slug}-layer2",
      prompt=layer2_section_a_brief)  # A2A_ENABLED=true
Agent(subagent_type="general-purpose", model="claude-sonnet-4.6",
      name="section-b", team_name="hive-{project-slug}-layer2",
      prompt=layer2_section_b_brief)  # A2A_ENABLED=true

# Wait for layer 2 to complete, then TeamDelete before starting layer 3
SendMessage(to="*", message={"type": "shutdown_request", "reason": "Layer 2 complete"})
TeamDelete()
```

**Rule:** If within-layer components are truly independent (coupling < 0.4 within the layer),
anonymous parallel `Agent` calls without TeamCreate are also acceptable. Use TeamCreate
only when within-layer A2A is expected to be needed.

---

**When to use sequential:**
- Complex strategic decisions where the problem framing (Layer 1) changes everything
- Long-form narratives where early chapters constrain later ones thematically
- Research where each finding changes the next research question
- Iterative design processes with high emergent complexity

**Speedup:** Only 1.2–2x vs pure sequential. This mode prioritizes correctness over speed.

**Layer iteration limit:** Max 3 fix attempts per layer. After 3, move on and let
Integration Lead resolve any remaining issues. Infinite iteration loops destroy projects.

**Contract evolution:** Unlike Parallel mode, the contract CAN evolve between layers
in Sequential mode. If Layer 1 produces a surprising insight, update the contract
before spawning Layer 2.

**Spawning pattern:**
```python
# Layer 1
output_layer_1 = Agent(
    subagent_type="general-purpose", model="opus",
    prompt=f"You are Layer 1. CONTRACT: {contract}. Your deliverable: {layer_1_spec}"
)

# Validate Layer 1
# Fix if needed (max 3 tries)
# Freeze Layer 1

# Layer 2 — receives frozen Layer 1 output
output_layer_2 = Agent(
    subagent_type="general-purpose", model="sonnet",
    prompt=f"CONTRACT: {contract}. FROZEN LAYER 1: {output_layer_1}. Build Layer 2: {layer_2_spec}"
)
```

---

## Fallback Logic

### Parallel → Hybrid (when parallel fails)

If >30% of parallel agents produce outputs that don't align with contract:
1. Stop
2. Re-analyze coupling (it was higher than estimated)
3. Switch to Hybrid Wave
4. Use successful parallel outputs as Wave 1 input
5. Re-run failed agents with more explicit dependencies

### Hybrid → Sequential (when hybrid fails)

If wave validation repeatedly fails:
1. Identify which wave has circular dependencies
2. Convert that wave to sequential within-wave ordering
3. Continue

### Iteration Cap (prevent infinite loops)

- Any single agent: max 3 fix iterations
- Any single layer: max 3 iterations
- Entire project: max 2 full swarm retries

After caps are reached, proceed with partial output + document what failed.

---

## Mode Selection by Task Type

| Task | Typical Coupling | Recommended Mode |
|------|-----------------|-----------------|
| Parallel research on independent topics | 0.15–0.3 | Parallel |
| Multi-format content adaptation | 0.1–0.25 | Parallel |
| Report with distinct independent sections | 0.3–0.45 | Parallel or Hybrid |
| Full research → analysis → strategy | 0.45–0.6 | Hybrid |
| Multi-section narrative document | 0.5–0.65 | Hybrid |
| Complex strategic plan with cascading decisions | 0.65–0.8 | Sequential |
| Investigative research where each finding changes the next | 0.7–0.9 | Sequential |
| Design iteration with emergent requirements | 0.75–0.95 | Sequential |
