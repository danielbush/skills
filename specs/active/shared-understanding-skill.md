---
status: active
created: 2026-03-17
summary: Generalize the jsed vocab + architecture narrative + orientation pattern into a reusable skill that bootstraps shared understanding for any codebase
parent: ../discussion/discussion-driven-development.md
theme: "Theme 1: Shared Understanding"
---

## Goal

Create a skill that helps human and agent build shared understanding of a codebase, modeled on what exists for jsed.

## Artifacts to produce (per codebase)

1. `docs/vocabulary.md` — UPPER_SNAKE_CASE domain terms
2. `docs/architecture.md` — bottom-up narrative (each section depends only on what came before)
3. AGENTS.md updates — point to the above, explain deep modules structure
4. Orientation skill — reads the above in order, summarizes for a new session

## Key question to resolve first

Is this a **bootstrapping** skill (agent scans code and proposes vocab + narrative) or a **collaborative** skill (agent assists human in naming things and writing the narrative session by session)?

The jsed experience suggests collaborative — terms like FOCUSABLE and SHALLOW_TOKENIZATION emerged from discussions, not from static analysis. But a bootstrapping pass could give the human something to react to.

**Proposed answer:** Both. The skill has two modes:
- `/understand init` — agent reads the code, proposes a draft vocab and narrative for human review
- `/understand update` — after a session where new concepts emerged, agent proposes additions

## Next steps

1. Review the jsed reference implementation to extract the repeatable pattern
2. Draft the skill SKILL.md with both modes
3. Test on a second codebase (not jsed) to validate generalizability
