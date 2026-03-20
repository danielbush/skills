---
status: active
created: 2026-03-17
summary: Generalize the jsed vocab + architecture narrative + orientation pattern into a reusable skill that bootstraps shared understanding for any codebase
parent: ../../docs/discussion-driven-development.md
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

## Grounding: vocabulary terms need key examples

A definition tells you what to call something; an example tells you why it matters. Every vocabulary term should be paired with a **key example** — the one concrete instance that makes the concept click.

The agent can help with this in several ways:
- **Find** the best existing example in the codebase for a term
- **Construct** a throwaway test or snippet that isolates the concept (can be discarded — the understanding is what stays)
- **Contrast** — show what happens *without* the concept (e.g. remove the DELAYED_INSTANTIATION and watch it break)

This suggests a `/explain <TERM>` command: given a vocabulary term, find or construct the most illustrative example in the codebase. Not a definition — a grounded demonstration.

## Automatic orientation via AGENTS.md routing

Rather than requiring the human to invoke `/jsed` (or `/understand`) to load orientation, hardwire it into each package's AGENTS.md. The root AGENTS.md acts as a router:

> "If you're working in `packages/foo/`, read `packages/foo/AGENTS.md` and orient yourself."

Each package AGENTS.md then contains (or points to) the vocabulary, architecture narrative, and entry-point questions for that package. Orientation happens automatically based on where the work is — no skill invocation needed.

This means the orientation skill's artifacts (vocab, narrative, entry-point questions) live in the package itself, not as a separate skill the human has to remember to call.

## Next steps

1. Review the jsed reference implementation to extract the repeatable pattern
2. Draft the skill SKILL.md with all three modes (`init`, `update`, `explain`)
3. Test on a second codebase (not jsed) to validate generalizability
4. Design the AGENTS.md routing pattern for workspaces with multiple packages
