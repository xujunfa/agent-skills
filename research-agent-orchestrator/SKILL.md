---
name: research-agent-orchestrator
description: >
  Route research tasks to the best external AI agent(s) — ChatGPT, Gemini, Grok,
  Claude, Manus, or combinations. Performs foundation web search, decides which
  agent(s) to use and why, generates high-quality prompts, and supports
  single-agent, parallel, and serial multi-agent workflows.
  Triggers: "research orchestration"|"choose best research agent"|
  "route to ChatGPT"|"route to Gemini"|"route to Grok"|"route to Claude"|
  "route to Manus"|"which AI for this research"|"multi-agent research"|
  "generate research prompt"|"delegate research to external agents"|
  "研究编排"|"选哪个AI研究"|"多Agent研究"|"research workflow"
---

# Research Agent Orchestrator

Meta-skill for routing research tasks to external AI agents. Decides which agent, why, in what order, with what prompt, and what output to expect.

**Relationship to `agent-research`:** That skill generates briefs and handles quality gating. This skill sits upstream — it decides *who gets the brief* and *how agents coordinate*. Use both together for full orchestration.

## Workflow

### Phase 1 — Foundation Search

Before delegating anything, build enough context to make informed routing decisions.

1. Run 1–3 rounds of WebSearch on the topic.
2. Prioritize authoritative and recent sources.
3. Extract: topic scope, key terminology, existing state-of-the-art, open questions, recency requirements.
4. If the user hasn't specified, clarify: timeframe, constraints, desired output format, depth.

**Exit condition:** You have enough context to (a) explain the topic in 2–3 sentences and (b) identify what types of knowledge are needed (breadth, depth, recency, synthesis, execution).

### Phase 2 — Routing Decision

Based on Phase 1 findings, decide the dispatch plan. Read `references/agent-selection-matrix.md` for capability profiles and decision heuristics.

**Decision dimensions:**

| Dimension | Signal from Phase 1 |
|---|---|
| Breadth exploration | Topic spans multiple domains or subfields |
| Technical depth | Needs code, architecture, or implementation detail |
| Reasoning quality | Requires careful analysis, tradeoffs, nuance |
| Recency / live-web | Needs information from last days/weeks |
| Synthesis quality | Final output needs polished writing or structured comparison |
| Long-context handling | Source material is large (papers, docs, codebases) |
| Execution-oriented | Needs actionable steps, not just analysis |
| Ambiguity reduction | Topic is vague; needs hypothesis generation first |

**Routing output (present to user):**

```
## Routing Decision

**Topic:** {1-sentence summary}
**Knowledge needs:** {which dimensions matter}
**Dispatch plan:** {single / parallel / serial}

| Step | Agent | Role | Why this agent |
|------|-------|------|----------------|
| 1 | {agent} | {role} | {1-sentence justification} |
| 2 | {agent} | {role} | {1-sentence justification} |

**Expected workflow:** {brief description of how results flow}
```

Ask user to confirm or adjust before proceeding.

### Phase 3 — Prompt Generation

Generate high-quality prompts for each agent in the dispatch plan. Read `references/prompt-generation-guide.md` for the prompt structure and quality bar.

Every generated prompt MUST include:
1. **Role & objective** — what the agent should act as and accomplish
2. **Context summary** — key findings from Phase 1 (not raw dumps)
3. **Exact questions** — numbered, specific, deep (not keywords)
4. **Boundaries** — what to exclude, time constraints, scope limits
5. **Evidence requirements** — citations, sources, confidence signals
6. **Output format** — structure, length, organization
7. **Acceptance criteria** — checkboxes the user can verify
8. **Handoff instructions** (if serial) — what downstream agents need from this output

Present all prompts to the user. For each prompt, briefly note which agent it targets and why.

### Phase 4 — Orchestration (if multi-agent)

For multi-agent plans, read `references/orchestration-patterns.md` for coordination templates.

**Responsibilities:**
- Specify execution order (parallel groups, serial dependencies)
- Define what each agent's output feeds into the next step
- Provide a synthesis prompt for the final agent if applicable
- Include a fallback plan if an agent underperforms

**After prompts are delivered:**
- User dispatches prompts to external agents
- When results return, hand off to `agent-research` for quality gating (Step 4 of that skill)
- If results are insufficient, loop back to Phase 3 with adjusted prompts or Phase 2 with adjusted routing

## Edge Cases

| Case | Action |
|------|--------|
| Phase 1 search fully answers the question | Tell user; skip delegation. No agents needed |
| User already knows which agent to use | Skip Phase 2; go directly to prompt generation |
| Topic too vague after Phase 1 | Ask user to narrow scope before routing |
| User wants to add/change agents mid-workflow | Re-enter Phase 2 with updated constraints |
| Serial workflow: early agent fails quality gate | Re-route that step to a different agent or adjust prompt |
| No clear agent advantage for the topic | Default to single best-reasoner; note uncertainty |

## Maintenance

- **Version:** 1.0.0
- **Created:** 2026-03-10
