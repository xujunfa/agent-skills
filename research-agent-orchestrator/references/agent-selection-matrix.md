# Agent Selection Matrix

Capability profiles and routing heuristics for external research agents.

> **Freshness notice:** Treat these profiles as heuristics, not fixed truth. Model capabilities, access methods, and platform features change frequently. Before routing a high-stakes task, verify current capabilities of the target platform — especially for recency, context limits, and tool access. Last reviewed: 2026-03.

## Capability Profiles

### ChatGPT

| Dimension | Rating | Notes |
|---|---|---|
| Breadth exploration | Strong | Wide training data, good at surveying a field |
| Technical depth | Strong | Solid code generation and architecture reasoning |
| Reasoning quality | Good | Reliable for structured analysis |
| Recency / live-web | Strong | Built-in web browsing with source citations |
| Synthesis quality | Strong | Excels at structured summaries and comparisons |
| Long-context handling | Good | Large context window; handles long inputs well (verify current limits) |
| Execution-oriented | Good | Clear step-by-step outputs |
| Ambiguity reduction | Good | Good at generating structured options |

**Best for:** Structured synthesis, survey-style research, well-scoped technical questions, producing organized deliverables.

### Gemini

| Dimension | Rating | Notes |
|---|---|---|
| Breadth exploration | Strong | Google Search integration, broad knowledge |
| Technical depth | Good | Solid but less consistent on niche technical topics |
| Reasoning quality | Good | Competitive on reasoning benchmarks; improving rapidly |
| Recency / live-web | Strong | Native Google Search, strong for recent information |
| Synthesis quality | Good | Adequate but typically less polished than ChatGPT/Claude |
| Long-context handling | Very strong | Among the largest context windows available (verify current limits) |
| Execution-oriented | Good | Reasonable at action plans |
| Ambiguity reduction | Good | Can digest large amounts of conflicting info |

**Best for:** Digesting large document sets (papers, codebases, long docs), broad web surveys, recency-sensitive topics, cross-referencing many sources.

### Grok

| Dimension | Rating | Notes |
|---|---|---|
| Breadth exploration | Good | Decent general knowledge |
| Technical depth | Good | Adequate for most technical topics |
| Reasoning quality | Good | Reasonable analysis capability |
| Recency / live-web | Very strong | Real-time X/social media access, very recent web data |
| Synthesis quality | Adequate | Less polished long-form output |
| Long-context handling | Good | Context limits have expanded significantly; verify current limits before routing large-document tasks |
| Execution-oriented | Adequate | Better at information gathering than action planning |
| Ambiguity reduction | Good | Good at surfacing diverse viewpoints from social signals |

**Best for:** Real-time / social signal research, trending topics, sentiment analysis, very recent events, community pulse on technical topics.

### Claude

| Dimension | Rating | Notes |
|---|---|---|
| Breadth exploration | Good | Strong general knowledge |
| Technical depth | Very strong | Excellent code reasoning and technical analysis |
| Reasoning quality | Very strong | Strong nuanced analysis and tradeoffs |
| Recency / live-web | Varies | Depends on access method; standard chat has no built-in browsing, but tool-augmented deployments may differ |
| Synthesis quality | Very strong | Excellent long-form writing and structured output |
| Long-context handling | Strong | Large context window; very faithful to long inputs (verify current limits) |
| Execution-oriented | Strong | Clear, actionable outputs with good judgment |
| Ambiguity reduction | Very strong | Excels at exploring nuance, caveats, tradeoffs |

**Best for:** Deep reasoning, tradeoff analysis, nuanced synthesis, technical writing, final-stage synthesis from pre-gathered material, tasks requiring careful judgment.

### Manus

| Dimension | Rating | Notes |
|---|---|---|
| Breadth exploration | Good | Can browse and interact with web pages |
| Technical depth | Good | Can execute code and interact with tools |
| Reasoning quality | Good | Adequate analysis |
| Recency / live-web | Strong | Active web browsing and page interaction |
| Synthesis quality | Adequate | Functional but less polished prose |
| Long-context handling | Good | Handles multi-step workflows well |
| Execution-oriented | Very strong | Can browse, code, create files, run workflows |
| Ambiguity reduction | Good | Can iteratively explore and refine |

**Best for:** Execution-heavy research (scraping, data collection, comparison tables from live sites), multi-step web workflows, tasks requiring tool use beyond text generation.

## Routing Decision Tree

```
START: What does the research primarily need?

├─ Very recent info (days/weeks)?
│   ├─ Social signals / trending → Grok
│   ├─ General recent web → ChatGPT or Gemini
│   └─ Need to interact with live sites → Manus
│
├─ Digest large documents (>50 pages)?
│   └─ Gemini (long context advantage)
│
├─ Deep reasoning / tradeoff analysis?
│   └─ Claude
│
├─ Structured survey of a field?
│   └─ ChatGPT (synthesis strength)
│
├─ Execution-heavy (scraping, data collection, tool use)?
│   └─ Manus
│
├─ Multiple dimensions needed?
│   ├─ Breadth + Depth → Gemini (gather) → Claude (analyze)
│   ├─ Recency + Synthesis → Grok (gather) → ChatGPT or Claude (synthesize)
│   ├─ Survey + Reasoning → ChatGPT (survey) → Claude (deep analysis)
│   └─ Data + Analysis → Manus (collect) → Claude (synthesize)
│
└─ No clear winner?
    └─ Pick the agent with best general availability and broadest capability for the user's access (no fixed default — assess per-task)
```

## Single vs Multi-Agent Decision

| Signal | Recommendation |
|---|---|
| Clear single-dimension need | Single agent |
| Tight deadline, user wants simplicity | Single agent |
| Need both recency AND deep analysis | Serial: recency agent → reasoning agent |
| Need multiple independent perspectives | Parallel: 2-3 agents on same questions |
| Complex topic with gather + analyze phases | Serial: gather agent → synthesize agent |
| User has access to only one external agent | Single agent (obviously) |
| Contradictions expected in source material | Parallel agents → comparison synthesis |

## Anti-Patterns

- **Don't use Grok for deep technical synthesis** — it's a recency/signal tool, not a reasoning engine.
- **Don't assume Claude always — or never — has web browsing** — capability depends on the deployment and access method. Verify before routing; pre-gather material if browsing is unavailable.
- **Don't send the same generic prompt to all agents** — tailor each prompt to the agent's strengths.
- **Don't default to multi-agent when single suffices** — complexity has a cost (user effort, time).
- **Don't hardcode "Agent X is always best for Y"** — capabilities evolve; use these as heuristics, not rules.
