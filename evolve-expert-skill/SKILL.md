---
name: evolve-expert-skill
description: >-
  Use when evolving or updating an existing expert skill. Two modes:
  Distill (captures experience from current conversation into experiences/)
  and Evolve (searches sources for updates to references/).
  Auto-detects mode based on conversation context.
  Triggers: "record to xx-expert", "distill experience", "save to expert",
  "evolve xx-expert", "update xx-expert", "xx-expert outdated",
  "expert skill outdated", "refresh expert knowledge",
  "save this to xx-expert", "remember this in xx-expert".
---

# Evolve Expert Skill

## Overview

Incrementally evolve an existing expert skill. Two modes auto-detected:
- **Distill**: conversation has relevant context -> extract experience -> write to `experiences/`
- **Evolve**: no context / user says "update" -> search sources for updates -> refresh `references/`

Both modes update SKILL.md trigger keywords (self-evolution).

## Mode Detection

1. Check target `{name}-expert/` directory exists. If not -> error, suggest `create-expert-skill`.
2. Check current conversation: does it contain problem-solving or discussion about the target tool?
   - YES -> Distill mode. Confirm: "Detected experience about {tool}. Enter Distill mode?"
   - NO -> Evolve mode.

## Distill Mode

Read `references/distill-guide.md` for the full 4-step workflow:
D1 Experience Extraction -> D2 Dedup Check -> D3 Write -> D4 Trigger Evolution

## Evolve Mode

Read `references/evolve-guide.md` for the full 5-step workflow:
E1 Version Probe -> E2 Incremental Fetch -> E3 Incremental Distill -> E4 New Source Discovery -> (Optional) Agent Research -> E5 Trigger Evolution

After E4, if coverage gaps remain, offer `agent-research` for deeper research via external agents. User can skip this (e.g., `--no-research`) to keep evolve fully automated.

## Dedup Strategy

Read `references/dedup-strategy.md` for tag-matching + semantic fallback rules.

## Edge Cases

| Case | Action |
|------|--------|
| Target skill doesn't exist | Error: "Use create-expert-skill first" |
| experiences/ has 20+ entries | Suggest housekeeping: merge similar, archive outdated to `_archived/` |
| Tool discontinued | Mark all sources `deprecated`, add warning to SKILL.md |
| No version change found | Inform user, ask if force-update desired |

## Living Document

Current version: 1.0.0
