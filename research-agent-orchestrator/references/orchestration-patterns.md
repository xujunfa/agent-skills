# Orchestration Patterns

Reusable multi-agent coordination templates. Each pattern describes when to use it, the agent flow, and how to handle the handoff between stages.

## Pattern 1: Gather → Synthesize (Serial)

**When:** Need both breadth/recency AND deep analysis. One agent gathers raw material, another produces the final deliverable.

**Flow:**
```
Agent A (gatherer) → raw findings → Agent B (synthesizer) → final output
```

**Common instantiations:**

| Variant | Gatherer | Synthesizer | Use case |
|---|---|---|---|
| Recency + Depth | Grok | Claude | Topic needs latest social/web signals + careful analysis |
| Breadth + Reasoning | Gemini | Claude | Large doc corpus needs digestion + tradeoff analysis |
| Survey + Polish | ChatGPT | Claude | Structured survey needs deeper reasoning pass |
| Data + Analysis | Manus | ChatGPT or Claude | Live data collection needs structured interpretation |

**Handoff format:**
The gatherer's prompt should end with:
```
Structure your output as a numbered list of findings. For each finding include:
- Claim (1 sentence)
- Evidence (source URL + key quote)
- Confidence (high/medium/low)
- Date of information

Do not synthesize or draw conclusions — that will be done in a separate analysis step.
```

The synthesizer's prompt starts with:
```
Below are raw research findings gathered by another analyst on {topic}.
Your task is to synthesize these into {deliverable description}.
Resolve contradictions, identify patterns, and produce {output format}.
```

## Pattern 2: Parallel Perspectives (Fan-out)

**When:** Want independent perspectives on the same question from multiple agents, then combine. Good for reducing bias or catching blind spots.

**Flow:**
```
Agent A ─┐
Agent B ─┼→ Manual comparison or synthesis agent
Agent C ─┘
```

**When to use:**
- Controversial or subjective topic where different agents may have different biases
- High-stakes decision needing validation from multiple sources
- Topic where you suspect one agent may have training data gaps

**Prompt rules:**
- Send the SAME core questions to each agent
- Tailor framing/format to each agent's strengths (see prompt-generation-guide.md)
- Do NOT tell agents about each other — keep perspectives independent

**Synthesis step:**
After collecting results, either manually compare or generate a synthesis prompt:
```
You have received independent research reports on {topic} from three analysts.
Report A focuses on: {summary}
Report B focuses on: {summary}
Report C focuses on: {summary}

Your task:
1. Identify points of agreement across all reports
2. Identify contradictions and assess which source is more credible
3. Identify unique insights that only one report contains
4. Produce a unified analysis that incorporates the best of all three
```

## Pattern 3: Hypothesis → Validation (Serial)

**When:** Topic is ambiguous. Use one agent to generate hypotheses/options, then another to validate or stress-test them.

**Flow:**
```
Agent A (explorer) → hypotheses/options → Agent B (validator) → validated conclusions
```

**Common instantiations:**

| Explorer | Validator | Use case |
|---|---|---|
| ChatGPT | Claude | Generate solution options → deep tradeoff analysis |
| Grok | ChatGPT | Surface emerging trends → structured evaluation |
| Gemini | Claude | Broad literature scan → critical assessment |

**Explorer prompt addition:**
```
For each option/hypothesis you identify:
- State the hypothesis clearly
- List supporting evidence
- List potential weaknesses or counter-evidence
- Rate your confidence (high/medium/low)

Present at least 3 distinct options. Do not pre-select a winner.
```

**Validator prompt addition:**
```
An analyst has identified the following hypotheses for {topic}:
{hypotheses from explorer}

For each hypothesis:
1. Assess the strength of the supporting evidence
2. Identify what would need to be true for this hypothesis to hold
3. Identify the most likely failure mode
4. Provide your independent confidence rating with justification

Then recommend which hypothesis is strongest and why.
```

## Pattern 4: Broad Scan → Contradiction Check → Final Synthesis (3-Stage)

**When:** Complex, high-stakes topic where accuracy matters more than speed.

**Flow:**
```
Stage 1: Agent A (broad scan) → comprehensive findings
Stage 2: Agent B (contradiction check) → verified findings + flagged conflicts
Stage 3: Agent C (synthesis) → final deliverable
```

**Typical agents:** Gemini or ChatGPT → Grok (recency cross-check) → Claude (synthesis)

**Stage 2 prompt pattern:**
```
A comprehensive research scan on {topic} produced the following findings:
{Stage 1 output}

Your task is a verification pass:
1. Cross-reference key claims against current web sources
2. Flag any claims that appear outdated (check dates)
3. Flag any claims that contradict each other
4. Add any significant recent developments not covered
5. Mark each finding as: ✅ verified, ⚠️ needs update, ❌ contradicted

Do NOT rewrite the findings. Annotate them.
```

## Pattern Selection Guide

| Your situation | Recommended pattern | Why |
|---|---|---|
| Need latest info + deep analysis | Gather → Synthesize | Separates recency from reasoning |
| Want to reduce single-agent bias | Parallel Perspectives | Independent viewpoints |
| Topic is vague, need to explore options | Hypothesis → Validation | Structured exploration |
| High-stakes, accuracy critical | 3-Stage | Maximum verification |
| Simple, well-scoped question | No pattern (single agent) | Don't over-engineer |
| Tight deadline | Single agent or Gather → Synthesize | Minimize coordination overhead |

## Coordination Rules

1. **Each agent's prompt is self-contained.** Never assume an agent can access another agent's output unless you explicitly paste it.
2. **Trim between stages.** Don't paste raw Stage 1 output into Stage 2 verbatim if it's very long. Summarize or extract the relevant parts.
3. **Preserve source URLs across stages.** When trimming, keep citation links intact.
4. **Budget user effort.** Each stage requires the user to copy-paste. Prefer 2-stage over 3-stage unless accuracy demands it.
5. **Fail fast.** If Stage 1 results are poor, don't proceed to Stage 2. Re-route via Phase 2 of the main workflow instead.
