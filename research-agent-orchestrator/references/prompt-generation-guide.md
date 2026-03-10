# Prompt Generation Guide

How to generate high-quality prompts for external research agents. Prompt quality is the single biggest lever on research output quality.

## Prompt Structure (Mandatory Sections)

Every research prompt MUST contain these sections, in order:

### 1. Role & Objective

Set the agent's persona and goal in 2–3 sentences.

```
You are a {domain} research analyst. Your task is to {specific objective}
with emphasis on {key priority}. Prioritize {quality dimension} over {lesser priority}.
```

**Rules:**
- Be specific about the domain — "frontend build tooling expert" not "technical researcher"
- State the objective as a deliverable — "produce a comparison of X, Y, Z" not "research X"
- Name the priority tradeoff — helps the agent allocate effort

### 2. Context Summary

Provide the condensed output from Phase 1 foundation search. This prevents the agent from wasting time on basics you already know.

**Rules:**
- 3–8 sentences of distilled context, not raw search dumps
- Frame as "what we already know" to set a higher starting point
- Include specific versions, dates, or numbers found during Phase 1
- Mention what authoritative sources were already consulted

**Example:**
```
Context: We've already reviewed the official Turborepo docs (v2.3, Jan 2026)
and confirmed it uses content-addressable storage for local caching in
node_modules/.cache/turbo. Vercel offers a hosted Remote Cache, but we
need to understand self-hosted alternatives for air-gapped environments.
The GitHub repo shows 15.2k stars with active maintenance.
```

### 3. Exact Questions

Numbered, specific questions that map directly to your research gaps.

**Rules:**
- 2–5 questions per prompt (more dilutes focus)
- Each question should be answerable independently
- Frame questions at the level of expertise you expect — show depth
- Avoid yes/no questions — prefer "how", "what tradeoffs", "compare"

**Bad vs Good:**

| Bad | Good |
|---|---|
| "What is X?" | "How does X handle Y under Z conditions?" |
| "Tell me about caching" | "In monorepos with 100+ packages, what are the common cache invalidation failures?" |
| "Compare A and B" | "Compare A and B on {dimension 1} and {dimension 2}, with examples from projects over {size}" |

### 4. Boundaries & Exclusions

Prevent the agent from going off-track.

```
Boundaries:
- Focus on versions released after {date}
- Exclude {topic} — already covered separately
- Do NOT provide basic introductions or tutorials
- Limit scope to {specific context, e.g., "self-hosted deployments"}
```

### 5. Evidence Requirements

Tell the agent how to support its claims.

```
Evidence requirements:
- Cite sources with URLs for every factual claim
- Distinguish between official documentation, community experience, and inference
- For community experience, note the date and project scale
- Flag any claims that are your own reasoning rather than sourced
```

### 6. Output Format

Be explicit about structure and length.

```
Output format:
- Answer each question as a separate section with a clear heading
- Start each answer with a 2–3 sentence summary, then provide detail
- Use tables for comparisons
- Total length: {approximate word count or page count}
- Include a "Sources" section at the end with all referenced URLs
```

### 7. Acceptance Criteria

Checkboxes the user can verify. These should mirror the criteria from `agent-research`'s brief format for compatibility.

```
Acceptance criteria:
- [ ] {Criterion 1 — specific, binary-verifiable}
- [ ] {Criterion 2}
- [ ] {Criterion 3}
```

**Rules:**
- 3–6 criteria per prompt
- Each must be binary (pass/fail) — no "try to" or "as much as possible"
- At least one criterion should require a concrete artifact (code snippet, config example, data point)

### 8. Handoff Instructions (Serial Workflows Only)

When this agent's output feeds into a downstream agent:

```
Handoff note: Your output will be consumed by a downstream analyst for
{purpose}. Structure your output so that {specific requirement — e.g.,
"each comparison can be extracted as a standalone section"}.
Do not include: {what the downstream agent doesn't need}.
```

## Agent-Specific Tailoring

Adapt the prompt style to the target agent's strengths:

| Agent | Tailoring |
|---|---|
| ChatGPT | Emphasize structured output format; it follows format instructions well. Can request markdown tables, numbered lists, specific headings. |
| Gemini | For large-context tasks, explicitly tell it to reference specific sections of uploaded docs. Be precise about what to extract vs. summarize. |
| Grok | Emphasize recency; ask for timestamps on information. Request social/community signal summaries. Keep prompts shorter and more focused. |
| Claude | Can handle more nuanced instructions. Emphasize tradeoffs, caveats, and edge cases. Good with "think through X before answering." |
| Manus | Frame as step-by-step workflows. Be explicit about what to browse, what to collect, what to output. More procedural than analytical prompts. |

## Quality Self-Check

Before delivering a prompt to the user, verify:

- [ ] Role is domain-specific, not generic
- [ ] Context summary saves the agent at least 1 search round
- [ ] Every question is specific enough that two experts would interpret it the same way
- [ ] Boundaries prevent the most likely off-track directions
- [ ] Evidence requirements specify source types expected
- [ ] Output format is concrete enough to verify compliance
- [ ] Acceptance criteria are all binary-verifiable
- [ ] If serial: handoff instructions specify what downstream needs
- [ ] Prompt is tailored to the specific target agent's style
- [ ] Total prompt length is under 800 words (longer prompts dilute focus)
